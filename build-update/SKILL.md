---
name: build-update
description: >
  Tự động tạo gói cập nhật (manifest.json + ZIP) cho SkillDo CMS v8 bằng cách
  đọc git diff, remap đường dẫn, và đóng gói bằng Python — KHÔNG dùng PHP scripts.

  LUÔN dùng skill này khi người dùng nhắc đến: "tạo file update", "build update",
  "đóng gói update", "tạo gói cập nhật", "build release", "tạo zip deploy",
  "xuất bản bản cập nhật", "tạo file để upload lên server", /build-update.
  Kể cả khi họ chỉ nói "update xong rồi" hay "cần package để deploy".
---

# Build Update Package

Claude tự thực hiện toàn bộ quy trình — không gọi PHP script nào.

## Cơ chế theo dõi commit range

`manifest.json` lưu trường `built_from` chứa **commit hash của HEAD tại thời điểm build**.
Lần build tiếp theo sẽ dùng hash đó để lấy TẤT CẢ file thay đổi từ lần build trước — kể cả
các commit đã push — không chỉ các file uncommitted.

```
Lần build 1:  built_from = null    → diff HEAD  (chưa committed)
              → lưu built_from = <hash_hiện_tại>

Lần build 2:  built_from = abc123  → diff abc123..HEAD (tất cả commit từ sau abc123)
                                   + diff HEAD         (uncommitted hiện tại)
              → lưu built_from = <hash_mới>
```

## Bước 1 — Đọc manifest.json hiện có

Nếu `manifest.json` đã tồn tại → đọc để lấy:
- `built_from` — hash commit của lần build trước (dùng làm base cho diff)
- `migrations` — giữ nguyên
- `delete` — giữ nguyên

Nếu chưa có → `built_from = null`, `migrations = {}`, `delete = []`.

## Bước 2 — Lấy HEAD commit hash hiện tại

```powershell
git rev-parse HEAD
git log --oneline -5
```

Lưu hash này vào `new_built_from` — sẽ ghi vào manifest sau.

## Bước 3 — Lấy danh sách file thay đổi

**Nếu `built_from` có giá trị** (có lần build trước):
```powershell
# Tất cả file thay đổi từ sau commit built_from đến HEAD (các commit đã push)
git diff --name-only <built_from>..HEAD
# File uncommitted hiện tại (chưa commit)
git diff --name-only HEAD
# File đã staged nhưng chưa commit
git diff --cached --name-only
```

**Nếu `built_from` là null** (lần build đầu tiên):
```powershell
git diff --name-only HEAD
git diff --cached --name-only
```

Gộp tất cả, loại trùng, loại bỏ:
- `manifest.json`
- Bất kỳ file nào trong `.claude/`
- `CLAUDE.md`
- Bất kỳ file nào trong `views/admin/assets/js/bundle/` — thư mục này chứa source JS chưa build, sẽ được compile thành `script.bundle.js` và `element-builder-review.bundle.js`; chỉ đóng gói các file bundle đã build, không đóng gói source

Nếu danh sách rỗng → thông báo cho người dùng và dừng.

## Bước 4 — Đọc version CMS

Đọc `packages/skilldo/cms/src/config/cms.php`, lấy giá trị `'version'`.

## Bước 5 — Ghi manifest.json

```json
{
    "version": "<version>",
    "built_from": "<new_built_from — hash HEAD hiện tại>",
    "migrations": {},
    "delete": [],
    "files": ["<danh sách file đã lọc>"]
}
```

Trường `built_from` ghi hash đầy đủ (40 ký tự) để git có thể tra cứu chính xác.

## Bước 6 — Tạo ZIP bằng Python

Ưu tiên lấy file từ thư mục **encoded ionCube** (`E:\Source-Sikido\source-v8\sourcev8`),
fallback về source gốc nếu không tìm thấy. Encoded path dùng cấu trúc production (`vendor/`).

