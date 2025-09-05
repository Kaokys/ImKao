---
title: THCTT 2025 Write‑up | Thailand Cyber Top Talent 2025 Junior [Qualifier]
published: 2025-09-01
description: "https://medium.com/@chsirawich098/thctt-2025-write-up-ea2df20ffdf5"
image: "./thctt.jpg"
tags: ["ctf", "Write-Up", "NCSA"]
category: Write-Up
draft: false
---

# THCTT 2025 Write‑up
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Lo_FLkyUdwdEDrMHpATQdw.png)
*Note: อันไหนทำได้เอามาหมด อันที่ไม่ได้เฉลยเพื่อนทำ*
![img](https://miro.medium.com/v2/resize:fit:330/format:webp/1*aH9jfGjNkQi6cNvSkQFm0Q.gif)
---
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*QreXzGtj7dLN-EwMbCo_Kg.png)
## 1. 6278
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*JO6t1vRIFH46E6dNYXmw3g.png)
โจทย์ไฟล์ `6278.zip` มา ในไฟล์นี้มีไฟล์ zip ซ้อนเข้าไปเรื่อยๆ เป้าหมายของผมคือการแตกไฟล์ zip จนถึงไฟล์สุดท้ายสุดท้าย
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*lIm_Cp4vPeJ8ipbf61_6vA.png)
![img](https://miro.medium.com/v2/resize:fit:576/format:webp/1*WN7Y8XP55D_93Zg6habLcw.png)
![img](https://miro.medium.com/v2/resize:fit:466/format:webp/1*zWOoCcDMAEEXrepObcmR_w.png)
![img](https://miro.medium.com/v2/resize:fit:404/format:webp/1*nwQ68KcwbTO6Kp5DInP54g.png)
ตอนแรกว่าจะสแปมเปิดไปเรื่อยๆ แต่ไม่น่าทันเวลา  
ผมเลยเขียนโค้ด Python ในการแตกไฟล์ zip แบบอัตโนมัติ ทีละชั้นไปเรื่อยๆ จนกว่าเจอไฟล์สุดท้าย

```python
import zipfile
import os

# ระบุพาธไฟล์ ZIP ที่เริ่มต้น
zip_path = '/path/to/your/6278.zip'
extracted_folder = '/path/to/extracted_files/'  # โฟลเดอร์ที่จะแตกไฟล์

# สร้างโฟลเดอร์เพื่อเก็บไฟล์ที่แตกออก
os.makedirs(extracted_folder, exist_ok=True)

# ฟังก์ชันในการแตกไฟล์ ZIP
def extract_zip(file_path, destination):
    with zipfile.ZipFile(file_path, 'r') as zip_ref:
        zip_ref.extractall(destination)
        return zip_ref.namelist()

# เริ่มแตกไฟล์ ZIP
extracted_files = extract_zip(zip_path, extracted_folder)
# ตรวจสอบว่ามีไฟล์ ZIP อยู่ในผลลัพธ์หรือไม่
while any(f.endswith('.zip') for f in extracted_files):
    # หาไฟล์ ZIP ที่ถัดไป
    next_zip_file = next(f for f in extracted_files if f.endswith('.zip'))
    next_zip_path = os.path.join(extracted_folder, next_zip_file)

    # แตกไฟล์ ZIP ถัดไป
    extracted_files = extract_zip(next_zip_path, extracted_folder)

# เมื่อหมดการแตกไฟล์แล้ว ให้แสดงผลลัพธ์
extracted_files
```

แต่โน้ตบุ๊คผมช้าผมเลยให้ ChatGPT แตกไฟล์ให้
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fE0peQ9zE1zqM-EhU8pWCg.png) 
GPT แตกไฟล์จนได้ `flag.txt` ผมเลยให้ GPT เปิดไฟล์ดูเลย…  
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JITDPCZc5rEa2uFTZnYa4g.png)

🚩หลังจากนั้นก็ได้ flag:

```
flag{n07h!in9_b347_j37_7w0_h01!d4y_627846261}
```

---

## 2. Recycle Secrets
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*tJwKJUqjW1O2rwlnpbwbXA.png)
โจทย์ให้ไฟล์ `recycle_secrets.zip` มา  
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*FYuiOS4hQWIFhGqVA8uQbw.png)
หลังจากแตกไฟล์จะได้เป็น `recycle_secrets.img` ซึ่งน่าจะเป็น disk image/forensic challenge โดย hint บอกว่าข้อมูลบางอย่างถูกซ่อนไว้ใน Recycle Bin ของ Windows
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*mF_nRt51nzAOLwedMhCY9w.png)
อย่างแรกตรวจดูว่าเป็นไฟล์อะไร ใช้ `file` ตรวจสอบ:

```bash
file recycle_secrets.img
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*45hLj-ZYB7sdqW9w7Kwnag.png)
ผลลัพธ์บอกว่าเป็น DOS/MBR boot sector, นั่นหมายถึงไฟล์นี้น่าจะเป็น FAT/NTFS image

hint บอกว่าข้อมูลบางอย่างถูกซ่อนไว้ใน Recycle Bin ของ Windows  
และใน Windows, Recycle Bin จะมีไฟล์ `$Ixxxx` และ `$Rxxxx`  
เราลองสแกน `strings`:

```bash
strings recycle_secrets.img | grep '\$R'
```

พบ `$R` และ `$I` หลายไฟล์ → บ่งบอกว่ามีไฟล์ถูกลบและ flag น่าจะซ่อนในนั้น

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Y-2_b49OMj76hJ-zFNkzgQ.png)

เลยลองให้ ChatGPT เขียน Python หา sequences:

```python
import re
data = open("recycle_secrets.img","rb").read()
for m in re.finditer(rb"[A-Za-z0-9]{20,60}", data):
    print(m.start(), m.group(0).decode('ascii', errors='replace'))
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Y-2_b49OMj76hJ-zFNkzgQ.png)
เจอ string ที่ต่างจากอันอื่นแบบชัดเจน:

```
83312 246ovpbPtH1VEa1wFrVY9mqfuwJMZiku419ZZjz
```

เลยลองเอาเข้า CyberChef ให้ถอดรหัสให้
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*M74CzxmlFqKJiBYm2tqT1w.png)

🚩ได้ flag:

```
flag{recycle_bin_caught_you}
```

---

## 3. hideden_palyoad
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*uibzB275AWWZf0W0hY8GFg.png)
ข้อนี้ให้ไฟล์ zip
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*_RO_IbHY6wlce3-f9mJbSg.png)
ที่แตกไฟล์ออกได้เป็น `hideden_palyoad.iso`  
![img](https://miro.medium.com/v2/resize:fit:616/format:webp/1*-MlzL4XenRTR0GbEmODKaA.png)
ไฟล์ที่ได้มาเป็น hidden_payload.iso
ISO คืออิมเมจของแผ่นดิสก์ สามารถ mount หรือแตกไฟล์ออกมาเพื่อดูโครงสร้างไฟล์ข้างในได้ เลยลองแตกไไฟล์ดู

```bash
7z x hidden_payload.iso
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*iYDmQThcej65n7ZPducv-Q.png)

จาก log ที่ได้มา:

```
ISO size = 370,688 bytes (~362 KB)
Physical size inside ISO = 61,440 bytes (~60 KB)
Tail size = 309,248 bytes (~302 KB)
```

แปลว่า ISO จริง ๆ มีแค่ 61,440 bytes ส่วนท้าย (tail) อีก 309,248 bytes มีข้อมูลแอบซ่อนอยู่

เลยลองดึง tail ออกมา (เอาเฉพาะส่วนหลัง 61440 ไบต์แรกเก็บออกมา)
```bash
dd if=hidden_payload.iso of=hidden_tail.bin bs=1 skip=61440
```
bs=1 คืออ่านทีละ 1 ไบต์ (แม้จะช้าแต่ตรงที่สุด)
skip=61440 หมายถึงข้าม 61,440 ไบต์แรกออกไป

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Nl78amAQgADiriak15zWaQ.png)

หลังรันเสร็จจะได้ไฟล์ hidden_tail.bin ขนาด 309,248 bytes (เท่ากับ Tail Size ที่ 7z แจ้งไว้)

![img](![alt text](image.png))

เมื่อดูด้วย `hexdump` จะเห็นว่าไฟล์แบ่งเป็นบล็อกขนาด 16 bytes พอดี → ลักษณะเข้ากับ AES‑ECB (เพราะ AES ใช้ block size 16)

```bash
dd hexdump hidden_tail.bin
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*4RqMlKFIvJAmNATaROvAzg.png)

