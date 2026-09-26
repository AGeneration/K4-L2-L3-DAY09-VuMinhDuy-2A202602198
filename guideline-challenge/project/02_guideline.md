# Annotation guideline — TODO tên bài toán

**Version:** v0

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

TODO — label để làm gì; object/region nào trong scope, cái nào ngoài scope.

## 2. Annotation unit

TODO — image, frame hay track? Instance hay region? Khi nào một object được tính là instance mới?

## 3. Geometry rule

Mỗi nhóm road element dùng **một loại shape duy nhất** — không trộn shape trong cùng nhóm.

| Nhóm element | CVAT shape | Ghi chú |
|---|---|---|
| Traffic light | Rectangle | Một box cho mỗi **light head** (cụm đèn tín hiệu) |
| Traffic sign | Rectangle | Một box cho mỗi **sign panel** (mặt biển) |
| Lane marking | Polyline | Một polyline cho mỗi đoạn vạch/road edge nhìn thấy |
| Drivable area | Polygon | Một polygon cho mỗi vùng liền nhau |

### 3.1 Rectangle — Traffic light & Traffic sign

- **Tight box**: box bao sát phần đối tượng nhìn thấy được trên ảnh, hạn chế tối đa phần nền thừa.
  - Traffic light: box quanh **signal head** (phần đèn), **không** lấy cả pole/gantry/cần đèn trừ khi guideline batch yêu cầu.
  - Traffic sign: box quanh **sign panel/face** (mặt biển), **không** lấy cả cột/khung đỡ. Biển xếp chồng (stacked signs) có ý nghĩa khác nhau → tách thành box riêng cho từng panel.
- **Mỗi object = một annotation riêng.** Không dùng một box bao nhiều object độc lập.
- **Box không được vượt ra ngoài biên ảnh.** Nếu object bị cắt mép, box dừng tại biên ảnh.
- **Không bỏ sót object rõ ràng** chỉ vì kích thước nhỏ hoặc ở xa. Nếu quá nhỏ/mờ để xác định class chắc chắn → dùng `readable = uncertain` hoặc `needs_review = true` thay vì đoán.

### 3.2 Polyline — Lane marking

- **Annotation unit**: một marking/road-edge segment nhìn thấy có meaning nhất quán. Không suy luận lane centerline nếu task chỉ hỏi lane marking.
- **Start / end**: bắt đầu tại nơi marking đủ rõ; dừng khi marking biến mất, quá mờ, ra khỏi ảnh hoặc bị che.
- **Point density**: đoạn thẳng dùng ít điểm; đoạn cong thêm điểm ở nơi curvature thay đổi. Không "rải điểm" theo texture asphalt.
- **Occlusion**: không nối polyline xuyên qua xe/vật che. Nếu vạch bị gián đoạn do vật che → tách thành hai polyline riêng.
- **Crosswalk / curb**: là category riêng (xem mục 4 Taxonomy), không gộp tất cả thành "lane line".

### 3.3 Polygon — Drivable area

- **Bám ranh giới vật lý**: curb, island, barrier, sidewalk, parked vehicle tạo boundary thật. Polygon phải bám theo các ranh giới này.
- **Functional area, not color**: không tô toàn bộ vùng có texture asphalt. Xét curb/island/sidewalk, hướng giao thông, right-of-way và vật cản.
- **Không tự giao (self-intersection)**: polygon không được tự cắt.
- **Hạn chế điểm thừa** trên đoạn thẳng; tăng mật độ điểm ở đoạn cong/phức tạp.
- **Vùng tách rời (disconnected)**: tạo nhiều polygon riêng nếu vùng hợp lệ không liền nhau.
- **Không kéo polygon** quá xa đến horizon khi lane không còn nhìn rõ, trừ khi guideline batch cho phép extrapolation.

### 3.4 Occlusion & Truncation (áp dụng cho Rectangle)

| Attribute | Khi nào bật | Ví dụ |
|---|---|---|
| `occluded = true` | Object vẫn nằm trong scene nhưng bị object khác che một phần | Biển báo bị cây che một phần; đèn bị xe tải che |
| `truncated = true` | Object bị cắt bởi biên ảnh, phần còn lại nằm ngoài frame | Biển báo chỉ xuất hiện một phần ở mép ảnh |
| Cả hai = true | Object vừa bị che vừa bị cắt biên ảnh | Đèn ở mép ảnh và đồng thời bị object khác che |

