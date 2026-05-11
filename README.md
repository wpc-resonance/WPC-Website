# Wild Petal Collective — Static Marketing Website

This project is a static, mobile-responsive marketing website prototype for Wild Petal Collective, ready for GitHub Pages deployment.

## Project Structure

```text
wild-petal-website/
  index.html
  design.html
  talismans.html
  styles.css
  assets/
    images/
      .gitkeep
    mockups/
      .gitkeep
  copy/
    home.md
    design.md
    talismans.md
  README.md
```

## What Each File Does

- `index.html` — umbrella brand homepage
- `design.html` — plant styling and interiors services page + Design leads form
- `talismans.html` — non-technical wearable concept page + Talismans waitlist form
- `styles.css` — shared styling system and responsive layout
- `copy/*.md` — editable source copy for each page
- `assets/images/` — production website images
- `assets/mockups/` — visual references and draft mockups

## Deploy to GitHub Pages

1. Create a new GitHub repository named **WPC Website** under your `WPC_resonance` account.
2. Upload the full `wild-petal-website` contents to the repository root (not inside another nested folder).
3. In GitHub, go to **Settings → Pages**.
4. Under **Build and deployment**, set:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main` (or `master`), folder `/ (root)`
5. Save, then wait for GitHub Pages to publish.
6. Open the generated Pages URL and verify navigation and forms.

## Google Forms + Google Sheets Workflow (Current Setup)

This site currently posts to Google Form endpoints from [`design.html`](design.html) and [`talismans.html`](talismans.html).

- Design page now includes:
  - consultation intake
  - one-time order intake
  - recurring order intake (weekly/biweekly/monthly/custom)
- Talismans page includes waitlist intake.

### Google Form Mapping Requirement

For full structured data capture, map every live field in [`design.html`](design.html) to a real Google Form field name (`entry.#########`).

Right now, this project already maps:
- `entry.103622328` (email)
- `entry.2023600763` (compiled summary string)

Any non-`entry.*` field names must be replaced with matching Google Form entry IDs if you want those fields in separate Sheet columns.

## Automatic Invoice Draft + Internal Email via Apps Script

Use your Google Form response Sheet to:
- send an internal notification to `hello@wildpetalcollective.com`
- generate a draft invoice Google Doc for review
- store the draft Doc URL in the Sheet

### 1) Prepare the Sheet

In the response Sheet connected to your Google Form:
1. Add a new tab named `Settings`.
2. Put these values:
   - `A1`: `INVOICE_TEMPLATE_DOC_ID`
   - `B1`: your Google Doc template ID
   - `A2`: `INVOICE_OUTPUT_FOLDER_ID`
   - `B2`: Drive folder ID where draft invoices should be saved
   - `A3`: `INTERNAL_NOTIFICATION_EMAIL`
   - `B3`: `hello@wildpetalcollective.com`

3. In the form response tab, add these columns at the end:
   - `Invoice Number`
   - `Invoice Doc URL`
   - `Order Status`
   - `Reviewed By`
   - `Reviewed At`

Suggested status flow: `New` → `Reviewed` → `Sent to Client`.

### 2) Create Invoice Template Doc

Create a Google Doc with merge tokens like:

- `{{INVOICE_NUMBER}}`
- `{{SUBMISSION_DATE}}`
- `{{CUSTOMER_NAME}}`
- `{{CUSTOMER_EMAIL}}`
- `{{CUSTOMER_PHONE}}`
- `{{ORDER_TYPE}}`
- `{{ORDER_FREQUENCY}}`
- `{{START_DATE}}`
- `{{END_DATE}}`
- `{{ORDER_DESCRIPTION}}`
- `{{DISTRIBUTOR_PRIMARY}}`
- `{{DISTRIBUTOR_BACKUP}}`
- `{{INTERNAL_NOTES}}`
- `{{SUMMARY}}`

### 3) Add Apps Script

From the response Sheet: **Extensions → Apps Script**, then paste this script:

```javascript
function onFormSubmit(e) {
  var sheet = e.range.getSheet();
  var row = e.range.getRow();
  var header = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
  var rowValues = sheet.getRange(row, 1, 1, sheet.getLastColumn()).getValues()[0];

  var data = {};
  header.forEach(function (key, i) {
    data[String(key).trim()] = rowValues[i];
  });

  var settings = getSettings_();
  var invoiceNumber = buildInvoiceNumber_(row);
  var now = new Date();

  var merge = {
    INVOICE_NUMBER: invoiceNumber,
    SUBMISSION_DATE: formatDate_(now),
    CUSTOMER_NAME: pick_(data, ['Full Name', 'contact_name']),
    CUSTOMER_EMAIL: pick_(data, ['Email', 'entry.103622328']),
    CUSTOMER_PHONE: pick_(data, ['Phone', 'contact_phone']),
    ORDER_TYPE: pick_(data, ['Order Type', 'order_type']),
    ORDER_FREQUENCY: pick_(data, ['Order Frequency', 'order_frequency']),
    START_DATE: pick_(data, ['Preferred Start Date', 'order_start_date']),
    END_DATE: pick_(data, ['Recurring End Date (optional)', 'order_end_date']),
    ORDER_DESCRIPTION: pick_(data, ['Order Description', 'order_description']),
    DISTRIBUTOR_PRIMARY: pick_(data, ['Preferred Distributor (if known)', 'distributor_primary']),
    DISTRIBUTOR_BACKUP: pick_(data, ['Backup Distributor Option', 'distributor_backup']),
    INTERNAL_NOTES: pick_(data, ['Internal Coordination Notes (optional)', 'internal_notes']),
    SUMMARY: pick_(data, ['entry.2023600763'])
  };

  var invoiceDoc = createInvoiceDoc_(settings.INVOICE_TEMPLATE_DOC_ID, settings.INVOICE_OUTPUT_FOLDER_ID, merge);

  writeBack_(sheet, header, row, {
    'Invoice Number': invoiceNumber,
    'Invoice Doc URL': invoiceDoc.getUrl(),
    'Order Status': 'New'
  });

  sendInternalNotification_(settings.INTERNAL_NOTIFICATION_EMAIL, merge, invoiceDoc.getUrl());
}

function createInvoiceDoc_(templateId, folderId, merge) {
  var fileName = 'WPC Invoice Draft - ' + merge.INVOICE_NUMBER + ' - ' + (merge.CUSTOMER_NAME || 'Customer');
  var copy = DriveApp.getFileById(templateId).makeCopy(fileName, DriveApp.getFolderById(folderId));
  var doc = DocumentApp.openById(copy.getId());
  var body = doc.getBody();

  Object.keys(merge).forEach(function (key) {
    body.replaceText('{{' + key + '}}', sanitize_(merge[key]));
  });

  doc.saveAndClose();
  return doc;
}

function sendInternalNotification_(to, merge, invoiceUrl) {
  var subject = '[WPC] New ' + (merge.ORDER_FREQUENCY || 'Order') + ' Submission - ' + (merge.CUSTOMER_NAME || 'Unknown');
  var lines = [
    'A new design order/consultation submission was received.',
    '',
    'Invoice Draft: ' + invoiceUrl,
    'Invoice #: ' + merge.INVOICE_NUMBER,
    'Customer: ' + merge.CUSTOMER_NAME,
    'Email: ' + merge.CUSTOMER_EMAIL,
    'Phone: ' + merge.CUSTOMER_PHONE,
    'Order Type: ' + merge.ORDER_TYPE,
    'Frequency: ' + merge.ORDER_FREQUENCY,
    'Start: ' + merge.START_DATE,
    'End: ' + merge.END_DATE,
    'Primary Distributor: ' + merge.DISTRIBUTOR_PRIMARY,
    'Backup Distributor: ' + merge.DISTRIBUTOR_BACKUP,
    '',
    'Order Description:',
    merge.ORDER_DESCRIPTION,
    '',
    'Internal Notes:',
    merge.INTERNAL_NOTES,
    '',
    'Summary:',
    merge.SUMMARY
  ];

  MailApp.sendEmail({
    to: to,
    subject: subject,
    body: lines.join('\n')
  });
}

function writeBack_(sheet, header, row, values) {
  Object.keys(values).forEach(function (colName) {
    var index = header.indexOf(colName);
    if (index === -1) return;
    sheet.getRange(row, index + 1).setValue(values[colName]);
  });
}

function getSettings_() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.getSheetByName('Settings');
  var values = sh.getRange(1, 1, 20, 2).getValues();
  var map = {};
  values.forEach(function (r) {
    if (r[0]) map[String(r[0]).trim()] = String(r[1]).trim();
  });
  return map;
}

function buildInvoiceNumber_(row) {
  var d = new Date();
  var y = d.getFullYear();
  var m = ('0' + (d.getMonth() + 1)).slice(-2);
  var day = ('0' + d.getDate()).slice(-2);
  return 'WPC-' + y + m + day + '-' + row;
}

function pick_(obj, keys) {
  for (var i = 0; i < keys.length; i++) {
    var v = obj[keys[i]];
    if (v !== undefined && v !== null && String(v).trim() !== '') return String(v).trim();
  }
  return '';
}

function sanitize_(value) {
  return value == null ? '' : String(value);
}

function formatDate_(date) {
  return Utilities.formatDate(date, Session.getScriptTimeZone(), 'yyyy-MM-dd HH:mm');
}
```

