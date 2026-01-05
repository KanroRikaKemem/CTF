## A. Lý thuyết:
### I. AES:
#### 1. Định nghĩa:
- Advanced Encryption Standard (AES) hay còn gọi là tiêu chuẩn mã hoá nâng cao theo phương pháp mật mã khối. Đây là một thuật toán mã hoá đối xứng được sử dụng để bảo vệ dữ liệu trong các hệ thống máy tính và mạng.
- AES được sử dụng rộng rãi trong các ứng dụng bảo mật, bao gồm truyền dữ liệu an toàn qua Internet, bảo vệ dữ liệu trong các thiết bị lưu trữ như USB và ổ cứng di động và trong các ứng dụng mã hoá phần mềm và trò chơi điện tử.

#### 2. Cách hoạt động:
- Gồm ba mật mã khối AES - 128, AES - 192, AES - 256 tương ứng với độ dài của key là 128 bit, 192 bit và 256 bit. Số vòng của key khác nhau, cụ thể 10 vòng cho 128 bit, 12 vòng cho 192 bit và 14 vòng cho 256 bit. Mỗi vòng đều thực hiện ba bước thay thế, biến đổi và hoà trộn khối plaintext đầu vào để biến nó thành ciphertext. Nói đơn giản hơn là plaintext được chia ra thành các khối để đưa vào mã hoá từng khối, với độ dài cố định là 128 bit. Nếu độ dài plaintext nhiều hơn rất nhiều so với 128 bit thì ta sẽ chia nó thành các phần 128 bit.
![image](https://hackmd.io/_uploads/BkAEYoGwkx.png)
- Thông tin được chính phủ phân loại theo ba cấp độ: bảo mật, bí mật, tối mật. Tất cả các độ dài của key từ 128, 192, 256 bit đều được dùng ở cấp độ bảo mật, bí mật. Riêng với những thông tin tối mật để đảm bảo không xảy ra bất cứ sai sót nào phải cần đến key 192 hoặc 256 bit. Mật mã sẽ dùng một key riêng tư để mã hoá và giải mã dữ liệu và tất nhiên cả người gửi và người nhận đều phải nhận biết và sử dụng được key này.

#### 3. Tiêu chí cần có:
- **Bảo vệ:** Là một trong những tính năng hàng đầu AES cần phải có để đánh bại đối thủ khác.
- **Chi phí:** Mở ra nhiều cơ hội cho người dùng bằng cách phát hành trên toàn cầu và miễn phí bản quyền.
- **Khả năng thực hiện:** Linh hoạt, phù hợp và đơn giản là ba yếu tố quan trọng.

#### 4. Sự khác biệt giữa AES - 128 và AES - 256:
- Mã hoá 256 bit khó đoán hơn rất nhiều so với key 128 bit, vì vậy nó rất mạnh mẽ và bảo vệ dữ liệu trước các cuộc tấn công brute - force. Tuy nhiên, key 256 bit đòi hỏi sức mạnh xử lý cao hơn key 128 bit và thời gian thực thi cũng lâu hơn, nếu với các thiết bị nhỏ hoặc nguồn điện tải trọng thấp thì nên cân nhắc sử dụng.

#### 5. Sự khác biệt giữa AES và RSA, AES và DES:
##### a) AES và RSA:
- AES được sử dụng để bảo vệ dữ liệu ở trạng thái nghỉ (dữ liệu lưu trữ trong kho). RSA được sử dụng để kết nối các website trong các trình duyệt và nhiều lĩnh vực khác.
- Trong khi AES sử dụng key riêng tư thì RSA dùng key công khai. Bên cạnh đó hiệu suất hoạt động của RSA chậm hơn so với AES. Giải pháp tốt nhất để bảo vệ truyền dữ liệu ở khoảng cách xa là kết hợp cả hai.

##### b) AES và DES:
![image](https://hackmd.io/_uploads/BJIsniMPkx.png)
Cả AES và DES đều là mật mã khối đối xứng nhưng AES hiệu quả hơn bởi độ dài key mà nó sở hữu. Mã hoá AES cũng nhanh hơn DES.

#### 6. Ưu và nhược điểm:
##### a) Ưu điểm:
- Tính bảo mật cao.
- Tốc độ nhanh.
- Linh hoạt.
- Chuẩn quốc tế.
##### b) Nhược điểm:
- Phụ thuộc vào độ dài khoá để đảm bảo tính bảo mật.
- Không thể đảm bảo tính bảo mật nếu hệ thống bị tấn công bởi các phương thức khác như tấn công mạng, tấn công từ chối dịch vụ (DoS), tấn công đánh cắp thông tin đăng nhập,...
- Việc triển khai có thể đòi hỏi chi phí phần cứng và phần mềm khá lớn để bảo vệ dữ liệu, đặc biệt là trong các hệ thống lớn và phức tạp.

### II. Block cipher modes of operation:
#### 1. Định nghĩa:
- Bản thân các mã hoá khối như AES chỉ làm việc hiệu quả với từng khối (block) nên ta phải có cách thức hay thuật toán nào đó để có thể làm việc với nhiều khối khác nhau, do đó ta có **Block cipher mode of operation**.
- **Block cipher mode of operation** diễn tả cách thức lặp đi lặp lại của một thuật toán mã hoá khối để có thể mã hoá một chuỗi các khối hiệu quả và bảo mật.

#### 2. Các Block cipher mode phổ biến để mã hoá dữ liệu:
![image](https://hackmd.io/_uploads/SkJgxnfv1l.png)
##### a) Electronic codeblock (ECB):
- Là chế độ đơn giản nhất:
    - Plaintext được chia thành từng block $16$ byte.
    - Mỗi block được mã hóa riêng lẻ bằng cùng một khóa mà không có liên kết giữa các block.
- Công thức mã hoá: $$C_i = E_k(P_i)$$
- Công thức giải mã: $$P_i = D_k(C_i)$$
- **Ưu điểm:** Nhanh, có thể mã hoá song song (Encryption parallelizable), giải mã song song (Decryption parallelizable) và có thể truy cập bất cứ block nào tuỳ ý (Random read access).
- **Nhược điểm:** 
    - **Không có sự ngẫu nhiên:** Nếu một block plaintext lặp lại, ciphertext cũng sẽ lặp lại.
    - **Không bảo mật trước các mẫu dữ liệu:** Nếu dữ liệu có nhiều phần giống nhau, ta có thể nhận diện được cấu trúc dữ liệu từ ciphertext.
![image](https://hackmd.io/_uploads/S1XFG2Mw1e.png)
![image](https://hackmd.io/_uploads/SkYKznGwke.png)

##### b) Cipher block chaining (CBC):
- CBC khắc phục điểm yếu của ECB bằng cách xâu chuỗi các block với nhau bằng một $IV$ (Vector khởi tạo: Một chuỗi ngẫu nhiên $16$ byte được thêm vào để đảm bảo tính ngẫu nhiên).
- CBC mã hoá từng khối theo công thức: $$C_i = AES_{Encrypt}(P_i \oplus C_{i - 1})$$ Với:
    - $C_0 = IV$
    - $C_1 = AES(P_1 \oplus IV)$
    - $C_2 = AES(P_2 \oplus C_1)$
    - ...
    - $P_i = AES_{Decrypt}(C_i) \oplus C_{i - 1}$
![image](https://hackmd.io/_uploads/H1h573GP1l.png)
![image](https://hackmd.io/_uploads/S1Zjm2MP1g.png)
- CBC được dùng rất phổ biến trong mã hoá, nó có một hạn chế là phải mã hoá tuần tự và phải padding block cuối cùng.

##### c) Counter (CTR):
- Trong counter mode, ta sẽ mã hoá (nonce $||$ count) trước ($||$ là phép nối), sau đó sẽ cộng (XOR) với plaintext để ra ciphertext.
![image](https://hackmd.io/_uploads/HJhmrhfDJg.png)
![image](https://hackmd.io/_uploads/ry74H3MDkg.png)
- Nonce chính là một initialization vector (IV) như CBC nhưng chỉ được sử dụng một lần duy nhất, còn counter là một số tăng liên tiếp. Ví dụ trong AES thì đầu vào là 128 bit, ta thích một counter bằng 32 bit thì IV sẽ là 128 - 32 = 96 bit.
- Cách hoạt động:
Với:
    - $P = P_0 || P_1 || ... || P_n$: Plaintext chia thành các khối $128$ bit ($16$ byte).
    - $C = C_0 || C_1 || ... || C_n$: Ciphertext tương ứng.
    - $K$: Khoá bí mật AES.
    - $CTR_0 \in$ {$0$, $1$}$^{128}$: Bộ đếm ban đầu $IV$.

Với mỗi khối $i = 0, 1,..., n$:
- Mã hoá: $$CTR_i = CTR_0 + i (\mod 2^{128})$$ $$KS_i = AES_K(CTR_i) (keystream)$$ $$C_i = P_i \oplus KS_i$$
- Giải mã: $$P_i = C_i \oplus AES_K(CTR_0 = i)$$
- CTR vừa có ưu điểm nhanh, có thể Encryption parallelizable, Decryption parallelizable và Random read access. Đồng thời nó cũng khuếch tán dữ liệu, mỗi lần encrypt khác nhau là sử dụng một $IV$ khác nhau. 

### III. DES:
#### 1. Định nghĩa:
- Data Encryption Standard (DES) hay còn gọ là Tiêu chuẩn mã hoá dữ liệu bằng phương pháp khoá đối xứng.
- Đầu vào của DES là khối 64 bit, đầu ra cũng là khối 64 bit. Khoá mã hoá có độ dài 56 bit nhưng thực chất ban đầu là 64 bit, được lấy đi các bit ở vị trí chia hết cho 8 dùng để kiểm tra tính chẵn lẻ.

#### 2. Nguyên lý hoạt động:
![image](https://hackmd.io/_uploads/BJomoAXv1e.png)
- Để thực hiện thao tác và giải mã tin nhắn, DES sử dụng cùng một key riêng tư mà cả người nhận và người gửi đều nhận biết và sử dụng được. Một số tính năng cơ bản tác động tới cách thức hoạt động của DES:
    - **Key mật mã:** DES sử dụng phương pháp mật mã khối, đều này có nghĩa là mỗi khối dữ liệu sẽ được áp dụng bởi một key mật mã và thuật toán. DES sẽ nhóm plaintext thành các khối 64 bit. Bằng cách kết hợp và hoán vị các khối trên, plaintext sẽ được chuyển đổi thành ciphertext.
    - **Vòng mã hoá:** Dữ liệu sẽ được DES mã hoá 16 lần với bốn chế độ khác nhau. Từng khối riêng lẻ sẽ được mã hoá hoặc bắt buộc các khối mật mã phải phụ thuộc vào những khối trước đó. Riêng về giải mã thì đơn giản chỉ là nghịch đảo của mã hoá, tức là quy trình thực hiện tương tự nhưng đảo ngược thứ tự các key.
    - **Phím 64 bit:** Thực tế cho thấy mặc dù DES sử dụng key 64 bit nhưng có 8 bit trong số đó đã được dùng để kiểm tra chẵn lẻ, vì vậy key hiệu dụng chỉ có 56 bit.
    - **Thay thế và hoán vị:** Đây là hai quy trình mà ciphertext phải trải qua trong quá trình mã hoá.
    - **Khả năng tương thích ngược:** Trong một số trường hợp DES cũng cung cấp khả năng này.

#### 3. Chi tiết hơn về DES:
https://viblo.asia/p/encryption-des-Qpmleq27lrd#_1-tong-quan-ve-des-0

#### 4. Một số biến thể của DES:
##### a) 2DES:
![image](https://hackmd.io/_uploads/H1WDaln2Je.png)
- Mã hoá: $C = DES_{K2}\left\lfloor DES_{K1}(M) \right\rfloor$
- Giải mã: $M = DES_{K1}^{-1}\left\lfloor DES_{K2}^{-1}(C) \right\rfloor$

Mặc dù có $2^{56}$ sự lựa chọn cho khoá $K_1$ và $2^{56}$ sự lựa chọn cho khoá $K_2$ (dẫn tới có $2^{112} sự lựa chọn cho cặp khoá $(K_1, K_2)$) nhưng sức mạnh của DES bội hai không lớn như vậy.

##### b) 3DES (TDES):
- DES bội hai có thể bị tấn công bằng cách thám mã từ hai phía theo đề xuất của Diffie - Hellman. Để khắc phục yếu điểm này người ta đã xây dựng TDES với hai khoá $K_1$, $K_2$ như sau:
    - Mã hoá: $C = DES_{K1} (DES_{K2}^{-1} \left\lfloor DES_{K1}(M) \right\rfloor)$
    - Giải mã: $M = DES_{K1}^{-1} (DES_{K2}\left\lfloor DES_{K2}^{-1}(C) \right\rfloor)$
-  **Cách hoạt động của 3DES và Weak Key trong 3DES:**
    - 3DES hoạt động bằng cách mã hoá ba lần với DES theo công thức: $$C = E_{K1}(D_{K2}(E_{K3}(P)))$$ Với:
        - $E_K(P)$: Mã hoá DES với khoá $K$.
        - $D_K(P)$: Giải mã DES với khoá $K$
    - Weak Key là những khoá đặc biệt khiến 3DES trở nên kém an toàn. Một số loại khoá yếu:
        - **Khoá lặp (Repeated Key):**
            - Nếu $K1 = K2 = K3$, 3DES thực tế chỉ là DES (không tăng độ an toàn).
            - Nếu $K1 = K3$, 3DES hoạt động như một biến thể của DES với chỉ một vòng mã hoá thật sự.
        - **Khoá yếu của DES:** Mã hoá hai lần sẽ ra chính nó, giúp rút gọn số lượng phép toán cần brute - force: $$E_K(E_K(P)) = P$$

### IV. PKCS#7 Padding:
#### 1. Mô tả vấn đề:
- Mã hoá khối chuyển đổi một block có kích thước cố định (thường là $8$ hoặc $16$ byte) của plaintext thành ciphertext. Hầu như không bao giờ ta muốn chuyển đổi một block duy nhất mà ta mã hoá các thông điệp có kích thước không đều.
- Một cách khác để tính toán các thông điệp có kích thước không đều là padding, tạo ra một plaintext là bội số chẵn của kích thước block, phổ biến nhất là PKCS#7 Padding.
- Ví dụ: Thêm vào bất kỳ block nào một số byte đệm vào cuối khối: `YELLOW SUBMARINE` $\rightarrow$ `YELLOW SUBMARINE\x04\x04\x04\x04`

#### 2. Giải pháp:
- Khởi tạo một bài kiểu tra đơn vị dựa trên dữ liệu đầu vào và đầu ra từ mô tả vấn đề:
```py
import unittest
from Crypto.Util.Padding import pad

def pkcs7_padding(data: bytes, block_size: int) -> bytes:
    return pad(data, block_size)

class TestPKCS7Padding(unittest.TestCase):
    def test_padding_needed(self):
        data = b"YELLOW SUBMARINE"
        expected = b"YELLOW SUBMARINE\x04\x04\x04\x04"
        block_size = 20

        actual = pkcs7_padding(data, block_size)
        self.assertEqual(actual, expected, f"Expected {expected}, was {actual}")

    def test_no_padding_needed(self):
        data = b"YELLOW SUBMARINE"
        block_size = 16

        actual = pkcs7_padding(data, block_size)
        self.assertEqual(actual, data, f"Expected {data}, was {actual}")

if __name__ == "__main__":
    unittest.main()
```
- Ta đang sử dụng `YELLOW SUBMARINE` và kích thước block là $20$ làm tham số input cho hàm. Output sau đó được xác nhận với biến `expected` là `YELLOW SUBMARINE\x04\x04\x04\x04`. Ngoài ra còn có đoạn kiểm tra khi nào dữ liệu có kích thước có thể chia đều cho kích thước block.
- Việc triển khai PKCS#7 Padding chức năng có thể như: 
```py
import unittest
from Crypto.Util.Padding import pad

def pkcs7_padding(data: bytes, block_size: int) -> bytes:
    rest = len(data) % block_size
    if rest == 0:
        return data
    
    padding_size = block_size - rest
    padded = data + bytes([padding_size] * padding_size)
    return padded

class TestPKCS7Padding(unittest.TestCase):
    def test_padding_needed(self):
        data = b"YELLOW SUBMARINE"
        expected = b"YELLOW SUBMARINE\x04\x04\x04\x04"
        block_size = 20

        actual = pkcs7_padding(data, block_size)
        self.assertEqual(actual, expected, f"Expected {expected}, was {actual}")

    def test_no_padding_needed(self):
        data = b"YELLOW SUBMARINE"
        block_size = 16

        actual = pkcs7_padding(data, block_size)
        self.assertEqual(actual, data, f"Expected {data}, was {actual}")

if __name__ == "__main__":
    unittest.main()
```
- Tính toán kích thước padding bằng cách sử dụng modulo để xem dữ liệu có thể được chia đều cho kích thước block hay không, nếu không có thì `rest` và không cần phải padding dữ liệu, ngược lại thì tính toán số byte cần padding để hoàn thành block cuối cùng.
- Vòng lặp `for` thực hiện việc padding, trong đó mỗi byte được thêm vào để làm padding được đặt thành số byte được thêm vào, nghĩa là giá trị của mỗi byte padding giống với biến `paddingSize`.

## B. GIẢI THÍCH TẤN CÔNG:
### I. ECB Oracle:
#### 1. Oracle Attack:
- Trong ngữ cảnh bảo mật, Oracle là một hệ thống hoặc dịch vụ cung cấp thông tin phản hồi về kết quả của một thao tác mã hoá hoặc giải mã mà không tiết lộ trực tiếp khoá bí mật.
- Trong tấn công AES ECB Oracle, tin tặc khai thác thông tin phản hồi từ Oracle để từng bước giải mã hoặc mã hoá dữ liệu.

#### 2. Cách thức hoạt động của tấn công AES ECB Oracle:
- **Bước 1:** Để phát hiện kích thước khối mã hoá đầu tiên, cần xác định kích thước khối mã hoá của AES. Bằng cách gửi các chuỗi có độ dài tăng dần và quan sát sự thay đổi trong chuỗi ciphertext trả về, ta có thể phát hiện kích thước khối là bao nhiêu (thường là $16$ bytes - $128$ bits, đây là đặc điểm của AES).
- **Bước 2:** Để xây dựng oracle kế tiếp, ta lợi dụng việc biết trước một phần plaintext để xây dưng một oracle. Giả sử đã biết một phần của plaintext là `KnownTest`, ta có thể gửi chuỗi `P = "KnownTest" + "A"*(16 - len("KnownTest"))` đến dịch vụ mã hoá. Kết quả trả về sẽ là ciphertext của khối đầu tiên chứa `"KnownTest" + "A"*(16 - len("KnownTest"))`.
- Bước 3: Để tấn công từng byte tiếp theo, ta thử từng ký tự có thể có ($256$ ký tự từ `0x00` đến `0xFF`) vào vị trí tiếp theo của `KnownTest`. Giả sử `"KnownTest" = "KnownTest" + chr(0x00)`, `"KnownTest" = "KnownTest" + chr(0x01)`,... cho đến khi tìm được ký tự mà khi mã hoá, khối đầu tiên của ciphertext trùng khớp với ciphertext của chuỗi ban đầu. Khi đó ta xác định được ký tự tiếp theo của plaintext. Bằng cách lặp lại quá trình này, ta có thể từng bước giải mã toàn bộ dữ liệu bí mật.

#### 3. Ví dụ thực tế (CTF):
##### Mô tả bài toán:
- Có một dịch vụ web cho phép mã hoá một chuỗi bất kì bằng AES ECB với một khoá bí mật không được tiết lộ. Dịch vụ này cung cấp kết quả là chuỗi ciphertext tương ứng. Nhiệm vụ là giải mã được một đoạn dữ liệu bí mật (Flag) mà dịch vụ này mã hoá.
- Mã nguồn server:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import string
import random
from binascii import hexlify, unhexlify


def get_key():
    list = string.ascii_letters + string.digits
    key = random.choices(list, k=16)
    key = "".join(key)
    return key


def encrypt(data: str, key: str):
    e = AES.new(key.encode(), AES.MODE_ECB)
    return hexlify(e.encrypt(pad(data.encode(), 16))).decode()


flag = open("flag.txt", "r").read().strip()
secret_key = get_key()
flag = encrypt(flag, secret_key)
print("Flag encrypt: " + flag)
try:
    for i in range(1100):
        data = input("Message you want to encode: ")
        data += secret_key
        print("Cipher: " + encrypt(data, secret_key))
except Exception as e:
    print("Something error!")
    exit(0)
```

##### Phân tích bài toán:
- `get_key()`: Trả về một string có độ dài $16$ ký tự được lấy ngẫu nhiên từ các chữ cái, chữ số.
- `encrypt(data, key)`: Thực hiện mã hoá dữ liệu đầu vào `data` với khoá `key` bằng thuật toán `ECB` mode `ECB`. Dữ liệu trước khi mã hoá sẽ được cộng thêm chuỗi padding để đảm bảo độ dài dữ liệu là bội số của $16$. Kết quả trả về dưới dạng hex.
- Server thực hiện mã hoá `flag` sau đó trả về phía người dùng bản mã của flag sau khi mã hoá.
- Server nhận input từ người dùng, sau đó nối chuỗi đầu bào từ người dùng với `secret_key`. Tiếp theo sẽ thực hiện mã hoá sau khi đã nối chuỗi và trả kết quả về phía người dùng. Tối đa người dùng có thể mã hoa $1100$ lần với mỗi lần kết nối.

##### Quá trình exploit:
- **Xác định kích thước khối:**
Dựa vào độ dài khoá là $16$ byte từ mã nguồn, ta có thể biết kích thước khối mã hoá là $16$ byte hoặc có thể thử như sau:
![image](https://hackmd.io/_uploads/Hkny_Wh3kg.png)
    - Gửi lần lượt các chuỗi có độ dài từ $1$ đến $15$ byte, nhận thấy dữ liệu trả về có độ dài $32$ byte.
    - Gửi một chuỗi có độ dài $16$ byte, lúc này dữ liệu trả về có độ dài là $48$ byte.
    $\rightarrow$ Độ dài khối mã hoá là $48 - 32 = 16$ byte.
- **Brute force từng byte tại từng vị trí:**
Ban đầu `secret_key = ""`
    - Xác định giá trị byte đầu tiên của `secret_key`: 
        - Tạo chuỗi có độ dài $(16 - 1)$ + `secret_key`, ví dụ `aaaaaaaaaaaaaaa` ($15$ byte). Gửi chuỗi này lên server, chuỗi sẽ được nối với `secret_key` trước khi mã hoá: `aaaaaaaaaaaaaaa` $+$ `secret_key[0][1][...][15]`.
        - Vì độ dài khối mã hoá là $16$ byte nên khối mã hoá đầu tiên sẽ là của bản rõ `aaaaaaaaaaaaaaa` $+$ `secret_key[0]`. Lưu lại khối $16$ byte đầu tiên trong ciphertext mà server trả về, tạm gọi là `X`.
        - Tiếp theo thực hiện nối chuỗi `aaaaaaaaaaaaaaa` ($15$ byte) với từng byte mà `secret_key[0]` có thể nhận (chữ cái, chữ số). Ta được các chuỗi như `aaaaaaaaaaaaaaa1`, `aaaaaaaaaaaaaaa2`, `aaaaaaaaaaaaaaaA`, `aaaaaaaaaaaaaaaB`,...
        - Gửi từng chuỗi lên server và so sánh khối $16$ byte đầu tiên trong ciphertext mà server trả về với khối `X`, nếu hai khối có cùng giá trị thì giá trị `secret_key[0]` chính là giá trị byte cuối cùng trong chuỗi đang brute force vì khi sử dụng mode `ECB` thì các plaintext giống nhau sẽ cho ra các ciphertext giống nhau.
    - Xác định giá trị byte đầu tiên của `secret_key`:
        - Tạo chuỗi có độ dài ($16 - 2$) + `secret_key`, ví dụ `aaaaaaaaaaaaaa` ($14$ byte). Gửi chuỗi này lên server, chuỗi sẽ được nối với `secret_key` trước khi mã hoá: `aaaaaaaaaaaaaaa` $+$ `secret_key[0][1][...][15]`.
        - Vì độ dài khối mã hoá là $16$ byte nên khối mã hoá đầu tiên sẽ là của bản rõ `aaaaaaaaaaaaaa` ($14$ byte) $+$ `secret_key[0][1]`, trong đó `secret_key[0]` đã biết. Lưu lại khối $16$ byte đầu tiên trong ciphertext mà server trả về, tạm gọi là `X`.
        - Tiếp theo thực hiện nối chuỗi `aaaaaaaaaaaaaaa` ($15$ byte) với từng byte mà `secret_key[0]` có thể nhận (chữ cái, chữ số). Ta được các chuỗi như `aaaaaaaaaaaaaa` $+$ `secret_key[0]1`, `aaaaaaaaaaaaaa` $+$ `secret_key[0]2`, `aaaaaaaaaaaaaa` $+$ `secret_key[0]A`, `aaaaaaaaaaaaaa` $+$ `secret_key[0]B`,... 
        - Gửi từng chuỗi lên server và so sánh khối $16$ byte đầu tiên trong ciphertext mà server trả về với khối `X`, nếu hai khối cùng giá trị thì giá trị `secret_key[1]` chính là giá trị byte cuối cùng trong chuỗi đang brute force vì khi sử dụng mode `ECB` thì các plaintext giống nhau sẽ cho ra các ciphertext giống nhau.
    - Lặp lại quá trình trên cho đến khi lấy được khoá `seccret_key` và thực hiện giải mã bản mã của flag.
- **Script exploit:**
```py
from pwn import *
import string
from tqdm import tqdm
from Crypto.Cipher import AES
from binascii import unhexlify
from Crypto.Util.Padding import unpad


def decrypt(cipher, key: str):
    d = AES.new(key.encode(), AES.MODE_ECB)
    return d.decrypt(unhexlify(cipher))


list_char_secret_key = string.ascii_letters + string.digits
r = remote("192.168.1.4", 8000) #thay đổi địa chỉ phù hợp
r.recvuntil(b"Flag encrypt: ")
flag_encrypt = r.recvline().decode().strip()
secret_key = ""

for i in tqdm(range(16)):
    payload = 'a' * (15 - i)
    r.recvuntil(b"encode: ")
    r.sendline(payload.encode())
    r.recvuntil(b"Cipher: ")
    cipher = r.recvline().decode().strip()
    for char in list_char_secret_key:
        new_payload = payload + secret_key + char
        r.recvuntil(b"encode: ")
        r.sendline(new_payload.encode())
        r.recvuntil(b"Cipher: ")
        new_cipher = r.recvline().decode().strip()
        if (new_cipher[:32] == cipher[:32]):
            secret_key += char
            break
    print("Secret key recover: " + secret_key)

print(unpad(decrypt(flag_encrypt, secret_key), 16))
```

#### 4. Kết luận:
Tấn công AES ECB Oracle cho thấy rõ ràng những nhược điểm của chế độ mã hoá ECB và tầm quan trọng của việc sử dụng các chế độ mã hoá an toàn hơn như CBC hay GCM. Hiểu rõ về các kỹ thuật tấn công này không chỉ giúp phòng tránh mà còn nâng cao khả năng thiết kế hệ thống bảo mật an toàn hơn.

### II. Meet in the middle (MITM) với 2DES:
#### 1. Một ví dụ chung về tấn công 2D - MITM:
- Trong tấn công MITM với 2DES, phương pháp này nhằm đạt được hai trạng thái trung gian trong quá trình mã hoá nhiều lần của plaintext.
- Minh hoạ của tấn công 2D - MITM:
![image](https://hackmd.io/_uploads/r138d7xaJe.png)
- Thuật toán 2D - MITM:
    - Tính: $$SubCipher_1 = ENC_{f_1}(k_{f_1}, P) \forall k_{f_1} \in K$$ và lưu từng giá trị $SubCipher_1$ cùng với khoá tương ứng $k_{f_1}$ vào tập hợp $A$.
    - Tính: $$SubCipher_2 = DEC_{b_2}(k_{b_2}, C) \forall k_{b_2} \in K$$ và lưu từng giá trị $SubCipher_2$ cùng với khoá tương ứng $k_{b_2}$ vào tập hợp $B$.
    - Với mỗi giả định về trạng thái trung gian $s$ giữa $SubCipher_1$ và $SubCipher_2$:
        - Tính: $$SubCipher_1 = DEC_{b_1}(k_{b_1}, s) \forall k_{b_1} \in K$$ và với mỗi giá trị khớp với $SubCipher_1$ trong tập $A$, lưu $k_{b_1}$ vào tập hợp $T$.
        - Tính: $$SubCipher_2 = ENC_{f_2}(k_{f_2}, s) \forall k_{b_2} \in K$$ và với mỗi giá trị khớp với $SubCipher_2$ trong tập $B$, kiểm tra xem nó có khớp với tập $T$ hay không. Nếu có, thực hiện kiểm tra tiếp theo.
    - Sử dụng tổ hợp khoá con $k_{f_1}, k_{b_1}, k_{f_2}, k_{b_2}$ trên một cặp plaintext - ciphertext khác để xác minh tính chính xác của khoá.

#### 2. Độ phức tạp của 2D - MITM:
- Độ phức tạp thời gian của tấn công này nếu không sử dụng brute force là: $$2^{|k_{f_1}|} + 2^{|k_{b_2}|} + 2^{|s|}.(2^{|k_{b_1}|} + 2^{|k_{f_2}|})$$ Trong đó $|.|$ biểu diễn độ dài của khoá.
- Giới hạn bộ nhớ chính bị ràng buộc bởi việc xây dựng các tập hợp $A$ và $B$, trong đó tập $T$ nhỏ hơn nhiều so với các tập còn lại.

### III. Padding Oracle Attack với CBC:
Trong mật mã đối xứng, tấn công Padding Oracle có thể được áp dụng vào CBC mode of operation. Dữ liệu bị rò rỉ về tính hợp lệ của phần padding có thể cho phép kẻ tấn công giải mã (và đôi khi mã hoá) các thông điệp thông qua oracle bằng cách sử dụng khoá của oracle mà không cần biết khoá mã hoá.

#### 1. Tấn công Padding Oracle với CBC:
- Triển khai tiêu chuẩn của quá trình giải mã CBC trong các mã khối là giải mã tất cả các khối bản mã, xác thực phần padding, loại bõ phần padding theo tiêu chuẩn PKCS7 và trả về plaintext của thông điệp. Nếu máy chủ trả về **"invalid padding"** thay vì lỗi chung chung **"decryption failed"**, kẻ tấn công có thể sử dụng máy chủ như một oracle để giải mã hoặc mã hoá thông điệp.
![image](https://hackmd.io/_uploads/SkmPb4gaJl.png)
- Công thức toán học của quá trình giải mã CBC: $$P_i = D_K(C_i) \oplus C_{i - 1}, i \in {1, N}$$ $$P_0 = IV \oplus D_K(C_0)$$
- Cách thực hiện tấn công: Giả sử kẻ tấn công có hai khối ciphertext $C_1$, $C_2$ và muốn giải mã khối thứ hai để thu được $P_2$:
    - Kẻ tấn công thay đổi byte cuối cùng của $C_1$ (tạo thành $C_1'$) và gửi $(IV, C_1', C_2)$ đến máy chủ.
    - Máy chủ kiểm tra phần padding của khối giải mã cuối cùng ($P_2'$) và phản hồi xem phần padding có hợp lệ hay không (theo PKCS#7).
        - Nếu phần padding hợp lệ, kẻ tấn công biết rằng byte cuối của $D_K(C_2) \oplus C_1'$ có giá trị là `0x01`.
        - Nếu byte cuối hợp lệ, kẻ tấn công có thể trực tiếp thay đổi byte kế cuối để đảm bảo rằng hai byte cuối có giá trị `0x02`, sau đó tiếp tục với các byte còn lại.
    - Kẻ tấn công có thể thử tối đa $256$ giá trị để xác định byte cuối cùng của $P_2$, nhưng thực tế chỉ cần $255$ giá trị do nguyên lý bồ câu.
    - Sau khi tìm ra byte cuối của $P_2$, kẻ tấn công tiếp tục sử dụng kỹ thuật tương tụ để tìm byte kế cuối bằng cách đặt byte cuối của $P_2$ thành `0x02`.
    - Độ phức tạp: Với khối $128$ bit (AES) (tức là $16$ byte), kẻ tấn công cần tối đa $256$ x $16 = 4096$ lần thử để giải mã toàn bộ $P_2$, điều này nhỏ hơn rất nhiều so với $2^{128}$ lần thử cần thiết nếu brute force khoá $128$ bit.

#### 2. Mã hoá thông điệp bằng tấn công Padding Oracle (CBC - R):
- CBC - R biến một oracle giải mã thành một oracle mã hoá và chủ yếu được khai thác dựa trên các padding oracle.
- Bằng cách sử dụng tấn công padding oracle, CBC - R có thể tạo ra vector khởi tạo ($IV$) và khối ciphertext cho bất kì plaintext nào:
    - Giải mã bất kì ciphertext nào: $$P_i = PODecrypt(C_i) \oplus C_{i - 1}$$
    - Chọn tuỳ ý khối ciphertext trước đó $C_{x - 1}$.
    - Tạo cặp ciphertext - plaintext hợp lệ: $$C_{x - 1} = P_x \oplus PODecrypt(C_i)$$
- Để tạo một ciphertext có $N$ khối, kẻ tấn công phải thực hiện $N$ lần tấn công padding oracle. Các cuộc tấn công này được xâu chuỗi lại sao cho bản rõ được xây dựng theo thứ tự ngược lại, từ cuối thông điệp ($C_n$) đến đầu thông điệp ($C_0$, $IV$). Ở mỗi bước, tấn công padding oracle được sử dụng để xây dựng $IV$ cho ciphertext đã chọn trước đó.
- Tấn công CBC - R sẽ không có tác dụng với các lược đồ mã hoá xác thực ciphertext (chẳng hạn như sử dụng Message Authentication Code - MAC) trước khi giải mã.
