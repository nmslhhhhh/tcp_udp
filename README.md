TCP:
TCP 程序说明文档
一、项目概述
---------------------
本项目包含一个 TCP 服务器程序（tcpserver.py）和一个 TCP 客户端程序（tcpclient.py），
通过自定义的简易协议，实现客户端将文本分块发送给服务器，服务器将每一块文本内容进行反转处理后返回，
客户端接收所有响应并输出最终的完整反转文本。
功能特点：
- 支持随机块大小的文本拆分；
- 使用自定义协议格式进行通信（含类型字段和数据长度）；
- 使用 `struct` 模块实现网络数据的打包与解包；
- 支持多线程并发服务多个客户端请求。

二、运行环境要求
1. Python 3.6 及以上版本；
2. 操作系统：Windows 或类 Unix 系统；
3. 无需安装额外第三方库（仅使用标准库：`socket`, `struct`, `threading`, `random`, `sys`）；
4. 建议在同一局域网或本机进行测试。

三、文件结构
---------------------
- `tcpserver.py`：TCP 服务器端脚本，接收客户端数据并返回反转字符串；
- `tcpclient.py`：TCP 客户端脚本，发送初始化及数据块，接收服务器响应；
- `input.txt`：客户端发送的原始文本内容（需用户自行提供）；
- `reversed_output.txt`：客户端接收的反转文本输出文件。

四、使用说明
1. **准备输入文件**
  在项目根目录创建或提供一个文本文件，命名为 `input.txt`，用于客户端读取数据。例如：

2. **启动服务器端和客户端**
打开命令行，运行：
python server.py
python client.py <server_ip> <server_port> <input_file> <Lmin> <Lmax>


参数说明：
<server_ip>：服务器IP地址，例如 127.0.0.1（本机）；

<server_port>：服务器端口，例如 9999；

<input_file>：要读取的原始文本文件（如 input.txt）；

<Lmin>：每个块最小长度（例如 10）；

<Lmax>：每个块最大长度（例如 20）。

系统会自动将输入文件按随机长度（介于 Lmin 和 Lmax）切割成若干段，并发送到服务器。

运行结果
客户端输出：
显示发送的每个文本块和对应的反转结果；
最终提示完整反转内容写入 reversed_output.txt 文件。

示例输出：
[Client] Sending Initialization: 5 blocks
[Client] Received Agree from server.
[Client] Block 1: .txet siht esreveR
[Client] Block 2: !gnitset rof elif tset A
...
[Client] Full reversed text written to reversed_output.txt

UDP:
UDP 程序说明文档
=====================

一、项目概述
---------------------
本项目模拟了基于 UDP 协议的“可靠数据传输协议”，实现了以下特性：
- 客户端将文本文件内容分片，通过滑动窗口协议发送至服务器；
- 服务器端模拟丢包（按概率丢弃数据包）；
- 客户端实现了基于超时的 Go-Back-N 重传机制；
- 客户端记录 RTT（往返时间）并输出统计信息；
- 服务端通过 ACK 响应机制，保证可靠传输。

适用于计算机网络课程设计中“UDP 可靠传输”实验任务。

二、运行环境要求
---------------------
1. Python 3.6 及以上；
2. 依赖库：
   - `pandas`（客户端统计 RTT 使用）——安装方法：`pip install pandas`；
3. 建议在同一局域网或本机测试，避免外网影响丢包模拟。

三、文件说明
---------------------
- `udpserver.py`：服务器端脚本，模拟丢包接收并发送 ACK；
- `udpclient.py`：客户端脚本，负责文件分片发送、重传与RTT统计；
- `input.txt`：客户端发送的输入文本文件；
- 运行结束后客户端输出：丢包率、RTT 最大/最小/平均值等。

四、协议设计
---------------------
1. **握手阶段**  
客户端发送 `HELLO`，服务器返回 `WELCOME` 表示准备就绪。

2. **数据包结构（客户端 → 服务器）**  
- 头部（6字节）：
  - 4 字节：序列号（`!I`，无符号整数）
  - 2 字节：数据长度（`!H`，无符号短整数）
- 数据体：字符串内容的 UTF-8 编码字节

3. **ACK 报文结构（服务器 → 客户端）**  
- 4 字节：确认的序列号（`!I`）

五、使用方法
启动服务端：
python udpserver.py
默认监听在本地所有网卡的 9999 端口；
可通过修改 udpserver.py 文件中的 SERVER_PORT 和 LOSS_PROB 来调整监听端口与模拟丢包概率（如 LOSS_PROB=0.3 表示 30% 概率丢包）。

启动客户端：
python udpclient.py <server_ip> <server_port>
示例：
python udpclient.py 127.0.0.1 9999

说明：
<server_ip> 为服务器的 IP 地址；
<server_port> 为服务器的端口号，必须与服务端一致；
客户端程序读取 input.txt，将数据切分为固定大小数据包（默认每包 80 字节），并通过 UDP 发送到服务器。

运行结果：
客户端将实时显示包发送、重传、确认状态及 RTT；
传输结束后，输出丢包率、最大/最小/平均 RTT、RTT 标准差；
服务端实时输出收到的包、是否丢弃、是否发送 ACK 等信息。

配置选项
以下为常用配置项，可在代码中修改：
PACKET_SIZE：单个包的大小（单位：字节，默认80）
WINDOW_SIZE_BYTES：发送窗口总大小（单位：字节，默认400）
LOSS_PROB：服务端模拟丢包的概率（0.0 ~ 1.0）
TIMEOUT：客户端接收 ACK 的等待超时时间（单位：秒，默认0.3）


