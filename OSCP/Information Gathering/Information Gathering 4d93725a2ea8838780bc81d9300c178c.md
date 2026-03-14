# Information Gathering

# Thu thập thông tin

---

Mục tiêu của kiểm thử xâm nhập (pentest) là phát hiện các lỗ hổng an ninh để cải thiện khả năng phòng thủ của doanh nghiệp được kiểm thử. Do mạng, thiết bị và phần mềm trong môi trường của doanh nghiệp luôn thay đổi theo thời gian, kiểm thử xâm nhập là một hoạt động mang tính chu kỳ. “Bề mặt tấn công” của doanh nghiệp sẽ biến động định kỳ vì các lỗ hổng phần mềm mới được phát hiện, sai sót cấu hình phát sinh từ hoạt động nội bộ, hoặc việc tái cấu trúc hạ tầng CNTT có thể làm lộ ra các phân đoạn mới để trở thành mục tiêu.

Trong Mô-đun Học này, chúng ta sẽ học cách lập bản đồ bề mặt tấn công một cách có phương pháp bằng cả phương pháp thụ động và chủ động, cũng như hiểu cách tận dụng thông tin này trong suốt vòng đời của một bài pentest.

---

# 1. Vòng đời Kiểm thử Xâm nhập

---

Đơn vị Học này bao gồm các Mục tiêu Học tập sau:

- Hiểu các giai đoạn của một bài Kiểm thử Xâm nhập
- Tìm hiểu vai trò của Thu thập Thông tin trong từng giai đoạn
- Hiểu sự khác biệt giữa Thu thập Thông tin Chủ động và Thụ động

Để giữ tư thế an ninh của doanh nghiệp được kiểm soát chặt chẽ nhất có thể, chúng ta nên tiến hành kiểm thử xâm nhập theo chu kỳ định kỳ và sau mỗi lần có sự thay đổi đáng kể trong kiến trúc CNTT của mục tiêu.

Một bài kiểm thử xâm nhập điển hình bao gồm các giai đoạn sau:

- Xác định Phạm vi
- Thu thập Thông tin
- Phát hiện Lỗ hổng
- Thiết lập Chân Trận ban đầu (Initial Foothold)
- Tâng quyền (Privilege Escalation)
- Di chuyển Ngang (Lateral Movement)
- Báo cáo / Phân tích
- Rút kinh nghiệm / Khắc phục (Lessons Learned / Remediation)

Trong Mô-đun này, chúng ta sẽ giới thiệu ngắn gọn về việc xác định phạm vi trước khi chuyển trọng tâm sang mục tiêu chính - Thu thập Thông tin. Chúng ta sẽ học thêm về các giai đoạn khác trong phần còn lại của khóa học.

Phạm vi của một hợp đồng kiểm thử xâm nhập xác định những dải IP, máy chủ và ứng dụng nào sẽ là đối tượng kiểm thử trong thời gian cam kết, so với những mục không thuộc phạm vi (out-of-scope) không được phép kiểm thử.

Khi đã thống nhất với khách hàng về phạm vi và khung thời gian của hợp đồng, chúng ta có thể tiến tới bước thứ hai: thu thập thông tin. Ở bước này, mục tiêu là thu thập càng nhiều dữ liệu về mục tiêu càng tốt.

Để bắt đầu thu thập thông tin, chúng ta thường thực hiện thu thập tình báo (reconnaissance) để lấy chi tiết về hạ tầng, tài sản và nhân sự của tổ chức mục tiêu. Việc này có thể thực hiện theo cách thụ động hoặc chủ động. Kỹ thuật thụ động cố gắng thu thập thông tin mà hầu như không tương tác trực tiếp với mục tiêu, trong khi kỹ thuật chủ động thăm dò hạ tầng một cách trực tiếp. Thu thập thông tin chủ động để lộ diện nhiều hơn, vì vậy người ta thường ưu tiên thu thập thụ động để tránh bị phát hiện.

Cần lưu ý rằng thu thập thông tin (còn gọi là enumeration) không kết thúc sau khi hoàn tất reconnaissance ban đầu. Chúng ta cần tiếp tục thu thập dữ liệu khi bài pentest tiến triển, xây dựng hiểu biết về bề mặt tấn công của mục tiêu khi phát hiện thêm thông tin qua việc có được chân trạn hoặc di chuyển ngang.

Trong Mô-đun này, trước tiên chúng ta sẽ tìm hiểu reconnaissance thụ động, sau đó khám phá cách tương tác chủ động với mục tiêu cho mục đích enumeration.

---

# 2. Thu thập Thông tin Thụ động

---

Đơn vị Học này bao gồm các Mục tiêu Học tập sau:

- Hiểu hai cách tiếp cận khác nhau trong Thu thập Thông tin Thụ động
- Tìm hiểu về Tình báo Mã nguồn mở (OSINT)
- Hiểu thu thập thông tin thụ động liên quan đến Máy chủ Web và DNS

Thu thập Thông tin Thụ động, hay còn gọi là Tình báo Mã nguồn mở (OSINT), là quá trình thu thập thông tin có sẵn công khai về một mục tiêu, thường là không tương tác trực tiếp với mục tiêu đó.

Trước khi bắt đầu, ta cần xem xét hai trường phái tư duy khác nhau về việc thế nào được coi là “thụ động” trong bối cảnh này.

- Trong cách hiểu nghiêm ngặt nhất, ta không bao giờ giao tiếp trực tiếp với mục tiêu. Ví dụ: ta có thể dựa vào bên thứ ba để lấy thông tin, nhưng không truy cập bất kỳ hệ thống hay máy chủ nào của mục tiêu. Cách tiếp cận này giữ bí mật rất cao về hành động và ý định của ta, nhưng có thể rườm rà và giới hạn kết quả đạt được.
- Trong cách hiểu lỏng hơn, ta có thể tương tác với mục tiêu nhưng chỉ như một người dùng internet bình thường. Ví dụ: nếu website của mục tiêu cho phép đăng ký tài khoản, ta có thể đăng ký. Tuy nhiên, ở giai đoạn này ta sẽ không thử nghiệm website để tìm lỗ hổng.

Cả hai cách tiếp cận đều có ích, tùy thuộc vào mục tiêu của bài kiểm thử. Vì lý do đó, ta cần cân nhắc phạm vi và các quy tắc tham gia (rules of engagement) của bài pentest trước khi quyết định sử dụng cách nào.

Trong Mô-đun này, chúng ta sẽ áp dụng cách hiểu ít nghiêm ngặt hơn (cách lỏng hơn) cho phương pháp của mình.

Có nhiều nguồn lực và công cụ mà ta có thể dùng để thu thập thông tin, và quy trình mang tính chu kỳ hơn là tuyến tính. Nói cách khác, “bước tiếp theo” của bất kỳ giai đoạn nào trong quy trình phụ thuộc vào những gì ta tìm thấy ở các bước trước đó, tạo thành các “vòng lặp” quy trình. Vì mỗi công cụ hoặc nguồn có thể tạo ra rất nhiều kết quả khác nhau, sẽ khó để định nghĩa một quy trình chuẩn hóa. Mục tiêu cuối cùng của thu thập thông tin thụ động là thu được dữ liệu làm rõ hoặc mở rộng bề mặt tấn công, giúp ta thực hiện chiến dịch lừa đảo (phishing) thành công, hoặc bổ trợ cho các bước pentest khác như đoán mật khẩu — tất cả có thể dẫn tới việc chiếm đoạt tài khoản.

Thay vì trình bày các kịch bản liên kết, chúng ta sẽ chỉ bao quát các nguồn lực và công cụ khác nhau, giải thích cách chúng hoạt động, và trang bị cho bạn các kỹ thuật cơ bản cần thiết để xây dựng một chiến dịch thu thập thông tin thụ động.

Trước khi bắt đầu bàn về nguồn lực và công cụ, hãy chia sẻ một ví dụ thực tế từ một bài kiểm thử đã thành công nhờ chiến dịch thu thập thông tin thụ động.

> Lưu ý từ các Tác giả
> 

Vài năm trước, nhóm OffSec được giao thực hiện một bài pentest cho một công ty nhỏ. Công ty này hầu như không có hiện diện trên Internet và rất ít dịch vụ phơi bày ra bên ngoài; tất cả những dịch vụ đó đều được bảo mật tốt. Về cơ bản, không có bề mặt tấn công nào đáng kể để khai thác. Sau một chiến dịch thu thập thông tin thụ động tập trung, tận dụng các toán tử tìm kiếm Google, kết nối các mẩu thông tin “được pipe” vào các công cụ trực tuyến khác, cùng một chút tư duy sáng tạo và logic, chúng tôi tìm thấy một bài đăng trên diễn đàn do một nhân viên mục tiêu đăng trong một diễn đàn sưu tập tem:

```bash
Hi!
I'm looking for rare stamps from the 1950's - for sale or trade.
Please contact me at david@company-address.com
Cell: 999-999-9999
```

                                               *Bài đăng 1 - Một ví dụ mồi nhử trên diễn đàn*

Chúng tôi dùng thông tin này để phát động một cuộc tấn công phía khách hàng (client-side) bán tinh vi. Nhanh chóng chúng tôi đăng ký một tên miền liên quan đến tem và thiết kế một trang đích trưng bày các con tem quý từ thập niên 1950 — những mẫu tem này chúng tôi tìm thấy qua Google Images. Tên miền và thiết kế trang rõ ràng làm tăng độ tin cậy được cảm nhận của website giao dịch tem.

Tiếp theo, chúng tôi nhúng mã khai thác phía client độc hại vào các trang web và gọi cho “David” vào giờ làm việc. Trong cuộc gọi, chúng tôi đóng vai một người sưu tập tem thừa hưởng bộ sưu tập lớn từ ông nội.

David phấn khởi và truy cập trang web độc hại để xem bộ sưu tập mà không chút do dự. Khi duyệt trang, mã khai thác thực thi trên máy của anh ta và gửi về cho chúng tôi một reverse shell.

Đây là ví dụ điển hình cho việc một vài thông tin thu thập thụ động tưởng chừng vô hại — như một nhân viên dùng email công ty cho việc cá nhân — có thể dẫn tới một chân trạn trong bài pentest. Đôi khi những chi tiết nhỏ nhất lại quan trọng nhất.

*Mặc dù “David” không tuân theo các thực hành tốt, nhưng chính chính sách của công ty và việc thiếu một chương trình nâng cao nhận thức về an ninh đã tạo điều kiện cho lỗ hổng này. Vì lý do đó, khi báo cáo bằng văn bản, chúng tôi tránh đổ lỗi cho một cá nhân. Mục tiêu của chúng ta, với tư cách là pentester, là cải thiện an ninh của tài nguyên khách hàng, chứ không nhắm vào một nhân viên. Chỉ loại bỏ “David” không giải quyết được vấn đề.*

Hãy cùng xem lại một số công cụ và kỹ thuật phổ biến có thể giúp ta tiến hành một chiến dịch thu thập thông tin hiệu quả. Chúng ta sẽ sử dụng MegaCorp One, một công ty hư cấu do OffSec tạo ra, làm đối tượng trong chiến dịch này.

---

## 2.1. Triển khai Whois

---

Whois là một dịch vụ TCP, một công cụ và một loại cơ sở dữ liệu có thể cung cấp thông tin về một tên miền, chẳng hạn như name server và registrar. Thông tin này thường công khai, vì các nhà đăng ký (registrars) thu phí cho dịch vụ đăng ký ẩn danh (private registration).

Chúng ta có thể thu thập thông tin cơ bản về một tên miền bằng cách thực hiện truy vấn forward thông thường và đưa tên miền `megacorpone.com` vào lệnh whois, đồng thời truyền địa chỉ IP của máy chủ WHOIS Ubuntu của ta làm đối số cho tham số `-h` (host).

```bash
kali@kali:~$ whois megacorpone.com -h 192.168.50.251
   Domain Name: MEGACORPONE.COM
   Registry Domain ID: 1775445745_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.gandi.net
   Registrar URL: http://www.gandi.net
   Updated Date: 2019-01-01T09:45:03Z
   Creation Date: 2013-01-22T23:01:00Z
   Registry Expiry Date: 2023-01-22T23:01:00Z
...
Registry Registrant ID:
Registrant Name: Alan Grofield
Registrant Organization: MegaCorpOne
Registrant Street: 2 Old Mill St
Registrant City: Rachel
Registrant State/Province: Nevada
Registrant Postal Code: 89001
Registrant Country: US
Registrant Phone: +1.9038836342
...
Registry Admin ID:
Admin Name: Alan Grofield
Admin Organization: MegaCorpOne
Admin Street: 2 Old Mill St
Admin City: Rachel
Admin State/Province: Nevada
Admin Postal Code: 89001
Admin Country: US
Admin Phone: +1.9038836342
...
Registry Tech ID:
Tech Name: Alan Grofield
Tech Organization: MegaCorpOne
Tech Street: 2 Old Mill St
Tech City: Rachel
Tech State/Province: Nevada
Tech Postal Code: 89001
Tech Country: US
Tech Phone: +1.9038836342
...
Name Server: NS1.MEGACORPONE.COM
Name Server: NS2.MEGACORPONE.COM
Name Server: NS3.MEGACORPONE.COM
...
```

