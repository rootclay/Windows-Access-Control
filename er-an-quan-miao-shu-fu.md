---
description: 安全描述符，与被访问对象关联。由对象所有者的SID、所属组的SID、DACL、SACL、一组控制Bit组成。
---

# （二）安全描述符（SD）

## **安全描述符\(**Security Descriptors,SD**\)**

安全描述符是与被访问对象关联的，它含有这个对象所有者的SID，以及一个访问控制列表（ACL，Access Control List），访问控制列表又包括了DACL（Discretionary Access Control List）和SACL（System Access Control List）以及一组控制位，用于限定安全描述符含义。

安全描述符包含与安全对象关联的安全信息（安全对象是可以具有安全描述符的对象。比如文件、管道、进程、注册表等）。安全描述符由`SECURITY_DESCRIPTOR`结构及其关联的安全信息组成。安全描述符的结构体如下：

```text
typedef struct _SECURITY_DESCRIPTOR {
  BYTE                        Revision;
  BYTE                        Sbz1;
  SECURITY_DESCRIPTOR_CONTROL Control;
  PSID                        Owner;
  PSID                        Group;
  PACL                        Sacl;
  PACL                        Dacl;
} SECURITY_DESCRIPTOR, *PISECURITY_DESCRIPTOR;
```

安全描述符可以包括以下安全信息：

* 对象的所有者和所属组的安全标识符（SID）
* DACL：它里面包含零个（可以为0，之后会有详细介绍）或多个访问控制项（ACE，Access Control Entry），每个访问控制项的内容描述了允许或拒绝特定账户对这个对象执行特定操作。
* SACL：主要是用于系统审计的，它的内容指定了当特定账户对这个对象执行特定操作时，记录到系统日志中。
* 一组控制位，用于限定安全描述符或其单个成员的含义

应用程序不能直接操纵安全描述符的内容。 Windows API提供了用于在对象的安全描述符中设置和检索安全信息的功能。此外，还有用于创建和初始化新对象的安全描述符的函数。

在`Active Directory`对象上使用安全描述符的应用程序可以使用Windows安全功能或`Active Directory`服务接口（ADSI）提供的安全接口。有关ADSI安全接口的详细信息，我会在之后的系列中详细介绍。

### **如何使用API操作安全描述符**

Windows API提供了用于获取和设置与安全对象关联的安全描述符的组件的功能。使用`GetSecurityInfo`和`GetNamedSecurityInfo`函数检索指向对象的安全描述符的指针。这些函数还可以检索指向安全描述符的各个组件的指针：DACL、SACL、owner SID和主要组SID。使用`SetSecurityInfo`和`SetNamedSecurityInfo`函数来设置对象的安全描述符的组件。`SetSecurityInfo`的结构如下所示：

```text
DWORD SetSecurityInfo(
  HANDLE               handle,
  SE_OBJECT_TYPE       ObjectType,
  SECURITY_INFORMATION SecurityInfo,
  PSID                 psidOwner,
  PSID                 psidGroup,
  PACL                 pDacl,
  PACL                 pSacl
);
```

通常，应将`GetSecurityInfo`和`SetSecurityInfo`与通过句柄标识的对象一起使用，将`SetNamedSecurityInfo`和`GetNamedSecurityInfo`与通过名称标识的对象一起使用。

要在安全描述符中获取控制信息，可调用`GetSecurityDescriptorControl`函数。要设置与自动ACE继承相关的控制位，可调用`SetSecurityDescriptorControl`函数。其他控制位由设置安全描述符组件的各种功能设置。例如，如果使用`SetSecurityInfo`更改对象的DACL，则该函数将适当地设置或清除位，以指示安全描述符是否具有DACL，是否为默认DACL，等等。另一个示例是安全描述符中包含的资源管理器（RM）控制位。这些位根据资源管理器的实现使用，并通过`GetSecurityDescriptorRMControl`和`SetSecurityDescriptorRMControl`函数进行访问。

### **创建新对象时引入安全描述符**

创建安全对象时，可以将安全描述符分配给新对象。用于创建安全对象的函数（例如`CreateFile`或`RegCreateKeyEx`）具有指向`SECURITY_ATTRIBUTES`结构的参数，该结构可以包含指向新对象的安全描述符的指针。

管理对象的系统组件或服务器可以存储指定的或默认的安全描述符，以使其成为对象的持久属性。如果对象的创建者未指定安全描述符，系统将继承或使用默认的安全信息来创建安全描述符。可以使用函数来更改对象的安全描述符中的信息。（这就是一开始讲到的默认安全描述符）

域内的对象、文件、目录、注册表项和桌面是安全对象，可以具有父对象。创建这些对象时，系统会在父对象的安全描述符中检查可继承的ACE。系统通常将任何可继承的ACE合并到新对象的安全描述符的ACL中。但是可以通过在安全描述符的控制位中设置`SE_DACL_PROTECTED`或`SE_SACL_PROTECTED`位来防止`DACL`或`SACL`继承`ACE`。

