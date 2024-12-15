# Mini-telegram-app
A small version of a mini app for verifying transactions on Warden protocol.
To build a Telegram bot that integrates with the **Warden platform** and notifies users about **pending approvals**, we need to:

1. **Create a Telegram Bot**: Set up a Telegram bot using BotFather and get the API token.
2. **Integrate with Warden**: Get information from the Warden system, such as pending approval requests.
3. **Notify Users**: Send notifications to Telegram users about pending approvals via the Telegram Bot API.
4. **Use Webhooks**: Set up a webhook to listen for events, such as pending approvals, and trigger notifications.

### Steps to Create the Telegram Bot

#### 1. **Create a Telegram Bot**

1. Open Telegram and search for the **BotFather**.
2. Type `/newbot` and follow the instructions to create your bot. You will receive a token at the end. Save this token; you'll use it to interact with the Telegram API.

#### 2. **Set Up the Backend**

Now, let's create the backend server to handle Warden events and send notifications to the Telegram bot. We'll use **Node.js** and **Telegraf**, a popular library for interacting with Telegram's Bot API.

##### Dependencies
You need to install the following dependencies:
- `express`: To set up an HTTP server.
- `axios`: To make HTTP requests to the Warden platform (for pending approvals).
- `telegraf`: To interact with the Telegram Bot API.

To install the dependencies:

```bash
npm init -y
npm install express axios telegraf body-parser
```

##### 3. **Backend Code (Node.js)**

Create a new file `server.js` to set up the server, interact with the Warden platform, and send messages via the Telegram bot.

```javascript
const express = require('express');
const axios = require('axios');
const { Telegraf } = require('telegraf');
const bodyParser = require('body-parser');

// Telegram Bot Token (from BotFather)
const TELEGRAM_TOKEN = 'YOUR_BOT_TOKEN';
const bot = new Telegraf(TELEGRAM_TOKEN);

// Warden platform API endpoint (mock for now)
const WARDEN_API_URL = 'http://localhost:3000/api/pending-approvals'; // Replace with actual Warden API endpoint

// Setting up the Express server
const app = express();
app.use(bodyParser.json());

// Handle Telegram webhook
app.post('/webhook', (req, res) => {
  const update = req.body;

  // Process incoming messages and set up commands
  bot.handleUpdate(update);
  res.send('OK');
});

// Function to fetch pending approvals from Warden
const fetchPendingApprovals = async () => {
  try {
    const response = await axios.get(WARDEN_API_URL);
    return response.data; // Assume this returns an array of pending approvals
  } catch (error) {
    console.error('Error fetching pending approvals:', error);
    return [];
  }
};

// Send Telegram notification about pending approvals
const sendTelegramNotification = async (userId, approvalData) => {
  const message = `
  📝 **Pending Approval**
  - **Requester**: ${approvalData.requester}
  - **Amount**: ${approvalData.amount}
  - **Reason**: ${approvalData.reason}
  - **Timestamp**: ${approvalData.timestamp}
  
  Please review the request.
  `;

  try {
    await bot.telegram.sendMessage(userId, message, { parse_mode: 'Markdown' });
  } catch (error) {
    console.error('Error sending message:', error);
  }
};

// Function to check for pending approvals and notify users
const checkForPendingApprovals = async () => {
  const pendingApprovals = await fetchPendingApprovals();

  // Replace with the logic of fetching the correct user(s) from Warden
  const userId = 'USER_TELEGRAM_ID'; // Replace with actual Telegram user ID

  for (let approval of pendingApprovals) {
    await sendTelegramNotification(userId, approval);
  }
};

// Set up a cron job or use a setInterval to check for pending approvals periodically
setInterval(checkForPendingApprovals, 60000); // Check every minute

// Start the bot webhook
bot.startWebhook('/webhook', null, 3000);
app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

### Explanation of the Code

1. **Telegraf Setup**: 
   - The `Telegraf` library is used to interact with Telegram's Bot API. You provide your bot token here.
   - The `bot.handleUpdate(update)` function is used to handle incoming updates like commands or messages from Telegram users.

2. **Fetching Pending Approvals from Warden**:
   - The `fetchPendingApprovals` function makes a GET request to the Warden platform's API to fetch the pending approval requests. Replace the mock URL with the actual API endpoint that returns pending approvals.
   - Pending approval data is assumed to contain fields like requester, amount, reason, and timestamp.

3. **Sending Notifications**:
   - The `sendTelegramNotification` function sends a formatted message to a user on Telegram using `bot.telegram.sendMessage`.
   - The message includes details like the requester, amount, and reason for the approval, in a Markdown format for better formatting.

4. **Checking for Pending Approvals**:
   - The `checkForPendingApprovals` function is called periodically (every minute) using `setInterval`. It fetches pending approval data and sends notifications to the relevant users.
   - For this example, the Telegram user ID (`userId`) is hardcoded. In a real implementation, this could be dynamically fetched from the Warden platform or stored in a database.

5. **Webhook for Telegram**:
   - The webhook listens for incoming updates from Telegram and processes them accordingly. This allows the bot to interact with users.

### Step 4: Set Up the Telegram Webhook

To enable Telegram notifications, you must set up a webhook to handle incoming messages. The webhook URL is provided when you deploy the app. For local development, tools like [ngrok](https://ngrok.com/) can expose your local server to the internet.

1. **Expose Your Local Server with ngrok**:
   - Install ngrok:
     ```bash
     npm install -g ngrok
     ```
   - Start ngrok:
     ```bash
     ngrok http 3000
     ```
   - This will provide you with a publicly accessible URL (e.g., `https://xxxx.ngrok.io`). You need to set this URL as the webhook for your bot.

2. **Set the Webhook for Your Bot**:
   Use the Telegram API to set the webhook for your bot:
   ```bash
   curl -F "url=https://xxxx.ngrok.io/webhook" https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook
   ```

### Step 5: Testing the Application

1. **Run Your Backend**:
   Start your backend server with:
   ```bash
   node server.js
   ```

2. **Test Notifications**:
   - Ensure that the Telegram bot is correctly sending notifications to your user(s).
   - You should receive a notification on Telegram for each pending approval that Warden detects.

### Step 6: Enhancements

- **User Management**: Implement logic to fetch and store user information dynamically. For example, track which users are authorized to receive approval notifications.
- **Approval Actions**: Add functionality for users to approve or reject requests directly from the Telegram bot by sending commands (e.g., `/approve <approvalId>`).
- **Multiple Users**: If your system supports multiple users, you can send notifications to different users based on their roles or permissions in the Warden system.

### Conclusion

This is a basic example of how to build a Telegram bot that integrates with the **Warden platform** for notifications about pending approvals. The bot fetches data from the Warden API, processes it, and sends notifications to users via Telegram. You can extend this solution by adding more features such as approval actions, user management, and enhanced security.
