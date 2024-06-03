---
title: 免费 零基础自行动手 pdf转文字
catalog: true
header-img: img/header_img/roman.png
subtitle: The quick brown fox jumps over the lazy dog
date: 2024-06-03 14:42:19
tags:
catagories:
---

# 免费 零基础自行动手 pdf转文字

## 工具

[ocrmypdf 文档](https://ocrmypdf.readthedocs.io/en/latest/index.html)
[ocrmypdf 文档 官网具体下载教程](https://ocrmypdf.readthedocs.io/en/latest/installation.html)
[Ghostscript 下载地址](https://ghostscript.com/releases/gsdnld.html)

- Python 64-bit
- Tesseract 64-bit
- Ghostscript 64-bit

```bat
winget install -e --id Python.Python.3.11
winget install -e --id UB-Mannheim.TesseractOCR
python3 -m pip install ocrmypdf
```

## 当前目录 批量转化pdf为文字 执行代码

[代码文件](batch.py)

```sh
python ./batch.py '.'
```

### batch.py 源码
```py
#!/usr/bin/env python3
from __future__ import annotations

import logging
import sys
from pathlib import Path

import ocrmypdf

# pylint: disable=logging-format-interpolation
# pylint: disable=logging-not-lazy

script_dir = Path(__file__).parent
print(script_dir)

if len(sys.argv) > 1:
    start_dir = Path(sys.argv[1])
else:
    start_dir = Path('.')

print(start_dir)

if len(sys.argv) > 2:
    log_file = Path(sys.argv[2])
else:
    log_file = script_dir.with_name('ocr-tree.log')

print(log_file)

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(message)s',
    filename=log_file,
    filemode='a',
)

ocrmypdf.configure_logging(ocrmypdf.Verbosity.quiet)

for filename in start_dir.glob("**/*.pdf"):
    logging.info(f"Processing {filename}")
    print(f"Processing {filename}")
    print(filename)
    output_file = f"{filename}_tmp.pdf"
    print(output_file)

    result = ocrmypdf.ocr(filename, output_file, deskew=True, language=["chi_sim","eng"], jobs=2, skip_text=True, progress_bar=True, optimize=1, png_quality=30, jpeg_quality=40)
    #, jbig2_page_group_size=1

    if result == ocrmypdf.ExitCode.already_done_ocr:
        logging.error("Skipped document because it already contained text")
    elif result == ocrmypdf.ExitCode.ok:
        logging.info("OCR complete")
    logging.info(result)
```

## 然后就能通过朗读 边读边看 提升阅读效率