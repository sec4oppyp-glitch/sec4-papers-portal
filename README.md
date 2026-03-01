# Sec 4 Past Year Papers Portal

A clean, single-page portal for students to request access to past year papers, submit papers, and contact the admin.

## Features

- **Access Request Form** — Auto-grants Google Drive access and logs to spreadsheet
- **Paper Submission Form** — Netlify form with file upload (PDF only)
- **Contact Form** — Netlify form with reply-to functionality
- **Mobile responsive** — Works on all devices
- **Inline validation** — Real-time form validation
- **No page reload** — Success messages shown instantly

## Tech Stack

- Pure HTML, CSS, JavaScript (single file)
- Google Apps Script (for access requests)
- Netlify Forms (for submissions and contact)
- No frameworks, no build step

---

## Deployment Guide

### Step 1: Deploy to Netlify from GitHub

1. **Create a GitHub repository**
   - Go to [github.com](https://github.com) and create a new repository
   - Name it `sec4-papers-portal` (or any name you prefer)

2. **Upload the files**
   - Upload `index.html`, `netlify.toml`, `README.md`, and `appscript.md`
   - Or use Git: `git init`, `git add .`, `git commit -m "Initial commit"`, `git push`

3. **Connect to Netlify**
   - Go to [netlify.com](https://netlify.com) and log in
   - Click **Add new site** → **Import an existing project**
   - Select **GitHub** and authorize Netlify
   - Choose your repository
   - Build settings:
     - Build command: *(leave empty)*
     - Publish directory: `.`
   - Click **Deploy site**

4. **Your site is live!**
   - Netlify will give you a URL like `https://your-site-name.netlify.app`
   - You can customize this in **Site settings** → **Domain management**

---

### Step 2: Set Up Google Apps Script

This handles access requests — logging to sheet, granting Drive access, and emailing admin.

#### 2.1 Create the Script Project

1. Go to [script.google.com](https://script.google.com)
2. Click **New project**
3. Delete the default `myFunction()` code
4. Copy the **entire code** from `appscript.md` and paste it into the editor

#### 2.2 Configure the Script

1. **Set your Google Drive Folder ID:**
   - Find your Google Drive folder (where papers are stored)
   - Open the folder — the URL looks like: `https://drive.google.com/drive/folders/1ABC123xyz`
   - Copy the ID part (`1ABC123xyz`)
   - In the script, replace:
     ```javascript
     const FOLDER_ID = "YOUR_GOOGLE_DRIVE_FOLDER_ID_HERE";
     ```
     with:
     ```javascript
     const FOLDER_ID = "1ABC123xyz";
     ```

2. **Verify admin email:**
   - The default is `sec4oppyp@gmail.com`
   - Change if needed:
     ```javascript
     const ADMIN_EMAIL = "sec4oppyp@gmail.com";
     ```

#### 2.3 Save and Test

1. Click **Save** (floppy disk icon) or press `Ctrl+S`
2. Run the test function:
   - Select the `testSetup` function in the dropdown
   - Click **Run**
   - Check the logs (View → Logs) to verify folder access works

#### 2.4 Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Type" and select **Web app**
3. Configure:
   - **Description**: "Sec 4 Papers Access Handler"
   - **Execute as**: Me (your Google account)
   - **Who has access**: Anyone
4. Click **Deploy**
5. Review permissions (click through the warnings)
6. **Copy the Web App URL** (looks like `https://script.google.com/macros/s/ABC123/exec`)

#### 2.5 Update Your Website

1. Go back to your `index.html` file
2. Find this line near the top of the `<script>` section:
   ```javascript
   const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';
   ```
3. Replace with your actual Web App URL:
   ```javascript
   const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/ABC123/exec';
   ```
4. Commit and push the change to GitHub
5. Netlify will auto-deploy the update

---

### Step 3: Set Up Netlify Form Notifications

Configure email notifications for the two Netlify forms.

#### 3.1 Paper Contribution Form (`contribute`)

1. In Netlify dashboard, go to **Forms**
2. You should see the "contribute" form (appears after first submission, or you can set it up now)
3. Click on the form name
4. Go to **Form notifications** → **Add notification** → **Email notification**
5. Configure:
   - **Email**: `sec4oppyp@gmail.com`
   - **Subject**: `New Paper Contribution — {{name}}`
6. Click **Save**

#### 3.2 Contact Form (`contact`)

1. In Netlify dashboard, go to **Forms**
2. Click on the "contact" form
3. Go to **Form notifications** → **Add notification** → **Email notification**
4. Configure:
   - **Email**: `sec4oppyp@gmail.com`
   - **Subject**: `New Contact Message — {{name}}`
5. Click **Save**

> **Note:** The contact form automatically uses the submitted email as reply-to, so you can just hit "Reply" in your email client.

---

## How It All Works

| Action | How It Works | Manual Step? |
|--------|--------------|--------------|
| Access request logged | Apps Script appends to Google Sheet | ❌ Automatic |
| Drive access granted | Apps Script calls Drive API | ❌ Automatic |
| Admin notified (access) | Apps Script sends email | ❌ Automatic |
| Paper contribution received | Netlify Forms | ❌ Automatic |
| Admin notified (paper) | Netlify email notification | ❌ Automatic |
| Contact message received | Netlify Forms | ❌ Automatic |
| Admin notified (contact) | Netlify email notification | ❌ Automatic |

---

## How to Revoke Drive Access Manually

If you need to remove someone's access:

1. Go to [Google Drive](https://drive.google.com)
2. Find your papers folder
3. Right-click → **Share**
4. Click the dropdown next to the person's name
5. Select **Remove access**

---

## File Structure

```
sec4-papers-portal/
├── index.html       # Main page (HTML + CSS + JS)
├── netlify.toml     # Netlify configuration
├── appscript.md     # Google Apps Script code
└── README.md        # This file
```

---

## Troubleshooting

### Access form not working

1. Check browser console for errors
2. Verify `APPS_SCRIPT_URL` in `index.html` is correct
3. Make sure the Apps Script is deployed and set to "Anyone" can access
4. Check Apps Script logs (View → Executions)

### Drive access not being granted

1. Verify `FOLDER_ID` is correct in the Apps Script
2. Make sure the folder is shared appropriately in Google Drive
3. Check the Apps Script has permission to access Drive

### Not receiving emails

1. Check spam/junk folders
2. Verify `ADMIN_EMAIL` is correct in both Apps Script and Netlify
3. For Netlify forms, check Form notifications are configured

### Netlify forms not working

1. Ensure `data-netlify="true"` is on the `<form>` tag
2. Make sure each form has a unique `name` attribute
3. Verify the honeypot field is present
4. Test with a real submission (forms may not appear until first submission)

---

## Customization

### Change Colors

Edit the CSS variables in `index.html`:

```css
:root {
  --navy: #1a3c5e;    /* Primary color */
  --gold: #f0a500;    /* Accent color */
  /* ... */
}
```

### Add More Subjects

Edit the subject dropdown in `index.html` (around line 350):

```html
<select id="contrib-subject" name="subject" class="form-select" required>
  <option value="">Select a subject...</option>
  <option value="New Subject">New Subject</option>
  <!-- ... -->
</select>
```

---

## Security Notes

- Admin email (`sec4oppyp@gmail.com`) never appears in HTML source
- Honeypot fields protect against spam bots
- All forms use HTTPS via Netlify
- Google Apps Script requires authentication to deploy

---

## Support

For issues or questions:
- Check the troubleshooting section above
- Review Netlify docs: [docs.netlify.com](https://docs.netlify.com)
- Review Google Apps Script docs: [developers.google.com/apps-script](https://developers.google.com/apps-script)
