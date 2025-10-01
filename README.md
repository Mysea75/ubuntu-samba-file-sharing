# 📘 File Sharing Between Ubuntu and Windows 11 Using Samba

This guide will help you (as a student) understand the **basic concept of Samba**, and then walk you through the **step-by-step setup** with commands, configuration, reset instructions, and references. By the end, you’ll be able to transfer files easily between your Ubuntu desktop and Windows 11 laptop on the same network.

---

## 🔹 1. Understanding the Basics

### What is Samba?

* **Samba** is a software package that allows Linux/Ubuntu systems to share files and printers with Windows systems using the SMB (Server Message Block) protocol.
* With Samba, your Ubuntu PC can act like a Windows file server, and Windows can see Ubuntu folders just like normal shared folders.

### Key Concepts

* **Share**: A folder that you expose from Ubuntu to Windows (e.g., `/home/ghulam/SharedFolder`).
* **Samba User**: Even if you already have a Linux user (like `ghulam`), Samba requires a separate password set using `smbpasswd`.
* **Protocol**: Windows 11 prefers **SMB2/SMB3** (SMB1 is deprecated for security reasons).

📸 *Reference screenshot (Windows Explorer mapping Samba share)*: ![Windows Explorer Samba Share](https://learn.microsoft.com/en-us/windows-server/storage/file-server/images/smb-share.png)

---

## 🔹 2. Ubuntu Configuration

### Step 1: Install Samba

```bash
sudo apt update
sudo apt install samba -y
```

### Step 2: Reset Samba to Default (if needed)

If Samba was previously configured and you want a clean reset:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
sudo cp /usr/share/samba/smb.conf /etc/samba/smb.conf
```

*(On some systems the default file is `/usr/share/samba/smb.conf.default`)*

Restart services:

```bash
sudo systemctl restart smbd nmbd
```

### Step 3: Edit Samba Config

```bash
sudo nano /etc/samba/smb.conf
```

Minimal working configuration:

```ini
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   server min protocol = SMB2
   server max protocol = SMB3
   map to guest = Never
   ntlm auth = ntlmv2-only

[UbuntuShare]
   path = /home/ghulam/SharedFolder
   browseable = yes
   read only = no
   guest ok = no
   valid users = ghulam
   force user = ghulam
   create mask = 0664
   directory mask = 0775
```

### Step 4: Create Shared Folder

```bash
mkdir -p /home/ghulam/SharedFolder
chown ghulam:ghulam /home/ghulam/SharedFolder
```

### Step 5: Create Samba User

```bash
sudo smbpasswd -a ghulam
sudo smbpasswd -e ghulam
```

### Step 6: Restart Samba

```bash
sudo systemctl restart smbd nmbd
```

### Step 7: Verify Config

```bash
testparm
```

📸 *Reference screenshot (testparm output)*: ![testparm](https://phoenixnap.com/kb/wp-content/uploads/2021/04/samba-testparm.png)

---

## 🔹 3. Windows 11 Configuration

### Step 1: Verify Connection

```powershell
ping 192.168.31.83
Test-NetConnection 192.168.31.83 -Port 445
```

### Step 2: Access Share

Open **File Explorer** → type:

```
\\192.168.31.83\UbuntuShare
```

Enter username: `ghulam` and Samba password.

Or from PowerShell:

```powershell
net use * /delete /y
net use \\192.168.31.83\UbuntuShare /user:ghulam <SambaPassword>
```

📸 *Reference screenshot (Windows run share path)*: ![Run Samba Share](https://www.linuxtechi.com/wp-content/uploads/2020/07/Access-Samba-Share-Windows10.png)

### Step 3: Map as Network Drive

1. Open **This PC** → **Map Network Drive**
2. Choose drive letter (e.g., Z:)
3. Enter `\\192.168.31.83\UbuntuShare`
4. Check **Reconnect at sign-in**
5. Provide Samba credentials

📸 *Reference screenshot (Map drive)*: ![Map Network Drive](https://www.ubuntubuzz.com/wp-content/uploads/2021/01/map-network-drive.png)

---

## 🔹 4. Accessing Windows Shares from Ubuntu (Reverse Sharing)

### Step 1: Share a Folder in Windows

1. Right-click folder → **Properties → Sharing → Advanced Sharing**.
2. Check **Share this folder** → name it (e.g., `WinShare`).
3. Set permissions for user `ghulam` (local Windows account).
4. Ensure NTFS Security tab also grants access.

### Step 2: Connect from Ubuntu

List shares:

```bash
smbclient -L //192.168.31.244 -U ghulam
```

Mount share:

```bash
sudo mkdir -p /mnt/winshare
sudo mount -t cifs //192.168.31.244/WinShare /mnt/winshare -o username=ghulam,password=YourPassword,vers=3.0
```

### Step 3: GUI Access

1. Open **Files → Other Locations → Windows Network**.
2. Select `WORKGROUP` → choose your Windows PC (e.g., `INTLAP-136-Ghulam`).
3. Enter credentials: username `ghulam`, password = Windows login password.

📌 *Note*: If your Windows account uses a PIN, you must enable password login. Create/set a password in **Settings → Accounts → Sign-in options → Password** and use that.

---

## 🔹 5. Troubleshooting

* **System error 67** → Wrong share name. Must match `[UbuntuShare]` exactly.
* **Extended error** → Windows blocking guest access. Fix by using Samba users instead of guest.
* **Permission denied** → Ensure correct folder permissions + Samba user credentials.
* **Windows not visible in Network** → Use `smb://<PCName>` or bookmark. Modern Windows disables SMB1 discovery, so browsing is unreliable.

Useful logs on Ubuntu:

```bash
sudo journalctl -u smbd -f
```

---

## 🔹 6. Summary

* Samba bridges Linux ↔ Windows file sharing.
* Reset config if things break, then reconfigure cleanly.
* Always use **SMB2/SMB3**.
* Define shares clearly in `/etc/samba/smb.conf`.
* Create Samba passwords with `smbpasswd`.
* For reverse sharing, ensure Windows local account has a real password (not just PIN).
* Use direct paths (`smb://PCName/Share`) or bookmarks if network browsing fails.

✅ With this setup, you can now drag-and-drop files between Ubuntu and Windows both ways.

---

📚 **References**

* [Ubuntu Samba Server Guide](https://ubuntu.com/server/docs/samba-introduction)
* [Microsoft SMB File Sharing](https://learn.microsoft.com/en-us/windows-server/storage/file-server/file-server-smb-overview)
* [PhoenixNAP Samba Tutorial](https://phoenixnap.com/kb/ubuntu-samba)
