# **Insecure deserialization**

![](./img_ID/Screenshot%202023-07-03%20100612.png)

## **Serialization là gì?**

- Serialization là quá trình chuyển đổi cấu trúc dữ liệu phức tạp, chẳng hạn như các `objects and their fields`, sang định dạng "flatter" để có thể gửi và nhận dưới dạng luồng byte tuần tự. Tuần tự hóa dữ liệu làm cho nó đơn giản hơn nhiều để:
  - Ghi dữ liệu phức tạp vào bộ nhớ liên tiến trình, tệp hoặc cơ sở dữ liệu
  - Gửi dữ liệu phức tạp, chẳng hạn như qua mạng, giữa các thành phần khác nhau của ứng dụng hoặc trong lệnh gọi API
- Crucially, when serializing an object, its state is also persisted. In other words, the object's attributes are preserved, along with their assigned values.

## **Serialization vs deserialization**

- Deserialization là quá trình khôi phục luồng byte này thành một bản sao đầy đủ chức năng của đối tượng ban đầu, ở trạng thái chính xác như khi nó được tuần tự hóa. Sau đó, logic của trang web có thể tương tác với đối tượng được Deserialization này, giống như với bất kỳ đối tượng nào khác.

![](./img_ID/Screenshot%202023-07-03%20101220.png)

## **Insecure deserialization là gì?**

- Insecure deserialization là khi dữ liệu do người dùng kiểm soát được giải tuần tự hóa bởi một trang web. Điều này có khả năng cho phép kẻ tấn công thao túng các đối tượng được tuần tự hóa để chuyển dữ liệu có hại vào mã ứng dụng.

- Thậm chí có thể thay thế một đối tượng được tuần tự hóa bằng một đối tượng của một lớp hoàn toàn khác. Đáng báo động là các đối tượng của bất kỳ lớp nào có sẵn trên trang web sẽ được giải tuần tự hóa và khởi tạo, bất kể lớp nào được mong đợi. Vì lý do này, insecure deserialization đôi khi được gọi là lỗ hổng "object injection".

> `Tác động lỗ hổng Insecure deserialization:`

- Tác động của Insecure deserialization có thể rất nghiêm trọng vì nó cung cấp một điểm vào cho bề mặt tấn công gia tăng ồ ạt. Nó cho phép kẻ tấn công sử dụng lại mã ứng dụng hiện có theo những cách có hại, dẫn đến nhiều lỗ hổng khác, thường là thực thi mã từ xa.

- Ngay cả trong trường hợp không thể thực thi mã từ xa, quá trình giải tuần tự hóa không an toàn có thể dẫn đến leo thang đặc quyền, truy cập tệp tùy ý và tấn công từ chối dịch vụ.

## **Exploit insecure deserialization vulnerabilities**

> `PHP serialization format`

- PHP sử dụng định dạng human-readable như con người có thể đọc được, với các chữ cái biểu thị kiểu dữ liệu và các số biểu thị độ dài của mỗi mục nhập. Ví dụ: hãy xem xét một đối tượng Người dùng với các thuộc tính:

```php
$user->name = "carlos";
$user->isLoggedIn = true;
```

Khi được tuần tự hóa ( `serialized` ), đối tượng này có thể trông giống như thế này:

```php
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

- Các phương thức để tuần tự hóa PHP là serialize() và giải tuần tự hóa là unserialize(). Nếu bạn có quyền truy cập mã nguồn, bạn nên bắt đầu bằng cách tìm kiếm unserialize() ở bất kỳ đâu trong mã và điều tra thêm.
![](./img_ID/Screenshot%202023-07-03%20153205.png)

> `Java serialization format`

- Một số ngôn ngữ, chẳng hạn như Java, sử dụng các định dạng tuần tự hóa nhị phân. Điều này khó đọc hơn, nhưng bạn vẫn có thể xác định dữ liệu được đăng nhiều kỳ nếu bạn biết cách nhận ra một vài dấu hiệu nhận biết. Ví dụ: các đối tượng Java được tuần tự hóa luôn bắt đầu bằng cùng một byte, được mã hóa thành `ac ed ở dạng thập lục phân` và `rO0 ở dạng Base64`.
- Bất kỳ lớp nào triển khai giao diện java.io.Serializable đều có thể được tuần tự hóa và giải tuần tự hóa. Nếu bạn có quyền truy cập mã nguồn, hãy lưu ý bất kỳ mã nào sử dụng phương thức readObject(), được sử dụng để đọc và giải tuần tự hóa dữ liệu từ InputStream.

## **Manipulating serialized objects**

- Khai thác một số lỗ hổng deserialization có thể dễ dàng như thay đổi một thuộc tính trong một đối tượng được tuần tự hóa. Khi trạng thái đối tượng được duy trì, bạn có thể nghiên cứu dữ liệu được tuần tự hóa để xác định và chỉnh sửa các giá trị thuộc tính thú vị. Sau đó, bạn có thể chuyển đối tượng độc hại vào trang web thông qua quá trình deserialization của nó. Đây là bước đầu tiên cho một khai thác deserialization cơ bản.

- Nói chung, có hai cách tiếp cận bạn có thể thực hiện khi thao tác với các đối tượng được tuần tự hóa. Bạn có thể chỉnh sửa đối tượng trực tiếp ở dạng luồng byte của nó hoặc bạn có thể viết một tập lệnh ngắn bằng ngôn ngữ tương ứng để tự tạo và sắp xếp theo thứ tự đối tượng mới. Cách tiếp cận thứ hai thường dễ dàng hơn khi làm việc với các định dạng tuần tự hóa nhị phân.

> `Modifying object attributes`

- Khi giả mạo dữ liệu, miễn là kẻ tấn công bảo tồn một đối tượng tuần tự hóa hợp lệ, quá trình giải tuần tự hóa sẽ tạo một đối tượng phía máy chủ với các giá trị thuộc tính đã sửa đổi.

Ví dụ đơn giản, hãy xem xét một trang web sử dụng Object User được tuần tự hóa để lưu trữ dữ liệu về phiên của người dùng trong cookie. Nếu kẻ tấn công phát hiện đối tượng được tuần tự hóa này trong một yêu cầu HTTP, chúng có thể giải mã nó để tìm luồng byte sau:

```
O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}
```

> Lab: Modifying serialized objects

- Ở trang web này sau khi tôi đăng nhập với tài khoản `wiener:peter` tôi nhận được cookie `sesion` như sau:

![](./img_ID/Screenshot%202023-07-03%20191804.png)

- Tôi thực hiện decode bằng base64 và tôi nhận thấy rằng đây có khả năng sẽ tân công thành công insecure deserialization:

![](./img_ID/Screenshot%202023-07-03%20192012.png)

- Tôi thực hiện sửa admin thành true và thực hiện encode rồi thay vào biến session cookie để xem liệu tôi có tấn công thành công hay không:

![](./img_ID/Screenshot%202023-07-03%20193038.png)

>`Modifying data types`

- Trong PHP sẽ cố gắng chuyển đổi chuỗi thành một số nguyên, nghĩa là 5 == "5" đánh giá là đúng.
- Trong trường hợp này, PHP sẽ chuyển đổi hiệu quả toàn bộ chuỗi thành giá trị số nguyên dựa trên số ban đầu. Phần còn lại của chuỗi bị bỏ qua hoàn toàn. Do đó, `5 == "5 of something"`trong thực tế được coi là `5 == 5`.

```php
0 == "Example string" // true
```

- Ví dụ:

```php
$login = unserialize($_COOKIE)
if ($login['password'] == $password) {
// log in successfully
}
```

- Giả sử kẻ tấn công đã sửa đổi thuộc tính mật khẩu để nó chứa số nguyên 0 thay vì chuỗi dự kiến. Miễn là mật khẩu được lưu trữ không bắt đầu bằng một số, điều kiện sẽ luôn trả về true, cho phép bỏ qua xác thực. Lưu ý rằng điều này chỉ có thể thực hiện được vì quá trình deserialization bảo tồn kiểu dữ liệu. Nếu mã lấy trực tiếp mật khẩu từ yêu cầu, thì 0 sẽ được chuyển đổi thành một chuỗi và điều kiện sẽ được đánh giá là sai.

> Lab: Modifying serialized data types

- Tương tự trang web trên nhưng cách thực hiện tấn công ở đây là không phải thay đổi thuộc tính object mà mình sẽ thay đổi data types:

- Ta nhận cooike sau khi decode như sau:
`O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"m6qd7nf91do8i2n339xy6u8jqmmwe8oe";}`

- Ta sửa data types như sau:`O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}`

## **Using application functionality**

- Ngoài việc chỉ kiểm tra các giá trị thuộc tính, chức năng của trang web cũng có thể thực hiện các thao tác nguy hiểm đối với dữ liệu từ một đối tượng được giải tuần tự hóa. Trong trường hợp này, bạn có thể sử dụng giải tuần tự hóa không an toàn để chuyển dữ liệu không mong muốn và tận dụng chức năng liên quan để gây thiệt hại.

> Lab: Using application functionality to exploit insecure deserialization

- Ở trang web này có chức năng xóa tài khoản, dường như nó sẽ xóa tất cả thông tin liên quan đến tài khoản:

![](./img_ID/Screenshot%202023-07-03%20204056.png)

- Tôi thực hiện tấn công bằng thay đổi như sau và quả thực trang web đã xóa những thông tin mà tôi thay đổi:`O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"tdc91fdaz6421i3o9sqpgz8cf7v1zxqh";s:11:"avatar_link";s:23:"/home/carlos/morale.txt"}`

## **Injecting arbitrary objects**

- Nếu kẻ tấn công có thể thao túng lớp đối tượng nào đang được truyền vào dưới dạng dữ liệu tuần tự hóa, thì chúng có thể ảnh hưởng đến mã nào được thực thi sau đó và thậm chí trong quá trình giải tuần tự hóa.

- Nếu kẻ tấn công có quyền truy cập vào mã nguồn, họ có thể nghiên cứu chi tiết tất cả các lớp có sẵn. Để xây dựng một khai thác đơn giản, họ sẽ tìm kiếm các lớp chứa các phương thức ma thuật deserialization, sau đó kiểm tra xem có bất kỳ lớp nào trong số chúng thực hiện các thao tác nguy hiểm trên dữ liệu có thể kiểm soát được hay không. Sau đó, kẻ tấn công có thể chuyển vào một đối tượng được tuần tự hóa của lớp này để sử dụng phương thức ma thuật của nó để khai thác.

> Arbitrary object injection in PHP

- Ở trang web sau khi thực hiện scan directory tôi phát hiện ra có một đường đẫn đến file giúp tôi đọc được nôi dung source:

![](./img_ID/Screenshot%202023-07-03%20210207.png)

![](./img_ID/Screenshot%202023-07-03%20210422.png)

- Với nội dung source như sau:

```php
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>

