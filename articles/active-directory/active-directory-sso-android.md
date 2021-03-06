<properties
	pageTitle="如何使用 ADAL 在 Android 上启用跨应用 SSO | Azure"
	description="如何使用 ADAL SDK 的功能跨应用程序启用单一登录。"
	services="active-directory"
	documentationCenter=""
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/16/2016"
	wacn.date="07/26/2016"/>


# 如何使用 ADAL 在 Android 上启用跨应用 SSO


提供单一登录 (SSO)，以便用户只需一次输入其凭据并使这些凭据自动跨工作是现在所需的客户应用程序。在小屏幕上，通常时间加上的其他因素 (2FA)，如电话呼叫或发送短代码中，输入其用户名和密码的难度导致快速不满，如果用户必须执行此操作一次以上为你的产品。

此外，如果你利用其他应用程序可以使用的标识平台（例如 Microsoft 帐户或 Office365 中的工作帐户），则客户希望不管供应商是谁，他们都可以在所有应用程序中使用这些凭据。

Microsoft 标识平台以及 Microsoft 标识 SDK 能够为你完成所有这些繁琐的工作，并可让你使用 SSO 在自己的应用程序套件中让客户感到满意，或者使用中转站功能和验证器应用程序在整个设备上提供良好的用户体验。

本演练将介绍如何在 SDK 中配置应用程序，以便向客户提供此项优点。

本演练适用于：

