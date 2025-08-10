---
title: How I Repaired a Samsung PM981 SSD to Recover Lost Data
categories: [hardware-hacking]
date: '2025-08-10 10:04:23'
tags: [ssd, pm981, repair, data-recovery, hardware-hack]
---

**TL;DR:**  
I fixed a dead clock on a Samsung PM981 NVMe SSD by replacing the failed oscillator and recovered the customer’s data.  
Below are the diagnosis steps, the repair procedure I used in the lab, and notes on safe post-repair handling.

![Samsung PM981 SSD board](/assets/img/pm981-repair/pm981.jpeg){: .shadow }

---

## Background  
A PM981 (OEM Samsung) NVMe SSD arrived *not detected* by the host. The owner needed the data urgently. Initial checks showed no SMART access and the drive produced no NVMe responses — typical symptom of a failed or dead SSD controller clock.

---

## My Repair Steps

1. **Initial power check**  
   Hooked the SSD to an external enclosure and monitored current draw.  
   - At first: small current draw.  
   - Then: it dropped to zero → likely the controller stopped running.

2. **Check for shorts**  
   Removed the SSD and measured between **Vcc** and **GND** using the PM981 schematics[^1].  
   - No short detected → ruled out a common SSD power rail short failure.

3. **Visual inspection**  
   - Found a **discolored SMD oscillator** near the controller.  
   ![Discolored clock](/assets/img/pm981-repair/discolored_clock.png){: .shadow }  
   - Probed the oscillator output with a scope → no signal.

4. **Clock testing with jumpers**  
   - Removed the dead oscillator; as you can see, it was really discolored.  
   ![Removed clock](/assets/img/pm981-repair/removed_clock.png){: .shadow }  
   - Soldered **long jumper wires** to test multiple replacement oscillators.  
   ![Jumper wires](/assets/img/pm981-repair/jumper_wire.png){: .shadow }  
   - Tried: **10 MHz, 12 MHz, 16 MHz, 20 MHz** → no detection.  
   - Tried: **25 MHz** → SSD detected but data unreadable.  
   - Tried: **26 MHz** → SSD detected and data became readable.

5. **Finding a donor clock**  
   - Salvaged a **26 MHz oscillator** from an old earpod PCB because I wasn’t ready to ship a new clock.  
   - Shortened jumper wires and soldered it in place.  
   ![New clock](/assets/img/pm981-repair/new_clock.png){: .shadow }  
   - SSD booted cleanly on every power cycle and power draw was constant.  
   ![Current draw](/assets/img/pm981-repair/current_draw.jpg){: .shadow }

6. **Data recovery**  
   - Imaged the full **512 GB** with `ddrescue` to a separate drive.  
   - Verified integrity — all data was recovered.

---

## Parts & Tools  
- Replacement SMD oscillator compatible with the PM981 frequency (check part marking or controller datasheet) — I used a 26.0 MHz SMD oscillator (confirm your board’s spec)  
- Hot air rework station (or fine-tip iron with hot air backup)  
- Flux, solder wick, solder paste  
- Tweezers, microscope (or loupe)  
- Multimeter  
- External SSD enclosure  
- (Optional) Oscilloscope to confirm signal  
- (Optional) USB-C current meter  

> **Warning:** working on live drives risks data loss. Do this in a lab with backups and written permission.

---

## Data Recovery Steps  
- **Don’t run destructive tools** (no secure erase).  
- Use a read-only approach: `ddrescue` or commercial tools to image the drive.  
- If imaging succeeds, mount or run file-system recovery tools on the image.  
- In this case, I created a full image with `ddrescue` then copied critical files to backup media.

---

## Pitfalls & Notes  
- **Frequency matters.** Using the wrong frequency or a noisy oscillator can make the controller unstable.  
- **ESD & heat.** SMD rework near the controller is risky; uncontrolled heat can desolder nearby passive components.  
- **Controller-specific quirks.** Some OEM boards include extra power gating; confirm rails are stable before assuming oscillator fault.  
- **Document everything** for the customer and your own reproducibility.

---

## Summary  
A dead oscillator can make a perfectly healthy SSD controller appear permanently dead. With careful diagnosis, an appropriate replacement oscillator, and conservative imaging practices (read-only imaging), it’s often possible to bring the drive back to life and recover user data.

---

## References  
[^1]: Samsung PM981 NVMe SSD Datasheet – [Samsung Semiconductor](https://www.compuram.biz/documents/datasheet/Samsung_PM981_Rev_1_1.pdf)