Giải thích từng phần:

1. `private $template_file_path;` và `private $lock_file_path;`: Đây là hai thuộc tính (biến thành viên) riêng tư của lớp `CustomTemplate`. `$template_file_path` chứa đường dẫn tới tệp mẫu và `$lock_file_path` chứa đường dẫn tới tệp khóa (lock file) tương ứng.

2. `public function __construct($template_file_path) { ... }`: Đây là phương thức khởi tạo của lớp `CustomTemplate` được gọi khi tạo một đối tượng từ lớp này. Phương thức này nhận một tham số `$template_file_path` đại diện cho đường dẫn tới tệp mẫu. Trong phương thức khởi tạo, các thuộc tính `$template_file_path` và `$lock_file_path` được khởi tạo với các giá trị tương ứng.

3. `private function isTemplateLocked() { ... }`: Đây là một phương thức riêng tư (private) trong lớp `CustomTemplate`. Phương thức này kiểm tra xem tệp khóa có tồn tại hay không. Nếu tệp khóa tồn tại, tức là mẫu đang bị khóa.

4. `public function getTemplate() { ... }`: Đây là một phương thức công khai (public) trong lớp `CustomTemplate`. Phương thức này trả về nội dung của tệp mẫu bằng cách sử dụng hàm `file_get_contents()`.

5. `public function saveTemplate($template) { ... }`: Đây là một phương thức công khai (public) trong lớp `CustomTemplate`. Phương thức này được sử dụng để lưu nội dung mẫu mới. Trước khi lưu, phương thức kiểm tra xem mẫu có bị khóa hay không bằng cách gọi phương thức `isTemplateLocked()`. Nếu mẫu không bị khóa, phương thức sẽ ghi nội dung mới vào tệp mẫu bằng cách sử dụng hàm `file_put_contents()`. Nếu ghi tệp không thành công, ngoại lệ (Exception) sẽ được ném.

6. `function __destruct() { ... }`: Đây là phương thức hủy (destructor) của lớp `CustomTemplate`. Phương thức này được gọi khi một đối tượng từ lớp này bị hủy. Trong phương thức hủy này, nếu tệp khóa tồn tại, nó sẽ được xóa bằng cách sử

 dụng hàm `unlink()`.
```

- Từ đấy, tôi thực hiện tấn công như sau:

![](./img_ID/Screenshot%202023-07-03%20212216.png)

## **Một số tool và tiện ích giúp mình tấn công deserialization vulnerabilities**

- [**`ysoserial`**](https://github.com/frohoff/ysoserial)
- [**`"PHP Generic Gadget Chains" (PHPGGC)`**](https://github.com/ambionics/phpggc)

## **Cách phòng chống insecure deserialization vulnerabilities**

- Tránh deserialization đầu vào của người dùng trừ khi thực sự cần thiết. Mức độ nghiêm trọng cao của các hoạt động khai thác mà nó có khả năng kích hoạt và khó khăn trong việc bảo vệ chống lại chúng, lớn hơn lợi ích trong nhiều trường hợp.

- Nếu bạn cần giải tuần tự hóa dữ liệu từ các nguồn không đáng tin cậy, hãy kết hợp các biện pháp mạnh mẽ để đảm bảo rằng dữ liệu không bị giả mạo. Ví dụ: bạn có thể triển khai chữ ký điện tử để kiểm tra tính toàn vẹn của dữ liệu.
Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==
