## Azure Front door(Level 7)
- define, manage and monitor the global routing for your web traffic

Web application firewall是其中一个feature(可以限速)
https://learn.microsoft.com/en-us/azure/web-application-firewall/afds/waf-front-door-rate-limit-configure?pivots=portal
Lab:
## 1. Create two webapps(different regions)

![alt text](image-27.png)

![alt text](image-28.png)

## 2. Create Azure front door.

![alt text](image-29.png)

![alt text](image-30.png)

添加frontend host(front door),自定义域名

![alt text](image-31.png)

把创建的web app添加到backend：

![alt text](image-32.png)

![alt text](image-33.png)

![alt text](image-34.png)

添加routing rule：

![alt text](image-35.png)

![alt text](image-36.png)

## 3. Test front door:

关掉一个webapp，front door会自动failover到另一个webapp

![alt text](image-37.png)