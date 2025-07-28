Great progress — if QEMU is running but no graphical window appears, here are the most likely causes and how to fix them:


---

✅ Problem: QEMU GUI not showing inside Docker

Likely Causes:

1. You're not forwarding X11 or Wayland into the container


2. QEMU is running in -nographic or headless mode


3. You're on Wayland (e.g. Ubuntu 22+, Fedora) and need extra setup


4. Tails or security-focused OS is blocking X11/GUI access




---

🛠️ Solutions


---

✅ 1. Remove -nographic / -display none if set

Make sure your QEMU launch command doesn't include:

-nographic
-display none

Instead, use the default (GTK or SDL window):

qemu-system-x86_64 -enable-kvm -m 2048 -hda /android/android.qcow2


---

✅ 2. Run Docker with X11 GUI Access (Linux)

On host:

xhost +local:docker

Then run Docker with:

docker run --rm -it \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --device /dev/kvm \
  --cap-add=SYS_ADMIN \
  --security-opt seccomp=unconfined \
  your-qemu-image

This allows GUI apps from the container to use your host's X11 display.


---

✅ 3. Use GTK/QEMU GUI Display

Inside the container, install QEMU with GTK support:

apt update && apt install -y qemu-system-gui

Then run QEMU with GTK display:

qemu-system-x86_64 -enable-kvm -m 2048 -hda /android/android.qcow2 -display gtk


---

✅ 4. Wayland/Flatpak Users (Optional Advanced)

If your host uses Wayland, and xhost doesn't work, try using X11 compatibility mode, or run the host terminal with env GDK_BACKEND=x11.

Tails, for example, uses Wayland and AppArmor, which might prevent X11 access even with xhost.


---

🧪 Test GUI Access First

Run this on your host and from inside the container:

xclock

If xclock shows up inside the container, QEMU GUI will work too.

If not, the GUI is still blocked — fix that first.


---

✅ Alternative: VNC Display (Cross-Platform)

To make this universal (even for Tails), run QEMU with a built-in VNC server:

qemu-system-x86_64 -enable-kvm -m 2048 -hda /android/android.qcow2 -display vnc=:1

Then on host:

vncviewer :1

Or connect with a GUI VNC client.


---

✅ Summary

Issue	Fix

No GUI	Remove -nographic, add -display gtk
Host display not forwarded	Use -e DISPLAY and -v /tmp/.X11-unix
Running on Wayland	Try GDK_BACKEND=x11 or VNC fallback
Want portability	Use -display vnc=:1 and provide viewer instructions



---

Would you like me to help generate a Dockerfile + entrypoint that:

Detects KVM

Uses GTK by default

Falls back to VNC if no X11

Works with one-line run script?


Let me know!

