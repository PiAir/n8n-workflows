# Implementation Plan - Energie Update

## Goal
Create an n8n workflow that synchronizes local P1 meter data to a Google Sheet daily at 01:00 AM.

## User Review Required
> [!NOTE]
> **Data Mapping Confirmation**:
> *   **Power/Gas**: `Gas`=Index 9, `Stroom`=Index 6, `Teruglevering`=Index 7.
> *   **Weather**: `Graaddagen` assumed to be the **last value** in the array (Index 19), based on magnitude comparison with the sample sheet.
> *   **Colors**: Conditional formatting will be handled by the user in Google Sheets, not the workflow.

## Proposed Workflow Design

### Triggers
- **Schedule Trigger**: Runs every day at 01:00:00.
- **Manual Trigger**: For testing and ad-hoc updates.

### Branch 1: Graaddagen (Degree Days)
1.  **Fetch Data**:
    - GET `http://192.168.0.119/api/v1/powergas/day?sort=asc&starttime=2025-01-01%2000:00:00`
    - GET `http://192.168.0.119/api/v1/weather/day?sort=asc&starttime=2025-01-01%2000:00:00`
2.  **Filter**: Pre-filtered by API `starttime`.
3.  **Merge**: Combine Power/Gas data with Weather data (matching by Date at Index 0).
4.  **Calculate**:
    - `Dag`: Format from Index 0 (`2026-02-16 00:00:00`) to `DD-MM-YYYY`.
    - `m3` (Gas): Index 9 from PowerGas API.
    - `Graaddag`: Last Index from Weather API.
    - `m3 / GD`: `Gas / Graaddag`.
    - `max m3`: `Graaddag * 0.65` (Rounded to 0 decimals).
5.  **Target**: Google Sheet "Graaddagen".
    - **Method**: Clear Sheet (except headers) & Insert (Simplest to ensure consistency), OR Upsert based on Date key. Given the "overwrite" instruction, a full sync of the 2024+ window is robust.

### Branch 2: Verbruik per maand (Monthly Usage)
1.  **Fetch Data**:
    - GET `http://192.168.0.119/api/v1/powergas/month?sort=asc&starttime=2025-01-01%2000:00:00`
2.  **Filter**: Pre-filtered by API `starttime`.
3.  **Calculate**:
    - `Maand`: Convert date (Index 0) to Dutch month name.
    - `Jaar`: Extract Year from date.
    - `Gas` (m3): Index 9.
    - `Stroom` (kWh): Index 6.
    - `Teruglevering` (kWh): Index 7.
    - `Totaal`: `Stroom - Teruglevering`.
    - `Water`: Ignored.
4.  **Target**: Google Sheet "Verbruik per maand".
    - **Method**: Synced update similar to Branch 1.

## Verification Plan
1.  **Mock Data Testing**: I will create mock JSON data structures based on the user's provided examples (once received) to test the `Code` nodes and calculations.
2.  **Dry Run**: The user will need to run the workflow locally to verify it hits their local API correctly.
