# PA0

#### 关于推送

dsm@cubic:~/ysyx-workbench/nemu$ git remote -v
master	git@github.com:Dsmical/ysyx.git (fetch)
master	git@github.com:Dsmical/ysyx.git (push)
origin	git@github.com:OSCPU/ysyx-workbench.git (fetch)
origin	git@github.com:OSCPU/ysyx-workbench.git (push)

+ **Compiling and Running NEMU**中缺少三个工具。安装即可！

```
sudo apt-get install bison
sudo apt-get install flex
sudo apt-get install libncurses5-dev
```

+ 文档搜索使用`/内容`n跳转到下一个匹配，q退出

## Verilator 安装

这份安装说明提供了在 Git 上安装 Verilator 的简要步骤。下面是一些关键点的说明和可能的修改：

1. **Prerequisites（先决条件）：**

   ```bash
   bashCopy codesudo apt-get install git help2man perl python3 make autoconf g++ flex bison ccache
   sudo apt-get install libgoogle-perftools-dev numactl perl-doc
   sudo apt-get install libfl2 libfl-dev  # 如果出现错误，可以忽略
   sudo apt-get install zlibc zlib1g zlib1g-dev  # 如果出现错误，可以忽略
   ```

2. **Clone Git 仓库：**

   ```bash
   git clone git@github.com:verilator/verilator.git   # 仅第一次
   ```

3. **每次构建：**

   ```bash
   git pull         # 确保 Git 仓库是最新的
   git tag          # 查看存在的版本
   ```

4. **选择版本：** 你可以根据需要选择不同的版本，比如：

   ```bash
   checkout master      # 使用开发分支（最新的 bug 修复）
   git checkout stable      # 使用最新的稳定版本
   git checkout v{version}  # 切换到指定的发布版本
   ```

   这里选择指定的版本`git checkout v5.008`

5. **构建和安装：**

   ```bash
   autoconf         # 创建 ./configure 脚本
   ./configure      # 配置并创建 Makefile
   make -j `nproc`  # 构建 Verilator 本身（如果出现错误，尝试只运行 'make'）
   sudo make install
   ```

这些步骤基本上是正确的，但具体操作可能会因为系统配置、网络状态、权限等原因而有所不同。确保按照文档中提供的操作进行，并根据需要进行适当的调整。

6. **安装后查看版本，大功告成**

   ```bash
   verilator --version
   ```

   

### verilator使用

