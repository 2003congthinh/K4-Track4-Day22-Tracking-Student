# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** ………………………… **Thành viên:** Thinh (2003congthinh), …………………………

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.7 | Detector chỉ thấy khoảng 6/22 người mỗi frame ở conf 0.3, nên phần lớn người ở xa hoặc bị che không có hộp. Người gần camera giữ ID ổn định. iou 0.7 giữ được hộp của hai người đứng sát nhau. | `bytetrack` 0.3/0.5: HOTA 26.9, bỏ sót nhiều người vì chỉ mở track mới khi conf ≥ 0.5. `ocsort` 0.3/0.5: HOTA 27.5, 42 lần đổi ID. `strongsort` 0.3/0.5: HOTA 28.7. `deepocsort` 0.3/0.5: HOTA 27.4, 51 lần đổi ID. `botsort` conf 0.5: HOTA 27.2, mất nhiều người. |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.15 | 0.5 | Đám đông ở cuối phố quá nhỏ, không tracker nào có hộp. Người ở nửa dưới ảnh giữ cùng màu ID suốt đoạn xem. conf 0.15 bắt thêm người đeo balo ở góc phải trên (ID 21) mà conf 0.3 bỏ sót. Không thấy hộp giả trên đèn đường hay biển hiệu sáng. | `bytetrack` 0.3: người đang đi giữa phố (frame 120) không có hộp. `ocsort` / `strongsort` / `deepocsort` 0.3: track bị đứt 59–67 lần so với 30 của botsort trên 200 frame đầu. `botsort` conf 0.3: 30 lần đứt so với 12 ở conf 0.15. |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.15 | 0.5 | Hai người sát camera (áo sọc, áo khoác xám) giữ ID 1 và cùng màu suốt đoạn chờ qua đường. Người bị cắt ở mép trái giữ một ID từ frame 80 đến 160. conf 0.15 bắt thêm người nhỏ ở vạch qua đường phía xa. Nửa sau là phố đi bộ đông, người đi sát camera bị cắt ở mép ảnh nên ID tăng nhanh (161 ID / 837 frame). | `ocsort` 0.3: người ở mép trái đổi ID 23 → 14, track đứt 80 lần. `deepocsort` 0.3: 69 lần đứt. `botsort` conf 0.5: mất người ở mép trái và người ở xa. |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.15 | 0.5 | Hai người đi trước camera (áo đỏ, áo trắng) giữ ID suốt đoạn xem dù camera tiến lên. Không thấy hộp trên bóng phản chiếu ở sàn và kính. Đổi iou 0.4 / 0.5 / 0.7 cho kết quả giống hệt, vì người ít chồng lên nhau. | `bytetrack` 0.3: người áo đen đi về phía camera (frame 80) không có hộp. `ocsort` 0.3: 22 ID và 48 lần đứt, so với 19 ID và 27 lần đứt của botsort. `botsort` conf 0.3: 27 lần đứt so với 10 ở conf 0.15. |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.15 | 0.5 | Người áo khoác đỏ trên vỉa hè giữ ID 2 từ frame 60 đến 130 trong khi xe bus chạy. Đám đông ở giao lộ phía xa quá nhỏ, không có hộp. | `ocsort` 0.3: mất hộp của người áo xám đang đi bộ (frame 150–199), track đứt 74 lần. `deepocsort` 0.3: 71 lần đứt. `botsort` conf 0.5: mất người đeo balo (ID 43) ở frame 130, 70 lần đứt. |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: thinh_video1-pedestrian      HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
COMBINED                           29.969    18.408    49.061    19.176    74.385    52.38     80.959    83.019    30.624    37.181    76.409    28.409

CLEAR: thinh_video1-pedestrian     MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag
COMBINED                           19.025    80.817    19.202    22.491    87.244    14.516    17.742    67.742    14.71     4179      14402     611       33        9         11        42        105

Identity: thinh_video1-pedestrian  IDF1      IDR       IDP       IDTP      IDFN      IDFP
COMBINED                           29.703    18.68     72.463    3471      15110     1319

