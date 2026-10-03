Assignment: Practical Development Setup
Flutter  |  Python  |  MySQL  |  Visual Studio Code
Field
Details
Name
Ayom Leek Ayom
Operating System
Windows 11 Pro

 
Ayom Leek - Assignment: Practical Development Setup

Overview: This report documents the installation, configuration and verification of a modern software development environment. It covers the Flutter/Dart SDK, a MySQL database server, an isolated Python virtual environment, and a configured Visual Studio Code workspace. Each task includes the exact commands used, the results obtained, and written explanations.
Section 1: Task 1 - Flutter & Dart Setup
1.1 Verifying the Installation with flutter doctor
The following command checks that the Flutter SDK, Dart, Android toolchain, VS Code and connected devices are correctly installed and configured:
flutter doctor
 
Output summary (paste your own terminal output below, replacing the sample):
C:\Users\admin>flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[√] Flutter (Channel stable, 3.47.5, on Microsoft Windows [Version 10.0.26300.9457], locale en-US)
[√] Windows Version (Windows 11 or higher, 26H2, 2009)
[√] Android toolchain - develop for Android devices (Android SDK version 36.0.0)
[√] Chrome - develop for the web
[√] Visual Studio - develop Windows apps (Visual Studio Community 2026 18.10.3)
[√] Connected device (3 available)
[√] Network resources


• No issues found!
 
Interpretation: A green tick [✓] against each component means it is installed and working. Any [!] or [✗] entry would indicate a missing dependency (for example, unaccepted Android licences, fixed with flutter doctor --android-licenses).

Figure 1.1: flutter doctor output in the terminal

 
1.2 Creating the my_first_app Project and Listing Devices
A new Flutter application was created, the project folder was entered, and all available target devices were listed:
flutter create my_first_app
cd my_first_app
flutter devices
 
Explanation: flutter create generates the project scaffold (lib/main.dart, pubspec.yaml, platform folders). cd my_first_app moves into the project. flutter devices lists every emulator, physical device, browser and desktop target on which the app can be run.

Figure 1.2: flutter create my_first_app - project creation

 

Figure 1.3: flutter devices - available target devices

 
1.3 Conceptual Question: Hot Reload vs Hot Restart
Hot Reload injects updated source code into the running Dart Virtual Machine and rebuilds the widget tree, without restarting the app. The application state is preserved, so the screen, counters, form inputs and navigation stack stay exactly as they were. It normally completes in under a second (shortcut: r in the terminal, or the lightning icon in VS Code).
Hot Restart recompiles the code and restarts the app from main(), destroying all current state. Every variable returns to its initial value and the app reloads from its first screen. It is slower than hot reload but much faster than a full stop-and-rebuild (shortcut: R in the terminal, or the green restart icon in VS Code).
Aspect
Hot Reload
Hot Restart
What it does
Injects changed code into the running VM and rebuilds widgets
Recompiles and restarts the app from main()
App state
Preserved
Lost and reset to defaults
Speed
Fastest (sub-second)
Fast, but slower than hot reload
Runs main() again?
No
Yes
Runs initState() again?
No (for existing widgets)
Yes
Typical shortcut
r  /  Ctrl+S in VS Code
R  /  Ctrl+Shift+F5 in VS Code

 
When to use each:
• Use Hot Reload for UI and logic tweaks inside build() methods: changing colours, text, padding, layout, widget structure or styling. It lets you see results immediately while keeping your place in the app, for example when tuning a form that is several screens deep.
• Use Hot Restart when a change affects state initialisation and hot reload does not pick it up: editing main(), changing global variables or static fields, modifying initState() logic, or altering enum and generic type definitions. It is also useful when the app has reached a broken or unexpected state and you want a clean start.
• Use a full stop and re-run (neither of the above) when changing native Android/iOS code, adding assets or dependencies in pubspec.yaml, or changing app permissions, because these require a full rebuild.
Section 2: Task 2 - MySQL Database Management
2.1 Logging In and Executing SQL Commands
First, connect to the local MySQL server as root from the terminal (enter the root password when prompted):
mysql -u root -p
 
Then run the following SQL script inside the MySQL shell:
-- 1. Create the database
CREATE DATABASE school;
USE school;
 
-- 2. Create the students table
CREATE TABLE students (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(150) UNIQUE,
    enrolled_on DATE
);
 
-- 3. Insert two sample records
INSERT INTO students (name, email, enrolled_on) VALUES
    ('Amina Otieno', 'amina.otieno@example.com', '2025-09-01'),
    ('Brian Kamau',  'brian.kamau@example.com',  '2025-09-02');
 
-- 4. Retrieve all records
SELECT * FROM students;
 
Expected result of SELECT * FROM students;
mysql> SELECT * FROM students;
+----+--------------+--------------------------+-------------+
| id | name         | email                    | enrolled_on |
+----+--------------+--------------------------+-------------+
|  1 | Amina Otieno | amina.otieno@example.com | 2025-09-01  |
|  2 | Brian Kamau  | brian.kamau@example.com  | 2025-09-02  |
+----+--------------+--------------------------+-------------+
2 rows in set (0.00 sec)
 
Explanation: AUTO_INCREMENT makes MySQL generate a unique id automatically. PRIMARY KEY guarantees every row is uniquely identifiable. UNIQUE on email prevents two students from registering with the same address. The DATE type stores the enrolment date in YYYY-MM-DD format.

Figure 2.1: MySQL login and database/table creation

 

