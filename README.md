# TryHackMe---W1seGuy-CTF
[![Solved](https://img.shields.io/badge/Solved%20By-Jull3Hax0r-blue?style=flat-square&logo=gnu-bash)](https://tryhackme.com/p/Jull3)
[![TryHackMe Room](https://img.shields.io/badge/TryHackMe-Chill%20Hack-success?style=flat-square&logo=tryhackme)](https://tryhackme.com/room/w1seguy)
XOR crypto challenge 
# 🧙‍♂️ TryHackMe: W1seGuy – XOR CTF Writeup

**Room:** [W1seGuy](https://tryhackme.com/room/w1seguy)  
**Prompt:** "Our friend told me you were wise, but I don't believe them. Can you prove me wrong?"  

---

## Challenge Overview

- Deploy the VM and connect to port `1337` (TCP) using netcat or a similar tool:
  ```bash
  nc <MACHINE_IP> 1337
  ```
- The challenge is all about **XOR decryption** and understanding simple crypto weaknesses.

---

## TL;DR – Answers

- **First Flag:**  
  `THM{XXXXXXXXXXXXXXXXXXXX}`

- **Second Flag:**  
  `THM{XXXXXXXXXXXXXXXXXXXX}`

---

## Solution Walkthrough

At first, I fell into the classic trap: I tried to brute-force flags I found via Nmap enumeration, but those responses were just decoy loops.  
**Key lesson:** *Always read the description and instructions carefully!*  
Once I realized I should connect to the server on port `1337` using `nc`, everything became much clearer.

The real fun began: the server presents you with an XOR-encrypted flag, and asks for the decryption key.  
After some trial and error, I solved it efficiently with a simple Python script:

```python
from binascii import unhexlify

def crack_xor_flag(xor_hex, flag_prefix="THM{", flag_suffix="}"):
    xor_bytes = unhexlify(xor_hex)
    key_len = 5  # Common length in CTFs

    # Recover key based on known flag format
    key = [
        chr(xor_bytes[0] ^ ord(flag_prefix[0])),
        chr(xor_bytes[1] ^ ord(flag_prefix[1])),
        chr(xor_bytes[2] ^ ord(flag_prefix[2])),
        chr(xor_bytes[3] ^ ord(flag_prefix[3])),
        chr(xor_bytes[-1] ^ ord(flag_suffix[0]))
    ]
    key_str = "".join(key)
    print(f"Recovered key: {key_str}")

    # Decrypt the flag
    flag = ""
    for i in range(len(xor_bytes)):
        flag += chr(xor_bytes[i] ^ ord(key_str[i % key_len]))
    print("Decrypted flag:", flag)

# Example usage:
crack_xor_flag("6d1f062d3108362738357c2f3f17354d63283d227839396520551b323e144b233266344b2f04243c")
```

---

## XOR CTF Decryption – Quick Handbook

### What is XOR?

- **XOR** (`^` in Python) is a bitwise operation:
  - If bits are the same (0/0 or 1/1): result is **0**
  - If bits are different (1/0 or 0/1): result is **1**
- **In CTFs**, XOR is commonly used for lightweight encryption using a short, repeating key.

---

### Typical XOR-CTF Algorithm

```python
# Encryption
for i in range(len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i % len(key)]))
```
- The **key** (e.g., `"abcde"`) repeats (modulo) for the flag length.
- The flag becomes "garbage" unless you know the key.

---

### Why Are XOR Flags Easy to Break in CTFs?

- The **flag format is known** (usually starts with `THM{` and ends with `}`).
- The **key is short and often given** (commonly 5 characters).
- **Flag chaining:** The key for the next step is often the previous flag (or part of it).

---

### Step-by-Step XOR Decryption

#### 1. Convert hex to bytes

```python
from binascii import unhexlify
xor_bytes = unhexlify("your_hex_string")
```

#### 2. Guess key from flag format

Example for a 5-character key:
```python
key = [
    chr(xor_bytes[0] ^ ord('T')),
    chr(xor_bytes[1] ^ ord('H')),
    chr(xor_bytes[2] ^ ord('M')),
    chr(xor_bytes[3] ^ ord('{')),
    chr(xor_bytes[-1] ^ ord('}'))
]
key = "".join(key)
```

#### 3. Decrypt the flag

```python
flag = ""
for i in range(len(xor_bytes)):
    flag += chr(xor_bytes[i] ^ ord(key[i % len(key)]))
print(flag)
```

#### 4. If you don't get a flag:

- Double-check key length (try 6 or 7).
- Check for alternate flag formats (`HTB{`, `picoCTF{`).
- Make sure you are XORing bytes, not hex chars.

---

### Plug-and-Play XOR-flag Decoder

```python
from binascii import unhexlify

def crack_xor_flag(xor_hex, flag_prefix="THM{", flag_suffix="}"):
    xor_bytes = unhexlify(xor_hex)
    key_len = 5
    key = [
        chr(xor_bytes[0] ^ ord(flag_prefix[0])),
        chr(xor_bytes[1] ^ ord(flag_prefix[1])),
        chr(xor_bytes[2] ^ ord(flag_prefix[2])),
        chr(xor_bytes[3] ^ ord(flag_prefix[3])),
        chr(xor_bytes[-1] ^ ord(flag_suffix[0]))
    ]
    key_str = "".join(key)
    flag = ""
    for i in range(len(xor_bytes)):
        flag += chr(xor_bytes[i] ^ ord(key_str[i % key_len]))
    print(f"Key: {key_str}")
    print(f"Flag: {flag}")

# Example usage:
# crack_xor_flag("392d010c075c04201903281d38360319512f1c142c0b3e44160129351f221f113547021f1d03050a")
```

---

### Tips for Flag Chaining

- **Next key** is often the flag you just got (without `THM{}`).
- If you get "close but no cigar", tweak the key (add/remove a character, try different case, etc.).
- Always brute-force small key tweaks for chained XOR CTFs.

---

### Common Pitfalls

- Forgetting to convert hex to bytes.
- Wrong key length.
- Wrong prefix/suffix.
- Using the whole flag (with or without braces) as key.
- Not using modulo when looping key.

---

### Minimal XOR-CTF Cheat Sheet

1. Copy the hex string.
2. Convert to bytes: `from binascii import unhexlify`
3. Guess key using known flag prefix/suffix.
4. XOR the entire string using the key (with modulo).
5. If you get junk, try another key length or tweak the key.

---

### Further Reading

- [Wikipedia: XOR cipher](https://en.wikipedia.org/wiki/XOR_cipher)
- [CryptoPals Set 1](https://cryptopals.com/sets/1)
- [CTF Writeups on ctftime.org](https://ctftime.org/writeups)
- [TryHackMe: W1seGuy](https://tryhackme.com/room/w1seguy)

---

> **Keep this as your XOR-CTF decryption playbook. Don’t just brute-force—look for patterns, and you’ll always crack it faster! 🚩**
