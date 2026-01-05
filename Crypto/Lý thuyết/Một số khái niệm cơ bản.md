**I. Cryptography:**
1. Khái niệm:
- Là thuật ngữ được sử dụng phổ biến trong ngành nghiên cứu mật mã, mã hoá và giải mã thông tin.
- Ngành này chuyên thực hiện các nghiên cứu về cách thức và phương pháp nhằm chuyển đổi thông tin từ phức tạp sang đơn giản Plaintext (đọc hiểu) sang dạng Ciphertext (động nhưng không hiểu ý nghĩa) và ngược lại.

2. Phân loại:
Gồm nhiều cách khác nhau. Tuy nhiên cách phổ biến nhất là dựa vào số lượng khoá học sử dụng để mã hoá và giải mã:
- **SKC - Mật mã khoá bí mật**: Dùng một khoá cho tất cả các mã hoá và giải mã. Kiểu khoá này được gọi là mã hoá đối xứng và dùng trong môi trường riêng tư, bảo mật.
- **PKC - Mật mã công khai hay mã hoá bất đối xứng**: Dùng một khoá để mã hoá và một khoá để giải mã, thường được ứng dụng trong việc xác thực thông tin.
- **Hàm băm**: Biến thông tin thành mã hoá không thể đảo ngược bởi thuật toán thông minh và dùng dấu vân tay để xác định và nó thường được ứng dụng để bảo toàn tin nhắn.

3. Chức năng:
- Quyền riêng tư/bảo mật: Đảm bảo không ai có thể đọc tin nhắn ngoại trừ người nhận dự định.
- Xác thực: Quá trình chứng minh danh tính của một người.
- Tính toàn vẹn: Đảm bảo tin nhắn đã nhận không bị thay đổi theo bất kỳ cách nào so với bản gốc.
- Không thoái thác: Một cơ chế để chứng minh người gửi đã thực sự gửi tin nhắn này.
- Trao đổi khoá: Phương thức các khoá bảo mật được chia sẻ giữa người gửi và người nhận.
Trong mật mã, dữ liệu không được mã hoá gọi là bản rõ. Bản rõ được mã hoá thành bản mã, lần lượt (thường) được giải mã trở lại thành bản rõ có thể sử dụng được. Việc mã hoá và giải mã dựa trên loại sơ đồ mã hoá đang được sử dụng và một số dạng khoá. Vì điều này, một số chức năng khác có thể được hỗ trợ bởi tiền điện tử và các điều khoản khác: 
- Chuyển tiếp bí mật (Bí mật chuyển tiếp hoàn hảo): Bảo vệ các phiên được mã hoá trong quá khứ khỏi sự thoả hiệp ngay cả khi máy chủ giữ tin nhắn bị xâm phạm bằng cách tạo một khoá khác nhau cho mỗi phiên mã để việc thoả hiệp một khoá duy nhất không đe doạ toàn bộ thông tin liên lạc.
- Bảo mật hoàn hảo: Một hệ thống không thể phá vỡ và trong đó bản mã truyền tải không có thông tin về bãn rõ hoặc khoá. Để đạt được bảo mật hoàn hảo, khoá tối thiểu phải dài bằng bản rõ, khiến cho việc phân tích và thậm chí các cuộc tấn công vũ phu là không thể. Miếng đệm một lần là một ví dụ về một hệ thống như vậy.
- Xác thực từ chối (Từ chối tin nhắn): Phương thức người tham gia trao đổi tin nhắn có thể được đảm bảo về tính xác thực của tin nhắn nhưng theo cách đó, người gửi sau đó có thể từ chối một cách hợp lý sự tham gia của họ với bên thứ ba.

4. Tại sao nên sử dụng:
- Đảm bảo thông tin được truyền đi mà không bị đọc và hiểu một cách dễ dàng.
- Tránh khỏi nguy cơ bị mất thông tin.

5. Một số thuật ngữ được sử dụng trong Cryptography:
- Sender/Receiver: Người gửi/nhận dữ liệu.
- Attacker/Hacker: Người tấn công hoặc vận chuyển thông tin trên đường truyền.
- Plaintext: Thông tin trước khi được mã hoá, dữ liệu ban đầu ở dạng có thế đọc - hiểu được.
- Ciphertext: Thông tin đã được mã hoá, dữ liệu ở dạng đọc được nhưng không hiểu được.
- Encryption: Quá trình mã hoá thông tin (Plaintext --> Ciphertext).
- Decryption: Quá trình giải mã lấy lại thông tin ban đầu (Ciphertext --> Plaintext).
6. Các kỹ thuật phổ biến trong mã hoá điện tử: 
- Symmetric encryption.
- Asymmetric encryption.
- Hash functions.
- Digita; signatures.

**II. Cryptanalysis - Thám mã:**
   1. Định nghĩa:
    Là quá trình nghiên cứu, phân tích các hệ thống mã hoá(crytographic systems) nhằm tìm cách giải mã thông tin mà không có khoá giải mã (decryptiion key).
Mục tiêu: Phá vỡ, đánh bại các biện pháp bảo mật của hệ thống mã hoá, giúp thu hồi thông tin ban đầu mà không cần phải biết khoá mật.

2. Chức năng:
- **Tìm lỗ hổng bảo mật**: Phát hiện những yếu điểm trong các thuật toán mã hoá, từ đó tìm cách khôi phục thông tin mà không cần phải biết khoá.
- **Giải mã thông tin**: Trong các tình huống cụ thể, có thể phục hồi thông tin mã hoá mà không có quyền truy cập vào khoá giải mã.
- **Phát hiện và ngăn chặn các cuộc tấn công**: Tìm cách phòng ngừa các phương pháp tấn công vào hệ thống mã hoá.

3. Các phương pháp cryptanalysis:
- Brute force: Thử tất cả các khả năng.
- Frequency analysis: Phân tích mật mã.
- Tìm kiếm các mẫu hoặc yếu tố lặp lại trong dữ liệu mã hoá.
- ...
4. Ý nghĩa:
    Rất quan trọng trong việc đảm bảo an toàn và bảo mật trong các ứng dụng như truyền thông điện tử, ngân hàng trực tuyến, giao dịch tiền điện tử,...
    
