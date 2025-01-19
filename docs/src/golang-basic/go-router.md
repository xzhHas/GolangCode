---
title: 如何制作一个后台管理页面的路由以及功能实现
shortTitle: 18.用Go制作一个后台管理路由
description: 如何制作一个后台管理页面的路由以及功能实现
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-09
---

:::tip

- 原文链接：[https://blog.csdn.net/m0_73337964/article/details/138321631](https://blog.csdn.net/m0_73337964/article/details/138321631)
:::
## 一、RESFUL API

1、分类模块

```go
	// 分类模块
	category := auth.Group("/category")
	{
		category.GET("/list", categoryAPI.GetList)     // 分类列表
		category.POST("", categoryAPI.SaveOrUpdate)    // 新增/编辑分类
		category.DELETE("", categoryAPI.Delete)        // 删除分类
		category.GET("/option", categoryAPI.GetOption) // 分类选项列表
	}
```

2、评论模块

```go
	// 评论模块
	comment := auth.Group("/comment")
	{
		comment.GET("/list", commentAPI.GetList)        // 评论列表
		comment.DELETE("", commentAPI.Delete)           // 删除评论
		comment.PUT("/review", commentAPI.UpdateReview) // 修改评论审核
	}
```

3、留言模块

```go
	// 留言模块
	message := auth.Group("/message")
	{
		message.GET("/list", messageAPI.GetList)        // 留言列表
		message.DELETE("", messageAPI.Delete)           // 删除留言
		message.PUT("/review", messageAPI.UpdateReview) // 审核留言
	}
```



4、博客设置



```go
	// 博客设置
	setting := auth.Group("/setting")
	{
		setting.GET("/about", blogInfoAPI.GetAbout)    // 获取关于我
		setting.PUT("/about", blogInfoAPI.UpdateAbout) // 编辑关于我
	}
```

5、标签模块

```go
	// 标签模块
	tag := auth.Group("/tag")
	{
		tag.GET("/list", tagAPI.GetList)     // 标签列表
		tag.POST("", tagAPI.SaveOrUpdate)    // 新增/编辑标签
		tag.DELETE("", tagAPI.Delete)        // 删除标签
		tag.GET("/option", tagAPI.GetOption) // 标签选项列表
	}
```

6、友情链接

```go
	// 友情链接
	link := auth.Group("/link")
	{
		link.GET("/list", linkAPI.GetList)  // 友链列表
		link.POST("", linkAPI.SaveOrUpdate) // 新增/编辑友链
		link.DELETE("", linkAPI.Delete)     // 删除友链
	}
```

7、资源模块

```go
	// 资源模块
	resource := auth.Group("/resource")
	{
		resource.GET("/list", resourceAPI.GetTreeList)          // 资源列表(树形)
		resource.POST("", resourceAPI.SaveOrUpdate)             // 新增/编辑资源
		resource.DELETE("/:id", resourceAPI.Delete)             // 删除资源
		resource.PUT("/anonymous", resourceAPI.UpdateAnonymous) // 修改资源匿名访问
		resource.GET("/option", resourceAPI.GetOption)          // 资源选项列表(树形)
	}
```

8、菜单模块

```go
	// 菜单模块
	menu := auth.Group("/menu")
	{
		menu.GET("/list", menuAPI.GetTreeList)      // 菜单列表
		menu.POST("", menuAPI.SaveOrUpdate)         // 新增/编辑菜单
		menu.DELETE("/:id", menuAPI.Delete)         // 删除菜单
		menu.GET("/user/list", menuAPI.GetUserMenu) // 获取当前用户的菜单
		menu.GET("/option", menuAPI.GetOption)      // 菜单选项列表(树形)
	}
```

9、角色模块

```go
	// 角色模块
	role := auth.Group("/role")
	{
		role.GET("/list", roleAPI.GetTreeList) // 角色列表(树形)
		role.POST("", roleAPI.SaveOrUpdate)    // 新增/编辑菜单
		role.DELETE("", roleAPI.Delete)        // 删除角色
		role.GET("/option", roleAPI.GetOption) // 角色选项列表(树形)
	}
```

10、操作日志模块

```go
	// 操作日志模块
	operationLog := auth.Group("/operation/log")
	{
		operationLog.GET("/list", operationLogAPI.GetList) // 操作日志列表
		operationLog.DELETE("", operationLogAPI.Delete)    // 删除操作日志
	}
```


11、页面模块

```go
	// 页面模块
	page := auth.Group("/page")
	{
		page.GET("/list", pageAPI.GetList)  // 页面列表
		page.POST("", pageAPI.SaveOrUpdate) // 新增/编辑页面
		page.DELETE("", pageAPI.Delete)     // 删除页面
	}
```
12、用户模块

```go
	// 用户模块
	user := auth.Group("/user")
	{
		user.GET("/list", userAPI.GetList)          // 用户列表
		user.PUT("", userAPI.Update)                // 修改用户信息
		user.PUT("/disable", userAPI.UpdateDisable) // 修改用户禁用状态
		// user.PUT("/password", userAPI.UpdatePassword)                // 修改普通用户密码
		user.PUT("/current/password", userAPI.UpdateCurrentPassword) // 修改管理员密码
		user.GET("/info", userAPI.GetInfo)                           // 获取当前用户信息
		user.PUT("/current", userAPI.UpdateCurrent)                  // 修改当前用户信息
		user.GET("/online", userAPI.GetOnlineList)                   // 获取在线用户
		user.POST("/offline/:id", userAPI.ForceOffline)              // 强制用户下线
	}
```
13、文章模块

```go
	auth.GET("/home", blogInfoAPI.GetHomeInfo) // 后台首页信息
	auth.POST("/upload", uploadAPI.UploadFile) // 文件上传
	
	// 文章模块
	articles := auth.Group("/article")
	{
		articles.GET("/list", articleAPI.GetList)                 // 文章列表
		articles.POST("", articleAPI.SaveOrUpdate)                // 新增/编辑文章
		articles.PUT("/top", articleAPI.UpdateTop)                // 更新文章置顶
		articles.GET("/:id", articleAPI.GetDetail)                // 文章详情
		articles.PUT("/soft-delete", articleAPI.UpdateSoftDelete) // 软删除文章
		articles.DELETE("", articleAPI.Delete)                    // 物理删除文章
		articles.POST("/export", articleAPI.Export)               // 导出文章
		articles.POST("/import", articleAPI.Import)               // 导入文章
	}
```

## 二、各模块路由处理

### 1、分类模块

#### 1.1、GET  /list  分类列表

```go
// 获取分类列表
func (*Category) GetList(c *gin.Context) {
	var query PageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	data, total, err := model.GetCategoryList(GetDB(c), query.Page, query.Size, query.Keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.CategoryVO]{
		Total: total,
		List:  data,
		Size:  query.Size,
		Page:  query.Page,
	})
}

// gorm 数据库查询
func GetCategoryList(db *gorm.DB, num, size int, keyword string) ([]CategoryVO, int64, error) {
	var list = make([]CategoryVO, 0)
	var total int64

	db = db.Table("category c").
		Select("c.id", "c.name", "COUNT(a.id) AS article_count", "c.created_at", "c.updated_at").
		Joins("LEFT JOIN article a ON c.id = a.category_id AND a.is_delete = 0 AND a.status = 1")

	if keyword != "" {
		db = db.Where("name LIKE ?", "%"+keyword+"%")
	}

	result := db.Group("c.id").
		Order("c.updated_at DESC").
		Count(&total).
		Scopes(Paginate(num, size)).
		Find(&list)

	return list, total, result.Error
}
```



#### 1.2、POST  /   新增|编辑分类

```go
// 添加或修改分类
func (*Category) SaveOrUpdate(c *gin.Context) {
    var req AddOrEditCategoryReq
    if err := c.ShouldBindJSON(&req); err != nil {
       ReturnError(c, g.ErrRequest, err)
       return
    }

    category, err := model.SaveOrUpdateCategory(GetDB(c), req.ID, req.Name)
    if err != nil {
       ReturnError(c, g.ErrDbOp, err)
       return
    }

    ReturnSuccess(c, category)
}

// 
func SaveOrUpdateCategory(db *gorm.DB, id int, name string) (*Category, error) {
	category := Category{
		Model: Model{ID: id},
		Name:  name,
	}

	var result *gorm.DB
	if id > 0 {
		result = db.Updates(category)
	} else {
		result = db.Create(&category)
	}

	return &category, result.Error
}
```

#### 1.3、DELETE  /  删除分类

```go
func (*Category) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)

	// 检查分类下是否存在文章
	count, err := model.Count(db, &model.Article{}, "category_id in ?", ids)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	if count > 0 {
		ReturnError(c, g.ErrCateHasArt, nil)
		return
	}

	rows, err := model.DeleteCategory(db, ids)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}
	ReturnSuccess(c, rows)
}

func DeleteCategory(db *gorm.DB, ids []int) (int64, error) {
	result := db.Where("id IN ?", ids).Delete(Category{})
	if result.Error != nil {
		return 0, result.Error
	}
	return result.RowsAffected, nil
}
```



#### 1.4、GET   /option   分类选项列表

 

```go
func (*Category) GetOption(c *gin.Context) {
	list, err := model.GetCategoryOption(GetDB(c))
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}
	ReturnSuccess(c, list)
}

func GetCategoryOption(db *gorm.DB) ([]OptionVO, error) {
	var list = make([]OptionVO, 0)
	result := db.Model(&Category{}).Select("id", "name").Find(&list)
	return list, result.Error
}
```



### 2、评论模块

#### 2.1、GET    /list   评论列表

```go
func (*Comment) GetList(c *gin.Context) {
	var query CommentQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}
	list, total, err := model.GetCommentList(GetDB(c), query.Page, query.Size, query.Type, query.IsReview, query.Nickname)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.Comment]{
		Total: total,
		List:  list,
		Size:  query.Size,
		Page:  query.Page,
	})
}

// 获取后台评论列表
func GetCommentList(db *gorm.DB, page, size, typ int, isReview *bool, nickname string) (data []Comment, total int64, err error) {
	if typ != 0 {
		db = db.Where("type = ?", typ)
	}
	if isReview != nil {
		db = db.Where("is_review = ?", *isReview)
	}
	if nickname != "" {
		db = db.Where("nickname LIKE ?", "%"+nickname+"%")
	}

	result := db.Model(&Comment{}).
		Count(&total).
		Preload("User").Preload("User.UserInfo").
		Preload("ReplyUser").Preload("ReplyUser.UserInfo").
		Preload("Article").
		Order("id DESC").
		Scopes(Paginate(page, size)).
		Find(&data)

	return data, total, result.Error
}
```



#### 2.2、DELETE    /   删除评论

```go
func (*Comment) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	result := GetDB(c).Delete(model.Comment{}, "id in ?", ids)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```



#### 2.3、PUT   /review   修改评论审核

```go
func (*Comment) UpdateReview(c *gin.Context) {
	var req UpdateReviewReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}
	maps := map[string]any{"is_review": req.IsReview}
	result := GetDB(c).Model(model.Comment{}).Where("id in ?", req.Ids).Updates(maps)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```



### 3、留言模块

#### 3.1、GET    /list   留言列表

```go
func (*Message) GetList(c *gin.Context) {
	var query MessageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	data, total, err := model.GetMessageList(GetDB(c), query.Page, query.Size, query.Nickname, query.IsReview)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}
	ReturnSuccess(c, PageResult[model.Message]{
		Total: total,
		List:  data,
		Size:  query.Size,
		Page:  query.Page,
	})
}

func GetMessageList(db *gorm.DB, num, size int, nickname string, isReview *bool) (list []Message, total int64, err error) {
	db = db.Model(&Message{})

	if nickname != "" {
		db = db.Where("nickname LIKE ?", "%"+nickname+"%")
	}

	if isReview != nil {
		db = db.Where("is_review = ?", isReview)
	}

	db.Count(&total)
	result := db.Order("created_at DESC").Scopes(Paginate(num, size)).Find(&list)
	return list, total, result.Error
}
```

#### 3.2、DELETE    /    删除留言

```go
func (*Message) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	rows, err := model.DeleteMessages(GetDB(c), ids)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, rows)
}

func DeleteMessages(db *gorm.DB, ids []int) (int64, error) {
	result := db.Where("id in ?", ids).Delete(&Message{})
	return result.RowsAffected, result.Error
}
```

#### 3.3、PUT      /review    审核留言

```go
func (*Message) UpdateReview(c *gin.Context) {
	var req UpdateReviewReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	rows, err := model.UpdateMessagesReview(GetDB(c), req.Ids, req.IsReview)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, rows)
}

func UpdateMessagesReview(db *gorm.DB, ids []int, isReview bool) (int64, error) {
	result := db.Model(&Message{}).Where("id in ?", ids).Update("is_review", isReview)
	return result.RowsAffected, result.Error
}
```

### 4、博客设置

#### 4.1、GET  /about   获取关于我的info

```go
func (*BlogInfo) GetAbout(c *gin.Context) {
	ReturnSuccess(c, model.GetConfig(GetDB(c), g2.CONFIG_ABOUT))
}

func GetConfig(db *gorm.DB, key string) string {
	var config Config
	result := db.Where("key", key).First(&config)

	if result.Error != nil {
		return ""
	}

	return config.Value
}
```



#### 4.2、PUT   /about   编辑关于我的info



```
func (*BlogInfo) UpdateAbout(c *gin.Context) {
	var req AboutReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	err := model.CheckConfig(GetDB(c), g2.CONFIG_ABOUT, req.Content)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, req.Content)
}


func CheckConfig(db *gorm.DB, key, value string) error {
	var config Config

	result := db.Where("key", key).FirstOrCreate(&config)
	if result.Error != nil {
		return result.Error
	}

	config.Value = value
	result = db.Save(&config)

	return result.Error
}
```



### 5、标签模块

#### 5.1、GET   /list    标签列表

```go
// 根据筛选条件 获取标签列表
func (*Tag) GetList(c *gin.Context) {
	var query PageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	data, total, err := model.GetTagList(GetDB(c), query.Page, query.Size, query.Keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.TagVO]{
		Total: total,
		List:  data,
		Size:  query.Size,
		Page:  query.Page,
	})
}


// 根据参数搜索
func GetTagList(db *gorm.DB, page, size int, keyword string) (list []TagVO, total int64, err error) {
	db = db.Table("tag t").
		Joins("LEFT JOIN article_tag at ON t.id = at.tag_id").
		Select("t.id", "t.name", "COUNT(at.article_id) AS article_count", "t.created_at", "t.updated_at")

	if keyword != "" {
		db = db.Where("name LIKE ?", "%"+keyword+"%")
	}

	result := db.
		Group("t.id").Order("t.updated_at DESC").
		Count(&total).
		Scopes(Paginate(page, size)).
		Find(&list)

	return list, total, result.Error
}
```



#### 5.2、POST  /  新增|修改标签

```go
func (*Tag) SaveOrUpdate(c *gin.Context) {
	var form AddOrEditTagReq
	if err := c.ShouldBindJSON(&form); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	tag, err := model.SaveOrUpdateTag(GetDB(c), form.ID, form.Name)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, tag)
}

func SaveOrUpdateTag(db *gorm.DB, id int, name string) (*Tag, error) {
	tag := Tag{
		Model: Model{ID: id},
		Name:  name,
	}

	var result *gorm.DB
	if id > 0 {
		result = db.Updates(tag)
	} else {
		result = db.Create(&tag)
	}

	return &tag, result.Error
}
```



#### 5.3、DELETE   /    删除标签

```go
// 添加强制删除，有关联数据将数据删除
func (*Tag) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}
	db := GetDB(c)

	// 检查标签下面有没有文章
	count, err := model.Count(db, &model.ArticleTag{}, "tag_id in ?", ids)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}
	if count > 0 {
		// ReturnError(c, g.ERROR_TAG_ART_EXIST, nil)
		ReturnError(c, g.ErrTagHasArt, nil)
		return
	}

	result := db.Delete(model.Tag{}, "id in ?", ids)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```



#### 5.4、GET     /option   标签选项列表



```go
func (*Tag) GetOption(c *gin.Context) {
	list, err := model.GetTagOption(GetDB(c))
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}
	ReturnSuccess(c, list)
}

func GetTagOption(db *gorm.DB) ([]OptionVO, error) {
	list := make([]OptionVO, 0)
	result := db.Model(&Tag{}).Select("id", "name").Find(&list)
	return list, result.Error
}
```



### 6、友情链接

#### 6.1、GET   /list     友链列表

```go
func (*Link) GetList(c *gin.Context) {
	var query PageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	data, total, err := model.GetLinkList(GetDB(c), query.Page, query.Size, query.Keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.FriendLink]{
		Total: total,
		List:  data,
		Size:  query.Size,
		Page:  query.Page,
	})
}

func GetLinkList(db *gorm.DB, num, size int, keyword string) (list []FriendLink, total int64, err error) {
	db = db.Model(&FriendLink{})
	if keyword != "" {
		db = db.Where("name LIKE ?", "%"+keyword+"%")
		db = db.Or("address LIKE ?", "%"+keyword+"%")
		db = db.Or("intro LIKE ?", "%"+keyword+"%")
	}
	db.Count(&total)
	result := db.Order("created_at DESC").
		Scopes(Paginate(num, size)).
		Find(&list)
	return list, total, result.Error
}
```

#### 6.2、POST    /     新增|修改

```go
func (*Link) SaveOrUpdate(c *gin.Context) {
	var req AddOrEditLinkReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	link, err := model.SaveOrUpdateLink(GetDB(c), req.ID, req.Name, req.Avatar, req.Address, req.Intro)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, link)
}

func SaveOrUpdateLink(db *gorm.DB, id int, name, avatar, address, intro string) (*FriendLink, error) {
	link := FriendLink{
		Model:   Model{ID: id},
		Name:    name,
		Avatar:  avatar,
		Address: address,
		Intro:   intro,
	}

	var result *gorm.DB
	if id > 0 {
		result = db.Updates(&link)
	} else {
		result = db.Create(&link)
	}

	return &link, result.Error
}
```

#### 6.3、DELETE    /     删除

```go
func (*Link) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	result := GetDB(c).Delete(&model.FriendLink{}, "id in ?", ids)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```



### 7、资源模块

#### 7.1、GET    /list   资源列表（树形）

```go
// 获取资源列表(树形)
func (*Resource) GetTreeList(c *gin.Context) {
	keyword := c.Query("keyword")

	resourceList, err := model.GetResourceList(GetDB(c), keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, resources2ResourceVos(resourceList))
}

func GetResourceList(db *gorm.DB, keyword string) (list []Resource, err error) {
	if keyword != "" {
		db = db.Where("name like ?", "%"+keyword+"%")
	}

	result := db.Find(&list)
	return list, result.Error
}
```

#### 7.2、POST  /   新增|修改

```go
// 新增或编辑资源, 关联更新 casbin_rule 中数据
func (*Resource) SaveOrUpdate(c *gin.Context) {
	var req AddOrEditResourceReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)
	err := model.SaveOrUpdateResource(db, req.ID, req.ParentId, req.Name, req.Url, req.Method)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

func SaveOrUpdateResource(db *gorm.DB, id, pid int, name, url, method string) error {
	resource := Resource{
		Model:    Model{ID: id},
		Name:     name,
		Url:      url,
		Method:   method,
		ParentId: pid,
	}

	var result *gorm.DB
	if id > 0 {
		result = db.Updates(&resource)
	} else {
		result = db.Create(&resource)
		// TODO: ????
		// * 解决前端的 BUG: 级联选中某个父节点后, 新增的子节点默认会展示被选中, 实际上未被选中值
		// * 解决方案: 新增子节点后, 删除该节点对应的父节点与角色的关联关系
		db.Delete(RoleResource{}, "resource_id", id)
	}
	return result.Error
}
```

#### 7.3、DELETE    /:id   删除

```go
// TODO: 考虑删除模块后, 其子资源怎么办? 目前做法是有子资源无法删除
// TODO: 强制删除?
func (*Resource) Delete(c *gin.Context) {
	resourceId, err := strconv.Atoi(c.Param("id"))
	if err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)

	// 检查该资源是否被角色使用
	use, _ := model.CheckResourceInUse(db, resourceId)
	if use {
		ReturnError(c, g.ErrResourceUsedByRole, nil)
		return
	}

	// 获取该资源
	resource, err := model.GetResourceById(db, resourceId)
	if err != nil {
		if errors.Is(err, gorm.ErrRecordNotFound) {
			ReturnError(c, g.ErrResourceNotExist, nil)
			return
		}
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	// 如果作为模块, 检查模块下是否有子资源
	if resource.ParentId == 0 {
		hasChild, _ := model.CheckResourceHasChild(db, resourceId)
		if hasChild {
			ReturnError(c, g.ErrResourceHasChildren, nil)
			return
		}
	}

	rows, err := model.DeleteResource(db, resourceId)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, rows)
}

// 检查该资源是否被角色使用
func CheckResourceInUse(db *gorm.DB, id int) (bool, error) {
	var count int64
	result := db.Model(&RoleResource{}).Where("resource_id = ?", id).Count(&count)
	return count > 0, result.Error
}

// 获取该资源
func GetResourceById(db *gorm.DB, id int) (resource Resource, err error) {
	result := db.First(&resource, id)
	return resource, result.Error
}

// 删除该资源 
func DeleteResource(db *gorm.DB, id int) (int, error) {
	result := db.Delete(&Resource{}, id)
	if result.Error != nil {
		return 0, result.Error
	}
	return int(result.RowsAffected), nil
}
```

#### 7.4、PUT    /anonymous    修改资源匿名访问

```go
// 编辑资源的匿名访问, 关联更新 casbin_rule 中数据
func (*Resource) UpdateAnonymous(c *gin.Context) {
	var req EditAnonymousReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	err := model.UpdateResourceAnonymous(GetDB(c), req.ID, req.Anonymous)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// mysql 修改
func UpdateResourceAnonymous(db *gorm.DB, id int, anonymous bool) error {
	result := db.Model(&Resource{}).Where("id = ?", id).Update("anonymous", anonymous)
	return result.Error
}
```

#### 7.5、GET   /option    资源选项列表

```go
// 获取数据选项(树形)
func (*Resource) GetOption(c *gin.Context) {
	result := make([]TreeOptionVO, 0)

	db := GetDB(c)
	resources, err := model.GetResourceList(db, "")
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	parentList := getModuleList(resources)
	childrenMap := getChildrenMap(resources)

	for _, item := range parentList {
		var children []TreeOptionVO
		for _, re := range childrenMap[item.ID] {
			children = append(children, TreeOptionVO{
				ID:    re.ID,
				Label: re.Name,
			})
		}
		result = append(result, TreeOptionVO{
			ID:       item.ID,
			Label:    item.Name,
			Children: children,
		})
	}
	ReturnSuccess(c, result)
}

// 获取一级资源 (parent_id == 0)
func getModuleList(resources []model.Resource) []model.Resource {
	list := make([]model.Resource, 0)
	for _, r := range resources {
		if r.ParentId == 0 {
			list = append(list, r)
		}
	}
	return list
}


// 存储每个节点对应 [子资源列表] 的 map
// key: resourceId
// value: childrenList
func getChildrenMap(resources []model.Resource) map[int][]model.Resource {
	m := make(map[int][]model.Resource)
	for _, r := range resources {
		if r.ParentId != 0 {
			m[r.ParentId] = append(m[r.ParentId], r)
		}
	}
	return m
}
```



### 8、菜单模块

#### 8.1、GET     /list     菜单列表

```go
func (*Menu) GetTreeList(c *gin.Context) {
	keyword := c.Query("keyword")

	menuList, _, err := model.GetMenuList(GetDB(c), keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, menus2MenuVos(menuList))
}

func GetMenuList(db *gorm.DB, keyword string) (list []Menu, total int64, err error) {
	db = db.Model(&Menu{})
	if keyword != "" {
		db = db.Where("name like ?", "%"+keyword+"%")
	}
	result := db.Count(&total).Find(&list)
	return list, total, result.Error
}
```

#### 8.2、POST     /    新增|编译菜单

```go
func (*Menu) SaveOrUpdate(c *gin.Context) {
	var req model.Menu
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	if err := model.SaveOrUpdateMenu(GetDB(c), &req); err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

func SaveOrUpdateMenu(db *gorm.DB, menu *Menu) error {
	var result *gorm.DB

	if menu.ID > 0 {
		result = db.Model(menu).
			Select("name", "path", "component", "icon", "redirect", "parent_id", "order_num", "catalogue", "hidden", "keep_alive", "external").
			Updates(menu)
	} else {
		result = db.Create(menu)
	}

	return result.Error
}
```

#### 8.3、DELETE    /:id    删除菜单

```go
func (*Menu) Delete(c *gin.Context) {
	menuId, err := strconv.Atoi(c.Param("id"))
	if err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)

	// 检查要删除的菜单是否被角色使用
	use, _ := model.CheckMenuInUse(db, menuId)
	if use {
		ReturnError(c, g.ErrMenuUsedByRole, nil)
		return
	}

	// 如果是一级菜单, 检查其是否有子菜单
	menu, err := model.GetMenuById(db, menuId)
	if err != nil {
		if errors.Is(err, gorm.ErrRecordNotFound) {
			ReturnError(c, g.ErrMenuNotExist, nil)
			return
		}
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	// 一级菜单下有子菜单, 不允许删除
	if menu.ParentId == 0 {
		has, _ := model.CheckMenuHasChild(db, menuId)
		if has {
			ReturnError(c, g.ErrMenuHasChildren, nil)
			return
		}
	}

	if err = model.DeleteMenu(db, menuId); err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

func DeleteMenu(db *gorm.DB, id int) error {
	result := db.Delete(&Menu{}, id)
	return result.Error
}
```

#### 8.4、GET      /user/list    获取当前用户的菜单

```go
// 获取当前用户菜单: 生成后台管理界面的菜单
func (*Menu) GetUserMenu(c *gin.Context) {
	db := GetDB(c)
	auth, _ := CurrentUserAuth(c)

	var menus []model.Menu
	var err error

	if auth.IsSuper {
		menus, err = model.GetAllMenuList(db)
	} else {
		menus, err = model.GetMenuListByUserId(GetDB(c), auth.ID)
	}

	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, menus2MenuVos(menus))
}

// 根据 user_id 获取菜单列表
func GetMenuListByUserId(db *gorm.DB, id int) (menus []Menu, err error) {
	var userAuth UserAuth
	result := db.Where(&UserAuth{Model: Model{ID: id}}).
		Preload("Roles").Preload("Roles.Menus").
		First(&userAuth)

	if result.Error != nil {
		return nil, result.Error
	}

	set := make(map[int]Menu)
	for _, role := range userAuth.Roles {
		for _, menu := range role.Menus {
			set[menu.ID] = menu
		}
	}

	for _, menu := range set {
		menus = append(menus, menu)
	}

	return menus, nil
}

// 获取所有菜单列表（超级管理员用）
func GetAllMenuList(db *gorm.DB) (menu []Menu, err error) {
	result := db.Find(&menu)
	return menu, result.Error
}
```

#### 8.5、GET    /option    菜单选项列表（树形）

```go
func (*Menu) GetOption(c *gin.Context) {
	menus, _, err := model.GetMenuList(GetDB(c), "")
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	result := make([]TreeOptionVO, 0)
	for _, menu := range menus2MenuVos(menus) {
		option := TreeOptionVO{ID: menu.ID, Label: menu.Name}
		for _, child := range menu.Children {
			option.Children = append(option.Children, TreeOptionVO{ID: child.ID, Label: child.Name})
		}
		result = append(result, option)
	}

	ReturnSuccess(c, result)
}

func GetMenuList(db *gorm.DB, keyword string) (list []Menu, total int64, err error) {
	db = db.Model(&Menu{})
	if keyword != "" {
		db = db.Where("name like ?", "%"+keyword+"%")
	}
	result := db.Count(&total).Find(&list)
	return list, total, result.Error
}

func DeleteMenu(db *gorm.DB, id int) error {
	result := db.Delete(&Menu{}, id)
	return result.Error
}
```



### 9、角色模块

#### 9.1、GET   /list    角色列表（树形）

```go
func (*Role) GetTreeList(c *gin.Context) {
	var query PageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)

	result := make([]model.RoleVO, 0)

	list, total, err := model.GetRoleList(db, query.Page, query.Size, query.Keyword)
	if err != nil && !errors.Is(err, gorm.ErrRecordNotFound) {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	for _, role := range list {
		role.ResourceIds, _ = model.GetResourceIdsByRoleId(db, role.ID)
		role.MenuIds, _ = model.GetMenuIdsByRoleId(db, role.ID)
		result = append(result, role)
	}

	ReturnSuccess(c, PageResult[model.RoleVO]{
		Size:  query.Size,
		Page:  query.Page,
		Total: total,
		List:  result,
	})
}

func GetRoleList(db *gorm.DB, num, size int, keyword string) (list []RoleVO, total int64, err error) {
	db = db.Model(&Role{})
	if keyword != "" {
		db = db.Where("name like ?", "%"+keyword+"%")
	}
	db.Count(&total)
	result := db.Select("id", "name", "label", "created_at", "is_disable").
		Scopes(Paginate(num, size)).
		Find(&list)
	return list, total, result.Error
}
```

#### 9.2、POST      /     新增|编辑菜单

```go
func (*Role) SaveOrUpdate(c *gin.Context) {
	var req AddOrEditRoleReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)

	if req.ID == 0 {
		err := model.SaveRole(db, req.Name, req.Label)
		if err != nil {
			ReturnError(c, g.ErrDbOp, err)
			return
		}
	} else {
		err := model.UpdateRole(db, req.ID, req.Name, req.Label, req.IsDisable, req.ResourceIds, req.MenuIds)
		if err != nil {
			ReturnError(c, g.ErrDbOp, err)
			return
		}
	}

	ReturnSuccess(c, nil)
}

func SaveRole(db *gorm.DB, name, label string) error {
	role := Role{
		Name:  name,
		Label: label,
	}
	result := db.Create(&role)
	return result.Error
}
```

#### 9.3、DELETE     /     删除角色

```go
func (*Role) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	err := model.DeleteRoles(GetDB(c), ids)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// 删除角色: 事务删除 role, role_resource, role_menu
func DeleteRoles(db *gorm.DB, ids []int) error {
	return db.Transaction(func(tx *gorm.DB) error {

		result := db.Delete(&Role{}, "id in ?", ids)
		if result.Error != nil {
			return result.Error
		}

		result = db.Delete(&RoleResource{}, "role_id in ?", ids)
		if result.Error != nil {
			return result.Error
		}

		result = db.Delete(&RoleMenu{}, "role_id in ?", ids)
		if result.Error != nil {
			return result.Error
		}

		return nil
	})
}
```

#### 9.4、GET     /option    角色选项列表

```go
func (*Role) GetOption(c *gin.Context) {
	list, err := model.GetRoleOption(GetDB(c))
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, list)
}

func GetRoleOption(db *gorm.DB) (list []OptionVO, err error) {
	result := db.Model(&Role{}).Select("id", "name").Find(&list)
	if result.Error != nil {
		return nil, result.Error
	}
	return list, nil
}
```



### 10、操作日志模块

#### 10.1、GET    /list     操作日志列表

```go
func (*OperationLog) GetList(c *gin.Context) {
	var query PageQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	list, total, err := model.GetOperationLogList(GetDB(c), query.Page, query.Size, query.Keyword)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.OperationLog]{
		Total: total,
		List:  list,
		Size:  query.Size,
		Page:  query.Page,
	})
}

func GetOperationLogList(db *gorm.DB, num, size int, keyword string) (data []OperationLog, total int64, err error) {
	db = db.Model(&OperationLog{})
	if keyword != "" {
		db = db.Where("opt_module LIKE ?", "%"+keyword+"%").
			Or("opt_desc LIKE ?", "%"+keyword+"%")
	}
	db.Count(&total)
	result := db.Order("created_at DESC").
		Scopes(Paginate(num, size)).
		Find(&data)
	return data, total, result.Error
}
```

#### 10.2、DELETE    /     删除日志操作

```go
func (*OperationLog) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	result := GetDB(c).Delete(&model.OperationLog{}, "id in ?", ids)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```



### 11、页面模块

#### 11.1、GET   /list    页面列表

```go
func (*Page) GetList(c *gin.Context) {
	db := GetDB(c)
	rdb := GetRDB(c)

	// get from cache
	cache, err := getPageCache(rdb)
	if cache != nil && err == nil {
		slog.Debug("[handle-page-GetList] get page list from cache")
		ReturnSuccess(c, cache)
		return
	}

	switch err {
	case redis.Nil:
		break
	default:
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	// get from db
	data, _, err := model.GetPageList(db)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	// add to cache
	if err := addPageCache(GetRDB(c), data); err != nil {
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	ReturnSuccess(c, data)
}

// 从 Redis 中获取页面列表缓存
// rdb.Get 如果不存在 key, 会返回 redis.Nil 错误
func getPageCache(rdb *redis.Client) (cache []model.Page, err error) {
	s, err := rdb.Get(rctx, g.PAGE).Result()
	if err != nil {
		return nil, err
	}

	if err := json.Unmarshal([]byte(s), &cache); err != nil {
		return nil, err
	}

	return cache, nil
}
```

#### 11.2、POST   /     新增|编辑列表

```go
func (*Page) SaveOrUpdate(c *gin.Context) {
	var req model.Page
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	db := GetDB(c)
	rdb := GetRDB(c)

	page, err := model.SaveOrUpdatePage(db, req.ID, req.Name, req.Label, req.Cover)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	// delete cache
	if err := removePageCache(rdb); err != nil {
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	ReturnSuccess(c, page)
}

func SaveOrUpdatePage(db *gorm.DB, id int, name, label, cover string) (*Page, error) {
	page := Page{
		Model: Model{ID: id},
		Name:  name,
		Label: label,
		Cover: cover,
	}

	var result *gorm.DB
	if id > 0 {
		result = db.Updates(&page)
	} else {
		result = db.Create(&page)
	}

	return &page, result.Error
}

// 删除 Redis 中页面列表缓存
func removePageCache(rdb *redis.Client) error {
	return rdb.Del(rctx, g.PAGE).Err()
}
```

#### 11.3、DELETE    /    删除

```go
func (*Page) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g.ErrRequest, err)
		return
	}

	result := GetDB(c).Delete(model.Page{}, "id in ?", ids)
	if result.Error != nil {
		ReturnError(c, g.ErrDbOp, result.Error)
		return
	}

	// delete cache
	if err := removePageCache(GetRDB(c)); err != nil {
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	ReturnSuccess(c, result.RowsAffected)
}
```
### 12、用户模块
#### 12.1、user.GET("/list", userAPI.GetList)  // 用户列表

```go
// 获取用户列表
func (*User) GetList(c *gin.Context) {
	var query UserQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	list, count, err := model.GetUserList(GetDB(c), query.Page, query.Size, query.LoginType, query.Nickname, query.Username)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, PageResult[model.UserAuth]{
		Size:  query.Size,
		Page:  query.Page,
		Total: count,
		List:  list,
	})
}

// model
func GetUserList(db *gorm.DB, page, size int, loginType int8, nickname, username string) (list []UserAuth, total int64, err error) {
	if loginType != 0 {
		db = db.Where("login_type = ?", loginType)
	}

	if username != "" {
		db = db.Where("username LIKE ?", "%"+username+"%")
	}

	result := db.Model(&UserAuth{}).
		Joins("LEFT JOIN user_info ON user_info.id = user_auth.user_info_id").
		Where("user_info.nickname LIKE ?", "%"+nickname+"%").
		Preload("UserInfo").
		Preload("Roles").
		Count(&total).
		Scopes(Paginate(page, size)).
		Find(&list)

	return list, total, result.Error
}
```





#### 12.2、user.PUT("", userAPI.Update)  // 修改用户信息



```go
// 更新用户信息: 昵称 + 角色
func (*User) Update(c *gin.Context) {
	var req UpdateUserReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	if err := model.UpdateUserNicknameAndRole(GetDB(c), req.UserAuthId, req.Nickname, req.RoleIds); err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// 数据库修改 gorm
// 更新用户昵称及角色信息
func UpdateUserNicknameAndRole(db *gorm.DB, authId int, nickname string, roleIds []int) error {
	userAuth, err := GetUserAuthInfoById(db, authId)
	if err != nil {
		return err
	}

	userInfo := UserInfo{
		Model:    Model{ID: userAuth.UserInfoId},
		Nickname: nickname,
	}
	result := db.Model(&userInfo).Updates(userInfo)
	if result.Error != nil {
		return result.Error
	}

	// 至少有一个角色
	if len(roleIds) == 0 {
		return nil
	}

	// 更新用户角色, 清空原本的 user_role 关系, 添加新的关系
	result = db.Where(UserAuthRole{UserAuthId: userAuth.UserInfoId}).Delete(UserAuthRole{})
	if result.Error != nil {
		return result.Error
	}

	var userRoles []UserAuthRole
	for _, id := range roleIds {
		userRoles = append(userRoles, UserAuthRole{
			RoleId:     id,
			UserAuthId: userAuth.ID,
		})
	}
	result = db.Create(&userRoles)

	return result.Error
}
```



#### 12.3、user.PUT("/disable", userAPI.UpdateDisable) // 修改用户禁用状态

```go
// 修改用户禁用状态
func (*User) UpdateDisable(c *gin.Context) {
	var req UpdateUserDisableReq

	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	err := model.UpdateUserDisable(GetDB(c), req.UserAuthId, req.IsDisable)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// 修改用户禁用信息
func UpdateUserDisable(db *gorm.DB, id int, isDisable bool) error {
	userAuth := UserAuth{
		Model:     Model{ID: id},
		IsDisable: isDisable,
	}
	result := db.Model(&userAuth).Select("is_disable").Updates(&userAuth)
	return result.Error
}
```



#### 12.4、user.PUT("/current/password", userAPI.UpdateCurrentPassword) // 修改管理员密码



```go
// 修改当前用户密码: 需要输入旧密码进行验证
func (*User) UpdateCurrentPassword(c *gin.Context) {
	var req UpdateCurrentPasswordReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	auth, _ := CurrentUserAuth(c)

	if !utils.BcryptCheck(req.OldPassword, auth.Password) {
		ReturnError(c, g2.ErrOldPassword, nil)
		return
	}

	hashPassword, _ := utils.BcryptHash(req.NewPassword)
	err := model.UpdateUserPassword(GetDB(c), auth.ID, hashPassword)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	// TODO: 修改完密码后，强制用户下线

	ReturnSuccess(c, nil)
}

// 修改管理员密码
func UpdateUserPassword(db *gorm.DB, id int, password string) error {
	userAuth := UserAuth{
		Model:    Model{ID: id},
		Password: password,
	}
	result := db.Model(&userAuth).Updates(userAuth)
	return result.Error
}
```





#### 12.5、user.GET("/info", userAPI.GetInfo)// 获取当前用户信息



```go
// 根据 Token 获取用户信息
func (*User) GetInfo(c *gin.Context) {
	rdb := GetRDB(c)

	user, err := CurrentUserAuth(c)
	if err != nil {
		ReturnError(c, g2.ErrTokenRuntime, err)
		return
	}

	userInfoVO := model.UserInfoVO{UserInfo: *user.UserInfo}
	userInfoVO.ArticleLikeSet, err = rdb.SMembers(rctx, g2.ARTICLE_USER_LIKE_SET+strconv.Itoa(user.UserInfoId)).Result()
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}
	userInfoVO.CommentLikeSet, err = rdb.SMembers(rctx, g2.COMMENT_USER_LIKE_SET+strconv.Itoa(user.UserInfoId)).Result()
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, userInfoVO)
}
```





#### 12.6、user.PUT("/current", userAPI.UpdateCurrent)  // 修改当前用户信息

```go
// 更新当前用户信息, 不需要传 id, 从 Token 中解析出来
func (*User) UpdateCurrent(c *gin.Context) {
	var req UpdateCurrentUserReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	auth, _ := CurrentUserAuth(c)
	err := model.UpdateUserInfo(GetDB(c), auth.UserInfoId, req.Nickname, req.Avatar, req.Intro, req.Website)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// 修改当前用户信息
func UpdateUserInfo(db *gorm.DB, id int, nickname, avatar, intro, website string) error {
	userInfo := UserInfo{
		Model:    Model{ID: id},
		Nickname: nickname,
		Avatar:   avatar,
		Intro:    intro,
		Website:  website,
	}

	result := db.
		Select("nickname", "avatar", "intro", "website").
		Updates(userInfo)
	return result.Error
}
```





#### 12.7、user.GET("/online", userAPI.GetOnlineList)  // 获取在线用户



```go
// 查询当前在线用户
func (*User) GetOnlineList(c *gin.Context) {
	keyword := c.Query("keyword")

	rdb := GetRDB(c)

	onlineList := make([]model.UserAuth, 0)
	keys := rdb.Keys(rctx, g2.ONLINE_USER+"*").Val()

	for _, key := range keys {
		var auth model.UserAuth
		val := rdb.Get(rctx, key).Val()
		json.Unmarshal([]byte(val), &auth)

		if keyword != "" &&
			!strings.Contains(auth.Username, keyword) &&
			!strings.Contains(auth.UserInfo.Nickname, keyword) {
			continue
		}

		onlineList = append(onlineList, auth)
	}

	// 根据上次登录时间进行排序
	sort.Slice(onlineList, func(i, j int) bool {
		return onlineList[i].LastLoginTime.Unix() > onlineList[j].LastLoginTime.Unix()
	})

	ReturnSuccess(c, onlineList)
}
```





#### 12.8、user.POST("/offline/:id", userAPI.ForceOffline)   // 强制用户下线



```go
// 强制离线
func (*User) ForceOffline(c *gin.Context) {
	id := c.Param("id")
	uid, err := strconv.Atoi(id)
	if err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	auth, err := CurrentUserAuth(c)
	if err != nil {
		ReturnError(c, g2.ErrUserAuth, err)
		return
	}

	// 不能离线自己
	if auth.ID == uid {
		ReturnError(c, g2.ErrForceOfflineSelf, nil)
		return
	}

	rdb := GetRDB(c)
	onlineKey := g2.ONLINE_USER + strconv.Itoa(uid)
	offlineKey := g2.OFFLINE_USER + strconv.Itoa(uid)

	rdb.Del(rctx, onlineKey)
	rdb.Set(rctx, offlineKey, auth, time.Hour)

	ReturnSuccess(c, "强制离线成功")
}
```

### 13、文章模块以及上传功能
#### 13.1、/home 后台首页页面

```go
func (*BlogInfo) GetHomeInfo(c *gin.Context) {
	db := GetDB(c)
	rdb := GetRDB(c)
	// 获得article数量
	articleCount, err := model.Count(db, &model.Article{}, "status = ? AND is_delete = ?", 1, 0)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}
	// 获得user数量
	userCount, err := model.Count(db, &model.UserInfo{})
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}
	// 获得message数量
	messageCount, err := model.Count(db, &model.Message{})
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}
	// 获得留言数量
	viewCount, err := rdb.Get(rctx, g2.VIEW_COUNT).Int()
	if err != nil && err != redis.Nil {
		ReturnError(c, g2.ErrRedisOp, err)
		return
	}

	ReturnSuccess(c, BlogHomeVO{
		ArticleCount: articleCount,
		UserCount:    userCount,
		MessageCount: messageCount,
		ViewCount:    viewCount,
	})
}
```

#### 13.2、/upload  文件上传功能

1.主功能函数

```go
func (*Upload) UploadFile(c *gin.Context) {
	_, fileHeader, err := c.Request.FormFile("file")
	if err != nil {
		ReturnError(c, g.ErrFileReceive, err)
		return
	}

	oss := upload.NewOSS()
	filePath, _, err := oss.UploadFile(fileHeader)
	fmt.Println(filePath)
	if err != nil {
		ReturnError(c, g.ErrFileUpload, err)
		return
	}

	ReturnSuccess(c, filePath)
}
```

2.使用七牛云上传

```go
// 七牛云文件上传
type Qiniu struct{}

func (*Qiniu) UploadFile(file *multipart.FileHeader) (filePath, fileName string, err error) {
	putPolicy := storage.PutPolicy{
		Scope: g.GetConfig().Qiniu.Bucket,
	}
	mac := qbox.NewMac(g.GetConfig().Qiniu.AccessKey, g.GetConfig().Qiniu.SecretKey)
	upToken := putPolicy.UploadToken(mac)
	formUploader := storage.NewFormUploader(qiniuConfig())
	ret := storage.PutRet{}

	// 获取文件的 Content-Type
	contentType := file.Header.Get("Content-Type")

	// 设置上传额外参数，包括文件名和文件的 Content-Type
	putExtra := storage.PutExtra{
		Params:   map[string]string{"x:name": "github logo"},
		MimeType: contentType,
	}

	f, openError := file.Open()
	if openError != nil {
		return "", "", errors.New("function file.Open() Filed, err:" + openError.Error())
	}
	defer f.Close()

	// 文件名格式 建议保证唯一性
	fileKey := fmt.Sprintf("%d%s%s", time.Now().Unix(), utils.MD5(file.Filename), path.Ext(file.Filename))
	putErr := formUploader.Put(context.Background(), &ret, upToken, fileKey, f, file.Size, &putExtra)
	if putErr != nil {
		return "", "", errors.New("function formUploader.Put() Filed, err:" + putErr.Error())
	}
	return g.GetConfig().Qiniu.ImgPath + "/" + ret.Key, ret.Key, nil
}

func (*Qiniu) DeleteFile(key string) error {
	mac := qbox.NewMac(g.GetConfig().Qiniu.AccessKey, g.GetConfig().Qiniu.SecretKey)
	cfg := qiniuConfig()
	bucketManager := storage.NewBucketManager(mac, cfg)
	if err := bucketManager.Delete(g.GetConfig().Qiniu.Bucket, key); err != nil {
		return errors.New("function bucketManager.Delete() Filed, err:" + err.Error())
	}
	return nil
}

// 七牛云配置信息
func qiniuConfig() *storage.Config {
	cfg := storage.Config{
		UseHTTPS:      false,
		UseCdnDomains: false,
	}
	switch g.GetConfig().Qiniu.Zone { // 根据配置文件进行初始化空间对应的机房
	case "ZoneHuadong":
		cfg.Zone = &storage.ZoneHuadong
	case "ZoneHuabei":
		cfg.Zone = &storage.ZoneHuabei
	case "ZoneHuanan":
		cfg.Zone = &storage.ZoneHuanan
	case "ZoneBeimei":
		cfg.Zone = &storage.ZoneBeimei
	case "ZoneXinjiapo":
		cfg.Zone = &storage.ZoneXinjiapo
	}
	return &cfg
}
```

**注：** 对文章的查询 的结构体

  ```go
  type ArticleQuery struct {
  	PageQuery
      /*
      // pageQuery 结构体
      type PageQuery struct {
  		Page    int    `form:"page_num"`  // 当前页数（从1开始）
  		Size    int    `form:"page_size"` // 每页条数
  		Keyword string `form:"keyword"`   // 搜索关键字
  	}
      */
  	Title      string `form:"title"`
  	CategoryId int    `form:"category_id"`
  	TagId      int    `form:"tag_id"`
  	Type       int    `form:"type"`
  	Status     int    `form:"status"`
  	IsDelete   *bool  `form:"is_delete"`
  }
  ```

  

#### 13.3、/list  获取文章列表  GetList

```go
func (*Article) GetList(c *gin.Context) {
	var query ArticleQuery
	if err := c.ShouldBindQuery(&query); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}
	// 获取数据库和Redis连接
	db := GetDB(c)
	rdb := GetRDB(c)
	// 查询
	list, total, err := model.GetArticleList(db, query.Page, query.Size, query.Title, query.IsDelete, query.Status, query.Type, query.CategoryId, query.TagId)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}
	// 获取点赞数和浏览量
	likeCountMap := rdb.HGetAll(rctx, g2.ARTICLE_LIKE_COUNT).Val()
	viewCountZ := rdb.ZRangeWithScores(rctx, g2.ARTICLE_VIEW_COUNT, 0, -1).Val()
	// 获取浏览量
	viewCountMap := make(map[int]int)
	for _, article := range viewCountZ {
		id, _ := strconv.Atoi(article.Member.(string))
		viewCountMap[id] = int(article.Score)
	}
	// 封装数据
	data := make([]ArticleVO, 0)
	for _, article := range list {
		likeCount, _ := strconv.Atoi(likeCountMap[strconv.Itoa(article.ID)])
		data = append(data, ArticleVO{
			Article:   article,
			LikeCount: likeCount,
			ViewCount: viewCountMap[article.ID],
		})
	}
	
	ReturnSuccess(c, PageResult[ArticleVO]{
		Size:  query.Size,
		Page:  query.Page,
		Total: total,
		List:  data,
	})
}
```

#### 13.4、/ 新增|编辑文章    SaveOrUpdate

- 路由功能函数

  ```go
  func (*Article) SaveOrUpdate(c *gin.Context) {
  	var req AddOrEditArticleReq
  	if err := c.ShouldBindJSON(&req); err != nil {
  		ReturnError(c, g2.ErrRequest, err)
  		return
  	}
  	// 获取当前用户
  	db := GetDB(c)
  	auth, _ := CurrentUserAuth(c) // 获取当前登录的用户信息
  	// 获取图片
  	if req.Img == "" {
  		req.Img = model.GetConfig(db, g2.CONFIG_ARTICLE_COVER) // 默认图片
  	}
  	// 获取类型
  	if req.Type == 0 {
  		req.Type = 1 // 默认为原创
  	}
  	// 保存文章
  	article := model.Article{
  		Model:       model.Model{ID: req.ID},
  		Title:       req.Title,
  		Desc:        req.Desc,
  		Content:     req.Content,
  		Img:         req.Img,
  		Type:        req.Type,
  		Status:      req.Status,
  		OriginalUrl: req.OriginalUrl,
  		IsTop:       req.IsTop,
  		UserId:      auth.UserInfoId,
  	}
  
  	err := model.SaveOrUpdateArticle(db, &article, req.CategoryName, req.TagNames)
  	if err != nil {
  		ReturnError(c, g2.ErrDbOp, err)
  		return
  	}
  
  	ReturnSuccess(c, article)
  }
  ```

- 数据库底层操作信息

  ```go
  // 新增/编辑文章, 同时根据 分类名称, 标签名称 维护关联表
  func SaveOrUpdateArticle(db *gorm.DB, article *Article, categoryName string, tagNames []string) error {
  	return db.Transaction(func(tx *gorm.DB) error {
  		// 分类不存在则创建
  		category := Category{Name: categoryName}
  		result := db.Model(&Category{}).Where("name", categoryName).FirstOrCreate(&category)
  		if result.Error != nil {
  			return result.Error
  		}
  		article.CategoryId = category.ID
  
  		// 先 添加/更新 文章, 获取到其 ID
  		if article.ID == 0 {
  			result = db.Create(&article)
  		} else {
  			result = db.Model(&article).Where("id", article.ID).Updates(article)
  		}
  		if result.Error != nil {
  			return result.Error
  		}
  
  		// 清空文章标签关联
  		result = db.Delete(ArticleTag{}, "article_id", article.ID)
  		if result.Error != nil {
  			return result.Error
  		}
  
  		var articleTags []ArticleTag
  		for _, tagName := range tagNames {
  			// 标签不存在则创建
  			tag := Tag{Name: tagName}
  			result := db.Model(&Tag{}).Where("name", tagName).FirstOrCreate(&tag)
  			if result.Error != nil {
  				return result.Error
  			}
  			articleTags = append(articleTags, ArticleTag{
  				ArticleId: article.ID,
  				TagId:     tag.ID,
  			})
  		}
  		result = db.Create(&articleTags)
  		return result.Error
  	})
  }
  ```

#### 13.4、/top  更新文章置顶  UpdateTop

```
// 修改置顶信息
func (*Article) UpdateTop(c *gin.Context) {
	var req UpdateArticleTopReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	err := model.UpdateArticleTop(GetDB(c), req.ID, req.IsTop)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// model 修改数据库置顶信息
func UpdateArticleTop(db *gorm.DB, id int, isTop bool) error {
	result := db.Model(&Article{Model: Model{ID: id}}).Update("is_top", isTop)
	return result.Error
}
```

#### 13.5、/:id  文章详情   GetDetail

```go
// 获取文章详细信息
func (*Article) GetDetail(c *gin.Context) {
	id, err := strconv.Atoi(c.Param("id"))
	if err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	article, err := model.GetArticle(GetDB(c), id)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, article)
}

// 数据库根据 id 搜索这个文章信息
func GetArticle(db *gorm.DB, id int) (data *Article, err error) {
	result := db.Preload("Category").Preload("Tags").
		Where(Article{Model: Model{ID: id}}).
		First(&data)
	return data, result.Error
}
```



#### 13.6、/soft-delete  软删除文章    UpdateSoftDelete

```go
func (*Article) UpdateSoftDelete(c *gin.Context) {
	var req SoftDeleteReq
	if err := c.ShouldBindJSON(&req); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	rows, err := model.UpdateArticleSoftDelete(GetDB(c), req.Ids, req.IsDelete)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, rows)
}

// 数据库软删除信息，也就是说再回收站能够恢复的文件，所以说只需要修改软删除标志即可
// 软删除文章（修改）
func UpdateArticleSoftDelete(db *gorm.DB, ids []int, isDelete bool) (int64, error) {
	result := db.Model(Article{}).
		Where("id IN ?", ids).
		Update("is_delete", isDelete)
	if result.Error != nil {
		return 0, result.Error
	}
	return result.RowsAffected, nil
}
```



#### 13.7、/  物理删除文章    Delete

```go
func (*Article) Delete(c *gin.Context) {
	var ids []int
	if err := c.ShouldBindJSON(&ids); err != nil {
		ReturnError(c, g2.ErrRequest, err)
		return
	}

	rows, err := model.DeleteArticle(GetDB(c), ids)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, rows)
}

// 物理删除文章  把数据库里的信息删掉
func DeleteArticle(db *gorm.DB, ids []int) (int64, error) {
	// 删除 [文章-标签] 关联
	result := db.Where("article_id IN ?", ids).Delete(&ArticleTag{})
	if result.Error != nil {
		return 0, result.Error
	}

	// 删除 [文章]
	result = db.Where("id IN ?", ids).Delete(&Article{})
	if result.Error != nil {
		return 0, result.Error
	}

	return result.RowsAffected, nil
}
```



#### 13.8、/export   导出文章   Export

```
sucss
```



#### 13.9、/import   导入文章     Import

```go
// 导入文章: 题目 + 内容
func (*Article) Import(c *gin.Context) {
	db := GetDB(c)
	auth, _ := CurrentUserAuth(c)

	_, fileHeader, err := c.Request.FormFile("file")
	if err != nil {
		ReturnError(c, g2.ErrFileReceive, err)
		return
	}

	fileName := fileHeader.Filename
	title := fileName[:len(fileName)-3]
	content, err := readFromFileHeader(fileHeader)
	if err != nil {
		ReturnError(c, g2.ErrFileReceive, err)
		return
	}

	defaultImg := model.GetConfig(db, g2.CONFIG_ARTICLE_COVER)
	err = model.ImportArticle(db, auth.ID, title, content, defaultImg)
	if err != nil {
		ReturnError(c, g2.ErrDbOp, err)
		return
	}

	ReturnSuccess(c, nil)
}

// md 文件读取
func readFromFileHeader(file *multipart.FileHeader) (string, error) {
	open, err := file.Open()
	if err != nil {
		slog.Error("文件读取, 目标地址错误: ", err)
		return "", err
	}
	defer open.Close()
	all, err := io.ReadAll(open)
	if err != nil {
		slog.Error("文件读取失败: ", err)
		return "", err
	}
	return string(all), nil
}

// 导入文章 也就是数据库新建文章信息
func ImportArticle(db *gorm.DB, userAuthId int, title, content, img string) error {
	article := Article{
		Title:   title,
		Content: content,
		Img:     img,
		Status:  STATUS_DRAFT,
		Type:    TYPE_ORIGINAL,
		UserId:  userAuthId,
	}

	result := db.Create(&article)
	return result.Error
}
```

