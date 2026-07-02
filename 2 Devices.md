# Training2601 — Labeling 2 Device (PC + Laptop) sampai Training

**Device yang dipakai:**
- PC (utama): user `bmrt`, hostname `bmrt-desktop`
- Laptop Acer Nitro: user `rein`, hostname `tewks`

Total foto: 5273 | Classes: `bh`, `bm`, `sbox`, `ubox`, `bb`

Semua kode Python (`import`, `from`, `for`, dll) **wajib** ditulis lewat `nano` jadi file `.py` dulu, baru dijalankan pakai `python3 namafile.py`. Perintah biasa (`cp`, `mv`, `ls`, `mkdir`) bisa langsung diketik di terminal.

---

## Tahap 1 — Di PC: Pisahkan Foto yang Sudah vs Belum Dilabel

Ini buat mastiin foto yang udah kamu kerjain sebelumnya nggak ke-mix ulang.

### 1. Buat file script
```bash
nano ~/Training2601/split_labeled.py
```

### 2. Isi dengan kode ini
```python
import shutil
from pathlib import Path

SRC = Path.home() / "Training2601/raw_images"
DONE = Path.home() / "Training2601/sudah_label"
BELUM = Path.home() / "Training2601/belum_label"
DONE.mkdir(exist_ok=True)
BELUM.mkdir(exist_ok=True)

imgs = sorted(SRC.glob("*.jpg"))
done_count = belum_count = 0

for img in imgs:
    label = img.with_suffix(".txt")
    if label.exists():
        shutil.copy(img, DONE / img.name)
        shutil.copy(label, DONE / label.name)
        done_count += 1
    else:
        shutil.copy(img, BELUM / img.name)
        belum_count += 1

print(f"Sudah dilabel: {done_count}")
print(f"Belum dilabel: {belum_count}")
```

### 3. Simpan (`Ctrl+O` → Enter) lalu keluar (`Ctrl+X`)

### 4. Jalankan
```bash
python3 ~/Training2601/split_labeled.py
```

---

## Tahap 2 — Di PC: Bagi Folder `belum_label/` Jadi 2 Bagian

### 1. Buat file script
```bash
nano ~/Training2601/split_to_laptop.py
```

### 2. Isi dengan kode ini
```python
import shutil
from pathlib import Path

SRC = Path.home() / "Training2601/belum_label"
PC  = Path.home() / "Training2601/belum_label_PC"
LAPTOP = Path.home() / "Training2601/belum_label_LAPTOP"
PC.mkdir(exist_ok=True)
LAPTOP.mkdir(exist_ok=True)

imgs = sorted(SRC.glob("*.jpg"))
half = len(imgs) // 2

for img in imgs[:half]:
    shutil.copy(img, PC / img.name)
for img in imgs[half:]:
    shutil.copy(img, LAPTOP / img.name)

print(f"PC lanjut: {half} foto")
print(f"Laptop kerjain: {len(imgs)-half} foto")
```

### 3. Simpan lalu keluar (sama seperti Tahap 1)

### 4. Jalankan
```bash
python3 ~/Training2601/split_to_laptop.py
```

### 5. Copy `classes.txt` ke kedua folder baru (perintah biasa, langsung di terminal)
```bash
cp ~/Training2601/raw_images/classes.txt ~/Training2601/belum_label_PC/
cp ~/Training2601/raw_images/classes.txt ~/Training2601/belum_label_LAPTOP/
```

---

## Tahap 3 — Siapkan Folder Tujuan di Laptop Acer Nitro

Di terminal **laptop** (`rein@tewks`):
```bash
mkdir -p ~/Training2601
```

---

## Tahap 4 — Pindahin Folder `belum_label_LAPTOP/` ke Laptop (via Flashdisk)

### Di PC:
1. Colok flashdisk ke PC.
2. Copy folder `~/Training2601/belum_label_LAPTOP/` ke flashdisk (drag & drop di file manager, atau lewat terminal):
```bash
# cek dulu flashdisk ke-mount di mana
lsblk
```
```bash
cp -r ~/Training2601/belum_label_LAPTOP /media/bmrt/NAMA_FLASHDISK/
```

