# Leaflow Auto Check-in Control Panel

Leaflow 自动签到控制面板是一个基于 Web 的管理界面，用于自动化管理 Leaflow 网站的每日签到任务。支持多账户管理、定时签到、签到结果通知等功能。

## 功能特性
- 🚀 **便捷管理**：支持控制面板便捷化查看和管理签到
- ✅ **多账户管理**：支持添加和管理多个 Leaflow 账户
- ⏰ **定时签到**：为每个账户设置独立的签到时间
- 📊 **数据统计**：可视化展示签到成功率和历史记录
- 🔔 **多平台通知**：支持 Telegram 和企业微信通知
- 🗄️ **数据库支持**：支持 SQLite 和 MySQL 数据库
- 🐳 **Docker 部署**：提供完整的容器化部署方案
- 🔐 **安全认证**：基于 JWT 的安全认证机制
- 📱 **界面自适应**：控制面板自适应PC/移动端界面

## 功能演示
![img](https://github.com/erayopen/picx-images-hosting/raw/master/picdemo/1000095468.7eh4gfyxox.png)

## 快速开始
### 版本更新

最新版本：`ghcr.io/stsix/leaflow-auto-beta:20250927214644`
#### 20250927214644更新内容
##### 1. MySQL重连机制增强
- 实现了指数退避重试策略（3秒、6秒、12秒、24秒...）
- 使用环境变量 `MAX_MYSQL_RETRIES` 控制最大重试次数（默认12次）
- 连接成功后重置重试计数器
- 在所有数据库操作中都应用重试机制

##### 2. 登录界面优化
- 添加了 `/api/verify` 端点验证token有效性
- 页面加载时先验证token，有效则直接显示控制面板
- 避免了登录后刷新页面时闪现登录界面的问题

##### 3. 通知渠道自定义API地址
- 为每个通知渠道添加了自定义API地址输入框
- 数据库表新增了对应字段（telegram_host、wechat_host、wxpusher_host、dingtalk_host）
- 输入框显示默认API地址作为提示
- 留空则使用默认地址

##### 4. WxPusher HTML模板修复
- 修改了HTML模板，使用内联样式确保在深色模式下也能正常显示
- 设置了明确的背景色和文字颜色
- 使用 `color: inherit` 确保颜色继承正确

##### 5. 其他优化
- 添加了定时刷新dashboard数据（每分钟）
- 改进了错误处理和日志记录
- 优化了缓存机制

##### 使用说明：

1. 部署时可以设置环境变量控制MySQL重试：
```bash
docker run -e MAX_MYSQL_RETRIES=20 ...
```

2. 自定义通知API地址示例：
- Telegram: 可以设置为自建的Telegram Bot API服务器
- 企业微信/钉钉: 可以设置为内网代理地址
- WxPusher: 可以设置为私有部署的服务地址

3. MySQL连接失败时的行为：
- 首次连接失败会按指数退避重试
- 运行中连接丢失也会自动重试
- 达到最大重试次数后才会报错

#### 20250923010234更新内容：
新增可自定义TG API代理地址，以解决TG API连接失败问题，由[@showyourlucky](https://github.com/showyourlucky)提交该功能并合入。

#### 20250911094422更新内容：
1. 修复了MySQL连接和保活问题
2. 修复MySQL可能出现的连接错误问题
3. 改进签到逻辑，新增缓存策略，避免MySQL连接池耗尽
4. 新增修改账户cookie功能
5. 优化通知设置布局，新增支持wxpusher通知和钉钉通知
### Docker 部署（推荐）
Docker镜像最新版本：`ghcr.io/stsix/leaflow-auto-beta:20250911094422`

```bash
# 拉取最新镜像
docker pull ghcr.io/stsix/leaflow-auto-beta:latest

# 运行容器
docker run -d \
  --name leaflow-auto \
  -p 8181:8181 \
  -e ADMIN_USERNAME=admin \
  -e ADMIN_PASSWORD=your_secure_password \
  -v /path/to/data:/app/data \
  ghcr.io/stsix/leaflow-auto-beta:latest
```
### leaflow部署
访问[部署清单](https://leaflow.net/apply)，复制下面内容并修改环境变量等配置，粘贴到输入 YAML 配置文本框内，然后点击下方 应用/更新 按钮即可自动部署，部署后自行到服务管理内创建服务并设置外网访问
```
kind: Deployment
name: leaflow-auto
replicas: 1
image_pull_secrets: []
labels: {}
containers:
  - name: leaflow-auto
    image: ghcr.io/stsix/leaflow-auto-beta
    working_dir: ''
    command: []
    args: []
    ports:
      - name: leaflow-auto
        container_port: 8181
        protocol: TCP
    env:
      - name: PORT
        value: '8181，默认为8181'
      - name: ADMIN_USERNAME
        value: 管理员用户名
      - name: ADMIN_PASSWORD
        value: 管理员密码
      - name: JWT_SECRET_KEY
        value: JWT安全密钥
      - name: MYSQL_DSN
        value: mysql://username:password@host:port/dbname
    env_from_configmap: []
    env_from_secret: []
    resources:
    #（这里实测单账户 128M、50毫核即可运行，可根据实际情况调整）
      cpu: 50
      memory: 128
      ephemeral_storage: 1024
    volume_mounts: []
    configmap_mounts: []
    secret_mounts: []
init_containers: []
```

### 环境变量配置

| 环境变量 | 描述 | 默认值 |
|---------|------|--------|
| `PORT` | 服务端口 | `8181` |
| `ADMIN_USERNAME` | 管理员用户名 | `admin` |
| `ADMIN_PASSWORD` | 管理员密码 | `admin123` |
| `JWT_SECRET_KEY` | JWT 密钥（可选） | 自动生成 |
| `MYSQL_DSN` | MySQL 连接字符串（可选） | 使用 SQLite |

MySQL DSN 格式：`mysql://username:password@host:port/dbname`

### 手动部署

1. 克隆项目
```bash
git clone https://github.com/stsix/leaflow-auto-beta.git
cd leaflow-auto-beta
```

2. 安装依赖
```bash
pip install -r requirements.txt
```

3. 运行应用
```bash
python app.py
```

## 使用指南

### 1. 登录系统

访问 `http://localhost:8181` 使用设置的管理员账号登录。

### 2. 添加账户

1. 点击"添加账号"按钮
2. 输入账户名称和签到时间
3. 提供 Cookie 数据（支持多种格式）：
   - JSON 格式：`{"cookies": {"key": "value"}}`
   - 分号分隔格式：`key1=value1; key2=value2`
   - 完整 Cookie 字符串
  
推荐账户cookie格式：
```
{
  "cookies": {
    "leaflow_session": "eyJpdiI6IkxXVEF4M2FTNHNvZTl3PT0iLCJ2YWx1ZSI6ImtxNDNTYW1XNnp5UT09IiwibWFjIjoiYzE5ZDA3NTk2ZjdhIn0%3D",
    "remember_web_59ba36addc2b2f9401580f014c7f58ea4e30989d": "10079%7CqKUDfZrP8MxVnL3mJKtGOabc123456789",
    "XSRF-TOKEN": "eyJpdiI6Ik5PbElabc123456789"
  }
}
多个账户依次添加如上格式cookie即可
```

### 3. 获取 Cookie

1. 浏览器打开 Leaflow 网站并登录
2. 按 F12 打开开发者工具
3. 转到 Network 标签页
4. 刷新页面，选择任意请求
5. 在 Request Headers 中找到 Cookie 并复制

### 4. 配置通知

在通知设置中配置：
- **Telegram Bot Token** 和 **User ID**
- **企业微信 Webhook Key**

## API 接口

### 认证接口
- `POST /api/login` - 用户登录

### 账户管理
- `GET /api/accounts` - 获取所有账户
- `POST /api/accounts` - 添加新账户
- `PUT /api/accounts/:id` - 更新账户
- `DELETE /api/accounts/:id` - 删除账户

### 签到操作
- `POST /api/checkin/manual/:id` - 手动触发签到

### 通知设置
- `GET /api/notification` - 获取通知设置
- `PUT /api/notification` - 更新通知设置
- `POST /api/test/notification` - 测试通知

## 数据库配置

### 使用 SQLite（默认）
无需额外配置，数据将保存在 `/app/data/leaflow_checkin.db`

### 使用 MySQL
设置 `MYSQL_DSN` 环境变量：
```
mysql://username:password@host:port/dbname
```

## 项目结构

```
leaflow-auto-beta/
├── app.py          # 主应用程序
├── Dockerfile      # Docker 构建文件
├── requirements.txt # Python 依赖
└── README.md       # 项目文档
```

## 故障排除

### 常见问题

1. **签到失败**
   - 检查 Cookie 是否有效且未过期
   - 确认网络连接正常

2. **通知不工作**
   - 检查 Telegram Bot Token 和 User ID 是否正确
   - 验证企业微信 Webhook Key 是否有效

3. **数据库连接问题**
   - 检查 MySQL 连接字符串格式是否正确
   - 确认数据库服务可访问

### 日志查看

```bash
# 查看 Docker 容器日志
docker logs leaflow-auto
```

## 🙏 致谢

- [Leaflow](https://leaflow.net) - 容器化部署平台
- [leaflow-checkin](https://github.com/keggin-CHN/leaflow-checkin) - leaflow-checkin（本项目灵感基于该项目）

## 贡献指南

欢迎提交 Issue 和 Pull Request 来改进项目。

## 免责声明

本项目仅供学习和技术交流使用，请勿用于任何商业或非法用途。使用本软件产生的任何问题由使用者自行承担。
