#include<iostream>
#include<fstream>
#include<sstream>
#include<vector>
using namespace std;
class LM {
public:
    string name, author, search;
    string issuedTo, returnDate;
    fstream file;
    int dateToDays(string dateStr) {
    int d, m, y;
    char dash;
    stringstream ss(dateStr);
    ss >> d >> dash >> m >> dash >> y;
    return y * 365 + m * 30 + d;
}
int countBooksByStudent(string studentName) {
    ifstream borrowFile("borrowedBooks.txt");
    string line;
    int count = 0;
    while (getline(borrowFile, line)) {
        if (line == studentName) count++;
    }
    borrowFile.close();
    return count;
}
void addBook() {
    cout << "Enter Book Name : ";
    getline(cin, name);
    cout << "Enter Book's Author name : ";
    getline(cin, author);
    issuedTo = "";
    returnDate = "";
    file.open("bookData.txt", ios::out | ios::app);
    file << name << "*" << author << "*" << issuedTo << "*" << returnDate << endl;
    file.close();
    cout << "Book Added Successfully!" << endl;
}
void showAll() {
    file.open("bookData.txt", ios::in);
    if (!file) {
        cout << "File not found!" << endl;
        return;
    }
    cout << "\n\nBook Name\t\t\tAuthor\t\t\tIssued To\t\tReturn Date\n";
    while (getline(file, name, '*') &&
           getline(file, author, '*') &&
           getline(file, issuedTo, '*') &&
           getline(file, returnDate)) {
        cout << name << "\t\t" << author << "\t\t";
        cout << (issuedTo.empty() ? "Available" : issuedTo) << "\t\t";
        cout << (returnDate.empty() ? "-" : returnDate) << endl;
    }
    file.close();
}
void extractBook() {
    cout << "Enter Book Name to search : ";
    getline(cin, search);
    file.open("bookData.txt", ios::in);
    if (!file) {
        cout << "File not found!" << endl;
        return;
    }
    bool found = false;
    while (getline(file, name, '*') &&
           getline(file, author, '*') &&
           getline(file, issuedTo, '*') &&
           getline(file, returnDate)) {
        if (search == name) {
            cout << "\nBook Found:\n";
            cout << "Name       : " << name << endl;
            cout << "Author     : " << author << endl;
            cout << "Issued To  : " << (issuedTo.empty() ? "Available" : issuedTo) << endl;
            cout << "Return Date: " << (returnDate.empty() ? "-" : returnDate) << endl;
            found = true;
            break;
        }
    }
    if (!found) {
        cout << "Book not found!" << endl;
    }
    file.close();
}
void checkReturnStatus() {
    cout << "Enter Book Name to check return status : ";
    getline(cin, search);
    file.open("bookData.txt", ios::in);
    if (!file) {
        cout << "File not found!" << endl;
        return;
    }
    bool found = false;
    while (getline(file, name, '*') &&
           getline(file, author, '*') &&
           getline(file, issuedTo, '*') &&
           getline(file, returnDate)) {
        if (search == name) {
            cout << "\nBook Found:\n";
            cout << "Name       : " << name << endl;
            cout << "Issued To  : " << (issuedTo.empty() ? "Available" : issuedTo) << endl;
            cout << "Return Date: " << (returnDate.empty() ? "-" : returnDate) << endl;
            if (!returnDate.empty()) {
                string today;
                cout << "Enter today's date (dd-mm-yyyy): ";
                getline(cin, today);
                int due = dateToDays(returnDate);
                int now = dateToDays(today);
                if (now > due) {
                    int late = now - due;
                    int fine = late * 10;
                    cout << "Returned Late by " << late << " days. Penalty: " << fine << " Taka.\n";
                } else {
                    cout << "Returned on time or not yet due.\n";
                }
            } else {
                cout << "Book not issued to anyone.\n";
            }
            found = true;
            break;
        }
    }
    if (!found) {
        cout << "Book not found!" << endl;
    }
    file.close();
}
void issueBookToStudent() {
    cout << "Enter Book Name to issue : ";
    getline(cin, search);
    ifstream infile("bookData.txt");
    ofstream tempFile("temp.txt");
    bool found = false;
    while (getline(infile, name, '*') &&
           getline(infile, author, '*') &&
           getline(infile, issuedTo, '*') &&
           getline(infile, returnDate)) {
        if (search == name && issuedTo.empty()) {
            cout << "Enter student name : ";
            string student;
            getline(cin, student);
            int count = countBooksByStudent(student);
            if (count >= 2) {
                cout << "Student has already borrowed 2 books. Cannot issue more.\n";
                tempFile << name << "*" << author << "*" << issuedTo << "*" << returnDate << endl;
                continue;
            }
            cout << "Enter Return Date (dd-mm-yyyy) : ";
            string newDate;
            getline(cin, newDate);
            ofstream log("borrowedBooks.txt", ios::app);
            log << student << endl;
            log.close();
            tempFile << name << "*" << author << "*" << student << "*" << newDate << endl;
            cout << "Book issued to " << student << "!\n";
            found = true;
        } else {
            tempFile << name << "*" << author << "*" << issuedTo << "*" << returnDate << endl;
        }
    }
    infile.close();
    tempFile.close();
    remove("bookData.txt");
    rename("temp.txt", "bookData.txt");
    if (!found)
        cout << "Book not found or already issued!\n";
}
void confirmReturn() {
    cout << "Enter Book Name to confirm return : ";
    getline(cin, search);
    ifstream infile("bookData.txt");
    ofstream tempFile("temp.txt");
    bool found = false;
    while (getline(infile, name, '*') &&
           getline(infile, author, '*') &&
           getline(infile, issuedTo, '*') &&
           getline(infile, returnDate)) {
        if (search == name && !issuedTo.empty()) {
            cout << "Book '" << name << "' was issued to " << issuedTo << ".\n";
            cout << "Confirm return? (Y/N): ";
            char confirm;
            cin >> confirm;
            cin.ignore();
            if (confirm == 'Y' || confirm == 'y') {
                tempFile << name << "*" << author << "*" << "" << "*" << "" << endl;
                found = true;
                cout << "Book returned successfully!\n";
            } else {
                tempFile << name << "*" << author << "*" << issuedTo << "*" << returnDate << endl;
                cout << "Return cancelled.\n";
            }
        } else {
            tempFile << name << "*" << author << "*" << issuedTo << "*" << returnDate << endl;
        }
    }
    infile.close();
    tempFile.close();
    remove("bookData.txt");
    rename("temp.txt", "bookData.txt");
    if (!found) {
        cout << "Book not found or not issued to anyone!\n";
    }
}
};
int main() {
    LM obj;
    char choice;
    do {
        cout << "\n\n" << endl;
        cout << "1-Show All Books" << endl;
        cout << "2-Search Book" << endl;
        cout << "3-Add Book (ADMIN)" << endl;
        cout << "4-Issue Book to Student" << endl;
        cout << "5-Confirm Book Return" << endl;
        cout << "6-Check Return Status & Fine" << endl;
        cout << "7-Exit" << endl;
        cout << "\n\n" << endl;
        cout << "Enter Your Choice : ";
        cin >> choice;
        cin.ignore();
        switch (choice) {
            case '1':
                obj.showAll();
                break;
            case '2':
                obj.extractBook();
                break;
            case '3':
                obj.addBook();
                break;
            case '4':
                obj.issueBookToStudent();
                break;
            case '5':
                obj.confirmReturn();
                break;
            case '6':
                obj.checkReturnStatus();
                break;
            case '7':
                return 0;
            default:
                cout << "Invalid Choice...!" << endl;
        }
    } while (true);
    return 0;
}
