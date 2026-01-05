### I. Microsoft Office Documents:
Có hai loại file formats chính được dùng bởi Microsoft Office documents:

#### 1. OLE (Object Linking and Embedding):
- Là file format được dùng trong các versions của Microsoft Office giữa 1997 và 2003. Nó định nghĩa một cấu trúc "file trong file" cho phép các files khác được nhúng trong một file. Ví dụ, một sheet Excel có thể được nhúng trong một tài liệu Word.
- Nó hỗ trợ các phần mở rộng file như `.rtf`, `.doc`, `.ppt`, `xls`,...

#### 2. OOXML (Office Open XML):
##### a) Định nghĩa:
- File format đang được dùng trong Microsoft Office, dựa vào một XML - based format cho các tài liệu office.
- Phần mở rộng cho các documents này gồm `.docx`, `pptx`, `xlsx`,...
- OOXML format lưu trữ Office documents dưới dạng tệp ZIP. Điều này có nghĩa là các tài liệu như Word, Excel và PowerPoint thực chất chỉ là ZIP files. Bằng cách đổi tên phần mở rộng từ `docx`, `xlsx` hay `.pptx` thành `.zip`, ta có thể giải nén nội dung tài liệu lưu trữ và xem XML files cá nhân. Đây là một feature hữu ích cho digital forensics, như cách nó cho phép việc điều tra nội dung của một tài liệu mà không cần thay đổi nó.

##### b) Cấu trúc của một tài liệu OOXML:
Xét ví dụ, tạo một tài liệu Word với text là `Hello World!`, chuyển nó sang máy Kali rồi giải nén nó:
```kali
$ unzip Doc1.docx                       
Archive:  Doc1.docx
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: word/document.xml       
  inflating: word/_rels/document.xml.rels  
  inflating: word/theme/theme1.xml   
  inflating: word/settings.xml       
  inflating: word/styles.xml         
  inflating: word/webSettings.xml    
  inflating: word/fontTable.xml      
  inflating: docProps/core.xml       
  inflating: docProps/app.xml
```
Cấu trúc của một tài liệu Word:
```
.
├── [Content_Types].xml
├── docProps
│   ├── app.xml
│   └── core.xml
├── _rels
│   └── .rels
└── word
    ├── document.xml
    ├── fontTable.xml
    ├── _rels
    │   └── document.xml.rels
    ├── settings.xml
    ├── styles.xml
    ├── theme
    │   └── theme1.xml
    └── webSettings.xml
```
- `[Content_Types].xml`: File này chứa thông tin về các loại content có trong tài liệu và file extensions tương ứng của chúng.
- `docProps`: Folder này gồm hai files:
    - `app.xml`: Chứa thông tin về app được dùng để tạo ra tài liệu.
    ```xml
    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <Properties xmlns="http://schemas.openxmlformats.org/officeDocument/2006/extended-properties" xmlns:vt="http://schemas.openxmlformats.org/officeDocument/2006/docPropsVTypes">
      <Template>Normal.dotm</Template>
      <TotalTime>0</TotalTime>
      <Pages>1</Pages>
      <Words>1</Words>
      <Characters>12</Characters>
      <Application>Microsoft Office Word</Application>
      <DocSecurity>0</DocSecurity>
      <Lines>1</Lines>
      <Paragraphs>1</Paragraphs>
      <ScaleCrop>false</ScaleCrop>
      <Company/>
      <LinksUpToDate>false</LinksUpToDate>
      <CharactersWithSpaces>12</CharactersWithSpaces>
      <SharedDoc>false</SharedDoc>
      <HyperlinksChanged>false</HyperlinksChanged>
      <AppVersion>16.0000</AppVersion>
    </Properties>
    ```
    - `core.xml`: Chứa metadata của tài liệu, như tên author, thời gian tạo, thời gian điều chỉnh.
    ```xml
    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <cp:coreProperties xmlns:cp="http://schemas.openxmlformats.org/package/2006/metadata/core-properties" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:dcterms="http://purl.org/dc/terms/" xmlns:dcmitype="http://purl.org/dc/dcmitype/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <dc:title/>
      <dc:subject/>
      <dc:creator>Saad Javed</dc:creator>
      <cp:keywords/>
      <dc:description/>
      <cp:lastModifiedBy>Saad Javed</cp:lastModifiedBy>
      <cp:revision>2</cp:revision>
      <dcterms:created xsi:type="dcterms:W3CDTF">2023-02-04T15:44:00Z</dcterms:created>
      <dcterms:modified xsi:type="dcterms:W3CDTF">2023-02-04T15:44:00Z</dcterms:modified>
    </cp:coreProperties>
    ```
