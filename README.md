# ZBNetworking    [介绍文档](http://www.jianshu.com/p/55cda3341d11)
 
 本来想发布到cocoaPods的发现名字已经被占用了，也不想改名了。大家就手动下载用吧

[变更日志](https://github.com/Suzhibin/ZBNetworking/blob/master/CHANGELOG)
 
优点:

1.请求类型丰富 /**GET请求*//**POST请求*//**PUT请求*//**PATCH请求*//**DELETE请求*//**Upload请求*//**DownLoad请求*/

2.低耦合，易扩展。

3.通过Block配置信息，有Block回调，delegate回调 ,支持公共配置;

4.有插件机制  可以统一 预处理 所有 请求,响应,错误 处理逻辑的方法

5.内存缓存，沙盒缓存，有缓存文件过期机制 默认一周

6.显示缓存大小/个数，全部清除缓存/单个文件清除缓存/按时间清除缓存/按路径清除缓存  方法多样  并且都可以自定义路径   可扩展性强

7.有缓存key过滤功能

8.离线下载功能 ，批量请求功能

9.多种请求缓存类型的判断。也可不遵循，自由随你定。

```objective-c
  /**
     重新请求:   不读取缓存，不存储缓存
     没有缓存需求的，单独使用
     */
    ZBRequestTypeRefresh,
    
    /**
     重新请求:   不读取缓存，但存储缓存
     可以与 ZBRequestTypeCache 配合使用
     */
    ZBRequestTypeRefreshAndCache,
    /**
     读取缓存:   有缓存,读取缓存--无缓存，重新请求并存储缓存
     可以与ZBRequestTypeRefreshAndCache 配合使用
     */
    ZBRequestTypeCache,
    /**
     重新请求：  上拉加载更多业务，不读取缓存，不存储缓存
     用于区分业务 可以不用
     */
    ZBRequestTypeRefreshMore,
```
10.可见的缓存文件

![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/uls*nG1YySR.EpyYI8*lFu9kW.lwzjgW.cnPbGMUBG8!/b/dPgAAAAAAAAA&bo=aAHwAAAAAAACDLE!&rf=viewer_4)

## 使用 
#### 公共配置
```objective-c
 /**
     基础配置
     需要在请求之前配置，设置后所有请求都会带上 此基础配置
     */
    NSMutableDictionary *parameters = [NSMutableDictionary dictionary];
    parameters[@"github"] = @"https://github.com/Suzhibin/ZBNetworking";
    parameters[@"jianshu"] = @"https://www.jianshu.com/p/55cda3341d11";
    NSTimeInterval timeInterval = [[NSDate date] timeIntervalSince1970];
    NSString *timeString = [NSString stringWithFormat:@"%.2f",timeInterval];
    parameters[@"timeString"] =timeString;//时间戳

    NSMutableDictionary *headers = [NSMutableDictionary dictionary];
    headers[@"Token"] = @"Token";
    
    [ZBRequestManager setupBaseConfig:^(ZBConfig * _Nullable config) {
        config.baseURL=server_URL;//如果同一个环境，有多个域名 不建议设置baseURL
        config.parameters=parameters;//公告参数
        // filtrationCacheKey因为时间戳是变动参数，缓存key需要过滤掉 变动参数,如果 不使用缓存功能 或者 没有变动参数 则不需要设置。
        config.filtrationCacheKey=@[@"timeString"];
        config.headers=headers;//请求头
        config.requestSerializer=ZBJSONRequestSerializer; //全局设置 请求格式 默认JSON
        config.responseSerializer=ZBJSONResponseSerializer; //全局设置 响应格式 默认JSON
        config.timeoutInterval=15;//超时时间  优先级 小于 单个请求重新设置
        config.retryCount=2;//请求失败 所有请求重新连接次数
        config.consoleLog=YES;//开log
        config.responseContentTypes=@[@"text/aaa",@"text/bbb"];//添加新的响应数据类型
        /**
         内部已存在的响应数据类型
         @"text/html",@"application/json",@"text/json", @"text/plain",@"text/javascript",@"text/xml",@"image/*",@"multipart/form-data",@"application/octet-stream",@"application/zip"
         */
    }];
```
#### 插件机制
```
    /**
       插件机制
       自定义 所有 请求,响应,错误 处理逻辑的方法

       比如 1.自定义缓存逻辑 感觉ZBNetworking缓存不好，想使用yycache 等
           2.自定义响应逻辑 服务器会在成功回调里做 返回code码的操作
           3.一个应用有多个服务器地址，可在此进行配置
           4.统一loading 等UI处理
           5. ......
       */
    [ZBRequestManager setRequestProcessHandler:^(ZBURLRequest * _Nullable request, id  _Nullable __autoreleasing * _Nullable setObject) {
         NSLog(@"请求之前");
    }];
    
    [ZBRequestManager setResponseProcessHandler:^id(ZBURLRequest * _Nullable request, id  _Nullable responseObject, NSError * _Nullable __autoreleasing * _Nullable error) {
        NSLog(@"成功回调 数据返回之前");
       
    }];
    [ZBRequestManager setErrorProcessHandler:^(ZBURLRequest * _Nullable request, NSError * _Nullable error) {
        if (error.code==NSURLErrorCancelled){
            NSLog(@"请求取消❌------------------");
        }else if (error.code==NSURLErrorTimedOut){
            NSLog(@"请求超时");
        }else{
            NSLog(@"请求失败");
        }
    }];
```
#### Block 请求方法
```
//请求方法 会默认创建缓存路径    
    NSMutableDictionary *parameters = [NSMutableDictionary dictionary];
    parameters[@"path"] = @"HomeViewController";
    NSMutableDictionary *headers = [NSMutableDictionary dictionary];
    headers[@"headers"] = @"herader";
    [ZBRequestManager requestWithConfig:^(ZBURLRequest *request){
        request.URLString=list_URL;
        request.methodType=ZBMethodTypeGET;//默认为GET
        request.apiType=ZBRequestTypeRefresh;//（默认为ZBRequestTypeRefresh 不读取缓存，不存储缓存）
        request.parameters=parameters;
        request.headers=headers;
        /**
         保留第一次或最后一次请求结果 只在请求时有用  读取缓存无效果。默认ZBResponseKeepNone 什么都不做
         使用场景是在 重复点击造成的 多次请求，如发帖，评论，搜索等业务
         */
        request.keepType=ZBResponseKeepNone;
        request.filtrationCacheKey=@[@""];//与basefiltrationCacheKey 兼容
        request.requestSerializer=ZBJSONRequestSerializer; //单次请求设置 请求格式 默认JSON，优先级大于 全局设置，不影响其他请求设置
        request.responseSerializer=ZBJSONResponseSerializer; //单次请求设置 响应格式 默认JSON，优先级大于 全局设置,不影响其他请求设置
        request.retryCount=2;//请求失败 所有请求重新连接次数
        request.timeoutInterval=10;//默认30 //优先级 高于 全局设置,不影响其他请求设置
      
    }  success:^(id responseObj,ZBURLRequest * request){
        if (request.apiType==ZBRequestTypeRefresh) 
             //结束刷新
        }
        if (request.apiType==ZBRequestTypeLoadMore) {
            //结束上拉加载
        }
        //请求成功
          NSLog(@"得到数据:%@",responseObject);
      
    } failure:^(NSError *error){
    }];

```
#### Delegate 请求方法
```
  [ZBRequestManager requestWithConfig:^(ZBURLRequest *request) {
       request.URLString=listUrl;
       request.apiType=type;
  } target:self];//ZBRequestDelegate
  
#pragma mark - ZBURLRequestDelegate
- (void)request:(ZBURLRequest *)request successForResponseObject:(id)responseObject{
        if (request.apiType==ZBRequestTypeRefresh) 
             //结束刷新
        }
        if (request.apiType==ZBRequestTypeLoadMore) {
            //结束上拉加载
        }
        //请求成功
          NSLog(@"得到数据:%@",responseObject);
}
- (void)request:(ZBURLRequest *)request failedForError:(NSError * _Nullable)error{
}
- (void)request:(ZBURLRequest *)request forProgress:(NSProgress *)progress{
    NSLog(@"onProgress: %.f", 100.f * progress.completedUnitCount/progress.totalUnitCount);
}
- (void)request:(ZBURLRequest *)request finishedForResponseObject:(id)responseObject forError:(NSError *)error{
//    NSLog(@"code:%ld",error.code);
//    NSLog(@"URLString:%@",request.URLString);
}

```
#### 批量下载
```objective-c
 [ZBRequestManager sendBatchRequest:^(ZBBatchRequest *batchRequest)
            for (NSString *urlString in offlineArray) {
            ZBURLRequest *request=[[ZBURLRequest alloc]init];
            request.URLString=urlString;
            [batchRequest.urlArray addObject:request];
        }
    }  success:^(id responseObj,ZBURLRequest * request){
    } failure:^(NSError *error){
    } finished:^(NSArray * _Nullable responseObjects, NSArray<NSError *> * _Nullable errors, NSArray<ZBURLRequest *> * _Nullable requests) {
            NSLog(@"批量完成事件");
    }];

//具体演示看demo
```
![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/cY8K3L2*GJ9RO3i*z1If9XTmzas0cylmafMXWqdFe4o!/b/dK0AAAAAAAAA&bo=aAHwAAAAAAACLJE!&rf=viewer_4)

#### 取消请求
```
 //取消当前请求
 [ZBRequestManager cancelRequest:identifier];
 //取消批量请求
 [ZBRequestManager cancelBatchRequest:batchRequest];
 //取消所有请求
 [ZBRequestManager cancelAllRequest];
  ```
#### 缓存相关
```objective-c
 //显示缓存大小 可以自定义路径
 [[ZBCacheManager sharedInstance]getCacheSize];
  //显示缓存个数  可以自定义路径
 [[ZBCacheManager sharedInstance]getCacheCount];
 //清除沙盒缓存
 [[ZBCacheManager sharedInstance]clearCache];
 //清除内存缓存
 [[ZBCacheManager sharedInstance]clearMemory];
  //清除单个缓存文件
 [[ZBCacheManager sharedInstance]clearCacheForkey:list_URL];
  //按路径清除缓存
 [[ZBCacheManager sharedInstance]clearDiskWithpath:@"路径" completion:nil];
 ```

![](https://upload-images.jianshu.io/upload_images/1830250-3636c0621ebb6fa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/621)

#### 缓存key过滤
 ```
    NSDate* dat = [NSDate dateWithTimeIntervalSinceNow:0];
    NSTimeInterval a=[dat timeIntervalSince1970];
    NSString *timeString = [NSString stringWithFormat:@"&time=%f", a];
    
     // 使用了parameters 的请求 缓存key会是URLString+parameters，parameters里有是时间戳或者其他动态参数,key一直变动 无法拿到缓存。所以定义一个 filtrationCacheKey 过滤掉parameters 缓存key里的 变动参数比如 时间戳
    [ZBRequestManager requestWithConfig:^(ZBURLRequest *request){
        request.URLString=@"http://URL";
        request.methodType=ZBMethodTypePOST;//默认为GET
        request.apiType=ZBRequestTypeRefresh;//默认为ZBRequestTypeRefresh
        request.parameters=@{@"1": @"one", @"2": @"two", @"time": @"12345667"};
        request.filtrationCacheKey=@[@"time"];//过滤掉time
    }success:nil failure:nil];
  ```


## License

ZBNetworking is released under the MIT license. See LICENSE for details.
