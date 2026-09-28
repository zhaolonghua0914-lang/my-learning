# 倍福 TwinCAT 3 切叠一体机/切刀双工位控制系统工程源码 (QDYT_XWD)

本目录收录了基于倍福 (Beckhoff) TwinCAT 3 平台开发的切叠一体机控制系统源码工程（`QDYT_XWD`，TwinCAT XAE Shell 3.1.4024）。工程实现了模切与叠片双 PLC 任务协同、多轴电子凸轮 (E-Cam) 轨迹规划、张力摆杆闭环控制、电子齿轮同步、CCD 视觉纠偏测量及扫码短路测试等工业自动化核心算法。

---

## 📁 项目架构与工程组织

```
TwinCAT-QDYT-XWD/
├── QDYT_XWD.sln                # Visual Studio / TcXaeShell 解决方案入口
├── QDYT_XWD.tsproj             # TwinCAT 3 系统配置与轴拓扑工程
├── DPPLC/                      # 叠片控制 PLC 项目 (Port 852)
│   ├── P2POUs/                 # 叠片段功能块与工序状态机 (ST/SCL)
│   │   ├── BasicFunction/      # 基础驱动层 (轴控、气缸、真空、滤波、数据持久化)
│   │   │   ├── 03_Motion/      # 轴回原点、多轴联动控制功能块
│   │   │   ├── 04_Gear/        # 电子齿轮动态同步 (FB_GearInDyn / FB_GearInPos)
│   │   │   ├── 07_Ten/         # 隔膜张力与极耳张力 PID 控制算法 (FB_TensionGM)
│   │   │   ├── 09_User1/       # 凸轮曲线生成器 (FB_CreatCamGMBuffer / FB_CamTrapezoid)
│   │   │   └── 10_MES/         # CCD、扫码枪与 MES TCP 通信客户端
│   │   └── POUS/               # 叠台主流程 FB (P2Program_FB, MainState, MAIN)
│   ├── P2DUTs/                 # 叠片数据类型定义 (DUT / 结构体 / 枚举)
│   │   ├── 01_AxisCO_Ntrol/    # 伺服轴参数与状态结构体
│   │   ├── 35_CCD/             # CCD 视觉检测与纠偏交互结构体
│   │   └── 38_User1/           # 系统控制字与工位配方参数
│   └── P2GVLs/                 # 全局变量列表 (IO、运动控制、系统参数)
├── MQPLC/                      # 模切控制 PLC 项目 (Port 851)
│   ├── POUs/                   # 模切段状态机与极片裁切控制
│   │   ├── BasicFunction/      # 模切电机驱动、送料皮带真空与纠偏
│   │   └── POUs/               # 模切主状态机 (MainState, FB_UserMain, MAIN)
│   ├── DUTs/                   # 模切数据类型 (极片长度、张力、视觉数据)
│   └── GVLs/                   # 模切全局变量
├── _Boot/                      # 编译与运行时 Boot 配置与镜像
├── _UserDB/                    # 用户权限与配方数据库 (HysonDB.tcudb)
└── README.md                   # 本工程说明文档
```

---

## ⚙️ 核心技术与控制算法

### 1. 软件架构范式 (IEC 61131-3)
- **多核多任务实时调度**：
  - `Task_P1` (Port 851): 模切主控任务，负责开卷、放卷纠偏、极片模切裁断。
  - `Task_P2Main` / `Task_P2M1` ~ `Task_P2M3` (Port 852): 叠台主流程与 3 工位叠片并行控制。
  - `Task_P3`: 尾卷与下料输送任务。
- **状态机模型**：采用分层有限状态机 (FSM) 架构，严格遵循初始化 (`INIT`)、待机 (`IDLE`)、自动运行 (`RUNNING`)、异常处理 (`ERROR`) 的安全流转规范。

### 2. 运动控制与电子凸轮 (E-Cam / Gear)
- **隔膜送进与摆杆动态补偿**：通过 `FB_CreatCamGMBuffer` 与 `FB_CreatCamVirGMDriver` 实时计算隔膜随动凸轮表，消除高速往复运动中的张力波动。
- **电子齿轮动态切入**：通过 `FB_GearInDyn` / `FB_GearInPos` 实现叠台各轴与主虚轴的位置精确相位同步。

### 3. 张力闭环控制 (Tension Control)
- 结合模拟量采样滤波（`FB_Filter_AD`）与实时 PID 算法，实现放卷张力摆杆恒张力闭环控制。

### 4. 视觉与外设通信
- **CCD 纠偏与 Overhang 测量**：通过 TCP 协议与工控机视觉系统实时握手，读取极片对齐偏移量并在凸轮周期内进行位置前馈补偿。
- **MES 与扫码**：集成条码/二维码扫码通信解析（`ICode` / `CS_CodeTcp`）与电芯短路测试数据采集。

---

## 🛠️ 打开与使用方式

1. **环境要求**：
   - 宿主环境：Beckhoff TwinCAT 3 (推荐版本 `3.1.4024.12` 或更高版本)。
   - 开发平台：Visual Studio 2017/2019 + TcXaeShell。
2. **打开步骤**：
   - 双击打开 `TwinCAT-QDYT-XWD/QDYT_XWD.sln`。
   - 在 TwinCAT 方案中检查各 PLC 项目引用及 Target 运行环境。
