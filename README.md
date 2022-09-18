# macOS Subsystem for Linux
Linux terminal/development environment running under macOS, as a proof of concept only.

## FAQ
I recently made a [tweet](https://twitter.com/DistroHopper39B/status/1571279732867534848) about this, and it went viral in an unprecedented way (I gained 100 followers in 12 hours, thanks for that!). Since then, I have received all sorts of questions about it, so here we go.

#### Q: What did you mean by "accidentally made?"
A: I was trying to make a Linux and macOS development environment for myself for some future projects (check out my GitHub profile), and was experimenting with QEMU virtual machines for this process. I discovered a feature that allowed for running QEMU headless with the output displayed in a Terminal window instead of having full VGA support. I immediately got sidetracked and... this is the result.
#### Q: OMG WTF HOW DID YOU DO THAT SHOUIHFREFSPWEJFWEW
A: Scroll down further. Although I will warn you, this is **very** unpolished.
#### Q: Why the same naming scheme as WSL?
A: It's a proof of concept name only. If I were to distribute this as a finished piece of software, which I probably won't (see why below) I would choose a different name.
#### Q: Isn't this literally just a QEMU virtual machine? How simple is that!
A: Yes, it is. And yes, it's simple. Why did I choose to put this on my Twitter, you ask? Because I couldn't find something exactly like what I did, and because before I posted it, [I had 50 Twitter followers, total.](https://socialblade.com/twitter/user/distrohopper39b) I did not expect to get the over 700 likes that it did when I posted it; as a matter of fact, I would have been perfectly satisfied if it had gotten 7 likes. 
#### Q: What, you won't be distributing this as a finished piece of software!? Why?
A: From what I can tell (and admittedly, I haven't tested this piece of software), it [already exists](https://github.com/lima-vm/lima). Lima seems to have a lot more features (such as file sharing, something that I have not implemented) and seems to be geared towards a different application (containers).
#### Q: You're an idiot.
A: Very true, but that's not a question.

## Process (Intel Mac)
*Note: this will work on Apple Silicon with slight modifications to the `qemu` commands, although I haven't tested this since I don't own one.*
1. Install [Homebrew](https://brew.sh).
2. Install QEMU: `brew install qemu`
3. Download the 64-bit Debian 11 installation image from https://debian.org/download. Other distributions should work as well, but I have only tested Debian (and these instructions are Debian-centric).
4. Create a new directory for running the Linux VM out of: `mkdir msl && cd msl` (This could be any directory you want, of course.)
5. Create the Linux system disk. 40GB should be fine, but you can make it as large or as small as you want (it will dynamically allocate space). To do this, type the following:
```
qemu-img create -f qcow2 msl.qcow2 40G
```
6. Start the VM with the following command:
```
qemu-system-x86_64 -accel hvf \
-m 4096 \
-hda msl.qcow2 \
-cdrom <path to debian iso> \
-boot d \
-smp 2 \
-net nic \
-net user,hostfwd=tcp::4022-:22 \
```
*Note: to adjust the RAM and CPU core amount, adjust the `-m` and `-smp` values.*

7. Start the VM.
8. Set up the VM as normal, making sure to a) leave the root password blank in order to allow for sudo access; b) make sure that the "username" of Debian is the same as your macOS username (not strictly necessary, but recommended); and c) uncheck all desktop environments and check the "SSH server" option.
9. The VM will reboot back to the installer once the installation is completed. When it does, close the QEMU window.
10. Restart the VM with `-boot d` changed to `-boot c` and the `-cdrom` line removed. For example:
```
qemu-system-x86_64 -accel hvf \
-m 4096 \
-hda msl.qcow2 \
-boot c \
-smp 2 \
-net nic \
-net user,hostfwd=tcp::4022-:22 \
```
11. Upon rebooting into Debian, log in and type `sudo nano /etc/default/grub` to edit the GRUB configuration file.
12. Set `GRUB_TIMEOUT` to `0`.
13. Set `GRUB_CMDLINE_LINUX_DEFAULT` to `"quiet console=ttyS0"`. *This is an S, the letter, not a 5.*
14. Add the following line: `GRUB_SERIAL_COMMAND="serial"`
15. Uncomment `GRUB_TERMINAL=console` and change it to `GRUB_TERMINAL="serial console"` (the quotes are important here).
16. Press Ctrl-X, then Y, then Return.
17. Run `sudo update-grub`. If you see any errors, make sure that all text is in quotes.
18. Run `sudo shutdown -h now`.
19. Rerun the VM, but add `-nographic` to the command line arguments. For example:
```
qemu-system-x86_64 -accel hvf \
-m 4096 \
-hda msl.qcow2 \
-boot c \
-smp 2 \
-net nic \
-net user,hostfwd=tcp::4022-:22 \
-nographic
```

*Note: This is the command you will use to launch the VM, so it's a good idea to put this in a script if you're launching it often.*

20. Restart the VM, and you should now see the GRUB screen appear, followed by the kernel boot. If it just hangs at `Welcome to GRUB`, or at `Loading initial ramdisk...` that probably means that you didn't run `sudo update-grub` or didn't add all the required lines to `/etc/default/grub`. 
21. That's about it! The concept has been proven.

## Bonus: Graphical applications with X11 forwarding and XQuartz
1. Shut down the VM with `sudo shutdown -h now`.
2. Install [XQuartz](http://xquartz.org), log out, and log back in.
3. Restart the VM in another terminal.
4. Open XQuartz.
5. Open a new terminal tab.
6. Run `ssh -X -p 3022 localhost`. Type `yes`, and enter your Debian password.
7. Install any graphical application; e.g. `vlc`.
8. Run the application from the terminal.
