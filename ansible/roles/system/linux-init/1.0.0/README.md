# linux-init

一键初始化全新 Linux 服务器：网络配置、SSH 加固、常用工具安装、oh-my-zsh、防火墙。

## 快速开始

```bash
# 1. 复制并编辑 inventory
cp inventory.example inventory
vim inventory

# 2. Dry-run 检查
ansible-playbook -i inventory init.yaml --check

# 3. 正式执行
ansible-playbook -i inventory init.yaml
```

## inventory 配置

```ini
[servers]
my-server ansible_host=192.168.1.100 ansible_user=root

[servers:vars]
# 以下为必填项
static_ip=192.168.1.100/24
gateway=192.168.1.1
dns_servers=['192.168.1.1', '8.8.8.8']

# 以下为可选项
# hostname=my-custom-hostname
# ssh_port=22
# zsh_theme="bira"
```

## 支持系统

- Debian / Ubuntu（netplan 配置网络）
- CentOS / RHEL / Rocky / Alma（nmcli 配置网络）

## 功能清单

| 角色 | 说明 |
|------|------|
| `network` | 设置主机名、静态IP、DNS |
| `ssh-hardening` | 备份 sshd_config，设置 PermitRootLogin prohibit-password |
| `base-packages` | 安装常用工具：tree, vim, curl, wget, git, htop, lsof, rsync, tcpdump, jq, tar |
| `oh-my-zsh` | 安装 zsh + oh-my-zsh，bira 主题，zsh-autosuggestions + zsh-syntax-highlighting 插件 |
| `firewall` | Debian→ufw，RHEL→firewalld，放行 SSH 端口，默认拒绝入站 |

## 选择性执行

通过 `conf.yaml` 中的开关变量可跳过部分 role。将对应变量设为 `false` 即可：

```yaml
# conf.yaml
linux_init_network: true
linux_init_ssh: true
linux_init_packages: true
linux_init_zsh: true      # 改为 false 则跳过 oh-my-zsh
linux_init_firewall: true  # 改为 false 则跳过防火墙
```

也可在命令行指定：

```bash
ansible-playbook -i inventory init.yaml -e "linux_init_zsh=false"
```

## 配置表单

Web UI 读取 `conf.yaml` 生成配置表单：
- Boolean 变量 → checkbox（功能开关）
- String 变量 → 文本输入框
- List 变量 → 多行文本

## 自定义

变量优先级：`extra-vars > inventory > group_vars > conf.yaml > role defaults`。

所有可配置变量集中在 `conf.yaml`，各 role 的 `defaults/main.yaml` 提供兜底默认值。

## 注意事项

- 修改静态IP可能导致 SSH 断连（网络重启在 play 结束时触发）
- 防火墙启用后默认拒绝所有入站连接，仅放行 SSH 端口
- 首次运行会备份 `/etc/ssh/sshd_config` 为 `/etc/ssh/sshd_config.bak`
