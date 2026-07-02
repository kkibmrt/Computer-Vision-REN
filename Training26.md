# Buat Folder

mkdir -p ~/Training26/data_train/images/train <br>
mkdir -p ~/Training26/data_train/images/val <br>
mkdir -p ~/Training26/data_train/labels/train <br>
mkdir -p ~/Training26/data_train/labels/labels/val <br>
mkdir -p ~/Training26/raw_images

<br>

# Taruh video

<br>

# extract_frame.py
nano ~/Training26/extract_frames.py

```python
import cv2, os

video  = os.path.expanduser("~/Training26/video.mp4")
output = os.path.expanduser("~/Training26/raw_images/")
os.makedirs(output, exist_ok=True)

cap   = cv2.VideoCapture(video)
fps   = cap.get(cv2.CAP_PROP_FPS)
total = cap.get(cv2.CAP_PROP_FRAME_COUNT)
print(f"FPS: {fps:.0f} | Total frame: {total:.0f} | Durasi: {total/fps:.1f} detik")

n_skip = 5  # ambil 1 frame setiap 5 frame → ~900 foto
print(f"Estimasi foto: {int(total/n_skip)}")

count = saved = 0
while True:
    ret, frame = cap.read()
    if not ret: break
    if count % n_skip == 0:
        cv2.imwrite(f"{output}frame_{saved:04d}.jpg", frame)
        saved += 1
    count += 1

cap.release()
print(f"Selesai! {saved} foto tersimpan di {output}")
```
<br>

# jalanin
```
cd ~/Training26 
python3 extract_frames.py
```


# hapus foto yg gk itu
nautilus ~/Training26/raw_images/

# cek
bisa ini jga
ls ~/Training26/raw_images/*.jpg | wc -l

# hapus foto jelek
nautilus ~/Training26/raw_images/

# cek foto yg tersisa
ls ~/Training26/raw_images/*.jpg | wc -l   # cek sisa

# buat class
cat > ~/Training26/raw_images/classes.txt << 'EOF'
hijau
merah
biru
EOF

# labelImg

# Split data set
nano ~/Training26/split_dataset.py

```python
import shutil, random
from pathlib import Path

SRC  = Path.home() / "Training26/raw_images"
OUT  = Path.home() / "Training26/data_train"

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

# jalankan
```
python3 ~/Training26/split_dataset.py
```
cek:
ls ~/Training26/data_train/images/train/ | wc -l <br>
ls ~/Training26/data_train/labels/train/ | wc -l <br>
ls ~/Training26/data_train/images/val/   | wc -l <br>
ls ~/Training26/data_train/labels/val/   | wc -l

<br>

# data.yaml
```
cat > ~/Training26/data_train/data.yaml << 'EOF'
path: /home/bmrt/Training26/data_train
train: images/train
val:   images/val

nc: 3
names:
  0: hijau
  1: merah
  2: biru
EOF
```

<br>

# Training
nano ~/Training26/train.py
```
from ultralytics import YOLO
import torch

print(f"GPU: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU only'}")

model = YOLO("yolov8n.pt")  # auto download

results = model.train(
    data        = "/home/rein/Training26/data_train/data.yaml",
    epochs      = 150,
    imgsz       = 640,
    batch       = 8,       # turunkan ke 4 kalau error memory
    name        = "3bola_v1",
    project     = "/home/rein/Training26/runs",
    device      = 0 if torch.cuda.is_available() else "cpu",
    save        = True,
    save_period = 10,
    hsv_s       = 0.7,    # variasi saturasi warna, penting untuk 3 warna bola
    hsv_v       = 0.4,
    hsv_h       = 0.015,
    fliplr      = 0.5,
    mosaic      = 1.0,
    verbose     = True
)

print(f"\nSelesai! Model terbaik:")
print(f"{results.save_dir}/weights/best.pt")
```

<br>

# jalanin
python3 ~/Training26/train.py


# cek auto safe 
```
ls -ld ~/Training2601/raw_images
```












