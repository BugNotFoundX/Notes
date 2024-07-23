# SoC设计

- 本章以LainSoC为例介绍如何构建片上系统（SoC）
- 代码位于https://github.com/LainChip/LainSoC

## 1 地址空间划分

- 在SoC上，都有很多功能模块的集成，例如SDRAM控制模块，UART模块，GPIO模块等。这个时候，就涉及到各个模块的地址。通俗讲，就是CPU核是如何找到这些模块的呢，是如何配置这些模块相关寄存器，如何向这些模块存取数据？

- 这个时候，关系到SoC片上资源地址空间的划分问题。例如，对于LainSoC来说，地址映射关系如下表：

  | 功能模块    | 起始地址                     | 结尾地址                     |
  | ----------- | ---------------------------- | ---------------------------- |
  | UART        | 0x1FE4_0000                  | 0x1FE4_FFFF                  |
  | SPI         | 0x1FE8_0000<br />0x1C00_0000 | 0x1FE8_0000<br />0x1C0F_FFFF |
  | USB         | 0x1C17_0000                  | 0x1C17_FFFF                  |
  | JpegDecoder | 0x1D11_0000                  | 0x1D11_FFFF                  |
  | I2S         | 0x1C12_0000                  | 0x1C15_FFFF                  |
  | FreamBuffer | 0x1D10_0000                  | 0x1D10_FFFF                  |
  | SD Card     | 0x1FE1_0000                  | 0x1FE1_FFFF                  |
  | DDR3        | 0x0000_0000                  | 0x07FF_FFFF                  |
  | SM3         | 0x1D12_0000                  | 0x1D12_FFFF                  |
  
- 在SoC中，一般使用AXI_DEMUX控制总线数据流向

  ```systemverilog
  // 该函数通过地址比较，判断demux应选择的slaver
  function automatic logic [3:0] periph_addr_sel(input logic [ 31 : 0 ] addr);
    automatic logic [3:0] select;
    if (addr[31:27] == 5'b0) // MIG
      select = 1;
    else if (addr[31:20]==12'h1c0 || addr[31:16]==16'h1fe8) // SPI
      select = 5;
    else if (addr[31:16]==16'h1fe4 /* || addr[31:16]==16'h1fe7 || addr[31:16] == 16'h1fe5 */) // APB
      select = 3;
    else if (addr[31:16]==16'h1c12) // i2s mod
      select = 4;
    else if (addr[31:16]==16'h1c14 || addr[31:16]==16'h1c15) // i2s tx 
      select = 6;
    else if (addr[31:16]==16'h1c17) // USB Host Controller
      select = 10;
    else if (addr[31:16]==16'h1fe1) // SD Controller
      select = 7;
    else if (addr[31:16]==16'h1d10 || addr[31:16]==16'h1d13) // VGA Controller
      select = 8;
    else if (addr[31:16]==16'h1d11) // JPEG Decoder
      select = 9;
    else if (addr[31:16]==16'h1d12) // conf
      select = 2;
    else // ERROR
      select = 0;
    return select;
  endfunction
  
  my_axi_demux_intf #(
    .AXI_ID_WIDTH(4),
    .AXI_ADDR_WIDTH(32),
    .AXI_DATA_WIDTH(32),
    .NO_MST_PORTS(11),
    .MAX_TRANS(2),
    .AXI_LOOK_BITS(2),
    .FALL_THROUGH(1)
  ) cpu_demux (
    .clk_i(soc_clk),
    .rst_ni(soc_aresetn),
    .slv_aw_select_i(periph_addr_sel(cpu_m.aw_addr)),  // 写选择
    .slv_ar_select_i(periph_addr_sel(cpu_m.ar_addr)),  // 读选择
    .slv(cpu_m),       // CPU输出的控制信号
    .mst0(err_s),      // ERROR
    .mst1(mem_m),      // 去往内存控制器
    .mst2(cfg_s),      // 去往CONFREG
    .mst3(apb_s),      // 去往APB总线
    .mst4(i2s_mod_s),  // 去往I2S控制器
    .mst5(spi_s),      // 去往SPI控制器
    .mst6(i2s_tx_s),   // not used
    .mst7(sdc_s),      // 去往SD卡控制器
    .mst8(vga_s),      // 去往FrameBuffer
    .mst9(jpeg_s),     // 去往JpegDecoder
    .mst10(usb_s)      // 去往USB控制器
  );
  ```

## 2 时钟域划分

