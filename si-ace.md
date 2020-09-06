# （四）ACE

## **访问控制项**

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

### **受托者**

受托者是访问控制条目（ACE）应用到的用户帐户、组帐户或登录会话。访问控制列表（ACL）中的每个ACE都有一个安全标识符（SID），用于标识受托者。

用户帐户包括人类用户或Windows Services等程序用于登录本地计算机的帐户。

组帐户不能用于登录计算机，但是在ACE中它们非常有用，可以允许或拒绝对一个或多个用户帐户的一组访问权限。

标识当前登录会话的登录SID仅在用户注销之前才允许访问控制功能使用TRUSTEE结构来标识受托者。 TRUSTEE结构使您可以使用名称字符串或SID来标识受托者。如果使用名称，则从TRUSTEE结构创建ACE的功能将执行分配SID缓冲区并查找与帐户名称相对应的SID的任务。有两个帮助程序函数`BuildTrusteeWithSid`和`BuildTrusteeWithName`，它们使用指定的SID或名称初始化TRUSTEE结构。 `BuildTrusteeWithObjectsAndSid`和`BuildTrusteeWithObjectsAndName`允许您使用特定于对象的ACE信息初始化TRUSTEE结构。其他三个帮助函数`GetTrusteeForm`、`GetTrusteeName`和`GetTrusteeType`检索TRUSTEE结构的各个成员的值。或拒绝访问权限。

TRUSTEE结构的`ptstrName`成员可以是指向`OBJECTS_AND_NAME`或`OBJECTS_AND_SID`结构的指针。这些结构除了指定受托人名称或SID之外，还指定有关特定于对象的ACE的信息。这使诸如`SetEntriesInAcl`和`GetExplicitEntriesFromAcl`之类的功能可以将特定于对象的ACE信息存储在`EXPLICIT_ACCESS`结构的Trustee成员中。

### **特定对象的ACE**

目录服务（DS）对象支持特定于对象的ACE。特定于对象的ACE包含一对GUID，它们扩展了ACE保护对象的方式。

| GUID | 描述 |
| :---: | :---: |
| **ObjectType** | 标识以下之一：  一种 子对象。 ACE控制创建指定类型的子对象的权限。有关更多信息，请参见在C ++中控制子对象的创建。 属性集或属性。 ACE控制读取或写入属性或属性集的权限。有关更多信息，请参见用于控制对对象属性的访问的ACE。 延长的权利。 ACE控制执行与扩展权限相关的操作的权限。 经过验证的写入。 ACE控制执行某些写操作的权限。这些在ACL编辑器中定义和公开的经过验证的写入权限为经过验证的属性写入提供了权限，而不是对被授予“写入属性”权限的属性的任何值进行未经检查的低级写入。 |
| **InheritedObjectType** | 指示可以继承ACE的子对象的类型。继承也由ACE\_HEADER中的继承标志以及放置在子对象上的任何防止继承的保护来控制。有关更多信息，请参见ACE继承。 |

### **DACL中的ACE顺序**

当进程尝试访问安全对象时，系统逐步浏览该对象的任意访问控制列表（DACL）中的访问控制项（ACE），直到找到允许或拒绝所请求访问的ACE。 DACL允许用户的访问权限可能会根据DACL中ACE的顺序而有所不同。因此，Windows XP操作系统在安全对象的DACL中为ACE定义了首选顺序。首选顺序提供了一个简单的框架，可确保拒绝访问的ACE实际上拒绝访问。有关系统用于检查访问的算法的更多信息，请参见DACL如何控制对对象的访问。

对于Windows Server 2003和Windows XP，引入特定于对象的ACE和自动继承会使得ACE的正确顺序变得复杂。

以下步骤描述了首选顺序：

1. 将所有显式ACE放在一个组中，然后放在任何继承的ACE之前。
2. 在显式ACE组中，拒绝访问的ACE放置在允许访问的ACE之前。
3. 继承的ACE按照继承的顺序放置。从子对象的父对象继承的ACE首先出现，然后是从祖父母那里继承的ACE，依此类推。
4. 对于继承的ACE的每个级别，将拒绝访问的ACE放置在允许访问的ACE之前。

当然，并不是ACL中需要所有ACE类型。

### **ACE继承**

对象的ACL可以包含从其父容器继承的ACE。例如，注册表子项可以从注册表层次结构中位于其上方的项继承ACE。同样，NTFS文件系统中的文件可以从包含该文件的目录继承ACE。

ACE的`ACE_HEADER`结构包含一组继承标志，这些继承标志控制ACE继承以及ACE对它所连接的对象的影响。系统根据ACE继承规则解释继承标志和其他继承信息。

这些规则通过以下功能得到增强：

* 支持自动传播可继承的ACE。
* 一个标志，用于区分继承的ACE和直接应用于对象的ACE。
* 特定于对象的ACE，允许您指定可以继承ACE的子对象的类型。
* 通过将除`SYSTEM_RESOURCE_ATTRIBUTE_ACE`和`SYSTEM_SCOPED_POLICY_ID_ACE`之外的安全描述符的控制位中的`SE_DACL_PROTECTED`或`SE_SACL_PROTECTED`位设置来防止DACL或SACL继承ACE的能力。

### **自动传播可继承的ACE**

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

### **ACE继承规则**

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

### **ACE控制对对象属性的访问**

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

## \*\*\*\*

