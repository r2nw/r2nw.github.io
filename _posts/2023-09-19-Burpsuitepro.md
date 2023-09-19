---
title: Burp Suite Professional
date: 2023-09-19 3:55:00 +0700
categories: [Tools, BurpSuite]
tags: [tools, burpsuite pro]     # TAG names should always be lowercase
---
## LINUX & WINDOWS:

### PART 1 LINUX

- **STEP 1:**
    
    Download : **[Burp Suite Professional platform JAR](https://portswigger.net/burp/releases/professional-community-2023-10-1-1?requestededition=professional&requestedplatform=)**
    
    ![Untitled](/assets/burpsuite/Untitled.png)
    
- **STEP 2:**
    
    Download: [Burploader](https://drive.google.com/file/d/1e742o6T8nDPjzOBNbRommy0CzxCJUizN/view)
    
- **STEP 3:**
    
    COPY vào chung 1 thư mục
    
    ![Untitled](/assets/burpsuite/Untitled%201.png)
    

- **STEP 4:**
    
    Open terminal 
    
    ![Untitled](/assets/burpsuite/Untitled%202.png)
    
    Use command: 
    
    ```python
    java -jar burploader.jar
    ```
    
    ```python
    java -jar burpsuite_pro_v2023.10.2.jar
    ```
    
    ## **WINDOWS: (CÁC FILE DOWNLOAD ĐỂ CHUNG 1 THƯ MỤC GIỐNG LINUX)**
    
    ### **PART 1 WINDOWS**
    
    **STEP 1:** INSTALL JDK [CLICK](https://www.oracle.com/java/technologies/downloads/#jdk20-windows) (máy linux nào chưa có java thì chọn version linux rồi tải)
    
    Chọn 1 trong 2 để cài đặt.
    
    Download: [Burploader](https://drive.google.com/file/d/1e742o6T8nDPjzOBNbRommy0CzxCJUizN/view)
    
    ![Untitled](/assets/burpsuite/Untitled%203.png)
    
    ![Untitled](/assets/burpsuite/Untitled%204.png)
    
    ![Untitled](/assets/burpsuite/Untitled%205.png)
    
    Sau khi cài đặt khởi động lại máy.
    
    **STEP 2: Mở Powershell**
    
    ```python
    java -jar .\burploader.jar
    ```
    
    ![Untitled](/assets/burpsuite/Untitled%206.png)
    
    ### **PART 2 LINUX + WINDOWN**
    
    **STEP 1:** Tích vào Auto Run & RUN
    
    ![Untitled](/assets/burpsuite/Untitled%207.png)
    
    **STEP 2:** Sau khi OPEN ta COPY License vào và ấn NEXT
    
    ![Untitled](/assets/burpsuite/Untitled%208.png)
    
    **STEP 3:** Chọn Manual activation
    
    ![Untitled](/assets/burpsuite/Untitled%209.png)
    
    **STEP 4:** Copy theo các bước trong ảnh
    
    ![Untitled](/assets/burpsuite/Untitled%2010.png)
    
    **STEP 5:**DONE
    
    ![Untitled](/assets/burpsuite/Untitled%2011.png)
    
    Sử dụng bình thường
    
    ![Untitled](/assets/burpsuite/Untitled%2012.png)
    

## ADVANCED

Tại vì mình lười nên mỗi lần mở Burpsuite Pro là phải truy cập vào thư mục và gỏ lệnh 

Java -jar burploader.jar nên rất là lười nên mình đã viết 1 script gỏ open ứng dụng như các ứng dụng khác.

## Tại windows:

```bash
@echo off
cd /d "C:\Users\nishi\Documents\BurpsuitePro"
java -jar burploader.jar
```

Mở notepad copy các dòng code trên và lưu với đuôi là `.bat`

![Untitled](/assets/burpsuite/Untitled%2013.png)

Thử tắt burpsuite và click vào xem nó có mở lại không nhé.

![Untitled](/assets/burpsuite/Untitled%2014.png)

Thành công

nếu nó còn hiện bản burploader cứ tắt đi nhé r mở lại xem nó còn không.

Nhưng giờ nó vẫn phải mở thư mục vào để mở burpsuite nên giờ thêm thêm các bước sau đây.

![Untitled](/assets/burpsuite/Untitled%2015.png)

Send to Desktop

![Untitled](/assets/burpsuite/Untitled%2016.png)

đỗi tên mới cho sang choảnh

![Untitled](/assets/burpsuite/Untitled%2017.png)

Nhưng hình icon xấu quá :)) 

![Untitled](/assets/burpsuite/Untitled%2018.png)

Chuột phải → kiểm tra

![Untitled](/assets/burpsuite/Untitled%2019.png)

Download và chuyển sang dạng ICO

![Untitled](/assets/burpsuite/Untitled%2020.png)

![Untitled](/assets/burpsuite/Untitled%2021.png)

![Untitled](/assets/burpsuite/Untitled%2022.png)

Làm theo các bước trong ảnh.

![Untitled](/assets/burpsuite/Untitled%2023.png)

Làm theo các bước trong ảnh.

![Untitled](/assets/burpsuite/Untitled%2024.png)

![Untitled](/assets/burpsuite/Untitled%2025.png)

Làm theo các bước trong ảnh.

![Untitled](/assets/burpsuite/Untitled%2026.png)

TÈN TEN :)) Như hàng REAL

## Tại LINUX:

```bash
#!/bin/bash

# Đường dẫn đến thư mục chứa burploader.jar
BURP_DIR="/home/kali/Tools/BurpSuitePro"

# Kiểm tra xem thư mục tồn tại không
if [ -d "$BURP_DIR" ]; then
  # Thay đổi thư mục làm việc đến thư mục Burp Suite
  cd "$BURP_DIR"
  # Chạy Burp Suite Professional
  java -jar burploader.jar
else
  echo "Thư mục không tồn tại: $BURP_DIR"
fi
```

lưu dưới dạng `.sh` xong rồi rename bỏ đuôi cũng được

![Untitled](/assets/burpsuite/Untitled%2027.png)

Để mở được như thế này…

Nếu bạn đang sài Kali nó có sẳn bản burp comunity

Nên như vậy mở nó sẽ bị trùng tên

nên bạn hãy xóa trước bản burp community trong /usr/bin/burpsuite đi nha

command:

```bash
sudo rm /usr/bin/burpsuite
```

Rồi copy cái file script mới vừa tạo bỏ vào

```bash
sudo mv burpsuitepro /usr/bin
```

điều chỉnh câu lệnh sao cho phù hợi đường dẫn bạn lưu hoặc bạn kéo thả tay cũng được.

xong rồi  test thành quả :))

![Untitled](/assets/burpsuite/Untitled%2028.png)

Bạn nào muốn đỗi tên sao cho đẹp thì nạp card 20k nha haha…

Bạn nào gặp lỗi trong quá trình thì comment ở đây.

[Click ở đây nè](https://github.com/r2nw/r2nw.github.io/issues/2)

![Untitled](/assets/burpsuite/Untitled%2029.png)

Cảm ơn các bạn đã xem.