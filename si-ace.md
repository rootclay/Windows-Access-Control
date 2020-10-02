# （四）ACE

## **访问控制项**

访问控制条目（ACE）是访问控制列表（ACL）中的元素。一个ACL可以包含零个或多个ACE。每个ACE控制或监视指定受托者（Trustees）对对象的访问。

ACE有六种类型，所有安全对象都支持其中三种。其他三种类型是目录服务对象支持的特定于对象的ACE.

所有类型的ACE都包含以下访问控制信息：

* 安全标识符（SID），用于标识ACE适用于的受托者
* 一个访问掩码（Access Mask），该掩码指定了具体的访问权限（Access Rights），也就是可以对该对象执行的操作（此掩码非常重要，后续很多漏洞会用到）
* 一个位标记，指示ACE类型的标志。
* 一组位标志，指示了安全描述符所属对象的子对象是否继承这个ACE。

下表列出了所有安全对象支持的三种ACE类型。

| 类型 | 描述 |
| :---: | :---: |
| Access-denied ACE | 在自由访问控制列表（DACL）中使用，以拒绝对受托者的访问 |
| Access-allowed ACE | 在DACL中使用，以允许对受托者的访问权限。 |
| System-audit ACE | 当受托者尝试行使指定的访问权限时，用于系统访问控制列表（SACL）中以生成审核记录。 |

### **受托者**

受托者是访问控制条目（ACE）应用到的**用户帐户、组帐户或登录会话等**。访问控制列表（ACL）中的每个ACE都有一个安全标识符（SID），用于标识受托者。

1. 用户帐户包括人类用户或Windows Services等程序用于登录本地计算机的帐户。
2. 组帐户不能用于登录计算机，但是在ACE中它们非常有用，可以允许或拒绝组访问权限来达到对一个或多个用户帐户的权限控制。
3. 标识当前登录会话的登录SID仅在用户注销之前才允许允许或拒绝访问权限。

访问控制功能使用TRUSTEE结构来标识受托者。TRUSTEE结构可以使用名称字符串或SID来标识受托者。如果使用名称，则从TRUSTEE结构创建ACE的功能将执行分配SID缓冲区并查找与帐户名称相对应的SID的任务。有两个帮助程序函数`BuildTrusteeWithSid`和`BuildTrusteeWithName`，它们使用指定的SID或名称初始化TRUSTEE结构。 `BuildTrusteeWithObjectsAndSid`和`BuildTrusteeWithObjectsAndName`允许使用特定于对象的ACE信息初始化TRUSTEE结构。其他三个帮助函数`GetTrusteeForm`、`GetTrusteeName`和`GetTrusteeType`检索TRUSTEE结构的各个成员的值。

TRUSTEE结构的`ptstrName`成员可以是指向`OBJECTS_AND_NAME`或`OBJECTS_AND_SID`结构的指针。这些结构除了指定受托人名称或SID之外，还指定有关特定于对象的ACE的信息。这使诸如`SetEntriesInAcl`和`GetExplicitEntriesFromAcl`之类的功能可以将特定于对象的ACE信息存储在`EXPLICIT_ACCESS`结构的Trustee成员中。

### **特定对象的ACE**

目录服务即域服务（DS）对象支持特定于对象的ACE。特定于对象的ACE包含一对GUID，它们扩展了ACE保护对象的方式。

