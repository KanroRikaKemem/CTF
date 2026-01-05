### I. HOW AES WORKS:
#### 1. Keyed Permutations:
##### a) Mô tả đề bài:
- AES, giống như tất cả các mật mã khối tốt, thực hiện "hoán vị có khóa". Điều này có nghĩa là nó ánh xạ mọi khối đầu vào có thể có vào một khối đầu ra duy nhất, với một khóa xác định hoán vị nào sẽ thực hiện.
- Một "khối" chỉ đề cập đến một số bit hoặc byte cố định, có thể đại diện cho bất kỳ loại dữ liệu nào. AES xử lý một khối và xuất ra một khối khác. Chúng ta sẽ nói cụ thể về biến thể của AES hoạt động trên các khối 128 bit (16 byte) và khóa 128 bit, được gọi là AES - 128.
- Sử dụng cùng một khóa, hoán vị có thể được thực hiện ngược lại, ánh xạ khối đầu ra trở lại khối đầu vào ban đầu. Điều quan trọng là phải có sự tương ứng một - một giữa các khối đầu vào và đầu ra, nếu không chúng ta sẽ không thể dựa vào bản mã để giải mã trở lại cùng bản rõ mà chúng ta đã bắt đầu.
- Thuật ngữ toán học cho sự tương ứng một - một là gì?
    
##### b) Phân tích cách làm:
- Thuật ngữ toán học cho sự tương ứng một  - một giữa hai tập hợp (mỗi phần tử của tập hợp này được **ánh xạ** đến phần tử duy nhất trong tập hợp kia) là **bijective**
- Trong ngữ cảnh của AES - 128, điều này có nghĩa là quá trình mã hoá (sử dụng khoá) tạo ra một mối quan hệ một - một giữa khối dữ liệu đầu vào (plaintext) và khối dữ liệu đầu ra (ciphertext), đảm bảo rằng mỗi đầu vào được ánh xạ đến một đầu ra duy nhất và có thể được đảo ngược để phục hồi lại dữ liệu gốc. 
    
##### c) Kết quả:
`crypto{bijection}`

#### 2. Resisting Bruteforce:
##### a) Mô tả đề bài:
- Đề bài giải thích về bảo mật của AES - 128 và những yếu tố liên quan đến các cuộc tấn công vào nó:
    - Một block cipher an toàn sẽ không có cách nào để kẻ tấn công phân biệt giữa đầu ra của AES và một hoán vị ngẫu nhiên.
    - Để "bẻ khoá" AES - 128, cách duy nhất là thử tất cả các khoá (brute - force).
    - Một cuộc tấn công trên AES - 128 đã được phát hiện, giảm mức độ bảo mật của AES - 128 xuống còn 126.1 bit, nhưng không có tiến bộ nào trong 8 năm qua.
    - AES - 128 vẫn được coi là an toàn vì sự giảm bảo mật này không đáng kể.
    - Mặc dù máy tính lượng tử có thể làm yếu đi các hệ thống mã khoá công khai (như RSA), chúng chỉ làm giảm một nửa bảo mật của các hệ thống mã đối xứng (như AES).
- Yêu cầu điền tên của cuộc tấn công tốt nhất hiện nay vào AES với một khoá duy nhất.
    
##### b) Phân tích cách làm:
Tên của cuộc tấn công đó là Biclique.

##### c) Kết quả:
`crypto{biclique}`

