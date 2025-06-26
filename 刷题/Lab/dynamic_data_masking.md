## 🔍 实际例子：使用 Dynamic Data Masking（DDM）

你有一张 `Customers` 表：

```sql
+----+------------+-------------------+
| ID | Name       | CreditCard        |
+----+------------+-------------------+
| 1  | John Doe   | 4111-1111-1111-1111
| 2  | Jane Smith | 5000-0000-0000-0000
```
你想让普通用户看不到完整卡号，只显示部分内容。


## ✅ 设置 DDM 掩码规则：
```sql
ALTER TABLE Customers  
ALTER COLUMN CreditCard 
ADD MASKED WITH (FUNCTION = 'partial(0,"XXXX-XXXX-",4)');
```

## 🛠️ 实际使用步骤（在 Azure SQL 中配置 Dynamic Data Masking）

1. 登录 [Azure Portal](https://portal.azure.com/)
2. 找到你的 **Azure SQL 数据库实例**
3. 在左侧导航中点击 **“Dynamic Data Masking”**
4. 点击 `+ Add mask` 添加字段遮罩规则，例如：
   - `Email` → 使用 `email()` 函数
   - `CreditCard` → 使用 `partial(0,"XXXX-XXXX-",4)` 函数
   - `SSN` → 使用 `default()` 或 `random()`
5. 配置 **例外用户（Privileged Users）**
   - 添加可以看到真实数据的 SQL 登录用户或 AAD 身份
   - 未被列入例外的用户将看到遮罩结果
6. 点击 **“Save”** 保存配置

📌 **备注：**
- DDM 是“即席遮罩”，不会修改数据库中的原始数据
- 只影响查询结果的可视化，不影响应用功能或性能（基本）
