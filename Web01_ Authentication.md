# Số bài giải / Tổng số bài: 14/14
#  **1. Lab: Username enumeration via different responses**
> Đề bài cho ta 2 list tài khoản và mật khẩu, yêu cầu đi tìm tài khoản hợp lệ, login vào đó và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B1uukE-lA.png)

> Vào trang web ta thấy giao diện của trang login, ta dùng burp bắt request khi sự kiện login được gửi đi và cho vào intruder để tiến hành brute-force.

![image](https://hackmd.io/_uploads/S1cqCRsyA.png)

> Đề bài cho 2 list tài khoản và mật khẩu để brute-force, mỗi list gồm 100 giá trị. Nếu dùng brute-force cho cả 2 trường cùng một lúc thì phải tốn tầm 100 * 100 = 10000 giá trị, như thế sẽ rất lâu và không hiệu quả. Thay vào đó, ta sẽ chỉ tấn công vào trường username thôi, và sẽ đặt mật khẩu tĩnh(ví dụ để '123'), và start attack, vì chỉ có tài khoản đúng (đã tồn tại) thì mới trả về giá trị **"Invalid password"** nên ta sẽ lọc ra được payload có độ dài khác với tất cả những payload còn lại: 

![image](https://hackmd.io/_uploads/Sysoe13yC.png)

![image](https://hackmd.io/_uploads/HJAw413yR.png)



> Lúc này đã có được tài khoản rồi thì ta chỉ việc thử với list 100 mật khẩu kia thôi: 

![image](https://hackmd.io/_uploads/SJEKVJhyA.png)


> Lọc ra được mật khẩu có mã trạng thái và độ dài trả về là khác biệt nhất là **'computer'**:

![image](https://hackmd.io/_uploads/ByXbEkn1C.png)

> Nhập tài khoản mật khẩu và hoàn thành bài lab:

![image](https://hackmd.io/_uploads/rJ5sN13JC.png)

 # **2. Lab: Username enumeration via subtly different responses**
![image](https://hackmd.io/_uploads/B1-dDJ3kR.png)


> Tiếp tục là 1 bài tấn công vào phần login như ở trên, ta thử cách cũ, bật intercept và bắt request login. Ta lại đi tìm username đã tồn tại trong hệ thống bằng cách quan sát các mã respone trả về: 
> 
![image](https://hackmd.io/_uploads/HkOIdy31R.png)

> Nhưng lần này ta không khai thác được điều gì vì respone trả về đều mã 200 và cũng không có Length nào là khác biệt. Ta sẽ dùng cách "trâu bò" để tấn công vào 2 trường username và password (Mỗi user name sẽ thử với 100 password). Sử dụng kiểu tấn công Cluster Bomb để attack: ![image](https://hackmd.io/_uploads/Hk1Ifgh1C.png)


> Nhập vào payload 1 và 2 là list usrename và password: ![image](https://hackmd.io/_uploads/HJy9fxnkA.png)

>Sau 5p chờ đợi thì tiến hành lọc tìm status code khác biệt và ta đã tìm được username và mật khẩu đúng: ![image](https://hackmd.io/_uploads/HJmf7lnyA.png)
 
> Submit và solve bài: ![image](https://hackmd.io/_uploads/SJSNQxn1R.png)

# **3. Lab: 2FA simple bypass**
> Đề bài cho ta tài khoản và mật khẩu của mình và victim: ![image](https://hackmd.io/_uploads/rJ0HW1ayA.png)

> Dùng tài khoản của "wiener" đăng nhập vào thì sẽ được chuyển hướng tới trang **/login2** và thông báo phải nhập mã xác thực 4 số được gửi qua email mới có thể login thành công: ![image](https://hackmd.io/_uploads/HyGHV1pkC.png)

> Ta có thể suy rằng khi nhập đúng tài khoản mật khẩu thì đã vượt qua bước xác minh thứ nhất và đang trong trạng thái log-in chưa hoàn chỉnh..

> Vào hộp thư của "wiener" để lấy mã và vượt qua bước login: ![image](https://hackmd.io/_uploads/Bk9fXyaJR.png)

> Sau khi đăng nhập thành công thì ta sẽ vào được trang cá nhân, và để ý trên URL thấy hệ thống sẽ hiển thị id của người dùng hiện tại: ![image](https://hackmd.io/_uploads/rksbBJTyC.png)

> Điều này có nghĩa, chỉ khi người dùng vượt qua xác minh 2 bước thì mới hiển thị đường link này. Vậy nếu ta chỉ đang ở bước xác minh thứ nhất mà truyền vào tham số "id" ứng với người dùng thì có thể bypass bước 2 không?

> Tiếp theo, ta sẽ đăng nhập với tài khoản "carlos" để thử nghiệm suy luận trên, khi ở bước nhập mã xác minh thì thay đổi URL để chuyển tới trang cá nhân: **"/my-account?id=carlos"**: ![image](https://hackmd.io/_uploads/rJCKdyak0.png)

> Enter và thực sự trang web đã thực sự không kiểm tra bước xác minh thứ 2 trước khi tải lại trang. => Đến được trang cá nhân của "carlosd" và solved bài lab. ![image](https://hackmd.io/_uploads/B1Ww51ayC.png)

# **4. Lab: 2FA broken logic**
 > Lần này thì đề bài không cho mật khẩu của victim nữa, nên cách thay tham số trong URL là vô nghĩa khi không vượt qua được bước xác minh 1: ![image](https://hackmd.io/_uploads/HJWdp1TJA.png)
 
 > Ta sẽ thử dùng Burp bắt để quan sát các request, và 1 điều khá thú vị là khi người dùng gửi mã code ở trang **"/login2"** thì request sẽ đính kèm mã xác minh và  1  http header **Cookie** với tham số **"verify"** ứng với tên người dùng đó. Lúc này ta nghĩ ngay tới việc đổi tham số **verify** ứng với tên nạn nhân và mã code có thể brute-force được, cách làm này rất khả quan: ![image](https://hackmd.io/_uploads/SJtaJe6yR.png)

> Đầu tiên phải tạo code cho tài khoản carlos trước đã, bắt request "GET /login2", chuyển tới Repeater sửa lại tham số cho "verify" và Send: ![image](https://hackmd.io/_uploads/Sytk4gaJA.png)



> Chuyển request **"POST /login2"** sang Intruder và thiết lập payload cho trường "mta-code" để chạy từ 0001 tới 9999, thay đổi tham số cho verify và attack: ![image](https://hackmd.io/_uploads/BJMMMeayR.png)


> Lọc ra lấy respone 302 là code đúng, chuột phải vào và "Show respone in browser": ![image](https://hackmd.io/_uploads/SyCYrxa1C.png)

> Copy link và dán, enter và ta đã solved lab: ![image](https://hackmd.io/_uploads/SyYULx6yR.png)

# **5. Lab: Brute-forcing a stay-logged-in cookie**
> Đề bài cho tài khoản mật khẩu, ta sẽ thử đăng nhập vào xem có gì: ![image](https://hackmd.io/_uploads/B1xMYga10.png)

>Có hiển thị id người dùng trong phần URL khi đăng nhập thành công: ![image](https://hackmd.io/_uploads/BkRTtl6kA.png)

> Khi cố thay đổi id này sang victim thì fail và nó sẽ chuyển hướng tới trang login. Trong trang login có thêm trường **"Stay logged in"**, nghĩa là khi người dùng đăng nhập thì sẽ tạo 1 cookie để lần sau người dùng vào lại trang web vẫn trong trạng thái "Đã đăng nhập": ![image](https://hackmd.io/_uploads/H1vfieTJ0.png)

> Xem Cookie trong trang web thì ta thấy 2 trường là "session" và "stay-logged-in": ![image](https://hackmd.io/_uploads/HJ3YngTkA.png)

>Sau khi thử đăng xuất và đăng nhập lại, giá trị **session** đã thay đổi nhưng **stay-logged-in** vẫn như vậy không thay đổi, kể cả khi người dùng tắt trang web thì giá trị này vẫn sẽ không bị destroy. Vậy ta sẽ đi tấn công vào giá trị này :D Dùng tool để check xem dòng **"d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw"** được mã hóa theo thuật toán gì. Phát hiện nó tỉ lệ cao được mã hóa theo Base64: ![image](https://hackmd.io/_uploads/S1DHAx6JA.png)


>Tiến hành mã hóa nó và nhận được:  ![image](https://hackmd.io/_uploads/r1ecBJ-TkR.png)

> Ra là cookie được lưu trữ dưới dạng "id:dòng mã hóa". Lại đi phân tích tiếp dòng mã hóa "51dc30ddc473d43a6011e9ebba6ca770" là gì: ![image](https://hackmd.io/_uploads/HyrWgZp1A.png)

> Biết được nó sử dụng hàm băm MD5, lại thử phá mã và nhận được kết quả đó là mật khẩu của người dùng: ![image](https://hackmd.io/_uploads/Hyorx-TkR.png)

> Đi đến kết luận, 1 cookie **stay-logged-in** được tạo ra với cấu trúc như sau:  **base64(username+':'+md5(password))** => Ta đã có tài khoản victim là "carlos" và list mật khẩu. Vì vậy việc brute-force cookie này là rất khả thi!! Vào burp và bắt request khi login thành công vào Intruder, thiết lập payload cho trường **"Cookie: stay-logged-in"**: ![image](https://hackmd.io/_uploads/S1s4rqA1A.png)


> Và thêm list mật khẩu đã cho vào danh sách, đồng thời thêm các rules để áp dụng mã hóa cho payload khi request được gửi (sao cho đúng với cấu trúc ở trên): ![image](https://hackmd.io/_uploads/HJlLr9AkR.png)

> Attack theo kiểu Sniper, và nhận được 1 payload trả về kết quả đúng với respone mã 200: ![image](https://hackmd.io/_uploads/HJ8jHqRyR.png)

> Phân tích cookie đúng này và ta tìm được mật khẩu của victim: ![image](https://hackmd.io/_uploads/HkUXIqCy0.png)

> Đăng nhập vào tài khoản victim và solved bài lab: ![image](https://hackmd.io/_uploads/HyFBUqCyA.png)


# **6. Lab:  Offline password cracking**
> Đề bài cho ta biết trong lab có chứa lỗ hổng XSS trong phần comment và account của "wiener". Yêu cầu ta lấy được cookie của victim và hoàn thành bài lab bằng cách xóa tài khoản của victim: ![image](https://hackmd.io/_uploads/rkO5CqC1A.png)

>Login vào tài khoản đã cho trước, phát hiện rằng cookie "Stay-logged-in" cũng được lưu trữ với cấu trúc như bài trước:  **base64(username+':'+md5(password))**: ![image](https://hackmd.io/_uploads/rJXGWiRJR.png)
![image](https://hackmd.io/_uploads/H1YUZjRJC.png)

>Trong lab có chứa máy chủ để xem thông tin các log  và địa chỉ máy chủ: ![image](https://hackmd.io/_uploads/SyxkGjC1A.png)

>![image](https://hackmd.io/_uploads/BkB7gs0JA.png)

> Vào phần comment của mỗi post, ta test phần này có bị dính XSS không:
![image](https://hackmd.io/_uploads/S1D5yjA10.png)

> Và xác nhận được rằng ta có thể khai thác cookie ở chỗ này sử dụng XSS:
![image](https://hackmd.io/_uploads/H14sksRJC.png)

> Lúc này, ta sẽ thử tiếp để lấy cookie xem có khả thi không: ![image](https://hackmd.io/_uploads/rkxNXiA1R.png)

>Gần như đã chắc chắn rằng trang web này không dùng cờ HttpOnly để chống việc trộm cookie. Tiếp theo ta dùng "document.location" trong JS để trỏ tới máy chủ expolit tiến hành khai thác cookie: ![image](https://hackmd.io/_uploads/HkOWNsCJC.png)

>Sau khi post comment, chuyển sang kiểm tra log bên phía máy chủ, phát hiển phần cookie đã biết lấy cắp: ![image](https://hackmd.io/_uploads/B1epVs010.png)

>Cuối cùng, ta chỉ việc lấy giá trị "stay-logged-in" đi giải mã và lấy được mật khẩu của victim: ![image](https://hackmd.io/_uploads/HJ7EHo01C.png)

>Phá mã MD5 để lấy mật khẩu: ![image](https://hackmd.io/_uploads/SJKFBjAkC.png)

> Cuối cùng đăng nhập vào victim và xóa tài khoản này để solved bài lab: ![image](https://hackmd.io/_uploads/HykyLs0JC.png)
![image](https://hackmd.io/_uploads/BkQeIj0JC.png)
![image](https://hackmd.io/_uploads/HyHWLsRJ0.png)

# **7. Lab: Password reset broken logic**

> Đề bài cho ta account sẵn, và tài khoản của victim, và mục tiêu là dựa vào lỗ hổng trong phần reset mật khẩu để reset được mật khẩu nạn nhân và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B1D78exlA.png)

> Vào bài lab, ta thấy chức năng Forgot Password: ![image](https://hackmd.io/_uploads/HkvjdxegR.png)

>Thử reset mật khẩu của tài khoản có sẵn: ![image](https://hackmd.io/_uploads/SkkbFexxR.png)

>Thông báo đã gửi 1 link để reset password về hòm thư email: ![image](https://hackmd.io/_uploads/SJn7YexgA.png)

>Vào check mail và click vào link: ![image](https://hackmd.io/_uploads/r1-IKxxeR.png)

> Lúc này, sẽ hiện 1 form để ta lấy lại mật khẩu: ![image](https://hackmd.io/_uploads/Bk55FlxgR.png)

>Đặt lại mật khẩu tùy ý mình, và vào  HTTP History của Burp để kiểm tra xem có những request gì đã được gửi đi...

>Phát hiện rằng khi submit mật khẩu mới, một request **"POST /forgot-password?temp-forgot-password-token"** được gửi đi, kèm theo đó là username, mật khẩu mới và token, thử nghiên cứu 1 chút về request này, gửi đến Repeater: 
![image](https://hackmd.io/_uploads/HJLO9ggeC.png)

>Xóa hết các trường "token" trong request này và gửi đi thì phát hiện mặc dù thiếu "token" nhưng server vẫn trả về kết quả **"200 OK"**, chứng tỏ nó vẫn hoạt động: ![image](https://hackmd.io/_uploads/rJIojgxxA.png)

> Kết luận rằng server sẽ không kiểm tra token khi submit mật khẩu mới, nên hướng đi bây giờ của ta sẽ là đổi username trong phần request thành "carlos" và xóa phần tham số token truyền đi và reset lại password của victim: ![image](https://hackmd.io/_uploads/BkycRglxR.png)

>Server trả về respone **"302 Found"**, điều này có nghĩa suy luận của ta là đúng, và đã reset được mật khẩu của victim thành "123": ![image](https://hackmd.io/_uploads/Hyr7JbgeR.png)

>Quay lại trang login và solved bài lab: ![image](https://hackmd.io/_uploads/BylHJWxeR.png)

#  **8. Lab:  Password reset poisoning via middleware**
>  Đề bài cho biết rằng người dùng carlos sẽ bất cẩn click vào link được gửi tới Email client của anh ấy. Để giải quyết bài này, chúng ta cần khai thác lỗ hổng trong tính năng đặt lại mật khẩu để lấy được token đặt lại mật khẩu của victim carlos. Hệ thống cung cấp một exploit server dùng để lấy các dữ liệu thông tin từ victim.
![image](https://hackmd.io/_uploads/SJdoyVeeA.png)

> Cũng như bài **"Password reset broken logic"**, ta sẽ sử dụng chức năng Forgot Password và 1 đường dẫn URL để đặt lại mật khẩu được gửi đến trong hòm thư: ![image](https://hackmd.io/_uploads/Hy43eHxg0.png)

>Thực hiện đổi mật khẩu và quan sát bên trong Request khi submit đặt lại mật khẩu: 
![image](https://hackmd.io/_uploads/HkugMrgl0.png)

>Rõ ràng là bây giờ đã không còn trường "username" để mình thay đổi thành username của victim để tấn công nữa. Vậy bây giờ ta phải thay đổi phương thức... Sau khi lục tìm lại và xem hết các request thì thấy 1 request có chứa trường "username" được gửi khi người dùng yêu cầu lấy lại mật khẩu: ![image](https://hackmd.io/_uploads/rkm8UBll0.png)

>Ta thử thay "username" thành "carlos" và Send thì nhận được respone 200: ![image](https://hackmd.io/_uploads/rJE6UBle0.png)

>Theo đúng kịch bản, "carlos" sẽ click vào link đặt lại mật khẩu đó và “forward” tới exploit server của chúng ta. Như vậy, vấn đề cần giải quyết ở đây là làm sao khi carlos click vào đường dẫn đó, exploit server của chúng ta sẽ nhận được dữ liệu của anh ấy? Sau 1 hồi tra mạng thì ta có HttpHeader **"X-Forwarded-Host"**. 
>Thông thường, khi một yêu cầu HTTP đi qua nhiều máy chủ trung gian, như các proxy hoặc load balancer, các máy chủ trung gian này thường sẽ thêm các header như "X-Forwarded-For", "X-Forwarded-Proto", và "X-Forwarded-Host" để chuyển tiếp thông tin về yêu cầu gốc. Trong trường hợp của **"X-Forwarded-Host"**, nó chứa tên miền hoặc địa chỉ IP của máy chủ gốc mà yêu cầu đã được gửi đến trước khi đến máy chủ hiện tại.


>Do đó, ta sẽ lấy URL của máy chủ exploit và thêm vào header: ![image](https://hackmd.io/_uploads/ryAwdrxgR.png)

> Cho request vào Repeater và thêm header, và Send nhận được respone 200: ![image](https://hackmd.io/_uploads/rypMKrxeC.png)

> Vào access log và kiểm tra thì phát hiện token của "carlos" đã thành công chuyển hướng tới exploit server của ta: ![image](https://hackmd.io/_uploads/BJ7hFrgeA.png)

>Và việc còn lại chỉ là thay token của victim vào request "POST /forogt-password" ở trên và set mật khẩu mới là "123". Bùm và ta có respone trả về thành công: ![image](https://hackmd.io/_uploads/B1RdqSggC.png)
 
> Cuối cùng là đăng nhập vào tài khoản "carlos" và solved bài lab: ![image](https://hackmd.io/_uploads/B10T5reeR.png)

# **9. Lab: Password brute-force via password change**
> Chúng ta được cung cấp 1 tài khoản hợp lệ wiener:peter, username victim và 1 danh sách passwords chứa password đúng của carlos: ![image](https://hackmd.io/_uploads/HJFmzIxeC.png)

>Đăng nhập tài khoản **wiener:peter** và thử đổi mật khẩu, nếu current password nhập đúng, password mới và password confirm không giống nhau thì hệ thống trả về thông báo **New passwords do not match**: ![image](https://hackmd.io/_uploads/rJPqmIxxA.png)

>Nếu current password nhập sai, password mới và password confirm giống nhau thì hệ thống trả về trang login.

> Nếu current password nhập sai, password mới và password confirm không giống nhau thì hệ thống trả về thông báo **Current password is incorrect**: ![image](https://hackmd.io/_uploads/ry8pXLlgC.png)

> => Như vậy ta có thể cố tình nhập password mới và password confirm không giống nhau, sử dụng tham số current password thực hiện tấn công Brute force với danh sách password được cung cấp.

> Bắt request login và chuyển sang Intruder, tấn công vào tham số Current password, và cho 2 giá trị mật khẩu mới khác nhau: ![image](https://hackmd.io/_uploads/H15rVLegR.png)

> Trong phần Options, thêm luật để lọc ra những respone có chứa **"New passwords do not match"**: ![image](https://hackmd.io/_uploads/HJlTNLllC.png)
> ![image](https://hackmd.io/_uploads/HJA04Lgl0.png)

> Thực hiện tấn công và bắt được mật khẩu đúng: ![image](https://hackmd.io/_uploads/ByMWrUglC.png)

> Cuối cùng là login vào tài khoản victim để solved bài lab: ![image](https://hackmd.io/_uploads/S1dXrIglR.png)

# **10. Lab: Username enumeration via response timing**
> Đề bài cung cấp cho ta tài khoản hợp lệ wiener:peter và 2 danh sách tài khoản và mật khẩu. Nhiệm vụ của chúng ta là sử dụng kĩ thuật tấn công vét cạn tìm kiếm tên đăng nhập và mật khẩu người dùng, truy cập vào tài khoản của victim. Ngoài ra còn có gợi ý: chúng ta cần vượt qua cơ chế bảo vệ chống brute-force ở bài này: 
![image](https://hackmd.io/_uploads/SkYp7Pgl0.png)

> Nếu đã có 2 danh sách tài khoản và mật khẩu, đầu tiên ta thử cho request login vào và tiến hành brute-force: ![image](https://hackmd.io/_uploads/rkbVSwlxC.png)

>Nhưng sau 5 lần thử sai liên tiếp thì ta đã bị chặn và yêu cầu đăng nhập lại sau 30 phút: ![image](https://hackmd.io/_uploads/rJH_HDge0.png)

>Bởi dấu hiệu có thể đăng nhập lại sau 30 phút, từ đó suy đoán rằng hệ thống thực hiện block địa chỉ IP của chúng ta trong 30 phút. Vì vậy ta sẽ tìm cách để bypass việc server chặn IP của ta. Tìm kiếm trên Google header để thực hiện việc này và ta có được header "X-Forwarded-For: ![image](https://hackmd.io/_uploads/BkZDUPexA.png)

>Thử thêm header này với IP ngẫu nhiên và quả nhiên đã bypass được: ![image](https://hackmd.io/_uploads/HJysIvlx0.png)

> Như vậy chúng ta có thể thay đổi IP với từng request trong tấn công brute force để bypass cơ chế block IP của hệ thống.

>  Vì mật khẩu được lưu trong cơ sở dữ liệu thì thường được mã hóa dưới dạng hàm băm, và khi người dùng nhập mật khẩu thì server sẽ thực hiện mã hóa mật khẩu vừa nhập và so sánh với mật khẩu được lưu trong cơ sở dữ liệu để xác thực người dùng. Và thời gian thực hiện việc này cũng khác nhau đối với các loại mật khẩu ngắn hoặc dài. Dựa vào cơ chế này thì  để tìm được username của victim, ta sẽ dựa vào thời gian phản hồi của server để xác định đúng hay sai. 

>Thử đăng nhập với một username không tồn tại và một password rất dài, chúng ta nhận được phản hồi ngay lập tức: ![image](https://hackmd.io/_uploads/BJ96dwegR.png)

>Đăng nhập với username wiener và password rất dài, response được trả về chậm hơn (tầm 5-6 giây): ![image](https://hackmd.io/_uploads/BJ7-KDllC.png)

> Từ sự khác biệt về thời gian phản hồi này,  ta có thể suy rằng hệ thống thực hiện kiểm tra username trước, nếu không tồn tại username trong cơ sở dữ liệu sẽ ngay lập tức trả về đăng nhập thất bại, nếu username tồn tại (ở đây là wiener), hệ thống thực hiện kiểm tra password và vì ta sử dụng một password rất dài nên dẫn tới thời gian xử lý lâu hơn ở cơ chế mã hóa, dẫn đến thời gian phản hồi chậm hơn.

> => Ta sẽ thực hiện brute-force kết hợp thay đổi địa chỉ IP cho mỗi username, sử dụng password dài mặc định cho mỗi lần thử và kiểu tấn công PitchFork: ![image](https://hackmd.io/_uploads/H1WN5DlxC.png)

>Và thiết lập cho dải IP chạy từ 123.12.1.0 tới 123.12.1.100: ![image](https://hackmd.io/_uploads/SkvI9veg0.png)

>Tiến hành attack và phát hiện tài khoản "apache" có thời gian phản hồi rất lâu, chứng tỏ đây là tài khoản đã tồn tại: ![image](https://hackmd.io/_uploads/Byx5LiPeeA.png)

> Đương nhiên tìm được tài khoản rồi thì việc brute-force ra mật khẩu rất dễ dàng vì đã có list mật khẩu. Thay đổi payload trong request nhắm vào trường password: ![image](https://hackmd.io/_uploads/HJKb2vexR.png)

> Và lọc ra được mật khẩu đúng với respone 302 khác biệt: ![image](https://hackmd.io/_uploads/HkpQ3wegC.png)

> Cuối cùng chỉ việc login vào tài khoản nạn nhân và solved bài lab: ![image](https://hackmd.io/_uploads/r1eBK3vlgR.png)

# **11. Lab: Broken brute-force protection, IP block**

>Tiếp tục đề bài cho biết rằng lab này chứa lỗi logic trong việc chống tấn công brute-force. Cho sẵn 1 tài khoản hợp lệ "wiener:peter" và nhiệm vụ là dựa vào list mật khẩu để đăng nhập vào tài khoản victim: 
![image](https://hackmd.io/_uploads/rJuOUMZeC.png)

>Cũng như bài trước thì ta cũng sẽ bắt request login và cho vào Repeater , thử đăng nhập với tài khoản đã có và mật khẩu sai, sẽ báo về **"Incorrect password"**:![image](https://hackmd.io/_uploads/ry50wzbeC.png)

>Tuy nhiên nếu ta thử sai 3 lần, đến lần thứ 4 thì sẽ bị chặn đăng nhập trong 1 phút: 
![image](https://hackmd.io/_uploads/BJfr_Mbe0.png)

>Đoán rằng bài này cũng có thể bypass cơ chế block IP như bài trước, ta sẽ thêm header **"X-Forwarded-For"** nhưng kết quả nó không khả quan trong bài này: 
![image](https://hackmd.io/_uploads/HJ86dfZeC.png)

>Tuy nhiên nếu ở lần thứ 3 chúng ta đăng nhập với tài khoản hợp lệ **`wiener:peter`** thì cơ chế này được làm mới lại. => Tức là chúng ta có thể thực hiện brute-force username, password với quy tắc: sau mỗi 2 lượt thực hiện brute-force, sẽ thực hiện 1 lần đăng nhập hợp lệ bằng tài khoản **`wiener:peter`** (sử dụng kiểu Attack: Pitchfork).

>Ta sẽ sử dụng list payload sau cho trường `username`
```
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
......
```
>Và list payload cho trường `password`:

```
123456
password
peter
12345678
qwerty
peter
....
```
> Bắt đầu tấn công, quan sát được mật khẩu đúng cho `carlos` có respone 302 là `thunder`: ![image](https://hackmd.io/_uploads/HkhTwmWeC.png)

>Cuối cùng là login vào tài khoản victim và solved bài lab: ![image](https://hackmd.io/_uploads/HJqx_mWxR.png)

# **12. Lab: Username enumeration via account lock**
>Đề lần này không cho ta tài khoản hợp lệ nữa mà chỉ cung cấp 2 danh sách tài khoản và mật khẩu, yêu cầu sử dụng brute-force để tìm được username đang tồn tại và đăng nhập vào account đó để hoàn thành: ![image](https://hackmd.io/_uploads/Byv_F7WeC.png)


>Bắt request `/login` và cho vào Repeater và thử nhập sai nhiều lần và không hề nhận được bất kì 1 hành động ngăn chặn nào như block IP... Ta nghĩ ngay tới việc: đề bài đã cho sẵn list tài khoản và mật khẩu thế kia, mỗi list 100 giá trị thì sao không brute-forte tới chết, cũng chỉ có 100 * 100 = 10,000 giá trị thôi mà. Bắt tay vào việc, cho request login vào Intruder và dùng kiểu Attack Cluster Bomb cho 2 trường username và password: ![image](https://hackmd.io/_uploads/BkzYn7-lA.png)

>Thực hiện tấn công và quan sát thấy giá trị `as:zxcvbn` có Length duy nhất khác với các giá trị còn lại: ![image](https://hackmd.io/_uploads/BkvR27WxC.png)

>Cuối cùng là đăng nhập vào tài khoản victim và solved bài lab: ![image](https://hackmd.io/_uploads/r1eZ6Q-eA.png)

# **13. Lab: Broken brute-force protection, multiple credentials per request**
>Đề mô tả rằng bài lab này có chứa lỗi logic trong cơ chế bảo vệ chống việc bị brute-force. Nhiệm vụ của chúng ta là phải vượt qua được cơ chế bảo vệ này, brute-force được mật khẩu của `carlos` dựa trên list mật khẩu đã có sẵn:
![image](https://hackmd.io/_uploads/Ski08dGgA.png)

>Truy cập bài lab, vào giao diện đăng nhập: ![image](https://hackmd.io/_uploads/SycgOdfx0.png)

>Thử login với mật khẩu ngẫu nhiên, quan sát request được gửi đi: ![image](https://hackmd.io/_uploads/rkiXd_zlC.png)

>Điều khác biệt ở đây so với các lab trước là các giá trị `username` và `password` được gửi đi được đặt trong cặp ngoặc nhọn {}. Điều này chứng tỏ khi gửi form đi thì sẽ không sử dụng `type=submit` nữa mà sử dụng code JSON để xử lí. Xem bên trong form: ![image](https://hackmd.io/_uploads/rkheFOzl0.png)

>Do giá trị trong cặp`value` của JSON chấp nhận các kiểu dữ liệu khác nhau, thử thay đổi kiểu dữ liệu khác của `password` thành dạng mảng và gửi đi:![image](https://hackmd.io/_uploads/ryohY_MeA.png)

>Không thấy server trả về kết quả lỗi định dạng, suy luận rằng server chấp nhận kiểu dữ liệu mảng của password mà không chuyển đổi về dạng string. Như vậy, có thể thực hiện gửi tất cả password theo định dạng mảng tới hệ thống. 

>Từ list mật khẩu cho sẵn, tiến hành chuyển đổi toàn bộ sang định dạng mảng trong JSON (Có thể dùng ChatGPT cho nhanh): 
>![image](https://hackmd.io/_uploads/rJ3A5ufgR.png)

>Sau đó paste mảng này vào trường `password` và send request: ![image](https://hackmd.io/_uploads/ByBBsuzgR.png)

> Respone trả về là **"302 Found"**, chứng tỏ ta đã thành công! Cuối cùng chỉ việc chuột phải chọn **"Show respone in browser"** để solved bài lab: ![image](https://hackmd.io/_uploads/H1Jx2dzgA.png)

# **14. Lab: 2FA bypass using a brute-force attack**
>Đề bài cho biết rằng cơ chế xác thực 2FA trong lab tồn tại lỗ hổng có thể brute-force. Họ cung cấp cho ta tài khoản và mật khẩu hợp lệ `carlos:montoya` nhưng không được quyền lấy mã code 2FA. Nhiệm vụ của bài này là phải brute-force được code 2FA và truy cập vào tài khoản `carlos`
![image](https://hackmd.io/_uploads/H18CndfeA.png)

>Đăng nhập vào tài khoản của `carlos`, đến bước xác thực 2FA thì ta không thể có thêm bất cứ thông tin gì về cách lấy mã code này: ![image](https://hackmd.io/_uploads/SyZW1tGeR.png)

>Thử nhập sai mã code này, thông báo hiển thị: ![image](https://hackmd.io/_uploads/B154yYGxA.png)

>Nhưng đến lần thử sai thứ 2, web sẽ chuyển hướng mình thẳng tới trang login: ![image](https://hackmd.io/_uploads/rkNPkFGxA.png)

>Thử cho request gửi mã vào Repeater và Send: ![image](https://hackmd.io/_uploads/ryKtlKzlC.png)

> Respone trả về **"Invalid CSRF token (session does not contain a CSRF token)"**, chứng tỏ sau 2 lần liên tiếp nhập mã sai thì server sẽ kết thúc luôn phiên đăng nhập của người dùng và trở về trang login.
> Quan sát thứ tự các request được gửi đi:  ![image](https://hackmd.io/_uploads/r1nDztMx0.png)
 
> Vậy làm thế nào để ta có thể bypass được cơ chế này và brute-force được code 2FA gồm 4 số đây? Ta sẽ cần có cách để sau tự động đăng nhập lại sau mỗi 2 lần thử sai và tiếp tục brute-force code. Trong Burp có cơ chế **"Burp's session handling features"** sẽ giúp ta thực hiện tự động việc này.

>Đầu tiên, thêm một hành vi **Session handling rule editor** mới:
![image](https://hackmd.io/_uploads/HJeqrtGgR.png)

>Ở mục **Detail**, thêm **Rule actions** bằng cách **Add** -> **Run a macro**
![image](https://hackmd.io/_uploads/r1zXSFfgR.png)

>Chọn các request cần thực hiện theo thứ tự:![image](https://hackmd.io/_uploads/HkefvtGeA.png)

>Lưu macro, tiếp theo, chuyển request `/login2` tới Intruder để thực hiện brute-force. Đặt mục tiêu tấn công là trường `mfa-code` và chọn Sniper. Thiết lập payload chạy từ 0000-9999 và Attack:
![image](https://hackmd.io/_uploads/HJ_QfgXe0.png)

>Lọc ra đươc request có respone trả về là **"302 Found"**. Cuối cùng chuột phải chọn **"Show respone in browser"** và paste vào web để solved bài lab: ![image](https://hackmd.io/_uploads/rJHDzl7lA.png)
