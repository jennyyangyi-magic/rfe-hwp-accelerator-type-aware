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

Hardware Profiles with Accelerator Type Awareness enables the platform to detect and expose specific GPU types (NVIDIA H200, A100, etc.) instead of treating all GPUs as generic resources. This feature addresses a critical user experience blocker where model serving cannot utilize model validation metrics for serving recommendations, and Training cannot provide training time estimation without knowing which specific hardware types are available in hardware profiles.

**Who benefits:**
- **Model Serving Teams**: Can programmatically query hardware profiles to automatically recommend optimal hardware for model deployment based on validation metrics
- **Data Scientists**: Can see training time estimates for each GPU type and make informed hardware selection decisions
- **Platform Administrators**: Eliminate manual hardware-to-model mapping and node labeling overhead

**Current state vs. future state:**
- **Today**: GPUs are treated as generic resources; model serving teams cannot integrate validation metrics; training teams cannot provide GPU-specific time estimates; administrators must manually map models to hardware
- **With this feature**: Hardware profiles expose specific accelerator types; model catalog automatically recommends optimal GPUs; data scientists see training time estimates per GPU type; workloads schedule on specific GPU types without manual intervention

**User narrative:**
When a data scientist initiates a model deployment or training job, they can query available hardware profiles and receive specific GPU type information (e.g., "H200 141GB", "A100 80GB"). For model serving, the model catalog automatically matches models to compatible hardware based on performance insights. For training, the system displays estimated training durations for each GPU type, enabling data scientists to select hardware based on time-to-result and cost considerations.

### The Why

**Why now:**
This feature unblocks critical internal dependencies identified by the Model Serving team (Jenny Yi) and DT team. Without accelerator type awareness, model serving cannot fully integrate model catalog and validation metrics, and training recommendations cannot function. GPU costs represent 60-70% of AI infrastructure spend, making intelligent GPU selection essential for cost management.

**Platform value:**
- Enables 2025-2026 roadmap goals for model serving automation and self-service user experience
- Reduces administrator overhead through automatic hardware discovery
- Improves cost efficiency through informed GPU selection
- Differentiates RHOAI from competitors who already provide accelerator-specific visibility

**Customer acquisition:**
- **Large Enterprises**: Need cost attribution and governance for GPU access across teams
- **AI Startups**: Need simplicity and auto-discovery to minimize platform team overhead
- **Research Institutions**: Need fair-share access to premium GPUs with support for legacy hardware
- **Regulated Industries**: Need audit trails and controlled rollout processes

**Supporting data:**
- Heterogeneous GPU adoption grew 320% YoY
- Competitors (AWS SageMaker, Azure ML) all provide accelerator-specific visibility
- Model Serving and Training teams have identified this as a blocker for critical features

**Success metrics:**
- Model serving team successfully integrates model catalog with hardware profiles
- Training time estimation accuracy within acceptable variance (to be defined with design partners)
- Reduction in administrator support tickets for manual hardware mapping
- Design partner validation with 2-3 customers per segment before GA

### High Level Requirements

*Format: As a [user role/persona], I want [capability/feature], So that [benefit/business value]*

1. As a **Model Serving Engineer**, I want to programmatically query hardware profiles for specific GPU types and metadata, So that I can automatically recommend optimal hardware for model deployment based on validation metrics without manual administrator mapping

2. As a **Data Scientist**, I want to see training time estimates for each available GPU type when configuring training jobs, So that I can make informed hardware selection decisions based on time-to-result and cost considerations

3. As a **Data Scientist**, I want to see specific GPU types with availability information in the spawn UI, So that I can understand which hardware is available and make informed selection decisions

4. As a **Platform Administrator**, I want hardware profiles to automatically detect GPU types from node annotations, So that I can eliminate manual node labeling and hardware configuration overhead

5. As a **Platform Administrator**, I want existing hardware profile functionality to continue working unchanged, So that I can adopt accelerator type awareness without disrupting current workloads

6. As a **Training Time Estimation Engine**, I want to query hardware profiles for GPU performance characteristics, So that I can calculate and present accurate training duration estimates to users

7. As a **Model Catalog System**, I want to query hardware profiles for accelerator metadata including GPU type, memory, and architecture, So that I can match models to compatible hardware based on performance requirements and service level objectives

8. As a **Platform Administrator**, I want the system to handle missing or malformed GPU annotations gracefully, So that platform stability is not affected by annotation variations across OpenShift versions

### Non-Functional Requirements

