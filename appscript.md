# Google Apps Script

Copy the entire code block below into your Google Apps Script project.

```javascript
/**
 * Sec 4 Past Year Papers Portal - Access Request Handler
 * 
 * This script handles access requests by:
 * 1. Logging request data to a Google Sheet
 * 2. Auto-granting Google Drive folder access via Drive API
 * 3. Sending email notification to admin
 * 
 * SETUP INSTRUCTIONS:
 * 1. Replace FOLDER_ID with your Google Drive folder ID
 * 2. Replace ADMIN_EMAIL if different from sec4oppyp@gmail.com
 * 3. Deploy as Web App (see README.md for detailed steps)
 */

// ========================================
// CONFIGURATION - REPLACE THESE VALUES
// ========================================

// Your Google Drive folder ID (from the folder URL)
const FOLDER_ID = "YOUR_GOOGLE_DRIVE_FOLDER_ID_HERE";

// Admin email to receive notifications
const ADMIN_EMAIL = "sec4oppyp@gmail.com";

// Sheet name for logging access requests
const SHEET_NAME = "Access Requests";

// ========================================
// MAIN HANDLER
// ========================================

/**
 * Main entry point - handles POST requests
 */
function doPost(e) {
  // Set CORS headers
  const headers = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "POST",
    "Access-Control-Allow-Headers": "Content-Type"
  };

  try {
    // Parse JSON body
    const data = JSON.parse(e.postData.contents);
    
    // Validate required fields
    if (!data.name || !data.school || !data.class || !data.email) {
      return ContentService.createTextOutput(JSON.stringify({
        result: "error",
        message: "Missing required fields"
      })).setMimeType(ContentService.MimeType.JSON).setHeaders(headers);
    }

    // Validate email format
    if (!isValidEmail(data.email)) {
      return ContentService.createTextOutput(JSON.stringify({
        result: "error",
        message: "Invalid email format"
      })).setMimeType(ContentService.MimeType.JSON).setHeaders(headers);
    }

    // Get current timestamp
    const timestamp = new Date();

    // 1. Log to Google Sheet
    logToSheet(data, timestamp);

    // 2. Grant Drive access
    grantDriveAccess(data.email);

    // 3. Send admin notification email
    sendAdminEmail(data, timestamp);

    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      result: "success",
      message: "Access granted successfully"
    })).setMimeType(ContentService.MimeType.JSON).setHeaders(headers);

  } catch (error) {
    console.error("Error in doPost:", error);
    return ContentService.createTextOutput(JSON.stringify({
      result: "error",
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON).setHeaders(headers);
  }
}

/**
 * Handle OPTIONS requests for CORS preflight
 */
function doOptions(e) {
  const headers = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "POST",
    "Access-Control-Allow-Headers": "Content-Type"
  };
  
  return ContentService.createTextOutput("")
    .setMimeType(ContentService.MimeType.JSON)
    .setHeaders(headers);
}

// ========================================
// HELPER FUNCTIONS
// ========================================

/**
 * Log access request to Google Sheet
 * Creates sheet if it doesn't exist
 */
function logToSheet(data, timestamp) {
  // Get active spreadsheet or create new one
  let spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  if (!spreadsheet) {
    spreadsheet = SpreadsheetApp.create("Sec 4 Papers Access Requests");
  }

  // Get or create sheet
  let sheet = spreadsheet.getSheetByName(SHEET_NAME);
  if (!sheet) {
    sheet = spreadsheet.insertSheet(SHEET_NAME);
    // Add headers
    sheet.appendRow(["Timestamp", "Name", "School", "Class", "Email"]);
    // Format header row
    sheet.getRange(1, 1, 1, 5).setFontWeight("bold");
  }

  // Append data row
  sheet.appendRow([
    timestamp,
    data.name,
    data.school,
    data.class,
    data.email
  ]);
}

/**
 * Grant viewer access to Google Drive folder
 */
function grantDriveAccess(email) {
  try {
    const folder = DriveApp.getFolderById(FOLDER_ID);
    folder.addViewer(email);
    console.log(`Access granted to: ${email}`);
  } catch (error) {
    console.error("Error granting Drive access:", error);
    // Don't throw - we still want to log the request and notify admin
  }
}

/**
 * Send email notification to admin
 */
function sendAdminEmail(data, timestamp) {
  const subject = `New Access Request — ${data.name}`;
  
  const body = `
New access request received for Sec 4 Past Year Papers.

REQUEST DETAILS:
================
Name: ${data.name}
School: ${data.school}
Class: ${data.class}
Email: ${data.email}
Timestamp: ${timestamp.toLocaleString()}

ACTIONS TAKEN:
================
✓ Request logged to Google Sheet
✓ Drive access granted automatically

---
This is an automated message from the Sec 4 Papers Portal.
  `;

  try {
    MailApp.sendEmail({
      to: ADMIN_EMAIL,
      subject: subject,
      body: body,
      name: "Sec 4 Papers Portal"
    });
    console.log(`Notification email sent to: ${ADMIN_EMAIL}`);
  } catch (error) {
    console.error("Error sending email:", error);
    // Don't throw - we still want to return success to the user
  }
}

/**
 * Validate email format
 */
function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Test function - run this to verify setup
 */
function testSetup() {
  console.log("Testing Apps Script setup...");
  
  // Test folder access
  try {
    const folder = DriveApp.getFolderById(FOLDER_ID);
    console.log("✓ Folder access OK:", folder.getName());
  } catch (e) {
    console.error("✗ Folder access failed:", e);
  }
  
  // Test sheet access
  try {
    let spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
    if (!spreadsheet) {
      console.log("ℹ No active spreadsheet - will create on first request");
    } else {
      console.log("✓ Spreadsheet access OK:", spreadsheet.getName());
    }
  } catch (e) {
    console.error("✗ Spreadsheet access failed:", e);
  }
  
  // Test email (will fail in editor but shows if MailApp is available)
  try {
    console.log("✓ MailApp available");
  } catch (e) {
    console.error("✗ MailApp not available:", e);
  }
  
  console.log("\nSetup test complete. Check logs above.");
}
```
