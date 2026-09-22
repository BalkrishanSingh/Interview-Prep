# Module 05: GenAI & ML — LLMs, Prompt Engineering & Fine-Tuning

---

## 1. Tokenization & Autoregressive Generation

Large Language Models (LLMs) operate not on characters or raw words, but on subword **Tokens** generated via compression algorithms such as **Byte-Pair Encoding (BPE)** or **WordPiece**.

```
Input String: "Transformer architectures are scalable."
   │
   ▼ (Tokenizer)
Tokens: ["Transform", "er", " architectures", " are", " scal", "able", "."]
   │
   ▼ (Embedding Lookup + Transformer Decoder Layers)
Output Logits Vector: (Vocabulary Size ~ 32,000 - 128,000 dimensions)
   │
   ▼ (Softmax + Sampling Strategy)
Next Predicted Token: " They..."
```

---

## 2. Inference Hyperparameters Explained

Once an LLM computes raw unnormalized log-probabilities (**Logits** $z_i$) for each vocabulary token, sampling parameters govern token selection.

### 2.1 Temperature ($T$)
Modulates the sharpness of the probability distribution during the softmax calculation:

$$P(x_i) = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}$$

- **$T \to 0$ (Greedy Decoding / Argmax)**: Maximizes certainty. The model deterministically picks the highest probability token. Best for SQL generation, code compilation, and math.
- **$T = 0.7 - 0.9$**: Standard creative balance for natural dialogue and prose.
- **$T > 1.0$**: Flattens the distribution, increasing the likelihood of selecting rare tokens (risks incoherence and hallucinations).

### 2.2 Top-K vs. Top-P (Nucleus) Sampling

```
VOCABULARY SORTED BY PROBABILITY:
Token:      "the"   "a"   "this"  "one"  │  "blue"  "dog"  "zebra"
Prob:        0.40   0.25   0.15   0.10   │  0.05    0.03   0.02
Cumulative:  0.40   0.65   0.80   0.90   │  0.95    0.98   1.00
                                         │
Top-K (e.g., K = 4) ────────────────────┘ (Strictly limits pool to top 4 tokens)
Top-P (e.g., P = 0.90) ───────────────────┘ (Dynamically limits pool until cumulative sum >= 0.90)
```

| Parameter | Mechanism | Strength |
| :--- | :--- | :--- |
| **Top-K** | Keeps strictly the $K$ most probable tokens; zeroes out all others. | Simple, but fixed: can include junk tokens when confidence is very high, or cut off valid tokens when confidence is diffuse. |
| **Top-P (Nucleus)** | Keeps the dynamic smallest set of tokens whose cumulative probability exceeds threshold $P$. | **Adaptive**: Automatically expands candidate set when uncertainty is high and contracts to 1-2 tokens when certainty is high. |

---

## 3. Prompt Engineering Paradigms

```
┌────────────────────────────────────────────────────────────────────────┐
│                        PROMPTING PARADIGMS                             │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          ▼                         ▼                         ▼
┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
│     ZERO-SHOT     │     │     FEW-SHOT      │     │ CHAIN-OF-THOUGHT  │
├───────────────────┤     ├───────────────────┤     ├───────────────────┤
│ Direct instruction│     │ 2-5 In-context    │     │ Forces explicit   │
│ with zero prior   │     │ exemplars showing │     │ intermediate      │
│ examples          │     │ input -> output   │     │ reasoning steps   │
└───────────────────┘     └───────────────────┘     └───────────────────┘
```

### 3.1 Chain-of-Thought (CoT) Prompting
- **Mechanism**: Prompting the model with *"Let's think step by step"* or providing multi-step reasoning examples forces the model to allocate forward-pass token compute to intermediate deductions before generating the final answer.
- **Impact**: Dramatically improves accuracy on arithmetic, logic puzzles, and multi-hop reasoning.

### 3.2 The ReAct Framework (Reason + Act)
Interleaves thought processes with external tool invocations:
1. **Thought**: Model reasons about what information is missing.
2. **Action**: Model emits a structured tool call (e.g., `execute_sql("SELECT ...")` or `search_web(...)`).
3. **Observation**: Application runtime executes the tool and feeds output back into model context.
4. **Final Response**: Model synthesizes observations to answer the user.

---

## 4. The Model Customization Spectrum

```
LOW COST / HIGH AGILITY                                      HIGH COST / HIGH COMPUTE
─────────────────────────────────────────────────────────────────────────────────────►
Prompt Engineering  ──►  Retrieval-Augmented  ──►  Parameter-Efficient  ──► Full Fine-
 (In-Context)             Generation (RAG)           Fine-Tuning (LoRA)       Tuning
• Zero weight update     • Zero weight update      • Updates ~0.1% weights • Updates 100%
• Fast iteration         • Dynamic live data       • Adapts style/grammar   weights
• Context window bound   • Low hallucination       • High domain training  • Massive GPU cost
```

### 4.1 Parameter-Efficient Fine-Tuning: LoRA (Low-Rank Adaptation)
Full fine-tuning of a 70-billion parameter model requires updating and storing all 70B weights and optimizer states (demanding hundreds of gigabytes of VRAM).

**LoRA** (Hu et al., 2021) freezes the pre-trained weight matrix $W_0 \in \mathbb{R}^{d \times k}$ and decomposes the weight update $\Delta W$ into two low-rank matrices:

$$W = W_0 + \Delta W = W_0 + B \cdot A$$

$$\text{where } B \in \mathbb{R}^{d \times r}, \quad A \in \mathbb{R}^{r \times k}, \quad \text{with rank } r \ll \min(d, k)$$

```
Input Vector x
      │
      ├─────────────────────────────────────────┐
      ▼                                         ▼
[ Frozen Base Weight W_0 ]              [ Low-Rank Matrix A (k x r) ]
(d x k dimension - No gradients)                ▼
      │                                 [ Low-Rank Matrix B (r x d) ]
      │                                         │
      ▼                                         ▼
Output W_0 · x                       Output ΔW · x = B · A · x
      │                                         │
      └───────────────────┬─────────────────────┘
                          ▼
               Final Output = (W_0 + B · A) · x
```

- If $d = 4096$ and $r = 8$, full matrix $W$ contains $4096 \times 4096 \approx 16.7\text{ million}$ parameters.
- LoRA matrices $A$ and $B$ contain $(4096 \times 8) + (8 \times 4096) = 65,536$ parameters (**a 99.6% parameter reduction**).
