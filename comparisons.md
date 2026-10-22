# MLA-C02 Commonly Confused Services and Concepts

Side-by-side comparisons of the MLA-C02 services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Data Preparation for ML and AI](#domain-1-data-preparation-for-ml-and-ai)
- [Domain 2: ML Model and Foundation Model (FM) Development](#domain-2-ml-model-and-foundation-model-fm-development)
- [Domain 3: Deployment and Orchestration of ML and AI Workflows](#domain-3-deployment-and-orchestration-of-ml-and-ai-workflows)
- [Domain 4: Operating, Monitoring, and Securing ML and AI Solutions](#domain-4-operating-monitoring-and-securing-ml-and-ai-solutions)

## Domain 1: Data Preparation for ML and AI

### File vs FastFile vs Pipe input mode

|                  | File mode (default)                               | FastFile mode                                  | Pipe mode                          |
| ---------------- | ------------------------------------------------- | ---------------------------------------------- | ---------------------------------- |
| How data arrives | Whole channel downloaded before the script starts | Objects streamed as the script reads them      | Streamed through a Unix named pipe |
| Script change    | None (local files)                                | None (POSIX file paths)                        | Must read from the pipe            |
| Local storage    | Must hold the full channel                        | Small                                          | Small                              |
| Exam cue         | "small dataset, random access"                    | "large dataset, slow start-up, no code change" | "existing pipe-reading code"       |

**Remember:** FastFile is the usual fix for a slow start or a full disk without touching code. None of the three fixes millions of tiny objects; packing them into shards does.

### Kinesis Data Streams vs Data Firehose vs MSK vs Managed Flink

|               | Kinesis Data Streams                         | Data Firehose                            | Amazon MSK                    | Managed Service for Apache Flink           |
| ------------- | -------------------------------------------- | ---------------------------------------- | ----------------------------- | ------------------------------------------ |
| What it is    | Durable, ordered, replayable shards          | Managed delivery to S3 and other targets | Managed Apache Kafka          | Managed stateful stream processing         |
| Consumer code | Yes                                          | No                                       | Yes (Kafka clients)           | Your Flink application                     |
| Replay        | Yes, 24 hours default up to 365 days         | No                                       | Yes                           | No (it reads a stream)                     |
| Exam cue      | "several applications", "replay after a bug" | "land in S3 with no code"                | "producers already use Kafka" | "sliding windows", "joins", "exactly-once" |

**Remember:** Firehose delivers, Data Streams retains and fans out, MSK is for Kafka, Flink computes.

### Enhanced fan-out vs retention period

|          | Enhanced fan-out                                     | Retention period                           |
| -------- | ---------------------------------------------------- | ------------------------------------------ |
| Solves   | Consumers competing for 2 MB/s per shard, latency    | Re-reading data after a failure or bug fix |
| Effect   | Dedicated 2 MB/s per consumer per shard, about 70 ms | Records kept from 24 hours up to 365 days  |
| Exam cue | "new consumer slows the others down"                 | "reprocess the last week"                  |

**Remember:** they are independent settings. A scenario that needs both needs both.

### S3 Vectors vs OpenSearch Serverless vs pgvector

|              | Amazon S3 Vectors                         | OpenSearch Serverless (vector search)   | Aurora/RDS PostgreSQL + pgvector               |
| ------------ | ----------------------------------------- | --------------------------------------- | ---------------------------------------------- |
| Strength     | Lowest cost, no infrastructure            | Millisecond latency at high query rates | Vectors in the same tables as business data    |
| Search types | Similarity only                           | Vector, lexical and hybrid              | SQL plus similarity in one query               |
| Freshness    | Copy of the data, kept in sync            | Copy of the data, kept in sync          | Reflects committed rows immediately            |
| Exam cue     | "billions of vectors, infrequent queries" | "production RAG, keyword plus semantic" | "already on PostgreSQL", "join with inventory" |

**Remember:** cost and scale point to S3 Vectors, query rate and hybrid search to OpenSearch, relational freshness to pgvector.

### HNSW vs IVFFlat

|               | HNSW                                 | IVFFlat                                         |
| ------------- | ------------------------------------ | ----------------------------------------------- |
| How it works  | Graph of neighbours                  | Clusters vectors into lists                     |
| Training step | None; can be built on an empty table | Needs representative data before building       |
| Trade-off     | Strong speed-to-recall, more memory  | Faster build, less memory, usually lower recall |

**Remember:** HNSW is the default choice for recall and for indexes built before data arrives.

### Glue vs DataBrew vs EMR vs Data Wrangler

|                   | AWS Glue ETL              | Glue DataBrew         | Spark on EMR                      | SageMaker Data Wrangler              |
| ----------------- | ------------------------- | --------------------- | --------------------------------- | ------------------------------------ |
| Who builds        | Engineers writing code    | Analysts, no code     | Engineers who tune Spark          | Data scientists                      |
| Infrastructure    | Serverless                | Serverless            | Clusters you configure            | Managed; exports run as jobs         |
| Distinctive lever | Job bookmarks, Flex class | Recipes, profile jobs | Native libraries, Spot task nodes | Export to Pipelines or Feature Store |

**Remember:** read the scenario for who builds it and how much infrastructure they accept; all four read and write S3.

### Online store vs offline store (SageMaker Feature Store)

|           | Online store                             | Offline store                             |
| --------- | ---------------------------------------- | ----------------------------------------- |
| Holds     | Latest record per record identifier      | Every version, append-only in S3          |
| Read with | GetRecord / BatchGetRecord, milliseconds | Athena or Spark via the Glue Data Catalog |
| Used for  | Real-time inference lookups              | Point-in-time correct training sets       |
| TTL       | Supported                                | Not applied; history kept                 |

**Remember:** serving reads the online store, training reads the offline store with a point-in-time join on event time.

### Standardization vs min-max vs robust scaling

|          | Standardization    | Min-max normalization      | Robust scaling              |
| -------- | ------------------ | -------------------------- | --------------------------- |
| Formula  | (x − mean) / std   | (x − min) / (max − min)    | (x − median) / IQR          |
| Result   | Mean 0, variance 1 | Bounded range such as 0–1  | Outliers no longer dominate |
| Use when | Default            | Bounded range, no outliers | Real outliers must stay     |

**Remember:** fit any scaler on the training split only. Tree models rarely need scaling.

### Fine-tuning vs continued pre-training vs distillation

|          | Fine-tuning                     | Continued pre-training    | Distillation                                 |
| -------- | ------------------------------- | ------------------------- | -------------------------------------------- |
| Data     | Labeled prompt-completion pairs | Unlabeled domain text     | Prompts only, plus a teacher model           |
| Teaches  | A task or behaviour             | Vocabulary and style      | A smaller model to imitate a larger one      |
| Exam cue | "thousands of approved answers" | "large corpus, no labels" | "cheaper, faster model with similar quality" |

**Remember:** what data you have decides the method.

### Class imbalance (CI) vs difference in proportions of labels (DPL)

|          | CI                              | DPL                                          |
| -------- | ------------------------------- | -------------------------------------------- |
| Measures | How many rows each facet has    | How often each facet gets the positive label |
| Formula  | (n(a) − n(d)) / (n(a) + n(d))   | q(a) − q(d)                                  |
| Exam cue | "one group is rare in the data" | "approval rate differs by group"             |

**Remember:** a dataset can be badly imbalanced on CI with DPL near zero, and the reverse.

### Glue Data Quality vs DataBrew rulesets

|                | AWS Glue Data Quality                        | Glue DataBrew ruleset                       |
| -------------- | -------------------------------------------- | ------------------------------------------- |
| Runs in        | Data Catalog runs, or an ETL job transform   | A profile job with the ruleset attached     |
| Stops bad rows | Yes, in an ETL job (row-level results)       | No; reports after the fact                  |
| Result signal  | Rule outcomes and score                      | EventBridge Ruleset Validation Result event |
| Exam cue       | "quarantine failing rows", "recommend rules" | "analysts, no code"                         |

**Remember:** only the ETL transform can quarantine rows in the same run. A DataBrew job's SUCCEEDED state does not mean the data passed.

## Domain 2: ML Model and Foundation Model (FM) Development

### RAG vs fine-tuning

|                       | RAG (for example a Bedrock knowledge base)   | Fine-tuning                                |
| --------------------- | -------------------------------------------- | ------------------------------------------ |
| Fixes                 | Missing or changing knowledge                | Behaviour: tone, layout, label conventions |
| Changing facts        | Re-sync the data source, no training         | Each run bakes in a snapshot               |
| Citations             | Yes                                          | No                                         |
| Per-user restrictions | Metadata filtering at query time             | No                                         |
| Exam cue              | "policies change monthly", "cite the clause" | "always use our report layout"             |

**Remember:** RAG for knowledge, fine-tuning for behaviour. When a scenario needs both current facts and a fixed style, use both.

### Supervised fine-tuning vs reinforcement fine-tuning vs distillation vs continued pre-training

|            | Supervised fine-tuning        | Reinforcement fine-tuning      | Distillation                  | Continued pre-training  |
| ---------- | ----------------------------- | ------------------------------ | ----------------------------- | ----------------------- |
| Input      | Labeled prompt-response pairs | Prompts plus a reward function | Prompts plus a strong teacher | Unlabeled domain text   |
| Changes    | Task behaviour                | Outputs that can be graded     | A smaller, cheaper student    | Vocabulary and language |
| Fails when | No good examples              | No automatic grader            | No teacher does the task well | You need a task learned |

**Remember:** ask what data and signal you have before asking which method is best.

### LoRA vs full fine-tuning

|                       | LoRA (PEFT)                                           | Full fine-tuning           |
| --------------------- | ----------------------------------------------------- | -------------------------- |
| What trains           | Small adapters; base frozen                           | Every weight               |
| Compute and memory    | Low                                                   | High                       |
| Artifact per variant  | Small adapter                                         | Full model copy            |
| Hosting many variants | Share one base on one endpoint (inference components) | Each needs its own hosting |
| Forgetting risk       | Lower                                                 | Higher                     |

**Remember:** many low-traffic variants point to LoRA adapters served on one shared base model.

### Data parallelism vs model parallelism

|             | Data parallelism (SMDDP)               | Model / sharded data parallelism (SMP v2)           |
| ----------- | -------------------------------------- | --------------------------------------------------- |
| What splits | Batches; full model copy on each GPU   | Parameters, gradients, optimizer state              |
| Use when    | Model fits on one GPU, data is large   | Model state does not fit on one GPU even at batch 1 |
| Wrong use   | Does not fix model-state out-of-memory | Adds overhead for a model that already fits         |

**Remember:** slow with a model that fits means data parallelism; out of memory because of the model means model parallelism.

### Bayesian vs random vs grid vs Hyperband tuning

|                          | Bayesian (default)               | Random              | Grid                        | Hyperband                       |
| ------------------------ | -------------------------------- | ------------------- | --------------------------- | ------------------------------- |
| Learns from earlier jobs | Yes                              | No                  | No                          | Uses intermediate metrics       |
| Best for                 | Limited budget                   | Fully parallel runs | Small categorical spaces    | Long iterative training         |
| Watch for                | High parallelism makes it random | Ignores results     | Categorical parameters only | Early stopping type must be Off |

**Remember:** Bayesian needs sequential results to be smarter than random.

### IdenticalDataAndAlgorithm vs TransferLearning warm start

|                 | IdenticalDataAndAlgorithm | TransferLearning                                  |
| --------------- | ------------------------- | ------------------------------------------------- |
| Data            | Same as parent            | May differ                                        |
| Algorithm image | Same                      | May be a different version                        |
| Ranges          | May change                | May change                                        |
| Exam cue        | "rerun with wider ranges" | "the dataset has grown since the last tuning job" |

**Remember:** new data or a new algorithm version means TransferLearning.

### Overfitting vs underfitting vs catastrophic forgetting

|         | Overfitting                                     | Underfitting                                        | Catastrophic forgetting                                   |
| ------- | ----------------------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| Symptom | Training great, validation worse                | Both poor and close                                 | New narrow task good, general skills lost                 |
| Fix     | Regularize, simplify, more data, early stopping | More capacity, less regularization, better features | LoRA, mix general data, lower learning rate, fewer epochs |

**Remember:** read the gap between training and validation scores before choosing a fix.

### Bagging vs boosting vs stacking

|         | Bagging (random forest)              | Boosting (XGBoost)                       | Stacking                                       |
| ------- | ------------------------------------ | ---------------------------------------- | ---------------------------------------------- |
| How     | Parallel models on bootstrap samples | Sequential models fixing residual errors | Meta-model learns from out-of-fold predictions |
| Reduces | Variance                             | Bias                                     | Errors of diverse models                       |

**Remember:** unstable model → bagging; too simple → boosting; several strong but different models → stacking.

### Shadow variant vs production variant vs blue/green canary

|                      | Shadow variant                       | Weighted production variant   | Blue/green canary              |
| -------------------- | ------------------------------------ | ----------------------------- | ------------------------------ |
| Customers see output | Never                                | The weighted share            | Canary share, then everyone    |
| Purpose              | Compare on live traffic, zero impact | Measure business impact (A/B) | Roll out a chosen model safely |

**Remember:** "no customer may receive the new model's answers" means shadow testing.

### Data drift vs concept drift

|              | Data drift                                | Concept drift                                 |
| ------------ | ----------------------------------------- | --------------------------------------------- |
| What changes | Input distributions                       | The relationship between inputs and the label |
| Inputs look  | Different from training                   | Possibly identical                            |
| Detect with  | Input statistics vs the training baseline | Ground-truth labels joined to predictions     |

**Remember:** a flat input monitor does not prove the model is still right.

### Local SHAP vs mean absolute SHAP vs partial dependence

|          | Local SHAP                          | Mean absolute SHAP                  | Partial dependence plot                     |
| -------- | ----------------------------------- | ----------------------------------- | ------------------------------------------- |
| Answers  | Why this one prediction             | Which features matter most overall  | How predictions change as one feature moves |
| Exam cue | "explain this applicant's decision" | "rank drivers across the portfolio" | "what if we change the price"               |

**Remember:** a per-person question always needs local attributions.

### BLEU vs ROUGE vs BERTScore

|          | BLEU                          | ROUGE                                       | BERTScore                               |
| -------- | ----------------------------- | ------------------------------------------- | --------------------------------------- |
| Measures | n-gram precision vs reference | n-gram recall or longest common subsequence | Embedding similarity of tokens          |
| Typical  | Translation                   | Summarization                               | Answers that paraphrase the reference   |
| Weakness | Penalizes synonyms            | Penalizes paraphrase                        | Needs an embedding model and references |

**Remember:** if correct answers are routinely reworded, move from n-gram metrics to BERTScore or embedding similarity.

### Automatic vs LLM-as-a-judge vs human evaluation (Bedrock)

|          | Automatic                   | LLM-as-a-judge                            | Human-based                     |
| -------- | --------------------------- | ----------------------------------------- | ------------------------------- |
| Scorer   | Fixed algorithms            | An evaluator model                        | Your own work team              |
| Best for | Repeatable regression tests | Subjective quality at scale, custom rules | Expert or confidential judgment |

**Remember:** "repeatable, no model judgment" is automatic; "no human raters" is a judge; "only attorneys can judge" is human.

### Citation precision vs citation coverage vs faithfulness

|          | Citation precision                           | Citation coverage                 | Faithfulness                               |
| -------- | -------------------------------------------- | --------------------------------- | ------------------------------------------ |
| Asks     | Do the given citations support their claims? | How much of the answer is cited?  | Are claims grounded in the context at all? |
| Low when | Citations point at wrong passages            | Grounded claims carry no citation | The model adds unsupported claims          |

**Remember:** run a retrieve-only job first; if context scores are high and faithfulness is low, fix generation.

## Domain 3: Deployment and Orchestration of ML and AI Workflows

### Real-time vs serverless vs asynchronous inference vs batch transform

|                  | Real-time endpoint                   | Serverless inference            | Asynchronous inference                  | Batch transform               |
| ---------------- | ------------------------------------ | ------------------------------- | --------------------------------------- | ----------------------------- |
| Response         | Synchronous                          | Synchronous                     | Result written to S3                    | Job output in S3              |
| Idle cost        | Pays for instances                   | None                            | None when scaled to zero                | None between jobs             |
| GPUs             | Yes                                  | No                              | Yes                                     | Yes                           |
| Payload and time | Small, fast                          | Small, fast                     | Up to 1 GB, up to about an hour         | Whole datasets                |
| Exam cue         | "steady traffic, consistent latency" | "idle for hours, cold start OK" | "large video files, minutes to process" | "score the table every night" |

**Remember:** synchronous with idle gaps is serverless; large or slow with scale-to-zero is async; a fixed dataset on a schedule is batch.

### Multi-model vs multi-container vs serial pipeline vs inference components

|             | Multi-model endpoint             | Multi-container (Direct)             | Serial inference pipeline          | Inference components                    |
| ----------- | -------------------------------- | ------------------------------------ | ---------------------------------- | --------------------------------------- |
| Containers  | One shared                       | Up to 15 different                   | 2–15 chained                       | One per component                       |
| Per request | One model, chosen by TargetModel | One container, chosen by hostname    | All containers in order            | One component, chosen by name           |
| Resources   | Shared memory, models evicted    | Shared instances                     | Same instance                      | Declared accelerators and memory each   |
| Best for    | Many similar, rarely used models | A few models on different frameworks | Preprocess → predict → postprocess | Several FMs sharing GPUs, scale to zero |

**Remember:** same framework and many models is MME; different frameworks called separately is multi-container; chained steps is a serial pipeline; guaranteed GPUs per model is inference components.

### Inferentia vs Trainium vs Graviton

|           | Inferentia2 (ml.inf2)                    | Trainium (ml.trn1/trn2)            | Graviton (ml.c7g/m7g)                  |
| --------- | ---------------------------------------- | ---------------------------------- | -------------------------------------- |
| Built for | Low-cost deep learning inference         | Training and fine-tuning           | General CPU price-performance          |
| Requires  | Model compiles with the Neuron SDK       | Neuron SDK                         | An arm64 container image               |
| Exam cue  | "lowest cost per inference, transformer" | "train or fine-tune at lower cost" | "tree model, better price-performance" |

**Remember:** "purpose-built for inference" is Inferentia, not Trainium.

### Bedrock on-demand vs Provisioned Throughput vs cross-Region inference vs batch inference

|            | On-demand                  | Provisioned Throughput                               | Cross-Region inference profile      | Batch inference                  |
| ---------- | -------------------------- | ---------------------------------------------------- | ----------------------------------- | -------------------------------- |
| Billing    | Per token                  | Model units, billed used or not                      | Per token                           | Discounted                       |
| Commitment | None                       | 1 or 6 months (no-commitment for some custom models) | None                                | None                             |
| Solves     | Variable traffic           | Reserved capacity for one workload                   | Peak throttling                     | Large offline jobs               |
| Exam cue   | "prototype, unpredictable" | "steady, high, interactive"                          | "throttled at peaks, no commitment" | "nobody waits for each response" |

**Remember:** only Provisioned Throughput reserves capacity. Global inference profiles leave the geography; geographic ones do not.

### Bedrock Agents (Classic) vs AgentCore Runtime

|                | Bedrock Agents (Classic)                            | AgentCore Runtime                                       |
| -------------- | --------------------------------------------------- | ------------------------------------------------------- |
| You provide    | Instructions, model, action groups, knowledge bases | Agent code in any framework, as an arm64 container      |
| Reasoning loop | Run by Bedrock                                      | Your code                                               |
| Availability   | Closed to new customers                             | The path for new agents                                 |
| Versioning     | Agent versions and aliases                          | Immutable runtime versions; DEFAULT vs pinned endpoints |

**Remember:** new agents and existing framework agents go to AgentCore.

### MCP vs A2A

|           | Model Context Protocol (MCP)                           | Agent2Agent (A2A)                         |
| --------- | ------------------------------------------------------ | ----------------------------------------- |
| Connects  | An agent to tools                                      | An agent to another agent                 |
| Use for   | Discovering and calling tools with input schemas       | Delegating a task to another team's agent |
| AWS piece | AgentCore Gateway turns APIs and Lambda into MCP tools | —                                         |

**Remember:** tools speak MCP, agents delegate over A2A.

### Retrieve vs RetrieveAndGenerate

|                     | Retrieve                                        | RetrieveAndGenerate                          |
| ------------------- | ----------------------------------------------- | -------------------------------------------- |
| Returns             | Chunks with metadata, sources and scores        | A cited answer from a Bedrock model          |
| Control             | Your own prompt, model, threshold               | Managed; you give up prompt control          |
| Query decomposition | Not available; split the question yourself      | Available in the orchestration configuration |
| Exam cue            | "generate with a model on a SageMaker endpoint" | "simplest managed RAG answer"                |

**Remember:** if anything must happen between retrieval and generation, use Retrieve.

### Gateway vs interface VPC endpoints

|          | Gateway endpoint                       | Interface endpoint (PrivateLink)                                   |
| -------- | -------------------------------------- | ------------------------------------------------------------------ |
| Services | Amazon S3 and DynamoDB only            | Everything else, including SageMaker AI and Bedrock                |
| How      | Route table entry                      | Network interfaces with private IPs; private DNS for SDK hostnames |
| Exam cue | "download model artifacts with no NAT" | "call InvokeEndpoint or bedrock-runtime privately"                 |

**Remember:** runtime and control-plane APIs are separate endpoint services: `sagemaker.runtime` vs `sagemaker.api`, `bedrock-runtime` vs `bedrock`.

### CloudFormation exports vs Parameter Store dynamic references

|          | Export + Fn::ImportValue                     | Parameter Store `{{resolve:ssm:...}}`               |
| -------- | -------------------------------------------- | --------------------------------------------------- |
| Coupling | Producer locked while the export is imported | No lock                                             |
| Updates  | Producer cannot change the exported value    | Producer replaces it; consumers read it next deploy |

**Remember:** if the producer must stay free to change the value, use Parameter Store.

### Blue/green vs rolling deployment (SageMaker AI endpoints)

|          | Blue/green (all at once, canary, linear) | Rolling                                       |
| -------- | ---------------------------------------- | --------------------------------------------- |
| Capacity | A full second fleet                      | Batches replace old capacity                  |
| Fits     | Clean cut-over with baking periods       | Large or GPU fleets with tight quotas or cost |
| Rollback | Alarms in AutoRollbackConfiguration      | Same                                          |

**Remember:** a tight instance quota rules out every blue/green mode. Without alarms, nothing rolls back.

### SageMaker deployment guardrails vs CodeDeploy

|               | SageMaker AI deployment guardrails | AWS CodeDeploy                                      |
| ------------- | ---------------------------------- | --------------------------------------------------- |
| Targets       | SageMaker AI real-time endpoints   | EC2/on-premises, Lambda, ECS                        |
| Configured in | DeploymentConfig on UpdateEndpoint | Deployment group, AppSpec, deployment configuration |
| Rollback      | CloudWatch alarms                  | CloudWatch alarms on the deployment group           |

**Remember:** CodeDeploy is for inference code on Lambda or ECS, never for SageMaker AI endpoints.

### AgentCore Memory short-term events vs long-term strategies

|          | Short-term (events)            | Long-term (strategies)                      |
| -------- | ------------------------------ | ------------------------------------------- |
| Holds    | Exact conversation turns       | Extracted facts, preferences, summaries     |
| Scope    | Actor ID and session ID        | Actor ID, across sessions                   |
| Exam cue | "replay exactly what was said" | "remember the user's preferences next week" |

**Remember:** a Memory resource with no strategy configured never produces long-term records.

## Domain 4: Operating, Monitoring, and Securing ML and AI Solutions

### Data quality vs model quality vs bias drift vs feature attribution drift

|              | Data quality                             | Model quality                     | Bias drift (Clarify)                | Feature attribution drift (Clarify)       |
| ------------ | ---------------------------------------- | --------------------------------- | ----------------------------------- | ----------------------------------------- |
| Watches      | Input distributions and schema           | Accuracy, precision, recall, RMSE | Outcome gaps between facet values   | The SHAP importance ranking               |
| Needs labels | No                                       | Yes, joined by InferenceId        | Depends on metric                   | No                                        |
| Exam cue     | "upstream changed how a value is scaled" | "fraud labels arrive weeks later" | "approval gap between regions grew" | "model now relies on a different feature" |

**Remember:** every monitor needs data capture and a baseline; only model quality needs ground truth.

### InvocationLatency vs TimeToFirstToken

|          | InvocationLatency                 | TimeToFirstToken                                |
| -------- | --------------------------------- | ----------------------------------------------- |
| Measures | Time to the complete response     | Delay before the first streamed token           |
| Exam cue | "responses take too long overall" | "nothing appears on screen for several seconds" |

**Remember:** a slow start in streaming can hide inside an unchanged total latency.

### UpdateEndpoint vs UpdateEndpointWeightsAndCapacities

|          | UpdateEndpoint                               | UpdateEndpointWeightsAndCapacities           |
| -------- | -------------------------------------------- | -------------------------------------------- |
| Changes  | The whole endpoint configuration (new model) | Variant weights and instance counts in place |
| Needs    | A new endpoint configuration                 | Nothing new                                  |
| Exam cue | "deploy a new model version"                 | "shift A/B traffic from 90/10 to 50/50"      |

**Remember:** weights only means UpdateEndpointWeightsAndCapacities.

### TargetVariant vs InvokedProductionVariant

|          | TargetVariant                       | InvokedProductionVariant                  |
| -------- | ----------------------------------- | ----------------------------------------- |
| Where    | Request parameter on InvokeEndpoint | Field in the InvokeEndpoint response      |
| Does     | Pins one request to a named variant | Names the variant that served the request |
| Exam cue | "QA must hit variant B"             | "join conversion outcomes to the variant" |

**Remember:** one routes, the other attributes.

### CloudTrail vs Bedrock model invocation logging vs AWS Config

|          | AWS CloudTrail                         | Model invocation logging        | AWS Config                                      |
| -------- | -------------------------------------- | ------------------------------- | ----------------------------------------------- |
| Records  | Who called which API, when, from where | Full prompts and responses      | Resource configuration and compliance over time |
| Default  | Management events on; data events off  | Off                             | Off until rules are deployed                    |
| Exam cue | "who deleted the endpoint"             | "what exactly did the user ask" | "prove notebooks never had internet access"     |

**Remember:** CloudTrail never holds prompt text, and Config detects after the fact while IAM and SCP conditions prevent.

### SageMaker Savings Plans vs Compute Savings Plans

|             | SageMaker Savings Plans                                        | Compute Savings Plans         |
| ----------- | -------------------------------------------------------------- | ----------------------------- |
| Covers      | SageMaker AI training, inference, batch, processing, notebooks | EC2, Fargate, Lambda          |
| Flexibility | Any family, size, Region, component                            | Any family and Region         |
| Exam cue    | "steady SageMaker endpoint spend"                              | "steady EC2 and Lambda spend" |

**Remember:** Compute Savings Plans do not apply to SageMaker AI. Commit to the baseline, not the average or peak.

### Bedrock Flex vs Priority vs batch inference vs Provisioned Throughput

|            | Flex tier                             | Priority tier                    | Batch inference                   | Provisioned Throughput / Reserved         |
| ---------- | ------------------------------------- | -------------------------------- | --------------------------------- | ----------------------------------------- |
| Price      | Discounted per token                  | Premium per token                | Discounted per token              | Billed by time, used or not               |
| Call style | Normal synchronous calls              | Normal synchronous calls         | JSONL jobs in S3                  | Provisioned model ARN                     |
| Exam cue   | "overnight agent jobs, keep the code" | "latency-critical, served first" | "reprocess millions of documents" | "steady high volume, guaranteed capacity" |

**Remember:** Flex is the discount when calls must stay synchronous; batch only when the work can move to S3 files.

### AWS Budgets forecast alerts vs Budgets actions vs Cost Anomaly Detection vs CloudWatch anomaly detection

|          | Budgets forecasted alert       | Budgets action                              | Cost Anomaly Detection               | CloudWatch anomaly detection alarm |
| -------- | ------------------------------ | ------------------------------------------- | ------------------------------------ | ---------------------------------- |
| Does     | Warns before month-end overrun | Applies an IAM policy or SCP, stops EC2/RDS | ML over billing data                 | Learns a usage metric's pattern    |
| Speed    | Projected                      | At the threshold                            | Lags with billing data               | Minutes                            |
| Exam cue | "warn before we exceed"        | "automatically block new training jobs"     | "finance wants unusual spend alerts" | "page when token output spikes"    |

**Remember:** Budgets actions cannot stop SageMaker AI resources; deny the create actions instead.

### Network isolation vs VPC configuration

|             | EnableNetworkIsolation                         | VPC config (subnets + security groups)         |
| ----------- | ---------------------------------------------- | ---------------------------------------------- |
| Effect      | No outbound network from the container at all  | Container traffic follows your VPC routes      |
| Still works | Input channels and artifact upload             | Anything your routes and endpoints allow       |
| Exam cue    | "untrusted vendor container must not call out" | "reach private resources", "use VPC endpoints" |

**Remember:** a VPC alone does not stop outbound calls; network isolation does.

### IAM role credentials vs short-term vs long-term Bedrock API keys

|          | IAM role credentials    | Short-term API key                | Long-term API key        |
| -------- | ----------------------- | --------------------------------- | ------------------------ |
| Lifetime | Temporary, auto-rotated | Up to 12 hours, one Region        | Until expiry or deletion |
| Tied to  | A role                  | The generating principal          | An IAM user              |
| Use for  | Production apps on AWS  | Tools that only accept an API key | Exploration only         |

**Remember:** production on AWS means role credentials; never a long-term key.

### Denied topics vs word filters vs content filters vs prompt attack filter

|          | Denied topics                  | Word filters             | Content filters                     | Prompt attack filter                 |
| -------- | ------------------------------ | ------------------------ | ----------------------------------- | ------------------------------------ |
| Matches  | A subject, however phrased     | Exact words and phrases  | Harm categories at a strength       | Jailbreaks, injection, leakage       |
| Exam cue | "never give investment advice" | "block competitor names" | "filter violent or hateful content" | "users try to override instructions" |

**Remember:** a roundabout phrasing beats a word filter but not a denied topic; the prompt attack filter only checks tagged user input.

### ECR scanning vs Inspector SBOM Generator in the build

|          | ECR basic or Inspector enhanced scanning           | Inspector SBOM Generator + Scan API in CodeBuild |
| -------- | -------------------------------------------------- | ------------------------------------------------ |
| Runs     | After the image is in the registry                 | Before the push                                  |
| Strength | Continuous re-checks as new CVEs appear (enhanced) | Vulnerable images never stored                   |

**Remember:** "never stored" needs the in-build scan; "keep checking old images" needs continuous enhanced scanning. Many scenarios need both.

[← Back to the study guide](README.md)
