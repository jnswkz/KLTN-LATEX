# Sườn tổng quan Khóa luận tốt nghiệp

> **Đề tài:** Nghiên cứu phát hiện mối đe dọa tại tác nhân trong hệ thống giám sát và ứng phó điểm cuối trên Windows bằng liên kết hành vi và trí tuệ nhân tạo
> *(Agent-side Threat Detection in Windows EDR using Behavioral Correlation and Artificial Intelligence)*
>
> **Mục đích file này:** cung cấp *bộ khung* để tự viết — mỗi mục chỉ ghi "cần trình bày cái gì", không viết sẵn nội dung. Kèm danh mục hình/sơ đồ/bảng đề xuất ở cuối.
> Khung chương bám theo template `chapters/chapter1–5.tex`.

---

## CHƯƠNG 1 — TỔNG QUAN

**Vai trò:** giới thiệu khái quát, đặt vấn đề. Không đi sâu kỹ thuật (để Chương 2–3).

- **1.1 Lý do chọn đề tài**
  - Bối cảnh & tầm quan trọng: thị phần Windows, tỷ lệ bị tấn công, APT/ransomware/fileless/LOLBins.
  - Hạn chế giải pháp hiện tại: AV/EDR chữ ký tĩnh thất bại; mô hình EDR tập trung (Cloud/SIEM) → 3 điểm yếu: độ trễ cao, quá tải băng thông, mất phòng vệ khi cô lập mạng.
  - Động lực: dịch chuyển năng lực phát hiện về phía tác tử (agent-side) + AI biên + đồ thị phả hệ.
  - *→ cần citation thật (UNICORN, ATLAS, ShadeWatcher, MITRE, Sigma).* 
- **1.2 Mục tiêu đề tài**
  - 1.2.1 Tổng quát: 1 đoạn — tác tử EDR tự hành dạng Windows Service.
  - 1.2.2 Cụ thể: 4 mục tiêu (telemetry+graph / rule engine / AI engine / phản ứng đa tầng + đóng gói).
- **1.3 Đối tượng và phạm vi nghiên cứu**
  - 1.3.1 Đối tượng: Windows Internals, provenance graph, Sigma, ONNX Runtime, mô hình ML/DL.
  - 1.3.2 Phạm vi: chỉ Windows 10/11/Server; 4 nhóm sự kiện Process/File/Registry/Network; dạng Windows Service.
- **1.4 Phương pháp nghiên cứu**
  - Khái quát cách tiếp cận: kế thừa UNICORN/ATLAS/ShadeWatcher; kiến trúc 3 tầng; pipeline C++/Python; đo F1/FPR/MTTD/CPU-RAM. (*Chi tiết để Chương 3.*)
- **1.5 Cấu trúc khóa luận**
  - Giới thiệu 1–2 câu cho mỗi chương 1→5.

**Lưu ý tách nội dung:** phần "Bài toán (Input/Output)" trong đề cương KHÔNG viết ở Chương 1 → đưa vào mục "Phát biểu bài toán" của Chương 3.

---

## CHƯƠNG 2 — CƠ SỞ LÝ THUYẾT VÀ CÁC NGHIÊN CỨU LIÊN QUAN

**Vai trò:** cung cấp nền tảng để hiểu Chương 3; tổng thuật & chỉ ra khoảng trống.

- **2.1 Cơ sở lý thuyết**
  - 2.1.1 Khái niệm liên quan:
    - EDR và mô hình điểm cuối; APT; tấn công fileless; LOLBins.
    - ETW (Event Tracing for Windows) & Sysmon; 4 nhóm sự kiện nhân.
    - Đồ thị nhân quả / phả hệ tiến trình (provenance graph), partial ordering.
    - MITRE ATT&CK; định dạng luật Sigma.
  - 2.1.2 Các kỹ thuật nền tảng:
    - Kỹ thuật tấn công: Process Hollowing, DLL Injection, Thread Hijacking, persistence qua Registry Run Keys.
    - Học máy: cây tăng cường (LightGBM, GOSS, leaf-wise), đặc trưng hành vi, chuẩn hóa min–max/log.
    - Triển khai mô hình: ONNX, lượng tử hóa INT8, ONNX Runtime C++.