**III. Encode - Decode:**
1. Encode - Mã hoá:
*a) Định nghĩa:*
- Là quá trình chuyển đổi dữ liệu từ một định dạng thông thường sang một định dạng khác không thể đọc được, giúp giảm dung lượng lưu trữ và tăng tốc độ truyền tải dữ liệu.
- Quá trình mã hoá gồm các bước:
B1: Dung chương trình biên dịch để thực hiện mã hoá.
B2: Truyền tải, nén và lưu trữ dữ liệu sau khi đã được mã hoá.
B3: Xử lý dữ liệu bằng cách chuyển đổi các tập tin và ứng dụng.
- Thường áo dụng để lưu trữ các tập tin đa phương tiện trong ổ đĩa.

*b) Phân loại:*
- **Mã hoá âm thanh**: Quá trình chuyển đổi cách thông tin về tần số, biên độ, bước sóng, chu kỳ và vận tốc truyền tải âm thanh từ định dạng này sang định dạng khác (công nghệ phổ biến là Advanced Audio Coding - AAC).
- **Mã hoá ký tự**: Quá trình chuyển đổi cấu trúc của văn bản thành một dạng mà con người có thể nhìn thấy nhưng không thể đọc được nếu không có phương tiện giải mã.
- **Mã hoá hình ảnh**: Quá trình chuyển đổi định dạng của hình ảnh từ một định dạng thông thường sang định dạng khác để đảm bảo an toàn trong quá trình truyền tải qua mạng. --> Ngăn chặn người khác xem được nội dung bên trong hình ảnh.

*c) Ý nghĩa:*
- **Mã hoá ký tự**: Quá trình mã hoá giúp máy tính hiểu và hiển thị ký tự nhập vào. Dữ liệu mỗi ký tự sẽ được lưu trữ dưới dạng mã hoá trong bộ nhớ máy tính, thường là theo định dạng byte hoặc bit.
- **Encode và mô hình mã hoá UNICODE**: Mô hình mã hoá UNICODE đặt ra tiêu chuyển cho định danh của mỗi ký tự trong nhiều ngôn ngữ khác nhau. UNICODE sử dụng **Điểm Mã** để định danh mỗi ký tự, cung cấp giải pháp chuẩn cho việc mã hoá ký tự.
- **Encode Video**: Quá trình chuyển mã hoặc chuyển đổi video để đảm bảo khả năng tương thích với nhiều thiết bị và trình duyệt khác nhau.

2. Decode - Giải mã:
*a) Định nghĩa:*
    - Là quá trình chuyển đổi dữ liệu đã được mã hoá về dạng ban đầu có thể đọc hiểu được bởi con người hay máy tính, được thực hiện thông qua việc huỷ các tác vụ mã hoá để đưa về dữ liệu gốc.
    - Mã hoá và giải mã luôn được sử dụng song song với nhau để phát huy được hết hiệu quả của mật mã.

    *b) Phân loại:*
    - **Giải mã cổ điển**: Hoạt động dựa trên cơ sở bảng chữ cái, thường thực hiện thủ công bằng tay hoặc một số máy móc cơ khí đơn giản và chỉ áp dụng cho dữ liệu được mã hoá cổ điển, không gây nhiều khó khăn cho người giải mã. Hiện nay đã lỗi thời vì các dữ liệu dễ dàng bị xử lý.
    - **Giải mã một chiều**: Phương pháp giải mã thông tin mà không cần dịch nguyên văn văn bản gốc.
Ví dụ: Hash Function: Đổi một chuỗi ký tự có độ dài cố định nhằm kiểm tra tính toàn vẹn dữ liệu.
    - **Giải mã đối xứng**: Đại diện cho một dữ liệu được phân hưởng bởi hai hoặc nhiều bên đối xứng nhau. Gồm hai loại:
Stream Ciphers (Mã luồng): Giải mã từng bit của thông điệp.
Block Ciphers (Mã khối): Gộp một số bit lại với nhau thành đơn vị để giải mã.
    - **Giải mã bất đối xứng**: Cần hai chìa khoá (đóng vai trò như chìa khoá vạn năng) để giải mã thông tin bí mật là Public Key và Private Key.
        + Quy trình mã hoá và giải mã bất đối xứng:
B1: Sử dụng Public Key để mã hoá các dữ liệu thông tin và chuyển file cho người nhận.
B2: Người nhận dùng Private Key để đọc các dữ liệu đó.
        + Nhược điểm: CPU cần tốn nhiều năng lực xử lý hơn, tốc độ khá chậm so với giải mã đối xứng. --> Chi phí cao.

    *c) Một số thiết bị máy móc giải mã hiện nay:*
    - Secret Service for Mac: Thiết bị chuyên dụng để giải mã dành cho máy Mac.
    - Encryption and Decryption: Hỗ trợ tính năng mã hoá và giải mã dữ liệu hiệu quả, đảm bảo bảo mật tuyệt đối các dữ liệu.
    - Encrypt Easy: Công cụ hỗ trợ giải mã mạnh mẽ cho những ai không chuyên.
    - Crypt for Mac: Cung cấp tính năng giải mã và mã hoá dữ liệu, phù hợp sử dụng cho công ty doanh nghiệp.
    
