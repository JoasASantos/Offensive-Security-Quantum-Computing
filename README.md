# Offensive Security in Quantum Computing
### A Comprehensive Technical Reference for Security Researchers and Red Team Practitioners

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Quantum Computing Fundamentals for Security Practitioners](#2-quantum-computing-fundamentals-for-security-practitioners)
3. [The Cryptographic Threat Model](#3-the-cryptographic-threat-model)
4. [Harvest Now, Decrypt Later (HNDL) Attacks](#4-harvest-now-decrypt-later-hndl-attacks)
5. [Quantum Attacks on Symmetric Cryptography](#5-quantum-attacks-on-symmetric-cryptography)
6. [Quantum Attacks on Asymmetric Cryptography](#6-quantum-attacks-on-asymmetric-cryptography)
7. [Shor's Algorithm — Offensive Implications](#7-shors-algorithm--offensive-implications)
8. [Grover's Algorithm — Offensive Implications](#8-grovers-algorithm--offensive-implications)
9. [Quantum Side-Channel Attacks](#9-quantum-side-channel-attacks)
10. [Quantum Network Attacks](#10-quantum-network-attacks)
11. [Post-Quantum Cryptography (PQC) Attack Surface](#11-post-quantum-cryptography-pqc-attack-surface)
12. [Attacks on Quantum Key Distribution (QKD)](#12-attacks-on-quantum-key-distribution-qkd)
13. [Quantum Machine Learning in Offensive Operations](#13-quantum-machine-learning-in-offensive-operations)
14. [Quantum Random Number Generator (QRNG) Attacks](#14-quantum-random-number-generator-qrng-attacks)
15. [Threat Actors and Nation-State Quantum Programs](#15-threat-actors-and-nation-state-quantum-programs)
16. [Red Team Methodologies for Quantum-Era Threats](#16-red-team-methodologies-for-quantum-era-threats)
17. [Tooling and Frameworks](#17-tooling-and-frameworks)
18. [CVEs and Known Vulnerabilities in Quantum Stacks](#18-cves-and-known-vulnerabilities-in-quantum-stacks)
19. [Defensive Mitigations and Crypto Agility](#19-defensive-mitigations-and-crypto-agility)
20. [Research Repositories and Datasets](#20-research-repositories-and-datasets)
21. [Scientific References](#21-scientific-references)

---

## 1. Introduction

Quantum computing is no longer a distant theoretical concept. With IBM's 1,000+ qubit processors, Google's quantum supremacy demonstrations, and nation-state investment in quantum infrastructure exceeding hundreds of billions of dollars, the offensive security community must urgently reckon with the cryptographic and systemic implications of quantum-capable adversaries.

This document addresses quantum computing from the **attacker's perspective** — covering threat models, known quantum algorithms with offensive utility, attack surfaces on classical and quantum infrastructure, weaknesses in Post-Quantum Cryptography implementations, and red team methodologies for a post-quantum world.

**Key timelines to internalize:**

| Milestone | Estimated Year | Confidence |
|-----------|---------------|------------|
| Cryptographically Relevant Quantum Computer (CRQC) | 2030–2035 | Medium |
| Shor's attack on RSA-2048 viable | 2033–2040 | Low-Medium |
| Nation-state adversaries with covert CRQC | Unknown | Unclassifiable |
| NIST PQC standards fully deployed | 2027–2030 | High |

---

## 2. Quantum Computing Fundamentals for Security Practitioners

### 2.1 Qubits vs. Classical Bits

A classical bit holds either 0 or 1. A **qubit** exists in a superposition of both states simultaneously until measured. This enables quantum computers to process exponentially larger solution spaces in parallel.

```
Classical: 3 bits → 1 of 8 states at a time
Quantum:   3 qubits → all 8 states simultaneously
```

### 2.2 Key Quantum Principles

| Principle | Security Relevance |
|-----------|-------------------|
| **Superposition** | Enables parallel evaluation of all possible keys simultaneously |
| **Entanglement** | Enables quantum key distribution; also enables correlated multi-qubit attacks |
| **Interference** | Used to amplify correct answers in algorithms like Grover's |
| **No-cloning theorem** | Prevents copying quantum states — relevant to QKD security proofs |
| **Decoherence** | Current limitation; quantum states collapse under environmental noise |

### 2.3 Quantum Gate Operations

Relevant gates for cryptanalysis:

- **Hadamard (H)**: Creates superposition — the foundation of quantum parallelism
- **CNOT**: Entangles two qubits — used extensively in Shor's algorithm
- **Toffoli (CCNOT)**: Universal reversible gate used in oracle constructions for Grover's search
- **QFT (Quantum Fourier Transform)**: Core of Shor's period-finding — the operation that breaks RSA/ECC

### 2.4 Current Hardware Landscape

| Platform | Qubits (2024-2025) | Error Rate | Relevance to Attacks |
|----------|-------------------|------------|---------------------|
| IBM Heron | 133–156 | ~0.1% (2Q) | NISQ era; no cryptographic threat yet |
| Google Willow | 105 | Low error | Demonstrated error correction advances |
| IonQ Forte | 35 (algorithmic) | Very low | Higher quality; slower |
| Quantinuum H2 | 56 | Best-in-class | Highest fidelity available commercially |
| Microsoft Azure Quantum | Varies | — | Topological qubit research |

> **Red Team Note**: No currently available quantum hardware can break RSA-2048. A CRQC would require approximately **4,000 logical (error-corrected) qubits**, translating to **millions of physical qubits** with current error rates.

---

## 3. The Cryptographic Threat Model

### 3.1 Q-Day Definition

**Q-Day** refers to the moment a Cryptographically Relevant Quantum Computer (CRQC) becomes operational — either publicly or covertly by a nation-state adversary. At Q-Day, all RSA, ECC, DH, and DSA-protected communications become retroactively and proactively decryptable.

### 3.2 Algorithm Vulnerability Matrix

| Algorithm | Type | Quantum Threat | Status Post-Q-Day |
|-----------|------|---------------|-------------------|
| RSA-2048 | Asymmetric | **BROKEN** (Shor's) | Completely insecure |
| ECDSA P-256 | Asymmetric | **BROKEN** (Shor's) | Completely insecure |
| Diffie-Hellman | Key Exchange | **BROKEN** (Shor's) | Completely insecure |
| AES-128 | Symmetric | **WEAKENED** (Grover's) | Effectively AES-64 |
| AES-256 | Symmetric | **Weakened** (Grover's) | Effectively AES-128; still acceptable |
| SHA-256 | Hash | **Weakened** (Grover's) | Collision resistance halved |
| SHA-3 | Hash | Minimal impact | Acceptable with larger output |
| ChaCha20 | Symmetric | Weakened | Acceptable at 256-bit |
| CRYSTALS-Kyber | PQC KEM | Believed secure | NIST selected |
| CRYSTALS-Dilithium | PQC Signature | Believed secure | NIST selected |
| FALCON | PQC Signature | Believed secure | NIST selected |
| SPHINCS+ | PQC Signature | Believed secure | NIST selected |

---

## 4. Harvest Now, Decrypt Later (HNDL) Attacks

### 4.1 Attack Concept

HNDL (also called **"Store Now, Decrypt Later"** or **SNDL**) is the most immediately relevant quantum-era attack vector. It does not require a CRQC today — only collection infrastructure.

**Attack Flow:**

```
[TODAY]
Adversary intercepts and stores encrypted traffic
      ↓
[PASSIVE COLLECTION PHASE]
Mass storage of TLS sessions, VPN tunnels, encrypted emails,
government/military communications, financial transactions
      ↓
[Q-DAY + N years]
Adversary operates or accesses CRQC
      ↓
[DECRYPTION PHASE]
Retroactively decrypts all stored data using Shor's algorithm
```

### 4.2 Evidence of Active HNDL Operations

Intelligence community assessments and leaked NSA documents indicate that several nation-state programs are actively collecting and archiving encrypted communications with post-quantum decryption as the stated objective.

**HNDL-relevant interception points:**
- Submarine cable taps (MUSCULAR, TEMPORA programs)
- IXP (Internet Exchange Point) mirroring
- BGP hijacking for traffic redirection
- Compromised CDN/cloud infrastructure
- TLS 1.2 sessions with non-PFS cipher suites

### 4.3 Data Prioritization by Adversaries

```
Priority 1 (Decades-sensitive): Nuclear secrets, weapons designs, 
                                 diplomatic cables, agent identities
Priority 2 (15-year shelf life): Military strategy, intelligence sources
Priority 3 (5-10 year shelf life): Financial data, IP, trade secrets
Priority 4 (1-5 year shelf life): PII, credentials, business comms
```

### 4.4 HNDL Attack Surface Assessment

**High-risk protocols still widely deployed:**
- TLS 1.2 with RSA key exchange (no forward secrecy)
- IPsec IKEv1 with RSA authentication
- S/MIME encrypted email
- PGP/GPG with RSA keys ≤ 3072-bit
- SSH with RSA host keys

---

## 5. Quantum Attacks on Symmetric Cryptography

### 5.1 Grover's Algorithm Applied to Symmetric Keys

Grover's algorithm provides a **quadratic speedup** for unstructured search problems, effectively halving the bit-security of symmetric encryption:

```
AES-128: Classical O(2^128) → Quantum O(2^64) [practical threshold concern]
AES-192: Classical O(2^192) → Quantum O(2^96) [acceptable]
AES-256: Classical O(2^256) → Quantum O(2^128) [recommended]
```

**However**, Grover's speedup faces significant practical constraints:

1. **Serial execution requirement**: Grover iterations are sequential, not parallelizable across multiple quantum computers
2. **Oracle construction overhead**: Implementing AES as a quantum oracle requires ~2,000–3,000 T-gates per AES-128 round
3. **Error correction overhead**: Logical qubit requirements for AES attack dwarf current hardware

> **Research Reference**: Jaques et al. (2020) — *"Implementing Grover Oracles for Quantum Key Search on AES and LowMC"* (EUROCRYPT 2020). Estimated 2,953 logical qubits and 2^86.8 quantum operations for AES-128.

### 5.2 Quantum Attacks on Hash Functions

**Birthday attacks using BHT (Brassard-Høyer-Tapp) algorithm:**

```
Classical birthday attack on SHA-256: O(2^128)
Quantum BHT attack on SHA-256: O(2^85.3) [in QRAM model]
```

**Grover preimage attack:**
```
SHA-256 preimage:   Classical O(2^256) → Quantum O(2^128)
SHA-256 collision:  Classical O(2^128) → Quantum O(2^85) [BHT]
```

**Impact on blockchain security:**
- Bitcoin's SHA-256 proof-of-work becomes significantly easier to mine with quantum hardware
- UTXO addresses using P2PK (pay-to-public-key) are vulnerable to Shor's attack
- P2PKH addresses are initially protected until first spend reveals public key

---

## 6. Quantum Attacks on Asymmetric Cryptography

### 6.1 RSA Vulnerability

RSA security relies on the **integer factorization problem**. Shor's algorithm solves this in **polynomial time** — specifically O((log N)^3) — rendering all RSA key sizes broken.

**Factorization complexity comparison:**

```
RSA-2048 Classical (GNFS): ~2^112 operations → infeasible
RSA-2048 Quantum (Shor's): ~O((2048)^3) operations → feasible with CRQC
```

**Estimated CRQC resources for RSA-2048:**
- ~4,000 logical qubits (error-corrected)
- ~10^10 quantum gates
- Execution time: hours to days (highly dependent on clock speed)

### 6.2 ECC Vulnerability

Elliptic Curve cryptography relies on the **elliptic curve discrete logarithm problem (ECDLP)**. Shor's algorithm (specifically the **quantum algorithm for discrete logarithm** by Shor, 1994) breaks ECDLP in polynomial time.

**Vulnerable curves:**
- NIST P-256 / P-384 / P-521
- Curve25519 / Ed25519
- secp256k1 (Bitcoin/Ethereum)
- Brainpool curves

> **Note**: The elliptic curve variant of Shor's requires roughly **2n** qubits to attack an **n-bit** ECC key, making ECC-256 (requiring ~512 logical qubits) theoretically breakable *before* RSA-2048.

### 6.3 Diffie-Hellman and DSA

Both rely on the **discrete logarithm problem** in finite fields — directly solvable by Shor's algorithm. This includes:
- Classical DH (MODP groups)
- DSA (Digital Signature Algorithm)  
- ElGamal encryption

---

## 7. Shor's Algorithm — Offensive Implications

### 7.1 Algorithm Overview

Shor's algorithm (1994) factors integers and computes discrete logarithms in polynomial time using quantum Fourier transform for period-finding.

**High-level factoring attack flow:**

```
INPUT: N (number to factor, e.g., RSA modulus)

Step 1: Choose random a < N, compute gcd(a, N)
        If gcd ≠ 1, found factor (classical)

Step 2: [QUANTUM] Find period r of function f(x) = a^x mod N
        using Quantum Phase Estimation + QFT

Step 3: [CLASSICAL] Compute gcd(a^(r/2) ± 1, N)
        → yields non-trivial factors of N with high probability

OUTPUT: p, q such that N = p × q
```

### 7.2 Resource Estimation

| RSA Key Size | Logical Qubits Needed | Physical Qubits (current error rates) |
|-------------|----------------------|---------------------------------------|
| RSA-1024 | ~2,000 | ~2–4 million |
| RSA-2048 | ~4,000 | ~4–8 million |
| RSA-4096 | ~8,000 | ~8–16 million |

> **Reference**: Webber et al. (2022) — *"The impact of hardware specifications on reaching quantum advantage in the fault tolerant regime"* — estimated that breaking RSA-2048 in 8 hours requires 317×106 physical qubits with surface code error correction.

### 7.3 Implementation Research

Key papers on optimized Shor's implementations:

- **Beauregard (2003)**: Circuit for Shor's using 2n+3 qubits — the foundational efficient construction
- **Zalka (1998)**: Fast Shor's without Quantum Fourier Transform
- **Roetteler et al. (2017)**: Quantum resource estimates for computing elliptic curve discrete logarithms (*ASIACRYPT 2017*)
- **Häner et al. (2020)**: Improved quantum circuits for elliptic curve discrete logarithms

---

## 8. Grover's Algorithm — Offensive Implications

### 8.1 Algorithm Overview

Grover's algorithm searches an unsorted database of N items in O(√N) time — a quadratic speedup over classical O(N) search.

**Offensive applications:**

| Target | Classical Complexity | Quantum (Grover) |
|--------|---------------------|-----------------|
| AES-128 brute force | O(2^128) | O(2^64) |
| Password cracking (MD5-hashed) | O(2^128) | O(2^64) |
| Finding hash collisions | O(2^128) | O(2^85) (BHT) |
| Pre-image attack on SHA-256 | O(2^256) | O(2^128) |
| Breaking WPA2-PSK | O(2^256) | O(2^128) |

### 8.2 Parallel Grover's — The Limits

A common misconception: multiple quantum computers running Grover's in parallel do **not** multiply the speedup. Parallel Grover reduces time but requires proportionally more resources, maintaining the same total quantum operation count. Classical computing retains its parallelism advantage for raw brute-force tasks.

### 8.3 Grover's Oracle Construction for Cryptanalysis

```python
# Conceptual quantum circuit pseudocode (Qiskit-like)
# Grover oracle for AES-128 key search

def grover_aes_oracle(circuit, key_register, data_register, target_ciphertext):
    """
    Mark |key⟩ states that encrypt known_plaintext to target_ciphertext
    """
    # Apply AES forward transformation (as reversible quantum circuit)
    apply_quantum_aes(circuit, key_register, data_register)
    
    # Phase kickback: flip phase if output matches target
    phase_oracle(circuit, data_register, target_ciphertext)
    
    # Uncompute AES (reverse transformation)
    apply_quantum_aes_inverse(circuit, key_register, data_register)
```

---

## 9. Quantum Side-Channel Attacks

### 9.1 Electromagnetic Side-Channels on Classical Cryptographic Hardware

Even pre-CRQC, quantum sensing technologies enable **dramatically enhanced classical side-channel attacks**:

**Quantum-enhanced EM side-channel:**
- Quantum sensors (atomic magnetometers, SQUIDs) offer sensitivity orders of magnitude beyond classical EM probes
- Can recover AES keys from distances of several meters in controlled environments
- NIST Special Publication 800-140C acknowledges quantum sensor threats to FIPS-validated modules

**Nitrogen-Vacancy (NV) Centers for Side-Channel:**
- Diamond NV-center magnetometers achieve ~fT/√Hz sensitivity
- Demonstrated key recovery from unshielded cryptographic hardware at centimeter distances
- Research Reference: Gusmeroli et al. (2023) — *"Side-channel attacks with quantum sensors"*

### 9.2 Quantum Timing Attacks

Quantum algorithms can be used to enhance the statistical analysis of timing data:
- Quantum Monte Carlo methods accelerate correlation power analysis (CPA)
- Quantum machine learning improves template attacks on side-channel leakage

### 9.3 Attacks on Quantum Hardware Itself

**Crosstalk attacks:**
- In multi-tenant quantum cloud systems (IBM Q, AWS Braket), qubit crosstalk between users' circuits leaks information
- Demonstrated in: Ash-Saki et al. (2020) — *"CHAOS: A Novel Cross-talk Model for Quantum Systems"*

**Fault injection on quantum circuits:**
- Deliberate decoherence injection to bias quantum computation outcomes
- Applicable to quantum key generation circuits in QKD systems

---

## 10. Quantum Network Attacks

### 10.1 Quantum Internet Attack Surface

As quantum networks develop, new attack vectors emerge:

```
[Classical Internet Attacks]          [Quantum Network Attacks]
───────────────────────────          ─────────────────────────
BGP Hijacking              →         Entanglement Distribution Poisoning
DNS Spoofing               →         Quantum Repeater Node Compromise  
TLS MITM                   →         Beam-Splitter MITM on QKD
Replay Attacks             →         Photon Number Splitting (PNS) Attack
DDoS                       →         Quantum Memory Depletion Attack
```

### 10.2 Intercept-Resend Attack on QKD

The most basic QKD attack:

```
Alice ──[qubits]──→ Eve ──[re-encoded qubits]──→ Bob
                    ↓
              Measures qubits,
              re-sends with ~25% error rate
              → Detectable (QBER > 11%)
```

**Limitation**: Causes detectable quantum bit error rate (QBER) elevation above threshold (~11% for BB84), triggering abort.

### 10.3 Photon Number Splitting (PNS) Attack

Exploits a vulnerability in practical QKD implementations using **weak coherent pulse (WCP)** laser sources:

```
Weakness: WCP sources sometimes emit 2+ photons per pulse

Attack:
1. Eve intercepts all multi-photon pulses
2. Stores one photon, forwards the other to Bob
3. Waits for Alice to announce basis choices
4. Measures stored photon in correct basis
5. Extracts key information without disturbing single-photon pulses
```

**Impact**: PNS allows Eve to extract **full key material** from implementations without decoy-state countermeasures.

> **Reference**: Brassard et al. (2000) — *"Limitations on Practical Quantum Cryptography"* — first formal analysis of PNS attacks.

### 10.4 Trojan-Horse Attack on QKD

```
Attack Vector: Injecting bright light pulses INTO Alice/Bob's QKD device
               and analyzing reflections to determine internal state settings

Impact: Completely passive; no quantum disturbance introduced
        Recovers Alice's basis choices → full key compromise

Countermeasure: Optical isolators, watchdog detectors, IEC 62351-8
```

> **Reference**: Vakhitov et al. (2001); Gisin et al. (2006) — *"Trojan-horse attacks on quantum-key-distribution systems"*

### 10.5 Detector Blinding Attack

**Most practically demonstrated QKD attack:**

```
Attack Mechanics:
1. Eve continuously illuminates Bob's single-photon detectors
   with bright light (mW range)
2. Detectors enter "linear mode" — behave like classical detectors
3. Eve sends tailored trigger pulses that only activate one detector
   (the one matching Eve's chosen bit value)
4. Bob's measurement results become deterministic — controlled by Eve
5. Eve learns full key with zero QBER

Real-World Impact: Demonstrated against commercial Clavis2 (ID Quantique)
                   and MagiQ QPN 5505 systems
```

> **Reference**: Lydersen et al. (2010) — *"Hacking commercial quantum cryptography systems by tailored bright illumination"* (**Nature Photonics** — the landmark paper)

---

## 11. Post-Quantum Cryptography (PQC) Attack Surface

### 11.1 NIST PQC Standards (FIPS 203/204/205)

The NIST standardization process selected four algorithms in 2024:

| Standard | Algorithm | Problem | Key Use |
|----------|-----------|---------|---------|
| FIPS 203 | ML-KEM (Kyber) | Module Learning With Errors (MLWE) | Key Encapsulation |
| FIPS 204 | ML-DSA (Dilithium) | Module LWE + Module SIS | Digital Signatures |
| FIPS 205 | SLH-DSA (SPHINCS+) | Hash-based (stateless) | Digital Signatures |
| FIPS 206 | FN-DSA (FALCON) | NTRU lattice | Digital Signatures |

### 11.2 Known Weaknesses and Side-Channel Attacks on PQC

**Kyber (ML-KEM) side-channel vulnerabilities:**

```
Attack Type: Timing side-channel on decapsulation
Mechanism:   Secret-dependent branches in message decoding
Impact:      Full key recovery in ~220 decapsulation queries
Reference:   Ravi et al. (2019) — "Side-channel assisted existential 
             forgery attack on Dilithium"
```

**CRYSTALS-Dilithium weaknesses:**
- **Rejection sampling side-channels**: Statistical leakage during signature generation
- **Lattice-based fault attacks**: Differential fault analysis on NTT (Number Theoretic Transform) operations
- Reference: Migliore et al. (2019) — *"Masking Dilithium: Efficient Implementation and Side-Channel Evaluation"*

**FALCON implementation pitfalls:**
- Fast Fourier Transform operations leak through cache timing
- Signature generation uses Gaussian sampling — notoriously difficult to implement in constant time
- Reference: Guerreau-Lambet et al. (2022) — *"The Hidden Parallelepiped Is Back Again: Power Analysis Attacks on Falcon"*

**SPHINCS+ concerns:**
- Stateless design introduces large signature sizes (~8–50 KB)
- Hash function security assumptions could weaken with new classical/quantum cryptanalysis

### 11.3 Lattice Reduction Attacks

PQC lattice-based schemes rely on the hardness of **Learning With Errors (LWE)** and **Shortest Vector Problem (SVP)**. Classical and quantum lattice reduction algorithms are the primary threat:

| Algorithm | Type | Security Impact |
|-----------|------|----------------|
| LLL (Lenstra-Lenstra-Lovász) | Classical | Polynomial; breaks only weak parameters |
| BKZ (Block Korkine-Zolotarev) | Classical | Practical threat for small dimensions |
| Quantum BKZ | Quantum | Grover-enhanced SVP oracle speeds lattice reduction |
| Hybrid attacks | Classical+Quantum | Mixes lattice reduction with Grover's meet-in-the-middle |

> **Reference**: Albrecht et al. (2021) — *"Lattice Attacks on NTRU and LWE: A History of Refinements"*

### 11.4 Side-Channel Attack Taxonomy for PQC

```
PQC Side-Channel Attack Surface
├── Power Analysis
│   ├── Simple Power Analysis (SPA) on NTT
│   ├── Differential Power Analysis (DPA) on sampling
│   └── Template attacks on polynomial operations
├── Timing Attacks
│   ├── Cache-timing on memory access patterns
│   ├── Branch-timing on rejection sampling
│   └── I/O timing on key generation
├── Electromagnetic Analysis
│   ├── Horizontal EM attacks on LWE decoding
│   └── DEMA on signature computation
├── Fault Attacks
│   ├── Laser fault injection on NTT butterfly units
│   ├── Voltage glitching on decapsulation
│   └── Clock glitching on Gaussian sampling
└── Combined Attacks
    ├── Differential Fault + Statistical (DFIA)
    └── SCA + Lattice reduction hybridization
```

---

## 12. Attacks on Quantum Key Distribution (QKD)

### 12.1 QKD Attack Classification

```
QKD Attacks
├── Implementation Attacks (most practical)
│   ├── Detector side-channels
│   │   ├── Detector Blinding Attack ← [DEMONSTRATED]
│   │   ├── Time-Shift Attack ← [DEMONSTRATED]
│   │   └── Dead-Time Attack
│   ├── Source side-channels
│   │   ├── Photon Number Splitting (PNS) ← [DEMONSTRATED]
│   │   ├── Trojan Horse Attack ← [DEMONSTRATED]
│   │   └── Laser Side-Channel
│   └── Classical Control Attacks
│       ├── Wavelength-Dependent Attack
│       ├── Phase-Remapping Attack
│       └── Authentication Bypass
├── Protocol Attacks (theoretical)
│   ├── Intercept-Resend
│   ├── Man-in-the-Middle (requires authentication bypass)
│   └── Coherent Attack (information-theoretic)
└── Quantum Repeater Attacks (emerging)
    ├── Entanglement Swapping Manipulation
    └── Quantum Memory Eavesdropping
```

### 12.2 Time-Shift Attack

```
Mechanism:
Eve shifts the detection time window of Bob's detectors
by exploiting efficiency imbalances between detector elements

Mathematical Basis:
η1(t) ≠ η2(t) — detectors have time-dependent efficiency curves
Eve shifts timing to exploit maximum efficiency differential

Impact:
0% QBER increase possible (within certain efficiency imbalance bounds)
Demonstrated against gap-engineered InGaAs APDs
```

### 12.3 Wavelength-Dependent Beam-Splitter Attack

```
Target: QKD systems using 50:50 beam splitters for basis selection
Weakness: Real beam splitters have wavelength-dependent splitting ratios

Attack:
1. Eve probes Bob's device with different wavelengths
2. Identifies wavelength where BS acts as 100:0 (fully deterministic)
3. Sends photons at attack wavelength to Bob
4. Predicts which detector fires → recovers basis + bit value
```

---

## 13. Quantum Machine Learning in Offensive Operations

### 13.1 Quantum-Enhanced Cryptanalysis via QML

Quantum machine learning (QML) algorithms may offer advantages in:

**Variational Quantum Eigensolver (VQE) for cryptanalysis:**
- Finding minimum energy states analogous to finding shortest lattice vectors
- Potential acceleration of LWE-based PQC attacks

**Quantum Neural Networks for pattern recognition:**
- Side-channel trace classification with fewer samples
- Quantum kernel methods for template attacks

> **Note**: QML advantages over classical ML for cryptanalysis remain largely unproven. Current QML algorithms (e.g., HHL, QSVM) face significant practical limitations including input/output bottlenecks ("dequantization" problem).

> **Reference**: Tang (2019) — *"A quantum-inspired classical algorithm for recommendation systems"* — demonstrated many proposed QML speedups are dequantizable.

### 13.2 Quantum-Enhanced OSINT and Signal Intelligence

- **Quantum optimization** (QAOA) for graph problems in network topology analysis
- **Quantum annealing** (D-Wave) for combinatorial optimization in traffic analysis
- **Grover's algorithm** applied to database search in large intelligence datasets

### 13.3 Classical ML Enhanced by Quantum Data

Hybrid approaches where quantum sensors generate data processed by classical ML:
- Quantum radar for stealth penetration/detection
- Quantum LiDAR for physical reconnaissance
- Quantum-enhanced spectrum analysis for RF intelligence

---

## 14. Quantum Random Number Generator (QRNG) Attacks

### 14.1 QRNG in Cryptographic Infrastructure

QRNGs are increasingly deployed in:
- Hardware Security Modules (HSMs) with quantum entropy sources
- QKD systems (for basis selection)
- Cloud cryptography services
- Next-gen TLS implementations

### 14.2 Attack Vectors on QRNGs

**Entropy source manipulation:**
```
Attack: Physical interference with photon source or detector
Method: RF interference, temperature manipulation, optical injection
Impact: Predictable "random" output → key compromise
```

**Bias attacks:**
```
Attack: Exploiting non-idealities in photon detection
Method: Statistical analysis of QRNG output for detectable bias
        (Von Neumann extractor bypass)
Impact: Reduced effective entropy → brute-force feasibility
```

**Supply chain compromise:**
```
Attack: Hardware-level backdoor in QRNG chip
Method: Modified LFSR post-processing stage introduces periodic bias
        (analogous to Dual EC DRBG backdoor)
Impact: Covert key prediction by adversary with backdoor knowledge
```

### 14.3 NIST Testing Requirements

NIST SP 800-90B defines statistical tests for entropy sources. Attackers targeting QRNG implementations focus on:
- Defeating post-processing conditioning
- Attacking the interface between QRNG hardware and host system
- Timing analysis of entropy accumulation

---

## 15. Threat Actors and Nation-State Quantum Programs

### 15.1 Known Nation-State Quantum Investments

| Country | Key Program | Estimated Investment | Offensive Quantum Focus |
|---------|------------|---------------------|------------------------|
| **China** | National Quantum Initiative (CNKI) | $15B+ | QKD networks, HNDL, Shor's hardware |
| **USA** | NQI (National Quantum Initiative) | $1.8B (2018-2023) + classified | PQC transition, quantum sensing |
| **Russia** | Rostech Quantum Center | ~$800M | Cryptanalysis, quantum radar |
| **EU** | Quantum Flagship | €1B | QKD infrastructure |
| **India** | National Quantum Mission | $730M | Cryptography, communication |

### 15.2 Attributed Quantum-Adjacent Operations

**"Volt Typhoon" (China-attributed):**
- CISA advisory confirms pre-positioning in critical infrastructure
- Assessed to be conducting HNDL operations for future quantum decryption
- Reference: CISA Advisory AA24-038A (2024)

**NSA BULLRUN Program (declassified Snowden documents):**
- Deliberately weakened NIST DRBG standards (Dual EC DRBG backdoor)
- Demonstrates willingness of nation-states to sabotage cryptographic standards
- Raises concern about potential influence on PQC standardization

### 15.3 GCHQ/NCSC Threat Assessment

UK NCSC assesses that:
- Adversaries are actively collecting high-value encrypted data for future quantum decryption
- Nation-states with advanced programs may achieve CRQC capability in the 2030s
- Critical national infrastructure should begin PQC migration immediately

---

## 16. Red Team Methodologies for Quantum-Era Threats

### 16.1 Quantum Cryptography Assessment Framework

```
Phase 1: RECONNAISSANCE
├── Identify cryptographic algorithms in use
│   ├── TLS certificate inspection (cipher suites, key sizes)
│   ├── SSH host key algorithms
│   ├── VPN/IPsec configuration analysis
│   └── Code signing certificate analysis
├── Map data sensitivity and retention requirements
└── Identify critical communications channels

Phase 2: THREAT MODELING
├── Classify data by HNDL risk
│   ├── Long-term secrets (>10yr) → CRITICAL
│   ├── Medium-term secrets (5-10yr) → HIGH
│   └── Short-term secrets (<5yr) → MEDIUM
├── Assess adversary capability (nation-state vs. criminal)
└── Map HNDL interception vectors

Phase 3: EXPLOITATION SIMULATION
├── Passive collection simulation
│   ├── TLS session capture and archival testing
│   ├── Cipher suite downgrade testing
│   └── Perfect Forward Secrecy validation
├── Implementation vulnerability testing
│   ├── PQC library side-channel assessment
│   ├── QRNG entropy validation
│   └── Quantum-safe protocol compliance testing
└── Hybrid classical/quantum attack simulation

Phase 4: REPORTING
├── Cryptographic inventory
├── Quantum risk timeline mapping
├── Migration roadmap recommendations
└── Crypto-agility assessment
```

### 16.2 Cryptographic Inventory Tooling

```bash
# TLS cipher suite enumeration
testssl.sh --cipher-per-proto target.com

# SSH algorithm enumeration  
nmap --script ssh2-enum-algos target.com

# Certificate analysis
sslyze --certinfo --elliptic_curves target.com:443

# Check for PFS (forward secrecy)
openssl s_client -connect target.com:443 | grep Cipher

# ECDHE vs RSA key exchange detection
curl -v --tlsv1.2 --ciphers ECDHE-RSA-AES256-GCM-SHA384 target.com
```

### 16.3 Quantum-Safe Protocol Assessment Checklist

```
TLS Configuration:
[ ] TLS 1.3 enforced (disables non-PFS cipher suites)
[ ] ECDHE/DHE key exchange (not RSA key exchange)
[ ] Hybrid PQC key exchange deployed (X25519Kyber768)
[ ] Certificate key sizes: RSA ≥ 3072-bit or ECC ≥ 384-bit
[ ] HSTS with preloading enabled

SSH:
[ ] Ed25519 or ECDSA-521 host keys (not RSA < 3072)
[ ] OpenSSH ≥ 9.0 with sntrup761x25519 (hybrid PQC) enabled
[ ] Key rotation schedule defined

Code Signing:
[ ] SHA-256 or SHA-3 minimum
[ ] ECC P-384 or RSA-4096 for signing keys
[ ] Certificate validity ≤ 1 year

PKI/CA Infrastructure:
[ ] Root CA transition plan to PQC
[ ] Intermediate CA key sizes ≥ 4096-bit RSA
[ ] Certificate Transparency log monitoring

VPN/IPsec:
[ ] IKEv2 only (IKEv1 deprecated)
[ ] Perfect Forward Secrecy enforced
[ ] Hybrid PQC groups in IKEv2 (RFC 8784 compliant)
```

### 16.4 Crypto-Agility Assessment

**Crypto-agility** is the ability to rapidly swap cryptographic algorithms without major architectural changes. A system fails this assessment if:

- Cryptographic algorithms are hardcoded (not configurable)
- Algorithm updates require full redeployment
- No cryptographic abstraction layer exists
- Certificate/key rotation takes >30 days
- Legacy system dependencies force use of deprecated algorithms

---

## 17. Tooling and Frameworks

### 17.1 Quantum Computing SDKs (for Security Research)

| Tool | Language | Focus | Repository |
|------|----------|-------|------------|
| **Qiskit** | Python | General QC; IBM backends | [github.com/Qiskit/qiskit](https://github.com/Qiskit/qiskit) |
| **Cirq** | Python | Google; NISQ algorithms | [github.com/quantumlib/Cirq](https://github.com/quantumlib/Cirq) |
| **PennyLane** | Python | QML; hybrid algorithms | [github.com/PennyLaneAI/pennylane](https://github.com/PennyLaneAI/pennylane) |
| **Q#** | Q#/Python | Microsoft; fault-tolerant | [github.com/microsoft/qsharp](https://github.com/microsoft/qsharp) |
| **Braket SDK** | Python | AWS quantum backends | [github.com/aws/amazon-braket-sdk-python](https://github.com/aws/amazon-braket-sdk-python) |
| **ProjectQ** | Python | Compiler; resource estimation | [github.com/ProjectQ-Framework/ProjectQ](https://github.com/ProjectQ-Framework/ProjectQ) |

### 17.2 Cryptanalysis and PQC Research Tools

| Tool | Purpose | Repository |
|------|---------|------------|
| **liboqs** | Open Quantum Safe — PQC implementations | [github.com/open-quantum-safe/liboqs](https://github.com/open-quantum-safe/liboqs) |
| **OQS-OpenSSL** | PQC in OpenSSL | [github.com/open-quantum-safe/openssl](https://github.com/open-quantum-safe/openssl) |
| **FPYLLL** | Lattice reduction attacks (LLL, BKZ) | [github.com/fplll/fpylll](https://github.com/fplll/fpylll) |
| **G6K** | General Sieve Kernel for lattice attacks | [github.com/fplll/g6k](https://github.com/fplll/g6k) |
| **SageMath** | Algebraic cryptanalysis | [github.com/sagemath/sage](https://github.com/sagemath/sage) |
| **SEAL** | Microsoft CKKS/BFV homomorphic encryption | [github.com/microsoft/SEAL](https://github.com/microsoft/SEAL) |
| **Pqcrypto** | Collection of PQC implementations | [github.com/rustpq/pqcrypto](https://github.com/rustpq/pqcrypto) |

### 17.3 Shor's Algorithm Implementations (Research/Educational)

```bash
# Qiskit implementation of Shor's algorithm (educational)
pip install qiskit qiskit-algorithms

# Reference implementation
git clone https://github.com/Qiskit/qiskit-textbook
# Navigate to: content/ch-algorithms/shors_algorithm.ipynb

# ProjectQ Shor's with resource estimation
git clone https://github.com/ProjectQ-Framework/ProjectQ
# See: examples/shor.py
```

### 17.4 QKD Simulation and Attack Research

| Tool | Description | Repository |
|------|------------|------------|
| **SimulaQron** | QKD network simulator | [github.com/SoftwareQuTech/SimulaQron](https://github.com/SoftwareQuTech/SimulaQron) |
| **NetSquid** | Quantum network simulator (Delft) | [netsquid.org](https://netsquid.org) |
| **QuNetSim** | Quantum network protocol simulator | [github.com/tqsd/QuNetSim](https://github.com/tqsd/QuNetSim) |
| **QKDP** | QKD protocol implementations | [github.com/jaronnix/qkdpy](https://github.com/jaronnix/qkdpy) |

### 17.5 Cryptographic Assessment Tools

```bash
# testssl.sh — comprehensive TLS analysis
git clone https://github.com/drwetter/testssl.sh

# SSLyze — Python TLS scanner with PQC awareness
pip install sslyze

# Cryptcheck — deep TLS/SSH assessment
docker run --rm -it didierstevens/cryptcheck target.com:443

# Nmap TLS scripts
nmap --script ssl-enum-ciphers,ssl-dh-params target.com

# quantum-safe-migration-toolkit (NIST NCCoE)
# https://www.nccoe.nist.gov/crypto-agility-considerations-migrating-post-quantum-cryptographic-standards
```

---

## 18. CVEs and Known Vulnerabilities in Quantum Stacks

### 18.1 Classical Cryptography Implementation CVEs (Quantum-Relevant)

| CVE | Affected System | Issue | Quantum Relevance |
|-----|----------------|-------|-------------------|
| CVE-2023-38408 | OpenSSH | RCE in ssh-agent | RSA key exposure |
| CVE-2022-0778 | OpenSSL | Infinite loop in BN_mod_sqrt() | ECC key handling |
| CVE-2021-3450 | OpenSSL | CA cert verification bypass | PKI chain trust |
| CVE-2020-0601 | Windows CryptoAPI | ECC spoofing (CurveBall) | ECDSA signature forgery |
| CVE-2019-14899 | Linux kernel | VPN tunnel inference | Traffic analysis |
| CVE-2017-5715 | Intel CPUs (Spectre) | Speculative execution | Key material leakage |

### 18.2 PQC Implementation Vulnerabilities

| Finding | Algorithm | Severity | Reference |
|---------|-----------|----------|-----------|
| Timing side-channel in decapsulation | Kyber | High | Ravi et al. (2020) |
| Fault attack on NTT | Dilithium | Critical | Bindel et al. (2022) |
| Cache-timing in FALCON FFT | FALCON | High | Guerreau-Lambet (2022) |
| PRNG weakness in reference implementation | NTRU | Medium | NIST PQC Round 3 report |
| Stack buffer overflow | liboqs v0.7.0 | Critical | GHSA-2jqr-44fj-4942 |

### 18.3 QKD System Vulnerabilities

| Vulnerability | Affected Systems | Demonstrated | Reference |
|--------------|-----------------|--------------|-----------|
| Detector blinding | ID Quantique Clavis2, MagiQ QPN 5505 | Yes (2010) | Lydersen et al. |
| Time-shift attack | Commercial APD-based QKD | Yes (2008) | Qi et al. |
| PNS attack (no decoy states) | Early commercial QKD | Yes (2000) | Brassard et al. |
| Laser side-channel | Coherent-source QKD | Yes (2019) | Huang et al. |
| Trojan horse | Plug-and-play QKD | Yes (2006) | Gisin et al. |

---

## 19. Defensive Mitigations and Crypto Agility

### 19.1 NIST PQC Migration Roadmap

```
2024: FIPS 203/204/205 published → Begin PQC library integration
2025: Legacy algorithm deprecation warnings (NIST SP 800-131B draft)
2026: Hybrid classical/PQC deployment recommended for all new systems  
2027: RSA/ECC deprecated for new implementations (NIST guidance)
2030: Full PQC transition required for federal systems (OMB M-23-02)
2035: Legacy algorithm support removal in FIPS-validated modules
```

### 19.2 Hybrid Key Exchange (Transitional Defense)

Deploy hybrid schemes combining classical and PQC algorithms:

```
X25519 + Kyber768 (IETF draft-ietf-tls-hybrid-design)
→ Implemented in: Chrome, Cloudflare, AWS, Cloudflare HPKE

IKEv2 with KEM combination (RFC 9370)
→ Combines ECDH with ML-KEM for IPsec/VPN

OpenSSH 9.0+ default:
sntrup761x25519-sha512@openssh.com
→ Already hybrid PQC by default
```

### 19.3 Crypto-Agility Design Patterns

```python
# Python example: Crypto-agile TLS configuration
from cryptography.hazmat.primitives.asymmetric import ec, padding
from oqs import KeyEncapsulation  # liboqs Python bindings

class CryptoAgileKEM:
    def __init__(self, algorithm='Kyber768'):
        """
        Algorithm registry allows hot-swapping without code changes
        """
        self.algorithm = algorithm
        self._kem_backends = {
            'Kyber768': lambda: KeyEncapsulation('Kyber768'),
            'Kyber1024': lambda: KeyEncapsulation('Kyber1024'),
            'ClassicMcEliece': lambda: KeyEncapsulation('ClassicMcEliece348864'),
        }
    
    def encapsulate(self, public_key):
        kem = self._kem_backends[self.algorithm]()
        ciphertext, shared_secret = kem.encap_secret(public_key)
        return ciphertext, shared_secret
```

### 19.4 HNDL-Specific Mitigations

```
Immediate (Today):
✓ Enable TLS 1.3 exclusively (mandatory ECDHE/DHE)
✓ Deploy HSTS with preloading
✓ Rotate long-lived RSA certificates to ECC P-384
✓ Identify and archive high-sensitivity data transmission logs
✓ Enable hybrid PQC key exchange where available

Short-Term (1-2 years):
✓ Deploy PQC KEM in VPN/IPsec gateways
✓ Replace DKIM/SPF/SMIME with PQC equivalents
✓ Certificate lifecycle automation (shorter validity periods)
✓ Establish quantum risk register

Medium-Term (2-5 years):
✓ Full PKI migration to PQC algorithms
✓ Code signing infrastructure PQC transition
✓ Hardware token/HSM firmware updates for PQC support
✓ Supply chain verification for PQC hardware
```

---

## 20. Research Repositories and Datasets

### 20.1 Key Research Repositories

| Repository | Description | URL |
|-----------|-------------|-----|
| **NIST PQC Project** | All submissions, evaluations, final standards | [csrc.nist.gov/projects/post-quantum-cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography) |
| **Open Quantum Safe (OQS)** | Production PQC libraries and integrations | [openquantumsafe.org](https://openquantumsafe.org) |
| **FPLLL** | Lattice reduction algorithms (LLL, BKZ, Sieve) | [github.com/fplll/fplll](https://github.com/fplll/fplll) |
| **QuantumKatas** | Microsoft quantum learning + algorithms | [github.com/microsoft/QuantumKatas](https://github.com/microsoft/QuantumKatas) |
| **Awesome Post-Quantum** | Curated PQC resources | [github.com/veorq/awesome-post-quantum](https://github.com/veorq/awesome-post-quantum) |
| **CryptoHack** | Cryptography CTF platform | [cryptohack.org](https://cryptohack.org) |
| **QRL (Quantum Resistant Ledger)** | Blockchain using XMSS (PQC) | [github.com/theQRL/QRL](https://github.com/theQRL/QRL) |
| **Lattice Estimator** | Security parameter estimation for LWE | [github.com/malb/lattice-estimator](https://github.com/malb/lattice-estimator) |
| **CHES/TCHES Papers** | Side-channel + hardware security research | [tches.iacr.org](https://tches.iacr.org) |
| **eprint.iacr.org** | Cryptology ePrint — preprint archive | [eprint.iacr.org](https://eprint.iacr.org) |

### 20.2 Threat Intelligence and Policy

| Resource | Type | URL |
|---------|------|-----|
| CISA PQC Initiative | Policy/guidance | [cisa.gov/quantum](https://www.cisa.gov/quantum) |
| NSA CNSS Advisory | National security guidance | [nsa.gov/Press-Room/Cybersecurity-Advisories-Guidance](https://www.nsa.gov) |
| ENISA Quantum Threat Landscape | EU threat report | [enisa.europa.eu](https://www.enisa.europa.eu) |
| NCCoE Migration Project | Practical migration tooling | [nccoe.nist.gov](https://www.nccoe.nist.gov) |
| ANSSI Recommendations | French NCSA PQC guidance | [ssi.gouv.fr](https://www.ssi.gouv.fr) |

---

## 21. Scientific References

### Foundational Quantum Algorithms

1. **Shor, P.W.** (1994). *Algorithms for quantum computation: Discrete logarithms and factoring*. Proceedings 35th Annual Symposium on Foundations of Computer Science. IEEE. https://doi.org/10.1109/SFCS.1994.365700

2. **Grover, L.K.** (1996). *A fast quantum mechanical algorithm for database search*. Proceedings of the 28th Annual ACM Symposium on Theory of Computing. https://doi.org/10.1145/237814.237866

3. **Brassard, G., Høyer, P., Tapp, A.** (1998). *Quantum cryptanalysis of hash and claw-free functions*. LATIN'98: Theoretical Informatics. Springer. https://doi.org/10.1007/BFb0054319

### Quantum Cryptography and QKD

4. **Bennett, C.H., Brassard, G.** (1984). *Quantum cryptography: Public key distribution and coin tossing*. Proceedings of IEEE International Conference on Computers, Systems, and Signal Processing. (BB84 protocol — foundational)

5. **Lydersen, L., Wiechers, C., Wittmann, C., Elser, D., Skaar, J., Makarov, V.** (2010). *Hacking commercial quantum cryptography systems by tailored bright illumination*. Nature Photonics, 4(10), 686–689. https://doi.org/10.1038/nphoton.2010.214

6. **Brassard, G., Lütkenhaus, N., Mor, T., Sanders, B.C.** (2000). *Limitations on practical quantum cryptography*. Physical Review Letters, 85(6), 1330. https://doi.org/10.1103/PhysRevLett.85.1330

7. **Vakhitov, A., Makarov, V., Hjelme, D.R.** (2001). *Large pulse attack as a method of conventional optical eavesdropping in quantum cryptography*. Journal of Modern Optics, 48(13), 2023–2038. https://doi.org/10.1080/09500340110075988

8. **Qi, B., Fung, C.H.F., Lo, H.K., Ma, X.** (2007). *Time-shift attack in practical quantum cryptosystems*. Quantum Information & Computation, 7(1-2), 73–82.

### Resource Estimation for Quantum Attacks

9. **Webber, M., Elfving, V., Weidt, S., Hensinger, W.K.** (2022). *The impact of hardware specifications on reaching quantum advantage in the fault tolerant regime*. AVS Quantum Science, 4(1). https://doi.org/10.1116/5.0073075

10. **Jaques, S., Naehrig, M., Roetteler, M., Virdia, F.** (2020). *Implementing Grover oracles for quantum key search on AES and LowMC*. EUROCRYPT 2020, Lecture Notes in Computer Science. Springer. https://doi.org/10.1007/978-3-030-45724-2_10

11. **Roetteler, M., Naehrig, M., Svore, K.M., Lauter, K.** (2017). *Quantum resource estimates for computing elliptic curve discrete logarithms*. ASIACRYPT 2017, Lecture Notes in Computer Science. https://doi.org/10.1007/978-3-319-70697-9_29

12. **Beauregard, S.** (2003). *Circuit for Shor's algorithm using 2n+3 qubits*. Quantum Information & Computation, 3(2), 175–185.

### Post-Quantum Cryptography

13. **Avanzi, R., Bos, J., Ducas, L., et al.** (2021). *CRYSTALS-Kyber: Algorithm specifications and supporting documentation*. NIST PQC Round 3 Submission. https://pq-crystals.org/kyber/

14. **Ducas, L., Kiltz, E., Lepoint, T., et al.** (2021). *CRYSTALS-Dilithium: Algorithm specifications and supporting documentation*. NIST PQC Round 3 Submission. https://pq-crystals.org/dilithium/

15. **Albrecht, M.R., Ducas, L., Herold, G., Kirshanova, E., Postlethwaite, E.W., Stevens, M.** (2021). *The general sieve kernel and new records in lattice reduction*. EUROCRYPT 2019. https://doi.org/10.1007/978-3-030-17656-3_23

16. **Chen, L., Jordan, S., Liu, Y.K., Moody, D., Peralta, R., Perlner, R., Smith-Tone, D.** (2016). *Report on post-quantum cryptography*. NISTIR 8105. https://doi.org/10.6028/NIST.IR.8105

### Side-Channel Attacks on PQC

17. **Ravi, P., Roy, S.S., Chattopadhyay, A., Bhasin, S.** (2019). *Generic side-channel attacks on CCA-secure lattice-based PKE and KEMs*. IACR Transactions on Cryptographic Hardware and Embedded Systems. https://doi.org/10.13154/tches.v2020.i3.307-335

18. **Migliore, V., Gérard, B., Tibouchi, M., Fouque, P.A.** (2019). *Masking Dilithium: Efficient implementation and side-channel evaluation*. ACNS 2019. https://doi.org/10.1007/978-3-030-21568-2_14

19. **Guerreau-Lambet, P., Karaklajic, D., Wafo-Tapa, G.** (2022). *The hidden parallelepiped is back again: Power analysis attacks on Falcon*. IACR Transactions on Cryptographic Hardware and Embedded Systems. https://doi.org/10.46586/tches.v2022.i1.141-164

### Quantum Machine Learning and Security

20. **Tang, E.** (2019). *A quantum-inspired classical algorithm for recommendation systems*. Proceedings of the 51st Annual ACM STOC. https://doi.org/10.1145/3313276.3316310

21. **Biamonte, J., Wittek, P., Pancotti, N., Rebentrost, P., Wiebe, N., Lloyd, S.** (2017). *Quantum machine learning*. Nature, 549(7671), 195–202. https://doi.org/10.1038/nature23474

### Harvest Now Decrypt Later / HNDL

22. **Mosca, M.** (2018). *Cybersecurity in an era with quantum computers: Will we be ready?* IEEE Security & Privacy, 16(5), 38–41. https://doi.org/10.1109/MSP.2018.3761723

23. **Bernstein, D.J., Lange, T.** (2017). *Post-quantum cryptography*. Nature, 549(7671), 188–194. https://doi.org/10.1038/nature23461

### Policy and Standardization

24. **NIST.** (2024). *Module-Lattice-Based Key-Encapsulation Mechanism Standard*. FIPS 203. https://doi.org/10.6028/NIST.FIPS.203

25. **NIST.** (2024). *Module-Lattice-Based Digital Signature Standard*. FIPS 204. https://doi.org/10.6028/NIST.FIPS.204

26. **NIST.** (2024). *Stateless Hash-Based Digital Signature Standard*. FIPS 205. https://doi.org/10.6028/NIST.FIPS.205

27. **NSA/CISA.** (2022). *Quantum-Resistant Cryptography: Preparing for the future*. NSA Cybersecurity Information Sheet. https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/3148990/

28. **Alagic, G., et al.** (2022). *Status report on the third round of the NIST post-quantum cryptography standardization process*. NISTIR 8413. https://doi.org/10.6028/NIST.IR.8413

---

## Appendix A — Quantum Threat Readiness Self-Assessment

```
Score each item 0-3: 0=Not Started, 1=Planned, 2=In Progress, 3=Complete

VISIBILITY
[ ] Cryptographic algorithm inventory completed
[ ] Data classification by quantum sensitivity completed  
[ ] Crypto-agility assessment performed
[ ] Vendor quantum roadmap collected

IMMEDIATE HARDENING
[ ] TLS 1.3 enforced organization-wide
[ ] Perfect Forward Secrecy enabled on all endpoints
[ ] SSH hybrid PQC key exchange enabled
[ ] RSA key sizes ≥ 3072-bit or migrated to ECC P-384

PLANNING
[ ] HNDL risk register established
[ ] PQC migration roadmap approved by leadership
[ ] NIST FIPS 203/204/205 integration timeline defined
[ ] Certificate lifecycle automation planned

TESTING
[ ] PQC library implementation tested for side-channels
[ ] Hybrid TLS deployment validated
[ ] Quantum-safe VPN piloted in test environment
[ ] QRNG entropy validation performed (if applicable)

GOVERNANCE
[ ] Quantum risk included in annual security review
[ ] Cryptography officer / PQC lead designated
[ ] Vendor questionnaire for PQC readiness deployed
[ ] Incident response plan updated for cryptographic break scenarios

SCORE: ___/60
0-15: Critical quantum risk exposure
16-30: High risk — begin immediate PQC planning
31-45: Moderate — on track, accelerate deployment
46-60: Low — maintain and monitor
```

---

## Appendix B — Glossary

| Term | Definition |
|------|-----------|
| **CRQC** | Cryptographically Relevant Quantum Computer — capable of breaking RSA/ECC at practical scale |
| **HNDL** | Harvest Now, Decrypt Later — storing encrypted data for future quantum decryption |
| **QKD** | Quantum Key Distribution — using quantum mechanics for provably secure key exchange |
| **PQC** | Post-Quantum Cryptography — classical algorithms believed resistant to quantum attacks |
| **QBER** | Quantum Bit Error Rate — error metric in QKD; elevated QBER indicates eavesdropping |
| **LWE** | Learning With Errors — mathematical hardness assumption underlying Kyber/Dilithium |
| **NTRU** | N-th degree Truncated polynomial Ring Units — lattice problem underlying FALCON |
| **NTT** | Number Theoretic Transform — efficient polynomial multiplication in lattice schemes |
| **PNS** | Photon Number Splitting — QKD attack exploiting multi-photon pulses |
| **SPA/DPA** | Simple/Differential Power Analysis — side-channel attacks on hardware |
| **Q-Day** | The day a CRQC becomes operational — industry shorthand for the cryptographic transition |
| **SVP** | Shortest Vector Problem — NP-hard lattice problem foundational to PQC security |
| **QFT** | Quantum Fourier Transform — core operation in Shor's algorithm |
| **VQE** | Variational Quantum Eigensolver — hybrid QML algorithm for optimization |
| **NISQ** | Noisy Intermediate-Scale Quantum — current era of imperfect, limited quantum computers |
| **Crypto-agility** | System design allowing rapid algorithm replacement without full architectural change |

---

*Document Version: 1.0 | Classification: Public Research Reference*  
*Last Updated: 2025 | Maintained for: Red Team Leaders Research Division*  
*All techniques described are for authorized security research only.*

> "The question is not whether quantum computers will break our cryptography. The question is whether we will have migrated before they do." — Michele Mosca
