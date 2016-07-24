title: 初识ReactiveCocoa    
date: 2016-03-17 23:12:45    
category: iOS     
tags:
---

[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 是一个Objective-C 框架，受 [Functional Reactive Programming](https://en.wikipedia.org/wiki/Functional_reactive_programming)的启发。它提供了一系列用来**组合和转换值流**的API。

如果你早已熟悉了函数响应式编程或者知道ReactiveCocoa的基本前提，看看[Documentation](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/v2.5/Documentation)这个文件夹里的framework overview等文件更深一步来了解它是怎样在实践中工作的。

## 介绍

ReactiveCocoa受[functional reactive programming](http://blog.maybeapps.com/post/42894317939/input-and-output)的启发。在那些能被替换和修改的地方，RAC提供信号(由`RACSignal`代表)来捕获当前和将来的值而不是使用可变的变量。

通过链接，组合，和反馈的信号，软件可以不需要写那些持续观察和更新value的代码。

例如，一个文本框能够根据它的改变被绑定到最后一次的值，而不是使用额外的代码每秒去监控时钟和更新文本框。这点跟KVO很像，不过是使用了block，而非`-observeValueForKeyPath:ofObject:change:context:`

信号也可以进行异步操作，就像[futures and promises](https://en.wikipedia.org/wiki/Futures_and_promises)。这极大的简化了异步软件中网络连接的代码。

RAC的重大优势之一就是它提供信号(`signal`)这种方式来统一的处理所有异步的行为，包括代理方法、block 回调、target-action 机制、通知和KVO。

![](http://7xn9bi.com1.z0.glb.clouddn.com/cheaptalk.png) 

这里是简单的例子：


```
// 当self.username改变时，打印新的名字到控制台
//
// RACObserve(self, username)创建一个新的RACSignal，当前self.username的值发生改变时，发送新值给newName
// -subscribeNext: 当信号发送值时将触发block
[RACObserve(self, username) subscribeNext:^(NSString *newName) {
	NSLog(@"%@", newName);
}];
```

与KVO 通知不同的是信号能够进行统一的链式操作：

```
// 只有当名字的开头为"j"时才打印
//
// -filter 只有当block返回YES时才会创建一个新的RACSignal发送一个新值
[[RACObserve(self, username)
	filter:^(NSString *newName) {
		return [newName hasPrefix:@"j"];
	}]
	subscribeNext:^(NSString *newName) {
		NSLog(@"%@", newName);
	}];
```

信号也能被用来派生状态。在响应新值中RAC代替观察属性和设置其他的属性，能够在信号和运行周期内传达属性：

```
// 当self.password 和 self.passwordConfirmation相同时创建一个单向的binding使得self.createEnabled为true
//
// RAC() 是一个宏指令使得binding看起来nicer
// 
// +combineLatest:reduce: 建一个信号数组
// 当任一个信号的最后一个值发生改变时触发这个block，返回一个新的RACSignal，将block返回的值作为values发送出去
RAC(self, createEnabled) = [RACSignal 
	combineLatest:@[ RACObserve(self, password), RACObserve(self, passwordConfirmation) ] 
	reduce:^(NSString *password, NSString *passwordConfirm) {
		return @([passwordConfirm isEqualToString:password]);
	}];
```

信号不仅是在KVO上，还能在建立在随着时间而改变的值流上。例如，它们可以代表按钮点击：

```
// 当按钮被点击时打印信息
//
// RACCommand创建信号去表示UI行为。例如，每一个信号可以表示一个按钮的点击、与它相关联的附加工作
//
// -rac_command是封装的NSButton方法. 当按钮被点击时触发该命令
self.button.rac_command = [[RACCommand alloc] initWithSignalBlock:^(id _) {
	NSLog(@"button was pressed!");
	return [RACSignal empty];
}];
```

或者是异步网络操作：

```
// 连接"Log in"按钮给网络登录
//
// 当登录命令执行时运行block，开始登录进度
self.loginCommand = [[RACCommand alloc] initWithSignalBlock:^(id sender) {
	// 假设当网络请求完成时 -logIn 方法返回一个信号发送一个value
	return [client logIn];
}];

// -executionSignals 每次执行该命令时，这个方法返回一个信号，包括以前的block返回的信号
[self.loginCommand.executionSignals subscribeNext:^(RACSignal *loginSignal) {
	// 成功登录时打印信息
	[loginSignal subscribeCompleted:^{
		NSLog(@"Logged in successfully!");
	}];
}];

// 按钮被点击时执行登录命令
self.loginButton.rac_command = self.loginCommand;
```

信号也可以代表定时器，其他的UI事件，或者别的什么随时间而改变的事件。

在异步操作方面，通过链接和转换信号可以建立更复杂的行为。在一组完整的操作之后更简单的来执行工作：

```
// 执行2个网络操作，当它们都完成时打印信息到控制台
//
// +merge: 当数组里的所有信号完成时，返回一个新的RACSignal
//
// -subscribeCompleted: 当信号完成时将执行这个block
[[RACSignal 
	merge:@[ [client fetchUserRepos], [client fetchOrgRepos] ]] 
	subscribeCompleted:^{
		NSLog(@"They're both done!");
	}];
```

信号可以被链接到顺序执行异步操作，而不是使用一堆block回调。通常这样简单的来使用[futures and promises](https://en.wikipedia.org/wiki/Futures_and_promises)：

```
// 用户登录，下载缓存信息，获取服务器信息。都完成后将信息打印到控制台
//
// 假设登录之后 -logInUser 方法返回一个信号
//
// -flattenMap: 当信号发送一个value时触发这个block
// 并且返回一个新的RACSignal来整合从block返回的所有的信号到一个单一信号中
[[[[client 
	logInUser] 
	flattenMap:^(User *user) {
		// 下载缓存信息，给用户返回一个信号
		return [client loadCachedMessagesForUser:user];
	}]
	flattenMap:^(NSArray *messages) {
		// Return a signal that fetches any remaining messages.
		return [client fetchMessagesAfterMessage:messages.lastObject];
	}]
	subscribeNext:^(NSArray *newMessages) {
		NSLog(@"New messages: %@", newMessages);
	} completed:^{
		NSLog(@"Fetched all messages.");
	}];
```

RAC甚至可以简单的建立在一个异步操作的结果上：

```
// 创建一个单向的binding，让 self.imageView.image 来放置下载下来的user的头像
//
// 假设 -fetchUserWithUsername: 方法返回一个信号发送给user
//
// -deliverOn: 创建新的信号在其他的队列中进行他们的工作
// 在这个例子中，此方法被用来将工作转移到后台队列和回到主线程
//
// -map: 每个user调用这个block，获取并且返回一个新的RACSignal，并且将从block返回的值发送出去
RAC(self.imageView, image) = [[[[client 
	fetchUserWithUsername:@"joshaber"]
	deliverOn:[RACScheduler scheduler]]
	map:^(User *user) {
		// 下载头像 (在后台队列中进行).
		return [[NSImage alloc] initWithContentsOfURL:user.avatarURL];
	}]
	// 此时这个任务将在主线程中执行
	deliverOn:RACScheduler.mainThreadScheduler];
```
这是一些使用RAC的示范操作，但是它并不能说明RAC为什么如此强大。
更多示例代码参见[C-41](https://github.com/AshFurrow/C-41) 或 [GroceryList](https://github.com/jspahrsummers/GroceryList),这些是使用ReactiveCocoa编写的iOS APP。在这个文件夹[Documentation](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/v2.5/Documentation)中可以查到更多的关于RAC的信息。

## 使用ReactiveCocoa

乍一看ReactiveCocoa是非常抽象的，很难理解该怎样将它应用到具体的问题上。

这有一些示例来展示RAC的优势

### 处理异步或事件驱动的数据源

许多Cocoa编程的重点是对用户事件的反应或应用状态的变化。处理这些事件的代码很快变得非常复杂的就像意大利面一样，伴随着许多回调函数和状态变量处理顺序的问题。

表面上看起来模式不同，比如UI回调，网络响应和KVO通知，实际上有很多共同之处。[RACSignal](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v2.5/ReactiveCocoa/RACSignal.h)统一了所有的这些不同的API，使他们可以组合在一起，并以同样的方式操纵。

例如这样的代码：

```
static void *ObservationContext = &ObservationContext;

- (void)viewDidLoad {
	[super viewDidLoad];

	[LoginManager.sharedManager addObserver:self forKeyPath:@"loggingIn" options:NSKeyValueObservingOptionInitial context:&ObservationContext];
	[NSNotificationCenter.defaultCenter addObserver:self selector:@selector(loggedOut:) name:UserDidLogOutNotification object:LoginManager.sharedManager];

	[self.usernameTextField addTarget:self action:@selector(updateLogInButton) forControlEvents:UIControlEventEditingChanged];
	[self.passwordTextField addTarget:self action:@selector(updateLogInButton) forControlEvents:UIControlEventEditingChanged];
	[self.logInButton addTarget:self action:@selector(logInPressed:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)dealloc {
	[LoginManager.sharedManager removeObserver:self forKeyPath:@"loggingIn" context:ObservationContext];
	[NSNotificationCenter.defaultCenter removeObserver:self];
}

- (void)updateLogInButton {
	BOOL textFieldsNonEmpty = self.usernameTextField.text.length > 0 && self.passwordTextField.text.length > 0;
	BOOL readyToLogIn = !LoginManager.sharedManager.isLoggingIn && !self.loggedIn;
	self.logInButton.enabled = textFieldsNonEmpty && readyToLogIn;
}

- (IBAction)logInPressed:(UIButton *)sender {
	[[LoginManager sharedManager]
		logInWithUsername:self.usernameTextField.text
		password:self.passwordTextField.text
		success:^{
			self.loggedIn = YES;
		} failure:^(NSError *error) {
			[self presentError:error];
		}];
}

- (void)loggedOut:(NSNotification *)notification {
	self.loggedIn = NO;
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
	if (context == ObservationContext) {
		[self updateLogInButton];
	} else {
		[super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
	}
}
```

… 可以用RAC这样的表示：

```
- (void)viewDidLoad {
	[super viewDidLoad];

	@weakify(self);

	RAC(self.logInButton, enabled) = [RACSignal
		combineLatest:@[
			self.usernameTextField.rac_textSignal,
			self.passwordTextField.rac_textSignal,
			RACObserve(LoginManager.sharedManager, loggingIn),
			RACObserve(self, loggedIn)
		] reduce:^(NSString *username, NSString *password, NSNumber *loggingIn, NSNumber *loggedIn) {
			return @(username.length > 0 && password.length > 0 && !loggingIn.boolValue && !loggedIn.boolValue);
		}];

	[[self.logInButton rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *sender) {
		@strongify(self);

		RACSignal *loginSignal = [LoginManager.sharedManager
			logInWithUsername:self.usernameTextField.text
			password:self.passwordTextField.text];

			[loginSignal subscribeError:^(NSError *error) {
				@strongify(self);
				[self presentError:error];
			} completed:^{
				@strongify(self);
				self.loggedIn = YES;
			}];
	}];

	RAC(self, loggedIn) = [[NSNotificationCenter.defaultCenter
		rac_addObserverForName:UserDidLogOutNotification object:nil]
		mapReplace:@NO];
}
```

### 链接依赖操作

依赖在网络请求中是常见的，在下一个请求建立之前，需要完成当前对服务器的请求，比如：

```
[client logInWithSuccess:^{
	[client loadCachedMessagesWithSuccess:^(NSArray *messages) {
		[client fetchMessagesAfterMessage:messages.lastObject success:^(NSArray *nextMessages) {
			NSLog(@"Fetched all messages.");
		} failure:^(NSError *error) {
			[self presentError:error];
		}];
	} failure:^(NSError *error) {
		[self presentError:error];
	}];
} failure:^(NSError *error) {
	[self presentError:error];
}];
```

在ReactiveCocoa中可以这样简单的实现：

```
[[[[client logIn]
	then:^{
		return [client loadCachedMessages];
	}]
	flattenMap:^(NSArray *messages) {
		return [client fetchMessagesAfterMessage:messages.lastObject];
	}]
	subscribeError:^(NSError *error) {
		[self presentError:error];
	} completed:^{
		NSLog(@"Fetched all messages.");
	}];
```

### 并行独立工作

与独立的数据集合并行工作，然后将它们合并成一个non-trivial函数到Cocoa，并经常涉及大量的同步：

```
__block NSArray *databaseObjects;
__block NSArray *fileContents;
 
NSOperationQueue *backgroundQueue = [[NSOperationQueue alloc] init];
NSBlockOperation *databaseOperation = [NSBlockOperation blockOperationWithBlock:^{
	databaseObjects = [databaseClient fetchObjectsMatchingPredicate:predicate];
}];

NSBlockOperation *filesOperation = [NSBlockOperation blockOperationWithBlock:^{
	NSMutableArray *filesInProgress = [NSMutableArray array];
	for (NSString *path in files) {
		[filesInProgress addObject:[NSData dataWithContentsOfFile:path]];
	}

	fileContents = [filesInProgress copy];
}];
 
NSBlockOperation *finishOperation = [NSBlockOperation blockOperationWithBlock:^{
	[self finishProcessingDatabaseObjects:databaseObjects fileContents:fileContents];
	NSLog(@"Done processing");
}];
 
[finishOperation addDependency:databaseOperation];
[finishOperation addDependency:filesOperation];
[backgroundQueue addOperation:databaseOperation];
[backgroundQueue addOperation:filesOperation];
[backgroundQueue addOperation:finishOperation];
```

上面的代码可以用简单的合成信号来清理和优化：

```
RACSignal *databaseSignal = [[databaseClient
	fetchObjectsMatchingPredicate:predicate]
	subscribeOn:[RACScheduler scheduler]];

RACSignal *fileSignal = [RACSignal startEagerlyWithScheduler:[RACScheduler scheduler] block:^(id<RACSubscriber> subscriber) {
	NSMutableArray *filesInProgress = [NSMutableArray array];
	for (NSString *path in files) {
		[filesInProgress addObject:[NSData dataWithContentsOfFile:path]];
	}

	[subscriber sendNext:[filesInProgress copy]];
	[subscriber sendCompleted];
}];

[[RACSignal
	combineLatest:@[ databaseSignal, fileSignal ]
	reduce:^ id (NSArray *databaseObjects, NSArray *fileContents) {
		[self finishProcessingDatabaseObjects:databaseObjects fileContents:fileContents];
		return nil;
	}]
	subscribeCompleted:^{
		NSLog(@"Done processing");
	}];
```

### 简化collection转换

高阶函数比如 `map`, `filter`, `fold`/`reduce`在Foundation中是非常缺少的，导致循环中的代码像这样：

```
NSMutableArray *results = [NSMutableArray array];
for (NSString *str in strings) {
	if (str.length < 2) {
		continue;
	}

	NSString *newString = [str stringByAppendingString:@"foobar"];
	[results addObject:newString];
}
```

[RACSequence](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v2.5/ReactiveCocoa/RACSequence.h)允许所有Cocoa collection在统一的和声明的方式下被操作：

```
RACSequence *results = [[strings.rac_sequence
	filter:^ BOOL (NSString *str) {
		return str.length >= 2;
	}]
	map:^(NSString *str) {
		return [str stringByAppendingString:@"foobar"];
	}];
```

## 后记
* 以上文章摘译自ReactiveCocoa的Objective-C官方文档[ReactiveCocoa Documentation](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v2.5/README.md#when-to-use-reactivecocoa)

* 以上内容介绍了RAC的基本用法，仅限于使用，所以墙裂建议仔细学习下节的**参考链接**，了解RAC原理及高阶用法。

* 小白出手，请多指教。如言有误，还望斧正！

* 转载请保留原文地址[http://gonghonglou.com/2016/03/17/MeetingReactiveCocoa](http://gonghonglou.com/2016/03/17/MeetingReactiveCocoa)

## 参考链接
* ReactiveCocoa的Objective-C官方文档[ReactiveCocoa Documentation](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v2.5/README.md#when-to-use-reactivecocoa)
* 雷纯锋的[ReactiveCocoa v2.5 源码解析之架构总览](http://blog.leichunfeng.com/blog/2015/12/25/reactivecocoa-v2-dot-5-yuan-ma-jie-xi-zhi-jia-gou-zong-lan/)
* 吖了个峥的[最快让你上手ReactiveCocoa之基础篇](http://www.jianshu.com/p/87ef6720a096) 和 [最快让你上手ReactiveCocoa之进阶篇](http://www.jianshu.com/p/e10e5ca413b7)
* 李忠的[ReactiveCocoa与Functional Reactive Programming](http://limboy.me/ios/2013/06/19/frp-reactivecocoa.html)
* 唐巧的[ReactiveCocoa 讨论会](http://devtang.com/2016/01/03/reactive-cocoa-discussion/)
