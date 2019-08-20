---
title: Linux文件描述符
date: 2018/05/28 19:07:09
tags:
    - Linux
    - 文件描述符
categories:
    - Linux
comments: true
---

> Linux一切皆文件

`Linux文件描述符`是一个非负整数，它是一个索引值，指向内核中每个进程打开的文件记录；当打开一个现有文件或者创建文件，内核返回一个文件描述符给进程用于对文件的后续操作；当需要读写文件时，也需要将文件描述符传递给相应的函数。
<!-- more -->
当程序加载进内存后，即成为一个进程，系统会默认打开三个文件，标准输入(stdin)、标准输出(stdout)和标准错误(stderr)。这三个打开的文件对应的文件描述符分别为0、1、2。因此就有了我们经常看到的一个写法: `2>&1`，把标准错误重定向到标准输出。

文件描述符与对应文件的关系图如下(图片来源[CSDN keep_belief](https://blog.csdn.net/qq_34992845/article/details/71446333)):![关系图](https://img-blog.csdn.net/20170509121308111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ5OTI4NDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## FILE结构体
```c
struct file {
    /*
     * fu_list becomes invalid after file_free is called and queued via
     * fu_rcuhead for RCU freeing
     */
    union {
        struct list_head    fu_list;
        struct rcu_head     fu_rcuhead;
    } f_u;
    struct path     f_path;
#define f_dentry    f_path.dentry
#define f_vfsmnt    f_path.mnt
    const struct file_operations    *f_op;
    spinlock_t      f_lock;  /* f_ep_links, f_flags, no IRQ */
    atomic_long_t       f_count;
    unsigned int        f_flags;
    fmode_t         f_mode;
    loff_t          f_pos;
    struct fown_struct  f_owner;
    const struct cred   *f_cred;
    struct file_ra_state    f_ra;

    u64         f_version;
#ifdef CONFIG_SECURITY
    void            *f_security;
#endif
    /* needed for tty driver, and maybe others */
    void            *private_data;

#ifdef CONFIG_EPOLL
    /* Used by fs/eventpoll.c to link all the hooks to this file */
    struct list_head    f_ep_links;
#endif /* #ifdef CONFIG_EPOLL */
    struct address_space    *f_mapping;
#ifdef CONFIG_DEBUG_WRITECOUNT
    unsigned long f_mnt_write_state;
#endif
};
```

文件描述符与FILE struct和file_operator关系如下：
![关系图如下](https://wechat-lvyun.oss-cn-shenzhen.aliyuncs.com/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png?Expires=1527514942&OSSAccessKeyId=TMP.AQEUc_bURUf15ynTngG2tV3uao75kBMoXtD9ybhBvzJ5BWUv3iXBSaSMourLADAtAhRLrY9sJbmw_5qIA33PtYbgAVAF4wIVAMUgnjqoG5Z6IdaL_B9JEDgLM8a8&Signature=Su1Fp1sQUHINWU%2B6tzMDjZ0beB0%3D)

## 总结
文件描述符的分配数量是有限的，最大值为0 ~ OPEN_MAX-1。
文件描述符是操作系统用来管理文件的数据结构；我们执行应用，即生成进程时，内核自动生成文件描述符表，同时会自动打开三个文件，进程控制块PBC中的fs指针指向文件描述符表；当我们创建或者打开文件，会指向该文件的指针FILE*关联一个文件描述符并添加到文件描述符表中。在文件描述符表中fd相当于数组的索引，FILE*相当于数组的值，指向一个FILE结构体。