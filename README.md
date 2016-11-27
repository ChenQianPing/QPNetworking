# QPNetworking
iOS封装网络请求接口

# Example Usage

```
#import "QPNetworking.h"

@property QPNetworking *qPNetworking;

// 得到单例
_qPNetworking = [QPNetworking getInstance];
//将_qPNetworking的代理与ViewController连接
_qPNetworking.delegate = self;

- (void)requestSuccessFeedback:(id)feedbackInfo api:(NSString *)api {
    if ([api isEqualToString:@"getGroupListByPhone"]) {
        NSLog(@"相等");
    }
    else
    {
        NSLog(@"不相等");
    }
}

- (void)requestFailFeedback:(id)failInfo api:(NSString *)api {
}
```

- QPNetworking.h
```
#import <Foundation/Foundation.h>
#import "AFNetworking.h"

// 代理
@protocol QPNetworkingDelegate <NSObject>

/**
 * 代理回调方法:一个请求成功、一个请求失败。
 *
 * @param feedbackInfo 服务器返回的数据
 */
- (void)requestSuccessFeedback:(id)feedbackInfo api:(NSString *)api;
- (void)requestFailFeedback:(id)failInfo api:(NSString *)api;

@end

@interface QPNetworking : NSObject

// Delegate的核心的作用就是来实现类之间的数据传递
@property (nonatomic,strong) id<QPNetworkingDelegate> delegate;


/**
 * 获取Net类的单例
 *
 * @return Net类的单例 实例（对象）
 */
+ (QPNetworking *)getInstance;


/**
 * 1.根据用户Id取得部门列表
 */
- (void)getGroupListByPhone;

/**
 * 2.根据部门id查找企业通讯录中的联系人列表
 */
- (void)selectBeanProjectMember: (NSString *)proid;


@end
```

- QPNetworking.m
```
#import "QPNetworking.h"
#import "UrlDefine.h"

__strong static AFHTTPSessionManager *AFHTTPMgr;
__strong static QPNetworking *NetInstance = nil;

@implementation QPNetworking

+ (QPNetworking *)getInstance {
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken,^{
        // 初始化实例
        NetInstance = [[QPNetworking alloc] init];
        AFHTTPMgr = [AFHTTPSessionManager manager];
        
        // AFHTTPMgr.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/html"];
        // AFHTTPMgr.responseSerializer = [AFHTTPResponseSerializer serializer];
        
        // 设置超时时间为10秒
        [AFHTTPMgr.requestSerializer willChangeValueForKey:@"timeoutInterval"];
        AFHTTPMgr.requestSerializer.timeoutInterval = 10.f;
        [AFHTTPMgr.requestSerializer didChangeValueForKey:@"timeoutInterval"];
        
    });
    return NetInstance;
}

/**
 * 1.根据用户Id取得部门列表
 */
- (void)getGroupListByPhone {
    
    NSUserDefaults *userDefault =[NSUserDefaults standardUserDefaults];
    NSDictionary *dic = [userDefault objectForKey:@"userDatas"];
    NSDictionary *parameter = [NSDictionary dictionaryWithObjectsAndKeys:dic[@"user_phone"], @"phone", nil];
    
    
    [AFHTTPMgr POST:GRTGROUPLIST parameters:parameter progress:nil success:^(NSURLSessionDataTask * _Nonnull operation, id  _Nonnull responseObject) {
        // 请求成功Block,将返回数据传入代理方法
        [self.delegate requestSuccessFeedback:responseObject api:@"getGroupListByPhone"];
        
    } failure:^(NSURLSessionDataTask  * _Nullable operation, NSError * _Nonnull error) {
        // 请求失败Block,将错误信息传入代理方法
        [self.delegate requestFailFeedback:error api:@"getGroupListByPhone"];
    }];    
}

/**
 * 2.根据部门id查找企业通讯录中的联系人列表
 */
- (void)selectBeanProjectMember: (NSString *)proid {
    
    NSDictionary *parameter = [NSDictionary dictionaryWithObjectsAndKeys:proid, @"proid", nil];
    [AFHTTPMgr POST:SELECTMEMBER parameters:parameter progress:nil success:^(NSURLSessionDataTask * _Nonnull operation, id  _Nonnull responseObject) {
        [self.delegate requestSuccessFeedback:responseObject api:@"selectBeanProjectMember"];
        
    } failure:^(NSURLSessionDataTask  * _Nullable operation, NSError * _Nonnull error) {
        [self.delegate requestFailFeedback:error api:@"selectBeanProjectMember"];
    }];
}



@end
```