- **2.2 Các nghiên cứu liên quan**
  - 2.2.1 Nghiên cứu trong nước (nếu có — tìm thêm).
  - 2.2.2 Nghiên cứu ngoài nước: UNICORN, ATLAS, ShadeWatcher, Windows Internals; mỗi công trình ghi: ý tưởng, ưu điểm, hạn chế khi áp dụng trên điểm cuối Windows.
- **2.3 Nhận xét**
  - Bảng so sánh các hướng tiếp cận; chỉ ra khoảng trống (Linux-centric, hồi tố thay vì chặn real-time, GNN nặng tài nguyên, rời rạc luật/AI) → định vị hướng đi của khóa luận.

---

## CHƯƠNG 3 — PHƯƠNG PHÁP ĐỀ XUẤT

**Vai trò:** phần lõi — mô tả kiến trúc & thuật toán.

- **3.1 Phát biểu bài toán**
  - Input: luồng sự kiện ETW/Sysmon thô (PID/PPID, ngữ cảnh thực thi, command-line, quyền, vector đặc trưng 16 chiều).
  - Output: điểm nguy hiểm tổng hợp 3 tầng, 5 mức cảnh báo (BENIGN→CRITICAL), hành động phản ứng.
- **3.2 Mô hình/Kiến trúc tổng quan**
  - Sơ đồ luồng dữ liệu end-to-end; mô tả kiến trúc 3 tầng.
- **3.3 Chi tiết phương pháp** (mỗi thành phần 1 subsection)
  - 3.3.1 Tầng nền tảng — ETW Telemetry Collector + In-Memory Provenance Graph Engine: đăng ký ETW, Normalizer (UUID, nhãn hành vi, làm giàu), Ring Buffer + SQLite, xây đồ thị có hướng, cơ chế cắt tỉa bộ nhớ, độ phức tạp O(R×|E|).
  - 3.3.2 Tầng phân tích — Rule Engine: Sigma → AST → sinh mã C++ tĩnh, `std::string_view`, whitelist ràng buộc kép (tên + đường dẫn chuẩn), 3 trạng thái (Clean/Critical/Unknown).
  - 3.3.3 Tầng phân tích — Edge AI Engine: 16 đặc trưng (cửa sổ trượt, command-line, ngữ cảnh, đường dẫn/hành vi), chuẩn hóa, huấn luyện LightGBM + Optuna, xuất ONNX + INT8, suy diễn MLInferenceEngine.
  - 3.3.4 Tầng hợp nhất quyết định — Decision Fusion: tính `S_base`, điểm phạt đồ thị `P_graph`, khuếch đại `S_final` khi nhiều dấu hiệu, ánh xạ 5 mức.
  - 3.3.5 Cơ chế phản ứng an toàn đa tầng (LocalResponse): tiêu diệt tiến trình, cắt chuỗi lây nhiễm phả hệ, cách ly mạng tạm thời, log SQLite + đồng bộ bất tuần tự.
- **3.4 Độ phức tạp và phân tích**
  - Phân tích chi phí tính toán/bộ nhớ; chứng minh đạt ngưỡng tài nguyên (CPU <3%, RAM <150MB).

---

## CHƯƠNG 4 — THỰC NGHIỆM VÀ ĐÁNH GIÁ

- **4.1 Dữ liệu và môi trường thực nghiệm**
  - 4.1.1 Bộ dữ liệu: DARPA TC (THEIA), OpTC (dự phòng); dữ liệu Benign + Atomic Red Team; tỷ lệ 80/20.
  - 4.1.2 Cấu hình thực nghiệm: máy ảo Windows 10/11, cấu hình phần cứng, phiên bản công cụ.
