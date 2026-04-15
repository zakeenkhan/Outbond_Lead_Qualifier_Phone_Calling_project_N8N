# Outbond_Lead_Qualifier_Phone_Calling_project_N8N

![Workflow Screenshot](outbound%20lead%20Qualifier.JPG)

## Overview

This N8N workflow automates outbound lead qualification through AI-powered phone calls using Vapi. It captures lead information from a web form, validates phone numbers, initiates intelligent phone conversations with prospects, and logs call outcomes to Google Sheets for follow-up.

## How This Workflow Helps

- **Automated Lead Qualification**: Eliminates manual calling by using AI to qualify leads 24/7
- **Instant Response**: Leads are contacted immediately after form submission, increasing conversion rates
- **Data-Driven Insights**: All call outcomes are logged systematically for analysis and CRM integration
- **Cost Efficiency**: Reduces human labor costs while maintaining high-volume outreach
- **Scalability**: Can handle unlimited leads simultaneously without human intervention
- **Consistent Quality**: AI delivers the same professional message every time

## ROI Benefits

### Time Savings
- **Manual Calling**: ~5-10 minutes per lead including dialing, conversation, and logging
- **Automated Workflow**: <1 minute per lead (setup time only)
- **Time Saved**: 80-90% reduction in qualification time

### Cost Efficiency
- **Human SDR Cost**: $3,000-$5,000/month per employee
- **Vapi API Cost**: Pay-per-minute (typically $0.10-$0.30/minute)
- **N8N Hosting**: $20-$50/month (self-hosted free)
- **Estimated Savings**: 70-90% reduction in qualification costs

### Conversion Impact
- **Immediate Follow-up**: 400% higher response rate vs delayed follow-up
- **24/7 Availability**: Capture leads outside business hours
- **Consistent Engagement**: No human fatigue or inconsistency

## How to Approach This Workflow

### Prerequisites
1. **N8N Instance**: Self-hosted or cloud version
2. **Vapi Account**: API key, assistant ID, and phone number ID
3. **Google Sheets**: For logging call results
4. **Form Integration**: Webhook-enabled form (N8N form trigger or external)

### Setup Steps
1. **Import Workflow**: Load the JSON file into your N8N instance
2. **Configure Credentials**:
   - Vapi Bearer Token (HTTP Bearer Auth)
   - Google Sheets OAuth2 API
3. **Update Vapi Settings**:
   - Replace `assistantId` with your Vapi assistant ID
   - Replace `phoneNumberId` with your Vapi phone number ID
4. **Configure Google Sheets**:
   - Update spreadsheet IDs for logging
   - Create sheets for incorrect phones, voicemails, and completed calls
5. **Test Form Submission**: Submit a test form to verify the entire flow
6. **Monitor Results**: Check Google Sheets for logged outcomes

### Customization Options
- **Form Fields**: Add/remove fields based on your qualification criteria
- **AI Script**: Customize the Vapi assistant's conversation script
- **Call Logic**: Add additional branches for different call outcomes
- **CRM Integration**: Replace Google Sheets with direct CRM API calls
- **Notification**: Add email/SMS alerts for high-priority leads

## Workflow Nodes Explained

### 1. On form submission (Form Trigger)
- **Purpose**: Captures lead data when someone submits the web form
- **Fields Collected**: Name, Phone Number, Email, Company name, Role, Request, Company Size
- **Output**: JSON object with all form data

### 2. Standerized (Code Node)
- **Purpose**: Normalizes phone number formats for consistency
- **Logic**:
  - Strips all non-digit characters
  - Validates 10-digit numbers (standard format)
  - Handles 11-digit numbers with country code "1" (US/Canada)
  - Flags invalid formats
- **Output**: Cleaned phone number or "Incorrect format" error

### 3. Incorrect phone? (IF Node)
- **Purpose**: Routes leads based on phone number validity
- **Condition**: Checks if phone equals "incorrect format"
- **True Path**: Logs to incorrect phone sheet
- **False Path**: Proceeds to call initiation

### 4. log incorrect phone (Google Sheets)
- **Purpose**: Records leads with invalid phone numbers for manual review
- **Operation**: Append row to Google Sheets
- **Data Logged**: Phone number and form data