Với object bị che/cắt, **vẫn annotate** nếu còn đủ bằng chứng thị giác để xác định class; gán attribute phù hợp.

### 3.5 Tolerance

- **Rectangle**: box lệch ≤ **3 pixel** mỗi cạnh so với tight boundary là chấp nhận được.
- **Polyline**: điểm lệch ≤ **3 pixel** so với tim/biên marking là chấp nhận được.
- **Polygon**: boundary lệch ≤ **5 pixel** so với ranh giới vật lý quan sát được.
- Sai lệch vượt tolerance → **Major defect** nếu gần ego vehicle / intersection, **Minor defect** nếu ở xa.

## 4. Taxonomy

Project dùng **4 label chính** (mỗi label = một loại road element) và **2 label phụ trợ**. Thuộc tính thay đổi theo object/frame là **attribute**, không tạo thêm class.

Bảng đầy đủ ở `03_ontology_and_cvat_setup.md` — hai nơi phải khớp nhau.

### 4.1 Bảng class & attribute

#### Label: `traffic_light` — Rectangle

| Attribute | Input type | Allowed values | Default | Mutable? |
|---|---|---|---|---|
| `state` | select | `__undefined__`, `red`, `yellow`, `green`, `red_yellow`, `off`, `unknown` | `__undefined__` | true |
| `relevance` | select | `__undefined__`, `ego_relevant`, `not_relevant`, `unknown` | `__undefined__` | true |
| `occluded` | checkbox | `false` | `false` | false |
| `truncated` | checkbox | `false` | `false` | false |
| `needs_review` | checkbox | `false` | `false` | false |

#### Label: `traffic_sign` — Rectangle

| Attribute | Input type | Allowed values | Default | Mutable? |
|---|---|---|---|---|
| `sign_family` | select | `__undefined__`, `prohibitory`, `mandatory`, `danger`, `other`, `unknown` | `__undefined__` | false |
| `sign_class` | text | GTSDB class ID hoặc mô tả ngắn (ví dụ `speed_limit_50`); để trống nếu không đủ evidence | _(trống)_ | false |
| `readable` | select | `__undefined__`, `yes`, `no`, `uncertain` | `__undefined__` | false |
| `occluded` | checkbox | `false` | `false` | false |
| `truncated` | checkbox | `false` | `false` | false |
| `needs_review` | checkbox | `false` | `false` | false |

#### Label: `lane_marking` — Polyline

| Attribute | Input type | Allowed values | Default | Mutable? |
|---|---|---|---|---|
| `laneType` | select | `__undefined__`, `single_white`, `single_yellow`, `double_white`, `double_yellow`, `single_other`, `double_other`, `crosswalk`, `road_curb` | `__undefined__` | false |
| `laneStyle` | select | `__undefined__`, `solid`, `dashed`, `unknown` | `__undefined__` | false |
| `laneDirection` | select | `__undefined__`, `parallel`, `vertical` | `__undefined__` | false |
| `visibility` | select | `visible`, `partially_occluded`, `faded`, `truncated` | `visible` | false |
| `needs_review` | checkbox | `false` | `false` | false |

#### Label: `drivable_area` — Polygon

| Attribute | Input type | Allowed values | Default | Mutable? |
|---|---|---|---|---|
| `areaType` | select | `__undefined__`, `direct`, `alternative` | `__undefined__` | false |
| `needs_review` | checkbox | `false` | `false` | false |

#### Label: `ignore_region` — Polygon (phụ trợ)

Dùng cho vùng bỏ qua rõ ràng (ví dụ: dashboard reflection, ảnh bị hỏng, vùng không thuộc scene giao thông). Không có attribute.

#### Label: `image_escalate` — Tag (phụ trợ)

Dùng tag cho cả ảnh khi cần escalate toàn bộ ảnh lên reviewer (ví dụ: ảnh quá tối/mờ, scene không phải giao thông, không chắc chắn scope). Không có attribute.

### 4.2 Class hay attribute — Rationale