- **4.2 Độ đo đánh giá**
  - Định nghĩa từng độ đo: Precision/Recall/F1/ROC-AUC, FPR, MTTD/Latency, CPU/RAM/Disk I/O, ETW drop rate; kèm mục tiêu định lượng.
- **4.3 Kết quả thực nghiệm**
  - Kết quả phát hiện theo kỹ thuật MITRE; so sánh với baseline (Sequence Autoencoder / Quantized GNN); đo tài nguyên & độ trễ.
- **4.4 Phân tích và thảo luận**
  - Phân tích case tiêu biểu; lý giải FP/FN; đánh giá so với mục tiêu; điểm mạnh/yếu.

---

## CHƯƠNG 5 — KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN

- **5.1 Kết luận:** đối chiếu kết quả với mục tiêu Chương 1.
- **5.2 Hạn chế:** phạm vi, dữ liệu, kỹ thuật chưa xử lý.
- **5.3 Hướng phát triển:** mở rộng nền tảng, kỹ thuật tấn công mới, tích hợp SIEM, học liên tục.

---

## DANH MỤC HÌNH / SƠ ĐỒ / DIAGRAM ĐỀ XUẤT

| # | Tên hình (dự kiến) | Loại | Mô tả / dùng để minh họa | Chương |
|---|---|---|---|---|
| H1 | Mô hình EDR tập trung và các điểm nghẽn | Sơ đồ khối | Minh họa 3 hạn chế (độ trễ, băng thông, mất phòng vệ) → đặt vấn đề | 1 |
| H2 | So sánh EDR tập trung vs tác tử tự hành | Sơ đồ đối sánh | Làm nổi bật động lực dịch chuyển về agent-side | 1 |
| H3 | Kiến trúc ETW/Sysmon & 4 nhóm sự kiện nhân | Sơ đồ phân lớp | Nền tảng thu thập telemetry | 2 |
| H4 | Ví dụ đồ thị phả hệ tiến trình (provenance graph) | Đồ thị có hướng | Giải thích khái niệm nhân quả, quan hệ cha–con | 2 |
| H5 | Minh họa kỹ thuật Process Hollowing / DLL Injection | Sơ đồ bộ nhớ | Giải thích tấn công can thiệp bộ nhớ ngầm | 2 |
| H6 | Dòng đời luật Sigma → phát hiện | Sơ đồ quy trình | Bối cảnh cho Rule Engine ở Chương 3 | 2 |
| H7 | **Sơ đồ luồng dữ liệu tổng quan** (end-to-end) | Data-flow diagram | Bản vẽ trung tâm — từ Kernel → Collector → Graph → 2 engine → Fusion → Response (có sẵn trong đề cương) | 3 |
| H8 | Kiến trúc 3 tầng của tác tử | Sơ đồ khối phân tầng | Nền tảng / phân tích song song / hợp nhất quyết định | 3 |
| H9 | Pipeline chuẩn hóa & xây đồ thị trong Ring Buffer | Sơ đồ khối | Chi tiết Normalizer + Ring Buffer + SQLite | 3 |
| H10 | Quy trình biên dịch Sigma → AST → mã C++ | Sơ đồ/cây AST | Minh họa Rule Engine | 3 |
| H11 | Cơ chế whitelist ràng buộc kép & 3 trạng thái | Sơ đồ quyết định | Clean/Critical/Unknown | 3 |
| H12 | Vector đặc trưng 16 chiều & nhóm đặc trưng | Sơ đồ/bảng hình | Trực quan hóa đầu vào AI | 3 |
| H13 | Pipeline huấn luyện → ONNX → INT8 → Agent | Sơ đồ quy trình | Vòng đời mô hình AI | 3 |
| H14 | Lược đồ chấm điểm hợp nhất 3 tầng (S_base→S_final) | Flowchart | Decision Fusion + 5 mức cảnh báo | 3 |
| H15 | Cơ chế phản ứng đa tầng & cắt chuỗi phả hệ | Sơ đồ | LocalResponse (kill, cô lập, cắt nhánh) | 3 |
| H16 | Sơ đồ môi trường thực nghiệm | Sơ đồ hạ tầng | Máy ảo, Atomic Red Team, thu thập | 4 |
| H17 | Đường cong ROC / PR | Biểu đồ | Năng lực phát hiện | 4 |
| H18 | Biểu đồ cột F1 theo kỹ thuật MITRE ATT&CK | Bar chart | So sánh hiệu quả theo kỹ thuật | 4 |
| H19 | Biểu đồ CPU/RAM theo thời gian khi chạy ngầm | Line chart | Chứng minh ngưỡng tài nguyên | 4 |
| H20 | Biểu đồ so sánh với baseline | Bar/line chart | Agent vs Autoencoder/GNN | 4 |
| H21 | Ví dụ đồ thị tấn công được phát hiện (case study) | Đồ thị có hướng | Phân tích định tính ở 4.4 | 4 |

