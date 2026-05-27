# Ubuntu 系统诊断命令手册

> 用途：检查硬件配置、定位卡顿原因、分析资源瓶颈

---

## 一、硬件配置查看

### 1. CPU 信息

```bash
# CPU 详细架构（核数、线程、架构、频率）
lscpu

# CPU 型号名称（最简）
cat /proc/cpuinfo | grep "model name" | head -1

# CPU 物理核心 / 逻辑线程分布
grep -E "physical id|core id|cpu cores|siblings" /proc/cpuinfo | sort -u
```

### 2. 内存信息

```bash
# 总容量 / 已用 / 可用 / swap（最常用）
free -h

# 原始内存数据（MemTotal / SwapTotal / 缓存大小等）
cat /proc/meminfo | head -20

# 查看内存硬件详情：插了几个槽、每根容量、频率、型号
# ⚠️ 需要 sudo 密码
sudo dmidecode -t memory | grep -E "Locator|Size|Type|Speed|Manufacturer|Part Number"

# 查看内存最大支持容量
sudo dmidecode -t memory | grep "Maximum Capacity"
```

### 3. 磁盘信息

```bash
# 列出所有块设备：大小 / 型号 / 接口 / 是否机械盘
# ROTA=1 → 机械盘 HDD；ROTA=0 → 固态 SSD
lsblk -d -o NAME,SIZE,MODEL,ROTA,TRAN

# 快速判断单块盘是固态还是机械（1=HDD，0=SSD）
cat /sys/block/sda/queue/rotational

# 磁盘详细型号（需安装 smartctl）
sudo smartctl -i /dev/sda

# 检查是否有 NVMe 固态
lspci | grep -i nvme
ls /dev/nvme*

# 各分区大小、使用率
df -h

# swap 交换分区
swapon --show
```

### 4. 主板 / 整机信息

```bash
# 主板厂商、型号
# 方法一（需 sudo）：
sudo dmidecode -t baseboard | grep -E "Manufacturer|Product|Version"

# 方法二（免 sudo）：
cat /sys/devices/virtual/dmi/id/board_vendor
cat /sys/devices/virtual/dmi/id/board_name

# 整机型号（笔记本 / 品牌机适用）
sudo dmidecode -t system | grep -E "Manufacturer|Product"

# BIOS 版本
sudo dmidecode -t bios | grep -E "Vendor|Version|Date"
```

### 5. 外设与扩展槽

```bash
# 全部 PCIe 设备（显卡 / SATA控制器 / NVMe / USB控制器…）
lspci

# 只看存储相关控制器
lspci | grep -iE 'sata|ahci|nvme|storage|m.2'

# PCIe 总线拓扑 — 哪些槽在用、哪些空着
lspci -t

# SATA 接口数量（数 host 数量，有几个就能插几个 SATA 盘）
ls /sys/class/scsi_host/
```

---

## 二、系统资源占用

### 1. 负载与运行时间

```bash
# 运行时间 + 平均负载（1min / 5min / 15min）
# 负载 > CPU 核数 → 过载
uptime
```

### 2. 进程排行

```bash
# 按 CPU 占用排序（看谁在吃 CPU）
ps aux --sort=-%cpu | head -20

# 按内存占用排序（看谁在吃内存）
ps aux --sort=-%mem | head -20
```

### 3. 动态交互监控

```bash
# 实时监控（CPU / 内存 / 进程 / 负载）
top

# top 增强版（颜色、更直观）
htop
```

---

## 三、性能瓶颈定位

### 1. I/O 等待与磁盘瓶颈（最关键的卡顿指标）

```bash
# CPU 时间分布：us=用户态  sy=内核态  id=空闲  wa=等待IO
# ⚠️ wa > 30% → 磁盘是瓶颈
vmstat 1 5

# 磁盘 I/O 详细统计
# %util ~100%  → 磁盘满载
# await > 30ms → 慢盘（SSD 通常 < 2ms）
sudo iostat -x 1 3
```

### 2. 内存压力

```bash
# swap 是否被用满（swap 满了说明物理内存严重不足）
swapon --show
free -h

# 每个进程实际物理内存占用（PSS 更准确，会按共享比例分摊）
sudo smem -t | head -30
```

### 3. 启动耗时（看哪些服务拖慢开机）

```bash
# 按耗时从高到低排列
systemd-analyze blame | head -20
```

### 4. 内核错误日志

```bash
# OOM（内存溢出）、进程被杀、温度过高
sudo dmesg | grep -iE 'error|oom|killed|temperature|thermal|throttl'
```

---

## 四、关键指标速查表

| 指标 | 命令 | 正常范围 | ⚠️ 异常 |
|---|---|---|---|
| **I/O 等待** | `vmstat` / `top` 的 `wa` | < 5% | > 30% → **磁盘瓶颈** |
| **磁盘利用率** | `iostat -x` 的 `%util` | < 60% | > 90% → 磁盘满载 |
| **Swap 使用率** | `free -h` / `swapon` | < 50% | **100%** → 内存严重不足 |
| **系统负载** | `uptime` 的 load average | < CPU 核数 | > 核数 → 排队过载 |
| **可用内存** | `free -h` 的 available | > 20% | < 10% → 内存紧张 |
| **磁盘响应延迟** | `iostat -x` 的 `await` | SSD < 2ms, HDD < 15ms | HDD > 30ms → 慢盘 |
| **是否机械盘** | `lsblk -d -o ROTA` | 0 = 固态 (SSD) | **1 = 机械 (HDD)** → 较慢 |

---

## 五、一键诊断脚本

快速跑一遍所有关键指标：

```bash
echo "===== CPU =====" \
  && lscpu | grep "Model name" \
  && echo "===== 内存 =====" \
  && free -h \
  && echo "===== 磁盘 =====" \
  && lsblk -d -o NAME,SIZE,MODEL,ROTA,TRAN \
  && echo "===== 负载 =====" \
  && uptime \
  && echo "===== 谁吃内存最多 =====" \
  && ps aux --sort=-%mem | head -8 \
  && echo "===== 交换分区 =====" \
  && swapon --show \
  && echo "===== 主板 =====" \
  && cat /sys/devices/virtual/dmi/id/board_vendor \
  && cat /sys/devices/virtual/dmi/id/board_name
```

---

## 六、卡顿排查思路（决策树）

```
系统卡顿？
│
├─ 先看负载：uptime
│   └─ 负载高？
│       ├─ CPU 高？→ top 看哪个进程吃 CPU
│       ├─ I/O 高（wa）？→ iostat -x 看磁盘
│       └─ 内存高？→ free -h → swap 满了？
│
├─ 看磁盘：iostat -x 1 3
│   ├─ %util ≈ 100% → 磁盘满载
│   ├─ await > 30ms → 盘太慢（HDD vs SSD）
│   └─ ROTA=1 → 机械盘，换 SSD 能解决
│
├─ 看内存：free -h
│   ├─ available < 10% → 内存不足
│   ├─ swap 满 → 内存严重不足
│   └─ 谁在吃？→ ps aux --sort=-%mem
│
└─ 看进程：ps aux --sort=-%cpu
    └─ 异常进程？高 CPU / 高内存？
```
