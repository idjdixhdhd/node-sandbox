# 逆熵 Node 前端项目 · GitHub Codespaces 开发环境部署说明

本套配置让任何人在 GitHub 上打开 Codespaces 后，**自动**完成：Node 22 安装、Continue 插件、VSCode 中文语言包、Prettier、ESLint，并让 Continue 直接可用智谱免费模型。**全程不写死任何密钥**，智谱 API Key 通过 GitHub 仓库 Secret 注入。

---

## 一、文件清单

```
项目根目录/
├── .devcontainer/
│   ├── devcontainer.json        # Codespaces 容器配置（核心）
│   └── continue.config.json     # Continue 配置模板（启动时自动复制生效）
└── SETUP.md                     # 本说明文件
```

| 文件 | 作用 |
|---|---|
| `.devcontainer/devcontainer.json` | 定义容器镜像（Node 22）、自动安装 4 个插件、写入编辑器设置、启动时自动装 prettier/eslint 并复制 Continue 配置 |
| `.devcontainer/continue.config.json` | Continue 的模型配置模板：智谱 GLM-4.7-Flash（对话/编辑/补全）+ GLM-4V-Flash（识图）+ embedding-3（向量） |

---

## 二、部署步骤（GitHub 网页操作，约 3 分钟）

### 第 1 步：把文件放进你的仓库

把 `.devcontainer/` 整个文件夹和 `SETUP.md` 提交到你的 GitHub 仓库根目录，推送到 GitHub。

```bash
git add .devcontainer SETUP.md
git commit -m "feat: add Codespaces devcontainer (Node22 + Continue + zh + prettier + eslint)"
git push
```

### 第 2 步：配置智谱 API Key 为仓库 Secret（关键，别写进代码）

1. 打开你的 GitHub 仓库页面
2. 进入 **Settings**（设置）
3. 左侧菜单点 **Secrets and variables** → **Codespaces**
4. 点右上角 **New repository secret**
5. Name 填：`ZHIPU_API_KEY`
6. Secret 填：你在 [bigmodel.cn](https://bigmodel.cn) 申请到的 API Key（形如 `xxxxxxxx.yyyyyyyy`）
7. 点 **Add secret** 保存

> 说明：Codespaces 会自动把仓库的 Codespaces Secret 注入为容器内的环境变量。Continue 配置里的 `${env:ZHIPU_API_KEY}` 会在运行时读取这个环境变量，密钥不会出现在任何代码或配置里。

### 第 3 步：打开 Codespaces

1. 回到仓库首页，点绿色 **Code** 按钮
2. 切到 **Codespaces** 标签页 → 点 **Create codespace on main**
3. 等待容器构建（首次约 1-3 分钟），构建完成后自动完成：
   - Node 22 就绪（`node -v` 应显示 `v22.x`）
   - 4 个插件自动安装（Continue / 中文语言包 / Prettier / ESLint）
   - prettier、eslint 全局安装
   - Continue 配置自动复制到 `~/.continue/config.json`
   - 若根目录有 `package.json`，自动执行 `npm install`

### 第 4 步：验证

在 Codespaces 终端里依次执行：

```bash
node -v          # 期望 v22.x
npm -v           # 期望 10.x
prettier --version   # 期望 3.x
eslint --version     # 期望 9.x
```

打开任意 JS 文件，按 `Ctrl+Shift+P` 输入 `Continue: Open Chat`，能正常对话即代表智谱 Key 注入成功。

---

## 三、密钥安全说明（务必遵守）

- **严禁**把 `ZHIPU_API_KEY` 写进任何代码、配置文件、README。
- Continue 配置里只写 `${env:ZHIPU_API_KEY}` 占位符，运行时从环境变量读取。
- 密钥只存在 GitHub 的 Secret 里，只有 Codespaces 容器内可见，不会进 git 历史。
- 如果换电脑/换仓库，只需重新在目标仓库配置同一个 Secret，无需改代码。
- 万一 Key 泄露：去 bigmodel.cn 后台重置 Key，然后在 GitHub 更新 Secret 即可。

---

## 四、常见问题

| 问题 | 原因 | 解决 |
|---|---|---|
| Continue 报 401 / 无权限 | Secret 没配好或名字不对 | 检查 Secret 名是否为 `ZHIPU_API_KEY`，值是否完整（含 `.` 前后两段） |
| 打开 Codespaces 没自动装插件 | 缓存了旧容器 | 在 Codespaces 里点右下角齿轮 → **Rebuild Container** |
| 想换模型 | 修改 `.devcontainer/continue.config.json` | 改完 Rebuild Container 或手动复制到 `~/.continue/config.json` |
| 项目有独立的 prettier/eslint 版本 | 全局版可能与项目版冲突 | 以项目 `package.json` 的 devDependencies 为准，全局版仅作兜底 |
| 想用工作区级 Continue 配置 | 不用全局复制 | 把 `continue.config.json` 放到项目根目录 `.continue/config.json`，Continue 会自动读取 |

---

## 五、模型清单（当前模板已配置）

| 用途 | 模型 | 费用 |
|---|---|---|
| 对话 / 编辑 / 应用 | `glm-4.7-flash` | 免费（200K 上下文） |
| 图片识别 | `glm-4v-flash` | 免费（图像理解） |
| 自动补全 | `glm-4.7-flash` | 免费 |
| 向量嵌入 | `embedding-3` | 按量计费（极低） |

如需调整，直接编辑 `.devcontainer/continue.config.json` 后 Rebuild Container 即可。
