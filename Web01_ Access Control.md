 # Số bài giải / Tổng số bài: 13/13

# **1. Lab: Unprotected admin functionality**
> Đề bài cho ta biết trang này có trang admin không được bảo vệ, và nhiệm vụ của ta là truy cập vào trang quản trị và xóa tài khoản `carlos` để hoàn thành: ![image](https://hackmd.io/_uploads/BkoFr4WlR.png)

> Một trong những việc làm đầu tiên khi thực hiện Pentest một trang web là tìm kiếm các thông tin hữu ích liên quan tới mục tiêu. Và một trong những tệp tin thường được đọc đầu tiên là `robots.txt` (Khi thực hiện quét các đường dẫn cũng có tệp tin này). Tệp `robots.txt` chủ yếu dùng để quản lý lưu lượng truy cập của trình thu thập dữ liệu vào trang web và thường dùng để ẩn một tệp khỏi Google. Đôi khi nó cũng chứa một số thông tin hữu ích.

>Thử truy cập vào tệp `robots.txt` và nhận đưược: ![image](https://hackmd.io/_uploads/Hkg384ZxR.png)

>Điều này có nghĩa là trang web hạn chế các bot crawl tới link nhạy cảm `/administrator-panel` – là đường dẫn tới trang quản trị admin.

> Tiếp tục sửa URL bằng cách thêm `/administrator-panel` vào URL và trang web đã chuyển hướng tới trang quản trị người dùng. Rõ ràng trang web này không thực hiện cài đặt quyền tốt tới từng vai trò của người dùng, dẫn đến bất kì ai đều có quyền truy cập vào đường dẫn này và thực hiện vai trò quản trị!![image](https://hackmd.io/_uploads/HygSDEbgA.png)

>Cuối cùng là thực hiện xóa tài khoản `carlos` và solved bài lab: ![image](https://hackmd.io/_uploads/S1ROv4-lC.png)

# **2. Lab: Unprotected admin functionality with unpredictable URL**
> Đề bài cho biết chức năng quản trị không được bảo vệ tốt. Nó nằm ở một vị trí không thể đoán trước, nhưng vị trí đó được tiết lộ ở đâu đó trong ứng dụng. Nhiệm vụ của ta là tìm được vị trí trang quản trị và xóa tài khoản `carlos` để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJbjFVbeC.png)

>Truy cập vào lab, ta thấy có chức năng login nhưng không hề được cung cấp 1 chút thông tin gì về tài khoản, nên mục tiêu sẽ không phải là ở trong này: ![image](https://hackmd.io/_uploads/Bksk9V-x0.png)

>Tiếp đến, vào thư mục `/home`, và chỉ có chức năng xem chi tiết các bài viết: ![image](https://hackmd.io/_uploads/H1T75VbgR.png)

> Vào từng bài thì cũng không có gì để khai thác, cũng không có phần comment gì...![image](https://hackmd.io/_uploads/HkM89V-eC.png)

>Xem ở phần Front-End không khai thác được gì thì ta sẽ cho vào Burp để xem các request: 
![image](https://hackmd.io/_uploads/S1hY9VbxR.png)

>Và phát hiện khi xem chi tiết 1 bài nào đó, thì đoạn code Javascript hoạt động để lộ đường dẫn tới trang quản trị là `/admin-kzkw3z`: ![image](https://hackmd.io/_uploads/BkdYoEWxC.png)

> Cách hoạt động của đoạn code trên: Kiểm tra vai trò người dùng có phải quản trị viên hay không thông qua giá trị biến `isAdmin`, nếu đúng sẽ tạo và hiển thị từ `Admin panel` trên giao diện – đường dẫn tới trang quản trị. Và vô tình cũng đã để lộ đường dẫn này tới người dùng.

> Thêm đường dẫn trên vào URL để tới trang quản trị: ![image](https://hackmd.io/_uploads/SyCg34-l0.png)

> Xóa tài khoản `carlos` và solved bài lab: ![image](https://hackmd.io/_uploads/HJOXnNblR.png)

# **3. Lab: Unprotected admin functionality with unpredictable URL**
> Đề bài cho biết đường dẫn tới trang quản trị là `/admin`, và khi truy cập vào trang đó thì trang web sẽ dựa vào cookie để xác thực bạn có phải là Admin hay không. Hoàn thàn bài lab bằng cách truy cập vào trang quản trị và xóa người dùng `carlos`:
![image](https://hackmd.io/_uploads/S1IATEbxC.png)

> Vì đề đã cho sẵn tài khoản hợp lệ, ta thử đăng nhập vào và thử tiếp cận `/admin` và truy cập bị từ chối vì không phải Admin: ![image](https://hackmd.io/_uploads/HkVK0NZgC.png)

>Vào Burp bắt request và phát hiện có 1 cookie được gửi đi cùng yêu cầu truy cập `/admin`: ![image](https://hackmd.io/_uploads/r1c2C4Zl0.png)

>Thử đổi giá trị này thành `True` và send và nhận được quyền truy cập tới trang quản trị: ![image](https://hackmd.io/_uploads/rkZgyBbe0.png)

>Cuối cùng chỉ việc đổi giá trị cookie bên trong trang web và tải lại trang: ![image](https://hackmd.io/_uploads/BkK7yBWeC.png)

> Xóa người dùng `carlos` và solved bài lab: ![image](https://hackmd.io/_uploads/HyFr1HZlC.png)

# **4. Lab: User role can be modified in user profile**
>Ngay tiêu đề cũng đã cho ta biết hướng giải quyết của bài lab, đó là phải sửa `role` của người dùng thành của 2 và xóa tài khoản `carlos` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/Syd91r-l0.png)

>Lần này đăng nhập vào tài khoản hợp lệ, truy cập `/admin` nhưng không có cookie nào được gửi đi để xác thực có phải là Admin không như bài trước: ![image](https://hackmd.io/_uploads/BJgfWBbg0.png)

> Do đề bài cho biết hệ thống sử dụng tham số `roleid` để xác định vai trò người dùng, nên ta cần tìm cách “gửi kèm” giá trị này tới hệ thống với tài khoản `wiener`. Điều đầu tiên nghĩ đến là chúng ta sẽ lựa chọn thời điểm gửi tại thời điểm đăng nhập.

>Truyền thêm tham số `roleid=2` khi đăng nhập: ![image](https://hackmd.io/_uploads/ry2RmH-lA.png)

>Và chọn "Follow redirection" nhưng điều này không thể khiến ta trở thành admin được: ![image](https://hackmd.io/_uploads/r1n7NSbgR.png)

>Tiếp tục thử thêm cách nữa bằng cách truyền tham số vào URL:![image](https://hackmd.io/_uploads/SJM9EHblR.png)

>Nhưng nó sẽ chuyển hướng mình ngay tới trang login, vậy cách này cũng không khả thi cho lắm!

>Tìm khắp cả lab và chỉ còn mỗi chức năng `Update email` là chưa sờ tới: ![image](https://hackmd.io/_uploads/Hyr0VSblR.png)

>Ta thử update email và phát hiện 1 điều rằng trong respone của server trả về khi update có trường `roleid=1` !! ![image](https://hackmd.io/_uploads/SyvqBBbgR.png)

>Thử thêm vào request gửi đi tham số `roleid=2` và Send: ![image](https://hackmd.io/_uploads/ByVRBrWeR.png)

>Quay lại thử truy cập `/admin` và thành công truy cập được! !![image](https://hackmd.io/_uploads/rkQWUHWlA.png)

>Xóa tài khoản `carlos` và solved bài lab: ![image](https://hackmd.io/_uploads/H1LrLHZg0.png)

# **5. Lab: URL-based access controll can be circumvented**
> Đề bài cho biết đường dẫn tới trang `/admin` đã được thiết lập để chặn các yêu cầu từ bên ngoài truy cập. Tuy nhiên, phần back-end hỗ trợ header X-Original-URL. Hoàn thành bài lab bằng cách truy cập trang quản trị và xóa người dùng `carlos`
![image](https://hackmd.io/_uploads/rkYXvS-g0.png)

> Truy cập vào trang web, ta thấy có nút `Admin Panel` để trỏ tới trang quản trị: ![image](https://hackmd.io/_uploads/SJFof9-xA.png)

> Truy cập thử vào đó nhưng bị từ chối: ![image](https://hackmd.io/_uploads/S1HkXq-g0.png)

>Bắt request và quan sát thấy mã trạng thái trả về là `403 Forbidden` ![image](https://hackmd.io/_uploads/SkKB7qbxA.png)

>Và như trong mô tả của đề bài thì phản hồi `Access denied` trong lab này có thể được đưa ra từ  front-end, tức là nó đã chặn tất cả requests truy cập tới /admin thông qua URL. Vậy làm thế nào để có thể bypass được nó??

>Có 1 HTTP Header rất hữu ích trong trường hợp này, đó là `X-Original-URL`. Nó sẽ giúp ta ghi đè nội dung URL khi request được gửi đi. Ví dụ ta sẽ đi tới đường dẫn `/` nhưng sẽ ghi đè lên đó bằng `X-Original-URL: /test`![image](https://hackmd.io/_uploads/ryKxPcZg0.png)

>Rõ ràng lần này thì respone trả về là `404 Not Found`, có nghĩa là yêu cầu đã đi qua được rào chắn phía FE nhưng không tìm thấy tài nguyên trong server tương ứng với `/test` phía BE. Và lần này ta sẽ đi tới trang quản trị bằng `/admin`![image](https://hackmd.io/_uploads/HkSdP5We0.png)

> Kết quả thuận lợi, chắc chắn rằng ta đã có thể truy cập vào Admin Panel. Giờ chỉ cần lấy URL để xóa người dùng `carlos`![image](https://hackmd.io/_uploads/SJl9dqZgA.png)

>Thay tham số vào request, Send và solved bài lab:![image](https://hackmd.io/_uploads/BycGKqWeA.png)
![image](https://hackmd.io/_uploads/Bkk4F5-lA.png)


# **6. Lab: Method-based access control can be circumvented**
>Đề bài miêu tả lab này chứa lỗi kiểm soát truy cập một phần dựa vào phương thức HTTP request. Họ cho ta 1 tài khoản `administrator:admin` để làm quen với giao diện quản trị của Admin. Nhiệm vụ của chúng ta là từ người dùng bình thường `wiener:peter`, sẽ tìm cách khai thác lỗ hổng kiểm soát truy cập để biến mình trở thành Admin:
![image](https://hackmd.io/_uploads/ryhXh5ZxC.png)

>Đầu tiên ta đăng nhập vào tài khoản của quản trị viên và quan sát Admin Panel: ![image](https://hackmd.io/_uploads/HkgD09-xR.png)

> Tính năng cho phép chúng ta có thể upgrade hoặc downgrade vai trò của bất kì người dùng nào. Thử upgrade vai trò của của người dùng `carlos` và quan sát request trong Burp Suite:![image](https://hackmd.io/_uploads/HkhcCcWxC.png)

> Hệ thống chuyển tới đường dẫn `/admin-roles` và truyền vào hai tham số `username=carlos&action=upgrade` bằng phương thức POST. Lưu ý giá trị tại header Cookie session=ymQIriK7aenJ3lfnN1MTnCRgdSkPDunE được sử dụng để xác thực người dùng.

>Đăng nhập với tài khoản `wiener:peter` để lấy seesion hiện tại của wiener: ![image](https://hackmd.io/_uploads/HySzWi-e0.png)

> Thay giá trị session của wiener vào session trong request upgrade role:![image](https://hackmd.io/_uploads/Hy9iZiWlR.png)

> Chúng ta thu được thông báo **“Unauthorized”**. Do hệ thống không thực hiện kiểm tra các HTTP request method từ người dùng, nên ta có thể vượt qua lớp bảo vệ này bằng cách sử dụng HTTP request method **POSTX**:![image](https://hackmd.io/_uploads/rkSgGjWgA.png)

> Ta thấy rằng hệ thống trả về thống báo **“Missing parameter ‘username'”**, điều này chứng tỏ chúng ta đã vượt qua cơ chế xác thực session. Thêm tham số username gửi tới hệ thống bằng cách sử dụng HTTP request method GET và thay giá trị username=wiener:![image](https://hackmd.io/_uploads/H1yy7sZxR.png)

>Thông báo `wiener` đã được upgrade thành công và solved bài lab: ![image](https://hackmd.io/_uploads/H1AmXoZxC.png)

# **7. Lab: User ID controlled by request parameter**
>Đề bài mô tả lab này chứa lỗ hổng trong dạng kiểm soát truy cập theo chiều ngang. Mỗi người dùng có một giá trị API duy nhất, nhiệm vụ của chúng ta sẽ dựa vào lỗ hổng này để thu thập giá trị API của `carlos` và submit. Chúng ta được cung cấp một tài khoản hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/SJgrNiWgC.png)

> Đăng nhập vào tài khoản `wiener`, trong trang cá nhân của người dùng chứa API của người dùng tương ứng: ![image](https://hackmd.io/_uploads/SyUoroWlR.png)

>Nhìn lên URL thì thấy có tham số `id=wiener`. Gía trị id này là username của người dùng, và khi truyền lên thông qua URL thì server sẽ trả về kết quả là hồ sơ người dùng tương ứng với id đó: ![image](https://hackmd.io/_uploads/SywFUjblC.png)
>
>Thực hiện thay đổi tham số cho id thành `carlos` và chuyển hướng tới trang cá nhân họ, lúc này giá trị API của victim được hiển thị ngay trong FE: ![image](https://hackmd.io/_uploads/S1IRLoZgC.png)

>Copy API của `carlos` và submit để solved bài lab: 
>![image](https://hackmd.io/_uploads/BknevjWlA.png)

![image](https://hackmd.io/_uploads/HkO-DsblR.png)

# **8. Lab: User ID controlled by request parameter, with unpredictable user IDs**
>Miêu tả đề bài cho biết lab này tồn tại lỗ hổng trong dạng kiểm soát truy cập theo chiều ngang. Mã định danh người dùng GUIDs là một giá trị không thể đoán được, tuy nhiên chúng có thể được tìm thấy đâu đó xung quanh trang web. Chúng ta cần truy cập vào hồ sơ carlos là lấy được giá trị API key của anh ấy. Chúng ta được cung cấp một tài khoản hợp lệ `wiener:peter`.
![image](https://hackmd.io/_uploads/BkgZjibgA.png)

>Đăng nhập vào `wiener` và được thấy trong hồ sơ của tài khoản có chứa API key tương ứng. Nhìn lên URL thì phần id đã được mã hóa và không còn ở dạng bản rõ nữa, nên ta không thể biết được giá trị cho `carlos` là bao nhiêu để truy cập: ![image](https://hackmd.io/_uploads/r1XpojWxR.png)

>Nhưng đề bài cho biết nó được giấu ở đâu đó nằm trong bài thôi, ta tiến hành đi 1 vòng trang web, xem các request gửi đi thì cũng không khai thác được gì.

>Phát hiện khi xem chi tiết từng bài viết thì có thể click vào tác giả để truy cập tới profile của họ: ![image](https://hackmd.io/_uploads/SJjrhi-eC.png)

>Vào trang cá nhân của `carlos` thì thấy trên URL có trường `userId=f89e125f-b95b-45ed-96fd-b71677302498` có định dạng khá giống với định dạng của wiener ở trên: ![image](https://hackmd.io/_uploads/r12K2o-l0.png)

>Khả năng 90% là giá trị này là server đã để lộ id của người dùng ra bên ngoài URL, mặc dù đã mã hóa nó và không thể đoán được. Lúc này ta chỉ việc truy cập vào profile của `carlos` và submit API key để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r12N6j-xR.png)
![image](https://hackmd.io/_uploads/BySUpjZlA.png)

# **9. Lab: User ID controlled by request parameter with data leakage in redirect**
>Đề bài mô tả bài lab chứa lỗ hổng kiểm soát truy cập khiến thông tin nhạy cảm bị rò rỉ trong nội dung phản hồi chuyển hướng. Họ cung cấp 1 tài khoản hợp lệ `wiener:peter` và mục tiêu của ta là lấy được API key của `carlos` và submit:
![image](https://hackmd.io/_uploads/BkWpToWeA.png)

>Đăng nhập vào `wiener` thì tới được trang cá nhân: ![image](https://hackmd.io/_uploads/rybBk2WlA.png)

>Trong URL chứa tham số `id=wiener`, thử thay đổi thành `carlos` và enter nhưng nó lại chuyển hướng sang trang login chứ không phải là profile của `carlos` như mong muốn: ![image](https://hackmd.io/_uploads/B1ack2blC.png)

> Quan sát HTTP history trong Burp Suite, request `/my-account?id=carlos` vẫn được server thực hiện và trả về response thành công: ![image](https://hackmd.io/_uploads/B1QZehZg0.png)

> Và ta có được API key của `carlos`: ![image](https://hackmd.io/_uploads/BJkBlnZxA.png)

>Submit nó và solved bài lab: ![image](https://hackmd.io/_uploads/rk9Pg3ZgA.png)

# **10. Lab: User ID controlled by request parameter with password disclosure**
> Đề bài mô tả trang cá nhân của người dùng trực tiếp chứa mật khẩu hiện tại ở dạng ẩn. Chúng ta cần khai thác lỗ hổng kiếm soát truy cập, thu thập mật khẩu tài khoản administrator và thực hiện xóa tài khoản người dùng carlos. Lab cung cấp một tài khoản hợp lệ là `wiener:peter`
![image](https://hackmd.io/_uploads/BkhslhWgC.png)

>Đăng nhập vào tài khoản của `wiener`, trang cá nhân có thêm 1 trường để người dùng có thể đổi mật khẩu của họ. Và dường như thông tin mật khẩu này đã được điền sẵn. Nhưng giá trị điền sẵn này lấy ở đâu ra? ![image](https://hackmd.io/_uploads/BJlWmvSzgC.png)

> Theo nghi ngờ thì ta xem request`GET /my-account?id=wiener` trong Burp thì phát hiện 1 trường ấn dùng để lưu giá trị cho password: ![image](https://hackmd.io/_uploads/SkBADSfg0.png)

>Ngoài ra thì trên URL còn chứa tham số `id=wiener`. id mang giá trị là username của người dùng. Khi truyền lên hệ thống sẽ trả về giao diện hồ sơ tương ứng với giá trị tham số id. Thay đổi giá trị `id=administrator` trong URL và truyền lên server, kết quả hiển thị trang cá nhân của administrator và chúng ta thu được mật khẩu của admin: ![image](https://hackmd.io/_uploads/rJheYBGx0.png)

>Tiến hành đăng nhập vào admin và xóa người dùng `carlos` để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BJdNtHGe0.png)
![image](https://hackmd.io/_uploads/BkOvYHGxR.png)

# **11. Lab: Insecure direct object references**
> Mô tả nói rằng lab này lưu trữ nhật ký trò chuyện của người dùng trực tiếp trên hệ thống tệp của máy chủ và truy xuất chúng bằng URL tĩnh.Nhiệm vụ của ta là tìm mật khẩu của người dùng `carlos` và đăng nhập vào tài khoản của nạn nhân.
![image](https://hackmd.io/_uploads/Hyty9HzeR.png)

>Vào web thì ta thấy có phần "Live chat" để người dùng nhắn tin với nhau: ![image](https://hackmd.io/_uploads/B1Xd5SMeC.png)
![image](https://hackmd.io/_uploads/r1wY9BflA.png)

>Thử gửi 1 tin nhắn và quan sát request được gửi đi trong Burp: ![image](https://hackmd.io/_uploads/Sy_pjSfeR.png)

>Có vẻ như không khai thác được gì, tiếp tục chọn `View transcipt` và ta sẽ được tải về 1 file .txt có nội dung là lịch sử tin nhắn: ![image](https://hackmd.io/_uploads/By7G3SzgR.png)

>Xem trong Burp request này: ![image](https://hackmd.io/_uploads/Hy0I2rGx0.png)

>Tại sao lại tải về nội dung trò chuyện bắt đầu từ "2.txt" mà không phải 0 hoặc 1?? Có vẻ như còn có 1 cuộc trò chuyện khác trong lịch sử! Thử thay đổi để lấy nội dung trong file "1.txt" xem: ![image](https://hackmd.io/_uploads/ryRxarzlC.png)

>Oh, và trong cuộc trò chuyện giữa 2 người này thì có chứa mật khẩu ta cần tìm. Đăng nhập vào `carlos` và hoàn thành bài lab thôi: ![image](https://hackmd.io/_uploads/SycraHMxC.png)

# **12. Lab: Multi-step process with no access control on one step**
>Đề bài mô tả trang quản trị của web này gồm nhiều bước để có thể thay đổi role của người dùng. Ta có thể làm quen với Admin Panel bằng tài khoản có role admin cho sẵn `administrator:admin`. Nhiệm vụ của bài này là từ người dùng thường cho trước `wiener:peter`, hãy tìm lỗ hổng về kiểm soát truy cập và tự leo thang đặc quyền lên admin: 
![image](https://hackmd.io/_uploads/S1ycpHfeA.png)

>Đăng nhập vào tài khoản admin và vào trang Admin Panel thì có giao diện để có thể sửa đổi quyền cho người dùng: ![image](https://hackmd.io/_uploads/Hy2VDDGlR.png)

>Thử thực hiện nâng quyền của `carlos` lên admin, và có bước để xác nhận hiện lên:![image](https://hackmd.io/_uploads/Sy7_PvGeR.png)

>Xác nhận và hệ thống đã nâng cấp `carlos` lên thành admin: ![image](https://hackmd.io/_uploads/H1XoPwfxR.png)

>Vào HTTP History của Burp để xem lại các request đã gửi đi: ![image](https://hackmd.io/_uploads/HkFDdDflA.png)

>Có 2 request với method POST đã được gửi đi, ta sẽ đi phân tích từng cái. Đầu tiên là request dùng để xác nhận hành động nâng quyền người dùng: ![image](https://hackmd.io/_uploads/SkXhOwfxC.png)

>Khi đó, nó sẽ gửi request tới đường dẫn `/admin-roles` với 2 tham số `username=carlos&action=upgrade`.

>Tiếp theo, ở request thứ 2 sẽ thực hiện nâng quyền. Điều khác biệt ở đây là nó có thêm tham số `confirm=true` khi gửi đi: ![image](https://hackmd.io/_uploads/rJnuYwzxR.png)

>Câu hỏi đặt ra ở đây là: liệu người dùng bình thường có thể thực hiện thao tác upgrade này để tự leo thang đặc quyền hay không? Ta sẽ đăng nhập vào tài khoản `wiener` và lấy copy session của người dùng thường này và cho vào request thứ nhất ở trên và thử nghiệm: ![image](https://hackmd.io/_uploads/rkY_qwGxC.png)

>Thay vào request dùng để xác nhận thao tác upgrade nhưng nhận được thông báo là **"Unauthorized"**:![image](https://hackmd.io/_uploads/SJ2AcPfgR.png)

>Chúng ta biết rằng chức năng thay đổi role người dùng hoạt động trong hai bước. Vì vậy, tiếp tục thử bỏ qua bước thứ nhất (xác nhận hành động), trực tiếp gửi các tham số `action=upgrade&confirmed=true&username=wiener` bằng phương thức **POST** tới `/admin-roles` để xem kết quả: ![image](https://hackmd.io/_uploads/H1X_oDGeA.png)

>Kết quả là **"302 Found"**, tức là ta đã thành công, trực tiếp tự leo thang đặc quyền của mình mà bỏ qua **bước 1: Xác nhận** của hệ thống. Cuối cùng solved bài lab: ![image](https://hackmd.io/_uploads/S13khPzx0.png)




# **13. Lab: Referer-based access control**
>Đề bài mô tả bài lab dựa trên thông tin header **Referer** để xác thực người dùng. Ta được cung cấp một tài khoản với role quản trị viên `administrator:admin` để thu thập các đường dẫn cũng như thông tin liên quan tới chức năng upgrade vai trò người dùng. Ngoài ra còn được cung cấp một tài khoản người dùng bình thường `wiener:peter`, nhiệm vụ của ta là khai thác lỗ hổng kiểm soát truy cập, thực hiện leo quyền tài khoản `wiener` lên quyền quản trị viên.
![image](https://hackmd.io/_uploads/ryoATBzl0.png)

>Đăng nhập vào tài khoản admin và có nút để vào phần quản trị: ![image](https://hackmd.io/_uploads/rkPaRPfg0.png)

> Truy cập vào đó và có giao diện để có thể sửa đổi quyền cho người dùng: ![image](https://hackmd.io/_uploads/H1jVJ_Gl0.png)


>Thử nâng quyền cho `carlos`, thì không yêu cầu phải qua bước trung gian để xác nhận hành động như bài trước mà trực tiếp nâng quyền lên admin luôn: ![image](https://hackmd.io/_uploads/B1PwyuGxA.png)

> Quan sát bên trong HTTP History: ![image](https://hackmd.io/_uploads/ryeoJdfgC.png)

>Lần này thì request sẽ không được gửi theo method **POST** như mọi lần nữa, thay vào đó là method **GET** cùng với các tham số `?username=carlos&action=upgrade` được truyền đi tới `/admin`. Ngoài ra, còn có thêm 1 HTTP Header nữa **Referer** dùng để xác định nguồn gốc của request được gửi tới từ URL nào, từ đó quyết định xem có thực hiện hành động hay không: 

![image](https://hackmd.io/_uploads/SyT_edzlR.png)

>Thử thay đổi giá trị trong trường này thành 1 giá trị URL nào đó, ví dụ `https://test.com`: ![image](https://hackmd.io/_uploads/r1_Ce_fl0.png)

>Kết quả trả về sẽ là **"401 Unauthorized"**. Điều này chứng tỏ hệ thống tồn tại cơ chế xác thực danh tính người dùng qua tiêu đề **Referer**, trong đó người dùng phải gửi yêu cầu upgrade tài khoản từ đường dẫn `/admin` mới có thể thực hiện.

>Từ những suy luận ở trên, nghi ngờ rằng người dùng bình thường cũng có thể tự nâng quyền cho bản thân bằng cách thay đổi giá trị trong header  **Referer** thành `/admin`. Cùng thử thôi!! Đầu tiên đăng nhập  và lấy session của `wiener`: ![image](https://hackmd.io/_uploads/r1bGfOMx0.png)

>Tiếp tục sửa các giá trị trong request, đổi `username=wiener`, `session` và giá trị của header Referer thành `/admin`: ![image](https://hackmd.io/_uploads/Hyl3fdGx0.png)

>Respone trả về **"302 Found"**, tức là suy luận của ta đã đúng: ![image](https://hackmd.io/_uploads/HkCafOMeC.png)

>Hoàn thành bài lab vì ta đã upgrade `wiener` lên admin thành công: ![image](https://hackmd.io/_uploads/H1W4QOzeR.png)

 