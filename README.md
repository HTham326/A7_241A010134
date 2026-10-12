# 📱 LAB A7 – XIN QUYỀN TRUY CẬP TÀI NGUYÊN (RUNTIME PERMISSION)

> Bài thực hành môn **Lập trình trên các thiết bị di động (INT4211)** – xây dựng ứng dụng Android bằng Java và XML, thực hành quản lý quyền CAMERA và POST_NOTIFICATIONS.

## 📌 1. Thông tin sinh viên và dự án

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Nguyễn Thị Hồng Thắm |
| Mã số sinh viên | 241A010134 |
| Lớp | CNTT – LTDD |
| Trường | Đại học Văn Hiến |
| Môn học | Lập trình trên các thiết bị di động (INT4211) |
| Bài thực hành | Lab A7 – Runtime Permission |
| Tên ứng dụng | A7_241A010134 |
| Package | `vn.edu.vhu.ltdd.a7permission` |
| Ngôn ngữ | Java, XML |
| Bài nâng cao | NC1 – Xin hai quyền cùng lúc; NC3 – Nhấn thông báo mở lại ứng dụng |

## 🎯 2. Giới thiệu và mục tiêu

Ứng dụng minh họa cách xin và kiểm tra quyền truy cập tài nguyên trong Android. Người dùng có thể cấp hoặc từ chối quyền CAMERA để chụp ảnh xem trước, cấp quyền gửi thông báo trên Android 13 trở lên và thay đổi quyền trực tiếp trong phần Cài đặt ứng dụng.

Mục tiêu: phân biệt quyền thường và quyền nguy hiểm; khai báo quyền trong Manifest; xử lý các tình huống cấp quyền, từ chối và không còn được hỏi lại; sử dụng Activity Result API; tạo NotificationChannel; đồng bộ trạng thái quyền khi quay lại ứng dụng.

## 🛠️ 3. Công nghệ và môi trường

| Thành phần | Công nghệ / phiên bản |
|---|---|
| IDE | Android Studio |
| Nền tảng | Android SDK |
| Ngôn ngữ | Java 11, XML |
| Giao diện | Android Views, LinearLayout |
| Công cụ build | Gradle Kotlin DSL; Android Gradle Plugin 9.4.1 |
| Compile SDK / Target SDK | API 37 / API 37 |
| Minimum SDK | API 24 (Android 7.0) |
| AndroidX | AppCompat, Activity Result API, Core |
| Material Components | 1.14.0 |
| Quản lý mã nguồn | Git, GitHub |

## ✨ 4. Các chức năng chính

### 4.1. Xin quyền CAMERA

- Kiểm tra quyền trước khi mở camera bằng `ContextCompat.checkSelfPermission()`.
- Nếu đã cấp quyền, mở camera ngay.
- Nếu từng từ chối và còn có thể hỏi lại, hiển thị `AlertDialog` giải thích lý do cần quyền.
- Nếu hệ thống không tiếp tục hiển thị hộp thoại cấp quyền, hướng dẫn người dùng mở trang Cài đặt ứng dụng.
- Dùng `ActivityResultContracts.TakePicturePreview()` để lấy ảnh `Bitmap` thu nhỏ và hiển thị trong `ImageView`.
- Hiển thị kích thước ảnh theo chiều rộng × chiều cao (pixel) trong `tvAnh`.

### 4.2. Gửi thông báo

- Trên Android 13 (API 33) trở lên, kiểm tra và xin `POST_NOTIFICATIONS` trước khi gửi.
- Trên phiên bản cũ hơn, không yêu cầu runtime permission này.
- Tạo `NotificationChannel` từ Android 8 (API 26) trở lên.
- Gửi thông báo bằng `NotificationCompat.Builder` và `NotificationManagerCompat`.
- Bắt `SecurityException` nếu không thể gửi thông báo do thay đổi quyền.

### 4.3. Theo dõi quyền và mở Cài đặt

- Hiển thị trạng thái CAMERA và THÔNG BÁO là **ĐÃ CẤP** hoặc **CHƯA CẤP**.
- Gọi `capNhatTrangThai()` trong `onResume()` để kiểm tra lại sau khi người dùng quay về từ Cài đặt.
- Mở đúng trang thông tin ứng dụng bằng `Settings.ACTION_APPLICATION_DETAILS_SETTINGS`.

## 🚀 5. Hai bài nâng cao đã thực hiện

### NC1 – Xin hai quyền cùng lúc

Bổ sung nút **Xin hai quyền cùng lúc** và sử dụng `ActivityResultContracts.RequestMultiplePermissions()` để gửi một yêu cầu gồm CAMERA và POST_NOTIFICATIONS (trên Android 13+). Callback nhận kết quả cho từng quyền độc lập, ghi Logcat và cập nhật trạng thái trên màn hình. Trên Android cũ hơn, chỉ đưa CAMERA vào danh sách xin quyền.

```java
private final ActivityResultLauncher<String[]> xinNhieuQuyen =
        registerForActivityResult(
                new ActivityResultContracts.RequestMultiplePermissions(),
                ketQua -> {
                    Boolean camera = ketQua.get(Manifest.permission.CAMERA);
                    Boolean thongBao = ketQua.get(Manifest.permission.POST_NOTIFICATIONS);
                    Log.d(TAG, "NC1 - CAMERA" + camera);
                    Log.d(TAG, "NC1 - POST_NOTIFICATIONS" + thongBao);
                    capNhatTrangThai();
                });
```

Báo cáo có minh chứng cho bốn tổ hợp trạng thái: chưa cấp quyền nào, chỉ cấp CAMERA, chỉ cấp thông báo và đã cấp cả hai quyền.

### NC3 – Nhấn thông báo để mở lại ứng dụng

Trong `guiThongBao()`, tạo `Intent` hướng đến `MainActivity` và `PendingIntent.getActivity()`; gắn vào thông báo bằng `setContentIntent()`. Sử dụng `FLAG_UPDATE_CURRENT | FLAG_IMMUTABLE` và `setAutoCancel(true)` để khi nhấn vào thông báo, ứng dụng được mở và thông báo tự đóng.

```java
Intent intent = new Intent(this, MainActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(
        this, 0, intent,
        PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);

// Gắn vào NotificationCompat.Builder:
// .setContentIntent(pendingIntent)
// .setAutoCancel(true)
```

## 📁 6. Cấu trúc dự án và vai trò các file

```text
A7/
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/vn/edu/vhu/ltdd/a7permission/
│       │   └── MainActivity.java
│       └── res/
│           ├── layout/activity_main.xml
│           ├── values/strings.xml
│           ├── values/colors.xml
│           └── values/themes.xml
├── gradle/libs.versions.toml
├── settings.gradle.kts
└── README.md
```

| File / thư mục | Vai trò |
|---|---|
| `MainActivity.java` | Xử lý quyền CAMERA, thông báo, chụp ảnh, cập nhật trạng thái, NC1 và NC3 |
| `AndroidManifest.xml` | Khai báo quyền CAMERA, POST_NOTIFICATIONS, camera tùy chọn và Activity khởi chạy |
| `activity_main.xml` | Thiết kế giao diện, vùng ảnh xem trước, dòng kích thước, trạng thái và các nút |
| `strings.xml` | Quản lý chuỗi giao diện, nội dung hộp thoại và thông báo |
| `colors.xml`, `themes.xml` | Khai báo màu và giao diện ứng dụng |
| `app/build.gradle.kts` | Cấu hình SDK, Java, dependencies và build ứng dụng |
| `gradle/libs.versions.toml` | Quản lý phiên bản thư viện và plugin |

## 🔐 7. Luồng xử lý quyền CAMERA

```text
Người dùng nhấn "Chụp ảnh"
          │
          ▼
Kiểm tra quyền CAMERA
          │
          ├── Đã cấp ────────────► Mở camera ─► Hiển thị ảnh và kích thước
          │
          ├── Đã từ chối, có thể hỏi lại
          │        └──► Giải thích ─► Xin quyền lần nữa
          │
          └── Lần đầu / không còn hộp thoại hệ thống
                   └──► Gửi yêu cầu quyền
                              ├── Được cấp ─► Mở camera
                              └── Bị từ chối ─► Giải thích hoặc hướng dẫn Cài đặt
```

> `shouldShowRequestPermissionRationale()` trả `false` không tự nó phân biệt được lần đầu và trường hợp hệ thống không hỏi lại. Ứng dụng dựa thêm vào kết quả callback và luồng xin quyền.

## 🧪 8. Kiểm thử chức năng

| STT | Kịch bản | Kết quả mong đợi |
|---|---|---|
| 1 | Cài mới, nhấn Chụp ảnh lần đầu | Xuất hiện hộp thoại xin quyền CAMERA |
| 2 | Chọn Cho phép | Mở camera và hiển thị ảnh xem trước |
| 3 | Từ chối, sau đó nhấn Chụp lại | Hiện hộp thoại giải thích của ứng dụng |
| 4 | Từ chối lần nữa và thử chụp | Hướng dẫn mở Cài đặt khi hệ thống không hỏi lại |
| 5 | Bật lại CAMERA trong Cài đặt rồi quay về | Trạng thái quyền cập nhật thành ĐÃ CẤP |
| 6 | Nhấn Gửi thông báo trên Android 13+ | Xin quyền thông báo; khi được cấp thì gửi thông báo |
| 7 | Xoay màn hình trong khi đang xin quyền | Ứng dụng tiếp nhận kết quả quyền sau khi tạo lại Activity |

**Kiểm thử nâng cao:** NC1 kiểm tra bốn tổ hợp cấp/từ chối của hai quyền; NC3 kiểm tra việc nhấn vào thông báo để quay về ứng dụng. Ảnh minh chứng chi tiết được trình bày trong báo cáo thực hành.

## ⚙️ 9. Hướng dẫn cài đặt và chạy

1. Clone repository hoặc tải mã nguồn về máy.
2. Mở thư mục project trong Android Studio.
3. Đợi Gradle Sync hoàn tất.
4. Chọn thiết bị Android có API 24 trở lên. Để kiểm thử POST_NOTIFICATIONS, nên dùng Android 13+.
5. Nếu dùng Emulator, vào **Device Manager → Edit → Camera** và chọn `Emulated` hoặc webcam thay vì `None`.
6. Nhấn **Run** để cài và khởi động ứng dụng.
7. Thử các nút Chụp ảnh, Gửi thông báo, Mở Cài đặt và Xin hai quyền cùng lúc.

Lệnh clone sau khi đã tạo repository:

```bash
git clone https://github.com/HTham326/A7_241A010134.git
```

## 🧩 10. Kiến thức rút ra

- Quyền nguy hiểm cần được khai báo trong Manifest và kiểm tra tại thời điểm sử dụng.
- `ActivityResultContracts.RequestPermission()` xử lý một quyền; `RequestMultiplePermissions()` xử lý nhiều quyền và trả về kết quả riêng biệt.
- `registerForActivityResult()` cần đăng ký đúng vòng đời Activity, không đăng ký bên trong sự kiện nhấn nút khi Activity đã RESUMED.
- `onResume()` phù hợp để đồng bộ lại trạng thái quyền sau khi quay về từ Cài đặt.
- `TakePicturePreview()` chỉ cung cấp ảnh xem trước thu nhỏ, không lưu ảnh gốc vào bộ nhớ.
- Thông báo cần tương thích với phiên bản Android và có thể sử dụng `PendingIntent` để mở lại Activity.

## 📚 11. Tài liệu tham khảo

- Tài liệu **Lab A7 – Xin quyền truy cập tài nguyên (Runtime Permission)**, học phần INT4211.
- [Android Developers – Request runtime permissions](https://developer.android.com/training/permissions/requesting)
- [Android Developers – Activity Result APIs](https://developer.android.com/training/basics/intents/result)
- [Android Developers – Notifications](https://developer.android.com/develop/ui/views/notifications)

---

**Thực hiện bởi:** Nguyễn Thị Hồng Thắm – **MSSV:** 241A010134  
**Học phần:** INT4211 – Lập trình trên các thiết bị di động
