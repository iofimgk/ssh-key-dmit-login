# SSH key DMIT 登录教程全攻略：为什么连不上？密钥怎么下载？Windows/macOS/Linux 三端操作手把手带你过一遍（附套餐选购指南）

买了 DMIT 的 VPS，打开 XShell 或者 PuTTY，填好 IP、用户名 root，然后卡在密码那一步——输什么都不对。

这不是你的问题，是根本就没有密码这回事。

DMIT 所有套餐默认禁用密码登录，只接受 SSH 密钥认证。这对新手来说确实是第一道坎，但搞懂之后你会觉得其实比密码方便多了——不用记一串复杂密码，暴力破解也没有用武之地。这篇文章就是专门给第一次用 DMIT 的朋友写的，把整个 SSH key 登录流程从头到尾说清楚。

---

## SSH 密钥登录是什么，DMIT 为什么只用它

**SSH 密钥认证**的原理是一对密钥：公钥放在服务器上，私钥留在你自己手里。连接时，服务器用公钥验证你的私钥，两把钥匙对上了才放行。整个过程没有密码参与。

为什么更安全？密码可以被猜、被撞库。一个 2048 位的 RSA 私钥，穷举不现实。DMIT 从一开始就选了这套方案，不是为了给你添麻烦，是帮你省去被扫描器盯上的麻烦。

DMIT 在你创建 VPS 的时候，会自动帮你生成这对密钥，公钥直接配置到服务器上，你要做的只有一件事：把私钥文件下载下来保存好。

---

## 第一步：下载 DMIT 私钥文件

VPS 开通后，第一次进后台管理页面，会自动弹出 **SSH 金钥管理** 的对话框。

这里要注意一个很多人踩的坑：**私钥只能下载一次**。弹窗关掉之后就再也拿不回那个密钥包了——只能重新生成，然后重启 VPS 才能生效。所以第一次看到这个弹窗，不管当时在做什么，先把私钥下载下来再说。

点击"**下载私密金钥（Download Private Key）**"，会下载一个压缩包。解压之后看到两个文件夹：

- `public_key/` — 公钥，已经自动配置到 VPS 里，你不需要管它
- `private_key/` — 这是你要用的，里面有两个文件：
  - `id_rsa.pem` — 给 macOS、Linux 和 Windows OpenSSH 用
  - `id_rsa.ppk` — PuTTY 专用格式

把这两个文件放到一个你记得住的地方。

> **万一弹窗错过了怎么办**？进入 VPS 管理页面，找"SSH 金钥管理"，点"重新生成 SSH 金钥"，然后下载，之后记得在管理面板里给 VPS 执行 Reboot。

