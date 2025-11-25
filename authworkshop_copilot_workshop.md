# 🚀 GitHub Copilot Full Workflow Workshop  
### .NET 9 Console App – Authentication + In-Memory Database (with `AuthUser` model)

This workshop shows how to use **all three GitHub Copilot entry points** to build a complete .NET 9 console app:

- 🖥 **GH Copilot in the Terminal**
- 💬 **GitHub Copilot Chat (sidebar)**
- ✍️ **Inline Chat (Ctrl+I / ⌘I in the editor)**

To avoid name collisions with other frameworks or libraries, we’ll use **`AuthUser`** as our main model instead of `User`.

You’ll end up with:

- PBKDF2 password hashing  
- JWT token generation + validation  
- Lockout & failed login tracking  
- Role-based authorization  
- In-memory user database  
- Command-based console UI  

---

## 🧱 Section 0 — Target Architecture & Naming

**Namespaces & classes**

- Namespace: `AuthWorkshop`
- Models namespace: `AuthWorkshop.Models`
- Services namespace: `AuthWorkshop.Services`
- Main model: `AuthUser` (not `User`)

Example final model:

```csharp
namespace AuthWorkshop.Models;

public sealed class AuthUser
{
    public static readonly string DefaultRole = "User";

    public Guid Id { get; set; } = Guid.NewGuid();
    public string Username { get; set; } = string.Empty;
    public byte[] PasswordHash { get; set; } = Array.Empty<byte>();
    public byte[] Salt { get; set; } = Array.Empty<byte>();
    public string Role { get; set; } = DefaultRole;
    public int FailedAttempts { get; set; }
    public DateTimeOffset? LockedUntil { get; set; }
    public DateTimeOffset? LastLogin { get; set; }
}
```

We’ll have services like:

- `InMemoryAuthUserStore`
- `PasswordHasher`
- `JwtTokenGenerator`
- `AuthService`

---

# 🔧 Section 1 — GH Copilot Terminal Prompts

Use **GH Copilot in the terminal** to scaffold the project.

> All prompts are typed *after* `gh copilot suggest` (or the equivalent in your setup).

---

### 🧩 Prompt 1 — Create Solution + Project

```bash
gh copilot suggest "Create a .NET 9 console application solution named AuthWorkshop and put the project inside a folder named AuthWorkshop."
```

Accept a suggestion similar to:

```bash
dotnet new sln -n AuthWorkshop
dotnet new console -n AuthWorkshop
dotnet sln add AuthWorkshop/AuthWorkshop.csproj
```

---

### 🧩 Prompt 2 — Add Required NuGet Packages

```bash
gh copilot suggest "Add packages for System.IdentityModel.Tokens.Jwt, Microsoft.Extensions.Caching.Memory, and Microsoft.Extensions.Logging.Console to the AuthWorkshop project."
```

You should get commands like:

```bash
dotnet add AuthWorkshop package System.IdentityModel.Tokens.Jwt
dotnet add AuthWorkshop package Microsoft.Extensions.Caching.Memory
dotnet add AuthWorkshop package Microsoft.Extensions.Logging.Console
```

---

### 🧩 Prompt 3 — Create Folder Structure

```bash
gh copilot suggest "Inside the AuthWorkshop project, create Models and Services folders."
```

Expected commands:

```bash
mkdir AuthWorkshop/Models
mkdir AuthWorkshop/Services
```

---

# 💬 Section 2 — GitHub Copilot Chat Prompts (Sidebar)

Now switch to **Copilot Chat** in your editor to generate the model and services.

In each step, open the target file first (or create an empty file), then paste the prompt into Copilot Chat.

---

### 🧩 Prompt 4 — Generate `AuthUser` Model

Create file: `Models/AuthUser.cs`.

**Prompt:**

```text
Create an AuthUser class in the AuthWorkshop.Models namespace for a .NET 9 console app authentication system with:

- Guid Id
- string Username
- byte[] PasswordHash
- byte[] Salt
- string Role with default "User"
- int FailedAttempts
- DateTimeOffset? LockedUntil
- DateTimeOffset? LastLastLogin

Make the class sealed and all properties public. Also add a public static readonly string DefaultRole = "User".
```

Review the generated code and ensure the class name is **AuthUser** (not `User`).

---

### 🧩 Prompt 5 — Create `InMemoryAuthUserStore`

Create file: `Services/InMemoryAuthUserStore.cs`.

**Prompt:**

```text
Generate a class InMemoryAuthUserStore in the AuthWorkshop.Services namespace for .NET 9.

Requirements:
- Use Microsoft.Extensions.Caching.Memory.MemoryCache.
- Maintain a Dictionary<string, AuthUser> keyed by username.
- Provide async methods:
  - Task<AuthUser?> GetAsync(string username)
  - Task AddAsync(AuthUser user)
  - Task<IEnumerable<AuthUser>> GetAllAsync()
Use AuthWorkshop.Models.AuthUser as the model type.
```

