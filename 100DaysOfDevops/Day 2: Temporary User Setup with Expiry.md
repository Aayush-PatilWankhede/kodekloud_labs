# Day 02 - Create a Linux User with an Expiry Date

## Overview

This challenge focuses on creating a temporary Linux user account with a predefined expiration date. Temporary user accounts are commonly used to provide short-term access to developers, contractors, or external team members while ensuring that access is automatically revoked after a specified date.

The objective is to create a user named `john` and configure the account to expire on `2027-04-15`.

---

## Objective

- Create a Linux user named `john`
- Set the account expiry date to `2027-04-15`
- Ensure the username is created in lowercase
- Verify that the expiry date has been configured correctly

---

## Services Used

| Service | Purpose |
|----------|----------|
| Linux User Management | Create and manage user accounts |
| `useradd` | Create new user accounts |
| `chage` | Manage user password and account aging information |
| SSH | Connect to remote servers |
| Sudo | Execute administrative commands |

---

## Implementation

### Connect to App Server 3

Log in to the target server where the user account needs to be created.

```bash
ssh banner@stapp03
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote Linux systems |
| `banner` | Existing user account used for server access |
| `stapp03` | Target application server hostname |

---

### Switch to Root User

Obtain administrative privileges to perform user management operations.

```bash
sudo su -
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `sudo` | Execute commands with elevated privileges |
| `su` | Switch user account |
| `-` | Load the target user's environment variables and profile |

---

### Create the User with an Expiry Date

Create the user account and set the expiration date to `2027-04-15`.

```bash
useradd -e 2027-04-15 john
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `useradd` | Creates a new Linux user account |
| `-e` | Specifies the account expiration date |
| `2027-04-15` | Date after which the account will be disabled |
| `john` | Username being created |

---

## Verification

### Verify User Expiry Information

Display account aging and expiration details for the user.

```bash
chage -l john
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `chage` | Manage user account aging information |
| `-l` | List account aging and expiration details |
| `john` | User account being verified |

Expected Output:

```text
Last password change                                    : Never
Password expires                                        : Never
Password inactive                                       : Never
Account expires                                         : Apr 15, 2027
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

The important section is:

```text
Account expires : Apr 15, 2027
```

which confirms that the account has been configured with the required expiration date.

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
│ via SSH             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Switch to Root      │
│ User                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create User         │
│ john                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Set Expiry Date     │
│ 2027-04-15          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Account      │
│ Expiration          │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh banner@stapp03` | Connect to App Server 3 |
| `sudo su -` | Switch to the root user |
| `useradd -e 2027-04-15 john` | Create a user with an expiry date |
| `chage -l john` | Verify account expiration details |

---

## Key Learnings

- Learned how to create Linux user accounts using `useradd`.
- Understood how to configure account expiration dates.
- Learned how temporary user accounts help improve security.
- Verified account aging information using the `chage` command.
- Practiced Linux user lifecycle management.

---

## Best Practices

- Assign expiration dates to temporary user accounts.
- Remove or disable unused accounts regularly.
- Follow the principle of least privilege.
- Periodically audit user accounts and expiration policies.
- Verify account settings immediately after creation.

---

## Challenge Status

✅ Successfully Completed
