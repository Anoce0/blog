---
title: ring-buffer 环形缓冲区
date: 2022-03-17 23:08:07
tags: 
    - ios
    - tcp/ip
    - data structure
---

## 前言

最近做 iOS 相关的项目的时候遇到一个 bug: client 发给 server 的数据发着发着会乱掉, server 收到的数据包被截断或者迟滞, 调查之后发现是由于项目底层使用 circuler buffer (也叫 ring buffer, 环形缓冲区) 使用的 cbUsed 来表示已缓冲的 buffer 长度, 但是没有给这个变量加锁, 导致并发时出现 bug.
所以研究了一下环形缓冲区的原理和特性.

## 什么是环形缓冲区以及其实现

环形缓冲区可以参考 Wikipedia 介绍页面的[这张图示](https://upload.wikimedia.org/wikipedia/commons/f/fd/Circular_Buffer_Animation.gif)
它实际上是一个数组, 使用两个指针分别用于写入和读取, 指正移动到数组末尾后, 就会跳到数组的起始位置进行循环, 实际缓冲的数据存在于 tail 到 head 之间的区域之内
具体流程如下:
初始化一个数组 A, 数组长度 length 最大缓冲长度, 设置两个指针分别是  head 和 tail 指向数组开头

- 写入数据时:
  - 从 head 指针所在位置往后写入, 写入完成后更新 head 指针指向到最后写入位置的下一位
  - 当写入到数组末尾是, 从将 head 指针移动到数组开头, 继续写入
  - 当写入到 tail 时, 表示数组已满, 停止写入
- 读取数据时:
  - 从 tail 指正所在位置往后读取, 读取完成后将 tail 指正指向到最后读取位置的下一位
  - 当读取到数组末尾时, 将 tail 指针移动到数组开头继续读取
  - 当读取到 head 指针所在位置时, 表示读取完毕, 停止读取

具体实现代码如下

```c
int BUFFER_LENGTH = 1024;

byte*  pBuffer; // circuler buffer 
byte*  pBufferEnd; // circular buffer end
byte*  pHead; // head pointer
byte*  pTail; // tail pointer

-(void)alloc_cir_buffer() 
{
    pBuffer =  host_malloc(BUFFER_LENGTH);
    pbufferEnd = pBuffer + BUFFER_LENGTH;
    pHead = pTail = pBuffer;
}
-(void)delloc_cir_buffer()
{
    free(pBuffer);
}

-(int)buffer_length() // 计算 buffer 已经占用的长度
{
    if(pHead < pTail) 
    {
        return (pBufferEnd - pTail) + (pBufferHead - pBuffer);
    } else 
    {
        return pHead - pTail;
    }
}

-(void)read_cir_buffer(&readBuffer, length, &outLength)
{
    int readLenth = MIN(length, buffer_length());
    *outLength = readLenth;
    if(pTail + readLength <= pBufferEnd) 
    {   
        memcpy(readBuffer, pBuffer, readLenth);
        pTail += readLenth;
    } else 
    {   // 数据区间跨国了 bufferEnd
        int part1 = pBufferEnd - pTail;
        int part2 = readLenth = part1;
        memcpy(readBuffer, tail, part1);
        memcpy(readBuffer + part1, pBuffer, part2);
        pTail = pBbufer + part2;
    }
}

-(int)write_cir_buffer(&writeBuffer, length)
{
    int freeLength = pBufferEnd - pBuffer - buffer_length();
    if(freeLenth < lenth) {
        return CLIENT_ERROR_BUFFER_FULL;
    }
    if(pHead + freeLength <= pBufferEnd)
    {
        memcpy(pHead, *writeBuffer, length);
        pHead += length
    } else 
    {   // 写入区间跨国了 bufferEnd
        int part1 = pBufferEnd - pHead;
        int part2 = length - part1;
        memcpy(*writeBuffer, pHead, part1);
        memcpy(*writeBuffer + part1, pBuffer, part2);
        pHead = pBuffer + part2;
    }
}
```

## 环形缓冲区的优势

从上述过程可以看出, 环形缓冲区在读取和写入时只需要移动 head / tail 指针即可, 相比如使用 queue 结构来说,在读取和写入时不需要移动数据只, 性能更高.

## 环形缓冲区在多线程情况下的处理

通常这种缓冲区都用于流处理,例如 TCP/IP 实现, 业务层往缓冲区写入, 驱动层从缓冲区读取, 单一生产者单一消费者, 在这种情况下, 由于head / tail 指针和缓冲数组成员都不存在同时写操作, 也就是线程安全的,但是如果我们需要生成一个变量 bufferUsed 来表示缓冲区已经使用的长度, 这个变量就会同时在 生产端 和消费端 被写入, 会有线程安全问题, 需要对其加读写锁.
