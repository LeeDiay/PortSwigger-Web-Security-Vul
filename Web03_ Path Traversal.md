# Số bài giải được / Tổng số : 6/6

# **1. Lab: File path traversal, simple case**
> Đề bài cho biết có 1 lổ hổng path traversal ở phần hiển thị hình ảnh sản phẩm. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` : 
![image](https://hackmd.io/_uploads/rytfYDtx0.png)

>Truy cập trang chủ, ấn vào xem chi tiết sản phẩm, quan sát trong HTTP History có các request gửi đi để lấy ảnh hiển thị lên: ![image](https://hackmd.io/_uploads/BJdd2wteR.png)

>Ta thử cho 1 request đó sàn Repeater: ![image](https://hackmd.io/_uploads/HyD2nwFlA.png)

>Có vẻ như nó sẽ lấy ảnh về web thông qua tham số `filename`. Ảnh này có vẻ nằm ở đường dẫn `/var/www/html` (web root directory của Linux).

>Tiến hành đổi giá trị cho tham số `filename` để đọc file `/etc/passwd` bằng cách thêm tăng dần `../` đến khi nào nhận được phản hồi đúng: ![image](https://hackmd.io/_uploads/SkOY6PFe0.png)

>Thông báo trả về **"No such file"**. Tiếp tục tới lần thứ 2 thì ta đã đọc được file cần tìm: ![image](https://hackmd.io/_uploads/B1n6TwtgA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HkR0TDKgA.png)

# **2. Lab: File path traversal, traversal sequences blocked with absolute path bypass**
> Đề bài cho biết có 1 lổ hổng path traversal ở phần hiển thị hình ảnh sản phẩm. Nhưng lần này thì ứng dụng đã chặn những câu để truyền tải. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` : 
![image](https://hackmd.io/_uploads/HysMyuKeR.png)

>Tiếp tục thì có 1 request với method POST để lấy ảnh từ server về với tham số `filename`: ![image](https://hackmd.io/_uploads/Sy1mWOtgC.png)

>Thử đọc file `/etc/passwd` bằng cách như lab trước nhưng đều bị trả về mã **"400 Bad request"**, chứng tỏ trang web đã chặn những hành động này của ta: ![image](https://hackmd.io/_uploads/rJg2ZdFxA.png)

>Nhưng khi viết thẳng trực tiếp đường dẫn tuyệt đối để tham chiếu thẳng vào `/etc/passwd` thì ta lại có thể bypass nó và đọc được nội dung file: ![image](https://hackmd.io/_uploads/H1Yef_tgA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HyY-MdYgC.png)

# **3. Lab: File path traversal, traversal sequences stripped non-recursively**
> Đề bài cho biết có 1 lổ hổng path traversal khi hiển thị hình ảnh sản phẩm. Nhưng lần này thì ứng dụng đã loại bỏ những câu để truyền tải trước khi đưa vào sử dụng. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` : 
![image](https://hackmd.io/_uploads/ByR7QOFgA.png)

>Tiếp tục thì có 1 request với method POST để lấy ảnh từ server về với tham số `filename`:
>![image](https://hackmd.io/_uploads/BkqXN_txC.png)

>Thử tấn công bằng cách của 2 bài lab trước: truyền trực tiếp đường dẫn tuyệt đối và tương đối vào nhưng không có hiệu quả: ![image](https://hackmd.io/_uploads/BkImHuYxC.png)
![image](https://hackmd.io/_uploads/BkzUHOtgC.png)

> Dữ kiện đề bài cho 1 chi tiết rằng: ứng dụng đã thực hiện tự động loại bỏ các câu để truyền tải trước khi sử dụng. Có nghĩa là khi ta nhập vào **"/.../...//"** thì nó bị xóa(lọc) chỉ còn lại **"/../"** mà không tiếp tục lọc tiếp(không đệ quy) ? Vậy thì có 1 cách để bypass nó rất hay, đó là ta sẽ viết lại thành **"/....//....//"**, và sau khi được server xử lí sẽ thành: **"/../../"**, điều này có thể thực hiện tấn công như mong muốn! Thử nghiệm với payload `....//....//....//etc/passwd`: ![image](https://hackmd.io/_uploads/ryfmvOKeR.png)

>Đã đọc được file `/etc/passwd` thành công và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BkpNDOtgR.png)

# **4. Lab: File path traversal, traversal sequences stripped with superfluous URL-decode**
>Đề bài cho biết có 1 lổ hổng path traversal khi hiển thị hình ảnh sản phẩm. Nhưng lần này thì ứng dụng đã chặn những câu để truyền tải, và thực hiện URL-decode đầu vào trước khi đưa vào sử dụng. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` :
![image](https://hackmd.io/_uploads/BkzuOuYxR.png)

>Tiếp tục thì có 1 request với method POST để lấy ảnh từ server về với tham số `filename`:![image](https://hackmd.io/_uploads/r1TsZtKxC.png)

>Thử tấn công bằng payload `../../../etc/passwd` nhưng bị trả về **"400 Bad Request"**, chứng tỏ rằng ứng dụng chặn `../`: ![image](https://hackmd.io/_uploads/S1CMGYYx0.png)

> Đề cho biết rằng server thực hiện url-decode trước khi sử dụng tham số `filename` =>  ta có thể bypass cơ chế bảo vệ bằng cách double url-encode: ![image](https://hackmd.io/_uploads/SkudIKtl0.png)

>Thay đổi giá trị của `filename` thành payload: 
>`filename=..%252F..%252F..%252Fetc/passwd`
>![image](https://hackmd.io/_uploads/Syp6IFtg0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJ9C8YKg0.png)

# **5. Lab: File path traversal, validation of start of path**
>Đề bài cho biết có 1 lổ hổng path traversal khi hiển thị hình ảnh sản phẩm. Ứng dụng sẽ truyền đi đường dẫn đầy đủ trong request gửi tới server, và validate `filename` phải bắt đầu bằng `/var/ww/html/`. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` :
![image](https://hackmd.io/_uploads/ryrvvFtxC.png)

>Ứng dụng lấy ảnh từ server từ 1 request với tham số `filename` đi kèm với đường dẫn tuyệt đối `/var/www/file.jpg`
![image](https://hackmd.io/_uploads/Sk-cOtKgA.png)

> Thử nhập payload `/etc/passwd` thì lỗi báo **"Missing parameter 'filename'"** do thiếu giá trị server mong đợi `/var/www/images` ở đầu: 
![image](https://hackmd.io/_uploads/SyntYtFe0.png)

> Với điều kiện đó, ta chỉ cần thêm tiền tố `/var/www/image` vào trước các payload để tấn công. Chỉnh sửa thành: 
> `/var/www/images/../../../etc/passwd`
>![image](https://hackmd.io/_uploads/SJakstKg0.png)

>Đọc được file `/etc/passwd` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1-ZsFteC.png)

# **6. Lab: File path traversal, validation of file extension with null byte bypass**
>Đề bài cho biết có 1 lổ hổng path traversal khi hiển thị hình ảnh sản phẩm. Ứng dụng sẽ validate `filename` phải kết thúc bằng 1 giá trị mong muốn. Để solve bài lab thì cần phải lấy được nội dung của file `/etc/passwd` :
![image](https://hackmd.io/_uploads/BytXiKtlC.png)

>Khác với bài trước thì bài này yêu cầu hậu tố của input bắt buộc phải kết thúc bằng `.jpg`: ![image](https://hackmd.io/_uploads/rkgTjnttxA.png)

> Để vừa thỏa mãn điều kiện của server đồng thời vẫn path traversal để đọc file, ta có thể dùng payload như sau: `../../../etc/passwd%00.jpg`
> Bằng cách thêm kí tự trống `null` vào sau payload tấn công: khi server lấy file sẽ lấy phần trước giá trị `null` và payload được thực thi, đồng thời vẫn thỏa mãn phần đuôi có định dạng là `.jpg`

>Đọc được file `/etc/passwd` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJ6C6ttlA.png)
