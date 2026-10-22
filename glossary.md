# MLA-C02 Glossary

220 MLA-C02 terms and services, each in one plain-English sentence.

[#](#) · [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [X](#x)

## #

- **.sync integration**: A Step Functions pattern that waits for a job to finish before moving on.

## A

- **Action group**: The tool definition of a Bedrock agent, executed by Lambda or returned to the application.
- **ADOT**: AWS Distro for OpenTelemetry, used to emit agent spans and logs to CloudWatch.
- **Agent trace**: Step-by-step InvokeAgent output showing rationale, tool inputs and observations.
- **Agent2Agent (A2A)**: A protocol that lets one agent delegate tasks to another.
- **AgentCore Evaluations**: LLM-as-a-judge scoring of agent sessions, online by sampling or on demand.
- **AgentCore Gateway**: A service that turns Lambda functions and APIs into shared MCP tools.
- **AgentCore Identity**: A service that handles inbound authorization and outbound OAuth tokens for agents.
- **AgentCore Memory**: Managed agent state with short-term events and long-term memory strategies.
- **Amazon Bedrock**: A managed service for invoking, customizing and evaluating foundation models through one API.
- **Amazon Bedrock AgentCore Runtime**: Serverless hosting for agent code in any framework, with isolated microVM sessions.
- **Amazon Bedrock Guardrails**: Configurable policies that block or mask content in prompts and responses.
- **Amazon Data Firehose**: A managed service that buffers streaming records and delivers them to S3 and other destinations.
- **Amazon ECR enhanced scanning**: Amazon Inspector scanning of OS and language packages in stored images, optionally continuous.
- **Amazon Inspector enhanced scanning**: Continuous scanning of OS and language packages in ECR images.
- **Amazon Macie**: A service that discovers personal and sensitive data inside S3 objects.
- **Amazon S3 Vectors**: Vector buckets and indexes in S3 for low-cost similarity search on large collections.
- **Anomaly detection band**: A CloudWatch alarm model that learns a metric's normal pattern instead of a fixed threshold.
- **Apache Parquet**: A compressed columnar file format that lets engines read only the columns and row groups they need.
- **Application inference profile**: A tagged Bedrock resource that separates usage and cost per application or team.
- **ApplyGuardrail**: A Bedrock API that checks text against a guardrail's policies without invoking a model.
- **AppSpec file**: The CodeDeploy file that names the target, versions and lifecycle hooks.
- **Asynchronous inference**: A SageMaker AI option that queues requests with S3 payloads, writes results to S3 and can scale to zero.
- **AsyncInferenceConfig**: Endpoint configuration for asynchronous inference, including success and error SNS topics.
- **Automatic model tuning (AMT)**: A SageMaker AI feature that runs many training jobs to find the best hyperparameters.
- **AWS Budgets action**: An automatic response to a budget threshold, such as applying an IAM policy or SCP.
- **AWS CodeBuild**: A managed build service driven by a buildspec.
- **AWS CodeConnections**: Authorised links from AWS developer tools to GitHub, GitLab or Bitbucket.
- **AWS CodePipeline**: A release orchestrator with source, build, test, approval and deploy stages.
- **AWS Cost Anomaly Detection**: Machine-learning monitors that flag unusual spend in billing data.
- **AWS Glue FindMatches**: A Glue ML transform that learns to identify records referring to the same entity.
- **AWS Glue Schema Registry**: A registry of stream schemas that rejects new versions breaking the chosen compatibility mode.
- **AWS X-Ray**: A tracing service that shows each hop of a request and its duration.
- **aws:SourceVpce**: A condition key that restricts requests to those arriving through a specific VPC endpoint.

## B

- **Back-translation**: Translating text to another language and back to create a label-preserving paraphrase.
- **Bagging**: Training many models on bootstrap samples and averaging them to reduce variance.
- **baseline_drift_check**: A data quality violation raised when a feature's distribution moves past the threshold.
- **Batch transform**: A SageMaker AI job that scores a whole S3 dataset and shuts down when it finishes.
- **Bedrock API key**: A bearer-style credential for Bedrock, short-term (up to 12 hours) or long-term (tied to an IAM user).
- **Bedrock Custom Model Import**: A feature that hosts fine-tuned weights of supported architectures in Bedrock with on-demand billing.
- **Bedrock Marketplace**: Specialised models deployed to endpoints you size and called through Bedrock APIs.
- **Bedrock Prompt Management**: A Bedrock feature that stores prompts as drafts, immutable versions and comparable variants.
- **bedrock:GuardrailIdentifier**: A condition key used to require a specific guardrail version on inference calls.
- **BERTScore**: A metric that compares output and reference tokens using contextual embeddings.
- **Binary embeddings**: Embeddings stored at 1 bit per dimension to shrink vector indexes.
- **Binning**: Grouping a numeric feature into ranges, often one-hot encoded so a linear model can learn non-linear effects.
- **BlazingText**: A built-in algorithm for Word2Vec embeddings and supervised text classification.
- **BLEU**: A precision-oriented n-gram overlap metric used for machine translation.
- **Boosting**: Adding weak models in sequence, each fixing the previous errors, to reduce bias.
- **Broadcast join**: A Spark join that copies a small table to every executor so the large table is not shuffled.
- **Buffering hints**: Firehose size and interval limits; an object is written when either is reached.

## C

- **cachePoint**: A Converse API marker that ends the prompt prefix to cache.
- **Catastrophic forgetting**: Loss of a model's general abilities after fine-tuning on narrow data.
- **Chunking**: Splitting documents into the passages that a RAG system embeds and retrieves.
- **CloudTrail data events**: High-volume API events, such as InvokeEndpoint, that are recorded only when selected.
- **CloudWatch generative AI observability**: Prebuilt CloudWatch views for Bedrock model invocations and AgentCore agents.
- **CloudWatch Logs anomaly detection**: A feature that flags new or unusual log patterns in a log group.
- **CloudWatch Logs data protection policy**: A log group policy that masks sensitive data for readers without logs:Unmask.
- **CloudWatch Transaction Search**: An account setting required for trace spans to be stored for AgentCore observability.
- **Concept drift**: A change in the relationship between inputs and the correct label.
- **Condition key**: A policy element such as sagemaker:NetworkIsolation that tests request properties.
- **Conditional demographic disparity (CDD)**: A bias metric that measures disparity within subgroups to expose Simpson's paradox.
- **ConditionStep**: A SageMaker Pipelines step that branches on a value such as an evaluation metric.
- **Conformance pack**: A bundle of AWS Config rules deployable across an organization.
- **Context coverage**: A RAG metric for whether retrieved passages contain the information in the reference answer.
- **Context window**: The maximum number of input tokens a model accepts in one request.
- **Contextual grounding check**: A guardrail policy that flags responses not supported by the source or not relevant to the query.
- **Continued pre-training**: Further training of a foundation model on unlabeled domain text.
- **Cost allocation tag**: A resource tag activated in Billing so it appears as a cost dimension.
- **Cross-account observability**: A CloudWatch feature linking source accounts to a central monitoring account.
- **Cross-Region inference profile**: A Bedrock profile that routes on-demand requests to other Regions to absorb peaks.

## D

- **Data capture**: A SageMaker AI endpoint setting that writes requests and responses to S3 for monitoring.
- **Data drift**: A change in the distribution of model inputs compared with training data.
- **Data parallelism**: Distributed training where each GPU holds the full model and processes different batches.
- **DeepAR**: A built-in algorithm that trains one recurrent model across many related time series for probabilistic forecasts.
- **Deployment guardrails**: SageMaker AI blue/green and rolling update modes with alarm-based rollback.
- **DPPL**: A post-training bias metric comparing predicted positive rates between groups.
- **DQDL**: Data Quality Definition Language, the rule syntax used by AWS Glue Data Quality.
- **Dynamic partitioning**: A Firehose feature that builds S3 prefixes from values inside each record.
- **Dynamic reference**: A CloudFormation placeholder such as {{resolve:ssm:name}} resolved at deploy time.

## E

- **Early stopping**: Ending training or a tuning trial once the metric stops improving or falls behind.
- **ECR lifecycle policy**: A rule that expires images automatically, such as untagged images past an age.
- **Embedding**: A fixed-length numeric vector whose distances reflect the meaning of the text or image it came from.
- **Enhanced fan-out**: A Kinesis consumer type with its own dedicated 2 MB/s of read throughput per shard.
- **Epoch**: One full pass over the training data.
- **Event time**: The feature that records when a Feature Store record's values were true.
- **Excessive agency**: An agent tool that can do more than its use case needs.
- **Execution role**: The IAM role a SageMaker AI job or endpoint uses for everything it does.

## F

- **Facet**: A group defined by a sensitive or relevant attribute, compared in bias metrics.
- **FailStep**: A SageMaker Pipelines step that ends the run as Failed with a message.
- **Faithfulness**: Whether a response's claims are supported by the context it was given.
- **FastFile mode**: An input mode that exposes S3 objects as local files and streams them as the script reads them.
- **Feature attribution drift**: A change in which features drive predictions, measured on the SHAP ranking.
- **File mode**: The default SageMaker AI input mode, which downloads the whole channel to the instance before training starts.
- **Flex tier**: A discounted Bedrock service tier for latency-tolerant synchronous requests.
- **Fn::ImportValue**: A CloudFormation function that reads a value exported by another stack.
- **Foundation model (FM)**: A large pre-trained model that can be prompted or customized for many tasks.
- **FullyReplicated**: The default data distribution setting, which copies the whole channel to every training instance.

## G

- **Gateway endpoint**: A route-table VPC endpoint, available only for S3 and DynamoDB.
- **Gold-standard item**: A labeling task with a known answer, used to measure each annotator's accuracy.
- **Gradient accumulation**: Summing gradients over several small batches before one update to mimic a larger batch.
- **GraphRAG**: RAG that uses extracted entities and relationships, in Bedrock via Amazon Neptune Analytics, for multi-hop questions.
- **Guardrail input tags**: Tags that mark user input so the prompt attack filter evaluates it.

## H

- **Hierarchical chunking**: Small child chunks for matching, nested in larger parent chunks returned for context.
- **HNSW**: A graph-based vector index with strong recall that needs no training step.
- **Hybrid search**: Retrieval that combines keyword matching with vector similarity.

## I

- **iam:PassRole**: The permission a caller needs to hand an IAM role to a service such as SageMaker AI.
- **idleRuntimeSessionTimeout**: The AgentCore Runtime setting that ends idle sessions, 900 seconds by default.
- **Implicit filtering**: Letting a model derive metadata filters from the user's query; not a security control.
- **Indirect prompt injection**: Instructions hidden in content an agent reads that try to trigger unintended actions.
- **Inference component**: A model deployed on an endpoint with its own declared resources and independent copy scaling.
- **InferenceId**: A unique request ID passed on InvokeEndpoint and used to join late ground-truth labels.
- **Inferentia2**: An AWS accelerator built for low-cost deep learning inference with the Neuron SDK.
- **Inspector code security**: Inspector SAST, SCA and IaC scanning for GitHub and GitLab repositories.
- **Intelligent prompt routing**: A Bedrock feature that sends each prompt to one of two models in a family based on predicted quality.
- **Inter-container traffic encryption**: A setting that encrypts traffic between instances in distributed training.
- **Interface endpoint**: A PrivateLink VPC endpoint with private IPs for other AWS services.
- **InvocationLatency**: A Bedrock metric for the time until the whole response is complete.
- **InvokedProductionVariant**: A response field naming the variant that served an InvokeEndpoint request.
- **IVFFlat**: A cluster-based vector index that builds quickly but needs representative data first.

## J

- **Job bookmark**: A Glue feature that tracks processed data so each run reads only what is new.
- **JoinSource**: A batch transform setting that appends each prediction to its original input record.
- **JSON Lines (JSONL)**: A format with one complete JSON object per line, used for Bedrock customization datasets.

## K

- **KV cache**: Memory that stores attention keys and values during generation and grows with batch and context length.

## L

- **Large model inference (LMI) container**: A SageMaker AI container for serving large language models with engines such as vLLM.
- **LLM-as-a-judge**: Evaluation in which a separate model scores responses against named criteria.
- **LoRA**: A parameter-efficient fine-tuning method that freezes base weights and trains small low-rank adapters.

## M

- **Managed MLflow on SageMaker AI**: A hosted MLflow tracking server or App for experiments and model registry.
- **Managed Spot training**: SageMaker AI training on spare capacity at a discount, resumed from checkpoints after interruption.
- **Metadata filtering**: Restricting retrieval to chunks whose metadata attributes match a filter.
- **Metric definition**: A regex that extracts a metric from training job logs for tuning or CloudWatch.
- **Model Context Protocol (MCP)**: A protocol that lets agents discover and call tools.
- **Model distillation**: Training a smaller student model on responses generated by a larger teacher model.
- **Model invocation logging**: A Bedrock setting, off by default, that logs full prompts and responses to CloudWatch Logs or S3.
- **Model Package State Change event**: An EventBridge event emitted when a model version's approval status changes.
- **Model parallelism**: Distributed training that splits the model's state across GPUs.
- **ModelLatency**: The time a SageMaker AI model container takes to respond.
- **ModelSetupTime**: The cold-start time of a serverless endpoint.
- **Multi-container endpoint**: An endpoint hosting up to 15 different containers, each invoked directly.
- **Multi-model endpoint (MME)**: An endpoint that loads many models into one shared container on demand.

## N

- **Network isolation**: A model setting that blocks all network access from the container.
- **NoCapacityInvocationFailures**: A metric whose alarm drives scaling an inference component up from zero.
- **NVIDIA Triton Inference Server**: The serving container used for GPU multi-model endpoints.

## O

- **One-hot encoding**: Turning each category value into its own binary column.
- **OpenSearch Compute Unit (OCU)**: The capacity unit billed by OpenSearch Serverless for indexing and search.
- **OverheadLatency**: The time SageMaker AI adds to a request on top of model latency.

## P

- **ParallelizationFactor**: An event source mapping setting that lets Lambda process up to 10 batches per shard at once.
- **Partial dependence plot (PDP)**: A chart of how the average prediction changes as one feature varies.
- **Partition key (Kinesis)**: The value hashed to choose a shard for each record; low cardinality creates hot shards.
- **pgvector**: A PostgreSQL extension that stores vectors and runs similarity search alongside relational data.
- **Pipe mode**: An input mode that streams data to the script through a Unix named pipe.
- **Point-in-time correct join**: A join that matches each label to the feature values that existed at or before the label's time.
- **Point-in-time recovery (PITR)**: Continuous DynamoDB backups, required before a table can be exported to S3.
- **PR AUC**: The area under the precision-recall curve, suited to rare positive classes.
- **Priority tier**: A premium Bedrock service tier for latency-critical requests.
- **Privileged mode**: A CodeBuild setting required for the Docker daemon to run.
- **Profile job**: The DataBrew job type that evaluates an attached data quality ruleset.
- **Prompt caching**: Reusing a repeated prompt prefix across requests to lower cost and latency.
- **Provisioned concurrency**: Pre-initialised serverless workers that remove cold starts.
- **Provisioned Throughput**: Reserved Bedrock model units that give committed capacity for one model in one Region.
- **push_down_predicate**: A Glue read option that prunes partitions in the Data Catalog before any data is read.

## Q

- **QLoRA**: LoRA training on a base model loaded in 4-bit quantized form to cut GPU memory.
- **Quantization**: Storing model weights at lower precision, such as 4-bit, to reduce memory.
- **Query decomposition**: Splitting a multi-part question into sub-queries for retrieval.

## R

- **Random Cut Forest (RCF)**: An unsupervised built-in algorithm that gives each record an anomaly score.
- **Real-time endpoint**: A SageMaker AI endpoint on instances you choose that answers requests synchronously.
- **Record identifier**: The feature that names which entity a Feature Store record describes.
- **Reinforcement fine-tuning**: Improving a model using a reward function that grades its outputs.
- **requestMetadata**: Caller-supplied key-value tags on Bedrock calls that appear in invocation logs.
- **Reranker**: A model that rescores retrieved candidates against the query more precisely than vector similarity.
- **Retrieval Augmented Generation (RAG)**: Retrieving relevant passages at query time and giving them to the model to ground its answer.
- **Return of control**: An action group mode in which the agent returns parameters for the application to execute.
- **RMSE**: Root mean squared error, which weights large errors more heavily than MAE.
- **Robust scaling**: Rescaling with the median and interquartile range so outliers do not dominate.
- **ROUGE**: A recall-oriented n-gram or subsequence overlap metric used for summarization.

## S

- **S3 Intelligent-Tiering**: A storage class that moves objects between access tiers automatically, with no retrieval fees.
- **S3 Object Lock**: A setting that blocks deletion or overwrite of object versions for a retention period, in governance or compliance mode.
- **SageMaker AI training plan**: A reservation of accelerated instances for a time window and one target resource.
- **SageMaker Feature Store**: A managed store of feature groups with an online store for serving and an offline store for history.
- **SageMaker Inference Recommender**: A tool that load-tests a model on candidate instances and reports latency, throughput and cost.
- **SageMaker JumpStart**: A hub of pre-trained models and solution templates that can be fine-tuned and deployed in SageMaker AI.
- **SageMaker Model Monitor**: A service that compares captured endpoint traffic with a baseline on a schedule; closed to new customers.
- **SageMaker Model Registry**: The catalog of model package versions with metrics, lineage and approval status.
- **SageMaker Pipelines**: A serverless ML workflow service with steps, parameters, caching and lineage.
- **SageMaker Savings Plans**: An hourly spend commitment that discounts eligible SageMaker AI usage for 1 or 3 years.
- **SBOM Generator**: An Inspector tool that produces a software bill of materials from a built image.
- **scale_pos_weight**: An XGBoost hyperparameter that increases the weight of positive examples in binary problems.
- **Script mode**: Running your own training script inside an AWS-managed framework container.
- **SEARCH expression**: A CloudWatch metric math query that picks up matching metrics, including new resources, at load time.
- **Semantic chunking**: Splitting text where its meaning changes, controlled by a breakpoint percentile threshold.
- **Serial inference pipeline**: A model made of 2 to 15 containers that every request passes through in order.
- **Serverless inference**: A SageMaker AI option that runs CPU models per request, scales to zero and may cold start.
- **Service control policy (SCP)**: An organization policy that sets the maximum permissions for member accounts.
- **Shadow variant**: A model variant that receives copied live traffic but whose responses are never returned.
- **SHAP**: Shapley additive explanations, which attribute a prediction to each feature's signed contribution.
- **ShardedByS3Key**: A data distribution setting that gives each training instance a different subset of the S3 objects.
- **SM_MODEL_DIR**: The path (/opt/ml/model) where a training script must save the model so SageMaker AI uploads it.
- **SMOTE**: A technique that creates synthetic minority-class rows by interpolating between nearby minority rows.
- **SplitType**: A batch transform setting that tells the job where record boundaries are in each file.
- **Stacking**: Training a meta-model on several models' predictions to learn how much to trust each.
- **Standardization**: Rescaling a feature to mean 0 and variance 1.
- **StartIngestionJob**: The API that syncs a knowledge base data source incrementally.
- **stopReason**: The field in a stream's messageStop event that says why generation ended.
- **Stratified split**: A split that keeps the same class mix in every subset.
- **Supervised fine-tuning**: Updating a model on labeled prompt-response pairs.

## T

- **Target tracking**: An auto scaling policy that holds a metric near a target value.
- **TargetVariant**: An InvokeEndpoint parameter that sends one request to a named production variant.
- **Tensor parallelism**: Splitting each layer's weights across several GPUs so a large model fits.
- **TimeToFirstToken**: A Bedrock metric for the delay before the first token of a streaming response.
- **Tokenization**: Splitting text into the subword units a model reads and counts against its limits.
- **Trainium**: An AWS accelerator built mainly for training and fine-tuning.
- **Tumbling window**: Fixed, non-overlapping time windows, available on Lambda event source mappings for up to 15 minutes.

## U

- **update_endpoint**: The API that moves an endpoint to a new configuration without changing its name.
- **UpdateEndpointWeightsAndCapacities**: An API that changes variant weights and instance counts on a running endpoint.

## V

- **VPC only mode**: A SageMaker AI domain setting that routes all Studio traffic through your VPC.
- **VpcConfig**: The subnets and security groups set on a SageMaker AI model so it runs in your VPC.

## W

- **Warm pool**: Training instances kept provisioned between jobs to cut start-up time.
- **Warm start**: Starting a tuning job from the results of earlier tuning jobs.
- **Watermark**: A Spark Structured Streaming setting that bounds how late an event may arrive and still be counted.

## X

- **XGBoost**: A gradient-boosted tree algorithm and a strong default for tabular data.

[← Back to the study guide](README.md)
