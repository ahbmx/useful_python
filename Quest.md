Great! To send the questionnaire responses to a MySQL database, you’ll follow these steps:


---

✅ Requirements

1. Install mysql-connector-python:



pip install mysql-connector-python

2. Set up your MySQL database (one-time setup):

Create a database (e.g., questionnaire_db)

Create a table for responses (e.g., responses)





---

📦 Step 1: MySQL Table Structure Example

CREATE DATABASE questionnaire_db;

USE questionnaire_db;

CREATE TABLE responses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    q1 TEXT,
    q2 TEXT,
    q3 TEXT,
    q4 TEXT,
    q5 TEXT,
    q6 TEXT,
    q7 TEXT,
    q8 TEXT,
    q9 TEXT,
    q10 TEXT,
    submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


---

🐍 Step 2: Python Script with MySQL Integration

import mysql.connector
import pandas as pd

# --- Update with your actual MySQL credentials ---
DB_CONFIG = {
    'host': 'localhost',
    'user': 'your_username',
    'password': 'your_password',
    'database': 'questionnaire_db'
}

def ask_question(question):
    print("\n" + question['text'])

    if question['type'] == 'multiple_choice':
        for idx, option in enumerate(question['options'], 1):
            print(f"{idx}. {option}")
        while True:
            try:
                choice = int(input("Enter the number of your choice: "))
                if 1 <= choice <= len(question['options']):
                    return question['options'][choice - 1]
                else:
                    print("Please enter a valid option number.")
            except ValueError:
                print("Invalid input. Please enter a number.")
    elif question['type'] == 'open':
        return input("Your answer: ")

def insert_to_db(responses):
    try:
        conn = mysql.connector.connect(**DB_CONFIG)
        cursor = conn.cursor()

        sql = """
        INSERT INTO responses (q1, q2, q3, q4, q5, q6, q7, q8, q9, q10)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """
        cursor.execute(sql, tuple(responses))
        conn.commit()

        print("\n✅ Responses successfully saved to MySQL database.")
    except mysql.connector.Error as err:
        print("❌ Error:", err)
    finally:
        if conn.is_connected():
            cursor.close()
            conn.close()

def main():
    questions = [
        {"text": "What is your favorite color?", "type": "open"},
        {"text": "Choose your preferred programming language:", "type": "multiple_choice",
         "options": ["Python", "Java", "C++", "JavaScript"]},
        {"text": "How many hours do you code per day?", "type": "open"},
        {"text": "What is your operating system?", "type": "multiple_choice",
         "options": ["Windows", "macOS", "Linux", "Other"]},
        {"text": "What motivates you to code?", "type": "open"},
        {"text": "Choose your preferred development environment:", "type": "multiple_choice",
         "options": ["VS Code", "PyCharm", "Vim", "Other"]},
        {"text": "Where did you learn programming?", "type": "open"},
        {"text": "How do you rate your coding skills?", "type": "multiple_choice",
         "options": ["Beginner", "Intermediate", "Advanced", "Expert"]},
        {"text": "What is your dream tech company?", "type": "open"},
        {"text": "What do you want to improve most in coding?", "type": "open"}
    ]

    responses = []

    print("=== Welcome to the Developer Questionnaire ===")

    for question in questions:
        answer = ask_question(question)
        responses.append(answer)

    # Optionally show answers using pandas
    df = pd.DataFrame([responses], columns=[f"Q{i+1}" for i in range(10)])
    print("\n=== Your Responses ===")
    print(df.T)

    # Save to MySQL
    insert_to_db(responses)

if __name__ == "__main__":
    main()


---

🧪 What to Customize:

Replace 'your_username' and 'your_password' with your DB credentials.

Modify column names if your MySQL schema is different.



---

🔒 Optional Enhancements:

Add user ID or email before starting the questionnaire.

Auto-create the table if it doesn’t exist.

Log timestamps or IP address.


Would you like to:

Automatically create the table from the script if it’s missing?

Add user identification for each response?


Let me know, and I can include that too.

