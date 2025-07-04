# Ardur-Team-Tracker
<!DOCTYPE html>
<html lang="en" class="light"> <!-- Default to light mode -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Employee Extended Hours Tracker</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for Inter font and some utility classes not directly in Tailwind */
        body {
            font-family: 'Inter', sans-serif;
            @apply bg-gray-100 text-gray-800; /* Default light mode */
        }
        /* Dark mode specific styles (kept for future proofing if you re-add it, but not actively used) */
        html.dark body {
            @apply bg-gray-900 text-gray-100;
        }

        /* Custom scrollbar for table */
        .table-container::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        .table-container::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
        }
        html.dark .table-container::-webkit-scrollbar-track {
            background: #333;
        }
        .table-container::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 10px;
        }
        .table-container::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
        html.dark .table-container::-webkit-scrollbar-thumb {
            background: #666;
        }
        html.dark .table-container::-webkit-scrollbar-thumb:hover {
            background: #999;
        }


        /* Message Box Styling - Updated for central dialog */
        .message-dialog-backdrop {
            position: fixed; /* Explicitly fixed positioning to viewport */
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent black overlay */
            display: flex; /* Use flexbox for centering content */
            align-items: center; /* Center vertically */
            justify-content: center; /* Center horizontally */
            z-index: 9999; /* High z-index to ensure it's on top of everything */
        }
        .message-dialog {
            /* Tailwind classes for styling the dialog box itself */
            @apply bg-white dark:bg-gray-800 p-8 rounded-lg shadow-xl max-w-sm w-full text-center;
        }

        /* Confirmation Dialog Styling (same as message dialog for consistency) */
        .confirm-dialog-backdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 9999;
        }
        .confirm-dialog {
            @apply bg-white dark:bg-gray-800 p-8 rounded-lg shadow-xl max-w-sm w-full text-center;
        }

        /* Admin Settings Modal Styling */
        .admin-settings-modal-backdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7); /* Darker overlay for settings */
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10000; /* Even higher z-index */
        }
        .admin-settings-modal {
            @apply bg-white dark:bg-gray-900 p-8 rounded-lg shadow-2xl max-w-2xl w-11/12 text-left overflow-y-auto max-h-[90vh];
        }
        .admin-settings-modal h3 {
            @apply text-2xl font-bold mb-6 text-gray-900 dark:text-gray-100;
        }
        .admin-settings-modal pre {
            @apply bg-gray-100 dark:bg-gray-700 p-4 rounded-md text-sm overflow-x-auto;
        }
        .admin-settings-modal button {
            @apply mt-6 px-6 py-3 bg-blue-600 text-white font-semibold rounded-md shadow-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-150 ease-in-out;
        }
    </style>