| Thuộc tính | Là class hay attribute? | Lý do |
|---|---|---|
| `traffic_light` / `traffic_sign` / `lane_marking` / `drivable_area` | **Class** (label) | Mỗi loại element có geometry khác nhau (rectangle vs polyline vs polygon) và meaning hoàn toàn khác nhau |
| `state` (red/green/…) | **Attribute** | Cùng một đèn, state thay đổi theo frame; tạo class riêng cho mỗi state sẽ gây explosion và không track được identity |
| `relevance` | **Attribute** | Relevance phụ thuộc vào context (ego lane/route), không phải bản chất vật lý của đèn/biển |
| `sign_family` / `sign_class` | **Attribute** | Số lượng class biển báo rất lớn (hàng trăm); dùng hierarchy qua attribute giúp label menu gọn, giảm disagreement, và cho phép downstream flatten thành class ID sau QC |
| `laneType` / `laneStyle` | **Attribute** | Marking về hình học giống nhau (polyline), chỉ khác ý nghĩa vận hành |
| `areaType` (direct/alternative) | **Attribute** | Cùng là vùng drivable, chỉ khác mức ưu tiên/right-of-way |

### 4.3 Default values và bias

- Mọi attribute dạng select đều dùng **`__undefined__`** làm default (ngoại trừ `visibility` mặc định `visible` vì phần lớn marking nhìn rõ).
- `__undefined__` buộc annotator phải chủ động chọn giá trị. Nếu `__undefined__` còn lại trong export → chưa gán, cần rework.
- **Cảnh báo bias**: nếu default là một giá trị thật (ví dụ `green`, `direct`), annotator quên gán sẽ tạo ra data sai "im lặng" — không phát hiện được khi QC tự động.

### 4.4 Khi nào dùng `unknown`

| Tình huống | Giá trị dùng | Khi nào |
|---|---|---|
| Đèn quá xa / bị lóa / motion blur → không xác định được màu | `state = unknown` | Có evidence vật lý là traffic light nhưng không đủ evidence cho state cụ thể |
| Đèn không rõ áp cho lane nào | `relevance = unknown` | Không đủ evidence từ position/arrow/gantry/lane geometry để quyết định |
| Biển quá nhỏ / bị che → không đọc được nội dung | `readable = uncertain`, `sign_class` = _(trống)_ | Nhận ra hình dạng biển nhưng không đủ pixel để exact-class |
| Biển nhận ra family nhưng không chắc class cụ thể | `sign_family` gán được, `sign_class` = _(trống)_ | Ví dụ: nhận ra biển tròn đỏ (prohibitory) nhưng không rõ nội dung |
| Vạch mờ / đổi màu → không rõ style | `laneStyle = unknown` | Vạch bị phai, glare hoặc night scene không đủ evidence |

> **Nguyên tắc**: `unknown` là quyết định có ý thức, **không phải default**. Annotator phải ghi nhận rằng mình đã kiểm tra và không đủ evidence, thay vì bỏ qua. Mỗi `unknown` nên đi kèm `needs_review = true` nếu downstream cần expert quyết định.

## 5. Inclusion / exclusion

TODO — trường hợp bắt buộc label; trường hợp ignore.

## 6. Visibility / occlusion

TODO — bị che một phần, bị cắt mép ảnh, nhỏ/xa, phản chiếu, loá, độ tin cậy thấp.

## 7. Ambiguity / escalation

TODO — khi nào LABEL / IGNORE / UNKNOWN / ESCALATE khi bằng chứng không đủ. Ghi rõ **thể hiện mỗi quyết định trong
CVAT bằng cách nào** (attribute, giá trị, tag…), để quyết định đó nhìn thấy được trong file export.

## 8. Temporal rule

TODO — nếu là video/track: track bắt đầu/kết thúc khi nào, attribute nào mutable, xử lý chuyển trạng thái và bị che
ngắn. Task ảnh tĩnh ghi "Không áp dụng — task ảnh tĩnh".

## 9. Examples

TODO — positive, negative và edge case, mỗi ví dụ có sample_id (split example/calibration) và expected output.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

## 10. Common mistakes

TODO — những lỗi reviewer có khả năng gặp nhiều nhất và cách tránh.
