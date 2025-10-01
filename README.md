# 📘 File Sharing Between Ubuntu and Windows 11 Using Samba

This guide will help you (as a student) understand the **basic concept of Samba**, and then walk you through the **step-by-step setup** with commands, configuration, and references. By the end, you’ll be able to transfer files easily between your Ubuntu desktop and Windows 11 laptop on the same network.

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

### Step 2: Backup and Reset Config (optional)

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
sudo cp /usr/share/samba/smb.conf /etc/samba/smb.conf
```

### Step 3: Edit Samba Config

Open config file:

```bash
sudo nano /etc/samba/smb.conf
```

Use this minimal working configuration:

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

Save and exit (`CTRL+O`, `Enter`, `CTRL+X`).

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

Check if Ubuntu is reachable:

```powershell
ping 192.168.31.83
Test-NetConnection 192.168.31.83 -Port 445
```

### Step 2: Access Share

Open **File Explorer** and type:

```
\\192.168.31.83\UbuntuShare
```

Enter username: `ghulam` and Samba password.

If you prefer the command line, clear any existing mappings and connect directly:

```powershell
net use * /delete /y
net use \\192.168.31.83\UbuntuShare /user:ghulam <SambaPassword>
```

📸 *Reference screenshot (Windows run share path)*: ![Run Samba Share](https://www.linuxtechi.com/wp-content/uploads/2020/07/Access-Samba-Share-Windows10.png)

### Step 3: Map as Network Drive

1. Open **This PC** → **Map Network Drive**
2. Choose drive letter (e.g., Z:)
3. Enter:

   ```
   \\192.168.31.83\UbuntuShare
   ```
4. Check **Reconnect at sign-in**
5. Provide Samba credentials

📸 *Reference screenshot (Map drive)*: ![Map Network Drive](https://www.ubuntubuzz.com/wp-content/uploads/2021/01/map-network-drive.png)

---

## 🔹 4. Troubleshooting

### Common Issues

* **System error 67** → Wrong share name. Must match the config section `[UbuntuShare]` exactly.
* **Extended error** → Windows blocking guest access. Fix by using Samba users instead of guest.
* **Permission denied** → Check folder ownership and Samba user permissions.

### Useful Logs

On Ubuntu, monitor logs while testing:

```bash
sudo journalctl -u smbd -f
```

---

## 🔹 5. Summary

* Samba bridges Linux ↔ Windows file sharing.
* Always use **SMB2/SMB3** for security.
* Define a clear share in `/etc/samba/smb.conf`.
* Use `smbpasswd` to create a Samba password.
* On Windows, access with `\\IP\ShareName` or map as a drive.

✅ With this setup, you can now drag-and-drop files between your Ubuntu desktop and Windows 11 laptop as if they were on the same OS.

---

📚 **References for further reading**

* [Ubuntu Samba Server Guide](https://ubuntu.com/server/docs/samba-introduction)
* [Microsoft SMB File Sharing](https://learn.microsoft.com/en-us/windows-server/storage/file-server/file-server-smb-overview)
* [PhoenixNAP Samba Tutorial](https://phoenixnap.com/kb/ubuntu-samba)
