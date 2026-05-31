# Strategies and best practices for zero trust architecture - Diagrams
## High-level zero trust architecture
```mermaid
flowchart LR

U[User]
D[Device]
IAM[Identity and Access Management]
PEP[Policy Enforcement Point]
PDP[Policy Decision Point]
APP[Enterprise Applications]
DATA[Protected Data]

U --> IAM
D --> IAM
IAM --> PDP
PDP --> PEP
PEP --> APP
APP --> DATA
```
### Explanation
This diagram presents the fundamental structure of a Zero Trust Architecture (ZTA). Unlike traditional perimeter-based security models, ZTA assumes that no user, device, or application should be trusted by default. Every access request must undergo authentication, authorization, and continuous verification before access is granted.
The Identity and Access Management (IAM) system verifies identities and device posture. The Policy Decision Point evaluates contextual information and security policies, while the Policy Enforcement Point controls access to enterprise resources. This diagram establishes the conceptual foundation of the dissertation 
because all subsequent strategies and best practices are based on these core components.

## Use case diagram (zero trust access request)
```mermaid
graph TD
    actor_user("End User")
    actor_device("User Device")
    actor_admin("Security Admin")

    system_boundary("Zero Trust Architecture Framework")

    actor_user -->|initiates| UC1("Request Access to Resource")
    actor_device -->|provides| UC2("Device Posture Data")
    actor_admin -->|defines| UC3("Create Access Policy")
    actor_admin -->|monitors| UC4("View Audit Logs")

    subgraph system_boundary
        UC5("Multi-Factor Authentication (MFA)")
        UC6("Evaluate Trust Score")
        UC7("Policy Decision Point (PDP)")
        UC8("Policy Enforcement Point (PEP)")
        UC9("Continuous Session Monitoring")
        UC10("Logging & Anomaly Detection")
    end

    UC1 --> UC5
    UC5 --> UC6
    UC2 --> UC6
    UC3 --> UC7
    UC6 --> UC7
    UC7 --> UC8
    UC8 --> UC9
    UC9 --> UC10
    UC10 --> UC4
```
### Explanation
It captures the high‑level functional requirements of a zero trust architecture (ZTA) from an actor perspective. It shows how a user, device, and administrator interact with the core zero trust components: policy decision point (PDP), policy enforcement point (PEP), and continuous monitoring.
The diagram includes the main use cases for ZTA: requesting access to a resource, performing multi‑factor authentication (MFA), device posture assessment, generating a trust score, enforcing least‑privilege access, and logging all actions for audit.

This diagram serves as the functional baseline. It illustrates that ZTA is not just about authentication but about continuous verification, context‑aware policy, and micro‑segmentation. It supports the thesis argument that best practices must address all these use cases.

## Class diagram (policy engine and trust evaluator)
```mermaid
classDiagram
    class AccessRequest {
        +String userId
        +String deviceId
        +String resourceId
        +String action (read/write/execute)
        +Timestamp requestTime
        +toPolicyInput() Map
    }

    class TrustScoreCalculator {
        -float identityWeight
        -float deviceWeight
        -float behaviourWeight
        +calculate(AccessRequest, DevicePosture, UserHistory) float
        +updateModel(Anomaly) void
    }

    class DevicePosture {
        +String deviceId
        +boolean diskEncrypted
        +boolean firewallEnabled
        +String osVersion
        +boolean patched
        +float riskScore
    }

    class PolicyRule {
        +String ruleId
        +String subjectPattern
        +String resourcePattern
        +List~String~ allowedActions
        +Condition condition (e.g., trustScore > 0.75)
        +match(AccessRequest, float) boolean
    }

    class PolicyDecisionPoint {
        -List~PolicyRule~ rules
        +evaluate(AccessRequest, float) Decision
    }

    class PolicyEnforcementPoint {
        -PolicyDecisionPoint pdp
        -Logger logger
        +enforce(AccessRequest) boolean
        +logOutcome(AccessRequest, Decision) void
    }

    class SessionContext {
        +String sessionId
        +AccessRequest originalRequest
        +float currentTrustScore
        +long lastVerified
        +reEvaluate() void
    }

    TrustScoreCalculator --> AccessRequest : uses
    TrustScoreCalculator --> DevicePosture : uses
    PolicyDecisionPoint --> PolicyRule : contains
    PolicyDecisionPoint --> TrustScoreCalculator : uses
    PolicyEnforcementPoint --> PolicyDecisionPoint : delegates
    SessionContext --> AccessRequest : references
    SessionContext --> TrustScoreCalculator : periodically re-evaluates
```
### Explanation
The class diagram provides a blueprint for implementing a ZTA prototype. It demonstrates how the thesis translates abstract principles (e.g., “never trust, always verify”) into concrete software components.
It represents the static structure of a software implementation of the core zero trust logic: policy rules, trust scores, session context, and enforcement.

The diagram includes main classes:
- AccessRequest (user, device, resource, time),
- TrustScoreCalculator (combines identity, device health, behavioural anomalies),
- PolicyRule (subject, action, resource, condition),
- PolicyDecisionPoint (evaluates rules against request),
- PolicyEnforcementPoint (grants/denies and logs),
- SessionContext (maintains continuous trust during a session).