### 5. Calling Customer/vapi (HTTP Request)
- **Purpose**: Initiates AI-powered phone call via Vapi API
- **Method**: POST to `https://api.vapi.ai/call`
- **Payload Includes**:
  - `assistantId`: Your Vapi AI assistant configuration
  - `phoneNumberId`: Your Vapi phone number
  - `customer.number`: Lead's phone number with +1 prefix
  - `assistantOverrides.variableValues`: Lead context (name, company, request)
- **Output**: Call object with call ID for tracking

### 6. Wait (Wait Node)
- **Purpose**: Pauses execution to allow call to initiate
- **Duration**: Configurable delay (default: immediate)

### 7. Get Call details (HTTP Request)
- **Purpose**: Fetches current call status from Vapi API
- **Method**: GET to `https://api.vapi.ai/call/{call_id}`
- **Output**: Call status, duration, transcript, ended reason

### 8. Limit (Limit Node)
- **Purpose**: Prevents infinite polling loops
- **Setting**: Maximum iterations (safety mechanism)

### 9. Ended? (IF Node)
- **Purpose**: Checks if call has completed
- **Condition**: `status == "ended"`
- **True Path**: Proceeds to outcome categorization
- **False Path**: Returns to polling

### 10. pooling (Wait Node)
- **Purpose**: Polling delay between status checks
- **Loop**: Returns to "Get Call details" if not ended
- **Duration**: Configurable polling interval

### 11. voicemail? (IF Node)
- **Purpose**: Categorizes call outcome
- **Condition**: `endedReason` contains "voicemail"
- **True Path**: Logs to voicemail sheet
- **False Path**: Logs to completed calls sheet

### 12. log voicemail (Google Sheets)
- **Purpose**: Records voicemail outcomes for follow-up
- **Operation**: Append row to Google Sheets
- **Data Logged**: Status, call details, lead information

### 13. log complete (Google Sheets)
- **Purpose**: Records successfully completed calls
- **Operation**: Append row to different Google Sheets sheet
- **Data Logged**: Status, call details, lead information

## How the Workflow Works

### Step-by-Step Process

1. **Lead Submission**
   - Prospect fills out web form with contact information
   - Form trigger captures data and passes to workflow

2. **Phone Validation**
   - Code node standardizes phone number format
   - Invalid numbers are flagged and logged separately

3. **Call Initiation**
   - Valid leads trigger Vapi API call
   - AI assistant is configured with lead context (name, company, request)
   - Phone call is placed to the lead's number

4. **Call Monitoring**
   - Workflow polls Vapi API for call status
   - Waits for call to complete (ended status)
   - Safety limit prevents infinite loops

5. **Outcome Categorization**
   - Checks if call reached voicemail
   - Separates voicemail outcomes from completed conversations

6. **Result Logging**
   - Voicemails logged for manual follow-up
   - Completed calls logged with full details
   - All data stored in Google Sheets for analysis

### Data Flow
```
Form → Phone Validation → Vapi Call → Status Polling → Outcome Check → Google Sheets Logging
```

## Technical Requirements

- **N8N Version**: Compatible with current N8N versions
- **APIs Required**:
  - Vapi API (Bearer token authentication)
  - Google Sheets API (OAuth2)
- **External Services**:
  - Vapi (AI phone calling platform)
  - Google Sheets (data storage)
- **Network**: Stable internet connection for API calls

## Best Practices

- **Test Thoroughly**: Use test phone numbers before production deployment
- **Monitor Costs**: Track Vapi API usage to manage expenses
- **Review Logs**: Regularly check Google Sheets for quality assurance
- **Update Scripts**: Periodically refine AI assistant conversation scripts
- **Backup Data**: Maintain backups of Google Sheets data
- **Rate Limiting**: Consider adding delays for high-volume campaigns

## Troubleshooting

- **Calls Not Initiating**: Check Vapi API credentials and phone number ID
- **Phone Validation Issues**: Review code node logic for your region's format
- **Polling Loops**: Verify limit node settings and API response format
- **Google Sheets Errors**: Confirm OAuth2 credentials and spreadsheet permissions
- **Missing Data**: Check field mappings between form and Vapi variables

## Future Enhancements

- Add CRM integration (Salesforce, HubSpot, Pipedrive)
- Implement SMS follow-up for voicemails
- Add sentiment analysis from call transcripts
- Create dashboard for real-time metrics
- Add email notifications for qualified leads
- Implement A/B testing for call scripts
- Add time zone awareness for optimal call timing

