---
title: "Android"
date: 2022-07-06T22:26:42+08:00
---

## 安卓新建项目
### Server 
```cpp
#include <iostream>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <arpa/inet.h>
#include <pthread.h>

constexpr int BUFFER_SIZE = 1024;
constexpr int PORT = 8080;

void *handleClient(void *arg) {
    int clientSocket = *((int *)arg);

    char buffer[BUFFER_SIZE];
    while (true) {
        // 接收客户端消息
        ssize_t bytesRead = recv(clientSocket, buffer, BUFFER_SIZE - 1, 0);
        if (bytesRead <= 0) {
            // 客户端关闭连接或发生错误
            close(clientSocket);
            std::cout << "Client disconnected" << std::endl;
            break;
        }

        buffer[bytesRead] = '\0';
        std::cout << "Received from client: " << buffer << std::endl;

        // 原样返回消息给客户端
        send(clientSocket, buffer, strlen(buffer), 0);
    }

    pthread_exit(NULL);
}

int main() {
    // 创建服务器套接字
    int serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket == -1) {
        std::cerr << "Error creating server socket" << std::endl;
        exit(EXIT_FAILURE);
    }

    // 配置服务器地址
    sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(PORT);

    // 绑定套接字
    if (bind(serverSocket, (struct sockaddr *)&serverAddr, sizeof(serverAddr)) == -1) {
        std::cerr << "Error binding server socket" << std::endl;
        close(serverSocket);
        exit(EXIT_FAILURE);
    }

    // 监听连接
    if (listen(serverSocket, 10) == -1) {
        std::cerr << "Error listening for connections" << std::endl;
        close(serverSocket);
        exit(EXIT_FAILURE);
    }

    std::cout << "Server listening on port " << PORT << std::endl;

    while (true) {
        // 接受客户端连接
        sockaddr_in clientAddr;
        socklen_t clientAddrLen = sizeof(clientAddr);
        int clientSocket = accept(serverSocket, (struct sockaddr *)&clientAddr, &clientAddrLen);
        if (clientSocket == -1) {
            std::cerr << "Error accepting client connection" << std::endl;
            continue;
        }

        std::cout << "Client connected: " << inet_ntoa(clientAddr.sin_addr) << ":" << ntohs(clientAddr.sin_port) << std::endl;

        // 创建一个新线程处理客户端请求
        pthread_t thread;
        if (pthread_create(&thread, NULL, handleClient, &clientSocket) != 0) {
            std::cerr << "Error creating thread" << std::endl;
            close(clientSocket);
        } else {
            pthread_detach(thread);
        }
    }

    // 关闭服务器套接字
    close(serverSocket);

    return 0;
}

```


### Client 

```cpp
#include <bits/stdc++.h>

#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <arpa/inet.h>

using namespace std;
constexpr int BUFFER_SIZE = 1024;
constexpr int PORT = 8080;
const char *SERVER_IP = "127.0.0.1";  // 服务器的IP地址，这里使用本地回环地址

struct Socket {
    int sock;

    Socket() {
        sock = socket(AF_INET, SOCK_STREAM, 0);
    }

    template<class T>
    auto write(const T &buf) {
        return ::send(sock, buf.data(), (buf).size(), 0);
    }

    template<class T>
    auto read(T &buf) {
        return recv(sock, buf.data(), buf.size(), 0);
    }
};


Socket *newSocket() {
    return new Socket();
}


int main() {
    // 创建客户端套接字
    int clientSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientSocket == -1) {
        std::cerr << "Error creating client socket" << std::endl;
        exit(EXIT_FAILURE);
    }

    // 配置服务器地址
    sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = inet_addr(SERVER_IP);
    serverAddr.sin_port = htons(PORT);

    // 连接到服务器
    if (connect(clientSocket, (struct sockaddr *) &serverAddr, sizeof(serverAddr)) == -1) {
        std::cerr << "Error connecting to server" << std::endl;
        close(clientSocket);
        exit(EXIT_FAILURE);
    }

    std::cout << "Connected to server" << std::endl;
    char buffer[BUFFER_SIZE];
    for (;;) {

        // 发送消息到服务器

        //        const char *message = "Hello, server!";
        string message;
        cin >> message;
        send(clientSocket, message.c_str(), (message).size(), 0);
        std::cout << "Message sent to server: " << message << std::endl;

        // 接收服务器的响应

        ssize_t bytesRead = recv(clientSocket, buffer, BUFFER_SIZE - 1, 0);
        if (bytesRead <= 0) {
            std::cerr << "Error receiving response from server" << std::endl;
            break;
        } else {
            buffer[bytesRead] = '\0';
            std::cout << "Received from server: " << buffer << std::endl;
        }
    }

    // 关闭客户端套接字
    close(clientSocket);

    return 0;
}
```