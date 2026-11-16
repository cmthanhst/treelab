# TREE LAB — Đặc tả dự án và hướng dẫn bảo trì cho agent

**Cập nhật:** 16/11/2026.  
**Đơn vị phát triển:** Công ty CP Đầu Tư Việt Tín.  
**Phạm vi đối chiếu:** `TREE_LAB.html` và `tests/tree-lab.test.cjs` trong cùng thư mục.  
**Mục đích:** giúp agent tiếp quản hiểu hành vi hiện tại, giữ đúng yêu cầu đã thống nhất và biết sửa/kiểm tra ở đâu.

Đây là đặc tả của **bản hiện tại**, không phải danh sách tính năng cần xây dựng lại. Các mục “giới hạn” và “hướng phát triển” chưa được coi là yêu cầu triển khai. Nếu tài liệu và mã khác nhau, kiểm tra cả yêu cầu mới của người dùng và hành vi thực tế, sau đó cập nhật tài liệu theo thay đổi đã làm.

## 1. Đọc nhanh khi tiếp quản

1. Đọc tài liệu này để hiểu quy tắc nghiệp vụ và dữ liệu.
2. Đọc phần liên quan trong `TREE_LAB.html`; không viết lại toàn bộ chỉ để chỉnh một chức năng nhỏ.
3. Đọc `tests/tree-lab.test.cjs` trước khi đổi tên hàm, tách module hoặc sửa định dạng bài tập.
4. Xem `HUONG_DAN_TREE_LAB.md` khi cần hướng dẫn sử dụng và lịch sử thay đổi. Tài liệu đó có cả kết quả của các bản cũ; **không coi tất cả mô tả cũ là bố cục hiện hành**.
5. Sau sửa: kiểm tra phù hợp với phần thay đổi, cập nhật tài liệu và nói rõ phần đã/chưa kiểm tra thực tế.

Các nguyên tắc quan trọng nhất:

- Cây là hình học **3D WebGL thật**; không thay bằng canvas 2D giả phối cảnh.
- Cắt phải giữ đúng phần gốc và loại đúng phần phía ngoài, bao gồm hậu duệ phụ thuộc.
- Cây gốc phải còn nguyên để so sánh, hoàn tác và lưu/giao bài.
- File bài nộp phải khôi phục đúng cây đã giao, không tạo cây khác từ seed.
- Không khôi phục bảng thực hành ở cạnh/dưới viewport: người dùng đã yêu cầu bỏ.
- Không tự chấm đúng/sai kỹ thuật lâm sinh khi chưa có quy trình được người dùng duyệt.

## 2. Mục tiêu, người dùng và phạm vi

Ứng dụng tiếng Việt để đào tạo thao tác cắt tỉa cây xanh đô thị. Người dùng quan sát cây từ mọi phía, chọn vị trí trên phần gỗ, xem hậu quả hình học của vết cắt và đối chiếu trước/sau.

| Người dùng | Công việc |
|---|---|
| Người giao bài | Tạo cây, nhập mã/tên bài và yêu cầu, xuất cây gốc |
| Công nhân | Mở bài, nhập tên, thực hành cắt, lưu và gửi file bài nộp |
| Người đánh giá | Mở bài nộp, xem trước/sau, xem từng thao tác, ghi nhận xét và lưu lại |

Ứng dụng không có cơ chế đăng nhập/phân quyền; các vai trò trên là luồng sử dụng, không phải quyền được hệ thống cưỡng chế.

Không có backend, tài khoản hoặc cơ sở dữ liệu. Không tự gửi file cho ai. Người dùng chuyển file qua kênh họ đang dùng.

Không thuộc phạm vi hiện tại: xác định loài chính xác, sức chịu tải/cơ học, cây đổ, rách vỏ, lành vết thương, bệnh hại, sinh trưởng, xác thực công nhân, chống sửa bài, tự động cho điểm.

## 3. Tệp và cách chạy

| Tệp | Vai trò |
|---|---|
| `TREE_LAB.html` | Toàn bộ HTML, CSS, JavaScript, dựng cây và xử lý bài tập |
| `DAC_TA_DU_AN_TREE_LAB.md` | Đặc tả tổng hợp cho người/agent bảo trì |
| `HUONG_DAN_TREE_LAB.md` | Hướng dẫn sử dụng, lịch sử cập nhật và phạm vi kiểm tra từng bản |
| `tests/tree-lab.test.cjs` | Kiểm tra tích hợp bằng Node, DOM giả lập và Three.js thật |

Các thư mục khác trong workspace không phải phụ thuộc chạy của TREE LAB. Bản mô tả ban đầu từng được cung cấp ngoài workspace cũng không phải tài nguyên cần tải khi chạy.

Ứng dụng là một HTML độc lập, dùng import map tải **Three.js 0.170.0** và **OrbitControls cùng phiên bản** từ jsDelivr. Cần Internet nếu chưa đóng gói thư viện cục bộ. Không trộn các phiên bản Three.js/addons.

- Máy tính: mở HTML bằng trình duyệt hỗ trợ module/import map và WebGL.
- Điện thoại: mở bằng trình duyệt có thể chạy HTML/JavaScript; trình xem trước file có thể không chạy được. Có thể dùng một bản phục vụ web tĩnh khi môi trường cho phép.
- Có thông báo tải, lỗi CDN, lỗi WebGL và mất context. Bootstrap script thường nằm ngoài module để vẫn báo lỗi nếu module không tải được; timeout tải là 25 giây.

## 4. Những luồng phải giữ đúng

### 4.1. Thực hành cắt

