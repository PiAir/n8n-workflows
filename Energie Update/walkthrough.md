# Workflow Walkthrough: Energie Update

## Workflow Configuration
- **Status**: Operational.
- **Data Write Mode**: `Append` (Adds new rows).
- **Data Limit**: Currently set to **50 items** for testing. To sync all history, remove the `.slice(0, 50)` in the code node.

## Setting Up Conditional Formatting (Google Sheets)
To visualize high/low gas usage relative to degree days:

1.  Open your Google Sheet "Graaddagen".
2.  Select **Column D** (`m3 / GD`).
3.  Go to **Format** > **Conditional formatting**.
4.  Select the **Color scale** tab.
    - **Minpoint**: Number `0` (Green)
    - **Midpoint**: Number `0.5` (Yellow)
    - **Maxpoint**: Number `1.0` (Red)
5.  Click **Done**.

*Target values:*
- `< 0.3`: Very efficient / Warm day
- `0.3 - 0.6`: Normal
- `> 0.6`: High usage relative to cold

## Restoring Full History Sync
To sync all data from Jan 1, 2025 (instead of just 50 rows):
1.  Open `Code Graaddagen` node.
2.  Remove `return results.slice(0, 50);` at the bottom.
3.  Replace with `return results;`.
4.  **Important**: If you have >1000 rows, consider re-enabling the `SplitInBatches` loop to avoid errors.
