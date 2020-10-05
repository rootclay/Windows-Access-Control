# （一）基于条件表达式ACE的SDDL

## ACE在的SDDL中的条件表达式

条件访问控制项（ACE）允许在执行访问检查时评估访问条件。安全描述符定义语言（SDDL）提供了用于以字符串格式定义条件ACE的语法。

条件ACE的SDDL与其他ACE相同，条件语句的语法是写在ACE字符串的末尾。

资源属性中的“＃”符号与“ 0”同义。例如，`D:AI(XA; OICI; FA ;;; WD;(OctetStringType ==#1#2#3##))`等效并解释为`D:AI(XA; OICI; FA ;;; WD;(OctetStringType ==#01020300))`。

### **条件ACE字符串格式**

安全描述符字符串中的每个ACE都用括号括起来。 ACE的字段按以下顺序排列，并用分号（;）分隔。

_AceType\*\*_;_**AceFlags**_;_**Rights**_;_**ObjectGuid**_;_**InheritObjectGuid**_;_**AccountSid**_;\(_**ConditionalExpression**_\)\*\*

这些字段如ACE字符串中所述，但以下情况除外。

* AceType字段可以是以下字符串之一。（当然不止这些，更多可以参考ACE字符串章节）

| ACE 类型的字符串 | Sddl.h中的常量 | AceType 值 |
| :--- | :--- | :--- |
| "XA" | SDDL\_CALLBACK\_ACCESS\_ALLOWED | ACCESS\_ALLOWED\_CALLBACK\_ACE\_TYPE |
| "XD" | SDDL\_CALLBACK\_ACCESS\_DENIED | ACCESS\_DENIED\_CALLBACK\_ACE\_TYPE |

* ACE字符串包含一个或多个条件表达式，并在字符串末尾的括号中。

### **条件表达式**

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

### **属性**

属性表示客户端上下文中`AUTHZ_SECURITY_ATTRIBUTES_INFORMATION`数组中的元素。属性名称可以包含任何字母数字字符和任何字符“：”，“ /”，“。”和“ \_”。

属性值可以是以下任何类型。

| 类型 | 描述 |
| :--- | :--- |
| Integer | 以十进制或十六进制表示的64位整数。 |
| String | 用引号分隔的字符串值。 |
| SID | 必须位于Member\_of或Device\_Member\_of的RHS上。 |
| BLOB | ＃后跟十六进制数字。如果数字的长度为奇数，则将＃转换为0使其变为偶数。同样，在值的其他位置出现的＃也将转换为0。 |

### **操作符**

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

### Unknown

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

### **条件ACE评估**

下表描述了根据条件表达式的最终评估得出的条件ACE的访问检查结果。

| ACE type | TRUE | FALSE | UNKNOWN |
| :--- | :--- | :--- | :--- |
| Allow | Allow | Ignore ACE | Ignore ACE |
| Deny | Deny | Ignore ACE | Deny |

### **案例**

下面通过一些示例说明SDDL定义的条件ACE是如何来表示指定的访问策略。

条件：如果同时满足以下两个条件，则允许对所有人执行

* Title = PM
* Division = Finance or Division = Sales

SDDL:

`D:(XA; ;FX;;;S-1-1-0; (@User.Title=="PM" && (@User.Division=="Finance" || @User.Division ==" Sales")))`



条件: 如果任何用户项目与文件项目相交，则允许执行。

SDDL：

`D:(XA; ;FX;;;S-1-1-0; (@User.Project Any_of @Resource.Project))`

## 

