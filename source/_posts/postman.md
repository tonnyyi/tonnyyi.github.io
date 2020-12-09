---
title: postman
tags:
  - postman
  - test
categories:
  - test
date: 2020-10-21 17:29:13
---



## 基础操作

### 请求参数

- 在**Params**标签下添加参数
- 在url中输入类型`:param`格式定义pathVariable
- 右击参数值, 可以对其进行转码, 点击**Params**右侧的**Settings**可以设置该请求默认进行参数值码
- 点击**Bulk Edit**可以更改参数编辑展示方式, 编辑起来更方便



<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022170836.png" alt="image-20201022170834177" style="zoom:50%;" />

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022171009.png" alt="image-20201022171007473" style="zoom:50%;" />



### 响应对象

#### 查看响应对象

可以按不同方式查看响应对象, 格式化后/纯文本的/预览等, 还可以查看本请求的网络信息, 请求耗时, 响应体大小

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022172207.png" alt="image-20201022172205335" style="zoom:50%;" />

#### 响应示例

可以将某次的响应内容保存起来, 作为样例(或者保存到文件), 从而能随时查看比如 正常响应/异常响应 是什么样子, 这在服务无法访问时挺有用. 另外如果在Postman里定义Mock服务的话也会用到.

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022171835.png" alt="image-20201022171834591" style="zoom:50%;" />

定义响应示例名称

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022171918.png" alt="image-20201022171917208" style="zoom:50%;" />

点击右上角查看响应示例

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022172019.png" alt="image-20201022172016936" style="zoom:50%;" />



### 请求组

可以将多个请求保存到一个请求组(Collections)中, 好处是: 增加额外Collection变量作用域, 增加公共的PreRequest和Test脚本, 批量执行Collection下的请求, 定义Collection内请求的顺序

点击左上角**New** > **Collection**, 或者点击请求路径右边的**Save**, 保存请求的同时创建Collection

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022173350.png" alt="image-20201022173345337" style="zoom:50%;" />

Collection下还可以新建文件夹, 对请求进一步划分, 比如: 用户模块, 订单模块...

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022173712.png" alt="image-20201022173711403" style="zoom:50%;" />

### 环境

同一个接口, 在不同的环境都有服务, 比如: 研发环境, 测试环境, 线上环境, 环境不同参数值可能有不同, 比如: 研发环境服务ip为127.0.01, 测试环境服务IP为172.31.11.12, 线上环境为www.test.com, 此时可以先定义好环境数据, 然后在不同环境下定义同名变量, 但变量值不同. 实际发送请求前, 选择不提供的环境, 这样变量值就会更新. 

点击左上角**New** > **Environment** 新建环境, 也可以点击右上角**Manager Environments**新建

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022181220.png" alt="image-20201022181218873" style="zoom:50%;" />

设置变量值

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022181512.png" alt="image-20201022181511177" style="zoom:50%;" />

这里需要区分一下**INITIAL VALUE** 和 **CURRENT VALUE**, **INITIAL VALUE** 就是在定义该变量时赋予的初始值, 这个值会同步到Postman服务器, 而**CURRENT VALUE**是发送请求时实际被使用变量值, 默认与**INITIAL VALUE** 的值相同, 但不会同步到Postman服务. 对于一些敏感数据(比如密码)不应该设置**INITIAL VALUE**. 

**INITIAL VALUE用作团队合作时定义一个初始值,  CURRENT VALUE适合运行时根据实际情况去调整**



可以点击小眼睛按钮快速查看Global和当前环境的变量列表

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022182048.png" alt="image-20201022182046452" style="zoom:50%;" />

调用请求时在环境选择下拉菜单中选择需要的运行环境就可以了



## 变量

可以在不同作用域定义好变量后, 在某个请求中引用该变量, 从而做到变量复用和统一管理

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022174158.png" alt="image-20201022174156959" style="zoom:50%;" />

