# Rust 实战: 启动多线程 Web 服务

@[TOC](文章目录)

<!-- TOC -->

- [Rust 实战: 启动多线程 Web 服务](#rust-实战-启动多线程-web-服务)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 监听 TCP 连接请求](#1-监听-tcp-连接请求)
  - [2. 创建多线程运行环境](#2-创建多线程运行环境)
    - [2.1 线程池 ThreadPool](#21-线程池-threadpool)
    - [2.2 任务执行 Worker](#22-任务执行-worker)
  - [3. 运行效果](#3-运行效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 基本信息

Rust 语言内置提供了 `TcpListener` 的结构体，来实现监听 TCP 的网络连接，不过默认情况下程序都是同步并阻塞在 `incoming` 方法上的。

因此本篇除了介绍使用 `TcpListener` 监听 TCP 连接请求之外，再加上使用 Rust 异步编程的方式，实现一个极简易的 Http 服务器

## 1. 监听 TCP 连接请求

首先我们从 TCP 请求的监听开始看起

- `/src/main.rs`

首先我们使用 `TcpListener` 建立一个监听实例，监听本地上的 8080 端口

```rust
fn main() -> io::Result<()> {
    let addr = "127.0.0.1:8080";
    let listener = TcpListener::bind(addr)?;

    println!("TcpListener listen at {} ...", addr);
```

接下来我们可以调用 `incoming` 方法，会接收到一个 `Result<TcpStream>` 类型的 TCP 流，然后我们传入 `handle_client` 方法进行处理

```rust
    let mut stream_id = 0;

    for stream in listener.incoming() {
        stream_id += 1;
        let id = stream_id;
        let stream = stream.unwrap();

        handle_client(stream, id);
    }

    Ok(())
}
```

这里注意到主线程会阻塞在 `incoming` 方法上，所以同步的情况下第一个请求尚未完成处理的话，下一个请求是没办法进来的。

最后就是我们的 `handle_client` 处理函数，处理比较简陋，完全没有检查 Http 信息，只是简单响应 Http 请求罢了

```rust
fn handle_client(mut stream: TcpStream, id: i32) {
    println!("handle stream({}) ...\n", id);
    let mut buffer = [0u8; 512];
    stream.read(&mut buffer).unwrap();

    println!("stream({}) detail in String:", id);
    println!("{}\n", String::from_utf8_lossy(&buffer));
    println!("stream({}) detail in [u8]:", id);
    println!("{:?}\n", buffer);

    let get = b"GET / HTTP/1.1\r\n";
    let (status_line, content) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "Hello World")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "Not found")
    };

    let response = format!("{}{}", status_line, content);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();

    stream.shutdown(Shutdown::Both).unwrap();

    println!("stream({}) process success.\n", id);
}
```

网上好多教程都没写 `shutdown` 来关闭流，所以默认情况下开启 keep-alive 会造成一直没有响应

## 2. 创建多线程运行环境

前面实际上就已经完成 TCP 服务的创建了，不过问题在于单线程情况下会产生阻塞。下面我们使用 `mpsc` 包提供的 `channel` 函数来创建多线程通道

### 2.1 线程池 ThreadPool

我们直接上线程池的概念，野线程的问题就不再多提了

- `/src/pool.rs`

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}
```

一个线程池会保存一个工人队列(`Worker` 代表线程执行者)，以及一个任务分发函数。`Sender` 对象是 channel 通道返回的函数，用于接受任务并传入线程池进行调用

接下来则是线程池的方法实现

```rust
impl ThreadPool {
    pub fn new(size: usize) -> Self {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for i in 0..size {
            workers.push(Worker::new(i, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender,
        }
    }
```

对于构造函数，我们就是根据 `size` 参数决定创建多少个 Worker，并使用 `Arc<Mutext<Receiver>>` 来包裹 Receiver 对象，来保证他的线程安全

```rust
    pub fn execute<F>(&self, f: F) where F: FnOnce() + Send + 'static {
        self.sender.send(Box::new(f)).unwrap();
    }
}
```

第二个 `execute` 接受一个 `f` 参数，也就是一个匿名函数，相当于是外部提交一个任务交给线程池执行，`ThreadPool` 再通过调用 `sender.send` 方法来交给真正的 channel 进行任务分发

### 2.2 任务执行 Worker

前面提过线程池创建了对应线程池容量的工人队列，每个工人会维护一个线程持续执行

```rust
pub struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}
```

接下来我们会直接在构造函数里面启动一个线程，然后从 `receiver` 接受任务来执行

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                println!("Worker({}) is waiting ...", id);

                let job = receiver.lock().unwrap().recv().unwrap();

                println!("Worker({}) took the stream.", id);

                job();

                println!("Worker({}) finished.", id);
            }
        });

        Worker {
            id,
            thread,
        }
    }
}

type Job = Box<dyn FnOnce() + Send + 'static>;
```

我们可以看到由于 `receiver` 使用 `Arc<Mutex<mpsc::Receiver<Job>>>` 类型，因此我们可以使用 `lock` 方法来保证线程安全

同时当我们没有传入任务的时候，它会阻塞在 `recv` 方法上，直到等到下一个任务的传入才会继续执行

- `/src/main.rs`

最后我们只要稍稍的将 TCP 服务的处理函数包装成一个任务(匿名函数)，并传入线程池进行处理就可以啦

```rust
fn main() -> io::Result<()> {
    // ...
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        // ...

        pool.execute(move || {
            handle_client(stream, id);
        });
    }

    Ok(())
}
```

## 3. 运行效果

到此就完成啦，秀个请求成功的截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_multithread_http_server_1_console.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_multithread_http_server_2_request.png)

# 其他资源

## 参考连接

| Title                                     | Link                                                                                                                                       |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Struct std::net::TcpStream - Rust doc     | [https://doc.rust-lang.org/std/net/struct.TcpStream.html](https://doc.rust-lang.org/std/net/struct.TcpStream.html)                         |
| Module std::sync::mpsc - Rust doc         | [https://doc.rust-lang.org/std/sync/mpsc/](https://doc.rust-lang.org/std/sync/mpsc/)                                                       |
| Struct std::thread::JoinHandle - Rust doc | [https://doc.rust-lang.org/std/thread/struct.JoinHandle.html](https://doc.rust-lang.org/std/thread/struct.JoinHandle.html)                 |
| 用Rust实现一个多线程的web server          | [https://blog.csdn.net/lcloveyou/article/details/105214332](https://blog.csdn.net/lcloveyou/article/details/105214332)                     |
| HTTP请求报文和HTTP响应报文                | [https://www.cnblogs.com/biyeymyhjob/archive/2012/07/28/2612910.html](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/28/2612910.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_multithread_http_server](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_multithread_http_server)
