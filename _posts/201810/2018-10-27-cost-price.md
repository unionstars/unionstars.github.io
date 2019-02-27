---
layout: post
title:  "仓报价系统"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

![电商系统业务架构图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-06-11-architecture/25.gif)

>就是你仓库里现货的一个平均价格，可能你每一个批次的进货价不一样，统计现在时间的整体平均价格。

### **联营结算平台概述**

线下店的经营方式，一般分为自营、联营、和租赁模式。所谓联营模式，就是商家借用线下店门店内部场地进行经营，由线下店门店统一管理的经营模式。

本文档旨在给出一种联营模式下商家结算的业务通用解决方案，其中自营模式和租赁模式，不在此文的研究范围内，但实际上，设计上是可以放在一起的。本文档只给出一个该业务模式下，一种可行的解决思路，并不提供设计的细节，更多细节请参考《联营结算详细设计》。

### **商户入驻结算**

#### **商户入驻结算概述**

这里研究的商户入驻，只关心与结算相关的部分。按照概述中所描述，线下店有自营、联营、租赁模式。自营和联营，涉及到商家上的区别，自营一般是平台方从供应商采购商品，平台采购后，进行了物权的转移，物权从供应商转移到平台方，利润所得归属平台方。而联营商实际物权，还是归属联营商家本身，当订单销售时，需要将利润进行清分计算，并按一定的规则结算给相关参与方。

随着电商的发展，商业形态开始多元化，多元化的商业合作模式，带来了结算的复杂化，这里简要举几个常见的形态，力在表述这些形态给系统层面带来的变化，比如：供销模式、门店销售模式、租户模式。

**供销模式**：供销模式的本质是，一个订单的结算，订单虽然属于分销商销售，但是此订单在某些商业场景内需要给供应商进行分润，增加了分润的关系，一般分润的关系，会通过计费系统进行清分计算。这种模式主要是引入了商家类型的多元化，由原始单一类型商家结算，变化为多类型商家结算的支持。相关的计费结算出的明细类似如下表格：

| 订单编号 | 商家编号 | ```商家类型``` | 付款商家编号 | 结算金额 |
| -------- | -------- | -------------- | ------------ | -------- |
| 9000001  | 1001     | 分销商         | 1000         | $1600    |
| 9000001  | 2001     | 供应商         | 1001         | $1200    |

**门店模式**：如现在比较火的线下零售店，线下店由联营商进行联合营运，而联营商在入驻不同门店时，实际上签署协议的时候很可能分门店进行合同签署，而不同门店的销售情况联营商也希望能分别结算到不同的账户，并且提供不同的流水支持。相关的计费结算出的明细类似如下表格：

| 订单编号 | 商家编号 | ```门店编号``` | 付款商家编号 | 结算金额 |
| -------- | -------- | -------------- | ------------ | -------- |
| 8000002  | 1001     | 0001           | 1000         | $1600    |
| 8000003  | 1001     | 0002           | 1000         | $1200    |

**租户模式**：多租户模式（英语：multi-tenancy technology），在SAAS化场景下，多租户模式一般有三种数据存储方式，数据库库分开；数据库相同，schema不同；同一数据库，同一schema，只是用租户编号进行区分。每一种都有自己的优缺，这里不赘述，这里的目的主要是引申出租户模式对商家账户的入驻与创建机制所带来的影响，这里以最简单的方式，也就是同一数据库、同一schema存储的方式进行表述，相关的计费结算出的明细类似如下表格：

| 订单编号 | ```租户编号``` | 商家编号 | 门店编号 | 付款商家编号 | 结算金额 |
| -------- | -------------- | -------- | -------- | ------------ | -------- |
| 8000002  | 1              | 1001     | 0001     | 1000         | $1600    |
| 8000003  | 2              | 1001     | 0002     | 1000         | $1200    |

而现实的商业模式中，一个公司实际上可以经营多个范围，那他在平台允许的情况下，可以建立多个商家，有时候，为满足个性化的促销需要，每个商家的下属门店又会根据订单的销售情况，给店内的导购员等进行分佣，现在这个表格看起来像是这样：