👉 [购买 DMIT VPS，开通后即可下载密钥](https://www.dmit.io/aff.php?aff=13832)

---

## 第二步：根据你的系统选工具

### macOS / Linux：直接用终端命令

这两个系统内置 OpenSSH，不需要装任何额外软件。

1. 把 `id_rsa.pem` 放到一个顺手的目录，比如 `~/.ssh/`
2. 修改文件权限（这一步不能跳，否则 SSH 会拒绝使用该密钥）：

bash
chmod 600 ~/.ssh/id_rsa.pem


3. 连接命令：

bash
ssh -i ~/.ssh/id_rsa.pem root@你的VPS_IP


第一次连接会提示是否信任服务器指纹，输入 `yes` 回车就行。

**如果嫌每次都要带 `-i` 参数麻烦**，可以把私钥配置到 `~/.ssh/config` 文件里：


Host dmit-vps
    HostName 你的VPS_IP
    User root
    IdentityFile ~/.ssh/id_rsa.pem


配置完之后，直接 `ssh dmit-vps` 就能连上，不用带参数。

---

### Windows：三种方式任选

#### 方式一：Windows 内置 OpenSSH（推荐，Win10 1803 以上都有）

打开 PowerShell 或 Windows Terminal，和 macOS 一样的命令：

powershell
ssh -i C:\Users\你的用户名\.ssh\id_rsa.pem root@你的VPS_IP


同样可以配置 `~/.ssh/config` 文件来省去每次输路径的麻烦。

#### 方式二：PuTTY

PuTTY 用的是 `.ppk` 格式，DMIT 下载包里已经包含了，直接用：

1. 打开 PuTTY，在 **Session** 页面填入 VPS 的 IP 地址，端口 22
2. 左侧导航栏点 **Connection → SSH → Auth → Credentials**
3. 在"Private key file for authentication"那里，点 Browse，选择 `id_rsa.ppk`
4. 回到 Session，保存会话名，点 Open
5. 弹出安全警告选"Accept"，用户名填 `root`

#### 方式三：XShell

XShell 支持 `.pem` 格式直接导入：

1. 菜单栏点"工具"→"用户密钥管理者"
2. 点"导入"，选择 `id_rsa.pem` 文件
3. 新建会话，填 IP 和端口 22
4. 左侧"用户身份验证"，方法选"Public Key"，用户名填"root"，密钥选刚才导入的
5. 连接，第一次弹出安全提示选"接受并保存"

---

## 第三步：连接常见报错处理

碰到连不上，先对照这几条查一查。

**报错：`Permission denied (publickey)`**

最常见的情况有三个：
- 私钥文件权限不对（macOS/Linux 必须是 600，执行 `chmod 600 id_rsa.pem`）
- 用了公钥文件而不是私钥文件（确认用的是 `private_key/` 目录下的文件）
- 密钥已重新生成但 VPS 没有重启（重启之后再试）

**报错：`Connection refused`**

端口问题。确认 SSH 端口是 22，以及 VPS 状态是"Running"而不是"Suspended"。

**报错：`WARNING: UNPROTECTED PRIVATE KEY FILE!`**

权限太宽松。`chmod 600 ~/.ssh/id_rsa.pem` 修复。

**弹窗关掉了，密钥没下载**

进 VPS 管理页面，SSH 金钥管理，重新生成，下载，重启 VPS。

---

## 进阶：上传自己的公钥替换 DMIT 默认密钥

如果你有多台服务器，或者想用同一套密钥统一管理，可以上传自己生成的公钥。

在本地生成密钥对：

bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"


一路回车，会在 `~/.ssh/` 下生成 `id_rsa`（私钥）和 `id_rsa.pub`（公钥）。

然后进 DMIT 后台，对应 VPS 的管理页面，找到"SSH Key Management"→"SSH Public Key"，把 `id_rsa.pub` 的内容粘贴进去，保存，重启 VPS。

之后直接 `ssh root@IP` 就能连，SSH 会自动用默认私钥验证，不需要每次带 `-i` 参数。

---

## DMIT 套餐选购：买什么看你用来干什么

说完登录方法，顺便把套餐的事说一下。DMIT 分三大机房（洛杉矶、香港、东京），每个机房又有 Premium / Eyeball / Tier 1 三条线路，逻辑是这样的：

| 系列 | 线路特点 | 适合场景 |
|---|---|---|
| **Premium（Pro）** | 三网 CN2 GIA 双向优化 | 对国内访问速度有要求的，建站、代理节点 |
| **Eyeball（EB）** | CMIN2 回程，联通移动更友好 | 性价比优先，联通移动用户 |
| **Tier 1（T1）** | 国际线路，无中国优化 | 面向国际业务，或当中转落地用 |

### 洛杉矶 Pro 套餐（AN4 平台）

| 套餐 | CPU | 内存 | 硬盘 | 流量 | 带宽 | 价格 |
|---|---|---|---|---|---|---|
| TINY | 1核 | 2GB | 20GB SSD | 1TB | 1Gbps | [$88.88/年](https://www.dmit.io/aff.php?aff=13832&pid=237) |
| Pocket | 2核 | 2GB | 40GB SSD | 1.5TB | 4Gbps | [$159.98/年](https://www.dmit.io/aff.php?aff=13832&pid=238) |
| STARTER | 2核 | 2GB | 80GB SSD | 3TB | 10Gbps | [$29.90/月](https://www.dmit.io/aff.php?aff=13832&pid=239) |
| MINI | 4核 | 4GB | 80GB SSD | 5TB | 10Gbps | [$58.88/月](https://www.dmit.io/aff.php?aff=13832&pid=240) |
| MICRO | 4核 | 4GB | 160GB SSD | 7TB | 10Gbps | [$74.99/月](https://www.dmit.io/aff.php?aff=13832&pid=241) |

### 洛杉矶 Eyeball 套餐（AN4 平台）

| 套餐 | CPU | 内存 | 硬盘 | 流量 | 带宽 | 价格 |
|---|---|---|---|---|---|---|
| TINY | 1核 | 2GB | 20GB SSD | 1.5TB | 2Gbps | [$88.88/年](https://www.dmit.io/aff.php?aff=13832&pid=245) |
| Pocket | 2核 | 2GB | 40GB SSD | 3TB | 4Gbps | [$159.98/年](https://www.dmit.io/aff.php?aff=13832&pid=246) |
| STARTER | 2核 | 2GB | 80GB SSD | 5TB | 10Gbps | [$29.90/月](https://www.dmit.io/aff.php?aff=13832&pid=247) |
| MINI | 4核 | 4GB | 80GB SSD | 10TB | 10Gbps | [$58.88/月](https://www.dmit.io/aff.php?aff=13832&pid=248) |

### 香港套餐（部分）

| 套餐 | 线路 | CPU | 内存 | 价格 |
|---|---|---|---|---|
| HKG.Pro.TINY | CN2 GIA | 1核 | 2GB | [$39.90/月](https://www.dmit.io/aff.php?aff=13832&pid=72) |
| HKG.T1.WEE | 国际线路 | 1核 | 1GB | [$36.9/年](https://www.dmit.io/aff.php?aff=13832) |

大多数人的选法：

- **跑代理/梯子，电信用户**：洛杉矶 Pro TINY，$88.88/年，三网 CN2 GIA 回程，够用
- **联通移动用户，注重性价比**：洛杉矶 Eyeball 系列，CMIN2 回程，比 Pro 低一档的价格
- **延迟要求极高、主要服务国内用户**：香港 Pro，贵但延迟可以压到 30ms 以内
- **流量大、不在意线路优化**：Tier 1 系列，T1 超量不关机，直接限速到一定带宽继续跑

说句实在的，DMIT 在这个圈子里的评价基本是"除了贵没有缺点"。晚高峰 CN2 GIA 线路延迟稳在 150-180ms 上下，丢包极少，这在同价位里确实难找对手。觉得价格还能接受的话，这个稳定性是有保障的。

👉 [查看 DMIT 全部套餐与最新价格](https://www.dmit.io/aff.php?aff=13832)

---

## FAQ：SSH key 登录 DMIT 的常见疑问

**Q：登录用户名是什么？**  
默认是 `root`。DMIT 没有强制改初始用户名，直接用 root 登录就行。

**Q：SSH 端口是多少？**  
默认 22。如果你自己改过 sshd 配置就另说，初始状态是 22。

**Q：私钥文件丢了能找回吗？**  
找不回来了。DMIT 生成的密钥只能下载一次，丢了只能去后台重新生成，然后重启 VPS。新密钥生成后旧的就作废了。

**Q：能不能开启密码登录？**  
可以，但不建议。要开的话，先用密钥连上去，然后修改 `/etc/ssh/sshd_config`，把 `PasswordAuthentication no` 改成 `yes`，重启 SSH 服务。但这样就暴露在密码扫描器下了，至少要设个够复杂的密码。

**Q：私钥文件能在多台电脑上用吗？**  
可以，把 `id_rsa.pem` 复制过去就行，权限记得改成 600。或者用前面说的"上传自己公钥"的方法，在每台电脑上生成各自的密钥对，把公钥都上传到 DMIT，更安全。

**Q：DMIT 支持哪些支付方式？**  
支付宝、微信、PayPal、信用卡都支持。国内用户用支付宝最方便。

---

整个流程就这些。SSH key 登录对新手来说确实有点绕，但走一遍之后基本就形成肌肉记忆了。后台下私钥、改权限、带 `-i` 参数连接——三步。

如果还没买 DMIT，下面这个链接是当前套餐入口，按需选就行。

👉 [前往 DMIT 选择适合你的套餐方案](https://www.dmit.io/aff.php?aff=13832)
