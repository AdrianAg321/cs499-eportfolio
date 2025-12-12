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
//
    Node(const Course& c) {
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

    // Recursively insert a course 
    Node* insert(Node* node, const Course& course) {
        if (node == nullptr) return new Node(course);

        if (course.courseNumber < node->course.courseNumber) {
            node->left = insert(node->left, course);
        } 
        else if (course.courseNumber > node->course.courseNumber) {
            node->right = insert(node->right, course);
        }
        else {
            // My Note:
            // If the same course number appears again, I update the existing node.
            // This keeps the tree consistent when loading or reloading data.
            node->course = course;
        }
        return node;
    }

    // Recursively search for a course
    Node* search(Node* node, const string& courseNumber) {
        if (node == nullptr || node->course.courseNumber == courseNumber) return node;

        if (courseNumber < node->course.courseNumber) {
            return search(node->left, courseNumber);
        }
        return search(node->right, courseNumber);
    }

    // Find minimum node (used during deletion)
    Node* findMin(Node* node) {
        while (node != nullptr && node->left != nullptr) {
            node = node->left;
        }
        return node;
    }

    // Recursively remove a course by number
    Node* remove(Node* node, const string& courseNumber) {
        if (node == nullptr) return nullptr;

        if (courseNumber < node->course.courseNumber) {
            node->left = remove(node->left, courseNumber);
        } 
        else if (courseNumber > node->course.courseNumber) {
            node->right = remove(node->right, courseNumber);
        } 
        else {
            // My Note:
            // Added full delete logic for the BST.
            // I had to handle:
            // - removing a leaf node,
            // - removing a node with one child,
            // - and removing a node with two children by using the successor.
            // Working through these cases helped me better understand how the tree restructures itself.

            // Case 1: no children
            if (node->left == nullptr && node->right == nullptr) {
                delete node;
                return nullptr;
            }
            // Case 2: one child
            else if (node->left == nullptr) {
                Node* temp = node->right;
                delete node;
                return temp;
            } 
            else if (node->right == nullptr) {
                Node* temp = node->left;
                delete node;
                return temp;
            }
            // Case 3: two children
            else {
                Node* successor = findMin(node->right);
                node->course = successor->course;
                node->right = remove(node->right, successor->course.courseNumber);
            }
        }
        return node;
    }

public:
    CourseBST() {
        root = nullptr;
    }

    bool isEmpty() const {
        return root == nullptr;
    }

    // Insert method
    void insert(const Course& course) {
        root = insert(root, course);
    }

    // Remove method
    void removeCourse(const string& courseNumber) {
        // My Note:***
        // This removal option wasn't in my original version.
        // Adding it made the tree more complete and gave me practice
        // working with deletion logic and pointer handling.
        root = remove(root, courseNumber);
    }

    // Print all courses (sorted)
    void printAllCourses() {
        // My Note:***
        // Added a protection check so the program doesn't try to print
        // when the BST has no data loaded.
        if (isEmpty()) {
            cout << "No courses loaded. Load data first.\n";
            return;
        }
        inOrder(root);
    }

    // Print one course and its prerequisites******
    void printCourse(const string& courseNumber) {
        if (isEmpty()) {
            cout << "No courses loaded. Load data first.\n";
            return;
        }

        Node* found = search(root, courseNumber);
        if (found == nullptr) {
            cout << "Course not found.\n";
        } 
        else {
            cout << found->course.courseNumber << ", " << found->course.courseTitle << endl;

            if (!found->course.prerequisites.empty()) {
                cout << "Prerequisites: ";
                for (size_t i = 0; i < found->course.prerequisites.size(); ++i) {
                    cout << found->course.prerequisites[i];
                    if (i < found->course.prerequisites.size() - 1) cout << ", ";
                }
                cout << endl;
            } 
            else {
                cout << "Prerequisites: None" << endl;
            }
        }
    }
};

// Load courses from CSV
void loadCoursesFromFile(const string& filename, CourseBST& bst) {
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
            if (!prereq.empty()) {
                course.prerequisites.push_back(prereq);
            }
        }

        // My Note:
        // The insert method now supports overwriting duplicate course entries,
        // which helps when reloading or updating the dataset.
        bst.insert(course);
    }

    file.close();
    cout << "Courses loaded successfully!\n";
}

// Menu
void displayMenu() {
    cout << "\nWelcome to the course planner.\n" << endl;
    cout << "1. Load Data Structure." << endl;
    cout << "2. Print Course List." << endl;
    cout << "3. Print Course." << endl;
    cout << "4. Remove Course." << endl;  // new option ***
    cout << "9. Exit\n" << endl;
    cout << "Select a menu option by entering 1, 2, 3, 4, or 9: ";
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
            cout << "Enter 'courses.csv' to load course data: ";
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
            cout << "Enter the course number: ";
            cin >> courseNumber;

            transform(courseNumber.begin(), courseNumber.end(), courseNumber.begin(), ::toupper);
            bst.printCourse(courseNumber);
            break;
        }

        case 4: {
            // My Note:
            // New feature that lets the user remove a course directly from the tree.
            // This completed the main BST operations: insert, search, traverse, and delete.
            string courseNumber;
            cout << "Enter the course number to remove: ";
            cin >> courseNumber;

            transform(courseNumber.begin(), courseNumber.end(), courseNumber.begin(), ::toupper);
            bst.removeCourse(courseNumber);

            cout << "If the course existed, it has been removed.\n";
            break;
        }

        case 9:
            cout << "Thank you for using the course planner!" << endl;
            break;

        default:
            cout << "\n" << choice << " is not a valid option.\n" << endl;
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

