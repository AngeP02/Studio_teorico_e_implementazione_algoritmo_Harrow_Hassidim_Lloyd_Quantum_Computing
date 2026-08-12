# Algoritmo Harrow Hassidim Lloyd — Soluzione Quantistica di Sistemi Lineari

Studio e implementazione dell'algoritmo HHL in Qiskit per la soluzione di sistemi lineari $Ax = b$.

## Contenuto

- Studio teorico delle diverse fasi dell'algoritmo
- Implementazione del circuito HHL per sistemi 2×2 e 4×4
- Simulazione e confronto con la soluzione classica
- Esecuzione su hardware reale IBM Quantum

## Descrizione

L'algoritmo HHL risolve un sistema lineare $Ax = b$ producendo lo stato quantistico $|x\rangle \propto A^{-1}|b\rangle$, sfruttando la Quantum Phase Estimation (QPE) per codificare gli autovalori e una rotazione condizionale per realizzarne l'inversione. In condizioni ideali offre uno speedup esponenziale rispetto ai risolutori classici per matrici sparse e ben condizionate.

Il progetto copre l'intera pipeline:

- Analisi spettrale e normalizzazione degli autovalori
- Costruzione dell'operatore unitario $U = e^{iAt}$ tramite esponenziazione matriciale
- Circuito QPE con registro di controllo configurabile
- Rotazione condizionale $R_Y$ per l'inversione degli autovalori
- QPE inversa (uncompute) e post-selezione sul qubit ausiliario
- Esecuzione su simulatore locale (Aer) e su hardware IBM reale

## Pipeline dell'algoritmo

| Fase | Descrizione |
|---|---|
| Preparazione stato | Codifica il vettore $b$ normalizzato nel registro di sistema |
| QPE | Estrae gli autovalori come fasi binarie tramite Hadamard + $C$-$U^{2^k}$ + IQFT |
| Rotazione condizionale | Applica $R_Y(2\arcsin(C/\tilde{\lambda}_j))$ al qubit ausiliario |
| QPE inversa | Esegue l'uncompute del registro di controllo per disaccoppiarlo dal sistema |
| Post-selezione | Filtra i risultati con qubit ausiliario = 1; il registro di sistema contiene $|x\rangle$ |

## Configurazione

All'inizio del notebook un blocco di parametri globali controlla tutte le modalità di esecuzione:

```python
DIMENSIONE_SISTEMA = 2      # 2 → sistema 2×2 (1 qubit di sistema)
                            # 4 → sistema 4×4 (2 qubit di sistema)

BACKEND_TIPO = "simulatore" # "simulatore" → AerSimulator locale
                            # "hardware"   → dispositivo IBM reale

M_QUBIT_CONTROLLO = 3       # Precisione QPE: risoluzione = 1/2^m
NUM_SHOTS = 16384
```
## Risultati sperimentali

Sono state testate quattro configurazioni: sistemi 2×2 e 4×4, ciascuno su simulatore e su hardware IBM reale.

### Simulatore

| Sistema | Fedeltà $F$ | Prob. successo $P_1$ | Limite teorico $1/\kappa^2$ |
|---|---|---|---|
| 2×2 | 0.957 | 0.349 | 0.250 |
| 4×4 | 0.999 | 0.289 | 0.143 |

Il sistema 4×4 raggiunge fedeltà superiore al 2×2 perché le sue fasi spettrali si adattano più favorevolmente alla griglia di quantizzazione della QPE a $m = 3$ bit.

### Hardware IBM (reale)

| Sistema | Fedeltà $F$ | $\Delta F$ rispetto al simulatore | Gate a 2 qubit |
|---|---|---|---|
| 2×2 | 0.934 | −0.023 | 70 |
| 4×4 | 0.706 | −0.293 | 242 |

Il circuito 2×2 (profondità 8, 70 gate a 2 qubit) è eseguibile in modo affidabile su hardware NISQ corrente. Il circuito 4×4 (profondità 13, 242 gate a 2 qubit) accumula rumore sufficiente a rendere la distribuzione post-selezione quasi uniforme; la probabilità di successo $P_1$ rimane comunque sopra il limite teorico in entrambi i casi.

### Precisione QPE al variare di $m$

| $m$ | Risoluzione $1/2^m$ | Verificato |
|---|---|---|
| 2 | 0.250 | ✗ |
| 3 | 0.125 | ✓ |
| 4 | 0.0625 | ✓ |
| ≥5 | dimezza per ogni bit | ✓ |

L'errore si dimezza ad ogni qubit aggiuntivo, confermando la convergenza esponenziale teorica. $m = 2$ è insufficiente per entrambi i sistemi testati.


## Limitazioni note

**Preparazione dello stato.** Caricare un vettore classico $b$ nelle ampiezze quantistiche può richiedere $O(N)$ operazioni in assenza di una Quantum RAM (QRAM), annullando potenzialmente il vantaggio per input puramente classici.

**Simulazione hamiltoniana.** L'implementazione utilizza l'esponenziazione matriciale esatta (`scipy.linalg.expm`), che produce gate unitari densi. Il gate count scala sfavorevolmente con $N$; per matrici sparse di grandi dimensioni sarebbe necessaria una decomposizione di Trotter–Suzuki.

**Numero di condizionamento.** La complessità dell'algoritmo è $O(s^2 \kappa^2 \log N / \varepsilon)$. Il vantaggio rispetto ai solver classici è significativo soltanto per matrici sparse e ben condizionate ($\kappa \ll N$).

**Estrazione dell'output.** HHL produce uno stato quantistico, non un vettore classico. Ricostruire tutte le componenti richiede $O(N)$ misure ripetute; il caso d'uso pratico è la stima di valori attesi $\langle x | M | x \rangle$.

## Autori
Angelica Porco, Hanna Andrieieva