| 订单编号 | 租户编号 | 商家编号 | 门店编号 | 付款商家编号 | 结算金额 | ```公司编号``` | ```导购员编号``` |
| -------- | -------- | -------- | -------- | ------------ | -------- | -------------- | ---------------- |
| 8000002  | 1        | 1001     | 0001     | 1000         | $1600    | 1000           | 100001           |
| 8000003  | 2        | 1001     | 0002     | 1000         | $1200    | 1000           | 100002           |

上文中提到的这些角色，都一某种形态进行了业务的参与，必然会产生资金的归属划分。那这些参与方要产生对资金的结算，必然要维护自己的结算银行信息，现在这个表格看起来像是这样：

| 系统来源 | 租户编号 | 公司编号 | 商家编号 | 商家类型 | 门店编号 | 导购员编号 | 结算方式 | 结算资金编号 |
| -------- | -------- | -------- | -------- | -------- | -------- | ---------- | -------- | ------------ |
| 1        | 1        | 1000     | 1001     | 联营商   | 1000     | 1000001    | 银行卡   | 100001       |
| 2        | 2        | 1000     | 1001     | 供应商   | 1000     | 1000002    | 支付宝   | 100002       |

 我们可以看到随着着业务形态不断的变化，业务表在不断的庞大，如何在商户入驻的时候，解决这种结算关系，使业务系系统与结算平台解耦，是业务系统入驻结算时核心要解决的问题。这里提供一种可行的方案，我们可以将影响业务结算的这一组关系抽象出来，我们把这个抽象关系叫做结算的主体，这样结算平台的计费信息、结算信息、账务信息，都基于这个抽象的主体号进行数据流转，而不侵染具体的业务，后续结算、收付费、发票开具都通过此主体进行开展，做到自身的平台化。而这层关系的保存，也可以做到业务的回溯。

 结算主体创建的触发，要放在商家与平台合作关系正式创建时进行触发，一般是资质和合同审核通过后，触发结算平台按照一定的规则创建结算账户。

#### **商户入驻结算整体架构图**

![联营商户入驻结算](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/01-01.jpg)

#### **商户入驻结算整体架构图描述**

顺序的描述架构设计图中的几个核心组件：

**合同消息接收**：接收各业务平台商户合同审核通过的消息，对消息进行合法性校验，校验完成后触发结算主体的创建工作。

**结算主体的创建**：结算主体的创建，是强业务相关的，他关联一组关系，他强烈依赖于合同怎么签署，这一部分的规则策略，可以渐进迭代的设计，不建议一开始就抽象很高，后面通过公司运营方式的不通而逐步抽象完善。

**通知商家维护资金信息**：结算平台账户服务发送提醒商家维护收款方式，商家登录自己的管理端后进行收款方式及信息的维护。

**资金信息存储**：将商家选择的收款方式（微信、支付宝、银行卡）进行维护，并更新结算主体信息的结算方式，通过此方式路由具体的资金信息，之后发起商家资金信息的有效性验证。

#### **商户入驻结算交互序列描述**

```bash

@startuml
业务平台商户中心 -[#red]> 结算账户服务 : 合同审核(MQ)
结算账户服务 -[#red]> 结算账户服务 : 通过关系组生成结算主体:\n表:关联关系表\n表:结算主体信息表
结算账户服务 -[#red]> 商家门户 : 通知商家维护收款信息（MQ）
商家门户 -[#red]> 结算账户服务 : 维护商家信息、更新结算方式
商家门户 -[#red]> 鉴权服务中心 : （RPC）
鉴权服务中心 -[#red]> 结算账户服务 : 鉴权结果(MQ)
@enduml

```

#### **商户入驻结算交互序列图**

![商户入驻结算交互序列图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/01-02.png)

更多业务关联关系表和结算账户详细设计与缘由，参考《结算账户体系设计》

### **联营商计费**

#### **计费概述**

联营计费系统的职责是订单在销售过程中，产生的金额能够给参与方正确的清分。以联营计费来说，当用户订单完成时，它需要将该订单商家所得、平台方所得，进行正确的计算，大多数电商平台业所支持的业务，计费一般都是由业务系统在某个时点触发计费，一般都是通过MQ的方式异步通知计费系统进行计费操作，而金额的计算，计费系统又需要从各个业务系统中获取计费的因子，比如货款的计算公式为：```货款``` = ```商品价格``` - ```优惠券```，该公式所需要的商品价格和优惠券又需要从不同的系统获取。

