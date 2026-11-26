# Test case: Real-Time Sync Cart (Cart ↔ Delivery Schedule)

File test: `index.html` (mở bằng Chrome/Edge, double-click là chạy). Trên máy tính app hiện trong khung điện thoại 390×844, trên điện thoại hiện full màn hình.

Mọi chữ tiếng Anh trong cột "Kết quả mong đợi" là chữ hiển thị trên UI, so sánh đúng từng ký tự.

---

## 0. Quy ước và chuẩn bị

### 0.1 Quy ước

| Ký hiệu | Ý nghĩa |
|---|---|
| **[RESET]** | Tab **My Profile** → bottom sheet "Demo controls" → bấm **Đặt lại toàn bộ dữ liệu demo**. Kết quả: toast "Đã đặt lại dữ liệu demo", giỏ về trạng thái gốc, xóa mọi lịch giao, Order ID về 2677. |
| **[Ngày X]** | Ngày giao tự chọn. Lịch chỉ cho chọn **từ ngày mai trở đi**. Video dùng 19/11/2026, chỉ chọn được nếu hôm nay ≤ 18/11/2026. Ngày hôm nay hiện ra có viền tròn, các ngày ≤ hôm nay bị mờ. |
| **Loading** | Lớp mờ trắng có vòng xoay và chữ (ví dụ "Loading...", "Saving...") hiện ngắn dưới 1 giây khi chuyển màn hoặc lưu. |
| **Banner vàng** | Dòng chữ cam trên nền vàng nhạt dưới tiêu đề "Cart": "Some items have quantity mismatches between the cart and delivery schedule." |
| **Popup đỏ** | Popup căn giữa: tiêu đề đậm căn giữa, nội dung chữ xám, nút chính màu **đỏ** nằm trên, nút Cancel viền xám nằm dưới. |

### 0.2 Giỏ hàng gốc (sau [RESET])

| Sản phẩm | UOM | Đơn giá | Số lượng | Total dòng |
|---|---|---|---|---|
| BAKING SODA (SODIUM BICARBONATE) | BAG | $1.30 | 10 | $13.00 |
| ACTIVE-GO | CAR | $13.77 | 20 | $275.40 |
| ACTIVE-GO | TIN | $13.77 | 30 | $413.10 |
| WHOLEMEAL FLOUR - HCS | BAG | $18.00 (giá gạch $20.00) | 40 | $720.00 |

Footer Cart: **100 items**, **$1,549.44** (đã gồm GST 9%). Giảm giá "HCS Trade Discount Discount (Flour): -$80.00".

### 0.3 Trạng thái dựng sẵn (dùng làm điều kiện đầu cho nhiều test case)

**[PRE-S0]**: giao ít hơn giỏ, chưa Update Cart
1. [RESET] → tab Cart → **Add Delivery**.
2. Chọn [Ngày X].
3. Nhập ACTIVE-GO TIN = 10, CAR = 5; BAKING SODA BAG = 5; WHOLEMEAL FLOUR BAG = 10.
4. **Save** → **Done**.

Kết quả: Cart giữ 100 items $1,549.44, có 4 dòng "Unallocated" (xem TC-05 bước 4).

**[PRE-S1]**: như PRE-S0, rồi tiếp
5. **Check Out** → popup → **Update Cart**.

Kết quả: giỏ BS 5, CAR 5, TIN 10, WF 10. Footer **30 items, $428.42**. Không còn banner, không còn nút "Add Delivery", chỉ có link "1st Schedule".

---

## TC-01. Màn Cart ban đầu

Điều kiện: [RESET], đang ở tab Cart.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Nhìn phần đầu màn | Tiêu đề **Cart**. Khối xanh nhạt: "Customer Name: **SGQ CUSTOMER**", "Address: **232 PANDAN LOOP 232 WILMAR INTERNATIONAL SINGAPORE 128420**". **Không** có banner vàng. |
| 2 | Xem danh sách sản phẩm (cuộn xuống) | Thứ tự: **BAKING SODA**, **ACTIVE-GO**, **WHOLEMEAL FLOUR - HCS**.<br>Mỗi sản phẩm có: nút thùng rác đỏ nhạt bên trái, ảnh (icon "không có ảnh"), tên, "Brand: ...", khung "Pack Size: ...", dòng UOM, giá, ô "Total: $...", stepper −/số/+.<br>BAKING SODA: Brand: No Brand, Pack Size: IN 20 X 1KG BAG PER CAR, UOM (BAG) $1.30, số lượng 10, Total: $13.00.<br>ACTIVE-GO: Brand: MILO, Pack Size: IN 6 X 1.8KG TIN PER CAR, hai dòng: UOM (CAR) $13.77 SL 20 Total: $275.40; UOM (TIN) $13.77 SL 30 Total: $413.10 (có vạch ngăn giữa hai dòng).<br>WHOLEMEAL FLOUR - HCS: Brand: SW WHOLEMEAL, nhãn xanh **Halal**, góc khuyến mãi (huy hiệu %) trên ảnh, Pack Size: IN 25KG PAPER BAG, UOM (BAG) **$18.00** kèm giá gạch **$20.00**, SL 40, Total: $720.00, dòng "HCS Trade Discount Discount (Flour): -$80.00" kèm icon % xanh dương. |
| 3 | Cuộn xuống cuối danh sách | Thanh xám **Delivery Schedule**. Bên dưới là nút nền xanh nhạt **(+) Add Delivery**. |
| 4 | Nhìn footer | "Total:" và dòng nhỏ "**100 items**" bên dưới, số tiền bên phải **$1,549.44**. Nút **Check Out** màu xám nhạt, chữ trắng (không bấm được). |
| 5 | Bấm nút Check Out đang mờ | Không có phản ứng gì. |
| 6 | Nhìn thanh điều hướng dưới | 5 tab: Home, Category, Promotion, **Cart** (đang active, màu xanh đậm, có badge **4** trên icon), My Profile. Góc dưới phải có ruy băng chéo "UAT". |
| 7 | Bấm tab **Home**, rồi **Promotion** | Mỗi lần hiện toast: "Bản demo này chỉ mô phỏng luồng Cart & Delivery Schedule". Vẫn ở màn Cart. |

