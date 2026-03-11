#  Payment Gateway API

## Ringkas
Semua request dibuat ke endpoint `/api/v1` menggunakan parameter `request`:

1. `new` untuk buat transaksi
2. `status` untuk cek status transaksi
3. `cancel` untuk batalkan transaksi
4. `profile` untuk lihat data akun merchant
5. `withdraw_auto` untuk tarik saldo otomatis

Server akan memanggil `callback_url` milik merchant jika transaksi sukses.

## Base URL
```
https://pay.zannstore.com
```

## Pendaftaran Akun
Jika belum punya akun, silakan daftar terlebih dahulu di:
```
https://pay.zannstore.com
```

## Autentikasi dan Signature
Semua request memakai signature berbasis SHA-256.

Aturan signature:

1. Transaksi `new`, `status`, `cancel`
   - `signature = sha256(merchant + secret_key + trx_id)`
2. `profile`
   - `signature = sha256(merchant + secret_key + pin)`
3. `withdraw_auto`
   - `signature = sha256(merchant + pin)`

Catatan:
1. `merchant` dan `secret_key` berasal dari akun merchant.
2. `trx_id` harus unik per merchant, sistem menolak duplikat.

## Endpoint Utama

### 1. Buat Transaksi
`POST /api/v1`

Parameter wajib:
1. `request` = `new`
2. `merchant`
3. `trx_id`
4. `payment`
5. `signature`

Parameter tambahan (opsional):
1. `amount` (integer)
2. `type_fee` = `merchant` atau `user` (default `merchant`)
3. `callback_url` (default `-`)
4. `note` (default kosong)
5. `expired_time` (default `600`)

Format `expired_time`:
1. Angka detik, contoh `600`
2. Format `Xm` atau `Xj` (menit atau jam), contoh `30m` atau `2j`

Daftar `payment`:
1. `QRIS`
2. `QRIS2`
3. `QRIS3`
4. `BRIVA`
5. `MANDIRIVA`
6. `BNCVA`
7. `PERMATAVA`
8. `BNIVA`
9. `SINARMASVA`
10. `DANAMONVA`
11. `CIMBVA`
12. `MUAMALATVA`
13. `BSIVA`
14. `MAYBANKVA`
15. `ALFAMART`
16. `INDOMARET`
17. `PAYLINK`

Respon sukses (contoh struktur umum):
```json
{
  "status": true,
  "message": "Berhasil membuat transaksi",
  "data": {
    "trx_svr": "TX_SERVER",
    "trx_id": "TRX_MERCHANT",
    "method_code": "BRIVA",
    "method_name": "BANK RAKYAT INDONESIA VA",
    "status": "Pending",
    "amount": 10000,
    "diterima": 6500,
    "fee": 3500,
    "type_fee": "merchant",
    "virtual_account": "1234567890",
    "note": "",
    "checkout_url": "https://.../invoice/...",
    "created_at": "2026-03-11 19:00:00",
    "expired_at": "2026-03-11 19:10:00",
    "callback_url": "https://merchant.com/callback"
  }
}
```

Catatan:
1. Field di `data` tergantung metode pembayaran.
2. QRIS menggunakan `qr_url` dan `qr_content`.
3. Retail menggunakan `payment_code`.
4. VA menggunakan `virtual_account`.
5. Paylink menggunakan `checkout_url` menuju halaman pembayaran internal.

### 2. Cek Status Transaksi
`POST /api/v1`

Parameter wajib:
1. `request` = `status`
2. `merchant`
3. `trx_id`
4. `signature`

Respon sukses:
```json
{
  "status": true,
  "msg": "Yeay, detail transaksi berhasil diambil",
  "data": {
    "trx_svr": "TX_SERVER",
    "trx_id": "TRX_MERCHANT",
    "metode_name": "BRIVA",
    "status": "Pending",
    "amount": 10000,
    "fee": 3500,
    "unique_code": 0,
    "diterima": 6500,
    "note": "",
    "type_fee": "merchant",
    "issuer_bank": "002",
    "rrn": "-",
    "paid_at": "-",
    "created_at": "2026-03-11 19:00:00",
    "expired_at": "2026-03-11 19:10:00"
  }
}
```

### 3. Batalkan Transaksi
`POST /api/v1`

Parameter wajib:
1. `request` = `cancel`
2. `merchant`
3. `trx_id`
4. `signature`

Catatan:
1. Hanya transaksi `Pending` yang bisa dibatalkan.

### 4. Ambil Profile Merchant
`POST /api/v1`

Parameter wajib:
1. `request` = `profile`
2. `merchant`
3. `pin`
4. `secret_key`
5. `signature` = `sha256(merchant + secret_key + pin)`

Respon sukses:
```json
{
  "status": true,
  "msg": "Yeay, data profile berhasil diambil",
  "data": {
    "nama_pemilik": "Nama",
    "merchant": "MERCHANT_ID",
    "saldo_kliring": "Rp 0",
    "saldo_tersedia": "Rp 0",
    "whatsapp": "628xxxx",
    "email": "a@b.com",
    "created_at": "2026-03-11 19:00:00"
  }
}
```

### 5. Withdraw Otomatis
`POST /api/v1`

Parameter wajib:
1. `request` = `withdraw_auto`
2. `merchant`
3. `pin`
4. `signature` = `sha256(merchant + pin)`
5. `amount` (integer)
6. `type_bank` = `bank` atau `emoney`
7. `tujuan` (no rekening / no e-wallet)
8. `bank_code`

