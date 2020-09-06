# 3. 权限控制其他内容

### 权限

特权是帐户（例如用户帐户或组帐户）在本地计算机上执行各种与系统相关的操作（例如关闭系统，加载设备驱动程序或更改系统时间）的权利。特权与访问权限在两个方面有所不同：

* 特权控制对系统资源和系统相关任务的访问，而访问权限控制对安全对象的访问。
* 系统管理员为用户帐户和组帐户分配特权，而系统则根据对象的DACL中ACE中授予的访问权限来授予或拒绝对安全对象的访问。

每个系统都有一个帐户数据库，用于存储用户帐户和组帐户所拥有的特权。当用户登录时，系统会生成一个访问令牌，其中包含用户特权的列表，包括授予用户或用户所属组的特权。请注意，特权仅适用于本地计算机。域帐户在不同的计算机上可以具有不同的特权。

当用户尝试执行特权操作时，系统检查用户的访问令牌以确定该用户是否拥有必要的特权，如果是，则检查是否启用了特权。如果用户未通过这些测试，则系统不会执行该操作。

要确定访问令牌中保留的特权，请调用`GetTokenInformation`函数，该函数还指示启用了哪些特权。默认情况下，大多数特权是禁用的。

Windows API定义了一组字符串常量，例如`SE_ASSIGNPRIMARYTOKEN_NAME`，以标识各种特权。这些常量在所有系统上都相同，并在`Winnt.h`中定义。有关`Windows`定义的特权的表，请参阅特权常数。但是，获取和调整访问令牌中的特权的函数使用`LUID`类型来标识特权。一台计算机的特权的`LUID`值可能会不同，而同一台计算机上的一次引导到另一台计算机也会有所不同。若要获取与字符串常量之一相对应的当前LUID，请使用`LookupPrivilegeValue`函数。使用`LookupPrivilegeName`函数将LUID转换为其相应的字符串常量。

系统提供了一组描述每个权限的显示名称。当您需要向用户显示特权说明时，这些功能很有用。使用`LookupPrivilegeDisplayName`函数可检索与特权字符串常量相对应的描述字符串。例如，在使用美国英语的系统上，`SE_SYSTEMTIME_NAME`特权的显示名称为“更改系统时间”。

您可以使用`PrivilegeCheck`函数来确定访问令牌是否持有指定的特权集。这主要用于模拟客户端的服务器应用程序。

系统管理员可以使用管理工具（例如用户管理器）来添加或删除用户帐户和组帐户的特权。管理员可以以编程方式使用本地安全机构（LSA）功能来使用特权。 `LsaAddAccountRights`和`LsaRemoveAccountRights`函数可从帐户添加或删除特权。`LsaEnumerateAccountRights`函数枚举指定帐户所拥有的特权。 `LsaEnumerateAccountsWithUserRight`函数枚举具有指定特权的帐户。

### 审核生成

C2级安全性要求指定系统管理员必须能够审核与安全性有关的事件，并且对此审核数据的访问必须仅限于授权管理员。 Windows API提供了使管理员能够监视与安全性有关的事件的功能。

可保护对象的安全描述符可以具有系统访问控制列表（SACL）。 SACL包含访问控制条目（ACE），它们指定生成审核报告的访问尝试的类型。每个ACE标识一个受托者，一组访问权限和一组标志，这些标志指示系统是否为失败的访问尝试和/或成功的访问尝试生成审核消息。

系统将审核消息写入安全事件日志。有关访问安全事件日志中的记录的信息，请参阅事件日志。

要读取或写入对象的`SACL`，线程必须首先启用`SE_SECURITY_NAME`特权。有关更多信息，请参见`SACL`访问权限。

### 安全对象

安全对象是可以具有安全描述符的对象。所有命名的`Windows`对象都是安全的。一些未命名的对象（例如进程和线程对象）也可以具有安全描述符。对于大多数安全对象，可以在创建对象的函数调用中指定对象的安全描述符。例如，您可以在`CreateFile`和`CreateProcess`函数中指定安全描述符。

