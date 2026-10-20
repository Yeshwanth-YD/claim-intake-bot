# Day 1 · Performer logs in and signs off

**Goal for this hour:** run `Main.xaml`, and watch the bot open the mock claims app, sign on, find no work in the queue, then sign off and close. That proves config, assets, credentials, the browser and REFramework's Init and End states all work.

Your Studio (2026.0) uses the newest REFramework template. It keeps settings in **`Data\Config.json`** (plain text, no Excel needed) and reads assets directly with activities. The template's own guide is in `ClaimIntake.Performer\Framework\AGENTS.md` if you want the full reference.

Open `ClaimIntake.Performer` in Studio. Keep this file open in VS Code beside it.

---

## Step 1 · Point Studio at the Shared folder (2 min)

At the bottom of Studio, find the **Orchestrator folder** dropdown, and pick **Shared**. Studio now reads assets and queues from Shared when you run from Studio.

## Step 2 · Config.json (already done for you)

`Data\Config.json` now has these Settings:

| Key | Value | Used for |
|---|---|---|
| `logF_BusinessProcessName` | `ClaimIntake` | Groups this bot's logs in Orchestrator |
| `LegacyApp_PathAsset` | `LegacyApp_Path` | Name of the asset holding the path to `claims.html` |
| `LegacyApp_CredentialAsset` | `LegacyApp_Credential` | Name of the credential asset |
| `OrchestratorFolder` | `Shared` | Folder the assets and queue live in |

Read any of them in a workflow with `in_Config("LegacyApp_PathAsset").ToString`.

The config holds asset **names**, never passwords: the file ships in plain text. `MaxRetryNumber` stays **0**, because the queue does the retries (set to 2).

## Step 3 · Select the queue (3 min)

1. Open `Framework\GetTransactionData.xaml`.
2. Find the **Get transaction item** activity (inside "Try Catch Get transaction item").
3. In **Queue name**, pick **ClaimIntake** from the dropdown. Leave the folder empty, so it uses Shared from Step 1.

Until a queue is selected, Studio shows an error "Value for a required activity argument 'Queue name' was not supplied". That's expected on a fresh project.

## Step 4 · Pick the bot's browser (1 min)

`KillAllProcesses` force-closes the bot's browser when something goes wrong. If the bot used your everyday browser, it would close your own tabs too.

So use **Edge for the bot** if you browse in Chrome, or the other way round. Make sure the UiPath extension is installed in that browser, with **Allow access to file URLs** on.

## Step 5 · InitAllApplications.xaml (20 min)

Open `Framework\InitAllApplications.xaml`. Keep the existing log message. Add, below it:

1. **Get the app path.** Add **Get Asset** (Orchestrator activities):
   - Asset name: `in_Config("LegacyApp_PathAsset").ToString`
   - Folder path: `in_Config("OrchestratorFolder").ToString`
   - Output: create variable `strAppPath` (String)

2. **Get the credentials.** Add **Get Credential**:
   - Asset name: `in_Config("LegacyApp_CredentialAsset").ToString`
   - Folder path: `in_Config("OrchestratorFolder").ToString`
   - Username: create variable `strUsername` (String)
   - Password: create variable `secPassword` (SecureString)

3. **Open the app.** Add **Use Application/Browser**:
   - First open `claims.html` in the bot's browser, so Studio can see it.
   - Click **Indicate application to automate** and click the mock app window.
   - In Properties, set **Browser URL** to:
     `"file:///" + strAppPath.Replace("\", "/")`
   - **Open:** `IfNotOpen` · **Close:** `Never` (the app must stay open for the Process state)

4. **Sign on**, inside the Use Application/Browser:
   - **Type Into** → indicate the **User ID** box → text `strUsername`
   - **Type Into** → indicate the **Password** box → in Properties use **Secure Text** = `secPassword`
   - **Click** → indicate **Sign On**

5. **Confirm it worked.** Add **Check App State** → indicate the **Claim Number** field on the New Claim screen.
   - Target appears: **Log Message** (Info): `"Signed on to legacy claims app"`
   - Target does not appear: **Throw** `New System.Exception("Sign-on to legacy claims app failed")`
   Throwing a *system* exception here is deliberate: REFramework then re-initialises and retries.

**Object Repository:** as you indicate each element, add it with a friendly name:
`Screen_Login: UserID, Password, SignOn` · `Screen_NewClaim: ClaimNumber`.
The app's real IDs are cryptic (`f1`, `txtUser`) on purpose, as in many legacy systems. Friendly names in one repository are a strong interview point: one place to fix selectors when the legacy UI changes.

## Step 6 · CloseAllApplications.xaml (5 min)

Open `Framework\CloseAllApplications.xaml`. It has no `in_Config`, so it just attaches to the open app:
- **Use Application/Browser**, indicating the same app. **Open:** `Never` · **Close:** `Always`
- Inside: **Click** → **Sign Off** (add to the Object Repository as `Screen_NewClaim: SignOff`)

## Step 7 · KillAllProcesses.xaml (2 min)

Open `Framework\KillAllProcesses.xaml` (it ships empty):
- **Kill Process** → ProcessName `"msedge"` (or `"chrome"`, matching the bot's browser from Step 4)

## Step 8 · Run it (5 min)

1. Close the mock app if it's open.
2. Open `Main.xaml` and click **Debug File** (or Run).
3. Watch: the browser opens the app, signs on, finds the queue empty, signs off and closes.
4. In the **Output** panel you should see `Signed on to legacy claims app`, then a message that there are no more transactions.

**If it fails:**
- *Error code 1101* → no Orchestrator folder in scope: set Studio's folder to **Shared** (Step 1).
- *Error code 1002* → the queue isn't in that folder: check Step 3 and that `ClaimIntake` is in Shared.
- *Asset not found* → an asset name in `Config.json` doesn't match Orchestrator exactly.
- *Browser opens but nothing is typed* → the extension isn't enabled, or file URL access is off, for that browser.
- *Page not found* → check the `LegacyApp_Path` asset value has no quotes and points at `claims.html`.

## Step 9 · Commit (2 min)

```
git add .
git commit -m "Day 1: Performer signs on and off the legacy app via config, assets and Object Repository"
git push
```

**Done when:** Main runs cleanly end to end with an empty queue, and the commit is on GitHub.

Tomorrow (Day 2): `Process.xaml` enters a claim and reads the status message.
