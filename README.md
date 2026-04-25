 
📝 **Google Form (Data Input):** https://forms.gle/qQjMHcSqsq54EWAq9  
📊 **Google Sheet (Live Data):** https://docs.google.com/spreadsheets/d/1092SBWZLRQyQf3LYjfuxYxls5ruNP4a98MHpZfRFFYQ/edit?usp=sharing  

# Content QC Automation System

An automated **Content Quality Control (QC) System** built using **Google Sheets** and **Google Apps Script** to validate educational content, detect errors, generate QC dashboards, and send automated email reports.

---

## 🚀 Project Overview

This project helps automate the quality-checking process for large-scale educational content.

It checks content for:

- Missing questions
- Missing options
- Missing correct answers
- Invalid answer format
- Missing explanations
- Short explanations
- Invalid difficulty level
- Duplicate questions

The system automatically marks each content item as **Pass** or **Fail**, logs all errors, creates a dashboard, and sends a daily QC report via email.

---

## 🎯 Problem Statement

Manual content QC is time-consuming, repetitive, and error-prone.  
This project automates the QC workflow to improve accuracy, consistency, and operational efficiency.

---

## 🛠️ Tech Stack

- Google Sheets
- Google Apps Script
- Time-driven Triggers
- MailApp Service

---

## 📁 Google Sheet Structure

### 1. `Content_Raw`

Main content database.

| Content ID | Subject | Topic | Question | Option A | Option B | Option C | Option D | Correct Answer | Explanation | Difficulty | QC Status |
|---|---|---|---|---|---|---|---|---|---|---|---|

Example:

| C101 | Math | Algebra | Solve x + 2 = 5 | 1 | 2 | 3 | 4 | C | x + 2 = 5, so x = 3 | Easy | Pass |

---

### 2. `QC_Checks`

Stores all detected errors.

| Content ID | Error Type | Description |
|---|---|---|

---

### 3. `QC_Dashboard`

Shows automated QC summary.

Metrics included:

- Total Content
- Passed Content
- Failed Content
- Pass %
- Fail %
- Error Type Count

---

## ⚙️ Features

- Automated content validation
- Duplicate question detection
- Format validation
- QC error logging
- Pass/Fail status update
- Automated QC dashboard
- Daily email report
- Time-based automation trigger

---

## ✅ Validation Rules

The system checks:

1. Question should not be empty
2. Question length should be at least 10 characters
3. Options A, B, C, D should not be empty
4. Correct answer should not be empty
5. Correct answer must be A/B/C/D
6. Explanation should not be empty
7. Explanation should be at least 30 characters
8. Difficulty must be Easy, Medium, or Hard
9. Duplicate questions should be flagged

---

## 🧠 How It Works

1. Content is added to the `Content_Raw` sheet
2. Apps Script reads all rows
3. QC validation checks are applied
4. Errors are logged in `QC_Checks`
5. QC status is updated as Pass/Fail
6. Dashboard is generated in `QC_Dashboard`
7. Daily QC summary is sent via email

---

## 📌 Apps Script Code

