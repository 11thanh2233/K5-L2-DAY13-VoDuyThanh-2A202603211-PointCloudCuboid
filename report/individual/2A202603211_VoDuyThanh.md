# Báo cáo Day 13 — Võ Duy Thành, 2A202603211

## 1. Kết luận trước
Mã số `2A202603211`, task `D13-2A202603211`. 10 frame: `s01f24 s01f39 s02f19 s02f53 s03f09 s03f42 s04f19 s04f44 s05f08 s05f42`. Máy: Windows 10 Pro, Chrome, CVAT server 2.74.1 (chạy local). Bạn ngồi cạnh: 2A202603118.

Bảng A có **102 cuboid** (75 Car, 14 Cyclist, 8 Pedestrian, 3 Truck, 2 Bus), 84 cuboid có tâm trong 30 m. Frame ít nhất là `s03f42` (5 cuboid): xe đen ngay phía trước che gần hết làn trước, hai bên là dải cây và đường vắng, nên chỉ còn vài vật đủ điểm. `selfcheck` cờ **1/102** cuboid (`s05f42` #6, cao 2.35 m, giữ nguyên, lý do ở §5). Tôi tin nhất phần **đáy** (khoảng hở với mặt đường ≤ 0.20 m ở mọi hộp có số đo trong phiếu tự kiểm) và hướng của xe gần; yếu nhất là **kích thước** (nhiều xe chỉ thấy một mặt nên dài/rộng lấy theo kích thước điển hình), **class** van chở hàng (Car hay Truck) và hướng của vật xa trên 30 m.

Cách dùng CVAT: đổi góc nhìn bằng chuột (cuộn để zoom, kéo trái để xoay, kéo phải để dời) và nút nhìn từ trên xuống ở khung 3D.
Vẽ cuboid dạng Shape, nháy đúp để chọn, chỉnh kích thước và vị trí ở ba khung Top, Side, Front; Save sau mỗi frame.
Bật **Cuboid orientation** để thấy mũi tên đầu xe; xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f24` (`t1-s01f24-3d`, `t1-s01f24-top`) | Phía trước (−y) ở **cạnh dưới** ảnh; +y (phía sau) chỉ lên; +x (bên trái xe) chỉ sang phải ảnh. Kiểm: sedan trắng trước-trái (x ≈ +3.3 m, y ≈ −5.8 m) nằm phải-dưới tâm | Ngã tư có vạch qua đường, gần xe phẳng, thấp hơn LiDAR ~1.7–1.8 m; phía trước-trái ~29 m thấp hơn ~2.3 m (dốc xuống nhẹ) | ~36 m (xe con phía sau) | Xe con, xe buýt, xe máy có người lái, người đi bộ | Sau-phải: xe con trắng ~6.7 m; sau-trái: xe buýt ~23 m; bên phải: nhóm xe máy chờ đèn 7–18 m | Cụm quanh quầy hàng có dù lớn bên phải và các cột đèn, cọc giao thông: không phải xe, không gán |
| `s01f39` (`t1-s01f39-3d`, `t1-s01f39-top`) | Phía trước (−y) ở **cạnh dưới**; +y chỉ lên; +x chỉ sang phải ảnh (cùng hướng trục trong CVAT như s01f24) | Đường thẳng nhiều làn, phẳng, thấp hơn LiDAR ~1.8 m; vỉa hè bên phải cao hơn mặt đường ~0.6 m | ~36 m (xe con phía sau) | Xe con, xe buýt, xe tải, xe đạp có người | Sát bên trái: sedan trắng vượt lên ~3.6 m; bên phải: người đi xe đạp trên vỉa hè sau hàng cây ~10.5 m; sau-trái: xe tải ~31 m | Cụm dài ~13 m, cao ~2.4 m bên trái ~12 m: hàng cây/dải phân cách; các cụm cao mảnh bên phải: thân cây, cột |

Chỉ camera cho biết: (1) màu và kiểu xe (sedan trắng, xe buýt, xe máy có người lái hay dựng không); (2) đầu hay đuôi xe nhờ đèn pha/đèn hậu, nhất là xe đứng yên ở s01f24; (3) đèn giao thông, vạch qua đường và quầy hàng có dù bên phải s01f24 (LiDAR chỉ thấy một cụm điểm không rõ hình).
Chỉ LiDAR cho biết: (1) khoảng cách và kích thước bằng mét (ví dụ xe buýt s01f24 cách ~23 m); (2) độ cao mặt đường (−1.7 m gần xe, ~−2.3 m ở trước-trái 29 m của s01f24; vỉa hè cao hơn ở s01f39); (3) vật ở góc mù hoặc bị cây che trên ảnh, như người đi xe đạp sau hàng cây bên phải s01f39.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f19` | Cạnh dưới (−y) | Rào giữa đường: xe bên trái (x > 0) đi ngược chiều; trạm chờ xe buýt bên phải tạo cụm lớn giống xe buýt, không gán |
| `s02f53` | Cạnh dưới (−y) | Bên trái là công viên sau lan can, nhiều thân cây tạo cụm cao mảnh giống người, không gán |
| `s03f09` | Cạnh dưới (−y) | Hàng xe chờ dày; xe sát bên trái bị gộp với rào chắn thành một cụm dài ~9 m |
| `s03f42` | Cạnh dưới (−y) | Xe đen ngay phía trước che làn trước; bên phải ~20–25 m có nền cao hơn mặt đường ~1.7 m (bờ dốc, cây bụi) |
| `s04f19` | Cạnh dưới (−y) | Ngã tư đông xe; rào thấp dài ~10 m hai bên tạo cụm giống xe, không gán |
| `s04f44` | Cạnh dưới (−y) | Ít vật; hai xe cùng chiều sát bên phải |
| `s05f08` | Cạnh dưới (−y) | Phía sau ~16 m có cụm 8.7 × 3.5 m gộp nhiều xe, chưa tách được |
| `s05f42` | Cạnh dưới (−y) | Vài cột mảnh ~0.1 m bên phải ~8 m; xe sát trước-trái rất dày điểm |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f24` #1, sedan trắng trước-trái 6.63 m | Car | Có: yaw 268.0°, cùng chiều xe thu, đuôi xe thấy trong cam01 | Có: z đáy −1.796 m, hở 0.03 m | Có | Có: 4.600 × 1.850 × 1.493 m trong khoảng sedan (3.8–5.0 × 1.70–1.90 × 1.40–1.60 m) |
| (b) xe buýt hoặc xe tải | `s01f24` #11, xe buýt sau-trái 23.11 m | Bus | Có: yaw 270.0°, đầu xe hướng về phía trước (thấy kính lái trong cam05, cam09) | Có: z đáy −1.642 m, hở 0.04 m | Có | Dài 11.500 m, cao 3.100 m trong khoảng; rộng 2.650 m vượt 2.60 m một chút (0.05 m) |
| (c) người đi bộ hoặc xe hai bánh | `s01f24` #4, xe máy có người lái bên phải 7.41 m | Cyclist | Có: yaw 44.0°, đầu xe chéo về phía trước-trái (đang rẽ qua ngã tư, đối chiếu cam07) | Có: z đáy −1.716 m, hở 0.01 m | Có | Có: 1.900 × 0.750 × 1.621 m trong khoảng xe máy có người (1.8–2.3 × 0.70–0.95 × 1.40–1.80 m) |

Vòng hiệu chuẩn: tôi mở task chung `D13-HC485087` để xem lại cách đặt hướng và đáy, và dùng hai frame của task này cho §4. Cách hiểu tôi áp dụng: xe máy có người lái là Cyclist, xe máy dựng không người thì không vẽ; van chở hàng là Truck; cụm không nhận ra trên ảnh thì không đoán.

Vật bỏ qua vì không đủ cơ sở:
- `s01f39`, sau-trái ~34 m, xe con đang chạy, ~69 điểm, không đọc được đầu/đuôi: LOW_VISIBILITY.
- `s01f39`, phía trước bên phải ~27 m, nhóm xe đạp dựng trên vỉa hè, không thấy người lái: không vẽ (không phải Cyclist).
- `s02f19`, bên phải ~24 m, cụm dài ~13 m cao ~3.6 m: trạm chờ/mặt tiền, không phải xe buýt. Trước-trái ~28 m: bảng quảng cáo, không phải người.
- `s02f53`, bên trái ~25–27 m sau lan can công viên, hai cụm cao ~1.5–1.8 m: trên ảnh chỉ thấy thân cây: UNCERTAIN_CLASS, không vẽ.
- `s03f09`, phía sau ~19 m giữa hai làn xe chờ, cụm ~33 điểm bị che: LOW_VISIBILITY.
- `s03f42`, phía trước ~19 m sau xe đen, cụm ~26 điểm bị che: LOW_VISIBILITY.
- `s05f08`, phía sau-trái ~16 m, cụm 8.7 × 3.5 m gộp nhiều xe sát nhau, chưa tách được ranh giới: UNCERTAIN_SCOPE.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng                           | Frame / tên ảnh / nguồn                                                | Nguyên nhân (giả thuyết)                                                                           | Thấy gì trên point cloud                                                                                                                          | Hệ quả khi gán (theo guideline)                                                                                                                                                                       |
| ------------------------------------ | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Bóng che                          | `s02f00`, `t4-1-bong-che`, **mượn từ task chung `D13-HC485087`**       | Giả thuyết: xe buýt đỏ sát bên trái (~4 m) chặn tia laser, tia không tới được vùng phía sau nó     | Sau xe buýt có một hình quạt gần như không có điểm; hai bên hình quạt vẫn dày điểm mặt đường                                                      | Vùng trống là **chưa quan sát được**, không phải đường trống. Vật nằm một phần trong bóng che thì bật `occluded` và vẽ hộp bao phần suy luận được; vật hoàn toàn trong bóng thì không vẽ              |
| 2. Thưa ở xa                         | `s04f19` #17 (xe con phía sau-phải 38.77 m), `t4-2-thua-o-xa`          | Giả thuyết: khoảng cách giữa các tia tăng theo khoảng cách, nên ở xa chỉ vài tia chạm xe           | Chỉ 70 điểm trong hộp, chủ yếu ở mặt trước xe; cùng frame, xe ở 8.34 m (#1) có 2250 điểm                                                          | Vẫn thấy được hình dạng mặt trước và ảnh cam09 có xe, nên vẽ hộp theo kích thước điển hình (quy tắc 15–20%). Vật ít hơn nữa (ví dụ `s03f42` #5, 18 điểm ở 30.38 m) là LOW_VISIBILITY, ghi vào báo cáo |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f19` #2 (xe MPV màu đen sát bên trái 10.27 m), `t4-3-phan-xa-thap` | Giả thuyết: sơn đen và kính tối hấp thụ phần lớn tia laser nên cường độ phản hồi thấp              | Cường độ trung bình của điểm trong hộp ~0.02, trong khi xe màu trắng `s04f19` #5 (13.17 m) ~0.31; hình dạng vẫn đủ điểm (2685 điểm) vì xe rất gần | Chỉ ghi giả thuyết, không khẳng định. Không nới hộp theo cường độ; vẽ theo hình dạng điểm và ảnh camera                                                                                               |
| 4. Điểm lạ                           | `s02f35`, `t4-4-diem-la`, **mượn từ task chung `D13-HC485087`**        | Giả thuyết: tia phản xạ nhiều lần (ví dụ qua bề mặt kính hoặc nước) nên LiDAR tính sai khoảng cách | Cụm ~46 điểm phía sau bên trái ~29 m, ở z ≈ −13.5 m, thấp hơn mặt đường quanh đó ~11 m; chỉ thấy rõ khi xoay nghiêng khung 3D                     | Không gán thành vật, không vẽ cuboid                                                                                                                                                                  |

## 5. Số từ cuboid của tôi (Bước 5)
Số lấy từ `submission/D13-2A202603211-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603211-tukiem.csv`. Mặt đường khoảng −1.7 m: z đáy lệch hơn 0.3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f24` #1 Car | −1.050 m | 1.493 m | −1.796 m | 6.63 m | 6.71 m | 268.0° | Chạm đường (lệch −0.10 m so với −1.7 m; phiếu tự kiểm hở 0.03 m) |
| `s01f24` #13 Car | −1.408 m | 1.724 m | −2.270 m | 29.20 m | 29.23 m | 272.0° | Thấp hơn −1.7 m tới 0.57 m nhưng **không chìm**: đường ở trước-trái dốc xuống, phiếu tự kiểm hở 0.03 m với mặt đường tại chỗ |
| `s01f39` #2 Cyclist | −0.484 m | 1.433 m | −1.201 m | 10.53 m | 10.54 m | 264.0° | Cao hơn −1.7 m tới 0.50 m nhưng **không lơ lửng**: người đi xe đạp trên vỉa hè cao; hở 0.00 m với mặt vỉa hè |

(a) Gần nhất: `s01f39` #1 Car, `dist_m` = 3.56 m, r = 3.72 m. Xa nhất: `s04f19` #17 Car, `dist_m` = 38.77 m, r = 38.78 m. Ở gần, r lớn hơn `dist_m` rõ hơn vì z (−1.07 m) đáng kể so với 3.56 m; ở xa, z gần như không đổi r.

(b) Người đi bộ `s02f19` #1: l 0.607 m, w 0.600 m, h 1.673 m, cả ba trong khoảng người lớn (0.4–0.9 × 0.5–0.9 × 1.50–1.95 m). Xe con `s05f42` #2: l 4.707 m, w 1.850 m, h 2.112 m. Dài và rộng trong khoảng SUV/minivan (4.4–5.3 × 1.80–2.10 m); cao 2.112 m vượt 2.10 m 0.012 m. Lý do có thể: hộp bao cả giá nóc hoặc vài điểm nhiễu trên nóc xe sát LiDAR (6666 điểm); sai số nhỏ hơn 1%, tôi giữ.

(c) Xe cùng chiều phía trước: `s03f42` #2 (ngay trước 10.76 m) yaw 270.0°, đúng ~270°. Xe ngược chiều: `s02f53` #1 (xe đỏ trước-trái 9.64 m) yaw 90.0°, đúng ~90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s04f44` #1 Car | 6.97 m | 3081 |
| `s01f24` #6 Car | 12.33 m | 1334 |
| `s01f24` #10 Car | 19.01 m | 434 |
| `s04f19` #17 Car | 38.77 m | 70 |

Quy luật: xa gấp khoảng 2.7 lần (6.97 → 19.01 m) thì số điểm còn khoảng 1/7 (3081 → 434); xa gấp 5.6 lần (6.97 → 38.77 m) thì còn khoảng 1/44. Số điểm giảm nhanh hơn 1/d², vì xe xa còn bị xe gần che một phần và chỉ một mặt quay về LiDAR. Bốn số này cùng dạng với đường cong ở slide (1968 ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m): xe 38.77 m của tôi có 70 điểm, gần đúng mức 72 của khoảng 30–40 m.

Tự kiểm, các cuboid đã sửa sau `preview` / `selfcheck` (sau khi sửa: Save, `sync` lại):
- `s02f53`, xe SUV phía sau-phải ~31 m: `selfcheck` báo cao 2.33 m; `preview` cho thấy hộp trùm cả tán cây phía trên. Hộp không bám đúng xe, đã **xoá**.
- `s03f09`, cụm cột cách ~33 m gán Pedestrian: `selfcheck` báo đáy cao hơn mặt đường 0.51 m; ảnh cho thấy cột, không có người. Đã **xoá**.
- `s02f19`, `s02f53`: tám xe bên kia rào giữa đường (ví dụ `s02f19` #2, `s02f53` #2) bị đặt yaw ~270° hoặc ~294°. Ảnh cam01 thấy đầu xe, cam09 thấy đuôi xe, nên đổi yaw về ~90° (ngược chiều).
- `s02f19` #6: người ngồi trên xe máy bị gán Pedestrian, đổi thành Cyclist. `s04f19` #3: xe máy có người phía sau, đổi Pedestrian thành Cyclist.
- `s03f09` #6 và #9: hai xe van chở hàng, đổi Car thành Truck, yaw 270°.
- Vẽ thêm các xe gần còn sót (ví dụ `s03f09` #1–#3, `s04f19` #2, `s03f42` #2), tổng từ 98 lên 102 cuboid.

Cảnh báo giữ nguyên: `s05f42` #6 Car cao 2.349 m (ngoài 1.2–2.3 m của Car). Đây là xe phía trước bên phải 19.48 m có thân cao; điểm LiDAR ở nóc lên tới mức này. Tôi chưa chắc đây là SUV cao hay van chở hàng (UNCERTAIN_CLASS), nên giữ Car và ghi lại ở đây thay vì chỉnh chiều cao để tắt cờ.

Đổi kiểm: chưa thực hiện được với bạn ngồi cạnh; phần này để trống, không ghi số chưa đối chiếu.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model phát hiện 3D học dự đoán vị trí, kích thước và hướng vật bằng mét trong hệ toạ độ xe, để bộ lập kế hoạch tính được khoảng cách phanh và làn đi; nhãn bằng pixel thì đổi theo camera và không cho biết vật cách bao xa. Vật thưa điểm cần quy tắc riêng vì model chỉ thấy vài chục điểm: nếu lúc vẽ lúc bỏ, hoặc hộp co theo phần điểm thấy được, model sẽ học sai kích thước và học bỏ sót vật ở xa.

(b) Dùng AI: Không có.

(c) Điều còn chưa chắc:
- Kích thước dài và rộng của phần lớn xe lấy theo kích thước điển hình (4.60 × 1.85 m), vì LiDAR thường chỉ thấy một hoặc hai mặt xe; xe dài hơn hoặc ngắn hơn sẽ lệch tâm hộp.
- Hướng của vật xa trên 30 m và của người đi bộ đứng yên chưa chắc (ví dụ `s05f08` #10, #11).
- Class của xe thân cao (`s05f42` #6) và hai xe van chở hàng ở `s03f09` (Car hay Truck): UNCERTAIN_CLASS.