我们可以看到，计费系统本身与外界业务系统具有强耦合性，他需要业务系统对计费进行触发，而金额计算公式所需要的因子，又根据业务的不同，通过不同的系统获取。那么如何降低这种耦合性，如何 ```快速支持不同类型业务的快速计费的接入，正确的对费用进行清分计算```，是计费系统核心要解决的问题。

#### **计费整体架构图**

![联营计费服务](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/02-01.jpg)

#### **计费整体架构图描述**

顺序的描述架构设计图中的几个核心组件：

**101 消息校验、生成指令任务**  此组件核心解决的问题是，接收业务系统的消息，做必要性校验，校验不通过的消息，直接做异常告警处理，通过的消息，将原消息落入es存储，并调用指令接收器。

**102 指令接收器** 由于大多数电商平台的业务计费都是在某个时点触发，此组件核心解决的问题是将外界系统与计费系统的耦合性，隔离在计费引擎之外，生成一条任务，在订单计费生命周期内进行调度。

**103 计费引擎** 计费引擎核心解决的是，提供一个通用的自动化计费装置，通过少量的设置即可进行自动化计费。引擎由3大核核心组件组成: 业务规则服务、数据快照服务、以及费用的计算执行服务,调度任务控制订单经过消息校验、计费引擎然后将订单的参与方的费用进行清分，生成可以结算计费明细。

**103-01 计费引擎-业务规则匹配器** 业务规则匹配器核心解决的是，业务系统的数据推送过来之后，如何识别并匹配到系统计费引擎中已经支持的业务规则。

![计费规则匹配流程](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/02-02.jpg)

**103-02 计费引擎-业务规则配置服务** 计费引擎业务规则的配置服务，配置计费引擎中可以支持的业务规则，业务规则实现后，供 ```103-01 计费引擎-业务规则匹配器``` 使用。

**103-03 计费引擎-数据快照服务** 数据快照核心解决的是获取计费过程中需要的计费要素（计费公式中的因子），例：```货款``` = ```商品价格``` - ```优惠券```，商品价格和优惠券是计费公式中的因子，快照的保存也为计费结果出现问题时可以回溯。

![数据快照服务](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/02-03.jpg)

**103-04 计费引擎-数据策略服务** 核心提供了计费公式中锁依赖的计费因子的获取来源关系，例：A业务的计费逻辑，其中某一费用的计算拿上边货款的计费公式来说，```货款``` = ```商品价格``` - ```优惠券```，其中商品价格需要从 ```商品系统``` 获取，优惠券需要从 ```优惠券系统``` 系统获取，这个时候，我们为每个系统配置一套数据策略，策略的实现中获取计费的因子，这样当一个业务流转到快照服务时，系统通过业务编号，获取业务与数据策略组，并执行数据策略组，返回计费公式所依赖的计费因子值。

**103-05 计费引擎-计费要素数据字典服务** 主要是计费要素的定位，我们称之为计费因子，计费因子通过计费因子服务来获取，每一项其实就是计费因子的定义。

**103-06 计费引擎-费用计算执行器** 核心完成的功能是接收前几步的机构，调用计费公式配置服务，获取计费计费公式进行计费并输出结果。

![费用计算执行器](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/02-04.jpg)

**103-07 计费引擎-计费公式配置服务** 主要维护各业务需要计算的费用的信息包括：费用状态（是否启用），费用类型，费用计算公式等。该数据主要存储在MySql当中。

**104 结算中心** 计费系统将结果进行相应的转换然后将明细推送给结算系统给商家进行结算。

#### **计费引擎交互序列图描述**

``` bash

@startuml
业务系统 -[#red]> 计费中心 : 订单妥投、售后消息通知(MQ)
计费中心 -[#red]> 计费中心 : 校验、保存消息:\nt表:订单消息表
计费中心 -[#red]> 计费中心 : 转换消息、生成指令(任务):\n表:订单计费任务表
计费中心 -[#red]> 计费中心 : 配置业务规则、匹配业务规则:\n表:计费业务规则表
计费中心 -[#red]> 计费中心 : 配置计费要素、配置要素来源、配置获取策略:\n表:计费因子字典表
计费中心 -[#red]> 计费中心 : 配置费用计算公式:\n表:费用计算表
计费中心 -[#red]> 计费中心 : 获取因子快照:\n表:计费数据快照表
计费中心 -[#red]> 计费中心 : 执行计费:\n获取快照、获取计费因子、执行公式、输出结果\n表:计费明细表
计费中心 -[#red]> 结算中心 : 推送计费结果(RPC)
结算中心 -[#red]> 计费中心 : 通知结算结果(MQ)
@enduml

```

