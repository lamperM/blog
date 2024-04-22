
## 源文件

模拟了一个调度器的行为，支持输入多个进程，每个进程可以包含一部分CPU计算指令和IO指令，IO指令可以设置等待时间。

http://pages.cs.wisc.edu/~remzi/OSTEP/Homework/HW-CPU-Intro.tgz

## 运行参数解析
利用python OptionParser 模块
```python
from optparse import OptionParser
```

生成实例，添加支持的参数列表：
```python
parser = OptionParser()
# -s ==> 短参数
# --seed ==> 长参数
# default ==> 默认值
# type ==> type of option value
# action='store' ==> store option value to dest
# action='store_true' ==> 无需携带参数，只要出现就认为True
# dest ==> member of parser for storing value
# help ==> description of this option, -h时会输出
parser.add_option('-s', '--seed', default=0, help='the random seed', action='store', type='int', dest='seed')
parser.add_option('-l', '--processlist', default='',
                  help='a comma-separated list of processes to run, in the form X1:Y1,X2:Y2,... where X is the number of instructions that process should run, and Y the chances (from 0 to 100) that an instruction will use the CPU or issue an IO',
                  action='store', type='string', dest='process_list')
parser.add_option('-L', '--iolength', default=5, help='how long an IO takes', action='store', type='int', dest='io_length')
parser.add_option('-S', '--switch', default='SWITCH_ON_IO',
                  help='when to switch between processes: SWITCH_ON_IO, SWITCH_ON_END',
                  action='store', type='string', dest='process_switch_behavior')
parser.add_option('-I', '--iodone', default='IO_RUN_LATER',
                  help='type of behavior when IO ends: IO_RUN_LATER, IO_RUN_IMMEDIATE',
                  action='store', type='string', dest='io_done_behavior')
parser.add_option('-c', help='compute answers for me', action='store_true', default=False, dest='solve')
parser.add_option('-p', '--printstats', help='print statistics at end; only useful with -c flag (otherwise stats are not printed)', action='store_true', default=False, dest='print_stats')
```

根据以上定义的规则，解析本次传入的参数列表。返回两个变量：
- options: <dest,value>的键值对。`options.seed` 访问seed参数的值
- args: 按照规则解析后残余的参数list
```python
(options, args) = parser.parse_args()
```


## 运行

输出我们给定的策略，包括每个进程的指令流，调度器的任务切换逻辑。
```python
if options.solve == False:
    print 'Produce a trace of what would happen when you run these processes:'
    for pid in range(s.get_num_processes()):
        print 'Process %d' % pid
        for inst in range(s.get_num_instructions(pid)):
            print '  %s' % s.get_instruction(pid, inst)
        print ''
    print 'Important behaviors:'
    print '  System will switch when',
    if options.process_switch_behavior == SCHED_SWITCH_ON_IO:
        print 'the current process is FINISHED or ISSUES AN IO'
    else:
        print 'the current process is FINISHED'
    print '  After IOs, the process issuing the IO will',
    if options.io_done_behavior == IO_RUN_IMMEDIATE:
        print 'run IMMEDIATELY'
    else:
        print 'run LATER (when it is its turn)'
    print ''
    exit(0)
```