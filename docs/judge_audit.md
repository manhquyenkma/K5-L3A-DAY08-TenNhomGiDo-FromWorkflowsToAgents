# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757, P=0.857, R=0.800) | Dùng làm gate |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.300 < 0.7) | Loại khỏi gate |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Đề xuất sửa Rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* — Xưng hô "anh - em" và từ "nha" quá suồng sã, thiếu văn phong chuẩn mực của CSKH LogiX. Cần bổ sung ví dụ tiêu cực vào prompt rubric tone. |
| 2 | H-017 | PASS | FAIL | Người đúng | Reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* — Xưng hô "bạn ơi" và dùng từ đuôi thân mật. Cần cập nhật rubric quy định rõ đại từ nhân xưng trang trọng ("anh/chị - LogiX"). |
| 3 | H-029 | PASS | FAIL | Người đúng | Reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* — Giọng điệu cộc lốc, mang tính xua đuổi khách hàng ("không cần nhắn thêm"). Rubric cần cấm ngôn từ áp đặt, thiếu đồng cảm. |
| 4 | H-042 | FAIL | PASS | Người đúng | Reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* — Câu trả lời nhã nhặn, đầy đủ thông tin thời gian nhận tiền. Judge bắt bẻ nhầm (FP). Cần tinh chỉnh rubric để không phạt câu có thông tin thời gian xử lý. |
| 5 | HL-015 | PASS | FAIL | Người đúng | Reply: *"Dạ, LogiX đã hoàn 55.000đ cho đơn O-112 ạ."* — Khách yêu cầu giải trình lý do giao trễ và gửi hóa đơn, agent chỉ báo tiền mà không giải thích. Judge Helpful bị mù lỗi (FN) khiến Recall chỉ đạt 30%. |

## Kết luận
- **Tiêu chí bị loại:** `judge_helpful` bị loại hoàn toàn khỏi CI Quality Gate. Ma trận nhầm lẫn của `judge_helpful` là `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Tỉ lệ bỏ sót lỗi (False Negative) lên tới 21/30 (Recall chỉ đạt 0.3, trong khi sàn yêu cầu ≥ 0.7; Cohen's Kappa chỉ đạt 0.375 < 0.6). Nếu dùng judge này làm gate, 70% các phản hồi vô dụng sẽ lọt qua kiểm duyệt.
- **Tiêu chí được chọn:** `judge_tone` đạt đủ các ngưỡng sàn khắt khe (κ = 0.757 ≥ 0.6, Precision = 0.857 ≥ 0.8, Recall = 0.800 ≥ 0.7), ma trận nhầm lẫn `{'tp': 24, 'fp': 4, 'fn': 6, 'tn': 66}`, tỉ lệ lỗi quan sát 28% được Rogan-Gladen hiệu chỉnh về 30%.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ được đo trên tập dữ liệu replay đã công khai (public replay dataset). Để triển khai production an toàn, cần có một tập held-out mới do con người gắn nhãn độc lập để tránh hiện tượng overfit rubric.
