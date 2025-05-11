# jail o party

![image](https://github.com/user-attachments/assets/59800047-232d-4a56-b988-d675c31e97f5)



![image](https://github.com/user-attachments/assets/92075798-34ff-46c9-8eb6-ca9590db2586)
![image](https://github.com/user-attachments/assets/a25ea6d6-abed-46cd-995d-51f7014285cb)

Đầu tiên đọc src em thấy để lấy được flag bước đầu tiên là phải vượt qua được hàm checkAUth, sau đó là thiết lập phiên hợp lệ req.session.role = ADMIN = 0


![image](https://github.com/user-attachments/assets/123bc75b-97c8-4d0c-ad54-1bafbdab1c78)
## Bước 1: Thiết lập phiên hợp lệ (`req.session.valid = true`)
- **Mục tiêu**: Gửi yêu cầu để đặt `req.session.valid = true`, giúp vượt qua hàm `checkAuth()`.
- **Thách thức**: HAProxy chặn các yêu cầu tới đường dẫn bắt đầu bằng `/register` với quy tắc `http-request deny if { path_beg /register }`.
- **Giải pháp**: 
  - Quy tắc `path_beg /register` không phân biệt hoa thường trong thử thách này.
  - Sử dụng `POST /Register` (chữ 'R' in hoa) để vượt qua quy tắc.
Gửi yêu cầu `POST /Register` để đăng ký và nhận cookie phiên với `req.session.valid = true`.

## Bước 2: Bypassing quy tắc body của HAProxy
- **Mục tiêu**: Vượt qua quy tắc `http-request deny if { req.body -m sub dHJ1ZQ }` và thỏa mãn logic ứng dụng `if (atob(req.body.confirm) === "true")`.
- **Thách thức**: 
  - HAProxy chặn các yêu cầu có chuỗi `dHJ1ZQ` (base64 của "true") trong phần thân.
  - Ứng dụng yêu cầu `req.body.confirm` phải giải mã base64 thành `"true"`.
- **Giải pháp**: 
  - Sử dụng ký tự xuống dòng (`%0A`) để ngắt chuỗi `dHJ1ZQ`.
  - Payload: `confirm=dHJ1%0AZQ==`.
    - HAProxy không phát hiện `dHJ1ZQ` do bị ngắt.
    - Ứng dụng giải mã `atob("dHJ1\nZQ==")` thành `"true"`.
Gửi yêu cầu với payload `confirm=dHJ1%0AZQ==`.

## Bước 3: Đặt `req.session.role = 0` 
- **Mục tiêu**: Gửi yêu cầu để đặt `req.session.role = 0`.
- **Thách thức**: 
  - HAProxy chỉ cho phép header `role = 1` với quy tắc `http-request deny if { hdr_val(role) lt 1 } || { hdr_val(role) gt 1 }`.
  - Ứng dụng yêu cầu `Number(req.headers.role) === 0`.
- **Giải pháp**: 
  - Sử dụng header `role: 1e-10000000000`.
    - HAProxy không từ chối giá trị này.
    - Express.js chuyển `Number("1e-10000000000")` thành `0`.
- **Hành động**: Gửi yêu cầu `HEAD /admin` với header `role: 1e-10000000000`.

## Bước 4: Truy cập  `/admin` để lấy flag
Lấy flag từ  `/admin`.
- **Giải pháp**: 
  - Sử dụng cookie phiên từ bước 1.
  - Gửi yêu cầu `POST /admin`.
- **Kết quả**: Ứng dụng trả về flag.
![Screenshot 2025-05-11 144928](https://github.com/user-attachments/assets/d8757084-5c73-404c-b9c4-864f6ea57a39)
![Screenshot 2025-05-11 145717](https://github.com/user-attachments/assets/37b66916-b85a-4df0-aca8-7015705ade59)
![Screenshot 2025-05-11 145701](https://github.com/user-attachments/assets/98368465-6492-4a94-85c6-72a7e28cad66)

