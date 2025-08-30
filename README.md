# inventory_cli_lab

A playground to develop idea about the Inventory project

## Google Sheets Inventory Notebook

Notebook: `notebooks/google_sheets_inventory.ipynb`

Features:

- Service Account authentication to Google Sheets
- List / update inventory rows in an `Items` sheet
- Helper CLI script generation (`scripts/update_sheet.py`)

### Prerequisites

1. Enable Google Sheets API in a GCP project.
2. Create Service Account + JSON key.
3. Share target spreadsheet with the service account email (Editor).

### Environment Variables

```
GOOGLE_APPLICATION_CREDENTIALS=./service_account.json
GSHEET_INVENTORY_ID=YOUR_SHEET_ID
```

Optionally store them in a `.env` file (the notebook auto-loads with `python-dotenv`).

### Run Notebook

Open the notebook and run cells in order. The first install cell adds dependencies into the current environment.

### CLI Script Usage

The notebook creates `scripts/update_sheet.py`.

Example:

```
export GOOGLE_APPLICATION_CREDENTIALS=./service_account.json
export GSHEET_INVENTORY_ID=YOUR_SHEET_ID
python scripts/update_sheet.py --sku SKU123 --delta 5
```

### Notes

- API keys alone cannot modify private sheets; use service account credentials.
- Adjust column indices / sheet names if your structure differs.

---

Feel free to request further scaffolding (tests, packaging, stock movement logging, etc.).
