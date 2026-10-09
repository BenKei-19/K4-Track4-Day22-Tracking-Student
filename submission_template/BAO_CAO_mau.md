# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** 01 **Thành viên:** Trần Thị Lan, Phạm Minh Hiếu

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.6 | Hai phụ nữ đi sát nhau giữa quảng trường, áo khoác xanh nhạt (ID 23) và áo khoác tím (ID 3), giữ nguyên ID từ frame 240 đến 360 dù hai hộp chồng lên nhau. Lỗi thấy được: người áo tím có thêm nhãn ID 19 nằm khít lên ID 3, tức một người bị đếm thành hai track (xem phần 3). Người nhỏ ở cuối quảng trường phần lớn không có hộp: detector bỏ sót, tracker không có gì để nối. | `strongsort` conf 0.15: thêm nhiều hộp người nhỏ ở xa nhưng những người đó đổi ID liên tục (IDSW 110, HOTA 29.18). `bytetrack` conf 0.3: ít đổi ID (IDSW 12) nhưng bỏ sót nhiều người hơn (HOTA 26.91). |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.1 | 0.5 | Người đi lẻ ở tiền cảnh giữ một ID suốt quãng dài. Đám đông dày ở góc trên bên trái gần như không có hộp ở mọi tracker: người quá nhỏ khi ảnh 1920 px bị thu về 640 px, nên hạ conf để lấy thêm người. Một lần đổi ID thấy rõ: người mặc gi-lê đứng cạnh dãy xe đạp mang ID 5 đến frame 508; một phụ nữ đi ngang che anh ta, hộp mất đến frame 568 rồi anh ta được cấp ID mới 61. Các hộp đứng yên lâu đều là người thật đang đứng, không phải cột hay xe. | `bytetrack` conf 0.15: ID rất bền nhưng ít hộp hơn hẳn, bỏ sót thêm người trong cảnh đông. `ocsort` conf 0.3: nhiều track ngắn bị cắt vụn hơn. |
| video_3 (camera di động, ảnh nhỏ) | ocsort | 0.3 | 0.5 | Người đi sát ống kính nhiều lần che gần hết khung hình. Hai người đàn ông ở xa giữ ID 59 và 93 đến khoảng frame 467; ở frame 440 người đi ngang che làm ID 93 mất hộp vài frame, rồi được nối lại đúng ID 93. Nhiều track vẫn ngắn vì người lướt qua camera rất nhanh rồi ra khỏi khung. | `botsort` conf 0.3 (có Re-ID và bù chuyển động camera): mất hai người đàn ông đó khi bị che ở frame 440 (ID 126 và 110), đến frame 450 cấp ID mới 174 và 175; cả video đứt ID nhiều hơn rõ. `ocsort` conf 0.15: hộp chập chờn, ID nhảy nhiều. |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.15 | 0.5 | Không thấy hộp giả trên bóng phản chiếu ở lan can kính hay sàn bóng, kể cả ở conf 0.15. Bắt được cả người bị che một nửa sau người áo đỏ (ID 9, frame 60). Lỗi đổi ID thấy rõ: người áo sơ mi trắng mang ID 6 đến frame 322; một người áo sẫm đi ngược về phía camera che kín anh ta (frame 310–337), khi hiện lại anh ta thành ID 45. | `bytetrack` conf 0.15: cũng đổi ID người áo trắng ở đúng chỗ bị che đó (5 thành 38), và nhìn chung có ít hộp hơn BoT-SORT trên mỗi frame. `ocsort` / `deepocsort` conf 0.3: nhiều track bị cắt ngắn khi camera tiến lên. |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.1 | 0.5 | Xe bus chạy và rung làm cả khung hình trượt. Người áo khoác trắng (ID 99) và người quần đỏ (ID 92) đi sát nhau, hộp chồng lên nhau, vẫn giữ đúng ID từ frame 340 đến 400. Hạ conf bắt thêm người nhỏ ở xa (ID 116, 119, 120). Không thấy hộp giả trên cột chắn vàng hay biển báo. | `strongsort` conf 0.3: hai người đàn ông đi cạnh nhau bên trái nhóm đó bị tráo ID; một người đổi ID 4 lần trong frame 347–438 (122, 130, 125, 137, trong đó 125 là ID của người đi bên cạnh), trong khi BoT-SORT giữ người này một ID (108) suốt đoạn. `bytetrack` conf 0.3: giữ ID nhưng mất hộp hai người đàn ông bên trái ở nhiều frame (một người chỉ có hộp khoảng 4/10 thời gian) và bỏ sót phần lớn người ở xa. `ocsort` conf 0.3: rất nhiều track ngắn. |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nhom_video1-pedestrian       HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            30.384    18.16     51.07     18.769    76.968    54.446    82.244    83.457    30.921    37.398    77.96     29.155
COMBINED                           30.384    18.16     51.07     18.769    76.968    54.446    82.244    83.457    30.921    37.398    77.96     29.155

CLEAR: nhom_video1-pedestrian      MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag
video_1                            19.472    81.251    19.617    22.001    90.223    12.903    19.355    67.742    15.346    4088      14493     443       27        8         12        42        100
COMBINED                           19.472    81.251    19.617    22.001    90.223    12.903    19.355    67.742    15.346    4088      14493     443       27        8         12        42        100

Identity: nhom_video1-pedestrian   IDF1      IDR       IDP       IDTP      IDFN      IDFP
video_1                            30.218    18.793    77.069    3492      15089     1039
COMBINED                           30.218    18.793    77.069    3492      15089     1039

