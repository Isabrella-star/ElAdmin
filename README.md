# ELAdmin 文件管理与数据导出改造说明

> 本 README 是个人改造记录，只描述我在原项目基础上新增加强的前端能力。
> 原项目版权、使用说明仍以各子目录内原作者 README 为准，未做覆盖。

原文参考：https://github.com/elunez/eladmin-web

## 项目结构

```text
E:\ElAdmin
├── eladmin       # 后端项目（本次未改动）
└── eladmin-web   # 前端项目（本次改造集中在这里）
```

## 本次改造内容

### 1. 文件管理模块强化

在原有文件存储页面的基础上，新增和加强了以下能力：

- 拖拽上传
- 多文件上传
- 上传进度条
- 上传前校验文件大小和类型
- 图片预览
- PDF 预览
- 文件下载
- 批量删除
- 上传失败重试
- 空状态与加载状态

主要页面：

- 本地存储：`系统工具 -> 存储管理 -> 本地存储`
- 对象存储：`系统工具 -> 存储管理 -> 对象存储`

### 2. 数据导出统一组件

将原本散落的导出按钮统一为可复用组件，并新增以下能力：

- 统一选择导出 Excel 或 CSV
- 导出过程 loading
- 成功 / 失败提示
- 文件名处理
- 权限控制
- Excel 走后端 `/download` 接口
- CSV 使用前端数据直接生成，不依赖后端

### 3. 顶部标签拖拽排序

基于 `vuedraggable` 让顶部访问标签页支持拖拽排序：

- 拖动标签后松手不再回弹，顺序实时更新
- 通过 Vuex `SORT_VISITED_VIEWS` mutation 与 `sortVisitedViews` action 保存顺序
- `visitedViews` 改为带 `getter / setter` 的计算属性，拖拽结果自动写回 Vuex

### 4. 面包屑导航可点击跳转

优化面包屑，使其除末级外均可点击：

- 中间层页面点击后跳转到对应页面
- 目录节点点击后跳转到该目录下的第一个可点击页面
- 移除 `path-to-regexp` 依赖，改为 `fullPath` 拼接与手写 `pathCompile` 替换路径参数

### 5. 表格列配置化抽象

把页面中重复的“普通数据列”抽成 `columns` 配置，并新增公共列组件：

- 新增 `CrudColumn` 组件，用 `render` 函数渲染 `el-table-column`
- `selection`、操作列、自定义 `slot` 列仍保留手写，不放进 `columns`
- 已迁移页面：字典管理、字典详情、岗位管理、本地存储
- 剩余 12 个已配置化页面待批量替换为 `CrudColumn`

## 涉及文件

### 新增文件

| 文件 | 作用 |
| --- | --- |
| `eladmin-web/src/components/FileUpload/index.vue` | 可复用的文件上传组件 |
| `eladmin-web/src/components/ExportButton/index.vue` | 可复用的导出按钮组件 |
| `eladmin-web/src/utils/file.js` | 文件判断、下载、格式化等工具函数 |
| `eladmin-web/src/utils/export.js` | CSV / Excel 前端导出工具 |
| `eladmin-web/src/components/Crud/Column.vue` | 配置驱动的表格列公共组件 |

### 修改文件

| 文件 | 改动说明 |
| --- | --- |
| `eladmin-web/src/utils/upload.js` | 新增 `uploadWithProgress`，支持上传进度 |
| `eladmin-web/src/components/Crud/CRUD.operation.vue` | 统一接入新的导出按钮组件 |
| `eladmin-web/src/views/tools/storage/local/index.vue` | 强化本地存储页上传、预览、下载、空状态，并完成表格列配置化 |
| `eladmin-web/src/views/tools/storage/s3/index.vue` | 强化对象存储页上传、预览、下载、空状态 |
| `eladmin-web/src/views/system/timing/log.vue` | 将旧的散落导出按钮替换为统一导出组件 |
| `eladmin-web/src/store/modules/tagsView.js` | 新增标签排序 mutation 与 action |
| `eladmin-web/src/layout/components/TagsView/index.vue` | 接入 `vuedraggable`，实现标签拖拽排序 |
| `eladmin-web/src/components/Breadcrumb/index.vue` | 面包屑改为可点击跳转 |
| `eladmin-web/src/views/system/dict/index.vue` | 迁移到 `CrudColumn` 组件 |
| `eladmin-web/src/views/system/dict/dictDetail.vue` | 表格列配置化 |
| `eladmin-web/src/views/system/job/index.vue` | 表格列配置化，并修复创建日期列重复 |

