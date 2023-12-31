# **File upload vulnerabilities**

## **Lỗ hổng file upload là gì?**

<p align="center">
    <img src="./img_file/1.png" style="width: 600px">
</p>

- Lỗ hổng file upload được tạo ra khi máy chủ web cho phép người dùng tải tệp lên hệ thống của nó mà không xác thực đầy đủ như tên, loại tệp và nội dung kích thước.

- Tác động của file_upload:

  - Trong trường hợp xấu nhất, loại tệp không được xác thực đúng cách và cấu hình máy chủ cho phép một số loại tệp nhất định (chẳng hạn như .php và .jsp) được thực thi dưới dạng mã. Trong trường hợp này, kẻ tấn công có khả năng có thể tải lên tệp mã phía máy chủ  cấp cho chúng toàn quyền kiểm soát máy chủ một cách hiệu quả.

  - Nếu tên tệp không được xác thực hợp lệ, điều này có thể cho phép kẻ tấn công ghi đè lên các tệp quan trọng chỉ bằng cách tải tệp có cùng tên lên. Nếu máy chủ cũng dễ bị tấn công khi duyệt thư mục, điều này có thể có nghĩa là những kẻ tấn công thậm chí có thể tải tệp lên các vị trí không lường trước được.
  
  - Việc không đảm bảo rằng kích thước của tệp nằm trong ngưỡng dự kiến cũng có thể kích hoạt một dạng tấn công từ chối dịch vụ (DoS).

> ### **Exploiting unrestricted file uploads to deploy a web shell**

- Nếu bạn có thể tải lên thành công web shell, bạn thực sự có toàn quyền kiểm soát máy chủ. Điều này có nghĩa là bạn có thể đọc và ghi các tệp tùy ý, lọc dữ liệu nhạy cảm, thậm chí sử dụng máy chủ để xoay vòng các cuộc tấn công chống lại cả cơ sở hạ tầng nội bộ và các máy chủ khác bên ngoài mạng.
- **Remote code execution via web shell upload**
  - Tôi đã demo lại một đoạn code sau về ví dụ minh họa:

  ```php
  <?php    
    if (isset($_POST["submit"])) {
        $file_dir = "uploads/";
        $file_name = $_FILES["file"]["name"];
        $file_location = $file_dir . basename($file_name);
        if (file_exists($file_location)) {
            echo "This image already exists!";
            exit();
        } else {
            if (move_uploaded_file($_FILES["file"]["tmp_name"], $file_location)) {
                echo "<pre>{$file_name} succesfully uploaded at </pre><a href='" . $file_location . "'>here</a>";
            } else {
                echo "<pre>Your image was not uploaded!</pre>";
            }
        }
    }

    ?>
  ```

  - Sau khi mở lap lên ta đã thấy một chức năng file upload. Tôi thử tải lên một tập php.

    <p align="center">
    <img src="./img_file/2.png" style="width: 600px">
    </p>

  - Sau khi tôi post một ảnh của tôi lên thì web đã upload file ảnh của tôi lên.

    <p align="center">
    <img src="./img_file/4.png" style="width: 600px">
    </p>

  - Tôi đã thử upload file php với nội dung như sau:

    ```php
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```

  - Hàm `file_get_contents` có chức năng đọc nội dung file thành một chuỗi.
  - Sau khi upload lên tôi đã đọc được nội dung của `/home/carlos/secret`:

    <p align="center">
    <img src="./img_file/5.png" style="width: 600px">
    </p>

> ### **Exploiting flawed validation of file uploads**

- **Web shell upload via Content-Type restriction bypass**
  - Bypass lỗ hổng File upload bằng header Content-Type:

  ```php
  $file_content_type = $_FILES["file"]["type"];

    if (!in_array($file_content_type, array("image/png", "image/jpg", "image/jpeg"))) {
        die("Only allowed png, jpg, jpeg file!");
    }
  ```

  - Đối với lap này tôi đã thử với file php như trên:

    <p align="center">
    <img src="./img_file/6.png" style="width: 600px">
    </p>

  - Nó đã báo file không được chấp nhận và nó chỉ chấp nhận file ảnh png và jpeg. Và bản chất của việc xác thực file này nó sẽ nhằm tới header : `Content-Type: application/octet-stream`

  - Vậy tôi chỉ cần sửa header về `Content-Type: image/jpeg`

    <p align="center">
    <img src="./img_file/7.png" style="width: 600px">
    </p>

> ### **Preventing file execution in user-accessible directories**

