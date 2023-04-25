## File uploads

Thử thách lần này là về lỗ hổng File Upload, được xây dựng bởi team CyberJutsu, nó bao gồm tất cả 9 levels mà theo mình thấy thì được sắp xếp từ dễ cho đến khó, ban đầu thì đơn giản nhưng càng về sau thì bạn càng phải xâu chuỗi những kiến thức mà bạn có được thì mới giải được nó.

### Thử thách 1 (Đề bài: RCE)

Vừa vào, ta thấy trang cung cấp cho chức năng upload file. Mình sẽ thử upload 1 file mang tên test.txt, bên trong mình soạn 2 dòng "hello world" để xem sao.

![image](https://user-images.githubusercontent.com/90561566/234287666-2b542d7e-e85e-4fb8-9b35-03644fad3d3f.png)

Ok vậy theo như dòng trên thì mình được biết là mình upload file thành công. Mình ấn thử vào đường link màu xanh thì thấy:

![image](https://user-images.githubusercontent.com/90561566/234287744-58f64e2f-0557-4a4a-a857-b0904f06e17d.png)

Ồ, nó hiện ra dòng chữ "hello world" này.

Debug source:

![image](https://user-images.githubusercontent.com/90561566/234287792-86eb11cb-27cc-4482-ae05-14dcd21ea7f0.png)

Ta thấy 2 dòng này không có đoạn filter nào. Do vậy chúng ta có thể upload 1 file .php có chứa mã để RCE.

Soạn và gửi 1 file mang tên hack.php với nội dung sau:

![image](https://user-images.githubusercontent.com/90561566/234287841-af0672dc-ff95-4b35-9a91-0d94d6db4dd0.png)

Đi tới đường link chứa file .hack.php ta thấy:

![image](https://user-images.githubusercontent.com/90561566/234287880-7fcfcbad-b5bb-4d99-8183-88df7efdb290.png)

Ok vậy là chạy được hàm phpinfo() thành công, phiên bản php trên server là 7.3.30.

Ta nên test thử có RCE bằng hàm phpinfo() thay vì các hàm nguy hiểm như system(). Bởi rất có server chứa firewall, antivirus sẽ chặn không cho chạy các hàm nhạy cảm như vậy.

Mình spoil trước luôn là toàn bộ các thử thách lần này đều có thể chạy hàm system() mà ko bị chặn :V

Soạn và gửi file hack.php với nội dung sau để xem toàn bộ file và directory nằm ở thư mục root:

![image](https://user-images.githubusercontent.com/90561566/234288070-dfc02469-855d-4eb0-aabc-8918b9657b64.png)

Ồ, ta thấy chạy được system() và nó trả về toàn bộ file và directory nằm ở thư mục root này.

![image](https://user-images.githubusercontent.com/90561566/234288122-c5d8df26-f6a6-4321-b6b4-fdff81cd9f5c.png)

Flag nằm trong file 71c99ec9c94-secret.txt.

Dùng lệnh cat để đọc nội dung 1 file chỉ định:

![image](https://user-images.githubusercontent.com/90561566/234288160-e4207a72-0326-45c1-a829-8334196e2833.png)

> Flag thử thách 1: `CBJS{why-php-run-what?}`

### Thử thách 2 (Đề bài: RCE)

Có thể thấy thử thách 2 đã filter tên file

![image](https://user-images.githubusercontent.com/90561566/234288283-4ccd0b99-b251-445d-a6c2-fc8b2f4f30c5.png)

Dòng 2, hàm explode() đã giúp tách tên của file thành các phần khác nhau, các phần ngăn cách nhau bằng dấu "."

Kết quả trả về của hàm explode() là 1 array.

Nếu tên file là hack.php thì nó sẽ tách thành 1 array như sau array["hack", "php"], trong đó array[0] mang giá trị "hack", array[1] mang giá trị "php".

Dòng 3, Kiểm tra xem array[1] có trùng giá trị với "php" không, nếu trùng thì Hack detected

Cách bypass của mình là mình sẽ để tên file là hack.abc.php.

Dòng 2, đi qua hàm explode() thì nó sẽ thành array["hack","abc","php"] và array[1] có giá trị "abc" !== "php". Vậy ta bypass thành công.

Soạn nội dung file hack.abc.php như sau:

![image](https://user-images.githubusercontent.com/90561566/234288451-ef55f536-953e-49e9-aa9d-60277484995a.png)

Tiếp tục làm như thử thách 1 mình làm, kết quả trả về:

![image](https://user-images.githubusercontent.com/90561566/234288542-b4449bb7-821e-409b-8d21-a2b590c978b9.png)

Flag nằm trong file 1ad7e7cd851-secret.txt

> Flag thử thách 2: `CBJS{wr0nGlY_ImplEm3nt}`

### Thử thách 3 (Đề bài: RCE)

Có thể thấy thử thách 3 đã thay đổi đoạn filter đuôi file bằng đoạn này:

![image](https://user-images.githubusercontent.com/90561566/234288789-f5866620-fb9e-4d1c-b835-3939b9beb0ea.png)

Lần này có thêm hàm end() bọc bên ngoài explode() nên lúc này thì cái giá trị mà biến $extension trả về sẽ luôn là giá trị cuối của cái array mà mình phân tích phía trên.

Nghĩa là bạn đặt tên file là hack.abc.xyz.php thì cái $extension sẽ luôn trả về là php và bị phát hiện là hack.

Tuy nhiên mình biết được 1 file php không nhất thiết phải là .php mà có thể là các đuôi khác như .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc.

Đoạn này thử vài đuôi tệp thì mình thấy có 2 đuôi hợp lệ là .phar và .phtml (thật ra bạn không muốn thử nhiều thì ấn sang debug ở thử thách 4 cũng có gợi ý đó :v)

Soạn file hack.phar với nội dung:

![image](https://user-images.githubusercontent.com/90561566/234288907-a257caf5-4ba6-417a-8087-7b3742eab539.png)

Tiến hành làm như các bài trước, ta thấy:

![image](https://user-images.githubusercontent.com/90561566/234288959-7049ec89-1063-4d41-bcce-06ed7bd97fea.png)

Flag nằm trong file 1ad7e7cd851-secret.txt

> Flag thử thách 3: `CBJS{bl4ck_list?}`

### Thử thách 4 (Đề bài: RCE)

Có thể thấy thử thách 4 đã thay đổi cách filter:

![image](https://user-images.githubusercontent.com/90561566/234289404-7a59a3bd-85b1-4301-9a05-954742294c47.png)

Dòng 4, filter nốt 2 đuôi .phar và .phtml bằng cách sử dụng hàm in_array().

Hàm in_array() nhận 2 tham số (tham số 1 là giá trị của một phần tử, tham số 2 là một mảng), hàm này sẽ duyệt xem giá trị của phần tử kia có nằm trong mảng kia ko.

Vậy thì nếu $extension có giá trị nằm trong 3 đuôi ["php", "phtml", "phar"] thì đều bị chặn.

Nhưng bài này lại cho ta đọc được cấu hình của Apache.

Vậy trong cấu hình Apache có một đoạn khá "hay ho", đó chính là đoạn này:

![image](https://user-images.githubusercontent.com/90561566/234289469-8ec21a80-e073-4687-9583-cfb44fe2ebee.png)

Thoạt nhìn qua trông file này giống mấy file XML/HTML phải không ?

Dòng 1, Khai báo tag Directory kèm attribute /var/www là đường dẫn tới directory.

Nằm bên trong tag Directory là các đoạn cấu hình mà mình tạm gọi là các luật lệ mà directory đó phải thực hiện, các luật lệ đó là: Options Indexes FollowSymLinks, AllowOverride All, Require all granted

Chú ý đến đoạn AllowOverride All, đoạn này có nghĩa là ta có thể sử dụng file .htaccess nằm trong thư mục /var/www để ghi đè cấu hình của Apache.

Bạn có nhớ tại video 2 của CBJS thì để thực thi được các file .php mỗi khi có request tới thì apache phải sử dụng module libapache2-mod-php.

Để lấy ra module libapache2-mod-php và bảo nó thực thi file có đuôi .php thì tại file apache2.conf nằm ở đường dẫn /etc/apache2/apache2.conf thường sẽ có 2 dòng sau:

![image](https://user-images.githubusercontent.com/90561566/234289585-6b37f5a4-ba69-464a-baec-15f1beaac0c3.png)

LoadModule cho phép bạn thêm module libapache2-mod-php. AddType cho phép các file có đuôi .php ánh xạ với 1 loại MIMETYPE - MIMETYPE này chính là application/x-httpd-php.

Có nghĩa là bạn có thể cho phép 1 đuôi bất kì như .abc được ánh xạ như 1 file php bình thường.

Ta biết: .htaccess cho phép ghi đè apache config, AddType chỉ định apache sẽ thực thi các file .php.

Vậy thì xâu chuỗi 2 thứ đó lại, ta có được 1 cách bypass, đó chính là sử dụng .htaccess để ghi đè cấu hình AddType của apache.

Mình sẽ gửi file .htaccess với nội dung sau để cho phép các file có đuôi .abc cũng được ánh xạ như 1 file php bình thường, do đó có thể thực thi được file đuôi .abc như 1 file .php:

![image](https://user-images.githubusercontent.com/90561566/234289718-35d3501f-6684-4c1e-94f0-1bd9cb0857a9.png)

Mình sử dụng phần mềm Burp Suite để hỗ trợ gửi file .htaccess:

![image](https://user-images.githubusercontent.com/90561566/234289940-01c2bef4-bd2e-4144-9d4b-1c3a2c485672.png)

![image](https://user-images.githubusercontent.com/90561566/234289963-0081a51f-681a-4564-971b-9ee2c2cd7575.png)

Khi gửi gói tin chứa file .htaccess thì mình đã sửa 3 phần:

Dòng 16, sửa trường filename thành .htaccess

Dòng 17, sửa Content-Type thành text/plain

Dòng 19, mình đổi nội dung file

Sau khi gửi gói tin chứa file .htaccess thì response nhận được là:

![image](https://user-images.githubusercontent.com/90561566/234290021-4348a5f0-a0f4-4cf7-9345-336ce814cc87.png)

Vậy là mình đã thành công gửi file .htaccess. Vậy thì giờ để kiểm chứng, mình sẽ soạn 1 file hack.abc với nội dung bên dưới để gửi lên server

![image](https://user-images.githubusercontent.com/90561566/234290415-d3b3a910-5bc4-4914-93a1-e069b211b049.png)

Response nhận được là:

![image](https://user-images.githubusercontent.com/90561566/234290462-e620f423-5d17-47f9-99e1-bdb63c1e3267.png)

Thành công gửi file hack.abc, mình truy cập theo đường dẫn tới file hack.abc và thấy:

![image](https://user-images.githubusercontent.com/90561566/234290500-abda2a5f-fc52-453b-b6b1-275d76c80bcc.png)

Flag nằm trong file fead248f338-secret.txt:

> Flag thử thách 4: `CBJS{so_magic_I_wondeR_what_about_other_system?}`

### Thử thách 5 (Đề bài: RCE)

Tiếp tục thay đổi cách filter:

![image](https://user-images.githubusercontent.com/90561566/234291022-c4bd05c7-2523-4e0c-9ba3-18898e165d23.png)

Bài 5 ko cho ta xem file cấu hình apache nữa, nếu như bài 4 ta có thể bypass bằng cách lợi dụng tệp .htaccess để gửi lên 1 file tuy có đuôi khác nhưng MIMETYPE của nó vẫn có thể được thực thi như 1 file php.

Ở bài này lọc luôn MIMETYPE bằng hàm in_array(), lúc này nếu như MIMETYPE không nằm trong 3 giá trị ["image/jpeg", "image/png", "image/gif"] thì sẽ bị phát hiện hack.

Mình thấy bài này bypass còn dễ hơn bài trước, mình vẫn sẽ gửi 1 file hack.php nhưng mình sẽ đổi MIMETYPE của nó thành 1 trong 3 giá trị bên trên. Trông request nó sẽ như này:

![image](https://user-images.githubusercontent.com/90561566/234291133-325927e5-7f29-42ab-a7ad-32dfabfb8159.png)

Dòng 17 mình sửa Content-Type thành image/png. Ấn gửi request và response trả về là:

![image](https://user-images.githubusercontent.com/90561566/234291202-d1f7e2db-9c91-4fc3-bae5-ffaca1b0c250.png)

Đơn giản phải không? Truy cập theo đường dẫn ta thấy:

![image](https://user-images.githubusercontent.com/90561566/234291258-85b344b5-5f7b-47ad-8772-c72f1cb75ded.png)

Flag nằm trong file fead248f338-secret.txt:

> Flag thử thách 5: `CBJS{why_you_check_with_useR_input}`

### Thử thách 6 (Đề bài: RCE)

Lúc vào thử thách 6, bạn có thể thấy dòng chữ I checked the wrong way, I've just fixed it, hope I dont have bug anymore không?

Liệu bài này đã check đúng cách chưa, cùng xem đoạn code đã được thay đổi:

![image](https://user-images.githubusercontent.com/90561566/234291405-574d3d62-171f-4d44-906c-605e23eaa760.png)

Dòng 1, hàm finfo_open() với tham số FILEINFO_MIME_TYPE để lấy ra magic_database.

Dòng 2, hàm finfo_file() được dùng để kiểm tra MIMETYPE của file vừa được upload. Việc kiểm tra này được thực hiện bằng cách so sánh file signature (hay còn gọi là chữ ký đầu tệp của file vừa được upload với file signature nằm trong magic_database.

Dòng 3 và 4, ta thấy chỉ có 3 dạng MIMETYPE được chấp nhận là "image/jpeg", "image/png", "image/gif".

Hàm finfo_file() chỉ check MIMETYPE bằng các ký tự đầu tệp, cho nên các ký tự phía sau sẽ không được check. Từ đó ta có thể thêm code của mình đằng sau để bypass cơ chế check này.

Mình dùng tool Exiftool để tạo ra 1 file có ký tự đầu tệp giống các file .jpg nhưng các ký tự sau chứa code của mình.

![image](https://user-images.githubusercontent.com/90561566/234291579-8e34a6da-f4c0-4a20-b136-42dc1c3b99fe.png)

Tiến hành gửi file hack.php

![image](https://user-images.githubusercontent.com/90561566/234291619-6c812fe2-758e-4aa0-877e-72bac6559517.png)

Gửi thành công, đi tới đường dẫn chứa file ta thấy:

![image](https://user-images.githubusercontent.com/90561566/234291663-00fb1d9a-0a33-435b-a1b8-c59835d00a4e.png)

Flag nằm trong file 414ed63690-secret.txt:

![image](https://user-images.githubusercontent.com/90561566/234291705-ba991b54-f956-4db3-bfc8-03ba5bda003b.png)

> Flag thử thách 6: `CBJS{ch3ck_mag1c_bite_iz_tragic}`

### Thử thách 7 (Đề bài: RCE)

Truy cập vào thư thách 7, ta thấy dòng chữ CHANGELOG: From this challenge onwards, we have configured apache securely, you can read the config if you like:. Có vẻ như bài này cấu hình apache đã an toàn.

Mình giải thích vì sao cấu hình apache ở bài này an toàn. Đọc thử file cấu hình apache trước.

Ta thấy có 2 đoạn config đáng chủ ý, đó là:

```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

và

```
<Directory "/usr/upload/">
        AllowOverride None
        Require all granted
        <FilesMatch ".*">
                SetHandler None
        </FilesMatch>
        Header set Content-Type application/octet-stream
        <FilesMatch ".+\.jpg$">
                Header set Content-Type image/jpeg
        </FilesMatch>
        <FilesMatch ".+\.png$">
                Header set Content-Type image/png
        </FilesMatch>
        <FilesMatch ".+\.(html|txt|php)">
                Header set Content-Type text/plain
        </FilesMatch>
</Directory>
```

Dòng AllowOverride đã trở về None, vậy ta loại trừ đi khả năng có thể sử dụng cách ghi đè cấu hình bằng .htaccess.

Đoạn mã PHP và HTML cũng được thay đổi

![image](https://user-images.githubusercontent.com/90561566/234292453-cb3536a8-de24-4196-bea8-8020ce36a362.png)

![image](https://user-images.githubusercontent.com/90561566/234292465-99f27d72-d477-4ba5-a895-79b64b8e5943.png)

Trước tiên nói đến phần mã PHP đã được thay đổi, có các phần đáng chú ý như đoạn code khai báo thêm 2 biến là $cmd và $debug tại dòng 13 và dòng 16, đoạn kiểm tra nằm trong khối try..catch cũng đã được thay đổi.

Tạm gọi đây là đoạn code mang tên "thư mục upload file 1"

```
 if (!isset($_SESSION['dir'])) {
      $_SESSION['dir'] = '/usr/upload/' . session_id();
 }
$dir = $_SESSION['dir'];
$newFile = $dir . "/" . $_FILES["file"]["name"];
move_uploaded_file($_FILES["file"]["tmp_name"], $newFile);
```

Đoạn "thư mục upload file 1" ra ngay trên mô tả cách mà file mới được lưu, lúc này nó sẽ được lưu vào thư mục /usr/upload/tên_session_id/tên_file.

Tạm gọi đây là đoạn code mang tên "thư mục upload file 2"

```
$user_dir = substr($dir, 5);
$success = 'Successfully uploaded and unzip files into ' . $user_dir . '/' . $_FILES["file"]["name"];
```

Đoạn "thư mục upload file 2" này nhằm mục đích in ra cho người dùng thông báo về nơi lưu trữ file đã được upload. Bạn thấy như mình giải thích thì file được upload sẽ được lưu vào /usr/upload/tên_session_id/tên_file, tuy nhiên trong file 000-default.conf có ghi:

```
# CHANGELOG: if request to /upload/* then serve /usr/upload/*
Alias "/upload/" "/usr/upload/"
```

nghĩa là toàn bộ các request tới /upload sẽ lấy ra các file trong thư mục /usr/upload để đưa lên cho bạn xem.

Vậy nên thông dòng thông báo ở "thư mục upload file 2" sử dụng hàm substr() nhằm cắt phần /usr/ đi và in ra thông báo nơi chứa thư mục là bắt nguồn từ /upload/tên_session_id/tên_file.

Tuy nhiên file thật vẫn năm trong /usr/upload/tên_session_id/tên_file.

Chỗ mình giải thích kia chỉ là phần mở rộng để bạn hiểu thêm về code đang làm gì.

Vì đa phần các bạn sẽ nghĩ theo hướng vẫn có thể upload file .php chứa mã cho phép RCE như thử thách 1.

Nhưng hãy nhìn lại đoạn cấu hình apache đã được thêm như sau:

```
<Directory "/usr/upload/">
        AllowOverride None
        Require all granted
        <FilesMatch ".*">
                SetHandler None
        </FilesMatch>
        Header set Content-Type application/octet-stream
        <FilesMatch ".+\.jpg$">
                Header set Content-Type image/jpeg
        </FilesMatch>
        <FilesMatch ".+\.png$">
                Header set Content-Type image/png
        </FilesMatch>
        <FilesMatch ".+\.(html|txt|php)">
                Header set Content-Type text/plain
        </FilesMatch>
</Directory>
```

Riêng dòng này cho ta thấy, mọi request tới file đều sẽ không được thực thi:

```
<FilesMatch ".*">
       SetHandler None
</FilesMatch>
```

Muốn tải lên file .htaccess để ghi đè thì đã có dòng này chặn:

```
AllowOverride None
```

Hack

Dòng 26 là nơi mà một unsafe method là hàm shell_exec() được gọi.

Tham số mà hàm shell_exec() nhận lại đến từ một untrusted data là biến $cmd. Truy về nguồn gốc tạo nên biến $cmd ta thấy nó tạo từ biến $newFile ở dòng 21, tại đây biến $_FILES["file"]["name"] chính là user input hay một unstrusted data

Mình sẽ truyền giá trị ;sleep 5; cho biến $_FILES["file"]["name"] để test 5 giây sau server có trả về response không.

![image](https://user-images.githubusercontent.com/90561566/234293294-6817f763-4d47-46d4-a574-4ad8307d6b85.png)

Kết quả là sau 5 giây server mới trả về response, thành công RCE .Vậy mình sẽ tạo payload như sau:

![image](https://user-images.githubusercontent.com/90561566/234293333-79964df5-1bb1-4c58-aae8-8e14522812c7.png)

Nói qua 1 một chút về đoạn payload thì mình sử dụng dấu ; để tạo thành nhiều command nối liên tục trong linux.

Vì thư mục mà mình gửi request tới là /var/www/html nên mình sử dụng lệnh cd .. 3 lần để trở về thư mục root, sau đó dùng lệnh ls - la để hiển thị các file và directory.

Dòng 26, kết quả chạy của hàm shell_exec() được lưu vào biến $debug.

Dòng 81, echo cho phép in ra kết quả của biến $debug.

Ta thấy response trả về là toàn bộ file và directory của thư mục root (cảm ơn biến $debug):

![image](https://user-images.githubusercontent.com/90561566/234293395-fd65d065-50f7-4674-afbf-48e0bac80229.png)

Đổi payload thành ;cd ..;cd ..;cd ..;cat cf2f5cab2-secret.txt; để đọc nội dung file cf2f5cab2-secret.txt, ta được:

> Flag thử thách 7: `CBJS{w0w_s0_buggy_filename}`

### Thử thách 8 (Đề bài: đọc file /etc/passwd)

Debug source:

![image](https://user-images.githubusercontent.com/90561566/234293534-85e6f15d-e8b3-43c1-9cec-29bb4aa71861.png)

Dòng 6, biến $game có giá trị là biến $GET['game'], như vậy $game là 1 untrusted data.

Dòng 27, chứa biểu thức include cho phép evaluates một file được chỉ định, như vậy include là 1 unsafe method.

Kết hợp 2 thứ này lại cho phép ta evaluates một file bất kì trên hệ thống.

Đề bài bảo ta cần đọc file /etc/passwd. Ta kết hợp 1 loại attack mang tên Path Traversal - loại tấn công cho phép ta di chuyển qua lại giữa các thư mục trên hệ thống.

Thư mục của file hiện tại nằm ở /var/www/html/views, vậy để tới được /etc/passwd thì ta dùng payload sau: /../../../../etc/passwd

Thành công evaluates được file /etc/passwd

![image](https://user-images.githubusercontent.com/90561566/234293608-f989aaaf-df25-435b-bb04-effc8efb4ba5.png)

Việc đọc được file vừa xong được gọi là Local File Inclusion (LFI) - 1 loại attack mà hacker đánh lừa ứng dụng web để có thể đọc và thực thi một file bất kì trên hệ thống.

> Flag thử thách 8: `CBJS{baby_LFI}`

### Thử thách 9 (Đề bài: RCE)

Vẫn là một cái game như bài 8, nhưng có thêm phần Hall Of Fame (ghi danh) và Profile (upload avatar).

Trong phần /game.php vẫn dính lỗi LFI tương tự thử thách 8, vậy payload của mình như sau: /game.php?game=/../../../../etc/passwd

![image](https://user-images.githubusercontent.com/90561566/234293798-ea153a1e-fa6b-48f8-84be-bb4ae550a761.png)

> Flag thử thách 9: `CBJS{LFI+FileUpload=Bomb}`

Vì đề bài là RCE nên đọc được file /etc/passwd chưa phải kết thúc, tác giả còn để lại 1 đường link để tham khảo thêm về lỗ hổng này

Từ cái flag ta có thể thấy tác giả đã gợi ý về việc kết hợp LFI và File Upload.

Ở đây ý tưởng của mình là upload file chứa mã để RCE nhờ chức năng upload avatar ở phần Profile. Sau đó sử dụng LFI để evaluates file này.

Đầu tiên, soạn và upload file với nội dung:

![image](https://user-images.githubusercontent.com/90561566/234293932-b9411675-fdba-4b91-bbcb-faff210a3e18.png)

Đoạn code này cho ta biết file của chúng ta luôn được lưu tại /usr/upload/tên_session/ và với cái tên cố định là avatar.jpg. Giả sử tên_session của mình là sheon thì nó luôn được lưu tại /usr/upload/sheon/avatar.jpg

![image](https://user-images.githubusercontent.com/90561566/234293976-8e855e7e-4ea9-4771-8fe7-ab40c453697d.png)

Sau đó, sử dụng LFI để evaluates file vừa upload lên. Sử dụng payload sau: /game.php?game=/../../../../usr/upload/sheon/avatar.jpg

Thành công evaluates file usr/upload/sheon/avatar.jpg

![image](https://user-images.githubusercontent.com/90561566/234294013-e45865c5-5cb5-4f67-aa18-d74866515560.png)

Response cho thấy RCE thành công, việc của mình là chỉ cần truy cập vào database và chỉnh data cho lên bảng điểm số thôi.

![image](https://user-images.githubusercontent.com/90561566/234294303-f902c8d4-e8be-497a-b89f-332cfd47237f.png)