## 运行方式

### 后端

1.启动Redis和MySQL

2.导入数据库（CMD）

```
mysql -uroot -p你的密码 -e "CREATE DATABASE IF NOT EXISTS eladmin CHARACTER SET utf8mb4"
mysql -uroot -p你的密码 eladmin < (eladmin.sql真实路径)
mysql -uroot -p你的密码 eladmin < (quartz.sql真实路径)
```

3.修改数据库密码

修改 `eladmin/eladmin-system/src/main/resources/config/application-dev.yml`

把 `username`、`password` 改成本机的 MySQL 账号密码

4..在IDEA里面启动项目

启动 `eladmin` 项目的 Spring Boot 服务，默认接口地址为：

```text
http://localhost:8000
```

需要保证数据库已初始化，并按需配置本地存储、S3 对象存储相关配置。

### 前端

进入 `eladmin-web` 目录：

```bash
cd eladmin-web
npm install
npm run dev
```

默认访问地址：

```text
http://localhost:8013
```

使用管理员账号登录后即可验证。

## 验证页面

### 文件管理

进入 `系统工具 -> 存储管理 -> 本地存储`，可依次验证：

- 点击上传，拖拽多个文件或点选多文件
- 上传时查看每个文件的进度条
- 上传超过 100MB 文件时提示大小超限
- 上传不支持类型时提示格式不支持
- 上传失败后点击重试
- 图片和 PDF 的预览
- 文件下载
- 勾选多行后批量删除
- 空数据时的空状态展示

对象存储页同样支持以上交互，但需要先正确配置 S3。

### 数据导出

以下页面右上角会出现统一的导出按钮，可下拉选择 Excel 或 CSV：

- `系统管理 -> 用户管理`
- `系统管理 -> 角色管理`
- `系统管理 -> 字典管理`
- `系统监控 -> 操作日志`
- `系统监控 -> 异常日志`
- `系统监控 -> 在线用户`
- `系统工具 -> 存储管理`
- `系统管理 -> 任务调度` 的任务日志弹窗

### 标签拖拽与面包屑

- 打开多个页面后，在顶部标签栏拖动标签，观察顺序变化且松手不回弹
- 进入多级页面后，点击面包屑中间层，验证能否跳转到对应页面
- 点击目录类面包屑，验证是否跳转到该目录下第一个可点击页面

### 表格列配置化

- 进入 `系统管理 -> 字典管理`、`系统管理 -> 岗位管理`，确认表格列正常渲染
- 进入 `系统工具 -> 存储管理 -> 本地存储`，确认自定义列与配置列均正常显示

## 技术要点

- 使用 `Element UI` 的 `el-upload` 作为拖拽入口，真正的上传请求交给 `axios` 控制。
- 使用 `onUploadProgress` 实现每个文件独立的进度条。
- 将上传、下载、导出逻辑抽离为可复用组件和工具函数，降低页面间重复代码。
- 通过 `props` 注入业务函数，让同一个上传组件兼容本地存储和 S3 存储。
- 通过“状态机”管理每个文件的上传状态：准备、上传中、成功、失败。
- 在 CSV / Excel 导出中增加公式注入防护，避免以 `= + - @` 开头的单元格被当作公式执行。
- 只在共用的 CRUD 工具栏组件中替换导出按钮，实现最小侵入式改造。
- 标签排序通过 `vuedraggable` 的 `v-model` 配合计算属性 `getter / setter` 写回 Vuex。
- 面包屑用路由 `matched` 逐层拼接 `fullPath`，并用手写 `pathCompile` 替换 `:param`。
- `CrudColumn` 使用 `render` 函数按需转发 `scopedSlots`，避免普通数据列被误判为自定义单元格。

## 与原项目的关系

- 后端代码未改动。
- 原项目 README、LICENSE 等文件均保留。
- 本 README 仅作为个人开发与学习记录。