## Sequence diagram (continuous verification flow)
```mermaid
sequenceDiagram
    participant U as User
    participant D as Device (Agent)
    participant PEP as Policy Enforcement Point
    participant PDP as Policy Decision Point
    participant TS as Trust Score Calculator
    participant RES as Protected Resource

    U->>PEP: Request access to resource
    activate PEP
    PEP->>PDP: evaluate(request, deviceID, userID)
    activate PDP
    PDP->>D: request MFA + device posture
    D-->>PDP: MFA token, posture data
    PDP->>TS: calculateTrust(user, device, context)
    TS-->>PDP: trustScore = 0.92
    PDP->>PDP: match against policy rules
    alt trustScore >= threshold (0.75)
        PDP-->>PEP: Decision = GRANT (sessionID)
        PEP->>RES: Allow access
        PEP-->>U: Access granted
        Note over PDP,TS: Start continuous re-evaluation loop
        loop every 5 minutes
            PDP->>D: refresh posture & behavioural data
            D-->>PDP: updated data
            PDP->>TS: recalculate trust
            TS-->>PDP: new trustScore = 0.65
            alt trustScore < threshold
                PDP->>PEP: Decision = REVOKE
                PEP->>RES: Terminate connection
                PEP-->>U: Session terminated
            end
        end
    else trustScore < threshold
        PDP-->>PEP: Decision = DENY
        PEP-->>U: Access denied
    end
    deactivate PEP
    deactivate PDP
```
### Explanation
This sequence diagram is the heart of the zero trust demonstration. It visually proves that the architecture is not a one‑time authentication but a continuous, context‑aware process. It shows the dynamic interactions during a typical zero trust access attempt, including initial verification and periodic re‑evaluation during the session.

The diagram follows a complete scenario:
1. User requests access to an internal database.
2. PEP asks PDP for a decision.
3. PDP requests authentication (MFA) and device posture from the user’s device.
4. Trust score is calculated.
5. If above threshold, access is granted and a session context is created.
6. While the session is active, the PDP periodically re‑evaluates trust (e.g., every 5 minutes). If trust drops (e.g., device becomes non‑compliant), the session is terminated.

## Component and deployment diagram (zero trust reference architecture)
```mermaid
graph TD
    subgraph User_Device["User Device"]
        A[Agent: Posture Collector]
        B[App: Browser / Client]
    end

    subgraph Edge["Edge / Service Mesh"]
        C[PEP: Envoy Sidecar]
    end

    subgraph Control_Plane["Central Control Plane (Cloud / On‑Prem)"]
        D[Policy Engine]
        E[Policy Administrator]
        F[Trust Score Engine]
        G[Directory Service - IdP]
    end

    subgraph Logging["Logging & Analytics"]
        H[SIEM / Audit Store]
        I[Anomaly Detector]
    end

    subgraph Resources["Protected Resources"]
        J[Database]
        K[Internal API]
    end

    B -->|request| C
    A -->|periodic posture| G
    C -->|authorization query| E
    E --> D
    D --> F
    D --> G
    C -->|logs| H
    F --> I
    I --> H
    C -->|allowed traffic| J
    C -->|allowed traffic| K
```
### Explanation
This deployment diagram demonstrates how zero trust components can be integrated within a modern enterprise environment. The architecture combines a component diagram (for logical separation) and a deployment diagram (for runtime nodes). It clarifies the separation of control plane (policy) from data plane (enforcement) – a key best practice.

Logical components (based on NIST SP 800‑207):
- Policy Engine (PE) – makes decisions.
- Policy Administrator (PA) – pushes decisions to PEP.
- Policy Enforcement Point (PEP) – intercepts and enforces traffic.
- Trust Score Engine – continuous evaluation.
- Logging & Analytics – audit and anomaly detection.

Deployment nodes:
- End‑user device (runs agent for posture collection)
- Edge PEP (e.g., sidecar or gateway)
- Central Policy Server (hosts PE, PA, Trust Engine)
- SIEM node (logging)

This diagram is particularly relevant to the dissertation because many of the analyzed case studies (e.g., Google BeyondCorp and U.S. Department of Defense initiatives) rely on similar architectural principles.

## References
> [1] S. Rose, O. Borchert, S. Mitchell, and S. Connelly, “Zero trust architecture,” NIST Special Publication 800-207, vol. 1, no. 800–207, Aug. 2020, doi: 10.6028/nist.sp.800-207.

> [2] N. F. Syed, S. W. Shah, A. Shaghaghi, A. Anwar, Z. Baig and R. Doss, "Zero Trust Architecture (ZTA): A Comprehensive Survey," in IEEE Access, vol. 10, pp. 57143-57179, 2022, doi: 10.1109/ACCESS.2022.3174679.

> [3] Rajender Pell Reddy, "Zero Trust Architectures in Modern Enterprises: Principles, Implementation Challenges, and Best Practices," International Journal of Computer Trends and Technology (IJCTT), vol. 73, no. 6, pp. 48-57, 2025. Crossref, https://doi.org/10.14445/22312803/IJCTT-V73I6P107

> [4] Mushtaq, S.; Mohsin, M.;Mushtaq, M.M. "A Systematic Literature Review on the Implementation and Challenges of Zero Trust Architecture Across Domains," Sensors 2025, 25, 6118, https://doi.org/10.3390/s25196118

> [5] E. B. Fernandez and A. Brazhuk, “A critical analysis of Zero Trust Architecture (ZTA),” Computer Standards & Interfaces, vol. 89, p. 103832, Apr. 2024, doi: 10.1016/j.csi.2024.103832.
‌
