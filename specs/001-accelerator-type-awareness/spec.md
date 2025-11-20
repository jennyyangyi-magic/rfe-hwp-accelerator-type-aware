# Feature Specification: Hardware Profiles with Accelerator Type Awareness

**Feature Branch**: `001-accelerator-type-awareness`
**Created**: 2025-11-20
**Status**: Draft
**Input**: User description: "Hardware Profiles with Accelerator Type Awareness enables the platform to detect and expose specific GPU types (NVIDIA H200, A100, etc.) instead of treating all GPUs as generic resources. This solves a critical user experience blocker: model serving cannot make use of model validation metrics for serving recommendations without knowing which hardware types are in hardware profiles, and Training cannot provide training estimation without accelerator type information."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Model Serving Hardware Recommendations (Priority: P1)

Model catalog teams can programmatically query hardware profiles to get specific GPU types and automatically recommend optimal hardware for model deployment based on validation metrics, eliminating manual administrator mapping.

**Why this priority**: This unblocks the Model Serving team's critical dependency on hardware profile accelerator awareness for integrating model catalog and validation metrics. Without this, model serving recommendations cannot function.

**Independent Test**: Can be fully tested by deploying a model through the model catalog API, which queries hardware profiles and receives accelerator metadata, then automatically selects the optimal GPU type for deployment.

**Acceptance Scenarios**:

1. **Given** model catalog has a "llama-3-70b" model requiring 80GB+ GPU memory, **When** deployment is initiated, **Then** hardware profile API returns accelerator metadata showing available H200 and A100 GPUs with memory specifications
2. **Given** hardware profile API returns multiple compatible GPU types, **When** model catalog evaluates validation metrics, **Then** system automatically recommends H200 (optimal for Hopper architecture) without administrator intervention
3. **Given** recommended hardware is selected, **When** deployment proceeds, **Then** workload schedules successfully on the selected GPU type and validation metrics flow through correctly

---

### User Story 2 - Training Time Estimation (Priority: P2)

Data scientists configuring training jobs can see estimated training durations for each available GPU type, enabling informed hardware selection based on time-to-result requirements.

**Why this priority**: This enables the Training team to provide GPU recommendations and helps data scientists make cost-effective hardware choices, but model serving integration has higher business impact.

**Independent Test**: Can be fully tested by configuring a training job in the workbench, selecting different GPU types, and verifying that time estimates are displayed and accurately reflect GPU performance characteristics.

**Acceptance Scenarios**:

1. **Given** data scientist is configuring a llama-3-70b training job, **When** hardware profile is queried, **Then** system displays training time estimates for each GPU type (e.g., "H200: 2.1 hours, A100: 5.3 hours")
2. **Given** training time estimates are displayed, **When** data scientist selects H200 for best time-to-result, **Then** training job is submitted with H200 hardware profile
3. **Given** training job completes, **When** actual duration is compared to estimate, **Then** completion time aligns with the provided estimate within acceptable variance

---

### User Story 3 - GPU Type Visibility for Users (Priority: P3)

Data scientists viewing the spawn UI can see specific GPU types with availability information instead of generic "GPU" labels, improving transparency and enabling informed hardware selection.

**Why this priority**: This improves user experience and transparency but is lower priority than enabling critical downstream integrations. The exact UI presentation requires further refinement.

**Independent Test**: Can be fully tested by launching the spawn UI and verifying that specific GPU types are displayed with availability counts, and that workloads schedule correctly on selected hardware.

**Acceptance Scenarios**:

1. **Given** data scientist opens the spawn UI for notebook or workbench, **When** hardware options are displayed, **Then** system shows specific GPU types with availability (e.g., "NVIDIA H200 141GB - 4 available", "NVIDIA A100 80GB - 2 available")
2. **Given** data scientist selects a specific GPU type, **When** workload is submitted, **Then** workload schedules correctly on the selected GPU type
3. **Given** no GPUs of the selected type are available, **When** workload is submitted, **Then** system provides clear feedback about availability and alternative options

---

### Edge Cases

