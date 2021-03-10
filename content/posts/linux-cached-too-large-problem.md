---
title: "Linux Cached 过大问题"
date: 2021-03-10T16:44:27+08:00
draft: false
toc: false
images:
tags:
  - Linux
---

## Linux Cached 过大问题

某日,愉悦Coding.

负责人抛过来一个问题:“XXX说咱们的服务内存使用量过大,你接下来看看这个问题.”

此时的我一脸蒙蔽的ssh到VM上输入如下命令 ```free -h``` 此时bash显示

```bash
free -h
            total        used        free      shared  buff/cache   available
Mem:          11G        1.5G        3.5G        184M        6.6G        9.6G
Swap:        4.0G          0B        4.0G
```

懵逼的我不禁发出了感叹--这内存状况良好得运维看了都想给机器减配!

只好再看看服务的状况了 于是我敲下了以下命令 ```top``` 映入眼帘的景象让我大呼上当

```bash
 PID USER   PR  NI VIRT RES    SHR S  %CPU %MEM     TIME+ COMMAND
 **** ****  20   0 **** ****  **** S  21.3  2.2 147:22.09 node /home/www
 **** ****  20   0 **** ****  **** S  14.0  2.2 136:32.45 node /home/www
 **** ****  20   0 **** ****  **** S   5.3  2.6 145:11.27 node /home/www
 **** ****  20   0 **** ****  **** S   3.0  2.2 147:21.91 node /home/www
```

平均每个进程3%不到的占用,怎么就内存使用过大了?

由于感觉到事情已经超出了自己的认知能力,只好找到负责人:“我看服务状况蛮好啊,XXX为啥说过大?”

XXX加入会话:“监控系统显示内存剩余量只有1.5G左右了,怕流量打起来出故障.”

我一脸问号,并贴出了大大的❓❓❓

老哥怕不是把cached内存算在不可使用内存范围了,后续沟通证明事实确实如此,误会接触.

### 那么为啥会产生这样的误会呢

我们搞前端的兄弟们写习惯了UI,少有接触linux知识,我们先来了解下Linux下的buff/cache memery 到底是个啥?

- buff（Buffer Cache）是一种I/O缓存，用于内存和硬盘的缓冲，是io设备的读写缓冲区。根据磁盘的读写设计的，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而提高系统性能。

- cache（Page Cache）是一种高速缓存，用于CPU和内存之间的缓冲 ,是文件系统的cache。把读取过的数据保存起来，重新读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除。

有心的小伙伴可以扩展查看[《Linux System Administrators Guide:Chapter 6. Memory Management》](https://tldp.org/LDP/sag/html/buffer-cache.html)

简单点来说:Buffer Cache是即将写入磁盘的数据,Page Cache则是从磁盘中读取的数据.他们都占用内存,且都存储在RAM中.

不过它们虽然都占用内存但是在合适的时机会被内核释放掉,而一般情况下不需要我们去关注cached内存,它的存在可以增加文件的读写性能.

那么看到这里的看官可能已经猜到为啥我们这个服务的cached内存占用8Gb了--是的没错,写日志文件!

好了今天的总结就到此为止.顺带贴上一篇贼出名的文章[《linux ate my ram》](https://www.linuxatemyram.com/)

### 但是有的人看着不爽

既然看着不爽,那就只能手动或者定时任务来清理它啦!

手动清理

```bash
server# sync #一定要记得这个写入硬盘，防止数据丢失
server# echo 1 > /proc/sys/vm/drop_caches
server# echo 1 > /proc/sys/vm/drop_caches
server# echo 1 > /proc/sys/vm/drop_caches
```

写成脚本

```bash
#!/bin/bash
echo "清除cached缓存"
sync
sleep 10
echo 1 > /proc/sys/vm/drop_caches
echo 2 > /proc/sys/vm/drop_caches
echo 3 > /proc/sys/vm/drop_caches
```

然后就是创建该脚本的定时任务啦!
