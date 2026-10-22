# Domain 4: Operating, Monitoring, and Securing ML and AI Solutions (24%)

This domain covers what happens after deployment: detecting drift and quality problems in models, foundation models and agents, finding where time and money go, and locking down data, endpoints and AI applications. Questions describe a production system with a symptom (often backed by metric values or an error message) and ask which monitor, metric, tool, purchasing option or control fixes it. Typical deciding constraints: no ground-truth labels, labels arriving weeks later, users must never see a candidate's output, requests must stay in one Region, running work must not stop, no fixed thresholds, or a vulnerable image must never be stored.

## 4.1 Monitor ML and AI model inference and performance

- **Pick the tool from the failing signal:**

| Question                                      | Mechanism                                      | Needs                              |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------- |
| Latency, errors, throttles, tokens on Bedrock | `AWS/Bedrock` metrics; Model Invocations view  | Nothing (automatic)                |
| The exact prompt and response                 | Bedrock **model invocation logging**           | Logging turned on (off by default) |
| SageMaker endpoint inputs drifted             | Model Monitor data quality                     | Data capture + baseline            |
| Accuracy or recall dropped                    | Model Monitor model quality                    | Data capture + ground truth        |
| Bias or feature reliance changed              | Clarify bias drift / feature attribution drift | Data capture + Clarify baseline    |
| Is version B better on live users             | Production variants (A/B)                      | Two variants                       |
| Is version B safe with no user exposure       | Shadow test                                    | Copied traffic                     |
| Are FM or RAG answers good                    | Bedrock evaluations                            | A prompt dataset                   |
| Are agent sessions and tool calls good        | Agent trace; AgentCore Evaluations             | Trace or AgentCore observability   |

- **CloudWatch generative AI observability:** the Model Invocations view shows counts, tokens, latency percentiles, throttles and errors. Its **per-request table** fills only from invocation logs delivered to **CloudWatch Logs** (S3-only logging leaves it empty).
- **Bedrock metrics:**
  - **InvocationLatency** = whole response; **TimeToFirstToken** = slow start of a stream.
  - Quota rejections count as client errors, not InvocationThrottles.
  - **EstimatedTPMQuotaUsage** is a capacity signal.
  - Requests that fail before the model is resolved appear only in metrics **without the ModelId dimension**, so alarm on the dimensionless series too.
- Agents outside AgentCore Runtime: enable **CloudWatch Transaction Search** and instrument with the **ADOT SDK**.
- **Invocation logging:** per account and Region, to CloudWatch Logs, S3 or both. Bodies over **100 KB** need an S3 large-data location. Tag calls with `requestMetadata` for per-app filtering (a shared role makes `identity.arn` identical). Alarm on the delivery-failure metric for each destination. CloudTrail never holds prompt bodies.
- **Model Monitor** (SageMaker AI endpoints and batch transform only, never Bedrock) needs data capture, a baseline and a schedule.

| Monitor                   | Detects                                   | Baseline from                     | Labels? |
| ------------------------- | ----------------------------------------- | --------------------------------- | ------- |
| Data quality              | Input distributions, types, nulls, schema | Training data                     | No      |
| Model quality             | Accuracy, precision, recall, RMSE drop    | Predictions alongside true labels | Yes     |
| Bias drift (Clarify)      | Outcome gaps between facet values         | Clarify bias baseline             | Depends |
| Feature attribution drift | SHAP ranking changes (NDCG)               | Clarify SHAP baseline             | No      |

- Data quality checks: `data_type_check`, `completeness_check`, `missing_column_check`, `baseline_drift_check`. A value-scale shift with the same type and nulls trips only `baseline_drift_check`.
- **Delayed ground truth:** pass a unique **InferenceId** on InvokeEndpoint, upload labels keyed by it, and the model quality monitor merges and compares. The model quality baseline needs predictions (and probabilities) next to labels and the problem type; training features alone produce feature statistics only.
- **Automating responses:**
  - Drift → alarm on the drift metric → EventBridge rule on the **alarm state change** → SageMaker pipeline. A job finishing says nothing about drift.
  - Pipeline step or job **Failed** event → EventBridge → SNS.
  - Seasonal metrics → **anomaly detection band** alarm. Known error string → metric filter. New log patterns while jobs succeed → **CloudWatch Logs anomaly detection**. Top contributors → Contributor Insights.