### Di Laptop (`rein@tewks`):
1. Colok flashdisk yang sama ke laptop.
2. Cek flashdisk ke-mount di mana:
```bash
lsblk
```
3. Copy folder dari flashdisk ke laptop:
```bash
cp -r /media/rein/NAMA_FLASHDISK/belum_label_LAPTOP ~/Training2601/
```
4. Cek hasilnya sudah masuk dengan benar:
```bash
ls ~/Training2601/belum_label_LAPTOP/*.jpg | wc -l
ls ~/Training2601/belum_label_LAPTOP/classes.txt
```
Jumlah foto harus sama dengan angka "Laptop kerjain: XXXX foto" dari Tahap 2, dan `classes.txt` harus kebaca (bukan error).

---

## Tahap 5 — Install & Jalankan labelImg di Kedua Device

### Kalau labelImg belum ada di laptop, install dulu:
```bash
pip install labelImg --break-system-packages
```

### Di PC — kerjain folder `belum_label_PC`:
```bash
labelImg ~/Training2601/belum_label_PC ~/Training2601/belum_label_PC/classes.txt
```

### Di Laptop — kerjain folder `belum_label_LAPTOP`:
```bash
labelImg ~/Training2601/belum_label_LAPTOP ~/Training2601/belum_label_LAPTOP/classes.txt
```

### ⚠️ Wajib dicek tiap buka labelImg (di kedua device):
1. **Open Dir** → sudah otomatis kepilih dari command di atas
2. **Change Save Dir** → arahin ke folder yang **sama persis** (folder yang lagi dibuka)
3. Pastikan tombol format di kiri nunjukin **YOLO** (bukan PascalVOC)
4. Menu **View → Auto Save mode** harus dicentang (✓)

Shortcut biar cepat: `w` = buat box, `d` = next image, `a` = previous image.

---

## Tahap 6 — Setelah Selesai Label, Balikin Hasil dari Laptop ke PC

### Di Laptop (`rein@tewks`):
1. Colok flashdisk lagi.
2. Cek mount point:
```bash
lsblk
```
3. Copy folder `belum_label_LAPTOP` (yang sekarang sudah ada file `.txt`-nya) ke flashdisk:
```bash
cp -r ~/Training2601/belum_label_LAPTOP /media/rein/NAMA_FLASHDISK/
```

### Di PC (`bmrt@bmrt-desktop`):
1. Colok flashdisk yang sama.
2. Cek mount point:
```bash
lsblk
```
3. Copy folder itu balik ke PC, **replace** folder lama:
```bash
rm -rf ~/Training2601/belum_label_LAPTOP
cp -r /media/bmrt/NAMA_FLASHDISK/belum_label_LAPTOP ~/Training2601/
```
4. Cek jumlah label yang sudah masuk:
```bash
ls ~/Training2601/belum_label_LAPTOP/*.txt | wc -l
```
Harus sama dengan jumlah foto di folder itu (`*.jpg`).

---

## Tahap 7 — Gabung Semua Hasil Jadi Satu Folder Final (di PC)

### 1. Buat file script
```bash
nano ~/Training2601/merge_final.py
```

### 2. Isi dengan kode ini
```python
import shutil
from pathlib import Path

FOLDERS = [
    Path.home() / "Training2601/sudah_label",
    Path.home() / "Training2601/belum_label_PC",
    Path.home() / "Training2601/belum_label_LAPTOP",
]

FINAL = Path.home() / "Training2601/raw_images_final"
FINAL.mkdir(exist_ok=True)

count = 0
for folder in FOLDERS:
    for f in folder.glob("*.jpg"):
        shutil.copy(f, FINAL / f.name)
        label = f.with_suffix(".txt")
        if label.exists():
            shutil.copy(label, FINAL / label.name)
        count += 1

print(f"Total foto digabung: {count}")
```

### 3. Simpan lalu keluar

### 4. Jalankan
```bash
python3 ~/Training2601/merge_final.py
```

### 5. Cek hasil akhir — kedua angka ini harus sama-sama 5273
```bash
ls ~/Training2601/raw_images_final/*.jpg | wc -l
ls ~/Training2601/raw_images_final/*.txt | wc -l
```

---

## Tahap 8 — Cek Distribusi Kelas

```bash
cat ~/Training2601/raw_images_final/*.txt | awk '{print $1}' | sort | uniq -c
```

Ini nunjukin jumlah instance per kelas (`0=bh, 1=bm, 2=sbox, 3=ubox, 4=bb`). Kalau ada kelas yang jauh lebih sedikit dari yang lain, catat dulu — bisa jadi pertimbangan buat nambah foto kelas itu sebelum training.

---

## Tahap 9 — Split Dataset (Train / Val)

### 1. Buat file script
```bash
nano ~/Training2601/split_dataset.py
```

