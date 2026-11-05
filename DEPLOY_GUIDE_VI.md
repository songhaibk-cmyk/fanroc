# Hướng dẫn triển khai ứng dụng chấm điểm FAnRoC 2026

## 1. Bản hiện tại làm được gì

- Nhập đội thủ công hoặc tải danh sách CSV; xuất danh sách đội CSV.
- Chọn số trận mục tiêu/đội, tạo lịch ngẫu nhiên cân bằng gần nhất và xem trước số trận từng đội.
- Chấm bóng, cây, cờ, trạng thái Safe Zone, penalty/thẻ; tính điểm trận chung và điểm từng đội.
- Theo dõi bảng xếp hạng, chọn trận để trình chiếu, lưu nháp trên trình duyệt.
- Tải/khôi phục bản sao lưu JSON; xuất lịch và bảng xếp hạng.
- Có giao diện responsive và PWA cơ bản.

## 2. Giới hạn quan trọng của bản đầu

Bản này chưa kết nối cơ sở dữ liệu dùng chung. Dữ liệu nằm trong trình duyệt của thiết bị đang sử dụng, nên máy tính bảng trọng tài và máy tính bàn chưa tự đồng bộ; link khán giả trên một thiết bị khác chưa nhận điểm mới; sao lưu hiện là thao tác tải JSON thủ công. Hãy sử dụng bản này để thử nghiệm giao diện và quy trình, đồng thời thường xuyên tải backup. Chưa nên dùng làm nguồn điểm duy nhất cho giải chính thức.

## 3. Đưa bản web lên mạng để thử

1. Tạo hoặc chọn một kho Git riêng.
2. Tải toàn bộ thư mục `fanroc-app` lên kho Git; giữ nguyên `dist/index.html`, `dist/app.js`, `dist/styles.css`, `dist/manifest.webmanifest`, `dist/sw.js` và `.openai/hosting.json`.
3. Có thể dùng GitHub Pages, Netlify, Vercel hoặc Cloudflare Pages để host static site. Chọn thư mục xuất bản là `dist`; không cần lệnh build.
4. Mở URL trên máy tính, máy tính bảng và điện thoại. Thêm ứng dụng ra màn hình chính nếu muốn dùng như PWA.
5. Vào Cài đặt & sao lưu, tạo một bản JSON thử, rồi khôi phục trên trình duyệt khác để kiểm tra quy trình dự phòng (lưu ý: dữ liệu sẽ thay thế dữ liệu hiện có trên thiết bị đó).

## 4. Quy trình dùng thử

1. Vào Danh sách đội → nhập thủ công hoặc chọn CSV với các cột `teamId,name,school`.
2. Vào Cài đặt & sao lưu → chọn số trận mục tiêu mỗi đội và tên vòng.
3. Chọn Tạo lịch ngẫu nhiên → rà soát số trận phân bổ → xác nhận tạo.
4. Mở một trận → nhập số bóng, cây theo từng núi/rãnh; vào tab Cờ & núi, Safe Zone, Lỗi & thẻ để hoàn thiện kết quả.
5. Gửi chấm điểm → xác nhận → công bố. Bảng xếp hạng tính từ trận đã công bố/khóa.
6. Sau mỗi vòng, tải bản sao lưu JSON và lưu thêm một bản tại máy tính của Ban Tổ chức hoặc ổ lưu trữ riêng.

## 5. Để triển khai bản thi đấu nhiều thiết bị

Cần bổ sung backend chia sẻ dữ liệu, ví dụ Firebase:

1. Tạo Firebase project và bật Firestore Database, Firebase Authentication và Hosting.
2. Thêm người dùng cho Ban Tổ chức/trọng tài. Không cho phép khách ghi dữ liệu; trang khán giả chỉ đọc dữ liệu đã công bố.
3. Chuyển thao tác đọc/ghi đội, trận, điểm, lỗi và lịch sử từ `localStorage` sang Firestore. Giữ local draft khi offline; khi online, gửi event có `matchId`, `userId`, `updatedAt`, `revision` để tránh ghi đè.
4. Chuyển logic tính điểm dùng chung sang module kiểm thử và xác thực lại ở backend trước khi công bố.
5. Tạo Cloud Function hoặc tác vụ server định kỳ để xuất snapshot JSON/CSV của các collection lên Cloud Storage. Bật versioning/retention; tạo bản sao thủ công trước/sau mỗi vòng.
6. Tạo trang restore chỉ cho quản trị viên; trước khi khôi phục, tạo snapshot hiện tại, xem trước thời điểm/phạm vi backup và yêu cầu xác nhận.
7. Kiểm tra quyền bảo mật Firestore: chỉ tài khoản trọng tài được ghi nháp; trưởng trọng tài được xác nhận; quản trị viên được sửa/khóa; khán giả chỉ đọc bản công bố. Không đưa service account private key hoặc secret backend vào frontend.
8. Kiểm thử 2 thiết bị nhập cùng lúc, mất mạng rồi kết nối lại, backup/restore và kiểm tra bảng điểm công khai trước giải.
9. Deploy Hosting bằng Firebase CLI (`firebase deploy`) sau khi build/copy thư mục static. Lưu cấu hình project trong tài liệu nội bộ, không ghi secret vào Git.

## 6. Các bước backup khẩn cấp hiện có

- Trên thiết bị giữ dữ liệu mới nhất, mở Cài đặt & sao lưu → Tạo bản sao lưu JSON.
- Lưu tệp ở ít nhất hai nơi tách biệt (máy Ban Tổ chức và ổ/đám mây có quyền truy cập giới hạn).
- Khi cần phục hồi, mở app trên thiết bị đích → Khôi phục → chọn JSON. Tệp được nhập sẽ thay thế dữ liệu hiện có trên trình duyệt đó.
- Nếu máy chủ/ngắt mạng xảy ra sau khi có backend: giữ thiết bị không xóa dữ liệu local, ghi lại trạng thái trận bằng phiếu giấy dự phòng, xuất JSON khi có thể, rồi đồng bộ có đối chiếu khi hệ thống trở lại.

## 7. Điểm luật cần BTC xác nhận

Luật đính kèm nêu ví dụ 2 với tổng 299 điểm, nhưng các hạng mục trong ví dụ cộng ra 269 điểm. Ứng dụng tính từ các mục điểm, không hard-code tổng mẫu 299. Hãy rà soát bản luật trước khi dùng kết quả kiểm tra làm chuẩn.
