# 概述
消息队列是消息的链表，存放在内核中并由消息队列标识符表示。

1.消息队列提供了一个从一个进程向另一个进程发送数据块的方法，每个数据块都可以被认为是有一个类型，接受者接受的数据块可以有不同的类型。

2.但是同管道类似，它有一个不足就是每个消息的最大长度是有上限的(MSGMAX)，每个消息队列的总的字节数(MSGMNB)，系统上消息队列的总数上限(MSGMNI)。可以用cat /proc/sys/kernel/msgmax查看具体的数据。

3.内核为每个IPC对象维护了一个数据结构struct ipc_perm，用于标识消息队列，让进程知道当前操作的是哪个消息队列。每一个msqid_ds表示一个消息队列，并通过msqid_ds.msg_first、msg_last维护一个先进先出的msg链表队列，当发送一个消息到该消息队列时，把发送的消息构造成一个msg的结构对象，并添加到msqid_ds.msg_first、msg_last维护的链表队列。


生命周期随内核，消息队列会一直存在，需要我们显示的调用接口删除或使用命令删除

消息队列可以双向通信

克服了管道只能承载无格式字节流的缺点

## 1.msgget
功能：创建和访问一个消息队列
原型：
"""
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgget(key_t key, int msgflag);
"""
参数：
key：某个消息队列的名字，用ftok()产生
msgflag：有两个选项IPC_CREAT和IPC_EXCL，单独使用IPC_CREAT，如果消息队列不存在则创建之，如果存在则打开返回；单独使用IPC_EXCL是没有意义的；两个同时使用，如果消息队列不存在则创建之，如果存在则出错返回。
返回值：成功返回一个非负整数，即消息队列的标识码，失败返回-1
"""
#include <sys/types.h>
#include <sys/ipc.h>
key_t ftok(const char *pathname, int proj_id);
"""
调用成功返回一个key值，用于创建消息队列，如果失败，返回-1

## 2.msgctl
功能：消息队列的控制函数
原型：
"""
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
"""
参数：
msqid：由msgget函数返回的消息队列标识码
cmd：有三个可选的值，在此我们使用IPC_RMID
IPC_STAT 把msqid_ds结构中的数据设置为消息队列的当前关联值
IPC_SET  在进程有足够权限的前提下，把消息队列的当前关联值设置为msqid_ds数据结构中给出的值
IPC_RMID 删除消息队列
返回值：成功返回0，失败返回-1

## 3.msgsnd
功能：把一条消息添加到消息队列中
原型：
"""
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
"""
参数：
msgid：由msgget函数返回的消息队列标识码
msgp：指针指向准备发送的消息
msgze：msgp指向的消息的长度（不包括消息类型的long int长整型）
msgflg：默认为0
返回值：成功返回0，失败返回-1
消息结构一方面必须小于系统规定的上限，另一方面必须以一个long
int长整型开始，接受者以此来确定消息的类型
struct msgbuf
{
     long mtye;
     char mtext[1];
};

## 4.msgrcv
功能：是从一个消息队列接受消息
原型：
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
参数：与msgsnd相同
返回值：成功返回实际放到接收缓冲区里去的字符个数，失败返回-1

此外，我们还需要学习两个重要的命令
前面我们说过，消息队列需要手动删除IPC资源
ipcs:显示IPC资源
ipcrm:手动删除IPC资源 


