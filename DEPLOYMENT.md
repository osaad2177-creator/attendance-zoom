# Deployment Guide

Follow the steps below to deploy the attendance system with a Google Apps Script backend and a GitHub Pages frontend.

---

## 1. Prepare the Google Sheet

1. Create a new Google Sheet named `Attendance`.
2. Rename the first worksheet (tab) to `Logs` (or any name you prefer, but keep it consistent with the script).
3. In row 1, add the following headers in **exact order**:
   - `Employee Name`
   - `Type`
   - `Date & Time`
   - `Latitude`
   - `Longitude`
4. Note the spreadsheet ID from its URL (`https://docs.google.com/spreadsheets/d/<SPREADSHEET_ID>/edit`).

---

## 2. Create the Google Apps Script backend

1. In the Google Sheet, open **Extensions → Apps Script**.
2. Replace the default code with the script below and update `SPREADSHEET_ID` and `SHEET_NAME`:

```javascript
const SPREADSHEET_ID = '1dLO94p2n8wvRug_AhpoDPlBsvTiHSjkYozSN4TBtB8s';
const SHEET_NAME = 'Logs'; // Must match the worksheet created earlier.

function doGet(e) {
  if (!e || !e.parameter) {
    return ContentService.createTextOutput('Missing required data').setMimeType(ContentService.MimeType.TEXT);
  }

  const employeeName = (e.parameter.employee_name || '').trim();
  const attendanceType = (e.parameter.attendance_type || '').trim();
  const latitude = (e.parameter.latitude || '').trim();
  const longitude = (e.parameter.longitude || '').trim();

  if (!employeeName || !attendanceType || !latitude || !longitude) {
    return ContentService.createTextOutput('Missing required data').setMimeType(ContentService.MimeType.TEXT);
  }

  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getSheetByName(SHEET_NAME);
  if (!sheet) {
    return ContentService.createTextOutput('Sheet not found').setMimeType(ContentService.MimeType.TEXT);
  }

  sheet.appendRow([employeeName, attendanceType, new Date(), latitude, longitude]);

  return ContentService.createTextOutput('Attendance recorded successfully').setMimeType(ContentService.MimeType.TEXT);
}
```

3. Save the script and set a clear project name (e.g., `Attendance Backend`).

4. Click **Deploy → Test deployments → Select type: Web app → Deploy**.
5. Set the deployment access to **Anyone**.
6. Copy the Web App URL (it will look similar to `https://script.google.com/macros/s/.../exec`).  
   This URL goes into the `APPS_SCRIPT_URL` constant in `index.html`.

---

## 3. Configure the frontend

1. Open `index.html` and update the configuration block near the top of the script:
   - `APPS_SCRIPT_URL`: Paste the Web App URL from step 2.
   - `OFFICE_LAT` & `OFFICE_LNG`: Defaults are `30.108528, 31.337788` for testing—replace with your exact coordinates.
   - `ALLOWED_DISTANCE`: Defaults to `1000` meters (perimeter for check in/out). Adjust to your actual policy.
   - `COMPANY_LOGO_URL`: Defaults to the absolute GitHub Pages URL for `assets/logo.svg`; replace with any HTTPS-accessible logo you prefer.
2. Test locally by opening `index.html` in a browser.  
   Ensure:
   - Location permissions are granted.
   - The success and error messages display correctly.
   - Rows appear in the Google Sheet after check-in/out.

---

## 4. Deploy to GitHub Pages

1. Commit and push `index.html` (and any supporting files) to the `main` branch of the `Attendance-Zoom` repository.
2. In GitHub, navigate to **Settings → Pages**.
3. Under **Build and deployment**, choose:
   - Source: `Deploy from a branch`
   - Branch: `main`, folder `/ (root)`
4. Save the configuration. Within a few minutes, GitHub Pages will provide your live URL.
5. Share the URL with employees. The app will communicate with the Apps Script backend in real time.

---

## 5. Ongoing maintenance tips

- Monitor the Google Sheet for capacity (it can handle tens of thousands of rows).
- Protect the sheet or move data periodically if needed.
- Re-deploy the Apps Script whenever you edit the backend code.
- Consider creating filters or pivot tables in the sheet for quick analytics.
- Regularly verify that geolocation permissions and HTTPS are functioning in modern browsers (Chrome, Edge, Safari, Firefox).
- The distance indicator displayed below the message area represents the live Haversine distance (in meters) between the employee’s GPS reading and the configured office coordinates; values above `ALLOWED_DISTANCE` will block submissions.

---

## 6. Troubleshooting tips

- **“Request timed out. Please try again.”**
  - Ensure the Apps Script deployment access is set to “Anyone” and that the URL pasted into `APPS_SCRIPT_URL` ends with `/exec`.
  - Re-deploy the script whenever you change the code; otherwise the endpoint may return stale or no responses.
  - Test the Apps Script URL directly in a browser with sample parameters to confirm it responds instantly.
- **Logo looks distorted or fails to load**
  - Confirm `assets/logo.svg` exists on the branch served by GitHub Pages (or change `COMPANY_LOGO_URL` to another HTTPS-hosted image).
  - The UI now uses an absolute URL to avoid cache issues; keep replacements square to preserve the curved frame.
- **Distance display seems incorrect**
  - Double-check `OFFICE_LAT`/`OFFICE_LNG`, and ensure the user grants high-accuracy GPS permission on mobile.
- **Location request timed out repeatedly**
  - The app now retries with a lower-accuracy, longer timeout after the first failure. If it still times out, verify the browser has permission to access location on HTTPS and that the device can see Wi-Fi/GPS signals.

That’s it! The system is ready for production use across web and mobile browsers.