1. Ở chế độ **Sau cắt**, chọn điểm cắt luôn sẵn sàng; không có công tắc bật/tắt.
2. Nhấp/chạm nhanh trên phần gỗ để raycast. Kéo vẫn dùng để xoay.
3. Hiện vòng cam và thông tin nhánh, đường kính, vị trí, cao độ. Nút kính lúp trong thẻ điểm chọn đưa camera tới vị trí này.
4. Nhấn icon kéo cắt → hộp xác nhận → Thực hiện.
5. Cập nhật hình học, bộ đếm, lịch sử; xóa điểm chọn.

Hủy chọn chỉ xóa lựa chọn/marker, không thay cây. Không cắt khi chỉ nhấp vào cây. Không bỏ bước xác nhận.

Nhận diện click hiện dùng ngưỡng di chuyển 7 px, thời gian dưới 650 ms và không có đa chạm. Pointer cancel phải hủy gesture. Chạm hai ngón/di chuyển camera không được bị hiểu thành vết cắt.

### 4.2. Hoàn tác, làm lại và đặt lại

- Undo giảm `cursor`, redo tăng `cursor`, rồi tính và dựng lại trạng thái.
- Cắt mới sau undo phải xóa phần redo cũ bằng `history.slice(0, cursor)` trước khi thêm lệnh.
- Reset có xác nhận; xóa toàn bộ lịch sử và khôi phục cây gốc, giữ hình dáng/cấu hình đang dùng.
- Áp dụng thông số/tạo ngẫu nhiên dựng cây gốc mới và mã bài mới. Nếu đang có lịch sử, hỏi xác nhận trước khi thay cây/xóa lịch sử.

### 4.3. Giao bài → làm bài → đánh giá

- **Lưu cây gốc để giao bài:** xuất đủ hình học gốc và yêu cầu; `history=[]`, `cursor=0`, tên công nhân/nhận xét trống. Không sửa kết quả đang xem.
- **Lưu cây hiện tại / bài nộp:** xuất hình học gốc, toàn bộ lịch sử, cursor hiện tại, metadata và tùy chọn hiển thị.
- Công nhân nhận HTML cùng file JSON giao bài; nếu đã có cùng bản HTML thì chỉ cần JSON.
- Người quản lý mở JSON bài nộp bằng icon thư mục trong ứng dụng, không phải sửa JSON bằng tay.
- Nhận xét là văn bản do người đánh giá nhập. Không có tự động chấm điểm.
- Để góp ý tại một vị trí, mở **Lịch sử**, chọn lần cắt, nhập nhận xét, chọn **Về bài làm** rồi lưu bài nộp. Xem lại giữ nguyên `cursor`. Nếu dùng nút Hoàn tác hoặc Làm lại, file lưu sẽ phản ánh số lệnh đang áp dụng.

## 5. Thông số cây và ngẫu nhiên

Nguồn định nghĩa duy nhất là mảng `fields`: `[key, label, min, max, step, defaultValue, unit]`. Mảng này sinh giao diện, đọc cấu hình, kiểm tra file và khôi phục giá trị.

| Key | Ý nghĩa | Min–max | Bước | Mặc định |
|---|---|---|---|---|
| `treeHeight` | Chiều cao cây | 6–25 m | 0,5 | 10 |
| `diameter` | Đường kính gốc | 0,25–1,2 m | 0,05 | 0,55 |
| `branchHeight` | Chiều cao bắt đầu phân cành | 1–6 m | 0,25 | 3 |
| `mainCount` | Cành chính **mỗi tầng** | 2–7 | 1 | 4 |
| `levels` | Số tầng cành | 1–4 | 1 | 3 |
| `secondary` | Cành phụ **mỗi cành chính** | 1–6 | 1 | 3 |
| `spread` | Bán kính xòe tán mục tiêu | 2–6 m | 0,25 | 3,5 |
| `density` | Độ rậm lá | 0–100% | 5 | 40 |
| `trunkBend` | Độ uốn thân | 0–100% | 5 | 45 |
| `trunkRandom` | Ngẫu nhiên thân | 0–100% | 5 | 45 |
| `branchBend` | Độ uốn cành chính/phụ | 0–100% | 5 | 70 |
| `branchRandom` | Ngẫu nhiên cành chính/phụ | 0–100% | 5 | 55 |
| `droopRatio` | Tỷ lệ cành chính mọc võng | 0–100% | 5 | 40 |
| `droopDepth` | Mức võng xuống | 0–100% | 5 | 60 |

Chiều cao phân cành hiệu dụng được giới hạn bởi `min(branchHeight, treeHeight * 0.67)`; giá trị cấu hình gốc không bị sửa. Các kích thước là tham số mô hình, không phải đặc tả thực vật chính xác. Thân đạt cao độ `treeHeight`; tán sinh theo tham số, không có phép đo bao hình học để cưỡng chế chiều cao toàn tán bằng đúng nhãn này.

Mỗi thông số có checkbox `keep-${key}`, mặc định bỏ chọn:

- **Áp dụng thông số** lấy giá trị hiện tại của thanh trượt, không ngẫu nhiên hóa.
- **Xúc xắc** gọi `randomizeFields()`: chỉ thay các giá trị chưa đánh dấu Giữ, theo đúng min/max/step; sau đó tạo seed mới và dựng cây.
- Giá trị giữ là giá trị **đang chọn trên form**, kể cả chưa áp dụng, không phải giá trị cũ trong `config`.
- Giữ tất cả: giữ các tham số, vẫn đổi seed/hình dáng. Bỏ giữ tất cả: cả 14 tham số có thể đổi. Giá trị ngẫu nhiên có thể tình cờ bằng giá trị trước.
- Checkbox không khóa chỉnh tay. Không được vô hiệu thanh trượt chỉ vì đã chọn Giữ.

