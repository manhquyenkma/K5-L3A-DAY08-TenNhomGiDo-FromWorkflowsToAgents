# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, Run ID: `E-seeded_regression-20261117-0967` |
| Triệu chứng | `db_fact_kept` fail (False), `refund_amount_correct` fail (35.000đ vs 0đ), `pass_rate` fail; reply báo *"LogiX đã hoàn 35.000đ cho đơn O-202"* và ledger ghi hoàn 35.000đ trong khi nhãn kỳ vọng là denied (0đ). |
| Bước sai đầu tiên | Event #3 — `model_override` `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event #1 (`handoff`) khởi tạo đúng contract; Event #2 (`investigator`) truy vấn đúng dữ liệu DB thực tế: đơn `O-202` có `delay_days: 0`, `status: delivered_late=False`. Bước #3 tự ý lấy số ngày 5 do khách khai trong ticket đè lên DB fact. |
| Failure mode | FM-03: `ticket_trust — tin ticket hơn DB` |
| Đối chứng | Cùng claim `C-002@11` trên shape `fixed`: Bước #3 không xuất hiện `model_override`. Agent giữ nguyên `delay_days: 0` từ DB, tính quote bồi thường 0đ, host từ chối hoàn tiền và kết quả khớp 100% với nhãn kỳ vọng. |
| Regression test | Grader `DbFactKept` trong `src/lab9/graders.py` kiểm tra `events` không chứa `model_override`; unit test `test_override_event_breaks_db_fact` trong `tests/test_lab9_graders.py`. Grader phát hiện lỗi trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi đè dữ liệu; không xác định được LLM thật sự "tin khách" do reasoning hay do system prompt bị thiếu ràng buộc ưu tiên nguồn dữ liệu xác thực (system of record). |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
