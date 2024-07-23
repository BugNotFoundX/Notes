# CPU设计


- 本章通过讲解 LainCore 代码，介绍如何设计一个 LA32R 指令集的顺序双发射处理器
- 本章内容按照处理器流水线从前到后顺序组织
- 项目地址https://github.com/LainChip/LainCore
- 项目开发环境基于Ubuntu22.04-LTS
- 本章仅关注如何实现，相关知识参阅[教程文档]()

## 1 分支预测

- 代码主要位于`LainCore/rtl/core_npc_complex.sv`文件中
- 本小节以LainCore分支预测器为例，介绍如何构建一个简单的二级分支预测器
- 本节仅关注如何实现一个简单二级分支预测器，分支预测相关知识参阅[教程文档]()

### 1.1 整体架构

- 整体上实现了二级分支预测器，架构如图：（TODO）
- 由于是顺序双发射架构，预测器每周期给出相邻8对齐两条PC的预测结果
- BPU更新信息生成（TODO）

### 1.2 具体实现

- `pipeline.svh`中相关宏定义

  ```systemverilog
  // 定义4种跳转类型
  `define _BPU_TARGET_NPC  (2'd0)    // 不跳转
  `define _BPU_TARGET_CALL (2'd1)    // 函数调用类型
  `define _BPU_TARGET_RETURN (2'd2)  // 调用返回类型
  `define _BPU_TARGET_IMM (2'd3)     // 目标地址为pc + imm
  ```

- `pipeline.svh`中相关结构体定义

  ```systemverilog
  typedef struct packed {
    logic taken;  // 预测是否跳转
    logic [                31:0] predict_pc ;  // 预测跳转地址
    logic [                 1:0] lphr       ;  // 历史模式信息
    logic [`_BHT_DATA_WIDTH-1:0] history    ;  // 分支历史
    logic [                 1:0] target_type;  // 指令类型
    logic                        dir_type   ;  // 是否条件跳转
    logic [`_RAS_ADDR_WIDTH-1:0] ras_ptr;  // RAS栈指针
  } bpu_predict_t;  // 分支预测信息
  
  typedef struct packed {
    logic                        miss       ;  // 预测失败
    logic [                31:0] pc         ;  // 预测失败的PC
    logic                        true_taken ;  // 是否跳转
    logic [                31:0] true_target;  // 跳转目标
    logic [                 1:0] lphr       ;  // 预测信息携带的旧的lphr，更新时需要进一步计算
    logic [`_BHT_DATA_WIDTH-1:0] history    ;  // 预测信息携带的旧的历史，更新时需要进一步计算
  
    logic need_update         ;  // BPU是否需要更新
    logic true_conditional_jmp;  // 是否是条件跳转
  
    logic ras_miss_type;  // 是否需要更新RAS
  
    logic[1:0] true_target_type;  // 指令分支类型
  
    logic [`_RAS_ADDR_WIDTH-1:0] ras_ptr;  // 预测信息携带的旧的RAS栈指针
  } bpu_correct_t;  // 正确的分支信息，也是BPU更新信息
  
  
  ```
  
- 顶层接口定义：

  ```systemverilog
  module core_npc (
    input  logic              clk       ,
    input  logic              rst_n     ,
    input  logic              rst_jmp   ,  // 流水线冲刷信号
    input  logic [31:0]       rst_target,  // 流水线冲刷时的目标地址
    input  logic              f_stall_i ,  // 暂停信号
    output logic [31:0]       pc_o      ,
    output logic [31:0]       npc_o     ,
    output logic [ 1:0]       valid_o   ,  // 2'b00 | 2'b01 | 2'b11 | 2'b10
    output bpu_predict_t [1:0]predict_o ,
    input  bpu_correct_t      correct_i
  );
  ```
  
  
  
- 本节一些通用的命名含义：

| 名称   | 全称                        | 含义        |
| ---- | ------------------------- | --------- |
| pc   | program counter           | 程序计数器     |
| npc  | next pc                   | 下一周期的pc   |
| ppc  | predicted pc              | pc的预测结果   |
| _q   |                           | 触发器输出端的标识 |
| lphr | local pattern history reg | 部模式历史寄存器  |

#### 1.2.1 RAS实现

- RAS（Return Address Stack）相关知识见[教程文档]()
- 一般的RAS的大小不超过16，**LainCore中RAS大小为8**

- 定义存储结构`logic[7:0][31:0] ras_q;`

- 定义读写指针` logic[2:0] ras_w_ptr_q, ras_ptr_q;`

- 定义分支预测逻辑中用到的RAS值的寄存器`logic[31:0] ppc_ras_q;`

- RAS写入：在预测指令为CALL类型指令时被写入

  ```systemverilog
  // 如果pc是CALL类型指令且流水线不发生暂停
  if(pc_is_call && !f_stall_i) begin
    // 按照指令是第一条还是第二条计算写入的PC值，并进行写入
    ras_q[ras_w_ptr_q] <= {pc[31:3], 3'b000} + (inst_0_jmp ? 32'd4 : 32'd8);
    // 更新写指针
    ras_w_ptr_q <= ras_w_ptr_q + 3'd1;
    // 更新读指针
    ras_ptr_q <= ras_ptr_q + 3'd1;
  end
  ```

- RAS读出：在预测指令为RETURN类型指令时读出

  ```systemverilog
  // 在第一个周期将栈顶元素读出到寄存器，优化时序
  always_ff @(posedge clk) begin
  	ppc_ras_q <= {ras_q[ras_ptr_q][31:2], 2'b00};
  end
  
  // 第二周期，如果pc是RETURN类型指令且流水线不发生暂停
  if(pc_is_return && !f_stall_i) begin
  	// 更新读写指针
  	ras_w_ptr_q <= ras_w_ptr_q - 3'd1;
  	ras_ptr_q <= ras_ptr_q - 3'd1;
  end
  ```
  
- RAS更新：在分支预测失败时更新

  ```systemverilog
  // RETURN MISS: 更新读写指针
  // CALL   MISS: 更新读写指针和RAS栈顶元素
  if(correct_i.miss || correct_i.ras_miss_type) begin
    // 更新写指针
    ras_w_ptr_q <= correct_i.ras_ptr + 3'd1;
    // 更新读指针
    ras_ptr_q <= correct_i.ras_ptr;
    // CALL 类型的MISS 还需要更新RAS栈顶元素
    if(correct_i.true_target_type == `_BPU_TARGET_CALL) begin
      ras_q[correct_i.ras_ptr] <= correct_i.pc + 3'd4;
    end
  end
  ```


#### 1.2.2 BTB实现

- BTB（Branch Target Buffer）相关知识见[教程文档]()

- BTB被分为两个部分：地址部分（btb部分）和信息部分（info部分）

- **注意：**历史遗留问题，代码命名中btb仅指代地址缓存，info指代指令信息缓存，**切勿将代码btb与教程中BTB含义混淆**

- btb部分包含1024个表项，分2个BANK，结构为2 * 512 entry

- info部分包含512个表项，分2个BANK，结构为2 * 256 entry

- 定义btb ram相关变量：

  ```systemverilog
  logic[8:0] btb_waddr,btb_raddr;
  logic[1:0] btb_we;
  logic[31:0] btb_wdata;
  logic[1:0][31:0] btb_rdata;
  ```

- 定义info ram相关变量：

  ```systemverilog
  typedef struct packed {
    logic [1:0] target_type; // 0 npc, 1 call, 2 return, 3 imm
    logic conditional_jmp;   // 0：非条件跳转；1：条件跳转
    logic [4:0] history;     // 分支历史
    logic [5:0] tag;         // pc对应的tag，确定pc身份
  } branch_info_t;
  
  logic[7:0] info_waddr,info_raddr;
  logic[1:0] info_we;
  branch_info_t winfo;
  branch_info_t [1:0]rinfo_q;
  ```

- 存储结构定义：

  ```systemverilog
  // 创建两个 btb 和 info mem，用于写更新时区别开来。
  for(genvar p = 0 ; p < 2 ; p++) begin  
    simpleDualPortRamRE #(
      .dataWidth(30 ),
      .ramSize  (512),
      .latency  (1  ),
      .readMuler(1  )
    ) btb_table (
      .clk     (clk       ),
      .rst_n   (rst_n     ),
      .addressA(btb_waddr ),
      .we      (btb_we[p] ),
      .addressB(btb_raddr ),
      .re      (!f_stall_i),
      .inData  (btb_wdata[31:2]),
      .outData (btb_rdata[p][31:2])
    );
    assign btb_rdata[p][1:0] = '0;
    simpleDualPortLutRam #(
      .dataWidth($bits(branch_info_t)),
      .ramSize  (256),
      .latency  (1  ),
      .readMuler(1  )
    ) info_table (
      .clk     (clk       ),
      .rst_n   (rst_n     ),
      .addressA(info_waddr),
      .we      (info_we[p] | !rst_n_q),
      .addressB(info_raddr ^ rst_addr_q),
      .re      (!f_stall_i),
      .inData  (winfo),
      .outData (rinfo_q[p])
    );
  end
  ```

- btb读写逻辑：

  ```systemverilog
  assign btb_raddr = npc[11:3];              // 读地址来自NPC
  assign btb_waddr = correct_i.pc[11:3];     // 写地址来自BPU更新请求
  assign btb_wdata = correct_i.true_target;  // 将正确的跳转目标写入btb ram
  always_comb begin
    btb_we = '0;  // 默认不发生写入
    // 仅更新PC对应的BANK
    // 仅CALL和IMM类型指令需要缓存目标地址
    // 因为：NPC类型不跳转，RETURN类型地址来自RAS
    btb_we[correct_i.pc[2]] = correct_i.need_update && 
                             (correct_i.true_target_type == `_BPU_TARGET_CALL ||
                              correct_i.true_target_type == `_BPU_TARGET_IMM);
  end
  ```

- info读写逻辑

  ```systemverilog
  assign info_raddr = npc[10:3];          // 读地址来自NPC
  assign info_waddr = correct_i.pc[10:3]; // 写地址来自BPU更新请求
  always_comb begin
  	info_we = '0;  // 默认不发生写入
    // 仅更新PC对应的BANK
  	info_we[correct_i.pc[2]] = correct_i.need_update;
  end
  always_comb begin
    // 复位时写入信息应采用NPC类型
  	winfo.target_type = rst_n_q ? correct_i.true_target_type : `_BPU_TARGET_NPC;
  	// 更新条件跳转信息
    winfo.conditional_jmp = correct_i.true_conditional_jmp;
  	// 滚动更新分支历史
    winfo.history = {correct_i.history[3:0], correct_i.true_taken};
  	// 更新PC对应的tag
    winfo.tag = get_tag(correct_i.pc);
  end
  ```

#### 1.2.3 PHT实现

- PHT（Pattern History Table）相关知识见[教程文档]()

- 本文BPU架构中，PHT共32个表项

- 定义PHT相关信号：

  ```systemverilog
  logic level2_we;
  logic [4:0] level2_waddr;
  logic [1:0] level2_wdata;
  logic [1:0][4:0] level2_raddr;
  logic [1:0][1:0] level2_cnt;  // PHT的读出数据
  ```

- 存储结构定义：

  ```systemverilog
  for(genvar p = 0 ; p < 2; p++) begin
    simpleDualPortLutRam #(
      .dataWidth(2 ),
      .ramSize  (32),
      .latency  (0 ),  // 注意这里的延迟是0，
      .readMuler(1 )
    ) l2_table (
      .clk     (clk            ),
      .rst_n   (rst_n          ),
      .addressA(level2_waddr   ),
      .we      (level2_we      ),
      .addressB(level2_raddr[p]),
      .re      (1'b1           ),
      .inData  (level2_wdata   ),
      .outData (level2_cnt[p]  )
    );
  end
  ```

- 读写逻辑：

  ```systemverilog
  // 按照两位饱和计数器的规范计算要写入的pht表项信息
  assign level2_wdata = gen_next_lphr(correct_i.lphr, correct_i.true_taken);
  // 写入地址是更新信息中携带的分支历史
  assign level2_waddr = correct_i.history;
  // 是条件跳转的指令才会更新这个表
  assign level2_we = correct_i.true_conditional_jmp && correct_i.need_update;
  for(genvar p = 0 ; p < 2; p++) begin
    // 使用分支历史来寻址pht
    assign level2_raddr[p] = rinfo_q[p].history;
  end
  ```