Không phải mọi dữ liệu trong kết quả này đều hữu ích, nhưng chúng ta đã tìm được một số thông tin giá trị. Đầu tiên, đầu ra cho thấy tên miền được đăng ký bởi Alan Grofield. Theo trang Liên hệ của MegaCorp One, Alan là “Giám đốc CNTT và An ninh” (IT and Security Director).

Chúng ta cũng tìm thấy các name server của MegaCorp One. Name server là một thành phần của DNS mà chúng ta sẽ không đi sâu ngay bây giờ, nhưng vẫn nên ghi lại những server này vào ghi chú.

Giả sử ta có một địa chỉ IP, ta cũng có thể dùng client whois để thực hiện truy vấn ngược (reverse lookup) và thu thập thêm thông tin.

```bash
kali@kali:~$ whois 38.100.193.70 -h 192.168.50.251
...
NetRange:       38.0.0.0 - 38.255.255.255
CIDR:           38.0.0.0/8
NetName:        COGENT-A
...
OrgName:        PSINet, Inc.
OrgId:          PSI
Address:        2450 N Street NW
City:           Washington
StateProv:      DC
PostalCode:     20037
Country:        US
RegDate:
Updated:        2015-06-04
...
```

Kết quả của truy vấn ngược cung cấp cho ta thông tin về tổ chức đang host địa chỉ IP đó. Thông tin này có thể hữu ích sau này, và như mọi thông tin chúng ta thu thập được, cần ghi lại vào sổ tay/ghi chú.

---

## 2.2. Google Hacking

---

Thuật ngữ **"Google Hacking"** được phổ biến bởi Johnny Long vào năm 2001. Qua nhiều bài thuyết trình và một cuốn sách rất nổi tiếng (*Google Hacking for Penetration Testers*), ông trình bày cách các công cụ tìm kiếm như Google có thể được dùng để khám phá thông tin nhạy cảm, lỗ hổng và các website cấu hình sai.

Cốt lõi của kỹ thuật này là sử dụng các chuỗi tìm kiếm thông minh và các toán tử để tinh chỉnh truy vấn — hầu hết các toán tử này làm việc tốt trên nhiều công cụ tìm kiếm khác nhau. Quá trình mang tính lặp (iterative): bắt đầu với một truy vấn rộng, sau đó thu hẹp bằng các toán tử để loại bỏ kết quả không liên quan hoặc ít thú vị.

Ta sẽ bắt đầu bằng cách giới thiệu một số toán tử để học cách sử dụng chúng.

`site:` — toán tử giới hạn tìm kiếm trong một tên miền duy nhất. Ta có thể dùng toán tử này để có cái nhìn sơ bộ về hiện diện web của một tổ chức.

![image.png](image.png)

                                                            *Hình 1: Tìm kiếm với toán tử `site:`*

(Hình minh họa: toán tử `site:` giới hạn kết quả trong domain `megacorpone.com`.)

Tiếp theo, ta có thể dùng các toán tử khác để thu hẹp hơn nữa. Ví dụ, toán tử `filetype:` (hoặc `ext:`) giới hạn kết quả chỉ về loại tập tin được chỉ định.

Trong ví dụ dưới đây, ta kết hợp các toán tử để tìm file TXT (`filetype:txt`) trên `www.megacorpone.com` (`site:megacorpone.com`):

![image.png](image%201.png)

                                                          *Hình 2: Tìm kiếm với toán tử `filetype:`*

(Kết quả thú vị: truy vấn tìm thấy file `robots.txt`, có nội dung sau.)

Nội dung `robots.txt` hướng dẫn các web crawler (ví dụ: crawler của Google) cho phép hoặc cấm các tài nguyên cụ thể. 

```bash
User-agent: *
Allow: /
Allow: /nanites.php
```

Trong trường hợp này nó tiết lộ một trang PHP cụ thể (`/nanites.php`) mà bình thường bị ẩn khỏi tìm kiếm thông thường, mặc dù được cho phép theo chính sách.

Toán tử `ext:` cũng hữu ích để nhận biết ngôn ngữ lập trình được sử dụng trên một trang web. Các truy vấn như `ext:php`, `ext:xml`, `ext:py` sẽ tìm các trang PHP, XML và Python đã được lập chỉ mục tương ứng.

Ta cũng có thể loại bỏ (exclude) những mục không mong muốn bằng dấu `-` trước toán tử để thu hẹp kết quả.

Ví dụ, để tìm các trang không phải HTML hấp dẫn, ta có thể dùng `site:megacorpone.com` để giới hạn trong domain và các subdomain, sau đó thêm `-filetype:html` để loại bỏ trang HTML khỏi kết quả.

![image.png](image%202.png)

                                                        *Hình 3: Tìm kiếm với toán tử loại bỏ (`-`)*

(Trong ví dụ, ta tìm thấy một số trang thú vị, bao gồm các trang liệt kê thư mục — directory indices.)

Trong một ví dụ khác, ta có thể tìm những trang liệt kê thư mục bằng truy vấn:

`intitle:"index of" "parent directory"`

Truy vấn này tìm những trang có `"index of"` trong thẻ tiêu đề (title) và chứa cụm từ `"parent directory"` trên trang.

![image.png](image%203.png)

                                               *Hình 4: Dùng Google để tìm Directory Listings*

(Kết quả thường là các trang liệt kê nội dung thư mục khi không có trang index — cấu hình sai kiểu này có thể lộ các file nhạy cảm.)

Những ví dụ cơ bản trên chỉ là phần nổi của tảng băng. **Google Hacking Database (GHDB)** chứa rất nhiều truy vấn sáng tạo minh họa sức mạnh của việc kết hợp các toán tử.

![image.png](image%204.png)

                                                   *Hình 5: Google Hacking Database (GHDB)*

Một cách khác để thử nghiệm Google Dorks là dùng **DorkSearch portal**, cung cấp tập các truy vấn đã dựng sẵn và công cụ builder giúp tạo truy vấn nhanh.

Thành thục các toán tử này, kết hợp với khả năng suy luận tốt, là kỹ năng then chốt để thực hiện "hack" bằng công cụ tìm kiếm hiệu quả.

---

## 2.3. Netcraft

---

Netcraft là một công ty dịch vụ internet có trụ sở tại Anh, cung cấp một cổng web miễn phí thực hiện nhiều chức năng thu thập thông tin, chẳng hạn như phát hiện công nghệ nào đang chạy trên một website nhất định và tìm những host khác chia sẻ cùng IP netblock.

Sử dụng các dịch vụ như Netcraft được coi là kỹ thuật thụ động, vì ta không bao giờ tương tác trực tiếp với mục tiêu.

Hãy xem qua một số khả năng của Netcraft. Ví dụ, ta có thể dùng trang tìm kiếm DNS của Netcraft để thu thập thông tin về tên miền `megacorpone.com`:

![image.png](image%205.png)

                                   *Hình 6: Kết quả Netcraft cho tìm kiếm `*.megacorpone.com`*

Với mỗi server được tìm ra, ta có thể xem một “báo cáo site” cung cấp thông tin bổ sung và lịch sử về server đó bằng cách bấm vào biểu tượng file bên cạnh mỗi URL site.

![image.png](image%206.png)

                                      *Hình 7: Báo cáo Site Netcraft cho `www.megacorpone.com`*

Phần bắt đầu của báo cáo đề cập đến thông tin đăng ký. Tuy nhiên, nếu ta cuộn xuống, sẽ thấy nhiều mục “site technology” (công nghệ site).

![image.png](image%207.png)

                                         *Hình 8: Công nghệ Site cho `www.megacorpone.com`*

Danh sách các subdomain và công nghệ này sẽ hữu ích khi ta chuyển sang thu thập thông tin chủ động và khai thác. Hiện tại, ta chỉ ghi lại chúng vào ghi chú.

Gần đây, Netcraft đã quyết định ngưng một phần dịch vụ này. Thông tin để trả lời các câu hỏi nằm trong các hình ảnh, hoặc bạn có thể truy cập Wappalyzer (ví dụ: `https://www.wappalyzer.com/lookup/megacorpone.com/`) để tìm các câu trả lời trên một trang trực tiếp.

---

## 2.4. Mã Nguồn Mở

---

Trong các phần sau, chúng ta sẽ khám phá nhiều công cụ và nguồn trực tuyến có thể dùng để thu thập thông tin thụ động. Điều này bao gồm các dự án mã nguồn mở và kho lưu trữ mã trực tuyến như:

- GitHub
- GitHub Gist
- GitLab
- SourceForge

Mã nguồn được lưu trữ trực tuyến có thể cho ta cái nhìn về ngôn ngữ lập trình và các framework mà một tổ chức đang sử dụng. Trong một vài trường hợp hiếm hoi, các nhà phát triển đã vô tình cam kết (commit) dữ liệu nhạy cảm và thông tin xác thực lên các repo công khai.

Các công cụ tìm kiếm trên một số nền tảng này hỗ trợ các toán tử tìm kiếm của Google mà chúng ta đã thảo luận trong Mô-đun này.

Ví dụ, công cụ tìm kiếm của GitHub khá linh hoạt. Ta có thể dùng GitHub để tìm kiếm repo của một user hoặc tổ chức; tuy nhiên, ta cần một tài khoản nếu muốn tìm kiếm trên tất cả các repo công khai.

Để thực hiện bất kỳ tìm kiếm GitHub nào, trước tiên ta cần đăng ký một tài khoản cơ bản trên GitHub — tài khoản này miễn phí cho cá nhân và tổ chức.

Khi đã đăng nhập vào tài khoản GitHub, ta có thể thực hiện nhiều tìm kiếm theo từ khóa bằng cách gõ vào ô tìm kiếm ở góc trên bên phải.

![image.png](image%208.png)

                                                                        *Hình 9: GitHub Search*

Hãy tìm trong các repo của MegaCorp One những thông tin thú vị. Ta có thể dùng `path:users` để tìm các file có từ "users" trong tên file và nhấn ENTER.

![image.png](image%209.png)

                                                          *Hình 9: File Operator in GitHub Search*

Lần tìm này chỉ tìm thấy một file — `xampp.users`. Dù vậy file này vẫn đáng chú ý vì XAMPP là một môi trường phát triển web. Hãy kiểm tra nội dung file.

![image.png](image%2010.png)

                                                               *Hình 10: xampp.users File Content*

File này có vẻ chứa tên người dùng và hash mật khẩu, thứ có thể rất hữu ích khi ta bắt đầu giai đoạn tấn công chủ động. Ghi lại thông tin này vào ghi chú.

Cách làm thủ công này sẽ phù hợp nhất với các repo nhỏ. Với repo lớn hơn, ta có thể dùng một số công cụ để tự động hóa một phần quá trình tìm kiếm, ví dụ Gitrob và Gitleaks. Hầu hết các công cụ này yêu cầu token truy cập để sử dụng API của nhà cung cấp dịch vụ lưu trữ mã nguồn.

Ảnh chụp màn hình sau đây là ví dụ Gitleaks tìm thấy một AWS access key ID trong một file.

![image.png](image%2011.png)

                                                                *Hình 11: Example Gitleaks Output*

Có được các thông tin xác thực này có thể cho phép truy cập không giới hạn vào cùng một tài khoản AWS và có thể dẫn tới việc xâm phạm bất kỳ dịch vụ đám mây nào do identity đó quản lý.

*Các công cụ tìm kiếm secrets trong mã nguồn, như Gitrob hay Gitleaks, thường dựa vào biểu thức chính quy hoặc phát hiện dựa trên entropy để nhận diện thông tin có khả năng hữu ích. Phát hiện dựa trên entropy cố gắng tìm chuỗi có tính ngẫu nhiên cao. Ý tưởng là một chuỗi dài các ký tự ngẫu nhiên rất có khả năng là mật khẩu. Dù dùng phương pháp nào, không có công cụ nào hoàn hảo — chúng có thể bỏ sót những gì mà một kiểm tra thủ công có thể tìm ra.*

---

## 2.5. Shodan

---

Khi ta thu thập thông tin về mục tiêu, cần nhớ rằng các website truyền thống chỉ là một phần của Internet.

**Shodan** là một công cụ tìm kiếm (search engine) quét các thiết bị kết nối Internet, bao gồm các server chạy website, nhưng cũng có cả các thiết bị như router và thiết bị IoT.

Nói cách khác, Google và các công cụ tìm kiếm khác tìm kiếm nội dung trên web server, trong khi Shodan tìm kiếm các thiết bị kết nối Internet, tương tác với chúng và hiển thị thông tin về chúng.

Mặc dù Shodan không bắt buộc để hoàn thành nội dung trong Mô-đun hoặc phòng lab này, nhưng rất đáng để khám phá một chút. Trước khi dùng Shodan, ta phải đăng ký một tài khoản miễn phí, cho phép truy cập giới hạn.

