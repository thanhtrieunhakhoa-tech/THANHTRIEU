# CLAUDE.md — Schema & Cẩm nang "Bộ não thứ hai"

> Đây là file cấu hình then chốt của vault. Mọi phiên làm việc (session) đọc file này
> TRƯỚC TIÊN để hiểu cách tổ chức, quy ước và quy trình. Đọc kỹ trước khi động vào wiki.

---

## 1. TRIẾT LÝ

Hệ thống này **KHÔNG** hoạt động theo kiểu RAG (Retrieval-Augmented Generation — sinh câu
trả lời bằng cách truy xuất lại tài liệu thô mỗi lần hỏi). Thay vào đó, nó **DUY TRÌ MỘT
WIKI** liên kết chéo, tích lũy dần theo thời gian.

Nguyên tắc cốt lõi:

- Mỗi khi nạp một nguồn mới, ta **ĐỌC** nó, **TRÍCH** thông tin then chốt, rồi **TÍCH HỢP**
  vào wiki hiện có: cập nhật trang thực thể, chỉnh tóm tắt chủ đề, ghi chú mâu thuẫn với dữ
  liệu cũ.
- Kiến thức được **biên dịch MỘT LẦN** rồi giữ cập nhật — **không suy lại từ đầu mỗi lần hỏi**.
- Wiki là **tài sản bền vững**, lớn dần theo thời gian. Càng dùng lâu, càng giá trị.

Nói ngắn gọn: *nguồn thô là quặng, wiki là kim loại đã luyện*. Ta luyện một lần, rồi giữ sạch.

---

## 2. BA TẦNG (kiến trúc)

| Tầng | Thư mục | Vai trò | Quyền |
|------|---------|---------|-------|
| **Nguồn thô** | `raw/` | Nguồn chân lý, BẤT BIẾN (file `.srt`, tài liệu, ảnh trong `raw/assets/`) | **Chỉ đọc — KHÔNG BAO GIỜ sửa** |
| **Wiki** | `wiki/` | Các trang markdown do LLM sinh ra và sở hữu hoàn toàn | Tạo, cập nhật, giữ liên kết chéo |
| **Sản phẩm** | `production/` | Tài liệu đầu ra sinh ra khi người dùng ra lệnh (blog, bài Facebook, kịch bản video, PDF...) | Tạo theo yêu cầu |
| **Schema** | `CLAUDE.md` | File này — quy tắc, quy ước, quy trình | Cập nhật khi quy ước thay đổi |

### Chi tiết từng tầng

- **`raw/`** — Nguồn thô, BẤT BIẾN. Chỉ đọc, KHÔNG BAO GIỜ sửa. Đây là nguồn chân lý.
  Mọi khẳng định trong wiki phải truy vết được về đây. Ảnh/tệp đính kèm để trong `raw/assets/`.
- **`wiki/`** — Do LLM sinh ra và sở hữu. Gồm các loại trang: tóm tắt nguồn, thực thể, khái
  niệm, bảng so sánh, tổng quan, tổng hợp. Ta tự do tạo, sửa, tái tổ chức miễn giữ nhất quán.
- **`production/`** — Tài liệu thành phẩm cho người dùng. Sinh ra **sau này**, khi có lệnh cụ
  thể. Được phép trích dẫn wiki và nguồn thô làm chất liệu.

### Cây thư mục

```
.
├── CLAUDE.md                  ← file này (schema)
├── raw/                       ← nguồn thô, bất biến
│   └── assets/                ← ảnh, tệp đính kèm của nguồn
├── wiki/
│   ├── index.md               ← danh mục MỌI trang wiki
│   ├── log.md                 ← nhật ký chỉ-thêm (append-only)
│   ├── tong-quan.md           ← trang tổng quan toàn vault
│   ├── thuc-the/              ← trang thực thể (người, tổ chức, sản phẩm...)
│   ├── khai-niem/             ← trang khái niệm (ý tưởng, phương pháp, thuật ngữ)
│   ├── tom-tat-nguon/         ← một trang tóm tắt cho mỗi nguồn trong raw/
│   └── so-sanh/               ← bảng so sánh, đối chiếu
└── production/                ← tài liệu thành phẩm theo yêu cầu
```

