---
name: N8N-Google-Sheets
description: Expert guide for reliable Google Sheets interactions in n8n. Use this when configuring Google Sheets nodes, fixing sync issues, updating reports, or handling rate limits.
license: Creative Commons Zero v1.0 Universal
---

# N8N Google Sheets Expert Guide

This skill encapsulates best practices for reading, writing, and updating Google Sheets in n8n, based on real-world debugging of energy monitoring workflows.

## Core Concepts

### 1. Operation Mode vs. Intent
A common pitfall is configuring the node but selecting the wrong **Operation**.
*   **Get Row(s)**: READ only. Will not change data.
*   **Append**: Adds new rows to the bottom. Best for logs or initial population.
*   **Upsert (Append or Update)**: Updates existing rows if a key matches, otherwise appends. Best for synchronizing state (e.g., "Daily Report").

### 2. Rate Limiting (The "429" Error)
Google Sheets API has strict quotas (e.g., 60 writes per minute).
*   **Symptom**: "Quota exceeded for quota metric 'Read/Write requests'".
*   **Fix**: Do not send 100+ items in parallel.
*   **Pattern**:
    1.  **SplitInBatches** Node: Set batch size to `50`.
    2.  **Google Sheets** Node.
    3.  **Wait** Node: Set to `2-5 seconds`.
    4.  Loop back to SplitInBatches.

## Common Patterns

### Pattern 1: Robust Upsert (Syncing Daily Data)
When updating rows based on a date (e.g., "2025-01-01"), strict string matching applies.

*   **Requirement**: The formatted date string in n8n MUST match the displayed format in Google Sheets exactly.
*   **The Trap**: n8n might send `1-1-2025`, but Sheets expects `01-01-2025`. The match fails, and n8n inserts a duplicate row.
*   **Solution**: Force formatting in Code node with leading zeros.
    ```javascript
    const d = new Date(row.timestamp);
    const day = String(d.getDate()).padStart(2, '0');
    const month = String(d.getMonth() + 1).padStart(2, '0');
    // Result: "01-01-2025"
    const fmtDate = `${day}-${month}-${d.getFullYear()}`;
    ```

### Pattern 2: Fixing Silent Failures
If the node executes successfully (Green) but no data appears:
1.  **Check Operation**: Is it set to `Get Row(s)` instead of `Append`?
2.  **Value Input Option**: Set `options.valueInputOption` to `USER_ENTERED`. This forces Google to parse numbers and dates (e.g., "2025" becomes a number, not text '2025).

### Pattern 3: Handling API Arrays & Loops
When an HTTP node returns a list (array) of items, n8n splits them.
*   **Issue**: Subsequent HTTP nodes will run *once per item* (e.g., 400 times), causing a loop.
*   **Fix**:
    *   Set **Execute Once** on the subsequent HTTP node if it doesn't need input from *each* individual item.
    *   Use `$input.all()` in a Code node to re-aggregate the items into a single array for processing.

## Troubleshooting Checklist

| Issue | Check | Solution |
| :--- | :--- | :--- |
| **"Quota exceeded"** | Processing > 50 items? | Add `SplitInBatches` (50) + `Wait` (2s). |
| **Duplicate Rows** | Upserting? | Check Key format. Does "1-1-2025" match "01-01-2025"? Add `padStart`. |
| **No Data in Sheet** | Execution Green? | Check `Operation`. Is it "Get"? Change to "Append/Upsert". |
| **Infinite HTTP Requests** | Multiple items input? | Toggle `Execute Once` on the HTTP node or aggregate data. |
