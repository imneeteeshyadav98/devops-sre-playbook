# Day 2: IAM Access Architecture

## Journey Context

- **Journey:** DevOps Engineer → Forward Deployed Engineer
- **Day:** 2 of 168
- **Week:** 1 of 24
- **Phase:** AWS Solution Architecture, Security & Customer Discovery
- **SAA-C03 domain:** Design Secure Architectures
- **Primary outcome:** Design secure cross-account AWS access for KubePreflight without permanent access keys

---

## 1. Customer Requirement

KubePreflight ko customer ke AWS account se Amazon EKS metadata read karna hai so that it can assess:

- Cluster version
- EKS add-ons
- Managed node groups
- Upgrade insights
- Rollback-related provider evidence
- Supporting account and region metadata

The access model must satisfy these requirements:

1. KubePreflight ko write access nahi chahiye.
2. Permanent AWS access keys allowed nahi honi chahiye.
3. Customer AWS account aur KubePreflight backend account separate honge.
4. Access auditable hona chahiye.
5. Customer independently access revoke kar sake.
6. One customer ka access doosre customer ko expose nahi hona chahiye.
7. Missing permissions ki wajah se complete multi-account workflow fail nahi hona chahiye.
8. Reduced-IAM aur manifests-only fallback modes available hone chahiye.

---

## 2. Business Objective

Customer ko secure, controlled aur reversible access model dena hai jisse KubePreflight required EKS evidence collect kar sake without:

- Shared credentials
- Excessive permissions
- Long-term secrets
- Customer lock-in
- Unclear auditability
- Uncontrolled blast radius

The solution should reduce onboarding friction while preserving customer ownership of access.

---

## 3. Security Constraints

| Constraint | Required design response |
|---|---|
| No permanent credentials | Use AWS STS temporary credentials |
| Cross-account access | Use a dedicated customer-side IAM role |
| Least privilege | Allow only required `List` and `Describe` APIs |
| Customer-controlled revocation | Customer can delete the role or update trust policy |
| Tenant isolation | Unique role mapping and unique External ID per customer |
| Auditability | Use AWS CloudTrail for role assumption and API activity |
| Limited blast radius | No wildcard write permissions |
| Safe degradation | Continue in reduced-IAM or manifests-only mode |
| Session control | Use constrained role session duration |
| Secret protection | Do not log temporary credentials or External IDs |

---

## 4. Authentication Model

### Authentication question

> KubePreflight backend AWS ke saamne apni identity kaise prove karega?

Recommended model:

```text
KubePreflight Backend
        ↓
Backend Execution Role
        ↓ sts:AssumeRole
Customer AWS Account
        ↓
KubePreflightReadOnlyRole
        ↓
Temporary STS Credentials
```

The KubePreflight backend should run with its own AWS workload identity, such as:

- EC2 instance profile
- ECS task role
- EKS workload identity
- Lambda execution role
- Another approved AWS runtime role

That execution identity should call `sts:AssumeRole` against a dedicated role in the customer account.

### Why this model?

- No customer access key storage
- Temporary credentials automatically expire
- Every customer can maintain an independent trust relationship
- Access can be revoked without coordinating key deletion
- Role assumptions can be audited
- The same platform can support multiple customer accounts safely

---

## 5. Authorization Model

### Authorization question

> Role assume hone ke baad KubePreflight kya kar sakta hai?

Authorization should be controlled through a customer-side permissions policy attached to `KubePreflightReadOnlyRole`.

The policy should include only APIs needed by the product.

Initial examples:

```text
eks:ListClusters
eks:DescribeCluster
eks:ListAddons
eks:DescribeAddon
eks:ListNodegroups
eks:DescribeNodegroup
eks:ListInsights
eks:DescribeInsight
```

Additional APIs should be added only when:

1. A real product workflow requires them.
2. The requirement is documented.
3. The action is validated against actual AWS calls.
4. The policy change is tested.
5. The customer-facing IAM documentation is updated.

---

## 6. Why IAM Role Instead of IAM User

### IAM user approach

An IAM user would normally require:

- Access key ID
- Secret access key
- Secure storage
- Rotation process
- Revocation workflow
- Leak detection
- Tenant-specific secret management

This creates unnecessary operational and security overhead.

### IAM role approach

An IAM role provides:

- Temporary STS credentials
- Short-lived access
- Central trust policy
- Customer-controlled revocation
- Better workload identity
- CloudTrail auditability
- No long-term customer secret storage

### Decision

> Use a dedicated cross-account IAM role with temporary STS credentials. Do not use customer IAM-user access keys.

---

## 7. Architecture

```text
┌───────────────────────────────────────┐
│ KubePreflight Platform AWS Account    │
│                                       │
│ KubePreflightExecutionRole            │
│ - No customer long-term credentials   │
│ - Permission to call sts:AssumeRole   │
└───────────────────┬───────────────────┘
                    │
                    │ sts:AssumeRole
                    │ + ExternalId
                    ▼
┌───────────────────────────────────────┐
│ Customer AWS Account                  │
│                                       │
│ KubePreflightReadOnlyRole             │
│ - Trusts approved platform role       │
│ - Requires customer-specific ID       │
│ - Read-only EKS permissions           │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│ Amazon EKS APIs                       │
│ - DescribeCluster                     │
│ - List/DescribeAddons                 │
│ - List/DescribeNodegroups             │
│ - List/DescribeInsights               │
└───────────────────────────────────────┘
```

---

## 8. Trust Policy

The trust policy answers:

> Who is allowed to assume this role?

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowKubePreflightBackend",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:role/KubePreflightExecutionRole"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "customer-specific-random-value"
        }
      }
    }
  ]
}
```

### Trust policy rules

- Use the exact platform role ARN.
- Do not trust the entire external account unless necessary.
- Require a unique External ID per customer.
- Do not reuse one shared External ID across all customers.
- Do not expose External IDs in public repositories or application logs.
- Keep role-session duration aligned with scan duration.
- Review trust relationships during customer offboarding.

---

## 9. Permissions Policy

The permissions policy answers:

> After the role is assumed, what can the session do?

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadEKSMetadata",
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster",
        "eks:ListClusters",
        "eks:ListAddons",
        "eks:DescribeAddon",
        "eks:ListNodegroups",
        "eks:DescribeNodegroup",
        "eks:ListInsights",
        "eks:DescribeInsight"
      ],
      "Resource": "*"
    }
  ]
}
```

### Policy-hardening approach

1. Start from the actual API-call inventory.
2. Add only required actions.
3. Avoid broad managed policies such as `AdministratorAccess`.
4. Avoid assuming `ReadOnlyAccess` is least privilege.
5. Use resource scoping where the service API supports it.
6. Use CloudTrail and Access Analyzer to inspect usage.
7. Remove unused permissions after validation.
8. Version and document the customer IAM policy.

---

## 10. External ID Purpose

The External ID helps reduce the confused-deputy risk in third-party cross-account access.

### Problem

A platform may be authorised to assume roles in multiple customer accounts. Without customer-specific context, an attacker could attempt to trick the platform into assuming a role that belongs to another customer.

### Recommended model

```text
Customer A → Role A → External ID A
Customer B → Role B → External ID B
Customer C → Role C → External ID C
```

The platform should maintain a strict mapping:

```text
workspace/customer ID
        ↓
customer AWS account
        ↓
customer role ARN
        ↓
unique External ID
```

### Important limitation

External ID is not a password replacement. Security still depends on:

- Correct trusted principal
- Proper tenant mapping
- Secure backend logic
- Least-privilege permissions
- Audit monitoring
- Strong customer isolation

---

## 11. Credential Lifecycle

Recommended sequence:

1. Customer registers role ARN and account details.
2. Platform resolves the correct tenant mapping.
3. Backend execution role calls `sts:AssumeRole`.
4. Customer role returns temporary credentials.
5. Credentials remain in memory only.
6. KubePreflight invokes approved read-only APIs.
7. Temporary credentials expire automatically.
8. Credentials are never written to reports or logs.

### Failure handling

When role assumption fails:

- Record a sanitised error category.
- Do not expose secret values.
- Mark the account as inaccessible.
- Continue scanning other authorised accounts.
- Offer reduced-IAM or manifests-only mode.
- Provide a customer-facing remediation message.

Example:

```text
Account 123456789012 could not be assessed because the configured
cross-account role was unavailable or not authorised.

Result: account skipped
Next action: validate role ARN, trust policy, External ID, and required APIs
```

---

## 12. Audit Strategy

Use AWS CloudTrail to audit:

- `sts:AssumeRole`
- EKS `List` and `Describe` calls
- Trust-policy changes
- Permissions-policy changes
- Role deletion
- Access-denied events

Recommended audit fields:

- Customer/workspace identifier
- AWS account ID
- Region
- Role ARN
- Session name
- Assessment ID
- Start and end time
- API failure category
- Final evidence mode

Do not log:

- Secret access keys
- Session tokens
- Full External IDs
- Kubeconfig certificate or key data
- Bearer tokens
- Sensitive customer payloads

---

## 13. Revocation Strategy

Customer should be able to revoke access without depending on the platform team.

Available methods:

### Option 1: Remove trusted principal

Update the trust policy so that the platform role is no longer trusted.

### Option 2: Delete the customer role

Delete `KubePreflightReadOnlyRole`.

### Option 3: Add explicit deny

Apply an emergency deny through an applicable permissions control.

### Option 4: Disable customer integration

The platform removes the role mapping and refuses future assumptions.

### Session consideration

Already issued temporary credentials may remain valid until expiry. Therefore:

- Avoid unnecessarily long sessions.
- Match duration to expected scan time.
- Do not cache sessions longer than required.

---

## 14. Reduced-IAM and Manifests-Only Fallback

A production-ready assessment platform should not have only one binary access mode.

### Full-access evidence mode

Available evidence:

- EKS control-plane metadata
- Add-on versions
- Managed node-group details
- Provider insights
- Kubernetes API evidence, if configured

### Reduced-IAM evidence mode

Available evidence may be partial:

- Some EKS APIs succeed
- Some provider checks are marked unavailable
- Assessment continues with explicit evidence gaps
- Recommendation confidence is reduced

### Manifests-only mode

When AWS or cluster access is unavailable:

- Analyse supplied Kubernetes manifests
- Detect deprecated APIs
- Analyse PDBs, webhooks, workloads and CRDs where possible
- Clearly state that live provider evidence was not collected
- Never claim full upgrade readiness

---

## 15. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Broad permissions | Larger blast radius | Custom least-privilege policy |
| Shared External ID | Weak tenant isolation | Unique ID per customer |
| Wrong tenant-to-role mapping | Cross-customer access risk | Strict backend mapping and tests |
| Long STS session | Delayed revocation | Short session duration |
| Credential logging | Secret exposure | In-memory handling and redaction |
| Missing permissions | Incomplete evidence | Explicit evidence gaps and fallback mode |
| One account failure breaks all scans | Poor resilience | Per-account error isolation |
| Policy drift | Unexpected failure or excess access | Versioned policy and periodic review |
| Trusts full account | Unnecessary principals may assume role | Trust exact execution-role ARN |
| Unclear customer offboarding | Access remains active | Automated revocation checklist |

---

## 16. Security, Reliability, Performance, Cost and Operations

### Security

- Temporary STS credentials
- Least privilege
- Unique External ID
- Exact trusted principal
- No secrets in logs
- CloudTrail audit
- Customer-controlled revocation

### Reliability

- Per-account error isolation
- Retries for transient AWS failures
- Region-aware execution
- Explicit partial-evidence status
- No false “ready” result when evidence is unavailable

### Performance

- Reuse temporary session only within the permitted assessment lifecycle
- Avoid unnecessary API calls
- Use controlled concurrency
- Handle AWS throttling with backoff

### Cost

Main cost drivers may include:

- CloudTrail data events if enabled beyond normal management events
- CloudWatch logging
- Cross-region API patterns
- Backend runtime
- Repeated scans

The IAM role itself does not create a direct recurring charge.

### Operations

- Version IAM policy templates
- Maintain a required-action manifest
- Test onboarding and offboarding
- Track failed assumptions
- Provide customer-readable diagnostics
- Revalidate when AWS APIs change

---

## 17. Customer Discovery Questions

### Question 1

**Access kis workload ko chahiye—human operator ko ya automated backend ko?**

#### Model customer response

> Automated SaaS backend ko.

#### Recommended reaction

- IAM user access keys reject karo.
- Cross-account role recommend karo.
- Workload identity and STS sessions use karo.

### Question 2

**Kya multiple customers same backend use karenge?**

#### Model customer response

> Haan, har customer ka AWS account separate hai.

#### Recommended reaction

- Unique role mapping
- Unique External ID
- Per-tenant configuration
- Independent revocation
- Tenant isolation tests

