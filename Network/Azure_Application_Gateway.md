## Azure Application Gateway(Level 7)
- web traffic load balancer that enables you to manage traffic to your web applications.(仅限相同的azure region)

*概念图：*

![alt text](image-19.png)

1. **创建application gateway:**

    ![alt text](image-20.png)

    Network创建一个AG subnet和backend subnet：

    ![alt text](image-21.png)

    给application gateway创建一个frontend ip：
    
    ![alt text](image-22.png)

    未来可以创建backend pool：

    ![alt text](image-23.png)

    创建routing rule，创建一个listner监听http 80端口的流量：

    ![alt text](image-24.png)

    以及backend pool：

    ![alt text](image-25.png)

2. 将vm加入到backend pool：

    ![alt text](image-26.png)