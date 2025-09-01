# Building-Intelligent-Automation-Solutions-with-Azure-Power-Automate
Azure Power Automate

Two bite-size demos you can reproduce in minutes. This README is written to be copy-paste friendly and classroom-ready.

🧩 What’s Inside

Demo 01: Microsoft Form → email the submitter a receipt.

Demo 02: SharePoint List + CSV import → conditional approval → update status (triggered by item creation).

Tools used: Microsoft Forms, SharePoint, Power Automate (Cloud flows), Outlook (Send email / Send email with options).

✅ Prerequisites

Microsoft 365 tenant with:

Microsoft Forms

SharePoint Online (can create lists)

Power Automate

Outlook (or Exchange Online)

Permission to create flows and SharePoint lists

(Optional) A test mailbox to receive emails

🧪 Demo 01 — Form submission → Email receipt

Goal

When someone submits your Microsoft Form, automatically send them a friendly, formatted email with their answers.

Form (Duplicate & Use)

Use your shared form link (duplicate it first so you’re not editing the original):

https://forms.office.com/Pages/ShareFormPage.aspx?id=oBzDhDusrk6tEVGdgCM-b6YrNZuvrGZJgW-WJlWEYYFUN0FMSTZWTU1VVzNOMlNaUUJKNlE4QUNBSS4u&sharetoken=aXXWnKFJbtwLE8M2lmbL


Ensure your form collects the submitter’s email (Settings → “Record name” or add an “Email” question).

Flow Steps (Power Automate)

Trigger: Microsoft Forms → When a new response is submitted

Form Id: Select your duplicated form

Action: Microsoft Forms → Get response details

Form Id: same as above

Response Id: Response Id from the trigger

Action: Outlook → Send an email (V2)

