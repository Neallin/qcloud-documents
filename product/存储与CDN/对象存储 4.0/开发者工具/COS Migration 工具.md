## 功能说明
COS Migration 是一个集成了 COS 数据迁移功能的一体化工具。通过简单的配置操作，用户可以将源地址数据快速迁移至 COS 中，它具有以下特点：
- 丰富的数据源
   - 本地数据：将本地存储的数据迁移到 COS。

   - 其他云存储：目前支持 AWS S3，阿里云 OSS，七牛存储迁移至 COS，后续会不断扩展。

   - URL 列表：根据指定的 URL 下载列表进行下载迁移到 COS。
   
   - Bucket 相互复制：COS 的 Bucket 数据相互复制, 支持跨账号跨地域的数据复制。

- 断点续传

- 分块上传 

- 并行上传

- 迁移校验

## 使用环境
### 系统环境
Linux 或 Windows 环境

### 软件依赖
- JDK1.7 或以上, 有关 JDK 的安装与配置请参考 [Java 安装与配置](https://cloud.tencent.com/document/product/436/10865)。

## 使用方法
### 1. 获取工具
下载链接：[COS Migration 工具](https://github.com/tencentyun/cos_migrate_tool_v5)。

### 2. 解压缩工具包
#### Windows
 解压并保存到某个目录，例如
<pre>
C:\Users\Administrator\Downloads\cos_migrate
</pre>

#### Linux
解压并保存到某个目录
<pre>
unzip cos_migrate_tool_v5-master.zip && cd cos_migrate_tool_v5-master
</pre>

#### 迁移工具结构
正确解压后的 COS Migration 工具目录结构如下所示：
<pre>
COS_Migrate_tool
|——conf  #配置文件所在目录
|   |——config.ini  #迁移配置文件
|——db    #存储迁移成功的记录
|——dep   #程序主逻辑编译生成的JAR包
|——log   #工具执行中生成的日志
|——opbin #用于编译的脚本
|——src   #工具的源码
|——tmp   #临时文件存储目录
|——pom.xml #项目配置文件
|——README  #说明文档
|——start_migrate.sh  #Linux 下迁移启动脚本
|——start_migrate.bat #Windows 下迁移启动脚本
</pre>

>**说明：**
 - db 目录主要记录工具迁移成功的文件标识，每次迁移任务会优先对比 db 中的记录，若当前文件标识已被记录，则会跳过当前文件，否则进行文件迁移。
 - log 目录记录着工具迁移时的所有日志，若在迁移过程中出现错误，请先查看该目录下的 error.log。

### 3. 修改 config.ini 配置文件
在执行迁移启动脚本之前，需先进行 config.ini 配置文件修改（路径：`./conf/config.ini`），config.ini 内容可以分为以下几部分：

#### 3.1 配置迁移类型
type 表示迁移类型，用户根据迁移需求填写对应的标识。例如，需要将本地数据迁移至 COS，则`[migrateType]`的配置内容是`type=migrateLocal`。
<pre>[migrateType]
type=migrateLocal
</pre>

目前支持的迁移类型如下：

| migrateType | 描述 |
| ------| ------ |
| migrateLocal| 从本地迁移至 COS |
| migrateAws| 从 AWS S3 本地迁移至 COS |
| migrateAli| 从阿里  OSS  迁移至 COS |
| migrateQiniu| 从七牛迁移至 COS |
| migrateUrl| 下载 URL 迁移到 COS |
| migrateBucketCopy| 从源 Bucket 复制到目标 Bucket|

#### 3.2 配置迁移任务
用户根据实际的迁移需求进行相关配置，主要包括迁移至目标 COS 信息配置及迁移任务相关配置。
<pre>
# 迁移工具的公共配置分节，包含了要迁移到得目标 COS 的账户信息 
[common]
secretId=AKIDXXXXXXXXXXXXXXXXX
secretKey=GYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
bucketName=mybcket-1251668577
region=ap-guangzhou
storageClass=Standard
cosPath=/
https=off
tmpFolder=./tmp
smallFileThreshold=5242880
smallFileExecutorNum=64
bigFileExecutorNum=8
entireFileMd5Attached=on
daemonMode=off
daemonModeInterVal=60
executeTimeWindow=00:00,24:00
</pre>

| 名称 | 描述 |默认值|
| ------| ------ |----- |
| secretId| 用户密钥 SecretId，可在 [云 API 密钥控制台](https://console.cloud.tencent.com/cam/capi) 查看 |-|
| secretKey| 用户密钥 SecretKey，可在 [云 API 密钥控制台]( https://console.cloud.tencent.com/cam/capi) 查看|-|
| bucketName| 目的 Bucket 的名称, 合法命名规则为 {name}-{appid}，即 Bucket 名必须包含 APPID，例如 movie-1251000000 |-|
| region| 目的 Bucket 的 Region 信息。COS 的地域简称请参照 [可用地域](https://cloud.tencent.com/document/product/436/6224) |-|
| storageClass|存储类型：Standard - 标准存储，Standard_IA - 低频存储 |Standard|
| cosPath|要迁移到的 COS 路径。**/** 表示迁移到 Bucket 的根路径下，**/aaa/bbb/** 表示要迁移到 Bucket的 /aaa/bbb/ 下，若 /aaa/bbb/ 不存在，则会自动创建路径|/|
| https| 是否使用 HTTPS 传输：on 表示开启，off 表示关闭。开启传输速度较慢，适用于对传输安全要求高的场景|off|
| tmpFolder|从其他云存储迁移至 COS 的过程中，用于存储临时文件的目录，迁移完成后会删除。要求格式为绝对路径：<br>Linux 下分隔符为单斜杠，如 /a/b/c； <br>Windows 下分隔符为两个反斜杠，如E:\\\a\\\b\\\c。<br>默认为工具所在路径下的 tmp 目录|./tmp|
| smallFileThreshold| 小文件阈值的字节，大于等于这个阈值使用分块上传，否则使用简单上传，默认 5MB |5242880|
| smallFileExecutorNum|小文件（文件小于 smallFileThreshold）的并发度，使用简单上传。如果是通过外网来连接 COS，且带宽较小，请减小该并发度|64|
| bigFileExecutorNum| 大文件（文件大于等于 smallFileThreshold）的并发度，使用分块上传。如果是通过外网来连接 COS，且带宽较小，请减小该并发度|8|
| entireFileMd5Attached|表示迁移工具将全文的 MD5 计算后，存入文件的自定义头部 x-cos-meta-md5 中，用于后续的校验，因为 COS 的分块上传的大文件的 etag 不是全文的 MD5|on|
| daemonMode|是否启用 damon 模式：on 表示开启，off 表示关闭。damon 表示程序会循环不停的去执行同步，每一轮同步的间隔由 damonModeInterVal 参数设置|off|
| daemonModeInterVal|表示每一轮同步结束后，多久进行下一轮同步，单位为秒 |60|
| executeTimeWindow|执行时间窗口，时刻粒度为分钟，该参数定义迁移工具每天执行的时间段。例如：<br>参数03:30,21:00, 表示在凌晨03:30到晚上21:00之间执行任务，其他时间则会进入休眠状态，休眠态暂停迁移并会保留迁移进度, 直到下一个时间窗口自动继续执行|00:00,24:00|

#### 3.3 配置数据源信息
根据`[migrateType]`的迁移类型配置相应的分节。例如`[migrateType]`的配置内容是`type=migrateLocal`, 则用户只需配置`[migrateLocal]`分节即可。

**3.3.1 配置本地数据源 migrateLocal**

若从本地迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre>
# 从本地迁移到COS配置分节
[migrateLocal]
localPath=E:\\code\\java\\workspace\\cos_migrate_tool\\test_data
exeludes=
ignoreModifiedTimeLessThanSeconds=
</pre>

| 配置项 | 描述 |
| ------| ------ |
|localPath|本地路径，要求格式为绝对路径：<br>Linux 下分隔符为单斜杠，如 /a/b/c； <br>Windows 下分隔符为两个反斜杠，如E:\\\a\\\b\\\c。|
|excludes| 要排除的目录或者文件的绝对路径，表示将 localPath 下面某些目录或者文件不进行迁移，多个绝对路径之前用分号分割，不填表示 localPath 下面的全部迁移|
|ignoreModifiedTimeLessThanSeconds| 排除更新时间与当前时间相差不足一定时间段的文件，单位为秒, 默认不设置, 表示不根据lastmodified时间进行筛选, 适用于客户在更新文件的同时又在运行迁移工具, 并要求不把正在更新的文件迁移上传到COS, 比如设置为300, 表示只上传更新了5分钟以上的文件|

**3.3.2 配置阿里 OSS 数据源 migrateAli**

若从阿里云 OSS 迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre># 从阿里 OSS 迁移到 COS 配置分节
[migrateAli]
bucket=mybucket-test
accessKeyId=xxxxxxxxxx
accessKeySecret=yyyyyyyyyyy
endPoint= OSS -cn-shenzhen.aliyuncs.com
prefix=
proxyHost=
proxyPort=
</pre>

| 配置项 | 描述 |
| ------| ------ |
|bucket|阿里云 OSS  Bucket 名称|
|accessKeyId|用户的密钥 accessKeyId |
|accessKeySecret| 用户的密钥 accessKeySecret|
|endPoint|阿里云 endpoint 地址|
|prefix|要迁移的路径的前缀, 如果是迁移 Bucket 下所有的数据, 则 prefix 为空|
|proxyHost|如果要使用代理进行访问，则填写代理 IP 地址|
|proxyPort|代理的端口|

**3.3.3 配置AWS数据源 migrateAws**

若从 AWS 迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre># 从 AWS 迁移到 COS 配置分节
[migrateAws]
bucket=aws-emr-test
accessKeyId=xxxxxxxxxx
accessKeySecret=yyyyyyyyyyyyyyyy
endPoint=s3.us-east-1.amazonaws.com
prefix=
proxyHost=
proxyPort=
</pre>

| 配置项 | 描述 |
| ------| ------ |
|bucket| AWS 对象存储 Bucket 名称|
|accessKeyId|用户的密钥 accessKeyId |
|accessKeySecret| 用户的密钥 accessKeySecret|
|endPoint|AWS 的 endpoint 地址,  必须使用域名, 不能使用region|
|prefix|要迁移的路径的前缀, 如果是迁移 Bucket下所有的数据, 则 prefix 为空|
|proxyHost|如果要使用代理进行访问，则填写代理 IP 地址|
|proxyPort|代理的端口|

 
**3.3.4 配置七牛数据源 migrateQiniu**

若从七牛迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre># 从七牛迁移到COS配置分节
[migrateQiniu]
bucket=mybuckettest
accessKeyId=xxxxxxxxxx
accessKeySecret=yyyyyyyyyyyyyyyy
endPoint=wwww.bkt.clouddn.com
prefix=
proxyHost=
proxyPort=
</pre>

| 配置项 | 描述 |
| ------| ------ |
|bucket|七牛对象存储 Bucket 名称|
|accessKeyId|用户的密钥 accessKeyId |
|accessKeySecret| 用户的密钥 accessKeySecret|
|endPoint|七牛下载地址，对应 downloadDomain|
|prefix|要迁移的路径的前缀, 如果是迁移 Bucket下所有的数据，则 prefix 为空|
|proxyHost|如果要使用代理进行访问，则填写代理 IP 地址|
|proxyPort|代理的端口|

 
**3.3.5 配置 URL 列表数据源 migrateUrl**

若从指定 URL 列表迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre>
# 从 URL 列表下载迁移到 COS 配置分节
[migrateUrl]
</pre>
     
| 配置项 | 描述 |
| ------| ------ |
|urllistPath|url列表项，要求格式为绝对路径：<br>Linux 下分隔符为单斜杠，如 /a/b/c； <br>Windows 下分隔符为两个反斜杠，如E:\\\a\\\b\\\c。<br>如果填写的是目录，则会将该目录下的所有文件视为 urllist 文件去扫描迁移|

 
**3.3.6 配置 Bucket 相互复制 migrateBucketCopy**

若从指定 URL 列表迁移至 COS，则进行该部分配置，具体配置项及说明如下：
<pre>
# 从源 Bucket 迁移到目标 Bucket 配置分节
[migrateBucketCopy]
srcRegion=ap-shanghai  
srcBucketName=mysrcbucket-1251668555
srcSecretId=xxxxxxxxxxx
srcSecretKey=yyyyyyyyyyyyyyyy
srcCosPath=/
</pre>

| 配置项 | 描述 |
| ------| ------ |
|srcRegion|源 Bucket 的 Region 信息，请参照 [可用地域](https://cloud.tencent.com/document/product/436/6224)|
|srcBucketName|源 Bucket 的名称, 合法命名规则为 {name}-{appid}，即 Bucket 名必须包含 APPID，例如 movie-1251000000|
|srcSecretId|源 Bucket 隶属的用户的密钥 SecretId，可在[云 API 密钥](https://console.cloud.tencent.com/cam/capi) 查看。如果是同一用户的数据，则 srcSecretId 和 common 中的 SecretId 相同，否则是跨账号 Bucket 拷贝。|
|srcSecretKey|源 Bucket 隶属的用户的密钥 secret_key，可在 [云 API 密钥](https://console.cloud.tencent.com/cam/capi) 查看。如果是同一用户的数据，则 srcSecretId 和 common 中的 secretId 相同，否则是跨账号 Bucket 拷贝。|
|srcCosPath|要迁移的 COS 路径，表示该路径下的文件要迁移至目标 Bucket|


### 4. 运行迁移工具
#### Windows
双击 **start_migrate.bat** 即可运行。

#### Linux
1.从config.ini配置文件读入配置，运行命令为：
<pre>
sh start_migrate.sh
</pre>
2.部分参数从命令行读入配置，运行命令为：
<pre>
sh start_migrate.sh -Dcommon.cosPath=/savepoint0403_10/
</pre>

>** 特别说明**
> - 工具支持配置项读取方式有两种：命令行读取或配置文件读取。

> - 命令行优先级高于配置文件，即相同配置选项会优先采用命令行里的参数。

> - 命令行中读取配置项的形式方便用户同时运行不同的迁移任务，但前提是两次任务中的关键配置项不完全一样，例如 Bucket 名称，COS 路径，要迁移的源路径等。因为不同的迁移任务写入的是不同的 db 目录，可以保证并发迁移。请参照前文中的工具结构中的 db 信息。
    
> - 配置项的形式为 **-D{sectionName}.{sectionKey}={sectionValue}** 的形式。其中 sectionName 是配置文件的分节名称，sectionKey 表示分节中配置项名称，sectionValue 表示分节中配置项值。如设置要迁移到的 COS 路径，则以 **-Dcommon.cosPath=/bbb/ddd** 表示。

## 迁移机制及流程
### 迁移机制原理
COS 迁移工具是有状态的，已经迁移成功的会记录在 db 目录下，以 KV 的形式存储在 leveldb 文件中。每次迁移前对要迁移的路径，先查找下 db 中是否存在， 如果存在，且属性和 db 中存在的一致， 则跳过迁移，否则进行迁移。这里的属性根据迁移类型的不同而不同，对于本地迁移，会判断 mtime。对于其他云存储迁移与 Bucket 复制，会判断源文件的 etag 和长度是否与 db 一致。因此，我们参照 db 中是否有过迁移成功的记录，而不是查找 COS，如果绕过了迁移工具，通过别的方式（如 COSCMD 或者控制台）删除修改了文件，那么运行迁移工具由于不会察觉到这种变化，是不会重新迁移的。

### 迁移流程步骤
1. 读取配置文件，根据迁移 type，读取响应的配置分节，并执行参数的检查。

2. 根据指定的迁移类型，扫描对比 db 下对所要迁移文件的标识，判断是否允许上传。

3. 迁移执行过程中会打印执行结果，其中 inprogress 表示迁移中，skip 表示跳过，fail 表示失败，ok 表示成功, condition_not_match表示因为表示因不满足迁移条件而跳过的文件(如lastmodifed和excludes)。失败的详细信息可以在 log 的 error 日志中查看。执行过程示意图如下图所示：
 ![](https://main.qcloudimg.com/raw/7561d07ea315c9bacbb228b36d6ad6d6.png)

4. 整个迁移结束后会打印统计信息，包括累积的迁移成功量，失败量，跳过量，耗时。对于失败的情况，请查看 error 日志，或重新运行，因为迁移工具会跳过已迁移成功的，对未成功的会跳过。运行完成结果示意图如下图所示：
![](https://main.qcloudimg.com/raw/2534fd390218db29bb03f301ed2620c8.png)


## 常见问题
#### 1. 迁移工具中途异常退出怎么办？ 
工具支持上传时断点续传, 对于一些大文件, 如果中途退出或者因为服务故障, 可重新运行工具, 会对未上传完的文件进行续传。

#### 2. 对于迁移成功的文件，用户通过控制台或其他方式删除了 COS 上的文件，迁移工具会将这些文件进行重新上传吗？
不会。原因是，所有迁移成功的文件会被记录在 db 中，迁移工具运行之前会先扫描 db 目录，对于已被记录的文件不会再次上传，具体原因请参照 [迁移机制及流程](#.E8.BF.81.E7.A7.BB.E6.9C.BA.E5.88.B6.E5.8F.8A.E6.B5.81.E7.A8.8B)。

#### 3. 迁移失败，日志显示 403 Access Deny，该怎么办？
请确认密钥信息，Bucket 信息，Region 信息是否正确，并且是否具有操作权限。如果是子账号，请让父账号授予相应的权限；如果是本地迁移和其他云存储迁移，需要对 Bucket 具有数据写入和读取权限；如果是 Bucket copy，还需要对源 Bucket 具有数据读取权限。

#### 4. 从其他云存储迁移 COS 失败，显示 Read timed out，该怎么办？
一般来说，这种失败情况是由网络带宽不足所造成，导致从其他云存储下载数据超时。比如，将 AWS 海外的数据迁移到 COS，在下载数据到本地时由于带宽能力不足，导致时延较高，可能会出现 read time out。因此，解决方法为增大机器的网络带宽能力，建议在迁移之前用 wget 测试下载速度。

#### 5. 迁移失败，日志显示 503 Slow Down，该怎么办？
这是触发频控所导致，COS 目前对一个账号具有每秒 800 QPS 的操作限制。建议调小配置中小文件的并发度,，并重新运行工具，则会将失败的重新运行。

#### 6. 迁移失败，日志显示 404 NoSuchBucket，该怎么办？
请确认您的密钥信息，Bucket 信息，Region 信息是否正确。

#### 7. 其他问题
请重新运行迁移工具，若仍然失败，请将配置信息（密钥信息请隐藏）与 log 目录打包后 [提交工单](https://console.cloud.tencent.com/workorder/category)。
