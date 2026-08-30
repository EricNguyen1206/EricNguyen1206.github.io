---
layout: post
title: "Chào mừng blog mới"
date: 2026-08-23 10:00:00 +0700
categories: programming
---

Blog vừa chuyển sang Jekyll — từ giờ viết bài chỉ cần tạo một file Markdown
trong thư mục `_posts/` rồi push lên GitHub, mọi thứ còn lại tự động.

## Viết bài như thế nào?

1. Tạo file `_posts/2026-08-23-ten-bai-viet.md` (tên file phải có dạng `YYYY-MM-DD-ten-bai.md`)
2. Thêm front matter ở đầu file:

```yaml
---
layout: post
title: "Tiêu đề bài viết"
date: 2026-08-23 10:00:00 +0700
categories: javascript web
---
```

3. Viết nội dung bằng Markdown (chuẩn GFM — giống hệt khi viết trên GitHub)
4. Commit & push — GitHub Pages tự build trong ~1 phút

## Markdown hỗ trợ đầy đủ

Đây là bảng, task list và code block:

| Tính năng | Trạng thái |
|---|---|
| Bảng (tables) | ✅ |
| Task list | ✅ |
| Code highlight | ✅ |

- [x] Setup Jekyll
- [ ] Viết bài đầu tiên thật sự
- [ ] Thêm trang portfolio

```js
console.log("Hello from the new blog!");
```
