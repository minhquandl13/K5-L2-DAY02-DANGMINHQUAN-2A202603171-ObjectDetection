Báo cáo cá nhân — Ngày 2 Object Detection

1. Thông tin học viên
   Họ tên: Đặng Minh Quân
   MSSV: 2A202603171
   Email: minhquandl13@gmail.com
   Lớp: (điền mã lớp)
   Link repo bài làm: https://github.com/minhquandl13/K5-L2-DAY02-DANGMINHQUAN-2A202603171-ObjectDetection
   Link Colab đã chạy: (dán link notebook Colab, mở quyền xem cho người chấm)
2. Schema và mini guideline
   Mini guideline đầy đủ: GUIDELINE_MINI.md.
   Schema cố định của bài (không đổi): 0 car, 1 truck, 2 bus, 3 van, tất cả là rectangle. Mỗi xe một box, box bám sát phần thân xe nhìn thấy; xe bị cắt bởi rìa ảnh thì box dừng ở rìa; xe bị che khuất vẫn đánh box cho phần nhìn thấy; không đánh dấu người đi bộ, xe máy, biển báo, vệt phản quang; khung không có xe thì để file .txt rỗng.
   Ba ca mơ hồ và quyết định đã áp dụng:
   Xe van cỡ nhỏ, vuông, màu trắng (dạng microvan) — van hay truck? Quyết định: nếu thùng xe kín, dạng hộp, dùng cho chở hàng thì gán truck; nếu là xe khách cỡ nhỏ có cửa kính liên tục thì gán van. Căn cứ: hình dáng thùng và kiểu cửa hậu, không dựa vào màu.
   Xe khách dài (coach) — bus hay van? Quyết định: chỉ gán bus khi thấy rõ dạng xe buýt thành phố (cửa lên xuống bên hông, chiều cao tầng, biển tuyến). Các xe khách đường dài, xe giường nằm, xe 45 chỗ thông thường gán van.
   Xe bị cắt mạnh bởi rìa ảnh (chỉ thấy một phần đầu hoặc hông) — có đánh box không? Quyết định: vẫn đánh, box dừng ở rìa ảnh, không mở rộng ra ngoài khung hình. Chỉ bỏ qua khi phần nhìn thấy quá nhỏ để phân biệt với vệt sáng hoặc bóng xe.
   Điểm cần giảng viên làm rõ: trong tập nhãn tham chiếu, rất nhiều xe buýt thành phố (đỏ/trắng, xanh/trắng) lại được gán van, và chỉ một số ít được gán bus. Vì vậy ranh giới bus/van không mang tính mô tả khách quan mà phụ thuộc quy ước của người gán nhãn. Nếu có định nghĩa chính thức (theo loại xe đăng ký) thì nên ghi rõ trong guideline để các bài nộp so sánh được với nhau.
3. Annotation và self-QC
   Số file nhãn train: 50 file drive_001.txt … drive_050.txt, tương ứng 1-1 với 50 ảnh trong dataset/images/train.
   Kết quả checker và lỗi đã xử lý: chạy python tools/check_yolo_labels.py cho kết quả ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Ngoài checker, tôi tự kiểm thêm: đúng 5 cột mỗi dòng, class id chỉ trong 0–3, w/h nằm trong (0, 1], không có file rỗng ngoài ý muốn, không có toạ độ âm.
   Danh sách ít nhất 10 ảnh đã rà bằng mắt: drive_001, drive_003, drive_006, drive_007, drive_008, drive_009, drive_012, drive_014, drive_027, drive_028, drive_031, drive_041, drive_044, drive_049.
   Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
   19 cặp box trùng trên cùng một xe nhưng khác class (dấu vết của việc đổi class mà quên xoá box cũ; ví dụ drive_016 dòng 23 truck và dòng 26 bus trùng khít, IoU = 1.00). Cách sửa: xoá box cũ, giữ box đã đổi class.
   Nhầm van/truck ở xe van trắng dạng hộp (10 ca, ví dụ drive_033 dòng 5 là xe khách đỏ bị cắt bởi rìa ảnh nên phải là van, không phải truck). Cách sửa: đổi class theo hình dáng thùng/cửa.
   Đánh dấu thừa quá nhiều xe ở vùng xa, dày (dải xe xếp hàng phía trên khung hình và bãi đỗ bên đường). Cách sửa: xoá các box không tương ứng với vật thể nào trong nhãn tham chiếu, xem mục 4.
   Nhầm car/truck ở microvan trắng và bus/van ở xe khách dài; sửa theo quy tắc ở mục 2.
   Commit khóa nhãn ban đầu: b1a3472 (update annotation) — trạng thái nhãn đầu tiên đã đẩy lên GitHub trước khi nhận gold.