- **Performance**: Hardware profile queries must return accelerator metadata with minimal latency to support real-time model deployment and training job configuration workflows

- **Security**: Hardware profile metadata must be accessible to authorized systems (model catalog, training estimators) while maintaining appropriate access controls; no sensitive node information should be exposed beyond GPU specifications

- **Disconnected Environments**: Feature must support disconnected/air-gapped environments where external GPU performance databases may not be accessible; core accelerator detection and metadata exposure must function offline

- **User Expectations**:
  - Data scientists expect GPU type information to be accurate and up-to-date
  - Training time estimates should be within acceptable variance (to be defined with design partners)
  - UI should clearly distinguish between different GPU types and show real-time availability
  - Error messages should be clear when GPU annotations are missing or workloads cannot be scheduled

- **Upgrade Considerations**:
  - Feature must support graceful migration from existing hardware profiles without GPU type awareness
  - Legacy hardware profiles created before this feature must continue to function
  - Backward compatibility with existing workloads is mandatory
  - Phased rollout approach to minimize risk

- **Reliability**: System must handle GPU annotation format variations across different OpenShift versions without requiring manual configuration updates

- **Scalability**: Hardware profile queries must scale to support multiple concurrent requests from model catalog and training estimation systems

### Out-of-Scope

The following items are explicitly out-of-scope for this MVP:

- AI-powered GPU recommendation models that predict optimal hardware based on historical usage patterns
- Real-time cost estimation and cost attribution for GPU usage by team or project
- Performance benchmarking databases that store and compare GPU performance across workloads
- Multi-GPU topology awareness for distributed training configurations
- Full node management UI similar to Kubernetes Lens for cluster-wide GPU visualization
- Fractional GPU sharing capabilities such as NVIDIA MIG (Multi-Instance GPU)
- Automatic GPU migration or rebalancing when preferred GPU types become available

### Acceptance Criteria

*Format: Given [context or precondition], When [action taken], Then [expected outcome]*

#### Model Serving Integration (P1)

1. **Given** model catalog has a "llama-3-70b" model requiring 80GB+ GPU memory, **When** deployment is initiated, **Then** hardware profile API returns accelerator metadata showing available H200 and A100 GPUs with memory specifications

2. **Given** hardware profile API returns multiple compatible GPU types, **When** model catalog evaluates validation metrics, **Then** system automatically recommends H200 (optimal for Hopper architecture) without administrator intervention

3. **Given** recommended hardware is selected, **When** deployment proceeds, **Then** workload schedules successfully on the selected GPU type and validation metrics flow through correctly

#### Training Time Estimation (P2)

4. **Given** data scientist is configuring a llama-3-70b training job, **When** hardware profile is queried, **Then** system displays training time estimates for each GPU type (e.g., "H200: 2.1 hours, A100: 5.3 hours")

5. **Given** training time estimates are displayed, **When** data scientist selects H200 for best time-to-result, **Then** training job is submitted with H200 hardware profile

6. **Given** training job completes, **When** actual duration is compared to estimate, **Then** completion time aligns with the provided estimate within acceptable variance

#### GPU Type Visibility (P3)

7. **Given** data scientist opens the spawn UI for notebook or workbench, **When** hardware options are displayed, **Then** system shows specific GPU types with availability (e.g., "NVIDIA H200 141GB - 4 available", "NVIDIA A100 80GB - 2 available")

8. **Given** data scientist selects a specific GPU type, **When** workload is submitted, **Then** workload schedules correctly on the selected GPU type

9. **Given** no GPUs of the selected type are available, **When** workload is submitted, **Then** system provides clear feedback about availability and alternative options

#### Edge Cases & Error Handling

10. **Given** GPU annotations are missing or malformed on nodes, **When** hardware profile is queried, **Then** system handles gracefully with clear error messaging and fallback to generic GPU handling without affecting platform stability

11. **Given** a node has mixed GPU types (e.g., both H200 and A100), **When** hardware profile is queried, **Then** system correctly identifies and exposes all GPU types available on that node

12. **Given** a GPU type becomes unavailable between query time and scheduling time, **When** workload attempts to schedule, **Then** system detects availability change and provides clear feedback to user

13. **Given** legacy hardware profiles created before accelerator type awareness, **When** these profiles are used, **Then** existing functionality continues to work without degradation

14. **Given** external systems query hardware profiles on a cluster with no GPUs, **When** query is executed, **Then** system returns appropriate response indicating no accelerators available without errors

### Risks & Assumptions

