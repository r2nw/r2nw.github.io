---
title: Write-Up Faster Shop
date: 2023-09-15 3:55:00 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
Địa chỉ: [Faster Shop](https://battle.cookiearena.org/challenges/web/faster-shop)

## **Challenge Details**

Use username `guest` and password `guest` for logging to the shop. You only has 20 bucks, how to buy flag with 21 bucks.

## **Tags**

- Race Condition
- WeCTF

## **Race Condition là gì ?**

Là một loại lỗ hổng phổ biến có liên quan chặt chẽ đến [các lỗi logic](https://portswigger.net/web-security/logic-flaws) nghiệp vụ . Chúng xảy ra khi các trang web xử lý yêu cầu đồng thời mà không có biện pháp bảo vệ thích hợp. Điều này có thể dẫn đến nhiều luồng riêng biệt tương tác với cùng một dữ liệu cùng một lúc, dẫn đến "va chạm" gây ra hành vi ngoài ý muốn trong ứng dụng. Cuộc tấn công theo điều kiện chủng tộc sử dụng các yêu cầu được tính thời gian cẩn thận để gây ra xung đột có chủ ý và khai thác hành vi ngoài ý muốn này cho mục đích xấu.

## ****Web Analysis:****

Tại `/` trang web:

Là 1 trang login mà bài đã cung cấp username và passwd 

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled.png)

Tiến hành login
Xuất hiện 1 trang với tính năng mua và bán lại.

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%201.png)

Ta để ý thấy `Flag` được bán với giá là `21$` tuy nhiên trang web chỉ cung cấp cho chúng ta `20$`.

hmm

`+1 IDEA Sau 1 thời gian mua đi bán lại và để ý tới đề bài là faster + tag race condition thì liệu ta có thể mua đi bán lại thật nhanh làm cho trang web bán nhầm không.`

## ****Exploiting RACE CONDITIONS****

Kiểm chứng ý tưởng

Mình đã mua và bán lại lặp lại nhiều lần.

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%202.png)

Khi mình spam mua thì để ý mình chỉ mua 15$ mà trang web đã báo lỗi. Vậy đã chứng minh trang web nó đã hiểu nhầm số lượt click.

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%203.png)

Giờ mình tiến hành spam bán lại

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%204.png)

Tèn ten 20$ mua được 5 món

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%205.png)

giờ bán lại là được 25$ mà đủ tiền mua FLAG rồi

*challenge này nay nó bị lag cứ bị lỗi này mọi người chứ refesh lại web rồi ấn tiếp tục nha*

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%206.png)

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%207.png)

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%208.png)

mua FLAG bán lại lần nữa nha

![Untitled](CTF%20Write-Up%20Faster%20Shop%207a23fb151fc94fa1a6a12e8b62b73bdb/Untitled%209.png)

DONEEE

`***Written by Ren***`