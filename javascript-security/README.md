## Javascript Security

Khi mà PHP có vấn nạn mang tên Type Juggling, thì JavaScript cũng tồn tại 1 vấn nạn mang tên Type Coercion. Thử thách lần này chính là về Type Coercion trong JavaScript.

### Challenge 1 - Goal: RCE

Vừa vào ta thấy trang cho ta chức năng tính toán các phép như cộng, trừ, nhân, chia cơ bản.

![image](https://user-images.githubusercontent.com/90561566/234295319-51bd4b0a-c51c-4085-bd30-d7edd8f2f479.png)

Thử một phép tính cơ bản: "một cộng hai bằng mấy ?"

![image](https://user-images.githubusercontent.com/90561566/234295454-ccf00b39-ef27-416f-887a-142b675ae68c.png)

Đọc source code tại client

![image](https://user-images.githubusercontent.com/90561566/234295576-23b54b34-1e66-4c73-92dc-b84d8e316800.png)

Sau khi ấn nút Submit thì đây là đoạn script sẽ được chạy ở client.

Chức năng của hàm tinh() là gửi 1 POST request tới server, gói tin này có data là object data ở dòng 5, nhờ JSON.stringify() mà javascript object được chuyển sang dạng JSON.

Đọc source code tại server

![image](https://user-images.githubusercontent.com/90561566/234295721-e80ab765-477f-46cb-a627-fcaa099ebb50.png)

Cấu trúc thường thấy của 1 server viết bằng NodeJS + Express Framework:

+ File app.js sẽ là nơi khởi động server.
+ Folder views chứa các file html để cho người dùng tương tác.
+ Folder routes sẽ chứa các hàm để xử lý các request đến từ user.

Đọc file app.js.

![image](https://user-images.githubusercontent.com/90561566/234295772-61be3303-1e8d-4e22-a71f-d92a114b0391.png)

Dòng 10, ta thấy việc xử lý các gói tin đến "/" được thực hiện bởi module indexRouter nằm trong đường dẫn tới thư mục routes.

Ta đi đến module indexRouter

![image](https://user-images.githubusercontent.com/90561566/234296236-32b574d0-5218-40f0-9bfb-dde3a518404d.png)

Dòng 24->44 chính là hàm xử lý gói tin POST mà ban nãy client gửi.

Dòng 39 là nơi 1 unsafe method mang tên eval() xuất hiện.

Dòng 26 là nơi các untrusted data xuất hiện, các data này chính là 3 tham số mà client gửi tới.

Để untrusted data đến được unsafe method thì phải qua được khối if từ dòng 29->36.

![image](https://user-images.githubusercontent.com/90561566/234296680-4dbd3ac9-7776-4768-9a9e-b5cab591b513.png)

Dòng 29, khối if sử dụng 2 lần toán tử OR, điều đó có nghĩa 3 toán hạng kia chỉ cần 1 toán hạng trả về true thì sẽ thực hiện khối if.

Để qua được khối if thì cả 3 toán hạng kia đều phải trả về false, vậy làm sao để 3 toán hạng đều trả về false.

Ta thấy trước các hàm kiemTra() đều có dấu !, vậy thì kết quả trả về của hàm kiemTra() phải trả về true.

![image](https://user-images.githubusercontent.com/90561566/234297023-6182cd7d-0135-4887-b99b-2f2699b6e014.png)

Dòng 9->16 của hàm kiemTra(), khối if của hàm này sẽ kiểm tra xem nếu tham số input có phải dạng string không ? Nếu input là string mà chứa các ký tự ko hợp lệ, ko nằm trong tham số whitelist thì lập tức hàm kiemTra() sẽ trả về false.

Nhưng nếu ở đây input không phải là string, mà lại là array thì sao ? Vậy thì có phải ta đã bypass được khối if trong hàm kiemTra(), do đó hàm kiemTra() sẽ luôn trả về true.

Có thể bạn đã biết, với POST Request ta có thể chuyển thông tin tới server thông qua JSON, và JSON còn cho phép ta truyền cả dữ liệu dạng array.

![image](https://user-images.githubusercontent.com/90561566/234300094-02292b1e-9d5e-4b0b-9ade-c5e645621866.png)

Một gói tin bình thường với thamSo1, thamSo2, phepTinh đều được truyền dưới dạng string -> hợp lệ. Kết quả = 4.

![image](https://user-images.githubusercontent.com/90561566/234300322-5e7203d7-bc86-46d9-89cf-e058a3cd35a9.png)

Một gói tin khác thường, thamSo1 lúc này được truyền dưới dạng array -> hợp lệ. Kết quả vẫn = 4.

Lý do vì sao ["2"] + 2 = 4 thì bạn tự thử nhé. Đó là vì cơ chế Type Coercion của JavaScript.

["2"] sẽ được hàm toString() convert thành 2 và 2+2=4. Không tin bạn có thể thử.

Quay trở về với hàm xử lý POST Request.

![image](https://user-images.githubusercontent.com/90561566/234300559-e905ce94-dd75-4858-bb9d-53f891e0b221.png)

Tóm lại, biến bieuthuc ở dòng 38 sẽ có dạng: `require('child_process').execSync('ls -la')` `+` `1`

Kết quả khi gửi payload trên

![image](https://user-images.githubusercontent.com/90561566/234300809-4eab6a30-b99e-49b5-abcc-8decefba5d8f.png)

Để ý thấy số 1 ở đoạn cuối phần response, chỗ views\n1 không. Đó chính là 1 đến từ thamSo2 đó.

Ở đây ta thấy có 1 file mang tên secret.txt. Ta thử dùng 'cat secret.txt' để đọc nội dung file này xem.

![image](https://user-images.githubusercontent.com/90561566/234300995-981a4c1d-3f89-4af9-bf27-7ab9fdb86a63.png)

> Flag thử thách 1: `CBJS{dc6a2318ff561b711c08fa41ae95e752}`

<details>
<summary> Giải thích payload </summary>
Trong NodeJS, module child_process là một module có sẵn khi ta cài đặt NodeJS, nó cung cấp cho ta chức năng tạo 1 tiến trình con (child process).

Cái này kiến thức về hệ điều hành, 1 phần mềm khi chạy sẽ được tính là 1 tiến trình (1 process), cái process này sẽ gọi nhiều process khác và các process khác này là các process con.

Giả sử, khi khởi động trò chơi League Of Legends, file thực thi để chạy game LoL sẽ là process cha.

Sau đó việc hiển thị champ sẽ dùng 1 process (tạm gọi là process hiển thị), tính toán sát thương sẽ sử dụng 1 process (tạm gọi là process tính toán). Process hiển thị và process tính toán sẽ là con của process cha.

Ta có 2 phương thức có thể sử dụng từ module này để RCE đó là exec() và execSync(). 2 phương thức này đều có tác dụng sinh ra 1 tiến trình để thực thi các câu lệnh được truyền vào bên trong nó.

Tuy nhiên do NodeJS chạy theo kiến trúc bất đồng bộ (Asynchronous). Nghĩa là NodeJS sẽ ưu tiên thực hiện các process khác nhẹ hơn rồi mới thực hiện các process nặng.

Do đó nếu ta truyền payload dạng require('child_process').exec('ls -la') thì process sinh ra sẽ nặng hơn và được thực thi sau. Việc này đồng nghĩa NodeJS sẽ tạm thời không thực thi nó và coi payload này chỉ là 1 object thông thường.

Vậy nên kết quả khi truyền payload require('child_process').exec('ls -la') sẽ là: "[object Object]1".

Để khắc phục, ta dùng execSync(). Lúc này 1 cờ synchronous được sinh ra để báo hiệu rằng NodeJS phải thực thi process này trước, sau khi thực thi xong mới tiếp tục đến các process khác.

Do đó khi dùng require('child_process').execSync('ls -la') thì process này sẽ được thực thi trước và với ls -la thì nó sẽ in ra các directory và file ở thư mục hiện tại. Sau đó mới thực thi đến đoạn + 1 phía sau.
</details>

### Challenge 2 - Goal: RCE

Về cơ bản thử thách 2 giống với thử thách 1, nhưng có 1 vài phần thay đổi

![image](https://user-images.githubusercontent.com/90561566/234301681-57d320b0-3cbb-44c4-8a76-9827b5b0f3f3.png)

Dòng 11, đổi route từ "/" sang "/calc".

Dòng 12, hàm tinh() đã đổi giao thức gửi dữ liệu từ POST sang GET.

![image](https://user-images.githubusercontent.com/90561566/234301993-56e4439c-45d5-4158-a4e2-7e1d332b72fb.png)

3 dữ liệu thamSo1, thamSo2, phepTinh được gửi qua GET Param.

Source code tại server

Tại file app.js, có thêm dòng 11 và 14 để khai báo module xử lý các request tới "/calc"

![image](https://user-images.githubusercontent.com/90561566/234302110-69a86060-fc55-4937-a3a4-6b08df32b28f.png)

Ta thấy module calcRouter không thay đổi gì nhiều.

![image](https://user-images.githubusercontent.com/90561566/234302363-64bf3892-847a-433c-a764-af5bdd2bce7c.png)

Tại dòng 21 thay vì nhận dữ liệu từ req.body như thử thách 1 thì nay nhận dữ liệu từ req.query.

Đến đây thì ta không thể sử dụng cách gửi dữ liệu dưới dạng array thông qua body của POST Request được. Vì đây là GET Request mà.

Tuy nhiên thì ta có 1 điểm chung giữa PHP và NodeJS. Cái tuy nhiên này rất là ...

Đó là vì 1 lý do nào đó mà các dev NodeJS cũng cho phép ta có thể truyền array thông qua GET param.

Giả sử nếu ở url bạn truyền ?thamSo1=1 thì kết quả nhận về qua dòng trên sẽ là:

```
const thamSo1 = 1;
```

thamSo1 được lưu dưới dạng là number

Tuy nhiên nếu bạn truyền ?thamSo1=1&thamSo1=3 thì kết quả nhận về sẽ là:

```
const thamSo1 = [1,3];
```

thamSo1 được lưu dưới dạng là array

Việc truyền thamSo1 khiến nó trở thành array là 1 dạng attack được gọi dưới cái tên HTTP Parameter Pollution.

Hack

Vậy thì mình sẽ truyền payload qua GET Param như sau: `/calc?thamSo1=1&thamSo1=require('child_process').execSync('ls+-la')&phepTinh=%2B&thamSo2=1`

Do truyền qua GET Param nên phải encode các ký tự khi truyền. Khoảng trắng giữa 'ls -la' phải thay bằng dấu +, dấu + ở phepTinh sẽ thay bằng %2B.

Tóm lại, biến bieuthuc sẽ có dạng: 1,require('child_process').execSync('ls -la') + 1

Kết quả khi gửi payload trên

![image](https://user-images.githubusercontent.com/90561566/234305335-ae258c70-4fe6-49ea-a513-7107b9de33bd.png)

Ở đây ta thấy có 1 file mang tên secret.txt. Ta thử dùng 'cat secret.txt' để đọc nội dung file này.

![image](https://user-images.githubusercontent.com/90561566/234305394-bb15c908-29d9-4ad5-9acb-9bfa45a45804.png)

> Flag thử thách 2: `CBJS{130e35d9d4b850aa8d720e5896975cc5}`
