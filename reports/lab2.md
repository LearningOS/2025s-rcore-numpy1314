# lab2
### 重写 sys_get_time 和 sys_trace
#### 对于sys_get_time()来说，对指针 *ts 所指向的内存空间赋值时间信息，在引入虚存之前应用空间和内核空间之间不存在隔离，二者都可以直接访问到 *ts 所在位置。在引入虚存之后，每个应用以及内核本身都有独立的地址空间，没办法访问了，所以就由内核要处理，且TimeVal可能是跨页的，不过translated_byte_buffer能正确处理这些问题
#### 对于sys_trace()来说，主要就是trap进内核态了之后_id是虚拟地址空间，就用translated_byte_buffer()进行读写

### sys_mmap()
#### 主要还是用find_pte()检查页是否已被分配，和查看剩余页面数，然后参照insert_frame_area()进行分配

### sys_munmap()
#### 其实已经有了unmap，就检查一下地址范围，用Vpn_range()看看内存页是否对的上然后调用unmap就行