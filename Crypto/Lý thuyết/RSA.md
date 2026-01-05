## LÝ THUYẾT:
### I. Khái niệm:
#### 1. Bài toán: 
- Đối với các hệ thống giao tiếp giữa hai endpoint thông qua một server trung gian (hệ thống HelpDesk, chat client - server,...)
![image](https://hackmd.io/_uploads/ryVP9W7zgx.png)
- Có thể áp dụng cơ chế như SSL\TLS để xác thực hai chiều, mã hoá kênh truyền nhưng phương pháp này chỉ đảm bảo an toàn giao tiếp giữa client - server và vẫn tiềm toàn nguy cơ bị tấn công khi kẻ gian kiểm soát được đường truyền mạng giữa hai bên hay kiểm soát được chính server.
$\rightarrow$ Yêu cầu đặt ra phương pháp trao đổi hoá mã hoá chung giữa hai đầu endpoint (client) và đảm bảo rằng key mã hoá chung chỉ có hai client biết với nhau (hoàn toàn bí mật đối với server hoặc kẻ gian nằm giữa kênh truyền).

#### 2. Diffie - Hellman:
- Là phương pháp trao đổi khoá cho phép hai bên client thiết lập một khoá bí mật chung để mã hoá dữ liệu sử dụng trên kênh truyền thông không an toàn mà hai bên không cần có sự thoả thuận trước về khoá bí mật. Khoá bí mật tạo ra sẽ được sử dụng để mã hoá dữ liệu với phương pháp mã hoá khoá đối xứng.
- Mặc dù là giao thức trao đổi khoá ẩn danh (không xác thực), Diffie - Hellman đã đưa ra một nền tảng cơ sở cho nhiều loại giao thức xác thực và được sử dụng để tạo nên bí mật chuyển tiếp hoàn hảo trong chế độ ngắn hạn của giao thức Transport Layer Security (EDH hoặc DHE tuỳ bộ mã hoá).

### II. Mô tả thuật toán:
#### 1. Ý tưởng cơ bản:
- Alice và Bob trao đổi màu sơn bí mật thông qua hỗn hợp sơn:
    - Đầu tiên cả hai trộn màu đã biết chung (vàng) với màu bí mật riêng của mỗi người.
    - Mỗi người chuyển hỗn hợp của mình tới người kia thông qua một kênh vận chuyển công cộng.
    - Khi nhận được hỗn hợp của người kia, mỗi người sẽ trộn thêm với màu bí mật của riêng mình và nhận được hỗn hợp cuối cùng.
- Hỗn hợp sơn cuối cùng giống nhau và chỉ có hai người biết, người ngoài sẽ rất khó (về mặt tính toán) để tìm ra bí mật chung (hỗn hợp cuối).
![image](https://hackmd.io/_uploads/H1kX1M7Mlx.png)

#### 2. Dạng toán học:
![image](https://hackmd.io/_uploads/rJ7kpkNzgl.png)
- Thiết lập khoá:
    - Alice và Bob thoả thuận sử dụng chung một số nguyên tố $p$ và căn nguyên thuỷ $g$ (phần tử sinh trong nhóm $Z^*_p$).
    - Alice chọn một số nguyên bí mật $a$ và gửi cho Bob giá trị: $$A = g^a \mod p$$
    - Bob chọn một số nguyên bí mật $b$ và gửi cho Alice giá trị: $$B = g^b \mod p$$
    - Alice tính: $$s = B^a \mod p$$
    - Bob tính: $$s = A^b \mod p$$
    $\rightarrow$ Cả hai đều có giá trị chung cuối cùng: $$s = (g^a)^b \mod p = (g^b)^a \mod p$$ Sau khi Alice và Bob tính được bí mật chung, cả hai có thể sử dụng nó làm khoá mã hoá chung chỉ có hai người biết để gửi dữ liệu trên kênh truyền thông mở.
- Mã hoá: Thông điệp $m$ trước khi được gửi đi bởi Alice hoặc Bob sẽ được mã hoá thành $mg^{ab}$.
- Giải mã:
    - Thông điệp $m$ gửi dưới dạng bản mã: $$c = m.g^{ab}$$ Khi nhận được $c$ Alice hoặc Bob phải tính được: $$(g^ab)^{-1} = g^{-ab}$$ Và khôi phục thông điệp: $$m = c(g^{ab})^{-1} = m.g^{ab}.g^{-ab} = m$$

### III. Code minh hoạ:
```py
import hashlib
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from Crypto.Random import get_random_bytes

# HÀM KIỂM TRA PADDING:
def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))

# CÁC THÔNG SỐ CÔNG KHAI:
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919

# KHOÁ RIÊNG CỦA ALICE VÀ BOB:
a = 972107443837033796245864316200458246846904598488981605856765890478853088246897345487328491037710219222038930943365848626194109830309179393018216763327572120124760140018038673999837643377590434413866611132403979547150659053897355593394492586978400044375465657296027592948349589216415363722668361328689588996541370097559090335137676411595949335857341797148926151694299575970292809805314431447043469447485957669949989090202320234337890323293401862304986599884732815
b = 120192332529039903445985225357749630203957704094452967240343784334979768401678059705899609622219482909518733877281021159968314544822992432268394909997137634404121779658615087734205322664846191267105664149142275601037153366961932103798505750477303883783482661809349461391004798313398358965834436915293727039545890715077179171369067701220777398142622984886621380856087361034186017508616984173402642138677538346793591914270981958871120645031045104896104482944207206

# KHOÁ CÔNG KHAI:
A = pow(g, a, p)
B = pow(g, b, p)

# KHOÁ CHUNG:
s_Alice = pow(B, a, p)
s_Bob = pow(A, b, p)
s = s_Alice

# DẪN XUẤT KHOÁ AES TỪ SHARED SECRET (s) BẰNG SHA-1:
sha1 = hashlib.sha1()
sha1.update(str(s).encode('ascii'))
key = sha1.digest()[:16]

# MÃ HOÁ THÔNG ĐIỆP:
message = b"DH{ehehehehehe}"
iv = get_random_bytes(16)
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(pad(message, 16))

print("Ciphertext (hex):", ciphertext.hex())
print("IV (hex):", iv.hex())

# GIẢI MÃ AES - CBC:
def decrypt_flag(s: int, iv: str, ciphertext: str):
    ciphertext_bytes = bytes.fromhex(ciphertext)
    iv_bytes = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv_bytes)
    plaintext = cipher.decrypt(ciphertext_bytes)

    # Bỏ padding nếu có:
    if is_pkcs7_padded(plaintext):
        return unpad(plaintext, 16).decode('ascii')
    else:
        return plaintext.decode('ascii')

# GIẢI MÃ:
M = decrypt_flag(s, iv.hex(), ciphertext.hex())
print("Message: ", M)
```

### IV. CÁCH TẤN CÔNG:
#### 1. Pohlig - Hellman:
##### a) Lý thuyết:
- Là một kỹ thuật tấn công để giải bài toán logarit rời rạc (DLP) trong nhóm hữu hạn khi bậc của nhóm ($p - 1$) là một số có nhiều thừa số nhỏ (smooth).
- Trong Diffe - Hellman:
    - $p$: Số nguyên tố lớn
    - $g$: Số sinh modulo $p$
    - $A = g^a \mod p$: Khoá công khai
    - Mục tiêu: Tìm $a$ khi biết $A$, $g$, $p$
- Ý tưởng:
    - Giả sử $n = p - 1$
    - Phân tích $n$ thành các thừa số nguyên tố: $$n = \displaystyle \prod_{i = 1}^{k}{q_i^{e_i}}$$ Khi đó:
        - Giải bài toán logarit rời rạc trên từng modulo $q_i^{e_i}$: $$a \equiv a_i \mod q_i^{e_i}$$
            - Tính $g_i = g^{\frac{n}{q_i^{e_i}}} \mod p$
            - Tính $A_i = A^{\frac{n}{q_i^{e_i}}} \mod p$
            - Tìm $a_i$ sao cho $g_i^{a_i} \equiv A_i \mod p$$
            - Từ đó tìm được $a \mod q_i^{e_i}$
        - Sử dụng Chinese Remainder Theorem (CRT) để tái hợp các phần: $$a \equiv a_i \mod q_i^{e_i} \rightarrow a \mod n$$
            - Sau khi đã biết $$
\begin{aligned}
a &\equiv a_1 \mod q_1^{e_1} \\
a &\equiv a_2 \mod q_2^{e_2} \\
&\vdots \\
a &\equiv a_k \mod q_k^{e_k}
\end{aligned}
$$ Sử dụng CRT sẽ thu được nghiệm duy nhất modulo $n = p - 1$: $$a \mod n$$
            - Nếu: $$a \equiv a_i \mod m_i$$ Với $gcd(m_i, m_j) = 1$ cho mọi $i \neq j$.
            - Nghiệm tổng quát: $$a = \displaystyle \sum_{i = 1}^{k} a_i.M_i.y_i \mod n$$ Trong đó:
                - $M_i = \frac{n}{m_i}$
                - $y_i = M_i^{-1} \mod m_i$
> Ví dụ:
> - Giải phương trình: $$g^x \equiv h \mod p$$ Với:
> $g = 2$
> $h = 22$
> $p = 29$
> $n = p - 1 = 28 = 2^2.7$
> - Tìm $x_1 \equiv x \mod 2^2$:
> $\left\{
\begin{aligned}
> g_1 = g^{\frac{n}{2^2}} = 2^7 \mod 29 = 12 \\
> h_1 = h^{\frac{n}{2^2}} = 22^7 \mod 29 = 28
> \end{aligned}
> \right.
> \\
> \Rightarrow g_1^x \equiv h_1 \mod 29 \leftrightarrow 12^x \equiv 28 \mod 29 \rightarrow x = 2 \rightarrow x_1 = 2 \mod 2^2$
> - Tìm $x_2 \equiv x \mod 7$:
> $\left\{
\begin{aligned}
> g_2 = g^{\frac{n}{7}} = 2^4 \mod 29 = 16 \\
> h_2 = h^{\frac{n}{7}} = 22^4 \mod 29 = 23
> \end{aligned}
> \right.
> \\
> \Rightarrow g_2^x \equiv h_2 \mod 29 \leftrightarrow 16^x \equiv 23 \mod 29 \rightarrow x = 5 \rightarrow x_2 = 5 \mod 7$
> - Áp dụng CRT: 
> $\left\{
\begin{aligned}
> x = 2 \mod 4 \\
> x = 5 \mod 7
> \end{aligned}
> \right.
> \\
> \Rightarrow x = 26$

##### b) Ví dụ:
- Đề bài:
```py
from Crypto.Util.number import *

p = 72582273207584409523836416205503873456840672244861668902068962428022358668644213717033410220094858213785909158082084409697781833525734642650576180002727864094178853739590888445753712196268567738582111704293273201721006648707008913242412196989487167851968618303659742648153352883917176492356546118351747721810800909873736282570227177961197335450387989276806079489
g = 3
FLAG = b"CYBORG{???}"
a = pow(g, bytes_to_long(FLAG), p)
print(a)

"""
24393771488717960431147269064624631828310604373526026598603386491263061338072489803153972728250242949112187407825532440328751180404635401465476512488685185622725060580628770654048867200033806585934697471249921972700552978079752695585970921337459580789152970187925768085334409084092041192304935279345047595337816976845617649400223935358270007572542969925561362228
```
- Phân tích cách làm:
    - Bài toán yêu cầu tìm flag là số nguyên $x \in [0, p - 2]$ sao cho: $$a = g^x \mod p$$
    - Phân tích $p - 1$ thành tích của các thừa số nguyên tố (vì số cuối cùng quá lớn nên có thể bỏ đi):
    ```py
    factor = [2**10, 787**4, 32587**3, 708667**7, 19964029**6, 856892137**2, 1279562789201591523940850597505137258079950871699945159663662131835076279131726053889024495522041177924458398143694947568877887370555653768499066503948935672363148134562050374459082232131445656948264915239888005511288832804262243257]
    ```
    - Sử dụng thuật toán Pohlig - Hellman, source code chạy trong SageMath:
    ```py
    #!/usr/bin/env sage
    from Crypto.Util.number import long_to_bytes
    from sage.all import *

    p = 72582273207584409523836416205503873456840672244861668902068962428022358668644213717033410220094858213785909158082084409697781833525734642650576180002727864094178853739590888445753712196268567738582111704293273201721006648707008913242412196989487167851968618303659742648153352883917176492356546118351747721810800909873736282570227177961197335450387989276806079489
    n = p - 1
    g = 3
    a = 24393771488717960431147269064624631828310604373526026598603386491263061338072489803153972728250242949112187407825532440328751180404635401465476512488685185622725060580628770654048867200033806585934697471249921972700552978079752695585970921337459580789152970187925768085334409084092041192304935279345047595337816976845617649400223935358270007572542969925561362228
    factor = [2**10, 787**4, 32587**3, 708667**7, 19964029**6, 856892137**2]

    Fp = Zmod(p)

    residues = []
    mods = []
    for r in factor:
        e = ZZ(n) // ZZ(r)
        gr = Fp(g)**e
        ar = Fp(a)**e
        x_i = discrete_log(ar, gr, ord=Integer(r))
        residues.append(x_i)
        mods.append(r)
        print(f"log mod {r} = {x_i}")

    if len(residues) == len(mods) and len(residues) > 0:
        x = Integer(crt(residues, mods))
        flag = long_to_bytes(int(x))
        print(flag)
    ```
    - Giải thích source code: Chạy trong trường hữu hạn modulo $p$:
        - `e = ZZ(n) // ZZ(r)`: $e = \frac{p - 1}{r}$
        - `gr = Fp(g)**e`: $g^e \equiv g^r \mod p$ sinh ra một nhóm con cấp $r$.
        - `ar = Fp(a)**e`: $a^e \equiv a^r \mod p$ cũng thuộc nhóm con đó.
        - `x_i = discrete_log(ar, gr, ord=Integer(r))`: Giải phương trình logarit rời rạc: $$g_r^{x_i} \equiv a_r \mod p$$ Tức là tìm $x_i \equiv x \mod r$ và lưu lại các $(x_i, r)$.
        - `x = Integer(crt(residues, mods))`: Dùng CRT để tìm nghiệm duy nhất $x \mod (p - 1)$.
- Kết quả: `CYBORG{p0hl1g_h3llm4n_f7w!!}`

#### 2. Small Subgroup Attack:
##### a) Lý thuyết:
- Sử dụng khi Diffie - Hellman hoặc các giao thức mật mã sử dụng nhóm không đủ mạnh hoặc không kiểm tra đúng đầu vào.
- Mục tiêu: Trích xuất thông tin về khoá bí mật bằng cách lợi dụng các nhóm con nhỏ (small subgroups) trong nhóm sinh của Diffie - Hellman.
- Lý thuyết toán học:
    - Giả sử nhóm $G$ là $Z^*_p$ với:
        - $p$ là số nguyên tố lớn.
        - $g \in Z^*_p$ là phần tử sinh của một nhóm con cấp $q$: $$g_q \equiv 1 \mod p$$
    - Attacker có thể gửi một phần tử $h \in Z^*_p$ sao cho:
        - Bậc của $g'$ là một số nhỏ $r$ sao cho $g^r \equiv 1 \mod p$
        - Alice tính $(g')^a \mod p$.
    - Do đó:
        - $(g')^a \mod p \in \langle {h} \rangle$, nghĩa là chỉ nằm trong tập con nhỏ gồm $r$ phần tử.
        - Attacker chỉ cần thử $r$ giá trị $r$ giá trị để đoán $(g')^a \rightarrow$ Rút gọn không gian tìm kiếp của $a$.
- Lỗ hổng: Bậc của $g$ và nhóm con nhỏ
    - Nếu $g$ không có bậc nguyên tố lớn thì nhóm sinh bởi $g$ chứa các nhóm con có bậc nhỏ (ví dụ $g^k$ có thể có bậc $q << p$).
    - Khi đó, attacker có thể:
        - Gửi public key giả mạo $g'$ thuộc nhóm con nhỏ (bậc $q$): $$g' = g^{\frac{p - 1}{q}}$$
        - Nếu victim thực hiện $s = (g')^a \mod p$ thì $s \in$ nhóm con nhỏ (bậc $q$) $\rightarrow s \in \{g'^0, g'^1,..., g'^{q - 1}\}$
    $\rightarrow$ Attacker có thể enumerate tất cả các giá trị có thể của $s$ (Vì chỉ có $q$ khả năng) $\rightarrow$ Brute - force $a \mod q$.
- Ý tưởng chính:
    - Tạo một phần tử $g_i$ có bậc nhỏ $q_i$: $$g_i = g^{\frac{p - 1}{q_i}} \mod p \rightarrow ord(g_i) = q_i$$
    - Gửi $g_i$ làm khoá công khai giả mạo với victim:
        - Victim tính shared secret: $$s_i = g_i^a$$ $\rightarrow s_i \in$ nhóm con bậc $q_i$.
        - Vì $s_i$ chỉ có $q_i$ giá trị nên ta thử từng $s_i$ $\rightarrow$ Tìm được $a \mod q_i$.
    - Gửi nhiều giá trị thuộc nhóm con bậc nhỏ khác nhau $q_i$, sau đó dùng CRT: $$
\left\{
\begin{aligned}
a &\equiv a_1 \mod q_1 \\
a &\equiv a_2 \mod q_2 \\
&\vdots \\
a &\equiv a_k \mod q_k
\end{aligned}
\right.
\quad \Rightarrow \quad
a \mod q_1 q_2 \cdots q_k
$$ Nếu $q_1q_2...q_k > a$, attacker hoàn toàn khôi phục được $a$.
> Ví dụ 1:
> - Với $p = 211$, $g = 71$ (có bậc là $105$), $g^3 \mod p = 61$ (có bậc là $5$, nghĩa là nhóm con nhỏ bậc $5$).
> - Attacker gửi giá trị: $$g' = 61 \equiv g^3 \mod 211$$
> - Alice tính: $$s = (g')^a \mod 211 = 61^a \mod 211$$
> - Do $61$ có bậc $5$ nên: $$s \in \{61^0, 61^1, 61^2, 61^3, 61^4\} \mod 211$$ $\rightarrow s$ chỉ có năm giá trị.
> - Attacker có thể tính trước toàn bộ năm giá trị đó và thử từng giá trị để kiểm tra phản hồi (nếu có) và thu hẹp tập hơn khoá bí mật của Alice.

> Ví dụ 2:
> - Với $p = 467$, $n = p - 1 = 466 = 2.233$, $g = 2$, attacker tính: $$g' = 2^{\frac{n}{2}} = 2^{233} \mod 467$$ $\rightarrow$ Bậc của $g'$ bằng $2$.
> - Attacker gửi $g'$ đến Alice, Alice tính: $$s = (g')^a \rightarrow s \in \{1, 466\}$$ $\rightarrow$ Attacker biết $a \mod 2$
> - Lặp lại với: $$g'' = 2^{\frac{n}{233}} \rightarrow \text{Bậc 233}$$ $\rightarrow$ Attacker biết $a \mod 233$ 
> - Biết $a \mod 2$ và $a \mod 233 \rightarrow a \mod 466$ (CRT).
- Cách phòng tránh:
    - Đầu vào (khoá công khai từ bên kia) thực sự thuộc nhóm sinh đầy đủ bậc $q$: $$y^q \not \equiv 1 \mod p$$ $\rightarrow y$ không thuộc nhóm con nhỏ.
    - Chọn $p$ sao cho $p - 1$ có bậc lớn: $p = 2q + 1$, với $q$ là số nguyên tố.
    - Sử dụng các nhóm có cấu trúc khó kiểm soát (ví dụ Elliptic Curve Diffie - Hellman).
    
##### b) Ví dụ:
- Đề bài:
```py
from Crypto.Util.number import getPrime
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES 
from hashlib import sha256 
from random import randint
from secret import flag 

# Params 
p = getPrime(512)
a = randint(2**33, 2**35)
g = 2050493

# Exchange 
gA = pow(g, a, p)
r = randint(0, p - 1)
B = pow(r, (p - 1)//g, p)
gB = pow(B, a, p)

# Encrypt 
key = sha256(str(gB).encode()).digest()[:16]
cipher = AES.new(key,AES.MODE_ECB)
ciphertext = cipher.encrypt(pad(flag, 16))

# Print
print(f'{ciphertext = }')
print(f'{p = }')
print(f'{B = }')

"""
ciphertext = b'\xe6\xcb\xc6\x1d\xb19\xa4\xb8)V\x95u\xf8\x1b\x139 PI8\xa3\xcb\xb3\x85\x1ap\xbai.\xea\x1d\xab\x9a\x90\x82\x1a3\x91_;\xa5\x0e \x05\xf7\xf58%'
p = 12373179506263461354514623365077132800661338897023082048202681278072369025695953393083636918398048660173612137128780210954723129668855320275665251703736793
B = 12000841720785201162570293150131106089496068719117766176856306894850593806350814073092246930470170348368143014207968119599751108320780548445547875340920785
"""
```
- Phân tích cách làm:
    - Theo lý thuyết:
        - Nhóm nhân $Z^*_p$ có bậc $p - 1$.
        - Nếu $d \mid (p - 1)$ thì tồn tại nhóm con có bậc $d$: $$H_d = \{x \in Z^*_p \mid x^d = 1\}$$
    - Theo đề: $$B = r^{\frac{p - 1}{g}} \mod p \rightarrow B^g = r^{p - 1} \equiv 1 \mod p$$ $\rightarrow$ Cấp của $B$ chia hết cho $g$ nên mọi luỹ thừa của $B$ chỉ cho ra $g$ khả năng khác nhau.
    Ta có: $$g_B = B^a \equiv B^{a \mod g} \mod p$$ **$\rightarrow$ Small Subgroup Confinement Attack.**
    - Source code:
    ```py
    from Crypto.Util.Padding import unpad
    from Crypto.Cipher import AES
    from hashlib import sha256
    from tqdm import trange

    ciphertext = b'\xe6\xcb\xc6\x1d\xb19\xa4\xb8)V\x95u\xf8\x1b\x139 PI8\xa3\xcb\xb3\x85\x1ap\xbai.\xea\x1d\xab\x9a\x90\x82\x1a3\x91_;\xa5\x0e \x05\xf7\xf58%'
    p = 12373179506263461354514623365077132800661338897023082048202681278072369025695953393083636918398048660173612137128780210954723129668855320275665251703736793
    B = 12000841720785201162570293150131106089496068719117766176856306894850593806350814073092246930470170348368143014207968119599751108320780548445547875340920785
    g = 2050493  

    for i in trange(g):
        shared_secret = pow(B, i, p)
        key = sha256(str(shared_secret).encode()).digest()[:16]
        cipher = AES.new(key, AES.MODE_ECB)
        try:
            plaintext = unpad(cipher.decrypt(ciphertext), 16)
            if b'KCSC{' in plaintext:
                print("[+] Found flag:", plaintext.decode())
                break
        except:
            continue
    ```
- Kết quả: `KCSC{5m4ll_5ub_6r0up_4774ck_1n_p0hl16_h3llm4n}`

## CRYPTOHACK:
### I. STARTER:
#### 1. Working with Fields:
##### a) Mô tả đề bài:
Cho số nguyên tố $p = 991$ và phần tử $g = 209$. Tìm nghịch đảo $d = g^{-1}$ sao cho $g.d \mod 991 = 1$

##### b) Phân tích cách làm:
- Vì $p$ là số nguyên tố và $g$ khác $0$, theo định lý Fermat nhỏ (Fermat's Little Theorem): $$g^{-1} \equiv g^{p - 2} \mod p$$ Hay: $$d = g^{-1} = g^{p - 2} \mod p = 209^{989} \mod 991 = 569$$
- Source code:
```py
p = 991
g = 209

d = pow(g, p - 2, p)
print(d)
```

##### c) Kết quả:
`569`

#### 2. Generators of Groups:
##### a) Mô tả đề bài:
Làm việc trong trường hữu hạn $F_p$ với $p = 28151$, tìm phần tử nguyên nhỏ nhất $g$ sao cho sao cho $g$ sinh ra toàn bộ nhóm $F^*_p$ (nhóm các phần tử khác $0$ trong $F_p$), nói cách khác là tìm số nguyên $g$ nhỏ nhất sao cho $g^n \mod p$ sinh ra tất cả các số từ $1$ đến $p - 1$.

##### b) Phân tích cách làm:
- Trong trường $F_p$, tập các phần tử khác $0$ tạo thành nhóm nhân $F^*_p$ có số phần tử bằng $p - 1$, với $p = 28151$ thì $φ(p) = 28150$.
- Phân tích $p - 1 = 28150$ thành các số nguyên tố $q$, khi đó $28150 = 2.5^2.563$ hay $q \in$ {$2$, $5$, $563$}.
- Kiểm tra từng $g \in [2, 21850]$ sao cho $g^{\frac{p - 1}{q}} \not\equiv 1 \mod p$ cho đến khi tìm được $g$ đầu tiên. Nếu không có kết quả nào bằng $1$ thì $g$ nguyên thuỷ.
- Source code:
```py
p = 28151

def prime_factors(n):
    factors = set()
    i = 2
    while i*i <= n:
        while n%i == 0:
            factors.add(i)
            n //= i
        i += 1
    if n > 1:
        factors.add(n)
    return list(factors)

factors = prime_factors(p - 1)

for g in range (2, p):
    is_primitive = True
    for q in factors:
        if pow(g, (p - 1)//q, p) == 1:
            is_primitive = False
            break
    if is_primitive:
        print(g)
        break
```

##### c) Kết quả:
`7`

#### 3. Computing Public Values:
##### a) Mô tả đề bài:
- Đây là bài toán liên quan đến **Diffie - Hellman Key Exchange** trong lý thuyết mật mã, cụ thể yêu cầu tính giá trị công khai từ khoá bí mật.
- Cho:
    - Số nguyên tố lớn $p$ (dài $1024$ bit, dạng chuẩn NIST).
    - Phần tử sinh $g = 2$
    - Số bí mật $a$ (rất lớn).
::: spoiler Thông số
```py
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
a = 972107443837033796245864316200458246846904598488981605856765890478853088246897345487328491037710219222038930943365848626194109830309179393018216763327572120124760140018038673999837643377590434413866611132403979547150659053897355593394492586978400044375465657296027592948349589216415363722668361328689588996541370097559090335137676411595949335857341797148926151694299575970292809805314431447043469447485957669949989090202320234337890323293401862304986599884732815
```
:::
- Tính: $$g^a \mod p$$

##### b) Phân tích đề bài:
Source code:
```py
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
a = 972107443837033796245864316200458246846904598488981605856765890478853088246897345487328491037710219222038930943365848626194109830309179393018216763327572120124760140018038673999837643377590434413866611132403979547150659053897355593394492586978400044375465657296027592948349589216415363722668361328689588996541370097559090335137676411595949335857341797148926151694299575970292809805314431447043469447485957669949989090202320234337890323293401862304986599884732815

print(pow(g, a, p))
```

##### c) Kết quả:
```
1806857697840726523322586721820911358489420128129248078673933653533930681676181753849411715714173604352323556558783759252661061186320274214883104886050164368129191719707402291577330485499513522368289395359523901406138025022522412429238971591272160519144672389532393673832265070057319485399793101182682177465364396277424717543434017666343807276970864475830391776403957550678362368319776566025118492062196941451265638054400177248572271342548616103967411990437357924
```

#### 4. Computing Shared Secrets:
##### a) Mô tả đề bài:
- Bạn và Alice đang thực hiện trao đổi khoá Diffie - Hellman để tạo ra một khoá chung bí mật mà người ngoài không thể tính được.
- Các thông số được cung cấp:
::: spoiler Thông số công khai (NIST parameters)
```py
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
```
:::
::: spoiler Thông số nhận được từ Alice
$$A = g^a \mod p$$
```py
A = 70249943217595468278554541264975482909289174351516133994495821400710625291840101960595720462672604202133493023241393916394629829526272643847352371534839862030410331485087487331809285533195024369287293217083414424096866925845838641840923193480821332056735592483730921055532222505605661664236182285229504265881752580410194731633895345823963910901731715743835775619780738974844840425579683385344491015955892106904647602049559477279345982530488299847663103078045601
```
:::
::: spoiler Thông tin bạn tự tạo
$$B = g^b \mod p$$
```py
b = 12019233252903990344598522535774963020395770409445296724034378433497976840167805970589960962221948290951873387728102115996831454482299243226839490999713763440412177965861508773420532266484619126710566414914227560103715336696193210379850575047730388378348266180934946139100479831339835896583443691529372703954589071507717917136906770122077739814262298488662138085608736103418601750861698417340264213867753834679359191427098195887112064503104510489610448294420720
B = 518386956790041579928056815914221837599234551655144585133414727838977145777213383018096662516814302583841858901021822273505120728451788412967971809038854090670743265187138208169355155411883063541881209288967735684152473260687799664130956969450297407027926009182761627800181901721840557870828019840218548188487260441829333603432714023447029942863076979487889569452186257333512355724725941390498966546682790608125613166744820307691068563387354936732643569654017172
```
:::
- Sử dụng $b$ và $A$ để tính: $$shared_-secret = A^b \mod p$$

##### b) Phân tích đề bài:
Source code:
```py
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
A = 70249943217595468278554541264975482909289174351516133994495821400710625291840101960595720462672604202133493023241393916394629829526272643847352371534839862030410331485087487331809285533195024369287293217083414424096866925845838641840923193480821332056735592483730921055532222505605661664236182285229504265881752580410194731633895345823963910901731715743835775619780738974844840425579683385344491015955892106904647602049559477279345982530488299847663103078045601
b = 12019233252903990344598522535774963020395770409445296724034378433497976840167805970589960962221948290951873387728102115996831454482299243226839490999713763440412177965861508773420532266484619126710566414914227560103715336696193210379850575047730388378348266180934946139100479831339835896583443691529372703954589071507717917136906770122077739814262298488662138085608736103418601750861698417340264213867753834679359191427098195887112064503104510489610448294420720
B = 518386956790041579928056815914221837599234551655144585133414727838977145777213383018096662516814302583841858901021822273505120728451788412967971809038854090670743265187138208169355155411883063541881209288967735684152473260687799664130956969450297407027926009182761627800181901721840557870828019840218548188487260441829333603432714023447029942863076979487889569452186257333512355724725941390498966546682790608125613166744820307691068563387354936732643569654017172

res = pow(A, b, p)
print(res)
```

##### c) Kết quả:
```
1174130740413820656533832746034841985877302086316388380165984436672307692443711310285014138545204369495478725102882673427892104539120952393788961051992901649694063179853598311473820341215879965343136351436410522850717408445802043003164658348006577408558693502220285700893404674592567626297571222027902631157072143330043118418467094237965591198440803970726604537807146703763571606861448354607502654664700390453794493176794678917352634029713320615865940720837909466
```

#### 5. Deriving Symmetric Keys:
##### a) Mô tả đề bài:
Mô tả quá trình trao đổi khoá Diffie - Hellman để tạo ra một khoá bí mật dùng cho AES encryption:
- Thông tin được cung cấp sử dụng chuẩn NIST:
::: spoiler Thông số
```py
g = 2
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
```
:::
- Trao đổi khoá Diffie - Hellman:
::: spoiler Alice chọn số bí mật a và gửi giá trị công khai
$$A = g^a \mod p$$
```py
A = 112218739139542908880564359534373424013016249772931962692237907571990334483528877513809272625610512061159061737608547288558662879685086684299624481742865016924065000555267977830144740364467977206555914781236397216033805882207640219686011643468275165718132888489024688846101943642459655423609111976363316080620471928236879737944217503462265615774774318986375878440978819238346077908864116156831874695817477772477121232820827728424890845769152726027520772901423784
```
:::
::: spoiler Bạn chọn số bí mật b và gửi lại
$$B = g^b \mod p$$
```py
b = 197395083814907028991785772714920885908249341925650951555219049411298436217190605190824934787336279228785809783531814507661385111220639329358048196339626065676869119737979175531770768861808581110311903548567424039264485661330995221907803300824165469977099494284722831845653985392791480264712091293580274947132480402319812110462641143884577706335859190668240694680261160210609506891842793868297672619625924001403035676872189455767944077542198064499486164431451944
B = 1241972460522075344783337556660700537760331108332735677863862813666578639518899293226399921252049655031563612905395145236854443334774555982204857895716383215705498970395379526698761468932147200650513626028263449605755661189525521343142979265044068409405667549241125597387173006460145379759986272191990675988873894208956851773331039747840312455221354589910726982819203421992729738296452820365553759182547255998984882158393688119629609067647494762616719047466973581
```
:::
- Cả hai tính khoá chung:
Alice tính: $$s = B^a \mod p$$ Bạn tính: $$s = A^b \mod p$$ Vì $(g^a)^b = (g^a)^b \mod p$
- Mã hoá AES:
    - Từ khoá bí mật $s$, cả hai tạo một AES Key (thường bằng cách băm $s$ với SHA - 256).
    - Alice mã hoá flag bằng AES và gửi `iv` và `encrypted_flag` (flag đã mã hoá bằng AES - CBC)
::: spoiler Thông số
```py
{'iv': '737561146ff8194f45290f5766ed6aba', 'encrypted_flag': '39c99bf2f0c14678d6a5416faef954b5893c316fc3c48622ba1fd6a9fe85f3dc72a29c394cf4bc8aff6a7b21cae8e12c'}
```
:::
- Nhiệm vụ:
    - Tính **shared_secret** $s = A^b \mod p$
    - Băm $s$ để tạo AES Key
    - Dùng AES (chế độ CBC, với `iv` đã cho) để giải mã `encrypted_flag`
- Hai source được cho:
::: spoiler source.py
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import hashlib
import os
from secret import shared_secret

FLAG = b'crypto{????????????????????????????}'


def encrypt_flag(shared_secret: int):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Encrypt flag
    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    ciphertext = cipher.encrypt(pad(FLAG, 16))
    # Prepare data to send
    data = {}
    data['iv'] = iv.hex()
    data['encrypted_flag'] = ciphertext.hex()
    return data


print(encrypt_flag(shared_secret))
```
:::
::: spoiler decrypt.py
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    if is_pkcs7_padded(plaintext):
        return unpad(plaintext, 16).decode('ascii')
    else:
        return plaintext.decode('ascii')


shared_secret = ?
iv = ?
ciphertext = ?

print(decrypt_flag(shared_secret, iv, ciphertext))
```
:::
##### b) Phân tích cách làm:
Từ source code `decrypt.py` đã cho, thêm một vài thông số như sau vào:
```py
A = 112218739139542908880564359534373424013016249772931962692237907571990334483528877513809272625610512061159061737608547288558662879685086684299624481742865016924065000555267977830144740364467977206555914781236397216033805882207640219686011643468275165718132888489024688846101943642459655423609111976363316080620471928236879737944217503462265615774774318986375878440978819238346077908864116156831874695817477772477121232820827728424890845769152726027520772901423784
b = 197395083814907028991785772714920885908249341925650951555219049411298436217190605190824934787336279228785809783531814507661385111220639329358048196339626065676869119737979175531770768861808581110311903548567424039264485661330995221907803300824165469977099494284722831845653985392791480264712091293580274947132480402319812110462641143884577706335859190668240694680261160210609506891842793868297672619625924001403035676872189455767944077542198064499486164431451944
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
backpage = {'iv': '737561146ff8194f45290f5766ed6aba', 'encrypted_flag': '39c99bf2f0c14678d6a5416faef954b5893c316fc3c48622ba1fd6a9fe85f3dc72a29c394cf4bc8aff6a7b21cae8e12c'}

s = pow(A, b, p)
```
Ta có:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    if is_pkcs7_padded(plaintext):
        return unpad(plaintext, 16).decode('ascii')
    else:
        return plaintext.decode('ascii')


A = 112218739139542908880564359534373424013016249772931962692237907571990334483528877513809272625610512061159061737608547288558662879685086684299624481742865016924065000555267977830144740364467977206555914781236397216033805882207640219686011643468275165718132888489024688846101943642459655423609111976363316080620471928236879737944217503462265615774774318986375878440978819238346077908864116156831874695817477772477121232820827728424890845769152726027520772901423784
b = 197395083814907028991785772714920885908249341925650951555219049411298436217190605190824934787336279228785809783531814507661385111220639329358048196339626065676869119737979175531770768861808581110311903548567424039264485661330995221907803300824165469977099494284722831845653985392791480264712091293580274947132480402319812110462641143884577706335859190668240694680261160210609506891842793868297672619625924001403035676872189455767944077542198064499486164431451944
p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
backpage = {'iv': '737561146ff8194f45290f5766ed6aba', 'encrypted_flag': '39c99bf2f0c14678d6a5416faef954b5893c316fc3c48622ba1fd6a9fe85f3dc72a29c394cf4bc8aff6a7b21cae8e12c'}
shared_secret = pow(A, b, p)
iv = backpage['iv']
ciphertext = backpage['encrypted_flag']

print(decrypt_flag(shared_secret, iv, ciphertext))
```

##### c) Kết quả:
`crypto{sh4r1ng_s3cret5_w1th_fr13nd5}`

### II. MAN IN THE MIDDLE:
#### 1. Parameter Injection:
##### a) Mô tả đề bài:
- *"Bạn đang ở một vị trí không chỉ chặn trao đổi khóa DH của Alice và Bob, mà còn viết lại các tin nhắn của họ. Hãy suy nghĩ về cách bạn có thể chơi với phương trình DH mà họ tính toán, và do đó vượt qua sự cần thiết phải bẻ khóa mọi vấn đề logarit riêng biệt.
Sử dụng tập lệnh từ "Deriving Symmetric Keys" để giải mã flag sau khi bạn đã phục hồi secret được chia sẻ."*
- Kết nối tại `socket.cryptohack.org 13371`

##### b) Phân tích cách làm:
- Lý thuyết gốc về giao thức trao đổi khoá Diffie - Hellman:
    - Alice và Bob thoả thuận một số nguyên tố $p$ và phần tử sinh $g$ (public).
    - Mỗi bên chọn một số bí mật riêng:
        - Alice chọn $a$ và tính $A = g^a \mod p$.
        - Bob chọn $b$ và tính $B = g^b \mod p$.
    - Hai bên gửi công khai $A$ và $B$ cho nhau.
    - Mỗi bên tính shared secret:
        - Alice: $s = B^a \mod p = g^{ab} \mod p$
        - Bob: $s = A^b \mod p = g^{ab} \mod p$
- Ý tưởng tấn công Man in the middle trong challenge: Đứng giữa Alice và Bob để giả mạo giá trị $A$ và $B$ (sửa đổi $A = B = P$).
    - Nếu Alice tính shared secret: $s = B^a \mod p = p^a \mod p = 0$
    - Nếu Bob tính shared secret: $s = A^b \mod p = p^b \mod p = 0$
    $\rightarrow$ Cả hai bên đều dùng khoá bí mật là 0.
- Source code:
```py
import pwn
import json
import hashlib
from Crypto.Cipher import AES


HOST = "socket.cryptohack.org"
PORT = 13371

def sha1_key_from_secret(secret: int) -> bytes:
    sha1 = hashlib.sha1()
    sha1.update(str(secret).encode())
    return sha1.digest()[:16]

def mitm_attack():
    conn = pwn.connect(HOST, PORT)

    try:
        # Bước 1: Nhận thông tin từ Alice (p, g, A)
        conn.readuntil(b": ")
        from_alice = json.loads(conn.readline().decode().strip())

        p = int(from_alice['p'], 16)
        g = int(from_alice['g'], 16)
        A = int(from_alice['A'], 16)

        print("[+] Intercepted from Alice")
        print(f"p = {p}\ng = {g}\nA = {A}")

        # Bước 2: Gửi p, g, A = p đến Bob (A giả)
        fake_to_bob = {
            "p": hex(p),
            "g": hex(g),
            "A": hex(p)
        }
        conn.sendlineafter(b": ", json.dumps(fake_to_bob).encode())
        print("[+] Sent forged parameters to Bob (A = p)")

        # Bước 3: Nhận B từ Bob (trên lý thuyết là B = p)
        conn.readuntil(b": ")
        from_bob = json.loads(conn.readline().decode().strip())

        B = int(from_bob['B'], 16)
        print(f"[+] Intercepted from Bob: B = {B}")

        # Bước 4: Gửi B = p lại cho Alice
        fake_to_alice = {
            "B": hex(p)
        }
        conn.sendlineafter(b": ", json.dumps(fake_to_alice).encode())
        print("[+] Sent forged B = p back to Alice")

        # Bước 5: Nhận ciphertext từ Alice (đã mã hóa bằng key từ SHA1(0))
        conn.readuntil(b": ")
        encrypted = json.loads(conn.readline().decode().strip())

        iv = bytes.fromhex(encrypted['iv'])
        ciphertext = bytes.fromhex(encrypted['encrypted_flag'])

        # Tính key từ secret = 0
        key = sha1_key_from_secret(0)
        cipher = AES.new(key, AES.MODE_CBC, iv)

        plaintext = cipher.decrypt(ciphertext)
        print("[+] Flag:")
        print(plaintext)

    finally:
        conn.close()

if __name__ == "__main__":
    mitm_attack()
```

##### c) Kết quả:
`crypto{n1c3_0n3_m4ll0ry!!!!!!!!}`

#### 2. Export - grade:
##### a) Mô tả đề bài:
- *"Alice và Bob đang sử dụng các cơ sở mã kế thừa và cần phải thương lượng các tham số mà cả hai hỗ trợ. Bạn đã man-in-the-middled bước đàm phán này và có thể quan sát một cách thụ động sau đó. Làm thế nào bạn sẽ phá hỏng ngày của họ lần này?"*
- Connect at `socket.cryptohack.org 13379`

##### b) Phân tích cách làm:
- Alice đưa ra các DH mà cô ấy có thể hỗ trợ:
```
{"supported": ["DH1536", "DH1024", "DH512", "DH256", "DH128", "DH64"]}
```
- Sửa lại thông điệp, ta chỉ gửi cho Bob `{"supported": ["DH64"]}` (vì với DH54 $p$ chỉ có $64$ bit, dễ dàng brute - force $a$).
- Sau khi Bob phản hồi (ta gửi `{"chosen": "DH64"}
`), server sẽ gửi ta các thông số $p$, $g$, $A$, $B$, $iv$, $encrypted_-flag$.
- Hướng khai thác: Sử dụng **Pohlig - Hellman** để tấn công.
- Source code:
```py
#!/usr/bin/env sage
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib
from sage.all import *


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(len(padding)))


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]

    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    if is_pkcs7_padded(plaintext):
        return unpad(plaintext, 16).decode('ascii')
    else:
        return plaintext.decode('ascii')


p = int("0xde26ab651b92a129", 16)
g = int("0x2", 16)
A = int("0x8ae65604208f72ab", 16)
B = int("0xc01dbb043fe91f03", 16)
iv = "4884a6e2a91e139a143720c021bb230a"
encrypted_flag = "3b550bc68446d6c2ef1869b191e2ac59d1bce3f18638360d3e00ebd5c33cdac3"
n = p - 1
factors_n = [2**3, 3, 293, 5417, 420233272499]

Fp = Zmod(p)
Gr = []
Ar = []
residues = []
mods = []

for r in factors_n:
    e = n // r
    gr = Fp(g)**e
    ar = Fp(A)**e
    x_i = discrete_log(ar, gr)
    residues.append(x_i)
    mods.append(r)

a = crt(residues, mods)
s = pow(B, a, p)
print(decrypt_flag(s, iv, encrypted_flag))
```

##### c) Kết quả:
`crypto{d0wn6r4d35_4r3_d4n63r0u5}`

#### 3. Static Client:
##### a) Mô tả đề bài:
- "*Bạn vừa hoàn thành việc nghe lén cuộc trò chuyện giữa Alice và Bob. Bây giờ bạn có cơ hội nói chuyện với Bob. Bạn sẽ nói gì?*"
- Kết nối tại `socket.cryptohack.org 13373`

##### b) Phân tích cách làm:
- Alice và Bob trao đổi với nhau qua hai quá trình:
    - Quá trình $1$: $p$, $g_1$, $A$, $B$, $IV_1$, $C_1$ (đã hoàn thành).
    - Quá trình $2$: Giả mạo Alice và gửi cho Bob các thông số giả mạo.
- Từ tên bài có thể thấy khoá bí mật $b$ của Bob luôn cố định.
- Hướng khai thác: **Man in the middle**:
    - Gửi một nhóm thông số giả ($p$, $g_2 = A$, $A$) cho Bob, Bob gửi lại $B$, $IV_2$, $C_2$: $$B = (g_2)^b \mod p$$ Mà $g_2 = A$ nên: $$B = A^b \mod p = shared_-secret = s$$ Vì ta dùng lại $A$ và $g$ ban đầu nên $s$ vừa tìm được cũng là khoá chung của cả hai quá trình.
    - Dùng $shared_-secret$ vừa tìm được để giải $C_2$, ta được thông điệp của Bob: `Hey, what's up. I got bored generating random numbers did you see?` $\rightarrow$ $B$ là cố định. Giải $C_1$, ta được thông điệp của Alice chính là Flag cần tìm.
- Source code:
```py
from pwn import *
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(len(padding)))

def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]

    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    try:
        if is_pkcs7_padded(plaintext):
            return unpad(plaintext, 16).decode('utf-8', errors='ignore')
        else:
            return plaintext.decode('utf-8', errors='ignore')
    except:
        return ""


HOST, PORT = "socket.cryptohack.org", 13373
io = remote(HOST, PORT)


alice_data, bob_data, cipher_data = {}, {}, {}

Alice = eval(io.recvline()[len("Intercepted from Alice: "):])
Bob = eval(io.recvline()[len("Intercepted from Bob: "):])
alice = eval(io.recvline()[len("Intercepted from Alice: "):])

p = Alice["p"]
g = Alice["A"]
iv = alice["iv"]
encrypted = alice["encrypted"]
B = Bob["B"]


request = {"p" : p, "g" : g, "A" : "0x01"}
request = json.dumps(request)
io.sendlineafter(b"Bob connects to you, send him some parameters: ", request.encode())

print("iv = ", iv)
print("encrypted =", encrypted)
print("B = ", B)

s = eval(io.recvline()[len("Bob says to you: "):])["B"]
s = int(s[2:], 16)
print(s)

flag = decrypt_flag(s, iv, encrypted)
print("FLAG: ", flag)
```

##### c) Kết quả:
`crypto{n07_3ph3m3r4l_3n0u6h}`

### III. GROUP THEORY:
#### 1. Additive:
##### a) Mô tả đề bài:
- "*Alice and Bob decided to do their DHKE in an additive group rather than a multiplicative group. What could go wrong?
Use the script from "Deriving Symmetric Keys" to decrypt the flag once you've recovered the shared secret.*"
- Kết nối tại `socket.cryptohack.org 13380`

##### b) Phân tích cách làm:
- Đây là challenge thay đổi cách dùng nhóm trong **Diffie – Hellman Key Exchange** (DHKE) — từ nhóm nhân modulo p (multiplicative group) sang nhóm cộng modulo $p$ (additive group).
- Nguyên lý hoạt động của DHKE bình thường: Trong một nhóm nhân modulo nguyên tố $Z^*_p$:
    - Số nguyên tố lớn $p$
    - Phần tử sinh $g \in Z^*_p$
    - Alice chọn $a$ bí mật và gửi $A = g^a \mod p$
    - Bob chọn $b$ bí mật và gửi $B = g^b \mod p$
    - Cả hai tính $shared_-secret = s = g^{ab}$ qua:
        - Alice: $B^a = (g^b)^a = g^{ab}$
        - Bob: $A^b = (g^a)^b = g^{ab}$
- Nguyên lý hoạt động của DHKE nếu thay bằng nhóm cộng modulo $Z_p$ (additive group):
    - Phần tử sinh $g \in Z_p$
    - Alice chọn $a$ bí mật và gửi $A = g.a \mod p$
    - Bob chọn $b$ bí mật và gửi $B = g.b \mod p$
    - Khoá chung: $$shared_-secret = s = g.a.b \mod p$$
$\rightarrow$ Dễ dàng khôi phục $s$ với: $\left\{
\begin{aligned}
a = A/g \mod p \\
b = B/g \mod p
\end{aligned}
\right.$
- Source code:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(len(padding)))


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]

    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    try:
        if is_pkcs7_padded(plaintext):
            return unpad(plaintext, 16).decode('utf-8', errors='ignore')
        else:
            return plaintext.decode('utf-8', errors='ignore')
    except:
        return ""


data_Alice = {"p": "0xffffffffffffffffc90fdaa22168c234c4c6628b80dc1cd129024e088a67cc74020bbea63b139b22514a08798e3404ddef9519b3cd3a431b302b0a6df25f14374fe1356d6d51c245e485b576625e7ec6f44c42e9a637ed6b0bff5cb6f406b7edee386bfb5a899fa5ae9f24117c4b1fe649286651ece45b3dc2007cb8a163bf0598da48361c55d39a69163fa8fd24cf5f83655d23dca3ad961c62f356208552bb9ed529077096966d670c354e4abc9804f1746c08ca237327ffffffffffffffff", "g": "0x02", "A": "0x48bbba113bb85dd228557b6f080a939d06e8babfce5441f64bb890ebfde6f360759982de92216636f087e96ef9da4e16f40c6892f68778113949ea95a2b375147a997fe72b2cd7fdb376f2a90af4512fef5550e4d5033273ca2e543269dacac5bc5cc9aeb1a28d2f5424537005d1008db652f5691054efd82715ee31820dec366450dc1d9df00b7b2657206e26ae3c69c34f151c99315a9603722174a0dd4813b26a19b4844f4bfca28d523c43077a5822c1001a2194f9fe8cdf300b3fcdeca0"}
data_Bob = {"B": "0x773948cfd3bb268ffd18d2e1e62b02edda94b45137b7b6e808a84dafcdda04eaffaa8bd800b727491cc98665102d2ed6e18b47ee91b1b5e88dc9a0f56283a7a768ba1f55b58216e974a91dae426efe6bcda575b13249db42da133b52af8d65f79f7cc944e56ff9884e51f9afd0102b41a3b63c5d55ba0cdee3c135c88cee039bf685219ed169ab13283bd2de2f340bb66cb91a6e7a72b9fe2b3dd524a8d6107baf0f98725bf167d5bc4d8be6212d1918557232ca5babbc2c1d47053fba78bc39"}
iv = "22d43ca93b750dd58309ffca8f9221a4"
encrypted = "b973f441446e7c8b73cf79d4aafccaba67740391698273804adba267fb1c7b04460a086fd5b0435198373c3199e58969"
p = int(data_Alice["p"], 16)
g = int(data_Alice["g"], 16)
A = int(data_Alice["A"], 16)
B = int(data_Bob["B"], 16)

g_inv = pow(g, -1, p)
a = (A*g_inv)%p
b = (B*g_inv)%p
s = (g*a*b)%p

print(decrypt_flag(s, iv, encrypted))
```

##### c) Kết quả:
`crypto{cycl1c_6r0up_und3r_4dd1710n?}`

#### 2. Static Client 2:
##### a) Mô tả đề bài:
- "*Bob got a bit more careful with the way he verifies parameters. He's still insisting on using the $p$ and $g$ values provided by his partner. Wonder if he missed anything?*"
- Kết nối tại `socket.cryptohack.org 13378`

##### b) Phân tích cách làm:
- Đây là challenge tiếp nối nâng cao hơn của Static Client trong phần Man in the middle. Ở bài này, Bob đã khôn hơn khi đặt điều kiện để kiểm tra $p$, $g$ nên ta không thể thử với cách làm như trước đó, nhưng Bob vẫn dùng tới $p$ và $g$ của Alice.
- Hướng tấn công: Gửi $p$ giả mạo là smooth number, giữ nguyên $g$ và $A$.
    - Trong các challenge trước, $p$ có độ dài khoảng $1538$ bits nên nếu ta gửi $p$ giả mạo nhỏ hơn khoảng đó thì Bob sẽ không nhận.
    - Source code:
    ```py
    from pwn import *
    import json
    from Crypto.Cipher import AES
    from Crypto.Util.Padding import unpad
    import hashlib
    from Crypto.Util.number import isPrime, getPrime


    def make_smooth_prime():
        smooth_number = 0
        while not isPrime(smooth_number + 1):
            smooth_number = 2
            while len(bin(smooth_number)) < 1540:
                prime = getPrime(16)
                smooth_number *= prime
        return hex(smooth_number + 1)


    def is_pkcs7_padded(message):
        padding = message[-message[-1]:]
        return all(padding[i] == len(padding) for i in range(len(padding)))


    def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
        sha1 = hashlib.sha1()
        sha1.update(str(shared_secret).encode('ascii'))
        key = sha1.digest()[:16]

        ciphertext = bytes.fromhex(ciphertext)
        iv = bytes.fromhex(iv)
        cipher = AES.new(key, AES.MODE_CBC, iv)
        plaintext = cipher.decrypt(ciphertext)

        try:
            if is_pkcs7_padded(plaintext):
                return unpad(plaintext, 16).decode('utf-8', errors='ignore')
            else:
                return plaintext.decode('utf-8', errors='ignore')
        except:
            return ""


    HOST, PORT = "socket.cryptohack.org", 13378
    io = remote(HOST, PORT)


    line = io.recvline().decode()
    print(line)
    Alice = json.loads(line.split(":", 1)[1].strip())

    line = io.recvline().decode()
    print(line)
    Bob = json.loads(line.split(":", 1)[1].strip())

    line = io.recvline().decode()
    print(line)
    alice = json.loads(line.split(":", 1)[1].strip())

    p = Alice["p"]
    A = Alice["A"]
    g = Alice["g"]
    iv = alice["iv"]
    encrypted = alice["encrypted"]

    g_int = int(g, 16)
    A_int = int(A, 16)
    p_fk = make_smooth_prime()


    request = {"p": p_fk, "g": g, "A": A}
    request = json.dumps(request)
    io.sendlineafter(b"Bob connects to you, send him some parameters: ", request.encode())

    line = io.recvline().decode()
    Bob = json.loads(line.split(":", 1)[1].strip())
    B = Bob["B"]

    print("p =", p_fk)
    print("g =", g)
    print("B =", B)
    print("A =", A_int)
    print("iv =", iv)
    print("encrypted =", encrypted)
    ```
    ::: spoiler Kết quả:
    ```terminal
    [x] Opening connection to socket.cryptohack.org on port 13378
    [x] Opening connection to socket.cryptohack.org on port 13378: Trying 134.122.111.232
    [+] Opening connection to socket.cryptohack.org on port 13378: Done
    Intercepted from Alice: {"p": "0xffffffffffffffffc90fdaa22168c234c4c6628b80dc1cd129024e088a67cc74020bbea63b139b22514a08798e3404ddef9519b3cd3a431b302b0a6df25f14374fe1356d6d51c245e485b576625e7ec6f44c42e9a637ed6b0bff5cb6f406b7edee386bfb5a899fa5ae9f24117c4b1fe649286651ece45b3dc2007cb8a163bf0598da48361c55d39a69163fa8fd24cf5f83655d23dca3ad961c62f356208552bb9ed529077096966d670c354e4abc9804f1746c08ca237327ffffffffffffffff", "g": "0x02", "A": "0xaccb04d86dc88a32aa254819900b55a978893a2ce04189cb619909a84bf982d01eaafca12bda8ff42b5046361ee8058772309ddd8ce816bccec92e627aed7fd14fd609aea4253c7e42d1629dc28a54746527a676716952d04de4fb1a4b07a8ea182973400568b9cf93004c5b45ce4ce4f21527be5dfbfff42cb9ef7eb6726fe6b71878f8b48bc891a830c7e451aa217bffbd4a86cacfc57a65986a4cb827be7cc7ac3b9fb98a37d56b9e8cda7156484b3add054395c8b0fa3cbb3afbb8c987f"}

    Intercepted from Bob: {"B": "0xd0d69585c6586c3b1a23e04245826be6db4aed1c9bc70f7110a30165ca878d31434aa357c2bd26d3c398284a17319504e1aeead141234afeb57dfef11417fdec44b21cea83920f300f4e0c3fb573a895371b24652c5e6ea0539b7719f0f966ac7adb9a292cc49f4d8b39560e02fa82aab3c273cc7df512a80e2de6f0e8840c00554f09460eaa2e221173a9ca13182d4e1342b1e54965e16ca5fc23b1aae80aedc7fb80e1aa9be8b0274812676e8e570e1abf65eea0c49f18794a5afba975c7c7"}

    Intercepted from Alice: {"iv": "ebd3c617e8c4dd804dca34b7a48b2076", "encrypted": "58f4b780078d29d7c7d582d4814879dcd3c2debdbb2b7a75903ce681aafa62364b399cadb5f9befd7f2f33daeddc6265"}

    p = 0x88b9f4287c7729d41d495961e38c2fa946f5ad1dcd4aa089c8aa795a7483ce4a70664652b0d6c803931287e3e3f0f751c19b76945bf4ea872fda83a4cd8f372dcc3e8671f0ce0c30df3bb3bbf5ebc5ac707ee5772c443e4574362a6d2e7812cdff49dd5ba9f981fe7851f486f3f5ad2e6609c43cfc59433a947cc5397c275e3c3568cf0312adb87195cfc1bb9c601971ea23fb20ac667c861b8af221efc53fb403e30bd8ebe40dd08f5d5756040792e45127872dc2502c3aaec0d95bf199b1bb7
    g = 0x02
    B = 0x5a483e25258d9bc13b3febeeb7fae5f69db4189a5b95a61e7b36613214d5ccb1384b2e9f8006ef28ee8f1b4785a7e91279ccf277ed9d581de2d991dacffd503e79d441f0f8e70439581489b2ebd45cb29a9831095691312f32ae4338871f21551efdd618690380e814bd01fc36dc7634b4dbaf088460f22df506b945363c486121db564fdb2b301bd4c47da89a43ec412964a5d978d74562f6f1418dba1137b042124de26d5db06db1ec0e8c201a457dd64c46d334c67bdcfbd251ddf4c2f9040
    A = 101680961442852616108775756711978398658020465638043052372300486767867582515418527189197457235259152337420502074327462265360037428144939560118901628407913104915663522410484280449906715020616011776350011947778438489938096886855220734792880546558711804647587408255689433923676242311691048687788001407006798792519007963025111859825608352916601939365718156325598725920047094137693621631063961449397399891415978307082976933058246120889332769221578175981835303883937919
    iv = ebd3c617e8c4dd804dca34b7a48b2076
    encrypted = 58f4b780078d29d7c7d582d4814879dcd3c2debdbb2b7a75903ce681aafa62364b399cadb5f9befd7f2f33daeddc6265
    ```
    :::
- Lấy các thông số vừa có được để tìm $b$ trong SageMath:
    - Source code:
    ```py
    #!/usr/bin/env sage
    from sage.all import *

    p = 0x88b9f4287c7729d41d495961e38c2fa946f5ad1dcd4aa089c8aa795a7483ce4a70664652b0d6c803931287e3e3f0f751c19b76945bf4ea872fda83a4cd8f372dcc3e8671f0ce0c30df3bb3bbf5ebc5ac707ee5772c443e4574362a6d2e7812cdff49dd5ba9f981fe7851f486f3f5ad2e6609c43cfc59433a947cc5397c275e3c3568cf0312adb87195cfc1bb9c601971ea23fb20ac667c861b8af221efc53fb403e30bd8ebe40dd08f5d5756040792e45127872dc2502c3aaec0d95bf199b1bb7
    B = Mod(0x5a483e25258d9bc13b3febeeb7fae5f69db4189a5b95a61e7b36613214d5ccb1384b2e9f8006ef28ee8f1b4785a7e91279ccf277ed9d581de2d991dacffd503e79d441f0f8e70439581489b2ebd45cb29a9831095691312f32ae4338871f21551efdd618690380e814bd01fc36dc7634b4dbaf088460f22df506b945363c486121db564fdb2b301bd4c47da89a43ec412964a5d978d74562f6f1418dba1137b042124de26d5db06db1ec0e8c201a457dd64c46d334c67bdcfbd251ddf4c2f9040, p)
    g = Mod(0x02, p)
    
    b = discrete_log(B, g)
    print("b =", b)
    ```
    ::: spoiler Kết quả:
    ```py
    b = 1919572943691512325783103720167834163677411292709378502535498859989993544026380143919501049584589675317643993465536543895780854808442293000014297210200227069779643763121704810281976733978781152126062646602812482025293137787739116693980988513420732289020477701182639042794562638875881378349771734410919106042203493166198706573467903966100368713572415175654342828296086659529676015616513470105470901979846373335352656586302787870238998914215908919919219987614105175
    ```
    :::
    - Giá trị $b$ của Bob là cố định.
- Sau khi có được $b$, kết hợp với các giá trị $p$, $g$, $A$, $iv$ và $encrypted$ ban đầu của Alice để giải ra flag:
```py
from pwn import *
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(len(padding)))

def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]

    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    try:
        if is_pkcs7_padded(plaintext):
            return unpad(plaintext, 16).decode('utf-8', errors='ignore')
        else:
            return plaintext.decode('utf-8', errors='ignore')
    except:
        return ""


p = int("0xffffffffffffffffc90fdaa22168c234c4c6628b80dc1cd129024e088a67cc74020bbea63b139b22514a08798e3404ddef9519b3cd3a431b302b0a6df25f14374fe1356d6d51c245e485b576625e7ec6f44c42e9a637ed6b0bff5cb6f406b7edee386bfb5a899fa5ae9f24117c4b1fe649286651ece45b3dc2007cb8a163bf0598da48361c55d39a69163fa8fd24cf5f83655d23dca3ad961c62f356208552bb9ed529077096966d670c354e4abc9804f1746c08ca237327ffffffffffffffff", 16)
A = int("0xaccb04d86dc88a32aa254819900b55a978893a2ce04189cb619909a84bf982d01eaafca12bda8ff42b5046361ee8058772309ddd8ce816bccec92e627aed7fd14fd609aea4253c7e42d1629dc28a54746527a676716952d04de4fb1a4b07a8ea182973400568b9cf93004c5b45ce4ce4f21527be5dfbfff42cb9ef7eb6726fe6b71878f8b48bc891a830c7e451aa217bffbd4a86cacfc57a65986a4cb827be7cc7ac3b9fb98a37d56b9e8cda7156484b3add054395c8b0fa3cbb3afbb8c987f", 16)
iv = "ebd3c617e8c4dd804dca34b7a48b2076"
encrypted = "58f4b780078d29d7c7d582d4814879dcd3c2debdbb2b7a75903ce681aafa62364b399cadb5f9befd7f2f33daeddc6265"
b = 1919572943691512325783103720167834163677411292709378502535498859989993544026380143919501049584589675317643993465536543895780854808442293000014297210200227069779643763121704810281976733978781152126062646602812482025293137787739116693980988513420732289020477701182639042794562638875881378349771734410919106042203493166198706573467903966100368713572415175654342828296086659529676015616513470105470901979846373335352656586302787870238998914215908919919219987614105175
s = pow(A, b, p)
print(decrypt_flag(s, iv, encrypted))
```
##### c) Kết quả:
`crypto{uns4f3_pr1m3_sm4ll_oRd3r}`

### IV. MISC:
#### 1. Script Kiddie:
##### a) Mô tả đề bài:
"*Found this cool script on Github and I've been using it to keep my secrets from anyone listening in on the school wifi!*"
::: spoiler script.py
```pyfrom Crypto.Cipher import AES
import hashlib
import secrets


def header():
    print("""  _____  _  __  __ _
 |  __ \(_)/ _|/ _(_)
 | |  | |_| |_| |_ _  ___
 | |  | | |  _|  _| |/ _ \
 | |__| | | | | | | |  __/
 |_____/|_|_| |_| |_|\___|
 | |  | |    | | |
 | |__| | ___| | |_ __ ___   __ _ _ __
 |  __  |/ _ \ | | '_ ` _ \ / _` | '_ \
 | |  | |  __/ | | | | | | | (_| | | | |
 |_|  |_|\___|_|_|_| |_| |_|\__,_|_| |_|

                                        """)


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def pkcs7_unpad(message, block_size=16):
    if len(message) == 0:
        raise Exception("The input data must contain at least one byte")
    if not is_pkcs7_padded(message):
        return message
    padding_len = message[-1]
    return message[:-padding_len]


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)
    return pkcs7_unpad(plaintext).decode('ascii')


def generate_public_int(g, a, p):
    return g ^ a % p


def generate_shared_secret(A, b, p):
    return A ^ b % p


def goodbye():
    print('Goodbye!')


def main():
    header()
    print('[-] Collecting data from Alice')
    p = int(input('> p: '))
    q = (p - 1) // 2
    g = int(input('> g: '))
    A = int(input('> A: '))
    print('[+] All data collected from Alice')

    print('[+] Generating public integer for alice')
    b = secrets.randbelow(q)
    B = generate_public_int(g, b, p)
    print(f'[+] Please send the public integer to Alice: {B}')
    print('')
    input('[+] Press any key to continue')
    print('')

    print('[+] Generating shared secret')
    shared_secret = generate_shared_secret(A, b, p)

    query = input('Would you like to decrypt a message? (y/n)\n')
    if query == 'y':
        iv = input('[-] Please enter iv (hex string)\n')
        ciphertext = input('[-] Please enter ciphertext (hex string)\n')
        flag = decrypt_flag(shared_secret, iv, ciphertext)
        print(f'[+] Flag recovered: {flag}')
        goodbye()
    else:
        goodbye()


if __name__ == '__main__':
    main()
```
:::
::: spoiler output.txt
```py
p: 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
g: 2
A: 539556019868756019035615487062583764545019803793635712947528463889304486869497162061335997527971977050049337464152478479265992127749780103259420400564906895897077512359628760656227084039215210033374611483959802841868892445902197049235745933150328311259162433075155095844532813412268773066318780724878693701177217733659861396010057464019948199892231790191103752209797118863201066964703008895947360077614198735382678809731252084194135812256359294228383696551949882
B: 652888676809466256406904653886313023288609075262748718135045355786028783611182379919130347165201199876762400523413029908630805888567578414109983228590188758171259420566830374793540891937904402387134765200478072915215871011267065310188328883039327167068295517693269989835771255162641401501080811953709743259493453369152994501213224841052509818015422338794357540968552645357127943400146625902468838113443484208599332251406190345653880206706388377388164982846343351
iv: 'c044059ae57b61821a9090fbdefc63c5'
encrypted_flag: 'f60522a95bde87a9ff00dc2c3d99177019f625f3364188c1058183004506bf96541cf241dad1c0e92535564e537322d7'
```
:::

##### b) Phân tích cách làm:
- Từ script.py, có thể thấy rằng hàm `generate_public_int(g, a, p)` trả về: $$(g \oplus a) \mod p$$ Thay vì: $$g^a \mod p$$ Tương tự, hàm `generate_shared_secret(A, b, p)` trả về: $$s = (A \oplus b) \mod p$$ Thay vì: $$s = A^b \mod p$$
- Ta có $q = \frac{p - 1}{2}$ và $b$ là một giá trị ngẫu nhiên trong khoảng $q$. Với giá trị $B$ đã có: $$B = g \oplus b \mod p$$ Tương đương: $$g \oplus b = k.p + B, \text{ } k \in N$$ Suy ra: $$b = (g \oplus b) \oplus g = (k.p + B) \oplus g, \text{ } k \in N$$
- Sau khi tìm được $b$ bằng cách brute - force $k$, ta tính khoá bí mật $s = (A \oplus b) \mod p$ và giải mã flag.
- Source code:
```py
from Crypto.Cipher import AES
from tqdm import trange
import hashlib


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def pkcs7_unpad(message, block_size=16):
    if len(message) == 0:
        raise Exception("The input data must contain at least one byte")
    if not is_pkcs7_padded(message):
        return message
    padding_len = message[-1]
    return message[:-padding_len]


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)
    return pkcs7_unpad(plaintext).decode('ascii')


p = 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
g = 2
A = 539556019868756019035615487062583764545019803793635712947528463889304486869497162061335997527971977050049337464152478479265992127749780103259420400564906895897077512359628760656227084039215210033374611483959802841868892445902197049235745933150328311259162433075155095844532813412268773066318780724878693701177217733659861396010057464019948199892231790191103752209797118863201066964703008895947360077614198735382678809731252084194135812256359294228383696551949882
B = 652888676809466256406904653886313023288609075262748718135045355786028783611182379919130347165201199876762400523413029908630805888567578414109983228590188758171259420566830374793540891937904402387134765200478072915215871011267065310188328883039327167068295517693269989835771255162641401501080811953709743259493453369152994501213224841052509818015422338794357540968552645357127943400146625902468838113443484208599332251406190345653880206706388377388164982846343351
iv = 'c044059ae57b61821a9090fbdefc63c5'
encrypted_flag = 'f60522a95bde87a9ff00dc2c3d99177019f625f3364188c1058183004506bf96541cf241dad1c0e92535564e537322d7'

q = (p - 1)//2
for k in trange (2**24):
    gXORb = k*p + B
    b = gXORb^g
    if b < q:
        print("b =", b)
    s = (A^b)%p
    flag = decrypt_flag(s, iv, encrypted_flag)
    print(f'[+] Flag recovered: {flag}')
    break
```

##### c) Kết quả:
`crypto{b3_c4r3ful_w1th_y0ur_n0tati0n}`

#### 2. The Matrix:
