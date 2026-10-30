# Passcode Bypass Leading to Unauthorized KYC Access and Passcode Reset in WazirX Android Application

## Summary
The WazirX Android application (`com.wrx.wazirx`) fails to enforce passcode/session authentication on two sensitive activities. An attacker with temporary physical or session access to an unlocked device (or via direct activity invocation) can bypass the app's passcode lock to (1) access and complete the KYC submission flow, and (2) reset the account passcode — without ever supplying the existing passcode.

## Affected Product
- **Vendor:** WazirX (Zettai Pte. Ltd.)
- **Component:** WazirX Android App
- **Affected version(s):** <!-- fill in app version tested -->
- **Platform:** Android

## Vulnerability Classification
- CWE-862: Missing Authorization
- CWE-287: Improper Authentication
- CWE-926: Improper Export of Android Application Components

## CVSS v3.1
`AV:L/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N` — Score: ~7.5 (High)

## Vulnerability 1 — KYC Access Without Passcode Verification
`com.wrx.wazirx.views.kyc.level1.country.CountryActivity` is declared in the manifest and reachable directly, bypassing the lock screen.

**Evidence — AndroidManifest.xml declaration of `CountryActivity` (decompiled via JADX):**
<!-- SCREENSHOT 1: Image 2 — manifest excerpt highlighting CountryActivity -->

**Steps to Reproduce:**
1. Enable passcode lock in the app.
2. Close or lock the application.
3. Without entering passcode, trigger direct access to `com.wrx.wazirx.views.kyc.level1.country.CountryActivity`.
4. Observe the KYC screen opens with no authentication prompt.
5. Complete the KYC process.

**Result — KYC "Identity Verification" screen reached without passcode:**
<img width="1366" height="768" alt="Screenshot 2026-04-25 175016" src="https://github.com/user-attachments/assets/e29994fa-5bd5-4f72-a169-b28c82a00760" />


**Result — KYC flow continues to "KYC Type" selection:**
<img width="436" height="662" alt="Screenshot 2026-04-25 175314" src="https://github.com/user-attachments/assets/07a70219-0144-4e23-a422-e370091f99ca" />


Backend accepts progression through the KYC flow despite no passcode verification on the client.

## Vulnerability 2 — Passcode Reset Without Verifying Existing Passcode
`com.wrx.wazirx.views.settings.PasscodeActivity` is declared in the manifest and can be launched directly while locked, allowing a new passcode to be set without the old one.

**Evidence — AndroidManifest.xml declaration of `PasscodeActivity` (decompiled via JADX):**
<img width="1366" height="768" alt="Screenshot 2026-04-25 175128" src="https://github.com/user-attachments/assets/0883db70-6c79-4e7e-bb5f-5e6f5edac620" />


**Steps to Reproduce:**
1. Ensure passcode lock is enabled.
2. Do not unlock the application.
3. Trigger direct access to `com.wrx.wazirx.views.settings.PasscodeActivity`.
4. Observe the passcode reset screen opens directly.
5. Set a new passcode — no prompt for the old one.

**Result:** Passcode changed successfully with zero verification of the prior credential.

## Combined Impact
An attacker with momentary device/session access can bypass the passcode gate, reset the passcode, gain persistent app access, and complete KYC under the victim's identity — all without authentication.

## Root Cause
Static analysis of the manifest (above) shows these activities are exported/reachable without a lock-state re-check at entry, rather than authentication being enforced centrally across every sensitive screen or validated server-side.

## Recommended Fix
1. Enforce passcode/session verification at the entry point of every sensitive activity.
2. Require the existing passcode or strong re-authentication before passcode reset.
3. Validate authentication/session state server-side before processing sensitive actions.
4. Audit all exported activities for similar auth-gate gaps.

## Proof of Concept
Full video PoC :https://drive.google.com/drive/u/2/folders/1lQ4HwBj1Q5n-71Blw8TH19BPxJygFgxB ): screenshots above are stills from static (manifest) and dynamic (runtime) testing.



## Credit
Discovered and reported by **Soyam Arya** (aka honest_corrupt) — independent Android security researcher.
Contact: soyamarya96ethical@gmail.com