Hiện lá và độ hiện lá là tùy chọn hiển thị riêng: opacity 0–70%, mặc định 24%, cập nhật ngay không cần dựng lại cây. Chúng không thuộc 14 thông số được xúc xắc ngẫu nhiên hóa.

## 6. Kiến trúc mô hình 3D

### 6.1. Trạng thái chính

| Biến | Ý nghĩa |
|---|---|
| `nodes`, `nodeMap` | Mảng cây theo thứ tự cha trước con, và tra cứu theo ID |
| `config`, `seed` | Thông số cây đã áp dụng và hạt dựng hình |
| `history`, `cursor` | Tất cả lệnh còn giữ và số lệnh đang áp dụng |
| `selected` | `null` hoặc `{id, t}` đang chờ xác nhận |
| `treeGroup`, `woodMeshes`, `leafMesh` | Hình học kết quả; danh sách phần gỗ dùng raycast |
| `beforeGroup` | Bản dựng toàn bộ cây gốc, tạo khi cần xem trước/so sánh |
| `viewMode` | `after`, `before`, `split` |
| `exerciseId` | Mã bài được tạo mới khi dựng cây mới, giữ nguyên khi tải/nộp bài |
| `saveDirectory`, `saveInProgress` | Thư mục lưu trong phiên và khóa thao tác lưu đồng thời |

Một node runtime có: `id`, `parentId`, `attachT`, `start`, `end`, `points`, `radiusStart`, `radiusEnd`, `children`, `meshes`, `leaves`, `limit`, `visible`.

- `T0` là thân gốc; các cành tạo mới có mã `C001`, `C002`…
- `points` là mảng `THREE.Vector3` theo thứ tự từ gốc nhánh đến đầu nhánh.
- `attachT` là tham số vị trí mọc trên **trục cây gốc của nhánh cha**.
- `limit` là phần trục còn giữ, ban đầu 1. Không cắt ngắn `points` gốc sau mỗi lệnh.
- `children` chứa ID, không phải mesh. Khi tải file phải tái lập quan hệ đúng ngay cả khi ID không liên tiếp.

### 6.2. Dựng hình

- Đơn vị thế giới: mét; trục Y hướng lên. Lưới mặt đất mỗi ô 1 m.
- Thân: 33 điểm/32 đoạn; đường uốn có đổi hướng, không chỉ nghiêng nhẹ.
- Cành: Bézier bậc ba 3D, lấy điểm theo khoảng cách dọc đường cong; 24 đoạn/cành chính và 16 đoạn/cành phụ.
- Cành võng có hướng mọc ban đầu chúc xuống, qua điểm thấp rồi cong lên đầu. Không thay bằng đường luôn đi lên.
- Điều khiển võng tắt nếu `branchBend=0` hoặc `droopDepth=0`. Các điểm điều khiển cành được chặn cao độ tối thiểu 0,22 m; đây không phải phát hiện va chạm toàn bộ thể tích gỗ với đất.
- Tỷ lệ võng trên cành chính được phân bố, làm tròn theo số nhánh hữu hạn. Cành phụ có xác suất võng bằng `droopRatio / 100 * 0.65`.
- Số node được tạo: `1 + levels * mainCount * (1 + secondary)`. Mặc định 49; cấu hình tối đa hiện tại 197.
- Phần gỗ dùng các đoạn `CylinderGeometry` thuôn, hướng theo trục; mặt cắt dùng đĩa màu gỗ sáng.
- Lá là phiến dài/hẹp, có độ cong và sống giữa nhẹ, dùng `InstancedMesh`; không quay lại các khối cầu/đa diện đặc che cành. Vật liệu hai mặt, bán trong suốt, `depthWrite=false`.
- Hai luồng RNG riêng cho gỗ và lá. Thay mật độ lá không được làm đổi hình dáng phần gỗ với cùng seed/tham số gỗ.

`sample(node,t)` nội suy theo chỉ số các đoạn trong `points`, `t` trong [0,1]. Cành Bézier được lấy mẫu gần đều theo chiều dài, còn thân được lấy mẫu theo cao độ. Vì vậy `t` **không phải tỷ lệ chiều dài cung chính xác cho mọi node**. Khoảng cách trong thông tin chọn hiện là xấp xỉ; khi sửa phần đo đạc phải chú ý giới hạn này.

### 6.3. Người tham chiếu kích thước và lá ngọn

