# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | <<từ threshold_sweep>> | y26_c025, y26_c05: conf 0.25 bắt được vật nhỏ ở xa nhưng thêm box thừa; conf 0.5 mất vật nhỏ (215 box xuống 139 trên 8 ảnh pilot) |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | <<>> | yoloe_tw: chỉ 8 box trên 8 ảnh pilot, ít box thừa nhưng sót xích lô (img_002) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<dán lệnh bạn đã chạy>>
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`). Lý do: mask bám sát thân tuk-tuk và người ở img_013, img_004 (dòng sam21_b); chạy trên Colab T4 nên không lo RAM; SAM 3 cần quyền truy cập HuggingFace chưa có nên bỏ qua.
- Lỗi không sửa được bằng đổi ngưỡng: YOLO26 gọi tuk-tuk là `car` (img_013, img_004), `truck` (img_007) hoặc bỏ sót hẳn (img_011). Đổi ngưỡng không đổi nhãn. Cách sửa: đổi nhãn sang `three_wheeler` bằng tay khi review, và dùng YOLOE bổ sung.
- Box điểm cao mà sai: img_007 `truck 0.77` cho xe xanh ở mép trái. Box điểm thấp mà có thể đúng: img_007 YOLOE `three_wheeler 0.28` (ca mơ hồ, <<đối chiếu guideline xem có tính là three_wheeler không>>).
- Giới hạn: chọn ngưỡng trên chính 4 ảnh gold nên số đo máy trên gold là lạc quan.


## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 620 | 96.7% | 641 box máy |
| edited | 1 | 0.2% | 641 box máy |
| deleted | 20 | 3.1% | 641 box máy |
| added | 8 | 1.3% | 629 đối tượng sau duyệt |

(Đã loại 27 đối tượng quanh các bẫy. Trong deleted + added có 5 ca đổi lớp, nên số vật máy sót thật ≈ 3.)

- Lớp máy tệ nhất: **three_wheeler**. Máy ra 38 box (23 accepted, 1 edited, 14 deleted), tức xóa 14/38 = 36.8%, so với person 2/295 và motorbike 2/239. <<dẫn ảnh cụ thể bạn đã xóa box three_wheeler và vì sao: ví dụ box YOLOE sai vật, trùng với box YOLO26>>
- So với gold: precision của **máy** 50.5% (48/95 box máy, KTC 95%: 41–60%), recall 88.9% (48/54 đối tượng gold, KTC 95%: 78–95%). Máy ít sót nhưng sinh nhiều box thừa (47 FP). Con số này lạc quan vì chọn ngưỡng trên chính 4 ảnh gold, và chỉ có 54 đối tượng. <<giải thích 47 FP: bạn đã kiểm FP nằm ở lớp nào / vật nhỏ ở xa hay chưa?>>

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **4/12 = 33.3%** (KTC 95%: 14–61%), dưới ngưỡng đạt 80%. Mỗi loại bẫy bắt 1/3: wrong_class, extra, missing, loose.
- Bẫy lọt (8): img_022 (wrong_class), img_003 (wrong_class), img_031 (extra), img_010 (extra), img_005 (missing), img_029 (missing), img_009 (loose), img_018 (loose). Ba trong tám bẫy lọt nằm ở ảnh dày xe máy (005, 018, 029).
- Vì sao lọt, bạn đã bỏ bước nào của checklist? <<viết theo trải nghiệm thật: ví dụ có quét ảnh trước khi bật nhãn máy không, có nghỉ giữa chừng không, ảnh dày có bị duyệt vội không>>
- Accepted 96.7% mà bắt bẫy chỉ 33.3%: số liệu cho thấy tôi giữ nguyên gần hết nhãn máy và bẫy lọt qua vì tôi tin nhãn thay vì nhìn ảnh (thiên kiến tự động hóa). Hai lỗi nhãn sai lớp lọt 2/3 cho thấy phần kiểm nhãn lớp yếu nhất. <<sửa lại nếu thực tế bạn duyệt khác>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<15 phút>> / 4 ảnh = <<3.75 phút/ảnh>> vs duyệt <<24 phút>> / 24 ảnh = << 1 phút/ảnh>>. << ước lượng>>. Thiên lệch: tôi gán gold trước nên đã quen cảnh và loại vật, <<nêu hướng ảnh hưởng tới so sánh thời gian>>.
- Kết luận: **chỉ cho các lớp person, motorbike, car** (accepted gần như nguyên, xóa/thêm rất ít); **không cho three_wheeler** (xóa 36.8%, tuk-tuk còn bị gán `car`/`truck` hoặc sót ở Part 2). Dựa trên bảng theo lớp ở trên. <<thêm kết luận về thời gian khi có số>>
- **Ba quy tắc giảm thiên kiến** (bản nháp, chỉ giữ quy tắc bạn thực sự sẽ làm):
  1. Quét ảnh và tự đếm vật chính **trước khi** bật nhãn máy, ghi số đếm; kiểm tra bằng số vật tự đếm so với số sau duyệt và tỉ lệ bắt bẫy `missing`.
  2. Ảnh dày xe máy (005, 018, 021, 024, 028, 029) duyệt từng vùng sau khi zoom, không duyệt cả ảnh một lượt; kiểm tra bằng số bẫy bắt được ở các ảnh này.
  3. Sau khi duyệt xong, quét lần hai **chỉ để kiểm nhãn lớp** của mọi box (đặc biệt `three_wheeler`, `car`, `truck`); kiểm tra bằng số bẫy `wrong_class` bắt được (hiện 1/3).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>



## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có, dùng Claude để (1) gỡ lỗi cài đặt (pip.exe bị Application Control chặn, lệch phiên bản torch/torchvision), (2) hướng dẫn thứ tự các Part, (3) soạn nháp nhận xét cho `model_log.csv` từ ảnh viz tôi gửi. Tôi đã đối chiếu nhận xét với ảnh viz và `runs.csv` trước khi ghi. <<sửa nếu bạn làm khác>>
- Gold có bị nhiễm không? <<không / có>>. Thứ tự thực tế: gán tay 4 ảnh gold và commit `gold_manual.json` trước, sau đó mới chạy model ở Part 2. <<xác nhận bạn không xem pre-label hay hỏi AI về 4 ảnh gold trước khi gán xong>>
- Máy / GPU / CVAT: máy Windows chỉ CPU; model chạy trên Colab T4; CVAT chạy bằng Docker trên máy. Vướng mắc: `pip.exe` bị chặn bởi Application Control, `torch` và `torchvision` lệch phiên bản gây lỗi `NP_SUPPORTED_MODULES` khi chạy `prelabel.py` trên máy, nên chuyển sang Colab.
- Dùng cache `data/cache/` thay vì tự chạy? Không <<xác nhận>>.
- Lỗi / bug script đã gặp: lỗi `NP_SUPPORTED_MODULES` nêu trên; CLIP phải tự cài khi chạy YOLOE lần đầu trên Colab (có cảnh báo restart runtime nhưng vẫn chạy ra kết quả).
- Một điều bạn làm khác lần sau: <<tự viết>>