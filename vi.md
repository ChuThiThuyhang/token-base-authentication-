## Giới thiệu

Xác thực trên cơ sở token là phương thức khá nổi trên các web hiện nay. Hầu hết các công ty  về web hiện nay đang sử dụng API thì token là cách tốt nhất để xử lý việc xác thực cho nhiều người dùng.
Có vài yếu tố quan trọng khi lựa chọn xác thực dựa trên token cho các ứng dụng của bạn. Các lý do chính đó là:

Server không cần cố định và có thể dễ dàng mở rộng
Luôn sẵn sàng hỗ trợ các ứng dụng di động
Dễ dàng xác thực ngay trên các ứng dụng khác
Nâng cao tính năng bảo mật

## Ai là người sử dụng cơ chế xác thực dựa trên token?

Bất kì API hay ứng dụng web mà bạn gặp phải đều có khả năng sử dụng token.. Các ứng dụng như Facebook, Twitter, Google+, Github và rất nhiều ứng dụng khác cũng sử dụng token.

Hãy thử tìm hiểu xem chính xác là nó hoạt động như thế nào?

## Tại sao token lại được sinh ra?

Trước khi chúng ta tìm hiểu xác thực dựa trên token hoạt động như nào và các lợi ích nó mang lại, chúng ta xem lại các cách xác thực cũ trong quá khứ.

### Xác thực dựa trên server (Phương thức truyền thống)

``
Vì giao thức HTTP là giao thức không lưu trạng thái, điều này có nghĩa là nếu chúng thông xác thực người dùng thông qua username và password, thì sau đó ở những lần request tiếp theo, ứng dụng của chúng tôi sẽ không thể biết chúng tôi là ai. Và chúng tôi phải xác thực lại.
``

Cách truyền thống trên ứng dụng của chúng tôi để ghi nhớ chúng ta là ai là **lưu trữ thông tin người dùng trên server**. Việc này có thể thực hiện theo vài cách khác nhau trên mỗi phiên, thường thì sẽ ở trong bộ nhớ hoặc lưu lại trên ổ đĩa.

Đây là sơ đồ thể hiện workflow xác thực dựa trên server sẽ trông như thế nào: 