[It's Embedded! (itsembedded.com)](https://www.itsembedded.com/)

+ 一般来说就是.v文件和一个c++的testbench文件。需要包含头文件`verilated.h`和`V$(模块名).h`固定的初始化，赋初值，`eval()`更新模块状态。

```c++
#include <verilated.h>
#include "Vswitch.h"

int main(int argc, char** argv) {
    // Initialize Verilator and trace
    Verilated::commandArgs(argc, argv);
    Vswitch* s = new Vswitch;
    

    // Run simulation for 10 clock cycles
    for (int cycle = 0; cycle < 10; ++cycle) {
        // Generate random inputs
        int a = rand() & 1;
        int b = rand() & 1;

        // Set inputs
        s->a = a;
        s->b = b;

        // Evaluate the module
        s->eval();

        // Print the results
        printf("Cycle %d: a = %d, b = %d, f = %d\n", cycle, a, b, s->f);

        // Advance the clock (assuming a simple rising edge-triggered clock)
  
        
    }

    // Cleanup
    delete s;
    exit(0);
}
```

+ Makefile文件编写,主要就是8—10行的三条命令

  + `verilator -Wall --trace -cc alu.sv --exe tb_alu.cpp` 

    这会将我们的`alu.sv`源代码转换为 C++ 并生成用于构建模拟可执行文件的构建文件。我们用来`-Wall`启用所有 C++ 错误、`--trace`启用波形跟踪、`-cc alu.sv`将 alu.sv 模块转换为 C++，并`--exe tb_alu.cpp`告诉 Verilator 哪个文件是我们的 C++ 测试平台。

  + `make -C obj_dir -f Valu.mk Valu` 

    这将根据测试平台和转换后的源构建我们的模拟可执行文件。我们告诉 Make 将工作目录更改为`obj_dir`，使用名为 的构建文件`Valu.mk`并构建名为 的目标`Valu`

  + `./obj_dir/Valu` 

    这将运行我们的模拟可执行文件，该可执行文件模拟测试台并生成我们的波形。

```bash
all:
	@echo "Write this Makefile by your self."

sim:
	$(call git_commit, "sim RTL") # DO NOT REMOVE THIS LINE!!!
	@echo "Write this Makefile by your self."
	
	verilator -Wall --trace --cc switch.v --exe tb_switch.cpp
	make -C obj_dir -f Vswitch.mk Vswitch
	./obj_dir/Vswitch
clean:
	rm -rf obj_dir/
include ../Makefile
```

+ 阅读verilator编译出的C++代码, 然后结合verilog代码, 尝试理解仿真器进行仿真的时候都发生了什么.

  + **Verilator 仿真流程：**

    1. **初始化：**
       - 创建 `Vtop___024root` 的实例。
       - 初始化信号状态，例如设置输入信号 `a` 和 `b` 的初值。
    2. **仿真循环：**
       - 调用 `Vtop___024root` 的 `eval` 函数，该函数包含了 Verilog 模块中的所有逻辑。
       - 仿真器通过循环处理敏感信号和时钟边沿，执行相应的逻辑。
    3. **敏感信号和时钟边沿：**
       - 当输入信号 `a` 或 `b` 发生变化时，或者在时钟的上升/下降沿，Verilog 中定义的 `always` 块将被触发执行。
    4. **内部逻辑计算：**
       - 在 `eval` 函数中，Verilog 模块的内部逻辑（例如 `always_comb` 中的逻辑）被计算。
       - 在这个例子中，`f` 的值由 `a` 和 `b` 的异或操作计算得到。
    5. **触发敏感信号更新：**
       - 通过调用相应的 `_eval` 函数，更新敏感信号的状态。
       - 这包括 `_eval_ico`、`_eval_act`、`_eval_nba` 等函数，用于更新输入信号的状态。
    6. **Triggers 和 Debug（调试）：**
       - Triggers 用于检测输入信号是否在一定迭代次数内稳定，否则会触发错误。
       - Debug 相关的函数用于输出调试信息，例如在调试模式下，输出触发的敏感信号。
    7. **VCD 跟踪：**
       - 使用 Verilated VCD 类，将模拟过程中的信号变化记录到 VCD 文件中。
    8. **仿真终止：**
       - 当仿真达到预定的时间或其他条件时，仿真终止。

    通过这个过程，Verilator 实现了对 Verilog 模块的硬件描述的仿真。生成的 C++ 代码模拟了硬件的行为，允许我们在软件层面验证硬件描述的正确性。

### **实现一个带有逻辑运算的简单ALU**

重点就是两点，

+ **第一个就是最小负数溢出问题。**

比如四位有符号数最小负数-8，补码表示为1000.(相反数的补码，就是原来补码取反+1)也是1000.这样会导致溢出位判断的时候出现问题。 所以方法一是对的。先取反保证符号正确，求和的时候再将1加上。

方法一：

```verilog
assign t_no_Cin = {n{ Cin }}^B;
assign {Carry,Result} = A + t_no_Cin + Cin;
assign Overflow = (A[n-1] == t_no_Cin[n-1]) && (Result [n-1] != A[n-1]);
```

方法二：

```verilog
assign t_add_Cin =( {n{Cin}}^B )+ Cin;  //  在这里请注意^运算和+运算的顺序
assign { Carry, Result } = A + t_add_Cin;
assign Overflow = (A[n-1] == t_add_Cin[n-1]) && (Result [n-1] != A[n-1]);
```

+ **第二个就是减法过程中减数为0，会造成错误进位**

主要就是b=0，取反之后b_comp=1111。直接判断 a+b_comp+{1'b0,instr}必然会进位，所以需要中间变量 value =  b_comp + {1'b0,instr}将固有的进位消除。

```
  /*加减法*/
    3'd0, 3'd1: begin
    /*不能把a+b_comp+{1'b0,instr}直接赋给{CF,value},否则当b=0时,b_comp+1会错误的进位*/
    value =  b_comp + {1'b0,instr};
    {CF,value} = value +a;
```

+ 完整的代码展示

```verilog
module seg(
  /*输入输出定义,输入4位编码值,输出7位数码管的显示控制*/
  input  [3:0] b,
  output reg [6:0] h
);
/*每当b发生变化,都会重新更新输出*/
always @(*) begin
  	case(b[3:0])
      4'd0 : h = ~7'b0111111;
      4'd1 : h = ~7'b0000110;
      4'd2 : h = ~7'b1011011;
      4'd3 : h = ~7'b1001111;
      4'd4 : h = ~7'b1100110;
      4'd5 : h = ~7'b1101101;
      4'd6 : h = ~7'b1111101;
      4'd7 : h = ~7'b0000111;
      4'd8 : h = ~7'b1111111;
      4'd9 : h = ~7'b1101111;
      4'd14 : h = 7'b0111111;//负号
      default : h = 7'b1111111;//空
    endcase
end
endmodule

module Button(
    input  [2:0]button,
    output reg [2:0]mode,
	output reg enable);
 always @( posedge button[1]or posedge button[0]or posedge button[2] ) begin
		if(button[2]) begin
			enable <= ~enable;
		end
        else 
			case(button)
			3'b010:  begin mode <= mode+1; enable <= 0;end
			3'b001:  begin mode <= mode-1; enable <= 0;end
			default:   enable <= 1;
			endcase
	end
endmodule


module Alu(
	input [3:0] a_in,
	input [3:0] b_in,
	input [2:0] button,
	
	output reg OF,//溢出标志
	output reg CF,//进位标志
	output reg ZF,//0标志
	output reg [2:0] mode,
	output [6:0] mode_seg,
	output [13:0] a_seg,
	output [13:0] b_seg,
	output [20:0] value_seg
);


wire en;
wire [3:0]a= a_in[3]? {a_in[3],~a_in[2:0]+3'b1}:a_in;
wire [3:0]b= b_in[3]? {b_in[3],~b_in[2:0]+3'b1}:b_in;
wire [3:0]b_comp = {4{mode[0]}}^b;//根据操作码来对b进行处理，0：加法b_comp=b,1:减法b_comp=b按位取反
reg [3:0] value;				 //注意①：这里为什么不是按位取反+1（-b）。因为考虑到最小负数溢出问题，+1会在后面求和中添加					

Button Button_1 (.button(button),.mode(mode),.enable(en));
seg seg7(.b({1'b0,mode}),.h(mode_seg));

seg seg6(.b({1'b1,a_in[3],2'b10}),.h(a_seg[13:7]));
seg seg5(.b({1'b0,a_in[2:0]}),.h(a_seg[6:0]));

seg seg4(.b({1'b1,b_in[3],2'b10}),.h(b_seg[13:7]));
seg seg3(.b({1'b0,b_in[2:0]}),.h(b_seg[6:0]));

seg seg2(.b(value_sign),.h(value_seg[20:14]));
seg seg1(.b(value_num[7:4]),.h(value_seg[13:7]));
seg seg0(.b(value_num[3:0]),.h(value_seg[6:0]));

 always @(*) begin
	if (en) begin
		case (mode)
		3'd0, 3'd1: begin
			value =  b_comp + {1'b0,mode};//注意②不能把a+b_comp+{1'b0,instr}直接赋给{CF,value},否则当b=0时,b_comp+1会错误的进位/
			{CF,value} = value +a;
			OF = (a[3] == b_comp[3]) && (value[3] != a[3]);
			end
		3'd2 : begin  value = ~a;    {CF,  OF} =2'd0;  end
		3'd3 : begin  value = a & b; {CF,  OF} =2'd0;  end
		3'd4 : begin  value = a | b; {CF,  OF} =2'd0;  end
		3'd5 : begin  value = a ^ b; {CF,  OF} =2'd0;  end
		3'd6 : begin  value = {3'b000,a<b}; {CF, OF} =2'd0; end
		3'd7 : begin  value = {3'b000,a==b};{CF, OF} =2'd0; end
		endcase	
		ZF=value==0;
		end
	else begin
    value = 0;
    {CF, ZF, OF} =3'd0;
	end
end

wire [3:0] value_abs;
wire [3:0] value_sign;
wire [7:0] value_num;

assign value_abs = value[3] ?(~value+1):value;
assign value_sign =  {1'b1,value[3],2'b10};
assign value_num[7:4]=value_abs/10;
assign value_num[3:0]=value_abs % 10;

endmodule

```

### 有限状态机

请设计一个区别两种特定时序的有限状态机FSM：该有限状态机有一个输入w和一个输出z。当w是4个连续的0或4个连续的1时，输出z=1，否则z=0，时序允许重叠。即：若w是连续的5个1时，则在第4个和第5个时钟之后，z均为1。

+ **Mealy（米里）型有限状态机**

Mealy有限状态机的输出不仅仅与状态机的当前状态有关，而且与输入信号的当前值也有关。这使得Mealy有限状态机对输入的响应发生在当前时钟周期。

```verilog
module FSM_mealy
(
  input   clk, in, reset,
  output  out
);

parameter[3:0] S0 = 0, S1 = 1, S2 = 2, S3 = 3,
          S4 = 4, S5 = 5, S6 = 6, S7 = 7, S8 = 8;

reg [3:0] state;

always @(posedge clk or posedge reset) begin
  if (reset)
    state <= S0;
  else begin
    case(state)
      S0: state <= (in == 0) ? S1 : S5;
      S1: state <= (in == 0) ? S2 : S5;
      S2: state <= (in == 0) ? S3 : S5;
      S3: state <= (in == 0) ? S4 : S5;
      S4: state <= (in == 0) ? S4 : S5;
      S5: state <= (in == 1) ? S6 : S1;
      S6: state <= (in == 1) ? S7 : S1;
      S7: state <= (in == 1) ? S8 : S1;
      S8: state <= (in == 1) ? S8 : S1;
      default: state <= S0;
    endcase
  end
end

assign out = (state == S4) || (state == S8);

endmodule
```

+ **Moore有限状态机**

最重要的特点就是将输入与输出信号隔离开来。输入对输出的影响要到下一个时钟周期才能反映出来即Moore 型有限状态机的输出信号是直接由状态寄存器译码得到

```verilog
module FSM_bin
(
  input   clk, in, reset,
  output reg out
);

parameter[3:0] S0 = 0, S1 = 1, S2 = 2, S3 = 3,
          S4 = 4, S5 = 5, S6 = 6, S7 = 7, S8 = 8;

wire [3:0] state_din, state_dout;
wire state_wen;

SimReg#(4,0) state(clk, reset, state_din, state_dout, state_wen);

assign state_wen = 1;

MuxKeyWithDefault#(9, 4, 1) outMux(.out(out), .key(state_dout), .default_out(0), .lut({
  S0, 1'b0,
  S1, 1'b0,
  S2, 1'b0,
  S3, 1'b0,
  S4, 1'b1,
  S5, 1'b0,
  S6, 1'b0,
  S7, 1'b0,
  S8, 1'b1
}));

MuxKeyWithDefault#(9, 4, 4) stateMux(.out(state_din), .key(state_dout), .default_out(S0), .lut({
  S0, in ? S5 : S1,
  S1, in ? S5 : S2,
  S2, in ? S5 : S3,
  S3, in ? S5 : S4,
  S4, in ? S5 : S4,
  S5, in ? S6 : S1,
  S6, in ? S7 : S1,
  S7, in ? S8 : S1,
  S8, in ? S8 : S1
}));

endmodule
```

## NEMU代码框架

##### 介绍一下GDB的基本使用

+ `r`重新开始执行程序
+ `s`单步执行一行源代码/`n`类似但不进入函数(可用于跳过库函数)

+ `finish`- 执行直到当前函数返回

+ `p`-打印变量或寄存器的值
+ `X`-扫描内存

  + 格式: `x /nfu 地址`
  + `n`表示要显示的内存单元的个数
  + `f`表示显示方式, 可取如下值：

    x 按十六进制格式显示变量。 d 按十进制格式显示变量。 u 按十进制格式显示无符号整型。 o 按八进制格式显示变量。 t 按二进制格式显示变量。 a 按十六进制格式显示变量。 i 指令地址格式 c 按字符格式显示变量。 f 按浮点数格式显示变量
  + `u`表示一个地址单元的长度：

    b表示单字节， h表示双字节， w表示四字节， g表示八字节

+ `bt`- 查看调用栈
+ `b`-设置断点/`watch`- 设置监视点
+ `help` xxx-查看xxx命令的帮助

##### Makefile阅读

```makefile
# Sanity check
#wildcard函数检查该文件是否存在，错误则输出错误信息
ifeq ($(wildcard $(NEMU_HOME)/src/nemu-main.c),)
  $(error NEMU_HOME=$(NEMU_HOME) is not a NEMU repo)
endif

# Include variables and rules generated by menuconfig
-include $(NEMU_HOME)/include/config/auto.conf
-include $(NEMU_HOME)/include/config/auto.conf.cmd

#定义了一个 Makefile 中的函数 remove_quote，该函数用于去除字符串中的双引号
#$(patsubst pattern,replacement,text)，
#$(1) 表示函数的第一个参数
remove_quote = $(patsubst "%",%,$(1))

# Extract variabls from menuconfig
#call函数在Makefile中可以用来调用自定义函数
GUEST_ISA ?= $(call remove_quote,$(CONFIG_ISA))
ENGINE ?= $(call remove_quote,$(CONFIG_ENGINE))
NAME    = $(GUEST_ISA)-nemu-$(ENGINE)

# Include all filelist.mk to merge file lists
#-L选项表示对符号链接进行解引用，即如果./src是一个符号链接，则会搜索它指向的实际目录。
FILELIST_MK = $(shell find -L ./src -name "filelist.mk")
include $(FILELIST_MK)

# Filter out directories and files in blacklist to obtain the final set of source files
DIRS-BLACKLIST-y += $(DIRS-BLACKLIST)
SRCS-BLACKLIST-y += $(SRCS-BLACKLIST) $(shell find -L $(DIRS-BLACKLIST-y) -name "*.c")
SRCS-y += $(shell find -L $(DIRS-y) -name "*.c")
SRCS = $(filter-out $(SRCS-BLACKLIST-y),$(SRCS-y))

# Extract compiler and options from menuconfig
CC = $(call remove_quote,$(CONFIG_CC))
CFLAGS_BUILD += $(call remove_quote,$(CONFIG_CC_OPT))
CFLAGS_BUILD += $(if $(CONFIG_CC_LTO),-flto,)
CFLAGS_BUILD += $(if $(CONFIG_CC_DEBUG),-Og -ggdb3,)
CFLAGS_BUILD += $(if $(CONFIG_CC_ASAN),-fsanitize=address,)
CFLAGS_TRACE += -DITRACE_COND=$(if $(CONFIG_ITRACE_COND),$(call remove_quote,$(CONFIG_ITRACE_COND)),true)
CFLAGS  += $(CFLAGS_BUILD) $(CFLAGS_TRACE) -D__GUEST_ISA__=$(GUEST_ISA)
LDFLAGS += $(CFLAGS_BUILD)

# Include rules for menuconfig
include $(NEMU_HOME)/scripts/config.mk

ifdef CONFIG_TARGET_AM
include $(AM_HOME)/Makefile
LINKAGE += $(ARCHIVES)
else
# Include rules to build NEMU
include $(NEMU_HOME)/scripts/native.mk
endif
```

##### 函数解读

+ ###### **getopt函数**

  + 定义：

    ```c
    int getopt(int argc, char * const argv[], const char *optstring);
    ```

  + 描述：

    ```c
    getopt是用来解析命令行选项参数的，但是只能解析短选项: -d 100,不能解析长选项：--prefix
    ```

  + 参数：

    ```c
    argc：main()函数传递过来的参数的个数
    argv：main()函数传递过来的参数的字符串指针数组
    optstring：选项字符串，告知 getopt()可以处理哪个选项以及哪个选项需要参数
    ```

  + 返回

    ```c
    如果选项成功找到，返回选项字母；如果所有命令行选项都解析完毕，返回 -1；如果遇到选项字符不在 optstring 中，返回字符 '?'；如果遇到丢失参数，那么返回值依赖于 optstring 中第一个字符，如果第一个字符是 ':' 则返回':'，否则返回'?'并提示出错误信息。
    ```

  + 参数

    ```c
    optarg —— 指向当前选项参数(如果有)的指针。
    optind —— 再次调用 getopt() 时的下一个 argv指针的索引。
    optopt —— 最后一个未知选项。
    opterr —— 如果不希望getopt()打印出错信息，则只要将全域变量opterr设为0即可。
    ```

  + 实例

    ```c
    #include<stdio.h>
    #include<unistd.h>
    #include<getopt.h>
    int main(intargc, char *argv[])
    {
        int opt;
        char *string = "a::b:c:d";
        while ((opt = getopt(argc, argv, string))!= -1)
        {  
            printf("opt = %c\t\t", opt);
            printf("optarg = %s\t\t",optarg);
            printf("optind = %d\t\t",optind);
            printf("argv[optind] = %s\n",argv[optind]);
        }  
     //   运行 gcc -o test test.c
    //   ./test  -a100 -b 200 -c 300 -d
    ```

+ ###### **getopt_long函数**

  + 定义：

    ```c
    int getopt_long(int argc, char * const argv[], const char *optstring, const struct option *longopts,int *longindex);
    ```

  + 描述：

    ```c
    包含 getopt 功能，增加了解析长选项的功能如：--prefix --help
    ```

  + 参数

    ```c
    longopts    指明了长参数的名称和属性
    longindex   如果longindex非空，它指向的变量将记录当前找到参数符合longopts里的第几个元素的描述，即是longopts的下标值
    ```

  + 返回

    ```c
    对于短选项，返回值同getopt函数；对于长选项，如果flag是NULL，返回val，否则返回0；对于错误情况返回值同getopt函数
    ```

  + struct option

    ```c
    struct option {
    const char  *name;       /* 参数名称 */
    int          has_arg;    /* 指明是否带有参数 */
    int          *flag;      /* flag=NULL时,返回value;不为空时,*flag=val,返回0 */
    int          val;        /* 用于指定函数找到选项的返回值或flag非空时指定*flag的值 */
    }; 
    ```

  + 参数说明：

    ```c
    has_arg  指明是否带参数值，其数值可选：
    no_argument         表明长选项不带参数，如：--name, --help
    required_argument  表明长选项必须带参数，如：--prefix /root或 --prefix=/root
    optional_argument  表明长选项的参数是可选的，如：--help或 –prefix=/root，其它都是错误
    ```

  + 实例

    ```c
    int main(intargc, char *argv[])
    {
        int opt;
        int digit_optind = 0;
        int option_index = 0;
        char *string = "a::b:c:d";
        static struct option long_options[] =
        {  
            {"reqarg", required_argument,NULL, 'r'},
            {"optarg", optional_argument,NULL, 'o'},
            {"noarg",  no_argument,         NULL,'n'},
            {NULL,     0,                      NULL, 0},
        }; 
        while((opt =getopt_long_only(argc,argv,string,long_options,&option_index))!= -1)
        {  
            printf("opt = %c\t\t", opt);
            printf("optarg = %s\t\t",optarg);
            printf("optind = %d\t\t",optind);
            printf("argv[optind] =%s\t\t", argv[optind]);
            printf("option_index = %d\n",option_index);
        }  
    }
    ```


+ **Log** 

  ```c
  #define Log(format, ...) _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n",__FILE__, __LINE__, __func__, ## __VA_ARGS__)//相关的宏在utils.h中定义
  ```

  + `Log()`宏与`printf()`类似, 但进行了如下增强
    
    + 可以输出当前位置,`__FILE__, __LINE__, __func__` - C语言提供的预定义宏`__func__`其实是个预定义的变量
    + 可以输出颜色 ANSI escape code(`man console_codes`)
    + 还可以自动换行
    
  + Log的实现
  
    + 颜色实现通过定义的`ANSI_FMT`宏实现
    + 后续就是打印输出
    + 以及写入日志文件
  
    ```bash
    #define ANSI_BG_BLUE    "\33[1;44m"
    #define ANSI_BG_MAGENTA "\33[1;35m"
    #define ANSI_BG_CYAN    "\33[1;46m"
    #define ANSI_BG_WHITE   "\33[1;47m"
    #define ANSI_NONE       "\33[0m"
    
    #define ANSI_FMT(str, fmt) fmt str ANSI_NONE
    
    #define log_write(...) IFDEF(CONFIG_TARGET_NATIVE_ELF, \
      do { \
        extern FILE* log_fp; \
        extern bool log_enable(); \
        if (log_enable()) { \
          fprintf(log_fp, __VA_ARGS__); \
          fflush(log_fp); \
        } \
      } while (0) \
    )
    
    #define _Log(...) \
      do { \
        printf(__VA_ARGS__); \
        log_write(__VA_ARGS__); \
      } while (0)
    
    ```
  

##### sscanf函数和**fscanf函数**

```c
int sscanf(const char *str, const char *format, ...);
//示例
 sscanf("abcde","%4s",str);
     str="abcd\0"
//fscanf是从文件读取，sscanf字符串读取
```



##### make相关

```makefile
make -nB  # -B可以强制make构建所有目标, 即使它们已经是最新的
make -nB | vim -
```

输出了好多信息, 在vim中稍作处理提升可读性

```bash
`make -nB | vim -` 表示将 `make -nB` #命令的输出通过管道传递给 `vim -` 命令，`-` 表示将管道传递的内容作为 `vim` 的输入。
```

```Makefile
# 只保留gcc或g++开头的行
:%!grep "^\(gcc\|g++\)"

# 将环境变量$NEMU_HOME所指示字符串替换为$NEMU_HOME
:%!sed -e "s+$NEMU_HOME+\$NEMU_HOME+g"

# 将$NEMU_HOME/build/obj-riscv32-nemu-interpreter替换为$OBJ_DIR
:%s+\$NEMU_HOME/build/obj-riscv32-nemu-interpreter+$OBJ_DIR+g

# 将-c之前的内容替换为$CFLAGS
:%s/-O2.*=riscv32/$CFLAGS/g

# 将最后一行的空格替换成换行并缩进两格
:$s/  */\r  /g
```



##### 搜索

```bash
grep -nr "\bmain\b" nemu/src
```

+ `grep`：搜索文本的命令

+ `-nr`：

  + `-n`：显示匹配行的行号
  + `-r`：递归搜索指定目录及其子目录下的文件

+ `"\bmain\b"`：这是要搜索的文本模式或正则表达式，其中 `\b` 表示单词边界，`main`是搜索的单词

+ `nemu/src`：搜索路径

  

##### c语言中正则表达式处理

标准的C和C++都不支持正则表达式，但有一些函数库可以辅助C/C++程序员完成这一功能，其中最著名的当数Philip Hazel的Perl-Compatible Regular Expression库，许多Linux发行版本都带有这个函数库。

C语言处理正则表达式常用的函数有regcomp()、regexec()、regfree()和regerror()，一般分为三个步骤，如下所示：

+ ###### regcomp()编译正则表达式

  ```c
  int regcomp (regex_t *compiled, const char *pattern, int cflags)
  //这个函数把指定的正则表达式pattern编译成一种特定的数据格式compiled，这样可以使匹配更有效。函数regexec 会使用这个数据在目标文本串中进行模式匹配。执行成功返回０。 　
  ```

  参数说明：
  ①regex_t 是一个结构体数据类型，用来存放编译后的正则表达式，它的成员re_nsub 用来存储正则表达式中的子正则表达式的个数，子正则表达式就是用圆括号包起来的部分表达式。
  ②pattern 是指向我们写好的正则表达式的指针。
  ③cflags 有如下4个值或者是它们或运算(|)后的值：
  REG_EXTENDED 以功能更加强大的扩展正则表达式的方式进行匹配。
  REG_ICASE 匹配字母时忽略大小写。
  REG_NOSUB 不用存储匹配后的结果。
  REG_NEWLINE 识别换行符，这样’$’就可以从行尾开始匹配，’^’就可以从行的开头开始匹配。

+ ######  regexec()匹配正则表达式

  ```c
  int regexec (regex_t *compiled, char *string, size_t nmatch, regmatch_t matchptr [], int eflags)
  //当我们编译好正则表达式后，就可以用regexec 匹配我们的目标文本串了，如果在编译正则表达式的时候没有指定cflags的参数REG_NEWLINE，则默认情况下是忽略换行符的，也就是把整个文本串当作一个字符串处理。执行成功返回０。regmatch_t 是一个结构体数据类型，在regex.h中定义：
  typedef struct
  {
     regoff_t rm_so;
     regoff_t rm_eo;
  } regmatch_t;
  //成员rm_so 存放匹配文本串在目标串中的开始位置，rm_eo 存放结束位置。通常我们以数组的形式定义一组这样的结构。因为往往我们的正则表达式中还包含子正则表达式。数组0单元存放主正则表达式位置，后边的单元依次存放子正则表达式位置。
  ```

  参数说明：
  ①compiled 是已经用regcomp函数编译好的正则表达式。
  ②string 是目标文本串。
  ③nmatch 是regmatch_t结构体数组的长度。
  ④matchptr regmatch_t类型的结构体数组，存放匹配文本串的位置信息。
  ⑤eflags 有两个值
  REG_NOTBOL 按我的理解是如果指定了这个值，那么’^’就不会从我们的目标串开始匹配。总之我到现在还不是很明白这个参数的意义；
  REG_NOTEOL 和上边那个作用差不多，不过这个指定结束end of line。

+ ###### regfree()释放正则表达式

  ```c
  void regfree (regex_t *compiled)
  //当我们使用完编译好的正则表达式后，或者要重新编译其他正则表达式的时候，我们可以用这个函数清空compiled指向的regex_t结构体的内容，请记住，如果是重新编译的话，一定要先清空regex_t结构体。
  ```

+ ###### regerror()报错信息

  ```c
  size_t regerror (int errcode, regex_t *compiled, char *buffer, size_t length)
  //当执行regcomp 或者regexec 产生错误的时候，就可以调用这个函数而返回一个包含错误信息的字符串。
  ```

  参数说明：
  ①errcode 是由regcomp 和 regexec 函数返回的错误代号。
  ②compiled 是已经用regcomp函数编译好的正则表达式，这个值可以为NULL。
  ③buffer 指向用来存放错误信息的字符串的内存空间。
  ④length 指明buffer的长度，如果这个错误信息的长度大于这个值，则regerror 函数会自动截断超出的字符串，但他仍然会返回完整的字符串的长度。所以我们可以用如下的方法先得到错误字符串的长度。

##### 调试总结：

对于nemu可以打开

Build Options
  [*] Enable address sanitizer

事实上, GCC还支持更多的sanitizer, 它们可以检查各种不同的错误, 你可以在man gcc中查阅-fsanitize相关的选项. 如果你的程序在各种sanitizer开启的情况下仍然能正确工作, 就说明你的程序还是有一定质量的.

##### makefile自动变量

1. `$@`：表示规则的目标文件名。
2. `$<`：表示规则的第一个依赖文件名。
3. `$^`：表示规则的所有依赖文件名列表，以空格分隔。
4. `$?`：表示规则中所有比目标文件更新的依赖文件名列表，以空格分隔。
5. `$*`：表示规则的目标文件名去除后缀的部分。
6. `$(@D)`：表示规则的目标文件所在的目录路径。
7. `$(@F)`：表示规则的目标文件的文件名部分（不包含路径）。
8. `$(<D)`：表示规则的第一个依赖文件所在的目录路径。
9. `$(<F)`：表示规则的第一个依赖文件的文件名部分（不包含路径）

## PA2

### PA2.2

##### 修改Makefile通过批处理模式运行nemu

```bash
思路：首先RTFSC，在nemu/src/monitor/monitor.c中的parse_args解析函数函数中存在一个批处理模式的匹配，当参数为-b时会调用函数sdb_set_batch_mode打开批处理模式，而parse_args的参数又来自main函数，因此，只需要在对应的AM的Makefile中在运行nemu时传入一个 -b的参数就可以打开批处理模式。
在AM的Makefile代码框架中，第四步会选择对应的架构配置
这里选择nemu对应的架构mk文件$(AM_HOME)/script/riscv32-nemu.mk,发现如下Makefile内容：
发现运行nemu的mk不在这里，再进入运行nemu时的mk，进入$(AM_HOME)/script/platform/nemu.mk
可以看到这里就是运行nemu的mk所在，我们只需要在运行nemu时传入一个-b参数就能打开批处理模式，因此在NEMUFLAGS后加入一个-b就能打开批处理模式了。
NEMUFLAGS += -l $(shell dirname $(IMAGE).elf)/nemu-log.txt -b
在/home/dsm/ysyx-workbench/am-kernels/tests/cpu-tests路径下运行make ARCH=riscv32-nemu ALL=add run
```