<table>
  <thead>
    <tr>
      <th style="text-align:center">GUID</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center"><b>ObjectType</b>
      </td>
      <td style="text-align:left">
        <p>&#x6807;&#x8BC6;&#x4EE5;&#x4E0B;&#x4E4B;&#x4E00;&#xFF1A;
          <br />
        </p>
        <ol>
          <li>&#x4E00;&#x79CD;&#x5B50;&#x5BF9;&#x8C61;&#x3002; ACE&#x63A7;&#x5236;&#x521B;&#x5EFA;&#x6307;&#x5B9A;&#x7C7B;&#x578B;&#x7684;&#x5B50;&#x5BF9;&#x8C61;&#x7684;&#x6743;&#x9650;&#x3002;</li>
          <li>&#x5C5E;&#x6027;&#x96C6;&#x6216;&#x5C5E;&#x6027;&#x3002; ACE&#x63A7;&#x5236;&#x8BFB;&#x53D6;&#x6216;&#x5199;&#x5165;&#x5C5E;&#x6027;&#x6216;&#x5C5E;&#x6027;&#x96C6;&#x7684;&#x6743;&#x9650;&#x3002;</li>
          <li>&#x6743;&#x9650;&#x6269;&#x5C55;&#x7684;&#x6743;&#x5229;&#x3002; ACE&#x63A7;&#x5236;&#x6267;&#x884C;&#x4E0E;&#x6269;&#x5C55;&#x6743;&#x9650;&#x76F8;&#x5173;&#x7684;&#x64CD;&#x4F5C;&#x7684;&#x6743;&#x9650;&#x3002;</li>
          <li>&#x7ECF;&#x8FC7;&#x9A8C;&#x8BC1;&#x7684;&#x5199;&#x5165;&#x3002; ACE&#x63A7;&#x5236;&#x6267;&#x884C;&#x67D0;&#x4E9B;&#x5199;&#x64CD;&#x4F5C;&#x7684;&#x6743;&#x9650;&#x3002;&#x8FD9;&#x4E9B;&#x5728;ACL&#x7F16;&#x8F91;&#x5668;&#x4E2D;&#x5B9A;&#x4E49;&#x548C;&#x516C;&#x5F00;&#x7684;&#x7ECF;&#x8FC7;&#x9A8C;&#x8BC1;&#x7684;&#x5199;&#x5165;&#x6743;&#x9650;&#x4E3A;&#x7ECF;&#x8FC7;&#x9A8C;&#x8BC1;&#x7684;&#x5C5E;&#x6027;&#x5199;&#x5165;&#x63D0;&#x4F9B;&#x4E86;&#x6743;&#x9650;&#xFF0C;&#x800C;&#x4E0D;&#x662F;&#x5BF9;&#x88AB;&#x6388;&#x4E88;&#x201C;&#x5199;&#x5165;&#x5C5E;&#x6027;&#x201D;&#x6743;&#x9650;&#x7684;&#x5C5E;&#x6027;&#x7684;&#x4EFB;&#x4F55;&#x503C;&#x8FDB;&#x884C;&#x672A;&#x7ECF;&#x68C0;&#x67E5;&#x7684;&#x4F4E;&#x7EA7;&#x5199;&#x5165;&#x3002;&#xFF08;&#x8FD9;&#x6BB5;&#x8BDD;&#x6458;&#x81EA;&#x5FAE;&#x8F6F;&#x5B98;&#x65B9;&#xFF0C;&#x7B80;&#x5355;&#x6765;&#x8BF4;&#x5C31;&#x662F;&#x4E3A;&#x5199;&#x5165;&#x7684;&#x6743;&#x9650;&#x8FDB;&#x884C;&#x4E86;&#x683C;&#x5F0F;&#x7684;&#x9A8C;&#x8BC1;&#x3002;&#xFF09;</li>
        </ol>
      </td>
    </tr>
    <tr>
      <td style="text-align:center"><b>InheritedObjectType</b>
      </td>
      <td style="text-align:left">&#x6307;&#x793A;&#x53EF;&#x4EE5;&#x7EE7;&#x627F;ACE&#x7684;&#x5B50;&#x5BF9;&#x8C61;&#x7684;&#x7C7B;&#x578B;&#x3002;&#x7EE7;&#x627F;&#x4E5F;&#x7531;ACE_HEADER&#x4E2D;&#x7684;&#x7EE7;&#x627F;&#x6807;&#x5FD7;&#x4EE5;&#x53CA;&#x653E;&#x7F6E;&#x5728;&#x5B50;&#x5BF9;&#x8C61;&#x4E0A;&#x7684;&#x4EFB;&#x4F55;&#x9632;&#x6B62;&#x7EE7;&#x627F;&#x7684;&#x4FDD;&#x62A4;&#x6765;&#x63A7;&#x5236;&#x3002;</td>
    </tr>
  </tbody>