4. Gold QC và rework
   Chỉ số
   Trước sửa
   Sau sửa nếu có
   TP
   469
   555
   FP
   563
   225
   FN
   184
   98
   Precision
   0.4545
   0.7115
   Recall
   0.7182
   0.8499
   F1
   0.5567
   0.7746
   mean_iou_of_matches
   0.8434
   0.8358
   passed
   false
   false

Kết quả JSON: outputs/annotation_evaluation.json. Ngưỡng của bài là F1 ≥ 0.80 và mean IoU ≥ 0.70, nên lần chấm cuối vẫn chưa đạt (F1 0.7746 < 0.80; mean IoU 0.8358 đã đạt). Tôi ghi đúng kết quả chưa đạt thay vì chỉnh số.
Rework đã làm, theo từng nhóm và có kiểm bằng ảnh chứ không suy từ FP/FN:
Ảnh đã kiểm: dựng trang duyệt cho cả 50 frame, mỗi frame là một ảnh chồng lớp bốn màu (xanh dương = box tham chiếu, xanh lá = box của tôi đã khớp, đỏ = box thừa, vàng = box bị bỏ sót), kèm số đếm từng frame để xếp thứ tự sửa. Vì vậy nhận định "thừa box" là quan sát trực tiếp, không phải suy luận từ con số FP.
Quyết định sửa hay giữ: với vùng xe xếp hàng phía xa và bãi đỗ bên đường, nhãn tham chiếu không đánh dấu các xe này; tôi quyết định xoá các box tương ứng (khoảng 400 box), vì đó là lý do chính khiến precision thấp (0.45 → 0.71) trong khi mean IoU vẫn cao (~0.84, tức box đã vẽ thì vẽ đúng). Với các box đã khớp tốt ở gần, tôi giữ nguyên, không nới hay siết box.
Ca lỗi class: 91 → 78 lỗi class còn lại ở lần chấm cuối, nhóm lớn nhất là car→truck (29), bus→van (26), truck→bus (19). Mỗi ca được cắt riêng thành ảnh phóng to, ghi rõ tên file và số dòng, có viền đỏ (class tôi gán) và viền cam (class tham chiếu) để đối chiếu trước khi sửa trong CVAT.
Những gì còn thiếu: khoảng 157 box thừa và 29 xe bị bỏ sót vẫn còn khi dừng rework; nếu xoá/sửa hết thì F1 ước tính quanh 0.90, nhưng tôi dừng ở mức đã ghi trong bảng trên. 5. Kết quả YOLO
Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, CPU (Intel Xeon @ 2.20 GHz — phiên chạy không chọn runtime GPU T4), ultralytics 8.4.208, torch 2.14.1+cpu; 50 epoch, tổng thời gian train 0.570 giờ.
Các tham số thay đổi so với notebook nếu có và lý do: không đổi tham số nào. Notebook tự clone repo từ REPO_URL, tự ghi lại path tuyệt đối trong data.yaml, và tự assert 50 ảnh train + 10 ảnh eval khớp nhãn. yolo26n.pt, epochs=50, imgsz=640, batch=-1, seed=20260909 giữ nguyên.
Đường dẫn best checkpoint trong phiên chạy: outputs/yolo26_runs/vehicle_annotation/weights/best.pt (không nộp file .pt, chỉ ghi đường dẫn).
Chỉ số trên 10 ảnh eval
Giá trị
Precision
0.4596
Recall
0.2351
mAP50
0.2735
mAP50–95
0.2033

