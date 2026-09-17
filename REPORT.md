# Báo cáo kiểm tra repository Day 5 — Segmentation Data Lab
**Mã học viên theo lớp:**  2A202602103
**Ngày / CVAT local:** 17/09/2026  
**Phạm vi:** toàn bộ cây thư mục hiện có, gồm tài liệu, mã nguồn, cấu hình CI, notebook, dữ liệu task, ảnh đầu vào và 9 ZIP trong `submissions/`.


## 1. Bài đã nộp

Tất cả ZIP trong `submissions/` đều tồn tại và được `scripts/inspect_submissions.py` đọc thành công khi chạy với UTF-8. Các số mask dưới đây là số annotation có trong export, **không phải số object/mask đúng theo đáp án**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Kiểm tra cấu trúc | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | --- | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | OK; 3 mask semantic | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | OK; 68 annotation RLE | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | OK; 98 annotation (93 RLE, 5 polygon) | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | OK; 5 annotation RLE | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | OK; 13 annotation RLE | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | OK; 41 annotation RLE | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | OK; 1 mask semantic | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | OK; 1 mask semantic | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | OK; 1 mask semantic | 3 |
| **Tổng tối đa** | **9 / 9 ZIP** | **14 / 14** | **Không có lỗi hợp đồng export** | **100** |

Phân bố annotation COCO đáng chú ý: Medium có 30 `person`, 32 `car`, 4 `motorcycle`, 2 `bus`; Hard có 54 `car`, 10 `building`, 7 `person`, 5 `vegetation`, 5 `truck` cùng các class còn lại. Các checkpoint instance tương ứng có 5, 13 và 41 annotation. Những con số này hữu ích cho QC thủ công, nhưng không thay thế đối chiếu reference.

## 2. Một quyết định trước khi dùng gợi ý

Kho lưu trữ không lưu lịch sử thao tác CVAT, thứ tự vẽ hoặc thông tin người thực hiện. Vì vậy không thể kiểm chứng trung thực object Medium nào được vẽ trước khi dùng gợi ý tự động; không nên bịa quyết định này từ ZIP.

- Object Medium đầu tiên tự vẽ: **chưa có bằng chứng trong repository**.
- Class và quy tắc chọn biên: cần người thực hiện bổ sung từ phiên CVAT; quy tắc của lab là chỉ mask phần nhìn thấy và mỗi vật đếm được là một instance riêng.
- Việc dùng gợi ý và hành động sửa/giữ: **không thể suy ra từ COCO export**.

## 3. Một lỗi tôi tìm thấy và sửa

**Lỗi môi trường đã quan sát được:** chạy `python scripts/inspect_submissions.py --dir submissions` trong PowerShell mặc định bị dừng sau ba tier do console dùng mã hóa `cp1252` không in được ký tự tiếng Việt `ỉ`. Chạy lại với `PYTHONIOENCODING=utf-8` cho kết quả đầy đủ: 9/9 ZIP là `OK`.

- Task/ảnh/vùng: bước QC toàn bộ submission, không liên quan đến một mask cụ thể.
- Loại lỗi: môi trường/mã hóa đầu ra.
- Bằng chứng: `UnicodeEncodeError: 'charmap' codec can't encode character '\u1ec9'` tại lệnh `print` của script.
- Cách xử lý đã xác nhận: trong PowerShell chạy `$env:PYTHONIOENCODING='utf-8'; python scripts/inspect_submissions.py --dir submissions`.
- Đã Save/export lại chưa: không cần; ZIP không có lỗi cấu trúc.

Chưa có điểm tự đánh giá vì reference được bảo vệ không nằm trong repo. Công cụ scoring được thiết kế để từ chối chấm khi thiếu reference, nên không diễn giải tình trạng này thành điểm 0.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` — vùng kính/khe | Khoét thành nền hoặc giữ trong mask vật | Manifest quy định windows/gaps ở trong mask; export có 5 instance | Khi QC trực quan, giữ kính/khe trong mask theo task; cần xem overlay CVAT để xác nhận biên thực tế. |
| `cp2_slice` — xe cùng class sát nhau | Một mask lớn hoặc nhiều instance | Manifest yêu cầu các xe kề nhau phải là instance riêng; export có 13 mask | Đối chiếu từng xe trong CVAT, không kết luận chỉ bằng số mask. |
| `cp5_occlusion` — vật bị che | Tách hai mảng nhìn thấy thành hai object hoặc giữ một instance | Manifest yêu cầu vật bị che vẫn là một mask/instance; export có 41 mask | Kiểm lại định danh object theo từng ảnh trước chấm; COCO export không ghi ý định của người gán nhãn. |

## Cấu trúc và thành phần chính

- `data/`: manifest, schema class/CVAT label và ảnh của 3 tier cùng 6 checkpoint. `classes.json` là nguồn chuẩn class; `cvat-labels.json` là payload để dán vào CVAT Raw Labels.
- `submissions/`: 9 export CVAT hiện diện. Semantic dùng **Segmentation mask 1.1**; instance và panoptic dùng **COCO 1.0**.
- `scripts/inspect_submissions.py`: kiểm tra tên ZIP, ảnh, class và dạng mask; không chấm biên hay số object đúng. `package_submission.py` đóng gói report/export kèm SHA-256. `convert_label_cvat.py` tái tạo label CVAT; `install_reference.py` cài package reference có kiểm soát.
- `scoring/` và `lab_utils.py`: tính mIoU cho semantic, matching IoU/precision/recall cho instance, PQ/SQ/RQ cho panoptic khi có ground truth. Điểm tự đánh giá ba tier bị giới hạn ở 82; checkpoint vẫn thuộc rubric 100.
- `notebooks/day5-segmentation-tu-kiem.ipynb`: notebook QC tùy chọn, không tạo đáp án hoặc tự chấm khi thiếu reference.
- `README.md`, `GUIDE.md`, `CVAT_SETUP.md`, `docs/`, `lab-guide.html` và `RUBRIC.md`: hướng dẫn thao tác CVAT, QC, fork/push/VLearn, rubric và tự đánh giá.
- `.github/workflows/day5-self-check.yml`: workflow tự kiểm khi push ZIP/report; sau release reference chính thức mới có thể chấm ba tier.
