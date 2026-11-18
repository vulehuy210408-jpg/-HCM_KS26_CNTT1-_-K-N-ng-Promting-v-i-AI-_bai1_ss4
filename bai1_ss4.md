# [Vận dụng chuyên sâu] Thiết kế giải pháp Prompting ứng dụng thực tế cho Kỹ thuật Prompting Nâng cao (Logic & Chuỗi)
## Phần 1: Báo cáo phân tích và Thiết kế giải pháp (I/O)
1. Bối cảnh & Mục tiêu bài toán
Bối cảnh thực tế: Một cửa hàng bán lẻ trực tuyến cần tự động hóa quy trình xử lý đơn hàng, kiểm tra tính hợp lệ của dữ liệu đầu vào (mã sản phẩm, số lượng, giá tiền, thông tin khách hàng) và tính toán tổng tiền thanh toán kèm thuế/phí vận chuyển bằng tư duy logic từng bước (Chain-of-Thought).

Mục tiêu: Xây dựng một giải pháp Prompting nâng cao giúp AI phân tích dữ liệu đơn hàng phức tạp, xử lý triệt để các trường hợp dữ liệu lỗi hoặc thiếu sót, và xuất kết quả chuẩn hóa dạng JSON phục vụ tích hợp hệ thống backend.

2. Mô tả dữ liệu đầu vào (Input)
Dữ liệu đơn hàng đầu vào bao gồm:

Thông tin khách hàng: Họ tên, số điện thoại, địa chỉ giao hàng.

Danh sách sản phẩm mua: Mã sản phẩm, tên sản phẩm, số lượng đặt mua, đơn giá niêm yết.

Mã giảm giá / Voucher (nếu có): Tỷ lệ hoặc giá trị giảm giá áp dụng.

3. Cấu trúc kết quả mong muốn (Output Schema)
Kết quả trả về phải tuân thủ cấu trúc JSON chuẩn hóa chứa các trường: trạng thái xử lý đơn hàng (status), danh sách chi tiết sản phẩm đã chuẩn hóa, tổng tiền tạm tính, chi phí vận chuyển, tổng giá trị thanh toán cuối cùng (final_total), và ghi chú logic xử lý từng bước.
## Phần 2: Nội dung Prompt hoàn chỉnh (Advanced Structured Prompting)
### [Context & Role]
Bạn là một hệ thống AI backend chuyên xử lý, kiểm tra dữ liệu và tính toán tự động các đơn hàng thương mại điện tử với độ chính xác tuyệt đối.

### [Input Data - Đơn hàng cần xử lý]
- Khách hàng: Nguyễn Văn A (SĐT: 0901234567)
- Địa chỉ: 123 Lê Lợi, Phường Bến Thành, Quận 1, TP.HCM
- Danh sách sản phẩm:
  1. Mã SP: SP01 | Tên: Tai nghe Bluetooth X1 | Số lượng: 2 | Đơn giá: 500.000 VNĐ
  2. Mã SP: SP02 | Tên: Cáp sạc nhanh Type-C | Số lượng: -1 | Đơn giá: 150.000 VNĐ (Lỗi số lượng âm)
  3. Mã SP: UNKNOWN | Tên: Sản phẩm không rõ mã | Số lượng: 1 | Đơn giá: 200.000 VNĐ
- Voucher giảm giá: Giảm 10% tổng đơn hàng (Mã: SALE10)

### [Ràng buộc & Xử lý lỗi biên (Edge Cases)]
1. **INVALID_INPUT:** Nếu số lượng sản phẩm nhỏ hơn hoặc bằng 0, hãy đánh dấu lỗi và loại bỏ sản phẩm đó ra khỏi danh sách tính tiền, đồng thời ghi nhận vào trường lỗi (`error_log`).
2. **UNKNOWN_PRODUCT:** Nếu mã sản phẩm không tồn tại trong hệ thống chuẩn, hãy gắn nhãn lỗi sản phẩm không xác định.
3. **MISSING_INFO:** Nếu thiếu thông tin bắt buộc của khách hàng, trả về trạng thái "REJECTED".

### [Quy trình lập luận bắt buộc (Chain-of-Thought Steps)]
- Bước 1: Kiểm tra tính hợp lệ của từng sản phẩm trong đơn hàng (kiểm tra số lượng dương, mã sản phẩm hợp lệ).
- Bước 2: Tính thành tiền cho từng dòng sản phẩm hợp lệ ($\text{Thành tiền} = \text{Số lượng} \times \text{Đơn giá}$).
- Bước 3: Cộng dồn tổng tiền tạm tính (`subtotal`) của toàn bộ các sản phẩm hợp lệ.
- Bước 4: Áp dụng mã giảm giá (Voucher SALE10 giảm 10%) và cộng thêm phí vận chuyển cố định 30.000 VNĐ để tính tổng thanh toán cuối cùng (`final_total`).

### [Action & Format Constraints]
- Xuất kết quả duy nhất dưới dạng JSON thô (Raw JSON), không kèm theo văn bản giải thích ngoài lề hoặc mã Markdown code block (như ```json).
## Phần 3: Kết quả chạy thử thực tế (Simulation Output của AI)
{
  "order_status": "PROCESSED_WITH_WARNINGS",
  "customer_info": {
    "name": "Nguyễn Văn A",
    "phone": "0901234567",
    "address": "123 Lê Lợi, Phường Bến Thành, Quận 1, TP.HCM"
  },
  "validated_items": [
    {
      "product_id": "SP01",
      "product_name": "Tai nghe Bluetooth X1",
      "quantity": 2,
      "unit_price": 500000,
      "line_total": 1000000,
      "status": "VALID"
    }
  ],
  "rejected_items": [
    {
      "product_id": "SP02",
      "product_name": "Cáp sạc nhanh Type-C",
      "quantity": -1,
      "reason": "INVALID_INPUT: Số lượng sản phẩm nhỏ hơn hoặc bằng 0."
    },
    {
      "product_id": "UNKNOWN",
      "product_name": "Sản phẩm không rõ mã",
      "quantity": 1,
      "reason": "UNKNOWN_PRODUCT: Mã sản phẩm không tồn tại trong hệ thống."
    }
  ],
  "financial_calculation": {
    "subtotal": 1000000,
    "discount_code": "SALE10",
    "discount_amount": 100000,
    "shipping_fee": 30000,
    "final_total": 930000
  },
  "reasoning_steps": [
    "Bước 1: Đã kiểm tra 3 sản phẩm. Sản phẩm SP02 bị loại do số lượng âm (-1). Sản phẩm UNKNOWN bị loại do mã không hợp lệ. Chỉ giữ lại sản phẩm SP01.",
    "Bước 2 & 3: Tính tạm tính cho sản phẩm SP01: 2 * 500.000 = 1.000.000 VNĐ.",
    "Bước 4: Áp dụng giảm giá 10% (giảm 100.000 VNĐ), cộng phí vận chuyển 30.000 VNĐ. Tổng thanh toán cuối cùng = 1.000.000 - 100.000 + 30.000 = 930.000 VNĐ."
  ]
}