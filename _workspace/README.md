# Workspace (Private Note & Content Pipeline)

Thư mục này dùng để lưu trữ toàn bộ ý tưởng, ghi chú tự do, tài liệu nghiên cứu và hình ảnh làm chất liệu trước khi viết bài.

> **Lưu ý:** Thư mục này đã được cấu hình trong `_config.yml` (`exclude: [_workspace/]`) và có tiền tố `_`, đảm bảo Jekyll và GitHub Pages **hoàn toàn không build ra web**.

## Cấu trúc thư mục

- `01-ideas/`: Nơi brainstorm nhanh ý tưởng, bullet points, liên kết thô.
- `02-research/`: Ghi chú nghiên cứu sâu, phân tích kỹ thuật, trích dẫn, case study.
- `03-drafts/`: Các bài viết đang viết dở, chưa sẵn sàng xuất bản.
- `assets/`: Hình ảnh nháp, screenshots, sơ đồ tư duy (mindmaps) làm tư liệu.
- `templates/`: Template mẫu định dạng chuẩn cho bài viết mới.

## Quy trình xuất bản bài viết

1. Viết nháp trong `03-drafts/` (có thể dùng template trong `templates/post-template.md`).
2. Khi hoàn thiện, copy/di chuyển bài viết vào thư mục `_posts/<category>/` tương ứng với định dạng tên:
   `YYYY-MM-DD-tieu-de-khong-dau.md`
3. Nếu bài viết có dùng ảnh chính thức, di chuyển ảnh từ `_workspace/assets/` sang thư mục public (ví dụ `assets/images/...`).