---

## TC-02. Mở màn Delivery Schedule và xem form

Điều kiện: [RESET], tab Cart.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm **Add Delivery** | Loading ngắn → chuyển sang màn **Delivery Schedule**. |
| 2 | Nhìn thanh tiêu đề | Nút quay lại "<" bên trái; tiêu đề **Delivery Schedule**; bên phải 2 nút vuông: **thùng rác đỏ** (bấm được) và **"+" xám mờ** (không bấm được). |
| 3 | Nhìn dưới tiêu đề | Nhãn "Incoterm", ô viền xanh giá trị **Delivery** (mặc định là Delivery, không phải Self-Pickup) với mũi tên ">". Thanh xám "Delivery Schedule". |
| 4 | Nhìn accordion | Hàng nền xanh nhạt "**1st Schedule**" kèm chip cam nhỏ **Unsaved**, mũi tên "^" (đang mở). |
| 5 | Nhìn phần thân accordion | "Incoterm": ô **xám**, chữ "Delivery" (không sửa được).<br>"Address" (icon ghim): thẻ viền xanh, dòng đậm **nhan.pham**, dòng xanh **517E JURONG WEST STREET 52 SINGAPORE 645517**, mũi tên ">".<br>"Delivery/Self-Pickup Date" có dấu **\*** đỏ: ô nền xanh nhạt hiện **Choose Date**.<br>"Delivery Instruction": ô nhập nhiều dòng, chữ mờ "Enter Delivery Instruction". |
| 6 | Cuộn xuống phần sản phẩm | 3 khối theo thứ tự: **ACTIVE-GO**, **BAKING SODA (SODIUM BICARBONATE)**, **WHOLEMEAL FLOUR - HCS**. Mỗi khối có ảnh, tên, Brand, nút **Reset** (nền xanh nhạt), khung Pack Size.<br>ACTIVE-GO có **hai** dòng: **UOM (TIN)** rồi **UOM (CAR)** (TIN đứng trước).<br>Mỗi UOM có stepper, số lượng đang là **0** (nút − mờ) và dòng bên dưới: TIN "Available **(30)** product in order", CAR "Available **(20)**", BAKING SODA BAG "Available **(10)**", WHOLEMEAL BAG "Available **(40)**". |
| 7 | Cuộn tới cuối | Nút **Save** (viền xanh, nền xanh nhạt). Footer cố định: nút **Done** màu xám (không bấm được). |
| 8 | Bấm "<" (chưa sửa gì) | Quay lại Cart ngay, **không** hiện popup. Cart vẫn hiện "Add Delivery", không có lịch nào được tạo. |

---

## TC-03. Chọn ngày giao (lịch)

Điều kiện: đang ở màn Delivery Schedule, 1st Schedule mở (làm lại TC-02 bước 1).

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm ô **Choose Date** | Nền tối + popup lịch. Đầu popup nền xanh đậm: chữ nhỏ "SELECT DATE" và dòng lớn **Choose Date**. Dưới là tháng hiện tại (ví dụ "November 2026"), lưới S M T W T F S.<br>Các ngày ≤ hôm nay bị mờ; hôm nay có viền tròn. Mũi tên "<" tháng trước mờ. Hai nút chữ **CANCEL** và **OK**; **OK mờ**. |
| 2 | Bấm một ngày hợp lệ (ví dụ [Ngày X]) | Ngày đó thành vòng tròn xanh đậm. Dòng lớn trên đầu đổi thành dạng "**Thu, Nov 19**" (thứ viết tắt, tháng viết tắt, ngày). **OK** sáng lên. |
| 3 | Bấm ">" | Chuyển sang "**December 2026**", nút "<" sáng. Bấm "<" quay lại tháng trước. |
| 4 | Bấm **CANCEL** | Popup đóng, ô vẫn ghi "Choose Date" (nếu chưa từng chọn). |
| 5 | Mở lại, chọn ngày, bấm **OK** | Popup đóng. Ô ngày hiện dạng **dd/mm/yyyy** (ví dụ 19/11/2026). Chip "Unsaved" vẫn còn. |
| 6 | Mở lại lịch, bấm vào vùng tối bên ngoài popup | Popup đóng, ngày đã chọn giữ nguyên. |

---

## TC-04. Kiểm tra dữ liệu khi Save

Điều kiện: màn Delivery Schedule, 1st Schedule mở, chưa chọn ngày.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm **Save** khi chưa chọn ngày | Ô ngày viền **đỏ**, dưới ô có dòng đỏ "**Please choose Delivery/Self-Pickup Date**", đồng thời toast đỏ cùng nội dung. Chip vẫn "Unsaved", Done vẫn mờ. |
| 2 | Chọn [Ngày X] (dòng đỏ biến mất), để mọi số lượng = 0, bấm **Save** | Toast đỏ "**Please allocate at least 1 product to this schedule**". Không lưu. |
| 3 | Ở ACTIVE-GO TIN gõ `abc` rồi bấm ra ngoài | Ô về **0**. |
| 4 | Gõ `12abc` rồi bấm ra ngoài | Ô thành **12** (chữ bị bỏ). |
| 5 | Gõ `99999` rồi bấm ra ngoài | Ô thành **9999**. Dưới ô có note xanh "Exceeds cart by 9969 TIN · cart will be increased to 9999 TIN on Save". |
| 6 | Gõ `0` | Ô = 0, nút **−** mờ, note xanh biến mất. |

