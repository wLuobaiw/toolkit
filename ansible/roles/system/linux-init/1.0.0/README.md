# linux-init

一键初始化全新 Linux 服务器：网络配置、SSH 加固、常用工具安装、oh-my-zsh、防火墙。

## 目录结构

```
linux-init/1.0.0/
├── conf.yaml              # 配置表单定义（sre-scaffold Web UI 读取）
├── defaults/main.yaml     # 默认变量
├── tasks/
│   ├── main.yaml          # 任务入口
│   ├── network.yaml       # 网络配置
│   ├── ssh-hardening.yaml # SSH 加固
│   ├── base-packages.yaml # 常用工具
│   ├── oh-my-zsh.yaml     # oh-my-zsh（可选）
│   └── firewall.yaml      # 防火墙
├── handlers/main.yaml     # handler 定义
├── templates/             # Jinja2 模板
├── inventory.example
└── README.md
```

## 功能清单

| 任务文件 | 说明 | 是否可选 |
|----------|------|----------|
| `network.yaml` | 设置主机名、静态IP、DNS | 必装 |
| `ssh-hardening.yaml` | 备份 sshd_config，设置 PermitRootLogin prohibit-password | 必装 |
| `base-packages.yaml` | 安装 tree, vim, curl, wget, git, htop, lsof, rsync, tcpdump, jq, tar | 必装 |
| `oh-my-zsh.yaml` | 安装 zsh + oh-my-zsh + 插件 | **可选** |
| `firewall.yaml` | Debian→ufw，RHEL→firewalld，放行 SSH 端口，默认拒绝入站 | 必装 |

## 配置选项（Web UI 表单）

`sre-scaffold` 第三步读取 `conf.yaml` 生成配置表单：

| 字段 | 类型 | 说明 |
|------|------|------|
| `linux_init_zsh` | checkbox | 是否安装 oh-my-zsh（默认勾选） |
| `zsh_theme` | 下拉框 | Zsh 主题（勾选 oh-my-zsh 后出现） |
| `zsh_plugins` | 文本输入 | Zsh 插件，空格分隔（勾选 oh-my-zsh 后出现） |

## 支持系统

- Debian / Ubuntu（netplan 配置网络）
- CentOS / RHEL / Rocky / Alma（nmcli 配置网络）

## 注意事项

- 修改静态IP可能导致 SSH 断连（网络重启在 play 结束时触发）
- 防火墙启用后默认拒绝所有入站连接，仅放行 SSH 端口
- 首次运行会备份 `/etc/ssh/sshd_config` 为 `/etc/ssh/sshd_config.bak`