Hai folder kết quả nằm cạnh báo cáo: assets/eval_predictions/ (10 ảnh eval có box dự đoán) và assets/yolo26_runs/ (log train, results.csv, confusion matrix, PR/F1 curve).
Ảnh và vật thể cụ thể: assets/eval_predictions/visuals/drive_052.jpg — chiếc xe buýt hai tầng lớn đang dừng ở trạm (giữa khung hình) không có box dự đoán nào, trong khi khoảng 10 xe con quanh đó đều được phát hiện với confidence 0.6–0.88. Đây là ví dụ rõ nhất cho việc model bỏ sót lớp bus.
Trả lời bốn câu hỏi cuối notebook:
Lớp nào recall thấp nhất? Ảnh nào minh họa? bus, recall = 0.000 (van 0.083, truck 0.200, car 0.657). Ảnh minh họa: assets/eval_predictions/visuals/drive_052.jpg — xe buýt hai tầng ở trạm không được phát hiện, các xe con xung quanh thì có. Nguyên nhân hợp lý: bus là lớp hiếm nhất trong tập train và ranh giới với van không rõ, nên model vừa đoán "bus" lan rộng (dự đoán 2–6 "buss" ở drive_052, drive_053, drive_058) vừa không khớp được hai instance thật trong eval.
Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box? Kiểm độ "chặt" của box: box có thừa biên (vẽ rộng hơn thân xe), lệch tâm, hoặc sai tỉ lệ ở xe bị cắt bởi rìa ảnh hay không. mAP50–95 lấy trung bình từ IoU 0.50 đến 0.95, nên box chỉ cần lệch một chút là điểm tụt mạnh dù vẫn "đúng". Dữ kiện trong bài: car có mAP50 0.725 nhưng mAP50–95 chỉ 0.514 (giảm 0.21) — tức xe con bị phát hiện nhưng box chưa khít. Điều này khớp với nhãn người vẽ: mean IoU của nhãn so với tham chiếu là 0.836, không phải 0.95, cho thấy box được vẽ kèm một khoảng biên nhỏ, và model học lại đúng khoảng biên đó.
Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét? Nhóm van ↔ bus ↔ truck: 26 ca bus→van, 19 ca truck→bus, 18–29 ca car→truck. Dữ kiện: nhãn train của tôi có 106 box bus trong khi tập tham chiếu chỉ có khoảng 20 instance bus trên toàn bộ 50 ảnh — tức tôi gán bus dư khoảng 5 lần, nên model học "xe dài = bus" và hệ quả là precision/recall lớp bus bằng 0. Nghĩa là vấn đề không nằm ở việc vẽ box mà ở định nghĩa lớp: quy tắc "xe khách dài mặc định là van" chỉ được áp dụng sau khi đối chiếu tham chiếu, đúng ra phải nằm trong guideline ngay từ đầu. Ví dụ cụ thể: drive_009 dòng 8 (xe buýt đỏ/trắng thành phố), drive_033 dòng 5 (xe khách đỏ bị cắt bởi rìa ảnh).
Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì? Vì chúng là tập đo lường giữ riêng: nếu đưa vào train thì con số mAP chỉ còn đo khả năng ghi nhớ, không còn đo khả năng tổng quát hoá, và kết quả của các bài nộp không so sánh được với nhau. Trong notebook, 10 ảnh này là val: của data.yaml, nên chúng còn được dùng để chọn best.pt qua từng epoch — nghĩa là vai trò thực tế là validation set, không phải test set độc lập. Notebook cũng assert 50 ảnh train khớp 50 nhãn và 10 ảnh eval khớp 10 nhãn, đảm bảo không có ảnh eval nào lẫn vào train. 6. Khó khăn và cách xử lý
Bước/thời điểm
Lỗi quan sát
Cách đã thử
Kết quả/còn thiếu
Gán nhãn trong CVAT → file trên máy
Sửa trong CVAT nhưng .txt trong repo không đổi; chấm lại vẫn ra đúng con số cũ
Nhận ra Ctrl+S chỉ lưu task, không ghi file; phải Export task dataset (định dạng YOLO 1.1, Save images = No) rồi chép obj_train_data/\*.txt đè lên dataset/labels/train
Đã xử lý; từ đó kiểm bằng thời gian sửa file của 50 .txt trước mỗi lần chấm
Thiết kế lớp bus/van/truck
Ở 640 px không phân biệt được xe khách dài và xe buýt thành phố; nhãn tham chiếu cũng không nhất quán
Cắt từng ca thành ảnh phóng to, đối chiếu cặp class, đưa ra quy tắc "xe khách dài mặc định van"
Đã giảm bus dư từ 121 xuống 106 box; ranh giới vẫn còn mơ hồ, cần định nghĩa chính thức
Đánh dấu thừa vùng xa
Model precision thấp dù box vẽ đúng (mean IoU 0.84)
Xếp hạng từng frame theo số box thừa, xoá theo thứ tự frame nặng nhất trước
F1 0.5567 → 0.7746; còn khoảng 157 box thừa khi dừng
Chấm điểm khi tập tham chiếu không còn trong repo
Script chấm báo không tìm thấy thư mục gold_train_labels, và file JSON chính thức vẫn giữ kết quả cũ 0.5567
Chép lại kết quả từ lần chấm cuối do chính script sinh ra (annotation_evaluation_now.json) sang annotation_evaluation.json
Đã xử lý; nhưng từ đó không chấm lại được nữa nên 0.7746 là kết quả cuối
Đẩy bài lên GitHub
git push bị từ chối: Invalid username or token / credential manager bị huỷ
Thử lại qua credential manager
Chưa xử lý xong: nhãn rework và báo cáo chưa đẩy lên remote tại thời điểm viết báo cáo

