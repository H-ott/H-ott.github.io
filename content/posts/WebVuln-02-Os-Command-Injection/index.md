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
    - ```ping google.com && sleep 5```
    - ```google.com || sleep 5```
    - ```google.com; `sleep 5'```
    - ```google.com=$(sleep 5)```
  - Nhiều cách khác - [Tham khảo](https://book.hacktricks.xyz/pentesting-web/command-injection)
- Chú ý là khi chèn lệnh cần đúng với cú pháp của câu lệnh lập trình viên viết và lệnh hệ thống để lệnh có thể chạy được, ví dụ thiếu dấu ```'``` thì phải thêm ```'```, thừa thì thêm dấu commment ```#```.
## 3. Cách khai thác OS command injection
  - Khi một chức năng bị lỗ hổng này có thể khai thác bằng nhiều cách và lấy được rất nhiều thông tin vì đây là lỗ hổng vô cùng nguy hiểm hacker có thể chiếm quyền toàn bộ ứng dụng của bạn
  - Ví dụ:
    - Khi có thể vào được tài khoản admin thì sau khi chèn được lệnh hệ thống bạn có thể xoá cả sever, phân quyền admin cho bất cứ người dùng nào,...
    - Ví dụ ở chức năng ping của 1 trang web được lập trình viên viết bằng php như sau:
     ```php
     $host = $_GET['host'];
     $result = system("ping {$host}");
     echo ($result);
     ```
    Cho phép người dùng nhập giá trị host nếu giá trị host không được validate kỹ thì có thể nhập như sau ```"8.8.8.8" | cat /etc/passwd"``` để được:
      ```php
      $host = $_GET['host'];
      $result = system("ping "8.8.8.8" | cat /etc/passwd");
      echo ($result);
      ```
    là có thể ngay sau lệnh ping thì sẽ hiển thị file passwd trong hệ thống.
    - Bypass 4 ký tự, 5 ký tự.
    - ```'l's, 'i'd,...```