Count: nhom_video1-pedestrian      Dets      GT_Dets   IDs       GT_IDs
video_1                            4531      18581     51        62
COMBINED                           4531      18581     51        62
```

Một số cấu hình đã chấm trên video_1 (đủ 600 frame) để chọn cấu hình nộp:

| Cấu hình | HOTA | MOTA | IDF1 | IDSW | Hộp giả (FP) |
|---|---|---|---|---|---|
| **botsort conf 0.3 iou 0.6 (nộp)** | **30.38** | 19.47 | 30.22 | 27 | 443 |
| botsort conf 0.1 iou 0.6 | 30.33 | 20.97 | 31.11 | 23 | 562 |
| botsort conf 0.3 iou 0.5 | 29.46 | 19.80 | 29.34 | 25 | 337 |
| strongsort conf 0.15 iou 0.5 | 29.18 | 19.87 | 32.57 | 110 | 1438 |
| ocsort conf 0.3 iou 0.4 | 28.62 | 19.85 | 29.83 | 36 | 247 |
| ocsort conf 0.3 iou 0.5 | 27.46 | 19.80 | 28.72 | 43 | 254 |
| deepocsort conf 0.3 iou 0.5 | 27.40 | 19.78 | 27.82 | 50 | 249 |
| bytetrack conf 0.3 iou 0.5 | 26.91 | 17.29 | 25.71 | 12 | 107 |
| ocsort conf 0.15 iou 0.5 | 26.00 | 19.43 | 29.30 | 164 | 1490 |

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

**video_1 (đánh giá bằng số).** Camera đứng yên, ban ngày, mật độ vừa; điểm nghẽn là detector: khoảng 78% hộp nhãn bị bỏ sót vì người ở xa quá nhỏ khi ảnh thu về 640 px, nên đổi tracker chỉ làm HOTA chênh khoảng 3–4 điểm. BoT-SORT hợp nhất vì có ngưỡng riêng bên trong: hạ conf thì nó dùng thêm hộp yếu để nối track cũ mà không mở track mới bừa bãi, còn OC-SORT, DeepOCSORT và StrongSORT nhận mọi hộp, nên ở conf 0.15 hộp giả tăng khoảng 6 lần và số lần đổi ID tăng 2.5–4 lần. Nâng `iou` NMS lên 0.6 giữ lại hộp của những người đi cạnh nhau, AssA tăng từ 48.2 lên 51.1. ByteTrack ở conf 0.3 chỉ đổi ID 12 lần nhưng MOTA thấp hơn, vì MOTA phạt cả bỏ sót chứ không chỉ đổi ID. Track trùng ID 3 và 19 là lỗi của BoT-SORT trong boxmot 10.0.42 (bước liên kết thứ hai lọc hộp bằng `np.logical_or` thay vì `np.logical_and`, nên nhận lại cả hộp đã gán cho track khác); chấm thử một bản sao đã bỏ hộp trùng thì hộp giả giảm từ 443 xuống 334 nhưng HOTA gần như không đổi (30.27).

**video_3 (đánh giá bằng mắt).** Camera cầm tay đi bộ, ảnh nhỏ, ít khung hình/giây, nên người dịch chuyển rất xa giữa hai frame và hay bị người đi sát ống kính che kín. OC-SORT hợp hơn vì khi người xuất hiện lại sau lúc bị che, nó nối track bằng các lần quan sát thật gần nhất thay vì tin vào dự đoán Kalman đã trôi xa. Các tracker có Re-ID kém hơn ở cảnh này: ảnh người nhỏ và nhòe, ngoại hình đổi nhiều khi người lướt sát camera, và bù chuyển động camera dễ ước lượng sai khi ít frame. Hạ conf làm OC-SORT tệ đi vì nó không tự lọc hộp yếu, nên giữ conf 0.3.

**video_5 (đánh giá bằng mắt).** Quay từ xe bus đang chạy và rung, nên cả khung hình trượt chứ không riêng người di chuyển. Tracker chỉ dựa vào chuyển động như OC-SORT sinh rất nhiều track ngắn vì dự đoán vị trí không tính đến việc camera đang đi. BoT-SORT bù chuyển động camera và dùng thêm ngoại hình, nên những người đi sát nhau vẫn giữ đúng ID: hai phụ nữ ID 99 và 92, và người đàn ông ID 108 mà StrongSORT tráo ID với người bên cạnh. Người ở xa rất nhỏ nên hạ conf xuống 0.1 để bắt thêm người, và nhờ ngưỡng nội bộ của BoT-SORT nên không thấy hộp giả nhấp nháy trên cột hay biển báo.

**video_2 (đánh giá bằng mắt).** Camera tĩnh trên cao nhưng cảnh rất đông: lỗi lớn nhất là đám đông nhỏ phía xa không được phát hiện, không phải lỗi gán ID. Người đi lẻ được giữ ID tốt với mọi tracker, nên chọn cấu hình bắt được nhiều người nhất mà không thêm hộp giả. Lần đổi ID quan sát được xảy ra khi một người mất hộp khoảng 2 giây (60 frame), chạm giới hạn 60 frame mà BoT-SORT còn giữ track đã mất, nên khi người đó hiện lại tracker không nối được với track cũ.

## 4. Nếu có thêm thời gian

Thử sửa `logical_or` thành `logical_and` trong BoT-SORT của boxmot để bỏ track trùng, tăng `track_buffer` cho video_2 để giữ track lâu hơn khi người bị che, và thử Re-ID lớn hơn (ví dụ `osnet_x1_0`) cho video_3 và video_4. Với video_2 và video_5, thử chia ảnh thành ô (tiling) hoặc tăng kích thước ảnh đầu vào để bắt người nhỏ. Tất cả ghi rõ là phần mở rộng, vì bài chính khóa thư viện và detector ở 640 px.
