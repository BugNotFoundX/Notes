# SoC设计

- 本章以MegaSoC为例介绍如何构建片上系统（SoC）

- 代码位于https://github.com/BUAA-CI-LAB/MegaSoC

- 整体架构如图：

  ![百芯部署整体框架图](https://github.com/BUAA-CI-LAB/MegaSoC/raw/master/picture/megasoc.drawio.png)

## 1 地址空间划分

- 在SoC上，都有很多功能模块的集成，例如SDRAM控制模块，UART模块，GPIO模块等。这个时候，就涉及到各个模块的地址。通俗讲，就是CPU核是如何找到这些模块的呢，是如何配置这些模块相关寄存器，如何向这些模块存取数据？

- 这个时候，关系到SoC片上资源地址空间的划分问题。例如，对于MegaSoC来说，地址映射关系如下表：

- 以下是该款SoC的地址划分表：

- AXI4 Peripheral Address Mapping

| Address Range                                            | Device                    | Select Value |
| -------------------------------------------------------- | ------------------------- | ------------ |
| `0x00000000 - 0x0FFFFFFF`                                | Memory Controller         | 0            |
| `0x1C000000 - 0x1C0FFFFF`<br />`0x1D000000 - 0x1D00FFFF` | SPI Device                | 1            |
| `0x1D000000 - 0x1D0FFFFF`                                | AXI-Lite Device           | 2            |
| `0x1D100000 - 0x1D1FFFFF`                                | USB Controller            | 3            |
| Other Addresses                                          | AXI-Lite Device (Default) | 2            |
- AXI4-Lite Peripheral Address Mapping:

| Address Range             | Device                         | Select Value |
| ------------------------- | ------------------------------ | ------------ |
| `0x1D100000 - 0x1D3FFFFF` | APB Devices (UART, I2C, CDBUS) | 1            |
| `0x1D400000 - 0x1D4FFFFF` | Configuration Registers        | 2            |
| `0x1D500000 - 0x1D5FFFFF` | Ethernet Controller            | 3            |
| `0x1D600000 - 0x1D6FFFFF` | Interrupt Controller           | 4            |
| `0x1D700000 - 0x1D7FFFFF` | SD Controller                  | 5            |
| `0x1DA00000 - 0x1DAFFFFF` | JPEG Controller                | 6            |
| `0x1DB00000 - 0x1DBFFFFF` | I2S Controller-0（DMA）          | 7            |
| `0x1DC00000 - 0x1DCFFFFF` | I2S Controller-1（GEN）          | 8            |
| `0x1DD00000 - 0x1DDFFFFF` | VGA Controller                 | 9            |
| Other Addresses           | SRAM Controller (Default)      | 0            |

- 在SoC中，一般使用AXI_DEMUX控制总线数据流向
```systemverilog
// 该函数通过地址比较，判断demux应选择的slaver
function automatic logic [1:0] axi4_periph_addr_sel(input logic [ 31 : 0 ] addr);
  automatic logic [1:0] select;
  if (addr[31:28] == 4'b0000)     // Memory Controller == 256MB
    select = 0;
  else if (addr[31:20]==12'h1c0 || addr[31:16] == 16'h1d00)
    select = 1;                 // SPI device
  else if (addr[31:20]==12'h1d0)  // all axi-lite device
    select = 2;
  else if (addr[31:20]==12'h1d1)  // USB Controller
    select = 3;
  else  // ERROR
    select = 2;
  return select;
endfunction

my_axi_demux_intf #(
  .AXI_ID_WIDTH(4),
  .AXI_ADDR_WIDTH(32),
  .AXI_DATA_WIDTH(32),
  .NO_MST_PORTS(4),
  .MAX_TRANS(2),
  .AXI_LOOK_BITS(2)
) cpu_demux (
  .clk_i(soc_clk),
  .rst_ni(aresetn),
  .test_i(1'b0),
  .slv_aw_select_i(axi4_periph_addr_sel(cpu_m.aw_addr)),
  .slv_ar_select_i(axi4_periph_addr_sel(cpu_m.ar_addr)),
  .slv(cpu_m),
  .mst0(mem_m),
  .mst1(spi_s),
  .mst2(axilite_axi4_m),  // lite
  .mst3(usb_s)
);
```

## 2 IP集成

- SoC中除了CPU以外的几乎所有模块都不需要我们手动实现，这些IP的实现已经超越了体系结构的范畴，不是我们关心的主要问题
- 本文IP或模块汇总：

| 名称                       | 来源                                                         | 作用                                        |
| :------------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| loongson-blocks            | https://gitee.com/loongson-edu/chiplab/tree/chiplab_diff/IP  | 提供UART、CONFREG、SPI控制器                |
| pulp-axi                   | https://github.com/pulp-platform/axi                         | 用于高性能片上通信的 AXI SystemVerilog 模块 |
| pulp-common_cells          | https://github.com/pulp-platform/common_cells                | 常用模块、头文件                            |
| register_interface         | https://github.com/pulp-platform/register_interface          | 通用寄存器接口                              |
| ultraembedded-jpeg_decoder | https://github.com/ultraembedded/core_jpeg_decoder           | JPEG解码器                                  |
| Xilinx-AXI-Intc            | Xilinx开源IP                                                 | 中断控制器                                  |
| Xilinx-emaclite            | Xilinx开源IP                                                 | 以太网控制器                                |
| Xilinx-primitive           | Xilinx开源IP                                                 | cdc、fifo                                   |
| Xilinx-AXIAHB              | Xilinx开源IP                                                 | AXI-AHB总线桥                               |
| ZipCPU-wb2axip             | https://github.com/ZipCPU/wb2axip                            | 总线互连、桥接器和其他组件                  |
| i2c-slave                  | https://github.com/freecores/i2cslave                        | I2C从设备                                   |
| opencores-i2c              | https://github.com/fabriziotappero/ip-cores/tree/communication_controller_i2c_controller_core#vhdlverilog-ip-cores-repository | I2C主设备                                   |
| axi_to_i2s                 | 自主设计                                                     | AXI控制的I2S输出模块                        |
| axi-hdmi                   | 自主设计                                                     | SII-146芯片控制器                           |
| dukelec-cdbus_ip           | https://github.com/dukelec/cdbus                             | CDBUS控制器                                 |
| axi-sdc                    | https://github.com/mczerski/SD-card-controller               | SD卡控制器                                  |

- 下面以FrameBuffer为例介绍如何集成IP核，其他模块大同小异，主要的操作逻辑是一样的
- 完整代码位于：https://github.com/BUAA-CI-LAB/MegaSoC/blob/master/General/SoC/fb_wrapper.sv
- 主要任务：
	- 如果需要，对数据流进行跨时钟域转换
	- 将AXI总线打包为SystemVerilog的Interface形式
- 顶层接口定义

```systemverilog
module fb_wrapper (
  input aclk,
  input aresetn,

  input vgaclk,
  input vgaresetn,

  output wire [7:0] vga_r_o,
  output wire [7:0] vga_g_o,
  output wire [7:0] vga_b_o,
  output wire vga_de_o,
  output wire vga_h_o,
  output wire vga_v_o,

  AXI_LITE.Slave   slv,
  AXI_BUS.Master   dma_mst
);
```

- 接口声明，用于模块之间连线

```systemverilog
AXI_BUS  #(.AXI_ADDR_WIDTH(32), .AXI_DATA_WIDTH(32), .AXI_ID_WIDTH(4)) vga_dma();  // cdc前的FrameBuffer的输出信号
AXI_LITE #(.AXI_ADDR_WIDTH(32), .AXI_DATA_WIDTH(32)) slv_lite();                   // 经cdc后的CPU控制信号
```

  - 时钟域转换（Clock Domain Convert，CDC）

```systemverilog
axi_lite_cdc_intf #(
  .AXI_ADDR_WIDTH(32),
  .AXI_DATA_WIDTH(32),
  .LOG_DEPTH(2)
) cfg_slv_cdc (
  .src_clk_i(aclk),      // SoC时钟
  .src_rst_ni(aresetn),  //
  .src(slv),             // CDC前CPU控制信号

  .dst_clk_i(vgaclk),    // VGA时钟
  .dst_rst_ni(vgaresetn),//
  .dst(slv_lite)         // CDC后的CPU控制信号
);

axi_cdc_intf #(
  .AXI_ID_WIDTH(4),
  .AXI_ADDR_WIDTH(32),
  .AXI_DATA_WIDTH(32),
  .LOG_DEPTH(4)
) dma_mst_cdc (
  .src_clk_i(vgaclk),    // VGA时钟
  .src_rst_ni(vgaresetn),//
  .src(vga_dma),         // VGA输出的DMA控制信号

  .dst_clk_i(aclk),      // SoC时钟
  .dst_rst_ni(aresetn),  //
  .dst(dma_mst)          // CDC后的DMA控制信号
); 
```

  - 实例化FrameBuffer并连线

```systemverilog
vga_framebuffer # (
  .VIDEO_WIDTH(1280),
  .VIDEO_HEIGHT(720),
  .VIDEO_REFRESH(24),
  .VIDEO_ENABLE(1),
  .VIDEO_X2_MODE(0)
)
vga_framebuffer_inst (
  .clk_i(vgaclk),
  .rst_i(~vgaresetn),
  // CPU控制信号
  .cfg_awvalid_i(slv_lite.aw_valid),
  .cfg_awaddr_i (slv_lite.aw_addr),
  .cfg_wvalid_i (slv_lite.w_valid),
  ...
  // DMA控制信号
  .outport_awready_i(vga_dma.aw_ready),
  .outport_wready_i(vga_dma.w_ready),
  .outport_bvalid_i(vga_dma.b_valid),
  ...
  // 中断信号
  .intr_o(intr),
  // vga输出
  .vga_r_o(vga_r_o),
  .vga_g_o(vga_g_o),
  .vga_b_o(vga_b_o),
  .vga_de_o(vga_de_o),
  .vga_h_o(vga_h_o),
  .vga_v_o(vga_v_o)
);
```
