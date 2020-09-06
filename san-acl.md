# （三）ACL

## **访问控制列表**

访问控制列表（ACL）是访问控制条目（ACE）的列表。 ACL中的每个ACE都标识一个受托者，并指定允许、拒绝或审核该受托者的访问权限。可保护对象的安全描述符可以包含两种类型的ACL：DACL和SACL。

任意访问控制列表（DACL）标识允许或拒绝访问安全对象的受托者。当进程尝试访问安全对象时，系统将检查该对象的DACL中的ACE，以确定是否授予对该对象的访问权限。如果对象没有DACL，则系统将授予所有人完全访问权限。如果对象的DACL没有ACE，则系统将拒绝所有尝试访问该对象的尝试，因为DACL不允许任何访问权限。系统依次检查ACE，直到找到一个或多个允许所有请求的访问权限的ACE，或者直到拒绝任何请求的访问权限为止。有关更多信息，请参见DACL如何控制对对象的访问。有关如何正确创建DACL的信息，请参阅创建DACL。

系统访问控制列表（SACL）使管理员可以记录访问安全对象的尝试。每个ACE指定指定受托者尝试访问的类型，这些尝试使系统在安全事件日志中生成记录。当访问尝试失败、成功或失败时，SACL中的ACE可以生成审核记录。有关SACL的更多信息，请参见审核生成和SACL访问权限。

不要尝试直接使用ACL的内容。为确保ACL在语义上正确，请使用适当的函数来创建和操作ACL。有关更多信息，请参见从ACL获取信息和创建或修改ACL。

ACL还提供对`Microsoft Active Directory`目录服务对象的访问控制。 `Active Directory`服务接口（ADSI）包括用于创建和修改这些ACL内容的例程。有关更多信息，请参见控制对Active Directory对象的访问。

### **从ACL获取信息**

提供了几种从访问控制列表（ACL）中检索访问控制信息的功能。这些功能包括确定ACL授予或审核指定受托者的访问权限的功能。其他功能使您能够提取有关ACL中访问控制项（ACE）的信息。

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
2. 对于每个新的ACE，请调用`BuildExplicitAccessWithName`函数以使用描述ACE的信息填充`EXPLICIT_ACCESS`结构。
3. 调用`SetEntriesInAcl`，为新ACE指定现有的ACL和`EXPLICIT_ACCESS`结构的数组。`SetEntriesInAcl`函数分配并初始化ACL及其ACE。
4. 调用`SetSecurityInfo`或`SetNamedSecurityInfo`函数，将新的ACL附加到对象的安全描述符。

如果调用方指定了现有的ACL，则`SetEntriesInAcl`会将新的ACE信息与ACL中的现有ACE合并。例如，考虑以下情况：现有ACL授予对指定受托者的访问权限，而`EXPLICIT_ACCESS`结构拒绝对同一受托者的访问权限。在这种情况下，`SetEntriesInAcl`为受托者添加一个新的拒绝访问的ACE，并为受托者删除或修改现有的允许访问的ACE。

有关将新ACE合并到现有ACL中的示例代码，请参见[Modifying the ACLs of an Object in C++](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/modifying-the-acls-of-an-object-in-c--).

## 

