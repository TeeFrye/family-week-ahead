# "Send to Family" — iOS Share Sheet Shortcut

Sends any screenshot, photo, or PDF to the **Family Inbox** Drive folder in one tap, from the share sheet in any app (ClassDojo, GroupMe, Photos, Messages, Safari).

**Before you start:** the Google Drive app is installed and signed in, and a folder named `Family Inbox` exists in My Drive.

## Build it (about 5 minutes)

1. Open **Shortcuts** → tap **+** (top right).
2. Tap the shortcut name at the top → **Rename** → `Send to Family`. Pick an icon if you want one.
3. Tap the **ⓘ** (info) button at the bottom → turn on **Show in Share Sheet** → tap **Done**.
   - A block appears at the top: *Receive **Any** input from **Share Sheet***.
   - Tap **Any** → deselect everything except **Images** and **PDFs** → tap **Done**.
   - *If there's no input* → set it to **Get Clipboard**. Then, if you copied a screenshot, running the shortcut from the widget still works.
4. Google Drive doesn't offer its own Shortcuts action. It saves through the **Files** app instead. Turn that on once:
   - Open the **Files** app → **Browse** → **•••** (top right) → **Edit** → turn on **Google Drive** → **Done**.
   - Tap **Google Drive** in Files to confirm you can see the `Family Inbox` folder.
5. Back in the shortcut: **Search for apps and actions** → **Save File** (the Files action).
   - Tap the arrow (›) on the action → turn off **Ask Where to Save**.
   - Tap the folder field → **Browse** → **Google Drive** → **Family Inbox** → **Open**.
   - Make sure the action is saving **Shortcut Input**.
6. *(Optional)* Add **Show Notification** → text `Sent to Family Inbox ✅`.
7. Tap **Done**.

## Use it
Take a screenshot → tap the thumbnail → **Share** (□↑) → **Send to Family**.

The first time, iOS asks for permission to use Drive. Tap **Always Allow**.

**Pin it:** in the share sheet, scroll the app row → **Edit Actions** → tap **+** next to *Send to Family* so it shows near the top.

## For text (no screenshot)
Long-press the message → **Copy** → paste it into a new email to your `+family` address. The Gmail filter labels it **Family Inbox**.

## Troubleshooting
| Symptom | Fix |
|---|---|
| Not in the share sheet | The ⓘ setting "Show in Share Sheet" is off, or the input types don't include Images. |
| Saves to the wrong place | Turn off "Ask Where to Save" and pick the folder again. |
| Google Drive not listed in Files | Open the Google Drive app once and sign in, then go back to Files → ••• → Edit. |