* Azure Active Directory
* Azure Active Directory B2C
* Azure Active Directory B2B。请注意，以下文档假设你已了解如何[在旧版门户中为 Azure Active Directory 预配应用程序](/documentation/articles/active-directory-how-to-integrate/)，并且你已将应用程序与 [Microsoft Identity Android SDK](https://github.com/AzureAD/azure-activedirectory-library-for-android) 集成。

## Microsoft 标识平台中的 SSO 概念

### Microsoft 标识中转站

Microsoft 提供了为来自不同供应商的应用程序之间过渡凭据允许为每个移动平台的应用程序，以及允许特殊的增强功能，需要一个安全的位置，从何处来验证凭据。我们称之为“中转站”。在 iOS 和 Android 提供这些方法通过可下载的应用程序的客户可以独立安装或由公司负责管理其员工的部分或全部设备推送到设备。这些中转站只是为了某些应用程序或整个根据 IT 管理员所需的设备支持安全管理。在 Windows 内置于操作系统，已知技术作为 Web 身份验证中转站帐户选择器提供此功能。

为了理解我们如何使用这些中转站以及你的客户如何在 Microsoft 标识平台的登录流中发现它们的用途，请继续阅读以了解详细信息。

### 在移动设备上登录的模式

在设备上访问凭据遵循 Microsoft 标识平台的两种基本模式：

* 非中转站辅助的登录
* 中转站辅助的登录

#### 非中转站辅助的登录

非中转站的辅助登录是应用程序内联的登录体验，使用该应用程序的设备上的本地存储。此存储可以跨应用程序共享，但凭据紧密绑定到应用或使用该凭据的应用套件。这是最有可能过许多移动应用程序中的体验，其中输入用户名和密码在应用程序本身中。

这种登录具有以下优点：

-  完全在应用程序存在的用户体验。
-  凭据可跨应用程序共享并由同一个证书签名，为应用程序套件提供单一登录体验。 
-  围绕体验中的日志记录的控件提供给应用程序之前和之后登录。

这种登录具有以下缺点：

- 用户不能在使用 Microsoft 标识的所有应用上体验单一登录，而只能在使用应用程序拥有并且已配置的 Microsoft 标识的应用上体验单一登录。
- 应用程序不可以用于更高级的业务功能，如条件性访问，或使用 InTune 产品套件。
- 应用程序不能使业务用户支持基于证书的身份验证。

下面介绍了 Microsoft 标识 SDK 如何与应用程序的共享存储配合工作以启用 SSO：


	+------------+ +------------+  +-------------+
	|            | |            |  |             |
	|   App 1    | |   App 2    |  |   App 3     |
	|            | |            |  |             |
	|            | |            |  |             |
	+------------+ +------------+  +-------------+
	| Azure SDK  | | Azure SDK  |  | Azure SDK   |
	+------------+-+------------+--+-------------+
	|                                            |
	|            App Shared Storage              |
	+--------------------------------------------+


#### 中转站辅助的登录

中转站辅助的登录在中转站应用程序中发生，使用存储和中转站的安全性在设备上的所有应用程序利用 Microsoft 身份平台之间共享的凭据登录体验。这意味着，应用程序将依赖于中转站才能将用户登录。在 iOS 和 Android 提供这些方法通过可下载的应用程序的客户可以独立安装或由公司负责管理其用户的设备推送到设备。这种类型的应用程序示例是 iOS 上的 Azure 验证器应用程序。在 Windows 内置于操作系统，已知技术作为 Web 身份验证中转站帐户选择器提供此功能。体验因平台而异，如果未正确管理，有时会给用户带来麻烦。如果脸已安装 Facebook 应用程序并在另一个应用程序中使用 Facebook 登录功能，你可能很熟悉这种模式。Microsoft 标识平台利用相同的模式。

对于 iOS，这会导致“过渡”动画，其中，应用程序将发送到后台，而 Azure 验证器应用程序将发送到前台，让用户选择他们登录时使用的帐户。

对于 Android 和 Windows 帐户选择器将在对用户中断较小的基础上显示应用程序。

#### 如何调用中转站

如果在设备上安装了 Azure 验证器应用程序等兼容的中转站，当用户指出他们希望在 Microsoft 标识平台上使用任何帐户登录时，Microsoft 标识 SDK 将自动为你调用中转站。这可能是个人 Microsoft 帐户、工作或学校帐户或你提供和使用我们 B2C 和 B2B 的产品在 Azure 中托管的帐户。通过使用极高的安全性的算法和加密我们确保要求提供的凭据并且发送回应用程序以安全的方式。这些机制的确切的技术详细信息未发布，但由 Apple 和 Google 开发与协作。

**开发人员可以选择是由 Microsoft 标识 SDK 调用中转站，还是使用非中转站辅助的流。** 但是如果开发人员选择不使用中转站辅助的流，他们会丢失利用 SSO 凭据，用户可能已添加在设备上，以及阻止与 Microsoft 提供的条件性访问 Intune 管理功能，例如其客户的业务功能一起使用其应用程序和基于证书的身份验证的好处。

这种登录具有以下优点：

-  无论供应商是谁，用户都可以在所有应用程序中体验 SSO。
-  应用程序可以利用更高级的业务功能，如条件性访问，或使用 InTune 产品套件。
-  应用程序可以使业务用户支持基于证书的身份验证。
- 登录体验要安全得多，因为中转站应用程序将使用附加的安全算法和加密来验证应用程序与用户的标识。

这种登录具有以下缺点：

- 在 iOS 中用户时选择了凭据，则利用应用程序体验转换。
- 管理你的客户应用程序中的登录体验的能力的丢失。



下面介绍了 Microsoft 标识 SDK 如何与应用程序的中转站应用程序配合工作以启用 SSO：
	
	
	+------------+ +------------+   +-------------+
	|            | |            |   |             |
	|   App 1    | |   App 2    |   |   Someone   |
	|            | |            |   |    Else's   |
	|            | |            |   |     App     |
	+------------+ +------------+   +-------------+
	| Azure SDK  | | Azure SDK  |   | Azure SDK   |
	+-----+------+-+-----+------+-  +-------+-----+
	      |              |                  |
	      |       +------v------+           |
	      |       |             |           |
	      |       | Microsoft   |           |
	      +-------> Broker      |^----------+
	              | Application
	              |             |
	              +-------------+
	              |             |
	              |   Broker    |
	              |   Storage   |
	              |             |
	              +-------------+

              
了解这些背景信息后，你应该可以更好地理解 SSO 并使用 Microsoft 标识平台和 SDK 在应用程序中实现它。


## 使用 ADAL 启用跨应用 SSO

这里，我们将使用 ADAL Android SDK 来执行以下操作：

- 为应用套件打开非中转站辅助的 SSO
- 打开对中转站辅助 SSO 的支持


### 打开对非中转站辅助 SSO 的 SSO

对于跨应用程序的非中转站辅助 SSO，使用 Microsoft 标识 SDK 可以大大消除 SSO 的复杂性。这包括在缓存中查找适当的用户，以及维护登录用户的列表供你查询。

若要跨你拥有的应用程序启用 SSO，需要执行以下操作：

1. 确保所有应用程序使用相同的客户端 ID 或应用程序 ID。 
* 确保所有应用程序具有相同的 SharedUserID 集。
* 确保所有应用程序共享来自 Google Play Store 的相同签名证书，以便可以共享存储。

#### 步骤 1：对应用套件中的所有应用程序使用相同的客户端 ID/应用程序 ID

为了让 Microsoft 标识平台知道你可以跨应用程序共享令牌，每个应用程序需要共享同一个客户端 ID 或应用程序 ID。这是在门户中注册第一个应用程序时为你提供的唯一标识符。

如果应用使用相同的应用程序 ID，你可能想要知道如何在 Microsoft 标识服务中标识不同的应用。答案是使用“重定向 URI”。每个应用程序可以在登记门户中注册多个重定向 URI。套件中的每个应用程序具有不同的重定向 URI。下面显示了这种情况的示例：

应用 1 重定向 URI：`msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D`

应用 2 重定向 URI：`msauth://com.example.userapp1/KmB7PxIytyLkbGHuI%2UitkW%2Fejk%4E`

应用 3 重定向 URI：`msauth://com.example.userapp2/Pt85PxIyvbLkbKUtBI%2SitkW%2Fejk%9F`

....

这些应用嵌套在同一个客户端 ID/应用程序 ID 下，可以根据你在 SDK 配置中返回给我们的重定向 URI 来查找。

	
	+-------------------+
	|                   |
	|  Client ID        |
	+---------+---------+
	          |
	          |           +-----------------------------------+
	          |           |  App 1 Redirect URI               |
	          +----------^+                                   |
	          |           +-----------------------------------+
	          |
	          |           +-----------------------------------+
	          +----------^+  App 2 Redirect URI               |
	          |           |                                   |
	          |           +-----------------------------------+
	          |
	          +----------^+-----------------------------------+
	                      |  App 3 Redirect URI               |
	                      |                                   |
	                      +-----------------------------------+
	



请注意，下面介绍了这些重定向 URI 的格式。你可以使用任何重定向 URI，除非你想要支持中转站，在这种情况下，它们必须如上所示


#### 步骤 2：在 Android 中配置共享存储

设置 `SharedUserID` 超出了本文的范围，但你可以阅读有关[清单](http://developer.android.com/guide/topics/manifest/manifest-element.html)的 Google Android 文档来了解具体操作。重要的是，你需要决定 sharedUserID 的调用方式，并在所有应用程序中使用它。

在所有应用程序中设置 `SharedUserID` 后，你便可以使用 SSO。

> [AZURE.WARNING] 
在应用程序之间共享存储之后，任何应用程序都可以删除用户，更糟的是，删除整个应用程序的所有令牌。如果你的应用程序依赖于这些令牌来执行后台工作，则这是特别严重的后果。要共享存储，就必须十分警惕通过 Microsoft 标识 SDK 执行的任意和所有删除操作。

就这么简单！ Microsoft 标识 SDK 现在将在所有应用程序之间共享凭据。此外还将在应用程序实例之间共享用户列表。

### 打开对中转站辅助 SSO 的 SSO

应用程序使用安装在设备上的任何中转站的功能默认情况下已关闭。若要向中转站使用应用程序必须执行一些额外配置，并将一些代码添加到你的应用程序。

要遵循的步骤如下：

1. 启用应用程序代码调用到 MS SDK 中的中转站模式
2. 建立新的重定向 URI，并为应用程序和应用程序的注册提供程序
3. 在 Android 清单中设置正确的权限


#### 步骤 1：在应用程序中启用中转站模式
应用程序使用了中转站的功能被打开的当你创建的设置或身份验证的实例的初始设置。通过在代码中设置 ApplicationSettings 类型中执行此操作：


	AuthenticationSettings.Instance.setUseBroker(true);

#### 步骤 2：使用 URL 方案建立新的重定向 URI

为了确保我们始终返回凭据令牌到正确的应用程序，我们需要确保我们回调到你的应用程序可以验证 Android 操作系统的方式。Android 操作系统使用 Google Play Store 中的证书哈希。此证书不能受到恶意应用程序的欺骗。因此，我们会利用这以及中转站应用程序以确保令牌返回到正确的应用程序的 URI。我们要求你在你的应用程序中建立此唯一重定向 URI 这两个并为我们的开发人员门户中的重定向 URI 进行设置。

重定向 URI 必须采用正确的格式：

`msauth://packagename/Base64UrlencodedSignature`

例如：msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D

需要使用 [Azure 经典门户](https://manage.windowsazure.cn/)在应用注册中指定此重定向 URI。有关 Azure AD 应用注册的详细信息，请参阅[与 Azure Active Directory 集成](/documentation/articles/active-directory-how-to-integrate/)。


#### 步骤 3：在应用程序中设置正确的权限

Android 中的中转站应用程序使用 Android OS 的帐户管理器功能来管理跨应用程序的凭据。若要在 Android 中使用该中转站应用程序清单中必须有权使用 AccountManager 帐户。[Google 帐户管理器文档](http://developer.android.com/reference/android/accounts/AccountManager.html)中作了详细介绍。

具体而言，这些权限如下：

		
	GET_ACCOUNTS
	USE_CREDENTIALS
	MANAGE_ACCOUNTS


### 你已配置 SSO！

现在，Microsoft 标识 SDK 将自动在应用程序之间共享凭据并调用中转站（如果在设备上存在）。
<!---HONumber=Mooncake_0718_2016-->