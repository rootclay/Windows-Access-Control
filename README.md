# index

> 原文地址：[https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control)

\[TOC\]

## 介绍

**访问控制**是指控制谁可以访问操作系统中资源的安全功能。 应用程序调用访问控制功能来设置谁可以访问特定资源，或者控制对应用程序提供的资源的访问。

此概述描述了用于控制对Windows对象（例如文件）的访问，以及用于控制对管理功能（例如设置系统时间或审核用户的操作）的访问的安全模型。 访问控制模型主题提供了访问控制各部分以及它们之间如何交互的高级描述。

## 访问控制模型

访问控制模型提供了可以控制进程访问安全对象或执行各种系统管理任务的能力。

### 访问控制模型的各个部分

访问控制模型有两个基本部分：

1. 访问令牌（Access tokens）：包含了有关已登录用户的信息（与特定的windows账户关联）

> 用户登录时，系统会验证用户的帐户名和密码。如果登录成功，系统将创建一个访问令牌，用户通过使用令牌的副本去创建和访问进程。访问令牌包含安全标识符，这些标识符标识用户帐户以及用户所属组。令牌还包含用户或用户组所拥有的权限列表。当进程尝试访问安全对象或执行需要特权的系统管理任务时，系统将使用此令牌来标识关联的用户是否拥有相应的权限。

1. 安全描述符（Security descriptors）：包含了保护安全对象的安全信息（与被访问对象关联）

> 创建安全对象后，系统会为其分配安全描述符，该描述符包含由其创建者指定的安全信息，如果未指定，则为默认安全信息。应用程序可以使用函数来检索和设置现有对象的安全性信息。

安全描述符标识指出对象的所有者，并且还可以包含以下访问控制列表：

1. discretionary access control list（DACL）: 用于标识哪些用户和组对目标对象有访问权限
2. system access control list（SACL）: 用于记录对安全对象访问的日志

ACL包含访问控制项（access control entries）（ACEs）的列表。每个ACE指定一组访问权限，并包含一个SID，用于标识其权限被允许、拒绝或审核的受托者。受托者可以是用户帐户、组帐户或登录会话。

Windows访问控制流程图： 当一个线程尝试去访问一个对象时，系统会检查线程持有的令牌以及被访问对象的安全描述符中的DACL。 如果安全描述符中不存在DACL，则系统会允许线程进行访问。如果存在DACL，系统会顺序遍历DACL中的每个ACE，检查ACE中的SID在线程的令牌中是否存在。以访问者中的User SID或Group SID作为关键字查询被访问对象中的DACL。顺序：先查询类型为DENY的ACE，若命中且权限符合则访问拒绝；未命中再在ALLOWED类型的ACE中查询，若命中且类型符合则可以访问；以上两步后还没命中那么访问拒绝。

![](https://raw.githubusercontent.com/myoss114/oss/master/uPic/Uu4bcV.jpg)

#### **\#\#\#** 

#### \#\#\# 访问令牌（Access tokens）

访问令牌是描述进程或线程的安全上下文的对象。令牌中的信息包括与进程或线程关联的用户帐户的标识和特权信息。当用户登录时，系统通过将用户密码与安全数据库中存储的信息进行比较来验证用户密码。如果密码经过验证，则系统将生成访问令牌。代表该用户执行的每个进程都有此访问令牌的副本。（通常我们在输入密码登陆进入Windows界面时就是一个生成访问令牌的过程）

当线程与安全对象进行交互或尝试执行需要特权的系统任务时，系统使用访问令牌来标识用户。访问令牌包含以下信息：

* 用户帐户的安全标识符（SID）
* 用户所属的组的SID
* 标识当前登录会话（logon session）的登录SID
* 用户或用户组拥有的特权列表
* 所有者SID
* 主要组的SID
* 用户创建安全对象而不指定安全描述符时系统使用的默认DACL
* 访问令牌的来源
* 令牌是主要令牌（内核分配的令牌）还是模拟令牌（模拟而来的令牌）
* 限制SID的可选列表
* 当前的模拟级别
* 其他数据

每个进程都有一个主要令牌，用于描述与该进程关联的用户帐户的安全上下文。默认情况下，当进程的线程与安全对象进行交互时，系统使用主令牌。此外，线程可以模拟客户端帐户。模拟允许线程使用客户端的安全上下文与安全对象进行交互。模拟客户端的线程同时具有主令牌和模拟令牌。（出现这种情况是因为服务操作是在寄宿进程中执行，在默认的情况下，服务操作是否具有足够的权限访问某个资源（比如文件）取决于执行寄宿进程Windows帐号的权限设置，而与作为客户端的Windows帐号无关。在有多情况下，我们希望服务操作执行在基于客户端的安全上下文中执行，以解决执行服务进行的帐号权限不足的问题。简单来说就是我们希望不同客户端来访问服务时，服务可以模拟客户端的身份去访问服务，而不是用自己的主进程Token身份去访问。）

使用`OpenProcessToken`函数可检索进程的主令牌的句柄。使用`OpenThreadToken`函数检索线程的模拟令牌的句柄。

您可以使用以下功能来操作访问令牌。

| 函数 | 描述 |
| :---: | :---: |
| [**AdjustTokenGroups**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokengroups) | 更改访问令牌中的组信息。 |
| [**AdjustTokenPrivileges**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges) | 启用或禁用访问令牌中的特权。它不会授予新特权或撤销现有特权。 |
| [**CheckTokenMembership**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-checktokenmembership) | 确定是否在指定的访问令牌中启用了指定的SID。 |
| [**CreateRestrictedToken**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-createrestrictedtoken) | 创建一个新令牌，它是现有令牌的受限版本。受限令牌可以具有禁用的SID、已删除的特权以及受限的SID列表。 |
| [**DuplicateToken**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetoken) | 创建一个新的模拟令牌，该令牌复制现有令牌。 |
| [**DuplicateTokenEx**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetokenex) | 创建一个新的主要令牌或模拟令牌，该令牌可复制现有令牌。 |
| [**GetTokenInformation**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation) | 检索有关令牌的信息。 |
| [**IsTokenRestricted**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-istokenrestricted) | 确定令牌是否具有限制SID列表。 |
| [**OpenProcessToken**](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken) | 检索进程的主要访问令牌的句柄。 |
| [**OpenThreadToken**](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openthreadtoken) | 检索线程的模拟访问令牌的句柄。 |
| [**SetThreadToken**](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadtoken) | 分配或删除线程的模拟令牌。 |
| [**SetTokenInformation**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-settokeninformation) | 更改令牌的所有者、主要组或默认DACL。 |

访问令牌功能使用以下结构来描述访问令牌的各个部分：

| 结构体 | 描述 |
| :---: | :---: |
| [**TOKEN\_CONTROL**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_control) | 标识访问令牌的信息。 |
| [**TOKEN\_DEFAULT\_DACL**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_default_dacl) | 系统在线程创建的新对象的安全描述符中使用的默认DACL。 |
| [**TOKEN\_GROUPS**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_groups) | 指定访问令牌中的SID和组SID的属性。 |
| [**TOKEN\_OWNER**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_owner) | 新对象的安全描述符的默认所有者SID。 |
| [**TOKEN\_PRIMARY\_GROUP**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_primary_group) | 新对象的安全描述符的默认主要组SID。 |
| [**TOKEN\_PRIVILEGES**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_privileges) | 与访问令牌关联的特权。还确定是否启用了特权。 |
| [**TOKEN\_SOURCE**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_source) | 访问令牌的来源。 |
| [**TOKEN\_STATISTICS**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_statistics) | 与访问令牌关联的统计信息。 |
| [**TOKEN\_USER**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ns-winnt-token_user) | 与访问令牌关联的用户的SID。 |

访问令牌功能使用以下枚举类型：

| 枚举类型 | 指定 |
| :---: | :---: |
| [**TOKEN\_INFORMATION\_CLASS**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ne-winnt-token_information_class) | 标识正在设置或从访问令牌检索的信息的类型。 |
| [**TOKEN\_TYPE**](https://docs.microsoft.com/en-us/windows/desktop/api/Winnt/ne-winnt-token_type) | 将访问令牌标识为主要令牌或模拟令牌。 |

**受限令牌**

受限令牌是已由`CreateRestrictedToken`函数修改的主要或模拟访问令牌。在受限令牌的安全上下文中运行的进程或模拟线程在访问安全对象或执行特权操作方面的能力受到限制。 `CreateRestrictedToken`函数可以通过以下方式限制令牌：

* 从令牌中删除特权。
* 将“仅拒绝”属性应用于令牌中的SID，以便它们不能用于访问受保护的对象。有关“仅拒绝”属性的更多信息，请参阅访问令牌中的SID属性\([https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-attributes-in-an-access-token\)。](https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-attributes-in-an-access-token%29。)
* 指定限制SID的列表，这可以限制对安全对象的访问。

当系统检查令牌对安全对象的访问时，系统将使用限制SID列表。当受限制的进程或线程尝试访问安全对象时，系统执行两项访问检查：一项使用令牌启用的SID，另一项使用限制SID列表。仅当两个访问检查都允许所请求的访问权限时，才授予访问权限。有关访问检查的更多信息，请参见DACL如何控制对对象的访问: [How DACLs Control Access to an Object](https://docs.microsoft.com/en-us/windows/win32/secauthz/how-dacls-control-access-to-an-object)

您可以在对`CreateProcessAsUser`函数的调用中使用受限制的主令牌。通常调用`CreateProcessAsUser`的进程必须具有`SE_ASSIGNPRIMARYTOKEN_NAME`特权，该特权通常仅由系统代码或`LocalSystem`帐户中运行的服务保留。但是，如果`CreateProcessAsUser`调用指定了调用者主令牌的受限版本，则不需要此特权。这使普通应用程序可以创建受限进程。

您还可以在`ImpersonateLoggedOnUser`函数中使用受限制的主要或模拟令牌。

若要确定令牌是否具有限制SID列表，请调用`IsTokenRestricted`函数。

注意：使用受限令牌的应用程序应在默认桌面以外的其他桌面上运行受限应用程序, 防止受限制的应用程序使用`SendMessage`或`PostMessage`攻击默认桌面上不受限制的应用程序。如有必要，请根据您的应用程序在桌面之间切换。

**访问令牌中的SID属性**

访问令牌中的每个用户和组安全标识符（SID）具有一组属性，这些属性控制系统在访问检查中如何使用SID。下表列出了控制访问检查的属性。

| 属性 | 描述 |
| :---: | :---: |
| SE\_GROUP\_ENABLED | 启用具有此属性的SID进行访问检查。当系统执行访问检查时，它将检查适用于访问令牌中已启用的SID之一的允许访问和拒绝访问的访问控制项（ACE）。除非设置了`SE_GROUP_USE_FOR_DENY_ONLY`属性，否则在访问检查期间将忽略不具有此属性的SID。 |
| SE\_GROUP\_USE\_FOR\_DENY\_ONLY | 具有此属性的SID是仅拒绝的SID。当系统执行访问检查时，它会检查适用于SID的拒绝访问的ACE，但是会忽略SID允许访问的ACE。如果设置了此属性，则不会设置SE\_GROUP\_ENABLED属性，并且无法重新启用SID |

要设置或清除组SID的`SE_GROUP_ENABLED`属性，请使用`AdjustTokenGroups`函数。您不能禁用具有`SE_GROUP_MANDATORY`属性的组SID。您不能使用`AdjustTokenGroups`禁用访问令牌的用户SID。

若要确定令牌中是否启用了SID，即它是否具有`SE_GROUP_ENABLED`属性，请调用`CheckTokenMembership`函数。

若要设置SID的`SE_GROUP_USE_FOR_DENY_ONLY`属性，请在调用`CreateRestrictedToken`函数时指定的拒绝专用SID列表中包含的该SID。 `CreateRestrictedToken`可以将`SE_GROUP_USE_FOR_DENY_ONLY`属性应用于任何SID，包括用户SID和具有`SE_GROUP_MANDATORY`属性的组SID。但是，您不能从SID中删除仅拒绝属性，也不能使用`AdjustTokenGroups`在仅拒绝SID上设置`SE_GROUP_ENABLED`属性。

要获取SID的属性，请使用`TokenGroups`值调用`GetTokenInformation`函数。该函数返回一个`SID_AND_ATTRIBUTES`结构数组，该结构标识组SID及其属性。

**访问令牌对象的访问权限**

除非应用程序有权更改对象的访问控制列表，否则该应用程序不能更改该对象的访问控制列表。这些权限由对象的访问令牌中的安全描述符控制。有关安全性的更多信息，请参见访问[控制模型](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-model)。

要获取或设置访问令牌的安全描述符，请调用`GetKernelObjectSecurity`和`SetKernelObjectSecurity`函数。

当您调用`OpenProcessToken`或`OpenThreadToken`函数以获取访问令牌的句柄时，系统将根据令牌的安全描述符中的DACL检查请求的访问权限。

以下是访问令牌对象的有效访问权限：

* `DELETE`，`READ_CONTROL`，`WRITE_DAC`和`WRITE_OWNER`标准访问权限。访问令牌不支持`SYNCHRONIZE`标准访问权限。
* 获得或设置对象的安全描述符中的`SACL`的`ACCESS_SYSTEM_SECURITY`权限。
* 下表列出了访问令牌的特定访问权限:
* | 值 | 含义 |
  | :---: | :---: |
  | TOKEN\_ADJUST\_DEFAULT | 更改访问令牌的默认所有者，主要组或DACL是必需的。 |
  | TOKEN\_ADJUST\_GROUPS | 需要在访问令牌中调整组的属性 |
  | TOKEN\_ADJUST\_PRIVILEGES | 启用或禁用访问令牌中的特权 |
  | TOKEN\_ADJUST\_SESSIONID | 调整访问令牌的会话ID。必须具有`SE_TCB_NAME`特权。 |
  | TOKEN\_ASSIGN\_PRIMARY | 将主令牌附加到进程。要完成此任务，还需要`SE_ASSIGNPRIMARYTOKEN_NAME`特权。 |
  | TOKEN\_DUPLICATE | 复制访问令牌 |
  | TOKEN\_EXECUTE | 结合`STANDARD_RIGHTS_EXECUTE`和`TOKEN_IMPERSONATE` |
  | TOKEN\_IMPERSONATE | 将模拟访问令牌附加到进程 |
  | TOKEN\_QUERY | 查询访问令牌 |
  | TOKEN\_QUERY\_SOURCE | 查询访问令牌的来源 |
  | TOKEN\_READ | 结合`STANDARD_RIGHTS_READ`和`TOKEN_QUERY` |
  | TOKEN\_WRITE | 合并`STANDARD_RIGHTS_WRITE`，`TOKEN_ADJUST_PRIVILEGES`，`TOKEN_ADJUST_GROUPS`和`TOKEN_ADJUST_DEFAULT` |
  | TOKEN\_ALL\_ACCESS | 合并令牌的所有可能的访问权限 |

**安全描述符**

安全描述符包含与安全对象关联的安全信息。安全描述符由`SECURITY_DESCRIPTOR`结构及其关联的安全信息组成。安全描述符可以包括以下安全信息：

* 对象的所有者和主要组的安全标识符（SID）
* 指定特定用户或组允许或拒绝的访问权限的DACL
* 一个SACL，它指定为对象生成审核记录的访问尝试的类型
* 一组控制位，用于限定安全描述符或其单个成员的含义

应用程序不得直接操纵安全描述符的内容。 Windows API提供了用于在对象的安全描述符中设置和检索安全信息的功能。此外，还有用于创建和初始化新对象的安全描述符的函数。

在`Active Directory`对象上使用安全描述符的应用程序可以使用Windows安全功能或`Active Directory`服务接口（ADSI）提供的安全接口。有关ADSI安全接口的详细信息，请参阅访问控制在`Active Directory`中的工作方式：[https://docs.microsoft.com/en-us/windows/desktop/AD/how-access-control-works-in-active-directory-domain-services](https://docs.microsoft.com/en-us/windows/desktop/AD/how-access-control-works-in-active-directory-domain-services)

**安全描述符操作**

Windows API提供了用于获取和设置与安全对象关联的安全描述符的组件的功能。使用`GetSecurityInfo`和`GetNamedSecurityInfo`函数检索指向对象的安全描述符的指针。这些函数还可以检索指向安全描述符的各个组件的指针：DACL、SACL、所有者SID和主要组SID。使用`SetSecurityInfo`和`SetNamedSecurityInfo`函数来设置对象的安全描述符的组件。

通常，应将`GetSecurityInfo`和`SetSecurityInfo`与通过句柄标识的对象一起使用，将`SetNamedSecurityInfo`和`GetNamedSecurityInfo`与通过名称标识的对象一起使用。有关在使用各种类型的对象时要使用的特定功能的更多信息，请参见安全对象:[https://docs.microsoft.com/en-us/windows/win32/secauthz/securable-objects](https://docs.microsoft.com/en-us/windows/win32/secauthz/securable-objects)

Windows API提供了用于操纵安全描述符的组件的其他功能。有关使用访问控制列表（DACL或SACL）的信息，请参阅从ACL获取信息和创建或修改ACL。有关SID的信息，请参阅安全标识符（SID）。

要在安全描述符中获取控制信息，请调用`GetSecurityDescriptorControl`函数。要设置与自动ACE继承相关的控制位，请调用`SetSecurityDescriptorControl`函数。其他控制位由设置安全描述符组件的各种功能设置。例如，如果使用`SetSecurityInfo`更改对象的DACL，则该函数将适当地设置或清除位，以指示安全描述符是否具有DACL，是否为默认DACL，等等。另一个示例是安全描述符中包含的资源管理器（RM）控制位。这些位根据资源管理器的实现使用，并通过`GetSecurityDescriptorRMControl`和`SetSecurityDescriptorRMControl`函数进行访问。

**新对象的安全描述符**

创建安全对象时，可以将安全描述符分配给新对象。用于创建安全对象的函数（例如`CreateFile`或`RegCreateKeyEx`）具有指向`SECURITY_ATTRIBUTES`结构的参数，该结构可以包含指向新对象的安全描述符的指针。有关构建安全描述符，然后调用`RegCreateKeyEx`将该安全描述符分配给新注册表项的示例代码，请参见在C ++中为新对象创建安全描述符。

管理对象的系统组件或服务器可以存储指定的或默认的安全描述符，以使其成为对象的持久属性。如果对象的创建者未指定安全描述符，则系统将使用继承的或默认的安全信息来创建安全描述符。您可以使用函数来更改对象的安全描述符中的信息。

目录服务对象、文件、目录、注册表项和桌面是安全对象，可以具有父对象。创建这些对象之一时，系统会在父对象的安全描述符中检查可继承的ACE。系统通常将任何可继承的ACE合并到新对象的安全描述符的ACL中。您可以通过在安全描述符的控制位中设置`SE_DACL_PROTECTED`或`SE_SACL_PROTECTED`位来防止`DACL`或`SACL`继承`ACE`。有关更多信息，请参见ACE继承。

_新对象的DACL_

系统使用以下算法为大多数类型的新可保护对象构建DACL：

1. 对象的DACL是来自对象创建者指定的安全描述符的DACL。除非在安全描述符的控制位中设置了`SE_DACL_PROTECTED`位，否则系统会将所有可继承的ACE合并到指定的DACL中。
2. 如果创建者未指定安全描述符，则系统将从可继承的ACE构建对象的DACL。
3. 如果未指定安全描述符，并且没有可继承的ACE，则对象的DACL是来自创建者的主令牌或模拟令牌的默认DACL。
4. 如果没有指定的、继承的或默认的DACL，则系统将创建不具有DACL的对象，从而允许所有人完全访问该对象。

系统使用不同的算法为新的Active Directory对象构建DACL。有关更多信息，请参见如何在新目录对象上设置安全描述符 ： [https://docs.microsoft.com/en-us/windows/desktop/AD/how-security-descriptors-are-set-on-new-directory-objects](https://docs.microsoft.com/en-us/windows/desktop/AD/how-security-descriptors-are-set-on-new-directory-objects)

_新对象的SACL_

系统使用以下算法为大多数类型的新安全对象构建SACL：

1. 对象的SACL是对象创建者指定的安全描述符中的SACL。除非在安全描述符的控制位中设置了`SE_SACL_PROTECTED`位，否则系统会将所有可继承的ACE合并到指定的SACL中。即使`SE_SACL_PROTECTED`位置已经设置，来自父对象的`SYSTEM_RESOURCE_ATTRIBUTE_ACE`和`SYSTEM_SCOPED_POLICY_ID_ACE`也将合并到新对象。
2. 如果创建者未指定安全描述符，则系统将从可继承的ACE构建对象的SACL。
3. 如果没有指定的或继承的SACL，则该对象没有SACL。

要为新对象指定SACL，对象的创建者必须启用`SE_SECURITY_NAME`特权。如果为新对象指定的SACL仅包含`SYSTEM_RESOURCE_ATTRIBUTE_ACE`，则不需要`SE_SECURITY_NAME`特权。如果对象的SACL是从继承的ACE构建的，则创建者不需要此特权。

系统使用不同的算法为新的Active Directory对象构建SACL。有关更多信息，请参见如何在新目录对象上设置安全描述符: [https://docs.microsoft.com/en-us/windows/desktop/AD/how-security-descriptors-are-set-on-new-directory-objects](https://docs.microsoft.com/en-us/windows/desktop/AD/how-security-descriptors-are-set-on-new-directory-objects)

_新对象的所有者_

对象的所有者隐式具有对该对象的`WRITE_DAC`访问权限。这意味着所有者可以修改对象的自由访问控制列表（DACL），从而可以控制对对象的访问。

新对象的所有者是创建过程的主令牌或模拟令牌中的默认所有者安全标识符（SID）。要获取或设置访问令牌中的默认所有者，请使用`TOKEN_OWNER`结构调用`GetTokenInformation`或`SetTokenInformation`函数。系统不允许您将令牌的默认所有者设置为无效的SID，例如另一个用户帐户的SID。

启用了`SE_TAKE_OWNERSHIP`特权的进程可以将自身设置为对象的所有者。启用了`SE_RESTORE_NAME`特权或对对象具有`WRITE_OWNER`访问权限的进程可以将任何有效的用户或组SID设置为对象的所有者。

_新对象的主要组_

新对象的主要组是对象创建者指定的安全描述符中的主要组。如果对象的创建者未指定主要组，则该对象的主要组是创建者的主要或模拟令牌中的默认主要组。

**安全描述符字符串**

有效的功能安全描述符包含二进制格式的安全信息。 `Windows API`提供了用于将二进制安全描述符与文本字符串相互转换的功能。字符串格式的安全描述符不起作用，但是对于存储或传输安全描述符信息很有用。

要将安全描述符转换为字符串格式，请调用 [`ConvertSecurityDescriptorToStringSecurityDescriptor`](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertsecuritydescriptortostringsecuritydescriptora)函数。要将字符串格式的安全描述符转换回有效的功能安全描述符，请调用[`ConvertStringSecurityDescriptorToSecurityDescriptor`](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertstringsecuritydescriptortosecuritydescriptora)函数。

**访问控制列表**

访问控制列表（ACL）是访问控制条目（ACE）的列表。 ACL中的每个ACE都标识一个受托者，并指定允许、拒绝或审核该受托者的访问权限。可保护对象的安全描述符可以包含两种类型的ACL：DACL和SACL。

任意访问控制列表（DACL）标识允许或拒绝访问安全对象的受托者。当进程尝试访问安全对象时，系统将检查该对象的DACL中的ACE，以确定是否授予对该对象的访问权限。如果对象没有DACL，则系统将授予所有人完全访问权限。如果对象的DACL没有ACE，则系统将拒绝所有尝试访问该对象的尝试，因为DACL不允许任何访问权限。系统依次检查ACE，直到找到一个或多个允许所有请求的访问权限的ACE，或者直到拒绝任何请求的访问权限为止。有关更多信息，请参见DACL如何控制对对象的访问。有关如何正确创建DACL的信息，请参阅创建DACL。

系统访问控制列表（SACL）使管理员可以记录访问安全对象的尝试。每个ACE指定指定受托者尝试访问的类型，这些尝试使系统在安全事件日志中生成记录。当访问尝试失败、成功或失败时，SACL中的ACE可以生成审核记录。有关SACL的更多信息，请参见审核生成和SACL访问权限。

不要尝试直接使用ACL的内容。为确保ACL在语义上正确，请使用适当的函数来创建和操作ACL。有关更多信息，请参见从ACL获取信息和创建或修改ACL。

ACL还提供对`Microsoft Active Directory`目录服务对象的访问控制。 `Active Directory`服务接口（ADSI）包括用于创建和修改这些ACL内容的例程。有关更多信息，请参见控制对Active Directory对象的访问。

**从ACL获取信息**

提供了几种从访问控制列表（ACL）中检索访问控制信息的功能。这些功能包括确定ACL授予或审核指定受托者的访问权限的功能。其他功能使您能够提取有关ACL中访问控制项（ACE）的信息。

`GetExplicitEntriesFromAcl`函数检索描述在ACL中的ACE的`EXPLICIT_ACCESS`结构的数组。将ACE信息从一个ACL复制到另一个ACL时，此功能很有用。例如，对`GetExplicitEntriesFromAcl`的调用以在一个ACL中获取有关ACE的信息之后，可以通过在对`SetEntriesInAcl`函数的调用中传递返回的`EXPLICIT_ACCESS`结构，以在新的ACL中创建等效的ACE。

`GetEffectiveRightsFromAcl`函数使您能够确定DACL授予指定受托者的有效访问权限。受托人的有效访问权是DACL授予受托人或受托人为成员的任何组的访问权。 `GetEffectiveRightsFromAcl`检查指定DACL中的所有允许访问和拒绝访问的ACE。

使用以下步骤确定受托者对对象的访问权限:

1. 调用`GetSecurityInfo`或`GetNamedSecurityInfo`函数以获取指向对象的DACL的指针。
2. 调用`GetEffectiveRightsFromAcl`函数以检索DACL授予指定受托者的访问权限。

`GetAuditedPermissionsFromAcl`函数使您可以检查SACL，以确定指定受托人或受托人所属成员的任何组的审核访问权限。审核的权限指示导致系统在安全事件日志中生成审核记录的访问尝试的类型。该函数返回两个访问掩码：一个包含对失败的访问尝试进行监视的访问权限，另一个包含对成功的访问进行监视的访问权限。 `GetAuditedPermissionsFromAcl`检查SACL中的所有系统审核的ACE。

**创建或修改ACL**

Windows支持一组功能，这些功能可以创建访问控制列表（ACL）或修改现有ACL中的访问控制项（ACE）。

`SetEntriesInAcl`函数创建一个新的ACL。 `SetEntriesInAcl`可以为ACL指定一组全新的ACE，也可以将一个或多个新ACE与现有ACL的ACE合并。 `SetEntriesInAcl`函数使用`EXPLICIT_ACCESS`结构的数组来指定新ACE的信息。每个EXPLICIT\_ACCESS结构都包含描述单个ACE的信息。此信息包括访问权限、ACE的类型、控制ACE继承的标志以及标识受托者的`TRUSTEE`结构。

向现有ACL添加新ACE

1. 使用`GetSecurityInfo`或`GetNamedSecurityInfo`函数可从对象的安全描述符获取现有的DACL或SACL。
2. 对于每个新的ACE，请调用`BuildExplicitAccessWithName`函数以使用描述ACE的信息填充`EXPLICIT_ACCESS`结构。
3. 调用`SetEntriesInAcl`，为新ACE指定现有的ACL和`EXPLICIT_ACCESS`结构的数组。`SetEntriesInAcl`函数分配并初始化ACL及其ACE。
4. 调用`SetSecurityInfo`或`SetNamedSecurityInfo`函数，将新的ACL附加到对象的安全描述符。

如果调用方指定了现有的ACL，则`SetEntriesInAcl`会将新的ACE信息与ACL中的现有ACE合并。例如，考虑以下情况：现有ACL授予对指定受托者的访问权限，而`EXPLICIT_ACCESS`结构拒绝对同一受托者的访问权限。在这种情况下，`SetEntriesInAcl`为受托者添加一个新的拒绝访问的ACE，并为受托者删除或修改现有的允许访问的ACE。

有关将新ACE合并到现有ACL中的示例代码，请参见[Modifying the ACLs of an Object in C++](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/modifying-the-acls-of-an-object-in-c--).

**访问控制项**

访问控制条目（ACE）是访问控制列表（ACL）中的元素。一个ACL可以包含零个或多个ACE。每个ACE控制或监视指定受托者对对象的访问。有关在对象的ACL中添加，删除或更改ACE的信息，请参阅在C ++中修改对象的ACL。

ACE有六种类型，所有安全对象都支持其中三种。其他三种类型是目录服务对象支持的特定于对象的ACE.

所有类型的ACE都包含以下访问控制信息：

* 安全标识符（SID），用于标识ACE适用于的受托者
* 一个访问掩码，用于指定由ACE控制的访问权限。
* 指示ACE类型的标志。
* 一组位标志，用于确定子容器或对象是否可以从附加了ACL的主对象继承ACE。

下表列出了所有安全对象支持的三种ACE类型。

| 类型 | 描述 |
| :---: | :---: |
| Access-denied ACE | 在自由访问控制列表（DACL）中使用，以拒绝对受托者的访问 |
| Access-allowed ACE | 在DACL中使用，以允许对受托者的访问权限。 |
| System-audit ACE | 当受托者尝试行使指定的访问权限时，用于系统访问控制列表（SACL）中以生成审核记录。 |

**受托者**

受托者是访问控制条目（ACE）应用到的用户帐户、组帐户或登录会话。访问控制列表（ACL）中的每个ACE都有一个安全标识符（SID），用于标识受托者。

用户帐户包括人类用户或Windows Services等程序用于登录本地计算机的帐户。

组帐户不能用于登录计算机，但是在ACE中它们非常有用，可以允许或拒绝对一个或多个用户帐户的一组访问权限。

标识当前登录会话的登录SID仅在用户注销之前才允许访问控制功能使用TRUSTEE结构来标识受托者。 TRUSTEE结构使您可以使用名称字符串或SID来标识受托者。如果使用名称，则从TRUSTEE结构创建ACE的功能将执行分配SID缓冲区并查找与帐户名称相对应的SID的任务。有两个帮助程序函数`BuildTrusteeWithSid`和`BuildTrusteeWithName`，它们使用指定的SID或名称初始化TRUSTEE结构。 `BuildTrusteeWithObjectsAndSid`和`BuildTrusteeWithObjectsAndName`允许您使用特定于对象的ACE信息初始化TRUSTEE结构。其他三个帮助函数`GetTrusteeForm`、`GetTrusteeName`和`GetTrusteeType`检索TRUSTEE结构的各个成员的值。或拒绝访问权限。

TRUSTEE结构的`ptstrName`成员可以是指向`OBJECTS_AND_NAME`或`OBJECTS_AND_SID`结构的指针。这些结构除了指定受托人名称或SID之外，还指定有关特定于对象的ACE的信息。这使诸如`SetEntriesInAcl`和`GetExplicitEntriesFromAcl`之类的功能可以将特定于对象的ACE信息存储在`EXPLICIT_ACCESS`结构的Trustee成员中。

**特定对象的ACE**

目录服务（DS）对象支持特定于对象的ACE。特定于对象的ACE包含一对GUID，它们扩展了ACE保护对象的方式。

| GUID | 描述 |
| :---: | :---: |
| **ObjectType** | 标识以下之一：  一种 子对象。 ACE控制创建指定类型的子对象的权限。有关更多信息，请参见在C ++中控制子对象的创建。 属性集或属性。 ACE控制读取或写入属性或属性集的权限。有关更多信息，请参见用于控制对对象属性的访问的ACE。 延长的权利。 ACE控制执行与扩展权限相关的操作的权限。 经过验证的写入。 ACE控制执行某些写操作的权限。这些在ACL编辑器中定义和公开的经过验证的写入权限为经过验证的属性写入提供了权限，而不是对被授予“写入属性”权限的属性的任何值进行未经检查的低级写入。 |
| **InheritedObjectType** | 指示可以继承ACE的子对象的类型。继承也由ACE\_HEADER中的继承标志以及放置在子对象上的任何防止继承的保护来控制。有关更多信息，请参见ACE继承。 |

**DACL中的ACE顺序**

当进程尝试访问安全对象时，系统逐步浏览该对象的任意访问控制列表（DACL）中的访问控制项（ACE），直到找到允许或拒绝所请求访问的ACE。 DACL允许用户的访问权限可能会根据DACL中ACE的顺序而有所不同。因此，Windows XP操作系统在安全对象的DACL中为ACE定义了首选顺序。首选顺序提供了一个简单的框架，可确保拒绝访问的ACE实际上拒绝访问。有关系统用于检查访问的算法的更多信息，请参见DACL如何控制对对象的访问。

对于Windows Server 2003和Windows XP，引入特定于对象的ACE和自动继承会使得ACE的正确顺序变得复杂。

以下步骤描述了首选顺序：

1. 将所有显式ACE放在一个组中，然后放在任何继承的ACE之前。
2. 在显式ACE组中，拒绝访问的ACE放置在允许访问的ACE之前。
3. 继承的ACE按照继承的顺序放置。从子对象的父对象继承的ACE首先出现，然后是从祖父母那里继承的ACE，依此类推。
4. 对于继承的ACE的每个级别，将拒绝访问的ACE放置在允许访问的ACE之前。

当然，并不是ACL中需要所有ACE类型。

**ACE继承**

对象的ACL可以包含从其父容器继承的ACE。例如，注册表子项可以从注册表层次结构中位于其上方的项继承ACE。同样，NTFS文件系统中的文件可以从包含该文件的目录继承ACE。

ACE的`ACE_HEADER`结构包含一组继承标志，这些继承标志控制ACE继承以及ACE对它所连接的对象的影响。系统根据ACE继承规则解释继承标志和其他继承信息。

这些规则通过以下功能得到增强：

* 支持自动传播可继承的ACE。
* 一个标志，用于区分继承的ACE和直接应用于对象的ACE。
* 特定于对象的ACE，允许您指定可以继承ACE的子对象的类型。
* 通过将除`SYSTEM_RESOURCE_ATTRIBUTE_ACE`和`SYSTEM_SCOPED_POLICY_ID_ACE`之外的安全描述符的控制位中的`SE_DACL_PROTECTED`或`SE_SACL_PROTECTED`位设置来防止DACL或SACL继承ACE的能力。

**自动传播可继承的ACE**

`SetNamedSecurityInfo`和`SetSecurityInfo`函数支持可继承访问控制项（ACE）的自动传播。例如，如果使用这些功能将可继承的ACE添加到NTFS中的目录，则系统会将ACE适当地应用于任何现有子目录或文件的访问控制列表（ACL）

直接应用的ACE优先于继承的ACE。系统通过将直接应用的ACE放在任意访问控制列表（DACL）中继承的ACE之前来实现此优先级。当您调用`SetNamedSecurityInfo`和`SetSecurityInfo`函数设置对象的安全信息时，系统会将当前继承模型强加于目标对象下方层次结构中所有对象的ACL上。对于已转换为当前继承模型的对象，在对象的安全描述符的控制字段中设置`SE_DACL_AUTO_INHERITED`和`SE_SACL_AUTO_INHERITED`位。

当您构建一个反映当前继承模型的新安全描述符时，请注意不要更改安全描述符的语义。因此，允许和拒绝ACE绝不会彼此相对移动。如果需要进行此类移动（例如，将所有非继承的ACE放在ACL的前面），则将该ACL标记为受保护，以防止语义更改。

将继承的ACE传播到子对象时，系统使用以下规则：

* 如果没有DACL的子对象继承了ACE，则结果是带有DACL的子对象仅包含继承的ACE。
* 如果带有空DACL的子对象继承了ACE，则结果是带有DACL的子对象仅包含继承的ACE。
* 如果从父对象中删除可继承的ACE，则自动继承将删除子对象继承的ACE的所有副本。
* 如果自动继承导致从子对象的DACL中删除所有ACE，则子对象具有空的DACL，而不是没有DACL。

这些规则可能会产生意外结果，即将没有DACL的对象转换为具有空DACL的对象。没有DACL的对象允许完全访问，但是具有空DACL的对象不允许访问。作为这些规则如何创建空DACL的示例，假设您将可继承的ACE添加到对象树的根对象。自动继承将可继承的ACE传播到树中的所有对象。现在，没有DACL的子对象具有带有继承的ACE的DACL。如果从根对象中删除可继承的ACE，则系统会自动将更改传播到子对象。现在，没有DACL（允许完全访问）的子对象具有空的DACL（不允许访问）。

为确保没有DACL的子对象不受继承ACE的影响，请在对象的安全描述符中设置`SE_DACL_PROTECTED`标志。

**ACE继承规则**

系统根据一组继承规则将可继承访问控制项（ACE）传播到子对象。系统会根据DACL中ACE的首选顺序，将继承的ACE放置在子代的自由访问控制列表（DACL）中。系统在所有继承的ACE中设置`INHERITED_ACE`标志。

容器和非容器子对象继承的ACE有所不同，具体取决于继承标志的组合。这些继承规则对DACL和系统访问控制列表（SACL）均起作用。

| 父ACE标志 | 对子ACL的影响 |
| :---: | :---: |
| OBJECT\_INHERIT\_ACE only | 非容器子对象：继承为有效的ACE。容器子对象：除非还设置了`NO_PROPAGATE_INHERIT_ACE`位标志，否则容器将继承仅继承ACE。 |
| CONTAINER\_INHERIT\_ACE only | 非容器子对象：对子对象无影响。容器子对象：子对象继承有效的ACE。除非还设置了`NO_PROPAGATE_INHERIT_ACE`位标志，否则继承的ACE是可继承的。 |
| CONTAINER\_INHERIT\_ACE and OBJECT\_INHERIT\_ACE | 非容器子对象：继承为有效的ACE。容器子对象：子对象继承有效的ACE。除非还设置了`NO_PROPAGATE_INHERIT_ACE`位标志，否则继承的ACE是可继承的。 |
| No inheritance flags set | 对子容器或非容器对象没有影响。 |

如果继承的ACE是子对象的有效ACE，则系统会将所有通用权限映射到子对象的特定权限。类似地，系统将诸如`CREATOR_OWNER`之类的通用安全标识符（SID）映射到适当的SID。如果继承的ACE是仅继承的ACE，则任何通用权限或通用SID均保持不变，以便在下一代子对象继承ACE时可以正确地映射它们。

对于容器对象继承既对容器有效又可由其后代继承的ACE的情况，容器可以继承两个ACE。如果可继承ACE包含通用信息，则会发生这种情况。容器继承了一个仅继承ACE，该ACE包含了一般信息，而一个仅有效ACE映射了一般信息。

特定于对象的ACE具有`InheritedObjectType`成员，该成员可以包含GUID来标识可以继承ACE的对象的类型。

如果未指定`InheritedObjectType GUID`，则特定于对象的ACE的继承规则与标准ACE的继承规则相同。

如果指定了`InheritedObjectType GUID`，则如果设置了`OBJECT_INHERIT_ACE`，则可以由与GUID匹配的对象继承ACE，如果设置了`CONTAINER_INHERIT_ACE`，则可以由与GUID匹配的容器继承。请注意，当前只有DS对象支持特定于对象的ACE，并且DS将所有对象类型都视为容器。

**ACE控制对对象属性的访问**

目录服务（DS）对象的任意访问控制列表（DACL）可以包含访问控制项（ACE）的层次结构，如下所示：

1. 保护对象本身的ACE
2. 特定对象的ACE，可保护对象上的指定属性集
3. 特定对象的ACE，用于保护对象上的指定属性

在此层次结构中，较高级别授予或拒绝的权利也适用于较低级别。例如，如果属性集上的特定于对象的ACE允许受托者获得`ADS_RIGHT_DS_READ_PROP`权限，则受托者具有对该属性集所有属性的隐式读取访问权限。同样，在对象本身上的ACE允许`ADS_RIGHT_DS_READ_PROP`访问，使受托者可以对对象的所有属性进行读取访问。

下图显示了假设的DS对象的树及其属性集和属性:

![directory service object hierarchy](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl2.png)

假设您希望允许以下访问此DS对象的属性：

* 允许A组对所有对象属性的读/写权限
* 允许其他所有人对除属性D之外的所有属性具有读/写权限

为此，如下表所示，在对象的DACL中设置ACE:

| 受托者 | 对象 GUID | ACE 类型 | 访问权限 |
| :--- | :--- | :--- | :---: |
| Group A | None | Access-allowed ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |
| Everyone | Property Set 1 | Access-allowed object ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |
| Everyone | Property C | Access-allowed object ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |

组A的ACE没有对象GUID，这意味着它允许访问所有对象的属性。属性集1的特定于对象的ACE允许所有人访问属性A和B。另一个特定于对象的ACE允许所有人访问属性C。请注意，尽管此DACL没有任何访问被拒绝的ACE，但它隐式拒绝了属性D。与除A组之外的所有人访问。

当用户尝试访问对象的属性时，系统将按顺序检查ACE，直到显式授予，拒绝所请求的访问，或者不再有ACE，在这种情况下，访问将被隐式拒绝。

系统评估：

* 适用于对象本身的ACE
* 适用于包含正在访问的属性的属性集的特定于对象的ACE
* 适用于所访问属性的特定于对象的ACE

系统将忽略适用于其他属性集或属性的特定于对象的ACE。

**访问权限和访问掩码**

访问权限是一个位标志，它对应于线程可以在可保护对象上执行的一组特定操作。例如，注册表项具有`KEY_SET_VALUE`访问权限，这对应于线程在项下设置值的能力。如果线程尝试对某个对象执行操作，但没有对该对象的必要访问权限，则系统不会执行该操作。

访问掩码是一个32位的值，其位与对象支持的访问权限相对应。所有Windows安全对象均使用访问掩码格式，该格式包含用于以下类型的访问权限的位：

* 通用访问权限
* 标准访问权限
* SACL访问权限
* 目录服务访问权限

当线程尝试打开对象的句柄时，该线程通常会指定访问掩码以请求一组访问权限。例如，需要设置和查询注册表项的值的应用程序可以通过使用访问掩码来请求`KEY_SET_VALUE`和`KEY_QUERY_VALUE`访问权限来打开注册表项。

下表显示了用于处理每种安全对象类型的安全信息的功能。

| Object type | Security descriptor functions |
| :--- | :--- |
| [Files or directories](https://docs.microsoft.com/en-us/windows/desktop/FileIO/file-security-and-access-rights) on an NTFS file system | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Named pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipe-security-and-access-rights)[Anonymous pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/anonymous-pipe-security-and-access-rights) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| Console screen buffers | Not supported. |
| [Processes](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/process-security-and-access-rights)[Threads](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/thread-security-and-access-rights) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [File-mapping objects](https://docs.microsoft.com/en-us/windows/desktop/Memory/file-mapping-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Access tokens](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/access-rights-for-access-token-objects) | [**SetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setkernelobjectsecurity), [**GetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getkernelobjectsecurity) |
| Window-management objects \([window stations](https://docs.microsoft.com/en-us/windows/desktop/winstation/window-station-security-and-access-rights) and [desktops](https://docs.microsoft.com/en-us/windows/desktop/winstation/desktop-security-and-access-rights)\) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Registry keys](https://docs.microsoft.com/en-us/windows/desktop/SysInfo/registry-key-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Windows services](https://docs.microsoft.com/en-us/windows/desktop/Services/service-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| Local or remote printers | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| Network shares | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Interprocess synchronization objects](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights) \(events, mutexes, semaphores, and waitable timers\) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Job objects](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/job-object-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |

**访问掩码格式**

所有安全对象都使用下图所示的访问掩码格式来安排其访问权限。

![access mask format](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl4.png)

在这种格式中，低16位用于特定对象的访问权限，后8位用于标准访问权限，这些权限适用于大多数类型的对象，而4个高位用于指定通用访问权限每种对象类型可以映射到一组标准和特定于对象的权限。 `ACCESS_SYSTEM_SECURITY`位对应于访问对象的SACL的权限。

**通用访问权限**

全对象使用访问掩码格式，其中四个高位指定通用访问权限。每种可保护对象的类型都将这些位映射到一组其标准和特定于对象的访问权限。例如，Windows文件对象将`GENERIC_READ`位映射到`READ_CONTROL`和`SYNCHRONIZE`标准访问权限以及`FILE_READ_DATA`，`FILE_READ_EA`和`FILE_READ_ATTRIBUTES`对象特定的访问权限。其他类型的对象将`GENERIC_READ`位映射到适合该类型对象的任何访问权限集。

您可以使用通用访问权限来指定打开对象的句柄时所需的访问类型。这通常比指定所有相应的标准和特定权限要简单。

下表显示了为通用访问权限定义的常量。

| 常量 | 通用含义 |
| :--- | :--- |
| GENERIC\_ALL | All possible access rights |
| GENERIC\_EXECUTE | Execute access |
| GENERIC\_READ | Read access |
| GENERIC\_WRITE | Write access |

**标准访问权限**

每种类型的可保护对象都有一组访问权限，这些访问权限对应于特定于该类型对象的操作。除了这些特定于对象的访问权限之外，还有一组标准访问权限，它们对应于大多数类型的可保护对象的通用操作。

访问掩码格式包括一组用于标准访问权限的位。 Winnt.h中定义了以下Windows标准访问权限常量。

| 常量 | 含义 |
| :--- | :--- |
| DELETE | 删除对象的权利 |
| READ\_CONTROL | 有权读取对象的安全描述符中的信息，但不包括系统访问控制列表（SACL）中的信息。 |
| SYNCHRONIZE | 使用对象进行同步的权利。这使线程可以等待，直到对象处于信号状态。某些对象类型不支持此访问权限。 |
| WRITE\_DAC | 修改对象的安全描述符中的任意访问控制列表（DACL）的权限。 |
| WRITE\_OWNER | 在对象的安全描述符中更改所有者的权利。 |

Winnt.h还定义了标准访问权限常量的以下组合

| 常量 | 含义 |
| :--- | :--- |
| STANDARD\_RIGHTS\_ALL | 合并DELETE，READ\_CONTROL，WRITE\_DAC，WRITE\_OWNER和SYNCHRONIZE访问 |
| STANDARD\_RIGHTS\_EXECUTE | 当前定义为等于READ\_CONTROL。 |
| STANDARD\_RIGHTS\_READ | 当前定义为等于READ\_CONTROL。 |
| STANDARD\_RIGHTS\_REQUIRED | 合并DELETE，READ\_CONTROL，WRITE\_DAC和WRITE\_OWNER访问。 |
| STANDARD\_RIGHTS\_WRITE | 当前定义为等于READ\_CONTROL |

**SACL访问权限**

`ACCESS_SYSTEM_SECURITY`访问权限控制在对象的安全描述符中获取或设置SACL的能力。仅当在请求线程的访问令牌中启用`SE_SECURITY_NAME`特权时，系统才会授予此访问权限。

访问对象的SACL :

1. 调用`AdjustTokenPrivileges`函数以启用`SE_SECURITY_NAME`特权。
2. 打开对象的句柄时，请求`ACCESS_SYSTEM_SECURITY`访问权限。
3. 通过使用诸如`GetSecurityInfo`或`SetSecurityInfo`之类的函数来获取或设置对象的SACL。
4. 调用`AdjustTokenPrivileges`以禁用`SE_SECURITY_NAME`特权

要使用`GetNamedSecurityInfo`或`SetNamedSecurityInfo`函数访问SACL，请启用`SE_SECURITY_NAME`特权。该函数在内部请求访问权限。

`ACCESS_SYSTEM_SECURITY`访问权限在DACL中无效，因为DACL不控制对SACL的访问。但是，您可以在SACL中使用`ACCESS_SYSTEM_SECURITY`访问权限来审核使用该访问权限的尝试。

**目录服务访问权限**

每个`Active Directory`对象都有一个分配给它的安全描述符。可以在这些安全描述符中设置特定于目录服务对象的一组受托者权限。下表列出了这些权利。有关更多信息，请参见控制访问权限。

| 权限 | Meaning |
| :--- | :--- |
| ACTRL\_DS\_OPEN | 打开DS对象 |
| ACTRL\_DS\_CREATE\_CHILD | 创建子DS对象 |
| ACTRL\_DS\_DELETE\_CHILD | 删除子DS对象 |
| ACTRL\_DS\_LIST | 枚举DS对象 |
| ACTRL\_DS\_READ\_PROP | 读取DS对象的属性 |
| ACTRL\_DS\_WRITE\_PROP | 写入DS对象的属性 |
| ACTRL\_DS\_SELF | 仅在执行对象支持的经过验证的权限检查后才允许访问。该标志可以单独用于执行对象的所有已验证权限检查，也可以与特定已验证权限的标识符组合以仅执行该检查。 |
| ACTRL\_DS\_DELETE\_TREE | 删除DS对象树 |
| ACTRL\_DS\_LIST\_OBJECT | 列出DS对象树 |
| ACTRL\_DS\_CONTROL\_ACCESS | 仅在执行对象支持的扩展权限检查之后才允许访问。该标志可以单独用于对对象执行所有扩展权限检查，也可以与特定扩展权限的标识符组合以仅执行该检查。 |

**请求对对象的访问权限**

当您打开对象的句柄时，返回的句柄具有对该对象的访问权限的某种组合。某些功能（例如`CreateSemaphore`）不需要特定的请求访问权限集。这些功能总是尝试打开手柄以进行完全访问。其他功能，例如`CreateFile`和`OpenProcess`，允许您指定所需的访问权限集。您应该仅请求所需的访问权限，而不是为完全访问打开句柄。这样可以防止意外使用该句柄，并增加了如果对象的DACL仅允许有限的访问，则访问请求成功的机会。

使用通用访问权限来指定打开对象的句柄时所需的访问类型。这通常比指定所有相应的标准和特定权限要简单。或者，使用`MAXIMUM_ALLOWED`常量请求以对调用者有效的所有访问权限打开对象。

**集中授权政策**

动态访问控制（DAC）方案可对企业文件服务器方案进行集中式访问控制管理。大多数组织都希望在多个区域中控制访问权限。

例如：

* 控制对敏感信息的访问，其中标记为敏感的文件将具有特定权限
* 控制对包含个人身份信息（PII）的文件的访问
* 根据组织保留策略限制对文档的访问

提供了几种新的授权策略抽象，以允许管理员集中定义这些策略，并通过允许分别定义和维护这些访问需求中的每一个但作为一个策略应用来简化定义过程。

Windows 8中引入了两个新的`Active Directory`策略对象，即中央授权策略（cap）和中央授权策略规则（capr），以基于声明和资源属性的表达式定义和应用集中式授权策略。在使用这些对象时，管理员将capr定义为特定的授权策略，可以将其应用于具有特定属性或满足特定适用性条件的资源。例如标记为“对业务有重大影响”的文件。在Windows 8 dac表达式方面，可以为可以表达的组织中的每个所需访问控制策略定义capes，并可以标识应该应用的资源。封顶是可一起应用于资源的封顶的集合。下图显示了cap和cape的关系，以及定义和将这些对象应用于文件资源所涉及的概念性步骤。

![relationship of capes and caps](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/cap.png)

**中央授权政策**

中央授权策略（CAP）将特定的授权规则（CAPR）收集到单个策略中。为了将特定的授权规则（CAPR）合并到组织的整体授权策略中，可以将CAPR一起引用，并应用于一组资源。这是通过将多个（通过引用）收集到CAP来完成的。一旦定义了CAP，资源管理器就可以分发它，以将组织授权策略应用于资源。

CAP具有以下属性：

* CAPR的集合–对现有CAPR对象的引用列表
* 标识符（Sid）
* 描述
* 名称

在访问评估期间，对管理员启用了文件和文件夹的CAP进行了评估。在`AccessCheck`呼叫期间，CAP检查在逻辑上与任意的ACL检查结合在一起。这意味着，为了获得对CAP适用的文件的访问权限，用户需要同时具有CAP（与之关联的CAPR）和文件上的任意ACL的访问权限。

CAP 例子：

```text
CORPORATE-FINANCE-CAP]
CAPID=S-1-5-3-4-56-45-67-123
Policies=HBI-CAPE;RETENTION-CAPR
```

CAP的定义

使用ADAC（或`PowerShell`）中的新UX在`Active Directory`中创建和编辑CAP，从而允许管理员创建CAP并指定组成该CAP的一组CAPR。

**中央授权政策规则**

中央授权策略规则（CAPR）的目的是为组织的授权策略的孤立方面提供域范围的定义。管理员定义CAPR以执行特定的授权要求之一。由于CAPR仅定义了授权策略的一个特定的期望要求，因此，与将组织的所有授权策略要求汇总到单个策略定义中相比，可以更简单地定义和理解它。

CAPR具有以下属性：

* 名称–管理员标识CAPR
* 描述-定义CAPR的目的以及CAPR使用者可能需要的任何信息
* 适用性表达 - 定义将在其中应用策略的资源或情况
* ID - 用于审核CAPR更改的标识符
* 有效的访问控制策略 - Windows安全描述符，其中包含定义有效授权策略的DACL
* 异常表达 - 一个或多个表达式，提供了一种方法来覆盖策略并根据表达式的评估来授予对主体的访问权限
* 分期政策 - 可选的Windows安全描述符，其中包含定义了建议的授权策略（访问控制条目列表）的DACL，该授权策略将针对有效策略进行测试，但未执行。如果有效策略的结果与登台策略的结果之间存在差异，则该差异将记录在审核事件日志中。
  * 由于分段可能会对系统性能产生不可预测的影响，因此组策略管理员必须能够选择将在其中生效分段的特定计算机。这允许在OU的大多数计算机上采用现有策略，而在其中一部分计算机上进行暂存。
  * P2 –如果特定计算机上的暂存造成过多的性能下降，则该计算机上的本地管理员应该能够禁用暂存。
* CAP的反向链接–指向可能引用此CAPR的任何CAP的反向链接的列表。

在访问检查期间，将根据适用性表达式评估CAPR的适用性。如果CAPR适用，则需要评估CAPR是否向请求用户提供对标识资源的请求访问权限。然后，AND在逻辑上将CAPE评估的结果与资源上DACL的结果以及对资源有效的任何其他适用的CAPR进行合并。

CAPRs 例子：

```text
[HBI-POLICY]
APPLIES-TO="(@resource.confidentiality == HBI"
SD ="D:(A;;FA;;;AU;(@memberOf("Smartcard Logon")))"
StagingSD = "D:(A;;FA;;;AU;(@memberOf("Smartcard Logon") AND memberOfAny(Resource.ProjectGroups)))"
description="Control access to sensitive information"

[RETENTION-POLICY]
Applies-To="@resource.retention == true"
SD ="D:(A;;;FA;;BA)(A;;FR;;;WD)"
description="If the document is marked for retention, then it is read-only for everyone however Local Admins have 
full control to them to put them out of retention when the time comes"

[TEST-FINANCE-POLICY]
Applies-To="@resource.label == 'finance'"
SD="D:(A;;FA;;;AU;(member_of(FinanceGroup))"
description="Department: Only employees of the finance department should be able to read documents labeled as finance"
```

_CAPE定义_

通过Active Directory管理中心（ADAC）中提供的新UX创建CAPR。在ADAC中，提供了一个新的任务选项来创建CAPR。选择此任务后，ADAC将通过对话框提示用户，要求用户输入CAPR名称和说明。提供这些功能后，将启用用于定义任何其余CAPR元素的控件。对于其余的每个CAPR元素，UX将调出ACL-UI以允许定义表达式和/或ACL。

**安全标识符**

安全标识符（SID）是用于标识受托者的可变长度的唯一值。每个帐户都有一个由权威机构（例如Windows域控制器）颁发的唯一SID，并存储在安全数据库中。每次用户登录时，系统都会从数据库中检索该用户的SID，并将其放在该用户的访问令牌中。系统使用访问令牌中的SID在与Windows安全性的所有后续交互中识别用户。当SID用作用户或组的唯一标识符时，就不能再使用它来标识另一个用户或组。

Windows安全在以下安全元素中使用SID：

* 在安全描述符中标识对象和主要组的所有者
* 在访问控制条目中，标识允许，拒绝或审核访问的受托者
* 在访问令牌中，用于标识用户和该用户所属的组

除了分配给特定用户和组的唯一创建的，特定于域的SID外，还有一些知名的SID用于标识通用组和通用用户。例如，众所周知的SID（每个人和世界）标识包含所有用户的组。

大多数应用程序永远不需要使用SID。因为众所周知的SID的名称可能会有所不同，所以您应该使用函数从预定义的常量构建SID，而不要使用众所周知的SID的名称。例如，美国英语版本的Windows操作系统有一个众所周知的SID，名为“ `BUILTIN \ Administrators`”，在国际版本的系统上可能具有不同的名称。有关构建众所周知的SID的示例，请参见在C ++中搜索访问令牌中的SID。

如果确实需要使用SID，请不要直接操作它们。而是使用以下功能。

| 函数 | Description |
| :--- | :--- |
| [**AllocateAndInitializeSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-allocateandinitializesid) | 使用指定数量的子权限分配和初始化SID |
| [**ConvertSidToStringSid**](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertsidtostringsida) | 将SID转换为适合于显示、存储或传输的字符串格式。 |
| [**ConvertStringSidToSid**](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertstringsidtosida) | 将字符串格式的SID转换为有效的功能性SID |
| [**CopySid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-copysid) | 将源SID复制到缓冲区 |
| [**EqualPrefixSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-equalprefixsid) | 测试两个SID前缀值是否相等。 SID前缀是除最后一个子权限值以外的整个SID |
| [**EqualSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-equalsid) | 测试两个SID是否相等。它们必须完全匹配才能被视为相等 |
| [**FreeSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-freesid) | 通过使用`AllocateAndInitializeSid`函数释放先前分配的SID。 |
| [**GetLengthSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getlengthsid) | 检索SID的长度 |
| [**GetSidIdentifierAuthority**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsididentifierauthority) | 检索指向SID标识符权限的指针 |
| [**GetSidLengthRequired**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsidlengthrequired) | 检索存储具有指定数量的子权限的SID所需的缓冲区大小 |
| [**GetSidSubAuthority**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsidsubauthority) | 检索指向SID中指定的子机构的指针 |
| [**GetSidSubAuthorityCount**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsidsubauthoritycount) | 检索SID中的子机构数. |
| [**InitializeSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-initializesid) | 初始化SID结构 |
| [**IsValidSid**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-isvalidsid) | 通过验证修订号在已知范围内并且子授权机构的数量小于最大数量，来测试SID的有效性 |
| [**LookupAccountName**](https://docs.microsoft.com/en-us/windows/desktop/api/Winbase/nf-winbase-lookupaccountnamea) | 检索与指定帐户名对应的SID |
| [**LookupAccountSid**](https://docs.microsoft.com/en-us/windows/desktop/api/Winbase/nf-winbase-lookupaccountsida) | 检索与指定的SID对应的帐户名 |

**SID组件**

SID值包括提供有关SID结构信息的组件和唯一标识受托者的组件。

SID由以下组件组成：

* SID结构的修订级别
* 一个48位的标识符授权值，用于标识发布SID的授权
* 可变数量的子机构或相对标识符（RID）值，用于相对于发布SID的机构唯一地标识受托人

标识符权限值和子权限值的组合确保即使两个不同的SID颁发机构发布相同的RID值组合，也不会有两个SID相同。每个SID颁发机构仅发出一次给定的RID。

SID以二进制格式存储在SID结构中。要显示SID，可以调用`ConvertSidToStringSid`函数将二进制SID转换为字符串格式。要将SID字符串转换回有效的功能性SID，请调用`ConvertStringSidToSid`函数。

这些函数对SID使用以下标准化的字符串符号，这使得可视化其组件更加简单:

S-_R_-_I_-_S_…

在这种表示法中，文字字符“ S”将一系列数字标识为SID，R是修订级别，I是标识符授权值，S…是一个或多个子授权值。

以下示例使用此表示法来显示本地Administrators组的众所周知的相对于域的SID：

S-1-5-32-544

在此示例中，SID具有以下组件。括号中的常量是Winnt.h中定义的众所周知的标识符权限和RID值：

* 修订级别1
* 标识符授权值为5（`SECURITY_NT_AUTHORITY`）
* 第一子授权值32（`SECURITY_BUILTIN_DOMAIN_RID`）
* 第二个子权限值544（`DOMAIN_ALIAS_RID_ADMINS`）

**知名的SID**

众所周知的安全标识符（SID）标识通用组和通用用户。例如，有一些知名的SID可以标识以下组和用户：

* 每个人或世界，这是一个包括所有用户的组。
* `CREATOR_OWNER`，在可继承ACE中用作占位符。继承ACE后，系统将CREATOR\_OWNER SID替换为对象创建者的SID。
* 本地计算机上内置域的Administrators组。

有通用的知名SID，这些ID在使用此安全模型的所有安全系统上都有意义，包括Windows以外的其他操作系统。此外，还有一些众所周知的SID仅在Windows系统上有意义。

Windows API为众所周知的标识符授权和相对标识符（RID）值定义了一组常量。您可以使用这些常量来创建众所周知的SID。下面的示例结合了`SECURITY_WORLD_SID_AUTHORITY和SECURITY_WORLD_RID`常数，以显示代表所有用户（所有人或世界）的特殊组的通用众所周知的SID：

S-1-1-0

本示例对SID使用字符串表示法，其中S将字符串标识为SID，前1是SID的修订级别，其余两位数字是`SECURITY_WORLD_SID_AUTHORITY`和`SECURITY_WORLD_RID`常数。

您可以使用`AllocateAndInitializeSid`函数通过将标识符授权值与最多八个子授权值组合来构建SID。例如，要确定已登录的用户是否为特定知名组的成员，请调用AllocateAndInitializeSid为该知名组构建一个SID，然后使用EqualSid函数将该SID与该用户中的组SID进行比较。访问令牌。有关示例，请参见在C ++中搜索访问令牌中的SID。您必须调用FreeSid函数以释放由AllocateAndInitializeSid分配的SID。

本节的其余部分包含可用于构建已知SID的知名SID表以及标识符授权和子授权常量表。

以下是一些通用的众所周知的SID。

| known SID | String value | Identifies |
| :--- | :--- | :--- |
| Null SID | S-1-0-0 | 没有成员的群组。当SID值未知时，通常使用此方法。 |
| World | S-1-1-0 | 包括所有用户的组。 |
| Local | S-1-2-0 | 在本地（物理上）登录到系统的终端上登录的用户。 |
| Creator Owner ID | S-1-3-0 | 由创建新对象的用户的安全标识符替换的安全标识符。此SID在可继承ACE中使用。 |
| Creator Group ID | S-1-3-1 | 将由创建新对象的用户的主要组SID替换的安全标识符。在可继承ACE中使用此SID。 |

下表列出了预定义的标识符授权常量。前四个值与通用的众所周知的SID一起使用。最后一个值与Windows众所周知的SID一起使用。

| Identifier authority | Value | String value |
| :--- | :--- | :--- |
| SECURITY\_NULL\_SID\_AUTHORITY | 0 | S-1-0 |
| SECURITY\_WORLD\_SID\_AUTHORITY | 1 | S-1-1 |
| SECURITY\_LOCAL\_SID\_AUTHORITY | 2 | S-1-2 |
| SECURITY\_CREATOR\_SID\_AUTHORITY | 3 | S-1-3 |
| SECURITY\_NT\_AUTHORITY | 5 | S-1-5 |

以下RID值与通用的众所周知的SID一起使用。标识符授权列显示标识符授权的前缀，您可以将其与RID结合使用以创建通用的知名SID

| Relative identifier authority | Value | String value |
| :--- | :--- | :--- |
| SECURITY\_NULL\_RID | 0 | S-1-0 |
| SECURITY\_WORLD\_RID | 0 | S-1-1 |
| SECURITY\_LOCAL\_RID | 0 | S-1-2 |
| SECURITY\_LOCAL\_LOGON\_RID | 1 | S-1-2 |
| SECURITY\_CREATOR\_OWNER\_RID | 0 | S-1-3 |
| SECURITY\_CREATOR\_GROUP\_RID | 1 | S-1-3 |

`SECURITY_NT_AUTHORITY（S-1-5）`预定义的标识符授权产生的SID不是通用的，但仅在Windows安装上才有意义。您可以将以下RID值与`SECURITY_NT_AUTHORITY`一起使用以创建众所周知的SID。

| 常量 | 字符串 | 识别 |
| :--- | :--- | :--- |
| SECURITY\_DIALUP\_RID | S-1-5-1 | 使用拨号调制解调器登录终端的用户。这是一个组身份 |
| SECURITY\_NETWORK\_RID | S-1-5-2 | 跨网络登录的用户。这是在通过网络登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_NETWORK。 |
| SECURITY\_BATCH\_RID | S-1-5-3 | 使用批处理队列工具登录的用户。这是在作为批处理作业记录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_BATCH |
| SECURITY\_INTERACTIVE\_RID | S-1-5-4 | 登录进行交互操作的用户。这是在以交互方式登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_INTERACTIVE。 |
| SECURITY\_LOGON\_IDS\_RID | S-1-5-5-_X_-_Y_ | 登录会话。这用于确保只有给定登录会话中的进程才能访问该会话的窗口站对象。对于每个登录会话，这些SID的X和Y值都不同。值SECURITY\_LOGON\_IDS\_RID\_COUNT是此标识符（5-X-Y）中RID的数量。 |
| SECURITY\_SERVICE\_RID | S-1-5-6 | 授权作为服务登录的帐户。这是在作为服务登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_SERVICE |
| SECURITY\_ANONYMOUS\_LOGON\_RID | S-1-5-7 | 匿名登录，或空会话登录 |
| SECURITY\_PROXY\_RID | S-1-5-8 | 代理. |
| SECURITY\_ENTERPRISE\_CONTROLLERS\_RID | S-1-5-9 | 企业控制器. |
| SECURITY\_PRINCIPAL\_SELF\_RID | S-1-5-10 | The PRINCIPAL\_SELF security identifier can be used in the ACL of a user or group object. During an access check, the system replaces the SID with the SID of the object. The PRINCIPAL\_SELF SID is useful for specifying an inheritable ACE that applies to the user or group object that inherits the ACE. It the only way of representing the SID of a created object in the default [_security descriptor_](https://docs.microsoft.com/en-us/windows/desktop/SecGloss/s-gly) of the schema. |
| SECURITY\_AUTHENTICATED\_USER\_RID | S-1-5-11 | The authenticated users. |
| SECURITY\_RESTRICTED\_CODE\_RID | S-1-5-12 | Restricted code. |
| SECURITY\_TERMINAL\_SERVER\_RID | S-1-5-13 | Terminal Services. Automatically added to the security token of a user who logs on to a terminal server. |
| SECURITY\_LOCAL\_SYSTEM\_RID | S-1-5-18 | A special account used by the operating system. |
| SECURITY\_NT\_NON\_UNIQUE | S-1-5-21 | SIDS are not unique. |
| SECURITY\_BUILTIN\_DOMAIN\_RID | S-1-5-32 | The built-in system domain. |
| SECURITY\_WRITE\_RESTRICTED\_CODE\_RID | S-1-5-33 | Write restricted code. |

以下RID与每个域有关。

| RID | Value | Identifies |
| :--- | :--- | :--- |
| DOMAIN\_ALIAS\_RID\_CERTSVC\_DCOM\_ACCESS\_GROUP | 0x0000023E | 可以使用分布式组件对象模型（DCOM）连接到证书颁发机构的用户组。 |
| DOMAIN\_USER\_RID\_ADMIN | 0x000001F4 | 域中的管理用户帐户。 |
| DOMAIN\_USER\_RID\_GUEST | 0x000001F5 | 域中的来宾用户帐户。没有帐户的用户可以自动登录到该帐户。 |
| DOMAIN\_GROUP\_RID\_ADMINS | 0x00000200 | 域管理员组。该帐户仅存在于运行服务器操作系统的系统上。 |
| DOMAIN\_GROUP\_RID\_USERS | 0x00000201 | 包含域中所有用户帐户的组。所有用户都将自动添加到该组中。 |
| DOMAIN\_GROUP\_RID\_GUESTS | 0x00000202 | 域中的来宾组帐户。 |
| DOMAIN\_GROUP\_RID\_COMPUTERS | 0x00000203 | 域计算机的组。域中的所有计算机都是该组的成员。 |
| DOMAIN\_GROUP\_RID\_CONTROLLERS | 0x00000204 | 域控制器的组。域中的所有DC都是该组的成员。 |
| DOMAIN\_GROUP\_RID\_CERT\_ADMINS | 0x00000205 | 证书发布者组。运行证书服务的计算机是该组的成员 |
| DOMAIN\_GROUP\_RID\_ENTERPRISE\_READONLY\_DOMAIN\_CONTROLLERS | 0x000001F2 | 企业只读域控制器组。 |
| DOMAIN\_GROUP\_RID\_SCHEMA\_ADMINS | 0x00000206 | 模式管理员组。该组的成员可以修改Active Directory架构。 |
| DOMAIN\_GROUP\_RID\_ENTERPRISE\_ADMINS | 0x00000207 | 企业管理员组。该组的成员具有对Active Directory林中所有域的完全访问权限。企业管理员负责林级操作，例如添加或删除新域。 |
| DOMAIN\_GROUP\_RID\_POLICY\_ADMINS | 0x00000208 | 策略管理员组。 |
| DOMAIN\_GROUP\_RID\_READONLY\_CONTROLLERS | 0x00000209 | 只读域控制器组。 |
| DOMAIN\_GROUP\_RID\_CLONEABLE\_CONTROLLERS | 0x0000020A | 可克隆域控制器的组 |
| DOMAIN\_GROUP\_RID\_CDC\_RESERVED | 0x0000020C | 保留的CDC组 |
| DOMAIN\_GROUP\_RID\_PROTECTED\_USERS | 0x0000020D | 受保护的用户组。 |
| DOMAIN\_GROUP\_RID\_KEY\_ADMINS | 0x0000020E | 关键管理员组 |
| DOMAIN\_GROUP\_RID\_ENTERPRISE\_KEY\_ADMINS | 0x0000020F | 企业密钥管理员组 |

以下RID用于指定强制完整性级别。

| RID | Value | Identifies |
| :--- | :--- | :--- |
| SECURITY\_MANDATORY\_UNTRUSTED\_RID | 0x00000000 | Untrusted. |
| SECURITY\_MANDATORY\_LOW\_RID | 0x00001000 | Low integrity. |
| SECURITY\_MANDATORY\_MEDIUM\_RID | 0x00002000 | Medium integrity. |
| SECURITY\_MANDATORY\_MEDIUM\_PLUS\_RID | SECURITY\_MANDATORY\_MEDIUM\_RID + 0x100 | Medium high integrity. |
| SECURITY\_MANDATORY\_HIGH\_RID | 0X00003000 | High integrity. |
| SECURITY\_MANDATORY\_SYSTEM\_RID | 0x00004000 | System integrity. |
| SECURITY\_MANDATORY\_PROTECTED\_PROCESS\_RID | 0x00005000 | Protected process. |

下表提供了相对域RID的示例，您可以使用它们来为本地组（别名）形成众所周知的SID。有关本地和全局组的更多信息，请参见本地组功能和组功能。

| RID | Value | String Value | Identifies |
| :--- | :--- | :--- | :--- |
| DOMAIN\_ALIAS\_RID\_ADMINS | 0x00000220 | S-1-5-32-544 | 用于管理域的本地组。 |
| DOMAIN\_ALIAS\_RID\_USERS | 0x00000221 | S-1-5-32-545 | 代表域中所有用户的本地组 |
| DOMAIN\_ALIAS\_RID\_GUESTS | 0x00000222 | S-1-5-32-546 | 代表域来宾的本地组。 |
| DOMAIN\_ALIAS\_RID\_POWER\_USERS | 0x00000223 | S-1-5-32-547 | 一个本地组，用于代表希望将系统视为其个人计算机而不是多个用户的工作站的用户或一组用户。 |
| DOMAIN\_ALIAS\_RID\_ACCOUNT\_OPS | 0x00000224 | S-1-5-32-548 | 仅在运行服务器操作系统的系统上存在的本地组。此本地组允许控制非管理员帐户。 |
| DOMAIN\_ALIAS\_RID\_SYSTEM\_OPS | 0x00000225 | S-1-5-32-549 | 仅在运行服务器操作系统的系统上存在的本地组。该本地组执行系统管理功能，不包括安全功能。它建立网络共享，控制打印机，解锁工作站并执行其他操作。 |
| DOMAIN\_ALIAS\_RID\_PRINT\_OPS | 0x00000226 | S-1-5-32-550 | 仅在运行服务器操作系统的系统上存在的本地组。此本地组控制打印机和打印队列。 |
| DOMAIN\_ALIAS\_RID\_BACKUP\_OPS | 0x00000227 | S-1-5-32-551 | 一个用于控制文件备份和还原特权的分配本地组。 |
| DOMAIN\_ALIAS\_RID\_REPLICATOR | 0x00000228 | S-1-5-32-552 | 一个本地组，负责将安全数据库从主域控制器复制到备份域控制器。这些帐户仅由系统使用。 |
| DOMAIN\_ALIAS\_RID\_RAS\_SERVERS | 0x00000229 | S-1-5-32-553 | 代表RAS和IAS服务器的本地组。该组允许访问用户对象的各种属性。 |
| DOMAIN\_ALIAS\_RID\_PREW2KCOMPACCESS | 0x0000022A | S-1-5-32-554 | 仅在运行Windows 2000 Server的系统上存在的本地组。有关更多信息，请参见允许匿名访问。 |
| DOMAIN\_ALIAS\_RID\_REMOTE\_DESKTOP\_USERS | 0x0000022B | S-1-5-32-555 | 代表所有远程桌面用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_NETWORK\_CONFIGURATION\_OPS | 0x0000022C | S-1-5-32-556 | 代表网络配置的本地组。 |
| DOMAIN\_ALIAS\_RID\_INCOMING\_FOREST\_TRUST\_BUILDERS | 0x0000022D | S-1-5-32-557 | 代表任何林信任用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_MONITORING\_USERS | 0x0000022E | S-1-5-32-558 | 代表所有受监视用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_LOGGING\_USERS | 0x0000022F | S-1-5-32-559 | 负责记录用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_AUTHORIZATIONACCESS | 0x00000230 | S-1-5-32-560 | 代表所有授权访问的本地组。 |
| DOMAIN\_ALIAS\_RID\_TS\_LICENSE\_SERVERS | 0x00000231 | S-1-5-32-561 | 仅在运行允许终端服务和远程访问的服务器操作系统的系统上存在的本地组。 |
| DOMAIN\_ALIAS\_RID\_DCOM\_USERS | 0x00000232 | S-1-5-32-562 | 一个本地组，代表可以使用分布式组件对象模型（DCOM）的用户。 |
| DOMAIN\_ALIAS\_RID\_IUSERS | 0X00000238 | S-1-5-32-568 | 代表Internet用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_CRYPTO\_OPERATORS | 0x00000239 | S-1-5-32-569 | 一个本地组，代表对密码运算符的访问。 |
| DOMAIN\_ALIAS\_RID\_CACHEABLE\_PRINCIPALS\_GROUP | 0x0000023B | S-1-5-32-571 | 表示可以缓存的主体的本地组。 |
| DOMAIN\_ALIAS\_RID\_NON\_CACHEABLE\_PRINCIPALS\_GROUP | 0x0000023C | S-1-5-32-572 | 表示无法缓存的主体的本地组。 |
| DOMAIN\_ALIAS\_RID\_EVENT\_LOG\_READERS\_GROUP | 0x0000023D | S-1-5-32-573 | 代表事件日志读取器的本地组。 |
| DOMAIN\_ALIAS\_RID\_CERTSVC\_DCOM\_ACCESS\_GROUP | 0x0000023E | S-1-5-32-574 | 可以使用分布式组件对象模型（DCOM）连接到证书颁发机构的本地用户组。 |
| DOMAIN\_ALIAS\_RID\_RDS\_REMOTE\_ACCESS\_SERVERS | 0x0000023F | S-1-5-32-575 | 代表RDS远程访问服务器的本地组。 |
| DOMAIN\_ALIAS\_RID\_RDS\_ENDPOINT\_SERVERS | 0x00000240 | S-1-5-32-576 | 代表端点服务器的本地组。 |
| DOMAIN\_ALIAS\_RID\_RDS\_MANAGEMENT\_SERVERS | 0x00000241 | S-1-5-32-577 | 代表管理服务器的本地组 |
| DOMAIN\_ALIAS\_RID\_HYPER\_V\_ADMINS | 0x00000242 | S-1-5-32-578 | 代表hyper-v管理员的本地组 |
| DOMAIN\_ALIAS\_RID\_ACCESS\_CONTROL\_ASSISTANCE\_OPS | 0x00000243 | S-1-5-32-579 | 代表访问控制辅助OPS的本地组。 |
| DOMAIN\_ALIAS\_RID\_REMOTE\_MANAGEMENT\_USERS | 0x00000244 | S-1-5-32-580 | 代表远程管理用户的本地组。 |
| DOMAIN\_ALIAS\_RID\_DEFAULT\_ACCOUNT | 0x00000245 | S-1-5-32-581 | 代表默认帐户的本地组。 |
| DOMAIN\_ALIAS\_RID\_STORAGE\_REPLICA\_ADMINS | 0x00000246 | S-1-5-32-582 | 代表存储副本管理员的本地组。 |
| DOMAIN\_ALIAS\_RID\_DEVICE\_OWNERS | 0x00000247 | S-1-5-32-583 | 代表的本地组可以为设备所有者进行预期的设置。 |

`WELL_KNOWN_SID_TYPE`枚举定义了常用SID的列表。此外，安全描述符定义语言（SDDL）使用SID字符串以字符串格式引用众所周知的SID。

#### 访问控制如何工作

当线程尝试访问安全对象时，系统会授予或拒绝访问。如果对象没有任意访问控制列表（DACL），则系统授予访问权限；否则，系统将授予访问权限。否则，系统将在对象的DACL中查找适用于该线程的访问控制项（ACE）。对象的DACL中的每个ACE都指定受托者允许或拒绝的访问权限，可以是用户帐户、组帐户或登录会话。

**DACLs**

系统将每个ACE中的受托者与线程的访问令牌中标识的受托者进行比较。访问令牌包含安全标识符（SID），用于标识用户和用户所属的组帐户。令牌还包含用于标识当前登录会话的登录SID。在访问检查期间，系统将忽略未启用的组SID。有关启用，禁用和仅拒绝SID的更多信息，请参阅访问令牌中的SID属性。

通常，系统使用请求访问的线程的主要访问令牌。但是，如果线程模拟其他用户，则系统将使用线程的模拟令牌。

系统依次检查每个ACE，直到发生以下事件之一：

* 拒绝访问的ACE明确拒绝对线程访问令牌中列出的一个受托者的任何请求的访问权限。
* 线程的访问令牌中列出的针对受托者的一个或多个允许访问的ACE明确授予所有请求的访问权限。
* 已检查所有ACE，并且仍然存在至少一个未明确允许的请求访问权限，在这种情况下，访问被隐式拒绝。

下图显示了对象的DACL如何允许访问一个线程而拒绝访问另一个线程。

![dacl that grants different access rights to different threads](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl1.png)

对于线程A，系统将读取ACE 1并立即拒绝访问，因为拒绝访问的ACE适用于线程访问令牌中的用户。在这种情况下，系统将不检查ACE 2和ACE3。对于线程B，ACE 1不适用，因此系统进入允许写入访问的ACE 2和允许读取和执行访问的ACE 3。

因为当明确授予或拒绝所请求的访问权限时系统会停止检查ACE，所以DACL中ACE的顺序很重要。请注意，如果示例中的ACE顺序不同，则系统可能已授予访问线程A的权限。对于系统对象，操作系统在DACL中定义了ACE的首选顺序。

#### 线程与安全对象之间的交互

当线程尝试使用安全对象时，系统会在允许线程继续进行之前执行访问检查。在访问检查中，系统将线程访问令牌中的安全信息与对象的安全描述符中的安全信息进行比较。

* 访问令牌包含安全标识符（SID），用于标识与线程关联的用户。
* 安全描述符标识对象的所有者，并包含一个自由访问控制列表（DACL）。 DACL包含访问控制项（ACE），每个访问控制项都指定对特定用户或组允许或拒绝的访问权限。

系统检查对象的DACL，从线程的访问令牌中查找适用于用户的ACE和组SID。系统将检查每个ACE，直到授予访问权限或拒绝访问为止，或者直到不再有要检查的ACE。可以想象，访问控制列表（ACL）可以具有多个应用于令牌的SID的ACE。并且，如果发生这种情况，则会累积每个ACE授予的访问权限。例如，如果一个ACE授予对组的读访问权限，而另一个ACE授予对组成员的用户的写访问权限，则该用户可以同时具有对该对象的读和写访问权限。

下图显示了这些安全信息块之间的关系：

![relationships between processes, aces, and dacls](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/cssec-02.png)

#### DACLs和ACEs

如果Windows对象没有任意访问控制列表（DACL），则系统允许所有人对其进行完全访问。如果对象具有DACL，则系统仅允许DACL中的访问控制项（ACE）明确允许的访问。如果DACL中没有ACE，则系统不允许访问任何人。同样，如果DACL具有允许访问有限的一组用户或组的ACE，则系统暗中拒绝访问未包括在ACE中的所有受托者。

在大多数情况下，可以使用允许访问的ACE控制对对象的访问。您无需明确拒绝访问对象。例外是当ACE允许访问组并且您要拒绝对组成员的访问时。为此，将用户拒绝访问的ACE放置在DACL中，然后放置在该组允许访问的ACE之前。请注意，ACE的顺序很重要，因为系统会按顺序读取ACE，直到允许或拒绝访问为止。用户的拒绝访问的ACE必须首先出现；否则，当系统读取该组的访问允许的ACE时，它将向受限用户授予访问权限。

下图显示了DACL，该DACL拒绝对一个用户的访问并向两个组授予访问权限。 A组的成员通过累积允许给A组的权限和允许给所有人的权限来获得读取，写入和执行访问权限。 Andrew是一个例外，尽管他是Everyone Group的成员，但拒绝访问的ACE拒绝了他的访问。![dacl that grants differing access rights based on group membership](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl1.png)

#### 空DACL和空DACL（授权）

如果属于对象的安全描述符的自由访问控制列表（DACL）设置为NULL，则会创建一个空DACL。空DACL授予对请求它的任何用户的完全访问权限；不对该对象执行正常的安全检查。空的DACL不应与空的DACL混淆。空的DACL是正确分配和初始化的DACL，其中不包含访问控制项（ACE）。空的DACL不允许访问对其分配的对象。

#### 允许匿名访问

默认安全策略将匿名本地访问限制为没有权限。然后，管理员可以根据需要添加或减少权限。

对于具有与所有人相同访问权限的应用程序，存在本地访问组。然后，管理员可以适当地增加或减少该组中的用户数量，该组名为Windows 2000之前的兼容访问组。

### 

### 

