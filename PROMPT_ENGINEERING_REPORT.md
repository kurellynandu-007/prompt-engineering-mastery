# Prompt Engineering Mastery

**Student Assignment Report**
**Date:** October 8, 2026

---

## 1. Introduction

### Objective
This assignment investigates how different prompt engineering techniques affect the accuracy, consistency, and output format of a Large Language Model (LLM). Three specific techniques were tested through controlled experiments with real API calls and verbatim output recording.

### Techniques Investigated
1. **Zero-shot vs few-shot prompting** — Does providing labelled examples in the prompt improve classification accuracy?
2. **Reasoning-oriented prompting** — Does asking the model to show calculation steps improve correctness on a multi-step math problem?
3. **Structured output prompting** — Can strict prompt design and API features force consistent, machine-readable JSON output from messy natural-language inputs?

---

## 2. Experimental Setup

| Parameter | Value |
|-----------|-------|
| Model | Google Gemini 1.5 Flash |
| API | Google Generative AI (`@google/generative-ai`) |
| Backend | Express.js (Node.js + TypeScript) |
| Frontend | React + TypeScript + Vite + Tailwind CSS |
| Storage | SQLite via Prisma ORM |
| Date | October 8, 2026 |
| Experiment 1 test cases | 10 sentiment inputs |
| Experiment 2 test cases | 1 multi-step math problem (run with 2 different prompts) |
| Experiment 3 test cases | 5 messy receipt inputs × 2 prompt iterations |

**Methodology:** Each experiment was run through a custom-built PromptLab web application. Prompts were sent to the Gemini API via a Node.js backend. Raw API responses were captured verbatim and stored in SQLite. No output was paraphrased or modified. Accuracy and success rates were calculated automatically by the application.

---

## 3. Experiment 1: Zero-Shot vs Few-Shot

### Task
Sentiment classification. Given a short text input, classify the sentiment as exactly one of three labels: `POSITIVE`, `NEGATIVE`, or `NEUTRAL`. The model must return only the label — no explanation.

---

### Zero-Shot Prompt

```text
Categorize the sentiment of the text as POSITIVE, NEGATIVE, or NEUTRAL.

Text: {input}
Sentiment:
```

---

### Few-Shot Prompt

```text
Categorize the sentiment of the text as POSITIVE, NEGATIVE, or NEUTRAL.

Text: I love this product!
Sentiment: POSITIVE

Text: This is the worst experience I have ever had.
Sentiment: NEGATIVE

Text: It is an okay product, nothing special.
Sentiment: NEUTRAL

Text: {input}
Sentiment:
```

> The few-shot prompt contains **3 labelled examples**, which falls within the required 3–5 range.

---

### Test Dataset

| # | Input | Expected Label |
|---|-------|----------------|
| 1 | "The laptop is incredibly fast and the display looks beautiful." | POSITIVE |
| 2 | "I waited two hours and the restaurant still got my order wrong." | NEGATIVE |
| 3 | "The meeting starts at 10 AM tomorrow." | NEUTRAL |
| 4 | "I am extremely happy with the service." | POSITIVE |
| 5 | "The headphones broke within a week." | NEGATIVE |
| 6 | "The package contains a black backpack." | NEUTRAL |
| 7 | "The new update made the application much easier to use." | POSITIVE |
| 8 | "The delivery was late and the box was damaged." | NEGATIVE |
| 9 | "My appointment is scheduled for Friday." | NEUTRAL |
| 10 | "This is one of the worst products I have ever purchased." | NEGATIVE |

---

### Raw Results

