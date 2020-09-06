# （七）安全标识符

## **安全标识符**

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

### **SID组件**

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

### **知名的SID**

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

## 