---

## TC-05. Luồng 1: giao ít hơn giỏ → Update Cart → Order Confirmation → Submit

Điều kiện: [RESET], tab Cart.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | **Add Delivery** → chọn [Ngày X] | Màn lịch giao, Incoterm = Delivery, địa chỉ nhan.pham, ô ngày hiện dd/mm/yyyy. |
| 2 | Nhập **TIN = 10** | Dòng dưới ô: "Available **(20)** product in order". |
| 3 | Nhập **CAR = 5** | "Available **(15)**". |
| 4 | Nhập BAKING SODA **BAG = 5** | "Available **(5)**". |
| 5 | Nhập WHOLEMEAL **BAG = 10** | "Available **(30)**". Không có note xanh/cam nào (không vượt giỏ). Chip vẫn **Unsaved**. |
| 6 | Bấm **Save** | Loading "Saving..." → toast xanh lá "**1st Schedule saved successfully**". Chip "Unsaved" biến mất. Nút Save đổi thành "✓ **Saved**" và mờ. **Done** sáng lên (xanh đậm). Nút **"+"** trên tiêu đề sáng lên. |
| 7 | Bấm **Done** | Về màn Cart. **Banner vàng** hiện trên cùng.<br>Số lượng và Total các dòng **giữ nguyên** (10/20/30/40). Footer vẫn **100 items, $1,549.44**.<br>Dưới ô số lượng có chữ cam đậm: BAKING SODA "**Unallocated: 5 BAG**"; ACTIVE-GO CAR "**Unallocated: 15 CAR**", TIN "**Unallocated: 20 TIN**"; WHOLEMEAL "**Unallocated: 30 BAG**".<br>Nút **Check Out** sáng (xanh đậm).<br>Mục Delivery Schedule: dòng "**1st Schedule**" (chữ xanh) với dòng nhỏ "Delivery · dd/mm/yyyy" và mũi tên ">"; bên dưới nút "**Add Delivery (2nd Schedule)**". |
| 8 | Bấm **Check Out** | Popup đỏ **Update Cart Quantity**: "The scheduled quantity doesn't match your cart." rồi 4 dòng:<br>• BAKING SODA (SODIUM BICARBONATE): 5 BAG not assigned<br>• ACTIVE-GO: 15 CAR not assigned<br>• ACTIVE-GO: 20 TIN not assigned<br>• WHOLEMEAL FLOUR - HCS: 30 BAG not assigned<br>Hai nút: **Update Cart** (đỏ) và **Cancel**. |
| 9 | Bấm **Update Cart** | Loading "Updating cart..." → toast "**Cart quantity updated to match the delivery schedule**".<br>Banner và mọi dòng Unallocated **biến mất**.<br>Số lượng: BAG soda **5** (Total $6.50), CAR **5** ($68.85), TIN **10** ($137.70), WHOLEMEAL **10** ($180.00).<br>Dòng giảm giá thành "HCS Trade Discount Discount (Flour): **-$20.00**".<br>Footer **30 items, $428.42**. Nút "Add Delivery (2nd Schedule)" **biến mất**, chỉ còn link "1st Schedule". |
| 10 | Bấm link **1st Schedule** | Màn lịch giao, 1st Schedule mở với TIN 10, CAR 5, BAG 5, BAG 10. Cả 4 dòng "Available **(0)** product in order". Nút "+" sáng, Done sáng. Bấm **Done** quay lại Cart. |
| 11 | Bấm **Check Out** | Loading → màn **Order Confirmation** (không có popup). |
| 12 | Xem màn Order Confirmation (cuộn) | Tiêu đề "Order Confirmation", nút "<".<br>Hàng nền xanh nhạt "Order ID: **2677**" với chip cam "Pending Submit".<br>Thẻ **Customer**: Customer Name SGQ CUSTOMER; Address 232 PANDAN LOOP ....<br>Thẻ **Order Information**: Order Date = hôm nay (dd/mm/yyyy); **Validity Period** = hôm nay → hôm nay + 30 ngày (ví dụ "17/11/2026 - 17/12/2026"); **Payment Term** = 30 days; Incoterm = Delivery.<br>Thẻ **Delivery Schedule**: "1st Schedule", "Delivery/Self-Pickup Date: [Ngày X]", "Address: nhan.pham – 517E JURONG WEST STREET 52 SINGAPORE 645517", rồi 3 dòng "ACTIVE-GO: 10 TIN, 5 CAR", "BAKING SODA (SODIUM BICARBONATE): 5 BAG", "WHOLEMEAL FLOUR - HCS: 10 BAG".<br>Thẻ **Order Items (30 items)**: BAKING SODA 5 BAG × $1.30 = $6.50; ACTIVE-GO 5 CAR × $13.77 = $68.85 và 10 TIN × $13.77 = $137.70; WHOLEMEAL 10 BAG × $18.00 = $180.00 kèm dòng "HCS Trade Discount Discount (Flour): -$20.00".<br>Thẻ **Summary**: Subtotal (before discount) **$413.05**; HCS Trade Discount **-$20.00**; Total Discount **-$20.00**; Subtotal **$393.05**; GST (9%) **$35.37**; Total **$428.42** (chữ đậm lớn).<br>Footer: "Total: / 30 items", **$428.42**, nút **Submit**. |
| 13 | Bấm "<" | Về Cart (giỏ vẫn $428.42, 30 items). |
| 14 | Bấm **Check Out** lại | Vào lại Order Confirmation, Order ID vẫn **2677** (không đổi). |
| 15 | Bấm **Submit** | Loading "Submitting order..." → popup "**Order Submitted**": "Order ID 2677 has been submitted successfully." với 1 nút **OK**. Bấm vùng tối ngoài popup **không** đóng được. |
| 16 | Bấm **OK** | Về Cart rỗng: icon giỏ xám, "**Your cart is empty**", dòng "Demo: khôi phục giỏ hàng ban đầu để chạy lại luồng.", nút **Khôi phục giỏ hàng demo**. Footer "0 items", "$0.00", Check Out mờ. Badge trên tab Cart biến mất. |
| 17 | Bấm **Khôi phục giỏ hàng demo** | Toast "Đã khôi phục giỏ hàng demo". Giỏ về 4 dòng gốc, $1,549.44, 100 items, có "Add Delivery". Đơn kế tiếp sẽ mang Order ID **2678**. |

