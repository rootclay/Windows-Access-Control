# （一）Access Token

访问控制模型提供了可以控制进程访问安全对象或执行各种系统管理任务的能力。

## 访问控制模型的组成

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

#### 

## 访问令牌（Access tokens）

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

### **受限令牌**

受限令牌是已由`CreateRestrictedToken`函数修改的主要或模拟访问令牌。在受限令牌的安全上下文中运行的进程或模拟线程在访问安全对象或执行特权操作方面的能力受到限制。 `CreateRestrictedToken`函数可以通过以下方式限制令牌：

* 从令牌中删除特权。
* 将“仅拒绝”属性应用于令牌中的SID，以便它们不能用于访问受保护的对象。有关“仅拒绝”属性的更多信息，请参阅访问令牌中的SID属性\([https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-attributes-in-an-access-token\)。](https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-attributes-in-an-access-token%29。)
* 指定限制SID的列表，这可以限制对安全对象的访问。

当系统检查令牌对安全对象的访问时，系统将使用限制SID列表。当受限制的进程或线程尝试访问安全对象时，系统执行两项访问检查：一项使用令牌启用的SID，另一项使用限制SID列表。仅当两个访问检查都允许所请求的访问权限时，才授予访问权限。有关访问检查的更多信息，请参见DACL如何控制对对象的访问: [How DACLs Control Access to an Object](https://docs.microsoft.com/en-us/windows/win32/secauthz/how-dacls-control-access-to-an-object)

您可以在对`CreateProcessAsUser`函数的调用中使用受限制的主令牌。通常调用`CreateProcessAsUser`的进程必须具有`SE_ASSIGNPRIMARYTOKEN_NAME`特权，该特权通常仅由系统代码或`LocalSystem`帐户中运行的服务保留。但是，如果`CreateProcessAsUser`调用指定了调用者主令牌的受限版本，则不需要此特权。这使普通应用程序可以创建受限进程。

您还可以在`ImpersonateLoggedOnUser`函数中使用受限制的主要或模拟令牌。

若要确定令牌是否具有限制SID列表，请调用`IsTokenRestricted`函数。

注意：使用受限令牌的应用程序应在默认桌面以外的其他桌面上运行受限应用程序, 防止受限制的应用程序使用`SendMessage`或`PostMessage`攻击默认桌面上不受限制的应用程序。如有必要，请根据您的应用程序在桌面之间切换。

### **访问令牌中的SID属性**

访问令牌中的每个用户和组安全标识符（SID）具有一组属性，这些属性控制系统在访问检查中如何使用SID。下表列出了控制访问检查的属性。

| 属性 | 描述 |
| :---: | :---: |
| SE\_GROUP\_ENABLED | 启用具有此属性的SID进行访问检查。当系统执行访问检查时，它将检查适用于访问令牌中已启用的SID之一的允许访问和拒绝访问的访问控制项（ACE）。除非设置了`SE_GROUP_USE_FOR_DENY_ONLY`属性，否则在访问检查期间将忽略不具有此属性的SID。 |
| SE\_GROUP\_USE\_FOR\_DENY\_ONLY | 具有此属性的SID是仅拒绝的SID。当系统执行访问检查时，它会检查适用于SID的拒绝访问的ACE，但是会忽略SID允许访问的ACE。如果设置了此属性，则不会设置SE\_GROUP\_ENABLED属性，并且无法重新启用SID |

要设置或清除组SID的`SE_GROUP_ENABLED`属性，请使用`AdjustTokenGroups`函数。您不能禁用具有`SE_GROUP_MANDATORY`属性的组SID。您不能使用`AdjustTokenGroups`禁用访问令牌的用户SID。

若要确定令牌中是否启用了SID，即它是否具有`SE_GROUP_ENABLED`属性，请调用`CheckTokenMembership`函数。

若要设置SID的`SE_GROUP_USE_FOR_DENY_ONLY`属性，请在调用`CreateRestrictedToken`函数时指定的拒绝专用SID列表中包含的该SID。 `CreateRestrictedToken`可以将`SE_GROUP_USE_FOR_DENY_ONLY`属性应用于任何SID，包括用户SID和具有`SE_GROUP_MANDATORY`属性的组SID。但是，您不能从SID中删除仅拒绝属性，也不能使用`AdjustTokenGroups`在仅拒绝SID上设置`SE_GROUP_ENABLED`属性。

要获取SID的属性，请使用`TokenGroups`值调用`GetTokenInformation`函数。该函数返回一个`SID_AND_ATTRIBUTES`结构数组，该结构标识组SID及其属性。

### **访问令牌对象的访问权限**

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

## \*\*\*\*

## \*\*\*\*