- `_rels`: Folder này gồm file `.rels` - chứa thông tin về mối quan hệ giữa các phần khác nhau của tài liệu ví dụ như đối với `app.xml` và `core.xml`.
- `word`: Folder này chứa nội dung thực tế của tài liệu:
    - `document.xml`: Phần text thực tế của tài liệu.
    ```xml
    <!-- SNIP -->
      <w:body>
        <w:p w14:paraId="68F74602" w14:textId="442B891B" w:rsidR="00D473D4" w:rsidRPr="00B20485" w:rsidRDefault="00B20485">
          <w:pPr>
            <w:rPr>
              <w:lang w:val="en-US"/>
            </w:rPr>
          </w:pPr>
          <w:r>
            <w:rPr>
              <w:lang w:val="en-US"/>
            </w:rPr>
            <w:t>Hello World!</w:t>
          </w:r>
        </w:p>
        <w:sectPr w:rsidR="00D473D4" w:rsidRPr="00B20485">
          <w:pgSz w:w="11906" w:h="16838"/>
          <w:pgMar w:top="1440" w:right="1440" w:bottom="1440" w:left="1440" w:header="708" w:footer="708" w:gutter="0"/>
          <w:cols w:space="708"/>
          <w:docGrid w:linePitch="360"/>
        </w:sectPr>
      </w:body>
    </w:document>
    ```
    - `fontTable.xml`: Chứa thông tin về các fonts được dùng trong tài liệu.
    - `_rels`: Chứa file `document.xml.rels` - Gồm thông tin về mối quan hệ giữa các phần khác nhau của tài liệu như styles, themes, settings cũng như URLs cho links bên ngoài.
    - `settings.xml`: Chứa document settings và thông tin cấu hình.
    - `styles.xml`: Chứa thông tin về styles được dùng trong tài liệu.
    - `theme`: Chứa các filé về theme được dùng trong tài liệu. `theme1.xml` chứa theme thực tế của content.
    - `webSettings.xml`: Chứa thông tin về web - specific settings của tài liệu, như HTML frameset settings cũng như cách tài liệu được xử lý khi saved as HTML.
