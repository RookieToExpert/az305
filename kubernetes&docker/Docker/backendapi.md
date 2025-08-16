#### 打包fastapi镜像
1. 创建打包文件：

```mkdir -p /opt/timeapp```

2. 编写fastapi代码：
要点：
- PG/Redis 都是可选：没配也能跑；登录/注册需要 PG，没 PG 会返回 503。

- 访问计数：有 Redis 用 Redis；否则用内存；如果同时有 PG 也会把总数写回 PG。

- 健康检查 /healthz 会告诉你当前 PG/Redis 的连接状态。
```cat >main.py <<'python'
```python
from fastapi import FastAPI, Request, Response, HTTPException
from pydantic import BaseModel, EmailStr
from datetime import datetime, timezone
from zoneinfo import ZoneInfo
import os, bcrypt, jwt, psycopg2, redis
from dotenv import load_dotenv

load_dotenv()

# ---- 配置（通过环境变量）----
JWT_SECRET      = os.getenv("JWT_SECRET", "change_me")
SECURE_COOKIES  = os.getenv("SECURE_COOKIES", "false").lower() == "true"
PG_DSN          = os.getenv("PG_DSN")     # 例如：host=xxxxx.postgres.database.azure.com dbname=appdb user=app@xxxxx password=*** sslmode=require
REDIS_URL       = os.getenv("REDIS_URL")  # 例如：rediss://:<key>@<name>.redis.cache.windows.net:6380/0

# Azure PG 常见：若是 *.postgres.database.azure.com 且未包含 sslmode，自动补上
if PG_DSN and "postgres.database.azure.com" in PG_DSN and "sslmode=" not in PG_DSN:
    PG_DSN = PG_DSN.strip() + " sslmode=require"

app = FastAPI()

# ---- 可选依赖：PG / Redis（允许缺省）----
pg = None
r  = None
memory_counters = {"visits_total": 0}  # 没有 Redis 时的内存计数（进程内，重启丢失）

def try_connect_pg():
    global pg
    if not PG_DSN:
        return None
    try:
        pg = psycopg2.connect(PG_DSN)
        # 初始化表（若不存在）
        with pg.cursor() as cur:
            cur.execute("""
              CREATE TABLE IF NOT EXISTS users(
                id SERIAL PRIMARY KEY,
                email CITEXT UNIQUE NOT NULL,
                password_hash TEXT NOT NULL,
                created_at TIMESTAMPTZ DEFAULT now()
              );
            """)
            cur.execute("""
              CREATE TABLE IF NOT EXISTS site_counters(
                id INT PRIMARY KEY DEFAULT 1,
                total BIGINT NOT NULL DEFAULT 0,
                updated_at TIMESTAMPTZ DEFAULT now()
              );
            """)
            # 确保有 id=1 这一行
            cur.execute("INSERT INTO site_counters(id,total) VALUES (1,0) ON CONFLICT (id) DO NOTHING;")
        pg.commit()
        return pg
    except Exception as e:
        print("PG connect/init failed:", e)
        pg = None
        return None

def try_connect_redis():
    global r
    if not REDIS_URL:
        return None
    try:
        r = redis.Redis.from_url(REDIS_URL, decode_responses=True)
        r.ping()
        return r
    except Exception as e:
        print("Redis connect failed:", e)
        r = None
        return None

try_connect_pg()
try_connect_redis()

# ---- 模型 ----
class Register(BaseModel):
    email: EmailStr
    password: str

class Login(BaseModel):
    email: EmailStr
    password: str

# ---- 健康检查 ----
@app.get("/healthz")
def healthz():
    return {
        "ok": True,
        "pg": bool(pg),
        "redis": bool(r)
    }

# ---- 业务：时间 ----
@app.get("/time/now")
def time_now():
    cities = [
        ("New York", "America/New_York"),
        ("Beijing",  "Asia/Shanghai"),
        ("Sydney",   "Australia/Sydney"),
        ("Delhi",    "Asia/Kolkata"),
    ]
    return {"times": [
        {"label": label, "tz": tz, "iso": datetime.now(ZoneInfo(tz)).isoformat()}
        for label, tz in cities
    ]}

