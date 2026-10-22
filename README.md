# KLTN-LATEX — Mẫu luận văn tốt nghiệp (UIT)

Repo này tự động **biên dịch LaTeX trên GitHub Actions** và **xuất bản bản xem trước
PDF lên GitHub Pages** mỗi khi push lên nhánh `main`.

## Cấu trúc thư mục

| Đường dẫn | Nội dung |
|---|---|
| `main.tex` | File chính, `\input` tất cả các chương |
| `chapters/` | Nội dung từng chương (`chapter1.tex` … `chapter5.tex`) |
| `misc/` | Trang bìa, lời cảm ơn, từ viết tắt, tóm tắt, hội đồng, logo |
| `appendix/` | Phụ lục |
| `references.bib` | Tài liệu tham khảo (Biber) |
| `uit-thesis.sty` | Gói định dạng của trường |
| `web/index.html` | Trang preview nhúng PDF (dùng cho GitHub Pages) |
| `.github/workflows/latex.yml` | Pipeline CI biên dịch + deploy |

> Thêm chương mới: xem [HUONG-DAN-THEM-CHUONG.md](HUONG-DAN-THEM-CHUONG.md).

## Cách biên dịch tại máy (tùy chọn)

Cần MiKTeX/TeX Live + `latexmk` + `biber`:

```bash
latexmk -pdf -outdir=build main.tex
```

Hoặc dùng VS Code + extension *LaTeX Workshop* (đã cấu hình sẵn trong `.vscode/settings.json`).

## CI/CD hoạt động thế nào

Workflow `.github/workflows/latex.yml`:

1. **Job `build`** — checkout, biên dịch `main.tex` bằng `xu-cheng/latex-action`
   (TeX Live full, `latexmk` tự chạy đủ pass + `biber`), gom `main.pdf` và
   `web/index.html` vào `public/`, tải `main.pdf` làm artifact của run.
2. **Job `deploy`** — xuất bản thư mục `public/` lên GitHub Pages (chỉ khi push `main`).

Pull request sẽ chỉ build để kiểm tra, không deploy.

## Thiết lập lần đầu

```bash
git init
git add .
git commit -m "Initial thesis + CI"
# tạo repo GitHub rồi:
git remote add origin https://github.com/<user>/<repo>.git
git branch -M main
git push -u origin main
```

Sau đó trên GitHub:

1. Vào **Settings → Pages**.
2. Mục **Build and deployment → Source** chọn **GitHub Actions**.
3. Push (hoặc chạy lại workflow). Khi job xanh, bản xem trước có tại:
   `https://<user>.github.io/<repo>/`

## Ghi chú

- Engine mặc định là **pdflatex** (khớp cấu hình local), hỗ trợ tiếng Việt qua
  `fontenc[T1,T5]`. Không nên đổi sang xelatex/lualatex để tránh khác biệt render.
- Thư mục `build/` và các file trung gian không được commit (xem `.gitignore`);
  CI tự sinh lại `main.pdf`.
