> - Containers đã cách mạng hoá sự phát triển của phần mềm hiện đại bằng cách cho phép chúng ta xây dựng và triển khai các ứng dụng theo cách nhanh và đáng tin cậy hơn. Kết quả là, containerization trở nên phổ biến hơn trong những năm gần đây. Tuy nhiên, như bất kì những công nghệ khác, chúng không tránh khỏi các rủi ro bảo mật và lỗ hổng. Chúng cũng có thể được dùng bởi tội phạm cho các mục đích độc hại như điều khiển malware hay điều khiển các hoạt động bất hợp pháp.
> - Trong lab này, chúng ta sẽ hiểu cơ bản về containers, và tiếp đó sẽ khảo sát các tools cà công cụ có thể dùng để thực hiện điều tra trên môi trường container để nhận diện các mối đe doạ bảo mật tiềm tàng và hoạt động độc hại.

## I. Containers là gì?
### 1. Định nghĩa:
> - Containers là các packages của phần mềm, chứa tất cả những yếu tố cần thiết để chạy trong bất kì môi trường nào. Theo cách này, containers ảo hoá hệ thống vận hành và chạy bất cứ nơi nào, từ trung tâm dữ liệu riêng tư cho đến cloud công khai hay là trên laptop cá nhân của lập trình viên. Từ Gmail đến Youtube, tất cả mọi thứ trên Google đều chạy trong containers.
> - [Google Cloud](https://cloud.google.com/learn/what-are-containers#section-1)

Nói một cách đơn giản, một container là một gói các ứng dụng mà có tất cả những gì nó cần để chạy, bao gồm cả code và cấu hình, cũng như các denpendencies và thư viện.

### 2. Ví dụ:
- Tưởng tượng rằng ta đang lập trình một ứng dụng, và phần code cần một versions riêng của ngôn ngữ chương trình và các thư viện để chạy. Ta có thể cài đặt chúng trên máy cá nhân, nhưng chú có thể gây ra xung đột với các versions khác của ngôn ngữ hay thư viện có trên máy, ta có thể gặp phải những xung đột tương tự trên hosting server nơi đang triển khai ứng dụng.
- Có một cách thay thế, ta có thể đóng gói ứng dụng và dependencies của nó trong một container, nơi sẽ chạy ứng dụng với các versions được yêu cầu của ngôn ngữ chương trình và các thư viện mà không có bất kì xung đột nào với những thứ đã được cài trên máy host. Tiếp đó ta có thể di chuyển container này đến bất kì máy tính hay server nào khác, và ứng dụng sẽ chạy chính xác như cách nó đã chạy trên máy của chúng ta.

### 3. Sự khác nhau giữa Containers và VMs:
- Có thể thấy Containers tương tự như VMs: Một hệ thống vận hành khách như Linux hay Windows chạy trên hệ thống vận hành chủ và có quyền truy cập vào phần cứng bên dưới. Containers thường được so sánh với VMs. Giống như những máy ảo khác, containers cho phép đóng gói ứng dụng cùng với thư viện và các denpendencies khác, cung cấp môi trường cách ly để chạy các dịch vụ phần mềm.
- Để biết thêm về sự khác biệt: https://www.ibm.com/think/topics/containers-vs-vms

### 4. Docker là gì?
Docker là một công cụ điều phối container giúp quản lý toàn bộ vòng đời của các containers bao gồm xây dựng, chạy, và kết thúc. Các tools tương tự khác là Podman, Cri-o, Runc, Containerd,... cũng cung cấp các tính năng trên.

## II. Bằng chứng tìm được:
- Trong bối cảnh của container forensics, bằng chứng của ta thường bao gồm bất kì thông tin nào có thể thu thập được từ containers - cho dùng đang hoạt động hay đã dừng, container images, container logs, logs thời gian chạy/động cơ,... Có một số cách để thu thập những chứng cứ liên quan đến một container, bao gồm trích xuất filesystem của nó, kiểm tra sự khác biệt giữa container và base image của nó, kiểm tả metadata và cấu hình của container, thu thập memory dump của các tiến trình đang chạy trong container.
- Khảo sát một số phương pháp một:

### 1. Trích xuất Fileysytem:
Một cách để thu thập bằng chứng là trích xuất filesystem của container theo dạng nén:
- Để minh hoạ quá trình này, trước tiên khởi tạo một container:
    ``` ubuntu
    docker run -it alpine sh
    ```
    ![image](https://hackmd.io/_uploads/r1MFTf9RZx.png)
- Trong container này, tạo một file tên là `hello_world` với nội dung là `Hello, World!`:
    ``` container
    #/ echo "Hello, World!" > hello_world
    ```
- Trong khi vẫn để container vừa rồi chạy, mở một terminal khác và kiểm tra ID của container:
    ``` ubuntu
    docker ps --all
    ```
    ![image](https://hackmd.io/_uploads/r1rvAfq0-x.png)
- Xuất filesystem của container ở dạng nén:
    ``` ubuntu
    docker export 64af19c5b443 -o output.tar
    ```
- Tiếp theo, xuất filesystem và xác nhận rằng file `hello_world` đang tồn tại bằng cách chạy lần lượt các command:
    ``` ubuntu
    mkdir output
    tar -xf output.tar -C output
    ls -la output
    cat output/hello_world 
    ```
    ![image](https://hackmd.io/_uploads/SkaVem9Abg.png)

### 2. Tìm sự khác biệt giữa Container và Base Image:
- Gồm việc so sánh trạng thái hiện tải của container đang chạy hay đã dừng lại với base image của nó để tìm ra những sự thay đổi đã thực hiện. Việc này rất hữu ích trong việc nhận diện bất kì sự thay đổi nào có thể đã thực hiện trên container bởi malware hay các thực thể trái phép.
- Một cách để tìm kiếm sự khác biệt là dùng command `docker diff`, và theo tài liệu của Docker thì có ba sự thay đổi được theo dõi:
![image](https://hackmd.io/_uploads/ryDwZmcC-g.png)
- Có thể thử nó trên container đã tạo ở ví dụ trước:
![image](https://hackmd.io/_uploads/r1qC-Qq0Zx.png)
Có thể thấy trong output, thư mục `/root` đã thay đổi, mà một file mới được thêm vào tên là `/root/.ash_history` (tự động tạo khi thực thi command đầu tiên trong container), cùng với `/hello_world` đã tạo.
> Đáng chú ý rằng shell mặc định của `Alpine` là `ash`, nên file lịch sử tên là `.ash_history` thay vì `.sh_history`, mặc dù ta đã chỉ định `sh` trong lệnh `docker run`.

### 3. Kiểm tra Container Metadata và Cấu hình:
- Kiểm tra cấu hình của container bao gồm việc đánh giá chi tiết cấu hình như cài đặt networking, các biến môi trường, và cấu hình lưu trữ.
- Một cách để kiểm tra cấu hình container là dùng command `docker inspect`, command này cung cấp cái nhìn chi tiết và cấu hình và metadata của container:
![image](https://hackmd.io/_uploads/H1RnN7qAWg.png)
![image](https://hackmd.io/_uploads/SJaCEQ9AZx.png)
![image](https://hackmd.io/_uploads/HJpgS75RZl.png)
![image](https://hackmd.io/_uploads/SyXfr750Zl.png)
![image](https://hackmd.io/_uploads/HJCXHQcCWg.png)

### 4. Hiển thị Container Logs:
Để xem bất kỳ command nào đã được thực thi trong container, hay logs của bất kỳ dịch vụ nào có thể đã đang chạy, dùng command `docker logs`:
![image](https://hackmd.io/_uploads/SkriBXq0Zg.png)

### 5. Kiểm tra Lịch sử Image:
- Kiểm tra lịch sử của một image giúp tìm ra bất kỳ sự thay đổi nào đã thực hiện, bao gồm những sự điều chỉnh độc hại tiềm tàng hay sự các files hoặc layers độc hại được thêm vào. Điều này đặc biệt hữu ích trong việc kiểm tra xem một image đã bị can thiệp trái phép chưa.
- Dùng command `docker history` để hiển thị lịch sử của image theo thứ tự thời gian ngược, cho thấy mỗi layer đã được thêm vào image:
![image](https://hackmd.io/_uploads/H10gDQ5AZx.png)

### 6. Thu thập Memory Dump của Container:
- Thu thập memory dump của một tiến trình trong container có thể hữu ích trong việc điều tra trạng thái runtime của nó, nhận diện các hoạt động độc hại, hoặc trích xuất secrets.
- Có thể dùng các tools như `dd`, `gdb`, hay `gcore` để thu thập memory dump của một tiến trình trong container. Ta sẽ dùng `gcore` trong các options này vì nó yêu cầu process ID.
    - Cài `gdb`:
    ``` ubuntu
    sudo apt update && sudo apt install gdb -y
    ```
    - Kiểm tra tiến trình đang tồn tại trong container:
    ``` ubuntu
    docker inspect --format '{{.State.Pid}}' 64af19c5b443
    ```
    ![image](https://hackmd.io/_uploads/S1eAomqRZe.png)
    - Thực hiện dump:
    ``` ubuntu
    sudo gcore 2079
    ```
    ![image](https://hackmd.io/_uploads/rkUGhQcCbx.png)
- Một khi có được memory dump, ta có thể tìm kiếm bất kì thông tin thú vị nào như secrets hay strings độc hại. Ví dụ, ta đã viết `Hello, World` vào một file trong container, thử tìm strings này:
![image](https://hackmd.io/_uploads/rkU93X5R-e.png)

## III. Tổng kết
- Các công cụ khác tự động hoá các quy trình cơ bản trong containers: [docker forensics toolkit](https://github.com/docker-forensics-toolkit/toolkit), [docker explorer](https://github.com/google/docker-explorer), [container explorer](https://github.com/google/container-explorer), và [docker layer extract](https://github.com/micahyoung/docker-layer-extract).
- Kiến thức sâu hơn về containers và bảo mật: [Container Security](https://www.oreilly.com/library/view/container-security/9781492056690/)
- Cách dùng Dockerfile: https://docs.docker.com/reference/dockerfile
