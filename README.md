# CIP-B105-CS4-C11_26_DFIT_17300 | SEED Morris Worm Forensic Investigation
[![Forensic](https://img.shields.io/badge/Forensic-COMPLETE-green?style=for-the-badge)](./)
[![Chain of Custody](https://img.shields.io/badge/Chain-16%20Evidences-blue?style=for-the-badge)](./evidence)
[![Hash](https://img.shields.io/badge/Hash-e8a77eb1-orange?style=for-the-badge)](./)
[![Status](https://img.shields.io/badge/Status-CONTAINED-red?style=for-the-badge)](./)

**Case: SEED-MORRIS-2026-09-21 | Date: 21 Sep 2026 03:14:05-16:08:55 UTC**
**Course: CIP-B105 / CS4 / C11 / 26_DFIT_17300 | Analyst: Simonadeka**
**Victim: b579b916f322 = 10.152.0.71 | Attacker: 7064d925363c = 10.151.0.71**

> **Executive Summary:** Isolated emulation of 1988 Morris Worm in SEED internet-nano (3 AS). Full NIST SP 800-86 lifecycle from acquisition 2029834b to containment with 16 timestamped evidences. **Pre-detonation confirmed: ret=0x00, offset=0x00, no badfile.** Zero external exposure.

---

## 1. Methodology - NIST SP 800-86
- **Identification:** Official SEED Labsetup.zip from seedsecuritylabs.org via `wget --no-check-certificate` (expired LetsEncrypt logged)
- **Collection:** `unzip`, `docker-compose build/up`, `docker exec -it {ID} /bin/zsh`, `ps aux`, `ss -tunap`, `sha256sum`, `find`, `grep`, `date -u`
- **Preservation:** SHA256 `e8a77eb10841d6fb...afdb4ab`, container IDs `b579`/`7064`, digests `9d65ed5a` & `7504e96a`
- **Examination:** `bytearray(0x90 for 500)` NOP sled, `ret=0x00`, `offset=0x00`, `getNextTarget() -> 10.151.0.71`, `nc -w3 9090`
- **Analysis:** Correlation Container ID -> IP -> PID 59 -> Port 9090 LISTEN
- **Containment:** Isolated bridge networks `net_151/152/153`, routers `.254`, final `docker ps empty`

## 2. Key Findings

| Finding | Evidence | Value |
|---|---|---|
| **Build Chain Verified** | 04 | Base digest `9d65ed5a -> 170a7a0b8e75`, Map digest `7504e96a` |
| **Network Isolation** | 05 | 3 bridges, 3 routers, no egress |
| **Attribution** | 06 | 7064=10.151.0.71 Attacker, b579=10.152.0.71 Victim |
| **Service Baseline** | 08-11 | PID 57 `tail -f /dev/null`, PID 59 `./server LISTEN 0.0.0.0:9090` |
| **Pre-Weaponized** | 07 | 500x NOP `0x90`, shellcode placement, ret/offset placeholder |
| **Negative Finding** | 13 | `find badfile*` EMPTY = pre-detonation proof |
| **Propagation IoC** | 15 | `cat badfile \| nc -w3 {targetIP} 9090` Line 72 |
| **DNS Troubleshooting** | 03 | `127.0.0.53:53 timeout` -> `8.8.8.8` fix documented |

## 3. Evidence Map - 16 Screenshots (Chronological)

| # | Name | File | Proves |
|---|---|---|---|
| 01 | Wget Acquisition | `01_Wget` | 185.199.108.153, 2029834b, 1.98MB/s |
| 02 | Unzip Structure | `02_Unzip` | internet-mini/nano, worm/, shellcode/ |
| 03 | DNS Failure **FULL PAGE** | `03_DNS_Failure` | 127.0.0.53 timeout, ping 8.8.8.8 0% loss, fix |
| 04 | Build Success | `04_Build_Success` | Digest `9d65ed5a` |
| 05 | Network Create | `05_Network_Create` | net_151/152/153, digest `7504e96a` |
| 06 | Container Mapping **ENLARGE** | `06_Mapping` | 7064=10.151.0.71, b579=10.152.0.71, 6e1b=73 |
| 07 | Code NOP500 **LARGEST** | `07_Code_NOP500` | `0x90 x500`, `ret=0x00`, `getNextTarget 10.151.0.71` |
| 08 | PID 57/59 Baseline | `08_PID57_59` | `tail -f` + `./server` |
| 09 | Attacker 7064 | `09_Attacker` | 10.151.0.71 ps aux |
| 10 | Victim b579 | `10_Victim` | 10.152.0.71 ps aux no infection |
| 11 | Port 9090 LISTEN | `11_ss_9090` | `0.0.0.0:9090 LISTEN pid=59 fd=3` |
| 12 | Hash Integrity | `12_Hash` | `e8a77eb1...afdb4ab` 2.7K |
| 13 | Find Negative **RED BOX** | `13_Find_negative` | No badfile = pre-detonation |
| 14 | Shellcode Assets | `14_Shellcode` | call_shellcode.c, shellcode_32/64.py |
| 15 | Grep IoC | `15_grep` | Line 49, 66, 72 propagation |
| 16 | Containment FINAL | `16_Containment` | `docker ps empty`, `date -u 16:08:55`, `down` |

## 4. Chain of Custody
03:14:05 Acquisition 2029834b (01) -> 03:15 Extraction (02) -> 03:20 DNS Fix 9d65ed5a (03,04)
-> 03:30 Network 7504e96a (05) -> 03:32 Mapping 7064=10.151.0.71 b579=10.152.0.71 (06)
-> 12:25 Code e8a77eb1 PID59:9090 (07-15) -> 16:08:55 Containment empty (16)


## 5. Commands
```bash
wget Labsetup.zip --no-check-certificate
docker-compose build # fail 127.0.0.53
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
docker-compose build # 9d65ed5a
docker-compose up # 7504e96a
docker ps; docker exec -it b579 /bin/zsh; ps aux; ss -tunap
sha256sum worm.py; find -name "badfile*"
docker-compose down; date -u

6. Mapping7064d925363c10.151.0.71 Attackerb579b916f32210.152.0.71 VictimAnalyst: Simonadeka | Hash: e8a77eb1 | Status: CONTAINED
