---
layout: post
title: "[Rails Action Cable #1] WebSockets và Action Cable"
description: "Giao thức WebSocket là phần bổ sung cho giao thức HTTP tiêu chuẩn dùng để tạo ra kết nối liên tục giữa servers và clients, cho phép giao tiếp 2 chiều giữa chúng. Vì vậy, các developers có thể dùng WebSocket để tạo ra các ứng dụng real-time như chat app hoặc game..."
comments: true
keywords: "rails, websocket, actioncable"
---

## Giới thiệu

Giao thức WebSocket là phần bổ sung cho giao thức HTTP tiêu chuẩn dùng để tạo ra kết nối liên tục giữa servers và clients, cho phép giao tiếp 2 chiều giữa chúng. 

Vì vậy, các developers có thể dùng WebSocket để tạo ra các ứng dụng real-time như chat app hoặc game server, những ứng dụng có nhiều tương tác theo thời gian thực. 

## Các phương thức truyền tin

**Bán song công** là khi một thiết bị chỉ có thể thu hoặc phát tại 1 thời điểm nhất định (không thể vừa thu vừa phát). 
Ví dụ, trong giao thức HTTP giữa server và client. Server chỉ đóng vai trò nhận dữ liệu hoặc gửi dữ liệu trong một thời điểm nhất định, client cũng thế.

![](https://images.viblo.asia/fc4c1f08-75b7-4be6-9bd6-1cff16011521.png)

Ngược lại với bán song công là **song công**, đó là khi một thiết bị có thể vừa thu, vừa phát trong cùng 1 thời điểm. WebSocket cho phép giao tiếp song công giữa client và server (sau khi gửi 1 request sử dụng header có tên là `Upgrade`)

![](https://images.viblo.asia/37f5a296-90b5-46b8-8449-669de8137c84.png)
