# Training2601 —Training YOLO (5 Kelas: bh, bm, sbox, ubox, bb)



Ganti `Training2601` di semua path kalau nama folderny beda, dan ganti `bmrt` dengan username Linux (cek pakai `whoami`).

---

## 1. Buat Folder

```bash
mkdir -p ~/Training2601/data_train/images/train
mkdir -p ~/Training2601/data_train/images/val
mkdir -p ~/Training2601/data_train/labels/train
mkdir -p ~/Training2601/data_train/labels/val
mkdir -p ~/Training2601/raw_images
```

**Penjelasan:**
- `raw_images/` → tempat foto mentah sebelum dilabel & displit.
- `data_train/images/train` & `images/val` → foto final setelah displit.
- `data_train/labels/train` & `labels/val` → file label `.txt` (format YOLO) hasil labeling, mengikuti nama file foto masing-masing.

---

## 2. Isi Foto (Manual)

**pindahin/copy foto-foto folder:**
```
~/Training2601/raw_images/
```

drag-drop lewat file manager (`nautilus ~/Training2601/raw_images/`), atau `cp` manual dari folder-folder lama.

**Catatan kalau foto dari banyak folder lama:**
- Kalau ada nama file yang sama persis dari folder berbeda (misal `frame_0001.jpg` muncul dua kali), salah satunya bisa ketiban/hilang. Rename dulu biar unik (bisa ditambahin prefix per sumber, misal `src1_frame_0001.jpg`).
- Kalau foto lama itu sudah ada label `.txt`-nya, ikutan dicopy juga ke `raw_images/` dengan nama yang sama persis (cuma beda extension), supaya nanti kebaca otomatis di Tahap 5 (split).
- Kalau class_id di label lama beda urutan dari class baru di bawah, itu label harus di-mapping ulang manual, karena kalau tidak, nanti nunjuk ke kelas yang salah.

cek jumlahnya:
```bash
ls ~/Training2601/raw_images/*.jpg | wc -l
```

---

## 3. Cek & Bersihkan Foto

Buka folder, buang foto yang blur/gelap/nggak jelas objeknya:
```bash
nautilus ~/Training2601/raw_images/
```

Cek sisa jumlah foto:
```bash
ls ~/Training2601/raw_images/*.jpg | wc -l
```

---

## 4. Buat File Classes

```bash
cat > ~/Training2601/raw_images/classes.txt << 'EOF'
bh
bm
sbox
ubox
bb
EOF
```

**Penjelasan:** urutan di file ini menentukan `class_id` saat labeling nanti:
- `0` = bh (bola hijau)
- `1` = bm (bola merah)
- `2` = sbox
- `3` = ubox
- `4` = bb (bola biru)

Urutan ini **harus sama persis** dengan urutan `names` di `data.yaml` nanti (Tahap 6).

---

## 5. Labeling (labelImg)

Jalankan labelImg, arahkan ke folder `raw_images/`, load `classes.txt`, lalu kotakin tiap objek dan pilih kelasnya satu-satu untuk semua foto.

```bash
labelImg ~/Training2601/raw_images ~/Training2601/raw_images/classes.txt
```

Hasilnya: tiap `namafoto.jpg` akan punya pasangan `namafoto.txt` berisi koordinat bounding box + class_id, disimpan di folder yang sama (`raw_images/`).

---

## 6. Split Dataset (Train / Val)

```bash
nano ~/Training2601/split_dataset.py
```