#### **计费引擎交互序列图**

![计费引擎交互序列图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/02-05.png)

更多联合营商计费详细涉及与缘由，参考《联营商计费设计》

### **结算**

#### **结算概述**

联营结算系统根据合同中与商家签订的结算周期，为商家生成结算单，当计费中心的数据推送过来时，按照计费的业务完成时间，关联到该商家相对应的结算周期的结算单中，有一点很重的事，当业务复杂时，不同业务的不同费用，比如上边计费中我们所说的，业务:A业务；费用：货款，都有可能用不通的结算主体进行结算。这个配置关系看起来是这样：

| 租户编号 | 业务编号 | 费用编号 | 结算主体 |
| -------- | -------- | -------- | -------- |
| 1        | 1001     | 31       | 900001   |
| 2        | 1001     | 32       | 900002   |

然后我们根据配置关系，找到相应的结算主体，在结算单的周期结束时，我们关闭单子，然后进行结算单金额的计算，这里如果费用时收支两条线的时候，我们需要计算应收应付，然后根据结算主体和应收应付的不通，拆分为不同的收付款单据，进行收付款操作。

那我们看到，结算系统核心要解决的问题是：如何关联到正确的结算单、如何正确的找到自己的结算主体，一般称这个过程为清算分账（清分）、如何正确的进行费用的实时统计和关单后的全量统计。

#### **结算整体架构图**

![结算系统服务](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/03-01.jpg)

#### **结算整体架构图描述**

顺序的描述架构设计图中的几个核心组件：

**201 结算单生成服务** 根据商家合同定义的结算周期，为商家生成结算单。

**101 接收并校验明细** 接收计费系统或业务系统的计费明细，并进行有效性校验。

**102 明细的清分** 结算明细的清分指的是什么业务的什么费用，由谁来结算，比如彩票业务的A商家发生的货款由平台方的哪个主体为这个商家进行结算。

**103 明细增量统计** 当明细关联到结算单时，对该结算单待结算的费用进行增量统计，主要是商家可以实时看到自己待结算的金额。

**202 结算单关单** 结算单在账期结束的时候进行关单操作，关单后不再接受新的明细进入。

**104 结算单全量统计** 全量统计该结算单，关联到的明细，与增量统计结果进行比对，然后生成应收应付，商家确认应收应付无误后，进入平台的运营与财务审核，通过后触发下一步流程。

**105 拆分结算单** 按照结算的机构和应收应付拆分结算单为收付款单，当结算的方式为流水倒扣时，应收应付进行轧差，为收支两线时分开收款和付款。

#### **结算中心交互序列图描述**

```bash

@startuml
结算中心 -[#red]> 结算中心 : 创建结算单
计费中心 -[#red]> 结算中心 : 推送结算明细(RPC)
结算中心 -[#red]> 结算中心 : 接收并校验明细
结算中心 -[#red]> 结算中心 : 明细清分
结算中心 -[#red]> 结算中心 : 明细增量统计
结算中心 -[#red]> 结算中心 : 结算单关单
结算中心 -[#red]> 结算中心 : 结算单全量统计、商家确认
结算中心 -[#red]> 审批流中心 : 提交到审批流(RPC)
审批流中心 -[#red]> 结算中心 : 审批流状态(MQ)
结算中心 -[#red]> 收付款中心 : 拆分结算单、生成收付款单(RPC)
收付款中心 -[#red]> 结算中心 : 收付款状态(MQ)
@enduml

```

#### **结算中心交互序列图**

![结算中心序列图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/03-02.png)

联合营商结算详细涉及与缘由，参考《联营商结算设计》

### **收付费、收付款**

#### **收付费、收付款概述**

对于非订单、服务单相关的、非常态发生的费用（例如：质保金、押金、罚款、平台使用费等）走收付费系统，对应单据为收付费单，一般和结算单一样，经过审批之后，生成收付款单，那我们可以看到结算单和收付费单都是更贴近业务的平级的服务。收付款系统，相比来说比较纯碎，是一种付款的通道，按照上游系统结算主体的不通，路由不同的渠道进行收付。收付费核心实现的功能：收付费同步、查询、审批、生成收付款单等功能；收付款的的功能包括收付款单查询、发送等

