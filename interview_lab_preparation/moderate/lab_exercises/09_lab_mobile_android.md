# 🔬 Lab Exercise 9: Mobile Application Testing — Android

> **Challenge:** Reverse engineer and test an Android application for security vulnerabilities
> **Where:** Kali Linux + Android Studio Emulator (or static analysis only)
> **Time:** ~60-75 minutes

---

## Exercise 9A: Set Up the Android Testing Environment

```bash
# Option A: Static Analysis Only (No emulator needed)
# Install APK analysis tools on Kali

# Step 1: Install apktool
sudo apt install apktool -y

# Step 2: Install jadx (Java decompiler)
# Download latest from: https://github.com/skylot/jadx/releases
# Or install via apt:
sudo apt install jadx -y

# Step 3: Download DIVA (Damn Insecure and Vulnerable App)
wget https://github.com/payatu/diva-android/raw/master/diva-beta.apk -O ~/Downloads/diva-beta.apk
# If download fails, search "DIVA APK download" — multiple mirrors available

# Step 4: Verify the APK
file ~/Downloads/diva-beta.apk
# ✅ EXPECTED: Java archive data (JAR) or Zip archive
ls -la ~/Downloads/diva-beta.apk
# 📝 File size: ______________
```

```bash
# Option B: Full Setup with Android Emulator (Optional, more complete)
# Requires Android Studio — install only if you want dynamic testing

# Step 1: Download Android Studio
# https://developer.android.com/studio
# Install and create an AVD (Android Virtual Device)
# Recommended: Pixel device with API 28-30

# Step 2: Start the emulator
# Open Android Studio → Tools → AVD Manager → Start emulator

# Step 3: Install DIVA on the emulator
# adb install ~/Downloads/diva-beta.apk
# ✅ EXPECTED: Success
```

---

## Exercise 9B: Decompile APK with apktool

```bash
# Step 1: Decompile the APK (extracts resources, manifest, smali code)
apktool d ~/Downloads/diva-beta.apk -o /tmp/diva_apktool
# ✅ EXPECTED: Decompilation complete
ls /tmp/diva_apktool/
# 📝 You should see: AndroidManifest.xml, res/, smali/, assets/, lib/

# Step 2: Examine AndroidManifest.xml
cat /tmp/diva_apktool/AndroidManifest.xml

# 📝 Look for security issues:
# Is android:debuggable="true"? ______
# Is android:allowBackup="true"? ______
# Are there exported activities? ______
# Are there custom URL schemes? ______
# What permissions does the app request? ______

# Step 3: List all activities (screens in the app)
grep -i "activity" /tmp/diva_apktool/AndroidManifest.xml
# 📝 List the activities: ______________
# 📝 Are any marked as exported="true" without proper protection?
```

---

## Exercise 9C: Decompile to Java Source with jadx

```bash
# Step 1: Decompile APK to Java source code
jadx ~/Downloads/diva-beta.apk -d /tmp/diva_jadx
# ✅ EXPECTED: Decompilation complete

# Step 2: Explore the source code structure
find /tmp/diva_jadx -name "*.java" | head -20
# 📝 List the main Java files: ______________

# Step 3: Look at the main activity
find /tmp/diva_jadx -name "MainActivity.java" -exec cat {} \;
# 📝 What does the main activity do?
```

---

## Exercise 9D: Search for Hardcoded Secrets

```bash
# Step 1: Search for hardcoded passwords
grep -ri "password" /tmp/diva_jadx/ --include="*.java"
# 📝 Found any hardcoded passwords? ______________

# Step 2: Search for API keys
grep -ri "api_key\|apikey\|api-key" /tmp/diva_jadx/ --include="*.java"
grep -ri "secret" /tmp/diva_jadx/ --include="*.java"
# 📝 Found any API keys or secrets? ______________

# Step 3: Search for hardcoded URLs
grep -ri "http://" /tmp/diva_jadx/ --include="*.java"
grep -ri "https://" /tmp/diva_jadx/ --include="*.java"
# 📝 Any insecure HTTP URLs? ______________
# 📝 Any internal/staging URLs exposed? ______________

# Step 4: Search for Firebase/database URLs
grep -ri "firebase\|firebaseio" /tmp/diva_jadx/ --include="*.java"
grep -ri "amazonaws\|s3\.\|dynamodb" /tmp/diva_jadx/ --include="*.java"
# 📝 Found any cloud service URLs? ______________

# Step 5: Search for cryptographic issues
grep -ri "AES\|DES\|RSA\|MD5\|SHA" /tmp/diva_jadx/ --include="*.java"
grep -ri "SecretKeySpec\|Cipher" /tmp/diva_jadx/ --include="*.java"
# 📝 Is the app using weak crypto (DES, MD5)? ______________
# 📝 Are encryption keys hardcoded? ______________

# Step 6: Search for logging sensitive data
grep -ri "Log\.\|println" /tmp/diva_jadx/ --include="*.java" | grep -i "pass\|token\|key\|secret"
# 📝 Is sensitive data being logged? ______________
```

---

