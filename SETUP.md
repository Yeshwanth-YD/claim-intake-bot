# Setup guide: do this once, then build from minute 1

Time: about 60–75 minutes. Work top to bottom. Each part ends with a check, so you know it worked before moving on.
Your project folder is: `C:\Users\rcdha\source\repos\claim-intake-bot`

---

## Part 0 · What each tool is for

| Tool | What you use it for | Where it lives |
|---|---|---|
| **UiPath Studio** | Building the bots (your main workspace, like an IDE for UiPath) | Installed on your PC |
| **UiPath Automation Cloud** | Website that holds Orchestrator, Integration Service, Action Center and Document Understanding | cloud.uipath.com |
| **Orchestrator** | Stores queues and assets (settings, passwords) and runs published bots | Inside Automation Cloud |
| **Integration Service** | The OneDrive connection and the "file dropped" trigger | Inside Automation Cloud |
| **Action Center** | Where a person checks low-confidence fields | Inside Automation Cloud |
| **Chrome or Edge + UiPath extension** | Lets Studio click and type in the mock claims app | Your browser |
| **Git for Windows + GitHub** | Version control and your public portfolio link | PC + github.com |
| **VS Code** | Editing README and SPEC, previewing Markdown, viewing Git changes. **Not** for building bots. Never edit `.xaml` files in VS Code. | Already installed |

---

## Part 1 · Accounts (15 min)

You need three. Use **personal** accounts for all of them, not your SJS Consulting work account. Work tenants often block third-party connections, and a personal demo shouldn't live in an employer's systems.

### 1.1 UiPath account
1. Go to **cloud.uipath.com** and choose **Sign up**. Sign up with your personal Gmail.
2. When asked, pick the **Community** (free) plan.
3. Organization name: something like `yeshwanth-demo`. Region: **Australia** if offered, otherwise the nearest.
4. You land on the Automation Cloud home page. Leave this tab open.

### 1.2 Microsoft personal account (for OneDrive)
You already have personal OneDrive on this PC. Confirm which email it uses: click the OneDrive cloud icon in the taskbar, then **Settings > Account**. That's the account you'll connect in Part 5.

### 1.3 GitHub account
If you don't have one, sign up at **github.com** with your personal email. Pick a professional username (it appears in the link you'll send Davidson).

**Check:** you can sign in to cloud.uipath.com and github.com, and you know your personal OneDrive email.

---

## Part 2 · Install software (20 min)

### 2.1 Git for Windows
1. Open **PowerShell** (Start menu, type PowerShell).
2. Run `git --version`. If you see a version number, skip to step 4.
3. If not, download from **git-scm.com/download/win**, install with all defaults, then close and reopen PowerShell.
4. Tell Git who you are (use your GitHub email):
   ```
   git config --global user.name "Yeshwanth Sai Vivek"
   git config --global user.email "you@example.com"
   ```

