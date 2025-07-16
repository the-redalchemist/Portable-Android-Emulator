# Portable-Android-Emulator

Great choice — QEMU is powerful, scriptable, and more portable than VirtualBox. Let’s walk through how to:


---

✅ Build a Redistributable Python App Using QEMU + Android x86 + uiautomator2


---

🧱 High-Level Architecture

MyApp/
├── android_x86.qcow2        # Android x86 disk image
├── qemu/                    # Portable QEMU binaries
│   └── qemu-system-x86_64.exe
├── adb/                     # ADB + uiautomator2 APKs
│   ├── adb.exe
│   ├── atx-agent.apk
│   └── uiautomator.apk
├── scripts/
│   └── launch_android.py    # QEMU + ADB + uiautomator2 controller
└── dist/
    └── myapp.exe            # After PyInstaller


---

🛠️ Step-by-Step Setup


---

1. ✅ Download & Prepare Android x86 Image

Download the ISO: https://www.android-x86.org/download.html

Use QEMU to install Android x86 into a .qcow2 disk:

qemu-img create -f qcow2 android_x86.qcow2 8G

qemu-system-x86_64 \
  -m 2048 -enable-kvm -smp 2 \
  -cdrom android-x86_64.iso \
  -hda android_x86.qcow2 \
  -boot d

Go through the installation steps inside Android.

Enable Developer Options and ADB over network (usually under Settings > Developer Options).



---

2. ✅ Set Up Port Forwarding (for ADB and UIA)

Configure QEMU networking to expose Android’s ADB port to the host:


qemu-system-x86_64 \
  -m 2048 -enable-kvm -smp 2 \
  -hda android_x86.qcow2 \
  -net nic -net user,hostfwd=tcp::5555-:5555 \
  -nographic

> Port forwarding allows adb connect 127.0.0.1:5555 to reach the guest Android.




---

3. ✅ Install uiautomator2 Inside the Android VM

From host:

adb connect 127.0.0.1:5555
adb install -r adb/atx-agent.apk
adb install -r adb/uiautomator.apk

# Optional: Auto-start at boot using `am start` or shell script
adb shell am start -n com.github.uiautomator/.MainActivity


---

4. ✅ Python Controller Script

import subprocess
import time
import uiautomator2 as u2

# Start QEMU
qemu_cmd = [
    "./qemu/qemu-system-x86_64",
    "-m", "2048",
    "-smp", "2",
    "-hda", "android_x86.qcow2",
    "-net", "nic",
    "-net", "user,hostfwd=tcp::5555-:5555",
    "-nographic"
]
qemu_proc = subprocess.Popen(qemu_cmd)

# Wait for Android to boot (tune as needed)
print("Waiting for Android to boot...")
time.sleep(30)

# Connect ADB
subprocess.run(["./adb/adb", "connect", "127.0.0.1:5555"])

# Connect UIAutomator2
d = u2.connect("127.0.0.1")
print(d.info)
d.app_start("com.android.settings")

# Optional: shut down VM later
# qemu_proc.terminate()


---

5. ✅ Package It All with PyInstaller

pyinstaller --onefile --add-data "qemu/*;qemu" --add-data "adb/*;adb" scripts/launch_android.py


---

✅ Bonus Features (Optional)

Add health checks to adb shell getprop sys.boot_completed

Support reboots or snapshot restore (via qemu-img snapshot)

Headless window or GUI toggle (via -display none vs SDL/GTK)



---

🚀 Want a Starter Kit?

I can generate a full ready-to-run QEMU Android project, including:

Example qemu-system-x86_64 binary (Windows/Linux)

Clean Android x86 qcow2 setup

ADB and uiautomator2 APKs

Python launcher script

PyInstaller config


Just tell me:

1. Your target OS for distribution (Windows/Linux/macOS?)


2. Do you want headless QEMU or visible window?


3. Any specific Android x86 version preferred?



Once you confirm, I’ll zip it into a proper starter bundle layout for you.


--------------------

Perfect — since both you and your users are on Linux, you can avoid most of the Windows-specific headaches and make this setup very smooth and fully self-contained. Here's the complete guide to what you need, plus an offer to generate a ready-to-run starter kit.


---

✅ Goal Recap

You're building a Linux Python app that:

Uses a QEMU-based Android x86 VM

Connects to it via ADB

Controls it via uiautomator2

Can be distributed as a single package that runs without extra installs



---

🧱 Project Structure for Packaging

myapp/
├── android_x86.qcow2          # Android VM disk image
├── qemu/                      # QEMU portable binaries
│   ├── qemu-system-x86_64
│   └── ...
├── adb/                       # ADB binaries + APKs
│   ├── adb
│   ├── uiautomator.apk
│   └── atx-agent.apk
├── launcher.py                # Python controller script
├── requirements.txt
└── build.sh                   # PyInstaller build script


---

🔧 Steps to Build and Use

1. ✅ Get QEMU and Android x86

Install QEMU locally for testing:

sudo apt install qemu-system-x86

Download and install Android x86 into a .qcow2 image:

qemu-img create -f qcow2 android_x86.qcow2 8G
qemu-system-x86_64 -enable-kvm -m 2048 -smp 2 \
  -cdrom android-x86_64.iso \
  -hda android_x86.qcow2 -boot d

During Android install:

Enable Developer options

Enable ADB over network

(Optional) Auto-start atx-agent on boot




---

2. ✅ Python Launcher Script

import subprocess
import time
import uiautomator2 as u2
import os

BASE = os.path.dirname(__file__)

QEMU_BIN = os.path.join(BASE, "qemu", "qemu-system-x86_64")
DISK_IMG = os.path.join(BASE, "android_x86.qcow2")
ADB_BIN = os.path.join(BASE, "adb", "adb")

def start_qemu():
    return subprocess.Popen([
        QEMU_BIN,
        "-enable-kvm",
        "-m", "2048",
        "-smp", "2",
        "-hda", DISK_IMG,
        "-net", "nic",
        "-net", "user,hostfwd=tcp::5555-:5555",
        "-nographic"
    ])

def connect_adb():
    subprocess.run([ADB_BIN, "connect", "127.0.0.1:5555"], check=True)

def setup_uiautomator():
    d = u2.connect("127.0.0.1")
    d.app_start("com.android.settings")
    print(d.info)

if __name__ == "__main__":
    proc = start_qemu()
    print("Waiting for Android to boot...")
    time.sleep(45)

    connect_adb()
    setup_uiautomator()

    input("Press Enter to stop VM...")
    proc.terminate()


---

3. ✅ PyInstaller Packaging

Install:

pip install pyinstaller

Create requirements.txt:

uiautomator2

Build:

pyinstaller --onefile --add-data "android_x86.qcow2:." \
            --add-data "qemu/*:qemu" \
            --add-data "adb/*:adb" launcher.py



---

🚀 Next Step — Want a Starter Project?

I can generate and send you a complete pre-filled project folder with:

✅ Prewritten launcher.py

✅ QEMU Linux binaries (statically linked, if needed)

✅ ADB + uiautomator2 APKs

✅ Proper folder structure

✅ PyInstaller spec file and build.sh

✅ (Optional) Prebooted Android x86 .qcow2 with adb + uiautomator2 ready


All you’ll need to do is:

cd myapp
chmod +x build.sh
./build.sh
./dist/myapp


---

Would you like me to assemble this as a downloadable ZIP or just generate all the files here in chat so you can run it locally?

Also:

1. Which Android x86 version do you want? (e.g., 9.0 Pie, 8.1, etc.)


2. Should it boot headless by default or with a visible QEMU window for debugging?



