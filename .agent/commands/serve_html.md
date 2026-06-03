---
description: 启动一个静态 HTTP 服务预览本地 HTML，并返回基于内网 IP 的完整可访问 URL
---

# serve_html

为本地某个 HTML 文件（通常是 `files/visual-reading/<slug>/index.html` 这类自包含整页）起一个静态 HTTP 服务，让用户能在浏览器里通过**完整 IP 的 URL**（而非 `localhost`）访问。

## 输入

用户会给：
- `html_path`（必填）：要预览的本地 HTML 文件路径

可选：
- `port`：监听端口；缺省自动挑一个空闲端口

`html_path` 缺失或不存在时先问用户，不要猜。

## 关键约定

- **服务根目录要取 HTML 所在目录**，不要直接 serve 单个文件。因为页面常引用同目录的相对资源（`<img src="xxx.png">`、相对 CSS/JS），根目录设错图片就裂。
- **必须 `--bind 0.0.0.0`**，否则只能本机 `localhost` 访问，拿不到对外 IP。
- 返回给用户的 URL 必须是 `http://<IP>:<port>/<相对路径>`，用真实内网 IP，**不能用 `localhost` 或 `127.0.0.1`**。

## 步骤

### 1. 校验输入
- 确认 `html_path` 存在且是 `.html` 文件
- 算出服务根目录 `root_dir = dirname(html_path)`，相对路径 `rel = basename(html_path)`

### 2. 取本机内网 IP
```bash
hostname -I | awk '{print $1}'
```
取第一个非回环地址作为 `<IP>`。

### 3. 选端口
若用户没给 `port`，挑一个空闲端口（例如 9123 起，被占就顺延）：
```bash
for p in 9123 9300 9400 9500; do ss -tln | grep -q ":$p " || { echo $p; break; }; done
```

### 4. 后台启动服务
在 `root_dir` 下启动，绑定所有网卡：
```bash
cd <root_dir> && python3 -m http.server <port> --bind 0.0.0.0
```
用后台方式启动（不要阻塞），记下进程，方便用户之后停。

### 5. 验证可访问
```bash
curl -sI "http://<IP>:<port>/<rel>" | head -3
```
应返回 `200 OK`。非 200 则排查（路径、端口占用、bind 是否生效）。

### 6. 交付
给用户：
- 完整访问 URL：`http://<IP>:<port>/<rel>`
- 停止方式：在对应终端 `Ctrl+C`，或结束该 `python3 -m http.server` 进程
- 若页面引用了同目录下的图片/资源但目录里缺这些文件，提醒用户补齐后刷新

## 验证标准

- `curl -I` 对外 IP 的 URL 返回 `200 OK`
- 返回的 URL 用真实内网 IP，不含 `localhost`/`127.0.0.1`
- 服务根目录是 HTML 所在目录，相对资源能正常加载

## 反面教训

1. **绑定到 `localhost`/默认**：`python3 -m http.server` 默认监听 `0.0.0.0` 其实可以，但显式 `--bind 0.0.0.0` 更稳；关键是给用户的 URL 必须换成真实 IP，别回填 `localhost`。
2. **serve 单文件或起在错误目录**：相对路径的图片/CSS 全裂。永远把服务根设在 HTML 所在目录。
3. **端口硬编码**：常用端口（8000/8080/8888）经常被别的进程占着，先探测再用。
