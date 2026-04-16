# K3 CPUFREQ 文档

## 介绍

CPUFREQ 子系统负责在 CPU 运行时通过动态调整 CPU 运行频率与电压，在确保性能需求的前提下，将功耗降至最低。

## 模块介绍

CPUFREQ 子系统负责在 CPU 运行时通过动态调整 CPU 运行频率与电压，在确保性能需求的前提下，将功耗降至最低。

### 功能介绍

**cpufreq core**：cpufreq framework 的核心模块，它主要实现三类功能：
- 抽象调频调压的公共逻辑接口
- 以 sysfs 的形式向用户空间提供统一的接口，以 notifier 的形式向其他 driver 提供频率变化的通知
- 提供CPU频率和电压控制的驱动框架

**cpufreq governor**：负责调频调压的各种策略

**cpufreq driver**：负责平台相关调频调压机制的实现

**cpufreq stats**：负责调频信息和各频点运行事件统计，提供每个CPU的cpufreq有关的统计信息

## 源码结构介绍

CPU 调频平台驱动目录如下：

```
drivers/cpufreq/
├── cpufreq.c
├── cpufreq_conservative.c
├── cpufreq-dt.c
├── cpufreq-dt.h
├── cpufreq-dt-platdev.c
├── cpufreq_governor_attr_set.c
├── cpufreq_governor.c
├── cpufreq_governor.h
├── cpufreq_ondemand.c
├── cpufreq_ondemand.h
├── cpufreq_performance.c
├── cpufreq_powersave.c
├── cpufreq_stats.c
├── cpufreq_userspace.c
├── freq_table.c
└── spacemit-k3-cpufreq.c --> K3平台驱动
```

## K3 平台特性

### 异构多核架构

K3 采用异构多核设计，包含两种不同的 CPU 核心类型：

| 核心类型 | CPU编号 | 数量 | 特性 | 调频调压 |
|---------|--------|------|------|---------|
| X100 | 0-7 | 8个 | 通用计算核心 | 支持动态调频调压 |
| A100 | 8-15 | 8个 | AI加速核心 | 仅支持调频，不支持调压 |

### 支持动态调频调压

实时根据负载切换频率与电压，支持频率 boost 到 2.4GHz。

### X100 核心性能参数

**支持频率档位与对应电压**：

| 频率 (GHz) | 电压 (mV) | 频率 (GHz) | 电压 (mV) |
|-----------|----------|-----------|----------|
| 2.4 | 1000 | 1.2 | 850 |
| 2.3 | 880 | 1.1 | 850 |
| 2.2 | 880 | 1.0 | 800 |
| 2.15 | 880 | 0.8192 | 800 |
| 2.1 | 880 | 0.6144 | 800 |
| 2.0 | 880 | | |
| 1.9 | 880 | | |
| 1.85 | 880 | | |
| 1.8 | 880 | | |
| 1.7 | 880 | | |
| 1.6 | 880 | | |
| 1.5 | 850 | | |
| 1.4 | 850 | | |
| 1.3 | 850 | | |

**共18个频率档位**，频率范围：614.4 MHz ~ 2.4 GHz

### A100 核心性能参数

**支持频率档位**（无电压调节）：

| 频率 (GHz) | 频率 (GHz) |
|-----------|-----------|
| 2.0 | 1.2 |  
| 1.9 | 1.1 | 
| 1.85 | 1.0 | 
| 1.8 | 0.8192| 
| 1.7 | 0.6144 | 
| 1.6 | | 
| 1.5 | | 
| 1.4 | | 
| 1.3 | | 

**共13个频率档位**，频率范围：614.4 MHz ~ 2.0 GHz

## 配置介绍

### CONFIG 配置

CPUFREQ 配置如下：

