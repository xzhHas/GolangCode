---
title: go使用viper去读取yaml配置文件
shortTitle: 20.用viper去读取yaml配置文件
description: go使用viper去读取yaml配置文件
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-12
---

### 1、配置文件yaml
```yaml
logging:
  format: json 
  file:
    path: /var/log/myapp.log 
    max_size: 50MB 
    max_backups: 10 
    compress: true 
features:
  - name: feature1 
    description: This is feature 1 
    enabled: true 
    options: 
      option1: value1 
      option2: value2 
  - name: feature2 
    description: This is feature 2 
    enabled: false 
    options: 
      option1: value3 
      option2: value4
```
### 2、对应的go结构体
```go
type DatabaseConfig struct {
    Name       string           `mapstructure:"name"`
    Connection ConnectionConfig `mapstructure:"connection"`
    }

type ConnectionConfig struct {
    URL      string `mapstructure:"url"`
    Username string `mapstructure:"username"`
    Password string `mapstructure:"password"`
}

type ServerConfig struct {
    Host      string           `mapstructure:"host"`
    Port      int              `mapstructure:"port"`
    Databases []DatabaseConfig `mapstructure:"databases"`
}

type LoggingConfig struct {
    Format string `mapstructure:"format"`
    File   struct {
        Path       string `mapstructure:"path"`
        MaxSize    string `mapstructure:"max_size"`
        MaxBackups int    `mapstructure:"max_backups"`
        Compress   bool   `mapstructure:"compress"`
    } `mapstructure:"file"`
}

type FeatureConfig struct {
    Name        string            `mapstructure:"name"`
    Description string            `mapstructure:"description"`
    Enabled     bool              `mapstructure:"enabled"`
    Options     map[string]string `mapstructure:"options"`
}

type AppConfig struct {
    Server   ServerConfig    `mapstructure:"sever"`
    Logging  LoggingConfig   `mapstructure:"logging"`
    Features []FeatureConfig `mapstructure:"features"
}
```
### 3、使用 viper 去读取配置文件
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181828828.png)

