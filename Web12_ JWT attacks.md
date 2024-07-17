---
title: 'Web12: JWT attacks'
tags: [web12]

---

I. TỔNG QUAN
----------------------

### 1\. Kiểu dữ liệu JSON
**JSON (JavaScript Object Notation)** là một định dạng dữ liệu nhẹ và độc lập với ngôn ngữ lập trình, được thiết kế để truyền tải và lưu trữ dữ liệu. JSON có định dạng giống như JavaScript Object, với các cặp **key - value** trong **dấu ngoặc nhọn { }** và được **phân cách bằng dấu phẩy**.

Ví dụ:
```
{
  "name": "John",
  "age": 30,
  "address": {
    "street": "123 Main St",
    "city": "Anytown",
    "state": "CA",
    "zip": "12345"
  },
  "phoneNumbers": [
    {
      "type": "home",
      "number": "555-555-1234"
    },
    {
      "type": "work",
      "number": "555-555-5678"
    }
  ]
}
```
### 2\. Khái niệm JWT (JSON Web Token)
Các phương pháp xác thực truyền thống như **cookie và session** được sử dụng rộng rãi trong ứng dụng web. Tuy nhiên, theo thời gian, với sự phát triển của các ứng dụng di động và các ứng dụng web ngày càng trở nên phức tạp hơn, dẫn đến một số vấn đề như sau:

* Mỗi khi người dùng xác thực thành công trong ứng dụng, hệ thống cần lưu trữ một bản ghi chứa thông tin xác thực này (phiên) trong bộ nhớ. Với sự gia tăng số lượng người dùng xác thực trong thời điểm ngắn, chi phí sử dụng cho việc lưu trữ và xử lý sẽ tăng lên đáng kể.
* Sau khi người dùng xác thực, nếu bản ghi được lưu trữ trong máy chủ đó sẽ đồng nghĩa với việc người dùng chỉ có quyền truy cập vào các tài nguyên cấp phép trên cùng máy chủ. Đồng nghĩa với việc ứng dụng sẽ bị giới hạn về khả năng mở rộng.
* Thông tin lưu trữ trong session thường không nhiều. Giới hạn về khả năng lưu trữ thông tin xác thực.
* Nếu session bị đánh cắp, ứng dụng và người dùng có nguy cơ bị tấn công CSRF, XSRF.

Từ đó, JSON web **tokens (JWT)** được thiết kế nhằm cung cấp một cách thức đơn giản và bảo mật để xác thực và trao đổi thông tin giữa các ứng dụng web và di động. JWT là một chuẩn mã hóa thông tin dựa trên định dạng JSON, được đưa vào các tiêu chuẩn Internet trong RFC 7519 vào năm  2015.

