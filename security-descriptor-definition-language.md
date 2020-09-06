# 安全描述定义语言

### 安全描述符定义语言

安全描述符定义语言（SDDL）定义了`ConvertSecurityDescriptorToStringSecurityDescriptor`和`ConvertStringSecurityDescriptorToSecurityDescriptor`函数用来将安全描述符描述为文本字符串的字符串格式。该语言还定义了字符串元素，用于描述安全描述符的组成部分中的信息。

#### 安全描述符字符串格式

安全描述符字符串格式是一种文本格式，用于在安全描述符中存储或传输信息。 `ConvertSecurityDescriptorToStringSecurityDescriptor`和`ConvertStringSecurityDescriptorToSecurityDescriptor`函数使用此格式。

格式是一个以空值结尾的字符串，带有令牌，用于指示安全描述符的四个主要组成部分：所有者（O :\)，主要组（G :\)，DACL（D :\)和SACL（S :\)。

```cpp
O:owner_sid
G:group_sid
D:dacl_flags(string_ace1)(string_ace2)... (string_acen)
S:sacl_flags(string_ace1)(string_ace2)... (string_acen)
```

owner\_sid : 标识对象所有者的SID字符串。

group\_sid : 一个SID字符串，用于标识对象的主要组。

dacl\_flags : 适用于DACL的安全描述符控制标志。有关这些控制标志的说明，请参见SetSecurityDescriptorControl函数。 dacl\_flags字符串可以是零个或多个以下字符串的串联。

| Control | Constant in Sddl.h | Meaning |
| :--- | :--- | :--- |
| "P" | SDDL\_PROTECTED | The SE\_DACL\_PROTECTED flag is set. |
| "AR" | SDDL\_AUTO\_INHERIT\_REQ | The SE\_DACL\_AUTO\_INHERIT\_REQ flag is set. |
| "AI" | SDDL\_AUTO\_INHERITED | The SE\_DACL\_AUTO\_INHERITED flag is set. |
| "NO\_ACCESS\_CONTROL" | SSDL\_NULL\_ACL | The ACL is null. |

sacl\_flags : 适用于SACL的安全描述符控制标志。 sacl\_flags字符串使用与dacl\_flags字符串相同的控制位字符串。

string\_ace : 在安全描述符的DACL或SACL中描述ACE的字符串。有关ACE字符串格式的说明，请参阅ACE字符串。每个ACE字符串都括在括号（（））中。

可以从安全描述符字符串中省略不需要的组件。例如，如果未在输入安全描述符中设置`SE_DACL_PRESENT`标志，则`ConvertSecurityDescriptorToStringSecurityDescriptor`在输出字符串中不包含D：组件。您还可以使用`SECURITY_INFORMATION`位标志来指示要包含在安全描述符字符串中的组件。

安全描述符字符串格式不支持NULL ACL。

为了表示一个空的ACL，安全描述符字符串包括D：或S：令牌，没有其他字符串信息。

安全描述符字符串以不同的方式存储“安全描述符控制”位。`SE_DACL_PRESENT`或`SE_SACL_PRESENT`位由字符串中D：或S：令牌的存在指示。适用于DACL或SACL的其他位存储在`dacl_flags`和`sacl_flags`中。 `SE_OWNER_DEFAULTED`、`SE_GROUP_DEFAULTED`、`SE_DACL_DEFAULTED`和`SE_SACL_DEFAULTED`位未存储在安全描述符字符串中。 `SE_SELF_RELATIVE`位未存储在字符串中，但是`ConvertStringSecurityDescriptorToSecurityDescriptor`始终在输出安全描述符中设置此位。

以下示例显示了安全描述符字符串和关联的安全描述符中的信息。

```cpp
"O:AOG:DAD:(A;;RPWPCCDCLCSWRCWDWOGA;;;S-1-0-0)"
```

安全描述符1：

```cpp
Revision:  0x00000001
    Control:   0x0004
        SE_DACL_PRESENT
    Owner: (S-1-5-32-548)
    PrimaryGroup: (S-1-5-21-397955417-626881126-188441444-512)
DACL
    Revision: 0x02
    Size:     0x001c
    AceCount: 0x0001
    Ace[00]
        AceType:       0x00 (ACCESS_ALLOWED_ACE_TYPE)
        AceSize:       0x0014
        InheritFlags:  0x00
        Access Mask:   0x100e003f
                            READ_CONTROL
                            WRITE_DAC
                            WRITE_OWNER
                            GENERIC_ALL
                            Others(0x0000003f)
        Ace Sid      : (S-1-0-0)
SACL
    Not present
```

字符串 2:

```cpp
"O:DAG:DAD:(A;;RPWPCCDCLCRCWOWDSDSW;;;SY)
(A;;RPWPCCDCLCRCWOWDSDSW;;;DA)
(OA;;CCDC;bf967aba-0de6-11d0-a285-00aa003049e2;;AO)
(OA;;CCDC;bf967a9c-0de6-11d0-a285-00aa003049e2;;AO)
(OA;;CCDC;6da8a4ff-0e52-11d0-a286-00aa003049e2;;AO)
(OA;;CCDC;bf967aa8-0de6-11d0-a285-00aa003049e2;;PO)
(A;;RPLCRC;;;AU)S:(AU;SAFA;WDWOSDWPCCDCSW;;;WD)"
```

安全描述符2：

```cpp
Revision:  0x00000001
    Control:   0x0014
        SE_DACL_PRESENT
        SE_SACL_PRESENT
    Owner: (S-1-5-21-397955417-626881126-188441444-512)
    PrimaryGroup: (S-1-5-21-397955417-626881126-188441444-512)
DACL
    Revision: 0x04
    Size:     0x0104
    AceCount: 0x0007
    Ace[00]
        AceType:       0x00 (ACCESS_ALLOWED_ACE_TYPE)
        AceSize:       0x0014
        InheritFlags:  0x00
        Access Mask:   0x000f003f
                            DELETE
                            READ_CONTROL
                            WRITE_DAC
                            WRITE_OWNER
                            Others(0x0000003f)
        Ace Sid:       (S-1-5-18)
    Ace[01]
        AceType:       0x00 (ACCESS_ALLOWED_ACE_TYPE)
        AceSize:       0x0024
        InheritFlags:  0x00
        Access Mask:   0x000f003f
                            DELETE
                            READ_CONTROL
                            WRITE_DAC
                            WRITE_OWNER
                            Others(0x0000003f)
        Ace Sid:       (S-1-5-21-397955417-626881126-188441444-512)
    Ace[02]
        AceType:       0x05 (ACCESS_ALLOWED_OBJECT_ACE_TYPE)
        AceSize:       0x002c
        InheritFlags:  0x00
        Access Mask:   0x00000003
                            Others(0x00000003)
        Flags:         0x00000001, ACE_OBJECT_TYPE_PRESENT
        ObjectType:    GUID_C_USER
        InhObjectType: GUID ptr is NULL
        Ace Sid:       (S-1-5-32-548)
    Ace[03]
        AceType:       0x05 (ACCESS_ALLOWED_OBJECT_ACE_TYPE)
        AceSize:       0x002c
        InheritFlags:  0x00
        Access Mask:   0x00000003
                            Others(0x00000003)
        Flags:         0x00000001, ACE_OBJECT_TYPE_PRESENT
        ObjectType:    GUID_C_GROUP
        InhObjectType: GUID ptr is NULL
        Ace Sid:       (S-1-5-32-548)
    Ace[04]
        AceType:       0x05 (ACCESS_ALLOWED_OBJECT_ACE_TYPE)
        AceSize:       0x002c
        InheritFlags:  0x00
        Access Mask:   0x00000003
                            Others(0x00000003)
        Flags:         0x00000001, ACE_OBJECT_TYPE_PRESENT
        ObjectType:    GUID_C_LOCALGROUP
        InhObjectType: GUID ptr is NULL
        Ace Sid:       (S-1-5-32-548)
    Ace[05]
        AceType:       0x05 (ACCESS_ALLOWED_OBJECT_ACE_TYPE)
        AceSize:       0x002c
        InheritFlags:  0x00
        Access Mask:   0x00000003
                            Others(0x00000003)
        Flags:         0x00000001, ACE_OBJECT_TYPE_PRESENT
        ObjectType:    GUID_C_PRINT_QUEUE
        InhObjectType: GUID ptr is NULL
        Ace Sid:       (S-1-5-32-550)
    Ace[06]
        AceType:       0x00 (ACCESS_ALLOWED_ACE_TYPE)
        AceSize:       0x0014
        InheritFlags:  0x00
        Access Mask:   0x00020014
                            READ_CONTROL
                            Others(0x00000014)
        Ace Sid:       (S-1-5-11)
    SACL
        Revision: 0x02
        Size:     0x001c
        AceCount: 0x0001
        Ace[00]
            AceType:       0x02 (SYSTEM_AUDIT_ACE_TYPE)
            AceSize:       0x0014
            InheritFlags:  0xc0
                SUCCESSFUL_ACCESS_ACE_FLAG
                FAILED_ACCESS_ACE_FLAG
            Access Mask:    0x000d002b
                                DELETE
                                WRITE_DAC
                                WRITE_OWNER
                                Others(0x0000002b)
            Ace Sid:       (S-1-1-0)
```

#### 条件ACE的安全描述符定义语言

条件访问控制项（ACE）允许在执行访问检查时评估访问条件。安全描述符定义语言（SDDL）提供了用于以字符串格式定义条件ACE的语法。

