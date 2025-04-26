# Summary Evaluation Flowchart

Start: **Summary Evaluation Application**
|
|--> **1. Setup and Preparation**
|    |
|    |--> Import Required Libraries
|    |    - torch, numpy, os, json, hashlib
|    |    - nltk (BLEU, METEOR)
|    |    - sacrebleu (CHRF)
|    |    - sentence-transformers (SIDE embedding)
|    |    - transformers (HuggingFace models)
|    |
|    |--> Define Directories
|    |    - `summary_dir`: Folder for all summaries (`/content/gdrive/MyDrive/CodeClarity/summaries/`)
|    |    - `backtranslation_dir`: Folder for all cached translations and scores
|    |    - Create backtranslation directory if missing
|    |
|    |--> Load Language Code Mappings
|    |    - Map spoken language (e.g., "Spanish") ➔ NLLB language code (e.g., "spa_Latn")
|    |
|    |--> Load NLLB Backtranslation Model
|    |    - `facebook/nllb-200-distilled-600M`
|    |    - Load AutoTokenizer and AutoModelForSeq2SeqLM
|    |    - Move model to GPU (if available)
|
|--> **2. Caching Setup**
|    |
|    |--> Define Global Variables
|    |    - `embedding_model = None` (for SIDE metric)
|    |    - `bertscore_model = None`, `bertscore_tokenizer = None` (for BERTScore metric)
|    |
|    |--> Define `NumpyEncoder`
|    |    - Custom encoder to save numpy numbers into JSON files without crashing
|
|--> **3. Metrics Computation Functions**
|    |
|    |--> **compute_bertscore(ref_file, cand_file, language)**
|    |    |
|    |    |--> Detect file format (JSON or JSONL)
|    |    |--> Load references and candidates from files
|    |    |--> Load BERT model (`xlm-roberta-large`) once
|    |    |--> Tokenize, embed, and compute similarity
|    |    |--> Return precision, recall, and F1
|    |
|    |--> **compute_side_score(references, candidates)**
|    |    |
|    |    |--> Load SentenceTransformer model once (`paraphrase-multilingual-mpnet-base-v2`)
|    |    |--> Encode sentences into vectors
|    |    |--> Compute cosine similarity for each pair
|    |    |--> Return average SIDE score
|    |
|    |--> **compute_meteor_score(references, candidates)**
|    |    |
|    |    |--> Tokenize words using NLTK
|    |    |--> Compute METEOR score for each pair
|    |    |--> Return average METEOR score
|    |
|    |--> **compute_chrf_score(references, candidates)**
|    |    |
|    |    |--> Compute CHRF score using sacrebleu
|    |    |--> Return CHRF score
|
|--> **4. Backtranslation Functions**
|    |
|    |--> **backtranslate_with_nllb(text, source_lang, code_lang)**
|    |    |
|    |    |--> Find correct language code from mapping
|    |    |--> Check if translation is cached
|    |    |    - If yes ➔ Load from cache
|    |    |    - If no ➔ Perform translation:
|    |    |        - Tokenize input
|    |    |        - Generate translation with NLLB model (force English output)
|    |    |        - Cache the result to file
|    |    |--> Return translated text
|    |
|    |--> **get_cached_backtranslation(text, lang, code_lang)**
|    |    |
|    |    |--> Generate a SHA256 hash key for text
|    |    |--> Check if translation already cached
|    |    |--> If not, call `backtranslate_with_nllb`
|    |    |--> Save and return the backtranslated text
|
|--> **5. Complete Metrics Evaluation Function**
|    |
|    |--> **compute_all_metrics(references, candidates, lang, code_lang)**
|    |    |
|    |    |--> Backtranslate all candidate summaries to English
|    |    |--> Compute:
|    |    |    - BLEU (on backtranslated candidates)
|    |    |    - ROUGE-L (on backtranslated candidates)
|    |    |    - METEOR (on backtranslated candidates)
|    |    |    - CHRF (on backtranslated candidates)
|    |    |    - SIDE (on backtranslated candidates)
|    |    |--> Return a dictionary of all scores
|
|--> **6. Main Evaluation Runner**
|    |
|    |--> **run_evaluation()**
|    |    |
|    |    |--> Loop over programming languages:
|    |    |    - php, python, java, go, javascript, ruby
|    |    |
|    |    |--> For each programming language:
|    |    |    - Load English reference summaries
|    |    |
|    |    |--> Loop over spoken languages:
|    |    |    - Spanish, Mandarin Chinese, Arabic, Swahili, Yoruba, Tamil, Hindi, Portuguese, Filipino, French
|    |    |
|    |    |--> For each spoken language:
|    |    |    |
|    |    |    |--> Load candidate summaries in that language
|    |    |    |--> Compute BERTScore (direct, without backtranslation)
|    |    |    |--> Compute SIDE similarity (direct)
|    |    |    |--> Compute backtranslation-based metrics (BLEU, ROUGE, METEOR, CHRF, SIDE)
|    |    |    |--> Store all evaluation results
|    |
|    |--> Save results for each programming language (`scores_<code_lang>.json`)
|    |--> At the end, combine all results into one file (`all_scores.json`)
|
|--> **7. Output**
|    |
|    |--> All evaluation scores saved into:
|    |    - Per-language JSON files (e.g., `scores_python.json`)
|    |    - One combined master file (`all_scores.json`)
|
|--> End: **Evaluation complete!**
