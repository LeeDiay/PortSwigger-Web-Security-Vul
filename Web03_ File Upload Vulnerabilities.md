# Số bải giải / Tổng số: 6/7

# **1. Lab: Remote code execution via web shell upload**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Nó không thực hiện bất kì 1 sự validate nào khi người dùng tải file lên trước khi lưu file vào trong hệ thống. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell đơn giản để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 gười dùng hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/H1lJnDql0.png)

> Đăng nhập vào `wiener` và thấy có chức năng cho phép upload ảnh: ![image](https://hackmd.io/_uploads/r1fdAw5x0.png)

>Thử upload 1 ảnh hợp lệ, sẽ có 1 request với method **POST** được gửi đi: ![image](https://hackmd.io/_uploads/B1cZd_5gC.png)

>Và khi load lại trang thì sẽ có 1 request để fetch ảnh vừa upload để hiển thị lên: ![image](https://hackmd.io/_uploads/BksBOOqlR.png)

> Vì đề bài nói server không thực hiện bất kì 1 hành động lọc đầu vào nào khi upload file nên ta có thể upload 1 file PHP với chức năng lấy nội dung bên trong file `/home/carlos/secret`
> ![image](https://hackmd.io/_uploads/ryXLKO5xC.png)

>Submit thành công: ![image](https://hackmd.io/_uploads/S1tYtdcxA.png)

> Tải lại trang và thấy trong history, file đã được thực thi và ta có thể đọc được nội dung file cần tìm: ![image](https://hackmd.io/_uploads/SJ_RY_5x0.png)

>Submit nội dung cần tìm: ![image](https://hackmd.io/_uploads/Syrx9_qlC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SyEZqOqlA.png)

# **2. Lab: Web shell upload via Content-Type restriction bypass**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Nó đã thực hiện ngăn chặn người dùng upload các file có kiểu không mong muốn. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell đơn giản để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 gười dùng hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/Bkmr5_9xA.png)

>Truy cập lab với tài khoản `wiener:peter` và thử upload file PHP trong lab trước `test.php`: ![image](https://hackmd.io/_uploads/SJY4h_ceC.png)

>Nhưng lần này server đã chặn ý định upload shell của mình lại vi kiểu file không được cho phép: 
>![image](https://hackmd.io/_uploads/HyqDhd9g0.png)

>Tiến hành thử upload 1 file avatar hợp lệ để xem kiểu file được cho phép là như thế nào: ![image](https://hackmd.io/_uploads/Bkxw6OqxC.png)

>Lấy được định dạng mà server mong muốn là `Content-Type: image/jpeg`, lúc này mình thử đổi lại định dạng trước khi Submit shell xem có bypass được không: ![image](https://hackmd.io/_uploads/H1306_9eR.png)

>Không ngoài dự đoán thì ta có thể bypass bằng cách này!

>Cuối cùng chỉ cần tải lại trang và quan sát history để lấy nội dung trong file: ![image](https://hackmd.io/_uploads/HktV0dcxR.png)

>Submit và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1I8AdcxR.png)

# **3. Lab: Web shell upload via path traversal**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Nó đã được cấu hình để chặn việc thực thi các file mà người dùng upload, nhưng có thể khai thác bằng 1 cách khác. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/BkklgK9x0.png)

>Đăng nhập vào `wiener:peter` và thử upload 1 file code PHP: ![image](https://hackmd.io/_uploads/SkE5fF5xR.png)

> Server trả về thông báo upload thành công: ![image](https://hackmd.io/_uploads/H1ioftqe0.png)

>Nhưng khi tải lại trang và xem request fetch file đó về, code trong file không được thực thi => server đã chặn việc thực thi code trong file ta upload lên: ![image](https://hackmd.io/_uploads/Sk5k7Fqx0.png)

>Từ gợi ý của đề bài, nếu không dùng được cách này thì ta có thể dùng **"Path Traversal"**, exploit vào trường `filename` trong request khi **"POST"** yêu cầu lên!
>Thay vì file được lưu ở trong thư mục `/files/avatars/`, ta sẽ khiến cho file upload lên lưu ở trong `/files` bằng cách dùng payload: `../<file-name>.php` khi gửi đi: ![image](https://hackmd.io/_uploads/BkhDEtqe0.png)

>Có vẻ như cách này có hiệu quả, nhưng thông báo **"The file avatars/test.php has been uploaded"** cho biết rằng file ta vừa upload lên vẫn chưa được lưu ở chỗ ta mong muốn! Có lẽ như server đã thực hiện decode lại `filename` khi nhận được yêu cầu. Thử URL-encode lại `../<file-name>.php`: ![image](https://hackmd.io/_uploads/HyOfBYcg0.png)

>Send lại và lần này thì file đã được lưu ở vị trí ta cần: ![image](https://hackmd.io/_uploads/Hk1IBK9eC.png)

>Sửa lại request fetch ảnh sao cho lấy đúng vị trí file được lưu và Send:  ![image](https://hackmd.io/_uploads/rk4G8K5eR.png)

>Lần này thì code trong file đã được thực thi và lấy được nội dung trong `/home/carlos/secret` ! Chứng tỏ server chỉ cấu hình để chặn việc thực thi code bên trong `/files/avatars` chứ không chặn ở trong `/files/`...

>Submit flag và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1_FLYclR.png)

# **4. Lab: Web shell upload via path traversal**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Một số phần mở rộng tệp nhất định bị đưa vào danh sách đen, nhưng biện pháp bảo vệ này có thể bị bỏ qua do lỗ hổng cơ bản trong cấu hình của danh sách đen này. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/SJoWOtcgR.png)

>Đăng nhập vào `wiener:peter` và thử upload 1 file code PHP: ![image](https://hackmd.io/_uploads/SkE5fF5xR.png)

>Nhưng server đã chặn và không cho phép tải lên file PHP: ![image](https://hackmd.io/_uploads/rJ6NF05gC.png)

>Thử đổi tên thành file có đuôi hợp lệ: `test.php` thành `test.php.png` và Send: ![image](https://hackmd.io/_uploads/Sk--q05eR.png)

>Thấy rằng có thể bypass được blacklist, tiến hành vào 1 request để fetch ảnh từ server và thay đổi đường dẫn tới `avatars/test.php.png` và đọc nội dung trả về: ![image](https://hackmd.io/_uploads/rJjIqC5xR.png)

>Như vậy thì dù có bypass được blacklist nhưng lệnh vẫn không được thực thi... Nhưng trong respone trả về lại cho biết server đang sử dụng Apache → Có thể server chặn thực thi code PHP tại thư `mục /avatars` bằng file `.htaccess`  → ta hoàn toàn có thể ghi đè file `.htaccess` bằng cách upload file `.htaccess` mới. Cụ thể, ghi thêm nội dung: 
>`AddType application/x-httpd-php .hack`
>Tác dụng của lệnh trên là config lại sao cho server cho phép thực thi tất cả những file có phần mở rộng là `.hack` 

>Thực hiện upload file `.htaccess` với cấu hình mới để ghi đề lên cấu hình cũ: ![image](https://hackmd.io/_uploads/Hy7GTR5lA.png)

>Sau đó, đổi `test.php` thành `test.hack` rồi upload file lên: ![image](https://hackmd.io/_uploads/SymQR09xR.png)

>Nhận được respone 200, chứng tỏ cách của ta là đúng! Bây giờ chỉ việc thay đổi đường dẫn tới `avatars/test.hack` và đọc nội dung trả về: ![image](https://hackmd.io/_uploads/SydDykslR.png)

>Submit flag và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HyLKJJjl0.png)

# **5. Lab: Web shell upload via obfuscated file extension**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Một số phần mở rộng tệp nhất định bị đưa vào danh sách đen, nhưng biện pháp bảo vệ này có thể bị bỏ qua bằng cách che giấu. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/Sy9xHyilR.png)

>Đăng nhập vào `wiener:peter` và thử upload 1 file code PHP: ![image](https://hackmd.io/_uploads/SkE5fF5xR.png)

>Nhưng server đã chặn và chỉ cho phép tải lên file với định dạng phần mở rộng là JPG và PNG: ![image](https://hackmd.io/_uploads/SJ7LIyjxA.png)

>Thử đổi `filename=test.php.jpg` và có thể bypass được bộ lọc:![image](https://hackmd.io/_uploads/Sk9X_yslA.png)

>Thay đổi đường dẫn trỏ tới file vừa upload trong 1 request để fetch ảnh từ server: ![image](https://hackmd.io/_uploads/r1_8ukilR.png)

>Có thể thấy rằng code không được thực thi! Ta sẽ bypass bằng cách chèn null byte vào sau `.php` để trở thành `test.php%00.jpg`. Lúc này server vẫn check phần mở rộng file là `.jpg`.Tuy nhiên, khi lưu file thì hệ thống sẽ thấy null byte và ngắt chuỗi → file được lưu là `test.php` : ![image](https://hackmd.io/_uploads/HyVZtkoxC.png)

>Cuối cùng là truy cập vào đường dẫn chứa file và lấy được chuỗi cần tìm: ![image](https://hackmd.io/_uploads/B1vrYyseR.png)

>Submit và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryRUF1olC.png)

# **6. Lab: Remote code execution via polyglot web shell upload**
> Mô tả cho biết bài lab chứa một lỗ hổng trong chức năng upload ảnh. Lần này thì server sẽ kiểm tra nội dung bên trong của file chứ không phải type hay phần extension nữa. Nhiệm vụ của chúng ta là upload 1 file PHP Webshell để lấy được nội dung bên trong file có sẵn `/home/carlos/secret` và subit nội dung đó để hoàn thành bài lab. Cho sẵn 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/H1kgv_ogA.png)

>Đăng nhập vào `wiener:peter` và thử upload 1 file code PHP: ![image](https://hackmd.io/_uploads/SkE5fF5xR.png)

>Nhưng vì nội dung trong file không phải là định dạng mà 1 ảnh nên có, nên bị server từ chối: ![image](https://hackmd.io/_uploads/BkSWauoeC.png)

>Vì các file ảnh thường có phần mã hex cố định, ví dụ như trong file PNG sẽ có các bytes đầu, gọi là "magic bytes" là `89 50 4E 47 0D 0A 1A 0A` : ![image](https://hackmd.io/_uploads/HyXmlYsgR.png)

> Nên hướng đi của ta là sẽ vào app để chèn thêm phần đầu này vào đoạn shell ta muốn thực thi để có thể bypass lưu lên server!! Ví dụ ta dùng phần mềm HxD để chèn thêm: 
![image](https://hackmd.io/_uploads/By4OxKsxA.png)

> Sau đó lưu và upload lại file và đã được server chấp nhận: ![image](https://hackmd.io/_uploads/rkz5eFilR.png)

>Cuối cùng là truy cập vào đường dẫn chứa file và lấy được chuỗi cần tìm:
![image](https://hackmd.io/_uploads/HkfPbYoxR.png)

>Submit chuỗi cần tìm và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1idZKseA.png)
