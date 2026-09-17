# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên VLearn. Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: D5_2A202602168
- Ngày / CVAT local: 2026-09-17 / CVAT chạy local qua Docker Compose
- Công cụ đã dùng: Brush/Polygon vẽ tay trong CVAT; có thử AI Tools (model EoMT Cityscapes Semantic) cho phần semantic nhưng gặp lỗi 504 Gateway Time-out (model CPU chạy quá chậm so với timeout), nên chuyển hẳn sang vẽ tay

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh VLearn đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task                     | File ZIP đúng tên |                                           Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| ------------------------ | -------------------- | ---------------------------------------------------------------: | --------------------------------: |
| easy_semantic            | easy_semantic.zip    |                                                       3 / 3 (OK) |                                20 |
| medium_instance          | medium_instance.zip  |                                        3 / 3 (OK, 66 annotation) |                                32 |
| hard_panoptic            | hard_panoptic.zip    | 2 / 2 (cấu trúc OK, nhưng thiếu stuff class — xem ghi chú) |                                30 |
| cp1_holes                | cp1_holes.zip        |                                         1 / 1 (OK, 6 annotation) |                                 3 |
| cp2_slice                | cp2_slice.zip        |                                        1 / 1 (OK, 14 annotation) |                                 3 |
| cp5_occlusion            | cp5_occlusion.zip    |                                        1 / 1 (OK, 28 annotation) |                                 3 |
| cp3_thin                 | cp3_thin.zip         |                         1 / 1 nhưng**LỖI** — xem mục 3 |                                 3 |
| cp4_curb                 | cp4_curb.zip         |                       1 / 1 nhưng**LỖI** — xem ghi chú |                                 3 |
| cp6_coverage             | cp6_coverage.zip     |  1 / 1 (đọc được, nhưng thiếu car/person — xem ghi chú) |                                 3 |
| **Tổng tối đa** |                      |                                                                  |                     **100** |

Ghi chú QC (chạy `python scripts/inspect_submissions.py`, tương đương notebook BƯỚC 3–7), tất cả 9/9 ZIP đã có trong `submissions/`:

- `hard_panoptic.zip`: COCO hợp lệ (46 annotation, toàn polygon), nhưng **chưa thấy stuff class nào ngoài road** (thiếu building, sidewalk, sky, vegetation). Cần kiểm lại CVAT xem đã vẽ các vùng stuff này chưa.
- `cp3_thin.zip`: **LỖI hợp đồng** — labelmap chứa nhãn ngoài `classes.json` (bicycle, building, bus, car, motorcycle, person, sidewalk, traffic light, truck, vegetation), đồng thời **hoàn toàn không có `pole` và `traffic sign`** — đúng 2 class trọng tâm của checkpoint "nét mảnh". Soi pixel: 70% ảnh là background chưa gán, 12% bị gán nhầm thành `car`. Nghi ngờ đã dùng nhầm label set (bộ Cityscapes đầy đủ) cho task này. Cần vẽ lại đúng `pole`/`traffic sign`/`sky`/`road`.
- `cp4_curb.zip`: cùng kiểu lỗi labelmap thừa nhãn, và **thiếu hẳn `sidewalk`** — đúng class cần so sánh với `road` theo yêu cầu bó vỉa. 66% ảnh là background, 19% bị gán nhầm `car`.
- `cp6_coverage.zip`: cấu trúc đọc được, nhưng **`car` và `person` chưa xuất hiện** dù nằm trong 7 class bắt buộc; 38.76% ảnh vẫn là background — đáng chú ý vì đây chính là checkpoint kiểm tra độ phủ toàn ảnh.

3 file `cp3_thin`, `cp4_curb`, `cp6_coverage` nên được mở lại trong CVAT và sửa trước khi nộp chính thức.

