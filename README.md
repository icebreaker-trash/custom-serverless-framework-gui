# custom-serverless-framework-cli

构建自定义的 serverless framework cli

tags: 定制化

优势显而易见，可以随意植入自己的逻辑

## 看源码

重点看 Serverless v2 components

可以看到四个包

- `@serverless/platform-client`
- `@serverless/platform-client-china`
- `@serverless/utils`
- `@serverless/utils-china`

`platform-client` 是 `sdk`
`utils` 里面放了许多的工具类和操作的 api 地址

这四个包虽然开源，却没有填写 Github 地址，然而我们还是可以从 `node_modules/@serverless` 文件夹中看到他的源码

通过

```js
new Intl.DateTimeFormat('en', { timeZoneName: 'long' })
  .format()
  .includes('China Standard Time')
```

这个布尔值判断是否在中国，然后进入不同的 `cli` 分支

所以我们结合自身情况，重点看 `@serverless/platform-client-china`

由于腾讯云获得了独家的合作？(猜想) 所以 china 的包,本质上都是腾讯云`@tencent-sdk/capi`接口的封装

> 2 个工具包中,都含有大量的对外接口信息,主要以2个域名为主
`apigw.tencentcs.com` 腾讯云api网关,上海为开发站,广州为正式站
`serverless-platform.com` serverless公司
<!-- ，腾讯云获得了独家的合作？ -->
<!-- , apigw -->

sdk初始化代码

```js
  // Load Instance Credentials
  // 帮你从环境变量中加载
  const instanceCredentials = await utils.loadInstanceCredentials(instanceYaml.stage);

  // initialize SDK
  const orgUid = await tencentUtils.getOrgId();
  const sdk = new ServerlessSDK({
    accessKey: tencentUtils.buildTempAccessKeyForTencent({
      SecretId: process.env.TENCENT_SECRET_ID,
      SecretKey: process.env.TENCENT_SECRET_KEY,
      Token: process.env.TENCENT_TOKEN,
    }),
    context: {
      orgUid,
      orgName: instanceYaml.org,
      traceId: uuidv4(),
    },
    agent: `ComponentsCLI_${version}`,
  });
```
基于 SDK 的另一种选择

[components](https://github.com/serverless/components)

看源码


```js

```

## serverless v1(Services) and v2(Components)

## 中国版本(腾讯云) serverless cli 为例

sls deploy 自动执行当前 cwd 下的 serverless.yml

serverless ⚡framework

Serverless 指令

- 您可以输入 “serverless“ 或简称 ”sls“
- 在 “serverless" 指令后输入 “--help” 获取使用帮助
  快速开始
- 直接输入 “serverless" (或缩写 “sls”) 进行项目初始化

serverless init {name} 通过 应用中心 或指定链接初始化一个模版项目
--debug 获取项目初始化过程中的详细信息
--name 自定义项目目录名称

serverless deploy 部署 Serverless 实例到云端
--debug 获取部署过程的详细信息
--target 部署该目录下指定 Serverless 实例
--inputs 增加实例部署参数
--profile 使用全局授权名称信息的密钥信息, 默认 'default'
--login 忽略全局认证信息

serverless info 获取并展示一个 Serverless 实例的相关信息
--profile 使用全局授权名称信息的密钥信息, 默认 'default'

serverless dev 启动 DEV MODE 开发者模式，支持在命令行中实时输出运行日志，同时支持对 Node.js 应用进行云端调试
--profile 使用全局授权名称信息的密钥信息, 默认 'default'

serverless remove 从云端移除一个 Serverless 实例
--debug 获取移除过程的详细信息
--target 移除该目录下指定 Serverless 实例
--profile 使用全局授权名称信息的密钥信息, 默认 'default'

serverless credentials 管理全局用户授权信息
set 存储用户授权信息
--secretId / -i (必填)腾讯云 CAM 账号 secretId
--secretKey / -k (必填)腾讯云 CAM 账号 secretKey
--profile / -n {name} 身份名称. 默认为 "default"
--overwrite / -o 覆写已有身份名称授权信息
remove 删除用户授权信息
--profile / -n {name} 身份名称. 默认为 "default"
list 查看用户授权信息

serverless registry 显示 应用中心 里的组件与模版信息
serverless registry {name} 显示 应用中心 里的指定组件或模版的详细信息

serverless publish 发布一个组件或模版到 应用中心

serverless bind role 重新为当前用户分配使用 Serverless 所需权限

当前命令行版本: v3.9.0

产品文档: https://cloud.tencent.com/document/product/1154
控制面板: https://serverless.cloud.tencent.com/
应用中心: https://registry.serverless.com/

我们最常用的肯定是
serverless deploy (部署)
serverless info (查看详情)
serverless dev (单实例调试)
serverless remove (移除)

## 探索

cli 其实并不方便，它和 nodejs 进行通信不是很友好

### 之前的一种麻烦的思路(stdout)

一种思路是 nodejs 的 child_process 思路 , 创建子进程去执行命令

比如 exec

```shell
serverless deploy // --no-color
```

然后在 nodejs 里去获取它的 stdout 进行解析

当然结果往往是一大堆的 loading.... ，中间一段成功 yaml ,和一个 footer result

而且由于结果中，带了一些命令行的颜色信息，往往我们需要使用 `strip-ansi` 主动的把这些东西干掉

不得不说这种思路，容错率很低,一有改动很容易挂

### SDK 方案

这个方案，[yugasun](https://github.com/yugasun) 大佬的 [用例](https://github.com/serverless-components/tencent-framework-components/blob/master/scripts/example.ts)就已经开始使用了，我也是从他这学到很多东西。

官方提供了 2 个 sdk，`@serverless/platform-client`, `@serverless/platform-client-china`

因为我们是中国人，所以我们就用 `platform-client-china`

这个默认是部署到腾讯云上的

开始吧