- Có mô hình người 3D đứng cạnh gốc, cao **1,7 m** tính từ chân y=0 đến đỉnh đầu y=1,7 theo cùng đơn vị mét của cây. Tạo một lần bằng `createScalePerson()`, có quần áo, đầu, tay, chân và vạch thước 0–1,7 m.
- `humanReference` nằm ngoài `treeGroup`/`beforeGroup`, có `userData.scaleReference=true`; không phải bộ phận cây, không được tính vào số cành, không chọn để cắt và không biến mất khi cắt thân.
- `positionHumanReference()` đặt người cạnh gốc theo đường kính thân khi dựng/tải cây, giữ nguyên tỷ lệ người. Không phóng đại người theo chiều cao cây.
- Hai DOM label `human-label` và `human-label-right` chỉ ghi “1,7m”, nằm dọc bên thước đứng; `updateHumanLabel()` chiếu hai đầu thước ra màn hình, đặt chữ ở giữa và xoay theo đoạn chiếu. Nhãn tạm ẩn khi nhìn gần trùng phương thước (đoạn chiếu dưới 6 px). Split phải chiếu đúng offset/chiều rộng từng nửa; chế độ thường chỉ dùng một label. Label không chặn thao tác chuột/chạm.
- Người/thước là vật tham chiếu được tạo lại bởi ứng dụng, không serialize vào danh sách node của bài tập.
- Người tham chiếu dùng một lưới 1.880 tam giác với màu theo đỉnh. Các vòng tiết diện tạo dáng người đứng dồn trọng tâm sang một chân, tay thả lệch, mặc áo phản quang và giày bảo hộ. Lưới được tạo trong HTML và dùng chung một vật liệu.
- Chỉ nhóm giải phẫu bên trong người được chuẩn hóa bounding box về chiều cao 1,7 m và chân y=0 trước khi thêm thước. Khi sửa hình dáng phải giữ bước chuẩn hóa này; kiểm tra chiều normals của bề mặt tạo bằng các vòng, tránh mặt quay vào trong.
- Cây mới có lá trên phần thân dẫn ngọn: gắn vào `T0.leaves` từ t=0,78 trở lên; một lá gần đầu t=0,998 để không luôn trơ đọt. Số lá ngọn là `max(6,round(density*0.6))` nếu density>0; density=0 vẫn hoàn toàn không lá.
- Lá ngọn dùng cùng vật liệu/InstancedMesh, opacity và tùy chọn hiện lá; lọc theo limit thân như các lá khác. Không tạo một cụm lá trang trí độc lập còn treo sau khi cắt ngọn.
- Tạo lá ngọn sau khi tạo các nhánh, dùng RNG lá, không thay RNG gỗ hoặc số nhánh. Mặc định 49 bộ phận gỗ và 432 phiến lá (408 ở cành + 24 ở ngọn).
- Khi tải file cũ không có lá ngọn, giữ đúng tọa độ/lá trong file; không tự thêm lá làm thay đổi bài đã giao. Muốn cây có lá ngọn theo thuật toán mới phải tạo cây mới.

## 7. Quy tắc cắt và lịch sử — bất biến quan trọng

Raycast chỉ xét phần gỗ kết quả. `mesh.userData.nodeId` ánh xạ về node; `projectPoint()` tìm điểm gần nhất trên đường trục để suy ra `t`. Trước raycast phải cập nhật world matrices để nhánh vừa dựng lại có thể được chọn ngay.

Một lệnh lưu `{id, t, note?}` theo **nhánh gốc**, không theo chiều dài đoạn còn lại tại thời điểm cắt.

`remainingAt(end)`:

1. Tạo bảng giới hạn độc lập với giá trị ban đầu bằng 1 cho mỗi node.
2. Áp dụng các lệnh trước chỉ số `end`, lấy `limit=min(limit,t)` cho node được cắt.
3. Duyệt cha trước con; cành con còn nếu cha còn hiện, cha còn đoạn gỗ và `attachT < parent.limit - 1e-7`.
4. `rebuild()` chỉ dựng từ gốc nhánh tới `limit`, giữ lại một phần của đoạn cuối nếu điểm cắt nằm giữa đoạn.

**Không được dùng cao độ Y để quyết định phần bị bỏ.** Cành có thể mọc xuống rồi lên; “phía ngoài” được xác định theo thứ tự dọc trục và quan hệ cha–con.

Cắt giữa cành cha không mặc định xóa mọi con: con mọc trên phần gốc còn giữ phải sống. Con mọc tại/sau mặt cắt mất cùng toàn bộ hậu duệ. Cắt thân áp dụng cùng quy tắc, không xóa ngẫu nhiên các tầng còn ở dưới vết cắt.

Cây gốc và seed không thay đổi vì cut/undo/redo/reset/so sánh. Dựng cây mới hoặc tải file cây khác mới thay nguồn hình học gốc đang dùng.

### 7.1. Xem lại và nhận xét từng nhát cắt

Chọn mục trong Lịch sử để dựng cây tại bước i bằng drawState với hai bảng remainingAt(i) và remainingAt(i+1). Phần chênh lệch được tô đỏ, kể cả nhánh con và lá. Khung xem lại có camera tập trung tại điểm cắt, giữ nguyên cursor và kết quả bài làm. Nút kính lúp trong thẻ điểm chọn dùng chung focusCut.

Trường note tùy chọn trong mỗi lệnh chứa tối đa 1.000 ký tự văn bản thuần. Nhận xét đi theo lệnh qua hoàn tác, làm lại và lưu bài nộp. Khi cắt mới sau hoàn tác, các lệnh bị thay thế mất cả nhận xét. File giao bài gốc có lịch sử trống.

## 8. Giao diện hiện hành và responsive

### 8.1. Cấu trúc màn hình

Ngay dưới tên TREE LAB / 3D, phần đầu trang hiển thị nguyên văn **“Sản phẩm của Công ty CP Đầu Tư Việt Tín”**. Phần tử `.brand .product-credit` giữ dòng này hiển thị trên máy tính, điện thoại dọc và điện thoại ngang. Trong phần `head` của HTML, `meta[name="author"]` chứa tên “Công ty CP Đầu Tư Việt Tín”.

```text
header: TREE LAB / 3D và dòng ghi tên công ty | Bài tập | Lịch sử | Hướng dẫn
main.layout
└─ section.stage
   ├─ .scene-area
   │  ├─ #viewport / canvas WebGL
   │  ├─ .view-tools: chính diện, trên xuống, góc đầu, xúc xắc, mở, lưu
   │  ├─ .compare-tools + nhãn trước/sau
   │  ├─ .viewport-bottom
   │  │  ├─ #selection (chỉ hiện khi chọn)
   │  │  ├─ .cut-actions: cắt, hủy, undo, redo, reset
   │  │  └─ .tree-dock: #tree-info rồi #settings
   │  └─ #loading
   └─ .status-bar: ba bộ đếm + #status
dialogs ngoài main: bài tập, lịch sử, hướng dẫn, xác nhận
```

Desktop dùng `display:contents` cho `.viewport-bottom` để các con định vị riêng. Điện thoại dùng cột flex thực tế, thứ tự selection → hàng thao tác → hai mục thông số. Không thay đổi thứ tự này khi chỉnh CSS.