#### 3. Structure of AES:
##### a) Mô tả đề bài:
- Mô tả tổng quan về AES: AES - 128 sử dụng một khoá 128 bit để mã hoá một khối văn bản (state) 4x4 byte. Quá trình mã hoá gồm 10 vòng, bao gồm các bước biến đổi ngược để có thể đảo ngược được
- Các bước mã hoá trong AES:
    ![image](https://hackmd.io/_uploads/Sk6gJrZT1e.png)
    - **KeyExpansion:** Từ khoá 128 bit ban đầu, tạo ra 11 khoá vòng (round keys) 128 bit, mỗi vòng sử dụng một khoas vòng khác nhau.
    - **AddRoundKey:** XOR khoá vòng đầu tiên vào với trạng thái (state).
    - **Mười vòng chính (main rounds):**
        - **SubBytes:** Thay thế mỗi byte trong trạng thái bằng một byte khác dựa trên bảng tra cứu (S - box).
        - **ShiftRows:** Di chuyển các dòng trong ma trận trạng thái.
        - **MixColumns:** Thực hiện phép nhân ma trận trên các cột trong trạng thái (bước này bị bỏ qua ở vòng cuối).
        - **AddRoundKey:** XOR khoá vòng hiện tại với trạng thái.
    - **Final round:** Giống với vòng chính nhưng không có bước **MixColumns**.
- Nhiệm vụ:
    - Ta được cung cấp một hàm `bytes2matrix` để chuyển văn bản ban đầu thành ma trận trạng thái.
    - Cần viết hàm `matrix2bytes` để chuyển ma trận trạng thái trở lại thành các byte và giải mã plaintext từ đó.

##### b) Phân tích cách làm:
```py
def bytes2matrix(text):
    """ Converts a 16-byte array into a 4x4 matrix.  """
    return [list(text[i:i+4]) for i in range(0, len(text), 4)]

def matrix2bytes(matrix):
    """ Converts a 4x4 matrix into a 16-byte array.  """
    return [byte for row in matrix for byte in row]

matrix = [
    [99, 114, 121, 112],
    [116, 111, 123, 105],
    [110, 109, 97, 116],
    [114, 105, 120, 125],
]

byte_array = matrix2bytes(matrix)
res = bytes(byte_array).decode()
print(res)
```
- Ta triển khai hàm `matrix2bytes` bằng cách chuyển đổi ma trận 4x4 (được biểu diễn dưới dạng danh sách trong Python) thành một chuỗi byte gồm 16 phần tử bằng cách duyệt từng hàng và nối các giá trị với nhau thành chuỗi byte.
`[99, 114, 121, 112, 116, 111, 123, 105, 110, 109, 97, 116, 114, 105, 120, 125]`
- Sau đó, dùng hàm `bytes()` để chuyển danh sách byte thành đối tượng byte và giải mã thành chuỗi ký tự và in ra màn hình.

##### c) Kết quả:
`crypto{inmatrix}`

#### 4. Round Keys:
##### a) Mô tả đề bài:
- Giới thiệu về KeyExpansion và AddRoundKey:
    ![image](https://hackmd.io/_uploads/H1gq3eBbayl.png)
    - Trong thuật toán AES, khoá ban đầu 16 byte sẽ được mở rộng để tạo ra 11 khoá vòng (round keys), mỗi khoá vòng là một ma trận 4x4.
    - Trong bước **AddRoundKey**, mỗi khoá vòng sẽ XOR trạng thái (state) với khoá vòng hiện tại và đây là cách mà khoá được trộn vào trạng thái trong AES.
- Yêu cầu:
    - Hoàn thành hàm `add_round_key`, một phần trong bước mã hoá AES bằng cách thực hiện phép XOR giữa trạng thái và khoá vòng trong mỗi bước.
    - Sau khi hoàn thành hàm `add_round_key`, sử dụng hàm `matrix2bytes` để chuyển ma trận 4x4 đã xử lý thành một chuỗi byte, từ đó tìm ra flag.

##### b) Phân tích cách làm:
- Với cách làm sau ta sẽ ra được kết quả:
```python
state = [
    [206, 243, 61, 34],
    [171, 11, 93, 31],
    [16, 200, 91, 108],
    [150, 3, 194, 51],
]

round_key = [
    [173, 129, 68, 82],
    [223, 100, 38, 109],
    [32, 189, 53, 8],
    [253, 48, 187, 78],
]

def add_round_key(s, k):
    return [[s[i][j] ^ k[i][j] for j in range(4)] for i in range (4)]

def matrix2bytes(matrix):
    return [byte for row in matrix for byte in row]

byte_array = matrix2bytes(add_round_key(state, round_key))
res = bytes(byte_array).decode()
print(res)
```
- Ngoài ra còn có cách khác là XOR hai phần tử với nhau và in ra mã ASCII của kết quả.
```py
state = [
    [206, 243, 61, 34],
    [171, 11, 93, 31],
    [16, 200, 91, 108],
    [150, 3, 194, 51],
]

round_key = [
    [173, 129, 68, 82],
    [223, 100, 38, 109],
    [32, 189, 53, 8],
    [253, 48, 187, 78],
]

def add_round_key(s, k):
    for i in range(4):
        for j in range(4):
            print(chr(s[i][j] ^ k[i][j]), end = "")

print(add_round_key(state, round_key))
```

##### c) Kết quả:
`crypto{r0undk3y}`

#### 5. Confusion through Substitution:
##### a) Mô tả đề bài:
- Đề yêu cầu ta triển khai bước SubBytes trong thuật toán AES, trong đó mỗi byte của ma trận trạng thái sẽ được thay thế bằng một byte khác từ bảng tra cứu 16x16 (S - box). Mục đích của S - box là tạo ra sự "mơ hồ", làm cho mối quan hệ giữa ciphertext và khoá trở nên phức tạp. Các bước chính để thực hiện SubBytes gồm:
    - Lấy mỗi byte trong ma trận trạng thái.
    - Thực hiện thay thế byte đó bằng byte tương ứng từ S - box.
    - Sử dụng S - box ngược để thực hiện thay thế các giá trị byte theo phương pháp đã được chỉ định.
    ![image](https://hackmd.io/_uploads/SkWJbBbpye.png)
- S - box trong AES được thiết kế để có độ phi tuyến cao và kháng lại việc ướng lượng mỗi quan hệ tuyến tính giữa ciphertext, khoá và plaintext. Quá trình này là một phần của việc làm tăng độ bảo mật của thuật toán.
- Cuối cùng, sau khi thực hiện SubBytes, ta cần chuyển đổi kết quả thành byte để tìm flag.

##### b) Phân tích cách làm:
- Ta triển khai hàm `sub_bytes` bằng cách thay thế mỗi byte trong ma trận `state` bằng giá trị tương ứng trong bảng S - box hoặc S - box ngược.
- Bảng S - box là một bảng tra cứu 256 giá trị, trong đó mỗi giá trị từ `0x00` đến `0xFF` (0 đến 255) có một giá trị tương ứng. Ta tra cứu giá trị bằng `sbox[byte]`, ví dụ nếu giá trị của `byte` là `251`, ta sẽ tra cứu `sbox[251]` để thay thế byte này.
- Source code:
```py
s_box = (
    0x63, 0x7C, 0x77, 0x7B, 0xF2, 0x6B, 0x6F, 0xC5, 0x30, 0x01, 0x67, 0x2B, 0xFE, 0xD7, 0xAB, 0x76,
    0xCA, 0x82, 0xC9, 0x7D, 0xFA, 0x59, 0x47, 0xF0, 0xAD, 0xD4, 0xA2, 0xAF, 0x9C, 0xA4, 0x72, 0xC0,
    0xB7, 0xFD, 0x93, 0x26, 0x36, 0x3F, 0xF7, 0xCC, 0x34, 0xA5, 0xE5, 0xF1, 0x71, 0xD8, 0x31, 0x15,
    0x04, 0xC7, 0x23, 0xC3, 0x18, 0x96, 0x05, 0x9A, 0x07, 0x12, 0x80, 0xE2, 0xEB, 0x27, 0xB2, 0x75,
    0x09, 0x83, 0x2C, 0x1A, 0x1B, 0x6E, 0x5A, 0xA0, 0x52, 0x3B, 0xD6, 0xB3, 0x29, 0xE3, 0x2F, 0x84,
    0x53, 0xD1, 0x00, 0xED, 0x20, 0xFC, 0xB1, 0x5B, 0x6A, 0xCB, 0xBE, 0x39, 0x4A, 0x4C, 0x58, 0xCF,
    0xD0, 0xEF, 0xAA, 0xFB, 0x43, 0x4D, 0x33, 0x85, 0x45, 0xF9, 0x02, 0x7F, 0x50, 0x3C, 0x9F, 0xA8,
    0x51, 0xA3, 0x40, 0x8F, 0x92, 0x9D, 0x38, 0xF5, 0xBC, 0xB6, 0xDA, 0x21, 0x10, 0xFF, 0xF3, 0xD2,
    0xCD, 0x0C, 0x13, 0xEC, 0x5F, 0x97, 0x44, 0x17, 0xC4, 0xA7, 0x7E, 0x3D, 0x64, 0x5D, 0x19, 0x73,
    0x60, 0x81, 0x4F, 0xDC, 0x22, 0x2A, 0x90, 0x88, 0x46, 0xEE, 0xB8, 0x14, 0xDE, 0x5E, 0x0B, 0xDB,
    0xE0, 0x32, 0x3A, 0x0A, 0x49, 0x06, 0x24, 0x5C, 0xC2, 0xD3, 0xAC, 0x62, 0x91, 0x95, 0xE4, 0x79,
    0xE7, 0xC8, 0x37, 0x6D, 0x8D, 0xD5, 0x4E, 0xA9, 0x6C, 0x56, 0xF4, 0xEA, 0x65, 0x7A, 0xAE, 0x08,
    0xBA, 0x78, 0x25, 0x2E, 0x1C, 0xA6, 0xB4, 0xC6, 0xE8, 0xDD, 0x74, 0x1F, 0x4B, 0xBD, 0x8B, 0x8A,
    0x70, 0x3E, 0xB5, 0x66, 0x48, 0x03, 0xF6, 0x0E, 0x61, 0x35, 0x57, 0xB9, 0x86, 0xC1, 0x1D, 0x9E,
    0xE1, 0xF8, 0x98, 0x11, 0x69, 0xD9, 0x8E, 0x94, 0x9B, 0x1E, 0x87, 0xE9, 0xCE, 0x55, 0x28, 0xDF,
    0x8C, 0xA1, 0x89, 0x0D, 0xBF, 0xE6, 0x42, 0x68, 0x41, 0x99, 0x2D, 0x0F, 0xB0, 0x54, 0xBB, 0x16,
)

inv_s_box = (
    0x52, 0x09, 0x6A, 0xD5, 0x30, 0x36, 0xA5, 0x38, 0xBF, 0x40, 0xA3, 0x9E, 0x81, 0xF3, 0xD7, 0xFB,
    0x7C, 0xE3, 0x39, 0x82, 0x9B, 0x2F, 0xFF, 0x87, 0x34, 0x8E, 0x43, 0x44, 0xC4, 0xDE, 0xE9, 0xCB,
    0x54, 0x7B, 0x94, 0x32, 0xA6, 0xC2, 0x23, 0x3D, 0xEE, 0x4C, 0x95, 0x0B, 0x42, 0xFA, 0xC3, 0x4E,
    0x08, 0x2E, 0xA1, 0x66, 0x28, 0xD9, 0x24, 0xB2, 0x76, 0x5B, 0xA2, 0x49, 0x6D, 0x8B, 0xD1, 0x25,
    0x72, 0xF8, 0xF6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xD4, 0xA4, 0x5C, 0xCC, 0x5D, 0x65, 0xB6, 0x92,
    0x6C, 0x70, 0x48, 0x50, 0xFD, 0xED, 0xB9, 0xDA, 0x5E, 0x15, 0x46, 0x57, 0xA7, 0x8D, 0x9D, 0x84,
    0x90, 0xD8, 0xAB, 0x00, 0x8C, 0xBC, 0xD3, 0x0A, 0xF7, 0xE4, 0x58, 0x05, 0xB8, 0xB3, 0x45, 0x06,
    0xD0, 0x2C, 0x1E, 0x8F, 0xCA, 0x3F, 0x0F, 0x02, 0xC1, 0xAF, 0xBD, 0x03, 0x01, 0x13, 0x8A, 0x6B,
    0x3A, 0x91, 0x11, 0x41, 0x4F, 0x67, 0xDC, 0xEA, 0x97, 0xF2, 0xCF, 0xCE, 0xF0, 0xB4, 0xE6, 0x73,
    0x96, 0xAC, 0x74, 0x22, 0xE7, 0xAD, 0x35, 0x85, 0xE2, 0xF9, 0x37, 0xE8, 0x1C, 0x75, 0xDF, 0x6E,
    0x47, 0xF1, 0x1A, 0x71, 0x1D, 0x29, 0xC5, 0x89, 0x6F, 0xB7, 0x62, 0x0E, 0xAA, 0x18, 0xBE, 0x1B,
    0xFC, 0x56, 0x3E, 0x4B, 0xC6, 0xD2, 0x79, 0x20, 0x9A, 0xDB, 0xC0, 0xFE, 0x78, 0xCD, 0x5A, 0xF4,
    0x1F, 0xDD, 0xA8, 0x33, 0x88, 0x07, 0xC7, 0x31, 0xB1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xEC, 0x5F,
    0x60, 0x51, 0x7F, 0xA9, 0x19, 0xB5, 0x4A, 0x0D, 0x2D, 0xE5, 0x7A, 0x9F, 0x93, 0xC9, 0x9C, 0xEF,
    0xA0, 0xE0, 0x3B, 0x4D, 0xAE, 0x2A, 0xF5, 0xB0, 0xC8, 0xEB, 0xBB, 0x3C, 0x83, 0x53, 0x99, 0x61,
    0x17, 0x2B, 0x04, 0x7E, 0xBA, 0x77, 0xD6, 0x26, 0xE1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0C, 0x7D,
)

state = [
    [251, 64, 182, 81],
    [146, 168, 33, 80],
    [199, 159, 195, 24],
    [64, 80, 182, 255],
]

def sub_bytes(s, sbox = s_box):
    result = [[0]*len(row) for row in s]
    for i in range (len(s)):
        for j in range (len(s[i])):
            byte = s[i][j]
            result[i][j] = sbox[byte]
    return result

def matrix2bytes(matrix):
    return (byte for row in matrix for byte in row)
    
matrix = sub_bytes(state, sbox = inv_s_box)
byte_array = matrix2bytes(matrix)
res = bytes(byte_array).decode()
print(res)
```

##### c) Kết quả: 
`crypto{l1n34rly}`

#### 6. Diffusion through Permutation:
##### a) Mô tả đề bài:
- Đề bài yêu cầu triển khai hai bước trong thuật toán AES để giải mã ma trận `state` và tìm flag:
    - **ShiftRows:** Là một bước đơn giản trong AES, giữ nguyên hàng đầu tiên của ma trận, dịch các hàng còn lại theo một số cột nhất định. Hàng thứ hai dịch một cột, hàng thứ ba dịch hau cột, hàng thứ tư dịch ba cột.
    ![image](https://hackmd.io/_uploads/BkUb-H-6Je.png)
    - **MixColumns:** Một phép toán phức tạp hơn, thực hiện phép nhân ma trận trong trường Galois của Rijndael, nơi mỗi byte trong một cột sẽ ảnh hưởng đến tất cả các byte trong cột đó, tạo ra sự khuếch tán.
    ![image](https://hackmd.io/_uploads/S14Mbr-ayx.png)
- Nhiệm vụ của ta là triển khai các bước:
    - `inv_shift_rows`: Đảo ngược phép dịch của ShiftRows.
    - `inv_mix_columns`: Đảo ngược phép nhân ma trận MixColumn.
- Sau khi triển khai, ta sẽ áp dụng các bước đảo ngược này vào ma trận `state` và chuyển kết quả thành byte để lấy flag.

##### b) Phân tích cách làm:
- Source code đề cho yêu cầu ta bổ sung hàm `inv_shift_rows`, sau đó triển khai theo thứ tự lần lượt các bước `inv_mix_columns` và `inv_shift_rows`.
- Source code:
```py
def shift_rows(s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[1][1], s[2][1], s[3][1], s[0][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[3][3], s[0][3], s[1][3], s[2][3]

def inv_shift_rows(s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[3][1], s[0][1], s[1][1], s[2][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[1][3], s[2][3], s[3][3], s[0][3]

# learned from http://cs.ucsb.edu/~koc/cs178/projects/JT/aes.c
xtime = lambda a: (((a << 1) ^ 0x1B) & 0xFF) if (a & 0x80) else (a << 1)

def mix_single_column(a):
    # see Sec 4.1.2 in The Design of Rijndael
    t = a[0] ^ a[1] ^ a[2] ^ a[3]
    u = a[0]
    a[0] ^= t ^ xtime(a[0] ^ a[1])
    a[1] ^= t ^ xtime(a[1] ^ a[2])
    a[2] ^= t ^ xtime(a[2] ^ a[3])
    a[3] ^= t ^ xtime(a[3] ^ u)

def mix_columns(s):
    for i in range(4):
        mix_single_column(s[i])

def inv_mix_columns(s):
    # see Sec 4.1.3 in The Design of Rijndael
    for i in range(4):
        u = xtime(xtime(s[i][0] ^ s[i][2]))
        v = xtime(xtime(s[i][1] ^ s[i][3]))
        s[i][0] ^= u
        s[i][1] ^= v
        s[i][2] ^= u
        s[i][3] ^= v
    mix_columns(s)

state = [
    [108, 106, 71, 86],
    [96, 62, 38, 72],
    [42, 184, 92, 209],
    [94, 79, 8, 54],
]

inv_mix_columns(state)
inv_shift_rows(state)
for i in range(4):
        for j in range (4):
            print(chr(state[i][j]), end = "")
```

##### c) Kết quả:
`crypto{d1ffUs3R}`

#### 7. Bringing It All Together:
##### a) Mô tả đề bài:
- Đề yêu cầu ta hoàn thiện hàm giải mã trong source triển khai AES - 128 có sẵn bằng cách thực hiện các bước đảo ngược quá trình mã hoá AES. Các bước này bao gồm:
    ![image](https://hackmd.io/_uploads/rJpXWBZT1e.png)
    - **Key Expansion:** Tạo ra các khoá vòng để sử dụng trong suốt quá trình mã hoa và giải mã.
    - **SubBytes:** Thực hiện phép thay thế các byte trong state với các giá trị tương ứng từ bảng S - box.
    - **ShiftRows:** Dịch các hàng trong state, tạo sự phân tán.
    - **MixColumns:** Trộn các cột của state, tạo sự phân tán thêm.
    - **AddRoundKey:** XOR state với khoá vòng tương ứng.
- Trong quá trình giải mã, các bước này cần được thực hiện theo thứ tự ngược lại:
    - **Inverse SubBytes:** Sử dụng bảng S - box đảo ngược.
    - **Inverse ShiftRows:** Dịch lại các hàng trong state theo chiều ngược lại.
    - **Inverse MixColumns:** Trộn lại các cột theo cách ngược lại.
    - **AddRoundKey:** XOR state với khoá vòng (vì XOR là phép toán nghịch đảo của chính nó).
- Kết quả cuối cùng sau khi giải mã chính là plaintext.

##### b) Phân tích cách làm:
Dựa vào các challenge trên, ta gộp các bước của AES lại như sau:
```py
N_ROUNDS = 10

key        = b'\xc3,\\\xa6\xb5\x80^\x0c\xdb\x8d\xa5z*\xb6\xfe\\'
ciphertext = b'\xd1O\x14j\xa4+O\xb6\xa1\xc4\x08B)\x8f\x12\xdd'

s_box = (
    0x63, 0x7C, 0x77, 0x7B, 0xF2, 0x6B, 0x6F, 0xC5, 0x30, 0x01, 0x67, 0x2B, 0xFE, 0xD7, 0xAB, 0x76,
    0xCA, 0x82, 0xC9, 0x7D, 0xFA, 0x59, 0x47, 0xF0, 0xAD, 0xD4, 0xA2, 0xAF, 0x9C, 0xA4, 0x72, 0xC0,
    0xB7, 0xFD, 0x93, 0x26, 0x36, 0x3F, 0xF7, 0xCC, 0x34, 0xA5, 0xE5, 0xF1, 0x71, 0xD8, 0x31, 0x15,
    0x04, 0xC7, 0x23, 0xC3, 0x18, 0x96, 0x05, 0x9A, 0x07, 0x12, 0x80, 0xE2, 0xEB, 0x27, 0xB2, 0x75,
    0x09, 0x83, 0x2C, 0x1A, 0x1B, 0x6E, 0x5A, 0xA0, 0x52, 0x3B, 0xD6, 0xB3, 0x29, 0xE3, 0x2F, 0x84,
    0x53, 0xD1, 0x00, 0xED, 0x20, 0xFC, 0xB1, 0x5B, 0x6A, 0xCB, 0xBE, 0x39, 0x4A, 0x4C, 0x58, 0xCF,
    0xD0, 0xEF, 0xAA, 0xFB, 0x43, 0x4D, 0x33, 0x85, 0x45, 0xF9, 0x02, 0x7F, 0x50, 0x3C, 0x9F, 0xA8,
    0x51, 0xA3, 0x40, 0x8F, 0x92, 0x9D, 0x38, 0xF5, 0xBC, 0xB6, 0xDA, 0x21, 0x10, 0xFF, 0xF3, 0xD2,
    0xCD, 0x0C, 0x13, 0xEC, 0x5F, 0x97, 0x44, 0x17, 0xC4, 0xA7, 0x7E, 0x3D, 0x64, 0x5D, 0x19, 0x73,
    0x60, 0x81, 0x4F, 0xDC, 0x22, 0x2A, 0x90, 0x88, 0x46, 0xEE, 0xB8, 0x14, 0xDE, 0x5E, 0x0B, 0xDB,
    0xE0, 0x32, 0x3A, 0x0A, 0x49, 0x06, 0x24, 0x5C, 0xC2, 0xD3, 0xAC, 0x62, 0x91, 0x95, 0xE4, 0x79,
    0xE7, 0xC8, 0x37, 0x6D, 0x8D, 0xD5, 0x4E, 0xA9, 0x6C, 0x56, 0xF4, 0xEA, 0x65, 0x7A, 0xAE, 0x08,
    0xBA, 0x78, 0x25, 0x2E, 0x1C, 0xA6, 0xB4, 0xC6, 0xE8, 0xDD, 0x74, 0x1F, 0x4B, 0xBD, 0x8B, 0x8A,
    0x70, 0x3E, 0xB5, 0x66, 0x48, 0x03, 0xF6, 0x0E, 0x61, 0x35, 0x57, 0xB9, 0x86, 0xC1, 0x1D, 0x9E,
    0xE1, 0xF8, 0x98, 0x11, 0x69, 0xD9, 0x8E, 0x94, 0x9B, 0x1E, 0x87, 0xE9, 0xCE, 0x55, 0x28, 0xDF,
    0x8C, 0xA1, 0x89, 0x0D, 0xBF, 0xE6, 0x42, 0x68, 0x41, 0x99, 0x2D, 0x0F, 0xB0, 0x54, 0xBB, 0x16,
)

inv_s_box = (
    0x52, 0x09, 0x6A, 0xD5, 0x30, 0x36, 0xA5, 0x38, 0xBF, 0x40, 0xA3, 0x9E, 0x81, 0xF3, 0xD7, 0xFB,
    0x7C, 0xE3, 0x39, 0x82, 0x9B, 0x2F, 0xFF, 0x87, 0x34, 0x8E, 0x43, 0x44, 0xC4, 0xDE, 0xE9, 0xCB,
    0x54, 0x7B, 0x94, 0x32, 0xA6, 0xC2, 0x23, 0x3D, 0xEE, 0x4C, 0x95, 0x0B, 0x42, 0xFA, 0xC3, 0x4E,
    0x08, 0x2E, 0xA1, 0x66, 0x28, 0xD9, 0x24, 0xB2, 0x76, 0x5B, 0xA2, 0x49, 0x6D, 0x8B, 0xD1, 0x25,
    0x72, 0xF8, 0xF6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xD4, 0xA4, 0x5C, 0xCC, 0x5D, 0x65, 0xB6, 0x92,
    0x6C, 0x70, 0x48, 0x50, 0xFD, 0xED, 0xB9, 0xDA, 0x5E, 0x15, 0x46, 0x57, 0xA7, 0x8D, 0x9D, 0x84,
    0x90, 0xD8, 0xAB, 0x00, 0x8C, 0xBC, 0xD3, 0x0A, 0xF7, 0xE4, 0x58, 0x05, 0xB8, 0xB3, 0x45, 0x06,
    0xD0, 0x2C, 0x1E, 0x8F, 0xCA, 0x3F, 0x0F, 0x02, 0xC1, 0xAF, 0xBD, 0x03, 0x01, 0x13, 0x8A, 0x6B,
    0x3A, 0x91, 0x11, 0x41, 0x4F, 0x67, 0xDC, 0xEA, 0x97, 0xF2, 0xCF, 0xCE, 0xF0, 0xB4, 0xE6, 0x73,
    0x96, 0xAC, 0x74, 0x22, 0xE7, 0xAD, 0x35, 0x85, 0xE2, 0xF9, 0x37, 0xE8, 0x1C, 0x75, 0xDF, 0x6E,
    0x47, 0xF1, 0x1A, 0x71, 0x1D, 0x29, 0xC5, 0x89, 0x6F, 0xB7, 0x62, 0x0E, 0xAA, 0x18, 0xBE, 0x1B,
    0xFC, 0x56, 0x3E, 0x4B, 0xC6, 0xD2, 0x79, 0x20, 0x9A, 0xDB, 0xC0, 0xFE, 0x78, 0xCD, 0x5A, 0xF4,
    0x1F, 0xDD, 0xA8, 0x33, 0x88, 0x07, 0xC7, 0x31, 0xB1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xEC, 0x5F,
    0x60, 0x51, 0x7F, 0xA9, 0x19, 0xB5, 0x4A, 0x0D, 0x2D, 0xE5, 0x7A, 0x9F, 0x93, 0xC9, 0x9C, 0xEF,
    0xA0, 0xE0, 0x3B, 0x4D, 0xAE, 0x2A, 0xF5, 0xB0, 0xC8, 0xEB, 0xBB, 0x3C, 0x83, 0x53, 0x99, 0x61,
    0x17, 0x2B, 0x04, 0x7E, 0xBA, 0x77, 0xD6, 0x26, 0xE1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0C, 0x7D,
)

def add_round_key(s, k):
    for i in range(4):
        for j in range(4):
            s[i][j]^=k[i][j]

def bytes2matrix(text):
    """ Converts a 16-byte array into a 4x4 matrix.  """
    return [list(text[i:i+4]) for i in range(0, len(text), 4)]

def matrix2bytes(matrix):
    res = ""
    for i in range(4):
        for j in range(4):
            res += chr(matrix[i][j])
    return res

def shift_rows(s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[1][1], s[2][1], s[3][1], s[0][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[3][3], s[0][3], s[1][3], s[2][3]

def inv_shift_rows(s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[3][1], s[0][1], s[1][1], s[2][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[1][3], s[2][3], s[3][3], s[0][3]

xtime = lambda a: (((a << 1) ^ 0x1B) & 0xFF) if (a & 0x80) else (a << 1)
def mix_single_column(a):
    # see Sec 4.1.2 in The Design of Rijndael
    t = a[0] ^ a[1] ^ a[2] ^ a[3]
    u = a[0]
    a[0] ^= t ^ xtime(a[0] ^ a[1])
    a[1] ^= t ^ xtime(a[1] ^ a[2])
    a[2] ^= t ^ xtime(a[2] ^ a[3])
    a[3] ^= t ^ xtime(a[3] ^ u)

def mix_columns(s):
    for i in range(4):
        mix_single_column(s[i])

def inv_mix_columns(s):
    # see Sec 4.1.3 in The Design of Rijndael
    for i in range(4):
        u = xtime(xtime(s[i][0] ^ s[i][2]))
        v = xtime(xtime(s[i][1] ^ s[i][3]))
        s[i][0] ^= u
        s[i][1] ^= v
        s[i][2] ^= u
        s[i][3] ^= v
    mix_columns(s)

def inv_sub_byte(s):
    for i in range(4):
        for j in range(4):
            s[i][j] = inv_s_box[s[i][j]]

def expand_key(master_key):
    """
    Expands and returns a list of key matrices for the given master_key.
    """

    # Round constants https://en.wikipedia.org/wiki/AES_key_schedule#Round_constants
    r_con = (
        0x00, 0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40,
        0x80, 0x1B, 0x36, 0x6C, 0xD8, 0xAB, 0x4D, 0x9A,
        0x2F, 0x5E, 0xBC, 0x63, 0xC6, 0x97, 0x35, 0x6A,
        0xD4, 0xB3, 0x7D, 0xFA, 0xEF, 0xC5, 0x91, 0x39,
    )

    # Initialize round keys with raw key material.
    key_columns = bytes2matrix(master_key)
    iteration_size = len(master_key) // 4

    # Each iteration has exactly as many columns as the key material.
    i = 1
    while len(key_columns) < (N_ROUNDS + 1) * 4:
        # Copy previous word.
        word = list(key_columns[-1])

        # Perform schedule_core once every "row".
        if len(key_columns) % iteration_size == 0:
            # Circular shift.
            word.append(word.pop(0))
            # Map to S-BOX.
            word = [s_box[b] for b in word]
            # XOR with first byte of R-CON, since the others bytes of R-CON are 0.
            word[0] ^= r_con[i]
            i += 1
        elif len(master_key) == 32 and len(key_columns) % iteration_size == 4:
            # Run word through S-box in the fourth iteration when using a
            # 256-bit key.
            word = [s_box[b] for b in word]

        # XOR with equivalent word from previous iteration.
        word = bytes(i^j for i, j in zip(word, key_columns[-iteration_size]))
        key_columns.append(word)

    # Group key words in 4x4 byte matrices.
    return [key_columns[4*i : 4*(i+1)] for i in range(len(key_columns)//4)]


def decrypt(key, ciphertext):
    round_keys = expand_key(key) # Remember to start from the last round key and work backwards through them when decrypting

    # Convert ciphertext to state matrix
    text = bytes2matrix(ciphertext)
    # Initial add round key step
    add_round_key(text, round_keys[10])
    for i in range(N_ROUNDS - 1, 0, -1):
        inv_shift_rows(text)
        inv_sub_byte(text)
        add_round_key(text, round_keys[i])
        inv_mix_columns(text)

    # Run final round (skips the InvMixColumns step)
    inv_shift_rows(text)
    inv_sub_byte(text)
    add_round_key(text, round_keys[0])
    # Convert state matrix to plaintext
    plaintext = matrix2bytes(text)
    '''plaintext = bytes(byte_array).decode()'''
    return plaintext

# print(decrypt(key, ciphertext))
print(decrypt(key, ciphertext))
```

##### c) Kết quả:
`crypto{MYAES128}`

### II. SYMMETRIC STARTER:
#### 1. Modes of Operation Stater:
##### a) Mô tả đề bài:
- Đề bài giải thích về các chế độ hoạt động (Modes of Operation) của AES khi xử lý các tin nhắn dài hơn một khối dữ liệu. Mỗi chế độ có những điểm yếu nghiêm trọng nếu không được sử dụng đúng cách. Ta được đưa một đường link để có thể tương tác với các API và khai thác những điểm yếu trong đó để tìm được flag.
- Link được cho: https://aes.cryptohack.org/block_cipher_starter
- Mô tả trong link: Hướng dẫn cách sử dụng các chế độ hoạt động của AES để mã hóa dữ liệu dài. Mỗi chế độ có điểm yếu nếu dùng sai cách và ta phải khai thác những điểm yếu đó để tìm FLAG. Có thể tương tác với API hoặc gửi yêu cầu GET để lấy dữ liệu và tìm ra FLAG.

##### b) Phân tích cách làm:
- Source code trong link:
```py
from Crypto.Cipher import AES


KEY = ?
FLAG = ?


@chal.route('/block_cipher_starter/decrypt/<ciphertext>/')
def decrypt(ciphertext):
    ciphertext = bytes.fromhex(ciphertext)

    cipher = AES.new(KEY, AES.MODE_ECB)
    try:
        decrypted = cipher.decrypt(ciphertext)
    except ValueError as e:
        return {"error": str(e)}

    return {"plaintext": decrypted.hex()}


@chal.route('/block_cipher_starter/encrypt_flag/')
def encrypt_flag():
    cipher = AES.new(KEY, AES.MODE_ECB)
    encrypted = cipher.encrypt(FLAG.encode())

    return {"ciphertext": encrypted.hex()}
```
- Ta chú ý thấy dòng `@chal.route('/block_cipher_starter/encrypt_flag/')` trong source code trên. Thêm phần `/encrypt_flag/` vào đường link (link mới là https://aes.cryptohack.org/block_cipher_starter/encrypt_flag/), ta tìm được ciphertext là `a3c22d1bfcccd351b995f478a962da376a2cb1620cfda8c02aa6befcda831698`.
- Đưa ciphertext vào phần **DECRYPT (CIPHERTEXT)**:
![image](https://hackmd.io/_uploads/By7zZfCKyx.png)
Ta được phần plaintext là `63727970746f7b626c30636b5f633170683372355f3472335f663435375f217d`. 
- Vì plaintext ở dạng hex nên ta tiếp tục đưa nó vào phần **HEX ENCODER/DECODER**:
![image](https://hackmd.io/_uploads/HyX3bf0Y1g.png)

##### c) Kết quả: 
`crypto{bl0ck_c1ph3r5_4r3_f457_!}`

#### 2. Passwords as Keys:
##### a) Mô tả đề bài:
- Khoá trong thuật toán mã hoá đối xứng phải là những byte ngẫu nhiên thay vì mật khẩu hoặc dữ liệu có thể dự đoán được.
- Những byte ngẫu nhiên này phải được tạo ra từ một bộ sinh số giả ngẫu nhiên an toàn với mật mã (CSPRNG).
- Nếu khoá dễ đoán, độ bảo mật của thuật toán mã hoá sẽ giảm và có thể bị tấn công để giải mã.
- Trong challenge này, khoá được tạo ra từ một mật khẩu đơn giản thông qua một hàm băm, điều này khiến mã hoá trở nên yếu và có thể bị tấn công.
- Có thể sử dụng các request HTTP tự động hoặc tấn công ngoại tuyến để giải mã ciphertext.
- Link đề cho: https://aes.cryptohack.org/passwords_as_keys
- Mô tả trong link: Khóa trong mã hóa đối xứng phải là chuỗi byte ngẫu nhiên thay vì mật khẩu dễ đoán. Nếu khóa được tạo từ mật khẩu qua hàm băm, ciphertext có thể bị tấn công. Có thể gửi yêu cầu HTTP hoặc phân tích ciphertext offline để tìm FLAG.

##### b) Phân tích cách làm:
- Source code trong link:
```py
from Crypto.Cipher import AES
import hashlib
import random


# /usr/share/dict/words from
# https://gist.githubusercontent.com/wchargin/8927565/raw/d9783627c731268fb2935a731a618aa8e95cf465/words
with open("/usr/share/dict/words") as f:
    words = [w.strip() for w in f.readlines()]
keyword = random.choice(words)

KEY = hashlib.md5(keyword.encode()).digest()
FLAG = ?


@chal.route('/passwords_as_keys/decrypt/<ciphertext>/<password_hash>/')
def decrypt(ciphertext, password_hash):
    ciphertext = bytes.fromhex(ciphertext)
    key = bytes.fromhex(password_hash)

    cipher = AES.new(key, AES.MODE_ECB)
    try:
        decrypted = cipher.decrypt(ciphertext)
    except ValueError as e:
        return {"error": str(e)}

    return {"plaintext": decrypted.hex()}


@chal.route('/passwords_as_keys/encrypt_flag/')
def encrypt_flag():
    cipher = AES.new(KEY, AES.MODE_ECB)
    encrypted = cipher.encrypt(FLAG.encode())

    return {"ciphertext": encrypted.hex()}
```
- Tương tự bài trên, ta vào link https://aes.cryptohack.org/passwords_as_keys/encrypt_flag/ và tìm được ciphertext là `c92b7734070205bdf6c0087a751466ec13ae15e6f1bcdd3f3a535ec0f4bbae66`.
- Vào link https://gist.githubusercontent.com/wchargin/8927565/raw/d9783627c731268fb2935a731a618aa8e95cf465/words, ta được một danh sách các từ khoá và lưu danh sách này thành file `key.txt`. Vì việc đoán được từ nào là khoá rất khó khăn nên ta sẽ brute force.
- Từ source code đề cho, ta được source code sau:
```py
from Crypto.Cipher import AES
import hashlib

def decrypt(ciphertext, password_hash):
    ciphertext = bytes.fromhex(ciphertext)
    key = password_hash

    cipher = AES.new(key, AES.MODE_ECB)
    try:
        decrypted = cipher.decrypt(ciphertext)
    except ValueError as e:
        return {"error": str(e)}

    return decrypted

ct = "c92b7734070205bdf6c0087a751466ec13ae15e6f1bcdd3f3a535ec0f4bbae66"

with open ("C:/Users/Administrator/Documents/Ngôn ngữ lập trình/Python/key.txt") as f:
    words = [w.strip() for w in f.readlines()]

for i in range (len(words)):
    KEY = hashlib.md5(words[i].encode()).digest()
    flag = decrypt(ct, KEY)
    if b'crypto' in flag:
        print(flag)
```

##### c) Kết quả:
`crypto{k3y5__r__n07__p455w0rdz?}`

### III. BLOCK CIPHERS 1:
#### 1. ECB CBC WTF:
##### a) Mô tả đề bài:
- *"Ở đây bạn có thể mã hóa trong CBC nhưng chỉ giải mã trong ECB. Đó không phải là một điểm yếu bởi vì chúng là các chế độ khác nhau ... phải không?"*
- Link đề cho: https://aes.cryptohack.org/ecbcbcwtf
- Source code trong link: 
```py
from Crypto.Cipher import AES

KEY = ?
FLAG = ?

@chal.route('/ecbcbcwtf/decrypt/<ciphertext>/')
def decrypt(ciphertext):
    ciphertext = bytes.fromhex(ciphertext)

    cipher = AES.new(KEY, AES.MODE_ECB)
    try:
        decrypted = cipher.decrypt(ciphertext)
    except ValueError as e:
        return {"error": str(e)}

    return {"plaintext": decrypted.hex()}


@chal.route('/ecbcbcwtf/encrypt_flag/')
def encrypt_flag():
    iv = os.urandom(16)

    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(FLAG.encode())
    ciphertext = iv.hex() + encrypted.hex()

    return {"ciphertext": ciphertext}
```

##### b) Phân tích cách làm:
- Tương tự các bài trên, lấy được $C$:
```
d5a5de7558da0f4300da2951c62285528fa45fa0ce4748c2ee012d579c8e5542dcb940ab3884d2716d51fbe664ac376c
```
- Từ source đề cho, ta thấy $C$ là phép nối $||$: $$C = IV || c$$ Trong đó:
    - $C$: Ciphertext tổng thể.
    - $IV$ - Vector khởi tạo: Một chuỗi ngẫu nhiên $16$ byte được thêm vào để đảm bảo tính ngẫu nhiên.
    - $c$: Phần dữ liệu được mã hoá từ plaintext bằng AES - CBC.
- Vì AES - CBC mã hoá từng khối theo công thức: $$C_i = AES_{Encrypt}(P_i \oplus C_{i - 1})$$ Với:
    - $C_0 = IV$
    - $C_1 = AES(P_1 \oplus IV)$
    - $C_2 = AES(P_2 \oplus C_1)$
    - ...
    - $P_i = AES_{Decrypt}(C_i) \oplus C_{i - 1}$
- Mô tả đề cho phép ta giải mã AES - ECB, mà AES - ECB chỉ là một phần trong công thức giải mã AES - CBC. Ta thấy rằng AES - ECB giải mã từng block độc lập (không XOR với block trước đó) nhưng trong AES - CBC thì phải XOR với block trước đó để lấy plaintext $\rightarrow$ Có thể gửi $C_i$ vào API AES - ECB để lấy $AES_{Decrypt}(C_i)$, sau đó XOR với $C_{i - 1}$ để tìm $P_i$.
- Trong ECB plaintext được chia thành từng block $16$ byte, vì vậy ta tách $IV$ là $16$ byte đầu tiên và những block tiếp theo sẽ là ciphertext.
- Gửi `ct_b1` và `ct_b2` vào link "https://aes.cryptohack.org/ecbcbcwtf/decrypt/<ciphertext>/" và lấy được hai plaintext $AES_{Decrypt}(C_1)$ và $AES_{Decrypt}(C_2)$:
```py
pt_b1 = "b6d7a7052cb5747063b87664b341ee67"
pt_b2 = "d0902990ff2317f3d95e0c76bdaf743f"
```
- Chuyển về dạng byte và thực hiện XOR để tìm plaintext $P_1$ và $P_2$: $$P_1 = AES_{Decrypt}(C_1) \oplus IV$$ $$P_2 = AES_{Decrypt}(C_2) \oplus C_1$$
- Nối chúng lại với nhau: $$P = P_1 || P_2$$
- Source code: 
```py
from Crypto.Util.number import long_to_bytes, bytes_to_long

C = "d5a5de7558da0f4300da2951c62285528fa45fa0ce4748c2ee012d579c8e5542dcb940ab3884d2716d51fbe664ac376c"

def xor(a, b):
    return long_to_bytes(bytes_to_long(a)^bytes_to_long(b))

C = bytes.fromhex(C)
iv = C[:16]
block1 = C[16:32]
block2 = C[32:]

# print(block1.hex())
# print(block2.hex())
pt_b1 = "b6d7a7052cb5747063b87664b341ee67"
pt_b2 = "d0902990ff2317f3d95e0c76bdaf743f"

de_b1 = xor(bytes.fromhex(pt_b1), iv)
de_b2 = xor(bytes.fromhex(pt_b2), block1)
print(de_b1 + de_b2)
```
- Ngoài ra, ta có thể dùng API để lấy các plaintext và ciphertext:
```py
import requests
from Crypto.Util.number import long_to_bytes, bytes_to_long

def response(byte_string):
	url = "http://aes.cryptohack.org/ecbcbcwtf/decrypt/"
	url += byte_string.hex()
	url += "/"
	r = requests.get(url)
	js = r.json()
	return bytes.fromhex(js["plaintext"])

def encrypt_flag():
	url = "http://aes.cryptohack.org/ecbcbcwtf/encrypt_flag/"
	r = requests.get(url)
	js = r.json()
	return bytes.fromhex(js["ciphertext"])

def xor(a, b):
	return long_to_bytes(bytes_to_long(a)^bytes_to_long(b))

enc = encrypt_flag()

iv = enc[:16]
block1 = enc[16:32]
block2 = enc[32:]

decrypt_block1 = xor(response(block1), iv)
decrypt_block2 = xor(response(block2), block1)
print(decrypt_block1 + decrypt_block2)
```

##### c) Kết quả:
`crypto{3cb_5uck5_4v01d_17_!!!!!}`

#### 2. ECB Oracle:
##### a) Mô tả đề bài:
- *"ECB là chế độ đơn giản nhất, với mỗi khối văn bản được mã hóa hoàn toàn độc lập. Trong trường hợp này, đầu vào của bạn được chuẩn bị cho flag bí mật và được mã hóa và đó là nó. Chúng tôi thậm chí không cung cấp chức năng giải mã. Có lẽ bạn không cần một nhà tiên tri đệm khi bạn có "ECB Oracle"?"*
- Link đề cho: https://aes.cryptohack.org/ecb_oracle
- Source code trong link:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

KEY = ?
FLAG = ?

@chal.route('/ecb_oracle/encrypt/<plaintext>/')
def encrypt(plaintext):
    plaintext = bytes.fromhex(plaintext)

    padded = pad(plaintext + FLAG.encode(), 16)
    cipher = AES.new(KEY, AES.MODE_ECB)
    try:
        encrypted = cipher.encrypt(padded)
    except ValueError as e:
        return {"error": str(e)}

    return {"ciphertext": encrypted.hex()}
```
    
##### b) Phân tích cách làm:
- Ta đã biết một phần flag là `crypto{`, vì không có hàm giải mã nên ta sẽ brute - force từng byte một theo block AES bằng cách lợi dụng ECB mode.
- Đầu tiên, thiết lập một hàm `encrypt()` gửi dữ liệu lên server để mã hóa bằng AES - ECB, hàm này sẽ trả về ciphertext dạng hex tuỳ vào plaintext được đưa vào (`hexlify()` là một hàm trong thư viện `binascii`, dùng để chuyển đổi dữ liệu dạng bytes thành chuỗi hex).
- Đặt biến $n = 31$ giúp căn chỉnh độ dài đầu vào sao cho Flag rơi vào block mà ta muốn giải mã ($31 = 16$ (full block đầu tiên) $+ 15$ (full block thứ hai, trừ đi một byte là ký tự mới cần tìm)) $\rightarrow$ Khi gửi `payload` lên server, Flag sẽ được đẩy vào vị trí ta có thể kiểm soát.
- Tấn công từng byte của Flag bằng vòng lặp `while True`: Gửi chuỗi byte `1` có độ dài `n - len(flag)`:
    - Khi Flag càng dài, càng ít padding `1` để giữ Flag đúng vị trí.
    - Khi Flag là `crypto{` (có độ dài là $7$) thì `payload` có $31 - 7 = 24$ bytes.
    - Khi Flag là `crypto{?` (có độ dài là $8$) thì `payload` có $31 - 8 = 23$ bytes.
    - Gửi `payload` lên server để lấy ciphertext thử nghiệm.
```py
while 1:
    payload = b'1' * (n - len(flag))
    kq = encrypt(payload)
```
- Brute - force từng ký tự tiếp theo: 
    - Duyệt qua từng `i` trong tất cả các ký tự có thể in trong thư viện `printable`, bao gồm: ```abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}!@#$%^&*()```
    - Tạo chuỗi thử nghiệm `payload + flag + i` và gửi chuỗi lên server và lưu vào biến `enc`.
    - Kiểm tra `kq[32:64] == enc[32:64]`:
        - `[0:32]` chứa phần padding `1` không quan trọng và `[32:64]` chứa phần Flag bị mã hoá.
        - Nếu hai phần trên giống nhau, ký tự `i` vừa tìm được thuộc Flag.
        - Kiểm tra cho đến khi nào ký tự `i` tìm được là `}` thì dừng vòng lặp, lúc này ta đã tìm được Flag.
```py
for i in printable:
        enc = encrypt(payload + flag.encode() + i.encode())
        if kq[32:64] == enc[32:64]:
            flag += i
            print("flag letter: ", i)
            break
    if flag[-1] == '}':
        break
```
- Source code đầy đủ:
```py
from binascii import hexlify
import requests
import json
from string import printable

def encrypt(pt):
    p = hexlify(pt).decode()
    url = "http://aes.cryptohack.org/ecb_oracle/encrypt/" + p
    r = requests.get(url)
    ct = (json.loads(r.text))['ciphertext']
    return ct

flag = "crypto{"
n = 31
while 1:
    payload = b'1'*(n - len(flag))
    kq = encrypt(payload)
    for i in printable:
        enc = encrypt(payload + flag.encode() + i.encode())
        if kq[32:64] == enc[32:64]:
            flag += i
            print("flag letter: ", i)
            break
    if flag[-1] == '}':
        break
print(flag)
```
:::spoiler Kết quả trả về:
```
flag letter:  p
flag letter:  3
flag letter:  n
flag letter:  6
flag letter:  u
flag letter:  1
flag letter:  n
flag letter:  5
flag letter:  _
flag letter:  h
flag letter:  4
flag letter:  7
flag letter:  3
flag letter:  _
flag letter:  3
flag letter:  c
flag letter:  b
flag letter:  }
crypto{p3n6u1n5_h473_3cb}
```
:::

##### c) Kết quả:
`crypto{p3n6u1n5_h473_3cb}`

#### 3. Flipping Cookie:
##### a) Mô tả đề bài:
- *"Bạn có thể nhận cookie cho trang web của tôi, nhưng nó sẽ không giúp bạn đọc flag ... Tôi nghĩ vậy."*
- Link đề cho: https://aes.cryptohack.org/flipping_cookie
- Source code:
```py
from Crypto.Cipher import AES
import os
from Crypto.Util.Padding import pad, unpad
from datetime import datetime, timedelta

KEY = ?
FLAG = ?

@chal.route('/flipping_cookie/check_admin/<cookie>/<iv>/')
def check_admin(cookie, iv):
    cookie = bytes.fromhex(cookie)
    iv = bytes.fromhex(iv)

    try:
        cipher = AES.new(KEY, AES.MODE_CBC, iv)
        decrypted = cipher.decrypt(cookie)
        unpadded = unpad(decrypted, 16)
    except ValueError as e:
        return {"error": str(e)}

    if b"admin=True" in unpadded.split(b";"):
        return {"flag": FLAG}
    else:
        return {"error": "Only admin can read the flag"}


@chal.route('/flipping_cookie/get_cookie/')
def get_cookie():
    expires_at = (datetime.today() + timedelta(days=1)).strftime("%s")
    cookie = f"admin=False;expiry={expires_at}".encode()

    iv = os.urandom(16)
    padded = pad(cookie, 16)
    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(padded)
    ciphertext = iv.hex() + encrypted.hex()

    return {"cookie": ciphertext}

```

##### b) Phân tích cách làm:
- Đây là bài **Bit Flipping Attack** trên **AES - CBC**, ta sẽ lợi dụng tính chất của CBC để chỉnh sửa nội dung giải mã mà không cần biết key.
- CBC mã hoá từng khối theo công thức: $$C_i = AES_{Encrypt}(P_i \oplus C_{i - 1})$$ Với:
    - $C_0 = IV$
    - $C_1 = AES(P_1 \oplus IV)$
    - $C_2 = AES(P_2 \oplus C_1)$
    - ...
    - $P_i = AES_{Decrypt}(C_i) \oplus C_{i - 1}$
- Dễ thấy rằng hàm `get_cookie()` của server trả về `ciphertext = iv.hex() + encrypted.hex()` với `iv` ngẫu nhiên và `cookie` đã được mã hoá và chuyển sang dạng hex. `ciphertext` (hay cookie đã được mã hoá thành ciphertext) được trả về có dạng: 
```
b3deac928caec1b5902b56b867aafddaeff1be628fa3b8bc11f14d1702b93be0b2ae7f72c7c8265b1228175bb1c0f468
```
- Theo đề, `cookie = "admin=False" + something` và ta cần chuyển `admin=False` thành `admin=True`.
- Ta có: $$P_1 = AES_{Decrypt}(C_1) \oplus IV$$ Với: 
    - `P1 = b'admin=False;expi'`, có độ dài là `len(origin) = 16`, là byte gốc. 
    - `P = b'admin=True;\x05\x05\x05\x05\x05'`, có độ dài là `len(origin) = 16` (thêm các padding `\x05\x05\x05\x05\x05` để hai byte trên có cùng độ dài), là byte mục tiêu.
- Thay đổi $IV$ để kiểm soát $P_1$ bằng phép XOR: $$IV'= IV \oplus P_1 \oplus P$$
    - Phân tích rõ: XOR với $P_1$ để loại bỏ giá trị không mong muốn: $$P_1 \oplus P_1 = (AES_{Decrypt}(C_1) \oplus IV) \oplus P_1$$ Tương đương: $$0 = AES_{Decrypt}(C_1) \oplus IV \oplus P_1$$ XOR cả hai vế với $P$: $$0 \oplus P = (AES_{Decrypt}(C_1) \oplus IV \oplus P_1) \oplus P$$ Tương đương: $$P = (AES_{Decrypt}(C_1) \oplus IV \oplus P_1) \oplus P$$ Hay: $$P = P$$ Khi đó gửi lên hàm `check_admine(block1, IV)` của server, ta được: $$P = AES_{Decrypt}(C_1) \oplus IV'$$
- Source code:
```py
from Crypto.Util.number import long_to_bytes, bytes_to_long
import requests

def get_cookie():
    link = "http://aes.cryptohack.org/flipping_cookie/get_cookie/"
    r = requests.get(link)
    js = r.json()
    return bytes.fromhex(js["cookie"])

def check_admin(cookie, iv):
    link = "http://aes.cryptohack.org/flipping_cookie/check_admin/"
    link += cookie.hex()
    link += "/"
    link += iv.hex()
    link += "/"
    r = requests.get(link)
    js = r.json()
    return js

def xor(a, b):
    return long_to_bytes(bytes_to_long(a)^bytes_to_long(b))

cookie = get_cookie()

P1 = b'admin=False;expi'
P = b'admin=True;\x05\x05\x05\x05\x05'

iv = cookie[:16]
block1 = cookie[16:32]
block2 = cookie[32:]

IV = xor(iv, xor(P1, P))
print(check_admin(block1, IV))
```

##### c) Kết quả:
`crypto{4u7h3n71c4710n_15_3553n714l}`

#### 4. Lazy CBC:
##### a) Mô tả đề bài:
- *"Tôi chỉ là một dev lười biếng và muốn mã hóa CBC của tôi hoạt động. Tất cả những gì nói về các vector khởi tạo là gì? Nghe có vẻ không quan trọng."*
- Link được cho: https://aes.cryptohack.org/lazy_cbc
- Source code:
```py
from Crypto.Cipher import AES

KEY = ?
FLAG = ?

@chal.route('/lazy_cbc/encrypt/<plaintext>/')
def encrypt(plaintext):
    plaintext = bytes.fromhex(plaintext)
    if len(plaintext) % 16 != 0:
        return {"error": "Data length must be multiple of 16"}

    cipher = AES.new(KEY, AES.MODE_CBC, KEY)
    encrypted = cipher.encrypt(plaintext)

    return {"ciphertext": encrypted.hex()}


@chal.route('/lazy_cbc/get_flag/<key>/')
def get_flag(key):
    key = bytes.fromhex(key)

    if key == KEY:
        return {"plaintext": FLAG.encode().hex()}
    else:
        return {"error": "invalid key"}


@chal.route('/lazy_cbc/receive/<ciphertext>/')
def receive(ciphertext):
    ciphertext = bytes.fromhex(ciphertext)
    if len(ciphertext) % 16 != 0:
        return {"error": "Data length must be multiple of 16"}

    cipher = AES.new(KEY, AES.MODE_CBC, KEY)
    decrypted = cipher.decrypt(ciphertext)

    try:
        decrypted.decode() # ensure plaintext is valid ascii
    except UnicodeDecodeError:
        return {"error": "Invalid plaintext: " + decrypted.hex()}

    return {"success": "Your message has been received"}
```
##### b) Phân tích cách làm:
- Theo source đề, trong hàm `encrypt()`, plaintext được mã hoá bằng **AES - CBC** và dùng `KEY` để mã hoá thay vì $IV$.
- Cách hoạt động của CBC: $$P_1 = AES_{Decrypt}(C_1) \oplus IV$$ $$P_2 = AES_{Decrypt}(C_2) \oplus IV$$ Vì $IV$ là `KEY`: $$P_1 = AES_{Decrypt}(C_1) \oplus KEY$$ $$P_2 = AES_{Decrypt}(C_2) \oplus C_1$$
- Gửi một plaintext giả mạo $c$ là $32$ byte `x00` và  tìm KEY bằng cách phân tích plaintext mà server trả về. Ta có: $$p_1 = AES_{Decrypt}(c) \oplus KEY$$ $$p_2 = AES_{Decrypt}(c) \oplus c$$ XOR vế theo vế: $$p_1 \oplus p_2 = (AES_{Decrypt}(c_1) \oplus KEY) \oplus (AES_{Decrypt}(c_1) \oplus c_1) = KEY \oplus c_1$$ Vì $c_1$ là $32$ byte `x00` nên: $$KEY = p_1 \oplus p_2$$
- Source code:
```py
from Crypto.Util.number import long_to_bytes, bytes_to_long
import requests

def get_flag(key):
    link = "http://aes.cryptohack.org/lazy_cbc/get_flag/"
    link += key.hex()
    link += "/"
    r = requests.get(link)
    js = r.json()
    return bytes.fromhex(js["plaintext"])

def receive(ct):
    link = "http://aes.cryptohack.org/lazy_cbc/receive/"
    link += ct.hex()
    link += "/"
    r = requests.get(link)
    js = r.json()
    return bytes.fromhex(js["error"][len("Invalid plaintext: "):])

def xor(a, b):
    return long_to_bytes(bytes_to_long(a)^bytes_to_long(b))

c = b'\x00'*32
p = receive(c)
p1 = p[16:]
p2 = p[:16]
KEY = xor(p1, p2)
print(get_flag(KEY))
```

##### c) Kết quả:
`crypto{50m3_p30pl3_d0n7_7h1nk_IV_15_1mp0r74n7_?}`

#### 5. Triple DES:
##### a) Mô tả đề bài:
- *"Tiêu chuẩn mã hóa dữ liệu (Data Encryption Standard - DES) là tiền thân của AES và vẫn được sử dụng rộng rãi trong một số khu vực di chuyển chậm như ngành công nghiệp thẻ thanh toán. Challenge này cho thấy một điểm yếu kỳ lạ của DES mà một mật mã khối an toàn không nên có."*
- Link được cho: https://aes.cryptohack.org/triple_des
- Source code:
```py
from Crypto.Cipher import DES3
from Crypto.Util.Padding import pad

IV = os.urandom(8)
FLAG = ?

def xor(a, b):
    # xor 2 bytestrings, repeating the 2nd one if necessary
    return bytes(x ^ y for x,y in zip(a, b * (1 + len(a) // len(b))))


@chal.route('/triple_des/encrypt/<key>/<plaintext>/')
def encrypt(key, plaintext):
    try:
        key = bytes.fromhex(key)
        plaintext = bytes.fromhex(plaintext)
        plaintext = xor(plaintext, IV)

        cipher = DES3.new(key, DES3.MODE_ECB)
        ciphertext = cipher.encrypt(plaintext)
        ciphertext = xor(ciphertext, IV)

        return {"ciphertext": ciphertext.hex()}

    except ValueError as e:
        return {"error": str(e)}


@chal.route('/triple_des/encrypt_flag/<key>/')
def encrypt_flag(key):
    return encrypt(key, pad(FLAG.encode(), 8).hex())
```

##### b) Phân tích kết quả:
- Từ source code, dễ dàng thấy $IV$ chỉ có $8$ bit.
- Challenge này sử dụng AES - ECB và 3DES nhưng không kiểm tra Weak Key, ta có thể lợi dụng điều này.
-  Cách hoạt động của 3DES và Weak Key trong 3DES:
    - 3DES hoạt động bằng cách mã hoá ba lần với DES theo công thức: $$C = E_{K1}(D_{K2}(E_{K3}(P)))$$ Với:
        - $E_K(P)$: Mã hoá DES với khoá $K$.
        - $D_K(P)$: Giải mã DES với khoá $K$
    - Weak Key là những khoá đặc biệt khiến 3DES trở nên kém an toàn. Một số loại khoá yếu:
        - **Khoá lặp (Repeated Key):**
            - Nếu $K1 = K2 = K3$, 3DES thực tế chỉ là DES (không tăng độ an toàn).
            - Nếu $K1 = K3$, 3DES hoạt động như một biến thể của DES với chỉ một vòng mã hoá thật sự.
        - **Khoá yếu của DES:** Mã hoá hai lần sẽ ra chính nó, giúp rút gọn số lượng phép toán cần brute - force: $$E_K(E_K(P)) = P$$
- Khai thác Weak Key: Sử dụng khoá yếu $K$ là `key = b'\x00'*8 + b'\xff'*8` ($16$ byte), ta có công thức: $$E_K(E_K(P)) = P$$ Khi mã hoá `flag` lần đầu: $$C = E_K(flag)$$ Khi mã hoá lại $C$: $$P = E_K(C) = flag$$
- Kết quả:
```
b'crypto{n0t_4ll_k3ys_4r3_g00d_k3ys}\x06\x06\x06\x06\x06\x06'
```
- Source code: 
```py
import requests

def encrypt(key, pt):
    link = "https://aes.cryptohack.org/triple_des/encrypt/"
    link += key
    link += "/"
    link += pt.hex()
    link += "/"
    r = requests.get(link)
    js = r.json()
    return bytes.fromhex(js["ciphertext"])

def encrypt_flag(key):
    link = "https://aes.cryptohack.org/triple_des/encrypt_flag/"
    link += key
    link += "/"
    r = requests.get(link)
    js = r.json()
    return bytes.fromhex(js["ciphertext"])

key = b'\x00'*8 + b'\xff'*8
flag = encrypt_flag(key.hex())
flag = encrypt(key.hex(), flag)
print(flag)
```
    
##### c) Kết quả:
`crypto{n0t_4ll_k3ys_4r3_g00d_k3ys}`

### IV. STREAM CIPHERS:
#### 1. Symmetry:
##### a) Mô tả đề bài:
- *"Một số chế độ mật mã khối, chẳng hạn như OFB, CTR hoặc CFB, biến một mật mã khối thành mật mã luồng. Ý tưởng đằng sau các mật mã luồng là tạo ra một phím keysorandom mà sau đó được XOR với bản rõ. Một lợi thế của mật mã luồng là chúng có thể làm việc với các bản rõ có độ dài tùy ý, không cần padding.
OFB là một chế độ mật mã tối nghĩa, không có lợi ích thực sự trong những ngày này khi sử dụng CTR. Challenge này giới thiệu một tính chất bất thường của OFB."*
- Link đề cho: https://aes.cryptohack.org/symmetry
- Source code:
```py
from Crypto.Cipher import AES

KEY = ?
FLAG = ?

@chal.route('/symmetry/encrypt/<plaintext>/<iv>/')
def encrypt(plaintext, iv):
    plaintext = bytes.fromhex(plaintext)
    iv = bytes.fromhex(iv)
    if len(iv) != 16:
        return {"error": "IV length must be 16"}

    cipher = AES.new(KEY, AES.MODE_OFB, iv)
    encrypted = cipher.encrypt(plaintext)
    ciphertext = encrypted.hex()

    return {"ciphertext": ciphertext}


@chal.route('/symmetry/encrypt_flag/')
def encrypt_flag():
    iv = os.urandom(16)

    cipher = AES.new(KEY, AES.MODE_OFB, iv)
    encrypted = cipher.encrypt(FLAG.encode())
    ciphertext = iv.hex() + encrypted.hex()

    return {"ciphertext": ciphertext}
```

##### b) Phân tích cách làm:
- Ciphertext thu được:
```
d00b0cf47362d4b84aff2eba1a1d3730b4a11bb62c146de8961c9c3f64aa8fbf03a8e58d8ff127b47f24df17a7553bd7a6
```
- OFB tạo ra cơ chế keystream bằng cách mã hoá $IV$ với khoá, sau đó XOR với plaintext: $$C = P \oplus keystream$$
- Với $c_1$ thu được ở trên và $c_2$ thu được bằng cách gửi $p_2$ lên server (`p2 = b'\x00'*len(p1)`): $$c_1 = p_1 \oplus keystream$$ $$c_2 = p_2 \oplus keystream$$ XOR vế theo vế: $$c_1 \oplus c_2 = p_1 \oplus p_2$$ Suy ra: $$p_1 = (c_1 \oplus c_2) \oplus p_2$$ Vì $p_2 = 0$: $$p_1 = c_1 \oplus c_2$$
- Source code: 
```py
from Crypto.Util.number import long_to_bytes, bytes_to_long
import requests

def encrypt(pt, iv):
    link = "https://aes.cryptohack.org/symmetry/encrypt/"
    link += pt
    link += "/"
    link += iv
    link += "/"
    r = requests.get(link).json()
    return bytes.fromhex(r["ciphertext"])

def xor(a, b):
    return long_to_bytes(bytes_to_long(a)^bytes_to_long(b))

ct = "d00b0cf47362d4b84aff2eba1a1d3730b4a11bb62c146de8961c9c3f64aa8fbf03a8e58d8ff127b47f24df17a7553bd7a6"
c1 = bytes.fromhex(ct)
iv = c1[:16]
p1 = c1[16:]

p2 = b'\x00'*len(p1)
c2 = encrypt(p2.hex(), iv.hex())
res = xor(c1, c2)
print(res)
```
- Kết quả:
```
b'\xd0\x0b\x0c\xf4sb\xd4\xb8J\xff.\xba\x1a\x1d70crypto{0fb_15_5ymm37r1c4l_!!!11!}'
```

##### c) Kết quả:
`crypto{0fb_15_5ymm37r1c4l_!!!11!}`
    
#### 2. Bean Counter:
##### a) Mô tả đề bài:
- *"Tôi đã đấu tranh để có được chế độ truy cập của Pycrypto làm những gì tôi muốn, vì vậy tôi đã tự mình biến chế độ ECB thành CTR. Quầy của tôi có thể đi lên cả lên và xuống để loại bỏ các loại tiền mã hóa! Họ sẽ không có cơ hội nào để có thể đọc được hình ảnh của tôi."*
- Link: https://aes.cryptohack.org/bean_counter
- Source code:
```py
from Crypto.Cipher import AES

KEY = ?

class StepUpCounter(object):
    def __init__(self, step_up=False):
        self.value = os.urandom(16).hex()
        self.step = 1
        self.stup = step_up

    def increment(self):
        if self.stup:
            self.newIV = hex(int(self.value, 16) + self.step)
        else:
            self.newIV = hex(int(self.value, 16) - self.stup)
        self.value = self.newIV[2:len(self.newIV)]
        return bytes.fromhex(self.value.zfill(32))

    def __repr__(self):
        self.increment()
        return self.value



@chal.route('/bean_counter/encrypt/')
def encrypt():
    cipher = AES.new(KEY, AES.MODE_ECB)
    ctr = StepUpCounter()

    out = []
    with open("challenge_files/bean_flag.png", 'rb') as f:
        block = f.read(16)
        while block:
            keystream = cipher.encrypt(ctr.increment())
            xored = [a^b for a, b in zip(block, keystream)]
            out.append(bytes(xored).hex())
            block = f.read(16)

    return {"encrypted": ''.join(out)}
```
##### b) Phân tích cách làm:
- Phân tích source code thì có `step_up=False(=0)` dẫn đến việc `self.newIV = hex(int(self.value, 16)` hay `iv` không thay đổi trong quá trình mã hoá. $\rightarrow$ **Challenge giả vờ dùng AES - CTR nhưng lại dùng sai counter.** 
- Theo cách hoạt động của AES - CTR: $$C_i = P_i \oplus AES_{ECB}(CTR_i)$$ Do CTR bị sai (không thay đổi): $$keystream = AES_{ECB}(fixed_-IV) = K$$ $\rightarrow$ **Tất cả block đều được mã hoá bằng cùng một keystream block:** $$C_i = P_i \oplus K \rightarrow P_i = C_i \oplus K$$
- Vì $16$ bytes đầu của file `.png` luôn cố định: $89$ $50$ $4E$ $47$ $0D$ $0A$ $1A$ $0A$ $00$ $00$ $00$ $0D$ $49$ $48$ $44$ $52$ (dạng byte: `b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR'`)
- Lấy keystream: $$K = P_0 \oplus C_0$$ Với:
    - $P_0$: $16$ bytes đầu của `.png`
    - $C_0$: $16$ bytes đầu của ciphertext lấy từ server.
- Dùng $K$ để giải mã toàn bộ ảnh: $$P_i = C_i \oplus K$$
- Ghi ảnh ra file và mở bằng PIL để xem ảnh.
- Source code:
```py
import requests
from PIL import Image
from pwn import xor

def encrypt():
    url = "https://aes.cryptohack.org/bean_counter/encrypt/"
    r = requests.get(url).json()
    return bytes.fromhex(r["encrypted"])

png = b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR'
ct = encrypt()

keystream = xor(png, ct[:16])
png_flag = xor(keystream, ct)

image = open('flag.png', 'wb').write(png_flag)
flag = Image.open('flag.png')
flag.show()
```
- Kết quả:
    ![tmplqtgw4cu](https://hackmd.io/_uploads/HJbIVzWCkl.png)

##### c) Kết quả:
`crypto{hex_bytes_beans}`

#### 3. CTRIMIE:
##### a) Mô tả đề bài:
- *"Có thể có rất nhiều sự dư thừa trong bản rõ của chúng tôi, vậy tại sao không nén nó trước?"*
- Link: https://aes.cryptohack.org/ctrime
- Source code:
```py
from Crypto.Cipher import AES
from Crypto.Util import Counter
import zlib

KEY = ?
FLAG = ?

@chal.route('/ctrime/encrypt/<plaintext>/')
def encrypt(plaintext):
    plaintext = bytes.fromhex(plaintext)

    iv = int.from_bytes(os.urandom(16), 'big')
    cipher = AES.new(KEY, AES.MODE_CTR, counter=Counter.new(128, initial_value=iv))
    encrypted = cipher.encrypt(zlib.compress(plaintext + FLAG.encode()))

    return {"ciphertext": encrypted.hex()}
```

##### b) Phân tích cách làm:
- Theo đề, vì `plaintext + FLAG.encode()` được nén rồi mới mã hoá nên có thể lợi dụng tính chất của **zlib compression**: Khi đoán đúng thêm một ký tự trong Flag, zlib sẽ nén tốt hơn $\rightarrow$ Dung lượng giảm $\rightarrow$ Ciphertext ngắn hơn.
- Nếu brute - force sai, một vài kí tự sẽ làm `len(ct)` bị thay đổi, nên thay vì brute - force từ `crypto{` thì bắt đầu với `crypto{crypto{` sẽ chắc chắn hơn. Sau đó, so sánh độ dài ciphertext trước và sau khi thêm kí tự, nếu `sau < trước` thì brute - force đúng.
- Source code:
```py
import requests
from string import printable

def encrypt(plaintext):
    url = "https://aes.cryptohack.org/ctrime/encrypt/"
    url += plaintext.hex()
    url += '/'
    r = requests.get(url).json()
    return bytes.fromhex(r["ciphertext"])

flag = b'crypto{'
while True:
    temp = (flag + b'.')*2
    check = len(encrypt(temp))

    for i in printable:
        pt = (flag + i.encode())*2
        enc = encrypt(pt)
        if len(enc) < check:
            flag += i.encode()
            print(flag.decode())

    if flag[-1] == '}':
        break
```
::: spoiler Kết quả:
```
crypto{C
crypto{CR
crypto{CRI
crypto{CRIM
crypto{CRIME
crypto{CRIME_
crypto{CRIME_5
crypto{CRIME_57
crypto{CRIME_571
crypto{CRIME_571l
crypto{CRIME_571ll
crypto{CRIME_571ll_
crypto{CRIME_571ll_p
crypto{CRIME_571ll_p4
crypto{CRIME_571ll_p4y
crypto{CRIME_571ll_p4y5
crypto{CRIME_571ll_p4y5}
```
:::

##### c) Kết quả:
`crypto{CRIME_571ll_p4y5}`

#### 4. Logon Zero:
##### a) Mô tả đề bài:
- *"Trước khi sử dụng network, bạn phải xác thực với Active Directory bằng giao thức đăng nhập CFB-8 theo thời gian của chúng tôi."*
- Kết nối tại `socket.cryptohack.org 13399`
- Source code:
```py
#!/usr/bin/env python3

from Crypto.Cipher import AES
from Crypto.Util.number import bytes_to_long
from os import urandom
from utils import listener

FLAG = "crypto{???????????????????????????????}"


class CFB8:
    def __init__(self, key):
        self.key = key

    def encrypt(self, plaintext):
        IV = urandom(16)
        cipher = AES.new(self.key, AES.MODE_ECB)
        ct = b''
        state = IV
        for i in range(len(plaintext)):
            b = cipher.encrypt(state)[0]
            c = b ^ plaintext[i]
            ct += bytes([c])
            state = state[1:] + bytes([c])
        return IV + ct

    def decrypt(self, ciphertext):
        IV = ciphertext[:16]
        ct = ciphertext[16:]
        cipher = AES.new(self.key, AES.MODE_ECB)
        pt = b''
        state = IV
        for i in range(len(ct)):
            b = cipher.encrypt(state)[0]
            c = b ^ ct[i]
            pt += bytes([c])
            state = state[1:] + bytes([ct[i]])
        return pt


class Challenge():
    def __init__(self):
        self.before_input = "Please authenticate to this Domain Controller to proceed\n"
        self.password = urandom(20)
        self.password_length = len(self.password)
        self.cipher = CFB8(urandom(16))

    def challenge(self, your_input):
        if your_input['option'] == 'authenticate':
            if 'password' not in your_input:
                return {'msg': 'No password provided.'}
            your_password = your_input['password']
            if your_password.encode() == self.password:
                self.exit = True
                return {'msg': 'Welcome admin, flag: ' + FLAG}
            else:
                return {'msg': 'Wrong password.'}

        if your_input['option'] == 'reset_connection':
            self.cipher = CFB8(urandom(16))
            return {'msg': 'Connection has been reset.'}

        if your_input['option'] == 'reset_password':
            if 'token' not in your_input:
                return {'msg': 'No token provided.'}
            token_ct = bytes.fromhex(your_input['token'])
            if len(token_ct) < 28:
                return {'msg': 'New password should be at least 8-characters long.'}

            token = self.cipher.decrypt(token_ct)
            new_password = token[:-4]
            self.password_length = bytes_to_long(token[-4:])
            self.password = new_password[:self.password_length]
            return {'msg': 'Password has been correctly reset.'}


import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
listener.start_server(port=13399)
```

##### b) Phân tích kết quả:
- Đây là challenge về CFB8 mode (Cipher Feedback mode với block size 8 - bit) trong AES, có thể reset mật khẩu bằng cách tạo token giả mạo.
- CFB8: Mỗi byte của plaintext được mã hoá bằng byte đầu tiên của `AES - ECB (state)`: $$C_i = P_i \oplus MSB_1(E_K(S_i))$$ Cập nhật lại state: $$S_{i + 1} = leftshift(S_i) + C_i$$ Trong đó:
    - $C_i$: Ciphertext tại byte thứ $i$
    - $E_K$: Phép mã hoá AES với key $K$.
    - $S_i$: State hiện tại, bắt đầu bằng $IV$ ($16$ byte)
    - $MSB_1$: Lấy một byte đầu sau khi mã hoá AES.
    - $leftshift(S_i)$: Bỏ byte đầu của $S_i$, dịch trái một byte.
![image](https://hackmd.io/_uploads/H17RMamR1g.png)
- Chức năng của server trong challenge:
    - `authenticate`: So sánh `your_password.encode()` với `self.password`
    - `reset_connection`: Tạo lại AES key (vô hiệu hoá reuse keystream).
    - `reset_password`: Giải mã `token_ct` bằng AES - CFB8 để lấy passwword mới và length.
    ```py
    token = self.cipher.decrypt(token_ct)
    new_password = token[:-4]
    self.password_length = bytes_to_long(token[-4:])
    self.password = new_password[:self.password_length]
    ```
- Nếu gửi `token` là chuỗi lặp lại $28$ kí tự, `password` cũng là chuỗi lặp lại $8$ kí tự, nên idea sẽ là brute - force $8$ kí tự này đến khi tìm được Flag.
- Khi gửi một `token` có giá trị là `exploit = b'\x00'*28`, khi đó:
    - `token = new_password || bytes_length` ($4$ bytes)
    - `token_ct = IV (16 byte) + ciphertext (12 byte)`
- Khi giải mã trong CFB8: $$P_i = C_i \oplus MSB_1(E_K(S_i))$$ Nhưng vì $C_i = 0$: $$P_i = MSB_1(E_K(S_i))$$ Hay ta thu được keystream rõ ràng sau giải mã: $$plaintext = keystream$$
- Gửi thử `password` là chuỗi rỗng `""`. Sau khi reset password, nếu plaintext của `token` có `password_length = 0` thì đúng là password rỗng. Ngược lại, nếu brute - force không thành công thì gọi `reset_connection` để server tạo lại AES Key mới $\rightarrow$ Kết quả giải mã `b'\x00'*28` cũng thay đổi.
- Source code:
```py
from pwn import *
import json

t = (b"\x00" * 28).hex()
data = {"option": "reset_password", "token": t}
r = connect('socket.cryptohack.org', 13399)
r.recvline()
json_data = json.dumps(data).encode()

while True:
    r.sendline(json_data)
    r.recvline()
    p = {"option": "authenticate", "password": ""}
    json_password = json.dumps(p).encode()
    r.sendline(json_password)
    res = (r.recvline()).decode()
    if "crypto" in res:
        print(res)
        break
    reset = {"option": "reset_connection"}
    json_reset = json.dumps(reset).encode()
    r.sendline(json_reset)
    r.recvline()
```
- Kết quả:
```
{"msg": "Welcome admin, flag: crypto{Zerologon_Windows_CVE-2020-1472}"}
```

##### c) Kết quả:
`crypto{Zerologon_Windows_CVE-2020-1472}`

#### 5. Stream of Consciousness:
##### a) Mô tả đề bài:
- *"Nói chuyện với tôi và nghe một câu từ dòng ý thức được mã hóa của tôi."*
- Link: https://aes.cryptohack.org/stream_consciousness
- Source code:
```py
from Crypto.Cipher import AES
from Crypto.Util import Counter
import random

KEY = ?
TEXT = ['???', '???', ..., FLAG]

@chal.route('/stream_consciousness/encrypt/')
def encrypt():
    random_line = random.choice(TEXT)

    cipher = AES.new(KEY, AES.MODE_CTR, counter=Counter.new(128))
    encrypted = cipher.encrypt(random_line.encode())

    return {"ciphertext": encrypted.hex()}
```

##### b) Phân tích cách làm:
- Theo source code, `KEY` là cố định và ciphertext lấy được từ `encrypt()` sau mỗi lần gửi sẽ là một giá trị ngẫu nhiên tương ứng với lựa chọn từ `TEXT`.
- Cách hoạt động của stream cipher: $$C = P \oplus keystream$$ $$P = C \oplus keystream$$ Giả sử có: $$C_1 = P_1 \oplus keystream$$ $$C_2 = P_2 \oplus keystream$$ Khi đó: $$C_1 \oplus C_2 = P_1 \oplus P_2$$
- Gửi requests lên server để lấy danh sách ciphertext tương ứng với `TEXT` (loại bỏ ciphertext trùng lặp).
- Với một phần flag đã biết là `crypto{`, đặt nó làm $P_i$, vì challenge này hoạt động với CTR và `KEY` yếu nên: $$temp = P_i \oplus P_{i - 1} = C_i \oplus C_{i - 1}$$ Suy ra: $$P_{i - 1} = temp \oplus P_i$$
- Lấy tổ hợp chập hai từ danh sách ciphertext thu được và XOR như trên, ta được một danh sách plaintext vụn nhặt:
:::spoiler Plaintext
```py
b'nr\x7fo\x1d\x18d'
b'mioo\x1d\x18`'
b"hun<'\x15~"
b'Our? Wh'
b'{oj;tY3'
b'es+,5V4'
b"e'x'5T\x7f"
b'xoy*1\x18q'
b'mt+&2\x18Z'
b"bh'o\x1d\x1f\x7f"
b'dh|o$J|'
b'{oro0W3'
b"e'x'5T\x7f"
b'xono ]a'
b'{h~#0\x18Z'
b"|by'5H`"
b'e fo!V{'
b'`h}*x\x18c'
b'hhg#-\x18d'
b'xon<1\x18{'
b'{oj;tY3'
b'`iipto\x7f'
b'euh#Nba'
b'But I w'
b'vol$\x1d.,'
b'hs-3\\!+'
b"h'~8\\#`"
b'uo\x7f5Xon'
b'`t-9[oE'
b'oh!pth`'
b'ihzpM=c'
b'votpY ,'
b"h'~8\\#`"
b'uohpI*~'
b'vhx<YoE'
b'qb\x7f8\\?\x7f'
b'h `pH!d'
b'mh{5\x11o|'
b'eha<Do{'
b'uoh#Xod'
b'vol$\x1d.,'
b'fnx#Nbe'
b'And I s'
b'ut|$\x1d.('
b'kh=3\\!/'
b'k<n8\\#d'
b'vto5Xoj'
b'co=9[oA'
b'ls1pthd'
b'jsjpM=g'
b'utdpY ('
b'k<n8\\#d'
b'vtxpI*z'
b'ush<YoA'
b'ryo8\\?{'
b'k;ppH!`'
b'nsk5\x11ox'
b'fsq<Do\x7f'
b'vtx#Xo`'
b'ut|$\x1d.('
b'Dress-m'
b"ph}w'#6"
b'nt<`f,1'
b'n okf.z'
b'shnfbbt'
b'fs<jab_'
b'io0#Nez'
b'ook#w0y'
b'phe#c-6'
b'n okf.z'
b"shy#s'd"
b'poiocb_'
b'wenkf2e'
b"n'q#r,~"
b'kojf+bf'
b'copo~ba'
b'shypbb~'
b"ph}w'#6"
b'What a '
b"It can'"
b'I shall'
b'Three b'
b'As if I'
b"No, I'l"
b'How pro'
b'Why do '
b'I shall'
b'The ter'
b'Would I'
b'Perhaps'
b"I'm unh"
b'Love, p'
b'Dolly w'
b'These h'
b'What a '
b'}n8g5`|'
b'}:kl5b7'
b'`rja1.9'
b'ui8m2.\x12'
b'zu4$\x1d)7'
b'|uo$$|4'
b'cra$0a{'
b'}:kl5b7'
b'`r}$ k)'
b'cumh0.\x12'
b'd\x7fjl5~('
b'}=u$!`3'
b'xunax.+'
b'puth-.,'
b'`r}w1.3'
b'crypto{'
b'c&*{tm0'
b'~n+vp!>'
b'kuyzs!\x15'
b'diu3\\&0'
b'bi.3es3'
b'}n 3qn|'
b'c&*{tm0'
b'~n<3ad.'
b'}i,\x7fq!\x15'
b'zc+{tq/'
b'c!43`o4'
b'fi/v9!,'
b'ni5\x7fl!+'
b'~n<`p!4'
b'}n8g5`|'
b'~:x}p#u'
b'k!*qs#^'
b'd=&8\\${'
b'b=}8eqx'
b'}:s8ql7'
b'crypto{'
b'~:o8afe'
b'}=\x7ftq#^'
b'z7xptsd'
b'cug8`m\x7f'
b'f=|}9#g'
b'n=ftl#`'
b'~:okp#\x7f'
b'}:kl5b7'
b'vi+|woP'
b"yu'5Xhu"
b'\x7fu|5a=v'
b'`rr5u 9'
b'~:x}p#u'
b'crn5e*k'
b'`u~yuoP'
b'g\x7fy}p?j'
b'~=f5d!q'
b'{u}p=oi'
b'sugyhon'
b'crnftoq'
b'`rja1.9'
b'lnu9[h^'
b'jn.9b=]'
b'ui 9v \x12'
b'k!*qs#^'
b'vi<9f*@'
b'un,uvo{'
b'rd+qs?A'
b'k&49g!Z'
b'nn/|>oB'
b'fn5ukoE'
b'vi<jwoZ'
b'ui8m2.\x12'
b'er"pM:x'
b"zu,pY'7"
b'd=&8\\${'
b'yu0pI-e'
b'zr <Yh^'
b"}x'8\\8d"
b'd:8pH&\x7f'
b'ar#5\x11hg'
b'ir9<Dh`'
b'yu0#Xh\x7f'
b'zu4$\x1d)7'
b'|uwp`r4'
b'b=}8eqx'
b'\x7fukppxf'
b'|r{<`=]'
b'{x|8emg'
b'b:cpqs|'
b'grx5(=d'
b'orb<}=c'
b'\x7fuk#a=|'
b'|uo$$|4'
b'}:s8ql7'
b'`repde)'
b'cuu<t \x12'
b'd\x7fr8qp('
b'}=mpen3'
b'xuv5< +'
b'pul<i ,'
b'`re#u 3'
b'cra$0a{'
b'~:o8afe'
b'}=\x7ftq#^'
b'z7xptsd'
b'cug8`m\x7f'
b'f=|}9#g'
b'n=ftl#`'
b'~:okp#\x7f'
b'}:kl5b7'
b'`ui<d*@'
b'g\x7fn8azz'
b'~=qpuda'
b'{uj5,*y'
b'sup<y*~'
b'cry#e*a'
b'`r}$ k)'
b'dx~tq?A'
b'}:a<e!Z'
b'xrzy<oB'
b'pr`pioE'
b'`uiouoZ'
b'cumh0.\x12'
b'z0f8`q`'
b'\x7fx}}9?x'
b'wxgtl?\x7f'
b'g\x7fnkp?`'
b'd\x7fjl5~('
b'f:b5-!c'
b'n:x<x!d'
b'~=q#d!{'
b'}=u$!`3'
b'krcy!o|'
b'{ujf=oc'
b'xunax.+'
b'supohod'
b'puth-.,'
b'`r}w1.3'
```
:::
- Source code:
```py
import requests
from pwn import xor
from itertools import combinations

def encrypt():
    url = "https://aes.cryptohack.org/stream_consciousness/encrypt/"
    r = requests.get(url).json()
    return bytes.fromhex(r["ciphertext"])

C = []

for i in range(1000):
    ct = encrypt()
    if ct not in C:
        C.append(ct)

C = sorted(C, key=len)
flag = b'crypto{'
stream = list(combinations(C, 2))

for c in stream:
    temp = xor(c[0], c[1])[:len(flag)]
    res = xor(temp, flag)
    print(res)
```
- Chú ý có đoạn khá rõ nghĩa:
```py
b'What a '
b"It can'"
b'I shall'
b'Three b'
b'As if I'
b"No, I'l"
b'How pro'
b'Why do '
b'I shall'
b'The ter'
b'Would I'
b'Perhaps'
b"I'm unh"
b'Love, p'
b'Dolly w'
b'These h'
```
Ta thấy có một số từ ta có thể đoán được như `I'm unhappy`. Thay từ trên vào `flag = b'crypto{` $\rightarrow$ `flag = b'I'm unhappy'` và cho chạy source, cứ tiếp tục đoán như thế cho đến khi tìm ra Flag:
```py
flag = b"I'm unhappy"
flag = b"It can't be "
flag = b"I shall, I'll "
flag = b"Why do they go "
flag = b"As if I had any "
flag = b"Why do they go on "
flag = b"Would I have believed "
flag = b"What a lot of things that "
flag = b"These horses, this carriage "
flag = b"I shall, I'll lose everything "
flag = b"Love, probably? They don't know "
flag = b"Dolly will think that I'm leaving "
flag = b"No, I'll go in to Dolly and tell her "
```
::: spoiler Source code
```py
import requests
from pwn import xor
from itertools import combinations
from tqdm import trange

def encrypt():
    url = "https://aes.cryptohack.org/stream_consciousness/encrypt/"
    r = requests.get(url).json()
    return bytes.fromhex(r["ciphertext"])

C = []

for i in trange(30):
    ct = encrypt()
    if ct not in C:
        C.append(ct)

C = sorted(C, key=len)
flag = b"No, I'll go in to Dolly and tell her "
stream = list(combinations(C, 2))

for c in stream:
    temp = xor(c[0], c[1])[:len(flag)]
    res = xor(temp, flag)
    print(res)
```
:::
- Tới đây, ta đã tìm ra `crypto{k3y57r34m_r3u53_15_f474l}`

##### c) Kết quả:
`crypto{k3y57r34m_r3u53_15_f474l}`

#### 6. Dancing Queen:
##### a) Mô tả đề bài:
- *"Tôi không tin tưởng các developer khác vì vậy tôi đã thực hiện triển khai Chacha20 của riêng mình. Bằng cách này, tôi chắc chắn bạn sẽ không bao giờ có thể đọc Flag của tôi!"*
- Source code:
```py
#!/usr/bin/env python3

from os import urandom


FLAG = b'crypto{?????????????????????????????}'


def bytes_to_words(b):
    return [int.from_bytes(b[i:i+4], 'little') for i in range(0, len(b), 4)]

def rotate(x, n):
    return ((x << n) & 0xffffffff) | ((x >> (32 - n)) & 0xffffffff)

def word(x):
    return x % (2 ** 32)

def words_to_bytes(w):
    return b''.join([i.to_bytes(4, 'little') for i in w])

def xor(a, b):
    return b''.join([bytes([x ^ y]) for x, y in zip(a, b)])


class ChaCha20:
    def __init__(self):
        self._state = []

    def _inner_block(self, state):
        self._quarter_round(state, 0, 4, 8, 12)
        self._quarter_round(state, 1, 5, 9, 13)
        self._quarter_round(state, 2, 6, 10, 14)
        self._quarter_round(state, 3, 7, 11, 15)
        self._quarter_round(state, 0, 5, 10, 15)
        self._quarter_round(state, 1, 6, 11, 12)
        self._quarter_round(state, 2, 7, 8, 13)
        self._quarter_round(state, 3, 4, 9, 14)

    def _quarter_round(self, x, a, b, c, d):
        x[a] = word(x[a] + x[b]); x[d] ^= x[a]; x[d] = rotate(x[d], 16)
        x[c] = word(x[c] + x[d]); x[b] ^= x[c]; x[b] = rotate(x[b], 12)
        x[a] = word(x[a] + x[b]); x[d] ^= x[a]; x[d] = rotate(x[d], 8)
        x[c] = word(x[c] + x[d]); x[b] ^= x[c]; x[b] = rotate(x[b], 7)
    
    def _setup_state(self, key, iv):
        self._state = [0x61707865, 0x3320646e, 0x79622d32, 0x6b206574]
        self._state.extend(bytes_to_words(key))
        self._state.append(self._counter)
        self._state.extend(bytes_to_words(iv))

    def decrypt(self, c, key, iv):
        return self.encrypt(c, key, iv)

    def encrypt(self, m, key, iv):
        c = b''
        self._counter = 1

        for i in range(0, len(m), 64):
            self._setup_state(key, iv)
            for j in range(10):
                self._inner_block(self._state)
            c += xor(m[i:i+64], words_to_bytes(self._state))

            self._counter += 1
        
        return c
    

if __name__ == '__main__':
    msg = b'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula.'
    key = urandom(32)
    iv1 = urandom(12)
    iv2 = urandom(12)

    c = ChaCha20()
    msg_enc = c.encrypt(msg, key, iv1)
    flag_enc = c.encrypt(FLAG, key, iv2)

    print(f"iv1 = '{iv1.hex()}'")
    print(f"iv2 = '{iv2.hex()}'")
    print(f"msg_enc = '{msg_enc.hex()}'")
    print(f"flag_enc = '{flag_enc.hex()}'")
```
- Output:
```py
iv1 = 'e42758d6d218013ea63e3c49'
iv2 = 'a99f9a7d097daabd2aa2a235'
msg_enc = 'f3afbada8237af6e94c7d2065ee0e221a1748b8c7b11105a8cc8a1c74253611c94fe7ea6fa8a9133505772ef619f04b05d2e2b0732cc483df72ccebb09a92c211ef5a52628094f09a30fc692cb25647f'
flag_enc = 'b6327e9a2253034096344ad5694a2040b114753e24ea9c1af17c10263281fb0fe622b32732'
```

##### b) Phân tích cách làm:
- **ChaCha20** là một thuật toán mã hoá dòng được thiết kế bởi Daniel J. Bernstein. Tuy nhiên, trong source code challenge có một lỗi nghiêm trọng làm nó không được an toàn như ChaCha20 gốc.
    - Cách hoạt động của ChaCha20 gốc: Sau khi `_setup_state`, quá trình mã hoá phải:
        - Sao chép trạng thái ra biến `working_state`.
        - Thực hiện $20$ vòng ChaCha trên `working_state`.
        - Cộng `working_state` với trạng thái ban đầu để tạo keystream.
        - XOR keystream với plaintext để tạo ciphertext.
    - Nhưng trong challenge: Thực hiện $20$ vòng ChaCha trực tiếp lên `self.state` và sử dụng chính nó là keystream $\rightarrow$ Vô hiệu hoá phần lớn tính an toàn của ChaCha20.
- Từ source code challenge, các hàm được làm một chiều nên chỉ cần đảo ngược các hàm lại:
::: spoiler reverse_inner_block()
```py
def reverse_inner_block(self, state):
    self.reverse_quarter_round(state, 3, 4, 9, 14)
	self.reverse_quarter_round(state, 2, 7, 8, 13)
	self.reverse_quarter_round(state, 1, 6, 11, 12)
	self.reverse_quarter_round(state, 0, 5, 10, 15)
	self.reverse_quarter_round(state, 3, 7, 11, 15)
	self.reverse_quarter_round(state, 2, 6, 10, 14)
	self.reverse_quarter_round(state, 1, 5, 9, 13)
	self.reverse_quarter_round(state, 0, 4, 8, 12)
```
:::
::: spoiler reverse_quarter_round()
```py
def reverse_quarter_round(self,x, a, b, c, d):
	x[b] = rotate(x[b], 32-7)
	x[b] ^= x[c]
	x[c] = word(x[c] - x[d])

	x[d] = rotate(x[d], 32-8)
	x[d] ^= x[a]
	x[a] = word(x[a] - x[b])

	x[b] = rotate(x[b], 32-12)
	x[b] ^= x[c]
	x[c] = word(x[c] - x[d])

	x[d] = rotate(x[d], 32-16)
	x[d] ^= x[a]
	x[a] = word(x[a] - x[b])
```
:::
- Phân tích hàm `_setup_state()`:
::: spoiler _setup_state()
```py
def _setup_state(self, key, iv):
	self._state = [0x61707865, 0x3320646e, 0x79622d32, 0x6b206574]
	self._state.extend(bytes_to_words(key))
	self._state.append(self._counter)
	self._state.extend(bytes_to_words(iv)
```
:::
State gồm:
    - Chuỗi biểu diễn đầu tiên là $4$ giá trị cố định.
    - Tiếp theo là $32$ bytes của `key`.
    - Cuối cùng là $12$ bytes của `iv`.
Nếu tìm được state ban đầu thì sẽ tìm được `key`.
- Phân tích hàm `encrypt()`:
::: spoiler encrypt()
```py
def encrypt(self, m, key, iv):
	c = b''
	self._counter = 1

	for i in range(0, len(m), 64):
		self._setup_state(key, iv)
		for j in range(10):
			self._inner_block(self._state)
		c += xor(m[i:i+64], words_to_bytes(self._state))
		self._counter += 1

	return c
```
:::
Xét $i$ lần lặp, mỗi lần sẽ:
    - `_setup_state()`: state chỉ thay đổi mỗi giá trị `counter`, giá trị `key` và `iv` không thay đổi.
    - `inner_block()` mười lần.
    - Kết quả sẽ `word_to_bytes` state và XOR với $64$ bytes của `msg`.
    - `counter` tăng một đơn vị.
Ta sẽ tìm `key` bằng cách đảo ngược lại hàm `encrypt()` (thao tác với $64$ bytes đầu tiên vì chỉ có `counter` thay đổi).
::: spoiler find_key()
```py
def find_key(self, msg, msg_enc):
	self._state = bytes_to_words(xor(msg, msg_enc)[:64])
	for j in range(10):
		self.reverse_inner_block(self._state)
	key = words_to_bytes(self._state[4:])[:32]
	return key
```
:::
- Source code:
```py
def bytes_to_words(b):
    return [int.from_bytes(b[i:i+4], 'little') for i in range(0, len(b), 4)]

def rotate(x, n):
    return ((x << n) & 0xffffffff) | ((x >> (32 - n)) & 0xffffffff)

def word(x):
    return x % (2**32)

def words_to_bytes(w):
    return b''.join([i.to_bytes(4, 'little') for i in w])

def xor(a, b):
    return b''.join([bytes([x ^ y]) for x, y in zip(a, b)])


class ChaCha20:
	def __init__(self):
		self._state = []

	def _inner_block(self, state):
		self._quarter_round(state, 0, 4, 8, 12)
		self._quarter_round(state, 1, 5, 9, 13)
		self._quarter_round(state, 2, 6, 10, 14)
		self._quarter_round(state, 3, 7, 11, 15)
		self._quarter_round(state, 0, 5, 10, 15)
		self._quarter_round(state, 1, 6, 11, 12)
		self._quarter_round(state, 2, 7, 8, 13)
		self._quarter_round(state, 3, 4, 9, 14)

	def reverse_inner_block(self, state):
		self.reverse_quarter_round(state, 3, 4, 9, 14)
		self.reverse_quarter_round(state, 2, 7, 8, 13)
		self.reverse_quarter_round(state, 1, 6, 11, 12)
		self.reverse_quarter_round(state, 0, 5, 10, 15)
		self.reverse_quarter_round(state, 3, 7, 11, 15)
		self.reverse_quarter_round(state, 2, 6, 10, 14)
		self.reverse_quarter_round(state, 1, 5, 9, 13)
		self.reverse_quarter_round(state, 0, 4, 8, 12)

	def _quarter_round(self, x, a, b, c, d):
		x[a] = word(x[a] + x[b]); x[d] ^= x[a]; x[d] = rotate(x[d], 16)
		x[c] = word(x[c] + x[d]); x[b] ^= x[c]; x[b] = rotate(x[b], 12)
		x[a] = word(x[a] + x[b]); x[d] ^= x[a]; x[d] = rotate(x[d], 8)
		x[c] = word(x[c] + x[d]); x[b] ^= x[c]; x[b] = rotate(x[b], 7)
		
	def reverse_quarter_round(self,x, a, b, c, d):
		x[b] = rotate(x[b], 32-7)
		x[b] ^= x[c]
		x[c] = word(x[c] - x[d])

		x[d] = rotate(x[d], 32-8)
		x[d] ^= x[a]
		x[a] = word(x[a] - x[b])

		x[b] = rotate(x[b], 32-12)
		x[b] ^= x[c]
		x[c] = word(x[c] - x[d])

		x[d] = rotate(x[d], 32-16)
		x[d] ^= x[a]
		x[a] = word(x[a] - x[b])
    
	def _setup_state(self, key, iv):
		self._state = [0x61707865, 0x3320646e, 0x79622d32, 0x6b206574]
		self._state.extend(bytes_to_words(key))
		self._state.append(self._counter)
		self._state.extend(bytes_to_words(iv))

	def decrypt(self, c, key, iv):
		return self.encrypt(c, key, iv)

	def encrypt(self, m, key, iv):
		c = b''
		self._counter = 1

		for i in range(0, len(m), 64):
			self._setup_state(key, iv)
			for j in range(10):
				self._inner_block(self._state)
			c += xor(m[i:i+64], words_to_bytes(self._state))
			self._counter += 1

		return c

	def find_key(self, msg, msg_enc):
		self._state = bytes_to_words(xor(msg, msg_enc)[:64])
		for j in range(10):
			self.reverse_inner_block(self._state)
		key = words_to_bytes(self._state[4:])[:32]
		return key


c = ChaCha20()
msg = b'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula.'
iv1 = 'e42758d6d218013ea63e3c49'
iv2 = 'a99f9a7d097daabd2aa2a235'
msg_enc = 'f3afbada8237af6e94c7d2065ee0e221a1748b8c7b11105a8cc8a1c74253611c94fe7ea6fa8a9133505772ef619f04b05d2e2b0732cc483df72ccebb09a92c211ef5a52628094f09a30fc692cb25647f'
flag_enc = 'b6327e9a2253034096344ad5694a2040b114753e24ea9c1af17c10263281fb0fe622b32732'

iv1 = bytes.fromhex(iv1)
iv2 = bytes.fromhex(iv2)
msg_enc = bytes.fromhex(msg_enc)
flag_enc = bytes.fromhex(flag_enc)

key = c.find_key(msg, msg_enc)
flag = c.decrypt(flag_enc, key, iv2)
print(flag)
```

##### c) Kết quả:
`crypto{M1x1n6_r0und5_4r3_1nv3r71bl3!}`

### V. PADDING ATTACKS:
#### 1. Pad Thai:
##### a) Mô tả đề bài:
- *"Đôi khi những challenges cổ điển có thể dễ chịu nhất"*
- Kết nối tại `socket.cryptohack.org 13421`
- Source code:
```py
#!/usr/bin/env python3

from Crypto.Util.Padding import unpad
from Crypto.Cipher import AES
from os import urandom

from utils import listener

FLAG = 'crypto{?????????????????????????????????????????????????????}'

class Challenge:
    def __init__(self):
        self.before_input = "Let's practice padding oracle attacks! Recover my message and I'll send you a flag.\n"
        self.message = urandom(16).hex()
        self.key = urandom(16)

    def get_ct(self):
        iv = urandom(16)
        cipher = AES.new(self.key, AES.MODE_CBC, iv=iv)
        ct = cipher.encrypt(self.message.encode("ascii"))
        return {"ct": (iv+ct).hex()}

    def check_padding(self, ct):
        ct = bytes.fromhex(ct)
        iv, ct = ct[:16], ct[16:]
        cipher = AES.new(self.key, AES.MODE_CBC, iv=iv)
        pt = cipher.decrypt(ct)  # does not remove padding
        try:
            unpad(pt, 16)
        except ValueError:
            good = False
        else:
            good = True
        return {"result": good}

    def check_message(self, message):
        if message != self.message:
            self.exit = True
            return {"error": "incorrect message"}
        return {"flag": FLAG}

    #
    # This challenge function is called on your input, which must be JSON
    # encoded
    #
    def challenge(self, msg):
        if "option" not in msg or msg["option"] not in ("encrypt", "unpad", "check"):
            return {"error": "Option must be one of: encrypt, unpad, check"}

        if msg["option"] == "encrypt": return self.get_ct()
        elif msg["option"] == "unpad": return self.check_padding(msg["ct"])
        elif msg["option"] == "check": return self.check_message(msg["message"])

import builtins; builtins.Challenge = Challenge # hack to enable challenge to be run locally, see https://cryptohack.org/faq/#listener
listener.start_server(port=13421)
```

##### b) Phân tích cách làm:
- Từ source code đề, ta thấy đây là Padding Oracle Attack AES - CBC. CBC mã hoá từng khối theo công thức: $$C_i = AES_{Encrypt}(P_i \oplus C_{i - 1})$$ Với:
    - $C_0 = IV$
    - $C_1 = AES(P_1 \oplus IV)$
    - $C_2 = AES(P_2 \oplus C_1)$
    - ...
    - $P_i = AES_{Decrypt}(C_i) \oplus C_{i - 1}$
- Do `ct` server trả về dài $48$ byte nên `ct = iv | c1 | c2`
##### c) Kết quả:
### VI. AUTHENTICATED ENCRYPTION:
#### 1. Paper Plane:
##### a) Mô tả đề bài:
- *"Tôi đã tìm thấy một chế độ mã hóa được xác thực được gọi là tiện ích mở rộng Garble Infinite trong đó lỗi trong một bit sẽ làm hỏng phần còn lại của bản mã. Có vẻ như là một cách tuyệt vời để bảo vệ các thông điệp trò chuyện khỏi các cuộc tấn công bằng cách sử dụng bitflipping, padding oracles, v.v.?"*
- Link: https://aes.cryptohack.org/paper_plane
- Source code:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import os


KEY = ?
FLAG = ?


class AesIge:
    def __init__(self, key):
        self.cipher = AES.new(key, AES.MODE_ECB)

    def encrypt(self, data, m0=os.urandom(16), c0=os.urandom(16)):
        data = pad(data, 16, 'pkcs7')

        last_block_plaintext = m0
        last_block_ciphertext = c0
        result = b''
        for i in range(0, len(data), 16):
            block = data[i: i + 16]
            x = AesIge._xor_blocks(block, last_block_ciphertext)
            x = self.cipher.encrypt(x)
            x = AesIge._xor_blocks(x, last_block_plaintext)
            result += x

            last_block_plaintext = block
            last_block_ciphertext = x

        return result, m0, c0

    def decrypt(self, data, m0, c0):
        last_block_plaintext = m0
        last_block_ciphertext = c0
        result = b''

        for i in range(0, len(data), 16):
            block = data[i: i + 16]
            x = AesIge._xor_blocks(block, last_block_plaintext)
            x = self.cipher.decrypt(x)
            x = AesIge._xor_blocks(x, last_block_ciphertext)
            result += x

            last_block_ciphertext = block
            last_block_plaintext = x

        if AesIge._is_pkcs7_padded(result):
            return unpad(result, 16, 'pkcs7')
        else:
            return None

    @staticmethod
    def _is_pkcs7_padded(message):
        padding = message[-message[-1]:]
        return all(padding[i] == len(padding) for i in range(0, len(padding)))

    @staticmethod
    def _xor_blocks(a, b):
        return bytes([x ^ y for x, y in zip(a, b)])



@chal.route('/paper_plane/encrypt_flag/')
def encrypt_flag():
    ciphertext, m0, c0 = AesIge(KEY).encrypt(FLAG.encode())
    return {"ciphertext": ciphertext.hex(), "m0": m0.hex(), "c0": c0.hex()}


@chal.route('/paper_plane/send_msg/<ciphertext>/<m0>/<c0>/')
def send_msg(ciphertext, m0, c0):
    ciphertext = bytes.fromhex(ciphertext)
    m0 = bytes.fromhex(m0)
    c0 = bytes.fromhex(c0)
    if len(ciphertext) % 16 != 0:
        return {"error": "Data length must be a multiple of the blocksize!"}
    if len(c0) != 16 or len(m0) != 16:
        return {"error": "m0 and c0 must be 16 bytes long!"}

    plaintext = AesIge(KEY).decrypt(ciphertext, m0, c0)
    if plaintext is not None:
        return {"msg": "Message received"}
    else:
        return {"error": "Can't decrypt the message."}
```

##### b) Phân tích cách làm:
- Đây là challenge về **AES - IGE (Infinite Garble Extension)** tự cài đặt bằng AES - ECB làm nền tảng.
- Thu được:
```py
c0 = "0dc3a7ec3c06af23001a8f2919218708"
ciphertext = "9d864e9f1c0e9f80ae76929e434840970dde6200fe6805e067727b04504f7925"
m0 = "76eb7f9dedc35b2be9051537646cc022"
```
##### c) Kết quả:
### VII. LINEAR CRYPTANALYSIS:
