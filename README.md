#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h> // Include for date and time functions

// Define the default home page
#define HOME_PAGE "https://www.google.com"

// Define the structure for a doubly linked list node
typedef struct DoublyNode {
    char url[100];
    struct DoublyNode* prev;
    struct DoublyNode* next;
    char timestamp[100]; // Add a timestamp field
} DoublyNode;

// Function to create a new doubly linked list node
DoublyNode* createNode(char* url) {
    DoublyNode* newNode = (DoublyNode*)malloc(sizeof(DoublyNode));
    strcpy(newNode->url, url);
    newNode->prev = newNode->next = NULL;

    // Get the current date and time
    time_t now = time(NULL);
    struct tm* t = localtime(&now);
    strftime(newNode->timestamp, sizeof(newNode->timestamp), "%Y-%m-%d %H:%M:%S", t); // Format the date and time

    return newNode;
}

// Global variables to track the current tab, history list, and saved URLs list
DoublyNode* currentTab = NULL;
DoublyNode* historyList = NULL;
DoublyNode* historyTail = NULL;  // Tail pointer for history list
DoublyNode* savedList = NULL;
DoublyNode* savedTail = NULL;    // Tail pointer for saved URLs list

// Function to clear the console screen
void clearScreen() {
#ifdef _WIN32
    system("cls"); // For Windows
#else
    system("clear"); // For macOS and Linux
#endif
}

// Function to visit a new URL
void visit(char* url) {
    if (strcmp(url, "home") == 0) {
        url = HOME_PAGE;
    }

    // Validate URL format
    if (strncmp(url, "http://", 7) != 0 && strncmp(url, "https://", 8) != 0) {
        printf("Invalid URL format. Please use format: http(s)://example.com\n");
        return;
    }

    // Create a new node for the new tab
    DoublyNode* newNode = createNode(url);

    // Link the current node to the new node
    if (currentTab != NULL) {
        currentTab->next = newNode;
        newNode->prev = currentTab;
    }

    // Update the current tab to the new node
    currentTab = newNode;

    // Add the new node to the history list using the tail pointer if it is not the home page
    if (historyList == NULL) {
        if (strcmp(url, HOME_PAGE) != 0) {
            historyList = historyTail = createNode(url);
        }
    } else {
        if (strcmp(url, HOME_PAGE) != 0) {
            DoublyNode* newHistoryNode = createNode(url);
            historyTail->next = newHistoryNode;
            newHistoryNode->prev = historyTail;
            historyTail = newHistoryNode;
        }
    }

    printf("Visited: %s\n", currentTab->url);

    // Open the URL in the default web browser
    char command[200];
#ifdef _WIN32
    sprintf(command, "start %s", url); // For Windows
#elif __APPLE__
    sprintf(command, "open %s", url); // For macOS
#else
    sprintf(command, "xdg-open %s", url); // For Linux
#endif
    system(command);
}

// Function to go back to the previous URL
void goBack() {
    if (currentTab == NULL) {
        printf("No previous tab to go back to.\n");
        return;
    }

    if (currentTab->prev == NULL) {
        printf("Already at the home page.\n");
        return;
    } else {
        // Go back to the previous tab
        currentTab = currentTab->prev;
        printf("Went back to: %s\n", currentTab->url);
    }

    // Open the URL in the default web browser
    char command[200];
#ifdef _WIN32
    sprintf(command, "start %s", currentTab->url); // For Windows
#elif __APPLE__
    sprintf(command, "open %s", currentTab->url); // For macOS
#else
    sprintf(command, "xdg-open %s", currentTab->url); // For Linux
#endif
    system(command);
}

void visitHomePage() {
    if (currentTab != NULL) {
        currentTab = createNode(HOME_PAGE);
        printf("Visited: %s\n", currentTab->url);

        // Open the home page in the default web browser
        
        char command[200];
#ifdef _WIN32
        sprintf(command, "start %s", currentTab->url); // For Windows
#elif __APPLE__
        sprintf(command, "open %s", currentTab->url); // For macOS
#else
        sprintf(command, "xdg-open %s", currentTab->url); // For Linux
#endif
        system(command);
    }
}

void goForward() {
    if (currentTab == NULL || currentTab->next == NULL) {
        printf("No next tab to go forward to.\n");
        return;
    }
    currentTab = currentTab->next;
    printf("Went forward to: %s\n", currentTab->url);

    // Open the URL in the default web browser
    char command[200];
#ifdef _WIN32
    sprintf(command, "start %s", currentTab->url); // For Windows
#elif __APPLE__
    sprintf(command, "open %s", currentTab->url); // For macOS
#else
    sprintf(command, "xdg-open %s", currentTab->url); // For Linux
#endif
    system(command);
}

// Function to clear the history and reset to the home page
void clearHistory() {
    // Free the memory for all nodes in the current list
    while (currentTab != NULL) {
        DoublyNode* temp = currentTab;
        currentTab = currentTab->prev;
        free(temp);
    }
    currentTab = createNode(HOME_PAGE);
    printf("History cleared. Current URL: %s\n", currentTab->url);

    // Free the memory for the history list
    while (historyList != NULL) {
        DoublyNode* temp = historyList;
        historyList = historyList->next;
        free(temp);
    }
    historyList = historyTail = NULL;

    // Open the home page in the default web browser
    char command[200];
#ifdef _WIN32
    sprintf(command, "start %s", HOME_PAGE); // For Windows
#elif __APPLE__
    sprintf(command, "open %s", HOME_PAGE); // For macOS
#else
    sprintf(command, "xdg-open %s", HOME_PAGE); // For Linux
#endif
    system(command);
}

// Function to display the currently active tab
void showCurrentTab() {
    if (currentTab != NULL) {
        printf("Currently active tab: %s\n", currentTab->url);
    } else {
        printf("No active tab.\n");
    }
}

// Function to show the history of visited pages excluding the home page
void showHistory() {
    if (historyList == NULL) {
        printf("No history available to show.\n");
        return;
    }

    printf("History of visited pages:\n");
    DoublyNode* temp = historyList;
    int count = 0;
    while (temp != NULL) {
        printf("%d. %s (Opened at: %s)\n", ++count, temp->url, temp->timestamp);
        temp = temp->next;
    }
}

// Function to revisit a previously opened page excluding the home page
void revisitPage() {
    if (historyList == NULL) {
        printf("No history available to revisit.\n");
        return;
    }

    showHistory();

    printf("Enter the number of the page you want to revisit: ");
    int choice;
    scanf("%d", &choice);

    DoublyNode* temp = historyList;
    for (int i = 1; i < choice && temp != NULL; i++) {
        temp = temp->next;
    }

    if (temp == NULL || choice < 1) {
        printf("Invalid choice.\n");
    } else {
        visit(temp->url);
    }
}

// Function to save the current URL to the saved URLs list
void saveCurrentURL() {
    if (currentTab == NULL || strcmp(currentTab->url, HOME_PAGE) == 0) {
        printf("No active tab to save.\n");
        return;
    }

    DoublyNode* newSavedNode = createNode(currentTab->url);
    if (savedList == NULL) {
        savedList = savedTail = newSavedNode;
    } else {
        savedTail->next = newSavedNode;
        newSavedNode->prev = savedTail;
        savedTail = newSavedNode;
    }

    printf("Saved URL: %s\n", currentTab->url);
}

// Function to show the saved URLs with an option to visit them
void showSavedURLs() {
    if (savedList == NULL) {
        printf("No saved URLs available to show.\n");
        return;
    }

    printf("Saved URLs:\n");
    DoublyNode* temp = savedList;
    int count = 0;
    while (temp != NULL) {
        // Display the timestamp along with the saved URL
        printf("%d. %s (Saved at: %s)\n", ++count, temp->url, temp->timestamp);
        temp = temp->next;
    }

    printf("Enter the number of the URL you want to visit: ");
    int choice;
    scanf("%d", &choice);

    temp = savedList;
    for (int i = 1; i < choice && temp != NULL; i++) {
        temp = temp->next;
    }

    if (temp == NULL || choice < 1) {
        printf("Invalid choice.\n");
    } else {
        visit(temp->url);
    }
}

// Function to delete all saved URLs
void deleteSavedURLs() {
    if (savedList == NULL) {
        printf("No URLs to delete.\n");
        return;
    }

    while (savedList != NULL) {
        DoublyNode* temp = savedList;
        savedList = savedList->next;
        free(temp);
    }
    savedList = savedTail = NULL;
    printf("All saved URLs have been deleted.\n");
}

// Main function to demonstrate the browser tab manager
int main() {
    currentTab = createNode(HOME_PAGE); // Initialize current URL to home page

    int choice;
    char url[100];

    do {
        clearScreen(); // Clear screen before displaying the menu
        printf("\n");
        printf("========================================\n");
        printf("       🌐 Browser Tab Management 🌐      \n");
        printf("========================================\n");
        printf("1.  🚀 Visit a new URL\n");
        printf("2.  🔙 Go back\n");
        printf("3.  🔜 Go forward\n");
        printf("4.  📂 Show current tab\n");
        printf("5.  📌 Save current URL\n");
        printf("6.  🌟 Show saved URLs\n");
        printf("7.  🕑 Show history of visited pages\n");
        printf("8.  🔄 Revisit a previously opened page\n");
        printf("9.  🧹 Clear history\n");
        printf("10. 🗑️  Delete saved URLs\n");
        printf("11. 🚪 Exit\n");
        printf("========================================\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);
        switch (choice) {
            case 1:
                printf("Enter URL: ");
                scanf("%s", url);
                visit(url);
                break;
            case 2:
                goBack();
                break;
            case 3:
                goForward();
                break;
            case 4:
                showCurrentTab();
                break;
            case 5:
                saveCurrentURL();
                break;
            case 6:
                showSavedURLs();
                break;
            case 7:
                showHistory();
                break;
            case 8:
                revisitPage();
                break;
            case 9:
                clearHistory();
                break;
            case 10:
                deleteSavedURLs();
                break;
            case 11:
                printf("Exiting...\n");
                break;
            default:
                printf("Invalid choice. Please try again.\n");
        }
        // Wait for user input before clearing the screen and displaying the menu again
        printf("\nPress Enter to continue...");
        while (getchar() != '\n'); // Clear any leftover newline characters in the input buffer
        getchar(); // Wait for Enter key
    } while (choice != 11);
    return 0;
}
