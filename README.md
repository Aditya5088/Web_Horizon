# Web_Horizon
"Web Horizon" is a lightweight, command-line application built in C that simulates the core functionalities of a web browser's tab and history management.

It uses doubly linked lists to manage user navigation, allowing you to move back and forward through visited tabs. The application also maintains two separate lists: a persistent history log that timestamps every URL visited and a "saved URLs" list for bookmarking sites. It is cross-platform and uses system calls to open the selected URLs in the user's default web browser.

Key Features:

Tab Navigation: Visit new URLs, go back, and go forward.

History Management: Automatically logs visited URLs with timestamps and allows you to view or clear the entire history.

Bookmarking: Save the current URL to a dedicated list, view all saved URLs, and delete them when no longer needed.

System Integration: Launches URLs in your computer's default browser (supports Windows, macOS, and Linux).
