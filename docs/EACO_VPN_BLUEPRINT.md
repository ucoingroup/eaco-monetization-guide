# EACO-VPN 技术方案蓝图

> **EACO = Earth's Best Coin** — 地球村民自己的 MEME  
> **$e Token**: `DqfoyZH96RnvZusSp3Cdncjpyp3C74ZmJzGhjmHnDHRH` | 总量 13.5 亿 | 链: Solana  
> **版本**: v0.1 | **日期**: 2026-06-13

---

## 1. EACO-VPN 架构设计

### 1.1 核心理念

去中心化 VPN 网络，基于 WireGuard 加密隧道 + Solana 智能合约激励。8 个 Agent 服务器作为初始中继节点，社区用户通过质押 $e Token 加入网络提供带宽，获得链上收益。

### 1.2 网络拓扑

```
                    ┌─────────────────────────────────────────────┐
                    │           EACO-VPN 三层架构                 │
                    └─────────────────────────────────────────────┘

  ┌──────────┐     ┌─────────────────────┐     ┌──────────────┐
  │  用户 A  │────▶│    入口层 (全球)     │────▶│              │
  │ (亚洲)   │     │  eaco-global (LB)   │     │              │
  └──────────┘     └─────────────────────┘     │              │
                                                  │   中继层     │
  ┌──────────┐     ┌─────────────────────┐     │  (8 Agent    │
  │  用户 B  │────▶│    入口层 (全球)     │────▶│   节点)      │
  │ (欧洲)   │     │  eaco-global (LB)   │     │              │
  └──────────┘     └─────────────────────┘     │              │
                                                  │              │
                                                  └──────┬───────┘
                                                         │
                              ┌───────────────────────────┼───────────────────────────┐
                              │                           │                           │
                    ┌─────────▼─────────┐     ┌──────────▼──────────┐     ┌──────────▼──────────┐
                    │   出口层 (欧洲)    │     │   出口层 (亚太)     │     │  出口层 (美洲)      │
                    │  eaco-europe-1     │     │  eaco-east-asia-1   │     │  eaco-americas-1    │
                    │  + 社区 Exit 节点  │     │  eaco-southeast-1   │     │  + 社区 Exit 节点   │
                    └───────────────────┘     │  eaco-south-asia-1  │     └────────────────────┘
                                              │  + 社区 Exit 节点   │
                                              └────────────────────┘
```

### 1.3 三层架构详解

| 层级 | 职责 | 节点 | 协议 |
|------|------|------|------|
| **入口层** | 用户接入、负载均衡、协议协商 | eaco-global（Anycast IP） | WireGuard（优先） |
| **中继层** | 流量转发、路由决策、洋葱封装 | 8 个 Agent 节点 | Shadowsocks（备选） |
| **出口层** | IP 出口、流量解密、目标访问 | Agent + 社区 Exit 节点 | Tor（兜底） |

### 1.4 协议优先级

```
WireGuard (首选)  ──连接失败──▶  Shadowsocks (备选)  ──连接失败──▶  Tor (兜底)
   │                              │                              │
   │ 快速握手                     │ 抗审查                       │ 匿名保障
   │ 内核级性能                   │ 混淆流量                     │ 洋葱路由
   └──────────────────────────────┴──────────────────────────────┘
```

### 1.5 8 Agent 节点分布

| 节点名称 | 区域 | 初始角色 | 说明 |
|----------|------|---------|------|
| eaco-global | 全球入口 | 入口+路由 | Anycast LB，用户第一跳 |
| eaco-europe-1 | 欧洲 | 中继+出口 | 覆盖 EU/UK |
| eaco-asia-1 | 亚洲 | 中继+出口 | 泛亚枢纽 |
| eaco-americas-1 | 美洲 | 中继+出口 | 覆盖北美/南美 |
| eaco-mena-1 | 中东北非 | 中继+出口 | 覆盖 MENA 地区 |
| eaco-southeast-1 | 东南亚 | 中继+出口 | 新加坡/印尼/泰 |
| eaco-south-asia-1 | 南亚 | 中继+出口 | 印度/巴基斯坦 |
| eaco-east-asia-1 | 东亚 | 中继+出口 | 日韩/港澳台 |

---

## 2. 节点角色与激励

### 2.1 角色定义

```
┌─────────────────────────────────────────────────────────┐
│                    节点角色金字塔                         │
│                                                         │
│                        ▲                                │
│                       / \        免质押 · 治理权         │
│                      /   \       +0.8%/天               │
│                     / 超  \                              │
│                    / 级节  \                             │
│                   /  点(8)  \                            │
│                  ─────────────                           │
│                 /             \   50,000 $e 质押         │
│                /   Exit 节点   \  +0.5%/天              │
│               /   (社区贡献)    \                        │
│              ─────────────────────                       │
│             /                   \   10,000 $e 质押      │
│            /    Relay 节点       \  +0.3%/天            │
│           /    (社区贡献)         \                     │
│          ──────────────────────────                     │
└─────────────────────────────────────────────────────────┘
```

