# Telegram WeatherBot

An automated weather notification system built with **n8n**. This workflow monitors weather forecasts and alerts specific Telegram users if rain, snow, or extreme temperatures are expected during class hours (8 AM - 5 PM) or evening plans (post-5 PM).

It also logs predictions vs. actual weather data into Google Sheets for accuracy tracking.
**[View my Demo Google Sheet](https://docs.google.com/spreadsheets/d/1OaOczzJYu0bQVdxowieUxfSqqHm83aAmB0Ej2sZ3ruU/edit?usp=sharing)**

## 📂 Repository Structure

* **`WeatherBot-V2.0.json`** (Main Branch): The latest version containing Rain, Temperature, and Snow alerts.
* **`OlderVersions/WeatherBot-V1.0.json`**: The legacy version (Rain alerts only).

## 🚀 Features

* **Multi-User Support:** Easily scalable array of user objects to notify multiple friends/roommates.
* **Smart Rain Alerts:** Sends specific alerts for rain during class vs. rain during evening plans.
* **🌡️ Temperature Alerts:** Warns users to bring a jacket if temperatures drop below a certain threshold.
* **❄️ Snow Reports:** Dedicated alerts if snow probability is high.
* **Data Logging:** Logs daily predictions and verifies them against actual historical data the next day.

## Prerequisites

* **n8n:** Self-hosted (Docker/Node.js) or Cloud version.
* **Telegram Bot:** You need a Bot Token from `@BotFather`.
* **WeatherAPI Key:** Free API key from [WeatherAPI.com](https://www.weatherapi.com/).
* **Google Cloud Console Project:** With Google Sheets API enabled (service account credentials).

## Setup & Installation

### 1. Import the Workflow
1.  Download `WeatherBot-V2.0.json` from the main branch of this repository.
2.  Open your n8n dashboard.
3.  Click **Add Workflow** -> **Import from File** and select the JSON file.

### 2. Configure Credentials
You must set up the following credentials in n8n for the nodes to authenticate:
* **Telegram API:** Create a credential named `Telegram account` with your Bot Token.
* **Google Sheets API:** Create a credential named `Google Sheets account` using your Service Account email and Private Key.

### 3. Replace Placeholders
To protect privacy and make this template location-neutral, the workflow contains placeholders. You must **Find and Replace** these values in the nodes before activating the workflow.

| Placeholder / Value | Where to find it | Value to replace with |
| :--- | :--- | :--- |
| `PLACEHOLDER_CHAT_ID_X` | **Code Nodes** (UserData, etc.) | The numeric Telegram Chat ID for the user you want to message. |
| `YOUR_WEATHERAPI_KEY` | **HTTP Request Nodes** | Your API Key from WeatherAPI.com. |
| `PLACEHOLDER_CITY` | **HTTP Request Nodes** | Your city (e.g., `college park` or `new york`). |
| `0.00` (Lat/Long) | **Code Node** ("get yesterday's date") | The specific Latitude and Longitude of your location (required for historical data). |
| `PLACEHOLDER_SPREADSHEET_ID` | **Google Sheets Nodes** | The long ID string found in your Google Sheet URL. |
| `PLACEHOLDER_LOCATION` | **Google Sheets Node** (Predicted) | The text name of your location (e.g., "College Park, MD"). |
| `email@example.com` | **OpenMeteo Request Node** | Your email address (required by OpenMeteo for "User-Agent" headers). |

### 4. Google Sheets Setup
Create a Google Sheet with a tab named `RawData`. The workflow expects specific columns to exist for logging.
* **Required Columns:** `Date`, `Location`, `Predicted Rain During Day`, `Actual Rain During Day`, `Predicted 8am`... `Predicted 11pm`, `Actual 8am`... `Actual 11pm`.
* *Tip: You can copy the structure directly from my demo Google Sheet linked above.*

## 🧠 Logic Overview (V2.0)

1.  **8:00 AM Trigger:** Fetches daily forecast.
    * **Rain Check:** If > 60%, sends an umbrella alert. If clear, sends a "no umbrella needed" message.
    * **Temp Check:** If "feels like" temp is low, sends a "Bring a Jacket" alert.
    * **Snow Check:** If snow probability is high, sends a Snow Report.
    * **Logging:** Logs all hourly predictions to Google Sheets.
2.  **5:00 PM Trigger:** Fetches evening forecast.
    * Checks for Rain, Snow, and Low Temps specifically for the hours of 6 PM - 11 PM.
    * Sends a warning if you have evening plans.
3.  **6:00 AM (Next Day) Trigger:** Fetches historical weather data for the previous day.
    * Logs daily and hourly comparisons for `Predicted` vs `Actual` into Google Sheets to track bot accuracy.
