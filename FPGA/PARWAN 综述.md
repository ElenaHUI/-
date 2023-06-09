[TOC]

## PARWAN 综述

PARWAN处理器是一个非常简单的处理器，但是却包含了指令操作的基本功能，用简单的硬件构架和必要的程序语言，构建出了一个较为复杂的CPU结构。

#### 1.指令操作

PARWAN CPU的支持直接寻址和间接寻址，指令也可以是单字节指令或者双字节指令。

#### 2.主要架构

- **AC寄存器：**是唯一一个用户可以直接操作的寄存器，可以将数据写入AC寄存器中，并载入到ALU运算器。
- **IR是指令寄存器：**存放当前取指得到的指令
- **ALU是运算单元**：指定简单的加法、减法和逻辑与操作
- **SHU是移位寄存器**：可以对ALU输出的数据进行移位操作，也可以不进行操作，仅当做数据传递的中间件
- **SR**：用于记录ALU运算得到的标志位
- **CONTROLLER**：核心控制器，它有九个状态，通过状态转移关系来发出控制信号，进而完成CPU的各种应用功能
- **PC地址寄存器**：用于记录下一条指令的地址，分为页地址和偏移量地址两部分
- **MAR寄存器**：用于存储内存地址，它也分为页地址和偏移量两部分
- **内存**：4KB，是PARWAN CPU的外设内存

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609184626104.png" alt="image-20230609184626104" style="zoom:50%;" />

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609203351019.png" alt="image-20230609203351019" style="zoom:50%;" />

#### 3.内存管理

- Page Address

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609185240690.png" alt="image-20230609185240690" style="zoom:50%;" />

PARWAN CPU的内存地址使用12位，而数据线只有8位，因此需要进行内存的分页管理。为了实现分页，我们将12位的地址总线划分为4位的页号和8位的偏移量，这样可以得到16个256字节的页。对于完整的地址（Full Address），我们需要使用12位的内存地址。

在这个基础上，我们可以进一步细分内存寻址方式。例如，JSR指令和分支指令的寻址方式是受限的。这两个指令都是双字节指令，其中第二字节存放地址偏移量，而第一字节只包含指令的操作码。对于地址的页号，直接采用当前指令所在的页号，即PC地址寄存器中的页号。通过当前指令的页号和下一字节的偏移量，我们可以组合得到完整的12位地址。这种只能在页内寻址的方式称为页面寻址（Page Address），如上图所示。

- 直接寻址和间接寻址



**直接寻址：**将指令第一字节的后四位和第二字节拼在一起得到12位的地址。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609185434401.png" alt="image-20230609185434401" style="zoom:50%;" />

**间接寻址：**把指令第一字节的后四位和第二字节拼凑得到12位的地址，根据这个地址找到内存中的某一个字节，这个字节存储的8位数据就是地址的偏移量，将这个偏移量和第一字节后四位的页号拼凑在一起得到最终的12位地址。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609185559541.png" alt="image-20230609185559541" style="zoom:50%;" />

#### 4.指令集

17个用户可能用到的指令如下所示：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609185639010.png" alt="image-20230609185639010" style="zoom:50%;" />

指令具体编码方式作如下图所示：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609192812939.png" alt="image-20230609192812939" style="zoom:50%;" />

#### 5.功能模块

**ALU（算术逻辑单元）**是用于执行算术和逻辑运算的组件。它具有三个输入端口。`a_side`和`b_side`是数据输入端口，ALU可以对这两个输入的数据执行加法、减法和逻辑与操作。ALU具有两个输出端口。8位的`alu_out`端口输出运算的结果，而4位的`alu_flags`端口输出运算的标志位。标志位可以通过SHU和SR重新输入到ALU，这构成了ALU的第三个输入端口。

ALU具有六根控制线，用于控制其行为。它们分别是逻辑运算控制线alu_and、alu_not，数据载入指令控制线`alu_a`、`alu_b`，以及加法和减法运算指令控制线`alu_add`、`alu_sub`

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609192947986.png" alt="image-20230609192947986" style="zoom: 33%;" />

**SHU（移位单元）**是用于执行移位操作的组件，它具有两个输入端口。它可以接收来自ALU运算的8位结果，并通过移位运算将结果输出到总线OBUS。此外，SHU还可以接收来自ALU的4位标志位输出，但不进行任何操作，直接传递给SR（状态寄存器）。由于SHU执行移位操作，因此它具有两根控制线。这两根控制线的控制信号分别用于控制左移和右移操作。如果控制信号为1，则执行相应的移位操作，如果为0，则不进行移位。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609193047876.png" alt="image-20230609193047876" style="zoom:50%;" />


