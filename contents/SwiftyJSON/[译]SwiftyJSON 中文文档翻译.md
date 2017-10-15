最近正在编写 iOS APP 的后台接口，调试的时候要解析JSON数据，所以就来学习使用 SwiftyJSON 这个第三方库，但网上搜的资料感觉写的都不怎么样，最后就只能硬着头皮看官方的文档了，尝试着翻译，借此机会学习并且提高一下自己的 English 水平吧。Let's GO. (PS: 虽然官网也有中文翻译，但是好像不是很全。。。。。😆).如果有翻译错误❎，请见谅也请指出，谢谢！

> SwiftyJSON 使得在 Swift 下处理 JSON 数据变得非常容易。

1. [为什么在 Swift 下处理 JSON 数据的方式不是很好](#为什么在 Swift 下处理 JSON 数据的方式不是很好)
2. [需求](#需求)
3. [整合](#整合)
4. [用法](#用法)
   - [初始化](#初始化)
   - [下标](#下标)
   - [循环](#循环)
   - [错误](#错误)
   - [Optional getter](#Optional getter)
   - [Non-optional getter](#Non-optional getter)
   - [setter](#setter)
   - [元对象](#元对象)
   - [文字转换](#文字转换)
   - [合并](#合并)
5. [和 Alamofire 一起使用](#和 Alamofire 一起使用)

## 为什么在 Swift 下处理 JSON 数据的方式不是很好

Swift 是一种严格的类型安全的语言。尽管明确的类型有利于我们减少错误，但是当我们在处理 JSON 和 其他隐式数据类型的时候就变得很痛苦。

拿 Twitter API 举个栗子。我将在 Swift 的下取回一个用户的 “名字”（根据 Twitter 的 API https://dev.twitter.com/docs/api/1.1/get/statuses/home_timeline）。

代码是这样子的：

```swift
if let statusesArray = try? JSONSerialization.jsonObject(with: data, options: .allowFragments) as? [[String: Any]],
    let user = statusesArray[0]["user"] as? [String: Any],
    let username = user["name"] as? String {
    // Finally we got the username
}
```

这样不是很好。

即使我们使用可选绑定，看起来也是一团糟：

```swift
if let JSONObject = try JSONSerialization.jsonObject(with: data, options: .allowFragments) as? [[String: Any]],
    let username = (JSONObject[0]["user"] as? [String: Any])?["name"] as? String {
        // There's our username
}
```

不可读的、一团糟。但是有些东西应该是简单的。

用 SwiftyJSON 你只需这么做：

```swift
let json = JSON(data: dataFromNetworking)
if let userName = json[0]["user"]["name"].string {
  //now you got your value
}
```

并且你也不用担心可选类型拆包。这些都是自动帮你完成。

```swift
let json = JSON(data: dataFromNetworking)
if let userName = json[999999]["wrong_key"]["wrong_name"].string {
  //calm down, take it easy. the ".string" property still produces the current Optional String type with safety
} else {
  //print the error
  print(json[999999]["wrong_ket"]["wrong_name"])
}
```

## 需求

- iOS 8.0+ | macOS 10.10+|watch 2.0+

- Xcode 8

  ​

## 整合

##### CocoaPods(iOS 8+, OS X 10.9+)

你可以使用 CocoaPods 来 安装 `SwiftyJSON` ，只要把一下的添加到 `Podfile` :

```
platform :iOS, '8.0'
use_frameworks!

target 'YourAPPName' do
pod 'SwiftyJSON'
end
```

注意这需要 CocoaPods 版本 36，并且你的 iOS 开发版本至少是 8.0:

##### Carthage(iOS 8+,OS X 10.9+)

你可以通过使用 Cathage 来安装 `SwiftyJSON` ,只需把一下的添加到 `Cartfile`:

```
github "SwiftyJOSN/SwiftyJSON"
```

##### Swift Package Manager

你可以使用 `The Swift Package Manager `  来安装 `SwiftyJSON` ，只需添加合适的描述到你的 `Package.swift` 文件:

```swift
import PackageDescription

let package = Package(
	name: "YOUR_PROJECT_NAME",
	targets: [],
	dependencies: [
      		.Package(url: "https://github.com/SwiftyJSON/SwiftyJSON.git", versions: Version(1, 0, 0)..<Version(3, .max, .max)),
	]
)
```

注意， Swift Package Manager 任然处于早期设计和开发阶段，更多信息请查看 [GitHub Page](https://github.com/apple/swift-package-manager)

##### 手动添加(iOS7+,OS X 10.9+)

为了在你的项目中使用这个库你通常需要:

1. 对于单个项目，只需将 `SwiftyJSON.swift` 拖进你的项目
2. 对于工作组，包含整个 `SwiftyJSON.xcodeproj`

## 用法

##### 初始化

```swift
import SwiftyJSON	
```

```swift
let json = JSON(data: dataFromNetworking)
```

```swift
let json = JSON(jsonobject)
```

```swift
if let dataFromString = jsonString.data(using: .utf8, allowLossyConversion: false) {
  let json = JSON(data: dataFromString)
}
```

##### 下标

```swift
//从 JSON数组 中获取一个 double
let name = json[0].double
```

```swift
//从一个 JSON数组 里获取一个 字符串数组
let arrayNames = json["users"].arrayValue.map({$0["name"].stringValue})
```

```swift
//从一个 JSON字典 中获取一个 string
let name = json["name"].stringValue
```

```swift
//通过元素的路径来获取 string
let path: [JSONSubscriptType] = [1, "list", 2, "name"]
let name = json[path].string
//同样的
let name = json[1]["list"][2]["name"].string
//或者
let name = json[1, "list", 2, "name"].string
```

```swift
//条件苛刻的情况
let name = json[].string
```

```swift
//普通的方法
let keys: [SubscriptType] = [1, "list", 2, "name"]
let name = json[keys].string
```

##### 循环

```swift
//如果 JSON 是字典
for (key, subJson):(String, JSON) in json {
  //做你想做的事情
}
```

即使 JSON 是数组，第一个元素也总是字符串。

```swift
//如果 JSON是数组
//index 是0到JSON数量的一个字符串值
for (index, subJson):(String, JSON) in json {
  //做你想做的事情
}
```

##### 错误

使用下标来获取或设置字典或数组里的一个值

如果 JSON 是：

- 一个数组，应用可能奔溃由于数组越界。
- 一个字典，它将返回 `nil` 并且没有原因。
- 不是字典或者数组，应用可能因为“不认识选择器”而奔溃。

这些在 SwiftyJSON 中都不会发生。

```swift
let json = JSON(["name", "age"])
if let name = json[999].string {
  //做你想做的事情
} else {
  print(json[999].error)//"Array[999] 已经越界了"
}
```

```swift
let json = JSON(["name":"Jack", "age": 25])
if let name = json["address"].string {
  //做你想做的事情
} else {
  print(json["address"].error)//"Dictionary["address"]并不存在"
}
```

```swift
let json = JSON(12345)
if let age = json[0].string {
  //做你想做的事情
} else {
  print(json[0]) // "Array[0] 失败，这不是一个数组"
  print(json[0].error) // "Array[0] 失败， 这不是一个数组"
}

if let name = json["name"].string {
  //做你想做的事情
} else {
  print(json["name"]) //"Dictionary[\"name"] 失败， 这不是一个字典"
  print(json["name"].error) //"Dictionary[\"name"] 失败，这不是一个字典"
}
```

##### Optional getter

```swift
//数字
if let id = json["user"]["favourites_count"].number {
  //做你想做的事情
} else {
  //打印错误
  print(json["user"]["favourites_count"].error)
}
```

```swift
//字符串
if let id = json["user"]["name"].string {
  //做你想做的事情
} else {
  //打印错误
  print(json["user"]["name"])
}
```

```swift
//bool
if let id = json["user"]["is_translator"].bool {
  //做你想做的事情
} else {
  //打印错误
  print(json["user"]["is_translator"])
}
```

```swift
//整型
if let id = json["user"]["id"].int {
  //做你想做的事情
} else {
  //打印错误
  print(json["user"]["id"])
}
...
```

##### Non-optional getter

非可选 getter 名为 `xxxValue`

```swift
//如果不是数字或者空，返回 0
let id : Int = json["id"].intValue
```

```swift
//如果不是字符串或者空，返回 ""
let name: String = json["name"].stringValue
```

```swift
//如果不是数组或者空，返回 []
let list: Array<JSON> = json["list"].arrayValue
```

```swift
 //如果不是字典或者空，返回 [:]
 let user: Dictionary<String, JSON> = json["user"].dictionaryValue
```

##### setter

```swift
json["name"] = JSON("new-name")
json[0] = JSON(1)
```

```swift
json["id"].int = 1234567890
json["coordinate"].double = 9766.766
json["name"].string = "jack"
json.arrayObject = [1, 2, 3, 4]
json.dictionaryObject = ["name" : "Jack", "age" : 25]
```

##### 元对象

```swift
let jsonObject: Any = json.object
```

```swift
if let jsonObject: Any = json.rawValue
```

```swift
//把 JSON 转化为字符串元
if let data = json.rawData() {
  //做你想做的事情
}
```

```swift
//把 字符串元转化为 JSON
if let string = json.rawString() {
  //做你想做的事情
}
```

##### 存在性

```swift
//用来显示 JSON 中是否有指定的值
if json["name"].exist()
```

##### 文字转换

更多关于文字转换的信息:[Swift Literal Convertibles](http://nshipster.com/swift-literal-convertible/)

```swift
//字符串转换
let json: JSON = "I'm a json"
```

```swift
//整型转换
let json: JSON = 1234
```

```swift
//Bool型转换
let json: JSON = true
```

```swift
//单精度浮点数转换
let json: JSON = 2.8756
```

```swift
//字典转换
let json: JSON = ["I" : "am", "a" : "json"]
```

```swift
//数组转换
let json: JSON = ["I", "am", "a", "json"]
```

```swift
//空转换
let json: JSON = nil
```

```swift
//带有下标的数组
var json: JSON = [1, 2, 3]
json[0] = 100
json[1] = 200
json[3] = 300
json[999] = 300 //别担心， 什么都不会发生
```

```swift
//带有下标的字典
var json: JSON = ["name" : "Jack", "age" : 25, "list" : ["a", "b", "c", ["what" : "this"]]]
json["list"][3]["what"] = "that"
json["list", 3, "what"] = "that"
let path: [JSONSubscriptType] = ["list", 3, "what"]
jaon[path] = "that"
```

```swift
//带有其他的 JSON 对象
let user: JSON = ["username" : "Strve", "password" : "supersecurepassword"]
let auth: JSON = [
  "user": user.object //使用 user.object 替代 user
  "apikey": "supersecuretapitoken"
]
```

##### 合并

把一个 JSON 与另外一个 JSON 合并是可能的。合并两个 JSON 会把原始 JSON 中不存在的只出现在另外一个 JSON 中的数据添加进来。

如果两个 JSON 包含了的值有相同的键，通常情况下会重写原JSON的值，但是有两种情况下会有特别的处理：

- 在两个值都是`JSON.Type.array` 的情况下，值形成在另一个JSON中的数组，附加到原始JSON的数组值。
- 在两个值都是 `JSON.Type.dictionary` 的情况下，两个JSON值都以合并封装JSON的相同方式合并。

在JSON中的两个字段具有不同类型的情况下，值将始终被覆盖。

有两种不同的方式合并： `merge` 修改原始的 JSON ，而合并在副本上以非破坏性方式工作。

```swift
let original: JSON = [
    "first_name": "John",
    "age": 20,
    "skills": ["Coding", "Reading"],
    "address": [
        "street": "Front St",
        "zip": "12345",
    ]
]

let update: JSON = [
    "last_name": "Doe",
    "age": 21,
    "skills": ["Writing"],
    "address": [
        "zip": "12342",
        "city": "New York City"
    ]
]

let updated = original.merge(with: update)
// [
//     "first_name": "John",
//     "last_name": "Doe",
//     "age": 21,
//     "skills": ["Coding", "Reading", "Writing"],
//     "address": [
//         "street": "Front St",
//         "zip": "12342",
//         "city": "New York City"
//     ]
// ]
```

##### 文字表示

有两种可选可以用：

- 使用 Swift 默认的
- 使用一个自定义的，可以很好地处理可选项，并将 `nil` 表示为 `“null”` :

```swift
let dict = ["1":2, "2":"two", "3": nil] as [String: Any?]
let json = JSON(dict)
let representation = json.rawString(options: [.castNilToNSNull: true])
// representation is "{\"1\":2,\"2\":\"two\",\"3\":null}", which represents {"1":2,"2":"two","3":null}
```



## 和 Alamofire 一起使用

SWiftyJSON 很好地包装了 Alamofire JSON 相应的结果：

```swift
Alamofire.request(url, mothod: .get).validate().responseJSON {
  response in
  switch response.result {
    case .success(let value):
    	let json = JSON(value)
    	print("JSON:\(json)")
    case .failure(let error):
    	print(error)
  }
}
```