```
CONFIG_SPACEMIT_K3_CPUFREQ:

 This adds the CPUFreq driver support for Spacemit K3 SoCs
 which are capable of changing the CPU's frequency dynamically.
 
 Symbol: SPACEMIT_K3_CPUFREQ [=y]
 Type  : tristate
 Defined at drivers/cpufreq/Kconfig:256
 Prompt: CPU frequency scaling driver for Spacemit K1X
 Depends on: OF [=y] && COMMON_CLK [=y]
 Location:
  -> CPU Power Management
   -> CPU Frequency scaling
    -> CPU Frequency scaling (CPU_FREQ [=y])
     -> CPU frequency scaling driver for Spacemit K1X (SPACEMIT_K3_CPUFREQ [=y])
 Selects: CPUFREQ_DT [=y] && CPUFREQ_DT_PLATDEV [=y]
```

### DTS 配置

完整 OPP 表位于：
```
arch/riscv/boot/dts/spacemit/k3_opp_table.dtsi
```

#### X100 核心 OPP 表配置

OPP 表名称：`clst_core_opp_table0_x100`

**时钟源配置**：
- `pll_clst0`: CLK_PLL3
- `pll_clst1`: CLK_PLL4
- `pll_src`: CLK_PLL3_D1
- `clt_pll_src`: CLK_APMU_CPU_C1_PLL_SRC

**时钟延迟**：200 µs

**电压范围**：
- 800 mV：614.4 MHz, 819.2 MHz, 1.0 GHz
- 850 mV：1.1 GHz ~ 1.5 GHz
- 880 mV：1.6 GHz ~ 2.3 GHz
- 1000 mV：2.4 GHz

#### A100 核心 OPP 表配置

OPP 表名称：`clst_core_opp_table0_a100`

**时钟源配置**：
- `pll_clst0`: CLK_PLL5
- `pll_clst1`: CLK_PLL8
- `pll_src`: CLK_PLL5_D1
- `clt_pll_src`: CLK_APMU_CPU_C3_PLL_SRC

**时钟延迟**：200 µs

**注意**：A100 核心不支持电压调节

#### CPU 供电配置

**X100 核心（CPU 0-7）**：
```
clst-supply = <&edcdc1>;
clocks = <&syscon_apmu CLK_APMU_CPU_C0_CORE>, <&syscon_apmu CLK_APMU_CPU_C1_CORE>;
clock-names = "cls0", "cls1";
operating-points-v2 = <&clst_core_opp_table0_x100>;
```

**A100 核心（CPU 8-15）**：
```
clocks = <&syscon_apmu CLK_APMU_CPU_C2_CORE>, <&syscon_apmu CLK_APMU_CPU_C3_CORE>;
clock-names = "cls0", "cls1";
operating-points-v2 = <&clst_core_opp_table0_a100>;
```

## 驱动实现

### 平台驱动文件

**文件路径**：`drivers/cpufreq/spacemit-k3-cpufreq.c`

### 核心功能

#### 1. 频率转换处理

驱动通过 `CPUFREQ_TRANSITION_NOTIFIER` 和 `CPUFREQ_POLICY_NOTIFIER` 两个 notifier 处理频率转换：

**关键频率阈值**：
- `TURBO0_FREQUENCY = 1.0 GHz`：Turbo 模式阈值
- `STABLE_FREQUENCY = 819.2 MHz`：稳定运行频率

**频率转换流程**：
1. 在 `CPUFREQ_PRECHANGE` 阶段，如果目标频率 > 1.0 GHz，则设置 PLL 时钟
2. 在 `CPUFREQ_POSTCHANGE` 阶段，完成频率切换后的处理

#### 2. PLL 时钟管理

驱动管理两个 PLL 时钟源：
- `pll_clst0`：第一个 PLL 时钟
- `pll_clst1`：第二个 PLL 时钟

在高频率运行时，驱动会动态调整 PLL 时钟频率以支持 Turbo 模式。

#### 3. 电压调节

- **X100 核心**：支持通过 `edcdc1` 稳压器进行动态电压调节
- **A100 核心**：不支持电压调节（CPU >= 8 时禁用稳压器配置）

#### 4. OPP 表初始化

