# （五）访问权限与权限码

## **访问权限和访问掩码**

访问权限是一个位标志，它对应于线程可以在可保护对象上执行的一组特定操作。例如，注册表项具有`KEY_SET_VALUE`访问权限，这对应于线程在项下设置值的能力。如果线程尝试对某个对象执行操作，但没有对该对象的必要访问权限，则系统不会执行该操作。

访问掩码是一个32位的值，其位与对象支持的访问权限相对应。所有Windows安全对象均使用访问掩码格式，该格式包含用于以下类型的访问权限的位：

* 通用访问权限
* 标准访问权限
* SACL访问权限
* 目录服务（域）访问权限

当线程尝试打开对象的句柄时，该线程通常会指定访问掩码以请求一组访问权限。例如，需要设置和查询注册表项的值的应用程序可以通过使用访问掩码来请求`KEY_SET_VALUE`和`KEY_QUERY_VALUE`访问权限来打开注册表项。

下表显示了用于处理每种安全对象类型的安全信息的功能。这就是前两节ACL修改中会使用到的对象，以及其安全信息的资料。

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

### **访问掩码格式**

所有安全对象都使用下图所示的访问掩码格式来安排其访问权限。

![access mask format](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl4.png)

在这种格式中，低16位用于特定对象的访问权限，后8位用于标准访问权限，这些权限适用于大多数类型的对象，而4个高位用于指定通用访问权限每种对象类型可以映射到一组标准和特定于对象的权限。 `ACCESS_SYSTEM_SECURITY`位对应于访问对象的SACL的权限。

### **通用访问权限**

全对象使用访问掩码格式，其中四个高位指定通用访问权限。每种可保护对象的类型都将这些位映射到一组其标准和特定于对象的访问权限。（也就是说通用权限是由标准位和特殊位映射出来的）例如，Windows文件对象将`GENERIC_READ`位映射到`READ_CONTROL`和`SYNCHRONIZE`标准访问权限以及`FILE_READ_DATA`，`FILE_READ_EA`和`FILE_READ_ATTRIBUTES`这三个对象特定的访问权限。其他类型的对象将`GENERIC_READ`位映射到适合该类型对象的任何访问权限集。

可以使用通用访问权限来指定打开对象的句柄时所需的访问类型。会比指定所有相应的标准和特定权限要简单。

下表显示了为通用访问权限定义的常量。

| 常量 | 通用含义 |
| :--- | :--- |
| GENERIC\_ALL | All possible access rights |
| GENERIC\_EXECUTE | Execute access |
| GENERIC\_READ | Read access |
| GENERIC\_WRITE | Write access |

### **标准访问权限**

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

### **SACL访问权限**

`ACCESS_SYSTEM_SECURITY`访问权限控制在对象的安全描述符中获取或设置SACL的能力。仅当在请求线程的访问令牌中启用`SE_SECURITY_NAME`特权时，系统才会授予此访问权限。

访问对象的SACL :

1. 调用`AdjustTokenPrivileges`函数以启用`SE_SECURITY_NAME`特权。
2. 打开对象的句柄时，请求`ACCESS_SYSTEM_SECURITY`访问权限。
3. 通过使用诸如`GetSecurityInfo`或`SetSecurityInfo`之类的函数来获取或设置对象的SACL。
4. 调用`AdjustTokenPrivileges`以禁用`SE_SECURITY_NAME`特权

要使用`GetNamedSecurityInfo`或`SetNamedSecurityInfo`函数访问SACL，请启用`SE_SECURITY_NAME`特权。该函数在内部请求访问权限。

`ACCESS_SYSTEM_SECURITY`访问权限在DACL中无效，因为DACL不控制对SACL的访问。但是，您可以在SACL中使用`ACCESS_SYSTEM_SECURITY`访问权限来审核使用该访问权限的尝试。

### **域服务访问权限**

每个`Active Directory`对象都有一个分配给它的安全描述符。可以在这些安全描述符中设置特定于目录服务对象的一组受托者权限。下表列出了这些权利也对应域服务ACE的权限图。

![&#x57DF;&#x670D;&#x52A1;ACE&#x7F16;&#x8F91;&#x5668;](.gitbook/assets/image%20%2814%29.png)

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

### **请求对对象的访问权限**

当您打开对象的句柄时，返回的句柄具有对该对象的访问权限的某种组合。某些功能（例如`CreateSemaphore`）不需要特定的请求访问权限集。这些功能总是尝试打开手柄以进行完全访问。其他功能，例如`CreateFile`和`OpenProcess`，允许您指定所需的访问权限集。您应该仅请求所需的访问权限，而不是为完全访问打开句柄。这样可以防止意外使用该句柄，并增加了如果对象的DACL仅允许有限的访问，则访问请求成功的机会。

使用通用访问权限来指定打开对象的句柄时所需的访问类型。这通常比指定所有相应的标准和特定权限要简单。或者，使用`MAXIMUM_ALLOWED`常量请求以对调用者有效的所有访问权限打开对象。