**SR（状态寄存器）**用于存储ALU输出的状态标志位，并可以将其传输回ALU。SR具有四位，分别表示四种状态。这四位分别是：

V：当V为1时，表示运算出现了溢出。 C：当C为1时，表示加法产生了进位或减法没有产生借位。 Z：当Z为1时，表示运算的结果为0。 N：当N为1时，表示运算的结果为负数。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609193640539.png" alt="image-20230609193640539" style="zoom:50%;" />
**PC（程序计数器）**用于存储下一条要执行的指令的地址。在PARWAN CPU中，由于地址是12位的，因此PC也是12位的。它被分为4位的页存储（PC_PAGE）和8位的偏移量存储（PC_OFFSET）。

PC具有两个输入端口，分别从MAR_PAGE和MAR_OFFSET接收地址数据。同时，PC具有三个输出端口。其中两个端口将页地址和偏移量地址分别输出到`mar_page_bus`和`mar_offset_bus`总线上。第三个输出端口直接将偏移量地址输出到DBUS总线上。这样的设计方便了部分指令直接获取偏移量地址的需要。例如，JSR指令需要将PC的偏移量地址存储到子程序的第一字节，通过这种方式可以直接获取PC的偏移量地址。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609193720801.png" alt="image-20230609193720801" style="zoom:50%;" />

**MAR（存储器地址寄存器）**用于存放内存地址信息。类似于PC，MAR也由页地址和偏移量地址组成。MAR的地址可以直接输出到PC中使用。同时，MAR具有两个输出端口，这些端口可以将地址进行聚合。它们可以将4位的页地址和8位的偏移量地址组合成12位的内存地址，该内存地址可以直接输出到ADBUS（地址总线）中。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609194004351.png" alt="image-20230609194004351" style="zoom:50%;" />


**IR（指令寄存器）**从OBUS（数据总线）接收输入，并将指令存储在其中。IR能够将八位指令输出到状态机CONTROLLER中，以便状态机根据输入指令的内容进行逻辑判断和分析。

同时，IR还可以将指令的低四位单独输出到mar_page_bus总线上。该总线可以将地址传递给MAR_PAGE。这样设计的原因是对于双字节的Full Address指令，指令的第一字节的后四位存放着寻址的页号。IR可以将页号地址单独输出到MAR_PAGE中，从而有助于进一步的寻址操作。

IR还具有一根控制线，该控制线负责将OBUS中的数据载入到IR中。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609194103637.png" alt="image-20230609194103637" style="zoom:50%;" />

**状态控制机（CONTROLLER）**中存放着PARWAN CPU设计的9种状态转移方式。根据输入的IR指令，状态控制机确定状态的转移，并输出相关的控制信号。因此，CONTROLLER是控制整个CPU行为的核心，它是各种控制指令的根源。

由于控制指令较多，在图中并没有将所有的控制信号从CONTROLLER引出，而是将各个控制信号分配到各自控制的设备上。图中只展示了两条控制信号：read_mem和write_mem。这两根控制信号分别用于控制内存的读取和写入操作。此外，还有一个外部的interrupt控制信号。当interrupt控制信号为1时，用于初始化CONTROLLER。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609194120223.png" alt="image-20230609194120223" style="zoom:50%;" />

#### 6.总线


PARWAN CPU中存在6个主要的总线，它们负责数据的传递和连接整个数据通路。

