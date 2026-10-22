# MLA-C02 Sample Questions with Answers

20 worked practice questions for the **AWS Certified Machine Learning Engineer - Associate (MLA-C02)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [MLA-C02 practice questions](https://www.savemycert.com/practice/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide) on SaveMyCert.

## Question 1

*Data Preparation for ML and AI*

**An ML engineer trains a PyTorch model on 3 TB of TFRecord shards stored under one Amazon S3 prefix. Each SageMaker AI training job uses the default input mode and spends about 40 minutes copying data before the first epoch begins. The training script opens the shards through ordinary file paths and must not be changed. Which solution will remove the start-up delay with the LEAST operational overhead?** Choose one.

- **a.** Create an FSx for Lustre file system linked to the bucket and mount it as the channel
- **b.** Set the S3 channel's input mode to Pipe and keep the existing S3 prefix
- **c.** Set the S3 channel's input mode to FastFile and keep the existing S3 prefix
- **d.** Keep File mode on the S3 channel and raise the training volume size for the dataset

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ FSx for Lustre linked to the bucket also removes the copy, but it adds a file system to provision, pay for and run, and the job must be attached to a VPC subnet, so it carries more operational overhead than changing the input mode.
- **b** ❌ Pipe mode streams data into named FIFO pipes that the script has to read as a stream, so the file-path code would have to be rewritten.
- **c** ✅ FastFile mode presents the S3 objects through a POSIX file interface and streams them on demand, so training starts without the up-front copy and the script's file reads keep working unchanged.
- **d** ❌ A larger volume avoids running out of disk but File mode still downloads the full 3 TB before training starts, so the 40-minute delay stays.

**The concept.** SageMaker AI training input modes: File (copy first), FastFile (stream with file semantics) and Pipe (stream through FIFO pipes).

**Why this is correct.** File mode, the default, copies the whole dataset to the instance's storage before the script runs, which is where the 40 minutes go. Two options remove that copy without touching the script: FastFile mode and an FSx for Lustre channel. FastFile is only a channel setting on the existing S3 prefix; it keeps POSIX file access and streams objects as the script reads them. FSx for Lustre works too, but it is a new file system plus VPC networking to set up and operate, so it loses on operational overhead. Pipe mode would need the script rewritten to read named pipes, and a bigger volume keeps the copy.

**How to reason it out**

1. Identify the cause: File mode downloads the full dataset before training starts.
2. Rule out Pipe mode (script must change) and a bigger volume (the copy stays).
3. Compare the two that work: FastFile is a channel setting; FSx for Lustre is new infrastructure.
4. Choose FastFile mode on the same S3 prefix as the least overhead.

> **Exam tip:** Slow start in File mode with a file-based script: FastFile is the lowest-effort fix; FSx for Lustre also works but is more to run.

</details>

📖 Learn this topic: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 2

*Data Preparation for ML and AI*

**A data-parallel training job runs on eight ml.p4d instances and reads 24,000 image archives from one Amazon S3 prefix in File mode. Every instance runs out of local storage during the download. The training script opens the archives as ordinary files from a local path, does no sharding of its own and assumes that each instance receives a different part of the dataset. The script must not be changed. Which channel configuration meets these requirements?** Choose one.

- **a.** Pipe input mode with the S3 data distribution type set to FullyReplicated
- **b.** File input mode with the S3 data distribution type set to ShardedByS3Key
- **c.** File input mode with the S3 data distribution type set to FullyReplicated
- **d.** Pipe input mode with the S3 data distribution type set to ShardedByS3Key

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Pipe mode removes the disk pressure only by streaming into a FIFO pipe, which needs a script change, and replication still hands every instance the whole dataset.
- **b** ✅ ShardedByS3Key gives each instance a distinct eighth of the objects, which matches the script's assumption and cuts each instance's download to an eighth of the dataset, while File mode keeps the local file paths the script already reads.
- **c** ❌ FullyReplicated is the default that caused the problem: every instance downloads the whole dataset, and every instance would train on the same data.
- **d** ❌ Sharding is right, but Pipe mode streams the data into a FIFO pipe instead of files on disk, so the script would have to be rewritten to read from the pipe.

**The concept.** Two independent channel settings: the input mode (File, FastFile, Pipe) decides how data reaches the container; the S3 data distribution type (FullyReplicated, ShardedByS3Key) decides which objects each instance gets.

**Why this is correct.** Two facts must be combined. First, the default FullyReplicated distribution gives each instance the entire dataset, which is why every instance runs out of space and why the workers would train on identical data; ShardedByS3Key splits the objects across the instances. Second, Pipe mode delivers data through a FIFO pipe rather than as files, so it breaks the unchanged script that opens files from a local path. File mode with ShardedByS3Key fixes both the storage and the data split without touching the script.

**How to reason it out**

1. Separate the two settings: input mode and data distribution type.
2. Distribution: the script expects distinct data per worker, so choose ShardedByS3Key.
3. Input mode: the script reads local files and must not change, so rule out Pipe mode.
4. Combine them: File mode with ShardedByS3Key.

> **Exam tip:** Distinct data per worker is ShardedByS3Key; reading ordinary files without code changes rules out Pipe mode.

</details>

📖 Learn this topic: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 3

*Data Preparation for ML and AI*

**A retailer keeps 2 TB of order history in an Amazon DynamoDB table that serves production traffic with provisioned capacity. Each week an ML engineer needs a full copy of the table in Amazon S3 to build a training dataset. The copy must not consume the production table's read capacity. Which solution will meet these requirements with the LEAST operational overhead?** Choose one.

- **a.** Take a weekly on-demand backup and restore it to a new table for the training pipeline
- **b.** Enable PITR and start a weekly DynamoDB export to S3 from EventBridge Scheduler
- **c.** Turn on DynamoDB Streams and have an AWS Lambda function write each item change to S3
- **d.** Run a weekly AWS Glue ETL job that reads the table with parallel scans and writes to S3

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Scanning a restored copy spares the production table's capacity, but it means a weekly restore, a scan job to write the items to S3 and a table to delete afterwards, which is far more to operate than a managed export.
- **b** ✅ The export to S3 feature reads from the point-in-time recovery backup, not the live table, so it consumes no read capacity and needs no scanning code; EventBridge Scheduler can call the export API each week.
- **c** ❌ Streams capture only changes made after they are enabled, so the existing items never reach S3, and the Lambda writer is custom code to build and run.
- **d** ❌ Parallel scans read the live table and consume its provisioned read capacity, competing with production traffic.

**The concept.** DynamoDB export to Amazon S3 builds full or incremental exports from point-in-time recovery data without touching table capacity.

**Why this is correct.** DynamoDB's export to S3 feature requires point-in-time recovery to be enabled and writes the table's data to an S3 prefix in DynamoDB JSON or Amazon Ion format. Because it reads from the continuous backup, it uses no read capacity and needs no paging code; EventBridge Scheduler can start it each week. Restoring a backup to a new table and scanning that copy also leaves production alone, but it adds a restore, a scan-and-write job and a cleanup step every week. A Glue scan of the live table competes with production, and Streams miss the existing items.

**How to reason it out**

1. Note the constraints: full copy in S3, no production read capacity.
2. Rule out a scan of the live table (capacity) and Streams (miss existing items).
3. Compare the two that work: managed export versus restore plus scan.
4. Choose PITR plus a scheduled export to S3 as the least overhead.

> **Exam tip:** Full DynamoDB copy in S3 without read capacity and with least effort: PITR plus export to S3.

</details>

📖 Learn this topic: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 4

*Data Preparation for ML and AI*

**A retail company stores sales extracts as CSV files in Amazon S3. Business analysts who do not write code need to clean the files each week before the data science team trains a forecasting model: trim whitespace, fix inconsistent date formats and fill missing store codes. The analysts want to build the steps visually and rerun them as a saved, repeatable job. Which service should they use?** Choose one.

- **a.** Build an AWS Glue DataBrew recipe of cleanup steps and run it as a scheduled recipe job
- **b.** Load the files into Amazon Athena tables and write SQL views that clean each weekly extract
- **c.** Author the cleanup in an AWS Glue ETL job script and run it on a weekly Glue trigger
- **d.** Launch an Amazon EMR cluster each week and run the cleanup as a Spark step on the cluster

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Correct. DataBrew is the visual, no-code data preparation tool: analysts build a recipe from point-and-click transformations and schedule it as a repeatable recipe job.
- **b** ❌ Athena views need hand-written SQL and only reshape data at query time; they do not give a visual, saved recipe that writes cleaned output.
- **c** ❌ A Glue ETL job script is PySpark or Scala code. It runs serverless, but the analysts cannot write the code the job needs.
- **d** ❌ EMR gives full control of Spark clusters, but the cleanup must still be written as Spark code and the cluster managed, which the analysts cannot do.

**The concept.** AWS Glue DataBrew is a visual data preparation service whose saved transformation recipes can be run as scheduled jobs without writing code.

**Why this is correct.** The deciding constraints are users who do not write code and steps they build visually and rerun. DataBrew recipes are exactly that: hundreds of built-in transformations applied point-and-click, saved, versioned and run as recipe jobs on a schedule. Glue ETL jobs and EMR steps are code; Athena views are SQL and do not produce a cleaned dataset.

**How to reason it out**

1. Identify who builds the transformation: analysts without coding skills.
2. Rule out the code-based options (Glue ETL scripts, EMR Spark steps, SQL views).
3. Pick the visual recipe tool that can be scheduled as a job: DataBrew.

> **Exam tip:** Visual, no-code, repeatable cleaning of tabular data points to AWS Glue DataBrew recipes and recipe jobs.

</details>

📖 Learn this topic: [Data Transformation, Feature Engineering and Pre-processing on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-feature-engineering-data-transformation/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 5

*Data Preparation for ML and AI*

**An AWS Glue ETL job writes a daily batch of customer records to an S3 prefix that a churn model trains from. Some batches contain rows with a null label or a negative tenure, and those rows have already reached training. The ML engineer wants each run to separate the failing rows into a quarantine prefix while the passing rows continue to the training prefix in the same run. Which solution will meet these requirements with the LEAST operational overhead?** Choose one.

- **a.** Attach an AWS Glue DataBrew ruleset to a profile job on the output prefix and alert through Amazon EventBridge
- **b.** Add a custom PySpark step to the ETL job that tests each row's label and tenure and writes failures to a quarantine prefix
- **c.** Run AWS Glue Data Quality against the Data Catalog table for the training prefix on a schedule after each job run
- **d.** Add an Evaluate Data Quality transform to the ETL job and route rows by their row-level result to two S3 prefixes

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ A DataBrew ruleset validation reports whether the dataset passed or failed as a whole after it is written; it does not split individual rows into separate locations.
- **b** ❌ Custom code in the job would split the rows in the same run, but the engineer must write, test and maintain the checks and routing logic that the built-in transform already provides.
- **c** ❌ Data quality for the Data Catalog evaluates data at rest after it has landed, and it does not identify the individual records that failed, so bad rows would still reach the training prefix first.
- **d** ✅ Data quality in a Glue ETL job runs before the write and can mark each record as passed or failed, so the job can send failed rows to quarantine and passed rows to training in one run.

**The concept.** AWS Glue Data Quality has two entry points: the Data Catalog (data at rest) and ETL jobs (proactive checks on data in flight).

**Why this is correct.** The requirement is to stop bad rows before they land and keep the good rows flowing in the same run, with the least effort. Glue Data Quality inside the ETL job identifies the records that failed a rule, so the job can route row-level failures to a quarantine prefix with a managed transform and declarative DQDL rules. A hand-written PySpark check would also work, but it is custom code to build and maintain. Catalog-based checks and DataBrew rulesets judge a dataset that has already been written, so bad rows still reach training.

**How to reason it out**

1. Note where the check has to happen: before the data reaches the training prefix.
2. Note the granularity: individual rows must be separated, not the whole batch rejected.
3. Compare the two options that work in the run: a managed transform with DQDL rules or custom PySpark code.
4. Choose the Evaluate Data Quality transform with row-level results, which needs no custom routing code.

> **Exam tip:** Stop bad rows in flight with the Evaluate Data Quality transform in the ETL job rather than hand-written row checks; check data at rest with Data Catalog data quality.

</details>

📖 Learn this topic: [Data Quality Validation, Bias Metrics and Class Imbalance for ML (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-data-quality-validation-bias-class-imbalance/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 6

*Data Preparation for ML and AI*

**Data scientists want a monthly copy of several tables from an Amazon RDS for PostgreSQL production database in Amazon S3, stored in a columnar format they can query with Amazon Athena. The database administrator will not allow any extra queries against the production instance. Which approach meets these requirements?** Choose one.

- **a.** Create an AWS DMS full-load task from the production instance to Amazon S3 as Parquet
- **b.** Create a read replica and have the data scientists query it from a SageMaker notebook
- **c.** Run pg_dump from an EC2 instance each month and upload the CSV output to Amazon S3
- **d.** Export an automated or manual DB snapshot to Amazon S3, which writes the tables as Parquet

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ DMS can write Parquet, but a full load reads every table from the production instance, which adds the load the administrator ruled out.
- **b** ❌ A read replica keeps the data in PostgreSQL rather than putting columnar files in S3 for Athena, and replication still adds work on the source.
- **c** ❌ pg_dump queries the production instance, which the administrator forbids, and produces row-based text rather than a columnar format.
- **d** ✅ Snapshot export reads from the snapshot rather than the instance and writes the selected tables to S3 in Apache Parquet, a columnar format Athena reads directly.

**The concept.** RDS and Aurora snapshot export to Amazon S3 writes table data as Apache Parquet without querying the running instance.

**Why this is correct.** Exporting a DB snapshot to S3 extracts data from the snapshot, so the production instance serves no extra queries, and the output is Parquet, which Athena can query efficiently. pg_dump and a DMS full load both read from the production instance. A read replica does not put columnar files in S3 at all.

**How to reason it out**

1. Note the two requirements: columnar files in S3 and no queries on production.
2. Recall that snapshot export works from a snapshot and outputs Parquet.
3. Eliminate options that read from the live instance.
4. Choose snapshot export to S3.

> **Exam tip:** RDS data to S3 as Parquet without touching the instance: export a snapshot.

</details>

📖 Learn this topic: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 7

*ML Model and Foundation Model (FM) Development*

**A legal-tech company is building an assistant on Amazon Bedrock that answers questions about individual contracts. Each contract can run to 400 pages, and the product team wants the whole contract sent to the model in a single request so that clauses that refer to each other are read together. Which model attribute should the ML engineer check first when shortlisting foundation models?** Choose one.

- **a.** The maximum number of output tokens the model returns per response
- **b.** The maximum context window, measured in input tokens
- **c.** The number of dimensions in the model's embeddings
- **d.** The on-demand price for each thousand output tokens

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Output length limits how long an answer can be. The requirement here is about how much text goes in, which the output limit does not govern.
- **b** ✅ A 400-page contract has to fit in the prompt in one request, so the context window is the first hard filter; a model whose window is too small is out regardless of quality or price.
- **c** ❌ Embedding dimensions matter for vector search with an embedding model. The assistant sends the contract to a text generation model, so this attribute does not decide whether the document fits.
- **d** ❌ Price matters when comparing models that already meet the requirement. A cheaper model whose context window cannot hold the contract fails the stated need.

**The concept.** Shortlisting a foundation model starts with hard task requirements such as input size, modality and language; price and quality are compared among models that pass them.

**Why this is correct.** The deciding requirement is that a document of hundreds of pages goes into one request. The context window (the input tokens a model accepts) is therefore a pass/fail filter. Output token limits, embedding dimensions and price are real selection criteria, but each is compared only among models whose context window can hold the contract.

**How to reason it out**

1. Find the hard requirement: the full contract in one request.
2. Translate it into a model attribute: input tokens accepted, the context window.
3. Filter the model catalog on that attribute first.
4. Compare price, latency and quality among the models that remain.

> **Exam tip:** Hard requirements (context window, modality, language) filter the list; cost and quality rank what is left.

</details>

📖 Learn this topic: [Choosing ML, Foundation Model and RAG Approaches on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-modeling-approaches-fm-selection-rag/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 8

*ML Model and Foundation Model (FM) Development*

**An online furniture retailer wants shoppers to upload a photo of an item and get visually similar products from its catalog. The same vector index also has to answer typed searches such as "green velvet armchair" by matching them against product photos, not only against descriptions. Which embedding approach meets these requirements?** Choose one.

- **a.** Amazon Titan Text Embeddings V2 applied to the product descriptions
- **b.** Amazon Rekognition DetectLabels on the photos, with the labels embedded as text
- **c.** Amazon Titan Multimodal Embeddings G1 applied to product images and titles
- **d.** Amazon Titan Multimodal Embeddings G1 applied to the product titles and descriptions

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A text embedding model cannot embed an uploaded photo, so the image-to-product search is impossible and typed searches never touch the photos.
- **b** ❌ Generic labels such as "chair" lose the visual detail (shape, fabric, colour shade) that similarity search needs, and the uploaded photo would be reduced to a few words.
- **c** ✅ A multimodal embedding model places images and text in the same vector space, so a photo query and a typed query can both be compared with the product images.
- **d** ❌ The right model applied to the wrong input: with only text embedded, typed queries are never compared with the photos, and an uploaded photo has no images in the index to match against.

**The concept.** Embedding models are chosen by modality first: text embeddings for text-to-text search, multimodal embeddings when images and text must be compared in one vector space.

**Why this is correct.** Both requirements, photo-to-photo similarity and text-to-photo search, need the product images themselves embedded into a vector space shared with text. A multimodal embedding model provides that space, but only if the images go into the index. Applying the same multimodal model to titles and descriptions alone is the near-twin trap: queries would only ever be matched against text. A text-only model cannot embed the uploaded photo at all, and converting photos to Rekognition labels throws away the visual features that make two products look alike.

**How to reason it out**

1. List the query types: an uploaded image and free text.
2. List what they are compared against: product images.
3. Choose a model that embeds both modalities into one space.
4. Check what gets embedded: the index must contain the images, not just text about them.

> **Exam tip:** Image and text in the same search means a multimodal embedding model.

</details>

📖 Learn this topic: [Choosing ML, Foundation Model and RAG Approaches on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-modeling-approaches-fm-selection-rag/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 9

*ML Model and Foundation Model (FM) Development*

**A manufacturer streams temperature and vibration readings from thousands of factory sensors into Amazon S3. Nobody has labeled which past readings represented equipment faults. An ML engineer must train a model in SageMaker AI that assigns each new reading a score showing how unusual it is compared with normal behavior. Which built-in algorithm should the engineer use?** Choose one.

- **a.** SageMaker AI XGBoost in binary classification mode
- **b.** SageMaker AI IP Insights
- **c.** SageMaker AI k-means clustering
- **d.** SageMaker AI Random Cut Forest (RCF)

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ XGBoost classification is supervised and needs labeled fault and non-fault examples, which the company does not have.
- **b** ❌ IP Insights is also unsupervised, but it learns associations between entities and IPv4 addresses to flag unusual logins, not anomalies in numeric sensor readings.
- **c** ❌ k-means groups records into clusters but does not produce an anomaly score; using distance to centroid as a proxy is extra work and is not what the algorithm is built for.
- **d** ✅ RCF is the unsupervised built-in algorithm for anomaly detection: it learns the shape of normal data and returns an anomaly score per record, with no labels needed.

**The concept.** Random Cut Forest is SageMaker AI's built-in unsupervised anomaly-detection algorithm.

**Why this is correct.** The data is unlabeled and the requirement is an anomaly score per record. Random Cut Forest is designed for exactly that and outputs a score for each data point. XGBoost needs labels, k-means only clusters, and IP Insights is specific to entity/IP-address pairs.

**How to reason it out**

1. Notice there are no labels, so supervised algorithms are out.
2. Notice the output must be an unusualness score for each record.
3. Match unsupervised anomaly scoring on numeric data to Random Cut Forest.

> **Exam tip:** Unlabeled numeric data plus an anomaly score per record points to Random Cut Forest.

</details>

📖 Learn this topic: [Training, Hyperparameter Tuning and Fine-Tuning Models on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-training-hyperparameter-tuning-fine-tuning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 10

*ML Model and Foundation Model (FM) Development*

**A data science team trains dozens of model candidates each week in SageMaker AI and records results in a shared spreadsheet. Last month nobody could tell which hyperparameters and dataset produced the model that went to production. The team wants an open-source-compatible tool to record every run, compare runs side by side, and register the best model. Which solution meets these requirements with the LEAST operational overhead?** Choose one.

- **a.** Store each run's parameters and metrics as items in Amazon DynamoDB behind an Amazon QuickSight dashboard
- **b.** Log each run's parameters, metrics and artifacts to a managed MLflow App in SageMaker AI
- **c.** Run the open-source MLflow tracking server on Amazon EC2 with an Amazon RDS backend store and S3 artifacts
- **d.** Write each run's parameters and metrics to Amazon CloudWatch Logs and query them with Logs Insights

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ This stores values but means building and maintaining a custom tracking schema and dashboards; it is not open-source compatible and has no model registration.
- **b** ✅ Managed MLflow on SageMaker AI records parameters, metrics and artifacts per run, compares runs in the MLflow UI and can register models into SageMaker Model Registry, while AWS runs, patches and scales the tracking infrastructure.
- **c** ❌ This meets every functional requirement, but the team must provision, patch, secure, back up and scale the server and database itself, which is far more operational work than the managed MLflow App.
- **d** ❌ CloudWatch Logs can hold the values, but it has no notion of runs, artifacts, run comparison or model registration, so the team would be building and maintaining an experiment tracker by hand.

**The concept.** Managed MLflow on SageMaker AI is the AWS-managed experiment tracker for reproducible ML work.

**Why this is correct.** Reproducibility needs every run's inputs (parameters, data, code) and outputs (metrics, artifacts) recorded in one place where runs can be compared. Both open-source MLflow on EC2 and managed MLflow on SageMaker AI give that with the MLflow APIs, so the qualifier decides: the managed MLflow App removes the server, database and patching work, and integrates with SageMaker Model Registry. CloudWatch Logs or DynamoDB with QuickSight store fragments and leave the team to build comparison and registration themselves.

**How to reason it out**

1. Identify the need: track, compare and register experiment runs reproducibly with an open-source-compatible tool.
2. Notice that self-hosted MLflow and managed MLflow both meet the functional requirements.
3. Apply LEAST operational overhead: choose managed MLflow on SageMaker AI, where AWS runs the tracking infrastructure.

> **Exam tip:** MLflow experiment tracking with the least operations work on AWS: the managed MLflow App in SageMaker AI, not a self-hosted server.

</details>

📖 Learn this topic: [Evaluating ML and GenAI Models: Metrics, Drift, Explainability and RAG on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-and-genai-model-evaluation-drift-explainability/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 11

*ML Model and Foundation Model (FM) Development*

**An HR team wants an internal assistant that answers employee questions about company policies. Policies are revised most weeks, and every answer has to show which policy document it came from so employees can check the source. The team wants to avoid retraining a model whenever a policy changes. Which approach should the ML engineer choose?** Choose one.

- **a.** Connect the policy documents to an Amazon Bedrock knowledge base
- **b.** Paste summaries of the current policies into a fixed system prompt
- **c.** Run continued pre-training on the policy corpus each quarter
- **d.** Fine-tune a foundation model on the policy documents each week

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Retrieval Augmented Generation fetches the current policy text at question time and returns citations; re-syncing the data source picks up revisions without retraining a model.
- **b** ❌ Summaries drift from the real documents and must be edited by hand after each revision; a fixed prompt also gives no per-answer source document.
- **c** ❌ Quarterly training would answer from policies that are weeks out of date, and continued pre-training produces no citations.
- **d** ❌ Weekly fine-tuning is the retraining the team wants to avoid, and a fine-tuned model does not return the source document for its answers.

**The concept.** RAG grounds a model in documents retrieved at query time, which suits frequently changing knowledge and answers that need citations.

**Why this is correct.** Two requirements decide it: knowledge that changes weekly and answers that cite their source. A Bedrock knowledge base retrieves the relevant policy chunks for each question and returns citations, and a data source sync keeps it current with no model training. Fine-tuning and continued pre-training both bake a snapshot of the policies into the weights, go stale between runs and cannot point to a source. Hand-maintained prompt summaries are fragile and uncited.

**How to reason it out**

1. Note that the facts change frequently.
2. Note that answers need a traceable source.
3. Recall that RAG retrieves current documents at query time and returns citations.
4. Reject options that store knowledge in model weights.

> **Exam tip:** Changing facts plus citations points to RAG, not fine-tuning.

</details>

📖 Learn this topic: [Choosing ML, Foundation Model and RAG Approaches on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-modeling-approaches-fm-selection-rag/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 12

*Deployment and Orchestration of ML and AI Workflows*

**An insurer retrains a claims-risk model in SageMaker AI every month. Each night, about 20 million new claim records land in Amazon S3 as CSV files, and every record must be scored before analysts arrive at 8 AM. No application calls the model during the day. Which solution will meet these requirements MOST cost-effectively?** Choose one.

- **a.** Deploy a real-time endpoint and use a scheduled scaling action to scale it in each day
- **b.** Invoke a serverless endpoint once for each claim record
- **c.** Host the model on an asynchronous endpoint that scales in to zero instances
- **d.** Run a SageMaker AI batch transform job over the nightly S3 prefix

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Even with scheduled scaling, the endpoint bills while it is up, and the team must drive 20 million requests through it one call at a time. Batch transform bills only for the job and reads S3 directly.
- **b** ❌ Serverless scales to zero, but 20 million individual synchronous calls are slow and expensive compared with one offline job reading S3 directly.
- **c** ❌ Asynchronous inference suits individual large or long-running requests arriving over time; queuing millions of tiny records through it is the wrong fit for a bulk nightly dataset.
- **d** ✅ Batch transform provisions instances only for the job, scores the whole S3 dataset offline, writes results back to S3 and releases the compute when it finishes, so nothing is billed between runs. That makes it the cheapest option that meets the deadline.

**The concept.** Batch transform is SageMaker AI's offline inference option for scoring whole datasets in S3.

**Why this is correct.** The workload is a large dataset scored on a schedule with no interactive caller. Several options could score it: a real-time endpoint bills while it is up and needs 20 million requests driven through it, and a serverless endpoint called 20 million times pays per request and is slow. Batch transform reads the input from S3, runs instances for the job only, writes predictions to S3 and stops, so it is the most cost-effective way to meet the 8 AM deadline.

**How to reason it out**

1. Notice that no application needs an answer in real time.
2. Notice the input is a complete dataset already in S3.
3. Compare cost: endpoints bill while they are up or per request; batch transform bills only for the job.
4. Choose batch transform as the cheapest option that meets the deadline.

> **Exam tip:** Whole dataset in S3, no live caller: batch transform, not an endpoint.

</details>

📖 Learn this topic: [Deploying ML Models, Foundation Models and Agents on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-bedrock-model-deployment-inference-strategies/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 13

*Deployment and Orchestration of ML and AI Workflows*

**A logistics company has a 1.2 GB scikit-learn model that estimates delivery times. An internal web app calls it a few hundred times a day in short bursts, with idle gaps of several hours, and waits for the prediction in the same HTTP request. A few seconds of extra latency on the first call after an idle gap is acceptable. Which deployment will meet these requirements MOST cost-effectively?** Choose one.

- **a.** A real-time endpoint on one ml.m5.large instance with target tracking
- **b.** A batch transform job started by the web app for each incoming request
- **c.** A serverless inference endpoint with 3,072 MB of memory configured
- **d.** An asynchronous inference endpoint with a minimum instance count of zero

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A real-time endpoint answers synchronously and meets the requirements, but it keeps at least one instance running through the hours of idle time, so the company pays for idle capacity and it costs more than serverless inference.
- **b** ❌ Batch transform starts instances for every job, which takes minutes, and returns results to S3, so it cannot serve a synchronous web request.
- **c** ✅ Serverless inference scales to zero between bursts, bills only while processing, answers synchronously, and the tolerated cold start is exactly its trade-off. Memory must be at least the model size, so 3 GB fits a 1.2 GB model.
- **d** ❌ Asynchronous inference also scales to zero, but the caller gets back an S3 output location rather than the prediction, which breaks the requirement that the web app waits for the answer in the same request.

**The concept.** Serverless inference is the synchronous option that scales to zero and accepts cold starts.

**Why this is correct.** Three facts decide it: the caller needs a synchronous answer, traffic is intermittent with long idle gaps, and cold-start latency is acceptable. A real-time endpoint would also work, but it bills for an instance through every idle gap. Serverless inference answers synchronously, scales to zero and bills only for processing time, so it is the cheaper option; the tolerated cold start is its trade-off. Asynchronous inference also scales to zero but returns results through S3, which breaks the synchronous requirement.

**How to reason it out**

1. Check whether the caller waits for the answer in-line (synchronous).
2. Check whether traffic has long idle periods and cold starts are tolerated.
3. Check the model fits within serverless memory limits (it is CPU-only and up to 6 GB).
4. Of the synchronous options, choose serverless inference because it does not bill while idle.

> **Exam tip:** Synchronous, spiky, cold starts OK, CPU model: serverless inference.

</details>

📖 Learn this topic: [Deploying ML Models, Foundation Models and Agents on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-bedrock-model-deployment-inference-strategies/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 14

*Deployment and Orchestration of ML and AI Workflows*

**A retailer generates product descriptions with an Amazon Bedrock base model while sellers edit listings, so responses must return interactively. Traffic is high and steady around the clock, and peak-hour requests are now being throttled. The company will commit to this workload for at least six months and wants model capacity that other workloads in the account or Region cannot consume. What should an ML engineer do?** Choose one.

- **a.** Submit the requests as Amazon Bedrock batch inference jobs and read the generated text from Amazon S3.
- **b.** Invoke the model through a cross-Region inference profile and keep paying per token with on-demand pricing.
- **c.** Request a higher on-demand tokens-per-minute quota for the model and add exponential backoff retries in the client code.
- **d.** Purchase Provisioned Throughput model units for the model, and send the requests to the provisioned model ARN.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Batch inference is asynchronous and returns results later in S3, which breaks the requirement for interactive responses.
- **b** ❌ A cross-Region inference profile spreads on-demand traffic across Regions to absorb bursts, but it does not reserve capacity for the workload.
- **c** ❌ A quota increase raises the on-demand ceiling but still draws from shared capacity; retries hide throttling rather than reserving throughput.
- **d** ✅ Provisioned Throughput buys model units dedicated to the account at a fixed rate for a commitment term (no-commitment, one-month or six-month), which matches steady high volume that needs dedicated capacity.

**The concept.** Amazon Bedrock Provisioned Throughput reserves dedicated model capacity for a commitment term; on-demand is pay-per-token on shared capacity.

**Why this is correct.** The deciding facts are steady high volume, a multi-month commitment and a need for capacity no other workload can consume. Provisioned Throughput buys model units for a commitment term and is invoked through the provisioned model ARN. Cross-Region inference and quota increases improve on-demand headroom but dedicate nothing, and batch inference cannot serve interactive requests.

**How to reason it out**

1. Note the traffic shape: steady and high, not spiky.
2. Note the commitment term and the need for reserved capacity.
3. Rule out batch because responses must be interactive.
4. Choose Provisioned Throughput and call the provisioned model ARN.

> **Exam tip:** Steady, committed, high-volume Bedrock traffic that needs dedicated capacity points to Provisioned Throughput.

</details>

📖 Learn this topic: [Provisioning ML Resources: Endpoints, Scaling, Knowledge Bases and Agents (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-endpoint-scaling-vpc-bedrock-knowledge-bases/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 15

*Deployment and Orchestration of ML and AI Workflows*

**A payments company serves a fraud-scoring model from a SageMaker AI real-time endpoint that runs on eight instances. A new model version must reach production with a limited blast radius: about 10% of requests should go to the new fleet for 30 minutes, and if no CloudWatch alarm fires during that time, the remaining traffic should move in a single step. Which update configuration should the ML engineer use?** Choose one.

- **a.** Rolling update with a maximum batch size of 10% of instances, a 30-minute wait interval, and CloudWatch alarms for auto-rollback
- **b.** Blue/green update with linear traffic shifting, a 30-minute baking period, and CloudWatch alarms for auto-rollback
- **c.** Blue/green update with canary traffic shifting, a 30-minute baking period, and CloudWatch alarms for auto-rollback
- **d.** Blue/green update that shifts 100% of traffic in one step, a 30-minute baking period, and CloudWatch alarms for auto-rollback

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A rolling update replaces capacity batch by batch across the whole fleet; it is meant for large fleets and does not hold a fixed 10% traffic slice before shifting the rest.
- **b** ❌ Linear moves traffic in several equally sized steps with a bake after each, not one small slice followed by everything else.
- **c** ✅ Canary shifts a small slice first, waits for the baking period while watching the alarms, then shifts the remainder in one step, which is exactly the two-step pattern described.
- **d** ❌ A one-step shift sends 100% of requests to the new fleet immediately, so a regression would hit every request rather than a 10% slice.

**The concept.** SageMaker AI deployment guardrails offer three blue/green traffic-shifting modes: all at once, canary and linear.

**Why this is correct.** Canary is a two-step shift: a small canary portion first, a baking period during which the configured CloudWatch alarms are watched, then the rest of the traffic at once. Linear uses several equal steps, all at once has no partial step, and a rolling update replaces instances in batches rather than holding a traffic slice.

**How to reason it out**

1. Count the traffic steps the requirement describes: one small slice, then everything else.
2. Map two steps to canary, several equal steps to linear and a single step to all at once.
3. Keep the alarms and baking period so a regression in the canary slice triggers an automatic rollback.

> **Exam tip:** Small slice, bake, then the rest = canary; equal repeated steps = linear.

</details>

📖 Learn this topic: [MLOps CI/CD Pipelines, Model Versioning and Rollback on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-mlops-cicd-pipelines-model-versioning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 16

*Deployment and Orchestration of ML and AI Workflows*

**A radiology startup runs a GPU-based segmentation model in SageMaker AI. Each request is a 300 MB scan, and inference takes 6 to 9 minutes per scan. Scans arrive unpredictably, sometimes none for a whole day. Clinicians need results within 20 minutes of upload and want a notification when a result is ready. The startup does not want to pay for GPU instances while no scans are waiting. Which deployment meets these requirements?** Choose one.

- **a.** A real-time endpoint on a GPU instance with auto scaling and a 15-minute model timeout
- **b.** A serverless endpoint with 6,144 MB of memory and provisioned concurrency set to zero
- **c.** An asynchronous endpoint on a GPU instance that scales to zero and publishes to Amazon SNS
- **d.** A batch transform job on a GPU instance that an hourly Amazon EventBridge schedule starts

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Real-time endpoints accept small payloads and enforce a short invocation timeout, so 300 MB scans and multi-minute inference do not fit, and the endpoint keeps at least one GPU instance running.
- **b** ❌ Serverless inference does not support GPUs, and its payload size and processing time limits are far below a 300 MB scan that takes minutes to segment.
- **c** ✅ Asynchronous inference takes payloads up to 1 GB from S3, allows processing up to one hour, can scale its instance count to zero when the queue is empty, and sends success or error notifications through Amazon SNS.
- **d** ❌ Batch transform handles large inputs and releases compute afterwards, but an hourly schedule plus job start-up time can leave a scan waiting well beyond the 20-minute requirement.

**The concept.** Asynchronous inference queues large, long-running requests and can scale to zero.

**Why this is correct.** The deciding facts combine: payload size (300 MB) and processing time (minutes) rule out real-time and serverless inference; the need to pay nothing while idle rules out an always-on endpoint; and the 20-minute deadline rules out a scheduled batch job. Asynchronous inference accepts requests via S3 up to 1 GB, processes for up to an hour, scales to zero instances when nothing is queued and notifies through SNS.

**How to reason it out**

1. Compare the payload and processing time with each option's limits.
2. Eliminate options that cannot use a GPU or that keep an instance running.
3. Check the turnaround deadline against scheduled batch jobs.
4. Choose asynchronous inference with scale-to-zero and SNS notifications.

> **Exam tip:** Large payloads, minutes per request, near-real-time results, idle periods: asynchronous inference.

</details>

📖 Learn this topic: [Deploying ML Models, Foundation Models and Agents on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-bedrock-model-deployment-inference-strategies/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 17

*Operating, Monitoring, and Securing ML and AI Solutions*

**A company runs a customer-support assistant on Amazon Bedrock. The operations lead wants a view of invocation counts, token usage by model, latency percentiles and throttles. The lead also wants to open any single request and read the prompt and the model's response. The team is small and wants to avoid building or maintaining monitoring components. Which solution will meet these requirements with the LEAST operational overhead?** Choose one.

- **a.** Enable Bedrock model invocation logging with an Amazon S3 bucket as its sole destination, and use the Model Invocations view in CloudWatch generative AI observability.
- **b.** Enable Bedrock model invocation logging to a CloudWatch Logs log group, and use the Model Invocations view in CloudWatch generative AI observability.
- **c.** Turn on AWS CloudTrail data events for Amazon Bedrock, and build a CloudWatch dashboard from the trail's log group with Logs Insights widgets.
- **d.** Enable Bedrock model invocation logging to CloudWatch Logs, and build a CloudWatch dashboard with AWS/Bedrock metric widgets and a Logs Insights table of each prompt and response.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ The metric widgets would work, but the request-level table reads invocation logs from CloudWatch Logs. With an S3-only destination the prompts and responses are not available in that view.
- **b** ✅ The Model Invocations view is prebuilt (invocations, tokens, latency, throttles, errors) and, once invocation logs go to CloudWatch Logs, lists each request with its input and output, so nothing has to be built or maintained.
- **c** ❌ CloudTrail records who called which API, not the prompt and response bodies, and this option means building and maintaining a custom dashboard.
- **d** ❌ This works: the metric widgets cover counts, tokens, latency and throttles, and the Logs Insights widget shows request content. But the team must build and maintain every widget and query, which the prebuilt view already provides.

**The concept.** CloudWatch generative AI observability provides a prebuilt Model Invocations view for Amazon Bedrock; request-level inputs and outputs appear only when model invocation logging is sent to CloudWatch Logs.

**Why this is correct.** Two needs must be met (metrics plus per-request prompt/response detail) and the qualifier decides between the workable solutions: least operational overhead. A hand-built CloudWatch dashboard with AWS/Bedrock metric widgets and a Logs Insights table over the invocation log group would work, but every widget and query must be built and maintained. The Model Invocations view in CloudWatch generative AI observability covers the metrics out of the box, and its invocation table is populated from Bedrock model invocation logs delivered to CloudWatch Logs. Logging only to S3 leaves that table empty, and CloudTrail never records payloads.

**How to reason it out**

1. List the requirements: invocation metrics and per-request content.
2. Remember that request content comes from model invocation logging, which is off by default, delivered to CloudWatch Logs for the view to read it.
3. Compare the workable solutions on the qualifier: a prebuilt GenAI observability view beats a custom dashboard you must build and maintain.
4. Reject options that cannot show prompts and responses at all (S3-only destination for the view, CloudTrail).

> **Exam tip:** Prebuilt Bedrock invocation dashboards come from CloudWatch generative AI observability; prompts and responses need invocation logging to CloudWatch Logs.

</details>

📖 Learn this topic: [Monitoring ML Models, FMs and Agents in Production: Drift, A/B Tests and Bedrock Evaluations (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-model-monitoring-drift-detection-bedrock-evaluations/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 18

*Operating, Monitoring, and Securing ML and AI Solutions*

**A legal-research application sends long contracts to a model on Amazon Bedrock. Model invocation logging delivers to a CloudWatch Logs log group, and auditors use Logs Insights to review what the model was given. For the largest contracts, the log events contain the metadata and token counts but not the request body. The auditors need the full input for every request. What should the ML engineer change?** Choose one.

- **a.** Add an Amazon S3 location for large data delivery to the model invocation logging configuration.
- **b.** Move the log group to the Infrequent Access log class so that larger log events can be stored at a lower cost.
- **c.** Turn on AWS CloudTrail data events for the Bedrock InvokeModel operation so that the request bodies are recorded.
- **d.** Select the Embedding data type in the model invocation logging settings so that long text inputs are logged as vectors.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Bodies larger than 100 KB are not placed inline in the log event; with an S3 large-data location they are written to S3 and the log entry references them.
- **b** ❌ The log class changes price and features, not the size of the body that Bedrock places inline in an event.
- **c** ❌ CloudTrail records the API call and caller, not the prompt body, so the auditors still would not see the contract text.
- **d** ❌ Embedding logging captures embedding-model data; it does not change how a large text request body is delivered.

**The concept.** Bedrock model invocation logging keeps input and output bodies up to 100 KB inline; larger bodies and binary data go to an S3 large-data location.

**Why this is correct.** The symptom (metadata present, body missing only for the biggest requests) points at the inline size limit. Invocation log events hold JSON bodies up to 100 KB; anything larger is only kept when an S3 location for large data delivery is configured, and the log entry then points at the S3 object. Log class, embedding logging and CloudTrail do not capture the missing body.

**How to reason it out**

1. Notice that only the largest requests lose their body.
2. Recall that invocation logs inline bodies up to 100 KB.
3. Recall that larger bodies need an S3 large-data delivery location.
4. Rule out CloudTrail, which never records prompt content.

> **Exam tip:** Large or binary Bedrock invocation payloads are logged only when an S3 large-data location is configured.

</details>

📖 Learn this topic: [Monitoring ML Models, FMs and Agents in Production: Drift, A/B Tests and Bedrock Evaluations (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-model-monitoring-drift-detection-bedrock-evaluations/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 19

*Operating, Monitoring, and Securing ML and AI Solutions*

**A retailer has registered a new XGBoost demand model in SageMaker Model Registry and will host it on a SageMaker AI real-time endpoint. Before going live, the ML engineer wants load-tested recommendations that compare candidate ml instance types on latency, throughput and cost per inference. The team does not want to build its own benchmarking harness. What should the ML engineer use?** Choose one.

- **a.** Run an Amazon SageMaker Inference Recommender job against the registered model package version in the registry.
- **b.** Run an Amazon SageMaker Model Monitor data quality job on inference data captured from the registered model's endpoint.
- **c.** Generate an Amazon SageMaker Debugger profiling report for the training job that produced the registered model.
- **d.** Open the AWS Cost Explorer rightsizing recommendations for the inference instances that run in the account.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Inference Recommender deploys the model package to candidate instance types, load tests them and reports latency, throughput and cost per inference and per hour.
- **b** ❌ Model Monitor compares captured inference data with a baseline to detect drift. It does not benchmark instance types or report cost per inference.
- **c** ❌ Debugger profiles resource use during training. Training-time utilisation says nothing about the latency or cost of serving on different instance types.
- **d** ❌ Cost Explorer rightsizing recommendations analyse running EC2 instances. They do not load test a model package on ml instance types before deployment.

**The concept.** SageMaker Inference Recommender load tests a model across instance types and reports performance and cost metrics, so instance selection is evidence-based.

**Why this is correct.** The need is a pre-deployment comparison of instance types with cost and latency figures, without custom tooling. Inference Recommender does exactly this for a model package in the registry. Model Monitor is about drift after deployment, Cost Explorer rightsizing looks at existing EC2 usage, and Debugger reports on training rather than serving.

**How to reason it out**

1. Identify the goal: choose an inference instance type before deployment.
2. Note the constraint: no custom benchmarking harness.
3. Match the managed service that load tests a model package on candidate instance types.
4. Discard tools that monitor drift, training or existing EC2 usage.

> **Exam tip:** To choose an inference instance type with data, run SageMaker Inference Recommender on the model package.

</details>

📖 Learn this topic: [Optimising ML and GenAI Inference Cost and Performance on AWS: Instances, Savings Plans, Tokens and Vectors (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-genai-inference-cost-optimization/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

## Question 20

*Operating, Monitoring, and Securing ML and AI Solutions*

**An ML team builds custom SageMaker AI inference containers and pushes them to Amazon ECR. Images that were clean at build time often stay in production for months. The security team wants every image in the repositories re-evaluated when a newly published CVE affects its packages. Which solution will meet these requirements with the LEAST operational overhead?** Choose one.

- **a.** Turn on Amazon Inspector enhanced scanning for the registry, with continuous scanning for the ML repositories.
- **b.** Turn on AWS Config recording for the ECR repositories and evaluate them with the image scanning managed rule.
- **c.** Turn on Amazon ECR basic scanning with scan on push for the ML repositories, and make the image tags immutable.
- **d.** Schedule a nightly CodeBuild job that generates an SBOM of each stored image and scans it with the Inspector Scan API.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Enhanced scanning with continuous scanning re-evaluates stored images when new vulnerabilities are published, with nothing for the team to schedule or maintain.
- **b** ❌ A Config rule checks whether scanning is enabled on a repository; it does not scan images or find CVEs itself.
- **c** ❌ Basic scanning runs at push or on a manual request; an image that sits unchanged in the repository is not re-scanned when a new CVE appears.
- **d** ❌ This re-evaluates stored images, but the team must build, schedule and maintain the job, and a new CVE is only caught at the next nightly run.

**The concept.** Amazon Inspector enhanced scanning for Amazon ECR scans on push and, in continuous mode, re-scans stored images when new CVEs are published.

**Why this is correct.** Two options re-evaluate old images: a nightly SBOM-and-scan job that the team builds and runs, and Inspector enhanced scanning in continuous mode. The qualifier decides it: continuous scanning is a registry setting that re-scans stored images automatically when new CVE data is published, with no job to maintain. Basic scan on push never looks at an unchanged image again, and an AWS Config rule reports configuration, not vulnerabilities.

**How to reason it out**

1. Note that the images are old and not being rebuilt.
2. Find the options that actually re-evaluate stored images.
3. Compare their effort: a managed registry setting versus a scheduled job the team owns.
4. Pick Inspector enhanced scanning in continuous mode.

> **Exam tip:** Old images in ECR need Inspector enhanced, continuous scanning; a scheduled rescan job works but is extra machinery to run.

</details>

📖 Learn this topic: [Securing ML and AI Workloads on AWS: IAM, VPC Isolation, Auditing and Bedrock Guardrails (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/secure-ml-ai-workloads-iam-vpc-bedrock-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

---

[← Back to the MLA-C02 study guide](README.md)
