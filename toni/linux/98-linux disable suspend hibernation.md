https://www.tecmint.com/disable-suspend-and-hibernation-in-linux/

Suspending your system helps save power when you are not using your system. Getting back to using your system requires a simple mouse-click or a tap on any keyboard button. Sometimes, you may be required to press the power button.

There are 3 suspend modes in Linux:

- Suspend to RAM (Normal Suspend): This is the mode that most laptops automatically enter incase of inactivity over a certain duration or upon closing the lid when the PC is running on the battery. In this mode, power is reserved for the RAM and is cut from most components.
- Suspend to Disk (Hibernate): In this mode, the machine state is saved into swap space & the system is completely powered off. However, upon turning it on, everything is restored and you pick up from where you left.
- Suspend to both (Hybrid suspend): Here, the machine state is saved into swap, but the system does not go off. Instead, the PC is suspended to RAM. The battery is not used and you can safely resume the system from the disk and get ahead with your work. This method is much slower than suspending to RAM.

Disable Suspend and Hibernation in Linux
To prevent your Linux system from suspending or going into hibernation, you need to disable the following systemd targets:
```
$ sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```
You get the output shown below:

```
Created symlink /etc/systemd/system/sleep.target → /dev/null.
Created symlink /etc/systemd/system/suspend.target → /dev/null.
Created symlink /etc/systemd/system/hibernate.target → /dev/null.
Created symlink /etc/systemd/system/hybrid-sleep.target → /dev/null.
```
![Disable-Suspend-and-Hibernation-in-Ubuntu](https://github.com/user-attachments/assets/1abbfdb2-50ea-4f2e-a3c4-9e45fc8f0141)

Then reboot the system and log in again.

Verify if the changes have been effected using the command:
```
$ sudo systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
```
![Verify-Suspend-and-Hibernation-in-Ubuntu](https://github.com/user-attachments/assets/a4730e0b-c2d6-42fd-8a3b-c980a6ee3095)

Enable Suspend and Hibernation in Linux
To re-enable the suspend and hibernation modes, run the command:
```
$ sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```
Here’s the output that you will get.
![Enable-Suspend-and-Hibernation-in-Ubuntu-768x226](https://github.com/user-attachments/assets/bbb20aa6-d64f-4aea-b9a9-4a415f5cce7d)

To verify this, run the command;
```
$ sudo systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
```
![Check-Suspend-and-Hibernation-in-Ubuntu-768x478](https://github.com/user-attachments/assets/5d60ab04-5af6-4c3b-b9ae-0d081ee58497)

To prevent the system from going into suspend state upon closing the lid, edit the /etc/systemd/logind.conf file.
```
$ sudo vim /etc/systemd/logind.conf
```
Append the following lines to the file.
```
[Login] 
HandleLidSwitch=ignore 
HandleLidSwitchDocked=ignore
```
Save and exit the file. Be sure to reboot in order for the changes to take effect.

This wraps our article on how to disable Suspend and hibernation modes on your Linux system. It’s our hope that you found this guide beneficial. Your feedback is most welcome.