条件ACE的SDDL与任何ACE相同，条件语句的语法附加在ACE字符串的末尾。有关SDDL的信息，请参阅安全描述符定义语言。

资源属性中的“＃”符号与“ 0”同义。例如，`D：AI（XA; OICI; FA ;;; WD;（OctetStringType ==＃1＃2＃3 ##））`等效并解释为`D：AI（XA; OICI; FA ;;; WD ;（OctetStringType ==＃01020300））`。

**条件ACE字符串格式**

安全描述符字符串中的每个ACE都用括号括起来。 ACE的字段按以下顺序排列，并用分号（;）分隔。

_AceType\*\*_;_**AceFlags**_;_**Rights**_;_**ObjectGuid**_;_**InheritObjectGuid**_;_**AccountSid**_;\(_**ConditionalExpression**_\)\*\*

这些字段如ACE字符串中所述，但以下情况除外。

* AceType字段可以是以下字符串之一。

| ACE 类型的字符串 | Sddl.h中的字符串 | AceType 值 |
| :--- | :--- | :--- |
| "XA" | SDDL\_CALLBACK\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_CALLBACK\_ACE\_TYPE |
| "XD" | SDDL\_CALLBACK\_ACCESS\_DENIED | ACCESS\_DENIED\_CALLBACK\_ACE\_TYPE |

* ACE字符串包含一个或多个条件表达式，并在字符串末尾的括号中。

**条件表达式**

条件表达式可以包含以下任何元素。

| 表达式元素 | 描述 |
| :--- | :--- |
| _AttributeName_ | 测试指定的属性是否具有非零值。 |
| **exists** _AttributeName_ | 测试客户端上下文中是否存在指定的属性。 |
| _AttributeName_ _Operator_ _Value_ | 返回指定操作的结果。 |
| _ConditionalExpression\*\*_\|\|_\*\*ConditionalExpression_ | 测试指定的条件表达式中的任何一个是否为true。 |
| _ConditionalExpression_ **&&** _ConditionalExpression_ | 测试两个指定的条件表达式是否为真。 |
| **!\(\**\*ConditionalExpression\*_\_\)\*\* | 条件表达式的逆函数。 |
| **Member\_of{\**\*SidArray\*_\_}\*\* | 测试客户端上下文的`SID_AND_ATTRIBUTES`数组是否包含`SidArray`指定的逗号分隔列表中的所有安全标识符（SID）。 对于允许ACE，客户端上下文SID必须将`SE_GROUP_ENABLED`属性设置为被视为匹配项。 对于`Deny ACE`，客户端上下文SID必须将`SE_GROUP_ENABLED`或`SE_GROUP_USE_FOR_DENY_ONLY`属性设置为被视为匹配项。 `SidArray`数组可以包含SID字符串（例如“ S-1-5-6”）或SID别名（例如“ BA”） |

**属性**

属性表示客户端上下文中`AUTHZ_SECURITY_ATTRIBUTES_INFORMATION`数组中的元素。属性名称可以包含任何字母数字字符和任何字符“：”，“ /”，“。”和“ \_”。

属性值可以是以下任何类型。

| 类型 | 描述 |
| :--- | :--- |
| Integer | 以十进制或十六进制表示的64位整数。 |
| String | 用引号分隔的字符串值。 |
| SID | 必须位于Member\_of或Device\_Member\_of的RHS上。 |
| BLOB | ＃后跟十六进制数字。如果数字的长度为奇数，则将＃转换为0使其变为偶数。同样，在值的其他位置出现的＃也将转换为0。 |

**操作符**

定义了以下运算符，供在条件表达式中使用以测试属性值。所有这些都是二进制运算符，并以`AttributeName`运算符值的形式使用。

| Operator | Description |
| :--- | :--- |
| == | Conventional definition. |
| != | Conventional definition. |
| &lt; | Conventional definition. |
| &lt;= | Conventional definition. |
| &gt; | Conventional definition. |
| &gt;= | Conventional definition. |
| Contains | 如果指定属性的值是指定值的超集，则为TRUE；否则为TRUE。否则为FALSE. |
| Any\_of | 如果指定值是指定属性值的超集，则为TRUE；否则为TRUE。否则为FALSE。 |

此外，一元运算符Exists，Member\_of和negation（！）的定义如条件表达式表中所述。 “ Contains”运算符必须在空格之前和之后，而“ Any\_of”运算符必须在空格之前。

**未知值**

条件表达式的结果有时返回值Unknown。例如，当指定的属性不存在时，任何关系操作都将返回“未知”。

下表描述了两个条件表达式`ConditionalExpression1`和`ConditionalExpression2`之间的逻辑与运算的结果。

