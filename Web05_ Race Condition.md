# **1. Lab: Limit overrun race conditions**
>Mô tả cho biết luồng mua hàng của lab này có chứa lỗ hổng race condition, cho phép ta có thể mua sản phẩm với mức giá ngoài dự kiến. Cho sẵn người dùng hợp lệ `wiener:peter` , hoàn thành bài lab bằng cách mua sản phẩm "**Lightweight L33t Leather Jacket**": 
![image](https://hackmd.io/_uploads/ryeyiFP-R.png)

>Đăng nhập vào `wiener`, thì ta đã có sẵn trong tài khoản 50$, và có 1 mã giảm giá 20% cho đơn hàng: ![image](https://hackmd.io/_uploads/SJIo2FPW0.png)

>Nếu ta áp mã lần 2 thì nó sẽ báo lỗi mã đã được sử dụng: ![image](https://hackmd.io/_uploads/BJwK1cwbC.png)

>Lúc này, với request khi áp mã giảm giá `POST /cart/coupon`, ta sẽ thử send đồng thời rất nhiều request, xem server có xử lí trường hợp này hay không! 
![image](https://hackmd.io/_uploads/r17lZcDbR.png)

> Chuyển request tới Repeater, và thêm request này vào 1 nhóm mới: ![image](https://hackmd.io/_uploads/Sy_0-qwZA.png)

>Đặt tên cho nhóm rồi Create: ![image](https://hackmd.io/_uploads/Hk5xG9w-C.png)

>Lúc này, 1 nhóm các request đã được tạo ra, ta sẽ liên tục Ctrl + R để thêm các request y hệt vào nhóm. Ví dụ ta sẽ tạo 20 request: ![image](https://hackmd.io/_uploads/HJdBfcDZ0.png)

> Chuyển đổi chế độ của **Send**, từ gửi 1 lần với 1 request, thành gửi song song cả nhóm request cùng 1 lúc: ![image](https://hackmd.io/_uploads/Hk8_fqD-0.png)

>Tiến hành Send, và tải lại trang thì thấy các mã giảm giá đã áp thành công, chứng tỏ server không xử lí trường hợp này: ![image](https://hackmd.io/_uploads/r18zXcDZC.png)

>Do thời gian có thể chênh lệch nhau nên có request thành công, có request lại thất bại khiến cho tổng hóa đơn chưa thể giảm tới mức dưới 50$ để ra mua được. Do đó, có thể thực hiện remove coupon cũ đi và send lại nhiều lần cho tới khi đạt mục đích: 

>Sau 2 lần Send: ![image](https://hackmd.io/_uploads/r1maQcvWR.png)

>Tiếp tục xóa coupon và tăng thêm số lượng request trong nhóm: ![image](https://hackmd.io/_uploads/Sk-yI9wWC.png)

>Như vậy, số tiền cho hóa đơn đã giảm chỉ còn 15$ và ta có thể thực hiện thanh toán bình thường. Mua sản phẩm và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJ6GUcvZ0.png)

# **2. Lab: Bypassing rate limits via race conditions**
>Mô tả cho biết chức năng login trong lab đã chặn tấn công brute-force bằng cách giới hạn tốc độ gửi request, nhưng nó có thể bypass bởi 1 cuộc tấn công race condition. Nhiệm vụ của chúng ta là: tìm cách khai thác race condition để bypass giới hạn tốc độ, thành công tìm được mật khẩu của `carlos`, truy cập vào trang quản trị và xóa người dùng `carlos`:
![image](https://hackmd.io/_uploads/H1lpYjv-A.png)

>Ta sẽ thử dùng brute-force list mật khẩu cho sẵn của `carlos`. Bắt request login và cho vào Intruder, thêm payload cho mật khẩu và chọn Sniper, paste list mật khẩu và attack:
![image](https://hackmd.io/_uploads/Byx_YiPW0.png)

>Nhận thấy với payload `football` thì có phản hồi thành công, chứng tỏ đây là mật khẩu đúng:
![image](https://hackmd.io/_uploads/HyeKYiPZA.png)

>Đăng nhập thành công vào `carlos` và xóa `carlos` để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/SJtrtiP-0.png)

> Tuy nhiên, đây chỉ là may mắn khi mật khẩu ở ngay các dòng đầu nên nó chưa bị chặn!! Thực tế, khi khởi động lại bài lab, tiến hành cách cũ thì web sẽ không cho phép đăng nhập tiếp trong vòng 1 phút khi thử sai 8 lần: ![image](https://hackmd.io/_uploads/HkMtkhwbC.png)

>Mà lab này có giới hạn thời gian là 15 phút, hết giờ đếm ngược sẽ hết hiệu lực, cho nên ta không thể nhập tay để thử tay toàn bộ list mật khẩu được!
List mật khẩu gồm 30 mật khẩu, vì vậy, nếu ta gửi 1 lần toàn bộ 30 request login với các mật khẩu khác nhau thì server sẽ không thể nào check được số lần ta đã thử nhập sai!

> Cho request vào Intruder, đặt payload vào phần `password` và chọn kiểu Sniper, cài đặt payload là list mật khẩu cho trước: ![image](https://hackmd.io/_uploads/ryJP79_ZC.png)

>Cài đặt **"Resource pool"** để Burp gửi cùng lúc 30 request đồng thời: ![image](https://hackmd.io/_uploads/HJDKX9ubA.png)

>Tiến hành Attack: ![image](https://hackmd.io/_uploads/H1nWN9_WA.png)

>Có thể thấy, với payload `12345` thì có respone khác biệt nhất là 302, chứng tỏ đây là mật khẩu đúng! Đăng nhập vào `carlos`, truy cập vào Admin Panel: ![image](https://hackmd.io/_uploads/rk9vE5d-R.png)

>Xóa `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1puVcOWR.png)

# **3. Lab: Multi-endpoint race conditions**
> Mô tả cho biết luồng mua hàng của bài lab chứa 1 lỗ hổng race condition, cho phép ta có thể mua sản phẩm ngoài mức giá mong muốn. Nhiệm vụ của ta là phải mua được thành công "Lightweight L33t Leather Jacket" để hoàn thành bài lab. Cho trước 1 người dùng hợp lệ `wiener:peter`:
![image](https://hackmd.io/_uploads/H1RZFcu-A.png)

>Đăng nhập vào `wiener` và thấy tài khoản có sẵn 100$ , có chức năng nhập Gift card hoàn lại 10$( khi ta mua sản phẩm Gift card sẽ được tặng code này)
>![image](https://hackmd.io/_uploads/B1NN-oO-R.png)

>Thử thêm 1 sản phẩm vào giỏ hàng: ![image](https://hackmd.io/_uploads/Hke2Zi_ZR.png)

>Lúc này, thử đăng xuất và vào lại mục giỏ hàng, sẽ hiện thông báo giỏ hàng trống: ![image](https://hackmd.io/_uploads/By8mMjObC.png)

>Chứng tỏ server sẽ có 1 bước kiểm tra giỏ hàng của từng user!

>Xem lại HTTP History, và thấy có 2 request để tương tác với giỏ hàng, đó là `POST /card` - để thêm sản phẩm vào giỏ hàng, và `POST /cart/checkout` - để thanh toán đơn hàng: ![image](https://hackmd.io/_uploads/rJ7W7jO-0.png)

>Nghi ngờ rằng có thể xuất hiện race condition khi dùng 2 request này, gửi 2 request tới Repeater và nhóm chúng lại theo thứ tự: ![image](https://hackmd.io/_uploads/B1SsmidWC.png)

>Chuyển Send sang chế độ **"Send group (single connection)"** và sau khi gửi, nhận thấy rằng thời gian phản hồi của request `/cart` luôn chậm hơn 1 ít so với `/cart/checkout`! Tại sao?? Là do server còn phải kiểm tra giỏ hàng là của user nào ở request 1 nữa nên mất thêm chút ít thời gian! Vậy câu hỏi ở đây là: trong thời gian chút ít đó, có thể thêm sản phẩm mới vào trong giỏ, và sau khi server kiếm tra xong => thực hiện thanh toán cho giỏ hàng mà không hề hay biết ta vừa thêm sản phẩm mới vào! 

>Đầu tiên, ta sẽ thêm lại 1 gift card vào giỏ hàng: ![image](https://hackmd.io/_uploads/H1fsSj_bR.png)

>Chỉnh sửa request 1 để trỏ tới sản phẩm đề bài cần: ![image](https://hackmd.io/_uploads/Bk2arouWA.png)

> Chuyển thành chế độ Send group (in parallel) để gửi song song đồng thời các request: ![image](https://hackmd.io/_uploads/r1dGUjdZA.png)

>Nếu lần đầu chưa được, có thể thử lại nhiều lần, do thời gian phản hồi của các end point là khác nhau. Sau khi thực hiện 3 lần thì ta đã có thể mua được sản phẩm đề bài yêu cầu và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hyzkvs_W0.png)

# **4. Lab: Single-endpoint race conditions**
>Tính năng thay đổi email của lab này chứa lỗ hổng race condition cho phép bạn liên kết một địa chỉ email tùy ý với tài khoản của mình. Một người nào đó có địa chỉ carlos@ginandjuice.shop nhận được lời mời làm quản trị viên của trang web đang chờ xử lý nhưng họ vẫn chưa tạo tài khoản. Do đó, bất kỳ người dùng nào xác nhận thành công địa chỉ này sẽ tự động kế thừa các đặc quyền của quản trị viên. Nhiệm vụ của ta là thay đổi thành công email thành **"carlos@ginandjuice.shop"**, lấy được quyền quản trị và xóa người dùng `carlos`: 
![image](https://hackmd.io/_uploads/H13cAouZR.png)

>Đăng nhập vào `wiener`, thưc hiện thay đổi email thì có thông báo rằng, muốn thay đổi email thì cần phải truy cập vào đường link được gửi trong mail đó để xác nhận: ![image](https://hackmd.io/_uploads/Sy8912u-A.png)

>Ta sẽ dựa vào ví dụ sau: ![image](https://hackmd.io/_uploads/HJDXendWA.png)

>Tức là, ta sẽ gửi yêu cầu update email tới hòm thư email mà ta sở hữu, theo đúng logic thì link xác nhận sẽ gửi về hòm thư đó. Nhưng nếu ta gửi đồng thời 2 request update email, cái đầu là tới hòm thư của ta, cái sau tới **"carlos@ginandjuice.shop"**. Kết thúc quá trình xử lí, thì server sẽ trả về link xác nhận của mail, lúc này link của request 1 đã bị link của request 2 đè lên, nhưng link xác nhận của request 2 lại được gửi tới hòm thư của request 1!!

>Đầu tiên, nhóm 2 request update email lại và chỉnh sửa giá trị cho request 1 trỏ tới hòm thư của ta: ![image](https://hackmd.io/_uploads/rJSxM3OW0.png)

>Request 2 trỏ tới hòm thư **"carlos@ginandjuice.shop"**: ![image](https://hackmd.io/_uploads/SJefGn_ZR.png)

>Chuyển chế độ Send sang gửi song song: ![image](https://hackmd.io/_uploads/SJrXf2d-A.png)

>Thực hiện Send, quay lại kiểm tra hòm thư mail của ta: ![image](https://hackmd.io/_uploads/S1xLG2OWA.png)

>Thấy rằng link xác nhận đổi email thành **"carlos@ginandjuice.shop"** đã được gửi tới! Truy cập vào link xác nhận và đổi email thành công: ![image](https://hackmd.io/_uploads/r1b5GhObC.png)

>Nâng quyền thành công lên admin và xóa người dùng `carlos` để hoàn thành bài lab:![image](https://hackmd.io/_uploads/SyTnGhdZ0.png)

# **5. Lab: Exploiting time-sensitive vulnerabilities**
>Mô tả cho biết trong lab có chức năng reset password. Chúng ta có thể dựa vào thời gian gửi request để khai thác lỗ hổng. Hoàn thành bài lab bằng cách đăng nhập vào `carlos`, dùng quyền quản trị và xóa `carlos`: 
![image](https://hackmd.io/_uploads/H1T4m3_WA.png)

>Truy cập bài lab, và thấy có tính năng quên mật khẩu: ![image](https://hackmd.io/_uploads/S12vs0Ob0.png)

> Thử lấy lại mật khẩu của `wiener`: ![image](https://hackmd.io/_uploads/ryr9iAO-C.png)

>Một đường dẫn để reset password đã được gửi vào hộp thư: ![image](https://hackmd.io/_uploads/r1FhjCu-0.png)

>Check hộp thư: ![image](https://hackmd.io/_uploads/Syyy3Rub0.png)

>Truy cập link và thay đổi mật khẩu: ![image](https://hackmd.io/_uploads/rk6g2CubA.png)

>Trên URL có chứa `username=wiener`, thử thay đổi nó thành `carlos`: ![image](https://hackmd.io/_uploads/S1QEhRu-A.png)

>Nhưng token không hợp lệ nên cách này fail! Bây giờ xem lại các request đã gửi, thấy rằng request `GET forgot-password` có 1 cookie tạo phiên `phpsessionid` => Server dung back-end là PHP. Điều này có thể có nghĩa là máy chủ chỉ xử lý một yêu cầu tại một thời điểm trong mỗi phiên: ![image](https://hackmd.io/_uploads/ryBHpAd-C.png)

>Gửi yêu cầu `GET forgot-password` tới Repeater, xóa cookie phiên khỏi yêu cầu gửi  để nhận cookie phiên mới và mã thông báo csrf: ![image](https://hackmd.io/_uploads/S15c6CuW0.png)

>Từ phản hồi, sao chép cookie phiên và mã csrf mới được cấp và sử dụng chúng để thay thế các giá trị tương ứng trong một trong hai yêu cầu `POST forgot-password`. Bây giờ ta có một cặp yêu cầu đặt lại mật khẩu từ hai phiên khác nhau!

>Gửi song song hai yêu cầu **POST** từ 2 phiên để cùng yêu cầu lấy lại mật khẩu cho `wiener`:![image](https://hackmd.io/_uploads/Hk020AuW0.png)

>Kiểm tra hộp thư thì thấy 2 request được gửi tới cùng 1 lúc: ![image](https://hackmd.io/_uploads/SJeg1kt-C.png)

> Để ý kĩ thì phần token được đính kèm của 2 request là giống hệt nhau: ![image](https://hackmd.io/_uploads/rJ_Vk1FWC.png)

> => Yếu tố thời gian phải là 1 yếu tố trong input của token! Vậy nếu ta gửi song song 2 yêu cầu như hồi nãy, nhưng lần này 1 request cho `wiener`, 1 request cho `carlos` thì token lấy được có phải là y hệt nhau? Và ta có thể dùng token này để thay vào và đổi được mật khẩu cho `carlos!`

>Thực hiện gửi đồng thời 2 request như đã nói ở trên, kiếm tra thư thì thấy giờ chỉ còn 1 thư gửi tới: ![image](https://hackmd.io/_uploads/By2Ge1YWR.png)

>Lúc này, 2 token của `wiener` và `carlos` đã giống nhau, ta chỉ việc đổi giá trị của username trong URL: ![image](https://hackmd.io/_uploads/B12wxyF-0.png)

>Đổi mật khẩu của `carlos` và đăng nhập thành công: ![image](https://hackmd.io/_uploads/r1Jsx1FWA.png)

>Truy cập trang quản trị: ![image](https://hackmd.io/_uploads/rJKhlJK-0.png)

>Xóa người dùng `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkfAg1tb0.png)
