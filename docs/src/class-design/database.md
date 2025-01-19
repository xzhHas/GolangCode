---
title: 数据库商品管理系统【数据库原理与应用课设】
shortTitle: 2.MySQL商品管理系统
date: 2025-01-17
sticky: true
start: true
category:
  - 课程设计
tag:
  - 课程设计
---

## 1.系统需求分析

当今，不论是超市还是公司企业，都广泛采用信息化管理，旨在提升管理水平和工作效率，同时极大程度地减少手工操作可能引发的错误。在这样的背景下，进销存管理信息系统崭露头角。在工厂环境中，产品的进销存管理牵涉到原材料的采购、库存、生产投入、报损等多个环节，甚至有时还牵涉到销售。同时，针对产品，还有生产、库存、销售、报损等相应环节需要全面考虑。在其他非生产性单位，比如超市和商店，主要涉及进货、库存、销售和报损等四个方面。

网上商品管理系统以分类和分级的方式全面管理和监控仓库，显著缩短了商品信息流转时间，使企业的物资管理层次井然有序，为采购和销售提供可靠依据。系统能够自动提示存货短缺、超储等异常状况。此外，商品管理功能的完善，有助于全面控制和管理企业存货，优化商品成本结构，提升企业的市场竞争力。这种信息化管理系统为企业带来了更高效的生产流程和更精准的物资管理，是现代企业发展不可或缺的一部分。
### 1.2目标与任务
1.2.1 需求分析阶段的目标
(1) 通过实地考察和深入体验，详细了解商品管理系统的各项功能，以及它们之间的联系和操作流程。通过仔细观察，获取关于商品采购、库存管理、销售等各个环节的信息，全面了解系统的实现方式。
(2) 运用实地调查和问答记录的方式，系统地了解商品管理系统的工作业务流程，并记录和处理相关的数据。这包括对商品的进货流程、库存管理流程、销售流程等方面的调查，通过与系统操作人员的交流，收集实际数据，为后续的概念设计和逻辑设计提供充分的信息支持。
(3) 主动与指导教师交流个人想法，征求意见，并根据反馈改正不合理的地方。这个阶段的交流有助于更好地理解商品管理系统的需求和挑战，并为后续的概念设计和逻辑设计提供更为准确和可行的基础。通过与指导教师的深入沟通，确保系统设计符合实际需求，提高系统的实用性和有效性。
### 1.3需求分析阶段的任务
（1）商品管理功能分析
通过深入详细的调查和多方面搜集资料，结合实地考察等方法，总结研究出了商品管理系统的基本业务功能，同时加入销售大数据模型和商品流行两个方面，具体如下：
（1）商品信息维护：主要负责对新进商品进行编号、登记和入库等操作，确保系统中准确记录商品信息。
（2）供应商信息维护：完成供应商信息的添加、修改和删除等操作，只有合法供应商的商品才会被纳入系统管理。
（3）进货/销售处理：执行商品的进货和销售活动，记录相关进销情况，实时更新库存信息。
（4）交易记录查询：提供查询功能，让用户能够了解每笔交易的详细情况，包括交易时间、商品信息等。
（5）商品库存检索：用户能够根据不同信息（如商品名称、类别、关键词等）对库存情况进行检索，方便快速找到所需商品。
（6）库存预警通知：为系统管理员提供统计信息，及时通知库存状况，包括低库存、过量库存等异常情况。
（7）商品预定信息：用户通过系统可以进行商品预定，预定成功后可以进行购买。
（8）欠款信息查询：用户可以通过系统查询个人或企业的欠款信息，确保财务状况清晰可见。
（9）商品类别信息：用户可根据需要查询特定类别的商品，以便更快捷地找到目标商品。
（10）用户留言：用户如果有意见或建议，可以通过系统进行留言，提供一个良好的沟通渠道。
（11）在线商品浏览：用户可以通过系统进行在线浏览商品，有效用户可以登录系统进行浏览。
（12）销售大数据模型：引入销售大数据模型，分析销售数据，提供销售趋势、热门商品等信息，帮助商家更好地制定销售策略。
（13）商品流行分析：通过对销售数据的挖掘，分析商品的流行趋势，帮助商家了解市场需求，调整库存和采购策略，提高商品销售的效益。

## 2.概念设计
### 2.1概念设计任务

