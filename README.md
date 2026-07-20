# SLR Screening Project

This project performs **automated title and abstract screening** for a Systematic Literature Review (SLR) using **Ollama** with the **Gemma3:4b** Large Language Model.

The model evaluates each study against predefined **Inclusion Criteria** and **Exclusion Criteria**, returning a structured JSON decision for each paper.

---

## Model

The screening process uses:

- **Framework:** Ollama
- **Model:** `gemma3:4b`
- **Inference:** Local execution (no cloud API required)

Example command to download the model:

```bash
ollama pull gemma3:4b
```

To start the Ollama server:

```bash
ollama serve
```

Verify the model is available:

```bash
ollama list
```

Expected output:

```
NAME
gemma3:4b
```

---

## Processing Strategy

To improve efficiency and reliability, the screening is performed in batches.

### Configuration

- **Model:** Gemma3:4b
- **Inference Engine:** Ollama
- **Batch size:** 5 papers
- **Records processed per run:** 1,000 papers

Example:

```
Dataset
│
├── Paper 1
├── Paper 2
├── ...
├── Paper 1000
│
└── Processed as:

Batch 1     → Papers 1–5
Batch 2     → Papers 6–10
Batch 3     → Papers 11–15
...
Batch 200   → Papers 996–1000
```

For every run:

- Total papers: **1,000**
- Batch size: **5**
- Total batches: **200**

---

## Workflow

1. Load the input CSV file.
2. Read the first 1,000 records.
3. Split the dataset into batches of five papers.
4. Send each batch to **Gemma3:4b** through Ollama.
5. Parse the returned JSON response.
6. Save the screening results immediately.
7. Continue until all batches are completed.

---

## Output

Each processed paper includes:

| Column | Description |
|---------|-------------|
| Title | Paper title |
| Abstract | Paper abstract |
| Decision | Include / Exclude |
| Confidence | Confidence score |
| Reason | Brief explanation |

---

## Error Handling

If a batch fails:

1. Retry up to the configured `MAX_RETRY`.
2. If all retries fail:
   - Mark all papers in the batch as `ERROR`.
   - Continue processing the remaining batches.

Example output:

```
Decision    : ERROR
Confidence  : 0
Reason      : Connection error
```

This ensures that a single failed batch does not stop the entire screening process.

---

## Progress Monitoring

The project uses **tqdm** to display real-time progress.

Example:

```
Processing batches:
████████████████████████████ 100%
200/200 batches completed
```

---

## Recommended Workflow for Large Datasets

For datasets larger than 1,000 records:

```
Run 1
Records 1–1000

↓

Run 2
Records 1001–2000

↓

Run 3
Records 2001–3000

↓

...
```

This approach:

- Reduces memory usage
- Prevents long-running sessions
- Simplifies recovery after interruptions
- Minimizes the risk of losing progress

---

## Project Structure

```
project/
│
├── input.csv
├── screening.py
├── config.py
├── prompt.py
├── output/
│   ├── screened_1_1000.csv
│   ├── screened_1001_2000.csv
│   └── ...
└── README.md
```

---

## Requirements

- Python 3.10+
- Ollama
- Gemma3:4b model
- pandas
- tqdm
- ollama Python package

Install Python dependencies:

```bash
pip install pandas tqdm ollama
```

---

## Notes

- The project uses **Ollama** to run the **Gemma3:4b** model locally, eliminating the need for external API services.
- A **batch size of 5 papers** provides a balance between processing efficiency and response reliability.
- Processing **1,000 papers per run** is recommended to reduce interruptions and facilitate recovery in environments such as Kaggle or Google Colab.
- Results are saved incrementally after each batch to minimize data loss in the event of an unexpected interruption.
