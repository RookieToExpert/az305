## Azure Load Balancer(Layer 4)
- provides high-performance, low-latency Layer 4 load-balancing for all UDP and TCP protocols

如果要连接第三方NVA,需要选择gateway load balancer(是一种特殊的azure load balancer的SKU)

## Overview:

![alt text](image-38.png)

## Terms:

![alt text](image-39.png)

## N-tier application:

![alt text](image-40.png)

## Two types of SKUs:

![alt text](image-41.png)

## Lab:

*Overiew project:*

![alt text](image-42.png)

1. Create three VMs in the same VNet:

    ![alt text](image-43.png)

2. Create load balancer:

    ![alt text](image-44.png)

3. Add VMs to backend pool:

    ![alt text](image-45.png)

4. Create health probe:

    ![alt text](image-46.png)

5. Create load balancer rule:

    ![alt text](image-47.png)

    ![alt text](image-48.png)