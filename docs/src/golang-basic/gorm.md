---
title: gorm快速入门（只需一篇即可学会）
shortTitle: 14.Golang的orm框架-gorm
description: GORM 是 Go 语言中一个非常流行的 ORM（对象关系映射）库，它提供了一种简单而强大的方式来将 Go 结构体映射到数据库表，并提供了丰富的方法来操作数据库，比如查询、插入、更新和删除数据等。
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-23
---

:::tip

- gorm中文网址：[https://gorm.io/zh_CN/docs/index.html](https://gorm.io/zh_CN/docs/index.html)



:::

## 一、什么是GORM？

GORM 是 Go 语言中一个非常流行的 ORM（对象关系映射）库，它提供了一种简单而强大的方式来将 Go 结构体映射到数据库表，并提供了丰富的方法来操作数据库，比如查询、插入、更新和删除数据等。

## 二、GORM连接MySQL以及AutoMigrate创建表

#### 安装 GORM 和 MySQL 驱动

```sh
go get "gorm.io/driver/mysql"
go get "gorm.io/gorm"
```

#### 数据库连接

```go
import (
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
)

var db *gorm.DB
var err error

func initDB() {
	dsn := "root:123456@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
	db, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
		SkipDefaultTransaction: true,
		DisableForeignKeyConstraintWhenMigrating: true,
		NamingStrategy: schema.NamingStrategy{
			SingularTable: true,
		},
	})
	if err != nil {
		// handle error
	}
}
```

#### AutoMigrate 自动创建表

```go
type User struct {
	gorm.Model
	Name string
	Age  int
}

func autoMigrate() {
	db.AutoMigrate(&User{})
}
```

## 三、查询

#### 1. 检索单个对象（First、Take、Last）

```go
var user User
result := db.Where("name = ?", "张三").First(&user)
if result.Error != nil {
	if errors.Is(result.Error, gorm.ErrRecordNotFound) {
		// handle not found
	}
	// handle other errors
}
```

#### 2. 使用 Find() 方法检索多个对象

```go
var users []User
result := db.Where("age = ?", 20).Find(&users)
if result.Error != nil {
	// handle error
}
```

#### 3. 根据指定字段查询

```go
var users []User
result := db.Select("name", "age").Where("age = ?", 20).Find(&users)
if result.Error != nil {
	// handle error
}
```

#### 4. 使用 Raw SQL 进行查询

```go
var user User
db.Raw("SELECT * FROM users WHERE id = ?", 1).Scan(&user)
```

#### 5. 使用 Joins 进行联表查询

```go
type User struct {
	ID       uint
	Name     string
	Age      int
	Orders   []Order
}

type Order struct {
	ID       uint
	OrderNum string
	UserID   uint
}

func getUsersWithOrders() {
	var users []User
	db.Preload("Orders").Find(&users)
}
```

## 四、更新

#### 1. Save() 方法保存多个字段

```go
user := User{Name: "张三", Age: 18}
result := db.Save(&user)
if result.Error != nil {
	// handle error
}
```

#### 2. 更新单个字段

```go
result := db.Model(&User{}).Where("name = ?", "张三").Updates(map[string]interface{}{"age": 30, "name": "李四"})
if result.Error != nil {
	// handle error
}
```

#### 3. 使用 SQL 表达式更新

```go
db.Model(&User{}).Where("age > ?", 20).UpdateColumn("age", gorm.Expr("age + ?", 2))
```

## 五、删除

#### 软删除

```go
var user User
result := db.Where("age < ?", 98).Delete(&user)
if result.Error != nil {
	// handle error
}
```

#### 物理删除

```go
db.Unscoped().Delete(&User{}, 5, 6)
if db.Error != nil {
	// handle error
}
```

## 六、关联

#### 一对一关联

```go
type User struct {
	gorm.Model
	Name string
	Age  int
	CreditCard CreditCard
}

type CreditCard struct {
	gorm.Model
	Number string
	UserID uint
}

func autoMigrate() {
	db.AutoMigrate(&User{}, &CreditCard{})
}
```

#### 一对多关联

```go
type User struct {
	gorm.Model
	Name string
	Age  int
	Orders []Order
}

type Order struct {
	gorm.Model
	OrderNum string
	UserID uint
}

func autoMigrate() {
	db.AutoMigrate(&User{}, &Order{})
}
```

#### 多对多关联

```go
type User struct {
	gorm.Model
	Name string
	Age  int
	Roles []Role `gorm:"many2many:user_roles;"`
}

type Role struct {
	gorm.Model
	Name string
	Users []User `gorm:"many2many:user_roles;"`
}

func autoMigrate() {
	db.AutoMigrate(&User{}, &Role{})
}
```

## 七、迁移

GORM 提供了自动迁移功能，可以根据模型自动创建、更新表结构。

```go
func autoMigrate() {
	err := db.AutoMigrate(&User{}, &Order{}, &CreditCard{})
	if err != nil {
		// handle error
	}
}
```

## 八、事务

GORM 支持事务操作，可以使用 `Transaction` 方法来处理事务。

```go
func createUserWithTransaction() {
	err := db.Transaction(func(tx *gorm.DB) error {
		if err := tx.Create(&User{Name: "张三", Age: 18}).Error; err != nil {
			return err
		}
		if err := tx.Create(&Order{OrderNum: "123456", UserID: 1}).Error; err != nil {
			return err
		}
		return nil
	})
	if err != nil {
		// handle error
	}
}
```

## 九、钩子

GORM 提供了钩子函数，在创建、更新、删除等操作前后执行特定的逻辑。

```go
type User struct {
	gorm.Model
	Name string
	Age  int
}

func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
	if u.Name == "" {
		err = errors.New("name can't be empty")
	}
	return
}
```

## 十、批量操作

GORM 支持批量插入、更新、删除操作。

#### 批量插入

```go
users := []User{
	{Name: "张三", Age: 18},
	{Name: "李四", Age: 20},
}
db.Create(&users)
```

#### 批量更新

```go
db.Model(&User{}).Where("age > ?", 20).Updates(map[string]interface{}{"age": 30})
```

#### 批量删除

```go
db.Where("age > ?", 20).Delete(&User{})
```