- **A/B and shadow controls:**
  - Initial weights in the endpoint configuration (90/10).
  - **UpdateEndpointWeightsAndCapacities** changes weights in place, no new config. UpdateEndpoint needs a new config.
  - **TargetVariant** pins one request to a variant. **InvokedProductionVariant** in the response attributes outcomes to the variant that served them.
  - Shadow tests copy a sampling percentage of requests, return only production responses, and can promote the shadow variant.
- **GenAI input drift:** Model Monitor does not exist for Bedrock. Embed recent logged prompts on a schedule, compare with launch-period embeddings, publish a custom CloudWatch metric. Token counts only measure length.
- **Bedrock evaluations in production:** judge-based and RAG jobs accept **your own inference responses**, so logged answers are scored as delivered without re-invoking. Retrieve-only jobs isolate retrieval when comparing chunking. Custom judge metrics use `{{prompt}}` and `{{prediction}}` variables with a rating scale. Human jobs use a private Cognito work team; Amazon A2I is a per-prediction review loop, not a model comparison.
- **Agent monitoring:**
  - Enable **trace** on InvokeAgent: rationale, action group input, observation; multi-agent hand-offs appear as `AGENT_COLLABORATOR` inputs.
  - **Handled tool failures** never move Lambda Errors or agent error metrics. Emit a custom metric (embedded metric format) and alarm on it.
  - **Truncated streams:** HTTP 200 proves nothing. A complete ConverseStream ends with `messageStop` (stopReason) then a `metadata` event. `max_tokens` = cut by the limit; no `messageStop` = incomplete; only `end_turn` (or an expected `stop_sequence`) is a natural finish.
- **AgentCore Evaluations:** online evaluation samples live sessions (default 10%); on-demand scores sessions you supply. Evaluators by level: session (`GoalSuccessRate`), trace (`Helpfulness`), tool (`ToolSelectionAccuracy`, `ToolParameterAccuracy`). Scores land in the `Bedrock-AgentCore/Evaluations` namespace for alarms.

