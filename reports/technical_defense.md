# Thuyết Minh Kỹ Thuật (Technical Defense) — Lab 19: GraphRAG vs Flat RAG

**Học viên:** Đinh Hồng Đăng  
**Mã học viên:** 2A202601480  
**Khóa học:** AICB-K34 – Track 3: GraphRAG  
**Bộ dữ liệu Đánh giá:** `data/graphrag_golden_50_first5000.csv`  

---

### Câu 1: Cơ chế Coreference Resolution & Tình huống thực tế
- **Tình huống thực tế trong dữ liệu:** Trong các bài viết công nghệ (như chunk `43c9626ecf3e924d3289::c0000`), đoạn văn bản thường liên tiếp nhắc đến công ty cung cấp linh kiện và công ty tích hợp: *"Sineng Electric announced a partnership with onsemi. The company plans to deploy next-generation Silicon Carbide modules across its European power converters..."*
- **Rủi ro phân giải sai:** Đại từ *"The company"* hoặc *"its"* dễ bị gán nhầm cho `onsemi` thay vì chủ ngữ chính `Sineng Electric`.
- **Hậu quả trên Knowledge Graph:** Sinh ra False Edges gán sai thực thể sở hữu công nghệ hoặc hành vi mở rộng thị trường, dẫn đến suy luận sai lệch khi duyệt đồ thị đa bước (Multi-hop).

---

### Câu 2: Entity Resolution Threshold & Lexical Guard
- **Ngưỡng Cosine Similarity:** Chọn `threshold = 0.90` trên không gian embedding `sentence-transformers/all-MiniLM-L6-v2`.
- **Cặp thực thể điển hình bị Guard từ chối (REJECT_GUARD):**
  - `Sam Altman` (Person) vs `Steve Altman` (Person) (Cosine similarity = `0.875`)
  - `Apple International` (Company) vs `Apple Music` (Technology/Product) (Cosine similarity = `0.862`)
- **Lý do áp dụng Lexical Guard:** Ngăn chặn việc gộp hai thực thể hoàn toàn khác nhau nhưng có vector embedding gần nhau do tương đồng về ngữ cảnh họ hoặc tên thương hiệu.

---

### Câu 3: Top Super-nodes & Temporal Mitigation
- **Top Super-nodes thực tế trên Neo4j:**
  1. `Samsung Electronics Co. Ltd.` (Company, Degree = 4)
  2. `Renovus` (Company, Degree = 4)
  3. `Intelligent Technical Solutions` / `Walt Disney Co.` / `Ericsson` (Company, Degree = 3)
- **Ưu điểm & Rủi ro:**
  - *Ưu điểm:* Cắt tỉa `degree > 100` về $\le 50$ cạnh mới nhất giúp kiểm soát kích thước context (< 14,000 ký tự) và đảm bảo thông tin cập nhật.
  - *Rủi ro:* Có thể cắt mất các cạnh lịch sử thành lập/sáp nhập quan trọng trong quá khứ xa.

---

### Câu 4: Đánh Giá So Sánh Benchmark & Phân Tích Ca Lỗi (trên `graphrag_golden_50_first5000.csv`)
- **Kết quả Benchmark tổng hợp:**
  - *Comprehensiveness (Cross-doc):* GraphRAG đạt **4.000** so với Flat RAG **1.000** ($\Delta = +3.000$).
  - *Faithfulness (Cross-doc):* GraphRAG đạt **5.000** so với Flat RAG **1.000** ($\Delta = +4.000$).
  - *Multi-hop Reasoning (Cross-doc):* GraphRAG đạt **4.000** so với Flat RAG **1.000** ($\Delta = +3.000$).
- **Ca Flat RAG thất bại (G5000-27):** Vector search không thể xâu chuỗi và đối chiếu mốc thời gian giữa bài báo 1/6 (AMD powers cloud services) và tin Reuters 13/6 (AWS mới chỉ cân nhắc chip AMD); Graph traversal thể hiện rõ chuỗi cạnh thời gian kèm provenance.

---

### Câu 5: Đánh Đổi & Kiến Trúc Scale 350MB
- **Đánh đổi:** Flat RAG rẻ và nhanh trong việc tạo index nhưng chất lượng suy luận đa bước thấp; GraphRAG đầu tư chi phí offline extraction lớn hơn nhưng đem lại độ chính xác, khả năng suy luận và tính minh bạch vượt trội.
- **Quyết định kiểm soát AI Agent:** Từ chối thuật toán so sánh pairwise $O(N^2)$ entity matching gây tràn RAM, áp dụng FAISS Index FlatIP + Lexical Guard + Union-Find tối ưu.
- **Kiến trúc Scale 350MB:** Sử dụng Asynchronous Extraction Worker Queue (Kafka/Celery), Entity Blocking và Leiden Community Partitioning.