1. **ADBUS**（外部地址总线）：12位宽，连接到外部存储设备。ADBUS负责处理器与内存之间的直接地址交互。其中，控制线`mar_on_adbus`用于控制将MAR的12位地址输入到ADBUS中。
2. **DATABUS**（外部数据总线）：8位宽，用于从外部存储设备获取数据。数据会输出到内部的数据总线DBUS中。控制线`dbus_on_databus`用于控制DBUS向DATABUS输入数据。
3. **DBUS**（内部数据总线）：8位宽，DBUS的数据可以直接输入到ALU的a_side端口。DBUS具有三个输入端口，分别受到三个控制信号的控制。其中，控制线`databus_on_dbus`用于将DATABUS的数据输入到DBUS；控制线`obus_on_dbus`用于将OBUS的数据输入到DBUS；控制线`pc_offset_on_dbus`用于将PC_OFFSET的数据输入到DBUS。
4. **OBUS**（内部输出总线）：8位宽，用于接收ALU的输出，并可以将数据输入到AC、IR和DBUS中。OBUS没有控制线，它直接连接到相应的组件。
5. **MAR_PAGE_BUS**（内部地址总线的页地址总线）：直接连接到MAR_PAGE，用于传输页地址。它具有两条控制线，其中控制线`ir_on_mar_page`用于将IR的低四位输入到MAR_PAGE_BUS；控制线`pc_on_mar_page_bus`用于将PC_PAGE输入到MAR_PAGE_BUS。
6. **MAR_OFFSET_BUS**（内部地址总线的偏移量总线）：直接连接到MAR_OFFSET，用于传输偏移量。它具有两条控制线，其中控制线`pc_on_mar_offset_bus`用于将PC_OFFSET输入到MAR_OFFSET_BUS。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609194340763.png" alt="image-20230609194340763" style="zoom: 67%;" />

#### 7.指令执行

​	该部分是对PARWAN CPU指令执行流程的概述，由于不同指令的格式不同，执行方式不同，这里仅对指令执行的关键步骤进行大概的展示。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195019118.png" alt="image-20230609195019118" style="zoom: 33%;" />

- ##### 取第一字节

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195129147.png" alt="image-20230609195129147" style="zoom: 67%;" />

- ##### 取指令第二字节（直接寻址）

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195146342.png" alt="image-20230609195146342" style="zoom:67%;" />

- ##### 取指令第二字节（间接寻址）

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195217697.png" alt="image-20230609195217697" style="zoom:67%;" />

- ##### 数据的运算

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195237499.png" alt="image-20230609195237499" style="zoom:67%;" />

- ##### 写回AC数据到内存

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195304172.png" alt="image-20230609195304172" style="zoom:67%;" />

- ##### 跳转

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195330062.png" alt="image-20230609195330062" style="zoom:67%;" />

- ##### 写入子程序的返回地址

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195354436.png" alt="image-20230609195354436" style="zoom:67%;" />

指令执行的过程中，会涉及到各种控制信号对CPU总线和功能模块的操作，这些控制信号是从CONTROLLER中的不同状态中发出的。

#### 8.状态迁移关系

CONTROLLER模块内部存储着9个状态的迁移关系，这9个状态共同控制了PARWAN CPU的执行过程，相关的控制信号都能在上述指令执行的部分找到对应的执行过程，状态转换关系如下图所示：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609195531809.png" alt="image-20230609195531809" style="zoom:50%;" />

从电路图可以看到状态是通过D触发器实现的，D触发器的特点是，在下一次触发之前，触发器的内容会保留上一次的值，对于每个D触发器，当输入1时会迁移到该状态，如果多个状态都能迁移到同一状态，那么多个状态的输入通过或门连接，只要有一个输出为1，那么就可以迁移到该状态。

状态迁移的基本实现原理是通过D触发器实现的，对于每个D触发器，当输入1时会迁移到该状态，如果多个状态都能迁移到同一状态，那么多个状态的输入通过或门连接，只要有一个输出为1，那么就可以迁移到该状态。

下面是对各个状态的简单介绍：

##### S1：载入指令地址

该状态一般是初始状态，用于取指令的准备阶段。S1状态的下一个状态是S2，因此S2执行的就是从地址取指令的行为；S6，S8，S9都能转移到S1。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609200207279.png" alt="image-20230609200207279" style="zoom:50%;" />

##### S2：取指令

S2状态完成指令的后续取指工作。S2状态结束后会进入S3，S3就是指令第一字节的执行。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609200415839.png" alt="image-20230609200415839" style="zoom:50%;" />

##### S3：执行指令的第一字节


观察指令的编码构成可以得出以下结论：

- 如果指令的高四位不是1110，那么该指令是双字节指令，需要进一步处理。在电路图中，这种情况下状态会转移到S4。
- 如果指令的高四位是1110，则指令是单字节指令。指令会通过与门向下传递，并根据指令的低四位输出不同的指令操作控制信号。从电路图中可以看出，指令的执行依赖于对指令逐位的判断，并搭配逻辑门实现。只有当指令位符合逻辑门的输入时，才会触发对应行为的执行。这符合现在普遍的译码规则。

此外，我们还可以观察到以下情况：