### 8.2. Yêu cầu đã thống nhất

- Không có `aside`, bảng thực hành, `panel-toggle` hoặc `cut-mode`.
- Ba icon góc nhìn hiện dùng: cây nhìn mặt trước trong khung; tán tròn nhìn từ trên có mũi tên xuống; ngôi nhà kèm mũi tên trở về góc đầu. Không dùng lại ba ký hiệu quá giống nhau khó nhận biết.
- Năm nút thao tác và sáu nút góc nhìn/tạo cây/file chỉ có icon; giữ `title`, `aria-label`, bàn phím và vùng chạm 44 × 44 px.
- Bộ đếm nhỏ, nằm chung status bar **bên dưới viewport**, không phải các ô lớn phủ cây. Bộ đếm phản ánh **kết quả sau cắt**, kể cả đang xem cây trước cắt.
- Thông tin bài/người làm/yêu cầu/nhận xét ở cửa sổ Bài tập; lưu cây gốc giao bài và đổi thư mục lưu cũng ở đó.
- Tùy chọn hiện lá/opacity nằm đầu `#settings`.
- Màn hình rộng: hai mục thông số ở góc trái, các icon thao tác dưới phải.
- Điện thoại: hai mục thông số nằm **dưới** hàng icon thao tác; khi thu gọn cao khoảng 32 px/mục, bề ngang chỉ vừa nội dung. Khi mở mới có chiều rộng đủ để chỉnh và giới hạn cuộn.
- Trên điện thoại chỉ mở một trong `tree-info`/`settings`; đóng phần kia không làm mất giá trị đã nhập.
- Khi đang chọn điểm cắt, `.has-selection` tạm ẩn dock trên màn hình nhỏ để hiện thông tin chọn, không xóa form.
- Breakpoint chính: chiều rộng 760 px. Bố cục màn hình thấp nằm ngang: `max-height:500px` và `min-aspect-ratio:13/10`. Có đệm safe area và chiều cao `dvh`.

### 8.3. Trạng thái và xác nhận

| Trạng thái | Hành vi |
|---|---|
| Không có điểm chọn | Cắt/hủy mờ, cây vẫn xoay/zoom được |
| Có điểm chọn | Vòng cam, thông tin chọn; cắt mở hộp xác nhận |
| `before` / `split` | Không raycast để cắt; undo/redo bị khóa; lịch sử không đổi |
| Reset ở bất kỳ chế độ xem | Vẫn có thể dùng nếu có lịch sử; phải xác nhận |
| Tạo cây mới/thay file | Xác nhận theo luồng hiện tại trước khi thay dữ liệu |
| Đang lưu | Khóa hai nút lưu và đổi thư mục, chống nhấn lưu đồng thời |
| File lỗi | Báo ở status bar và thông tin file; giữ cây hiện tại |
| Tải/CDN/WebGL lỗi | Thông báo thân thiện, không để trang trắng |

`before`/`split` là chế độ chỉ xem đối với thao tác cắt; không phải cơ chế khóa mọi thao tác thay cây. Reset/tạo cây/tải bài vẫn theo xác nhận riêng.

## 9. Camera và so sánh

OrbitControls hỗ trợ kéo xoay, con lăn/pinch zoom, chuột phải/Shift kéo và hai ngón pan. Giới hạn góc xuống đất, khoảng cách và target; không có phát hiện va chạm chính xác với từng cành/lá.

- `after`: chỉ cây kết quả, được chọn điểm cắt.
- `before`: cây gốc đầy đủ, không thay `history`, `cursor` hoặc `limit` của kết quả.
- `split`: cùng scene/camera, render hai vùng bằng viewport/scissor; trái là gốc, phải là kết quả. Khôi phục viewport/scissor khi kết thúc mỗi frame.
- `beforeGroup` được cache; phải bỏ cache khi thay cây/tải cây khác. Không làm nó phụ thuộc vào `limit` hiện tại.
- Opacity và hiện/ẩn lá phải đồng bộ cho cả hai cây.
- Vào/ra split có điều chỉnh khoảng cách theo hệ số 1,8. `fitCamera()` cũng xét chế độ split.
- ResizeObserver theo dõi viewport thật, không dùng kích thước toàn cửa sổ thay cho vùng canvas. `resize()` điều chỉnh tỷ lệ và khoảng cách khi màn hình chuyển rộng/hẹp; bỏ qua kích thước bằng 0.

## 10. Định dạng file bài tập/bài nộp

Phần mở rộng `.json`, `format="tree-lab-exercise"`, `version=1`. Lưu **tọa độ cây gốc đầy đủ**, không serialize mesh/geometry/material của Three.js.

Mô tả kiểu dưới đây dùng để đọc cấu trúc, không phải file mẫu có thể tải trực tiếp:

```ts
type Vec3 = [number, number, number];
type Cut = { id: string; t: number; note?: string };
type Leaf = {
  t: number;
  pos: Vec3;
  scale: Vec3;
  rotation: Vec3; // Euler XYZ, radian
  color: Vec3;    // Các thành phần Color r/g/b hiện tại, không đổi hệ màu khi nạp
};
type SavedNode = {
  id: string;
  parentId: string | null;
  attachT: number;
  points: Vec3[];
  radiusStart: number;
  radiusEnd: number;
  leaves: Leaf[];
};
type TreeLabFile = {
  format: "tree-lab-exercise";
  version: 1;
  modelVersion: "2026-11-16-curves";
  exerciseId: string;
  savedAt: string; // ISO timestamp
  kind: "assignment" | "submission";
  metadata: { title: string; worker: string; task: string; review: string };
  tree: { seed: number; config: Record<string, number>; nodes: SavedNode[] };
  history: Cut[];
  cursor: number;
  display: { showLeaves: boolean; leafOpacity: number }; // opacity lưu theo %, không phải 0–1
  randomization?: { lockedFields: string[] };
};
```

