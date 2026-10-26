# Session06 - Bài 1 và 2: Bãi đỗ xe và API danh sách xe

Java 21, Spring Boot, JPA và PostgreSQL.

## Chạy

1. Mở project bằng IntelliJ, chọn JDK 21.
2. Chạy `BaitapSession06Application`.
3. Trong pgAdmin, mở database `parking_management_session06` → Schemas → public → Tables → Refresh.

Database và kết nối đã được chuẩn bị trên máy hiện tại. Mật khẩu nằm ngoài project tại `../.local-config/baitapSession06.properties`.

Máy khác cần tạo database bằng `CREATE DATABASE parking_management_session06;`, rồi đặt biến môi trường `DB_USERNAME`, `DB_PASSWORD` cho tài khoản PostgreSQL của máy đó. Có thể dùng `DB_URL` để đổi URL kết nối. Hibernate tự tạo bảng khi khởi động nhờ `ddl-auto=update`.

## Mapping

- `Zone` → `zones`: id, name, capacity, occupied_spots.
- `Vehicle` → `vehicles`: id, license_plate, color, type (CAR/BIKE).
- `ParkingTicket` → `parking_tickets`: id, check_in_time, check_out_time, vehicle_id, zone_id.
- Zone và Vehicle có `@OneToMany(mappedBy = ...)` trỏ về ParkingTicket.
- ParkingTicket dùng hai `@ManyToOne`, khóa ngoại vehicle_id và zone_id.
- check_out_time cho phép NULL khi xe chưa ra bãi.

## API tạo xe

POST `http://localhost:8080/api/v1/vehicles`, chọn Body → raw → JSON trong Postman:

```json
{
  "licensePlate": "18A-12345",
  "color": "blue",
  "vehicleType": "CAR"
}
```

`vehicleType` nhận CAR hoặc BIKE. Thành công trả HTTP 201 và `ApiResponse<VehicleResponse>`; không trả trực tiếp entity.

## API danh sách xe

```text
GET http://localhost:8080/api/v1/vehicles?page=0&size=5&sortBy=licensePlate&direction=ASC&keyword=18
```

- page mặc định 0; số âm tự về 0. size mặc định 10, phải lớn hơn 0.
- Chỉ sắp xếp khi có cả sortBy và direction. Nếu thiếu một trong hai, dùng Sort.unsorted().
- sortBy hỗ trợ id, licensePlate, color, vehicleType; direction nhận ASC/DESC.
- keyword tìm gần đúng theo biển số, không phân biệt hoa/thường. Bỏ keyword hoặc để rỗng sẽ lấy toàn bộ xe.
- Kết quả được bọc trong ApiResponse → PageResponse, gồm items, page, size, totalItems, totalPages, isLast.
- Mỗi item chỉ có id, licensePlate, color, vehicleType; totalItems dùng Long.

Repository dùng JPQL `SELECT new ...VehicleResponse(...)` kết hợp điều kiện `:keyword IS NULL OR ...` và Pageable. CAST tham số keyword sang string giúp PostgreSQL xử lý đúng trường hợp keyword=null.

Build/cache được tạo ngoài project khi kiểm tra để giữ số file nộp thấp. Khởi động lại ứng dụng để thử Postman; dữ liệu demo kiểm tra API được giữ trong database.