- LainSoC时钟域划分如下：

  ![lain soc](https://github.com/LainChip/LainSoC/raw/lainsoc_pub/images/lain_soc_new.png)

- 在SoC中不同模块的时钟信号可能不同，例如处理器核心的频率为100MHz，JpegDecoder频率为30MHz，二者之间的数据传输就必须使用时钟域转换模块，否则可能造成数据缺失或亚稳态问题

## 3 IP集成

- SoC中除了CPU以外的几乎所有模块都不需要我们手动实现，这些IP的实现已经超越了体系结构的范畴，不是我们关心的主要问题

- 本文IP或模块汇总：

  | 名称                        | 来源                                                        | 作用                                        |
  | :-------------------------- | ----------------------------------------------------------- | ------------------------------------------- |
  | loongson-blocks             | https://gitee.com/loongson-edu/chiplab/tree/chiplab_diff/IP | 提供UART、CONFREG、SPI控制器                |
  | mczerski-SD-card-controller | https://github.com/mczerski/SD-card-controller              | SD卡控制器                                  |
  | pulp-axi                    | https://github.com/pulp-platform/axi                        | 用于高性能片上通信的 AXI SystemVerilog 模块 |
  | pulp-common_cells           | https://github.com/pulp-platform/common_cells               | 常用模块、头文件                            |
  | ultraembedded-jpeg_decoder  | https://github.com/ultraembedded/core_jpeg_decoder          | JPEG解码器                                  |
  | ultraembedded-usbhost       | https://github.com/ultraembedded/core_usb_host              | USB控制器                                   |
  | Xilinx-AXI-Intc             | Xilinx开源IP                                                | 中断控制器                                  |
  | Xilinx-emaclite             | Xilinx开源IP                                                | 以太网控制器                                |
  | Xilinx-primitive            | Xilinx开源IP                                                | cdc、fifo                                   |
  | ZipCPU-wb2axip              | https://github.com/ZipCPU/wb2axip                           | 总线互连、桥接器和其他组件                  |
  

- 下面以JpegDecoder为例介绍如何集成IP核

- 需要将IP核的顶层转换为进行修饰

  - 完整代码位于：`LainSoC/rtl/SoC/jpeg_decoder_wrapper.sv`

  - 主要任务：

    - 如果需要，对数据流进行跨时钟域转换
    - 将AXI总线打包为SystemVerilog的Interface形式

  - 顶层接口定义

    ```systemverilog
    module jpeg_decoder_wrapper (
      input aclk,         // 100MHz
      input decoder_clk,  // 30MHz
      input aresetn,      // 复位
    
      input dma_clk,      // 120MHz
      input dma_resetn,   // 复位
    
      AXI_BUS.Slave  ctl_slv,  // 来自CPU，控制DMA
      AXI_BUS.Master dma_mst   // 来自DMA，控制内存读写
    );
    ```

  - 接口声明，用于模块之间连线

    ```systemverilog
    `AXI_LINE(ctl_slv_slow);       // 经cdc后的CPU控制信号
    `AXI_LINE(dma_slow);           // cdc前的JpegDecoder的输出信号
    `AXI_LITE_LINE(ctl_slv_lite);  // ctl_slv_slow转换为AXI_LITE总线后的信号
    ```

  - 时钟域转换（Clock Domain Convert，CDC）

    ```systemverilog
    // CPU控制信号CDC
    axi_cdc_intf #(
      .AXI_ID_WIDTH(4),
      .AXI_ADDR_WIDTH(32),
      .AXI_DATA_WIDTH(32),
      .LOG_DEPTH(3)
    ) slv_trans(
      .src_clk_i(aclk),         // 100MHz
      .src_rst_ni(aresetn),     //
      .src(ctl_slv),            // 100MHz下的控制信号
      .dst_clk_i(decoder_clk),  // 30MHz
      .dst_rst_ni(aresetn),     //
      .dst(ctl_slv_slow)        // 30MHz下的控制信号
    );
    
    // DMA输出的内存控制信号CDC
    axi_cdc_intf #(
      .AXI_ID_WIDTH(4),
      .AXI_ADDR_WIDTH(32),
      .AXI_DATA_WIDTH(32),
      .LOG_DEPTH(3)
    ) mst_trans(
      .src_clk_i(decoder_clk),  // 30MHz
      .src_rst_ni(aresetn),     // 
      .src(dma_slow),           // 30MHz下的控制信号
      .dst_clk_i(dma_clk),      // 120MHz
      .dst_rst_ni(dma_resetn),  // 
      .dst(dma_mst)             // 120MHz下的控制信号
    );
    ```

  - 总线协议转换

    - JpegDecoder的控制信号为AXI-Lite协议

    ```systemverilog
    xil_axi_to_axi_lite_wrapper axi2axil_mod (
      .aclk(decoder_clk),
      .aresetn(aresetn),
      .slv(ctl_slv_slow),  // AXI协议控制信号
      .mst(ctl_slv_lite)   // AXI_LITE协议控制信号
    );
    ```

  - 实例化JpegDecoder并连线

    ```systemverilog
    jpeg_decoder # (
      .AXI_ID(0)
    )
    jpeg_decoder_inst (
      .clk_i(decoder_clk),
      .rst_i(!aresetn),
      // 连接来自CPU的控制信号
      .cfg_awvalid_i (ctl_slv_lite.aw_valid),
      .cfg_awaddr_i  (ctl_slv_lite.aw_addr),
      .cfg_arvalid_i (ctl_slv_lite.ar_valid),
      ...
    	// 连接DMA输出信号
      .outport_awvalid_o (dma_slow.aw_valid),
      .outport_awaddr_o  (dma_slow.aw_addr),
      .outport_awid_o    (dma_slow.aw_id),
      ...
    );
    ```