</table>

### **DACL中的ACE顺序**

当进程尝试访问安全对象时，系统逐步浏览该对象的任意访问控制列表（DACL）中的访问控制项（ACE），直到找到允许或拒绝所请求访问的ACE。 DACL允许用户的访问权限可能会根据DACL中ACE的顺序而有所不同。因此，Windows XP操作系统在安全对象的DACL中为ACE定义了首选顺序。首选顺序提供了一个简单的框架，可确保拒绝访问的ACE实际上拒绝访问。上一节已经对该顺序有过介绍。

对于Windows Server 2003和Windows XP，引入特定于对象的ACE和自动继承会使得ACE的正确顺序变得复杂。

以下步骤描述了首选顺序：

1. 将所有显式ACE（即手动设置的ACE）放在一个组中，然后放在任何继承的ACE之前。
2. 在显式ACE组中，拒绝访问的ACE放置在允许访问的ACE之前。
3. 继承的ACE按照继承的顺序放置。从子对象的父对象继承的ACE首先出现，然后是从祖父母那里继承的ACE，依此类推。
4. 对于继承的ACE的每个级别，将拒绝访问的ACE放置在允许访问的ACE之前。

另外，并不是ACL中需要所有ACE类型。

### **ACE继承**

对象的ACL可以包含从其父容器继承的ACE。例如，注册表子项可以从注册表层次结构中位于其上方的项继承ACE。同样，NTFS文件系统中的文件可以从包含该文件的目录继承ACE。

ACE的`ACE_HEADER`结构（该结构见本节最后一小节）包含一组继承标志，这些继承标志控制ACE继承以及ACE对它所连接的对象的影响。系统根据ACE继承规则解释继承标志和其他继承信息。

这些规则通过以下功能得到增强：

* 支持自动传播可继承的ACE。
* 一个标志，用于区分继承的ACE和直接应用于对象的ACE。
* 特定于对象的ACE，允许指定可以继承ACE的子对象的类型。
* 通过将除`SYSTEM_RESOURCE_ATTRIBUTE_ACE`和`SYSTEM_SCOPED_POLICY_ID_ACE`之外的安全描述符的控制位中的`SE_DACL_PROTECTED`或`SE_SACL_PROTECTED`位设置来防止DACL或SACL继承ACE的能力。

### **自动传播可继承的ACE**

`SetNamedSecurityInfo`和`SetSecurityInfo`函数支持可继承访问控制项（ACE）的自动传播。**例如，如果使用这些功能将可继承的ACE添加到NTFS中的目录，则系统会将ACE适当地应用于任何现有子目录或文件的访问控制列表（ACL）。**

直接应用的ACE优先于继承的ACE。系统通过将直接应用的ACE放在任意访问控制列表（DACL）中继承的ACE之前来实现此优先级。当调用`SetNamedSecurityInfo`和`SetSecurityInfo`函数设置对象的安全信息时，系统会将当前继承模型强加于目标对象下方层次结构中所有对象的ACL上。对于已转换为当前继承模型的对象，在对象的安全描述符的控制字段中设置`SE_DACL_AUTO_INHERITED`和`SE_SACL_AUTO_INHERITED`位。

当构建一个反映当前继承模型的新安全描述符时，注意不要更改安全描述符的语义。因此，允许和拒绝ACE绝不会彼此相对移动。如果需要进行此类移动（例如，将所有非继承的ACE放在ACL的前面），则将该ACL标记为受保护，以防止语义更改。

