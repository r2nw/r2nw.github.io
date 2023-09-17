---
title: Write-Up Node Redis API
date: 2023-09-17 12:34:01 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up, Web]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
## **Challenge Details**

- Challenge: **[Node Redis API](https://battle.cookiearena.org/challenges/web/node-redis-api)**
- Topic: Download source code and find the way to obtain the flag. The challenge involves a Node.js application using Redis for logging and provides four API endpoints:
    - GET /login?userid=guest&userpw=guest
    - GET /show_logs
    - GET /show_logs?log_query=get/log_info
    - GET /flag

## **Tags**

- Session
- Redis
- HTTP Pollution
- Redis Command Injection
- NodeJS

## ****Web Analysis:****

Tại giao diện web có vẻ không có gì nổi bật…

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled.png)

Thử thách gợi ý bốn đường dẫn trên ứng dụng web. Chúng ta hãy xem từng cái một:

- **`/login?userid=guest&userpw=guest`**
    - Khi chúng tôi truy cập vào đường dẫn này với **`userid`** và **`userpw`** là `guest`, nó sẽ hiển thị một thông báo ,trang web thay đỗi từ"hello undefined" sang "hello + user.”

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%201.png)

- `**/show_logs**`
Đường dẫn này hiển thị một liên kết mới trên trang web.

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%202.png)

- **`/show_logs?log_query=get/log_info`**

Việc truy cập vào đường dẫn này sẽ hiển thị một khóa và các giá trị liên quan, gợi ý điều gì đó liên quan đến nhật ký..

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%203.png)

- **`/flag`**

Có vẻ như user 'guest' không thể truy cập/flag và không có lỗi hiển thị nào…hmm

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%204.png)

## ****Source Code Analysis****

ở tại **`/`** trên trang web:

```jsx
app.get('/', function (req, res) {
res.send('hello ' + req.session.userid);
});
```

Khi truy cập vào trang web, trang web sẽ in ra Hello + lấy từ session và in ra userid của session đã đăng nhập.

Next, the **`/flag`** route:

```jsx
app.get('/flag', function (req, res) {
if (req.session.userid === "admin") {
res.send(FLAG)
} else {
res.send('hello ' + req.session.userid);
}
});
```

Để xem được /flag chứa flag cuối cùng thì ta phải vào được user admin thì lúc đó trang web sẽ tự động in ra Flag của chúng ta.
Mà trong source code họ có cho xem db chứa username và passwd

Chức năng login kiểm tra thông tin xác thực của người dùng dựa trên cơ sở dữ liệu này:

```jsx
const db = {
'guest': 'guest',
'cookiearena': '1234',
'ADMIN': 'this_is_admin?'
}
```

```jsx
function login(user) {
return user.userpw && db[user.userid] == user.userpw;
}
```

Và mình chuyển qua quan xem đoạn tiếp theo.

Mình bắt đầu để ý tới đoạn.

Đoạn mã này sử dụng Redis để lưu trữ thông tin đăng nhập của người dùng. Nó tạo một khóa mới trong Redis với tên bắt đầu bằng '`**log_**`' và sau đó là thời gian hiện tại (dùng **`new Date().getTime()`** để lấy thời gian dưới dạng số miligiây) và giá trị là chuỗi 'userid: ' cộng với giá trị của thuộc tính **`req.session.userid`**. và thực hiện so sánh nó truyền vào **`req.query`**. Nếu **`login()`** trả về **`true`**, có nghĩa là đăng nhập thành công.

Lúc này ta có 1 chút ý tưởng và liên quan về session login của trang web.

```jsx
app.get('/login', function (req, res) {
redis_client.set('log_' + new Date().getTime(), 'userid: ' + req.session.userid);
if (login(req.query)) {
req.session.userid = req.query.userid;
res.send('<script>alert("login!");history.go(-1);</script>');
} else {
res.send('<script>alert("login failed!");history.go(-1);</script>');
}
});
```

ta thấy rằng session nó chứa log_info và userid.
Câu hỏi đặt ra là liệu mình có thể xem được cái plantext của cái session này không? điều này mình chưa chắc.

```jsx
app.use(session(sess));
redis_client.set('log_info', 'KEY: "log_" + new Date().getTime(), VALUE: userid');
```

Tiếp đến ta xem qua đoạn code chứa PATH `/show_logs` và parameter `?log_query=get/log_info.`

