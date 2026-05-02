# Starline Casino — CTF Write-Up

**Flag:** `CBC{st0p_gambling_st4rt_predicting!!_f8cad9}`

---

## Soalnya

Kita dikasih akses ke casino virtual. Modal awal 1.000 kredit, tapi flag harganya 50.000 kredit. Ada dua permainan: roulette dan slots.

Setiap main, server selalu nampilkan **ticket id** seperti ini:

```
ticket id: 3d339ba2
```

---

## Masalahnya

Server pakai `random.Random` bawaan Python untuk generate angka acak. Library ini **bisa diprediksi** — kalau kita udah kumpulkan 624 output-nya, kita bisa tahu semua angka berikutnya.

Ticket id yang ditampilkan server itu **langsung dari output RNG** — jadi kita bisa kumpulkan dari situ.

---

## Solusinya

1. **Main slots 624 kali** dengan taruhan minimum (1 kredit) → kumpulkan semua ticket id
2. **Rekonstruksi state RNG** server dari 624 ticket id tersebut
3. **Prediksi angka roulette** berikutnya → taruh semua kredit → pasti menang (36×)
4. Dua putaran roulette sudah cukup untuk melampaui 50.000 kredit

---

## Hasil Eksekusi

```
$ python3 exploit.py crypto.cbd2026.cloud 1337

[*] Phase 1: Collecting 624 ticket IDs from slots...
[+] Phase 1 done. 624 tickets. Balance: 640

[*] Phase 2: Cloning MT19937...
[+] RNG cloned!

[*] Round 1: bet 640 on 26  →  Hit 26!  →  Balance: 23040
[*] Round 2: bet 23040 on 13  →  Hit 13!  →  Balance: 829440

[+] FLAG: CBC{st0p_gambling_st4rt_predicting!!_f8cad9}
```

---

## Kesimpulan

`random.Random` Python tidak aman untuk keperluan keamanan karena outputnya bisa diprediksi setelah 624 angka terkumpul. Selain itu, server langsung menampilkan output RNG mentah lewat ticket id — ini yang membuat serangan ini bisa dilakukan.

---

*Write-up oleh: [nama kamu] | CTF: [nama event]*
