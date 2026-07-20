# Hướng dẫn: Lưu lead dự phòng, không sợ mất khi Make trục trặc

Mục tiêu: mỗi khách đăng ký trên `secondbrain.lethanhson.net` được ghi vào **2 nơi độc lập cùng lúc**, để Make có sập thì lead vẫn còn nguyên ở nơi kia.

```
                 ┌─────────────► Đường 1: Make  ──► (ghi Sheet chính, báo Telegram, gửi mail... tùy anh)
   Khách bấm ────┤
   "Đăng ký"     └─────────────► Đường 2: Apps Script ──► Google Sheet DỰ PHÒNG (ghi thẳng, KHÔNG qua Make)
```

- Đường 1 (Make): giữ nguyên như đang chạy. Đây là chỗ để **thêm bao nhiêu nơi cũng được** bằng kéo thả trong Make.
- Đường 2 (Apps Script): **mới thêm**. Đây là két sắt dự phòng, tối giản, chỉ ghi lead thô.

Điểm mấu chốt: Sheet dự phòng phải do **web ghi thẳng**, không đi qua Make. Sheet mà Make đang ghi hiện nay sẽ trống trơn nếu Make chết, nên nó không tính là dự phòng.

---

## Phần A - Anh tự làm: dựng Sheet dự phòng + Apps Script (10-15 phút, làm 1 lần)

### Bước 1. Tạo Google Sheet mới
1. Vào https://sheets.new (tạo bảng tính trắng).
2. Đặt tên, ví dụ: **Lead Second Brain - Du phong**.
3. Ở hàng đầu tiên, gõ tiêu đề cho 7 cột (mỗi ô một cột):

   `Thoi diem ghi` · `Ho ten` · `SDT` · `Email` · `Lop` · `Nguon` · `Gio khach dang ky`

### Bước 2. Mở trình soạn mã
1. Trên thanh menu của Sheet, bấm **Tiện ích mở rộng** (Extensions) → **Apps Script**.
2. Một tab mới mở ra, có sẵn một khối `function myFunction() {}`. **Xóa sạch** khối đó.
3. **Dán nguyên đoạn mã dưới đây** vào:

```javascript
function doPost(e) {
  try {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName('Leads') || ss.getSheets()[0];

    var data = {};
    if (e && e.parameter && e.parameter.value) {
      data = JSON.parse(e.parameter.value);        // web gui dang value=<JSON>
    } else if (e && e.postData && e.postData.contents) {
      try { data = JSON.parse(e.postData.contents); } catch (x) {}  // phong khi gui JSON thang
    }

    sheet.appendRow([
      new Date(),
      data.hoten || '',
      data.sdt || '',
      data.email || '',
      data.lop || '',
      data.nguon || '',
      data.thoi_gian || ''
    ]);

    return ContentService.createTextOutput('OK');
  } catch (err) {
    return ContentService.createTextOutput('ERR: ' + err);
  }
}

function doGet(e) {
  return ContentService.createTextOutput('Backup lead endpoint dang chay. Dung POST de ghi.');
}
```

4. Bấm biểu tượng **đĩa mềm (Lưu)** ở thanh trên.

### Bước 3. Đăng thành đường link nhận dữ liệu (Deploy)
1. Góc trên bên phải, bấm **Triển khai** (Deploy) → **Bản triển khai mới** (New deployment).
2. Bấm biểu tượng **bánh răng** cạnh chữ "Chọn loại" → chọn **Ứng dụng web** (Web app).
3. Điền:
   - **Thực thi với tư cách** (Execute as): **Tôi** (Me / chính anh).
   - **Ai có quyền truy cập** (Who has access): **Bất kỳ ai** (Anyone).  ← QUAN TRỌNG, chọn sai thì web ghi không vào.
4. Bấm **Triển khai** (Deploy).

### Bước 4. Vượt bảng cảnh báo cấp quyền (chỗ hay làm người mới hoảng)
1. Google hỏi quyền → bấm **Cho phép quyền truy cập** (Authorize access), chọn tài khoản Google của anh.
2. Hiện bảng **"Google chưa xác minh ứng dụng này"** (Google hasn't verified this app). Đây là app của **chính anh** nên an toàn.
   - Bấm dòng chữ nhỏ **Nâng cao** (Advanced) ở góc dưới bên trái.
   - Bấm **Đi tới [tên dự án] (không an toàn)** (Go to ... unsafe).
   - Bấm **Cho phép** (Allow).
3. Xong, Google hiện một đường link dạng:
   `https://script.google.com/macros/s/AKfy....../exec`
   **Copy đường link này** (bấm Sao chép). Đây là thứ em cần để ráp vào web.

### Bước 5. Kiểm tra nhanh đường link sống chưa
- Dán đường link `.../exec` vào một tab trình duyệt rồi Enter.
- Nếu hiện dòng chữ `Backup lead endpoint dang chay...` là **đường link đã sống**. 
- Đưa đường link đó cho em (dán vào khung chat).

---

## Phần B - Em làm: ráp vào web

Khi có đường link của anh, em sẽ:
1. Sửa `index.html`: thêm một lệnh gửi lead song song tới đường link đó, **chạy nền y hệt cách đang gửi Make** - bắn rồi quên, không làm chậm trang, không cản khách, lỗi ghi Sheet cũng không ảnh hưởng gì tới việc khách qua trang thanh toán.
2. Test tại máy: bắn 1 lead thử, xác nhận nó nhảy vào Sheet dự phòng và luồng cũ vẫn nguyên.
3. **Xin anh duyệt rồi mới** đưa lên GitHub cho web live cập nhật. Live xong bắn 1 lead thật, xác nhận vào đủ 2 nơi.

---

## Phần C - Muốn thêm nơi thứ 3 về sau

**Cách dễ nhất (khuyên dùng): nhét vào sau Make.**
Mở kịch bản Make đang chạy, kéo thêm một khối (module) mới nối vào cuối: ghi thêm một Sheet, gửi tin Telegram báo anh mỗi lead, đẩy vào CRM... Kéo thả là xong, **không cần đụng code web**. Nhược: nơi này sẽ ngừng nhận nếu Make sập.

**Cách để nơi thứ 3 cũng độc lập (chỉ làm khi thật cần):**
Báo em, em sẽ cho web bắn thêm một đường thẳng thứ ba (ví dụ tới một Apps Script/Sheet khác). Đổi lại anh phải trông thêm một chỗ. Với nhu cầu hiện tại, 2 đường độc lập là đủ.

---

## Phần D - Các lỗi hay gặp và cách xử

| Hiện tượng | Nguyên nhân thường gặp | Cách xử |
|---|---|---|
| Web chạy nhưng Sheet dự phòng không có dòng nào | Lúc đăng chọn nhầm quyền, không phải "Bất kỳ ai" | Deploy lại, chọn đúng "Bất kỳ ai" |
| Sửa code Apps Script mà không thấy tác dụng | Quên đăng bản mới sau khi sửa | Triển khai → Quản lý bản triển khai → bút chì Sửa → Phiên bản: Bản mới → Triển khai (đường link giữ nguyên) |
| Đường link `.../exec` mở ra báo lỗi quyền | Chưa cấp quyền xong ở Bước 4 | Làm lại Bước 4, nhớ bấm Nâng cao → Đi tới (không an toàn) → Cho phép |
| Lo lead có vào đều không | Đường dự phòng "bắn rồi quên", web không báo | Thỉnh thoảng liếc Sheet; hoặc sau này thêm cảnh báo (chưa cần vội) |

> Lưu ý: mỗi lần tạo "Bản triển khai mới" sẽ ra một đường link khác. Sau này muốn sửa code thì dùng **Quản lý bản triển khai → Sửa** để **giữ nguyên đường link cũ**, khỏi phải sửa lại web.
