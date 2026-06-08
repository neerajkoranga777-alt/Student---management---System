# Student---management---System


/*
 * ============================================================
 *   Student Management System
 *   Console-based application with file-persistent records
 * ============================================================
 */

#include <iostream>
#include <fstream>
#include <sstream>
#include <iomanip>
#include <string>
#include <vector>
#include <limits>
#include <algorithm>

// ─────────────────────────────────────────────
//  Constants
// ─────────────────────────────────────────────
const std::string DATA_FILE = "students.dat";
const char DELIM = '|';

// ─────────────────────────────────────────────
//  Student Structure
// ─────────────────────────────────────────────
struct Student {
    int    id;
    std::string name;
    int    age;
    std::string grade;   // e.g. "A", "B+", "C"
    double gpa;
    std::string email;
};

// ─────────────────────────────────────────────
//  Utility helpers
// ─────────────────────────────────────────────
void clearScreen() {
#if defined(_WIN32) || defined(_WIN64)
    system("cls");
#else
    system("clear");
#endif
}

void pauseScreen() {
    std::cout << "\n  Press ENTER to continue...";
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    std::cin.get();
}

void printHeader(const std::string& title) {
    std::cout << "\n";
    std::cout << "  ╔══════════════════════════════════════════════════╗\n";
    std::cout << "  ║" << std::setw(51) << std::left
              << ("  " + title) << "║\n";
    std::cout << "  ╚══════════════════════════════════════════════════╝\n\n";
}

void printDivider() {
    std::cout << "  ──────────────────────────────────────────────────\n";
}

void printStudentRow(const Student& s) {
    std::cout << "  │ "
              << std::left  << std::setw(5)  << s.id
              << std::setw(22) << s.name
              << std::setw(5)  << s.age
              << std::setw(6)  << s.grade
              << std::setw(6)  << std::fixed << std::setprecision(2) << s.gpa
              << "│\n";
}

// ─────────────────────────────────────────────
//  File I/O
// ─────────────────────────────────────────────
std::vector<Student> loadStudents() {
    std::vector<Student> students;
    std::ifstream fin(DATA_FILE);
    if (!fin.is_open()) return students;

    std::string line;
    while (std::getline(fin, line)) {
        if (line.empty()) continue;
        std::istringstream ss(line);
        std::string token;
        Student s;
        try {
            std::getline(ss, token, DELIM); s.id    = std::stoi(token);
            std::getline(ss, s.name,  DELIM);
            std::getline(ss, token, DELIM); s.age   = std::stoi(token);
            std::getline(ss, s.grade, DELIM);
            std::getline(ss, token, DELIM); s.gpa   = std::stod(token);
            std::getline(ss, s.email, DELIM);
            students.push_back(s);
        } catch (...) {
            // skip malformed lines
        }
    }
    fin.close();
    return students;
}

bool saveStudents(const std::vector<Student>& students) {
    std::ofstream fout(DATA_FILE, std::ios::trunc);
    if (!fout.is_open()) return false;

    for (const auto& s : students) {
        fout << s.id    << DELIM
             << s.name  << DELIM
             << s.age   << DELIM
             << s.grade << DELIM
             << std::fixed << std::setprecision(2) << s.gpa << DELIM
             << s.email << DELIM
             << "\n";
    }
    fout.close();
    return true;
}

int nextId(const std::vector<Student>& students) {
    int maxId = 0;
    for (const auto& s : students)
        if (s.id > maxId) maxId = s.id;
    return maxId + 1;
}

// ─────────────────────────────────────────────
//  Input helpers
// ─────────────────────────────────────────────
int inputInt(const std::string& prompt) {
    int val;
    while (true) {
        std::cout << prompt;
        if (std::cin >> val) { std::cin.ignore(); return val; }
        std::cout << "  [!] Invalid input. Try again.\n";
        std::cin.clear();
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }
}

double inputDouble(const std::string& prompt) {
    double val;
    while (true) {
        std::cout << prompt;
        if (std::cin >> val) { std::cin.ignore(); return val; }
        std::cout << "  [!] Invalid input. Try again.\n";
        std::cin.clear();
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }
}