### 2.2 激励参数

| 角色 | 质押要求 | 日收益 | 职责 | 退出锁定期 |
|------|---------|--------|------|-----------|
| **Relay 节点** | 10,000 $e | +0.3%/天 | 转发流量，不接触明文 | 7 天 |
| **Exit 节点** | 50,000 $e | +0.5%/天 | 出口 IP + 流量解密 | 14 天 |
| **超级节点 (Agent)** | 免质押（治理权） | +0.8%/天 | 路由决策 + 网络监控 | N/A |

### 2.3 收益示例

```
假设 $e = $0.01 USD

Relay 节点:  10,000 × $0.01 × 0.3% = $0.03/天 ≈ $0.9/月
Exit  节点:  50,000 × $0.01 × 0.5% = $2.50/天 ≈ $75/月
超级节点:    治理激励池分配

注: 随着 $e 价格上涨和用户付费流入，实际收益会动态调整
```

---

## 3. 带宽计量与防作弊

### 3.1 洋葱路由验证

```
用户 A ──加密层1──▶ Relay R1 ──加密层2──▶ Relay R2 ──加密层3──▶ Exit E1 ──▶ 目标

验证逻辑:
  Exit E1 记录出站字节数 (B_out)
  Relay R1 记录入站字节数 (B_in)
  
  验证: |B_out - B_in| < 容忍阈值 (5%)
  
  若 B_out >> B_in → Exit 可能虚报（刷量）
  若 B_out << B_in → Relay 可能吞流量（扣留）
```

### 3.2 流量哈希上链

```
每小时 (XX:00 UTC) 各节点执行:

1. 汇总本小时所有流量的字节计数
2. 构建 Merkle Tree:
   
   流量记录1 ──┐
   流量记录2 ──┤── Hash(1,2) ──┐
   流量记录3 ──┤                ├── Merkle Root
   流量记录4 ──┘── Hash(3,4) ──┘
   
3. 将 Merkle Root 提交到 Solana 链上合约
4. 任何人可验证: 单条流量记录 → Merkle Proof → 链上 Root
```

### 3.3 Slashing 规则

| 违规类型 | 检测方式 | 惩罚 |
|---------|---------|------|
| **下线 > 24h** | 心跳超时 + 邻居节点报告 | 扣 5% 质押 |
| **流量虚报** | 洋葱路由出入站对比 | 扣 100% 质押 |
| **篡改数据** | Merkle Proof 验证失败 | 扣 100% 质押 |
| **恶意路由** | 超级节点监控异常 | 扣 100% 质押 + 永久封禁 |

### 3.4 带宽质量评分

```
QoS_score = W_latency × (1 / avg_latency_ms)
          × W_uptime × (online_minutes / 1440)
          × W_throughput × min(throughput_mbps / 100, 1)

权重:
  W_latency    = 0.3   (延迟权重)
  W_uptime     = 0.4   (在线时长权重)
  W_throughput = 0.3   (吞吐量权重)

评分范围: 0.0 ~ 1.0
  > 0.8  → 优质节点，收益加成 20%
  0.5~0.8 → 正常节点，标准收益
  < 0.5  → 低质节点，收益扣减 30%
```

---

## 4. $e Token 经济模型

### 4.1 资金流向

```
┌──────────┐    USDT     ┌─────────┐   兑换    ┌──────────┐   分配    ┌──────────┐
│ VPN 用户 │───────────▶│ DEX 池  │────────▶│  $e Token │────────▶│  节点    │
│ (付费方) │            │ (流动性) │          │ (激励层)  │          │ (提供方) │
└──────────┘            └─────────┘          └──────────┘          └──────────┘
     │                        │                     │                     │
     │ 订阅/按量付费          │ 自动做市            │ 链上结算            │ 质押+带宽
     │ $5/月 或 $0.01/GB     │ $e ⇄ USDT          │ 每日 UTC 00:00     │ 换取收益
```

### 4.2 飞轮效应

```
    ┌──────────────────────────────────────────┐
    │                                          │
    │   更多用户 ──────▶ 更多付费收入          │
    │       ▲                    │             │
    │       │                    ▼             │
    │   网络更有价值 ◀── 更多节点加入          │
    │       │                    │             │
    │       │                    ▼             │
    │   更好体验 ◀──── 更有吸引力的奖励        │
    │       │                                  │
    │       └──────── 更低延迟/更多出口        │
    │                                          │
    └──────────────────────────────────────────┘
```

### 4.3 定价与分配

