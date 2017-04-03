---
layout:     post
title:      "UI Testing"
subtitle:   ""
date:       2016-08-04
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
image:
  feature: 
  teaser: london_night_5-wallpaper-5120x3200.jpg
  credit: Death to Stock Photo
  creditlink: ""

---
**软件开发过程中，不可避免的一个环节就是测试。在iOS开发中，UI测试更是重要且必要。如何快速且高效的完成UI测试，是我们需要思考的一个问题。**

### UI Testing
*****

苹果在Xcode7中新加入了一套UI Testing的工具，目的就是让我们更快速高效的完成自动化UI测试，UI Testing相对于以往的解决方案要简单很多。特别是在创建测试用例的时候，可以通过录制功能区完成部分测试代码的编写。

#### 集成 UI Testing
*****

**想要在项目中集成UI Testing非常容易，如果是新项目的话，可以在创建的时候勾选上UI Testing:**


![](http://7xl1kp.com1.z0.glb.clouddn.com/test111.png)

**如果是想在一个旧的项目中使用UI Testing的话，可以新建一个 iOS UI Testing 的 target：**

![](http://7xl1kp.com1.z0.glb.clouddn.com/test222.png)

创建好之后 Xcode将为你配置好你所需要的 UI 测试环境。

#### UI Record
*****
当我们创建完成UI Testing工程的时候，会默认生成一个${ProjectName}UITests的文件。首先来看看文件的大致结构：

```
import XCTest

class UITestingUITests: XCTestCase {
        
    override func setUp() {
        super.setUp()
        
        // Put setup code here. This method is called before the invocation of each test method in the class.
        
        // In UI tests it is usually best to stop immediately when a failure occurs.
        continueAfterFailure = false
        // UI tests must launch the application that they test. Doing this in setup will make sure it happens for each test method.
        XCUIApplication().launch()

        // In UI tests it’s important to set the initial state - such as interface orientation - required for your tests before they run. The setUp method is a good place to do this.
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
    func testExample() {
        // Use recording to get started writing UI tests.
        // Use XCTAssert and related functions to verify your tests produce the correct results.
    }
    
}


```

写过单元测试的同学对这个结构一定不会陌生，和单元测试的思路类似，在每一个 UI Testing 执行之前，我们都希望从一个“干净”的 app 状态开始进行。

Apple为UI Testing引入了三个新的APIS：

![](http://7xl1kp.com1.z0.glb.clouddn.com/test444.png)

- XCUIApplication。这是你正在测试的应用的代理。它能让你启动应用，这样你就能执行测试了。它每次都会新起一个进程，这会多花一些时间，但是能保证测试应用时的状态是干净的，这样你需要处理的变量就少了些。

- XCUIElement。这是你正在测试的应用中UI元素的代理。每个元素都有类型和标识符，结合二者就能找到应用中的UI元素。所有的元素都会嵌套在代表你的应用的树中。

- XCUIElementQuery。 当你想要找到某个元素时，就会用到 element query。每个 XCUIElement 里都包含一个query。这些query搜索 XCUIElement 树， 必须要找到一个匹配的。否则当你视图访问该元素时，测试就会失败。 例外是exists 属性，你可以使用这个属性来检查一个元素是否展示在树中。 这对于断言很有用。 更一般地你可以使用 XCUIElementQuery 来找到对accessibility可见的元素。Query会返回结果的集合。

**UI Testing带来的最大便利便是可以直接录制操作。当你点击左下角的小红点的时候，工程项目就会跑起来，这时候你你在app界面上所做的任何操作都会被录制，最终转换成代码显示在你的测试文件中。**

![](http://7xl1kp.com1.z0.glb.clouddn.com/test333.png)

**值得注意的是，有些时候我们发现小红点状态是enable的，这个时候是因为Xcode一直在indexing的状态，我们可以尝试执行下面的代码：**

```
rm -rf ~/Library/Developer/Xcode/DerivedData/

```

**等到Xcode reindex 我们的project完成后，录制功能就可以用来。**


##### 录制出错
*****

某些时候，在录制时，当你点击了一个元素，你可能会注意到生成的代码看上去不太对。这通常是因为你正在交互的元素对Accessibility不可见。你可以使用Xcode的Accessibility Inspector来检查是不是这种情况。

![](http://7xl1kp.com1.z0.glb.clouddn.com/test555.png)


**随着 UI Testing 的推出，Apple 也将原来的 UI Automation 一系列内容标记为弃用。这意味着 UI Testing 至少在今后一段时间内将会是 iOS 开发中的首选工具。所以赶紧在你的项目中使用起来吧。**



