Count: thinh_video1-pedestrian     Dets      GT_Dets   IDs       GT_IDs
COMBINED                           4790      18581     55        62
```

Bảng so sánh các cấu hình đã chấm trên `video_1` (đủ 600 frame):

| Tracker | conf | iou | HOTA | MOTA | IDF1 | IDSW | FP |
|---|---|---|---|---|---|---|---|
| bytetrack | 0.3 | 0.5 | 26.91 | 17.29 | 25.71 | 12 | 107 |
| ocsort | 0.3 | 0.5 | 27.46 | 19.81 | 28.73 | 42 | 253 |
| botsort | 0.3 | 0.5 | 29.46 | 19.81 | 29.35 | 25 | 337 |
| strongsort | 0.3 | 0.5 | 28.66 | 19.70 | 29.85 | 41 | 231 |
| deepocsort | 0.3 | 0.5 | 27.38 | 19.76 | 27.80 | 51 | 248 |
| botsort | 0.15 | 0.5 | 29.34 | 20.73 | 29.56 | 27 | 505 |
| botsort | 0.5 | 0.5 | 27.17 | 15.25 | 24.56 | 10 | 229 |
| botsort | 0.3 | 0.4 | 29.32 | 19.47 | 29.82 | 19 | 195 |
| **botsort (nộp)** | **0.3** | **0.7** | **29.97** | 19.03 | 29.70 | 33 | 611 |
| botsort | 0.15 | 0.7 | 29.66 | 20.33 | 29.88 | 39 | 700 |
| botsort | 0.2 | 0.7 | 29.45 | 19.92 | 29.16 | 35 | 660 |

Chọn theo HOTA (một số tổng, cân bằng phát hiện và giữ ID). Các cấu hình botsort tốt nhất chỉ chênh nhau dưới 0.7 HOTA. Điểm thấp ở mọi cấu hình chủ yếu do detector: recall chỉ khoảng 22% (DetRe 19.2), trong khi khả năng giữ ID khá hơn nhiều (AssA 49.1).

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- Tracker đã chọn giữ ID tốt hơn, hay ít hộp giả hơn, ở điểm nào bạn nhìn thấy?
- Cảnh đó (đứng yên / chuyển động, đông / thưa, sáng / tối, trong nhà / ngoài trời) khiến tracker này hợp hơn tracker kia như thế nào?

Cách đánh giá video không nhãn: với mỗi video, chạy cả 5 tracker trên 200 frame đầu và xem video có vẽ ID. Ngoài ra đếm từ file `.txt`: số ID, số track ngắn dưới 10 frame, và số lần một track biến mất rồi quay lại ("đứt"). Trên `video_1` có nhãn, tracker có ít lần đứt nhất trong nhóm có recall cao (botsort) cũng là tracker có HOTA cao nhất, nên số này dùng được làm chỉ báo phụ.

**video_1 (quảng trường, camera tĩnh, có nhãn).** Bảng số cho thấy vấn đề lớn nhất không nằm ở tracker mà ở detector: YOLO26n ở 640 px chỉ thấy khoảng 6 trong 22 người mỗi frame, nên DetA chỉ còn 18.4 trong khi AssA đạt 49.1. `bytetrack` có ít lần đổi ID nhất (12) nhưng HOTA thấp nhất (26.9). Lý do là trong cấu hình của boxmot, ByteTrack chỉ mở track mới khi conf ≥ 0.5, mà phần lớn người trong cảnh có conf thấp hơn, nên tracker này bỏ sót nhiều người. `botsort` mở track từ conf khoảng 0.21 và giữ track bị mất tới 60 frame, nên vừa bắt được nhiều người hơn, vừa đổi ID ít hơn `ocsort` / `strongsort` / `deepocsort` (25 so với 41–51 lần). Tăng `iou` lên 0.7 giữ được hộp của những người đứng sát nhau (recall 22.5%), đổi lại số hộp giả tăng (611 FP).

**video_2 (phố tối, camera tĩnh trên cao, rất đông; chỉ xem bằng mắt).** Camera đứng yên nên cả tracker theo chuyển động cũng chạy được, nhưng người đi cắt ngang nhau liên tục. Ở cảnh này Re-ID và bộ nhớ 60 frame của `botsort` giúp giữ ID: người đàn ông đội mũ đứng cạnh hàng xe máy giữ **ID 4 suốt cả 1050 frame**, kể cả khi có người đi sát ngay trước mặt (frame 400). Cảnh tối làm conf của detector thấp, nên hạ `conf` xuống 0.15 bắt thêm người mà không thấy hộp giả trên đèn đường hay biển hiệu sáng. Số lần đứt trên 200 frame đầu giảm từ 30 xuống 12. Đám đông nhỏ ở cuối phố thì không cấu hình nào bắt được, vì đó là giới hạn của ảnh 640 px.

**video_4 (trong nhà, camera tiến về phía trước, sàn và kính phản chiếu; chỉ xem bằng mắt).** Camera chuyển động làm mọi hộp trôi trong ảnh, nên tracker chỉ dựa vào vị trí (Kalman + IoU) như `ocsort` dễ ghép sai (22 ID và 48 lần đứt trên 200 frame, so với 19 ID và 27 lần đứt của botsort). `botsort` có bù chuyển động camera (sparse optical flow) trước khi ghép hộp, cộng thêm Re-ID, nên người áo đỏ đi trước camera giữ **ID 2 liên tục từ frame 1 đến 709**. Sàn bóng và kính không sinh hộp giả ở conf 0.15, nên giảm conf chỉ có lợi (thêm người ở xa, số lần đứt giảm từ 27 xuống 10). Đổi `iou` không có tác dụng vì ít người chồng lên nhau.

**video_3 và video_5 (camera di chuyển; chỉ xem bằng mắt).** Lý do tương tự video_4: bù chuyển động camera giúp `botsort` giữ ID người ở mép ảnh (video_3) và người đi bộ trên vỉa hè khi xe bus chạy (video_5), trong khi `ocsort` đổi ID hoặc mất hộp. Tuy vậy, nửa sau video_3 là phố đi bộ đông, người đi sát camera và bị cắt ở mép ảnh, khung hình lại thưa. Ở đoạn này ID tăng nhanh: 161 ID trong 837 frame, nhiều track ngắn. Đây là đoạn yếu nhất của bài nộp.

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).

Giới hạn lớn nhất là recall của detector, nên bước tiếp theo là quét `conf` mịn hơn (0.1–0.25) cùng `iou` 0.6–0.7 cho từng video. Ngoài ra sẽ xem kỹ nửa sau video_3 để biết ID tăng nhanh do người ra vào mép ảnh hay do Re-ID nhầm, rồi thử tracker có `track_buffer` dài hơn. Phần này chỉ là mở rộng, không đổi detector hay Re-ID trong bài nộp chính.
