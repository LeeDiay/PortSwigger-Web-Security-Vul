---
title: 'Web14: HTTP Host header attacks'
tags: [web14]

---

I. Đặt vấn đề
-------------

### 1\. HTTP Host Header

Trong giao thức HTTP, trường Header **Host** được sử dụng để chỉ định tên miền (domain name) của máy chủ web được đang được truy cập hoặc đang trả về nội dung. Host header được sử dụng trong các yêu cầu HTTP cho phép máy chủ web nhận biết tên miền được yêu cầu và phục vụ nội dung phù hợp.

Xét thao tác truy cập URL **`https://viblo.asia/followings`**, quan sát trong Burp Suite, có thể thấy header Host tại dòng 2 2 của request:

![image.png](https://images.viblo.asia/4c71da0e-2a4d-4bda-a7f0-ca68d0f4a69c.png)

Trong đó, **`viblo.asia`** là tên miền của máy chủ web mà yêu cầu này được gửi đến. Khi server nhận được yêu cầu này, Header Host được sử dụng để xác định nội dung phù hợp trả về cho yêu cầu này.

### 2\. Tấn công tiêu đề Host trong giao thức HTTP

![image.png](https://images.viblo.asia/de65a112-96ac-41e6-a6c0-8f37fb1189ed.png) 

**Tấn công tiêu đề Host** trong giao thức HTTP (HTTP Host header attack) là dạng tấn công dựa vào hành vi thay đổi hoặc sử dụng không đúng trường header **Host** và một số trường header có tính năng tương tự. Lỗ hổng xảy ra khi hệ thống cấu hình sai hoặc do các lỗi logic trong lập trình.

II. Phân tích các dạng HTTP Host header attack và ngăn chặn
-----------------------------------------------------------

### 1\. Warm up

Ví dụ với ngôn ngữ **PHP**, chúng ta có thể lấy giá trị Host header trong yêu cầu người dùng bằng **`$_SERVER['HTTP_HOST']`**. Và một trong những nguyên nhân phổ biến nhất dẫn đến dạng tấn công này là do lập trình viên sử dụng **`$_SERVER['HTTP_HOST']`** khi cần dùng tới giá trị của domain mà không có quy trình kiểm tra tốt cho giá trị này.

Ta có thể dựng một web đơn giản với mục đích thử nghiệm bằng đoạn code như sau:

```php=
$host = $_SERVER['HTTP_HOST'];
echo "Host: " . $host;

```

 

Chương trình sẽ hiển thị giá trị header Host khi chúng ta truy cập:

![image.png](https://images.viblo.asia/e99a55aa-cb5a-4b97-9211-e220c97a00dc.png)

Bởi vì trang web không thực hiện kiểm tra giá trị header Host được người dùng gửi tới, kẻ tấn công có thể dễ dàng thay đổi giá trị header Host bằng Burp Suite:

![image.png](https://images.viblo.asia/fcb22302-1af0-4b49-b989-deb74ac01c3c.png)

### 2\. Tấn công Host header với tính năng đặt lại mật khẩu

Một chức năng thường thấy trong dạng tấn công này là Đặt lại mật khẩu, đây cũng là một tính năng quen thuộc với người sử dụng và xuất hiện hầu hết trong tất cả website quản lý người dùng bằng tài khoản.

Thông thường, tính năng reset password sẽ gửi một đường dẫn đặt lại mật khẩu tới email của người dùng yêu cầu. Phần lớn đường dẫn có dạng như sau:

**`https://<domain-name-of-website>/?token=<random-token>`**

Khi lập trình viên xây dựng chương trình cho chức năng này sẽ cần sử dụng đến tên miền của ứng dụng. Xét đoạn code sau:

```php=
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Get user input
    $email = $_POST['email'];

    // Validate email
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'Invalid email address';
    } else {
        // Generate a random token
        $token = bin2hex(random_bytes(32));

        // Send reset link to user's email
        $reset_link = 'https://' . $_SERVER['HTTP_HOST'] . '/forgotpass?token=' . $token;

        // For testing:
        echo "Reset link: " . $reset_link;

        // Process ...
    }
}
?>

```

 

Đoạn code trên đã tạo một đường dẫn đặt lại mật khẩu gửi tới email của người dùng, trong đó phần domain website đã trực tiếp ghép chuỗi từ giá trị **`$_SERVER['HTTP_HOST']`**:

```none
$reset_link = 'https://' . $_SERVER['HTTP_HOST'] . '/forgotpass?token=' . $token;

```

 

**`$_SERVER['HTTP_HOST']`** lấy giá trị từ header Host trong HTTP request nên nó hoàn toàn có thể bị thay đổi bởi kẻ tấn công do chức năng không chứa các bước kiểm tra giá trị này. Chẳng hạn:

![image.png](https://images.viblo.asia/c16d2439-d24e-4550-beac-c1a6f532d60d.png)

Khi đó, kẻ tấn công có thể lợi dụng chức năng này nhằm gửi một email giả mạo domain (thường là domain do kẻ tấn công sở hữu) đến nạn nhân, khi họ không chú ý và click vào đường link đó, kẻ tấn công sẽ lấy được giá trị token của họ và thực hiện đổi mật khẩu email nạn nhân bằng token đó.

Ví dụ với Collaborator client trong Burp Suite:

![image.png](https://images.viblo.asia/c279ae73-411e-4e91-a996-c65e8f905174.png)

![image.png](https://images.viblo.asia/cf87d1ed-32f0-45ce-bf13-1fc906df3766.png)



Để ngăn chặn tấn công bằng cách thay đổi giá trị header Host trong request, có thể sử dụng một số cách như sau:

Đối với ứng dụng chứa domain xử lý đặt lại mật khẩu duy nhất, không cần lấy giá trị **`$_SERVER['HTTP_HOST']`**.

**`$reset_link = 'http://example.com/forgotpass?token=' . $token;`**

Nếu có nhiều domain, có thể sử dụng whitelist:

```php=
$expected_domains = array('example.com', 'example2.com', 'example3.com');
$host = $_SERVER['HTTP_HOST'];

// Compare the Host header value with the expected domains
if (!in_array($host, $expected_domains)) {
    // Log or handle the error, and reject the request
    die('Invalid Host header');
}

```

 

Trong trường hợp đường link đặt lại mật khẩu chứa thông tin do người dùng nhập, nên kiểm tra tính hợp lệ của giá trị input:

```php=
$email = $_POST['email']; // Assuming user input is used to retrieve email

// Validate and sanitize the email address
$email = filter_var($email, FILTER_VALIDATE_EMAIL);
if (!$email) {
    // Log or handle the error, and reject the request
    die('Invalid email address');
}

// Generate the reset password link using the sanitized email address
$reset_link = 'http://example.com/forgotpass?token=' . $token . '&email=' . urlencode($email);

```

 

### 3\. Ứng dụng hỗ trợ một số header khác

Trong quá trình kiểm tra khả năng tấn công HTTP Host header, bên cạnh header Host, chúng ta cũng cần chú ý đến một số header khác có tính năng tương tự:

```none
X-Forwarded-Host
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
Forwarded

```

 

Trong trường hợp ứng dụng hỗ trợ các header này, kẻ tấn công hoàn toàn có thể lợi dụng chúng nhằm bypass cơ chế phòng ngừa của ứng dụng, hoặc ghi đè giá trị Host.

Điều này thường xảy ra đối với các ứng dụng sử dụng kiến trúc hệ thống trung gian (chẳng hạn một máy chủ reverse proxy). Ví dụ, header **`X-Forwarded-Host`** thường được sử dụng để gửi thông tin về host gốc của yêu cầu từ client tới server backend. Header này được thêm vào yêu cầu HTTP bởi reverse proxy để truyền tải thông tin về host mà client gửi yêu cầu ban đầu. **`X-Forwarded-Host`** giúp khắc phục vấn đề một reverse proxy có thể phục vụ nhiều tên miền khác nhau trên cùng một địa chỉ IP bằng cách xác định tên miền gốc mà client đã sử dụng, cho phép reverse proxy định tuyến yêu cầu đúng đến server backend phù hợp với tên miền đã được yêu cầu.

Khi trong request chứa header **`X-Forwarded-Host`**, nhiều framework sẽ sử dụng nó thay vì header Host, dẫn đến kẻ tấn công có thể bypass cơ chế kiểm tra tính hợp lệ của request dựa vào thay đổi giá trị header **`X-Forwarded-Host`**.

```none
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here

```

 

Ví dụ một ứng dụng có mã nguồn như sau:

```javascript=
const express = require('express');
const app = express();

// Định nghĩa route
app.get('/example', (req, res) => {
  // Lấy giá trị của header Host từ header X-Forwarded-Host
  const host = req.header('x-forwarded-host') || req.header('host');

  // Trả về giá trị của header Host
  res.send(`Host: ${host}`);
});

app.listen(3000, () => {
  console.log('Listen on port 3000');
});

```

 

Trong ví dụ này, server sử dụng Express framework định nghĩa route **`/example`** để xử lý yêu cầu GET. Trong đó, giá trị của header Host có thể được lấy từ X-Forwarded-Host hoặc Host, tùy thuộc vào sự tồn tại hay không của X-Forwarded-Host. Sau đó, giá trị này được gửi về phản hồi cho người dùng.



Hình thức tấn công HTTP Host header mang nét tương đối đặc trưng bởi cách thức thực hiện dựa vào việc thêm / thay đổi giá trị các header đặc biệt như **`Host`**, **`X-Forwarded-Host`**, ... nên về phương pháp ngăn chặn cơ bản chúng ta chỉ cần thêm một bước kiểm tra các header này. Ví dụ đoạn code sau kiểm tra giá trị header **`X-Forwarded-Host`** trong request gửi tới:

```javascript=
// Nếu giá trị của header X-Forwarded-Host không hợp lệ, sử dụng giá trị của header Host
if (!isValidHost(xForwardedHost)) {
    res.send(`Host: ${host}`);
} else {
    // Xử lý lỗi khi giá trị của header X-Forwarded-Host không hợp lệ
    res.status(400).send('Invalid header X-Forwarded-For');
}

```

Ngoài ra, có thể tùy chỉnh cấu hình reverse proxy hoặc web server không cho phép ghi đè các header, hoặc thiết lập sẵn tên miền hiện tại trong cấu hình của server, không cho phép lấy giá trị tên miền từ người dùng.

### 4\. Sử dụng tấn công Host header khai thác SSRF

![image.png](https://images.viblo.asia/e5853efd-2de4-4abb-a55f-b6cbf7d7211f.png)

Một trong những phương pháp tấn công phổ biến kết hợp lỗ hổng Host header attack và SSRF là Routing-based SSRF. Kỹ thuật tấn công này thường được sử dụng trong các hệ thống đám mây. Kẻ tấn công sẽ khai thác thành phần trung gian (như máy chủ reverse proxy) để tấn công vào các hệ thống nội bộ. Thông thường, tiêu đề "Host" được sử dụng nhằm đánh lừa máy chủ trung gian chuyển hướng yêu cầu đến một địa chỉ IP mà kẻ tấn công đưa ra.

Xét một ứng dụng web được triển khai trên nền tảng đám mây của `**[AWS](https://aws.amazon.com/)**`. Ứng dụng web sử dụng một dịch vụ load balancer để phân phối tải cho các server back-end. Dịch vụ load balancer này được cấu hình để chuyển tiếp yêu cầu HTTP đến các server back-end bằng cách sử dụng tiêu đề "Host" trong request. Ví dụ cấu hình **nginx** như sau:

```html=
server {
  listen 80;
  server_name example.com;

  location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;
  }
}

upstream backend {
  server backend1.example.com;
  server backend2.example.com;
}

```

 

Trong đó, load balancer được cấu hình để lắng nghe trên cổng 80 và chuyển tiếp các yêu cầu tới địa chỉ IP của các server back-end được định nghĩa trong upstream "backend". Điều này cho phép load balancer phân phối các yêu cầu tới các server back-end khác nhau.

Chú ý rằng trong cấu hình trên, thông qua thiết lập `"proxy\_set\_header Host $host;"`, tiêu đề "Host" trong yêu cầu HTTP được chuyển tiếp đến server back-end mà không được kiểm tra và xác thực, điều này tạo ra một lỗ hổng SSRF tiềm ẩn.

Để khai thác lỗ hổng SSRF truy cập vào các máy chủ nội bộ, kẻ tấn công sẽ gửi một yêu cầu HTTP đến ứng dụng web nhưng thay đổi tiêu đề "Host" để trỏ đến một máy chủ mà họ muốn tấn công. Yêu cầu này sẽ được gửi đến dịch vụ load balancer, sau đó sẽ được chuyển tiếp đến máy chủ cụ thể mà kẻ tấn công nhắm tới.


III. Các bài lab
-------------
## **1. Lab: Basic password reset poisoning**
>![image](https://hackmd.io/_uploads/B1Gjhm3vC.png)

>Truy cập bài lab, ở `/login` có chức năng Forgot Password cho phép người dùng lấy lại mật khẩu:
>![image](https://hackmd.io/_uploads/HkoVTQ3DR.png)

>Nhập username `wiener` mà đề bài cho trước, truy cập hòm thư email, ta nhận được 1 đường link để reset mật khẩu:
>![image](https://hackmd.io/_uploads/Sy4ia7hDR.png)

>Để ý thấy đường link này gửi kèm theo token và phần header Host của request:
>![image](https://hackmd.io/_uploads/SkMnAQ2PA.png)

>Ta sẽ ghi đè **Host** header thành exploit-server và username cần reset password là `carlos`:
>![image](https://hackmd.io/_uploads/ryrWJV3vA.png)

>Lúc này, victim `carlos` sẽ nhận được mail và click vào đường link, vô tình token sẽ bị ta thu được bên phía exploit-server, kiểm tra log ta thu được token:
>![image](https://hackmd.io/_uploads/rymtyVhPA.png)

>Lấy đường dẫn trên cho ta tới trang reset password cho `carlos`:
>![image](https://hackmd.io/_uploads/ByFskNhwA.png)

>Thực hiện reset password bất kì cho `carlos`, và ta thành công login vào được tài khoản victim `carlos`:
>![image](https://hackmd.io/_uploads/HJB0y4hvR.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/HygyxE3PR.png)

## **2. Lab: Web cache poisoning via ambiguous requests**
![image](https://hackmd.io/_uploads/rJnzb02wA.png)

>Truy cập trang chủ tại `/` và thấy rằng respone có các header liên quan đến cache, chứng tỏ trang web có sử dụng cache server:
>![image](https://hackmd.io/_uploads/rJU6W03wA.png)

>Đồng thời, mỗi lần load trang chủ thì web cũng sữ request tới `/resources/js/tracking.js` để fetch dữ liệu:
>![image](https://hackmd.io/_uploads/HyofGAnDA.png)

>Sử dụng cache buster `?test=1` và thêm 1 header **Host** bất kì, ta thấy nó reflect ở src file js `/resources/js/tracking.js`. Gửi cho đến khi `X-Cache: hit`:
>![image](https://hackmd.io/_uploads/SJoFmAnP0.png)

>Xóa Host vừa thêm và giữ nguyên cache buster, ta thấy vẫn nhận được cached response ở trên:
>![image](https://hackmd.io/_uploads/HJnjQRhP0.png)

>Chứng tỏ ta có thể poison web cache chỗ này. Ta có thể tạo 1 file `tracking.js` giả chứa mã độc trên exploit server của ta, rồi chuyển hướng trang web truy cập vào exploit server => có thể thực thi mã độc!

>Tạo file `/resources/js/tracking.js` trên exploit-server chứa `alert(document.cookie)`:
>![image](https://hackmd.io/_uploads/rJu44CnwR.png)

>Xóa cache buster và chèn exploit-server vào trường Host. Gửi request cho đến khi `X-Cache:hit`:
>![image](https://hackmd.io/_uploads/r1s2VRhw0.png)

>Khi đó, khi victim truy cập vào trang chủ `/`, sẽ load file `tracking.js` được lưu ở exploit server và kích hoạt hàm alert thành công, hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/S11xrR3DR.png)

## **3. Lab: Host header authentication bypass**
![image](https://hackmd.io/_uploads/HyfPHA3PC.png)

>Truy cập vào `/admin` thì thông báo rằng chỉ những người dùng local mới được phép truy cập:
>![image](https://hackmd.io/_uploads/BktbUR3PC.png)

>Từ đó, ta sẽ nảy ra ý tưởng inject Host header bằng các giá trị local như `127.0.0.1`, `localhost`, `127.1`,...
>Ta sẽ đi thử từng giá trị:
>![image](https://hackmd.io/_uploads/S1au8R2D0.png)
![image](https://hackmd.io/_uploads/HycFU0hvR.png)

>Khi thử tới `localhost` thì ta truy cập thành công `/admin`:
>![image](https://hackmd.io/_uploads/S1giIChv0.png)

>Và thực hiện xóa người dùng `carlos`:
>![image](https://hackmd.io/_uploads/B123LR2DC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/BkYpUC2vA.png)

## **4. Lab: Routing-based SSRF**
![image](https://hackmd.io/_uploads/rJyaAAhwC.png)

>Thay đổi giá trị của header Host thành bất kì, ta thấy server phản hồi **Server Error: Gateway Timeout (3) connecting to abc.com**
>![image](https://hackmd.io/_uploads/ryGH1kpwR.png)

>Chứng tỏ rằng server có liên hệ gì đó với Host! Ta sẽ thử thay giá trị của header này bằng máy chủ Collab:
>![image](https://hackmd.io/_uploads/BJlHgkpw0.png)

>Bên phía Collab ta nhận được các DNS Query gửi tới, ta xác nhận được server sẽ truy cập vào domain mà ta controll trong hader Host! Từ đây ta có thể sử dụng chỗ này để thực hiện SSRF.

>Từ miêu tả bài lab chúng ta biết rằng địa chỉ IP private đặt tại `192.168.0.0/24`. Với subnetmask `/24`(nghĩa là 24 bits đầu cố định) suy ra địa chỉ private của máy chủ nằm trong dải địa chỉ từ `192.168.0.0` đến `192.168.0.255`.

>Do ứng dụng truy cập tới giá trị header Host của request nên chúng ta có thể "trỏ" header Host vào từng giá trị IP trong dải trên nhằm tìm kiếm địa chỉ IP private của máy chủ.

>Sử dụng chức năng Intruder gửi 256 requests tương ứng với tất cả dải địa chỉ IP trên (lưu ý cần bỏ chọn mục "Update Host header to match target"):
>![image](https://hackmd.io/_uploads/ryVBWy6PA.png)
>![image](https://hackmd.io/_uploads/rJJu-y6DC.png)

>Kết quả hiển thị một request có response trả về status code 302, chuyển hướng tới đường dẫn **`/admin`**
>![image](https://hackmd.io/_uploads/B1_jW1TPR.png)

>Từ đó, ta có thể truy cập vào `/admin`:
>![image](https://hackmd.io/_uploads/HkPeGkpDA.png)
>![image](https://hackmd.io/_uploads/SkGbfkaDR.png)

>Thực hiện xóa `carlos` với session và csrf token mới được thêm vào sau mỗi lần truy cập `/admin`:
>![image](https://hackmd.io/_uploads/By0C7JpvA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/Skt14kTwA.png)

## **5. Lab: SSRF via flawed request parsing**
![image](https://hackmd.io/_uploads/B1tcom6wR.png)

>Bài này nâng cấp từ bài trước khi đã thiết lập firewall và chống việc thay đổi Host:
>![image](https://hackmd.io/_uploads/S1nWRXpwR.png)

>Tuy nhiên ta có thể bypass được bằng cách gửi đường dẫn tuyệt đối trên request line và thêm Collaborator domain tại Host header, ta lại SSRF thành công. (Chú ý: phải sử dụng `https:://`)
>![image](https://hackmd.io/_uploads/HkXWJ4aP0.png)

>Kiểm tra Collab thì đã có các DNS Query đến:
>![image](https://hackmd.io/_uploads/BJjGJEawC.png)

>Tiếp tục như lab trước, ta sử dụng Intruder để tìm ra mạng internal để có thể truy cập `/admin`:
>![image](https://hackmd.io/_uploads/SyTD1N6DR.png)
![image](https://hackmd.io/_uploads/HJF_JVpPC.png)

>Ta thu được địa chỉ internal IP mà server sử dụng:
>![image](https://hackmd.io/_uploads/HJTn1NaPR.png)

>Truy cập `/admin` và có form để delete user:
>![image](https://hackmd.io/_uploads/Hk1egV6w0.png)

>Thực hiện xóa `carlos` với các giá trị session và csrf kèm theo trong respone trước:
>![image](https://hackmd.io/_uploads/SkSsx4TD0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rylA6lEpvA.png)

## **6. Lab: Host validation bypass via connection state attack**
![image](https://hackmd.io/_uploads/H1PYbNTD0.png)

>Thay đổi giá trị Host thành domain Collab, ta thấy có thể SSRF:
>![image](https://hackmd.io/_uploads/HkSDH4pvA.png)

>Bên phía Collab nhận được các request tới, chứng tỏ ta có thể SSRF như các lab trước:
>![image](https://hackmd.io/_uploads/BJtYBEpDC.png)

>Truy cập vào `/admin` và lấy được csrf token:
>![image](https://hackmd.io/_uploads/S1wFYVTDR.png)

>Xóa người dùng `carlos`:
>![image](https://hackmd.io/_uploads/ByDctVpwC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ByDiY4pP0.png)

## **7. Lab: Password reset poisoning via dangling markup**
![image](https://hackmd.io/_uploads/HJgl0d6DA.png)

>Solved bài lab:
>![image](https://hackmd.io/_uploads/H1l2lAdaPC.png)