ตอนนี้ยังไม่รู้ key แต่คาดว่าเป็น pattern ง่าย ๆ เช่น `"A"*16`, `"B"*16` …  
เลยเขียน Python brute‑force หา key:

```python
from Crypto.Cipher import AES
import string

ct = open("hidden_tail.bin","rb").read()

def printable_ratio(b: bytes) -> float:
    return sum(32 <= x < 127 or x in (9,10,13) for x in b) / len(b)

candidates = [c*16 for c in (string.ascii_letters + string.digits)]

for k in candidates:
    key = k.encode()
    aes = AES.new(key, AES.MODE_ECB)
    pt = b''.join([aes.decrypt(ct[i:i+16]) for i in range(0, len(ct), 16)])
    if printable_ratio(pt[:512]) > 0.8:
        print(f"[+] Possible key: {k}")
        print(pt[:200].decode(errors="ignore"))
        break
```

เจอคีย์ที่ถูกต้องคือ `"J"*16` เพราะผลลัพธ์หลังถอดรหัสขึ้นต้นว่า:

```
$key = 0x4A
$data = @(0x2F,0x3C,0x2A,0x28,...)
```

ตรงกับ ASCII `'J'` (0x4A) → ยืนยันว่า key ถูกต้อง

เมื่อรู้ key แล้ว ถอดทั้งไฟล์:

```python
from Crypto.Cipher import AES

ct = open("hidden_tail.bin","rb").read()
key = b"J"*16
aes = AES.new(key, AES.MODE_ECB)
pt = b''.join([aes.decrypt(ct[i:i+16]) for i in range(0, len(ct), 16)])

open("decrypted.bin","wb").write(pt)
```

ผลลัพธ์ `decrypted.bin` → ข้างในคือ PowerShell script  
ถอด XOR:

```python
arr = [0x2F,0x3C,0x2A,0x28,...]
decoded = "".join(chr(b ^ 0x4A) for b in arr)
print(decoded)
```

🚩ได้ Flag:

```
flag{powershell_xor_hidden_so_easy}
```

---
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ChU7KrqO7j7aGU1PHKz1Sw.png)
## 4. sPhone Pro Max
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*FAzvJWB0472P_Z5gIuRXbA.png)
โจทย์ให้ไฟล์ `.ipa` มา: `thctt2025_junior_mobile1_sPhoneProMax.ipa`  
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JAD1W1W8xivvH2mjDnQ9tw.png)
ไฟล์ `.ipa` คือ iOS Application Archive จริง ๆ แล้วก็คือ ZIP ที่มีโครงสร้าง `Payload/…`  


ผมเลยแตกไฟล์ออกมา:

```bash
unzip thctt2025_junior_mobile1_sPhoneProMax.ipa
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*0cm1xNZhco8A_ZGQapiphw.png)

ได้เป็นโฟลเดอร์ Payload ทีด้านในมี โฟลเดอร์ sPhoneProMax.app แล้วหลังจากเข้าไปก็ได้ไฟล์:

![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*iHWmCjd14oQAFWnn5oEotA.png)
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*iHWmCjd14oQAFWnn5oEotA.png)
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ZUCYAM3bGhvRiSEbOCjJtw.png)

หลังได้ไฟล์แล้วผมเลยลองหา flag โดยใช้ strings แล้ว grep หา:
ซึ่ง flag pattern ที่โจทย์ให้มาคือ flag{}

อย่างแรกผม change directory เข้ามาในโฟลเดอร์ sPhoneProMax.app


```bash
cd Payload/sPhoneProMax.app/
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*kKSKNbfUkrugIx5hv4CohQ.png)