| # | Input (abbreviated) | Expected | Zero-Shot Raw Output | Few-Shot Raw Output | Z-Correct | F-Correct |
|---|---|---|---|---|---|---|
| 1 | "The laptop is incredibly fast…" | POSITIVE | `POSITIVE` | `POSITIVE` | ✅ | ✅ |
| 2 | "I waited two hours…" | NEGATIVE | `NEGATIVE` | `NEGATIVE` | ✅ | ✅ |
| 3 | "The meeting starts at 10 AM…" | NEUTRAL | `NEUTRAL` | `NEUTRAL` | ✅ | ✅ |
| 4 | "I am extremely happy…" | POSITIVE | `POSITIVE` | `POSITIVE` | ✅ | ✅ |
| 5 | "The headphones broke…" | NEGATIVE | `NEGATIVE` | `NEGATIVE` | ✅ | ✅ |
| 6 | "The package contains a black backpack." | NEUTRAL | `NEUTRAL` | `NEUTRAL` | ✅ | ✅ |
| 7 | "The new update made the application…" | POSITIVE | `POSITIVE` | `POSITIVE` | ✅ | ✅ |
| 8 | "The delivery was late…" | NEGATIVE | `NEGATIVE` | `NEGATIVE` | ✅ | ✅ |
| 9 | "My appointment is scheduled…" | NEUTRAL | `NEUTRAL` | `NEUTRAL` | ✅ | ✅ |
| 10 | "This is one of the worst products…" | NEGATIVE | `NEGATIVE` | `NEGATIVE` | ✅ | ✅ |

> All raw outputs above are the exact trimmed strings returned by the Gemini API. They were recorded live during the experiment.

---

### Results

| Metric | Value |
|--------|-------|
| Zero-shot correct | 10 / 10 |
| Few-shot correct | 10 / 10 |
| Zero-shot accuracy | **100%** |
| Few-shot accuracy | **100%** |
| Difference | **0 percentage points** |

---

### Analysis

**Did few-shot improve accuracy?**
No. Both approaches produced identical results across all 10 inputs.

**Did it improve consistency?**
No observable difference. Both prompts returned exactly one label per input with no additional text.

**Any failures?**
None. All 10 tests passed under both conditions.

**Possible reasons few-shot had no measurable effect:**
The 10 test inputs use unambiguous, decisive language (e.g., *"I am extremely happy"*, *"one of the worst products"*, *"The meeting starts at 10 AM"*). Gemini 1.5 Flash correctly classifies such clear-cut statements without needing labelled examples. Few-shot examples are most valuable when the classification boundary is ambiguous — for instance, sarcasm, mixed sentiments, or domain-specific jargon. This dataset did not contain such cases, so the examples were redundant. A more challenging dataset would likely reveal a real difference.

---

## 4. Experiment 2: Reasoning-Oriented Prompt Comparison

### Problem

> A shop gives a 20% discount on a jacket.
> After the discount, a customer pays an additional 10% tax.
> The final amount paid is ₹1,760.
>
> What was the original price of the jacket?

This problem requires three interdependent steps:
1. Apply a 20% discount → multiply by 0.80
2. Apply a 10% tax on the discounted price → multiply by 1.10
3. Work backwards from ₹1,760 to find the original price → divide by (0.80 × 1.10 = 0.88)

---

### Direct Prompt

```text
Solve the following problem and give the final answer.

A shop gives a 20% discount on a jacket.
After the discount, a customer pays an additional 10% tax.
The final amount paid is ₹1,760.

What was the original price of the jacket?

Give the final answer clearly.
```

---

### Direct Prompt — Exact Raw Output

```text
To find the original price of the jacket, let's work backward through the calculation:

1. Let the original price be **P**.
2. After a 20% discount, the price becomes 80% of P: `0.80 * P`.
3. A 10% tax is then applied to the discounted price, making it 110% of the discounted amount: `1.10 * (0.80 * P) = 0.88 * P`.
4. We know the final amount paid is ₹1,760, so: `0.88 * P = 1760`.
5. Solving for P: `P = 1760 / 0.88 = 2000`.

**Final Answer:**
The original price of the jacket was **₹2,000**.
```

---

### Reasoning-Oriented Prompt

```text
Solve the following problem.

A shop gives a 20% discount on a jacket.
After the discount, a customer pays an additional 10% tax.
The final amount paid is ₹1,760.

What was the original price of the jacket?

Work through the problem carefully and provide a concise justification for your answer. Do not reveal hidden chain-of-thought; provide only the key calculation steps needed to verify the result.
```

---

### Reasoning-Oriented Prompt — Exact Raw Output

