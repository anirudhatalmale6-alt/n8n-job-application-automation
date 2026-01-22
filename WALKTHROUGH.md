# Job Application Automation - n8n Workflow Walkthrough

## Overview
This n8n workflow automates job application preparation by:
- Pulling jobs from a Google Sheet
- Extracting job details and keywords from job postings
- Generating customized cover letters
- Preventing duplicate applications
- Logging all activity
- Sending email notifications

## Setup Instructions

### Step 1: Create Google Sheets

Create a new Google Sheet with **two tabs**:

**Tab 1: "Jobs"** (main input sheet)
| Column | Description |
|--------|-------------|
| job_id | Unique identifier (e.g., JOB001, JOB002) |
| job_title | Role name (e.g., "Security Analyst") |
| company | Company name |
| job_url | Job posting link |
| job_description | Optional - paste description if URL parsing fails |
| location | Job location |
| source | LinkedIn / Company Site / Referral |
| status | Set to "NEW" for new jobs |
| notes | Will be auto-filled by workflow |

**Tab 2: "ApplicationLog"** (auto-populated log)
| Column |
|--------|
| timestamp |
| job_id |
| company |
| job_title |
| status |
| cover_letter_version |
| keywords_matched |
| extraction_method |
| notes |

### Step 2: Import Workflow to n8n

1. Open your n8n instance
2. Go to **Workflows** > **Import from File**
3. Select `job-application-workflow.json`
4. The workflow will appear with all nodes

### Step 3: Configure Credentials

**Google Sheets:**
1. In n8n, go to **Credentials** > **Add Credential** > **Google Sheets OAuth2**
2. Follow OAuth setup (you'll need a Google Cloud project with Sheets API enabled)
3. In the workflow, click each Google Sheets node and select your credential

**Gmail (for notifications):**
1. Add **Gmail OAuth2** credential in n8n
2. Update the "Send Email Notification" node with your credential

### Step 4: Update Sheet ID

1. Open your Google Sheet
2. Copy the ID from the URL: `https://docs.google.com/spreadsheets/d/[THIS_IS_YOUR_ID]/edit`
3. In the workflow, update ALL Google Sheets nodes:
   - "Trigger - New Jobs from Sheet"
   - "Get All Jobs for Duplicate Check"
   - "Update Sheet - Status & Notes"
   - "Update Duplicate Status"
   - "Append to Application Log"

Replace `YOUR_GOOGLE_SHEET_ID_HERE` with your actual ID.

### Step 5: Configure Personal Details

Open the **"CONFIG - Edit Settings Here"** node and update:
- `NOTIFICATION_EMAIL`: Your email for alerts
- `APPLICANT_NAME`: Your full name
- `APPLICANT_EMAIL`: Your email
- `APPLICANT_PHONE`: Your phone
- `APPLICANT_LOCATION`: Your location

---

## How to Use

### Adding New Jobs

1. Add a new row to the "Jobs" sheet
2. Fill in: job_id, job_title, company, job_url, location, source
3. Set status to **"NEW"**
4. The workflow polls every minute and will pick it up

### Status Values

| Status | Meaning |
|--------|---------|
| NEW | Ready to process |
| DRY-RUN | Processed in test mode (no submission) |
| MANUAL REQUIRED | Cover letter ready, needs manual submission |
| APPLIED | Successfully applied (update manually after submitting) |
| SKIPPED | Duplicate detected, skipped |

### Dry Run Mode

**Default: ON** (safe mode)

The workflow starts in dry-run mode. It will:
- Generate cover letters
- Log everything
- Send notifications
- BUT mark as "DRY-RUN" instead of requiring manual submission

To enable live mode:
1. Open "CONFIG - Edit Settings Here" node
2. Set `DRY_RUN_MODE` to `false`
3. Jobs will now be marked "MANUAL REQUIRED"

### Global Pause

To pause all processing:
1. Open "CONFIG - Edit Settings Here" node
2. Set `GLOBAL_PAUSE` to `true`
3. No new jobs will be processed until you set it back to `false`

---

## Workflow Flow

```
[Google Sheet Trigger]
        |
        v
[Check Global Pause] ---(paused)---> [Stop]
        |
        v (not paused)
[Filter NEW Status Only]
        |
        v
[Get All Jobs for Duplicate Check]
        |
        v
[Check for Duplicates]
        |
        +--(duplicate)---> [Mark SKIPPED] --> [Update Sheet]
        |
        v (not duplicate)
[Fetch Job Page]
        |
        v
[Extract Job Info & Keywords]
        |
        v
[Generate Cover Letter]
        |
        v
[Check Dry Run Mode]
        |
        +--(dry run ON)---> [Log as DRY-RUN]
        |
        v (dry run OFF)
[Log as MANUAL REQUIRED]
        |
        v
[Update Sheet Status]
        |
        v
[Append to Application Log]
        |
        v
[Send Email Notification]
        |
        v
[Rate Limit Delay (3s)]
```

---

## Cover Letter Customization

The cover letter generator automatically:
1. Extracts keywords from job descriptions
2. Selects focus area based on keywords:
   - Security/OSINT/Penetration keywords → Offensive security focus
   - Compliance/ISO/GDPR keywords → Compliance focus
   - Python/ML/AI keywords → AI/ML security focus
3. Adjusts skills mentioned accordingly

### To customize the template:

1. Open **"Generate Cover Letter"** node
2. Edit the JavaScript code
3. Modify the `coverLetter` template string
4. Add/change skill mappings in the keyword detection section

---

## Troubleshooting

**Jobs not being picked up:**
- Check status is exactly "NEW" (case-sensitive)
- Verify Google Sheet ID is correct in all nodes
- Check Google Sheets credentials are valid

**Duplicate detection too strict:**
- Edit "Check for Duplicates" node
- Modify comparison logic as needed

**Job page fetch failing:**
- Some sites block automated requests
- Add job_description manually in sheet
- The workflow will use manual description as fallback

**Email not sending:**
- Verify Gmail OAuth2 credential
- Check NOTIFICATION_EMAIL is set correctly

---

## Maintenance Tips

1. **Review DRY-RUN jobs** before switching to live mode
2. **Check ApplicationLog** sheet regularly for errors
3. **Update cover letter template** as your experience grows
4. **Clear old logs** periodically to keep sheet performant

---

## Future Extensions (Not in v1)

- Follow-up email drafting
- Referral outreach templates
- Advanced job matching/scoring
- Integration with specific ATS systems

---

## Support

If you have questions or need modifications, let me know!
