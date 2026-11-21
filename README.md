## 💻 The Code

### ❌ Vulnerable Version

The problem with this implementation is that it retrieves the requested user without checking whether the authenticated user is authorized to access that profile.

```javascript
function getProfile(loggedInUserId, requestedUserId) {

    const user = users.find(u => u.id === requestedUserId);

    if (!user) {
        return "User not found";
    }

    return user;
}
```

### ✅ The Correct Version

We introduce an authorization check before returning the requested profile.

```javascript
function getProfile(loggedInUserId, requestedUserId) {

    if (loggedInUserId !== requestedUserId) {
        return "Access denied";
    }

    const user = users.find(u => u.id === requestedUserId);

    if (!user) {
        return "User not found";
    }

    return user;
}
```

*Note: This is a simplified example where users can only access their own profiles. In a real application, the authenticated identity must come from a trusted session or token, and authorization rules may also permit access to public profiles, administrators, or shared resources.*

---

## 🐛 The Bug

The vulnerable version checks whether the requested user exists, but it doesn't check whether the requester has permission to access that user's profile.

### ❌ Vulnerable logic

`Does the user exist? → Return the profile.`

### ✅ Secure logic

`Is the requester authorized? → If yes, return the profile.`

The main problem is the absence of an authorization check between the requested resource and the authenticated user.

This can result in unauthorized access to private information or even unauthorized modification or deletion of resources, depending on the application's functionality.

---