### 2. Isi dengan kode ini
```python
import shutil, random
from pathlib import Path

SRC  = Path.home() / "Training2601/raw_images_final"
OUT  = Path.home() / "Training2601/data_train"

imgs  = sorted(SRC.glob("*.jpg"))
valid = [f.stem for f in imgs if (SRC / (f.stem + ".txt")).exists()]
print(f"Total foto berlabel: {len(valid)}")

random.seed(42)
random.shuffle(valid)
n_train     = int(len(valid) * 0.90)
train_files = valid[:n_train]
val_files   = valid[n_train:]

def copy_files(file_list, split):
    for stem in file_list:
        shutil.copy(SRC / (stem + ".jpg"), OUT / f"images/{split}/")
        shutil.copy(SRC / (stem + ".txt"), OUT / f"labels/{split}/")

copy_files(train_files, "train")
copy_files(val_files,   "val")
print(f"Train : {len(train_files)} foto")
print(f"Val   : {len(val_files)} foto")
print("Split selesai!")
```

### 3. Pastikan folder tujuan sudah ada (kalau belum pernah dibuat)
```bash
mkdir -p ~/Training2601/data_train/images/train
mkdir -p ~/Training2601/data_train/images/val
mkdir -p ~/Training2601/data_train/labels/train
mkdir -p ~/Training2601/data_train/labels/val
```

### 4. Jalankan
```bash
python3 ~/Training2601/split_dataset.py
```

### 5. Cek hasilnya
```bash
ls ~/Training2601/data_train/images/train/ | wc -l
ls ~/Training2601/data_train/labels/train/ | wc -l
ls ~/Training2601/data_train/images/val/   | wc -l
ls ~/Training2601/data_train/labels/val/   | wc -l
```
Jumlah `images` dan `labels` di tiap split (train/val) harus **sama persis**.

---

## Tahap 10 — Buat `data.yaml`

```bash
cat > ~/Training2601/data_train/data.yaml << 'EOF'
path: /home/bmrt/Training2601/data_train
train: images/train
val:   images/val
nc: 5
names:
  0: bh
  1: bm
  2: sbox
  3: ubox
  4: bb
EOF
```

---

## Tahap 11 — Training

### 1. Buat file script
```bash
nano ~/Training2601/train.py
```

### 2. Isi dengan kode ini
```python
from ultralytics import YOLO
import torch

print(f"GPU: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU only'}")

model = YOLO("yolo26n.pt")

results = model.train(
    data        = "/home/bmrt/Training2601/data_train/data.yaml",
    epochs      = 100,
    patience    = 20,
    imgsz       = 640,
    batch       = 8,
    name        = "5cls_v1",
    project     = "/home/bmrt/Training2601/runs",
    device      = 0 if torch.cuda.is_available() else "cpu",
    save        = True,
    save_period = 10,
    hsv_s       = 0.7,
    hsv_v       = 0.4,
    hsv_h       = 0.015,
    fliplr      = 0.5,
    mosaic      = 1.0,
    verbose     = True
)

print(f"\nSelesai! Model terbaik:")
print(f"{results.save_dir}/weights/best.pt")
```

### 3. Simpan lalu keluar

### 4. Jalankan (ini bisa makan waktu lama — jam-an — tergantung ada GPU atau tidak)
```bash
python3 ~/Training2601/train.py
```

---

## Tahap 12 — Rename Model Final

Setelah training selesai:
```bash
mv ~/Training2601/runs/5cls_v1/weights/best.pt ~/Training2601/runs/5cls_v1/weights/5class26.pt
```

---

## Tahap 13 — Cek Hasil Training

```bash
cat ~/Training2601/runs/5cls_v1/results.csv
```

Atau lihat langsung file grafik & confusion matrix di:
```
~/Training2601/runs/5cls_v1/
```
(cari `results.png` dan `confusion_matrix.png`)

---

## Catatan Penting
- **NAMA_FLASHDISK** di Tahap 4 & 6 harus diganti sesuai nama flashdisk asli kamu (cek pakai `lsblk` sebelum copy).
- Ganti `bmrt` jadi `rein` kalau ada perintah yang dijalankan di laptop, bukan di PC (path `Path.home()` di Python otomatis menyesuaikan, tapi path absolute di `data.yaml` **harus** tetap ke PC karena training dijalankan di PC).
- Kalau ada kode Python yang di-paste langsung ke terminal (bukan lewat `nano` dulu), bakal muncul error `syntax error near unexpected token` — itu tandanya kamu lupa buat file `.py`-nya dulu.