| _ConditionalExpression1_ | _ConditionalExpression2_ | _ConditionalExpression1_ **&&** _ConditionalExpression2_ |
| :--- | :--- | :--- |
| **TRUE** | **TRUE** | **TRUE** |
| **TRUE** | **FALSE** | **FALSE** |
| **TRUE** | **UNKNOWN** | **UNKNOWN** |
| **FALSE** | **TRUE** | **FALSE** |
| **FALSE** | **FALSE** | **FALSE** |
| **FALSE** | **UNKNOWN** | **FALSE** |
| **UNKNOWN** | **TRUE** | **UNKNOWN** |
| **UNKNOWN** | **FALSE** | **FALSE** |
| **UNKNOWN** | **UNKNOWN** | **UNKNOWN** |

下表描述了两个条件表达式`ConditionalExpression1`和`ConditionalExpression2`之间的逻辑或运算的结果。

| _ConditionalExpression1_ | _ConditionalExpression2_ | _ConditionalExpression1_ **\|\|** _ConditionalExpression2_ |
| :--- | :--- | :--- |
| **TRUE** | **TRUE** | **TRUE** |
| **TRUE** | **FALSE** | **TRUE** |
| **TRUE** | **UNKNOWN** | **TRUE** |
| **FALSE** | **TRUE** | **TRUE** |
| **FALSE** | **FALSE** | **FALSE** |
| **FALSE** | **UNKNOWN** | **UNKNOWN** |
| **UNKNOWN** | **TRUE** | **TRUE** |
| **UNKNOWN** | **FALSE** | **UNKNOWN** |
| **UNKNOWN** | **UNKNOWN** | **UNKNOWN** |

值为`UNKNOWN`的条件表达式的否定也是`UNKNOWN`。

**ACE条件评估**

下表描述了根据条件表达式的最终评估得出的条件ACE的访问检查结果。

| ACE type | TRUE | FALSE | UNKNOWN |
| :--- | :--- | :--- | :--- |
| Allow | Allow | Ignore ACE | Ignore ACE |
| Deny | Deny | Ignore ACE | Deny |

**案例：**

下面的示例说明如何通过使用SDDL定义的条件ACE来表示指定的访问策略。

条件：如果同时满足以下两个条件，则允许对所有人执行

* Title = PM
* Division = Finance or Division = Sales

SDDL:`D:(XA; ;FX;;;S-1-1-0; (@User.Title=="PM" && (@User.Division=="Finance" || @User.Division ==" Sales")))`

条件: 如果任何用户项目与文件项目相交，则允许执行。

SDDL：`D:(XA; ;FX;;;S-1-1-0; (@User.Project Any_of @Resource.Project))`

条件：如果用户已经使用智能卡登录，是备份操作员并且正在从启用了Bitlocker的计算机进行连接，则允许读取访问。

SDDL:

```text
D:(XA; ;FR;;;S-1-1-0; (Member_of {SID(Smartcard_SID), SID(BO)} && @Device.Bitlocker))
```

#### ACE字符串

安全描述符定义语言（SDDL）在安全描述符字符串的DACL和SACL组件中使用ACE字符串。 如“安全描述符字符串格式”示例所示，安全描述符字符串中的每个ACE都用括号括起来。 ACE的字段按以下顺序排列，并用分号（;）分隔

语法：`ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid;(resource_attribute)`

范围：

**ace\_type**

一个字符串，指示ACE\_HEADER结构的AceType成员的值。 ACE类型字符串可以是Sddl.h中定义的以下字符串之一。