驱动在初始化时：
1. 为每个 CPU 配置 OPP 表
2. 根据 CPU 类型（X100 或 A100）选择相应的稳压器配置
3. 初始化 cpufreq 频率表

## Debug 介绍

### sysfs 接口

所有节点位于：
```
/sys/devices/system/cpu/cpufreq/policy0/
```

| 节点 | 作用说明 |
|-----|--------|
| `affected_cpus`, `related_cpus` | 查看系统支持的策略 |
| `scaling_governor` | 当前生效的调频策略 |
| `scaling_available_frequencies` | 查看系统支持的频率点 |
| `scaling_max_freq` | 软件允许的最大频率限制 |
| `cpuinfo_cur_freq` | CPU 硬件实时频率 |
| `scaling_available_governors` | 查看系统支持的策略 |
| `scaling_min_freq` | 软件允许的最小频率限制 |
| `cpuinfo_max_freq` | 硬件支持的最大频率 |
| `scaling_setspeed` | userspace 模式下设置 CPU 频率的接口 |
| `cpuinfo_min_freq` | 硬件支持的最小频率 |
| `scaling_cur_freq` | 查看当前CPU跑的频率 |
| `cpuinfo_transition_latency` | 频率转换延迟 |
| `scaling_driver` | 当前cpu调频驱动的名称 |

## 测试方法

### 1. 将策略修改成 userspace 模式

```bash
echo userspace > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
```

### 2. 查看所支持的频率列表

```bash
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
```

**X100 核心输出示例**：
```
614400 819200 1000000 1100000 1200000 1300000 1400000 1500000 1600000 1700000 1800000 1850000 1900000 2000000 2100000 2150000 2200000 2300000 2400000
```

**A100 核心输出示例**：
```
614400 819200 1000000 1100000 1200000 1300000 1400000 1500000 1600000 1700000 1800000 1850000 1900000 2000000
```

### 3. 设置cpu频率

```bash
echo 1228800 > /sys/devices/system/cpu/cpufreq/policy0/scaling_setspeed
```

### 4. 查看频率是否设置成功

```bash
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_cur_freq
```

### 5. 查看当前电压（仅 X100 核心）

```bash
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_cur_volt
```

## 调频策略

系统支持以下调频策略：

| 策略 | 说明 |
|-----|------|
| `performance` | 始终运行在最高频率，性能最优但功耗最高 |
| `powersave` | 始终运行在最低频率，功耗最低但性能最差 |
| `ondemand` | 根据 CPU 负载动态调整频率，平衡性能和功耗 |
| `conservative` | 类似 ondemand，但频率变化更平缓 |
| `userspace` | 由用户空间程序控制频率 |
| `schedutil` | 由调度器根据任务需求动态调整频率 |

## 性能优化建议

### 1. 选择合适的调频策略

- **高性能场景**：使用 `performance` 策略
- **低功耗场景**：使用 `powersave` 策略
- **通常场景**：使用 `schedutil` 或 `ondemand` 策略

### 2. 设置频率限制

```bash
# 设置最小频率
echo 1000000 > /sys/devices/system/cpu/cpufreq/policy0/scaling_min_freq

# 设置最大频率
echo 2400000 > /sys/devices/system/cpu/cpufreq/policy0/scaling_max_freq
```

### 3. 监控频率和电压

```bash
# 实时监控频率变化
watch -n 1 'cat /sys/devices/system/cpu/cpufreq/policy0/scaling_cur_freq'

# 查看频率统计信息
cat /sys/devices/system/cpu/cpufreq/policy0/stats/time_in_state
```


## 相关文件

- 驱动源码：`drivers/cpufreq/spacemit-k3-cpufreq.c`
- OPP 表：`arch/riscv/boot/dts/spacemit/k3_opp_table.dtsi`
- CPU 配置：`arch/riscv/boot/dts/spacemit/k3-cpus.dtsi`
- Kconfig：`drivers/cpufreq/Kconfig`
- 内核文档：`Documentation/admin-guide/pm/cpufreq.rst`