#### _新对象的DACL_

系统使用以下方式为新对象构建DACL：

1. 对象当前的DACL是来自对象创建者指定的安全描述符的DACL。除非在安全描述符的控制位中设置了`SE_DACL_PROTECTED`位，否则系统会将所有可继承的ACE合并到指定的DACL中。
2. 如果创建者未指定安全描述符，则系统将从可继承的ACE构建对象的DACL。
3. 如果未指定安全描述符，并且没有可继承的ACE，则对象的DACL是来自创建者的主令牌或模拟令牌的默认DACL。
4. 如果没有指定的、继承的或默认的DACL，则系统将创建不具有DACL的对象，从而允许所有人完全访问该对象。

系统使用不同的算法为新的Active Directory对象构建DACL。有关域的安全构建后续会有新的章节单独详细介绍。

#### _新对象的SACL_

系统使用以下方式为新对象构建SACL：

1. 对象的SACL是对象创建者指定的安全描述符中的SACL。除非在安全描述符的控制位中设置了`SE_SACL_PROTECTED`位，否则系统会将所有可继承的ACE合并到指定的SACL中。即使`SE_SACL_PROTECTED`位置已经设置，来自父对象的`SYSTEM_RESOURCE_ATTRIBUTE_ACE`和`SYSTEM_SCOPED_POLICY_ID_ACE`也将合并到新对象。
2. 如果创建者未指定安全描述符，则系统将从可继承的ACE构建对象的SACL。
3. 如果没有指定的或继承的SACL，则该对象没有SACL。

要为新对象指定SACL，对象的创建者必须启用`SE_SECURITY_NAME`特权。如果为新对象指定的SACL仅包含`SYSTEM_RESOURCE_ATTRIBUTE_ACE`，则不需要`SE_SECURITY_NAME`特权。如果对象的SACL是从继承的ACE构建的，则创建者不需要此特权。

同样，域DACL一样，系统使用不同的算法为新的Active Directory对象构建SACL。有关域的安全构建后续会有新的章节单独详细介绍。

#### _新对象的所有者_

对象的所有者隐式具有对该对象的`WRITE_DAC`访问权限。这意味着所有者可以修改对象的自由访问控制列表（DACL），从而可以控制对对象的访问。

新对象的所有者是创建过程的主令牌或模拟令牌中的默认所有者安全标识符（SID）。要获取或设置访问令牌中的默认所有者，可使用`TOKEN_OWNER`结构调用`GetTokenInformation`或`SetTokenInformation`函数。系统不允许将令牌的默认所有者设置为无效的SID，例如另一个用户帐户的SID。（如果可以就会产生越权的安全问题）

启用了`SE_TAKE_OWNERSHIP`特权的进程可以将自身设置为对象的所有者。启用了`SE_RESTORE_NAME`特权或对对象具有`WRITE_OWNER`访问权限的进程可以将任何有效的用户或组SID设置为对象的所有者。

#### _新对象的主要组_

新对象的主要组是对象创建者指定的安全描述符中的主要组。如果对象的创建者未指定主要组，则该对象的主要组是创建者的主要或模拟令牌中的默认主要组。

下面的示例为新的注册表项创建安全描述符。可以使用类似的代码为其他对象类型创建安全描述符。

1. 该示例使用两个ACE的信息填充EXPLICIT\_ACCESS结构的数组。一个ACE允许所有人读取权限；另一个ACE允许管理员完全访问。 
2. 该EXPLICIT\_ACCESS结构传递给SetEntriesInAcl函数来创建的安全描述符DACL。 
3. 在为安全描述符分配了内存之后，将调用InitializeSecurityDescriptor和SetSecurityDescriptorDacl函数来初始化安全描述符并附加DACL。 
4. 然后将安全描述符存储在SECURITY\_ATTRIBUTES结构中，并传递给RegCreateKeyEx函数，该函数将安全描述符附加到新创建的key上。