```javascript
function runQC() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("Content_Raw");

  if (!sheet) throw new Error("Sheet not found");

  const data = sheet.getDataRange().getValues();

  let qcSheet = ss.getSheetByName("QC_Checks") || ss.insertSheet("QC_Checks");
  qcSheet.clear();
  qcSheet.appendRow(["Content ID", "Error Type", "Description"]);

  let seenQuestions = {};

  for (let i = 1; i < data.length; i++) {
    let row = data[i];
    let errors = [];

    let contentId = row[0];
    let question = row[3] ? row[3].toString().toLowerCase().trim() : "";
    let optionA = row[4];
    let optionB = row[5];
    let optionC = row[6];
    let optionD = row[7];
    let correctAnswer = row[8];
    let explanation = row[9] ? row[9].toString().trim() : "";
    let difficulty = row[10];

    if (!question) {
      errors.push("Missing Question");
      qcSheet.appendRow([contentId, "Missing Question", "Question is empty"]);
    }

    if (question && question.length < 10) {
      errors.push("Question Too Short");
      qcSheet.appendRow([contentId, "Question Too Short", "Question length is too short"]);
    }

    if (seenQuestions[question]) {
      errors.push("Duplicate Question");
      qcSheet.appendRow([contentId, "Duplicate", "Same question already exists"]);
    } else {
      seenQuestions[question] = true;
    }

    if (!optionA || !optionB || !optionC || !optionD) {
      errors.push("Missing Options");
      qcSheet.appendRow([contentId, "Missing Options", "One or more options are empty"]);
    }

    if (!correctAnswer) {
      errors.push("Missing Answer");
      qcSheet.appendRow([contentId, "Missing Answer", "Correct Answer is empty"]);
    }

    if (
      correctAnswer &&
      !["A", "B", "C", "D"].includes(correctAnswer.toString().trim().toUpperCase())
    ) {
      errors.push("Invalid Answer Format");
      qcSheet.appendRow([contentId, "Invalid Answer Format", "Correct Answer must be A/B/C/D"]);
    }

    if (!explanation) {
      errors.push("Missing Explanation");
      qcSheet.appendRow([contentId, "Missing Explanation", "Explanation is empty"]);
    }

    if (explanation && explanation.length < 30) {
      errors.push("Explanation Too Short");
      qcSheet.appendRow([contentId, "Explanation Too Short", "Explanation should be at least 30 characters"]);
    }

    if (!["Easy", "Medium", "Hard"].includes(difficulty)) {
      errors.push("Invalid Difficulty");
      qcSheet.appendRow([contentId, "Invalid Difficulty", "Must be Easy/Medium/Hard"]);
    }

    let statusCell = sheet.getRange(i + 1, 12);

    if (errors.length > 0) {
      statusCell.setValue("Fail");
    } else {
      statusCell.setValue("Pass");
    }
  }

  createDashboard();
  sendQCEmail();
}

function createDashboard() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const contentSheet = ss.getSheetByName("Content_Raw");
  const qcSheet = ss.getSheetByName("QC_Checks");

  let dashboard = ss.getSheetByName("QC_Dashboard") || ss.insertSheet("QC_Dashboard");
  dashboard.clear();

  const contentData = contentSheet.getDataRange().getValues();
  const qcData = qcSheet.getDataRange().getValues();

  let totalContent = contentData.length - 1;
  let failedContent = 0;
  let passedContent = 0;

  for (let i = 1; i < contentData.length; i++) {
    let status = contentData[i][11];
    if (status === "Pass") passedContent++;
    if (status === "Fail") failedContent++;
  }

  let errorCounts = {};

  for (let i = 1; i < qcData.length; i++) {
    let errorType = qcData[i][1];
    errorCounts[errorType] = (errorCounts[errorType] || 0) + 1;
  }

  dashboard.appendRow(["Metric", "Value"]);
  dashboard.appendRow(["Total Content", totalContent]);
  dashboard.appendRow(["Passed Content", passedContent]);
  dashboard.appendRow(["Failed Content", failedContent]);
  dashboard.appendRow(["Pass %", totalContent ? (passedContent / totalContent * 100).toFixed(2) + "%" : "0%"]);
  dashboard.appendRow(["Fail %", totalContent ? (failedContent / totalContent * 100).toFixed(2) + "%" : "0%"]);

  dashboard.appendRow(["", ""]);
  dashboard.appendRow(["Error Type", "Count"]);

  for (let error in errorCounts) {
    dashboard.appendRow([error, errorCounts[error]]);
  }
}

function sendQCEmail() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const dashboard = ss.getSheetByName("QC_Dashboard");
  const data = dashboard.getDataRange().getValues();

  let message = "Daily QC Report:\n\n";

  for (let i = 1; i < data.length; i++) {
    if (data[i][0]) {
      message += data[i][0] + ": " + data[i][1] + "\n";
    }
  }

  MailApp.sendEmail(
    "your-email@example.com",
    "Daily Content QC Report",
    message
  );
}
