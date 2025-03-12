# RackNerd VPS 重装 AlmaLinux - 完整教程

## 系统重装 AlmaLinux 的选择与步骤

### 可安装的操作系统
在 RackNerd 的 VPS 中，您可以选择安装以下操作系统：

- AlmaLinux
- CentOS
- Debian
- Fedora
- Rocky Linux
- Ubuntu

由于 AlmaLinux 排在列表首位，我们选择了它。关于 AlmaLinux 和 Debian的选择，我们通过 GPT-4 得到了以下建议：

> 如果您更注重稳定性、与 CentOS 的兼容性和不断增长的社区支持，AlmaLinux 可能是更合适的选择。而如果您更看重软件的新颖性和更广泛的软件包选择，那么 Debian 可能更适合您。

AlmaLinux 目前支持两个版本：AlmaLinux 8 和 AlmaLinux 9。在完成系统和版本选择后，点击 **Reinstall** 按钮即可开始重装。

👉 [【建议收藏】2025年Racknerd最新优惠套餐整理汇总 - 每日更新可用活动优惠](https://bit.ly/Rack_Nerd)

完成系统重装后，当尝试通过 SSH 登录时可能会遇到如下提示：
由于新的系统安装过程会更改远程服务器的指纹，本地保存的旧指纹信息会导致登录失败。解决方案如下：
1. 打开本地电脑的 `~/.ssh/known_hosts` 文件。
2. 删除对应远程服务器的旧指纹信息。

完成以上步骤后，即可成功登录新系统。

---

## 系统初始化配置

### 修改主机名
首先查看当前系统信息：
shell
hostnamectl


然后执行以下命令修改主机名：
shell
hostnamectl set-hostname my_host_name


### 禁止 ICMP 协议
为提高系统的安全性，可以禁止 ICMP 协议。执行以下步骤：
shell
echo "net.ipv4.icmp_echo_ignore_all = 1" >> /etc/sysctl.conf
sysctl -p

此设置将阻止外界通过 Ping 测试该 VPS 的 IP 地址。

### 关闭 SELinux
如果不需要使用 SELinux的安全策略，可以通过如下命令禁用它：
shell
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config


### 修改 SSH 端口
为减少安全风险，可以更改 SSH 端口。例如将端口设置为 1234：

1. 确定端口是否被占用：
   shell
   yum -y install lsof
   lsof -i:1234
   

2. 开放新的端口：
   shell
   firewall-cmd --add-port=1234/tcp --permanent
   firewall-cmd --reload
   

3. 编辑 SSH 配置文件：
   shell
   vi /etc/ssh/sshd_config
   
   将 `#Port 22` 修改为 `Port 1234`，然后重启 SSH 服务：
   shell
   systemctl restart sshd
   
此后连接 SSH 时需使用新端口。

---

## 用户与权限管理

### 创建新用户并赋予权限
创建一个新用户并设置密码：
shell
adduser username
passwd username


赋予新用户管理员权限：
shell
usermod -aG wheel username


### 禁止 root 用户远程登录
修改 SSH 配置文件：
shell
vi /etc/ssh/sshd_config

将 `PermitRootLogin yes` 改为 `PermitRootLogin no`，随后重启 SSH 服务：
shell
systemctl restart sshd


---

## 网络优化配置

### 启用 TCP BBR
TCP BBR 是一种拥塞控制算法，能够显著提高网络性能。在 RackNerd VPS 上，通常默认情况下已启用 BBR，无需手动设置。

### 设置 SSH 无密码登录
通过以下命令将公钥传输到远程服务器：
shell
ssh-copy-id -p 1234 username@server-ip


---

## 实用安装指南

### 安装基础工具
以下命令可用于快速安装开发相关的工具：
shell
sudo dnf clean all
sudo dnf update
sudo dnf groupinstall "Development Tools"
sudo yum makecache --refresh
sudo yum -y install wget git zsh tar util-linux-user lua


### 安装 Tailscale
使用以下命令安装轻量级 VPN 工具 Tailscale：
shell
curl -fsSL https://tailscale.com/install.sh | sh


### 安装 FZF
FZF 是一款快速预览文件的工具：
shell
sudo dnf install epel-release
sudo dnf install fzf
fzf --version


### 安装 Neovim
Neovim 是现代化的文本编辑器，具体安装请参考[如何在 AlmaLinux 中安装 Neovim](https://hahagood.com/post/2024/03/20240303121914-use_epel_repo_for_almalinux/)。

---

## Zsh 配置说明

### 终端配置
打开 `$HOME/.zshrc` 文件，并添加以下内容：
zsh
export TERM=xterm-256color

设置 Zsh 为默认终端：
shell
sudo chsh -s /bin/zsh


### 安装 PowerLevel10K 主题
使用以下命令安装 PowerLevel10K：
shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

配置主题：
zsh
ZSH_THEME="powerlevel10k/powerlevel10k"

重启 Zsh 后，即可完成 PowerLevel10K 的初始化。

如果需要重新配置主题，可以运行以下命令：
shell
p10k configure


完成所有安装和配置后，您的 RackNerd VPS 即可投入使用！