---

## TC-06. Luồng 2: giao nhiều hơn giỏ + ít hơn giỏ → Cancel → thêm 2nd Schedule

Điều kiện: [RESET], tab Cart.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | **Add Delivery** → chọn [Ngày X] | Màn lịch giao như TC-02. |
| 2 | ACTIVE-GO **TIN = 40** (giỏ đang 30) | Ô nhận 40 (không bị cắt). "Available **(0)** product in order". Dưới có note **xanh dương**: "**Exceeds cart by 10 TIN · cart will be increased to 40 TIN on Save**". |
| 3 | **CAR = 15** (giỏ 20) | "Available **(5)**". Không có note. |
| 4 | BAKING SODA **BAG = 20** (giỏ 10) | "Available **(0)**". Note xanh: "Exceeds cart by 10 BAG · cart will be increased to 20 BAG on Save". |
| 5 | WHOLEMEAL **BAG = 35** (giỏ 40) | "Available **(5)**". Không có note. |
| 6 | Bấm **Save** | Toast xanh lá: "**1st Schedule saved successfully. Cart quantity increased for 2 items to match the schedule**". Nút thành "✓ Saved". Hai note xanh **biến mất** (giỏ đã được tăng bằng số trong lịch), TIN và BAG soda vẫn "Available (0)". |
| 7 | Bấm **Done** | Màn Cart, Banner vàng hiện.<br>BAKING SODA: SL **20**, Total **$26.00**, hàng nháy nền vàng nhạt khoảng 2 giây (không còn nhãn chữ nào). Không có chữ Unallocated.<br>ACTIVE-GO CAR: SL **20**, Total $275.40, "**Unallocated: 5 CAR**".<br>ACTIVE-GO TIN: SL **40**, Total **$550.80** (hàng này cũng nháy vàng nhạt).<br>WHOLEMEAL: SL **40**, "**Unallocated: 5 BAG**", giảm giá vẫn **-$80.00**.<br>Footer: **120 items, $1,713.70**. Check Out sáng. Có link "1st Schedule" và nút "Add Delivery (2nd Schedule)".<br>Lưu ý: hiệu ứng nháy vàng chỉ chạy ở lần hiển thị đầu tiên sau khi Done. |
| 8 | Bấm **Check Out** | Popup **Update Cart Quantity** chỉ có **2 dòng**:<br>• ACTIVE-GO: 5 CAR not assigned<br>• WHOLEMEAL FLOUR - HCS: 5 BAG not assigned<br>(không có BAKING SODA và TIN vì đã khớp lịch). Nút Update Cart (đỏ), Cancel. |
| 9 | Bấm **Cancel** | Popup đóng, **ở lại Cart**. Giỏ giữ **$1,713.70, 120 items**; banner vàng và 2 dòng Unallocated vẫn còn; vẫn có nút "Add Delivery (2nd Schedule)". |
| 10 | Bấm **Add Delivery (2nd Schedule)** | Loading → màn lịch giao. **1st Schedule thu gọn** (hiện ngày bên dưới tiêu đề), **2nd Schedule mở** với chip "Unsaved". Nút "+" mờ. Available: TIN (0), CAR **(5)**, BAKING SODA (0), WHOLEMEAL **(5)**. |
| 11 | Chọn [Ngày Y] cho 2nd Schedule; nhập CAR = 5, WHOLEMEAL BAG = 5 | Cả 4 dòng Available đều **(0)**. |
| 12 | **Save** | Toast "**2nd Schedule saved successfully**" (không có đoạn "Cart quantity increased"). |
| 13 | **Done** | Cart: **không** banner, không Unallocated. Hai link "**1st Schedule**" và "**2nd Schedule**" (có vạch ngăn). **Không** có nút Add Delivery. Footer vẫn $1,713.70, 120 items. |
| 14 | **Check Out** | Vào thẳng Order Confirmation, không có popup. Thẻ Delivery Schedule có **2 lịch**: 1st Schedule (ACTIVE-GO: 40 TIN, 15 CAR; BAKING SODA (SODIUM BICARBONATE): 20 BAG; WHOLEMEAL FLOUR - HCS: 35 BAG) và 2nd Schedule (ACTIVE-GO: 5 CAR; WHOLEMEAL FLOUR - HCS: 5 BAG). Summary: Subtotal (before discount) **$1,652.20**; HCS Trade Discount **-$80.00**; Total Discount **-$80.00**; Subtotal **$1,572.20**; GST (9%) **$141.50**; Total **$1,713.70**. "Order Items (120 items)". |

---

## TC-07. Incoterm và địa chỉ

