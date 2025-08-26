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
    console.log("Dashboard Rendered");
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