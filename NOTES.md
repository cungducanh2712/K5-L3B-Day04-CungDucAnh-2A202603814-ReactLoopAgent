# Ghi chú phân tích · Day 04

Họ và tên: Cung Đức Anh
MSSV: 2A202603814

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: Trả về 5 sự cố tháng 11/2026 và kết luận: "Vì vậy, có vẻ như không có sự cố nào từ GitHub Actions trong năm 2025 theo dữ liệu hiện tại."
- Điểm ở lần chạy 1: 40/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
{
  "event": "tool",
  "step": 1,
  "name": "get_status",
  "arguments": {"provider": "github", "component": "actions"},
  "result": {
    "matching_incidents_in_last_50": 23,
    "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
  }
}
```
Nguyên nhân: Tool chỉ quét tối đa 50 incident gần nhất (toàn bộ dữ liệu nằm ở tháng 11/2026). Agent không nêu rõ giới hạn này mà tự suy diễn kết luận rằng năm 2025 không có sự cố, vi phạm yêu cầu về độ đầy đủ của dữ liệu.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Cập nhật `SYSTEM_PROMPT` trong `agent.py`, bổ sung quy tắc ranh giới dữ liệu: khi được hỏi danh sách đầy đủ hoặc mốc thời gian cũ (như năm 2025), bắt buộc nêu rõ giới hạn 50 incident gần nhất; không tìm thấy không đồng nghĩa là không có sự cố; đồng thời hướng dẫn điều tra sâu qua nhiều bước nếu có liên kết incident liên quan.
- Vì sao chọn thay đổi này: Giúp mô hình hiểu rõ giới hạn của tools, tránh ảo giác (hallucination) và không đưa ra các khẳng định vô căn cứ khi thiếu dữ liệu.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 82.18 / 100 | 79.55 / 100 |
| Điểm case đã chọn (A5) | 40 / 100 | 75 / 100 |
| Số model calls | 74 | 73 |
| Token, chi phí nếu có | 92,357 tokens (~$0.020) | 92,266 tokens (~$0.019) |

- Lỗi đã giải quyết: Case A5 tăng vọt từ 40 lên 75 điểm; agent đã nêu rõ giới hạn 50 incident và không tự ý kết luận "năm 2025 không có sự cố". Các case E3 (+15), A2 (+20), A8 (+17), H3 (+15), H7 (+25) cũng tăng điểm đáng kể.
- Lỗi còn lại hoặc mới xuất hiện: Case H2 bị tụt điểm (100 xuống 50) do agent chỉ gọi `get_incident_details` mà bỏ sót `get_status`, dẫn tới vi phạm cơ học. Ngoài ra, tính ngẫu nhiên của Judge khiến case H4 (viết thơ) giảm từ 100 xuống 80 dù câu trả lời không đổi.

## 5. Giới hạn

- Điểm Judge: LLM Judge có độ lệch ngẫu nhiên giữa các lần chấm, không phản ánh tuyệt đối chất lượng câu trả lời.
- Dữ liệu Snapshot: Dữ liệu tĩnh tại một thời điểm, chỉ quét tối đa 50 sự cố gần nhất nên không phản ánh được lịch sử dài hạn.
- Đánh giá một lần: Một lần chạy không đo lường hết độ ổn định của agent; việc thêm prompt sửa case này có thể tạo ra regression ở case khác (như case H2).
