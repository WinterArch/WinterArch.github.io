---
date: '2025-10-21T15:43:57+08:00'
draft: true
title: '基于MVVM模式手把手写出可测试性强的SwiftUI代码'
---
# 用SwiftUI框架绘制视图
首先在Xcode里新建一个Swift Package包，要写UI库选择Library，git版本管理可以不勾，带SwiftTesting确保在Swift6的较新环境。
项目结构：
```
ImmutableState
├── Package.swift
├── Sources
│   └── ImmutableState
│       └── ImmutableState.swift
└── Tests
    └── ImmutableStateTests
        └── ImmutableStateTests.swift
```
Package.swift（模板生成）：
```Swift
let package = Package(
    name: "ImmutableState",
    products: [
        // Products define the executables and libraries a package produces, making them visible to other packages.
        .library(
            name: "ImmutableState",
            targets: ["ImmutableState"]),
    ],
    targets: [
        // Targets are the basic building blocks of a package, defining a module or a test suite.
        // Targets can depend on other targets in this package and products from dependencies.
        .target(
            name: "ImmutableState"),
        .testTarget(
            name: "ImmutableStateTests",
            dependencies: ["ImmutableState"]
        ),
    ]
)
```
上方展示了创建时的工程结构和模板生成Package.swift。
其次在ImmutableState.swift 旁边创建 ImmutableDemo 文件夹安放接下来的整套代码，也就在ImmutableDemo文件夹下使用 SwiftUIView 模板创建代码，然后右键struct的名称，refactor重构rename改名为 ImmutableView。
```Swift
import SwiftUI

struct ImmutableView: View {
    var body: some View {
        Text("Hello, World!")
    }
}

#Preview {
    ImmutableView()
}
```
此时报错提到“'View' is only available in iOS 13.0 or newer”等字样，说明库包模板生成的初始代码有可能和SwiftUI所要求的不符，来想办法修复它。
不了解Package.swift的朋友可能想问类似“如何在Swift Package中支持iOS等问题”。
最后通过阅读各种形式的文档搞清楚Package.swift的配置格式后，了解到解决办法是在name和products之间增加一行配置：
`platforms: [.iOS(.v13)],`
这个字段指定整个包支持的版本，代码中指定了iOS13，以后根据报错要求，不妨逐渐提升。
ImmutableView.swift标签页右方尽头，在倒数第2个按钮的多条横线样式的按钮上单击，在弹出的菜单里把Canvas预览画板点上刷新，已经可以看到第一行SwiftUI代码了。
这些UI代码写到库包里，不仅可以把工程结构和依赖关系撘得更清晰，还可以在换地方搬砖时复用（小心违法）。

