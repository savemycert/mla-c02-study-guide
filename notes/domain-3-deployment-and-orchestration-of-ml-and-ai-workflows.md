# Domain 3: Deployment and Orchestration of ML and AI Workflows (24%)

This domain covers where models, foundation models and agents run, how the capacity behind them is provisioned, networked and scaled, and how releases are automated, tested, versioned and rolled back. Questions describe a model and its traffic, payload size, team skills and budget, or an existing architecture with a symptom. One constraint stated mid-scenario usually decides it: a synchronous caller, no idle cost, a fixed instance type, no capacity commitment, no outbound internet, no custom code, or existing Kubernetes tooling.

## 3.1 Manage deployment infrastructure for ML and AI model types

- **SageMaker AI inference options:**

| Option                 | Called with                           | Fits                                                               | Does not fit                                    |
| ---------------------- | ------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------- |
| Real-time endpoint     | InvokeEndpoint, synchronous           | Steady traffic, low consistent latency, GPU models                 | Long idle periods (bills while idle)            |
| Serverless inference   | InvokeEndpoint, synchronous           | Intermittent traffic, cold start acceptable; memory 1–6 GB         | GPUs, strict latency every call, large payloads |
| Asynchronous inference | InvokeEndpointAsync, S3 InputLocation | Payloads up to 1 GB, processing up to about an hour, scale to zero | A caller needing the answer in the same request |
| Batch transform        | Job over an S3 prefix                 | Whole dataset on a schedule                                        | Any request-by-request need                     |

- **Confusions:** serverless and async both cost nothing idle, but only serverless answers synchronously, and only async takes large payloads, long runs and GPUs. Async and batch both use S3, but async handles requests as they arrive while batch scores a fixed dataset.
- **Async config:** payload goes to S3, `InvokeEndpointAsync` returns an OutputLocation; success and error **SNS topics in AsyncInferenceConfig** report completion without polling.
- **Batch transform:**
  - `SplitType=Line` for CSV/JSON Lines, otherwise each file is one request (payload too large).
  - `BatchStrategy=MultiRecord` packs records up to `MaxPayloadInMB`.
  - Work splits **by file**, so one huge file uses one instance. Split the data into several files.
  - `InputFilter` → `JoinSource=Input` → `OutputFilter`, in that order, to drop an ID before inference and keep it in the output.
- **Compute targets:**

| Family                  | Use when                                                      | Watch for                                   |
| ----------------------- | ------------------------------------------------------------- | ------------------------------------------- |
| ml.inf2 (Inferentia2)   | Lowest cost per inference for Neuron-compatible deep learning | Custom CUDA operators rule it out           |
| ml.trn1/trn2 (Trainium) | Training and fine-tuning                                      | "Purpose-built for inference" = Inferentia  |
| ml.c7g/m7g (Graviton)   | CPU-bound models, better price-performance                    | Container must be arm64                     |
| ml.c/m (x86)            | Small classical models                                        | Tight latency on large deep learning models |
| ml.g5/g6/g6e            | Cost-effective GPU inference                                  | Right-size GPU count and memory             |
| ml.p4d/p5               | Largest models, highest throughput                            | Usually the over-provisioned distractor     |

- **Hosting target:** SageMaker AI endpoint for least effort. **EKS** when the organization requires its Kubernetes tooling. **ECS** on EC2 for GPUs (Fargate has none). **Lambda** for small CPU models (15-minute limit). EC2 Auto Scaling only when full control is required.
- **Several models on one endpoint:**

| Strategy                   | Hosts                                                  | Routing                                        | Best for                                         |
| -------------------------- | ------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------ |
| Multi-model endpoint (MME) | Many models sharing one container                      | `TargetModel`; loaded from S3, cached, evicted | Hundreds of similar, rarely used models          |
| Multi-container (Direct)   | Up to 15 different containers                          | `TargetContainerHostname`                      | A few low-traffic models on different frameworks |
| Serial inference pipeline  | 2–15 containers as one model                           | Every request through all, in order            | Preprocess → predict → postprocess               |
| Inference components       | Several models with declared CPU, memory, accelerators | Component name; each scales its own copies     | Sharing GPUs with guaranteed resources           |