std::string inputString(const std::string& prompt) {
    std::string val;
    std::cout << prompt;
    std::getline(std::cin, val);
    return val;
}

// ─────────────────────────────────────────────
//  Operations
// ─────────────────────────────────────────────

// 1. Add a student
void addStudent() {
    clearScreen();
    printHeader("ADD NEW STUDENT");

    auto students = loadStudents();
    Student s;
    s.id    = nextId(students);
    s.name  = inputString("  Full Name   : ");
    s.age   = inputInt   ("  Age         : ");
    s.grade = inputString("  Grade       : ");
    s.gpa   = inputDouble("  GPA (0-4.0) : ");
    s.email = inputString("  Email       : ");

    students.push_back(s);

    if (saveStudents(students))
        std::cout << "\n  ✓ Student added successfully! (ID: " << s.id << ")\n";
    else
        std::cout << "\n  ✗ Error saving record.\n";

    pauseScreen();
}

// 2. Display all students
void displayAllStudents() {
    clearScreen();
    printHeader("ALL STUDENTS");

    auto students = loadStudents();
    if (students.empty()) {
        std::cout << "  No records found.\n";
        pauseScreen();
        return;
    }

    std::cout << "  │ " << std::left
              << std::setw(5)  << "ID"
              << std::setw(22) << "Name"
              << std::setw(5)  << "Age"
              << std::setw(6)  << "Grade"
              << std::setw(6)  << "GPA"
              << "│\n";
    printDivider();

    for (const auto& s : students)
        printStudentRow(s);

    printDivider();
    std::cout << "  Total records: " << students.size() << "\n";
    pauseScreen();
}

// 3. Search / Display one student
void searchStudent() {
    clearScreen();
    printHeader("SEARCH STUDENT");

    int id = inputInt("  Enter Student ID to search: ");
    auto students = loadStudents();

    auto it = std::find_if(students.begin(), students.end(),
                           [id](const Student& s){ return s.id == id; });

    if (it == students.end()) {
        std::cout << "\n  ✗ Student with ID " << id << " not found.\n";
    } else {
        const Student& s = *it;
        std::cout << "\n";
        printDivider();
        std::cout << "  ID    : " << s.id    << "\n";
        std::cout << "  Name  : " << s.name  << "\n";
        std::cout << "  Age   : " << s.age   << "\n";
        std::cout << "  Grade : " << s.grade << "\n";
        std::cout << "  GPA   : " << std::fixed << std::setprecision(2) << s.gpa << "\n";
        std::cout << "  Email : " << s.email << "\n";
        printDivider();
    }
    pauseScreen();
}

// 4. Update a student
void updateStudent() {
    clearScreen();
    printHeader("UPDATE STUDENT");

    int id = inputInt("  Enter Student ID to update: ");
    auto students = loadStudents();

    auto it = std::find_if(students.begin(), students.end(),
                           [id](const Student& s){ return s.id == id; });

    if (it == students.end()) {
        std::cout << "\n  ✗ Student with ID " << id << " not found.\n";
        pauseScreen();
        return;
    }

    Student& s = *it;
    std::cout << "\n  Current record: " << s.name << " | Age: " << s.age
              << " | Grade: " << s.grade << " | GPA: " << s.gpa << "\n\n";
    std::cout << "  (Press ENTER to keep the current value)\n\n";

    auto promptKeep = [](const std::string& prompt, const std::string& current) {
        std::cout << prompt << " [" << current << "]: ";
        std::string val;
        std::getline(std::cin, val);
        return val.empty() ? current : val;
    };

    s.name  = promptKeep("  Name ", s.name);

    std::string ageStr = promptKeep("  Age  ", std::to_string(s.age));
    try { s.age = std::stoi(ageStr); } catch (...) {}

    s.grade = promptKeep("  Grade", s.grade);

    std::string gpaStr = promptKeep("  GPA  ",
        [&]{ std::ostringstream os; os << std::fixed << std::setprecision(2) << s.gpa; return os.str(); }());
    try { s.gpa = std::stod(gpaStr); } catch (...) {}

    s.email = promptKeep("  Email", s.email);

    if (saveStudents(students))
        std::cout << "\n  ✓ Record updated successfully!\n";
    else
        std::cout << "\n  ✗ Error saving record.\n";

    pauseScreen();
}

