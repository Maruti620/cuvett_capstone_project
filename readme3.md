// let currentUser = {
//     name: "Prashant Kumar Jha",
//     email: "prashant@gmail.com",
//     password: "123",
//     userId: "1B378D"
// };

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



function route() {
    const hash = window.location.hash || "#"; // ''
    renderHeader();

    if(currentUser) {
        const page = hash.substring(1) || 'dashboard';

        switch (page) {
            case 'dashboard':
                // show dashboard
                break;
            
            case 'tasks':
                //showtask
                break;

            case 'notes':
                //show notes
                break;
        
            default:
                showSections("dashboard");
        }

    } else {
        if(hash === "#signup") {
            showSections("signup");
        } else {
            showSections("login");
        }
    }

}

// dashboard || notes || login || signup || tasks
function showSections(sectionName) {

    const availableSections = Object.values(sections);
    console.log(availableSections)

    // remove all the sections from UI
    availableSections.forEach((item) => {
        item.classList.remove('active');
    });

    // just show the current section passed in the argument
    if(sections[sectionName]) {
        sections[sectionName].classList.add('active');
    }
}

showSections();
route();


function chnageURL() {
    const hash = window.location.hash
    showSections(hash.substring(1))
    route();
}