### 3\. Cấu tạo JWT (JSON Web Token)
![image](https://hackmd.io/_uploads/H1EzsqkPR.png)
Như hình vẽ, chúng ta thấy một chuỗi JWT được cấu tạo bởi ba bộ phận **header**, **payload, signature** theo thứ tự đó, phân cách nhau bởi dấu chấm `.`

`eyJraWQiOiIyNTg4NzYyMy1iNjljLTQ0ODEtYjQ3Ni1lNTk3MTdlZjg4ZGQiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Mzk1MzczNn0.yTR_bNCN4prjtvhkPy43n1CI93EOyxCP-xWtAgOrGSoIo0q2-nYhisGh5k0o-IfqlJGt6QrYSXykjvej_0_1WndSHUcv5rsC3b--37wyyNnyzVP3o37fFZOVxaLk6zpdpxeSJPmz0L1j1rCDeJqU73tIjpmwTulrxNq5zx-sO-AnopHaFzIfletzBAFQtrKETwaWc6rUtzVIL5BnkSTsrnQMFevg57CxfD8Qqvsu-ePfEPJ4xcmhSJcQLuriVUNmkFLmpGO2WRnrCgyga4uCIde0cdNsYN7UBrKs9E8akGxUU_HbRwmrBKaTzPDKrxQeV7nsb6TZOOexS0cFx7cJwg`

Chúng ta sẽ lần lượt tìm hiểu từng bộ phận.

#### 3.1\. **Header**
Bộ phận header trong JWT chứa thông tin về **loại token** và **thuật toán mã hóa** được sử dụng để tạo ra chữ ký số cho token. Nó được tạo thành bằng cách sử dụng **Base64 encode** đối tượng JSON. Thường bao gồm hai thuộc tính **alg (algorithm)** và **typ (type)**. Ta có thể decode Base64 để thấy rõ hơn, ví dụ header sau cho biết JWT sử dụng thuật toán mã hóa RSA-SHA256 (RS256) và type JWT đối với token JWT.

```
{
    "kid":"25887623-b69c-4481-b476-e59717ef88dd",
    "alg":"RS256",
    "typ": "JWT"
}
```

Khi xác thực JWT, phía bên nhận cần phải kiểm tra header để đảm bảo rằng token được mã hóa và giải mã bằng cùng một thuật toán.

#### 3.2\. **Payload**
Bộ phận payload **chứa thông tin liên quan đến người dùng** hoặc ứng dụng mà token đại diện. Giống với header, payload cũng được hiển thị dưới dạng mã hóa Base64. Nó chứa các thông tin khác nhau tùy thuộc vào mục đích sử dụng của JWT, nhưng thường bao gồm các thông tin như:

* `iss` (issuer): người phát hành token.
* `sub` (subject): chủ sở hữu của token, thường là ID người dùng.
* `aud` (audience): ứng dụng hoặc API sử dụng token.
* `exp` (expiration time): thời gian hết hạn của token.
* `nbf` (not before time): thời gian mà token không được sử dụng trước khi hợp lệ.
* `iat` (issued at time): thời gian phát hành token.
* `jti` (JWT ID): ID duy nhất cho mỗi token.
*
Các thuộc tính khác cũng có thể được thêm linh hoạt vào payload tùy thuộc vào mục đích sử dụng của JWT. Ví dụ, nếu JWT được sử dụng để xác thực người dùng, payload có thể chứa thông tin về tên người dùng, quyền hạn, thông tin liên lạc và các chi tiết khác về người dùng.

Ví dụ về payload trong JWT:

```
{
    "iss":"abc",
    "sub":"11d3-ae19-be29-8020",
    "admin": false,
    "email": "abc@gmail.com",
    "exp":1683953736
}
```

Payload trong JWT cung cấp cho phía bên nhận thông tin về người dùng hoặc ứng dụng được xác thực bởi token. Khi xác thực JWT, bên nhận có thể trích xuất thông tin từ payload để thực hiện các xử lý tùy thuộc vào mục đích sử dụng của JWT.

#### 3.3\. **Signature**
Hai bộ phận trước header và payload đều được mã hóa bằng Base64, giúp server front end có thể giải mã truy xuất nội dung dễ dàng. Phần signature trong JWT là một chuỗi được tạo ra bằng cách kết hợp Header và Payload với một Secret Key sử dụng một thuật toán hash.

Cụ thể, quá trình tạo signature trong JWT sử dụng thuật toán mã hóa hash như HMAC (Hash-based Message Authentication Code) hoặc RSA (Rivest-Shamir-Adleman) để tạo ra một chuỗi ký tự đại diện cho signature, được gắn vào cuối của JWT. Thuật toán HMAC sử dụng một khóa bí mật chung được biết đến bởi cả người tạo JWT và người xác thực JWT để tạo ra signature, trong khi thuật toán RSA sử dụng cặp khóa công khai và khóa riêng tư để thực hiện quá trình tạo signature.

Việc sử dụng signature giúp đảm bảo tính toàn vẹn (Integrity) của JWT và đảm bảo rằng không ai có thể sửa đổi dữ liệu trong Payload mà không bị phát hiện. Điều này đặc biệt quan trọng khi sử dụng JWT để truyền tải thông tin xác thực giữa các thành phần của hệ thống. Khi JWT được gửi đi, người nhận có thể kiểm tra signature để xác nhận tính hợp lệ của JWT.

Cuối cùng, các bạn có thể tổng hợp và ôn lại kiến thức từng bộ phận trong một token JWT qua hình ảnh sau:
![image](https://hackmd.io/_uploads/HJUw6ckDR.png)

Thoạt trông phức tạp là thế nhưng nếu hiểu, cấu trúc của một JWT chỉ đơn giản như sau:

`<base64-encoded header>.<base64-encoded payload>.<base64-encoded signature>`

Nói một cách khác, JWT là sự kết hợp (bởi dấu .) một Object Header dưới định dạng JSON được encode base64, một payload object dưới định dạng JSOn được encode base64 và một Signature cho URI cũng được mã hóa base64 nốt.

### 4\. JWT vs JWS vs JWE
![image](https://hackmd.io/_uploads/ryqqA9JwA.png)
1. **JWT (JSON Web Token)**

- Mục đích chính: JWT là một tiêu chuẩn mở để định nghĩa một cách thương mại các cách để trao đổi thông tin an toàn giữa các bên.
- Đặc điểm:
Là một chuỗi JSON được ký số để xác thực tính xác thực của một thông tin.
Thường được sử dụng cho việc xác thực và ủy quyền người dùng.
Không mã hóa dữ liệu bên trong, chỉ ký số (signature) để đảm bảo tính toàn vẹn của thông tin.
2. **JWS (JSON Web Signature)**
- Mục đích chính: JWS là một chuẩn để ký số dữ liệu JSON để xác thực tính toàn vẹn của dữ liệu và xác thực nguồn gốc của dữ liệu JSON.
- Đặc điểm:
Là một chuỗi JSON được ký số để xác thực tính toàn vẹn của một dữ liệu JSON.
Có thể sử dụng các thuật toán ký số như HMAC, RSA, ECDSA, etc.
Dữ liệu vẫn ở dạng plaintext.
3. **JWE (JSON Web Encryption)**
- Mục đích chính: JWE là một tiêu chuẩn để mã hóa các dữ liệu JSON để bảo vệ quyền riêng tư của dữ liệu trong khi chúng được truyền qua mạng.
- Đặc điểm:
Là một chuỗi JSON được mã hóa để bảo vệ quyền riêng tư của một dữ liệu JSON.
Có thể sử dụng các thuật toán mã hóa như RSA-OAEP, AES-GCM, etc.
Dữ liệu được mã hóa nên không thể đọc được trực tiếp mà không có khóa giải mã.
4. **So sánh chung:**
- Mục đích: JWT là chuẩn để trao đổi thông tin an toàn và ủy quyền người dùng. JWS và JWE là các phần mở rộng của JWT để cung cấp tính bảo mật hơn cho dữ liệu.
- Bảo mật: JWS thường được sử dụng để đảm bảo tính toàn vẹn của dữ liệu, trong khi JWE được sử dụng để bảo vệ quyền riêng tư của dữ liệu.
- Cơ chế: JWT dựa trên việc ký số (signature) để xác thực tính toàn vẹn, trong khi JWE dựa trên việc mã hóa dữ liệu để bảo vệ quyền riêng tư.

### 5\. Các tham số JOSE Headers và self-signed JWTs
#### 5.1\. JOSE Headers
**Headers** là một phần quan trọng của JWT, còn được gọi là JOSE (JSON Object Signing and Encryption) headers, chứa các thông tin xác định và định dạng của JWT. Theo tài liệu đặc tả về JSON Web Signature (JWS), JWT headers bao gồm một tập hợp các tham số và giá trị được đặt trong một đối tượng JSON. Các tham số này cung cấp thông tin quan trọng về việc mã hóa và xác minh JWT, bên cạnh hai tham số quen thuộc **alg** (thuật toán mã hóa) và **typ** (kiểu), JOSE headers còn chứa các trường tham số quan trọng sau:

* `jwk` (JSON Web Key): Được sử dụng để nhúng một đối tượng JSON biểu diễn một khóa.
* `jku` (JWK Set URL): Chỉ định URL chứa tập hợp các khóa công khai trong định dạng JSON Web Key (JWK).
* `kid` (Key ID): Được sử dụng để xác định một ID cho khóa công khai được sử dụng để xác minh chữ ký của JWT. ...

#### 5.2\. Tham số `jwk`
Khi sử dụng tham số `jwk` (JSON Web Key), người tạo JWT có thể nhúng một đối tượng JSON biểu diễn một khóa vào trong JWT headers. jwk chứa thông tin về khóa công khai được sử dụng để xác minh chữ ký của JWT (Thông thường là một cặp khóa public/private được tạo ra từ các thuật toán mã hóa RSA, ECDSA hoặc HMAC). Ví dụ:

```
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```

#### 5.3\. Tham số `jku`
Khi sử dụng tham số `jku` (JSON Web Key Set URL), người tạo JWT có thể cung cấp một URL trỏ đến tập hợp khóa công khai được sử dụng để xác minh chữ ký của JWT, thường có đường dẫn là /.well-known/jwks.json

```
{
    "alg": "RS256",
    "jku": "https://example.com/.well-known/jwks.json"
}
```

`jku` cho phép tách riêng việc nhúng khóa công khai, cho phép người nhận tải về khóa công khai từ một nguồn tin cậy. Điều này giúp giảm kích thước của JWT và giữ quá trình xác minh chữ ký dễ dàng và bảo mật hơn. Tuy nhiên, việc sử dụng tham số này yêu cầu sự tin cậy vào nguồn khóa công khai được chỉ định bởi URL.

#### 5.4\ Tham số `kid`
Trong JWT, tham số kid (Key ID) được sử dụng để xác định khóa công khai (public key) hoặc khóa bí mật (private key) trong xác minh chữ ký của JWT. `kid` giúp định danh và tìm kiếm khóa phù hợp trong trường hợp có nhiều khóa khác nhau được sử dụng, đồng thời cho phép hệ thống dễ dàng quản lý nhiều khóa khác nhau khi cần thiết.

#### 5.5\. Self-signed JWTs
**Self-signed JWTs (JSON Web Tokens)** là các JWT mà chữ ký được tạo và xác minh bằng cùng một khóa, không cần sử dụng khóa công khai của một bên thứ ba. Trong trường hợp này, người tạo JWT sẽ sử dụng một khóa bí mật riêng để ký JWT và sau đó xác minh chữ ký bằng cách sử dụng khóa bí mật đó.

Trong phương pháp này, người tạo và người xác minh JWT đều có cùng một khóa bí mật, nên không cần trao đổi khóa công khai hoặc tin tưởng bên thứ ba nào. Điều này đơn giản hóa quá trình triển khai và quản lý hơn, đồng thời giảm bớt sự phụ thuộc vào hạ tầng khóa công khai.

Tuy nhiên, một hạn chế của self-signed JWTs là sự thiếu tính chất xác minh bởi các bên thứ ba. Bởi vì không có khóa công khai được chia sẻ, người xác minh không thể đảm bảo rằng JWT được tạo bởi bên tin cậy và không bị chỉnh sửa trên đường truyền. Do đó, các bạn nên cân nhắc cài đặt ứng dụng với biện pháp self-signed JWTs.

II. TÁN CÔNG JWT
----------------------

**Tấn công JWT** là các phương pháp tấn công cố gắng tìm cách **xâm nhập, giả mạo, đánh cắp hoặc giải mã** các JWT không được cho phép. Dưới đây là một số phương pháp tấn công JWT phổ biến:

* **JWT Signature Spoofing**: Đây là phương pháp tấn công phổ biến nhất. Kẻ tấn công sẽ thay đổi chữ ký của JWT và tạo ra một JWT mới. Khi server xác thực JWT này, nó sẽ tin rằng JWT này được tạo ra bởi người dùng hợp lệ.
* **JWT Tampering**: Thay đổi các thông tin trong JWT, chẳng hạn như giá trị của các trường iss, sub, aud,... để tạo ra một JWT mới. Khi server xác thực JWT này, nó sẽ tin rằng JWT này được tạo ra bởi người dùng hợp lệ.
* **JWT Brute-force attacks**: Cố gắng tìm ra secret key sử dụng trong phần signature bằng cách thử tất cả các khả năng có thể. Từ đó có thể sửa đổi nội dung thông tin truyền đạt nhằm giả mạo người dùng.
* **JWT Encryption Attack**: Giả mạo một JWT mới bằng cách mã hóa thông tin từ một JWT khác đã được xác thực. Khi server giải mã JWT mới, nó sẽ tin rằng JWT này được tạo ra bởi người dùng hợp lệ.
* **JWT Timing Attack**: Sử dụng thời gian phản hồi từ server để suy ra các thông tin trong JWT. Kẻ tấn công có thể sử dụng kỹ thuật này để suy ra chữ ký hoặc mã hóa của JWT và tạo ra một JWT mới.

Tất cả các phương pháp tấn công JWT chủ yếu đều nhằm mục đích giả mạo JWT và đánh lừa server để tin rằng JWT này được tạo ra bởi người dùng hợp lệ.

III. Algorithm confusion attacks
----------------------
**Algorithm Confusion attacks** trong JWT (JSON Web Tokens) là một phương thức tấn công nhắm vào việc **lợi dụng sự nhầm lẫn hoặc thiếu sót** trong việc xác định thuật toán (algorithm) được sử dụng để ký hoặc xác minh JWT (Tham khảo thêm CVE-2016-5431/CVE-2016-10555). Trước hết, chúng ta sẽ tìm hiểu về hai dạng thuật toán thông dụng trong JWT: **Thuật toán đối xứng** (symmetric algorithms) và **bất đối xứng** (asymmetric algorithms).
### 1\. Thuật toán đối xứng và bất đối xứng trong JWT
![image](https://hackmd.io/_uploads/BySHD1-PC.png)

**Thuật toán đối xứng** (Symmetric algorithms) sử dụng **cùng một khóa** (secret key) cho cả quá trình ký (sign) và xác minh (verify) JWT. Một ví dụ tiêu biểu là HS256 (HMAC-SHA256), trong đó HMAC (Hash-based Message Authentication Code) được sử dụng để ký và xác minh chữ ký. Với thuật toán đối xứng, cả server và client đều biết khóa bí mật.
![image](https://hackmd.io/_uploads/BJjowkbw0.png)
Mặt khác, **thuật toán bất đối xứng** (Asymmetric algorithms) sử dụng một **cặp khóa** gồm **khóa riêng tư** (private key) và **khóa công khai** (public key) để thực hiện quá trình ký và xác minh JWT. Khi server tạo JWT, nó sẽ sử dụng khóa riêng tư để ký chữ ký. Sau đó, client sử dụng khóa công khai tương ứng để xác minh chữ ký.

Trong JWT, thuật toán được chỉ định trong phần header của token. Nếu thuật toán không được xác định hoặc không được hỗ trợ, quá trình xác minh chữ ký sẽ thất bại.

Việc chọn thuật toán phù hợp là rất quan trọng để đảm bảo tính toàn vẹn và bảo mật của JWT. Thuật toán đối xứng và bất đối xứng đều có ưu điểm và hạn chế riêng, và lựa chọn thuật toán phụ thuộc vào ngữ cảnh và yêu cầu bảo mật của ứng dụng.

### 2\. Sai sót dẫn đến lỗ hổng algorithm confusion attack
JWT cho phép các thuật toán khác nhau được sử dụng để ký và xác minh chữ ký. Tuy nhiên, khi không kiểm tra hoặc kiểm tra không chính xác thuật toán được chỉ định trong phần header của JWT, người tấn công có thể thay đổi thuật toán để tận dụng các lỗ hổng liên quan đến việc triển khai hoặc xử lý JWT.

Xem xét đoạn code sau:

```
function verify(token, secretOrPublicKey){
    algorithm = token.getAlgHeader();
    if(algorithm == "RS256"){
        // Use the provided key as an RSA public key
    } else if (algorithm == "HS256"){
        // Use the provided key as an HMAC secret key
    }
}
```

Chương trình trên chấp nhận xác thực JWT do người dùng cung cấp trong cả hai trường hợp token sử dụng thuật toán RS256 (bất đối xứng) và HS256 (đối xứng). Khi đó, nếu ứng dụng bị lộ RSA public key, kẻ tấn công có thể chuyển thể sang dạng PEM, mã hóa Base64 và sử dụng như secret key của thuật toán đối xứng (Điều ngược lại cũng có thể thực hiện).

IV. CÁC BÀI LAB
----------------------

# **1. Lab: JWT authentication bypass via unverified signature**
This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.
To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user carlos.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/SJxp4okvC.png)

>Đăng nhập vào `wiener`, xem lại respone thì thấy rằng sau khi đăng nhập, ta được cấp 1 JWT lưu vào session: 
>![image](https://hackmd.io/_uploads/Hkj-IjJPC.png)
![image](https://hackmd.io/_uploads/H11NUiyvA.png)

>Khi truy cập vào `/admin` thì ta bị chặn, nó chỉ cho phép `administrator` truy cập: 
>![image](https://hackmd.io/_uploads/H1VWhoywR.png)

>Để ý rằng `username` được chứa trong tham số `sub`:
>![image](https://hackmd.io/_uploads/BJWY2oyP0.png)

>Send request `GET /admin` tới Repeater, sử dụng extension **JWT Editor**, ta tiến hành chỉnh sửa nội dung trường `sub` thành username khác, cụ thể là `administrator`:
>![image](https://hackmd.io/_uploads/B1IxpsJD0.png)

>Gửi lại request truy cập trang `/admin` thì ta thấy đã truy cập thành công mà không còn bị chặn => chứng tỏ trang web dính lỗ hổng không signature JWT, mà không verify nốt, chỉ decode data!

>Cuối cùng, lấy đường dẫn để xóa người dùng `carlos` và Send: 
>![image](https://hackmd.io/_uploads/ryOPpskPA.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/rJNuaskDR.png)

# **2. Lab: JWT authentication bypass via flawed signature verification**
>This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to accept unsigned JWTs.
To solve the lab, modify your session token to gain access to the admin panel at /admin, then delete the user carlos.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/rkLlRsJwR.png)

>Tiếp tục như lab trước, nhưng lần này khi đổi `sub` thành `administrator` và gửi lên `/admin` thì đã bị server từ chối, do server đã verify JWT mình gửi không khớp:
>![image](https://hackmd.io/_uploads/S1Kol21DA.png)

>Ta có thể thay đổi giá trị của tham số `alg` trong Header. Tham số này sẽ cho ta biết server sử dụng thuật toán mã hóa nào. Thay đổi giá trị thành `none` và xóa luôn phần `Signature` đi, để đánh lừa server, do server chứa lỗ hổng không xác minh chữ kí một cách đúng đắn, vẫn chấp nhận 1 JWT không được kí:
>![image](https://hackmd.io/_uploads/ryAcMhkv0.png)

>Có thể thấy thành công bypass và truy cập được vào `/admin`. Lấy đường dẫn để xóa người dùng `carlos` và Send: 
>![image](https://hackmd.io/_uploads/HyB0GhyPR.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/HJBJ73kPR.png)

# **3. Lab: JWT authentication bypass via weak signing key**
>This lab uses a JWT-based mechanism for handling sessions. It uses an extremely weak secret key to both sign and verify tokens. This can be easily brute-forced using a wordlist of common secrets.
To solve the lab, first brute-force the website's secret key. Once you've obtained this, use it to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/ryVgOAyw0.png)

>Mô tả cho ta biết rằng secret key để signature JWT trong lab này có thể sử dụng tấn công brute-force để tìm ra, từ đó có thể thay đổi payload bên trong...

>Đăng nhập vào `wiener`, sau khi login thì ta được server cấp cho 1 JWT set làm Cookie để xác thực người dùng: 
>![image](https://hackmd.io/_uploads/SkkWYRkv0.png)

>Ta sẽ sử dụng tool **hashcat** trên kali để thực hiện việc vét cạn tìm ra secret_key của token này. Một câu lệnh hashcat có cấu trúc như sau:

`hashcat [options]... hash|hashfile|hccapxfile [dictionary|mask|directory]...`

>Sử dụng command `hashcat --help` để xem chi tiết hướng dẫn: 
>![image](https://hackmd.io/_uploads/rJT3FCJPR.png)

>Sử dụng option `-m 16500` tương ứng với JWT:
>![image](https://hackmd.io/_uploads/S1C7cA1v0.png)

>Tiếp theo, sử dụng option `-a` (--attack-mod) lựa chọn phương thức tấn công. Tìm kiếm cách tấn công Brute-force:
>![image](https://hackmd.io/_uploads/BkEtqCJDA.png)

>Và cuối cùng là tập secret_key sẵn để brute-force, đề bài hướng dẫn lấy tại:
>`https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list`
>![image](https://hackmd.io/_uploads/HkP8sC1vR.png)

>Ta có command hoàn chỉnh như sau:
>`hashcat -a 3 -m 16500 <MY-JWT> /path/to/jwt.secrets.list`
![image](https://hackmd.io/_uploads/HkBMnAJvR.png)

>Chạy xong, ta có kết quả tìm được là `secret1`:
>![image](https://hackmd.io/_uploads/By9eZylwC.png)

>Có được secret-key rồi, giờ ta sẽ đi sinh khóa giả mạo :) Đầu tiên, Base64-encode chuỗi nhận được ( `secret1`):
>![image](https://hackmd.io/_uploads/BJq4N1xwC.png)

>Chọn Extension **JWT Editor** Keys rồi chọn **New Symmetric Key**. Trong dialog, click **Generate** để gen ra key mới. Thay giá trị của trường `k` bằng chuỗi base64 ta vừa tạo từ key:
>![image](https://hackmd.io/_uploads/BJJ9EJxP0.png)

>Cuối cùng, sửa đổi nội dung payload trường `sub` thành `administrator` và click Sign để kí:
>![image](https://hackmd.io/_uploads/ryEjIkgwC.png)

>Hoặc ta có thể kí tại `https://jwt.io/`:
>![image](https://hackmd.io/_uploads/Hk5ka1xw0.png)

>Send lại request và thành công truy cập vào trang admin:
>![image](https://hackmd.io/_uploads/B1iphylw0.png)

>Lấy đường dẫn xóa người dùng `carlos` và Send:
>![image](https://hackmd.io/_uploads/SyhbpJgP0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkcMaylDA.png)

# **4. Lab: JWT authentication bypass via jwk header injection**
>This lab uses a JWT-based mechanism for handling sessions. The server supports the jwk parameter in the JWT header. This is sometimes used to embed the correct verification key directly in the token. However, it fails to check whether the provided key came from a trusted source.
To solve the lab, modify and sign a JWT that gives you access to the admin panel at /admin, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/rJ5jQlxD0.png)

>Đăng nhập vào `wiener`, sau khi login thì ta được server cấp cho 1 JWT set làm Cookie để xác thực người dùng: 
>![image](https://hackmd.io/_uploads/HJdBDgxP0.png)

>Thử truy cập vào `/admin` nhưng đã bị chặn:
>![image](https://hackmd.io/_uploads/rkxJuxgw0.png)

>Chú ý giá trị `kid` trong headers, có thể dự đoán trang web sử dụng `kid` thực hiện xác thực bằng danh sách **public keys** được lưu trữ tại server. Tham số `alg` cho biết thuật toán sử dụng ở đây là **RSA**.
>![image](https://hackmd.io/_uploads/Byw8veevA.png)

>Chúng ta có thể tự xây dựng một RSA key theo format jwk để nhúng vào phần headers theo các bước sau:
>![image](https://hackmd.io/_uploads/BkQ6DgxwA.png)

>Send request tới `/admin` đến Repeater, ta có thể thực hiện nhúng thêm `jwk` do ta custom vào header như sau:
>![image](https://hackmd.io/_uploads/H1RN_elP0.png)

>Chọn Signing key là cái mình vừa tạo ở trên rồi OK:
>![image](https://hackmd.io/_uploads/H1XL_eeDC.png)

>Tiếp đó, chỉnh sửa giá trị tham số `sub` ở phần Payload thành `administrator` rồi thực hiện **Sign**, cuối cùng là Send request:
>![image](https://hackmd.io/_uploads/BkTXYelwC.png)

>Server trả về **200 OK**, chứng tỏ ta đã thành công mạo danh admin và truy cập trang quản trị thành công!

>Tìm đường dẫn xóa người dùng `carlos` và Send:
>![image](https://hackmd.io/_uploads/HyxFYxeDC.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/Bk95KllDC.png)

# **5. Lab: JWT authentication bypass via jku header injection**
>This lab uses a JWT-based mechanism for handling sessions. The server supports the jku parameter in the JWT header. However, it fails to check whether the provided URL belongs to a trusted domain before fetching the key.
To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/HJ3Lsgxw0.png)

>Đăng nhập vào `wiener`, sau khi login thì ta được server cấp cho 1 JWT set làm Cookie để xác thực người dùng: 
>![image](https://hackmd.io/_uploads/Byxl3eevA.png)

>Thử truy cập vào `/admin` nhưng đã bị chặn:
>![image](https://hackmd.io/_uploads/HJ8Z3xlv0.png)
![image](https://hackmd.io/_uploads/HkGfneeDA.png)

>Việc sử dụng public key được nhúng trong jwt có thể chứa nhiều rủi ro, bởi vậy một số ứng dụng sử dụng tham số jku nhằm xác định một URL tham chiếu tới một bộ khóa công khai được đặt ở server.

>Tuy nhiên, việc triển khai không đúng cách có thể tạo ra lỗ hổng bảo mật nghiêm trọng. Một cuộc tấn công tham số jku trong self-signed JWTs thường xảy ra một kẻ tấn công giả mạo JWT bằng cách thay đổi giá trị của jku để trỏ đến một URL mà kẻ tấn công kiểm soát. Khi truy cập tới URL này, ứng dụng sẽ lấy và sử dụng các public keys do kẻ tấn công tạo ra.

>Chúng ta sẽ kiểm tra server có chấp nhận tham số `jku` hay không. Thực hiện thêm tham số `jku` trong phần Header JWT với giá trị sinh từ Collaborator client:
>![image](https://hackmd.io/_uploads/Hk_P6xlw0.png)

>Thấy rằng server sẽ redirect ta về trang `/login`, nhưng bên collab vẫn thấy có các request gửi đến:
>![image](https://hackmd.io/_uploads/S195aggvR.png)

>Điều này chứng tỏ ứng dụng xác thực danh tính người dùng bằng cách truy cập tới một URL chứa danh sách các public keys, kiểm tra ánh xạ qua giá trị tham số `kid` trong JWT. Tuy nhiên, khi tham số `jku` tồn tại trong JWT, giá trị URL này được ghi đè, dẫn đến ứng dụng sẽ truy cập tới `jku` trong JWT.

>Do sự cài đặt sai sót này, chúng ta có thể tự dựng một trang web chứa danh sách các public keys, từ đó giả mạo tham số `jku` để ứng dụng truy cập tới trang web giả mạo đó. Sử dụng exploit server mà lab cho sẵn:
>![image](https://hackmd.io/_uploads/ByIOAxeDA.png)

>Sử dụng **JWT Editor Keys** sinh một bộ khóa RSA dạng JWK:
>![image](https://hackmd.io/_uploads/ry42AexDA.png)

>Lưu payload vào trong server, đồng thời ta có thể đặt tên đường dẫn tùy ý, chẳng hạn `/public-key.json`, ví dụ:
```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
            "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw"
        }
    ]
}

```
>![image](https://hackmd.io/_uploads/Sy3lJbxw0.png)

>Khi truy cập tới `/public-key.json` có kết quả:
>![image](https://hackmd.io/_uploads/rkYmkbxD0.png)

>Trong phần Header JWT, thêm tham số `jku` có giá trị là đường dẫn URL exploit server, đồng thời thay đổi giá trị `kid` ánh xạ với bộ khóa trong exploit server. Thay đổi tham số `sub` trong phần Payload thành `administrator`. Chọn **Sign** với bộ key chúng ta đã sinh:
>![image](https://hackmd.io/_uploads/HJs7bZev0.png)

>Tìm đường dẫn xóa người dùng `carlos` và Send:
>![image](https://hackmd.io/_uploads/BkhHbWgwA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/SyT8Z-ewC.png)

### Tấn công tham số kid trong self-signed JWTs
Ngoài việc được sử dụng để xác định khóa công khai (public key) hoặc khóa bí mật (private key) trong xác minh chữ ký của JWT, `kid` có thể được chỉ định giá trị đường dẫn trỏ đến tệp chứa thông tin khóa xác minh. Kẻ tấn công có thể nhắm vào tham số này với kỹ thuật** Directory traversal**, khiến trang web tìm tới các tệp tùy ý trong server. Ví dụ đoạn mã PHP sau đây mô phỏng việc xác thực và xử lý JWT trên máy chủ chưa an toàn:
```
// Lấy JWT từ request
$jwt = $_GET['token'];

// Giải mã JWT
$decoded = jwt_decode($jwt);

// Lấy giá trị của kid từ JWT
$kid = $decoded['kid'];

// Kiểm tra và lấy khóa xác minh tương ứng với kid
$verificationKey = getVerificationKey($kid);

// Xác minh chữ ký của JWT bằng khóa xác minh
$isValid = verifySignature($jwt, $verificationKey);

if ($isValid) {
    echo "JWT is valid!";
} else {
    echo "JWT is invalid!";
}

// Hàm giả lập giải mã JWT
function jwt_decode($jwt) {
    // Giả sử JWT được giải mã thành mảng dữ liệu
    return json_decode(base64_decode($jwt), true);
}

// Hàm giả lập lấy khóa xác minh từ kid
function getVerificationKey($kid) {
    // Lấy khóa xác minh từ kid (lỗ hổng: không kiểm tra kid)
    $verificationKey = file_get_contents($kid);
    return $verificationKey;
}

// Hàm giả lập xác minh chữ ký JWT
function verifySignature($jwt, $verificationKey) {
    // Giả sử thực hiện xác minh chữ ký
    // Trong trường hợp này, chúng ta không kiểm tra tính hợp lệ của khóa xác minh (lỗ hổng)
    return true;
}

```
Trong đoạn mã trên, ứng dụng không kiểm tra tính hợp lệ của `kid` khi lấy khóa xác minh, dẫn đến kẻ tấn công có thể sử dụng tham số `kid` để truy cập vào các tệp tin tùy ý trên hệ thống và làm khóa xác minh. Một trong những phương pháp đơn giản nhất để tận dụng lỗ hổng này là sử dụng đường dẫn tệp tin `/dev/null`, một tệp tin trống rỗng có mặt trên hầu hết các hệ thống Linux. Khi server đọc nó sẽ trả về một chuỗi rỗng. Do đó, kẻ tấn công có thể ký JWT bằng một chuỗi rỗng dẫn tới trang web xác minh JWT chứa chữ ký hợp lệ.

# **6. Lab: JWT authentication bypass via kid header path traversal**
>This lab uses a JWT-based mechanism for handling sessions. In order to verify the signature, the server uses the kid parameter in JWT header to fetch the relevant key from its filesystem.
To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/Bk949HxD0.png)

>Đăng nhập vào `wiener`, sau khi login thì ta được server cấp cho 1 JWT set làm Cookie để xác thực người dùng: 
>![image](https://hackmd.io/_uploads/S1nMwUeP0.png)

>Thử truy cập vào `/admin` nhưng đã bị chặn:
>![image](https://hackmd.io/_uploads/BkgND8xDC.png)

>Thấy được rằng JWT được kí bằng thuật toán SHA256:
>![image](https://hackmd.io/_uploads/r1IPvUlDA.png)

>Nên đầu tiên ta sẽ tạo New Symetric Key:
>![image](https://hackmd.io/_uploads/BkAiFLlPA.png)

>Param k là `AA==` là `key = null` byte để server có thể decrypt.
Sửa tham số kid thành `../../../../../../../dev/null` và giá trị cho phần `sub` thành ` administrator`, và Sign bằng key ta vừa tạo, khi decrypt jwt, nó sẽ đi tìm `kid` ở vị trí đó và check xem file đó có tồn tại không (`/dev/null`)
![image](https://hackmd.io/_uploads/Hkuve1bPC.png)

>Ta thấy đã thành công truy cập vào `/admin`, tìm đường dẫn xóa `carlos` và Send request:
>![image](https://hackmd.io/_uploads/Hk2fWk-vR.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/SknmW1WPA.png)

# **7. Lab: JWT authentication bypass via algorithm confusion**
>This lab uses a JWT-based mechanism for handling sessions. It uses a robust RSA key pair to sign and verify tokens. However, due to implementation flaws, this mechanism is vulnerable to algorithm confusion attacks.
To solve the lab, first obtain the server's public key. This is exposed via a standard endpoint. Use this key to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/HkAfF1WwR.png)

>Sử dụng tool `dirb` của Kali và tìm thấy web tồn tại 1 đường dẫn `/jwks.json` chứa toàn bộ các public key mà server lưu trữ:

>Truy cập đường dẫn đó, ta lấy được public key:
>![image](https://hackmd.io/_uploads/r1e_3ybPA.png)

```
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "84d10779-563a-4e42-8dd5-7b38e6df6429",
      "alg": "RS256",
      "n": "rlQ_vQ1znLK04jJ_APHoo4bs95PV3ZIY6dn4vBnWI0C0L241UEDoDJJzrauesaHqoYpFQsfIEfXinq2yiGOPsB2m0II2zJhJ9CaDTQYc3_roMip3T4lpp0u5Zrp1d-aM8M7uq_zW1d8xGHoFOTIBJnzFrF2ktKG7tC9IwoMp9ERVEuSnueH_KeUKrOxknEX0V4yNZaZDwfAgXQBhgoE46inl8RfmZ5UpQHJfhxO286vnQWr11mhdwQdNLeLKrYCdJMjKvQQnmZSC4tntJX_BmMwNvKJkP-mg72FUUo70-P-qDVTLwSYuJPfJIWQOcyZZWaF6rNbrfn5zyLwHTi5ygQ"
    }
  ]
}
```

>Đăng nhập vào `wiener`, nhận thấy JWT sử dụng thuật toán bất đối xứng RS256:
>![image](https://hackmd.io/_uploads/HJLa3yZDA.png)

>Có thể dự đoán ứng dụng truy cập tới đường dẫn `/jwks.json` tìm kiếm bộ public key tương ứng với tham số `kid` trong JWT để xác thực người dùng.
Chúng ta sẽ chuyển thể bộ public key của thuật toán bất đối xứng RS256 sang chuỗi secret key của thuật toán đối xứng HS256.

>Tại extension **JWT Editor Keys** > chọn **New RSA Key** > Sao chép bộ public key trên `(kid)` và dán vào phần key, click OK:
![image](https://hackmd.io/_uploads/HyGHUeZvR.png)

>Bên ngoài, chuột phải và chọn **Copy Public Key as PEM**:
>![image](https://hackmd.io/_uploads/SJJjIgWvR.png)


>Base64-encode chuỗi vừa copy:
>![image](https://hackmd.io/_uploads/HyBa8lZvA.png)

>Truy cập `https://jwt.io/`, dán JWT người dùng `wiener` vào phần **Encoded** > Thay đổi tham số **alg** thành **HS256** > Dán secret key (dạng Base64) vào ô **SIGNATURE** > Trang web thông báo **Signature Verified**:
>![image](https://hackmd.io/_uploads/Sy7MPg-wC.png)

>Chứng tỏ secret key là chính xác, thay đổi phần `sub` thành `administrator`. Copy JWT vừa tạo và thay vào request, Send:
>![image](https://hackmd.io/_uploads/Bk6PDl-vC.png)

>Thành công mạo danh admin và có thể truy cập vào được trang quản trị!!
>Lấy đường dẫn để xóa `carlos` và Send request:
>![image](https://hackmd.io/_uploads/SJTqDebPA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rJDsDxZPC.png)

# **8. Lab: JWT authentication bypass via algorithm confusion with no exposed key**
>This lab uses a JWT-based mechanism for handling sessions. It uses a robust RSA key pair to sign and verify tokens. However, due to implementation flaws, this mechanism is vulnerable to algorithm confusion attacks.
To solve the lab, first obtain the server's public key. Use this key to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`
![image](https://hackmd.io/_uploads/r1DTKxWwA.png)

>Trong trường hợp chúng ta không thể thu thập thông tin về bộ public key của ứng dụng, chúng ta hoàn toàn có thể sử dụng một số công cụ tìm thử sai và tìm ra bộ public key của ứng dụng (trong trường hợp giá trị n nhỏ), ví dụ chương trình sig2n.py do Portswigger gợi ý: https://github.com/silentsignal/rsa_sign2n/blob/release/sig2n.py.

>Chương trình dựa vào hai JWT sinh ra sau hai lần đăng nhập khác nhau, tính toán một hoặc nhiều giá trị tiềm năng của n để kiểm tra chúng có khớp với giá trị của n trong bộ khóa của máy chủ hay không, từ đó trả về chuỗi JWT giả mạo cùng với bộ public key giả mạo tương ứng.

>Lab này nói rằng ta có thể dựa vào 2 JWT hợp lệ, từ đó có thể dò được public key (do `n` nhỏ dẫn tới độ phức tạp của thuật toán yếu)

>Đầu tiên, login vào `wiener` và copy JWT thứ nhất, sau đó log-out và log-in lại 1 lần nữa, copy JWT thứ 2.

>Ta sẽ sử dụng tool có sẵn được build bằng Docker của Port Swigger trên CLI bằng command sau:
>`docker run --rm -it portswigger/sig2n <token1> <token2>
`
![image](https://hackmd.io/_uploads/B1kOgZbPA.png)

>Trong đó, **Tampered JWT** là chuỗi JWT giả mạo và **Base64 encoded x509** key là bộ public key tương ứng. Copy chuỗi giả mạo và thử truy cập vào `/my-acount`:
>![image](https://hackmd.io/_uploads/BynW-WWD0.png)

>Có thể thấy rằng kết quả trả về là **"200 OK"**, chứng tỏ rằng key này hoạt động tốt. Tiếp theo, ta sẽ đi sinh ra **signing key** mới -> Copy Base64-encode x509 ở trên, tạo New Symmetric Key mới với tham số `k` thay bằng chuỗi vừa copy:
>![image](https://hackmd.io/_uploads/SJQ6W-WDC.png)

>Tiếp theo, thay `alg` thành `HS256` và giá trị tham số `sub` thành `administrator` và Sign lại bằng key mình vừa tạo ở bước trên. Tiến hành Send request:
>![image](https://hackmd.io/_uploads/SywrM-ZvC.png)

>Thấy rằng ta đã thành công truy cập tới `/admin` , tìm đường dẫn để xóa `carlos` và send request: 
>![image](https://hackmd.io/_uploads/rJLYzbWwA.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/BJG9MW-DR.png)
