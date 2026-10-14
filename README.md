# Kindle eBook workflow on Linux Mint with WinBoat

An interactive, version-aware workflow for creating a personal backup of
Kindle books you purchased or are otherwise authorized to process. It adapts
[Fabio Akita's 2026 article](https://akitaonrails.com/en/2026/07/30/removing-drm-from-kindle-ebooks-in-2026/)
for a Linux Mint host using [WinBoat](https://winboat.app/), rather than
Omarchy's Windows VM integration.

> DRM-circumvention rules vary by country. Confirm that you are authorized to
> process each book and that the workflow is lawful where you live. Amazon's
> app, encryption, and community tools change frequently; verify current
> compatibility before proceeding.

## Why this is separate from Linux Mint setup

`linux-mint-setup` can install Calibre, WinBoat, and create a private working
directory. The remaining steps require an interactive Windows Store sign-in,
WinBoat-specific shared-folder and TPM verification, a Kindle-app-version
matched archiver, and secure handling of account-derived key material. They
should not run unattended as part of a general Linux post-install script.

In particular, do not use the article's `omarchy-windows-vm` commands, its
`~/Windows` path, `Z:` drive, `\\host.lan\\Data` path, or Docker Compose
`TPM: "Y"` example with WinBoat. Those are Omarchy-specific.

## Prerequisites

- Linux Mint with [Calibre](https://calibre-ebook.com/download_linux) installed.
- WinBoat with a working Windows guest. Its local data may live under
  `/mnt/data/winboat`, but do not use that VM-data directory as your document
  workspace.
- A shared host folder configured through WinBoat's own UI or documentation.
  Confirm a harmless text file can move in both directions before using it.
- Your purchased Kindle books available to the Amazon account you will use in
  the Windows guest.

This setup uses an existing WinBoat share at
`/mnt/data/winboat/shared/DeDRM_tools`; `oldbooks.k4i` is generated directly
inside it. Configure that path in `linux-mint-setup/config.sh`:

```bash
KINDLE_DEDRM_DIR="/mnt/data/winboat/shared/DeDRM_tools"
KINDLE_DEDRM_BACKUP_DIR="/mnt/data/nextcloud/Documents/Credentials/Kindle-DeDRM"
```

The shared-folder mapping is already in place. Avoid editing WinBoat's VM-data
directory or substituting Omarchy's paths for this share.

## Workflow

1. In Calibre, install the current `DeDRM_plugin.zip` from the current
   pre-release of [Satsuoni's maintained fork](https://github.com/Satsuoni/DeDRM_tools),
   then install **KFX Input** through Calibre's “Get new plugins” screen.
   Keep the inner `DeDRM_plugin.zip` compressed. Record the plugin versions.
2. In WinBoat's Windows guest, install **Amazon Kindle: Reading App** from the
   Microsoft Store—not the legacy Kindle for PC installer. Sign in and download
   each authorized title for offline use; seeing it in the library is not
   enough.
3. Before processing a library, check that the guest exposes the TPM-related
   behavior needed by the current Kindle app and archiver. If it does not,
   consult WinBoat's current documentation; do not change VM internals based
   on instructions written for another launcher.
4. From the DeDRM release matching the installed Kindle app, select the
   `MSIXKFXArchiverMobi1_XXXXX.exe` whose suffix matches that app version.
   Copy it to the Windows-visible `tools` directory and run it from PowerShell.
   The release notes—not an old tutorial—are the source of truth.
5. The archiver should produce `archived_kfx/` and `oldbooks.k4i`. In this
   setup they are available in `/mnt/data/winboat/shared/DeDRM_tools`. The
   companion setup backs up `oldbooks.k4i`, `books.keyfile`, and
   `storage/.kinf2024` to `/mnt/data/nextcloud/Documents/Credentials/Kindle-DeDRM`.
   Never commit, publish, or screenshot them:

   ```bash
   chmod 600 /mnt/data/winboat/shared/DeDRM_tools/oldbooks.k4i
   ```

   If the archiver reports a missing `MSVCP140.dll`, install the official
   Microsoft Visual C++ Redistributable that matches its architecture. Do not
   download individual DLL files from third-party sites.
6. In Calibre, use **Preferences → Plugins → DeDRM → Customize plugin → Kindle
   for Mac/PC ebooks → Import Existing Keyfiles**. Select `oldbooks.k4i`, close
   the key list, click **OK** in the main dialog, and restart Calibre.
7. Import one `.kfx-zip` first. A successful import becomes `KFX`; if it stays
   `KFX-ZIP`, remove that library entry, correct the plugin/key/version issue,
   and import again. Converting it repeatedly will not decrypt it because
   DeDRM runs when the book is added.
8. Open the KFX, convert it to EPUB, and inspect the cover, table of contents,
   images, layout, and several pages in a non-Kindle reader before batch work.
9. Preserve the verified EPUBs, the original KFX-ZIP packages until validation
   is complete, and an encrypted backup of `oldbooks.k4i` stored separately.
   Do not delete downloaded Windows/VM content until the backups have been
   tested.

## Troubleshooting

| Symptom | Check first |
| --- | --- |
| Kindle app is not found | Use the Microsoft Store app, not legacy Kindle for PC. |
| Title is visible but no archive is produced | Open it in the app and let the offline download finish. |
| Archiver cannot access the app | Match the archiver to the installed app version and re-check WinBoat/TPM support. |
| Linux cannot see output | Verify the actual WinBoat shared-folder mapping and the archiver's working directory. |
| Calibre retains `KFX-ZIP` | Re-check the key import location, plugin versions, restart Calibre, then re-import. |
| EPUB conversion fails | First establish that the import created `KFX`; conversion is a later stage. |

## Security checklist

- Keep `oldbooks.k4i` mode `0600` and out of cloud-synced/public folders.
- Keep `books.keyfile` and `.kinf2024` private too when the current toolchain
  creates them.
- Never commit raw key material, even to a private repository.
- Treat shared-folder contents as sensitive while the Windows guest is running.
- Back up the key only in encrypted form, with the decryption key stored
  separately.

## References

- Fabio Akita, [Removing DRM from Kindle eBooks in 2026](https://akitaonrails.com/en/2026/07/30/removing-drm-from-kindle-ebooks-in-2026/)
- [Satsuoni's DeDRM tools releases](https://github.com/Satsuoni/DeDRM_tools/releases)
- [Calibre](https://calibre-ebook.com/)
- [KFX Input](https://www.mobileread.com/forums/showthread.php?t=291290)
