# Week15-Python-FinalProject

# Start Code

import sqlite3


def create_db():
    """Creates the database and the students table."""
    conn = sqlite3.connect('students.db')
    cursor = conn.cursor()

    # Create the table
    try:
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS students (
            student_id INTEGER PRIMARY KEY,
            full_name TEXT NOT NULL,
            age INTEGER,
            gender TEXT,
            email TEXT,
            address TEXT,
            phone TEXT,
            date_of_birth TEXT  -- Added a 7th column for date_of_birth
        )
        ''')
        conn.commit()
    except sqlite3.Error as err:
        print("Error creating the table:", err)
    finally:
        conn.close()


MIN_CHOICE = 1
MAX_CHOICE = 5
CREATE = 1
READ = 2
UPDATE = 3
DELETE = 4
EXIT = 5


def main():
    """Main function that handles menu options."""
    choice = 0
    while choice != EXIT:
        display_menu()
        choice = get_menu_choice()

        if choice == CREATE:
            create()
        elif choice == READ:
            read()
        elif choice == UPDATE:
            update()
        elif choice == DELETE:
            delete()


def display_menu():
    """Displays the menu options to the user."""
    print('\n----- Student Information Menu -----')
    print('1. Enter new student information')
    print('2. Read student information')
    print('3. Update student information')
    print('4. Delete student information')
    print('5. Exit the program')


def get_menu_choice():
    """Gets the menu choice from the user."""
    while True:
        try:
            choice = int(input('Enter your choice: '))
            if MIN_CHOICE <= choice <= MAX_CHOICE:
                return choice
            else:
                print(f'Valid choices are {MIN_CHOICE} through {MAX_CHOICE}.')
        except ValueError:
            print("Invalid input! Please enter a number.")


def create():
    """Prompts the user to input new student information."""
    print('Create New Student Information')
    full_name = input('Student Full Name: ')
    age = input('Student Age: ')
    gender = input('Student Gender: ')
    email = input('Student Email: ')
    address = input('Student Home Address: ')
    phone = input('Student Phone Number: ')
    date_of_birth = input('Student Date of Birth (YYYY-MM-DD): ')  # New field for date of birth

    # Input validation to ensure the age is an integer
    try:
        age = int(age)
    except ValueError:
        print("Invalid age entered! Age must be a number.")
        return

    # Insert the new student information
    insert_row(full_name, age, gender, email, address, phone, date_of_birth)


def read():
    """Allows the user to search for a student by name."""
    student_name = input('Enter a student name to search for: ')
    num_found = display_item(student_name)
    print(f'{num_found} row(s) found.')


def update():
    """Allows the user to update student information."""
    read()
    student_id = input('Select a Student ID to update: ')
    try:
        student_id = int(student_id)
    except ValueError:
        print("Invalid Student ID! It must be a number.")
        return

    full_name = input('Enter the new student full name: ')
    age = input('Enter the new student age: ')
    gender = input('Enter the new student gender: ')
    email = input('Enter the new student email: ')
    address = input('Enter the new student address: ')
    phone = input('Enter the new student phone number: ')
    date_of_birth = input('Enter the new student date of birth (YYYY-MM-DD): ')

    # Input validation to ensure the age is an integer
    try:
        age = int(age)
    except ValueError:
        print("Invalid age entered! Age must be a number.")
        return

    # Update the student's information
    num_updated = update_row(student_id, full_name, age, gender, email, address, phone, date_of_birth)
    if num_updated:
        print(f'{num_updated} student(s) updated.')
    else:
        print("No updates were made.")


def delete():
    """Allows the user to delete a student record by ID."""
    read()
    student_id = input('Select student ID to delete: ')
    try:
        student_id = int(student_id)
    except ValueError:
        print("Invalid Student ID! It must be a number.")
        return

    sure = input("Are you sure you want to delete this student information? (y/n): ")
    if sure.lower() == 'y':
        num_deleted = delete_students(student_id)
        print(f'{num_deleted} Student Information deleted.')
    else:
        print("Deletion canceled.")


def insert_row(full_name, age, gender, email, address, phone, date_of_birth):
    """Inserts a new student row into the database."""
    conn = None
    try:
        conn = sqlite3.connect('students.db')
        cur = conn.cursor()

        
        print(f"Inserting: {full_name}, {age}, {gender}, {email}, {address}, {phone}, {date_of_birth}")

        cur.execute('''INSERT INTO students (full_name, age, gender, email, address, phone, date_of_birth)
                       VALUES (?, ?, ?, ?, ?, ?, ?)''',
                    (full_name, age, gender, email, address, phone, date_of_birth))

        conn.commit()
        print("New student information added.")
    except sqlite3.Error as err:
        print("Error while inserting data:", err)  # Log specific database error
    finally:
        if conn:
            conn.close()


def display_item(name):
    """Displays information for students matching the given name."""
    conn = None
    results = []
    try:
        conn = sqlite3.connect('students.db')
        cur = conn.cursor()
        cur.execute('''SELECT * FROM students WHERE lower(full_name) = ?''', (name.lower(),))
        results = cur.fetchall()
        for row in results:
            print(f'student_id: {row[0]:<7} full_name: {row[1]:<21} '
                  f'age: {row[2]:<10} gender: {row[3]:<7} email: {row[4]:<30} '
                  f'address: {row[5]:<14} phone: {row[6]:<14} date_of_birth: {row[7]}')
    except sqlite3.Error as err:
        print('Database Error: ', err)
    finally:
        if conn:
            conn.close()
    return len(results)


def update_row(student_id, full_name, age, gender, email, address, phone, date_of_birth):
    """Updates the student information based on student_id."""
    conn = None
    try:
        conn = sqlite3.connect('students.db')
        cur = conn.cursor()
        cur.execute('''UPDATE students
                       SET full_name = ?, age = ?, gender = ?, email = ?, address = ?, phone = ?, date_of_birth = ?
                       WHERE student_id = ?''',
                    (full_name, age, gender, email, address, phone, date_of_birth, student_id))
        conn.commit()
        return cur.rowcount
    except sqlite3.Error as err:
        print('Database Error: ', err)
        return 0
    finally:
        if conn:
            conn.close()


def delete_students(student_id):
    """Deletes a student record based on student_id."""
    conn = None
    try:
        conn = sqlite3.connect('students.db')
        cur = conn.cursor()
        cur.execute('''DELETE FROM students WHERE student_id = ?''', (student_id,))
        conn.commit()
        return cur.rowcount
    except sqlite3.Error as err:
        print('Database Error: ', err)
        return 0
    finally:
        if conn:
            conn.close()


if __name__ == '__main__':
    create_db()  # Make sure the database and table are created before running the program
    main()