### 2.2 UiPath Studio
1. On the Automation Cloud home page, find **Download Studio** (also under **Resource Center** or the help menu). Download and run the installer.
2. Choose the **Community** / user-mode install if asked.
3. Launch **UiPath Studio**. Sign in with your UiPath account. It connects to your Automation Cloud automatically (you'll see your organization name at the bottom of the Studio home screen).
4. If asked which profile: choose **UiPath Studio** (not StudioX).

### 2.3 Browser extension (important)
1. In Studio: **Home > Tools > UiPath Extensions**, click **Chrome** (or **Edge**, whichever you'll use), and follow the prompt to add it in the browser.
2. In the browser, open the extensions page (`chrome://extensions` or `edge://extensions`), click **Details** on the UiPath extension, and turn on **Allow access to file URLs**.
   Without this, Studio can't see the mock app, because it opens from a file on your PC, not a website.

**Check:** `git --version` works, Studio opens and shows your org, and the UiPath extension shows "Allow access to file URLs" switched on.

---

## Part 3 · Look at the kit (5 min)

1. Open File Explorer at `C:\Users\rcdha\source\repos\claim-intake-bot`.
2. Double-click `mock-legacy-app\claims.html`. Sign on with **demo / demo123**, enter one made-up claim, click **Save**, then **Claim List**. Click **Delete All (reset demo)** when done.
3. Open `sample-claims\claim_01...pdf` and `claim_08...pdf` (the messy scan), and `ground_truth.csv`.
4. Open VS Code: **File > Open Folder** and choose the project folder. Open `docs\SPEC.md` and press **Ctrl+Shift+V** to read it formatted.

**Check:** the mock app saved a claim and showed it in the list.

---

## Part 4 · Orchestrator setup (10 min)

In Automation Cloud, open **Orchestrator** (left menu). Your work goes in the **Shared** folder (top-left folder picker). Creating a separate folder isn't needed for the demo.

### 4.1 Queue
1. **Queues > Add Queue**.
2. Name: `ClaimIntake`
3. **Unique Reference: Yes** · **Auto Retry: Yes** · **Max # retries: 2**
4. Save.

### 4.2 Assets
**Assets > Add Asset**, three times:

| Name | Type | Value |
|---|---|---|
| `LegacyApp_Credential` | Credential | Username `demo`, Password `demo123` |
| `LegacyApp_Path` | Text | `C:\Users\rcdha\source\repos\claim-intake-bot\mock-legacy-app\claims.html` |
| `Inbox_FolderName` | Text | `ClaimsInbox` |

### 4.3 Licences (look only)
Open **Admin** (left menu) **> Licenses**. Note whether you have **Unattended** robots and **AI Units**.
- Unattended available: the event trigger can start the bot by itself (Day 4).
- Not available: no problem. You'll run the bots from Studio or UiPath Assistant for the demo, which is still a full end-to-end demo.

**Check:** the queue and three assets show in Orchestrator.

---

## Part 5 · Integration Service connection (10 min)

1. In personal OneDrive (onedrive.live.com or your OneDrive folder), create a folder called **ClaimsInbox**.
2. In Automation Cloud, open **Integration Service > Connectors**.
3. Search **Microsoft OneDrive & SharePoint** and open it, then **Add connection**.
4. Sign in with your **personal** Microsoft account and accept the permissions.
5. Make sure the connection is in the **Shared** folder.

**Check:** the connection shows a green **Connected** status.

If sign-in fails or is blocked: don't fight it tonight. Day 4 has a fallback (a local folder trigger), and the rest of the build doesn't depend on it.

---

## Part 6 · GitHub repo (10 min)

1. On github.com: **New repository**. Name `claim-intake-bot`. Visibility **Public**. Do **not** add a README, .gitignore or licence (the kit already has them).
2. In PowerShell:
   ```
   cd C:\Users\rcdha\source\repos\claim-intake-bot
   git init
   git add .
   git commit -m "Starter kit: mock legacy app, synthetic claims, specs"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/claim-intake-bot.git
   git push -u origin main
   ```
3. A browser window opens to sign in to GitHub the first time. Approve it.

**Check:** refresh the repo page; you see the README and folders.

---

## Part 7 · Create the two Studio projects (10 min)

### 7.1 Performer
1. Studio home: **New Project > Robotic Enterprise Framework** (under Templates).
2. Name: `ClaimIntake.Performer`
3. Location: `C:\Users\rcdha\source\repos\claim-intake-bot`
4. Compatibility: **Windows**. Language: **VB** (most common in enterprise UiPath teams) or C# if you prefer.
5. Create. Studio opens `Main.xaml` showing the REFramework states. Close it for now.

### 7.2 Dispatcher
1. Studio home: **New Project > Process**.
2. Name: `ClaimIntake.Dispatcher`, same location, Windows, same language.

### 7.3 Git in Studio
Because the folder is already a Git repo, Studio detects it: the bottom status bar shows **main**. From tomorrow, commit from Studio's **Commit** button at the end of each session (or from VS Code's Source Control panel; either is fine).

### 7.4 First commit of the projects
In PowerShell (or VS Code Source Control):
```
git add .
git commit -m "Empty Performer (REFramework) and Dispatcher projects"
git push
```

**Check:** both project folders show on GitHub.

---

## Ready-for-tomorrow checklist

- [ ] UiPath, personal Microsoft and GitHub accounts working
- [ ] Studio installed, signed in, organization showing
- [ ] Browser extension installed with file URL access on
- [ ] Mock app saves a claim
- [ ] Queue `ClaimIntake` + 3 assets in Orchestrator Shared folder
- [ ] OneDrive connection Connected (or fallback noted)
- [ ] GitHub repo pushed with both empty projects
- [ ] Licence situation noted (unattended yes/no)

Tomorrow: open Studio > `ClaimIntake.Performer`, and VS Code > `docs\SPEC.md` at **Daily build order, Day 1**.