`config` phải có đủ 14 key trong bảng thông số. `children`, `start`, `end`, `meshes`, `limit`, `visible` được tái tạo khi nạp. Thứ tự node phải cha trước con. Không yêu cầu ID cành liên tiếp.

`randomization` là trường mở rộng tùy chọn để file cũ vẫn mở được; thiếu trường này tương đương không giữ thông số nào. `modelVersion`, `kind`, `savedAt` được xuất để mô tả file; validator hiện không dùng chúng để xác thực/migration. Không suy ra file có chữ ký hay danh tính đáng tin từ các trường này.

### 10.1. Kiểm tra dữ liệu trước khi nạp

- Giới hạn kích thước ở handler đọc file: 8 MiB.
- Đúng format/version; cấu hình nằm trong phạm vi và đúng bước của `fields`; seed là uint32.
- 2–250 node; đầu tiên là `T0` không có cha; ID duy nhất theo `T0|C\d{3}`; mọi cha đã xuất hiện trước con.
- Mỗi node có 2–65 điểm trục; tọa độ trong [-100,100], hữu hạn; không có hai điểm liên tiếp cách dưới `1e-6`.
- Điểm đầu con nối đúng trục cha tại `attachT`, sai số tối đa `1e-4`.
- Bán kính gốc từ 0,001–3 m; bán kính đầu từ 0,001 tới bán kính gốc.
- Tối đa 100 lá/node, 12.000 lá tổng; t 0–1, vị trí hợp lệ, scale 0,001–5, rotation -20–20, màu 0–1.
- Tối đa 5.000 lệnh; t cắt từ 0,002–1; ID phải tồn tại. Phát lại **cả phần redo**, không cho cắt ngoài đoạn còn lại hoặc cành đã mất do tổ tiên.
- `cursor` nguyên, từ 0 tới độ dài history.
- Title/worker tối đa 120 ký tự; task/review 4.000; exerciseId không rỗng, tối đa 100.
- `showLeaves` boolean, opacity 0–70; danh sách lockedFields chỉ chứa key hợp lệ, không trùng.

`validateSaveData()` trả cấu trúc sạch. Chỉ sau parse/validate thành công mới hỏi xác nhận và gọi `applySaveData()`. File bị từ chối hoặc người dùng hủy phải giữ nguyên cây/lịch sử hiện tại.

Không dùng nội dung JSON để thực thi mã. Metadata đưa vào DOM qua `.value`/`.textContent`, không chèn HTML tùy ý.

### 10.2. Tương thích về sau

- Không gọi lại `makeTree()` để sinh từ seed khi nạp file: thuật toán sinh cây có thể đã đổi.
- Không chỉ lưu ảnh hoặc mesh kết quả: sẽ mất cây gốc, lịch sử và khả năng đánh giá.
- Nếu thêm tham số bắt buộc vào `fields`, phải xử lý file cũ thiếu key; validator hiện yêu cầu đủ key.
- Khi đổi schema, thêm đường migration rõ hoặc báo phiên bản không hỗ trợ; không âm thầm diễn giải theo cấu trúc khác.
- Không đổi ý nghĩa `t`, ID nhánh hay thứ tự lệnh mà không xử lý file cũ.

## 11. Lưu file và tên trùng

Tên gốc là tên bài được làm sạch, tối đa 80 ký tự, cộng `_giao_bai` hoặc `_bai_nop`. Hậu tố trùng đứng trước đuôi: `ten_bai_nop.json`, `ten_bai_nop_1.json`, `ten_bai_nop_2.json`…

### Khi có quyền chọn thư mục

- Dùng `window.showDirectoryPicker({id:'tree-lab-save', mode:'readwrite'})` ở lần lưu đầu; cache handle trong phiên. Đổi thư mục qua nút trong Bài tập.
- `findAvailableFileName()` dò tên bằng `getFileHandle()` không tạo file. `NotFoundError` là chưa có; `TypeMismatchError` nghĩa là có thư mục cùng tên, phải thử hậu tố khác.
- Chỉ sau khi chọn tên mới tạo file và ghi Blob; đóng writable rồi mới báo thành công. Nếu ghi lỗi, thử abort và báo lỗi.
- Không chủ động ghi đè file đã tìm thấy. Khóa `saveInProgress` chống hai lần lưu đồng thời **trong cùng phiên ứng dụng**.
- Không coi quy trình dò tên rồi tạo file là thao tác độc quyền nguyên tử giữa mọi tab/chương trình. Đây là giới hạn cần xét nếu sau này hỗ trợ nhiều người/tiến trình cùng ghi một thư mục.

### Khi trình duyệt không có API chọn thư mục

- Dùng tải Blob bằng anchor. Chỉ tránh các tên ứng dụng đã biết trong `usedDownloadNames`.
- Cố nhớ tối đa 5.000 tên qua localStorage key `TREE_LAB_download_names_v1`; storage có thể không dùng được, khi đó vẫn phải tải được trong phiên.
- File được mở hợp lệ cũng được ghi nhận tên. Danh sách này không phải danh sách file thực tế trong Downloads.
- **Không thể kiểm tra toàn bộ file sẵn có trên máy bằng cơ chế tải thông thường.** Trình duyệt quyết định tên cuối cùng. Không tuyên bố chế độ dự phòng có bảo đảm kiểm tra thư mục như API trên.
- Hủy hộp chọn thư mục không được chuyển sang tải file âm thầm. Lỗi phải được báo, nút lưu phải được mở lại trong `finally`.

