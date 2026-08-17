# Bài 3: Phân tích Tài chính — Tính toán chi phí AI

**Sinh viên thực hiện:** Đào Trọng Trí (Mã SV: PTIT062)

## 1. Tính toán chi phí cơ bản (chưa tính lỗi)
Em xin phép thực hiện tính toán các chi phí như sau:
- **Tổng số hóa đơn 1 ngày:** 10,000
- **Tổng Token Input 1 ngày:** 10,000 × 1,500 = 15,000,000 tokens (15M)
- **Tổng Token Output 1 ngày:** 10,000 × 500 = 5,000,000 tokens (5M)

### Mô hình A (Direct API - DeepSeek Chat)
- Chi phí Input hàng ngày: 15 × $0.14 = $2.10
- Chi phí Output hàng ngày: 5 × $0.28 = $1.40
- **Tổng chi phí 1 ngày:** $2.10 + $1.40 = **$3.50**
- **Tổng chi phí 1 tháng (30 ngày):** $3.50 × 30 = **$105.00**

### Mô hình B (API Aggregator - Gemini 2.5 Flash qua OpenRouter)
- Chi phí Input hàng ngày: 15 × $0.075 = $1.125
- Chi phí Output hàng ngày: 5 × $0.30 = $1.50
- **Tổng chi phí 1 ngày:** $1.125 + $1.50 = **$2.625**
- **Tổng chi phí 1 tháng (30 ngày):** $2.625 × 30 = **$78.75**

---

## 2. Tính toán lại Mô hình B (có tính đến 0.5% tỉ lệ lỗi và retry)
- Lỗi kết nối khiến hệ thống phải retry, làm tăng 5% tổng số lượng token Input (token Output ở lượt fail thường không đáng kể hoặc bị drop).
- Token Input thực tế 1 ngày: 15M + (15M × 5%) = 15.75M tokens.
- Chi phí Input mới hàng ngày: 15.75 × $0.075 = $1.18125
- Chi phí Output hàng ngày (giữ nguyên): $1.50
- **Tổng chi phí 1 ngày sau điều chỉnh:** $1.18125 + $1.50 = **$2.68125**
- **Tổng chi phí 1 tháng sau điều chỉnh (30 ngày):** $2.68125 × 30 ≈ **$80.44**

*(Kết luận tài chính: Dù có thêm chi phí do retry, Mô hình B ($80.44/tháng) vẫn rẻ hơn đáng kể so với Mô hình A ($105.00/tháng)).*

---

## 3. Phân tích Trade-off Kiến trúc (Direct API vs Aggregator)
Đứng dưới góc độ Kỹ sư giải pháp, em nhận thấy quyết định chọn mô hình cho dự án lâu dài không chỉ dựa vào giá tiền mà phải xét đến các yếu tố phi tài chính:

1. **Vendor Lock-in (Sự phụ thuộc nhà cung cấp):** 
   - **Direct API:** Khá rủi ro. Nếu DeepSeek tăng giá hoặc thay đổi chính sách, việc chuyển đổi sang nhà cung cấp khác có thể yêu cầu sửa đổi thư viện/SDK trong hệ thống.
   - **Aggregator:** Tốt hơn nhiều. Thông qua chuẩn chung OpenAI-compatible của OpenRouter, ta có thể đổi model (sang Claude, GPT, Llama, v.v.) chỉ bằng việc sửa 1 dòng file cấu hình properties mà không phải sửa code.
2. **Latency (Độ trễ):**
   - **Direct API:** Nhanh hơn do kết nối thẳng từ máy chủ đến Data Center của nhà cung cấp mô hình.
   - **Aggregator:** Chậm hơn do mất thêm 1 network hop (phải đi qua trung gian OpenRouter). Với bài toán xử lý hóa đơn (chạy background/batch) thì độ trễ vài trăm ms không quá quan trọng, nhưng nếu là Real-time Chatbot thì đây là điểm trừ lớn.
3. **SLA và Ổn định (Stability):**
   - **Direct API:** Tốt hơn do SLA được cam kết trực tiếp từ một điểm chạm duy nhất.
   - **Aggregator:** Kém hơn (như đề bài đã nêu là 0.5% lỗi). Tính ổn định phụ thuộc vào 2 yếu tố: Server của Aggregator VÀ Server của nhà cung cấp LLM. Nếu 1 trong 2 gặp sự cố, hệ thống đều lỗi. Yêu cầu hệ thống phải có cơ chế Retry/Circuit Breaker mạnh mẽ.

**Quyết định đề xuất của em:**
- Nếu dự án là hệ thống xử lý nội bộ (back-office) chạy theo batch/lô như trích xuất hóa đơn, không yêu cầu độ phản hồi real-time khắt khe -> Theo em, nên **chọn Mô hình B (Aggregator)** để tối ưu chi phí và tránh Vendor Lock-in, giúp linh hoạt đổi mô hình sau này.
- Nếu dự án là hệ thống Core ảnh hưởng trực tiếp tới trải nghiệm người dùng cuối (ví dụ Chatbot Real-time CSKH) yêu cầu độ trễ thấp và SLA > 99.9% -> Em đề xuất cân nhắc **Mô hình A (Direct API)**.
