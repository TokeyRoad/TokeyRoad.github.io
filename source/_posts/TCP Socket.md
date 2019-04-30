---
title: TCP Socket
date: 2019-04-21 23:37:00
categories: 
- 网络协议
copyright: ture
---
一个简单的TCP socket通讯的实例。

功能简单说明：
服务端主线程一直阻塞在accept，每连接上一个客户端，就创建一个新的线程去处理这个客户端的消息。
其中新线程中一直处于接收客户端的消息中，每接收一次都回复一个keep alive，当收到来自客户端stop消息时，就断开连接，并关闭当前线程。

**一.服务端**

1. 创建套接字
		WORD sockVersion = MAKEWORD(2, 2);
		WSADATA wsaData;
		if (WSAStartup(sockVersion, &wsaData) != 0) {
			return 0;
		}
		//WSA(Windows Sockets Asynchronous，Windows异步套接字)的启动命令
2. 绑定套接字到IP和端口
		//创建并初始化socket
		SOCKET listen_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
		if (listen_socket == INVALID_SOCKET) {
			printf("socket init error code:%d", WSAGetLastError());
			return 0;
		}
		//绑定结构
		struct sockaddr_in sin;
		sin.sin_family = AF_INET;
		sin.sin_port = htons(6666);
		sin.sin_addr.S_un.S_addr = INADDR_ANY;
		//绑定
		if (bind(listen_socket, (LPSOCKADDR)&sin, sizeof(sin)) != 0) {
			printf("bind socket error code:%d", WSAGetLastError());
			closesocket(listen_socket);//注意socket回收
			return 0;
		}
3. 监听端口
		//int listen( int sockfd, int backlog);
		//sockfd：用于标识一个已捆绑未连接套接口的描述字。
		//backlog：等待连接队列的最大长度。
		if (listen(listen_socket, 5) != 0) {
			printf("listen socket error code:%d", WSAGetLastError());
			closesocket(listen_socket);
			return 0;
		}
4. 等待客户端连接，每连接一个可以创建一个对应的套接字和线程去处理对应的消息
		//等待客户端连接 线程在这里会阻塞
		sClient = accept(listen_socket, (LPSOCKADDR)&remoteAddr, &slen);

		if (sClient == INVALID_SOCKET) {
			printf("accept client error code:%d", WSAGetLastError());
			continue;
		}
		else {
			printf("connect success\n");
			DWORD handleID; 
			//当连接成功时，创建一个线程处理该客户端的消息，然后继续等待下一个客户端的连接
			HANDLE handleSecond = CreateThread(NULL, 0, ClientChat, (LPVOID)sClient, 0, &handleID);
		}
5. 处理完之后需要回收线程和回收socket
		//回收线程
		CloseHandle(handle);
		//回收socket
		closesocket(listen_socket);
		//WSA套接字清理
		WSACleanup();

**二.客户端**

1. 创建套接字，初始化等。（和服务端一致）

2. 连接服务端
		sockaddr_in sin;
		sin.sin_family = AF_INET;
		sin.sin_port = htons(6666);
		//sin.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
		inet_pton(AF_INET, "192.168.0.190", (void*)&sin.sin_addr.S_un.S_addr);
		// 与指定地址端口连接
		if (connect(clientSocket, (LPSOCKADDR)&sin, sizeof(sin)) == SOCKET_ERROR) {
			printf("connect error code:%d", WSAGetLastError());
			closesocket(clientSocket);
			return  0;
		}
		else {
			printf("connect success\n");
		}
3. 和服务端之间收发消息
		char rcv[256] = "";
		std::cout << "please input:";
		char sendData[128];
		while (std::cin >> sendData) {
			// 向服务端发数据
			int res = send(clientSocket, sendData, strlen(sendData), 0);
			if (res == SOCKET_ERROR) {
				printf("client send data error code:%d\n", WSAGetLastError());
			}
			else {
				printf("client send data success\n");
			}
			//检测到发送的数据是stop时 断开连接
			if (strcmp(sendData, "stop") == 0) {
				break;
			}
			//接收数据之前先清空缓冲区，否则会有前一次传输的脏数据
			memset(rcv, 0, sizeof(rcv));
			int rec_res = recv(clientSocket, rcv, sizeof(rcv), 0);
			if (rec_res > 0) {
				printf("receive server bytes:%d, data:%s\n", res, rcv);
				if (strcmp(rcv, "stop") == 0) {
					break;
				}
			}
			else if (rec_res == 0) {
				printf("socket has closed\n");
			}
			else {
				printf("recv failed: %d\n", WSAGetLastError());
			}
			std::cout << "please input:";
		}
4. socket回收 同服务端