Tài liệu API tham khảo: [showDirectoryPicker](https://developer.mozilla.org/en-US/docs/Web/API/Window/showDirectoryPicker), [getFileHandle](https://developer.mozilla.org/en-US/docs/Web/API/FileSystemDirectoryHandle/getFileHandle). Chỉ dùng các API này sau thao tác người dùng và theo quyền trình duyệt cung cấp.

## 12. Bản đồ sửa mã

| Muốn thay đổi | Vị trí cần xem | Liên quan cần kiểm tra |
|---|---|---|
| Min/max/mặc định, checkbox Giữ | `fields`, form sinh động, `randomizeFields()` | Validator, file cũ, bảng thông số trong đặc tả |
| Dáng cây/thân/cành võng | `makeTree()`, `branchPath()`, `sample()` | Nối cha–con, raycast, cắt ở đoạn đi xuống |
| Người tham chiếu/nhãn 1,7 m | `createScalePerson()`, `positionHumanReference()`, `updateHumanLabel()`, animation loop | Đúng đơn vị mét, không bị cắt/đếm là cành, label mỗi nửa split |
| Lá/độ nhìn xuyên | `makeLeafGeometry()`, `populateLeaves()`, vật liệu lá | Instancing, so sánh, opacity, giải phóng tài nguyên |
| Thông tin công ty | `.brand .product-credit`, `meta[name="author"]` | Chính tả tên công ty và khả năng đọc trên màn hình hẹp hoặc ngang |
| Xem lại và nhận xét | `startReview()`, `endReview()`, `drawState()`, `focusCut()` | Phần bị bỏ, vị trí camera, giữ cursor và nhận xét sau khi mở lại |
| Chọn điểm | Pointer handlers, `pick()`, `projectPoint()`, `selectNode()` | Click khác drag/pinch, cành vừa dựng, before/split |
| Cắt/undo/redo | Handler cut, `remainingAt(end)`, `rebuild()` | Phần redo, hậu duệ, cắt thân, nạp lại bài |
| Bố cục điện thoại | CSS `.viewport-bottom`, `.tree-dock`, `.selection`, media queries | Hai mục dưới hàng thao tác, không phủ status bar, xoay máy |
| So sánh | `ensureBeforeGroup()`, `clearBeforeGroup()`, `setViewMode()`, animation loop | Không sửa dữ liệu cắt, scissor/viewport, lá đồng bộ |
| Giao/nộp bài | `snapshotTree()`, `makeSaveData()`, `validateSaveData()`, `applySaveData()` | Round-trip toàn bộ hình học, metadata, history/cursor |
| Tên file/thư mục | `saveFile()`, `findAvailableFileName()`, registry | Hủy/lỗi, không ghi đè, fallback, async tests |
| Camera/đổi hướng | `fitCamera()`, `resize()`, OrbitControls | Khoảng cách, aspect, split, không áp camera theo toàn cửa sổ |

Khi tái tạo cây: giải phóng geometry đã thay và InstancedMesh cũ; không dispose vật liệu/hình học lá dùng chung khi chúng còn được nhóm khác dùng. Bỏ `beforeGroup` khi nguồn cây đổi, không để so sánh với cây cũ. Cây nhiều cành hiện có nhiều mesh; so sánh còn dựng thêm cây gốc, nên không suy ra “mượt trên mọi điện thoại” chỉ từ số lượng node.

## 13. Kiểm thử và bằng chứng hiện có

### 13.1. Kiểm tra tích hợp có trong workspace

`tests/tree-lab.test.cjs` dùng Three.js thật, jsdom và bộ dựng/OrbitControls thay thế. Không khởi chạy trình duyệt, không dùng GPU và không kiểm tra hình ảnh.

Môi trường đã dùng: Node 24.18.0, `three@0.170.0`, `jsdom@31.0.1`. Không cần cài các gói này để người dùng mở HTML; chúng chỉ phục vụ test.

Có thể cài phụ thuộc test vào thư mục tạm để không thêm node_modules/package files vào dự án. Ví dụ PowerShell, chạy từ thư mục dự án:

```powershell
$testDeps = Join-Path $env:TEMP 'tree-lab-test-deps'
npm install --prefix $testDeps three@0.170.0 jsdom@31.0.1
$env:NODE_PATH = Join-Path $testDeps 'node_modules'
node tests/tree-lab.test.cjs
```

Kết quả gần nhất: **23 nhóm kiểm tra đạt**, bao gồm khởi tạo/chọn mặc định/hủy chọn; không còn panel; dialogs và phần chỉnh mobile; camera đổi aspect; lưu toàn bộ cây gốc và lịch sử; so sánh không sửa dữ liệu; đồng bộ lá; hậu tố tên tải; giao bài không kèm đáp án; round-trip; undo/redo sau tải; file lỗi giữ phiên; ID cành không liên tiếp; cây 25 m tối đa; Giữ thông số; file cũ thiếu lockedFields; dò trùng trong thư mục giả lập; lỗi ghi/hủy/nhấn lưu liên tục; thứ tự dock dưới toolbar; chiều cao người 1,7 m/chân ở đất; lá tới đầu ngọn, bỏ/khôi phục khi cắt thân; density=0; file cũ không bị thêm lá. Nhóm kiểm tra xem lại bao gồm phóng tới điểm gần mặt đất, xem một lệnh đã hoàn tác, trở về chế độ so sánh và mở lại nhận xét đã lưu.

Harness hiện đọc module script bằng regex và chèn `window.__tests` trước đoạn catch cuối để lấy các hàm bên trong. Nó phụ thuộc chuỗi import/catch và tên hàm hiện tại. Nếu đổi module/refactor, cập nhật harness tương ứng; không kết luận logic hỏng chỉ vì cơ chế trích xuất không còn khớp. Khi kiểm tra hàm lưu async, `await button.onclick()`, không chỉ `.click()` rồi đọc kết quả ngay.

### 13.2. Kiểm tra thực tế cần làm khi có môi trường phù hợp

| Nhóm | Tình huống tối thiểu |
|---|---|
| Hiển thị | Desktop rộng, 390×844, màn nhỏ khoảng 320 px, điện thoại ngang; không tràn ngang/che nút |
| Chỉnh cây | Mở/đóng hai mục nhỏ; chỉnh đủ 14 giá trị; giữ tất cả/một phần/không giữ |
| Cảm ứng | Kéo một ngón, pinch, pan hai ngón; không tạo lựa chọn ngoài ý muốn |
| Cắt | Giữa cành, cành võng, phía trước/sau chỗ mọc con, thân trên/dưới tầng cành |
| Lịch sử | Nhiều cắt, undo/redo, cắt mới sau undo, reset |
| So sánh | Cả ba mode, camera đồng bộ, ẩn lá, resize/đổi hướng trong split |
| File | Giao → nộp → đánh giá; file hỏng; cursor có redo; metadata/nhận xét |
| Thư mục | File tên gốc đã có, nhiều hậu tố đã có, thư mục cùng tên; hủy quyền; ghi lỗi |
| Dự phòng | Trình duyệt không có API thư mục; storage không dùng được; tên tải lặp |
| Tài nguyên | Cấu hình tối đa, tạo cây lặp lại, trước/sau lặp, theo dõi lỗi và memory/draw calls |
| Lỗi nền tảng | CDN bị chặn, không WebGL, mất context |

Bản đầu từng được mở bằng Chrome/Playwright và giả lập cảm ứng. **Bố cục mới nhất và hộp chọn thư mục thật chưa có kiểm tra hiển thị trực tiếp**; công cụ điều khiển trước đó bị chặn truy cập trang tệp cục bộ. Không tái dùng kết quả bản đầu để tuyên bố bản hiện tại đã đạt kiểm tra UI/GPU/thiết bị thật. Khi không chạy được, ghi đúng phần chưa xác minh; không tìm cách vượt chính sách của công cụ.

## 14. Quy tắc sửa và bàn giao

1. Xác định thay đổi nằm ở UI, mô hình, lịch sử hay file; đọc các hàm trong bản đồ sửa trước khi can thiệp.
2. Giữ tên `TREE_LAB.html`, cách chạy độc lập và các hành vi đã chốt, trừ khi người dùng yêu cầu đổi.
3. Thay đổi schema phải có kế hoạch tương thích; thay đổi UI phải giữ đường truy cập giao/nộp bài, lịch sử, lá và hướng dẫn.
4. Không chèn nội dung kỹ thuật triển khai vào luồng công nhân nếu không giúp họ ra quyết định.
5. Kiểm tra đúng phần bị tác động; dữ liệu/hình học/lưu file cần test có ý nghĩa. Kiểm tra DOM không thay thế kiểm tra bố cục thật.
6. Cập nhật **tài liệu này** nếu đổi hợp đồng dữ liệu, giới hạn tham số, hành vi chính, vị trí điều khiển hoặc cách kiểm tra. Cập nhật hướng dẫn sử dụng nếu quy trình người dùng đổi.
7. Báo rõ tệp đã sửa, hành vi mới, kiểm tra đã chạy và giới hạn còn lại. Không tự triển khai hosting/gửi bài/đổi quyền khi công việc chỉ là sửa tệp trong workspace.

## 15. Giới hạn kỹ thuật và hướng phát triển chưa được giao làm

- Chưa có chấm điểm kỹ thuật, bộ tiêu chuẩn nghề hoặc mô phỏng sinh học.
- File JSON có thể sửa bằng tay, không ký số, không xác thực danh tính công nhân.
- Không có tự lưu phiên, đồng bộ cloud hay kho bài tập tập trung.
- Chưa đóng gói Three.js ngoại tuyến; chưa có pipeline build/package chính thức.
- Nhiều đoạn gỗ là nhiều mesh; cần đo GPU/điện thoại trước khi hứa hiệu năng hoặc thêm nhiều cấp nhánh.
- Lá trong suốt dùng instancing và không ghi depth: có thể có giới hạn thứ tự pha trộn; ưu tiên nhìn rõ cành.
- `t`/khoảng cách dọc thân là tham số/xấp xỉ, không phải đo chiều dài cung chính xác trong mọi trường hợp.
- Giới hạn camera/đất là các ràng buộc đơn giản, không phải mô phỏng va chạm hoàn chỉnh.
- Quy trình dò tên rồi ghi file chưa có bảo đảm độc quyền giữa nhiều tab/ứng dụng cùng ghi thư mục. Chế độ tải không được quyền đọc toàn bộ Downloads.
- Test dùng DOM/mocks chưa chứng minh bố cục, native picker, gesture trên thiết bị thật hay memory GPU dài hạn.

Những mục trên là thông tin để bảo trì và cân nhắc khi có yêu cầu mới; không tự biến thành phạm vi một lần chỉnh giao diện nhỏ.


### Ghi nhận kiểm tra hình người chi tiết

Đã xem một ảnh dựng riêng hình người bằng Three.js SVGRenderer trên CPU rồi raster hóa; đó là kiểm tra hình học dáng người, không phải ảnh chụp WebGL hoặc bố cục ứng dụng. GPU có cách chiếu sáng/làm mượt khác. Bộ kiểm tra 22 nhóm đã chạy lại, gồm chiều cao đúng 1,7 m và tọa độ/normal hữu hạn của mọi mesh người. Hình người hiện là mô hình hình học có chi tiết, không phải scan người hay tài sản photorealistic.
