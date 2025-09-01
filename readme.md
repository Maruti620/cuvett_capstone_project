# Changes Made

```javascript
function initializeApplication() {

    const storedUser = localStorage.getItem("user");
    currentUser = storedUser ? JSON.parse(storedUser) : null;
    route();

    handleSignup();  // chnage this line call this function
    handleLogin(); //  // chnage this line call this function
}
```
