# Feature Refinement Document - Hardware Profiles with Accelerator Type Awareness

**Document Title:** FeatureRefinement - [JIRA#] - Hardware Profiles with Accelerator Type Awareness

---

## Feature Metadata

| Field | Value |
|-------|-------|
| **Feature Jira Link** | [Link to XXXSTRAT-####] |
| **Slack Channel / Thread** | [Link or N/A] |
| **Feature Owner** | [Name] |
| **Delivery Owner** | [Name] |
| **RFE Council Reviewer** | [Name] |
| **Product** | RHOAI (self-managed) |
| **Status** | Not started |

---

## Feature Details

### Feature Overview

**Baseline capability:** Current state of hardware profiles treats all GPUs as generic resources (1 GPU or 2 GPU), while 1 H100 is very different from 1 L40. Hardware Profiles with Accelerator Type Awareness enables the platform to detect and expose specific GPU types (NVIDIA H200, A100, L40, etc.) instead of treating them as identical resources. This foundational capability benefits all models by surfacing accurate hardware information.

**For validated models:** Model serving can leverage validated model performance data (tested on curated hardware configs like 4xH100, 8xH100, 4xL40) to automatically recommend the hardware profile with optimal performance characteristics (e.g., lowest latency, best throughput).

**For non-validated models:** The platform surfaces specific GPU types and metadata (model name, memory capacity, architecture), enabling platform engineers to make informed hardware selection decisions based on their domain knowledge.

This feature addresses a critical blocker where model serving cannot utilize model validation metrics for serving recommendations, and unlocks future training_hub features for providing training time estimation, by knowing the specific hardware type available in hardware profiles.

**Who benefits:**
- **Platform Engineers deploying validated models**: Can receive automatic hardware recommendations based on model validation performance data, eliminating guesswork in hardware selection
- **Platform Engineers deploying non-validated models**: Can see specific GPU types and metadata (instead of generic "GPU") to make informed hardware selection decisions
- **Data Scientists**: Can see training time estimates for each GPU type and make informed hardware selection decisions based on time-to-result and cost considerations

### The Why

**Why now:**
This feature unblocks critical dependencies identified during the integration of Model Validation data in Model Serving.

One of the biggest customer painpoints today is finding optimal hardware to run LLMs. Model validation attempts to solve this problem by testing models on various hardware configs (4xH100, 8xH100, 4xL40) and reporting expected performance metrics. However, this validated performance data is only useful when connected with the customer's actual hardware configs at runtime.

Hardware Profiles with Accelerator Type Awareness is the key connector that enables two value scenarios:

**For validated models:** HWP exposes specific GPU types, allowing model serving to automatically match validation performance data with available hardware profiles and recommend the optimal configuration (e.g., "H100 recommended for lowest latency"). This guides customers to successful and optimized model serving without manual trial-and-error.

**For non-validated models:** Even without validation data, exposing specific GPU types (H200 vs A100 vs L40) instead of generic "GPU" empowers platform engineers to make informed decisions based on their knowledge of model requirements and hardware capabilities. This foundational improvement benefits all model deployments by providing transparency into actual hardware availability.

### High Level Requirements

*Format: As a [user role/persona], I want [capability/feature], So that [benefit/business value]*

1. As a **Platform Engineer deploying validated models**, I want to programmatically query HWP for specific GPU types and metadata, So that I can be recommended optimal hardware for model deployment based on validation metrics without manual administrator mapping.

1a. As a **Platform Engineer deploying non-validated models**, I want to see specific GPU types and metadata (model name, memory capacity, architecture) when querying HWP, So that I can make informed hardware selection decisions based on my domain knowledge of model requirements.

2. As a **Data Scientist**, I want to see training time estimates for my chosen HWP when configuring training jobs, So that I can make informed hardware selection decisions based on time-to-result and cost considerations

3. As a **Platform Administrator**, I want HWP to automatically detect GPU types from node annotations, So that I can eliminate manual node labeling and hardware configuration overhead

4. As a **Platform Administrator**, I want existing HWP functionality to continue working unchanged, So that I can adopt accelerator type awareness without disrupting current workloads

### Non-Functional Requirements

- **Performance**: HWP queries must return accelerator metadata with minimal latency to support real-time model deployment and training job configuration workflows

- **Security**: HWP metadata must be accessible to authorized systems (model catalog, training estimators) while maintaining appropriate access controls; no sensitive node information should be exposed beyond GPU specifications

- **Disconnected Environments**: Feature must support disconnected/air-gapped environments where external GPU performance databases may not be accessible; HWP, core accelerator detection and metadata exposure must function offline

- **User Expectations**:
  - Warning messages should be clear when GPU annotations are missing 

- **Upgrade Considerations**:
  - Feature must support graceful migration from existing hardware profiles without GPU type awareness
  - Legacy hardware profiles created before this feature must continue to function
  - Backward compatibility with existing workloads is mandatory
  - Phased rollout approach to minimize risk

- **Reliability**: System must handle GPU annotation format variations across different OpenShift versions without requiring manual configuration updates

- **Scalability**: HWP queries must scale to support multiple concurrent requests from model catalog and training estimation systems

### Out-of-Scope

The following items are explicitly out-of-scope for this MVP:

- AI-powered GPU recommendation models that predict optimal hardware based on historical usage patterns
- Real-time cost estimation and cost attribution for GPU usage by team or project
- Full node management UI similar to Kubernetes Lens for cluster-wide GPU visualization

### Acceptance Criteria

#### Model Serving Integration - Validated Models (P1)

1. **Given** model catalog has a validated "llama-3-70b" model with performance metrics on 4xH100 and 4xL40, **When** serving UI is initiated, **Then** hardware profile API returns with accelerator metadata showing available H100 and L40 GPUs with memory specifications

2. **Given** hardware profile API returns multiple compatible GPU types (H100, L40) for a validated model, **When** model catalog evaluates validation metrics, **Then** system automatically recommends H100 (based on validation performance data showing lower latency) without administrator intervention

#### Model Serving Integration - Non-Validated Models (P1)

4. **Given** model catalog has a non-validated "custom-llm" model without validation metrics, **When** serving UI is initiated, **Then** hardware profile API returns accelerator metadata showing available GPU types with specifications (e.g., "NVIDIA H200 141GB", "NVIDIA A100 80GB", "NVIDIA L40 48GB")

5. **Given** hardware profile API returns GPU metadata for a non-validated model, **When** platform engineer reviews hardware options, **Then** system displays GPU types and specifications WITHOUT automatic recommendations, allowing the engineer to make informed decisions based on domain knowledge

#### Workbench Integration (P2)

7. **Given** data scientist opens the workbench creation wizard, **When** hardware options are displayed, **Then** system shows specific GPU types and memory metdata (e.g., "NVIDIA H200 141GB", "NVIDIA A100 80GB")

8. **Given** data scientist is in the notebook, **When** training_hub training time estimator is submitted, **Then** the HWP API is queried and training time is estimated based on chosen HWP

#### Edge Cases & Error Handling

10. **Given** GPU annotations are missing or malformed on nodes, **When** hardware profile is queried, **Then** system handles gracefully with clear warning messaging and fallback to generic GPU handling without affecting platform stability

11. **Given** a node has mixed GPU types (e.g., both H200 and A100), **When** hardware profile is queried, **Then** system correctly identifies and exposes all GPU types available on that node

### Risks & Assumptions

**Risks:**
- **NVIDIA Annotation Format Variations**: NVIDIA annotation formats may vary across different OpenShift versions, requiring extensive testing and handling logic
- **External System Dependencies**: Model catalog and workbenches must be updated concurrently to consume hardware profile metadata; misaligned releases could delay value delivery

**Assumptions:**
- NVIDIA provides consistent annotations on OpenShift nodes that include GPU type and memory information
- Model catalog and workbenches have interfaces capable of querying and consuming hardware profile metadata
- Cluster administrators have already configured node labels or annotations for GPU nodes (no new labeling required beyond existing NVIDIA annotations)
- The platform supports heterogeneous GPU environments where multiple GPU types can coexist in a single cluster
- OpenShift versions in target customer environments support NVIDIA GPU annotations

### Supporting Documentation

- Design: [Link to design documents when available]
- Workflows: [Link to user workflow diagrams when available]
- Discovery Notes: [Link to discovery session notes]
- Technical Documentation: [Link to spec.md in feature branch]
  - Current spec: `specs/001-accelerator-type-awareness/spec.md`
- RFE Documentation:
  - Concise RFE: `rfe-concise.md`

### Additional Clarifying Information

**Key Open Questions:**
- Should non-admin users be able to see hardware specs directly, OR should it be abstracted from them (e.g., UI only shows estimated training time or "recommended for serving" without exposing specific GPU model details)?
- Which specific OpenShift versions must be supported, and which NVIDIA GPU Operator versions?

**Strategic Alignment:**
This feature aligns with product strategy and investment into first class experience on inference for generative AI.

---

## New Feature / Component Prerequisites & Dependencies

### ODH/RHOAI Build Process Onboarding

**Will this feature require onboarding of a new container Image or component?** ☐ YES ☒ NO

This feature extends existing hardware profile functionality and does not require new container images or components.

### License Validation

**Will this feature require bringing in new upstream projects or sub-projects into the product?** ☐ YES ☒ NO

This feature uses existing platform capabilities and NVIDIA annotations already provided by the NVIDIA GPU Operator.

### Accelerator/Package Support

**Does this feature require support from the AIPCC team?** ☐ YES ☒ NO

**Notes:** While this feature is focused on GPU accelerators, it consumes existing NVIDIA annotations rather than requiring new AIPCC packages. However, coordination with AIPCC team may be valuable for understanding annotation formats across different accelerator types.

**Consideration:** If future expansion beyond NVIDIA GPUs is planned (AMD, Intel, etc.), AIPCC engagement will be required.

### Architecture Review Check

**Does the feature have the label "requires_architecture_review"?** ☐ YES ☐ NO

**Does the related RFE indicate "Requires architecture review: YES"?** ☐ YES ☐ NO


### Additional Dependencies

**External System Dependencies:**
- NVIDIA GPU Operator and other supported accelerator operators

**Platform Dependencies:**
- **OpenShift Version**: Requires OpenShift version that supports NVIDIA GPU annotations (specific version to be confirmed during design partner validation)
- **Hardware Profile System**: Extends existing hardware profile functionality

**Design Partner Program:**
- Requires validation with 2-3 customers per segment (enterprise, startup, research) before GA
- Design partners must have heterogeneous GPU environments for realistic testing

---

## High Level Plan

### Teams Involved in Feature Delivery

| Team | Start Date | Work to Deliver (EPIC) | Team Dependencies | T-Shirt Size | Approval/Comments |
|------|------------|------------------------|-------------------|--------------|-------------------|
| team-ai-core-platform |  |  |  |  |  |

**T-Shirt Size Scale:**
- **XS**: < 1 sprint
- **S**: 1 sprint (3 weeks)
- **M**: 2 sprints (6 weeks)
- **L**: 3 sprints (9 weeks) - Highly recommend breaking down
- **XL**: 4 sprints (12 weeks) - Too big! Break it down!

---

## Documentation and UXD Engagement

### Documentation Team Engagement

- [ ] Added "Documentation" component to the feature
- [ ] Set "Product Documentation Required" field to Yes
- [ ] Added docs team to High Level Plan table
- [ ] Reviewed [Docs Intake Process](https://docs.google.com/document/d/1G_LKipII0DMX3UxpkxVEpgM9Pk5tHcfZdvnkjn9E1mI/edit?tab=t.0)
- [ ] Flagged for Release Notes (if applicable)

**Documentation Deliverables:**
- User Guide: GPU selection
- Admin Guide: NVIDIA annotation detection and configuration
- API Documentation: Hardware profile metadata query interface for external systems
- Migration Guide: Upgrading from legacy hardware profiles
- Release Notes: New accelerator type awareness capability

**UXD Deliverables:**
- GPU type visibility UI design for spawn interface
- Training time estimate display design
- Hardware recommendation presentation for model serving
- User flow for GPU selection and feedback when unavailable

**Open UX Question:** Should non-admin users see detailed hardware specs (GPU model, memory) or should it be abstracted (e.g., "Best for large models", "Fastest training time")?


