---
title: "SQL Injection"
date: 2024-12-10
draft: false
tags: ["web", "vuln", "pentest"]
categories: ["web Vulnerability"]
lightgallery: true
toc:
  enable: true
description: "SQL Injection"
---
# Mọi thứ có thể về SQL Injection: 
  Khái niệm SQLi ->SQLi có thể xuất hiện ở đâu -> Phát hiện SQLi như thế nào – Phân loại SQLi -> Cách khai thác SQLi như thế nào -> Mức độ ảnh hưởng đến tam giác CIA -> Khai thác được những gì -> Các trường hợp đặc biệt của SQLi -> Tools<br>
## 1. Khái niệm SQLi
- SQLi là một kỹ thuật tấn công cho phép kẻ tấn công chèn một untrusted data vào những chức làm việc với các truy vấn SQL cho phép tương tác với database trong ứng dụng như các form đăng nhập, đăng ký, hay url có chứa id của các bài post -> nhằm biến những thứ bất thường thành bình thường
- SQLi sảy ra chủ yếu là do người lập trình viên không validate dữ liệu đầu vào, để người dùng có thể nhập tuỳ ý
## 2. Phát hiện SQLi như nào
- Có nhiều cách, dấu hiệu để có thể kết luận là một chức năng hay một trang web bị dính SQLi. Ở các function liên quan đến truy vấn và database có thể test bằng những cách như sau: 
  - Thêm ký tự “ ‘ ” vào username, password, id, userid,… nếu lỗi không mong muốn sảy ra -> Lỗi SQLi
  - Thêm chuỗi “ ‘OR 1=1 LIMIT 1, 1-- - ” vào sau password nếu có thể truy cập được tài khoản một ai đó -> Lỗi SQLi
  - Dùng tính năng scan lỗ hổng SQLi trong Burpsuite
  - Dùng tools: sqlmap,…
  - Và còn rất nhiều cách để phát hiện SQLi – Tham khảo: https://book.hacktricks.xyz/pentesting-web/sql-injection