- **Web shell upload via path traversal**
  - Đôi khi, hệ thống cài đặt thư mục lưu trữ các tệp do người dùng tải lên không có quyền thực thi. Đây là một cách ngăn chặn tốt, tuy nhiên, kẻ tấn công vẫn có thể tìm kiếm sự "may mắn" ở các thư mục khác bằng cách kết hợp với kỹ thuật path traversal.

  - Đối với lab này tôi cũng thử với php như trên:

   <p align="center">
    <img src="./img_file/8.png" style="width: 600px">
    </p>

  - File đã được upload thành công nhưng nó không được thực thi. Có lẽ nó không có quyền thực thi file trong thư mục. Tôi đã thử đổi đường dẫn file qua kỹ thuật path traversal. `Content-Disposition: form-data; name="avatar"; filename="../atacker.php"`

  <p align="center">
    <img src="./img_file/9.png" style="width: 600px">
    </p>

- **Bypass qua file signature - chữ ký tệp tin**
  
  - Chữ ký tệp (file signature) là một chuỗi xác định các byte duy nhất được ghi vào tiêu đề của tệp. Trên hệ thống Windows, chữ ký tệp thường được chứa trong 20 byte đầu tiên của tệp. Các loại tệp khác nhau sẽ có các chữ ký tệp khác nhau, ví dụ: các tệp JPEG luôn bắt đầu bằng các byte FF D8 FF.

  - `exif_imagetype()` thực hiện cơ chế này:

  ```php
  <?php
    $file_tmp_name = $_FILES["file"]["tmp_name"];
    if (!exif_imagetype($file_tmp_name)) {
        die("Your upload file is not an image!");
    }
    ?>
  ```

   <p align="center">
    <img src="./img_file/10.png" style="width: 600px">
    </p>

  - Lưu ý rằng cách hoạt động của hàm exif_imagetype() là đọc các bytes đầu tiên của file upload và xác định kết quả có phải một tệp hình ảnh hay không dựa vào signature. Bởi vậy chúng ta có thể bypass cơ chế bảo vệ này bằng cách thêm GIF89a trong phần bắt đầu nội dung file, khi trang web kiểm tra ảnh bằng hàm exif_imagetype() sẽ xác định đây là một tệp GIF - thuộc một trong các định dạng ảnh quy định, khi đó hệ thống cho phép người dùng tải lên "hình ảnh" này. Thật vậy:

   <p align="center">
    <img src="./img_file/11.png" style="width: 600px">
    </p>

  - Xét một ví dụ khác xử lý phần mở rộng file:

  ```php
  $file_ext = pathinfo($file_name, PATHINFO_EXTENSION);
    $black_list = ['php', 'Php', 'pHp', 'phP', 'PHp', 'PhP', 'pHP', 'PHP'];

    if (in_array($file_ext, $black_list)) {
        return('Invalid extention!');
    }
  ```

> - **Web shell upload via obfuscated file extension**

  <p align="center">
    <img src="./img_file/12.png" style="width: 600px">
    </p>
  - Hệ thống chỉ cho phép file dạng JPG và PNG. Tất cả các cách bypass phía trên đều không hiệu quả, chúng ta cần nghĩ cách vượt qua black list hệ thống. Trong trường hợp này chúng ta có thể sử dụng ký tự null byte %00 để vượt qua cơ chế này (Lưu ý rằng bắt đầu từ phiên bản PHP 5.3.4, ký tự Null byte đã được fix). Sử dụng tên file shell.php%.png phần mở rộng trang web nhận được là .png không nằm trong black list, sau khi xử lý thì các ký tự bắt đầu từ ký tự Null byte được loại bỏ, tên file chỉ còn shell.php có thể thực thi.

  <p align="center">
    <img src="./img_file/13.png" style="width: 600px">
    </p>

> - **Web shell upload via extension blacklist bypass**

- Hệ thống ngăn chặn người dùng tải lên file dạng php. Tất cả các cách bypass phía trên đều không hiệu quả. Lưu ý rằng chúng ta có thể tải lên nhiều file ảnh với cùng tên, và khi truy cập tới đường dẫn file upload sẽ hiển thị hình ảnh cuối cùng upload (các hình ảnh đã upload có cùng tên). Như vậy khả năng lớn hệ thống sẽ thay thế ảnh cuối cùng vào vị trí ảnh trước đó có cùng tên. Điều này dẫn đến chúng ta có thể ghi đè nội dung các file đã có.

- Chúng ta nhắm tới file .htaccess - là một file có ở thư mục gốc của các hostting và do apache quản lý, cấp quyền. File .htaccess có thể điều khiển, cấu hình được nhiều yếu tố với đa dạng các thông số, nó có khả năng thay đổi các giá trị được set mặc định của Apache.

- Upload một file với tên .htaccess, thay đổi header Content-Type thành giá trị text/plain, nội dung file như sau:

  ```
  AddType application/x-httpd-php .viblo
  ```

  <p align="center">
    <img src="./img_file/14.png" style="width: 600px">
    </p>

  <p align="center">
    <img src="./img_file/15.png" style="width: 600px">
    </p>