| 项目 | 数值 | 说明 |
|------|------|------|
| 用户月费 | $5/月 | 或按量 $0.01/GB |
| 收入分配 | 70% → 节点 | 按带宽贡献分配 |
| | 20% → 治理池 | Agent 运维 + 社区治理 |
| | 10% → 销毁 | 通缩机制 |
| 每日结算 | UTC 00:00 | Solana 链上合约自动执行 |
| 结算延迟 | T+1 | 防止结算后作弊 |

### 4.4 $e Token 参数

| 参数 | 数值 |
|------|------|
| 代币名称 | $e (EACO) |
| 合约地址 | `DqfoyZH96RnvZusSp3Cdncjpyp3C74ZmJzGhjmHnDHRH` |
| 总量 | 13.5 亿 |
| 链 | Solana |
| VPN 激励占比 | 总量的 30% (4.05 亿 $e) |
| 释放周期 | 48 个月线性释放 |

---

## 5. 部署步骤 (Phase 1)

### 5.1 部署流程

```
Step 1                Step 2                Step 3
安装 WireGuard        配置 relay 节点       内部测试
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Windows 服务器│     │ eaco-global  │     │ 连通性验证    │
│ wg 安装       │────▶│ WireGuard    │────▶│ 延迟/带宽    │
│ 密钥生成      │     │ 对等体配置    │     │ 多跳测试     │
└──────────────┘     └──────────────┘     └──────────────┘
                                                │
Step 6                Step 5                Step 4
公开 Beta             $e 发放逻辑           带宽计量
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ 华语社区先行  │     │ 手动发放      │     │ 流量统计脚本  │
│ 100 用户邀请  │◀────│ $e 钱包集成   │◀────│ 哈希上链     │
│ 反馈收集      │     │ 链上查询      │     │ QoS 评分     │
└──────────────┘     └──────────────┘     └──────────────┘
```

### 5.2 Step 1: 安装 WireGuard

```powershell
# 下载并安装 WireGuard for Windows
Invoke-WebRequest -Uri "https://download.wireguard.com/windows-client/wireguard-installer.exe" -OutFile "$env:TEMP\wireguard-installer.exe"
Start-Process -FilePath "$env:TEMP\wireguard-installer.exe" -ArgumentList "/quiet" -Wait

# 生成服务器密钥对
wg genkey | Out-File -FilePath "C:\WireGuard\server_private.key" -Encoding ASCII
$pubKey = wg pubkey < "C:\WireGuard\server_private.key"
$pubKey | Out-File -FilePath "C:\WireGuard\server_public.key" -Encoding ASCII
```

### 5.3 Step 2: 配置 eaco-global relay

```ini
# C:\WireGuard\eaco-global.conf
[Interface]
PrivateKey = <server_private_key>
Address = 10.13.0.1/24
ListenPort = 51820

# 中继对等体 - eaco-east-asia-1
[Peer]
PublicKey = <eaco-east-asia-1_pubkey>
AllowedIPs = 10.13.1.0/24
Endpoint = <eaco-east-asia-1_ip>:51820
PersistentKeepalive = 25

# 中继对等体 - eaco-europe-1
[Peer]
PublicKey = <eaco-europe-1_pubkey>
AllowedIPs = 10.13.2.0/24
Endpoint = <eaco-europe-1_ip>:51820
PersistentKeepalive = 25

# ... 其他节点类似
```

### 5.4 Step 3-6 检查清单

- [ ] Step 3: 从 eaco-global ping 通所有 Agent 节点的 WireGuard 内网 IP
- [ ] Step 3: 端到端测试：用户 → eaco-global → eaco-east-asia-1 → 出口
- [ ] Step 4: 带宽计量脚本输出每分钟流量计数，每小时 Merkle Root
- [ ] Step 5: 手动向测试节点钱包发放 $e，验证链上交易
- [ ] Step 6: 邀请 50-100 名华语社区用户参与 Beta 测试

---

## 6. 风险与缓解

### 6.1 风险矩阵

| 风险类别 | 风险描述 | 严重度 | 缓解措施 |
|---------|---------|--------|---------|
| **法律** | VPN 服务涉及各国监管 | 🔴 高 | 不在中国大陆提供出口节点；Exit 节点部署在法律宽松地区 |
| **安全** | 节点被攻破/流量泄露 | 🔴 高 | WireGuard 端到端加密；零日志策略；Exit 节点看不到完整路由 |
| **扩展** | 从 8 到 1000+ 节点的增长 | 🟡 中 | 分阶段扩展；自动化节点注册合约；去中心化路由协议 |
| **竞争** | Orchid/Sentinel/Deeper 已有市场 | 🟡 中 | $e MEME 社区优势；更低的进入门槛；华语市场先行 |
| **经济** | $e 价格波动影响节点收益 | 🟡 中 | 收益以 USD 计价后换算 $e；治理池价格稳定基金 |
| **作恶** | 节点串谋/女巫攻击 | 🟡 中 | 质押门槛 + Slashing；超级节点监控；IP 多样性要求 |