## Exercise 9E: Check for Insecure Data Storage

```bash
# Step 1: Search for SharedPreferences usage
grep -ri "SharedPreferences\|getSharedPreferences" /tmp/diva_jadx/ --include="*.java"
# 📝 What data is stored in SharedPreferences? ______________
# SharedPreferences stores data as plain XML in:
# /data/data/com.app.name/shared_prefs/

# Step 2: Search for SQLite database usage
grep -ri "SQLiteDatabase\|openOrCreateDatabase\|execSQL" /tmp/diva_jadx/ --include="*.java"
# 📝 What data is stored in SQLite? ______________
# Databases are stored in:
# /data/data/com.app.name/databases/

# Step 3: Search for file storage on external/SD card
grep -ri "getExternalStorage\|Environment.getExternal\|WRITE_EXTERNAL" /tmp/diva_jadx/ --include="*.java"
# 📝 Does the app write to external storage? ______________
# ⚠️ External storage is readable by ALL apps — sensitive data should NOT be stored here

# Step 4: Check for temporary file usage
grep -ri "createTempFile\|getCacheDir\|FileOutputStream" /tmp/diva_jadx/ --include="*.java"
# 📝 Are temp files being created with sensitive data? ______________

# DIVA Specific: Look for insecure storage challenges
find /tmp/diva_jadx -name "*.java" | xargs grep -l "insecure\|storage\|credential"
# 📝 Which source files handle data storage? ______________
```

---

## Exercise 9F: Analyze App Permissions and Components

```bash
# Step 1: Extract permissions from AndroidManifest.xml
grep "uses-permission" /tmp/diva_apktool/AndroidManifest.xml
# 📝 List all permissions requested:
# - ____________________________
# - ____________________________
# - ____________________________

# 📝 Are there dangerous permissions?
# CAMERA, RECORD_AUDIO, READ_CONTACTS, ACCESS_FINE_LOCATION, READ_SMS
# These should be justified by app functionality

# Step 2: Check for exported components (attack surface)
# Exported activities can be launched by other apps
grep -A2 "exported" /tmp/diva_apktool/AndroidManifest.xml

# Step 3: Check for content providers
grep -i "provider" /tmp/diva_apktool/AndroidManifest.xml
# 📝 Are content providers exported? ______________
# ⚠️ Exported content providers can leak data to other apps

# Step 4: Check for broadcast receivers
grep -i "receiver" /tmp/diva_apktool/AndroidManifest.xml
# 📝 Are there receivers without permission protection? ______________

# Step 5: Check for intent filters (deep links / URL schemes)
grep -B1 -A5 "intent-filter" /tmp/diva_apktool/AndroidManifest.xml
# 📝 Any custom URL schemes? ______________
# ⚠️ Custom URL schemes can be exploited for unauthorized actions

# Step 6: Write a security summary
echo "
=== ANDROID APP SECURITY ASSESSMENT ===
App: DIVA (Damn Insecure and Vulnerable App)

1. Debuggable: ______
2. Allow Backup: ______
3. Hardcoded Secrets Found: ______
4. Insecure Storage: ______
5. Dangerous Permissions: ______
6. Exported Components: ______
7. Weak Cryptography: ______
8. HTTP (non-HTTPS) URLs: ______
"
```

---

## Exercise 9G: Proxy Android Traffic Through Burp (Emulator Only)

> **Skip this exercise if you don't have Android emulator set up.**

```bash
# Step 1: Configure Burp to listen on all interfaces
# Burp → Proxy → Options → Edit listener
# Bind to: All interfaces, Port: 8080

# Step 2: Configure emulator proxy
# On the emulator:
# Settings → Wi-Fi → Long press connected network → Modify
# Proxy: Manual
# Hostname: 10.0.2.2 (host machine from emulator)
# Port: 8080

# Step 3: Install Burp CA certificate on emulator
# Export Burp CA: Proxy → Options → Import/Export CA → Export Certificate in DER format
# Save as burp-cert.der
# Convert to PEM:
openssl x509 -inform der -in burp-cert.der -out burp-cert.pem
# Push to emulator:
# adb push burp-cert.pem /sdcard/
# On emulator: Settings → Security → Install from storage → select burp-cert.pem

# Step 4: Browse in the emulator's browser
# Traffic should now flow through Burp
# 📝 Can you see HTTPS traffic in Burp? ______________
```

---

## ✅ Completion Checklist

- [ ] Set up APK analysis tools: apktool + jadx (9A)
- [ ] Decompiled APK with apktool — examined AndroidManifest.xml (9B)
- [ ] Decompiled to Java source with jadx — explored code structure (9C)
- [ ] Searched for hardcoded secrets: passwords, API keys, URLs (9D)
- [ ] Identified insecure data storage: SharedPreferences, SQLite, external (9E)
- [ ] Analyzed permissions, exported components, and intent filters (9F)
- [ ] (Optional) Proxied Android traffic through Burp Suite (9G)

---

**Next:** [Challenge 18: Mobile iOS Testing →](./10_lab_mobile_ios.md)