หลังจากนั้นผมก็ strings grep ไฟล์ทั้งหมดในโฟลเดอร์
```bash
cd strings * | grep "flag{"
```
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*uumJJCW092_lZmpo_NgeSQ.png)

🚩เจอ flag:

```
flag{b99182477f02207f6e26cf839a20ebfe}
```

---

## 5. Black Rock Shooter
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*4lyxw-m7T04enjE5va9o5w.png)
ข้อนี้ให้ไฟล์ zip มา: `thctt2025_junior_mobile2_BlackRockShooter.zip`  

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*MQLUdMJLbpiN6IFqXzOVIA.png)

หลังจากแตกไฟล์ก็ได้เป็นไฟล์ : thctt2025_junior_mobile2_BlackRockShooter.apk

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JCZTrDmocP2w0EoCeRNBFA.png)

ไฟล์ .apk ก็คือ ZIP เช่นกัน สามารถแตกไฟล์ได้:

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*q9paFB2gnKqo-5Mkpn1GXg.png)

ข้อนี้ไฟล์เยอะผมเลยใช้find ก่อนใช้ strings แล้ว grep หา:

```bash
find . -type f -exec strings -a {} \; | grep "flag{
```

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*1chVaDe9RPXyZF97cTl4wg.png)

ผลคือ เจอ flag หลายอันมาก แต่ทุกอันเหมือนกันหมดคือ:

```
flag{b8b4efee48be709cd5b88ad9664e763f}
```

(แต่ตอนพิมพ์บรรทัดบอกได้ flag อันเดิมของข้อก่อน ซึ่งน่าจะผิดเล็กน้อย) เจ้าของระบุว่า:



🚩ได้ flag เป็น

```
flag{b99182477f02207f6e26cf839a20ebfe}
```

---
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*OmPqE8AX8ySMO-B2MyIGuQ.png)
## 6. medium_rare
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*2IWPziSSXi4tTeJ0GrmSJA.png)
![img](https://miro.medium.com/v2/resize:fit:570/format:webp/1*iiBLaWOm8_3mWW2NsP7HMg.png)
โจทย์ให้ไฟล์ `mediu_rare_challenge.zip` มา  
แตกไฟล์แล้วได้:

- `test_hardess.exe`
- `medium_rare.dll`
- `medium_rare.exp`

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Qftn4wc9yDSZg2X1vf8sNA.png)

คาดว่า flag ถูกซ่อนอยู่ในหนึ่งในไฟล์นี้  
เลยลองดึง string จาก `.dll` ด้วยคำสั่ง:

```bash
strings medium_rare.dll | grep -i flag
```

![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*7mdsBLu_t7sSmp8JaCIrcg.png)

ผลลัพธ์พบ flag 2 อัน คือ:

```
flag{fake_flag_try_harder_n00b}
flag{0b58400010f01d7e0fb096a565b553ab}
```

ผมเลยลองใส่ flag อันที่สองดู 


🚩ได้ flag เป็น

```
flag{0b58400010f01d7e0fb096a565b553ab}
```

---
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*TuV_WypzKTgON-cI3zCLIw.png)
## 7. Die With a Smile *(ข้อนี้ข้อแจก)*
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*y6HUMjveM-tfcFofY0vRJg.png)
ข้อนี้ให้ไฟล์ zip มา เมื่อแตกไฟล์แล้วจะได้ PNG มา 2 รูป  
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*5yLbu_byNb7VeoqNKULc2A.png)![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*iwdFeLsy1HjrP5K0IOIAfg.png)
เลยลองเอา QR code ไปสแกนดู… 
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*BHCiFk_DuTh-yQ7AWVrFoA.png)

🚩ได้ flag:

```
flag{1d_w4nn4_b3_n3x7_70_y0u}
```