- MME tuning: frequent unloads → more instance memory; a hot model → its own endpoint. GPU MMEs use **Triton**, one archive per model with `config.pbtxt`.
- **GPU sizing for an FM:** weights ≈ parameters × bytes (FP16 ≈ 2 bytes, so 7B ≈ 14 GB, 34B ≈ 68 GB) plus KV cache headroom.
  - **Tensor parallelism** (`OPTION_TENSOR_PARALLEL_DEGREE` in LMI) must equal the GPU count, or the other GPUs' memory is wasted.
  - **Quantization** (4-bit AWQ/GPTQ ≈ a quarter of FP16) when the instance is fixed.
  - Batching settings raise throughput and KV memory; they never make a model fit. Disk is not GPU memory.
- **FM deployment options:**

| Option                         | Billing            | Fits                                                                              |
| ------------------------------ | ------------------ | --------------------------------------------------------------------------------- |
| Bedrock on-demand              | Per token          | Low or unpredictable traffic                                                      |
| Cross-Region inference profile | On-demand          | Peak throttling; **geographic** keeps data in that geography, **global** does not |
| Provisioned Throughput         | Hourly model units | Steady heavy traffic; required for some custom models                             |
| Bedrock batch inference        | Discounted         | Offline JSONL jobs in S3                                                          |
| Bedrock Marketplace            | Instance-hours     | Specialised models on endpoints you size, still via Bedrock APIs                  |
| SageMaker JumpStart            | Instance-hours     | Open-weight models you control                                                    |

- **External models:**
  - **Custom Model Import**: supported architectures (Llama, Mistral, Mixtral, Flan-T5, Qwen families) in Hugging Face format (`.safetensors`, `config.json`, tokenizer). Not `.pth`, not embedding models, not with batch inference.
  - **SageMaker AI prebuilt container**: `model.tar.gz` with weights plus `inference.py` (`model_fn`, `input_fn`, `predict_fn`, `output_fn`). Least effort for standard frameworks.
  - **BYOC**: ECR image answering `/invocations` and `/ping` on port 8080.
- **Agents:**
  - **Bedrock Agents** (configured; Bedrock runs the loop) vs **AgentCore Runtime** (your framework code: LangGraph, Strands Agents, CrewAI; isolated sessions up to hours).
  - **Action groups**: OpenAPI schema or function details, executed by Lambda. **RETURN_CONTROL** sends parameters back to your application instead (no Lambda). **User confirmation** enforces a confirm step; a prompt instruction does not.
  - Private API: put the action group's Lambda in VPC subnets that can reach it. A Bedrock VPC endpoint does not help.
  - **MCP** connects agents to tools (AgentCore Gateway turns REST APIs and Lambda into MCP tools). **A2A** connects agent to agent.
- **RAG retrieval settings:**

| Symptom                                       | Setting                                                          |
| --------------------------------------------- | ---------------------------------------------------------------- |
| Right chunk ranked just below the cut-off     | Retrieve a wider set, **rerank**, pass the top few               |
| Answers use documents the user should not see | **Metadata filter** on the query                                 |
| Multi-part questions answered one-sidedly     | **Query decomposition** (RetrieveAndGenerate orchestration only) |
| Too little context                            | More results (more tokens unless reranked)                       |

- Reranking: knowledge base reranking configuration (Amazon Rerank, Cohere Rerank) on Retrieve or RetrieveAndGenerate; for your own OpenSearch pipeline, the **Bedrock Rerank API**.
- **Retrieve** returns chunks for your own prompt and model; **RetrieveAndGenerate** returns a cited answer but you lose prompt control.

