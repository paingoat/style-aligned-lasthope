# Hướng dẫn cài đặt môi trường (Style Aligned)

Tài liệu này bổ sung [README](../README.md) ở thư mục gốc: giúp lặp lại **môi trường gốc được mô tả** (Python **3.11**, PyTorch **2.1**, Diffusers trong khoảng tương thích với SDXL) và cài đủ dependency cho notebook cùng các script demo, **không cần chỉnh sửa mã nguồn** trong repo.

Phần dưới ưu tiên **Miniconda + Linux**; cuối tài liệu có ghi chú ngắn cho Windows.

---

## 1. Phần cứng và hệ điều hành

- **GPU NVIDIA** với VRAM đủ cho SDXL (thực tế thường từ khoảng 8 GB trở lên, tùy pipeline và tối ưu hóa).
- Các notebook và demo gọi `.to("cuda")` nên cần CUDA hoạt động với PyTorch.

---

## 2. Cài Miniconda (Linux x86_64)

Trên máy chủ / máy ảo Linux, làm mới shell và tải bộ cài từ thư mục home:

```bash
cd ~
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Trong wizard: có thể chấp nhận mặc định hoặc chỉnh đường dẫn cài đặt. Khi được hỏi **“conda init”**, nên chọn **yes** để shell tự nạp `conda`.

Nạp cấu hình shell (chọn một, tùy shell bạn dùng):

```bash
source ~/.bashrc
# hoặc: source ~/.zshrc
```

Kiểm tra:

```bash
conda --version
```

---

## 3. Tạo môi trường Conda Python 3.11

README ghi code được thử với **Python 3.11**. Tạo env riêng cho dự án (tên ví dụ `stylealigned`):

```bash
conda create -n stylealigned python=3.11 -y
conda activate stylealigned
python --version
```

Mọi lệnh `pip` / `python` trong các bước sau nên chạy **sau** `conda activate stylealigned`.

---

## 4. PyTorch 2.1 + CUDA

File `requirements.txt` **không** khai báo `torch` (tránh pip cài nhầm bản CPU). Cài **PyTorch 2.1** trùng CUDA máy bạn theo trang chính thức:

[https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)

Ví dụ tham khảo (Linux, CUDA 11.8) — **chỉ dùng lệnh do trang PyTorch sinh ra cho máy bạn**:

```bash
pip install torch==2.1.2 torchvision==0.16.2 torchaudio==2.1.2 --index-url https://download.pytorch.org/whl/cu118
```

Kiểm tra GPU:

```bash
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

Kỳ vọng: `2.1.x` và `True` nếu driver/CUDA hợp lệ.

---

## 5. Cài dependency từ `requirements.txt`

Clone hoặc `cd` vào thư mục gốc repo (cùng cấp với `requirements.txt`):

```bash
pip install -U pip
pip install -r requirements.txt
```

**Ghi chú Diffusers:** README nêu *Diffusers 0.16* theo thời điểm cũ; pipeline **SDXL** cần **diffusers ≥ 0.21**. File `requirements.txt` đã phản ánh điều đó.

---

## 6. `HF_TOKEN`, cache và file `.env` (không đổi code repo)

`diffusers` / `transformers` / `huggingface_hub` đọc **biến môi trường của process** (`os.environ`). Chúng **không** tự mở file `.env`; vì vậy bạn chỉ cần đảm bảo biến được nạp **trước** khi chạy Python hoặc Jupyter — **không cần sửa notebook hay script** trong repo.

### 6.1 Tên biến chuẩn của Hugging Face


| Mục đích                                                    | Biến nên dùng (khuyến nghị)              |
| ----------------------------------------------------------- | ---------------------------------------- |
| Token (repo gated, rate limit, v.v.)                        | `HF_TOKEN` hoặc `HUGGING_FACE_HUB_TOKEN` |
| Thư mục cache Hub (model, snapshot)                         | `HF_HUB_CACHE`                           |
| Thư mục “gốc” config + cache HF (tùy chọn thay cho chỉ hub) | `HF_HOME`                                |


**Lưu ý:** tên `**HF_CACHE` không phải** biến chuẩn mà `huggingface_hub` đọc. Nếu bạn muốn một key trong file `.env` cho dễ nhớ, hãy đặt **đúng tên chuẩn** trong `.env` (ví dụ `HF_HUB_CACHE=...`) để khi `source` file, mọi thứ hoạt động ngay.

Ví dụ file `.env` đặt trong thư mục repo (và **đưa `.env` vào `.gitignore`** cá nhân để không commit token):

```bash
# .env — ví dụ (không có khoảng trắng quanh dấu =)
HF_TOKEN=hf_xxxxxxxxxxxxxxxxxxxx
HF_HUB_CACHE=/path/to/large/disk/huggingface/hub
```

Nếu bạn chỉ cần một thư mục gốc cho toàn bộ dữ liệu HF, có thể thay bằng:

```bash
HF_TOKEN=hf_xxx
HF_HOME=/path/to/large/disk/huggingface
```

