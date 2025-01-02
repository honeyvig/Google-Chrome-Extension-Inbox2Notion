# Google-Chrome-Extension-Inbox2Notion

Transform your Gmail into organized action items

✔️Simplified Workflow
No more copying and pasting. Convert Gmail emails directly to Notion tasks, leads, and more without the fuss. Visualize your emails with clarity and quickly discern what needs your attention at a glance.

✔️Enhance Your Gmail Inbox
See Notion properties like status, due date, and assignee right in your Gmail.  
----------
To create a tool like Inbox2Notion which integrates Gmail with Notion and simplifies the workflow by converting Gmail emails into Notion tasks, we can use Google Apps Script for Gmail integration and the Notion API for managing tasks in Notion. Here's an overview of how you can achieve this, along with the basic code for setting up the Gmail to Notion integration.
Steps to Create the Extension

    Gmail API: To fetch emails from Gmail.
    Notion API: To create, manage, and update tasks in Notion.
    Google Apps Script: A script to handle Gmail-to-Notion integration, which automates the extraction of emails and converts them to tasks.

Steps Overview

    Fetch emails from Gmail.
    Create tasks in Notion using the Notion API.
    Display task properties like status, due date, and assignee in Gmail.
    Ensure that tasks in Notion are updated based on email actions.

1. Create the Google Apps Script Project

    Open Google Sheets or Google Drive.
    Go to Extensions → Apps Script.
    Delete the default code and replace it with the following script.

2. Google Apps Script - Connecting Gmail to Notion

// Notion API token and Database ID
const notionToken = 'YOUR_NOTION_INTEGRATION_TOKEN';
const databaseId = 'YOUR_NOTION_DATABASE_ID';  // The ID of your Notion database (Tasks)

function createNotionTaskFromEmail() {
  const threads = GmailApp.getInboxThreads(0, 5); // Get last 5 emails (you can adjust this number)
  const taskData = threads.map(thread => {
    const message = thread.getMessages()[0];  // Get the first message in the thread
    const subject = message.getSubject();
    const body = message.getPlainBody();
    const sender = message.getFrom();
    const date = message.getDate();

    return {
      subject,
      body,
      sender,
      date
    };
  });

  // Send the task data to Notion
  taskData.forEach(task => {
    createNotionTask(task);
  });
}

function createNotionTask(task) {
  const url = 'https://api.notion.com/v1/pages';
  
  const notionData = {
    parent: { database_id: databaseId },
    properties: {
      "Name": {
        title: [
          {
            text: {
              content: task.subject
            }
          }
        ]
      },
      "Sender": {
        rich_text: [
          {
            text: {
              content: task.sender
            }
          }
        ]
      },
      "Email Body": {
        rich_text: [
          {
            text: {
              content: task.body
            }
          }
        ]
      },
      "Date Received": {
        date: {
          start: task.date.toISOString()
        }
      }
    }
  };
  
  const options = {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${notionToken}`,
      'Content-Type': 'application/json',
      'Notion-Version': '2021-05-13'
    },
    payload: JSON.stringify(notionData)
  };

  UrlFetchApp.fetch(url, options);
}

// Trigger the function every hour (or your preferred time interval)
function createTimeDrivenTriggers() {
  ScriptApp.newTrigger('createNotionTaskFromEmail')
    .timeBased()
    .everyHours(1)
    .create();
}

3. Set Up the Script

    Notion Integration:
        Go to your Notion workspace.
        Create a database for your tasks (e.g., "Inbox Tasks").
        Create properties like:
            Name: The task name (Title).
            Sender: The email sender (Text).
            Email Body: The content of the email (Text).
            Date Received: The email's received date (Date).
        Create a Notion integration via Notion's developer portal to get your Integration Token and Database ID.

    Google Apps Script Authorization:
        Save the script.
        Run the createNotionTaskFromEmail function to trigger Gmail integration.
        Grant permission for the script to access Gmail and Notion.

4. Display Notion Properties in Gmail (For Gmail UI)

While it’s not feasible to directly modify Gmail’s UI (like adding a status column in Gmail), you can display task details in the email body. For example:

    Email content: In your email threads, you can append task-related details (status, assignee, due date) directly within the email body.

function appendTaskDetailsToEmail(task) {
  const threads = GmailApp.getInboxThreads(0, 5);  // Retrieve the last 5 threads
  const taskDetails = `
    Task in Notion:
    Subject: ${task.subject}
    Sender: ${task.sender}
    Status: Pending
    Due Date: ${task.dueDate}
  `;
  
  threads.forEach(thread => {
    const message = thread.getMessages()[0];
    message.reply(taskDetails);
  });
}

This function appends the task details (like status, assignee, and due date) to the email replies, so users can easily see task information in the context of their emails.
5. Notion API for Task Management

You’ll also want to manage your tasks in Notion (updating status, due date, etc.). For that, use the Notion API to modify tasks based on certain triggers (like updating the task when a specific email is received).

Here’s a quick example of how you can update the status of a task in Notion using its API.

function updateNotionTaskStatus(pageId, newStatus) {
  const url = `https://api.notion.com/v1/pages/${pageId}`;
  
  const data = {
    properties: {
      "Status": {
        select: {
          name: newStatus
        }
      }
    }
  };
  
  const options = {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${notionToken}`,
      'Content-Type': 'application/json',
      'Notion-Version': '2021-05-13'
    },
    payload: JSON.stringify(data)
  };

  UrlFetchApp.fetch(url, options);
}

6. Triggers

You can set up a time-based trigger to automatically run the createNotionTaskFromEmail function at regular intervals (e.g., every hour) or trigger it manually as needed.

function createTimeDrivenTriggers() {
  ScriptApp.newTrigger('createNotionTaskFromEmail')
    .timeBased()
    .everyHours(1)  // Runs every hour, change as per your needs
    .create();
}

Conclusion

With this setup, you can:

    Automatically fetch Gmail emails and convert them into tasks in Notion.
    Include task properties (status, assignee, due date) directly in Gmail replies.
    Manage tasks in Notion via the Notion API.

This is a basic example, and you can extend it further to include features like manual task creation, additional filters, email parsing for due dates and assignees, and more!
