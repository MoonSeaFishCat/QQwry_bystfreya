# 纯真 IP 库  [![build](https://github.com/nmgliangwei/qqwry/actions/workflows/newqqwry.yml/badge.svg)](https://github.com/nmgliangwei/qqwry/actions/workflows/newqqwry.yml)

> 🎓 **圣芙蕾雅学院·数据情报科 教学记录**  
> 『各位学员注意啦！今天的课程是关于 IP 地址情报系统的～』  
> —— 姬子老师

## 📚 项目简介

**纯真dat库文件恢复更新，使用 CZDB 重新制作，不支持 IPV6，如需要 IPV6 请使用下面 IPDB格式的库文件。**

**兼容ipip.net的IPDB格式并支持 IPV4/IPV6 的库文件，请关注[nmgliangwei/qqwry.ipdb](https://github.com/nmgliangwei/qqwry.ipdb)。**

**本仓库如果有帮助到您的话可以点点 [Star⭐](https://github.com/nmgliangwei/qqwry)支持下。**

---

## 🌸 琪亚娜的快速理解版

<details>
<summary>👉 点击展开琪亚娜的傻瓜式解释</summary>

**琪亚娜**：「呜哇，这么多专业术语，脑袋疼～让本小姐用人话翻译一下！」

### 🔍 这个项目到底是干啥的？

想象一下，互联网上的每台电脑都有一个"门牌号"，叫做 IP 地址（比如 `192.168.1.1`）。但是这些数字很难记对吧？

这个项目就像一本**超级大字典**，可以告诉你：
- 📍 这个 IP 地址在哪个城市？
- 🏢 是哪个网络运营商的？
- 🌍 属于哪个国家？

### ⚙️ 它是怎么工作的？

**布洛妮娅**：「用布洛妮娅的话来说，就是三个步骤：」

```
1. 下载情报 📥
   ↓ (从纯真网络获取最新的 IP 数据库)
   
2. 格式转换 🔄
   ↓ (把新格式 CZDB 转成经典的 DAT 格式)
   
3. 自动发布 🚀
   ↓ (每天自动检查更新，有新版本就发布)
```

**芽衣**：「简单来说，就像我每天准备便当一样，这个系统每天自动更新 IP 数据库～」

### 🎯 为什么需要两个 Token？

**德丽莎学园长**：「咳咳，让学园长来解释一下权限系统！」

- **`DOWNLOAD_TOKEN`** 🔑  
  就像学园的门禁卡，有了它才能从纯真官网下载最新的数据库文件。
  
- **`CZDB_TOKEN`** 🗝️  
  就像解密密码本的钥匙，有了它才能打开和读取下载的数据文件内容。

> 💡 **获取方式**：前往 [纯真官网](https://cz88.net/) 申请免费的社区版授权～

### 📊 数据统计

**符华**：「根据最新记录显示——」

- 📈 IP 记录总数：约 **151.7 万**条
- 🌐 覆盖地理位置：约 **17.2 万**个
- 🔄 更新频率：约每周一次
- ⏰ 自动检查：每天 5 次（UTC时间 2:10, 5:10, 9:10, 12:10, 15:10）

</details>

---

## 🔬 技术原理深度解析

<details>
<summary>👩‍🔬 展开查看爱因斯坦博士的技术笔记</summary>

### 🧪 核心技术架构

**爱因斯坦**：「让我来详细解释这个系统的工作原理～」

#### 1️⃣ 数据采集模块 (`src/build.js`)

```javascript
// 步骤 A: 数据下载
const download = async () => {
  // 通过纯真官方 API 获取 CZDB 格式数据库
  // CZDB = CZ Database (纯真数据库格式)
}

// 步骤 B: 数据解析与转换
const extract = async () => {
  // 使用 @ipdb/czdb 解码器读取 CZDB 文件
  // 过滤 IPv6，只保留 IPv4 记录
  // 分离地理位置(geo)和运营商(isp)信息
  // 调用 Packer 生成 DAT 二进制文件
}

// 步骤 C: 版本管理
const release = async () => {
  // 从特殊 IP (255.255.255.255) 提取版本号
  // 统计记录数和唯一地理位置数
  // 更新版本历史文件
}
```

#### 2️⃣ 二进制打包器 (`src/packer.js`)

**DAT 文件结构**：

```
┌──────────────────────────────────────────────┐
│  文件头 (Header, 8 bytes)                     │
│  ┌─────────────┬──────────────┐              │
│  │ 索引区起始   │ 索引区结束    │              │
│  └─────────────┴──────────────┘              │
├──────────────────────────────────────────────┤
│  记录区 (Record Area, 变长)                   │
│  ┌──────┬──────┬──────┐                      │
│  │ EndIP│ Geo  │ ISP  │ ...                  │
│  └──────┴──────┴──────┘                      │
├──────────────────────────────────────────────┤
│  索引区 (Index Area, 变长)                    │
│  ┌────────┬────────┐                         │
│  │StartIP │Offset  │ (7 bytes per record)   │
│  │StartIP │Offset  │                         │
│  │  ...   │  ...   │                         │
│  └────────┴────────┘                         │
└──────────────────────────────────────────────┘
```

**优化技术**：

1. **字符串缓存** 📦  
   相同的地理位置信息只存储一次，其他位置通过指针引用，大幅减少文件大小。

2. **重定向机制** 🔀  
   - `0x01` 模式：完整记录重定向
   - `0x02` 模式：单字段重定向

3. **编码方式** 📝  
   使用 GBK 编码保证与旧版软件兼容性。

#### 3️⃣ 自动化流水线 (GitHub Actions)

```mermaid
触发条件 → 下载数据 → 格式转换 → 版本检查 → 创建Release → 提交更新
   ↓           ↓          ↓          ↓           ↓          ↓
手动/定时/Push  wget     Packer    Tag存在?    GitHub    Git Push
```

</details>

---

## 📦 数据文件下载

|数据文件|说明|是否推荐|国内加速下载链|历史数据|
|:---:|---|---|---|---|
|`qqwry.dat`|dat 数据文件|✅ 推荐|[qqwry.dat](https://raw.gitmirror.com/nmgliangwei/qqwry/main/qqwry.dat)|[releases](https://github.com/nmgliangwei/qqwry/releases)|

---

## 🚀 快速开始

### 方法一：直接下载使用

```bash
# 下载最新版本
wget https://raw.gitmirror.com/nmgliangwei/qqwry/main/qqwry.dat

# 或使用 curl
curl -O https://raw.gitmirror.com/nmgliangwei/qqwry/main/qqwry.dat
```

### 方法二：本地构建

**丽塔**：「想自己动手的话，按照这个流程来～」

```bash
# 1. 克隆仓库
git clone https://github.com/nmgliangwei/qqwry.git
cd qqwry

# 2. 安装依赖
pnpm install

# 3. 配置环境变量
export DOWNLOAD_TOKEN="你的下载Token"
export CZDB_TOKEN="你的解析Token"

# 4. 执行构建
pnpm run build

# 5. 生成的文件在 ./dist/qqwry.dat
```

---

## 🔐 Token 获取指南

**姬子老师的授权课程**：

### 步骤 1：访问官网
前往 [纯真网络官网](https://cz88.net/)

### 步骤 2：申请授权
- 📝 注册账号
- 📋 填写申请表单
- ⏳ 等待审核通过

### 步骤 3：获取 Token
- 🔑 `DOWNLOAD_TOKEN` - 数据下载授权
- 🗝️ `CZDB_TOKEN` - 数据解析授权

### 步骤 4：配置到项目

如果要在 GitHub Actions 中使用，需要配置 Secrets：

```
仓库设置 → Settings → Secrets and variables → Actions → New repository secret
```

添加以下密钥：
- `DOWNLOAD_TOKEN` 
- `CZDB_TOKEN`
- `GIT_USERNAME`
- `GIT_EMAIL`
- `QQWRY` (GitHub Personal Access Token)

---

## 🎓 关于纯真社区版 IP 库

**德丽莎学园长的官方说明**：

纯真(CZ88.NET)自2005年起一直为广大社区用户提供社区版IP地址库，只要获得纯真的授权就能**免费使用**，并不断获取后续更新的版本。如果有需要免费版IP库的朋友可以前往纯真的官网进行申请。

纯真除了免费的社区版IP库外，还提供数据更加准确、服务更加周全的商业版IP地址查询数据。纯真围绕IP地址，基于 **网络空间拓扑测绘 + 移动位置大数据** 方案，对IP地址定位、IP网络风险、IP使用场景、IP网络类型、秒拨侦测、VPN侦测、代理侦测、爬虫侦测、真人度等均有近20年丰富的数据沉淀。

---

## 🎖️ 项目特点

> **渡鸦的战术分析**：「这个项目的优势在于——」

- ⚡ **全自动化** - GitHub Actions 定时任务，无需人工干预
- 🔄 **高频更新** - 每天检查 5 次，确保数据及时性
- 💾 **格式兼容** - 支持经典 DAT 格式，兼容老软件
- 📊 **版本追踪** - 完整的历史版本记录和统计信息
- 🚀 **快速部署** - 开箱即用，配置简单

---

## 📖 使用示例

### Node.js 中使用

```javascript
import libqqwry from 'lib-qqwry'

// 加载数据库
const qqwry = libqqwry(true, './qqwry.dat')

// 查询 IP
const result = qqwry.searchIP('8.8.8.8')
console.log(result)
// 输出: { Country: '美国', Area: 'Google' }
```

### Python 中使用

```python
from qqwry import QQwry

# 加载数据库
q = QQwry()
q.load_file('./qqwry.dat')

# 查询 IP
result = q.lookup('8.8.8.8')
print(result)
# 输出: ('美国', 'Google')
```

---

## 🤝 致谢

**全体学员的感谢名单**：

- 💙 感谢 [FW27623](https://github.com/FW27623)/[qqwry](https://github.com/FW27623/qqwry) - 由于fork出现一些问题导致异常所以单独建了项目
- 🌟 感谢 [纯真网络](https://cz88.net/) 提供免费的社区版 IP 数据库
- 🎯 感谢所有为项目点 Star 的小伙伴们

---

## 📜 License

MIT License

---

<div align="center">

**🌸 圣芙蕾雅学院·数据情报科 出品 🌸**

*「为了美好的网络世界～Fighting！」*

[![Star History Chart](https://api.star-history.com/svg?repos=nmgliangwei/qqwry&type=Date)](https://star-history.com/#nmgliangwei/qqwry&Date)

</div>