此外，Windows安全功能使您能够获取和设置在Windows以外的操作系统上创建的可安全对象的安全信息。 Windows安全功能还支持将安全描述符与专用的，应用程序定义的对象一起使用。有关私有安全对象的更多信息，请参见客户端/服务器访问控制。

每种类型的可保护对象都定义了自己的一组特定访问权限，并定义了自己的通用访问权限映射。有关每种安全对象的特定和通用访问权限的信息，请参阅该对象的概述。

下表显示了用于处理某些常见安全对象的安全信息的功能。

| 对象类型 | 安全描述符函数 |
| :--- | :--- |
| [Files or directories](https://docs.microsoft.com/en-us/windows/desktop/FileIO/file-security-and-access-rights) on an NTFS file system | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Named pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipe-security-and-access-rights) [Anonymous pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/anonymous-pipe-security-and-access-rights) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Processes](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/process-security-and-access-rights) [Threads](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/thread-security-and-access-rights) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [File-mapping objects](https://docs.microsoft.com/en-us/windows/desktop/Memory/file-mapping-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Access tokens](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/access-rights-for-access-token-objects) | [**SetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setkernelobjectsecurity), [**GetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getkernelobjectsecurity) |
| Window-management objects \([window stations](https://docs.microsoft.com/en-us/windows/desktop/winstation/window-station-security-and-access-rights) and [desktops](https://docs.microsoft.com/en-us/windows/desktop/winstation/desktop-security-and-access-rights)\) | [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Registry keys](https://docs.microsoft.com/en-us/windows/desktop/SysInfo/registry-key-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Windows services](https://docs.microsoft.com/en-us/windows/desktop/Services/service-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| Local or remote printers | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| Network shares | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Interprocess synchronization objects](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights) \(events, mutexes, semaphores, and waitable timers\) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |
| [Job objects](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/job-object-security-and-access-rights) | [**GetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getnamedsecurityinfoa), [**SetNamedSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setnamedsecurityinfoa), [**GetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-getsecurityinfo), [**SetSecurityInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/Aclapi/nf-aclapi-setsecurityinfo) |

### 低层访问控制

低级安全功能可帮助您使用安全描述符，访问控制列表（ACL）和访问控制项（ACE）。

有关模型的说明，请参阅访问控制模型。

| 主题 | 描述 |
| :--- | :--- |
| [Low-level Security Descriptor Functions](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/low-level-security-descriptor-functions) | 用于设置和检索对象的安全描述符的函数。 |
| [Low-level Security Descriptor Creation](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/low-level-security-descriptor-creation) | 用于创建安全描述符以及获取和设置安全描述符的组件的函数。 |
| [Absolute and Self-Relative Security Descriptors](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/absolute-and-self-relative-security-descriptors) | 用于在绝对或自相关格式之间进行检查或转换的功能。 |
| [Low-level ACL and ACE Functions](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/low-level-acl-and-ace-functions) | 用于管理ACL和ACE的功能。 |

#### 低级安全描述符函数

有几对低级函数，用于设置和检索对象的安全描述符。这些对中的每对仅适用于一组有限的Windows对象。例如，一对使用文件对象，另一对使用注册表项。下表显示了与不同类型的可保护对象一起使用的低级函数。

| 对象类型 | 低级函数 |
| :--- | :--- |
| [Files](https://docs.microsoft.com/en-us/windows/desktop/FileIO/file-security-and-access-rights)    [Directories](https://docs.microsoft.com/en-us/windows/desktop/FileIO/file-security-and-access-rights)    Mailslots    [Named pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipe-security-and-access-rights) | 使用`GetFileSecurity`和`SetFileSecurity`函数。这些函数使用字符串标识安全对象，而不是使用句柄。 |
| [Processes](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/process-security-and-access-rights)     [Threads](https://docs.microsoft.com/en-us/windows/desktop/ProcThread/thread-security-and-access-rights)    [Access tokens](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/access-rights-for-access-token-objects)    [File-mapping objects](https://docs.microsoft.com/en-us/windows/desktop/Memory/file-mapping-security-and-access-rights)    [Semaphores](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights)  [Events](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights)     [Mutexes](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights)    [Waitable timers](https://docs.microsoft.com/en-us/windows/desktop/Sync/synchronization-object-security-and-access-rights) | 使用 [**GetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-getkernelobjectsecurity) and [**SetKernelObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-setkernelobjectsecurity) 函数. |
| [Window stations](https://docs.microsoft.com/en-us/windows/desktop/winstation/window-station-security-and-access-rights)    [Desktops](https://docs.microsoft.com/en-us/windows/desktop/winstation/desktop-security-and-access-rights) | 使用 [**GetUserObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winuser/nf-winuser-getuserobjectsecurity) and [**SetUserObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winuser/nf-winuser-setuserobjectsecurity) 函数. |
| [Registry keys](https://docs.microsoft.com/en-us/windows/desktop/SysInfo/registry-key-security-and-access-rights) | 使用 [**RegGetKeySecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winreg/nf-winreg-reggetkeysecurity) and [**RegSetKeySecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winreg/nf-winreg-regsetkeysecurity) 函数. |
| [Windows service objects](https://docs.microsoft.com/en-us/windows/desktop/Services/service-security-and-access-rights) | 使用 [**QueryServiceObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winsvc/nf-winsvc-queryserviceobjectsecurity) and [**SetServiceObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/Winsvc/nf-winsvc-setserviceobjectsecurity) 函数. |
| Printer objects | 使用 [**PRINTER\_INFO\_2**](https://docs.microsoft.com/en-us/windows/desktop/printdocs/printer-info-2) 结构 在 [**GetPrinter**](https://docs.microsoft.com/en-us/windows/desktop/printdocs/getprinter) and [**SetPrinter**](https://docs.microsoft.com/en-us/windows/desktop/printdocs/setprinter) 函数中. |
| [Network shares](https://docs.microsoft.com/en-us/windows/desktop/NetMgmt/security-requirements-for-the-network-management-functions) | 使用502级别的 [**NetShareGetInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/lmshare/nf-lmshare-netsharegetinfo) and [**NetShareSetInfo**](https://docs.microsoft.com/en-us/windows/desktop/api/lmshare/nf-lmshare-netsharesetinfo) 函数. |
| [Private objects \(objects private to the creating application\)](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/acl-based-access-control) | 使用 [**CreatePrivateObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-createprivateobjectsecurity), [**DestroyPrivateObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-destroyprivateobjectsecurity), [**GetPrivateObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-getprivateobjectsecurity) and [**SetPrivateObjectSecurity**](https://docs.microsoft.com/en-us/windows/desktop/api/securitybaseapi/nf-securitybaseapi-setprivateobjectsecurity) 函数. |

#### 低级安全描述符创建

低级访问控制提供了一组用于创建安全描述符以及获取和设置安全描述符的组件的功能。用于初始化和设置安全描述符的组件的低级功能仅适用于绝对格式的安全描述符。用于获取安全描述符的组件的低级功能可同时使用绝对和自相关的安全描述符。

`InitializeSecurityDescriptor`函数初始化`SECURITY_DESCRIPTOR`缓冲区。初始化的安全描述符为绝对格式，没有所有者，主组，任意访问控制列表（DACL）或系统访问控制列表（SACL）。您可以使用以下低级函数来获取或设置指定安全描述符的特定组件。

| 函数 | 描述 |
| :--- | :--- |
| [**GetSecurityDescriptorControl**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptorcontrol) | 从安全描述符中检索修订和控制信息。 |
| [**GetSecurityDescriptorDacl**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptordacl) | 从安全描述符中检索DACL。 |
| [**GetSecurityDescriptorGroup**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptorgroup) | 从安全描述符中检索主组安全标识符（SID）。 |
| [**GetSecurityDescriptorLength**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptorlength) | 返回安全描述符的长度。 |
| [**GetSecurityDescriptorOwner**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptorowner) | 从安全描述符中检索所有者SID。 |
| [**GetSecurityDescriptorSacl**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-getsecuritydescriptorsacl) | 从安全描述符中检索SACL。 |
| [**SetSecurityDescriptorDacl**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setsecuritydescriptordacl) | 将DACL放入安全描述符中，取代任何现有的DACL。 |
| [**SetSecurityDescriptorGroup**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setsecuritydescriptorgroup) | 设置安全描述符的主要组SID。 |
| [**SetSecurityDescriptorOwner**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setsecuritydescriptorowner) | 设置安全描述符的所有者SID。 |
| [**SetSecurityDescriptorSacl**](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-setsecuritydescriptorsacl) | 将SACL放入安全描述符中，以取代任何现有的SACL。 |

**绝对和自相关的安全描述符**

安全描述符可以采用绝对或自相关格式。以绝对格式、安全描述符包含指向其信息的指针，而不是信息本身的指针。以自相关格式、安全描述符将`SECURITY_DESCRIPTOR`结构和关联的安全信息存储在连续的内存块中。要确定安全描述符是自相关的还是绝对的，请调用`GetSecurityDescriptorControl`函数并检查`SECURITY_DESCRIPTOR_CONTROL`参数的`SE_SELF_RELATIVE`标志。您可以使用`MakeSelfRelativeSD`和`MakeAbsoluteSD`函数在这两种格式之间进行转换。

当您构建安全描述符并具有指向所有组件的指针时，例如，当所有者、组和任意ACL的默认设置可用时，绝对格式很有用。在这种情况下，您可以调用`InitializeSecurityDescriptor`函数来初始化`SECURITY_DESCRIPTOR`结构，然后调用诸如`SetSecurityDescriptorDacl`之类的函数来将ACL和SID指针分配给安全描述符。

在自相关格式中，安全描述符始终以`SECURITY_DESCRIPTOR`结构开头，但是安全描述符的其他组件可以按任何顺序跟随该结构。安全描述符的组成部分不使用内存地址，而是通过从描述符开始处的偏移量来标识的。当安全描述符必须存储在磁盘上，通过通信协议传输或复制到内存中时，此格式很有用。

除了`MakeAbsoluteSD`，所有返回安全描述符的函数都使用自相关格式。作为参数传递给函数的安全描述符可以是自相关形式，也可以是绝对形式。有关更多信息，请参阅该功能的文档。

**底层ACL和ACE功能**

要使用低级功能创建访问控制列表（ACL），请为ACL分配一个缓冲区，然后通过调用`InitializeAcl`函数对其进行初始化。要将访问控制条目（ACE）添加到任意访问控制列表（DACL）的末尾，请使用`AddAccessAllowedAce`和`AddAccessDeniedAce`函数。 `AddAuditAccessAce`函数将ACE添加到系统访问控制列表（SACL）的末尾。您可以使用AddAce函数在ACL中的指定位置添加一个或多个ACE。 `AddAce`函数还允许您将可继承的ACE添加到ACL。 `DeleteAce`函数从ACL中的指定位置删除ACE。 `GetAce`函数从ACL中的指定位置检索ACE。 `FindFirstFreeAce`函数检索指向ACL中第一个空闲字节的指针。

要修改对象的安全描述符中的现有ACL，请使用`GetSecurityDescriptorDacl`或`GetSecurityDescriptorSacl`函数来获取现有ACL。您可以使用`GetAce`函数从现有ACL复制ACE。分配并初始化新的ACL后，请使用`AddAccessAllowedAce`和`AddAce`之类的功能向其添加ACE。完成新ACL的构建后，请使用`SetSecurityDescriptorDacl或SetSecurityDescriptorSacl`函数将新ACL添加到对象的安全描述符中。

您可以使用`AddAccessAllowedObjectAce`、`AddAccessDeniedObjectAce`或`AddAuditAccessObjectAce`函数将特定于对象的ACE添加到ACL的末尾。