概念设计是独立于数据库管理系统的设计，它的主要任务即时完成对现实事物，事物关系之间的转化，把抽象的事物转化成能够被人们易于理解的图形关系，更加直白的把现实的事物关系表达出来，从而为下一步的设计打下一个良好的基础，概念设计的主要任务就是如此，进行归类总结，识别网上商品后台管理系统中的实体，识别实体的属性，识别实体的关键字，识别实体间的联系，利用实体关系图（E—R图）来描述商品管理相关实体、属性及关系，从而达到为商品管理系统建立良好的数据模型的目的，
### 2.2概念模型设计
根据前面的设计，以及相应的数据项和数据结构之间的关系，我们可以将商品管理系统的数据库实体划分为几个主要的实体集。在商品管理系统中，这些实体集可以包括“商品信息实体集”、“用户信息实体集”、“仓库信息实体集”、“订单信息实体集”和“缺货信息实体集”，每个实体集中还包含不同的具体实体。

在商品管理系统中，用户必须注册并登录才能办理相关业务。新用户首先需要注册账号，而当用户丢失账号信息时，需要进行账号挂失，以防止账户信息的滥用。用户登录系统后可以执行多种操作，如查询订单状态、购买商品、查询订单历史等。如果用户发现所需商品缺货，可以进行缺货登记。

系统管理员定期检查缺货信息表，对于需求量大的商品进行补货采购。此外，用户还可以进行商品预订，登录账号后发布相关需求，管理员对这些需求进行分析和回应，给出相应的处理结果。用户还可以通过账号查询其他信息，例如费用欠缴情况、商品信息查询，以及在线浏览电子产品的描述和图片等。

每个实体定义的属性如下：
管理员：{管理员id，用户名, 密码, 权限, 电话, 邮箱, 状态}
用户：{用户id，用户名, 编号, 密码, 邮箱, 性别, 电话, 学历, 爱好}
权限角色：{权限id，角色名称, 权限控制, 控制器}
权限：{权限名称, 有该权限的用户, 权限等级}
商品：{商品名称, 价格, 数量, 重量, 类型, 详情, 图片, 时间, 状态}
商品属性：{attr_id,cat_id,属性名称, 类型, 时间, 选值}
订单：{order_id,会员信息, 订单编号, 金额, 支付方式, 交易账单, 发票, 公司名称, 收货人地址, 订单状态, 时间}  
收货人信息：{cgn_id，会员id, 收货人名称, 地址, 电话, 邮编, 时间}
快递信息：{express_id，订单id, 编号, 快递公司, 时间}
商品类别：{父类id, 名称, 分类, 标记, 时间}
## 3.逻辑设计
### 3.1逻辑设计的目标和任务
以上的概念设计阶段是独立于任何一种数据模型的，但是逻辑设计阶段就与选用的DBMS产品发生关系了，系统逻辑设计的任务就是将概念设计阶段设计好的基本E-R图转换为选用DBMS产品所支持的数据模型相符合的逻辑结构。具体内容包括数据组织（将E-R图转换成关系模型、模型优化、数据库模式定义、用户子模式设计）、数据处理（画出系统功能模块图）两大任务。其中最为关键的是把ER模型转换成相应的关系表结构，同时每个关系模型之间的范式应最好满足第三范式，只有这样的关系模式才可能尽可能的减小冗余，达到较好的效果。
### 3.2关系模型设计
#### 3.2.1 ER转化关系模型
ER图进行关系模型的转化时，应根据相应的规则进行转化，只有这样，才能尽可能的减小冗余，达到比较好的范式，使模型更加优化，通常的转换规则如下：

一对一联系 ：若双方部分的参与，则将联系定义为一个新的关系，属性为参与双方的码，若一方全部参与，则将联系另一方的码作为全部参与一方的属性，
一对多联系：将单方参与实体的码作为多方参与实体的属性，
多对多联系：将联系定为新的关系，属性为参与双方的码。

以上也就是基本的设计规则了，只要按照相应的规则转换，就能够得到所要的规范程度，得到一个良好的范式，根据得到的ER图，进行关系模式的转换。具体的关系模型如下：

管理员

```c
数据项名	数据类型	别名	isNull	键
mg_id	int(11)	主键id	否	主键
mg_name	varchar(32)	名称	否	无
mg_pwd	char(64)	密码	否	无
role_id	tinyint(11)	角色id	否	无
mg_mobile	varchar(32)		是	无
mg_email	varchar(64)		是	无
mg_state	tinyint(2)	1：表示启用 0:表示禁用	是	无
create_time	datetime		否	无
update_time	datetime		是	无
```

用户

