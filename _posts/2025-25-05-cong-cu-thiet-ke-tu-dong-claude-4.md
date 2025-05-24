---
title: Công cụ thiết kế hoàn toàn tự động lộ diện! Một prompt khiến Claude 4 tạo ra 3000 dòng code
date: 2025-05-21 14:10:00 +0700
categories: [AI, Claude]
tags: [claude, code, ai, thiết kế, prompt, ios]
author: toantranct
description: Khám phá cách sử dụng Claude 4 để tạo ra thiết kế giao diện iOS hoàn chỉnh chỉ với một prompt duy nhất, tạo ra hơn 3000 dòng code và nhiều trang giao diện cùng lúc.
media_subpath: /assets/img/posts/claude-code/
render_with_liquid: false
---



Bạn đã bao giờ trải qua điều này chưa? Trong đầu chợt lóe lên một ý tưởng sản phẩm tuyệt vời, nhưng lại khổ sở vì không có khả năng thiết kế để biến nó thành hiện thực? Hay chỉ vì một bản phác thảo giao diện đơn giản mà phải bỏ ra một số tiền lớn để thuê nhà thiết kế?

Gần đây tôi bị cuốn hút bởi một việc: **dùng Claude 4 tạo ra thiết kế giao diện iOS hoàn chỉnh chỉ trong một bước**. Sau một ngày gỡ lỗi hơn 40 phiên bản Prompt, cuối cùng tôi đã hoàn thiện được prompt thần kỳ này, để AI trực tiếp xuất ra thiết kế giao diện iOS đẹp mắt, hơn nữa còn **tạo ra nhiều trang cùng một lúc**!

## Xem thử hiệu quả trước nhé, thật sự rất tuyệt!

### Sản phẩm đầu tiên: **Cho Bạn Mã**

Ý tưởng bắt nguồn từ bài viết "Mã Đều Đến" của Đặc Công Vũ Trụ dạo trước, tôi đã cải tiến một chút và tạo ra "Cho Bạn Mã"!

![Cho Bạn Mã App](cho-ban-ma-app.webp){: width="700" height="400" }
_Giao diện ứng dụng "Cho Bạn Mã" được tạo bởi Claude 4_

### Sản phẩm thứ hai: **Ứng dụng Cầu Phúc Thi Đại Học**

Kỳ thi đại học sắp đến rồi, hãy cùng cầu phúc cho các thí sinh nhé:

![Cầu Phúc Thi Đại Học App](cau-phuc-thi-dai-hoc-app.webp){: width="700" height="400" }
_Giao diện ứng dụng "Cầu Phúc Thi Đại Học"_

Nhìn những giao diện này, bạn có thể sẽ ngạc nhiên: "Đây thật sự là do AI tạo ra sao? Chuyên nghiệp quá!" Đúng vậy, đây chính là sức mạnh của Claude 4! Và toàn bộ quá trình chỉ cần một prompt, không cần bất kỳ kiến thức nền tảng nào về thiết kế.

## Công khai toàn bộ prompt

