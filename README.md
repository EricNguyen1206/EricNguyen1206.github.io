# Blog cá nhân — Eric Nguyen

Blog tĩnh chạy bằng **Jekyll + theme minima**, được GitHub Pages tự động build.
Không cần cài đặt gì để viết bài.

## Categories

| Thư mục | Category | Dùng cho |
|---|---|---|
| `_posts/life/` | `life` | Đời sống, suy nghĩ, trải nghiệm |
| `_posts/programming/` | `programming` | Kỹ thuật, lập trình |
| `_posts/projects/` | `projects` | Viết về từng project cá nhân |

Thư mục chứa bài tự quyết định category — đặt bài vào đúng thư mục là xong
(URL bài viết không đổi theo thư mục). Tên category dùng ASCII, viết thường.

## Viết bài mới (workflow CMS)

1. Tạo file: `_posts/<category>/YYYY-MM-DD-ten-bai-viet.md`
   - Tên file **bắt buộc** đúng format `NĂM-THÁNG-NGÀY-ten-bai.md`
   - Tên bài nên dùng ASCII, gạch-ngang, không dấu (VD: `huong-dan-git.md`)
2. Đầu file phải có front matter:

   ```yaml
   ---
   layout: post
   title: "Tiêu đề bài viết"
   date: 2026-08-23 10:00:00 +0700
   categories: programming
   ---
   ```

   (`categories` nên khớp với thư mục — hiển thị trên bài viết và feed)

3. Viết nội dung bằng Markdown (GFM: bảng, task list, strikethrough, code fence).
4. Commit + push lên `main` → GitHub Pages tự build trong ~1 phút.

Hình ảnh bỏ vào `assets/img/`, chèn bằng `![mô tả](/assets/img/ten-anh.png)`.

## Cấu trúc

- `_posts/life|programming|projects/` — bài viết, chia theo category
- `about.md` — trang "Giới thiệu" (hiện trên menu)
- `index.md` — trang chủ (intro + danh sách bài, phân trang 5 bài)
- `_config.yml` — cấu hình (tên blog, theme, plugin)
- RSS: `/feed.xml`

## Xem trước ở local (tùy chọn, không bắt buộc)

```sh
gem install jekyll
jekyll serve
# mở http://localhost:4000
```

GitHub Pages vẫn build bình thường kể cả khi không bao giờ chạy local.