```python
import shutil, random
from pathlib import Path

SRC  = Path.home() / "Training2601/raw_images"
OUT  = Path.home() / "Training2601/data_train"

imgs  = sorted(SRC.glob("*.jpg"))
valid = [f.stem for f in imgs if (SRC / (f.stem + ".txt")).exists()]
print(f"Total foto berlabel: {len(valid)}")

random.seed(42)
random.shuffle(valid)
n_train     = int(len(valid) * 0.85)
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

Jalankan:
```bash
python3 ~/Training2601/split_dataset.py
```

Cek hasil split:
```bash
ls ~/Training2601/data_train/images/train/ | wc -l
ls ~/Training2601/data_train/labels/train/ | wc -l
ls ~/Training2601/data_train/images/val/   | wc -l
ls ~/Training2601/data_train/labels/val/   | wc -l
```

**Penjelasan:** rasio 85% train / 15% val itu standar untuk dataset kecil-menengah. Kalau foto berlabel kamu masih sedikit (< 200), pertimbangkan rasio 80/20 biar val set nggak terlalu kecil.

---

## 7. Buat `data.yaml`

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

**Penjelasan:** `path` harus absolute path, sesuaikan dengan lokasi foldermu (`whoami` buat cek username). `nc` = jumlah kelas (5), `names` harus urut sama persis dengan `classes.txt` di Tahap 4.

---

## 8. Training

```bash
nano ~/Training2601/train.py
```

```python
from ultralytics import YOLO
import torch

print(f"GPU: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU only'}")

model = YOLO("yolo26n.pt")  # bisa ganti "yolo11n.pt" kalau mau versi lebih matang/teruji

results = model.train(
    data        = "/home/bmrt/Training2601/data_train/data.yaml",
    epochs      = 200,
    imgsz       = 640,
    batch       = 8,       # turunkan ke 4 kalau error memory
    name        = "5cls_v1",
    project     = "/home/bmrt/Training2601/runs",
    device      = 0 if torch.cuda.is_available() else "cpu",
    save        = True,
    save_period = 10,
    hsv_s       = 0.7,     # penting buat bedain bola hijau/merah/biru
    hsv_v       = 0.4,
    hsv_h       = 0.015,
    fliplr      = 0.5,
    mosaic      = 1.0,
    verbose     = True
)

print(f"\nSelesai! Model terbaik:")
print(f"{results.save_dir}/weights/best.pt")
```

Jalankan:
```bash
python3 ~/Training2601/train.py
```

**Penjelasan:**
- `yolo26n.pt` ganti ke `yolo11n.pt` — tinggal ganti nama file, script lainnya sama.
- `epochs = 200` 
- `batch = 8` kalau training macet/crash karena memory, turunkan ke `4` atau `2`.
- Hasil model terbaik tersimpan di:
  ```
  ~/Training2601/runs/5cls_v1/weights/best.pt
  ```

---

## 9. Cek Hasil Training

Setelah training selesai, cek metrik akurasinya:
```bash
cat ~/Training2601/runs/5cls_v1/results.csv
```

Atau langsung lihat grafik (`results.png`), confusion matrix (`confusion_matrix.png`), dan contoh prediksi di folder:
```
~/Training2601/runs/5cls_v1/
```

---

Tinggal rename pakai mv:
```
mv ~/Training2601/runs/5cls_v1/weights/best.pt ~/Training2601/runs/5cls_v1/weights/5class26.pt
```

Kalau mau otomatis rename tiap kali training selesai (biar nggak lupa manual), bisa tambahin baris di akhir train.py, setelah results = model.train(...):
```
import shutil
best_path = f"{results.save_dir}/weights/best.pt"
new_path  = f"{results.save_dir}/weights/5class26.pt"
shutil.copy(best_path, new_path)
print(f"Model juga disalin ke: {new_path}")
```

---
# BAGI BAGI 
```
nano ~/Training2601/split_labeled.py
```
pisah
```
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


terus 
```
python3 ~/Training2601/split_labeled.py
```

bagi foto
```
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
# Pindahin CLASS
```
cp ~/Training2601/raw_images/classes.txt ~/Training2601/belum_label_PC/
cp ~/Training2601/raw_images/classes.txt ~/Training2601/belum_label_LAPTOP/
```

<br>

# Gabung semua hasil (sudah_label + PC + laptop) jadi satu folder final
pertama
```
nano ~/Training2601/merge_final.py
```
kedua
```
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

run
```
python3 ~/Training2601/merge_final.py
```

cek 
```
ls ~/Training2601/raw_images_final/*.jpg | wc -l
ls ~/Training2601/raw_images_final/*.txt | wc -l
```




