# Node-js-project-email_reminder_system/
│
├── email_reminder.py      # main logic
├── reminders.db           # database for reminders
├── .env                   # stores email credentials
└── requirements.txt
import smtplib
import sqlite3
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import schedule
import time
from datetime import datetime
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
EMAIL_USER = os.getenv("EMAIL_USER")
EMAIL_PASS = os.getenv("EMAIL_PASS")

# Connect to database
conn = sqlite3.connect('reminders.db', check_same_thread=False)
cursor = conn.cursor()

# Create table
cursor.execute('''CREATE TABLE IF NOT EXISTS reminders (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    email TEXT NOT NULL,
                    subject TEXT NOT NULL,
                    message TEXT NOT NULL,
                    time TEXT NOT NULL
                )''')
conn.commit()


def send_email(to_email, subject, message):
    """Send an email reminder"""
    msg = MIMEMultipart()
    msg['From'] = EMAIL_USER
    msg['To'] = to_email
    msg['Subject'] = subject
    msg.attach(MIMEText(message, 'plain'))

    try:
        with smtplib.SMTP('smtp.gmail.com', 587) as server:
            server.starttls()
            server.login(EMAIL_USER, EMAIL_PASS)
            server.send_message(msg)
        print(f"✅ Email sent to {to_email}")
    except Exception as e:
        print(f"❌ Failed to send email: {e}")


def check_reminders():
    """Check if any reminder time matches the current time"""
    now = datetime.now().strftime("%Y-%m-%d %H:%M")
    cursor.execute("SELECT * FROM reminders WHERE time = ?", (now,))
    reminders = cursor.fetchall()
    for r in reminders:
        _, email, subject, message, _ = r
        send_email(email, subject, message)
        cursor.execute("DELETE FROM reminders WHERE id = ?", (r[0],))
        conn.commit()


def add_reminder(email, subject, message, time_str):
    """Add a new reminder"""
    cursor.execute("INSERT INTO reminders (email, subject, message, time) VALUES (?, ?, ?, ?)",
                   (email, subject, message, time_str))
    conn.commit()
    print("✅ Reminder added successfully!")


# Example usage
if __name__ == "__main__":
    # Add a reminder (you can replace this with user input or a web form)
    # Format for time: "YYYY-MM-DD HH:MM"
    # add_reminder("recipient@example.com", "Meeting Reminder", "You have a meeting at 10 AM", "2025-10-31 14:30")

    print("📧 Email Reminder System running...")
    schedule.every(1).minutes.do(check_reminders)

    while True:
        schedule.run_pending()
        time.sleep(30)

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Email Reminder System</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #4a90e2, #5adbb5);
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }

    h1 {
      margin-bottom: 20px;
    }

    form {
      background: rgba(255, 255, 255, 0.1);
      padding: 20px 30px;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
      width: 350px;
    }

    label {
      display: block;
      margin-top: 10px;
      font-weight: bold;
      color: #fff;
    }

    input, textarea {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      border: none;
      border-radius: 5px;
      outline: none;
    }

    input[type="datetime-local"] {
      cursor: pointer;
    }

    button {
      width: 100%;
      margin-top: 20px;
      padding: 10px;
      background-color: #1e90ff;
      border: none;
      color: white;
      font-size: 16px;
      border-radius: 5px;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    button:hover {
      background-color: #0077cc;
    }

    footer {
      margin-top: 20px;
      font-size: 13px;
      color: #f0f0f0;
    }
  </style>
</head>
<body>
  <h1>Email Reminder System</h1>
  <form action="send_reminder.php" method="POST">
    <label for="email">Recipient Email:</label>
    <input type="email" id="email" name="email" placeholder="Enter recipient email" required>

    <label for="subject">Subject:</label>
    <input type="text" id="subject" name="subject" placeholder="Enter email subject" required>

    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="4" placeholder="Enter your reminder message" required></textarea>

    <label for="time">Reminder Date & Time:</label>
    <input type="datetime-local" id="time" name="time" required>

    <button type="submit">Set Reminder</button>
  </form>

  <footer>© 2025 Email Reminder System Project</footer>
</body>
</html>