### Question 3

**Tool ko kaunsi exact AWS APIs invoke karni hain?**

#### Model customer response

> EKS cluster, add-on, node group aur upgrade-insight metadata read karna hai.

#### Recommended reaction

Create a custom policy using the exact required `List` and `Describe` actions. Do not attach broad managed policies by default.

### Question 4

**Kya customer write operations allow karega?**

#### Model customer response

> Nahi, assessment read-only hona chahiye.

#### Recommended reaction

- No EKS update APIs
- No IAM write APIs
- No EC2 mutation APIs
- Clearly document read-only boundaries

### Question 5

**Access revoke kaise hoga?**

#### Model customer response

> Customer ko apni side se instantly revoke karna hai.

#### Recommended reaction

Use a role-based design. Customer can remove the trusted principal or delete the role. Keep session duration short because issued credentials can remain valid until expiry.

### Question 6

**Agar kuch permissions missing hon to expected behaviour kya hai?**

#### Model customer response

> Baaki assessment continue hona chahiye.

#### Recommended reaction

Implement:

- Account-level isolation
- Check-level evidence status
- Reduced-confidence recommendation
- No vacuous pass
- Clear remediation

---

## 18. Architecture Decision Record

### Decision

Use a dedicated customer-side IAM role assumed by the KubePreflight backend through AWS STS.

### Alternatives considered

#### Customer IAM-user access keys

Rejected because:

- Long-term secrets
- Rotation burden
- Leak risk
- Poor workload identity
- More difficult multi-tenant secret management

#### Broad AWS managed read-only policy

Rejected as the default because:

- More permissions than required
- Harder to explain to customers
- Larger blast radius
- Not true least privilege

#### Manual customer export only

Useful as fallback but insufficient as primary mode because:

- Evidence can become stale
- More manual operational effort
- Harder to validate live provider state

### Consequences

Positive:

- Secure temporary access
- Customer-controlled trust
- Better auditability
- Scalable multi-account pattern

Negative:

- Requires onboarding configuration
- Least-privilege policy requires maintenance
- Cross-account debugging can be complex
- Some AWS APIs may have limited resource-level scoping

---

## 19. SAA-C03 Practice Questions

### Question 1

An application running on Amazon EC2 must read objects from an S3 bucket. What is the most secure approach?

A. Store an access key in the application configuration  
B. Put an access key in EC2 user data  
C. Attach an IAM role to the EC2 instance  
D. Use AWS account root credentials  

**Answer: C**

**Reasoning:** An EC2 instance role provides temporary credentials without storing long-term secrets.

**Why others are wrong:**

- A: Creates long-term credential exposure and rotation overhead.
- B: User data can be retrieved and may expose secrets.
- D: Root credentials must not be used by workloads.

**Principle tested:** Workload identity and temporary credentials.

### Question 2

A third-party SaaS platform needs cross-account access to multiple customer AWS accounts. Which control helps reduce confused-deputy risk?

A. Security group  
B. External ID  
C. Network ACL  
D. Access-key rotation  

**Answer: B**

**Reasoning:** A customer-specific External ID binds the role assumption to the intended third-party/customer relationship.

### Question 3

No applicable IAM policy allows a requested action. What is the result?

A. Allow  
B. Implicit deny  
C. Explicit deny  
D. Region-dependent result  

**Answer: B**

### Question 4

An IAM role policy allows `s3:*`, but an applicable AWS Organizations SCP explicitly denies `s3:DeleteBucket`. What happens?

A. The role policy wins  
B. The latest policy wins  
C. Bucket deletion is allowed  
D. Bucket deletion is denied  

**Answer: D**

### Question 5

A company wants central workforce access across multiple AWS accounts. What is the best starting point?

A. Separate IAM users in every account  
B. Shared root credentials  
C. IAM Identity Center or federation  
D. EC2 instance profiles  

**Answer: C**

### Question 6

What does an IAM role trust policy define?

A. Which API actions the role can perform  
B. Who can assume the role  
C. How a resource is encrypted  
D. Which CloudWatch metrics are emitted  

**Answer: B**

### Question 7

Which policy best follows least privilege?

A. `Action: "*", Resource: "*"`  
B. `AdministratorAccess`  
C. Exact required `Describe` actions for the assessment  
D. General `ReadOnlyAccess` for every workload  