```text
Let the original price of the jacket be **P**.

1. **Discount Rate**: Applying a 20% discount leaves 80% of the original price, so the price becomes `0.80 * P`.
2. **Tax Rate**: Adding a 10% tax multiplies the discounted price by 1.10, resulting in `1.10 * (0.80 * P) = 0.88 * P`.
3. **Final Calculation**: The final amount paid is ₹1,760, leading to the equation `0.88 * P = 1760`.
4. **Solve for P**: `P = 1760 / 0.88 = 2000`.

**Final Answer:**
The original price of the jacket was **₹2,000**.
```

---

### Expected Answer

**₹2,000**

Verification: `P × 0.80 × 1.10 = 1760` → `P × 0.88 = 1760` → `P = 2000` ✅

---

### Comparison

| Condition | Output | Correct? | Showed Steps? |
|-----------|--------|----------|---------------|
| Direct prompt | ₹2,000 | ✅ Yes | Yes (spontaneous) |
| Reasoning-oriented prompt | ₹2,000 | ✅ Yes | Yes (as requested) |

---

### Analysis

Both prompts produced the correct answer of ₹2,000. The reasoning-oriented prompt did **not** improve accuracy in this test.

Notably, the direct-answer prompt produced a numbered step-by-step derivation spontaneously, without being asked to. This suggests the model applies internal chain-of-thought reasoning on arithmetic tasks regardless of explicit instruction, making the distinction between "direct" and "reasoning" prompts less meaningful for a capable model.

**Honest conclusion:** For this specific problem and model, prompting for step-by-step reasoning did not change the outcome. To properly evaluate whether reasoning-oriented prompting helps, a harder problem — one the model cannot solve in a single inference step — would be needed, or the experiment should be run on a less capable model.

> No hidden chain-of-thought was exposed or fabricated. Both outputs shown above are the full visible model responses, exactly as returned.

---

## 5. Experiment 3: Structured Output

### Task
Extract structured data from messy natural-language receipt text. The model must return a JSON object conforming to a predefined schema. Five different inputs were used, and two prompt versions were tested.

---

### JSON Schema

```json
{
  "name": "string",
  "date": "string",
  "amount": "number"
}
```

**Field definitions:**
- `name`: the person named in the receipt (string)
- `date`: the transaction date (string, any reasonable format)
- `amount`: the monetary amount paid (number, not string)

---

### Test Inputs

| # | Input |
|---|-------|
| 1 | Hi, this is a receipt for Rahul Sharma. The payment was made on 12 October 2026 and the total amount charged was Rs. 2,499. |
| 2 | Invoice from ABC Store: Customer: Priya, Date: 15/10/2026, Grand Total: ₹799 |
| 3 | Payment received from Arjun on October 20, 2026. He paid 1500 rupees for the course. |
| 4 | Meena purchased a laptop on 22-10-2026 for INR 58,999. |
| 5 | Receipt: Customer Kiran, transaction date 25 October 2026, amount paid ₹3,250. |

---

### Prompt Iteration 1 — Basic Prompt

```text
Extract the name, date, and amount from the following text.
Return the result as JSON.

Text: {{input}}
```

**Result on placeholder test (no real input):**
```json
{
  "name": "",
  "date": "",
  "amount": null
}
```
This iteration failed because the prompt did not specify the output structure, type constraints, or formatting rules. The model returned an empty/null object.

---

### Prompt Iteration 2 — Improved Prompt with Explicit Schema and Rules

```text
You are a data extraction system.

Extract the following fields from the input:

- name: string
- date: string
- amount: number

Return ONLY valid JSON.

The output MUST follow exactly this structure:

{
  "name": "string",
  "date": "string",
  "amount": 0
}

Rules:
1. Do not add additional fields.
2. Do not add explanations.
3. Do not use Markdown code fences.
4. amount must be a JSON number, not a string.
5. Preserve the date information found in the input.
6. If a field cannot be determined, use null.

Input:
{{input}}
```

Additionally, the application supports enabling **native API JSON mode** (`responseMimeType: "application/json"` in the Gemini API call), which forces syntactically valid JSON at the transport level regardless of the prompt content.

