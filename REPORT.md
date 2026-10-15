# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Đào Quang Cảnh |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602542 (lệnh đã chạy với `--student-id 202602542`, xem `evidence/*.json`) |
| GitHub | quangcanh02122005 (https://github.com/quangcanh02122005/K4-Track4-DAY12-DaoQuangCanh-202602542) |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 1638 / 3716 (các seed huấn luyện 3716..3720) |
| Số seed huấn luyện bạn dùng (`--seeds`) | 5 |

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

Từ `evidence/data_audit.json`: 992 679 dòng, 9 993 quỹ đạo thô, **383 quỹ đạo trùng hệt nhau** bị loại còn 9 610. Các bản trùng nằm ở id cách xa nhau, nên nếu chia ngẫu nhiên theo dòng hay theo id thì cùng một quỹ đạo có thể rơi vào cả train lẫn test, làm điểm test cao giả (rò rỉ); vì vậy chia theo quỹ đạo sau khi loại trùng. Cột `Acc_LV` không dùng được: tương quan với d(vận tốc)/dt chỉ 0,045 (trong khi `Acc_follower` là 0,929), nên mình suy gia tốc xe dẫn từ `Speed_LV` đã làm mượt. Ngoài ra `gap` đúng bằng hiệu vị trí trừ chiều dài xe (độ lệch trung bình 4,5 m, sai lệch tối đa 3,6e-11 m) và dữ liệu 10 Hz, mỗi quỹ đạo 70–109 bước.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Bản refactor float32 **khác bit ở 600/600 đoạn** nhưng **vượt dung sai ở 0 đoạn** (sai lệch tối đa 1,2e-4 so với dung sai 1e-3 m/s²): khác bit không đồng nghĩa khác hành vi, vì thứ tự cộng float khác nhau cho kết quả khác chút (ví dụ cộng 50 000 số float32 xuôi/ngược lệch tương đối 6,9e-6). Chạy lại y hệt cho 0 đoạn khác. Thay đổi AEB làm **9/600 đoạn** vượt dung sai (các đoạn 59, 77, 179, 201, 238, 244, 353, 368, 578), lệch tối đa 1,98 m/s²; lệch chỉ ở những đoạn có tình huống phanh khẩn nơi AEB can thiệp, còn các đoạn khác không đổi (mình suy ra từ cấu trúc kết quả, chưa mở riêng từng khung). Phát nhanh nhất cho 0 đoạn lệch, còn **phát theo thời gian thật** làm rơi 573 và trễ 582 trong 59 712 khung, kéo 382/600 đoạn vượt dung sai (lệch tối đa 15,7 m/s²): phát nhanh che mất lỗi về thời gian thực.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới: cruise 12 m/s, khoảng cách 2,76–59,3 m, dv 0–12 m/s, 2 700 ô. Ở độ trễ 0 s có 2,2 % ô không ai tránh được, ở 0,6 s tăng lên 7,4 % (vùng đâm của IDM đúng bằng vùng này vì IDM không lỗi ở phần tránh được). Log **không phủ** vùng khó: giới hạn phanh mạnh nhất của người lái (p99,9) chỉ 3,62 m/s² (p99 của các sự kiện phanh là 4,21 m/s²), trong khi 8,6 % lưới đòi hỏi phanh mạnh hơn mức đó; chỉ có 614 sự kiện phanh (6,4 % quỹ đạo). Constant-velocity đâm ở 77,6 % lưới.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 vòng hở (m) | Hạng | Va chạm vòng kín (%) | Hạng |
|---|---|---|---|---|
| PercMLP | 0,616 | 1 | 1,5 (0,8 .. 3,5) | 2 |
| EgoMLP | 0,755 | 2 | 26,9 (16,5 .. 30,7) | 4 |
| constant-velocity | 1,259 | 3 | 21,2 | 3 |
| IDM | 1,556 | 4 | 0,1 | 1 |

Các cặp đổi thứ tự: const-vel/IDM, const-vel/EgoMLP, IDM/EgoMLP, IDM/PercMLP. IDM có L2 tệ nhất vì nó là luật phản ứng chứ không bắt chước người lái, nên dự báo vận tốc 3 s lệch khỏi người thật, nhưng cách lái của nó vẫn giữ khoảng cách an toàn nên va chạm chỉ 0,1 %. EgoMLP chỉ thấy vận tốc của chính nó, không thấy xe dẫn nên không biết lúc nào phải phanh: dự báo trung bình tốt (L2 0,755 < 1,259) nhưng phanh gần như không (mức phanh đỉnh trung vị 0,04–0,12), nên đâm.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

EgoMLP dao động **16,5 .. 30,7 %** giữa 5 seed (khoảng 14 điểm). Khoảng tin cậy Wilson của mỗi seed rất hẹp (ví dụ seed 3716: 29,1–32,4 %; seed 3720: 15,2–17,9 %) và các khoảng của seed 3716 và 3720 không giao nhau: Wilson chỉ phản ánh độ ngẫu nhiên do chọn mẫu test, không phản ánh ngẫu nhiên do huấn luyện. Nếu chỉ chạy một seed mình có thể báo 16,5 % (hoặc 30,7 %) kèm khoảng tin cậy hẹp trông rất chắc, nhưng con số đó phụ thuộc may rủi của seed; nên phải báo dải qua các seed.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

`T` hiệu chỉnh = 1,4 s (khoảng cách Wasserstein tới phân bố của người là 0,354 m, so với 0,875 của tác nhân không phản ứng; headway trung vị của người là 1,87 s), xe sau cách 18,8 m, trễ phản ứng 0,6 s. Bảng ở t_react = 0,6 (phần lưới bị đâm, phía trước / phía sau):

| | xe sau phát lại (trước / sau / tổng) | xe sau phản ứng (trước / sau / tổng) |
|---|---|---|
| const-vel | 77,6 / 0 / 77,6 | 77,6 / 0 / 77,6 |
| IDM | 7,4 / 68,3 / 75,7 | 7,4 / 0 / 7,4 |
| EgoMLP | 81,9 / 0 / 81,9 | 81,9 / 0 / 81,9 |
| PercMLP | 30,0 / 52,9 / 82,7 | 31,8 / 0 / 31,8 |
| PercMLP+ | 7,6 / 67,7 / 75,3 | 7,6 / 0 / 7,6 |

Khi xe sau chỉ phát lại log, nó không phản ứng việc IDM phanh gấp nên đâm vào đuôi IDM, và IDM bị tính "va chạm giả" (68,3 % lưới) dù nó phanh đúng; IDM trông ngang constant-velocity (75,7 % so với 77,6 %). Với xe sau phản ứng, IDM chỉ còn 7,4 %, tốt nhất, và các bộ học (PercMLP 31,8 %, EgoMLP 81,9 %) mới lộ rõ khác biệt. Kết luận đúng chỉ ra khi thế giới có phản ứng.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP rớt 5/7 quy tắc (`evidence/gate.json`): R2 va chạm trên log 3,5 % (cận trên 4,2 %) so với IDM 0,1 % + 1 điểm; R3 lát phanh 14,3 % so với ngưỡng 5 điểm; R5 độ ổn định giữa seed 0,8 .. 3,5 %; R4 lưới cắt vào 24,4 % vùng tránh được bị đâm (IDM 0 %); R6 xe sau phản ứng 31,8 % lưới (IDM 7,4 %). Có hai nguyên nhân gốc: (a) R4 và R6 là kịch bản **log không chứa** (cắt vào ở tốc độ tiến gần lớn), mô hình chưa từng thấy nên không học được; (b) R2, R3, R5 là **sai số tích luỹ**: sai số nhỏ khi bắt chước đẩy xe vào trạng thái người lái chưa từng đi qua.

Cách sửa (`step_fix`): thêm 20 000 kịch bản cắt vào mô phỏng với nhãn IDM (chữa nguyên nhân a), và 2 vòng DAgger (`augment.dagger_windows`): cho mô hình tự lái trên log huấn luyện, ghi các trạng thái nó tự đi vào, gắn nhãn bằng dự báo 3 s của IDM ở đúng các trạng thái đó rồi huấn luyện lại (chữa nguyên nhân b). Kết quả PercMLP+ (5 seed, cùng quy tắc, không đổi ngưỡng): R2 0,5 % (cận trên 0,8 %), R3 0,6 %, R4 0,1 %, R5 0,2 .. 0,5 %, R6 7,6 % so với IDM 7,4 % => qua 7/7. Ở tốc độ cruise 20 m/s chưa từng thấy khi huấn luyện (<= 16), PercMLP+ còn 0 .. 2,1 % lưới bị đâm so với 17,4 .. 30,2 % của PercMLP, IDM 0 %.

PercMLP+ **chưa tốt hơn** IDM: va chạm trên log 0,2 .. 0,5 % so với 0,1 % của IDM, R6 xấu hơn chút (7,6 % so với 7,4 %). Nó học từ nhãn IDM nên chỉ có thể xấp xỉ thầy, không có tín hiệu để vượt thầy. L2 vòng hở của PercMLP+ (0,837 m) còn **tệ hơn** PercMLP (0,616 m), tức là sửa làm điểm vòng hở xấu đi trong khi lái vòng kín an toàn hơn nhiều: L2 vòng hở không đo được độ an toàn. Qua cổng chỉ nghĩa là không bị chặn bởi các quy tắc này, không phải bằng chứng nên thay IDM. Đề xuất không phát hành thay IDM; trước đó cần giữ IDM làm baseline, kiểm tra thêm trên kịch bản và tốc độ ngoài vùng huấn luyện (kết quả 20 m/s cho thấy còn lỗi), dữ liệu thật nhiều hơn và chạy bóng (shadow mode) so với IDM.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(1) **Mô hình một chiều, không cảm biến**: không có làn, cắt vào ngang, nhiễu và độ trễ cảm biến, nên "không va chạm" ở đây không đảm bảo trong thực tế. (2) **Xe dẫn trong log không phản ứng với xe thử** và xe sau là giả định của mình (IDM với T = 1,4 s hiệu chỉnh từ một thống kê, trễ 0,6 s): kết luận về xe sau (câu 6) phụ thuộc vào giả định này, kết quả có thể khác với người thật. (3) **Phạm vi dữ liệu hẹp**: 9 610 quỹ đạo từ một nguồn, tốc độ xe dẫn đến khoảng 16 m/s, chỉ một tốc độ cruise 12 m/s cho lưới; ở 20 m/s ngoài vùng huấn luyện PercMLP+ vẫn còn đâm 0–2,1 % lưới. (4) **Ngưỡng cổng là lựa chọn của đội** (1 điểm, 5 điểm, 2 điểm), và PercMLP+ chỉ xấp xỉ IDM nên "qua cổng" chưa có nghĩa là an toàn hơn. Để thu hẹp khoảng cách cần dữ liệu thật đa dạng hơn (tốc độ, thời tiết, tình huống hiếm), mô phỏng có cảm biến và nhiều tác nhân, chạy bóng (shadow mode) trên xe thật so với IDM, rồi mới thử có kiểm soát.
