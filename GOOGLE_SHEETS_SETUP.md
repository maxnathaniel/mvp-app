# Google Sheets Integration Setup Guide

This guide will help you set up your contact form to send data directly to Google Sheets.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **+ Blank** to create a new spreadsheet
3. Name it something like "Contact Form Submissions"
4. Leave the sheet empty - the script will automatically add headers on the first form submission!

## Step 2: Create Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
2. Delete any existing code in the editor
3. Copy and paste the following code:

```javascript
function doPost(e) {
  try {
    // Get the active spreadsheet
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Check if headers need to be added (if sheet is empty or first cell is empty)
    const lastRow = sheet.getLastRow();
    if (lastRow === 0 || sheet.getRange(1, 1).getValue() === '') {
      const headers = ['Timestamp', 'First Name', 'Last Name', 'Email', 'Company', 'Project Type', 'Timeline', 'Description'];
      sheet.getRange(1, 1, 1, headers.length).setValues([headers]);

      // Format headers (bold, blue background, white text)
      sheet.getRange(1, 1, 1, headers.length)
        .setFontWeight('bold')
        .setBackground('#4285f4')
        .setFontColor('#ffffff');
    }

    // Parse the incoming data
    const data = JSON.parse(e.postData.contents);

    // Prepare the row data in the order of your columns
    const rowData = [
      data.timestamp || new Date().toISOString(),
      data.firstName || '',
      data.lastName || '',
      data.email || '',
      data.company || 'N/A',
      data.projectType || '',
      data.timeline || 'N/A',
      data.description || ''
    ];

    // Append the data to the sheet
    sheet.appendRow(rowData);

    // Return success response
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': sheet.getLastRow() }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    // Return error response
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: Test function to verify the script works
function test() {
  const testData = {
    postData: {
      contents: JSON.stringify({
        timestamp: new Date().toISOString(),
        firstName: 'Test',
        lastName: 'User',
        email: 'test@example.com',
        company: 'Test Company',
        projectType: 'web-app',
        timeline: 'asap',
        description: 'This is a test submission'
      })
    }
  };

  Logger.log(doPost(testData).getContent());
}
```

4. Click the **Save** icon (💾) or press `Ctrl+S` (Windows) / `Cmd+S` (Mac)
5. Name your project (e.g., "Contact Form Script")

## Step 3: Deploy the Script

1. Click **Deploy** → **New deployment**
2. Click the gear icon (⚙️) next to "Select type"
3. Choose **Web app**
4. Fill in the deployment settings:
   - **Description**: "Contact Form Handler" (or any description)
   - **Execute as**: "Me"
   - **Who has access**: "Anyone"
5. Click **Deploy**
6. You may need to authorize the script:
   - Click **Authorize access**
   - Choose your Google account
   - Click **Advanced** → **Go to [Your Project Name] (unsafe)**
   - Click **Allow**
7. **IMPORTANT**: Copy the **Web App URL** that appears (it should look like: `https://script.google.com/macros/s/XXXXX/exec`)

## Step 4: Update Your HTML File

1. Open your `index.html` file
2. Find this line near the top of the `<script>` section (around line 820):
   ```javascript
   const GOOGLE_SCRIPT_URL = 'YOUR_GOOGLE_SCRIPT_URL';
   ```
3. Replace `'YOUR_GOOGLE_SCRIPT_URL'` with the Web App URL you copied in Step 3:
   ```javascript
   const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/XXXXX/exec';
   ```

## Step 5: Test Your Form

1. Open your `index.html` file in a web browser
2. Fill out the contact form with test data
3. Submit the form
4. Check your Google Sheet - you should see a new row with the submitted data!

## Troubleshooting

### Form submission shows success but no data in sheet
- Make sure you deployed the script as a **Web app** with "Anyone" access
- Verify the Web App URL is correct in your HTML file
- Check the Google Apps Script execution logs: **Extensions** → **Apps Script** → **Executions**

### Authorization errors
- Make sure you authorized the script to access your Google account
- Try redeploying the script: **Deploy** → **Manage deployments** → Click the pencil icon → **Deploy**

### CORS errors in browser console
- This is normal! The script uses `mode: 'no-cors'` which prevents reading the response but still works
- As long as data appears in your sheet, everything is working correctly

## Making Updates

If you need to modify the script:
1. Make changes in the Apps Script editor
2. Click **Save**
3. Click **Deploy** → **Manage deployments**
4. Click the pencil icon (✏️) next to your deployment
5. Change the version to **New version**
6. Click **Deploy**
7. Update the URL in your HTML file if it changed

## Data Privacy

⚠️ **Important**:
- This setup stores all form submissions in your Google Sheet
- Make sure your Google Sheet is set to private (not shared publicly)
- Consider adding email notifications for new submissions
- Regularly review and clean up old submissions

## Optional: Add Email Notifications

To get notified via email when someone submits the form, add this to your Apps Script:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Check if headers need to be added (if sheet is empty or first cell is empty)
    const lastRow = sheet.getLastRow();
    if (lastRow === 0 || sheet.getRange(1, 1).getValue() === '') {
      const headers = ['Timestamp', 'First Name', 'Last Name', 'Email', 'Company', 'Project Type', 'Timeline', 'Description'];
      sheet.getRange(1, 1, 1, headers.length).setValues([headers]);

      // Format headers (bold, blue background, white text)
      sheet.getRange(1, 1, 1, headers.length)
        .setFontWeight('bold')
        .setBackground('#4285f4')
        .setFontColor('#ffffff');
    }

    const data = JSON.parse(e.postData.contents);

    const rowData = [
      data.timestamp || new Date().toISOString(),
      data.firstName || '',
      data.lastName || '',
      data.email || '',
      data.company || 'N/A',
      data.projectType || '',
      data.timeline || 'N/A',
      data.description || ''
    ];

    sheet.appendRow(rowData);

    // Send email notification
    const emailSubject = `New Contact Form Submission from ${data.firstName} ${data.lastName}`;
    const emailBody = `
New contact form submission:

Name: ${data.firstName} ${data.lastName}
Email: ${data.email}
Company: ${data.company || 'N/A'}
Project Type: ${data.projectType}
Timeline: ${data.timeline || 'N/A'}

Message:
${data.description}

Submitted at: ${data.timestamp}
    `;

    MailApp.sendEmail({
      to: 'your-email@example.com', // Replace with your email
      subject: emailSubject,
      body: emailBody
    });

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': sheet.getLastRow() }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

Don't forget to replace `'your-email@example.com'` with your actual email address!
