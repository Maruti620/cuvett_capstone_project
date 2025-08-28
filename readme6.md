let currentUser = null;

const sections = {
    login: document.getElementById("login-section"),
    signup: document.getElementById("signup-section"),
    dashboard: document.getElementById("dashboard-section"),
    tasks: document.getElementById("tasks-section"),
    notes: document.getElementById("notes-section"),
};

// header
function renderHeader() {
    const userInfo = document.getElementById("user-info");
    const logoutButton = document.getElementById("logout-btn");
    const desktopNavLinks = document.getElementById("desktop-nav");

    if (currentUser) {
        userInfo.textContent = currentUser.email;
        logoutButton.style.display = "inline";
        desktopNavLinks.querySelector("#nav-dashboard").style.display = "inline";
        desktopNavLinks.querySelector("#nav-tasks").style.display = "inline";
        desktopNavLinks.querySelector("#nav-notes").style.display = "inline";

    }
    else {
        userInfo.textContent = "";
        logoutButton.style.display = "none";
        desktopNavLinks.querySelector("#nav-tasks").style.display = "none";
        desktopNavLinks.querySelector("#nav-notes").style.display = "none";
        desktopNavLinks.querySelector("#nav-dashboard").style.display = "none";
    }
}

document.getElementById("logout-btn").addEventListener('click', () => {
    localStorage.removeItem("user");
    currentUser = null;
    window.location.hash = "#login";
    route();
});

function route() {
    const hash = window.location.hash || "#";
    renderHeader();

    if (currentUser) {
        const page = hash.substring(1) || 'dashboard';
        showSections(page);
        switch (page) {
            case 'dashboard':
                renderDashboard();
                break;

            case 'tasks':
                renderTasks();
                break;

            case 'notes':
                renderNotes();
                break;

            default:
                showSections("dashboard");
        }

    } else {
        if (hash === "#signup") {
            showSections("signup");
        } else {
            showSections("login");
        }
    }

}

// dashboard || notes || login || signup || tasks
function showSections(sectionName) {

    const availableSections = Object.values(sections);
    // console.log(availableSections)

    // remove all the sections from UI
    availableSections.forEach((item) => {
        item.classList.remove('active');
    });

    // just show the current section passed in the argument
    if (sections[sectionName]) {
        sections[sectionName].classList.add('active');
    }
}

function renderDashboard() {
    sections.dashboard.innerHTML = `
        <h2>Welcome to your Dashboard</h2> 
        ${currentUser.userName} ! 
        <p>Choose Either Tasks or Notes to Continue</p>
    `;
    document.getElementById("user-info").textContent = currentUser.userEmail;
}

function renderNotes() {
    console.log("Notes Rendered");
}

function renderTasks() {
    console.log("Tasks Rendered");
}

function handleSignup() {
    const signUpForm = document.getElementById("signup-form");

    signUpForm.addEventListener("submit", async (e) => {
        e.preventDefault(); // prevent the default behaviour

        const userName = document.getElementById("signup-username").value;
        const userEmail = document.getElementById("signup-email").value;
        const userPassword = document.getElementById("signup-password").value;

        try {
            const res = await fetch(`http://localhost:3000/users?userEmail=${encodeURIComponent(userEmail)}`);
            const data = await res.json();

            if (data.length > 0) {
                throw new Error("User Already Exists");
            }

            const createdUser = await fetch(`http://localhost:3000/users`, {
                method: "POST",
                headers: {
                    'Content-Type': "application/json",
                },
                body: JSON.stringify({ userName, userEmail, userPassword })
            });

            const newUser = await createdUser.json();
            currentUser = newUser;

            localStorage.setItem("user", JSON.stringify(currentUser));

        } catch (error) {
            document.getElementById("signup-error").textContent = error.message;
        }

        // window.location.hash = "#dashboard";
    });

    window.location.hash = "#dashboard";
    route();
}

