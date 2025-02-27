# **OpenSSH Key Management on Windows**
This guide will walk you through generating, managing, and using SSH keys with OpenSSH on Windows.

---

## **1. Install OpenSSH on Windows**
Windows 10 and 11 come with OpenSSH pre-installed. You can check if it's installed and install it manually if needed.

### **Check if OpenSSH is Installed**
1. Open **PowerShell** as Administrator.
2. Run the command:
   ```powershell
   Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
   ```
3. If OpenSSH is installed, you will see:
   - `OpenSSH.Client~~~~0.0.1.0` (for SSH client)
   - `OpenSSH.Server~~~~0.0.1.0` (for SSH server)

### **Install OpenSSH (If Not Installed)**
If OpenSSH is missing, install it using the following:
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```
For SSH Server (if needed):
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
Enable the SSH server (optional, if running a local SSH server):
```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

---

## **2. Generate SSH Key Pair on Windows**
1. Open **PowerShell** or **Command Prompt**.
2. Run:
   ```powershell
   ssh-keygen -t rsa -b 4096 -f C:\Users\YourUsername\.ssh\id_rsa
   ```
   - Replace `YourUsername` with your actual Windows username.
   - You can change the filename (`id_rsa`) if you want multiple keys.
3. You will be prompted to enter a **passphrase** (optional but recommended for security).

---

## **3. Locate and Manage SSH Keys**
Your SSH keys are stored in:
```powershell
C:\Users\YourUsername\.ssh\
```
The generated files:
- **id_rsa** → Private key (keep this secure, NEVER share it).
- **id_rsa.pub** → Public key (can be shared for authentication).

---

## **4. Add Your SSH Key to SSH-Agent**
To automatically load the SSH key when using SSH:
1. Start the SSH-Agent service:
   ```powershell
   Start-Service ssh-agent
   Set-Service -Name ssh-agent -StartupType Automatic
   ```
2. Add the private key to the agent:
   ```powershell
   ssh-add C:\Users\YourUsername\.ssh\id_rsa
   ```
   - This allows SSH to use the key without manually entering the passphrase every time.

---

## **5. Copy Public Key to Remote Server**
To copy your SSH public key to a remote server (e.g., GitHub, GitLab, or another machine):

### **GitHub Example**
1. Open **PowerShell** and display the public key:
   ```powershell
   Get-Content C:\Users\YourUsername\.ssh\id_rsa.pub
   ```
2. Copy the output.
3. Go to **GitHub → Settings → SSH and GPG Keys → New SSH Key**.
4. Paste the key and save.

---

## **6. Test SSH Connection**
To verify SSH authentication:
```powershell
ssh -T git@github.com
```
If successful, you will see:
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

To connect to a remote SSH server:
```powershell
ssh user@remote-server-ip
```

---

## **7. Troubleshooting SSH Issues**
### **Check SSH Service Status**
If SSH is not working, check if the service is running:
```powershell
Get-Service ssh-agent
```
If stopped, start it:
```powershell
Start-Service ssh-agent
```

### **Enable Debug Mode**
Run SSH with verbose mode to troubleshoot issues:
```powershell
ssh -vvv user@remote-server-ip
```

### **Check Key Permissions**
Ensure the SSH key has the correct permissions:
```powershell
icacls C:\Users\YourUsername\.ssh\id_rsa /inheritance:r /grant:r "%USERNAME%:F"
```

---

## **8. Revoking and Deleting SSH Keys**
If you need to remove an SSH key:
1. **Delete the Key Files**
   ```powershell
   Remove-Item C:\Users\YourUsername\.ssh\id_rsa*
   ```
2. **Remove Key from SSH Agent**
   ```powershell
   ssh-add -D
   ```
3. **Remove from GitHub/GitLab** (Go to **Settings** → **SSH Keys** and delete it manually).

---

## **Conclusion**
By following this guide, you have successfully set up and managed SSH keys using OpenSSH on Windows. You can now securely authenticate with remote systems, GitHub, and other services.