| ACE 类型字符串 | Sddl.h中的常量 | Ace类型的值 |
| :--- | :--- | :--- |
| "A" | SDDL\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_ACE\_TYPE |
| "D" | SDDL\_ACCESS\_DENIED | ACCESS\_DENIED\_ACE\_TYPE |
| "OA" | SDDL\_OBJECT\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_OBJECT\_ACE\_TYPE |
| "OD" | SDDL\_OBJECT\_ACCESS\_DENIED | ACCESS\_DENIED\_OBJECT\_ACE\_TYPE |
| "AU" | SDDL\_AUDIT | SYSTEM\_AUDIT\_ACE\_TYPE |
| "AL" | SDDL\_ALARM | SYSTEM\_ALARM\_ACE\_TYPE |
| "OU" | SDDL\_OBJECT\_AUDIT | SYSTEM\_AUDIT\_OBJECT\_ACE\_TYPE |
| "OL" | SDDL\_OBJECT\_ALARM | SYSTEM\_ALARM\_OBJECT\_ACE\_TYPE |
| "ML" | SDDL\_MANDATORY\_LABEL | SYSTEM\_MANDATORY\_LABEL\_ACE\_TYPE |
| "XA" | SDDL\_CALLBACK\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_CALLBACK\_ACE\_TYPE**Windows Vista and Windows Server 2003:** Not available. |
| "XD" | SDDL\_CALLBACK\_ACCESS\_DENIED | ACCESS\_DENIED\_CALLBACK\_ACE\_TYPE**Windows Vista and Windows Server 2003:** Not available. |
| "RA" | SDDL\_RESOURCE\_ATTRIBUTE | SYSTEM\_RESOURCE\_ATTRIBUTE\_ACE\_TYPE**Windows Server 2008 R2, Windows 7, Windows Server 2008, Windows Vista and Windows Server 2003:** Not available. |
| "SP" | SDDL\_SCOPED\_POLICY\_ID | SYSTEM\_SCOPED\_POLICY\_ID\_ACE\_TYPE**Windows Server 2008 R2, Windows 7, Windows Server 2008, Windows Vista and Windows Server 2003:** Not available. |
| "XU" | SDDL\_CALLBACK\_AUDIT | SYSTEM\_AUDIT\_CALLBACK\_ACE\_TYPE**Windows Server 2008 R2, Windows 7, Windows Server 2008, Windows Vista and Windows Server 2003:** Not available. |
| "ZA" | SDDL\_CALLBACK\_OBJECT\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_CALLBACK\_ACE\_TYPE**Windows Server 2008 R2, Windows 7, Windows Server 2008, Windows Vista and Windows Server 2003:** Not available. |

**ace\_flags**

一个字符串，指示`ACE_HEADER`结构的`AceFlags`成员的值。 ACE标志字符串可以是Sddl.h中定义的以下字符串的串联。

| ACE 标志字符串 | Sddl.h中的常量 | Ace标志值 |
| :--- | :--- | :--- |
| "CI" | SDDL\_CONTAINER\_INHERIT | CONTAINER\_INHERIT\_ACE |
| "OI" | SDDL\_OBJECT\_INHERIT | OBJECT\_INHERIT\_ACE |
| "NP" | SDDL\_NO\_PROPAGATE | NO\_PROPAGATE\_INHERIT\_ACE |
| "IO" | SDDL\_INHERIT\_ONLY | INHERIT\_ONLY\_ACE |
| "ID" | SDDL\_INHERITED | INHERITED\_ACE |
| "SA" | SDDL\_AUDIT\_SUCCESS | SUCCESSFUL\_ACCESS\_ACE\_FLAG |
| "FA" | SDDL\_AUDIT\_FAILURE | FAILED\_ACCESS\_ACE\_FLAG |

一个字符串，指示`ACE_HEADER`结构的`AceFlags`成员的值。 ACE标志字符串可以是`Sddl.h`中定义的以下字符串的串联。

**rights**

一个字符串，指示由ACE控制的访问权限。该字符串可以是访问权限的十六进制字符串表示形式，例如“ 0x7800003F”，也可以是以下字符串的串联形式。

_通用访问权限_

| 访问权限字符串 | Sddl.h中的常量 | 访问权限值 |
| :--- | :--- | :--- |
| "GA" | SDDL\_GENERIC\_ALL | GENERIC\_ALL |
| "GR" | SDDL\_GENERIC\_READ | GENERIC\_READ |
| "GW" | SDDL\_GENERIC\_WRITE | GENERIC\_WRITE |
| "GX" | SDDL\_GENERIC\_EXECUTE | GENERIC\_EXECUTE |

_标准访问权限_

| 访问权限字符串 | Sddl.h中的常量 | 访问权限值 |
| :--- | :--- | :--- |
| "RC" | SDDL\_READ\_CONTROL | READ\_CONTROL |
| "SD" | SDDL\_STANDARD\_DELETE | DELETE |
| "WD" | SDDL\_WRITE\_DAC | WRITE\_DAC |
| "WO" | SDDL\_WRITE\_OWNER | WRITE\_OWNER |

_目录服务对象访问权限_

| 访问权限字符串 | Sddl.h中的常量 | 访问权限值 |
| :--- | :--- | :--- |
| "RP" | SDDL\_READ\_PROPERTY | ADS\_RIGHT\_DS\_READ\_PROP |
| "WP" | SDDL\_WRITE\_PROPERTY | ADS\_RIGHT\_DS\_WRITE\_PROP |
| "CC" | SDDL\_CREATE\_CHILD | ADS\_RIGHT\_DS\_CREATE\_CHILD |
| "DC" | SDDL\_DELETE\_CHILD | ADS\_RIGHT\_DS\_DELETE\_CHILD |
| "LC" | SDDL\_LIST\_CHILDREN | ADS\_RIGHT\_ACTRL\_DS\_LIST |
| "SW" | SDDL\_SELF\_WRITE | ADS\_RIGHT\_DS\_SELF |
| "LO" | SDDL\_LIST\_OBJECT | ADS\_RIGHT\_DS\_LIST\_OBJECT |
| "DT" | SDDL\_DELETE\_TREE | ADS\_RIGHT\_DS\_DELETE\_TREE |
| "CR" | SDDL\_CONTROL\_ACCESS | ADS\_RIGHT\_DS\_CONTROL\_ACCESS |

