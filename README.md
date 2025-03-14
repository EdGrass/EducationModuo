# EduMuduo 网络库

---

> 本项目的初衷是学习 C++ 面向对象编程的特性和 Muduo 库的实现方式。请勿用于生产环境。

EduMuduo 是一个基于 C++17 的高性能事件驱动网络库，参考了 muduo 库设计理念，提供了简洁易用的 TCP 网络编程接口。采用 Reactor 模式，使用 epoll 作为 I/O 多路复用机制，适用于 Linux 平台下的高并发服务器开发。

---

## 📜 特性

- **非阻塞 I/O**：基于 epoll 的事件驱动模型
- **多线程支持**：one loop per thread 线程模型
- **现代 C++**：使用 C++17 特性实现高效资源管理，安装简单
- **完整日志**：集成 fmt 库的现代化日志系统
- **简洁 API**：直观易用的类 muduo 接口设计

---

## ⚙️ 核心组件

每个`hpp`都声明了一个类（除了Callbacks、CurrentThread、Default），我在每个类前都简单写（ENG）了这个类的主要作用和核心函数/成员。       

---

## 🚀 安装

### 1. 克隆仓库

```sh
git clone https://github.com/EdGrass/EduMuduo.git
cd EduMuduo
```

### 2. 构建项目

```sh
mkdir build && cd build
cmake ..
make 
```

### 3. 安装依赖

```sh
sudo apt install libfmt-dev  # Ubuntu
brew install fmt             # macOS
```

---

## 🧑‍💻 使用示例

```cpp
#include <string>
#include <muduo/Logger.hpp>
#include <muduo/TcpServer.hpp>
class ExampleServer {
public:
    ExampleServer(EventLoop *loop, const InetAddress &addr, const std::string &name)
        : server_(loop, addr, name)
        , loop_(loop)
    {
        server_.setConnectionCallback(
            std::bind(&ExampleServer::onConnection, this, std::placeholders::_1));
        
        server_.setMessageCallback(
            std::bind(&ExampleServer::onMessage, this, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3));

        server_.setThreadNum(3);
    }
    void start()
    {
        server_.start();
    }
private:
    void onConnection(const TcpConnectionPtr &conn) {
		conn->connected();
    }
    void onMessage(const TcpConnectionPtr &conn, Buffer *buf, Timestamp time){
		std::string msg = buf->retrieveAllAsString();
        conn->send(msg);
		conn->shutdown();
    }
    EventLoop *loop_;
    TcpServer server_;
};
int main() {
    EventLoop loop;
    InetAddress addr(1145);
    ExampleServer server(&loop, addr, "ExampleServer-01");
    server.start();
    loop.loop();
    return 0;
}
```

---

## 📄 许可证

本项目采用 MIT 许可证。查看 [LICENSE](LICENSE) 文件了解更多详细信息。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 致谢

- 灵感来源于陈硕的《Linux多线程服务端编程》
- 感谢 muduo 库的开源贡献者