![](https://cask.scotch.io/2014/11/tokens-traditional.png)

Khi web, ứng dụng và sự nổi lên của các ứng dụng di động đã xuất hiện, phương pháp xác thực này đã cho thấy các vấn đề, đặc biệt là trong khả năng mở rộng.

## Các vấn đề của xác thực dựa trên server

Một số vấn đề lớn nảy sinh với phương phức xác thực này:

**Sessions**: Với mỗi lần user xác thực xong,  server sẽ cần phải tạo một bản ghi ở đâu đó trên server của chúng tôi. Thường thi nó sẽ thực hiện trên memory và khi có rất nhiều user xác thực, nó khiến cho server phải chịu tải tăng lên.

**Khả năng mở rộng**: Vì các session được lưu trữ trong bộ nhớ, chính nó dẫn tới vấn đề về tính mở rộng. Khi các bên cung cấp dịch vụ cloud của chúng tôi bắt đầu sao chép các server để xử lý tải của ứng dụng, các thông tin quan trọng có trong session trên bộ nhớ sẽ hạn chế khả năng mở rộng quy mô. 

**CORS**: Vì chúng tôi muốn mở rộng ứng dụng của mình, cho phép dữ liệu của chúng tôi có thể truy cập được trên nhiều thiết bị di động, chúng tôi quan tâm về vấn đề chia sẻ tài nguyên gốc (CORS). Khi sử dụng AJAX để gọi các tài nguyên được chuyển từ domain khác (mobile tới các API server của chúng tôi), chúng tôi có thể gặp sự cố với các request bị cấm.
   
**CSRF** Chúng tôi cũng sẽ bảo vệ tấn công qua giả mạo yêu cầu ( CSRF ). Người dùng cũng có thể bị tấn công CSRF vì khi họ đã xác thực với một trang web như ngân hàng và điều này có thể bị lợi dụng 
khi truy cập những trang khác.
Với những vấn đề này, khả năng mở rộng là vấn đề chính, bạn nên thử một cách tiếp cận khác.

## Cơ sở Token hoạt động như thế nào

Xác thực dựa trên token là một  phương thức phi trạng thái.  Chúng tôi không lưu trữ bất kỳ thông tin nào về người dùng của chúng tôi trên máy chủ hoặc trong một session.

Quan điểm này quan tâm tới nhiều vấn đề với việc phải lưu trữ thông tin lên server một cách độc lập.
```
Không có thông tin session nào có nghĩa là ứng dụng của bạn có thể mở rộng quy mô và thêm nhiều máy hơn nếu cần mà không phải lo lắng về nơi người dùng đăng nhập.
```

Mặc dù việc triển khai này có thể thay đổi được, nhưng nó vẫn có các phần chính như sau:

1. Sử dụng Request Access thông qua username/password.
2. Xác thực thông tin đăng nhập qua ứng dụng
3.  ứng dụng cung cấp một token đã xác thực cho client
4. Client lưu trữ lại token này và gửi nó trong mỗi request
5. Server xác thực token và gửi trả lại dữ liệu

**Với mỗi một request độc lập đều phải có token**. Token này có thể được gửi thông qua HTTP header vì chúng tôi giữ tư tưởng về các request HTTP không cố định.  Chúng tôi cũng cần phải thiết lập máy chủ của chúng tôi để chấp nhận yêu cầu từ tất cả các tên miền khác bằng cách sử dụng `` Access-Control-Allow-Origin``: \ *. Điều thú vị về việc chỉ định \ * trong header của ACAO là nó không cho phép các yêu cầu cung cấp thông tin xác thực như xác thực HTTP, chứng chỉ SSL phía máy khách hoặc cookie.

Đây là 1 bản mô tả để giải thích về quá trình này:

![](https://cask.scotch.io/2014/11/tokens-new.png)

Khi chúng tôi đã xác thực thông tin của mình và chúng tôi có token, chúng tôi có thể làm nhiều thứ với token đó.

chúng tôi thậm chí có thể tạo ra các quyền dựa trên token và gửi nó thông qua các bên thứ 3 (như là một ứng dụng mới cho mobile mà chúng tôi muốn sử dụng), và chúng tôi sẽ được truy cập vào dữ liệu của mình -- **nhưng chỉ có các thông tin mà chúng tôi cho phép với token đặc biệt này**.

## Lợi ích mà token mang lại

### Không cố định và có thể mở rộng

![](https://cask.scotch.io/2014/11/infinity.jpg)

Các token được lưu trữ ở phía client. Nó hoàn toàn không xác định được và có thể mở rộng dễ dàng. Các hệ thống cân bằng tải của chúng tôi cho phép chuyển các user tới bất kì server nào vì nó không cố định hoặc lưu các thông tin về session ở bất cứ đâu. 

Nếu chúng tôi giữ thông tin về session của người dùng đã đăng nhập, nó sẽ yêu cầu chúng tôi phải gửi user đó đến các server mà họ đã đăng nhập vào. (gọi là quan hệ giữa các session)

Việc này dẫn đến một vấn đề, có nhiều người dùng buộc phải sử dụng cùng một máy chủ và nó khiến cho traffic lớn hơn.
Đừng lo lắng! Những vấn đề này đã được giải quyết bằng token vì bản thân các token đã giữ dữ liệu cho người dùng đó.

### Bảo mật

![](https://cask.scotch.io/2014/11/you-shall-not-pass.jpg)

Token, nó không phải là cookie, nó được gửi trong mỗi request trong khi đó cookie thì không được gửi. điều này giúp ngăn được các tấn công thông qua CSRF. Ngay cả khi triển khai cụ thể của bạn lưu trữ token trong một cookie ở phía client, cookie vẫn chỉ là một cơ chế lưu trữ thay vì một cơ chế xác thực.

Token cũng hết hạn sau một khoảng thời gian, vì vậy người dùng bắt buộc phải login lại. Điều này giúp họ luôn an toàn. Ngoài ra còn có khái niệm thu hồi token cho phép chúng tôi vô hiệu hóa token cụ thể và thậm chí là một nhóm token dựa trên cùng một khoản cấp phép.

### Khả năng mở rộng (Bạn của bạn và các quyền)

![](https://cask.scotch.io/2014/11/share-candy.jpg)

Tokens sẽ cho phép chúng ta xây dựng các ứng dụng chia sẻ quyền với nhau. Ví dụ, chúng tôi đã liên kết các tài khoản xã hội ngẫu nhiên với những tài khoản chính của chúng tôi như Facebook hoặc Twitter.
Khi chúng tôi đăng nhập vào Twitter thông qua một service (giả sử như là Buffer), chúng tôi đang cho phép Buffer đăng lên Twitter stream của chúng tôi.

Bằng cách sử dụng token, đây là cách mà chúng tôi **cung cấp việc lựa chọn quyền cho ứng dụng bên thứ 3**. Chúng tôi có thể xây dựng các API của riêng mình và cung cấp các token với quyền đặc  biệt nếu người dùng của bạn cho phép truy cập dữ liệu của họ trên một ứng dụng khác. 

### Đa nền tảng và tên miền

Chúng tôi đã nói một chút về CORS trước đó. Khi ứng dụng và dịch vụ của chúng tôi mở rộng, chúng tôi sẽ cần cung cấp quyền truy cập vào tất cả các loại thiết bị và ứng dụng (vì ứng dụng của chúng tôi chắc chắn sẽ trở nên phổ biến!).

Khi API của chúng tôi chỉ phân phối dữ liệu, chúng tôi cũng có thể đưa ra lựa chọn thiết kế để phân phát nội dung từ CDN. Điều này giúp loại bỏ các vấn đề mà CORS mang lại sau khi chúng tôi thiết lập cấu hình tiêu đề nhanh cho ứng dụng của chúng tôi.

```

Access-Control-Allow-Origin: *

```
Dữ liệu và tài nguyên của chúng tôi sẵn có cho các request từ bất kỳ domain nào ngay bây giờ ** miễn là người dùng có token hợp lệ **.

## Dựa trên các tiêu chuẩn 

Khi tạo token, bạn có một vài tùy chọn. Chúng ta sẽ đi sâu hơn vào chủ đề này khi chúng ta cần bảo mật một API trong một bài viết tiếp theo, nhưng tiêu chuẩn để sử dụng sẽ là JSON Web Tokens.

Biểu đồ thư viện và trình debug tiện dụng này hiển thị hỗ trợ cho các token JSON Web. Bạn có thể thấy rằng nó có một số lượng hỗ trợ lớn trên nhiều ngôn ngữ. Điều này có nghĩa là bạn có thể thực sự chuyển đổi cơ chế xác thực của mình nếu bạn chọn làm như vậy trong tương lai!

## Kết luận
Đây chỉ là xem xét cách thức và lý do tại sao nên dùng xác thực dựa trên cơ chế token. Như mọi khi trong thế giới an ninh, có nhiều, nhiều, nhiều, nhiều (quá nhiều?) Nhiều hơn cho mỗi chủ đề và nó thay đổi theo từng trường hợp sử dụng. Thậm chí, chúng tôi còn nghiên cứu một số chủ đề về khả năng mở rộng cũng xứng đáng với cuộc trò chuyện của riêng họ.

Đây là một  phần xem lại nhanh, vì vậy xin vui lòng đưa ra bất cứ điều gì bạn đã bỏ lỡ hoặc bất kỳ câu hỏi nào bạn có về vấn đề này.
In our next article, we'll be looking at the ``anatomy of JSON Web Tokens``. For full code examples on how to authenticate a Node API using JSON Web Tokens, check out our book ``MEAN Machine``.
Trong bài viết tiếp theo của chúng tôi, chúng tôi sẽ xem xét giải thuật ``phân tích JSON Web Tokens``. Để biết đầy đủ các code ví dụ về cách xác thực API Node bằng cách sử dụng JSON Web Tokens, hãy xem cuốn sách của chúng tôi `` MEAN Machine``.

Nguồn: [link](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication)
