# Reverse Engineered Nintendo Switch App REST API

## 介绍

这是用于Nintendo Switch应用程序及其内部Splatoon 2 Web应用程序的REST API的文档。

所有测试均在17/30/17上使用Nintendo Switch应用的1.0.4版本在运行iOS 10.3.3的iPhone 7上完成。我使用[mitmproxy](https://mitmproxy.org)对该应用进行了逆向工程。因为该应用程序没有使用证书固定，所以比较容易。我没有使用Android版的app进行过测试，但是我觉得应该没有什么差别（除了user-agent不一样）。我测试时使用的是一个语言设置为英语的美国帐户。使用其他地区的账户应该没有影响。

为macOS用户推荐一个叫[Paw](https://paw.cloud) 的软件，该项目应有助于调试API。我强烈建议您首先尝试一下以弄清楚API的工作原理。查看环境变量以查看需要填写的内容。填写“Client ID”，“Login Page Token Code”，“Login Page Token Code Verifier”和“Birthday”后，您可以执行身份验证请求为了秩序，你应该很好。

Note：建议将Splatoon 2 API的所有请求的User-Agent设置为`Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_3 like Mac OS X) AppleWebKit/603.3.8 (KHTML, like Gecko) Mobile/14G60`，虽然目前看来服务器并不对User-Agent做检测，但有备无患。

## 蓝图
* [Nintendo Account](NintendoAccountBlueprint.md)
* [Nintendo Switch](SwitchBlueprint.md)
* [Splatoon 2](Splatoon2Blueprint.md)

## 开源组件

如果您对应用程序中使用的开源组件感到好奇，我在[这里](OpenSource.md)对其进行了介绍。

## 验证步骤

1.在浏览器中访问授权链接

该页面是一个HTML页面，其中加载了您在登录应用程序时通常通常会首先看到的身份验证流。通过使用帐户登录来进行验证流程。

https://accounts.nintendo.com/connect/1.0.0/authorize?state=[state here]&redirect_uri=[... continues]

我目前不知道如何生成此URL。我建议退出Switch应用程序，然后重新登录并在Safari中打开标志流链接。然后，您可以在计算机上打开它，然后从那里开始。

登录后，您将被重定向到类似`npf71b963c1b7b6d119://auth#session_state=[SessionStateReturnedHere]&session_token_code=[codehere]&state=[StateReturnedHere]` 

2.获取会话令牌

从该网址中提取session_state和state，然后从 [POST /connect/1.0.0/api/session_token](NintendoAccountBlueprint.md#GetSessionToken)

3.获取服务访问令牌

使用从第二步中获得的session_token来发送request[POST /connect/1.0.0/api/token](NintendoAccountBlueprint.md#GetServiceToken)

4.登录账户

使用从第三步中获得的id_token来发送request[POST /v1/Account/Login](SwitchBlueprint.md#LoginAccount)

5.获取游戏列表

使用从第四步中获取的accesstoken来获取游戏列表 [GET /v1/Game/ListWebServices](SwitchBlueprint.md#GetGameList)

6.获取splatoon2的access token

使用从第四步中获取的Splatoon 2的ID和第四步中的`webApiServerCredential["accesstoken"]` [GET /v1/Game/GetWebServiceToken](SwitchBlueprint.md#GetGameWebServiceToken)

7.在splatoon的requests获取cookies

使用从第六步中获取的accessToken发送request[GET /](Splatoon2Blueprint.md#GetHomepage)

8.使用Splatoon 2的API

现在你可以使用从第七步中获得cookies来发送任意的api请求了 [Splatoon 2 API](Splatoon2Blueprint.md)
