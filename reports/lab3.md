# lab3

### 编程作业
编程作业
关于之前的 syscall
你仍需要迁移上一章的 sys_get_time sys_mmap sys_munmap 以适应新的进程结构。不过，从本章节开始，不再要求维护 ``sys_trace`` 这一系统调用。

进程创建
大家一定好奇过为啥进程创建要用 fork + exec 这么一个奇怪的系统调用，就不能直接搞一个新进程吗？ 思而不学则殆，我们就来试一试！这章的编程练习请大家实现一个完全 DIY 的系统调用 spawn，用以创建一个新进程。

spawn 系统调用定义( 标准spawn看这里 )：

fn sys_spawn(path: *const u8) -> isize
syscall ID: 400

功能：新建子进程，使其执行目标程序。

说明：成功返回子进程id，否则返回 -1。

可能的错误：
无效的文件名。

小心

虽然测例很简单，但提醒读者 spawn 不必 像 fork 一样复制父进程的地址空间。

stride 调度算法
ch3 中我们实现的调度算法十分简单。现在我们要为我们的 os 实现一种带优先级的调度算法：stride 调度算法。

算法描述如下:

(1) 为每个进程设置一个当前 stride，表示该进程当前已经运行的“长度”。另外设置其对应的 pass 值（只与进程的优先权有关系），表示对应进程在调度后，stride 需要进行的累加值。

每次需要调度时，从当前 runnable 态的进程中选择 stride 最小的进程调度。对于获得调度的进程 P，将对应的 stride 加上其对应的步长 pass。

一个时间片后，回到上一步骤，重新调度当前 stride 最小的进程。

可以证明，如果令 P.pass = BigStride / P.priority 其中 P.priority 表示进程的优先权（大于 1），而 BigStride 表示一个预先定义的大常数，则该调度方案为每个进程分配的时间将与其优先级成正比。证明过程我们在这里略去，有兴趣的同学可以在网上查找相关资料。

其他实验细节：

stride 调度要求进程优先级 > 2
，所以设定进程优先级 ≤ 1
 会导致错误。

进程初始 stride 设置为 0 即可。

进程初始优先级设置为 16。

为了实现该调度算法，内核还要增加 set_prio 系统调用

// syscall ID：140
// 设置当前进程优先级为 prio
// 参数：prio 进程优先级，要求 prio >= 2
// 返回值：如果输入合法则返回 prio，否则返回 -1
fn sys_set_priority(prio: isize) -> isize;

### 简要过程
迁移三个函数其实不困难，主要是taskmanager被分成了TaskManager和Processor

关于spawn，结合TaskControlBlock的fork和exec方法，从中选出直接spawn应该如何构造TaskContext和TrapContext

stride 调度算法
TaskControlBlockInner中增加字段stride: u8和priority: u8，然后暴力遍历ready_queue，选出stride值最小的TaskControlBlock，其实应该把 Task 塞到 ready_queue 的时候，根据 stride 的值判断要塞到那里，这样下一次fetch的时候就可以直接拿到最小的，直接整一个小顶堆的，但是说实话这次训练营没时间了



