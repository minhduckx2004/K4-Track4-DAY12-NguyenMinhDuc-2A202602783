# REPORT — Lab Day 12

| | |
|---|---|
| Họ và tên | Nguyen Minh Duc (đối chiếu lại cách viết theo giấy tờ cá nhân) |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602783` |
| GitHub | `overratedMD04` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | split seed `1562`; model seed gốc `9074`; các seed huấn luyện `9074, 9075, 9076` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `3` |

## Kết quả cuối

Số liệu lấy từ evidence của pipeline chạy với MSSV này, gồm cả bước `fix` với 3 seed và hai vòng DAgger.

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---:|---:|---|
| constant-velocity | 1.271 | 21.4 | BLOCK |
| IDM | 1.519 | 0.07 | baseline |
| EgoMLP | 0.771–0.779 | 14.1–22.3 | BLOCK |
| PercMLP | 0.625–0.633 | 0.7–1.8 | BLOCK |
| PercMLP+ (sau khi sửa) | 0.818 | 0.14–0.28 | RELEASE |

## Câu 1 · Dữ liệu

Audit cho thấy dữ liệu có 992.679 dòng và 9.993 quỹ đạo thô; loại 383 bản trùng tuyệt đối, còn 9.610 quỹ đạo. Cột Acc_LV có tương quan chỉ 0,045 với đạo hàm vận tốc, nên không phản ánh đáng tin gia tốc xe dẫn; pipeline dùng gia tốc suy ra từ Speed_LV đã làm mượt. Chia train/test theo quỹ đạo tránh để các cửa sổ gần nhau của cùng một chuyến xe xuất hiện ở cả hai phía. Nếu chia ngẫu nhiên theo cửa sổ, mô hình có thể thấy gần như cùng trạng thái trong train và test, làm điểm số lạc quan giả tạo. Khoảng thời lượng mỗi quỹ đạo là 70–109 bước ở 10 Hz.

## Câu 2 · Phát lại

Trong replay 600 đoạn, float32 khác bit ở 600/600 đoạn nhưng không đoạn nào vượt dung sai; sai khác tuyệt đối lớn nhất là 0,000122. Thêm AEB làm đổi 2 đoạn, số 247 và 346, là các đoạn có hành vi phanh bổ sung. Phát nhanh nhất báo 0 frame trễ và 0 frame rơi, nhưng phát thời gian thực có 582 frame trễ, 576 frame rơi và 376/600 đoạn đổi kết quả. Vì thế chạy nhanh che mất ràng buộc thời gian thực và lỗi do lịch phát; tính lặp lại chính xác trong chế độ nhanh không chứng minh bộ điều khiển kịp hạn ngoài đời. Hai thứ tự cộng số thực cũng không bằng nhau ở bit cuối.

## Câu 3 · Kịch bản

Lưới dùng tốc độ cruise 14 m/s, 60 mức khoảng cách và 45 mức tốc độ tiến gần, tổng danh nghĩa 2.700 ô. Bằng chứng simulator ghi 2.682 ô ở độ trễ 0 giây và 2.662 ô ở 0,6 giây sau khi lọc điều kiện biên. Phần bất khả thi tăng từ 3,63% lên 10,22%, tăng 6,59 điểm phần trăm. Có 618 sự kiện phanh được khai thác từ log; giới hạn phanh mạnh nhất của người lái ở phân vị 99,9 là 3,62 m/s², trong khi 12,67% lưới đòi hỏi vượt giới hạn đó. Kiểm tra full-brake khớp biên vật lý với 0 ô lệch ngoài biên ở cả hai độ trễ, nhưng log tự nhiên vẫn thiếu một phần tình huống khó của lưới tổng hợp.

## Câu 4 · Vòng hở và vòng kín

Theo L2 vòng hở, PercMLP đứng đầu (0,628 m), EgoMLP thứ hai (0,775 m), constant-velocity thứ ba (1,271 m), IDM cuối (1,519 m). Theo tỷ lệ va chạm log vòng kín trung bình, IDM tốt nhất (0,07%), PercMLP kế tiếp (1,16%), EgoMLP (19,53%), constant-velocity (21,37%). IDM có L2 kém vì luật bám xe không dự đoán chính xác diễn tiến tốc độ mà tài xế con người đã ghi, nhưng nó chủ động giữ khoảng cách và phanh theo gap nên an toàn hơn khi tự điều khiển. EgoMLP thiếu gap và tốc độ xe dẫn; sai số nhỏ trên log tăng dần khi hành động của mô hình làm trạng thái lệch khỏi dữ liệu huấn luyện. Đây là lý do xếp hạng vòng hở không đại diện cho an toàn vòng kín.

## Câu 5 · Một lần chạy chưa đủ

EgoMLP có tỷ lệ va chạm từ 14,12% đến 22,27% giữa ba seed, dao động 8,15 điểm phần trăm. Ở seed xấu nhất 9074, có 642 va chạm trên 2.883 quỹ đạo; khoảng Wilson 95% là 20,79%–23,82%. Một seed may mắn có thể báo 14,1% và che khuất seed xấu hơn gần 22,3%, nên kết luận an toàn sẽ phụ thuộc lựa chọn seed. Khoảng Wilson mô tả bất định lấy mẫu trong từng lần chạy, còn độ trải giữa seed phản ánh bất ổn do khởi tạo huấn luyện; cần xem cả hai. PercMLP cũng dao động 0,66%–1,77%, và cận trên Wilson seed xấu là 2,32%, vượt ngưỡng R2 1,07%.

## Câu 6 · Tác nhân phản ứng

Hiệu chỉnh IDM cho xe sau chọn T = 1,4 s, với khoảng cách Wasserstein-1 là 0,342 so với headway của người lái; tác nhân không phản ứng có khoảng cách 0,853. Ở độ trễ cắt đầu 0,6 s, IDM tạo va chạm phía sau giả trên 70,63% lưới khi xe sau chỉ replay log, nhưng tỷ lệ đó về 0% khi xe sau phản ứng; va chạm bất kỳ giảm từ 80,85% xuống 10,22%. Xe sau replay không biết xe thử nghiệm đã phanh nên tiếp tục tốc độ cũ và đâm vào đuôi. Với PercMLP, va chạm sau giảm từ 42,22% xuống 0%, dù va chạm tổng với xe phản ứng vẫn là 35,22%. Vì vậy mô hình hóa phản ứng của các xe khác có thể đảo mức va chạm tổng và phải là một phần của đánh giá.

## Câu 7 · Cổng phát hành và sửa

PercMLP bị BLOCK ở R2–R6: cận trên Wilson log 2,32% vượt baseline IDM 0,07% cộng margin 1 điểm (giới hạn 1,07%); lát phanh có cận trên 11,9% so với giới hạn 5%; vùng cắt đầu ngoài bất khả thi là 25% so với IDM 0%; độ trải seed là 0,7–1,8%; và reactive-world là 35,2% so với baseline 10,2%. Nguyên nhân là thiếu tình huống cắt đầu trong log và covariate shift khi mô hình tự lái. Bản sửa thêm 20.000 cut-in scenarios và chạy 2 vòng DAgger, huấn luyện chuyên gia IDM tại trạng thái mô hình tự thăm. PercMLP+ đạt L2 0,818 m; va chạm vòng kín là 0,14–0,28%, với cận trên seed xấu 0,55%, và vùng cắt đầu ngoài bất khả thi giảm còn 0,19%. Nó đạt cả R1–R7 và được RELEASE theo cổng này; kết quả tốt hơn IDM ở tỷ lệ va chạm log 0,14–0,28% so với 0,07% IDM? Trên mẫu này IDM có 0,07% trung bình, nên PercMLP+ không tốt hơn IDM theo trung bình, nhưng vẫn đạt ngưỡng non-inferiority. Vì vậy kết quả cổng là bằng chứng cho phép qua tiêu chí bài lab, không phải khẳng định an toàn triển khai; cần thêm kiểm thử ngoài phân phối, cảm biến và tương tác thực tế.

## Câu 8 · Bài lab này không chứng minh được gì

Mô phỏng chỉ xét chuyển động dọc một chiều, không có đánh lái, địa hình, thời tiết hay tương tác làn đường; kết luận không tự mở rộng sang lái xe thực. Xe dẫn trong log được phát lại và không phản ứng với xe thử nghiệm, còn mô hình xe sau phản ứng chỉ là IDM hiệu chỉnh với T=1,4 s, nên tương tác giao thông vẫn đơn giản hóa mạnh. Dữ liệu đến từ một tập OpenPAV/Argoverse 2 cụ thể; 9.610 quỹ đạo sạch không đại diện mọi khu vực, loại xe hay hành vi con người. Ba seed và khoảng Wilson lượng hóa một phần bất định nhưng không bao quát sai lệch phân phối hoặc lỗi cảm biến. Dù PercMLP+ chỉ có 0,14–0,28% va chạm log và vùng cut-in ngoài biên 0,19%, các con số này vẫn chỉ thuộc môi trường thử nghiệm đã mô hình hóa. Tiếp theo cần kiểm thử đa chiều trong simulator có tương tác, thêm dữ liệu đa vùng và điều kiện, kiểm thử phần cứng trong vòng lặp, rồi thử nghiệm có giám sát trước khi cân nhắc triển khai thật.
