# toolkit

运维工具箱，同时作为 sre-scaffold 的物料仓库。所有部署物料（Ansible role、配置模板、运维脚本）按约定组织在本目录中。

## 目录说明

### ansible/roles/ — 版本化 Ansible role

按 `组件名/版本号/` 组织，一个 Git 仓库承载所有版本：

```
ansible/roles/<组件>/<版本>/
├── tasks/main.yml          # 部署任务（playbook 主体）
├── templates/*.j2          # Jinja2 配置模板（用户前端填参 → 渲染 → 部署）
├── defaults/main.yml       # 默认变量（双重用途：Ansible 默认值 + 前端表单字段定义）
└── files/                  # 静态文件（Dockerfile、证书等）
```

**版本切换方式**：切换目录路径，不依赖 Git 分支。Git 分支留给 staging/production 环境区分用。

```
mysql 8.0 → ansible/roles/mysql/8.0/tasks/main.yml
mysql 8.4 → ansible/roles/mysql/8.4/tasks/main.yml
```

### conf/ — 通用配置模板

不属于某个特定 role、可被多个组件引用的配置片段：

```
conf/nginx/
├── nginx.conf.j2           # Nginx 主配置模板
└── vhost.conf.j2           # 虚拟主机配置模板
```

### linux-init/ — Linux 服务器初始化

全新 Linux 服务器的一键初始化 playbook：

| 文件 | 说明 |
|------|------|
| `init.yml` | 主 playbook，编排 5 个 role |
| `inventory.example` | 主机清单示例 |
| `group_vars/all.yml` | 全局可配置变量 |
| `roles/network/` | 设置主机名、静态 IP、DNS |
| `roles/ssh-hardening/` | SSH 加固（禁止密码登录） |
| `roles/base-packages/` | 安装 tree/vim/curl/htop 等常用工具 |
| `roles/oh-my-zsh/` | 安装 zsh + oh-my-zsh + 插件 |
| `roles/firewall/` | 配置防火墙（ufw/firewalld） |

支持 Debian/Ubuntu 和 CentOS/RHEL/Rocky。

## 与 sre-scaffold 的协作

1. sre-scaffold 的 Web 界面扫描 `ansible/roles/` 列出可用组件和版本
2. 用户在界面选择组件+版本 → 后端渲染对应 `templates/*.j2` → Ansible 执行
3. 用户填写的参数通过 `--extra-vars` 注入 playbook

## 前端配置表单的生成逻辑

1. 读取组件 `defaults/main.yml` 中的变量定义
2. 将每个变量渲染为对应类型的表单字段
3. 用户填入的值在部署时注入 Ansible