```c
数据项名	数据类型	别名	isNull	键
user_id	int(11)	自增id	否	主键
username	varchar(128)	登录名	否	无
qq_open_id	char(32)	qq官方唯一编号信息	是	无
password	char(64)	登录密码	否	无
user_email	varchar(64)	邮箱	否	无
user_email_code	char(13)	新用户注册邮件激活唯一校验码	是	无
is_active	enum('是','否')	新用户是否已经通过邮箱激活帐号	是	无
user_sex	enum('保密','女','男')	性别	否	无
user_qq	varchar(32)	qq	否	无
user_tel	varchar(32)	手机	否	无
user_xueli	enum	学历	否	无
user_hobby	varchar(32)	爱好	否	无
user_introduce	text	简介	是	无
create_time	datetime	创建时间	否	无
update_time	datetime	更新时间	否	无
user_eamil	varchar(255)	邮箱	否	无
user_eamil_code	varchar(255)	邮箱code	是	无
```

购物车

```c
数据项名	数据类型	别名	isNull	键
cart_id	int(11) unsigned	主键	否	主键
user_id	int(11) unsigned	用户id	否	无
cart_info	text	购物车详情信息，二维数组序列化信息	是	无
created_at	timestamp		是	无
updated_at	timestamp		是	无
delete_time	timestamp		是	无
```

权限管理

```c
数据项名	数据类型	别名	isNull	键
ps_id	smallint(6) unsigned		否	主键
ps_name	varchar(20)	权限名称	否	无
ps_pid	smallint(6) unsigned	父id	否	外键
ps_c	varchar(32)	控制器	否	无
ps_a	varchar(32)	操作方法	否	无
ps_level	enum('0','2','1')	权限等级	否	无
ps_icon	char(30)		否	无
```

订单关联

```c
数据项名	数据类型	别名	isNull	键
id	int(10) unsigned	主键id	否	主键
order_id	int(10) unsigned	订单id	否	外键
goods_id	mediumint(8) unsigned	商品id	否	外键
goods_price	decimal(10,2)	商品单价	否	无
goods_number	tinyint(4)	购买单个商品数量	否	无
```


下订单

```c
数据项名	数据类型	别名	isNull	键
is_send	enum('是','否')	订单是否已经发货	否	无
trade_no	varchar(32)	支付宝交易流水号码	否	无
order_fapiao_title	enum('个人','公司')	发票抬头 个人 公司	否	无
order_fapiao_company	varchar(32)	公司名称	否	无
order_fapiao_content	varchar(32)	发票内容	否	无
consignee_addr	text	consignee收货人地址	否	无
pay_status	enum('0','1')	订单状态： 0未付款、1已付款	否	无
create_time	int(10) unsigned	记录生成时间	否	外键
update_time	int(10) unsigned	记录修改时间	否	无
```


```c
商品
数据项名	数据类型	别名	isNull	键
goods_id	mediumint(8) unsigned	主键id	否	主键
goods_name	varchar(255)	商品名称	否	无
goods_price	decimal(10,2)	商品价格	否	外键
goods_number	int(8) unsigned	商品数量	否	无
goods_weight	smallint(5) unsigned	商品重量	否	无
cat_id	smallint(5) unsigned	类型id	否	无
goods_introduce	text	商品详情介绍	是	无
goods_big_logo	char(128)	图片logo大图	否	无
goods_small_logo	char(128)	图片logo小图	否	无
is_del	enum('0','1')	0:正常  1:删除	否	无
add_time	int(11)	添加商品时间	否	外键
update_time	int(11)	修改商品时间	否	无
delete_time	int(11)	软删除标志字段	是	无
cat_one_id	smallint(5)	一级分类id	是	无
cat_two_id	smallint(5)	二级分类id	是	无
cat_three_id	smallint(5)	三级分类id	是	无
hot_mumber	int(11) unsigned	热卖数量	是	无
is_promote	smallint(5)	是否促销	是	无
goods_state	int(11)	商品状态 未通过审核已审核	是	无
```


商品关联

```c
数据项名	数据类型	别名	isNull	键
id	bigint(20)	Id	否	主键
product_id	bigint(20)	商品id	否	无
product_desc	longtext	信息	否	无
```

商品属性

```c
数据项名	数据类型	别名	isNull	键
id	int(10) unsigned	主键id	否	主键
goods_id	mediumint(8) unsigned	商品id	否	无
attr_id	smallint(5) unsigned	属性id	否	外键
attr_value	text	商品对应属性的值	否	无
add_price	decimal(8,2)	该属性需要额外增加的价钱	是	无
```

