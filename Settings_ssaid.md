You're absolutely on the right track — the settings_ssaid.xml file stores the SSAID (Settings.Secure.ANDROID_ID), which is per-app-per-user starting Android 8+, and it's the primary identifier many apps (especially Firebase-based) rely on.

You're right that:

Changing it via an app manually works, but isn't scalable.

adb shell settings put secure android_id ... does not work post-Android 8+, because Android ignores direct SSAID writes due to security model changes.


So let's walk through better, automatable ways to change this ID.


---

✅ GOAL: Automate SSAID (ANDROID_ID) Spoofing

Here are viable automatable approaches depending on your access level:


---

✅ Option 1: Direct File Injection via ADB + Root

Since settings_ssaid.xml is typically located at:

/data/system/users/0/settings_ssaid.xml

You can edit and replace this file using adb and root:

# Pull, modify, push method
adb root
adb remount

adb pull /data/system/users/0/settings_ssaid.xml

# Now modify the XML locally using Python or sed
# Example: change android_id value inside the file
sed -i 's/android_id" value=".*"/android_id" value="RANDOM_ID_HERE"/' settings_ssaid.xml

# Push it back
adb push settings_ssaid.xml /data/system/users/0/settings_ssaid.xml

# Set correct permissions
adb shell chmod 660 /data/system/users/0/settings_ssaid.xml
adb shell chown system:system /data/system/users/0/settings_ssaid.xml

# Reboot to apply
adb reboot

✅ Automatable. You can generate random SSAIDs each time with a script and fully automate it.


---

✅ Option 2: Modify in Memory via Magisk Module (Advanced)

If you're using Magisk, you can hook or override the call to Settings.Secure.ANDROID_ID:

Use XPrivacyLua, LSPosed, or write a custom Magisk module or Riru module that intercepts Settings.Secure.getString() for ANDROID_ID.


Or:

Use MagiskHidePropsConfig + resetprop to spoof a consistent Android ID across the system.


resetprop persist.service.adb.enable 1
resetprop persist.sys.android_id <your_new_random_id>

> ⚠️ Only works if the app accepts global spoofing and doesn’t double-check SSAID from settings_ssaid.xml.




---

✅ Option 3: Boot Fresh Emulator Images with Random SSAID

If you’re using QEMU-based Android images:

1. Prepare a base AOSP image.


2. Script qemu-img create -f qcow2 -b base.img fresh.img


3. Boot fresh image, push app, randomize SSAID using method above.


4. Use automation tool (e.g., uiautomator2) to perform login.


5. Destroy image and repeat.



✅ Fully stateless automation.


---

✅ Option 4: Use a Custom App to Set SSAID Programmatically

You mentioned doing it via an app — to automate:

Build a minimal APK that runs on startup and sets a random SSAID in /data/system/users/0/settings_ssaid.xml

Have it su and rewrite the file during boot, then soft-reboot.



---

✅ Bonus: Script Generator (Python + ADB)

Here’s a quick Python function to automate the SSAID rewrite:

import random
import subprocess

def generate_random_ssaid():
    return ''.join(random.choices('0123456789abcdef', k=16))

def replace_ssaid(ssaid):
    subprocess.run(["adb", "root"])
    subprocess.run(["adb", "remount"])
    subprocess.run(["adb", "pull", "/data/system/users/0/settings_ssaid.xml", "ssaid.xml"])

    with open("ssaid.xml", "r") as f:
        content = f.read()
    content = re.sub(r'android_id" value="[^"]+"', f'android_id" value="{ssaid}"', content)
    with open("ssaid.xml", "w") as f:
        f.write(content)

    subprocess.run(["adb", "push", "ssaid.xml", "/data/system/users/0/settings_ssaid.xml"])
    subprocess.run(["adb", "shell", "chmod", "660", "/data/system/users/0/settings_ssaid.xml"])
    subprocess.run(["adb", "shell", "chown", "system:system", "/data/system/users/0/settings_ssaid.xml"])
    subprocess.run(["adb", "reboot"])

replace_ssaid(generate_random_ssaid())


---

🔐 TL;DR

Method	Root Required	Fully Automatable	Notes

Modify settings_ssaid.xml	✅ Yes	✅ Yes	Best method
Use XPrivacyLua or Magisk	✅ Yes	✅ Yes	For deep spoofing
Use new emulator image	❌ Optional	✅ Yes	Stateless but slower
adb settings put secure	❌ No	❌ No	Ignored on Android 8+



---

Let me know your exact emulator base (AOSP, Android version, rooted or not) and I can write a full script tailored to it.