**IV. Encryption - Decryption:**
1. Encryption:
    *a) Định nghĩa:*
    - Là quá trình chuyển đổi thông tin từ dạng ban đầu thành dạng không đọc được (mã hoá) bằng cách sử dụng thuật toán và khoá mật mã.
    - Mục đích: Bảo vệ tính bảo mật và riêng tư của dữ liệu trước khi gửi hoặc lưu trữ.

   *b) Các loại mã hoá hiện nay:*
    - **Bring Your Own Encryption (BYOE):** Cho phép người dùng tự chọn và quản lý các khoá mã hoá của họ khi lưu trữ dữ liệu trên các dịch vụ lưu trữ đám mây.
    - **Cloud Storage Encryption:** Mã hoá dữ liệu trước khi lưu trữ trên các dịch vụ đám mây, đảm bảo tính riêng tư và bảo mật trong quá trình truyền và lưu trữ.
    - **Column - level Encryption:** Mã hoá dữ liệu trên mức cột (cột trong cơ sở dữ liệu), cho phép kiểm soát truy cập vào các cột cụ thể của dữ liệu.
    - **Deniable Encryption:** Mã hoá dữ liệu sao cho người khác không thể chắc chắn rằng dữ liệu đã được mã hoá, tạo sự phủ định và mơ hồ về tính nội dung của dữ liệu.
    - **Encryption as a Service (EaaS):** Cung cấp các dịch vụ mã hoá thông qua mô hình dịch vụ, giúp người dùng thực hiện mã hoá mà không cần quản lý trực tiếp hạ tầng mã hoá.
    - **End - to - End Encryption (E2EE):** Mã hoá dữ liệu tại nguồn và chỉ giải mã tại điểm đích cuối cùng, đảm bảo rằng dữ liệu chỉ có thể đọc được bởi người nhận.
    - **Field - level Encryption:** Tương tự Column - level Encryption nhưng mã hoá dữ liệu trên mức trường (phần của một cột), tạo sự kiểm soát cụ thể hơn trên dữ liệu.
    - **Full Disk Encryption (FDE):** Mã hoá toàn bộ ổ đĩa lưu trữ hoặc thiết bị, đảm bảo rằng tất cả dữ liệu trên đó được bảo bệ ngay cả khi thiết bị bị mất hoặc đánh cắp.
    - **Homomorphic Encryption:** Cho phép tính toán trên dữ liệu đã mã hoá mà không cần giải mã trước, giữ cho dữ liệu luôn ở dạng mã hoá trong quá trình tính toán.
    - **HTTPS:** Giao thức an toàn trên Internet, mã hoá dữ liệu truyền tải giữa máy khác và máy chủ web để đảm bảo tính bí mật và toàn vẹn của thông tin.
    - **Link - level Encryption:** Mã hoá dữ liệu trên mức liên kết, đảm bảo tính bí mật khi dữ liệu chuyển qua các kết nối mạng khác nhau, bao gồm cả mạng nội bộ và Internet.
    - **Networl - level Encryption:** Mã hoá dữ liệu trên mức mạng, đảm bảo rằng dữ liệu không thể bị đánh cắp hoặc theo dõi trong quá trình truyền tải qua các kết nối mạng.

    *c) Cơ chế hoạt động:*
Quá trình chuyển đổi thông tin gốc thành dạng không đọc được bằng cách sử dụng thuật toán và khoá mật với các bước:
    - **Mã hoá - Encryption:** Dữ liệu gốc được chuyển đổi thành dạng mã hoá bằng các áp dụng thuật toán mã hoá và khoá mật, kết quả là dữ liệu mã hoá không thể đọc được.
    - **Truyền thông tin mã hoá:** Dữ liệu mã hoá có thể được truyền từ nguồn tới đích thông qua mạng hoặc kênh truyền dữ liệu.
    - **Giải mã:** Người nhận sử dụng khoá mật để giải mã dữ liệu mã hoá và chuyển đổi nó trờ lại dạng ban đầu. 
    
    *d) Một số câu hỏi thường gặp:*
    - Thuật toán mã hoá là gì?
        - Là một chuỗi các bước và quy tắc được sử dụng để chuyển đổi thông tin từ dạng ban đầu thành dạng không đọc được (mã hoá) bằng cách sử dụng các khối mã hoá và khoá mật.
        - Xác định các thức thực hiện việc mã hoá và giải mã dữ liệu, đảm bảo tính bảo mật và riêng tư của thông tin. Mỗi thuật toán có cấu trúc và quy tắc riêng để thực hiện việc mã hoá và giải mã dữ liệu.
    - Một số thuật toán mã hoá Encryption phổ biến:
        - Thuật toán mã hoá đối xứng:
        1) AES
        2) 3-DES
        3) SNOW
        4) IDEA
        5) Blowfish
        6) Twofish
        7) RC4
        8) RC6
        - Thuật toán mã hoá không đối xứng:
        1) RSA
        2) ECC - Mật mã đường cong Elliptic.
        3) Diffie - Hellman
        4) DSA (Digital Signature Algorithm)
        5) ElGamal:
        6) Blum Blum Shub:
        7) Lattice - based Cryptography
    - Hàm băm (Hash function):
        - Là mã hoá một chiều (one - way cryptography) dùng một biến đổi toán học để mã hoá thông tin gốc thành một dạng không biến đổi ngược được, không có chìa khoá vì từ ciphertext không tìm ngược lại được plaintext.
        - Các loại hàm băm nổi tiếng: 
        1) MD5
        2) SHA-1
        3) SHA-256, SHA-384, SHA-512
        4) Whirlpool
        5) Bcrypt
        6) Scrypt
        7) SHA-3
    - Phương pháp Brute Force Attack:
        - Là phương pháp tấn công mà kẻ tấn công thử tất cả các khả năng có thể của mật khẩu hoặc khoá mã hoá để tìm ra giá trị chính xác. Kỹ thuật này không sử dụng bất kỳ thông tin hay kiến thức nào về mật khẩu hoặc khoá mã hoá mà chỉ dựa bào việc thử từng giá trị một cho đến khi tìm ra giá trị đúng.
        - Tuy mất rất nhiều thời gian và tài nguyên tính toán, nhưng nếu không có biện pháp bảo vệ phù hợp thì cách này sẽ giúp tìm ra mật khẩu hay khoá mã hoá chính xác nhất.

2. Decryption: 
    Là quá trình ngược lại với mã hoá, khôi phục lại những thông tin dạng ban đầu từ thông tin ở dạng được mã hoá.
    