#### 1.2.4 分支预测

- 本阶段主要任务如下：
  - 跳转方向预测
  - 有效性标记（标记输出的PC的有效性）
  - 跳转目标预测
  - 生成分支预测信息

- 跳转方向预测：

  ```systemverilog
  // 定义相关信号
  logic [1:0] branch_need_jmp;  // 是否需要跳转
  logic [1:0] tag_match;  // PC与读出的TAG是否匹配
  
  // 预测逻辑，valid_q参看指令有效性标记
  for(genvar p = 0 ; p < 2; p++) begin
    always_comb begin
      branch_need_jmp[p] = '0; // 默认不跳转
      // 判断info表项是否被PC命中
      assign tag_match[p] = rinfo_q[p].tag == get_tag(pc);
      // 如果PC命中info表项且是分支指令
      if(rinfo_q[p].target_type != `_BPU_TARGET_NPC && tag_match[p]) begin
        // 以下条件判断PC是否对应无条件跳转指令
        if(rinfo_q[p].target_type != `_BPU_TARGET_IMM || !rinfo_q[p].conditional_jmp) begin
          branch_need_jmp[p] = /*'1*/valid_q[p];
        end
        else begin
          // PC被判定为条件跳转指令
          // 根据PHT的结果判断是否跳转
          branch_need_jmp[p] = level2_cnt[p][1] && valid_q[p];
        end
      end
    end
  end
  ```

- 指令有效性标记

  - 在完成指令跳转情况判断后就可以对npc的有效性进行标记
  - 分两个周期进行
    - 第一周期：根据npc判断第一条指令是否有效
    - 第二周期：根据预测跳转信息判断第二条指令是否有效

  ```systemverilog
  // 定义相关信号
  // output logic [1:0] valid_o; 标记BPU输出PC的有效性
  logic[1:0] valid_q;  // nvalid缓存一拍，与npc流动一致
  logic[1:0] nvalid;   // 表示由npc值决定的有效性
  
  // nvalid 逻辑
  always_comb begin
    // 默认两条指令都有效
    nvalid = 2'b11;
    // 当npc的其实位置是相邻至指令的第二条指令
    // 将第一条指令无效化
    if(npc[2]) begin
      nvalid[0] = '0;
    end
  end
  
  // valid_o 逻辑
  always_comb begin
    valid_o = valid_q & {!f_stall_i, !f_stall_i};
    // 若第一条指令预测跳转，将第二条指令无效化
    if(inst_0_jmp) begin
      valid_o[1] = '0;
    end
  end
  
  // 更新valid_q
  always_ff @(posedge clk) begin
    if(!rst_n) begin
      valid_q <= 2'b11;
    end
    else begin
      // 当流水线不暂停或冲刷流水线时写入
      if(!f_stall_i || rst_jmp) begin
        valid_q <= nvalid;
      end
    end
  end
  ```

- 跳转目标预测

  - 目标地址共有4个来源
    - PPC_RAS_Q: RAS 方式预测的下一个跳转地址
    - PPC_BTB: BTB 方式预测的下一个跳转地址
    - PPC_PLUS8: 不预测跳转情况下，正常 +8
    - JPC: 后端刷新管线的跳转地址

  ```systemverilog
  always_comb begin
    // npc 默认为 align(pc, 8) + 8
    npc = ppc_plus8;
    if(branch_need_jmp[0]) begin
      // 如果第一条指令预测跳转，根据类型选择目标地址
      npc = rinfo_q[0].target_type == `_BPU_TARGET_RETURN ? ppc_ras_q : raw_ppc_btb[0];
    end else if(branch_need_jmp[1]) begin
      // 如果第一条指令跳转，自然无需再判断第二条指令的目标地址
      // 如果第二条指令预测跳转，根据类型选择目标地址
      npc = rinfo_q[1].target_type == `_BPU_TARGET_RETURN ? ppc_ras_q : raw_ppc_btb[1];
    end
    // 冲刷流水线优先级最高
    if(rst_jmp) begin
      npc = jpc;
    end
  end
  ```

- 分支信息输出

  ```systemverilog
  always_comb begin
    if(branch_need_jmp[0]) begin
      predict_o[0].taken = '1;
    end
    else if(branch_need_jmp[1]) begin
      predict_o[1].taken = '1;
    end
  end
  
  always_comb begin
    predict_o[0].predict_pc = npc;
    predict_o[1].predict_pc = npc;
    predict_o[0].lphr = level2_cnt[0];
    predict_o[1].lphr = level2_cnt[1];
    predict_o[0].history = rinfo_q[0].history;
    predict_o[1].history = rinfo_q[1].history;
    predict_o[0].target_type = rinfo_q[0].target_type;
    predict_o[1].target_type = rinfo_q[1].target_type;
    predict_o[0].dir_type = rinfo_q[0].conditional_jmp;
    predict_o[1].dir_type = rinfo_q[1].conditional_jmp;
    predict_o[0].ras_ptr = ras_ptr_q;
    predict_o[1].ras_ptr = ras_ptr_q;
  end
  ```

## 2 指令译码

- 手动构建译码器比较简单，一般采用AND-OR结构

- 可参考https://gitee.com/loongson-edu/open-la500/blob/master/id_stage.v

- 本节介绍如何使用 LainCore 提供的解码器脚本生成解码器，可大大提升开发效率

### 2.1 配置脚本运行环境

#### 2.1.1 安装Java

- 运行如下命令

  ```bash
  apt install openjdk-21-jre-headless  # 安装Java运行环境
  apt install openjdk-21-jdk-headless  # 安装Java开发环境
  ```

- 测试Java运行环境

  ```bash
  $ java --version
  openjdk 21.0.3 2024-04-16
  OpenJDK Runtime Environment (build 21.0.3+9-Ubuntu-1ubuntu122.04.1)
  OpenJDK 64-Bit Server VM (build 21.0.3+9-Ubuntu-1ubuntu122.04.1, mixed mode, sharing)
  ```

- 测试Java开发环境

  ```bash
  $ javac --version
  javac 21.0.3
  ```

#### 2.1.2 安装Scala

- 下载安装包

  ```bash
  wget https://downloads.lightbend.com/scala/2.11.12/scala-2.11.12.deb
  ```

- 安装

  ```bash
  sudo dpkg -i scala-2.11.12.deb
  ```

- 检查

  ```bash
  $ scala -version
  Scala code runner version 2.11.12 -- Copyright 2002-2017, LAMP/EPFL
  ```

#### 2.1.3 安装sbt

- **该步骤需要正确配置代理**

```bash
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
sudo apt-get update
sudo apt-get install sbt
```

### 2.2 脚本使用

#### 2.2.1 编写解码器相关信息

- 代码位于`LainCore/src/simple-decoder`文件夹下

- 脚本通过编写JSON文件定义解码器

  - 脚本支持读入多个JSON文件

  - 脚本一次性读入所有JSON后才进行分析，在A脚本中定义的内容可在B脚本中使用

  - JSON文件规范如下：

    ```json
    {
      // 支持3种指令信号值的定义方式
      "signal_values": {
        // 列表形式
        // 最终会在decoder.svh中生成对应宏定义
        "signal_name0": [
          "S0_VALUE0",  // 0
          "S0_VALUE1",  // 1
          "s0_VALUE3"   // 2
        ],
        
    		// MAP形式
        // 最终会在decoder.svh中生成对应宏定义
        // 宏定义值与此处赋值相同
        "signal_name1": {
          "S1_VALUE0" : 0,
          "S1_VALUE1" : 0,
          "S1_VALUE2" : 1,
          "S1_VALUE3" : 2,
          "S1_VALUE4" : 3,
          "S1_VALUE5" : 4
        },
    
        // 真值形式
        // 不会生成宏定义
        "signal_name2": [false, true]
      },
      "signals": {
        // 信号定义包括三个键值对
        // length：信号位宽
        // stage：信号最后一个有效的流水级，此后信号被抛弃，用于生成信号在流水线中传递的接线函数
        // default：信号默认值，需保证与信号位宽一致
        "signal_name0": {
          "length": 2,
          "stage": "ex",
          "default": "S0_VALUE0"
        },
        "signal_name1": {
          "length": 3,
          "stage": "m1",
          "default": "S1_VALUE0"
        },
        "signal_name2": {
          "length": 1,
          "stage": "m2",
          "default": false
        }
    
      },
      "instructs": {
        // 指令解码信息定义
        // opcode：指令标识码，原则上支持掩码形式，即支持形如“01010101----0101”的形式
        // signal_namex：指令关注的信号，例如addi指令需要将alu_op信号设置为"ADD"
        // 指令仅需要设置自己会影响到的信号，其余信号会采用默认值
        "instruct_name0": {
          "opcode": "01010101",
          "signal_name0": "S0_VALUE0",
          "signal_name1": "S1_VALUE0",
          "signal_name2": false
        },
        "instruct_name1": {
          "opcode": "01010110",
          "signal_name0": "S0_VALUE1",
          "signal_name2": true
        },
        "instruct_name2": {
          "opcode": "01010110"
        }
      }
    }
    ```

- 脚本Config文件

  ```scala
  object Config {
    val useJson = true  // 也可以直接编写SpinalHDL格式的解码文件，此时设置该值为false，但是此功能还不完整
    // 定义CPU流水级，需要考虑顺序，需与JSON中出现的一致
    val stages: List[String] = List("is", "ex", "m1", "m2", "wb")
    // 如果有想要添加到decoder.svh的其他和解码无关常量
    val constPatch: String =
      """
        ...
        |""".stripMargin
  
    val debug = true  // 控制添加指令原本信息
    val checkInvalidInst = true  // 检查无效指令
    val exceptionStage = "m1"  // 例外处理在哪一级？
    val bitWidth = 32  // 位宽
    val targetDirectory = "rtl"  // 脚本输出路径
    val defaultInstSet: InstSet = LoongArch32  // 无需修改
  }
  ```


#### 2.2.2 运行脚本

- 进入脚本目录

- 运行sbt命令行

  ```bash
  r$ sbt
  [info] [launcher] getting org.scala-sbt sbt 1.9.3  (this may take some time)...
  [info] [launcher] getting Scala 2.12.18 (for sbt)...
  [info] welcome to sbt 1.9.3 (Ubuntu Java 21.0.3)
  [info] loading project definition from /home/suyang/projects/complex_loong_cpu/src/simple-decoder/project
  [info] loading settings for project root from build.sbt ...
  [info] set current project to simple-decoder (in build file:/home/suyang/projects/complex_loong_cpu/src/simple-decoder/)
  [info] sbt server started at local:///home/suyang/.sbt/1.0/server/0534a0101acb45c8557d/sock
  [info] started sbt server
  sbt:simple-decoder> 
  ```

- 运行`run`命令（第一次运行需等待编译）

  ```bash
  sbt:simple-decoder> run
  
  Multiple main classes detected. Select one to run:
   [1] JsonInstSetLoader
   [2] decoder
  
  Enter number:
  
    | => root / Compile / selectMainClass 6s
  
  ```

- 先运行`[1] JsonInstSetLoader`，加载Json信息

  ```bash
  sbt:simple-decoder> run
  
  Multiple main classes detected. Select one to run:
   [1] JsonInstSetLoader
   [2] decoder
  
  Enter number: 1
  [info] running JsonInstSetLoader
  [Info] Load json files: alu.json, general.json, br.json, csr.json, lsu.json
  [success] Total time: 62 s (01:02), completed Jul 23, 2024, 11:37:07 AM
  ```

- 再运行`[2] decoder`生成解码器

  ```bash
  sbt:simple-decoder> run
  
  Multiple main classes detected. Select one to run:
   [1] JsonInstSetLoader
   [2] decoder
  
  Enter number: 2
  [info] running decoder
  [Runtime] SpinalHDL v1.10.2a    git head : a348a60b7e8b6a455c72e1536ec3d74a2ea16935
  [Runtime] JVM max memory : 1024.0MiB
  [Runtime] Current date : 2024.07.23 11:39:13
  [Progress] at 0.000 : Elaborate components
  [Info] Load json files: alu.json, general.json, br.json, csr.json, lsu.json
  [Info] 解码表
    ...
  [Info] 简化解码表
  	...
  [Info] 简化有效指令表，共xx条
    ...
  [Progress] at 0.295 : Checks and transforms
  [Progress] at 0.361 : Generate Verilog to rtl
  [Done] at 0.393
  [success] Total time: 2 s, completed Jul 23, 2024, 11:39:13 AM
  ```
