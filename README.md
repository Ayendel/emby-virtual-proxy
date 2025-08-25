# Emby Virtual Proxy — Advanced Virtual Libraries and Filters
[![Releases](https://img.shields.io/github/v/release/Ayendel/emby-virtual-proxy)](https://github.com/Ayendel/emby-virtual-proxy/releases)

![Media Server](https://upload.wikimedia.org/wikipedia/commons/3/3a/Play_button_icon.svg)

强大的 Emby 中间代理，扩展 Emby 原生功能，按需创建虚拟库并过滤媒体资源。

目录
- 功能亮点
- 快速上手
- 配置示例
- 虚拟库类型与筛选器
- 常见用例
- API 拦截与自定义规则
- 部署建议
- 常见问题
- 贡献指南
- 版本日志（重要更新）
- 发布与下载

功能亮点
- 作为 Emby 客户端和真实 Emby 服务器之间的代理。拦截并修改 API 请求与响应。
- 支持创造虚拟库（Virtual Library）。可基于标签、演员、年份、评分或自定义规则聚合媒体条目。
- 新增“全库”（All Libraries）类型：在一个虚拟库中包含所有物理库的条目，配合高级筛选器即可实现对整个媒体池的统一视图。
- 动态修改首页和栏目请求，保证“最新”等栏目显示正确的影视项目，而非虚拟库自身。
- 可选访问控制（密码与 API 密钥白名单），便于小规模自托管部署时限制访问。（该功能可启用或移除）
- 前端与后端双向调整：前端优化验证逻辑，后端简化代理处理流程，提升稳定性与可维护性。

快速上手
1. 访问发布页并下载发布包或可执行文件：
   https://github.com/Ayendel/emby-virtual-proxy/releases
2. 解压并运行代理主程序。常见命令示例（以 Linux 为例）：
   - 下载并解压（示例）：
     ```
     curl -L -o emby-virtual-proxy.tar.gz "https://github.com/Ayendel/emby-virtual-proxy/releases/download/v1.3.4/emby-virtual-proxy-1.3.4.tar.gz"
     tar -xzf emby-virtual-proxy.tar.gz
     cd emby-virtual-proxy
     ```
   - 运行（如果为 Python 程序）：
     ```
     python3 proxy_server.py --config config.yml
     ```
   按发布包内说明执行相应文件。请下载对应平台的发布文件并执行。

配置示例
基本配置文件（YAML/JSON 均可）示例，展示代理端口、后端 Emby 地址与虚拟库设置（示例为 YAML）：
```yaml
server:
  listen: 0.0.0.0
  port: 8080
emby:
  backend_url: "http://192.168.1.100:8096"
virtual_libraries:
  - id: "vl_all"
    name: "全库筛选"
    type: "all_libraries"   # 全库类型
    filters:
      - field: "Genres"
        op: "contains"
        value: "科幻"
      - field: "OfficialRating"
        op: "gte"
        value: 7
  - id: "vl_movies"
    name: "精选电影"
    type: "library"
    resource_ids:
      - "movies-library-id"
    filters:
      - field: "Year"
        op: "between"
        value: [2000, 2025]
access_control:
  enabled: false
  password: ""
  api_keys: []
```
虚拟库字段说明
- id: 唯一标识
- name: 显示名称
- type: 支持 library（单库）、collections（集合）和 all_libraries（全库）
- resource_ids: 适用于非“全库”类型，用以指定库或集合的 ID
- filters: 一组按需组合的筛选规则

虚拟库类型与筛选器
- 单库（library）：基于单个物理库建立虚拟集合，可交叉筛选。
- 集合（collections）：基于 Emby 的 Collection 概念合成库。
- 全库（all_libraries）：将所有物理库聚合到一处，再对条目进行统一筛选。若选中该类型，可不必填写 resource_ids。
筛选器操作符（示例）
- equals / contains / in / gt / gte / lt / lte / between
示例规则
- 按演员：field: "People", op: "contains", value: "Leonardo DiCaprio"
- 按评分：field: "CommunityRating", op: "gte", value: 8
- 按年份：field: "Year", op: "between", value: [1990, 2005]

常见用例
- 创建“全库经典电影”虚拟库，包含所有物理库中评分高于 8 的电影。
- 为儿童用户创建过滤器，排除成人分级和某些类别。
- 在首页“最新”栏目保持仅显示实际影视条目，避免将虚拟库实体自身显示为内容。
- 按演员或导演快速生成专题浏览页面。

API 拦截与自定义规则
代理核心监听并修改 Emby API 请求与响应。常见改动点：
- 修改 /Items/latest 类请求，强制添加媒体类型过滤，避免虚拟库条目出现在“最新”中。
- 拦截 Items 查询，合并来自多个物理库的条目，返回去重后的列表。
- 动态插入或移除响应字段，增强客户端显示逻辑或兼容某些移动端 App。
编写自定义规则
- 规则主要基于 URL 路径、请求方法与 JSON 内容。编写时请尽量保持规则粒度小，避免过度匹配。

部署建议
- 将代理部署在离 Emby 服务器同一局域网内，以减少网络延迟。
- 使用 systemd 或容器化部署（Docker）以便于管理。
- 若启用访问控制，请为密码与 API 密钥使用强随机值并妥善存储。
- 为长期运行建议启用日志轮换，保留最近 N 天日志。

常见问题（FAQ）
Q: 启用“全库”类型无法保存配置
A: 请使用最新前端及代理版本。近期修复了前端验证逻辑，无需指定具体资源 ID 即可保存“全库”类型。

Q: “最新”栏目仍然显示虚拟库项
A: 请确认代理启用了对首页请求的过滤逻辑。可在配置中强制对媒体类型进行筛选，确保只返回实际影视条目。

Q: 我需要访问控制但想保持简单
A: 使用内置的密码或 API key 白名单功能。若你不需要访问控制，可在配置中禁用，系统会移除相关检查。

贡献指南
- 欢迎提交 issue 以报告 BUG 或提出改进建议。
- 提交 PR 时请包含清晰的变更说明与必要的测试步骤。
- 代码风格遵循项目已制定的 lint 规则。对大型重构请先在 issue 中讨论设计方案。

版本日志（摘录）
- 1.3.4 (2025-08-18)
  - 新增“全库”虚拟库类型。新类型允许在创建虚拟库时选择包含所有媒体库的内容。
  - 修复“全库”类型保存问题：前端验证逻辑更新，允许在无资源 ID 时保存。
  - 修正首页“最新”栏目请求逻辑，确保全库类型不会导致不正确的内容展示，并通过强制筛选媒体类型解决虚拟库项错显的问题。
- 1.3.3 (2025-08-16)
  - 重构：移除访问控制模块（密码与 API key 白名单），以简化核心功能。前端与后端均删除了相关配置与代码。
- 1.3.2 (2025-08-16)
  - 新增可选访问控制：密码保护与 API 密钥白名单（可配置启用）。

发布与下载
请访问发布页下载对应平台的发布文件并执行：
https://github.com/Ayendel/emby-virtual-proxy/releases

从发布页下载适合你平台的资产（示例：zip、tar.gz、二进制可执行文件或 Python 包）。下载后，请按压缩包内的 README 或运行脚本启动代理。示例命令（替换为实际文件名）：
```
# 替换为实际下载链接 / 文件名
tar -xzf emby-virtual-proxy-1.3.4.tar.gz
cd emby-virtual-proxy
python3 proxy_server.py --config config.yml
```

许可
本项目采用 MIT 许可证。参见 LICENSE 文件获取完整文本。

联系方式与社区
- 提交问题请在仓库 Issue 页面发起。
- 提交补丁请使用 Pull Request 并附带测试说明与复现步骤。

截图与素材
- 代理不依赖特定 UI，工作在应用层。示例配图用于展示媒体服务器主题，非功能界面快照。

常用命令参考
- 启动：
  ```
  python3 proxy_server.py --config config.yml
  ```
- 后台运行（systemd 示例）：
  - 创建 /etc/systemd/system/emby-virtual-proxy.service，指向启动脚本或可执行文件。
- 日志查看：
  ```
  tail -f logs/proxy.log
  ```

调试提示
- 启动时将代理指向真实 Emby 后端地址，确保代理能连通 Emby。使用 curl 测试：
  ```
  curl http://127.0.0.1:8080/Items?Limit=1
  ```
- 若出现权限或 401 错误，检查后端 Emby 的身份验证策略与授权头是否正确传递。

贡献者守则
- 请在提交 PR 前确保变更经过本地测试。
- 对于重大功能，请先在 issue 讨论设计与兼容性。
- 保持提交信息简洁并说明目的。

再次访问发布页以获取最新版本并下载执行：
[![Release Download](https://img.shields.io/badge/Download-Releases-blue)](https://github.com/Ayendel/emby-virtual-proxy/releases)