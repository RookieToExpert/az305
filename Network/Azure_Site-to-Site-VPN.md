## Site-to-Site VPN
*Overview*

1. Use az CLI to create on-pre and cloud resources:

    ![alt text](image-63.png)

    ![alt text](image-64.png)

#### Pre-requistes for cloud:
1. Enable Web service for cloud server:

    为了让日后测试网络是否联通更加简单

    ![alt text](image-65.png)

    Web server works:

    ![alt text](image-66.png)

2. Create virtual network gateway:
    可以需要有一个单独的gateway subnet:

    ![alt text](image-67.png)

    需要有一个public IP:

    ![alt text](image-68.png)

#### Pre-requistes for on-premise:
1. Enable remote access(routing) service for on-premise server:
    让它变成一个local gateway device：

    ![alt text](image-69.png)

    deploy VPN only:

    ![alt text](image-70.png)

    Custom configuration:

    ![alt text](image-71.png)

    Demand-dial connections and LAN routing:

    ![alt text](image-72.png)

2. Create local network gateway:
    第一个ip是local gateway dedvice的ip(也就是前面的client VM的IP)，第二个网络段代表的是本地网络段

    ![alt text](image-73.png)

#### Set up the connection
1. Associate the local gateway to the cloud gateway:

    ![alt text](image-74.png)

    ![alt text](image-75.png)

2. Associate the cloud gateway to the local gateway:

    ![alt text](image-76.png)

    ![alt text](image-77.png)

    ![alt text](image-78.png)

    ![alt text](image-79.png)

    Paste the IP of the cloud gateway:

    ![alt text](image-80.png)

    ![alt text](image-81.png)

    Cloud IP address:

    ![alt text](image-82.png)
    
    keep empty for now:

    ![alt text](image-83.png)

    Enter the preshared key for connection:

    ![alt text](image-84.png)

#### Test
可以在本地网络直接用cloud VM的private IP访问，证明S2S VPN配置成功：

![alt text](image-85.png)