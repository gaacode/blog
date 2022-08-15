---
layout: post
title: "ActiveRecord scope returns all records when result of the scope is nil"
description: ""
comments: true
categories: [ruby, rails]
keywords: "Rails, scope, active record"
---

ActiveRecord scopes được sử dụng rất thường xuyên trong các ứng dụng Rails. Nó thêm 1 class method để truy xuất và truy vấn các objects.

Có một hành vi bất thường nhưng được mong đợi của scope, hãy xem điều đó qua ví dụ phía dưới.

Giả sử chúng ta cần tìm bài viết được xuất bản gần đây, scope sẽ trông như sau.

```
class Post < ApplicationRecord
  scope :recent_published, -> { where(published: true).order('published_at DESC').first }
end
```

Và hiện tại trong bảng Post, chúng ta không có bài viết nào với ```published: true```. Hãy xem rails console 

```
pry(main)> Post.recent_published
  Post Load (0.5ms)  SELECT "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
  Post Load (0.3ms)  SELECT "posts".* FROM "posts"
=> [#<Post:0x00007fa0445974f0  id: 6,  title: "Post {i}",  published_at: Mon, 25 Nov 2019 22:21:28 UTC +00:00,  published: false,  created_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00,  updated_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00>, #<Post:0x00007fa044597360  id: 7,  title: "Post {i}",  published_at: Fri, 30 Jan 2015 19:24:51 UTC +00:00,  published: false,  created_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00,  updated_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00>, #<Post:0x00007fa0445971d0  id: 8,  title: "Post {i}",  published_at: Mon, 29 Jan 2018 09:54:03 UTC +00:00,  published: false,  created_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00,  updated_at: Mon, 15 Aug 2022 15:31:52 UTC +00:00>]
```

## Đây rồi!!!

Bạn hẳn đang mong đợi [], nhưng bạn đã nhận được 1 số kết quả. Hãy xem truy vấn.

```
Post Load (0.5ms)  SELECT "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
Post Load (0.3ms)  SELECT "posts".* FROM "posts"
```

Truy vấn đầu tiên trả về nil và do đó truy vấn thứ 2 được thực thi: truy vấn tất cả dữ liệu từ bảng Post.

Nhưng tại sao điều này lại xảy ra?
Để tìm câu trả lời, hãy xem mã nguồn của active-record. Dòng [comment](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/scoping/named.rb#L71) về scope trong ActiveRecord nói:

```
# If it returns +nil+ or +false+, an
# {all}[rdoc-ref:Scoping::Named::ClassMethods#all] scope is returned instead.
```

Điều này rất quan trọng, vì qua đó ta biết được rằng, scope không được trả về nil hoặc false. Nếu không nó sẽ trả về tất cá các bản ghi.

Trong trường hợp của chúng ta, ```scope :recent_published```. Tức là ```Post.where(published: true).order('published_at DESC'].first``` trả về nil và do đó truy vấn thứ 2 ```SELECT "posts".* FROM "posts"``` được thực thi để lấy tất cả dữ liệu.

Hãy cố gắng để hiểu triển khai của ```scope``` method. Hãy xem method [_exec_scope#L403](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation.rb#L403)

```
instance_exec(*args, &block) || self
```
Đây là output của ```instance_exec(*args, &block)```

```
[1] pry(#<Post::ActiveRecord_Relation>)> instance_exec(*args, &block)
  Post Load (0.4ms)  SELECT "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
=> nil
```

Cái bên trên trả về nil, và do đó self được thực thi, trả về tất cá các bản ghi.

```
[2] pry(#<Post::ActiveRecord_Relation>)> self
  Post Load (0.2ms)  SELECT "posts".* FROM "posts"
```

## Vậy giải pháp là gì?

Nếu bạn nhận thấy trong ```scope :recent_published```, chúng ta sử dụng ```first``` và đó thực sự là 1 vấn đề. first method là từ module ActiveRecord::FinderMethods.

Chúng ta có thể sử dụng ```limit``` thay cho ```first```.

```
scope :recent_published, -> { where(published: true).order('published_at DESC').limit(1) }
```
## Làm thế nào / tại sao ```limit``` hoạt động, còn ```first``` thì không?

### Câu trả lời nhanh

```limit``` trả về ActiveRecord::Relation còn ```first``` trả về phần tử đầu tiên hoặc nil.

### Chi tiết hơn 1 chút

Kiểm tra cách hoạt động của điều kiện ```||``` trong ruby với ```nil``` và ```[]```

```
nil || anything => anything #trường hợp sử dụng `first` trong scope
[]  || anything => []       #trường hợp sử dụng `limit` trong scope
```

Đây là triển khai tương tự của ```_exec_scope``` là ```instance_exec(*args, &block) || self```

Để kiểm tra thêm, chúng ta nên xem việc triển khai của ```first``` method trong [active record](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation/finder_methods.rb#L116)

```first``` method gọi [find_nth](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation/finder_methods.rb#L504)

```
def find_nth(index)
  @offsets[offset_index + index] ||= find_nth_with_limit(index, 1).first
end
```

Bên trên, ```find_nth_with_limit(index, 1)``` trả về mảng trống: ```[]```
Và ```find_nth_with_limit(index, 1).first``` tương đương ```[].first``` và trả về nil.

Trong trường hợp ```limit``` method, nó trả về object là ActiveRecord::Relation và có thể kết hợp được với các scope khác (chainable).

## Tóm tắt

Điều rất quan trọng cần lưu ý rằng, scope nhằm trả về một ActiveRecord::Relation object mà có thể kết hợp với các scope khác. Nói ngắn gọn, scope phải kết hợp được với các scope khác.

Sử dụng class method thay vì scope khi object được trả về có thể là nil hoặc false. Hoặc không phải ```<ActiveRecord :: Relation []>``` như Array, Hash, v.v.

## Tài liệu tham khảo

[https://github.com/rails/rails/issues/21882](https://github.com/rails/rails/issues/21882)

[https://sagarjunnarkar.github.io/blogs/2019/09/15/activerecord-scope/](https://sagarjunnarkar.github.io/blogs/2019/09/15/activerecord-scope/)
