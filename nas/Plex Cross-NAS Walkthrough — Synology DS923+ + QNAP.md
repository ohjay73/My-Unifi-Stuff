# Plex Cross-NAS Walkthrough — Synology DS923+ + QNAP

Plex Media Server runs on the QNAP. This walkthrough makes that one Plex server see the
Synology's `SYN_Movies` folder too, so **Movies**, **Kids**, **TV Shows**, and **SYN_Movies**
all show up in the same library, from the same server.

```
Synology DS923+ (192.168.2.211)          QNAP (192.168.2.90)            Your TVs & apps
   [ SYN_Movies ]         ----->   Movies / Kids / TV Shows     --->   One Plex,
                                     + SYN_Movies (mounted)            everything
```

The QNAP mounts the Synology folder over the home network (SMB). Files stay on the
Synology — nothing is copied. Nothing in this setup opens your NAS to the internet; it's
all inside your home network (192.168.2.x).

**Time:** about 15–20 minutes.
**You need:** admin logins for both boxes, both on the same network (they are:
`192.168.2.90` and `192.168.2.211`).
Do the steps in order — Synology first, then QNAP, then Plex.

## Before you start — 2-minute checklist

- [ ] Both NAS boxes powered on and reachable: try `192.168.2.211` and `192.168.2.90` in a browser.
- [ ] You know the exact folder name on the Synology: `SYN_Movies` (capitals and underscore matter).
- [ ] Wired between the boxes is best — Plex streams *through* the QNAP, so Wi-Fi between NAS boxes can stutter.

*DSM = Synology screens, QTS = QNAP screens (DSM 7 / QTS 5 wording; search the same words if yours differ).*

## Step 1 — On the Synology: share `SYN_Movies`

Open DSM at `192.168.2.211` and sign in as admin. Three small things: turn on SMB (the
sharing language both boxes speak), confirm `SYN_Movies` is a shared folder, and give one
user read access.

### 1. Turn on SMB

1. Open **Control Panel** → **File Services** → **SMB** tab.
2. Tick **Enable SMB service** → **Apply**.

### 2. Confirm the shared folder

1. **Control Panel** → **Shared Folder**. You should see `SYN_Movies` in the list.
2. If it is not there: **Create** → name it exactly `SYN_Movies` → finish the wizard.
   (If your folder has a slightly different name, use your real name everywhere this guide says `SYN_Movies`.)

### 3. Create a read-only sharing user (recommended)

1. **Control Panel** → **User & Group** → **Create**: name `plexshare`, set a strong password, finish.
2. Select `SYN_Movies` → **Edit** → **Permissions** tab → give `plexshare` **Read only**.
   Also make sure your own admin account has at least Read.
3. Click **Save**.

Read-only is deliberate: Plex only needs to read and play files. It cannot delete or change
anything on the Synology this way. You can reuse your admin login instead, but the dedicated
user is safer and easier to fix later.

> **Write these down:** Synology IP `192.168.2.211` • folder `SYN_Movies` • user `plexshare`
> + its password. You type all four into the QNAP next.

## Step 2 — On the QNAP: mount `SYN_Movies`

Open QTS at `192.168.2.90` and sign in as admin. File Station can attach a folder from another
NAS so it appears as if it were a local QNAP folder. That attached folder is what Plex will use.

### 4. Open the mount tool

1. Open **File Station**.
2. Click the **Remote Mount** icon (globe / mount icon in the top toolbar) → **Create Remote Mount**.
   In some QTS versions: **File Station** → **Mount** → **Cloud / Remote mount**.
   Pick the SMB/CIFS option when asked.

### 5. Fill in the Synology details exactly

| Field | Enter this |
|---|---|
| Protocol | **CIFS / SMB** |
| Server / IP address | `192.168.2.211` |
| Shared folder / remote path | `SYN_Movies` (or `/SYN_Movies` if it asks for a path) |
| Username | `plexshare` |
| Password | the password you just set |
| Mount / display name | `SYN_Movies` |

1. Click **Create / Connect**. Wait for "connected" / green status.
2. In File Station's left sidebar you should now see `SYN_Movies`. Open it and confirm your movie files are listed.

> **If it fails here, do not keep guessing.** The three usual causes, in order: wrong password,
> SMB not enabled on the Synology (Step 1.1), or folder name typed differently. Fix the one
> thing, then retry. The full checklist is at the bottom.

## Step 3 — Point Plex at all four folders

Nothing to install on the Synology. Open Plex on the QNAP (`192.168.2.90:32400/web`) and sign in.

### 6. Check the three QNAP libraries you already have

1. In Plex, hover **Movies** → **…** → **Manage Library** → **Edit** → **Add folders**: it should
   point at the QNAP Movies folder. Leave it as is.
2. Repeat for **Kids** and **TV Shows**. No changes needed if they play today.