- Thông tin chi tiết hơn về bất kỳ additional files có thể hiển thị trong một tài liệu ở [Anatomy of a WordProcessingML File](http://officeopenxml.com/anatomyofOOXML.php).

### II. Macro - Enabled Documents:
- Macro - Enabled documents là tài liệu chứa các macros - những tập hợp chỉ thị để tự động hoá các tác vụ. Macros có thể được viết trong Visual Basic cho App (VBA) và có thể được dùng để thực hiện nhiều tác vụ khác nhau như định dạng text, thực hiện tính toán và tự động thực hiện các tiến trình phức tạp. Tuy nhiên, attackers thường dùng chức năng này của tài liệu Office để thực hiện phishing attacke mà nhúng các macros độc hại để thực hiện các hành động nguy hiển và cài đặt malware trên hệ thống.
- Các extensions cho các tài liệu này gồm `.docm`, `.pptm`, `xlsm`,...
> Xét một ví dụ, tạo một tài liệu Word mới và thực hiện các bước:
> - Click `View` $\rightarrow$ `Macros` $\rightarrow$ `View Macros`:
> ![image](https://hackmd.io/_uploads/H17w14rXWx.png)
> - Gõ một cái tên như `HelloWorld`, chọn `Document1` (cái hiện tại) dưới `Macros` và chọn `Create`:
> ![image](https://hackmd.io/_uploads/ryq_y4H7-l.png)
> - Paste đoạn code sau vào phần text box:
> ```
> Sub HelloWorld()
>
> Dim doc As Document
> Set doc = Word.ActiveDocument
> doc.Content.InsertAfter ("Hello, World!")
> 
> End Sub
> ```
> ![image](https://hackmd.io/_uploads/BJfhyEBm-x.png)
> - Đóng Microsoft Visual Basic for Application tab.
> - Lặp lại bước đầu tiên, chọn macro `HelloWord` và click `Run`:
> ![image](https://hackmd.io/_uploads/Bkcp1NBm-e.png)
> - Quan sát rằng `Hello, World!` đang được tự động viết trên document.
> - Lưu tài liệu dưới dạng `.docm`.

- Theo cấu trúc của các OOXML files, macro đang nằm trong `word/vbaProject.bin`, tuy nhiên ta sẽ không thể đọc nó dưới dạng binary. Ta có thể dùng bộ tools gọi là `oletools` để phân tích và trích macros từ OLE files như Microsoft Office Documents.
- Command để tải `oletools`: `sudo -H pip install -U oletools`
- Dùng `oleid` để phát hiện xem tài liệu có chứa bất kỳ macros nào được nhúng ở trong không:
```
$ oleid HelloWorld.docm   
XLMMacroDeobfuscator: pywin32 is not installed (only is required if you want to use MS Excel)
oleid 0.60.1 - http://decalage.info/oletools
THIS IS WORK IN PROGRESS - Check updates regularly!
Please report any issue at https://github.com/decalage2/oletools/issues

Filename: HelloWorld.docm
WARNING  For now, VBA stomping cannot be detected for files in memory
--------------------+--------------------+----------+--------------------------
Indicator           |Value               |Risk      |Description               
--------------------+--------------------+----------+--------------------------
File format         |MS Word 2007+ Macro-|info      |                          
                    |Enabled Document    |          |                          
                    |(.docm)             |          |                          
--------------------+--------------------+----------+--------------------------
Container format    |OpenXML             |info      |Container type            
--------------------+--------------------+----------+--------------------------
Encrypted           |False               |none      |The file is not encrypted 
--------------------+--------------------+----------+--------------------------
VBA Macros          |Yes                 |Medium    |This file contains VBA    
                    |                    |          |macros. No suspicious     
                    |                    |          |keyword was found. Use    
                    |                    |          |olevba and mraptor for    
                    |                    |          |more info.                
--------------------+--------------------+----------+--------------------------
XLM Macros          |No                  |none      |This file does not contain
                    |                    |          |Excel 4/XLM macros.       
--------------------+--------------------+----------+--------------------------
External            |0                   |none      |External relationships    
Relationships       |                    |          |such as remote templates, 
                    |                    |          |remote OLE objects, etc   
--------------------+--------------------+----------+--------------------------
```
- Kết quả cho thấy rằng tool tìm thấy VBA Macros và ước lượng rủi ro cỡ medium. Có thể tiếp tục dùng `olevba` để trích macros từ tài liệu:
```
$ olevba HelloWorld.docm
XLMMacroDeobfuscator: pywin32 is not installed (only is required if you want to use MS Excel)
olevba 0.60.1 on Python 3.10.8 - http://decalage.info/python/oletools
===============================================================================
FILE: HelloWorld.docm
Type: OpenXML
WARNING  For now, VBA stomping cannot be detected for files in memory
-------------------------------------------------------------------------------
VBA MACRO ThisDocument.cls 
in file: word/vbaProject.bin - OLE stream: 'VBA/ThisDocument'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO NewMacros.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/NewMacros'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub HelloWorld()

Dim doc As Document
Set doc = Word.ActiveDocument
doc.Content.InsertAfter ("Hello, World!")

End Sub
No suspicious keyword or IOC found.
```

### III. Steganography:
- Từ `steganography` (/ˌstɛɡəˈnɒɡrəfi/) đến từ `steganographia` (từ Hy Lạp), kết hợp bởi `steganos` (nghĩa là `covered or concealed`: bị che đậy hay giấu kín) với `graphia` (nghĩa là `to write`: viết). Nó chứa các bí mật ẩn giấu trong phần thông tin nhìn có vẻ vô hại. Một ví dụ của steganography là dùng mực tàng hình làm từ nước chanh hoặc giấm để viết lên giấy và tiết lộ chúng bằng cách hơ lửa tờ giấy đó.
- Trong thế giới kỹ thuật số ngày nay, steganography dùng để giấu message trong file nào đó, ví dụ như image hay audio file, theo cách nó không thể bị phát hiện hay nghe thấy bởi bất kì ai không biết sự tồn tại của nó. Công nghệ này có thể dùng cho mục đích vô hại, như gửi tin nhắn bí mật tới bạn bè, hay cho lý do nguy hiểm, như che đậy bằng chứng hay liên lạc mà không bị phát hiện.
> Theo Wikipedia: Những ghi chép đầu tiên về việc sử dụng kỹ thuật giấu tin có thể được tìm thấy ở Hy Lạp vào năm 440 trước Công nguyên, khi Herodotus đề cập đến hai ví dụ trong tác phẩm Sử ký của mình. Histiaeus đã gửi một thông điệp cho chư hầu của mình, Aristagoras, bằng cách cạo đầu người hầu cận đáng tin cậy nhất của ông, "đánh dấu" thông điệp lên da đầu của người đó rồi cho người đó lên đường sau khi tóc mọc lại với lời dặn dò: "Khi ngươi đến Miletus, hãy bảo Aristagoras cạo đầu ngươi và xem thông điệp đó."

- Một số ví dụ khác:
    - [Malicious Memes that Communicate with Malware](https://www.trendmicro.com/en_us/research/18/l/cybercriminals-use-malicious-memes-that-communicate-with-malware.html)
    - [Using steganography to obfuscate PDF exploits](https://securityaffairs.com/80342/hacking/steganography-obfuscate-pdf.html)
- Một số tool: [GitHub](https://github.com/karthik997/Forensic_Toolkit?utm_source=chatgpt.com), [toolkit](https://github.com/DominicBreuker/stego-toolkit), [Cheatsheet](https://neerajlovecyber.com/steganography-cheatsheet-for-ctf-beginners)

#### 1. Image Steganography:
##### a) Định dạng ảnh phổ biến:
PNG và JPEG là hai dạng image formats phổ biến, chúng có thể được dùng như một kênh để ẩn messages. Cả hai formats này đều dựa vào cấu trúc khác nhau để tạo nên một image, do đó có các công nghệ khác nhau để ẩn messages.

###### PNG:
- Với file PNG, một trong những công nghệ phổ biến để ẩn messages là điều chỉnh các bits ít quan trọng nhất (the Least Significant Bits - LSBs) của các pixels nào đó như những cái có tác động tối thiểu đến chất lượng của image. Thuật toán sau đó sẽ biết cần trích message được nhúng trong pixels nào.
- Tool dùng để dò PNG steganography là `zsteg`, cài đặt bằng command `sudo gem install zsteg`.
> - Xét ví dụ, bức Starry Night của Van Gogh ([link tải](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2003/files/starry_night.png)) chứa một message được nhúng trong nó.
> - Cách dùng tool để trích message ẩn:
> ``` ubuntu
> $ zsteg starry_night.png           
> b1,rgb,lsb,xy       .. text: "148:The fishermen know that the sea is dangerous and the storm terrible, but they have never found these dangers sufficient reason for remaining ashore."
> b2,r,lsb,xy         .. file: OpenPGP Public Key
> b4,g,lsb,xy         .. text: "*e~_^u|["
> ```
> ![image](https://hackmd.io/_uploads/HJWL9NrXZx.png)

###### JPEG (hay JPG):
- Với file JPEG/JPG, phương pháp steganography phổ biến gồm tìm cặp vị trí trong image sao cho việc trao đổi giá trị của chúng có tác dụng nhúng phần message bí mật tương ứng. Nếu không có cặp nào được tìm thấy, các pixels trong những vị trí còn lại đơn giản là bị ghi đè lên.
- Dùng tool `steghide` để ẩn và trích các messages khỏi JPEG files.
- Một tool khác là `exiftool`, nó hữu dụng trong việc trích metadata từ các files như image hay audio (chứa thông tin như thời gian tạo và điều chỉnh, author và vị trí chụp ảnh (kinh độ, vĩ độ)).

###### GIF:
- GIF dễ bị khai thác vì đa frame, palette và timing:
    - Nhiều frame ẩn.
    - Thay đổi delay time.
    - Dữ liệu giấu trong LSB của từng frame.
    - Trailing data sau EOF.
    - Palette bị chỉnh.
- Tool nên dùng:
    - `gifsicle`: Xem và tách frame.
    ```
    gifsicle --info file.gif
    gifsicle --explode file.gif
    ```
    - `ImageMagick`: Convert frame thành PNG để phân tích sâu hơn.
    ```
    convert file.gif frame_%03d.png
    identify -verbose file.gif
    ```
    - `Stegsolve`: Browse từng frame hoặc XOR, LSB, bit plane.
    - `binwalk`: Check data nhúng.
    ```
    binwalk file.gif
    binwalk -e file.gif
    ```
###### TIFF:
- TIF hỗ trợ multi - page, metadata cực nhiều và có thể chứa raw binary data. Những thứ cần kiểm tra:
    - Multiple pages
    - Compression lạ
    - Metadata custom tag
    - Dữ liệu sau IFD
    - Bit depth khác 8
- Các tool nên dùng: `tiffinfo`, `tiffsplit`, `exiftool`, `ImgaeMagick`, `binwalk`,...

###### BMP:
- BMP (bitmap Windows) xử lý các tập tin đồ hoạ trong hệ điều hành Microsoft Windows. Thông thường, các tập tin BMP không được nén nên nó lớn và mất dữ liệu. Lợi thế của nó là cấu trúc đơn giản và được chấp nhận rộng rãi trong các chương trình Windows.
- Những thứ nên kiểm tra:
    - LSB (RGB)
    - Padding bytes
    - Header bị sửa
    - Data sau pixel array
- Tool nên dùng: `zsteg`, `Stegsolve`, `binwalk`, `xxd`/`hexdump`, `ImageMagick`,...

##### b) Đọc RGB của ảnh:
- Với một ảnh có kích thước `W x H`: Mỗi pixel có RGB (8-bit): `(R, G, B)` $\in$ `[0 - 255]` hoặc RGBA, Grayscale, 16-bit,...
- Đọc RGB là duyệt từng pixel và lấy ra `(R, G, B)` của nó.
- Tool: ImageMagick (dùng với ảnh nhỏ), Stegsolve.
- Code Python (Pillow):
`pip install pillow`
```py
from PIL import Image

input_image = "image.png"
output = "rgb.txt"

img = Image.open(input_image).convert("RGB")
width, height = img.size
pixels = img.load()

with open(output, "w", encoding = "utf-8") as f:
    f.write(f"# Image size: {width} x {height}\n")
    f.write("# Format: x y R G B\n\n")
    for y in range(height):
        for x in range(width):
            r, g, b = pixels[x, y]
            f.write(f"{x} {y} {r} {g} {b}\n")
```

##### c) Code giấu tin:
- Giấu một chuỗi vào bit 1 của Red, bit 0 của Green và bit 2 của Blue.
``` py
from PIL import Image

input_img = "input.png"
output_img = "stego.png"
flag = "Ha Nguyen"


def text_to_bits(text):
    bits = ""
    for c in text:
        bits += format(ord(c), "08b")
    bits += "00000000"
    return bits

img = Image.open(input_img).convert("RGB")
pixels = img.load()
w, h = img.size

bits = text_to_bits(flag)
bit_idx = 0
max_bits = w*h*3

if len(bits) > max_bits:
    print("Ảnh quá nhỏ để giấu flag!")

for y in range(h):
    for x in range(w):
        if bit_idx >= len(bits):
            break
        r, g, b = pixels[x, y]

        #bit 1 Red
        if bit_idx < len(bits):
            r = (r & ~(1 << 1)) | (int(bits[bit_idx]) << 1)
            bit_idx += 1

        #bit 0 Green
        if bit_idx < len(bits):
            g = (g & ~1) | int(bits[bit_idx])
            bit_idx += 1

        #bit 2 Blue
        if bit_idx < len(bits):
            b = (b & ~(1 << 2)) | (int(bits[bit_idx]) << 2)
            bit_idx += 1

        pixels[x, y] = (r, g, b)

    if bit_idx >= len(bits):
        break

img.save(output_img)
```
- Code giải mã:
``` py
from PIL import Image

stego_img = "stego.png"
img = Image.open(stego_img).convert("RGB")
pixels = img.load()
w, h = img.size
bits = ""

for y in range(h):
    for x in range(w):
        r, g, b = pixels[x, y]
        bits += str((r >> 1) & 1)  #bit 1 Red
        bits += str(g & 1)         #bit 0 Green
        bits += str((b >> 2) & 1)  #bit 2 Blue

m = ""
for i in range(0, len(bits), 8):
    byte = bits[i:i+8]
    if len(byte) < 8:
        break
    char = chr(int(byte, 2))
    if char == "\x00":
        break
    m += char
print(m)
```

#### 2. Audio Steganography:
- Trong audio steganography, cách phổ biến nhất là nhúng message trong phổ âm thanh, tuy nhiên cách này làm audio trở nên ồn ào. Một phương pháp khác là nhúng thông điệp vào những bits ít quan trọng (LSBs).
- Dùng các tool như Sonic Visualizer hay Audacity để dò audio steganography.
> Xét một audio file ([link tải](https://github.com/vonderchild/digital-forensics-lab/blob/main/Lab%2003/files/super_secret_audio.wav)), mở nó trong Sonic Visualizer và click `Layer` $\rightarrow$ `Add Spectogram`:
> ![image](https://hackmd.io/_uploads/HyL7a4BQZl.png)
- Những cách khác:
    - Phase Coding: Thay đổi pha của sóng âm, spectrogram sẽ không rõ chữ. Tool nên dùng là tool custom hoặc script Python.
    - Echo hiding: Thêm echo rất nhỏ với khoảng trễ bằng bit 0/1, khi đó âm thanh sẽ hơi vang và phổ tần có pattern lặp. Tool nên dùng: Sonic Visualizer, MATLAB/Python.
    - Spread Spectrum: Trải dữ liệu ra toàn bộ tín hiệu giống như kỹ thuật truyền thông, thường rất khó detect và cần key. Dùng tool nghiên cứu hoặc script custom.
    - Spectrogram Steganography: Vẽ chữ/hình/flag trong spectrogram và chỉ thấy được khi chuyển sang miền tần số. Tool nên dùng: Audacity (`Tracks` $\rightarrow$ `Spectrogram` với `Scale` là `Log`), Sonic Visualiser.
    - Metadata/Comment Field: Giấu flag trong ID3 tag, comment hoặc Artist/Album, dấu hiệu là file mp3/flac hoặc metadata bất thường. Dùng `exfitool` hoặc `mediainfo` để phân tích.
    - Trailing Data (Data sau EOF): Append file ZIP/TXT sau audio, khi đó file size sẽ lớn bất thường và binwalk thấy signature khác. Thường dùng `binwalk`, `xxd` và `strings` để phân tích.
    - Silence Encoding: Độ dài đoạn im lặng bằng bit/ký tự, khi đó sẽ có nhiều đoạn silence kỳ lạ và duration không đều. Dùng Audacity (zoom waveform) hoặc Python để phân tích.
    - Stereo Channel Encoding (L/R): Thường `Left = 0`, `Right = 1` hoặc XOR giữa hai kênh, dấu hiệu sẽ là stereo file hoặc L/R channel khác nhau rõ. Dùng tool Audacit (Split Stereo) hoặc Python để phân tích.
    - Hybrid/Trick CTF: Thường kết hợp:
        - Base64 - LSB
        - XOR - Spectrogram
        - Morse - Beep
        - DTMF tones - SĐT - ASCII
