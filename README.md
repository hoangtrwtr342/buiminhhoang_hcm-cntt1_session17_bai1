# # BÁO CÁO KỸ THUẬT: THIẾT KẾ KHÔNG GIAN ĐA PROFILE & KIẾN TRÚC BOOKMARK CHO TRẠM KHÁM BỆNH CAREPULSE

---

## PHẦN 1: BÁO CÁO PHÂN TÍCH KIẾN TRÚC ĐA PROFILE

### 1. Nguyên nhân sâu xa của hiện tượng xung đột phiên làm việc (Session Conflict)
* **Dùng chung SQLite Store (`User Data`):** Khi toàn bộ nhân sự dùng chung một Chrome Profile mặc định (`Default`), mọi tab và cửa sổ đều trỏ về và ghi/đọc chung trên các file cơ sở dữ liệu cục bộ: `Network/Cookies` và `Local Storage/leveldb`.
* **Tranh chấp phiên & Ghi đè Token xác thực:** 
  * Các ứng dụng y tế (CarePulse) và cổng thông tin định danh người dùng qua Session Cookie/OAuth Token. 
  * Khi thực tập sinh mở thêm tab đăng nhập cổng học tập cá nhân (PTIT, GitHub...) hoặc thao tác làm mới trạng thái xác thực, tiến trình Network Service sẽ cập nhật lại Cookie Store.
  * Việc này làm thay đổi hoặc vô hiệu hóa session token hiện tại của hệ thống CarePulse ở tab bên cạnh. Cơ chế bảo mật phiên của CarePulse phát hiện sai lệch trạng thái sẽ lập tức kích hoạt sự kiện tự động đăng xuất (**Logout**) giữa chừng.

---

### 2. Mô hình 3 Profile chuyên dụng chuẩn kỹ sư phần mềm

| Tiêu chí | Profile 1: Work (Khám chữa bệnh) | Profile 2: Personal (Học tập & Nghiên cứu) | Profile 3: Sandbox/Test (Thực tập & Khách) |
| :--- | :--- | :--- | :--- |
| **Tên Profile** | `CarePulse_Work` | `Staff_Personal` | `Intern_Sandbox` |
| **Mục đích sử dụng** | Bác sĩ trực chính, Điều dưỡng kê đơn, mở HIS bệnh viện, tra cứu cổng BHYT. | Nhân viên/thực tập sinh tra cứu học tập, vào LMS, GitHub, việc riêng. | Sinh viên thực tập luân phiên thực hành nhập liệu thử nghiệm hoặc khách dùng tạm. |
| **Màu Theme nhận diện** | **Xanh dương / Teal** (Nghiêm túc, chuẩn nhận diện y tế) | **Xám / Xanh lá nhạt** (Trung tính, thư giãn) | **Cam / Vàng cảnh báo** (Cảnh báo vùng dữ liệu tạm) |
| **Độc lập Cookies & Sessions** | Cô lập tại `Profile 1`. Không bao giờ bị ghi đè bởi thao tác đăng nhập bên ngoài. | Lưu tại `Profile 2`. Tự do đăng nhập GitHub, email học tập, diễn đàn. | Lưu tại `Profile 3`. Xóa sạch dữ liệu sau mỗi ca thực tập. |
| **Độc lập History & Cache** | Chỉ chứa URL nghiệp vụ. Tuyệt đối sạch khi cắm máy chiếu hội chẩn. | Chứa lịch sử tìm kiếm, mua sắm cá nhân; hoàn toàn tách biệt khỏi máy chiếu. | Lịch sử thao tác của thực tập sinh; không lẫn vào hồ sơ bệnh viện. |
| **Độc lập Bookmarks** | Chỉ chứa các thư mục chuyên môn chuẩn hóa, tối ưu diện tích. | Bookmark bài giảng, tài liệu nghiên cứu cá nhân. | Bookmark bộ tài liệu hướng dẫn và biểu mẫu thực hành mẫu. |
| **Độc lập Extensions** | Chỉ cài tiện ích bảo mật (Adblock, chữ ký số điện tử y tế). | Cài tiện ích học tập (Google Dịch, GitHub Dev Tools). | Không cài tiện ích hoặc chỉ cài extension phục vụ kiểm thử. |

---

## PHẦN 2: THIẾT KẾ HỆ THỐNG BOOKMARK & QUY TRÌNH AN TOÀN

### 3. Cây thư mục Bookmark chuẩn (`Ctrl + Shift + O`)

```text
Thanh dấu trang (Bookmarks Bar)
├── 📁 01_CarePulse_Clinic
│   ├── CarePulse HIS (Hệ thống thông tin quản lý bệnh viện nội bộ)
│   ├── Cổng giám định BHYT (Hệ thống thông tin giám định BHYT Quốc gia)
│   └── Hồ sơ Sức khỏe Điện tử Phường (Quản lý dữ liệu sức khỏe dân cư)
├── 📁 02_Pharma_Portal
│   ├── Tra cứu Dược thư Quốc gia Việt Nam (Cục Quản lý Dược)
│   ├── Ngân hàng Dữ liệu Ngành Dược (DrugBank Việt Nam)
│   └── Trung tâm Quốc gia về Thông tin thuốc và Theo dõi phản ứng có hại (ADR)
└── 📁 03_Medical_Tools
    ├── UpToDate / Medscape Clinical Reference (Tra cứu phác đồ điều trị lâm sàng)
    ├── MDCalc (Công cụ tính liều lượng thuốc và chỉ số y khoa)
    └── Cổng Telehealth (Hội chẩn y khoa trực tuyến tuyến trên)
