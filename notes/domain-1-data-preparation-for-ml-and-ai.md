# Domain 1: Data Preparation for ML and AI (28%)

The heaviest domain. It covers getting data into the right AWS storage and format, transforming it into features, preparing documents and datasets for foundation models, and checking quality and bias before anything is trained. Questions are short pipeline scenarios with one or two constraints (no code changes, least operational effort, cost, latency, data location, freshness). The right answer meets every stated constraint, and options often differ by a single setting, so know what each setting actually does and which ones are fixed at creation.

## 1.1 Collect and store data

- **SageMaker AI training input (per channel):**

| Setting                | How data arrives                                        | Fits                                                 | Watch out for                                            |
| ---------------------- | ------------------------------------------------------- | ---------------------------------------------------- | -------------------------------------------------------- |
| S3 File mode (default) | Whole channel downloaded to the ML storage volume first | Small/medium data, random access, local-file scripts | Start-up grows with data size; volume must hold it all   |
| S3 FastFile mode       | Objects exposed as POSIX files, streamed as read        | Large, mostly sequential reads; no script change     | Millions of tiny objects stay slow (one S3 request each) |
| S3 Pipe mode           | Streamed through a Unix named pipe                      | Legacy pipe readers                                  | Script must read the pipe                                |

- AWS lists "configuring data to load into the model training resource (for example, Amazon EFS, Amazon FSx)" as removed in MLA-C02. EFS is still named as a data source to extract from.
- **Data distribution** is a separate setting from input mode. **FullyReplicated** (default) gives every instance everything. **ShardedByS3Key** gives each instance its own slice, which suits data-parallel training and fixes out-of-disk errors.
- EBS here is mostly the training instance's ML storage volume. A bigger volume does not remove the per-job download.
- **Extract without loading production:**
  - **DynamoDB export to S3** needs **PITR** enabled, consumes no read capacity, and writes DynamoDB JSON or Ion. Full or **incremental** (a 15-minute to 24-hour window). No built-in schedule, so trigger it from EventBridge Scheduler or a pipeline.
  - **RDS/Aurora snapshot export to S3** writes **Parquet** from a snapshot, never querying the instance.
  - Trap: anything that queries the live database competes with production, however it is scheduled. Change capture only records changes after it is turned on.
- **Storage choice:** S3 is the default for training data of every type. EFS for shared POSIX files, FSx for Lustre for maximum repeated read throughput, S3 Express One Zone for single-digit-millisecond access in one AZ.
- **S3 classes:** Intelligent-Tiering when access is unknown (no retrieval fees; its Archive tiers must be activated separately). A lifecycle rule when you know when data cools. Glacier Deep Archive for the cheapest long-term retention (about 12 hours standard, 48 bulk).
- **Compliance controls:**
  - "We control the keys" → **SSE-KMS with a customer managed key** (key policy, rotation, CloudTrail audit). SSE-S3 does not meet it.
  - Training jobs have two key settings: **KmsKeyId** (OutputDataConfig, model artifacts) and **VolumeKmsKeyId** (ResourceConfig, ML storage volumes).
  - **Object Lock** (needs versioning): **compliance mode** cannot be bypassed by anyone, including root; **governance mode** can be bypassed with `s3:BypassGovernanceRetention`. Versioning alone does not stop deletion.
  - **Residency** is decided by Region choice alone. A training job reads from its own Region, so cross-Region training means copying the data first.
  - **Macie** finds sensitive data inside S3 objects. Services that watch access or configuration do not look inside objects.
- **Capacity troubleshooting:**

| Symptom                                                 | Cause                         | Fix                                                                  |
| ------------------------------------------------------- | ----------------------------- | -------------------------------------------------------------------- |
| `ProvisionedThroughputExceededException`, one hot shard | Low-cardinality partition key | High-cardinality key (device or session ID), not more shards         |
| Rising IteratorAge, Lambda processing is the constraint | One batch per shard at a time | Raise **ParallelizationFactor** (1 to 10)                            |
| S3 503 Slow Down on one prefix                          | Per-prefix request-rate limit | More prefixes, fewer larger files (retries do not raise the ceiling) |
| Slow listing and opening, many KB-sized objects         | Small-file overhead           | Compact to roughly 100 MB+ files                                     |

