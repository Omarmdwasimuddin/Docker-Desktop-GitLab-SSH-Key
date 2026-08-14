# Docker Desktop: GitLab SSH Key Setup
SSH key generate করে local Docker Desktop-এ চলা GitLab instance-এর account-এ add করা, তারপর SSH দিয়ে repository clone করা — পুরো process ধাপে ধাপে এখানে দেওয়া আছে।

---

## Step 1: GitLab-এ SSH Key Add করার জায়গা খুঁজে বের করা
Local GitLab dashboard-এ গিয়ে SSH key add করার option বের করো:

**http://localhost:8000/dashboard/projects** → Click: **Edit profile** → Click: **Access** → Click: **SSH keys** → Click: **Add new key**

---

## Step 2: PowerShell-এ Folder Create করা
PowerShell open করে এই command গুলো run করো — একটা `gitlab` folder create হবে এবং সেই folder-এ ঢুকবে:

```bash
mkdir gitlab
cd gitlab
```

---

## Step 3: SSH Key Generate করা
এই command দিয়ে SSH key pair generate করো (email তোমার GitLab account-এর email দিয়ে replace করতে পারো):

```bash
ssh-keygen -t ed25519 -C "omar.labib.softwareengineer@gmail.com"
```

Command run করার পর কিছু prompt আসবে, সেগুলো এভাবে handle করতে হবে:

**Prompt 1:**
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\User/.ssh/id_ed25519):
```
এখানে file path দিয়ে দাও:
```bash
C:\Users\User\.ssh\
```

**Prompt 2:**
```
Enter passphrase (empty for no passphrase):
```
Enter press করো (খালি রাখো)।

**Prompt 3:**
```
Enter same passphrase again:
```
আবার Enter press করো।

---

## Step 4: Key কোথায় Save হয়েছে দেখা
উপরের command গুলো ঠিকমতো complete হলে, terminal-এ এই রকম output আসবে — এটা দিয়ে বোঝা যাবে key কোথায় save হয়েছে:

```
Your identification has been saved in C:\Users\User\.ssh\gitlab_ed25519
Your public key has been saved in C:\Users\User\.ssh\gitlab_ed25519.pub
```

<img width="671" height="351" alt="image" src="https://github.com/user-attachments/assets/f97f3b3b-8f75-4d40-900b-2d8efed9a177" />

---

## Step 5: SSH Config File Setup (একাধিক key একসাথে manage করার জন্য)

একের বেশি SSH key থাকলে (GitHub, GitLab.com, local Docker GitLab) config file ছাড়া SSH client confuse হয়ে যায় কোনটা use করবে, এবং বারবার `-i` flag দিয়ে key specify করা লাগে। এটা এড়াতে একটা `config` file বানানো হয়।

`C:\Users\User\.ssh\config` file create/edit করো (extension ছাড়া, শুধু নাম `config`):

```bash
notepad C:\Users\User\.ssh\config
```

এই content paste করো:

```
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile C:/Users/User/.ssh/gitlab_ed25519
    IdentitiesOnly yes

Host github.com
    HostName github.com
    User git
    IdentityFile C:/Users/User/.ssh/id_ed25519
    IdentitiesOnly yes

Host my-gitlab-server
    HostName localhost
    Port 2222
    User git
    IdentityFile C:/Users/User/.ssh/gitlab_local_ed25519
    IdentitiesOnly yes
```

**কী কী করে এই config:**
- `Host` → alias, terminal-এ এই নামটা দিয়ে reference করবে (আসল hostname না)
- `HostName` → আসল server address
- `IdentityFile` → কোন key ব্যবহার হবে সেটা fix করে দেয়, ফলে SSH agent wrong key try করে error দেয় না
- `IdentitiesOnly yes` → শুধু specified key ব্যবহার হবে, অন্য কোনো loaded key try করবে না (multiple key থাকলে এটা must)
- `Port 2222` → Docker Desktop যদি local GitLab-এর SSH port 22 না হয়ে অন্য কিছুতে (যেমন 2222) expose করে, তাহলে এইটা দরকার — নাহলে connection refuse হবে

> ⚠️ **মনে রাখবে:** Docker container-এর actual SSH port check করে নিতে হবে (`docker ps` দিয়ে port mapping দেখো, যেমন `0.0.0.0:2222->22/tcp`)। যেটাই হোক, config-এর `Port` value সেটার সাথে match করতে হবে। এই document-এ `2222` এবং key filename `gitlab_local_ed25519` example হিসেবে দেওয়া — নিজের actual port এবং Step 4-এ generate হওয়া key-এর নাম (`gitlab_ed25519`) অনুযায়ী adjust করে নাও।

---

## Step 6: Public Key Copy করে GitLab-এ Paste করা
1. `gitlab_ed25519.pub` file-টা Notepad দিয়ে open করো
2. Full content copy করো (এটা হচ্ছে তোমার **public key**, private key না)
3. Local GitLab-এর "SSH Key" input box-এ paste করো
4. Click করো: **Add key**

> ⚠️ **মনে রাখবে:** শুধু `.pub` extension-ওয়ালা file-এর content paste করবে। Private key (`gitlab_ed25519`, extension ছাড়া file) কখনো কাউকে share করবে না।

---

## Step 7: Test করা — Repository Clone করা
এখন test করে দেখা যাবে SSH key ঠিকমতো কাজ করছে কিনা।

1. Local GitLab-এ গিয়ে তোমার project-এ visit করো (যেমন: **http://localhost:8000/<group>/<project>**)
2. Click: **Code** → **Clone with SSH** → copy করো clone URL-টা
3. PowerShell-এ project folder-এ গিয়ে (যেমন: `PS C:\Users\User\Desktop\gitlab>`) এই command run করো — এখানে `localhost` এর জায়গায় config-এ define করা alias `my-gitlab-server` ব্যবহার করো, যাতে port এবং correct key automatically apply হয়:

```bash
git clone git@my-gitlab-server:<group>/<project>.git
```

✅ Command successful হলে, project SSH দিয়ে clone হয়ে যাবে — মানে SSH key setup ঠিকমতো কাজ করছে।

---

## Quick Summary

| Step | কাজ |
|------|-----|
| 1 | Local GitLab-এ SSH Keys page-এ যাওয়া |
| 2 | Local-এ `gitlab` folder create করা |
| 3 | `ssh-keygen` দিয়ে key pair generate করা |
| 4 | Key file path check করা |
| 5 | SSH config file বানিয়ে host alias + port + key mapping set করা |
| 6 | Public key (`.pub`) local GitLab-এ paste করে add করা |
| 7 | Config alias ব্যবহার করে SSH দিয়ে repo clone করে test করা |
