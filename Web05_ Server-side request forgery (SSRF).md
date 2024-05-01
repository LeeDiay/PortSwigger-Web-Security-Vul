# **1. Lab: Basic SSRF against the local server**
>Mô tả cho biết trong lab có tính năng kiếm tra lượng hàng trong kho, và dữ liệu được lấy từ 1 hệ thống nội bộ. Nhiệm vụ của chúng ta là truy cập được vào trang quản trị admin tại `http://localhost/admin` và xóa người dùng `carlos`
![image](https://hackmd.io/_uploads/SyzdZAi-C.png)

>Thử vào `/admin` nhưng bị chặn và thông báo rằng nó chỉ có thể truy cập bởi tài khoản có quyền admin, hoặc request này được gửi từ loopback:
![image](https://hackmd.io/_uploads/SJh_8As-0.png)

>Truy cập lab và thấy ở mỗi sản phẩm thì có mục để Check stock để kiếm tra số lượng hàng còn trong kho: ![image](https://hackmd.io/_uploads/r1TfV0o-R.png)

>Vào xem HTTP History thì có 1 request `POST /product/stock` được gửi đi, kèm theo tham số `stockApi` để lấy dữ liệu về sản phẩm trong hệ thống nội bộ:
![image](https://hackmd.io/_uploads/ByYNECoZR.png)

>Ta có thể thay đổi giá trị của tham số `stockApi` này, để nó không lấy dữ liệu từ URl kia nữa, mà nó sẽ tự trỏ về chính nó và truy cập vào admin, bằng việc thay payload `stockApi=http://127.0.0.1`: ![image](https://hackmd.io/_uploads/BJ83P0ibR.png)

>Lúc này thì đã hiện ra **"Admin Panel"**, vậy là ta đã thành công trong việc đánh lừa được server! Tiếp tục thêm `/admin` vào payload để vào trang quản tri: ![image](https://hackmd.io/_uploads/S1lcuRs-0.png)

>Lấy đường dẫn để xóa `carlos`: ![image](https://hackmd.io/_uploads/HyM3uCi-C.png)

>Gửi yêu cầu xóa `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1UkYAi-C.png)

![image](https://hackmd.io/_uploads/H1aetRiW0.png)

# **2. Lab: Basic SSRF against another back-end system**
>Mô tả cho biết trong lab có tính năng kiếm tra lượng hàng trong kho, và dữ liệu được lấy từ 1 hệ thống nội bộ. Nhiệm vụ của chúng ta là dò được địa chỉ mạng nội bộ `192.168.0.x` , truy cập được vào trang quản trị admin tại `http://localhost/admin` và xóa người dùng `carlos`
![image](https://hackmd.io/_uploads/SkntsRjZ0.png)

>Khi cố gắng truy cập `/admin`, thì bị thông báo lỗi không tìm thấy: ![image](https://hackmd.io/_uploads/BJg_tlhZ0.png)

>Mỗi sản phẩm sẽ có chức năng kiểm tra số lượng còn lại trong kho: ![image](https://hackmd.io/_uploads/HyBGYlnbR.png)

>Vào xem HTTP History thì có 1 request `POST /product/stock` được gửi đi, kèm theo tham số `stockApi` để lấy dữ liệu về sản phẩm trong hệ thống nội bộ: ![image](https://hackmd.io/_uploads/BkX9KenZC.png)

>Thay đổi payload `http://127.0.0.1` như bài trước thì không có hiệu quả: ![image](https://hackmd.io/_uploads/BkeZqehWA.png)

>Nhớ đến chi tiết trong đề bài, địa chỉ của mạng nội bộ có dạng `192.168.0.X` và trang admin ở port 8080, do đó, ta có thể sử dụng Intruder để scan tìm port!

>Cho request vào Intruder, đặt payload cho `stockApi` và chạy thử từ 0-255:
>![image](https://hackmd.io/_uploads/SJDTqx2W0.png)
![image](https://hackmd.io/_uploads/SyDz2lnZA.png)

>Lọc ra được payload `115` có độ dài gói tin trả về khác biệt nhất, thử thay vào và thành công truy cập được vào trang quản trị:
>![image](https://hackmd.io/_uploads/HksXTgnb0.png)
 ![image](https://hackmd.io/_uploads/rkbFnl3ZA.png)

>Lấy đường dẫn tương ứng với lệnh xóa `carlos`: ![image](https://hackmd.io/_uploads/H1Po3x2-C.png)

>Thực hiện xóa người dùng `carlos`: ![image](https://hackmd.io/_uploads/Bk7ype2b0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hyjgpl2W0.png)

# **3. Lab: SSRF with blacklist-based input filter**
>Mô tả cho biết trong lab có tính năng kiếm tra lượng hàng trong kho, và dữ liệu được lấy từ 1 hệ thống nội bộ. Lần này, thì lập tình viên đã triển khai các biện pháp để chặn SSRF. Nhiệm vụ của chúng ta là truy cập được vào trang quản trị admin tại `http://localhost/admin` và xóa người dùng `carlos`
![image](https://hackmd.io/_uploads/HJ-ATlhbC.png)

>Truy cập vào `/admin` thì thông báo lỗi như bài lab đầu: 
>![image](https://hackmd.io/_uploads/S1JYyb3bA.png)

>Thấy ở mỗi sản phẩm thì có mục để Check stock để kiếm tra số lượng hàng còn trong kho: ![image](https://hackmd.io/_uploads/H1OMlZnZ0.png)

>Vào xem HTTP History thì có 1 request `POST /product/stock` được gửi đi, kèm theo tham số `stockApi` để lấy dữ liệu về sản phẩm trong hệ thống nội bộ:
![image](https://hackmd.io/_uploads/H1DSeW2bR.png)

>Ta có thể thay đổi giá trị của tham số `stockApi` này, để nó không lấy dữ liệu từ URl kia nữa, mà nó sẽ tự trỏ về chính nó và truy cập vào admin, bằng việc thay payload `stockApi=http://127.0.0.1`:
>![image](https://hackmd.io/_uploads/SJLZWWn-C.png)


>Có vẻ như server đã chặn URL nhập vào mà có chứa `127.0.0.1` ! Nhưng ta có thể mã hóa chuỗi này thành 1 dạng khác, để bypass được bộ filter, mà server vẫn có thể hiều được!! Ví dụ ta có thể convert ip sang dạng decimal: ![image](https://hackmd.io/_uploads/HJ9TLWh-A.png)

>Nhưng nó vẫn bị chặn: ![image](https://hackmd.io/_uploads/rJJLdFhbC.png)

>Thử chuyển sang dạng nhị phân: ![image](https://hackmd.io/_uploads/r1tYuYhbR.png)

>Không có hiệu quả! Thử payload sang `127.1`, vì theo quy ước, bất kỳ octet nào bị thiếu giữa phần mạng và phần máy chủ trong địa chỉ IPv4 sẽ được lấp đầy bằng các số 0: ![image](https://hackmd.io/_uploads/HJLdFK3-R.png)

>Nhưng khi truy cập vào `/admin`, ta vẫn bị chặn: ![image](https://hackmd.io/_uploads/rJdnec2b0.png)

>Phải tìm cách bypass tiếp bằng việc mã hóa đi `/admin`, ví dụ dùng URL-encode: ![image](https://hackmd.io/_uploads/SJp5fchb0.png)

>Tiếp tục URL-encode lần 2: ![image](https://hackmd.io/_uploads/HkBpMc2-A.png)

>Thành công tiếp cận được `/admin` ! Lấy đường dẫn để xóa `carlos` và tiếp tục URL-encode đường dẫn này 2 lần: ![image](https://hackmd.io/_uploads/HJhzXq3bC.png)

>Xóa `carlos` thành công: ![image](https://hackmd.io/_uploads/BJgUm93WC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Bkyw7q2W0.png)

# **4. Lab: SSRF with blacklist-based input filter**
> Lab này có tính năng kiểm tra hàng tồn kho để lấy dữ liệu từ hệ thống nội bộ.
Để giải quyết bài lab, thay đổi URL kiểm tra hàng tồn để truy cập vào giao diện quản trị tại `http://192.168.0.12:8080/admin` và xóa người dùng `carlos`.
Phần stock checker đã bị hạn chế chỉ truy cập vào ứng dụng cục bộ, vì vậy trước tiên, cần tìm một chuyển hướng mở ảnh hưởng đến ứng dụng:
![image](https://hackmd.io/_uploads/r1Kxq1abR.png)

> Đầu tiên, truy cập vào đường dẫn admin nhưng bị chặn URL không hợp lệ, nó không cho phép external truy cập vào: ![image](https://hackmd.io/_uploads/Hk5q91pWC.png)

> Nhưng ở mỗi sản phẩm, sẽ có button `Next product` để xem chi tiết sản phẩm tiếp theo: ![image](https://hackmd.io/_uploads/By209J6W0.png)

>Khi thực hiện việc chuyển hướng sang trang tiếp theo, bắt request thì thấy nó truyền vào 1 URL trong `GET /product/nextProduct`: ![image](https://hackmd.io/_uploads/HJpzsJTbA.png)

>Mỗi khi thấy 1 request gửi đi với 1 URL, ta nghĩ ngay có thể khai thác ở chỗ này! Thử dùng payload `127.0.0.1` để test thì có phản hồi tích cực: ![image](https://hackmd.io/_uploads/HJxFiy6-0.png)

> Tiếp tục truy cập tới trang quản trị bằng đường dẫn đề bài cho thì thấy có thể truy cập: ![image](https://hackmd.io/_uploads/HyLS616-0.png)

>Thay payload vào trong request để fetch data: ![image](https://hackmd.io/_uploads/B1gtWea-0.png)

>Lấy đường dẫn để xóa `carlos`: ![image](https://hackmd.io/_uploads/H1Cq-l6WR.png)

>Xóa người dùng `carlos` và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/B1DeGlTbC.png)
> ![image](https://hackmd.io/_uploads/HybyfgpZR.png)

# **5. Lab: Blind SSRF with out-of-band detection**
>Trang web này sử dụng phần mềm phân tích để tìm nạp URL được chỉ định trong HTTP Header Referer khi trang sản phẩm được tải.
Để giải quyết bài lab, sử dụng chức năng Burp Collaborator: 
![image](https://hackmd.io/_uploads/ByD6feTbA.png)

>Truy cập bài lab, ở mỗi sản phẩm thì không còn chức năng Check Stock như những bài trước, cũng không có xem trang sau: ![image](https://hackmd.io/_uploads/By2QRzT-0.png)

>Theo gợi ý, ứng dụng lab này sử dụng 1 software khác luôn fetch đến URL tại trường header **Referer** mỗi khi user truy cập 1 trang sản phẩm bất kì. Như vậy ta có thể Out-Of-Band SSRF bằng cách đưa vào trường Referer URL mà mình control. Copy domain ta vừa tạo trong Burp Collaborator và thay vào request: ![image](https://hackmd.io/_uploads/H18GJ7TZ0.png)

>Gủi request và kiểm tra log của Burp Collaborator, ta thấy HTTP kèm theo DNS query được gửi đến URL. Điều này chứng tỏ software trên đã truy cập đến URL mà mình đã gán trong trường Referer: ![image](https://hackmd.io/_uploads/BJYEymp-R.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Bkqr1QpZR.png)

# **6. Lab: SSRF with whitelist-based input filter**
> Lab này có tính năng kiểm tra hàng tồn kho để lấy dữ liệu từ hệ thống nội bộ.
Để giải quyết bài lab, thay đổi URL kiểm tra hàng tồn để truy cập vào giao diện quản trị tại `http://localhost/admin` và xóa người dùng `carlos`:
![image](https://hackmd.io/_uploads/Sk29J7TZC.png)

>Tiếp tục là lỗ hổng SSRF tại chỗ Check Stock: 
![image](https://hackmd.io/_uploads/BJzziX6bR.png)

>Bài này đã thiết lập 1 white-list để URL đặt trong `stocApi` nhất định phải là ` stock.weliketoshop.net`, chứ filter theo black-list nữa: ![image](https://hackmd.io/_uploads/rJx-2Q6-0.png)

>Sử dụng fragment **#** để chỉ lấy phần trước ,còn phần sau là comment để bypass nhưng không thành công:
>![image](https://hackmd.io/_uploads/SJVdbE6bA.png)

>Thử dùng chèn thêm tài khoản trước hostname bằng `@` thì thấy server đã trả `500` response chứng tỏ server đã cố gắng connect đến URL đó nhưng không thành công: ![image](https://hackmd.io/_uploads/ryJbfE6WC.png)


>Lúc này thử kết hợp fragment`#` (encoded) vào sau localhost và `@`. Tuy nhiên kết quả lần này đã bị fail:
>![image](https://hackmd.io/_uploads/HysomVpZ0.png)


>Double encoded # thành `%25%32%33` thì thấy, ta đã truy cập thành công trang localhost → lỗi parse URL lúc validate URL:
>![image](https://hackmd.io/_uploads/SkCaQ4TWA.png)

>Lúc này thì chỉ cần truy cập vào `/admin`, lấy đường dẫn xóa người dùng `carlos`: ![image](https://hackmd.io/_uploads/B1nD4EabR.png)

>Thưc hiện xóa `carlos`: ![image](https://hackmd.io/_uploads/HJT5EVTWR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r1xh44a-A.png)