- Kinesis shard: 1 MB/s or 1,000 records/s in, 2 MB/s out shared by standard consumers.
- **Streaming service choice:**
  - **Data Firehose**: delivery only, no consumer code. Cannot replay or be read by your apps.
  - **Kinesis Data Streams**: several readers, replay (24 hours default, up to 365 days), low latency. Needs a consumer to reach S3.
  - **Amazon MSK**: producers already speak Kafka.
  - **Managed Service for Apache Flink**: stateful windows, joins, exactly-once state.
  - **Enhanced fan-out** gives each consumer its own 2 MB/s per shard (about 70 ms vs 200 ms+). It solves consumer contention; **retention** solves replay. Need both, set both.
- **Firehose settings:**
  - **Format conversion** to Parquet/ORC needs **JSON input** and a **Glue Data Catalog table** as schema. CSV must be turned into JSON by Lambda first.
  - **Lambda transform** returns each record with `recordId`, `result` (Ok, Dropped, ProcessingFailed) and base64 data. It changes content, not timing.
  - **Dynamic partitioning** builds prefixes from values inside records (`!{partitionKeyFromQuery:...}`). Without it, prefixes only use arrival time or Firehose namespaces.
  - **Buffering hints**: size up to 128 MiB, interval up to 900 s, whichever fills first. For bigger objects, raise the limit that is actually triggering.
- **Formats:** Parquet/ORC for column reads (Athena bills per byte scanned). CSV for built-in algorithms (**target first, no header**). **JSON Lines** for Bedrock fine-tuning data. Avro for streaming messages, with **Glue Schema Registry** compatibility modes (such as BACKWARD) rejecting breaking schemas.
- **Merging sources:** Glue for serverless Spark, EMR for cluster control.
  - `push_down_predicate` prunes partitions in the catalog before reading.
  - A JDBC read uses one connection by default; `hashfield`/`hashexpression` with `hashpartitions` parallelises it.
  - Job bookmarks = only new data; reprocessing a chosen date = partition predicate.
  - **Broadcast join** avoids shuffling the large table when the other side fits in memory. Auto-broadcast only below 10 MB by default, so larger tables need a hint or higher threshold.
  - Athena CTAS/INSERT INTO writes at most 100 partitions, so it suits ad hoc merges.
- **Vector stores:**

| Store                               | Best fit                                                       | Limits                                                           |
| ----------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- |
| Amazon S3 Vectors                   | Huge, cost-sensitive collections queried infrequently          | Similarity only, no keyword or hybrid search                     |
| OpenSearch Serverless vector search | High-rate production RAG, hybrid lexical plus vector search    | Higher baseline cost; a "search" collection has no vector fields |
| OpenSearch managed domain (k-NN)    | Teams already running OpenSearch, custom tuning                | You size, patch and scale                                        |
| Aurora/RDS PostgreSQL + pgvector    | Vectors beside relational data, results reflect committed rows | Scales as a database                                             |

- **Index configuration:** dimension is fixed at creation and must equal the model output (Titan Text Embeddings V2: 1,024 default, or 512/256; Titan G1 Text: 1,536). Metric should match training. **HNSW** needs no training step and works on an empty table; **IVFFlat** builds faster with less memory but needs representative data first and usually recalls less.
- **Bedrock knowledge base prerequisites:** OpenSearch index with a `knn_vector` field of the right dimension using the **faiss** engine, a text field and the Bedrock metadata field. Aurora pgvector: same account, **RDS Data API** enabled, table with id, embedding, chunks, metadata, HNSW index with `vector_cosine_ops`, GIN index for hybrid search, credentials in Secrets Manager.
- **Unstructured data:** objects in S3, attributes in DynamoDB or a relational table keyed by the S3 key (a DynamoDB item caps at 400 KB). Pack millions of small images into shard files (WebDataset tar, RecordIO, TFRecord). Knowledge base metadata comes from a `<file>.metadata.json` sidecar.
- **Feature Store ingestion:** `PutRecord` for streaming updates; Spark connector, `FeatureGroup.ingest()` or a Data Wrangler export for bulk. Backfill billions of rows with the Spark connector targeting **OfflineStore** only.
  - Online store keeps the newest event time per record; an older event time does not overwrite it.
  - Offline replication takes up to about 15 minutes, so read-after-write SQL lags.
  - Offline table format (Glue or **Iceberg**) is chosen only at creation. Compact small files on Iceberg with Athena `OPTIMIZE ... REWRITE DATA`.
  - **TTL** applies to the online store only; expiry is recorded in the offline store, so history stays.