Điều kiện: [RESET] → Add Delivery.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm ô **Incoterm** trên cùng | Bottom sheet tiêu đề "**Incoterm**" với 2 dòng: **Self-Pickup**, **Delivery** (Delivery có dấu ✓ và chữ đậm xanh). |
| 2 | Chọn **Self-Pickup** | Loading ~1 giây. Ô Incoterm trên cùng và ô xám trong 1st Schedule đều thành "**Self-Pickup**". Thẻ Address đổi thành **WILMAR PANDAN WAREHOUSE** / 232 PANDAN LOOP SINGAPORE 128420. |
| 3 | Bấm lại ô Incoterm, chọn **Delivery** | Loading → thẻ Address về **nhan.pham** / 517E JURONG WEST STREET 52 SINGAPORE 645517. |
| 4 | Bấm thẻ Address | Bottom sheet "**Select Address**": dòng **nhan.pham** (kèm địa chỉ nhỏ xanh bên dưới) có ✓; dòng **SGQ CUSTOMER** (232 PANDAN LOOP ...). |
| 5 | Chọn **SGQ CUSTOMER** | Sheet đóng, thẻ Address đổi sang SGQ CUSTOMER, chip "Unsaved". |
| 6 | Chọn [Ngày X], BAKING SODA BAG = 5, **Save**, **Done** | Về Cart. Mở lại **1st Schedule**: thẻ Address vẫn **SGQ CUSTOMER**. |
| 7 | Ở màn lịch giao đã lưu, đổi Incoterm sang **Self-Pickup** | Loading. Chip **Unsaved** quay lại ở 1st Schedule, nút Done **mờ**, địa chỉ reset về địa chỉ đầu tiên của Self-Pickup. |
| 8 | **Save** → **Done** | Ở Cart, dòng lịch ghi "**Self-Pickup · dd/mm/yyyy**". |

---

## TC-08. Unsaved / Discard / Reset / Thu gọn / Xóa lịch

Điều kiện: [RESET] → Add Delivery.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Chọn [Ngày X]; TIN = 10; BAKING SODA BAG = 5 | Chip "Unsaved". |
| 2 | Bấm **Reset** ở khối BAKING SODA | BAG về **0**, dòng "Available **(10)** product in order". TIN vẫn **10**. |
| 3 | Bấm "<" | Popup đỏ "**Discard Changes?**": "You have unsaved changes in the delivery schedule." xuống dòng "Leave without saving?". Nút **Discard** (đỏ) trên, **Cancel** dưới. |
| 4 | Bấm **Cancel** | Ở lại màn lịch giao, dữ liệu vừa nhập còn nguyên. |
| 5 | Bấm "<" → **Discard** | Về Cart. **Không** có lịch nào, vẫn "Add Delivery", không banner, giỏ không đổi ($1,549.44). |
| 6 | **Add Delivery** → ngày, TIN 10, CAR 5, BAG 5, BAG 10 → **Save** | Nút "✓ Saved", Done sáng, "+" sáng. |
| 7 | Sửa CAR từ 5 thành 6 | Chip "**Unsaved**" xuất hiện lại; nút đổi về "**Save**" (sáng); **Done mờ**; "+" **mờ**. |
| 8 | Sửa lại và **Save** | Trở về "✓ Saved". |
| 9 | Bấm vào tiêu đề "**1st Schedule**" | Thu gọn: chỉ còn hàng tiêu đề, bên dưới chữ "1st Schedule" có ngày nhỏ dd/mm/yyyy, mũi tên chuyển xuống. Bấm lần nữa để mở lại. |
| 10 | Bấm **thùng rác đỏ** trên tiêu đề | Popup đỏ "**Delete 1st Schedule?**": "This delivery schedule and its allocated quantities will be removed." Nút **Delete** (đỏ), **Cancel**. |
| 11 | Bấm **Delete** | Toast "**Delivery schedule deleted**". Vùng nội dung hiện "**No delivery schedule yet. Tap + to add a new schedule.**". Thùng rác **mờ**, "+" sáng, Done sáng. |
| 12 | Bấm **Done** | Cart: **không** banner, nút **Add Delivery**, Check Out mờ. Giỏ vẫn 100 items $1,549.44 (xóa lịch **không** làm giỏ tự giảm hay tăng lại). |

---

## TC-09. Nút "+" luôn sáng khi đã chia đủ, thêm 2nd Schedule vượt giỏ

Điều kiện: **[PRE-S1]**.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm link **1st Schedule** | Cả 4 dòng "Available **(0)** product in order". Nút **"+" sáng** (dù đã chia đủ). |
| 2 | Bấm **"+"** | 2nd Schedule xuất hiện và mở, 1st Schedule thu gọn. 2nd có chip "Unsaved", cả 4 dòng Available **(0)**. Nút "+" **mờ trở lại**, Done mờ. |
| 3 | Chọn [Ngày Y]; ACTIVE-GO **TIN = 3** | Available (0), note xanh "**Exceeds cart by 3 TIN · cart will be increased to 13 TIN on Save**". |
| 4 | **Save** | Toast "**2nd Schedule saved successfully. Cart quantity increased for 1 item to match the schedule**". Note xanh biến mất. "+" sáng trở lại. |
| 5 | **Done** | Cart: ACTIVE-GO TIN SL **13**, Total **$179.01** (hàng nháy vàng nhạt). BAKING SODA 5, CAR 5, WHOLEMEAL 10 giữ nguyên. Footer **33 items, $473.45**. Không banner. Có 2 link "1st Schedule", "2nd Schedule". Không có nút Add Delivery. |

---

## TC-10. Xóa sản phẩm khỏi giỏ

### 10a. Chưa có lịch giao

Điều kiện: [RESET].

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm thùng rác cạnh BAKING SODA | Popup đỏ "**Remove Item**": "Are you sure you want to remove BAKING SODA (SODIUM BICARBONATE) from your cart?" Nút **Remove** (đỏ), **Cancel**. |
| 2 | **Cancel** | Đóng popup, giỏ không đổi (vẫn 4 dòng). |
| 3 | Làm lại, bấm **Remove** | Toast "**Item removed from cart**". BAKING SODA biến mất. Badge Cart còn **3**. Footer **90 items, $1,535.27**. |

