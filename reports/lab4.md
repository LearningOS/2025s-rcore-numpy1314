# lab4

### 编程作业
硬链接
硬链接要求两个不同的目录项指向同一个文件，在我们的文件系统中也就是两个不同名称目录项指向同一个磁盘块。

本节要求实现三个系统调用 sys_linkat、sys_unlinkat、sys_stat 。

linkat：

syscall ID: 37

功能：创建一个文件的一个硬链接， linkat标准接口 。

Ｃ接口： int linkat(int olddirfd, char* oldpath, int newdirfd, char* newpath, unsigned int flags)

Rust 接口： fn linkat(olddirfd: i32, oldpath: *const u8, newdirfd: i32, newpath: *const u8, flags: u32) -> i32

参数：
olddirfd，newdirfd: 仅为了兼容性考虑，本次实验中始终为 AT_FDCWD (-100)，可以忽略。

flags: 仅为了兼容性考虑，本次实验中始终为 0，可以忽略。

oldpath：原有文件路径

newpath: 新的链接文件路径。

说明：
为了方便，不考虑新文件路径已经存在的情况（属于未定义行为）。除非出现新旧名字一致的情况，此时需要返回-1。

返回值：如果出现了错误则返回 -1，否则返回 0。

可能的错误
链接同名文件。

unlinkat:

syscall ID: 35

功能：取消一个文件路径到文件的链接, unlinkat标准接口 。

Ｃ接口： int unlinkat(int dirfd, char* path, unsigned int flags)

Rust 接口： fn unlinkat(dirfd: i32, path: *const u8, flags: u32) -> i32

参数：
dirfd: 仅为了兼容性考虑，本次实验中始终为 AT_FDCWD (-100)，可以忽略。

flags: 仅为了兼容性考虑，本次实验中始终为 0，可以忽略。

path：文件路径。

说明：
注意考虑使用 unlink 彻底删除文件的情况，此时需要回收inode以及它对应的数据块。

返回值：如果出现了错误则返回 -1，否则返回 0。

可能的错误
文件不存在。

fstat:

syscall ID: 80

功能：获取文件状态。

Ｃ接口： int fstat(int fd, struct Stat* st)

Rust 接口： fn fstat(fd: i32, st: *mut Stat) -> i32

参数：
fd: 文件描述符

st: 文件状态结构体

#[repr(C)]
#[derive(Debug)]
pub struct Stat {
    /// 文件所在磁盘驱动器号，该实验中写死为 0 即可
    pub dev: u64,
    /// inode 文件所在 inode 编号
    pub ino: u64,
    /// 文件类型
    pub mode: StatMode,
    /// 硬链接数量，初始为1
    pub nlink: u32,
    /// 无需考虑，为了兼容性设计
    pad: [u64; 7],
}

/// StatMode 定义：
bitflags! {
    pub struct StatMode: u32 {
        const NULL  = 0;
        /// directory
        const DIR   = 0o040000;
        /// ordinary regular file
        const FILE  = 0o100000;
    }
}

### 简要过程
linkat和unlinkat

首先对于创建硬链接来说，得先在diskinode加一个nlink用于引用计数，创建硬链接就先find_inode_id，然后modify_disk_inode为根增加direntry，name就是链接名，然后就创建完成，在通过目标文件的inode用modify增加引用计数nlink即可

unlinkat类似，先删direntry然后找到对应inode，引用计数-1，加一个判断如果nlink为0了就删文件回收inode和数据块

对sys_fstat来说，在Inode中增加get_inode_id、get_mode、get_nlink等函数，从fd_table中我们能拿到OSInode，OSInode中能拿到Inode，从而可以调用增加的函数获取文件信息

