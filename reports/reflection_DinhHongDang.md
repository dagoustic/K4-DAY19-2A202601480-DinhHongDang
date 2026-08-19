# Báo Cáo Suy Ngẫm & Kế Hoạch Đồ Án (Reflection & Action Plan)

**Học viên:** Đinh Hồng Đăng  
**Mã học viên:** 2A202601480  
**Khóa học:** AICB-K34 – Track 3: GraphRAG  

---

## 1. Bảng Mapping Bài Giảng vào Thực Thi Code
| Khái niệm trong bài giảng | Module tương ứng | Hàm / Khối code cụ thể | Đánh giá & Quan sát thực tế |
|:---|:---:|:---|:---|
| **Conservative Coreference** | Module 1 | `resolve_coref_batch()` | Giữ nguyên các entity mơ hồ và ghi nhận vào `unresolved_mentions` để tránh sinh False Facts. |
| **Schema & Allowlist Guard** | Module 2 | `ALLOWED_NODE_TYPES`, `ALLOWED_RELATIONS` | Đảm bảo chuẩn hóa 3 nhãn node và 8 quan hệ chuẩn mực, loại bỏ nhiễu ngữ nghĩa. |
| **Bulk Cypher Ingestion** | Module 2 | `bulk_insert_nodes()`, `bulk_insert_edges()` | Tối ưu hóa throughput nạp đồ thị qua `UNWIND $rows AS row`, nạp 111 nodes & 67 edges < 3 giây. |
| **Entity Resolution & Union-Find** | Module 3 | `build_resolution_map()`, `UF` | Ghép alias thành công (Google LLC -> Google), kết hợp Lexical Guard chặn gộp nhầm. |
| **Super-node Degree Cap** | Module 4 | `retrieve_graph_context()`, `recent_edges()` | Cắt tỉa các node degree cao về $\le 50$ cạnh mới nhất, tránh bùng nổ độ dài context. |
| **LLM-as-a-Judge Evaluation** | Module 5 | `judge_answer()`, `judge_json()` | Chấm điểm tự động và độc lập trên 3 thang điểm Comprehensiveness, Faithfulness, Multi-hop. |

---

## 2. Quá trình Debugging & Bài Học Kỹ Thuật
- **Kinh nghiệm gỡ lỗi API:** Khi gọi API trích xuất theo batch lớn, cần thiết lập cơ chế retry với Exponential Backoff để ứng phó với biến động độ trễ mạng và giới hạn tốc độ (Rate Limit).
- **Kiểm soát tính toàn vẹn dữ liệu:** Luôn gắn chặt `source_chunk_id` và `published_date` trên từng cạnh để đảm bảo 100% tính minh bạch dẫn chứng (Provenance Integrity) khi trích xuất câu trả lời.

---

## 3. Kế Hoạch Áp Dụng vào Đồ Án Thực Tế (Action Plan)
- **Tên dự án:** Hệ thống Trợ lý Pháp lý & Tra cứu Quy chế Doanh nghiệp dựa trên Knowledge Graph.
- **Lý do chọn GraphRAG:** Các văn bản quy phạm pháp luật và hợp đồng có mối liên hệ điều khoản chéo (`REFERENCES`, `SUPERSEDES`, `AMENDS`). GraphRAG giúp người dùng tra cứu toàn bộ chuỗi quy định liên đới thay vì chỉ trích xuất các đoạn văn bản rời rạc.
- **Cấu trúc Đồ thị:**
  - *Nodes:* `LawDocument`, `Article`, `Penalty`, `Organization`.
  - *Relations:* `REFERENCES`, `SUPERSEDES`, `APPLIES_TO`, `ENFORCES`.
- **Giải pháp Super-node & Entity Resolution:** Phân cấp theo hiệu lực thời gian và chuẩn hóa Primary Key bằng mã số văn bản chính thống.
