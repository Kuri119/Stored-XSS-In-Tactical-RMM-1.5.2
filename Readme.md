**Lỗ hổng :** Lưu trữ mã bất hợp pháp (Stored XSS) tại chức năng quản lý tên thiết bị/policy/script/client/site dẫn đến nguy cơ thực thi lệnh trái phép trong phiên làm việc của quản trị viên, chiếm quyền kiểm soát tài khoản hàng loạt

**Mô tả:** Ứng dụng không thực hiện việc lọc dữ liệu đầu vào và mã hóa dữ liệu đầu ra đối với trường thông tin tên thiết bị, tên policy, script, client và site. Khi người dùng nhập chuỗi mã HTML/JavaScript độc hại vào các trường này, dữ liệu sẽ được lưu trực tiếp vào CSDL.
Mã độc sẽ tự động thực thi trên trình duyệt của bất kỳ người dùng nào (đặc biệt là Quản trị viên/Kỹ thuật viên) khi họ truy cập vào danh sách quản lý hoặc mở các ô chọn thả xuống (Dropdown) chứa các đối tượng này.

**Nguy cơ:**
•	Chiếm đoạt quyền kiểm soát tài khoản (Account Takeover - ATO)
o	Thao tác mạo danh: Kẻ tấn công đính kèm access_token thu thập được vào API header để giả mạo nạn nhân thực hiện toàn bộ chức năng: thay đổi thông tin cá nhân, đổi mật khẩu, xóa/thêm cấu hình xác thực 2FA.
o	Bypass xác thực: Bỏ qua hoàn toàn các lớp bảo mật bề mặt (mật khẩu mạnh, OTP/2FA) vì token thu được đã đại diện cho một phiên làm việc hợp lệ.
•	Rủi ro tác động hạ tầng RMM (Critical Infrastructure Threat)
o	Đánh sập/Thao túng thiết bị hàng loạt (Mass Device Disruption): Nếu tài khoản bị chiếm đoạt sở hữu quyền quản trị hệ thống RMM, kẻ tấn công có thể lợi dụng các API quản lý để gửi lệnh tắt máy, reboot, ngắt kết nối mạng hoặc hủy dịch vụ trên tất cả các Endpoint/Agent thuộc quyền quản lý của tài khoản đó.
o	Thực thi mã từ xa hàng loạt (Mass Remote Code Execution - RCE): Thông qua tính năng chạy Script/Command tự động của RMM, kẻ tấn công có thể phát tán Malware, Ransomware hoặc script độc hại (đã đề xuất) để xóa sạch dữ liệu trên toàn bộ danh sách thiết bị chỉ với một lệnh duy nhất.
o	Mất khả năng sẵn sàng (Availability Impact - High): Gây ngưng trệ toàn bộ hoạt động vận hành của doanh nghiệp, làm gián đoạn các dịch vụ CNTT cốt lõi.
•	Rủi ro leo thang đặc quyền & Chiếm quyền quản trị cao nhất (Privilege Escalation)
o	Nếu nạn nhân dính payload là Admin/Super Admin, kẻ tấn công nghiễm nhiên nắm quyền tối cao trên toàn bộ hệ thống RMM mà không cần tốn thêm công sức leo thang đặc quyền.

**Vị trí:**
URL	Param
POST /clients/
PUT /clients/
PUT /clients/sites/
POST /automation/policies/
PUT /automation/policies/
POST /scripts/
PUT /scripts/	name


**Bằng chứng:**

Kịch bản tấn công ở đây, ta sử dụng tài khoản longnbh_2 (người dùng có quyền hạn chế) sẽ thực hiện stored XSS vào tên client và khi nạn nhân (ở đây là longnbh - admin) thực hiện chức năng dropdown để tìm client trong chức năng Audit log thì sẽ kích hoạt payload dẫn tới Authorization token của nạn nhân được chuyền về cho kẻ tấn công lại webhook và từ đây kẻ tấn công chiếm đoạt hoàn toàn tài khoản nạn nhân.
!(POC/Screenshot 2026-10-30 103703.png) 
 

**Bước 1: Thực hiện điền stored XSS**
Để có client với tên là stored XSS thì ta có thể tạo hoặc edit tên một client đã có từ trước, đây là ta thực hiện edit một client đã có từ trước.
Chức năng edit client (PUT /clients/)
  
Khi nạn nhân truy cập vào audit log, chọn dropdown client, thông báo hiện ra cho thấy script XSS đã được thực thi thành công.
 
Cùng kịch bản này thay vì script alert() ta có thể thay bằng mã lệnh độc hại nhằm thu thập thông tin Authorization token của người dùng khi họ bị dính payload.
 

**Bước 2: Nạn nhân khi thực hiện chức năng dropdown để chọn clients**
Người dùng admin (longnbh) thực hiện truy cập vào audio log và chọn phần dropdown clients
 
 

**Bước 3: Thay đổi mật khẩu và 2FA người dùng bắt được, chiếm quyền kiểm soát tài khoản thành công**
Từ token có được kẻ tấn công thực hiện thao tác thay đổi mật khẩu người dùng và reset 2FA chiếm quyền kiểm soát tài khoản nạn nhân.
 
 
Mật khẩu và 2FA đã được đổi, từ đấy kẻ tấn công chiếm hoàn toàn tài khoản admin longnbh

Lưu ý: Ta có thể thực hiện các thao tác nhúng payload tương tự trong các chức năng thêm và sửa site, script, policy và payload sẽ thực thi khi người dùng truy cập vào dropdown chứa các đối tượng bị nhúng này.