- 对于单字节指令，在S3状态结束后会转移到S2。在S2状态中，执行下一条指令的取指工作。无论是否是单字节指令，S3状态都会发送控制信号，将PC地址读取并载入MAR中。这部分与S1状态的行为完全相同，即获取下一条指令的地址并存入MAR。
- 这样的操作方式是有利的。如果是单字节指令，通过这一步我们无需转移到S1，直接转移到S2即可。如果是双字节指令，无论是间接寻址还是直接寻址，都需要PC_OFFSET的8位地址。而S3状态已经将PC_OFFSET的地址载入MAR_OFFSET，后续只需要将IR中的后四位载入即可。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609200535533.png" alt="image-20230609200535533" style="zoom: 50%;" />

##### S4：双字节指令的执行

S3将双字节指令转移到S4，S4判断如果需要直接寻址或者间接寻址，就通过发送ir_on_mar_page_bus控制信号，将IR寄存器中后四位载入到mar_page_bus中，再加载到MAR_PAGE中。

S4状态后如果是直接寻址会转移到S6，如果是间接寻址会转移到S5，JSR指令会转移到S7，branch指令会转移到S9

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609200715962.png" alt="image-20230609200715962" style="zoom: 50%;" />

##### S5：间接寻址

在S5状态，控制信号将MAR的地址传送到ADBUS，并从内存中读取数据。读取的数据将再次存储在MAR_OFFSET中。

S5状态转移到S6的原因是，在间接寻址中，第一步是获取数据的地址，接下来需要从该地址中取出实际数据。而对于直接寻址，直接得到了数据的地址。因此，在S5状态完成第一步获取地址后，进入S6状态执行后续指令操作，包括直接寻址的执行

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609201647010.png" alt="image-20230609201647010" style="zoom:50%;" />

##### S6：直接寻址

在S6状态，根据指令不同位的信息进行指令译码，并根据结果转移到不同的状态。下面是对几个特定指令的说明：

- 对于指令"st"（STA指令），控制信号将AC的数据写回内存，并将地址载入PC，然后转移到S2状态开始下一条指令的执行。
- 对于指令"rd"，控制信号将从内存读取的数据载入AC，并执行运算操作。这些指令执行完后会返回到S1状态，准备执行下一条指令。
- 对于指令"jm"，由于已经载入了PC地址，直接进入S2状态，开始下一条指令的执行。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609201724512.png" alt="image-20230609201724512" style="zoom:50%;" />

##### S7：JSR指令写入返回地址

在S7状态，控制信号使得返回地址被写入子程序的第一字节中。由于S7是从S4状态转移而来，此时MAR已经存储了子程序的入口地址。通过控制信号，PC的返回地址通过DBUS写入MAR的地址位置，即子程序的第一字节处。同时，子程序的偏移地址也被加载到PC_OFFSET中。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609202043518.png" alt="image-20230609202043518" style="zoom:50%;" />

##### S8：JSR指令的后续工作

S8将PC的地址加1，并转移到1状态，因为根据S7状态PC_OFFSET就存储着子程序第一字节的偏移量地址，加一后将地址指向了子程序执行部分的第一条指令，然后转移到状态S1，开始执行子程序中的指令。

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609202141520.png" alt="image-20230609202141520" style="zoom:50%;" />

##### S9：分支指令

S9处理四种分支指令，通过指令的低四位判断分支指令的类型，如果需要跳转就载入MAR的偏移量地址到PC_OFFSET中，最后回到S1，开始执行下一条指令

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609202207341.png" alt="image-20230609202207341" style="zoom:50%;" />

综上所述状态转移图如下所示：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609202243923.png" alt="image-20230609202243923" style="zoom:50%;" />

## VHDL设计和仿真

parwan的数据流描述如下：

<img src="C:\Users\111\AppData\Roaming\Typora\typora-user-images\image-20230609203748631.png" alt="image-20230609203748631" style="zoom: 67%;" />

仿真部分选择对Parwan Control Unit 进行仿真，通过在网站上查找到的资料和代码在本地电脑进行了模拟和仿真

资料来源

1. https://kipdf.com/chapter-10-cpu-modeling-and-design_5aeffbbc7f8b9ae95e8b463c.html
2. http://www.altera.co.kr/_hdl/1/RESOURCES/www.ece.neu.edu/info/vhdl/Sanders/sanders_df/par_ctrl.html



根据题目要求，我们选择设计状态s1-s6整理后编写vhdl设计代码如下：

