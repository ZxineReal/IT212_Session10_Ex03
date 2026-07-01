1. Prompt thiết kế cho Antigravity
# ROLE

Bạn là Senior System Analyst kiêm Business Analyst có kinh nghiệm xây dựng tài liệu SRS theo chuẩn IEEE 830/ISO/IEC/IEEE 29148.

# CONTEXT

Hệ thống Guai-api đã được index toàn bộ source code. Hãy phân tích module Cart, đặc biệt là phương thức `CartService.addToCart()`.

# OBJECTIVE

Thực hiện đồng thời hai nhiệm vụ:

## Nhiệm vụ 1 – Phân tích lỗ hổng logic

Đọc và phân tích logic hiện tại của phương thức `addToCart()`.

Xác định các lỗ hổng nghiệp vụ, bao gồm nhưng không giới hạn ở:

* Quantity có thể bằng 0 hoặc âm.
* Quantity thêm vào làm tổng số lượng trong giỏ bị âm.
* Quantity vượt quá số lượng tồn kho (Inventory).
* Không kiểm tra trạng thái sản phẩm (nếu cần).
* Các trường hợp có thể dẫn đến dữ liệu không hợp lệ hoặc lỗi nghiệp vụ.

Đối với mỗi lỗ hổng, trình bày theo cấu trúc:

* ID
* Vị trí trong code
* Mô tả lỗ hổng
* Rủi ro
* Hậu quả đối với hệ thống
* Đề xuất hướng xử lý

---

## Nhiệm vụ 2 – Sinh Business Rules chuẩn SRS

Xuất danh sách Business Rules để Dev sử dụng làm căn cứ vá lỗi.

Yêu cầu:

* Viết theo chuẩn SRS.
* Đánh mã BR-001, BR-002,...
* Mỗi Business Rule gồm:

  * Rule ID
  * Rule Name
  * Description
  * Condition
  * System Response
  * Error Message (nếu có)
  * Priority (High/Medium/Low)

Các Business Rules phải bao phủ tối thiểu:

1. Quantity phải lớn hơn 0.
2. Không cho phép quantity bằng 0.
3. Không cho phép quantity âm.
4. Tổng số lượng trong giỏ không được vượt quá tồn kho.
5. Nếu sản phẩm hết hàng thì từ chối thêm vào giỏ.
6. Nếu sản phẩm không tồn tại thì trả về lỗi.
7. Nếu quantity hợp lệ thì cập nhật hoặc tạo mới CartItem.
8. Không được tạo dữ liệu giỏ hàng có số lượng âm hoặc không hợp lệ.
9. Mọi vi phạm nghiệp vụ phải trả về thông báo lỗi rõ ràng.

---

# OUTPUT FORMAT

## Part A

Logic Issues

| ID | Location | Description | Risk | Recommendation |

## Part B

Business Rules (SRS)

| Rule ID | Rule Name | Description | Condition | System Response | Error Message | Priority |

Không sinh mã nguồn Java.
Chỉ tập trung vào phân tích nghiệp vụ và Business Rules theo chuẩn SRS.
2. Danh sách Business Rules (đã tích hợp vá lỗi logic)
# Business Rules (SRS)

| Rule ID | Rule Name                     | Description                                                                                               | Condition                                           | System Response                              | Error Message                               | Priority |
| ------- | ----------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------- | ------------------------------------------- | -------- |
| BR-001  | Product Must Exist            | Chỉ cho phép thêm sản phẩm đã tồn tại trong hệ thống.                                                     | Product ID không tồn tại.                           | Từ chối yêu cầu thêm vào giỏ hàng.           | Product not found.                          | High     |
| BR-002  | Quantity Must Be Positive     | Số lượng thêm vào giỏ phải lớn hơn 0.                                                                     | Quantity ≤ 0.                                       | Không thực hiện thêm hoặc cập nhật giỏ hàng. | Quantity must be greater than 0.            | High     |
| BR-003  | Reject Negative Quantity      | Không cho phép sử dụng giá trị âm để giảm hoặc thao tác số lượng khi thêm vào giỏ hàng.                   | Quantity < 0.                                       | Từ chối yêu cầu.                             | Invalid quantity.                           | High     |
| BR-004  | Reject Zero Quantity          | Không cho phép thêm sản phẩm với số lượng bằng 0.                                                         | Quantity = 0.                                       | Từ chối yêu cầu.                             | Quantity must be greater than 0.            | High     |
| BR-005  | Inventory Validation          | Tổng số lượng của sản phẩm trong giỏ không được vượt quá số lượng tồn kho hiện có.                        | Existing Quantity + Requested Quantity > Inventory. | Không cập nhật giỏ hàng.                     | Requested quantity exceeds available stock. | High     |
| BR-006  | Out-of-Stock Restriction      | Không cho phép thêm sản phẩm đã hết hàng.                                                                 | Inventory = 0.                                      | Từ chối thêm sản phẩm.                       | Product is out of stock.                    | High     |
| BR-007  | Update Existing Cart Item     | Nếu sản phẩm đã tồn tại trong giỏ, hệ thống cập nhật số lượng sau khi vượt qua tất cả kiểm tra nghiệp vụ. | CartItem tồn tại và dữ liệu hợp lệ.                 | Cộng thêm số lượng và lưu dữ liệu.           | N/A                                         | Medium   |
| BR-008  | Create New Cart Item          | Nếu sản phẩm chưa có trong giỏ và dữ liệu hợp lệ, hệ thống tạo mới CartItem.                              | CartItem chưa tồn tại và dữ liệu hợp lệ.            | Tạo mới bản ghi giỏ hàng.                    | N/A                                         | Medium   |
| BR-009  | Prevent Invalid Cart Data     | Hệ thống không được lưu CartItem có số lượng âm, bằng 0 hoặc vượt tồn kho.                                | Dữ liệu vi phạm quy tắc nghiệp vụ.                  | Hủy thao tác lưu dữ liệu.                    | Invalid cart data.                          | High     |
| BR-010  | Atomic Validation Before Save | Toàn bộ điều kiện nghiệp vụ phải được kiểm tra trước khi gọi `cartRepository.save()`.                     | Trước khi ghi dữ liệu.                              | Chỉ lưu khi tất cả điều kiện đều hợp lệ.     | Validation failed.                          | High     |
