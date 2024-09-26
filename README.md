# Setting Up an Arch Linux Droplet on DigitalOcean using `doctl`

#### This instruction will guide you to download and use doctl to create Droplet.
1. Create SSH keys
2. Install doctl
3. Create an API token
4. Use the API token to access to doctl

---
### Prerequisites
- DigitalOcean Account
- Uploaded Arch linux image in DigitalOcean Droplet
- Droplet environment with Arch linux 

### Install doctl
doctl is a command-line interface (CLI) tool on DigitalOcean. It allows you to interact with DigitalOcean's API from the command line, making it easier to automate tasks, and manage resources. 

**1. Open terminal as Administrator** 
**2. Type ssh arch to access Arch Linux environment**
```bash
ssh arch
```

**3. Install `wget`**
```bash
sudo pacman -Sy wget
```
- `sudo`: command that allows you to excute a command as the root user(super user).
- `pacman`: for Arch Linux, this command manage software packages.
- `-Sy`: Synchronize packages up to the latest versions.

	If this command shows you:
	`:: Proceed with installation? [Y/n]`
	Type `y` and press **Enter**

**4**. **Download the most recent version of doctl** 
```bash
cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.110.0/doctl-1.110.0-linux-amd64.tar.gz
```

**5. Extract the binary**
```bash
tar xf ~/doctl-1.110.0-linux-amd64.tar.gz
```

**6. Move the doctl binary into the path**
```bash
sudo mv ~/doctl /usr/local/bin
```

> [!note] Why we need to move the path?
> When you move `doctl` to `/usr/local/bin`, it ensures that the system can find and execute the `doctl` command from anywhere.
> This is the same action of modifying System variables in Windows OS.

Now you have installed `doctl` successfully!
### Create an API Token
You need to create a personal token to authenticate to the API. It allows the CLI tool to interact with your DigitalOcean account programmatically.

**1. log in to the [DigitalOcean Control Panel](https://cloud.digitalocean.com/)**
**2. Click API in the left menu**
![[Screenshot 2024-09-24 203825.png]]
**3. Click Generate New Token in the middle of Personal Access Tokens area** 
![[Screenshot 2024-09-24 204148.png]]
**4. Fill out Token name and choose options, then Click generate Token
![[Screenshot 2024-09-24 204619.png]]
- **Expiration**: You can choose depends on usage, the shorter expiry date is considered the more secure in general, but requires frequent renewal.
- **Scopes**: Since this token is for your own project purpose, choose Full access. Read Only access is recommended when modification should be prevented.
**5. Find the token and Click copy**
![[Screenshot 2024-09-24 210310.png]]

>[!Important!] 
>The token is only shown once. If you lose it, you need to create a new one.

**6. In arch linux shell, type below
```bash
echo "your_token_code" > ~/.ssh/my_token_name.txt
```
- This command write your copied token into a txt file in the folder .ssh so that you can save your token information.
- Make sure change **"your_token_code"** to **your actual code that you copied,** as well as change the **my_token_name** to the **filename** you want to use.

### Use API token to authenticate doctl
>[!Note] Get ready for your API token
>You will be required to type the API token that you just created.
>Copying it into clipboard before you start this steps. That will make steps easier

**1. To initiate authentication, type below in the arch linux shell**
```bash
doctl auth init --context <name>
```
- This will allow you to switch between multiple accounts that are authenticated.

**2. Type your API token**
`Enter your access token:    ✱ required`
type your API token above.
Then you will see:
`Validating token... ✔`
>[!Error Handling]
>It might show you an error:
>`Error: Unable to initialize DigitalOcean API client: access token is required. (hint: run 'doctl auth init')`
>If so:
>run `doctl auth init` and follow step 2 again.

**3.Validate doctl account**
You can check your account details by running codes below
```bash
doctl account get
```
If successful, you will see the output like:
```bash
Email              Droplet Limit    Email Verified    UUID              Status
sammy@example.org  10               true              3a56c5e109737c    active
```
- `Email`: The identity of the account
- `Droplet Limit`: The limits of your droplets on your account
- `Email Verified`: Your account has been verified
- `UUID`: The unique identifier used for internal operations in DigitalOcean
- `Status`: The status of your account

Your doctl account has successfully validated by API token.

### Create SSH keys 
SSH keys work as a pair which one is public, another is private. Because of that feature, this provides stronger security than passwords methods. SSH keys will authenticate you when connecting to a DigitalOcean droplet.

**1. Open Terminal**
**2. Type ssh arch to access Arch Linux environment**
```bash
ssh arch
```

> [!note] On Arch Linux, `ssh-keygen` is typically included with the OpenSSH package. 
>
> **Check if `ssh-keygen` is installed**
>```bash
ssh-keygen -v
>```
**If it's not installed, install it using `pacman`**
>```bash
sudo pacman -S openssh

**3. Create a new SSH key pair
```bash
ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"
```
- `-t ed25519`: Specifies the type of encryption (Ed25519)
- `-f ~/.ssh/do-key`: Specifies the filename and location to save the key.
- `-C "your email address"`: Adds a comment to the key

> [!note] Should I create passphrase?
> **You will be asked to:**
> `Enter passphrase (empty for no passphrase):`
> `Enter same passphrase again:`
> 
This will require you to type the passphrase every time you try to connect to your Arch Linux system.
> You may want to skip setting a passphrase for convenience if:
> - You are the only one who has access to your computer environment
>
**To skip**: Press **Enter** twice when prompted.
> 
> Consider setting a passphrase if:
> - You want an extra layer of security
> - You share your computer environment with others
>
>**To Create your own passphrase**: Enter a passphrase when prompted and then re-enter it to confirm.

**4. Check SSH Keys
```bash
ls ~/.ssh
```
If this command shows you:
	`do-key`: Your private key
	`do-key.pub`: Your public key

Your have successfully created your SSH Keys. 



### Connect SSH keys to doctl
To connect your SSH keys to doctl, follow these steps:

**1. Open Terminal**
**2. Type ssh arch to access Arch Linux environment**
```bash
ssh arch
```

**3. Connect SSH key to DigitalOcean**
```bash
doctl compute ssh-key import do-key --public-key-file ~/.ssh/do-key.pub
```
- `compute`: command to work with a Droplet
- `ssh-key import`: importing an SSH key.
- `do-key`: This is the name of SSH key you give within DigitalOcean. 
- `--public-key-file ~/.ssh/do-key.pub`: get public key from the path.

You will see the name of key and key ID like:
```bash
ID          Name      FingerPrint
43506344    do-key    db:7c:d0:4b:dc:6c:24:ac:2b:5d:c6:9e:d7:bc:d8:18
```

You have successfully connected your SSH key to DigitalOcean.

