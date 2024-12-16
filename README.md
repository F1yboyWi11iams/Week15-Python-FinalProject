# Week15-Python-FinalProject
This is for the Week 15 Python Final Project, due Thursday, December 19th, 2024, for the Python class taught by Professor Luke Snell

# Start Code

import sqlite3

# Function to create the database

def create_db():
    conn = sqlite3.connect('students.db')
    cursor = conn.cursor()

    # Create table
    
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS students (
        student_id INTEGER PRIMARY KEY,
        first_name TEXT NOT NULL,
        last_name TEXT NOT NULL,
        age INTEGER,
        gender TEXT,
        email TEXT,
        address TEXT,
        phone TEXT
    )
    ''')

    conn.commit()
    conn.close()