- **Traps:**
  - More shards do not fix a hot key.
  - A Lambda transform cannot produce Parquet files; format conversion does that.
  - A different embedding model with the same dimension still produces an unrelated vector space.

📖 Full lesson: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 1.2 Perform data transformation, feature engineering, and pre-processing

- **Tool choice** (all read and write S3, so the constraint decides):

| Tool                    | Best fit                                                             | Watch out for                                        |
| ----------------------- | -------------------------------------------------------------------- | ---------------------------------------------------- |
| AWS Glue ETL            | Code-based serverless Spark, incremental runs via job bookmarks      | Less control of Spark internals and native libraries |
| Glue DataBrew           | Analysts, visual no-code recipes, scheduled recipe jobs              | Tabular prep only, no custom algorithms or streaming |
| Spark on EMR            | Custom native libraries, tuned Spark, Spot cost control              | Highest operational overhead                         |
| SageMaker Data Wrangler | Data scientists preparing ML features (now part of SageMaker Canvas) | Interactive; production runs come from exports       |

- Glue bills per DPU-hour; the **Flex** execution class is cheaper for non-urgent work. Moving a PySpark script from EMR to Glue with bookmarks removes cluster management and makes it incremental.
- EMR cost: **Spot for task nodes**, On-Demand for core nodes (core nodes hold HDFS).
- **Productionise a Data Wrangler flow by exporting it**: to a SageMaker Pipelines processing step, to Feature Store, to S3 via a processing job, or to Python. Rewriting it by hand causes training/serving drift.
- **Feature Store use:** every feature group needs a **record identifier** and an **event time**. Online store = latest value in milliseconds (Standard or **InMemory** tier, TtlDuration). Offline store = every version in S3, queried through the Glue Data Catalog.
- **Point-in-time correct joins:** match each label to the feature version with the latest event time at or before the label's timestamp (SDK `create_dataset` or an Athena query). Never key on `write_time` or `api_invocation_time`, which record ingestion, not truth.
- **Streaming transforms:**
  - Stateless per record → Lambda (Firehose transform or event source mapping).
  - Short fixed per-key aggregates → event source mapping **tumbling window** (up to 15 minutes). Batching settings keep no state.
  - Sliding or event-time windows, late events, stream joins → Spark Structured Streaming or Flink.
  - Spark **watermark** (`withWatermark`, defined before the aggregation) sets how late data may arrive and bounds state. A trigger interval only controls micro-batch frequency.
- **Scaling:** needed for distance, dot-product and gradient methods (k-NN, k-means, SVM, linear/logistic, neural nets). Trees such as XGBoost are largely unaffected.

| Technique       | Formula                 | Use when                                  |
| --------------- | ----------------------- | ----------------------------------------- |
| Standardization | (x − mean) / std        | Default for distance and gradient methods |
| Min-max         | (x − min) / (max − min) | Bounded range needed, no extreme outliers |
| Robust scaling  | (x − median) / IQR      | Legitimate outliers must stay             |

- **Fit every transform (scaler, encoder, imputer) on the training split only**, then reuse the saved parameters on validation, test and inference.
- **Log transform** compresses a right tail; use `log(1 + x)` when zeros exist. With a log target, exponentiate predictions before computing RMSE in business units.
- **Binning** plus one-hot lets a linear model fit a non-monotonic relationship (demand high when cold, low when mild, high when hot). Quantile bins = similar counts; equal-width = fixed intervals.
- **Feature splitting:** split timestamps into hour, day of week, weekend flag, in the event's **local** time zone when behaviour follows local clocks. Split compound strings.
- **Encoding:** nominal → one-hot; ordinal → integers in real order. Hashing needs enough buckets to avoid collisions.
- **Embeddings:** the index dimension must equal the model output; documents and queries must use the same model, version and settings. Changing dimension means a new index and a full re-embed. Text-only models cannot embed images; use a multimodal model (Titan Multimodal Embeddings, Nova multimodal embeddings).
- **Tokenization:** limits are counted in the model's own tokens. IDs, URLs, stack traces and non-English text use many more tokens than words, so count with the target tokenizer and leave headroom.
- **Domain augmentation must preserve the label:** curated vocabulary swaps with expert review, **back-translation** (for example Amazon Translate), reviewed paraphrases. Generic synonym swaps can change meaning.
- **RAG chunking (Bedrock Knowledge Bases)**, fixed when the data source is created:

| Strategy     | Use when                                                                   |
| ------------ | -------------------------------------------------------------------------- |
| Default      | About 300-token chunks respecting sentences; a starting point              |
| Fixed-size   | Predictable sizes, with overlap for boundary-spanning sentences            |
| Hierarchical | Precise child matches, parent chunk returned for context                   |
| Semantic     | Topics shift unevenly; tune max tokens, buffer size, breakpoint percentile |
| No chunking  | Files are already short self-contained units, or you pre-split             |

- Semantic: a **higher** breakpoint percentile = fewer, larger chunks. Buffer size is not a size control.
- **Custom chunking:** no chunking plus a custom transformation Lambda and an intermediate S3 location.
- **Metadata:** `<document>.metadata.json` sidecar with `metadataAttributes`, picked up on the next sync, used as query-time filters.
- **Parsing:** for tables, charts and images in PDFs use an advanced parser (FM parser or Bedrock Data Automation). Chunking cannot recover what parsing dropped.
- **Masking and redaction** must change the training data itself: Comprehend PII redaction jobs, Glue Detect Sensitive Data, DataBrew masking and keyed hashing, Transcribe PII redaction. Macie only discovers, KMS only encrypts, Guardrails acts at runtime.
- Joinable pseudonyms need a **deterministic keyed hash** (secret in Secrets Manager). Unkeyed hashes of guessable IDs can be reversed by brute force.
- **FM customization data (JSONL in S3):**

| Method                 | You have                            | Record                                                                 |
| ---------------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| Fine-tuning (text)     | Labeled input-output pairs          | `{"prompt": ..., "completion": ...}`                                   |
| Fine-tuning (chat)     | Conversations                       | `bedrock-conversation-2024`: system block + alternating user/assistant |
| Continued pre-training | Unlabeled domain text               | `{"input": ...}`                                                       |
| Distillation           | Prompts only, plus a larger teacher | Prompts or invocation logs; teacher writes answers                     |

- **Traps:** continued pre-training teaches vocabulary and style, not a task. In distillation the larger model is the teacher, the smaller one the student.

