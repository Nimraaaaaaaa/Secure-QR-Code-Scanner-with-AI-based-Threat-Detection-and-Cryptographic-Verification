# 🔐 Secure QR Code Scanner with AI-based Threat Detection and Cryptographic Verification

<div align="center">

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AI Powered](https://img.shields.io/badge/AI-Powered-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)
![Security](https://img.shields.io/badge/Security-Cryptographic-1A6B3C?style=for-the-badge&logo=letsencrypt&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)

**A powerful, intelligent QR code scanner that combines AI-based threat detection with cryptographic verification to protect users from malicious QR codes.**

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [Architecture](#-architecture) • [Tech Stack](#-tech-stack) • [Contributing](#-contributing)

</div>

---

## 📌 Overview

QR codes have become ubiquitous in our daily lives — from payment systems to website links. However, they also present a significant security risk: malicious actors can embed harmful URLs, phishing links, or malware payloads inside QR codes that are visually indistinguishable from legitimate ones.

This project addresses that problem by building a **Secure QR Code Scanner** that:

- 🤖 Uses **AI/ML models** to detect malicious or suspicious content embedded in QR codes
- 🔐 Applies **cryptographic verification** (digital signatures / hashing) to ensure QR code integrity
- ⚠️ Alerts users in real-time when a threat is detected
- 📊 Logs and reports scan history with threat analysis

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔍 **QR Code Scanning** | Scan QR codes from camera feed or uploaded images |
| 🤖 **AI Threat Detection** | ML model classifies URLs/content as safe, suspicious, or malicious |
| 🔐 **Cryptographic Verification** | RSA/SHA-based digital signature verification for trusted QR codes |
| 🌐 **URL Analysis** | Deep inspection of embedded URLs for phishing, malware, and spoofing |
| 📋 **Scan History** | Stores past scans with threat level and timestamp |
| 🚨 **Real-time Alerts** | Instant visual/audio warnings when a threat is detected |
| 📊 **Threat Reports** | Generates detailed threat analysis reports |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│                  User Interface                  │
│         (Camera / Image Upload / CLI)            │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│              QR Code Decoder Module              │
│           (pyzbar / OpenCV / ZXing)              │
└──────────┬───────────────────────┬──────────────┘
           │                       │
           ▼                       ▼
┌──────────────────┐     ┌─────────────────────────┐
│  Cryptographic   │     │   AI Threat Detection   │
│  Verification    │     │        Engine           │
│  (RSA / SHA-256) │     │  (ML Model / NLP / API) │
└──────────┬───────┘     └────────────┬────────────┘
           │                          │
           └─────────────┬────────────┘
                         ▼
              ┌─────────────────────┐
              │   Result & Alerts   │
              │  (Safe / Warning /  │
              │     Malicious)      │
              └─────────────────────┘
```

---

## 🛠️ Tech Stack

- **Language:** Python 3.9+
- **QR Decoding:** `pyzbar`, `OpenCV`, `Pillow`
- **AI / ML:** `scikit-learn` / `TensorFlow` / `transformers` (for threat classification)
- **Cryptography:** `cryptography` library (RSA, SHA-256 digital signatures)
- **URL Analysis:** VirusTotal API / Google Safe Browsing API
- **GUI / Interface:** `tkinter` / `Streamlit` / CLI
- **Database:** SQLite / Firebase (scan history logging)

---

## 📦 Installation

### Prerequisites

- Python 3.9 or higher
- pip package manager
- (Optional) A VirusTotal API key for URL analysis

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/Nimraaaaaaaa/Secure-QR-Code-Scanner-with-AI-based-Threat-Detection-and-Cryptographic-Verification.git

# 2. Navigate to the project directory
cd Secure-QR-Code-Scanner-with-AI-based-Threat-Detection-and-Cryptographic-Verification

# 3. Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# 4. Install dependencies
pip install -r requirements.txt

# 5. Set up environment variables
cp .env.example .env
# Edit .env and add your API keys (VirusTotal, etc.)
```

---

## 🚀 Usage

### Scan a QR Code from an Image File

```bash
python main.py --image path/to/qr_image.png
```

### Scan Using Webcam (Live Mode)

```bash
python main.py --camera
```

### Verify Cryptographic Signature of a QR Code

```bash
python main.py --verify --image path/to/signed_qr.png --pubkey public_key.pem
```

### Generate a Cryptographically Signed QR Code

```bash
python main.py --generate --data "https://example.com" --privkey private_key.pem
```

### Run the Streamlit Web Interface

```bash
streamlit run app.py
```

---

## 🔐 How Cryptographic Verification Works

1. A **trusted QR code** is generated by signing its content with a **private RSA key**
2. The signature is embedded within the QR code (or stored separately)
3. When scanned, the app uses the corresponding **public key** to verify the signature
4. If the signature is **valid** → QR code is trusted ✅
5. If the signature is **invalid or missing** → QR code is flagged ⚠️

```
[QR Content] + [Private Key] ──→ Digital Signature
[QR Content] + [Signature] + [Public Key] ──→ Verified / Tampered
```

---

## 🤖 How AI Threat Detection Works

The AI module analyzes the decoded QR content using:

1. **URL Feature Extraction** — extracts features like domain age, HTTPS status, URL length, suspicious keywords
2. **ML Classification** — a trained model (Random Forest / Neural Network) classifies the URL as:
   - ✅ `SAFE` — No threat detected
   - ⚠️ `SUSPICIOUS` — Proceed with caution
   - 🚨 `MALICIOUS` — Block and alert user
3. **API Cross-Check** — optionally validates against VirusTotal / Google Safe Browsing for enhanced accuracy

---

## 📁 Project Structure

```
📦 Secure-QR-Code-Scanner/
├── 📂 src/
│   ├── scanner.py            # QR code scanning & decoding
│   ├── threat_detector.py    # AI threat detection engine
│   ├── crypto_verify.py      # Cryptographic verification module
│   ├── url_analyzer.py       # URL feature extraction & API checks
│   └── logger.py             # Scan history & logging
├── 📂 models/
│   └── threat_classifier.pkl # Trained ML model
├── 📂 keys/
│   ├── private_key.pem       # RSA private key (DO NOT SHARE)
│   └── public_key.pem        # RSA public key
├── 📂 data/
│   └── training_data.csv     # Dataset for model training
├── 📂 tests/
│   └── test_scanner.py       # Unit tests
├── app.py                    # Streamlit web interface
├── main.py                   # CLI entry point
├── requirements.txt          # Python dependencies
├── .env.example              # Environment variables template
└── README.md
```

---

## 🧪 Running Tests

```bash
python -m pytest tests/ -v
```

---

## 📸 Screenshots

> *(Add screenshots of your application here)*

| Live Scanner | Threat Alert | Scan Report |
|:---:|:---:|:---:|
| ![Scanner](screenshots/scanner.png) | ![Alert](screenshots/alert.png) | ![Report](screenshots/report.png) |

---

## 🔮 Future Enhancements

- [ ] Mobile app (Android/iOS) version
- [ ] Browser extension for inline QR scanning
- [ ] Blockchain-based QR certificate authority
- [ ] Support for NFC-based verification
- [ ] Multi-language support
- [ ] Real-time threat intelligence feed integration

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'Add: your feature description'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

Please make sure your code follows PEP8 standards and includes appropriate tests.

---

## ⚠️ Disclaimer

This tool is intended for **educational and security research purposes**. The authors are not responsible for any misuse. Always obtain proper authorization before scanning QR codes in production systems.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

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
