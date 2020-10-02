# 2. SDDL

## 安全描述符定义语言

安全描述符定义语言（SDDL）定义了`ConvertSecurityDescriptorToStringSecurityDescriptor`和`ConvertStringSecurityDescriptorToSecurityDescriptor`函数用来将安全描述符描述为文本字符串的字符串格式。该语言还定义了字符串元素，用于描述安全描述符的组成部分中的信息。

针对本章节的学习建议使用[AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk)工具，该工具可以直接展示出指定对象的权限信息，并且能从SDDL到已解析的权限全面覆盖。

常用参数有：

1. -L，即展示SDDL；
2. -l显示解析后的安全描述符；
3. 不使用参数为展示通用的权限格式；
4. -v展示详细信息

![&#x5C55;&#x793A;SDDL](.gitbook/assets/image%20%2812%29.png)

![&#x5C55;&#x793A;&#x5B89;&#x5168;&#x63CF;&#x8FF0;&#x7B26;](.gitbook/assets/image%20%2817%29.png)

![&#x5C55;&#x793A;&#x6743;&#x9650;](.gitbook/assets/image%20%2811%29.png)

## 安全描述符字符串格式

安全描述符字符串格式是一种文本格式，用于在安全描述符中存储或传输信息。 `ConvertSecurityDescriptorToStringSecurityDescriptor`和`ConvertStringSecurityDescriptorToSecurityDescriptor`函数使用此格式。

格式是一个以空值（**null**）结尾的字符串，带有令牌（tiken），用于指示安全描述符的四个主要组成部分：所有者（O :\)，主要组（G :\)，DACL（D :\)和SACL（S :\)。

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

string\_ace : 在安全描述符的DACL或SACL中描述ACE的字符串。有关ACE字符串格式的说明，可参考本章第二节的ACE字符串。每个ACE字符串都括在括号（（））中。

可以从安全描述符字符串中省略不需要的组件。例如，如果未在输入安全描述符中设置`SE_DACL_PRESENT`标志，则`ConvertSecurityDescriptorToStringSecurityDescriptor`在输出字符串中不包含D：组件。您还可以使用`SECURITY_INFORMATION`位标志来指示要包含在安全描述符字符串中的组件。

安全描述符字符串格式不支持NULL ACL。

为了表示一个空的ACL，安全描述符字符串包括D：或S：令牌，没有其他字符串信息。

安全描述符字符串以不同的方式存储“安全描述符控制”位。`SE_DACL_PRESENT`或`SE_SACL_PRESENT`位由字符串中D：或S：令牌的存在指示。适用于DACL或SACL的其他位存储在`dacl_flags`和`sacl_flags`中。 `SE_OWNER_DEFAULTED`、`SE_GROUP_DEFAULTED`、`SE_DACL_DEFAULTED`和`SE_SACL_DEFAULTED`位未存储在安全描述符字符串中。 `SE_SELF_RELATIVE`位未存储在字符串中，但是`ConvertStringSecurityDescriptorToSecurityDescriptor`始终在输出安全描述符中设置此位。

以下示例显示了安全描述符字符串和关联的安全描述符中的信息。

```cpp
c:\users\public\videos\desktop.ini
  O:SYD:AI(A;ID;FA;;;BA)(A;ID;FA;;;SY)(A;ID;0x1301ff;;;IU)(A;ID;0x1301ff;;;SU)(A
;ID;0x1301ff;;;S-1-5-3)
```

O: SY代表该对象的Owner为System

![SYSTEM](.gitbook/assets/image%20%2810%29.png)

D:AI代表SDDL\_AUTO\_INHERITED为权限继承，之后的内容为ACE（ACE的具体解释在后面的小节逐一介绍）

另外一些例子如下：

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

## 

