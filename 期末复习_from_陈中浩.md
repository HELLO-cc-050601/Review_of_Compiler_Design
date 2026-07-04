# 第一章 绪论

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627155525604.png" alt="image-20260627155525604" style="zoom:67%;" />

## 编译的概念

计算机中的编译就是将 <u>**<font color='blue'>高级语言</font>**</u> 翻译长 <u>**<font color='blue'>汇编语言或机器语言</font>**</u> 的过程

符编译器将标识符及其各种属性放到了**<font color='green'>符号表</font>**中。符号表为每一个标识符保存一个**<font color='green'>记录的数据结构</font>**，记录的域是标识符的各个属性。



## 编译器的几个基本阶段，及各阶段的功能

前端也通常称为分析部分，与源语言有关

后端也通常称为综合部分，与目标语言相关

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627153054094.png" alt="image-20260627153054094" style="zoom:80%;" />

- **词法分析**：从左向右逐行扫描源程序的字符，识别出各个单词、确定**<font color='red'>单词的类型</font>**。将识别出的单词转换为统一的机内表示：**<font color='red'>词法记号token</font>**（**<font color='blue'>token:\<记号名，属性值\></font>**）

  - 将正确的单词识别为记号
  - 剥去注释和空白（制表符、回车符、空格等分隔符）
  - 将各阶段产生的错误信息和源程序联系起来
  - ~~如果源程序支持宏，则宏的预处理可以在词法分析时实现~~（现代编译器往往有单独的预处理）

- **语法分析**：从词法分析器输出的token序列中**<font color='red'>识别出各类短语</font>**，并**<font color='red'>构造语法树</font>**（语法树syntax tree不是AST，注意区分！）

- **语义分析**：包括收集标识符的属性信息、类型检查、语义一致性检查、隐式类型转换等。目前比较流行的是使用”**<font color='green'>语法制导翻译</font>**“将语法分析和语义分析有机地组织起来

- **中间代码生成**：生成后缀表示（后缀表达式）、图形表示（DAG）、<u>三地址代码</u>（我称其为人性化的汇编）

- **代码优化***：进行等价程序变换

- **代码生成***：由中间代码生成目标代码（汇编代码或机器代码）

  

## 编译器阶段的分组：前端、后端；遍

#### 前端

也称为分析部分。与源语言相关，包含词法分析、语法分析、语义分析、中间代码生成

#### 后端

也称为综合部分，与目标语言相关，包含代码生成、代码优化（依赖机器）

#### 遍

- 几个阶段的活动可能被组合起来形成一”遍”
- 每一个“遍“都会读入一个输入文件（例如源程序或源程序的中间表示），并写一个输出文件

> 前端的四个阶段可以组合形成一”遍“
>
> 代码优化可以进行很多”遍“



## 编译器、解释器区别

编译器的工作原理如下

<img src="C:\Users\czh\AppData\Roaming\Typora\typora-user-images\image-20260627155506632.png" alt="image-20260627155506632" style="zoom: 67%;" />



解释器的工作原理如下

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627155140852.png" alt="image-20260627155140852" style="zoom:67%;" />

- 编译器会生成目标程序，运行时直接执行机器指令
- 解释器不会生成慕目标程序，而是将源代码直接在目标机器上运行。执行时，解释器读取一句源代码之后，先进行词法分析和语法分析，再将源代码转换为解释器能够执行的中间代码（字节码），最后，由解释器将中间代码解释为可执行的机器指令。

---



# 第二章 词法分析

![image-20260627223126210](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627223126210.png)

## 相关概念

### 词法记号和属性

#### 词法记号（记号）

- 也即是token，具体内容为 \<记号名，属性值>，其中属性值可选

- 记号名即记号分类，是语法分析的输入符号。确定记号名没有原则性规定，取决于处理上的方便。

#### 模式

- 描述一个词法记号的单词可能具有的**<font color='blue'>形式</font>**。当词法记号是一个关键字时，它的模式就是组成这个关键字的字符序列；当词法记号是一个标识符或其他，模式是一个更加复杂的结构，可以和很多字符串匹配

> 想想刷字符串算法题的时候，我们往往给匹配的字符串起名为pattern，就可以类比下这个编译原理的模式，也就是我们上面说的词法记号是一个关键字；对应到正则表达式是可以的

#### 单词

又叫词法单元，它和**<font color='blue'>某个词法记号的模式匹配</font>**，是该记号的**实例**



> 总之，我们可以看作，记号代表某一大类字符串，同一个记号的字符串满足某种模式。这一大串满足某种模式的字符串的其中一个字符串就是单词。

#### 词法记号的属性

如果有多个单词与一个模式匹配，那么词法分析器必须向编译器的后续阶段提供有关被匹配单词的附加信息。

属性值可能是某个具体的值；**<font color='orange'>指向符号表中某条目的指针</font>**（属性可能是一个结构化数据，包含单词、类型、第一次出现的位置等等）

#### 词法错误

词法分析只能发现单词内部”拼写“类错误。包括**“紧急方式”的错误恢复**（删除直到正确）和**错误修补**（最小改动策略：包括删一个、插一个、正确代替不正确、交换相邻）

### 词法记号的描述和识别

#### 字母表

<font color='green'>**符号的有限集合**</font>，用符号 $\Sigma$ 表示。例如ASCII字符集、英文字母表

#### 串

串是字母表中符号的 一个<font color='blue'>**有穷序列**</font>。串的长度是串中符号的个数（所以空串即长度为0的串，注意空格也是一个符号，所以全是空格的串不是空串）

串有（真）前缀、（真）后缀、（真）子串；具有**<font color='purple'>连接</font>**（concat）、**<font color='purple'>积</font>**（指数、幂，具体来说是$s=s^1,s^2$就是将s重复两次）等操作

#### 语言

**<font color='green'>字母表上的一个串集</font>**。例如所有语法正确的C程序集合，所有语法正确的英语句子等。

具有**<font color='purple'>Union</font>**、**<font color='purple'>Concat</font>**（两个集合求concat是笛卡尔积）、**<font color='purple'>指数</font>**（自己和自己求笛卡尔积，例如$\{0,1\}^3$就是长度为3的由0,1组成的串），**<font color='purple'>闭包</font>**，**<font color='purple'>正闭包</font>**（不包括空串）等运算。

![image-20260627165445186](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627165445186.png)

#### <font color='red'>正规式（重点：根据语义写正规式；根据正规式写含义）</font>

- 是一种特殊表达式，用于表示不太复杂的语言的”含义“。通常它按照一组定义规则，由较简单的正规式构成。

- 正规式不是串，也不是符号，它是一种**<font color='blue'>模式</font>**，匹配该模式的所有串组成了一个语言。这个语言也称为**<font color='blue'>正规集</font>**。

#### 有限自动机

- 有限自动机通常用**<font color='blue'>转换图</font>**表示，本质是识别器的模型化表示

- 分为确定的有限自动机（DFA）和不确定的有限自动机（NFA）。两种自动机都恰好能识别正规集。其中不确定的含义是：存在这样的状态，对于某个输入符号，它存在不止一种转换

##### 不确定的有限自动机（NFA）

是一个数学模型，包括

- 有限的状态集合 S
- 输入符号集合 $\Sigma$
- 转换函数$move:S \times (\Sigma \cup \{\epsilon\}) \rarr P(s)$    (S的幂集)
- 状态 $s_0$ 是唯一开始状态
- $F\subseteq S$是接受状态集合

##### 确定的有限自动机（DFA）

包括内容与NFA相同，区别在于

- 任何状态都没有 $\epsilon$ 转换
- 对于任何状态 s  和任何输入符号 $a$，DFA最多有一条标记为 $a$ 的边离开 s，而NFA可以有多条

![image-20260627185941659](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627185941659.png)

## 词法分析器功能：字符流 $\rarr$ 记号流

> 我的直观就是，这里好比tokenizer，我们将句子划分为单词，以便后续处理
>



## 词法分析器的实现：正规式 $\rarr$  NFA $\rarr$  DFA $\rarr$  最简DFA $\rarr$  语言识别器

