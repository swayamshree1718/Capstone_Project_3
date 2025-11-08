# Capstone_Project_3
A simple C++ client-server file sharing application using TCP sockets. Supports authentication, file upload (PUT), and file download (GET) between client and server.
# 🔄 C++ File Sharing System

A simple **client-server file sharing application** written in **C++** using **TCP sockets**.  
This project demonstrates basic socket programming, authentication, and file transfer between two systems.

---

## 🚀 Features
- 🔐 **User Authentication** (username/password)
- 📤 **Upload files** from client to server (PUT)
- 📥 **Download files** from server to client (GET)
- 🗂️ Files stored in a dedicated `files/` directory
- ⚙️ Works on **Linux / macOS**

---

## 🧱 Technologies Used
- **C++**
- **POSIX sockets (sys/socket.h, netinet/in.h, arpa/inet.h)**
- **File I/O streams (fstream)**

---

## 📦 Project Structure
03:file sharing project-
SWAYAMSHREE SINGH_2241019524_BATCH 02
server .cpp
#include <iostream>
#include <fstream>
#include <string>
#include <cstring>
#include <sys/socket.h>
#include <ne􀆟net/in.h>
#include <unistd.h>
#define PORT 8080
#define BUFFER_SIZE 1024
bool authen􀆟cate(int clientSocket) {
std::string username, password;
char buffer[BUFFER_SIZE] = {0};
read(clientSocket, buffer, BUFFER_SIZE);
username = buffer;
memset(buffer, 0, BUFFER_SIZE);
read(clientSocket, buffer, BUFFER_SIZE);
password = buffer;
std::cout << "Login a􀆩empt: " << username << std::endl;
if (username == "admin" && password == "1234") {
std::string ok = "AUTH_SUCCESS";
send(clientSocket, ok.c_str(), ok.size(), 0);
return true;
} else {
std::string fail = "AUTH_FAIL";
send(clientSocket, fail.c_str(), fail.size(), 0);
return false;
}
}
void sendFile(int clientSocket, const std::string &filePath) {
std::ifstream file(filePath, std::ios::binary);
if (!file.is_open()) {
std::string msg = "FILE_NOT_FOUND";
send(clientSocket, msg.c_str(), msg.size(), 0);
return;
}
std::string msg = "FILE_FOUND";
send(clientSocket, msg.c_str(), msg.size(), 0);
char buffer[BUFFER_SIZE];
while (!file.eof()) {
file.read(buffer, BUFFER_SIZE);
send(clientSocket, buffer, file.gcount(), 0);
}
file.close();
std::cout << "􈆆􎀱 File sent successfully.\n";
}
void receiveFile(int clientSocket, const std::string &filePath) {
char buffer[BUFFER_SIZE];
std::ofstream file(filePath, std::ios::binary);
ssize_t bytesRead;
while ((bytesRead = recv(clientSocket, buffer, BUFFER_SIZE, 0)) > 0) {
file.write(buffer, bytesRead);
if (bytesRead < BUFFER_SIZE) break; // end
}
file.close();
std::cout << "􈆆􎀱 File received and saved to " << filePath << "\n";
}
int main() {
int serverSocket, clientSocket;
sockaddr_in address;
int addrlen = sizeof(address);
serverSocket = socket(AF_INET, SOCK_STREAM, 0);
address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;
address.sin_port = htons(PORT);
bind(serverSocket, (struct sockaddr*)&address, sizeof(address));
listen(serverSocket, 3);
std::cout << "􊸡􊸢􊸣􊸤􊸥􊸦􊸧􊸪􊸨􊸩 Server running on port " << PORT << "...\n";
clientSocket = accept(serverSocket, (struct sockaddr*)&address, (socklen_t*)&addrlen);
std::cout << "Client connected.\n";
if (!authen􀆟cate(clientSocket)) {
std::cout << "􎈐 Authen􀆟ca􀆟on failed.\n";
close(clientSocket);
close(serverSocket);
return 0;
}
char command[BUFFER_SIZE] = {0};
read(clientSocket, command, BUFFER_SIZE);
if (strncmp(command, "GET", 3) == 0) {
std::string filename = command + 4;
sendFile(clientSocket, "files/" + filename);
} else if (strncmp(command, "PUT", 3) == 0) {
std::string filename = command + 4;
receiveFile(clientSocket, "files/" + filename);
}
close(clientSocket);
close(serverSocket);
return 0;
}
client.cpp
#include <iostream>
#include <fstream>
#include <string>
#include <cstring>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#define PORT 8080
#define BUFFER_SIZE 1024
void sendFile(int sock, const std::string &filePath) {
std::ifstream file(filePath, std::ios::binary);
if (!file.is_open()) {
std::cout << "􎈐 Cannot open file.\n";
return;
}
char buffer[BUFFER_SIZE];
while (!file.eof()) {
file.read(buffer, BUFFER_SIZE);
send(sock, buffer, file.gcount(), 0);
}
file.close();
std::cout << "􈆆􎀱 File uploaded successfully.\n";
}
void receiveFile(int sock, const std::string &filePath) {
char buffer[BUFFER_SIZE];
std::ofstream file(filePath, std::ios::binary);
ssize_t bytesRead;
while ((bytesRead = recv(sock, buffer, BUFFER_SIZE, 0)) > 0) {
file.write(buffer, bytesRead);
if (bytesRead < BUFFER_SIZE) break;
}
file.close();
std::cout << "􈆆􎀱 File downloaded successfully.\n";
}
int main() {
int sock = socket(AF_INET, SOCK_STREAM, 0);
sockaddr_in serv_addr;
serv_addr.sin_family = AF_INET;
serv_addr.sin_port = htons(PORT);
inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr);
if (connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
std::cout << "􎈐 Connec􀆟on failed.\n";
return -1;
}
// Authen􀆟ca􀆟on
std::string username, password;
std::cout << "Username: ";
std::cin >> username;
std::cout << "Password: ";
std::cin >> password;
send(sock, username.c_str(), username.size(), 0);
send(sock, password.c_str(), password.size(), 0);
char authRes[BUFFER_SIZE] = {0};
read(sock, authRes, BUFFER_SIZE);
if (strcmp(authRes, "AUTH_FAIL") == 0) {
std::cout << "􎈐 Authen􀆟ca􀆟on failed.\n";
close(sock);
return 0;
}
std::cout << "􈆆􎀱 Authen􀆟cated successfully.\n";
std::cout << "Enter command (GET filename | PUT filename): ";
std::string command;
std::getline(std::cin >> std::ws, command);
send(sock, command.c_str(), command.size(), 0);
if (command.rfind("GET", 0) == 0) {
std::string filename = command.substr(4);
receiveFile(sock, "downloaded_" + filename);
} else if (command.rfind("PUT", 0) == 0) {
std::string filename = command.substr(4);
sendFile(sock, filename);
}
close(sock);
return 0;
}



run commands:
g++ server.cpp -o server
g++ client.cpp -o client
mkdir files
./server
# in another terminal
./client
