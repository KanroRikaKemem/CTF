> Tham khảo ở [Cyber Kill Chains: Strategies & Tactics](https://www.splunk.com/en_us/blog/learn/cyber-kill-chains.html)
---
> **Những điểm chính cần ghi nhớ:**
> - Cyber Kill Chain là một khung chia nhỏ một cuộc tấn công mạng thành bảy bước - từ trinh sát cho đến hành động nhắm đến mục tiêu - giúp các tổ chức hiểu được, bảo vệ, và ngăn chặn các mối đe doạ ở mỗi bước.
> - Việc làm gián đoạn một cuộc tấn công ở bất kỳ bước nào của kill chain có thể làm giảm đáng kể rủi ro và ảnh hưởng của các mối đe doạ về mạng, và việc tập trung sớm hơn vào việc phòng thủ trong chuỗi có thể giới hạn hơn nữa các thiệt hại tiềm tàng.
> - Bằng cách tận dụng tối đa mô hình Cyber Kill Chain, đặc biệt là khi kết hợp với hiểu biết về mối đe doạ, sự tự động, và các quy trình bảo mật thống nhất, cho phép chủ động bảo vệ và phản ứng với sự cố nhanh hơn xuyên suốt tất cả các giai đoạn tấn công.

- Đôi khi được nhắc tới là CKC hay "vòng đời của cuộc tấn công mạng", Cyber Kill Chain là một mô hình phòng thủ an ninh được phát triển để tìm kiếm và ngăn chặn các cuộc tấn công mạng ([sophisticated cyberattacks](https://www.splunk.com/en_us/blog/learn/cybersecurity-attacks.html)) *trước khi* chúng gây ảnh hưởng tới một tổ chức.
- Thông thường bao gồm bảy bước, một mô hình Cyber Kill Chain chia nhỏ các bước của một cuộc tấn công mạng, cho phép các đội bảo mật nhận diện, chặn bắt, hoặc ngăn chặn chúng. Việc sử dụng một khung Cyber Kill Chain có thể giúp các tổ chức hiểu tố hơn về các mối đe doạ liên quan và cải thiện việc [quản lý và ứng phó sự cố](https://www.splunk.com/en_us/blog/learn/incident-management.html).
- Nếu thực hiện đúng cách, Cyber Kill Chain có thể có nhiều lợi ích bảo mật quan trọng - nhưng nếu thực hiện không đúng, chúng có thể khiến tổ chức đối mặt với các rủi ro. Những thiếu sót trong chuỗi tấn công dẫn đến những câu hỏi về tương lai của nó. Tuy nhiên, các doanh nghiệp vẫn có thể dùng phương pháp Cyber Kill Chain để hình thành các chiến lược bảo mật mạng của họ.
- Ngoài ra, phương pháp Cyber COBRA là một công cụ mới hơn giúp bạn tích hợp vào một chiến lược Cyber Kill Chain để cung cấp nhiều đánh giá về các mối đe doạ một cách chủ động và phù hợp với bối cảnh hơn.

## I. Chuỗi tấn công (kill chain) trong an ninh mạng là gì?
- Một Cyber Kill Chain là một khung bảo mật được thiết kế để tìm kiếm các cuộc [tấn công mạng](https://www.splunk.com/en_us/blog/learn/cybersecurity-attacks.html) bằng cách chia nhỏ cuộc tấn công ra thành nhiều bước. Mô hình này giúp các [teams bảo mật](https://www.splunk.com/en_us/blog/learn/cybersecurity-jobs-skills-responsibilities.html) nhận diện, chặn bắt, hoặc ngăn chặn các cuộc tấn công trước khi chúng ảnh hướng tới tổ chức.
- Concept của một chuỗi tấn công bắt nguồn từ các hoạt động quân sự, nơi một cuộc tấn công từ kẻ thù được nhận diện, chia nhỏ thành các bước, và các biện pháp phòng ngừa được thực hiện. Chiến lược quân sự này đã tạo cảm hứng cho nguồn gốc của chuỗi tấn công mạng, được tạo ra đầu tiên bởi Lockheed Martin vào năm 2011.
- Mục đích của Cyber Kill Chain là để cải thiện việc phòng thủ chống lại các mối đe doạ dai dẳng nâng cao ([Advanced Persistent Threats - APTs](https://www.splunk.com/en_us/blog/learn/apts-advanced-persistent-threats.html)), cũng được biết tới như các cuộc tấn công mạng. Những mối đe doạ phổ biến gồm:
    - Mã độc (Malware)
    - Phần mềm tống tiền ([Ransomware](https://www.splunk.com/en_us/blog/learn/ransomware-attack-types.html))
    - Trojan horses
    - [Phishing](https://www.splunk.com/en_us/blog/learn/phishing-scams-attacks.html)
    - Những kỹ thuật [social engineering](https://www.splunk.com/en_us/blog/learn/social-engineering-attacks.html) khác
- Cyber Kill Chains cho phép các doanh nghiệp chuẩn bị và đi trước các hackers một bước ở mọi bước của một cuộc tấn công, từ bối cảnh hoá cho đến thực thi. Việc kết hợp chặt chẽ Cyber COBRA vào quá trình của Kill Chain có thể nâng cao khung bảo mật bằng cách cung cấp đánh giá chi tiết hơn và cụ thể hơn về các mối đe doạ tiềm tàng, do đó cải thiện các chiến thuật phòng thủ.

### 1. Cyber Kill Chain vs. MITRE ATT&CK:
- Cyber Kill Chain thường được so sánh với khung [MITRE ATT&CK](https://attack.mitre.org/), một phương pháp của Liên đoàn MITRE. MITRE ATT&CK minh hoạ tương tự các giai đoạn của một cuộc tấn công mạng, nhiều cái trong số đó tương tự với mô hình Cyber Kill Chain.
> Sự khác biệt chính giữa Cyber Kill Chain và MITRE ATT&CK là các chiến lược MITRE được liệt kê mà không có thứ tự cụ thể - không giống như việc phân nhóm các bước cụ thể và cấu trúc tuyến tính của Cyber Kill Chain.
- Một sự khác biệt khác là khung Cyber Kill Chain chỉ tới quá trình tấn công mạng bảy bước ở mức cao, trong khi MITRE ATT&CK phân tích nhiều kỹ thuật và thủ tục khác nhau liên quan tới các thông tin chi tiết của một cuộc tấn công mạng.
![image](https://hackmd.io/_uploads/r1mEVixwMl.png)
- Phương pháp Cyber COBRA ở dưới có thể bổ sung cho cả hai khung bảo mật bằng cách thêm một [hệ thống chấm điểm để đánh giá mức độ đe doạ](https://www.splunk.com/en_us/blog/learn/risk-scoring.html) dựa trên ngữ cảnh cụ thể của cuộc tấn công, cho phép ưu tiên nhiều nỗ lực ứng phó chính xác hơn.
- Các thành phần của cả Cyber Kill Chain và ATT&CK có thể kết hợp chặt chẽ với nhau trong [chiến lược bảo mật mạng](https://www.splunk.com/en_us/blog/learn/cybersecurity.html#creating-cybersecurity-strategy).
> Tìm hiểu thêm về một khung bổ sung: [How to operationalize MITRE ATT&CK](https://www.splunk.com/en_us/blog/security/answered-your-most-burning-questions-about-planning-and-operationalizing-mitre-attack.html) & [MITRE D3FEND](https://www.splunk.com/en_us/blog/learn/mitre-defend.html)

## 2. Cyber COBRA (Contextual Objective Rating):
- Đây là một khung đánh giá an ning mạng được thiết kế để cung cấp sự hiểu biết chủ động và sâu sắc hơn về tình hình bảo mật của một tổ chức. Không giống như những khung bảo mật khác được thành lập như Cyber Kill Chain hay MITRE ATT&CK, các khung tập trung vào các bước cụ thể của các cuộc tấn công hay các chiến thuật, kỹ thuật, và quy trình ([TTPs](https://www.splunk.com/en_us/blog/learn/ttp-tactics-techniques-procedures.html)), Cyber COBRA nhấn mạnh ngữ cảnh theo thời gian thực. Được phát triển nhằm đáp ứng cho bối cảnh mối đe doạ trong mạng càng ngày càng phức tạp, nó tích hợp sự giám sát liên tục và phân tích ngữ cảnh để đánh giả các rủi ro dựa trên:
    - Môi trường hiện tại của mối đe doạ.
    - Giá trị của tài sản có nguy cơ gặp rủi ro
    - Các chiến thuật được dùng bởi đối thủ.
- Hướng tiếp cận này cho phép các tổ chức ưu tiên các phương pháp phòng thủ hiệu quả hơn, điều chỉnh những sự thay đổi thay vì chỉ phụ thuộc vào các đánh giá tĩnh định kỳ. Cyber COBRA thể hiện cách tiếp cận tiên tiến tới an ninh mạng phù hợp với xu hướng chuyển dịch tới các chiến lược phòng thủ chủ động và đáp ứng hơn của công nghiệp.
- Hãy khám phá các concepts cơ bản trong [Lockheed Martin’s Cyber COBRA whitepaper](https://www.lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Contextual-OBjective-RAting(COBRA)_FINAL.pdf). Tiếp đó, quan sát cuộc thảo luận về việc sử dụng ngày càng phát triển của các khung an ninh mạng trong các báo cáo công nghiệp như [SANS Institute's study](https://www.sans.org/reading-room/whitepapers/analyst/contextual-based-security-2021-40040) với ngữ cảnh dựa bảo mật, và trong các cộng đồng an ninh mạng như ISACA, sẽ nhấn mạnh tầm quan trọng của [các chiến lược nhận thức ngữ cảnh bảo mật](https://www.isaca.org/resources/news-and-trends/newsletters/atisaca/2023/volume-15/five-key-considerations-when-developing-a-collaboration-strategy-for-information-risk-and-security). Các nguồn này sẽ cung cấp sự hiểu biết đa dạng về cách những khung bảo mật như Cyber COBRA bắt đầu được tích hợp vào thực tiễn an ninh mạng trong thế giới thực.

## II. Bảy bước của một Cyber Kill Chain:
![image](https://hackmd.io/_uploads/rJUFZ8ZDzg.png)
Mô hình Cyber Kill Chain ban đầu của Lockheed Martin mô tả bảy bước. Đây là khuôn mẫu được đề cập tới phổ biến nhất trong ngành công nghiệp. Nó đưa ra  phương pháp và động cơ của tội phạm mạng trong suốt toàn bộ attack timeline, giúp cho các tổ chức hiểu được và đối phó với những mối đe doạ. Bảy giai đoạn đó là:

### Giai đoạn 1 - Trinh sát:
Kẻ tấn công thu thập thông tin về mục tiêu của họ để tìm ra các lỗ hổng và những điểm thâm nhập tiềm năng. Giai đoạn này bao gồm:
- Thu thập dữ liệu công khai ([OSINT](https://www.splunk.com/en_us/blog/learn/open-source-intelligence-osint.html))
- Triển khai các công cụ do thám
- Sử dụng các phần mềm quét tự động để dò tìm những hệ thống bảo mật hay các phần mềm thứ ba.
> Tìm hiểu thêm về [Cách các lỗ hổng bảo mật liên quan đến mối đe doạ và sự rủi ro](https://www.splunk.com/en_us/blog/learn/vulnerability-vs-threat-vs-risk.html).

### Giai đoạn 2 - Trang bị vũ khí tấn công:
Sau khi thu thập thông tin, kẻ tấn công tạo ra malware hoặc các payloads độc hại để khai thác các điểm yếu đã tìm được. Quá trình này có thể bao gồm việc thiết kế một dạng thức malware mới hay điều chỉnh những chương trình đang có để kết nối với các lỗ hổng cụ thể.

### Giai đoạn 3 - Phát tán:
Kẻ tấn công cố gắng thâm nhập vào mạng lưới của mục tiêu bằng cách phát tán malware. Những phương pháp phổ biến bao gồm gửi phishing emails, sử dụng các công cụ social engineering, và khai thác những lỗ hổng về phần cứng hoặc phần mềm.
> Tài liệu liên quan: [Các loại lỗ hổng bảo mật](https://www.splunk.com/en_us/blog/learn/vulnerability-types.html)

### Giai đoạn 4 - Khai thác:
Một khi malware được phát tán, kẻ tấn công sẽ khai thác các lỗ hổng của mục tiêu để thâm nhập sâu hơn vào mạng. Chúng thường di chuyển theo chiều ngang giữa các hệ thống để tìm ra các điểm xâm nhập và điểm yếu tiềm năng.

### Giai đoạn 5 - Cài đặt
Trong giai đoạn này, kẻ tấn công cài đặt malware để chiếm quyền kiểm soát bổ sung đối với mạng. Các chiến lược bao gồm sử dụng Trojan horses, thao túng token truy cập, các giao diện command-line, và backdoors để leo thang đặc quyền và thay đổi quyền hạn.

### Giai đoạn 6 - Thực thi và Kiểm soát:
Kẻ tấn công thiết lập một [kênh Command and Control (C2)](https://www.splunk.com/en_us/blog/learn/c2-command-and-control.html) để giám sát điều khiển từ xa các vũ khí mạng đã triển khai. Chúng dùng những kỹ thuật che đậy để giấu đi các dấu vết và chiến thuật DoS ([Denial of Service](https://www.splunk.com/en_us/blog/learn/ddos-attacks.html)) để đánh lạc hướng teams bảo mật khỏi các mục tiêu chính của cuộc tấn công.

### Giai đoạn 7 - Hành động:
- Giai đoạn cuối cùng là thực thi mục tiêu chính của cuộc tấn công, chẳng hạn như:
    - [Phát tán dữ liệu](https://www.splunk.com/en_us/blog/learn/data-exfiltration.html)
    - [Mã hoá](https://www.splunk.com/en_us/blog/learn/data-encryption-methods-types.html) (và có thể là những nỗ lực [đòi tiền chuộc hoặc tống tiền](https://www.splunk.com/en_us/blog/learn/ransomware-trends.html) mục tiêu).
    - Tấn công chuỗi cung ứng ([Supply chain attacks](https://www.splunk.com/en_us/blog/learn/supply-chain-attacks.html))
- Giai đoạn này có thể tốn thời gian tính bằng tuần và tháng, phụ thuộc vào sự thành công của các giai đoạn trước và sự phức tạp của cuộc tấn công.

### Liệu có giai đoạn thứ tám trong Cyber Kill Chain không?
- Một số chuyên gia bảo mật ủng hộ việc bổ sung giai đoạn thứ tám trong Cyber Kill Chains: **Kiếm tiền**. Đây cũng có thể được cân nhắc như là mục tiêu cuối cùng của một cuộc tấn công, nhưng nó tập trung cụ thể vào lợi ích tài chính màn tội phạm mạng thu được từ một cuộc tấn công. Kẻ tấn công có thể bắt đầu với một yêu cầu tống tiền - đòi tiền bằng cách đe doạ phát tán hoặc bán dữ liệu nhạy cảm (thông tin cá nhân hay các bí mật công nghiệp).
- Việc trục lợi từ các cuộc tấn công mạng đang ngày càng trở thành một vấn đề do sự phát triển của việc sử dụng tiền điện tử. Mã hoá giúp kẻ tấn công yêu cầu và nhận tiền dễ dàng và an toàn hơn, điều này góp phần làm gia tăng đáng kể việc kiếm tiền từ các hoạt động tấn công mạng.

## III. Ngăn chặn các cuộc tấn công mạng:
Việc ngăn chặn các cuộc tấn công mạng yêu cầu một phương pháp bảo mật chủ động và đa tầng. Sau đây là các chiến lược giúp nâng cao việc phòng thủ của tổ chức:
- **Các công cụ dò tìm mối đe doạ tiên tiến:** Các công cụ triển khai như hệ thống phát hiện xâm nhập ([Instrusion Detection System - IDS](https://www.splunk.com/en_us/blog/learn/ids-intrusion-detection-systems.html)), hệ thống ngăn chặn xâm nhập ([Instrusion Prevention System - IPS](https://www.splunk.com/en_us/blog/learn/ips-intrusion-prevention-systems.html)), và các giải pháp phát hiện và phản hồi ở endpoint ([Endpoint Detection and Response - EDR](https://www.splunk.com/en_us/blog/learn/endpoint-detection-response-edr.html)). Những công cụ này có thể nhận diện và giảm thiểu mối đe doạ theo thời giam thực, giảm thời gian có thể tấn công của attackers.
- **Thường xuyên kiểm thử các đánh giá & thâm nhập lỗ hổng:** Tổ chức kiểm thử đánh giá và thâm nhập lỗ hổng định kỳ để tìm và fix các điểm yếu bảo mật trước khi attackers có thể khai thác chúng. Điều này nên bao gồm cả quét tự động và kiểm thử thủ công bởi các chuyên gia bảo mật lành nghề.
- **[Quản lý bản vá](https://www.splunk.com/en_us/blog/learn/patch-management.html):** Đảm bảo rằng tất cả phần mềm, bao gồm các hệ thống vận hành, apps, và các công cụ bảo mật, luôn được cập nhật với bản vá và updates mới nhất. Điều này giảm thiểu rủi ro attackers khai thác vào các lỗ hổng đã biết.
- **Phân đoạn mạng:** Thực hiện phân đoạn mạng để giới hạn việc di chuyển của attackers trong mạng. Bằng cách chia mạng thành các phân đoạn nhỏ hơn và riêng biệt, bạn có thể ngăn chặn việc attackers truy cập vào các vùng nhạy cảm.
- **Xác thực đa yếu tố (Multi-factor authentication - MFA):** Bắt buộc xác thực đa yếu tố với tất cả tài khoản người dùng, đặc biệt là những cái có quyền truy cập và thông tin nhạy cảm và hệ thống quan trọng. MFA thêm một lớp bảo mật bổ sung, làm cho attackers khó có thể truy cập trái phép hơn.
- **Các chương trình đào tạo & nâng cao nhận thức cho nhân viên:** Thường xuyên tổ chức các chương trình đào tạo và nâng cao nhận thức cho các nhân viên. Giáo dục cho họ cách nhận diện các nỗ lực phishing, tấn công social engineering, và các biện pháp an toàn khi trực tuyến. Một lực lượng lao động được trang bị đầy đủ kiến thức là một phần quan trọng trong việc phòng thủ chống lại các mối đe doạ về mạng.
- **Kế hoạch ứng phó sự cố:** Phát triển và duy trì một [kế hoạch ứng phó sự cố](https://www.splunk.com/en_us/blog/learn/incident-response-plans.html) toàn diện. Kế hoạch nên triển khai các bước thực hiện khi gặp một sự kiện [xâm nhập mạng](https://www.splunk.com/en_us/blog/learn/security-breach-types.html), bao gồm những vai trò, trách nhiệm, phương thức giao tiếp, các quy trình phục hồi. Thường xuyên kiểm tra và update kết hoạch để bảo đảm tính hiệu quả của nó.
- **Phân tích hành vi:** Sử dụng các phương pháp [phân tích hành vi](https://www.splunk.com/en_us/blog/learn/behavioral-analytics.html) để tìm kiếm những sự bất thường ở hoạt động trong mạng. Bằng cách thiết lập một baseline cho hành vi bình thường, bạn có thể tìm ra các mối đe doạ tiềm tàng mà các phương pháp bảo mật truyền thống có thể bỏ qua.
- **Kiến trúc Zero-Trust:** Áp dụng một [mô hình bảo mật zero-trust](https://www.splunk.com/en_us/blog/learn/zero-trust.html), thứ vận hành dựa trên quy tắc "không bao giờ tin tưởng, luôn luôn xác thực". Hướng tiếp cận này yêu cầu liên tục thực hiện xác thực cho tất cả người dùng, thiết bị, và apps, cả ở bên trong lẫn bên ngoài mạng, trước khi cho phép truy cập.
- **Thường xuyên sao lưu dữ liệu:** Thường xuyên thực hiện việc sao lưu tất cả dữ liệu quan trọng và đảm bảo rằng các hệ thống sao luôn an toàn và được kiểm tra. Trong các sự kiện như tấn công tống tiền hay rò rỉ dữ liệu, việc có các bản sao lưu đáng tin cậy có thể giúp hồi phục nhanh chóng trong thời gian tối thiểu.

## IV. Điểm yếu trong Cyber Kill Chain:
- Mô hình Cyber Kill Chain của Lockheed Martin có thể có những điểm mạnh của nó, nhưng có một số người cho rằng khung bảo mật 2011 đã lỗi thời hay thiếu sự đổi mới. Một điểm yếu quan trọng của mô hình truyền thống này là nó được thiết kế để [tìm kiếm và ngăn chặn malware](https://www.splunk.com/en_us/blog/learn/malware-detection.html) và bảo vệ an ninh mạng vòng ngoài. Chúng là những cột chống kinh điển, cơ bản của an ning mạng. Tuy nhiên, hiện nay chúng ta phải đối mặt với nhiều mối đe doạ an ninh hơn, trên nhiều bề mặt tấn công hơn, và tội phạm mạng đang trở nên càng ngày càng ranh ma hơn.
- Sau đây là những khuyết điểm lớn của mô hình Cyber Kill Chain bảy bước:

### 1. Giới hạn trong profile phát hiện cuộc tấn công:
- Kill Chain bị giới hạn trong khả năng phát hiện nhiều loại tấn công của nó. Thiết kế ban đầu hướng tới việc chỉ ra các malware và payloads, nó không bao quát đủ các mối đe doạ hiện đại như:
    - File-less malware (malware không cần tệp)
    - [Living-off-the-land attacks](https://splunkresearch.com/endpoint/1be30d80-3a39-4df9-9102-64a467b24abc/)
    - Các cuộc tấn công mạng nâng cao như [SQL Injection](https://www.splunk.com/en_us/blog/learn/sql-injection.html), [Cross-site Scripting](https://www.splunk.com/en_us/blog/learn/cross-site-scripting-xss-attacks.html), và các lỗ hổng [zero-day](https://www.splunk.com/en_us/blog/learn/zero-day.html)
- Ngoài ra, nó cũng bỏ qua các cuộc tấn công bởi những bên không được uỷ quyền lợi dụng các thông tin đăng nhập bị xâm pháp.

### 2. Không phát hiện mối đe doạ nội bộ:
![image](https://hackmd.io/_uploads/S1V7XeGwzg.png)
Mô hình Cyber Kill Chain truyền thống không lý giải cho các mối đe doạ nội bộ, điều mà tiềm ẩn mối đe doạ đáng kể đối với các tổ chức. Những mối đe doạ nội có thể bao gồm các nhân viên hay nhà thầu với quyền truy cập hợp lệ nhưng lại dụng các quyền của mình. Việc tìm kiếm hiệu quả yêu cầu sự giám sát hành vi và hoạt động người dùng và trong mạng và apps, thường thông qua các hệ thống tự động thiết lập cảnh báo cho những hoạt động đáng ngờ.

### 3. Thiếu sự linh hoạt:
- Không phải tất cả attackers đều thực hiện tuyến tính theo Cyber Kill Chain. Chúng có thể bỏ qua, kết hợp, hay quay lại các giai đoạn. Ví dụ, một số cuộc tấn công có thể không trinh sát rộng rãi, thay vào đó sử dụng chiến lược "spray and pray" (gửi cho tất cả nạn nhân tiềm năng). Một báo cáo tiết lộ rằng có gần [90% cuộc tấn công kết hợp năm giai đoạn đầu của Cyber Kill Chain vào một hành động duy nhất](https://www.alertlogic.com/press-releases/alert-logic-releases-state-of-threat-detection-2018-report/).
- Cấu trúc cứng nhắc của Cyber Kill Chain truyền thống có thể khiến các tổ chức bỏ qua hoặc không phản ứng đầy đủ với những mối đe doạ không giống với mẫu có sẵn.

### 4. Các công nghệ chuyển đổi:
Sự tiên tiến trong công nghệ - điện toán đám mây, DevOps, IoT, machine learning, và tự động hoá - đã mở rộng phạm vi của các cuộc tấn công mạng. Những sự tiến bộ này tạo ra nhiều điểm xâm nhập và nguồn dữ liệu mới, là thử thách để điều chỉnh các khung kill chain. Hơn nữa, việc mô hình làm việc từ xa tăng lên và sự phổ biến của tiền điện tử đã tạo ra nhiều cơ hội để khai thác cho tội phạm mạng.

### 5. Các mối đe doạ và kỹ thuật mới:
- Tội phạm mạng liên tục phát triển nhiều phương thức vượt qua khả năng của kill chain ban đầu. Ví dụ, các công nghệ như lừa đảo khuôn mặt (deepfake phishing), những cuộc tấn công do AI điều khiển, và các chiến dịch tống tiền bằng mã độc tinh vi yêu cầu nhiều mô hình bảo mật chủ động và phản hồi nhanh hơn. Cyber Kill Chain truyền thống không cung cấp hướng dẫn đầy đủ để giảm thiểu các mối đe doạ nâng cao này.
- Việc chỉ ra các điểm yếu này yêu cầu sự tích hợp các khung bảo mật toàn diện và linh hoạt hơn, như MITRE ATT&CK và Cyber COBRA, những khung đưa ra các hướng tiếp cận chi tiết và thích ứng hơn với những mối đe doạ về mạng. Các tổ chức phải liên tục phát triển các chiến lược bảo mật của họ để đi trước attackers và bảo vệ tài sản kỹ thuật số của mình một cách hiệu quả.

## V. Làm thế nào Cyber Kill Chain có thể cải thiện sự bảo mật:
- Mặc dù bảy giai đoạn ban đầu của Cyber Kill Chain đã được xem xét kỹ lưỡng, các tổ chức vẫn có thể dùng những nguyên tắc này để chuẩn bị tốt hơn cho các cuộc tấn công mạng ở hiện tại và tương lai. Một khuôn mẫu Cyber Kill Chain có thể hướng dẫn cho chiến lược bảo mật mạng của một doanh nghiệp, cho dù đó là bằng cách xác định lỗi với chiến lược hiện tại hay xác nhận xem những gì đang hoạt động tốt. Ví dụ, nó có thể khuyến khích việc sử dụng các dịch vụ và giải pháp như:
    - Phần mềm bảo vệ điểm cuối
    - VPNs
    - Đào tạo nhân viên
- Việc bao gồm Cyber COBRA trong khuôn mẫu này có thể thêm vào một lớp đánh giá và ưu tiên bổ sung, đảm bảo rằng những lỗ hổng bảo mật quan trọng nhất được khắc phục kịp thời. Trong bối cảnh những cuộc tấn công mạng đang tiếp tục gia tăng, các tổ chức phải cân nhắc [chiến lược kết hợp nhiều lớp](https://www.sentinelone.com/cybersecurity-101/cyber-kill-chain/) biện pháp quản trị, kỹ thuật, và vật lý để bảo mật. Phương pháp Cyber Kill Chain có thể giúp các tổ chúc đạt được điều này, nhưng mô hình ban đầu chỉ có thể áp dụng đến một mức độ nhất định.

## VI. Những sự thay thế cho Cyber Kill Chain gốc:
Trong khi tất cả các doanh nghiệp đều yêu cầu một khuôn mẫu Cyber Kill Chain được thiết kế riêng, đây là những các khác để điều chỉnh quy trình chuỗi tấn công ban đầu:

### 1. Thống nhất Kill Chain (Unified Kill Chain):
- Concept của một [unified kill chain](https://thecyphere.com/blog/cyber-kill-chain/) kết hợp các kỹ thuật từ MITRE ATT&CK và mô hình Cyber Kill Chain ban đầu. Kết quả của việc kết hợp này là một khuôn mẫu tích hợp chi tiết bao gồm 18 giai đoạn cá nhân, có thể được gộp thành [ba giai đoạn chính](https://www.sentinelone.com/cybersecurity-101/cyber-kill-chain/):
    - Chỗ đứng ban đầu
    - Mở rộng mạng lưới
    - Hành động lên đối tượng
- Hướng tiếp cận này cho phép các teams so sánh mô phỏng các chỉ số xâm phạm ([indicators of compromise - IOCs](https://www.splunk.com/en_us/blog/learn/ioc-indicators-of-compromise.html)) với các nguồn thông tin tình báo về mối đe doạ để phản ứng với những mối đe doạ một cách hiệu quả. Một mô hình unified kill chain ATT&CK có thể được sử dụng bởi các teams phòng thủ và tấn công để phát triển các [biện pháp kiểm soát bảo mật](https://www.splunk.com/en_us/blog/learn/cis-critical-security-controls.html).

### 2. Mô phỏng Cyber Kill Chains:
- Các tổ chức cũng có thể dùng các mô hình kill chain để mô phỏng tấn công mạng, với số lượng lớn các nền tảng chuyên dụng có sẵn để mô phỏng quá trình của Cyber Kill Chain. Điều này cho phép bạn định vị và sửa đổi bất kỳ điểm xâm nhập hay lỗ hổng bảo mật hệ thống nào trong thời gian rất ngắn.
- Cũng như mô phỏng các mối đe doạ qua email, web, và firewall gateways, những nền tảng này có thể cung cấp bạn điểm/báo cáo rủi ro của các thực thể hệ thống để giúp teams xác định được các khu vực rủi ro chính. Sau đó, tổ chức có thể hành động và ngăn chặn các mối đe doạ tương lai với những biện pháp như thay đổi cấu hình và cài đặt các bản vá.

## VII. Đừng vội phá huỷ Cyber Kill Chain:
- Sự tiến hoá liên tục của các cuộc tấn công mạng đã đặt ra nhiều câu hỏi về tương lai của Cyber Kill Chain. Kết quả là, một kill chain nhanh kết hợp các yếu tố của MITRE ATT&CK và các chiến lược phát hiện và phản ứng mở rộng ([extended detection and reponse - XDR](https://www.splunk.com/en_us/blog/learn/xdr-extended-detection-response.html)) có thể xác định được nhiều mối đe doạ hơn và có thể ngăn chặn và vô hiệu hoá chúng một cách hiệu quả.
- Dù cho quan điểm của bạn về khuôn mẫu Cyber Kill Chain là gì, việc giải quyết các lỗ hổng bảo mật đang tồn tại và áp dụng [chiến lược an ninh mạng](https://www.splunk.com/en_us/blog/learn/cybersecurity.html#creating-cybersecurity-strategy) toàn diện là rất mật thiết cho sự an toàn của bất kỳ doanh nghiệp nào.
