# Báo Cáo Phân Tích Ca Lỗi (Failure Analysis) — Lab 19

**Học viên:** Đinh Hồng Đăng  
**Mã học viên:** 2A202601480  
**Tập dữ liệu Benchmark:** `data/graphrag_golden_50_first5000.csv`  

---

## 1. Ca lỗi Flat RAG thất bại — GraphRAG thành công
- **Question ID:** `G5000-27`
- **Câu hỏi:** *"How should the graph reconcile the statement that AMD powers multiple cloud services with the later Reuters report about AWS considering AMD AI chips?"*
- **Triệu chứng của Flat RAG:** Flat RAG chỉ trích xuất các đoạn văn bản có độ tương đồng từ khóa cao về "AMD" hoặc "AWS". Tuy nhiên, do thông tin đến từ 2 bài báo có mốc thời gian khác nhau (ngày 1/6: AMD powers cloud services; ngày 13/6: Reuters đưa tin AWS mới đang cân nhắc chip AI của AMD), Flat RAG không thể sắp xếp và đối chiếu được dòng thời gian logic, đưa ra câu trả lời mơ hồ và bị LLM Judge chấm điểm 1/5.
- **Cách GraphRAG giải quyết:** Graph retrieval duyệt qua các cạnh đồ thị có thuộc tính thời gian `published_date` và `source_chunk_id`. Đồ thị tuyến tính hóa thứ tự diễn biến: (1) bài báo tháng 6 ghi nhận AMD chip trong cloud services chung, (2) bài báo sau đó của Reuters nêu rõ AWS mới đang ở giai đoạn đánh giá/cân nhắc chưa chốt quyết định. GraphRAG đạt điểm tuyệt đối: **Faithfulness: 5/5, Comprehensiveness: 4/5, Multi-hop reasoning: 4/5**.

---

## 2. Ca lỗi GraphRAG gặp khó khăn & Giải pháp khắc phục
- **Question ID:** `G5000-47`
- **Câu hỏi:** *"In the Keysight–Synopsys IoT cybersecurity record, does the mention of Palo Alto Networks establish Palo Alto as a partner in that deal?"*
- **Triệu chứng:** Câu hỏi Factoid yêu cầu xác định mối quan hệ phủ định (Palo Alto Networks không phải đối tác trong thương vụ Keysight–Synopsys). Nếu Seed Matcher bắt nhầm nút Palo Alto và đưa vào context các cạnh không liên quan của Palo Alto ở bài viết khác, Graph context có thể gây nhiễu và làm giảm Faithfulness.
- **Nguyên nhân gốc rễ (Root-cause):** Đồ thị Knowledge Graph trích xuất các mối quan hệ khẳng định có cấu trúc, ít khi lưu quan hệ phủ định.
- **Đề xuất khắc phục (Self-Correction & Hybrid Fusion):**
  - Tích hợp pipeline **Hybrid Retrieval có khả năng tự sửa lỗi (Self-Correction)**: Đánh giá độ đầy đủ ngữ cảnh (`context_sufficient`). Đối với các câu hỏi phân định ranh giới quan hệ hoặc kiểm tra tính hợp lệ của đối tác, tự động kết hợp phân tích văn bản gốc từ Flat Vector Search để xác nhận bối cảnh bề mặt.
