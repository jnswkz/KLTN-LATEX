# Hướng dẫn thêm chương cho khóa luận

Tài liệu này hướng dẫn cách thêm một chương mới vào khóa luận (mẫu LaTeX UIT).

## 1. Cấu trúc thư mục liên quan

```
KLTN-LATEX/
├── main.tex              # File chính, nơi "input" tất cả các chương
├── chapters/             # Chứa nội dung từng chương
│   ├── chapter1.tex      # Chương 1: TỔNG QUAN
│   ├── chapter2.tex      # Chương 2: CƠ SỞ LÝ THUYẾT VÀ CÁC NGHIÊN CỨU LIÊN QUAN
│   ├── chapter3.tex      # Chương 3: PHƯƠNG PHÁP ĐỀ XUẤT
│   ├── chapter4.tex      # Chương 4: THỰC NGHIỆM VÀ ĐÁNH GIÁ
│   └── chapter5.tex      # Chương 5: KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN
├── references.bib        # Tài liệu tham khảo (BibTeX/Biber)
└── misc/                 # Trang bìa, lời cảm ơn, từ viết tắt, hình ảnh...
```

> Số thứ tự chương **tự động sinh** theo thứ tự lệnh `\input` trong `main.tex`, không phải theo tên file. Lệnh `\chapter{...}` chỉ quyết định **tiêu đề**, còn số "Chương N" do LaTeX đếm.

## 2. Các bước thêm một chương mới

### Bước 1 — Tạo file chương
Tạo file `chapters/chapter6.tex` (đổi số theo nhu cầu) với nội dung mẫu:

```latex
\chapter{TÊN CHƯƠNG MỚI}
\label{chap:tenchuong}

\section{Tiêu đề mục 1}
\subsection{Tiêu đề mục con 1.1}

\section{Tiêu đề mục 2}
```

### Bước 2 — Khai báo trong `main.tex`
Mở `main.tex`, tìm khối các lệnh `\input{chapters/...}` và thêm dòng mới **đúng vị trí** bạn muốn chương xuất hiện:

```latex
\input{chapters/chapter1.tex}
\input{chapters/chapter2.tex}
\input{chapters/chapter3.tex}
\input{chapters/chapter4.tex}
\input{chapters/chapter5.tex}
\input{chapters/chapter6.tex}   % <-- chương mới thêm vào
```

### Bước 3 — Biên dịch lại
Biên dịch `main.tex`. Vì có trích dẫn (biber) nên chạy đủ chuỗi lệnh:

```bash
latexmk -pdf -outdir=build main.tex
```

Hoặc thủ công:

```bash
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
```

> Nếu dùng VS Code + LaTeX Workshop, chỉ cần lưu file — cấu hình trong `.vscode/settings.json` sẽ tự biên dịch vào thư mục `build/`.

## 3. Các cú pháp thường dùng trong chương

| Mục đích | Cú pháp |
|---|---|
| Tiêu đề chương | `\chapter{...}` |
| Mục / mục con / mục con con | `\section{}` · `\subsection{}` · `\subsubsection{}` |
| Nhãn để tham chiếu | `\label{chap:ten}` → tham chiếu bằng `\ref{chap:ten}` |
| Trích dẫn tài liệu | `\cite{khoa_bibtex}` (khóa khai báo trong `references.bib`) |
| Từ viết tắt | `\ac{UIT}` (khai báo trong `misc/abbreviation.tex`) |

### Chèn hình ảnh
```latex
\begin{figure}[H]
    \centering
    \includegraphics[width=0.8\textwidth]{misc/ten_hinh.pdf}
    \caption{Chú thích hình.}
    \label{fig:tenhinh}
\end{figure}
```

### Chèn bảng
```latex
\begin{table}[H]
    \centering
    \caption{Chú thích bảng.}
    \label{tab:tenbang}
    \begin{tabular}{lcc}
        \hline
        Cột A & Cột B & Cột C \\
        \hline
        ... & ... & ... \\
        \hline
    \end{tabular}
\end{table}
```

## 4. Thêm tài liệu tham khảo
Mở `references.bib`, thêm một entry, ví dụ:

```bibtex
@article{tac_gia_2024,
  title   = {Tiêu đề bài báo},
  author  = {Nguyen, Van A and Tran, Thi B},
  journal = {Tên tạp chí},
  year    = {2024}
}
```

Sau đó dùng `\cite{tac_gia_2024}` trong chương. Nhớ chạy `biber` khi biên dịch.

## 5. Mẹo & lưu ý

- **Thứ tự `\input` = thứ tự chương.** Muốn đổi vị trí chương, chỉ cần đổi thứ tự dòng `\input`.
- **Xóa chương:** comment (`%`) hoặc xóa dòng `\input` tương ứng; các chương sau tự đánh số lại.
- **Đặt tên file nhất quán** (`chapterN.tex`) để dễ quản lý.
- Tiêu đề chương theo quy chuẩn thường viết **IN HOA**.
- Nếu lỗi biên dịch, xóa thư mục `build/` rồi biên dịch lại sạch.
