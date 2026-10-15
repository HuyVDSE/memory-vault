---
title: Nghĩ giải pháp phức tạp thì dễ → làm thì khó. Nghĩ đơn giản thì khó → làm thì dễ
tags:
  - tu-duy-thiet-ke
  - do-phuc-tap
  - dev
created: 2026-10-15
status: draft
audience: dev
---
# Nghĩ phức tạp thì dễ → làm khó. Nghĩ đơn giản thì khó → làm dễ

> Nghĩ giải pháp phức tạp thì dễ → làm thì khó.
> Nghĩ giải pháp đơn giản thì khó → làm thì dễ.

## 1. Ý tổng quát

**Chi phí của sự phức tạp không biến mất, nó chỉ bị dời từ lúc nghĩ sang lúc làm và lúc bảo trì.**

Ai trả chi phí và trả lúc nào là khác nhau giữa hai cách nghĩ:

| | Nghĩ phức tạp | Nghĩ đơn giản |
|---|---|---|
| Chi phí ở khâu **nghĩ** | Thấp | **Cao** (phải hiểu bối cảnh, quyết định bỏ gì) |
| Chi phí ở khâu **làm** | **Cao** | Thấp |
| Chi phí ở khâu **bảo trì/sửa lỗi** | **Rất cao** | Thấp |
| Bản chất | Cộng dồn (additive) | Loại trừ (subtractive) |

## 2. Giải thích thuật ngữ

| Thuật ngữ | Nghĩa |
|---|---|
| **Additive (cộng dồn)** | Có yêu cầu mới thì thêm thành phần mới: bảng, cờ, nhánh `if`, service. |
| **Subtractive (loại trừ)** | Có yêu cầu mới thì tự hỏi: cái gì là cốt lõi, cái gì bỏ được mà không hại. |
| **Coupling (ràng buộc chéo)** | Mức độ hai thành phần phụ thuộc vào nhau: đổi cái này thì cái kia có thể hỏng. |
| **State explosion (bùng nổ trạng thái)** | Số tổ hợp trạng thái tăng rất nhanh khi thêm biến trạng thái. |
| **Invariant (bất biến)** | Điều kiện luôn phải đúng trong hệ thống (ví dụ: hóa đơn đã phát hành thì không đổi nội dung). |

## 3. Cơ chế: vì sao nghĩ phức tạp thì dễ, làm thì khó

### 3.1. Thêm thì không cần hiểu toàn bộ

Khi nhận yêu cầu mới, cách rẻ nhất về tư duy là **thêm** một thứ vào chỗ đang có: thêm cột, thêm cờ `isXxx`, thêm nhánh `if`. Không cần biết cả hệ thống đang vận hành thế nào, chỉ cần biết chỗ mình sửa. Vì vậy dễ nghĩ ra.

### 3.2. Mỗi thành phần thêm vào đều tương tác với các thành phần cũ

Với n thành phần, số cặp có thể ảnh hưởng lẫn nhau là n(n-1)/2, tăng cỡ n². Số tổ hợp trạng thái còn tăng nhanh hơn nữa.

Ví dụ cụ thể với cờ boolean:

- 3 cờ `isPaid`, `isCancelled`, `isInvoiced` → 2³ = **8 tổ hợp**, trong đó nhiều tổ hợp vô nghĩa (đã hủy nhưng đã xuất hóa đơn và chưa thanh toán?).
- Thêm 3 cờ nữa → 2⁶ = **64 tổ hợp**.

Mỗi tổ hợp là một case có thể xảy ra trên production, cần được code xử lý hoặc cấm, và cần test.

```java
// Cách phức tạp (dễ nghĩ): mỗi yêu cầu mới thêm một cờ
if (order.isPaid() && !order.isCancelled() && order.isInvoiced() && !order.isRefunding()) {
    ...
}

// Cách đơn giản (khó nghĩ hơn): một trạng thái duy nhất, chuyển trạng thái có kiểm soát
enum OrderStatus { CREATED, PAID, INVOICED, CANCELLED, REFUNDED }
// Chỉ định nghĩa các chuyển trạng thái hợp lệ, các tổ hợp vô nghĩa không thể tồn tại
```

### 3.3. Kết quả: làm khó, sửa còn khó hơn

- **Lúc làm:** phải xử lý các tổ hợp và tương tác giữa các thành phần.
- **Lúc test:** số case cần phủ tăng theo tổ hợp, không theo số tính năng.
- **Lúc sửa:** sửa một chỗ có thể làm hỏng chỗ khác (coupling), nên dễ rơi vào vòng lặp "sửa một cái sai, đè thêm một cái sai".

## 4. Cơ chế: vì sao nghĩ đơn giản thì khó, làm thì dễ

