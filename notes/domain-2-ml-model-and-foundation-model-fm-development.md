# Domain 2: ML Model and Foundation Model (FM) Development (24%)

This domain covers choosing the approach (managed AI service, traditional ML, foundation model, RAG or a custom model), training and customizing it, and evaluating the result. Questions describe a business problem and its data, then hinge on one constraint in the middle of the story: no labels, a regulator's explanation rule, exact identifiers in queries, a new account, labels that arrive weeks late. The modern-sounding option, such as a foundation model for tabular prediction, is often the wrong one.

## 2.1 Choose appropriate modeling approaches for ML and AI solutions

- **Rule:** use the least custom approach that meets the requirement. Each step down adds control and effort.

| Approach                                | Fits when                                                                  | Cost of choosing it                      |
| --------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| AWS AI service                          | A standard task the service is pre-trained for; no labels or ML staff      | Least effort, limited control            |
| Pre-trained model (SageMaker JumpStart) | A known architecture you fine-tune or host yourself, including edge export | You run training and hosting             |
| Foundation model (Amazon Bedrock)       | Generation, summarization, extraction from free text, chat                 | Per-token cost on every request          |
| Traditional ML                          | Labeled tabular data, fixed classes, forecasts, millions of predictions    | Needs data and training; cheap inference |
| Custom architecture                     | Nothing above meets accuracy, interpretability or architecture needs       | Most effort and ownership                |