---

## 3. QUY ƯỚC NGÔN NGỮ

- **Toàn bộ vault và schema viết bằng TIẾNG VIỆT.**
- Nếu buộc phải dùng thuật ngữ nước ngoài, **luôn kèm chú thích tiếng Việt trong ngoặc ngay
  lần đầu xuất hiện**. Ví dụ: "embedding (vector nhúng)", "prompt (câu lệnh)", "pipeline (quy
  trình xử lý)".

---

## 4. QUY ƯỚC TRANG WIKI

### 4.1. Tên file

- Chữ **thường**, **không dấu**, nối bằng **gạch ngang**.
- Ví dụ: `pham-thanh-long.md`, `quy-trinh-8-2.md`, `ips-16.md`.

### 4.2. Frontmatter YAML (bắt buộc, đặt ở đầu mỗi trang)

```yaml
---
tieu_de: Tên đầy đủ có dấu của trang
loai: thuc-the        # một trong: thuc-the | khai-niem | tom-tat-nguon | so-sanh | tong-quan | tong-hop
ngay_tao: 2026-07-18
ngay_cap_nhat: 2026-07-18
nguon:                # danh sách file nguồn tham chiếu (để [] nếu chưa có)
  - ten-nguon.srt
tags:
  - the-loai
  - chu-de
---
```

Giá trị hợp lệ của `loai`:

| Giá trị | Dùng cho | Thư mục |
|---------|----------|---------|
| `thuc-the` | Người, tổ chức, sản phẩm, địa điểm cụ thể | `wiki/thuc-the/` |
| `khai-niem` | Ý tưởng, phương pháp, thuật ngữ, quy trình | `wiki/khai-niem/` |
| `tom-tat-nguon` | Tóm tắt một nguồn trong `raw/` | `wiki/tom-tat-nguon/` |
| `so-sanh` | Bảng đối chiếu nhiều đối tượng | `wiki/so-sanh/` |
| `tong-quan` | Trang tổng quan toàn vault | `wiki/` |
| `tong-hop` | Trang tổng hợp/phân tích liên nguồn | `wiki/` (hoặc thư mục phù hợp) |

### 4.3. Liên kết chéo

- Liên kết giữa các trang bằng cú pháp **`[[ten-file-khong-duoi]]`** (Obsidian wikilink).
  Ví dụ: `[[pham-thanh-long]]`, `[[quy-trinh-8-2]]`.
- Liên kết **liberally** (rộng tay): mỗi khi nhắc tới một thực thể/khái niệm đã (hoặc nên) có
  trang, hãy liên kết. Một `[[ten]]` chưa có trang là dấu hiệu "cần tạo sau", không phải lỗi.

### 4.4. Trích dẫn nguồn — BẮT BUỘC

- Mọi khẳng định rút ra từ nguồn **phải TRÍCH DẪN** ngay tại chỗ, theo dạng:
  **`(nguồn: ten-file.srt, mốc thời gian nếu có)`**.
- Ví dụ: "Bán hàng là phục vụ *(nguồn: buoi-1.srt, 00:12:30)*."
- Không có trích dẫn = không đưa vào wiki. Nếu là suy luận của LLM (không có trong nguồn),
  ghi rõ *(suy luận, chưa có nguồn)* để phân biệt.

---

## 5. QUY TRÌNH "NẠP NGUỒN" (Ingest)

> Mặc định nạp **TỪNG nguồn một, có giám sát**. Hỗ trợ nạp hàng loạt nếu người dùng yêu cầu.
> **Lưu ý:** một nguồn có thể chạm **10–15 trang** wiki — đây là việc bình thường, không phải
> làm quá. Tích hợp sâu mới là giá trị của hệ thống.

Checklist:

1. **Đọc** file nguồn trong `raw/` (đọc trọn vẹn, không lướt).
2. **Trao đổi** với người dùng vài ý chính rút ra được — xác nhận hiểu đúng trước khi ghi.
3. **Viết trang tóm tắt** trong `wiki/tom-tat-nguon/` (loai: `tom-tat-nguon`), có trích dẫn mốc
   thời gian.
