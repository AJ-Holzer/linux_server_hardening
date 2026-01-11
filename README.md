# Linux Server Hardening
## Steps

### Step 1 - Update Server Automatically
```shell
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Allow downloading and installing stable updates

**Verify it is running:**
```shell
sudo systemctl status unattended-upgrades
```

### Step 2 - Create User & add to Sudo Group
> [!warning]
> Another user than `root` might already exist on your machine! If so, you don't need to create an additional one.

#### Add the user
```shell
sudo adduser yourusername
```

#### Add to sudo group
```shell
sudo usermod -aG sudo yourusername
```

### Step 3 - Set Up Public & Private Keys
> [!warning]
> The `.ssh` directory might already exist on your machine! But change the permissions anyway...

#### Create the `.ssh` directory and change the permissions
**On the server (logged in as `yourusername`), create the `.ssh` directory:**
```shell
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

#### Create the public and private keys on your machine
> [!warning]
> If you do NOT enter a password, you don't need the `-a` flag at all. Because it just doesn't do anything then.

```shell
ssh-keygen -t ed25519 -a 100
```

> [!note]
> - `-t`: The type of the keypair.
> - `-a`: Number of key derivation function rounds.

After running the command it will ask you for a passphrase. You don't need to enter one, but it is more secure.

#### Upload Keys on your machine to the server
**On Windows:**
```shell
scp $env:USERPROFILE/.ssh/id_ed25519.pub yourusername@yourserverip:~/.ssh/authorized_keys
```

**On Linux:**
```shell
ssh-copy-id ~/.ssh/id_ed25519.pub yourusername@yourserverip
```


#### Set up Fail2Ban on the server
**Install and enable:**
```shell
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
```

**Verify Fail2Ban is running:**
```shell
sudo systemctl status fail2ban
```

### Step 4 - Disable Password Authentication
#### Login to your server again
```shell
ssh yourusername@yourserverip
```

#### Change sshd config
```shell
sudo nano /etc/ssh/sshd_config
```

**Uncomment and change the lines:**
- `PermitRootLogin`: Change to `no`
- `PasswordAuthentication`: Change to `no`
- `AuthenticationMethods`: Change to `publickey`
- `PubkeyAuthentication`: Change to `yes`
- `MaxAuthTries`: Max number of login tries (e.g. `3`)
- `LoginGraceTime`: How long the client has to authenticate in seconds (e.g. `30`)
- `AllowUsers`: This prevents other local accounts from authenticating via SSH, even if one is added later (set it to `yourusername`)

#### Restart sshd
**Verify and restart sshd:**
```shell
sudo sshd -t && sudo systemctl restart sshd
```

> [!warning]
> If SSH fails after restart, comment out `AuthenticationMethods` and rely on `PasswordAuthentication no`.

#### Login from another terminal
> [!warning]
> Do NOT close the other connection yet! There could still be some misconfigurations that can log you out.

**Should work:**
```shell
ssh yourusername@yourserverip
```

> [!note]
> You can close the previous connection if everything is working now.

### Step 5 - Set up Firewall
#### Install firewall
```shell
sudo apt install ufw
```

#### Open ssh port
> [!warning]
> You can NOT log in if the `ssh` port is not opened.

```shell
sudo ufw allow ssh
```

#### Enable firewall
```shell
sudo ufw enable
```

#### Verify firewall works
```shell
sudo ufw status
```

#### Log in from another terminal
> [!warning]
> Do NOT close the other connection yet! There could still be some misconfigurations that can log you out.

```shell
ssh yourusername@yourserverip
```

> [!note] Note
> You can close the previous connection if everything works perfectly fine.

#### Block ICMP echo requests (optional)
> [!tip]
> This step is optional, not a hardening requirement.

**Open the UFW rule file:**
```shell
sudo nano /etc/ufw/before.rules
```

**Add this line at the end of the section `# ok icmp codes for INPUT`:**
```nginx
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```

**Reboot the server:**
```shell
sudo reboot now
```

**Check on your machine if you can ping the server:**
```shell
ping yourserverip
```

You should see something like this:
```shell
Request timed out.
```
