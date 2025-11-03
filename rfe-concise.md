# Hardware Profiles with Accelerator Type Awareness

**Feature Overview:**

Hardware Profiles with Accelerator Type Awareness enables the platform to detect and expose specific GPU types (NVIDIA H200, A100, etc.) instead of treating all GPUs as generic resources. This solves a critical architectural blocker: model serving teams cannot integrate model catalog and validation metrics without knowing which hardware types are in hardware profiles, and Training cannot provide training estimation without accelerator type information. The solution leverages existing NVIDIA annotations on OpenShift nodes for automatic detection.

**Goals:**

Enable model serving automation by exposing accelerator types via API, allowing model catalog to automatically match models to appropriate hardware profiles without manual admin intervention. Enable DT team to provide GPU recommendations with training time estimates (e.g., "H200: 2 hours, A100: 5 hours"). Improve user experience by showing exact GPU types ("NVIDIA H200 141GB" vs generic "GPU") and reduce admin burden through node view UI that eliminates manual node labeling.

**Out of Scope:**

AI-powered GPU recommendation models, real-time cost estimation, performance benchmarking databases, multi-GPU topology awareness, Kubernetes Lens-style full node management UI, and fractional GPU sharing (MIG) are explicitly out of scope for this MVP.

**Requirements:**

- **Accelerator Type Awareness in Hardware Profiles [MVP]**: Hardware profiles shall be aware of and expose specific accelerator types (e.g., "NVIDIA H200 141GB", "NVIDIA A100 80GB") rather than treating all GPUs as generic resources. This enables external systems (model catalog, training estimators) to query accelerator metadata for hardware recommendations and time estimations.

- **Enable Model Serving Hardware Recommendations [MVP]**: Hardware profiles shall expose sufficient accelerator metadata (GPU type, memory capacity, architecture) to enable model catalog to programmatically match models to compatible hardware profiles based on performance insights and service level objectives, eliminating manual administrator mapping.

- **Enable Training Time Estimation [MVP]**: Hardware profiles shall expose accelerator performance characteristics (GPU type, architecture, compute capabilities) to enable external training time estimation engines to calculate and present training duration estimates when users select hardware profiles for workbench or training jobs.

**Done - Acceptance Criteria:**

- Model serving team can query hardware profile API and get accelerator-type, enabling automatic model-to-hardware recommendation without admin intervention.
- Training hub can query HWP API returns training time estimates for each GPU type.
- Data scientists see specific GPU types in spawn UI (e.g., "H200 141GB - 4 available") and workloads schedule correctly. (Open Question TBD in Refinement)
- Existing hardware profiles features continue to work unchanged.

**Use Cases - i.e. User Experience & Workflow:**

**Model Serving Integration**: Model catalog deploys "llama-3-70b" requiring 80GB+ GPU; queries hardware profile API; receives accelerator metadata showing H200 and A100 available; automatically recommends H200 (optimal for Hopper architecture); deployment succeeds with validation metrics flowing through—no manual admin mapping required.

**Training Time Estimator**: Data scientist configures llama-3-70b training job with H200; returns estimates "H200: 2.1 hours, A100: 5.3 hours "; data scientist selects H200 for best time-to-result; training completes as estimated.


**Documentation Considerations:**

New documentation required: User guide for GPU selection, Admin guide for Node View UI and NVIDIA annotation detection, API documentation for model serving integration, and migration guide for existing deployments. Update existing docs for hardware profiles, notebook launching, and cluster configuration. Primary risk is NVIDIA annotation format variations across OpenShift versions; mitigation through comprehensive testing with design partners.

**Questions to answer:**
- Should non-admin users be able to actually see the hardware specs, OR should it be abstracted from them? Aka, query/get specs behind the scenes, UI only shows estimated training time or "recommended for serving".

**Background & Strategic Fit:**

This feature unblocks critical internal dependencies: Model Serving team cannot fully integrate model catalog and validation metrics without hardware profile accelerator awareness (identified by Jenny Yi), and DT team cannot provide GPU recommendations without knowing available accelerator types. The solution is architecturally feasible because NVIDIA already provides annotations on OpenShift nodes. Industry context: GPU costs are 60-70% of AI infrastructure spend, heterogeneous GPU adoption grew 320% YoY, and competitors (AWS SageMaker, Azure ML) all provide accelerator-specific visibility. Strategic alignment: Enables 2025-2026 roadmap goals for model serving automation, cost management, and self-service user experience.

**Customer Considerations:**

Large enterprises need cost attribution and governance (which teams access which GPUs). AI startups need simplicity and auto-discovery to avoid manual setup with small platform teams. Research institutions need fair-share access to premium GPUs and support for legacy hardware. Regulated industries need audit trails and controlled rollout processes. All customer segments benefit from eliminating manual node labeling requirement through automatic NVIDIA annotation detection. Design partner program with 2-3 customers per segment will validate solutions before GA.