#### 3.2.2关系模型优化
关系模式管理员、用户、商品、购买、订单、权限等每一个关系不存在非主属性对主属性的部分函数依赖，也不存在传递函数依赖，已经达到了3NF，基本上都满足应用系统的要求，只是在应用中还有一部分功能的实现过于简单，没有考虑周全，还有待进一步修改。已得到更好的运行效率。
#### 3.2.3用户子模式设计
用户子模式的建立，其功能就是方便用户的查询并起到了一定的保护数据库的作用，视图的建立应根据具体的应用情况，根据用户的需求，进行相应的视图建立，建立视图的原则应在尽量满足用户的需求的前提下进行，并同时保护其他的数据的安全性，以免数据的泄露与破坏，数据库视图的建立在下面有相应的举例及应用，这里就不再多说了。

### 3.3数据处理
系统功能模块图
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182216950.png)

## 4.数据库设计
### 5.1建立数据库，数据表，视图，索引
#### 5.1.1建立数据库

```sql
CREATE DATABASE IF NOT EXISTS dbtest;
```

#### 5.1.2建立数据表
商品系统数据表建立：
1、建立管理员表

```sql
CREATE TABLE `sp_manager` (
  `mg_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `mg_name` varchar(32) NOT NULL COMMENT '名称',
  `mg_pwd` char(64) NOT NULL COMMENT '密码',
  `role_id` tinyint(11) NOT NULL DEFAULT '0' COMMENT '角色id',
  `mg_mobile` varchar(32) DEFAULT NULL,
  `mg_email` varchar(64) DEFAULT NULL,
  `mg_state` tinyint(2) DEFAULT '1' COMMENT '1：表示启用 0:表示禁用',
  `create_time` datetime NOT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`mg_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='管理员表';
```

2、建立商品描述表

```sql
CREATE TABLE `product_desc` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `product_id` bigint(20) NOT NULL,
  `product_desc` longtext NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品描述';
```

3、建立商品类别表

```sql
CREATE TABLE `product_type` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(12) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `pname` varchar(12) NOT NULL,
  `flag` tinyint(1) NOT NULL,
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  `create_user` bigint(20) NOT NULL,
  `update_user` bigint(20) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品的类别';
```

4、建立属性表

