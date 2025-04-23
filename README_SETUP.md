# Frappe Student App Quick Setup

## 1. Bật developer_mode

Thêm vào **cả hai file** sau:
- `sites/localhost/site_config.json`
- `sites/common_site_config.json`

```json
{
  ...
  "developer_mode": 1
}
```

## 2. Tạo Doctype Student và file student.py

- Tạo file: `apps/app_student/app_student/app_student/doctype/student/student.json`
- Tạo file: `apps/app_student/app_student/app_student/doctype/student/student.py`

> **Gợi ý:**
> - File `student.json` định nghĩa các trường (field) cho Student (xem ví dụ mẫu trong thư mục này).
> - File `student.py` có thể để trống hoặc kế thừa từ `frappe.model.document.Document`.

## 3. Tạo trang CRUD cho Student

- Tạo file: `apps/app_student/app_student/templates/pages/crud_student.html`
- Trang này sẽ dùng để thêm/sửa/xoá sinh viên qua giao diện web.

## 4. Thêm route vào `hooks.py`

Trong `apps/app_student/app_student/hooks.py` thêm:

```python
website_route_rules = [
    {"from_route": "/student", "to_route": "crud_student"}
]
```

## 5. Reload DocType và khởi động lại bench

Sau khi tạo/chỉnh sửa các file trên, chạy các lệnh sau:

```bash
bench --site localhost reload-doc app_student doctype student
bench clear-cache && bench restart
```

## 6. Truy cập giao diện CRUD

Mở trình duyệt và truy cập:

```
http://localhost:8001/student
```

Bạn sẽ thấy giao diện quản lý sinh viên (CRUD Student).

---

### Ví dụ thao tác với console Frappe

```python
In [1]: frappe.reload_doc('app_student', 'doctype', 'student')
Out[1]: True
```

---
**Nếu gặp lỗi hoặc cần thêm hướng dẫn, hãy kiểm tra lại các bước trên hoặc liên hệ admin.**