```yaml
#Vai trò
Bạn là một Giám đốc Sản phẩm, Nhà thiết kế Tương tác, Nhà thiết kế UI dày dạn kinh nghiệm

#Nhiệm vụ
Mục tiêu cuối cùng của nhiệm vụ này là tạo ra một bộ thiết kế UI cho ứng dụng di động. Đầu tiên, tạo tệp task.md trong thư mục dự án hiện tại, liệt kê tất cả các việc cần làm. Mỗi khi hoàn thành một việc, hãy chỉnh sửa tệp task.md, sử dụng ✅ để cập nhật trạng thái hoàn thành của mục tương ứng. Hoàn thành lần lượt các việc cho đến khi tất cả các mục đều ở trạng thái hoàn thành.

Việc cần làm 1: Thiết kế chức năng sản phẩm
- Thông tin ban đầu: Tôi là trợ lý thiết kế sản phẩm của bạn, bây giờ hãy cho tôi biết bạn muốn phát triển loại sản phẩm nào nhé~
- Phân tích thông tin người dùng gửi, đặt câu hỏi làm rõ các chi tiết chưa rõ ràng
- Kết hợp các câu trả lời nhận được, mô tả chi tiết để tạo thành tệp [Tài liệu thiết kế sản phẩm.md]

Việc cần làm 2: Thiết kế tương tác
Dựa trên chức năng cuối cùng được xuất ra từ {Việc cần làm 1}, xác định tất cả các trang mà sản phẩm này bao gồm, xuất thông tin của tất cả các trang theo định dạng ví dụ dưới đây.
Định dạng ví dụ:
<Tên trang>
Công dụng: <Vai trò chính của trang>
Chức năng cốt lõi: <Liệt kê các chức năng chính của trang này>

Sau khi xuất xong thông tin của tất cả các trang, cập nhật [Tài liệu thiết kế sản phẩm.md].

Việc cần làm 3: Thiết kế UI
- Dựa trên [Tài liệu thiết kế sản phẩm.md], đồng thời tuân thủ {Phong cách thiết kế UI} và {Quy cách thiết kế UI} dưới đây, tạo tệp html độc lập cho mỗi bản thiết kế.

Sau khi xuất xong tất cả các tệp html của các trang, tạm dừng nhiệm vụ, nhắc người dùng nhập lệnh "tiếp tục".

Việc cần làm 4: Nhắc người dùng nhập lệnh "tiếp tục"

Việc cần làm 5: Tạo một tệp UI.html
- Màu nền tổng thể của trang UI.html là #f6f6f6
- Sử dụng kỹ thuật iframe để sắp xếp tất cả các trang theo dạng lưới phù hợp trong UI.html, chiều rộng của mỗi iframe cố định là 400px, chiều cao cố định là 850px, đảm bảo mỗi bản thiết kế được hiển thị đầy đủ, không bị cắt xén hoặc che khuất.
- Màu nền của iframe là ##f6f6f6

Việc cần làm 6: Tự kiểm tra code
Kiểm tra lần lượt từng tệp html của các trang,
Kiểm tra: Bắt buộc ẩn tất cả thanh cuộn
Kiểm tra: Kích thước bản thiết kế là 375x812PX
Kiểm tra: Màu nền của thanh tín hiệu và thanh tiêu đề phải nhất quán (có thể đều là trong suốt)
Kiểm tra: Cách gọi biểu tượng và các kiểu dáng khác
Kiểm tra: Thanh tab dưới cùng phải có màu trắng, độ mờ đục 100%.

Việc cần làm 7: Kiểm tra tệp UI.html
Kiểm tra toàn bộ code của tệp UI.html, xóa các yếu tố trang trí thừa bên ngoài iframe, bên trong mỗi bản thiết kế đã có sẵn code kiểu dáng mock up, iframe trong UI.html không cần kiểu dáng mock up, nếu có cũng cần xóa đi.

#Phong cách thiết kế UI
Sự cân bằng hoàn hảo giữa thẩm mỹ tối giản thanh lịch và chức năng;
Bảng màu chuyển sắc tươi mới, nhẹ nhàng hòa quyện với màu sắc thương hiệu;
Thiết kế khoảng trắng vừa phải;
Trải nghiệm nhập vai nhẹ nhàng, thông suốt;
Phân cấp thông tin được trình bày rõ ràng thông qua hiệu ứng đổ bóng tinh tế và bố cục thẻ module hóa;
Tầm nhìn người dùng có thể tập trung tự nhiên vào các chức năng cốt lõi;
Các góc bo tròn được trau chuốt tỉ mỉ;
Tương tác vi mô tinh tế;
Tỷ lệ hình ảnh thoải mái;
Khoảng cách quy chuẩn;

#Quy cách thiết kế UI
1. Kích thước bản thiết kế đơn lẻ là 375x812PX, có kiểu dáng mock up mô phỏng điện thoại.
2. Biểu tượng: Tham chiếu biểu tượng từ thư viện biểu tượng vector trực tuyến
3. Hình ảnh: Sử dụng liên kết từ trang web hình ảnh mã nguồn mở Unsplash để chèn
4. Kiểu dáng phải được hoàn thành bằng cách sử dụng thẻ <link> để nhập CDN tailwindcss
5. Thanh trạng thái hiển thị thông tin mô phỏng như thời gian, tín hiệu
6. Màu nền của thanh tín hiệu và thanh tiêu đề phải nhất quán (có thể đều là trong suốt)
7. Thanh tab dưới cùng phải có màu trắng, độ mờ đục 100%.
8. Sử dụng chiều cao cố định để tránh biến dạng vùng chứa

#Những điều cần lưu ý
1. Màu mock up chỉ được sử dụng màu đen
2. Bắt buộc ẩn tất cả thanh cuộn trong tất cả các tệp html
```

> Lưu ý: Sử dụng Cursor + Claude 4 Sonnet (không phải mô hình thinking)
{: .prompt-tip }

## Cách sử dụng prompt này?

Cách sử dụng siêu đơn giản:

1. Tạo một thư mục mới (dùng để lưu trữ các tệp thiết kế được tạo ra)
2. Dùng Cursor mở thư mục này
3. Chọn chế độ Agent, chọn mô hình **Claude 4 Sonnet** (Lưu ý: không phải mô hình Thinking)
4. Nhập prompt ở trên, nhấn Enter

Chờ một lát, sau đây tôi sẽ minh họa cho các bạn!

## Thực hành quy trình phát triển

### Bước 1: Mô tả ý tưởng sản phẩm

Sau khi nhập prompt, Cursor sẽ nhanh chóng đưa ra một vài câu hỏi để bạn xác nhận:

![Cursor Prompt Interface](cursor-prompt-interface.webp){: width="700" height="400" }

Bước này yêu cầu bạn cho prompt đa năng này biết bạn muốn làm loại sản phẩm nào! Lấy ví dụ "Cho Bạn Mã" ở trên, mô tả sản phẩm tôi nhập là:

