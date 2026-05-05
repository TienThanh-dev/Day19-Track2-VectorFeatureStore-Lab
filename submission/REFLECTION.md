# Reflection — Lab 19

**Tên:** Vũ Tiến Thành
**Cohort:** 01
**Path đã chạy:** lite

---

## Câu hỏi (≤ 200 chữ)

> Trên golden set 50 queries, mode nào thắng ở loại query nào (`exact` /
> `paraphrase` / `mixed`), và tại sao? Khi nào bạn **không** dùng hybrid
> (i.e. khi nào pure BM25 hoặc pure vector là lựa chọn đúng)?

**Kết quả trên golden set:**
- **Exact queries** (15 câu): BM25 đạt 96.7%, hybrid 96.7%, vector 88.7%. BM25
  thắng vì query chứa đúng từ kỹ thuật verbatim (Kubernetes, OAuth, PostgreSQL...)
  mà corpus có sẵn — BM25 không cần hiểu ngữ nghĩa, chỉ cần trùng từ.
- **Paraphrase queries** (15 câu): BM25 33.3%, hybrid 32.0%, vector 24.0%.
  Cả 3 đều yếu vì dùng tiếng Việt paraphrase hoàn toàn không trùng từ corpus.
  Trên corpus 1000 docs này, `bge-small-en-v1.5` (English-trained) không đủ
  mạnh cho tiếng Việt. Đổi sang `bge-m3` multilingual sẽ cải thiện đáng kể.
- **Mixed queries** (20 câu): hybrid đạt **100.0%**, vector 98.5%, BM25 97.0%.
  Đây là kết quả ấn tượng nhất — hybrid thắng tuyệt đối vì kết hợp cả signal
  exact (BM25) lẫn ý nghĩa (vector).

**Trung bình:** hybrid 78.6% > BM25 77.8% > vector 73.2%.

**Khi KHÔNG dùng hybrid:**
1. Khi latency budget cực kỳ khắt khe — BM25 keyword ~3ms, hybrid ~10-15ms.
   Nếu cần P50 < 1ms (ví dụ autocomplete), hybrid không phù hợp.
2. Khi infrastructure không cho phép vector DB — chỉ cần SQLite, không có
   Qdrant/Milvus, thì BM25 là lựa chọn duy nhất.
3. Khi truy vấn là 100% exact technical term và corpus nhỏ — BM25 đủ tốt,
   vector overhead không đáng giá.
4. Khi dùng embedding model yếu cho ngôn ngữ đó (như bge-small-en cho tiếng
   Việt), vector signal gây nhiễu — lúc đó BM25 thuần có thể tốt hơn hybrid.

---

## Điều ngạc nhiên nhất khi làm lab này

Điều ngạc nhiên nhất là **hybrid RRF chỉ cần 14.8ms P99** trên server — nhanh
hơn cả pure semantic (150.8ms). Lý do là RRF gọi semantic với depth=50 rồi
BM25 cũng chạy depth=50, nhưng BM25 rất nhẹ, nên hybrid chỉ chậm hơn keyword
chút xíu mà vẫn nhanh hơn semantic thuần. Điều này có nghĩa trong thực tế,
**hybrid không đánh đổi latency** — ta nên luôn default hybrid.