### 4) Create Trigger

In Apps Script:
1. Open **Triggers**.
2. Add trigger for [`onFormSubmit()`](README.md:214).
3. Event source: **From spreadsheet**.
4. Event type: **On form submit**.
5. Authorize permissions.

### 5) Review + Send Client Invoice

1. New row arrives from customer.
2. Script writes invoice number + draft doc URL.
3. Team reviews draft doc, edits pricing/terms.
4. Export/send to customer manually (or add a second “Send Invoice” automation later).

## Distributor Routing + Backup Vendor Workflow

Use Google Sheets as your internal “routing brain” between form submission and final vendor order placement.

### 1) Add Two Internal Tabs

Create these tabs in the same spreadsheet:

1. `VendorMatrix`
2. `VendorTemplates`

### 2) `VendorMatrix` Structure (Recommendation Logic)

Add columns:

- `Vendor Name`
- `Vendor Type` (local nursery / wholesale grower / floral wholesaler / hardgoods)
- `Service Area`
- `Order Categories` (plant styling, maintenance, seasonal florals, vessels, etc.)
- `Min Order`
- `Lead Time Days`
- `Rush Capable` (TRUE/FALSE)
- `Recurring Capable` (TRUE/FALSE)
- `Quality Tier` (1–5)
- `Cost Tier` (1–5)
- `Reliability Tier` (1–5)
- `Seasonality Notes`
- `Preferred For` (comma list: weekly, premium installs, event florals, etc.)
- `Backup Rank` (1 = best backup)
- `Active` (TRUE/FALSE)

How to use:
- Primary vendor = best match for `Order Type` + `Order Frequency` + required lead time.
- Backup vendor = next best active row by same criteria, sorted by `Backup Rank`.

### 3) `VendorTemplates` Structure (Prefill Targets)

Add columns:

- `Vendor Name`
- `Template Type` (Google Form / Google Doc / External URL)
- `Template Base URL`
- `Field Map JSON`
- `Order Method` (email/web portal/phone)
- `Order Email`
- `Order Instructions`

`Field Map JSON` example:

```json
{
  "customer_name": "entry.123456",
  "customer_email": "entry.987654",
  "order_description": "entry.555555",
  "order_frequency": "entry.222222"
}
```

### 4) Add Routing Columns to Form Response Tab

Add these columns to your response sheet:

- `Primary Vendor`
- `Backup Vendor`
- `Primary Vendor Order Link`
- `Backup Vendor Order Link`
- `Vendor Routing Notes`
- `Vendor Order Status`

