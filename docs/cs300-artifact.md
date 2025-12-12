# CS 300 Artifact – Course Planner Binary Search Tree (Original and Enhanced)

This artifact comes from the CS 300 Algorithms and Data Structures course. The assignment required building a course planning tool using a Binary Search Tree (BST) to store and organize course information. The program loads course data from a file, displays a sorted list of courses, and prints details for any requested course. I selected this artifact because it represents one of my earliest experiences designing a full data structure from scratch. Enhancing it later in the program helped me strengthen my understanding of recursion, searching, tree balancing concepts, and error handling. The improvements shown below demonstrate my growth in writing cleaner, more maintainable, and more complete C++ code.

---

# Original Code

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <algorithm>

using namespace std;

// Struct to hold course details
struct Course {
    string courseNumber;
    string courseTitle;
    vector<string> prerequisites;
};

// Struct for each node in the BST
struct Node {
    Course course;
    Node* left;
    Node* right;

    Node(Course c) {
        course = c;
        left = nullptr;
        right = nullptr;
    }
};

// Binary Search Tree to store courses
class CourseBST {
private:
    Node* root;

    // In-order traversal: left → node → right
    void inOrder(Node* node) {
        if (node == nullptr) return;
        inOrder(node->left);
        cout << node->course.courseNumber << ", " << node->course.courseTitle << endl;
        inOrder(node->right);
    }

    // Insert a course into the BST
    Node* insert(Node* node, Course course) {
        if (node == nullptr) return new Node(course);
        if (course.courseNumber < node->course.courseNumber) {
            node->left = insert(node->left, course);
        } else {
            node->right = insert(node->right, course);
        }
        return node;
    }

    // Search for a course
    Node* search(Node* node, string courseNumber) {
        if (node == nullptr || node->course.courseNumber == courseNumber) return node;
        if (courseNumber < node->course.courseNumber) return search(node->left, courseNumber);
        return search(node->right, courseNumber);
    }

public:
    CourseBST() { root = nullptr; }

    void insert(Course course) {
        root = insert(root, course);
    }

    void printAllCourses() {
        inOrder(root);
    }

    void printCourse(string courseNumber) {
        Node* found = search(root, courseNumber);
        if (found == nullptr) {
            cout << "Course not found.\n";
        } else {
            cout << found->course.courseNumber << ", " << found->course.courseTitle << endl;
            if (!found->course.prerequisites.empty()) {
                cout << "Prerequisites: ";
                for (size_t i = 0; i < found->course.prerequisites.size(); ++i) {
                    cout << found->course.prerequisites[i];
                    if (i < found->course.prerequisites.size() - 1) cout << ", ";
                }
                cout << endl;
            } else {
                cout << "Prerequisites: None" << endl;
            }
        }
    }
};

// Load courses from file
void loadCoursesFromFile(string filename, CourseBST& bst) {
    cout << "Trying to open file: " << filename << endl;
    ifstream file(filename);
    if (!file.is_open()) {
        cout << "Error: Cannot open file.\n";
        return;
    }

    string line;
    while (getline(file, line)) {
        stringstream ss(line);
        string courseNumber, courseTitle, prereq;
        Course course;

        getline(ss, courseNumber, ',');
        getline(ss, courseTitle, ',');

        course.courseNumber = courseNumber;
        course.courseTitle = courseTitle;

        while (getline(ss, prereq, ',')) {
            course.prerequisites.push_back(prereq);
        }

        bst.insert(course);
    }

    file.close();
    cout << "Courses loaded successfully!\n";
}

// Menu display
void displayMenu() {
    cout << "\nWelcome to the course planner.\n" << endl;
    cout << "1. Load Data Structure." << endl;
    cout << "2. Print Course List." << endl;
    cout << "3. Print Course." << endl;
    cout << "9. Exit\n" << endl;
    cout << "Select a menu option: ";
}

