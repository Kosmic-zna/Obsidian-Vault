### 🔢 The Grand Master Conversion Matrix

|**From ↓ \ To →**|**To Binary (Base 2)**|**To Octal (Base 8)**|**To Decimal (Base 10)**|**To Hexadecimal (Base 16)**|
|---|---|---|---|---|
|**From Binary (Base 2)**|—|Group bits into **triplets (3s)** from right to left; convert each triplet to an octal digit ($0\text{--}7$).|Sum the **positional powers of 2**: $\sum (d_i \times 2^i)$.|Group bits into **nibbles (4s)** from right to left; convert each nibble to a hex character ($0\text{--}9, \text{A--F}$).|
|**From Octal (Base 8)**|Expand each octal digit directly into its **3-bit binary equivalent** ($000\text{ to }111$).|—|Sum the **positional powers of 8**: $\sum (d_i \times 8^i)$.|Convert each octal digit to **3 bits**, concatenate the binary string, then regroup into **4-bit nibbles**.|
|**From Decimal (Base 10)**|Repeatedly **divide by 2**; read remainders from bottom to top (MSB to LSB).|Repeatedly **divide by 8**; read remainders from bottom to top.|—|Repeatedly **divide by 16**; read remainders from bottom to top, converting remainders $10\text{--}15$ to $\text{A--F}$.|
|**From Hexadecimal (Base 16)**|Expand each hex digit directly into its **4-bit binary equivalent** ($0000\text{ to }1111$).|Convert each hex character to **4 bits**, concatenate the binary string, then regroup into **3-bit triplets**.|Sum the **positional powers of 16**: $\sum (d_i \times 16^i)$, substituting letters with their values ($A=10 \dots F=15$).|—|

### 🗺️ The Universal "Hub-and-Spoke" Architecture

Whenever you feel disoriented converting between bases that are not powers of two (e.g. converting between Base 8 and Base 16), **Binary (Base 2)** serves as your computational nexus:

```
          ┌─────────────┐
          │   Decimal   │
          │  (Base 10)  │
          └──────┬──────┘
       Repeated  │  Positional
       Division  │  Weights
                 ▼
          ┌─────────────┐
   ┌─────►│   Binary    │◄─────┐
   │      │  (Base 2)   │      │
   │      └──────┬──────┘      │
3-Bit            │            4-Bit
Expansion        │            Expansion
   │             ▼             │
┌──┴──────────┐     ┌──────────┴──┐
│    Octal    │     │ Hexadecimal │
│  (Base 8)   │     │  (Base 16)  │
└─────────────┘     └─────────────┘
```

- **Rule of 3 (Octal):** Because $2^3 = 8$, **1 octal digit $\Longleftrightarrow$ exactly 3 binary bits**.
    
- **Rule of 4 (Hexadecimal):** Because $2^4 = 16$, **1 hexadecimal digit $\Longleftrightarrow$ exactly 4 binary bits (a nibble)**.
    

### 🧮 Fast-Reference Lookup Tables

#### 1. Octal $\longleftrightarrow$ Binary (3-Bit Alignment)

|**Octal Digit**|**Binary Representation**|
|---|---|
|`0`|`000`|
|`1`|`001`|
|`2`|`010`|
|`3`|`011`|
|`4`|`100`|
|`5`|`101`|
|`6`|`110`|
|`7`|`111`|

#### 2. Hexadecimal $\longleftrightarrow$ Binary (4-Bit Alignment)

| **Hex Digit** | **Decimal Value** | **Binary (Nibble)** |
| ------------- | ----------------- | ------------------- |
| `0`           | 0                 | `0000`              |
| `1`           | 1                 | `0001`              |
| `2`           | 2                 | `0010`              |
| `3`           | 3                 | `0011`              |
| `4`           | 4                 | `0100`              |
| `5`           | 5                 | `0101`              |
| `6`           | 6                 | `0110`              |
| `7`           | 7                 | `0111`              |
| `8`           | 8                 | `1000`              |
| `9`           | 9                 | `1001`              |
| `A`           | 10                | `1010`              |
| `B`           | 11                | `1011`              |
| `C`           | 12                | `1100`              |
| `D`           | 13                | `1101`              |
| `E`           | 14                | `1110`              |
| `F`           | 15                | `1111`              |

### 🔬 Worked Conversions: Tracing the Number $173_{10}$

Let us trace a single decimal value, **$173_{10}$**, across every single target base to inspect the mechanics in practice:

#### 1. Decimal to Binary $\rightarrow 173_{10} \implies 10101101_2$

Repeated integer division by 2 recording remainder ($R$):

- $173 \div 2 = 86 \quad (R = 1)$ $\rightarrow \text{Least Significant Bit (LSB)}$
    
- $86 \div 2 = 43 \quad (R = 0)$
    
- $43 \div 2 = 21 \quad (R = 1)$
    
- $21 \div 2 = 10 \quad (R = 1)$
    
- $10 \div 2 = 5 \quad (R = 0)$
    
- $5 \div 2 = 2 \quad (R = 1)$
    
- $2 \div 2 = 1 \quad (R = 0)$
    
- $1 \div 2 = 0 \quad (R = 1)$ $\rightarrow \text{Most Significant Bit (MSB)}$
    
- Reading remainders bottom to top: **$10101101_2$**
    

#### 2. Binary to Octal $\rightarrow 10101101_2 \implies 255_8$

Group bits into triplets starting strictly from the right (padding the leftmost group with leading zeros):

$$\underbrace{010}_{2} \quad \underbrace{101}_{5} \quad \underbrace{101}_{5} \implies \mathbf{255_8}$$

- Check via powers of 8: $(2 \times 8^2) + (5 \times 8^1) + (5 \times 8^0) = 128 + 40 + 5 = 173_{10}$.
    

#### 3. Binary to Hexadecimal $\rightarrow 10101101_2 \implies \text{AD}_{16}$

Group bits into quartets (nibbles) starting from the right:

$$\underbrace{1010}_{\text{A (10)}} \quad \underbrace{1101}_{\text{D (13)}} \implies \mathbf{\text{AD}_{16}}$$

- Check via powers of 16: $(10 \times 16^1) + (13 \times 16^0) = 160 + 13 = 173_{10}$.
    

#### 4. Octal to Hexadecimal $\rightarrow 255_8 \implies \text{AD}_{16}$

Bridge through Binary:

1. Expand each octal digit into 3 bits: $2 \rightarrow 010$, $5 \rightarrow 101$, $5 \rightarrow 101$.
    
2. Form bit string: `010101101` $\rightarrow$ drop redundant leading zero $\rightarrow$ `10101101`.
    
3. Regroup into 4-bit nibbles: `1010` and `1101` $\rightarrow$ $\mathbf{\text{AD}_{16}}$.