#### **收付费、收付款整体架构图**

略

#### **收付费、收付款整体架构图描述**

**101 发起收付费** 一般由商户端或者运营端发起收付费申请。

**102 接收收付费** 接收收付费单、进行有效性验证、执行清分操作。

**103 进入审批流** 根据业务和费用判断是否需要进入审批流，审批后继续流转。

**104 生成收付款** 生成收付款单。

**104 渠道收付** 按渠道进行收付、完成后通知上游系统。

#### **收付费、收付款整体序列图描述**

```bash

@startuml
商户端 -[#red]> 收付费中心 : 创建收付费申请(RPC)
收付费中心 -[#red]> 收付费中心 : 接收收付费、验证、清分
收付费中心 -[#red]> OA : 进入审批流(RPC)
OA -[#red]> 收付费中心 : 审批完成状态(MQ)
收付费中心 -[#red]> 收付款中心 : 生成收付款单(RPC)
收付款中心 -[#red]> 收付费中心 : 收付款完成状态通知(MQ)
@enduml

```

#### **收付费、收付款整体序列图**

![收付费序列图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/04-01.png)

### **对账**

#### **对账系统概述**

以上的流程中，我们可以看到，商家的计费、结算、收付费、收付款，为了系统的性能考虑和高扩展性，都以积木方式构建，大多以微服务形式存在，并且对数据库的拆分会非常细化，但系统微服务化后，一个完整的业务单据比如(电商系统中的订单)，它的状态、分布于各个系统（订单的物流信息、订单的支付信息、订单的结算信息、开票情况），
这个时候，业务人员想要全流程跟踪此订单的时候，特别困难，需要在各个系统进行查看。账务系统中如何对这些数据进行整合，提供一个准确全面的账务信息，是对账系统核心要解决的问题。

这个时候需要对该订单的信息进行拼接一般采用如下方式:
    方式1：通过数据的离线加工，对数据进行建模，建立数据仓库，提供系统进行查询，这样建模是可以解决数据的多维度存储以及拼接查询。
    方式2:各个业务系统发送自己的MQ，然后由一个系统统一接收，接收之后对数据进行实时拼接，这样也可以作为一个准实时的拼接方案。
以上的两种方式都存在着自己明显的局限与问题
    方式1：此方式有明显的滞后性，不能满足实时的场景。
    方式2：虽然解决了拼接的实时性，但是对业务系统的侵入较大，必须由业务系统触发发送MQ消息。
如何做到对业务系统的解耦，并实现分布式数据库的数据的拼接，提供实时的查询，这里提供一个可行的方案：

#### **对账整体架构图**

![对账整体架构图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/05-02.jpg)

#### **对账款整体架构图描述**

**对账明细收集** 监控数据库从库binlog日志，将数据库的变化，实时发送到MQ中

**消费接收** 通过strom集群，接收每个待拼接数据源的MQ消息

**数据处理** 将消息后emit到多个bolt进行预处理，在预处理中对比本地关心的字段，如果字段关心，将字段放入到map中，然后将map通过fieldsGrouping交给聚合处理的bolt进行数据拼接

**数据存储** 数据存储，按业务时间1个月1个索引，然后半年的索引数据，关联到一个别名，数据获取层，通过别名提供服务，归档通过定时任务，将过期的索引数据，更新到归档的别名中。

这里为什么要引入storm和es的别名：

#### **对账整体序列图描述**

```bash

@startuml
binlog -[#red]> 消息中心  : 监控binlog日志并发送到消息中心
对账中心 -[#red]> 对账中心 : 通过storm集群，接收消息，并处理消息
对账中心 -[#red]> 对账中心 : 通过fieldsGrouping保证一个订单由一个节点处理，保证数据一致性
对账中心 -[#red]> 对账中心 : 通过别名提供服务，定时更改索引的别名，归档历史索引到历史别名中。
@enduml

```

#### **对账整体序列图**

![对账整体序列图](https://raw.githubusercontent.com/unionall/unionall.github.io/master/assets/images/pictures/2018-07-11-settle/05-03.png)