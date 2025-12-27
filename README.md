# 🚀 Conversation State Manager for N8N

## 📋 Problem Solved

This workflow addresses the fundamental challenge of managing non-linear conversations in chatbots: maintaining context across multiple steps. It ensures the bot waits for specific user inputs, manages the conversation state to prevent incorrect reactivation, and always knows the user's current position in the dialogue.

## 🔧 How It Works

### 1. **Persistent State**
- Utilizes Google Sheets as a database to store the conversation state for each user.
- Each user has a unique record containing: `from` (ID), `step`, `data` (JSON payload), `timestamp`, and `status`.

### 2. **Intelligent Workflow Logic**

Webhook → Message Processor → State Fetcher -> Action Decision Node -> State Updater


### 3. **State Stages**
- `null` or `empty`: New user - sends the welcome list/message.
- `awaiting_specialty`: User is waiting to choose a specialty.
- `awaiting_date`: User has chosen a specialty, awaiting a date.
- `awaiting_time`: User has provided a date, awaiting a time slot.
- `finished`: Conversation is concluded.

## 📊 Google Sheets Structure

Create a spreadsheet with a tab named "States" and the following columns:

| A (from) | B (step) | C (data) | D (timestamp) | E (status) |
|----------|----------|----------|---------------|------------|
| 5511999999999@c.us | awaiting_specialty | {} | 2025-01-17T10:30:00Z | active |

## ⚙️ Configuration

### 1. **N8N Environment Variables**
Add the following variables in your N8N instance:

GOOGLE_SHEET_ID = "your_google_sheet_id_here"


### 2. **Evolution API Webhook**
Configure the webhook in Evolution API to point to:

https://your-n8n-instance.com/webhook/whatsapp-webhook


### 3. **Google Sheets Credentials**
- Configure the Google Sheets connection in N8N.
- Ensure the service account has read/write permissions for the target spreadsheet.

## 🎯 Key Improvements

### ✅ **Robust State Control**
- Each user maintains an independent state.
- No conflicts between simultaneous conversations.
- State persists even if the N8N instance is restarted.

### ✅ **Input Validation**
- Verifies if the message is a valid response (e.g., interactive list selection).
- Handles standard text messages gracefully.
- Validates selected options against expected inputs.

### ✅ **Non-Linear Flow Support**
- Users can resume conversations exactly where they left off.
- Does not restart the flow from scratch if it is already in progress.
- Handles errors and invalid options effectively.

### ✅ **Scalability**
- Supports multiple users interacting simultaneously.
- Easy to extend with new steps and logic.
- Modular structure for easy expansion.

## 🔄 Expansion Guide

### Adding a New Step:
1. **Define the new state** (e.g., `awaiting_time`).
2. **Create a new check node** in the workflow to detect this state.
3. **Add processing logic** for that specific step (e.g., validate time format).
4. **Update the state** to the next step after processing.

### Example - Adding a Time Selection Step:
javascript
// Inside the state check node
if (userState.step === 'awaiting_date') {
  // Process provided date
  // Send list of available time slots
  // Update state to 'awaiting_time'
}


## 🚨 Important Notes

### **Why this architecture works:**
1. **Single Webhook**: All incoming messages are handled by one entry point, simplifying the API configuration.
2. **Centralized State**: Google Sheets acts as the source of truth, allowing the bot to maintain context without complex in-memory storage.
3. **Conditional Branching**: The workflow uses simple conditional logic to route users based on their stored state, ensuring a tailored response every time.

### **Prerequisites:**
- N8N instance (Cloud or Self-hosted).
- Google Cloud Project with Sheets API enabled.
- Evolution API instance running and accessible.

---
*This project is designed to be a starting point for complex conversational flows requiring state persistence.*