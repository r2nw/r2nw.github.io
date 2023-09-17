---
title: Write-Up Memo Baby CSRF
date: 2023-09-17 12:34:01 +0700
categories: [CTF, Cookie Hân Hoan, Write-Up]
tags: [web, write-up, ctf]     # TAG names should always be lowercase
---
## **Challenge Details**

Địa chỉ: **[Memo Baby CSRF](https://battle.cookiearena.org/challenges/web/memo-baby-csrf)**

FLAG được lưu trong `/admin/notice_flag` nhưng đã bị chặn quyền truy cập. Hãy khai thác các lỗ hổng XSS và CSRF để lấy được flag trong chức năng kiểm tra url nhập vào tại `/flag` . Ấn vào nút Download để xem source code và từ đó phân tích các vector tấn công.

## **Tags**

- CSRF FlaskSelenium
- Python
- Cross Site Request Forgery
- Flask
- Selenium

## ****CSRF là gì?****

Giả mạo yêu cầu chéo trang (còn được gọi là CSRF) là một lỗ hổng bảo mật web cho phép kẻ tấn công xúi giục người dùng thực hiện các hành động mà họ không có ý định thực hiện. Nó cho phép kẻ tấn công phá vỡ một phần chính sách xuất xứ tương tự, được thiết kế để ngăn chặn các trang web khác nhau can thiệp lẫn nhau.

## ****Web Analysis:****

Tại `/` trang chủ trang web:

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled.png)

Trang web đưa ra 4 đường dẫn, thử truy cập từng cái và quan sát.

- `**Vuln(csrf) page**`

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%201.png)

Ta quan sát thấy có một đoạn code script tạo 1 thông báo `alert(1)`  trên thanh `URL` được truyền vào parameter là `?param` sau đường dẫn là `/vuln` tuy nhiên đoạn script đã không được kích hoạt.

- `**memo**`

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%202.png)

Ta thấy xuất hiện trên màn hình là đường dẫn `/memo` và `hello` được truyền vào parameter `?memo`.

Tuy nhiên khi ta quay lại và truy cập `/memo`nhiều lần thì hello nó nối nhau xuất hiện theo số lần. hmm…

- `**notice flag**`

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%203.png)

Khi truy cập vào thì nó hiện truy cập từ chối…có vẻ như nó cần quyền Admin.

`+1 idea: user admin.`

- **`FLag`**

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%204.png)

Tại đây ta quan sát thấy trên màn hình xuất hiện 1 đường dẫn local với đường dẫn /vuln và parameter là ?param và 1 nút submit. Ta thấy /vuln?param giống với ở trên lúc đầu ta quan sát, tuy nhiên nó cho chúng ta điền vào nội dung.

giờ ta thử lấy cái đoạn script ở trên điền vào thử.

```jsx
<script>alert(1)</script>
```

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%205.png)

Ta thấy đoạn script này đoạn kích hoạt và hiện thông báo `good` .

`+1 idea : trang web này cho kích hoạt script có thể liên quan đến lỗi xss, và nhưng script này chỉ kích hoạt với lớp mạng local là lỗi ssrf nhưng Tags đề bài đã cho.`

## ****Source Code Analysis:****

Thành Phần:

```css
.memo-baby-csrf
├── [build-docker.sh](<http://build-docker.sh/>)
├── challenge
│   ├── [run.py](<http://run.py/>)
│   ├── static
│   │   ├── 404.png
│   │   ├── cookie.png
│   │   ├── css
│   │   │   └── non-responsive.css
│   │   ├── discord.png
│   │   ├── facebook.png
│   │   ├── favicon.ico
│   │   ├── index.css
│   │   ├── logo.png
│   │   ├── main.png
│   │   ├── SVN-Determination Sans.otf
│   │   ├── top-background.svg
│   │   └── upheavtt.ttf
│   └── templates
│       ├── 404.jinja2
│       ├── flag.jinja2
│       ├── index.jinja2
│       ├── layout.jinja2
│       └── memo.jinja2
├── config
│   └── supervisord.conf
├── Dockerfile
├── flag.txt
└── requirements.txt
```

Source được viết bằng ngôn ngữ python.

- **`View run.py`**

```python
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("/flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    options = webdriver.ChromeOptions()
    options = Options()
    for _ in [
        "--headless",
        "--disable-gpu",
        "--disable-extensions",
        "--no-sandbox",
        "--disable-dev-shm-usage",
        "--no-sandbox",
        "--enable-logging",
    ]:
        options.add_argument(_)
    driver = webdriver.Chrome(options=options)
    try:

        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:1337/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        print(str(e))
        # return str(e)
        return False
    driver.quit()
    return True

def check_csrf(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:1337/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)

@app.route("/")
def index():
    return render_template("index.jinja2")

@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower()
    xss_filter = ["frame", "script", "on"]
    for _ in xss_filter:
        param = param.replace(_, "*")
    return param

@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.jinja2")
    elif request.method == "POST":
        param = request.form.get("param", "")
        if not check_csrf(param):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

memo_text = ""

@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", None)
    if text:
        memo_text += text
    return render_template("memo.jinja2", memo=memo_text)

@app.route("/admin/notice_flag")
def admin_notice_flag():
    global memo_text
    if request.remote_addr != "127.0.0.1":
        return "Access Denied"
    if request.args.get("userid", "") != "admin":
        return "Access Denied 2"
    memo_text += f"[Notice] flag is {FLAG}\n"
    return "Ok"

app.run(host="0.0.0.0", port=1337)
```

