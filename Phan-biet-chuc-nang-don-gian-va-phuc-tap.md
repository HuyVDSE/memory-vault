---
title: Phân biệt chức năng đơn giản và chức năng phức tạp
tags:
  - phan-tich-yeu-cau
  - thiet-ke-he-thong
  - do-phuc-tap
created: 2026-10-15
status: draft
---
# Phân biệt chức năng đơn giản và chức năng phức tạp

## 1. Ý chính

**Độ lớn của chức năng không phải độ phức tạp của nó.** "Đơn giản/phức tạp" thường bị đánh giá theo cảm giác, nên cần các tiêu chí đo được.

- Nút **"Xuất hóa đơn"** nhìn rất nhỏ, nhưng có thể chạm vào: trạng thái đơn hàng, quy định thuế, hệ thống bên thứ ba, và việc retry khi lỗi → **phức tạp**.
- Màn hình **CRUD 20 trường** (thêm, xem, sửa, xóa) nhìn to, nhưng các trường không ràng buộc lẫn nhau → **đơn giản**.

## 2. Giải thích thuật ngữ

| Thuật ngữ | Nghĩa |
|---|---|
| **Simple (đơn giản)** | Ít thứ đan vào nhau. Đây là thuộc tính của *cấu trúc*, khách quan hơn. |
| **Easy (dễ)** | Quen tay, gần với thứ mình đã biết. Đây là thuộc tính của *người làm*, thay đổi theo kinh nghiệm. |
| **Happy path** | Luồng chạy lý tưởng, mọi thứ đều đúng. |
| **Ngoại lệ nghiệp vụ** | Các trường hợp "nhưng mà..." ngoài luồng chính: hủy, hoàn, sửa sau khi đã chốt... |
| **Đảo ngược được (reversible)** | Làm sai rồi vẫn sửa lại được mà không để lại hậu quả bên ngoài (tiền, thuế, dữ liệu đã gửi đi). |

> Phân biệt của Rich Hickey: *simple* ≠ *easy*. Một giải pháp có thể **dễ** làm (quen tay, copy từ chỗ cũ) nhưng **không đơn giản** (đan xen nhiều thứ). Giải pháp phức tạp dễ nghĩ ra vì cứ thêm vào là xong. Giải pháp đơn giản khó nghĩ vì phải hiểu đủ bối cảnh mới biết nên **bỏ gì**.

## 3. Bốn góc nhìn để đo

### Góc 1: Mức độ đan xen
- **Cách đo:** đếm số thực thể, số trạng thái, số hệ thống ngoài mà chức năng chạm tới. Càng nhiều thứ phụ thuộc lẫn nhau càng phức tạp.
- **Ưu:** đếm được, bám theo ERD.
- **Nhược:** sai nếu ERD chưa đầy đủ; không bắt được rủi ro nằm ngoài sơ đồ.

### Góc 2: Mức độ bất định
- **Cách đo:** yêu cầu đã rõ chưa? Đã có ai làm việc tương tự chưa? Nhiều ẩn số thì phức tạp, dù code ít.
- **Ưu:** bắt được loại chức năng "tưởng dễ mà làm mới thấy khó".
- **Nhược:** chủ quan, phụ thuộc kinh nghiệm người đánh giá.

### Góc 3: Chi phí khi sai
- **Cách đo:** nếu sai thì sửa có dễ không? Có ảnh hưởng tiền, thuế, dữ liệu đã ghi ra ngoài không? Có đảo ngược được không?
- **Ưu:** gắn thẳng với rủi ro "đè sửa một cái sai".
- **Nhược:** chức năng nhỏ nhưng nhạy cảm sẽ bị xếp "phức tạp" dù cấu trúc đơn giản.

### Góc 4: Số ngoại lệ nghiệp vụ
- **Cách đo:** hỏi người nghiệp vụ và đếm số lần họ nói "nhưng mà...".
- **Ưu:** sát thực tế, người nghiệp vụ trả lời được ngay.
- **Nhược:** phụ thuộc người nghiệp vụ chịu kể hết; bỏ sót ngoại lệ chưa ai từng gặp.

