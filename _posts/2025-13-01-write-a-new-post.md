---
title: Hướng dẫn viết bài mới
author: toantranct
date: 2025-01-13 14:10:00 +0700
categories: [Blogging, Hướng Dẫn]
tags: [post]
render_with_liquid: false
---

Hướng dẫn này sẽ chỉ cho bạn cách viết một bài viết trong mẫu _Chirpy_, và nó đáng để đọc ngay cả khi bạn đã sử dụng Jekyll trước đây, vì nhiều tính năng yêu cầu các biến cụ thể phải được thiết lập.

## Đặt Tên và Đường Dẫn

Tạo một tệp mới có tên `YYYY-MM-DD-TITLE.EXTENSION`{: .filepath} và đặt nó trong thư mục `_posts`{: .filepath} của thư mục gốc. Lưu ý rằng `EXTENSION`{: .filepath} phải là một trong các định dạng `md`{: .filepath} hoặc `markdown`{: .filepath}. Nếu bạn muốn tiết kiệm thời gian tạo tệp, hãy cân nhắc sử dụng plugin [`Jekyll-Compose`](https://github.com/jekyll/jekyll-compose) để thực hiện việc này.

## Front Matter

Về cơ bản, bạn cần điền [Front Matter](https://jekyllrb.com/docs/front-matter/) như sau ở đầu bài viết:

```yaml
---
title: TIÊU ĐỀ
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [DANH MỤC_CHÍNH, DANH MỤC_PHỤ]
tags: [TAG]     # Tên TAG nên luôn viết thường
---
```

> _Layout_ của bài viết đã được đặt mặc định là `post`, vì vậy không cần thêm biến _layout_ trong khối Front Matter.
{: .prompt-tip }

### Múi Giờ của Ngày

Để ghi chính xác ngày phát hành của bài viết, bạn không chỉ cần thiết lập `timezone` trong `_config.yml`{: .filepath} mà còn cần cung cấp múi giờ của bài viết trong biến `date` của khối Front Matter. Định dạng: `+/-TTTT`, ví dụ: `+0800`.

### Danh Mục và Thẻ

`categories` của mỗi bài viết được thiết kế để chứa tối đa hai phần tử, và số lượng phần tử trong `tags` có thể từ không đến vô hạn. Ví dụ:

```yaml
---
categories: [Động Vật, Côn Trùng]
tags: [ong]
---
```

### Thông Tin Tác Giả

Thông tin tác giả của bài viết thường không cần điền trong _Front Matter_, chúng sẽ được lấy từ các biến `social.name` và mục đầu tiên của `social.links` trong tệp cấu hình mặc định. Tuy nhiên, bạn cũng có thể ghi đè nó như sau:

Thêm thông tin tác giả trong `_data/authors.yml` (Nếu trang web của bạn không có tệp này, đừng ngần ngại tạo một tệp).

```yaml
<author_id>:
  name: <tên đầy đủ>
  twitter: <twitter_của_tác_giả>
  url: <trang_chủ_của_tác_giả>
```
{: file="_data/authors.yml" }

Sau đó sử dụng `author` để chỉ định một mục hoặc `authors` để chỉ định nhiều mục:

```yaml
---
author: <author_id>                     # cho một mục
# hoặc
authors: [<author1_id>, <author2_id>]   # cho nhiều mục
---
```

Nói cách khác, khóa `author` cũng có thể xác định nhiều mục.

> Lợi ích của việc đọc thông tin tác giả từ tệp `_data/authors.yml`{: .filepath } là trang sẽ có thẻ meta `twitter:creator`, giúp làm phong phú thêm [Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards/guides/getting-started#card-and-content-attribution) và tốt cho SEO.
{: .prompt-info }

### Mô Tả Bài Viết

Theo mặc định, các từ đầu tiên của bài viết được sử dụng để hiển thị trên trang chủ cho danh sách bài viết, trong phần _Đọc Thêm_ và trong XML của nguồn cấp RSS. Nếu bạn không muốn hiển thị mô tả tự động cho bài viết, bạn có thể tùy chỉnh nó bằng cách sử dụng trường `description` trong _Front Matter_ như sau:

```yaml
---
description: Tóm tắt ngắn gọn của bài viết.
---
```

Ngoài ra, văn bản `description` cũng sẽ được hiển thị dưới tiêu đề bài viết trên trang bài viết.

## Mục Lục

Theo mặc định, **M**ục **L**ục (TOC) được hiển thị ở bảng bên phải của bài viết. Nếu bạn muốn tắt nó toàn cục, hãy vào `_config.yml`{: .filepath} và đặt giá trị của biến `toc` thành `false`. Nếu bạn muốn tắt TOC cho một bài viết cụ thể, hãy thêm phần sau vào [Front Matter](https://jekyllrb.com/docs/front-matter/) của bài viết:

```yaml
---
toc: false
---
```

## Bình Luận

Cài đặt toàn cục cho bình luận được định nghĩa bởi tùy chọn `comments.provider` trong tệp `_config.yml`{: .filepath}. Khi một hệ thống bình luận được chọn cho biến này, bình luận sẽ được bật cho tất cả các bài viết.

Nếu bạn muốn tắt bình luận cho một bài viết cụ thể, hãy thêm phần sau vào **Front Matter** của bài viết:

```yaml
---
comments: false
---
```

## Phương Tiện

Chúng tôi đề cập đến hình ảnh, âm thanh và video như các tài nguyên phương tiện trong _Chirpy_.

### Tiền Tố URL

Thỉnh thoảng chúng ta phải định nghĩa tiền tố URL trùng lặp cho nhiều tài nguyên trong một bài viết, đây là một công việc nhàm chán mà bạn có thể tránh bằng cách thiết lập hai tham số.

- Nếu bạn đang sử dụng CDN để lưu trữ các tệp phương tiện, bạn có thể chỉ định `cdn` trong `_config.yml`{: .filepath }. Các URL của tài nguyên phương tiện cho avatar trang web và bài viết sau đó sẽ được thêm tiền tố bằng tên miền CDN.

  ```yaml
  cdn: https://cdn.com
  ```
  {: file='_config.yml' .nolineno }

- Để chỉ định tiền tố đường dẫn tài nguyên cho phạm vi bài viết/trang hiện tại, hãy đặt `media_subpath` trong _front matter_ của bài viết:

  ```yaml
  ---
  media_subpath: /đường/dẫn/đến/phương/tiện/
  ---
  ```
  {: .nolineno }

Tùy chọn `site.cdn` và `page.media_subpath` có thể được sử dụng riêng lẻ hoặc kết hợp để linh hoạt tạo URL tài nguyên cuối cùng: `[site.cdn/][page.media_subpath/]file.ext`

### Hình Ảnh

#### Chú Thích

Thêm chữ nghiêng vào dòng tiếp theo của hình ảnh, sau đó nó sẽ trở thành chú thích và xuất hiện ở dưới cùng của hình ảnh:

```markdown
![mô-tả-hình-ảnh](/đường/dẫn/đến/hình/ảnh)
_Chú Thích Hình Ảnh_
```
{: .nolineno}

#### Kích Thước

Để ngăn chặn việc bố cục nội dung trang bị thay đổi khi hình ảnh được tải, chúng ta nên đặt chiều rộng và chiều cao cho mỗi hình ảnh.

```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
```
{: .nolineno}

> Đối với SVG, bạn phải ít nhất chỉ định _chiều rộng_ của nó, nếu không nó sẽ không được hiển thị.
{: .prompt-info }

Bắt đầu từ _Chirpy v5.0.0_, `height` và `width` hỗ trợ viết tắt (`height` → `h`, `width` → `w`). Ví dụ sau có hiệu quả tương tự như trên:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }
```
{: .nolineno}

#### Vị Trí

Theo mặc định, hình ảnh được căn giữa, nhưng bạn có thể chỉ định vị trí bằng cách sử dụng một trong các lớp `normal`, `left`, và `right`.

> Khi vị trí được chỉ định, không nên thêm chú thích hình ảnh.
{: .prompt-warning }

- **Vị trí bình thường**

  Hình ảnh sẽ được căn trái trong mẫu dưới đây:

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .normal }
  ```
  {: .nolineno}

- **Nổi sang trái**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .left }
  ```
  {: .nolineno}

- **Nổi sang phải**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .right }
  ```
  {: .nolineno}

#### Chế Độ Tối/Sáng

Bạn có thể làm cho hình ảnh tuân theo tùy chọn chủ đề trong chế độ tối/sáng. Điều này yêu cầu bạn chuẩn bị hai hình ảnh, một cho chế độ tối và một cho chế độ sáng, sau đó gán cho chúng một lớp cụ thể (`dark` hoặc `light`):

```markdown
![Chỉ chế độ sáng](/đường/dẫn/đến/chế-độ-sáng.png){: .light }
![Chỉ chế độ tối](/đường/dẫn/đến/chế-độ-tối.png){: .dark }
```

#### Bóng Đổ

Ảnh chụp màn hình của cửa sổ chương trình có thể được xem xét để hiển thị hiệu ứng bóng đổ:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```
{: .nolineno}

#### Hình Ảnh Xem Trước

Nếu bạn muốn thêm hình ảnh ở đầu bài viết, hãy cung cấp hình ảnh có độ phân giải `1200 x 630`. Lưu ý rằng nếu tỷ lệ khung hình không đáp ứng `1.91 : 1`, hình ảnh sẽ được thu nhỏ và cắt xén.

Biết các điều kiện tiên quyết này, bạn có thể bắt đầu thiết lập thuộc tính của hình ảnh:

```yaml
---
image:
  path: /đường/dẫn/đến/hình/ảnh
  alt: văn bản thay thế hình ảnh
---
```

Lưu ý rằng [`media_subpath`](#url-prefix) cũng có thể được chuyển đến hình ảnh xem trước, tức là khi nó đã được thiết lập, thuộc tính `path` chỉ cần tên tệp hình ảnh.

Để sử dụng đơn giản, bạn cũng có thể chỉ sử dụng `image` để định nghĩa đường dẫn.

```yml
---
image: /đường/dẫn/đến/hình/ảnh
---
```

#### LQIP

Đối với hình ảnh xem trước:

```yaml
---
image:
  lqip: /đường/dẫn/đến/tệp-lqip # hoặc URI base64
---
```

> Bạn có thể quan sát LQIP trong hình ảnh xem trước của bài viết "[Văn Bản và Kiểu Chữ](../text-and-typography/)".

Đối với hình ảnh bình thường:

```markdown
![Mô tả hình ảnh](/đường/dẫn/đến/hình/ảnh){: lqip="/đường/dẫn/đến/tệp-lqip" }
```
{: .nolineno }

### Video

#### Nền Tảng Mạng Xã Hội

Bạn có thể nhúng video từ các nền tảng mạng xã hội với cú pháp sau:

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

Trong đó `Platform` là tên nền tảng viết thường, và `ID` là ID của video.

Bảng sau đây cho thấy cách lấy hai tham số chúng ta cần trong một URL video nhất định, và bạn cũng có thể biết các nền tảng video hiện được hỗ trợ.

| Video URL                                                                                          | Platform   | ID             |
| -------------------------------------------------------------------------------------------------- | ---------- | :------------- |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube`  | `H-B46URT4mg`  |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211)         | `twitch`   | `1634779211`   |
| [https://www.**bilibili**.com/video/**BV1Q44y1B7Wf**](https://www.bilibili.com/video/BV1Q44y1B7Wf) | `bilibili` | `BV1Q44y1B7Wf` |

#### Tệp Video

Nếu bạn muốn nhúng trực tiếp một tệp video, hãy sử dụng cú pháp sau:

```liquid
{% include embed/video.html src='{URL}' %}
```

Trong đó `URL` là URL đến một tệp video, ví dụ: `/đường/dẫn/đến/video.mp4`.

Bạn cũng có thể chỉ định các thuộc tính bổ sung cho video được nhúng. Dưới đây là danh sách đầy đủ các thuộc tính được phép.

- `poster='/đường/dẫn/đến/poster.png'` — hình ảnh poster cho video được hiển thị trong khi video đang tải
- `title='Văn Bản'` — tiêu đề cho video xuất hiện bên dưới video và trông giống như đối với hình ảnh
- `autoplay=true` — video tự động bắt đầu phát ngay khi có thể
- `loop=true` — tự động quay lại đầu khi kết thúc video
- `muted=true` — âm thanh sẽ bị tắt ban đầu
- `types` — chỉ định phần mở rộng của các định dạng video bổ sung được phân tách bằng `|`. Đảm bảo các tệp này tồn tại trong cùng thư mục với tệp video chính của bạn.

Xem xét một ví dụ sử dụng tất cả các thuộc tính trên:

```liquid
{%
  include embed/video.html
  src='/đường/dẫn/đến/video.mp4'
  types='ogg|mov'
  poster='poster.png'
  title='Video demo'
  autoplay=true
  loop=true
  muted=true
%}
```

### Âm Thanh

Nếu bạn muốn nhúng trực tiếp một tệp âm thanh, hãy sử dụng cú pháp sau:

```liquid
{% include embed/audio.html src='{URL}' %}
```

Trong đó `URL` là URL đến một tệp âm thanh, ví dụ: `/đường/dẫn/đến/audio.mp3`.

Bạn cũng có thể chỉ định các thuộc tính bổ sung cho âm thanh được nhúng. Dưới đây là danh sách đầy đủ các thuộc tính được phép.

- `title='Văn Bản'` — tiêu đề cho âm thanh xuất hiện bên dưới âm thanh và trông giống như đối với hình ảnh
- `types` — chỉ định phần mở rộng của các
