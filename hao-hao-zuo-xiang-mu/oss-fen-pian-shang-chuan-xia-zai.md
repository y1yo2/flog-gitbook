# OSS-分片上传下载

### 分片下载

[https://help.aliyun.com/document\_detail/84825.html?spm=5176.21213303.J\_6704733920.75.658253c9KMZMq1\&scm=20140722.S\_help%40%40%E6%96%87%E6%A1%A3%40%4084825.S\_0%2Bos.ID\_84825-RL\_oss%20%E8%8C%83%E5%9B%B4%E4%B8%8B%E8%BD%BD-LOC\_main-OR\_ser-V\_2-P0\_16](https://help.aliyun.com/document\_detail/84825.html?spm=5176.21213303.J\_6704733920.75.658253c9KMZMq1\&scm=20140722.S\_help%40%40%E6%96%87%E6%A1%A3%40%4084825.S\_0%2Bos.ID\_84825-RL\_oss%20%E8%8C%83%E5%9B%B4%E4%B8%8B%E8%BD%BD-LOC\_main-OR\_ser-V\_2-P0\_16)

GetObject的请求头

https://help.aliyun.com/document\_detail/31980.html?spm=5176.21213303.J\_6704733920.22.4bb353c9YxCiko\&scm=20140722.S\_help%40%40%E6%96%87%E6%A1%A3%40%4031980.S\_0%2Bos.ID\_31980-RL\_oss%20contentDASrange-LOC\_main-OR\_ser-V\_2-P0\_3

请求头：Range bytes=0-900

响应头：Content-Range

bytes 0-900/18334

### 分片上传

[https://help.aliyun.com/document\_detail/31850.html](https://help.aliyun.com/document\_detail/31850.html)

**使用场景**

*   大文件加速上传

    当文件大小超过5 GB时，使用分片上传可实现并行上传多个Part以加快上传速度。
*   网络环境较差

    网络环境较差时，建议使用分片上传。当出现上传失败的时候，您仅需重传失败的Part。
*   文件大小不确定

    可以在需要上传的文件大小还不确定的情况下开始上传，这种场景在视频监控等行业应用中比较常见。

**流程说明**如下：

1. 将待上传文件按照一定大小进行分片。
2. 使用[InitiateMultipartUpload](https://help.aliyun.com/document\_detail/31992.htm#reference-zgh-cnx-wdb)接口初始化一个分片上传任务。（文件大小不超过48.8 TB；Part数量1\~10, 000个）
3.  使用[UploadPart](https://help.aliyun.com/document\_detail/31993.htm#reference-pnq-2px-wdb)接口上传分片。（单个Part大小最小值为100 KB，最大值为5 GB。最后一个Part的大小无限制。）

    文件切分成Part之后，文件顺序是通过上传过程中指定的`partNumber`来确定，所以您可以并发上传这些碎片。并发数并非越多越快，请结合自身网络状况和设备负载综合考虑。

    如果您希望终止上传任务，可调用[AbortMultipartUpload](https://help.aliyun.com/document\_detail/31996.htm#reference-txp-bvx-wdb)接口，成功上传的Part会一并删除。
4. 使用[CompleteMultipartUpload](https://help.aliyun.com/document\_detail/31995.htm#reference-lq1-dtx-wdb)接口将Part组合成一个Object。

**实践说明**：

分片上传文件

```java
    /**
     * 第一步：初始化分片上传
     *
     * @return 返回
     * fileName 生成的文件名
     * uploadId，它是分片上传事件的唯一标识。您可以根据该uploadId发起相关的操作，例如取消分片上传、查询分片上传等。
     */
    @PostMapping("/part/init")
    @ApiOperation(value = "1.分片上传-初始化分片上传 ")
    @ResponseBody
    public Result<JSONObject> initiateMultipartUpload(@ApiParam(name = "saveFoleer", value = "文件夹可不填写", required = false) @RequestParam(required = false, defaultValue = "") String saveFolder,
                                                      @RequestParam(value = "contentType", required = false, defaultValue = "") String contentType, @RequestParam(value = "fileName", required = true) String fileName) {
        Result<JSONObject> result = new Result<>();
            result = new Result<>(fileStorage.initiateMultipartUpload(saveFolder, contentType, fileName, 0));
            return result;
    }

    /**
     * 第二步：上传分片
     *
     * @param index     当前文件为第几片
     * @param uploadId  uploadId它是分片上传事件的唯一标识，可以根据这个ID来发起相关的操作，如取消分片上传、查询分片上传等。
     * @param key       文件名
     * @param chunkSize 分片大小
     * @return partETag 前端记得存起来，合并分片的时候要用
     * 2.分片上传-上传分片，参数：
     * 分片好的文件(从body里拿)：file
     * 当前文件为第几片(从0开始)：index
     * init接口返回的uploadId：uploadId
     * init接口返回的key：key
     * 当前分片大小：chunkSize
     */
    @PostMapping("/part/upload")
    @ApiOperation(value = "2.分片上传-上传分片 ")
    @ResponseBody
    public Result<JSONObject> uploadPart(
//            @ApiParam(value = "小程序发送文件二进制数据content-type为application/octet-stream")@RequestBody String inputString,
            HttpServletRequest request,
            @ApiParam(value = "当前文件为第几片(从0开始)") @RequestParam("index") Integer index,
            @ApiParam(value = "uploadId它是分片上传事件的唯一标识") @RequestParam("uploadId") String uploadId,
            @ApiParam(value = "文件名") @RequestParam("key") String key,
            @ApiParam(value = "分片大小") @RequestParam("chunkSize") Long chunkSize) throws IOException {
//        PartETag partETag = fileStorage.uploadPart(file, key, uploadId, index+1, chunkSize);
//        byte[] bytes = inputString.getBytes(StandardCharsets.UTF_8);
//        byte[] bytes = inputString.getBytes(StandardCharsets.ISO_8859_1);
//        int len = bytes.length;
        Result<JSONObject> uploadResult = new Result<>();
        try {
            long start = System.currentTimeMillis();
            PartETag partETag = fileStorage.uploadPart(chunkSize, request.getInputStream(), key, uploadId, index + 1, chunkSize);
            JSONObject result = new JSONObject();
            result.put("partNumber", partETag.getPartNumber());
            result.put("partSize", partETag.getPartSize());
            result.put("eTag", partETag.getETag());
            //long 类型过长传（短点就没问题）到前端会出bug（这个bug必现），故转为String。bug 如下
            //后端传出的值：8677056718873628943， 前端接收的值： 8677056718873629000
            result.put("partCRC", partETag.getPartCRC().toString());
            logger.info("uploadPart 耗时: {} 秒, uploadId: {}, fileName:{}, partNumber:{}, partSize:{}", (System.currentTimeMillis() - start) / 1000, uploadId, key, index + 1, chunkSize);
            uploadResult = new Result<>(result);
            return uploadResult;
    }

    /**
     * 第三步：合并分片
     *
     * @return url 文件url
     */
    @GetMapping("/part/merge")
    @ApiOperation(value = "3.分片上传-合并分片 ")
    @ResponseBody
    @PassToken
    public Result<String> uploadPartMerge(
            @ApiParam(value = "uploadId它是分片上传事件的唯一标识") @RequestParam("uploadId") String uploadId,
            @ApiParam(value = "文件名") @RequestParam("key") String key) {
        Result<String> result = new Result<>();
            result = new Result<>(fileStorage.completeMultipartUpload(uploadId, key));
            return result;
    }
```

* 请求体的字符串数据，可通过 `@RequestBody String inputString` 获取；
* 小程序是请求体的二进制数据，通过 `HttpServletRequest request.getInputStream()` 获取；

#### 问题二：小程序无法处理每个分片的partETag，后端用redis的list管理分片信息

```java
/**
在第二步：上传分片后，以uploadId为key，使用redis的list保存本次上传的信息
Redis Lpush 命令	将一个或多个值插入到列表头部
Redis Expire 命令	seconds 为给定 key 设置过期时间。
**/
CacheManager.lpush(partETagKey, partETagJSON.toJSONString());
// 设置1天过期
CacheManager.expire(partETagKey, 60*60*24);

/**
在第三步：合并分片，通过 lrange key 0 -1 ，获取list第一个到最后一个元素
Redis Lrange 命令	获取列表指定范围内的元素
区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素
**/
List<String> partETagStrings = CacheManager.lrange(partETagKey, 0, -1);
CacheManager.expire(partETagKey, 1);

```

版本3.2之前，Redis 列表list使用两种数据结构作为底层实现：

* 压缩列表ziplist：数量比较少使用。存储在一段连续的内存上，所以存储效率很高。操作需要频繁的申请和释放内存。
* 双向链表linkedlist：数量大使用。除了要保存数据之外，还要额外保存两个指针。地址不连续产生碎片。

版本3.2之后，重新引入 quicklist，列表的底层都由quicklist实现。

* quickList是一个ziplist组成的双向链表。每个节点使用ziplist来保存数据。

```c
typedef struct quicklistNode {
    struct quicklistNode *prev; //上一个node节点
    struct quicklistNode *next; //下一个node
    unsigned char *zl;            //保存的数据 压缩前ziplist 压缩后压缩的数据
}
quickList就是一个标准的双向链表的配置，有head 有tail;
每一个节点是一个quicklistNode，包含prev和next指针。
每一个quicklistNode 包含 一个ziplist，*zp 压缩链表里存储键值。
所以quicklist是对ziplist进行一次封装，使用小块的ziplist来既保证了少使用内存，也保证了性能。

```

### 秒传

1. 利用redis的set方法存放文件上传状态，其中key为文件上传的md5，value为是否上传完成的标志位，
2. 当标志位true为上传已经完成，此时如果有相同文件上传，则进入秒传逻辑。如果标志位为false，则说明还没上传完成，此时需要在调用set的方法，保存块号文件记录的路径，其中key为上传文件md5加一个固定前缀，value为块号文件记录路径
