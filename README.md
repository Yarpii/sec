# OpenSSH Key Management on Windows

This README provides an overview of how to use OpenSSH on Windows for key-based authentication. Key-based authentication offers enhanced security compared to traditional username-password pairs, making it a valuable choice for securing your Windows systems, especially when working across different domains.

## Introduction

In Windows environments, most authentication is typically done using a username and password. However, this approach can be vulnerable to brute force attacks, especially when connecting systems across different domains. In contrast, Linux environments commonly use public-key/private-key pairs for authentication, which eliminates the need for easily guessable passwords.

OpenSSH, a widely-used tool for secure remote access, offers robust support for key-based authentication on Windows. This document will guide you through the process of using OpenSSH to set up key-based authentication.

If you are new to SSH key management, we recommend reviewing the NIST document IR 7966 titled "Security of Interactive and Automated Access Management Using Secure Shell (SSH)" for a deeper understanding of the topic.

## About Key Pairs

Key pairs consist of two essential files: a public key and a private key. These files are crucial for various authentication protocols.

- **Public Key**: This file is placed on the SSH server and can be shared without compromising security.

- **Private Key**: The private key is equivalent to a password and must be kept confidential. If someone gains access to your private key, they can impersonate you on any SSH server where you have access.

When using key-based authentication, the SSH server and client compare the public key provided by the user against the private key. If the server's public key cannot be verified against the client's private key, authentication fails.

Multi-factor authentication can be implemented with key pairs by adding a passphrase during key generation. The passphrase, combined with the presence of the private key on the SSH client, provides an additional layer of security during authentication.

## Host Key Generation

Public keys have specific access control requirements on Windows, allowing access only to administrators and the System account. When you first use `sshd`, the host's key pair is automatically generated.

**Note**: Ensure that you have OpenSSH Server installed. Refer to [Getting started with OpenSSH](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) for installation instructions.

By default, the `sshd` service is set to start manually. To start it automatically on server reboot, run the following commands in an elevated PowerShell prompt on your server:

```powershell
# Set the sshd service to start automatically
Get-Service -Name sshd | Set-Service -StartupType Automatic

# Start the sshd service
Start-Service sshd
```

Since there is no user associated with the `sshd` service, the host keys are stored under `C:\ProgramData\ssh`.

## User Key Generation

To enable key-based authentication, you need to generate public/private key pairs for your client. You can use `ssh-keygen.exe` to generate key files, specifying algorithms like DSA, RSA, ECDSA, or Ed25519. If no algorithm is specified, RSA is used. It is recommended to use strong algorithms and key lengths, such as Ed25519.

To generate key files using the Ed25519 algorithm, run the following command in a PowerShell or cmd prompt on your client:

```powershell
ssh-keygen -t ed25519
```

You'll be prompted to enter a file path and/or filename for your keys. You can press Enter to accept the default or specify a custom location. Optionally, you can set a passphrase to encrypt your private key files, enhancing security.

After generating the keys, you'll have a public/private Ed25519 key pair. The `.pub` files are public keys, while files without an extension are private keys.

Remember to protect private key files as you would a password. Use `ssh-agent` to securely store private keys within a Windows security context associated with your Windows login. To do this, follow these steps:

```powershell
# Allow manual start of the ssh-agent service (ensure you run as Administrator)
Get-Service ssh-agent | Set-Service -StartupType Manual

# Start the ssh-agent service
Start-Service ssh-agent

# Check if the service is running (it should return "Running")
Get-Service ssh-agent

# Load your key files into ssh-agent
ssh-add ~\.ssh\id_ed25519
```

After completing these steps, `ssh-agent` will automatically retrieve the local private key whenever authentication is required from this client.

**Important**: It is strongly recommended to back up your private key to a secure location and then delete it from the local system after adding it to `ssh-agent`. If you lose access to the private key, you will need to create a new key pair and update the public key on all systems where you use it.

## Deploying the Public Key

To use the user key you created, you need to place the contents of your public key (`~\.ssh\id_ed25519.pub`) on the server in a text file. The name and location of this file depend on whether your user account is a member of the local administrators group or a standard user account.

### Standard User

For standard user accounts, place the public key in a text file named `authorized_keys` in `C:\Users\username\.ssh\`. The `scp` utility, included with OpenSSH, can assist in this process. Use the following example, replacing "username" with your actual username:

```powershell
# Ensure that the .ssh directory exists in your server's user account home folder
ssh username@domain1@contoso.com mkdir C:\Users\username\.ssh\

# Use scp to copy the public key file from your client to the authorized_keys file on your server
scp C:\Users\username\.ssh\id_ed25519.pub user1@domain1@contoso.com:C:\Users\username\.ssh\authorized_keys
```

### Administrative User

For administrative user accounts, place the public key in a text file named `administrators_authorized_keys` in `C:\ProgramData\ssh\`. The `scp` utility can help with this process, and you need to configure the ACL to allow access only to administrators and the System account.

Use the following example, replacing "username" with your actual username:

```powershell
# Ensure that the .ssh directory exists in your server's user account home folder
ssh user1@domain1@contoso.com mkdir C:\ProgramData\ssh\

# Use scp to copy the public key file from your client to the administrators_authorized_keys file on your server
scp C:\Users\username\.ssh\id_ed25519.pub user1@domain1@contoso.com:C:\ProgramData\ssh\administrators_authorized_keys

# Configure the ACL for the authorized_keys file on your server
ssh --% user1@domain1@contoso.com icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

These steps complete the configuration required to use key-based authentication with OpenSSH on Windows. After setup, you can connect to the SSH host from any client with the private key stored in `ssh-agent`.

Please ensure you follow security best practices, including proper key management and access control, to maintain a secure SSH environment.