Nếu export lỗi, ghi task, dữ liệu đã Save đến đâu và lỗi đã báo coach.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg` (annotation id=1), một người đứng sát mép phải khung hình, bbox khoảng (599,134)–(611,153) — vật thể rất nhỏ (khoảng 12×19px).
- Class và quy tắc tôi dùng để chọn biên: class `person`. Vì object nằm sát mép ảnh và khá nhỏ, quy tắc tôi áp dụng là chỉ khoanh đúng phần cơ thể còn nhìn thấy trong khung hình, dừng mask tại đúng cạnh ảnh thay vì đoán phần bị cắt ra ngoài.
- Nếu dùng gợi ý sau đó: không dùng gợi ý tự động cho `medium_instance` — AI Tools trong CVAT ở máy tôi chỉ chạy ổn cho vài model, phần lớn thời gian tôi vẽ tay bằng Polygon để chắc từng người/xe được tách đúng theo ranh nhìn thấy.
- Nếu không dùng gợi ý: không dùng; quyết định gán nhãn ở trên (dừng mask theo mép ảnh, không đoán phần khuất) là ví dụ cụ thể.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `cp3_thin`, ảnh `839f7736-abe28069.jpg`, toàn bộ ảnh.
- Lỗi thuộc loại: sai lớp + thiếu vật (task yêu cầu `pole`, `road`, `sky`, `traffic sign` nhưng mask lại dùng nhầm bộ nhãn Cityscapes đầy đủ).
- Bằng chứng tôi nhìn thấy: chạy `python scripts/inspect_submissions.py` báo `[LỖI] cp3_thin` với "label ngoài classes.json: bicycle, building, bus, car, motorcycle, person, sidewalk, traffic light, truck, vegetation". Soi thêm pixel trong mask thì thấy 70.13% ảnh là `background` (chưa gán), 17.49% `road`, 12.21% bị gán nhầm thành `car`, chỉ 0.17% là `traffic light` — và hoàn toàn **không có pixel nào là `pole` hay `traffic sign`**, đúng hai class trọng tâm của checkpoint "nét mảnh".
- Quy tắc và hành động sửa: cần mở lại task `cp3_thin` trong CVAT, xóa các nhãn thừa không thuộc `classes.json` của trạm này, sau đó vẽ đúng cột đèn/biển báo bằng brush cỡ 2–3px theo đúng quy tắc "Thin structures" trong `data/manifest.json`, và phủ nốt phần `sky`/`road` còn thiếu.
- Sau sửa đã Save và export lại chưa? **Chưa** — lỗi này mới được phát hiện qua script tự kiểm, tôi ghi lại đây để nhớ sửa trước khi nộp, chưa kịp quay lại CVAT sửa và export lại ZIP.

Chưa chạy self-check trên GitHub Actions / chưa có điểm scorecard ba tier cho lần export này.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể                                                                            | Quy tắc/chứng cứ                                                                                                         | Quyết định hoặc câu hỏi cho coach                                                                                |
| ------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 1             | `medium_instance` / `000000458325.jpg`, hai xe car (id56, id64) chồng bbox tới 52%            | Có thể là 1 xe dài bị chia đôi khi vẽ, hoặc 2 xe đậu sát/che nhau                                               | Quy tắc`cp2_slice`/`cp5_occlusion`: xe cùng lớp sát nhau vẫn phải tách 2 instance riêng                    |
| 2             | `hard_panoptic` / `000000460147.jpg`, `car` (id38) chồng lên vùng `road` (id40) IoU=0.26 | Mask`road` có nên trừ phần bị xe che, hay cứ vẽ đủ road rồi để car đè lên trên                            | Nguyên tắc panoptic: mỗi pixel cuối cùng chỉ thuộc 1 nhãn, thing thường ưu tiên hơn stuff ở vùng chồng |
| 3             | `medium_instance` / `000000181542.jpg`, `car` (id2) chồng lên `bus` (id10) IoU=0.25       | Xe con đậu sát/che một phần trước đầu xe bus — có tính là 2 vật thể riêng hay gộp vì cùng nhóm "xe cộ" | Quy tắc chung: khác class thì luôn là 2 instance riêng, không gộp theo nhóm lớn                              |
