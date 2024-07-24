## 版本變化

1. 2019年之前，包括2019年，界面風格相對簡單。數據以原始文件方式存儲。





2. 2020-2022年5月份，包括22年5月份，界面風格開始變得豐富。數據仍以原始文件方式存儲。另外，部署時需要指定Console 端口，在compose 中使用方式：

   ```yaml
       command: server /data --console-address ":9001"
   ```

   

3. 2022年6月份之後，包括6月份，界面風格與上面相差無幾。但是數據存儲方式完全與以往不同，是以文件名做目錄名，來存儲具體的數據的，而非直接以源文件存儲的。

## 數據同步

### Minio 版本一致

直接拷貝Minio 服務器上面的數據到現場環境對應數據目錄下即可。注意，除了需要拷貝 **桶名** 目錄外，還需要同時拷貝隱藏目錄：`.minio.sys`。



### Minio 版本不一致

核心原理：

藉助Minio 原生客戶端來調用Minio 服務器端口來完成數據同步。



操作過程：

```shell
# 下載客戶端mc

# 添加執行權限
chmod +x mc


# 本地定義新舊Minio 服務器
# 請將這裡的Minio 服務器api地址、用戶名、密碼信息替換為自己的 
./mc alias set oldminio http://192.168.19.137:9000 minioadmin minioadmin

./mc alias set newminio http://192.168.19.137:19000 minioadmin minioadmin


# 查看新舊服務器上面所有的對象桶名稱
./mc ls oldminio

./mc ls newminio


# 遷移目標桶的數據
./mc mirror oldminio/demo newminio/demo2

# 查看新MINIO 桶內被轉移過來的數據
./mc ls newminio/demo2
```