```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use work.synthesis_utilities.ALL;

ENTITY par_control_unit IS 
	  --定义CONTROLLER模块
     PORT (clk : IN std_logic; 
     --定义加载控制信号
     load_ac, zero_ac, 
     load_ir, 
     increment_pc, load_page_pc, load_offset_pc, reset_pc, 
     load_page_mar, load_offset_mar, 
     load_sr, cm_carry_sr, 
     -- 定义总线传输控制信号
     pc_on_mar_page_bus, ir_on_mar_page_bus, 
     pc_on_mar_offset_bus, dbus_on_mar_offset_bus, 
     pc_offset_on_dbus, obus_on_dbus, databus_on_dbus, 
     mar_on_adbus, 
     dbus_on_databus, 
     --定义移位控制信号 
     arith_shift_left, arith_shift_right : OUT std_logic; 
     --定义ALU计算控制信号 
     alu_and,alu_not,alu_a,alu_add,alu_b,alu_sub : out std_logic;
     --定义两个输入信号
     ir_lines : IN byte; 
     status : IN nibble; 
     --定义内存读写信号
     read_mem, write_mem : OUT std_logic; 
     interrupt : IN std_logic; 
     present_state_value: OUT nibble
     ); 

END par_control_unit; 
--
ARCHITECTURE dataflow_synthesizable OF par_control_unit IS 
	  --定义九种状态
     TYPE cpu_states IS (s1,s2,s3,s4,s5,s6,s7,s8,s9); 
     SIGNAL present_state, next_state : cpu_states; 
     SIGNAL next_state_value: std_logic_vector(3 downto 0);

BEGIN 
     --定义时序
     clocking : PROCESS (clk, interrupt) 
     BEGIN 
          IF (interrupt = '1') THEN 
               present_state <= s1; 
               present_state_value <="0001";
          ELSIF clk'EVENT AND clk = '0' THEN 
               present_state <= next_state; 
               present_state_value<=next_state_value;
          END IF; 
     END PROCESS clocking; 
     sequencing : PROCESS ( present_state, ir_lines, status,
     interrupt) 
     BEGIN 
	       --定义控制信号的初始值
          load_ac <= '0'; zero_ac <= '0'; load_ir <= '0';
          increment_pc <= '0'; 
          load_page_pc <= '0'; load_offset_pc <= '0'; reset_pc <= '0'; 
          load_page_mar <= '0'; load_offset_mar <= '0'; 
          load_sr <= '0'; cm_carry_sr <= '0'; 
			 
          pc_on_mar_page_bus <= '0'; ir_on_mar_page_bus <= '0'; 
          pc_on_mar_offset_bus <= '0'; dbus_on_mar_offset_bus <= '0'; 
          pc_offset_on_dbus <= '0'; obus_on_dbus <= '0';
          databus_on_dbus <= '0'; 
          mar_on_adbus <= '0'; dbus_on_databus <= '0'; 
			 
          arith_shift_left <= '0'; arith_shift_right <= '0'; 
			 
          alu_and<='0';alu_not<='0';alu_a<='0';alu_add<='0';alu_b<='0';alu_sub<='0'; 
			 
          read_mem <= '0'; write_mem <= '0'; 
			 
			 --定义状态转移的条件判断语句
          CASE present_state IS 
               WHEN s1 =>
               -------------------------------------------
                    IF (interrupt = '1') THEN 
                         reset_pc <= '1'; 
                         next_state <= s1; 
                         next_state_value<="0001";
                    ELSE 
                         --输出该状态对应的控制信号
                         pc_on_mar_page_bus <= '1'; 
                         pc_on_mar_offset_bus <= '1'; 
                         load_page_mar <= '1'; 
                         load_offset_mar <= '1'; 
                         next_state <= s2; 
                         next_state_value<="0010";
                    END IF; 
               WHEN s2 =>
               ---------------------------------------
                    mar_on_adbus <= '1'; 
                    read_mem <= '1'; 
                    databus_on_dbus <= '1'; 
                    alu_a<='1';
                    load_ir <= '1'; 
                    increment_pc <= '1'; 
                    next_state <= s3; 
                    next_state_value<="0011";
               WHEN s3 =>
               --------------------------------------
                    pc_on_mar_page_bus <= '1'; 
                    pc_on_mar_offset_bus <= '1'; 
                    load_page_mar <= '1'; 
                    load_offset_mar <= '1'; 
                    IF (ir_lines (7 DOWNTO 4) /=
                    "1110") THEN 
                         next_state <= s4; 
                         next_state_value<="0100";
                    ELSE 
                         CASE ir_lines (3 DOWNTO 0) IS 
                              WHEN "0001" => 
                                   zero_ac <= '1'; 
                                   load_ac <= '1'; 
                              WHEN "0010" =>   
                                   alu_not<='1';
                                   load_sr <= '1'; 
                                   load_ac <= '1'; 
                              WHEN "0100" => 
                                   cm_carry_sr <= '1'; 
                              WHEN "1000" =>
                                   alu_b<='1';
                                   arith_shift_left <= '1'; 
                                   load_sr <= '1'; 
                                   load_ac <= '1'; 
                              WHEN "1001" =>   
                                   alu_b<='1';
                                   arith_shift_right <= '1'; 
                                   load_sr <= '1'; 
                                   load_ac <= '1'; 
                              WHEN OTHERS => NULL; 
                         END CASE; 
                         next_state <= s2; 
                         next_state_value<="0010";
                    END IF; 
               WHEN s4 =>
               ----------------------------------------
                    mar_on_adbus <= '1'; 
                    read_mem <= '1'; 
                    databus_on_dbus <= '1'; 
                    dbus_on_mar_offset_bus <= '1'; 
                    load_offset_mar <= '1'; 
                    IF ( ir_lines (7 DOWNTO 6) /=
                         "11" ) THEN 
                              ir_on_mar_page_bus <= '1'; 
                              load_page_mar <= '1'; 
                              IF ( ir_lines (4) = '1' )
                              THEN 
                                   next_state <= s5; 
                                   next_state_value<="0101";
                              ELSE 
                                   next_state <= s6; 
                                   next_state_value<="0110";
                              END IF; 
                         ELSE 
                              IF ( ir_lines (5) = '0' ) THEN 
                                   next_state <= s7; 
                                   next_state_value<="0111";
                              ELSE 
                                   next_state <= s9; 
                                   next_state_value<="1001";
                              END IF; 
                         END IF; 
                         increment_pc <= '1'; 
               WHEN s5 =>
               ---------------------------------------
                    mar_on_adbus <= '1'; 
                    read_mem <= '1'; 
                    databus_on_dbus <= '1'; 
                    dbus_on_mar_offset_bus <= '1'; 
                    load_offset_mar <= '1'; 
                    next_state <= s6; 
                    next_state_value<="0110";
               WHEN s6 =>
               --------------------------------------
                    IF ( ir_lines (7 DOWNTO 5) = "100" ) THEN 
                         load_page_pc <= '1'; 
                         load_offset_pc <= '1'; 
                         next_state <= s2; 
                         next_state_value<="0010";
                    ELSIF ( ir_lines (7 DOWNTO 5) = "101" ) THEN 
                         mar_on_adbus <= '1'; 
                         alu_b<='1';
                         obus_on_dbus <= '1'; 
                         dbus_on_databus <= '1'; 
                         write_mem <= '1'; 
                         next_state <= s1; 
                         next_state_value<="0001";
                    ELSIF ( ir_lines (7) = '0' ) THEN 
                         mar_on_adbus <= '1'; 
                         read_mem <= '1'; 
                         databus_on_dbus <= '1'; 
                         IF ( ir_lines (6) = '0' ) THEN
                              IF ( ir_lines (5) = '0' )
                                   THEN 
                                        alu_a<='1'; 
                                   ELSE 
                                        alu_and<='1'; 
                                   END IF; 
                        ELSE 
                                   IF ( ir_lines (5) = '0' )
                                   THEN 
                                       alu_add<='1'; 
                                   ELSE 
                                        alu_sub<='1'; 
                                   END IF; 
                        END IF; 
                              load_sr <= '1'; 
                              load_ac <= '1'; 
                              next_state <= s1; 
                              next_state_value<="0001";
                    END IF; 
          END CASE; 
     END PROCESS; 

END dataflow_synthesizable;

```

设计完成后，为了展示状态转移的过程，模拟了指令并进行循环执行进行测试，仿真结果如下：

<img src="C:\Users\111\Pictures\Camera Roll\11.jpg"  />

可以看到状态在s1->s2->s3->s4->s6->s1之间执行

<img src="C:\Users\111\Pictures\图片2.png" style="zoom: 80%;" />

可以看到在240ns处`alu_sub`控制信号被置位，表示指令执行的开始，仿真测试成功，说明设计状态成功。