### 10b. Món đang được chia vào lịch giao

Điều kiện: **[PRE-S0]** (hoặc [PRE-S1]).

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm thùng rác cạnh BAKING SODA | Popup đỏ "**Cannot Remove Item**": "This item is currently assigned to a delivery schedule." xuống dòng "Please remove it from the delivery schedule first." Chỉ có **một** nút **Confirm** (đỏ). |
| 2 | Bấm **Confirm** | Popup đóng, **không** xóa gì, giỏ và lịch giữ nguyên. |
| 3 | Làm tương tự với ACTIVE-GO hoặc WHOLEMEAL | Cùng popup "Cannot Remove Item" (chỉ cần một UOM của món đó đang có trong lịch là bị chặn). |

### 10c. Gỡ món khỏi lịch rồi mới xóa

Điều kiện: **[PRE-S0]**.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm **1st Schedule** → bấm **Reset** ở khối BAKING SODA | BAG về 0, chip Unsaved. |
| 2 | **Save** → **Done** | Cart: BAKING SODA hiện "Unallocated: 10 BAG". |
| 3 | Bấm thùng rác cạnh BAKING SODA | Lần này hiện popup **Remove Item** bình thường (không bị chặn). |
| 4 | **Remove** | BAKING SODA biến mất, badge **3**. Banner còn, vì các món khác còn Unallocated: CAR "Unallocated: 15 CAR", TIN "Unallocated: 20 TIN", WHOLEMEAL "Unallocated: 30 BAG". |

---

## TC-11. Giảm số lượng ở Cart thấp hơn số đã lên lịch (Overallocated)

Điều kiện: **[PRE-S1]** (giỏ 5/5/10/10 và lịch cùng số).

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm nút **−** của BAKING SODA (5 → 4) | Popup đỏ "**Reduce Cart Quantity?**": "Reducing BAKING SODA (SODIUM BICARBONATE) to 4 BAG will leave 1 BAG still scheduled." xuống dòng "Please update your delivery schedule." Nút **Confirm** (đỏ) trên, **Cancel** dưới. |
| 2 | Bấm vào vùng tối ngoài popup | Popup **không** đóng. |
| 3 | Bấm **Cancel** | Đóng popup. Ô số lượng quay về **5**. Không có banner. |
| 4 | Gõ **2** vào ô số lượng BAKING SODA, bấm ra ngoài (hoặc Enter) | Popup "Reduce Cart Quantity?": "...to **2** BAG will leave **3** BAG still scheduled." |
| 5 | Bấm **Confirm** | Ô số lượng = **2**, Total **$2.60**. Dưới ô có chữ cam "**Overallocated: 3 BAG**". **Banner vàng** hiện. Footer **27 items, $424.17**. Check Out sáng. **Không** có nút Add Delivery (vì không có phần Unallocated). |
| 6 | Bấm **+** (2 → 3) | **Không** hiện popup. Nhãn đổi thành "Overallocated: 2 BAG". |
| 7 | Bấm **−** (3 → 2) | Popup "Reduce Cart Quantity?" hiện lại ("...to 2 BAG will leave 3 BAG still scheduled."). **Confirm**. |
| 8 | Bấm **Check Out** | Popup đỏ **Update Cart Quantity** có đúng 1 dòng: "• BAKING SODA (SODIUM BICARBONATE): 3 BAG **overallocated**". **Cancel** → đóng, giữ nguyên. |
| 9 | **Check Out** → **Update Cart** | Loading "Updating cart..." → toast xanh. BAKING SODA **tăng lại 5** (số trong lịch). Banner biến mất. Footer **30 items, $428.42**. |
| 10 | Lặp bước 4–5 để về trạng thái Overallocated, rồi bấm **1st Schedule** | Dưới BAKING SODA (BAG) có note **cam** "**Overallocated: 3 BAG more than the cart (2 BAG)**". Available (0). Nút "+" sáng. |
| 11 | Sửa BAKING SODA BAG trong lịch từ 5 xuống **2** | Note cam biến mất. |
| 12 | **Save** → **Done** | Toast chỉ "1st Schedule saved successfully" (không tăng giỏ). Cart: BAKING SODA vẫn **2**, **không** banner, footer $424.17. |
| 13 | Giảm nhưng **không** thấp hơn lịch (làm trên **[PRE-S0]**: giỏ BAKING SODA 10, lịch 5): gõ **7** | **Không** có popup. Hiện "**Unallocated: 2 BAG**". |

---

## TC-12. Check Out khi vừa Overallocated vừa Unallocated

Điều kiện: **[PRE-S1]**.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Giảm BAKING SODA xuống **2** → Confirm (như TC-11 bước 4–5) | "Overallocated: 3 BAG". |
| 2 | Tăng ACTIVE-GO TIN từ 10 lên **11** (bấm **+**) | Không popup. Hiện "**Unallocated: 1 TIN**". |
| 3 | Bấm **Check Out** | Popup **Update Cart Quantity** có 2 dòng theo thứ tự trên Cart:<br>• BAKING SODA (SODIUM BICARBONATE): 3 BAG overallocated<br>• ACTIVE-GO: 1 TIN not assigned |
| 4 | **Update Cart** | BAKING SODA về **5**, TIN về **10**. Banner biến mất. Footer **30 items, $428.42**. |

---

## TC-13. Category → Results (danh sách sản phẩm), tìm kiếm, lọc, thêm vào giỏ