```text
#pragma comment(lib, "advapi32.lib")

#include <windows.h>
#include <stdio.h>
#include <aclapi.h>
#include <tchar.h>

void main()
{

    DWORD dwRes, dwDisposition;
    PSID pEveryoneSID = NULL, pAdminSID = NULL;
    PACL pACL = NULL;
    PSECURITY_DESCRIPTOR pSD = NULL;
    EXPLICIT_ACCESS ea[2];
    SID_IDENTIFIER_AUTHORITY SIDAuthWorld =
            SECURITY_WORLD_SID_AUTHORITY;
    SID_IDENTIFIER_AUTHORITY SIDAuthNT = SECURITY_NT_AUTHORITY;
    SECURITY_ATTRIBUTES sa;
    LONG lRes;
    HKEY hkSub = NULL;

    // Create a well-known SID for the Everyone group.
    if(!AllocateAndInitializeSid(&SIDAuthWorld, 1,
                     SECURITY_WORLD_RID,
                     0, 0, 0, 0, 0, 0, 0,
                     &pEveryoneSID))
    {
        _tprintf(_T("AllocateAndInitializeSid Error %u\n"), GetLastError());
        goto Cleanup;
    }

    // Initialize an EXPLICIT_ACCESS structure for an ACE.
    // The ACE will allow Everyone read access to the key.
    ZeroMemory(&ea, 2 * sizeof(EXPLICIT_ACCESS));
    ea[0].grfAccessPermissions = KEY_READ;
    ea[0].grfAccessMode = SET_ACCESS;
    ea[0].grfInheritance= NO_INHERITANCE;
    ea[0].Trustee.TrusteeForm = TRUSTEE_IS_SID;
    ea[0].Trustee.TrusteeType = TRUSTEE_IS_WELL_KNOWN_GROUP;
    ea[0].Trustee.ptstrName  = (LPTSTR) pEveryoneSID;

    // Create a SID for the BUILTIN\Administrators group.
    if(! AllocateAndInitializeSid(&SIDAuthNT, 2,
                     SECURITY_BUILTIN_DOMAIN_RID,
                     DOMAIN_ALIAS_RID_ADMINS,
                     0, 0, 0, 0, 0, 0,
                     &pAdminSID)) 
    {
        _tprintf(_T("AllocateAndInitializeSid Error %u\n"), GetLastError());
        goto Cleanup; 
    }

    // Initialize an EXPLICIT_ACCESS structure for an ACE.
    // The ACE will allow the Administrators group full access to
    // the key.
    ea[1].grfAccessPermissions = KEY_ALL_ACCESS;
    ea[1].grfAccessMode = SET_ACCESS;
    ea[1].grfInheritance= NO_INHERITANCE;
    ea[1].Trustee.TrusteeForm = TRUSTEE_IS_SID;
    ea[1].Trustee.TrusteeType = TRUSTEE_IS_GROUP;
    ea[1].Trustee.ptstrName  = (LPTSTR) pAdminSID;

    // Create a new ACL that contains the new ACEs.
    dwRes = SetEntriesInAcl(2, ea, NULL, &pACL);
    if (ERROR_SUCCESS != dwRes) 
    {
        _tprintf(_T("SetEntriesInAcl Error %u\n"), GetLastError());
        goto Cleanup;
    }

    // Initialize a security descriptor.  
    pSD = (PSECURITY_DESCRIPTOR) LocalAlloc(LPTR, 
                             SECURITY_DESCRIPTOR_MIN_LENGTH); 
    if (NULL == pSD) 
    { 
        _tprintf(_T("LocalAlloc Error %u\n"), GetLastError());
        goto Cleanup; 
    } 
 
    if (!InitializeSecurityDescriptor(pSD,
            SECURITY_DESCRIPTOR_REVISION)) 
    {  
        _tprintf(_T("InitializeSecurityDescriptor Error %u\n"),
                                GetLastError());
        goto Cleanup; 
    } 
 
    // Add the ACL to the security descriptor. 
    if (!SetSecurityDescriptorDacl(pSD, 
            TRUE,     // bDaclPresent flag   
            pACL, 
            FALSE))   // not a default DACL 
    {  
        _tprintf(_T("SetSecurityDescriptorDacl Error %u\n"),
                GetLastError());
        goto Cleanup; 
    } 

    // Initialize a security attributes structure.
    sa.nLength = sizeof (SECURITY_ATTRIBUTES);
    sa.lpSecurityDescriptor = pSD;
    sa.bInheritHandle = FALSE;

    // Use the security attributes to set the security descriptor 
    // when you create a key.
    lRes = RegCreateKeyEx(HKEY_CURRENT_USER, _T("mykey"), 0, _T(""), 0, 
            KEY_READ | KEY_WRITE, &sa, &hkSub, &dwDisposition); 
    _tprintf(_T("RegCreateKeyEx result %u\n"), lRes );

Cleanup:

    if (pEveryoneSID) 
        FreeSid(pEveryoneSID);
    if (pAdminSID) 
        FreeSid(pAdminSID);
    if (pACL) 
        LocalFree(pACL);
    if (pSD) 
        LocalFree(pSD);
    if (hkSub) 
        RegCloseKey(hkSub);

    return;

}
```

### **安全描述符字符串**

有效的功能安全描述符包含二进制格式的安全信息。 `Windows API`提供了用于将二进制安全描述符与文本字符串相互转换的功能。字符串格式的安全描述符不起作用，但是对于存储或传输安全描述符信息很有用。若要将安全描述符转换为字符串格式，需要调用 [`ConvertSecurityDescriptorToStringSecurityDescriptor`](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertsecuritydescriptortostringsecuritydescriptora)函数。要将字符串格式的安全描述符转换回有效的功能安全描述符，需要调用[`ConvertStringSecurityDescriptorToSecurityDescriptor`](https://docs.microsoft.com/en-us/windows/desktop/api/Sddl/nf-sddl-convertstringsecuritydescriptortosecuritydescriptora)函数。

