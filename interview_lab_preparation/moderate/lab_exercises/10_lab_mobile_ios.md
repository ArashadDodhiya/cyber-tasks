# 🔬 Lab Exercise 10: Mobile Application Testing — iOS

> **Challenge:** Test an iOS application for security vulnerabilities
> **Where:** Kali Linux + MobSF (Docker) for static analysis
> **Time:** ~45-55 minutes
>
> ⚠️ **Note:** Full iOS dynamic testing requires macOS + jailbroken iPhone.
> This lab focuses on **static analysis and concepts** practicable from Kali.

---

## Exercise 10A: iOS App Security Concepts

```
iOS App Architecture:
├── IPA file → Just a ZIP containing the app bundle
│   ├── Payload/
│   │   └── App.app/
│   │       ├── App binary (Mach-O executable)
│   │       ├── Info.plist (app configuration)
│   │       ├── Assets.car (compiled assets)
│   │       ├── Frameworks/ (embedded libraries)
│   │       └── _CodeSignature/ (code signing info)
│   └── iTunesArtwork
│
├── Storage Locations on Device:
│   ├── Keychain → Secure storage for passwords, tokens, keys
│   ├── NSUserDefaults → App preferences (like SharedPreferences)
│   ├── CoreData/SQLite → Local databases
│   ├── Documents/ → App documents directory
│   └── Caches/ → Temporary cached data
│
└── Security Features:
    ├── App Transport Security (ATS) → Forces HTTPS
    ├── Code Signing → Apps must be signed by Apple
    ├── Sandbox → Each app has isolated file system
    ├── Keychain → Hardware-backed credential storage
    └── Certificate Pinning → Prevents MITM attacks

📝 Key differences from Android:
| Feature | Android | iOS |
|---------|---------|-----|
| App format | APK (ZIP with DEX) | IPA (ZIP with Mach-O) |
| Language | Java/Kotlin | Objective-C/Swift |
| Decompile | jadx → readable Java | class-dump → headers only |
| Root/JB | Rooting | Jailbreaking |
| Secure storage | Keystore (limited) | Keychain (stronger) |
| App store | APK sideloading easy | IPA sideloading harder |
```

---

## Exercise 10B: Set Up MobSF for Automated Analysis

```bash
# MobSF (Mobile Security Framework) can analyze both Android + iOS apps
# It works on any OS via Docker — no macOS needed!

# Step 1: Run MobSF via Docker
sudo docker run -d -p 8000:8000 --name mobsf opensecurity/mobile-security-framework-mobsf

# Step 2: Verify it's running
curl -s -o /dev/null -w "%{http_code}" http://localhost:8000
# ✅ EXPECTED: 200

# Step 3: Open MobSF in browser
# Go to: http://localhost:8000
# 📝 You should see the MobSF upload page

# Step 4: Get a sample IPA for analysis
# Option 1: Download DVIA-v2 (Damn Vulnerable iOS App)
# https://github.com/prateek147/DVIA-v2/releases
# Download the IPA file

# Option 2: Use the DIVA APK you already have
# MobSF can analyze both APK and IPA files!
```

---

## Exercise 10C: Analyze an App with MobSF

```
Step 1: Upload the app to MobSF
  - Open http://localhost:8000 in Firefox
  - Drag and drop the APK or IPA file onto the upload area
  - Wait for analysis to complete (may take 1-3 minutes)

Step 2: Review the Security Score
  📝 Overall security score: ______ / 100
  📝 How many HIGH severity issues? ______
  📝 How many MEDIUM severity issues? ______

Step 3: Examine the findings

  Certificate Analysis:
  📝 Is the app signed? ______
  📝 Certificate details: ______

  Manifest Analysis (Android) / Info.plist Analysis (iOS):
  📝 List critical findings: ______

  Code Analysis:
  📝 Hardcoded secrets found? ______
  📝 Insecure crypto usage? ______
  📝 Logging sensitive data? ______

  Network Security:
  📝 Are there HTTP (non-HTTPS) connections? ______
  📝 Is certificate pinning implemented? ______

Step 4: Export the report
  - Click "Generate PDF Report" or "JSON Report"
  - Save it for your records
  📝 Report saved to: ______
```

---

## Exercise 10D: iOS Static Analysis Concepts and Commands

**Task:** Learn the commands used for iOS testing (for interview knowledge).

```bash
# === ON A JAILBROKEN iOS DEVICE (concept reference) ===

# 1. Decrypt and dump an app (apps from App Store are encrypted)
# Using frida-ios-dump:
# frida-ios-dump -l                    # List installed apps
# frida-ios-dump -n "AppName"          # Dump decrypted IPA

# 2. Extract IPA contents (IPA = ZIP file)
# mv app.ipa app.zip
# unzip app.zip -d extracted/
# ls extracted/Payload/App.app/

# 3. Analyze Info.plist
# plutil -p Info.plist                 # Pretty print
# Key things to check:
# - NSAppTransportSecurity             # ATS exceptions (HTTP allowed?)
# - CFBundleURLTypes                   # Custom URL schemes
# - NSCameraUsageDescription           # Camera permission
# - UIRequiredDeviceCapabilities       # Required features

# 4. Dump class headers (Objective-C apps)
# class-dump -H AppBinary -o headers/
# Find interesting method names:
# grep -r "password\|login\|auth\|token" headers/

# 5. Extract strings from binary
# strings AppBinary | grep -i "password\|secret\|key\|token\|http"

# 6. Keychain dump (using Objection)
# objection -g com.example.app explore
# ios keychain dump                    # Dump all keychain items
# ios cookies get                      # Get app cookies
# ios nsuserdefaults get               # Get UserDefaults data
# ios plist cat Info.plist             # Read Info.plist

# 7. Bypass jailbreak detection
# objection -g com.example.app explore
# ios jailbreak disable                # Bypass jailbreak checks

# 8. Bypass SSL pinning
# objection -g com.example.app explore
# ios sslpinning disable               # Bypass certificate pinning
# OR with Frida:
# frida -U -l ssl_bypass.js com.example.app
```

