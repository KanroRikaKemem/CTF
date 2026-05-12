> Trong thế giới bảo mật số, mật khẩu đóng vai trò quan trọng trong việc bảo vệ sự an toàn cho thông tin nhajt cảm, giống như những cánh cửa bảo vệ ngôi nhà khỏi những người xâm nhập. Mật khẩu là cách xác thực users và bảo vệ thông tin nhạy cảm khỏi những truy cập trái phép. Tuy nhiên, giống như một cái cửa có thể bị mở bằng những công cụ và công nghệ phù hợp, mật khẩu cũng có thể bị phá.
> Là một người điều tra pháp y số, ta có thể cần dùng tới những công nghệ crack mật khẩu để có quyền truy cập dữ liệu mã hoá, nhận diện vấn đề độc hại, và hành động. Do đó, trong lab này, ta sẽ thảo luận về định nghĩa của crack mật khẩu, những công nghệ khác nhau để làm việc này, và những tools hỗ trợ điều tra.

## I. Mật khẩu và Mã băm:
- Mật khẩu thường được lưu trữ dưới dạng mã băm, một hàm toán học một chiều chuyển đổi dữ liệu bản rõ của mật khẩu thành một chuỗi hoặc ký tự có độ dài cố định. Điều này có nghĩa là một khi mật khẩu được chuyển thành dạng mã băm, nó sẽ cực kỳ khó để chuyển nó ngược lại thành dạng bản rõ. Tuy nhiên, vẫn có những cách khác để quyết định xem mật khẩu có tương ứng với mã băm được cho không.
- Trước khi khảo sát những công nghệ crack mã băm, đầu tiên phải hiểu về cách các ứng dụng dùng mã băm để xác thực user khi đăng nhập.

### 1. Sử dụng mã băm trong việc xác thực:
- Khi đăng ký như một user trên một website hay ứng dụng, mật khẩu sẽ được chuyển thành dạng mã băm, và mã băm đó sẽ được lưu trữ trong database. Khi nhập mật khẩu để đăng nhập, website hay ứng dụng sẽ chuyển nó thành mã băm và so sánh với mã băm đã có trong database. Nếu chúng không giống nhau, user sẽ bị từ chối truy cập.
- Có nhiều loại hàm băm mà một ứng dụng có thể dùng để chứa mật khẩu: MD5, SHA-1, SHA-2, SHA-3, Bcrypt, Scrypt, Argon2.

### 2. Biểu diễn mã băm:
Hàm băm thường cho output là dữ liệu nhị phân, dữ liệu đó sẽ được chuyển thành dạng khác như hex hay base64.
> Ví dụ, hàm băm MD5 của chuỗi `test` là `098f6bcd4621d373cade4e832627b4f6`, nếu dùng SHA1 thì nó là `a94a8fe5ccb19ba61c4c0873d391e987982fbbd3`.

