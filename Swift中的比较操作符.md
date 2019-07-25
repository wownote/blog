"=========== Meta ============
"StrID : 41
"Title : Swift中的比较运算符
"Slug  : swift-comparable
"Cats  : 技术
"Tags  : Comparable, Swift
"Date  : 20170214T07:12:34
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
本文通过两种常用的情景：数组的排序和查找，来演示Swift中比较运算符的用法。实现了`Comparable`协议的类型，其对象可以像数字一样比较大小。为自定义类型实现`Compatable`协议，只需为你的类型定义 &lt; 和 == 两个静态方法，基于这两个基本的运算符，Swift提供了剩余4个比较运算符的默认实现。

<!--more-->

假设我们定义了一个名为Employee的类来描述每个雇员：

```swfit
struct Employee {
	var firstName: String = ""
	var lastName: String = ""
}
```

我们可以使用默认的初始化方法创建Employee对象：

```swift
let eric = Employee(firstName: "王", lastName: "一");
let john = Employee(firstName: "王", lastName: "二");
let sam = Employee(firstName: "张", lastName: "三");
```

这些对象可能会存储在一个数组中（也可能是Dictionary或Set）：

```swift
let team = [eric, john, sam]
```

那么接下来，当我们想知道某个雇员是否是这个Team中的一员时，该怎么做？可以使用`Array`提供的`contains(where:)`方法，这个方法接收一个**predicate closure**参数，这个参数期望返回一个布尔值来指示两个对象是否相等。所以为了测试某个成员是否在数组中，可以这样写：

```swift
let exists = team.contains { (em) -> Bool in
	return em.firstName == "王" &&
	em.lastName == "二";
}
```

上面的代码，问题在于，当你需要很多次查找时，每个调用`contains(where:)`方法的地方，都要写这些繁琐的比较代码。在实际的项目中，需要判断的字段可能更多，判断的逻辑可能更复杂，你就不得不写更多类似的重复代码，在不同调用的地方复制、粘贴，这显然不是一个好的做法。

Swift标准库提供了一个`Comparable`协议，来对两个对象进行比较，我们可以采用`Comparable`来重新设计前面的示例代码：

```swift
extension Employee : Comparable {
	static func == (lhs: Employee, rhs: Employee) -> Bool {
		return lhs.firstName == rhs.firstName &&
		lhs.lastName == rhs.lastName;
	}
}
```

上面代码中`==`操作符返回一个布尔值来指示两个Employee类型的参数是否相等。这个相等比较的代码，可以在任何需要判断Employee对象是否相等的地方复用。实现了`Comparable`协议后，我们可以使用一个更简单直观的`contains`方法来判断某个成员是否在数组中：

```swift
team.contains(Employee(firstName: "王", lastName: "二"))
```

那么，如果我们要对这个成员数组进行排序时，又该怎么做？排序就需要对两个值进行比较，`Array`类提供了`sorted(by:)`方法来返回一个排序后的数组，这个方法也接收一个闭包参数，来指示第一个参数是否小于第二个参数：

```swift
let sortedTeam = team.sorted(by: { $0.firstName < $1.firstName })
```

如果采用`Comparable`可以更优雅的实现排序：

```swift
extension Employee : Comparable {
	static func == (lhs: Employee, rhs: Employee) -> Bool {
		return lhs.firstName == rhs.firstName &&
		lhs.lastName == rhs.lastName;
	}
	static func < (lhs: Employee, rhs: Employee) -> Bool {
		return lhs.firstName < rhs.firstName ||
		(lhs.firstName == rhs.firstName && lhs.lastName < rhs.lastName);
	}
}
```

给自定义对象增加可比较能力，只需要实现**等于操作符**（==）和**小于操作符**（&lt;），Swift标准库基于这两个运算符提供了其它运算符（!=, >, >= 和 <=）的默认实现。现在Employee的对象已经有了可比较的特性，不再需要在排序时传入闭包：

```swift
let sortedTeam = team.sorted()
```

以上实际上就是面向对象设计中的封装，把对象比较的代码封装在内部，使对象具备**可比较**的能力，则任意容器（Array、Dictionary、Set）中的查找、排序等操作，都可以复用对象比较的代码。
