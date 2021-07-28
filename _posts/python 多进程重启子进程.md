# python 多进程重启子进程



## 需求背景

在多个进程任务中，需要对实时的添加、重启子进程的功能



## 解决

```python
import multiprocessing as np
import time
import os


def run(device):
    while True:
        print(f"{device} 号开始任务")
        print("----in 子进程 pid=%d ,父进程的pid=%d---" % (os.getpid(), os.getppid()))
        # print('================================')
        time.sleep(5)


def main():

    processes = {}
    for device in ["test1", "test2"]:
        p = np.Process(target=run, args=(device,))
        p.start()

        processes[device] = (p, device)

    while len(processes) > 0:
        # 监控是否有新设备进来
        for key in list(processes.keys()):
            p, device = processes[key]
            if p.exitcode is None and not p.is_alive():
                print("Not finished and not running")
                pass
            elif p.exitcode is None:
                pass
            elif p.exitcode < 0:
                print("进程以错误或终止结束")
                # 被杀死进程后自动重启
                processes.pop(key)

                p = np.Process(target=run, args=(device,))
                p.start()

                processes[device] = (p, device)
```

模拟需要重启一个子进程

```python
import os, signal


os.kill(87971, signal.SIGKILL)

```