**Risks:**
- **NVIDIA Annotation Format Variations**: NVIDIA annotation formats may vary across different OpenShift versions, requiring extensive testing and handling logic
- **External System Dependencies**: Model catalog and training estimation systems must be updated concurrently to consume hardware profile metadata; misaligned releases could delay value delivery
- **Design Partner Availability**: May be difficult to secure 2-3 design partners per customer segment (enterprise, startup, research) for validation
- **Backward Compatibility**: Risk of breaking existing hardware profile functionality if migration is not carefully planned and tested
- **Training Time Estimation Accuracy**: External training time estimation engines may not provide sufficiently accurate estimates, leading to poor user experience
- **Performance Impact**: Adding accelerator metadata queries to hardware profiles could introduce latency in model deployment and training job configuration workflows

**Assumptions:**
- NVIDIA provides consistent annotations on OpenShift nodes that include GPU type and memory information
- Model catalog and training estimation systems have interfaces capable of querying and consuming hardware profile metadata
- Cluster administrators have already configured node labels or annotations for GPU nodes (no new labeling required beyond existing NVIDIA annotations)
- The platform supports heterogeneous GPU environments where multiple GPU types can coexist in a single cluster
- Training time estimation engines exist or will be developed as separate systems that consume hardware profile data
- Validation metrics and model performance data are available from external model catalog systems
- OpenShift versions in target customer environments support NVIDIA GPU annotations

### Supporting Documentation

- Design: [Link to design documents when available]
- Workflows: [Link to user workflow diagrams when available]
- Wireframes: [Link to UI mockups for GPU type visibility when available]
- Discovery Notes: [Link to discovery session notes]
- Technical Documentation: [Link to spec.md in feature branch]
  - Current spec: `specs/001-accelerator-type-awareness/spec.md`
- RFE Documentation:
  - Concise RFE: `rfe-concise.md`
  - Full RFE: `rfe.md`

### Additional Clarifying Information

**Key Open Questions:**
- Should non-admin users be able to see hardware specs directly, OR should it be abstracted from them (e.g., UI only shows estimated training time or "recommended for serving" without exposing specific GPU model details)?
- What is the acceptable variance threshold for training time estimates to be considered successful?
- Which specific OpenShift versions must be supported, and which NVIDIA GPU Operator versions?

**Customer Segments & Design Partners:**
- **Large Enterprises**: Need cost attribution and governance (which teams access which GPUs)
- **AI Startups**: Need simplicity and auto-discovery with small platform teams
- **Research Institutions**: Need fair-share access to premium GPUs and support for legacy hardware
- **Regulated Industries**: Need audit trails and controlled rollout processes

Target: 2-3 design partners per segment for validation before GA

**Strategic Alignment:**
This feature is architecturally feasible because NVIDIA already provides annotations on OpenShift nodes. It unblocks critical roadmap items for model serving automation and training recommendations, aligning with 2025-2026 strategic goals.

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

**Recommendation:** ☒ Architecture review is RECOMMENDED

**Rationale:** This feature introduces a new queryable interface for hardware profile metadata that will be consumed by multiple external systems (model catalog, training estimators). Architecture review would ensure:
- Interface design aligns with overall product vision
- API contracts are well-defined and versioned
- Cross-team integration points are clearly documented
- Future extensibility (non-NVIDIA GPUs, other accelerator types) is considered

### Additional Dependencies