---
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*uSZ-6nVdehY61PuNwk1YjQ.png)
## 8. Factor Wars
![img](https://miro.medium.com/v2/resize:fit:640/format:webp/1*P1utmM_NIjj7iniIMACjhA.png)
โจทย์ให้ไฟล์ zip มา ซึ่งด้านในมีสองไฟล์:
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*HC8sqrWzKLJUxGLw4mIqEA.png)
![img](https://miro.medium.com/v2/resize:fit:720/format:webp/1*FuRf1W_mRcaP-xQYkBaI8A.png)

- chall.py เป็น สคริปต์ที่โจทย์ใช้สร้าง RSA
- ส่วน output.txt เป็น ไฟล์ที่ให้ข้อมูล:
- N = โมดูลัส (ผลคูณของ prime หลายตัว)
- e = public exponent (มักจะเป็น 65537)
- c = ciphertext (ข้อความที่ถูกเข้ารหัสแล้ว)


หน้าตาประมาณนี้:

```
N : 0x....
e : 0x10001
c : 0x....
```
ผมเลยให้chatgpt🤑🤑
เขียนโค้ดหา prime ของ RSA จากการ brute-force seed ให้

โค้ดโจมตี (ย่อ/อธิบาย)

หลักการสำคัญ:
ใช้ random.Random(); r.seed(s); r._randbelow(1<<256) | 1 วนจนเจอ prime
ค้นหา: วน seed แล้ว gcd(N, p)
CRT ถอดรหัส: รวมผล m_p เพื่อได้ plaintext ทั้งหมด


```python
import re, math, random
from Crypto.Util.number import isPrime, inverse


# อ่าน N,e,c จาก output.txt
with open('output.txt','r') as f:
txt = f.read()
get = lambda tag: int(re.search(rf"{tag}\s*:\s*(0x[0-9a-fA-F]+)", txt).group(1), 16)
N_full, e, c = get('N'), get('e'), get('c')


# สร้าง prime ตามวิธีในโจทย์
def gen_prime_by_seed(seed: int):
r = random.Random(); r.seed(seed)
while True:
cand = r._randbelow(1<<256) | 1
if isPrime(cand):
return cand


# brute-force seed สำหรับ n เล็ก ๆ และดึง g = gcd(N,p)
N = N_full
primes = []
for n in range(2, 12): # พอจน product เกิน 1024 บิต
got = None
for s in range(1<<n):
p = gen_prime_by_seed(s)
g = math.gcd(N, p)
if g != 1:
primes.append(g)
N //= g
got = (n, s)
break
# หยุดเมื่อผลคูณพอแล้ว
if math.prod(primes).bit_length() > 1024 + 32:
break


# ถอดรหัสด้วย CRT บนโมดูลัสย่อย
from functools import reduce


mods = []
for p in primes:
dp = pow(e, -1, p-1)
mp = pow(c, dp, p)
mods.append((p, mp))


# CRT สองตัวประกอบต่อครั้ง
def crt_pair(a1, n1, a2, n2):
t = ((a2 - a1) % n2) * pow(n1, -1, n2) % n2
return (a1 + n1 * t, n1 * n2)


x, mod = mods[0][1], mods[0][0]
for p, mp in mods[1:]:
x, mod = crt_pair(x, mod, mp, p)


m = x # เพราะ mod (ผลคูณ prime) > 2^1024
pt = m.to_bytes(128, 'big')
print(pt)
```

🚩จากการรัน เราได้ flag:

```
flag{multi_pr1me_cr4ck_by_RSA_CRT_47e1cc2ddc}
```
---




แข่งครั้งนี้ ChatGPT แบกจัด


ข้อนี้ใครทำได้มาบอกกันด้วยเด้อ
![img](https://miro.medium.com/v2/resize:fit:828/format:webp/1*zzGl7JvGb5uGmOYvO8XzoA.jpeg)![img](https://miro.medium.com/v2/resize:fit:330/format:webp/1*AHX5qUuHm89qyuONPJK6cg.gif)
---

## Thank for coming everyone