// 5. Delete a student
void deleteStudent() {
    clearScreen();
    printHeader("DELETE STUDENT");

    int id = inputInt("  Enter Student ID to delete: ");
    auto students = loadStudents();

    auto it = std::find_if(students.begin(), students.end(),
                           [id](const Student& s){ return s.id == id; });

    if (it == students.end()) {
        std::cout << "\n  ✗ Student with ID " << id << " not found.\n";
        pauseScreen();
        return;
    }

    std::cout << "\n  Found: " << it->name << " (ID: " << it->id << ")\n";
    std::string confirm = inputString("  Are you sure you want to delete? (yes/no): ");

    if (confirm == "yes" || confirm == "y") {
        students.erase(it);
        if (saveStudents(students))
            std::cout << "\n  ✓ Student deleted successfully!\n";
        else
            std::cout << "\n  ✗ Error saving record.\n";
    } else {
        std::cout << "\n  Operation cancelled.\n";
    }
    pauseScreen();
}

// 6. Display summary statistics
void displayStats() {
    clearScreen();
    printHeader("STATISTICS");

    auto students = loadStudents();
    if (students.empty()) {
        std::cout << "  No records to compute statistics.\n";
        pauseScreen();
        return;
    }

    double totalGpa = 0.0, maxGpa = students[0].gpa, minGpa = students[0].gpa;
    std::string topStudent;

    for (const auto& s : students) {
        totalGpa += s.gpa;
        if (s.gpa > maxGpa) { maxGpa = s.gpa; topStudent = s.name; }
        if (s.gpa < minGpa)   minGpa = s.gpa;
    }
    double avgGpa = totalGpa / students.size();

    printDivider();
    std::cout << "  Total Students  : " << students.size()  << "\n";
    std::cout << "  Average GPA     : " << std::fixed << std::setprecision(2) << avgGpa  << "\n";
    std::cout << "  Highest GPA     : " << maxGpa << " (" << topStudent << ")\n";
    std::cout << "  Lowest GPA      : " << minGpa << "\n";
    printDivider();

    pauseScreen();
}

// ─────────────────────────────────────────────
//  Main menu
// ─────────────────────────────────────────────
void showMenu() {
    clearScreen();
    std::cout << "\n";
    std::cout << "  ╔══════════════════════════════════════════════════╗\n";
    std::cout << "  ║       STUDENT MANAGEMENT SYSTEM  v1.0            ║\n";
    std::cout << "  ╠══════════════════════════════════════════════════╣\n";
    std::cout << "  ║                                                  ║\n";
    std::cout << "  ║   1.  Add Student                                ║\n";
    std::cout << "  ║   2.  Display All Students                       ║\n";
    std::cout << "  ║   3.  Search Student by ID                       ║\n";
    std::cout << "  ║   4.  Update Student Record                      ║\n";
    std::cout << "  ║   5.  Delete Student                             ║\n";
    std::cout << "  ║   6.  View Statistics                            ║\n";
    std::cout << "  ║   0.  Exit                                       ║\n";
    std::cout << "  ║                                                  ║\n";
    std::cout << "  ╚══════════════════════════════════════════════════╝\n";
    std::cout << "\n  Your choice: ";
}

int main() {
    int choice;
    do {
        showMenu();
        if (!(std::cin >> choice)) {
            std::cin.clear();
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            choice = -1;
        } else {
            std::cin.ignore();
        }

        switch (choice) {
            case 1: addStudent();        break;
            case 2: displayAllStudents(); break;
            case 3: searchStudent();     break;
            case 4: updateStudent();     break;
            case 5: deleteStudent();     break;
            case 6: displayStats();      break;
            case 0:
                clearScreen();
                std::cout << "\n  Goodbye! All records saved to '" << DATA_FILE << "'.\n\n";
                break;
            default:
                std::cout << "\n  [!] Invalid option. Please choose 0-6.\n";
                pauseScreen();
        }
    } while (choice != 0);

    return 0;
}