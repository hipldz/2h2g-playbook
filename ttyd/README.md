# 🖥️ ttyd 配置指南 — 2h2g 云服务器 Web 终端

> **ttyd** 是一个把终端通过浏览器共享出去的轻量级工具，无需 SSH 客户端，打开网页就能操作服务器。  
> 项目主页：https://github.com/tsl0922/ttyd

本目录提供支持移动端的自定义前端 [index.html](./index.html)，通过 ttyd 的 `-I` 参数加载。手机端增加终端快捷键、软键盘视口适配和字号调节，桌面端也可继续使用。

---

## 一、📦 下载 ttyd

Linux 推荐直接下载官方预编译二进制, 或者是 `apt/yum` 云服务器直接安装, 省去编译麻烦。

```bash
# 1. 以下以 1.7.7 为例，其他版本可去 releases 页面选择
# 下载 x86_64 二进制（根据你的架构选择，服务器通常是 x86_64）
wget -O /usr/local/bin/ttyd \
  https://github.com/tsl0922/ttyd/releases/download/1.7.7/ttyd.x86_64

# 2. 赋予可执行权限
chmod +x /usr/local/bin/ttyd

# 3. 验证安装
ttyd --version
```

用 `command -v ttyd` 确认安装路径。下文以 `/usr/bin/ttyd` 为例；若按上面的命令下载到 `/usr/local/bin/ttyd`，请替换手动启动命令和 systemd 的 `ExecStart` 中的二进制路径。

---

## 二、🔐 配置认证与端口

ttyd 通过命令行参数控制所有行为，核心参数说明如下：

| 参数 | 说明 | 示例 |
|------|------|------|
| `-p` | 监听端口 | `-p 7689` |
| `-i` | 绑定网卡 / IP，`0.0.0.0` 表示监听所有网卡 | `-i 0.0.0.0` |
| `-c` | Basic Auth 认证，格式 `用户名:密码` | `-c test:123` |
| `-W` | 允许客户端写入（默认只读，**必须加**才能操作终端） | `-W` |
| `-t` | 客户端选项，如字体大小 | `-t fontSize=16` |
| `-I` | 加载自定义前端 HTML 文件 | `-I /home/pldz1/local/2h2g-playbook/ttyd/index.html` |

**手动测试一下（前台运行）：**

```bash
/usr/bin/ttyd -i 0.0.0.0 -p 7689 -W -c test:123 -t fontSize=16 \
  -I /home/pldz1/local/2h2g-playbook/ttyd/index.html /bin/bash -l
```

将 `-I` 后的路径替换为本仓库 `ttyd/index.html` 的实际绝对路径，并确保运行 ttyd 的用户可以读取该文件。不加 `-I` 时使用 ttyd 内置页面。

## 云服务器测试一定要放行防火墙的端口和设置云服务器的出入站规则.

打开浏览器访问 `http://<你的服务器IP>:7689`，输入用户名 `test`、密码 `123` 即可进入终端。  
确认没问题后按 `Ctrl+C` 退出，接下来配置开机自启。

---


## 三、📱 移动端使用

### 3.1 页面特性

- **自包含前端**：内嵌 xterm.js 和 FitAddon，加载时无需访问 CDN；终端连接仍需访问 ttyd 服务。
- **移动端自动识别**：在触屏或粗指针设备上，屏幕 / 窗口宽度小于 900px 时显示底部工具栏。
- **软键盘适配**：根据可视视口调整终端尺寸，适配横竖屏和屏幕安全区；软键盘弹出时缩小快捷键高度。
- **连接管理**：显示连接状态，支持断线自动重连和手动重连。重连后终端会话是否延续取决于服务端命令；需要保留会话时可结合 tmux 使用。

### 3.2 快捷键与工具栏

底部采用类似 Termux 的固定布局，常用按键分为两排：

```text
ESC   /     -     HOME  ↑  END  PGUP
TAB   CTRL  ALT   ←     ↓  →    PGDN
```

| 控件 | 用法 |
|------|------|
| `CTRL` / `ALT` / `SHIFT` | 单击启用一次，快速双击锁定（显示 `◆`），锁定后再点击取消；作用于工具栏发送的按键 |
| `A−` / `A+` | 调整字号，范围 8–32px，保存到当前浏览器 |
| `PASTE` | 读取剪贴板并粘贴，需要安全上下文（通常为 HTTPS）及浏览器授权；不可用时按钮禁用 |
| `⋯` | 展开 / 收起扩展键，包括 F1–F12、插入 / 删除、退格 / 回车、常用符号和 `CTRL+C` 等组合键 |
| `⌄` / `⌃` | 折叠 / 恢复工具栏，折叠状态保存到当前浏览器 |
| 扩展区 `⌨` | 聚焦终端，便于唤起软键盘 |
| 扩展区 `⛶` | 切换全屏，仅在浏览器支持时显示 |
| 扩展区 `RECON` | 手动重连 |