Bắt đầu bằng cách dùng Shodan để tìm kiếm `hostname:megacorpone.com`.

![image.png](image%2012.png)

                                          *Hình 12: Tìm kiếm domain MegaCorp One với Shodan*

Trong trường hợp này, Shodan liệt kê các địa chỉ IP, dịch vụ và thông tin banner. Tất cả những dữ liệu này được thu thập một cách thụ động, tránh việc tương tác trực tiếp với website của khách hàng.

Thông tin này cung cấp cho ta một ảnh chụp (snapshot) về dấu chân Internet của mục tiêu. Ví dụ, có bốn máy chủ đang chạy SSH. Ta có thể khoanh vùng kết quả bằng cách bấm vào mục SSH dưới phần Top Ports ở pane bên trái.

![image.png](image%2013.png)

                                              *Hình 13: Các server MegaCorp One đang chạy SSH*

Dựa trên kết quả của Shodan, ta biết chính xác phiên bản OpenSSH nào đang chạy trên từng server. Nếu bấm vào một địa chỉ IP, ta có thể lấy bản tóm tắt về host đó.

![image.png](image%2014.png)

                                                             *Hình 14: Tóm tắt host trên Shodan*

Trên trang này ta có thể xem lại các cổng, dịch vụ và công nghệ đang được server sử dụng. Shodan cũng sẽ cho biết nếu có bất kỳ lỗ hổng công bố nào cho các dịch vụ hoặc công nghệ đã xác định trên cùng host. Thông tin này rất quan trọng khi quyết định điểm khởi đầu cho việc kiểm thử chủ động.

---

## 2.6. Security Headers và SSL/TLS

---

Có một số trang web chuyên dụng khác mà ta có thể dùng để thu thập thông tin về tư thế an ninh của một website hoặc một domain. Một vài trong số những trang này làm mờ ranh giới giữa thu thập thụ động và chủ động, nhưng điểm then chốt với mục đích của chúng ta là một bên thứ ba (third-party) khởi xướng các quét hoặc kiểm tra đó — tức là ta không trực tiếp quét mục tiêu.

Một trang như **Security Headers** sẽ phân tích các HTTP response header và cung cấp phân tích cơ bản về tư thế an ninh của site mục tiêu. Ta có thể dùng nó để có một nhận định sơ bộ về thói quen mã hoá và thực hành an ninh của tổ chức dựa trên kết quả trả về.

Hãy quét `www.megacorpone.com` và xem kết quả.

![image.png](image%2015.png)

                                            *Hình 15: Kết quả quét cho `www.megacorpone.com`*

Trang web thiếu một số header phòng thủ quan trọng, chẳng hạn `Content-Security-Policy` và `X-Frame-Options`. Việc thiếu các header này không nhất thiết là một lỗ hổng trực tiếp, nhưng nó có thể gợi ý rằng các nhà phát triển web hoặc quản trị viên server chưa nắm rõ các thực hành hardening cho server.

***Server hardening** là quá trình tổng thể để bảo mật một server thông qua cấu hình — bao gồm các việc như vô hiệu hoá các dịch vụ không cần thiết, xoá các dịch vụ hoặc tài khoản người dùng không dùng tới, thay đổi mật khẩu mặc định, đặt các header server thích hợp, v.v. Ta không cần biết tường tận cách cấu hình mọi loại server, nhưng hiểu các khái niệm cơ bản và biết nên tìm gì sẽ giúp ta định hướng cách tiếp cận một mục tiêu tiềm năng.*

Một công cụ quét khác ta có thể dùng là **SSL Server Test** của Qualys SSL Labs. Công cụ này phân tích cấu hình SSL/TLS của một server và so sánh với các thực hành tốt nhất hiện hành. Nó cũng sẽ chỉ ra một số lỗ hổng liên quan tới SSL/TLS, chẳng hạn **Poodle** hoặc **Heartbleed**. Hãy quét `www.megacorpone.com` và xem kết quả.

![image.png](image%2016.png)

                                     *Hình 16: Kết quả SSL Server Test cho `www.megacorpone.com`*