**External System Dependencies:**
- **Model Catalog System**: Must be updated to query hardware profile API and consume accelerator metadata
- **Training Time Estimation Engine**: Must be updated to query hardware profiles for GPU performance characteristics
- **NVIDIA GPU Operator**: Must be deployed and configured on the cluster with GPU node annotations enabled

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
| team-ai-core-platform | [TBD] | Hardware Profile API enhancement to expose accelerator metadata; NVIDIA annotation detection and parsing; backward compatibility testing | Model Serving team for API contract definition; Design partners for annotation format testing | M (2 sprints) | [What's needed: API contract review, design partner coordination] |
| Model Serving / Inference | [TBD] | Model catalog integration with hardware profile API; automatic hardware recommendation logic based on validation metrics | Core Platform team for API availability; UXD for recommendation UI | S (1 sprint) | [What's needed: Core platform API completion] |
| Training / Data Science Tools | [TBD] | Training time estimation engine integration with hardware profile API; training job UI updates to display estimates | Core Platform team for API availability; UXD for estimation display | S (1 sprint) | [What's needed: Core platform API completion] |
| UXD | [TBD] | UI design for GPU type visibility in spawn UI; training time estimate display; hardware recommendation presentation | Training and Model Serving teams for user flow definition | XS (<1 sprint) | [What's needed: User flow clarification] |
| Documentation | [TBD] | User guide for GPU selection; Admin guide for NVIDIA annotation detection; API documentation for model serving integration; Migration guide for existing deployments | All engineering teams for technical details | S (1 sprint) | [What's needed: Engineering implementation completion] |
| QE | [TBD] | Test plan creation; functional testing across different OpenShift versions; GPU annotation format variation testing; integration testing with model catalog and training estimators | All engineering teams for feature completion; Design partners for environment access | M (2 sprints) | [What's needed: Design partner environments, test scenarios] |

**T-Shirt Size Scale:**
- **XS**: < 1 sprint
- **S**: 1 sprint (3 weeks)
- **M**: 2 sprints (6 weeks)
- **L**: 3 sprints (9 weeks) - Highly recommend breaking down
- **XL**: 4 sprints (12 weeks) - Too big! Break it down!

**Total Estimated Effort:** ~6-8 sprints across teams (accounting for dependencies and sequential work)

**Critical Path:** Core Platform API development → Model Serving/Training integration → Design partner validation → QE certification

---

## Documentation and UXD Engagement

### Documentation Team Engagement

- [ ] Added "Documentation" component to the feature
- [ ] Set "Product Documentation Required" field to Yes
- [ ] Added docs team to High Level Plan table
- [ ] Reviewed [Docs Intake Process](https://docs.google.com/document/d/1G_LKipII0DMX3UxpkxVEpgM9Pk5tHcfZdvnkjn9E1mI/edit?tab=t.0)
- [ ] Flagged for Release Notes (if applicable)

**Documentation Deliverables:**
- User Guide: GPU selection for data scientists
- Admin Guide: NVIDIA annotation detection and configuration
- API Documentation: Hardware profile metadata query interface for external systems
- Migration Guide: Upgrading from legacy hardware profiles
- Release Notes: New accelerator type awareness capability

### UXD Team Engagement

- [ ] Added "UXD" component to the feature
- [ ] Added UXD team to High Level Plan table
- [ ] Contacted:
  - ☐ Jenn Giardino (jgiardin@redhat.com)
  - ☐ Beau Morley (bmorley@redhat.com)

**UXD Deliverables:**
- GPU type visibility UI design for spawn interface
- Training time estimate display design
- Hardware recommendation presentation for model serving
- User flow for GPU selection and feedback when unavailable

**Open UX Question:** Should non-admin users see detailed hardware specs (GPU model, memory) or should it be abstracted (e.g., "Best for large models", "Fastest training time")?

---

## Feature Owner Checklist

### Initial Setup
- [ ] Linked this document in the Jira Feature
- [ ] Created public Slack channel (if needed): [Link]
- [ ] Confirmed POCs from each team
- [ ] Updated status to "In Progress"

### Communication
- [ ] Announced feature via email to all stakeholder teams
- [ ] Scheduled review meeting with delivery teams

### Email Distribution Checklist
Ensure the following teams are included:
- [ ] All teams responsible for impacted components
- [ ] aipcc-eng-directs (for awareness of GPU-related feature)
- [ ] inference-ai-eng (Model Serving integration)
- [ ] team-psap@redhat.com
- [ ] team-ai-core-platform@redhat.com
- [ ] ai-bu@redhat.com
- [ ] openshift-ai-docs@redhat.com
- [ ] ai-ux-all@redhat.com
- [ ] rhai-service-teams@redhat.com
- [ ] openshift-ai-devtestops@redhat.com
- [ ] openshift-ai-qe@redhat.com
- [ ] dramseur@redhat.com

### Finalization
- [ ] All impacted components listed in Jira Feature's Components field
- [ ] All teams populated Epic's Parent Link field with Feature link
- [ ] All comments addressed
- [ ] All teams approved (check Approval/Comments column)
- [ ] Updated status to "Approved"
- [ ] Set document permissions to Read-Only
- [ ] Uploaded to [Feature Refinement Documents Folder](https://drive.google.com/drive/folders/1SZjUobSqm4t8etx9nVY0agPhH4Wu0IuS?usp=drive_link)

---

## Delivery Owner Checklist

- [ ] Reviewed and agreed to scope and deliverables
- [ ] Coordinated tracked work creation for all teams
- [ ] Ensured all teams signed off
- [ ] Verified Definition of Ready is met
- [ ] Announced feature development start to organization

---

**Document Status:** Not started

**Last Updated:** 2025-11-20