V. Symmetric - Asymmetric:
1. Symmetric Key (Hệ mật mã khoá đối xứng): 
    *a) Định nghĩa:*
    - Là những hệ mật được sử dụng chung một khoá trong quá trình mã hoá và giải mã, do đó khoá phải được giữ bí mật tuyệt đối.
    - Một số hệ mật mã khoá đối xứng hiện đại phổ biến:
        - DES
        - AES
        - RC4
        - RC5
        - ... 
        
    *b) Bao gồm:*
    - Bản rõ (Plaintext - M): Bản tin được sinh ra bởi bên gửi.
    - Bản mật (Ciphertext - C): Bản tin che giấu thông tin của bản rõ, được gửi tến bên nhận qua một kênh không bí mật.
    - Khoá (KS): Là giá trị ngẫu nhiên và bí mật được chia sẻ giữa các bên trao đổi thông tin và được tạo ra từ bên thứ ba được tin cậy và phân phối tới bên gửi, bên nhận hoặc bên gửi tạo ra và chuyển cho bên nhận.
    - Mã hoá(encrypt - E): **C = E(KS, M)**
    - Giải mã (decrypt): **M = D(KS, C) = D(KS, E(KS, M))**
    
    *c) Cơ chế hoạt động:*
    - Người gửi sử dụng khoá chung (KS) để mã hoá thông tin rồi gửi cho người nhận.
    - Người nhận nhận được thông tin sẽ dùng chính khoá chung (KS) để giải mã.
    
    *d) Hạn chế:*
    - Do dùng chung khoá để mã hoá và giải mã $\Rightarrow$ Nếu bị mất hoặc bị đánh cắp bởi hacker sẽ bị lộ thông tin, bảo mật không cao.
    - Cần kênh mật để chia sẻ khoá bí mật giữa các bên $\Rightarrow$ Làm sao để chia sẻ một cách an toàn ở lần đầu tiên.
    - Để đảm bảo liên lạc an toàn cho tất cả mọi người trong nhóm gồm n người, cần tổng số lượng lớn khoá là $frac{n(n-1)}/{2}$.
    - Khó ứng dụng trong các hệ thông mở.
    - Không thể dùng cho mục đích xác thực hay chống thoái thác.
    
2. Asymmetric Key (Hệ mật mã khoá bất đối xứng):
    *a) Định nghĩa:*
    - Ở hệ mật này thay vì dùng chung một khoá như ở hệ mật mã khoá đối xứng thì ở đây sẽ dùng một cặp khoá có tên là public key và private key.
    - Phổ biến nhất là RSA.

    *b) Bao gồm:*
    - Bản rõ (Plaintext - M): Bản tin được sinh ra bởi bên gửi.
    - Bản mật (Ciphertext - C): Bản tin che giấu thông tin của bản rõ, được gửi tới bên nhận thông qua một kênh không bí mật.
    - Khoá: Bên nhận có một cặp khoá:
        - Khoá công khai (Kub): Công bố cho tất cả mọi người biết (kể cả hacker).
        - Khoá riêng (Krb): Bên nhận giữ bí mật, không chia sẻ cho bất kì ai.
    - Mã hoá (encrypt - E): C = E(Kub, M).
    - Giải mã (devrypt): M = D(Krb, C) = D(Krb, E(Kub, M))
    *Yêu cầu với cặp khoá (Kub, Krb):
    - Hoàn toàn ngẫu nhiên.
    - Có quan hệ về mặt toán học 1 - 1.
    - Nếu chỉ có giá trị của Kub thì không thể tính được Krb.
    - Krb phải được giữ bí mật hoàn toàn.

    *c) Cơ chế hoạt động:*
    - Người gửi gửi thông tin đã được mã hoá bằng khoá công khai (Krb) tới người nhận thông qua kênh truyền thông tin không bí mật.
    - Người nhận nhận được thông tin đó sẽ giải mã bằng khoá riêng (Krb) của mình.
    - Hacker cũng sẽ biết khoá công khai (Kub) những không thể xem được thông tin gửi nếu khoogn có khoá riêng (Krb).

    *d) Ưu - Nhược điểm:*
    - Ưu điểm:
        - Không cần chia sẽ khoá mã hoá (khoá công khai) một cách bí mật. $\Rightarrow$ Dễ dàng ứng dụng trong các hệ thống mở.
        - Khoá giải mã (Khoá riêng) chỉ có người nhận biết $\Rightarrow$ An toàn hơn, có thể xác thực nguồn gốc thông tin.
        - n phần tử chỉ cần n cặp khoá.
    - Nhược điểm: 
        - Chưa có kênh an toàn để chia sẻ khoá $\Rightarrow$ Khả năng bị tấn công dạng tấn công người đứng giữa (man in the middle attack), có thể phòng ngừa bằng các phương pháp Trao đổi khoá **Diffie - Hellman** nhằm đảm bảo nhận thực người gửi và toàn vẹn thông tin.
        - Dạng tấn công người đứng giữa: Kẻ tấn công lợi dụng việc phân phối khoá công khai để thay đổi khoá công khai. Sau khi đã giả mạo được khoá công khai, kẻ tấn công đứng ở giữa hai bên để nhận các gói tin, giải mã rồi lại mã hoá với khoá đúng và gửi đến nơi nhận để tránh bị phát hiện.

VI. Block Cipher - Stream Cipher:
1. Block Cipher (Kiến trúc mã hoá khối):
    *a) Định nghĩa:*
