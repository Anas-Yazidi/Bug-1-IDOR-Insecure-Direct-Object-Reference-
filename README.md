Bug #1 : IDOR (Insecure Direct Object Reference)

🔎 1 Day → 1 Vulnerability → 1 Lesson Learned

If you're also a beginner in Bug Bounty Hunting, follow this series to learn the fundamentals of web security, understand common vulnerabilities, and build your knowledge step by step.

This is just the beginning. 🚀
---
<p align="center">
  <img src="blob:https://chatgpt.com/4d83d3bb-f6fb-4d8d-aca3-5dc6844a589c="IDOR example" width="700">
</p>
## 🔎 What's an IDOR?

**IDOR (Insecure Direct Object Reference)** is an access control vulnerability that occurs when an application allows users to access another user's resources without properly verifying their permissions.

The application may use identifiers such as IDs, filenames, account numbers, or references to retrieve specific resources. If the backend fails to check whether the requester is authorized to access the requested resource, sensitive information or actions may become accessible to unauthorized users.

### 👥 Example

Let's imagine we have two users:

- **User A** → Profile ID: `102`
- **User B** → Profile ID: `103`

User A is authenticated and can access their own profile.

However, if the application relies on the requested ID without checking ownership or permissions, User A may be able to access User B's profile.

The problem isn't the ID itself.

⚠️ **The problem is the missing authorization check on the backend.**

### ❓ How can the ID be changed, and what kind of IDs are involved?

IDs and references can appear in different places within an application, such as:

- URL parameters → `?id=102`
- URL paths → `/users/102`
- API endpoints → `/api/users/102`
- JSON request bodies
- Hidden form fields
- Resource references such as order IDs, document IDs, or message IDs

An application may use numeric IDs, UUIDs, strings, or other identifiers.

Changing an identifier doesn't automatically mean you've found a vulnerability. The security issue exists only if the server allows access to a resource that the requester isn't authorized to access.

---

## ℹ️ New Concepts

### ☑️ Authorization 🛡️

Authorization determines what a user is allowed to access or perform.

**The question it answers:**

> What are you allowed to access?

For example, even if User A is logged in, they shouldn't automatically have permission to access User B's private messages or personal information.

---

### ☑️ Authentication 🔐

Authentication is the process of verifying a user's identity.

**The question it answers:**

> Who are you?

For example, when you log into Facebook, the application verifies your identity through your credentials and establishes an authenticated session.

Authentication and authorization are different:

- Authentication → verifies your identity.
- Authorization → verifies your permissions.

---

### ☑️ GET Requests 📩

`GET` is an HTTP method used to retrieve data from a server.

For example:

`GET /profile?id=102`

This request asks the server to retrieve the profile associated with ID `102`.

The server must still verify whether the requester is authorized to access that profile.

---

### ☑️ API (Application Programming Interface) 🔗

An API allows different applications or components to communicate and exchange data.

Imagine Instagram:

- Frontend → The interface you interact with.
- API → Receives requests and communicates with the backend.
- Backend → Processes the request and applies business logic.
- Database → Stores user information, posts, messages, and more.

For example, when you open a profile, the frontend may send a request to an API to retrieve the profile's data.

An API is not an authentication mechanism. Authentication systems such as Google Login may be integrated with applications, but their purpose is different from the API itself.

---

### ☑️ HTTP Requests & Responses 🌐

HTTP is a protocol used for communication between clients and servers.

#### How it works:

**1. Browser 🌐**

You open your profile.

**2. HTTP Request 📩**

The browser sends a request to the server asking for data.

**3. Server ⚙️**

The server receives the request, processes it, and checks whether the user is authorized.

**4. Database 🗄️**

The backend retrieves the requested information from the database if the operation is permitted.

**5. HTTP Response 📤**

The server returns a response to the browser, such as the requested data or an error.

- `200 OK` → The request succeeded.
- `401 Unauthorized` → Authentication is missing or invalid.
- `403 Forbidden` → The user is not permitted to perform the operation.
- `404 Not Found` → The resource was not found or is intentionally hidden.

---

### ☑️ Backend Authorization Check ⚙️

The backend is the server-side part of an application.

It handles operations such as:

- Authentication
- Database operations
- User permissions
- API requests
- Business logic

A backend authorization check verifies whether the current authenticated user has permission to access or manipulate a specific resource.

The frontend alone cannot guarantee security because users can modify requests outside the normal interface. That's why access control must be enforced on the server side.

---

### ☑️ IDs & References 🆔

An ID (identifier) is used to distinguish one resource from another.

Examples:

- User → `102`
- Order → `ORD-5001`
- Document → `file_83921`
- Message → `msg_721`

A reference is a value used by an application to point to a particular resource.

These identifiers are commonly used by APIs and databases to locate specific objects.

However:

**Having an ID doesn't mean having permission to access the resource it identifies.**

---

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

## 🧠 What Did We Learn?

- Authentication doesn't mean you have access to everything.
- Authorization must be enforced by the backend.
- IDs are references, not security mechanisms.
- APIs must verify permissions for every protected resource.
- A resource being accessible through a request doesn't mean it should be accessible to everyone.

**Lesson learned:** Always verify authorization for the specific resource being requested.

---

## 🎯 Let's Play a Quiz!

### Question 1

What is the main difference between Authentication and Authorization?

A) Both verify passwords.  
B) Authentication verifies identity, while Authorization verifies permissions.  
C) Authorization is only performed by the frontend.  
D) They are exactly the same.

---

### Question 2

A user is logged in and changes an order ID from `5001` to `5002`.

Is this automatically an IDOR vulnerability?

A) Yes, every ID change is an IDOR.  
B) No, IDs should never be changed.  
C) Only if the server allows unauthorized access to the referenced resource.  
D) Yes, if the ID is numeric.

---

### Question 3

Where should an application's authorization checks be enforced?

A) Only in HTML.  
B) Only in JavaScript.  
C) Only in the browser.  
D) On the backend.

---

### Question 4

What is the main security problem in the vulnerable code?

A) It uses JavaScript.  
B) It uses `find()`.  
C) It doesn't check whether the requester is authorized to access the requested profile.  
D) The IDs are numbers.

---

### Question 5 🔥

User A is authenticated. User A requests User B's private document through an API. The document exists, but the backend doesn't verify ownership or permissions.

What type of security issue could this represent?

A) XSS  
B) IDOR / Broken Access Control  
C) Buffer Overflow  
D) DNS Spoofing

---

### 🏁 End of Bug #01

**Vulnerability:** IDOR — Insecure Direct Object Reference

**Category:** Broken Access Control

**Main cause:** Missing backend authorization checks

**Lesson:** Never trust a resource identifier as proof of permission.

➡️ Next: Bug #02 — Cross-Site Scripting (XSS)