- **Traps:**
  - A recurring score per customer from labeled columns is a tabular problem (XGBoost), not an FM prompt per row.
  - High-volume classification into fixed categories with a large labeled set → a small supervised classifier (BlazingText supervised, or Canvas custom text classification). FM cost repeats on every item.
  - k-means finds groups but cannot map them to your business categories.
  - If an AI service already returns exactly what is asked (Comprehend's four sentiment classes), training anything is extra effort.
- **Hosting is part of build vs buy:**

| Option                                         | Fits                                                                                          | Rules it out                               |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Bedrock Custom Model Import                    | Fine-tuned weights of a supported architecture (Hugging Face format in S3), on-demand billing | Modified architectures; no extra libraries |
| SageMaker AI real-time endpoint                | Custom containers, GPUs, steady traffic                                                       | Pays while idle                            |
| SageMaker AI Serverless Inference              | Small CPU models with intermittent traffic, scales to zero                                    | No GPUs, limited memory                    |
| Bedrock Provisioned Throughput (catalog model) | Committed capacity for that base model                                                        | Does not include your custom weights       |

- **Algorithm by problem:** labeled tabular → XGBoost. Explained with fixed weights → Linear Learner. Unlabeled anomaly scores → Random Cut Forest. Many related time series, including new items → DeepAR (probabilistic quantiles). Grouping → k-means. Images, audio, free text at scale → deep learning.
- **Interpretability is a requirement, not a preference:**
  - Linear coefficients: global, fixed, same for everyone. The only answer to "one published weight per feature".
  - SHAP values: per-prediction, vary by person.
  - Tree feature importance: ranking only, no direction or size.
  - XGBoost monotonic constraints: fix direction, not size.
- **FM selection in two passes:** hard filters first (input modality, **context window**, languages, features such as tool use or streaming, Region availability), then compare survivors on your own prompts for quality, latency and token cost. Max output tokens is a separate limit from the context window.
- **Embedding model choice:**

| Model                          | Use for                                      | Watch out for                                     |
| ------------------------------ | -------------------------------------------- | ------------------------------------------------- |
| Titan Text Embeddings V2       | Text retrieval; 256, 512 or 1,024 dimensions | Optimized for English; cross-language sub-optimal |
| Titan Multimodal Embeddings G1 | Image-to-image and text-to-image search      | Only helps if images are embedded; English text   |
| Cohere Embed English v3        | English retrieval                            | English only                                      |
| Cohere Embed Multilingual v3   | Multilingual and cross-language retrieval    | 512 tokens per text input                         |

- Visual search: embed the **images** with a multimodal model. Captions or labels discard visual features. Fewer dimensions (256 vs 1,024 is 75% less per vector) cut storage; any change means re-embedding the corpus.
- **RAG vs fine-tuning:** RAG for knowledge (changing facts, citations, per-user document restrictions). Fine-tuning for behaviour (tone, fixed layout, a proprietary label set). Many systems need both. Pasting context into a static prompt is not retrieval.
- **Customization by the data you have:**

| Method                    | Needs                                               | Changes                                  |
| ------------------------- | --------------------------------------------------- | ---------------------------------------- |
| Supervised fine-tuning    | Labeled prompt-response pairs                       | Format, tone, labeling conventions       |
| Reinforcement fine-tuning | Prompts plus a reward function (for example Lambda) | Outputs that can be graded automatically |
| Distillation              | Prompts and a good teacher model                    | Smaller, cheaper student                 |
| Continued pre-training    | Unlabeled domain text                               | Vocabulary and language                  |

- Distillation cannot exceed its teacher. Capacity options change throughput, not learning. Try prompt engineering first.
- **LoRA vs full fine-tuning:** LoRA freezes the base and trains small adapters (less GPU memory, small artifact). SageMaker AI can serve many LoRA adapters as **inference components** on one endpoint sharing one base model copy. Merging each variant back into a full model loses that saving.
- **RAG pattern by failing question type:**

| Symptom                                                | Pattern                                                                            |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| Exact part numbers or codes return similar wrong items | **Hybrid search** (`overrideSearchType` HYBRID)                                    |
| Facts linked across several documents                  | **GraphRAG** with Neptune Analytics                                                |
| Exact totals over warehouse tables                     | Knowledge base on a **structured data store** (Redshift), question turned into SQL |
| Users may see only some documents                      | **Metadata filtering**                                                             |
| One question bundles several                           | Query decomposition                                                                |

- Hybrid search in Bedrock Knowledge Bases works only on Aurora PostgreSQL (RDS), OpenSearch Serverless and MongoDB stores with a filterable text field. A reranker only reorders what was retrieved; more results do not fix wrong ranking.
- **Vector store behind a knowledge base:** mandatory features first (hybrid, graph, SQL joins), cost second. OpenSearch Serverless is the quick-create default with hybrid search but capacity-based billing; S3 Vectors is cheapest for huge, rarely queried, semantic-only collections; pgvector joins vectors with existing Aurora data; Neptune Analytics for GraphRAG.
- **Training trade-offs:**
  - Transfer learning beats training from scratch on modest labeled sets.
  - **Managed Spot training**: max wait time longer than max run time, plus **checkpoints to S3**. Spot without checkpoints risks losing progress.
  - **Warm pools** cut start-up latency but you pay for retained instances.
  - Rekognition Custom Labels models are served only by Rekognition, so they cannot be exported to the edge.
- **FM cost and latency levers:**

| Lever                      | Solves                                         | Does not fit when                 |
| -------------------------- | ---------------------------------------------- | --------------------------------- |
| Smaller model              | Cost on easy tasks                             | Hard requests must keep quality   |
| Intelligent prompt routing | Mixed easy and hard traffic, same model family | Prompts are mainly non-English    |
| Distillation               | Cheap student for specific intents             | No good teacher                   |
| Prompt caching             | Long identical prefix in every request         | No repeated prefix                |
| Batch inference            | Bulk non-interactive work, discounted          | Users wait for answers            |
| Provisioned Throughput     | Steady high load throttled on demand           | Sporadic traffic                  |
| Cross-Region inference     | Throttling at peaks                            | Inference must stay in one Region |

- Retrieved context counts as input tokens just like typed text.
- **AWS AI services:**
  - **Textract**: DetectDocumentText (raw text), AnalyzeDocument with FORMS / TABLES / QUERIES, **AnalyzeExpense** (invoices and receipts in any layout, normalized fields and line items), **AnalyzeID** (identity documents).
  - **Rekognition**: DetectModerationLabels for unsafe content (Custom Moderation adapters tune it); DetectLabels for general objects; Custom Labels for business-specific objects.
  - **Transcribe**: speech to text, PII redaction, custom vocabulary.
  - **Comprehend**: text only. Sentiment, entities, PII, key phrases. **Custom entity recognizer** extracts mentions; **custom classifier** labels whole documents.
  - Amazon Forecast is closed to new customers; use SageMaker AI (DeepAR).

📖 Full lesson: [Choosing ML, Foundation Model and RAG Approaches on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-modeling-approaches-fm-selection-rag/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 2.2 Train, fine-tune, and customize models for ML and AI solutions

- **Built-in algorithms** run in managed containers, emit metrics automatically and need no training code.

| Problem                              | Algorithm                                         |
| ------------------------------------ | ------------------------------------------------- |
| Tabular classification or regression | XGBoost (also Linear Learner, LightGBM, CatBoost) |
| Unlabeled anomaly scores             | Random Cut Forest                                 |
| Unusual entity-to-IP pairs           | IP Insights                                       |
| Many related time series             | DeepAR                                            |
| Grouping                             | k-means                                           |
| Sparse user-item recommendations     | Factorization Machines                            |
| Text classification or word vectors  | BlazingText                                       |
| Prediction from closest examples     | k-NN                                              |

- Read two things: labeled or not, and what the output must be (class, number, score, cluster, forecast).
- **Input:** CSV for built-ins means target in the first column, no header, `text/csv`. Declaring a different content type does not convert the file. FastFile removes the up-front copy; ShardedByS3Key stops every instance getting every file. Do not shard a script that already selects its own slice.
- **Script mode:** a framework estimator (PyTorch, TensorFlow, Hugging Face, scikit-learn, XGBoost) with `entry_point` and `framework_version`, in an AWS-managed container. SDK v3 replaces estimators with `ModelTrainer` plus `SourceCode`; container conventions stay the same.
  - Hyperparameters arrive as **command-line arguments**; parse with argparse. Hard-coded values defeat tuning.
  - Channels at `/opt/ml/input/data/<channel>` (`SM_CHANNEL_<NAME>`).
  - Save the model to **`SM_MODEL_DIR` (`/opt/ml/model`)**. Anything saved elsewhere is lost and the job still says Completed.
  - `/opt/ml/checkpoints` is for resuming, not the final model.
  - Extra packages: **`requirements.txt` in `source_dir`**. BYOC with the generic Estimator only when no managed container fits.
- **Automatic model tuning strategies:**

| Strategy           | How it picks                            | Use when                                    |
| ------------------ | --------------------------------------- | ------------------------------------------- |
| Bayesian (default) | Learns from completed jobs              | Limited budget, results should guide search |
| Random             | Independent samples                     | Many jobs fully in parallel                 |
| Grid               | Every combination; categorical only     | Small discrete spaces                       |
| Hyperband          | Stops weak jobs using per-epoch metrics | Long iterative training                     |

- High `MaxParallelTrainingJobs` turns Bayesian into random search. With a fixed total, lower parallelism usually finds better results.
- **Scaling type:** Logarithmic for values spanning orders of magnitude (learning rate, regularization). ReverseLogarithmic for values near 1 (momentum). Linear over 0.0001–1 barely explores small values.
- **Objective metric** for script mode: print it consistently and add a **metric definition regex**; name it with Maximize or Minimize.
- **Warm start:** IdenticalDataAndAlgorithm (same data and image, ranges may change) vs **TransferLearning** (new data, changed ranges, new algorithm version).
- **Early stopping:** `TrainingJobEarlyStoppingType=Auto` stops a job worse than the median of earlier jobs at the same epoch; custom scripts must log per-epoch metrics. Must be **Off** with Hyperband. XGBoost `early_stopping_rounds` needs a validation channel.
- **Distributed training:**
  - Model fits on one GPU, data is large → **data parallelism** (SMDDP).
  - Model state does not fit even at batch size 1 → **model parallelism / sharded data parallelism** (SMP v2 on PyTorch FSDP, tensor parallelism).
  - Data parallelism does not fix model-state OOM. A bigger instance is scale-up. Spot lowers cost, not time.
- **Fit diagnosis:**

| Problem                 | Symptom                                  | Fix                                                                                             |
| ----------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Overfitting             | Training great, validation clearly worse | Regularization (alpha/lambda), subsampling, shallower trees, dropout, early stopping, more data |
| Underfitting            | Both poor and close together             | More capacity, less regularization, better features                                             |
| Catastrophic forgetting | Fine-tuned model lost general abilities  | LoRA, mix in general data, lower learning rate, fewer epochs                                    |

- **Epochs, steps, batch:** steps per epoch = records ÷ batch size; total steps = steps per epoch × epochs (120,000 / 400 × 10 = 3,000). **Gradient accumulation** keeps a large effective batch on small memory. In data parallelism the global batch grows, so a common heuristic scales the learning rate with it and adds warmup.
- **Ensembles:** bagging reduces variance, boosting reduces bias, stacking learns how to weight diverse models, fixed voting cannot learn. **Cascades** send everything to a cheap, well-calibrated model and escalate low-confidence cases. **Bedrock intelligent prompt routing** is the managed version within one model family.
- **FM customization:** prompt engineering first (system prompt, explicit output format or schema, few-shot examples). Sampling parameters cannot create a structure the prompt never asked for. Then fine-tuning (labeled pairs), continued pre-training (unlabeled text), distillation (prompts plus teacher), Custom Model Import (hosts weights trained elsewhere, trains nothing).
  - A custom Bedrock model needs on-demand deployment where supported, otherwise Provisioned Throughput.
  - **QLoRA** loads the base in 4-bit (NF4) and trains adapters, so the job fits a smaller GPU.
- **Retrieval tuning:**

| Symptom                               | Change                                                                  |
| ------------------------------------- | ----------------------------------------------------------------------- |
| Exact identifiers return wrong chunks | Hybrid search                                                           |
| Right chunk, missing context          | Hierarchical chunking                                                   |
| Sentences split at boundaries         | Fixed-size with overlap (new data source, re-ingest)                    |
| Chunks mix topics                     | Semantic chunking                                                       |
| Storage and latency too high          | Smaller embedding dimension, re-embed, rebuild                          |
| Domain terms land too close together  | Fine-tune the embedding model on query-passage pairs (contrastive loss) |

- Chunking is fixed per data source; search type can be overridden per query on supported stores.

📖 Full lesson: [Training, Hyperparameter Tuning and Fine-Tuning Models on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-training-hyperparameter-tuning-fine-tuning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 2.3 Analyze and evaluate the performance of ML and AI systems

- **Reproducibility** = data version + code version + parameters + environment + metrics.
  - **Managed MLflow on SageMaker AI** (serverless MLflow App or tracking server): runs, `autolog()`, comparison UI, Model Registry.
  - A training job must call `mlflow.set_tracking_uri()` with the **tracking server ARN** and install the **sagemaker-mlflow** plugin. Without them MLflow writes locally, the container is discarded, and the job still succeeds.
  - Log data with `mlflow.log_input` (S3 URI plus digest) and tag the **Git commit**.
- **Bedrock Prompt Management:** a **draft** you edit, immutable numbered **versions** with their own ARN, and **variants** to compare side by side. Fix a released prompt by creating a new version and repointing; deleting a version breaks its callers.
- **Baselines** come from the **training data** and held-out metrics, never from recent traffic.

| Drift type        | What changes                 | Detect with                                         |
| ----------------- | ---------------------------- | --------------------------------------------------- |
| Data drift        | Input distributions          | Per-feature stats vs training (PSI, KS test)        |
| Concept drift     | Input-to-label relationship  | Ground truth joined to predictions, quality metrics |
| Attribution drift | Features the model relies on | Live vs training SHAP                               |
| Bias drift        | Fairness metrics over time   | Same bias metrics on live predictions               |

- Delayed labels: log an **inference ID**, join late labels by it, alarm when recall falls. Input monitors stay flat when only the relationship changes.
- **SageMaker Model Monitor, Clarify and Debugger are closed to new customers.** New accounts: endpoint data capture + scheduled job running **Evidently** presets + results in **MLflow** + **SNS** alerts + CloudWatch for endpoint health. Existing Model Monitor: data capture → baselining job (`statistics.json`, `constraints.json`) → schedule.
- **Shadow vs production variants:** a shadow variant gets a copy of live requests and its answers never reach callers. Needs an **instance-based real-time endpoint** (not serverless, async, multi-model, multi-container, Marketplace or Inf1). Enable data capture to compare per request. Completing the test can deploy the shadow variant on the same endpoint name.
- **Explanations:**

| Question                                   | Technique                          |
| ------------------------------------------ | ---------------------------------- |
| Why did this one person get this decision? | Local SHAP values                  |
| Which features matter most overall?        | Mean **absolute** SHAP per feature |
| What happens to predictions as X changes?  | Partial dependence plot            |

- **Post-training bias:** DPPL (gap in predicted positive rates), RD (gap in recall among true positives), DRR (correctness of rejections). DPL is pre-training.
- **Convergence debugging:**

| Symptom                                   | Fix                                                       |
| ----------------------------------------- | --------------------------------------------------------- |
| Loss drops, swings, then NaN              | Lower learning rate (warmup or decay)                     |
| Flat loss, tiny gradients in early layers | ReLU-family with He initialization, batch norm, residuals |
| Gradient norms spike                      | Gradient clipping, lower learning rate                    |
| Both losses plateau high                  | Raise learning rate or capacity                           |

- New-account training visibility: metric definition + CloudWatch alarm, TensorBoard on SageMaker AI, loss curves in MLflow.
- **Classic metrics:**

| Metric    | Use when                                    |
| --------- | ------------------------------------------- |
| Accuracy  | Balanced classes, equal error costs         |
| Recall    | Missing a positive is expensive             |
| Precision | False alarms are expensive                  |
| F1        | Balance both at a chosen threshold          |
| ROC AUC   | Threshold-free, reasonably balanced classes |
| PR AUC    | Rare positives, threshold not chosen        |
| MAE       | Cost grows linearly with error              |
| RMSE      | Large misses disproportionately costly      |

- **NLP metrics:** BLEU (precision of n-grams, translation), ROUGE (recall of n-grams or longest common subsequence, summaries), BERTScore and embedding similarity (credit paraphrases). Perplexity is not reference-based.
- **Bedrock evaluation jobs:**

| Job            | Scorer             | Choose when                                                                 |
| -------------- | ------------------ | --------------------------------------------------------------------------- |
| Automatic      | Fixed algorithms   | Repeatable regression tests (summarization accuracy by BERTScore, QA by F1) |
| LLM-as-a-judge | Evaluator model    | Subjective quality at scale; custom metrics for org rules                   |
| Human-based    | Your own work team | Expert or confidential review                                               |
| RAG evaluation | Judge model        | Knowledge bases or your own RAG pipeline                                    |

- Models hosted elsewhere: bring your own inference responses in the prompt dataset.
- Human jobs compare up to two models; set workers per prompt. Rating methods: choice buttons (preference only), Likert comparison (strength of preference), Likert individual, thumbs up/down, ordinal rank.
- **Judge metrics:** faithfulness, completeness, correctness, helpfulness, relevance, coherence, following instructions, professional style and tone, harmfulness, **stereotyping**, refusal. Judge biases (self-preference, position, verbosity) → use a judge from a **different model family** and calibrate against experts.
- **RAG evaluation:** **retrieve-only** (context relevance, context coverage) localizes retrieval faults; **retrieve-and-generate** adds faithfulness, citation precision, citation coverage and more. High context scores with low faithfulness = fix generation, not retrieval.
- **Confusions:** citation precision = are citations correct; citation coverage = how much of the answer is cited; faithfulness = grounded at all. Guardrails block at runtime and produce no evaluation report.

📖 Full lesson: [Evaluating ML and GenAI Models: Metrics, Drift, Explainability and RAG on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-and-genai-model-evaluation-drift-explainability/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

[← Back to the study guide](../README.md)
