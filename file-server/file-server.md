
文件服务器是基于Minio搭建的，我们访问控制台：（http://10.177.96.147:9000/minio），可以看到文件服务器的整个概览情况；


![minio-console](/image/fileserver/minio-console.png)


包括桶列表，磁盘情况，点击任意一个桶，可以查看桶内的文件对象;

# 存储桶
具体每个桶存储的作用如下：


> * account-rsa，存储区块链账号公钥
> * agency-image，存储机构认证图片相关信息
> * chain-ca，联盟链证书
> * csr，机构证书
> * material-package，节点安装物料包
> * pub-key，机构，链公钥
> * chain-log，节点日志



# 静态资源管理

针对平台logo和协议等静态资源，我们可以放到OSS上去，访问OSS控制：http://10.177.96.147:9000/minio

创建静态资源baas桶：


![create-bucket](/image/fileserver/create-bucket.png)


点击对应的 bucket ，edit policy 添加策略 *.*，Read Only，如下：


![bucket-privilege](/image/fileserver/bucket-privilege.png)




上传静态资源：

![/upload-files](/image/fileserver/upload-files.png)

如此就放开了访问，没有时间限制，同时只需要按http://${MINIO_HOST}:${MINIO_PORT}/${bucketName}/${fileName} 则可直接访问资源（不需要进行分享操作）。



# 相关问题
## 文件无法上传

针对文件无法上传，首先查看 相关的日志文件，minio.out.log，查看是否有响应的错误日志；


```
API: PutObjectPart(bucket=material-package, object=node-1018/10.177.96.148_33334.tar.gz)
Time: 14:02:35 CST 09/22/2020
DeploymentID: 6ae98293-6b99-4531-8672-10e0cfc632b9
RequestID: 163705C986BA1267
RemoteHost: 10.177.96.148
Host: 10.177.96.147:9000
UserAgent: MinIO (amd64; amd64) minio-java/dev
Error: disk path full
       6: github.com/minio/minio@/cmd/fs-v1-helpers.go:294:cmd.fsCreateFile()
       5: github.com/minio/minio@/cmd/fs-v1-multipart.go:321:cmd.(*FSObjects).PutObjectPart()
       4: github.com/minio/minio@/cmd/object-handlers.go:2250:cmd.objectAPIHandlers.PutObjectPartHandler()
       3: github.com/minio/minio@/cmd/http-tracer.go:128:cmd.Trace()
       2: github.com/minio/minio@/cmd/handler-utils.go:357:cmd.httpTraceHdrs.func1()
       1: net/http/server.go:2036:http.HandlerFunc.ServeHTTP()
```


这种情况一般是由于磁盘空间不足的问题导致。可以通过如下命令查看磁盘使用情况：



```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G   18G  1.4G  93% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  400M  3.5G  11% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vdc         59G   36G   21G  64% /srv/nbs/0
tmpfs           782M     0  782M   0% /run/user/0
tmpfs           782M     0  782M   0% /run/user/8205
tmpfs           782M     0  782M   0% /run/user/9097
tmpfs           782M     0  782M   0% /run/user/1519
tmpfs           782M     0  782M   0% /run/user/20757
```
解决方式有两种，一种扩容磁盘空间，或者删除节点物料包桶或日志桶的相关文件对象，尽量选择比较旧的文件。



针对集群模式，如果有节点的磁盘空间不足，则抛出如下异常


```
UserAgent: MinIO (amd64; amd64) minio-java/dev
Error: Write failed. Insufficient number of disks online
       6: github.com/minio/minio@/cmd/erasure-encode.go:100:cmd.(*Erasure).Encode()
       5: github.com/minio/minio@/cmd/erasure-multipart.go:370:cmd.erasureObjects.PutObjectPart()
       4: github.com/minio/minio@/cmd/erasure-sets.go:1071:cmd.(*erasureSets).PutObjectPart()
       3: github.com/minio/minio@/cmd/erasure-zones.go:1406:cmd.(*erasureZones).PutObjectPart()
       2: github.com/minio/minio@/cmd/object-handlers.go:2250:cmd.objectAPIHandlers.PutObjectPartHandler()
       1: net/http/server.go:2036:http.HandlerFunc.ServeHTTP()
```



