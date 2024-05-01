# **1. Lab: Excessive trust in client-side controls**
> Lab này không xác nhận đầy đủ thông tin đầu vào của người dùng. Ta có thể khai thác lỗ hổng logic trong quy trình mua hàng của nó để mua các mặt hàng với mức giá ngoài ý muốn. Để giải quyết bài lab, hãy mua "**Lightweight l33t leather jacket**": 
![image](https://hackmd.io/_uploads/rJ6d6CNWR.png)

>Truy cập lab, đăng nhập vào `wiener` và thấy rằng ta đã có sẵn số tiền 100$, và có chức năng thêm sản phẩm vào giỏ hàng, sau đó mới thanh toán: ![image](https://hackmd.io/_uploads/HyJty1HWA.png)


> Thử thêm sản phầm đề bài yêu cầu vào giỏ hàng và thanh toán, nhưng giá của nó là 1337$ và có thông báo ta không đủ tiền để mua sản phẩm này: ![image](https://hackmd.io/_uploads/SJoTykSZR.png)

>Xem các request đã gửi đi trong HTTP History, thì có 1 request với method POST với đường dẫn `cart`, trong này ngoài các tham số như id sản phẩm còn có tham số `price` là giá tiền của nó: ![image](https://hackmd.io/_uploads/ryqUg1BWC.png)

>Chuyển nó tới Repeater và tiến hành sửa đổi, thử đổi `price=999` và Send thì thấy request đã được server chấp nhận mà không hề xảy ra lỗi: ![image](https://hackmd.io/_uploads/Bk1hgyHWA.png)

>Bên ngoài trang web tiến hành tài lại trang và thấy rằng giá tiền sán phẩm đã bị chỉnh sửa dựa trên input khi nãy ta đã gửi: ![image](https://hackmd.io/_uploads/B1Bk-1r-A.png)

> => Kết luận rằng ta có thể tự do chỉnh sửa giá tiền của 1 sản phẩm mà trang web không thực hiện ngăn chặn hành vi này! Lúc này tiến hành thanh toán sản phẩm và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B1q4-kBWC.png)

# **2. Lab: High-level logic vulnerability**
> Lab này không xác nhận đầy đủ thông tin đầu vào của người dùng. Ta có thể khai thác lỗ hổng logic trong quy trình mua hàng của nó để mua các mặt hàng với mức giá ngoài ý muốn. Để giải quyết bài lab, hãy mua "**Lightweight l33t leather jacket**": 
![image](https://hackmd.io/_uploads/HyoXfkBWA.png)

>Truy cập lab, đăng nhập vào `wiener` và thấy rằng ta đã có sẵn số tiền 100$, và có chức năng thêm sản phẩm vào giỏ hàng, sau đó mới thanh toán: ![image](https://hackmd.io/_uploads/r1oZB1Hb0.png)

>Thử thêm sản phầm đề bài yêu cầu vào giỏ hàng và thanh toán, nhưng giá của nó là 1337$ và có thông báo ta không đủ tiền để mua sản phẩm này: ![image](https://hackmd.io/_uploads/S1T7HJrWC.png)

> Xem các request đã gửi đi trong HTTP History, thì có 1 request với method POST với đường dẫn `cart`, có các tham số là `productId` và `quantity`, nhưng lần này không có tham số `price` như lab trước nữa: ![image](https://hackmd.io/_uploads/Hk-Jw1S-A.png)

>Thử sửa lại cho `quantity=-9` thành số âm và Send và được server chấp nhận: ![image](https://hackmd.io/_uploads/ryXOvkSbC.png)

>Tải lại trang và thấy giá tiền lúc này là số âm: ![image](https://hackmd.io/_uploads/BJA9vySbC.png)

>Hí hững nghĩ rằng khi số tiền là số âm, theo đúng logic sẽ là: **"Số tiền của mình" - "Số tiền âm" = Mua được sản phẩm** và tiền trong tài khoản được tăng lên!! Nhưng không.. Server đã chặn trường hợp này và có thông báo rằng **"Cart total price cannot be less than zero"**

>Tức là số tiền thanh toán sẽ phải > 0 mới được tính. Vậy thì ta sẽ có thể chọn mua thêm 1 sản phẩm khác, sau đó để giá tiền của 2 sán phẩm cộng lại với nhau > 0 thì có thể bypass được.

>Chỉnh sửa để tổng giá 2 sản phẩm nằm trong đoạn 0 - 100 để có thể mua được: 
![image](https://hackmd.io/_uploads/Hk3791HWR.png)

>Thanh toán hóa đơn và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BkVB51SZA.png)

# **3. Lab: Inconsistent security controls**
> Logic thiếu sót của lab này cho phép người dùng tùy ý truy cập chức năng quản trị mà lẽ ra chỉ dành cho nhân viên công ty. Để giải quyết bài thí nghiệm, hãy truy cập bảng quản trị và xóa người dùng `carlos`.
![image](https://hackmd.io/_uploads/ry-40ySbR.png)

>Truy cập vào lab thì thấy có chức năng đăng kí tài khoản người dùng, và nếu ai làm việc cho DontWannaCry thì đăng kí email có đuôi là `@dontwannacry.com`:
>![image](https://hackmd.io/_uploads/Hy08JgBWC.png)

> Thử đăng kí với email `@dontwannacry.com`:
> ![image](https://hackmd.io/_uploads/rkp6kgrZR.png)

>Một đường dẫn đã được gửi tới hòm thư mà ta đăng kí, nhưng ta lại không có cách nào để truy cập vào nó: 
![image](https://hackmd.io/_uploads/ByHeleH-R.png)

>Lúc này chỉ đành tạo 1 tài khoản mới với hòm thư được cung cấp và nhận được mã: ![image](https://hackmd.io/_uploads/HkKUgeHbA.png)
![image](https://hackmd.io/_uploads/BkD_lgS-R.png)

>Tạo tài khoản thành công: ![image](https://hackmd.io/_uploads/S1Qjxxr-R.png)

> Nhưng trong mục **"My account"** lại có chức năng cho người dùng có thể tự cập nhật lại địa chỉ email: ![image](https://hackmd.io/_uploads/HJUCggHb0.png)

>Thử đổi email thành đuôi `@dontwannacry.com` và update: ![image](https://hackmd.io/_uploads/B1QW-eB-0.png)

>Có thể thấy rằng người dùng bình thường có thể lợi dụng vào phần cập nhập email này để tự biến mình trở thành người trong công ty và có quyền quản trị: ![image](https://hackmd.io/_uploads/H1lzVZxBZ0.png)

>Truy cập vào **"Admin Panel"**: ![image](https://hackmd.io/_uploads/S1P_bxrZ0.png)

>Xóa người dùng `carlos` để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SynFbeSZ0.png)

# **4. Lab: Flawed enforcement of business rules**
>Lab này có một lỗ hổng logic trong quy trình mua hàng của nó. Để giải quyết bài lab. Để giải quyết bài lab, hãy mua "**Lightweight l33t leather jacket**":
![image](https://hackmd.io/_uploads/HyyFMxHWC.png)

>Truy cập lab, đăng nhập vào `wiener` và thấy rằng ta đã có sẵn số tiền 100$, và có chức năng thêm sản phẩm vào giỏ hàng, sau đó mới thanh toán: ![image](https://hackmd.io/_uploads/r1UZmlSWA.png)

>Bài này khác các lab trước ở chỗ: hệ thống cho sẵn các coupon giảm giá cho người dùng, và khi thanh toán có thể apply nó và trừ tiền: ![image](https://hackmd.io/_uploads/BkMhVgHWC.png)

>Một suy nghĩ mà bất cứ ai cũng nghĩ là apply coupon này nhiều lần và tổng tiền bị trừ 5$ cho tới khi thanh toán được. Nhưng khi nhập lại coupon thì đời không như là mơ: ![image](https://hackmd.io/_uploads/rJazSlBbR.png)

>Trong trang chủ, còn có 1 trường ta có thể nhập vào nữa, là điền thông tin email để cập nhật các tin tức mới nhất và sẽ nhận được mã giảm giá 30% cho tổng tiền: ![image](https://hackmd.io/_uploads/S1V5SxH-R.png)
>![image](https://hackmd.io/_uploads/SkJLreBWA.png)


>Quay lại giỏ hàng và apply mã này vào và được giảm tiền: ![image](https://hackmd.io/_uploads/S13TBlrWR.png)

>Lúc này, lại nhập mã **"NEWCUST5"** vào và lần này mã này được sử dụng thành công mặc dù đã dùng trước đó: ![image](https://hackmd.io/_uploads/r1pbIgBb0.png)

>=> Đoán rằng, mỗi lần nhập 1 mã mới thành công thì ta lại có thể apply lại mã đã dùng trước đó mà không bị chặn bởi server, thực hiện submit nhiều lần các mã này cho tới khi tổng hóa đơn nằm trong khoảng 0 - 100 để ta có thể mua được: ![image](https://hackmd.io/_uploads/BJNFLlr-0.png)

>Thanh toán hóa đơn thành công và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJOcIgSZC.png)

# **5. Lab: Low-level logic flaw**
> Lab này không xác nhận đầy đủ thông tin đầu vào của người dùng. Ta có thể khai thác lỗ hổng logic trong quy trình mua hàng của nó để mua các mặt hàng với mức giá ngoài ý muốn. Để giải quyết bài lab, hãy mua "**Lightweight l33t leather jacket**": 
![image](https://hackmd.io/_uploads/Hk8KwgrWA.png)

>Truy cập lab, đăng nhập vào `wiener` và thấy rằng ta đã có sẵn số tiền 100$, và có chức năng thêm sản phẩm vào giỏ hàng, sau đó mới thanh toán: ![image](https://hackmd.io/_uploads/Byg0RdgSbR.png)

>Thử thêm sản phầm đề bài yêu cầu vào giỏ hàng và thanh toán, nhưng giá của nó là 1337$ và có thông báo ta không đủ tiền để mua sản phẩm này: ![image](https://hackmd.io/_uploads/HkSlFgH-C.png)

> Xem các request đã gửi đi trong HTTP History, thì có 1 request với method POST với đường dẫn `cart`, có các tham số là `productId` và `quantity`: ![image](https://hackmd.io/_uploads/H1CMseHZ0.png)

> Thử sửa lại cho `quantity=100` thì bị server filter: ![image](https://hackmd.io/_uploads/BJJ_ieSbR.png)

>Nhưng khi `quantity=99` thì vẫn được: ![image](https://hackmd.io/_uploads/H1iKieSZA.png)

>Chứng tỏ rằng giới hạn trong mỗi lần mua 1 sản phẩm là 99. Thử nhập số âm vào trường này, server vẫn chấp nhận nhưng khi số lượng sản phẩm < 0 thì lại không hiển thị trong giỏ hàng: ![image](https://hackmd.io/_uploads/rJ89heHbC.png)
![image](https://hackmd.io/_uploads/Bk0q3gH-0.png)

> Vậy nếu như ta gửi liên tục rất nhiều request và tổng số tiền vượt qua khỏi giới hạn nó được định nghĩa thì sẽ như thế nào?? Ta sẽ cho request vào Intruder, thiết lập `quantity=99` trong mỗi request và thêm payload=`null` tại bất cứ đâu: ![image](https://hackmd.io/_uploads/rkia7-r-A.png)
> ![image](https://hackmd.io/_uploads/rku07-rZA.png)
![image](https://hackmd.io/_uploads/BkBCVZBbC.png)

>Attack và thấy số tiền tăng lên liên tục, cho tới khi nó từ số dương bị reset thành số âm: ![image](https://hackmd.io/_uploads/HyezEZH-R.png)

> => Bị lỗi **Integer Overflow**, cụ thể khi ta mua quá nhiều sản phẩm dẫn đến tổng hóa đơn sẽ tràn từ **2147483647** sang **-2147483648** và tiến về **0**

> Vì ta mua `"Lightweight l33t leather jacket"` có giá `1337$` 1 lần `99` cái nên ta cần tính toán sao cho tổng hóa đơn sẽ là số âm bé lớn nhất có thể. Cụ thể đó là `-64060` sau 325 request: ![image](https://hackmd.io/_uploads/SJMaw-BZR.png)

>Và vì `64060 / 1337 = 47` nên ta sẽ mua thêm 1 lần 47 sản phẩm nữa. Không thể là 48 sản phẩm vì khi đó tổng tiền nó sẽ là số dương > `100$` tài khoản có: ![image](https://hackmd.io/_uploads/BkUbdbH-R.png)
>![image](https://hackmd.io/_uploads/Hk97u-S-C.png)

>Lúc này, ta sẽ mua thêm 1 sản phẩm khác để tổng tiền bù trừ cho nhau và ra được số dương trong khoảng 0 - 100. Ví dụ chọn sản phẩm với giá `63.54$`, số lượng 20: ![image](https://hackmd.io/_uploads/BkRp_-B-0.png)

>Thanh toán thành công và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryPJFbB-0.png)

# **6. Lab: Inconsistent handling of exceptional input**
> Lab này không xác nhận đầy đủ thông tin đầu vào của người dùng. Ta có thể khai thác lỗ hổng logic trong quá trình đăng ký tài khoản của nó để có quyền truy cập vào chức năng quản trị. Để giải quyết bài thí nghiệm, truy cập bảng quản trị và xóa người dùng `carlos`:
![image](https://hackmd.io/_uploads/BJtvY-BZR.png)

>Lab này cũng giống bài lab trước, chỉ khác nó không còn chức năng update email để ta dùng nữa: ![image](https://hackmd.io/_uploads/r1EJ2WHZR.png)

> Nhưng khi thử đăng kí với email rất dài, thì server sẽ chỉ lấy 255 kí tự đầu tiên để render ra: ![image](https://hackmd.io/_uploads/By0QhIB-R.png)

>Màn hình hiển thị: ![image](https://hackmd.io/_uploads/rkc82LrbR.png)
>![image](https://hackmd.io/_uploads/ry546LSWA.png)


> Mặt khác vì email được cấp có thể nhận được tất cả các mail kể cả các subdomain nên ta sẽ có thể chèn `@dontwannacry.com`. vào domain của email đăng kí và kết hợp với các kí tự khác sao cho khi server lấy 255 kí tự đầu thì kết quả sẽ là `<...>@dontwannacry.com`: ![image](https://hackmd.io/_uploads/BkVa2Irb0.png)

> Vì `@dontwannacry.com` gôm 17 kí tự nên ta sẽ chèn vào trước đó 238 kí tự để khi server render ra email thì email của account của ta sẽ có đuôi `@dontwannacry.com`, mà link xác nhận tài khoản vẫn được trả về email của ta. Cú pháp sẽ là: `'238_kí_tự + '@dontwannacry.com.' + '.exploit-<LABID>.exploit-server.net'`Có thể sử dụng payload: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com.exploit-0af6008f0494e10f85a0a80c010c007a.exploit-server.net`: ![image](https://hackmd.io/_uploads/rJUVHPH-R.png)

>Lúc này, kiểm tra hòm thư và thấy đã có email xác nhận tài khoản gửi về: ![image](https://hackmd.io/_uploads/H1GzHwrbC.png)

> Xác nhận tài khoản và ta đã bypass được, trở thành 1 nhân viên của công ty, có quyền quản trị viên: ![image](https://hackmd.io/_uploads/HJEuSDHWR.png)

> Vào "Admin Panel" và xóa `carlos` để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HkyiSPSbC.png)
![image](https://hackmd.io/_uploads/B19jSDHZC.png)

# **7. Lab: Weak isolation on dual-use endpoint**
> Để hoàn thành bài lab, thì ta cần phải truy cập được vào tài khoản `administrator` và xóa người dùng `carlos`: 
![image](https://hackmd.io/_uploads/SkD-LDH-A.png)

> Đăng nhập vào `wiener`, tại trang `/my-account`, có chức năng để người dùng có thể đổi mật khẩu: ![image](https://hackmd.io/_uploads/rk1Vb_BWA.png)

> Thử đổi mật khẩu: ![image](https://hackmd.io/_uploads/BJ7o-dH-C.png)

> Xem request đó trong HTTP History: ![image](https://hackmd.io/_uploads/rJeRWdHWR.png)

> Có thể thấy rằng đầu vào nhận 4 tham số: `username`, `current-password`, `new-password` và `new-password-2`
> Thử đổi giá trị của `username`: ![image](https://hackmd.io/_uploads/r1UzmdB-C.png)

>Kết luận rằng trường này thay đổi cũng không có tác dụng gì... Tiếp theo đến `current-password`, xóa giá trị của nó đi thì không có thêm tín hiệu gì: ![image](https://hackmd.io/_uploads/By7t7OSWC.png)

>Nhưng khi xóa hẳn tham số này đi, chỉ gửi 3 tham số còn lại thì server vẫn trả về thành công: ![image](https://hackmd.io/_uploads/Sy-pm_SZA.png)

>Như vậy thì ta đã đổi mật khẩu của `administrator` thành **123**, đăng nhập vào:![image](https://hackmd.io/_uploads/SkNUE_SbA.png)

> Xóa `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hkbd4uBWC.png)

# **8. Lab: Insufficient workflow validation**
>Nhiệm vụ của ta là sử dụng lỗ hổng logic trong luồng mua hàng để có thể mua được **"Lightweight l33t leather jacket"**. Cho trước 1 tài khoản hợp lệ `winer:peter`
![image](https://hackmd.io/_uploads/HkCkcyI-A.png)

>Khi ta thực hiện thanh toán thành công, thì đã có 2 request được gửi đi là: 
>**GET /cart/order-confirmation?order-confirmed=true** - xác nhận các sản phẩm mua
>**POST /cart/checkout** - đây là request để tiến hành thanh toán
>![image](https://hackmd.io/_uploads/S1u7jJIZ0.png)

>Tuy nhiên, ở request xác nhận mua thì không hề có bất kì 1 tham số nào trong này, vì thế nó không thể xác định được chính xác người dùng đang thanh toán sản phẩm nào => Có thể bypass bằng cách chọn sản phẩm ta định mua vào giỏ hàng, và chỉ việc Send lại request xác nhận mua thì nó sẽ tự động xác nhận rằng số tiền của ta đã đủ và mua hàng thành công! Điều này dẫn tới việc ta có thể mua bất cứ mặt hàng nào, với bất kì số lượng bao nhiêu 1 cách miễn phí...
>![image](https://hackmd.io/_uploads/B1H6hkLbC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BJ8Ahk8WC.png)

# **9. Lab: Authentication bypass via flawed state machine**
> Mô tả cho biết chuỗi hành động trong quá trình đăng nhập bị thiếu sót. Nhiệm vụ của ta là lợi dụng nó, truy cập vào giao diện của admin và xóa người dùng `carlos`. Cho người dùng hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/BkqIpkL-C.png)

>Khi đăng nhập vào `wiener` thì ta còn thêm 1 bước để chọn role:
>![image](https://hackmd.io/_uploads/BJlIglIWA.png)

>Chọn role 'User': ![image](https://hackmd.io/_uploads/SJWtzhUZC.png)

>Sẽ có 1 request với method POST được gửi đi khi chọn role: ![image](https://hackmd.io/_uploads/BJR2f3U-A.png)

>Và nếu đến bước này, ta Drop request chọn role, và chỉnh sửa lại URL về trang chủ, thì server sẽ set role của chúng ta mặc định là `admin`![image](https://hackmd.io/_uploads/ryINt3U-A.png)
![image](https://hackmd.io/_uploads/rkFUKhLZC.png)

>Xóa người dùng `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJNFKn8WC.png)

# **10. Lab: Infinite money logic flaw**
>Nhiệm vụ của ta là sử dụng lỗ hổng logic trong luồng mua hàng để có thể mua được **"Lightweight l33t leather jacket"**. Cho trước 1 tài khoản hợp lệ `winer:peter`
![image](https://hackmd.io/_uploads/rJVOqnIWR.png)

>Đăng nhập vào `wiener` và ở `/my-account` có chức năng update email và nhập Gift Card:
>![image](https://hackmd.io/_uploads/BJTk6EP-A.png)

>Lấy gift card bằng cách mua sản phẩm cùng tên giá 10$, và thanh toán xong thì nhận được:
![image](https://hackmd.io/_uploads/rkWrTNDWC.png)

>Khi sử dụng code này, ta sẽ được hoàn lại tiền 10$.

>Ngoài ra, web còn tặng 1 mã giảm giá 30% khi đăng kí email để nhận tin tức mới: ![image](https://hackmd.io/_uploads/SkzrANDZC.png)
![image](https://hackmd.io/_uploads/rkaBR4DWA.png)

>Khi áp mã 30% này, ví dụ với sản phẩm Gift Card giá 10$ , sẽ chỉ cần trả 7$ . Và sau khi tốn 7$ mua hàng rồi lại áp mã hoàn tiền 10$ thì có nghĩa là ta sẽ lãi 3$ được cộng thêm vào tài khoản. => Ta có thể thực hiện hành động áp mã rồi mua hàng vô số lần, và tiền sẽ được cộng vô hạn lần...

>Ta sẽ tiến hành việc lặp này bằng cách sử dụng macro trong Intruder. Các request theo thứ tự thực hiện như sau: 
>`POST /cart - thêm sản phẩm gift card vào giỏ hàng`
>`POST /cart/coupon - áp mã giảm giá 30%`
>`POST /cart/checkout -  tiến hành thanh toán`
>`GET /cart/order-confirmation?order-confirmed=true - xác nhận thanh toán và lấy được gift card`
> `POST /gift-card - áp mã giảm giá vừa lấy và được hoàn tiền`

>Cài đặt lại cho request 5 để lấy được mã code từ request 4 sau khi xác nhận thanh toán xong: 
>![image](https://hackmd.io/_uploads/BJhJBHw-C.png)

> Test và thấy macro đã hoạt động đúng: 
> ![image](https://hackmd.io/_uploads/Sy2UuHD-A.png)

>Bây giờ, chuyển request `GET /my-account` sang intruder để bắt đầu thực hiện: ![image](https://hackmd.io/_uploads/HyCxYrD-A.png)

>Set null payload và chạy với số lần vô hạn: ![image](https://hackmd.io/_uploads/HkKfFSPZA.png)

>Cho tối đa 1 request được gửi đi mỗi lần: ![image](https://hackmd.io/_uploads/H1DVtSvbC.png)

>Thực hiện attack và thấy được số tiền đang được tăng lên: ![image](https://hackmd.io/_uploads/Skzog8vZA.png)

>Khi số tiền đã >= sản phẩm mình cần thì dừng, và tiến hành thanh toán sản phẩm được yêu cầu mua và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SkKoQUPW0.png)

# **11. Lab: Authentication bypass via encryption oracle**
> Mô tả cho biết bài lab có lỗ hổng logic để lộ cách mã hóa người dùng. Nhiệm vụ của ta là truy cập được vào chức năng quản trị và xóa người dùng `carlos`. Cho sẵn 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/BJK6-IvWC.png)

>Truy cập vào lab và thấy có chức năng **"Stay logged in"** ở phần đăng nhập: ![image](https://hackmd.io/_uploads/r1jG4IPb0.png)

>Đăng nhập thành công thì thấy tồn tại 1 cookie `stay-logged-in`: ![image](https://hackmd.io/_uploads/Hy_LNIvb0.png)

>Tìm xem nó được mã hóa theo thuật toán nào: ![image](https://hackmd.io/_uploads/r1rBr8vW0.png)

>Nhưng khi decode ra lại không có kết quả gì: ![image](https://hackmd.io/_uploads/BJxwHIDbA.png)

>Mặt khác, tại mỗi bài post đều có chức năng comment cho mỗi bài. Khi nhập thử email không đúng định dạng nó không yêu cầu phải nhập đúng định dạng email: ![image](https://hackmd.io/_uploads/SJWuvIwb0.png)

>Thay vào đó là hiển thị thông báo lỗi: ![image](https://hackmd.io/_uploads/SkhYvLwZA.png)

>Lúc này, thì 1 cookie `notification` xuất hiện: ![image](https://hackmd.io/_uploads/HyJ6P8DbR.png)

>Thử thay đổi giá trị này thì có thông báo lỗi: ![image](https://hackmd.io/_uploads/HJxX_LDWC.png)

>Chứng tỏ rằng, server sẽ giải mã giá trị của `notification` và thông báo ra màn hình lỗi. Và thấy đuôi của cookie này là `%3d` => 2 cookie `sty-logged-in` và `notification` sử dụng cùng 1 thuật toán mã hóa!

>Thử lấy giá trị của `stay-logged-in` thay vào `notification` thì nó sẽ hiển thị lên `username:<timestamp>`: ![image](https://hackmd.io/_uploads/SyCktLPZR.png)

>=> Suy luận của ta là đúng khi 2 cookie này cùng sử dụng chung 1 thuật toán. Bây giờ, ta cần tạo một giá trị `stay-logged-in` cho administrator dựa vào cách mã hóa của trường `notification`. Và vì `notification` được tạo thông qua trường email nên ta sẽ truyền payload tại email. Ta sẽ truyền `administrator:<timestamp>` tại email: ![image](https://hackmd.io/_uploads/HkAmGtv-0.png)

> Có thể thấy thông báo `Invalid email address: administrator:1672593998338` với `notification` là dạng base64 mã hóa của nó.