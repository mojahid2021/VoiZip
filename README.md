# VoiZip

VoiZip puts your SIM cards and SIP accounts side by side. Dial once, choose the line, and talk on one call screen.

- Android 9 or newer
- Free software under the GNU GPL v3
- Download: [latest release](https://github.com/mojahid2021/VoiZip/releases/latest)

## Releases

The VoiZip website builds its download page and changelog from this repo's GitHub Releases. To publish a version:

1. Add the version to the top of `CHANGELOG.md`.
2. Create a release with tag `v<version>` (for example `v1.1.0`) and attach the signed APK named `VoiZip-<version>.apk`.
3. Write the notes in this format, which the website parses:

   ```markdown
   One-line summary of the release.

   ### Group title
   - Change
   - Change
   ```

The website picks up a new release within an hour. The newest non-draft, non-prerelease release is shown as Latest,
and its APK, size and SHA-256 checksum appear on the download page.
