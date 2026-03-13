# Goofy - Spotify 播放列表构建工具

## 项目概述

Goofy 是一个基于 Google Apps Script 平台的 Spotify 播放列表构建工具。提供丰富的功能来收集、过滤和管理 Spotify 音乐，包括收听历史追踪、推荐收集、高级过滤、定时更新等。

**版本：** 2.4.1

**平台：** Google Apps Script（云端 JavaScript 运行时）

**相关资源：**
- 文档：https://chimildic.github.io/goofy
- Telegram 社区：https://t.me/forum_goofy
- GitHub 讨论：https://github.com/Chimildic/goofy/discussions
- 配套 Android 应用：[Audiolist](https://play.google.com/store/apps/details?id=ru.chimildic.audiolist)

## 技术架构

### 核心 API 集成
- **Spotify Web API** - 主要数据源，用于获取播放列表、推荐、艺术家信息等
- **Last.fm API** - 收听历史同步
- **Musixmatch API** - 歌词语言检测

### 模块结构

| 模块 | 功能描述 |
|------|----------|
| `Source` | 获取 Spotify 数据（播放列表、推荐、艺术家等） |
| `Filter` | 过滤和筛选音乐元素 |
| `Selector` | 选择和分支逻辑 |
| `Combiner` | 组合多个数据源 |
| `Order` | 排序功能 |
| `Playlist` | 播放列表创建和管理 |
| `Player` | 播放器控制 |
| `Cache` | Google Drive 数据缓存管理 |
| `Clerk` | 定时任务调度 |
| `RecentTracks` | 收听历史管理 |
| `Search` | 高级搜索功能 |
| `Lastfm` | Last.fm API 集成 |
| `Library` | 喜欢和订阅管理 |
| `Audiolist` | Android 应用集成接口 |

## 项目结构

```
/Volumes/ssd/ai/github/goofy/
├── config.js          # 配置文件（API 密钥、参数设置）
├── library.js         # 主库文件（约4000行，核心逻辑）
├── license            # 许可证
├── readme.md          # 项目说明
├── addons/            # 扩展插件
│   ├── phone-launch/                   # 手机启动控制
│   ├── app-listening-history/          # 应用收听历史
│   ├── genre-releases/                 # 按流派新发布
│   └── fm-radio-tracks/                # FM 广播导入
└── docs/              # 文档（Docsify 构建）
    ├── reference/     # API 参考文档
    ├── img/           # 文档图片
    ├── script/        # 文档脚本
    └── style/         # 文档样式
```

## 配置说明

关键配置项（在 `config.js` 中设置）：

```javascript
// Spotify API 密钥
UserProperties.setProperty('CLIENT_ID', 'yourValue');
UserProperties.setProperty('CLIENT_SECRET', 'yourValue');
UserProperties.setProperty('PRIVATE_CLIENT_ID', 'yourValue');
UserProperties.setProperty('PRIVATE_CLIENT_SECRET', 'yourValue');

// Last.fm 配置
UserProperties.setProperty('LASTFM_API_KEY', 'yourValue');
UserProperties.setProperty('LASTFM_LOGIN', 'yourLogin');

// 收听历史设置
UserProperties.setProperty('ON_SPOTIFY_RECENT_TRACKS', 'true');
UserProperties.setProperty('COUNT_RECENT_TRACKS', '60000');

// 其他设置
UserProperties.setProperty('LOG_LEVEL', 'info');
UserProperties.setProperty('REQUESTS_IN_ROW', '20');
```

## 部署与运行

### 本地开发
此项目为 Google Apps Script 项目，不使用传统的本地构建流程。代码直接在 Google Apps Script 编辑器中编辑和运行。

### 部署步骤
1. 在 [Spotify Dashboard](https://developer.spotify.com/dashboard/) 创建应用，获取 API 密钥
2. 复制 [Apps Script 项目](https://script.google.com/d/1DnC4H7yjqPV2unMZ_nmB-1bDSJT9wQUJ7Wq-ijF4Nc7Fl3qnbT0FkPSr/edit?usp=sharing) 到自己的 Google 账户
3. 在 `config.js` 中配置 API 密钥
4. 运行 `setProperties` 函数初始化配置
5. 部署为 Web 应用

### 触发器设置
通过 Google Apps Script 触发器实现定时运行：
- 使用 `Clerk` 模块调度任务
- 支持 Tasker 事件触发（移动端自动化）

## 开发约定

### 代码风格
- 使用原型扩展（如 `String.prototype.formatName`）
- 模块化设计，每个模块返回公共方法对象
- 错误处理使用 `Admin.printError` 和 `Admin.printInfo`
- 使用 `CustomUrlFetchApp` 封装 HTTP 请求，自动处理速率限制

### 数据处理
- Spotify 对象包含 `popularity`、`duration_ms`、`explicit` 等标准字段
- 音频特征（features）包括 `danceability`、`energy`、`valence` 等（范围 0.0-1.0）
- 使用 `Cache` 模块将数据存储在 Google Drive（JSON 格式）

### API 请求注意事项
- 默认并行请求数：20（可配置 `REQUESTS_IN_ROW`）
- 自动处理 429 响应（速率限制）
- Spotify 可能实施临时 API 限制，需要合理控制请求频率

## 文档更新

文档位于 `docs/` 目录，使用 [Docsify](https://docsify.js.org/) 构建：

```bash
# 本地预览文档（需要本地服务器）
cd docs
python3 -m http.server 3000
# 访问 http://localhost:3000
```

## 常见任务

### 添加新功能
1. 在 `library.js` 中添加新模块或扩展现有模块
2. 在 `docs/reference/` 中添加相应的 API 文档
3. 如需要，在 `addons/` 中创建示例脚本

### 调试
- 设置 `LOG_LEVEL` 为 `info` 获取详细日志
- 使用 `console.log()` 输出调试信息（在 Apps Script 日志中查看）
- 使用 `Admin.setLogLevelOnce()` 临时调整日志级别

### 与 Audiolist 集成
Audiolist Android 应用通过 POST 请求与 Goofy 交互：
- 入口函数：`doPost(args)`
- 主要接口：`Audiolist.onPost()`
- 支持的功能：获取收听历史、缓存读写、同步等

## 语言说明

项目主要使用俄语编写（注释、文档、变量名）。如需修改代码或文档，请保持原有语言风格。

**文档多语言支持（2026-03）：** 文档现已支持俄语和英语双语。英语为默认语言。目录结构为 `docs/en/`（英语）和 `docs/ru/`（俄语）。

## Lessons Learned: 多语言文档迁移

### 项目背景
将原本纯俄语的 Docsify 文档迁移为英语（默认）+ 俄语双语支持。

### 成功实践

**1. Docsify 多语言方案选择**
- 采用子目录方案（`docs/en/`, `docs/ru/`）而非并行文件方案（`overview.en.md`）
- 优点：每种语言有独立的 sidebar/navbar，便于维护
- 配置简单：通过 `alias` 路由 sidebar，无需额外插件

**2. 目录重组策略**
```
docs/
├── index.html          # i18n 配置
├── en/                 # 英语（默认）
│   ├── _sidebar.md
│   ├── _navbar.md      # 包含语言切换器
│   └── reference/
└── ru/                 # 俄语
    └── ...（相同结构）
```

**3. 语言切换器实现**
在 `_navbar.md` 中添加简单的链接：
```markdown
* EN | [RU](/#/ru/)
```
Docsify 的 `router` 插件处理路径重定向。

**4. 翻译工作流**
- 使用专业翻译 agent 并行处理大文件
- 提供术语表确保一致性：
  - плейлист → playlist
  - трек → track
  - исполнитель → artist
  - лайк → like/favorite
- 代码块保持原样，仅翻译注释

### 遇到的问题

**1. Agent 并发限制**
- 问题：平台限制并发 agent 数量（约2个）
- 解决：分批启动 agent，或顺序处理

**2. 俄语目录名**
- 问题：addon 目录使用俄语名称（如 `Запуск с телефона`），URL 编码问题
- 解决：重命名为英文目录名（`phone-launch`）
- 注意：需要更新所有引用这些目录的文档

**3. 内部链接更新**
- 问题：俄语文档中的内部链接需要指向英语路径
- 解决：翻译时更新链接，使用 Docsify 的 alias 功能简化路由

### 配置示例

**index.html 关键配置：**
```javascript
window.$docsify = {
    homepage: 'en/overview.md',  // 默认英语
    loadNavbar: true,
    loadSidebar: true,
    alias: {
        '/.*/_sidebar.md': '_sidebar.md',
        '/en/.*/_sidebar.md': 'en/_sidebar.md',
        '/ru/.*/_sidebar.md': 'ru/_sidebar.md',
    },
    search: {
        placeholder: { '/en/': 'Search', '/ru/': 'Поиск' },
        noData: { '/en/': 'No results', '/ru/': 'Нет результатов' },
    },
};
```

### 文件统计
- 俄语原始文档：~5,640 行
- 英语翻译文档：~5,640 行
- 主要文档：14 个
- API 参考文档：16 个模块

### 后续维护建议

1. **同步更新**：更新文档时，需同时更新两种语言版本
2. **术语一致性**：维护术语对照表，确保翻译一致
3. **自动化**：可考虑使用翻译 API 辅助同步（如 DeepL）
4. **用户贡献**：接受社区翻译贡献，但需审核质量
