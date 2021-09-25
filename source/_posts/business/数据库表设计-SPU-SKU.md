---
title: 数据库表设计-SPU/SKU
date: 2021-09-18 14:27:33
tags:
categories:
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [SKU 和 SPU 有什么区别？](https://www.zhihu.com/question/29073730)

#### 名词解释

##### SPU

* Standard Product Unit (标准产品单位);
* 商品**信息聚合**的最小单位,是一组**可复用,易检索的标准化信息集合**,该集合描述了一个产品的特性;
* 属性值,特性相同的商品就可以称为一个SPU;
* SPU 是由品牌+型号+关键属性构成的

> **信息聚合：**意味着有识别度的信息被用来作为不同SPU的区分点；不是所有属性，使用的属性值是能够有区分度的关键属性值；
>
> **易检索：**信息聚合与易检索这两个说明，是通过关键属性+属性值的聚合来实现易检索这个目的；目的与使用场景相关联，并非万古不变；哪些属性和属性值会被选为区分SPU的关键属性是会随着场景变化的； 但对于一些场景，已经是共识；比如电商销售，对于标品基本都会选择品牌+型号+关键属性；这也是很多年前我所得到的解释。
>
> **标准化的信息集合：**说明SPU的本质是信息集合，是一个**抽象概念**，并非是看得见的东西。比如格力空调 KFR-25GW/E;

##### SKU

* Stock Keeping Unit 库存量单位;
* 库存进出计量的单位,可以是以件,盒,托盘等为单位;
* 物理上不可分割的最小存货单元.

#### 表设计

##### 商品分类表

| 字段名       | 类型      | 长度 | 限制               | 备注/说明        |
| ------------ | --------- | ---- | ------------------ | ---------------- |
| id           | int       | 10   | 主键               | 商品分类ID       |
| flag         | varchar   | 50   | 唯一/非空          | 分类唯一标识     |
| name         | varchar   | 100  | 唯一/非空          | 分类中文名称     |
| parent_id    | int       | 10   | 非空/顶级父分类为0 | 父分类id         |
| sort         | int       | 5    | 非空/默认为0       | 排序字段         |
| created_time | timestamp |      | 非空               | 创建时间         |
| updated_time | timestamp |      | 非空               | 最近一次更新时间 |
| delete_flag  | tinyint   | 2    | 非空/默认为0       | 逻辑删除字段     |

##### 商品表

| 字段名       | 类型      | 长度 | 限制         | 备注/说明              |
| ------------ | --------- | ---- | ------------ | ---------------------- |
| id           | int       | 10   | 主键         | 商品ID                 |
| category_id  | int       | 10   | 非空         | 商品所属分类ID         |
| name         | varchar   | 200  | 唯一/非空    | 商品名称               |
| status       | tinyint   | 2    | 非空         | 商品状态 1-上架,2-下架 |
| sell_num     | int       | 10   | 非空/默认为0 | 销量                   |
| created_time | timestamp |      | 非空         | 创建时间               |
| updated_time | timestamp |      | 非空         | 最近一次更新时间       |
| delete_flag  | tinyint   | 2    | 非空/默认为0 | 逻辑删除字段           |

#####  商品规格表

| 字段名       | 类型      | 长度 | 限制         | 备注/说明             |
| ------------ | --------- | ---- | ------------ | --------------------- |
| id           | int       | 10   | 主键         | 规格ID                |
| price        | DECIMAL   | 10,2 | 非空         | 售价                  |
| stock        | int       | 10   | 非空         | 库存                  |
| status       | tinyint   | 1    | 非空         | sku状态 1-上架;2-下架 |
| created_time | timestamp |      | 非空         | 创建时间              |
| updated_time | timestamp |      | 非空         | 最近一次更新时间      |
| delete_flag  | tinyint   | 2    | 非空/默认为0 | 逻辑删除字段          |

##### 商品规格属性表

| 字段名       | 类型      | 长度 | 限制         | 备注/说明        |
| ------------ | --------- | ---- | ------------ | ---------------- |
| id           | Int       | 10   | 主键         | 属性ID           |
| name         | varchar   | 50   | 唯一/非空    | 属性名称         |
| flag         | varchar   | 20   | 唯一/非空    | 属性英文标识     |
| sort         | int       | 5    | 非空/默认为0 | 属性排序字段     |
| created_time | timestamp |      | 非空         | 创建时间         |
| updated_time | timestamp |      | 非空         | 最近一次更新时间 |
| delete_flag  | tinyint   | 2    | 非空/默认为0 | 逻辑删除字段     |

##### 商品规格属性值表

| 字段名       | 类型      | 长度 | 限制         | 备注/说明        |
| ------------ | --------- | ---- | ------------ | ---------------- |
| id           | Int       | 10   | 主键         | 属性值ID         |
| attr_id      | int       | 10   | 非空         | 属性ID           |
| sort         | int       | 5    | 非空         | 属性值排序字段   |
| value        | varchar   | 200  | 非空         | 属性值           |
| type         | Varchar   | 50   | 非空         | 属性值类型       |
| created_time | timestamp |      | 非空         | 创建时间         |
| updated_time | timestamp |      | 非空         | 最近一次更新时间 |
| delete_flag  | tinyint   | 2    | 非空/默认为0 | 逻辑删除字段     |

##### 商品规格属性组-属性关系表

| 字段名        | 类型      | 长度 | 限制         | 备注/说明        |
| ------------- | --------- | ---- | ------------ | ---------------- |
| id            | int       | 10   | 主键         | 关系ID           |
| sku_id        | int       | 10   | 非空         | skuID            |
| attr_id       | int       | 10   | 非空         | 属性ID           |
| attr_value_id | int       | 10   | 非空         | 属性值ID         |
| created_time  | timestamp |      | 非空         | 创建时间         |
| updated_time  | timestamp |      | 非空         | 最近一次更新时间 |
| delete_flag   | tinyint   | 2    | 非空/默认为0 | 逻辑删除字段     |

#### SQL

```mysql
DROP TABLE
IF
	EXISTS `category`;
CREATE TABLE `category` (
	`id` INT ( 10 ) auto_increment COMMENT '商品分类ID',
	`flag` VARCHAR ( 50 ) UNIQUE NOT NULL COMMENT '分类唯一标识',
	`name` VARCHAR ( 100 ) UNIQUE NOT NULL COMMENT '分类中文名称',
	`parent_id` INT ( 10 ) NOT NULL DEFAULT 0 COMMENT '父分类id',
	`sort` INT ( 5 ) NOT NULL DEFAULT 0 COMMENT '排序字段',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
	PRIMARY KEY ( `id` ) 
) COMMENT '商品分类表';

DROP TABLE
IF
	EXISTS `product`;
CREATE TABLE `product` (
	`id` INT ( 10 ) auto_increment COMMENT '商品ID',
	`category_id` INT ( 10 ) NOT NULL COMMENT '商品所属分类ID',
	`name` VARCHAR ( 200 ) UNIQUE NOT NULL COMMENT '商品名称',
	`status` TINYINT ( 2 ) NOT NULL COMMENT '商品状态 1-上架,2-下架',
	`sell_number` INT ( 20 ) NOT NULL DEFAULT 0 COMMENT '销量',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
	PRIMARY KEY ( `id` ) 
) COMMENT '商品表';

DROP TABLE
IF
	EXISTS `product_specification`;
CREATE TABLE `product_specification` (
	`id` INT ( 10 ) auto_increment COMMENT '规格ID',
	`product_id` INT ( 10 ) NOT NULL COMMENT '所属产品ID',
	`price` DECIMAL ( 10, 2 ) NOT NULL COMMENT '售价',
	`status` TINYINT ( 2 ) NOT NULL COMMENT 'SKU状态 1-上架,2-下架',
	`stock` INT ( 10 ) NOT NULL COMMENT '库存',
	`attr_group_id` INT ( 10 ) NOT NULL COMMENT '属性组ID',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
	PRIMARY KEY ( `id` ) 
) COMMENT ' 商品规格表';

DROP TABLE
IF
	EXISTS `specification_attribute`;
CREATE TABLE `specification_attribute` (
	`id` INT ( 10 ) auto_increment COMMENT '属性ID',
	`name` VARCHAR ( 50 ) NOT NULL UNIQUE COMMENT '属性名称',
	`flag` VARCHAR ( 20 ) NOT NULL UNIQUE COMMENT '属性英文标识',
	`sort` INT ( 5 ) NOT NULL DEFAULT 0 COMMENT '属性排序字段',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
	PRIMARY KEY ( `id` ) 
) COMMENT '商品规格属性表';

DROP TABLE
IF
	EXISTS `specification_attribute_value`;
CREATE TABLE `specification_attribute_value` (
	`id` INT ( 10 ) auto_increment COMMENT '属性值ID',
	`attr_id` INT ( 10 ) NOT NULL COMMENT '属性ID',
	`value` VARCHAR ( 200 ) NOT NULL COMMENT '属性值',
	`type` VARCHAR ( 50 ) NOT NULL COMMENT '属性值类型',
	`sort` INT ( 5 ) NOT NULL DEFAULT 0 COMMENT '属性排序字段',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
	PRIMARY KEY ( `id` ) 
) COMMENT '商品规格属性值表';

DROP TABLE
IF
	EXISTS `specification_attribute_relation`;
CREATE TABLE `specification_attribute_relation` (
	`id` INT ( 10 ) auto_increment,
	`sku_id` INT ( 10 ) NOT NULL COMMENT 'skuID',
	`attr_id` INT ( 10 ) NOT NULL COMMENT '属性ID',
	`attr_value_id` INT ( 10 ) NOT NULL COMMENT '属性值ID',
	`created_time` TIMESTAMP COMMENT '创建时间',
	`updated_time` TIMESTAMP COMMENT '最近一次更新时间',
	`delete_flag` TINYINT ( 2 ) NOT NULL DEFAULT 0 COMMENT '逻辑删除字段',
PRIMARY KEY ( `id` ) 
) COMMENT '商品规格属性组-属性关系表';
```

