## I. Tổng quan File System
- File system được đùng để quản lý cách dữ liệu được đọc và lưu trên thiết bị.
- File system cho phép user truy cập nhanh chóng và an toàn khi cần thiết.
> Nếu không có nó, tất cả dữ liệu sẽ được đặt vào một thiết bị lưu trữ mà không biết được điểm đầu và cuối.
- Hiện nay có rất nhiều loại file system, mỗi loại khác nhau về cấu trúc, logic, tốc độ, độ linh hoạt, tính bảo mật, kích thước,...
- Có thể được dùng trên nhiều loại thiết bị lưu trữ khác nhau như HDD, SSD, USB, RAM,...
> ![image](https://hackmd.io/_uploads/BJz6VRZwbe.png)
> ![image](https://hackmd.io/_uploads/HJoGuRZPZx.png)

### 1) Cấu trúc Linux File System:
![image](https://hackmd.io/_uploads/rJ7_3AWw-x.png)
Được tổ chức thành ba lớp quan trọng, mỗi cái đảm nhiệm chức năng khác nhau. Chúng làm việc cùng như để việc truy cập, lưu trữ và quản lý file trở nên mượt mà:

#### a) Logical File System:
- Đóng vai trò như giao diện giữa người dùng và file system.
- Xử lý các theo tác chính như mở, đọc, ghi và đóng.
- Cung cấp các kiểm tra bảo mật như các quyền và kiểm soát việc truy cập file.
- Đảm bảo các app tương tác với files theo cách đơn giản và thích hợp mà không lo lắng về chi tiết lưu trữ.

#### b) Virtual File System (VFS):
- Cung cấp giao diện phổ biến cho các file systems khác nhau (`ext4`, `XFS`, `FAT32`, `NTFS`,...)
- Cho phép Linux dùng nhiều loại file system khác nhau tại cùng một thời điểm.
- Đóng vai trò như một lớp trừu tượng, ẩn đi sự phức tạp phía trong mỗi file system.
- Khiến file vận hành đồng bộ và tương thích bất kể format file system là gì.

#### c) Physical File System:
- Tương tác trực tiếp với phần cứng và ổ đĩa lưu trữ.
- Quản lý data blocks, inodes và phân vùng bộ nhớ vật lý.
- Đảm nhiệm việc ghi dữ liệu cho ổ đĩa và khôi phục nó một cách hiệu quả.
- Đảm bảo việc lưu trữ đáng tin cậy, xử lý lỗi và quản lý dữ liệu cấp thấp.

### 2. Cấu trúc dữ liệu trong Linux File System:
Một file system dựa vào các cấu trúc dữ liệu khác nhau để tổ chức và quản lý dữ liệu. Các cấu trúc này đảm bảo dữ liệu được lưu trữ, truy cập và duy trì một cách hiệu quả. Những cấu trúc dữ liệu chính gồm:
- Inodes: Lưu trữ metadata về files và thư mục.
- Superblock: Chứa thông tin về filesystem (size, block size, vị trí của các cấu trúc quan trọng khác).
- Block Groups: File systems lớn được chia thành các block groups, mỗi cái chứa tập các blocks, inodes và các cấu trúc dữ liệu liên kết với nó để cải thiện việc quản lý và vận hành.
- Bitmaps: Dùng để theo dõi cây, blocks và inodes được dùng trong file system.

#### a) Cấu trúc thư mục:
- Thư mục là một loại file đặc biệt lưu trữ danh sách các filename và số inode tương ứng. Tổ chức có thứ bậc này cho phép điều hướng và quản lý file một cách hiệu quả.
- Cấu trúc thư mục trong Linux file system thường tuân theo cấu trúc phân cấp dạng cây, với thư mục gốc `/` ở trên cùng và nhiều thư mục con phân nhánh ra.

#### b) File Allocation:
- Cho biết cách dữ liệu được lưu trữ trên đĩa. Có nhiều phương pháp khác nhau để phân bổ file:
    - Phân cấp kề: Files được lưu trữ trong các blocks kề nhau trên đĩa. Phương pháp này đơn giản và nhanh nhưng có thể gây ra sự phân mảnh và khó khăn trong việc tìm các không gian lớn kề nhau.
    - Phân cấp liên kết: Mỗi file là một danh sách liên kết với disk blocks. Phương pháp này tránh việc phân mảnh như có thể chậm hơn vì phải duyệt qua danh sách để truy cập dữ liệu.
    - Phân cấp index: Dùng index block để theo dõi toàn bộ disk blocks được phân bổ cho một file. Phương pháp này cung cấp khả năng truy cập ngẫu nhiên và tối thiểu hoá sự phân mảnh.
- Việc quản lý hiệu quả không gian trống cũng rất quan trọng với sự vận hành của file system. Một số công nghệ phổ biến:
    - Bitmaps: Dùng bit array để theo dõi blocks free và blocks đã dùng. Mỗi bit tượng trưng cho một block, với `0` là free và `1` là đã dùng.
    - Free Lists: Duy trì danh sách blocks free có thể được phân bổ nhanh khi cần.

### 3. Đặc điểm File System:
Một file system định nghĩa cấu trúc, quy tắc và phương pháp cho cách dữ liệu được tổ chức, lưu trữ, truy cập và quản lý trên thiết bị lưu trữ:

#### a) Quản lý không gian:
- Quyết định cách dữ liệu được lưu trữ bằng cách dùng blocks, phương pháp phân vùng và kiểm soát phân mảnh.
- Đảm bảo việc sử dụng hiệu quả không gian bằng việc quản lý các blocks tự do, bảng phân vùng và công nghệ lưu trữ tối ưu.

#### b) Filename:
- Áp dụng các quy tắc về độ dài, các ký tự được phép và phân biệt chữ hoa hay thường.
- Đảm bảo tên file phù hợp để tránh mâu thuẫn và duy trì sự tương thích trong hệ thống.

#### c) Directory:
- Tổ chức files theo cấu trúc tuyến tích hoặc có thứ bậc để dễ dàng điều hướng.
- Duy trì index/bảng để lần dấu files trong thư mục và thư mục con một cách hiệu quả.

#### d) Metadata:
- Lưu trữ các thông tin quan trọng về file như size, quyền, dấu thời gian và loại file.
- Giúp hệ thống quản lý file bằng cách theo dõi các thuộc tính cần thiết cho việc truy cập và chỉnh sửa.

#### e) Utilities:
- Cung cấp các hoạt động vận hành như khởi tạo, xoá, copy, di chuyển, đổi tên và backup file.
- Hỗ trợ các chức năng như các tools khôi phục và quản lý việc kiểm soát truy cập.

#### f) Design:
- Mỗi file system đều có giới hạn file size max, số lượng file và dung lượng lưu trữ.
- Bị ảnh hưởng bởi cấu trúc nội bộ, tác động đến hiệu năng, độ tin cậy và khả năng mở rộng.

### 4. Các loại file system phổ biến:
Các loại file system được Linux hỗ trợ:
- File system cơ bản: `EXT2`, `EXT3`, `EXT4`, `XFS`, `Btrfs`, `JFS`, `NTFS`,...
- File system dành cho dạng lưu trữ Flash: Thẻ nhớ,...
- File system dành cho hệ database cần lưu trữ và truy cập hiệu quả.
- File system mục đích đặc biệt: `procfs`, `sysfs`, `tmpfs`, `squashfs`, `debugfs`,...

### 5. Phân vùng và file system:
- Một phân vùng là một vùng chứa trong đó có một file system được lưu trữ, trong vài trường hợp thì file system có thể mở rộng hơn một phân vùng nếu file system dùng các liên kết.
- File system là một phương pháp lưu trữ hoặc tìm kiếm các files trên một đĩa cứng (trong một phân vùng).
- So sánh giữa file system trên Windows và Linux:
  
![image](https://hackmd.io/_uploads/H1Kfv0bPZl.png)

## II. Các tính năng nâng cao:
### 1. Journaling - Nhật ký:
Duy trì một log đặc biệt gọi là nhật ký ghi lại các sự thay đổi file system trước khi chúng được ghi vĩnh viễn vào đĩa.

#### a) Cách hoạt động:
- Các thay đổi được ghi lại trong nhật lý.
- Sau đó chúng được ghi vào vào ổ đĩa thật.
- Cuối cùng, chúng được đánh dấu là đã hoàn thành.

#### b) Journaling Modes:
Nhật ký có thể vận hành trong ba mode khác nhau, mỗi cái đều mang lại sự cân bằng giữa tốc độ và sự an toàn dữ liệu:
- Journal Mode (An toàn nhất):
    - Ghi lại cả data và metadata của file.
    - Đáng tin cậy nhưng chậm nhất do log nặng.
- Ordered Mode (Balanced):
    - Ghi lại mỗi metadata nhưng đảm bảo rằng data của file đã được ghi trước metadata.
    - Cân bằng tốt hiệu năng và sự an toàn.
- Writeback Mode (Nhanh nhất):
    - Ghi lại mỗi metadata mà không bắt buộc về thứ tự ghi.
    - Nhanh nhất nhưng kém an toàn nhất và dễ bị lỗi dữ liệu hơn.

### 2. Versioning - Phiên bản:
- Versioning file systems theo dõi phiên bản trước của một file, cho phép người dùng khôi phục lại bản copy cũ nếu cần.
- Key Points:
    - Lưu trữ trạng thái trước đó của files dựa trên commits, khoảng thời gian (phút/giờ) hay events hệ thống.
    - Hữu ích khi backup, theo dõi lịch sử và khôi phục dữ liệu xoá nhầm.

### 3. Inode - Index node:
- Là một cấu trúc dữ liệu lưu trữ các thông tin quan trọng về files và thư mục.
- Bao gồm:
    - Kích thước, quyền và chi tiết về quyền sở hữu.
    - Lưu trữ vị trí (trỏ đến data blocks thực tế trên đĩa). Những con trỏ có thể là:
        - Trỏ trực tiếp đến data blocks.
        - Trỏ không trực tiếp: Trỏ đến blocks chứa các trỏ khác đến data blocks.
        - Double indirect pointers: Trỏ đến blocks chứa con trỏ đến blocks of pointers.
        - Triple indirect pointers: Mở rộng cấu trúc phân cấp này hơn, cho phép quản lý các files lớn hiệu quả.
- Mỗi inode có mã định danh duy nhất là số inode. Số này được dùng bởi file system để truy cập tới inode và data liên kết với nó. Khi một file được truy cập, hệ thống vận hành số inode để định vị inode và có được thông tin cần thiết để truy cập đến data blocks của file.
- Đóng vai trò như sự định danh nội bộ của files trên Linux file systems.

#### a) Đánh số inode:
- Số inode trên filesystem bắng đầu từ $1$. Mười inode đầu tiên dành riêng cho việc sử dụng trong hệ thống. Các user file có superdata được lưu trữ từ inode $11$. Tất cả các inode được xếp chồng lên nhau gọn gàng trong một bảng inode.
- Một mục trong bảng inode có kích thước 256 bytes. Đối với một file, Linux sắp xếp thông minh tất cả các super data trong phạm vi 256 bytes này. Ngoài ra, mỗi inode cho một file cũng sẽ có thông tin về vị trí của dữ liệu trong filesystem. Chỉ có superdata của file được lưu trữ trong inode.
- Tổng số inode trong một filesystem phụ thuộc vào không gian có sẵn và số lượng file có thể được lưu trữ trên phân vùng.

#### b) Phân bổ và giải phóng inode:
- Khi user thêm file vào filesystem được định dạng mới, các inode bắt đầu từ 11 được phân bổ để giữ file superdata.
- Có một cấu trúc dữ liệu khác là Inode Bitmap dùng để theo dõi trạng thái phân bổ của một inode. Đây là một tập hợp các bit hoạt động như một bản đồ.
- Khi file có superdata bị xoá, trạng thái bitmap tương ứng sẽ trở thành `0`.
> ![image](https://hackmd.io/_uploads/rkoUHp-t-l.png)
> - Xem xét 8 bit trong bitmap inode để biểu thị trạng thái phân bổ của các inode 11 đến 18 trong bảng. Giá trị `1` trong bitmap nghĩa là inode đã được phân bổ để giữ superdata cho một file, giá trị `0` nghĩa là inode hiện không được dùng.
> - Trong trường hợp file có superdata trong inode 17 bị xoá thì trạng thái bitmap tương ứng của nó sẽ trở thành `0`, cho biết nó có thể được dùng tự do bởi file khác.
>   
> ![image](https://hackmd.io/_uploads/ry-yIaWKWe.png)

#### c) Cách xem số inode cho một file:
Có hai cách:
- Dùng lệnh `ls` với switch `-i`, theo sau là tên file. Trường đầu tiên trong đầu ra là số inode có superdata của file.
- Dùng `stat <filename>`:
  
![image](https://hackmd.io/_uploads/rJq88TbFbl.png)
- Để xem tổng số lượng inode có sẵn cho một phân vùng, dùng lệnh `df` với switch `-i`.

### 4. Block Sizes -  Đơn vị lưu trữ dữ liệu:
- Block là đơn vị lưu trữ dữ liệu nhỏ nhất trong file system.
- Block size quyết định độ chi tiết của việc lưu trữ dữ liệu và ảnh hưởng đến hiệu suất cũng như hiệu quả của file system.
- Block sizes thông thường là 512 bytes, 1KB, 2KB, 4KB, 8KB. Tuỳ kích thước mà  có thể ảnh hưởng đáng kể đến hiệu suất của file system và lượng dung lượng lưu trữ bị lãng phí (phân mảnh nội bộ).
- Việc chọn block size phù hợp phụ thuộc vào những yếu tố khác nhau:
    - Sự phân bổ kích thước file: Nếu file system lưu trữ nhiều files nhỏ, dùng block size nhỏ sẽ tránh việc không gian bị lãng phí và ngược lại với files lớn.
    - Yêu cầu về hiệu năng: Block sizes lớn hơn có thể dẫn đến hiệu năng tốt hơn đối với các thao tác đọc và ghi tuần tự lớn nhưng có thể làm tăng chi phí cho các thao tác I/O ngẫu nhiên nhỏ.
    - Hiệu quả lưu trữ: Block sizes nhỏ hơn giảm việc lãng phí dung lượng nhưng tăng chi phí để quản lý nhiều blocks.
- Block size tác động đến file system theo nhiều cách:
    - Hiệu quả đọc/ghi: Blocks lớn hơn có thể cải thiện hiệu quả của thao tác đọc và ghi files lớn vì số lượng blocks cần truy cập ít hơn. Tuy nhiên, nó có thể làm tăng việc phân mảnh và lãng phí dung lượng nếu có nhiều files nhỏ.
    - Thời gian truy cập file: Blocks nhỏ hơn có thời gian truy cập các files nhỏ nhanh hơn vì lượng data cần đọc hoặc ghi ít hơn. Tuy nhiên, việc quản lý nhiều file nhỏ làm tăng chi phí của file system.
    - Tối ưu hóa dung lượng lưu trữ: Các blocks nhỏ hơn giúp giảm dung lượng bị lãng phí (phân mảnh nội bộ) nhưng có thể tăng số blocks file system phải quản lý.

### 5. Superblock:
- Lưu trữ toàn bộ filesystem layout trong đĩa.
- Bắt đầu từ offset 1024 bytes từ đầu đĩa và trải dài 1024 bytes.
- Chứa block size, block count, group size và inode count cùng các tham số disk parameters được mô tả trong cấu trúc dữ liệu superblock.

#### a) Trích xuất thông tin đĩa:
Ta có thể trích xuất thông tin superblock từ đĩa:
``` linux
df -hT /
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda1      ext4   49G   41G  5.5G  89% /
sudo dd if=/dev/sda1 of=superblock.dat bs=1024 count=1 skip=1 status=none
```
- Ở đây, lệnh `df` in ra mức độ sử dụng và chi tiết về loại phân vùng root. Tên thiết bị là `/dev/sda1` và loại filesystem là `ext4`.
- Lệnh `dd` trích xuất dữ liệu superblock từ đĩa. Nó skip một block 1024 bytes đến vị trí bắt đầu của superblock. Tiếp theo nó đọc một block 1024 bytes chứa thông tin của superblock. Cuối cùng dữ liệu được lưu trong `superblock.dat`.

#### b) Important Fields:
Từ cấu trúc dữ liệu superblock, có thể thấy thông tin block size nằm ở offset `0x18`. Dùng lệnh `hexdump` để đọc nó:
``` linux
hexdump -e '"s_log_block_size: " /4 "%d\n"' -s 0x18 -n 4 superblock.img 
s_log_block_size: 2
```
- Các options được dùng trong command:
    - `-e`: Chỉ định chuỗi định dạng output.
    - `/4`: Biểu thị cần đọc ở dạng giá trị nguyên 4 byte.
    - `%d`: In ra dạng thập phân.
    - `\n`: In xuống dòng mới.
    - `-s`: Skip `0x18` bytes.
    - `-n`: Độ dài bytes cần xuất.
- Có thể thấy rằng, command trên in ra block size là `2`, nó là giá trị $log2$. Để chuyển nó thành thập phân, ta có thể dùng công thức $2^{10 + s\_log\_block\_size}$. Với giá trị $2%, ta có $4096$ ($2^{12}$) là block size.
- Tương tự, ta có thể xuất phần còn lại của các tham số trong superblock dựa trên cấu trúc dữ liệu.
- Một số key fields:
  
![image](https://hackmd.io/_uploads/Sk_veTbF-g.png)
- Từ đây, có thể suy ra rằng các vị trí bộ nhớ trên ổ đĩa được nhóm lại cùng nhau thành một block có size là $4096$. Các blocks này được kết hợp tiếp thành block groups có $32768$ blocks. Và có $400$ ($13106688/32768$) block groups.
- Một khi ta hiểu được các giá trị này, ta có thể xem xét mô tả về block group để tìm thông tin về inode.

### 6. Mô tả về Block Group:
- Là một bảng nội dung về các groups, chứa các block numbers nơi ta có thể tìm data block bitmap, inode bitmaps, bảng inode và các tham số khác.
- Nằm ở block thứ hai, chứa các data của tất cả groups khả dụng trên đĩa.
- Dữ liệu trong mô tả về block group được giải thích trong [Block Group Descriptors](https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout#Block_Group_Descriptors).
- Kích thước của một bảng mô tả block group là $64$ bytes.
- Vị trí của bảng mô tả cho bất kỳ group nào có thể được tính bằng công thức $64*(group - 1)$.
- Chúng ta chủ yếu tìm kiếm trong bảng vị trí inode và ở offset `0x8`:
``` linux
sudo dd if=/dev/sda1 of=bgd.dat bs=4096 count=1 skip=1 status=none
hexdump -v -e '"Inode table: " /4 "%d\n"' -s 0x8 -n 4 bgd.dat
Inode table: 1064
```
Lệnh `dd` trích xuất một block dữ liệu, bỏ qua cái đầu tiên. Lệnh `hexdump` in ra địa chỉ block của bảng inode là `1064`.

### 7. Mounting:
- Một filesystem phải được gắn kết để có thể được dùng bởi hệ thống. Để quan sát cái gì hiện tại được mount (có sẵn để dùng) trên hệ thống, dùng lệnh:
``` linux
$ mount
/dev/vzfs on / type reiserfs (rw,usrquota,grpquota)
proc on /proc type proc (rw,nodiratime)
devpts on /dev/pts type devpts (rw)
$
```
- Thư mục `/mnt`, theo quy ước, là vị trí của những sự mount tạm thời (như CD-ROM, các đĩa mềm,...). Nếu cần mount một file system, dùng lệnh `mount` với cú pháp:
``` linux
mount -t file_system_type device_to_mount directory_to_mount_to
```
> Ví dụ: Nếu muốn mount một CD-ROM đến `/mnt/cdrom`:
> ```
> $ mount -t iso9660 /dev/cdrom /mnt/cdrom
> ```
- Sau khi mount, có thể dùng `cd` để điều hướng filesystem có mới nhất thông qua điểm kết nối vừa tạo.
- Để unmount filesystem từ hệ thống, dùng lệnh `unmount` bằng xác định điểm kết nối hoặc thiết bị.
> Ví dụ: Để gỡ bỏ `cdrom`:
> ``` linux
> $ umount /dev/cdrom
> ```
- Lệnh `mount` cho ta khả năng truy cập vào filesystem nhưng trên các hệ thống Unix hiện đại nhất, chức năng tự động kết nối thực hiện ngầm tiến trình này cho user mà không yêu cầu sự can thiệp nào.
- Các lệnh `mount` khác: [Hướng dẫn cách sử dụng lệnh mount trong Linux](https://vietnix.vn/lenh-mount-trong-linux/)

## III. Filesytem Hierachy Standard (FHS):
- Là một tiêu chuẩn được thiết lập để quy định các tổ chức các thư mục và files trong Linux và Unix-like.
- Đảm bảo rằng các files và thư mục được sắp xếp một cách nhất quán trong các hệ thống, giúp việc quản lý và tìm kiếm dữ liệu trở nên thuận lợi hơn.
- Định nghĩa mục đích của mỗi thư mục, giúp việc bảo trì hệ thống, phát triển phần mềm và chia sẻ dữ liệu giữa các bản phân phối Linux trở nên dễ dàng hơn.
- Cấu trúc cây thư mục trong Linux:
  
![image](https://hackmd.io/_uploads/BkacvkMDWx.png)
![image](https://hackmd.io/_uploads/HJV-KyMPZl.png)
- Linux dùng `/` để tách các đường dẫn (Windows dùng `\`).
  
![image](https://hackmd.io/_uploads/ByIDdyGPWx.png)
- Các thư mục được mô tả:
    - `/root` - Thư mục gốc:
        - Mọi files và thư mục đều bắt đầu từ đây.
        - Chỉ root user mới có quyền ghi trong này.
        - `/root` là thư mục home của root user, khác với `/`.
    - `/bin` - Chứa các chương trình nhị phân:
        - Chứa các file thực thi dạng nhị phân, chương trình cơ bản.
        - Các lệnh Linux thông dụng cần thiệt cho chế độ user đơn được đặt tại đây.
        - Các lệnh được tất cả user của hệ thống dùng nằm ở đây (`ps`, `ls`, `ping`, `grep`, `cp`,...).
    - `/sbin` - Chứa các chương trình nhị phân hệ thống:
        - Giống như `/bin`, `/sbin` cũng chứa các file thực thi nhị phân.
        - Các lệnh Linux ở đây thường được admin hệ thống dùng cho mục đích bảo trì hệ thống (`iptables`, `reboot`, `fdisk`, `ifconfig`, `swapon`,...).
    - `/etc`: Chứa các file cấu hình cấu hình cần thiết cho tất cả chương trình, bao gồm cả các script khởi động và tắt máy dùng để bắt đầu/dừng các chương trình đơn lẻ (`/etc/resolv.conf`, `/etc/logrotate.conf`,...).
    - `/dev`: Chứa các tập tin thiết bị, bao gồm thiết bị đầu cuối, USB hoặc bất kì thiết bị nào được kết nối với hệ thống (`/dev/tty1`, `/dev/usbmon0`,...).
    - `/proc` - Thông tin quá trình:
        - Dùng cho nhân Linux, xuất dữ liệu sang không gian người dùng.
        - Chứa thông tin về các quá trình hệ thống.
        - Là hệ thống file ảo chứa thông tin về các quá trình đang chạy (`/proc/{pid}` chứa thông tin về quá trình có pid trong đó,...).
        - Là hệ thống file ảo với thông tin dạng văn bản về nguồn lực hệ thống (`/proc/uptime`,...).
    - `/var` - Variable files: Dữ liệu biến được xử lý bởi daemon. Nội dung các file dự kiến sẽ tăng lên có thể tìm thấy dưới thư mục này. Gồm:
        - File nhật ký hệ thống (`/var/log`).
        - Packages và database file (`/var/lib`).
        - Email (`/var/mail`).
        - Hàng đợi in ấn (`/var/spool`).
        - Lock files (`/var/lock`).
        - File tạm cần qua các lần khởi động lại (`/var/tmp`).
    - `/tmp` - Chứa các file tạm thời:
        - Thư mục chứa các file tạm thời được tạo bởi hệ thống và user.
        - Các file dưới thư mục này sẽ bị xoá khi hệ thống khởi động lại.
    - `/usr` - Chương trình người dùng:
        - Chứa các file nhị phân, thư viện, tài liệu và mã nguồn cho các chương trình cấp hai để phục vụ user.
        - `/usr/bin` chứa các file nhị phân cho chương trình người dùng. Nếu không tìm thấy file nhị phân người dùng dưới `/bin` thì tìm dưới `/usr/bin` (`at`, `awk`, `cc`, `less`, `scp`,...).
        - `/usr/sbin` chứa các file nhị phân cho admin hệ thống, nếu không tìm thấy dưới `/sbin` thì tìm dưới `/usr/sbin` (`atd`, `cron`, `sshd`, `useradd`, `userdel`,...).
        - `/usr/lib` chứa thư viện cho `/usr/bin` và `/usr/sbin`.
        - `/usr/local` chứa các chương trình người dùng cài đặt từ mã nguồn (ví dụ khi cài apache từ mã nguồn, nó sẽ được đặt dưới `/usr/local/apache2`).
    - `/home`: Thư mục Home cho tất cả user khác root, dùng để lưu trữ file cá nhân (`/home/yuto`, `/home/toan`,...).
    - `/boot`:
        - Chứa nhân Linux để khởi động, các file liên quan đến trình khởi động và các file system maps cũng như các file khởi động giai đoạn hai.
        - `Kernel initrd`, `vmlinux`, `grub` được đặt dưới `/boot` (`initrd.img-2.6.32-24-generic`, `vmlinuz-2.6.32-24-generic`,...).
    - `/lib` - Thư viện hệ thống:
        - Chứa các file thư viện hỗ trợ cho các file nhị phân dưới `/bin` và `/sbin`.
        - Tên file thư viện thường là `ld*` hoặc `lib*.so.*` (`ld-2.11.1.so`, `libncurses.so.5.7`,...).
    - `/opt` - Ứng dụng bổ sung tuỳ chọn:
        - Viết tắt của `optional`.
        - Chứa các ứng dụng bổ sung từ các nhà cung cấp riêng lẻ.
        - Các ứng dụng bổ sung nên được cài đặt dưới `/opt` hoặc thư mục con của `/opt/`.
    - `/mnt`: Thư mục Mount tạm thời nơi admin hệ thống có thể mount các hệ thống file kết nối bên ngoài.
    - `/media`: Thư mục Mount tạm thời cho các thiết bị di động (`/media/cdrom` cho ổ đĩa CD-ROM, `/media/floppy` cho ổ đĩa mềm, `/media/cdrecorder` cho ổ đĩa ghi CD,...).
    - `/srv` - Dữ liệu dịch vụ:
        - Viết tắt của `service`.
        - Chứa dữ liệu liên quan đến các dịch vụ cụ thể của server được lưu trữ trên hệ thống (`/srv/cvs` chứa dữ liệu liên quan đến CVS,...).

## IV. Phân quyền trong Linux:
> Chi tiết hơn: [Giới thiệu về phân quyền trên Linux](https://123host.vn/community/tutorial/gioi-thieu-ve-phan-quyen-tren-linux.html)

### 1. Giới thiệu:
- Phân quyền trong Linux là một khía cạnh rất quan trọng giúp quản lý quyền truy cập vào các tệp và thư mục trong filesystem.
- Hệ thống phân quyền trong Linux dựa trên mô hình Unix, sử dụng các quyền cho chủ sở hữu, nhóm và user khác nhau.
- Có hai quyền cơ bản:
    - Permission (Quyền truy cập):
        - Xác định quyền truy cập của user đối với một tệp hay thư mục cụ thể.
        - Ba loại cơ bản:
            - **Read (R):** Cho phép đọc nội dung của tệp hoặc thư mục.
            - **Write (W):** Cho phép sửa đổi nội dung tệp hoặc thư mục.
            - **Execute (X):** Cho phép thực thi tệp hoặc truy cập thư mục.
        -  Ba đối tượng áp dụng:
            -  **Chủ sở hữu (Owner):** Người tạo ra tệp hoặc thư mục, có thể quyết định quyền truy cập.
            - **Nhóm (Group):** Các user thuộc vào một nhóm cụ thể có thể có các quyền riêng biệt.
            - **Khác (Others):** Tất cả user khác ngoài owner và group.
    - Ownership (Quyền sở hữu):
        - Quy định người sở hữu và nhóm sở hữu của một tệp hoặc thư mục.
        - Ba đối tượng chính: Owner (Chủ sở hữu), Group (Nhóm), và Others (Người khác).
        - Chủ sở hữu thường là người tạo ra tệp hoặc thư mục. 
- Lệnh cơ bản:
    - `chmod`: Dùng để thay đổi quyền truy cập của tệp hoặc thư mục. Cho phép chủ sở hữu có quyền đọc, ghi và thực thi, nhóm có quyền đọc và ghi, người khác có quyền đọc.
    - `ls`: Dùng option `-l` để hiển thị thông tin chi tiết về tệp và thư mục kèm theo quyền truy cập.
    - `umask`: Đặt quyền mặc định cho tệp khi tạo mới.
    - Dùng lệnh `chown` để thay đổi chủ sở hữu và `chgrp` để thay đổi nhóm sở hữu.
    > `$ chown owner:group file.txt` và `$ chown alice:developers file.txt`: Thay đổi chủ sở hữu của `file.txt` thành người dùng `alice` và nhóm `developers`.
- Phân quyền đặc biệt:
    - **SetUID (s):** Cho phép user thực thi tệp với quyền của owner.
    - **SetGID (s):** Cho phép user thực thi tệp với quyền của nhóm chủ sở hữu.
- Tại sao phải phân quyền trên Linux:
    - Bảo mật hệ thống: Người quản trị có thể kiểm soát quyền truy cập vào các tệp và thư mục, giúp ngăn chặn truy cập trái phép và bảo vệ dữ liệu quan trọng.
    - Bảo vệ dữ liệu cá nhân: User có thể bảo vệ dữ liệu cá nhân khỏi việc truy cập bởi những người khác.
    - Quản lý tài nguyên: Giúp admin hệ thống quản lý tài nguyên hiệu quả bằng cách giới hạn quyền truy cập của user.
    - Ngăn chặn thực thi nguyên mã độc hại: Ngăn user không an toàn thực thi file độc hại bằng cách giới hạn quyền thực thi.
    - Tạo môi trường làm việc an toàn: Đảm bảo mỗi user chỉ có quyền truy cập vào những file và thư mục cần để thực hiện công việc.
    - Quản lý nhóm: Cho phép admin quản lý quyền truy cập của từng nhóm, giúp tổ chức và quản lý dự án.
    - Tính linh hoạt: Cho phép tạo ra nhiều tài khản có các quyền khác nhau, tăng tính linh hoạt trong quản lý hệ thống.

### 2. Cách xem quyền:
#### a) Phân loại các File Linux:
![image](https://hackmd.io/_uploads/Bkg4WdNY-e.png)
- Trong Linux, mọi thứ đều được coi như một file và khi dùng `ls -l`, mỗi dòng sẽ hiển thị về một file hoặc thư mục. Các ký tự đầu tiên trong output biểu thị loại file hoặc thư mục.
- Một số file type phổ biến:
    - `-`: File thường.
    - `d`: Thư mục.
    - `l`: Symbolic link.
    - `p`: Đường ống (pipe).
    - `c`: Character device.
    - `b`: Block device.
    - `s`: Socket.

#### b) Đọc hiểu được các kí tự phân quyền:
![image](https://hackmd.io/_uploads/Hy8d-d4Fbl.png)
Các chữ cái đầu tiên thường chỉ loại, chín ký tự tiếp theo biểu thị quyền truy cập của owner, group và others.
- Quyền truy cập (Các ký tự 2 - 10):
    - Các ký tự 2 - 4: Đại diện cho quyền của owner.
    - Các ký tự 5 - 7: Đại diện cho quyền của group.
    - Các ký tự 8 - 10: Đại diện cho quyền của others.
    - Mỗi nhóm ba ký tự này có thể có giá trị là `r`, `w`, `x` (trong trường hợp của thư mục, quyền này biểu thị quyền vào thư mục).
- Số liên kết (đứng trước owner và group): Là số liên kết tới file hoặc thư mục.
- Owner và Group: Là tên của owner và nhóm sở hữu của file hoặc thư mục.
- Kích thước (Size): Kích thước của file trong bytes.
- Thời gian sửa đổi: Thời điểm sửa đổi lần cuối của file hoặc thư mục.
- Tên file hoặc thư mục.
- Trong Linux, có thể thay đổi quyền của một file hoặc thư mục bằng lệnh `chmod`. Có hai cách sử dụng:

##### Chế độ ký tự:
Có thể sử dụng các ký tự để biểu thị quyền cần thêm hoặc gỡ bỏ:
- `u`: Owner
- `g`: Group
- `o`: Others
- `a`: Tất cả
> Ví dụ: `$ chmod u+x file.txt`: Thêm quyền thực thi cho chủ sở hữu `file.txt`.

##### Chế độ số:
- Mỗi quyền được biểu diễn bằng một con số:
    - `r` = `4`
    - `w` = `2`
    - `x` = `1`
- Cộng các giá trị này để đại diện cho quyền cụ thể.
> `7` (`rwx`): Đọc, ghi, thực thi.
> `5` (`r-x`): Đọc, thực thi.
> `4` (`r–`): Chỉ đọc.
> Ví dụ: `$ chmod 755 file.txt`: Thiết lập quyền `rwxr-xr-x` cho `file.txt`. Tức là chủ sở hữu có quyền đọc, ghi, thực thi; nhóm và người dùng khác có quyền đọc và thực thi.

#### c) Quyền đặc biệt:
![image](https://hackmd.io/_uploads/rkrafuNF-x.png)
Ngoài quyền truy cập thông thường còn có ba quyền đặc biệt khác là SUID, SGID và Sticky Bit. Các quyền này áp dụng cho cả file và thư mục.
- SUID (Ser User ID):
    - Khi một file có quyền này, nó chạy với quyền của owner thay vì quyền của người chạy file, cho phép user thực hiện các tác vụ mà họ không có quyền thực hiện thông thường.
    - Biểu thị bằng `s` trong quyền thực thi của chủ sở hữu (owner execute permission).
- SGID (Set Group ID):
    - Khi một file có quyền này, nó chạy với quyền của nhóm sở hữu file thay vì quyền của nhóm chạy file, thường dùng trong các thư mục chia sẻ để đảm bảo mọi file được tạo trong thư mục đều thuộc cùng một nhóm.
    - Biểu thị bằng `s` trong quyền thực thi của nhóm sở hữu (group execute permission).
- Sticky Bit:
    - Khi một thư mục có quyền này, chỉ file owner có thể xoá hoặc di chuyển file trong thư mục đó, ngay cả khi người khác có quyền ghi vào thư mục. Thường dùng trong các thư mục chia sẻ như `/tmp` để ngăn việc xoá file bởi người khác.
    - Biểu thị bằng `t` trong quyền thực thi của người khác (others execute permission).

#### d) Một số lệnh xem quyền:
##### Lệnh `ls -l`:
![image](https://hackmd.io/_uploads/HJdSEuVFbx.png)
Giải thích:
- Quyền truy cập: -`rw-r–r–` (Chủ sở hữu có quyền đọc và ghi; Nhóm và Người khác chỉ có quyền đọc)
- Chủ sở hữu: `kienthuc`
- Nhóm sở hữu: `root`
- Kích thước: `4470` bytes
- Ngày tạo hoặc sửa lần cuối: `Jan 17 21:25`

##### Lệnh `stat`:
Cung cấp thông tin chi tiết về tệp hoặc thư mục, bao gồm quyền truy cập.

![image](https://hackmd.io/_uploads/BkfsEOVt-l.png)
- Tên tệp: `kiemtraphanquyen.txt`
- Kích thước: `4470` bytes
- Blocks: `16`
- IO Block: `4096`
- Loại tệp: `regular file`
- Device: `fd01h/64769d`
- Inode: `77557`
- Links: `1`
- Quyền truy cập: `(0644/-rw-r--r--)`
- Uid (User ID): `1002` (`kienthuc`)
- Gid (Group ID): `0` (`root`)
- Thời điểm quyền truy cập cuối cùng: ``2024-01-17 21:25:43.781757712 -0500``
- Thời điểm sửa đổi cuối cùng: `2024-01-17 21:25:43.781757712 -0500`
- Thời điểm thay đổi cuối cùng: `2024-01-17 21:27:37.440620302 -0500`
- Thời điểm tạo: Không có thông tin (hiển thị là `-`).

##### Lệnh `ls -lh`:
Hiển thị kích thước file dễ đọc và quyền truy cập.

![image](https://hackmd.io/_uploads/H1e7XBdNtWl.png)
- Quyền truy cập: `-rw-r--r--` (Chủ sở hữu có quyền đọc và ghi; Nhóm và Người khác chỉ có quyền đọc)
- Chủ sở hữu: `kienthuc`
- Nhóm sở hữu: `root`
- Kích thước: `4.4K` bytes
- Ngày tạo hoặc sửa lần cuối: `Jan 17 21:25`

##### Lệnh `getfacl`:
Hiển thị thông tin phân quyền chi tiết bao gồm cả quyền đặc biệt và quyền mở rộng.

![image](https://hackmd.io/_uploads/ByCBrOVYZg.png)
- `# file: kiemtraphanquyen.txt`: Hiển thị tên của tệp mà ACL áp dụng.
- `# owner: kienthuc`: Chủ sở hữu của tệp là người dùng có tên là `kienthuc`.
- `# group: root`: Nhóm sở hữu của tệp là nhóm có tên là `root`.
- `user::rw-`: Quyền truy cập cho chủ sở hữu (`kienthuc`) là đọc và ghi, nhưng không có quyền thực thi.
- `group::r--`: Quyền truy cập cho nhóm sở hữu (`root`) là chỉ đọc, không có quyền ghi hay thực thi.
- `other::r--`: Quyền truy cập cho người khác (người dùng không phải là chủ sở hữu hay nhóm sở hữu) là chỉ đọc, không có quyền ghi hay thực thi.

##### Lệnh `namei`:
Hiển thị thông tin về quyền truy cập và quyền sở hữu của tất cả các thành phần trong một đường dẫn.

![image](https://hackmd.io/_uploads/SkM2S_4t-g.png)
- `f: namnh/kiemtraphanquyen.txt`: Đường dẫn của tệp.
- `d namnh`: Thư mục cha của tệp, có tên là `namnh`.
- `- kiemtraphanquyen.txt`: Tên của tệp.

##### Lệnh `lsattr`:
Hiển thị và thay đổi thuộc tính mở rộng của tệp hoặc thư mục (nếu có).

![image](https://hackmd.io/_uploads/BJUeIONYWg.png)
Lệnh `lsattr kiemtraphanquyen.txt `được sử dụng để hiển thị các thuộc tính của tệp hoặc thư mục trong hệ thống file ext2/ext3/ext4. Trong trường hợp trên, kết quả là `-------------e--`:
- `-------------`: Đây là phần của thuộc tính mà không có bất kỳ thuộc tính nào được thiết lập. Các dấu gạch ngang đại diện cho không có thuộc tính nào được thiết lập.
- `e`: Đây có thể là một trong những ký tự mở rộng của thuộc tính, tuy nhiên, để biết ý nghĩa chính xác của nó, cần tham khảo tài liệu hệ thống file cụ thể hoặc sử dụng lệnh `man chattr` để xem tất cả các ký tự mở rộng có thể xuất hiện.

## V. Log files trong Linux:
### 1. Giới thiệu về Log file:
- Là tập hợp các bản ghi mà Linux duy trì để admin theo dõi các event quan trọng.
- Chứa các thông báo về kernel, dịch vụ và ứng dụng đang chạy trên server.
- Cung cấp thời gian của các event cho hệ điều hành, ứng dụng và hệ thống Linux.
- Cung cấp kho lưu trữ tập trung các log file trong thư mục `/var/log`.
- Hầu hết được chia thành bốn loại:
    - Application Logs: Nhật ký ứng dụng
    - Event Logs: Nhật ký sự kiện
    - Service Logs: Nhật ký dịch vụ
    - System Logs: Nhật ký hệ thống
- Là công cụ quan trọng giúp khắc phục sự cố; nắm rõ hơn về hiệu suất server, bảo mật, thông báo lỗi và các vấn đề tiềm ẩn; cho phép dự đoán các vấn đề sắp tới.

### 2. Các log files quan trọng:
#### a) System Logs:
##### `/var/log/messages` hoặc `/var/log/syslog`:
`[root@localhost ~]# cat /var/log/messages`
Hoặc
`root@ubuntuserver:~# cat /var/log/syslog`
- Chứa nhật ký hoạt động hệ thống (System Logs). Chủ yếu dùng để lưu trữ các thông tin liên quan đến hệ thống như `mail`, `cron`, `daemon`, `kern`, `auth`,...
- Có thể theo dõi các lỗi khởi động trừ Kernel, lỗi dịch vụ liên quan đến ứng dụng và các thông báo được ghi lại trong quá trình khởi động hệ thống.
- Là log file đầu tiên mà admin nên kiểm tra khi có sự cố trên hệ thống.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <AppName>[PID]: <Message>`
> - Lưu ý: Các bản phân phối Linux thuộc họ Redhat (như CentOS hoặc RHEL) được lưu trữ trong `/var/log/message`, trong khi Ubuntu và các hệ thống dựa trên Debian khác được lưu trữ trong `/var/log/syslog`.
> - Ví dụ:
> ![image](https://hackmd.io/_uploads/BJezehBF-g.png)
> ![image](https://hackmd.io/_uploads/SJ32tfdKZx.png)

##### `/var/log/kern.log`:
`[root@localhost ~]# cat /var/log/kern.log`
- Là log file quan trọng chứa các thông tin được ghi bởi Kernel.
- Giúp khắc phục các lỗi và cảnh báo liên quan đến Kernel và phần cứng.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> kernel: [<Uptime>] <Message>`
> ![image](https://hackmd.io/_uploads/HJpMHnSYWe.png)
> ![image](https://hackmd.io/_uploads/r1dLTf_FWg.png)

##### `/var/log/dmesg`:
`[root@localhost ~]# cat /var/log/dmesg`
- Ghi nhận các thông tin về Kernel.
- Khi hệ thống khởi động các thông tin liên quan đến các thiết bị phần cứng, trình điều khiển của chúng được ghi lại. Vì Kernel phát hiện các thiết bị phần cứng vật lý được liên kết trong quá trình khởi động, nó ghi lại trạng thái thiết bị, lỗi phần cứng và các thông báo chung khác.
- Nếu một thiết bị phần cứng nào đó hoạt động không đúng hoặc không được phát hiện, có thể dựa vào file này để khắc phục sự cố.
- Cú pháp tổng quát: `[<Uptime>] <Subsystem/Module>: <Message>`
> ![image](https://hackmd.io/_uploads/rJ3lVnrF-l.png)
> ![image](https://hackmd.io/_uploads/Hkin6GOt-l.png)

##### `/var/log/daemon.log`:
- Chứa thông tin được ghi lại bởi các tiến trình nền khác nhau chạy trên hệ thống.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <DaemonName>[PID]: <Message>`

##### `/var/log/faillog`:
`[root@localhost ~]# cat /var/log/faillog`
- Chứa các thông tin user đã đăng nhập thất bại.
- Có thể dùng lệnh `faillog` để hiển thị nội dung của file.
- Là log file hữu ích để tìm ra các vi phạm bảo mật liên quan đến việc hack username hoặc password của user cũng như các cuộc tấn công.
- Cú pháp tổng quát:
``` linux
Login       Failures  Maximum  Latest                   Line
<Username>  <Count>   <Limit>  <Timestamp>              <TTY/Terminal>
```

##### `/var/log/lastlog`:
- Hiển thị thông tin đăng nhập gần đây cho tất cả user. Ta nên sử dụng lệnh `lastlog` để xem nội dung của file này.
- Cú pháp tổng quát:
``` linux
Username        Port     From             Latest
<Username>      <Line>   <IP/Host>        <Timestamp>
```
> ![image](https://hackmd.io/_uploads/r1Kv-TSYbe.png)

##### `/var/log/boot.log`:
`[root@localhost ~]# cat /var/log/boot.log`
- Lưu trữ tất cả thông tin liên quan đến khởi động và mọi thông báo được ghi lại trong quá trình khởi động gồm tập lệnh hệ thống, `/etc/init.d/bootmisc.sh`,...
- Phân tích để kiểm tra các vấn đề liên quan đến tắt máy không đúng cách, khởi động lại hoặc lỗi khởi động.
- Hữu ích để xác định thời gian ngừng hoạt động của hệ thống do tắt máy đột xuất.
- Cú pháp tổng quát:
``` linux
[  OK  ] Started <ServiceName>.
[  OK  ] Reached target <TargetName>.
[FAILED] Failed to start <ServiceName>.
```

##### `/var/log/auth.log`:
`[root@localhost ~]# cat /var/log/auth.log`
- Chứa thông tin xác thực trên hệ thống trong máy chủ Debian và Ubuntu được ghi lại.
- Dùng để tìm kiếm vấn đề liên quan đến cơ chế uỷ quyền của user.
- Thông qua log file này có thể xác định:
    - Các lần thử đăng nhập thất bại.
    - Điều tra các cuộc tấn công và các lỗ hổng liên quan đến cơ chế uỷ quyền của user.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <AppName>[PID]: <EventMessage>`
> ![image](https://hackmd.io/_uploads/SkV3-3rKbx.png)
> ![image](https://hackmd.io/_uploads/SJmvRzutbe.png)

##### `/var/log/cron`:
`[root@localhost ~]# cat /var/log/cron`
- Lưu trữ tất cả thông tin liên quan đến Crond (công việc `cron`), ví dụ như khi tiến trình nền `cron` khởi tạo một công việc, các thông báo lỗi liên quan,...
- Bất cứ khi nào một công việc `cron` chạy, log file này ghi lại tất cả thông tin liên quan gồm thực thi thành công và thông báo lỗi trong trường hợp thất bại.
- Nếu gặp vấn đề với `cron` theo lịch trình, cần kiểm tra file này.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> CRON[PID]: (<User>) CMD (<Command>)`

##### `/var/log/debug`:
- Chứa các messages chi tiết liên quan tới việc debug và giúp khắc phục sự cố các hoạt động hệ thống cụ thể.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <AppName>[PID]: <DebugMessage>`

##### `/var/log/yum.log`:
`[root@localhost ~]# cat /var/log/yum.log`
- Chứa thông tin được ghi lại khi gói được cài bằng `yum`.
- Giúp theo dõi việc cài đặt các thành phần hệ thống và gói phần mềm. Kiểm tra các thông tin được ghi lại ở đây để xem một gói đã được cài đặt chính xác chưa.
- Giúp khắc phục sự cố liên quan đến cài đặt phần mềm.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Action> <PackageName>-<Version>-<Release>.<Architecture>`
> Ví dụ: Server hoạt động bất thường và ta nghi ngờ gói phần mềm được cài đặt gần đây là nguyên nhân cho vấn đề này. Trong trường hợp này, cần kiểm tra log file này để tìm ra gói đã được cài gần đây và xác định chương trình lỗi.

#### b) Application Logs:
- Chứa thông tin liên quan đến app thực thi, gồm messages lỗi, chi tiết vận hành và dấu hiệu cho thấy hệ thống bị xâm phạm.
- Các log files thuộc loại này:
##### CUPS Print System logs:
Ghi lại chi tiết hoạt động liên quan đến máy in.

##### Rootkit Hunter log:
Tạo bởi Rootkit Hunter, dùng để quét tìm các [rootkits](https://phoenixnap.com/glossary/rootkit) tiềm ẩn và các lỗ hổng bảo mật khác.

##### Apache HTTP server logs:
- [Apache HTTP server logs](https://phoenixnap.com/kb/apache-access-log) là logs của [Apache](https://phoenixnap.com/glossary/what-is-apache) web server.
- Gồm access logs (ghi lại cilent requests) và error logs (bắt server errors và issues).
> **Ví dụ của một Apache Access Log:**
> ``` linux
> 127.0.0.1 - - [09/Feb/2024:15:36:14 +0100] "GET / HTTP/1.1" 200 3460 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0"
> ```
> - `127.0.0.1`: Địa chỉ IP của client.
> - `- -`: Phần giữ chỗ cho remote user xác thực, nếu có. Ví dụ này không có thông tin cụ thể nào, và các dấu gạch ngang chỉ là phần giữ chỗ.
> - `[09/Feb/2024:15:36:14 +0100]`: Dấu thời gian và timezone.
> - `"GET / HTTP/1.1"`: Phương thức request (`GET`), URL (`/`), và version của giao thức HTTP (`HTTP/1.1`).
> - `200`: Mã HTTP status mà server trả về (`200` nghĩa là request đã thành công).
> - `3460`:  Response size theo bytes, nghĩa là server gửi lại 3460 bytes cho client.
> - `"-"`: Trường tham chiếu chứa web đưa client đến requested URL. Giá trị trong ví dụ không khả dụng.
> - `"Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0"`: Chuỗi user-agent được gửi từ trình duyệt web của client. Chuỗi này chứa version của trình duyệt (`Firefox/120.0`) và hệ thống vận hành (`Ubuntu Linux`).
> 
> **Ví dụ của một Apache Error Log:**
> ``` linux
> [Fri Feb 09 15:35:24.252107 2024] [core:notice] [pid 6672:tid 139657266624384] AH00094: Command line: '/usr/sbin/apache2'
> ```
> - `[Fri Feb 09 15:35:24.252107 2024]`: Dấu thời gian khi bản ghi được add vào error log file. Format của dấu thời gian là `[Weekday Month Day Hour:Minute:Second.Microsecond Year]`.
> - `[core:notice]`: Thành phần tạo nên log entry (`core`) và mức độ nghiêm trọng (`notice`).
> - `[pid 6672:tid 139657266624384]`: [Process ID](https://phoenixnap.com/glossary/process-id) (`pid 6672`) và Thread ID (`tid 139657266624384`) liên quan đến entry.
> - `AH00094: Command line: '/usr/sbin/apache2'`: Error log message, ví dụ này cho thấy Apache đang hiển thị thông tin command-line của nó.

##### Samba SMB server logs:
Là logs của Samba server, cung cấp file và dịch vụ in cho SMB/CIFS clients và bắt các chi tiết về truy nhập và errors.

##### The X11 server log:
- Log liên quan tới [X11 Windows System](https://phoenixnap.com/glossary/what-is-x11).
- Bắt các chi tiết về vận hành và errors của máy chủ đồ hoạ.

#### c) Service Logs:
- Chứa thông tin liên quan đến dịch vụ hệ thống và các tiến trình nền, giúp quản lý sự vận hành của các dịch vụ này.
- Một số system logs như systemd service logs (`/var/log/syslog`, `/var/log/messages`), cron job logs (/`var/log/cron`) và Samba server logs (`/var/log/samba/log.smbd`)

#### d) Event Logs:
- Chứa các thông tin về system events và actions cụ thể, thường liên quan đến bảo mật, xác thực và hoạt động user.
- Cung cấp các records chi tiết của các nỗ lực đăng nhập, những thay đổi hệ thống và sự cố quan trọng xảy ra, giúp admin quản lý và kiểm tra các hoạt động hệ thống.
- Các event logs bao gồm audit logs (`/var/log/audit/audit.log`) và system logs như authentication logs (`/var/log/auth.log`) và Kernel logs (`/var/log/kern.log`).

#### Một số log files khác:
##### `/var/log/secure`:
`[root@localhost ~]# cat /var/log/secure`
- Với các hệ thống dùng RedHat và CentOS thì log file này thay thế cho `/var/log/auth.log`.
- Chứa các thông tin về xác thực trên hệ thống.
- Có thể lưu trữ tất cả thông tin liên quan đến bảo mật, lỗi xác thực.
- Giúp theo dõi thông tin đăng nhập `sudo`, SSH và các lỗi khác được ghi bởi tiến trình chạy nền của dịch vụ bảo mật hệ thống.
- Giúp thấy được chi tiết về các lần đăng nhập trái phép hoặc thất bại và nó cũng lưu trữ thông tin đăng nhập thành công và theo dõi các hoạt động của user hợp lệ.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <AppName>[PID]: <SecurityMessage>`

##### `/var/log/maillog` hoặc `/var/log/mail.log`:
`[root@localhost ~]# cat /var/log//maillog`
Hoặc
`[root@localhost ~]# cat /var/log/mail.log`
- Lưu trữ các thông tin từ server mail đang chạy trên hệ thống gồm các thông tin về `postfix`, `smtpd`, `MailScanner`, `SpamAssassain` hoặc bất kì dịch vụ liên quan đến email nào khác.
- Ghi lại tất cả email được gửi hoặc nhận trong khoảng thời gian cụ thể.
- Giúp kiểm tra các vấn đề gửi thư thất bại, nhận thông tin về spam có thể bị chặn bởi máy chủ mail.
- Theo dõi nguồn gốc của một email đến bằng cách xem kĩ log `maillog` hoặc `mail.log`.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <MailService>[PID]: <QueueID>: <MessageDetails>`

##### `/var/log/httpd`:
`[root@localhost ~]# cat /var/log/httpd`
- Lưu trữ các file `error_log` và `access_log` của tiến trình nền `httpd` Apache.
    - `error_log` chứa tất cả lỗi gặp phải `httpd`. Những lỗi này gồm các vấn đề về bộ nhớ và các lỗi liên quan đến hệ thống khác.
    - `access_log` chứa bản gh của tất cả request nhận được qua HTTP.
- Giúp theo dõi mọi trang được phục vụ và mọi file được tải bởi Apache.
- Ghi lại địa chỉ IP và ID người dùng của tất của máy khách thực hiện yêu cầu kết nối đến server.
- Lưu trữ thông tin về trạng thái các request truy cập cho dù phản hồi đã được gửi thành công hay dẫn đến lỗi.
- Khi gặp vấn đề với máy chủ web Apache, kiểm tra file này để biết thông tin chẩn đoán.
- Cú pháp tổng quát: `<ClientIP> - <User> [<Timestamp>] "<Method> <Path> <Protocol>" <Status> <Size>`

##### `/var/log/mysqld.log` hoặc `/var/log/mysql.log`:
- File log `MySQL` ghi lại tất cả thông báo gỡ lỗi, thất bại và thành công.
- Chứa thông tin về việc bắt đầu, dừng và khởi động lại `MySQL deamon mysqld`.
- Dùng file này để xác định các vấn đề trong khi bắt đầu, chạy hoặc dừng `mysqld`; nhận thông tin về các kết nối máy khác đến thư mục dữ liệu MySQL.
- Ta cũng có thể thiết lập tham số `long_query_time` để ghi thông tin về khoá truy vấn và truy vấn chạy chậm.
- Cú pháp tổng quát: `<Timestamp> <ThreadID> <Command> <Argument>`
> Lưu ý: Hệ thống RedHat, CentOS, Fedora và các hệ thống dựa trên RedHat khác dùng `/var/log/mysqld.log` trong khi Debian/Ubuntu dùng `var/log/mysql.log`.

##### `/var/log/dpkg.log`:
- Chứa thông tin được ghi lại khi gói được cài đặt hoặc gỡ bỏ bằng lệnh `dpkg`.
- Cú pháp tổng quát: `YYYY-MM-DD HH:MM:SS <Action> <PackageName> <InstalledVersion> <TargetVersion>`
> ![image](https://hackmd.io/_uploads/Sk0UgarK-e.png)
> ![image](https://hackmd.io/_uploads/Hy0KJQOY-e.png)

##### `/var/log/user.log`:
- Chứa thông tin về tất cả nhật ký cấp độ người dùng.
- Cú pháp tổng quát: `MMM DD HH:MM:SS <Hostname> <AppName>[PID]: <Message>`

##### `/var/log/alternigin.log`:
- Thông tin của các lựa chọn thay thế cập nhật được đăng nhập vào tệp nhật ký này.
- Cú pháp tổng quát: `update-alternatives <Timestamp>: <Action> <LinkGroup> <Path>`
> ![image](https://hackmd.io/_uploads/BkZAkX_KWx.png)

##### `/var/log/btmp`:
- Chứa thông tin về các thông tin đăng nhập thất bại. Sử dụng lệnh `last` để xem file `btmp`.
- Cú pháp tổng quát: `<Username>  <Tty/Service>  <Source_IP>  <Timestamp> - <End_Status> (<Duration>)`
> ![image](https://hackmd.io/_uploads/SJ9cbaBtZx.png)

##### `/var/log/anaconda.log`:
- Khi cài đặt Linux, tất cả các thông báo liên quan đến cài đặt được lưu trữ tại đây.
- Cú pháp tổng quát: `HH:MM:SS,ms <Level> <Module>: <Message>`

##### `/var/log/safe`:
- Chứa thông tin liên quan đến xác thực và đặc quyền ủy quyền.
- Cú pháp tổng quát: `HH:MM:SS,ms <Level> <Module>: <Message>`
> Ví dụ: `sshd` ghi lại tất cả các thông tin ở đây, bao gồm cả đăng nhập không thành công.

##### `/var/log/wtmp` hoặc `/var/log/utmp`:
Chứa các bản ghi đăng nhập. 
- Sử dụng `wtmp` có thể tìm ra ai đã đăng nhập vào hệ thống.
- Lệnh `who` sử dụng file này để hiển thị thông tin.
- Cú pháp tổng quát: `<Username>  <Terminal>     <Timestamp>           (<Source_IP/Host>)`
> ![image](https://hackmd.io/_uploads/r1dmlXuKbe.png)

### 3. Cách xem Log file:
- Để xem nội dung của `/var/log`:
    - Cho phép quyền root bằng cách dùng [`sudo` command](https://phoenixnap.com/kb/linux-sudo) hay đổi sang root bằng [`su`](https://phoenixnap.com/kb/su-command-linux-examples).
    - Chạy `cd` command để đi tới `/var/log` rồi dùng `ls`.
    > ![image](https://hackmd.io/_uploads/BJmTDpIF-x.png)
    > ![image](https://hackmd.io/_uploads/BkqZYG_K-x.png)
- Cách đọc Linux Logs: Linux logs thường theo dõi pattern có cấu trúc để đảm bảo tính toàn vẹn. Phân tích các thành phần tiêu chuẩn của log entry format:
    - Timestamp: Hiển thị thời gian event xảy ra.
    - Hostname: Tên của hệ thống nơi log entry được tạo.
    - Dịch vụ hay tên app tạo log entry.
    - Process ID (PID): Định danh quá trình nào tạo log entry.
    - Log Level: Mức độ nghiêm trọng của log entry (Ví dụ: `INFO`, `WARNING`, `ERROR`).
    - Message: Log message thực tế mô tả chi tiết event.
- Một số command phổ biến:

#### `cat`:
- [`cat` command](https://phoenixnap.com/kb/linux-cat-command) hiển thị toàn bộ nội dung của một file.
> `sudo cat /var/log/syslog`
> 
> ![image](https://hackmd.io/_uploads/Hy0CtM_tbe.png)
- Output gồm timestamp, hostname, process name, PID và một message.

#### `less`:
- [`less` command](https://phoenixnap.com/kb/less-command-in-linux) trong Linux cho phép hiển thị nội dung của log files từng màn hình một.
- Cho phép đi đến các file lớn một cách dễ dàng mà không cần tải toàn bộ file trong bộ nhớ.
- Command này cũng hỗ trợ chuyển trang bằng cách cuộn, search và các command điều hướng khác, lý tưởng cho việc kiểm tra chi tiết log files một cách hiệu quả.
> `sudo less /var/log/syslog`
> 
> ![image](https://hackmd.io/_uploads/rk985GutZx.png)
- Để điều hướng output, dùng các phím:
    - Phím Mũi Tên Xuống hoặc phím `j` để xuống một dòng.
    - Phím Mũi Tên Lên hoặc phím `k` để lên một dòng.
    - Phím `Space` hoặc phím Page Down để xuống một trang.
    - Phím Page Up hoặc phím `b` để lên một trang.
    - Phím `G` để đến cuối tập tin.
    - Phím `g` để đến đầu tập tin.

#### `more`:
- Dùng để hiển thị nội dung của log files từng trang một, tương tự như `less` như giới hạn khả năng điều hướng.
- Cho phép cuộn xuống từng dòng hay từng trang, hữu dụng cho việc hiển thị và phân tích nhanh log files quá lớn để hiển thị vừa vặn trên một màn hình.
> ``` linux
> sudo <span style="background-color: initial; font-family: inherit; font-size: inherit; color: initial;">more /var/log/syslog</span>
> ```
> 
> ![image](https://hackmd.io/_uploads/HyKtczOY-l.png)
- Không giống `less`, command `more` không hỗ trợ điều hước lùi và thiếu chức năng tìm kiếm và phân tích nâng cao. Để điều hướng output:
    - Nhấn phím `Spacebar` để chuyển tiếp một trang.
    - Nhấn `Enter` để chuyển tiếp một dòng.
    - Nhấn phím `b` để chuyển lùi một trang màn hình.
    - Nhấn phím `q` để thoát khỏi màn hình lệnh More.

#### `tail`:
- [`tail` command](https://phoenixnap.com/kb/linux-tail) hiển thị mười mục cuối cùng của một file, bao gồm log files.
- Tiện cho việc quản lý log files trong thời gian thực hay kiểm tram nhanh chóng các mục hiện tại.
> `sudo tail /var/log/syslog`
> 
> ![image](https://hackmd.io/_uploads/SJ32tfdKZx.png)

#### `head`:
- [`head` command](https://phoenixnap.com/kb/linux-head) in ra phần đầu của file. Nó hữu dụng cho việc kiểm tra nhanh các mục đầu tiên của log files.
> `sudo head /var/log/syslog`
> 
> ![image](https://hackmd.io/_uploads/BkQ39z_KZl.png)

#### `grep`:
- [`grep` command](https://phoenixnap.com/kb/grep-command-linux-unix-examples) tìm kiếm các mẫu cụ thể hoặc text trong một file.
- Khi dùng với log files, `grep` nhận diện các mục chứa keywords hoặc pattern cụ thể, khiến nó hữu ích trong việc phân lập thông tin liên quan trong quá trình khắc phục sự cố hay phân tích.
> `sudo grep "error" /var/log/syslog`
> 
> ![image](https://hackmd.io/_uploads/SkexiMut-x.png)

#### `awk`:
- [`awk`](https://phoenixnap.com/kb/awk-command-in-linux) là một tool linh hoạt để đọc và xử lý log files. 
- Cho phép user xuất thông tin cụ thể, lọc dựa trên điều kiện và thực hiện vận hành hiệu quả trên log data.
> Ví dụ: Để scan `/var/log/syslog` tìm các dòng chứa từ `error` và in ra các cột của trường đầu tiên, thứ hai, thứ và thứ năm chứa dòng phù hợp:
> ``` linux
> sudo awk '/error/ {print $1, $2, $3, $5}' /var/log/syslog
> ```
> 
> ![image](https://hackmd.io/_uploads/Sk17szuKWe.png)

#### `sed`:
[`sed` command](https://phoenixnap.com/kb/linux-sed) là trình chỉnh sửa luồng xử lý và điều chỉnh texts trong log files.
> Ví dụ: Để tìm các dòng chứa từ `error` và in những dòng đó từ `/var/log/syslog`:
> ``` linux
> sudo sed -n '/error/p' /var/log/syslog
> ```
> 
> ![image](https://hackmd.io/_uploads/BydIoMOKZx.png)

#### `dmesg`:
- [`dmesg` command](https://phoenixnap.com/kb/dmesg-linux) trong Linux cho phép hiển thị messages bộ đệm vòng kernel là các system logs liên quan đến events phần cứng và kernel.
- Các logs này có giá trị trong việc chẩn đoán vấn đề phần cứng, driver và tiến trình khởi tạo hệ thống.
- Có thể truy cập các logs này trực tiếp từ kernel, cung cấp cái nhìn low-level của các hoạt động hệ thống và events bằng `dmesg`.
- Để xem log files bằng cách dùng `dmesg`, chạy command mà không cần bất kỳ option nào: `sudo dmesg`.
> ![image](https://hackmd.io/_uploads/B1qKszutbl.png)

#### `journalctl`:
- Dùng [`journalctl`](https://phoenixnap.com/kb/journalctl-systemd-logs) để hiển thị logs được gom bởi systemd daemon.
- Nó cũng lọc logs với dịch vụ, thời gian và tiêu chuẩn khác.
> Ví dụ: 
> - Để xem logs với một dịch vụ cụ thể, trong trường hợp `apache2`, chạy:
> ``` linux
> sudo journalctl -u apache2.service
> ```
> - Để xem logs từ một thời gian cụ thể:
> ``` linux
> sudo journalctl --since "2024-06-19"
> ```
> 
> ![image](https://hackmd.io/_uploads/SkSxhGuYWg.png)

#### Dùng GUI Tools:
- Các [GUI](https://phoenixnap.com/glossary/what-is-gui) log viewer tools hiển thị giao diện trực quan, lọc các options và tìm kiếm các khả năng giúp ta đọc và phân tích hiệu quả Linux logs mà không phụ thuộc duy nhất vào command-line tools.
- **System Log Viewer** là một GUI tool dùng để giám sát system logs. Giao diện thân thiện với user, cung cấp các chức năng quản lý log khác nhau, bao gồm cả hiển thị thống kê log. Các chức năng hữu ích:
    - Cái nhìn trực tiếp các logs.
    - Số lượng dòng trong log.
    - Kích thước log.
    - Thời gian ghi log gần nhất.
    - Những điều chỉnh log.
    - Bộ lọc.
    - Phím tắt Keyboards.
> ![image](https://hackmd.io/_uploads/BkVrw-wKWx.png)

### 4. Cách cấu hình Linux Logs:
- Cấu hình Linux logs gồm việc cài đặt và điều chỉnh cấu hình cho logging daemons hệ thống.
- Linux dùng `rsylog` như một logging daemon mặc định.
- Các bước cấu hình Linux logs:
    - [File cấu hình](https://phoenixnap.com/glossary/config-file) chính cho `rsylog` là `/etc/rsylog.conf`. Mở file này trong [text editor](https://phoenixnap.com/kb/best-linux-text-editors-for-coding) như Nano: `sudo nano /etc/rsyslog.conf`
    - Trong `rsyslog.conf` file thiết lập các chỉ thị toàn cầu như file size, các khoảng luân phiên và các tham số khác.
    > Ví dụ:
    > https://phoenixnap.com/kb/wp-content/uploads/2024/06/set-global-directives.png
    > ``` linux
    > #Set the default log file owner
    > $FileOwner syslog
    > $FileGroup adm
    > #Set the default log file permissions
    > $FileCreateMode 0640
    > #Set the default directory permissions
    > $DirCreateMode 0755
    > #Set the maximum log file size (e.g., 10MB)
    > $MaxMessageSize 10k
    > ```
    - Định nghĩa log rules đến để chỉ rõ làm thế nào logs được xử lý.
    > Ví dụ: Để log lại tất cả messages xác thực đến một file cụ thể:
    > 
    > ![image](https://hackmd.io/_uploads/By57YbPFWx.png)
    > ``` linux
    > # Log all authentication messages 
    > auth.* /var/log/auth.log
    > ```
    - Sau khi thực hiện thay đổi đến file cấu hình, khởi động lại `rsylog` để apply bằng command `sudo systemctl restart rsyslog` (không có output).

### 3. Custom Log Messages:
Để tạo và quản lý custom log messages trong hệ thống Linux, dùng `logger` - một command-line tool cung cấp giao diện cho system log, `rsyslog`. Command `logger` cho phép add custom log messages vào system log.
> Ví dụ:
> - Để log một message đơn giản, dùng command `logger "This is a custom log message."`. Command này không có ouput nhưng nó add `"This is a custom log message."` vào system log, ta có thể tìm trong `/var/log/syslog`.
> - Có thể add một tag đến log message để việc nhận diện dễ hơn bằng command:
> ``` linux
> logger -t myscript "This is a custom log message from my script."
> ```
> Command không có output nhưng nó tags log entry với `myscript`.
- Một option khác để cấu hình `rsyslog` là xử lý custom log message và đưa trực tiếp nó đến một log file cụ thể. Để thực hiện, tạo một file cấu hình trong thư mục `/etc/rsyslog.d/`.
> Ví dụ:
> - Chạy command `sudo nano /etc/rsyslog.d/custom-log.conf`.
> - Add cấu hình sau tới messages trực tiếp từ tiện ích `local0` tới một custom log file: `local0.* /var/log/custom.log`
>   
> ![image](https://hackmd.io/_uploads/SkHHR-Dt-x.png)
> - Đảm bảo custom logfile tồn tại và có quyền phù hợp bằng cách dùng command [`touch`](https://phoenixnap.com/kb/touch-command-in-linux), [`chown`](https://phoenixnap.com/kb/linux-chown-command-with-examples) và [`chmod`](https://phoenixnap.com/kb/chmod-recursive) (không có output):
> ``` linux
> sudo touch /var/log/custom.log 
> sudo chown syslog:adm /var/log/custom.log 
> sudo chmod 640 /var/log/custom.log
> ```
> - Khởi động lại dịch vụ `rsyslog` để apply các thay đổi: `sudo systemctl restart rsyslog`

### 4. Log Rotation:
- Là quá trình quản lý log files để ngăn chúng chiếm quá nhiều không gian disk và khiến việc quản lý log file dễ hơn.
- Gồm việc di chuyển hay lưu trữ các log files và tạo một cái mới để thay thế chúng.
- Giúp đảm bảo việc logs không phát triển vô hạn, thứ gây ra các vấn đề về hiệu suất hệ thống và quản lý logs khó khăn.
- Quản lý bằng cách dùng tool như `logrotate`.
- Để rotate một log file:
    - Tạo hoặc edit một file trong `/etc/logrotate.d` cho custom log rotation. Ví dụ: `sudo nano /etc/logrotate.d/customlog`
    - Trong phần edit, add cấu hình sau vào:
      
    ![image](https://hackmd.io/_uploads/SkhQxGvF-l.png)
    ``` linux
    /var/log/lognamehere.log {
        missingok
        notifempty
        compress
        size 20k
        daily
        create 0600 root root
    }
    ```
    Command thực hiện các hành động:
        - `missingok`: Bảo `logrotate` không output lỗi nếu một log file đang bỏ qua.
        - `notifempty`: Không rotate log file nếu nó trống.
        - `size`: Đảm bảo rằng log file không vượt quá kích thước chỉ định và rotates nó nếu vượt quá.
        - `daily`: Rotates log files với lịch trình hàng ngày, việc này cũng có thể hoàn thành theo tuần hoặc tháng.
        - `create`: Tượng trưng cho nơi một log file có owner và group là root user.
    - Lưu và thoát file.
    - Chạy `logrotate` trong debug mode để test cấu hính mà không thực hiện bất kỳ thay đổi nào: `sudo logrotate -d /etc/logrotate.conf`
      
    ![image](https://hackmd.io/_uploads/B1evbfPKZx.png)
    - Để apply log rotation ngay lập tức, dùng command `sudo logrotate -f /etc/logrotate.conf` (không output).

### 5. Tổng hợp Linux Log:
- Là việc thu thập và tập trung log data từ các sources khác nhau vào một vị trí duy nhất để quản lý, phân tích và giám sát dễ hơn.
- Giúp khắc phục sự cố, nhận diện pattern, đảm bảo bảo mật và duy trì sự tuân thủ bằng cách cung cấp cái nhìn toàn vẹn về sự vận hành hệ thống.
- Các tools để tổng hợp log:
    - [ELK Stack](https://phoenixnap.com/kb/elk-stack) (Elasticsearch, Logstash, Kibana):
        - [Elasticsearch](https://phoenixnap.com/kb/install-elasticsearch-ubuntu): Một công cụ tìm kiếm và phân tích lưu trữ và lập chỉ mục log data.
        - Logstash: Quy trình xử lý data thu thập từ nhiều nguồn, biến đổi và gửi data đến Elasticsearch.
        - Công cụ trực quan hóa cho phép khám phá và thấy data được lưu trữ trong Elasticsearch.
    - Graylog:
        - Công cụ quản lý open-source log cung cấp khả năng tìm kiếm, phân tích và cảnh báo theo thời gian thực.
        - Tổng hợp log từ nhiều nguồn khác nhau và cung cấp giao diện dựa trên web để giám sát và phân tích.
    - Fluentd:
        - Công cụ thu thập data open-source giúp thống nhất quy trình thu thập và sử dụng data.
        - Hỗ trợ nhiều plugin cho việc nhập và xuất data, linh hoạt trong các trường hợp khác nhau.
    - [Splunk](https://phoenixnap.com/kb/elk-stack-vs-splunk):
        - Nền tảng toàn diện để tìm kiếm, giám sát và phân tích data do máy tạo ra.
        - Cung cấp khả năng trực quan hóa và cảnh báo mạnh mẽ và được dùng rộng rãi trong doanh nghiệp.
    - Prometheus: Chủ yếu được sử dụng để giám sát và cảnh báo, nhưng cũng thu thập và truy vấn log. Prometheus tích hợp tốt với Grafana cho mục đích trực quan hóa.

## VI. `auditd`:
### 1. Khái niệm:
- Là tool chạy ngầm phổ biến để audit hệ thống Linux, nghĩa là phân tích sâu về hệ thống với một mục tiêu cụ thể, hầu hết được tích hợp mặc định trên Linux.
- Một quy trình audit thực hiện bằng việc kiểm tra các vùng khác nhau của hệ thống đó, với đánh giá các vùng khác nhau của hệ thống.
- `auditd` là phương tiện theo dõi các thông tin liên quan đến an ninh hệ thống, sử dựng các quy tắc được cấu hình từ trước để thu thập lượng lớn thông tin về các events đang xảy ra và ghi lại chúng dưới dạng log:
    - Timestamp
    - Kiểu event, kết quả event.
    - User gây ra event.
    - Sửa đổi thực hiện trên file/database.
    - Event dùng các cơ chế xác thực hệ thống (PAM, LDAP, SSH,...).
    - Thay đổi thực hiện trên audit config file.
    - Nỗ lực nhập xuất thông tin từ hệ thống.
    - ...

### 2. Tại sao hệ thống audit trên Linux lại quan trọng?
- Tự chủ do không phải chạy bất cứ chương trình hay tiến trình nào từ bên ngoài hệ thống.
- Cho phép xem bất kỳ hoạt động nào của hệ thống.
- Hỗ trợ việc phát hiện hoặc phân tích những nguy cơ tiềm ẩn của một hệ thống.
- Có khả năng hoạt động như một hệ thống IDS.
- Có thể làm việc với hệ thống IDS để phát hiện xâm nhập.
- Là công cụ quan trọng trong forensics.

### 3. Các thành phần trong hệ thống Linux audit:
- Có hai thành phần chính:
    - User-space applications và các tiện ích/công cụ.
    - Hệ thống xử lý ở kernel-side: Chấp nhận các yêu cầu từ các user-space applications và chuyển chúng qua ba loại bộ lọc: `user`, `task`, `exit`, `exclude`
- Phần quan trọng nhất là `user-space audit daemon` (`auditd`) thu thập thông tin dựa trên các quy tắc được cấu hình từ trước, từ kernel và tạo ra các thông tin trong audit log (đường dẫn mặc định là `/var/log/audit/audit.log`).
- Ngoài ra, `audispd` (một daemon gửi đi các audit) là một multiplexor tương tác với `auditd` và gửi event đến các chương trình khác mà muốn thực hiện xử lý event theo thời gian thực.
- Một số công cụ mà user-space dùng cho việc quản lý và truy xuất thông tin từ hệ thống audit:
    - `auditctl`: Một tiện ích kiểm soát hệ thống kernel audit.
    - `ausearch`: Một tiện ích cho việc tìm kiếm các audit log với các event đặc biệt.
    - `aureport`: Một tiện ích để tạo các báo cáo về event được ghi lại.

### 4. Cách cài đặt `auditd`:
- Kiểm tra xem nó được cài chưa:
  
![image](https://hackmd.io/_uploads/BJAwALpFbx.png)
- Nếu chưa, tiến hành cài đặt. Command nếu dùng Ubuntu: `sudo apt install auditd audispd-plugins -y`
- Sau khi cài, kiểm tra trạng thái:
  
![image](https://hackmd.io/_uploads/HJ5TJD6Y-e.png)
- Để thực hiện log hoạt động của tất cả user:
    - Mở file `/etc/audit/rules.d/audit.rules` bằng `sudo nano` và thêm hai dòng sau vào cuối file:
    ```
    -a exit,always -F arch=b64 -S execve
    -a exit,always -F arch=b32 -S execve
    ```
    - Restart: `sudo service auditd restart`
    - Mặc định log sẽ được lưu trữ ở file `/var/log/audit/audit.log`.

### 5. Sử dụng `auditd`:
- Để hiểu cách `auditd` hoạt động thì cần phải hiểu cách hoạt động của Linux daemon:
    - Một daemon là tiến trình ngầm không phục thuộc vào sự tương tác của active user,... Một user chung sẽ không thể kiểm soát sự thực thi chu kỳ của một daemon. Nó được khởi chạy khi Linux khởi động, và một tiến trình ban đầu sẽ hoạt động như tiến trình cha của nó. Để bắt đầu và dừng daemon, `/etc/init.d` scripts trên OS nên được truy cập đầu tiên.
    - Trong Linux, một daemon process có hậu tố `d`. Dùng hậu tố này, ta có thể phân biệt tiến trình nào là daemon hay hệ thống, hay tiến trình user. Bởi định nghĩa này, `auditd` là một tiến trình daemon.
- Vị trí `auditd` file:
    - `auditd.conf`:Nằm ở `/etc/audit/directory` và là file cấu hình của `auditd`.
    - Chứa các thông tin cấu hình cụ thể của audit daemon và bao gồm các dòng khác nhau của cấu hình, mỗi dòng có một keyword, dấu bằng và giá trị cấu hình thích hợp. Một số keyword available trong file:
        - `log_file`
        - `log_format`
        - `flush`
        - `freq`
        - `num_logs`
        - `max_ log_file`
        - `max_log_ file_action`
    - Kiểm tra file cấu hình bằng `sudo less /etc/audit/auditd.conf`:
      
    ![image](https://hackmd.io/_uploads/SkE0FD6Ybl.png)
- Ba tiện ích hay dùng: `auditctl`, `ausearch`, `aureport`
- Các thông số trong log:
    - `time`: Khi audit done.
    - `name`: Tên object được audit.
    - `cwd`: Thư mục hiện tại.
    - `syscall`: Lời gọi hệ thống.
    - `auid`: Audit user ID.
    - `uid` và `gid`: User ID và Group ID.
    - ...
- Tìm kiếm `auditd` events:
    - `auditd` không để ta xem log, để xem được thì phải dùng `ausearch`.
    - Các options phổ biển để truy vấn logs:
        - `-p`: Search event bằng PID (Process ID).
        - `-m`: Search event bằng message type.
        - `-sv`: Search event bằng success value.
        - `-ua`: Search event bằng user ID, effective user ID, hay login user ID hoặc auid.
        - `-ts`: Search event với timestamps bằng hoặc sau end time được cho.
    - Các lệnh `ausearch` thường dùng:
        - `ausearch -m USER_AUTH`: Xem các nội dung liên quan đến xác thực.
        - `ausearch -m USER_CMD`: Xem các nội dung liên quan đến lệnh mà user thưc hiện.
        - Xem thêm tại `ausearch -m`.
        - `man ausearch this-month - <today, this-week, this-mounth,...>`: Xem tất cả các log trong tháng.
        - `man ausearch this-month --file csv > audit.csv`: Export log trong tháng ra file CSV.
        - Xem thêm tại `sudo man ausearch`.
- Một số options phổ biến trong `auditctl`:
    - `-w`: Thêm một trình theo dõi vào file, `auditd` sẽ ghi lại hành động của user trong file đó.
    - `-k`: Trong một `auditd` rule cụ thể, thiết lập một string hoặc key tuỳ chọn để dùng cho việc nhận diện rule(s) tạo ra log entry cụ thể.
    - `-F`: Xây một trường rule bằng cách dùng tên, toán số học và/hoặc và một giá trị.
    - `-l`: Liệt kê tất cả `auditd` rule đang được load hiện tại theo nhiều dòng, mỗi dòng hiển thị một rule.
    - `-t`: Cắt bớt nhánh con xuất hiện sau lệnh mount.
    - `-S`: Chỉ định tên hoặc số hiệu system call.
    - `-a`: Thêm rule vào cuối danh mục được phân tách bằng dấu phẩy và các cặp danh sách:
        - Tên danh sách hợp lệ: `task`, `exit`, `user`, `exclude`
        - Tên hành động hợp lệ: `never`, `always`
        - Các cặp có thể có thể được sắp xếp ở thứ tự `(list, action)` hoặc `(action, list)`.
- Tạo `auditd` reports:
    - `aureport`
- Các lệnh `aureport` thường dùng:
    - `sudo aureport -au`: Report về các nỗ lực xác thực.
    - `sudo aureport -m`: Report tất cả event liên quan đến việc chỉnh sửa account.
    - Xem thêm bằng `sudo man aureport`.

### VII. `/etc/shadow`:
- Là file hệ thống trong đó mật khẩu user được mã hóa để không sẵn có cho những người cố gắng xâm nhập vào hệ thống
- Chi tiết hơn: [Understanding `/etc/shadow` file format on Linux](https://www.cyberciti.biz/faq/understanding-etcshadow-file/)