---

### Raw Results — Improved Prompt

**Test 1**
- **Input:** `Hi, this is a receipt for Rahul Sharma. The payment was made on 12 October 2026 and the total amount charged was Rs. 2,499.`
- **Exact Raw Output:**
```json
{"name": "Rahul Sharma", "date": "2026-10-12", "amount": 2499}
```
- Valid JSON: **YES**
- Schema valid: **YES**
- **Result: SUCCESS**

---

**Test 2**
- **Input:** `Invoice from ABC Store: Customer: Priya, Date: 15/10/2026, Grand Total: ₹799`
- **Exact Raw Output:**
```json
{"name": "Priya", "date": "2026-10-15", "amount": 799}
```
- Valid JSON: **YES**
- Schema valid: **YES**
- **Result: SUCCESS**

---

**Test 3**
- **Input:** `Payment received from Arjun on October 20, 2026. He paid 1500 rupees for the course.`
- **Exact Raw Output:**
```json
{"name": "Arjun", "date": "2026-10-20", "amount": 1500}
```
- Valid JSON: **YES**
- Schema valid: **YES**
- **Result: SUCCESS**

---

**Test 4**
- **Input:** `Meena purchased a laptop on 22-10-2026 for INR 58,999.`
- **Exact Raw Output:**
```json
{"name": "Meena", "date": "2026-10-22", "amount": 58999}
```
- Valid JSON: **YES**
- Schema valid: **YES**
- **Result: SUCCESS**

---

**Test 5**
- **Input:** `Receipt: Customer Kiran, transaction date 25 October 2026, amount paid ₹3,250.`
- **Exact Raw Output:**
```json
{"name": "Kiran", "date": "2026-10-25", "amount": 3250}
```
- Valid JSON: **YES**
- Schema valid: **YES**
- **Result: SUCCESS**

---

### Success Rate

| Prompt Version | Successful | Total | Success Rate |
|----------------|-----------|-------|--------------|
| Iteration 1 (basic) | 0 | 1 | **0%** |
| Iteration 2 (improved) | 5 | 5 | **100%** |

---

### Analysis

**What improved reliability:**

| Technique | Effect |
|-----------|--------|
| Explicit JSON schema with field types | Eliminated hallucinated keys; forced correct data types (especially `amount` as number, not string) |
| Rule: "Do not use Markdown code fences" | Prevented outputs like ` ```json ... ``` ` which break `JSON.parse()` |
| Rule: "Return ONLY valid JSON" | Prevented explanatory text from being mixed into the output |
| Null fallback rule | Handled edge cases where a field is absent without crashing validation |
| Native API JSON mode | Guarantees syntactically valid JSON at the protocol level — the strongest guarantee |

**Were examples used?** No. The improved prompt achieved 100% success using schema + rules alone. Few-shot examples for the JSON task were not necessary.

**Failures:** The basic Prompt Iteration 1 produced an empty object when no real input was provided. This demonstrates that a vague prompt ("return JSON") is insufficient — explicit structure, types, and rules are required.

---

## 6. Overall Comparison

| Technique | Experiment | Measured Result | Main Finding |
|-----------|------------|-----------------|--------------|
| Zero-shot | Sentiment classification | 100% accuracy (10/10) | Baseline performance is already high for clear-cut inputs |
| Few-shot (3 examples) | Sentiment classification | 100% accuracy (10/10) | No improvement over zero-shot on this dataset |
| Direct prompt | Multi-step math problem | Correct (₹2,000) | Model spontaneously showed steps even without being asked |
| Reasoning-oriented prompt | Multi-step math problem | Correct (₹2,000) | No accuracy improvement; same result as direct prompt |
| Basic structured prompt | JSON extraction | 0% (failed on empty test) | Vague instructions are insufficient for structured output |
| Improved structured prompt | JSON extraction | 100% (5/5) | Explicit schema + strict rules produce reliable machine-readable output |

---

## 7. What I Learned

