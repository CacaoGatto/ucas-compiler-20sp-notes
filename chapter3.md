# 第三章 词法分析

[toc]

## 词法分析的作用

- 读输入的字符流，产生用于语法分析的词法单元/记号序列
- 完成用户接口的部分任务
  - 源程序行信息的记录
  - 剥离注解和空白
  - 预处理宏
- 与语法分析分离，简化语法分析复杂性，方便语言设计
- 利于设计高效的词法分析程序，（传统上）改善编译器效率
- 增加编译器的可移植性（将设备相关的输入字符集的不规则性限制在词法分析程序中）
- 能力有限
- 剩余输入的前缀无法匹配时
  - 紧急方式：删除剩余输入最前面的字符，直至发现正确字符
  - 删除/插入/替换/交换

### 词法分析的基本概念

- 单词/词素（字符序列）
- 模式（单词的分类规则）
- 词法单元/记号（词法单元名+可选的属性值，是枚举型的语法终结符）
- 分隔符（一般是空格，但有例外，如FORTRAN）
- 关键字和保留字
- 语法单元/记号的属性
  - 属性的值反映记号的特性或特征的值
  - 通常一个记号具有一个属性值，也有些记号不需要
  - 模式能匹配多个单词时，需要为记号提供附加信息
  - 记号影响语法分析的决策，属性影响记号的翻译（二元组）

### 输入缓冲区

- 满足词法分析向前看若干字符（以确定词素符号性质）的需求
- 单独缓冲区：不能保证词素符号不会被缓冲区边界截断
- 缓冲区对：把缓冲区分为相同两部分
  - 每引用一次读入，填满半个缓冲区，不足时用EOF标明
  - 保持开始指针和向前指针，初始化都指向下一词素符号的首字符
  - 向前指针向前扫描，直至确定词素符号
  - 此时开始指针指向词素的首字符，故返回该词素
  - 开始指针指向该词素后首个符号，向前指针退一位
  - 向前指针到达缓冲区末尾时，更新另一半缓冲区
  - 设每个缓冲区长度为n，则单词符号长度不得超过n+1
- 优化：哨兵标志
  - 原因：向前指针需要两次测试：确认字符/检查缓冲区末尾
  - 扩展：在缓冲区尾加EOF标识，则只需在EOF处进行两次测试

## 记号的描述与正则式

- 正则式表示词法结构的串格式
- 正则式与3型文法等价，正则式表示的语言称为**正则集**
- 构成规则：或/连接/括号/指数（闭包）。运算均为左结合，优先级逐个提高
- 有限次使用构成规则的表达式才是正则式
- 正则式等价意味着表示相同的语言
- 正则定义
  - $d_i \rightarrow r_i$，且$r_i$是字母表和上文$d_j$集合并集上的正则式
  - 正则式命名：名字是元符号，但不能递归定义
  - 正则定义左端是名字，右端是正则式
  - 正则文法产生式左端是非终结符，右端是文法符号串
- 一元后缀算符+：一个或多个实例
- 一元后缀算符？：一个或零个实例
- 字符组：如[A-Za-z0-9]
- 非正则集：如配对结构、嵌套结构、重复串

## 记号的识别

- 跳过空白符
- 状态转换图
  - 初态/终态
  - 若识别完成后需要回退一位，标注*
  - 边：没有转出边且是终态时宣布结束
- 识别保留字
  - 初始化时将保留字填入符号表，识别符号后查符号表（一致处理）
  - 为关键字建立单独的状态转换图
- 未匹配且非终态时，撤回输入指针，搜索下一个转换图
- 不存在下一个转换图时报错
- 优化：
  - 并行运行各个状态转换图（采用**最长子串方式**）
  - 合并转换图

## 有穷状态自动机

- 识别正则表达式给出的串格式
- 描述能力与正则式、正则文法相同
- 当前状态概括了有关过去输入的信息

### 非确定有穷状态自动机

- 开始状态唯一，转到状态不唯一
- 接受某个符号串：存在接受状态的路径
- 转换函数：状态和输入符号的二元组到状态的映射的集合
- 转换表：快速查找状态转换，但可能浪费空间（稀疏矩阵，请）

### 确定有穷状态自动机

- NFA的特殊情况，无$\epsilon$转换，且同一个状态开始没有相同条件的边
- 一个串最多只有一条接收路径
- 接收相同语言的DFA等价

### NFA到DFA的转换

- 子集构造法
- 建立状态的$\epsilon$闭包
- 状态集合的$\epsilon$闭包
- 转换函数的$\epsilon$闭包
- 至少包含一个接受状态的状态集是新的接收状态
- 理论上存在指数变化，实际上很少发生
- 相当于并行模拟所有可能移动

### NFA模拟实现

- 双栈（当前状态集合/转换函数后的状态集合）
- 布尔数组标明已在后栈中的状态
- 二维数组转换表

### 算法效率

- DFA模拟时间复杂度O(|x|)
- NFA模拟实际时间复杂度正比于|x|和转换图大小的乘积
- DFA更快但转换图可能更大（子集构造法的最坏情况），进而影响性能

## 正则式与有穷状态自动机的等价

- Thompson算法
- 每次引入最多两个状态
- 构造或/连接/闭包的NFA
- 构造得到NFA状态数最多是正则式中符号和算符个数的两倍

### DFA的化简

- 每个正则集可由唯一一个状态数最少的DFA识别
- 假设每个状态对每个输入符号都有转换（对应死状态）
- 串区别两个状态，从而分为不相交子集
- 划分为接收状态和非接受状态；此后按每个输入符号递归划分
- 用状态代替状态组，去掉死状态和不可达状态

## 词法分析程序的自动生成工具

- Lex程序（通过转换表模拟DFA）
  - 把/视为$\epsilon$