Batasan:
1. Bank: min Rp 10.000, max Rp 40.000.000
2. E-money: min Rp 1.000, max Rp 2.000.000
3. Biaya admin bervariasi dan dihitung otomatis di server

Respon sukses:
```json
{
  "status": true,
  "msg": "Withdraw berhasil diproses.",
  "trx_id": "ZNWD...",
  "nominal": 50000,
  "biaya": 3000,
  "total": 53000,
  "saldo_awal": 100000,
  "saldo_akhir": 47000,
  "tujuan": "1234567890",
  "jenis": "BANK",
  "bank_code": "BCA",
  "account_name": "NAMA REKENING",
  "bank_name": "BANK BCA"
}
```

## Tabel Metode Pembayaran

Metode | Min | Max | Fee Default | Catatan
---|---:|---:|---:|---
`QRIS` | 100 | 5.000.000 | 0,7% | `expired_time` harus format `Xm` atau `Xj`
`QRIS2` | 500 | 10.000.000 | 0,1% atau 0,4% + kode unik | Kode unik 1–999, fee 0,1% jika amount ≤ 499.000, 0,4% jika > 499.000
`QRIS3` | 100 | 10.000.000 | 0% atau 0,4% + kode unik | Kode unik 0–999, fee 0% jika amount ≤ 499.999, 0,4% jika > 499.999
`BRIVA` | 10.000 | 10.000.000 | 3.500 | VA BRI
`MANDIRIVA` | 10.000 | 10.000.000 | 3.500 | VA Mandiri
`BNCVA` | 10.000 | 2.000.000 | 3.500 | VA Neo Commerce
`PERMATAVA` | 10.000 | 10.000.000 | 3.000 | VA Permata
`BNIVA` | 10.000 | 10.000.000 | 3.500 | VA BNI
`SINARMASVA` | 10.000 | 10.000.000 | 2.500 | VA Sinarmas
`DANAMONVA` | 10.000 | 10.000.000 | 3.000 | VA Danamon
`CIMBVA` | 10.000 | 10.000.000 | 3.500 | VA CIMB
`MUAMALATVA` | 10.000 | 10.000.000 | 3.000 | VA Muamalat
`BSIVA` | 10.000 | 10.000.000 | 3.500 | VA BSI
`MAYBANKVA` | 10.000 | 10.000.000 | 3.000 | VA Maybank
`ALFAMART` | 10.000 | 10.000.000 | 2.500 | Retail Alfamart
`INDOMARET` | 10.000 | 10.000.000 | 2.500 | Retail Indomaret
`PAYLINK` | 1.000 | - | 0 | Link pembayaran internal

Catatan fee:
1. Jika `type_fee=merchant`, fee dipotong dari `diterima`.
2. Jika `type_fee=user`, fee ditambahkan ke nominal yang harus dibayar user.

## Callback ke Merchant (Webhook)

Server akan mengirim `POST` JSON ke `callback_url` saat transaksi sukses untuk QRIS.

### QRIS
```json
{
  "data": {
    "trx_svr": "TX_SERVER",
    "trx_id": "TRX_MERCHANT",
    "status": "Success",
    "amount": "10000",
    "fee": "70",
    "type_fee": "merchant",
    "issuer_bank": "BANK",
    "rrn": "RRN",
    "diterima": "9930",
    "paid_at": "2026-03-11 19:00:00"
  }
}
```

## API List Penarikan (Publik)

### List Kode E-Wallet
`GET /api/kode-ewallet`

Response: daftar kode e-wallet.

### List Kode Bank
`GET /api/kode-bank`

Response: daftar kode bank.

## Contoh Request

### Buat transaksi QRIS
```bash
curl -X POST "https://pay.zannstore.com/api/v1" \
  -d "request=new" \
  -d "merchant=MERCHANT_ID" \
  -d "trx_id=INV-1001" \
  -d "payment=QRIS" \
  -d "amount=10000" \
  -d "type_fee=merchant" \
  -d "expired_time=30m" \
  -d "callback_url=https://merchant.com/callback" \
  -d "signature=SHA256(merchant+secret_key+trx_id)"
```

### Cek status
```bash
curl -X POST "https://pay.zannstore.com/api/v1" \
  -d "request=status" \
  -d "merchant=MERCHANT_ID" \
  -d "trx_id=INV-1001" \
  -d "signature=SHA256(merchant+secret_key+trx_id)"
```

### Batalkan transaksi
```bash
curl -X POST "https://pay.zannstore.com/api/v1" \
  -d "request=cancel" \
  -d "merchant=MERCHANT_ID" \
  -d "trx_id=INV-1001" \
  -d "signature=SHA256(merchant+secret_key+trx_id)"
```

### Profile merchant
```bash
curl -X POST "https://pay.zannstore.com/api/v1" \
  -d "request=profile" \
  -d "merchant=MERCHANT_ID" \
  -d "pin=123456" \
  -d "secret_key=SECRET" \
  -d "signature=SHA256(merchant+secret_key+pin)"
```

### Withdraw otomatis
```bash
curl -X POST "https://pay.zannstore.com/api/v1" \
  -d "request=withdraw_auto" \
  -d "merchant=MERCHANT_ID" \
  -d "pin=123456" \
  -d "signature=SHA256(merchant+pin)" \
  -d "amount=50000" \
  -d "type_bank=bank" \
  -d "bank_code=BCA" \
  -d "tujuan=1234567890"
```