📖 Full lesson: [Monitoring ML Models, FMs and Agents in Production: Drift, A/B Tests and Bedrock Evaluations (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/ml-model-monitoring-drift-detection-bedrock-evaluations/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 4.2 Optimize and manage ML and AI infrastructure costs and performance

- **Instance family follows the bottleneck resource:**

| Metrics show                        | Family                                                         |
| ----------------------------------- | -------------------------------------------------------------- |
| CPU high, memory low, no GPU        | Compute optimised `ml.c*`                                      |
| Balanced                            | General purpose `ml.m*` (~4 GiB per vCPU)                      |
| Light compute, large in-memory data | Memory optimised `ml.r*` (~8 GiB per vCPU)                     |
| Needs a GPU                         | `ml.g4dn`, `g5`, `g6`, `g6e`, `p*`, chosen by GPU memory first |
| High-volume supported deep learning | Inferentia2 `ml.inf2`                                          |

- Remove an accelerator the model does not use first. One `m5.4xlarge` costs the same as two `m5.2xlarge`, so consolidating within a family saves nothing.
- **GPU memory:** FP16 ≈ 2 bytes per parameter (13B ≈ 26 GB plus KV cache). T4 (g4dn) 16 GB; A10G (g5) and L4 (g6) 24 GB; L40S (g6e) 48 GB. Larger single-GPU sizes add vCPUs and host RAM, not GPU memory. Just over 24 GB → the smallest 48 GB GPU is usually cheaper than multi-GPU.
- **Inference Recommender:** default job = quick instance shortlist (typically about 45 minutes). **Advanced load test** = your endpoint configurations, traffic phases and stopping conditions such as P95/P99 ModelLatency. Pre-deployment only.
- **Slow endpoint troubleshooting:**

| Metric                   | Namespace                  | Tells you                                 |
| ------------------------ | -------------------------- | ----------------------------------------- |
| ModelLatency             | `AWS/SageMaker`            | Time in the model container               |
| OverheadLatency          | `AWS/SageMaker`            | Time SageMaker AI adds                    |
| ModelSetupTime           | `AWS/SageMaker`            | Serverless cold start                     |
| CPU/MemoryUtilization    | `/aws/sagemaker/Endpoints` | Host CPU and RAM                          |
| GPU/GPUMemoryUtilization | `/aws/sagemaker/Endpoints` | GPU compute and whether weights still fit |

- If ModelLatency + OverheadLatency is small but users wait seconds, the time is elsewhere: enable **X-Ray active tracing** on API Gateway and Lambda.
- X-Ray sampling (default: first request each second plus 5%) is decided at the **entry service**; a rule on a downstream service never applies. Add a higher-priority rule matching the entry service and URL path.
- Serverless cold starts (ModelSetupTime spikes) → **provisioned concurrency**; more memory does not remove setup time.
- **AgentCore Observability** gives a per-session trace of every model call and tool step. Needs the **ADOT SDK** in the agent (started with `opentelemetry-instrument`; the ADOT Collector is not supported) and **Transaction Search** enabled once per account. Metrics without traces means Transaction Search is missing.
- **Dashboards:** plot **p90/p95/p99** for latency; **cross-account observability** links source accounts to one monitoring account; **SEARCH** expressions pick up new endpoints automatically. Useful Bedrock metrics include `CacheReadInputTokenCount` and `CacheWriteInputTokenCount`.
- **Capacity levers:**
  - Survive an AZ loss → **two or more instances** (spread across AZs); split one large instance into two smaller ones to keep cost flat.
  - Predictable daily peak → **scheduled actions** raising MinCapacity before the peak, alongside target tracking.
  - Many small rarely called models, same framework → multi-model endpoint.
  - No caller waiting → async (scale to zero). Intermittent CPU model → serverless.
- **Cost tools:**

| Need                             | Tool                                                                                     |
| -------------------------------- | ---------------------------------------------------------------------------------------- |
| Spend by team or project         | **Activated** cost allocation tags + Cost Explorer (backfill up to 12 months on request) |
| Per-team on-demand Bedrock usage | Tagged **application inference profiles**                                                |
| Warn before a monthly limit      | AWS Budgets **forecasted** alert                                                         |
| Act at a threshold               | **Budgets actions**: IAM policy, SCP, or stop EC2/RDS                                    |
| Unusual spend, no thresholds     | Cost Anomaly Detection (billing data lags)                                               |
| Unusual usage within minutes     | CloudWatch anomaly detection alarm on a usage metric                                     |
| Hard ceilings on throughput      | Service quotas                                                                           |

- Budgets actions cannot stop SageMaker AI resources; an IAM policy denying `sagemaker:CreateTrainingJob` and `sagemaker:CreateEndpoint` freezes new work without stopping running jobs. A shared role's principal tags cannot split teams.
- **Purchasing options:**

| Option                               | Covers                                                                                                                     |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| **SageMaker Savings Plans**          | SageMaker AI training, real-time inference, batch transform, processing, notebooks; any family, size, Region; 1 or 3 years |
| Compute Savings Plans                | EC2, Fargate, Lambda; **not** SageMaker AI                                                                                 |
| EC2 Instance SP / Reserved Instances | EC2 only                                                                                                                   |
| Managed Spot Training                | Interruptible training jobs                                                                                                |

- Size a commitment to the **steady hourly baseline**; unused commitment in any hour is lost. A commitment never fixes capacity that should not exist.
- **FM inference cost:**

| Option                                 | Fits                                                           |
| -------------------------------------- | -------------------------------------------------------------- |
| On-demand (Standard tier)              | Variable or low traffic                                        |
| **Flex tier**                          | Latency-tolerant synchronous calls, discounted, no code rework |
| Priority tier                          | Latency-critical traffic, premium price                        |
| Reserved tier / Provisioned Throughput | Steady high volume, billed whether used or not                 |
| Batch inference                        | Offline jobs moved to S3                                       |
| Self-hosting on SageMaker AI           | Sustained high utilisation or models not offered serverlessly  |

- A GPU endpoint at a few percent utilisation → move to Bedrock on-demand if the model is offered in the Region. Geographic and global inference profiles both leave a single Region.
- **Agent resource use (AgentCore Runtime):** metrics `CPUUsed-vCPUHours` and `MemoryUsed-GBHours` by endpoint. CPU is billed only while processing; **memory is billed until the session ends**. Sessions end on `StopRuntimeSession`, `idleRuntimeSessionTimeout` (default 900 s) or `maxLifetime` (default 28,800 s). Reuse one `runtimeSessionId` per conversation.
- **Token levers:**
  - Bedrock reserves **input + requested max output tokens** against the TPM quota until the response finishes. Oversized max tokens cause throttling at low real usage. Some models count output tokens at a higher burndown rate (for example 5x for recent Claude models).
  - **Prompt caching:** put the `cachePoint` after the static prefix. A checkpoint after per-request content gives writes and no reads. Below the model's minimum tokens (for example 1,024 for Claude Sonnet 4.5) nothing is cached.
  - Summarise older conversation turns; lower `numberOfResults` on Retrieve.
- **Embedding and vector cost:** sync incrementally (do not recreate data sources); bulk embedding through **batch inference** (Titan Text Embeddings V2 supports it); modest chunk overlap; S3 inclusion prefixes. OpenSearch Serverless: pool collections in one collection group and set minimum OCUs to 0. Shrink vectors with fewer dimensions or **binary embeddings** (re-embed and rebuild).

📖 Full lesson: [Optimising ML and GenAI Inference Cost and Performance on AWS: Instances, Savings Plans, Tokens and Vectors (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/aws-ml-genai-inference-cost-optimization/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

## 4.3 Secure ML and AI workloads and model endpoints

- **Scanning code and images:**

| Mechanism                                            | Runs                                   | Gives                               |
| ---------------------------------------------------- | -------------------------------------- | ----------------------------------- |
| ECR basic scanning                                   | On push or manual                      | OS package findings at that moment  |
| Inspector enhanced scanning, continuous              | On push, then whenever new CVEs apply  | OS and language package findings    |
| **Inspector SBOM Generator + Scan API** in CodeBuild | Inside the build, before push          | Fails the build; image never stored |
| **Inspector code security**                          | On changes in GitHub/GitLab, scheduled | SAST, SCA and IaC findings          |

- **CodeGuru Reviewer** stopped accepting new repository associations (7 Nov 2025) and **CodeGuru Security** was discontinued (20 Nov 2025); new setups use Inspector code security. Neither scanned image packages.
- **Least privilege on artifacts:**
  - `s3:GetObject` on the input prefix, `s3:PutObject` on the output prefix, `s3:ListBucket` on the **bucket ARN** with an `s3:prefix` condition. Avoid `s3:*`.
  - SSE-KMS readers need `kms:Decrypt` (writers `kms:GenerateDataKey`). Cross-account needs both the key policy and the role's identity policy.
  - Cross-account Model Registry: model package group **resource policy** + artifact read + `kms:Decrypt`.
- **Execution role vs caller:** the caller needs `sagemaker:Create*` plus **`iam:PassRole`** (scoped to one role ARN with `iam:PassedToService = sagemaker.amazonaws.com`). The **execution role** does everything the job does. PassRole AccessDenied at creation = caller; S3/KMS AccessDenied after start = execution role.
- **Preventive policies:** explicit Deny on SageMaker condition keys (`sagemaker:VpcSubnets`, `sagemaker:NetworkIsolation`, `sagemaker:InterContainerTrafficEncryption`, `sagemaker:VolumeKmsKey`, `sagemaker:OutputKmsKey`, `sagemaker:InstanceTypes`, notebook `DirectInternetAccess` and `RootAccess`).
  - An explicit Deny beats any Allow; a narrower Allow restricts nothing.
  - Conditions in one statement are ANDed, so write **one Deny per requirement**.
  - Negated operators (StringNotEquals) match a missing key; positive ones (Bool) need **IfExists** or a Null check.
  - Nothing in a member account overrides an SCP deny.
- **Network isolation settings:**

| Setting                            | Does                                                           | Does not                 |
| ---------------------------------- | -------------------------------------------------------------- | ------------------------ |
| VPC config                         | Places the job in your subnets                                 | Block outbound by itself |
| EnableNetworkIsolation             | Removes all container network access; channels still delivered | Encrypt anything         |
| Inter-container traffic encryption | Encrypts distributed training traffic                          | Restrict destinations    |
| VolumeKmsKeyId / KmsKeyId          | Encrypt volumes and output artifacts                           | Protect data in transit  |

- Under network isolation, stage downloads in S3 as an extra channel. No NAT and no S3 endpoint gives a **timeout**, not AccessDenied.
- **Studio VPC only mode** routes Studio through your subnets and needs interface endpoints (SageMaker API, runtime, STS and others).
- Bedrock has **interface endpoints only**: `bedrock-runtime`, `bedrock`, `bedrock-agent`, `bedrock-agent-runtime`. Deny inference unless `aws:SourceVpce` matches to make stolen credentials useless outside the VPC.
- **Auditing:**

| Event type        | Examples                                                                                                 | Default                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Management events | CreateTrainingJob, CreateEndpoint, Bedrock InvokeModel, Converse                                         | 90 days in event history; trail for longer |
| Data events       | SageMaker InvokeEndpoint; Bedrock InvokeAgent, Retrieve, RetrieveAndGenerate, InvokeFlow, ApplyGuardrail | Off; advanced event selectors on a trail   |

- CloudTrail records caller, time and resource, never prompts. CloudTrail Lake closed to new customers on 31 May 2026.
- **AWS Config** managed rules (detective) such as `sagemaker-notebook-no-direct-internet-access`, `sagemaker-endpoint-configuration-kms-key-configured`, `sagemaker-model-isolation-enabled`, `sagemaker-model-in-vpc`. Deploy organization-wide as a **conformance pack** with an aggregator. AWS Audit Manager cannot be set up in new accounts or Regions from 30 April 2026.
- **Troubleshooting:** failed `kms:Decrypt` in CloudTrail → key permission. "Explicit deny in a service control policy" → only the org admins can change it. Permissions boundary denial → change the boundary. Adding an Allow never fixes an explicit deny.
- **FM credentials:**

| Credential                   | Lifetime                                            | Use for                          |
| ---------------------------- | --------------------------------------------------- | -------------------------------- |
| IAM role credentials (SigV4) | Temporary, rotated automatically                    | Production apps on AWS           |
| Short-term Bedrock API key   | Up to 12 hours, one Region, principal's permissions | Key-only tools without IAM users |
| Long-term Bedrock API key    | Until expiry; tied to an IAM user                   | Exploration only                 |

- Govern keys: deny `iam:CreateServiceSpecificCredential` for `bedrock.amazonaws.com`; deny `bedrock:CallWithBearerToken` when `bedrock:bearerTokenType` is `LONG_TERM`.
- **Bedrock Guardrails policies:**

| Policy                        | Detects                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| Denied topics                 | A subject however it is phrased                              |
| Content filters               | Hate, insults, sexual, violence, misconduct                  |
| Prompt attack filter          | Jailbreaks, prompt injection, prompt leakage (Standard tier) |
| Word filters                  | Exact words, managed profanity list                          |
| Sensitive information filters | Built-in PII types and custom regex; block or mask           |
| Contextual grounding check    | Responses not grounded or not relevant (RAG)                 |

- With InvokeModel, the prompt attack filter evaluates only content inside the **guardrail input tags**.
- Guardrails do not clean logs: use a **CloudWatch Logs data protection policy** (only `logs:Unmask` sees originals).
- **ApplyGuardrail** gives a SageMaker-hosted model guardrails: check the prompt, invoke, check the response.
- **Enforce guardrails:** Allow inference when `bedrock:GuardrailIdentifier` equals the **versioned** guardrail ARN plus an explicit Deny otherwise. An unversioned ARN means DRAFT. Converse is authorized as InvokeModel. Do not use that role for RetrieveAndGenerate or InvokeAgent.
- **AI risks:**
  - Indirect prompt injection → **user confirmation** on the action, plus prompt attack filter on tagged input (it does not check tool results).
  - Excessive agency → least privilege on the action group's Lambda role.
  - Broken object-level authorization → pass the authenticated user ID in session attributes and filter on it in the tool.
  - Artifact tampering → least-privilege writes, S3 versioning, ECR tag immutability, Model Registry approval.

📖 Full lesson: [Securing ML and AI Workloads on AWS: IAM, VPC Isolation, Auditing and Bedrock Guardrails (MLA-C02)](https://www.savemycert.com/revision/aws-machine-learning-engineer-associate/secure-ml-ai-workloads-iam-vpc-bedrock-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=mla-c02-study-guide)

[← Back to the study guide](../README.md)
