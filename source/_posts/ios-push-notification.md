---
title: ios push & notification
date: 2017-04-19 15:26:55
tags:
categories:
---

# push & notification

摘要： 本文主要介绍iOS push过来的信息的处理，即对notification的响应。同时针对iOS 8、iOS 10不同系统版本，local和remote两种通知类型实际如何处理。并且依据两种通知类型，简单介绍本地推送和apns推送。
<!-- more -->  

## 一、通知权限申请、注册    
     
### 1. register RemoteNotifications    
    
	[[UIApplication sharedApplication] registerForRemoteNotifications]; 
	      
	
### 2. register localNotification    

> iOS10 later      
	
	 ```  
	UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        //必须写代理，不然无法监听通知的接收与点击事件
        center.delegate = self;
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert) completionHandler:^(BOOL granted, NSError * _Nullable error) {
            
        }];
	 ```
	 
> iOS8-10     
	  
	UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
	
    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];   
 ------    

## 二、发送通知    
### 1. remote notification 

系统接收到远程push时，直接发送remote notification   

### 2. local notification         
local notification需要在本地主动触发
   
> iOS8-10   
	 
	 UILocalNotification *notification = [[UILocalNotification alloc] init];
	 
    [notification setSoundName:UILocalNotificationDefaultSoundName];
    
    [notification setAlertBody:content];
    
    [notification setUserInfo:dic];
    
    [[UIApplication sharedApplication] presentLocalNotificationNow:notification];
   
> iOS10 later    
	   
	// 使用 UNUserNotificationCenter 来管理通知
    UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
    
    //需创建一个包含待通知内容的 UNMutableNotificationContent 对象
    UNMutableNotificationContent* content = [[UNMutableNotificationContent alloc] init];
    content.body = pushContent;
    content.sound = [UNNotificationSound defaultSound];
    content.userInfo = dic;
    
    // 在 alertTime 后推送本地推送
    UNTimeIntervalNotificationTrigger* trigger = [UNTimeIntervalNotificationTrigger
                                                  triggerWithTimeInterval:5 repeats:NO];
    
    UNNotificationRequest* request = [UNNotificationRequest requestWithIdentifier:[dic objectForKey:@"pushmsgid"]
                                                                          content:content trigger:trigger];
    
    //添加推送成功后的处理！
    [center addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
    }];
     
	
## 三、处理通知       

### 1. ios8-10     

> 关闭进程后，点击通知打开应用         
     
	  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	        
	    if (launchOptions != nil) {
	        UILocalNotification *localNotification = [launchOptions objectForKey:UIApplicationLaunchOptionsLocalNotificationKey];   
	        NSDictionary *remoteNotification = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
	    }
	
	    return YES;
	 }	  
      
从启动参数中获取remote或者local notification信息，处理通知       
  
> 应用在前台或后台时，点击通知       
     
	  //local 
	  - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
	    KLMLog(@"received local Ntification:%@",notification.userInfo);
	    //to do when click local notification
	  }
	  
	  // remote   
	  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
	    KLMLog(@"receive remote Notification:%@",userInfo);
	    //to do
	  }
      
  
### 2. iOS10 later（remote/local）       
iOS 10之后的系统，处理通知时并不根据local和remote提供不同的处理函数，但是在方法中可以根据notification的trigger信息区分是远程或本地通知。

> 在应用内展示通知。

App 处于前台时捕捉并处理即将触发的推送,此处可以做通知展示之前的一些处理，例如修改接收到的push信息等    
   
	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler{
	// to do something   
	 
	//display notifiaction	   
	completionHandler(UNNotificationPresentationOptionAlert | UNNotificationPresentationOptionSound);
	    
	 } 
   
 
> 点击操作（关闭进程/前台/后台）

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler {
	    //收到推送的请求
	    UNNotificationRequest *request = response.notification.request;
	    if([response.notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
	        //判断为远程通知
	        KLMLog(@"handle remote notification");
	     }else {
	        // 判断为本地通知
	        KLMLog(@"handle local notification");
	    }
	    completionHandler();
	}
        
 
## 四、iOS 8处理remote通知时的两个方法     

对于iOS 8-10，处理远程通知时两个方法，下面介绍了两个方法实际使用的不同之处。    

### 方法一：   
    
	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
	    //NSLog(@"%@",userInfo);
	    
	 }
   
 
> 应用程序在前台时：通知到，该方法自动执行   
 应用程序在后台且没有退出时：通知到，只有点击了通知查看时，该方法自动执行    
 应用程序退出：通知到，点击查看通知，只执行didFinishLauncing方法，不会执行该方法。    

### 方法二：   
   
	- (void)application:(UIApplication *)application        didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
	    //NewData就是使用新的数据 更新界面，响应点击通知这个动作
	    completionHandler(UIBackgroundFetchResultNewData);
	}      
 
> 如果应用程序在前台，则通知到，该方法自动执行   
> 如果应用程序在后台，则通知到，点击查看，该方法自动执行      
如果应用程序被关闭，则通知到，点击查看，先执行didFinish方法，再执行该方法   
 
## 五、apns推送相关信息   
apns推送，会接收到remote notification。

> ios8-10   

1. 当程序未启动，用户接收到消息。需要在AppDelegate中的didFinishLaunchingWithOptions得到消息内容
2. 当程序在前台运行，接收到消息不会有消息提示（提示框或横幅）。
3. 当程序运行在后台，接收到消息会有消息提示，点击消息后进入程序，AppDelegate的didReceiveRemoteNotification函数会被调用，消息做为此函数的参数传入     

> ios10 later   

程序运行在前台或后台，接收到消息，UNUserNotificationCenterDelegate的方法  

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler{     
	     completionHandler(UNNotificationPresentationOptionAlert | UNNotificationPresentationOptionSound);
	
	    }   
	}
    

会被调用，此处可以对收到的消息处理，然后弹出remote notification，   
点击通知时，调用下面方法：    

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler {
	    //收到推送的请求
	    UNNotificationRequest *request = response.notification.request;
	    if([response.notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
	        //判断为远程通知
	        KLMLog(@"handle remote notification");
	    }
	
	    completionHandler();
	}
    

## 六、本地推送(ios8-10)    

**注意：**一个App最多只能设置64个本地推送，当超过此限制的时候，系统会自动忽略多余的本地推送，而保留能最快触发的64个。

处理本地通知时的回调方法:
   
 
### App在前台或后台时，接收和点击通知的处理
如果App处于Background状态时，只有用户点击了通知消息时才会调用该方法；如果App处于Foreground状态，接收到本地通知时会直接调用该方法。
所以在方法中要根据app当前的状态，来区分实际的操作是接收还是点击，作分别处理。

 ```
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
    //to do 
    
    if([UIApplication sharedApplication].applicationState==UIApplicationStateActive) {
        //前台运行时，收到推送的通知（未点击）,如何处理
        
    } else if ([UIApplication sharedApplication].applicationState==UIApplicationStateInactive) {
        //（应用程序在前台或后台）点击推送通知进入界面时        
    }
} ```