## 3. Các dạng SQLi 
-	Dạng đầu tiên của SQLi là In-band SQLi – Classic SQLi: Đây là 1 kiểu tấn công mà khi chèn SQL kẻ tấn công có thể thấy được và nhận được kết quả trực tiếp ngay trên chính giao diện trang web đó. Ví dụ như bạn tấn công vào form đăng nhập thì kết quả là bạn sẽ thấy và vào được tài khoản người dùng.
-	Một dạng khác là Blind SQLi: Dạng này là dạng mà khi chèn SQL thì sever chỉ phản hồi về cho kẻ tấn công 2 trạng thái khác nhau từ đó hacker có thể dựa vào nó để đoán tên database, table, column hay thậm chí là cả data trong hệ thống.
-	Out-of-band SQLi – Kiểu tấn công này ít phổ biến hơn 2 kiểu trên vì nó còn phụ thuộc vào yếu tố khác
## 4. Cách khai thác với các dạng
  ### In-band (classic) SQLi: Kiểu này khá đơn giản vì kẻ tấn công có thể trực tiếp quan sát được kết quả từ đó có thể tư duy, logic để khai thác hiệu quả:
  - In-band: Đây là kiểu cơ bản đại diện cho kiểu tấn công này
    - “ ‘OR 1=1 LIMIT 1,1-- - “
    - “page.asp?id=1 or 1=1 -- -”
    - http://chall.tlualgosec.com:1337/post/1+1
    - Lấy tất cả các cột trong database hiện tại(My SQL): “SELECT TABLE_NAME FROM information_schema.tables WHERE table_schema=DATABASE()”
  - Error-based: là một dạng khác của In-band SQLi, kiểu tấn công này thì hacker sẽ tận dụng những thông báo lỗi khi chạy lệnh SQL từ phía sever hiển thị ra màn hình từ đó tận dụng khai thác. Cách khai thác lỗi này giống với việc khai thác In-band tuy nhiên hacker cần cố tình tạo ra 1 câu lệnh SQL sai với cú pháp hoặc logic của ngôn ngữ SQL nhằm tạo ra lỗi và hiển thị nó ra màn hình và từ lỗi hiển thị ra đó sẽ có thể chứa thông tin về database,…
    - Ví dụ: khi so sánh<br>
    
       | Bình thường->không lỗi    | Khác thường->lỗi |
       |---------------------------|------------------|
       | int = int                 | Varchar = int    |

    - Trong một ứng dụng web khi yêu cầu nhập id sản phẩm là một số nguyên kiểu int tuy nhiên hacker đã nhập hàm version() trong mysql mà hàm này trả vể 1 chuỗi kiểu string là phiên bản hiện tại của database vậy thông báo lỗi có thể là: “Cannot compare string to int!” 
  -	Union-based SQLi: Kiểu tấn công này cho phép hacker có thể lấy được gần như toàn bộ data trong database thông qua câu lệnh “UNION”
    -	Với kiểu tấn công này thì cần biết số lượng cột, kiểu dữ liệu của từng cột hay gọi là tham số và kiểu dữ liệu của tham số của câu lệnh SELECT trước đó vì lệnh UNION cần phải khớp về số lượng cột và kiểu dữ liệu thì mới hoạt động được. Có vẻ khó nhưng thật ra thì rất đơn giản vì để đoán được số lượng cột thì có thể viết đoạn code đơn giải rồi chạy vòng for từ 1 đến khi nào tìm được đúng số cột còn về kiểu dữ liệu thì nếu là thông tin người dùng thì thường chỉ có int, double, varchar, hay nvarchar
    -	Hoặc cách khác để biết số lượng cột là dùng ORDER BY “number” – lệnh sắp xếp trong SQL, đến khi nào nhập 1 “number” mà lỗi thì số number -1 là số lượng cột.
    -	GROUP BY “number” – giống ORDER BY
    -	SELECT NULL, NULL,…-- - Hay SELECT 1, 2, 3,…-- -. Chuỗi “-- -” comment hết lệnh phía sau và tránh trường hợp validate dấu cách ở cuối vì sau comment phải cách ra 1 cái thì mới hoạt động bình thường
    -	Lấy tất cả các bảng trong database hiện tại(My SQL): “' UNION SELECT table_name, NULL FROM information_schema.tables--” – trường hợp này số cột ở lệnh SELECT trước là 2.
    ### Blind-SQLi:
    -	Boolean-based SQLi: là kiểu tấn công mà hacker chèn mã SQL vào thì sever sẽ trả về 2 kiểu khác nhau ví dụ như: True – False, Found – Not Found, Yes – No,…
        -	“page.asp?id=1 AND 1=1 -- -“ // True
        -	“page.asp?id=1 AND 1=2 -- -“ // FALSE
        -	“?id=1 AND SELECT SUBSTR(table_name,1,1) FROM information_schema.tables = 'A'” – Lấy ký tự đầu tiên của tên bảng so sánh với ‘A’ nếu đúng->True, sai->False
        -	Với kiểu khai thác này thì chủ yếu là phải viết code chạy để tìm được tên database, tên bảng, cột, data,…Nếu dùng tay thì hơi lâu
        -	Thuật toán tìm kiếm nhị phân được áp dụng để khai thác lỗi SQLi này rất nhanh và phổ biến
    -	Time-Based SQLi: Cũng tương tự như Boolean-based nhưng hacker sẽ khai thác dựa vào thời gian phản hồi của sever – nếu đúng thì thời gian phải hồi là sẽ lớn hơn hoặc bằng thời gian mà hacker chèn vào, còn nếu sai thì ngược lại
        - Ví dụ: “1 and (select sleep(10) from users where SUBSTR(table_name,1,1) = 'A')#” – trong payload này thì nếu ký tự đầu tiên của bảng là ‘A’ thì phải hồi sẽ là sau 10s còn nếu ko đúng sẽ < 10s
	## Ảnh hưởng đến tam giác CIA
    ### Tam giác CIA
   	- Tam Giác CIA là tam giác đại diện cho độ an toàn của một ứng dụng thông qua 3 quy chuẩn sau để đánh giá độ an toàn, bảo mật của hệ thống đến đâu:
      - C – Confidental (Tính bảo mật)✅
      - I – Intergrity (Tính toàn vẹn)✅
      - A – Availability (Tính sẵn sàng)✅
    - Vậy SQLi ảnh hưởng đến tam giác CIA như thế nào. Giả sử một ứng dụng web bị mắc lỗ hổng SQLi thì:
      - Với kiểu tấn công In-band SQLi thì hacker có thể đăng nhập vào tài khoản của người dùng khác, tìm đọc được thông tin nhạy cảm của người dùng -> vi phạm tính bảo mật hệ thống(C)
      - Khi có thể chèn được mã SQL độc hại thì có thể thêm, sửa, xoá các dữ liệu trong database qua các câu lệnh như INSERT, UPDATE, DELETE,... -> vi phạm tính toàn vẹn hệ thống(I)
      - Tiếp theo nếu dữ liệu trong database bị xoá khi chèn thêm lệnh DELETE thì tính sẵn sàng của hệ thống cũng bị mất -> vi phạm tính sẵn sàng(A)<br>
   =>  Vậy có thể kết luận là lỗ hổng SQLi có thể vi phạm đến tất cả các cạnh của tam giác CIA.
	## Trường hợp đặc biệt
# Tài liệu tham khảo:
-	Post
-	Hackstrick 
-	Invicti.com
-	Cookie han hoan
