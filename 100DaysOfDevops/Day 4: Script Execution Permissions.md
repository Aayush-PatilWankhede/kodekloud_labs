# Day 04 - Manage Script Permissions in Linux

## Overview

This challenge focuses on managing file permissions in Linux. Proper permissions are essential to ensure that scripts can be executed by the required users while maintaining system security.

The objective is to grant execute permissions to the script `xfusioncorp.sh` on App Server 2 so that all users (Owner, Group, and Others) can execute it.

---

## Objective

- Connect to App Server 2
- Verify the current permissions of the script
- Grant execute permissions to all users
- Validate the updated permissions
- Ensure the script can be executed by Owner, Group, and Others

---

## Services Used

| Service | Purpose |
|----------|----------|
| Linux File System | Store and manage files |
| chmod | Modify file permissions |
| ls | Display file details and permissions |
| SSH | Remote server administration |

---

## Implementation

### Connect to App Server 2

Log in to the target server where the script is located.

```bash
ssh steve@stapp02
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote Linux systems |
| `steve` | User account used for login |
| `stapp02` | Target application server |

---

### Check Current Script Permissions

View the existing permissions assigned to the script.

```bash
ls -l /tmp/xfusioncorp.sh
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ls` | Lists files and directories |
| `-l` | Displays detailed file information including permissions |
| `/tmp/xfusioncorp.sh` | Target script file |

Example Output:

```text
-rw-r--r-- 1 root root 256 May 20 10:00 /tmp/xfusioncorp.sh
```

The output indicates that the script currently does not have execute permissions.

---

### Grant Execute Permission to All Users

Add execute permissions for the Owner, Group, and Others.

```bash
chmod a+x /tmp/xfusioncorp.sh
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `chmod` | Change file permissions |
| `a` | Applies to all users (Owner, Group, Others) |
| `+x` | Adds execute permission |
| `/tmp/xfusioncorp.sh` | Target script file |

Alternative command:

```bash
chmod 755 /tmp/xfusioncorp.sh
```

#### Command Explanation

| Permission | Description |
|------------|-------------|
| `7` | Owner: Read, Write, Execute |
| `5` | Group: Read, Execute |
| `5` | Others: Read, Execute |

Both commands achieve the required result.

---

## Verification

### Verify Updated Permissions

Check the script permissions after applying the changes.

```bash
ls -l /tmp/xfusioncorp.sh
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ls -l` | Display detailed file permissions and ownership information |

Expected Output:

```text
-rwxr-xr-x 1 root root 256 May 20 10:00 /tmp/xfusioncorp.sh
```

The important part is:

```text
-rwxr-xr-x
```

which confirms:

- Owner can Read, Write, and Execute
- Group can Read and Execute
- Others can Read and Execute

This satisfies the requirement that all users can execute the script.

---

## Workflow

```text
┌─────────────────────┐
│ System Administrator│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Connect to Server   │
│ stapp02             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Check Existing      │
│ Permissions         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Grant Execute       │
│ Permission          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Updated      │
│ Permissions         │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh steve@stapp02` | Connect to App Server 2 |
| `ls -l /tmp/xfusioncorp.sh` | View file permissions |
| `chmod a+x /tmp/xfusioncorp.sh` | Grant execute permission to all users |
| `chmod 755 /tmp/xfusioncorp.sh` | Set standard executable permissions |
| `ls -l` | Verify updated permissions |

---

## Key Learnings

- Learned how Linux file permissions control access to files.
- Understood how execute permissions affect script execution.
- Used `chmod` to modify file permissions.
- Verified permissions using `ls -l`.
- Learned the difference between symbolic and numeric permission formats.

---

## Best Practices

- Grant only the permissions required for a task.
- Verify permissions after making changes.
- Use symbolic permissions (`a+x`) for clarity when modifying specific rights.
- Use numeric permissions (`755`) for standardized permission settings.
- Regularly audit executable scripts for unnecessary permissions.

---

## Challenge Status

✅ Successfully Completed
