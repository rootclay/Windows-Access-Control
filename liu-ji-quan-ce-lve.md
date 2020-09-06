# （六）集权策略

## **集中授权政策**

动态访问控制（DAC）方案可对企业文件服务器方案进行集中式访问控制管理。大多数组织都希望在多个区域中控制访问权限。

例如：

* 控制对敏感信息的访问，其中标记为敏感的文件将具有特定权限
* 控制对包含个人身份信息（PII）的文件的访问
* 根据组织保留策略限制对文档的访问

提供了几种新的授权策略抽象，以允许管理员集中定义这些策略，并通过允许分别定义和维护这些访问需求中的每一个但作为一个策略应用来简化定义过程。

Windows 8中引入了两个新的`Active Directory`策略对象，即中央授权策略（cap）和中央授权策略规则（capr），以基于声明和资源属性的表达式定义和应用集中式授权策略。在使用这些对象时，管理员将capr定义为特定的授权策略，可以将其应用于具有特定属性或满足特定适用性条件的资源。例如标记为“对业务有重大影响”的文件。在Windows 8 dac表达式方面，可以为可以表达的组织中的每个所需访问控制策略定义capes，并可以标识应该应用的资源。封顶是可一起应用于资源的封顶的集合。下图显示了cap和cape的关系，以及定义和将这些对象应用于文件资源所涉及的概念性步骤。

![relationship of capes and caps](https://docs.microsoft.com/zh-cn/windows/win32/secauthz/images/cap.png)

### **中央授权政策**

中央授权策略（CAP）将特定的授权规则（CAPR）收集到单个策略中。为了将特定的授权规则（CAPR）合并到组织的整体授权策略中，可以将CAPR一起引用，并应用于一组资源。这是通过将多个（通过引用）收集到CAP来完成的。一旦定义了CAP，资源管理器就可以分发它，以将组织授权策略应用于资源。

CAP具有以下属性：

* CAPR的集合–对现有CAPR对象的引用列表
* 标识符（Sid）
* 描述
* 名称

在访问评估期间，对管理员启用了文件和文件夹的CAP进行了评估。在`AccessCheck`呼叫期间，CAP检查在逻辑上与任意的ACL检查结合在一起。这意味着，为了获得对CAP适用的文件的访问权限，用户需要同时具有CAP（与之关联的CAPR）和文件上的任意ACL的访问权限。

CAP 例子：

```text
CORPORATE-FINANCE-CAP]
CAPID=S-1-5-3-4-56-45-67-123
Policies=HBI-CAPE;RETENTION-CAPR
```

CAP的定义

使用ADAC（或`PowerShell`）中的新UX在`Active Directory`中创建和编辑CAP，从而允许管理员创建CAP并指定组成该CAP的一组CAPR。

### **中央授权政策规则**

中央授权策略规则（CAPR）的目的是为组织的授权策略的孤立方面提供域范围的定义。管理员定义CAPR以执行特定的授权要求之一。由于CAPR仅定义了授权策略的一个特定的期望要求，因此，与将组织的所有授权策略要求汇总到单个策略定义中相比，可以更简单地定义和理解它。

CAPR具有以下属性：

* 名称–管理员标识CAPR
* 描述-定义CAPR的目的以及CAPR使用者可能需要的任何信息
* 适用性表达 - 定义将在其中应用策略的资源或情况
* ID - 用于审核CAPR更改的标识符
* 有效的访问控制策略 - Windows安全描述符，其中包含定义有效授权策略的DACL
* 异常表达 - 一个或多个表达式，提供了一种方法来覆盖策略并根据表达式的评估来授予对主体的访问权限
* 分期政策 - 可选的Windows安全描述符，其中包含定义了建议的授权策略（访问控制条目列表）的DACL，该授权策略将针对有效策略进行测试，但未执行。如果有效策略的结果与登台策略的结果之间存在差异，则该差异将记录在审核事件日志中。
  * 由于分段可能会对系统性能产生不可预测的影响，因此组策略管理员必须能够选择将在其中生效分段的特定计算机。这允许在OU的大多数计算机上采用现有策略，而在其中一部分计算机上进行暂存。
  * P2 –如果特定计算机上的暂存造成过多的性能下降，则该计算机上的本地管理员应该能够禁用暂存。
* CAP的反向链接–指向可能引用此CAPR的任何CAP的反向链接的列表。

在访问检查期间，将根据适用性表达式评估CAPR的适用性。如果CAPR适用，则需要评估CAPR是否向请求用户提供对标识资源的请求访问权限。然后，AND在逻辑上将CAPE评估的结果与资源上DACL的结果以及对资源有效的任何其他适用的CAPR进行合并。

CAPRs 例子：

```text
[HBI-POLICY]
APPLIES-TO="(@resource.confidentiality == HBI"
SD ="D:(A;;FA;;;AU;(@memberOf("Smartcard Logon")))"
StagingSD = "D:(A;;FA;;;AU;(@memberOf("Smartcard Logon") AND memberOfAny(Resource.ProjectGroups)))"
description="Control access to sensitive information"

[RETENTION-POLICY]
Applies-To="@resource.retention == true"
SD ="D:(A;;;FA;;BA)(A;;FR;;;WD)"
description="If the document is marked for retention, then it is read-only for everyone however Local Admins have 
full control to them to put them out of retention when the time comes"

[TEST-FINANCE-POLICY]
Applies-To="@resource.label == 'finance'"
SD="D:(A;;FA;;;AU;(member_of(FinanceGroup))"
description="Department: Only employees of the finance department should be able to read documents labeled as finance"
```

_CAPE定义_

通过Active Directory管理中心（ADAC）中提供的新UX创建CAPR。在ADAC中，提供了一个新的任务选项来创建CAPR。选择此任务后，ADAC将通过对话框提示用户，要求用户输入CAPR名称和说明。提供这些功能后，将启用用于定义任何其余CAPR元素的控件。对于其余的每个CAPR元素，UX将调出ACL-UI以允许定义表达式和/或ACL。

