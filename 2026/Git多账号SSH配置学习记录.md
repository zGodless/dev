# Git 多账号 SSH 配置与协作者拉取私有仓库学习记录

> 日期：2026-09-23
> 环境：Windows + Git Bash（Git for Windows，OpenSSH 10.3p1）+ TortoiseGit + Clash 类代理（TUN / fake-ip，本地混合端口 7897），Windows 用户名含中文

---

## 一、目标

1. 在同一台电脑上，让两个 GitHub 账号各自使用独立的 SSH 密钥，互不干扰：
   - 账号 A：用户名 `zGodless`（邮箱 952715899@qq.com）
   - 账号 B：用户名 `zGodless-kkkkk`（邮箱 zgodless1035@gmail.com）
2. 以协作者身份（账号 B）拉取他人（Y4er）的私有仓库 `C137`。

## 二、多账号 SSH 的核心原理

Git 走 SSH 时，URL 的写法是：

```
git@主机别名:仓库所有者/仓库名.git
     │          └── 决定"访问哪个仓库"（照抄 GitHub 上的真实路径）
     └── 决定"以哪个身份连接"（对应 ~/.ssh/config 里的 Host 别名）
```

在 `~/.ssh/config` 里给每个账号定义一个**别名**，每个别名绑定各自的私钥：

| config 中的 Host 别名 | 实际登录的 GitHub 用户名 | 私钥 |
|---|---|---|
| `github.com-account_9527` | zGodless | `id_rsa_account_9527` |
| `github.com-account_1035` | zGodless-kkkkk | `id_rsa_account_1035` |

三条铁律：

- **别名决定身份，冒号后的路径决定仓库**，两者互相独立。
- **每个公钥只能绑定一个 GitHub 账号**，两个账号必须用各自的公钥。
- **user.email 只决定提交署名，与拉取权限无关**。能不能拉私有仓库，看的是你用哪把钥匙（哪个别名）连接。

## 三、最终可用方案（速查）

### 1. 生成两把密钥（邮箱写在注释里）

```bash
ssh-keygen -t ed25519 -C "952715899@qq.com" -f ~/.ssh/id_rsa_account_9527
ssh-keygen -t ed25519 -C "zgodless1035@gmail.com" -f ~/.ssh/id_rsa_account_1035
```

把两个 `.pub` 公钥分别添加到对应 GitHub 账号：网页 → Settings → SSH and GPG keys → New SSH key。

### 2. `~/.ssh/config`（最终版）

```ssh-config
Host github.com-account_9527
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile /c/Users/老爷亮/.ssh/id_rsa_account_9527
    IdentitiesOnly yes
    UserKnownHostsFile /c/Users/老爷亮/.ssh/known_hosts
    ProxyCommand connect -H 127.0.0.1:7897 %h %p

Host github.com-account_1035
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile /c/Users/老爷亮/.ssh/id_rsa_account_1035
    IdentitiesOnly yes
    UserKnownHostsFile /c/Users/老爷亮/.ssh/known_hosts
    ProxyCommand connect -H 127.0.0.1:7897 %h %p
```

要点：

- `HostName ssh.github.com` + `Port 443`：GitHub 官方的 443 端口 SSH 入口，绕开被墙的 22 端口。
- `ProxyCommand connect -H 127.0.0.1:7897 %h %p`：强制 SSH 走本地代理端口，比依赖 TUN 的 fake-ip 更稳定（`connect` 是 Git for Windows 自带工具）。
- **IdentityFile / UserKnownHostsFile 必须写绝对路径，不能写 `~`**（原因见坑 5）。

### 3. 让 git 强制读取该配置

```bash
git config --global core.sshCommand "ssh -F C:/Users/老爷亮/.ssh/config"
```

（用 Windows 风格绝对路径，保证 Git Bash 和 TortoiseGit 两种环境下都能找到。）

### 4. TortoiseGit 设置

右键 → TortoiseGit → Settings → Network → SSH client，改为 Git 自带的 OpenSSH：

```
C:\Program Files\Git\usr\bin\ssh.exe
```

**不能用默认的 TortoiseGitPlink.exe**，它不认 `~/.ssh/config`。

### 5. 拉取私有仓库（协作者场景）

以 zGodless-kkkkk 身份拉取 Y4er 的私有仓库 C137，TortoiseGit 克隆 URL 填：

```
git@github.com-account_1035:Y4er/C137.git
```

即：**别名用自己账号对应的（1035），路径照抄仓库所有者的（Y4er/C137）**。

---

## 四、踩坑全过程（问题 → 现象 → 原因 → 解决）

### 坑 1：22 端口连不上，`Connection closed by 198.18.0.x port 22`

**现象**：`ssh -T git@github.com` 报 `Connection closed by 198.18.0.131 port 22`。

**排查**：`nslookup github.com` 返回 `198.18.0.20`。`198.18.0.0/15` 是保留测试网段，GitHub 真实 IP 不可能是它——这是 Clash 类代理 TUN 模式的 **fake-ip**：DNS 被代理接管，22 端口流量进了代理后被上游关闭（GitHub 的 22 端口在国内网络经常不可用）。

**解决**：改用 GitHub 官方 443 端口 SSH 入口 `ssh.github.com:443`，并加 `ProxyCommand` 显式走本地代理端口。

**教训**：看到 `198.18.x.x` 就要想到代理 fake-ip，先怀疑网络层，不要急着查密钥。

### 坑 2：主机别名里带了 `@`，结果连到了 qq.com

**现象**：执行

```bash
ssh -T git@github.com-account_952715899@qq.com
```

日志显示 `Connecting to qq.com [198.18.0.21] port 22`，随后连接被关闭。

**原因**：OpenSSH 解析 `user@host` 时，以**最后一个 `@`** 为分界。于是这条命令被拆成：

- 用户名：`git@github.com-account_952715899`
- 主机：`qq.com`

SSH 连到了腾讯的 qq.com:22，对方不是 SSH 服务器，直接掐断连接。

**解决**：**主机别名里绝对不能包含 `@`**。邮箱应该写在密钥注释里（`ssh-keygen -C "邮箱"`），不要出现在别名里。

### 坑 3：密钥文件名与 config 不一致

**现象**：config 里写 `IdentityFile ~/.ssh/id_rsa_account_9527`，但实际文件叫 `id_rsa_account_952715899@qq.com`。

**解决**：统一改名（文件名里的 `@` 在 bash 中不需要转义）：

```bash
mv ~/.ssh/id_rsa_account_952715899@qq.com ~/.ssh/id_rsa_account_9527
mv ~/.ssh/id_rsa_account_952715899@qq.com.pub ~/.ssh/id_rsa_account_9527.pub
mv ~/.ssh/id_rsa_account_zgodless1035@gmail.com ~/.ssh/id_rsa_account_1035
mv ~/.ssh/id_rsa_account_zgodless1035@gmail.com.pub ~/.ssh/id_rsa_account_1035.pub
```

### 坑 4：ssh 静默不读 `~/.ssh/config`（中文用户名编码问题，本次最大的坑）

**现象**：`ssh -vT` 日志里只有 `Reading configuration data /etc/ssh/ssh_config`（系统配置），没有用户配置那一行；`Port 443` 完全不生效，仍然连 22 端口。

**诊断（关键两招，不联网即可判断）**：

```bash
# 自动加载：显示 port 22 → 自动加载失败
ssh -G github.com-account_9527 | grep -Ei '^(hostname|port|user|identityfile) '
