---
title: "OS Command Injection"
date: 2024-12-17
draft: false
tags: ["web", "vuln", "pentest"]
categories: ["web Vulnerability"]
lightgallery: true
toc:
  enable: true
description: "OS Command Injection"
---
# Phân tích lỗ hổng OS Command Injection
  OS Commmand Injection - Là lỗ hổng thuộc loại [**Technical vulnerability**](https://tlualgosec.com/posts/Blog101/#2-ph%C3%A2n-loa%CC%A3i-l%C3%B4%CC%83-h%C3%B4%CC%89ng).
## 1. OS commmand injection là gì?
- OS command injection là lỗ hổng cho phép kẻ tấn công có thể chèn và thực thi **lệnh hệ thống** trên các chức năng, ứng dụng của hệ thống,...
- OS command injection có thể gọi là Shell Injecion.
## 2. Phát hiện OS command injection như thế nào?
- Trước hết phải xác định xem sever đấy chạy hệ điều hành gì: Windows, Linux,...
- Nếu là Whitebox thì cần tìm và ghi ra tất cả các chức năng, hàm liên quan đến câu lệnh hệ thống như inlude_one, require_one, system, exec,... Mà ở những chỗ này cho phép nhập input.
- Nếu là Blackbox thì cần test tất cả các chức năng có thể tương tác, làm việc với lệnh hệ thống như ping, curl,...
  - Ý tưởng để test Blackbox là có thể chạy song song nhiều lệnh hệ thống trên 1 dòng lệnh.
  - Ví dụ:
    - ```ping 8.8.8.8 | cat /etc/passwd``` Nếu có thể chạy được -> Lỗi.
    - ```ping 8.8.8.8 | sleep 5``` Nếu chạy xong lệnh ping mà sever chờ 5s -> Lỗi.
  - Hay bất cứ đâu có thể nhập một lệnh hệ thống - [Tham khảo](https://book.hacktricks.xyz/pentesting-web/command-injection)
- Chú ý là khi chèn lệnh cần đúng với cú pháp của câu lệnh lập trình viên viết và lệnh hệ thống để lệnh có thể chạy được, ví dụ thiếu dấu ```'``` thì phải thêm ```'```, thừa thì thêm dấu commment ```#```