```sql
CREATE TABLE `sp_attribute` (
  `attr_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `attr_name` varchar(32) NOT NULL COMMENT '属性名称',
  `cat_id` smallint(5) unsigned NOT NULL COMMENT '外键，类型id',
  `attr_sel` enum('only','many') NOT NULL DEFAULT 'only' COMMENT 'only:输入框(唯一)  many:后台下拉列表/前台单选框',
  `attr_write` enum('manual','list') NOT NULL DEFAULT 'manual' COMMENT 'manual:手工录入  list:从列表选择',
  `attr_vals` text NOT NULL COMMENT '可选值列表信息,例如颜色：白色,红色,绿色,多个可选值通过逗号分隔',
  `delete_time` datetime DEFAULT NULL COMMENT '删除时间标志',
  PRIMARY KEY (`attr_id`) USING BTREE,
  KEY `type_id` (`cat_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=3806 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='属性表';
```

5、建立收获人信息表

```sql
CREATE TABLE `sp_consignee` (
  `cgn_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `user_id` int(11) NOT NULL COMMENT '会员id',
  `cgn_name` varchar(32) NOT NULL COMMENT '收货人名称',
  `cgn_address` varchar(200) NOT NULL DEFAULT '' COMMENT '收货人地址',
  `cgn_tel` varchar(20) NOT NULL DEFAULT '' COMMENT '收货人电话',
  `cgn_code` char(6) NOT NULL DEFAULT '' COMMENT '邮编',
  `delete_time` int(11) DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`cgn_id`) USING BTREE,
  KEY `user_id` (`user_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='收货人表';
```

6、建立快递表

```sql
CREATE TABLE `sp_express` (
  `express_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` int(10) unsigned NOT NULL COMMENT '订单id',
  `express_com` varchar(32) DEFAULT NULL COMMENT '订单快递公司名称',
  `express_nu` varchar(32) DEFAULT NULL COMMENT '快递单编号',
  `create_time` int(10) unsigned NOT NULL COMMENT '记录生成时间',
  `update_time` int(10) unsigned NOT NULL COMMENT '记录修改时间',
  PRIMARY KEY (`express_id`) USING BTREE,
  KEY `order_id` (`order_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='快递表';
```

7、建立关联表

```sql
CREATE TABLE `sp_goods_attr` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `goods_id` mediumint(8) unsigned NOT NULL COMMENT '商品id',
  `attr_id` smallint(5) unsigned NOT NULL COMMENT '属性id',
  `attr_value` text NOT NULL COMMENT '商品对应属性的值',
  `add_price` decimal(8,2) DEFAULT NULL COMMENT '该属性需要额外增加的价钱',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `attr_id` (`attr_id`) USING BTREE
) ENGINE=MyISAM AUTO_INCREMENT=3184 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='商品-属性关联表';
```

8、建立关联表

```sql
CREATE TABLE `sp_goods_pics` (
  `pics_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `goods_id` mediumint(8) unsigned NOT NULL COMMENT '商品id',
  `pics_big` char(128) NOT NULL DEFAULT '' COMMENT '相册大图800*800',
  `pics_mid` char(128) NOT NULL DEFAULT '' COMMENT '相册中图350*350',
  `pics_sma` char(128) NOT NULL DEFAULT '' COMMENT '相册小图50*50',
  PRIMARY KEY (`pics_id`) USING BTREE,
  KEY `goods_id` (`goods_id`) USING BTREE
) ENGINE=MyISAM AUTO_INCREMENT=4582 DEFAULT CHARSET=utf8 ROW_FORMAT=FIXED COMMENT='商品-相册关联表';
```

9、建立商品表

```sql
CREATE TABLE `sp_goods` (
  `goods_id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `goods_name` varchar(255) NOT NULL COMMENT '商品名称',
  `goods_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '商品价格',
  `goods_number` int(8) unsigned NOT NULL DEFAULT '0' COMMENT '商品数量',
  `goods_weight` smallint(5) unsigned NOT NULL DEFAULT '0' COMMENT '商品重量',
  `cat_id` smallint(5) unsigned NOT NULL DEFAULT '0' COMMENT '类型id',
  `goods_introduce` text COMMENT '商品详情介绍',
  `goods_big_logo` char(128) NOT NULL DEFAULT '' COMMENT '图片logo大图',
  `goods_small_logo` char(128) NOT NULL DEFAULT '' COMMENT '图片logo小图',
  `is_del` enum('0','1') NOT NULL DEFAULT '0' COMMENT '0:正常  1:删除',
  `add_time` int(11) NOT NULL COMMENT '添加商品时间',
  `update_time` int(11) NOT NULL COMMENT '修改商品时间',
  `delete_time` int(11) DEFAULT NULL COMMENT '软删除标志字段',
  `cat_one_id` smallint(5) DEFAULT '0' COMMENT '一级分类id',
  `cat_two_id` smallint(5) DEFAULT '0' COMMENT '二级分类id',
  `cat_three_id` smallint(5) DEFAULT '0' COMMENT '三级分类id',
  `hot_mumber` int(11) unsigned DEFAULT '0' COMMENT '热卖数量',
  `is_promote` smallint(5) DEFAULT '0' COMMENT '是否促销',
  `goods_state` int(11) DEFAULT '0' COMMENT '商品状态 0: 未通过 1: 审核中 2: 已审核',
  PRIMARY KEY (`goods_id`) USING BTREE,
  UNIQUE KEY `goods_name` (`goods_name`) USING BTREE,
  KEY `goods_price` (`goods_price`) USING BTREE,
  KEY `add_time` (`add_time`) USING BTREE,
  KEY `goods_name_2` (`goods_name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=930 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='商品表';
```

10、建立关联表

```sql
CREATE TABLE `sp_order_goods` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` int(10) unsigned NOT NULL COMMENT '订单id',
  `goods_id` mediumint(8) unsigned NOT NULL COMMENT '商品id',
  `goods_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '商品单价',
  `goods_number` tinyint(4) NOT NULL DEFAULT '1' COMMENT '购买单个商品数量',
  `goods_total_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '商品小计价格',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `order_id` (`order_id`) USING BTREE,
  KEY `goods_id` (`goods_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=86 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='商品订单关联表';
```

11、建立订单表

```sql
CREATE TABLE `sp_order` (
  `order_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `user_id` mediumint(8) unsigned NOT NULL COMMENT '下订单会员id',
  `order_number` varchar(32) NOT NULL COMMENT '订单编号',
  `order_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '订单总金额',
  `order_pay` enum('0','1','2','3') NOT NULL DEFAULT '1' COMMENT '支付方式  0未支付 1支付宝  2微信  3银行卡',
  `is_send` enum('是','否') NOT NULL DEFAULT '否' COMMENT '订单是否已经发货',
  `trade_no` varchar(32) NOT NULL DEFAULT '' COMMENT '支付宝交易流水号码',
  `order_fapiao_title` enum('个人','公司') NOT NULL DEFAULT '个人' COMMENT '发票抬头 个人 公司',
  `order_fapiao_company` varchar(32) NOT NULL DEFAULT '' COMMENT '公司名称',
  `order_fapiao_content` varchar(32) NOT NULL DEFAULT '' COMMENT '发票内容',
  `consignee_addr` text NOT NULL COMMENT 'consignee收货人地址',
  `pay_status` enum('0','1') NOT NULL DEFAULT '0' COMMENT '订单状态： 0未付款、1已付款',
  `create_time` int(10) unsigned NOT NULL COMMENT '记录生成时间',
  `update_time` int(10) unsigned NOT NULL COMMENT '记录修改时间',
  PRIMARY KEY (`order_id`) USING BTREE,
  UNIQUE KEY `order_number` (`order_number`) USING BTREE,
  KEY `add_time` (`create_time`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=69 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='订单表';
```

12、建立权限表

```sql
CREATE TABLE `sp_permission` (
  `ps_id` smallint(6) unsigned NOT NULL AUTO_INCREMENT,
  `ps_name` varchar(20) NOT NULL COMMENT '权限名称',
  `ps_pid` smallint(6) unsigned NOT NULL COMMENT '父id',
  `ps_c` varchar(32) NOT NULL DEFAULT '' COMMENT '控制器',
  `ps_a` varchar(32) NOT NULL DEFAULT '' COMMENT '操作方法',
  `ps_level` enum('0','2','1') NOT NULL DEFAULT '0' COMMENT '权限等级',
  `ps_icon` char(30) NOT NULL DEFAULT 'el-icon-menu',
  PRIMARY KEY (`ps_id`) USING BTREE,
  UNIQUE KEY `id_index` (`ps_id`) USING BTREE,
  KEY `pid_index` (`ps_pid`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=160 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='权限表';
```

13、建立类型表

```sql
CREATE TABLE `sp_type` (
  `type_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `type_name` varchar(32) NOT NULL COMMENT '类型名称',
  `delete_time` int(11) DEFAULT NULL COMMENT '删除时间标志',
  PRIMARY KEY (`type_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='类型表';
```

14、建立会员表

```sql
CREATE TABLE `sp_user` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `username` varchar(128) NOT NULL DEFAULT '' COMMENT '登录名',
  `qq_open_id` char(32) DEFAULT NULL COMMENT 'qq官方唯一编号信息',
  `password` char(64) NOT NULL DEFAULT '' COMMENT '登录密码',
  `user_email` varchar(64) NOT NULL DEFAULT '' COMMENT '邮箱',
  `user_email_code` char(13) DEFAULT NULL COMMENT '新用户注册邮件激活唯一校验码',
  `is_active` enum('是','否') DEFAULT '否' COMMENT '新用户是否已经通过邮箱激活帐号',
  `user_sex` enum('保密','女','男') NOT NULL DEFAULT '男' COMMENT '性别',
  `user_qq` varchar(32) NOT NULL DEFAULT '' COMMENT 'qq',
  `user_tel` varchar(32) NOT NULL DEFAULT '' COMMENT '手机',
  `user_xueli` enum('博士','硕士','本科','专科','高中','初中','小学') NOT NULL DEFAULT '本科' COMMENT '学历',
  `user_hobby` varchar(32) NOT NULL DEFAULT '' COMMENT '爱好',
  `user_introduce` text COMMENT '简介',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  `user_eamil` varchar(255) NOT NULL DEFAULT '',
  `user_eamil_code` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`user_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='会员表';
```

以下是相应的表中的约束条件的建立：

```sql
ALTER TABLE
    `product_desc` CHANGE `id` `id` bigint(20) NOT NULL AUTO_INCREMENT;
ALTER TABLE `sp_attribute` 
	CHANGE `attr_id` `attr_id` smallint(5) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键id' ;
ALTER TABLE `sp_category` 
	CHANGE `cat_id` `cat_id` int(32) NOT NULL AUTO_INCREMENT COMMENT '分类唯一ID' ;
ALTER TABLE `sp_consignee` 
	CHANGE `cgn_id` `cgn_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键id' ;
ALTER TABLE `sp_express` 
	CHANGE `express_id` `express_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键id' ;
ALTER TABLE `sp_goods` 
	CHANGE `goods_id` `goods_id` mediumint(8) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键id' ;
ALTER TABLE `sp_permission` 
	CHANGE `ps_id` `ps_id` smallint(6) UNSIGNED NOT NULL AUTO_INCREMENT ;
ALTER TABLE `sp_permission_api` 
	CHANGE `id` `id` int(11) NOT NULL AUTO_INCREMENT ;
ALTER TABLE `sp_permission_api` 
	CHANGE `ps_id` `ps_id` int(11) NOT NULL ;
ALTER TABLE `sp_role` 
	CHANGE `role_name` `role_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色名称' ;
ALTER TABLE `sp_role` 
	CHANGE `ps_ids` `ps_ids` varchar(512) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '权限ids,1,2,5' ;
ALTER TABLE `sp_user` 
	CHANGE `user_id` `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增id' ;5.1.3；
```

**建立视图、索引**
视图的建立：
（1）商品信息视图:

```sql
CREATE VIEW 商品信息视图 AS
SELECT 商品表.*, 商品描述表.product_desc, 商品类别表.name AS 类别名称
FROM 商品表
JOIN 商品描述表 ON 商品表.product_id = 商品描述表.product_id
JOIN 商品类别表 ON 商品表.category_id = 商品类别表.id;
```

（2）库存信息视图:

```sql
CREATE VIEW 库存信息视图 AS
SELECT 商品表.product_id, 商品表.product_name, 库存表.quantity
FROM 商品表
JOIN 库存表 ON 商品表.product_id = 库存表.product_id;
```

（3）销售统计视图:

```sql
CREATE VIEW 销售统计视图 AS
SELECT 商品表.product_id, 商品表.product_name, SUM(订单详情表.quantity) AS 销售数量
FROM 订单表
JOIN 订单详情表 ON 订单表.order_id = 订单详情表.order_id
JOIN 商品表 ON 订单详情表.product_id = 商品表.product_id
GROUP BY 商品表.product_id, 商品表.product_name;
```

（4）购物行为视图：

```sql
CREATE VIEW 用户购买历史视图 AS
SELECT 用户表.user_id, 用户表.username, 订单表.order_id, 订单详情表.product_id, 订单详情表.quantity, 订单表.order_date
FROM 用户表
JOIN 订单表 ON 用户表.user_id = 订单表.user_id
JOIN 订单详情表 ON 订单表.order_id = 订单详情表.order_id;
```

索引建立：

```sql
ALTER TABLE products DROP INDEX `PRIMARY`;
ALTER TABLE products ADD key (`id`)
ALTER TABLE sp_attribute DROP INDEX `PRIMARY`;
ALTER TABLE sp_attribute ADD key (`attr_id`)
ALTER TABLE sp_category DROP INDEX `PRIMARY`;
ALTER TABLE sp_category ADD key (`cat_id`)
ALTER TABLE sp_permission DROP INDEX `PRIMARY`;
ALTER TABLE sp_permission ADD key (`ps_id`)
ALTER TABLE sp_permission DROP INDEX `id_index`;
ALTER TABLE sp_permission ADD key (`ps_id`)；
```

#### 5.1.4建立存储过程
1.检索商品信息存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE GetProductInfo(IN productID INT)
BEGIN
    SELECT product_id, product_name, price
    FROM products
    WHERE product_id = productID;
END $$
DELIMITER ;
```

2.添加新商品存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE AddNewProduct(IN productName VARCHAR(255), IN productPrice DECIMAL(10, 2))
BEGIN
    INSERT INTO products (product_name, price)
    VALUES (productName, productPrice);
END $$
DELIMITER ;
```

3.更新商品价格存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE UpdateProductPrice(IN productID INT, IN newPrice DECIMAL(10, 2))
BEGIN
    UPDATE products
    SET price = newPrice
    WHERE product_id = productID;
END $$
DELIMITER ;
```

4.删除商品存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE DeleteProduct(IN productID INT)
BEGIN
    DELETE FROM products
    WHERE product_id = productID;
END $$
DELIMITER ;
```

5.按类别检索商品存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE GetProductsByCategory(IN categoryName VARCHAR(255))
BEGIN
    SELECT product_id, product_name, price
    FROM products
    WHERE category = categoryName;
END $$
DELIMITER ;
```

6.商品销量统计存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE GetProductSales(IN productID INT, OUT totalSales INT)
BEGIN
    SELECT SUM(quantity) INTO totalSales
    FROM order_details
    WHERE product_id = productID;
END $$
DELIMITER ;
```

7.库存不足提醒存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE LowStockAlert(IN thresholdQuantity INT)
BEGIN
    SELECT product_id, product_name, quantity
    FROM products
    WHERE quantity &lt; thresholdQuantity;
END $$
DELIMITER ;
```

8.更新商品库存存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE UpdateStock(IN productID INT, IN newStockQuantity INT)
BEGIN
    UPDATE products
    SET quantity = newStockQuantity
    WHERE product_id = productID;
END $$
DELIMITER ;
```

9.按价格范围检索商品存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE GetProductsByPriceRange(IN minPrice DECIMAL(10, 2), IN maxPrice DECIMAL(10, 2))
BEGIN
    SELECT product_id, product_name, price
    FROM products
    WHERE price BETWEEN minPrice AND maxPrice;
END $$
DELIMITER ;
```

10.商品销售额统计存储过程：

```sql
DELIMITER $$
CREATE PROCEDURE TotalSalesAmount(OUT totalAmount DECIMAL(15, 2))
BEGIN
    SELECT SUM(p.price * od.quantity) INTO totalAmount
    FROM products p
    JOIN order_details od ON p.product_id = od.product_id;
END $$
DELIMITER ;
```

以上就是部分图表的存储过程，其他都跟其类似，因此就不必重复了。
### 5.2数据入库
商品数据库数据量一般都比较大，因此，一般采用导入数据的方法录入。这样既简单又节省了很多时间，提高了效率，基本的数据包括商品编号，商品名称，商品描述，商品价格，商品类别，供应商，生产商，库存数量，上架日期，商品图片，重量，尺寸，生产日期，保质期等等。
### 5.3创建功能存储
**用户管理的基本功能实现**
编号	存取过程名	作用
1	搜索内容	搜索用户的信息
2	添加用户	添加用户
3	修改用户信息	修改用户的邮箱或电话
4	更改用户状态	确保用户账号状态正常
5	删除用户	删除
6	分配用户权限	分配权限

**权限管理的基本功能实现**
编号	存取过程名	作用
1	添加权限	添加
2	编辑权限	修改
3	删除权限	删除
4	分配权限	分配权限
5	批量导出	可多选导出
6	批量删除	可多选删除
7	修改权限	修改
8	搜索内容	查看搜索的人的权限

**商品管理的基本功能实现**
编号	存取过程名	作用
1	添加商品	添加
2	搜索商品	查看自己搜索的相关的商品
3	修改商品	修改内容
4	删除商品	删除，下架
5	添加商品分类	添加新的类型
6	添加参数类型	参数
7	修改参数	修改
8	删除参数	删除

**订单管理的基本功能实现**
编号	存取过程名	作用
1	搜索订单	查看订单信息
2	查看物流	查看当前物流信息


## 5.UI设计（仅供参考）
此部分截图过多，不在此展示，如果有需要可以关注我的公众号，获取报告+源码地址，请看文章末尾。

## 6.总结
通过本次实习，我不仅对数据库理论知识有了更深一层的认识，对数据库的创建过程更加透彻的了解。我越来越感觉到基础的重要性，这不仅来源于我在第一阶段的辛苦，更加体会深刻的是我在后面的实习中，不断的发现不足，不断的更改前一阶段的相关内容。但是鉴于时间的原因，设计过程中有的问题没有深入研究，考虑全面，不可避免的出现了一些问题，这也是有待改进的，也是情有可原的。知识的重新学习只是本次实习的一小方面，更重要的是让我学会了很多书本上学不到的东西，比如自己学习，自己设计，自己调查研究，从各种渠道获取有用知识的能力，自主创新，自主完成课题，自主设计，这也许就是本次实习的最终目的吧。
## 参考文献

[1] 萨师煊 王珊，数据库系统概论(第三版)，北京:高教出版社，2000
[2] 郑人杰，实用软件工程(第二版)，北京:清华大学出版社，2003
[3] 许柯，2006级数据库课程设计论文

## 附件
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182218925.png)
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182218886.png)
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182218059.png)

## 报告+源码获取地址

详细内容请关注微信公众号：**GolangCode**，输入“**课程设计报告**” 获取详细内容。如果对你有小小的帮助，也请给我点个小赞赞。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)