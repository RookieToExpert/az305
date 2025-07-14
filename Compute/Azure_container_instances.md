## Azure Container Instance


## Lab创建一个web application:
1. Create Container Instance:

    compute:

    ![alt text](image-14.png)

    network:

    ![alt text](image-15.png)

    Overview page:

    ![alt text](image-16.png)

2. 直接http访问cotainer ip或fqdn就可以访问这个服务：
    IP:

    ![alt text](image-17.png)
    FQDN:

    ![alt text](image-18.png)

## 如何将custom image存入container registry:
1. 创建container registry:

    ![ ](image-19.png)

2. 将这个custom image push到创建的container registry:

    ![alt text](image-20.png)

3. 到container registry查看此镜像：

    ![alt text](image-21.png)