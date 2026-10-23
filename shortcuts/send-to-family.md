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
4. Tap **Search for apps and actions** → search **Drive** → choose the Google Drive action named **Save File** or **Upload**.
   - If no Google Drive action appears, use **Save File** (from Files) instead. Set the location to **Google Drive › Family Inbox** in the Files picker, and turn off **Ask Where to Save**.
5. Set the action's input to **Shortcut Input** and the destination folder to **Family Inbox**.
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
| Drive action missing | Open the Drive app once, then search actions again, or use the Files fallback in step 4. |