📖 Full lesson: [Data Transformation, Feature Engineering and Pre-processing on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-feature-engineering-data-transformation/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 1.3 Validate data quality and manage bias

- **AWS Glue Data Quality** evaluates a **DQDL** ruleset and reports pass/fail per rule plus a score.

|                | Data Catalog data quality          | ETL data quality (Evaluate Data Quality transform) |
| -------------- | ---------------------------------- | -------------------------------------------------- |
| Checks         | Tables at rest in the Data Catalog | Data in transit inside a Glue job                  |
| User           | Stewards, console, no code         | Pipeline engineers                                 |
| Starting rules | **Rule recommendation** runs       | You write the ruleset                              |
| Results        | Dataset-level outcomes and score   | Plus **row-level** results to quarantine bad rows  |
| Only new data  | Partition predicate                | Job bookmarks                                      |

- A check after data is written can report a bad batch but cannot keep its rows out of training.
- **DQDL rules:** Boolean rules (`IsComplete`, `IsUnique`) allow zero failures. Ratio rules (`Completeness`, `Uniqueness`) take a threshold. Also `ColumnValues`, `RowCount`, `ColumnExists`, custom SQL.
- **where clauses** (Glue 4.0+) filter rows a rule evaluates. Filter on the condition column, check the target column.
- **Dynamic rules:** `last(k)` with k > 1 must be aggregated (avg, median, min, max, sum, std, index). `RowCount > avg(last(5)) * 0.8` fails only on a drop of more than 20%. `index(last(k), 0)` compares with the most recent run only.
- **DataBrew validation:** a ruleset is evaluated by a **profile job**; recipe jobs never validate. The job ends SUCCEEDED even when rules fail; act on the EventBridge **Ruleset Validation Result** event (`validationState` FAILED → SNS, SUCCEEDED → start a SageMaker pipeline).
- **Labeling quality:** several annotators plus consolidation improves labels; **gold-standard items** measure each annotator; **random audits** measure the dataset. FM pre-labeling (for example Bedrock batch inference) plus human sample review cuts cost.
- **SageMaker Ground Truth is closed to new customers.** For existing users: private workforce (Cognito or OIDC) for sensitive data, vendor workforce from Marketplace, public Mechanical Turk for non-sensitive volume. Automated labeling covers only single-label image classification, semantic segmentation, bounding boxes and single-label text classification, batch only.
- **Splits:**

| Split      | Use when                                                      |
| ---------- | ------------------------------------------------------------- |
| Randomized | Rows are independent                                          |
| Stratified | A rare class must appear in every split                       |
| Ordered    | Time series: train on the past, test on the future            |
| By key     | Several rows describe one entity (speaker, patient, customer) |

- Symptom of a wrong split: a high test score that collapses on new data. Stratifying or changing the ratio does not fix a leak.
- Feature leakage: rolling or lag features must use only data available before the prediction point. Shuffle sorted exports before a positional split.
- **Augmentation:** label-preserving transforms only (mirroring turns b into d), applied to the **training split only**, measured on real held-out examples.
- **Pre-training bias metrics** (facet a favoured, facet d disfavoured):

| Metric  | Measures                                                         |
| ------- | ---------------------------------------------------------------- |
| CI      | Row-count imbalance between facets, −1 to 1                      |
| DPL     | Difference in positive-label rates, q(a) − q(d)                  |
| KL, JS  | Difference between label distributions (JS symmetric, bounded)   |
| Lp, TVD | Distance between label distributions (TVD = half L1)             |
| KS      | Largest gap between label distributions                          |
| CDD     | Disparity per subgroup, size-weighted; exposes Simpson's paradox |

- **SageMaker Clarify is closed to new customers**; the metrics are simple formulas you can compute in pandas or Spark.
- **Confusions:** CI is about how many rows, DPL about how often the positive label. DPPL is post-training and needs predictions.
- **Facets by data type:** threshold or bin numeric attributes; derive text facets with a service (for example Comprehend dominant language), never from the label itself; take image facets from annotations or metadata, not pixel statistics.
- **Class imbalance:** split first, then resample the training split only; the test set keeps the real distribution.
  - Oversampling duplicates (overfitting risk), undersampling discards, **SMOTE** interpolates numeric minority rows.
  - Data must stay as recorded → weights. Binary XGBoost `scale_pos_weight` ≈ negatives / positives. Multi-class built-in XGBoost with CSV: instance-weight column after the label plus `csv_weights=1`. Deep learning: inverse-frequency class weights or focal loss.
  - Judge with per-class recall, precision or F1, not accuracy.
- **Fine-tuning data integrity:** JSONL schema correct, system prompt outside `messages`, turns alternate starting with user, records within the model's token limits, non-answers removed, near-duplicates collapsed, train/validation split by source ID. A near-zero validation loss with poor answers to new questions signals overlap.
- **Safety screening:** **ApplyGuardrail** reuses an existing guardrail's policies without invoking a model; Comprehend **DetectToxicContent** gives per-category scores (filter on the category the policy cares about); Rekognition **DetectModerationLabels** for images (StartContentModeration for stored video).
- **Cleaning:**
  - Outlier transforms in Data Wrangler: min-max (known valid range), standard deviation, **robust** standard deviation (extremes inflate the mean and hide themselves), quantile (skewed, no known range). Fix by Remove, Clip or Invalidate.
  - Sentinel codes (9999) are missing values, not outliers.
  - Impute skewed numbers with the **median**, categories with the **mode**, add a missing-indicator when missingness predicts the target. Fit on training data only.
  - Fuzzy duplicates → **Glue FindMatches**: favour precision when false merges are costly, recall when missed duplicates are worse. Deduplicate before splitting.
- **Traps:** keep informative extremes such as traffic spikes before an outage. "Replace rare" is a categorical transform, not a numeric outlier detector.

📖 Full lesson: [Data Quality Validation, Bias Metrics and Class Imbalance for ML (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-data-quality-validation-bias-class-imbalance/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

[← Back to the study guide](../README.md)