将继承的ACE传播到子对象时，系统使用以下规则：

* 如果没有DACL的子对象继承了ACE，则结果是带有DACL的子对象仅包含继承的ACE。
* 如果带有空DACL的子对象继承了ACE，则结果是带有DACL的子对象仅包含继承的ACE。
* 如果从父对象中删除可继承的ACE，则自动继承将删除子对象继承的ACE的所有副本。
* 如果自动继承导致从子对象的DACL中删除所有ACE，则子对象具有空的DACL，而不是没有DACL。

这些规则可能会产生意外结果，即将没有DACL的对象转换为具有空DACL的对象。没有DACL的对象允许完全访问，但是具有空DACL的对象不允许访问。作为这些规则如何创建空DACL的示例，假设将可继承的ACE添加到对象树的根对象。自动继承将可继承的ACE传播到树中的所有对象。现在，没有DACL的子对象具有带有继承的ACE的DACL。如果从根对象中删除可继承的ACE，则系统会自动将更改传播到子对象。现在，没有DACL（允许完全访问）的子对象具有空的DACL（不允许访问）。

为确保没有DACL的子对象不受继承ACE的影响，在对象的安全描述符中设置`SE_DACL_PROTECTED`标志。

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

如果指定了`InheritedObjectType GUID`，则如果设置了`OBJECT_INHERIT_ACE`，则可以由与GUID匹配的对象继承ACE，如果设置了`CONTAINER_INHERIT_ACE`，则可以由与GUID匹配的容器继承。注意当前只有DS对象支持特定于对象的ACE，并且DS将所有对象类型都视为容器。

### **ACE控制对对象属性的访问**

目录服务（DS）对象的任意访问控制列表（DACL）可以包含访问控制项（ACE）的层次结构，如下所示：

1. 保护对象本身的ACE
2. 特定对象的ACE，可保护对象上的指定属性集
3. 特定对象的ACE，用于保护对象上的指定属性

在此层次结构中，较高级别授予或拒绝的权利也适用于较低级别。例如，如果属性集上的特定于对象的ACE允许受托者获得`ADS_RIGHT_DS_READ_PROP`权限，则受托者具有对该属性集所有属性的隐式读取访问权限。同样，在对象本身上的ACE允许`ADS_RIGHT_DS_READ_PROP`访问，使受托者可以对对象的所有属性进行读取访问。

下图显示了假设的DS对象的树及其属性集和属性:

![directory service object hierarchy](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/accctrl2.png)

假设希望允许以下访问此DS对象的属性：

* 允许A组对所有对象属性的读/写权限
* 允许其他所有人对除属性D之外的所有属性具有读/写权限

为此，如下表所示，在对象的DACL中设置ACE:

| 受托者 | 对象 GUID | ACE 类型 | 访问权限 |
| :--- | :--- | :--- | :---: |
| Group A | None | Access-allowed ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |
| Everyone | Property Set 1 | Access-allowed object ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |
| Everyone | Property C | Access-allowed object ACE | ADS\_RIGHT\_DS\_READ\_PROP \| ADS\_RIGHT\_DS\_WRITE\_PROP |

组A的ACE没有对象GUID，这意味着它允许访问所有对象的属性。属性集1的ACE允许所有人访问属性A和B。另一个属性集ACE允许所有人访问属性C。注意尽管此DACL没有任何访问被拒绝的ACE，但它隐式拒绝了属性D，与除A组之外的所有人访问。

当用户尝试访问对象的属性时，系统将按顺序检查ACE，直到显式授予，拒绝所请求的访问，或者不再有ACE，在这种情况下，访问将被隐式拒绝。

该方式的适用性评估：

* 适用于对象本身的ACE
* 适用于包含正在访问的属性的属性集的特定于对象的ACE
* 适用于所访问属性的特定于对象的ACE

系统将忽略适用于其他属性集或属性的特定于对象的ACE。

