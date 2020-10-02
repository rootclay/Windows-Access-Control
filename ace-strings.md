# （二）ACE字符串

## ACE字符串

安全描述符定义语言（SDDL）在安全描述符字符串的DACL和SACL组件中使用ACE字符串。 如“安全描述符字符串格式”示例所示，安全描述符字符串中的每个ACE都用括号括起来。 ACE的字段按以下顺序排列，并用分号（;）分隔

语法：`ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid;(resource_attribute)`

范围：

**ace\_type：ACE的类型**

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

**ace\_flags标志位**

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

**rights：权限**

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

## 

