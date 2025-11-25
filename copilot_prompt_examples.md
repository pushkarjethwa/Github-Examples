# GitHub Copilot Prompt Engineering Examples

## 10 Bad vs. Better Prompts

### 1. Too vague
**Bad:**  
Write code for login.

**Better:**  
Write a Node.js Express route (`POST /login`) that validates email and password, checks credentials using a mock database array, and returns a JWT token if successful.

### 2. Missing context
**Bad:**  
Fix this function.

**Better:**  
Here is a Python function that is throwing a `TypeError`. Please analyze the function, identify the cause of the error, and rewrite it to correctly handle both strings and lists.

### 3. Overly broad
**Bad:**  
Build a React app.

**Better:**  
Create a minimal React component called `UserCard` that takes `name`, `email`, and `avatarUrl` as props and displays them inside a styled card layout.

### 4. No constraints
**Bad:**  
Optimize this code.

**Better:**  
Optimize the following JavaScript function for readability and performance, but keep it ES6-only and avoid introducing external libraries.

### 5. Unclear output format
**Bad:**  
Explain this code.

**Better:**  
Explain this code in bullet points at an intermediate developer level, and include a short summary of what the algorithm accomplishes.

### 6. Asking for everything at once
**Bad:**  
Create a full-stack app with authentication, database, and UI.

**Better:**  
(Using layered prompts)
1. Generate Express server boilerplate with routes folder setup.  
2. Add JWT authentication.  
3. Create a React login page that connects to `/login`.

### 7. Underspecified behavior
**Bad:**  
Write a sorting function.

**Better:**  
Write a Python function `sort_by_length` that sorts a list of strings by their string length in ascending order. If two strings have the same length, sort them alphabetically.

### 8. Missing edge cases
**Bad:**  
Create a CSV parser.

**Better:**  
Write a TypeScript CSV parsing function that:
- handles quoted fields,  
- preserves commas inside quotes,  
- trims whitespace,  
- returns an array of objects based on header row.

### 9. No project context
**Bad:**  
Suggest database schema.

**Better:**  
Suggest a PostgreSQL schema for a task management app with entities: `User`, `Task`, `Comment`, including relational keys and indexes.

### 10. Too dependent on AI creativity
**Bad:**  
Make this API more secure.

**Better:**  
Review this Express middleware for security issues and update it to sanitize inputs, enforce rate limiting using `express-rate-limit`, and validate JWT tokens.

---

## Improved Version of Your Original Question

Using the attached slide on prompt engineering foundations and best practices, generate 10 examples of *bad vs. improved* GitHub Copilot prompts. Ensure the examples reflect clarity, specificity, layered prompts, and contextualization. Also suggest how I could phrase future questions to get more effective AI responses.