### ACE Header 结构体

```text
typedef struct _ACE_HEADER {
  BYTE AceType;
  BYTE AceFlags;
  WORD AceSize;
} ACE_HEADER;
```



| 值 | 含义 |
| :--- | :--- |
| **ACCESS\_ALLOWED\_ACE\_TYPE** | 使用[ACCESS\_ALLOWED\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_allowed_ace)结构的允许访问的ACE 。 |
| **ACCESS\_ALLOWED\_CALLBACK\_ACE\_TYPE** | 使用[ACCESS\_ALLOWED\_CALLBACK\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_allowed_callback_ace)结构的允许访问的回调ACE 。 |
| **ACCESS\_ALLOWED\_CALLBACK\_OBJECT\_ACE\_TYPE** | 使用[ACCESS\_ALLOWED\_CALLBACK\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_allowed_callback_object_ace)结构的特定于对象的允许访问的回调ACE 。 |
| **ACCESS\_ALLOWED\_COMPOUND\_ACE\_TYPE** | 保留。 |
| **ACCESS\_ALLOWED\_OBJECT\_ACE\_TYPE** | 使用[ACCESS\_ALLOWED\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_allowed_object_ace)结构的特定于对象的允许访问的ACE 。 |
| **ACCESS\_DENIED\_ACE\_TYPE** | 使用[ACCESS\_DENIED\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_denied_ace)结构的拒绝访问的ACE 。 |
| **ACCESS\_DENIED\_CALLBACK\_ACE\_TYPE** | 使用[ACCESS\_DENIED\_CALLBACK\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_denied_callback_ace)结构的拒绝访问的回调ACE 。 |
| **ACCESS\_DENIED\_CALLBACK\_OBJECT\_ACE\_TYPE** | 使用[ACCESS\_DENIED\_CALLBACK\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_denied_callback_ace)结构的特定于对象的访问被拒绝的回调ACE 。 |
| **ACCESS\_DENIED\_OBJECT\_ACE\_TYPE** | 使用[ACCESS\_DENIED\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-access_denied_object_ace)结构的特定于对象的拒绝访问的ACE 。 |
| **ACCESS\_MAX\_MS\_ACE\_TYPE** | 与SYSTEM\_ALARM\_OBJECT\_ACE\_TYPE相同。 |
| **ACCESS\_MAX\_MS\_V2\_ACE\_TYPE** | 与SYSTEM\_ALARM\_ACE\_TYPE相同。 |
| **ACCESS\_MAX\_MS\_V3\_ACE\_TYPE** | 保留。 |
| **ACCESS\_MAX\_MS\_V4\_ACE\_TYPE** | 与SYSTEM\_ALARM\_OBJECT\_ACE\_TYPE相同。 |
| **ACCESS\_MAX\_MS\_OBJECT\_ACE\_TYPE** | 与SYSTEM\_ALARM\_OBJECT\_ACE\_TYPE相同。 |
| **ACCESS\_MIN\_MS\_ACE\_TYPE** | 与ACCESS\_ALLOWED\_ACE\_TYPE相同。 |
| **ACCESS\_MIN\_MS\_OBJECT\_ACE\_TYPE** | 与ACCESS\_ALLOWED\_OBJECT\_ACE\_TYPE相同。 |
| **SYSTEM\_ALARM\_ACE\_TYPE** | 保留以备将来使用。使用[SYSTEM\_ALARM\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_alarm_ace)结构的系统警报ACE 。 |
| **SYSTEM\_ALARM\_CALLBACK\_ACE\_TYPE** | 保留以备将来使用。使用[SYSTEM\_ALARM\_CALLBACK\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_alarm_callback_ace)结构的系统警报回调ACE 。 |
| **SYSTEM\_ALARM\_CALLBACK\_OBJECT\_ACE\_TYPE** | 保留以备将来使用。使用[SYSTEM\_ALARM\_CALLBACK\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_alarm_callback_object_ace)结构的特定于对象的系统警报回调ACE 。 |
| **SYSTEM\_ALARM\_OBJECT\_ACE\_TYPE** | 保留以备将来使用。使用[SYSTEM\_ALARM\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_alarm_object_ace)结构的特定于对象的系统警报ACE 。 |
| **SYSTEM\_AUDIT\_ACE\_TYPE** | 使用[SYSTEM\_AUDIT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_audit_ace)结构的系统审核ACE 。 |
| **SYSTEM\_AUDIT\_CALLBACK\_ACE\_TYPE** | 使用[SYSTEM\_AUDIT\_CALLBACK\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_audit_callback_ace)结构的系统审核回调ACE 。 |
| **SYSTEM\_AUDIT\_CALLBACK\_OBJECT\_ACE\_TYPE** | 使用[SYSTEM\_AUDIT\_CALLBACK\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_audit_callback_object_ace)结构的特定于对象的系统审核回调ACE 。 |
| **SYSTEM\_AUDIT\_OBJECT\_ACE\_TYPE** | 使用[SYSTEM\_AUDIT\_OBJECT\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_alarm_object_ace)结构的特定于对象的系统审核ACE 。 |
| **SYSTEM\_MANDATORY\_LABEL\_ACE\_TYPE**0x11 | 使用[SYSTEM\_MANDATORY\_LABEL\_ACE](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-system_mandatory_label_ace)结构的强制标签ACE 。 |