```jsx
app.get('/show_logs', function (req, res) {
// var  =get/log_info
var log_query = req.query.log_query;
try {
log_query = log_query.split('/');
if (log_query[0].toLowerCase() != 'get') {
log_query[0] = 'get';
}
log_query[1] = log_query.slice(1)
} catch (err) {
// Todo
// Error(403);
}
console.log(log_query)
try {
redis_client.send_command(log_query[0], log_query[1], function (err, result) {
if (err) {
res.send('ERR');
} else {
res.send(result);a
}
})
} catch (err) {
res.send('try /show_logs?log_query=get/log_info')
}
});
```

giải thích tóm tắt:

log_query[0] chứa command resdis được gán giá trị là GET và log_query[1] thường là tên của đối số ví dụ như **log_info** 
vậy ta có thể hiểu là send_command(get,log_info) mình lại nghĩ nó giống như linux như là send_command(cat,filename) chăng 😂 .
*”Lúc này mình chưa biết về command redis” nhờ chatgpt hổ trợ.*

```jsx
redis_client.send_command(log_query[0], log_query[1], function (err, result)
//String Typekey, value, callback => Command(command, [key, value], callback)
```

## ****Exploiting Redis Command Injection****

CHỨNG MINH Ý TƯỞNG:

1 Thông qua source code cung cấp ta có thể thấy trong code có hàm thực thi command redis ta có thể lợi dụng nó xem session được hay không? 

2 Nếu xem được thì mình có thể thêm userid và userpasswd của Admin hay không ?

Đầu tiên mình nhờ chatgpt liệt kê 1 số lệnh của Redis Command

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%205.png)

và mình test với Path đã cho của bài là truyền thử tham số GET /show_logs?log_query=get

`res.send('try /show_logs?log_query=get/log_info')`

mà `String Typekey, value, callback => Command(command, [key, value], callback)`
phải có 2 đối số truyền vào. Nên mình đã thử truyền vào 1 đối số.

`/show_logs?log_query[0]=get&log_query[1][0]=log_info`

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%206.png)

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%207.png)

Nó trả về kết quả là nội dung lúc chúng ta vào `/show_log` vậy có nghĩ là nếu mình thay `log_info` vào 1 `log_xxxx` chứa nội dung sess của userid

Lúc này mình thử test và search về command redis 1 lúc và mình có biết thử command là `keys`

Boom!!! mình đã show được tất cả sess trong redis

`/show_logs?log_query[0]=keys&log_query[1][0]=*`

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%208.png)

mà trong sess có chứa userid.

giờ mình đã biết được đối số nên giờ mình quay lại hàm GET ở trên và truyền vào thử 1 sess xem kết quả như nào.

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%209.png)

và mình đã xem được nội dung trong sess lúc này mình đã chứng minh được ý tưởng ban đầu là đúng.

mà ngồi xem từng sess hơi lâu nên các bạn có thể viết code bỏ kí tự “ và , rồi brute force xem cho nhanh nha

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2010.png)

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2011.png)

Xem qua 1 lượt thì mình thấy có 1 sess có chứa userid là guest

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2012.png)

Còn 1 cách nữa là xem cái session login ( cách này mình chỉ mới biết khi đang viết write up).

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2013.png)

Giờ mình chứng minh ý tưởng 2 là liệu mình có thêm 1 userid admin được không ?

Mà trong chatgpt lúc này mình có show command có GET và cả SET

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2014.png)

vậy giờ mình thí nghiệm thử SET liệu nó có ghi vào redis thật không? 
nếu được thì mình có thể tạo ra user admin hay ghi đè được không ?

Lúc này mình truyền vào đối số là 

`?log_query[0]=set&log_query[1]=admin&log_query[1]=1234`

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2015.png)

và hiện thành công giờ chúng ta show ra thử

và nó đã tạo mới cho chúng ta là Key là admin và nội dung là 1234

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2016.png)

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2017.png)

mà login thì nó phải so sánh nội dung trong sess có chưa userid theo 
`{"cookie":{"originalMaxAge":null,"expires":null,"httpOnly":true,"path":"/"},"userid":"guest"}`

mới được xem là hợp lệ vậy giờ liệu chúng ta có thể ghi đè lên nó hay không?

giờ mình thử sess có chứa userid guest thay thành admin và đăng nhập với user guest xem có được hay không?

payload: 

`/show_logs?log_query[0]=set&log_query[1]=sess:kUmcNMl23GZvk3DJeaqG-QH4sRCgCzPi&log_query[1]={"cookie":{"originalMaxAge":null,"expires":null,"httpOnly":true,"path":"/"},"userid":"admin"}`

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2018.png)

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2019.png)

Giờ nó đã ghi đè lên session guest giờ session guest đã thay đỗi thành admin chúng ta reload lại trang web và nhận flag.

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2020.png)

![Untitled](/assets/writeup/cookie/NodeRedisAPI/Untitled%2021.png)


`***Written by Ren***`