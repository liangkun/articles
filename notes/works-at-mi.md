# 小米工作总结

## 数据接入工作
### 手环数据每周批量导入
- hdfs数据, 已经进入数据工厂, 导入程序为xbd-importer, 部署在data03上.

### 天气数据实时导入
- hdfs数据, 已经进入数据工厂, 导入程序为weather-log, 部署并运行在data00上.

### passport数据实时导入user-event表
- hbase数据, 部署在data00上.

## 数据建设工作

### App相关基础数据
- app_profile, hdfs数据, 已经进入数据工厂, 每天更新.
- user_app_profile, hdfs数据, 已经进入数据工厂, 每天更新.
- xiaomi_device_info hbase表的app_infos, app_bitmap, phone_usage字段, 每天更新.

### 用户兴趣数据
下面两组字段都是每周更新

- xiaomi_device_info hbase表中的interest_xxx_level字段
- xiaomi_device_info hbase表中的curiosity_xxx_level字段

### 用户性别/年龄预测
在帅哥之前工作的基础上, 增加了url特征. 每周更新.

- xiaomi_device_info/xiaomi_user_olap 两hbase表中的sex, sex_ml字段.
- xiaomi_device_info/xiaomi_user_olap 两hbase表中的age, age_ml字段.

### data-event建设
- 主要是涉及event key结构的和快速查找的一些公共代码.
- passport导入的代码.

## xiaomi-data-api
这里面增加了以下一些api
- app相关的查询api, 如安装设备数, 活跃用户数等.
- data-event查询api.

## 基础库维护工作
基础库中有一些公共代码.