AceFlags：



<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x503C;</th>
      <th style="text-align:left">&#x542B;&#x4E49;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>CONTAINER_INHERIT_ACE</b>
      </td>
      <td style="text-align:left">&#x4F5C;&#x4E3A;&#x5BB9;&#x5668;&#x7684;&#x5B50;&#x5BF9;&#x8C61;&#xFF08;&#x4F8B;&#x5982;&#x76EE;&#x5F55;&#xFF09;&#x7EE7;&#x627F;ACE&#x4F5C;&#x4E3A;&#x6709;&#x6548;ACE&#x3002;&#x9664;&#x975E;&#x8FD8;&#x8BBE;&#x7F6E;&#x4E86;NO_PROPAGATE_INHERIT_ACE&#x4F4D;&#x6807;&#x5FD7;&#xFF0C;&#x5426;&#x5219;&#x7EE7;&#x627F;&#x7684;ACE&#x662F;&#x53EF;&#x7EE7;&#x627F;&#x7684;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>FAILED_ACCESS_ACE_FLAG</b>
      </td>
      <td style="text-align:left">&#x4E0E;&#x7CFB;&#x7EDF;<a href="https://docs.microsoft.com/en-us/windows/desktop/SecGloss/s-gly">&#x8BBF;&#x95EE;&#x63A7;&#x5236;&#x5217;&#x8868;</a>&#xFF08;SACL&#xFF09;&#x4E2D;&#x7684;
        <a
        href="https://docs.microsoft.com/en-us/windows/desktop/SecGloss/s-gly">&#x7CFB;&#x7EDF;</a>&#x5BA1;&#x6838;ACE&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x4EE5;&#x4E3A;&#x5931;&#x8D25;&#x7684;&#x8BBF;&#x95EE;&#x5C1D;&#x8BD5;&#x751F;&#x6210;&#x5BA1;&#x6838;&#x6D88;&#x606F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>INHERIT_ONLY_ACE</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x793A;&#x4EC5;&#x7EE7;&#x627F;&#x7684;ACE&#xFF0C;&#x5B83;&#x4E0D;&#x63A7;&#x5236;&#x5BF9;&#x5176;&#x9644;&#x52A0;&#x5BF9;&#x8C61;&#x7684;&#x8BBF;&#x95EE;&#x3002;&#x5982;&#x679C;&#x672A;&#x8BBE;&#x7F6E;&#x6B64;&#x6807;&#x5FD7;&#xFF0C;&#x5219;ACE&#x662F;&#x6709;&#x6548;&#x7684;ACE&#xFF0C;&#x5B83;&#x63A7;&#x5236;&#x5BF9;&#x5176;&#x9644;&#x52A0;&#x5BF9;&#x8C61;&#x7684;&#x8BBF;&#x95EE;&#x3002;</p>
        <p>&#x6709;&#x6548;ACE&#x548C;&#x4EC5;&#x7EE7;&#x627F;ACE&#x5747;&#x53EF;&#x88AB;&#x7EE7;&#x627F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x5176;&#x4ED6;&#x7EE7;&#x627F;&#x6807;&#x5FD7;&#x7684;&#x72B6;&#x6001;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>INHERITED_ACE</b>
      </td>
      <td style="text-align:left">&#x8868;&#x793A;ACE&#x5DF2;&#x88AB;&#x7EE7;&#x627F;&#x3002;&#x5F53;&#x7CFB;&#x7EDF;&#x5C06;&#x7EE7;&#x627F;&#x7684;ACE&#x4F20;&#x64AD;&#x5230;&#x5B50;&#x5BF9;&#x8C61;&#x65F6;&#xFF0C;&#x7CFB;&#x7EDF;&#x4F1A;&#x5C06;&#x5176;&#x8BBE;&#x7F6E;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>NO_PROPAGATE_INHERIT_ACE</b>
      </td>
      <td style="text-align:left">&#x5982;&#x679C;ACE&#x88AB;&#x5B50;&#x5BF9;&#x8C61;&#x7EE7;&#x627F;&#xFF0C;&#x5219;&#x7CFB;&#x7EDF;&#x6E05;&#x9664;&#x7EE7;&#x627F;&#x7684;ACE&#x4E2D;&#x7684;OBJECT_INHERIT_ACE&#x548C;CONTAINER_INHERIT_ACE&#x6807;&#x5FD7;&#x3002;&#x8FD9;&#x6837;&#x53EF;&#x4EE5;&#x9632;&#x6B62;ACE&#x88AB;&#x540E;&#x7EED;&#x7684;&#x5BF9;&#x8C61;&#x7EE7;&#x627F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>OBJECT_INHERIT_ACE</b>
      </td>
      <td style="text-align:left">
        <p>&#x975E;&#x5BB9;&#x5668;&#x5B50;&#x5BF9;&#x8C61;&#x7EE7;&#x627F;ACE&#x4F5C;&#x4E3A;&#x6709;&#x6548;ACE&#x3002;</p>
        <p>&#x5BF9;&#x4E8E;&#x4F5C;&#x4E3A;&#x5BB9;&#x5668;&#x7684;&#x5B50;&#x5BF9;&#x8C61;&#xFF0C;&#x9664;&#x975E;&#x8FD8;&#x8BBE;&#x7F6E;&#x4E86;NO_PROPAGATE_INHERIT_ACE&#x4F4D;&#x6807;&#x5FD7;&#xFF0C;&#x5426;&#x5219;ACE&#x5C06;&#x4F5C;&#x4E3A;&#x4EC5;&#x7EE7;&#x627F;ACE&#x8FDB;&#x884C;&#x7EE7;&#x627F;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SUCCESSFUL_ACCESS_ACE_FLAG</b>
      </td>
      <td style="text-align:left">&#x4E0E;SACL&#x4E2D;&#x7684;&#x7CFB;&#x7EDF;&#x5BA1;&#x6838;ACE&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x4EE5;&#x751F;&#x6210;&#x6210;&#x529F;&#x8BBF;&#x95EE;&#x5C1D;&#x8BD5;&#x7684;&#x5BA1;&#x6838;&#x6D88;&#x606F;&#x3002;</td>
    </tr>
  </tbody>
</table>

## \*\*\*\*