# ---- 业务：注册 / 登录（需要 PG）----
def require_pg():
    global pg
    if not pg:
        # 尝试重连一次
        try_connect_pg()
    if not pg:
        raise HTTPException(status_code=503, detail="PostgreSQL is not configured/available.")

@app.post("/auth/register")
def register(body: Register):
    require_pg()
    pw = bcrypt.hashpw(body.password.encode(), bcrypt.gensalt()).decode()
    with pg.cursor() as cur:
        cur.execute("INSERT INTO users(email,password_hash) VALUES(%s,%s)", (body.email, pw))
    pg.commit()
    return {"ok": True}

@app.post("/auth/login")
def login(body: Login, response: Response):
    require_pg()
    with pg.cursor() as cur:
        cur.execute("SELECT id,password_hash FROM users WHERE email=%s", (body.email,))
        row = cur.fetchone()
    if not row: return {"ok": False}
    uid, hashv = row
    if not bcrypt.checkpw(body.password.encode(), hashv.encode()):
        return {"ok": False}
    token = jwt.encode({"uid": uid}, JWT_SECRET, algorithm="HS256")
    response.set_cookie("access_token", token, httponly=True, samesite="Lax", secure=SECURE_COOKIES)
    return {"ok": True}

# ---- 业务：访问计数（优先 Redis；否则内存；若 PG 可用同时累计 PG.total）----
@app.post("/metrics/visit")
def visit(req: Request):
    # 统计唯一/日计数可以继续扩展；这里做最简单的总量 +（Redis优先，否则内存）
    if r:
        r.incr("visits:total")
    else:
        memory_counters["visits_total"] += 1

    # PG 有的话也累计 site_counters.total
    if pg:
        with pg.cursor() as cur:
            cur.execute("UPDATE site_counters SET total=total+1, updated_at=now() WHERE id=1")
        pg.commit()
    return {"ok": True}

@app.get("/metrics/total")
def total():
    # 先取 Redis
    if r:
        val = r.get("visits:total")
        if val is not None:
            return {"total": int(val)}
    # 没有 Redis，退化用 PG 或内存
    if pg:
        with pg.cursor() as cur:
            cur.execute("SELECT total FROM site_counters WHERE id=1")
            row = cur.fetchone()
            if row:
                return {"total": int(row[0])}
    return {"total": int(memory_counters["visits_total"])}
```

3. 编写requirements：
客户端库都在镜像里安装，但不会随镜像起服务；运行时只依赖外部 PaaS/自建实例。
```vim requirements.txt```
```txt
fastapi==0.111.0
uvicorn[standard]==0.30.0
bcrypt==4.2.0
psycopg2-binary==2.9.9
PyJWT==2.8.0
redis==5.0.7
python-dotenv==1.0.1
```

4. 编写dockerfile：
```vim dockerfile```
```docker
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8000

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY main.py .

# 非 root 运行（安全）
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --retries=3 CMD \
  python -c "import urllib.request,os;urllib.request.urlopen(f'http://127.0.0.1:{os.environ.get(\"PORT\",\"8000\")}/healthz').read()" || exit 1

# 监听 0.0.0.0，端口可被 $PORT 覆盖（Container Apps/容器平台友好）
CMD ["sh","-c","uvicorn main:app --host 0.0.0.0 --port ${PORT}"]
```

#### build和push镜像以及本地测试
1. build镜像
```docker build -t timeapp-api:latest .```

2. 本地测试：
docker run -d --rm -p 8000:8000 \
  -e PORT=8000 \
  -e JWT_SECRET=devsecret \
  -e SECURE_COOKIES=false \
  timeapi:latest

3. 测试api
curl -s http://127.0.0.1:8000/healthz
curl -s http://127.0.0.1:8000/time/now
curl -s -X POST http://127.0.0.1:8000/metrics/visit
curl -s http://127.0.0.1:8000/metrics/total

部署至container app需要的完整环境变量：
PORT=8000
JWT_SECRET=devsecret
SECURE_COOKIES=false

POSTGRES_HOST=xxx.postgres.database.azure.com
POSTGRES_DB=mydb
POSTGRES_USER=myuser@xxx
POSTGRES_PASSWORD=your_password

REDIS_HOST=xxx.redis.cache.windows.net
REDIS_PORT=6380
REDIS_PASSWORD=your_redis_key

