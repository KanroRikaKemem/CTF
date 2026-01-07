## A. Tổng quan Forensics:
> *Link tham khảo: [GitHub](https://github.com/vonderchild/digital-forensics-lab)*

- Computer Forensics (hay còn gọi là điều tra số) là công việc phát hiện, bảo vệ và phân tích thông tin được lưu trữ, truyền tải hoặc được tạo ra bởi một máy tính hoặc mạng máy tính, nhằm đưa ra các suy luận hợp lý để tìm nguyên nhân và giải thích các hiện tượng trong quá trình điều tra.
- Mục tiêu: Phát hiện, bảo quản, khai thác, tài liệu hoá và đưa ra kết luận về dữ liệu thu thập được.
> Nếu một website nào đó bị tấn công và có những hậu quả không mong muốn, chủ nhân trang web sẽ cần tới Forensics để xác định nguyên nhân tấn công và tìm cách khắc phục, hay xa hơn là tìm thủ phạm.
> Ngoài ra còn những trường hợp khác như phát hiện mã độc, kiểm tra sự bất thường trong mạng, phát hiện sự xâm nhập,...
- Đặc điểm của Computer Forensics:
    - Dữ liệu cần phân tích rất nhiều, nếu chỉ là text vài mb thì cũng có một lượng thông tin rất lớn.
    - Dữ liệu thường không còn nguyên vẹn (bị thay đổi, phân mảnh và có thể bị lỗi).
    - Việc bảo quản dữ liệu khó khăn, dữ liệu thu được có thể có tính toàn vẹn cao, chỉ một thay đổi nhỏ cũng có thể làm ảnh hưởng đến tất cả.
    - Dữ liệu Forensics có thể gồm nhiều loại khác nhau (file hệ thống, ứng dụng,...).
    - Vấn đề trừu tượng (mã máy, dump file, network packet,...).
    - Dữ liệu dễ bị giả mạo.
    - Việc xác định tội phạm khó khăn, có thể tìm ra được dữ liệu về hacker (IP, email, profile,...) nhưng để xác định được đối tượng thật ngoài đời thì cũng không hề đơn giản.
- Computer Forensics thường làm việc với các đối tượng:
    - Physical Media, Media Management: Liên quan đến phần cứng, tổ chức phân vùng, phục hồi dữ liệu khi bị xoá,...
    - File System: Phân tích các file hệ thống, hệ điều hành Windows, Linux, Android,...
    - Application: Phân tích dữ liệu từ ứng dụng như các file log, file cấu hình, reverse ứng dụng,...
    - Network: Phân tích gói tin mạng, sự bất thường trong mạng.
    - Memory: Phân tích dữ liệu trên bộ nhớ, thượng là dữ liệu trên RAM được dump ra.

### I. Common Windows Artifacts:
[Link write - up](https://hackmd.io/@bMGaKJbHSWqauGaKAUyYhA/B1FRjpblZe)

### II. Document Analysis and SteganographyFile Forensics (File Formats):
[Link Write - up](https://hackmd.io/@bMGaKJbHSWqauGaKAUyYhA/BJOv97VX-g)

### III. Web Attack Forensics:
[Link Write - up](https://hackmd.io/@bMGaKJbHSWqauGaKAUyYhA/rktTQB5Vbe)

### III. Network Forensics:
#### 1. Tổng quan:
Công việc chính của nhóm nay là thu thập và phân tích các gói tin được truyền quan các thiết bị đầu cuối, từ đó phát hiện, cảnh báo các dấu hiệu bất thường trong hệ thống mạng.

#### 2. Công cụ chính:
- **Wireshark, Network Miner**: Bắt và phân tích gói tin với giao diện đồ hoạ.
- **Tcpdump**: Phân tích gói tin với giao diện console.
- **p0f**: Để phát hiện hệ điều hạnh, console trên nền Linux.
- **netcat**: Debug kết nối, đóng vai trò cả client và server, console trên Windows và Linux.
- **Snort**: Opensource phát hiện xâm nhập.
- **Nmap, tcpxtract, ssldump, nslookup, maxmind,...**

### IV. Memory Forensics:
### V. Disk Forensics:
### VI. Mail Forensics:
### VII. Malware Analyst:
## B. Một số Linux commands:
### 1. Thay đổi thư mục làm việc hiện tại:
- Command cơ bản: `cd [path]`
- Các trường hợp thường gặp:
    - `cd /home/user/Documents`: Di chuyển đến thư mục `/home/users/Documents`
    - `cd ..`: Quay lại thư mục cha của thư mục hiện tại.
    - `cd ../..`: Quay lại hai cấp thư mục cha.
    - `cd ~` hoặc chỉ `cd`: Quay về thư mục Home của người dùng.
    - `cd -`: Quay lại thư mục trước đó.
    - `cd /`: Di chuyển đến thư mục gốc (root).
    - `cd /etc`: Di chuyển trực tiếp vào thư mục `/etc`.
- Ví dụ:
![image](https://hackmd.io/_uploads/rkFTbkd6lg.png)

### 2. Liệt kê file và thư mục trong một thư mục:
- Command cơ bản: `ls [tuỳ chọn] [đường dẫn]`
- Các trường hợp thường gặp:
    - `ls`: Liệt kê các file và folder trong thư mục hiện tại.
    - `ls /etc`: Liệt kê nội dung trong thư mục `/etc`
    - `ls -l`: Hiển thị chi tiết (dạng danh sách có quyền, kích thước, ngày sửa,...)
    - `ls -a`: Hiển thị tất cả (kể cả tệp ẩn bắt đầu bằng `.`).
    - `ls -la` hoặc `ls -al`: Kết hợp hai tuỳ chọn chi tiết và hiện file ẩn.
    - `ls -lh`: Hiển thị kích thước đọc (KB, MB, GB).
    - `ls -lt`: Sắp xếp theo thời gian sửa đổi (mới nhất ở trên).
    - `ls -R`: Liệt kê đệ quy, bao gồm nội dung trong các thư mục con.
- Ví dụ:
::: spoiler `ls`
``` ubuntu
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls
tempCodeRunnerFile.python  test.cpp  test.exe  test.py  test1.py  venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls /etc
ImageMagick-6                  dnsmasq.d        ld.so.cache          os-release     shells
NetworkManager                 docker           ld.so.conf           pam.conf       skel
PackageKit                     dpkg             ld.so.conf.d         pam.d          smi.conf
R                              e2scrub.conf     ldap                 papersize      ssh
X11                            environment      legal                passwd         ssl
adduser.conf                   environment.d    libaudit.conf        passwd-        subgid
alternatives                   ethertypes       libnl-3              perl           subgid-
apache2                        fonts            libpaper.d           pm             subuid
apparmor                       fstab            lighttpd             polkit-1       subuid-
apparmor.d                     fuse.conf        locale.alias         profile        sudo.conf
apport                         gai.conf         locale.gen           profile.d      sudo_logsrvd.conf
apt                            gdb              localtime            protocols      sudoers
bash.bashrc                    ghostscript      logcheck             pulse          sudoers.d
bash_completion                glvnd            login.defs           python3        sysctl.conf
bash_completion.d              gprc             logrotate.conf       python3.10     sysctl.d
bindresvport.blacklist         groff            logrotate.d          rc0.d          systemd
binfmt.d                       group            lsb-release          rc1.d          terminfo
byobu                          group-           machine-id           rc2.d          texmf
ca-certificates                gshadow          magic                rc3.d          timezone
ca-certificates.conf           gshadow-         magic.mime           rc4.d          tmpfiles.d
ca-certificates.conf.dpkg-old  gss              manpath.config       rc5.d          ubuntu-advantage
cloud                          gtk-3.0          matplotlibrc         rc6.d          ucf.conf
cni                            hdparm.conf      mime.types           rcS.d          udev
console-setup                  host.conf        mke2fs.conf          rearj.cfg      ufw
cron.d                         hostname         modprobe.d           resolv.conf    update-manager
cron.daily                     hosts            modules              rmt            update-motd.d
cron.hourly                    hosts.allow      modules-load.d       rpc            vim
cron.monthly                   hosts.deny       mtab                 rsyslog.conf   vtrgb
cron.weekly                    init             nanorc               rsyslog.d      wgetrc
crontab                        init.d           netconfig            screenrc       wireshark
dbus-1                         inputrc          netplan              security       wsl.conf
dconf                          iproute2         network              selinux        xattr.conf
debconf.conf                   issue            networkd-dispatcher  sensors.d      xdg
debian_version                 issue.net        networks             sensors3.conf  xml
default                        java-11-openjdk  newt                 services       zsh_command_not_found
deluser.conf                   jupyter          nftables.conf        sgml
depmod.d                       kernel           nsswitch.conf        shadow
dhcp                           landscape        opt                  shadow-
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls -l
total 156
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2847 Sep 29 23:35 tempCodeRunnerFile.python
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2195 Oct 11 10:46 test.cpp
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 139762 Oct 11 10:43 test.exe
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem    637 Oct 10 09:37 test.py
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   1396 Oct  9 22:06 test1.py
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Oct  9 21:32 venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls -a
.  ..  .vscode  tempCodeRunnerFile.python  test.cpp  test.exe  test.py  test1.py  venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls -la
total 156
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Oct 11 10:43 .
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Oct 11 10:48 ..
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Sep 26 09:51 .vscode
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2847 Sep 29 23:35 tempCodeRunnerFile.python
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2195 Oct 11 10:46 test.cpp
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 139762 Oct 11 10:43 test.exe
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem    637 Oct 10 09:37 test.py
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   1396 Oct  9 22:06 test1.py
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Oct  9 21:32 venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls -lh
total 156K
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 2.8K Sep 29 23:35 tempCodeRunnerFile.python
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 2.2K Oct 11 10:46 test.cpp
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 137K Oct 11 10:43 test.exe
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem  637 Oct 10 09:37 test.py
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 1.4K Oct  9 22:06 test1.py
drwxrwxrwx 1 kanrorikakemem kanrorikakemem  512 Oct  9 21:32 venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls -lt
total 156
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2195 Oct 11 10:46 test.cpp
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem 139762 Oct 11 10:43 test.exe
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem    637 Oct 10 09:37 test.py
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   1396 Oct  9 22:06 test1.py
drwxrwxrwx 1 kanrorikakemem kanrorikakemem    512 Oct  9 21:32 venv
-rwxrwxrwx 1 kanrorikakemem kanrorikakemem   2847 Sep 29 23:35 tempCodeRunnerFile.python
```
:::

### 3. Xem nội dung, ghép hoặc tạo tệp văn bản:
- Command cơ bản: `cat [tuỳ chọn] [tên tệp]`
- Các trường hợp thường gặp:
    - `cat file.txt`: Hiển thị toàn bộ nội dung của `file.txt`
    - `cat file1.txt file2.txt`: Nối hai file lại và hiển thị kết quả.
    - `cat file1.txt file2.txt > file3.txt`: Gộp hai file lại và lưu vào `file3.txt`
    - `cat > newfile.txt`: Tạo file mới và nhập nội dung (nhấn Ctrl + D để lưu và thoát).
    - `cat >> existing.txt`: Thêm nội dung vào cuối file có sẵn.
    - `cat -n file.txt`: Hiển thị file kèm số thứ tự dòng.
    - `cat -b file.txt`: Đánh số chỉ các dòng có nội dung.
    - `cat -E file.txt`: Hiển thị ký hiệu `$` ở cuối mỗi dòng (giúp nhận biết xuống dòng).
- Ví dụ:
::: spoiler `cat`
``` ubuntu
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ ls
tempCodeRunnerFile.python  test.cpp  test.exe  test.py  test1.py  venv
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ cat test.py
from math import gcd
import random
from Crypto.Cipher import AES

k1 = 721919140332708275664160621428853988653441049264644517303176376909
k2 = 762740838008948738628397951381835990843814483667554565638672490531

g = gcd(k1, k2)
k = pow(2024, 2*g) - 1

random.seed(k)
key = random.randbytes(16)

iv = b"S\x0f\xac'\xd0\x18\xfb\xe3\x92\xfdoc\x93\x7fJ\xfc"
ciphertext = b"8\x9a>&=Q\xc14\xcf\xab\xac\xbaa@\xa0s\xd4T\x18\x1f\x82\x04F\xdb\xa2\xc2\xb9V\x04\xcbG6\xe54U\x1d2\xe2q\x0b\xe6\x9b\x1e\x8d\xc6\x8c7/"

cipher = AES.new(key=key, iv=iv, mode=AES.MODE_CBC)
flag = cipher.decrypt(ciphertext)
print(flag)
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ cat test.py test1.py
from math import gcd
import random
from Crypto.Cipher import AES

k1 = 721919140332708275664160621428853988653441049264644517303176376909
k2 = 762740838008948738628397951381835990843814483667554565638672490531

g = gcd(k1, k2)
k = pow(2024, 2*g) - 1

random.seed(k)
key = random.randbytes(16)

iv = b"S\x0f\xac'\xd0\x18\xfb\xe3\x92\xfdoc\x93\x7fJ\xfc"
ciphertext = b"8\x9a>&=Q\xc14\xcf\xab\xac\xbaa@\xa0s\xd4T\x18\x1f\x82\x04F\xdb\xa2\xc2\xb9V\x04\xcbG6\xe54U\x1d2\xe2q\x0b\xe6\x9b\x1e\x8d\xc6\x8c7/"

cipher = AES.new(key=key, iv=iv, mode=AES.MODE_CBC)
flag = cipher.decrypt(ciphertext)
print(flag)#!/usr/bin/env python3
import json, time
from Crypto.Util.number import bytes_to_long
from hashlib import sha1
from ecdsa.ecdsa import Public_key, Private_key, Signature, generator_192
from pwn import remote

HOST, PORT = "socket.cryptohack.org", 13381
g, n = generator_192, generator_192.order()
def hash_like_server(m): return bytes_to_long(sha1(str(m).encode()).digest())

io = remote(HOST, PORT);
io.recvuntil(b"verify.\n")
seen = {}
while True:
    io.sendline(json.dumps({"option":"sign_time"}).encode())
    line = io.recvline().decode().strip()
    try:
        sig = json.loads(line)
    except:
        try:
            sig = eval(line, {"Signature": Signature})
        except:
            continue

    if not all(k in sig for k in ("r", "s", "msg")):
        continue

    r = int(sig["r"],16);
    s = int(sig["s"],16);
    z = hash_like_server(sig["msg"])
    if r in seen:
        z1, s1 = seen[r]["z"], seen[r]["s"];
        z2, s2 = z, s
        k = ((z1 - z2)*pow(s1 - s2, -1, n))%n
        d = ((s1*k - z1)*pow(r, -1, n))%n
        break
    seen[r] = {"s": s, "z": z}

pub = Public_key(g, g * d);
priv = Private_key(pub, d)
h = hash_like_server("unlock");
sig = priv.sign(h, k)
io.sendline(json.dumps({"option": "verify", "msg": "unlock", "r": hex(sig.r),"s": hex(sig.s)}).encode())
print(io.recvline().decode())
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$ cat -n test.py
     1  from math import gcd
     2  import random
     3  from Crypto.Cipher import AES
     4
     5  k1 = 721919140332708275664160621428853988653441049264644517303176376909
     6  k2 = 762740838008948738628397951381835990843814483667554565638672490531
     7
     8  g = gcd(k1, k2)
     9  k = pow(2024, 2*g) - 1
    10
    11  random.seed(k)
    12  key = random.randbytes(16)
    13
    14  iv = b"S\x0f\xac'\xd0\x18\xfb\xe3\x92\xfdoc\x93\x7fJ\xfc"
    15  ciphertext = b"8\x9a>&=Q\xc14\xcf\xab\xac\xbaa@\xa0s\xd4T\x18\x1f\x82\x04F\xdb\xa2\xc2\xb9V\x04\xcbG6\xe54U\x1d2\xe2q\x0b\xe6\x9b\x1e\x8d\xc6\x8c7/"
    16
    17  cipher = AES.new(key=key, iv=iv, mode=AES.MODE_CBC)
    18  flag = cipher.decrypt(ciphertext)
    19  print(flag)
kanrorikakemem@DESKTOP-ICMNRNE:/mnt/c/Users/Ha Nguyen/Documents/Ngon_ngu_lap_trinh/Cpp/Bai_tap/Giai_bai_tap$
```
:::

### 4. Trích xuất và hiển thị các chuỗi ký tự có thể đọc được từ một tện nhị phân (binary file) hoặc tệp thực thi (executable file):
- Command cơ bản: `strings [tuỳ chọn] [tên tệp]`
- Các trường hợp thường gặp:
    - `strings -a`: Quét toàn bộ file (mặc định có thể bỏ qua phần đầu nếu là định dạng object)
    - `strings -n <số>`: Độ dài tối thiểu của chuỗi (mặc định là 4).
    - `strings -t x`: Hiển thị offset (vị trí trong file) ở hex.
    - `strings -t d`: Hiển thị offset ở decimal.
    - `strings -e l,b,B`: Xác định kiểu mã hoá: little-endian, big-endian hoặc cả hai (dành cho UTF-16).
- Ví dụ:
![image](https://hackmd.io/_uploads/ByLSdJdpex.png)

### 5. Tìm kiếm chuỗi kí tự trong nội dung file hoặc dòng đầu ra của lệnh khác:
- Command cơ bản: `grep [tuỳ chọn] "chuỗi cần tìm" [tên tệp]`
- Các trường hợp thường gặp:
    - `-i`: Không phân biệt chữ hoa - thường.
    - `-n`: Hiển thị số dòng..
    - `-r` hoặc `-R`: Tìm trong thư mục và các thư mục con.
    - `-v`: Lấy dòng không khớp với mẫu.
    - `-c`: Đếm số dòng khớp.
    - `-l`: Chỉ ra tên file chứa chuỗi cần tìm.
    - `-w`: Khớp toàn bộ từ.
    - `-A <số>`: In `<số>` dòng sau dòng khớp.
    - `-B <số>`: In `<số>` dòng trước dòng khớp.
    - `-C <số>`: In `<số>` dòng trước và sau dòng khớp.
- Ví dụ:
![image](https://hackmd.io/_uploads/Bk9ysJ_Tel.png)

### 6. Tìm kiếm tệp hoặc thư mục trong hệ thống dựa trên điều kiện:
- Command cơ bản: `find [path bắt đầu] [điều kiện] [hành động]`
> Ví dụ:
> - `find . -name "test.cpp"`: Tìm file tên chính xác là `test.cpp` trong thư mục hiện tại.
> - `find /home -name "*.cpp"`: Tìm tất cả file `.cpp` trong thư mục `/home`.
- Các trường hợp thường gặp:
    - `-name "pattern"`: Tìm theo tên (phân biệt hoa thường).
    - `-iname "pattern"`: Tìm theo tên (không phân biệt hoa thường).
    - `-type f` hoặc `-type d`: Chỉ tìm file hoặc thư mục.
    - `-size +N` hoặc `-size -N`: Lớn hơn hoặc nhỏ hơn kích thước `N` (đơn vị `c`, `k`, `M`, `G`).
    - `-mtime -n` hoặc `-mtime +n`: Sửa đổi trong vòng `n` ngày hoặc hơn `n` ngày trước.
    - `-pern mode`: Theo quyền (ví dụ `644`, `755`)
    - `-user tên`: File thuộc quyền sở hữu của user.
    - `-exec`: Thực thi lệnh cho mỗi kết quả tìm được.
- Ví dụ:
![image](https://hackmd.io/_uploads/ry1_SQKaxe.png)

### 7. Tính và kiểm tra mã băm của tệp tin:
- Command cơ bản:
    - `md5sum [tên tệp]`
    - `sha1sum [tên tệp]`
- Ví dụ:
![image](https://hackmd.io/_uploads/ryTQU7Ypgl.png)

### 8. Hiển thị thông tin về kết nối mạng, cổng đang mở, bảng định tuyến, giao diện mạng và trạng thái socket.
- Command cơ bản: `netstat [tuỳ chọn]`
- Các trường hợp phổ biến:
    - `netstat -a`: Hiển thị tất cả kết nối và cổng đang lắng nghe.
    - `netstat -t`: Chỉ hiện thị kết nối TCP.
    - `netstat -u`: Chỉ hiển thị kết nối UDP.
    - `netstat -l`: Chỉ hiện các cổng đang lắng nghe.
    - `netstat -p`: Hiển thị PID và tên tiến trình sử dụng kết nối.
    - `netstat -n`: Hiển thị địa chỉ và cổng dưới dạng số, không dịch sang tên miền.
    - `netstat -r`: Hiển thị bảng định tuyến - routing table.
    - `netstat -i`: Hiển thị thông tin giao diện mạng - interface
    - `netstat -s`: Hiển thị thống kê giao thức - protocol.
    - `netstat -tuln`: Hiển thị tất cả cổng TCP/UDP đang lắng nghe, không dịch tên.
    - `netstat -tulnp`: Thêm cột PID/Process để biết chương trình nào đang chiếm cổng.
    - `netstat -rn`: Xem routing table ở dạng số.
- Ví dụ:
![image](https://hackmd.io/_uploads/rk7yR7Faee.png)

### 9. Xác định loại của tệp:
- Command cơ bản: `file [tuỳ chọn] [tên tệp]`
- Một số trường hợp phổ biến:
    - `-b`: Ẩn tên file, chỉ hiển thị loại file.
    - `-i`: Hiển thị MME type (ví dụ `text/plain; charset=utf-8`)
    - `-z`: Kiểm tra file nén (`gzip`, `zip`,...)
    - `-f [tên file]`: Kiểm tra loại của nhiều file được liệt kê trong file `[tên file]`
- Ví dụ:
![image](https://hackmd.io/_uploads/SJiFy4FTee.png)

### 10. Hiển thị hoặc chuyển đổi dữ liệu nhị phân sang dạng hexdump và ngược lại:
- Command cơ bản: `xxd [tuỳ chọn] [tên tệp]`
> Ví dụ cấu trúc dòng hexdump:
> ``` yami
> 00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
> ```
> - `00000000`: Địa chỉ byte (offset)
> - `8950 4e47 ...`: Dữ liệu ở dạng hex
> - `.PNG........IHDR`: Phần hiển thị ASCII tương ứng (nếu in được)
> $\rightarrow$ Dễ dàng nhận ra file này là ảnh PNG vì có magic byté `89 50 4E 47`.
- Một số trường hợp phổ biến:
    - `-l <số byte>`: Chỉ in ra số byte cố định.
    - `-s offset`: Bắt đầu từ vị trí offset nhất định.
    - `-c <số byte>`: Hiển thị bao nhiêu byte mỗi dòng.
    - `-p`: Không hiển thị offset/ASCII.
    - `-r`: Chuyển từ hex sang nhị phân.
    - `-e`: Đảo ngược thứ tự byte - Endian swap.

### 11. Trình soạn thảo hex tương tác chạy trong terminal cho phép xem song song hex và ASCII và chỉnh sửa trực tiếp từng byte:
- Cú pháp cơ bản: `hexedit filename.bin`
- Khi mở sẽ thấy hai cột, bên trái là hex bytes, bên phải là ASCII.
- Các thao tác cơ bản:
    - Di chuyển: Dùng các phím mũi tên, PageUp/PageDown để cuộn, Home/End để về đầu cuối (hành vi tuỳ terminal).
    - Chỉnh sửa: Đặt con trỏ vào ô hex hoặc ASCII và gõ để thay byte. Khi gõ trên hex, kí tự sẽ là `0` - `9` và `a` - `f`. Khi gõ ở ASCII, nhập ký tự hiển thị.
    > Trước khi sửa hãy sao lưu file vì việc chỉnh sửa có thể phá file ngay lập tức (ví dụ `cp file.bin file.bin.bak`).
    - Tìm kiếm/Chuyển đổi tới offset: Nhiều phiên bản có chức năng tìm hoặc jump-to-offset (kiểm tra help trong chương trình hoặc `hexedit --help`).
    > Keybiding/Behavior có thể khác nhau giữa các phiên bản và terminal, nếu cần thao tác chính xác thì gõ `hexedit --help` hoặc xem manual `man hexedit`.
- Mẹo an toàn:
    - Luôn backup.
    - Làm việc trên bản copy, không sửa file gốc nếu quan trọng.
    - Nếu file là chương trình thực thi, test trên môi trường an toàn (VM).
    - Nếu chỉ muốn xem mà không sửa, kiểm tra xem hexedit có option read-only không; nếu không thì dùng `xxd`/`hexdump`/`hd` để xem.

### 12. Liệt kê tiến trình (process) đang chạy trên hệ thống:
- Command cơ bản: `ps [option]`
- Các trường hợp phổ biến:
    - `ps aux`: Danh sách đầy đủ tiến trình (BSD style). Thường dùng để nhìn nhanh mọi tiến trình của tất cả users.
    - `ps -ef`: Danh sách đầy đủ (System V style). Thích hợp khi muốn dùng `awk`/`grep` theo cột truyền thống.
    - `ps -u username`: Tiến trình của user cụ thể.
    - `ps -C name`: Tìm tiến trình theo tên.
    - `ps -eo pid,ppid,cmd,%temp%,%cpu --sort=-%mem`: Xuất cột tuỳ chỉnh và sắp xếp theo `%mem`.
    - `ps -elf`: Liệt kê tất cả tiến trình và các threads.
    - `ps --forest`: Hiển thị cây tiến trình.
- Các trường/cột phổ biến:
    - `USER`: Chủ sở hữu tiến trình.
    - `PID`: Process ID.
    - `PPID`: Parent PID (PID của tiến trình cha).
    - `%CPU`: %CPU đang sử dụng (thời điểm).
    - `%MEM`: %RAM đang dùng.
    - `VSZ`: Vitural memory size (KB).
    - `RSS`: Resident Set Size - Bộ nhớ thực tế đang chiếm (KB).
    - `TTY`: Terminal liên kết (nếu có).
    - `STAT`: Trạng thái tiến trình:
        - `R`: Running
        - `S`: Sleeping
        - `D`: Uninterruptible sleep
        - `Z`: Zombie
        - `T`: Stopped
        - `+`: Foreground process group
    - `START`: Thời gian tiến trình bắt đầu.
    - `TIME`: Tổng thời gian CPU đã dùng.
    - `COMMAND`: Câu lệnh/Đường dẫn thực thi (Với tham số).
- Ví dụ:
::: spoiler
``` ubuntu
kanrorikakemem@DESKTOP-ICMNRNE:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.1 165840 10744 ?        Ss   20:03   0:00 /sbin/init
root           2  0.0  0.0   3072  1792 ?        Sl   20:03   0:00 /init
root           8  0.0  0.0   3088  1920 ?        Sl   20:03   0:00 plan9 --control-socket 7 --log-level 4 --server-fd 8
root          62  0.0  0.1  56020 14628 ?        S<s  20:03   0:00 /lib/systemd/systemd-journald
root          85  0.0  0.0  23156  5888 ?        Ss   20:03   0:00 /lib/systemd/systemd-udevd
root          95  0.0  0.0 153124  1664 ?        Ssl  20:03   0:00 snapfuse /var/lib/snapd/snaps/jq_6.snap /snap/jq/6 -o
root          96  0.0  0.0 152992  1664 ?        Ssl  20:03   0:00 snapfuse /var/lib/snapd/snaps/core_17247.snap /snap/c
root          97  0.0  0.1 452048 11720 ?        Ssl  20:03   0:02 snapfuse /var/lib/snapd/snaps/snapd_25202.snap /snap/
systemd+     113  0.0  0.1  26332 13680 ?        Ss   20:03   0:00 /lib/systemd/systemd-resolved
systemd+     114  0.0  0.0  89364  7040 ?        Ssl  20:03   0:00 /lib/systemd/systemd-timesyncd
root         210  0.0  0.0   4308  2688 ?        Ss   20:03   0:00 /usr/sbin/cron -f -P
message+     212  0.0  0.0   8592  4480 ?        Ss   20:03   0:00 @dbus-daemon --system --address=systemd: --nofork --n
root         218  0.0  0.2  30212 18688 ?        Ss   20:03   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-s
syslog       219  0.0  0.0 222404  5504 ?        Ssl  20:03   0:00 /usr/sbin/rsyslogd -n -iNONE
root         221  0.0  0.4 2144820 36800 ?       Ssl  20:03   0:00 /usr/lib/snapd/snapd
root         222  0.0  0.0  15324  7168 ?        Ss   20:03   0:00 /lib/systemd/systemd-logind
root         232  0.1  0.5 2393600 47488 ?       Ssl  20:03   0:07 /usr/bin/containerd
root         252  0.0  0.0   3240  2176 hvc0     Ss+  20:03   0:00 /sbin/agetty -o -p -- \u --noclear --keep-baud consol
root         254  0.0  0.0   3196  2176 tty1     Ss+  20:03   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root         261  0.0  0.2 107188 21120 ?        Ssl  20:03   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unatt
root         290  0.0  0.9 2358560 76860 ?       Ssl  20:03   0:01 /usr/bin/dockerd -H fd:// --containerd=/run/container
root         691  0.0  0.0   7524  4608 pts/1    Ss   20:03   0:00 /bin/login -f
kanrori+     748  0.0  0.1  17076  9472 ?        Ss   20:03   0:00 /lib/systemd/systemd --user
kanrori+     749  0.0  0.0 168888  5112 ?        S    20:03   0:00 (sd-pam)
kanrori+     767  0.0  0.0   6236  5120 pts/1    S+   20:03   0:00 -bash
root        4687  0.0  0.2 293056 20224 ?        Ssl  20:30   0:00 /usr/libexec/packagekitd
root        4691  0.0  0.0 234508  7296 ?        Ssl  20:30   0:00 /usr/libexec/polkitd --no-debug
root        5256  0.0  0.0   3080  1028 ?        Ss   21:12   0:00 /init
root        5257  0.0  0.0   3096  1032 ?        D    21:12   0:00 /init
root        5258  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
root        5259  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
root        5260  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
root        5261  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
root        5262  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
root        5263  0.0  0.0  23156  3696 ?        S    21:12   0:00 /lib/systemd/systemd-udevd
kanrori+    5264  0.6  0.0   6332  5120 pts/0    Ss   21:12   0:00 -bash
kanrori+    5277  0.0  0.0   7484  3072 pts/0    R+   21:13   0:00 ps aux
```
:::
- Mẹo và lưu ý:
    - `ps` cho snapshot tại thời điểm chạy, không cập nhật liên tục. Dùng `top` hoặc `htop` để theo dõi real time.
    - Trên hệ thống có nhiều thread, `ps -elf` sẽ cho thấy LWP (light-weight process/thread id).
    - Muốn output tuỳ chỉnh (dễ parse), dùng `-o` (ví dụ `ps -o pid,ppid,cmd`) để tiện cho script.
    - `STAT` có nhiều ký tự (ví dụ `Sl` = sleeping, multi-threaded, `S` + `l`).
