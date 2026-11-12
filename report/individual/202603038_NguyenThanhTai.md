# Báo cáo thực hành Day 3

- Mã học viên theo lớp: `2A202603038`
- Ngày / CVAT local: `12/11/2026 / CVAT local`
- Công cụ đã dùng: CVAT local và `scoring/score.py`

## 1. Bài đã nộp

Tôi đã hoàn thành ba task chính và đã Save/export các ZIP theo đúng format của từng task.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Kết quả tự kiểm ba tier hiện tại là `50.5 / 82` điểm, chỉ dùng để tự QC:

- `easy_semantic`: mIoU `0.689`, coverage `89.9%`, `12.9 / 20`.
- `medium_instance`: metric `0.712`, `22.2 / 32`; mean matched IoU `0.778`, P@0.5 `0.54`, R@0.5 `0.92`; TP `65`, FP `55`, FN `6`; submitted `120` so với GT `71`.
- `hard_panoptic`: PQ `0.432`, `15.4 / 30`; SQ `0.636`, RQ `0.568`.

Chi tiết class đáng chú ý trong các lần tự kiểm:

- Easy IoU: `road 0.956`, `sidewalk 0.000`, `building 0.842`, `vegetation 0.685`, `sky 0.962`.
- Hard PQ (TP/FP/FN): `car 0.541 (25/22/1)`, `truck 0.421 (2/2/2)`, `motorcycle 0.338 (1/0/3)`, `vegetation 0.621 (2/0/0)`, `person 0.196 (3/7/6)`, `traffic light 0.519 (3/1/1)`, `sidewalk 0.000 (0/2/2)`, `bicycle 0.000 (0/0/1)`, `sky 0.941 (2/0/0)`, `building 0.652 (2/0/0)`, `bus 0.536 (2/1/1)`, `road 0.413 (1/1/1)`.

Các điểm trên là kết quả tự kiểm bằng reference hiện có, không thay cho điểm coach chấm sau.

## 2. Một quyết định trước khi dùng gợi ý

Với `medium_instance`, object đầu tiên tôi tự vẽ trước khi xem gợi ý là người đi bộ ở giữa ảnh `000000181542.jpg`, đang đi qua vạch sang đường. Tôi chọn class `person` và chỉ vẽ phần silhouette nhìn thấy của người đó; ranh mask dừng ở biên cơ thể, không gộp phần xe máy che ở hai bên vào người.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, người đi bộ ở trung tâm, khoảng từ `(190, 131)` đến `(305, 460)`.
- Class và quy tắc tôi dùng để chọn biên: `person`; bám theo đường bao cơ thể và tách khỏi xe máy, ô tô, vạch đường và nền phía sau.
- Nếu dùng gợi ý sau đó: tôi kiểm tra lại từng biên bằng ảnh gốc; vùng gợi ý ăn sang vật đang chạy ngang hoặc nền thì giữ phần người nhìn thấy và sửa/xóa phần thừa.
- Nếu không dùng gợi ý: quyết định đầu tiên được vẽ thủ công trước, sau đó các object còn lại cũng được kiểm tra theo cùng quy tắc một vật thể vật lý là một mask.

## 3. Một lỗi tôi tìm thấy và sửa

### Easy semantic

- Task/ảnh/vùng: `easy_semantic`, dải vỉa hè ở hai bên đường trong `81ae7cbb-6bc63a4a` và `817bca71-00000000`.
- Lỗi thuộc loại: phủ vùng và sai/thiếu lớp.
- Bằng chứng: class `sidewalk` không xuất hiện trong các mask đã export; scorer báo IoU `sidewalk = 0.000`, coverage tổng `89.9%`.
- Hành động sửa: cần mở lại CVAT, tô phần vỉa hè nhìn thấy phía sau bó vỉa bằng class `sidewalk`, kiểm tra không ăn sang `road`, rồi Save/export lại.
- Trạng thái: chưa Save/export lại sau khi phát hiện lỗi.

### Medium instance

- Task/vùng: `medium_instance`, các ảnh có nhiều người, xe máy và ô tô chồng lấn.
- Lỗi thuộc loại: thừa object và biên/mask dư.
- Bằng chứng: submitted `120` object trong khi GT có `71`, count error `+49`; scorer ghi nhận `55` FP dù recall đạt `0.92`.
- Hành động sửa: rà từng ảnh theo số vật thể nhìn thấy, xóa mask trùng hoặc mask không tương ứng với vật thật, tách lại các vật bị gộp và kiểm tra biên ở vùng xe/người chồng lấn.
- Trạng thái: chưa export lại sau khi phát hiện count error; `medium_instance.zip` hiện tại vẫn là bản đã chấm ở trên.

### Hard panoptic

- Task/vùng: `hard_panoptic`, các vùng sidewalk, bicycle và person trong hai ảnh.
- Lỗi thuộc loại: sai lớp, thiếu object và chất lượng biên thấp.
- Bằng chứng: `sidewalk` có PQ `0.000` với `0/2/2`, `bicycle` có PQ `0.000` với `0/0/1`; `person` có PQ `0.196` với `3/7/6`. PQ toàn task là `0.432`.
- Hành động sửa: kiểm tra lại sidewalk theo ranh bó vỉa, tìm bicycle bị bỏ sót, tách từng person khỏi xe/nền và chỉ gán phần nhìn thấy; sau đó kiểm tra chồng lấn trước khi Save/export lại.
- Trạng thái: chưa export lại sau khi phát hiện các lớp có PQ thấp.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `000000181542.jpg`, người đi bộ ở giữa vạch sang đường | Phần thân người có thể bị gộp với xe máy đang chạy ngang hoặc bị tách thành nhiều object | Người là một vật thể riêng; xe máy có đường bao và chuyển động độc lập, chỉ gán phần người nhìn thấy | Tách `person` khỏi các xe máy, không nối mask qua vùng bị vật khác che. |
| `000000373353.jpg`, nhóm người phía trước taxi/bus | Vùng nhỏ giữa người, xe và nền có thể bị hiểu là người hoặc một phần phương tiện | Dựa vào đầu, thân, chân và đường bao nhìn thấy; mỗi người đếm một mask riêng, không gộp nhóm người | Giữ từng `person` riêng; phần bị xe che không đoán tiếp phía sau. |
| `000000460147.jpg`, dải mép đường và phương tiện ở hai bên | Vùng xám sát làn xe có thể là `road`, `sidewalk` hoặc biên của xe | Ưu tiên ranh bó vỉa/làn đường; `car`/`bus` là thing riêng nằm trên stuff, không để mask stuff ăn lên phương tiện | Gán `road`/`sidewalk` theo chức năng vùng và tách từng vehicle; cần coach xác nhận các đoạn sidewalk bị cây hoặc xe che. |