function handleLogin() {
    const logInForm = document.getElementById("login-form");

    logInForm.addEventListener("submit", async (e) => {
        e.preventDefault(); // prevent the default behaviour

        const userEmail = document.getElementById("login-email").value;
        const userPassword = document.getElementById("login-password").value;
        console.log(userEmail, userPassword)

        try {
            const res = await fetch(`http://localhost:3000/users?userEmail=${encodeURIComponent(userEmail)}`);
            const data = await res.json();

            if (data.length === 0) {
                throw new Error("Invalid Email and Password");
            }

            if (data.length > 0 && userPassword === data[0].userPassword) {
                currentUser = data[0];
                localStorage.setItem("user", JSON.stringify(currentUser));
                window.location.hash = "#dashboard";
            }
            else {
                throw new Error("Password is Not Correct");
            }

            // re-route the user

        } catch (error) {
            document.getElementById("login-error").textContent = error.message;
        }
    });
}

function initializeApplication() {

    const storedUser = localStorage.getItem("user");
    currentUser = storedUser ? JSON.parse(storedUser) : null;
    route();

    handleSignup();
    handleLogin();
}

initializeApplication();

window.addEventListener("hashchange", route);


// currentUserId === currentUser
function tasksCRUD(currentUserId) {

    const STORAGE_KEY = "tasks-" + currentUserId; // local storage key
    const BASE_URL = "http://localhost:3000/tasks"; // backend url

    // check in local storage
    function loadTasksFromLocalStorage() {
        const tasks = localStorage.getItem(STORAGE_KEY);
        return tasks ? JSON.parse(tasks) : [];
    }

    // save to local storage
    function saveTasksToLocalStorage(tasks) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
    }

    // create tasks
    async function createTask(task) {
        try {
            task.userId = currentUserId;
            task.completed = false;
            task.createdAt = new Date().toString();

            const response = await fetch(BASE_URL, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(task)
            });

            const savedTask = await response.json();

            const localStorageTasks = loadTasksFromLocalStorage();
            localStorageTasks.push(savedTask);

            saveTasksToLocalStorage(localStorageTasks);

        } catch (error) {
            console.log("Error While Creating Task: ", error.message);
            return null;
        }
    }

    // read task
    async function readTasks() {

        const localStorageTasks = loadTasksFromLocalStorage();
        if (localStorageTasks.length > 0) {
            return localStorageTasks;
        }

        try {
            const response = await fetch(`${BASE_URL}?userId=${currentUserId}`); // GET
            const tasks = await response.json();
            saveTasksToLocalStorage(tasks);
            return tasks;
        } catch (error) {
            console.log("Error While Reading Tasks: ", error.message);
            return [];
        }
    }

    // update task
    async function updateTask(task) {
        try {
            const response = await fetch(`${BASE_URL}/${task.id}`, {
                method: "PATCH",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(task)
            });

            const updatedTask = await response.json();
            const localStorageTasks = loadTasksFromLocalStorage();

            const taskIndex = localStorageTasks.findIndex(t => t.id === task.id);

            if (taskIndex !== -1) {
                localStorageTasks[taskIndex] = updatedTask;
                saveTasksToLocalStorage(localStorageTasks);
            }
        } catch (error) {
            console.log("Error While Updating Task: ", error.message);
        }
    }

    // delete task
    async function deleteTask(taskId) {
        try {
            const response = await fetch(`${BASE_URL}/${taskId}`, {
                method: "DELETE"
            });

            if (response.ok) {
                const localStorageTasks = loadTasksFromLocalStorage();
                const updatedTasks = localStorageTasks.filter(t => t.id !== taskId);
                saveTasksToLocalStorage(updatedTasks);
            } else {
                console.log("Error While Deleting Task: ", response.statusText);
            }
        } catch (error) {
            console.log("Error While Deleting Task: ", error.message);
        }
    }

    return {
        createTask,
        readTasks,
        updateTask,
        deleteTask
    };
}