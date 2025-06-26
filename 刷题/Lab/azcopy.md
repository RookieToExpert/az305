## Azcopy

快速从本地复制数据到azure blob/blob to blob等的工具

Lab: on-premise to blob

1. 下载azcopy：

    ![alt text](image-19.png)

2. 生成一个又write permission的SAS URL：

    ![alt text](image-20.png)

3. 跑命令：
    换到azcopy工具下的目录，跑az copy "上传文件的本地路径" "blob SAS URL"

    ![alt text](image-21.png)

    文件上传成功