Kết quả có vẻ tốt hơn so với kiểm tra Security Headers. Tuy nhiên, báo cáo cho thấy server vẫn hỗ trợ các phiên bản TLS như **1.0** và **1.1**, vốn được coi là đã lỗi thời vì chúng cho phép một số [cipher suite](https://www.notion.so/) không an toàn — điều này gợi ý mục tiêu chưa áp dụng các thực hành hardening SSL/TLS hiện hành. Ví dụ, vô hiệu hoá bộ mã `TLS_DHE_RSA_WITH_AES_256_CBC_SHA` đã được khuyến nghị từ nhiều năm trước do tồn tại nhiều vấn đề liên quan cả đến chế độ mã hoá AES-CBC và thuật toán băm SHA-1. Ta có thể dùng những phát hiện này để rút ra hiểu biết về mức độ tuân thủ các thực hành an ninh — hoặc thiếu nó — trong tổ chức mục tiêu.

---

# 3. Thu thập Thông tin Chủ động

---

**Các Mục tiêu Học tập của bài này:**

- Học cách quét cổng bằng Netcat và Nmap
- Thực hiện liệt kê (enumeration) DNS, SMB, SMTP và SNMP
- Hiểu các kỹ thuật “Living off the Land”

Trong bài học này, chúng ta sẽ đi xa hơn phần thu thập thông tin thụ động và khám phá các kỹ thuật liên quan đến việc tương tác trực tiếp với các dịch vụ mục tiêu. Cần ghi nhớ rằng có vô số dịch vụ có thể trở thành mục tiêu ngoài thực tế, ví dụ như Active Directory (sẽ được trình bày chi tiết trong một Mô-đun riêng). Dẫu vậy, trong Mô-đun này chúng ta sẽ điểm qua một số kỹ thuật thu thập thông tin chủ động phổ biến, bao gồm quét cổng và liệt kê DNS, SMB, SMTP và SNMP.

Chúng ta chủ yếu sẽ trình diễn các kỹ thuật thu thập thông tin chủ động có thể thực thi bằng các công cụ đã cài sẵn trên máy Kali của chúng ta. Tuy nhiên, trong một số tình huống kiểm thử thâm nhập, chúng ta sẽ không có “đặc quyền” sử dụng công cụ Kali ưa thích. Trong kịch bản **giả định đã bị xâm nhập** (*assumed breach*), khách hàng thường cấp cho chúng ta một máy trạm chạy Windows và chúng ta phải tận dụng những gì sẵn có trên Windows.

Khi **“Living off the Land”**, chúng ta có thể tận dụng nhiều **tệp thực thi Windows cài sẵn và được tin cậy** để thực hiện phân tích sau xâm nhập. Những tệp thực thi này được gọi tắt là **LOLBins**, hoặc gần đây là **LOLBAS** để bao quát cả **Binaries, Scripts, and Libraries** (tệp thực thi, script và thư viện).

*Xét một cách chặt chẽ, các binary LOLBAS thường được dùng theo cách **khác với mục đích thiết kế ban đầu**. Trong phạm vi này, chúng ta sẽ **nới lỏng định nghĩa** để bao gồm cả việc dùng các binary chuẩn của Windows “đúng như chúng vốn có” nhằm phục vụ thu thập thông tin.*

Ở các phần tiếp theo, chúng ta sẽ trình diễn những kỹ thuật LOLBAS phổ biến nhất cùng với các công cụ Kali thường dùng cho thu thập thông tin chủ động.

---

## 3.1. Liệt kê DNS

---

**Hệ thống Tên Miền (DNS)** là cơ sở dữ liệu phân tán chịu trách nhiệm ánh xạ các tên miền thân thiện với người dùng sang địa chỉ IP. Đây là một trong những hệ thống quan trọng nhất của Internet. DNS hoạt động theo cấu trúc phân cấp được chia thành nhiều **zone**, bắt đầu từ **root** cấp cao nhất.

Mỗi miền (domain) có thể sử dụng nhiều loại bản ghi DNS khác nhau. Một số loại thường gặp:

- **NS**: Bản ghi Nameserver chứa tên các máy chủ có thẩm quyền (authoritative) lưu trữ bản ghi DNS cho domain.
- **A**: Còn gọi là host record; chứa địa chỉ **IPv4** của một hostname (ví dụ: `www.megacorpone.com`).
- **AAAA**: “Quad A” host record; chứa địa chỉ **IPv6** của một hostname (ví dụ: `www.megacorpone.com`).
- **MX**: Bản ghi Mail Exchange chứa tên các máy chủ xử lý email cho domain. Một domain có thể có nhiều bản ghi MX.
- **PTR**: Bản ghi Pointer dùng trong **reverse lookup zones** để tra ngược từ IP về tên.
- **CNAME**: Bản ghi Canonical Name dùng để tạo **bí danh** (alias) trỏ đến bản ghi host khác.
- **TXT**: Bản ghi văn bản có thể chứa dữ liệu tùy ý, dùng cho nhiều mục đích (ví dụ: xác minh quyền sở hữu domain).

Vì DNS chứa rất nhiều thông tin, nó thường là mục tiêu “béo bở” cho hoạt động thu thập thông tin chủ động.

### Host

Trước hết, dùng lệnh `host` để tìm địa chỉ IP của `www.megacorpone.com`.

```bash
kali@kali:~$ host www.megacorpone.com
www.megacorpone.com has address 149.56.244.87
```

                                   *Listing 5 – Dùng `host` để tìm bản ghi A của `www.megacorpone.com`*

Mặc định, `host` tìm bản ghi **A**, nhưng ta có thể truy vấn các loại khác như **MX** hoặc **TXT** bằng cách chỉ định loại bản ghi với tùy chọn `-t`.

```bash
kali@kali:~$ host -t mx megacorpone.com
megacorpone.com mail is handled by 10 fb.mail.gandi.net.
megacorpone.com mail is handled by 20 spool.mail.gandi.net.
megacorpone.com mail is handled by 50 mail.megacorpone.com.
megacorpone.com mail is handled by 60 mail2.megacorpone.com.
```

                                 *Listing 6 – Dùng `host` để tìm các bản ghi MX của `megacorpone.com`*

Ở đây, chúng ta truy vấn riêng các bản ghi MX của `megacorpone.com` và nhận về bốn máy chủ thư với các mức **độ ưu tiên** khác nhau (10, 20, 50, 60). Máy chủ có số ưu tiên **thấp nhất** sẽ được dùng trước để chuyển tiếp thư đến domain (ở đây là `fb.mail.gandi.net`).

Tiếp theo, truy vấn các bản ghi TXT:

```bash
kali@kali:~$ host -t txt megacorpone.com
megacorpone.com descriptive text "Try Harder"
megacorpone.com descriptive text "google-site-verification=U7B_b0HNeBtY4qYGQZNsEYXfCJ32hMNV3GtC0wWq5pA"
```

                               *Listing 7 – Dùng `host` để tìm các bản ghi TXT của `megacorpone.com`*

Sau khi có dữ liệu ban đầu, ta tiếp tục dùng các truy vấn DNS khác để khám phá thêm hostname và IP thuộc cùng domain. Ví dụ, ta biết domain có web server với hostname `www.megacorpone.com`.

Chạy `host` với hostname hợp lệ:

```bash
kali@kali:~$ host www.megacorpone.com
www.megacorpone.com has address 149.56.244.87
```

                                                    *Listing 8 – Dùng `host` để tìm một hostname hợp lệ*

Thử kiểm tra hostname **không tồn tại** để so sánh đầu ra:

```bash
kali@kali:~$ host idontexist.megacorpone.com
Host idontexist.megacorpone.com not found: 3(NXDOMAIN)
```

                                              *Listing 9 – Dùng `host` để tìm một hostname không hợp lệ*

Trong Listing 8, truy vấn hostname hợp lệ cho kết quả phân giải IP. Ngược lại, Listing 9 trả lỗi **NXDOMAIN**, nghĩa là không có bản ghi DNS công khai cho hostname đó. Hiểu được cách phân biệt này, ta có thể **tự động hóa** việc tìm các hostname hợp lệ.

### Brute force subdomain/hostname

**Brute force** là kỹ thuật thử–sai để tìm thông tin hợp lệ (thư mục web, cặp username/password, hoặc ở đây là bản ghi DNS hợp lệ). Dùng một **wordlist** chứa các hostname phổ biến, ta “đoán” và kiểm tra phản hồi để phát hiện hostname hợp lệ.

Các ví dụ trên là **tra cứu thuận (forward lookup)**: hỏi IP của một hostname. Nếu `host` phân giải được tên về IP, đó có thể là dấu hiệu của một máy chủ hoạt động.

Ta có thể **tự động hóa** tra cứu thuận với `host` bằng một one-liner Bash.

Tạo danh sách hostname khả dĩ:

```bash
kali@kali:~$ cat list.txt
www
ftp
mail
owa
proxy
router
```

                                                            *Listing 10 – Một wordlist hostname nhỏ*

Dùng one-liner để thử phân giải từng mục:

```bash
kali@kali:~$ for ip in $(cat list.txt); do host $ip.megacorpone.com; done
www.megacorpone.com has address 149.56.244.87
Host ftp.megacorpone.com not found: 3(NXDOMAIN)
mail.megacorpone.com has address 51.222.169.212
Host owa.megacorpone.com not found: 3(NXDOMAIN)
Host proxy.megacorpone.com not found: 3(NXDOMAIN)
router.megacorpone.com has address 51.222.169.214
```

                                                *Listing 11 – Dùng Bash để vét cạn tra cứu DNS thuận*

Từ wordlist đơn giản này, ta phát hiện `www`, `mail`, `router` hợp lệ; `ftp`, `owa`, `proxy` không tồn tại. Trong thực tế, nên dùng **SecLists** (wordlist toàn diện) có thể cài tại `/usr/share/seclists` bằng `sudo apt install seclists`.

Ngoại trừ bản ghi `www`, quá trình brute force **forward** cho thấy nhiều IP nằm trong khoảng gần nhau (`51.222.169.X`). Nếu quản trị DNS có cấu hình bản ghi **PTR**, ta có thể quét **reverse lookup** trên dải IP xấp xỉ đó để hỏi **hostname** ứng với từng IP.

Quét `51.222.169.200` đến `51.222.169.254`, lọc bỏ các dòng “not found”:

```bash
kali@kali:~$ for ip in $(seq 200 254); do host 51.222.169.$ip; done | grep -v "not found"
...
208.169.222.51.in-addr.arpa domain name pointer admin.megacorpone.com.
209.169.222.51.in-addr.arpa domain name pointer beta.megacorpone.com.
210.169.222.51.in-addr.arpa domain name pointer fs1.megacorpone.com.
211.169.222.51.in-addr.arpa domain name pointer intranet.megacorpone.com.
212.169.222.51.in-addr.arpa domain name pointer mail.megacorpone.com.
213.169.222.51.in-addr.arpa domain name pointer mail2.megacorpone.com.
214.169.222.51.in-addr.arpa domain name pointer router.megacorpone.com.
215.169.222.51.in-addr.arpa domain name pointer siem.megacorpone.com.
216.169.222.51.in-addr.arpa domain name pointer snmp.megacorpone.com.
217.169.222.51.in-addr.arpa domain name pointer syslog.megacorpone.com.
218.169.222.51.in-addr.arpa domain name pointer support.megacorpone.com.
219.169.222.51.in-addr.arpa domain name pointer test.megacorpone.com.
220.169.222.51.in-addr.arpa domain name pointer vpn.megacorpone.com.
...
```

                                *Listing 12 – Dùng Bash để vét cạn tra cứu DNS ngược (reverse)*

Ta đã phân giải được nhiều IP → hostname bằng **reverse lookup**. Khi đánh giá thực tế, ta có thể suy diễn tiếp: quét thêm `mail2`, `router`, v.v., rồi reverse lookup những kết quả dương tính. Những vòng lặp như vậy mang tính **chu trình**: mở rộng phạm vi dựa trên thông tin thu được sau mỗi lượt.

### Kali tools

Có nhiều công cụ tự động hóa DNS enumeration. Hai công cụ tiêu biểu: **DNSRecon** và **DNSEnum**.

**DNSRecon** (Python) – chạy **standard scan** với domain đích:

```bash
kali@kali:~$ dnsrecon -d megacorpone.com -t std
[*] std: Performing General Enumeration against: megacorpone.com...
[-] DNSSEC is not configured for megacorpone.com
[*] 	 SOA ns1.megacorpone.com 51.79.37.18
[*] 	 NS ns1.megacorpone.com 51.79.37.18
[*] 	 NS ns3.megacorpone.com 66.70.207.180
[*] 	 NS ns2.megacorpone.com 51.222.39.63
[*] 	 MX mail.megacorpone.com 51.222.169.212
[*] 	 MX spool.mail.gandi.net 217.70.178.1
[*] 	 MX fb.mail.gandi.net 217.70.178.217
[*] 	 MX fb.mail.gandi.net 217.70.178.216
[*] 	 MX fb.mail.gandi.net 217.70.178.215
[*] 	 MX mail2.megacorpone.com 51.222.169.213
[*] 	 TXT megacorpone.com Try Harder
[*] 	 TXT megacorpone.com google-site-verification=U7B_b0HNeBtY4qYGQZNsEYXfCJ32hMNV3GtC0wWq5pA
[*] Enumerating SRV Records
[+] 0 Records Found
```

                                           *Listing 13 – Dùng `dnsrecon` để chạy standard scan*

Kết quả cho thấy ta đã quét thành công các loại bản ghi chính trên domain `megacorpone.com`.

Brute force subdomain bằng file `list.txt` đã tạo:

```bash
kali@kali:~$ cat list.txt
www
ftp
mail
owa
proxy
router
```

                               *Listing 14 – Wordlist dùng brute force subdomain với `dnsrecon`*

Chạy brute force: `-d` (domain), `-D` (file wordlist), `-t brt` (brute force):

```bash
kali@kali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
[*] Using the dictionary file: /home/kali/list.txt (provided by user)
[*] brt: Performing host and subdomain brute force against megacorpone.com...
[+] 	 A www.megacorpone.com 149.56.244.87
[+] 	 A mail.megacorpone.com 51.222.169.212
[+] 	 A router.megacorpone.com 51.222.169.214
[+] 3 Records Found
```

                                              *Listing 15 – Brute force hostname bằng `dnsrecon`*

**DNSEnum** – công cụ phổ biến khác để tự động hóa DNS enum:

```bash
kali@kali:~$ dnsenum megacorpone.com
...
dnsenum VERSION:1.2.6

-----   megacorpone.com   -----

...

Brute forcing with /usr/share/dnsenum/dns.txt:
_______________________________________________

admin.megacorpone.com.                   5        IN    A        51.222.169.208
beta.megacorpone.com.                    5        IN    A        51.222.169.209
fs1.megacorpone.com.                     5        IN    A        51.222.169.210
intranet.megacorpone.com.                5        IN    A        51.222.169.211
mail.megacorpone.com.                    5        IN    A        51.222.169.212
mail2.megacorpone.com.                   5        IN    A        51.222.169.213
ns1.megacorpone.com.                     5        IN    A        51.79.37.18
ns2.megacorpone.com.                     5        IN    A        51.222.39.63
ns3.megacorpone.com.                     5        IN    A        66.70.207.180
router.megacorpone.com.                  5        IN    A        51.222.169.214
siem.megacorpone.com.                    5        IN    A        51.222.169.215
snmp.megacorpone.com.                    5        IN    A        51.222.169.216
syslog.megacorpone.com.                  5        IN    A        51.222.169.217
test.megacorpone.com.                    5        IN    A        51.222.169.219
vpn.megacorpone.com.                     5        IN    A        51.222.169.220
www.megacorpone.com.                     5        IN    A        149.56.244.87
www2.megacorpone.com.                    5        IN    A        149.56.244.87

megacorpone.com class C netranges:
___________________________________

 51.79.37.0/24
 51.222.39.0/24
 51.222.169.0/24
 66.70.207.0/24
 149.56.244.0/24

Performing reverse lookup on 1280 ip addresses:
________________________________________________

18.37.79.51.in-addr.arpa.                86400    IN    PTR      ns1.megacorpone.com.
...
```

*Listing 16 – Dùng `dnsenum` để tự động hóa DNS enumeration*

Nhờ quá trình enum sâu rộng, ta đã phát hiện thêm nhiều host trước đó **chưa biết**. Như đã nói, thu thập thông tin có tính **chu trình**: cần tiếp tục các bước enum thụ động/chủ động khác trên **tập host mới** để lộ thêm chi tiết.

Các công cụ trên đều **thiết thực và dễ dùng** – nên thành thạo trước khi đi tiếp.

### DNS  Enumeration on Windows

Dù không nằm trong danh mục LOLBAS, **`nslookup`** vẫn là tiện ích tuyệt vời cho DNS enumeration trên Windows và thường được dùng trong tình huống **Living off the Land**.

*Các ứng dụng có thể dẫn đến thực thi mã ngoài ý muốn thường được liệt kê trong dự án LOLBAS; tuy nhiên, ở đây ta chỉ dùng nslookup đúng chức năng để thu thập thông tin.*

Kết nối vào máy Windows 11 bằng **xfreerdp**:

```bash
kali@kali:~$ xfreerdp /u:student /p:lab /v:192.168.50.152
```

                                                          *Listing 17 – Kết nối tới máy Windows 11*

Trên Windows 11, mở Command Prompt và chạy truy vấn đơn giản phân giải bản ghi A cho `mail.megacorptwo.com`:

```
C:\Users\student>nslookup mail.megacorptwo.com
DNS request timed out.
    timeout was 2 seconds.
Server:  UnKnown
Address:  192.168.50.151

Name:    mail.megacorptwo.com
Address:  192.168.50.154
```

                                           *Listing 18 – Dùng `nslookup` để enum host đơn giản*

Ở đầu ra trên, ta truy vấn **DNS mặc định** (`192.168.50.151`) để phân giải IP của `mail.megacorptwo.com`, và máy chủ DNS trả về `192.168.50.154`.

Tương tự `host` trên Linux, `nslookup` có thể chạy truy vấn chi tiết hơn, ví dụ hỏi bản ghi **TXT** của một host cụ thể (và có thể chỉ định **DNS server** để hỏi):

```
C:\Users\student>nslookup -type=TXT info.megacorptwo.com 192.168.50.151
Server:  UnKnown
Address:  192.168.50.151

info.megacorptwo.com    text =

        "greetings from the TXT record body"
```

                                                *Listing 19 – Dùng `nslookup` để truy vấn cụ thể hơn*

Trong ví dụ này, ta **chỉ định** máy chủ DNS `192.168.50.151` để hỏi mọi bản ghi **TXT** liên quan đến `info.megacorptwo.com`.

`nslookup` **linh hoạt** không kém `host` và có thể **tự động hóa** thêm bằng **PowerShell** hoặc **Batch**.

---

## 3.2. Lý thuyết Quét Cổng TCP/UDP

---

**Quét cổng** là quá trình kiểm tra các cổng TCP hoặc UDP trên một máy từ xa nhằm phát hiện những dịch vụ đang chạy trên mục tiêu và các khả năng tấn công tiềm ẩn.

*Lưu ý pháp lý: Quét cổng không phải là hoạt động người dùng thông thường và có thể bị coi là bất hợp pháp ở một số khu vực pháp lý. Vì vậy, không được thực hiện bên ngoài môi trường lab nếu không có sự cho phép bằng văn bản từ chủ sở hữu mạng mục tiêu.*

Cần hiểu rõ hệ quả của việc quét cổng và tác động của từng kiểu quét cụ thể. Do lượng traffic mà một số kiểu quét có thể tạo ra cùng tính chất xâm nhập, việc quét bừa có thể gây **ảnh hưởng xấu** đến hệ thống mục tiêu hoặc mạng của khách hàng, như **quá tải máy chủ/đường truyền** hoặc **kích hoạt IDS/IPS**. Chạy **sai loại quét** có thể dẫn tới **downtime** cho khách hàng.

Áp dụng **phương pháp quét có hệ thống** giúp nâng cao hiệu quả và giảm rủi ro. Tùy phạm vi bài kiểm thử, thay vì quét toàn bộ cổng ngay, ta có thể **bắt đầu** chỉ với cổng **80** và **443**. Khi đã có danh sách máy chủ web tiềm năng, ta chạy quét toàn bộ cổng **nền** trên các máy đó đồng thời tiếp tục các bước enum khác. Khi quét toàn bộ hoàn tất, ta **thu hẹp** và **đi sâu** hơn ở các lượt quét sau. Hãy xem quét cổng như một **quy trình động**, **riêng** cho từng bài test: **kết quả của lượt quét trước sẽ quyết định loại và phạm vi của lượt quét kế tiếp**.

### Netcat

Mở đầu với quét TCP và UDP đơn giản bằng **Netcat**. Lưu ý Netcat **không phải** là port scanner, nhưng ta có thể tận dụng nó ở mức cơ bản để minh họa cách một port scanner hoạt động.

Vì Netcat có mặt sẵn trên nhiều hệ thống, ta có thể **tái sử dụng** chức năng của nó để mô phỏng quét cơ bản khi không cần trình quét đầy đủ tính năng. (Phần sau sẽ dùng các công cụ chuyên dụng hơn.)

> Quét TCP – CONNECT scan (bắt tay 3 bước)
> 

Kỹ thuật TCP đơn giản nhất, thường gọi là **CONNECT scan**, dựa vào cơ chế **bắt tay 3 bước TCP**:

- Máy khách gửi **SYN** tới cổng đích.
- Nếu cổng **mở**, máy chủ đáp lại **SYN-ACK**.
- Máy khách gửi **ACK** để hoàn tất bắt tay → cổng được coi là **open**.

Ví dụ chạy Netcat quét các cổng **3388–3390**. Dùng `-w` đặt **timeout (giây)** và `-z` **zero-I/O** (chế độ quét, không gửi dữ liệu).

```bash
kali@kali:~$ nc -nvv -w 1 -z 192.168.50.152 3388-3390
(UNKNOWN) [192.168.50.152] 3390 (?) : Connection refused
(UNKNOWN) [192.168.50.152] 3389 (ms-wbt-server) open
(UNKNOWN) [192.168.50.152] 3388 (?) : Connection refused
 sent 0, rcvd 0
```

                                             *Listing 20 – Dùng Netcat để thực hiện quét cổng TCP*

Từ đầu ra trên: cổng **3389** mở; **3388** và **3390** bị **từ chối kết nối**.

![image.png](image%2017.png)

**Wireshark (Hình 17)** cho thấy Netcat gửi gói **TCP SYN** tới các cổng 3390, 3389, 3388 (các gói 1, 3, 7). Do thời gian, các gói có thể xuất hiện **không theo thứ tự**. Máy chủ gửi **SYN-ACK** từ cổng 3389 ở gói 4 → cổng mở. Các cổng còn lại không trả **SYN-ACK**, mà **từ chối** bằng **RST-ACK**. Cuối cùng, ở gói 6, Netcat đóng kết nối bằng **FIN-ACK**.

> Quét UDP
> 

Với UDP (giao thức **không trạng thái**, **không** có bắt tay 3 bước), cơ chế quét **khác** TCP.

Chạy Netcat quét UDP trên các cổng **120–123** (tùy chọn `-u`):

```bash
kali@kali:~$ nc -nv -u -z -w 1 192.168.50.149 120-123
(UNKNOWN) [192.168.50.149] 123 (ntp) open
```

                                           *Listing 21 – Dùng Netcat để thực hiện quét cổng UDP*

![image.png](image%2018.png)

**Wireshark (Hình 18)** cho thấy Netcat gửi **UDP rỗng** đến từng cổng (các gói 2, 3, 4, 6, 8).

- Nếu cổng UDP **mở**, gói được chuyển lên tầng ứng dụng; phản hồi **tùy** cách ứng dụng xử lý gói rỗng (ví dụ có thể **không trả lời**).
- Nếu cổng UDP **đóng**, mục tiêu thường **phản hồi** bằng **ICMP Port Unreachable** (như gói 5, 7, 9) do ngăn xếp UDP/IP của máy đích gửi.

Phần lớn trình quét UDP **suy luận trạng thái** cổng dựa vào **ICMP Port Unreachable**. Tuy nhiên, nếu cổng bị **tường lửa lọc**, ICMP có thể **không** quay về → trình quét có thể **nhầm** là **mở** do **thiếu phản hồi** ICMP.

### Một số “bẫy” thường gặp khi quét

- **UDP kém tin cậy:** Tường lửa/router có thể **drop ICMP**, dẫn đến **dương tính giả** (báo mở nhưng thực tế đóng).
- **Không quét đủ cổng:** Nhiều trình quét dùng **danh sách sẵn** các **“cổng thú vị”**, nên có thể **bỏ sót** UDP mở. Dùng **trình quét UDP theo giao thức cụ thể** (protocol-specific) giúp chính xác hơn.
- **Bỏ quên UDP:** Pentester hay tập trung TCP “thú vị” mà **quên** UDP, trong khi nhiều **vector tấn công** nằm sau các cổng UDP mở.
- **Chi phí traffic:** Quét **TCP** thường sinh **nhiều traffic hơn** quét UDP (do overhead và **retransmission**), dễ gây chú ý hoặc ảnh hưởng hệ thống hơn.

---

## 3.3. Quét Cổng với Nmap

---

Sau khi đã nắm vững nền tảng về quét cổng, giờ ta tìm hiểu **Nmap** – công cụ “de-facto” cho tác vụ này.

**Nmap** (do Gordon Lyon, aka *Fyodor*, phát triển) là một trong những trình quét cổng **phổ biến, linh hoạt và mạnh mẽ** nhất, được phát triển liên tục hơn hai thập kỷ và có **rất nhiều tính năng vượt xa** quét cổng thuần túy.

Một số ví dụ trong Mô-đun này chạy kèm `sudo` vì nhiều tùy chọn của Nmap cần truy cập **raw sockets** (yêu cầu đặc quyền root). **Raw sockets** cho phép “điều khiển vi phẫu” các gói TCP/UDP; nếu không có, Nmap bị giới hạn và phải “tự chế” gói qua **Berkeley socket API**.

Trước khi thực hành, ta cần hiểu **dấu vết (footprint)** mà mỗi kiểu quét để lại “trên dây” (wire) và trên host bị quét.

### Đo “dấu chân” của quét TCP mặc định

Mặc định, Nmap sẽ quét **1000 cổng TCP phổ biến nhất** trên một máy. Trước khi “quét bừa”, ta đo lượng traffic mà kiểu quét này tạo ra: quét một máy lab trong khi dùng `iptables` để đếm traffic gửi tới host đích.

Dùng các tùy chọn `iptables`: `-I` để **chèn rule** vào chain (`INPUT`/`OUTPUT`) ở vị trí số 1; `-s` chỉ nguồn, `-d` chỉ đích, `-j ACCEPT` để cho phép; `-Z` để **reset bộ đếm** packet/byte.

```bash
kali@kali:~$ sudo iptables -I INPUT 1 -s 192.168.50.149 -j ACCEPT

kali@kali:~$ sudo iptables -I OUTPUT 1 -d 192.168.50.149 -j ACCEPT

kali@kali:~$ sudo iptables -Z
```

                                                     *Listing 22 – Cấu hình `iptables` cho lần quét*

Chạy quét mặc định:

```bash
kali@kali:~$ nmap 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 05:12 EST
Nmap scan report for 192.168.50.149
Host is up (0.10s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl

Nmap done: 1 IP address (1 host up) scanned in 10.95 seconds
```

                                                           *Listing 23 – Quét 1000 cổng TCP phổ biến*

Xem thống kê `iptables` để biết **bao nhiêu traffic** đã phát sinh:

```bash
kali@kali:~$ sudo iptables -vn -L
Chain INPUT (policy ACCEPT 1270 packets, 115K bytes)
 pkts bytes target     prot opt in     out     source               destination
 1196 47972 ACCEPT     all  --  *      *       192.168.50.149      0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)

Chain OUTPUT (policy ACCEPT 1264 packets, 143K bytes)
 pkts bytes target     prot opt in     out     source               destination
 1218 72640 ACCEPT     all  --  *      *       0.0.0.0/0            192.168.50.149
```

                                  *Listing 24 – Dùng `iptables` theo dõi traffic của lần quét top-1000*

Theo đầu ra, quét mặc định **tạo khoảng 72 KB** traffic.

Reset bộ đếm rồi quét **tất cả cổng TCP**:

```bash
kali@kali:~$ sudo iptables -Z

kali@kali:~$ nmap -p 1-65535 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 05:23 EST
...
Nmap done: 1 IP address (1 host up) scanned in 2141.22 seconds

kali@kali:~$ sudo iptables -vn -L
Chain INPUT (policy ACCEPT 67996 packets, 6253K bytes)
 pkts bytes target     prot opt in     out     source               destination
68724 2749K ACCEPT     all  --  *      *       192.168.50.149      0.0.0.0/0

Chain OUTPUT (policy ACCEPT 67923 packets, 7606K bytes)
 pkts bytes target     prot opt in     out     source               destination
68807 4127K ACCEPT     all  --  *      *       0.0.0.0/0            192.168.50.149
```

                                        *Listing 25 – Theo dõi traffic cho lần quét **toàn bộ** cổng TCP*

Kết quả: quét 1–65535 sinh khoảng **4 MB** traffic (cao hơn nhiều) nhưng **tìm thêm** cổng mở. Suy rộng ra, quét **toàn bộ lớp C (254 host)** có thể gửi **> 1000 MB** traffic. Lý tưởng thì quét đủ TCP+UDP mọi host sẽ chính xác nhất, nhưng ta phải **cân bằng giới hạn băng thông/thời gian** với nhu cầu khám phá. Điều này càng quan trọng khi đánh giá mạng lớn (lớp A/B).

*Các trình quét hiện đại như **MASSCAN** hay **RustScan** **nhanh hơn** Nmap nhưng tạo **nhiều kết nối đồng thời**, gây tải băng thông. Ngược lại, Nmap có **rate limiting**, **đỡ ồn** và “kín đáo” hơn.*

### Kỹ thuật quét tiêu biểu của Nmap

1. **SYN (Stealth) Scan** – `-sS`

Quét **SYN** gửi gói SYN tới cổng đích **nhưng không hoàn tất** bắt tay TCP. Nếu cổng **mở**, máy đích trả **SYN-ACK**; Nmap **không** gửi ACK cuối.

```bash
kali@kali:~$ sudo nmap -sS 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 06:31 EST
Nmap scan report for 192.168.50.149
Host is up (0.11s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
...
```

                                                                         *Listing 26 – Quét SYN*

Vì **không hoàn tất** bắt tay, thông tin **không** lên tầng ứng dụng → **ít/không xuất hiện** trong log ứng dụng; đồng thời **nhanh và hiệu quả** hơn (ít gói hơn).

*Xin lưu ý rằng thuật ngữ "stealth" ám chỉ thực tế là trước đây, firewall không thể ghi lại các kết nối TCP chưa hoàn chỉnh. Điều này không còn đúng với firewall hiện đại nữa, và mặc dù thuật ngữ "stealth" vẫn còn tồn tại, nhưng nó có thể gây hiểu lầm.*

1. **TCP Connect Scan** – `-sT`

Khi **không có đặc quyền raw socket**, Nmap **mặc định** dùng **Connect scan** (qua **Berkeley sockets API**) và **hoàn tất** bắt tay TCP, nên **chậm hơn** `-sS`.

```bash
kali@kali:~$ nmap -sT 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 06:44 EST
Nmap scan report for 192.168.50.149
Host is up (0.11s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
...
```

                                                                 *Listing 27 – Quét TCP Connect*

Kết quả vẫn hữu ích để **suy đoán HĐH/role** (ví dụ nhiều cổng đặc trưng Domain Controller).

1. **UDP Scan** – `-sU`

Nmap dùng kết hợp hai cách: đa số cổng dùng **ICMP Port Unreachable** (gửi **UDP rỗng**); với cổng phổ biến (như **161/UDP SNMP**) Nmap sẽ gửi **payload đặc thù giao thức** để “mồi” phản hồi ứng dụng.

```bash
kali@kali:~$ sudo nmap -sU 192.168.50.149
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-04 11:46 EST
Nmap scan report for 192.168.131.149
Host is up (0.11s latency).
Not shown: 977 closed udp ports (port-unreach)
PORT      STATE         SERVICE
123/udp   open          ntp
389/udp   open          ldap
...
Nmap done: 1 IP address (1 host up) scanned in 22.49 seconds
```

                                                                          *Listing 28 – Quét UDP*

Có thể **kết hợp** SYN + UDP để có bức tranh đầy đủ:

```bash
kali@kali:~$ sudo nmap -sU -sS 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 08:16 EST
Nmap scan report for 192.168.50.149
Host is up (0.10s latency).
Not shown: 989 closed tcp ports (reset), 977 closed udp ports (port-unreach)
PORT      STATE         SERVICE
53/tcp    open          domain
88/tcp    open          kerberos-sec
135/tcp   open          msrpc
139/tcp   open          netbios-ssn
389/tcp   open          ldap
445/tcp   open          microsoft-ds
464/tcp   open          kpasswd5
593/tcp   open          http-rpc-epmap
636/tcp   open          ldapssl
3268/tcp  open          globalcatLDAP
3269/tcp  open          globalcatLDAPssl
53/udp    open          domain
123/udp   open          ntp
389/udp   open          ldap
...
```

                                                             *Listing 29 – Quét kết hợp UDP + SYN*

### Network Sweeping

Để xử lý **nhiều host** và **tiết kiệm traffic**, ta quét **rộng** trước, rồi **đi sâu** vào host đáng chú ý.

**Ping/Host discovery** – `-sn` không chỉ gửi **ICMP Echo** mà còn gửi **TCP SYN 443**, **TCP ACK 80**, **ICMP timestamp**:

```bash
kali@kali:~$ nmap -sn 192.168.50.1-253
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 03:19 EST
Nmap scan report for 192.168.50.6
Host is up (0.12s latency).
Nmap scan report for 192.168.50.8
Host is up (0.12s latency).
...
Nmap done: 254 IP addresses (13 hosts up) scanned in 3.74 seconds
```

                                                           *Listing 30 – Quét sweep phát hiện host*

Dạng output mặc định hơi khó grep, dùng **greppable output** `-oG`:

```bash
kali@kali:~$ nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 03:21 EST
Initiating Ping Scan at 03:21
...
Read data files from: /usr/bin/../share/nmap
Nmap done: 254 IP addresses (13 hosts up) scanned in 3.74 seconds
...

kali@kali:~$ grep Up ping-sweep.txt | cut -d " " -f 2
192.168.50.6
192.168.50.8
192.168.50.9
...
```

                                             *Listing 31 – Lưu kết quả dạng `-oG` rồi grep host sống*

**Sweep theo cổng cụ thể** (chính xác hơn ping sweep):

```bash
kali@kali:~$ nmap -p 80 192.168.50.1-253 -oG web-sweep.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 03:50 EST
Nmap scan report for 192.168.50.6
Host is up (0.11s latency).

PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for 192.168.50.8
Host is up (0.11s latency).

PORT   STATE  SERVICE
80/tcp closed http
...

kali@kali:~$ grep open web-sweep.txt | cut -d" " -f2
192.168.50.6
192.168.50.20
192.168.50.21
```

                                                      *Listing 32 – Quét tìm web server (cổng 80)*

**Quét nhanh nhiều IP với ít cổng “phổ biến”** – `--top-ports=20`, bật **OS detect + script + traceroute** bằng `-A` (ở đây dùng `-sT`):

```bash
kali@kali:~$ nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 04:04 EST
Nmap scan report for 192.168.50.6
Host is up (0.12s latency).

PORT     STATE  SERVICE       VERSION
21/tcp   closed ftp
22/tcp   open   ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 56:57:11:b5:dc:f1:13:d3:50:88:b8:ab:a9:83:e2:29 (RSA)
|   256 4f:1d:f2:55:cb:40:e0:76:b4:36:90:19:a2:ba:f0:44 (ECDSA)
|_  256 67:46:b3:97:26:a9:e3:a8:4d:eb:20:b3:9b:8d:7a:32 (ED25519)
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   closed domain
80/tcp   open   http          Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Under Construction
110/tcp  closed pop3
111/tcp  closed rpcbind
...
```

                                              *Listing 33 – Quét top-20 cổng, lưu dạng greppable*

Danh sách **“top ports”** lấy từ `/usr/share/nmap/nmap-services` (3 cột: **tên dịch vụ**, **cổng/proto**, **tần suất mở**):

```bash
kali@kali:~$ cat /usr/share/nmap/nmap-services 
...
finger    79/udp    0.000956
http    80/sctp    0.000000    # www-http | www | World Wide Web HTTP
http    80/tcp    0.484143    # World Wide Web HTTP
http    80/udp    0.035767    # World Wide Web HTTP
hosts2-ns    81/tcp    0.012056    # HOSTS2 Name Server
hosts2-ns    81/udp    0.001005    # HOSTS2 Name Server
...
```

                               *Listing 34 – `nmap-services` cho thấy “độ phổ biến” cổng 80/TCP*

Sau bước sweep, ta **quét sâu** từng host “giàu dịch vụ”/đáng chú ý.

### Nhận diện HĐH (OS Fingerprinting) – `O`

Nmap có thể **đoán HĐH** dựa trên khác biệt nhỏ ở **TCP/IP stack** (TTL mặc định, TCP window size, v.v.). Mặc định, Nmap chỉ in kết quả khi **độ chính xác cao**; dùng thêm `--osscan-guess` để **ép** in “đoán” thô.

```bash
kali@kali:~$ sudo nmap -O 192.168.50.14 --osscan-guess
...
Running (JUST GUESSING): Microsoft Windows 2008|2012|2016|7|Vista (88%)
OS CPE: cpe:/o:microsoft:windows_server_2008::sp1 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista::sp1:home_premium
Aggressive OS guesses: Microsoft Windows Server 2008 SP1 or Windows Server 2008 R2 (88%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (88%), Microsoft Windows Server 2012 R2 (88%), Microsoft Windows Server 2012 (87%), Microsoft Windows Server 2016 (87%), Microsoft Windows 7 (86%), Microsoft Windows Vista Home Premium SP1 (85%), Microsoft Windows 7 Professional (85%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
...
```

                                                                 *Listing 35 – Dò HĐH bằng Nmap*

Lưu ý: OS fingerprinting **không luôn chính xác 100%**, nhất là khi có firewall/proxy **chỉnh sửa header**.

### Banner grabbing

Dùng `-A` để bật **OS detect, version detect, script scan, traceroute**; hoặc chỉ **service/version** với `-sV`.

```bash
kali@kali:~$ nmap -sT -A 192.168.50.14
Nmap scan report for 192.168.50.14
Host is up (0.12s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
21/tcp  open  ftp?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, NULL, RPCCheck, SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|     220-FileZilla Server 1.2.0
|     Please visit https://filezilla-project.org/
|   GetRequest:
|     220-FileZilla Server 1.2.0
|     Please visit https://filezilla-project.org/
|     What are you trying to do? Go away.
|   HTTPOptions, RTSPRequest:
|     220-FileZilla Server 1.2.0
|     Please visit https://filezilla-project.org/
|     Wrong command.
|   Help:
|     220-FileZilla Server 1.2.0
|     Please visit https://filezilla-project.org/
|     214-The following commands are recognized.
|     USER TYPE SYST SIZE RNTO RNFR RMD REST QUIT
|     HELP XMKD MLST MKD EPSV XCWD NOOP AUTH OPTS DELE
|     CDUP APPE STOR ALLO RETR PWD FEAT CLNT MFMT
|     MODE XRMD PROT ADAT ABOR XPWD MDTM LIST MLSD PBSZ
|     NLST EPRT PASS STRU PASV STAT PORT
|_    Help ok.
| ftp-syst:
|_  SYST: UNIX emulated by FileZilla.
| ssl-cert: Subject: commonName=filezilla-server self signed certificate
| Not valid before: 2022-01-06T15:37:24
|_Not valid after:  2023-01-07T15:42:24
|_ssl-date: TLS randomness does not represent time
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Nmap done: 1 IP address (1 host up) scanned in 55.67 seconds
```

                                                         *Listing 36 – Banner grabbing / enum dịch vụ*

Banner grabbing **tăng traffic** và **làm chậm** quét; cân nhắc tùy chọn Nmap. Banner cũng có thể bị **giả mạo** để đánh lạc hướng.

### **NSE – Nmap Scripting Engine**

**NSE** cho phép chạy **script người dùng** để tự động hóa: từ enum DNS, brute force, đến nhận diện lỗ hổng. Script ở `/usr/share/nmap/scripts`.

Ví dụ script **`http-headers`**: cố gắng kết nối HTTP và liệt kê header:

```bash
kali@kali:~$ nmap --script http-headers 192.168.50.6
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 13:53 EST
Nmap scan report for 192.168.50.6
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-headers:
|   Date: Thu, 10 Mar 2022 18:53:29 GMT
|   Server: Apache/2.4.41 (Ubuntu)
|   Last-Modified: Thu, 10 Mar 2022 18:51:54 GMT
|   ETag: "d1-5d9e1b5371420"
|   Accept-Ranges: bytes
|   Content-Length: 209
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|
|_  (Request type: HEAD)

Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds
```

                                                        *Listing 37 – Dùng NSE (ví dụ `http-headers`)*

Xem mô tả/arg/usage của script với `--script-help`:

```bash
kali@kali:~$ nmap --script-help http-headers
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-10 13:54 EST

http-headers
Categories: discovery safe
https://nmap.org/nsedoc/scripts/http-headers.html
  Performs a HEAD request for the root folder ("/") of a web server and displays the HTTP headers returned.
...
```

                                                  *Listing 38 – `--script-help` để xem thông tin script*

Khi **không có Internet**, nhiều thông tin có ngay trong **file script**.

### Áp dụng tương tự từ Windows (Living off the Land)

Nếu enum ban đầu từ **Windows laptop** **không Internet** (không cài thêm công cụ, kể cả Nmap cho Windows), ta theo chiến lược **Living off the Land**. May mắn là có vài **hàm PowerShell** hữu ích.

**`Test-NetConnection`** kiểm tra **ICMP** và xem **một cổng TCP** có mở không:

```powershell
PS C:\Users\student> Test-NetConnection -Port 445 192.168.50.151

ComputerName     : 192.168.50.151
RemoteAddress    : 192.168.50.151
RemotePort       : 445
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.50.152
TcpTestSucceeded : True
```

                                              *Listing 39 – Kiểm tra cổng SMB 445 bằng PowerShell*

`TcpTestSucceeded : True` → cổng **445 mở**.

Có thể **script hóa** để quét **1..1024** cổng bằng `TcpClient` (tránh overhead thừa của `Test-NetConnection`):

```powershell
PS C:\Users\student> 1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
TCP port 88 is open
...
```

                                        *Listing 40 – Tự động quét cổng bằng PowerShell one-liner*

Chuỗi lệnh trên: tạo dãy 1..1024 → lặp gán vào `$_` → khởi tạo `Net.Sockets.TcpClient` và thử kết nối tới IP đích tại cổng tương ứng; nếu thành công thì **in ra cổng mở**.

> Đây mới chỉ là khởi đầu của PowerShell; bạn có thể mở rộng để tiệm cận nhiều tính năng “truyền thống” của Nmap.
> 

---

## 3.4. Liệt kê SMB

---

Thành tích an ninh của giao thức Server Message Block (SMB) đã ở mức kém trong nhiều năm do việc triển khai phức tạp và tính chất mở. Từ các phiên SMB null không xác thực trong Windows 2000 và XP, cho đến vô số lỗi và lỗ hổng SMB qua nhiều năm, SMB đã trải qua khá nhiều vấn đề.

Ghi nhớ điều này, giao thức SMB cũng đã được cập nhật và cải tiến song song với các phiên bản Windows.

Dịch vụ NetBIOS lắng nghe trên cổng TCP 139, cũng như một số cổng UDP. Cần lưu ý rằng SMB (cổng TCP 445) và NetBIOS là hai giao thức riêng biệt. NetBIOS là một giao thức và dịch vụ lớp phiên độc lập cho phép các máy tính trong mạng cục bộ giao tiếp với nhau. Mặc dù các triển khai SMB hiện đại có thể hoạt động không cần NetBIOS, nhưng NetBIOS over TCP (NBT) là cần thiết cho khả năng tương thích ngược và thường được bật cùng nhau. Điều này cũng có nghĩa là việc liệt kê hai dịch vụ này thường đi kèm với nhau. Các dịch vụ này có thể được quét bằng các công cụ như nmap, với cú pháp tương tự như sau:

```bash
kali@kali:~$ nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254

kali@kali:~$ cat smb.txt
# Nmap 7.92 scan initiated Thu Mar 17 06:03:12 2022 as: nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254
# Ports scanned: TCP(2;139,445) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 192.168.50.1 ()	Status: Down
...
Host: 192.168.50.21 ()	Status: Up
Host: 192.168.50.21 ()	Ports: 139/closed/tcp//netbios-ssn///, 445/closed/tcp//microsoft-ds///
...
Host: 192.168.50.217 ()	Status: Up
Host: 192.168.50.217 ()	Ports: 139/closed/tcp//netbios-ssn///, 445/closed/tcp//microsoft-ds///
# Nmap done at Thu Mar 17 06:03:18 2022 -- 254 IP addresses (15 hosts up) scanned in 6.17 seconds
```

                                          *Listing 41 - Sử dụng nmap để quét dịch vụ NetBIOS*

Chúng tôi đã lưu đầu ra của lần quét vào một tệp văn bản, qua đó cho thấy các host có cổng 139 và 445 mở.

Có những công cụ chuyên biệt khác để nhận diện thông tin NetBIOS, như nbtscan. Chúng ta có thể dùng công cụ này để truy vấn dịch vụ tên NetBIOS nhằm tìm các tên NetBIOS hợp lệ, chỉ định cổng UDP nguồn là 137 với tùy chọn -r.

```bash
kali@kali:~$ sudo nbtscan -r 192.168.50.0/24
Doing NBT name scan for addresses from 192.168.50.0/24

IP address       NetBIOS Name     Server    User             MAC address
------------------------------------------------------------------------------
192.168.50.124   SAMBA            <server>  SAMBA            00:00:00:00:00:00
192.168.50.134   SAMBAWEB         <server>  SAMBAWEB         00:00:00:00:00:00
...
```

                              *Listing 42 - Sử dụng nbtscan để thu thập thêm thông tin NetBIOS*

Lần quét này cho thấy hai tên NetBIOS thuộc về hai host. Kiểu thông tin này có thể được dùng để cải thiện thêm bối cảnh về các host đã quét, vì tên NetBIOS thường mô tả khá rõ vai trò của host trong tổ chức. Dữ liệu này có thể đưa vào vòng lặp thu thập thông tin của chúng ta, dẫn đến nhiều tiết lộ hơn.

Nmap cũng cung cấp nhiều script NSE hữu ích mà chúng ta có thể dùng để phát hiện và liệt kê dịch vụ SMB. Chúng ta sẽ tìm các script này trong thư mục /usr/share/nmap/scripts.

```bash
kali@kali:~$ ls -1 /usr/share/nmap/scripts/smb*
/usr/share/nmap/scripts/smb2-capabilities.nse
/usr/share/nmap/scripts/smb2-security-mode.nse
/usr/share/nmap/scripts/smb2-time.nse
/usr/share/nmap/scripts/smb2-vuln-uptime.nse
/usr/share/nmap/scripts/smb-brute.nse
/usr/share/nmap/scripts/smb-double-pulsar-backdoor.nse
/usr/share/nmap/scripts/smb-enum-domains.nse
/usr/share/nmap/scripts/smb-enum-groups.nse
/usr/share/nmap/scripts/smb-enum-processes.nse
/usr/share/nmap/scripts/smb-enum-sessions.nse
/usr/share/nmap/scripts/smb-enum-shares.nse
/usr/share/nmap/scripts/smb-enum-users.nse
/usr/share/nmap/scripts/smb-os-discovery.nse
...
```

                                         *Listing 43 - Tìm các script NSE SMB khác nhau của nmap*

Chúng ta đã tìm thấy một số script Nmap SMB NSE thú vị thực hiện nhiều tác vụ như phát hiện HĐH và liệt kê qua SMB.

*Script khám phá SMB chỉ hoạt động nếu SMBv1 được bật trên mục tiêu, điều này không phải mặc định trên các phiên bản Windows hiện đại. Tuy nhiên, vẫn còn nhiều hệ thống cũ chạy SMBv1, và chúng tôi đã bật phiên bản cụ thể này trên host Windows để mô phỏng kịch bản như vậy.*

Hãy thử module smb-os-discovery trên máy khách Windows 11.

```bash
kali@kali:~$ nmap -v -p 139,445 --script smb-os-discovery 192.168.50.152
...
PORT    STATE SERVICE      REASON
139/tcp open  netbios-ssn  syn-ack
445/tcp open  microsoft-ds syn-ack

Host script results:
| smb-os-discovery:
|   OS: Windows 10 Pro 22000 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: client01
|   NetBIOS computer name: CLIENT01\x00
|   Domain name: megacorptwo.com
|   Forest name: megacorptwo.com
|   FQDN: client01.megacorptwo.com
|_  System time: 2022-03-17T11:54:20-07:00
...
```

                      *Listing 44 - Sử dụng nmap scripting engine để thực hiện khám phá HĐH*

Script cụ thể này đã xác định một kết quả khớp tiềm năng cho hệ điều hành của host; tuy nhiên, chúng ta biết nó không chính xác vì host mục tiêu đang chạy Windows 11 thay vì Windows 10 như báo cáo.

*Như đã đề cập trước đó, mọi đầu ra liệt kê dịch vụ và HĐH của Nmap nên được xem xét với sự thận trọng, vì không thuật toán nào là hoàn hảo.*

Không giống các tùy chọn lấy dấu vân tay HĐH (OS fingerprinting) của Nmap mà chúng ta đã tìm hiểu trước đó, việc liệt kê HĐH qua script NSE cung cấp thêm thông tin như domain và các chi tiết khác liên quan đến Active Directory Domain Services. Cách tiếp cận này cũng có khả năng không bị chú ý, vì nó tạo ra ít lưu lượng hơn và có thể hòa vào hoạt động mạng doanh nghiệp bình thường.

Sau khi thảo luận về liệt kê SMB từ phía Kali, hãy tìm hiểu cách liệt kê nó từ máy khách Windows.

Một công cụ hữu ích để liệt kê các chia sẻ SMB trong môi trường Windows là net view. Nó liệt kê domain, tài nguyên và máy tính thuộc về một host nhất định. Ví dụ, khi đã kết nối vào máy ảo client01, chúng ta có thể liệt kê tất cả các chia sẻ đang chạy trên dc01.

```
C:\Users\student>net view \\dc01 /all
Shared resources at \\dc01

Share name  Type  Used as  Comment

-------------------------------------------------------------------------------
ADMIN$      Disk           Remote Admin
C$          Disk           Default share
IPC$        IPC            Remote IPC
NETLOGON    Disk           Logon server share
SYSVOL      Disk           Logon server share
The command completed successfully.
```

                                                 *Listing 45 - Chạy 'net view' để liệt kê chia sẻ từ xa*

Bằng cách cung cấp từ khóa /all, chúng ta có thể liệt kê các chia sẻ quản trị (administrative shares) kết thúc bằng ký tự đô la.

---

## **3.5. Liệt kê SMTP**

---

Chúng ta cũng có thể thu thập thông tin về một host hoặc mạng từ các máy chủ thư dễ tổn thương. **Simple Mail Transport Protocol (SMTP)** hỗ trợ một số lệnh thú vị như **VRFY** và **EXPN**. Yêu cầu **VRFY** đề nghị máy chủ xác minh một địa chỉ email, trong khi **EXPN** yêu cầu máy chủ cho biết các thành viên của một danh sách thư. Chúng thường có thể bị lạm dụng để xác minh người dùng hiện có trên máy chủ thư, là thông tin hữu ích trong một bài kiểm thử xâm nhập. Xem ví dụ sau:

```bash
kali@kali:~$ nc -nv 192.168.50.8 25
(UNKNOWN) [192.168.50.8] 25 (smtp) open
220 mail ESMTP Postfix (Ubuntu)
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table
^C
```

                                        *Listing 46 - Sử dụng `nc` để xác thực người dùng SMTP*

Chúng ta có thể quan sát sự khác nhau giữa các thông báo thành công và lỗi. Máy chủ SMTP xác minh sẵn sàng rằng người dùng tồn tại. Thủ tục này có thể được dùng để giúp đoán tên người dùng hợp lệ theo cách tự động. Tiếp theo, hãy xem xét đoạn script Python sau, nó mở một socket TCP, kết nối tới máy chủ SMTP và phát lệnh **VRFY** cho một tên người dùng được cung cấp:

```python
#!/usr/bin/python

import socket
import sys

if len(sys.argv) != 3:
        print("Usage: vrfy.py <username> <target_ip>")
        sys.exit(0)

# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the Server
ip = sys.argv[2]
connect = s.connect((ip,25))

# Receive the banner
banner = s.recv(1024)

print(banner)

# VRFY a user
user = (sys.argv[1]).encode()
s.send(b'VRFY ' + user + b'\r\n')
result = s.recv(1024)

print(result)

# Close the socket
s.close()
```

                                      *Listing 47 - Dùng Python để viết script liệt kê người dùng SMTP*

Chúng ta có thể chạy script bằng cách cung cấp tên người dùng cần kiểm tra làm đối số thứ nhất và IP đích làm đối số thứ hai.

```bash
kali@kali:~/Desktop$ python3 smtp.py root 192.168.50.8
b'220 mail ESMTP Postfix (Ubuntu)\r\n'
b'252 2.0.0 root\r\n'

kali@kali:~/Desktop$ python3 smtp.py johndoe 192.168.50.8
b'220 mail ESMTP Postfix (Ubuntu)\r\n'
b'550 5.1.1 <johndoe>: Recipient address rejected: User unknown in local recipient table\r\n'
```

                              *Listing 48 - Chạy script Python để thực hiện liệt kê người dùng SMTP*

Tương tự, chúng ta có thể lấy thông tin SMTP về mục tiêu từ máy khách Windows 11, như đã làm trước đó:

```powershell
PS C:\Users\student> Test-NetConnection -Port 25 192.168.50.8

ComputerName     : 192.168.50.8
RemoteAddress    : 192.168.50.8
RemotePort       : 25
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.50.152
TcpTestSucceeded : True
```

                                                     *Listing 49 - Quét cổng SMB qua PowerShell*

Thật không may, với **Test-NetConnection** chúng ta không thể tương tác đầy đủ với dịch vụ SMTP. Tuy nhiên, nếu chưa bật, chúng ta có thể cài bản Telnet client của Microsoft như sau:

```powershell
PS C:\Windows\system32> dism /online /Enable-Feature /FeatureName:TelnetClient
...
```

                                                           *Listing 50 - Cài đặt trình khách Telnet*

Cần lưu ý rằng cài đặt Telnet yêu cầu đặc quyền quản trị (administrator), điều này có thể gây khó khăn nếu chúng ta đang chạy với quyền thấp. Tuy nhiên, chúng ta có thể lấy tệp thực thi Telnet nằm trên một máy phát triển khác của mình tại `c:\windows\system32\telnet.exe` và chuyển nó sang máy Windows mà chúng ta đang kiểm thử.

Khi đã bật Telnet trên máy kiểm thử, chúng ta có thể kết nối tới máy đích và thực hiện liệt kê giống như đã làm từ Kali.

```bash
C:\Windows\system32>telnet 192.168.50.8 25
220 mail ESMTP Postfix (Ubuntu)
VRFY goofy
550 5.1.1 <goofy>: Recipient address rejected: User unknown in local recipient table
VRFY root
252 2.0.0 root
```

                                 *Listing 51 - Tương tác với dịch vụ SMTP qua Telnet trên Windows*

Đầu ra ở trên minh họa thêm một ví dụ về kiểu liệt kê mà chúng ta có thể thực hiện từ một host Windows đã bị xâm phạm khi không có sẵn Kali.

---

## 3.6. Liệt kê SNMP

---

Trong nhiều năm qua, chúng tôi thường nhận thấy rằng Giao thức Quản lý Mạng Đơn giản Simple Network Management Protocol  (SNMP) không được nhiều quản trị viên mạng hiểu rõ. Điều này thường dẫn đến các cấu hình sai SNMP, có thể gây rò rỉ thông tin đáng kể.

SNMP dựa trên UDP, một giao thức đơn giản, không trạng thái, do đó dễ bị giả mạo IP và tấn công phát lại (replay). Ngoài ra, các giao thức SNMP thường dùng như 1, 2 và 2c không cung cấp mã hóa lưu lượng, nghĩa là thông tin và thông tin xác thực SNMP có thể dễ dàng bị chặn trong mạng cục bộ. Các giao thức SNMP truyền thống cũng có cơ chế xác thực yếu và thường được để mặc định với chuỗi cộng đồng (community strings) public và private.

*Cho đến gần đây, SNMPv3 — giao thức cung cấp xác thực và mã hóa — chỉ được phát hành hỗ trợ DES-56, vốn đã được chứng minh là lược đồ mã hóa yếu và có thể bị brute-force dễ dàng. Bản triển khai SNMPv3 mới hơn hỗ trợ lược đồ mã hóa AES-256.*

Bởi vì tất cả những điều trên áp dụng cho một giao thức theo định nghĩa là để “Quản lý Mạng”, SNMP là một trong những giao thức yêu thích của chúng tôi để liệt kê thông tin.

*Vài năm trước, OffSec đã thực hiện một bài kiểm thử xâm nhập nội bộ cho một công ty cung cấp dịch vụ tích hợp mạng cho rất nhiều khách hàng doanh nghiệp, ngân hàng và các tổ chức tương tự khác. Sau vài giờ đánh giá hệ thống, chúng tôi phát hiện một mạng lớp B lớn với hàng nghìn router Cisco được kết nối. Người ta giải thích rằng mỗi router này là cổng (gateway) đến một khách hàng của họ, được dùng cho mục đích quản lý và cấu hình.*

*Một lần quét nhanh với thông tin xác thực telnet mặc định cisco/cisco đã phát hiện một router Cisco ADSL cấu hình thấp. Đào sâu hơn cho thấy một tập chuỗi cộng đồng SNMP public và private phức tạp trong tệp cấu hình của router. Hóa ra, các chuỗi cộng đồng public và private này được sử dụng trên mọi thiết bị mạng, cho toàn bộ dải lớp B, và còn vượt ra ngoài — quản lý thật “đơn giản”, phải không?*

*Một điều thú vị về phần cứng định tuyến doanh nghiệp là các thiết bị này thường hỗ trợ đọc và ghi tệp cấu hình thông qua truy cập chuỗi cộng đồng SNMP private. Vì các chuỗi cộng đồng private cho tất cả các router gateway giờ đã được chúng tôi biết, bằng cách viết một script đơn giản để sao chép tất cả cấu hình router trên mạng đó sử dụng các giao thức SNMP và TFTP, chúng tôi không chỉ xâm phạm cơ sở hạ tầng của toàn bộ công ty tích hợp mạng, mà còn cả cơ sở hạ tầng của khách hàng của họ.*

Giờ khi đã có hiểu biết cơ bản về SNMP, chúng ta có thể khám phá một trong những tính năng chính của nó: Cây MIB của SNMP.

Cơ sở Thông tin Quản lý SNMP Management Information Base (MIB) là một cơ sở dữ liệu chứa thông tin thường liên quan đến quản lý mạng. Cơ sở dữ liệu được tổ chức như một cái cây, với các nhánh đại diện cho các tổ chức hoặc chức năng mạng khác nhau. Các lá của cây (hoặc các điểm cuối cuối cùng) tương ứng với các giá trị biến cụ thể mà người dùng bên ngoài có thể truy cập và thăm dò. IBM Knowledge Center chứa rất nhiều thông tin về cây MIB.

Ví dụ, các giá trị MIB sau tương ứng với các tham số SNMP của Microsoft Windows cụ thể và chứa nhiều hơn thông tin dựa trên mạng:

| 1.3.6.1.2.1.25.1.6.0	 | Quy trình Hệ thống (System Processes) |
| --- | --- |
| 1.3.6.1.2.1.25.4.2.1.2	 | Chương trình Đang chạy (Running Programs) |
| 1.3.6.1.2.1.25.4.2.1.4	 | Đường dẫn Quy trình (Processes Path) |
| 1.3.6.1.2.1.25.2.3.1.4	 | Đơn vị Lưu trữ (Storage Units) |
| 1.3.6.1.2.1.25.6.3.1.2	 | Tên Phần mềm (Software Name) |
| 1.3.6.1.4.1.77.1.2.25	 | Tài khoản Người dùng (User Accounts) |
| 1.3.6.1.2.1.6.13.1.3	 | Các Cổng TCP Cục bộ (TCP Local Ports) |

                            *Bảng 1 - Các giá trị SNMP MIB của Windows*

Để quét tìm các cổng SNMP mở, chúng ta có thể chạy nmap với tùy chọn -sU để thực hiện quét UDP và tùy chọn --open để giới hạn đầu ra chỉ hiển thị các cổng mở.

```bash
kali@kali:~$ sudo nmap -sU --open -p 161 192.168.50.1-254 -oG open-snmp.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-14 06:02 EDT
Nmap scan report for 192.168.50.151
Host is up (0.10s latency).

PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds
...
```

                                               *Listing 52 - Sử dụng nmap để thực hiện quét SNMP*

Ngoài ra, chúng ta có thể dùng công cụ như onesixtyone, công cụ sẽ cố gắng brute-force dựa trên một danh sách địa chỉ IP. Trước hết, chúng ta phải tạo các tệp văn bản chứa các chuỗi cộng đồng và các địa chỉ IP muốn quét.

```bash
kali@kali:~$ echo public > community
kali@kali:~$ echo private >> community
kali@kali:~$ echo manager >> community

kali@kali:~$ for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips

kali@kali:~$ onesixtyone -c community -i ips
Scanning 254 hosts, 3 communities
192.168.50.151 [public] Hardware: Intel64 Family 6 Model 79 Stepping 1 AT/AT COMPATIBLE - Software: Windows Version 6.3 (Build 17763 Multiprocessor Free)
...
```

                      *Listing 53 - Sử dụng onesixtyone để brute-force các chuỗi cộng đồng*

Khi đã tìm ra các dịch vụ SNMP, chúng ta có thể bắt đầu truy vấn chúng để lấy các dữ liệu MIB cụ thể có thể thú vị.

Chúng ta có thể thăm dò và truy vấn các giá trị SNMP bằng công cụ như snmpwalk, với điều kiện chúng ta biết chuỗi cộng đồng chỉ-đọc SNMP, trong đa số trường hợp là “public”.

Sử dụng một số giá trị MIB được cung cấp trong Bảng 1, chúng ta có thể cố gắng liệt kê các giá trị tương ứng của chúng. Hãy thử ví dụ sau với một máy đã biết trong phòng lab, máy này mở cổng SNMP của Windows với chuỗi cộng đồng “public”. Lệnh này liệt kê toàn bộ cây MIB bằng cách dùng tùy chọn -c để chỉ định chuỗi cộng đồng, -v để chỉ định phiên bản SNMP, cũng như -t 10 để tăng thời gian chờ lên 10 giây:

```bash
kali@kali:~$ snmpwalk -c public -v1 -t 10 192.168.50.151
iso.3.6.1.2.1.1.1.0 = STRING: "Hardware: Intel64 Family 6 Model 79 Stepping 1 AT/AT COMPATIBLE - Software: Windows Version 6.3 (Build 17763 Multiprocessor Free)"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.311.1.1.3.1.3
iso.3.6.1.2.1.1.3.0 = Timeticks: (78235) 0:13:02.35
iso.3.6.1.2.1.1.4.0 = STRING: "admin@megacorptwo.com"
iso.3.6.1.2.1.1.5.0 = STRING: "dc01.megacorptwo.com"
iso.3.6.1.2.1.1.6.0 = ""
iso.3.6.1.2.1.1.7.0 = INTEGER: 79
iso.3.6.1.2.1.2.1.0 = INTEGER: 24
...
```

                                      *Listing 54 - Sử dụng snmpwalk để liệt kê toàn bộ cây MIB*

Theo một cách diễn giải khác, chúng ta có thể dùng đầu ra trên để lấy các địa chỉ email mục tiêu. Thông tin này có thể được dùng để xây dựng một cuộc tấn công kỹ nghệ xã hội nhắm vào các liên hệ mới phát hiện.

Để thực hành thêm những gì đã học, hãy khám phá một vài kỹ thuật liệt kê SNMP đối với mục tiêu Windows. Chúng ta sẽ dùng lệnh snmpwalk, lệnh có thể phân tích một nhánh cụ thể của Cây MIB gọi là OID.

Ví dụ sau liệt kê các người dùng Windows trên máy dc01.

```bash
kali@kali:~$ snmpwalk -c public -v1 192.168.50.151 1.3.6.1.4.1.77.1.2.25
iso.3.6.1.4.1.77.1.2.25.1.1.5.71.117.101.115.116 = STRING: "Guest"
iso.3.6.1.4.1.77.1.2.25.1.1.6.107.114.98.116.103.116 = STRING: "krbtgt"
iso.3.6.1.4.1.77.1.2.25.1.1.7.115.116.117.100.101.110.116 = STRING: "student"
iso.3.6.1.4.1.77.1.2.25.1.1.13.65.100.109.105.110.105.115.116.114.97.116.111.114 = STRING: "Administrator"
```

                            *Listing 55 - Sử dụng snmpwalk để liệt kê người dùng Windows*

Lệnh của chúng ta đã truy vấn một cây con MIB cụ thể được ánh xạ tới tất cả tên tài khoản người dùng cục bộ.

Một ví dụ khác, chúng ta có thể liệt kê tất cả các tiến trình đang chạy:

```bash
kali@kali:~$ snmpwalk -c public -v1 192.168.50.151 1.3.6.1.2.1.25.4.2.1.2
iso.3.6.1.2.1.25.4.2.1.2.1 = STRING: "System Idle Process"
iso.3.6.1.2.1.25.4.2.1.2.4 = STRING: "System"
iso.3.6.1.2.1.25.4.2.1.2.88 = STRING: "Registry"
iso.3.6.1.2.1.25.4.2.1.2.260 = STRING: "smss.exe"
iso.3.6.1.2.1.25.4.2.1.2.316 = STRING: "svchost.exe"
iso.3.6.1.2.1.25.4.2.1.2.372 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.472 = STRING: "svchost.exe"
iso.3.6.1.2.1.25.4.2.1.2.476 = STRING: "wininit.exe"
iso.3.6.1.2.1.25.4.2.1.2.484 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.540 = STRING: "winlogon.exe"
iso.3.6.1.2.1.25.4.2.1.2.616 = STRING: "services.exe"
iso.3.6.1.2.1.25.4.2.1.2.632 = STRING: "lsass.exe"
iso.3.6.1.2.1.25.4.2.1.2.680 = STRING: "svchost.exe"
...
```

                              *Listing 56 - Sử dụng snmpwalk để liệt kê các tiến trình Windows*

Lệnh đã trả về một mảng chuỗi, mỗi chuỗi chứa tên của tiến trình đang chạy. Thông tin này có thể có giá trị vì nó có thể tiết lộ các ứng dụng dễ tổn thương, hoặc thậm chí cho biết loại phần mềm chống virus nào đang chạy trên mục tiêu.

Tương tự, chúng ta có thể truy vấn tất cả phần mềm đã cài trên máy:

```bash
kali@kali:~$ snmpwalk -c public -v1 192.168.50.151 1.3.6.1.2.1.25.6.3.1.2
iso.3.6.1.2.1.25.6.3.1.2.1 = STRING: "Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.27.29016"
iso.3.6.1.2.1.25.6.3.1.2.2 = STRING: "VMware Tools"
iso.3.6.1.2.1.25.6.3.1.2.3 = STRING: "Microsoft Visual C++ 2019 X64 Additional Runtime - 14.27.29016"
iso.3.6.1.2.1.25.6.3.1.2.4 = STRING: "Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.27.290"
iso.3.6.1.2.1.25.6.3.1.2.5 = STRING: "Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.27.290"
iso.3.6.1.2.1.25.6.3.1.2.6 = STRING: "Microsoft Visual C++ 2019 X86 Additional Runtime - 14.27.29016"
iso.3.6.1.2.1.25.6.3.1.2.7 = STRING: "Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.27.29016"
...
```

                               *Listing 57 - Sử dụng snmpwalk để liệt kê phần mềm đã cài*

Khi kết hợp với danh sách tiến trình đang chạy mà chúng ta có trước đó, thông tin này có thể trở nên cực kỳ giá trị để đối chiếu chính xác phiên bản phần mềm mà một tiến trình đang chạy trên host mục tiêu.

Một kỹ thuật liệt kê SNMP khác là liệt kê tất cả các cổng TCP đang lắng nghe hiện tại:

```bash
kali@kali:~$ snmpwalk -c public -v1 192.168.50.151 1.3.6.1.2.1.6.13.1.3
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.88.0.0.0.0.0 = INTEGER: 88
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.135.0.0.0.0.0 = INTEGER: 135
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.389.0.0.0.0.0 = INTEGER: 389
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.445.0.0.0.0.0 = INTEGER: 445
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.464.0.0.0.0.0 = INTEGER: 464
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.593.0.0.0.0.0 = INTEGER: 593
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.636.0.0.0.0.0 = INTEGER: 636
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.3268.0.0.0.0.0 = INTEGER: 3268
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.3269.0.0.0.0.0 = INTEGER: 3269
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.5357.0.0.0.0.0 = INTEGER: 5357
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.5985.0.0.0.0.0 = INTEGER: 5985
...
```

                                   *Listing 58 - Sử dụng snmpwalk để liệt kê các cổng TCP mở*

Giá trị số nguyên trong đầu ra trên đại diện cho các cổng TCP đang lắng nghe hiện tại trên mục tiêu. Thông tin này có thể cực kỳ hữu ích vì nó có thể tiết lộ các cổng chỉ lắng nghe cục bộ và do đó hé lộ một dịch vụ mới trước đây chưa được biết đến.

---

# 4. Tổng kết

---

Trong Mô-đun này, chúng ta đã khám phá các khía cạnh nền tảng của quy trình lặp lại trong cả thu thập thông tin thụ động và chủ động. Trước hết, chúng ta đã đề cập đến nhiều kỹ thuật và công cụ để tìm thông tin về các công ty và nhân viên của họ. Những thông tin này thường có thể trở nên vô giá ở các giai đoạn sau của quá trình đánh giá. Sau đó, chúng ta tập trung vào cách chủ động quét và liệt kê các dịch vụ thường được phơi bày. Chúng ta đã học cách thực hiện các bước liệt kê này từ cả Kali Linux và một máy khách Windows.

Không bao giờ có một công cụ “tốt nhất” cho mọi tình huống, đặc biệt là khi nhiều công cụ trong Kali Linux chồng chéo về chức năng. Cách tốt nhất luôn là làm quen với càng nhiều công cụ càng tốt, học các sắc thái của chúng và bất cứ khi nào có thể, đo lường kết quả để hiểu điều gì đang diễn ra ở hậu trường. Trong một số trường hợp, công cụ “tốt nhất” chính là công cụ mà pentester quen thuộc nhất.

---

# 5. Luyện tập

---

## TryHackMe

---

[Information Gathering and Vulnerability Scanning](https://tryhackme.com/module/information-gathering-and-vulnerability-scanning)

[Nmap](https://tryhackme.com/module/nmap?utm_source=chatgpt.com)

[Shodan.io](https://tryhackme.com/room/shodan)

[OhSINT](https://tryhackme.com/room/ohsint?utm_source=chatgpt.com)

[Sakura Room](https://tryhackme.com/room/sakura?utm_source=chatgpt.com)

[Pentesting Fundamentals](https://tryhackme.com/room/pentestingfundamentals?utm_source=chatgpt.com)

[Google Dorking](https://tryhackme.com/room/googledorking?utm_source=chatgpt.com)

---

## HackTheBox

---

[Information Gathering - Web Edition Course | HTB Academy](https://academy.hackthebox.com/course/preview/information-gathering---web-edition?utm_source=chatgpt.com)

[Network Enumeration with Nmap Course | HTB Academy](https://academy.hackthebox.com/course/preview/network-enumeration-with-nmap?utm_source=chatgpt.com)

[Footprinting Course | HTB Academy](https://academy.hackthebox.com/course/preview/footprinting?utm_source=chatgpt.com)

[OSINT: Corporate Recon Course | HTB Academy](https://academy.hackthebox.com/course/preview/osint-corporate-recon?utm_source=chatgpt.com)

---

## PentesterLab

---

[PentesterLab: Learn with our Recon Badge](https://pentesterlab.com/badges/recon?utm_source=chatgpt.com)

---

## Cyberdefense

---

[Intel101 | Blue team challenge.](https://cyberdefenders.org/blueteam-ctf-challenges/intel101/?utm_source=chatgpt.com)

---

# 6. WriteUp

---

['Labs/Information Gathering' 카테고리의 글 목록](https://longhd.tistory.com/category/Labs/Information%20Gathering)