### 7. Add `SYN_Movies` — pick one of these

**Option A — Add it to your existing Movies library (most people want this).**
All movies in one place, regardless of which box they sit on.

1. **Movies** → **…** → **Manage Library** → **Edit** → **Add folders** → **Browse for Media Folder**.
2. Navigate to the mounted `SYN_Movies` folder (look under the remote mount name from Step 5) → **Add** → **Save Changes**.

**Option B — Make it its own library.** Keeps Synology movies separate in the sidebar.

1. **+** next to Libraries → type **Movies** → name it `SYN_Movies` → **Next** → **Browse for Media Folder** → pick the mounted `SYN_Movies` → **Add Library**.

Option A is simpler to browse; Option B shows which box a movie lives on.

### 8. Let Plex scan

1. **Movies** (or **SYN_Movies**) → **…** → **Scan Library Files**.
2. Wait for the spinner to finish. Big libraries over a network mount can take a while the first time — that is normal.

## Step 4 — Test it like Plex will use it

- [ ] Play 30 seconds of a movie that lives in `SYN_Movies`, from the TV or app you normally use.
      Playing proves reading works — just seeing the poster does not.
- [ ] Play one from QNAP Movies, one Kids title, and one TV episode. All four sources, one Plex.
- [ ] Add one new file to `SYN_Movies` on the Synology, then **Scan Library Files** again in Plex.
      It should appear. That is your repeatable "add a movie" routine from now on.

> **Your new routine:** drop files into `SYN_Movies` on the Synology (or Movies / Kids / TV Shows
> on the QNAP) → Plex scans → it shows up everywhere. No copying between boxes, ever — the mount does the reaching.

### Optional: see the QNAP folders from the Synology side too

Only if you like managing files from the Synology screens. DSM: **File Station** → **Tools** →
**Mount Remote Folder** → **CIFS Shared Folder** → `\\192.168.2.90\Movies`
(repeat for Kids / TV Shows). Plex does not need this.

## Keep it working after reboots

### Turn these on once

- **Auto-scan:** Plex → **Settings** → **Library** → tick **Scan my library automatically** and
  **Run a partial scan when changes are detected**. Note: change-detection may not fire across a
  network mount, so a manual scan (Step 8) is still your reliable fallback.
- **Scheduled scan:** also set **Scan my library periodically** (e.g. every 6–12 hours) so new
  Synology files appear on their own.
- **Keep the mount:** in QNAP File Station's Remote Mount list, confirm `SYN_Movies` shows
  **Connected** and any "reconnect / auto-mount" option is on.

### Reboot order matters

1. Synology on first, give it 2–3 minutes.
2. Then the QNAP.
3. If Plex ever shows `SYN_Movies` as empty after a power cut: QNAP File Station →
   Remote Mount → reconnect `SYN_Movies` → Plex → Scan Library Files. Thirty seconds, no re-setup.

Tip: leave **Empty trash automatically** off in Plex Library settings until the mount is
boringly reliable, so a brief disconnect cannot wipe your library entries.

## Troubleshooting

| Symptom | Most likely cause | Fix, in order |
|---|---|---|
| Mount fails / "connection failed" | Password or SMB off | 1) Re-type the `plexshare` password. 2) DSM: SMB enabled (Step 1.1). 3) Confirm you can open `192.168.2.211` in a browser. |
| "Folder not found" | Name mismatch | Compare letter-by-letter with DSM Shared Folder list: `SYN_Movies`, underscore, capitals. Try `/SYN_Movies`. |
| Mount works, folder looks empty in Plex | Permissions | DSM → Shared Folder → `SYN_Movies` → Permissions: `plexshare` = Read only (Step 1.3). Then reconnect the mount. |
| Posters show, playback fails / stutters | Network path | Wire both NAS boxes if you can; avoid Wi-Fi between them. Test one smaller file to confirm. |
| Worked yesterday, gone today | Mount dropped on reboot | Reconnect in File Station (above), then Scan Library Files in Plex. |
| New files never appear | No scan triggered | Manual Scan Library Files; then enable periodic scan (left card). |

### Quick recap — the whole thing in 6 lines

1. Synology: SMB on, `SYN_Movies` shared, `plexshare` = Read only.
2. QNAP File Station: Remote Mount → CIFS/SMB → `192.168.2.211` / `SYN_Movies` / `plexshare`.
3. Open the mount in File Station — see your files.
4. Plex: add the mounted `SYN_Movies` to Movies (or as its own library).
5. Scan Library Files.
6. Play one file from each of the four folders. Done.

### Safety notes

- Keep both NAS admin passwords different from the `plexshare` password; read-only means Plex
  (and anyone who somehow got that password) cannot delete your Synology files.
- Nothing in this setup opens your NAS to the internet — it is all inside your home network
  (192.168.2.x).