4. **Cập nhật `wiki/index.md`** — thêm trang mới vào đúng hạng mục.
5. **Cập nhật / tạo các trang thực thể và khái niệm liên quan** khắp wiki (đây là bước nặng
   nhất — có thể chạm 10–15 trang: bổ sung dữ kiện mới, thêm liên kết chéo, tạo trang mới cho
   thực thể/khái niệm chưa có).
6. **Ghi chú mâu thuẫn** nếu nguồn mới chọi với nguồn cũ — nêu rõ trang nào, khẳng định nào,
   nguồn nào nói gì. Không tự ý xóa dữ liệu cũ; đánh dấu để người dùng phân xử.
7. **Thêm 1 dòng vào `wiki/log.md`** (mục `ingest`).

---

## 6. QUY TRÌNH "TRUY VẤN" (Query)

1. **Đọc `wiki/index.md` TRƯỚC** để tìm các trang liên quan.
2. **Đọc các trang đó** (và các trang chúng liên kết tới nếu cần).
3. **Tổng hợp câu trả lời CÓ TRÍCH DẪN** — dẫn về nguồn `.srt` gốc, không chỉ dẫn về trang wiki.
4. Nếu câu trả lời **có giá trị tích lũy** (một so sánh mới, một phân tích, một mối liên hệ vừa
   phát hiện), **đề nghị lưu ngược lại** thành trang wiki mới (thường loai `tong-hop` hoặc
   `so-sanh`) để wiki lớn dần. Chỉ lưu khi người dùng đồng ý.
5. (Tùy chọn) Thêm 1 dòng vào `wiki/log.md` (mục `query`) nếu truy vấn tạo ra tri thức mới.

---

## 7. QUY TRÌNH "RÀ SOÁT" (Lint)

Định kỳ hoặc khi được yêu cầu, quét toàn wiki tìm:

- **Mâu thuẫn** giữa các trang (hai trang khẳng định trái nhau).
- **Khẳng định lỗi thời** (nguồn mới đã ghi đè nhưng trang cũ chưa cập nhật).
- **Trang mồ côi** (không có trang nào liên kết `[[...]]` trỏ tới).
- **Khái niệm quan trọng chưa có trang riêng** (được nhắc nhiều nhưng chưa được trích xuất).
- **Thiếu liên kết chéo** (nhắc tên thực thể/khái niệm đã có trang nhưng quên `[[...]]`).
- **Khoảng trống dữ liệu** (câu hỏi hiển nhiên mà wiki chưa trả lời được).

Kết thúc rà soát: **đề xuất câu hỏi mới cần làm rõ và nguồn cần đi tìm**. Ghi 1 dòng vào
`wiki/log.md` (mục `lint`).

---

## 8. VAI TRÒ `index.md` VÀ `log.md`

- **`wiki/index.md`** — *bản đồ* của wiki. Danh mục MỌI trang, tổ chức theo hạng mục, mỗi trang
  một dòng kèm tóm tắt một dòng. **Cập nhật MỖI lần nạp nguồn.** Truy vấn luôn bắt đầu từ đây.
- **`wiki/log.md`** — *dòng thời gian* của vault. Nhật ký **chỉ-thêm** (append-only), không sửa
  mục cũ. Mỗi mục có tiền tố nhất quán để `grep` được. Lấy 5 mục gần nhất:
  ```
  grep "^## \[" wiki/log.md | tail -5
  ```

---

## 9. NHẮC NHANH (cheat-sheet)

- Nguồn thô → **chỉ đọc**. Wiki → **của ta**. `production/` → **theo lệnh**.
- Tiếng Việt toàn bộ; thuật ngữ ngoại kèm chú thích ngoặc lần đầu.
- Tên file: thường-không-dấu-gạch-ngang. Luôn có frontmatter YAML.
- Mọi khẳng định → trích dẫn `(nguồn: file.srt, mốc giờ)`.
- Nạp nguồn = 7 bước, chạm 10–15 trang là bình thường.
- Truy vấn = đọc `index.md` trước.
- Không xóa dữ liệu cũ khi mâu thuẫn — đánh dấu để người dùng phân xử.
