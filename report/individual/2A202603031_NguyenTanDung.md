# Báo cáo Day 13 — Nguyễn Tấn Dũng, 2A202603031

## 1. Kết luận trước

Task cá nhân: `D13-2A202603031`. Môi trường Windows, WSL Ubuntu và Python venv; phiên bản CVAT, trình duyệt thực tế và MSSV bạn ngồi cạnh chưa xác nhận.

| Frame | s01f22 | s01f49 | s02f16 | s02f41 | s03f10 | s03f40 | s04f09 | s04f33 | s05f20 | s05f42 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Số cuboid | 9 | 15 | 11 | 10 | 16 | 5 | 17 | 5 | 5 | 6 |

Bảng A có 99 cuboid (78 Car, 11 Cyclist, 6 Pedestrian, 3 Truck, 1 Bus); phiếu tự kiểm cờ 9/99 (9,09%). Ba frame ít nhất là s03f40, s04f33 và s05f20, mỗi frame 5 hộp; chưa xác định nguyên nhân cụ thể. Phần nhận dạng xe rõ có cơ sở tốt hơn phần đáy hộp và vật xa. Có 5 hộp vượt 50 m; nếu loại chúng thì còn 94 hộp, frame đầu còn 7, chưa đạt yêu cầu 8 hộp trong phạm vi.

Thao tác CVAT: dùng Perspective để xoay, dời và phóng cảnh; Top để nhìn từ trên. Chọn Shape, nháy đúp đặt hộp; chỉnh tâm, dài, rộng và yaw ở Top, chỉnh đáy và cao ở Side/Front. Bật Cuboid orientation, đối chiếu hướng đầu vật với camera và chạy `preview`.

## 2. Đọc frame (Bước 2)

Ảnh: `t1-s01f22-3d.png`, `t1-s01f22-top.png`, `t1-s01f49-3d.png`, `t1-s01f49-top.png`. Tên frame gán theo thứ tự ảnh gửi; cần kiểm tra lại trên CVAT. Hệ VF: trước xe là −Y, +X sang trái, +Y ra sau; ảnh Top chưa có dấu trục nên chưa xác định được cạnh ảnh tương ứng.

| Frame | Hướng trước, +X, +Y | Mặt đường | Vật xa nhất đã gán | Loại vật | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f22 | −Y; +X trái; +Y sau | Khoảng −1,7 m theo mốc guideline; chưa đo độ dốc | 59,90 m (ngoài phạm vi) | Car, Bus | Car phía sau; Bus bên trái | Điểm thưa ngoài vùng đường, chưa rõ loại |
| s01f49 | −Y; +X trái; +Y sau | Khoảng −1,7 m theo mốc guideline; chưa đo độ dốc | 34,37 m | Car, Cyclist, Pedestrian | Car phía sau; người và xe hai bánh bên phải | Các cụm nhỏ rải rác, chưa rõ loại |

Camera hỗ trợ nhận biết màu sắc, kiểu phương tiện và hướng đầu xe. LiDAR cho phép đo khoảng cách theo mét, xác định vị trí/chiều cao trong không gian và quan sát sự phân bố điểm khi bị che. Chưa có ảnh camera của chính hai frame đầu để xác nhận từng nhận xét màu sắc và hướng đầu xe.

Tám frame lướt (thông tin từ bảng A và preview đã có):

| Frame | Hướng trước | Quan sát |
|---|---|---|
| s02f16 | −Y; BEV hướng lên | Vật xa, hai cảnh báo đáy |
| s02f41 | −Y; BEV hướng lên | Xe xa, 1 cảnh báo điểm ngoài mép |
| s03f10 | −Y; BEV hướng lên | Dãy xe che nhau, 4 cảnh báo đáy |
| s03f40 | −Y; cạnh ảnh chưa xác nhận | 5 hộp Car/Cyclist |
| s04f09 | −Y; cạnh ảnh chưa xác nhận | 17 hộp, có vật vượt 50 m |
| s04f33 | −Y; cạnh ảnh chưa xác nhận | 5 hộp Car/Cyclist |
| s05f20 | −Y; cạnh ảnh chưa xác nhận | 5 hộp Car |
| s05f42 | −Y; cạnh ảnh chưa xác nhận | 6 hộp Car |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Ảnh cận cảnh chưa hiện tên frame/ID; bảng dưới chọn các mẫu theo bảng A, chưa xác nhận chúng là chính các hộp trong ảnh.

| Vật | Frame, idx | Class | Yaw | Đáy | Kiểm Top/Side/Front + camera | Kích thước l × w × h (m) |
|---|---|---|---|---|---|---|
| Xe con gần | s01f22 #1 | Car | 268,4°, gần 270° | −1,734 m | Chưa xác nhận đủ góc | 4,524 × 2,002 × 1,510 |
| Xe buýt | s01f22 #2 | Bus | 270,4° | −1,658 m | Chưa xác nhận đủ góc | 12,660 × 2,759 × 3,444; dài/rộng cần đối chiếu |
| Người đi bộ | s01f49 #14 | Pedestrian | 277,9°; chưa xác nhận hướng mặt | −1,103 m; cần kiểm địa hình | Chưa xác nhận đủ góc | 0,435 × 0,622 × 1,457 |

Vòng hiệu chuẩn, quyết định chung của lớp và danh sách vật bỏ qua chưa có thông tin xác nhận; không ghi kết quả giả định.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Các ảnh do học viên cung cấp từ CVAT; mã frame của ảnh cận cảnh chưa xác nhận. Nguyên nhân dưới đây là giả thuyết.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân giả thuyết | Quan sát | Hệ quả khi gán |
|---|---|---|---|---|
| Bóng che | `t4-1-bong-che.png`, CVAT; ứng viên s03f10 #8 | Xe gần chắn tia tới xe sau | Dãy hộp nối tiếp nhau, một số hộp nét đứt | Bật occluded nếu xác nhận bị che, bao phần suy luận được |
| Thưa ở xa | `t4-2-thua-o-xa.png`, CVAT; ứng viên s02f16 #11 (44,03 m, 16 điểm) | Mật độ tia giảm theo khoảng cách | Các hộp xa ít điểm hơn; ảnh chưa xác nhận khoảng cách | Chỉ vẽ khi đủ cơ sở; nếu không ghi LOW_VISIBILITY |
| Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png`, CVAT; chưa rõ frame | Bề mặt phản xạ yếu hoặc góc tới bất lợi | Hộp người có ít điểm, chưa xác nhận khoảng cách hay cường độ phản xạ | Đối chiếu camera, không khẳng định nguyên nhân từ ảnh |
| Điểm lạ | Chưa có `t4-4-diem-la.png` | Nhiễu hoặc phản xạ bất thường | Chưa có minh chứng đủ rõ | Không gán điểm rời rạc thành vật khi thiếu căn cứ |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu lấy từ bảng A và phiếu tự kiểm của MSSV 2A202603031. Khoảng cách 3D tính từ tọa độ tâm; mốc mặt đường −1,7 m chỉ là tham chiếu.

| Cuboid | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw (°) | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| s01f22 #1 Car | −0,979 | 1,510 | −1,734 | 3,25 | 3,40 | 268,4 | Sát mốc đường |
| s01f22 #2 Bus | 0,064 | 3,444 | −1,658 | 6,90 | 6,90 | 270,4 | Sát mốc đường |
| s01f22 #8 Car | −1,443 | 1,510 | −2,198 | 54,42 | 54,44 | 268,4 | Thấp hơn mốc đường, ngoài phạm vi |

Gần nhất: s01f22 #1, dist_m 3,25 m, r 3,40 m. Xa nhất: s02f41 #10, dist_m 71,72 m, r 71,73 m (ngoài phạm vi). Xe con #1 dài 4,524 m, rộng 2,002 m, cao 1,510 m; chiều dài nằm trong khoảng 3,8–5,3 m. Người đi bộ s01f49 #14 cao 1,457 m, hơi thấp so với khoảng tham chiếu người lớn 1,5–1,95 m; chưa rõ do vóc dáng hay thiếu điểm. Xe s01f22 #3 ở trước có yaw 270,8°; xe s02f16 #8 có yaw 90,9°, gần hướng ngược chiều theo quy ước.

| Cuboid | dist_m (m) | points_in |
|---|---:|---:|
| s01f22 #1 Car | 3,25 | 3224 |
| s01f22 #3 Car | 15,06 | 1046 |
| s01f22 #5 Car | 20,30 | 338 |
| s01f22 #7 Car | 37,53 | 73 |

Khoảng cách tăng khoảng 11,55 lần thì số điểm còn khoảng 2,26% (73/3224). Xu hướng giảm phù hợp đường cong bài giảng (1968, 718, 214, 72 điểm ở bốn khoảng tương ứng); số thực tế khác do kích thước, che khuất và bề mặt vật.

Tự kiểm: 3 cảnh báo điểm ngoài mép (s01f49 #14, #15; s02f41 #3) và 6 cảnh báo đáy (s02f16 #10, #11; s03f10 #11, #12, #13, #16). Hai người s01f49 có tâm ngang cách nhau khoảng 0,47 m, có thể điểm ngoài hộp thuộc người bên cạnh. Cảnh báo đáy cần đối chiếu mặt đất cục bộ; chưa xác nhận đã sửa hộp nào, hiện giữ nguyên kết quả CSV. Hai ứng viên ưu tiên rà soát là s03f10 #11 (0,81 m) và #12 (0,58 m). Chưa có bằng chứng đã hoàn thành yêu cầu sửa hai hộp và đổi kiểm với học viên khác.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D dùng mét để mô hình học vị trí và kích thước thật của vật trong hệ tọa độ LiDAR, không phụ thuộc tỷ lệ pixel. Vật thưa điểm cần tiêu chí riêng để tránh học từ hộp suy đoán thiếu cơ sở.

- Công cụ AI: ChatGPT.
- Câu hỏi: cách thao tác CVAT, tìm cuboid cảnh báo, chọn minh chứng và tổng hợp CSV.
- Cách sử dụng: hỗ trợ đọc số liệu, lập báo cáo và đổi tên ảnh; không dùng AI tạo annotation hoặc thay ảnh chụp thật.

Điều còn chưa chắc: hướng và đáy của vật xa, nguyên nhân ít điểm, mã frame/idx của ảnh cận cảnh và minh chứng điểm lạ. Cần xác nhận các thông tin này trước khi coi bài nộp là hoàn chỉnh.
