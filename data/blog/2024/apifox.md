# Apifox操作指南





# 什么是Apifox

> [Apifox](https://apifox.com/) 是集 API 文档、API 调试、API Mock、API 自动化测试多项实用功能为一体的 API 管理平台，定位为 `Postman + Swagger + Mock + JMeter`。旨在通过一套系统、一份数据，解决多个工具之间的数据同步问题。只需在 Apifox 中定义 API 文档；API 调试、API 数据 Mock、API 自动化测试等功能就可以直接使用，无需再次定义。API 文档和 API 开发调试流程在同一个工具内闭环，API 调试完成后即可确保与 API 文档定义完全一致。高效、及时、准确！

简单理解Apifox就是一款可以帮助我们实现**API全流程管理**的工具

主要功能包含以下几部分

- 可视化API设计
- API调试功能（对标Postman）
- API自动化测试
- 文档发布功能
- Mock数据

接下来就围绕这几点详细描述具体的操作



# 前置工作

## 安装与更新

如果是开发者**强烈建议**使用桌面版，因为Web版会有一些功能不支持，可以通过[Apifox](https://apifox.com/) 链接下载，同时建议把自动更新打开



## 加入项目

可以让团队创建者发邀请链接，然后使用公司邮箱注册。进入项目后记得改名方便识别



# 可视化API设计

![image-20240126190604966](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126190604966.png)

进入接口管理模块 => 接口

可以看到**接口**就是用树形文件夹的结构来管理API的功能，我们可以再下面创建子文件夹也可以直接创建新接口

## 创建/编辑接口

点击加号或在对应文件夹下点击添加

![image-20240126191440029](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126191440029.png)

### 基础信息

必填项包括

- HTTP Method
- API URL（注意规范）
- API名称（用于搜索接口使用）

其他如说明、责任人、标签等信息，视情况填入

![image-20240126192354543](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126192354543.png)

### 请求参数

根据需要选择请求参数的方式或者**组合**

![image-20240126192718444](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126192718444.png)

如果使用**Params**(Query)，键入参数名和选择类型，其中加*****代表是必传参数，反之为可选参数

![image-20240126200702430](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126200702430.png)

使用**Body**传递参数

如果需要上传文件，使用`form-data`类型，字段类型选择`file`

除此之外都是用**json**来作为入参的格式

![image-20240126201031656](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126201031656.png)

此外还可以为接口设置Cookie、Header等参数



### 返回响应

默认情况下，我们都使用**JSON**作为接口的返回格式。接口会自动给我们创建一个空的状态码为200的返回，我们可以通过**手动**操作一个个把返回参数添加进去

![image-20240126201758339](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126201758339.png)

我们也可以通过设计好的示例数据来自动生成定义，这样同时也会保存一份成功示例。

但是这样生成的数据全部都是必填的，可能还需要后续调整

```json
{
  "code": 0,
  "data": {
    "name": "mcdaddy",
    "id": "xasdjfoahfhadjohjaohfw",
    "age": 18
  },
  "message": ""
}

```



![20240126202719_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240126202719_rec_.gif)

我们可以为一个接口在不同的HTTP Code下定义不同的返回结构。最后点击保存，一个接口就录入完成，编辑接口也是同理





## 数据类型

![image-20240126203406760](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126203406760.png)

### 基础类型

字段的类型根据需要选择

- **string/boolean/null/integer/number**都是基本类型，含义和我们代码中的类型是一样的，其中integer和number的区别就是前者不能是浮点数
- **object**就是对象类型，可以在它下面添加子属性
- **array**就是数组类型，如果选择了此类型，需要在其下的**ITEMS**中指定数组中元素的类型
  ![image-20240126203901613](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126203901613.png)
- **null**表示空，一般只在组合模式下使用
- **any**表示任意类型，除非后端开发自己真的不知道，否则用不到



### 组合模式

![image-20240126204750023](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126204750023.png)

组合模式是用于将两个或两个以上**对象类型**进行组合的定义方式，现在假设有两个类型

```ts
interface A {
  a: string;
  b: number;
}

interface B {
  c: boolean
}
```

- AND与：**allOf**组合后这个字段永远会返回a、b、c是三个字段
- OR或：**anyOf**组合后这个字段只会返回a、b或者c两种情况
- XOR异或：**oneOf**和OR的效果一样，区别是如果作用在array类型的ITEMS上时，同样定义类型可以为A或者B， OR可以让数组实际返回`[A,B,A,B]`这种情况，但XOR只能返回`[A,A,A,A]`或者`[B,B,B,B]`

### 引用类型

引用类型是非常常用的定义提效手段

![20240126210643_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240126210643_rec_.gif)

我们可以把我们在代码中定义实体类维护在**数据模型**中，在定义接口时只需要选择定义好的数据模型即可自动关联，而且一次定义多次使用，无需重复定义

在添加之后可以保持与模型的绑定关系，这样将来模型发生变化自动关联到API，但此时是不能修改的， 只能解除绑定后自定义修改

![image-20240126211053592](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240126211053592.png)

#### 应用场景

- 类型的**枚举值**，建议可以维护在数据模型中，在使用时不需要反复手敲
- CRUD时，我们要求Create/Read/Update时使用同一套数据结构

### 高级设置

除了上面提到的各种类型，我们往往还需要对字段进行精细化的限定，可以在任意字段类型旁点击图标，打开高级设置

![image-20240129102004412](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129102004412.png)

我们以一个string类型为例

![image-20240129102151719](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129102151719.png)

我们可以从以下维度来对他进行限定

- **必须**：是否是必填参数或者说是必须返回的出参
- **允许NULL**：如果选择了允许NULL，那么就和`anyOf(string, null)` 等价，属于一种简写方式
- **枚举**：可以通过录入枚举值，限定字段值的范围
  ![image-20240129103019273](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129103019273.png)
- **常量**：把值限定为一个常量值，效果等同于只有一个值的枚举（不建议使用）
- **format**：按格式来限制字段，这里预设了非常多常用的格式，这些是对现有类型的延伸，这个和OPENAPI的规范是一致的
  ![image-20240129104548587](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129104548587.png)

- **最大/最小长度**：这个可以帮助解决很多数据库溢出问题

## 默认响应模版

![image-20240129112826279](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129112826279.png)

在**组件库** => **默认响应模版**，可以定义全局的返回结构，这样我们就不需要每次创建接口时一个个输入结构体的内容



# API调试功能

API定义结束后，我们就可以开始前后端的开发了，针对我们后端同学而言，一旦开发结束，就需要把代码部署到测试环境进行联调测试，但在此之前我们还需要完成一步**API自测**

## 环境设定

![image-20240129170547946](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129170547946.png)

在定义好的API上点击运行，是类似Postman的一个发送请求的一个界面，默认是在本地Mock环境运行

我们可以通过点击右上角进行**环境管理**

![image-20240129171751968](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129171751968.png)

录入对应的测试环境、正式环境或者各种自定义环境

选择环境，回到API，输入相应的参数，点击**发送**，即可开始接口测试

## 登录鉴权

针对不同的场景工具有提供不同的鉴权方法，针对我们目前业务的实际场景，除了ToB的场景应该用的都是用**jwt token**做前后端通信鉴权

![image-20240202154305651](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202154305651.png)

打开全局设置，在**全局参数**中添加**Header**参数，预设一个`x-jwt-token`的参数，并且在默认值里填写一个变量，然后到**全局变量**中添加刚才设的默认值参数，**注意**要把值写在本地值的位置，否则就会成为团队共享

![image-20240202154551069](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202154551069.png)

经过这个设置，项目里的每个接口都会自动关联上这个全局参数，不必手动设置。而每次运行调试时，都会读取全局变量值来发送实际请求

![image-20240202154650008](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202154650008.png)



## 结果校验

![image-20240129173026231](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129173026231.png)

工具会自动根据定义来校验返回的结构体

如上图，id字段在定义中是`integer`，但实际返回是一个`string`，工具就会把错误在**校验响应**中展示

类似的我们还可以校验比如：数字的范围、字符串的长度、数组的长度、枚举的校验、字段的有无等等

如果验证无误，可以点击**保存为用例**把当前的入参保存成一个**测试用例**，将来可以随时查看历史的用例，同时也是一个自测的证明

![image-20240129174112409](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129174112409.png)



# API自动化测试

![image-20240129174934924](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129174934924.png)

进入自动化测试模块，可以看到类似接口管理的文件夹目录，可以在下面添加测试场景

一个测试场景是由N个测试步骤组成的

![image-20240129190530996](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129190530996.png)

我们可以选择**从接口导入**或者**从接口用例导入**，两者的区别是后者会带有用例中的请求参数，一般我们推荐还是从接口用例进行导入，同时导入方式选择**引用**，引用与复制的区别是如果用例本身发生改变，这里也会跟随改变，否则仅仅是一次性的关联关系，同时如果想修改接口参数的话也需要跳转到接口用例去修改

![image-20240129190955743](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129190955743.png)

## 场景1：接口前后关联

我们在自动化测试时，前后步骤是相互关联的，比如第一步创建一个资源，第二步通过get接口验证是否创建成功

![image-20240129203208587](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240129203208587.png)

在前一个接口中找到**后置操作** => **添加后置操作** => **提取变量**

![image-20240130102150906](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130102150906.png)

给变量取一个名字，选择来源，一般情况都是从Response的JSON提取，但也支持从Header，Cookie等提取

![20240130102442_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240130102442_rec_.gif)

这里工具提供了一个可视化的提取变量路径的方式，左侧是根据定义生成的Mock数据，在右边敲表达式可以实时拿到对应路径的数据，完成这步之后，我们就定义了一个名为`petId`的临时变量

现在来到后一个接口，在入参中输入`{{petId}}`就会读取由上一步产生的petId临时变量

![image-20240130111618554](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130111618554.png)

## 场景2：结果断言

假设第一步创建了一个资源，第二步是请求资源的列表接口，那么我们想验证返回列表根据创建时间倒序排序的第一条记录是不是刚刚创建的那个资源

![image-20240130112607253](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130112607253.png)

在列表接口中找到**后置操作** => **添加后置操作** => **断言**

![image-20240130112742917](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130112742917.png)

选择断言对象是返回的JSON，通过同样的方式或者目标数据的表达式，最后设置断言条件（等于，不等于等），条件的对比值就是上一个接口中**提取**出来的临时变量

如果断言失败那这个测试用例就失败了，这个概念和我们各种语言的单测逻辑是一致的，所以也可以变种出各种玩法

## 场景3：条件触发

除了串行得一个个执行测试用例以外，我们可能还会遇到需要条件触发的场景

![image-20240130144037726](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130144037726.png)



### 条件分支

假设我们获取一个接口，根据返回的结果判断类型，如果是a类型就请求A接口，如果是b类型就请求B接口，这时候可以使用**条件分支**，实现动态触发用例

![image-20240130144550712](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130144550712.png)

### For循环

假设我们有一个操作接口，它可以接受各种操作符或者状态枚举，比如`+/-/*/除`，那么我们不需要为4种操作添加四个操作步骤，而利用**ForEach**实现循环调用

![image-20240130160224544](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130160224544.png)

这里需要注意，填入的枚举值，必须是JSON形式的（**双引号不能是单引号**）

![image-20240130160437956](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130160437956.png)

同时在接口入参，需要写成`{{$.id.element}}`的形式，其中图中的id 13指的是ForEach循环这个步骤的ID

![image-20240130160331474](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130160331474.png)

这样运行就能得到4条用例的运行结果

同理，我们也可以根据接口运行结果，实现动态循环

![image-20240130161216068](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130161216068.png)

这里是通过一个列表接口，拿到一个数组，然后抽取出结构体中的id，成为ForEach循环的条件，然后传递给下面的详情接口调用，其中的15就是列表接口这个步骤的ID





## 运行与查看结果

![image-20240130141716435](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130141716435.png)

在测试场景的右侧，我们可以设定运行的环境，运行的次数和并发数，甚至可以性能压测的一些能力。设定完毕后点击运行即可开始自动化测试

![image-20240130142135833](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240130142135833.png)

运行结束后，可以看到一个执行结果列表，可以看到各个接口的执行结果，一旦失败可以展开查看失败的原因

## 如何设计自动化测试

- 自动化测试不是**一次性**的工作，
- 首次添加API可以寻找接口间的关联关系进行串联，比如CRUD，一般很少一个API是完全孤立存在
- 在开发迭代中，有线上或验收中发现的API问题，针对此特殊条件添加测试用例
- 更多的情况可能是我们在回归时发现历史的用例走不通了，此时如果排除了接口Bug的前提后就需要相应调整用例



# 文档发布功能

## 文档分享

我们的接口可能不仅仅是在组内流通，也可能需要暴露给外部用户调用，此时就需要一个脱离工具的文档展示形式

![image-20240202113419888](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202113419888.png)

![image-20240202113720816](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202113720816.png)

其中分享接口范围可以指定具体分享哪些接口，确保对外暴露的粒度

![image-20240202113950647](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202113950647.png)

通过以上步骤，我们就可以得到一个详细完整的API文档了，甚至可以在页面上进行调试，这点和swagger是一致的

## 接口导出

Apifox的定义是遵循**OpenAPI 3.0**[规范](https://openapi.apifox.cn/)的，那什么是OpenAPI规范呢？

> OpenAPI 规范（OAS），是定义一个标准的、与具体编程语言无关的RESTful API的规范。OpenAPI 规范使得人类和计算机都能在“不接触任何程序源代码和文档、不监控网络通信”的情况下理解一个服务的作用。如果您在定义您的 API 时做的很好，那么使用 API 的人就能非常轻松地理解您提供的 API 并与之交互了。
>
> 如果您遵循 OpenAPI 规范来定义您的 API，那么您就可以用文档生成工具来展示您的 API，用代码生成工具来自动生成各种编程语言的服务器端和客户端的代码，用自动测试工具进行测试等等。

那如何做到让所有人和计算机都能理解认识呢？本质就是定义个公共的Schema，**OpenAPI就是一种将接口信息用JSON或者YAML的格式描述的规范**，只要我们遵守了这套规范，那么我们就可以在任意支持此规范的工具上切换。

![image-20240202115612879](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202115612879.png)

我们可以通过**项目设置** => **导出数据** => **选择OpenAPI Spec**， 即可导出一份标准的OpenAPI 3.0接口文件

![image-20240202115913557](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202115913557.png)

我们可以把这份导出的文件直接粘贴到Swagger Editor里面去，结果是可以直接使用的

![image-20240202120329948](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202120329948.png)

相对应的，工具也提供反向的导入功能，所以不管是从别的工具迁移到Apifox还是从Apifox迁移到别的工具，基本都可以做到**无缝切换**



# Mock数据

这是面向前端同学的功能，是前端开发提效的核心功能。篇幅原因不在此展开，前端同学可以看下面这个B站视频进行了解

[Apifox Mock功能全解析！高级 Mock 自定义脚本功能尝鲜！](https://www.bilibili.com/video/BV1BZ4y1B7tD/?spm_id_from=333.880.my_history.page.click&vd_source=9271cbf6d691668cba44599b52916a37)





# 额外小功能

## 代码生成

在定义好的接口上，Apifox提供了简单的模型代码生成功能

![image-20240202155506464](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202155506464.png)

可以选择对应的前后端语言，生成对象模型结构体代码，省去手敲的时间

![image-20240202155522199](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202155522199.png)

## 历史记录

![image-20240202155820650](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240202155820650.png)



# 参考资料

[JSONPath](https://github.com/json-path/JsonPath)

[官方文档](https://apifox.com/help/)