### 3.3 URL 参数

默认自动识别设备，也可以在访问地址后加参数：

| 参数 | 说明 |
|------|------|
| `mobile=1` / `mobile=0` | 强制开启 / 关闭移动端界面，可用于桌面预览 |
| `toolbar=0` / `toolbar=1` | 隐藏 / 启用移动端工具栏；显示工具栏还需处于移动端界面 |
| `fontSize=16` | 指定字号（8–32），优先于浏览器保存的字号及服务端 `-t fontSize` |
| `debug=1` | 在浏览器控制台输出调试日志 |

例如，在桌面浏览器中预览移动端工具栏：

```text
http://<你的服务器IP>:7689/?mobile=1&fontSize=16
```

请通过 ttyd 服务地址访问页面，直接打开本地 HTML 文件无法连接终端。当前自定义前端未实现 ZMODEM、trzsz、sixel 或 `rendererType` 选项。

---

## 四、🚀 配置 systemd 开机自启

### 4.1 创建 service 文件

```bash
sudo vim /etc/systemd/system/ttyd.service
```

粘贴以下内容（**按需修改用户名、端口、账号密码、二进制及 HTML 路径**）：

```ini
[Unit]
Description=ttyd web terminal
After=network.target

[Service]
Type=simple
User=pldz1
WorkingDirectory=/home/pldz1
ExecStart=/usr/bin/ttyd -i 0.0.0.0 -p 7689 -W -c test:123 -t fontSize=16 -I /home/pldz1/local/2h2g-playbook/ttyd/index.html /bin/bash -l
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

> **字段说明：**
> - `User=pldz1`：以 `pldz1` 用户身份运行，终端权限与该用户一致
> - `WorkingDirectory`：终端启动时的工作目录
> - `ExecStart` 中的 `-I`：指定移动端自定义前端的绝对路径，`User` 指定的用户需有读取权限
> - `Restart=always`：进程崩溃后自动重启
> - `RestartSec=3`：重启前等待 3 秒

### 4.2 启用并启动服务

```bash
# 让 systemd 识别新文件
sudo systemctl daemon-reload

# 设置开机自启
sudo systemctl enable ttyd

# 立即启动
sudo systemctl start ttyd

# 查看运行状态
sudo systemctl status ttyd
```

看到 `Active: active (running)` 就说明成功了 ✅

本目录的 `ttyd.service.copy` 是基础配置示例，使用时也需在 `ExecStart` 中加入上述 `-I` 参数以加载移动端页面。

---

## 五、🔄 修改配置后如何重载服务

每次修改 `/etc/systemd/system/ttyd.service` 之后，**必须按以下顺序执行**，否则改动不会生效：

```bash
# 第一步：重新加载 systemd 守护进程，让它读取最新的 service 文件
sudo systemctl daemon-reload

# 第二步：重启 ttyd 服务，使新配置生效
sudo systemctl restart ttyd

# 第三步（可选）：确认服务正常运行
sudo systemctl status ttyd
```

> ⚠️ **常见错误**：只执行 `restart` 而跳过 `daemon-reload`，此时 systemd 仍然使用旧的配置文件内容，修改不会生效。**两步缺一不可。**

如果只更新 `ttyd/index.html`，无需执行 `daemon-reload`；重启 ttyd 后刷新浏览器页面即可。如果仍显示旧页面，请强制刷新或清理页面缓存，并检查服务的 `ExecStart` 是否指向更新后的文件。

---

## 六、🛠️ 常用管理命令速查

```bash
# 查看服务状态
sudo systemctl status ttyd

# 启动
sudo systemctl start ttyd

# 停止
sudo systemctl stop ttyd

# 重启（配置未改动时使用）
sudo systemctl restart ttyd

# 查看实时日志
sudo journalctl -u ttyd -f

# 查看最近 50 行日志
sudo journalctl -u ttyd -n 50
```

---

## 七、🔒 安全建议

1. **修改默认密码**：`-c test:123` 只是示例，生产环境务必换成强密码，如 `-c admin:Str0ng@Pass!`
2. **限制来源 IP**：安全组入站规则中，将来源从 `0.0.0.0/0` 改为你自己的固定 IP，大幅降低暴露风险
3. **建议配置 HTTPS**：在 ttyd 前面挂一个 Nginx 反向代理并配置 SSL 证书，避免账号密码明文传输

---

## 参考

- 官方仓库：https://github.com/tsl0922/ttyd
- 官方 Releases：https://github.com/tsl0922/ttyd/releases
- 使用示例 Wiki：https://github.com/tsl0922/ttyd/wiki/Example-Usage