> - **Bypass bằng cách chèn metadata trong file ảnh**
  <p align="center">
    <img src="./img_file/16.png" style="width: 600px">
    </p>
  - Tuy nhiên, khi đổi phần mở rộng một ảnh bình thường, chẳng hạn từ origin.jpg thành origin.php và upload file này sẽ được phép.
  <p align="center">
    <img src="./img_file/17.png" style="width: 600px">
    </p>
  - Từ đây, có thể suy đoán: hệ thống thực hiện kiểm tra tệp tải lên có thỏa mãn các tính chất của một file ảnh hay không, tuy nhiên không kiểm tra phần mở rộng file cũng như Content type. Nghĩa là nếu khi chúng ta bypass được cơ chế này upload một file php thành công, hệ thống có thể sẽ thực thi file php đó. Ý tưởng là chúng ta có thể thực hiện chèn metadata vào file ảnh, đồng thời đổi tên với phần mở rộng .php để hệ thống thực thi. Sử dụng công cụ Exiftool, payload như sau:

  ```
  exiftool -Comment="<?php echo 'VIBLO ' . file_get_contents('/home/carlos/secret') . ' VIBLO'; ?>" origin.png -o origin.php
  ```

  <p align="center">
    <img src="./img_file/18.png" style="width: 600px">
    </p>
  - Ta đã có một file png được chèn mã php và có đuôi file php.
  - Từ đó ta sẽ có thể gửi payload thành công cho trang web và thực hiện tấn công.

>- **Khai thác lỗ hổng file upload race conditions**

- Chức năng upload file hiện nay đều có khả năng chống lại các cuộc tấn công file upload tốt. Thay vì trực tiếp lưu file vào thư mục, chúng ta có thể đặt file upload vào một nơi "tạm thời", thay đổi tên file, xử lý qua các bước kiểm tra, khi xác định đó là một tệp thực sự an toàn thì mới lưu vào thư mục chính.

- Tuy nhiên, việc xử lý tệp cần thời gian nhất định. Từ đó, kẻ tấn công có thể up file mới. Và file mới sẽ trong hàng chờ để xử lý file cũ. Tận dụng thời gian đó ta sẽ có thể khai thác được.

- Mình có code xử lý file upload như sau:

  ```php
  <?php
      $target_dir = "avatars/";
      $target_file = $target_dir . $_FILES["avatar"]["name"];

      // Lưu trữ file tạm thời.
      move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);
      // checkViruses() và checkFileType() để kiểm tra tính an toàn của file upload.
      if (checkViruses($target_file) && checkFileType($target_file)) {
          echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
      } else {
          unlink($target_file);
          echo "Sorry, there was an error uploading your file.";
          http_response_code(403);
      }

      function checkViruses($fileName) {
          // checking for viruses
          ...
      }

      function checkFileType($fileName) {
          $imageFileType = strtolower(pathinfo($fileName,PATHINFO_EXTENSION));
          if($imageFileType != "jpg" && $imageFileType != "png") {
              echo "Sorry, only JPG & PNG files are allowed\n";
              return false;
          } else {
              return true;
          }
      }
  ?>

  ```

- Qua đó, Tôi sẽ thực hiện 2 request up file.php và trong khoảng thời gian file chưa kịp xóa thì tôi sẽ thực hiện lấy file để payload được thực thi.

  <p align="center">
    <img src="./img_file/19.png" style="width: 600px">
    </p>
  <p align="center">
    <img src="./img_file/20.png" style="width: 600px">
    </p>

## **Các phương pháp ngăn ngừa lỗ hổng file upload**

- Không cho quyền thực các tệp trong thư mục lưu trữ file upload từ người dùng: Kể cả khi người dùng upload file tấn công thành công nhưng không thể thực thi sẽ giảm đi phần lớn nguy cơ mang lại từ các tệp độc hại này.

- Kết hợp nhiều cơ chế ngăn chặn đảm bảo an toàn đối với các file upload: Có thể kết hợp white list, kiểm tra nội dung file, kiểm tra tên file có đúng định dạng, không chứa ký tự không mong muốn, ...

- Thay đổi tên tệp do người dùng tải lên kết hợp che giấu đường dẫn thư mục: Một số trường hợp người dùng bypass upload file thành công và truy cập tới tệp upload, nhưng không thể thực thi đã tên đã bị thay đổi thành các chuỗi ký tự ngẫu nhiên.

- Thường xuyên thực hiện rà soát, tái kiểm tra các tệp tin trong hệ thống.

- Máy chủ web cần được cấu hình hợp lý, phân quyền thư mục phù hợp, nâng cấp, cập nhật các phiên bản phần mềm, công nghệ kịp thời.
