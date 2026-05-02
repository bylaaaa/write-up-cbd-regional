# Waifu Shop — CTF Write-Up

**Flag:** `CBC{enterprise_is_min333_4d0b8a}`

---

## Soalnya

Kita diberi akses ke toko online bertema Azur Lane. Ada beberapa item yang dijual, tapi item paling mahal — **Shinano (999.999 kredit)** — tidak tersedia (`available: False`).

Untuk dapat flag, kita harus berhasil "memesan" Shinano dengan harga **0 kredit**.

---

## Cara Kerja Aplikasi

Saat checkout, server membuat **token order** seperti ini:

```
item=enterprise_gold&price=004800&buyer=guest&ship=standard
```

Teks ini dienkripsi dengan **AES-CTR** lalu di-encode ke Base64 dan dikirim ke browser. Saat klaim, token dikirim balik ke server, didekripsi, lalu dicek isinya.

---

## Masalahnya

AES-CTR bekerja dengan cara **XOR** plaintext dengan keystream:

```
ciphertext = plaintext XOR keystream
```

Kelemahannya: **keystream selalu sama** karena KEY dan NONCE tidak berubah selama server hidup. Ini artinya kalau kita tahu plaintext aslinya, kita bisa menghitung keystream-nya, lalu pakai keystream itu untuk membuat ciphertext palsu dengan isi apapun yang kita mau.

---

## Serangan: CTR Bitflip

Kita punya token dari order `enterprise_gold` yang plaintextnya kita tahu persis:

```
item=enterprise_gold&price=004800&buyer=guest&ship=standard
```

Target kita adalah memalsukan token dengan isi:

```
item=celestial_waifu&price=000000&buyer=guest&ship=standard
```

Panjang keduanya sama persis (59 byte), jadi kita bisa lakukan ini:

```
keystream    = ciphertext XOR plaintext_asli
forged_token = keystream  XOR plaintext_target
```

Hasilnya adalah token baru yang kalau didekripsi server, isinya adalah order Shinano dengan harga 0.

---

## Hasil Eksekusi

```
$ python3 exploit.py https://waifu-shop.cbd2026.cloud/

[*] Placing order for 'enterprise_gold'...
[+] Got token: GdmLYgZZbUqtTgf2NGkd7IupX1ORRmFx_nMHup3cC9MR...

[+] Forged token: GdmLYgZfZlKtTwPtPHYnxI2vVUKRRmFx_nMHup3YA9MR...

[*] Submitting forged token to /claim...
[+] FLAG: CBC{enterprise_is_min333_4d0b8a}
```

---

## Kesimpulan

AES-CTR **tidak boleh** dipakai dengan KEY dan NONCE yang sama untuk mengenkripsi lebih dari satu pesan — apalagi kalau plaintextnya bisa ditebak. Karena keystream-nya statis, siapapun yang tahu satu plaintext bisa memalsukan pesan lain tanpa perlu tahu kuncinya.

Solusinya: gunakan NONCE acak yang berbeda setiap kali enkripsi, atau gunakan mode yang punya autentikasi seperti **AES-GCM**.

---

*Write-up oleh: ByLaaaa | CTF: CBD Regional S3*
