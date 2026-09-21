# SSH - Simple Notes

## What is SSH?

SSH = **Secure Shell**.

It lets you remotely access another computer/server through a terminal.

```text
YOUR PC
   |
   | SSH
   ↓
SERVER
   |
   ├── Files
   ├── Programs
   ├── Docker
   └── Services
```

## Basic Command

```bash
ssh user@server
```

Example:

```bash
ssh paramesh@192.168.1.20
```

After connecting, you are working on the **server's terminal**.

For example:

```bash
docker ps
```

shows Docker containers running on that server.

```bash
ls
```

shows files on that server.

## Why SSH is Useful

Without physically being at the server:

```text
Your PC
   |
   | SSH
   ↓
Remote Server
```

You can:

- Run commands
- Check files
- Start/stop programs
- Check logs
- Troubleshoot the server

## SSH Keys

Instead of using a password, SSH can use an SSH key pair:

```text
Your PC
 ├── Private key 🔑
 └── Public key

        |
        | SSH
        ↓

Server
 └── Authorized public key
```

**Private key:** stays on your PC.

**Public key:** can be placed on the server.

## Simple Memory

```text
SSH = remotely enter a server's terminal
```

Example:

```bash
ssh user@server
```

Then you can run commands as if you were sitting in front of that server.
