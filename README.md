# 🔐 Secure QR Code Scanner with AI-based Threat Detection and Cryptographic Verification

<div align="center">

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![Keras](https://img.shields.io/badge/Keras-D00000?style=for-the-badge&logo=keras&logoColor=white)
![Cryptography](https://img.shields.io/badge/ECDSA-Cryptography-1A6B3C?style=for-the-badge)

**A real-time Streamlit-based QR code scanner with two modes: a 3-layer AI security pipeline (image tamper detection + malicious URL classification + TLS validation) and ECDSA cryptographic signature verification.**

[Features](#-features) • [Project Structure](#-project-structure) • [Installation](#-installation) • [How It Works](#-how-it-works) • [Usage](#-usage)

</div>

---

## 📌 Overview

This application provides two distinct scanning modes through a Streamlit web interface:

1. **Security Scanner** — Runs a 3-layer AI pipeline on any standard QR code
2. **Signature Scanner** — Verifies ECDSA cryptographic signatures embedded in specially generated QR codes

The system uses live webcam feed via OpenCV, AI models loaded with Keras and joblib, and cryptographic verification using the `cryptography` library with SECP256R1 elliptic curve keys.

---

## ✨ Features

### 🛡️ Security Scanner (3-Layer Pipeline)

| Layer | Module | What it does |
|---|---|---|
| Layer 1 | `qr_model.h5` (Keras CNN) | Detects if the scanned QR image has been tampered with |
| Layer 2 | `random_forest_model.joblib` | Classifies the embedded URL as safe or malicious using 21 extracted features |
| Layer 3 | `tls_utils.py` | Validates the TLS/SSL certificate of the URL (expiry, domain match, SAN/wildcard support) |

### 🔏 Signature Scanner (ECDSA)

- Parses QR codes containing a JSON payload with keys `"d"` (data/URL) and `"s"` (base64-encoded ECDSA signature)
- Verifies the signature against `ec_public.pem` using `SECP256R1` + `SHA256`
- Blocks any QR code that fails verification or is not in the expected signed JSON format

### 📊 Dashboard & Sidebar

- Live statistics: Total Scans, Safe Scans, Blocked Scans, Success Rate
- Last 5 scan activity log with timestamps
- Adjustable detection sensitivity slider (controls frame sampling rate via `frame_count % (11 - sensitivity)`)
- Toggle: Auto-open safe URLs, Sound alerts
- Clear all history button
- Change Scanner Mode button

---

## 📁 Project Structure

```
📦 Secure-QR-Code-Scanner/
├── app.py                          # Main Streamlit app — UI, camera loops, mode selection
├── crypto_utils.py                 # verify_signature() using ec_public.pem (ECDSA SHA256)
├── tls_utils.py                    # check_tls_certificate() — expiry + CN/SAN + wildcard check
├── utils.py                        # preprocess_qr_image() + extract_url_features() (21 features)
├── generate_keys.py                # Generates ECDSA key pair → ec_private.pem, ec_public.pem
├── qr_generator.py                 # Signs a URL with ec_private.pem → saves secure_qr.png
├── scanner.py                      # Standalone CLI-only signature scanner (no Streamlit)
├── models/
│   ├── qr_model.h5                 # Keras CNN — image tamper detection (input: 128×128 RGB)
│   └── random_forest_model.joblib  # Scikit-learn Random Forest — URL classification
├── ec_private.pem                  # ECDSA private key (DO NOT commit — add to .gitignore)
├── ec_public.pem                   # ECDSA public key (used by crypto_utils.py)
├── secure_qr.png                   # Example signed QR output from qr_generator.py
├── .env                            # Stores GOOGLE_API_KEY for Web Risk API
└── requirements.txt
```

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/Nimraaaaaaaa/Secure-QR-Code-Scanner-with-AI-based-Threat-Detection-and-Cryptographic-Verification.git
cd Secure-QR-Code-Scanner-with-AI-based-Threat-Detection-and-Cryptographic-Verification
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

Required packages:

```
streamlit
opencv-python
numpy
keras
tensorflow
joblib
scikit-learn
cryptography
qrcode[pil]
python-dotenv
requests
```

### 3. Set up environment variables

Create a `.env` file in the project root:

```
GOOGLE_API_KEY=your_google_webrisk_api_key_here
```

### 4. Generate ECDSA Key Pair

```bash
python generate_keys.py
```

This creates `ec_private.pem` and `ec_public.pem` using the `SECP256R1` curve. Output:

```
✅ ECDSA Keys generated!
ec_private.pem — secret rakho
ec_public.pem  — scanner mein use hogi
```

> ⚠️ **Never commit `ec_private.pem` to GitHub.** Add it to `.gitignore`.

### 5. Add trained models

Place the following files inside the `models/` folder before launching the Security Scanner:

- `qr_model.h5` — Keras CNN trained for QR image tamper detection
- `random_forest_model.joblib` — Scikit-learn Random Forest trained for URL threat classification

---

## 🚀 Usage

### Run the Streamlit App

```bash
streamlit run app.py
```

On launch, select a scanner mode:

- **🛡️ Security Scanner** — for standard QR codes (requires models to be loaded)
- **🔏 Signature Scanner** — for ECDSA-signed QR codes only

### Generate a Signed QR Code

```bash
python qr_generator.py
```

Signs the URL `https://botifyhub.io` with `ec_private.pem` and saves the output as `secure_qr.png`.

The QR encodes the following JSON:

```json
{"d": "https://botifyhub.io", "s": "<base64_ecdsa_signature>"}
```

### Run the Standalone CLI Scanner (Signature Mode only)

```bash
python scanner.py
```

Reads from webcam directly. Verifies signatures and opens valid URLs in browser. Press `Q` to quit.

---

## 🔍 How It Works

### Security Scanner Flow (`run_security_scanner()` in `app.py`)

```
Webcam Frame (OpenCV)
        │
        ▼
cv2.QRCodeDetector.detectAndDecode()
        │
        ├─► Layer 1 — Image Tamper Detection
        │       preprocess_qr_image(qr_image)
        │           → resize to 128×128, normalize to [0,1], expand dims
        │       qr_model.h5.predict()
        │           → label == 1  →  ❌ BLOCKED (Tampered)
        │           → label == 0  →  ✅ Continue
        │
        ├─► Layer 2 — Malicious URL Detection
        │       extract_url_features(url)  →  21 numeric features
        │       random_forest_model.predict()
        │           → pred == 1  →  ❌ BLOCKED (Malicious URL)
        │           → pred == 0  →  ✅ Continue
        │
        └─► Layer 3 — TLS Certificate Validation
                check_tls_certificate(url)
                    → SSL socket handshake on port 443
                    → Check notBefore / notAfter
                    → Match hostname vs CN + SAN (wildcard aware)
                    → Returns (True, "TLS certificate valid")
                            or (False, reason)
```

### Signature Scanner Flow (`run_signature_scanner()` in `app.py`)

```
Webcam Frame (OpenCV)
        │
        ▼
cv2.QRCodeDetector.detectAndDecode()
        │
        ▼
json.loads(data)  →  {"d": url, "s": signature}
        │
        ▼
verify_signature(url, signature)   [crypto_utils.py]
        │
        ├── base64.b64decode(signature)
        ├── load ec_public.pem (SECP256R1)
        └── public_key.verify(sig, data.encode(), ECDSA(SHA256))
                ├── Valid    →  ✅ URL opened in browser
                └── Invalid  →  ❌ BLOCKED (Forged / Tampered)
```

---

## 🧠 URL Feature Extraction — `extract_url_features()` in `utils.py`

The function extracts **21 numeric features** from a URL string for the Random Forest classifier:

| # | Feature Description |
|---|---|
| 1 | Has digit in URL |
| 2 | Starts with HTTPS |
| 3 | Contains `google.com` |
| 4 | Total dot (`.`) count |
| 5 | `www` occurrence count |
| 6 | `@` symbol count |
| 7 | Forward slash count in path |
| 8 | Reserved (0) |
| 9 | URL length < 20 |
| 10 | `https` substring count |
| 11 | `http` substring count |
| 12 | `%` character count |
| 13 | `?` character count |
| 14 | `=` character count |
| 15 | Total URL length |
| 16 | Hostname length |
| 17 | Reserved (0) |
| 18 | Total digit count in URL |
| 19 | Total alpha character count |
| 20 | Last path segment length |
| 21 | TLD (top-level domain) length |

---

## 🔐 TLS Validation — `check_tls_certificate()` in `tls_utils.py`

Three checks are performed on the SSL certificate of the URL's hostname:

1. **Validity Period** — `notBefore` and `notAfter` compared against `datetime.utcnow()`
2. **Domain Match** — Hostname matched against `commonName` (CN) and all `subjectAltName` (SAN) DNS entries, including wildcard patterns (e.g. `*.example.com`)
3. **SSL Handshake** — Uses `ssl.create_default_context()` with a 5-second socket timeout on port 443

---

## 🔑 Cryptographic Key Files

| File | Purpose |
|---|---|
| `generate_keys.py` | Generates `SECP256R1` ECDSA key pair |
| `ec_private.pem` | Signs URL data — **keep secret, never commit** |
| `ec_public.pem` | Verifies signatures in `crypto_utils.py` |
| `qr_generator.py` | Signs a URL and saves as `secure_qr.png` |
| `crypto_utils.py` | `verify_signature(data, signature)` → `True` / `False` |

---

## 📸 Scanner Mode Comparison

| Feature | 🛡️ Security Scanner | 🔏 Signature Scanner |
|---|---|---|
| Input QR type | Any standard QR code | Signed JSON QR only (`{"d":..., "s":...}`) |
| AI Models used | ✅ Keras CNN + Random Forest | ❌ Not used |
| TLS Check | ✅ Yes | ❌ Not used |
| Crypto Verification | ❌ Not used | ✅ ECDSA SHA256 |
| Auto-open URL | ✅ All 3 layers must pass | ✅ Signature must be valid |
| Blocks on | Tampered image or Malicious URL | Invalid or forged signature |
| Standalone CLI | ❌ Streamlit only | ✅ Also available via `scanner.py` |

---

## 🔮 Possible Improvements

- [ ] Add VirusTotal API integration (`check_google_webrisk()` is already implemented in `utils.py` but not called in the main pipeline)
- [ ] Support QR image file upload in addition to live webcam
- [ ] Export scan history as CSV from the sidebar
- [ ] Train `qr_model.h5` on a larger and more diverse tampered/authentic QR dataset
- [ ] Dockerize the full application
- [ ] Add support for multiple signed URLs in one session without cooldown

---

## ⚠️ Important Notes

- Add `ec_private.pem` to `.gitignore` before pushing to GitHub
- The `GOOGLE_API_KEY` in `.env` is loaded in `utils.py` via `python-dotenv` for the `check_google_webrisk()` function — it is implemented but not yet wired into the main scan pipeline
- Both AI models (`qr_model.h5` and `random_forest_model.joblib`) must exist inside the `models/` folder before the Security Scanner can be launched
- The 3-second cooldown (`current_time - last_scan_time > 3`) prevents duplicate scans of the same QR code

---

## 📄 License

This project is licensed under the **MIT License**.

---

## 👩‍💻 Author

**Nimra Abdulhaq**  
📧 [nimraaishere@gmail.com](mailto:nimraaishere@gmail.com)  
🐙 [GitHub: @Nimraaaaaaaa](https://github.com/Nimraaaaaaaa)

---

<div align="center">

⭐ **If you found this project helpful, please give it a star!** ⭐

Made with ❤️ by Nimra Abdulhaq

</div>