_档案存取权_

| 访问权限字符串 | Sddl.h中的值 | 访问权限值 |
| :--- | :--- | :--- |
| "FA" | SDDL\_FILE\_ALL | FILE\_ALL\_ACCESS |
| "FR" | SDDL\_FILE\_READ | FILE\_GENERIC\_READ |
| "FW" | SDDL\_FILE\_WRITE | FILE\_GENERIC\_WRITE |
| "FX" | SDDL\_FILE\_EXECUTE | FILE\_GENERIC\_EXECUTE |

_注册表项访问权限_

| 访问权限字符串 | Sddl.h中的常量 | 访问权限值 |
| :--- | :--- | :--- |
| "KA" | SDDL\_KEY\_ALL | KEY\_ALL\_ACCESS |
| "KR" | SDDL\_KEY\_READ | KEY\_READ |
| "KW" | SDDL\_KEY\_WRITE | KEY\_WRITE |
| "KX" | SDDL\_KEY\_EXECUTE | KEY\_EXECUTE |

_强制性标签权利_

| 访问权限字符串 | Sddl.h中的值 | 访问权限值 |
| :--- | :--- | :--- |
| "NR" | SDDL\_NO\_READ\_UP | SYSTEM\_MANDATORY\_LABEL\_NO\_READ\_UP |
| "NW" | SDDL\_NO\_WRITE\_UP | SYSTEM\_MANDATORY\_LABEL\_NO\_WRITE\_UP |
| "NX" | SDDL\_NO\_EXECUTE\_UP | SYSTEM\_MANDATORY\_LABEL\_NO\_EXECUTE\_UP |

**object\_guid**

GUID的字符串表示形式，它表示特定于对象的ACE结构（例如`ACCESS_ALLOWED_OBJECT_ACE`）的`ObjectType`成员的值。 GUID字符串使用`UuidToString`函数返回的格式。

下表列出了一些常用的对象GUID。

| Rights and GUID | Permission |
| :--- | :--- |
| CR;ab721a53-1e2f-11d0-9819-00aa0040529b | Change password |
| CR;00299570-246d-11d0-a768-00aa006e0529 | Reset password |

**inherit\_object\_guid**

GUID的字符串表示形式，它指示特定于对象的ACE结构的`InheritedObjectType`成员的值。 GUID字符串使用`UuidToString`格式。

**account\_sid**

标识ACE受托者的SID字符串。

**resource\_attribute**

可选: resource\_attribute仅用于资源ACE，并且是可选的。指示数据类型的字符串。资源属性ace数据类型可以是Sddl.h中定义的以下数据类型之一。

资源属性中的“＃”符号与“ 0”同义。例如，`D:AI(XA;OICI;FA;;;WD;(OctetStringType==#1#2#3##))` 等同于`D:AI(XA;OICI;FA;;;WD;(OctetStringType==#01020300))`.

**Windows Server 2008 R2, Windows 7, Windows Server 2008, Windows Vista and Windows Server 2003:** 资源属性不可用

| ace类型字符串资源属性 | Sddl.h中的常量 | 数据类型 |
| :--- | :--- | :--- |
| "TI" | SDDL\_INT | Signed integer |
| "TU" | SDDL\_UINT | Unsigned integer |
| "TS" | SDDL\_WSTRING | Wide string |
| "TD" | SDDL\_SID | SID |
| "TX" | SDDL\_BLOB | Octet string |
| "TB" | SDDL\_BOOLEAN | Boolean |

以下示例显示了允许访问的ACE的ACE字符串。它不是特定于对象的ACE，因此在`object_guid`和`Inherited_object_guid`字段中没有任何信息。 `ace_flags`字段也为空，表示未设置任何ACE标志。

```cpp
(A;;RPWPCCDCLCSWRCWDWOGA;;;S-1-1-0)
```

上面显示的ACE字符串描述了以下ACE信息。

```cpp
AceType:       0x00 (ACCESS_ALLOWED_ACE_TYPE)
AceFlags:      0x00
Access Mask:   0x100e003f
                    READ_CONTROL
                    WRITE_DAC
                    WRITE_OWNER
                    GENERIC_ALL
                    Other access rights(0x0000003f)
Ace Sid      : (S-1-1-0)
```

下面的示例显示了一个文件，该文件使用Windows的资源声明和“结构化查询语言（SQL）”将“保密性”设置为“高业务影响”。

```cpp
(RA;CI;;;;S-1-1-0; ("Project",TS,0,"Windows","SQL")) 
(RA;CI;;;;S-1-1-0; ("Secrecy",TU,0,3))
```

