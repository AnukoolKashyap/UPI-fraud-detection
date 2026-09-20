# Phase 2 — Research for Development

**Detecting Authorized Push Payment Fraud in UPI**
Building on reference papers **P3** and **P4** of the Phase 1 literature review.

---

## 0. What the faculty asked for, and how this plan answers it

Two instructions:

1. **Build on P3 and P4.** These are reference entries `[3]` and `[4]` in Section 5 of
   `Phase1_UPI_Fraud_Detection.docx`:

   | Label | Paper | Venue | What it contributes |
   |---|---|---|---|
   | **P3** | *HiFraud: Hierarchical Privacy-Preserving Federated Learning with Star-Chain Knowledge Transfer for Cross-Institutional Fraud Detection* | ScienceDirect, 2026 · [S1546221826005278](https://www.sciencedirect.com/org/science/article/pii/S1546221826005278) | Fraud detection **across institutions** without pooling raw transaction data |
   | **P4** | *Heterophily outlier temporal aware graph neural network for online fraud detection* | ScienceDirect, 2026 · [S0957417426021871](https://www.sciencedirect.com/science/article/pii/S0957417426021871) | Fraud detection on the **transaction graph**, aware of heterophily and time |

2. **Move from literature survey to research for development** — an implemented,
   experimental contribution rather than a review.

The move this plan makes: **take the detection signal off the individual account and put it
on the payment graph (P4), in the multi-institution setting UPI actually runs in (P3).**

That is not an arbitrary combination. It falls directly out of Phase 1's own central
finding, and it is the only direction that survives the APP-fraud framing.

---

## 1. Building on P3 and P4

### P3 — HiFraud: cross-institutional federated fraud detection

**What it does (as cited in Phase 1).** HiFraud performs fraud detection across multiple
financial institutions using hierarchical privacy-preserving federated learning with
star-chain knowledge transfer, so participating institutions collaborate on a shared
detection model without exchanging raw transaction records. Phase 1 cites it for its
PaySim benchmark (6.36M transactions, ~0.13% fraud).

**Its limitation, and the gap.** HiFraud is validated on general-purpose fraud benchmarks
where each institutional silo is an arbitrary partition of a dataset. Those partitions are
statistically convenient — they do not reproduce the structure of a real payment rail.

UPI's structure is the gap. A single UPI payment traverses **payer PSP → NPCI switch →
payee bank**, and the three parties see different halves of it. The payer's bank sees a
customer draining their balance to an unfamiliar VPA. The beneficiary's bank sees an
account receiving many small credits from unrelated strangers and cashing out. **Neither
sees the mule network.** That is not a privacy preference — it is the operating reality of
an interoperable rail, and it is the exact problem federated learning exists to solve.

**What Phase 2 closes.** Re-run the federated setting over a partition that mirrors UPI's
actual topology: split the transaction graph by *institution of the account*, not randomly,
producing silos that are **non-IID by construction** — a beneficiary-heavy PSP holds a very
different fraud distribution from a payer-heavy bank. Then measure what a bank actually
buys by federating: the lift of the federated model over each silo's best local model.
A silo-local model that already matches the federated one has no business federating; a
silo that gains 15 points of recall has a concrete argument to take to a risk committee.
Report that lift **per silo**, which the general-purpose benchmarks do not.

### P4 — Heterophily outlier temporal aware GNN

**What it does (as cited in Phase 1).** P4 applies a graph neural network to
PaySim-derived fraud detection, explicitly handling **heterophily** — fraudulent nodes
connect mostly to *normal* neighbours rather than to each other, which breaks the
homophily assumption ordinary GNNs rely on — together with **temporal** awareness of when
edges appear. Phase 1 cites it as the current state-of-the-art modelling direction for
this dataset family.

**Its limitation, and the gap.** P4 targets online fraud in the standard framing: the
fraudulent *actor* is the anomalous node, and the task is to find it.

**In APP fraud that framing inverts, and this is the crux of the whole paper.** The
transaction is authorized by the genuine account holder, from their own device, with their
own PIN. The victim is not anomalous — they are an ordinary customer having the worst day
of their year. There is no account-takeover signature to find, because there was no
takeover.

The anomalous node is on the **other side**: the **mule account** collecting the proceeds.
Its signature is structural and temporal, not arithmetic — a fan-in of small credits from
mutually unconnected strangers, a short active lifetime, a rapid fan-out or cash-out, and
a first-contact edge to each victim.

Phase 1 proved the negative case empirically. Its Decision Tree scored **1.0000 across
precision, recall, F1, ROC-AUC and AUC-PR** — and the draft is right to flag this rather
than celebrate it. That score is a *leak*: PaySim injects fraud through a deterministic
balance-draining rule, leaving an arithmetic signature that
`balanceDiffOrig` / `origBalanceZeroed` recover almost by construction. Real APP fraud
leaves no such trace. Balance arithmetic is a dead end, and Phase 1 established that.

**What Phase 2 closes.** Reformulate the task from *anomalous-payer detection* to
**beneficiary/mule-account detection**, and give P4's heterophily-aware temporal GNN the
graph where that signal actually lives. Heterophily is not incidental here — it is the
defining property of the mule subgraph: a mule's neighbours are overwhelmingly legitimate
victims, so any model assuming "fraud connects to fraud" is structurally blind to it.

### The joint contribution

> **A federated, heterophily-aware temporal graph model for APP-fraud mule-account
> detection, evaluated across UPI-shaped institutional silos.**

Neither half suffices alone. P4 without P3 assumes one institution sees the whole graph —
no UPI participant does. P3 without P4 federates tabular per-account features, which Phase 1
showed are the wrong features. The traceability the faculty asked for:

```
P3 gap: federated silos are artificial partitions, not real rail topology
      └─> Phase 2: partition by institution, non-IID by construction, report per-silo lift

P4 gap: assumes the fraudulent actor is the anomalous node
      └─> Phase 2: invert to mule-side detection, where APP fraud is actually visible

Phase 1 finding: balance arithmetic gives a leaked 1.0000 and generalizes to nothing
      └─> Phase 2: structural + temporal features, with the leaky features ablated out
```

---

## 2. Datasets

**Scope honesty, stated up front.** NPCI does not publicly release UPI transaction records.
No experiment in this paper runs on real UPI data, and the paper must not imply otherwise.
Everything below is a mobile-money **proxy**, and every claim is scoped to what the proxy
supports. This constraint is inherited from Phase 1 and is not a new weakness.

| Dataset | Role | Access | Why |
|---|---|---|---|
| **MoMTSim** | **Primary** | Reference [1], ScienceDirect 2025 | Phase 1 already identified it as the successor to PaySim, with calibrated SIM-swap, refund and fake-credential scenarios. Richer fraud mechanics than a balance-drain rule. |
| **PaySim** | Baseline / comparability | [kaggle.com/datasets/ealaxi/paysim1](https://www.kaggle.com/datasets/ealaxi/paysim1) | 6.3M transactions, 0.13% fraud. Keeps Phase 2 comparable to Phase 1 and to P3/P4's own benchmarks. |
| **IEEE-CIS Fraud Detection** | Optional robustness check | Kaggle | Different domain; tests whether the mule-side framing transfers. |

**Graph construction.** `nameOrig → nameDest` becomes a directed edge; accounts become
nodes; `step` (1 hour) gives the temporal ordering. Node features: transaction counts,
in/out degree over rolling windows, distinct-counterparty count, active lifetime, cash-out
ratio. Edge features: amount, type, hours since the pair's first interaction.

**Mule labelling.** PaySim labels the *transaction* as fraudulent. Derive an
account-level label: a destination account is a mule if it receives ≥1 `isFraud`
transaction. Report the resulting mule-account base rate — it will differ from the 0.13%
transaction-level rate, and the paper must state both so the imbalance is not misread.

> **Known proxy limitation, to be stated in the paper.** PaySim's fraud is account-takeover,
> not victim-authorized. The mule-side *structure* (fan-in, short lifetime, rapid cash-out)
> transfers; the victim-side *behaviour* does not. Phase 2 tests the structural hypothesis;
> the survey (§5) probes the behavioural half. Do not overclaim past that line.

---

## 3. Models

| Tier | Model | Purpose |
|---|---|---|
| **Baseline** | Logistic Regression, Decision Tree | Carried from Phase 1, re-run on mule-account labels, **with leaky balance features ablated** |
| **Strong tabular** | Random Forest, XGBoost, LightGBM | The honest bar a GNN must beat — a graph model that cannot beat XGBoost on engineered graph features is not worth the complexity |
| **Graph (P4 line)** | Heterophily-aware temporal GNN | The proposed model |
| **Unsupervised** | Isolation Forest, Autoencoder | Phase 1's stated plan; also the realistic deployment mode where labels lag by weeks |
| **Federated (P3 line)** | Best model above, trained via FedAvg across institutional silos | Tests the cross-institutional claim |

**Ablations — these carry the argument, not the headline number.**

1. **Leakage ablation.** With vs. without `balanceDiffOrig`, `balanceDiffDest`,
   `origBalanceZeroed`. Expect the 1.0000 to collapse. *That collapse is a result*, and it
   substantiates Phase 1's honest flag with evidence.
2. **Heterophily ablation.** Heterophily-aware GNN vs. a vanilla homophilic GCN/GraphSAGE.
   Isolates whether P4's specific contribution matters for mule detection.
3. **Temporal ablation.** Time-aware edges vs. a static aggregated graph.
4. **Federation ablation.** Centralized vs. federated vs. each silo local-only — the
   per-silo lift table from §1.

---

## 4. Evaluation

**Accuracy is banned from this paper.** At a 0.13% base rate, predicting "never fraud"
scores 99.87%. Report instead:

- **AUC-PR** — headline metric; the only summary statistic that behaves sensibly under
  extreme imbalance
- **Recall @ fixed FPR** (0.1%, 1%) — the operational question: how much fraud is caught
  at a false-positive rate a bank's review team can actually absorb
- **Precision @ k** (k = 100, 500, 1000) — an analyst queue is finite; this is what they see
- **Cost-sensitive loss** — a missed APP fraud costs the victim the transferred amount; a
  false positive costs one review. Weight accordingly and state the assumed cost ratio.
- **Detection latency** — hours from the mule's first fraudulent credit to a flag.
  In APP fraud, money is cashed out fast; a detection at t+72h saves nothing. This metric
  is where mule-side detection either justifies itself or does not.
- **Per-silo breakdown** for every federated result

**Splitting is temporal, never random.** Train on steps `0–500`, validate `501–620`, test
`621–744`. A random split leaks future edges into the training graph and inflates every
number — a standard and fatal error in graph fraud papers, worth one explicit sentence in
the methodology.

**Statistical care.** 5 seeds, report mean ± std. At ~170 positives in a Phase-1-sized
sample, single-run differences are noise.

---

## 5. Primary survey (carried from Phase 1 §6.1)

30–50 respondents, qualitative validation of the engineered behavioural features against
real reported UPI fraud experiences. This is the only component touching *actual* UPI
fraud, and it covers the victim-side behaviour the proxy datasets cannot.

Ask what the victim observed: first contact channel, whether the payee was new, time
pressure applied, amount relative to normal spend, time of day. Map each answer to a
feature the model could compute. Where a reported signal has no computable counterpart,
say so — that is an honest limitation and a Phase 3 hook.

Check institutional ethics/consent requirements before collecting. No personally
identifying information, no account numbers, no transaction IDs.

---

## 6. Schedule

| Week | Work | Deliverable |
|---|---|---|
| 1 | Download PaySim; obtain MoMTSim; replace the 150k synthetic proxy with real data | Real data in `data/raw/`, Phase 1 numbers re-run |
| 2 | Graph construction, mule-account labelling, temporal split | `src/build_graph.py`, base-rate report |
| 3 | Tabular baselines + **leakage ablation** | Ablation table — the 1.0000 collapse |
| 4–5 | Heterophily-aware temporal GNN; heterophily + temporal ablations | Model code, ablation results |
| 6 | Federated setup over institutional silos; per-silo lift | Federation table |
| 7 | SHAP across all models; detection-latency analysis | Figures in `results/` |
| 8 | Survey (run in parallel from week 4); write-up | Phase 2 document |

---

## 7. Open items to verify before writing the final paper

1. **Read P3 and P4 in full.** Everything above about them is taken from Phase 1's own
   one-line summaries. Before the final write-up, read both papers and confirm: P3's exact
   federated architecture and privacy guarantee, P4's precise heterophily mechanism, and
   the datasets and metrics each actually reports. Put the PDFs in `references/`.
   **Do not cite a method detail this plan asserts without checking it against the paper.**
2. **Confirm MoMTSim availability.** The plan leans on it as the primary dataset. If it
   is not obtainable, PaySim carries Phase 2 and MoMTSim moves to Phase 3.
3. **Confirm the ethics process** for the survey with the guide.
4. **Fill the title-page placeholders** — `Phase1_UPI_Fraud_Detection.docx` still contains
   `[Your Roll Number]`, `[Your Section]` and `[Guide Name]`.
