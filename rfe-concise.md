# Hardware Profiles with Accelerator Type Awareness

**Feature Overview:**

Hardware Profiles with Accelerator Type Awareness enables the platform to detect and expose specific GPU types (NVIDIA H200, A100, etc.) instead of treating all GPUs as generic resources. This solves a critical architectural blocker: model serving teams cannot integrate model catalog and validation metrics without knowing which hardware types are in hardware profiles, and Training cannot provide training estimation without accelerator type information. The solution leverages existing NVIDIA annotations on OpenShift nodes for automatic detection.

**Goals:**

Enable model serving automation by exposing accelerator types via API, allowing model catalog to automatically match models to appropriate hardware profiles without manual admin intervention. Enable DT team to provide GPU recommendations with training time estimates (e.g., "H200: 2 hours, A100: 5 hours"). Improve user experience by showing exact GPU types ("NVIDIA H200 141GB" vs generic "GPU") and reduce admin burden through node view UI that eliminates manual node labeling.

**Out of Scope:**

AI-powered GPU recommendation models, real-time cost estimation, performance benchmarking databases, multi-GPU topology awareness, Kubernetes Lens-style full node management UI, and fractional GPU sharing (MIG) are explicitly out of scope for this MVP.

**Requirements:**

1. **Model Serving: Performance-Based Hardware Recommendations [MVP]**: Given a model and use case (inference, fine-tuning), model catalog shall recommend hardware profiles to achieve service level objectives (throughput, latency, cost). Model catalog's performance insights (e.g., "llama-3-70b: 120 tokens/sec on H200, 75 tokens/sec on A100") shall inform recommendations, enabling users to select hardware based on their specific SLOs.

2. **Model Serving: Automatic Model-to-Hardware Matching [MVP]**: Hardware profiles shall expose accelerator metadata (GPU type, memory, architecture) enabling model catalog to programmatically determine compatible hardware profiles, eliminating manual administrator mapping of models to hardware.

3. **Training: Time Estimation for Hardware Selection [MVP]**: When selecting hardware profiles for workbench/training, users shall see training time estimates for each option (e.g., "H200: ~2 hours, A100: ~5 hours, V100: ~18 hours") based on model characteristics and hardware capabilities, enabling informed hardware selection.

4. **Training: Cost-Performance Trade-off Visibility [MVP]**: Users selecting training hardware shall see both time estimates and cost implications (e.g., "H200: 2 hours at $10.50 vs A100: 5 hours at $15.90"), enabling optimization for speed or cost based on priorities.

5. **Accelerator Type Awareness in Hardware Profiles [MVP]**: Hardware profiles shall expose specific accelerator types (e.g., "NVIDIA H200", "NVIDIA A100 80GB") rather than generic "GPU", enabling both model serving and training use cases above.

6. **Hardware Profile Discovery [MVP]**: Administrators shall discover cluster accelerator types and associate hardware profiles with them; users shall see specific accelerator types when selecting profiles (e.g., "Premium GPU (H200 141GB) - 4 available").

7. **Workload Scheduling to Correct Accelerators [MVP]**: Workloads (serving, training, workbench) shall schedule on nodes with specified accelerator types without manual node selector configuration, preventing accelerator type mismatches.

8. **Cost Attribution [MVP]**: Running workloads shall be annotated with actual accelerator type used (e.g., "nvidia-h200-141gb"), enabling accurate cost chargeback per team and audit trails.

9. **Backward Compatibility [MVP]**: Existing hardware profiles without accelerator types shall continue functioning as generic GPU profiles without migration or breaking changes.

10. **Performance Characteristics Query API [MVP]**: Hardware profiles shall expose accelerator performance characteristics queryable by external engines (model catalog, training estimators) with <100ms latency, supporting integration without hardcoded assumptions.

**Done - Acceptance Criteria:**

Model serving team can query hardware profile API and get accelerator metadata, enabling automatic model-to-hardware matching without admin intervention. DT team recommendation engine returns training time estimates for each GPU type. Platform admins use Node View UI to see all cluster nodes with detected GPU types, select nodes, and auto-create hardware profiles in under 5 minutes without manual node labeling. Data scientists see specific GPU types in spawn UI (e.g., "H200 141GB - 4 available") and workloads schedule correctly. Existing hardware profiles continue to work unchanged.

**Use Cases - i.e. User Experience & Workflow:**

**Model Serving Integration**: Model catalog deploys "llama-3-70b" requiring 80GB+ GPU; queries hardware profile API; receives accelerator metadata showing H200 and A100 available; automatically selects H200 (optimal for Hopper architecture); deployment succeeds with validation metrics flowing through—no manual admin mapping required.

**GPU Recommendations**: Data scientist configures GPT-2 training job; DT recommendation engine queries hardware profiles; returns estimates "H200: 2.1 hours ($10.50 total), A100: 5.3 hours ($15.90 total)"; data scientist selects H200 for best time-to-result; training completes as estimated.

**Node View Profile Creation**: Admin opens Node View, sees 24 nodes with detected GPU types (16x A100, 4x H200, 4x V100); filters to H200 nodes; selects all 4 H200 nodes; clicks "Create Hardware Profile"; system auto-fills form with "NVIDIA H200 141GB" and configures node selectors; profile created in under 5 minutes.

**Documentation Considerations:**

New documentation required: User guide for GPU selection, Admin guide for Node View UI and NVIDIA annotation detection, API documentation for model serving integration, and migration guide for existing deployments. Update existing docs for hardware profiles, notebook launching, and cluster configuration. Primary risk is NVIDIA annotation format variations across OpenShift versions; mitigation through comprehensive testing with design partners.

**Questions to answer:**

**Detection Strategy**: Should we require NVIDIA GPU Operator (automatic detection) or support manual GPU type entry when annotations missing? Recommendation: Support both with automatic preferred.

**Node View UI Scope**: Full Kubernetes Lens-style node management or focused view for hardware profile creation only? Recommendation: Focused view to avoid over-scoping MVP.

**Model Serving API Pattern**: REST API, GraphQL, direct CRD access, or event-driven? Needs decision from Model Serving team on which pattern fits their architecture.

**Accelerator Granularity**: Display "NVIDIA A100" (coarse), "NVIDIA A100-SXM4-80GB" (detailed), or "NVIDIA A100 80GB" (hybrid)? Recommendation: Hybrid for balance of accuracy and UX simplicity.

**Hardware Profile Versioning**: How should model serving handle profile changes after model deployment? Recommendation: Versioned profiles or manual migration requirement to avoid breaking deployed models.

**Background & Strategic Fit:**

This feature unblocks critical internal dependencies: Model Serving team cannot fully integrate model catalog and validation metrics without hardware profile accelerator awareness (identified by Jenny Yi), and DT team cannot provide GPU recommendations without knowing available accelerator types. The solution is architecturally feasible because NVIDIA already provides annotations on OpenShift nodes. Industry context: GPU costs are 60-70% of AI infrastructure spend, heterogeneous GPU adoption grew 320% YoY, and competitors (AWS SageMaker, Azure ML) all provide accelerator-specific visibility. Strategic alignment: Enables 2025-2026 roadmap goals for model serving automation, cost management, and self-service user experience.

**Customer Considerations:**

Large enterprises need cost attribution and governance (which teams access which GPUs). AI startups need simplicity and auto-discovery to avoid manual setup with small platform teams. Research institutions need fair-share access to premium GPUs and support for legacy hardware. Regulated industries need audit trails and controlled rollout processes. All customer segments benefit from eliminating manual node labeling requirement through automatic NVIDIA annotation detection. Design partner program with 2-3 customers per segment will validate solutions before GA.