### 从正规式到有限自动机

#### Thompson算法

```
input：字母表 Σ 上的正规式 r
输出：接受语言L(r)的NFA N(r)
```

##### 识别”或”正规式的NFA

```
s | t
```

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627190522436.png" alt="image-20260627190522436" style="zoom:50%;" />

##### 识别“连接”正规式的NFA

```
st
```

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627190703105.png" alt="image-20260627190703105" style="zoom:50%;" />

##### 识别“闭包”的正规式NFA

```
s*
```

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627190729896.png" alt="image-20260627190729896" style="zoom:50%;" />



##### 算法性质

- **<font color='blue'>状态数</font>**最多是 r 中符号和算符总数的**<font color='blue'>两倍</font>**

- 只有一个开始状态和一个接受状态，接受状态没有向外的转换
- 每个状态有**<font color='blue'>一个</font>**用 **<font color='blue'>Σ</font>** 的符号标记指向其他节点的**<font color='blue'>转换</font>**，或者最多**<font color='blue'>两个</font>**指向**其他**节点的 **<font color='blue'>ε 转换</font>**

### 从NFA到DFA的变换

NFA对同一输入字符可以进入多个状态，且存在 ε 转换，会引起二义性，难以用计算机模拟。所以需要转换为DFA



#### 子集构造法

##### $\textbf{ ε-closure(S) } $

即ε闭包，指从状态子集S中的任一状态出发，仅通过空转换所能到达的所有NFA状态的集合

$$\epsilon\text{-closure}(S) = S \cup \{ q \mid \exists p \in \epsilon\text{-closure}(S), \, p \xrightarrow{\epsilon} q \}$$

> 也就是说将Dstates中所有 ε 边达到的状态加进去

##### $\textbf{move(S,a)} $

从状态子集 S 中的任一状态出发，通过一条标号为 $a$ 的转换边所能到达的NFA状态的集合

$$\text{move}(S, a) = \{ q \mid \exists p \in S, \, p \xrightarrow{a} q \}$$

> 也就是说看看当前Dstates集合里面经过a这条边能达到哪些状态。

##### 算法

```
input：一个NFA
output:一个接受同样语言的DFA

初始：ε-closure(s0)是Dstates仅有的状态，并且尚未标记：
while(Dstates有尚未标记的状态T){
	标记T;
	for (每个输入符号 a){
		U = ε-closure(move(T,a));
		if (U不在Dstates中){
			把 U 作为尚未标记的状态加入Dstates；
		}
		Dtrans[T,a] = U
	}
}
```

##### 例子

```
(a|b)*ab
```

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627194154411.png" alt="image-20260627194154411" style="zoom:67%;" />

1. 先计算A = ε-closure(s0) = {0，1，2，4，7}
   <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627194435410.png" alt="image-20260627194435410" style="zoom: 67%;" />
2. 计算 ε-closure(move(A,a)) = ε-closure({3，8}) = {1，2，3，4，6，7，8} 
   <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627194705578.png" alt="image-20260627194705578" style="zoom:67%;" />
3. 将 B 作为尚未加入的状态加入表中
4. 计算 ε-closure(move(A,b)) = ε-closure(move({5})) ={1，2，4，5，6，7} 
5. 将 C 作为尚未加入的状态加入表中
   <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627195443787.png" alt="image-20260627195443787" style="zoom: 67%;" />
6. .......

计算ε-closure(move(B,a))、ε-closure(move(B,b))、ε-closure(move(C,a))、ε-closure(move(C,b))直到填满表格（无新状态）.......
<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627195656934.png" alt="image-20260627195656934" style="zoom: 67%;" />

由于D包含了原来的状态9，所以D是接受状态

> 这里需要注意一点，如果得到一个新集合内容为空，那么需要新增一个死状态。例子如下
>
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627213032867.png" alt="image-20260627213032867" style="zoom:67%;" />
>
> ![image-20260627213056214](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627213056214.png)

### DFA的化简

上述可知，自己构造法不一定得到最简的DFA。

- 理论上，每个正规集都可以由一个**<font color='blue'>状态数最少</font>**的DFA识别。若不记同构，则这个DFA是**<font color='blue'>唯一</font>**的。
- 如果两个自动机的状态只有名字不同，其他都相同，则我们说这两个自动机同构

#### 死状态

指对所有输入符号都转换到该状态本身。如下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627213413916.png" alt="image-20260627213413916" style="zoom:50%;" />

#### 可区别状态



从s出发，输入w，最终停留在某个接受状态；而从t出发，输入w，最终停留在非接受状态，或者反过来。

#### 不可区别的状态

<span id="DFA-example"></span>

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627213542045.png" alt="image-20260627213542045" style="zoom:50%;" />

也就是说找不到任何串来区别两个状态，也就是等价的状态。

如上述DFA中，A和B是可区别的状态；A和C是不可区别的状态：A和C通过a和b到达的状态完全相同。

#### 极小化DFA状态数

```
输入：DFA M
输出 DFA M'.它与M接受同样语言，且状态数最小

1. 初始划分 Π = {F，S-F}

2. 用下面过程对 Π 构造新的划分 Π_new
for(Π中的每个子集G){
	把G划分为若干子集：若要使G的两个状态s和t在同一子集中，当且仅当对任意输入符号a，s和t的a转换是到 Π 的同一子集中。
	在 Π_new中，用G的划分代替G
}

3. 如果 Π_new = Π，则令Π_final = Π，执行4

4. 在Π_final的每个状态子集中选一个状态代表它，这些状态就是最简DFA M‘的状态。相应要修改状态转换表。包含s0的状态子集的代表是M’的开始状态，原先属于F集合的代表是M’的接受状态

5. 去掉M‘中的死状态和不可及的状态
```

> 核心是划分，能划分的依据是不一致（存在某个move结果不一致，注意目标状态也是一个集合而非单个状态）。
>
> 结束即不可进一步划分，那么也就是说结束的条件就是一致（存在可合并的状态时，该状态集合中的所有状态都是不可区分的）

##### 例子1

