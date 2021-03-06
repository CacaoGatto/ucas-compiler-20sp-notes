# 第七章 运行时刻环境

[toc]

## 静态与动态

- 将程序的静态概念与运行时对象/机制建立联系
- 运行时支撑数据包

## 源语言问题

- 过程是一个声明，最简单形式是一个标识符和一个语句联系。标识符称为过程名，语句称为过程体，函数是返回值的过程
- 过程调用：包括调用者、被调用者和调用点
- 参数：形参和实参
- 活动：过程的一次执行（不重叠，只能不交或嵌套）
- **活动树**表示过程的执行
  - 结点和活动一一对应
  - 调用序列对应前序遍历，返回序列对应后序遍历
  - 控制栈：保存活跃过程，表示活动树上到根节点的一条路径
- 名字与数据对象
  - 局部和非局部/作用域
  - 名字和动态数据对象的关系
  - 环境将名字映射到左值，状态将存储单元映射到右值。赋值只改变状态
  - 结合：把左值联系到名字，即把名字结合到存储单元
  - 名字结合要考虑递归、返回、局部性、传参、分配和释放等问题

## 存储组织

- 代码区、静态区、堆区、空闲内存、栈区
- 堆可存放动态数据，但开销大于栈
- 活动记录：过程的一次执行所需信息的存储区（实参、返回值、控制链、访问链、保存机器状态、局部量、临时量）
- 过程调用时活动记录入栈，返回时出栈
- 过程调用时可以确定大部分域的长度
- 考虑数据对齐

## 存储分配策略

### 静态分配

- 编译时实现名字结合（逻辑地址）
- 长度和位置限制须在编译时知道，不允许递归和动态数据结构
- 存放static数据

### 栈分配

- 递归过程
- 先入后出，允许动态
- 控制栈top不确定，局部量相对记录开始点偏移量可在符号表记录
- 过程调用的实现：
  - 过程调用代码序列，根据调用序列分配活动记录、保存填充信息，返回序列恢复机器状态
  - 代码分为两部分，被调用者代码只生成1次，调用者生成n次
- 调用者到被调用者传递的值放在被调用者活动记录开始处，固定长度的项放在中间（控制链访问链），尾部放早期不知大小的项（临时数据/动态数组）
- 栈顶指针放在固定长度尾部
- 调用序列和返回序列
- 参数个数不确定时保存描述参数的信息，如printf的第一个变量
- 变长数据如动态数组在栈分配，但不在活动记录中。动态数组大小确定后在该过程的活动记录紧邻的栈中分配空间
- 活动停止时局部名字不保持（悬空引用）
- 必须先入后出，被调用者活动不能比调用者晚结束

### 堆分配

- 需要时分配，释放次序任意
- C++中分配动态对象，C中动态分配指针
- 活跃记录不一定相邻，可能存在空洞
- 显示释放/隐式释放

## 访问非局部名字

- 静态作用域（访问链访问非局部名字，选择最接近的嵌套规则）/动态作用域（控制链访问非局部名字）
- 名字的存储分配
  - 基于栈分配
  - 每次为一个完整过程体分配空间
- 无过程嵌套的静态作用域
  - 过程定义不能出现在另一个过程内
  - 存储分配简单（相对偏移），过程声明可以作为参数传递或结果返回
- 有过程嵌套的静态作用域
  - 寻找最接近的嵌套声明
  - 访问链实现
  - 嵌套深度：主程序为1
- 静态链（访问链）
  - 指向包围本过程的直接外层过程的最近活动记录
  - 实现非局部名字访问，但速度慢
  - 建立：新记录深度大时指向调用者，相等时与调用者相同，小时追踪访问链查找其直接外层
  - 过程作为参数时访问链应一起传递
- 动态链（控制链）
  - 指向调用者
  - 新结合仅为局部名字建立
  - 被调用者的结合暂时覆盖调用者结合
  - 深访问
    - 根据控制链查找，省去访问链
    - 编译时不确定搜索深度
    - 访问非局部名字需要较长时间，但始末无需额外开销
  - 浅访问
    - 静态分配当前值
    - 原值记录在其活动记录中
    - 非局部名字和直接访问（静态空间，但活动始末需要维护开销）
- Display表（静态链）
  - 规模为最大深度
  - 第i项保存深度为i的最新记录

## 参数传递