### 4.1. Phải biết bỏ gì, và bỏ gì thì đúng

Đơn giản hóa là quyết định **không làm** hoặc **làm khác đi**. Muốn quyết định đúng phải:

1. Hiểu đủ bối cảnh (dữ liệu vào ra thế nào, ai dùng, sai thì hậu quả gì).
2. Xác định đâu là invariant bắt buộc giữ.
3. Dám chịu trách nhiệm cho phần mình không làm.

Đây là phần tốn công suy nghĩ nhất, và là lý do "nghĩ đơn giản thì khó".

### 4.2. Khi đã chọn đúng, phần làm gần như tự nhiên

- Ít thành phần → ít tương tác → ít tổ hợp trạng thái.
- Invariant được mã hóa vào cấu trúc (ví dụ enum trạng thái, bản ghi bất biến) nên sai là bị chặn từ đầu, không phải kiểm tra rải rác.
- Dễ test, dễ đọc, dễ sửa.

## 5. Ví dụ đối chiếu: "cho phép sửa hóa đơn"

| | Cách nghĩ phức tạp (dễ nghĩ) | Cách nghĩ đơn giản (khó nghĩ) |
|---|---|---|
| Ý tưởng | Cho sửa trực tiếp, thêm cờ `isEdited`, thêm bảng lịch sử, thêm code đồng bộ lại với hệ thống thuế | Hóa đơn đã phát hành là **bất biến**. Muốn sửa thì phát hành bản **thay thế/điều chỉnh** |
| Cần hiểu gì trước | Gần như không | Quy định thuế về hóa đơn, ý nghĩa pháp lý của hóa đơn đã phát hành |
| Khi làm | Xung đột với hóa đơn đã gửi cơ quan thuế, nhiều case đồng bộ | Chỉ tạo thêm một bản ghi mới liên kết với bản cũ |
| Khi sai | Dữ liệu cũ bị ghi đè, khó truy vết | Bản cũ còn nguyên, truy vết tự nhiên |

## 6. Các cách diễn đạt ý này khi trình bày với dev

| Góc nhìn | Cách diễn đạt | Ưu | Nhược |
|---|---|---|---|
| **Dời chi phí** | Chi phí không mất đi, chỉ chuyển từ khâu nghĩ sang khâu làm và bảo trì | Gọn, dễ nhớ | Hơi trừu tượng nếu thiếu ví dụ |
| **Cộng và trừ** | Phức tạp là thêm vào, đơn giản là biết bỏ gì | Cụ thể, dev dễ hình dung qua code | Dễ hiểu nhầm là "càng ít tính năng càng tốt" |
| **Tương tác tăng nhanh** | Số thành phần tăng thì số tương tác và tổ hợp trạng thái tăng nhanh hơn nhiều | Có cơ chế rõ, hợp với tư duy kỹ thuật, minh họa được bằng ví dụ cờ boolean | Cần ví dụ thật, không thì nghe như lý thuyết |
| **Hiểu bối cảnh** | Đơn giản là kết quả của việc hiểu đủ bối cảnh, không phải của việc lười | Nối thẳng với việc tổng hợp bối cảnh trước khi làm | Không tự giải thích được vì sao phức tạp lại khó làm |

**Gợi ý với dev:** mở đầu bằng góc **tương tác tăng nhanh** (ví dụ cờ boolean, có số cụ thể), sau đó chốt bằng góc **cộng và trừ** và **hiểu bối cảnh**.

## 7. Lưu ý để tránh hiểu sai

- **Đơn giản không có nghĩa là thiếu sót.** Bỏ nhầm thứ cần thiết chỉ là *đơn giản hóa quá mức*, và sai sót sẽ lộ ra sau.
- **Đơn giản không có nghĩa là ít code hơn.** Một giải pháp đơn giản có thể nhiều dòng hơn nhưng ít ràng buộc chéo hơn.
- **Đơn giản không có nghĩa là quen tay.** Cái quen tay là *dễ* (easy), không nhất thiết *đơn giản* (simple). Xem thêm [[Phan-biet-chuc-nang-don-gian-va-phuc-tap]]
- Mục tiêu là **đơn giản nhất có thể mà vẫn đúng với bối cảnh**. Vì vậy cần tổng hợp bối cảnh trước khi quyết định bỏ gì.

## 8. Câu hỏi mở

- [ ] Chọn ví dụ code thật từ dự án của team để thay cho ví dụ cờ boolean và hóa đơn?
- [ ] Có nên thêm một bài tập ngắn: đưa một yêu cầu, cho dev tự viết hai phương án (cộng dồn và loại trừ) rồi so sánh số tổ hợp trạng thái?