如[上图](#DFA-example)



<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627221722987.png" alt="image-20260627221722987" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627221742313.png" alt="image-20260627221742313" style="zoom: 50%;" />

##### 例子2

其中s4，s5，s6是接受状态

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260627223022712.png" alt="image-20260627223022712" style="zoom: 67%;" />

## 重点例题

### 非形式描述的语言 <-> 正规式

> 做这种题的诀窍就是多写几个试试看。多多少少基于一点分块的思想。然后记得有空串的写上包括空串。

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628134849188.png" alt="image-20260628134849188" style="zoom: 50%;" />

```
由0，1组成的，首尾为0的，长度至少为2的串
```

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628134909498.png" alt="image-20260628134909498" style="zoom: 50%;" />

```
字母表{0，1}上所有串的集合（包括空串）
```

> 一种比较有说服力的做法是：0，1组成的所有串为 (0|1)*，而(ε | 0)就等价于0

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628134946730.png" alt="image-20260628134946730" style="zoom: 50%;" />

```
由0，1组成的，倒数第三个字符为0的，长度至少为3的串
```

> 这里的方法硬要说的话就是分块吧。闭包分块，固定的0分块，后面的二选一分块。标准答案说是....以000，001，010，011结尾的任意0，1串，长度大于等于3。我感觉我这样说也没问题

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628135002117.png" alt="image-20260628135002117" style="zoom: 50%;" />

```
含有且只含3个1的，长度至少为3的，由0，1组成的任意串
```

> 注意这里的含有**且只含**吧，有些时候只写了”含有“，语义不充分

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628135014733.png" alt="image-20260628135014733" style="zoom: 50%;" />

```
0,1组成且0,1个数都为偶数（包括0）的所有串，包含空串。
```

> 这个我是真很难直观看出来。 还是基于我们分块的思想，为了让这个更直观，我们将其化简为A*(BA\*BA\*)\*，这里就比较明显了，也就是说这个B可以出现偶数次（包含0），A可以出现任意次。而A是任意成对的00/11，B是01/10，1和0个数是奇数，两个B那么0/1出现的次数也是偶数了。

### 正规式 -> NFA（DFA、最简DFA）

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628142641984.png" alt="image-20260628142641984" style="zoom:50%;" />

<span id="hw-6"></span>

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628142903594.png" alt="image-20260628142903594" style="zoom:50%;" />

> 可能需要注意的是状态标号从0开始吧

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628142942314.png" alt="image-20260628142942314" style="zoom:50%;" />

略

### NFA -> DFA

##### 将上述的[NFA](#hw-6)变DFA

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628144827928.png" alt="image-20260628144827928" style="zoom:50%;" />

### DFA -> 最简DFA

### 非形式描述的语言/正规式 <-> NFA（DFA、最简DFA），手工构造法

### 简单的分析或证明



---



# 第三章 语法分析

![image-20260628145837549](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628145837549.png)

## 记号流 $\rarr$ 分析树

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628151147633.png" alt="image-20260628151147633" style="zoom:50%;" />

词法分析器中描述词法规则时使用正规式，语法分析时，很多语法规则使用正规式无法描述

> 为什么？我的直觉是，我们进行词法分析是一维的，产生的输出也是一维的token流。而代码不仅仅具有顺序结构，还有条件、循环等等这类层级的结构，一维结构已经无法完整描述代码。所以我们需要一个新的规则能让代码保证层级结构，同时能结束不会死循环。于是树就是一个很好的数据结构，所以我们需要某种规则将token流扩展到二维的树结构。
>
> 谈到构建树或者是遍历树，自然就会联想到递归，递归需要回溯会花费时间。所以我们往往可以通过栈这个数据结构来达成非递归。

#### 语法结构

包括函数、表达式、语句等。可以用分析书来表示

### 形式语言的Chomsky分类

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629164437570.png" alt="image-20260629164437570" style="zoom:67%;" />

## 上下文无关文法

### 定义

上下文无关文法（CFG，不是控制流图哈哈）G是一个四元组$(V_T,V_N,S,P)$

- $V_T$：指终结符集合（非空、有限，在谈论编程语言时，终结符即词法记号名）。终结符一般包括：定义的小写字母、粗体字符串、数字、标点符号和运算符号
- $V_N$：非终结符集合（非空集合）。终结符和非终结符交集为空。非终结符一般包括定义的大写字母，S，小写字母组成的名字
- S：开始符号，是一个非终结符，其定义的终结符串集就是文法定义的语言
- P：产生式有限集合 

> 上下文无关的直观理解是，任何情况，只要文内存在左部非终结符，就可以被替换成右边；同样只要找到符合产生式右边的串就可以归约成该非终结符。完全不依赖于句型中的其他符号。

#### 例子

很经典的一个例子，常常用来作为例题。考虑优先级、结合性、二义性等等

![image-20260628152834587](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628152834587.png)

### 推导

把产生式看成**<font color='blue'>重写规则</font>**，把符号串中的非终结符用其产生式右部得串来代替

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628153113749.png" alt="image-20260628153113749" style="zoom:67%;" />

对于上述例子，我们可以按照任意顺序对单个E不断应用各个产生式，得到一个替换序列，如：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628153150631.png" alt="image-20260628153150631" style="zoom:80%;" />

这样我们推导证明了串 `-(id+id)` 是表达式得一个实例

> 注意推导符号和产生符号分开来。推导一般有三种符号
>
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628153858925.png" alt="image-20260628153858925" style="zoom: 67%;" />

#### 递归

设给定文法$G:(V_T,V_N,S,P)$，若有 A $\implies$ α $\implies^*$ βAγ， β 和 γ 不同时为 ε ，则称非终结符 A 时**<font color='blue'>递归</font>**的

- 若存在产生式 A $\rarr$ βAγ，则称非终结符A是直接递归的
- 若 β = ε，则A是左递归
- 若 γ = ε，则A是右递归 

如果文法中至少含有一个递归的非终结符，则此文法称为递归文法。只有文法G递归定义时，L(G)中句子才是无穷的。

#### 最左推导 

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628155020418.png" alt="image-20260628155020418" style="zoom: 67%;" />

#### 最右推导

也叫规范推导

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628155027877.png" alt="image-20260628155027877" style="zoom:67%;" />

#### 消除左递归

##### 消除直接左递归

对于下面式子

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629160812558.png" alt="image-20260629160812558" style="zoom:67%;" />

直接套公式！！！

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629160728769.png" alt="image-20260629160728769" style="zoom:67%;" />

可拓展为：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629161645714.png" alt="image-20260629161645714" style="zoom:67%;" />

本质上是把左递归转换成右递归

###### 例题

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629160940086.png" alt="image-20260629160940086" style="zoom: 67%;" />

##### 间接左递归

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629161931687.png" alt="image-20260629161931687" style="zoom:50%;" />

也就是将非直接左递归转换为直接左递归后再消除直接左递归。

如果原文法G产生了<font color='red'>**回路**</font>，则需要对非终结符进行排序，用**<font color='blue'>排在后面的代换排在前面的非终结符。</font>**

#### 提取左公因子

对于
$$
A \rarr \alpha\beta_1 | \alpha\beta_2
$$
当不清楚应该用非终结符A的哪个候选式来替换它时，可以通过重写A产生式来推迟这种决定。等读入了足够多的输入，获得足够信息后再做正确的决定。

具体来说，我们也是引入新的非终结符来提取公因子。把不相同的部分记为因子
$$
A \rarr \alpha A'
\\
a' \rarr \beta_1 | \beta_2
$$


### 分析树

分析树是推导的图形表示。其**<font color='green'>根节点是开始符号</font>**；**<font color='green'>内部节点是非终结符</font>**，由该非终结符的这次推导所用产生式的右部各符号**<font color='green'>从左到右依次标记</font>**；<font color='green'>**叶子节点**</font>是非终结符或终结符，从左到右**<font color='green'>构成一个句型</font>**

以下是一个最左推导的分析树

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628155500482.png" alt="image-20260628155500482" style="zoom: 50%;" />

可见**<font color='blue'>推导的过程就是一个由上而下简历分析树的过程</font>**

#### 练习

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628155624144.png" alt="image-20260628155624144" style="zoom: 50%;" />

### 二义性

考虑如下例子

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628155705258.png" alt="image-20260628155705258" style="zoom:67%;" />

文法二义指的是：该文法存某个句子，有不止一个最左（最右）推导。

#### 消除文法二义性

但是大部分语法分析器都期望文法是无二义性的，否则我们就不能为一个句子唯一选定语法分析书。

消除二义性的关键是定义**<font color='red'>优先级</font>**和**<font color='red'>结合性</font>**，方法是**<font color='blue'>引入非终结符</font>**，限制每一步推导只有唯一的选择

##### 例一

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628220601997.png" alt="image-20260628220601997" style="zoom:67%;" />

> 具体来说，产生式**<font color='blue'>优先级从上到下</font>**依次为**<font color='blue'>从低到高</font>**，**<font color='blue'>左结合为左递归</font>**，**<font color='blue'>右结合为右递归</font>**。相同优先级的写在同一个产生式即可。

##### 例二

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628220933790.png" alt="image-20260628220933790" style="zoom:67%;" />

##### 例三-消除悬空ELSE

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260628221245278.png" alt="image-20260628221245278" style="zoom:67%;" />

规定else和左边最接近的还没有配对的then相配对

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629160049325.png" alt="image-20260629160049325" style="zoom: 67%;" />

## 自上而下分析

从文法开始符号出发，自上而下，从左到右为输入串建立分析树。总是选择每个句型的最左非终结符进行试探和替换。

这种方法存在的问题有：

- <font color='blue'>**不能处理左递归**</font>
- 需要更多临时变量来保存状态
- ..

预测分析是自上而下分析技术的一个特例，通过在输入中向前看固定个数（通常是一个符号）来选择正确的产生式。预测分析**<font color='blue'>不需要回溯</font>**，是一种确定的自顶向下分析方法。而LL(1)是符合预测出分析的一种方法

### LL(1)文法

关键点在于不存在回溯。没有回溯第一个前提就是没有左递归，第二是每次都能选择确定正确的产生式（也就是说没有左公因子和二义性）。对此我们保证选择正确的具体条件是：对于产生式 `A -> α | β`

- 任意终结符a不能同时出现在FIRST(α)和FIRST(β)中。
- 如果 β 能推导出空，那么a不能同时出现在FIRST(α) 和 FOLLOW(β)中

#### 定义

##### LL(1)的含义

- 第一个L：从左到右分析
- 第二个L：最左推导
- （1）：每步向前搜索一个输入符号

##### LL(1)文法的充要条件

任何两个产生式 `A-> α|β` 都满足下列条件

- $FIRST(\alpha) \cap FIRST(\beta) = \empty$ 
- 若 $\beta \implies *\epsilon$，那么$FIRST(\alpha) \cap FOLLOW(A) = \empty$

##### LL(1)的必要条件

- 没有公共左因子
- 不是二义的
- 不含左递归

#### 求FIRST和FOLLOW集

##### 求FIRST

```
1. 如果当前读取的符号是个终结符，那么直接加入FIRST集。（终结符的FIRST集只有自己）

2. 如果当前读取的符号是个非终结符
	a. 把该非终结符FIRST集里除了ε以外的所有元素加入目标FIRST集
	b. 如果该非终结符的FIRST集包含ε，则读取下一个符号，把它当作新的起点，回到步骤 1
	c. 如果达到产生式的末尾，发现该产生式上的所有符号都能推导出ε，则将ε加入目标的FIRST集（当然如果能直接推导出空，也将ε加入FIRST集）

3. 对文法的所有产生式反复执行1和2，直到FIRST集合彻底稳定，不再新增任何元素。
```

##### 求FOLLOW

```
1. 将 $ 加入开始符号的FOLLOW集中

2. 若有产生式 B -> αAβ，则 FIRST(β) 中 除ε以外 的一切符号都属于FOLLOW(A)

3. 若有产生式 B -> αA，或产生式 B -> αAβ 而 FIRST(β) 中有 ε ，则FOLLOW(B) 中一切符号都要放入 FOLLOW(A) 中
```

> 所以我们可以说FOLLOW集不会出现 ε

### 递归下降预测分析

略

### 非递归预测分析（LL（1）分析）

具体结构如下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629195653268.png" alt="image-20260629195653268" style="zoom:50%;" />

主要分为四个结构：输入缓冲区、栈、分析表、输出流

- 输入缓冲区：存在需要解析的句子。末尾必须加上一个特殊的结束符 $
- 符号栈：初始化时，需要先垫一个 $ ，然后再加入文法的起始符号、
- 控制器和预测表：根据栈顶符号和当前输入符号，去查表决定下一步动作

#### case study

对该文法求得FIRST和FOLLOW集后

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629202258623.png" alt="image-20260629202258623" style="zoom:67%;" />

构造预测分析表如下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629200325924.png" alt="image-20260629200325924" style="zoom: 80%;" />

> - 对于FIRST(α)中的每一个非空的终结符a，将产生式A -> α 无条件填入表中
>
> - 如果FIRST(α)中有 ε ，那么对于该非终结符中的每一个终结符 b，b在FOLLOW(α)中 ，将产生式A -> α 无条件填入表中

对于输入`id * id + id`预测分析器如下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629203316073.png" alt="image-20260629203316073" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629205152765.png" alt="image-20260629205152765" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629205225481.png" alt="image-20260629205225481" style="zoom:67%;" />



> ##### 查表展开
>
> 如果栈顶是个非终结符，当前输入是id，那么查表M[E,a]后
>
> - 将栈顶的E弹**<font color='blue'>出栈</font>**
> - 将产生式右部 **<font color='red'>逆序</font> ** **<font color='blue'>压入栈</font>** 中
> - 在**<font color='blue'>输出</font>**中记录这条产生式
>
> ##### 匹配消除
>
> 如果栈顶是个终结符，也就是说没有任何产生式可以展开了。此时我们只检查它和当前输入符号是否长得一模一样，如果一样，直接把**<font color='blue'>栈顶</font>**符号**<font color='blue'>弹出</font>**，同时把**<font color='blue'>输入</font>**指针向**<font color='blue'>后移动一位</font>**。注意此时的**<font color='blue'>输出为空</font>**。
>
> 
>
> 最后如果栈顶和缓冲区都只剩下 $ 的话，说明匹配完美结束了

## 自下而上分析

### 归约和句柄的概念

最左（最右）推导的逆过程就是最右（左）归约。其中最左归约也称为规范归约。

值得注意的是：二义文法不是LR(k)的，但是移**<font color='blue'>进-归约分析可以用来分析某些二义文法</font>**，只要给出解决冲突的策略

#### 句柄

句柄是归约的一个子串。如果一个句型由最右推导得到，则句型的句柄是和某产生式右部匹配的子串，并且，把它归约成该产生式左部的非终结符代表了最右推导过程的逆过程的一步

> 推导的时候用的哪一块，反过来归约的时候这一块就是句柄
>
> 所以如果文法二义，那么句柄**<font color='blue'>可能</font>**不唯一，例下
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630091202509.png" alt="image-20260630091202509" style="zoom:50%;" />
>
> 这里的`rm`是最右推导的意思（rightmost），`lf`就是最左推导（leftmost）
>
> 注意句柄是推导得到的结果，是一整个代换式哦

具有以下性质

- **<font color='blue'>如果文法无二义时，每个右句型都有唯一的句柄</font>**
- 对于**<font color='blue'>最左归约</font>**，任意一个规范句型中，**<font color='blue'>句柄的右边绝对不可能出现非终结符</font>**；反之对于最右归约，左边不存在终结符
- 对于**<font color='blue'>最左归约</font>**，任意一个规范句型中，**<font color='blue'>句柄的左边既可能有终结符也可能有非终结符</font>**；反之亦然

### 移进归约分析及冲突

归约主要包括两个问题：

- 如何确定右句型中将要归约的子串
- 如果该子串是多个产生式的右部，确定选择哪一个产生式

#### 用栈实现移进-归约分析

和非递归预测类似，我们使用**<font color='red'>栈</font>**保存文法符号，用**<font color='red'>输入缓冲区</font>**保存要分析的串。初始化如下：

```
栈：$
输入：w$
```

- 移进：把下一个符号压入栈
- 归约：将栈顶的句柄归约为非终结符
- 接受：宣告分析成功（栈是$S，输入是\$）
- 报错：发现错误，调用错误恢复例程

##### case study

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260629202258623.png" alt="image-20260629202258623" style="zoom:67%;" />

还是输入 `id * id + id`

我们采用LR分析法来进行归约分析。

> 一般做题来说，我们先写最右推导，然后画出句柄。其逆过程就是最左归约了。
>
> 这样做往往更简单快捷一些，不然分析的时候不太能明确知道这里该移进还是归约。一旦了解了这个归约过程，那么移进-归约分析也很简单了。

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630094320534.png" alt="image-20260630094320534" style="zoom:67%;" />

> 重点在于第六行<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630094402103.png" alt="image-20260630094402103" style="zoom:50%;" /> 是应该移进还是归约呢？如果不看推导的话，无论这里移进还是归约最后都可以化简为接受状态，这里就是典型的移进归约冲突。原因是因为语法具有二义性，同时分析器只具有局部视野。

#### 移进-归约冲突

正如上述所示，我们没法判断当前是否为句柄。其核心在于：算符的优先级和结合性未确定；分析器视野局限。简单来说就是结束和继续之间无法选择

#### 归约-归约冲突

核心在于：文法设计存在重叠；分析器视野局限。简单来说就是两个不同归约之间无法选择。

### LR分析算法

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630101540031.png" alt="image-20260630101540031" style="zoom: 33%;" />

#### 分析表

对于文法：

```
E -> E+T
E -> T
T -> T*F
T -> F
F -> (E)
F -> id
```



<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630105636032.png" alt="image-20260630105636032" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630105700884.png" alt="image-20260630105700884" style="zoom:66.9%;" />



> LR分析法分为SLR、LR(1)、LALR。三种LR的分析算法都一样，只是构造LR分析表的方法不同，分析能力也不同





<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630110208030.png" alt="image-20260630110208030" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630110225255.png" alt="image-20260630110225255" style="zoom:67%;" />

> ##### 1. 初始化
>
> 栈中填入0，输入填入表达式，以$结束
>
> ##### 2. 进行移进-归约分析
>
> 1. 将栈顶数字作为行，输入最左端元素为列，查询表中元素
>
>    a. 如果是`sn`，则将该输入元素压入栈中，并且加上数字n
>
>    b. 如果是`rn`，则按照对应的第n个产生式，将对应个数的符号和数字弹出，归约后将符号压入栈中，并计算新的数字。
>
> 2. 查到acc，结束；查到空白，error报错

**<font color='blue'>栈中的文法符号总是形成一个活前缀</font>**。所谓活前缀，就是右句型的前缀，该前缀不超过最右句柄的右端。也就是说，栈中文法符合和剩余输入组成一个右句型，其中栈中文法符号是该右句型的前缀。由于栈顶一旦发现该句柄就会被归约，所以栈中的文法符号肯定不会超过该句柄的右端

所以栈顶的状态符号包含了确定句柄所需要的一切信息，分析表的转移函数本质上是识别活前缀的DFA的转换函数

#### LR对比LL

|                      |                      LR(1)方法                       |                          LL(1)方法                          |
| :------------------: | :--------------------------------------------------: | :---------------------------------------------------------: |
| **对文法的显式限制** |          <font color='red'>无二义性</font>           | <font color='red'>    无二义性、无左递归、无公共左因</font> |
|    **分析表比较**    |              状态 x 文法符号，分析表大               |                 非终结符 x 终结符，分析表小                 |
|    **分析栈比较**    |        状态栈，通常状态比文法符号包含更多信息        |                         文法符号栈                          |
|     **确定句柄**     | 根据栈顶状态和下一个符号便可确定句柄和归约所用产生式 |                              \                              |
|     **语法错误**     |           绝不会将出错点后的符号移入分析栈           |                          和LR一样                           |

### 构造SLR(1)分析表

#### 项目

全称为 文法的LR(0)项目。具体来说是

- 我们在右部的某个地方加点的产生式。
- 加点的目的是用来表示分析过程中的状态。

例如：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630133622899.png" alt="image-20260630133622899" style="zoom:50%;" /> 
注意只能在右部加点嗷

#### 拓广文法

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630133749111.png" alt="image-20260630133749111" style="zoom:67%;" />

简单来说就是在开始符号之前加入一个新的文法符号和产生式。拓广文法的目的是**<font color='blue'>确定何时分析成功</font>**。也就是说我们使用拓广的这个第0条产生式归约时代表分析成功



#### 构造LR(0)项目集规范族

```
1. 先拓广文法
2. 从项目 S' -> ·S 出发，计算闭包，得到初始状态I0
3. 对当前状态，找出所有圆点后面的不同文法符号
4. 对每个符号执行GOTO：
	a. 把该符号移到圆点左边
	b. 这些移动原点后直接得到的项目是核心项目
	c. 计算闭包，不如非核心项目
5. 检查已有项目集
	a. 如果和已有状态完全相同，就连接到已有状态
	b. 如果不同，就加入为新状态
6. 对所有新状态重复上述过程，知道所有状态的出路都处理完，且不再产生新状态
```

关于闭包的计算，只有当 · 的后方紧挨着一个非终结符的时候，才计算这个非终结符的闭包。

> 有关官方版本的构造LR(0)项目集规范族算法如下
>
> ```
> (1) 初始时，I的每个项目都加入closure(I)。
> (2)如果A-> α·Bβ在closure(I)中，且 B->γ是产生式，那么如果项目B->·γ还不在closure(I)中，则把它加入。重复该规则，直到没有更多项目可加入为止。
> ```

#### 算法

构造ACTION-GOTO表

```
1. 构造LR(0)项目集规范族
2. 计算所有非终结符的FOLLOW集
3. 根据终结符转换填写移进动作
	如果状态 Ii 经过终结符 a 转移到状态 Ij。则在ACTION[i,a]中填入sj
4. 根据非终结符转换填写GOTO表
	如果状态 Ii 经过非终结符 A 转移到状态 Ij。则在GOTO[i,A]中填入j
5. 根据归约项目填写归约动作
	如果状态 Ii 中存在归约的项目 A -> α·，对FOLLOW(A)中的每个终结符 a，在ACTION[i,a]中填写 rn
6. 填写接受动作
	如果状态 Ii 中存在项目 S' -> S·，则在ACTION[i,$]中填写acc
```

能够构造出无冲突的SLT分析表的文法就是SLR文法。SLR文法都不是二义的，但是描述能力有限，无法处理 移进-归约冲突 和 归约-归约冲突。具体例子如下，可自行尝试构建

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630141137443.png" alt="image-20260630141137443" style="zoom:50%;" />

> 上述构造出得DFA应该如下
>
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/81c4125c3b5e81e03025004e4d9b2f25.jpg" alt="81c4125c3b5e81e03025004e4d9b2f25" style="zoom: 25%;" />
>
> 这里产生冲突得地点在I2，冲突得原因是一个活前缀可能有多个有效项目。具体来说是，由于FOLLOW(E)={=,$}，所以当I2遇见=得时候存在移进归约冲突。
>
> 根本原因是归约时缺乏对上下文的考虑。归约时只考虑了下一个输入符号是否属于归约项目相关联的FOLLOW(A)集合，而没考虑串α所在的右句型的上下文。对于这个例子来说，只要 = 左边的 E出现，前面一定会有 * ，此时一定会被归约成 V。因此下一个符号是 = 的情况下，分析表应该已经吧 *E 归约成 V 了，应该只执行移进操作

### 构造LR(1)分析表

对于上述情况，直观的解决方式是在不同的使用位置，归约时应按情况考虑FOLLOW的子集。也就是说A会要求不同的后记符号。核心解决方法时加上**<font color='red'>搜索符</font>**

#### 搜索符

对于
$$
[A \rarr \alpha \cdot \beta,a]
$$
搜索符是在子串`αβ`所在的右句型中直接跟在`β`后面的终结符。但 β 不为空的情况下没什么用；但当 `β` 为 `ε` 时，它决定了何时将 `αβ` 归约为 `A`

#### 计算搜索符

对于
$$
[A \rarr \alpha \cdot B \beta,~a]
$$
这个核心项目，有$[B \rarr \gamma] \in P$ 这个非核心项目，那么此时这个B的搜索符为：
$$
[B \rarr \cdot \gamma, ~b]
\\
b \in FIRST(\beta a)
$$
注意此时**<font color='red'>b不能为空，因为a不为空</font>**

#### 算法

和SLR的过程基本相同，只不过需要每个项目后面加上一个搜索符。如果项目完全相同但是搜索符不同，那么就是两个状态。所以LR(1)的状态数一定 >= SLR(1)

```
1. 构造LR(1)项目集规范族：修改closure(I)和goto(I,X)函数，使其带上搜索符
2. 根据终结符转换填写移进动作
	如果状态 Ii 经过终结符 a 转移到状态 Ij。则在ACTION[i,a]中填入sj
3. 根据非终结符转换填写GOTO表
	如果状态 Ii 经过非终结符 A 转移到状态 Ij。则在GOTO[i,A]中填入j
4. 根据归约项目填写归约动作
	如果状态 Ii 中存在归约的项目 A -> α，则对搜索符中的每个终结符 a，在ACTION[i,a]中填写 rn
5. 填写接受动作
	如果状态 Ii 中存在项目 S' -> S·，则在ACTION[i,$]中填写acc
```

注意和SLR算法只有1和4有差距哦。当然绘制出的LR(1)项目集也是有差距的

### 构造LALR分析表

LALR通过搜索符能描述SLR无法描述的文法，但代价是增加了很多状态数。其中增加的状态数项目集和某个状态是相同的，只是搜索符不同。LALR旨在使用和SLR相同多的状态，使其分析能力介于SLR和LR之间。

考虑文法：

```
S' -> S
S -> BB
B -> bB
B -> a
```

我们构造LR(1)：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630173734090.png" alt="image-20260630173734090" style="zoom: 50%;" />

其中同色的就是同心的LR(1)项目集，我们称其为**<font color='red'>同心集</font>**。合并这些状态，我们就可以得到构造LALR分析表的DFA

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630173911473.png" alt="image-20260630173911473" style="zoom: 50%;" />

值得注意的是

- <font color='red'>同心集的合并**不会**产生新的 **移进-归约冲突**</font>
- <font color='red'>同心集的合并**可能会**产生新的 **归约-归约冲突**</font>

### 二义文法在LR分析中的应用

如果我们直接使用二义文法，如下
$$
E \rarr E+E  ~|~E*E~|~(E)~|~id
$$
构造SLR分析表就会有冲突

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/fa48d209bb6e68e12d91c5b51f8b21ea.jpg" alt="fa48d209bb6e68e12d91c5b51f8b21ea" style="zoom: 33%;" />



<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630214133708.png" alt="image-20260630214133708" style="zoom:67%;" />

此时我们规定 * 的优先级高于 +，且两者都是左结合

> 我们可以粗略理解成，遇到E*E优先归约；遇到E+E的话先缓缓，万一后面有\*
>
> 而由于是左结合，所以前面出现E+E，又来一个+的话就先归约；前面出现E*E，后面又来一个\*的话就先归约。
>
> 总之我们可以通过语义来直观理解，或者通过写几个句子来判断先归约还是移进。

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630214510622.png" alt="image-20260630214510622" style="zoom:50%;" />

> 其他例子：TODO

### LR分析中的错误恢复

- 紧急方式错误恢复：抛弃若干个输入符号，直到合法位置

- 短语级错误恢复：针对分析表中每个空白条目确定不同的最合适的恢复方法

### 例题

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260630214624866.png" alt="image-20260630214624866" style="zoom:67%;" />

> or遇到and和or发生移进归约冲突，因为or优先级比and低，所以遇到and时移进，又因为or左结合，所以遇到or归约。



---



# 第四章 语法制导的翻译

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/%E7%AC%AC4%E7%AB%A0%E8%AF%AD%E6%B3%95%E5%88%B6%E5%AF%BC%E7%9A%84%E7%BF%BB%E8%AF%91%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE-1.png" alt="第4章语法制导的翻译思维导图-1" style="zoom: 40%;" />

## 语法制导定义（SDD）

语法分析、语义分析、中间代码生成通常通过**语法制导的翻译**有机得结合起来。

语法分析过程并不关心文法符号得含义。编译器得最终目的是将源程序翻译成汇编语言或机器语言形式得目标代码，这个翻译过程会涉及到语义的分析，例如需要产生代码，把信息存入符号表，显示出错信息等

值得区分的概念是**<font color='red'>语法制导定义</font>**（SDD）和**<font color='red'>语法制导翻译方案</font>**（SDT）。前者是抽象的，后者是具体的。其中SDD包括有**<font color='blue'>基础文法（CFG）、文法符号的属性、产生式的语义规则</font>**

### 语法制导定义的形式

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701100922012.png" alt="image-20260701100922012" style="zoom:80%;" />

语法制导定义可以看作上下文无关文法的扩展：每个文法符号多了一组属性，每个产生式多了一组语义规则。属性一般表示一种语义信息，包括串、数值、类型等等。每个文法符号可以有多个属性。语义规则一般描述属性的计算规则。

- 每个文法符号关联一组属性
- 每个文法产生式关联一组语义规则来计算该产生式中各文法符号的属性值

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701102028129.png" alt="image-20260701102028129" style="zoom:67%;" />

通常我们将这类无意义的返回值称为**<font color='blue'>副作用</font>**，相当于计算一个虚拟属性的值

对于上述终结符的属性，如 `digit.lexval`，我们称其为<font color='blue'>**记号值** </font>

## 继承属性和综合属性

### 综合属性

文法符号E的**<font color='red'>综合属性值</font>**是通过分析树中它的**<font color='blue'>子节点</font>**或它**<font color='blue'>本身的属性值</font>**来计算

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701105051115.png" alt="image-20260701105051115" style="zoom:67%;" />

同时**<font color='green'>终结符的属性是综合属性</font>**。终结符的综合属性值是由词法分析器提供的词法值，SDD中没有计算终结符属性值的语义规则。

### 继承属性

文法符号E的**<font color='red'>继承属性值</font>**是通过分析树中的**<font color='blue'>兄弟结点、父节点或它本身的属性值</font>**来计算

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701105646828.png" alt="image-20260701105646828" style="zoom: 33%;" />

也就是说这些属性依赖于它们所在的上下文。

例下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701105847848.png" alt="image-20260701105847848" style="zoom:67%;" />

这里可以看出，id、L1的类型依赖于L，L的类型依赖于T，所以我们需要优先计算T的属性，再计算L，最后是id的属性

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701110535042.png" alt="image-20260701110535042" style="zoom: 50%;" />

这种属性的依赖关系可以用一种叫做**<font color='red'>依赖图</font>**的有向图来描绘。其中对于一个文法符号，**<font color='blue'>左边是继承属性，右边是综合属性</font>**。绘制的算法为：

1. 为纯副作用语义引入虚拟属性（一般都是综合的）
2. 分析树中如果属性b依赖于属性c，则从c的结点到b的结点有一条有向边

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701110952591.png" alt="image-20260701110952591" style="zoom: 33%;" />



对于属性计算的次序，我们引入**<font color='red'>拓扑排序</font>**。按拓扑排序的次序计算属性。这种方法称为分析树方法，适合手工构造和判断。实际上有更高效的方法

### S-SDD

仅仅使用综合属性的语法制导定义称为S属性定义，简称为S-SDD

S-SDD可以按照**<font color='blue'>自下而上的顺序</font>**计算属性，可以和LR分析结合起来

### L-SDD

简称为L属性定义。在一个产生式所关联的各属性之间，可以从左到右，但不能从右到左。当一个SDD是L-SDD，当且仅当

1. 每个属性要么是综合属性
2. 要么是满足下列条件的继承属性：产生式 A -> X1, X2 ... Xn， 其右部符号Xj (1<=j<=n)的继承属性仅依赖：
   - **A的继承属性**（注意不能用父节点的综合属性！！！）
   - 该产生式中Xj**左边符号** X1，X2，...Xj-1 的**属性**
   - Xj **本身的属性**，但**不能**在依赖图中**形成环**



每一个S-SDD都是L-SDD

## 语法制导的翻译方案（SDT）

这里将语义规则进一步写成语义动作。我们可以把语义动作想想成一个文法符号，进行推导（或归约）的时候，就是该语义动作执行的时候。对于上述L-SDD的语法制导定义，我们可以写出如下语法制导翻译方案

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701120128893.png" alt="image-20260701120128893" style="zoom:50%;" />

### 用SDT实现S属性定义

主要用于**<font color='red'>自下而上的分析</font>**，应用LR分析。

#### 方法

将每个语义动作都放在产生式的最后

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701200522662.png" alt="image-20260701200522662" style="zoom:50%;" />

结果应该输出一棵AST。至于AST是什么以及相关查阅 [参考](https://pocon041.github.io/2026/06/06/%E5%B8%B8%E8%A7%81%E4%B8%AD%E9%97%B4%E4%BB%A3%E7%A0%81%E4%BB%A5%E5%8F%8A%E5%A4%84%E7%90%86%E6%96%B9%E5%BC%8F/#more)。值得注意的是：非终结符都是分支结点，终结符都是叶子结点。我们使用mkNode和mkLeaf函数分别制造出中间结点和叶子结点，并且返回一个指针。

具体来说，如果我们构造一棵 `a+5*b` 的AST如下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701200621149.png" alt="image-20260701200621149" style="zoom:50%;" />

综合属性可以由自下而上的分析器**<font color='red'>在分析输入的同时完成计算</font>**。实际上，我们也仅仅是用LR分析器增加一个域来保存综合属性值，归约时执行动作计算综合属性即可。

### 用SDT实现L属性定义

主要用于自上而下的分析，应用LL分析

## S属性定义的自下而上翻译

注释分析树指结点的**<font color='blue'>属性值都标注出来</font>**的分析树。计算结点属性值的过程叫做**<font color='blue'>注释</font>**（加注）或**<font color='blue'>修饰</font>**

对于全部都是综合属性的SDD，分析树各节点属性的计算可以自下而上得完成

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701105402473.png" alt="image-20260701105402473" style="zoom: 40%;" />

## 例题

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701114950138.png" alt="image-20260701114950138" style="zoom:50%;" />

> a是继承，b和c是综合



<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701115007317.png" alt="image-20260701115007317" style="zoom: 50%;" />

> B.b依赖于父节点，所以b是继承属性，s是综合属性（吗？）。B.i依赖于父节点和兄弟结点，所以 i 是继承属性。又因为B依赖于C而C在B的右边，所以不是L-SDD也不是S-SDD。



---



# 第五章 类型检查

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/%E7%AC%AC5%E7%AB%A0%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE-1.png" alt="第5章类型检查思维导图-1" style="zoom:67%;" />

类型检查在语法分析与中间代码生成之间，和语义分析高度结合。通常我们会在设计类型系统后，在语法制导翻译方案中加入类型检查和类型转换代码。

## 相关概念

- **执行错误**：程序运行时出现的错误。其中<font color='blue'>会被捕获的错误</font>会引起计算立即停止；<font color='blue'>不会被捕捉的错误</font>可能会有一段实践未引起注意。
- **良行为的程序**：一个程序的运行<font color='green'>**不可能引起不会被捕获的错误**</font>，则称它为良行为的
- **安全语言**：任何合法程序都是良行为的语言
- **类型化的语言**：为<font color='blue'>每种运算</font>都定义了各个运算<font color='blue'>对象</font>和运算结果所允许类型的语言。包括显式类型化语言（Java，C++）和隐式类型化语言（类型声明可忽略，系统自动分配类型）。**<font color='red'>类型化语言不一定是类型可靠的，比如说C语言</font>**
- **无类型语言**：不限制变量的取值范围，运算可以作用于任何运算对象，结果可能位置。比如汇编语言。
- **类型化语言的类型系统**：语言的组成部分由一组<font color='blue'>定型规则</font>构成，这组规则用来给各种语言构造指派类型
- **类型检查**：即根据定型规则来确定程序中各语法构造的类型。目的是拒绝那些有<font color='blue'>类型错误</font>的程序
- **良类型的程序**：能够通过**<font color='green'>类型检查</font>**的程序是良类型程序
- **类型可靠的语言**：若所有良类型程序都是**<font color='green'>良行为</font>**的，则称该语言是类型可靠的

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701203752811.png" alt="image-20260701203752811" style="zoom: 80%;" />

- **静态可靠**：<font color='red'>包括类型错误、类型化语言、良类型程序</font>
- **动态可靠**：<font color='red'>包括执行错误、安全语言、良行为程序。</font>

## 类型表达式

一个程序的构造类型可以用类型表达式来表示。类型表达式包括基本类型和从基本类型构造的构造类型。

其中基本类型有： `interger`，`real`, `char`, `boolen`, `void`\` ,`type_error` 等等

用基本类型构造出的类型就是构造类型，其中包括：

##### 数组构造符：array(num,T)

其中T是某个类型表达式，num是interger类型

##### 指针构造符：pointer(T)

其中T是某个类型白哦大是

##### 笛卡尔积 X 和 函数构造符 $\rarr$ 

例如 `int X char -> bool` 意思是第一个参数类型是int，第二个参数类型是char，返回结果是bool。两样都为空即 `void -> void`

##### 记录构造符record

若有标识符 N1...Nn 与类型表达式 T1，T2...Tn，则`record(N1:T1, N2:T2,..., Nn:Tn)`是类型表达式。注意是<font color='green'>先变量名再类型名</font>



这几类构造类型也可以互相嵌套使用，比如array中的T可以是指针或者record。

## 类型系统的描述语言

类型系统是一种符号系统，来自于自然推演。包括断言、定型规则和演绎推理、类型推断。

### 断言

$$
\Gamma \vdash S
$$

- $\Gamma$ 表示类型环境，包含所有变量，变量所属的类型。相当于程序中的类型声明语句和编译器里的符号表
- $\vdash$ 表示可证明出，或者 由此得出
- 这个断言的意思是：S的自由变量都声明在 $\Gamma$ 中

#### 环境断言

$$
\Gamma \vdash \Diamond
$$

表示 $\Gamma$ 是良性的环境 

#### 语法断言

$$
\Gamma \vdash nat
$$

这里说的是自然数在环境中。

#### 定型断言

$$
\Gamma \vdash M:T
$$

代表在环境 $\Gamma$ 下，M具有类型T。其中M的自由变量都出现在 $\Gamma$ 中。

例如$\empty \vdash true:boolean$，$\{x:nat\} \vdash x+1:nat$代表x是自然数，那么x+1也是自然数

### 推理

推理规则是在一组已知有效断言的基础上，声称某个断言的有效性。一般形式如下
$$
\frac{\Gamma_1 \vdash S_1,...,\Gamma_n \vdash S_n}{\Gamma \vdash S}
$$
其中横线上方是**<font color='red'>前提</font>**，横线下方是**<font color='red'>结论</font>**。前提为零的规则叫做**<font color='red'>公理</font>**

#### 环境规则

$$
\frac{}{\empty \vdash \Diamond}
$$

空环境是良形的。这是一个公里

#### 语法规则

$$
\frac{\Gamma \vdash \Diamond}{\Gamma \vdash boolean}
$$

任何良形环境下，boolean是一个布尔表达式

#### 定型规则

$$
\frac{\Gamma \vdash M:int,\Gamma \vdash N:int}{\Gamma \vdash M+N:int}
$$

两个int类型相加的表达式仍然是int类型

### 类型表达式等价

#### 结构等价

- 两个类型表达式完全相同（当无类型名时）
- 把所有的类型名字用它们定义的类型表达式代换后，两个类型表达式完全相同（有类型名时）

#### 名字等价

- 把每个类型名看成一个可区别的类型
- 两个类型表达式名字等价当且仅当这两个类型表达式不做名字代换就等价



---



# 第六章 运行时存储空间的组织和管理

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/%E7%AC%AC6%E7%AB%A0%E8%BF%90%E8%A1%8C%E6%97%B6%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4%E7%9A%84%E7%BB%84%E7%BB%87%E5%92%8C%E7%AE%A1%E7%90%86%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE-1.png" alt="第6章运行时存储空间的组织和管理思维导图-1" style="zoom:67%;" />

本章属于后端（综合部分）

## 活动记录的布局，局部数据的布局方式

### 活动记录

![image-20260701214434191](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260701214434191.png)

过程的活动所需要的信息用一块连续的存储区来管理，叫做**<font color='red'>活动记录</font>**或**<font color='red'>帧</font>**。上右图是一般的活动记录的域。不同编译器的域布局可能不同。

- 临时数据：保存临时值，如表达式**<font color='red'>中间结果</font>**
- 局部数据：作用域在本过程中的数据。通常按顺序连续分配，以便使用偏移查找地址。注意存在栈对齐
- 机器状态：**<font color='red'>过程调用前</font>**的机器状态信息，如返回地址、previous ebp等等
- 访问链：过程嵌套语言的**<font color='red'>非局部数据</font>**的访问
- 控制链：指向调用者的**<font color='red'>活动记录</font>**
- 返回值：存放被调用过程返回给调用过程的值
- 参数：存放调用过程中提供的**<font color='red'>实参</font>**

## 活动树、控制栈和运行栈

#### 活动树

可以用活动树来描绘控制进入和离开活动的方式。例如下方这p个快排，我们一个主函数一共会调用2次函数，其中q函数又会调用p函数。

所以四个函数的具体调用顺序是：`m -> r, q;   q -> p`。具体来说r和q是m的左右孩子，p是q的左孩子。

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702085420887.png" alt="image-20260702085420887" style="zoom:67%;" />

#### 控制栈

当前活跃着的过程活动可以保存在一个栈中。这个栈就是**<font color='red'>控制栈</font>**例下：

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702085634255.png" alt="image-20260702085634255" style="zoom:67%;" />

此时控制栈的内容就是：`m，q(1,9)，q(1,3), q(2,3)`

#### 运行栈

运行栈是把控制栈中的信息拓广到包括过程活动的活动记录

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702085826783.png" alt="image-20260702085826783" style="zoom:50%;" />

我们由下向上来看，分别对应了我们之前看的那个活动记录栈

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702090042688.png" alt="image-20260702090042688" style="zoom:50%;" />

## 值调用、引用调用和换名调用

### 值调用

值调用指实参的右值传给被调用过程。也就是说**<font color='red'>对形参的任何操作不会影响调用者的实参的值</font>**

### 引用调用

引用调用把实参的左值放入形参的存储单元；若实参无左值，则将实参的值存入新的存储单元，并传递该单元的地址。**<font color='red'>对形参的任何赋值都会影响调用者的实参</font>**。

### 换名调用

考虑内联函数



> TODO：左值与右值



## 例题

<img src="C:\Users\czh\AppData\Roaming\Typora\typora-user-images\image-20260702090312068.png" alt="image-20260702090312068" style="zoom: 50%;" />



<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702093109200.png" alt="image-20260702093109200" style="zoom:50%;" />



---



# 第七章 中间代码生成

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703203044000.png" alt="image-20260703203044000" style="zoom:80%;" />

## 中间表示

### 抽象语法树和有向无环图（DAG）

这两者都是图形化的中间表示。我觉得画出的关键是对运算符的先后次序进行排序吧。

如果需要转换成中缀表达式，就中序遍历；转成后缀表达式就后序遍历。

![image-20260702125226910](C:\Users\czh\AppData\Roaming\Typora\typora-user-images\image-20260702125226910.png)

### 后缀表达式

中缀表达式转后缀表达式可以使用  [调度场算法](https://pocon041.github.io/2024/11/05/c-%E5%AE%9E%E7%8E%B0%E8%AE%A1%E7%AE%97%E5%99%A8/)，也可以先构造语法树然后再后续遍历语法树

## 三地址码

三地址码（TAC）的一般形式是
$$
x = y ~op~ z
$$
其中x、y、z表示名字、常数或临时变量。op表示运算符。

三地址码可以看作树语法树或DAG的一种线性表示。我们进行后续遍历，遇到运算符产生对应的三地址码

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702125730862.png" alt="image-20260702125730862" style="zoom:50%;" />

## 如何翻译成三地址码

### 声明语句

<img src="C:\Users\czh\AppData\Roaming\Typora\typora-user-images\image-20260702132716075.png" alt="image-20260702132716075" style="zoom:50%;" />

### 赋值语句

为了让名字直接出现在三地址码中，实际上应该把名字理解为**<font color='red'>在符号表中位置的指针</font>**。

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260702132845816.png" alt="image-20260702132845816" style="zoom:50%;" />

具体来说哦我们使用 `lookup` 函数查询指针，并用 `emit` 输出对应三地址代码字符串

### 布尔表达式

这里的核心是**<font color='red'>短路运算</font>**。 

> 我的一些理解是，如果不能短路那么需要生成一个新的出口。框内部的继承属性code输出当前true or false，依据短路运算来决定最终是否可以输出最终的结果还是进入下一个表达式。

#### B -> B1 or B2

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703212222748.png" alt="image-20260703212222748" style="zoom:33%;" />

```
B1.true = B.true;
B1.false = newLabel();
B2.true = B.true;
B2.false = B.false;
B.code = B1.code || gen(B1.false,':') || B2.code
```



#### B -> B1 and B2

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703212231531.png" alt="image-20260703212231531" style="zoom:33%;" />

```
B1.true = newLabel();
B1.false = B.false;
B2.true = B.true;
B2.false = B.false;
B.code = B1.code || gen(B1.true, ':') || B2.code
```

#### B -> not B1

```
B1.true = B.false;
B1.false = B.true;
B.code = B1.code
```

#### B -> (B1)

```
B1.true = B.true;
B1.false = B.false;
B.code = B1.code;
```

因为code是继承属性

#### B -> E1 relop E2

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703212250575.png" alt="image-20260703212250575" style="zoom: 33%;" />

```
B.code = E1.code || E2.code || gen('if',E1.place, relop.op, E2.place, 'goto', B.true) || gen('goto', B.false)
```

#### B -> true/false

```
B.code = gen('goto', B.true/false)
```



### 控制流语句

我们将条件判断设置两个出口，分别是B.true和B.false。注意这些有关地址、行号的属性都是继承属性。

而B.code，S.code这类顺序执行块的三地址指令序列是综合属性。

三地址码中，`newLabel()`返回一个新的标号，而函数 `gen()`产生一个三地址指令或标号，并把这个三地址指令或标号的串值作为返回值



> 关于这一块的地址生成，我的方式是，从上到下（从B.true/begin，到S.next的上方，将每一个属性赋值决定，然后通过gen按从上到下的顺序输出即可）。
>
> 当然从上到下不是指顺序遍历。比如像goto ... 这种，就是说上面那一块，比如说S1，它的下一步，S1.next = xxx

#### S -> if B then S1

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703203442882.png" alt="image-20260703203442882" style="zoom:50%;" />

```
B.true = newLabel();
B.false = S.next();
S1.next = S.next;
S.code = B.code || gen(B.true,':' || S1.code)
```

#### S -> if B then S1 else S2

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703210724643.png" alt="image-20260703210724643" style="zoom:67%;" />

```
B.true = newLabel();
B.false = newLabel();
S1.next = S.next;
S2.next = S.next;
S1.code = B.code || gen(B.true,":") || S1.code || gen('goto',S.next) || gen(B.false,':') || S2.code
```

#### S -> while B do S1

![image-20260703204150747](https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703204150747.png)

```
S.begin = newLabel();
B.true = newLabel();
B.false = S.next;
S1.next = S.begin; //这句是关键，不要漏了
S.code = gen(S.begin,':') || B.code || gen(B.true,':') || S1.code || gen('goto',S.begin)  
```

#### S -> S1; S2

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703204158631.png" alt="image-20260703204158631" style="zoom:43%;" />

```
S1.next = newLabel();
S2.next = S.next;
S.code = S1.code || gen(S1.next,':') || S2.code
```

### 布尔表达式的控制流翻译

##### 1. 尝试翻译 `a<b or c<d and e<f`



> 一般我们按照运算符优先级从低到高进行分区。如第一次分块我们按 or 分成如下结果：
>
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703214247665.png" alt="image-20260703214247665" style="zoom:67%;" />
>
> 然后接下来在B2中按 and 展开
>
> <img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/image-20260703215128244.png" alt="image-20260703215128244" style="zoom:67%;" />



```
			if(a<b) goto B.true
			goto B1.false

B1.false:	if(c<d) goto B3.true
			goto B.false
			
B3.true:	if(e<f)	goto B.true
			goto B.false
```

通常我们将各个真假出口替换为Li

##### 2. 翻译如下语句

```
while(a<b){
	if(c<d) x=y+z;
	else x=y-z;
}
```

画成图会好很多。依旧是分层，while分一层，然后if分一层

<img src="https://cdn.jsdelivr.net/gh/Pocon041/blogimage2@main/fa6a6310e17b34f53657d2630595c405.jpg" alt="fa6a6310e17b34f53657d2630595c405" style="zoom: 50%;" />

```
S.begin:	if(a<b)	goto B.true
			goto S.next
			
S.true:		if(c<d) goto B.true
			goto B.false

B.true:		t1 = y+z
			x = t1
			goto S.begin
			
B.false:	t2 = y+z
			x = t2
			goto S.begin
			
S.next:		
```

