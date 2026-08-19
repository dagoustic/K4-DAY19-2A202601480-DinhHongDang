# Báo Cáo Phân Tích Ca Lỗi (Failure Analysis) — Lab 19

**Học viên:** Đinh Hồng Đăng  
**Mã học viên:** 2A202601480  

---

## 1. Ca lỗi Flat RAG thất bại — GraphRAG thành công
- **Question ID:** `G02`
- **Câu hỏi:** *"Which tech companies or executives partnered with or acquired IT solutions providers?"*
- **Triệu chứng của Flat RAG:** Flat RAG chỉ trích xuất các đoạn văn bản có độ tương đồng từ khóa cao về "partner" hoặc "acquired". Do các sự kiện mua lại được diễn đạt theo văn phong báo chí tài chính gián tiếp và phân bổ qua nhiều đoạn rời rạc, Flat RAG không liên kết được việc `Intelligent Technical Solutions` mua lại `GreenPages` cùng các đối tác phụ trợ. Điểm Comprehensiveness bị LLM Judge chấm 2/5.
- **Cách GraphRAG giải quyết:** Graph retrieval xuất phát từ seed matching các thực thể IT, duyệt BFS qua cạnh `[:ACQUIRED]` và `[:PARTNERED_WITH]`. Đồ thị tuyến tính hóa context dạng có cấu trúc kèm dẫn chứng `[chunk_id=...]`, giúp LLM sinh câu trả lời đầy đủ và đạt điểm tối đa Comprehensiveness 5/5.

---

## 2. Ca lỗi GraphRAG gặp khó khăn & Giải pháp khắc phục
- **Question ID:** `G01`
- **Câu hỏi:** *"Who was the CEO of Hugging Face in 2023?"*
- **Triệu chứng:** Khi câu hỏi yêu cầu tra cứu một thông tin đơn lẻ (Factoid) mà thực thể chưa kịp xuất hiện trong tập chunk trích xuất đồ thị giới hạn (extraction sample), Seed Matcher không tìm thấy Seed Node trên Neo4j, dẫn đến việc Graph Context bị rỗng.
- **Nguyên nhân gốc rễ (Root-cause):** Phụ thuộc hoàn toàn vào Seed Matching và độ bao phủ (coverage) của đồ thị tri thức offline.
- **Đề xuất khắc phục (Self-Correction & Fallback):**
  - Tích hợp pipeline **Hybrid Retrieval có khả năng tự sửa lỗi (Self-Correction)**: Đánh giá độ đầy đủ ngữ cảnh (`context_sufficient`). Nếu Graph retrieval trả về empty hoặc thiếu bằng chứng, hệ thống tự động fallback mở rộng Top-K Vector Search từ FAISS để bù đắp thông tin tức thì.
