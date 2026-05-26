# Lesson 4 - Taking Your Exam Online (In Which Pearson VUE Treats Your Bedroom Like a Regulated Financial Institution)

This module covers the online proctored exam process end to end — from preparing your room to waiting for a human being to verify that your passport is, in fact, your passport. None of this is Oracle Database content. All of it will absolutely ruin your exam day if you ignore it.

The system is run through **Pearson VUE** and their **OnVUE** application. It is not complicated, but it has rules, and the rules are enforced by a proctor who is watching you through your webcam and has the power to terminate your exam. Treat this accordingly.

By the end of this lesson, you should be able to:

- Set up a compliant exam environment before exam day
- Meet all technical system requirements
- Navigate the six-step check-in process without incident
- Know what to do if something goes wrong

---

## 1. Room Setup (The Proctor Is Watching)

Before exam day, prepare your physical space:

- **Find a quiet room with no other people present.** Not "nearby." Not "in the next room." No one in the room.
- **Clear your desk** of everything except: your computer, your mouse, and (for laptops) your charger.
- **Remove all writable surfaces** from the room — books, calendars, notepads, whiteboards. If it can be written on, it should not be visible.
- **Have valid photo ID ready.** A driver's licence or passport. You cannot take the exam without it.

> Water is permitted. Food, drinks, gum, and smoking are not. Apparently chewing gum is where they draw the line on academic integrity, and honestly, fair enough.

---

## 2. Technical Requirements (Run This Check Before Exam Day, on the Exact Machine You Will Use)

This is important enough to repeat: **run your system test on the same computer and same network you will use on the day of the exam.** Testing on your work laptop on office Wi-Fi and then sitting down at home on a hotspot is how you find out about problems at the wrong moment.

### Operating System
- Windows 10, 8.1, or 7
- macOS 10.9 or higher

### Display
- Minimum: **1024 × 768** resolution
- Higher is better — use it if available

### Internet Connection
- **3 Mbps download / 2 Mbps upload** minimum, sustained for the exam duration
- Must be a stable wired or Wi-Fi connection
- **Do not use a mobile hotspot**

### Browser
- Internet Explorer 11, Chrome, Firefox, or Safari
- Use the latest version available

### Hardware
- Laptop or desktop with a **working webcam** and **built-in microphone**
- **No external headsets or microphones** permitted

### Browser Settings
- **Cookies: enabled**
- **Pop-up blockers: disabled**

### Network
- **Disable any VPN** before the exam
- **Do not take the exam behind a company firewall**
- Make sure you have **administrative rights** on the machine to install OnVUE

---

## 3. Software to Close Before the Exam

The following categories of software will either cause your system check to fail or get your exam flagged. Close all of them before testing and before the exam:

| Category | Examples |
|----------|----------|
| Chat / messaging | Slack, Jabber, Teams, Discord |
| Remote access | VNC Server, TeamViewer, AnyDesk, RDP |
| Screen capture / recording | Greenshot, Snagit, TinyTake, OBS |
| Antivirus | McAfee and similar — disable during the exam as it can slow your system |
| Background system tray apps | WebEx, printer software, anything non-essential |

Check your **system tray** (bottom-right on Windows 10) and your **task manager** for anything running that shouldn't be. Kill it before starting the check-in.

---

## 4. Exam Day — The Six Steps

### Step 1 — Access Your Exam

Go to your **appointment confirmation email** and click the Pearson VUE link. This takes you to the Pearson VUE page.

### Step 2 — Log In via CertView

Click **Log In**, which redirects to the **Oracle CertView login page**. Sign in with your **Oracle SSO credentials**. Your Pearson VUE dashboard shows upcoming and completed exams. Click your exam.

### Step 3 — Begin Your Exam

The **Begin Exam** button becomes available **30 minutes before** your scheduled start time. Click it when you are ready.

### Step 4 — Download and Launch OnVUE

1. **Copy the access code** displayed on screen — you will need it shortly
2. Click **Download** and save `OnVUE.exe` to your system
3. Run the downloaded file
4. Enter the **access code** and your **phone number** (used by the proctor if they need to reach you during check-in)
5. Click **Get Started**

### Step 5 — System Check

OnVUE tests your:
- **Microphone** — speak into it when prompted, or the test will fail
- **Internet speed**
- **Webcam**

All checks must pass. Do not proceed if any check fails — use the **Pearson VUE Live Chat** option to resolve issues before continuing.

### Step 6 — Check-In Process

This is where it gets thorough. You will need to upload photos of:
- **Yourself**
- **Your ID document** (driver's licence or passport)
- **Your workspace** from multiple angles

You can take these photos using either:
- Your **mobile phone** — via text message link or a typed URL
- Your **webcam**

After uploading photos, read and observe the following rules — the proctor will verify compliance:

> - No one may enter your workspace
> - You will not leave your workspace
> - Mobile phone, headphones, and watches must be out of reach
> - You will not need your mobile phone again after this point
> - Food, drinks, smoking, and gum are prohibited
> - Water is permitted
> - Medical devices and medication are permitted

Close all remaining applications. Sit in front of your screen with your **face clearly visible to the webcam** and wait. Your proctor will be available within **15 minutes**.

---

## 5. Proctor Wait and Troubleshooting

Once check-in is complete, the proctor reviews your photos and workspace before launching the exam.

| Situation | What to Do |
|-----------|-----------|
| Proctor has an issue with your photos or ID | They will guide you through next steps |
| 15 minutes pass with no proctor contact | Speak into your microphone to get their attention |
| Proctor still does not respond | Go back to Step 3, click Begin Exam, and repeat the process |
| Ongoing technical issues | Use the **Pearson VUE Support** links provided in your exam portal |

---

## 6. Pre-Exam Day Checklist

Use this the day before and the morning of your exam:

**Environment**
- [ ] Quiet room, no other people
- [ ] Desk cleared to: computer, mouse, charger only
- [ ] All writable surfaces removed from the room
- [ ] Valid photo ID within reach

**Technical**
- [ ] System test run on this machine, on this network
- [ ] OS meets requirements
- [ ] Screen resolution ≥ 1024 × 768
- [ ] Internet speed ≥ 3 Mbps down / 2 Mbps up
- [ ] Webcam and built-in microphone confirmed working
- [ ] Browser updated, cookies enabled, pop-ups disabled
- [ ] VPN disabled
- [ ] Not behind a company firewall
- [ ] Admin rights on the machine

**Software**
- [ ] Chat apps closed (Slack, Teams, etc.)
- [ ] Remote access software closed
- [ ] Screen capture tools closed
- [ ] Antivirus disabled
- [ ] System tray cleaned up
- [ ] All non-essential applications closed

**Accounts**
- [ ] Oracle CertView credentials confirmed
- [ ] Appointment confirmation email accessible
- [ ] Phone number available for OnVUE entry

---

## 7. Wrap-Up (The Boring Part Is Now Behind You)

The online proctored exam process is straightforward but unforgiving about preparation. The most common day-of failures are:

- **Software that fails the system check** — close everything in advance
- **Photo ID not ready** — have it on the desk before you start
- **Workspace not compliant** — the proctor will see your room via webcam; clean it before you sit down
- **VPN still running** — disable it completely before launching OnVUE

Do the system test early. Know your CertView login. Set up your room the night before. Then sit down and show them what you know about Oracle databases.

Good luck. The prep is done. The rest is yours.
