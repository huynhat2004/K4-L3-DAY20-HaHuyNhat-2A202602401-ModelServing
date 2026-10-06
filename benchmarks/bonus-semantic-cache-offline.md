# Bonus B5/C8 - Chế độ semantic cache (offline)

Demo offline dùng vector bag-of-words và mô phỏng inference tốn 250 ms. Tại threshold
0.80, cache hit 3/8 lần (38%), bỏ qua ba lần gọi model và tiết kiệm khoảng 750 ms decode
mô phỏng. Sweep threshold từ 0.70 tới 0.95 giữ nguyên 3/8 vì similarity của phần mô phỏng
này gần như chỉ nhận 0 hoặc 1.

Lần chạy này kiểm chứng control flow serving, không kiểm chứng chất lượng ngữ nghĩa. Một
cache hit bỏ qua hoàn toàn prefill và decode, khác prefix/KV cache, nhưng phần mô phỏng
không thể hiện ranh giới false hit và false miss thực tế. Thí nghiệm production cần sentence
encoder như BGE-M3 hoặc Qwen3-Embedding, key có salt theo tenant và tập paraphrase/non-match
đã gán nhãn. Cache dùng chung không có salt còn có thể rò rỉ thông tin giữa tenant qua nội
dung hoặc timing. Đường cong phẳng là bằng chứng về giới hạn của stub, không chứng minh
việc chọn threshold là không quan trọng.
