## Azure Application Gateway(Level 7)
- web traffic load balancer that enables you to manage traffic to your web applications.(仅限相同的azure region)

Web application firewall是其中一个feature

![alt text](image-49.png)

*概念图：*

![alt text](image-19.png)

1. **创建application gateway:**

    ![alt text](image-20.png)

    Network创建一个AG subnet和backend subnet：

    ![alt text](image-21.png)

    给application gateway创建一个frontend ip：
    
    ![alt text](image-22.png)

2. **添加backend pool：**
    将image和video app service加入到分别的两个backend pool中：

    ![alt text](image-86.png)

    ![alt text](image-87.png)

3. **创建routing rule：**
    创建routing rule，创建一个listner监听http 80端口的流量：

    ![alt text](image-24.png)

    以及设定backend targets：

    先创建一个http settings，target端口是80，request time-out是20秒等：

    ![alt text](image-88.png)

    使用这个http settings以及选择backend pool为images：

    ![alt text](image-89.png)

    设定Path-based routing(二级路由，原本默认是到images)，根据http request URL的路径去route到不同的backend pool:

    ![alt text](image-90.png)

    ![alt text](image-91.png)

    此时实现了以下：

    ![alt text](image-92.png)

4. Allow vnet of application gateway traffic to the app service:
    create a service endpoint of Microsoft.web in gateway network, make sure the traffic go through the Microsoft backbond network to app service:

    ![alt text](image-94.png)

    Allow traffic from gateway vnet to app service:

    ![alt text](image-93.png)
    
    But still we are getting a gateway error when reaching the IP of application gateway:

    ![alt text](image-95.png)

    Need to add a health probe:

    ![alt text](image-96.png)

5. Add a health probe:
    host是app service的URL：

    ![alt text](image-97.png)

    ![alt text](image-98.png)

    此时访问，还是收到error，invalid host name：

    ![alt text](image-99.png)

    去修改http setting配置，将override hostname改成yes，并自动选择beckend pool host的域名：

    ![alt text](image-100.png)

    此时，报错又发生改变：

    ![alt text](image-101.png)
    
    原因：

    ![alt text](image-103.png)
    
    把URL路径改完整，成功访问： 

    ![alt text](image-102.png)