上面显示的ACE字符串描述了以下ACE信息。

```cpp
AceType:       0x12 (SYSTEM_RESOURCE_ATTRIBUTE_ACE_TYPE)
AceFlags:      0x1  (SDDL_CONTAINER_INHERIT)
Access Mask:   0x0
Ace Sid      : (S-1-1-0)
Resource Attributes: Project has the strings Windows and SQL, Secrecy has the unsigned int value of 3
```

#### SID字符串

在安全描述符定义语言（SDDL）中，安全描述符字符串将SID字符串用于安全描述符的以下组件：

* Owner
* Primary group
* The [trustee](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/trustees) in an ACE

安全描述符字符串中的SID字符串可以使用SID的标准字符串表示形式（S-R-I-S-S）或Sddl.h中定义的字符串常量之一。有关标准SID字符串表示法的更多信息，请参见SID组件。

在Sddl.h中定义了以下用于著名SID的SID字符串常量。有关相应的相对ID（RID）的信息，请参阅众所周知的SID。

| SDDL SID 字符串 | Sddl.h 中的长度 | 帐户别名和相应的RID |
| :--- | :--- | :--- |
| "AN" | SDDL\_ANONYMOUS | 匿名登录。相应的RID为SECURITY\_ANONYMOUS\_LOGON\_RID |
| "AO" | SDDL\_ACCOUNT\_OPERATORS | 帐户操作员。相应的RID为DOMAIN\_ALIAS\_RID\_ACCOUNT\_OPS。 |
| "AU" | SDDL\_AUTHENTICATED\_USERS | 经过身份验证的用户。相应的RID是SECURITY\_AUTHENTICATED\_USER\_RID。 |
| "BA" | SDDL\_BUILTIN\_ADMINISTRATORS | 内置管理员。相应的RID是DOMAIN\_ALIAS\_RID\_ADMINS。 |
| "BG" | SDDL\_BUILTIN\_GUESTS | 内置来宾。相应的RID是DOMAIN\_ALIAS\_RID\_GUESTS。 |
| "BO" | SDDL\_BACKUP\_OPERATORS | 备份运算符。相应的RID是DOMAIN\_ALIAS\_RID\_BACKUP\_OPS。 |
| "BU" | SDDL\_BUILTIN\_USERS | 内置用户。相应的RID是DOMAIN\_ALIAS\_RID\_USERS。 |
| "CA" | SDDL\_CERT\_SERV\_ADMINISTRATORS | 证书发布者。相应的RID是DOMAIN\_GROUP\_RID\_CERT\_ADMINS。 |
| "CD" | SDDL\_CERTSVC\_DCOM\_ACCESS | 可以使用分布式组件对象模型（DCOM）连接到证书颁发机构的用户。相应的RID是DOMAIN\_ALIAS\_RID\_CERTSVC\_DCOM\_ACCESS\_GROUP。 |
| "CG" | SDDL\_CREATOR\_GROUP | 创作者组。相应的RID是SECURITY\_CREATOR\_GROUP\_RID。 |
| "CO" | SDDL\_CREATOR\_OWNER | 创作者所有者。相应的RID是SECURITY\_CREATOR\_OWNER\_RID。 |
| "DA" | SDDL\_DOMAIN\_ADMINISTRATORS | 域管理员。相应的RID为DOMAIN\_GROUP\_RID\_ADMINS。 |
| "DC" | SDDL\_DOMAIN\_COMPUTERS | 域计算机。相应的RID是DOMAIN\_GROUP\_RID\_COMPUTERS。 |
| "DD" | SDDL\_DOMAIN\_DOMAIN\_CONTROLLERS | 域控制器。相应的RID是DOMAIN\_GROUP\_RID\_CONTROLLERS。 |
| "DG" | SDDL\_DOMAIN\_GUESTS | 域来宾。相应的RID是DOMAIN\_GROUP\_RID\_GUESTS。 |
| "DU" | SDDL\_DOMAIN\_USERS | 域用户。相应的RID是DOMAIN\_GROUP\_RID\_USERS。 |
| "EA" | SDDL\_ENTERPRISE\_ADMINS | 企业管理员。相应的RID为DOMAIN\_GROUP\_RID\_ENTERPRISE\_ADMINS。 |
| "ED" | SDDL\_ENTERPRISE\_DOMAIN\_CONTROLLERS | 企业域控制器。相应的RID是SECURITY\_SERVER\_LOGON\_RID。 |
| "HI" | SDDL\_ML\_HIGH | 高诚信度。相应的RID是SECURITY\_MANDATORY\_HIGH\_RID。 |
| "IU" | SDDL\_INTERACTIVE | 交互式登录的用户。这是在以交互方式登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_INTERACTIVE。相应的RID是SECURITY\_INTERACTIVE\_RID。 |
| "LA" | SDDL\_LOCAL\_ADMIN | 本地管理员。相应的RID是DOMAIN\_USER\_RID\_ADMIN。 |
| "LG" | SDDL\_LOCAL\_GUEST | 当地客人。相应的RID为DOMAIN\_USER\_RID\_GUEST。 |
| "LS" | SDDL\_LOCAL\_SERVICE | 本地服务帐户。相应的RID是SECURITY\_LOCAL\_SERVICE\_RID。 |
| "LW" | SDDL\_ML\_LOW | 完整性等级低。相应的RID是SECURITY\_MANDATORY\_LOW\_RID。 |
| "ME" | SDDL\_MLMEDIUM | 中等完整性级别。相应的RID是SECURITY\_MANDATORY\_MEDIUM\_RID。 |
| "MU" | SDDL\_PERFMON\_USERS | 性能监视器用户。 |
| "NO" | SDDL\_NETWORK\_CONFIGURATION\_OPS | 网络配置运营商。相应的RID是DOMAIN\_ALIAS\_RID\_NETWORK\_CONFIGURATION\_OPS。 |
| "NS" | SDDL\_NETWORK\_SERVICE | 网络服务帐户。相应的RID是SECURITY\_NETWORK\_SERVICE\_RID。 |
| "NU" | SDDL\_NETWORK | 网络登录用户。这是在通过网络登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_NETWORK。相应的RID是SECURITY\_NETWORK\_RID。 |
| "PA" | SDDL\_GROUP\_POLICY\_ADMINS | 组策略管理员。相应的RID为DOMAIN\_GROUP\_RID\_POLICY\_ADMINS。 |
| "PO" | SDDL\_PRINTER\_OPERATORS | 打印机操作员。相应的RID是DOMAIN\_ALIAS\_RID\_PRINT\_OPS。 |
| "PS" | SDDL\_PERSONAL\_SELF | 主要自我。相应的RID是SECURITY\_PRINCIPAL\_SELF\_RID。 |
| "PU" | SDDL\_POWER\_USERS | 超级用户。相应的RID是DOMAIN\_ALIAS\_RID\_POWER\_USERS。 |
| "RC" | SDDL\_RESTRICTED\_CODE | 受限制的代码。这是使用CreateRestrictedToken函数创建的受限令牌。相应的RID为SECURITY\_RESTRICTED\_CODE\_RID。 |
| "RD" | SDDL\_REMOTE\_DESKTOP | 终端服务器用户。相应的RID是DOMAIN\_ALIAS\_RID\_REMOTE\_DESKTOP\_USERS。 |
| "RE" | SDDL\_REPLICATOR | 复制器。相应的RID是DOMAIN\_ALIAS\_RID\_REPLICATOR。 |
| "RO" | SDDL\_ENTERPRISE\_RO\_DCs | 企业只读域控制器。相应的RID为DOMAIN\_GROUP\_RID\_ENTERPRISE\_READONLY\_DOMAIN\_CONTROLLERS。 |
| "RS" | SDDL\_RAS\_SERVERS | RAS服务器组。相应的RID是DOMAIN\_ALIAS\_RID\_RAS\_SERVERS。 |
| "RU" | SDDL\_ALIAS\_PREW2KCOMPACC | 为使用与Windows 2000之前的操作系统兼容的应用程序的帐户授予权限的别名。相应的RID为DOMAIN\_ALIAS\_RID\_PREW2KCOMPACCESS。 |
| "SA" | SDDL\_SCHEMA\_ADMINISTRATORS | 架构管理员。相应的RID是DOMAIN\_GROUP\_RID\_SCHEMA\_ADMINS。 |
| "SI" | SDDL\_ML\_SYSTEM | 系统完整性级别。相应的RID是SECURITY\_MANDATORY\_SYSTEM\_RID。 |
| "SO" | SDDL\_SERVER\_OPERATORS | 服务器操作员。相应的RID是DOMAIN\_ALIAS\_RID\_SYSTEM\_OPS。 |
| "SU" | SDDL\_SERVICE | 服务登录用户。这是在作为服务登录时添加到进程令牌中的组标识符。相应的登录类型为LOGON32\_LOGON\_SERVICE。相应的RID是SECURITY\_SERVICE\_RID。 |
| "SY" | SDDL\_LOCAL\_SYSTEM | 本地系统。相应的RID是SECURITY\_LOCAL\_SYSTEM\_RID。 |
| "WD" | SDDL\_EVERYONE | 每个人。相应的RID是SECURITY\_WORLD\_RID。 |

`ConvertSidToStringSid`和`ConvertStringSidToSid`函数始终使用标准SID字符串表示法，并且不支持SDDL SID字符串常量。

