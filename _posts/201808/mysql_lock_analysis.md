---
layout: post
title:  "mysql 死锁问题分析"
date:   2018-07-11 12:00:00

categories: other
tags: mysql
author: "张学刚"
---

### **场景描述**

2个用户之间转账功能的接口及其内部实现 的核心代码，尽量是一个可以运行的代码程序。

要求：完成接口设计、并实现其内部逻辑，以完成A用户转账给B用户的功能。2个用户的账户不在同一个数据库下。注意尽量不要用伪代码。

提示：接口发布后会暴露给外部应用进行服务调用，请考虑接口规范、安全、幂等、重试、并发、有可能的异常分支、分布式场景下的事务一致性、用户投诉、资金安全等的处理。

### **实现思路**
 这里的实现思路，主要是采用本地消息表的方式， 核心思想是把分布式事务拆分成本地事务进行处理，然后通过消息状态异步的保证最终的一致性

![本地消息表](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-03-transfer/01-01.png)

转账场景的实现思路：

这里为了便于阐述，下面将A、B之间的转账，以A、B两个系统来阐述，A为转款方，B为收款方，
消息生产方，在这里对应A系统，建立一个任务消息表，任务表和业务数据要在一个事务里提交，也就是说他们分库分表的策略键要一样。事务成功后发送扣款成功的消息。

消息消费方，在这里是B系统，接收扣款成功的消息，并完成自己的业务逻辑。此时如果本地事务处理成功，表明已经处理成功了，如果处理失败，那么就会重试执行。如果是业务上面的失败，可以给A系统发送一个业务补偿消息，通知A系统进行回滚等操作，如果成功也通知A系统修改状态为最终态。

这里如果B系统通知A系统的时候失败了，或者A系统消息在极端情况下没有到达B系统，A系统会按照业务的规则，对非最终状态的任务启动分布式任务进行补偿，这个时候数据可能会重发，B系统接口一定要保证幂等和防重处理。

这种方案遵循BASE理论，采用的是最终一致性，这种对于2PC的优点不会出现像2PC那样复杂的实现(当调用链很长的时候，2PC的可用性是非常低的)， 与TCC相比不会像TCC那样可能出现确认或者回滚不了的情况。

#### **系统交互流程**

![事务的处理流程](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-03-transfer/01-02.png)

这里面是以A系统的最终态，来保证数据的最终一致性，这里还有一种方案，将对账的这一块作为一个计算密集型的模块，单独拆分出来，作为一个微服务部署，然后，对账系统，负责接收A与B的消息，进行对账，如果A/B存在差异的时候，对账系统发起补偿，来调度A或者B系统，通过主动查询和补偿的方式，达到最终一致性。

### **考虑因素的保证措施**

- 接口规范： 定义接口规范、校验参数合法性及有效性，返回可识别的错误码
- 安全：采用了MD5加签的方式，用秘钥对传输数据进行最MD5签名，数据接收后，进行验签，防止篡改
- 幂等：通过任务消息表唯一索引以及与业务表同一事务的方式进行幂等控制。
- 重试：MQ重试，异步任务重试。
- 并发：使用乐观锁和唯一索引、事务进行并发的控制
- 异常分支：考虑安全性校验不通过、参数校验不通过、重试不通过、余额不足、扣款付款失败等情况
- 事务一致性保障的核心思路，遵循BASE理论，采用最终一致性的方式解决
- 用户投诉，使用枚举定义错误，客户端可以根据枚举类型对前端用户进行友好的提示
- 资金安全：通过乐观锁保证不扣错，使用唯一约束和事务保证不重复扣

#### **程序的关键点说明**

- 保证转账任务表和用户余额表在同一事务进行处理
- 有效的补偿策略

#### **技术选型与代码结构描述**

- 项目基于springboot
- 分库分表采用开源的sharding-jdbc， 分库分表策略是使用用户编号进行哈希取模。
- MQ采用的是京东开源的JMQ
- 分布式任务用的是elastic-job

项目包划分：

- constants 系统全局常量的一定义
- domain 实体对象
- dto 数据传输对象
- enums 枚举类定义
- exception 系统自定义异常
- job 分布式调度任务
- manager 事务处理
- mapper mybstis映射类
- mq 异步消息队列生产消费与实现
- service 核心服务接口
- util 系统所使用的工具类
- resources 系统相关配置信息，以及文档存放