**Answer: C**

### Question 8

What is the most appropriate use of the AWS account root user?

A. Daily deployments  
B. Application authentication  
C. Exceptional root-only tasks with MFA and protected credentials  
D. Shared operational access  

**Answer: C**

---

## 20. Interview Question

### Question

**How would you design secure cross-account access for a platform that assesses customer EKS clusters?**

### Model answer

I would not ask customers to create IAM users or share permanent access keys. Instead, each customer would create a dedicated read-only IAM role in their AWS account.

The role trust policy would allow only the platform’s approved execution role to call `sts:AssumeRole`. I would also require a unique customer-specific External ID to reduce confused-deputy risk and to strengthen tenant isolation.

The attached permissions policy would contain only the EKS and supporting `List` or `Describe` actions used by the assessment workflow. I would avoid broad policies such as `AdministratorAccess` and would not assume that general `ReadOnlyAccess` is least privilege.

The platform would obtain short-lived STS credentials, keep them in memory, and never write them to logs or reports. Role assumptions and AWS API calls would be auditable through CloudTrail.

The customer could revoke future access by removing the trusted principal or deleting the role. I would constrain session duration because already issued temporary credentials may remain valid until expiry.

For reliability, one inaccessible account would not break a multi-account assessment. The platform would record a sanitised access failure, skip or downgrade unavailable checks, continue other assessments, and clearly disclose evidence gaps. Reduced-IAM and manifests-only modes would remain available.

Finally, I would continuously validate the policy against real API calls, remove unused permissions, version the IAM template, and document the exact reason for every required action.

### Trade-offs

- A custom least-privilege policy is more secure but needs maintenance.
- A broad managed policy is simpler but increases blast radius.
- Short sessions improve revocation but may affect long scans.
- Resource scoping may not be supported equally by all EKS APIs.
- Strict access controls improve security but can increase onboarding support.

---

## 21. Common Exam Traps

- Treating an IAM role like an IAM group
- Confusing trust policy with permissions policy
- Assuming `ReadOnlyAccess` always equals least privilege
- Choosing long-term access keys because setup looks easier
- Forgetting that explicit deny overrides allow
- Treating IAM as a regional service
- Using security groups as an IAM substitute
- Assuming External ID alone makes cross-account access secure
- Forgetting that temporary sessions may remain valid until expiry

---

## 22. KubePreflight Connection

This IAM design can become part of KubePreflight’s future customer onboarding and evidence-access contract.

Recommended product artefacts:

```text
docs/
└── aws-integration/
    ├── architecture.md
    ├── customer-role-policy.json
    ├── customer-trust-policy.json
    ├── required-actions.md
    ├── onboarding.md
    ├── offboarding.md
    ├── troubleshooting.md
    └── evidence-modes.md
```

Future product capabilities:

- Validate role assumption before scan
- Explain missing AWS permissions
- Report evidence mode
- Continue on per-account failure
- Generate customer IAM template
- Detect unexpected policy expansion
- Record policy-template version
- Keep AWS evidence separate from Kubernetes evidence
- Prevent full-readiness claims when provider evidence is unavailable

---

## 23. Completion Checklist

- [ ] Authentication versus authorization explained
- [ ] IAM user, group, role and policy compared
- [ ] Temporary credential model documented
- [ ] Trust policy added
- [ ] Permissions policy added
- [ ] External ID purpose explained
- [ ] Audit and revocation strategy documented
- [ ] Reduced-IAM and manifests-only fallback defined
- [ ] Security and reliability risks documented
- [ ] Eight SAA-C03 questions reviewed
- [ ] Interview response rewritten in own words
- [ ] Final Markdown file committed

---

## 24. Score Tracker

```text
SAA-C03 Domain: Design Secure Architectures

Questions attempted: 8
Correct answers: __ / 8
Score: __%

Weak topics:
- __________________________________
- __________________________________
- __________________________________

Carry forward:
- __________________________________
```

---

## 25. Suggested Git Commit

```bash
git add fde-learning/week-01/day-02-iam-access-design.md
git commit -m "docs: design secure cross-account IAM access for KubePreflight"
```

---

## Final Takeaway

> Secure AWS architecture ka goal sirf access dena nahi hai. Goal hai right identity ko, right resource par, right duration ke liye, minimum required permissions ke saath, auditable aur reversible access dena.
