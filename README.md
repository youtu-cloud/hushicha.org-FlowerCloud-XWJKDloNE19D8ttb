
## 一、需求背景


性能压测时，发现某接口存在性能瓶颈，期望借助工具定位该瓶颈，最好能定位至具体慢方法。


## 二、cProfile 简介


cProfile 是 Python 标准库中的一个模块，用于对 Python 程序进行性能分析，它能输出每个函数的调用次数、执行耗时等详细信息，可帮助开发者识别程序中运行缓慢的方法，以便进行性能优化，适合作为上述需求的解决方案。


此外，Python 还内置了使用纯 Python 实现的 profile 模块，与 cProfile 功能一样，只不过 cProfile 是用 C 语言编写，性能更高、开销更小，适合在性能敏感的环境（如线上生产环境）中使用。profile 是纯 Python 实现的模块，性能开销相对较大，但因其用 Python 编写，易于理解和修改，适合学习时使用。


## 三、使用方法


cProfile 支持三种使用方法，一是硬编码于代码中；二是在 Python 应用启动时加载 cProfile 模块；三是通过 IDE（PyCharm）运行。开发环境建议采用方法三，因其简单易用且结果图表丰富；生产环境建议采用方法二，此方法对代码无侵入性。


### 1\. 硬编码于代码中


**示例代码：**



```
import cProfile

def my_function():
    # Some code to profile
    pass

profiler = cProfile.Profile()
profiler.enable()
my_function()
profiler.disable()
profiler.print_stats()

```

**执行结果：**



```
2 function calls in 0.000 seconds

Ordered by: standard name

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
    1    0.000    0.000    0.000    0.000 test.py:3(my_function)
    1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}

```

**结果字段说明：**


ncalls：函数调用的次数。


tottime：在该函数中花费的总时间，不包括调用子函数的时间。


percall：tottime 除以 ncalls。


cumtime：该函数及其所有子函数中花费的总时间。


percall：cumtime 除以原始调用次数。


filename:lineno(function)：函数所在的文件名、行号和函数名。


### 2\. 在 Python 应用启动时加载 cProfile 模块


**示例代码：**



```
python -m cProfile my_script.py # 方法一、将结果输出至控制台
python -m cProfile -o output.prof my_script.py # 方法二、将结果保存到指定的prof文件

```

可使用 snakeviz 插件（安装方法为`pip install snakeviz`）分析 prof 文件。执行`snakeviz output.prof`后，会将结果挂载到 web 容器中，支持通过 URL（如`http://127.0.0.1:8080/snakeviz/xxxoutput.prof`）访问。


### 3\. 通过 IDE（PyCharm）运行


**使用方法：**


通过菜单 \[run] \> \[profile 'app']（其中 app 为应用名称，下同）启动应用。待应用执行完毕且停止后，相关面板会输出相应的调用统计与调用链。


**调用统计：**


