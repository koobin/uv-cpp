# libuv_cpp11
<a href="https://github.com/wlgq2/libuv_cpp11/releases"><img src="https://img.shields.io/github/release/wlgq2/libuv_cpp11.svg" alt="Github release"></a>
[![Platform](https://img.shields.io/badge/platform-%20%20%20%20Linux,%20Windows-green.svg?style=flat)](https://github.com/wlgq2/libuv_cpp11)
[![License](https://img.shields.io/badge/license-%20%20MIT-yellow.svg?style=flat)](LICENSE)
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)


<br>對libuv的C++11風格的跨平臺封裝庫，用於線上項目。跑過壓測，很穩定，正確使用接口情況下，未發現core dump或內存泄漏。</br>
## 特性
** **
* C++11風格回調函數：非C語言函數回調，支持非靜態類成員函數及lambda。
* TCP相關類封裝：`TcpServer`、`TcpClient`、`TcpConnection`、`TcpAccept`。
* `Timer`及`TimerWheel`：定時器及時間復雜度為O(1)的心跳超時踢出機制。
* `Async`：異步機制封裝。相對於原生libuv async接口，優化了調用多次可能只運行壹次的問題。由於libuv幾乎所有api都非線程安全，建議使用writeInLoop接口代替直接write（writeInLoop會檢查當前調用的線程，如果在loop線程中調用則直接write，否則把write加到loop線程中執行）。
* libuv信號封裝。   
* `Packet`與`PacketBuffer`：包與緩存，發送/接受包，用於解決TCP殘包/粘包問題，由ListBuffer和CycleBuffer兩種實現，可通過宏配置（前者空間友好，後者時間友好）。
* Log日誌輸出接口，可綁定至自定義Log庫。
** **
簡單性能測試：單線程1k字節ping-pong。
<br>環境：Intel Core i5 6402 + ubuntu14.04.5 + gcc5.5.0 + libuv1.22.0 + O2優化</br>

   libuv_cpp | no use PacketBuffer|CycleBuffer|ListBuffer|
:---------:|:--------:|:--------:|:--------:|
次/秒     | 192857 |141487|12594|

## 第一個常式
```C++
#include <iostream>
#include <uv/uv11.h>


int main(int argc, char** args)
{
    //event's loop
    uv::EventLoop* loop = uv::EventLoop::DefalutLoop();

    //Tcp Server
    uv::SocketAddr serverAddr("0.0.0.0", 10002, uv::SocketAddr::Ipv4);
    uv::TcpServer server(loop, serverAddr);
    server.setMessageCallback(
        [](std::shared_ptr<uv::TcpConnection> conn, const char* data , ssize_t size)
    {
        std::cout << std::string(data, size) << std::endl;
        conn->write(data, size,nullptr);
    });
    server.start();

    //Tcp Client
    uv::TcpClient client(loop);
    client.setConnectCallback(
        [&client](bool isSuccess)
    {
        if (isSuccess)
        {
            char data[] = "hello world!";
            client.write(data, sizeof(data));
        }
    });
    client.connect(serverAddr);
       
    loop->run();
}

```
** **
<br>**！對於諸如`uv::Timer`,`uv::TcpClient`等對象的釋放需要調用close接口並在回調函數中釋放對象,否則可能會出錯。**</br>
<br>**！切勿在Loop線程外創建註冊該Loop下的事件相關對象（`uv::TcpClient`，`uv::TcpServer`，`uv::Timer`……），建議每個Loop都綁定獨立線程運行。**</br>
<br>壹點微小的工作。</br>