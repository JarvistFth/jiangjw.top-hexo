---
title: MIT6.824-LAB2-Summary
date: 2020-11-04 22:05:09
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---
6.824 lab2的总结，以及参考过的一些博文。
<!---more--->

## 总结

虽然lab2磕磕绊绊写完了，但是debug过程还是略显艰辛。

因为开始的时候并没有好好的看助教的instruction和一些文章。

raft架构：http://nil.csail.mit.edu/6.824/2020/labs/raft-structure.txt

raft锁：http://nil.csail.mit.edu/6.824/2020/labs/raft-locking.txt

助教博客student guide to raft：https://thesquareplanet.com/blog/students-guide-to-raft/

做之前一定要都看了才比较好做。

同时，还要严格遵守论文的所有规则；在遵守了论文里的规则以后如果还有问题，再严格查看上面三篇文章。一般来说花时间打log看log去debug，效率远不如看一下文章里面有什么地方没有遵循。。

对于我这种打了一大堆log看了很久的人，看log实在是一个很费时间并且效率很低的事情。。

下面说说我都遇到过的问题。

### 锁的使用
golang不提供raii，所以在lock的时候最好还是直接一把大锁 defer unlock。

如果发现死锁，可以在外面封装一层，在lock前打印日志，unlock后打印日志用作debug。

但是之前写了个webserver，所以对lock还是比较注意的。我尽量都会一把大锁去加。

部分没有用defer 解锁的时候，确实也难免会忘记unlock（特别是那些不符合情况直接return的函数里头，很容易就忘记unlock了）。

这时候go -test -race就很好用了。

还有就是注意在发送RPC前一定是要解锁的，不然就很可能会死锁。

### live locks
活锁，就是系统一直能运行下去，但是就是没有进展，不停地发送lock但是系统没有进展。

助教文章说了很可能是因为electionTimeout没有正确重置的原因。我也遇到了，确实是，需要注意只有三个地方才reset electionTimeout。

1. you get an AppendEntries RPC from the current leader (i.e., if the term in the AppendEntries arguments is outdated, you should not reset your timer); 
2. you are starting an election; or 
3. you grant a vote to another peer.

特别是第三点，只有真正vote了，才reset自己的electionTimeout

### RPC乱序
这里lab2c也说过了，就是在收到rpc回复的时候还要判断一下这个回复是不是已经过时了。

### timer的使用
timer确实很难使用，强如etcd也只是用了一个定时器ticker去做定时任务。在我all follow 上面的instruction的时候，在过TestFigure8Unreliable的时候还是有bug。我就知道timer确实是出问题了，在网上找了很久也没有找到没问题的、使用timer的代码。

所以最好还是使用sleep()然后定时check一些变量去做定时器。

用time.After也可以，但是time实现定时器最好只实现一个，不然不知道会不会还有奇怪的问题。

### 测试脚本
没有用助教提供的shell脚本，因为我用oh-my-zsh跑的时候第一个总是fail，就很奇怪。借鉴了一位大佬的Python脚本，然后根据goland的使用特性，改了下东西。

```Python
from multiprocessing import Pool
import multiprocessing
from typing import List
import subprocess
import os
import sys


cnt = multiprocessing.Value("i", 0)

def run_shell_and_get_result(args: List[str]) -> str:
    # print(f'executing {" ".join(args)}')
    return subprocess.run(args, stdout=subprocess.PIPE).stdout.decode().strip()


def run_one_test(test_name: str, i: int, log_path: str, time_limit: str, total: int):
    # print(f'running test {i}...')
    output = run_shell_and_get_result(['go', 'test', '-run', test_name, '-timeout', time_limit])
    result = output.split('\n')[-1][:2]
    if result == 'ok':
        cnt.value += 1
        print(f'test {i} passed, total passed: {cnt.value}/{total}')
        # with open(f"{log_path}/logs/debug_{i}_{test_name}_passed.txt", 'w') as f:
        #     f.write(output)
    else:
        print(f'test {i} failed')
        with open(f"{log_path}/logs/debug_{i}_{test_name}_failed.txt", 'w') as f:
            f.write(output)


if __name__ == '__main__':
    import argparse
    os.system('rm ./logs/*')
    os.environ["GOPATH"] += os.pathsep + '/home/jarvist/6.824'
    print(os.environ["GOPATH"])
    parser = argparse.ArgumentParser(description="process core num and test time")
    parser.add_argument('test_name', type=str, help='the name of the test function')
    parser.add_argument('-np', dest='num_process', type=int, default=1, required=False, help='number of processes used')
    parser.add_argument('-t', dest='test_times', type=int, default=1, required=False, help='number of test times')
    parser.add_argument('-o', dest='debug_output', type=str, default='.', required=False,
                        help='output directory path for debug log')
    parser.add_argument('-l', dest='limit', type=str, default='10m0s', required=False, help='time limit for test')
    args = parser.parse_args()
    p = Pool(args.num_process)
    test_name = args.test_name
    # print(f'test_name: {test_name}')

    p.starmap(run_one_test,
              [(test_name, i, args.debug_output, args.limit, args.test_times) for i in range(args.test_times)])

```

用这个脚本跑了很多代码，发现网上很多的代码跑一次test没问题，多跑几次还是会有fail的情况。

### 最后
附上图：500次test all passed

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/image.png)

一些看过的文章：

[1]. MIT 6.824 2020 Raft 实现细节汇总 https://zhuanlan.zhihu.com/p/103849249

[2]. MIT 6.824 Lab2 Raft实现全详解 https://zhuanlan.zhihu.com/p/261957679 

[3]. MIT 6.824 lab2c https://zhuanlan.zhihu.com/p/259777129

[4]. student guide 大神的总结：https://zhuanlan.zhihu.com/p/260189428

[5]. 一步一步带你实现raft（2C） https://www.jianshu.com/p/59a224fded77

[6]. timer.Reset的正确使用姿势：https://studygolang.com/articles/9289

[7]. raft lab 2c 使用timer的代码：https://www.cnblogs.com/mignet/p/6824_Lab_2_Raft_2C.html