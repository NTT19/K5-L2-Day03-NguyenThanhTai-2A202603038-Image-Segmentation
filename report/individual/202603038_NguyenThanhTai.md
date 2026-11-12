# Báo cáo thực hành Day 3

- Mã học viên theo lớp: `2A202603038`
- Ngày / CVAT local: `12/11/2026 / CVAT local`
- Công cụ đã dùng: CVAT local và `scoring/score.py`

## 1. Bài đã nộp

Hiện tôi mới hoàn thành task `easy_semantic`. ZIP đã được Save và export theo format **Segmentation mask 1.1**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | chưa có | 0 / 3 | 32 |
| hard_panoptic | chưa có | 0 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Kết quả tự kiểm riêng cho `easy_semantic`: mIoU `0.689`, coverage `89.9%`, tương đương `12.9 / 20` theo script. Đây là kết quả tự kiểm, không thay cho điểm coach chấm.

## 2. Một quyết định trước khi dùng gợi ý

Phần `medium_instance` chưa được thực hiện nên tôi chưa có object Medium đầu tiên để ghi lại và chưa dùng gợi ý tự động cho task này. Với `easy_semantic`, tôi phân biệt `road` và `sidewalk` theo chức năng của vùng: mặt asphalt dành cho xe là `road`; dải lát/nâng cao phía sau bó vỉa là `sidewalk`, không chỉ dựa vào màu ảnh.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: chưa có; chưa làm `medium_instance`.
- Class và quy tắc tôi dùng để chọn biên: `road` và `sidewalk`; lấy bó vỉa và mép mặt đường làm ranh, chỉ gán phần nhìn thấy.
- Nếu dùng gợi ý sau đó: chưa áp dụng.
- Nếu không dùng gợi ý: không dùng; phần easy được tô thủ công theo quy tắc trên.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `easy_semantic`, cả ba ảnh; đặc biệt dải vỉa hè ở hai bên đường trong `81ae7cbb-6bc63a4a` và `817bca71-00000000`.
- Lỗi thuộc loại: phủ vùng và sai/thiếu lớp.
- Bằng chứng tôi nhìn thấy: class `sidewalk` không xuất hiện trong các mask đã export; scorer báo IoU của `sidewalk` là `0.000`. Coverage tổng chỉ đạt `89.9%`.
- Quy tắc và hành động sửa: cần mở lại CVAT, phóng to vùng phía sau bó vỉa, tô phần vỉa hè nhìn thấy bằng class `sidewalk`, kiểm tra không ăn sang `road`, rồi Save và export lại.
- Sau sửa đã Save và export lại chưa? Chưa; đây là lỗi phát hiện sau lần tự kiểm hiện tại.

Kết quả scorer hiện tại: mIoU `0.689`, `12.9 / 20`; IoU theo lớp: `road 0.956`, `sidewalk 0.000`, `building 0.842`, `vegetation 0.685`, `sky 0.962`. Tôi chưa ghi nhận kết quả sau sửa vì chưa export lại.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `81ae7cbb-6bc63a4a`, mép đường bên phải | Dải sáng sát nhà có thể là `sidewalk` hoặc phần nền/driveway | Có bó vỉa và dải lát liên tục dành cho người đi bộ; mặt asphalt nằm phía lòng đường | Gán dải lát phía sau bó vỉa là `sidewalk`, asphalt là `road`; không tô phần cỏ vào sidewalk. |
| `817bca71-00000000`, góc giao/đường vào nhà bên phải | Phần pavement nối vào driveway có thể là `road` hoặc `sidewalk` | Ranh nâng cao và lối đi bộ chạy sát nhà; driveway chỉ là phần chuyển tiếp, dễ bị gộp với mặt đường | Giữ ranh theo bó vỉa và chức năng lối đi; nếu coach có quy ước khác cho driveway, cần xác nhận. |
| `7ee6d192-89e2408b`, vùng xa hai bên highway | Mép màu xám có thể là phần `road` hoặc nền ngoài đường | Chỉ gán `road` khi vùng đó liên tục với mặt asphalt/làn xe; vùng đất và đồi không có bằng chứng là road | Gán phần asphalt nhìn thấy là `road`, để vùng đất ngoài mép đường không gán lớp; không suy đoán road vào vùng khuất. |
