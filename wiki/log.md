---
tieu_de: Nhật ký hoạt động
loai: tong-quan
ngay_tao: 2026-07-18
ngay_cap_nhat: 2026-07-18
nguon: []
tags:
  - nhat-ky
  - log
---

# Nhật ký hoạt động (append-only)

> Nhật ký **CHỈ-THÊM** — không sửa/xóa mục cũ, chỉ thêm mục mới xuống dưới, theo thời gian.
>
> Mỗi mục bắt đầu bằng tiền tố nhất quán để `grep` được:
> - `## [YYYY-MM-DD] ingest | Tên nguồn`
> - `## [YYYY-MM-DD] query | Nội dung truy vấn`
> - `## [YYYY-MM-DD] lint | Kết quả rà soát`
>
> Lấy 5 mục gần nhất:
> ```
> grep "^## \[" wiki/log.md | tail -5
> ```

---

## [2026-07-18] init | Khởi tạo hệ thống

Khởi tạo toàn bộ "Bộ não thứ hai" hôm nay:

- Tạo cấu trúc ba tầng: `raw/` (nguồn thô bất biến), `wiki/` (do LLM sở hữu), `production/`
  (thành phẩm theo lệnh).
- Tạo các thư mục: `wiki/thuc-the/`, `wiki/khai-niem/`, `wiki/tom-tat-nguon/`, `wiki/so-sanh/`,
  `raw/assets/`, `production/`.
- Tạo file schema `CLAUDE.md` (triết lý, ba tầng, quy ước ngôn ngữ/trang, quy trình
  Ingest/Query/Lint).
- Khởi tạo `wiki/index.md`, `wiki/log.md`, `wiki/tong-quan.md`.
- Hiện `raw/` **chưa có nguồn** nào. Sẵn sàng nạp nguồn đầu tiên.

## [2026-07-18] ingest | Bảng giá Nha khoa Thanh Triều

Nạp nguồn `Copy of BẢNG GIÁ NHA KHOA (1).xlsx` (Excel 7 sheet) vào wiki. Chạm **12 trang mới**:

- Tóm tắt nguồn: [[bang-gia-nha-khoa-thanh-trieu]].
- Thực thể: [[nha-khoa-thanh-trieu]].
- Khái niệm dịch vụ (8): [[ve-sinh-rang-nha-chu]], [[nho-rang-va-tieu-phau]], [[tram-rang]],
  [[dieu-tri-tuy-noi-nha]], [[tay-trang-va-tham-my-rang]], [[rang-su-va-dan-su]],
  [[cay-ghep-implant]], [[ham-thao-lap]].
- So sánh (2): [[so-sanh-cac-dong-tru-implant]], [[so-sanh-cac-dong-rang-su]].
- Cập nhật `index.md` và `tong-quan.md`.
- Không có mâu thuẫn (nguồn đầu tiên). Ghi chú vài ô nhập tắt cần xác nhận (800/500/600 → nghìn).