</head>
<body class="p-4 sm:p-6 md:p-8">
    <!-- Message Dialog -->
    <div id="messageDialogBackdrop" class="message-dialog-backdrop hidden">
        <div id="messageDialog" class="message-dialog">
            <h3 id="messageDialogTitle">Notification</h3>
            <p id="messageDialogMessage"></p>
            <div class="message-dialog-buttons">
                <button id="messageDialogCloseBtn" class="px-6 py-3 bg-blue-600 text-white font-semibold rounded-md shadow-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-150 ease-in-out">Close</button>
            </div>
        </div>
    </div>

    <!-- Confirmation Dialog -->
    <div id="confirmDialogBackdrop" class="confirm-dialog-backdrop hidden">
        <div id="confirmDialog" class="confirm-dialog">
            <h3 id="confirmDialogTitle">Confirm Action</h3>
            <p id="confirmDialogMessage">Are you sure you want to proceed?</p>
            <div class="confirm-dialog-buttons">
                <button id="confirmYesBtn" class="px-6 py-3 bg-red-600 text-white font-semibold rounded-md shadow-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-150 ease-in-out">Yes</button>
                <button id="confirmNoBtn" class="px-6 py-3 bg-gray-300 text-gray-800 font-semibold rounded-md shadow-lg hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition duration-150 ease-in-out">No</button>
            </div>
        </div>
    </div>

    <!-- Admin Settings Modal -->
    <div id="adminSettingsModalBackdrop" class="admin-settings-modal-backdrop hidden">
        <div id="adminSettingsModal" class="admin-settings-modal">
            <h3 id="adminSettingsTitle">Admin Settings</h3>
            <div class="mb-4">
                <p class="text-gray-700 dark:text-gray-300 mb-2 font-semibold">Predefined Employee List (Read-Only via UI):</p>
                <pre id="employeeListDisplay" class="text-gray-800 dark:text-gray-200"></pre>
                <p class="text-sm text-gray-500 dark:text-gray-400 mt-2">To modify this list, please edit the 'predefinedTeamMembers' array directly in the application's HTML code.</p>
            </div>
            <button id="closeAdminSettingsBtn">Close Settings</button>
        </div>
    </div>


    <div class="max-w-7xl mx-auto bg-white dark:bg-gray-800 p-6 rounded-xl shadow-lg">
        <!-- Header -->
        <header class="mb-8 text-center relative">
            <h1 class="text-4xl sm:text-5xl font-extrabold text-indigo-700 dark:text-indigo-400 mb-2">Employee Extended Hours Tracker</h1>
            <p class="text-lg text-gray-600 dark:text-gray-300">Efficiently log and track employee extended working hours.</p>
        </header>

        <!-- User ID Display -->
        <div class="mb-4 text-center text-sm text-gray-500 dark:text-gray-400">
            Your Firebase User ID: <span id="currentUserIdDisplay" class="font-semibold text-gray-700 dark:text-gray-300">Loading...</span>
        </div>

        <!-- Control Buttons: Admin Mode and Admin Settings -->
        <section class="mb-8 flex flex-col sm:flex-row justify-end space-y-4 sm:space-y-0 sm:space-x-4">
            <button id="toggleAdminModeBtn" class="px-6 py-3 bg-yellow-500 text-white font-semibold rounded-md shadow-lg hover:bg-yellow-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-yellow-500 transition duration-150 ease-in-out hidden">
                Enable Admin Mode
            </button>
            <button id="adminSettingsBtn" class="px-6 py-3 bg-purple-600 text-white font-semibold rounded-md shadow-lg hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500 transition duration-150 ease-in-out hidden">
                Admin Settings
            </button>
        </section>

        <!-- Admin Access Section -->
        <section id="adminAccessSection" class="mb-10 p-6 bg-red-50 dark:bg-red-900 rounded-lg shadow-md">
            <h2 class="text-2xl font-bold text-red-800 dark:text-red-300 mb-6">Admin Access</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 items-end">
                <div>
                    <label for="adminIdInput" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Admin Employee ID</label>
                    <input type="text" id="adminIdInput" placeholder="Enter Admin ID" class="w-full p-3 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:ring-red-500 focus:border-red-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100">
                </div>
                <div>
                    <label for="adminPasswordInput" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Admin Password</label>
                    <input type="password" id="adminPasswordInput" placeholder="Enter Password" class="w-full p-3 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:ring-red-500 focus:border-red-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100">
                </div>
                <div class="flex flex-col sm:flex-row gap-4">
                    <button id="adminLoginBtn" class="flex-1 px-6 py-3 bg-red-600 text-white font-semibold rounded-md shadow-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-150 ease-in-out">Admin Login</button>
                    <button id="adminLogoutBtn" class="flex-1 px-6 py-3 bg-gray-500 text-white font-semibold rounded-md shadow-lg hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-400 transition duration-150 ease-in-out" disabled>Admin Logout</button>
                </div>
            </div>
            <p id="adminStatus" class="mt-4 text-center text-lg font-medium text-gray-700 dark:text-gray-300"></p>
        </section>

        <!-- Employee Login/Logout Section -->
        <section class="mb-10 p-6 bg-indigo-50 dark:bg-indigo-900 rounded-lg shadow-md">
            <h2 class="text-2xl font-bold text-indigo-800 dark:text-indigo-300 mb-6">Employee Login/Logout</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 items-end">
                <div>
                    <label for="loginMemberId" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Your Employee ID</label>
                    <input type="text" id="loginMemberId" placeholder="Enter your Employee ID" class="w-full p-3 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100">
                </div>
                <div class="flex flex-col sm:flex-row gap-4">
                    <button id="loginBtn" class="flex-1 px-6 py-3 bg-green-600 text-white font-semibold rounded-md shadow-lg hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition duration-150 ease-in-out">Login</button>
                    <button id="logoutBtn" class="flex-1 px-6 py-3 bg-red-600 text-white font-semibold rounded-md shadow-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-150 ease-in-out" disabled>Logout</button>
                </div>
            </div>
            <p id="employeeLoginStatus" class="mt-4 text-center text-lg font-medium text-gray-700 dark:text-gray-300"></p>
        </section>

        <!-- Filter Section -->
        <section class="mb-10 p-6 bg-blue-50 dark:bg-blue-900 rounded-lg shadow-md">
            <h2 class="text-2xl font-bold text-blue-800 dark:text-blue-300 mb-6">Filter Entries</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                    <label for="filterStartDate" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">From Date</label>
                    <input type="date" id="filterStartDate" class="w-full p-3 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100">
                </div>
                <div>
                    <label for="filterEndDate" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">To Date</label>
                    <input type="date" id="filterEndDate" class="w-full p-3 border border-gray-300 dark:border-gray-600 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100">
                </div>
            </div>
            <div class="mt-6 flex justify-end space-x-4">
                <button id="applyFilterBtn" class="px-6 py-3 bg-blue-600 text-white font-semibold rounded-md shadow-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-150 ease-in-out">Apply Filter</button>
                <button id="clearFilterBtn" class="px-6 py-3 bg-gray-300 text-gray-800 font-semibold rounded-md shadow-lg hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition duration-150 ease-in-out">Clear Filter</button>
            </div>
        </section>

        <!-- Table to Display Entries -->
        <section class="mb-10">
            <h2 class="text-2xl font-bold text-gray-800 dark:text-gray-100 mb-6">Extended Hours Log</h2>
            <div class="overflow-x-auto rounded-lg shadow-md table-container max-h-96">
                <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                    <thead class="bg-gray-50 dark:bg-gray-700 sticky top-0">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider delete-column-header hidden">
                                <input type="checkbox" id="selectAllCheckboxes" class="form-checkbox h-4 w-4 text-indigo-600 transition duration-150 ease-in-out">
                            </th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">Employee Name</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">Employee ID</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">Date</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">Start Time</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">End Time</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider">Total Hours</th>
                        </tr>
                    </thead>
                    <tbody id="entriesTableBody" class="bg-white dark:bg-gray-800 divide-y divide-gray-200 dark:divide-gray-700">
                        <!-- Entries will be dynamically inserted here -->
                    </tbody>
                </table>
            </div>
            <div class="mt-6 flex justify-end">
                <button id="deleteSelectedBtn" class="px-6 py-3 bg-red-600 text-white font-semibold rounded-md shadow-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-150 ease-in-out hidden">Delete Selected Entries</button>
                <button id="exportCsvBtn" class="px-6 py-3 bg-green-600 text-white font-semibold rounded-md shadow-lg hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition duration-150 ease-in-out ml-4 hidden">Export to CSV</button>
            </div>
        </section>

        <!-- Summary Section -->
        <section class="mb-10 p-6 bg-purple-50 dark:bg-purple-900 rounded-lg shadow-md">
            <h2 class="text-2xl font-bold text-purple-800 dark:text-purple-300 mb-6">Summary: Total Hours Per Employee</h2>
            <div id="summarySection" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                <!-- Summary will be dynamically inserted here -->
                <p class="text-gray-600 dark:text-gray-400 col-span-full" id="noSummaryMessage">No data to display summary.</p>
            </div>
        </section>
    </div>

    <script type="module">
        // Firebase Imports (required for cloud database)
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, doc, deleteDoc, onSnapshot, query, orderBy } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-firestore.js";

        // Firebase Initialization Variables
        let app;
        let db;
        let auth;
        let currentUserId = null; // Store Firebase Authentication User ID

        // Your Firebase Project Configuration
        const firebaseConfig = {
          apiKey: "AIzaSyBQkFyn2bahuD4jp6Z-1CTU2qg2sDwWTPQ",
          authDomain: "team-track-fa1b7.firebaseapp.com",
          projectId: "team-track-fa1b7",
          storageBucket: "team-track-fa1b7.firebasestorage.app",
          messagingSenderId: "764858028770",
          appId: "1:764858028770:web:6fcdcfcd921401c36284bb",
          measurementId: "G-3F59SXF215"
        };
        // Use your projectId as the appId for the Firestore collection path
        const appId = firebaseConfig.projectId;

        // --- EMPLOYEE & ADMIN ACCESS CONFIGURATION ---
        // Predefined list of employees with their IDs and Names.
        // You MUST populate this list with all your team members.
        const predefinedTeamMembers = [
            { id: "AT0001", name: "Alice Johnson" },
            { id: "AT0002", name: "Bob Williams" },
            { id: "AT0085", name: "rushi" }, // This is the admin ID and name
            { id: "AT0003", name: "Charlie Brown" },
            { id: "AT0004", name: "Diana Prince" }
            // Add more team members here: { id: "YOUR_EMPLOYEE_ID", name: "Employee Name" },
        ];

        // The Employee ID that will have admin privileges
        const ADMIN_EMPLOYEE_ID = "AT0085"; 
        // !!! IMPORTANT: FOR DEMONSTRATION ONLY - DO NOT USE HARDCODED PASSWORDS IN PRODUCTION !!!
        // For a real application, implement secure authentication (e.g., Firebase Auth with email/password).
        const ADMIN_PASSWORD = "RUSHI"; // <--- SET YOUR ADMIN PASSWORD HERE
        // ---------------------------------------------

        // Array to store extended hours entries (populated by Firestore)
        let extendedHoursEntries = [];
        let selectedEntryIds = new Set(); // To store IDs of selected entries for multi-delete

        // State variable for current logged-in employee details (for regular users)
        let currentLoggedInEmployee = null; // Stores { id: "AT0085", name: "rushi", loginTime: Date object }

        // State variable for admin login status (separate from employee login)
        let isAdminLoggedIn = false;

        // State variables for Admin Mode and Dark Mode
        let isAdminMode = false;
        let darkModeEnabled = false; // Still exists for local storage persistence, but no longer toggled by UI

        // DOM Elements - Employee Login
        const loginMemberIdInput = document.getElementById('loginMemberId');
        const loginBtn = document.getElementById('loginBtn');
        const logoutBtn = document.getElementById('logoutBtn');
        const employeeLoginStatus = document.getElementById('employeeLoginStatus');

        // DOM Elements - Admin Access
        const adminAccessSection = document.getElementById('adminAccessSection');
        const adminIdInput = document.getElementById('adminIdInput');
        const adminPasswordInput = document.getElementById('adminPasswordInput');
        const adminLoginBtn = document.getElementById('adminLoginBtn');
        const adminLogoutBtn = document.getElementById('adminLogoutBtn');
        const adminStatus = document.getElementById('adminStatus');

        // DOM Elements - General Controls
        const entriesTableBody = document.getElementById('entriesTableBody');
        const summarySection = document.getElementById('summarySection');
        const exportCsvBtn = document.getElementById('exportCsvBtn');
        const filterStartDateInput = document.getElementById('filterStartDate');
        const filterEndDateInput = document.getElementById('filterEndDate');
        const applyFilterBtn = document.getElementById('applyFilterBtn');
        const clearFilterBtn = document.getElementById('clearFilterBtn');
        const noSummaryMessage = document.getElementById('noSummaryMessage');
        const currentUserIdDisplay = document.getElementById('currentUserIdDisplay');

        const toggleAdminModeBtn = document.getElementById('toggleAdminModeBtn');
        const adminSettingsBtn = document.getElementById('adminSettingsBtn');
        // Removed: const toggleDarkModeBtn = document.getElementById('toggleDarkModeBtn'); // This line was removed
        // Removed: const darkModeIndicator = document.getElementById('darkModeIndicator');

        // DOM Elements - Multi-Delete
        const selectAllCheckboxes = document.getElementById('selectAllCheckboxes');
        const deleteSelectedBtn = document.getElementById('deleteSelectedBtn');

        // Admin Settings Modal Elements
        const adminSettingsModalBackdrop = document.getElementById('adminSettingsModalBackdrop');
        const closeAdminSettingsBtn = document.getElementById('closeAdminSettingsBtn');
        const employeeListDisplay = document.getElementById('employeeListDisplay');

        // Confirmation Dialog Elements
        const confirmDialogBackdrop = document.getElementById('confirmDialogBackdrop');
        const confirmDialogTitle = document.getElementById('confirmDialogTitle');
        const confirmDialogMessage = document.getElementById('confirmDialogMessage');
        const confirmYesBtn = document.getElementById('confirmYesBtn');
        const confirmNoBtn = document.getElementById('confirmNoBtn');

        // Message Dialog Elements
        const messageDialogBackdrop = document.getElementById('messageDialogBackdrop');
        const messageDialogTitle = document.getElementById('messageDialogTitle');
        const messageDialogMessage = document.getElementById('messageDialogMessage');
        const messageDialogCloseBtn = document.getElementById('messageDialogCloseBtn');


        // Function to display messages in a custom centered dialog
        function showMessage(message, type = 'info') {
            messageDialogMessage.textContent = message;
            if (type === 'success') {
                messageDialogTitle.textContent = "Success!";
                messageDialogTitle.classList.add('text-green-600');
                messageDialogTitle.classList.remove('text-red-600', 'text-blue-600');
            } else if (type === 'error') {
                messageDialogTitle.textContent = "Error!";
                messageDialogTitle.classList.add('text-red-600');
                messageDialogTitle.classList.remove('text-green-600', 'text-blue-600');
            } else {
                messageDialogTitle.textContent = "Notification";
                messageDialogTitle.classList.add('text-blue-600');
                messageDialogTitle.classList.remove('text-green-600', 'text-red-600');
            }
            messageDialogBackdrop.classList.remove('hidden');
            console.log(`[Message: ${type}] ${message}`); // Log messages to console for debugging
        }

        // Function to hide custom message dialog
        function hideMessageDialog() {
            messageDialogBackdrop.classList.add('hidden');
        }

        // Event listener for message dialog close button
        messageDialogCloseBtn.addEventListener('click', hideMessageDialog);


        // Function to show custom confirmation dialog
        let currentConfirmCallback = null;
        function showConfirmDialog(message, onConfirmCallback) {
            confirmDialogMessage.textContent = message;
            confirmDialogBackdrop.classList.remove('hidden');
            currentConfirmCallback = onConfirmCallback;
        }

        // Function to hide custom confirmation dialog
        function hideConfirmDialog() {
            confirmDialogBackdrop.classList.add('hidden');
            currentConfirmCallback = null;
        }

        // Event listeners for confirmation dialog buttons
        confirmYesBtn.addEventListener('click', () => {
            if (currentConfirmCallback) {
                currentConfirmCallback(true);
            }
            hideConfirmDialog();
        });

        confirmNoBtn.addEventListener('click', () => {
            if (currentConfirmCallback) {
                currentConfirmCallback(false);
            }
            hideConfirmDialog();
        });


        // Function to load local preferences from Local Storage (not data)
        function loadLocalState() {
            console.log("Loading local preferences from localStorage...");
            const data = localStorage.getItem('extendedHoursTrackerLocalState');
            if (data) {
                try {
                    const parsedData = JSON.parse(data);
                    if (parsedData.currentLoggedInEmployee && parsedData.currentLoggedInEmployee.id && parsedData.currentLoggedInEmployee.name && parsedData.currentLoggedInEmployee.loginTime) {
                        currentLoggedInEmployee = {
                            id: parsedData.currentLoggedInEmployee.id,
                            name: parsedData.currentLoggedInEmployee.name,
                            loginTime: new Date(parsedData.currentLoggedInEmployee.loginTime)
                        };
                        console.log("Local state: Logged in employee loaded.", currentLoggedInEmployee);
                    }
                    // Removed: darkModeEnabled = parsedData.darkModeEnabled || false;
                    isAdminMode = parsedData.isAdminMode || false; // Load admin mode state
                    isAdminLoggedIn = parsedData.isAdminLoggedIn || false; // Load admin login state
                    console.log("Local state: Admin mode?", isAdminMode, "Admin logged in?", isAdminLoggedIn);

                } catch (e) {
                    console.error("Error parsing local state from localStorage:", e);
                    currentLoggedInEmployee = null;
                    // Removed: darkModeEnabled = false;
                    isAdminMode = false;
                    isAdminLoggedIn = false;
                    showMessage("Error loading saved preferences. Resetting local settings.", "error");
                }
            } else {
                console.log("No local state found in localStorage.");
                currentLoggedInEmployee = null;
                // Removed: darkModeEnabled = false;
                isAdminMode = false;
                isAdminLoggedIn = false;
            }
            updateLoginButtons(); // Update button states after loading local state
            updateAdminUI(); // Update admin specific UI
            // Removed: applyDarkMode(darkModeEnabled); // Apply dark mode state
        }

        // Function to save local preferences to Local Storage
        function saveLocalState() {
            const dataToSave = {
                currentLoggedInEmployee: currentLoggedInEmployee ? {
                    id: currentLoggedInEmployee.id,
                    name: currentLoggedInEmployee.name,
                    loginTime: currentLoggedInEmployee.loginTime.toISOString() // Store Date as ISO string
                } : null,
                // Removed: darkModeEnabled: darkModeEnabled,
                isAdminMode: isAdminMode, // Save admin mode state
                isAdminLoggedIn: isAdminLoggedIn // Save admin login state
            };
            localStorage.setItem('extendedHoursTrackerLocalState', JSON.stringify(dataToSave));
            console.log("Local state saved to localStorage.", dataToSave);
        }

        // Function to calculate total hours between two Date objects
        function calculateTotalHoursFromDates(startDateObj, endDateObj) {
            if (!startDateObj || !endDateObj || !(startDateObj instanceof Date) || !(endDateObj instanceof Date)) {
                console.error("Invalid Date objects for total hours calculation.");
                return 'N/A';
            }

            const diffMs = endDateObj - startDateObj;
            if (diffMs < 0) {
                console.warn("End date is earlier than start date for calculation. Returning 0 hours.");
                return '0.00';
            }
            const totalHours = diffMs / (1000 * 60 * 60); // Convert milliseconds to hours
            return totalHours.toFixed(2); // Round to 2 decimal places
        }

        // Function to render the table with entries
        function renderTable(entriesToDisplay = extendedHoursEntries) {
            entriesTableBody.innerHTML = ''; // Clear existing entries
            selectedEntryIds.clear(); // Clear selections on re-render
            if (selectAllCheckboxes) selectAllCheckboxes.checked = false; // Uncheck select all

            const deleteColumnHeader = document.querySelector('.delete-column-header');
            
            if (isAdminMode) {
                if (deleteColumnHeader) deleteColumnHeader.classList.remove('hidden');
            } else {
                if (deleteColumnHeader) deleteColumnHeader.classList.add('hidden');
            }

            if (entriesToDisplay.length === 0) {
                const noDataRow = document.createElement('tr');
                noDataRow.innerHTML = `<td colspan="7" class="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-500 dark:text-gray-400">No entries to display.</td>`;
                entriesTableBody.appendChild(noDataRow);
                return;
            }

            entriesToDisplay.forEach((entry, index) => {
                const row = document.createElement('tr');
                row.className = index % 2 === 0 ? 'bg-white dark:bg-gray-800' : 'bg-gray-50 dark:bg-gray-700';
                row.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 dark:text-gray-100 ${isAdminMode ? '' : 'hidden'}">
                        <input type="checkbox" data-id="${entry.id}" class="entry-checkbox form-checkbox h-4 w-4 text-indigo-600 transition duration-150 ease-in-out">
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 dark:text-gray-100">${entry.memberName}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${entry.employeeId}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${new Date(entry.date).toLocaleDateString('en-GB')}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${entry.startTime}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${entry.endTime}</td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${entry.totalHours}</td>
                `;
                entriesTableBody.appendChild(row);

                // Add event listener for individual checkboxes
                const checkbox = row.querySelector('.entry-checkbox');
                if (checkbox) {
                    checkbox.addEventListener('change', (e) => {
                        if (e.target.checked) {
                            selectedEntryIds.add(e.target.dataset.id);
                        } else {
                            selectedEntryIds.delete(e.target.dataset.id);
                        }
                        updateDeleteSelectedButtonVisibility();
                    });
                }
            });
            updateDeleteSelectedButtonVisibility(); // Update button visibility after rendering
        }

        // Function to toggle individual entry selection
        function toggleEntrySelection(id, isChecked) {
            if (isChecked) {
                selectedEntryIds.add(id);
            } else {
                selectedEntryIds.delete(id);
            }
            updateDeleteSelectedButtonVisibility();
        }

        // Event listener for "Select All" checkbox
        if (selectAllCheckboxes) {
            selectAllCheckboxes.addEventListener('change', (e) => {
                const isChecked = e.target.checked;
                document.querySelectorAll('.entry-checkbox').forEach(checkbox => {
                    checkbox.checked = isChecked;
                    toggleEntrySelection(checkbox.dataset.id, isChecked);
                });
            });
        }

        // Function to update the visibility of the "Delete Selected" button
        function updateDeleteSelectedButtonVisibility() {
            if (isAdminMode && selectedEntryIds.size > 0) {
                deleteSelectedBtn.classList.remove('hidden');
            } else {
                deleteSelectedBtn.classList.add('hidden');
            }
        }

        // Event listener for "Delete Selected Entries" button
        deleteSelectedBtn.addEventListener('click', () => {
            if (selectedEntryIds.size === 0) {
                showMessage("No entries selected for deletion.", "info");
                return;
            }

            showConfirmDialog(`Are you sure you want to delete ${selectedEntryIds.size} selected entries?`, async (confirmed) => {
                if (confirmed) {
                    const idsToDelete = Array.from(selectedEntryIds);
                    let deletedCount = 0;
                    for (const id of idsToDelete) {
                        try {
                            const docRef = doc(db, `artifacts/${appId}/public/data/extendedHours`, id);
                            await deleteDoc(docRef);
                            deletedCount++;
                        } catch (e) {
                            console.error(`Error deleting document ${id}:`, e);
                            showMessage(`Failed to delete entry ${id}. Check console.`, "error");
                        }
                    }
                    showMessage(`${deletedCount} entries successfully deleted!`, "success");
                    selectedEntryIds.clear(); // Clear selections
                    // onSnapshot will re-render the table automatically
                } else {
                    showMessage("Deletion cancelled.", "info");
                }
            });
        });


        // Function to update the summary section
        function updateSummary(entriesForSummary = extendedHoursEntries) {
            const summary = {};
            entriesForSummary.forEach(entry => {
                const hours = parseFloat(entry.totalHours);
                if (!isNaN(hours)) {
                    // Summarize by employee name
                    summary[entry.memberName] = (summary[entry.memberName] || 0) + hours;
                }
            });

            summarySection.innerHTML = ''; // Clear existing summary

            if (Object.keys(summary).length === 0) {
                noSummaryMessage.classList.remove('hidden');
                summarySection.appendChild(noSummaryMessage);
                return;
            } else {
                noSummaryMessage.classList.add('hidden');
            }


            for (const member in summary) {
                const card = document.createElement('div');
                card.className = 'bg-white dark:bg-gray-700 p-4 rounded-lg shadow flex flex-col items-center justify-center';
                card.innerHTML = `
                    <p class="text-lg font-semibold text-purple-700 dark:text-purple-300">${member}</p>
                    <p class="text-2xl font-bold text-purple-900 dark:text-purple-200">${summary[member].toFixed(2)} hrs</p>
                `;
                summarySection.appendChild(card);
            }
        }

        // Function to update the state of employee login/logout buttons and status message
        function updateLoginButtons() {
            if (currentLoggedInEmployee) {
                loginMemberIdInput.value = currentLoggedInEmployee.id; // Display logged-in ID
                loginMemberIdInput.disabled = true;
                loginBtn.disabled = true;
                logoutBtn.disabled = false;
                employeeLoginStatus.textContent = `Currently logged in: ${currentLoggedInEmployee.name} (${currentLoggedInEmployee.id}) (since ${currentLoggedInEmployee.loginTime.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' })})`;
            } else {
                loginMemberIdInput.value = '';
                loginMemberIdInput.disabled = false;
                loginBtn.disabled = false;
                logoutBtn.disabled = true;
                employeeLoginStatus.textContent = 'No one is logged in.';
            }
            // Admin-specific buttons are controlled by updateAdminUI
            updateAdminUI();
        }

        // Employee Login Button Click Handler
        loginBtn.addEventListener('click', () => {
            const enteredEmployeeId = loginMemberIdInput.value.trim().toUpperCase(); // Convert to uppercase for case-insensitive matching
            
            // Find the employee in the predefined list
            const employee = predefinedTeamMembers.find(member => member.id === enteredEmployeeId);

            if (!enteredEmployeeId) {
                showMessage("Please enter your Employee ID to login.", "error");
                return;
            }
            if (!employee) {
                showMessage("Invalid Employee ID. Please enter a valid ID from your team.", "error");
                loginMemberIdInput.value = ''; // Clear input on invalid ID
                return;
            }

            if (currentLoggedInEmployee) {
                showMessage(`Error: ${currentLoggedInEmployee.name} (${currentLoggedInEmployee.id}) is already logged in. Please log out first.`, "error");
                return;
            }

            currentLoggedInEmployee = {
                id: employee.id,
                name: employee.name,
                loginTime: new Date()
            };
            showMessage(`Logged in as ${currentLoggedInEmployee.name} (${currentLoggedInEmployee.id})!`, "success");

            saveLocalState(); // Save the login state locally
            updateLoginButtons();
            // Data will be rendered by Firestore listener, no need to call renderTable here
        });

        // Employee Logout Button Click Handler
        logoutBtn.addEventListener('click', async () => {
            if (!currentLoggedInEmployee) {
                showMessage("No one is currently logged in.", "error");
                return;
            }

            const logoutTime = new Date();
            // Calculate total hours from the stored loginTime and current logoutTime
            const totalHours = calculateTotalHoursFromDates(currentLoggedInEmployee.loginTime, logoutTime);

            const newEntry = {
                employeeId: currentLoggedInEmployee.id, // Save employee ID
                memberName: currentLoggedInEmployee.name, // Save employee name
                date: currentLoggedInEmployee.loginTime.toISOString().split('T')[0], //YYYY-MM-DD string
                startTime: currentLoggedInEmployee.loginTime.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' }),
                endTime: logoutTime.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' }),
                totalHours: parseFloat(totalHours) // Ensure it's a number
            };

            try {
                if (!db) {
                    showMessage("Database not initialized. Cannot save entry. Please ensure Firebase is configured.", "error");
                    console.error("Add failed: DB not ready.");
                    return;
                }
                console.log("Attempting to add new document to Firestore:", newEntry);
                // The collection path will include the specific appId (your Firebase Project ID)
                const collectionRef = collection(db, `artifacts/${appId}/public/data/extendedHours`);
                await addDoc(collectionRef, {
                    employeeId: newEntry.employeeId,
                    memberName: newEntry.memberName,
                    date: newEntry.date,
                    startTime: newEntry.startTime,
                    endTime: newEntry.endTime,
                    totalHours: newEntry.totalHours,
                    createdAt: new Date().toISOString() // Add a timestamp for potential future ordering needs
                });
                console.log("Document successfully added to Firestore.");
                showMessage(`Logged out. ${newEntry.memberName} (${newEntry.employeeId}) worked ${newEntry.totalHours} hours. Entry saved!`, "success");
            } catch (e) {
                console.error("Error adding document to Firestore: ", e);
                showMessage("Failed to save entry. Please check console for details and Firebase setup.", "error");
            }

            currentLoggedInEmployee = null; // Clear logged in state
            saveLocalState(); // Clear logged in state locally
            updateLoginButtons();
            // applyFilter() is not needed here; onSnapshot will trigger renderTable
        });


        // Function to export table data to CSV
        exportCsvBtn.addEventListener('click', () => {
            if (!isAdminLoggedIn || !isAdminMode) { // Ensure only admin in admin mode can export
                showMessage("You must be logged in as an Admin and Admin Mode must be enabled to export data.", "error");
                return;
            }
            if (extendedHoursEntries.length === 0) {
                showMessage("No data to export.", "info");
                return;
            }

            // Get current filtered/displayed data (from the global array which is updated by Firestore)
            const filteredEntries = filterEntriesByDate(
                filterStartDateInput.value ? new Date(filterStartDateInput.value) : null,
                filterEndDateInput.value ? new Date(filterEndDateInput.value) : null
            );

            let csvContent = "Employee Name,Employee ID,Date,Start Time,End Time,Total Hours\n"; // CSV header

            filteredEntries.forEach(entry => {
                const row = [
                    entry.memberName,
                    entry.employeeId,
                    new Date(entry.date).toLocaleDateString('en-GB'),
                    entry.startTime,
                    entry.endTime,
                    entry.totalHours
                ].map(item => `"${item}"`).join(','); // Enclose items in quotes to handle commas within data
                csvContent += row + "\n";
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.setAttribute('download', 'extended_hours.csv');
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showMessage("Data exported to CSV!", "success");
        });

        // Function to filter entries by date range
        function filterEntriesByDate(startDate, endDate) {
            return extendedHoursEntries.filter(entry => {
                // Ensure entry.date is handled as a Date object for comparison, even if it comes as a string from Firestore
                const entryDate = new Date(entry.date);
                entryDate.setHours(0, 0, 0, 0); // Normalize to start of day for comparison

                const startMatch = !startDate || entryDate >= startDate;
                const endMatch = !endDate || entryDate <= endDate;
                return startMatch && endMatch;
            });
        }

        // Function to apply filter
        function applyFilter() {
            const startDate = filterStartDateInput.value ? new Date(filterStartDateInput.value) : null;
            const endDate = filterEndDateInput.value ? new Date(filterEndDateInput.value) : null;

            if (startDate) startDate.setHours(0, 0, 0, 0);
            if (endDate) endDate.setHours(23, 59, 59, 999); // Set to end of day for inclusive range

            // Basic validation for date range
            if (startDate && endDate && startDate > endDate) {
                showMessage("Filter 'From Date' cannot be after 'To Date'.", "error");
                return;
            }

            const filtered = filterEntriesByDate(startDate, endDate);
            renderTable(filtered);
            updateSummary(filtered);
            // showMessage("Filter applied.", "info"); // Only show if filter actually changes
        }

        // Event listeners for filter buttons
        applyFilterBtn.addEventListener('click', applyFilter);
        clearFilterBtn.addEventListener('click', () => {
            filterStartDateInput.value = '';
            filterEndDateInput.value = '';
            applyFilter(); // Clear filter and re-render all data
            showMessage("Filter cleared!", "info");
        });

        // Admin Login Button Click Handler
        adminLoginBtn.addEventListener('click', () => {
            const enteredAdminId = adminIdInput.value.trim().toUpperCase();
            const enteredAdminPassword = adminPasswordInput.value.trim();

            if (enteredAdminId === ADMIN_EMPLOYEE_ID && enteredAdminPassword === ADMIN_PASSWORD) {
                isAdminLoggedIn = true;
                isAdminMode = true; // Auto-enable admin mode on admin login
                adminStatus.textContent = `Admin (${ADMIN_EMPLOYEE_ID}) logged in. Admin Mode Enabled.`;
                showMessage("Admin login successful! Admin Mode activated.", "success");
            } else {
                isAdminLoggedIn = false;
                isAdminMode = false;
                adminStatus.textContent = "Invalid Admin ID or Password.";
                showMessage("Invalid Admin ID or Password.", "error");
            }
            saveLocalState();
            updateAdminUI(); // Update admin specific UI
            renderTable(); // Re-render table to show/hide admin controls
        });

        // Admin Logout Button Click Handler
        adminLogoutBtn.addEventListener('click', () => {
            isAdminLoggedIn = false;
            isAdminMode = false; // Disable admin mode on logout
            adminIdInput.value = '';
            adminPasswordInput.value = '';
            adminStatus.textContent = "Admin logged out.";
            showMessage("Admin logged out.", "info");
            saveLocalState();
            updateAdminUI(); // Update admin specific UI
            renderTable(); // Re-render table to hide admin controls
        });

        // Toggle Admin Mode Button Click Handler
        toggleAdminModeBtn.addEventListener('click', () => {
            if (isAdminLoggedIn) { // Only allow if admin is logged in
                isAdminMode = !isAdminMode;
                saveLocalState();
                updateAdminUI();
                renderTable(); // Re-render table to hide/show delete buttons
                showMessage(`Admin Mode: ${isAdminMode ? 'Enabled' : 'Disabled'}`, "info");
            } else {
                showMessage("You must be logged in as an Admin to toggle Admin Mode.", "error");
            }
        });

        // Function to open Admin Settings Modal
        adminSettingsBtn.addEventListener('click', () => {
            if (isAdminLoggedIn && isAdminMode) {
                employeeListDisplay.textContent = JSON.stringify(predefinedTeamMembers, null, 2); // Display formatted JSON
                adminSettingsModalBackdrop.classList.remove('hidden');
            } else {
                showMessage("You must be logged in as an Admin and Admin Mode must be enabled to access settings.", "error");
            }
        });

        // Function to close Admin Settings Modal
        closeAdminSettingsBtn.addEventListener('click', () => {
            adminSettingsModalBackdrop.classList.add('hidden');
        });


        // Update Admin UI (buttons, sections visibility)
        function updateAdminUI() {
            // Admin Access Section visibility
            if (isAdminLoggedIn) {
                adminIdInput.disabled = true;
                adminPasswordInput.disabled = true;
                adminLoginBtn.disabled = true;
                adminLogoutBtn.disabled = false;
            } else {
                adminIdInput.disabled = false;
                adminPasswordInput.disabled = false;
                adminLoginBtn.disabled = false;
                adminLogoutBtn.disabled = true;
            }

            // Admin-specific control buttons visibility
            if (isAdminLoggedIn) {
                toggleAdminModeBtn.classList.remove('hidden');
                adminSettingsBtn.classList.remove('hidden');
                toggleAdminModeBtn.textContent = isAdminMode ? 'Disable Admin Mode' : 'Enable Admin Mode';
                toggleAdminModeBtn.classList.toggle('bg-yellow-500', !isAdminMode);
                toggleAdminModeBtn.classList.toggle('hover:bg-yellow-600', !isAdminMode);
                toggleAdminModeBtn.classList.toggle('focus:ring-yellow-500', !isAdminMode);
                toggleAdminModeBtn.classList.toggle('bg-gray-500', isAdminMode);
                toggleAdminModeBtn.classList.toggle('hover:bg-gray-600', isAdminMode);
                toggleAdminModeBtn.classList.toggle('focus:ring-gray-400', isAdminMode);

                // Control CSV Export and Delete Selected buttons based on isAdminMode
                if (exportCsvBtn) {
                   exportCsvBtn.classList.toggle('hidden', !isAdminMode);
                }
                if (deleteSelectedBtn) {
                    deleteSelectedBtn.classList.toggle('hidden', !isAdminMode || selectedEntryIds.size === 0);
                }
                if (selectAllCheckboxes) {
                    const deleteColumnHeader = document.querySelector('.delete-column-header');
                    if (deleteColumnHeader) deleteColumnHeader.classList.toggle('hidden', !isAdminMode);
                }

            } else {
                toggleAdminModeBtn.classList.add('hidden');
                adminSettingsBtn.classList.add('hidden');
                isAdminMode = false; // Ensure admin mode is off when not logged in as admin
                if (exportCsvBtn) {
                   exportCsvBtn.classList.add('hidden');
                }
                if (deleteSelectedBtn) {
                    deleteSelectedBtn.classList.add('hidden');
                }
                if (selectAllCheckboxes) {
                    selectAllCheckboxes.checked = false; // Uncheck select all
                    const deleteColumnHeader = document.querySelector('.delete-column-header');
                    if (deleteColumnHeader) deleteColumnHeader.classList.add('hidden');
                }
            }
        }

        // Removed: Function to toggle Dark Mode
        // Removed: function toggleDarkMode() { ... }

        // Keeping applyDarkMode for initial load, but it no longer toggles
        function applyDarkMode(enabled) {
            // This function now just ensures the initial state is light mode
            // as dark mode toggle functionality has been removed.
            document.documentElement.classList.remove('dark');
            document.documentElement.classList.add('light');
            // Removed: toggleDarkModeBtn.textContent = 'Toggle Dark Mode';
        }


        // Event listener for toggle Dark Mode button (now just a placeholder/removed)
        // Removed: toggleDarkModeBtn.addEventListener('click', toggleDarkMode);


        // Initialize Firebase and set up Firestore listener
        document.addEventListener('DOMContentLoaded', () => {
            loadLocalState(); // Load local preferences first

            // --- IMPORTANT: PLACEHOLDER CHECK FOR FIREBASE CONFIG ---
            // This check ensures that you have actually replaced the placeholder values in firebaseConfig.
            // If these values are still the default placeholders, Firebase cannot connect and will show errors.
            if (firebaseConfig.apiKey === "YOUR_ACTUAL_API_KEY" || firebaseConfig.projectId === "YOUR_ACTUAL_PROJECT_ID" || !firebaseConfig.apiKey || !firebaseConfig.projectId) {
                console.error("Firebase Configuration Error: Placeholder API Key or Project ID found, or config is incomplete. Please update firebaseConfig with your actual Firebase project details from the Firebase Console.");
                showMessage("Firebase Setup Error: Your API Key is not valid or configuration is missing. Please replace the placeholder values in 'firebaseConfig' in the code with your actual Firebase project details from the Firebase Console.", "error");
                return; // Stop further Firebase initialization
            }
            // --- END PLACEHOLDER CHECK ---

            // Proceed with Firebase initialization only if the config seems valid
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                console.log("Firebase App, Firestore, and Auth initialized with provided config.");

                // Authenticate and then set up listener
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        currentUserId = user.uid;
                        currentUserIdDisplay.textContent = currentUserId;
                        console.log("Firebase authenticated. User ID:", currentUserId);
                        setupFirestoreListener(); // Start listening to Firestore data
                    } else {
                        currentUserId = null;
                        currentUserIdDisplay.textContent = 'Not authenticated';
                        console.log("Firebase: No user signed in. Attempting anonymous sign-in.");
                        try {
                            await signInAnonymously(auth); // Sign in anonymously
                            console.log("Signed in anonymously.");
                        } catch (error) {
                            console.error("Firebase Auth Error during anonymous sign-in:", error);
                            if (error.code === 'auth/configuration-not-found') {
                                showMessage("Firebase Auth Error: Authentication is not enabled for your project. Please enable 'Anonymous' sign-in in your Firebase Console (Build -> Authentication -> Get started -> Sign-in method).", "error");
                            } else {
                                showMessage("Failed to authenticate with Firebase. Data won't be saved or synchronized. Check console for details.", "error");
                            }
                        }
                    }
                    updateAdminUI(); // Update visibility based on auth state and logged-in member
                }, (error) => {
                    console.error("Error in onAuthStateChanged:", error);
                    showMessage("Firebase authentication state error. Check console.", "error");
                });
            } catch (e) {
                console.error("Error initializing Firebase:", e);
                showMessage("Error initializing Firebase. Data persistence will not work. Check console for details.", "error");
            }
        });

        // Firestore Real-time Listener Function
        function setupFirestoreListener() {
            if (!db) {
                console.warn("Firestore instance is not available. Cannot set up listener.");
                return;
            }
            if (!currentUserId) {
                console.warn("User ID is not available. Cannot set up listener.");
                return;
            }

            console.log(`Setting up Firestore listener for collection: artifacts/${appId}/public/data/extendedHours`);
            // Query for all entries, sorted by creation date
            const q = query(collection(db, `artifacts/${appId}/public/data/extendedHours`), orderBy("createdAt", "asc"));

            onSnapshot(q, (snapshot) => {
                console.log("Firestore data received. Processing snapshot...");
                const fetchedEntries = [];
                snapshot.forEach((doc) => {
                    const data = doc.data();
                    fetchedEntries.push({
                        id: doc.id, // Firestore document ID is crucial for deletion
                        employeeId: data.employeeId, // Include employee ID
                        memberName: data.memberName,
                        date: data.date, // Stored asYYYY-MM-DD string
                        startTime: data.startTime,
                        endTime: data.endTime,
                        totalHours: parseFloat(data.totalHours), // Ensure it's a number
                        createdAt: data.createdAt // Keep for sorting consistency
                    });
                });
                // Sort fetched entries for consistent display
                fetchedEntries.sort((a, b) => {
                    // Primary sort by date
                    const dateA = new Date(a.date);
                    const dateB = new Date(b.date);
                    if (dateA.getTime() !== dateB.getTime()) {
                        return dateA - dateB;
                    }
                    // Secondary sort by start time if dates are the same
                    const timeA = a.startTime.split(':').map(Number);
                    const timeB = b.startTime.split(':').map(Number);
                    if (timeA[0] !== timeB[0]) { // Compare hours
                        return timeA[0] - timeB[0];
                    }
                    return timeA[1] - timeB[1]; // Compare minutes
                });

                extendedHoursEntries = fetchedEntries; // Update the global array with Firestore data
                console.log("Extended hours entries updated from Firestore:", extendedHoursEntries);
                applyFilter(); // Re-render table and summary with fetched data (and current filter applied)
            }, (error) => {
                console.error("Error listening to Firestore:", error);
                showMessage("Error loading real-time data from database. Check console for details.", "error");
            });
        }
    </script>
</body>
</html>