To: the submitter’s email (e.g., dynamic value Responders' Email or the email question)

Subject: Your form submission receipt

Body (HTML):

<p>Hello,</p>
<p>Here is your form submission details:</p>
<p>
  <strong>date</strong> – @{utcNow()}<br/>
  <strong>topic</strong> – @{outputs('Get_response_details')?['body/YourTopicQuestionId']}<br/>
  <strong>details</strong> – @{outputs('Get_response_details')?['body/YourDetailsQuestionId']}
</p>
<p>Thank you so much!</p>


(Optional) Turn Is HTML on.

Replace YourTopicQuestionId / YourDetailsQuestionId with your actual dynamic tokens from Get response details.

Test it: Submit the form → check inbox.

<img width="430" height="463" alt="image" src="https://github.com/user-attachments/assets/b4a1b310-093b-4fb4-bbc5-88f3a09b87b4" />


🧪 Demo 02 — SharePoint List + CSV → Conditional Approval → Update Status
Goal

Create a SharePoint list, bulk-add items from a CSV, then use a Cloud flow:

Email a receipt to the requester

If product is “expensive”, request manager approval

If approved, update the item’s status (triggered when item is created)

<img width="609" height="560" alt="image" src="https://github.com/user-attachments/assets/28aa5f9d-32e3-4997-b714-4acc6245f5d2" />



 **** Step A — Create SharePoint List ****

Create a list (e.g., Requests) with columns:

Column (Internal Name)	Type	Example
Title	Single line	Laptop Purchase
RequesterEmail	Single line	someone@contoso.com

Product	Choice/Text	Laptop / Phone / Accessory
EstimatedCost	Number	2200
Status	Choice/Text	New, Pending, Approved, Rejected

You can add more fields as needed (CostCenter, Notes, etc.).

Step B — Import CSV (Quick Option)

Prepare a CSV (columns must match list columns by header name):

Title,RequesterEmail,Product,EstimatedCost,Status
Laptop Purchase,someone@contoso.com,Laptop,2200,New
Headset,someone@contoso.com,Accessory,120,New
Phone Upgrade,manager@contoso.com,Phone,1800,New


Import via:

SharePoint: Quick Edit → paste from Excel

or Power Automate / PowerShell / Graph (optional advanced route)

Step C — Build the Flow (Trigger Type)

Cloud Flow: Automated → When an item is created (SharePoint)

Trigger:

Site Address: your site

List Name: Requests

Action 1 — Send an Email (Receipt to Requester)

Outlook → Send an email (V2)

To: RequesterEmail

Subject: We received your request: @{triggerOutputs()?['body/Title']}

Body (HTML):

<p>Hello,</p>
<p>We received your request:</p>
<p>
  <strong>Title</strong> – @{triggerBody()?['Title']}<br/>
  <strong>Product</strong> – @{triggerBody()?['Product']}<br/>
  <strong>Estimated Cost</strong> – @{outputs('Formatted_Cost')}
</p>
<p>Thank you so much!</p>


Action 1.1 — (Optional) Format Currency
Add a Compose action named Formatted_Cost with:

@{formatNumber(triggerBody()?['EstimatedCost'], '$####,###,00')}


This matches your requested expression style. If you use decimals, try '$#,##0.00'.

Action 2 — Condition (Is “Expensive”?)
You can define “expensive” in two ways:

By text: Product equals expensive (case-insensitive):

Condition:

equals(toLower(triggerBody()?['Product']), 'expensive')


By amount: EstimatedCost ≥ 1000:

Condition:

greaterOrEquals(int(triggerBody()?['EstimatedCost']), 1000)


If YES (Expensive) → Approval Path

Action: Outlook → Send email with options
Configure:

To: Manager email (e.g., manager@contoso.com)

Subject: Approval needed: @{triggerBody()?['Title']}

User Options: Approve,Reject

Body: Include key details + link to the item:

<p>Hi Manager,</p>
<p>Please review this request:</p>
<p>
  <strong>Title</strong>: @{triggerBody()?['Title']}<br/>
  <strong>Product</strong>: @{triggerBody()?['Product']}<br/>
  <strong>Estimated Cost</strong>: @{outputs('Formatted_Cost')}
</p>
<p>Choose: <strong>Approve</strong> or <strong>Reject</strong> (buttons above).</p>


Action: Condition (Check selected option)
Expression (body of “Send email with options” output may differ; commonly use its dynamic value, e.g. SelectedOption):

equals(outputs('Send_email_with_options')?['SelectedOption'], 'Approve')


If Approve (Yes):
SharePoint → Update item

Id: ID (from trigger)

Status: Approved

(Optional) Email the requester: “Your request was approved”

If Reject (No):
SharePoint → Update item

Status: Rejected

(Optional) Email the requester: “Your request was rejected”

If NO (Not Expensive) → Auto-approve Path

SharePoint → Update item

Status: Approved

(Optional) Email requester with “Approved (No approval required)”.

🧠 Note: You mentioned “copy and paste and update messages in false condition part.” For the Reject/Not Expensive branch, duplicate the email action and edit the wording accordingly.

📐 Flow Summary (Demo 02)
Trigger: When an item is created (SharePoint: Requests)
  ├── Compose: Formatted_Cost = formatNumber(EstimatedCost,'$####,###,00')
  ├── Send email (receipt) → RequesterEmail
  └── Condition: Is Expensive?
       ├── YES:
       │    ├── Send email with options (To: Manager, Options: Approve,Reject)
       │    └── Condition: SelectedOption == 'Approve' ?
       │         ├── YES → Update item: Status = Approved; (Email requester: approved)
       │         └── NO  → Update item: Status = Rejected; (Email requester: rejected)
       └── NO:
            ├── Update item: Status = Approved
            └── (Email requester: auto-approved)

📨 Email Template (Plain Text Variant)
Hello 
Here is your form subbmision Details

date - {{utcNow}}
topic - {{Topic}}
details - {{Details}}
thank you so much!


In Power Automate, use dynamic tokens instead of the double curly braces.

🔧 Tips & Gotchas

HTML vs Plain Text: Turn on Is HTML if you’re using HTML in email bodies.

Internal Names: SharePoint “display names” differ from “internal names” if you renamed columns later. Use the dynamic content from trigger/actions to avoid typos.

Send email with options: Returns a selected value string. Name your action clearly (e.g., Send_email_with_options) so the dynamic output token is obvious.

Approvals Connector (Alternative): For richer approvals, you can use Start and wait for an approval instead of “Send email with options.”
