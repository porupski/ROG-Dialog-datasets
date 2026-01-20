# ROG-Dialog Sentiment Dataset

Slovenian conversational speech dataset with sentiment, dialogue dimension, and dialogue function annotations.

## Dataset Overview

**Source:** EXB (EXMARaLDA) files from ROG-Dialog corpus  
**Version:** v7  
**Tasks:** 3 (Sentiment, Dialogue Dimension, Dialogue Function)  
**Format:** JSON and TSV

## Tasks

### 1. Sentiment Analysis
- **Labels (original):** 6 classes
  - `predominantlyNegative`, `mixedNegative`,
  - `neutralNegative`, `neutralPositive`,
  - `mixedPositive`, `predominantlyPositive`
- **Labels (collapsed):** 3 classes
  - `negative`, `neutral`, `positive`
- **Text fields:** `norm` (normalized), `colloq` (colloquial)

### 2. Dialogue Acts - Dimension
- **Labels:** ISO dialogue act dimensions (e.g., `question`, `inform`, `commissive`)
- **Rare labels** (<50 instances): collapsed to `unspecifiedDimension`
- **Text fields:** `norm`, `colloq`

### 3. Dialogue Acts - Function
- **Labels:** ISO dialogue act functions (e.g., `propositionalQuestion`, `setQuestion`, `inform`)
- **Rare labels** (<50 instances): collapsed to `unspecifiedFunction`
- **Text fields:** `norm`, `colloq`

## Files

### Data Files
```
sentiment_v7.json          # All sentiment instances
dimension_v7.json          # All dimension instances  
function_v7.json           # All function instances
file_metadata_v7.json      # File-level metadata
speaker_metadata_v7.json   # Speaker demographics
```

### TSV Splits
For each task (sentiment/dimension/function):
```
{task}_split_orig_{train|dev|test}_v7.tsv    # Original splits from corpus
{task}_split_redo_{train|dev|test}_v7.tsv    # Rebalanced splits
```

## Data Structure

### JSON Fields
```json
{
  "instance_id": "ROG-Dia-GSO-P0005_ROG-dialog-0007_0.555_3.017",
  "file_id": "ROG-Dia-GSO-P0005",
  "speaker": "ROG-dialog-0007",
  "norm": "Kaj zdaj, si kupila karte za na Sardinijo?",
  "colloq": "Kaj zdej, si kupla karte za na Sardinijo?",
  "start_t": 0.555,
  "end_t": 3.017,
  "start_id": "T0-555",
  "end_id": "T3-017",
  "label_origC": "mixedNegative",           // Original 6-class label (sentiment only)
  "label_collapseC": "negative",            // Collapsed 3-class label (sentiment only)
  "label_orig": "question",                 // Original label (dimension/function)
  "label_collapse": "question",             // Collapsed label (dimension/function)
  "NER_orig": "B-mixedNegative",           // BIO tag for original labels
  "NER_collapse": "B-negative",            // BIO tag for collapsed labels
  "split_orig": "Train",                   // Original train/dev/test split
  "split_redo": "train",                   // Rebalanced split
  "annotation_type": "sentiment"
}
```

### Text Fields
- **`norm`**: Normalized Slovenian text (standard orthography)
- **`colloq`**: Colloquial transcription (as spoken)

Multiple `norm`/`colloq` intervals are combined when covered by a single annotation span.

### NER Tags (BIO Format)
- **`B-{label}`**: Beginning of label span
- **`I-{label}`**: Inside (continuation) of label span
- **`O`**: Outside any labeled span

Used for sequence labeling tasks within speaker turns.

## Splits

### `split_orig`
Original train/dev/test split from ROG-Dialog corpus.

### `split_redo` (Recommended)
Rebalanced split ensuring:
- No speaker leakage between splits
- Balanced label distribution across train/dev/test
- Gender balance (at least 1 male and 1 female speaker in dev/test)
- Better coverage of rare labels

**Ratios:** 70% train, 15% dev, 15% test

## Label Collapse Mapping

### Sentiment
```python
{
  'predominantlyNegative': 'negative',
  'mixedNegative': 'negative',
  'neutralNegative': 'neutral',
  'neutralPositive': 'neutral',
  'mixedPositive': 'positive',
  'predominantlyPositive': 'positive'
}
```

### Dimension/Function
Labels with <50 instances → `unspecifiedDimension` / `unspecifiedFunction`

## Chronological Ordering

All instances are sorted by:
1. `start_t` (timestamp)
2. `speaker` (for overlapping speech)

This preserves the actual conversation flow.

## Processing Logic:

- For each annotation interval, find all `norm`/`colloq` intervals within its timespan
- Combine those intervals into a single instance with that annotation's label
- Detect speaker turns based on temporal gaps (>0.5s)
- Generate BIO tags for NER-style sequence labeling

## Usage

### Load Dataset
```python
import json

with open('sentiment_v7.json') as f:
    data = json.load(f)

for instance in data:
    text = instance['norm']
    label = instance['label_collapseC']  # or label_origC
    ner_tag = instance['NER_collapse']   # for sequence labeling
```

### TSV Format
```
instance_id	file_id	speaker	colloq	norm	start_t	end_t	label_origC	label_collapseC	NER_orig	NER_collapse	split_orig	split_redo	annotation_type
```

## Citation

If you use this dataset, please cite:
```
[ROG-Dialog corpus citation]
```

## License

[Specify license]

## Notes

- **Removed tiers:** `sentimentAnnotated` (unreliable), `normSeg`, `colloqSeg` (misaligned)
- **Only using:** `sentimentCurated` for sentiment task
- **Annotation spans:** Can cover multiple `norm` intervals - these are automatically combined
- **Speaker turns:** Detected via 0.5s gap threshold for NER tag generation
