### While connecting Ec2 from power shall
PS E:\Vamshi_Java\AWS\Ec2\product>ssh -i product.pem ec2-user@16.112.192.197
Load key "product.pem": Permission denied
ec2-user@16.112.192.197: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

### Why did this happen?

AWS requires that your private key file be completely private to you. If OpenSSH detects that anyone else logged into your local machine could potentially browse into that folder and read your `.pem` file, it refuses to use it as a security precaution.

The error `Load key "product.pem": Permission denied` means that there are **still too many accounts** (like the generic `Users` group or backup accounts) that retain read access to the file.

To bypass this permanently in PowerShell, we need to reset the file permissions entirely, grant explicit ownership to **only your current logged-in user account**, and wipe everything else out.

Run this script directly in your PowerShell window:

PowerShell

```
# 1. Store your current Windows username into a variable
$WinUser = "$env:USERNAME"

# 2. Reset the file's inherited permissions completely
icacls.exe .\product.pem /reset

# 3. Take explicit ownership of the file
icacls.exe .\product.pem /grant:r "${WinUser}:(R)" /inheritance:r

# 4. Remove any remaining systemic access just to be sure
icacls.exe .\product.pem /remove "NT AUTHORITY\Authenticated Users"
icacls.exe .\product.pem /remove "BUILTIN\Users"

```

### What these commands do:

-   `/grant:r "${WinUser}:(R)"` gives **only your account** pure Read `(R)` access.
    
-   `/inheritance:r` aggressively **removes all inherited permissions** from the parent folders (`E:\`, `Vamshi_Java\`, etc.), ensuring no other group sneaks into the access list.
    

Once you run those lines, try your `ssh` command again:

Bash

```
ssh -i product.pem ec2-user@16.112.192.197

```

### The "Fallback" Alternative (If Windows permissions keep fighting you)

If Windows Access Control Lists (ACLs) continue to cause issues, you can bypass the native Windows filesystem permission check entirely by running the SSH command from **WSL (Windows Subsystem for Linux)** or **Git Bash**.

If you open **Git Bash** in that folder, you can use the standard Linux command that fixes this instantly:

Bash

```
# Run this inside Git Bash:
chmod 400 product.pem
ssh -i product.pem ec2-user@16.112.192.197

```