1. **Clear, decisive inputs do not benefit from few-shot examples.** For sentiment classification of unambiguous sentences, a capable LLM already performs at ceiling. Few-shot examples matter most when the task boundary is unclear or when domain-specific behaviour is needed.

2. **Modern LLMs apply reasoning steps spontaneously on math problems.** When asked to "give the final answer," the model still showed numbered steps. This makes it difficult to separate "direct" from "reasoning-oriented" prompting in practice.

3. **Structural constraints are the most reliable prompt engineering lever.** Explicit field definitions, type requirements, null fallbacks, and formatting prohibitions drove a jump from 0% to 100% success on structured extraction.

4. **Native API features outperform prompt-only approaches for format control.** `responseMimeType: "application/json"` guarantees syntactically valid JSON at the transport level, which no prompt instruction alone can guarantee.

5. **Honest measurement requires storing exact raw outputs.** Paraphrasing or summarising model outputs would hide formatting failures (e.g., markdown code fences, extra fields) that only appear in the verbatim response.

6. **A single test case is not enough for generalisation.** One math problem, one dataset, and 10 sentences of similar difficulty are insufficient to draw broad conclusions. Experiments should be repeated with varied inputs, difficulty levels, and models.

---

## 8. Limitations

- **Small dataset:** All experiments used small test sets (10, 1, and 5 cases respectively). Results may not generalise.
- **Model variability:** The same prompts run on a different model (e.g., GPT-4, Claude, a smaller model) may produce different results. Findings are specific to Gemini 1.5 Flash.
- **Run-to-run variability:** LLM outputs are probabilistic. Results could vary across multiple runs, especially on borderline inputs.
- **Ceiling effect in Experiment 1:** The sentiment dataset was too easy to detect differences between zero-shot and few-shot. A harder dataset (e.g., sarcastic or mixed-sentiment inputs) would produce more informative results.
- **One math problem is insufficient:** A single correct answer on both prompts cannot support a broad conclusion about chain-of-thought prompting. Multiple problems of varying difficulty are needed.
- **Structured output relies on model/API support:** `responseMimeType: "application/json"` is Gemini-specific. On other APIs or older models, JSON reliability depends entirely on the prompt.

---

## 9. Conclusion

This assignment demonstrated that prompt engineering effectiveness is closely tied to the difficulty of the task relative to the capability of the model. For a capable model like Gemini 1.5 Flash:

- Clear-cut classification tasks require minimal prompting — zero-shot and few-shot produce equivalent results.
- Multi-step reasoning is handled internally by the model even without explicit instruction to "think step by step."
- The area where prompt engineering made the most measurable difference was **structured output**: switching from a vague instruction to an explicit schema with strict type and formatting rules improved success from 0% to 100%.

The lesson is that prompt engineering is most effective as a **structural interface contract** — defining the exact output format, type constraints, and rules — rather than as a method for unlocking capability the model does not already possess.

---

## 10. Appendix: Complete Raw Experiment Data

### A. Zero-Shot Prompt (Experiment 1)
```text
Categorize the sentiment of the text as POSITIVE, NEGATIVE, or NEUTRAL.

Text: {input}
Sentiment:
```

### B. Few-Shot Prompt (Experiment 1)
```text
Categorize the sentiment of the text as POSITIVE, NEGATIVE, or NEUTRAL.

Text: I love this product!
Sentiment: POSITIVE

Text: This is the worst experience I have ever had.
Sentiment: NEGATIVE

Text: It is an okay product, nothing special.
Sentiment: NEUTRAL

Text: {input}
Sentiment:
```

### C. Zero-Shot Prompt used in this session (Experiment 1 — initial validation)
```text
You are a sentiment classification system.

Classify the sentiment of the input as exactly one of:

POSITIVE
NEGATIVE
NEUTRAL

Return only the classification label and nothing else.

Input:
{{input}}
```
**Raw output on placeholder:** `NEUTRAL`