```text
Làm một sản phẩm tên là: Cho Bạn Mã, hiện tại nhiều ứng dụng AI cần mã mời, và những "châu chấu" sản phẩm không tìm được nhưng lại muốn dùng mã mời, vì vậy có thể phát triển một nền tảng chia sẻ mã mời, để các ứng dụng AI có thể đến đây đăng mã mời, người dùng cũng có thể đăng mã mời của mình, phong cách cần ngầu một chút, những cái khác bạn với tư cách là một Giám đốc Sản phẩm AI dày dạn kinh nghiệm hãy bổ sung giúp tôi.
```

### Bước 2: Làm rõ chi tiết

Tiếp theo, Cursor sẽ tiếp tục hỏi thêm chi tiết:

![Cursor Detail Questions](cursor-detail-questions.webp){: width="700" height="400" }

Bạn có thể mô tả chi tiết yêu cầu, hoặc đơn giản như tôi, để AI tự do sáng tạo:

```text
Ứng dụng AI có thể chọn ba ví dụ: Nami AI Search, Lovart, Tiangong SkyWork, bạn có thể sử dụng tìm kiếm để lấy thông tin về ba sản phẩm này, những cái khác bạn tự quyết định.
```

### Bước 3: Tạo code tự động

Sau đó Claude sẽ bắt đầu xuất code, **một lần tạo ra hơn 3000 dòng code cũng không thành vấn đề**! Đúng vậy, Claude 3.7 chỉ có thể xuất khoảng 1000 dòng, trong khi khả năng xuất code của Claude 4 đã có bước nhảy vọt về chất.

![Claude Code Generation](claude-code-generation.webp){: width="700" height="400" }

Ở đây chúng tôi đã tạm dừng một chút để tránh mô hình xuất quá dài, chỉ cần nhập "tiếp tục", sau đó nó sẽ tiếp tục cho đến khi toàn bộ bản nháp UI được xuất xong:

![Claude UI Complete](claude-ui-complete.webp){: width="700" height="400" }

Cuối cùng, mở tệp UI.html được tạo ra, bạn sẽ thấy những giao diện tinh xảo đã được giới thiệu ở đầu bài viết này!

## Claude 4 thực sự mạnh hơn ở điểm nào?

Thực ra prompt xuất trực tiếp bản nháp thiết kế này đã được hoàn thành vào tháng 3, lúc đó dùng Claude 3.7 cũng có thể xuất được cả bộ bản nháp thiết kế, nhưng hiệu quả không ổn định. Mãi đến khi Claude 4 ra mắt, tôi mới điều chỉnh lại prompt.

Sau hơn 40 phiên bản prompt được gỡ lỗi, mới đạt được hiệu quả tương đối ổn định như hiện tại. Theo trải nghiệm cá nhân, tôi thấy **khả năng thẩm mỹ của Claude 4 Sonnet không tăng cường đáng kể**, nhưng **khả năng xuất code đã tăng gấp nhiều lần**!

### Cảm nhận trực quan nhất:

- **Mỗi lần có thể xuất hơn 3000 dòng code**, và gần như không có lỗi
- **Chất lượng code cao hơn**, HTML/CSS được tạo ra quy chuẩn và tối ưu hơn  
- **Hiểu chỉ thị chính xác hơn**, có thể tuân thủ tốt hơn các quy cách thiết kế UI

> Điều thú vị là, cùng một bộ prompt nhưng khi dùng mô hình Claude 4 Sonnet Thinking lại dễ xảy ra các lỗi khác nhau, vì vậy ở đây tôi đặc biệt khuyên dùng mô hình Claude 4 Sonnet.
{: .prompt-warning }

## Lời kết

Thông qua việc tạo thiết kế iOS chỉ bằng một cú nhấp chuột với Claude 4, chúng ta đã thấy được tiềm năng to lớn của AI trong lĩnh vực thiết kế. Nó không chỉ cải thiện đáng kể hiệu quả thiết kế mà còn đảm bảo chất lượng thiết kế, là một trợ thủ đắc lực trong quá trình phát triển sản phẩm.

Sự thành công của prompt này cũng chứng minh giá trị của kỹ thuật prompt (prompt engineering). Thông qua các prompt được thiết kế cẩn thận, chúng ta có thể hướng dẫn AI tạo ra nội dung chất lượng cao phù hợp với mong đợi, thực hiện chuyển đổi nhanh chóng từ ý tưởng đến thành phẩm.


Các bạn đã bao giờ dùng AI để thiết kế chưa? Hiệu quả thế nào? Chào mừng chia sẻ kinh nghiệm của bạn ở phần bình luận!

> **Ghi chú:** Bài viết này được dịch và chỉnh sửa từ bài gốc của AI产品黄叔. Bài gốc có thể xem tại: [https://mp.weixin.qq.com/s/6ylOzxiZ3k8nY4c0g3MWdQ](https://mp.weixin.qq.com/s/6ylOzxiZ3k8nY4c0g3MWdQ)
{: .prompt-info }