### 6.2 Cách nạp `.env` mà không đụng tới code

**Cách A — Nạp trong shell trước khi mở Jupyter (đơn giản):**

```bash
conda activate stylealigned
set -a
source /đường/dẫn/tới/style-aligned/.env
set +a
jupyter lab
# hoặc: jupyter notebook
```

(`set -a` khiến mọi biến được gán trong `.env` đều được `export`.)

**Cách B — Gắn biến vào env Conda (bền, mỗi lần `conda activate` là có):**

```bash
conda activate stylealigned
conda env config vars set HF_TOKEN="hf_xxx" HF_HUB_CACHE="/path/to/hub"
conda deactivate
conda activate stylealigned
```

Kiểm tra:

```bash
conda env config vars list
```

Cách này không cần file `.env` nếu bạn chấp nhận lưu đường dẫn/token trong cấu hình conda (vẫn nên bảo vệ quyền truy cập máy).

**Cách C — Chỉ dùng CLI HF (token):** tương đương một phần `HF_TOKEN`:

```bash
huggingface-cli login
```

Cache vẫn tuân theo `HF_HOME` / `HF_HUB_CACHE` nếu bạn đã set.

Khi các biến đã có trong môi trường, **chức năng chính của repo không đổi**; bạn chỉ điều hướng nơi lưu cache và xác thực khi cần.

---

## 7. Đăng ký kernel Jupyter cho env Conda

Cài `ipykernel` trong env (đã có trong `requirements.txt` nhờ gói `jupyter`; nếu thiếu):

```bash
conda activate stylealigned
pip install ipykernel
python -m ipykernel install --user --name stylealigned --display-name "Python 3.11 (stylealigned)"
```

Trong Jupyter / VS Code / Cursor: chọn kernel **“Python 3.11 (stylealigned)”**. Kernel kế thừa biến môi trường của process cha — nếu bạn mở Jupyter từ shell đã `source .env` hoặc đã cấu hình `conda env config vars`, notebook sẽ thấy `HF_TOKEN` và đường cache.

Liệt kê kernel đã đăng ký:

```bash
jupyter kernelspec list
```

Gỡ kernel (khi cần):

```bash
jupyter kernelspec uninstall stylealigned
```

---

## 8. Chạy notebook

```bash
conda activate stylealigned
# tùy chọn: nạp .env như mục 6.2
jupyter lab
```

Mở `style_aligned_sdxl.ipynb`, `style_aligned_transfer_sdxl.ipynb`, v.v.

---

## 9. Chạy demo Gradio (tùy chọn)

```bash
conda activate stylealigned
python demo_stylealigned_sdxl.py
```

Các file `demo_stylealigned_controlnet.py`, `demo_stylealigned_multidiffusion.py` cần thêm VRAM và model tương ứng.

---

## 10. Xử lý sự cố thường gặp


| Hiện tượng                                       | Hướng xử lý                                                                                                                         |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| `torch.cuda.is_available()` là `False`           | Cài lại PyTorch đúng bản **CUDA**; cập nhật driver NVIDIA.                                                                          |
| Lỗi thiếu symbol / không tương thích `diffusers` | Cài `requirements.txt` trong đúng env `stylealigned` đã có `torch`.                                                                 |
| Notebook không thấy token / cache                | Kernel phải là env đã set biến; thử `import os; print(os.environ.get("HF_TOKEN"), os.environ.get("HF_HUB_CACHE"))` trong một ô tạm. |
| OOM khi generate                                 | Giảm batch, dùng offload/VAE slicing như trong `demo_stylealigned_sdxl.py` (tùy chỉnh cục bộ, không bắt buộc sửa upstream).         |


---

## 11. Checklist tóm tắt

1. Cài Miniconda, `source ~/.bashrc` (hoặc `~/.zshrc`), `conda create -n stylealigned python=3.11`.
2. `conda activate stylealigned`, cài PyTorch 2.1 + CUDA từ pytorch.org.
3. `pip install -r requirements.txt` trong thư mục repo.
4. (Tùy chọn) Tạo `.env` với `HF_TOKEN` và `HF_HUB_CACHE` / `HF_HOME`; nạp bằng shell hoặc `conda env config vars`.
5. `python -m ipykernel install ...`, mở Jupyter và chọn kernel **stylealigned**.

---

## 12. Windows (tùy chọn)

Tải installer: [Miniconda3-latest-Windows-x64.exe](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x64.exe), chạy wizard, mở **Anaconda Prompt**, sau đó các bước `conda create` / `conda activate` / `pip` / `ipykernel` tương tự Linux. Nạp biến từ `.env` trong PowerShell có thể dùng lặp từng dòng `setx` hoặc công cụ bạn tin cậy; cách ổn định nhất trên Windows cũng là `conda env config vars set ...`.

Như vậy bạn giữ nguyên codebase gốc, chỉ dùng `requirements.txt` và tài liệu trong `doc/` để tái lập môi trường.