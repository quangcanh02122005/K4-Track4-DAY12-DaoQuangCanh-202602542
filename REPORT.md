# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | `<điền>` |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `<điền>` |
| GitHub | `<điền: tên tài khoản>` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `<điền>` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `<điền>` |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.259 | 21.2 | chặn |
| IDM | 1.556 | 0.1 | (baseline) |
| EgoMLP | 0.755 ± 0.005 | 16.5 .. 30.7 | chặn |
| PercMLP | 0.616 ± 0.003 | 0.8 .. 3.5 | chặn (R2–R6) |
| PercMLP+ (sau khi sửa) | 0.837 ± 0.012 | 0.2 .. 0.5 | qua 7/7 |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

(trả lời ở đây)

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

(trả lời ở đây)

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

(trả lời ở đây)

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

(trả lời ở đây)

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

(trả lời ở đây)

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

(trả lời ở đây)

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP rớt 5/7 quy tắc (`evidence/gate.json`): R2 va chạm trên log 3,5 % (cận trên 4,2 %) so với IDM 0,1 % + 1 điểm; R3 lát phanh 14,3 % so với ngưỡng 5 điểm; R5 độ ổn định giữa seed 0,8 .. 3,5 %; R4 lưới cắt vào 24,4 % vùng tránh được bị đâm (IDM 0 %); R6 xe sau phản ứng 31,8 % lưới (IDM 7,4 %). Có hai nguyên nhân gốc: (a) R4 và R6 là kịch bản **log không chứa** (cắt vào ở tốc độ tiến gần lớn), mô hình chưa từng thấy nên không học được; (b) R2, R3, R5 là **sai số tích luỹ**: sai số nhỏ khi bắt chước đẩy xe vào trạng thái người lái chưa từng đi qua.

Cách sửa (`step_fix`): thêm 20 000 kịch bản cắt vào mô phỏng với nhãn IDM (chữa nguyên nhân a), và 2 vòng DAgger (`augment.dagger_windows`): cho mô hình tự lái trên log huấn luyện, ghi các trạng thái nó tự đi vào, gắn nhãn bằng dự báo 3 s của IDM ở đúng các trạng thái đó rồi huấn luyện lại (chữa nguyên nhân b). Kết quả PercMLP+ (5 seed, cùng quy tắc, không đổi ngưỡng): R2 0,5 % (cận trên 0,8 %), R3 0,6 %, R4 0,1 %, R5 0,2 .. 0,5 %, R6 7,6 % so với IDM 7,4 % => qua 7/7. Ở tốc độ cruise 20 m/s chưa từng thấy khi huấn luyện (<= 16), PercMLP+ còn 0 .. 2,1 % lưới bị đâm so với 17,4 .. 30,2 % của PercMLP, IDM 0 %.

PercMLP+ **chưa tốt hơn** IDM: va chạm trên log 0,2 .. 0,5 % so với 0,1 % của IDM, R6 xấu hơn chút (7,6 % so với 7,4 %). Nó học từ nhãn IDM nên chỉ có thể xấp xỉ thầy, không có tín hiệu để vượt thầy. L2 vòng hở của PercMLP+ (0,837 m) còn **tệ hơn** PercMLP (0,616 m), tức là sửa làm điểm vòng hở xấu đi trong khi lái vòng kín an toàn hơn nhiều: L2 vòng hở không đo được độ an toàn. Qua cổng chỉ nghĩa là không bị chặn bởi các quy tắc này, không phải bằng chứng nên thay IDM. Đề xuất không phát hành thay IDM; trước đó cần giữ IDM làm baseline, kiểm tra thêm trên kịch bản và tốc độ ngoài vùng huấn luyện (kết quả 20 m/s cho thấy còn lỗi), dữ liệu thật nhiều hơn và chạy bóng (shadow mode) so với IDM.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(trả lời ở đây)