Điều kiện: [RESET], tab Cart.

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Bấm tab **Category** | Màn **Results**: nút "<", tiêu đề "Results". Hàng dưới có ô tìm kiếm nền xanh nhạt (icon kính lúp, chữ mờ "**Search Product Name or SKU #**") và nút **Filter** (icon phễu, khung trắng). Tab Category đang active. |
| 2 | Xem danh sách | 6 thẻ theo thứ tự:<br>1. **SUNFLOWER OIL** · Brand: GOLDEN SUN · Halal · Pack Size: IN 4 X 5L BOTTLE PER CAR · "BTL: $11.50 \| CAR: $44.00"<br>2. **WHOLEMEAL FLOUR - HCS** · Brand: SW WHOLEMEAL · Halal · góc khuyến mãi · Pack Size: IN 25KG PAPER BAG · "BAG: $18.00 ~~$20.00~~" · dòng xanh "**In cart: 40 BAG**"<br>3. **ACTIVE-GO** · Brand: MILO · Pack Size: IN 6 X 1.8KG TIN PER CAR · "CAR: $13.77 \| TIN: $13.77" · "In cart: 20 CAR, 30 TIN"<br>4. **CASTER SUGAR** · Brand: CRYSTAL FINE · Halal · góc khuyến mãi · Pack Size: IN 25KG PAPER BAG · "BAG: $32.00 ~~$35.00~~"<br>5. **BAKING SODA (SODIUM BICARBONATE)** · Brand: No Brand · Pack Size: IN 20 X 1KG BAG PER CAR · "BAG: $1.30" · "In cart: 10 BAG"<br>6. **BUTTER BLEND SPREAD** · Brand: CREAMY GOLD · Pack Size: IN 36 X 250G TUB PER CAR · "TUB: $12.50"<br>Mỗi thẻ có nút **"+"** vuông xanh đậm ở góc phải trên. |
| 3 | Gõ `sugar` vào ô tìm | Chỉ còn **CASTER SUGAR**. |
| 4 | Gõ `milo` (tìm theo brand) | Chỉ còn **ACTIVE-GO**. |
| 5 | Gõ `sku-oil` (tìm theo SKU, SKU không hiện trên thẻ) | Chỉ còn **SUNFLOWER OIL**. |
| 6 | Gõ `zzz` | Hiện chữ giữa màn "**No products found**". |
| 7 | Xóa hết chữ | Quay lại đủ 6 sản phẩm. |
| 8 | Bấm nút **Filter** | Bottom sheet "**Filter**": **All Products** (có ✓), **Halal**, **Promotion**. |
| 9 | Chọn **Halal** | Còn 3 thẻ: SUNFLOWER OIL, WHOLEMEAL FLOUR - HCS, CASTER SUGAR. Nút Filter có **chấm đỏ** nhỏ. |
| 10 | Mở Filter, chọn **Promotion** | Còn 2 thẻ: WHOLEMEAL FLOUR - HCS, CASTER SUGAR. |
| 11 | Mở Filter, chọn **All Products** | Đủ 6 thẻ, chấm đỏ biến mất. |
| 12 | Bấm **"+"** trên SUNFLOWER OIL | Bottom sheet nổi từ dưới: ảnh, "SUNFLOWER OIL", "Brand: GOLDEN SUN", "Pack Size: IN 4 X 5L BOTTLE PER CAR".<br>Hai dòng: "**UOM (BTL)** $11.50" stepper = **1**; "**UOM (CAR)** $44.00" stepper = **0**. Nút **Add to Cart** (xanh đậm). |
| 13 | Bấm vùng tối bên ngoài sheet | Sheet đóng, không thêm gì (badge vẫn 4). |
| 14 | Mở lại, giữ BTL = 1, bấm **Add to Cart** | Sheet đóng. Toast xanh "**Added 1 BTL SUNFLOWER OIL to cart**". Thẻ SUNFLOWER OIL hiện dòng xanh "**In cart: 1 BTL**". Badge Cart thành **5** (nhảy nhẹ). |
| 15 | Mở sheet SUNFLOWER OIL lần nữa | Dòng BTL có thêm chữ xanh nhỏ "**In cart: 1 BTL**". Đặt cả BTL và CAR về **0** → nút **Add to Cart** **mờ**. |
| 16 | Bấm "+" trên **CASTER SUGAR**, đặt BAG = **2**, **Add to Cart** | Toast "Added 2 BAG CASTER SUGAR to cart". Badge **6**. |
| 17 | Bấm "+" trên **BAKING SODA**, nhập BAG = **5**, **Add to Cart** | Cộng dồn: thẻ hiện "In cart: 15 BAG". Badge vẫn **6** (không thêm dòng mới). |
| 18 | Bấm "<" (hoặc tab Cart) | Về Cart. Thứ tự sản phẩm: BAKING SODA (**15**), ACTIVE-GO, WHOLEMEAL, rồi **SUNFLOWER OIL** (UOM (BTL) $11.50, SL 1, Total $11.50), rồi **CASTER SUGAR** (UOM (BAG) $32.00 giá gạch $35.00, SL 2, Total $64.00, dòng "Bulk Sugar Discount (Caster): -$6.00").<br>Sản phẩm mới có đủ thùng rác, ảnh, Pack Size, stepper như sản phẩm cũ. |

Số liệu footer để đối chiếu: chỉ thêm SUNFLOWER OIL 1 BTL vào giỏ gốc → **101 items, $1,561.97**. Thêm SUNFLOWER OIL 1 BTL + CASTER SUGAR 2 BAG (chưa tăng BAKING SODA) → **103 items, $1,631.73**. Sau bước 17 (BAKING SODA lên 15) → **108 items, $1,638.82**.

---

## TC-14. Thêm sản phẩm mới khi đã có lịch giao