Figure 2.2: INSERT and SELECT * FROM students results

 
2.2 Security Reflection: Why Not Use the root Account for Applications?
Using root for an application's database connection violates the Principle of Least Privilege, which states that every account should hold only the minimum permissions needed to do its job. The main risks are:
• Excessive privileges. root has full control over the whole server: it can DROP any database, alter any table, read every user's data, and create or delete other accounts. A typical web or mobile backend only needs to read and write rows in one database.
• Large blast radius if compromised. If the application suffers an SQL injection attack, a leaked configuration file or a stolen credential, the attacker inherits root power and can destroy or exfiltrate all data on the server, not just the application's own data.
• Risk of host-level access. Administrative privileges such as FILE can allow reading and writing files on the database server's operating system, which can be abused to escalate an attack beyond the database.
• Accidental damage. A coding bug, a faulty migration or a mistyped query could wipe out tables or entire databases because nothing restricts the connection.
• Poor accountability and hard rotation. If every service and person shares root, audit logs cannot show who did what, and changing the root password breaks every application that depends on it.
• Compliance. Security standards and data-protection laws (for example the Kenya Data Protection Act, ISO 27001, PCI-DSS) expect restricted, role-based access to personal data.
2.3 Creating a Dedicated, Least-Privileged Application User
The following commands (run as root) create a user restricted to one database, with only the data-manipulation rights that the application needs:
-- Create the application user (localhost only, strong password)
CREATE USER 'school_app'@'localhost' IDENTIFIED BY 'Str0ng!Passw0rd#2025';
 
-- Grant only the required privileges, only on the school database
GRANT SELECT, INSERT, UPDATE, DELETE ON school.* TO 'school_app'@'localhost';
 
-- Apply changes (optional with GRANT, but harmless)
FLUSH PRIVILEGES;
 
-- Verify the permissions granted
SHOW GRANTS FOR 'school_app'@'localhost';
 
Explanation of the design choices:
• 'school_app'@'localhost' limits the account to connections from the same machine, so it cannot be used remotely.
• SELECT, INSERT, UPDATE, DELETE are the only privileges granted; there is no DROP, ALTER, CREATE, GRANT OPTION or FILE privilege.
• school.* scopes the permissions to the school database only, so other databases on the server remain inaccessible.
• Read-only variant: for a reporting service, grant only SELECT. To remove a privilege later, use REVOKE DELETE ON school.* FROM 'school_app'@'localhost';
Test the restrictions by logging in as the new user (mysql -u school_app -p). SELECT and INSERT on school.students will succeed, while DROP TABLE students; will fail with an 'access denied' error, proving least privilege is enforced.

Figure 2.3: Creating the least-privileged user and SHOW GRANTS output

 
Section 3: Task 3 - Python Virtual Environment
A virtual environment isolates a project's Python packages from the system-wide installation, preventing version conflicts between projects and making the setup reproducible. The exact commands used are shown below.
3.1 Commands for Windows (PowerShell / Command Prompt)
mkdir python_setup_lab
cd python_setup_lab
python -m venv venv
venv\Scripts\activate
pip install requests
pip list
pip freeze > requirements.txt
type requirements.txt
 
3.2 Expected Results


(venv) PS C:\Users\Default\Desktop\python_setup_lab> pip list
Package  Version
-------- ---------
certifi  2026.7.22
chardet  4.0.0
idna     2.10
pip      26.2.1
requests 2.25.1
urllib3  1.26.20
 
Explanation: python -m venv venv builds an isolated interpreter and package folder named venv. Activating it makes python and pip point to that folder. pip freeze lists every installed package with its exact version, and redirecting (>) that output into requirements.txt lets anyone recreate the same environment with pip install -r requirements.txt.

Figure 3.1: Virtual environment creation and activation

 

Figure 3.2: requests installation, pip list and requirements.txt

 
Section 4: Task 4 - VS Code Workspace Screenshot
4.1 Extensions Installed
The following extensions were installed from the VS Code Extensions panel (Ctrl+Shift+X):
Extension
Publisher / ID
Purpose
Flutter
Dart Code (Dart-Code.flutter)
Flutter project support, device selection, hot reload, widget tools
Dart
Dart Code (Dart-Code.dart-code)
Dart language support, analysis, debugging, formatting
Python
Microsoft (ms-python.python)
Python editing, running, debugging, interpreter management
Pylance
Microsoft (ms-python.vscode-pylance)
Fast IntelliSense, type checking and code navigation for Python
MySQL
cweijan (cweijan.vscode-mysql-client2)
Connect to MySQL, browse tables and run queries inside VS Code

 
4.2 Selecting the Virtual Environment Interpreter
Steps followed:
• Open the python_setup_lab folder in VS Code (File > Open Folder).
• Press Ctrl+Shift+P to open the Command Palette.
• Type and select: Python: Select Interpreter.
• Choose the entry that points to the project's environment: .\venv\Scripts\python.exe, usually labelled 'Python 3.x.x (venv)'.
• Open a new integrated terminal (Ctrl+`). VS Code activates the environment automatically and the prompt begins with (venv).
 
4.3 Workspace Screenshot
The single full-screen screenshot below shows all three required elements: (1) the Extensions panel listing the installed plugins, (2) the integrated terminal with the (venv) prompt active, and (3) the status bar showing the selected Python interpreter.

Figure 4.1: Full-screen VS Code window (single screenshot)

 
Conclusion
This assignment set up and verified four core parts of a modern development workflow. Flutter was validated with flutter doctor and a starter project was generated; MySQL was used to build a database and table, supported by a least-privilege security design; a Python virtual environment provided dependency isolation with a reproducible requirements.txt; and VS Code was configured with the required extensions and the correct project interpreter. Together, these form a secure, organised and reproducible foundation for future application development.
