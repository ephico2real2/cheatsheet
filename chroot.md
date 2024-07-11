### Step 1: Create the Base Directory

First, create the base chroot directory and set the ownership and permissions:

```bash
sudo mkdir -p /sftp/chroot
sudo chown root:root /sftp/chroot
sudo chmod 755 /sftp/chroot
```

### Step 2: Create a New User and Directories

Create a new user with no login shell and set up their home directory and subdirectories:

```bash
USERNAME=aptestUser1
USER_HOME="/sftp/chroot/$USERNAME"
FOLDER1="$USER_HOME/folder1"
FOLDER2="$USER_HOME/folder2"

# Create the user
sudo useradd -m -d $USER_HOME -s /usr/sbin/nologin $USERNAME
sudo passwd $USERNAME

# Create the user's directories
sudo mkdir -p $FOLDER1 $FOLDER2
```

### Step 3: Set Ownership and Permissions

Set the ownership and permissions for the base directory and the user's directories:

```bash
# Set ownership and permissions for the base directory
sudo chown root:root /sftp/chroot/$USERNAME
sudo chmod 755 /sftp/chroot/$USERNAME

# Set ownership and permissions for the user's directories
sudo chown $USERNAME:$USERNAME $FOLDER1 $FOLDER2
sudo chmod 755 $FOLDER1 $FOLDER2
```

### Step 4: Set ACLs for the User's Directories

Set ACLs to ensure the user has the necessary permissions and set default ACLs to inherit permissions for any new files or directories created by the user:

```bash
# Set ACLs for the user's directories
sudo setfacl -m u:$USERNAME:rwx $FOLDER1
sudo setfacl -m u:$USERNAME:rwx $FOLDER2

# Set default ACLs so new files and directories inherit the correct permissions
sudo setfacl -d -m u:$USERNAME:rwx $USER_HOME
sudo setfacl -d -m u:$USERNAME:rwx $FOLDER1
sudo setfacl -d -m u:$USERNAME:rwx $FOLDER2

# Ensure the base chroot directory is not writable by the user
sudo setfacl -m u:$USERNAME:--x /sftp/chroot
```

### Step 5: Configure SSH for Chroot SFTP

Edit the SSH configuration file `/etc/ssh/sshd_config` to include the chroot settings for the users:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following lines at the end of the file:

```bash
Match User aptestUser*
    ChrootDirectory /sftp/chroot/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

### Step 6: Restart SSH Service

Restarting the SSH service is necessary to apply the configuration changes. The `sshd` daemon (SSH server) needs to reload its configuration to enforce the new settings.

```bash
sudo systemctl restart ssh
```

### Why Restart the SSH Service?

Restarting the SSH service is required because:

1. **Apply Configuration Changes**: Any changes made to the `sshd_config` file will not take effect until the SSH service is restarted or reloaded.
2. **Ensure Security**: Restarting ensures that all active SSH sessions are re-evaluated under the new configuration, ensuring consistency and security.

### Testing the Configuration

To test the setup, connect via SFTP as the new user:

```bash
sftp username@your_server_ip
```

You should be able to access the `folder1` and `folder2` directories and any new files or directories created within them should inherit the ACLs set.

By following these steps, you can manually set up a secure chroot environment for SFTP users with proper ACLs, ensuring that any new directories or files created by the users inherit the appropriate permissions.
