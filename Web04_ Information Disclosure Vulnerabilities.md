# Số bải giải / Tổng số: 5/5
# **1. Lab: Information disclosure in error messages**
>Thông báo lỗi của lab này tiết lộ rằng lab này đang sử dụng một phiên bản framework dễ bị tấn công của bên thứ ba. Để giải quyết bài lab, ta phải lấy và gửi số phiên bản của frame này.
![image](https://hackmd.io/_uploads/ByLeAssxC.png)

>Truy cập bài lab và thấy không có bất kì 1 chức năng nào khác ngoài xem chi tiết sản phẩm: ![image](https://hackmd.io/_uploads/SJMOCsogA.png)

>Nhưng khi vào xem thì cũng không có chức năng nào để khai thác như comment hay upload... 

>Xem các request đã gửi đi thì cũng không có gì: ![image](https://hackmd.io/_uploads/ryaXk2sx0.png)

> Mà đề bài nói phải làm xuất hiện lỗi thì mới có bước tiếp theo! Để ý khi ấn vào 1 sản phẩm bất kì thì đều có 1 request gồm `productID` được gửi đi: ![image](https://hackmd.io/_uploads/HyACy2sgA.png)

>Vậy nếu ta thay đổi giá trị INT này thành 1 dạng khác, ví dụ như chuỗi thì server có hiểu được không? Cho request sang Repeater và thực hiện, thấy 1 thông báo lỗi rất dài được trả về: ![image](https://hackmd.io/_uploads/BJjBgnjgC.png)

>Và ở dưới cùng thông báo lỗi có chứa thông tin phiên bản của Apache và đây là chuỗi ta cần tìm! ![image](https://hackmd.io/_uploads/r1uOlhieR.png)

>Submit và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryBcxnilC.png)

# **2. Lab: Information disclosure on debug page**
> Đề bài cho biết bài lab có 1 trang debug chứa thông tin nhạy cảm về ứng dụng. Nhiệm vụ của chúng ta là lấy được `SECRET_KEY` và submit để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/S1J5H3olR.png)

>Truy cập vào lab, không thấy có chức năng gì ngoài xem chi tiết thông tin sản phẩm: ![image](https://hackmd.io/_uploads/Hyz6LnjgA.png)

> Như bài trước, ta sẽ bắt 1 request khi truy cập vào 1 sản phẩm và đổi giá trị của `productID` thành dạng string: ![image](https://hackmd.io/_uploads/B1BbDnsxC.png)

>Nhưng lần này, server đã được cấu hình để chặn ta làm việc ấy! Lúc này ta phải đi tìm cách khác...

>Sau 1 hồi lục tung các request, và search theo từ khóa **"debug"**, thì ta phát hiện 1 chỗ thú vị, đó là trong source code đã quên xóa 1 comment khi deploy, và trong dòng comment này chứa đường dẫn để tới 1 trang gì đó: ![image](https://hackmd.io/_uploads/Sk2p_2sgA.png)

>Thay đổi URL và chuyến hướng tới 1 trang debug của trang web: ![image](https://hackmd.io/_uploads/HJ_eYnieA.png)

>Từ trang này, ta có thể biết được hết các thông tin cấu hình của bài lab!

>Ctrl F để tìm kiếm tên của chuỗi cần tìm và lấy được nó: ![image](https://hackmd.io/_uploads/SydIK3ixA.png)

>Submit và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B1pvK3jxA.png)

# **3. Lab: Source code disclosure via backup files**
> Mô tả cho biết bài lab này bị rò rỉ mã nguồn của nó thông qua các tập tin sao lưu trong một thư mục ẩn. Để giải quyết bài lab, ta phải xác định và gửi mật khẩu cơ sở dữ liệu được mã hóa cứng trong mã nguồn bị rò rỉ.
![image](https://hackmd.io/_uploads/BJauj3ixC.png)

>Truy cập vào lab, không thấy có chức năng gì ngoài xem chi tiết thông tin sản phẩm: ![image](https://hackmd.io/_uploads/rJi2T3jgC.png)

>Sau khi thử các cách của các lab trước thì nó không hiệu quả... Nhưng khi đề bài nói "thư mục ẩn" thì liệu nó sẽ được lưu ở đâu? 100% ta sẽ nghĩ ngay tới file `robots.txt` . Truy cập vào `/robots.txt` thì thấy có 1 thư mục là `/backup`: ![image](https://hackmd.io/_uploads/Bye8ChjgR.png)

>Truy cập vào `/backup` và thấy 1 file java: ![image](https://hackmd.io/_uploads/BkxOCnieC.png)

> Truy cập vào file này và phát hiện đây là 1 mã nguồn java bị rò rỉ: ![image](https://hackmd.io/_uploads/S1W3AhjlC.png)

>Để ý phần code để connect tới database, ở đây bao gồm các giá trị cho db_id, username và password db: ![image](https://hackmd.io/_uploads/SJKgk6ilC.png)

>Tiến hành lấy password cho db và submit để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ry6zJ6sgR.png)

# **4. Lab: Authentication bypass via information disclosure**
> Mô tả nói rằng ở phần giao diện quản trị bài lab có chứa 1 lỗ hổng Authen. Nhiệm vụ của chúng ta là từ người dùng hợp lệ `wiener:peter`, truy cập được vào trang admin và xóa tài khoản `carlos`:
![image](https://hackmd.io/_uploads/r1dtkaseC.png)

>Truy cập vào bài lab, ta thấy chức năng login: ![image](https://hackmd.io/_uploads/HkLdu6ixR.png)

>Đăng nhập với người dùng `wiener:peter`: ![image](https://hackmd.io/_uploads/HydcOpjxA.png)

>Tiến hành khai thác với cách của các lab trước và không thu lại được gì. Trong đề bài có đề cập tới method `TRACE`: ![image](https://hackmd.io/_uploads/Sks2caix0.png)

> Method `TRACE`được sử dụng để giao tiếp giữa máy khách và máy chủ web. Phương thức TRACE cho phép máy khách gửi một yêu cầu đến máy chủ và máy chủ sẽ phản hồi bằng chính yêu cầu đó, cho phép máy khách kiểm tra hoặc gỡ lỗi các biến đổi mà các máy trung gian có thể thực hiện đối với yêu cầu. Phương thức TRACE thường được sử dụng để kiểm tra xem các yêu cầu đã được nhận bởi máy chủ và có nhận được sự thay đổi nào không khi đi qua các proxy hoặc máy chủ trung gian. Vì vậy, nó thường được tắt khi deploy trang web lên. Nhưng trong lab này, `TRACE` vẫn chưa được tắt và có thể hoạt động được: ![image](https://hackmd.io/_uploads/rJDHj6ixC.png)

>Lúc này, ta có thể thấy được request mà server thực sự nhận được! => Nó có thể khiến user bình thường đọc được các header nhạy cảm. Cụ thể ở đây là `X-Custom-IP-Authorization`. Đây là header có chức năng xác định IP nguồn của request để từ đó trao quyền.

>Khi 1 user bình thường truy cập vào đường dẫn của trang quản trị `/admin`, nó sẽ thông báo chỉ người dùng local mới có thể truy cập vào chức năng quản trị: ![image](https://hackmd.io/_uploads/By9Hhpjx0.png)

>Từ đó, ta có thể bypass bằng cách đổi IP thành IP local như `127.0.0.1`. Tiến hành thêm vào request `X-Custom-Ip-Authorization: 127.0.0.1` và Send: ![image](https://hackmd.io/_uploads/Hkl03pseA.png)

>Cách này đã thật sự hiệu quả!! Cuối cùng là chuột phải và chọn "Show respone in browser" và truy cập vào trang quản trị: ![image](https://hackmd.io/_uploads/rJu-6aixR.png)

>Xóa người dùng `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkKV6polR.png)

# **5. Lab: Information disclosure in version control history**
>Mô tả cho biết bài lab này tiết lộ thông tin nhạy cảm thông qua lịch sử phiên bản của nó. Để giải quyết bài thí nghiệm, lấy mật khẩu cho người dùng quản trị `administrator`, đăng nhập và xóa người dùng `carlos`:
![image](https://hackmd.io/_uploads/S1-D66ogR.png)

>Truy cập vào bài lab, có chức năng đăng nhập nhưng đề không cho 1 tài khoản nào để đăng nhập: ![image](https://hackmd.io/_uploads/S1aLC6og0.png)

>Đề bài cho biết thư mục `/.git` không được giấu đi mà bị public: ![image](https://hackmd.io/_uploads/Sk7i1Aig0.png)

>Dùng `wget` để tải thư mục này về và tiến thành phân tích: 
>`wget -r https://0a3400120333c1108070b7d1002000da.web-security-academy.net/.git`
>![image](https://hackmd.io/_uploads/r1Pmz0oxC.png)

>Di chuyển tới thư mục vừa tải về, sử dụng câu lệnh `git log` để xem được các commit đã thực hiện. Có xuất hiện commit `624f0a11d3515b8669369dea7d3de8d1139b2aa1` thông báo đã xóa mật khẩu admin ra khỏi file config => ta có thể xem được mật khẩu: ![image](https://hackmd.io/_uploads/SyARfCsx0.png)

>Dùng `git show + id_commit` để có thể đọc được nội dung của commit đó. Thấy rằng lần commit này đã thực hiện xóa mật khẩu admin và lưu lại bằng `env('ADMIN_PASSWORD')` tại file `/admin.conf`: 
>![image](https://hackmd.io/_uploads/H1xtQCjgR.png)

>Tuy nhiên ta có thể lấy được mật khẩu cũ và đăng nhập vào `administrator`: 
>![image](https://hackmd.io/_uploads/rkOC7CieR.png)

>Xóa tài khoản `carlos` để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJJyB0jl0.png)
