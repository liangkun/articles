# 实时风险控制调研

## 名词

- PIN: Personal Identification Number
- PAN: Primary Account Number
- CNP: Card Not Present (transaction)
- CVV: Card Verification Value
- POS: Point Of Sale
- ATM: Automated Teller Machine
- Basis Point: 指标, 指平均每100元, 由于欺诈带来的损失是多少分
- POC: Point Of Compromise, 卡片信息泄漏的地方
- PoD: 指标, Point of Detection, 表示对于某个发生欺诈交易的账户,
  在系统在第几个欺诈交易发生时才发现(发出第一个预警), PoD为5表示放过了4个欺诈交易.

## 数据分类

- card profile
- customer profile, 描述用户的行为模式(常用的ATM地点, 交易频率, 经常的交易时间, 平均交易额, 是否常旅行,
  是否常出国等)
- current transaction
- calculated dynamic data

## 常见形式

- 使用卡片/账号信息进行欺诈交易: 柜台交易, ATM交易, POS机交易, 网络交易
- Checker, 欺诈者会通过小额交易尝试大量卡片信息, 如果某个成功, 则表示对应的账户信息可用, 然后再进行大额交易
- 重复支付, 通过重复进行一个正常交易, 可能会修改部分信息, 再次让账户支出资金.
- 使用假身份申请账户
- 账户控制, 比如, 通过修改一个账户的个人信息(地址, 电话), 通过挂失等手段获得账户的控制权.

## 常见原因

- 卡片丢失
- 账户信息泄露

## 常见规则

**异常** 指"针对特定的用户, 不是常用的或常常发生情况, 不符合该用户平常的行为模式".

**可疑** 指"这个地方(情况下)经常发生欺诈行为".

- 交易发生地异常
- 交易频率异常
- 交易金额异常
- 交易延时异常
- 交易序列(时间上的序列)异常
- 交易账号(卡片)是否去过POC (账号信息可能已经泄漏)
- 可疑交易(收款)账号
- 可疑交易地点
- 可疑交易(收款)商家

## 服务提供商

- FICO (Falcon, NN based)
- Actimize
- ACI (Proactive Risk Manager)
- SAS (SAS Fraud Management)
- BAE Systems
- IBM
- Ethoca

## 常用技术

- 专家系统, 类似于我们目前做的规则引擎
- 机器学习系统
  - neural network
  - fuzzy neural network
- 混合机器学习与专家系统, 这必然是我们的选择

## 产品功能

- 实时风险预测
  - 规则管理
  - 监控与展示
- case管理系统

## 项目计划

## 参考资料

[sentry资料文档]

[Credit Card Fraud](https://en.wikipedia.org/wiki/Credit_card_fraud)

[Data Analysis Techniques For Fraud Detection]
(https://en.wikipedia.org/wiki/Data_analysis_techniques_for_fraud_detection)

[5 Things to Know About Detecting Credit Card Fraud]
(https://www.ibm.com/developerworks/community/blogs/5things/entry/5_things_to_know_about_detecting_credit_card_fraud)