Ghi script ra `tool\_build_update_tmp.py` rồi chạy:

```python
import zipfile, os, json

BASE    = r"E:\GoogleDriverData\projects\source.5.x\sourcev8"
ENCODED = r"E:\Source-Sikido\source-v8\sourcev8"
RELEASES = os.path.join(BASE, "tool", "releases")
os.makedirs(RELEASES, exist_ok=True)

with open(os.path.join(BASE, "manifest.json"), encoding="utf-8") as f:
    manifest = json.load(f)

version  = manifest["version"]
zip_path = os.path.join(RELEASES, f"update-{version}.zip")

REMAPS = [
    ("packages/skilldo/cms/",       "vendor/skilldo/cms/"),
    ("packages/skilldo/framework/", "vendor/skilldo/framework/"),
]

def remap(path):
    for src, dst in REMAPS:
        if path.startswith(src):
            return dst + path[len(src):]
    return path

def resolve(rel, zip_entry):
    """Ưu tiên encoded (dùng zip_entry = cấu trúc production), fallback về source gốc."""
    encoded_abs = os.path.join(ENCODED, zip_entry.replace("/", os.sep))
    if os.path.isfile(encoded_abs):
        return encoded_abs, "encoded"
    source_abs = os.path.join(BASE, rel.replace("/", os.sep))
    if os.path.isfile(source_abs):
        return source_abs, "source"
    return None, "missing"

encoded_ready = os.path.isdir(ENCODED)
missed = []
added  = []

with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    z.write(os.path.join(BASE, "manifest.json"), "manifest.json")
    for rel in manifest["files"]:
        zip_entry = remap(rel)
        abs_path, from_where = resolve(rel, zip_entry)
        if abs_path:
            z.write(abs_path, zip_entry)
            added.append((rel, zip_entry, from_where))
        else:
            missed.append(rel)

size_kb = round(os.path.getsize(zip_path) / 1024, 2)
print(f"Version   : {version}")
print(f"Built from: {manifest.get('built_from', 'N/A')[:12]}...")
print(f"Encoded   : {'co' if encoded_ready else 'KHONG CO — dung source goc'}")
print(f"Dong goi  : {len(added)} file")
for src, dst, src_type in added:
    arrow = f" -> {dst}" if src != dst else ""
    tag   = "[encoded]" if src_type == "encoded" else "[source]"
    print(f"  + {src}{arrow}  {tag}")
if missed:
    print(f"\nMissing ({len(missed)}):")
    for m in missed: print(f"  ! {m}")
print(f"\nZIP : {zip_path}  ({size_kb} KB)")
```
print(f"\nZIP : {zip_path}  ({size_kb} KB)")
```

```powershell
python tool\_build_update_tmp.py
# Sau đó xóa file tạm:
Remove-Item tool\_build_update_tmp.py
```

## Bước 7 — Báo kết quả

Hiển thị:
- Phiên bản và `built_from` (12 ký tự đầu của hash — đủ để nhận dạng)
- Số file, đường dẫn ZIP, kích thước KB
- File missing nếu có
- Gợi ý: dùng `git log <built_from>..HEAD --oneline` để xem danh sách commit đã được đóng gói

## Tham số tùy chọn

| Người dùng nói | Hành động |
|---|---|
| `--staged-only` | Chỉ dùng `git diff --cached --name-only` |
| `--from=<hash/tag>` | Dùng hash/tag đó làm base thay vì `built_from` trong manifest |
| `--reset` | Bỏ qua `built_from`, chỉ lấy uncommitted (như lần đầu) |

## Lưu ý

- Python có sẵn tại `C:\Python310\python.exe`
- `manifest.json` luôn được cập nhật trước khi tạo ZIP
- ZIP dùng cấu trúc **production** (`vendor/`) không phải dev (`packages/`)
- `built_from` chỉ được cập nhật sau khi ZIP tạo thành công