![img](https://img2024.cnblogs.com/blog/2292062/202412/2292062-20241213003459513-782911418.png)


表头“Name”表示被调用的模块或函数；“Call Count”表示被调用次数；“Time (ms)”表示耗时及百分比，时间单位为毫秒。


点击表头列名可对该列进行排序。


在调用统计中，选择“name”列的单元格，右键选中“Navigate to Source”或“Show on Call Graph”，可打开其源码或对应的调用链及位置。


**调用链：**


![img](https://img2024.cnblogs.com/blog/2292062/202412/2292062-20241213003507690-356453407.png)


此外，通过菜单 \[run] \> \[Concurrency Diagram 'app'] 启动程序，可查看到线程和异步协程（Asyncio）的调用情况，如下图所示：


![img](https://img2024.cnblogs.com/blog/2292062/202412/2292062-20241213003519147-2006532557.png)


## 四、相关配置项


### 1\. cProfile



```
[root@test bin]# python3 -m cProfile -h
Usage: cProfile.py [-o output_file_path] [-s sort] [-m module | scriptfile] [arg] ...

Options:
  -h, --help            show this help message and exit
  -o OUTFILE, --outfile=OUTFILE
                        Save stats to  #  将分析结果输出到指定的文件中。
  -s SORT, --sort=SORT  Sort order when printing to stdout, based on pstats.Stats class # 指定输出结果的排序方式。可以根据不同的字段进行排序，如 time, cumulative, calls 等。
  -m                    Profile a library module # 分析一个模块，而不是一个脚本文件 

```

### 2\. snakeviz



```
[root@test bin]# snakeviz --help
usage: snakeviz [-h] [-v] [-H ADDR] [-p PORT] [-b BROWSER_PATH] [-s] filename

Start SnakeViz to view a Python profile.

positional arguments:
  filename              Python profile to view

options:
  -h, --help            show this help message and exit
  -v, --version         show program \`s version number and exit
  -H ADDR, --hostname ADDR hostname to bind to (default: 127.0.0.1) # 用于指定绑定的主机名，默认值为 127.0.0.1，即本地主机。
  -p PORT, --port PORT  port to bind to; if this port is already in use a free port will be selected automatically (default: 8080) # 用于指定绑定的端口。如果指定的端口已被占用，程序将自动选择一个空闲端口。默认值为 8080。
  -b BROWSER_PATH, --browser BROWSER_PATH  name of webbrowser to launch as described in the documentation of Python\'s webbrowser module: https://docs.python.org/3/library/webbrowser.html # 按照 Python 的 webbrowser 模块的文档描述，指定要启动的浏览器名称。用户可以通过指定浏览器的路径来控制使用哪个浏览器打开应用。
  -s, --server          start SnakeViz in server-only mode--no attempt will be made to open a browser # 仅在服务器模式下启动 SnakeViz，不会尝试打开服务器中的浏览器。对于非图形化或不带浏览器的服务器非常有用。

```

## 五、生产环境使用示例


生产环境系统版本为 CentOS 7\.9\.2009 (Core)，内核为 5\.15\.81，使用 4 核 4G 的容器运行[DB\-GPT](https://github.com) controller 子系统。


### 1\. 使用步骤


（1）执行脚本：`/usr/local/bin/python3.10 -m cProfile -o out.prof /usr/local/bin/dbgpt start controller &`，在 Python 应用启动时加载 cProfile 模块。


（2）执行相关接口压测。


（3）正常停止应用，生成性能分析结果文件（out.prof）。**注意：**性能分析结果只能是在程序正常停止后才能输出。常规两种做法：其一、后台守护进程，可以使用 `kill -2 {应用 PID}`；其二、前台进程，通过 Ctrl \+ C 退出。


（4）使用`snakeviz -H 0.0.0.0 -s out.prof`分析结果文件（其中 \-s 仅在服务端模式下运行，不会尝试打开服务器浏览器，一般情况下服务器不自带浏览器；\-H 0\.0\.0\.0，支持监听网卡所有接口），执行成功后，会输出可访问的 URL 地址，通过外部或本地浏览器打开。


### 2\. 结果分析


使用外部或本地浏览器访问 snakeviz 生成的 URL 地址。结果如下：


![](https://img2024.cnblogs.com/blog/2292062/202412/2292062-20241213003626969-1130237302.png)


**结果说明：**


（1）结果包含两部分，即上图和下表。上图展示选中方法及其子方法的调用关系、耗时及占比；下表展示所有方法及其总调用次数（ncalls）、方法本身总耗时（tottime）、方法本身平均耗时（percall）、方法及其子方法总耗时（cumtime）、方法及其子方法平均耗时（percall）以及方法所在文件位置及其行列号。


**使用说明：**


（1）表格任意列支持升降序操作，选中任意行，页面上方的图形会自动展示该方法及其子方法的调用关系、耗时及占比。


（2）单击图形中的任意子模块，可查看子模块所在方法及其子方法的调用关系、耗时及占比。


**分析建议：**


（1）选择 cumtime 列降序，选择入口代码，逐步细看，分析瓶颈点。


（2）使用 Sunburst 图样展示，更易体现各方法耗时占比。


### 3\. 评估加载 cProfile 对性能的影响


我们用 Jmeter 对未加载以及加载 cProfile 模块的 Python 应用性能进行评估，以判断生产环境加载 cProfile 对性能的影响程度。结果如下：




| 配置 | Jmeter 压测线程数 | CPU 使用率 | 吞吐量 | 平均响应时间 |
| --- | --- | --- | --- | --- |
| CASE1 某应用未加载 cProfile | 20 | 接近单核 100% | 527 | 36ms |
| CASE2 某应用加载 cProfile 后 | 20 | 接近单核 100% | 395 | 49ms |


从上表可知，加载 cProfile 后，应用吞吐量下降 25%，平均响应时间增加 13ms，对性能有一定影响。


## 五、遇到问题


### 1\. kill \-15 {应用 PID} 无法生成性能分析结果文件


由于 cProfile 仅支持监听中断（SIGINT）信号，导致 kill 15 发送 SIGTERM 信号时，无法生成性能分析结果文件。


解决办法：使用 kill \-2 {应用 PID}。


## 六、使用总结


（1）cProfile 可以生成详细的性能分布和调用链，非常适合作为分析和定位 Python 应用性能瓶颈的工具。


（2）由于生成性能分析结果文件需要停止应用，且对性能损耗较大（吞吐量降低 25%），所以一般情况下不建议在生产环境直接使用。不过可以使用流量复制，将生成环境的流量复制到测试或预生产环境，这样既能定位实际性能瓶颈，又不影响线上业务。


 本博客参考[FlowerCloud机场](https://hushicha.org)。转载请注明出处！