#### **核心类的关系图**

![事务的处理流程](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-03-transfer/01-03.png)

#### **待优化项**

- 对接入系统的校验与降级处理
- 消息中间件，没有启动一个服务端
- 接口高频次调用的处理方式，如果MQ不可用后的降级方式

### 画图源文件

```bash
@startuml
转账客户端 -[#red]> 系统A : 发起转账请求(HTTP/RPC)
系统A -[#red]> 系统A : 验签
系统A -[#red]> 系统A : 参数校验
系统A -[#red]> 系统A : 1. 创建本地任务消息表(防重和幂等基于唯一索引)\n2. 查询余额\n3. 扣款(数据库乐观锁防止并发)\n4. 发扣款消息\n[以上都在一个事务中进行]
系统A -[#red]> 消息中间件 : 发送扣款信息，在事务中发送(MQ)
消息中间件 -[#red]> 系统B : 拉取扣款消息(MQ)
系统B -[#red]> 系统B : 验签
系统B -[#red]> 系统B : 参数校验
系统B -[#red]> 系统B : 1. 创建本地消息表(防重和幂等基于唯一索引)\n2. 加款(数据库乐观锁防止并发)\n3. 发送成功或者失败的消息\n[以上都在一个事务中进行]
系统B -[#red]> 消息中间件 : 发送加款成功/失败的消息(MQ)
系统A -[#red]> 消息中间件 : 接收加款成功/失败的消息(MQ)
系统A -[#red]> 系统A : 更新消息表状态
系统A -[#red]> 系统A : 未达到最终态的任务消息，进行补偿重发
对账系统 -[#red]> 对账系统: 考虑引入？
@enduml

@startuml
interface AccountOperationService {
   boolean signCheck(String signStr, String transferInfo)
   boolean decrease(TransferDTO transferDTO)
   boolean increase(TransferDTO transferDTO)
   boolean revertDecrease(TransferDTO transferDTO)
}
interface TransferAccountsService{
    transferAccounts(FundTransToaccountTransferRequest request)
}
class AccountOperationServiceImpl{
   boolean signCheck(String signStr, String transferInfo)
   boolean decrease(TransferDTO transferDTO)
   boolean increase(TransferDTO transferDTO)
   boolean revertDecrease(TransferDTO transferDTO)
}
class TransferAccountsServiceImpl {
  transferAccounts(FundTransToaccountTransferRequest request)
}
class TansferTaskDecreaseMessageListener {
  onMessage(String message)
}
class TansferTaskIncreaseMessageFailListener {
  onMessage(String message)
}
class TansferTaskIncreaseMessageSuccessListener {
  onMessage(String message)
}
package job <<Rectangle>> {
  class TransferTaskDealElasticJob {
  process(JobExecutionMultipleShardingContext jobExecutionMultipleShardingContext)
}
}
AccountOperationService <|-- AccountOperationServiceImpl
TransferAccountsService <|-- TransferAccountsServiceImpl
AccountOperationServiceImpl <.. TransferAccountsServiceImpl
AccountOperationServiceImpl <.. TansferTaskDecreaseMessageListener
AccountOperationServiceImpl <.. TansferTaskIncreaseMessageFailListener
TransferTaskManagerImpl <.. TansferTaskIncreaseMessageSuccessListener
AccountOperationServiceImpl <.. TransferTaskDealElasticJob]
enum TaskStatusEnum {
  SUCESS(1, "处理成功"),
  FAILED(2, "收款系统业务异常"),
  FINISHED(3, "收款完成");
}
enum ErrorCodeEnum {
   BAD_REQUESTS("BAD_REQUESTS", "非法请求"),
   ARG_ERROR("ARG_ERROR", "参数为空!"),
   BALANCE_ERROR("BALANCE_ERROR", "账户余额不足"),
   DECREASE_ERROR("DECREASE_ERROR", "账户余额扣减失败"),
   INCREASE_ERROR("INCREASE_ERROR", "账户余额增加失败"),
   TRANSFER_ERROR("TRANSFER_ERROR", "账户转账失败");
}
@enduml
```
