---
title: "Upgrading Old Windows Computers to Windows 11"
date: 2025-09-01T13:00:00-07:00
type: post
author: James
categories:
  - Computer Upgrades
tags:
  - Mini Desktop Computer
  - Upgrade
  - Windows 11
comments: true
---

Windows 10 goes End of Life (EOL) on October 14, 2025 meaning they won't
provide updates for function or security without a paid extended support plan. 
Microsoft wants everybody to upgrade
to Windows 11 but the machine requirements are stringent
and some older computers will not automatically upgrade. 
I will discuss alternatives as well as my experience with the upgrade.

There are several alternatives:

#### 1. Do nothing, keep Windows 10

**Pros**
  * No effort or expense

**Cons**
  * May get some security issues or viruses due to lack of updates.
  * Probably won't work with some programs such as [TurboTax 2025](https://ttlc.intuit.com/turbotax-support/en-us/help-article/download-products/end-support-windows-8-affect-turbotax-experience/L4v9atO3O_US_en_US?uid=mf1kwbok "Turbo Tax 2025").

#### 2. Upgrade to Windows 11

**Pros**
  * Solves the Cons from #1.
  * Some methods to do it even if the processor is not supported by Windows 11.

**Cons**
  * May not be possible, especially if the machine is older than 5 years.
  * Must get used to some differences in the UI.

#### 3. Pay Microsoft for extended support for Windows 10.

**Pros**
  * Gets the security updates to keep it running another year.
  * Not too expensive ($30 for home users)

**Cons**
  * Probably won't work with some programs such as [TurboTax 2025](https://ttlc.intuit.com/turbotax-support/en-us/help-article/download-products/end-support-windows-8-affect-turbotax-experience/L4v9atO3O_US_en_US?uid=mf1kwbok "Turbo Tax 2025").
  * Eventually will have to upgrade.

#### 4. Replace with a new computer with Windows 11 preinstalled.

**Pros**
  * Solves the Cons from #1.
  * Computer likely runs more efficiently than old one.

**Cons**
  * Will cost more than other options.


## My Experience With Upgrading Windows 10 to Windows 11

#### Relatively Modern Laptop

I took the route of option #2 (upgrading to Windows 11) for an HP
laptop that was less than five years old. Although it was modern enough to upgrade
the Windows installer didn't like the processor version.  Basically I used a helper
script called FlyBy11 that would bypass the processor check and allow the upgrade. So far
there is no issue in doing this.

To identify a problem with a Windows 11 upgrade you can run an app called PC Health Check
and it will identify the exact reason that an upgrade can't easily be done:
[How to use PC Health Check app](https://support.microsoft.com/en-us/windows/how-to-use-the-pc-health-check-app-9c8abd9b-03ba-4e67-81ef-36f37caa7844). If it says
that the only problem is support for the processor then FlyBy11 can probably help with that,
but it's also useful to check the web for your processor to check if others have
successfully upgraded to Windows 11 with that processor.

If the processor is the only issue then it's worth trying the upgrade using the FlyBy11
app as it worked for me on two laptops. The FlyBy11 can be downloaded from this link
[FlyBy11 Download](https://github.com/builtbybel/Flyby11/releases/latest). Recommend
downloading **FlyBy11 classic**. 
[This YouTube Video](https://youtu.be/C_p3dBrr_Sg?si=bZLr-l7mk-Affdlm) describes the
process very well. As always, **be sure to backup your files** before performing such an upgrade.

#### Old Desktop Computer

I looked at upgrading an old Dell desktop computer that was 14 years old. In this
case the PC Health Check not only identified the processor as an issue but also
identified the lack of both Secure Boot and TPM 2.0 as issues:

![PC Health Check](/images/pc-health-check.png "PC Health Check")

When I ran FlyBy11 it claimed that there was a good chance that the upgrade would be
successful. I then made sure I had a good backup of my files and went ahead and tried
to upgrade to Windows 11 using FlyBy11. It got most of the way through the upgrade
and then failed on rebooting during the process where it reboots during the installation.
It gave me the option to try again but it failed a second time to reboot. I was afraid
it was bricked but after the third try the Windows 11 installation automatically restored
it back to Windows 10 which was a relief.

#### Replacement for Desktop Computer

After the failed upgrade I decided to go to Option #4 above and bought a 
replacement - a small Mini PC with Windows 11 Pro for $169.
It's only 5"L x 5"W X 2"H as shown with a pen for scale:

![Mini PC](/images/mini-pc.jpg "Mini PC")

It's much faster than the old Dell and has better video and rendering performance
(old desktop PC is 17"L X 7"W X 15"H in size). Quality of the new one is high with a metal
case rather than the usual plastic.  So far it has been very stable and easy to
set up and use. It can drive three high resolution monitors and I noticed it ha
room inside for a second SSD drive as well. Power consumption will also be way less
than the large desktop.

[Mini PC on Amazon](https://www.amazon.com/dp/B0FBWGBHTN "Mini PC on Amazon")

## Conclusion

As discussed there are several alternatives and workarounds to upgrade an older PC to
Windows 11. If you use Option #2 to try an upgrade using the FlyBy11 method
**be sure to backup your data** before the upgrade. There's usually the option of going
back to Windows 10 but it's better to be safe. 