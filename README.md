# Receipt & Expense Analyzer

A command-line tool that reads receipt photos with OCR, extracts the vendor, date, and total, and turns a folder of receipts into a spending report — entirely offline, with no cloud service, account, or API key involved.

```bash
python expense_cli.py --add receipt1.jpg receipt2.jpg
python expense_cli.py --report
```

That's the whole workflow: add receipts as you collect them, then generate an updated report whenever you want a picture of spending.

## What it covers

- **OCR extraction** — vendor name, date, and total amount, straight from a photo or scan
- **Automatic categorization** — vendor names are matched against spending categories (Groceries, Dining, Transportation, Health, Shopping, Utilities, Entertainment) with no manual tagging
- **Spending trend over time** — monthly totals
- **Category breakdown** — where the money is actually going
- **Top vendors by spend**

## Quick start

```bash
pip install -r requirements.txt
```

`pytesseract` is a Python wrapper — it needs the actual Tesseract OCR engine installed separately:
- **Windows**: install the [UB-Mannheim build](https://github.com/UB-Mannheim/tesseract/wiki) and keep the default install location — this tool automatically checks for it there
- **macOS**: `brew install tesseract`
- **Linux**: `sudo apt install tesseract-ocr`

If it's installed somewhere non-default and the tool still can't find it, open `receipt_ocr.py` in a text editor and add this line right after `import pytesseract`, with your actual install path:
```python
pytesseract.pytesseract.tesseract_cmd = r"C:\path\to\tesseract.exe"
```

```bash

# Try it instantly with sample data — no images or OCR needed
python expense_cli.py --demo

# Process your own receipts
python expense_cli.py --add path/to/receipt1.jpg path/to/receipt2.jpg
python expense_cli.py --list
python expense_cli.py --report
```

Output lands in `output/`:
- `report.html` — a single self-contained file (charts embedded inline) — open it in any browser
- `receipts.csv` — the underlying parsed data
- individual `.png` charts

Receipts you add are stored locally in `data/receipts.json`, so the database persists between runs — each `--report` reflects everything added so far.

## How it works

| File | Responsibility |
|---|---|
| `receipt_ocr.py` | Runs OCR on a receipt image (Tesseract) and parses out vendor, date, and total using a set of pattern-matching heuristics. |
| `categorize.py` | Assigns a spending category from the vendor name via keyword matching. |
| `database.py` | Local JSON storage for parsed receipts — add, list, load. |
| `analysis.py` | Converts stored receipts into a pandas DataFrame and computes report metrics. |
| `visualize.py` | Builds the matplotlib charts. |
| `report.py` | Assembles everything into one portable HTML file. |
| `expense_cli.py` | CLI entry point that wires it all together. |

## Design notes

- **Receipt parsing is heuristic by nature** — layouts vary too much for a rigid template, so the parser makes pragmatic choices: the vendor is the first non-empty line (store names are almost always printed at the top), the date is the first recognizable date pattern found anywhere in the text, and the total is read from the line containing a "total"-style keyword — explicitly distinguishing "Total" from "Subtotal" so the pre-tax amount isn't mistaken for the final one.
- **No total line? No problem.** If no total keyword is found, the parser falls back to the largest currency-looking amount on the receipt, since line items are almost always smaller than the final total.
- **OCR accuracy depends on image quality.** Clear, well-lit, reasonably high-resolution photos parse far more reliably than blurry or low-contrast ones — this is a property of OCR in general, not specific to this tool.

## Demo dataset

`sample_data/demo_receipts.json` is a set of synthetic but realistic receipts spanning several months and categories, so the tool can be tried and demoed without any real receipt images. Run `python expense_cli.py --demo` to see it in action.

`sample_data/sample_receipt_images/` also has a few synthetic receipt images to try the real OCR path end-to-end:
```bash
python expense_cli.py --add sample_data/sample_receipt_images/*.png
python expense_cli.py --report
```

## Possible extensions

- Support PDF receipts (scanned or digital) in addition to images
- Export a monthly summary to a spreadsheet
- A `--budget` mode that flags categories exceeding a set limit