Điều kiện: **[PRE-S1]** (30 items, $428.42, đã chia đủ, không có nút Add Delivery).

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Tab **Category** → "+" trên **BUTTER BLEND SPREAD** → nhập TUB = **6** → **Add to Cart** | Toast "Added 6 TUB BUTTER BLEND SPREAD to cart". |
| 2 | Về Cart (tab Cart) | **Banner vàng** hiện lại. Cuối danh sách có BUTTER BLEND SPREAD: UOM (TUB) $12.50, SL 6, Total **$75.00**, chữ cam "**Unallocated: 6 TUB**". Nút "**Add Delivery (2nd Schedule)**" **hiện lại**. Footer **36 items, $510.17**. Badge **5**. |
| 3 | **Check Out** | Popup Update Cart Quantity có 1 dòng: "• BUTTER BLEND SPREAD: 6 TUB not assigned". Bấm **Cancel**. |
| 4 | Bấm **1st Schedule** | Khối **BUTTER BLEND SPREAD** nằm **cuối** (sau WHOLEMEAL), UOM (TUB) = 0, "Available **(6)** product in order". |
| 5 | Nhập TUB = **6** | "Available (0)". Không có note xanh (vừa đủ giỏ). |
| 6 | **Save** → **Done** | Toast "1st Schedule saved successfully". Cart **không** banner, **không** nút Add Delivery. |
| 7 | **Check Out** | Order Confirmation: "Order Items (36 items)", lịch 1st có thêm dòng "BUTTER BLEND SPREAD: 6 TUB", GST (9%) **$42.12**, Total **$510.17**. |
| 8 | Nhánh khác: làm lại bước 1–2, ở bước 3 chọn **Check Out → Update Cart** | Giỏ **bỏ** BUTTER BLEND SPREAD (vì chưa lên lịch). Footer về **30 items, $428.42**, badge **4**. |

---

## TC-15. Lưu dữ liệu, reset, đóng popup

| # | Thao tác | Kết quả mong đợi trên UI |
|---|---|---|
| 1 | Làm **[PRE-S0]**, rồi nhấn **F5** | Giỏ, lịch giao, banner, các dòng Unallocated **giữ nguyên** (dữ liệu lưu trong trình duyệt). |
| 2 | Bấm tab **My Profile** | Bottom sheet "**Demo controls**" với 1 mục: "Đặt lại toàn bộ dữ liệu demo" và dòng nhỏ "Giỏ hàng ban đầu, xoá lịch giao, Order ID về 2677". |
| 3 | Bấm mục đó | Về Cart gốc ($1,549.44, 100 items, "Add Delivery"). Toast "**Đã đặt lại dữ liệu demo**". |
| 4 | Mở bottom sheet/lịch/popup rồi nhấn phím **Esc** hoặc bấm vùng tối | Sheet, lịch và popup thường **đóng**. Ngoại lệ **không** đóng được: popup "Reduce Cart Quantity?" và popup "Order Submitted". |
| 5 | Gõ số lượng ở Cart: `0` hoặc để trống rồi bấm ra ngoài | Toast "**Quantity must be at least 1**", ô quay về số cũ. |
| 6 | Gõ `99999` ở ô số lượng BAKING SODA (Cart) | Ô thành **9999**, footer tăng tương ứng (đối chiếu: 10089 items, $15,703.85 trên giỏ gốc). Đặt lại về 10 thì về $1,549.44. |
| 7 | Bấm nút **−** khi số lượng = 1 | Nút **−** mờ, không giảm thêm. |

---

## Phụ lục: bảng số liệu tham chiếu

| Trạng thái giỏ | Items | Trước giảm giá | Giảm giá | Subtotal | GST 9% | **Total** |
|---|---|---|---|---|---|---|
| Gốc (BS 10, CAR 20, TIN 30, WF 40) | 100 | $1,501.50 | -$80.00 | $1,421.50 | $127.94 | **$1,549.44** |
| Sau luồng 1 (BS 5, CAR 5, TIN 10, WF 10) | 30 | $413.05 | -$20.00 | $393.05 | $35.37 | **$428.42** |
| Sau luồng 2 (BS 20, CAR 20, TIN 40, WF 40) | 120 | $1,652.20 | -$80.00 | $1,572.20 | $141.50 | **$1,713.70** |
| Luồng 1 + BS giảm còn 2 | 27 | $409.15 | -$20.00 | $389.15 | $35.02 | **$424.17** |
| Luồng 1 + TIN 13 | 33 | $454.36 | -$20.00 | $434.36 | $39.09 | **$473.45** |
| Luồng 1 + BUTTER BLEND SPREAD 6 TUB | 36 | $488.05 | -$20.00 | $468.05 | $42.12 | **$510.17** |
| Gốc − BAKING SODA (đã xóa) | 90 | | | | | **$1,535.27** |
| Gốc + SUNFLOWER OIL 1 BTL | 101 | | | | | **$1,561.97** |
| Gốc + SUNFLOWER OIL 1 BTL + CASTER SUGAR 2 BAG | 103 | | | | | **$1,631.73** |

Ghi chú:
- Badge trên tab Cart đếm **số dòng UOM** trong giỏ (giỏ gốc = 4), không đếm tổng số lượng.
- Chữ "Unallocated" (giỏ nhiều hơn lịch) và "Overallocated" (lịch nhiều hơn giỏ) đều màu cam đậm.
- Khi Save lịch, giỏ chỉ **tự tăng** cho UOM bạn vừa nhập tăng trong lịch đó. Dòng đang Overallocated do bạn giảm giỏ **không** bị tăng ngược lại khi Save lịch khác.
- Order ID: đơn đầu 2677, sau mỗi lần Submit tăng 1. [RESET] đưa về 2677.