7. Tự đánh giá theo rubric
   Đã có kết quả và kiểm được (đối chiếu RUBRIC.md):
   50 file nhãn đúng schema car/truck/bus/van, checker cấu trúc đạt 50/50, có self-QC bằng ảnh chồng lớp.
   Có mini guideline với định nghĩa lớp, quy tắc box và 3 ca mơ hồ kèm lý do.
   Có JSON chấm nhãn và bảng trước/sau rework với số liệu thật.
   Colab đã chạy hết notebook; có metrics, best checkpoint path, cả hai folder assets/eval_predictions/ và assets/yolo26_runs/.
   Đã trả lời 4 câu hỏi cuối notebook, mỗi câu có số liệu và ảnh minh họa.
   Chưa đạt hoặc chưa kiểm được — ghi rõ, không nhận là đã xong:
   Ngưỡng gold QC chưa đạt: F1 0.7746 < 0.80 (mean IoU 0.8358 đã đạt).
   Phiên Colab chạy trên CPU, không dùng GPU T4; thời gian train 0.570 giờ và kết quả không phải benchmark.
   Chưa đẩy được lên GitHub (lỗi xác thực), nên repo remote hiện vẫn là bản nhãn trước rework.
   Báo cáo mục 1 còn thiếu mã lớp và link Colab.
8. Công cụ AI và tài liệu tham khảo
   Công cụ AI đã dùng: trợ lý AI (Doubao) trong vai trò chẩn đoán và dựng công cụ, cụ thể:
   Sinh script chấm và so sánh nhãn với tập tham chiếu (chạy tools/evaluate_annotations.py, lưu kết quả từng lần).
   Dựng ảnh chồng lớp 4 màu cho 50 frame, cắt ảnh phóng to cho từng ca lỗi class (kèm tên file và số dòng), và bảng xếp hạng frame theo số box thừa — đây là công cụ để tôi tự quyết định sửa gì.
   Soạn bản nháp báo cáo này; toàn bộ số liệu trong báo cáo lấy từ các file do chính script trong repo sinh ra (outputs/annotation_evaluation.json, model_eval_metrics.json, log Colab).
   Phần tôi tự làm và tự kiểm: toàn bộ việc vẽ box và quyết định class trong CVAT, quyết định xoá/giữ từng box sau khi xem ảnh chồng lớp, cấu hình project/task và export trong CVAT, chạy notebook trên Colab, và đối chiếu lại từng con số trong bảng với file JSON tương ứng.
   Nguồn ngoài repo: chỉ tài liệu chính thức của Ultralytics (hướng dẫn model.train, model.val) để hiểu ý nghĩa mAP50 / mAP50–95. Không dùng nhãn tham chiếu của người khác làm nhãn bài mình, không đính kèm gold_train_labels, không ghi token hay mật khẩu trong repo.
