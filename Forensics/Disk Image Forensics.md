## I. Disk Image Forensics:
- Các thiết bị lưu trữ kỹ thuật số, như ổ cứng, solid-state drives, USB, lưu trữ một lượng lớn dữ liệu có thể có ích trong điều tra forensics kỹ thuật số. Disk image forensics là quá trình phân tích các thiết bị này và nội dung của nó để tìm kiếm các thông tin hữu ích cho cuộc điều tra.
- Trong lab này, ta sẽ khảo sát cơ bản disk image forensics, bao gồm thuật ngữ cơ bản với các bước thu được và phân tích một disk image.

### Các thuật ngữ cơ bản:
#### 1. Disk Image:
- Là bản copy từng bit của toàn bộ disk (như ổ cứng, USB,...) hoặc partion giữ chính xác nội dung và cấu trúc của dữ liệu gốc.
- Không chỉ chứa mỗi files và folders mà có cả empty space, metadata và các hidden data khác không thể nhìn thấy bằng cách bình thường.

#### 2. Disk Imaging:
- Là quá trình khởi tạo bản copy forensics của thiết bị lưu trữ như ổ cứng và USB. Đây là bước quan trọng trong digital forensics vì nó đảm bảo rằng dữ liệu gốc vẫn chưa bị đụng đến và điều chỉnh.
- Các mã băm dùng để nhận diện rằng bản copy giống chính xác với bản gốc, đảm bảo rằng không có sự điều chỉnh nào được thực hiện với dữ liệu gốc.
- Hướng tiếp cận này cho phép việc điều tra forensics làm việc với bản copy mà không lo lắng về việc vô tình làm thay đổi dữ liệu gốc.

#### 3. d:
- Là quá trình phân tích một disk image để tìm kiếm bằng chứng.
- Gồm các việc như dùng tools để khám phá các thông tin hữu ích và phân tích các artifacts hệ thống như Windows Registry, trình duyệt web, LNK Files, event logs, lịch sử Command Prompt,...
- Có các tools khác nhau có thể dùng như Autopsy và FTK Imager. Trong khi Autopsy offers nhiều features như phân tích, ta sẽ dùng FTK Imager vì nó nhẹ.

## II. Acquiring a Disk Image:
Các bước tạo một disk image bằng FTK Imager:
- `File` $\rightarrow$ `Create Disk Image`:

![image](https://hackmd.io/_uploads/SJggigKiL-e.png)
- Chọn `Physical Drive` $\rightarrow$ `Next >` và chọn drive ta muốn tạo một image:

![image](https://hackmd.io/_uploads/rJF0xYo8Zx.png)
![image](https://hackmd.io/_uploads/Hy2lWti8bl.png)
- Click `Add` dưới phần `Image Destination(s)`, sau đó chọn `Raw (dd)` rồi `Next >`:

![image](https://hackmd.io/_uploads/BJquZKs8Ze.png)
![image](https://hackmd.io/_uploads/Sk1K-tjIZg.png)
- Điền các thông tin liên quan về bằng chứng rồi chọn một folder và tên để lưu disk image:

![image](https://hackmd.io/_uploads/r1J3WFjUWx.png)
![image](https://hackmd.io/_uploads/SJ8hZYsUZl.png)
- Click `Start` để bắt đầu quá trình khởi tạo, thời gian bao lâu tuỳ thuộc vào size của drive:

![image](https://hackmd.io/_uploads/ByaxztjUbl.png)
![image](https://hackmd.io/_uploads/H1ObzYiIZl.png)

## III. Analyzing a Disk Image:
- Một khi disk image được tạo, bước tiếp theo là phân tích nó để tìm bất kì bằng chứng nào liên quan đến việc điều tra.
> Mặc dù FTK Imager nhẹ, nó yêu ta phải có kiến thức nhất định nằm trong một disk image và vị trí của nó. Mặt khác, Autopsy tự động phân tích các thông tin hữu ích như images, lịch sử Internet, vị trí địa lý, timeline,... Nó có thể khôi phục các files đã xoá, tìm kiếm theo patterns trong disk image và tạo báo cáo chi tiết.
- [Link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2006/files/Image.ad1) download disk image của lab.
- Để mở một disk image trong FTK Imager, chọn `File` $\rightarrow$ `Add Evidence Item...` $\rightarrow$ `Image File` và chọn disk image ta muốn phân tích:

![image](https://hackmd.io/_uploads/HyJhmFo8Ze.png)
![image](https://hackmd.io/_uploads/HyQTXYo8We.png)
![image](https://hackmd.io/_uploads/r1oJEtiLbx.png)
- Khi mở disk image trên FTK Imager, ta sẽ thấy bốn phần:

![image](https://hackmd.io/_uploads/BkrbLKoUWl.png)
    - `Evidence Tree`: Góc trái trên, hiển thị layout của disk image theo thứ tự.
    - `Properties`: Góc trái dưới, hiển thị metadata liên kết với file được chọn như tên, lần cuối điều chỉnh và thời gian truy cập, `md5` và mã băm `sha1`,...
    - `File List`: Center trên, hiển thị danh sách các files và thư mục trong partition hay image được chọn.
    - Preview: Phần dưới, hiển thị preview hay hex contents của file được chọn.
- Ta có thể giải nén bất kì files nào bằng cách chuột phải và chọn `Export Files`:

![image](https://hackmd.io/_uploads/Byp4IKo8Ze.png)

### Một số files quan trọng trong disk images:
- `$MFT` - The Master File Table: Là một file quan trọng trong hệ thống NTFS files, chứa thông tin về tất cả các files và thư mục trên một ổ đĩa, gồm tên, quyền và các thuộc tính. Nó cũng chứa thông tin về vị trí của mỗi file trên disk.
- `$MFTMirr` - MFT Mirror: Được dùng như backup của `$MFT` và rất quan trọng trong trường hợp `$MFT` gốc bị sửa đổi.
- `$LogFile`: Ghi lại thông tin nhật ký giao dịch của metadata (vùng MFT), có thể dùng để khôi phục từ hệ thống bị lỗi.
> Có nhiều files có thể hiện thị trong disk image như `$Boot`, `$Secure`, `$Volume`,...

$\rightarrow$ Để phân tích file này, có thể dùng các tool khác nhau như [analyzeMFT](https://github.com/dkovar/analyzeMFT) hay [MFTECmd](https://github.com/EricZimmerman/MFTECmd) bằng cách tải ở [link](https://ericzimmerman.github.io/#!index.md).