int main() {
    CourseBST bst;
    int choice = 0;

    while (choice != 9) {
        displayMenu();
        cin >> choice;

        switch (choice) {
        case 1: {
            string filename;
            cout << "Enter 'courses.csv': ";
            cin >> filename;
            loadCoursesFromFile(filename, bst);
            break;
        }
        case 2:
            cout << "\nHere is a sample schedule:\n" << endl;
            bst.printAllCourses();
            break;
        case 3: {
            string courseNumber;
            cout << "Enter course number: ";
            cin >> courseNumber;
            transform(courseNumber.begin(), courseNumber.end(), courseNumber.begin(), ::toupper);
            bst.printCourse(courseNumber);
            break;
        }
        case 9:
            cout << "Thank you for using the course planner!" << endl;
            break;
        default:
            cout << "\nInvalid option.\n" << endl;
        }
    }
    return 0;
}
```

---

# Enhanced Code

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <algorithm>

using namespace std;

struct Course {
    string courseNumber;
    string courseTitle;
    vector<string> prerequisites;
};

struct Node {
    Course course;
    Node* left;
    Node* right;

    Node(const Course& c) {
        course = c;
        left = nullptr;
        right = nullptr;
    }
};

class CourseBST {
private:
    Node* root;

    void inOrder(Node* node) {
        if (node == nullptr) return;
        inOrder(node->left);
        cout << node->course.courseNumber << ", "
             << node->course.courseTitle << endl;
        inOrder(node->right);
    }

    Node* insert(Node* node, const Course& course) {
        if (node == nullptr) return new Node(course);

        if (course.courseNumber < node->course.courseNumber) {
            node->left = insert(node->left, course);
        } else if (course.courseNumber > node->course.courseNumber) {
            node->right = insert(node->right, course);
        } else {
            node->course = course;
        }
        return node;
    }

    Node* search(Node* node, const string& courseNumber) {
        if (node == nullptr || node->course.courseNumber == courseNumber)
            return node;

        if (courseNumber < node->course.courseNumber)
            return search(node->left, courseNumber);

        return search(node->right, courseNumber);
    }

    Node* findMin(Node* node) {
        while (node && node->left != nullptr)
            node = node->left;
        return node;
    }

    Node* remove(Node* node, const string& courseNumber) {
        if (node == nullptr) return nullptr;

        if (courseNumber < node->course.courseNumber) {
            node->left = remove(node->left, courseNumber);
        } else if (courseNumber > node->course.courseNumber) {
            node->right = remove(node->right, courseNumber);
        } else {
            if (node->left == nullptr && node->right == nullptr) {
                delete node;
                return nullptr;
            } else if (node->left == nullptr) {
                Node* temp = node->right;
                delete node;
                return temp;
            } else if (node->right == nullptr) {
                Node* temp = node->left;
                delete node;
                return temp;
            } else {
                Node* successor = findMin(node->right);
                node->course = successor->course;
                node->right = remove(node->right, successor->course.courseNumber);
            }
        }
        return node;
    }

public:
    CourseBST() { root = nullptr; }

    bool isEmpty() const { return root == nullptr; }

    void insert(const Course& course) { root = insert(root, course); }

    void removeCourse(const string& courseNumber) {
        root = remove(root, courseNumber);
    }

    void printAllCourses() {
        if (isEmpty()) {
            cout << "No courses loaded.\n";
            return;
        }
        inOrder(root);
    }

    void printCourse(const string& courseNumber) {
        if (isEmpty()) {
            cout << "No courses loaded.\n";
            return;
        }

        Node* found = search(root, courseNumber);
        if (!found) {
            cout << "Course not found.\n";
        } else {
            cout << found->course.courseNumber << ", "
                 << found->course.courseTitle << endl;

            if (found->course.prerequisites.empty()) {
                cout << "Prerequisites: None\n";
            } else {
                cout << "Prerequisites: ";
                for (size_t i = 0; i < found->course.prerequisites.size(); ++i) {
                    cout << found->course.prerequisites[i];
                    if (i < found->course.prerequisites.size() - 1) cout << ", ";
                }
                cout << endl;
            }
        }
    }
};

void loadCoursesFromFile(const string& filename, CourseBST& bst) {
    ifstream file(filename);
    if (!file.is_open()) {
        cout << "Error opening file.\n";
        return;
    }

    string line;
    while (getline(file, line)) {
        stringstream ss(line);
        string courseNumber, courseTitle, prereq;
        Course course;

        getline(ss, courseNumber, ',');
        getline(ss, courseTitle, ',');

        course.courseNumber = courseNumber;
        course.courseTitle = courseTitle;

        while (getline(ss, prereq, ',')) {
            if (!prereq.empty())
                course.prerequisites.push_back(prereq);
        }

        bst.insert(course);
    }

    cout << "Courses loaded successfully!\n";
}

void displayMenu() {
    cout << "\nWelcome to the course planner.\n";
    cout << "1. Load Data Structure.\n";
    cout << "2. Print Course List.\n";
    cout << "3. Print Course.\n";
    cout << "4. Remove Course.\n";
    cout << "9. Exit\n";
    cout << "Select an option: ";
}

int main() {
    CourseBST bst;
    int choice = 0;

    while (choice != 9) {
        displayMenu();
        cin >> choice;

        if (choice == 1) {
            string filename;
            cout << "Enter 'courses.csv': ";
            cin >> filename;
            loadCoursesFromFile(filename, bst);
        } else if (choice == 2) {
            bst.printAllCourses();
        } else if (choice == 3) {
            string courseNumber;
            cout << "Enter course number: ";
            cin >> courseNumber;
            transform(courseNumber.begin(), courseNumber.end(), courseNumber.begin(), ::toupper);
            bst.printCourse(courseNumber);
        } else if (choice == 4) {
            string courseNumber;
            cout << "Enter course number to remove: ";
            cin >> courseNumber;
            transform(courseNumber.begin(), courseNumber.end(), courseNumber.begin(), ::toupper);
            bst.removeCourse(courseNumber);
            cout << "If the course existed, it has been removed.\n";
        } else if (choice == 9) {
            cout << "Thank you for using the course planner!\n";
        } else {
            cout << "Invalid option.\n";
        }
    }
    return 0;
}
```

---

# Summary of Enhancements

The enhanced version improves:

- **Reliability** through duplicate handling and consistent tree updates  
- **Maintainability** through reorganized class structure and cleaner methods  
- **Error handling** with checks for empty trees and invalid operations  
- **Functionality** by implementing full BST deletion logic  
- **Usability** with clearer feedback and the option to remove a course  

These enhancements demonstrate growth in recursion, memory management, data structure operations, and overall C++ development.