可以直接接url上使用变量, 也可以在请求参上使用, 格式是`{{变量名}}`, 这里变量名可以是手动定义的变量名, 也可以是动态变量名(如: `{{$randomInt}}`随机数字)



### 变量作用域

postman中变量有多个作用域, **越往内的作用域优先级越高**(global < collection < environment < ...), 举例来说, 如果global作用域和Environment作用域里都有一个变量叫`username`, 那么最终使用的是Environment作用域里的变量值.

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021200103.jpg" alt="Variable Scope" style="zoom:33%;" />

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022175110.png" alt="Variable Scopes" style="zoom: 80%;" />



### 管理变量

可以基于现有数据定义变量

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022105419.jpg" alt="Set as variable" style="zoom:50%;" />

依次点击**Set as variable** > **Set as a new variable**.

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022105514.jpg" alt="Set as variable" style="zoom:50%;" />

输入变量名, 确认变量值, 选择作用域

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022105613.jpg" alt="Set as variable" style="zoom:50%;" />

#### 管理global和environment作用域变量

点击环境下拉菜单旁边的眼镜按钮, 在弹出层中点击edit进行编辑

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022105918.jpg" alt="Environment Quick Look" style="zoom:50%;" />

点击旁边的齿轮按钮, 可以创建/删除/编辑运行环境配置, 点击某个运行环境项后, 可以对其下的变量进行管理

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022110044.jpg" alt="Manage Environments" style="zoom:50%;" />

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022110110.jpg" alt="Manage Environments" style="zoom:50%;" />

#### 管理collection作用域的变量

在左侧collections列表中, 点击collection右侧的 **...** > **Edit** > **Variables**, 然后就可以对变量进行管理了

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022110429.jpg" alt="Edit Collection" style="zoom:50%;" />

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022110505.jpg" alt="Collection Variables" style="zoom: 33%;" />

#### 在脚本中操作变量

```javascript
// 读取
pm.variables.get("username");	// 读取变量, 依次从各个作用域查找
pm.globals.get("username");
pm.collectionVariables.get("username");
pm.environment.get("username");

// 赋值
pm.variables.set("username", "Admin");	// 临时覆盖, 优先级最高, 请求结束后失效
pm.globals.set("username", "Admin");
pm.collectionVariables.set("username", "Admin");
pm.environment.set("username", "Admin");

// 清理
pm.environment.unset("username");
```

#### 请求时使用变量

在发送请求时, 使用`{{变量名}}`的格式获取变量值, 使用动态变量的方式是`{{$randomInt}}`

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022110748.jpg" alt="Variables in Request" style="zoom:50%;" />

#### data作用域变量操作

点击左上角的**Runner**按钮, 在弹出页面中选择要执行collection, 点击**Select File**按钮, 注意**文件必须是JSON或者CSV格式**, json文件样例如下

```json
[{
  "path": "post",
  "value": "1"
}, {
  "path": "post",
  "value": "2"
}, {
  "path": "post",
  "value": "3"
}, {
  "path": "post",
  "value": "4"
}]
```



<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201022111225.png" alt="Setup Collection Run" style="zoom:50%;" />

另外, 在脚本里访问data文件需要使用`iterationData`对象, 脚本样例如下

```javascript
pm.iterationData.get("value")
```





## 脚本

