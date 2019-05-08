---
layout: post
title: "[Rails] form_with, form_tag và form_for"
description: "form_tag và form_for sẽ được sớm thay bằng form_with trong tương lai. Trong bài này chúng ta sẽ cùng tìm hiểu sự khác nhau giữa form_tag, form_for và form_with và một số code ví dụ."
comments: true
keywords: "rails, form_for, form_with, form_tag"
---

## Giới thiệu
`form_tag` và `form_for` sẽ được sớm thay bằng `form_with` trong tương lai. Trong bài này chúng ta sẽ cùng tìm hiểu sự khác nhau giữa `form_tag`, `form_for` và `form_with` và một số code ví dụ.

## Một cú pháp cho tất cả
Trước đây, khi bạn muốn tạo một form nhưng không có một model cho nó, bạn sẽ sử dụng `form_tag`.
```
<%= form_tag users_path do %>
  <%= text_field_tag :email %>
  <%= submit_tag %>
<% end %>
```

Khi có một model, ta sẽ sử dụng `form_for`
```
<%= form_for @user do |form| %>
  <%= form.text_field :email %>
  <%= form.submit %>
<% end %>
```

Như bạn có thể thấy, chúng ta sử dụng **form builder field helpers** (form.text_field, form.submit) với `form_for`. Nhưng chúng ta không thể làm giống như vậy với `form_tag`.

Vì vậy, cú pháp ở `form_tag` và `form_for` là khác nhau. Trong `form_with` thì khác, chúng ta có thể sử dụng **form builder** mọi lúc.

`form_with` khi không có model
```
<%= form_with url: users_path do |form| %>
  <%= form.text_field :email %>
  <%= form.submit %>
<% end %>
```

`form_with` với một model
```
<%= form_with model: @user do |form| %>
  <%= form.text_field :email %>
  <%= form.submit %>
<% end %>
```

Khi bạn set đối số `model` thì **scope** và **url** sẽ được tự động lấy từ nó. Đây là hành vi giống với `form_for`.

## Không còn tự động tạo các id và class
> Update: `form_with` sẽ tự động tạo các thuộc tính id từ Rails 5.2, vì vậy bạn không cần chỉ định nó thủ công nữa.

`form_for` tự động tạo các thuộc tính id cho form fields, `form_tag` cũng vậy.
```
<%= form_for User.new do |form| %>
  <%= form.text_field :email %>
<% end %>
```

Code trên tạo ra:
```
<form class="new_user" id="new_user" action="/users" ...>
  ...
  <input type="text" name="user[email]" id="user_email" />
</form>
```

Với `form_with`, bạn phải chỉ định tất cả các id và class thủ công.
```
<%= form_with model: @user do |form| %>
  <%= form.text_field :name %>
  <%= form.text_field :email, id: :email, class: :email %>
<% end %>
```

Tạo ra:
```
<form action="/users" ...>
  ...
  <input type="text" name="user[name]" />
  <input id="email" class="email" type="text" name="user[email]" />  </form>
```

## Các thuộc tính id và class của form không được bọc nữa
Xưa kia (với `form_for`),
```
<%= form_for @user, html: { id: :custom_id, class: :custom_class } do |form| %>
<% end %>
```

Bây giờ (với `form_with`),
```
<%= form_with model: @user, id: :custom_id, class: :custom_class do |form| %>
<% end %>
```

## Các form fields không phải tương ứng với các model attributes
Tôi nghĩ đây là bổ sung tuyệt vời nhất giữ cho cú pháp được đồng nhất.

Trường hợp bạn muốn thêm một check box vào form tạo Post của bạn, để cho phép tác giả của bài đăng thông báo với các độc giả của mình qua email.

Check box này không thể map với một attribute trong model. Tuy nhiên, bạn vẫn có thể làm được điều đó bằng cách sử dụng FormBuilder với `form_with` ví dụ:
```
<%= form_with model: @post do |form| %>
  <%= form.text_field :author %>
  <%= form.check_box :notify_readers %>
  <%= form.submit “Create” %>
<% end %>

<form action=”/posts” accept-charset=”UTF-8" data-remote=”true” method=”post”>
  <input name=”utf8" type=”hidden” value=”✓”>
  <input type=”hidden” name=”authenticity_token” value=”…”>
  <input type=”text” name=”post[author]”>
  <input name=”post[notify_readers]” type=”hidden” value=”0"><input type=”checkbox” value=”1" name=”post[notify_readers]”>
  <input type=”submit” name=”commit” value=”Create” data-disable-with=”Create”>
</form>
```

Giá trị của check box sẽ được scope trong post. Nếu bạn không muốn nó được scope, hãy sử dụng `check_box_tag` thay vì `form.check_box` như trong ví dụ trên.

## Các form mặc định là remote
Thay đổi này thú vị nhất đối với tôi. Tất cả form được tạo bởi `form_with` sẽ được mặc định submit bởi một **XHR (Ajax)** request. Không cần chỉ định `remote: true` như bạn phải dùng trong `form_tag` và `form_for`.

Tôi thích tính năng này, bởi vì nó gửi một message. Cảm ơn Turbolinks, hầu hết tất cả các GET requests mặc định là XHR request. Và nhờ có `form_with`, điều này có thể trở thành hiện thực cho các forms.

Bạn chỉ cần thực hiện một chút công việc (tạo Javascript response template), và bạn có thể loại bỏ tất cả các lần reload toàn bộ trang.

Tuy nhiên, nếu bạn muốn vô hiệu hóa remote, bạn có thể làm điều đó với `local: true`.
```
<%= form_with model: @user, local: true %>
<% end %>
```

Bạn có biết các remote forms trong app của bạn được xử lý như thế nào không? Hãy tìm hiểu về điều đó thông qua bài viết: [Do you really need that fancy JavaScript framework?](https://m.patrikonrails.com/do-you-really-need-that-fancy-javascript-framework-e6f2531f8a38)

## Hãy sử dụng `form_with` kể từ bây giờ và đừng bao giờ nhìn lại
Bài viết này không phải là tất cả, nhưng nó đủ để bạn bắt đầu với `form_with`.

Vì vậy bạn đừng sử dụng `form_tag` và `form_for` thêm một lần nào nữa (dù sao, chúng cũng sẽ bị xóa).

Vậy nên, hãy đọc thêm [tài liệu này](https://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with) để tìm hiểu về `form_with`.

## Tài liệu tham khảo
[https://m.patrikonrails.com/rails-5-1s-form-with-vs-old-form-helpers-3a5f72a8c78a](https://m.patrikonrails.com/rails-5-1s-form-with-vs-old-form-helpers-3a5f72a8c78a)

Cảm ơn bạn đã theo dõi bài viết, rất mong nhận được những thảo luận và đóng góp của bạn.

**Write the code as a pleasure ^_^**