---

## Exercise 10E: Hands-On — Analyze IPA Structure

```bash
# Even without macOS/iPhone, you can analyze IPA file structure

# Step 1: If you have DVIA-v2 IPA, inspect its structure
# If not, create a mock IPA for learning:
mkdir -p /tmp/mock_ipa/Payload/DemoApp.app

# Create a mock Info.plist
cat > /tmp/mock_ipa/Payload/DemoApp.app/Info.plist << 'PLISTEOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleName</key>
    <string>VulnerableApp</string>
    <key>CFBundleIdentifier</key>
    <string>com.example.vulnerable</string>
    <key>CFBundleVersion</key>
    <string>1.0</string>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>vulnapp</string>
                <string>myapp</string>
            </array>
        </dict>
    </array>
    <key>NSCameraUsageDescription</key>
    <string>We need camera for profile photos</string>
    <key>NSLocationAlwaysUsageDescription</key>
    <string>We track your location always</string>
</dict>
</plist>
PLISTEOF

# Step 2: Analyze the Info.plist for security issues
echo "=== SECURITY ANALYSIS ==="

# Check App Transport Security
grep -A3 "NSAppTransportSecurity" /tmp/mock_ipa/Payload/DemoApp.app/Info.plist
# 📝 Is NSAllowsArbitraryLoads set to true?
# ⚠️ This means the app allows HTTP connections — insecure!

# Check URL schemes
grep -A5 "CFBundleURLSchemes" /tmp/mock_ipa/Payload/DemoApp.app/Info.plist
# 📝 What custom URL schemes are registered?
# ⚠️ URL schemes like vulnapp:// can be exploited by malicious apps

# Check permissions
grep "Usage" /tmp/mock_ipa/Payload/DemoApp.app/Info.plist
# 📝 What permissions does the app request?
# ⚠️ "NSLocationAlwaysUsageDescription" → app tracks location ALWAYS — suspicious!

# Step 3: Write iOS security findings
echo "
=== iOS APP SECURITY ASSESSMENT ===
App: VulnerableApp (com.example.vulnerable)

1. ATS Disabled: NSAllowsArbitraryLoads = true → ⚠️ HTTP allowed
2. Custom URL Schemes: vulnapp://, myapp:// → ⚠️ Potential URL scheme hijacking
3. Permissions:
   - Camera: Justified (profile photos)
   - Location Always: ⚠️ Excessive — should use 'When In Use'
4. Code Signing: _______
5. Keychain Usage: _______
6. Certificate Pinning: _______
"
```

---

## Exercise 10F: Android vs iOS Security Comparison

**Task:** Document the key differences for interview preparation.

```
📝 Fill in this comparison table:

| Security Aspect | Android | iOS |
|----------------|---------|-----|
| App Package | APK | IPA |
| Decompilation | Easy (jadx → Java) | Harder (headers only without source) |
| Root/Jailbreak | Rooting | Jailbreaking |
| App Store Review | Less strict | Very strict |
| Sideloading | Easy (enable in settings) | Difficult (needs enterprise cert) |
| Secure Storage | Android Keystore | iOS Keychain (hardware-backed) |
| Local Data | SharedPreferences (XML) | NSUserDefaults (plist) |
| Network Security | Network Security Config | App Transport Security (ATS) |
| Code Obfuscation | ProGuard/R8 | Bitcode/Swift (inherently harder) |
| Dynamic Analysis | Frida, Objection | Frida, Objection (need jailbreak) |
| Proxy Setup | Wi-Fi proxy settings | Wi-Fi proxy + profile install |
| Certificate Pinning | OkHttp, custom | NSURLSession, custom |

Common interview questions:
1. "How would you test a mobile app for security?"
   📝 Your answer: ________________________________

2. "What's the difference between static and dynamic analysis?"
   📝 Static: ________________________________
   📝 Dynamic: ________________________________

3. "How do you bypass certificate pinning?"
   📝 Tools: ________________________________
   📝 Method: ________________________________

4. "What are the OWASP Mobile Top 10?"
   📝 List at least 5: ________________________________
```

---

## ✅ Completion Checklist

- [ ] Understood iOS app architecture and security features (10A)
- [ ] Set up MobSF via Docker for automated analysis (10B)
- [ ] Analyzed an app with MobSF — reviewed security findings (10C)
- [ ] Learned iOS static analysis commands and tools (10D)
- [ ] Analyzed IPA structure and Info.plist for security issues (10E)
- [ ] Completed Android vs iOS security comparison for interviews (10F)

---

**🎉 Congratulations! You've completed all 10 Moderate Challenge Lab Exercises!**

Go back to [Prerequisites](./00_lab_prerequisites.md) to review your overall progress.