[Postman API文档](https://www.postmanlabs.com/postman-collection/index.html)

### 脚本执行顺序

执行每个请求时会依次执行如下脚本:

- Collection级别的pre-request脚本
- Folder级别的pre-request脚本
- Request级别的pre-prequest脚本
- Request级别的test脚本
- Folder级别的test脚本
- Collection级别的test脚本

如果对应的脚本没有定义, 则会跳过. Collection和Folder级别的脚本在执行每个请求时都会被调用, 所以适合写一些需要复用的逻辑.

![workflow for request in collection](https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021173020.png)



Collection或Folder脚本编辑入口是点击右侧的"...", 在弹出参数中点击"edit"

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021175352.jpg" alt="Collection Actions" style="zoom:50%;" />

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021175514.png" alt="image-20201021175511460" style="zoom:50%;" />



### Test脚本

test脚本用于对请求结果进行验证处理, 比如: 判断响应码是否为200, 保存请求结果到本地文件等

```javascript
pm.test("响应码验证", function(){
    pm.response.to.have.status(200);
})

// expect语法
pm.test("响应码为200", function(){
    pm.expect(pm.response.code).to.equal(200);
    pm.expect(pm.response).to.have.property("code", 200)
})
```

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021180821.png" alt="image-20201021180816278" style="zoom:50%;" />

发送请求之后, 在**Tests Results**标签下就能看到测试结果

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021180353.png" alt="image-20201021180348328" style="zoom:50%;" />



Postman使用 [Chai Assertion Library BDD](https://www.chaijs.com/api/bdd/)库提供的 [断言语法](https://www.chaijs.com/api/bdd/) 进行验证

```javascript
// 链式调用, 增加可读性
to
be
been
is
that
which
and
has
have
with
at
of
same
but
does
still

expect({a: 1}).to.not.have.property('b');
```

#### 常用测试代码

```javascript
pm.test("响应状态码为200", function () {
  pm.response.to.have.status(200);
  pm.expect(pm.response.code).to.eql(200);
});

// GET https://gank.io/api/v2/banners
pm.test("成功响应且有数据", function() {
	pm.expect(pm.response.code).to.eql(200);

    const respJson = pm.response.json();
    pm.expect(respJson.data).to.length.gt(0);
})

// GET https://gank.io/api/v2/data/category/GanHuo/type/iOS/page/1/count/10
pm.test("分页数据满页", function() {
    const respJson = pm.response.json();
    pm.expect(respJson.data.length).to.eq(10);
    pm.expect(respJson.data).to.have.lengthOf(10);
})

// 响应数据转换
// json
const responseJson = pm.response.json();
// xml
const responseJson = xml2Json(pm.response.text());
// csv
const parse = require('csv-parse/lib/sync');
const responseJson = parse(pm.response.text());
// html https://cheerio.js.org
const $ = cheerio.load(pm.response.text());
console.log($.html());
console.log($(".header"));
// 纯文本
pm.expect(pm.response.text()).to.include("customer_id");
pm.response.to.have.body("whole-body-text");
```

#### 常用API

```javascript
// 请求体验证
const responseJson = pm.response.json();
pm.expect(responseJson.name).to.eql("Jane");
// 状态码
pm.response.to.have.status(200);
pm.expect(pm.response.code).to.be.oneOf([200,404,500]);
pm.response.to.have.status("OK");
// 请求头
pm.response.to.have.header("Content-Type");
pm.expect(pm.response.headers.get('Content-Type')).to.eql('application/json');
});
// cookie
pm.expect(pm.cookies.has('JSESSIONID')).to.be.true;
pm.expect(pm.cookies.get('isLoggedIn')).to.eql('1');
// 响应时间 ms
pm.expect(pm.response.responseTime).to.be.below(200);

// 在脚本中发送请求
pm.sendRequest("https://gank.io/api/v2/banners", function(err, resp) {
    console.log(resp.json())
})
```

#### 常用断言

```javascript
const jsonData = pm.response.json();

// 响应值等于某个预先定义的变量值
pm.expect(jsonData.name).to.eql(pm.environment.get("name"));
// 值类型
pm.expect(jsonData).to.be.an("object");
pm.expect(jsonData.name).to.be.a("string");
pm.expect(jsonData.age).to.be.a("number");
pm.expect(jsonData.hobbies).to.be.an("array");
pm.expect(jsonData.website).to.be.undefined;
pm.expect(jsonData.email).to.be.null;
// 数组属性
pm.expect(jsonData.errors).to.be.empty;
pm.expect(jsonData.areas).to.include("goods");
const contactSettings = jsonData.settings.find(m => m.type === "contact");
pm.expect(contactSettings).to.be.an("object", "找不到联系方式配置信息");
pm.expect(contactSettings.detail).to.have.members(["email", "sms"]);
// 对象
pm.expect({a: 1, b: 2}).to.have.all.keys('a', 'b');
pm.expect({a: 1, b: 2}).to.have.any.keys('a', 'b');
pm.expect({a: 1, b: 2}).to.not.have.any.keys('c', 'd');
pm.expect({a: 1}).to.have.property('a');
pm.expect({a: 1, b: 2}).to.be.an('object').that.has.all.keys('a', 'b');
pm.expect({a: 1, b: 2}).to.deep.include({a:1});	// 包含部分属性
// 集合
pm.expect({"a": 1, "b": 123}.a).to.be.oneOf([1, 123]);
```



### API

#### 动态变量

每次请求时, 如果希望自动生成一个值作为参数使用, 或者在脚本中使用, 这时可以使用PostMan内置的动态变量. Postman使用的是[faker.js](https://www.npmjs.com/package/faker)库来生成变量值, 这个库是支持生成中文随机数据的, 但是目前在Postman里支持英文随机数据.

使用方式如下, 实际请求时url就变成了`https://gank.io/api/v2/banners?testId=107`: 

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021192442.png" alt="image-20201021192440620" style="zoom:50%;" />

在脚本中的使用方式是`pm.variables.replaceIn('{{$randomInt}}')`

<img src="https://tonnyblog.oss-cn-beijing.aliyuncs.com/img/20201021192909.png" alt="image-20201021192907770" style="zoom:50%;" />



##### 通用

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $guid       | guid | "611c2e81-2ccb-42d8-9ddc-2d0bfa65c1b4" |
| $timestamp  | 时间戳,秒 | 1562757107 |
| $randomUUID  | UUID | "6929bb52-3ab2-448a-9796-d6480ecad36b" |

##### 文本数字颜色

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomAlphaNumeric    | 单字符数字 |  6, "y", "z" |
| $randomBoolean         | 布尔 |  true, false |
| $randomInt             | 数字, 1到1000 | 78 |
| $randomColor           | 颜色 |  "red", "fuchsia", "grey" |
| $randomHexColor        | 颜色 |  "#47594a", "#431e48" |
| $randomAbbreviation    | 随机缩写 |  SQL, PCI, JSON |

##### 网络

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomIP            | ip地址 | 241.102.234.100|
| $randomIPV6          | ip地址V6 | dbe2:7ae6:119b:c161:1560:6dda:3a9b:90a9|
| $randomMACAddress    | mac地址 | 33:d4:68:5f:b4:c7|
| $randomPassword      | 密码 | t9iXe7COoDKv8k3, QAzNFQtvR9cg2rq|
| $randomLocale        | 两字符 | "ny", "sr", "si"|
| $randomUserAgent     | 浏览器UA | Mozilla/5.0 (Macintosh; U; Intel Mac...|
| $randomProtocol      | 协议 | "http", "https"|
| $randomSemver        | 随机版本号 | 7.0.5, 2.5.8, 6.4.9|

##### 姓名

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomFirstName    | 名 | Ethan, Chandler, Megane |
| $randomLastName     | 姓 | Schaden, Schneider, Willms |
| $randomFullName     | 全名 | Sylvan Fay, Jonathon Kunze |
| $randomNamePrefix   | 前缀 | Dr., Ms., Mr. |
| $randomNameSuffix   | 后缀 | I, MD, DDS |

##### 职业

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomJobArea        | 地区 |  Mobility, Intranet, Configuration |
| $randomJobDescriptor  | 描述 |  Forward, Corporate, Senior |
| $randomJobTitle       | 职称 |  International Creative Liaison |
| $randomJobType        | 类型 |  Supervisor, Manager, Coordinator |

##### 电话地址

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomPhoneNumber    | 电话 |  700-008-5275 |
| $randomPhoneNumberExt | 电话12位 |  27-199-983-3864, |
| $randomCity           | 城市 |  Spinkahaven, Korbinburgh |
| $randomStreetName     | 街道 |  Kuhic Island, General Street |
| $randomStreetAddress  | 地址 |  5742 Harvey Streets |
| $randomCountry        | 国家 |  Lao People's Democratic Republic, Austria |
| $randomCountryCode    | 国家代码 |  CV, MD, TD |
| $randomLongitude      | 经度 | 171.7139, -159.9757 |
| $randomLatitude       | 纬度 |  55.2099, 27.3644 |

##### 图片

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomImage |  图片  | http://lorempixel.com/640/480/technics |
| $randomAvatarImage |  头像   | https://s3.amazonaws.com/uifaces/faces/twitter/johnsmithagency/128.jpg |
| $randomImageUrl | 图片url   | http://lorempixel.com/640/480 |
| $randomAbstractImage |    图片url   | http://lorempixel.com/640/480/abstract |
| $randomAnimalsImage | 动物图片 | http://lorempixel.com/640/480/animals |
| $randomBusinessImage | 股票业务图像 | http://lorempixel.com/640/480/business |
| $randomCatsImage |    猫图片   | http://lorempixel.com/640/480/cats |
| $randomCityImage |    城市   | http://lorempixel.com/640/480/city |
| $randomFoodImage |    食物   | http://lorempixel.com/640/480/food |
| $randomNightlifeImage |   夜生活  | http://lorempixel.com/640/480/nightlife |
| $randomFashionImage | 时尚潮流    | http://lorempixel.com/640/480/fashion |
| $randomPeopleImage |  人物图片    | http://lorempixel.com/640/480/people |
| $randomNatureImage |  自然 | http://lorempixel.com/640/480/nature |
| $randomSportsImage |  体育 | http://lorempixel.com/640/480/sports |
| $randomTechnicsImage |    科技   | http://lorempixel.com/640/480/technics |
| $randomTransportImage |   运输 | http://lorempixel.com/640/480/transport |
| $randomImageDataUri | base64 | data:image/svg+xml;charset=UTF-8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20version%3D%221.1%22%20baseProfile%3D%22full%22%20width%3D%22undefined%22%20height%3D%22undefined%22%3E%20%3Crect%20width%3D%22100%25%22%20height%3D%22100%25%22%20fill%3D%22grey%22%2F%3E%20%20%3Ctext%20x%3D%220%22%20y%3D%2220%22%20font-size%3D%2220%22%20text-anchor%3D%22start%22%20fill%3D%22white%22%3Eundefinedxundefined%3C%2Ftext%3E%20%3C%2Fsvg%3E |


##### 金融

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomBankAccount  | 8位数字的银行账号 |  09454073, 65653440, 75728757 |
| $randomBankAccountName  | 银行账号名 | Home Loan Account, Checking Account, Auto Loan Account |
| $randomCreditCardMask  | masked credit card number |    3622, 5815, 6257 |
| $randomBankAccountBic  | 银行标识 |   EZIAUGJ1, KXCUTVJ1, DIVIPLL1 |
| $randomBankAccountIban  | 15-31位国际银行账户号码 |   MU20ZPUN3039684000618086155TKZ |
| $randomTransactionType  | 交易类型 |  invoice, payment, deposit |
| $randomCurrencyCode  | 3个字符的货币标识 | CDF, ZMK, GNF |
| $randomCurrencyName  | 货币名称 | CFP Franc, Cordoba Oro, Pound Sterling |
| $randomCurrencySymbol  | 货币符号 |     $, £ |
| $randomBitcoin  | 比特币地址 |    3VB8JGT7Y4Z63U68KGGKDXMLLH5 |

##### 商务

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomCompanyName  | 公司名 |   Johns - Kassulke, Grady LLC |
| $randomCompanySuffix  | 公司后缀 |  Inc, LLC, Group |
| $randomBs  | 商业用语 |    killer leverage schemas |
| $randomBsAdjective  | 商务用语形容词 |  viral, 24/7, 24/365 |
| $randomBsBuzz  | 商业流行语 |    repurpose, harness, transition |
| $randomBsNoun  | 商业用词 |    e-services, markets, interfaces |

##### 流行语

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomCatchPhrase | 流行语 |  Future-proofed heuristic open architecture |
| $randomCatchPhraseAdjective | 流行语形容词 |  Self-enabling, Business-focused, Down-sized |
| $randomCatchPhraseDescriptor | 流行语描述 |   bandwidth-monitored, needs-based, homogeneous |
| $randomCatchPhraseNoun | 流行语名词 |    secured line, superstructure,installation |

##### 数据库

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomDatabaseColumn | 字段名 |   updatedAt, token, group |
| $randomDatabaseType |  字段类型|  tinyint, text |
| $randomDatabaseCollation | 字符集 |   cp1250_bin, utf8_general_ci, cp1250_general_ci |
| $randomDatabaseEngine | 数据库引擎 |  MyISAM, InnoDB, Memory |

##### 日期

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomDateFuture | 未来日期 |   Tue Mar 17 2020 13:11:50 GMT+0530 (India Standard Time) |
| $randomDatePast | 过去日期 |  Sat Mar 02 2019 09:09:26 GMT+0530 (India Standard Time) |
| $randomDateRecent |  最近时间 |   Tue Jul 09 2019 23:12:37 GMT+0530 (India Standard Time) |
| $randomWeekday | 星期 |    Thursday, Friday, Monday |
| $randomMonth | 月份 |     February, May, January |

##### 网站

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomDomainName | 域名 |  gracie.biz, armando.biz, trevor.info |
| $randomDomainSuffix | 顶级域名 |  org, net, com |
| $randomDomainWord | 非法域名 |    gwen, jaden, donnell |
| $randomEmail | 邮箱 |   Pablo62@gmail.com, Ruthe42@hotmail.com |
| $randomExampleEmail | "example"域名的邮箱 |Talon28@example.com, Quinten_Kerluke45@example.net|
| $randomUserName | 用户名 |  Jarrell.Gutkowski, Lottie.Smitham24, Alia99 |
| $randomUrl | URL |  https://anais.net, https://tristin.net, http://jakob.name |

##### 文件目录

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomFileName | 文件名(包括不常见的文件类型) |  neural_sri_lanka_rupee_gloves.gdoc |
| $randomFileType | 文件类型(包括不常见的) |  model, application, video |
| $randomFileExt | 文件后缀(包括不常见的) |  war, book, fsc |
| $randomCommonFileName | 文件名(常见的文件类型) |  well_modulated.mpg4 |
| $randomCommonFileType | 常见的文件类型 |   application, audio |
| $randomCommonFileExt | 常见的文件后缀 |  m2v, wav, png |
| $randomFilePath | 文件路径 | /home/programming_chicken.cpio, |
| $randomDirectoryPath | 目录路径 | /usr/bin, /root, /usr/local/bin |
| $randomMimeType | MIME类型 |audio/vnd.vmx.cvsd |

##### 商店

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomPrice  | 价格, 100.00 到 999.00 |  531.55, 488.76, 511.56 |
| $randomProduct  | 商品 |  Towels, Pizza, Pants |
| $randomProductAdjective  | 随机的产品形容词 | Unbranded, Incredible, Tasty |
| $randomProductMaterial  | 产品材料 | Steel, Plastic, Frozen |
| $randomProductName  | 商品名 |  Handmade Concrete Tuna, Refined Rubber Hat |
| $randomDepartment  | 商业分类 |  Tools, Movies, Electronics |

##### 语法

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomNoun | 名词 |   matrix, bus, bandwidth |
| $randomVerb | 动词 | parse, quantify, navigate |
| $randomIngverb | 现在进行时 "-ing"  |  synthesizing, navigating, backing up |
| $randomAdjective | 形容词 | auxiliary, multi-byte, back-end |
| $randomWord | 单词 |  withdrawal, infrastructures, IB |
| $randomWords | 多个单词 |  Samoa Synergistic sticky copying Grocery |
| $randomPhrase | 一句话 |  You can't program the monitor without navigating the mobile XML program! |

##### 乱数假文

| 变量名 | 说明 | 示例  |
|-----|----|-----|
| $randomLoremWord  | 一个假词 | est |
| $randomLoremWords  | 多个假词 |  vel repellat nobis |
| $randomLoremSentence  | 一句胡话 |  Molestias consequuntur nisi non quod. |
| $randomLoremSentences  | 2-6句胡话 | Et sint voluptas similique iure amet perspiciatis vero sequi atque. Ut porro sit et hic. Neque aspernatur vitae fugiat ut dolore et veritatis. Ab iusto ex delectus animi. Voluptates nisi iusto. Impedit quod quae voluptate qui. |
| $randomLoremParagraph  | 一段胡话 |  Ab aliquid odio iste quo voluptas voluptatem dignissimos velit. Recusandae facilis qui commodi ea magnam enim nostrum quia quis. Nihil est suscipit assumenda ut voluptatem sed. Esse ab voluptas odit qui molestiae. Rem est nesciunt est quis ipsam expedita consequuntur. |
| $randomLoremParagraphs  | 3段胡话 | Voluptatem rem magnam aliquam ab id aut quaerat. Placeat provident possimus voluptatibus dicta velit non aut quasi. Mollitia et aliquam expedita sunt dolores nam consequuntur. Nam dolorum delectus ipsam repudiandae et ipsam ut voluptatum totam. Nobis labore labore recusandae ipsam quo. |
| $randomLoremText  | 一堆胡话 | Quisquam asperiores exercitationem ut ipsum. Aut eius nesciunt. Et reiciendis aut alias eaque. Nihil amet laboriosam pariatur eligendi. Sunt ullam ut sint natus ducimus. Voluptas harum aspernatur soluta rem nam. |
| $randomLoremSlug  | 假url | eos-aperiam-accusamus, beatae-id-molestiae, qui-est-repellat |
| $randomLoremLines  | 1-5行胡话 | Ducimus in ut mollitia.\nA itaque non.\nHarum temporibus nihil voluptas.\nIste in sed et nesciunt in quaerat sed. |



#### pm对象

##### variable属性

`pm.variables`用来对变量进行管理

```javascript
pm.variables.has("username");
pm.variables.get("username");
pm.variables.set("username", "Admin");
pm.variables.replaceIn("Hi, my name is {{$randomFirstName}}"); // 访问动态变量
pm.variables.toObject();	// 合并后的所有变量

// pm.globals pm.collectionVariables pm.environment pm.iterationData 都有下列API
pm.collectionVariables.has("username");
pm.collectionVariables.get("username");
pm.collectionVariables.set("username", "Admin");
pm.collectionVariables.unset("username");
pm.collectionVariables.clear()
pm.collectionVariables.replaceIn("hi {{username}}");	// 访问变量
```

##### request response cookies属性

```javascript
// ------------ request对象 --------------
pm.request.url; 	// Url对象, 包含: 协议, 主机, 路径, 参数, 变量
pm.request.headers;		// Header数组
pm.request.method;		// "GET"
pm.request.body;		// 请求体对象

// 添加请求头
pm.request.headers.add({
  key: "client-id",
  value: "abcdef"
});

// ------------ response对象 --------------
pm.response.code;		// 响应码(数字) 
pm.response.status; 	// "OK"
pm.response.headers;	// 响应头数组
pm.response.responseTime;	// 响应时间ms
pm.response.responseSize;	// 响应体大小
pm.response.text();			// 响应体转字符串格式
pm.response.json();			// 响应体转json格式

// ------------ cookies对象 --------------
pm.cookies.has("test");
pm.cookies.get("test");		// 获取cookie值(字符串)
pm.cookies.toObject();
// 设置cookie的作用域名
const jar = pm.cookies.jar();
jar.set("baidu.com", "username", "Admin", (error, cookie) => {});
jar.set("baidu.com", {"name":"username", "value":"Admin", httpOnly:false},
        (error, cookie) => {});
jar.get("baidu.com", "username", (error, value) => {});
jar.getAll("baidu.com", (error, cookies) => {});
jar.unset("baidu.com", "username", (error) => {});
jar.clear("baidu.com", (error) => {});

// ------------ info对象 --------------
pm.info.eventName;		// "prerequest" "test"
pm.info.iteration;		// run collection时的第几次循环
pm.info.iterationCount;	// 总计划循环次数
pm.info.requestName;	// 本次请求的保存名称
pm.info.requestId;		// 本次请求的唯一标识GUID
```

##### 脚本里发送请求

```javascript
pm.sendRequest('https://postman-echo.com/get', (error, response) => {
  if (error) {
    console.log(error);
  } else {
  console.log(response);
  }
});

// Example with a full-fledged request
const postRequest = {
  url: 'https://postman-echo.com/post',
  method: 'POST',
  header: {
    'Content-Type': 'application/json',
    'X-Foo': 'bar'
  },
  body: {
    mode: 'raw',
    raw: JSON.stringify({ key: 'this is json' })
  }
};
pm.sendRequest(postRequest, (error, response) => {
  console.log(error ? error : response.json());
});

// Example containing a test
pm.sendRequest('https://postman-echo.com/get', (error, response) => {
  if (error) {
    console.log(error);
  }

  pm.test('response should be okay to process', () => {
    pm.expect(error).to.equal(null);
    pm.expect(response).to.have.property('code', 200);
    pm.expect(response).to.have.property('status', 'OK');
  });
});
```

在**Runner**中运行collection下的请求时, 可以通过`postman.setNextRequest(请求名)` 或者 `postman.setNextRequest(请求id)`来指定下一个请求, 这样可以构建起一个调用工作流, 比如先请求数据列表接口, 然后从响应中获取数据id, 再请求数据详情接口.

##### 使用外部库

使用`require`方法能够使用postman内置库, 包括:

- [ajv](https://www.npmjs.com/package/ajv)
- [atob](https://www.npmjs.com/package/atob)
- [btoa](https://www.npmjs.com/package/btoa)
- [chai](http://chaijs.com/)
- [cheerio](https://cheerio.js.org/)
- [crypto-js](https://www.npmjs.com/package/crypto-js)
- [csv-parse/lib/sync](http://csv.adaltas.com/parse)
- [lodash](https://lodash.com/)
- [moment](http://momentjs.com/docs/)
- [postman-collection](http://www.postmanlabs.com/postman-collection/)
- [tv4](https://github.com/geraintluff/tv4)
- [uuid](https://www.npmjs.com/package/uuid)
- [xml2js](https://www.npmjs.com/package/xml2js)

除此之外, 一些NodeJS的模块可以使用:

- [path](https://nodejs.org/api/path.html)
- [assert](https://nodejs.org/api/assert.html)
- [buffer](https://nodejs.org/api/buffer.html)
- [util](https://nodejs.org/api/util.html)
- [url](https://nodejs.org/api/url.html)
- [punycode](https://nodejs.org/api/punycode.html)
- [querystring](https://nodejs.org/api/querystring.html)
- [string-decoder](https://nodejs.org/api/string_decoder.html)
- [stream](https://nodejs.org/api/stream.html)
- [timers](https://nodejs.org/api/timers.html)
- [events](https://nodejs.org/api/events.html)

```javascript
// 外部库调用示例
const url = require('url');
const myURL = url.parse('https://sub.example.com:8080/p/a/t/h?query=string');

console.log(myURL.protocol, myURL.host, myURL.path);
```



## Mock服务

