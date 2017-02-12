"=========== Meta ============
"StrID : 21
"Title : 使用NSProxy实现对象的动态代理
"Slug  : ios-dynamic-proxy
"Cats  : 移动端
"Tags  : 
"Date  : 20151118T03:47:25
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
动态代理可以在运行时期为协议（*protocol*）动态生成实现（implementations），请注意，这个很重要，是在运行时期动态为一个方法生成实现。使用动态代理可以为一系列功能相同的方法提供统一的实现，而不需要为每个接口实现相同的逻辑。

<!--more-->

一般情况下，如果一个对象收到了不能识别的 *selector* 调用，OC 的运行时会抛出这样的异常：

> 2015-03-31 14:27:05.313 MyApp[58439:6819367] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[SCAppDelegate someMethod]: unrecognized selector sent to instance 0x7ff44af1de50'

基于`NSProxy`创建的代理对象，如果收到未实现方法的调用，代理对象会获得处理这个消息的机会（`forwardInvocation:`），它可以获取到方法的返回类型、方法名以及传入的参数值，代理对象再对这些信息进行加工处理，达到动态实现这个方法的目的。

`NSProxy`是一个抽象类，它没有提供初始化方法，并且接收到任何未被处理的消息时会抛出一个异常。所以`NSProxy`的子类必须要实现初始化方法，以及`forwardInvocation:`和`methodSignatureForSelector:`方法来处理自身未实现的消息。`forwardInvocation:`用来处理消息（例如可以通过网络将调用转发至服务器），最让人蛋疼的
就这个`methodSignatureForSelector:`方法，看看它的定义：

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
```

它要求返回一个方法的签名，也就是方法的完整定义，包括调用方法的参数信息，但是大多数情况下，在代理类的实现中并不知道在运行时期要处理什么样的消息，每个方法有几个参数，每个参数的类型分别是什么，如果这些都已知的话，也就不是动态代理了。好在我们还有一些技巧绕过这个限制，在后面可以看到。

说一点点题外话，在 Java 里也有动态代理，但很明显要比 OC 优雅的多：

```objc
public Object invoke( Object proxy, Method method, Object[] args ) throws Throwable {
	return method.invoke( proxied, args);  
}  
```

回归正题，在网络模块的设计中，使用动态代理可以对外提供定义良好的强类型的接口。对一个远程服务的请求都有以下通用的处理逻辑：

1. 将请求的服务名及参数以特定格式组合成数据包；
2. 通过网络把数据包发送给远程服务器；
3. 接收服务器返回的数据并解析；

而对于一个方法的定义，又包含以下三个信息：

1. 方法名
2. 参数列表
3. 返回值类型

方法名可以定义为远程服务的名字，返回值类型可以让网络模块知道要将服务端返回的数据解析成何种类型的对象；知道方法的这三个信息，代理对象就可以完成一个远程服务的请求。

例如对于登录的网络服务，可以这样定义接口：

```objc
@protocol TCAccountService <NSObject>
- (TCAccountInfo *)loginWithName:(NSString *)name password:(NSString *)password;
@end
```
这仅仅是一个 *protocol* 的定义，不提供任何实现，但是我们可以通过下面的方法来调用它：

```objc
- (void)doLogin {
    id<TCAccountService> as = [context proxyObject];
    self.accountInfo = [as loginWithId:@"admin" password:@"123"];
}
```

上面的`proxyObject`方法返回一个**代理对象**：

```objc
- (NSProxy *)proxyObject {
	return [[TCRpcProxy alloc] init];
}
```

这个代码对象用于处理`loginWithId:password:`方法的调用。代理类必须要继承自`NSProxy`并提供初始化方法，所以会有如下有声明：

```objc
@interface TCRpcProxy : NSProxy
- (instancetype)init;
@end
```

在`TCRpcProxy`的定义中，必须实现`forwardInvocation:`和`methodSignatureForSelector:`来处理未实现消息：

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    static dispatch_once_t once;
    static NSMethodSignature* ms = nil;
    dispatch_once(&once, ^{
        ms = [super methodSignatureForSelector:@selector(__mockSelector:b:c:d:e:f:g:h:i:j:k:l:m:)];
    });
    return ms;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
	// 获取 selector 的名字（如前面的 loginWithId:password:）
	NSString *selName = [NSString stringWithUTF8String:sel_getName(anInvocation.selector)];
	// 解析 selName 中冒号的个数，即是参数的个数。
	NSInteger argc = [self numberOfArgumentsOfSelector:selName];
	for (NSInteger i = 0; i < argc; ++i) {
		// 读取第i个参数，之所以 i+2 的原因是前两个参数分别对应 self 和 selector。
		_unsafe_unretained id obj = nil;
		[anInvocation getArgument:&obj atIndex:i + 2];
	}
}
```

在`methodSignatureForSelector:`的实现中，返回了一个**魔术方法**的签名，也就是`__magicSelector:b:c:d:e:f:g:h:i:j:k:l:m:`的签名，这是在`TCRpcProxy`中定义的一个私有方法。因为在代理类的实现中，对运行时期调用的方法是未知的（有多少个参数，每个参数的类型分别是什么），所以这个**魔术方法**必须是一个很通用的方法，只有这样才能处理运行时期所有方法的调用。**魔术方法**的定义，取决于以下几个实事：

- 魔术方法的作用仅仅是为了在`methodSignatureForSelector:`中返回一个方法的签名，不需要任何实现；
- 魔术方法的参数要尽可能的多，要多于在运行时期实际调用的方法的参数个数，否则在`forwardInvocation:`中会取不到所有的参数值；
- 魔术方法所接收的所有参数只能是`id`类型的，这也限制了在运行时期调用的方法的参数也必须是`id`类型；

下面这个方法的定义在我们的实践中，基本能处理到所有方法调用的情况：

```objc
- (id)__magicSelector:(id)a b:(id)b c:(id)c d:(id)d e:(id)e f:(id)f g:(id)g h:(id)h i:(id)i j:(id)j k:(id)k l:(id)l m:(id)m {
    return nil;
}
```

