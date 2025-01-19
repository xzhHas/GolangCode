---
title: goframe 目录分块
shortTitle: 22.goframe框架
description: goframe 目录分块
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-19
---
:::tip

- goframe官网：[https://wiki.goframe.org/pages/viewpage.action?pageId=57183742](https://wiki.goframe.org/pages/viewpage.action?pageId=57183742)

:::

## goframe 目录分块

```go
├─api
├─hack
├─internal
│ ├─cmd
│ ├─consts
│ ├─controller
│ ├─dao
│ ├─logic
│ ├─model
│ │ ├─do
│ │ └─entity
│ ├─packed
│ └─service
├─manifest
│ ├─config
│ ├─deploy
│ │ └─kustomize
│ │ ├─base
│ │ └─overlays
│ │ └─develop
│ ├─docker
│ ├─i18n
│ └─protobuf
├─resource
│ ├─public
│ │ ├─html
│ │ ├─plugin
│ │ └─resource
│ │ ├─css
│ │ ├─image
│ │ └─js
│ └─template
```

### 顶级目录

- `api`: 存放与外部接口（如 HTTP、gRPC 等）相关的代码。通常包括路由、请求处理和响应逻辑。
- `hack`: 一般用于存放临时的脚本、工具或调试代码，这些内容可能对项目有用但不是核心的一部分。
- `internal`: 存放内部逻辑和功能的代码，防止被其他包依赖，确保模块化和封装性。

### `internal` 子目录

- `cmd`: 存放项目的命令行相关代码，通常是应用程序的入口点。
- `consts`: 定义项目中的常量，便于统一管理和使用。
- `controller`: 控制器层，负责处理请求，调用相应的服务逻辑，并返回结果。通常是业务逻辑的入口。
- `dao`: 数据访问层（Data Access Object），负责与数据库交互，执行 CRUD 操作。
- `logic`: 业务逻辑层，包含具体的业务规则和逻辑处理。
- `model`: 存放数据模型，定义项目中的结构体。
  - `do`: 数据对象（Data Object），用于数据库交互的对象。
  - `entity`: 实体对象（Entity），用于业务逻辑处理的对象。
- `packed`: 一般用于存放打包后的文件或资源，可以是编译后的静态资源等。
- `service`: 服务层，提供对外的服务接口，实现具体的业务功能。通常由多个 controller 调用。

### `manifest` 子目录

- `config`: 配置文件和相关代码，管理项目的配置项。
- `deploy`: 部署相关的文件和脚本，定义项目的部署方式。
  - `kustomize`: Kustomize 是 Kubernetes 的一种资源定制工具，存放与 Kubernetes 相关的资源定制文件。
    - `base`: 基础配置，通常包含所有环境的共有配置。
    - `overlays`: 覆盖层，包含不同环境的特定配置。
      - `develop`: 开发环境的配置。
- `docker`: Docker 相关的文件，包括 Dockerfile 和其他 Docker 配置。
- `i18n`: 国际化相关的文件和代码，管理多语言支持。
- `protobuf`: 存放 Protobuf 定义文件，用于定义 gRPC 接口和消息格式。

### `resource` 子目录

- `public`: 对外公开的静态资源，通常用于前端页面展示。
  - `html`: HTML 文件。
  - `plugin`: 插件相关文件。
  - `resource`: 资源文件。
    - `css`: 样式表文件。
    - `image`: 图像文件。
    - `js`: JavaScript 文件。
- `template`: 模板文件，通常用于渲染动态内容。

### `utility`

- `utility`: 工具包，包含项目中使用的各种工具函数和辅助功能，通常是与业务逻辑无关的通用功能。
