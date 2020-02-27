---
title: Notification的封装
categories: [iOS]
tags: [note]
seo:
  date_modified: 2020-02-28 02:57:00 +0800
---

### 新项目采用了MVC架构, 在公司以往的几个项目采用的MVVM/MVVM-C架构下, 一些全局的数据同步都是使用的ReactiveCocoa/RxSwift中的可监听序列, 当然, 也不是说MVC用不了Functional reactive programming框架, 只是当前项目决定不用

### 鉴于此背景下, 新项目中全局的数据同步尝试使用系统的Notification, 有以下几个问题
* 对于Notification.Name, 有时根据业务需要自定义通知, 此时系统提供的无法满足需求
* 参数问题, 即Notification.userInfo, 如果想给通知追加参数, 系统是以字典的方式提供的, 这种方式不便于对发送和接收进行维护, 若是参数需要更改, 很有可能会漏掉某些地方, 而且编译器也是不会给你提供警告帮助的, 还有可能发送的参数改掉之后, 接收通知就无法正确解析了
* 在某个类中添加监听通知时, 需要在此类的的析构函数中去除此实例类, 这一点是非常容易忽略掉的

### 针对上面的问题, 分别进行一下封装
#### Notificaiton.Name
在Objective-C中, 是使用全局常量的方式构造一个通知名称对象, Swift提供了Extension的方式让用户进行自定义

```
extension Notification.Name {    
    static let CustomNotificationName: Notification.Name = Notification.Name(rawValue: "CustomNotificationNameRawValue")
}

// usage
NotificationCenter.default.addObserver(forName: .CustomNotificationName, object: nil, queue: nil, using: { _ in })
```
这样使用起来便方便了许多, 但是会有和系统通知名称重复的可能, 若是极致优化, 可以参考[大神的建议](https://www.jianshu.com/p/105f6b133bd2)

#### 参数的封装
参考网络请求中, 对返回数据的反序列化, 我们可以创建自己的通知对象

```
struct CustomNotificationn {
    let frame: CGRect
    let duration: Double
}

extension CustomNotificationn {
    init(note: Notification) {
        frame = note.userInfo?["CustomNotification.Frame"] as? CGRect ?? .zero
        duration = note.userInfo?["CustomNotification.Duration"] as? Double ?? .zero
    }
    
    func info() -> [AnyHashable : Any] {
        return ["CustomNotification.Frame" : frame,
                "CustomNotification.Duration" : duration]
    }
}
```
数据的封装有了, 但是怎么使用呢, 直接的做法就是在监听回调中直接初始化实例对象然后使用, 但是还可以进一步封装, 让NotificationCenter直接发送和监听自定义通知对象, 达到使用者无感的地步

* 定义一个通知协议

	```
	protocol CustomNotificationProtocol {
	    var name: Notification.Name { get }
	    var objc: Any? { get }
	    var userInfo: [AnyHashable : Any]? { get }
	}
	
	extension CustomNotificationProtocol {
	    var objc: Any? { nil }
	    var userInfo: [AnyHashable : Any]? { nil }
	}
	```
* 让自定义通知对象遵守此协议

	```
	extension CustomNotificationn: CustomNotificationProtocol {
	    var name: Notification.Name { .CustomNotificationName }
	    
	    var userInfo: [AnyHashable : Any]? {
	        ["CustomNotification.Frame" : frame,
	         "CustomNotification.Duration" : duration]
	    }
	}
	```

* 定义一个NotificationDescriptor用户桥接

	```
	struct NotificationDescriptor<A> {
	    let name: Notification.Name
	    let convert: (Notification) -> A
	}
	
	// usage
	let customNotificationDescriptor = NotificationDescriptor(name: .CustomNotificationName, convert: CustomNotificationn.init)
	```
* 最后, 给NotificationCenter增加一个扩展, 这样, 就可以直接发送自定义通知对象, 并且监听时直接读取属性的方式拿到参数了

	```
	extension NotificationCenter {
	    @discardableResult
	    func addObserver<A>(forDescriptor d: NotificationDescriptor<A>,
	                        object obj: Any,
	                        queue: OperationQueue?,
	                        using block: @escaping (A) -> ()) -> NSObjectProtocol {
	        return addObserver(forName: d.name, object: nil, queue: nil, using: { note in
	            block(d.convert(note))
	        })
	    }
	    
	    func post(note: CustomNotificationProtocol) {
	        post(name: note.name, object: note.objc, userInfo: note.userInfo)
	    }
	}
	```
* 使用

	```
	let customNotificationDescriptor = NotificationDescriptor(name: .CustomNotificationName, convert: CustomNotificationn.init)
	
	NotificationCenter.default.addObserver(forDescriptor: customNotificationDescriptor) { descriptor in
	    print(descriptor.frame, descriptor.duration)
	}
	
	NotificationCenter.default.post(note: CustomNotificationn(frame: .zero, duration: .zero))
	```

#### 监听者的移除
创建一个包含通知中心和监听类的类别, 使用deinit将观察者的生命周期与对象的生命周期联系起来

```
class Token {
    let center: NotificationCenter
    let observer: NSObjectProtocol
    
    init(observer: NSObjectProtocol, center: NotificationCenter = .default) {
        self.observer = observer
        self.center = center
    }
    
    deinit {
        center.removeObserver(observer)
    }
}
```
优化NotificationCenter的扩展函数

```
extension NotificationCenter {
    func addObserver<A>(forDescriptor d: NotificationDescriptor<A>,
                        object obj: Any? = nil,
                        queue: OperationQueue? = nil,
                        using block: @escaping (A) -> ()) -> Token? {
        let observer = addObserver(forName: d.name, object: nil, queue: nil, using: { note in
            block(d.convert(note))
        })
        return Token(observer: observer, center: self)
    }
}
```
在启用监听的类中, 生命一个可选值的属性用来保存Token, 这样, 在这个类即将析构时, 也会调用token属性的deinit函数, 触发移除监听操作, 这种方式被称之为[获取资源即初始化](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
虽然这种方式要求使用者要额外创建一个属性, 但是避免了使用者忘记移除监听者, 因为如果不接受addObserver返回值的话, 编译器会报警告来提示开发者
如果想提前结束监听, 也可以手动置nil来移除监听

```
var token: Token?

token = NotificationCenter.default.addObserver(forDescriptor: customNotificationDescriptor) { descriptor in
    print(descriptor.frame, descriptor.duration)
}

token = nil
```

### 参考文章
[Swift Talk episode 27: Typed Notifications](https://talk.objc.io/episodes/S01E27-typed-notifications-part-1)