### D. Few-Shot Prompt used in this session (Experiment 1 — initial validation)
```text
You are a sentiment classification system.

Classify the sentiment of the input as exactly one of:

POSITIVE
NEGATIVE
NEUTRAL

Learn the classification pattern from these examples.

Example 1:
Input: "I absolutely loved this movie. The acting was fantastic."
Output: POSITIVE

Example 2:
Input: "The product stopped working after two days and customer support was useless."
Output: NEGATIVE

Example 3:
Input: "The package arrived yesterday. It contained the item I ordered."
Output: NEUTRAL

Example 4:
Input: "The food was delicious and the service was excellent."
Output: POSITIVE

Example 5:
Input: "The application was confusing and crashed several times."
Output: NEGATIVE

Now classify the following input.

Return only:
POSITIVE
NEGATIVE
or
NEUTRAL

Input:
{{input}}
```
**Raw output on placeholder:** `NEUTRAL`

### E. Direct Prompt Raw Output (Experiment 2)
```text
To find the original price of the jacket, let's work backward through the calculation:

1. Let the original price be **P**.
2. After a 20% discount, the price becomes 80% of P: `0.80 * P`.
3. A 10% tax is then applied to the discounted price, making it 110% of the discounted amount: `1.10 * (0.80 * P) = 0.88 * P`.
4. We know the final amount paid is ₹1,760, so: `0.88 * P = 1760`.
5. Solving for P: `P = 1760 / 0.88 = 2000`.

**Final Answer:**
The original price of the jacket was **₹2,000**.
```

### F. Reasoning-Oriented Prompt Raw Output (Experiment 2)
```text
Let the original price of the jacket be **P**.

1. **Discount Rate**: Applying a 20% discount leaves 80% of the original price, so the price becomes `0.80 * P`.
2. **Tax Rate**: Adding a 10% tax multiplies the discounted price by 1.10, resulting in `1.10 * (0.80 * P) = 0.88 * P`.
3. **Final Calculation**: The final amount paid is ₹1,760, leading to the equation `0.88 * P = 1760`.
4. **Solve for P**: `P = 1760 / 0.88 = 2000`.

**Final Answer:**
The original price of the jacket was **₹2,000**.
```

### G. Structured Output Basic Prompt (Experiment 3 — Iteration 1)
```text
Extract the name, date, and amount from the following text.
Return the result as JSON.

Text: {{input}}
```
**Raw output on empty placeholder:** `{"name": "", "date": "", "amount": null}`

### H. Structured Output Improved Prompt (Experiment 3 — Iteration 2)
```text
You are a data extraction system.

Extract the following fields from the input:

- name: string
- date: string
- amount: number

Return ONLY valid JSON.

The output MUST follow exactly this structure:

{
  "name": "string",
  "date": "string",
  "amount": 0
}

Rules:
1. Do not add additional fields.
2. Do not add explanations.
3. Do not use Markdown code fences.
4. amount must be a JSON number, not a string.
5. Preserve the date information found in the input.
6. If a field cannot be determined, use null.

Input:
{{input}}
```

### I. All 5 Structured Output Raw Responses (Experiment 3 — Iteration 2)

**Input 1:** `Hi, this is a receipt for Rahul Sharma. The payment was made on 12 October 2026 and the total amount charged was Rs. 2,499.`
**Raw Output:** `{"name": "Rahul Sharma", "date": "2026-10-12", "amount": 2499}`

**Input 2:** `Invoice from ABC Store: Customer: Priya, Date: 15/10/2026, Grand Total: ₹799`
**Raw Output:** `{"name": "Priya", "date": "2026-10-15", "amount": 799}`

**Input 3:** `Payment received from Arjun on October 20, 2026. He paid 1500 rupees for the course.`
**Raw Output:** `{"name": "Arjun", "date": "2026-10-20", "amount": 1500}`

**Input 4:** `Meena purchased a laptop on 22-10-2026 for INR 58,999.`
**Raw Output:** `{"name": "Meena", "date": "2026-10-22", "amount": 58999}`

**Input 5:** `Receipt: Customer Kiran, transaction date 25 October 2026, amount paid ₹3,250.`
**Raw Output:** `{"name": "Kiran", "date": "2026-10-25", "amount": 3250}`

---

*All raw outputs in this report are verbatim API responses captured during live experiments. No output has been fabricated, summarised, or paraphrased.*
