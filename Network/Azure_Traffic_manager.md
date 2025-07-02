## Azure Traffic Manager(Level 4)
- DNS-based traffic load balancer that enables you to distribute traffic optimally to services across global Azure regions

## Routing methods
1. **Performance based method**:

    ![alt text](image-8.png)

2. **Weighted based method**:

    ![alt text](image-9.png)

3. **Geographical based method**:

    ![alt text](image-10.png)

4. **Priority based method**:

    ![alt text](image-11.png)

5. **Subnet based method**:

    ![alt text](image-12.png)

Lab:
**建立两个app和两个client vm分别坐落于North Europe和East US，用Traffic manager导流：**


*概念图如下：*

![alt text](image-14.png)

1. 创建Traffic manager，并且选择routing method：

    ![alt text](image-13.png)

2. 创建endpoints,连接到两个endpoints：

    ![alt text](image-15.png)

    ![alt text](image-16.png)

3. 分别用east us和north europe两台vm去测试，会发现基于performance based method会将clinet导入延迟更低的app：

    
    ![alt text](image-17.png)


4. Routing method可以更改：

    ![alt text](image-18.png)