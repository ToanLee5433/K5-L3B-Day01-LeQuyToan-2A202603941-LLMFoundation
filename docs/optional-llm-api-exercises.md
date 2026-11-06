# Ngày 1 — Bài Tập & Phản Ánh (side lab TUỲ CHỌN)

> Side lab này **không** thuộc điểm chính của Day 01 (xem [RUBRIC.md](../RUBRIC.md) mục Bonus). Model: `gpt-6.1-terra` (lớn) và `gpt-6-luna` (nhỏ). `gpt-4o` / `gpt-4o-mini` bị gỡ khỏi API ngày 14/01/2027 ([nguồn](https://developers.openai.com/api/docs/deprecations)). Tên khoá `gpt4o_*` / `mini_*` trong kết quả là hợp đồng của test, giữ nguyên để tương thích: đọc là "model lớn" / "model nhỏ".
## Nền Tảng LLM API | Phiếu Thực Hành

**Thời lượng:** 1:30 giờ  
**Cấu trúc:** Lập trình cốt lõi (60 phút) → Bài tập mở rộng (30 phút)

---

## Phần 1 — Lập Trình Cốt Lõi (0:00–1:00)

Chạy các ví dụ trong Google Colab tại: https://colab.research.google.com/drive/172zCiXpLr1FEXMRCAbmZoqTrKiSkUERm?usp=sharing

Copy `src/optional_llm_api/template.py` thành `src/optional_llm_api/solution/solution.py`, triển khai mọi TODO, rồi chạy từ gốc repo: `pytest tests/test_solution.py -v`. Chưa có `solution/solution.py` thì test tự bỏ qua.

**Điểm kiểm tra:** Sau khi hoàn thành 4 nhiệm vụ, chạy:
```bash
python src/optional_llm_api/solution/solution.py
```
Bạn sẽ thấy output so sánh phản hồi của `gpt-6.1-terra` và `gpt-6-luna`.

---

## Phần 2 — Bài Tập Mở Rộng (1:00–1:30)

### Bài tập 2.1 — Độ Nhạy Của Temperature
Gọi `call_openai` với các giá trị temperature 0.0, 0.5, 1.0 và 1.5 sử dụng prompt **"Hãy kể cho tôi một sự thật thú vị về Việt Nam."** (nhớ gửi `reasoning_effort="none"`: model GPT-6 chỉ nhận `temperature` khi tắt suy luận)

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Khi temperature tăng dần từ 0.0 lên 1.5, tính ngẫu nhiên, độ phong phú từ vựng và tính sáng tạo của mô hình tăng rõ rệt nhưng mức độ kiểm soát và tính ổn định giảm đi. Ở mức 0.0 - 0.5, câu trả lời mang tính quy chuẩn, lặp lại các sự thật phổ biến nhất (như Việt Nam xuất khẩu cà phê Robusta hay hạt điều số 1 thế giới) với cấu trúc câu chặt chẽ. Khi lên mức 1.0 - 1.5, mô hình khai thác các sự thật ít phổ biến hơn, giọng văn bay bổng bất ngờ hơn, nhưng ở mức 1.5 bắt đầu xuất hiện câu từ lan man, dấu câu lạ và nguy cơ ảo giác (hallucination) tăng cao.

**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Tôi sẽ đặt temperature ở mức **0.0 đến 0.1** (tối đa 0.2). Lý do: Chatbot chăm sóc khách hàng đòi hỏi sự nhất quán tuyệt đối, độ chính xác thông tin cao (factual accuracy), bám sát tài liệu chính sách/FAQ và tri thức doanh nghiệp (RAG). Thiết lập nhiệt độ thấp triệt tiêu tính ngẫu nhiên, đảm bảo cùng một thắc mắc về chính sách/bảo hành sẽ luôn nhận được câu trả lời chuẩn mực, ngăn ngừa mô hình "sáng tạo" thông tin sai lệch gây tổn hại uy tín hoặc vi phạm pháp lý.

---

### Bài tập 2.2 — Đánh Đổi Chi Phí
Xem xét kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người thực hiện 3 lần gọi API, mỗi lần trung bình ~350 token.

**Ước tính xem `gpt-6.1-terra` đắt hơn `gpt-6-luna` bao nhiêu lần cho workload này** (giá 11/2026: terra $1.00 / $5.00, luna $0.10 / $0.50 mỗi 1M token input / output):
> `gpt-6.1-terra` đắt hơn `gpt-6-luna` đúng **10 lần** ($10\times$).
> **Chi tiết tính toán:**
> - Tỷ lệ giá: Input của Terra (\$1.00) gấp 10 lần Luna (\$0.10); Output của Terra (\$5.00) cũng gấp 10 lần Luna (\$0.50). Do tỷ số đơn giá ở cả input và output đều là $10\times$, nên với bất kỳ tỷ lệ phân chia input/output tokens nào, tổng chi phí của Terra luôn gấp đúng 10 lần Luna.
> - Tổng workload: $10.000 \times 3 = 30.000$ calls/ngày, tương đương $30.000 \times 350 = 10.500.000$ tokens/ngày (10.5M tokens).
> - Giả sử trung bình 250 token input + 100 token output:
>   - **`gpt-6-luna`**: $(30.000 \times 250 / 10^6) \times \$0.10 + (30.000 \times 100 / 10^6) \times \$0.50 = \$0.75 + \$1.50 = \mathbf{\$2.25\text{/ngày}}$ (~$67.5/tháng).
>   - **`gpt-6.1-terra`**: $(30.000 \times 250 / 10^6) \times \$1.00 + (30.000 \times 100 / 10^6) \times \$5.00 = \$7.50 + \$15.00 = \mathbf{\$22.50\text{/ngày}}$ (~$675/tháng).

**Mô tả một trường hợp mà chi phí cao hơn của `gpt-6.1-terra` là xứng đáng, và một trường hợp `gpt-6-luna` là lựa chọn tốt hơn:**
> - **Xứng đáng dùng `gpt-6.1-terra`**: Các bài toán suy luận phức tạp, phân tích điều khoản hợp đồng pháp lý / hồ sơ tài chính, chẩn đoán y tế, tự động sinh mã nguồn kiến trúc hệ thống (complex agentic workflow/tool calling), hoặc trích xuất dữ liệu có cấu trúc lồng nhau phức tạp. Ở các tác vụ này, một sai sót logic nhỏ có thể gây thiệt hại nghiệp vụ hàng ngàn đô la, vượt xa mức chênh lệch \$20/ngày của API.
> - **`gpt-6-luna` là lựa chọn tốt hơn**: Các tác vụ tần suất cao, logic đơn giản, nhạy cảm về độ trễ và chi phí: phân loại ý định người dùng (intent classification), lọc spam/tục tĩu, định tuyến câu hỏi (router), trích xuất từ khóa/NER cơ bản, hoặc tóm tắt nhanh văn bản ngắn. Luna vừa đảm bảo SLA tốc độ cao vừa tiết kiệm $90\%$ ngân sách vận hành.

---

### Bài tập 2.3 — Trải Nghiệm Người Dùng với Streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất trong các giao diện tương tác hội thoại trực tiếp với người dùng (chatbot, trợ lý ảo, interactive coding copilot) khi phản hồi có độ dài từ trung bình đến lớn. Streaming giúp tối ưu hóa chỉ số Time-To-First-Token (TTFT), đưa nội dung xuất hiện ngay lập tức (vài trăm mili-giây) thay vì bắt người dùng nhìn màn hình loading chờ đợi 5–10 giây, mang lại trải nghiệm mượt mà và trực quan. Ngược lại, non-streaming lại phù hợp và tối ưu hơn trong các tác vụ xử lý ngầm (background jobs, ETL batch pipelines), khi cần parse kết quả có cấu trúc hoàn chỉnh như JSON/Function Calling trước khi chuyển cho bước xử lý tiếp theo, hoặc các bài toán phân loại nhãn cực ngắn (như trả về nhãn 'spam'/'ham') nơi tổng thời gian sinh từ chỉ mất vài chục mili-giây.

## Danh Sách Kiểm Tra Nộp Bài
- [x] Tất cả tests pass: `pytest tests/test_solution.py -v`
- [x] `call_openai` đã triển khai và kiểm thử
- [x] `call_openai_mini` đã triển khai và kiểm thử
- [x] `compare_models` đã triển khai và kiểm thử
- [x] `streaming_chatbot` đã triển khai và kiểm thử
- [x] `retry_with_backoff` đã triển khai và kiểm thử
- [x] `batch_compare` đã triển khai và kiểm thử
- [x] `format_comparison_table` đã triển khai và kiểm thử
- [x] `exercises.md` đã điền đầy đủ
- [x] Sao chép bài làm vào folder `solution` và đặt tên theo quy định 
