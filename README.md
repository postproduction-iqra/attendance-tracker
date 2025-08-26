# attendance-tracker
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minimal Employee Attendance Tracker</title>
    <!-- Tailwind CSS for modern styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        /* Custom font and base styling */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        /* Button styling with gradients and shadows */
        .btn {
            @apply px-4 py-3 font-bold rounded-full text-white shadow-lg transition-all duration-300 transform hover:scale-105;
        }
        .btn-green {
            background-image: linear-gradient(to right, #10b981, #059669);
            box-shadow: 0 4px 6px -1px rgba(16, 185, 129, 0.3), 0 2px 4px -1px rgba(16, 185, 129, 0.1);
        }
        .btn-green:hover {
            background-image: linear-gradient(to right, #059669, #047857);
            box-shadow: 0 10px 15px -3px rgba(16, 185, 129, 0.4), 0 4px 6px -2px rgba(16, 185, 129, 0.2);
        }
        .btn-yellow {
            background-image: linear-gradient(to right, #f59e0b, #d97706);
            box-shadow: 0 4px 6px -1px rgba(245, 158, 11, 0.3), 0 2px 4px -1px rgba(245, 158, 11, 0.1);
        }
        .btn-yellow:hover {
            background-image: linear-gradient(to right, #d97706, #b45309);
            box-shadow: 0 10px 15px -3px rgba(245, 158, 11, 0.4), 0 4px 6px -2px rgba(245, 158, 11, 0.2);
        }
        .btn-blue {
            background-image: linear-gradient(to right, #38bdf8, #0ea5e9);
            box-shadow: 0 4px 6px -1px rgba(56, 189, 248, 0.3), 0 2px 4px -1px rgba(56, 189, 248, 0.1);
        }
        .btn-blue:hover {
            background-image: linear-gradient(to right, #0ea5e9, #0284c7);
            box-shadow: 0 10px 15px -3px rgba(56, 189, 248, 0.4), 0 4px 6px -2px rgba(56, 189, 248, 0.2);
        }
        .btn-red {
            background-image: linear-gradient(to right, #f43f5e, #e11d48);
            box-shadow: 0 4px 6px -1px rgba(244, 63, 94, 0.3), 0 2px 4px -1px rgba(244, 63, 94, 0.1);
        }
        .btn-red:hover {
            background-image: linear-gradient(to right, #e11d48, #be123c);
            box-shadow: 0 10px 15px -3px rgba(244, 63, 94, 0.4), 0 4px 6px -2px rgba(244, 63, 94, 0.2);
        }
        .btn-purple { @apply bg-purple-600 hover:bg-purple-700; }
        .btn-disabled { @apply bg-gray-400 cursor-not-allowed; }
        
        /* New click animation for all buttons */
        .btn:active:not([disabled]) {
            transform: scale(0.95);
        }

        /* Table styling for better readability */
        th, td { @apply p-4 text-left; }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-4">
    <div class="bg-white p-8 rounded-2xl shadow-2xl w-full max-w-4xl transition-all duration-500 ease-in-out">
        <h1 class="text-4xl font-extrabold text-center text-gray-900 mb-2">Attendance Dashboard</h1>
        <p id="userIdDisplay" class="text-sm text-center text-gray-500 mb-6 hidden"></p>

        <!-- Employee and Month Selector (simplified for August) -->
        <div class="flex flex-col md:flex-row items-center justify-between mb-6 gap-4">
            <div class="relative inline-flex items-center gap-2">
                <i data-lucide="user" class="h-6 w-6 text-gray-500"></i>
                <select id="employeeSelect" class="px-4 py-2 rounded-full border border-gray-300 bg-white font-medium text-lg focus:outline-none focus:ring-2 focus:ring-sky-500">
                    <!-- Employee options will be added by JavaScript -->
                </select>
            </div>
            <span id="monthDisplay" class="text-xl font-bold">August 2024</span>
        </div>

        <!-- Action Buttons for Current Day -->
        <div class="grid grid-cols-2 sm:grid-cols-4 gap-4 mb-8">
            <button id="clockInBtn" class="btn btn-green">Clock In</button>
            <button id="lunchOutBtn" class="btn btn-yellow" disabled>Lunch Out</button>
            <button id="lunchInBtn" class="btn btn-blue" disabled>Lunch In</button>
            <button id="clockOutBtn" class="btn btn-red" disabled>Clock Out</button>
        </div>
        
        <!-- Monthly Attendance Table and Export Button -->
        <div class="flex justify-end mb-4">
            <button id="exportCsvBtn" class="btn btn-purple flex items-center gap-2" disabled>
                <i data-lucide="download" class="h-5 w-5"></i>
                Export to CSV
            </button>
        </div>

        <div class="overflow-x-auto rounded-xl shadow-lg">
            <table class="w-full text-left text-gray-700">
                <thead class="bg-gray-200">
                    <tr>
                        <th class="p-4 font-bold">Date</th>
                        <th class="p-4 font-bold">Clock In</th>
                        <th class="p-4 font-bold">Lunch Out</th>
                        <th class="p-4 font-bold">Lunch In</th>
                        <th class="p-4 font-bold">Clock Out</th>
                        <th class="p-4 font-bold">Total Hours</th>
                    </tr>
                </thead>
                <tbody id="attendanceTableBody">
                    <!-- Rows will be dynamically added here -->
                </tbody>
                <tfoot>
                    <tr class="bg-gray-200 font-bold">
                        <td colspan="5" class="p-4 text-right">Monthly Total:</td>
                        <td id="monthlyTotal" class="p-4 text-sky-700">0.00</td>
                    </tr>
                </tfoot>
            </table>
        </div>
    </div>

    <script type="module">
        // Import Firebase libraries directly within the module scope
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for Firebase and authentication
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        let db, auth, userId = null;
        let isAuthReady = false; // New flag to track authentication state
        let currentEmployee;
        // Hardcode the date to a fixed month
        const currentDate = new Date('2024-08-01');
        let attendanceData = {};
        const employees = ['Rishi', 'Ashil', 'Dipesh', 'Soumen', 'Badal', 'Nawaid', 'Shahidur', 'Shahin'];
        
        // DOM element references
        const employeeSelect = document.getElementById('employeeSelect');
        const monthDisplay = document.getElementById('monthDisplay');
        const clockInBtn = document.getElementById('clockInBtn');
        const lunchOutBtn = document.getElementById('lunchOutBtn');
        const lunchInBtn = document.getElementById('lunchInBtn');
        const clockOutBtn = document.getElementById('clockOutBtn');
        const exportCsvBtn = document.getElementById('exportCsvBtn');
        const attendanceTableBody = document.getElementById('attendanceTableBody');
        const monthlyTotalDisplay = document.getElementById('monthlyTotal');
        const userIdDisplay = document.getElementById('userIdDisplay');

        // --- Utility Functions ---
        const formatTime = (date) => {
            if (!date) return '';
            const d = date instanceof Date ? date : new Date(date.toDate());
            const hours = d.getHours();
            const minutes = d.getMinutes();
            const ampm = hours >= 12 ? 'PM' : 'AM';
            const formattedHours = hours % 12 === 0 ? 12 : hours % 12;
            const formattedMinutes = minutes < 10 ? '0' + minutes : minutes;
            return `${formattedHours}:${formattedMinutes} ${ampm}`;
        };

        const calculateHours = (start, end) => {
            if (!start || !end) return 0;
            const startMs = start instanceof Date ? start.getTime() : start.toDate().getTime();
            const endMs = end instanceof Date ? end.getTime() : end.toDate().getTime();
            const diffInMinutes = (endMs - startMs) / (1000 * 60);
            return diffInMinutes / 60;
        };

        const getDayKey = (date) => date.toISOString().slice(0, 10);
        const getMonthKey = (date) => `${date.getFullYear()}-${date.getMonth() + 1}`;

        // --- Firebase Initialization and Data Handling ---

        const setupFirebaseAndAuth = async () => {
            try {
                const app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);

                // Use anonymous sign-in directly for shared access
                await signInAnonymously(auth);

                // Wait for the auth state to be established before proceeding.
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = `User ID: ${userId}`;
                        userIdDisplay.classList.remove('hidden');
                    }
                    isAuthReady = true;
                    setupApp();
                });
            } catch (error) {
                console.error('Firebase initialization or authentication error:', error);
            }
        };

        const setupApp = () => {
            // Clear the select element before populating it
            employeeSelect.innerHTML = '';
            
            // Populate employee select dropdown
            employees.forEach(name => {
                const option = document.createElement('option');
                option.value = name;
                option.textContent = name;
                employeeSelect.appendChild(option);
            });
            
            // Initialize state
            currentEmployee = employees[0];
            updateDisplay();
            listenForData();
            
            // Add event listeners
            employeeSelect.addEventListener('change', (e) => {
                currentEmployee = e.target.value;
                attendanceData[currentEmployee] = {};
                listenForData();
                updateDisplay();
            });
            
            clockInBtn.addEventListener('click', () => handleClockAction('clockIn'));
            lunchOutBtn.addEventListener('click', () => handleClockAction('lunchOut'));
            lunchInBtn.addEventListener('click', () => handleClockAction('lunchIn'));
            clockOutBtn.addEventListener('click', () => handleClockAction('clockOut'));
            exportCsvBtn.addEventListener('click', exportToCsv);
            
            // Initialize Lucide icons
            lucide.createIcons();
        };

        const listenForData = () => {
            if (!db || !userId) return;
            // The path is now a PUBLIC path so everyone can access and edit the same data
            const docRef = doc(db, `/artifacts/${appId}/public/data/employees/${currentEmployee}`);

            onSnapshot(docRef, (docSnap) => {
                if (docSnap.exists()) {
                    attendanceData[currentEmployee] = docSnap.data();
                } else {
                    attendanceData[currentEmployee] = {};
                }
                updateTable();
                setButtonStates();
            }, (error) => {
                console.error("Error fetching document: ", error);
            });
        };
        
        const handleClockAction = async (action) => {
            if (!db || !userId) return;

            const now = new Date();
            const employeeData = attendanceData[currentEmployee] || {};
            const monthData = employeeData[getMonthKey(currentDate)] || {};
            const dayData = monthData[getDayKey(now)] || {};
            
            dayData[action] = now;
            
            if (dayData.clockIn && dayData.clockOut && dayData.lunchIn && dayData.lunchOut) {
                const workTime = calculateHours(dayData.clockIn, dayData.clockOut);
                const lunchTime = calculateHours(dayData.lunchOut, dayData.lunchIn);
                dayData.totalHours = workTime - lunchTime;
            } else {
                dayData.totalHours = 0;
            }

            const dataToSave = {
                ...employeeData,
                [getMonthKey(currentDate)]: {
                    ...monthData,
                    [getDayKey(now)]: dayData
                }
            };

            try {
                // Use the public path for writing shared data
                const docRef = doc(db, `/artifacts/${appId}/public/data/employees/${currentEmployee}`);
                await setDoc(docRef, dataToSave, { merge: true });
            } catch (error) {
                console.error("Error writing document to Firestore: ", error);
            }
        };
        
        // --- UI Update Functions ---
        
        const updateDisplay = () => {
            monthDisplay.textContent = currentDate.toLocaleString('default', { month: 'long', year: 'numeric' });
        };

        const updateTable = () => {
            attendanceTableBody.innerHTML = '';
            const monthData = attendanceData[currentEmployee]?.[getMonthKey(currentDate)] || {};
            const monthDays = Object.keys(monthData).sort();

            if (monthDays.length > 0) {
                monthDays.forEach(dayKey => {
                    const dayData = monthData[dayKey];
                    const row = document.createElement('tr');
                    row.className = 'border-b border-gray-200 bg-white hover:bg-gray-50';
                    row.innerHTML = `
                        <td class="p-4">${dayKey}</td>
                        <td class="p-4">${dayData.clockIn ? formatTime(dayData.clockIn) : '-'}</td>
                        <td class="p-4">${dayData.lunchOut ? formatTime(dayData.lunchOut) : '-'}</td>
                        <td class="p-4">${dayData.lunchIn ? formatTime(dayData.lunchIn) : '-'}</td>
                        <td class="p-4">${dayData.clockOut ? formatTime(dayData.clockOut) : '-'}</td>
                        <td class="p-4 font-semibold text-sky-700">${dayData.totalHours ? dayData.totalHours.toFixed(2) : '0.00'}</td>
                    `;
                    attendanceTableBody.appendChild(row);
                });
            } else {
                const row = document.createElement('tr');
                row.className = 'bg-white';
                row.innerHTML = `<td colspan="6" class="p-4 text-center text-gray-500 italic">No attendance records for this month.</td>`;
                attendanceTableBody.appendChild(row);
            }
            
            updateMonthlyTotal();
        };

        const updateMonthlyTotal = () => {
            const monthData = attendanceData[currentEmployee]?.[getMonthKey(currentDate)];
            if (!monthData) {
                monthlyTotalDisplay.textContent = '0.00';
                return;
            }
            const total = Object.values(monthData).reduce((sum, day) => sum + (day.totalHours || 0), 0);
            monthlyTotalDisplay.textContent = total.toFixed(2);
        };

        const setButtonStates = () => {
            const today = getDayKey(new Date());
            const attendance = attendanceData[currentEmployee]?.[getMonthKey(currentDate)]?.[today] || {};
            const isClockedIn = !!attendance.clockIn;
            const isLunchOut = !!attendance.lunchOut;
            const isLunchIn = !!attendance.lunchIn;
            const isClockedOut = !!attendance.clockOut;
            const hasDataToExport = Object.keys(attendanceData[currentEmployee]?.[getMonthKey(currentDate)] || {}).length > 0;

            clockInBtn.disabled = isClockedIn || isClockedOut;
            lunchOutBtn.disabled = !isClockedIn || isLunchOut || isClockedOut;
            lunchInBtn.disabled = !isLunchOut || isLunchIn || isClockedOut;
            clockOutBtn.disabled = !isClockedIn || isClockedOut || !isLunchIn;
            
            exportCsvBtn.disabled = !isAuthReady || !hasDataToExport;
            
            const buttons = [clockInBtn, lunchOutBtn, lunchInBtn, clockOutBtn, exportCsvBtn];
            buttons.forEach(btn => {
                if (btn.disabled) {
                    btn.classList.add('btn-disabled');
                } else {
                    btn.classList.remove('btn-disabled');
                }
            });
        };

        const exportToCsv = () => {
            const monthData = attendanceData[currentEmployee]?.[getMonthKey(currentDate)];
            if (!monthData || Object.keys(monthData).length === 0) {
                console.log('No data to export for this month.');
                return;
            }

            let csvContent = "Date,Clock In,Lunch Out,Lunch In,Clock Out,Total Hours\n";
            const sortedDays = Object.keys(monthData).sort();

            sortedDays.forEach(dayKey => {
                const dayData = monthData[dayKey];
                const clockIn = dayData.clockIn ? formatTime(dayData.clockIn) : '';
                const lunchOut = dayData.lunchOut ? formatTime(dayData.lunchOut) : '';
                const lunchIn = dayData.lunchIn ? formatTime(dayData.lunchIn) : '';
                const clockOut = dayData.clockOut ? formatTime(dayData.clockOut) : '';
                const totalHours = dayData.totalHours ? dayData.totalHours.toFixed(2) : '0.00';
                
                csvContent += `"\t${dayKey}","${clockIn}","${lunchOut}","${lunchIn}","${clockOut}",${totalHours}\n`;
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.setAttribute('href', url);
            const filename = `${currentEmployee}_Attendance_${currentDate.toLocaleString('default', { month: 'long', year: 'numeric' }).replace(/\s/g, '_')}.csv`;
            link.setAttribute('download', filename);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            URL.revokeObjectURL(url);
        };
        
        document.addEventListener('DOMContentLoaded', () => {
            setupFirebaseAndAuth();
        });
    </script>
</body>
</html>