源代码：
服务端：

		//Server.cpp
		#include <stdio.h>
		#include <WinSock2.h>
		#include <iostream>
		#include <process.h>
		
		#pragma comment( lib, "ws2_32.lib")
		
		#define MAX_THREAD 10
		struct ThreadMessage{
			SOCKET socket;
			DWORD id;
		};
		
		DWORD WINAPI ClientChat(LPVOID param);
		
		int main() {
			WORD sockVersion = MAKEWORD(2, 2);
			WSADATA wsaData;
		
			if (WSAStartup(sockVersion, &wsaData) != 0) {
				return 0;
			}
			SOCKET listen_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
			if (listen_socket == INVALID_SOCKET) {
				printf("socket init error code:%d", WSAGetLastError());
				return 0;
			}
		
			struct sockaddr_in sin;
			sin.sin_family = AF_INET;
			sin.sin_port = htons(6666);
			sin.sin_addr.S_un.S_addr = INADDR_ANY;
		
			if (bind(listen_socket, (LPSOCKADDR)&sin, sizeof(sin)) != 0) {
				printf("bind socket error code:%d", WSAGetLastError());
				closesocket(listen_socket);
				return 0;
			}
		
			if (listen(listen_socket, 5) != 0) {
				printf("listen socket error code:%d", WSAGetLastError());
				closesocket(listen_socket);
				return 0;
			}
		
			sockaddr_in remoteAddr;
			SOCKET sClient;
			int slen = sizeof(remoteAddr);
			char rcv[512];
			ThreadMessage thread[MAX_THREAD];
			int i = 0;
			while (true) {
				printf("server is ready, waiting for client ...");
				sClient = accept(listen_socket, (LPSOCKADDR)&remoteAddr, &slen);
		
				if (sClient == INVALID_SOCKET) {
					printf("accept client error code:%d", WSAGetLastError());
					continue;
				}
				else {
					printf("connect success\n");
					DWORD handleID; 
					thread[i].id = i;
					thread[i].socket = sClient;
					HANDLE handleSecond = CreateThread(NULL, 0, ClientChat, &thread[i], 0, &handleID);
					i++;
				}
				
			}
		
			closesocket(listen_socket);
			WSACleanup();
		
			return 0;
		}
		
		DWORD WINAPI ClientChat(LPVOID param) {
			ThreadMessage* msg = (ThreadMessage*)param;
			HANDLE handle = GetCurrentThread();
			if (msg->socket == INVALID_SOCKET) {
				printf("thread change error code:%d", WSAGetLastError());
				CloseHandle(handle);
				return 0;
			}
			char rcv[256] = "";
			while (1) {
				char sendData[256] = "keep alive"; 
				memset(rcv, 0, strlen(rcv));
				int res = recv(msg->socket, rcv, sizeof(rcv), 0);
				if (res > 0) {
					printf("tag:%d, receive client bytes:%d, data:%s\n", msg->id, res, rcv);
					if (strcmp(rcv, "stop") == 0) {
						memset(sendData, 0, strlen(sendData));
						sprintf_s(sendData, "stop");
						send(msg->socket, sendData, strlen(sendData), 0);
						break;
					}
					send(msg->socket, sendData, strlen(sendData), 0);
				}
				else if (res == 0) {
					printf("socket has closed\n");
				}
			}
			printf("close thread! \n");
			CloseHandle(handle);
		
		}
客户端：

		//Client.cpp
		#include <stdio.h>
		#include <WinSock2.h>
		#include<WS2tcpip.h>
		#include <iostream>
		#pragma comment(lib, "ws2_32.lib")
		
		int main() {
			WORD wsaVer = MAKEWORD(2, 2);
			WSADATA wsaData;
		
			if (WSAStartup(wsaVer, &wsaData) != 0) {
				return 0;
			}
		
			SOCKET clientSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
			if (clientSocket == INVALID_SOCKET) {
				printf("clientSocket error code:%d", WSAGetLastError());
				return 0;
			}
		
			sockaddr_in sin;
			sin.sin_family = AF_INET;
			sin.sin_port = htons(6666);
			//sin.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
			inet_pton(AF_INET, "192.168.0.190", (void*)&sin.sin_addr.S_un.S_addr);
		
			if (connect(clientSocket, (LPSOCKADDR)&sin, sizeof(sin)) == SOCKET_ERROR) {
				printf("connect error code:%d", WSAGetLastError());
				closesocket(clientSocket);
				return  0;
			}
			else {
				printf("connect success\n");
			}
		
			char * sData = "Hello Server!";
			int res = send(clientSocket, sData, strlen(sData), 0);
			if (res == SOCKET_ERROR) {
				printf("client send data error code:%d\n", WSAGetLastError());
			}
			else {
				printf("client send data success\n");
			}
			char rcv[256] = "";
			std::cout << "please input:";
			char sendData[128];
			while (std::cin >> sendData) {
				int res = send(clientSocket, sendData, strlen(sendData), 0);
				if (res == SOCKET_ERROR) {
					printf("client send data error code:%d\n", WSAGetLastError());
				}
				else {
					printf("client send data success\n");
				}
				if (strcmp(sendData, "stop") == 0) {
					break;
				}
				memset(rcv, 0, sizeof(rcv));
				int rec_res = recv(clientSocket, rcv, sizeof(rcv), 0);
				if (rec_res > 0) {
					printf("receive server bytes:%d, data:%s\n", res, rcv);
					if (strcmp(rcv, "stop") == 0) {
						break;
					}
				}
				else if (rec_res == 0) {
					printf("socket has closed\n");
				}
				else {
					printf("recv failed: %d\n", WSAGetLastError());
				}
				std::cout << "please input:";
			}
			closesocket(clientSocket);
			std::cout << "socket close!\n";
			system("pause");
			return 0;
		}