***Phân tích nhanh***

- Trang `**/Vuln(csrf) page`.**
    
    Trang `**/Vuln**` của trang web trong quá trình xử lý tham số "param" truyền vào từ yêu cầu HTTP, các từ khóa potently nguy hiểm như "frame", "script", và "on" sẽ được bị thay thế bằng ký tự "*". Điều này làm giảm nguy cơ tấn công XSS (Cross-Site Scripting) bằng cách ngăn chặn việc sử dụng các tập lệnh JavaScript bất hợp pháp hoặc các frame bất hợp pháp trên trang web.
    
- Trang `**/memo`.**
    
    Trang `/memo` khi chúng ta truy cập nó sẽ truyền vào param là `memo`và biến `memo_text`có thể tùy ý thay đỗi. và `memo_text += text` ********cứ mỗi request thì ta thấy tại sao hello nó lại nối đuôi nhau.
    
- Trang **`/admin/notice_flag`.**
    
    Trang **`/admin/notice_flag`**thông báo hiển thị chuỗi `Access Denied` nếu địa chỉ IP của người dùng yêu cầu truy cập không phải là `127.0.0.1`.
    
    Nếu giá trị tham số `userid`không phải là `admin` thì flag không được hiển thị.
    
    Nếu địa chỉ IP là `127.0.0.1` và userid là `admin`, thì nó viết vào `memo`.
    
- Trang **`/flag`.**
    
    Trang **`/flag`** truyền giá trị tham số vào `?param=` và xuất ra một chuỗi `good`nếu hàm `check_csrf(param)` hoạt động bình thường.
    
    Nếu hàm `check_csrf(param)` không hoạt động bình thường thì chuỗi `wrong??` sẽ được xuất ra.
    
- Hàm `check_csrf()` và `read_url` :
    
    Hàm read_url() là hàm lấy url và cookie làm tham số.
    
    Khi hàm được gọi, giá trị biến cookie được cập nhật thành `{"domain": "127.0.0.1"}`.
    
    Và đối tượng trình điều khiển được khai báo.
    
    `driver`: Biến nơi lưu trữ đối tượng điều khiển trình duyệt web Chrome (webdriver) bằng thư viện Selenium của Python .
    
    Trình điều khiển kết nối với trang http://127.0.0.1:1337/ và thêm giá trị được lưu trong cookie.
    
    Và trình điều khiển kết nối với `http://127.0.0.1:1337/vuln?csrf={urllib.parse.quote(param)}`.
    
    `{urllib.parse.quote(param)}`: Đây là hàm mã hóa url một chuỗi không có định dạng ASCII.
    

**SẮP XẾP MANH MỐI**

1. Trang web này trong code có tính năng lọc ra các cú pháp (script,frame,on)  điều này có ta không thể sử dụng tấn công xxs 1 cách thông thường.
2. Trang **`memo`**là nơi flag xuất hiện cuối cùng.
3. trang **`notion_flag`**được thêm giá trị flag là biến cục bộ nếu giá trị truyền vào là `127.0.0.1` và `userid=admin` thì cho phép hiển thị flag.
4. Trang  flag là trang kết nối trang **`/vuln`** với mạng `127.0.0.1` của trang web.

## ****Exploiting SSRF****

Các cú pháp frame và script , on đã bị thay thế tuy nhiên ta có thể sử dụng các cú pháp khác như img, data…

**IDIE:** 

1. Để giải quyết vấn đề thì ta phải truy cập ip 127.0.0.1 mà ở trang /flag đã cung cấp cho chúng ta điều đó.
2. Mà flag được nằm ở đường dẫn /admin/notice_flag nếu userid=admin.

**KẾT HỢP Ý TƯỞNG :**

Ta tạo 1 tag img và chứa đường dẫn tới /admin/notice_flag  và gán parameter là userid với giá trị là admin.

**PAYLOAD:**

```jsx
<img src=/admin/notice_flag?userid=admin>
```

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%206.png)

hiện thông báo good có vẻ như hoạt động bình thường.

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%207.png)

Giờ ta qua trang memo xem kết quả.

![Untitled](/assets/writeup/cookie/MemoBabyCSRF/Untitled%208.png)

DONEEE ta đã được FLAG bài này

`Cảm ơn đã đọc nếu có góp ý gì xin liên hệ mình nhé`.

`Hack như là xem phim thám tử connan, thu thập manh mối và phân tích <3`



`***Written by Ren***`