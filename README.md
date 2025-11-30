# ✅ Webhook Instructions – Upmedik (FINAL VERSION)

This document explains how to securely send SOAP notes to the Upmedik webhook.

---

## 🔗 Webhook URL

https://upmedik.com/webhook/

---

## ✅ Required Fields

| Field Name         | Type     | Notes |
|--------------------|----------|-------|
| id_pasien_visit    | INT      | Visit ID |
| id_reg             | INT      | Registration ID |
| no_rm_pasien       | VARCHAR  | Patient medical record number |
| id_dept            | INT      | Department ID |
| tipe_cppt          | STRING   | CPPT type (e.g. "1" or "soap") |
| subjective         | TEXT     | Subjective notes |
| objective          | TEXT     | Objective notes |
| assessment         | TEXT     | Assessment notes |
| plan               | TEXT     | Plan notes |
| created_by         | STRING   | Source user/system |
| no_rm_dokter       | VARCHAR  | Doctor medical record number |
| nama_dokter        | STRING   | Doctor full name |

⚠️ ALL FIELDS ABOVE ARE REQUIRED AND MUST BE PRESENT.

---

## ✅ Example JSON Payload (FULL)

```json
{
  "id_pasien_visit": 12345,
  "id_reg": 98765,
  "no_rm_pasien": "RM0001234",
  "id_dept": 12,
  "tipe_cppt": "1",
  "subjective": "Pasien mengeluh pusing sejak 2 hari terakhir.",
  "objective": "Tekanan darah 110/70 mmHg, nadi 82x/menit.",
  "assessment": "Kemungkinan vertigo perifer.",
  "plan": "Berikan obat antihistamin.",
  "created_by": "dr ALDY NUGRAHA",
  "no_rm_dokter": "100687",
  "nama_dokter": "dr ALDY NUGRAHA"
}
```

---

## 🔐 Security Requirements

All data must be encrypted and authenticated before sending.

- Encryption Algorithm: AES-256-CBC
- HMAC Algorithm: SHA256
- API Key: Must be included in the POST request

You will receive the following from Upmedik:

- ENCRYPTION_KEY
- HMAC_SECRET
- API_KEY

---

## ✅ PHP Example (COPY–PASTE READY)

```php
<?php
$ENCRYPTION_KEY = "YOUR_32_CHAR_ENCRYPTION_KEY";
$HMAC_SECRET    = "YOUR_32_CHAR_HMAC_SECRET";
$API_KEY        = "YOUR_API_KEY";

$payload = [
    "id_pasien_visit" => 12345,
    "id_reg"          => 98765,
    "no_rm_pasien"    => "RM0001234",
    "id_dept"         => 12,
    "tipe_cppt"       => "1",
    "subjective"      => "Pasien mengeluh pusing sejak 2 hari terakhir.",
    "objective"       => "Tekanan darah 110/70 mmHg, nadi 82x/menit.",
    "assessment"      => "Kemungkinan vertigo perifer.",
    "plan"            => "Berikan obat antihistamin.",
    "created_by"      => "dr ALDY NUGRAHA",
    "no_rm_dokter"    => "100687",
    "nama_dokter"     => "dr ALDY NUGRAHA"
];

$iv = openssl_random_pseudo_bytes(16);
$encrypted = openssl_encrypt(json_encode($payload), "AES-256-CBC", $ENCRYPTION_KEY, 0, $iv);
$encrypted_base64 = base64_encode($iv . $encrypted);

$signature = hash_hmac("sha256", $encrypted_base64, $HMAC_SECRET);

$ch = curl_init("https://upmedik.com/webhook/index.php");
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query([
    "api_key"   => $API_KEY,
    "data"      => $encrypted_base64,
    "signature" => $signature
]));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

echo $response;
?>
```

---

## ✅ Python Example (COPY–PASTE READY)

```python
import json, base64, hashlib, hmac, requests
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

ENCRYPTION_KEY = b'YOUR_32_CHAR_ENCRYPTION_KEY'
HMAC_SECRET    = b'YOUR_32_CHAR_HMAC_SECRET'
API_KEY        = "YOUR_API_KEY"
WEBHOOK_URL    = "https://upmedik.com/webhook/index.php"

payload = {
    "id_pasien_visit": 12345,
    "id_reg": 98765,
    "no_rm_pasien": "RM0001234",
    "id_dept": 12,
    "tipe_cppt": "1",
    "subjective": "Pasien mengeluh pusing sejak 2 hari terakhir.",
    "objective": "Tekanan darah 110/70 mmHg, nadi 82x/menit.",
    "assessment": "Kemungkinan vertigo perifer.",
    "plan": "Berikan obat antihistamin.",
    "created_by": "dr ALDY NUGRAHA",
    "no_rm_dokter": "100687",
    "nama_dokter": "dr ALDY NUGRAHA"
}

iv = get_random_bytes(16)
cipher = AES.new(ENCRYPTION_KEY, AES.MODE_CBC, iv)
data_bytes = json.dumps(payload).encode("utf-8")
pad_len = 16 - len(data_bytes) % 16
data_bytes += bytes([pad_len])*pad_len
encrypted = cipher.encrypt(data_bytes)
encrypted_base64 = base64.b64encode(iv + encrypted).decode()

signature = hmac.new(HMAC_SECRET, encrypted_base64.encode(), hashlib.sha256).hexdigest()

response = requests.post(WEBHOOK_URL, data={
    "api_key": API_KEY,
    "data": encrypted_base64,
    "signature": signature
})

print(response.text)
```

---

## ✅ JavaScript (Node.js) Example (COPY–PASTE READY)

```javascript
const crypto = require("crypto");
const axios = require("axios");

const ENCRYPTION_KEY = "YOUR_32_CHAR_ENCRYPTION_KEY";
const HMAC_SECRET    = "YOUR_32_CHAR_HMAC_SECRET";
const API_KEY        = "YOUR_API_KEY";
const WEBHOOK_URL    = "https://upmedik.com/webhook/index.php";

const payload = {
    id_pasien_visit: 12345,
    id_reg: 98765,
    no_rm_pasien: "RM0001234",
    id_dept: 12,
    tipe_cppt: "1",
    subjective: "Pasien mengeluh pusing sejak 2 hari terakhir.",
    objective: "Tekanan darah 110/70 mmHg, nadi 82x/menit.",
    assessment: "Kemungkinan vertigo perifer.",
    plan: "Berikan obat antihistamin.",
    created_by: "dr ALDY NUGRAHA",
    no_rm_dokter: "100687",
    nama_dokter: "dr ALDY NUGRAHA"
};

const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv("aes-256-cbc", Buffer.from(ENCRYPTION_KEY), iv);

let encrypted = cipher.update(JSON.stringify(payload), "utf8", "base64");
encrypted += cipher.final("base64");

const encrypted_base64 = Buffer.concat([
    iv,
    Buffer.from(encrypted, "base64")
]).toString("base64");

const signature = crypto
    .createHmac("sha256", HMAC_SECRET)
    .update(encrypted_base64)
    .digest("hex");

axios.post(WEBHOOK_URL, new URLSearchParams({
    api_key: API_KEY,
    data: encrypted_base64,
    signature: signature
})).then(res => {
    console.log(res.data);
}).catch(err => {
    console.error(err.response ? err.response.data : err.message);
});
```