### 6.2 竞品对比

```
                  EACO-VPN    Orchid(OXT)   Sentinel     Deeper Network
  ─────────────────────────────────────────────────────────────────────
  底层链          Solana      Ethereum      Cosmos       自有链
  加密协议        WG/SS/Tor   WG/OpenVPN    WG           WG
  进入门槛        10K $e质押  购买OXT       质押DVPN     购买硬件
  社区驱动        ✅ MEME社区  ❌ VC主导     ✅           ❌ 硬件销售
  华语市场        ✅ 先行      ❌             ❌           ✅
  初始节点        8 Agent     少量          较多         较多
  通缩机制        ✅ 10%销毁  ❌             ✅           ❌
```

---

## 7. 路线图

### 7.1 时间线

```
2026
 Jun                    Jul-Sep                  Oct-Dec                 2027 Jan-Jun
  │                       │                        │                       │
  │  Phase 1              │  Phase 2               │  Phase 3              │  Phase 4
  │  8 节点内测           │  华语社区 Beta         │  全球 1000 节点       │  正式运营
  │                       │  100 节点              │  CEX 上所             │  智能合约自动化
  │                       │                        │                       │
  ├─ WireGuard 部署      ├─ 社区节点注册合约     ├─ 全球节点激励计划     ├─ 全自动链上结算
  ├─ 内部连通测试        ├─ 带宽计量上链         ├─ CEX 上所配合         ├─ DAO 治理启动
  ├─ 带宽计量脚本        ├─ $e 自动发放          ├─ 移动端 App          ├─ 跨链桥拓展
  └─ 手动 $e 发放        └─ Beta 用户邀请        └─ 多协议支持          └─ 企业级 SLA
```

### 7.2 Phase 详解

**Phase 1 — 8 节点内测 (Month 1)**
- 在 8 台 Agent 服务器部署 WireGuard
- 建立全网状中继拓扑
- 内部团队测试连通性、延迟、吞吐
- 手动记录带宽消耗，手动发放 $e 奖励
- 目标：验证架构可行性

**Phase 2 — 华语社区 Beta (Month 2-3)**
- 开发社区节点注册 Solana 合约
- 实现带宽计量 + Merkle Root 上链
- $e 自动发放逻辑（合约驱动）
- 邀请 100 名华语社区用户 + 50 个社区节点
- 目标：验证经济模型和防作弊机制

**Phase 3 — 全球扩展 (Month 4-6)**
- 全球节点激励计划，目标 1000 节点
- CEX 上所配合，提升 $e 流动性
- 移动端 App（iOS/Android）
- 支持更多协议（Shadowsocks v2, V2Ray）
- 目标：网络效应起飞

**Phase 4 — 正式运营 (Month 7-12)**
- 全自动链上结算，零人工干预
- DAO 治理启动，社区投票决策
- 跨链桥拓展（Ethereum, Cosmos）
- 企业级 SLA 选项
- 目标：可持续运营的全球去中心化 VPN

---

## 附录

### A. 技术栈总览

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| VPN 协议 | WireGuard | 内核级性能，现代密码学 |
| 备选协议 | Shadowsocks / Tor | 抗审查 / 匿名兜底 |
| 智能合约 | Solana (Anchor) | 高吞吐、低 Gas |
| 代币 | $e (SPL Token) | CA: `DqfoyZH96RnvZusSp3Cdncjpyp3C74ZmJzGhjmHnDHRH` |
| 节点发现 | libp2p + Gossip | 去中心化节点发现 |
| 带宽计量 | 自研 + Merkle Tree | 可验证、防篡改 |
| 前端 | React Native | 跨平台移动端 |

### B. 关键合约接口

```rust
// Solana Anchor 合约 - 节点注册
#[program]
pub mod eaco_vpn {
    // 注册为 Relay 节点
    pub fn register_relay(ctx: Context<RegisterNode>, stake_amount: u64) -> Result<()>;
    
    // 注册为 Exit 节点
    pub fn register_exit(ctx: Context<RegisterNode>, stake_amount: u64) -> Result<()>;
    
    // 提交带宽证明 (Merkle Root)
    pub fn submit_bandwidth_proof(ctx: Context<SubmitProof>, merkle_root: [u8; 32]) -> Result<()>;
    
    // 每日结算
    pub fn daily_settlement(ctx: Context<DailySettlement>) -> Result<()>;
    
    // Slashing
    pub fn slash_node(ctx: Context<SlashNode>, reason: SlashReason) -> Result<()>;
}
```

---

*EACO-VPN — 让每一字节流量都由地球村民共享。* 🌍
