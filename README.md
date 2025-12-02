# n8n Telegram WeatherBot

An automated weather notification system built with **n8n**. This workflow monitors weather forecasts and alerts specific Telegram users if rain is expected during class hours (8 AM - 5 PM) or evening plans (post-6 PM).

Also logs predictions vs. actual weather data into Google Sheets. My demo google sheets: https://docs.google.com/spreadsheets/d/1OaOczzJYu0bQVdxowieUxfSqqHm83aAmB0Ej2sZ3ruU/edit?usp=sharing

* **Multi-User Support:** Easily scalable array of user objects to notify multiple friends/roommates.
* **Different Messages Based on Rain Proability:** Sends custom messages for different rain probabilties to keep it fresh.

## Prerequisites

* **n8n:** Self-hosted (Docker/Node.js) or Cloud version.
* **Telegram Bot:** You need a Bot Token from `@BotFather`.
* **WeatherAPI Key:** Free API key from [WeatherAPI.com](https://www.weatherapi.com/).
* **Google Cloud Console Project:** With Google Sheets API enabled (service account credentials).

## Setup & Installation

### 1. Import the Workflow
1.  Download the `weather-bot.json` file from this repository.
2.  Open your n8n dashboard.
3.  Click **Add Workflow** -> **Import from File** and select the JSON file.

### 2. Configure Credentials
You must set up the following credentials in n8n for the nodes to authenticate:
* **Telegram API:** Create a credential named `Telegram account` with your Bot Token.
* **Google Sheets API:** Create a credential named `Google Sheets account` using your Service Account email and Private Key.

### 3. Replace Placeholders (Crucial Step)
To protect privacy, the workflow contains placeholders. You must **Find and Replace** these values in the nodes before activating the workflow.

| Placeholder | Where to find it | Value to replace with |
| :--- | :--- | :--- |
| `PLACEHOLDER_CHAT_ID_X` | **Code Nodes** (UserData, UserData1, UserData2) | The numeric Telegram Chat ID for the user you want to message. |
| `YOUR_WEATHERAPI_KEY` | **HTTP Request Nodes** (Rain HTTP Request, etc.) | Your API Key from WeatherAPI.com. |
| `PLACEHOLDER_SPREADSHEET_ID` | **Google Sheets Nodes** | The long ID string found in your Google Sheet URL. |
| `PLACEHOLDER_CREDENTIAL_ID` | **Various Nodes** | *Note: If you set up credentials in n8n (Step 2), you can simply select them from the dropdown menu in the node UI instead of pasting an ID.* |
| `email@example.com` | **OpenMeteo Request Node** | Your email address (required by OpenMeteo for "User-Agent" headers). |

### 4. Google Sheets Setup
Create a Google Sheet with a tab named `RawData`. The workflow expects specific columns to exist for logging.  
* **Required Columns:** `Date`, `Location`, `Predicted Rain During Day`, `Actual Rain During Day`, `Predicted 8am`... `Predicted 11pm`, `Actual 8am`... `Actual 11pm`.  
*Use Google Apps Script to Calculate accuracy through comparisons between Predicted and Actual columns.*

## Logic Overview

1.  **8:00 AM Trigger:** Fetches forecast.
    * If rain > 60%: Sends warning.
    * If clear: Sends "No umbrella needed" or "Clear skies" message.
    * Logs prediction to Google Sheets.
2.  **5:00 PM Trigger:** Fetches evening forecast.
    * If rain detected after 6 PM: Sends specific evening warning.
3.  **6:00 AM (Next Day) Trigger:** * Fetches historical weather data for the previous day.
    * Logs daily and hourly comparisons for `Predicted` vs `Actual` into Google Sheets.
