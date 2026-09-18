---
license: mit
task_categories:
- image-text-to-text
- object-detection
language:
- ja
tags:
- manga
- comics
- text-detection
- reading-order
- OCR
pretty_name: Manga109-s Text Line Annotations
---
# Manga109-s Text Line Annotations

High-precision, line-level bounding box and polygon annotations for the [Manga109-s Dataset](https://huggingface.co/datasets/hal-utokyo/Manga109-s), supporting both full manga pages and speech bubble crops. Furigana is not labeled and is almost entirely excluded from line labels. Includes 8-point oriented polygons for slanted/rotated text lines. The annotation process is documented in [METHODOLOGY.md](METHODOLOGY.md) (WIP).

> **Notice**: This dataset contains **zero dialogue text and zero images**. It requires your own local copy of Manga109-s (or Manga109) to reconstruct dialogue text and process images.

---

## Dataset Statistics

| Metric | Count |
| :--- | :--- |
| **Manga Books** | 87 |
| **Manga Pages** | 380 |
| **Speech Bubble Crops** | 5,988 |
| **Text Lines** | 14,182 |
| **Oriented Polygons** | 392 |
| **Total Characters** | 74,506 |
| **Average Lines / Bubble** | 2.37 |
| **Average Characters / Line** | 5.25 |

---

## Quickstart

### 1. Prerequisites & Installation
Obtain an official copy of Manga109-s from [Hugging Face (`hal-utokyo/Manga109-s`)](https://huggingface.co/datasets/hal-utokyo/Manga109-s) (or the official Manga109 website; the 2026 version is recommended).

Ensure you have Python 3.8+. You can install the companion package or run directly with `uv`:

```bash
# Option A: Install as an editable package (provides the `manga109-lines` CLI)
pip install -e .
manga109-lines --help

# Option B: Run directly with uv (recommended, no manual install needed)
uv run manga109-lines --help
# or
uv run python build_dataset.py --help
```

> **Note**: In all subsequent examples, `manga109-lines` and `uv run python build_dataset.py` can be used interchangeably.

### 2. Verify Alignment with Local Manga109-s
Check that your local Manga109-s files match the annotation geometry:
```bash
uv run python build_dataset.py verify --manga109-dir ./manga109s-v2026
```

### 3. Reconstruct Full Annotations with Dialogue Text
Fills in the official dialogue text from your local Manga109-s XMLs or CSV into a complete `verified_data.json`:
```bash
uv run python build_dataset.py reconstruct \
  --manga109-dir ./manga109s-v2026 \
  --output verified_data_reconstructed.json
```

---

## Exporting to Machine Learning Formats

### 1. YOLO Format (Ultralytics YOLOv8 / YOLO11)

#### Mode A: Full Page Line Detection
Detects text lines across entire manga pages (`images/{book}/{page}.jpg`):
```bash
uv run python build_dataset.py export-yolo \
  --manga109-dir ./manga109s-v2026 \
  --target page \
  --task detect \
  --output-dir yolo_line_pages
```

#### Mode B: Bubble Crop Line Detection
Detects individual text lines within cropped speech bubbles (`crops/{id}.png`):
```bash
uv run python build_dataset.py export-yolo \
  --manga109-dir ./manga109s-v2026 \
  --target crop \
  --task detect \
  --output-dir yolo_line_crops
```

*Options*:
- `--task segment`: Exports normalized 8-point polygon segmentations for oriented lines.
- `--include-images`: Automatically copies or symlinks images into `images/train` and `images/val`.

### 2. COCO Instances JSON
Exports standard COCO instances JSON with `text_line` (category 1) and `text_block` (category 2):
```bash
# Page-level COCO
uv run python build_dataset.py export-coco \
  --manga109-dir ./manga109s-v2026 \
  --target page \
  --output manga109s_lines_coco_page.json

# Crop-level COCO
uv run python build_dataset.py export-coco \
  --manga109-dir ./manga109s-v2026 \
  --target crop \
  --output manga109s_lines_coco_crop.json
```

### 3. Enhanced Manga109 XML Files
Inserts `<line index="..." xmin="..." ymin="..." xmax="..." ymax="...">` tags directly into the official Manga109 XML files:
```bash
uv run python build_dataset.py export-xml \
  --manga109-dir ./manga109s-v2026 \
  --output-dir manga109s_xml_with_lines
```

Sample output element:
```xml
<text id="00000d6f" xmin="192" ymin="957" xmax="263" ymax="1036">セリフ１\nセリフ２\nセリフ３
  <line index="0" xmin="236" ymin="957" xmax="257" ymax="1014">セリフ１</line>
  <line index="1" xmin="213" ymin="957" xmax="235" ymax="1036">セリフ２</line>
  <line index="2" xmin="192" ymin="957" xmax="211" ymax="1036">セリフ３</line>
</text>
```

### 4. Line OCR Dataset (JSONL)
Exports a line-level OCR mapping file for text recognition training:
```bash
uv run python build_dataset.py export-ocr \
  --manga109-dir ./manga109s-v2026 \
  --output manga109s_crops_ocr.jsonl
```

---

## Data Schema (`line_annotations.json`)

```json
{
  "images/ARMS/065.jpg": {
    "book": "ARMS",
    "page_index": 65,
    "width": 1654,
    "height": 1170,
    "texts": [
      {
        "id": "00000d6f",
        "xmin": 192,
        "ymin": 957,
        "xmax": 263,
        "ymax": 1036,
        "line_lengths": [3, 4],
        "lines": [
          {
            "line_index": 0,
            "xmin": 236,
            "ymin": 957,
            "xmax": 257,
            "ymax": 1014,
            "char_count": 3
          },
          {
            "line_index": 1,
            "xmin": 213,
            "ymin": 957,
            "xmax": 235,
            "ymax": 1036,
            "char_count": 4,
            "polygon": [213, 960, 230, 1036, 235, 1033, 218, 957]
          }
        ]
      }
    ]
  }
}
```

### Key Fields
- `line_lengths`: Array of character slice lengths for each line in reading order.
- `char_count`: Number of characters corresponding to this line.
- `polygon`: *(Optional)* 8-point coordinate array `[x1, y1, x2, y2, x3, y3, x4, y4]` specifying the oriented bounding box for tilted or slanted lines (present on 392 lines).

---

## License & Citation

### Annotation & Code License
The line annotations and accompanying tooling (`build_dataset.py`) are released under the [MIT License](LICENSE).

> **Note**: This license applies solely to the line annotation geometry files and associated utility code. The underlying Manga109-s dataset and artwork remain subject to the Manga109 Terms of Use.

### Manga109 Terms of Use Notice
This release strictly abides by the Manga109-s Terms of Use. To use this dataset with original text or imagery, you must obtain a legitimate copy of Manga109-s from:
- [Hugging Face Manga109-s](https://huggingface.co/datasets/hal-utokyo/Manga109-s)
- [Manga109 Official Site](https://manga109.github.io/manga109-project-website/en/index.html)

## Citation

If you use these line annotations or conversion tools in your research, please cite this repository:

```bibtex
@misc{bluolightning2026manga109slines,
  author       = {Nav (bluolightning)},
  title        = {{Manga109-s Text Line Annotations}},
  year         = {2026},
  publisher    = {Hugging Face},
  howpublished = {\url{https://huggingface.co/datasets/bluolightning/manga109s-line-annotations}}
}
```

Please also cite the underlying Manga109 / Manga109-s dataset and annotation papers:
```
@inproceedings{baek2026mangav26,
  title     = {{Manga109-v2026: Revisiting Manga109 Annotations for Modern Manga Understanding}},
  author    = {Baek, Jeonghun and Miyai, Atsuyuki and Onohara, Shota and Ikuta, Hikaru and Aizawa, Kiyoharu},
  booktitle = {Culture × AI Workshop at ICML 2026},
  year      = {2026}
}

@article{multimedia_aizawa_2020,
  author  = {Aizawa, Kiyoharu and Fujimoto, Azuma and Otsubo, Atsushi and Ogawa, Toru and Matsui, Yusuke and Tsubota, Koki and Ikuta, Hikaru},
  title   = {Building a Manga Dataset ``{Manga109}'' with Annotations for Multimedia Applications},
  journal = {IEEE MultiMedia},
  volume  = {27},
  number  = {2},
  pages   = {8--18},
  doi     = {10.1109/mmul.2020.2987895},
  year    = {2020}
}

@article{mtap_matsui_2017,
  author  = {Matsui, Yusuke and Ito, Kota and Aramaki, Yuji and Fujimoto, Azuma and Ogawa, Toru and Yamasaki, Toshihiko and Aizawa, Kiyoharu},
  title   = {Sketch-based Manga Retrieval using {Manga109} Dataset},
  journal = {Multimedia Tools and Applications},
  volume  = {76},
  number  = {20},
  pages   = {21811--21838},
  doi     = {10.1007/s11042-016-4020-z},
  year    = {2017}
}
```