## DANH MỤC BẢNG ĐỀ XUẤT

| # | Tên bảng (dự kiến) | Mô tả / nội dung | Chương |
|---|---|---|---|
| B1 | So sánh các nghiên cứu liên quan | UNICORN/ATLAS/ShadeWatcher… theo tiêu chí: nền tảng, real-time, tài nguyên, hạn chế | 2 |
| B2 | 4 nhóm sự kiện nhân & trường thu thập | Process/File/Registry/Network và chi tiết sự kiện | 2 hoặc 3 |
| B3 | Danh sách 16 đặc trưng hành vi | Tên, ý nghĩa, cách chuẩn hóa | 3 |
| B4 | Ánh xạ điểm số → 5 mức cảnh báo | Khoảng [0,1] ↔ BENIGN/LOW/MEDIUM/HIGH/CRITICAL | 3 |
| B5 | Tập luật Sigma được sử dụng | Kỹ thuật ATT&CK ↔ luật ↔ hành động | 3 |
| B6 | Cấu hình phần cứng/phần mềm thực nghiệm | VM, OS, phiên bản công cụ | 4 |
| B7 | Thống kê bộ dữ liệu | Số sự kiện, tỷ lệ Benign/Malware, Train/Test | 4 |
| B8 | Bộ chỉ số đánh giá & mục tiêu định lượng | (lấy từ bảng mục 8 đề cương: F1>0.92, FPR<0.5%, MTTD<100ms, CPU<3%, RAM<150MB, drop<0.1%) | 4 |
| B9 | Kết quả phát hiện tổng hợp | Precision/Recall/F1/ROC-AUC/FPR | 4 |
| B10 | Kết quả đo tài nguyên & độ trễ | CPU/RAM/Disk I/O, MTTD, drop rate | 4 |
| B11 | So sánh với baseline | Agent vs Autoencoder/Quantized GNN | 4 |

---

## GHI CHÚ KHI TRIỂN KHAI

- **Tài liệu tham khảo:** thêm 6 mục của đề cương vào `references.bib` (UNICORN, ATLAS, ShadeWatcher, Windows Internals, MITRE ATT&CK, Sigma) và thay citation placeholder.
- **Hình trong LaTeX:** dùng mẫu `\begin{figure}[H] … \includegraphics … \caption … \label \end{figure}` (đã có comment gợi ý trong `chapter3.tex`, `chapter4.tex`).
- **Đánh số:** mỗi hình/bảng cần có `\label` + `\caption` và được tham chiếu ít nhất 1 lần trong thân bài.
- **Cân đối:** Chương 3 là trọng tâm (nhiều hình nhất); Chương 1 nên gọn (1–2 hình là đủ).