- What happens when GPU annotations are missing or malformed on nodes (e.g., NVIDIA annotation format variations across OpenShift versions)?
- How does system handle nodes with mixed GPU types (e.g., node with both H200 and A100)?
- What happens when a GPU type becomes unavailable between query time and scheduling time?
- How does system handle legacy hardware profiles created before accelerator type awareness was implemented?
- What happens when external systems (model catalog, training estimator) query hardware profiles but receive no accelerator metadata (e.g., cluster with no GPUs)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Hardware profiles MUST detect and expose specific accelerator types (e.g., "NVIDIA H200 141GB", "NVIDIA A100 80GB") rather than treating all GPUs as generic resources
- **FR-002**: Hardware profiles MUST provide accelerator metadata including GPU type, memory capacity, and architecture through a queryable interface
- **FR-003**: External systems (model catalog, training estimators) MUST be able to programmatically query hardware profile metadata to retrieve accelerator information
- **FR-004**: Hardware profiles MUST automatically detect accelerator types from existing node annotations without requiring manual administrator configuration
- **FR-005**: System MUST expose sufficient GPU performance characteristics to enable external training time estimation engines to calculate training duration estimates
- **FR-006**: Hardware profiles MUST maintain backward compatibility with existing hardware profile features and configurations
- **FR-007**: System MUST handle scenarios where GPU annotations are missing, incomplete, or malformed without affecting overall platform stability
- **FR-008**: Hardware profiles MUST support multiple GPU types within a single cluster environment
- **FR-009**: System MUST provide accelerator type information with sufficient detail to support model-to-hardware matching based on performance requirements and service level objectives

### Key Entities

- **Hardware Profile**: Represents a cluster hardware configuration that now includes specific accelerator type information (GPU model, memory, architecture, compute capabilities) instead of generic GPU counts. Relates to Nodes and Accelerators.
- **Accelerator**: Represents a specific GPU type (e.g., NVIDIA H200 141GB, NVIDIA A100 80GB) with attributes including model name, memory capacity, architecture generation, and compute capabilities. Multiple accelerators may be available in a cluster.
- **Node**: Represents a cluster node that may contain one or more Accelerators. Provides GPU type information through vendor annotations.
- **Model**: Represents a machine learning model with hardware requirements (memory, architecture compatibility) that must be matched to compatible Accelerators for deployment or training.
- **Training Job**: Represents a user-initiated training task that requires Accelerator selection and benefits from training time estimates based on Accelerator performance characteristics.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Model serving team can query hardware profile interface and receive accelerator type information, enabling automatic model-to-hardware recommendations without administrator intervention
- **SC-002**: Training time estimation queries return duration estimates for each available GPU type with accuracy sufficient for user decision-making
- **SC-003**: Data scientists can identify specific GPU types when selecting hardware for workloads, improving hardware selection transparency
- **SC-004**: Workloads schedule successfully on selected GPU types without manual node labeling or administrator mapping
- **SC-005**: Existing hardware profile functionality continues to operate without degradation or breaking changes
- **SC-006**: System handles GPU annotation variations across OpenShift versions without manual configuration updates

## Assumptions

- NVIDIA provides consistent annotations on OpenShift nodes that include GPU type and memory information
- Model catalog and training estimation systems have interfaces capable of querying and consuming hardware profile metadata
- Cluster administrators have already configured node labels or annotations for GPU nodes (no new labeling required beyond existing NVIDIA annotations)
- The platform supports heterogeneous GPU environments where multiple GPU types can coexist in a single cluster
- Training time estimation engines exist or will be developed as separate systems that consume hardware profile data
- Validation metrics and model performance data are available from external model catalog systems

## Out of Scope

The following capabilities are explicitly excluded from this MVP:

- AI-powered GPU recommendation models that predict optimal hardware based on historical usage patterns
- Real-time cost estimation and cost attribution for GPU usage by team or project
- Performance benchmarking databases that store and compare GPU performance across workloads
- Multi-GPU topology awareness for distributed training configurations
- Full node management UI similar to Kubernetes Lens for cluster-wide GPU visualization
- Fractional GPU sharing capabilities such as NVIDIA MIG (Multi-Instance GPU)
- Automatic GPU migration or rebalancing when preferred GPU types become available

## Dependencies

- **External System Integration**: Model catalog and training estimation systems must be updated to consume hardware profile accelerator metadata
- **NVIDIA GPU Operator**: Cluster must have NVIDIA annotations enabled on GPU nodes for automatic detection
- **OpenShift Version**: Requires OpenShift version that supports NVIDIA GPU annotations (version to be confirmed during design partner validation)
- **Design Partner Program**: Requires validation with 2-3 customers per segment (enterprise, startup, research) before GA

## Risk Mitigation

- **NVIDIA Annotation Format Variations**: Comprehensive testing with design partners across different OpenShift versions to identify and handle annotation format variations
- **Backward Compatibility**: Phased rollout with existing hardware profiles continuing to function during migration
- **Missing Annotations**: Graceful degradation when GPU annotations are unavailable, with clear error messaging and fallback to generic GPU handling
- **External System Dependencies**: Clear interface contracts and versioning for hardware profile API to prevent breaking changes for downstream consumers