📖 Full lesson: [Deploying ML Models, Foundation Models and Agents on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-bedrock-model-deployment-inference-strategies/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 3.2 Provision and configure resources for ML and AI workloads based on existing architecture and requirements

- **On-demand vs provisioned:** on-demand for bursty, idle or uncommitted traffic; provisioned for steady, high, latency-sensitive traffic.
  - Only **Provisioned Throughput** reserves Bedrock capacity for one workload. Terms of 1 or 6 months (custom and imported models can be no-commitment, hourly). Invoke the **provisioned model ARN**, not the base model ID.
  - **Serverless provisioned concurrency** removes cold starts; Application Auto Scaling can schedule it or track `SageMakerVariantProvisionedConcurrencyUtilization`. Its scalable target floors at **1**, not 0.
- **Cross-stack values:**

| Mechanism                             | Watch out for                                                   |
| ------------------------------------- | --------------------------------------------------------------- |
| Export + `Fn::ImportValue`            | Producer cannot change or delete an export while it is imported |
| Parameter Store `{{resolve:ssm:...}}` | No lock; consumers pick up changes on next deploy               |
| Nested stack outputs                  | Merges ownership into one stack tree                            |
| `Fn::GetAtt` / `Ref`                  | Same template only                                              |

- **Step Functions with SageMaker AI:** `.sync` exists only for **jobs** (training, processing, transform, tuning, labeling); `.waitForTaskToken` is not supported. CreateEndpoint and UpdateEndpoint are request-response, so wait with a **Wait → DescribeEndpoint → Choice** loop (Failed → Fail state).
- **Container contract:** inference containers answer `GET /ping` and `POST /invocations` on **port 8080**; artifacts extract to `/opt/ml/model`. Training: data in `/opt/ml/input/data/<channel>`, config in `/opt/ml/input/config`, save the model to `/opt/ml/model`, checkpoints in `/opt/ml/checkpoints`.
- **Customizing images:** `requirements.txt` (Python only, needs a package repository at startup such as CodeArtifact via VPC endpoint) → extend the AWS image `FROM` it (OS packages, baked in, no internet needed) → build from scratch (you maintain everything).
- **ECR upkeep:** basic scanning (OS packages), **enhanced scanning** with Inspector (OS and language packages, continuous), **lifecycle policies** expire images, tag immutability prevents overwrites, replication copies.
- **Endpoints in a VPC:** `VpcConfig` goes on the **model** (CreateModel).

| Need                          | Endpoint service    | Type                   |
| ----------------------------- | ------------------- | ---------------------- |
| S3 artifacts or data          | `s3`                | Gateway (or interface) |
| DynamoDB                      | `dynamodb`          | Gateway (or interface) |
| InvokeEndpoint                | `sagemaker.runtime` | Interface              |
| CreateModel, DescribeEndpoint | `sagemaker.api`     | Interface              |
| Bedrock InvokeModel, Converse | `bedrock-runtime`   | Interface              |
| Bedrock control plane         | `bedrock`           | Interface              |

- Rules: only S3 and DynamoDB have gateway endpoints; runtime and control-plane endpoints are separate; default SDK hostnames need **private DNS** enabled; the endpoint security group must allow 443 from callers.
- **EnableNetworkIsolation** (CreateModel) cuts all container network access; SageMaker AI still pulls the image and artifacts. Use it for untrusted vendor code.
- **Programmatic deployment:** `create_model` → `create_endpoint_config` → `create_endpoint`. Configurations are immutable: new model + new config + **`update_endpoint`** keeps the same name with no downtime. Wait with the **`endpoint_in_service`** waiter on the endpoint name. The SageMaker Python SDK `model.deploy()` wraps all three.
- **Auto scaling metrics:**

| Metric                                                    | Use when                           |
| --------------------------------------------------------- | ---------------------------------- |
| `SageMakerVariantInvocationsPerInstance`                  | Short, uniform requests            |
| `...ConcurrentRequestsPerModelHighResolution` (10-second) | Long or streaming LLM requests     |
| `SageMakerVariantProvisionedConcurrencyUtilization`       | Serverless provisioned concurrency |
| `ApproximateBacklogSizePerInstance` (customized)          | Asynchronous endpoints             |
| `GPUUtilization`, `CPUUtilization` (customized)           | Request cost varies widely         |

- Namespaces: invocation metrics in `AWS/SageMaker`; instance metrics in `/aws/sagemaker/Endpoints`. Endpoint-wide totals and latency make poor targets.
- **Scale from zero:** target tracking cannot scale out from zero.
  - Inference components: managed instance scaling `MinInstanceCount=0`, scalable target min 0, plus a step policy on **NoCapacityInvocationFailures**. Only endpoints with inference components can scale real-time to zero.
  - Async: min 0, target tracking on backlog, step policy on **HasBacklogWithoutCapacity**.
- **GPU scaling:** inference components declare accelerators and memory per model; managed instance scaling adds instances when copies do not fit. Production variants are for A/B splits, not packing.
- **GPU capacity:** a quota error needs a quota increase; an **insufficient capacity** error does not. A **SageMaker AI training plan** reserves an instance type and count for a time window, for one target chosen at purchase (training jobs, HyperPod, inference endpoints or Studio apps). Spot and Savings Plans reserve nothing.
- **Bedrock knowledge base setup:**
  - Vector store choice as in Domain 1 and 2; Bedrock creates OpenSearch Serverless indexes with **faiss**.
  - Aurora: **RDS Data API** + **Secrets Manager secret**; the cluster can stay private.
  - Vector field dimension must equal the embedding model output (Titan V2: 1,024, 512 or 256); a mismatch fails every ingestion and needs a new index.
  - Parsing and chunking are fixed per data source. No chunking suits one-record-per-file sources.
  - Metadata only from `<file>.metadata.json` sidecars (not object tags). **StartIngestionJob** sync is incremental, including deletions.
- **Retrieval pipeline:** Retrieve (chunks and scores, your own generation or threshold), RetrieveAndGenerate, RetrieveAndGenerateStream. Tenant isolation = an **equals filter set on the backend from the authenticated identity**, never from the question; implicit filtering is not a security control. Aggregations → structured data store (Redshift) with generated SQL.
- **Agent infrastructure (AgentCore):**
  - **Bedrock Agents is now Agents Classic, closed to new customers**; new agents use AgentCore.
  - **Runtime**: `linux/arm64` image in ECR, `/invocations` and `/ping` on 8080; each session in its own microVM; idle timeout default 15 minutes, max lifetime up to 8 hours. A resumed session after that loses in-process state.
  - VPC mode has private IPs only; outbound internet needs private subnets with a NAT gateway route.
  - **Gateway** publishes shared MCP tools. **Identity**: inbound auth (IAM or JWT such as Cognito) and outbound OAuth2 with a **token vault**.
  - **Memory**: short-term **events** replay exact turns; long-term **strategies** (semantic facts, preferences, summaries) persist across sessions. No strategy = no long-term records.
  - Agents Classic: `sessionAttributes` last the session; `promptSessionAttributes` last one turn.

📖 Full lesson: [Provisioning ML Resources: Endpoints, Scaling, Knowledge Bases and Agents (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/sagemaker-endpoint-scaling-vpc-bedrock-knowledge-bases/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 3.3 Implement automated orchestration and continuous integration and continuous delivery (CI/CD) pipelines for MLOps and AI workloads

- **Two pipelines:** a model-build pipeline (SageMaker Pipelines) and a release pipeline (CodePipeline).

| Service               | Job                                                                    | Not                                   |
| --------------------- | ---------------------------------------------------------------------- | ------------------------------------- |
| CodePipeline          | Release stages, V2 triggers with filters, stage rollback               | An ML workflow engine                 |
| CodeBuild             | Tests, Docker builds, ECR pushes, scripts calling SageMaker or Bedrock | A server or deployment service        |
| CodeConnections       | Link to GitHub, GitLab, Bitbucket                                      | A repository                          |
| CodeCommit            | AWS-hosted Git; EventBridge rule starts the pipeline                   | Needed when code is already elsewhere |
| CodeDeploy            | Traffic shifting and rollback for EC2, **Lambda**, **ECS**             | Used for SageMaker AI endpoints       |
| SageMaker Pipelines   | Serverless ML DAG with lineage in Studio                               | A source-triggered release tool       |
| Step Functions / MWAA | General orchestration                                                  | Studio lineage                        |

- **CodeBuild troubleshooting:**
  - "Cannot connect to the Docker daemon" → enable **privileged mode**.
  - ECR auth error → service role needs `ecr:GetAuthorizationToken`, layer upload and `ecr:PutImage`.
  - Wrong architecture → `docker buildx build --platform` or an ARM CodeBuild environment.
- **CodeConnections:** connections created by CLI or CloudFormation stay **PENDING** until completed in the console. Default zip output needs `codeconnections:UseConnection` on the pipeline role; **full clone** also needs it on the CodeBuild role.
- **V2 triggers:** push filters by tags, or branches and file paths (`training/**`); pull-request filters by branch, path and event type. Filters stop an execution from starting at all.
- **CodeCommit:** pipeline never starts → EventBridge rule or its role (`codepipeline:StartPipelineExecution`). Source stage starts then fails → pipeline role's CodeCommit read actions.
- **SageMaker Pipelines** steps: Processing, Training, Tuning, Model, Transform, Condition, Fail, Lambda, Callback, QualityCheck, ClarifyCheck. Parameters for reuse, **step caching** to skip unchanged expensive steps, retry policies for transient errors, parallelism caps.
- **Training job settings:** Spot with **checkpoints** (`/opt/ml/checkpoints` synced to S3). `MaxWaitTimeInSeconds` ≥ `MaxRuntimeInSeconds`; set the wait to the deadline. Warm pools shorten start-up, not work.
- **Batch transform step** settings: SplitType, BatchStrategy, MaxPayloadInMB, AssembleWith (Line writes one record per line), MaxConcurrentTransforms, plus InputFilter/JoinSource/OutputFilter.
- **Test gates:**

| Gate                    | Where                          | Mechanism                                                                                    |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------- |
| Unit tests              | CodeBuild before the image     | pytest; non-zero exit fails                                                                  |
| Model metric threshold  | SageMaker Pipelines            | Evaluation JSON as **PropertyFile**, **ConditionStep** with JsonGet, **FailStep** on failure |
| Integration and latency | CodeBuild after staging deploy | Replay requests against staging                                                              |
| Kept results            | CodeBuild report groups        | JUnit XML                                                                                    |
| Human sign-off          | CodePipeline                   | **Manual approval** action, SNS notification                                                 |

- A rejected model must end the run as **Failed**. A test about the deployed endpoint belongs after the staging deploy.
- **Endpoint deployment guardrails:**

| Mode                   | Traffic                                 | Fits                                    |
| ---------------------- | --------------------------------------- | --------------------------------------- |
| Blue/green all at once | 100% to green, then bake                | Simple updates                          |
| Blue/green canary      | Small slice, bake, then the rest        | Limited blast radius                    |
| Blue/green linear      | Equal steps with baking                 | Gradual exposure                        |
| Rolling                | Batches (`MaximumBatchSize`) with waits | Quota or cost rules out two full fleets |

- Auto-rollback needs **CloudWatch alarms in AutoRollbackConfiguration** (for example Invocation5XXErrors, ModelLatency) and baking periods long enough to evaluate them. No alarms, no rollback. Shadow testing is validation, not rollback.
- CodePipeline V2 **stage rollback** reruns a stage with the last successful execution's revisions.
- **CodeDeploy for Lambda or ECS:** deployment group (target, role, config, alarms, rollback settings), **AppSpec** (Lambda alias and versions, or ECS task definition, container name and port, plus hooks such as BeforeAllowTraffic and AfterAllowTestTraffic), deployment configuration (for example `LambdaCanary10Percent5Minutes`, `ECSAllAtOnce`). Hooks must call `PutLifecycleEventHookExecutionStatus` or the deployment times out.
- **Re-training triggers:** EventBridge schedule targeting a SageMaker pipeline with parameters; a **CloudWatch alarm state change** on a Model Monitor drift metric (the event carries the decision); S3 object events for new data.
- **Knowledge base refresh:** `StartIngestionJob` on a schedule or from events, covering **deletions** too. A new embeddings model needs a new knowledge base, full ingestion, then switch the ID (switch back to roll back).
- **Model Registry:** model package groups hold versions with artifact, metrics, lineage and approval status (PendingManualApproval, Approved, Rejected). The **Model Package State Change** event to Approved starts the release. MLflow with **Model Registry sync** enabled creates packages automatically.
- **Prompts:** call a versioned prompt ARN ending in `:<version>` as the `modelId` in Converse; the unversioned ARN runs the draft. Request-level overrides of a managed prompt's config are rejected, so a setting change means a new version.
- **Evaluation gates:** a CodeBuild step runs a Bedrock evaluation job and fails on low scores. Model evaluation jobs bypass the knowledge base, so use RAG retrieve-only jobs to test retrieval.
- **Fine-tuned FMs:** invoke through a **custom model deployment ARN** (on-demand, supported models) or a **provisioned model ARN**. `UpdateProvisionedModelThroughput` swaps in a newer fine-tune of the same base without changing the ARN.
- **AgentCore versions:** every update creates an immutable version; the **DEFAULT** endpoint follows the latest; a custom endpoint (for example production) moves only with `UpdateAgentRuntimeEndpoint`, which is how you promote and roll back.

📖 Full lesson: [MLOps CI/CD Pipelines, Model Versioning and Rollback on AWS (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/mla-c02-mlops-cicd-pipelines-model-versioning/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

[← Back to the study guide](../README.md)