Verify it uses `AuthUser` from `AuthWorkshop.Models`.

---

### 🧩 Prompt 6 — Create `PasswordHasher` (PBKDF2)

Create file: `Services/PasswordHasher.cs`.

**Prompt:**

```text
Create a static PasswordHasher class in AuthWorkshop.Services for .NET 9.

Requirements:
- Use PBKDF2 with SHA256, 100_000 iterations, and 32-byte output length.
- Methods:
  - (byte[] hash, byte[] salt) HashPassword(string password)
  - bool Verify(string password, byte[] hash, byte[] salt)
- Use RandomNumberGenerator.Create() for salt.
- Use CryptographicOperations.FixedTimeEquals for comparison.
```

---

### 🧩 Prompt 7 — Create `JwtTokenGenerator`

Create file: `Services/JwtTokenGenerator.cs`.

**Prompt:**

```text
Create a static JwtTokenGenerator class in AuthWorkshop.Services.

Requirements:
- Method string Generate(AuthUser user, string secretKey)
  - Use SymmetricSecurityKey and HMAC-SHA256.
  - Add claims: "username" and role (ClaimTypes.Role).
  - Token expires after 20 minutes.
- Method ClaimsPrincipal? Validate(string token, string secretKey)
  - Validate signature, lifetime, and key.
  - Do not validate issuer or audience.
- Use AuthWorkshop.Models.AuthUser for the user type.
```

---

### 🧩 Prompt 8 — Build `AuthService` (Core Logic)

Create file: `Services/AuthService.cs`.

**Prompt:**

```text
Create an AuthService class in AuthWorkshop.Services that coordinates authentication.

Requirements:
- Constructor: AuthService(InMemoryAuthUserStore store, ILogger<AuthService> logger, string jwtKey, PasswordHasher passwordHasher)
- Methods:
  - Task<bool> RegisterAsync(string username, string password, string? role = null)
    * Validate non-empty username and password.
    * Enforce a strong password policy (min 8 chars, number, special char).
    * Ensure username is unique.
    * Hash password with PasswordHasher.
    * Create an AuthUser with Username, PasswordHash, Salt, Role (default AuthUser.DefaultRole if null/whitespace).
    * Store via InMemoryAuthUserStore.
  - Task<(string? token, string? error)> LoginAsync(string username, string password)
    * Retrieve AuthUser.
    * Check LockedUntil; return ("LOCKED", "Account locked") if still locked.
    * Verify password via PasswordHasher.
    * On failure: increment FailedAttempts; lock for 15 minutes after 5 failures.
    * On success: reset FailedAttempts, set LastLogin, generate JWT via JwtTokenGenerator and return it.
- Use AuthWorkshop.Models.AuthUser everywhere, never User.
- Use ILogger<AuthService> to log key events (registration, login success, failure, lockout).
```

---

# ✍️ Section 3 — Inline Chat Prompts for `Program.cs`

Use Inline Chat inside `Program.cs`.

---

### 🧩 Prompt 9 — Create Main Program Skeleton

```text
Create the main entry point for this AuthWorkshop .NET 9 console app.

Requirements:
- Configure a console logger factory.
- Read a JWT secret key from environment variable JWT_KEY, or fallback to a hard-coded dev-only value.
- Create instances of InMemoryAuthUserStore, PasswordHasher, and AuthService.
- Seed a default admin AuthUser with username "admin" and role "Admin" by calling RegisterAsync.
- Then display a simple menu in a loop:
  - register
  - login
  - list-users
  - validate-token
  - quit
- Implement each command using AuthService and InMemoryAuthUserStore.
- Protect list-users so that only a valid token with Admin role can execute it.
- When validating a token, display the username and role from the claims if valid.
```

---

### 🧩 Prompt 10 — Enhance Menu UX

```text
Improve the command menu UX by:
- Showing available commands on each loop.
- Handling invalid input gracefully.
- Prompting username/password/role when needed.
- Showing lockout messages, failed attempts, and success messages.
Keep using AuthUser as the model type and AuthService as the API surface.
```

---

# 🧪 Section 4 — Test Flow

```bash
dotnet build
dotnet run --project AuthWorkshop
```

---

# 🎁 Section 5 — Optional Enhancements

### Unit tests  
```text
Generate xUnit tests for AuthService for:
- registration
- login
- lockout
- role handling
```

### Docker support  
```text
Create a Dockerfile for running this .NET 9 console app.
```

### Refactor menu  
```text
Refactor Program.cs into smaller classes for cleaner separation of concerns.
```
