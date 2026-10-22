# MLA-C02 Study Guide: AWS Certified Machine Learning Engineer - Associate

A free, open study guide for the **AWS Certified Machine Learning Engineer - Associate (MLA-C02)** exam: revision notes for every domain, side-by-side comparisons of commonly confused services, a glossary, 20 worked sample questions, and the official syllabus as a checklist with a full free lesson for every topic.

Maintained by [SaveMyCert](https://www.savemycert.com/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide), where you can read every lesson free, [practice with explained questions](https://www.savemycert.com/practice/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide) and [take timed mock exams](https://www.savemycert.com/mocks/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [Exam domains](#exam-domains)
- [What is in this repo](#what-is-in-this-repo)
- [Syllabus checklist](#syllabus-checklist)
  - [Domain 1: Data Preparation for ML and AI](#domain-1-data-preparation-for-ml-and-ai)
  - [Domain 2: ML Model and Foundation Model (FM) Development](#domain-2-ml-model-and-foundation-model-fm-development)
  - [Domain 3: Deployment and Orchestration of ML and AI Workflows](#domain-3-deployment-and-orchestration-of-ml-and-ai-workflows)
  - [Domain 4: Operating, Monitoring, and Securing ML and AI Solutions](#domain-4-operating-monitoring-and-securing-ml-and-ai-solutions)
- [How to study for MLA-C02](#how-to-study-for-mla-c02)
- [Sample questions](sample-questions.md)
- [Free resources](#free-resources)

## Exam at a glance

| | |
|---|---|
| Exam code | MLA-C02 |
| Level | Associate |
| Questions | 65 (beta: 85) |
| Time limit | 130 min (beta: 170) |
| Passing score | 720 / 1000 |
| Format | Multiple choice & multiple response |
| Exam fee | $150 (beta: $75) |
| Valid for | 3 years |

Exam details change. Always confirm them in the official [AWS Certified Machine Learning Engineer – Associate (MLA-C02) exam guide](https://docs.aws.amazon.com/aws-certification/latest/machine-learning-engineer-associate-02/machine-learning-engineer-associate-02.html) from Amazon Web Services.

## Exam domains

| # | Domain | Weight | Topics |
|---|---|---|---|
| 1 | [Data Preparation for ML and AI](#domain-1-data-preparation-for-ml-and-ai) | 28% | 3 |
| 2 | [ML Model and Foundation Model (FM) Development](#domain-2-ml-model-and-foundation-model-fm-development) | 24% | 3 |
| 3 | [Deployment and Orchestration of ML and AI Workflows](#domain-3-deployment-and-orchestration-of-ml-and-ai-workflows) | 24% | 3 |
| 4 | [Operating, Monitoring, and Securing ML and AI Solutions](#domain-4-operating-monitoring-and-securing-ml-and-ai-solutions) | 24% | 3 |

That is 4 domains and 12 topics. Spend your time in proportion to the weights: the heaviest domain decides more of your score than the lightest.

## What is in this repo

| File | What it gives you |
|---|---|
| [Domain 1 notes](notes/domain-1-data-preparation-for-ml-and-ai.md) | Data Preparation for ML and AI: condensed revision notes per topic |
| [Domain 2 notes](notes/domain-2-ml-model-and-foundation-model-fm-development.md) | ML Model and Foundation Model (FM) Development: condensed revision notes per topic |
| [Domain 3 notes](notes/domain-3-deployment-and-orchestration-of-ml-and-ai-workflows.md) | Deployment and Orchestration of ML and AI Workflows: condensed revision notes per topic |
| [Domain 4 notes](notes/domain-4-operating-monitoring-and-securing-ml-and-ai-solutions.md) | Operating, Monitoring, and Securing ML and AI Solutions: condensed revision notes per topic |
| [Commonly confused services](comparisons.md) | Side-by-side tables of the services questions set against each other |
| [Glossary](glossary.md) | Every in-scope term and service in one sentence |
| [Sample questions](sample-questions.md) | 20 worked questions with answers and reasoning |
| [Exam-day guide](exam-day-guide.md) | Booking, testing options, scoring, results and retakes |

## Syllabus checklist

Tick each topic off once you can explain it without notes. The "Must know" facts are the ones questions turn on. Each lesson link goes to the complete, free lesson.

### Domain 1: Data Preparation for ML and AI

**Weight: 28%.** Collecting and storing data for ML and AI, transforming it and engineering features, and validating data quality and managing bias — for traditional ML and foundation models alike.

📝 Revision notes: [Domain 1: Data Preparation for ML and AI](notes/domain-1-data-preparation-for-ml-and-ai.md)

- [ ] **1 Collect and store data**
  <br>1 (MLA-C02). Extracting data from sources (Amazon S3, EBS, EFS, RDS, DynamoDB, OpenSearch Service); storage decisions by cost, performance, data structure and compliance; troubleshooting ingestion and storage capacity/scalability issues; streaming ingestion (Amazon Kinesis, Apache Flink, Apache Kafka); data formats by access pattern (Apache Parquet, JSON, CSV, ORC); merging sources (code, AWS Glue, Apache Spark); scalable vector databases for AI (OpenSearch Service, Amazon RDS with pgvector, Amazon S3); ingesting and storing text, image and audio data; ingesting into SageMaker Feature Store.
  - 📖 Lesson: [Collecting and Storing Data for ML and AI on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-data-ingestion-storage-feature-store/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: File mode downloads the whole channel before training; FastFile streams with file semantics and no script change; Pipe needs pipe-reading code. ShardedByS3Key splits data across instances, FullyReplicated copies all of it to each.
  - Must know: EFS and FSx for Lustre channels need VPC configuration; FSx for Lustre is the throughput choice for repeated multi-instance training and lives in one Availability Zone.
- [ ] **1 Perform data transformation, feature engineering, and pre-processing**
  <br>2 (MLA-C02). Transforming data with AWS tools (AWS Glue, Glue DataBrew, Spark on Amazon EMR, SageMaker Data Wrangler); creating and managing features (SageMaker Feature Store); transforming streaming data (AWS Lambda, Spark); feature engineering (scaling, standardization, feature splitting, binning, log transformation, normalization); embedding models for text and image data; advanced text pre-processing (tokenization, domain-specific augmentation); preparing documents for Retrieval Augmented Generation (chunking strategies, metadata extraction); masking, redacting and anonymizing data; preparing data for FM fine-tuning, continuous pre-training and model distillation.
  - 📖 Lesson: [Data Transformation, Feature Engineering and Pre-processing on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-feature-engineering-data-transformation/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Glue = serverless Spark (job bookmarks for incremental runs); DataBrew = visual no-code recipes; EMR = cluster control and Spot task nodes; Data Wrangler = ML-focused visual prep you export to Pipelines, Feature Store or a processing job.
  - Must know: Every feature group needs a record identifier and an event time; the online store serves the latest value in milliseconds, the offline store keeps every version in S3.
- [ ] **1 Validate data quality and manage bias**
  <br>3 (MLA-C02). Validating data quality (Glue DataBrew, AWS Glue Data Quality); labeling and annotating data; identifying and mitigating bias in data (dataset splitting, shuffling, augmentation); applying bias metrics across numeric, text and image assets; resolving class imbalance; validating AI training-data integrity (prompt-response pair validation, content safety screening); cleaning data (outlier detection, imputing missing values, deduplication).
  - 📖 Lesson: [Data Quality Validation, Bias Metrics and Class Imbalance for ML (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-data-quality-validation-bias-class-imbalance/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Glue Data Quality in an ETL job gives row-level results so bad rows can be quarantined in the same run; Data Catalog runs check data at rest, recommend rules and filter with partition predicates.
  - Must know: DQDL Boolean rules (IsComplete, IsUnique) allow zero failures; ratio rules (Completeness, Uniqueness) take a threshold; last(k) with k > 1 needs an aggregation such as min or avg.

### Domain 2: ML Model and Foundation Model (FM) Development

**Weight: 24%.** Choosing modeling approaches — traditional ML, managed AI services, foundation models and RAG — then training, fine-tuning and customizing models, and evaluating ML and GenAI performance.

📝 Revision notes: [Domain 2: ML Model and Foundation Model (FM) Development](notes/domain-2-ml-model-and-foundation-model-fm-development.md)

- [ ] **2 Choose appropriate modeling approaches for ML and AI solutions**
  <br>1 (MLA-C02). Selecting foundation models in Amazon Bedrock by task requirements and performance; fine-tuning strategies for pre-trained FMs; comparing ML models, GenAI models, algorithms and solution templates (interpretability, domain-specific performance, latency); trade-offs between custom solutions, managed services, pre-trained models and FMs; choosing Retrieval Augmented Generation architecture patterns; trade-offs between model performance, training time, latency and cost; applying AWS AI services to business problems (Amazon Textract, Rekognition, Comprehend, Transcribe).
  - 📖 Lesson: [Choosing ML, Foundation Model and RAG Approaches on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-modeling-approaches-fm-selection-rag/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Use the least custom approach that works: AI service, then FM or traditional ML, then a custom model; FMs are a poor fit for high-volume tabular or fixed-class prediction when labels exist.
  - Must know: Interpretability requirements decide the model: fixed published weights mean a linear model; SHAP values vary per prediction, importance rankings lack direction, monotonic constraints fix direction only.
- [ ] **2 Train, fine-tune, and customize models for ML and AI solutions**
  <br>2 (MLA-C02). SageMaker AI built-in algorithms and common ML libraries; SageMaker AI script mode with supported frameworks; hyperparameter optimization (SageMaker AI automatic model tuning); reducing training time (early stopping, distributed training); preventing overfitting, underfitting and catastrophic forgetting; combining models (ensembles) for performance or cost; fundamental hyperparameters (epochs, steps, batch size); customization techniques for AI solutions (task-specific prompt engineering, fine-tuning); optimizing retrieval components and embedding models.
  - 📖 Lesson: [Training, Hyperparameter Tuning and Fine-Tuning Models on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-training-hyperparameter-tuning-fine-tuning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Built-in algorithm choice follows labels and output: RCF for unlabeled anomaly scores, DeepAR for many related time series, XGBoost for tabular prediction, k-means only clusters.
  - Must know: Built-in CSV input means the label in the first column and no header; FastFile streams data and ShardedByS3Key gives each instance its own subset.
- [ ] **2 Analyze and evaluate the performance of ML and AI systems**
  <br>3 (MLA-C02). Reproducible experiments (MLflow on SageMaker AI, Amazon Bedrock evaluations, Bedrock Prompt Management); performance baselines and drift detection; comparing shadow variants with production variants; explaining model outputs; debugging convergence; evaluation techniques for traditional ML and GenAI models; human evaluation (human-in-the-loop, text generation quality assessment); NLP metrics (BLEU, ROUGE, BERTScore, semantic similarity); AI evaluation (output assessment, content quality, bias detection, LLM-as-a-judge); RAG monitoring and retrieval accuracy.
  - 📖 Lesson: [Evaluating ML and GenAI Models: Metrics, Drift, Explainability and RAG on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-and-genai-model-evaluation-drift-explainability/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Reproducible runs need the data version (MLflow dataset input with a digest) and the code version (Git commit tag), not only parameters and metrics; a training job reaches managed MLflow only with the tracking server ARN as tracking URI and the sagemaker-mlflow plugin.
  - Must know: In Bedrock Prompt Management, iterate on the draft, compare alternatives as variants, and ship immutable numbered versions by ARN; fix a prompt by creating a new version, never by editing or deleting the old one.

### Domain 3: Deployment and Orchestration of ML and AI Workflows

**Weight: 24%.** Selecting deployment infrastructure for ML models, foundation models and agents, provisioning and scaling the resources behind them, and automating MLOps with CI/CD pipelines.

📝 Revision notes: [Domain 3: Deployment and Orchestration of ML and AI Workflows](notes/domain-3-deployment-and-orchestration-of-ml-and-ai-workflows.md)

- [ ] **3 Manage deployment infrastructure for ML and AI model types**
  <br>1 (MLA-C02). Selecting compute environments and deployment targets; deployment orchestrators and multi-model or multi-container strategies; inference strategies (real-time, batch); foundation model deployment options; deploying models built outside AWS (SageMaker AI, Amazon Bedrock Custom Model Import); deploying and configuring agents, their tool integrations and agent communication protocols; FM hosting and resource allocation; RAG system configuration (retrieval strategies, reranking).
  - 📖 Lesson: [Deploying ML Models, Foundation Models and Agents on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-bedrock-model-deployment-inference-strategies/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Synchronous plus idle gaps plus acceptable cold start means serverless inference; large payloads or minutes of processing with scale-to-zero means asynchronous inference; a scheduled dataset means batch transform.
  - Must know: Asynchronous requests pass the payload as an S3 InputLocation, and success and error SNS topics in AsyncInferenceConfig report completion without polling.
- [ ] **3 Provision and configure resources for ML and AI workloads based on existing architecture and requirements**
  <br>2 (MLA-C02). Balancing on-demand and provisioned resources; automating compute provisioning across stacks and orchestration services; building and maintaining containers for ML and AI; SageMaker AI endpoints in a VPC; deploying models programmatically (SageMaker AI Python SDK, AWS CLI, Boto3); auto scaling metrics; Amazon Bedrock knowledge bases (vector database configuration, document indexing, retrieval optimization); retrieval pipelines; agent state management; GPU resource scaling; agentic workflow infrastructure.
  - 📖 Lesson: [Provisioning ML Resources: Endpoints, Scaling, Knowledge Bases and Agents (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-endpoint-scaling-vpc-bedrock-knowledge-bases/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Only Bedrock Provisioned Throughput reserves model capacity for one workload; cross-Region inference profiles absorb bursts while staying on-demand, and batch inference is never interactive.
  - Must know: Serverless provisioned concurrency removes cold starts and can be scheduled with Application Auto Scaling, but its scalable target floors at 1, and serverless has no GPUs.
- [ ] **3 Implement automated orchestration and continuous integration and continuous delivery (CI/CD) pipelines for MLOps and AI workloads**
  <br>3 (MLA-C02). Automated deployment strategies and rollback; configuring and troubleshooting AWS CodeBuild, CodeCommit, CodeDeploy, CodePipeline and CodeConnections; training and inference jobs; automated testing in ML and AI pipelines; re-training mechanisms; model versioning for repeatability and audit (SageMaker Model Registry, MLflow on SageMaker AI); prompt management (Amazon Bedrock Prompt Management); agent deployment pipelines and agent versioning; AI model and prompt testing frameworks; FM deployment automation with fine-tuned model versioning; pipeline orchestration for RAG updates and knowledge base refresh.
  - 📖 Lesson: [MLOps CI/CD Pipelines, Model Versioning and Rollback on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-mlops-cicd-pipelines-model-versioning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: CodePipeline releases, CodeBuild builds and tests, CodeConnections links external Git, and SageMaker Pipelines runs the ML workflow; CodeDeploy shifts traffic for inference on Lambda or ECS, while SageMaker AI endpoints use their own guardrails.
  - Must know: Docker builds in CodeBuild need privileged mode; ECR pushes need service-role permissions; a full-clone source needs codeconnections:UseConnection on the CodeBuild role too.

### Domain 4: Operating, Monitoring, and Securing ML and AI Solutions

**Weight: 24%.** Monitoring models, agents and data in production, optimizing infrastructure and inference cost, and securing ML and AI workloads and endpoints.

📝 Revision notes: [Domain 4: Operating, Monitoring, and Securing ML and AI Solutions](notes/domain-4-operating-monitoring-and-securing-ml-and-ai-solutions.md)

- [ ] **4 Monitor ML and AI model inference and performance**
  <br>1 (MLA-C02). Monitoring production model performance (Amazon CloudWatch generative AI observability, Amazon Bedrock Model Evaluation, drift detection pipelines); detecting anomalies or errors in data processing and inference; detecting data distribution changes; A/B testing in production; monitoring and automating agent performance and coordination (coordination failures, truncated streaming, tool failures); FM-specific performance monitoring (Amazon Bedrock evaluations).
  - 📖 Lesson: [Monitoring ML Models, FMs and Agents in Production: Drift, A/B Tests and Bedrock Evaluations (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-model-monitoring-drift-detection-bedrock-evaluations/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Bedrock invocation logging is off by default; the Model Invocations view's per-request table needs it delivered to CloudWatch Logs, and bodies over 100 KB need an S3 large-data location.
  - Must know: TimeToFirstToken measures a slow start in streaming; InvocationLatency measures the whole response. Early failures appear only in Bedrock metrics without the ModelId dimension.
- [ ] **4 Optimize and manage ML and AI infrastructure costs and performance**
  <br>2 (MLA-C02). Choosing inference instance families; troubleshooting tools (Amazon CloudWatch, Amazon Bedrock AgentCore Observability, AWS X-Ray); performance dashboards; capacity optimization for cost, performance and reliability; cost management tools and quotas; purchasing options; cost of FM inference in production; agent resource consumption; FM inference cost optimization; AI-specific cost patterns (token usage, embedding computation, vector database storage).
  - 📖 Lesson: [Optimising ML and GenAI Inference Cost and Performance on AWS: Instances, Savings Plans, Tokens and Vectors (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-genai-inference-cost-optimization/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Pick the instance family that supplies the bottleneck resource: c for CPU-bound, r for memory-heavy, GPU only when the model uses it, and check GPU memory (16, 24 or 48 GB) before LLM fit.
  - Must know: Inference Recommender default jobs shortlist instances; advanced load test jobs take your traffic phases, endpoint configurations and percentile-latency stopping conditions.
- [ ] **4 Secure ML and AI workloads and model endpoints**
  <br>3 (MLA-C02). Securing CI/CD pipelines by scanning code and images (Amazon CodeGuru, Amazon Inspector); least-privilege access to ML and AI artifacts; IAM policies and roles for users and applications; monitoring, auditing, compliance and logging (AWS CloudTrail, AWS Config); troubleshooting security issues; VPCs, subnets and security groups to isolate ML and AI systems; identifying and mitigating ML and AI security risks; choosing credential types for FMs (Amazon Bedrock API keys, IAM credentials); safeguards and sensitive-data protection for responsible AI (Amazon Bedrock Guardrails).
  - 📖 Lesson: [Securing ML and AI Workloads on AWS: IAM, VPC Isolation, Auditing and Bedrock Guardrails (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/secure-ml-ai-workloads-iam-vpc-bedrock-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
  - Must know: Any registry-side image scan runs after the push; a 'never stored' rule needs the Inspector SBOM Generator and Scan API inside the build, and 'keep re-checking old images' needs Inspector continuous enhanced scanning.
  - Must know: CodeGuru Reviewer is closed to new repository associations and CodeGuru Security is discontinued; Amazon Inspector code security provides SAST, SCA and IaC scanning for GitHub and GitLab.

## How to study for MLA-C02

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [MLA-C02 revision notes](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide).
2. **Practice straight after reading.** Answer [MLA-C02 practice questions](https://www.savemycert.com/practice/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [MLA-C02 mock exam](https://www.savemycert.com/mocks/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [MLA-C02 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked MLA-C02 questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [AWS Certified Machine Learning Engineer – Associate (MLA-C02) exam guide](https://docs.aws.amazon.com/aws-certification/latest/machine-learning-engineer-associate-02/machine-learning-engineer-associate-02.html): the official source (Amazon Web Services)
- [MLA-C02 certification overview](https://www.savemycert.com/certifications/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)
- [MLA-C02 revision notes](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide): every lesson, free to read
- [MLA-C02 practice questions](https://www.savemycert.com/practice/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide): with an explanation on every option
- [MLA-C02 mock exams](https://www.savemycert.com/mocks/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide): full-length and timed
- [MLA-C02 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-machine-learning-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide): the key facts on one page
- [All certification study guides](https://github.com/savemycert/certification-study-guides)

## Contributing

Spotted an error or an out-of-date fact? [Open an issue](../../issues) with the topic and a link to the official source. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and disclaimer

This guide is licensed under [CC BY 4.0](LICENSE). You can reuse and adapt it, including commercially, as long as you credit **SaveMyCert** with a link to https://www.savemycert.com/.

This is an independent study resource. It is not affiliated with or endorsed by Amazon Web Services. AWS Certified Machine Learning Engineer - Associate and MLA-C02 are trademarks of their respective owner. Exam domains and weights are taken from the official exam guide linked above.