Là phương pháp mã hoá dữ liệu trong Block nhằm tạo ra các bản mã dựa vào các thuật toán và khoá mật mã. Khác với Stream Cipher chỉ có thể mã hoá dữ liệu từng bit một, Block Cipher giúp xử lí các Block có kích thước cố định cùng một lúc và thường là 64 hoặc 128 bit.

    *b) Cách thức hoạt động:*
    ![image](https://hackmd.io/_uploads/r16Sjt_7kx.png)
    - Block Cipher sử dụng khoá và thuật toán Symmetric để mã hoá và giải mã một Block dữ liệu. Trong đó Block Cipher này yêu cầu một IV được bổ sung vào văn bản gốc đầu vào nhằm mở rộng không gian khoá của mật mã. Điều này giúp cho việc giải mã khoá của kẻ tấn công trở nên khó khăn hơn.
    - IV được tạo ngẫu nhiên từ trình tạo số tích hợp với văn bản trong Block đầu tiên và khoá. Điều này nhằm đảm bảo rằng tác cả các Block tiếp theo tạo ra văn bản mã hoá không trùng lặp với văn bản của khối mã hoá đầu tiên. Kích thước Block của Block Cipher tức là số lượng bit được xử lý cùng một lúc.
    - DES Block Cipher được IBM triển khai vào năm 1975 gồm các Block 64 bit và khoá 56 bit. Loại mật mã này không được đánh giá cao về độ an toàn bởi kích thước khoá ngắn. Do đó, năm 1998, IBM đã thiết kế ra AES với kích thước Block là 128 bit và kích thước khoá là 128-, 192- hoặc 256- bit.

    *c) Các chế độ hoạt động:*
    Block Cipher có nhiệm vụ mã hoá các tin nhắn có cùng kích thước với chiều dài Block. Do đó, mỗi văn bản cần có các Block được mã hoá riêng. Một số chế độ hoạt động của Block Cipher.
    - **Chế độ Electronic Codeblock (ECB):**
    ![image](https://hackmd.io/_uploads/rJ4fpK_Xye.png)
        - Chế độ ECB có nhiệm vụ mã hoá điện tử các tin nhắn dưới dạng văn bản gốc có cách hoạt động đơn giản nhất trong Block Cipher. Chế độ này không yêu cầu thêm bất kì yếu tố nào vào Stream Key. Nguyên nhân do đây là chế độ duy nhất có khả năng mã hoá luồng một bit.
        - Mỗi Block dẽ được mã hoá độc lập với tất cả các Block khác. Cụ thể đó là mỗi một Block sẽ có một lý hiệu chữ cái riêng được chuyển thành ký hiệu bản mã bằng cách sử dụng khoá mật mã và bản chữ cái thay thế. 
    
    - **Chế độ Cipher Block Chaining (CBC):**
    Là phương pháp mã hoá dữ liệu nhằm đảm bảo mỗi văn bản gốc đều được gắn một Block bản mã. Thuật toán Symmetric Key tạo ra một bản mã hoạt động dựa vào các Block văn bản gốc được xử lý trước đố trong một luồng dữ liệu. Chế độ CBC được ứng dụng phổ biến trong các phần mềm bảo mật như Secure Sockets Layer và Transport Layer Security với mục đích để mã hoá dữ liệu được truyền qua Internet.
    
    - **Chế độ Ciphertext Feedback (CFB):**
        - Khác với CBC, chế độ CFB mã hoá các bit văn bản gốc ngay lập tức theo thứ tự. Tuy nhiên, cả hai chế độ này vẫn có đặc điểm chung đó là đều sử dụng IV. Ngoài ra CFB sử dụng Block bản mã giống như một yếu tố của trình tạo số ngẫu nhiên.
        - Trong chế độ CFB, Block bản mã trước đó được mã hoá và đầu ra được Exclusive OR với Block văn bản goocs để tạo một Block bản mã hiện tại. XOR này được sử dụng để ẩn các mẫu văn gốc.
    
    - **Chế độ Output Feedback (OFB):**
    Phù hợp với hầu hết các Block Cipher. Nó hoạt động bằng cách sử dụng cơ chế phản hồi để XOR với văn bản gốc sau khi được mã hoá. Điều này khác với việc XOR Block bản mã trước khi được mã hoá.

    - **Chế độ Counter (CTR):**
    Hoạt động bằng cách sử dụng chế độ Block Chaining cho Building Block. Quá trình mã hoá dữ liệu được thực hiện bằng cách XOR văn bản gốc với một chuỗi các giá trị ngẫu nhiên giả. Trong đó, mỗi giá trị đều được tái tạo dựa vào bản mã thông qua hàm Feedback. Quá trình mã hoá của chế độ CTR về cơ bản là một loạt các XOR giữa những Block băn bản gốc và Block bản mã tương ứng nhau.
    
    *d) Mã hoá xác thực với các chế độ dữ liệu bổ sung:*
    Các chế độ cung cấp hình thức mã hoá tin nhắn và các dữ liệu bổ sung (Chẳng hạn như số thứ tự hoặc tiêu đề không có sẵn trong bản mã):
    - **Chế độ Galois/Counter (GCM):** Bao gồm các Block được gắn IV và mã hoá bằng AES. Kết quả được XOR với văn bản gốc để tạo ra bản mã yêu cầu.
    - **Chế độ CCMP** tích hợp với AES có kích thước Block 188 bit và kích thước khoá 128 bit. Nó có khả năng xử lý tin nhắn có kích thước lên đến 16 byte. Ngoài ra, chế độ này còn được triển khai để giải quyết một số vấn đề về chế độ hoạt động cuaru CBC. Trong đó, cùng một Block văn bản gốc có thể được mã hoá thành nhiều bản mã khác nhau.
    - **Synthetic IV:** Là thuật toán AES mạng thay thế định hướng byte. Cụ thể, nó sẽ nhận một văn bản gốc và một khoá bảo mật để mã hoá chúng thành một bản mã hoàn chỉnh. Một đặc điểm khác của Synthetic IV đó là nó sử dụng Key Stream cố định được tạo từ trình tạo số ngẫu nhiên giả thay vì Key Stream ngẫu nhiên.
    - **AES - GCM - SIV:** Được tích hợp giữa Block Cipher AES, GCM và tính năng bảo mật khoá bổ sung của SIV. Nó có thể mã hoá nhiều tin nhắn hơn với một khoá.

