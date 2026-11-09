# Zen for Claude: a second Zen that Claude can drive

This sets up a second copy of Zen next to your everyday one. It has its own profile, its own logins, and remote control turned on, so an AI assistant can open pages, click, read logs and take screenshots without touching your main browser.

Your main Zen stays untouched. Only the copy in `/Applications/Zen for Claude.app` is changed.

## What you get

- A second app with a black icon, named **Zen for Claude** in the Dock and the menu bar.
- Remote control on `localhost:2828` (Marionette), always on for this app only.
- A "Only Sidebar" layout, reopen-last-session on start, and no welcome screen.
- The same domain-only address and page-coloured toolbar as the mod in this repo (optional).

## Before you start

- macOS with Zen installed in `/Applications/Zen.app`.
- Quit Zen completely first (Cmd+Q).

## 1. Make the copy

```bash
cp -R /Applications/Zen.app "/Applications/Zen for Claude.app"
```

## 2. Give it its own identity

Change the bundle id so macOS treats it as a separate app:

```bash
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier app.zen-browser.zen.claude" "/Applications/Zen for Claude.app/Contents/Info.plist"
```

Set the names, so the Dock and menu bar say "Zen for Claude":

```bash
P="/Applications/Zen for Claude.app/Contents/Info.plist"
/usr/libexec/PlistBuddy -c "Set :CFBundleName 'Zen for Claude'" "$P"
/usr/libexec/PlistBuddy -c "Set :CFBundleDisplayName 'Zen for Claude'" "$P"
```

Zen also has a localized name file that overrides the bundle name. Without this step the menu bar still says "Zen":

```bash
F="/Applications/Zen for Claude.app/Contents/Resources/en.lproj/InfoPlist.strings"
T=$(mktemp -d)
plutil -convert xml1 -o "$T/s.xml" "$F"
sed -i '' 's|<string>Zen</string>|<string>Zen for Claude</string>|' "$T/s.xml"
plutil -convert binary1 -o "$F" "$T/s.xml"
```

## 3. Turn on remote control for this app only

Add these environment variables to the app's `Info.plist` so they apply only when it's launched:

```bash
P="/Applications/Zen for Claude.app/Contents/Info.plist"
/usr/libexec/PlistBuddy -c "Add :LSEnvironment dict" "$P"
/usr/libexec/PlistBuddy -c "Add :LSEnvironment:MOZ_MARIONETTE string 1" "$P"
/usr/libexec/PlistBuddy -c "Add :LSEnvironment:MOZ_REMOTE_ALLOW_SYSTEM_ACCESS string 1" "$P"
```

Your everyday Zen doesn't read these, so it's still a normal browser.

> **Security note:** remote control listens on `localhost:2828`. Any program on your Mac can reach it while Zen for Claude is open. Don't leave it open with logins you don't want exposed, and quit it when you're not using it.

## 4. Give it a separate profile and settings

Zen keeps each profile in `~/Library/Application Support/zen/Profiles/`. Create one for this app by opening Zen for Claude once (it creates a profile), quitting it, then adding a `user.js` file to that profile folder:

```js
// Zen for Claude: Claude's own Zen (automation always on via the app's LSEnvironment)
user_pref("zen.welcome-screen.seen", true);
user_pref("browser.aboutwelcome.enabled", false);
user_pref("browser.shell.checkDefaultBrowser", false);
user_pref("zen.view.use-single-toolbar", true);   // "Only Sidebar" layout
user_pref("zen.view.sidebar-expanded", true);
user_pref("zen.view.compact.enable-at-startup", false);
user_pref("browser.startup.page", 3);             // reopen the last session
```

The profile folder name has a random part, so find it with `ls ~/Library/Application\ Support/zen/Profiles/` and pick the one that appeared when you first opened Zen for Claude.

## 5. Change the icon (optional)

Put a black icon file into `Contents/Resources/firefox.icns` (the app's icon file). Then re-register the app so the Dock updates:

```bash
/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister -f "/Applications/Zen for Claude.app"
killall Dock
```

## 6. Re-sign the app

After changing the bundle, macOS needs a fresh signature:

```bash
codesign --force --deep --sign - "/Applications/Zen for Claude.app"
```

Then open it once from Finder to check the name and icon.

## Keeping it working after Zen updates

Zen updates replace `Zen.app` but not `Zen for Claude.app`, so this copy stays as you set it. The menu bar name and icon are not affected by a Zen update. If you also use the startup-script part of this repo, the `reapply.sh` idea in this repo's `startup-script/` can be pointed at the copy as well.

## Undo

```bash
rm -rf "/Applications/Zen for Claude.app"
```

Your profile data stays in `~/Library/Application Support/zen/Profiles/`; delete that folder too if you want it gone.
