# 基于internlm2.5-7b-chat的dify生成农业知识问答并导出飞书文档工作流

## 教程简介

### 本教程将会学到什么？

1. dify 工作流搭建
2. 熟悉如何在dify里调用飞书官方API

### 本教程流程图

![1724493232610](https://github.com/user-attachments/assets/048aeb41-c9a5-434f-a514-6b8ea02ef008)


## 1.（准备工作） dify配置Internlm2.5

进入硅基流动官网：https://cloud.siliconflow.cn/s/internlm

点击“API密钥”复制密钥 

![1724153564787](https://github.com/user-attachments/assets/6e31de94-c3e7-46ac-a651-e748dd3ca2d7)

点击“头像”，点击“设置”,“模型供应商”里选择“SILICONFLOW”,点击“设置”，填入api key,模型列表可以看到internlm 2.5

![image](https://github.com/user-attachments/assets/8a83bb6d-ad60-4df0-9715-e1254dd3ae9e)
![image](https://github.com/user-attachments/assets/5aa9331d-955d-45b4-9ca9-c4f6cf045c21)
![image](https://github.com/user-attachments/assets/235927bb-7035-4f44-b71c-aadb7f2e83f3)
![image](https://github.com/user-attachments/assets/7053ce1c-1b92-4817-861e-2a7323ec644d)

## 2.（准备工作）飞书创建应用

进入[飞书开放平台](https://open.feishu.cn/?lang=zh-CN)，点击“开发文档”，选择“服务端API”，进行文档查看。

![1724374702192](https://github.com/user-attachments/assets/1a26e006-b1fb-4e74-9285-2ba197f04e1d)

![1724374997998](https://github.com/user-attachments/assets/99172e17-c425-4f2d-a4a5-8021b386e82c)

### 2.1 飞书API调用流程概述

飞书开放平台提供的 API 遵循 RESTful 风格，请求 URL 的格式为：

![image](https://github.com/user-attachments/assets/407a079e-7e18-4fca-bb64-319632ce8d27)

调用服务端 API 的流程如下图所示。

![image](https://github.com/user-attachments/assets/d21c6d7e-5324-4c82-9225-65093fbb6b08)

基于我们的需求，4个流程（除了设置白名单、配置应用数据）就够了。每个流程都做了超链接，页面点击跳转即可。

![1724375698744](https://github.com/user-attachments/assets/07c60146-7f71-4b52-90d8-1b6f99486b09)

### 2.2 创建应用

点击[开发者后台](https://open.feishu.cn/app)，进行【创建应用】。

![1724378554198](https://github.com/user-attachments/assets/6b159f09-6b83-4974-8903-447b491f2230)

创建完成会进入到应用后台，此次教程主要用到“凭证与基础信息”、“权限管理”、“日志检索”。

- 凭证与基础信息：用于生成Token鉴权
- 权限管理：用于设置API权限、数据权限
- 日志检索：查看调用日志，在外部调用接口日志会保留在这里，排错时可以查看。

![1724378684422](https://github.com/user-attachments/assets/f0392443-5fdd-4de5-88fa-3cb044eec7a6)

### 2.3 了解飞书访问凭证

#### 2.3.1 访问凭证介绍

飞书开放平台提供以下不同类型的访问凭证，用于对调用方身份进行鉴权。在实际开发过程中，你可以根据业务需要选择适用的访问凭证。

| 访问凭证类型            | 是否需要用户授权 | 说明                                                                 |
|-------------------------|------------------|----------------------------------------------------------------------|
| **tenant_access_token** | 否               | 以应用身份调用 API 时需要使用的凭证，可读写的数据范围由应用的[数据权限范围](https://open.feishu.cn/document/ukTMukTMukTM/uMDNyYjLzETO14yM3ATN)决定。该类凭证的值以 `t-` 为前缀，示例值：`t-24b5bf4e00b2af1234`。 |
| **user_access_token**   | 是               | 以用户身份调用 API 时需要使用的凭证，可读写的数据范围由用户可读写的数据范围决定。该类凭证的值以 `u-` 为前缀，示例值：`u-Lr1RT7S8fS03rn1234`。 |
| **app_access_token**    | 否               | 应用身份的短期令牌。开放平台根据 app_access_token 识别调用方的应用身份。该类凭证的值以 `a-` 或者 `t-` 为前缀，示例值：`a-24b5cef00b1234`。 |

#### 2.3.2 访问凭证选择

在调用 API 时，访问凭证需要填充到 HTTP 请求头中，提供两种访问凭证选择：tenant_access_token、user_access_token。阅读两者区别可以发现一个是操作应用空间，一个操作用户空间。本教程要实现的是在【用户空间】下【新建云文档】并填入内容，所以凭证选择【user_access_token】。获取访问凭证先暂停一下，我们先去dify创建工作流。

![1724379448950](https://github.com/user-attachments/assets/0f3bb3d4-9682-46bd-8e14-b74696550351)

## 3. dify工作流

点击[dify](https://cloud.dify.ai/apps)，需要科学上网~

### 3.1 新建工作流

点击“工作室”----“创建空白应用”----“选择工作流”

![1724382126100](https://github.com/user-attachments/assets/b9c65788-66ce-4b54-847a-0e3b50b18db0)

![1724382219922](https://github.com/user-attachments/assets/489b84a5-832d-4d65-bdc9-4a42d757abc7)

### 3.2 internLM2.5 生成问答

#### 3.2.1 新建LLM节点

主要写了一个关于农业知识问答的prompt，所以在“开始”节点不做其他输入。直接新建一个【LLM节点】。

![1724382318252](https://github.com/user-attachments/assets/b1b10bec-de9f-4218-a45a-3a8f20dbc376)

#### 3.2.2 配置LLM节点

第一步：选模型。选完模型会多出"system"来配置提示词，这里我们还需要"添加信息"，配置"user"和"assistant"的提示词。

![1724382517854](https://github.com/user-attachments/assets/86f7ad36-0852-423e-9f04-ce72e5a30acb)

更改名字，方便后面阅读。完整的配置如下图所示。

![1724383169786](https://github.com/user-attachments/assets/6cd95cf8-dc97-40a3-acfc-ed5de30e6501)

##### system提示词

```Plain
- Role: 你是一位植物种植专家，专注于提供针对具体种植问题的详细解决方案。
- Background: 你具备丰富的植物种植经验和知识，能够对各种植物的种植方法给予专业解答。
- Profile: 你的任务是提供的种植问题并回答。
- Skills: 植物种植知识、解决问题能力、专业解答。
- Goals: 根据用户的请求，生成一个具体的植物种植问题示例，确保每次只生成一个问题，并针对此问题进行回答。
- Constrains: 生成单个具体的植物种植问题示例，每次仅生成一个问题，并回答。
- OutputFormat: 生成一个具体的植物种植问题示例。
```

##### user提示词

```Plain
用户希望获得一个具体的植物种植问题示例。请生成一个单一且具体的种植问题，并回答它。
```

##### assiatant提示词

```Plain
用户需要一个具体的植物种植问题示例和针对问题的回答。请确保每次只生成一个单独的种植问题，例如“如何种植西瓜？”确保问题清晰且具体，并针对生成的问题，准确回答。
```

#### 3.2.3 调试“生成问答”工作流

点击配置页面【右上角】，运行此步骤。

![1724383257134](https://github.com/user-attachments/assets/8b15c842-f1ea-409d-bb04-33f054fd933a)

运行结果如下图所示，我们可以看到它把“问题”和“答案”统一包裹在"text"字段，不符合我们想要的。下一步我们要把“问题”、“答案”提取出来。

![1724383307484](https://github.com/user-attachments/assets/bdf4b106-3d81-4741-9321-63d7c5507290)

### 3.3 提取“问题”和“答案”并格式输出

新建一个【参数提取器】并配置。

![image](https://github.com/user-attachments/assets/f3e8f9f9-41ae-4b82-a24c-c22b15629ec7)


第一步：【输入变量】点击选择【生成问答】节点返回的【text】字段。

第二步：选择internlm2.5模型

第三步：设置【提取参数】。这里设置名称"conversation",类型选择Array[Object]。"conversation"就会作为输出字段。

![1724392062601](https://github.com/user-attachments/assets/78535e02-59bf-4afb-93de-a6d0545b4325)


第四步：输入【指令】。注意：下面指令中的"text"，请删除并手动输入反斜杠(/)选择LLM输出的text。

```Plain
将"text"字段内的“问题”作为input，“回答”作为output
Example: 
{
  "conversation": [
   {
      "input": "如何种植番茄以获得最佳的产量和品质？",
      "output": "种植番茄以获得最佳产量和品质需要细心规划和适当的管理。以下是详细的种植步骤和注意事项：\n\n1. 选择合适的品种：根据您的地理位置、气候条件和种植目的选择合适的番茄品种。一些受欢迎的品种包括樱桃番茄、大番茄和长形番茄。\n\n2. 准备土壤：番茄需要排水良好、富含有机物的土壤。在种植前，确保土壤经过适当的耕作和肥料处理，以提高土壤的肥力和结构。\n\n3. 播种或移栽：番茄可以通过播种或移栽幼苗的方式种植。播种通常在春季进行，而移栽则可在春季末期或秋季进行。保证幼苗或种子之间的距离合适，以便于生长和通风。\n\n4. 浇水和施肥：番茄需要充足的水分和适当的营养。保持土壤湿润但不要积水是关键。使用有机肥料或均衡的化学肥料来提供必要的营养。\n\n5. 支持植物生长：番茄植株可能需要支持以防止倒伏。使用笼子、木棍或绑绳等方式为植株提供支撑。\n\n6. 病虫害管理：定期检查植物，注意常见的病虫害，如蚜虫、白粉病和灰霉病，并采取相应的防治措施。\n\n7. 收获：当番茄果实完全成熟并变成您品种特有的颜色时，就可以收获了。小心地将番茄从植株上摘下，避免损坏其他果实。\n\n通过遵循这些步骤和注意事项，您可以大大提高种植番茄的产量和品质。"
   }
  ]
}
```

配置完成如图所示。
![1724392109010](https://github.com/user-attachments/assets/838d110d-a7fc-4aae-8164-7d23c039037b)

####   调试结果

最终调试结果如图所示。
![007f14328759d8c4bb6e487891429c9](https://github.com/user-attachments/assets/50cb56f1-7859-4f4b-85c0-4846313b4fe5)

### 3.4 数组转文本

上一个节点给我们返回的是一个数组，我们在插入文档时要求的是“文本”类型，这里我们用 python3 语法做一个数组转文本处理。

新建【代码执行】节点，配置参数，并复制代码。

配置截图

![1724491856514](https://github.com/user-attachments/assets/7f478339-06a8-47f1-bf97-49b17cfbf661)

```Plain
def main(conversion: list) -> dict:
    conversion_str = str(conversion)
    return {
        "result": conversion_str
    }
```


### 3.5 获取飞书授权

#### 3.5.1 获取飞书授权登录授权码

#### 选择指定应用
登录[开发者后台](https://open.feishu.cn/app)，选择指定应用。

#### 配置重定向url

在【开发配置】----【安全设置】页面配置重定向URL。dify地址：https://cloud.dify.ai/

![1724408104985](https://github.com/user-attachments/assets/6d2163cc-ae45-490f-9537-a56040cd8d47)

#### 获取授权登录授权码

调用[获取授权登录授权码]([https://open.feishu.cn/app](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/authen-v1/authorize/get))，获取登录预授权码【code】。

前置准备
>1. 记下【新建及编辑云文档】的scope值
>2. 开通【新建及编辑云文档】权限
>3. 发布版本
>4. https://cloud.dify.ai/ 的url编码 #这个我会直接在下面贴给大家

我们需要授权新建及编辑云文档的权限，需要填写它的scope值。点击[API 权限列表](https://open.feishu.cn/document/server-docs/application-scope/scope-list)，搜索到相对接口。图上标出的就是scope值。

![1724405093850](https://github.com/user-attachments/assets/50bde467-0d38-42d1-8194-37681c9cffe4)

【新建及编辑云文档】权限开通一下。

点击左侧列表，“云文档”----“文档”----“创建文档”----右侧选择“权限配置”，勾选并开通权限。

![1724404582234](https://github.com/user-attachments/assets/5923db34-b339-4d5c-b474-b19313826413)

配置完成后一定要发布版本！！！！！

![1724408161370](https://github.com/user-attachments/assets/7d1d215e-74f4-4119-9044-145f1c275e8b)

现在我们准备工作已经完成，列出我们需要的详细值，我会给大家贴出命令，直接替换掉APP ID：

>1. App ID  #登录[开发者后台](https://open.feishu.cn/app)，选择指定应用内的【凭证与基础信息】
>2. dify url编码：https%3A%2F%2Fcloud.dify.ai%2F
>3. scope值：docx:document
>4. state值：RANDOMSTATE

复制并【替换】你的【APP ID】到浏览器
```Plain
https://open.feishu.cn/open-apis/authen/v1/authorize?app_id=【替换】&redirect_uri=https%3A%2F%2Fcloud.dify.ai%2F&scope=docx:document&state=RANDOMSTATE
```

会跳转应用授权页面，进行授权

![image](https://github.com/user-attachments/assets/ec72fc03-a637-4763-b3d8-91ca690b7fdd)

授权成功后跳转到dify页面。地址里已经包含我们要用的code了，记录下这个code。用于换user_access_token。

![1724410783686](https://github.com/user-attachments/assets/dfc9b707-9126-457f-8b16-fc14e8b69869)

#### 3.5.2 获取user_access_token

##### 先自建应用获取app_access_token

只用到了 APP ID和APP secret

![1724412275292](https://github.com/user-attachments/assets/3bc82f8b-8e03-448e-9246-4831956fbfdc)

回到dify工作流，我们新建一个【HTTP请求】节点，依次填入最后点击节点测试。

![1724412506988](https://github.com/user-attachments/assets/65cd0ae4-81a8-40a2-b75a-e817939d722a)

运行结果如图所示，已经有我们要的app_access_token了

![9ec3a57a0a456c99942e18c94740062](https://github.com/user-attachments/assets/9500d0fa-f23c-46b5-b672-693cee971e65)

接下来我们就要拿着app_access_token去获取 user_access_token

##### 用app_access_token获取 user_access_token

新建一个【参数提取】节点，用来提取app_access_token传递给下个【HTTP请求】节点。和前面一样的操作。仍然要新建一个“提取参数”，并给出指令。

指令
```Plain
提取出app_access_token
```

![1724412801907](https://github.com/user-attachments/assets/1e44bcca-8723-4bd1-b241-596f2887bf08)

![1724412922774](https://github.com/user-attachments/assets/69cd3f8f-1145-4e46-bbb9-cbeeeb88c098)

测试结果：

![1724413145763](https://github.com/user-attachments/assets/a4fc1592-e301-4e00-8e7d-05447e858cbe)


新建一个【HTTP请求】节点，用app_access_token获取 user_access_token。继续参照开发文档给的接口。[获取user_access_token文档链接](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/authen-v1/oidc-access_token/create?appId=cli_a64d51f6a179d00e)

![1724413246502](https://github.com/user-attachments/assets/6d8d256a-7302-4594-b40d-77bc45f966fd)

注意请求头“Authorization”的值。Bearer后有一个空格，直接/选取参数提取器返回的结果。

![1724413351567](https://github.com/user-attachments/assets/b3537015-c292-4324-9638-b4fc7f7bcf42)

群组配置机器人

![image](https://github.com/user-attachments/assets/3f4b0631-dba3-4818-9181-bbffd8cb11d1)


因为code只有5分钟时效，所以我们要再次进行授权，拿到code。

复制并【替换】你的【APP ID】到浏览器
```Plain
https://open.feishu.cn/open-apis/authen/v1/authorize?app_id=【替换】&redirect_uri=https%3A%2F%2Fcloud.dify.ai%2F&scope=docx:document%20drive:drive%20docs:permission.setting:write_only&state=RANDOMSTATE
```

配置完的截图如下：

![1724413842170](https://github.com/user-attachments/assets/61ac700a-4e2d-4d50-a18e-07d1336b9f53)

测试结果如图所示：
![image](https://github.com/user-attachments/assets/7671b2d9-7e99-4336-8eb1-28512600b7f3)

### 3.6 写入飞书云文档

#### 3.6.1 文档概述

先来看一段[官方文档](https://open.feishu.cn/document/server-docs/docs/docs/docx-v1/docx-overview)

> 关于文档
> ![1724469127076](https://github.com/user-attachments/assets/4a436145-7035-48eb-be7c-8c9da56ff6d7)

> 关于块
> 
> 在一篇文档中，有多个不同类型的段落，这些段落被定义为块（Block）。块是文档中的最小构建单元，是内容的结构化组成元素，有着明确的含义。块有多种形态，可以是一段文字、一张电子表格、一张图片或一个多维表格等。每个块都有唯一的 block_id 作为标识。
> 每一篇文档都有一个根块，即页面块（Page block）。页面块的 block_id 与其所在文档的 document_id 相同。在数据结构中，文档的页面块与其它块形成父子关系，页面块为父块，称为 Parent，其它块为子块，称为 Children。其它块之间也可形成父子关系

![image](https://github.com/user-attachments/assets/be7daa21-a7d0-4042-bc38-7b00d9df370b)

![image](https://github.com/user-attachments/assets/202ee9cf-90d9-4b2c-993b-06bad6af6226)

通过文档，我们可以得知2个信息：
- 首先有个根块(page),它和document_id一样
- 想插入文本，先得新建一个text块

#### 3.6.2 应用授权为文档协助者

第一步：把新建的空白文档在【浏览器】里打开。点击右上角【...】

![1724487332559](https://github.com/user-attachments/assets/0cb34e25-69cd-4706-b1db-b5444bc31b0e)

第二步：选择“更多”----“添加文档应用”----输入应用名----添加

![1724487671226](https://github.com/user-attachments/assets/56880b38-a527-45b1-a23f-f5dc518e1776)

![1724487544973](https://github.com/user-attachments/assets/f738ecf0-b753-4aa9-a518-7b88d33d8a9c)

![1724487583073](https://github.com/user-attachments/assets/cd8f932f-f918-4ca0-a513-cc8d82ef6920)


#### 3.6.3 新建块并写入内容

阅读文档可以看到我们符合请求“创建块”的条件。

![1724478364185](https://github.com/user-attachments/assets/54b821d8-06ce-48c7-840d-c72172456970)

根据[创建块文档](https://open.feishu.cn/document/server-docs/docs/docs/docx-v1/document-block/create)，新增一个【HTTP】请求节点，进行配置。

先用先前在网页打开的“0824”文档的document_id做节点测试，测试效果如下，code:200成功，我们去看下文档也成功写入了：

![1724487907716](https://github.com/user-attachments/assets/9fac0e61-f8ec-46d6-94b5-0d7792d9a6ea)

![1724487952732](https://github.com/user-attachments/assets/76ab9a15-e83a-4bbf-be66-4e0532caca01)

接下来完整得跑一下整个工作流。这里注意，要先替换下code。

复制并【替换】你的【APP ID】到浏览器,拿到code,重新配置【获取user_access_token的请求器】的code。
```Plain
https://open.feishu.cn/open-apis/authen/v1/authorize?app_id=【替换】&redirect_uri=https%3A%2F%2Fcloud.dify.ai%2F&scope=docx:document&state=RANDOMSTATE
```

最终结果：

![1724488343001](https://github.com/user-attachments/assets/64d1a9d2-85ae-4644-8414-6efe311fd9c2)


### 3.7 数据处理



## 4. （选看）删除不需要的应用

### 4.1 关闭应用

进入[飞书管理后台](https://f0u6olb9y9.feishu.cn/admin/index)，点击“工作台”----选择“应用管理”----筛选开发者----点击“配置”----关闭右上角的“启用”。
![image](https://github.com/user-attachments/assets/673a2ce0-7707-4cfe-bf78-9fa43e024ba5)
![1724378206108](https://github.com/user-attachments/assets/e858bb77-1507-4a47-a6de-6986e8463a24)

### 4.2删除应用

回到[飞书开放平台](https://open.feishu.cn/app?lang=zh-CN)，点击进入【已停用】的应用，点击“删除应用”。

![1724378441102](https://github.com/user-attachments/assets/c5fe54e5-de93-4757-8232-af6d0a8e68b2)

## 5. 下期教程预告

### (coze or dify) AI面试官-企业级招聘面试分析全流程系统
实现自动化面试问答及候选人能力分析，提高招聘效率和准确性。

### (coze) 基于notion连接器打造个人助理