2. Stream Cipher (Dòng mã hoá):
    *a) Định nghĩa:*
    - Là loại thuật toán mã hoá trong đó mỗi bit hoặc byte dữ liệu của văn bản rõ (plaintext) được mã hoá một cách độc lập, một lần duy nhất, thông qua chuỗi khoá (Stream Key). Stream Cipher thực hiện mã hoá bằng cách kết hợp (thường là XOR) từng bit của văn bản rõ với từng bit của chuỗi khoá. Khoá này phải được sinh ra một cách ngẫu nhiên hoặc từ một khoá bí mật ban đầu (key seed).
    - Một đặc điểm quan trọng của Stream Cipher là nó mã hoá dữ liệu từng phần một thay vì mã hoá toàn bộ khối dữ liệu như trong Block Cipher (mã hoá khối) $\Rightarrow$ Giúp Stream Cipher trở nên nhanh chóng và hiệu quả, đặc biệt khi xử lý dữ liệu theo dòng.
    - Hai đại diện tiêu biểu là máy tạo dòng khoá tuyến tính và máy chạy và dừng luân phiên.
    
    *b) Các đặc điểm:*
    - Tốc độ cao: Vì mã hoá từng bit hoặc byte nên có thể xử lý dữ liệu rất nhanh.
    - Tiết kiệm bộ nhớ: Không cần lưu trữ các khối dữ liệu lớn, chỉ cần lưu trữ chuỗi khoá.
    - Tính đơn giản: Thuật toán mã hoá đơn giản nhưng yêu cầy chuỗi khoá phải ngẫu nhiên và không bao giờ tái sử dụng (trong những trường hợp bảo mật cao).
    
    *c) Các vấn đề:*
    - Khoá phải an toàn: Nếu cùng một chuỗi khoá được sử dụng nhiều lần, hoặc nếu chuỗi khoá có yếu tố dự đoán được thì sự bảo mật sẽ bị đe doạ.
    - Độ dài của khoá: Khoá cần đủ dài để tránh bị lặp lại hoặc dự đoán.
    --> Thích hợp cho các tình huống yêu cầu tốc độ cao và khả năng xử lý trực tiếp dữ liệu theo dòng.
    
    *d) Stream Key (Dòng khoá):*
    - Dòng khoá mã hoá ${z_{i}}$ sẽ được sinh ra từ một khoá ban đầu K. Cần phải tạo dòng khoá {Zi} sao cho các bit của chúng không phụ thuộc lẫn nhau để tránh kẻ thám mã phát hiện ra quy luật. Nói cách khác, các ${z_{i}}$ phải được tạo ra hoàn toàn ngẫu nhiên. 
    - Sơ đồ quá trình tạo khoá và mã hoá có dạng:
    ![image](https://hackmd.io/_uploads/BJOlb-jmJe.png)
    - Quy trình tạo khoá trên thực tế có thể độc lập hoặc phụ thuộc vào kết quả mã hoá.
    
    *e) Thuật toán A5 (Tạo khoá tuyến tính):*
    Máy tạo dòng khoá tuyến tính (Linear Feedback Shift Registers) - LFSR-3 có sơ đồ như sau:
    ![image](https://hackmd.io/_uploads/HkfIWZjXJl.png)
    - Khởi tạo:
        - Có ba bit $z_{0}$, $z_{1}$, $z_{2}$ trong các hộp tương ứng. Máy hoạt động theo từng xung (clock). Tại mỗi xung, các bit sẽ dịch chuyển sang hộp bên phải.
        - Bit ở hộp $K_{0}$ sẽ xuất ra ngoài; Bit ở hộp $K_{1}$ sẽ chuyển sang hộp $K_{0}$; Bit ở hộp $K_{2}$ sẽ chuyển sang hộp $K_{1}$. Các bit ở hộp $K_{0}$ và $K_{1}$, theo sơ đồ, sẽ XOR với nhau và ghi kết quả vào hộp $K_{2}$. Quá trình này cứ chạy liên tục và ta sẽ nhận được dòng bit đầu ra ${z_{i}}$ để dùng làm khoá mã hoá.
        - Yêu cầu của dòng khoá mã hoá ${z_{i}}$ là các bit của chúng phải thật ngẫu nhiên với nhau. Nhưng trên thực tế điều đó khó có thể thực hiện. Khảo sát ví dụ của máy LFSR - 3 trên, với đầu vào là các bit $z_{0} = 1$, $z_{1} = 0$, $z_{2} = 0$. Kết quả chạy của máy sẽ có dạng như sau:
        ![image](https://hackmd.io/_uploads/ryt_GbimJg.png)
        Dòng bit khoá thu được là **1001011100101…**
        - Nếu gọi mỗi dòng của bảng là một trạng thái của máy thì tổng số trạng thái khác nhau có thể có là $2^3 = 8$.
        - Do đó, các trạng thái này sẽ lặp lại sau không quá 8 lần thực hiện (8 xung). Hơn nữa, việc chuyển từ một trạng thái này sang trạng thái khác là hoàn toàn xác định, vì vậy dòng bit khoá sẽ lặp lại.
        - Trong trường hợp cụ thể trên thì dòng bit khoá sẽ có dạng **1001011**. Số phần tử của dòng bit là 7 và được gọi là chu kỳ. Có thể thấy chuy kỳ này luôn không lớn hơn 8. Từ đó có thể nhận xét, nếu muốn các bit của dòng khoá càng ngẫu nhiên thì chu kỳ càng cần phải lớn, tương ứng với số hộp càng phải nhiều.
    - Máy LFSR - 3 này có thể biểu diễn như sau: 
        - $[3, 1 + x + x^{3}, (z_{0}, z_{1}, z_{2}) = (1, 0, 0)]$. Trong đó 3 là số hộp chứa các bit, biểu thức $1 + x + x^3$ là cấu trúc của máy và $(z_{0}, z_{1}, z_{2}) = (1, 0, 0)$ là giá trị các bit ban đầu. Biểu diễn ngắn gọn hơn: $[3, 1 + x + x^{3}, (1, 0, 0)]$.
        - Đối với dòng khoá sinh ra là ${z_{i}}$ thì $z_{i + 3} = z_{i + 1} + z_{i} \mod 2$, với i = 0, 1, 2,...
        - Máy tổng quát LFSR - 3 có công thức $[m, C_{0} + C_{1}.x + ... + C_{m-1}.x^{m - 1} + x^m, (z_{0}, z_{1},..., z_{m - 1})]$ và được biểu diễn như sau:
        ![image](https://hackmd.io/_uploads/SyA38WsmJl.png)
            - Với $z_{0}, z_{1},..., z_{m - 1}$ là các giá trị khởi tạo ban đầu.
            - $(C_{0}, C_{1},..., C_{m - 1}) Î {0, 1}$ là các hệ số phản hồi.
            - Nếu hệ số $C_{i} = 0$ thì mạch hởm ngược lại $C_{i} = 1$ thì mạch đóng.
            - Công thức tính bit $z_{i + m}$ với i = 0, 1, 2,...
            
    *f) Máy chạy và dừng luân phiên (Anternating stop - and - go generator):*
    - Để nâng cao độ an toàn của máy tạo dòng khoá, người ta kết hợp ba máy tạo dòng khoá tuyến tính tạo thành một máy chạy và dừng luân phiên: 
    ![image](https://hackmd.io/_uploads/S1islyy41l.png)
    - Máy chạy và dừng luân phiên G được cấy trúc từ ba máy tạo dòng tuyến tính A, B, C.
    - Các máy A, B, C tạo ra các dòng bit riêng lẻ không phụ thuộc vào nhau. Hoạt động của máy G dựa trên xung (clock).
    - Theo sơ đồ, nếu trong một xung nào đó máy A xuất ra 1 bit thì máy B sẽ làm việc và sinh ra but mới còn máy C thì không sinh bit.
    - Bit mới của máy B sẽ XOR với bit cũ của máy C để có bit kết quả xuất ra ngoài. Trường hợp ngược lại, trong một xung mà máy A xuất ra bit 0 thì bit cũ của máy B sẽ XOR với bit mới của máy C để tạo thành bit của luồng khoá.
    - Như vậy, máy A có chức năng điều khiển hoạt động của máy B và C. Còn các bit kết quả sẽ do các bit của máy B và C XOR lại với nhau mà tạo thành.
    - Giả sử đầu ra của các máy tương ứng là A: ${a_{0}, a_{1},...}$; ${b_{0}, b_{1},...}$ và ${c_{0}, c_{1},...}$; dòng khoá nhận được là Z: ${z_{0}, z_{1},...}$. Đặt $b_{-1} = c_{-1} = 0$ và giả sử dòng bit máy A có giá trị {100100110} thì các bit của dòng khoá Z sẽ được tính như trong hình sau:
    ![image](https://hackmd.io/_uploads/Sy0pGJJEkl.png)
    - Ở đây $z_{0} = b_{0} + c_{-1}$, $z_{1} = b_{0} + c_{0}$,...
    - Đặt $t(n) =$ (**???**). Ta thấy $t(n)$ chính là số bit một trong dãy ${a_{0}, a_{1},..., a_{n}}$ đó cũng đồng thời là số lần B chạy. Do đó, tại xung thứ n + 1, máy B sẽ xuất ra bit $b_{t(n) - 1}$.
    - Tương tự, $n + 1 - t(n)$ chính là số bit 0 trong dãy trên. Do đó, tại xung thứ n + 1, máy C sẽ xuất xa bit $c_{n - t(n)}$.
    - Như vậy, ta có công thức tính bit thứ n + 1 của dòng khoá: $z_{n} = b_{t(n) - 1} + c_{n - t(n)}$ với $t(n) =$ (**???**).
    
    *g) Thuật toán RC4 (Rivest Cipher 4):*
    - Trong mật mã học, RC4 (ARC4 hay ARCFOUR) là mật mã dòng. Ưu điểm của RC4 chính là đơn giản, dễ hiện thực trong phần mềm, tốc độ mã hoá cao. Tuy nhiên RC4 ngày nay được coi là hệ mã có nhiều lỗ hổng và không an toàn. 
    - RC4 là Stream Cipher với độ dài key không cố định (tuỳ người dùng) với đơn vị thao tác trên byte (8 bit). Dựa trên thuật toán hoán vị ngẫu nhiên với chu kỳ lặp lại của mã lên đến (**???**), mỗi byte của dữ liệu sẽ được thao tác từ 8 đến 16 toán tử để thành chuỗi mã hoá cho nên thời gian vận hành nhanh, dễ thực hiện trong các phần mềm.
    ![image](https://hackmd.io/_uploads/ry8DwJfE1g.png)
    - Để hiểu được thuật toán RC4, ta tìm hiểu 3 gia đoạn thực hiện chính gồm: 
        - **Khởi tạo S (Inititial Vector - IV):** Mọi phần tử của A (256 byte) được gán giá trị mặc định tăng dần từ 0 đến 255.
        Ví dụ: S[0] = 0; S[1] = 1; S[2] = 2, S[255] = 255.
        Bên cạnh đó ở giai đoạn này T cũng được tạo ra từ khoá bí mật với cách thức tạo khoá như sau:
            - Nếu khoá bí mật có độ lớn là 256 bytes thì T là khoá bí mật.
            - Nếu khoá bí mật có độ lớn nhỏ hơn 256 bytes thì khoá bí mật sẽ viết liền lặp lại cho đến khi đủ 256 bytes.
        - **Key - Scheduling Algorithm (KSA):** Vector S và vector T sau khi được khởi tạo thì được đưa vào hàm hoán vị KSA với hàm KSA được định nghĩa như sau:
            - Tiến hành lặp với mỗi giá trị i có giá trị từ 0 đến 255, Mỗi vòng lặp thực hiện hai bước:
                - Gán 4j = (j+ S[i] + T[j]) \mod 256$.
                - Hoán vị S[i] và S[j].
            - Sau khi vector S hoán vị xong sẽ tiếp tục đưa vào giai đoạn sau.
        - **Pseudo - Random Generation Algorithm (PRGA):** S được đưa vào giai đoạn PRGA để phát sinh thành dòng key với kích thước bất kỳ với hàm PRGA được định nghĩa như sau: 
            - Tiến hành lặp cho đến khi dừng sinh key.
            - Mỗi vòng lặp thực hiện các bước:
                - Gán $i = (i + 1) \mod 256$.
                - Gán $j = (j + S[i]) \mod 256$.
                - Hoán vị S[i] và S[j].
                - Gán $t = (S[i] + S[j]) \mod 256$.
                - Xuất $k = S[t]$.
                - Tiếp tục vòng lặp cho đến khi dừng sinh key.
            - Mỗi khoá k được XOR với byte tiếp theo của plaintext để tạo thành ciphertext (để giải mã ta XOR byte tiếp theo của cipher text với khoá k).
    - Sự an toàn của hệ mã RC4 phụ thuộc vào sự an toàn của hai giai đoạn KSA và PRGA. Rất nhiều nghiên cứu để làm cho RC4 an toàn hơn và có khả năng chống lại các cuộc tấn công. Ngày nay, RC4 được khuyến cáo không nên sử dụng vì nhiều lỗ hổng trong KSA và PRGA.
    - Một số thuật toán sinh ra để cải thiện RC4 như Spritz, RC4A, VMPC, RC4+,...

    *h) One - time pad(OTP):*
    - Là mật mã không thể phá vỡ về mặt lý thuyết. Tuy nhiên trong thực tế, đây là mật mã khó có khả nặng sử dụng được vì nó yêu cầu khoá bí mật chia sẻ trước ít nhất có cùng độ dài với tin nhắn. Tạo các khoá thực sự ngẫu nhiên và chia sẻ khoá bí mật một cách an toàn là vấn đề khó trong thực tế.
    - Thuật toán OTP là một mật mã Vigenère thoả mãn các điều kiện:
        - Khoá bí mật có độ lớn bằng đúng độ lớn của dữ liệu khoá.
        - Khoá bí mật được tạo một cách ngẫu nhiên thực sự.
        - Khoá chỉ sử dụng một lần và không sử dụng cho lần mã hoá khác.
        - Khoá bí mật phải được giữ bí mật.
    - Năm 1919, một phiên bản khác của OTP mã hoá Vernam được phát minh và sử dụng phép toán XOR thay vì cộng modulo như thuật toán Vigenère.

    *i) Các câu hỏi phổ biến:* 
    - Stream Cipher là gì?
    - Stream Cipher khác với Block Cipher như thế nào?
    - Stream Cipher được sử dụng trong các ứng dụng nào?
    - StreaM Cipher có độ bảo mật như thế nào?
    - Stream Cipher khác với One - Time Pad như thế nào?
    - Stream Cipher có thể được sử dụng để thực hiện mã hoá và giải mã thông tin như thế nào?
    - Stream Cipher có được sử dụng phổ biến trong các giao thức bảo mật như TLS và SSL không?
    - Stream Cipher có ảnh hưởng đến tốc độ xử lý dữ liệu không?
    - Có những thuật toán Stream Cipher nào được sử dụng phổ biến nhất hiện nay?
    
VII. Hash Function:
1. Định nghĩa:
    - Là thuật toán mã hoá nhận một đầu bào bất kỳ (mỗi chuỗi dữ liệu bất kỳ như văn bản, số hoặc dữ liêu phức tạp) và tạo ra ra một chuỗi ký tự với độ dài cố định, thường được gọi là "hash" "dighest".
    - Đặc điểm nối bật:
        - Đầu vào và đầu ra nhất quán: Với cùng một đầu vào, hàm băm sẽ luôn tạo ra cùng một chuỗi đầu ra.
        - Tính chất một chiều: Không thể quay ngược từ chuỗi đầu ra để suy ra dữ liệu ban đầu, tứ là không thể giải mã.
        - Dễ dàng tính toán: Có thể nhanh chóng tạo ra chuỗi hash từ bất kỳ đầu vào nào.
        - Thay đổi nhỏ, kết quả khác: Một thay đổi nhỏ trong dữ liệu đầu vào sẽ tạo ra một chuỗi băm hoàn toàn khác biệt, tính chất này được gọi là "Hiệu ứng lở tuyết" - "Avalanche effect".
    - Thường được sử dụng trong thị trường Crypto nhằm bảo vệ thông tin người dùng, tăng cường bảo mật cho giao dịnh và ngăn chặn các thao tác sửa đổi dữ liệu.

3. Ứng dụng:
    Không chỉ là một công cụ bảo mật mà còn là nền tảng cho rất nhiều hệ thống blockchain và tiền mã hoá.
    - Xác minh tính toàn vẹn của giao dịch: Mỗi giao dịch blcokchain được mã hoá thành một chuỗi hash duy nhất $\Rightarrow$ Bất kỳ thay dổi nào trong giao dịch cũng sẽ làm thay đổi chuỗi hash, cho phép phát hiện sự giả mạo và đảm bảo rằng dữ liệu không bị thay đổi.
    - Giúp bảo mật các khối trong chuỗi blockchain: Trong các hệ thống như Bitcoin, mỗi khối chứa một chuỗi hash duy nhất đực liên kết với các giao dịch trong khối. Khi một khối mới được tạo ra, nó sẽ chứa chuỗi hash của khối trước đó, tạo thành một chuỗi khối liên kết. Nhờ vậy, nếu một hacker muốn thay đổi một khối, họ phải thay đổi tất cả các khối tiếp theo và điều này rất khó thực hiện.
    - Hash function cũng là yếu tố quan trọng trong quá trình khai thác tiền mã hoá. Các miner phải giải các bài toán hash để tìm ra khối mới và nhận phần thưởng. Bài toán này thường yêu cầu giải một chuỗi hash có một số tính chất đặc biệt, làm cho quá trình khai thác trở thành một công việc tốn tài nguyên nhưng cần thiết để duy trì an toàn cho mạng lưới.
    - Hash function còn bảo vệ thông tin cá nhân người dùng. Khi người dùng tạo một ví tiền mã hoá, hash function mã hoá các private key của họ, giúp bảo vệ ví khỏi bị truy cập trái phép.

5. Một số hash function phổ biến trong Crypto:
    *a) SHA - 256 (Secure Hash Algorithm 256 - bit):*
    Là thuật toán phổ biến nhất được sử dụng trong Bitcoin và nhiều blockchain khác. SHA - 256 tạo ra một chuỗi 256 bit từ dữ liệu bất kỳ và có độ bảo mật cao với khoảng $2^{256}$ khả năng kết hợp. Điều này làm cho việc crute force giải mã các chuỗi hash SHA - 256 gần như không thể thực hiện được. SHA - 256 cũng đóng vai trò quan trọng trong cơ chế PoW của Bitcoin, giúp ngăn chặn các cuộc tấn công và duy trì tính phi tập trung của mạng lưới.
    ![image](https://hackmd.io/_uploads/B13Mf_4E1e.png)

    *b) Ethash:*
    Là thuật toán hash được dùng trong Ethereum trước khi chuyển sang Ethereum 2.0 và Proof of Stake (PoS). Ethash yêu cầu bộ nhớ lớn, giúp ngăn chạn ASIC mining và khuyến khích sử dụng GPU mining. Nhờ vào đó, Ethereum duy trì được tính phi tập trung vì người dùng phổ thông có thể tham gia khai thác.

    *c) Scrypt:*
    Được sử dụng trong Litecoin và một số loại tiền mã hoá khác. Scrypt ít tiêu tốn tài nguyên hơn SHA - 256, cho phép khai thác dễ dàng bằng CPU hoặc GPU, là lựa chọn phổ biến cho các miner nhỏ lẻ và không cần thiết bị khai thác đặc biệt.

    *d) Keccak - 256:*
Là thuật toán được Ethereum sử dụng sau khi chuyển sang PoS. Keccak - 256 là một phiên bản cải tiến từ SHA - 3, tối ưu hoá tính bảo mật và hiệu suất cho các hệ thống blockhain hiện đại.