# SwiftUI绘制原理
所有人都在让SwiftUI新手写下第一个`@State`属性：
```Swift
struct ImmutableView: View {
    @State var counter = 0
    
    var body: some View {
        Button(action: {
            
        }, label: {
            Text("Hello, World!\(counter)")
        })
    }
}
```
counter代表计数器。
预览画板中将会出现蓝色样式的按钮，按时是灰色。鼠标放到`@State`上按Option键单击，查看快速帮助（也可以在右上角打开右侧边栏点问号那个按钮，或把光标移动到位之后按下Ctrl+Command+?）。
学过Swift的朋友一定知道属性包装器和协议，这种@xx字样的不是attribute特性，就是属性包装器。还是鼠标放到`@State`上按 Command 键单击，进去看看，果然有@propertyWrapper，还发现它是遵循了`DynamicProperty`协议，再点进这个协议，可以看到它要求名为update的可变异方法。
这个方法会在`@State`包装的属性发生变化时，要求`View`重新绘制。
有人说Swift语言为编程引入了一种面向协议的范式。
同样的道理，出来到 ImmutableView.swift 标签页，点击 ImmutableView: View 的`View`进去看看，可以看到`View`也是协议，要求实现body属性，且需要是另一个遵循View协议的类型。
甚至能以一种不推荐的方式改写而不报错：
```Swift
struct ImmutableView {
    @State var counter = 0
}

extension ImmutableView: View {
    var body: some View {
        Button(action: {
            
        }, label: {
            Text("Hello, World!\(counter)")
        })
    }
}
```
这只是展示，还是Command+Z撤销还原回去。为了让counter计数器生效，可以在action闭包中间写入
`counter = counter + 1`
这行代码给counter赋值为比原值大1的值。
把预览画板作为UI测试器，点击按钮，计数器的结果已经如预期增加了。
再写一个一次增加5点计数的按钮：
```Swift
struct ImmutableView: View {
    @State var counter = 0
    
    var body: some View {
        VStack {
            Text("counter: \(counter)")
            Button(action: {
                counter = counter + 1
            }, label: {
                Text("+1!")
            })
            Button(action: {
                counter = counter + 5
            }, label: {
                Text("+5!")
            })
        }
    }
}
```
可以发现写了很多相似的逻辑代码，而且分散在视图之间，编程里的重复总是不太好，可以这么写：
```Swift
struct ImmutableView: View {
    @State var counter = 0
    
    var body: some View {
        VStack {
            Text("counter: \(counter)")
            Button(action: tapButtonPlus1, label: {
                Text("+1!")
            })
            Button(action: tapButtonPlus5, label: {
                Text("+5!")
            })
        }
    }
    
    func tapButtonPlus1() {
        plus(with: 1)
    }
    
    func tapButtonPlus5() {
        plus(with: 5)
    }
    
    func plus(with number: Int) {
        counter = counter + number
        print(counter)
    }
}
```
这次改动把按钮们的action放到方法内，集中在同一块区域写出，并且把相似的逻辑整合到一个方法里。
可以想象到随着视图逐渐变大，在涉及执行未必成功的代码时，还要写相应的异常处理、界面提示，方法区会随着这些的增长大到一定地步。
下面的测试写在/Tests/ImmutableStateTests/ImmutableStateTests.swift中，分别+1和+5之后，counter不等于预期的结果6：
```Swift
@MainActor // ImmutableView要求@MainActor或async/await，这行把测试方法放在主线程的行为体中运行，避免引入await
@Test func example() async throws {
    let view = ImmutableView()
    
    view.tapButtonPlus1()
    view.tapButtonPlus5()
    
    assert(view.counter == 6) // Thread 1: Assertion failed
}
```
没有人乐意每次都手动点开一个路由很深的页面，甚至维护在XCTest中录下自动打开的过程也会在工程长大之后变得繁琐。
我们需要另外一个语义来让视图与逻辑清晰地区分开，尤其要便于测试。

# M-V-VM模式
SwiftUI原生的模式是M-V模式，这种模式把逻辑集成在视图里，数据流向是Model <-> View，测试起来不太方便，极其依赖预览画板的展示，好处是在写一些小视图时表达性很强。
本文推荐的模式是通过苹果官方支持的Combine响应式框架实现的M-V-VM模式，这种模式把状态和逻辑摘到类中，可以写出不用预览画板就能在单元测试中使用的代码，视图规模但凡稍微上去一点就建议这么写，数据流向是Model <-> ViewModel <-> View。
其中，Model表示结构与逻辑，View表示视图与动作源，ViewModel表示动作与响应源。
工程中只有合适与不合适之分，没有绝对的好坏。
从`import SwiftUI`点进去，可以看到SwiftUI自身也有`import Combine`，它背后做了一些事，因此有SwiftUI的地方绝大部分时候不需要导入Combine。

