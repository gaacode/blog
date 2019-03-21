---
layout: post
title: "AJAX Introduction"
description: "AJAX là viết tắt của Asynchronous Javascript and XML. Đây là một kỹ thuật trong Javascript, được dùng để cập nhật lại dữ liệu, giao diện một phần trên trang web mà không cần tải lại toàn bộ trang..."
comments: true
keywords: "JavaScript, Ajax"
---

## Giới thiệu

AJAX là viết tắt của Asynchronous Javascript and XML. Đây là một kỹ thuật trong Javascript, được dùng để cập nhật lại dữ liệu, giao diện một phần trên trang web mà không cần tải lại toàn bộ trang.   

Đây là sự kết hợp của:

* Một object XMLHttpRequest được xây dựng trong trình duyệt (để request data từ web server).
* Javascript và HTML DOM (để hiển thị data hoặc lấy data để gửi lên server)

**AJAX có thể sử dụng XML, JSON hoặc văn bản thuần túy để vận chuyển dữ liệu.**

Nó cho phép các trang web cập nhật bất đồng bộ bằng cách trao đổi data với web server đằng sau hậu trường. Nghĩa là nó có thể load lại và thay đổi 1 phần của trang web mà không phải tải lại toàn bộ trang.

## AJAX hoạt động như thế nào?

![](https://images.viblo.asia/83fc6eef-17b8-43e8-a47d-45d6831d323b.gif)

1. Một sự kiện xảy ra trong web page (khi web đã load xong, và một button được nhấn)
2. Một XMLHttpRequest object được tạo bởi Javascript
3. XMLHttpRequest object gửi một request đến web server
4. Server xử lý request (yêu cầu)
5. Server gửi response (phản hồi) về trang web
6. Response được đọc và xử lý bởi Javascript
7. Update nội dung trang.

Bài viết này đến đây là hết, cảm ơn mọi người đã theo dõi! 

## Tài liệu tham khảo

https://vuongphan139.github.io/ajax-intro/
https://www.w3schools.com/js/js_ajax_intro.asp
