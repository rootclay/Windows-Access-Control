# （三）ACL

## **访问控制列表**

![ACL&#x5206;&#x7C7B;](.gitbook/assets/image%20%281%29.png)

DACL：自主访问控制列表\(DACL\)是安全描述符中最重要的，它里面包含零个或多个访问控制项（ACE，Access Control Entry），每个访问控制项的内容描述了允许或拒绝特定账户对这个对象执行特定操作。

SACL：系统访问控制列表（SACL） 主要是用于系统审计的，它的内容指定了当特定账户对这个对象执行特定操作时，记录到系统日志中。

访问控制列表（ACL）是访问控制条目（ACE）的列表。 ACL中的每个ACE都标识一个对象（通常称这个对象为受托者，受托者可以是一个用户、用户组或者是一个登陆会话），并指定允许、拒绝或审核该受托者的访问权限。可保护对象的安全描述符可以包含两种类型的ACL：DACL和SACL。

DACL标识是否允许或拒绝访问安全对象。当进程尝试访问安全对象时，系统将检查该对象的DACL中的ACE，以确定是否授予对该对象的访问权限。

1. 如果对象没有DACL，则系统将授予所有人完全访问权限。
2. 如果对象的DACL没有ACE，则系统将拒绝所有尝试访问该对象的尝试，因为DACL不允许任何访问权限。
3. 系统依次检查ACE，直到找到一个或多个允许所有请求的访问权限的ACE，或者直到拒绝任何请求的访问权限为止。

下图显示了对象的DACL如何允许访问一个线程而拒绝访问另一个线程。

![](.gitbook/assets/image%20%2810%29.png)

对于线程A，系统将读取第一条ACE并立即拒绝访问，因为拒绝访问的ACE适用于线程访问令牌中的用户。在这种情况下，系统将不检查之后的ACE。对于线程B，第一条ACE不适用，因此系统进入允许写入访问的ACE 2和允许读取和执行访问的ACE 3。

SACL使管理员可以记录任何人对安全对象的访问。每个ACE指定受托者尝试访问的类型，这些访问使系统在安全事件日志中生成记录。当访问尝试失败或成功时，SACL中的ACE可以生成审核记录。

不要尝试直接使用ACL的内容。为确保ACL在语义上正确，需使用适当的函数来创建和操作ACL。

ACL还提供对`Microsoft Active Directory`目录服务对象的访问控制。 `Active Directory`服务接口（ADSI）包括用于创建和修改这些ACL内容的例程。相关内容会另开文章详细介绍。

### **从ACL获取信息**

微软提供了几种从访问控制列表（ACL）中检索访问控制信息的功能。这些功能包括确定ACL授予或审核指定受托者的访问权限的功能。其他功能使您能够提取有关ACL中访问控制项（ACE）的信息。

`GetExplicitEntriesFromAcl`函数检索描述在ACL中的ACE的`EXPLICIT_ACCESS`结构的数组。将ACE信息从一个ACL复制到另一个ACL时，此功能很有用。例如，对`GetExplicitEntriesFromAcl`的调用以在一个ACL中获取有关ACE的信息之后，可以通过在对`SetEntriesInAcl`函数的调用中传递返回的`EXPLICIT_ACCESS`结构，以在新的ACL中创建等效的ACE。

`GetEffectiveRightsFromAcl`函数使您能够确定DACL授予指定受托者的有效访问权限。受托人的有效访问权是DACL授予受托人或受托人为成员的任何组的访问权。 `GetEffectiveRightsFromAcl`检查指定DACL中的所有允许访问和拒绝访问的ACE。

使用以下步骤确定受托者对对象的访问权限:

1. 调用`GetSecurityInfo`或`GetNamedSecurityInfo`函数以获取指向对象的DACL的指针。
2. 调用`GetEffectiveRightsFromAcl`函数以检索DACL授予指定受托者的访问权限。

`GetAuditedPermissionsFromAcl`函数使您可以检查SACL，以确定指定受托人或受托人所属成员的任何组的审核访问权限。审核的权限指示导致系统在安全事件日志中生成审核记录的访问尝试的类型。该函数返回两个访问掩码：一个包含对失败的访问尝试进行监视的访问权限，另一个包含对成功的访问进行监视的访问权限。 `GetAuditedPermissionsFromAcl`检查SACL中的所有系统审核的ACE。

### **创建或修改ACL**

Windows支持一组功能，这些功能可以创建访问控制列表（ACL）或修改现有ACL中的访问控制项（ACE）。

`SetEntriesInAcl`函数创建一个新的ACL。 `SetEntriesInAcl`可以为ACL指定一组全新的ACE，也可以将一个或多个新ACE与现有ACL的ACE合并。 `SetEntriesInAcl`函数使用`EXPLICIT_ACCESS`结构的数组来指定新ACE的信息。每个EXPLICIT\_ACCESS结构都包含描述单个ACE的信息。此信息包括访问权限、ACE的类型、控制ACE继承的标志以及标识受托者的`TRUSTEE`结构。

向现有ACL添加新ACE

1. 使用`GetSecurityInfo`或`GetNamedSecurityInfo`函数可从对象的安全描述符获取现有的DACL或SACL。
2. 对于每个新的ACE，可调用`BuildExplicitAccessWithName`函数以使用描述ACE的信息填充`EXPLICIT_ACCESS`结构。
3. 调用`SetEntriesInAcl`，为新ACE指定现有的ACL和`EXPLICIT_ACCESS`结构的数组。`SetEntriesInAcl`函数分配并初始化ACL及其ACE。
4. 调用`SetSecurityInfo`或`SetNamedSecurityInfo`函数，将新的ACL附加到对象的安全描述符。

如果调用方指定了现有的ACL，则`SetEntriesInAcl`会将新的ACE信息与ACL中的现有ACE合并。例如，考虑以下情况：现有ACL授予对指定受托者的访问权限，而`EXPLICIT_ACCESS`结构拒绝对同一受托者的访问权限。在这种情况下，`SetEntriesInAcl`为受托者添加一个新的拒绝访问的ACE，并为受托者删除或修改现有的允许访问的ACE。

相关示例代码我准备了C++与C\#两个版本：

C++版本代码，示例使用GetNamedSecurityInfo函数获取现有的DACL。然后，它使用有关ACE的信息填充EXPLICIT\_ACCESS结构，并使用SetEntriesInAcl函数将新ACE与DACL中的任何现有ACE合并。最后，该示例调用SetNamedSecurityInfo函数将新的DACL附加到对象的安全描述符。：

```text
#include <windows.h>
#include <stdio.h>

DWORD AddAceToObjectsSecurityDescriptor (
    LPTSTR pszObjName,          // name of object
    SE_OBJECT_TYPE ObjectType,  // type of object
    LPTSTR pszTrustee,          // trustee for new ACE
    TRUSTEE_FORM TrusteeForm,   // format of trustee structure
    DWORD dwAccessRights,       // access mask for new ACE
    ACCESS_MODE AccessMode,     // type of ACE
    DWORD dwInheritance         // inheritance flags for new ACE
) 
{
    DWORD dwRes = 0;
    PACL pOldDACL = NULL, pNewDACL = NULL;
    PSECURITY_DESCRIPTOR pSD = NULL;
    EXPLICIT_ACCESS ea;

    if (NULL == pszObjName) 
        return ERROR_INVALID_PARAMETER;

    // Get a pointer to the existing DACL.

    dwRes = GetNamedSecurityInfo(pszObjName, ObjectType, 
          DACL_SECURITY_INFORMATION,
          NULL, NULL, &pOldDACL, NULL, &pSD);
    if (ERROR_SUCCESS != dwRes) {
        printf( "GetNamedSecurityInfo Error %u\n", dwRes );
        goto Cleanup; 
    }  

    // Initialize an EXPLICIT_ACCESS structure for the new ACE. 

    ZeroMemory(&ea, sizeof(EXPLICIT_ACCESS));
    ea.grfAccessPermissions = dwAccessRights;
    ea.grfAccessMode = AccessMode;
    ea.grfInheritance= dwInheritance;
    ea.Trustee.TrusteeForm = TrusteeForm;
    ea.Trustee.ptstrName = pszTrustee;

    // Create a new ACL that merges the new ACE
    // into the existing DACL.

    dwRes = SetEntriesInAcl(1, &ea, pOldDACL, &pNewDACL);
    if (ERROR_SUCCESS != dwRes)  {
        printf( "SetEntriesInAcl Error %u\n", dwRes );
        goto Cleanup; 
    }  

    // Attach the new ACL as the object's DACL.

    dwRes = SetNamedSecurityInfo(pszObjName, ObjectType, 
          DACL_SECURITY_INFORMATION,
          NULL, NULL, pNewDACL, NULL);
    if (ERROR_SUCCESS != dwRes)  {
        printf( "SetNamedSecurityInfo Error %u\n", dwRes );
        goto Cleanup; 
    }  

    Cleanup:

        if(pSD != NULL) 
            LocalFree((HLOCAL) pSD); 
        if(pNewDACL != NULL) 
            LocalFree((HLOCAL) pNewDACL); 

        return dwRes;
}
```

C\#版本，修改文件的ACL：

```text
using System;
using System.IO;
using System.Security.AccessControl;

namespace FileSystemExample
{
    class FileExample
    {
        public static void Main()
        {
            try
            {
                string FileName = "c:/test.xml";

                Console.WriteLine("Adding access control entry for " + FileName);

                // Add the access control entry to the file.
                // Before compiling this snippet, change MyDomain to your 
                // domain name and MyAccessAccount to the name 
                // you use to access your domain.
                AddFileSecurity(FileName, @"MyDomain\MyAccessAccount", FileSystemRights.ReadData, AccessControlType.Allow);

                Console.WriteLine("Removing access control entry from " + FileName);

                // Remove the access control entry from the file.
                // Before compiling this snippet, change MyDomain to your 
                // domain name and MyAccessAccount to the name 
                // you use to access your domain.
                RemoveFileSecurity(FileName, @"MyDomain\MyAccessAccount", FileSystemRights.ReadData, AccessControlType.Allow);

                Console.WriteLine("Done.");
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }

        }

        // Adds an ACL entry on the specified file for the specified account.
        public static void AddFileSecurity(string FileName, string Account, FileSystemRights Rights, AccessControlType ControlType)
        {
            // Create a new FileInfo object.
            FileInfo fInfo = new FileInfo(FileName);

            // Get a FileSecurity object that represents the 
            // current security settings.
            FileSecurity fSecurity = fInfo.GetAccessControl();

            // Add the FileSystemAccessRule to the security settings. 
            fSecurity.AddAccessRule(new FileSystemAccessRule(Account,
                                                            Rights,
                                                            ControlType));

            // Set the new access settings.
            fInfo.SetAccessControl(fSecurity);

        }

        // Removes an ACL entry on the specified file for the specified account.
        public static void RemoveFileSecurity(string FileName, string Account, FileSystemRights Rights, AccessControlType ControlType)
        {
            // Create a new FileInfo object.
            FileInfo fInfo = new FileInfo(FileName);

            // Get a FileSecurity object that represents the 
            // current security settings.
            FileSecurity fSecurity = fInfo.GetAccessControl();

            // Add the FileSystemAccessRule to the security settings. 
            fSecurity.RemoveAccessRule(new FileSystemAccessRule(Account,
                                                            Rights,
                                                            ControlType));

            // Set the new access settings.
            fInfo.SetAccessControl(fSecurity);

        }
    }
}
//This code produces output similar to the following; 
//results may vary based on the computer/file structure/etc.:
//
//Adding access control entry for c:\test.xml
//Removing access control entry from c:\test.xml
//Done.
//
```

