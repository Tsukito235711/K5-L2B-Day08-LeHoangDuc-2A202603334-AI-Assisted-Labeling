# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 641; đối tượng sau duyệt: 629; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 620 | 620/641 = 96.7% | 641 box máy |
| edited (sửa) | 1 | 1/641 = 0.2% | 641 box máy |
| deleted (xoá) | 20 | 20/641 = 3.1% | 641 box máy |
| added (vẽ thêm) | 8 | 8/629 = 1.3% | 629 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 3.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 293 | 0 | 2 | 1 |
| bicycle | 7 | 0 | 1 | 1 |
| motorbike | 237 | 0 | 2 | 1 |
| car | 54 | 0 | 1 | 1 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 0 | 1 |
| three_wheeler | 23 | 1 | 14 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 54 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 47 | 6 | 50.5% (KTC 95%: 41–60%) | 88.9% (KTC 95%: 78–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - extra: 1/3
  - missing: 1/3
  - loose: 1/3
  - lọt: img_022.jpg (wrong_class), img_031.jpg (extra), img_003.jpg (wrong_class), img_005.jpg (missing), img_009.jpg (loose), img_010.jpg (extra), img_018.jpg (loose), img_029.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