## II. Crack mã băm:
- Các phương pháp crack mã băm đều sử dụng cách so sánh mã băm. Nghĩa là lấy mã băm của một mật khẩu tiềm năng và so sánh nó với mã băm mà ta đang cố gắng crack. Nếu cả hai khớp với nhau, ta đã tìm ra mật khẩu.
> - Thông thường, hai chuỗi phân biệt sẽ không tạo ra được hai mã băm giống nhau. Tuy nhiên, trong trường hợp rất hiếm, việc gặp được hiện tượng va chạm mã băm hoàn toàn có thể, xảy ra khi hai đầu vào khác nhau cho ra mã băm đầu ra giống nhau.
> Có thể tìm hiểu thêm ở: https://en.wikipedia.org/wiki/Hash_collision
- Có nhiều phương pháp khác nhau để crack mã băm như tấn công brute-force, tấn công từ điển, hay tấn công bảng cầu vòng. Để thực hiện các loại tấn công này, ta có thể dùng hai tool nổi tiếng là [John The Ripper](https://github.com/openwall/john) và [Hashcat](https://github.com/hashcat/hashcat).
- Có thể tải bằng command `sudo apt-get install -y hashcat john`.

### 1. Nhận diện loại mã băm:
- Việc biết được loại mã băm nào ta cần giải quyết rất quan trọng, nó giúp quyết định xem ta sẽ dùng công nghệ nào để để crack mã băm. Có thể dùng [hash-identifier](https://github.com/blackploit/hash-identifier) hoặc online tool như [Hash Type Identifier](https://hashes.com/en/tools/hash_identifier).
- Xét ví dụ là mã băm MD5 của chuỗi `test` là `098f6bcd4621d373cade4e832627b4f6`:
![image](https://hackmd.io/_uploads/Sykjn3eyGl.png)
Kết quả là MD5.
> Tải `hash-identifier`:
> ``` ubuntu
> git clone https://github.com/blackploit/hash-identifier.git
> cd hash-identifier
> chmod +x hash-id.py
> python3 hash-id.py <hash_value>
> ```

### 2. Tấn công Brute-force:
- Tấn công này thử dùng tất cả những sự kết hợp có thể của các kí tự cho đến khi tìm ra mật khẩu. Nó tốn thời gian và nguồn lực, nhưng hiệu quả nếu mật khẩu yếu.
- Có thể dùng Hashcat để làm rõ các bộ ký tự, độ dài mật khẩu, và các tham số khác để điều chỉnh cuộc tấn công.
- Hashcat có một danh sách các chế độ băm [link](https://hashcat.net/wiki/doku.php?id=example_hashes) và các chế độ tấn công ở [link](https://hashcat.net/wiki/). Ta có thể chỉ ra mask `?l?l?l?l`, nghĩa là mật khẩu đang nhắm đến gồm bốn chữ cái viết thường.
    ``` ubuntu
    kanrorikakemem@DESKTOP-ICMNRNE:~/hash-identifier$     hashcat -m 0 -a 3 098f6bcd4621d373cade4e832627b4f6 ?l?l?l?l
    hashcat (v6.2.5) starting

    OpenCL API (OpenCL 2.0 pocl 1.8  Linux, None+Asserts, RELOC, LLVM 11.1.0, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
    =====================================================================================================================================
    * Device #1: pthread-12th Gen Intel(R) Core(TM) i5-12450H, 2820/5705 MB (1024 MB allocatable), 12MCU

    Minimum password length supported by kernel: 0
    Maximum password length supported by kernel: 256

    Hashes: 1 digests; 1 unique digests, 1 unique salts
    Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates

    Optimizers applied:
    * Zero-Byte
    * Early-Skip
    * Not-Salted
    * Not-Iterated
    * Single-Hash
    * Single-Salt
    * Brute-Force
    * Raw-Hash

    ATTENTION! Pure (unoptimized) backend kernels selected.
    Pure kernels can crack longer passwords, but drastically reduce performance.
    If you want to switch to optimized kernels, append -O to your commandline.
    See the above message to find out about the exact limits.

    Watchdog: Temperature abort trigger set to 90c

    Host memory required for this attack: 3 MB

    098f6bcd4621d373cade4e832627b4f6:test

    Session..........: hashcat
    Status...........: Cracked
    Hash.Mode........: 0 (MD5)
    Hash.Target......: 098f6bcd4621d373cade4e832627b4f6
    Time.Started.....: Tue May 12 21:51:27 2026 (0 secs)
    Time.Estimated...: Tue May 12 21:51:27 2026 (0 secs)
    Kernel.Feature...: Pure Kernel
    Guess.Mask.......: ?l?l?l?l [4]
    Guess.Queue......: 1/1 (100.00%)
    Speed.#1.........:   628.3 kH/s (1.36ms) @ Accel:512 Loops:26 Thr:1 Vec:8
    Recovered........: 1/1 (100.00%) Digests
    Progress.........: 159744/456976 (34.96%)
    Rejected.........: 0/159744 (0.00%)
    Restore.Point....: 0/17576 (0.00%)
    Restore.Sub.#1...: Salt:0 Amplifier:0-26 Iteration:0-26
    Candidate.Engine.: Device Generator
    Candidates.#1....: sari -> xhir
    Hardware.Mon.#1..: Util:  7%

    Started: Tue May 12 21:51:00 2026
    Stopped: Tue May 12 21:51:28 2026
    ```
    Ta đã tìm ra chuỗi `test`.
> Tìm hiểu sâu hơn về tấn công Brute-force và Mask bằng cách sử dụng Hashcat: https://hashcat.net/wiki/doku.php?id=mask_attack

### 3. Tấn công từ điển:
Tấn công này dùng một từ điển đã tạo trước gồm các từ và cụm từ, gọi là từ điển, để thử và đoán mật khẩu. Ý tưởng này là mọi người tạo mật khẩu từ các từ và cụm từ phổ biến, vì vậy nếu thử tất cả các từ có trong từ điển có thể đoán được mật khẩu.
- Có thể dùng John The Ripper để thực hiện tấn công:
![image](https://hackmd.io/_uploads/SJI2JalJfg.png)
- Lưu mã băm dưới dạng file: `echo "098f6bcd4621d373cade4e832627b4f6" > hash.txt`
- Ta phải có một từ điển để crack mã băm, thường là `rockyou.txt` (chứa hơn một triệu mật khẩu). Nó nằm ở `/usr/share/wordlists/` trong Kali Linux, để giải nén:
    ``` ubuntu
    gunzip /usr/share/wordlists/rockyou.txt.gz
    ```
    Nếu không dùng Kali, có thể tải bằng command:
    ``` ubuntu
    $ wget https://github.com/praetorian-inc/Hob0Rules/raw/master/wordlists/rockyou.txt.gz
    ```
- Thử tấn công:
![image](https://hackmd.io/_uploads/BkRrM6lyzx.png)
Output đã đưa ra chuỗi `test`.

## III. Kết luận:
Tóm lại, ta đã học được những kiến thức cơ bản về crack hàm băm và các tool liên quan là John the Ripper và Hashcat. Tuy nhiên, đây chỉ là phần nổi của tảng băng vì còn rất nhiều kỹ thuật nâng cao hơn có thể được sử dụng để bẻ khóa các hàm băm phức tạp hơn. Mặc dù vậy, bài lab này đã cung cấp nền tảng vững chắc để tiếp tục khám phá chủ đề này.
