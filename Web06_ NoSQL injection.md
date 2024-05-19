Giới thiệu về NoSQL
===============
**NoSQL** là một thuật ngữ tổng quát được sử dụng để mô tả các hệ thống quản lý cơ sở dữ liệu (DBMS) không sử dụng mô hình quan hệ SQL (Structured Query Language) để tổ chức dữ liệu. Thay vì đó, NoSQL thường tập trung vào các mô hình lưu trữ dữ liệu linh hoạt hơn và có thể mở rộng tốt khi xử lý dữ liệu lớn và phân tán. Có nhiều loại NoSQL databases, bao gồm:
*  **Cơ sở dữ liệu Key-Value** (Ví dụ: Redis, DynamoDB): Lưu trữ dữ liệu dưới dạng cặp key-value, giúp truy xuất dữ liệu nhanh chóng.
*  **Cơ sở dữ liệu Document** (Ví dụ: MongoDB, CouchDB): Lưu trữ dữ liệu dưới dạng các tài liệu (document), thường sử dụng định dạng như JSON hoặc BSON.
*  **Cơ sở dữ liệu Wide-Column Store** (Ví dụ: Apache Cassandra, HBase): Dữ liệu được tổ chức thành các cột thay vì hàng, giúp tối ưu cho việc truy xuất dữ liệu theo cột.
*  **Cơ sở dữ liệu Graph** (Ví dụ: Neo4j, ArangoDB): Dùng để lưu trữ dữ liệu có mối quan hệ phức tạp, ví dụ như mạng lưới mối quan hệ giữa các đối tượng.
![image](https://hackmd.io/_uploads/ByW-rPQGA.png)

**Ví dụ**, nếu bạn sử dụng MongoDB, bạn có thể lưu trữ thông tin về một người dùng trong một collection (tương đương với bảng trong SQL) dưới dạng tài liệu JSON như sau:
```
{
  "_id": ObjectId("5a934e000102030405000000"),
  "username": "example_user",
  "email": "user@example.com",
  "age": 25,
  "address": {
    "city": "Example City",
    "country": "Example Country"
  }
}
```
Ở đây, mỗi người dùng được biểu diễn bằng một tài liệu JSON với các trường khác nhau, và không có cấu trúc cố định cho mỗi người dùng.

Dưới đây là một số ví dụ về truy vấn NoSQL, sử dụng MongoDB làm cơ sở dữ liệu document-based. Đối với MongoDB, bạn có thể sử dụng ngôn ngữ truy vấn giống JSON, được gọi là BSON.

**Truy vấn Tìm kiếm:**
_**Tìm tất cả các người dùng có độ tuổi lớn hơn 21:**_

```json
db.users.find({ "age": { $gt: 21 } })
```

_**Tìm người dùng có tên đăng nhập là "exampleuser":**_

```json
db.users.find({ "username": "example_user" })
```

**Truy vấn Thêm dữ liệu:**

_**Thêm một người dùng mới:**_

```json
db.users.insertOne({
  "username": "new_user",
  "email": "new_user@example.com",
  "age": 30,
  "address": {
    "city": "New City",
    "country": "New Country"
  }
})
```
**Truy vấn Cập nhật dữ liệu:**

_**Cập nhật độ tuổi của người dùng có tên đăng nhập là "exampleuser":**_

```json
db.users.updateOne({ "username": "example_user" }, { $set: { "age": 26 } })

```

**Truy vấn Xóa dữ liệu:**

_**Xóa tất cả các người dùng có độ tuổi dưới 18:**_

```json
db.users.deleteMany({ "age": { $lt: 18 } })
```
NoSQL Injection
===============
Vậy liệu NoSQL Injection có phải là No SQL Injection (không bị SQL Injection)? Câu trả lời là không đúng. Khi bạn sử dụng NoSQL, bạn vẫn có thể bị tấn công injection như bình thường.

NoSQL injection là một lỗ hổng xảy ra khi một kẻ tấn công có khả năng can thiệp vào các truy vấn mà một ứng dụng thực hiện đến cơ sở dữ liệu NoSQL. NoSQL injection có thể cho phép kẻ tấn công:

-   Vượt qua các cơ chế xác thực hoặc bảo vệ.
-   Trích xuất hoặc chỉnh sửa dữ liệu.
-   Tấn công từ chối dịch vụ.
-   Thực thi mã tùy ý trên máy chủ.
![image](https://hackmd.io/_uploads/r1ZAHwQfR.png)

Có hai loại NoSQL injection khác nhau:

**Syntax injection** \- Lỗ hổng này xảy ra khi bạn có thể phá vỡ cú pháp truy vấn NoSQL, cho phép bạn chèn vào payload của mình. Phương pháp này tương tự như trong SQL injection. Tuy nhiên, tính chất của cuộc tấn công khác nhau đáng kể, vì cơ sở dữ liệu NoSQL sử dụng nhiều ngôn ngữ truy vấn, loại cú pháp truy vấn khác nhau và cấu trúc dữ liệu khác nhau.

**Operator injection** \- Lỗ hổng này xảy ra khi bạn có thể sử dụng các toán tử truy vấn NoSQL để thao tác truy vấn.

# NoSQL syntax Injection
Chúng ta có thể kiểm tra lỗ hổng NoSQL injection có thể xảy ra bằng cách thử tìm cách thay đổi nhằm phá vỡ cú pháp truy vấn. Để làm điều này, chúng ta có thể fuzzing và ký tự đặc biệt có thể gây ra lỗi cơ sở dữ liệu. Chúng ta có thể thu thập thông tin ngôn ngữ API của cơ sở dữ liệu để từ đó sử dụng ký tự đặc biệt và chuỗi fuzz phù hợp với ngôn ngữ đó.

### Phát hiện lỗ hổng syntax injection trong MongoDB

Giả sử có một ứng dụng mua sắm hiển thị sản phẩm trong các danh mục khác nhau. Khi người dùng chọn danh mục "fizzy", trình duyệt của họ yêu cầu URL sau:
`https://insecure-website.com/product/lookup?category=fizzy`
Điều này khiến ứng dụng gửi một truy vấn JSON để lấy các sản phẩm liên quan từ bộ sưu tập sản phẩm trong cơ sở dữ liệu MongoDB:
`this.category == 'fizzy'`
Để kiểm tra xem ứng dụng có thể bị lỗ hổng không, hãy gửi một chuỗi fuzz trong giá trị của tham số `category`. Một chuỗi mẫu cho MongoDB có thể là:
```none
'"`{
;$Foo}
$Foo \xYZ
```
Sử dụng chuỗi fuzz này để tạo ra cuộc tấn công như sau:

https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00

Khi chèn các ký tự đặc biệt, ứng dụng trả về response khác so với truy vấn ban đầu điều này có thể chỉ ra rằng đầu vào người dùng không được lọc hoặc làm sạch đúng cách.

Trong một số ứng dụng khác, chúng ta có thể cần chèn payload của mình thông qua một thuộc tính JSON. Trong trường hợp này, payload sẽ trở thành: 
> '\"`{\r;$Foo}\n$Foo \\xYZ\u0000`
### a. Xác định ký tự nào được ứng dụng xử lý

Để xác định ký tự nào được ứng dụng hiểu là cú pháp, bạn có thể chèn từng ký tự một. Ví dụ, bạn có thể gửi `'`, dẫn đến truy vấn MongoDB như sau:

```
this.category == '''
```

 

Nếu điều này gây ra thay đổi so với phản hồi ban đầu, điều này có thể chỉ ra rằng ký tự ' đã làm hỏng cú pháp truy vấn và gây ra lỗi cú pháp. Bạn có thể xác nhận điều này bằng cách gửi một chuỗi truy vấn hợp lệ trong đầu vào, ví dụ như việc thoát khỏi dấu nháy đơn sử dụng `\'`:

```javascript

this.category == '\''
```

Nếu điều này không gây ra lỗi cú pháp, điều này có thể có nghĩa là ứng dụng có thể bị lỗ hổng NoSQL injection.

### b. Kiểm tra điều kiện câu truy vấn

Sau khi phát hiện một lỗ hổng, bước tiếp theo là xác định liệu bạn có thể ảnh hưởng đến các điều kiện boolean bằng cú pháp NoSQL hay không.

Để kiểm thử điều này, gửi hai yêu cầu, một với điều kiện sai và một với điều kiện đúng. Ví dụ, bạn có thể sử dụng các câu lệnh điều kiện **False**: `' && 0 && 'x` và **True**: `' && 1 && 'x` như sau:

```html
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x

```
```html
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x

```

Nếu ứng dụng trả về kết quả khác nhau, điều này có thể thấy rằng điều kiện sai ảnh hưởng đến logic truy vấn.

### c. Ghi đè các điều kiện hiện tại

Sau khi bạn đã xác định bạn có thể ảnh hưởng đến điều kiện boolean, bạn có thể thử ghi đè các điều kiện hiện tại để lợi dụng lỗ hổng. Ví dụ, bạn có thể chèn một điều kiện luôn đúng, như `'||1||'`:

```html
https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%31%7c%7c%27

```

Điều này dẫn đến truy vấn MongoDB như sau:

```html
this.category == 'fizzy'||'1'=='1'
```
Vì điều kiện được chèn luôn là đúng, truy vấn đã được sửa đổi trả về tất cả các mục. Điều này cho phép bạn xem tất cả các sản phẩm trong bất kỳ danh mục nào, bao gồm cả các danh mục ẩn hoặc không biết.

### d. Sử dụng ký tự null

Chúng ta có thể sử dụng ký tự `null` (`%00`) vào câu truy vấn để kiểm tra xem ứng dụng có thực hiện xử lý ký tự này không.

Trong ví dụ trên, truy vấn MongoDB gốc bao gồm một điều kiện rằng sản phẩm phải thuộc danh mục `'fizzy'` và phải đã phát hành `(this.category == 'fizzy' && this.released == 1)`. Tuy nhiên, bằng cách chèn một ký tự null vào tham số danh mục `https://insecure-website.com/product/lookup?category=fizzy'%00`, truy vấn NoSQL kết quả trở thành:

```html
this.category == 'fizzy'\u0000' && this.released == 1
```
Nếu MongoDB thực sự bỏ qua tất cả các ký tự sau ký tự null, điều này hủy bỏ phần còn lại của truy vấn, kết quả là, tất cả sản phẩm trong danh mục 'fizzy' đều được hiển thị, kể cả những sản phẩm chưa được phát hành.


# **1. Lab: Detecting NoSQL injection**
>Mô tả nói rằng bộ lọc danh mục sản phẩm cho phòng thí nghiệm này được cung cấp bởi cơ sở dữ liệu MongoDB NoSQL. Nó dễ bị tấn công bởi NoSQL. Để giải quyết bài lab, hãy thực hiện một cuộc tấn công NoSQL injection khiến ứng dụng hiển thị các sản phẩm chưa được phát hành.
![image](https://hackmd.io/_uploads/r1wV3PmM0.png)

>Truy cập lab và có chức năng lọc sản phẩm hiển thị theo danh mục ta chọn: ![image](https://hackmd.io/_uploads/r1iGaDmMA.png)

>Bắt request lọc sản phẩm theo danh mục: ![image](https://hackmd.io/_uploads/S1HfCwQzC.png)

> Thêm kí tự đặc biệt `'` để kiểm tra web có bị NoSQL Injection không:
> ![image](https://hackmd.io/_uploads/rk280v7fC.png)

>Thử tiếp tục bằng payload `\'`, câu truy vấn sẽ là: `category='\''` : 
>![image](https://hackmd.io/_uploads/rJwP1dQfA.png)

> Payload không gây ra lỗi cú pháp => ứng dụng có thể bị lỗ hổng NoSQL injection!
> Sử dụng truy vấn điều kiện để kiểm tra: `' && 0 && 'x` : ![image](https://hackmd.io/_uploads/ryCHg_mMR.png)

> Khi chèn điều kiện `false`, không có sản phẩm nào trả về! Thử với điều kiện `true`: ![image](https://hackmd.io/_uploads/H1bceuQzA.png)

> Khi chèn kết quả là `true`, trả về toàn bộ sản phẩm có trong category Lifestyle..
> Chèn câu truy vấn điều kiện luôn đúng để trả về toàn bộ dữ liệu: `'||1||'` : ![image](https://hackmd.io/_uploads/r1UVbOXGA.png)

> Như vậy, ta đã lấy được tất cả các sản phẩm, bao gồm các sản phẩm chưa được phát hành và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/SysIZOmMR.png)

# NoSQL operator injection
Cơ sở dữ liệu NoSQL thường sử dụng toán tử truy vấn, cung cấp các cách để chỉ định các điều kiện mà dữ liệu phải đáp ứng để được đưa vào kết quả truy vấn. Ví dụ về toán tử truy vấn MongoDB bao gồm:
* **$where** - So khớp các tài liệu thỏa mãn biểu thức JavaScript.
* **$ne** - Khớp tất cả các giá trị không bằng giá trị được chỉ định.
* **$in** - Khớp tất cả các giá trị được chỉ định trong một mảng.
* **$regex** - Chọn tài liệu có giá trị khớp với biểu thức chính quy được chỉ định.

Bạn có thể chèn các toán tử truy vấn để thao tác các truy vấn NoSQL. Để thực hiện việc này, hãy gửi các toán tử khác nhau một cách có hệ thống vào một loạt thông tin đầu vào của người dùng, sau đó xem xét phản hồi để tìm thông báo lỗi hoặc các thay đổi khác.

### Gửi toán tử truy vấn
Trong thông điệp JSON, bạn có thể chèn toán tử truy vấn dưới dạng đối tượng lồng nhau. Ví dụ: `{"username":"wiener"}
` trở thành `{"username":{"$ne":"invalid"}}`. 
Đối với đầu vào dựa trên URL, bạn có thể chèn toán tử truy vấn thông qua tham số URL. Ví dụ: tên `username=wiener` trở thành `{"username":{"$ne":"invalid"}}`. Nếu cách này không hiệu quả, bạn có thể thử cách sau:
1. Chuyển đổi phương thức yêu cầu từ **GET** sang **POST**.
2. Thay đổi header **Content-Type** thành `application/json`.
3. Thêm JSON vào body request.
4. Chèn toán tử truy vấn vào JSON.

>Lưu ý: có thể sử dụng extension **Content Type Converter** để tự động chuyển đổi phương thức yêu cầu và thay đổi yêu cầu POST được URL-encoded thành JSON.

### Phát hiện việc chèn toán tử trong MongoDB
Hãy xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong nội dung của yêu cầu POST: `{"username":"wiener","password":"peter"}`

Kiểm tra từng đầu vào với một loạt toán tử. Ví dụ: để kiểm tra xem thông tin nhập vào tên người dùng có xử lý toán tử truy vấn hay không, bạn có thể thử thao tác chèn sau:
`{"username":{"$ne":"invalid"},"password":{"peter"}}`

Nếu toán tử `$ne` được áp dụng, điều này sẽ truy vấn tất cả người dùng có tên người dùng khác "invalid".

Nếu cả thông tin nhập vào tên người dùng và mật khẩu đều xử lý toán tử thì có thể bỏ qua xác thực bằng cách sử dụng trọng tải sau:
`{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}`

Truy vấn này trả về tất cả thông tin đăng nhập trong đó cả tên người dùng và mật khẩu đều không hợp lệ. Kết quả là bạn đã đăng nhập vào ứng dụng với tư cách là người dùng đầu tiên trong bộ sưu tập.

Để nhắm mục tiêu một tài khoản, bạn có thể tạo payload bao gồm tên người dùng đã biết hoặc tên người dùng mà bạn đã đoán. Ví dụ:
`{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}`

# **2. Lab: Exploiting NoSQL operator injection to bypass authentication**
>Để giải quyết bài lab, đăng nhập vào ứng dụng với tư cách người dùng quản trị viên và xóa người dùng `carlos`.
Cho sẵn 1 user hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/H1o1-MFfA.png)

>Đăng nhập vào `wiener`: ![image](https://hackmd.io/_uploads/ByRNPMtM0.png)

>Trong request `POST /login` thấy phần body chứa username và password được đặt bằng JSON: ![image](https://hackmd.io/_uploads/SkiPPfYz0.png)

>Thử chèn toán tử truy vấn `{"$ne":"test"}`  vào password cho `wiener` đã tồn tại sẵn, điều này nghĩa là nó sẽ thử đăng nhập với `username=wiener` và các password không bằng `test`: ![image](https://hackmd.io/_uploads/r1g_5zYMC.png)

>Bypass thành công đăng nhập vào `wiener` !Nghĩ ngay đến việc có thể bypass login vào `administrator`  với payload cho mật khẩu là `{"$ne":"test"}`. Nhưng server báo là tên đăng nhập này không tồn tại: ![image](https://hackmd.io/_uploads/S11gKzYMC.png)

>Thay đổi payload để lấy được tất cả username và password: ![image](https://hackmd.io/_uploads/B1_oKfKfA.png)

>Nhưng server đã giới hạn số lượng kết quả trả về để render... Lúc này, mình sẽ tiến hành tìm username của admin, có thể đoán thử nó là `admin`, `root`, `Admin`,... Và mình sẽ dùng toán tử `$in` trong MongoDB, nó sẽ cho list các tài khoản này vào 1 mảng, và thử đăng nhập với username lần lượt từng phần tử trong mảng đó! Payload: `{"$in":["admin","administrator","superadmin","root"]}` ![image](https://hackmd.io/_uploads/SkuioMtzC.png)

>Vẫn không thể đoán được username của admin... Tiếp tục thử đến toán tử `$regex`, nó sẽ chọn các giá trị khớp với biểu thức chính quy được chỉ định. Payload: `{"$regex":"ad"}`: ![image](https://hackmd.io/_uploads/SkWVnzYMR.png)

>Lần này đã đúng khi đoán username của admin có chứa `ad` và đăng nhập vào thành công! Cuối cùng dùng "Show respone in browser" để hoàn thành bài lab: ![image](https://hackmd.io/_uploads/rkCc3fYzC.png)

>Tài khoản admin đặt thế này thì ai mà mò ra được...

# Khai thác cú pháp để trích xuất dữ liệu
Trong nhiều cơ sở dữ liệu NoSQL, một số toán tử truy vấn hoặc hàm có thể chạy mã JavaScript hạn chế, chẳng hạn như toán tử `$where` và hàm **mapReduce()** của MongoDB. Điều này có nghĩa là nếu một ứng dụng dễ bị tấn công sử dụng các toán tử hoặc hàm này thì cơ sở dữ liệu có thể đánh giá JavaScript như một phần của truy vấn. Do đó, bạn có thể sử dụng các hàm JavaScript để trích xuất dữ liệu từ cơ sở dữ liệu.

### Lọc dữ liệu trong MongoDB
Hãy xem xét một ứng dụng dễ bị tấn công cho phép người dùng tra cứu tên người dùng đã đăng ký khác và hiển thị vai trò của họ. Điều này kích hoạt một yêu cầu tới URL: `https://insecure-website.com/user/lookup?username=admin`

Điều này dẫn đến truy vấn NoSQL sau đây của tập người dùng: 
`{"$where":"this.username == 'admin'"}`

Vì truy vấn sử dụng toán tử `$where`, bạn có thể thử đưa các hàm JavaScript vào truy vấn này để nó trả về dữ liệu nhạy cảm. Ví dụ: bạn có thể gửi payload sau:
`admin' && this.password[0] == 'a' || 'a'=='b`

Điều này trả về ký tự đầu tiên của chuỗi mật khẩu của người dùng, cho phép bạn trích xuất ký tự mật khẩu theo từng ký tự.

Bạn cũng có thể sử dụng hàm JavaScript match() để trích xuất thông tin. Ví dụ: tải trọng sau cho phép bạn xác định xem mật khẩu có chứa các chữ số hay không: 
`admin' && this.password.match(/\d/) || 'a'=='b`

# **3. Lab: Exploiting NoSQL injection to extract data**
>Chức năng tra cứu người dùng cho phòng thí nghiệm này được cung cấp bởi cơ sở dữ liệu MongoDB NoSQL. Nó dễ bị tấn công bởi NoSQL. Để giải quyết bài lab, đăng nhập vào ứng dụng với tư cách người dùng quản trị viên và xóa người dùng `carlos`. Cho sẵn 1 user hợp lệ `wiener:peter`
![image](https://hackmd.io/_uploads/BJkGk7KMA.png)

>Đăng nhập vào `wiener` và trong các request thì thấy có 1 request để tìm kiếm thông tin người dùng dựa trên username **"GET /user/lookup?user=wiener"**, nếu tồn tại username thì mới vào trang `/my-account`:
> ![image](https://hackmd.io/_uploads/B1DMW7KzC.png)

>Ta có thể sử dụng request này để kiểm tra xem tài khoản `administrator` tồn tại hay không: ![image](https://hackmd.io/_uploads/HJBjWQtGC.png)

>Chứng minh được trong hệ thống tồn tại username `administrator`, ta sẽ đi tìm mật khẩu của nó. Ta sẽ gửi các điều kiện boolean và xem server sẽ xử lí như thế nào... Dùng payload điều kiện đúng: `administrator' && '1'=='1`: ![image](https://hackmd.io/_uploads/S1wqzXtfR.png)

>Khi điều kiện đúng thì kết quả vẫn được trả về bình thường! Vậy với điều kiện sai thì sao? Payload: `administrator' && '1'=='2`
>![image](https://hackmd.io/_uploads/B1IlXXFzA.png)

>Thông báo lỗi không thể tìm thấy người dùng => Ta có thể lợi dụng điểm này để có thể đoán độ dài mật khẩu, cũng như từng kí tự của mật khẩu dựa trên thông báo khi respone trả về !!

>Dùng Intruder để đoán được độ dài mật khẩu! Payload: 
>`administrator' && this.password.length == $$ || 'a'=='b` : ![image](https://hackmd.io/_uploads/B1HxNmtG0.png)

>Cho payload chạy từ 1 -> 50: ![image](https://hackmd.io/_uploads/HyplEmtzC.png)

>Attack và thấy rõ khi **payload = 8** thì có **Length** trả về là khác biệt => Password chứa 8 kí tự! 
>![image](https://hackmd.io/_uploads/H1xN4QYzA.png)

>Tiếp tục đoán từng kí tự của password, sử dụng payload:
> `administrator' && this.password[§0§]=='§§` : ![image](https://hackmd.io/_uploads/BJF6N7tfR.png)

>Thiết lập tham số cho payload 1, chạy từ 0 -> 7 cho 8 kí tự mật khẩu: 
>![image](https://hackmd.io/_uploads/r1E4SQtG0.png)

>Tham số cho payload 2 là tất cả các kí tự chữ cái + số: ![image](https://hackmd.io/_uploads/BkxPH7FzR.png)

>Attack và quan sát thấy các giá trị của password có Length trả về khác biệt: ![image](https://hackmd.io/_uploads/BkncU7Fz0.png)

> Như vậy, mật khẩu cho `administrator` là `edwpwfuv`. Đăng nhập và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/r1rfvQKfA.png)


###  Xác định tên trường
Vì MongoDB xử lý dữ liệu bán cấu trúc không yêu cầu lược đồ cố định nên bạn có thể cần xác định các trường hợp lệ trong bộ sưu tập trước khi có thể trích xuất dữ liệu bằng cách sử dụng tính năng chèn JavaScript.

Ví dụ: để xác định xem cơ sở dữ liệu MongoDB có chứa trường `password` hay không, bạn có thể gửi tải trọng sau:
`https://insecure-website.com/user/lookup?username=admin'+%26%26+this.password!%3d'`

Gửi lại tải trọng cho trường hiện có và cho trường không tồn tại. Trong ví dụ này, bạn biết rằng trường `username` tồn tại, vì vậy bạn có thể gửi các tải trọng sau:
`admin' && this.username!=' `

`admin' && this.foo!='`

Nếu trường `password` tồn tại, bạn sẽ mong đợi phản hồi giống hệt với phản hồi cho trường hiện tại (username), nhưng khác với phản hồi cho trường không tồn tại (foo).

Nếu muốn kiểm tra các tên trường khác nhau, bạn có thể thực hiện tấn công từ điển bằng cách sử dụng danh sách từ để duyệt qua các tên trường tiềm năng khác nhau.

> Lưu ý: Ngoài ra, bạn có thể sử dụng tính năng chèn toán tử NoSQL để trích xuất tên trường theo từng ký tự. Điều này cho phép bạn xác định tên trường mà không cần phải đoán hoặc thực hiện tấn công từ điển. 

# Khai thác tính năng chèn toán tử NoSQL để trích xuất dữ liệu
Ngay cả khi truy vấn ban đầu không sử dụng bất kỳ toán tử nào cho phép bạn chạy JavaScript tùy ý, bạn vẫn có thể tự chèn một trong các toán tử này vào. Sau đó, bạn có thể sử dụng các điều kiện boolean để xác định xem ứng dụng có thực thi bất kỳ JavaScript nào mà bạn đưa vào thông qua toán tử này hay không.

### Chèn các toán tử trong MongoDB
Hãy xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong nội dung của yêu cầu POST:
`{"username":"wiener","password":"peter"}`

Để kiểm tra xem bạn có thể thêm toán tử hay không, bạn có thể thử thêm toán tử `$where` làm tham số bổ sung, sau đó gửi một yêu cầu trong đó điều kiện đánh giá là sai và một yêu cầu khác đánh giá là đúng. Ví dụ:
`{"username":"wiener","password":"peter", "$where":"0"}`
`{"username":"wiener","password":"peter", "$where":"1"}`

Nếu có sự khác biệt giữa các câu trả lời, điều này có thể cho biết biểu thức JavaScript trong mệnh đề `$where` đang được đánh giá.

**Trích xuất tên trường:**

Nếu bạn đã chèn một toán tử cho phép bạn chạy JavaScript, bạn có thể sử dụng phương thức **key()** để trích xuất tên của các trường dữ liệu. Ví dụ: bạn có thể gửi tải trọng sau:
`"$where":"Object.keys(this)[0].match('^.{0}a.*')"`

Thao tác này sẽ kiểm tra trường dữ liệu đầu tiên trong đối tượng người dùng và trả về ký tự đầu tiên của tên trường. Điều này cho phép bạn trích xuất tên trường theo từng ký tự.

**Lọc dữ liệu bằng toán tử**
Ngoài ra, bạn có thể trích xuất dữ liệu bằng cách sử dụng các toán tử không cho phép bạn chạy JavaScript. Ví dụ: bạn có thể sử dụng toán tử `$regex` để trích xuất dữ liệu theo từng ký tự.

Hãy xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong nội dung của yêu cầu POST. Ví dụ:
`{"username":"myuser","password":"mypass"}`

Bạn có thể bắt đầu bằng cách kiểm tra xem toán tử $regex có được xử lý như sau không:
`{"username":"admin","password":{"$regex":"^.*"}}`

Nếu phản hồi cho yêu cầu này khác với phản hồi bạn nhận được khi gửi mật khẩu không chính xác, điều này cho thấy ứng dụng có thể dễ bị tấn công. Bạn có thể sử dụng toán tử $regex để trích xuất dữ liệu theo từng ký tự. Ví dụ: phần tải trọng sau sẽ kiểm tra xem mật khẩu có bắt đầu bằng chữ a hay không:
`{"username":"admin","password":{"$regex":"^a*"}}`

# **4. Lab: Exploiting NoSQL operator injection to extract unknown fields**
> Chức năng tra cứu người dùng cho phòng thí nghiệm này được cung cấp bởi cơ sở dữ liệu MongoDB NoSQL. Nó dễ bị tấn công bởi NoSQL. Để giải quyết bài lab, đăng nhập vào `carlos`
![image](https://hackmd.io/_uploads/S1-adXYf0.png)

>Bài lab có chức năng "Forgot password": ![image](https://hackmd.io/_uploads/ByOrRQKGA.png)

> Nhập thử `carlos` vào thì họ nói send link qua email, nhưng lab không cung cấp 1 hòm thư email nào cho ta cả... 
> ![image](https://hackmd.io/_uploads/SkuuA7KzA.png)

>Bắt request `/login` thì thấy dữ liệu được gửi đi dưới dạng JSON: ![image](https://hackmd.io/_uploads/rkLpC7tMA.png)

>Thử thêm các payload NoSQL injection vào đây thì không có gì xảy ra: 
![image](https://hackmd.io/_uploads/r1IggEKM0.png)

>Nó nói rằng `carlos` đã bị khóa, và phải reset password, và hint trong đề cũng nói ta phải lấy được resetToken... 

> Kiểm tra xem ta có thể chèn toán tử vào không, sử dụng toán tử `$where` làm tham số bổ sung!
> Thêm điều kiện sai:  
`{"username":"wiener","password":{"$ne":"test"}, "$where":"0"}`
![image](https://hackmd.io/_uploads/SyUBfNFMA.png)

>Thử với điều kiện đúng: 
`{"username":"wiener","password":{"$ne":"test"}, "$where":"1"}`
![image](https://hackmd.io/_uploads/HJLQf4FMA.png)

>Thấy rằng 2 điều kiện đúng/sai trả về 2 respone khác nhau, chứng tỏ rằng ta có thể chèn thêm toán tử vào chỗ này!! 

>Ta có thể dựa vào chèn toán tử boolean để tìm tên của các cột, sử dụng payload sau để xem kí tự 0 có phải là chữ cái 'a' hay không: 
>`"$where":"Object.keys(this)[0].match('^.{0}a.*')"`
>![image](https://hackmd.io/_uploads/HJ4e4VFzA.png)

>Lỗi => không phải chữ 'a' ở kí tự đầu! Và để tự động hóa nhanh hơn, ta sẽ sử dụng Intruder! Dùng payload sau:
>`"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"`
![image](https://hackmd.io/_uploads/HkNvENYG0.png)

>Thiết lập tham số: ![image](https://hackmd.io/_uploads/By7NHNYfR.png)
![image](https://hackmd.io/_uploads/HkqyuEYzC.png)

>Lấy được cột đầu tiên là `id`: ![image](https://hackmd.io/_uploads/SkRGdEYGR.png)

>Lấy được cột thứ 2 là `username`: ![image](https://hackmd.io/_uploads/B1kAP4tzC.png)

>Lấy được cột thứ 3 là `password`: ![image](https://hackmd.io/_uploads/HyvlYVKzC.png)

>Lấy được cột thứ 4 là `email`: ![image](https://hackmd.io/_uploads/HJbZ9EKzA.png)

>Lấy được cột thứ 5 là `changePwd`:
>![image](https://hackmd.io/_uploads/SyOOj4tzA.png)

>Thử đến cột thứ 6 thì bị lỗi 500 => tồn tại 5 bảng trong db: ![image](https://hackmd.io/_uploads/ByfkhEtM0.png)

>Cột `changePwd` có khả năng cao chứa 1 cái gì đó, ta sẽ đi đoán giá trị trong này, payload: 
>`"$where":"this.changePwd.match('^.{§§}§§.*')"`
![image](https://hackmd.io/_uploads/B1mKvHFMC.png)
    
>Lấy được `token=9659ee6109b7adfd`.  Cho token vào request `GET /forgot-password?changePwd=9659ee6109b7adfd`:
>![image](https://hackmd.io/_uploads/BydBvHYGA.png)

>Vào được giao diện đổi mật khẩu cho `carlos` ! **"Show respone in browser"** và đổi mật khẩu. Đăng nhập vào `carlos` và hoàn thành bài lab: ![image](https://hackmd.io/_uploads/ByCldrKGC.png)

# Time-base injection
Đôi khi việc gây ra lỗi cơ sở dữ liệu không gây ra sự khác biệt trong phản hồi của ứng dụng. Trong trường hợp này, bạn vẫn có thể phát hiện và khai thác lỗ hổng bằng cách sử dụng tính năng chèn JavaScript để kích hoạt độ trễ thời gian có điều kiện.

Để tiến hành chèn NoSQL dựa trên thời gian:
1. Tải trang nhiều lần để xác định thời gian tải cơ bản.
2. Chèn tải trọng dựa trên thời gian vào đầu vào. Tải trọng dựa trên thời gian gây ra sự chậm trễ có chủ ý trong phản hồi khi được thực thi. Ví dụ: `{"$where": "sleep(5000)"}` gây ra độ trễ có chủ ý là 5000 mili giây khi tiêm thành công.
3. Xác định xem phản hồi có tải chậm hơn không. Điều này cho thấy việc tiêm thành công.

Tải trọng dựa trên thời gian sau đây sẽ kích hoạt độ trễ thời gian nếu mật khẩu có chữ cái a:
`admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'`

`admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'`

# Ngăn chặn NoSQL injection
Cách thích hợp để ngăn chặn các cuộc tấn công tiêm nhiễm NoSQL phụ thuộc vào công nghệ NoSQL cụ thể đang được sử dụng. Vì vậy, chúng tôi khuyên bạn nên đọc tài liệu bảo mật cho cơ sở dữ liệu NoSQL mà bạn lựa chọn. Điều đó nói lên rằng, những hướng dẫn rộng rãi sau đây cũng sẽ hữu ích:
* Dọn dẹp và xác thực thông tin đầu vào của người dùng bằng cách sử dụng danh sách cho phép gồm các ký tự được chấp nhận.

* Chèn thông tin đầu vào của người dùng bằng cách sử dụng các truy vấn được tham số hóa thay vì nối trực tiếp thông tin đầu vào của người dùng vào truy vấn.

* Để ngăn chặn việc chèn toán tử, hãy áp dụng danh sách cho phép các keys được chấp nhận.