### Bảng tóm tắt

| Góc nhìn | Đo bằng | Bắt tốt | Dễ bỏ sót |
|---|---|---|---|
| Đan xen | Số thực thể, trạng thái, hệ thống ngoài | Phụ thuộc chéo | Rủi ro ngoài ERD |
| Bất định | Yêu cầu rõ/mơ hồ, tiền lệ | "Tưởng dễ mà khó" | Khác nhau theo kinh nghiệm |
| Chi phí khi sai | Đảo ngược được không | Việc nhạy cảm (tiền, thuế) | Cấu trúc vs hệ quả bị lẫn |
| Ngoại lệ nghiệp vụ | Số "nhưng mà..." | Thực tế vận hành | Ngoại lệ chưa ai gặp |

## 4. Bộ câu hỏi nhanh trước khi làm

1. Chức năng này làm thay đổi **bao nhiêu thực thể và trạng thái**?
2. Nếu sai, **có sửa lại được không**, hay đã ghi tiền, thuế hoặc dữ liệu ra ngoài?
3. Có phần nào **chưa rõ yêu cầu** không?
4. Có ai (hệ thống khác, bộ phận khác) **phụ thuộc** vào kết quả này không?

### Quy tắc đọc kết quả

- Trả lời "nhiều/có" ở **từ 2 câu trở lên** → coi là **phức tạp**: phân tích kỹ bối cảnh trước khi làm (ghi nhận/truy vấn, quản lý, giám sát).
- Dưới 2 câu → coi là **đơn giản**: lướt nhanh, làm gọn.

→ **Độ sâu phân tích tỉ lệ với độ phức tạp đo được.**

> Ngưỡng "từ 2 câu" là heuristic để bắt đầu, không phải chân lý. Nên hiệu chỉnh theo thực tế của team sau vài chức năng.

## 5. Ví dụ đối chiếu

| | Danh mục sản phẩm (CRUD) | Xuất hóa đơn từ đơn hàng |
|---|---|---|
| Thực thể/trạng thái | 1 thực thể, vài trạng thái bật/tắt | Đơn hàng, hóa đơn, khách hàng, thuế; nhiều trạng thái liên động |
| Sai thì sao | Sửa lại trực tiếp | Hóa đơn đã phát hành phải điều chỉnh/thay thế, không xóa được |
| Yêu cầu rõ chưa | Rõ | Phụ thuộc quy định thuế, nhiều trường hợp riêng |
| Phụ thuộc bên ngoài | Không | Hệ thống thuế/nhà cung cấp hóa đơn, có thể lỗi và phải retry |
| **Kết luận** | **Đơn giản** | **Phức tạp** |

## 6. Sai lầm hay gặp

- **Đánh giá theo số màn hình/số trường** thay vì theo mức đan xen.
- **Nhầm "dễ" với "đơn giản":** thấy quen tay nên cho là đơn giản, dù cấu trúc rối.
- **Nhầm "chưa biết" với "phức tạp":** có khi chỉ là thiếu thông tin; hỏi thêm là hết. Còn bất định thật sự (yêu cầu mâu thuẫn, chưa ai làm) mới là phức tạp.
- **Chỉ nhìn happy path:** quên ngoại lệ, nên đánh giá thấp độ phức tạp.
- **Gán nhãn một lần rồi thôi:** chức năng có thể bắt đầu đơn giản rồi phức tạp dần khi thêm ngoại lệ; cần đánh giá lại khi yêu cầu đổi.

## 7. Câu hỏi mở

- [ ] Ngưỡng "từ 2 câu" có phù hợp với team không, hay cần trọng số khác nhau cho từng câu (ví dụ câu 2 về khả năng đảo ngược quan trọng hơn)?
- [ ] Có nên thêm góc nhìn về **ràng buộc phi chức năng** (khối lượng dữ liệu, hiệu năng, đồng thời) vào bộ đo không?