# 用Combine框架发布视图状态更新
简单介绍一下要用到的Combine框架的属性包装器。
`@Published`可以理解为是在可观察类中的`@State`。
`ObservableObject`可观察类是一个能在观察到`@Published`成员属性的值发生变化时发出通知的类协议。
`@StateObject`是供可观察类使用的属性包装器，接到存放的可观察类的变化通知时会调用update方法使`View`更新。@StateObject要求iOS14。
试试把counter和那些方法搬到新的可观察类ImmutableViewModel里，然后用`@StateObject`存放类实例viewModel：
```Swift
struct ImmutableView: View {
    @StateObject var viewModel = ImmutableViewModel()
    
    var body: some View {
        VStack {
            Text("counter: \(viewModel.counter)")
            Button(action: viewModel.tapButtonPlus1, label: {
                Text("+1!")
            })
            Button(action: viewModel.tapButtonPlus5, label: {
                Text("+5!")
            })
        }
    }
}

class ImmutableViewModel: ObservableObject {
    @Published var counter = 0
    
    func tapButtonPlus1() {
        plus(with: 1)
    }
    
    func tapButtonPlus5() {
        plus(with: 5)
    }

    func plus(with number: Int) {
        counter = counter + number
        print(counter)
    }
}
```
这些属性包装器在类型中增加了以下划线为前缀加上原名的属性，在需要手写初始化代码时可以在初始化器中显式赋值。
有发出通知的可观察类`ObservableObject`，`View`中就要有存放实例并接收通知的属性包装器`@StateObject`。
顺便一提，enum枚举体虽然可以实现`View`协议，但是不能含有如@StateObject属性、@State属性等作为视图状态变量存放的存储属性。以enum为载体的`View`讲好听点叫纯函数式，作为单状态视图是可行的，不过它更适合当Model。
好，可以到ImmutableStateTests.swift中测试了：
```Swift
// 这行不需要@MainActor，是因为没有@StateObject等属性包装器要求
// 生产代码中，ViewModel自身和面向View层的成员应当尽量被@MainActor修饰
@Test func example() async throws {
    let viewModel = ImmutableViewModel()
    
    viewModel.tapButtonPlus1()
    viewModel.tapButtonPlus5()
    
    assert(viewModel.counter == 6)
}
```
分别输出1和6并且测试通过。
这看似很简单的测试，原先M-V代码来是无法运行通过的。

# 对外不可变的封装
有一个很自然而然的想法：既然已经把代码写在ViewModel中了，就不太希望成员被外部的View层修改。当然如TextFeild等双向绑定时依旧要能修改，不过对于其他不需要双向绑定的成员，仍希望在代码中明确它单向发布通知的语义时，可以通过setter设置器的访问控制来达成。写全了是这样的：
```Swift
@Published internal private(set) var counter = 0
```
其中`internal`可以和`private(set)`互换位置，当然也可以照常把`internal`省略以使用类型的总访问级别。
这也还是不全，`counter = 0`被我们写的方法和无小数点的语境推断为`Int`，如果有位数的要求可以写明，并在plus方法中修改：
```Swift
@Published private(set) var counter: Int64 = 0 // 此处以64位为例
```
这个counter的类型就是所谓Model。
一般来说，Model用struct结构体作为载体是比较灵活的，enum枚举体稍显死板却具备语义。实际上大部分Model都会写成struct，其中内嵌一层可解析JSON字段的enum。这里重点介绍enum枚举体的Model写法。
在函数中，只依赖参数值、不依赖外部可变值的函数是**纯函数**。
纯函数理念重视把可变的部分转移到函数体外部，保证在函数体中涉及的量都是可预测的，重视可靠性。尽管写程序总是需要可变的部分，不变的部分的保障对代码可靠性提升却不可忽视。
在业务流程图中，总是会有节点面临不同的分支流向，它们往往会在某个节点折返，或者有着错综复杂的变化走向，这些分支流向就是**业务状态**。
业务状态的抽象形式与enum枚举体相性很好，它们都接收一些固定的参数，都可以根据自身所在位置推断出某个结果。
另外，ViewModel原则上不应当拥有子类，因此也应在class前添加`final`关键词，这还将使编译速度得到些许提高。
试试写一个以enum枚举体为载体的Model，忽略进位，仅关注个位：
```Swift
enum ImmutableModel {
    case zero
    case increment(counter: Int)
    
    mutating func plus(with number: Int) {
        // 限定number是正整数
        guard number > 0 else { return }
        let rest = number % 10
        switch self {
        case .zero:
            self = .increment(counter: rest)
        case .increment(counter: let counter):
            let sumaryRest = (counter + rest) % 10
            if sumaryRest == 0 {
                self = .zero
            } else {
                self = .increment(counter: sumaryRest)
            }
        }
    }
}
```
其中.zero用例是初始状态，.increment用例是计数器不为0的状态，在plus方法中，计数器超过9时，跳转到zero状态。
无论你的工程的业务状态有没有复杂到必须要引入enum来管理状态，你都需要在心中建立一个状态模型。边缘状态和最经常出现的状态同样重要。

# 总结
我们建立了一个小型工程，绘制了一个简单的视图并通过迭代使得它易于测试。为了易于测试，我们基于MVVM模式创建了ViewModel，并借助Combine响应式框架抽象、发布、更新视图状态。最后，我们引入了setter的访问级别、final class和纯函数理念，用于使代码明确封装的不可变性。