Recommended statuses:
- `Needs Review`
- `Ready to Order`
- `Ordered with Primary`
- `Ordered with Backup`
- `Completed`

### 5) Apps Script: Auto-Recommend + Generate Prefilled Links

Add this helper flow to your existing script after invoice generation.

```javascript
function recommendVendors_(order) {
  var vendorRows = getSheetData_('VendorMatrix');
  var active = vendorRows.filter(function (v) { return String(v['Active']).toUpperCase() === 'TRUE'; });

  var scored = active.map(function (v) {
    var score = 0;
    if ((v['Order Categories'] || '').toLowerCase().indexOf((order.orderType || '').toLowerCase()) > -1) score += 30;
    if ((v['Preferred For'] || '').toLowerCase().indexOf((order.frequency || '').toLowerCase()) > -1) score += 20;
    if (String(v['Recurring Capable']).toUpperCase() === 'TRUE' && /weekly|biweekly|monthly|recurring/i.test(order.frequency || '')) score += 20;

    score += Number(v['Reliability Tier'] || 0) * 5;
    score += Number(v['Quality Tier'] || 0) * 3;
    score -= Number(v['Lead Time Days'] || 0);

    if (/rush/i.test(order.timeline || '') && String(v['Rush Capable']).toUpperCase() === 'TRUE') score += 10;
    return { vendor: v, score: score };
  });

  scored.sort(function (a, b) { return b.score - a.score; });
  return {
    primary: scored[0] ? scored[0].vendor : null,
    backup: scored[1] ? scored[1].vendor : null
  };
}

function buildPrefillLink_(vendorName, order) {
  var templates = getSheetData_('VendorTemplates');
  var row = templates.find(function (t) { return t['Vendor Name'] === vendorName; });
  if (!row) return '';

  var base = row['Template Base URL'] || '';
  var map = JSON.parse(row['Field Map JSON'] || '{}');
  var params = [];

  Object.keys(map).forEach(function (key) {
    var fieldId = map[key];
    var value = order[key] || '';
    params.push(encodeURIComponent(fieldId) + '=' + encodeURIComponent(value));
  });

  return base + (base.indexOf('?') > -1 ? '&' : '?') + params.join('&');
}

function getSheetData_(name) {
  var sh = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(name);
  var values = sh.getDataRange().getValues();
  var header = values.shift();
  return values.map(function (r) {
    var obj = {};
    header.forEach(function (h, i) { obj[h] = r[i]; });
    return obj;
  });
}
```

Then in [`onFormSubmit()`](README.md:214), after creating invoice data:
- Build an `order` object from the submission.
- Call [`recommendVendors_()`](README.md:349).
- Generate links with [`buildPrefillLink_()`](README.md:377).
- Write those values to routing columns.
- Include both links in your internal email body.

### 6) Daily Team Process (Simple + Reliable)

1. Customer submits order form.
2. Sheet receives row.
3. Script creates invoice draft + recommends primary/backup vendor + prefill links.
4. Internal email arrives with:
   - invoice draft URL
   - primary vendor recommendation
   - backup vendor recommendation
   - one-click prefilled order links
5. Team reviews, adjusts quantities/pricing, places order with primary.
6. If unavailable, place with backup and update `Vendor Order Status`.

This gives you a repeatable flow from customer request → internal review → vendor execution with backup resilience.

## Replace Images

1. Add final images to `assets/images/`.
2. Keep filenames or update image `src` values in HTML pages.
3. Recommended export sizes:
   - Hero images: ~1800px wide JPG/WEBP
   - Card images: ~1000px wide JPG/WEBP
4. Compress images for speed.

## Edit Website Text

1. Edit source messaging in `copy/home.md`, `copy/design.md`, and `copy/talismans.md`.
2. Copy final text into corresponding HTML sections.
3. Commit changes and push to GitHub; Pages redeploys automatically.

## Compliance Summary

- Static files only (HTML/CSS)
- No backend code
- No database in the project
- No analytics/tracking scripts
- No API integrations beyond static form-post endpoints
- Talismans messaging remains non-medical and non-diagnostic
