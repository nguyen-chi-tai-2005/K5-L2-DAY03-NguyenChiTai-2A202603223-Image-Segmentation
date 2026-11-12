# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên: 2A202603223
- Họ tên: Nguyễn Chí Tài
- Ngày lập báo cáo: 12/11/2026
- Công cụ đã dùng: CVAT local, SAM; GitHub để lưu ZIP bài làm. Kiểm tra cấu trúc ZIP bằng `scripts/inspect_submissions.py` với sự hỗ trợ của trợ lý AI.

## 1. Bài đã nộp

Đã xuất và đưa đủ 9 ZIP lên fork cá nhân. Số ảnh dưới đây là số ảnh có trong export, không phải xác nhận mọi mask đều đúng. Hard, CP1 và CP2 còn lỗi cấu trúc cần xử lý.

| Task | File ZIP trong `submissions/` | Số ảnh trong export | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | **9 ZIP, 14 ảnh** | **14 / 14** | **100** |

Kết quả kiểm tra cấu trúc ZIP tại thời điểm lập báo cáo:

- Easy, Medium, CP3, CP4, CP5 và CP6: đạt kiểm tra cấu trúc.
- Hard: annotation 1, 3, 5 và 7 của ảnh `000000350023.jpg` thiếu dữ liệu segmentation hợp lệ.
- CP1: annotation 1, lớp `motorcycle`, ảnh `000000144300.jpg` có `segmentation: []`.
- CP2: annotation 2, lớp `car`, ảnh `000000017627.jpg` có `segmentation: []`.

Các kết quả trên không xác nhận đúng biên, đủ vật hay đúng điểm. Chưa có điểm tự đánh giá đối chiếu reference được xác minh trong báo cáo này.

## 2. Một quyết định trước khi dùng gợi ý

- Object Medium đầu tiên theo ghi nhớ của tôi: xe buýt, class `bus`; công cụ sử dụng là SAM.
- Ảnh/vị trí cụ thể: chưa xác nhận, cần mở lại task Medium để bổ sung.
- Quy tắc cần dùng để kiểm đề xuất: mỗi xe buýt là một instance, mask chỉ bao gồm phần xe nhìn thấy; không lấy nền hoặc vật khác vào mask và không tự vẽ phần bị che.
- Hiện chưa có thông tin xác nhận tôi đã tự vẽ object này trước khi xem gợi ý SAM, hoặc đã sửa/giữ đề xuất vì dấu hiệu cụ thể nào. Vì vậy báo cáo chưa khẳng định đã đáp ứng yêu cầu tự vẽ object đầu tiên trước gợi ý. Cần bổ sung đúng thao tác thực tế, không mô tả việc dùng SAM thành tự vẽ thủ công.

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** `cp1_holes`, ảnh `000000144300.jpg`, annotation 1 thuộc lớp `motorcycle`.
- **Loại lỗi:** dữ liệu export có annotation thiếu mask hợp lệ.
- **Bằng chứng:** khi kiểm tra ZIP bằng script, annotation này có `segmentation: []`, dù vẫn có `bbox` và `area`. File chứa thêm một annotation `motorcycle` có polygon; chưa thể kết luận vật bị bỏ sót chỉ từ cảnh báo này.
- **Hướng xử lý cần thực hiện:** mở lại ảnh trong CVAT, đối chiếu các object lớp `motorcycle`. Nếu object lỗi chỉ là khung thừa và vật đã có mask riêng thì xóa khung thừa; nếu chưa có mask thì tạo mask theo phần nhìn thấy. Không sửa trực tiếp JSON trong ZIP.
- **Sau sửa đã Save và export lại chưa?** Chưa xác nhận; ZIP hiện có vẫn còn lỗi. Chưa ghi nhận lỗi đã được sửa xong.
- **Phản hồi kỹ thuật:** CP1 hiện có 10 annotation, gồm 6 polygon hợp lệ, 3 RLE hợp lệ và 1 annotation không có segmentation hợp lệ. Đây là kiểm tra cấu trúc, không phải điểm chấm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Ba ca dưới đây được đối chiếu với ảnh gốc và quy tắc của repo khi soạn báo cáo, có trợ lý AI hỗ trợ diễn đạt. Chúng là những điểm cần kiểm lại trên bài làm; không phải khẳng định đã sửa hoặc đã đưa ra các quyết định này trong lúc vẽ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` — `000000144300.jpg`, phần kính chắn gió trong suốt ở đầu xe máy đỏ-trắng bên trái ảnh | Loại kính vì nhìn thấy nền phía sau, hoặc giữ kính như chi tiết thuộc xe | Phiếu quy tắc nêu kính/chi tiết trên xe không tự động là lỗ cần khoét; CP1 yêu cầu giữ kính/khe theo quy tắc task | Khi kiểm lại, giữ phần kính thuộc xe theo quy tắc CP1, phân biệt với nền nằm ngoài biên xe. Cần đối chiếu mask hiện tại để biết có phải sửa hay không. |
| `cp2_slice` — `000000017627.jpg`, cụm xe sáng màu đỗ sát nhau bên phải ảnh | Gộp vì cùng class và biên gần nhau, hoặc tách theo từng chiếc xe | Cùng class không đồng nghĩa cùng instance. Trên ảnh có thể phân biệt các thân xe và kính xe riêng | Khi kiểm lại, mỗi xe là một instance; chỉ lấy phần nhìn thấy của từng xe. Cần đối chiếu danh sách Objects để kiểm có bị gộp hoặc trùng mask không. |
| `cp4_curb` — `7d83710e-4697c3b2.jpg`, ranh mặt đường và vỉa hè quanh gốc cây phía giữa ảnh | Chọn lớp theo màu sáng/tối, hoặc chọn theo bó vỉa và chức năng bề mặt | Quy tắc road–sidewalk dựa vào chức năng/bó vỉa; ảnh có mép vỉa cong phân tách lòng đường với khu vực đi bộ | Dùng mép bó vỉa làm căn cứ khi kiểm ranh. Với ô đất quanh gốc cây, cần đối chiếu schema/quy tắc hoặc hỏi coach, không ép gán lớp chỉ để lấp kín vùng. |
