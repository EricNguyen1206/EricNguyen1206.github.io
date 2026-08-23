# Blog cá nhân — Eric Nguyen

Blog tĩnh chạy bằng **Jekyll + theme minima**, được GitHub Pages tự động build.
Không cần cài đặt gì để viết bài.

## Viết bài mới (workflow CMS)

1. Tạo file: `_posts/YYYY-MM-DD-ten-bai-viet.md`
   - Tên file **bắt buộc** đúng format `NĂM-THÁNG-NGÀY-ten-bai.md`
   - Tên bài nên dùng ASCII, gạch-ngang, không dấu (VD: `huong-dan-git.md`)
2. Đầu file phải có front matter:

   ```yaml
   ---
   layout: post
   title: "Tiêu đề bài viết"
   date: 2026-08-23 10:00:00 +0700
   categories: javascript web
   ---
   ```

3. Viết nội dung bằng Markdown (GFM: bảng, task list, strikethrough, code fence).
4. Commit + push lên `main` → GitHub Pages tự build trong ~1 phút.

Hình ảnh bỏ vào `assets/img/`, chèn bằng `![mô tả](/assets/img/ten-anh.png)`.

## Cấu trúc

- `_posts/` — bài viết (nguồn nội dung duy nhất)
- `about.md` — trang "Giới thiệu" (hiện trên menu)
- `index.md` — trang chủ, tự liệt kê bài mới nhất
- `_config.yml` — cấu hình (tên blog, theme, plugin)
- `portfolio.html` — trang portfolio cũ, truy cập tại `/portfolio.html`
- RSS: `/feed.xml`

## Xem trước ở local (tùy chọn, không bắt buộc)

```sh
gem install jekyll
jekyll serve
# mở http://localhost:4000
```

GitHub Pages vẫn build bình thường kể cả khi không bao giờ chạy local.
