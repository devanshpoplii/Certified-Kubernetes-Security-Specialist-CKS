# **In-Depth Guide to Pod Security in Kubernetes: PSP, PSA, and PSS** 

Pod security is a critical aspect of Kubernetes, ensuring that workloads run with the least privilege necessary to maintain security and compliance. Kubernetes initially provided **PodSecurityPolicy (PSP)** as a mechanism for enforcing security policies, but it has since been deprecated. The **Pod Security Admission (PSA)** controller, along with **Pod Security Standards (PSS)**, replaced PSP in Kubernetes 1.25. 

This guide will explore: 
- The history and limitations of **PodSecurityPolicy (PSP)**. 
- The introduction and implementation of **Pod Security Admission (PSA)** and **Pod Security Standards (PSS)** as replacements. 
- A detailed breakdown of **PSA modes, exemptions, and real-world usage** with examples. 

# **1. Pod Security Policies (PSP) - Deprecated** 
## **What is PSP?** 
**PodSecurityPolicy (PSP)** was an **admission controller** that enforced security constraints on pods before they were scheduled. It defined conditions that a pod must meet to be accepted by the API server. 

However, PSP was **not enabled by default**. Cluster administrators had to explicitly enable it, create PSP objects, and then grant users permissions via **RBAC (Role-Based Access Control)** to use those policies. 
## **How PSP Worked** 
1. **Enable the PSP Admission Controller** (Deprecated in Kubernetes 1.21, removed in 1.25). 
2. **Define a PodSecurityPolicy object** specifying security constraints. 
3. **Grant RBAC permissions** to users or service accounts so they could use a PSP. 
4. **Admission Controller intercepts pod requests**, checks against the available PSPs, and either **mutates (adjusts settings)** or **validates (rejects the pod if non-compliant)**. 

### **Example of a PSP** 
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
ALL
  volumes:
'configMap'
'emptyDir'
'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'MustRunAs'
    ranges:
min: 1000
        max: 2000
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
min: 1000
        max: 2000
```
This policy enforces **non-root execution, disables privilege escalation, limits volume types, and restricts user/group IDs**. 

### **Example Pod Request Evaluation with PSP** 
Let’s say a user submits the following pod: 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
name: nginx
    image: nginx
    securityContext:
      privileged: true
```
The PSP admission controller will: 
- **Mutate**: If allowed, it adjusts the pod settings (e.g., removes `privileged: true`). 
- **Validate**: If settings are too insecure and cannot be modified to conform, it rejects the pod creation. 

## **Limitations of PSP** 
- **Complexity**: PSP enforcement required **both RBAC and policy creation**, making it difficult to manage. 
- **RBAC Dependency**: Users needed explicit RBAC permissions to apply PSPs, leading to **misconfiguration risks**. 
- **Ambiguous Behavior**: If multiple PSPs were available, the **most permissive** one could be applied, **weakening security**. 
- **Not Namespace-Scoped**: PSPs applied at the **cluster level**, making per-namespace security enforcement cumbersome. 
- **Deprecation & Removal**: PSP was **deprecated in Kubernetes 1.21** and **removed in 1.25**, requiring migration to **Pod Security Admission (PSA)**. 

# **2. Pod Security Admission (PSA) & Pod Security Standards (PSS)** 
## **What is PSA?** 
**Pod Security Admission (PSA)** is the replacement for PSP. It is an **admission controller enabled by default** in Kubernetes 1.25+. 
### **Key Differences from PSP:** 
:white_check_mark: **Namespace-based enforcement** (instead of cluster-wide policies). 

:white_check_mark: **Does not require RBAC** (policies apply automatically). 

:white_check_mark: **Three predefined security standards**: **Privileged, Baseline, Restricted**. 

:white_check_mark: **Three enforcement modes**: **Enforce, Audit, Warn**. 

## **Pod Security Standards (PSS)** 
PSS defines **three levels of security** that PSA enforces: 
| Standard | Description | Target Use Cases |
|----------|------------|------------------|
| **Privileged** | No restrictions, allows privileged containers. | Legacy apps, testing environments. |
| **Baseline** | Minimal restrictions to prevent known privilege escalations. | General workloads, default setting. |
| **Restricted** | Strongest security, enforces least privilege principles. | Multi-tenant clusters, security-critical apps. |

### **Comparison of PSS Levels** 
| Feature | Privileged | Baseline | Restricted |
|---------|------------|------------|------------|
| Root User Allowed | :white_check_mark: Yes | :white_check_mark: Yes | :x: No |
| Privileged Mode | :white_check_mark: Allowed | :x: Denied | :x: Denied |
| Host Networking/Ports | :white_check_mark: Allowed | :x: Denied | :x: Denied |
| Volume Types | :white_check_mark: Any | :x: No HostPath | :x: Limited |
| Seccomp | :x: Not Enforced | :x: Not Enforced | :white_check_mark: Must be set |
| AppArmor | :x: Not Enforced | :x: Not Enforced | :white_check_mark: Must be set |

# **3. How PSA Works** 
PSA operates at the **namespace level**, applying **Pod Security Standards** in one of three **enforcement modes**: 
1. **Enforce**: Blocks non-compliant pods from being created. 
2. **Audit**: Logs violations but does not block pods. 
3. **Warn**: Displays warnings but allows pod creation. 

## **Configuring PSA in a Namespace** 
To apply **Restricted** policies in **Enforce** mode to a namespace: 
```sh
kubectl label namespace secure-ns pod-security.kubernetes.io/enforce=restricted
```
This ensures that only **Restricted-compliant** pods can be created in `secure-ns`. 

To enable **Audit** mode instead: 
```sh
kubectl label namespace secure-ns pod-security.kubernetes.io/audit=restricted
```
This logs violations instead of blocking pods. 
### Example Pod Rejected by PSA
If a pod violates `restricted` security policies: 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
name: nginx
    image: nginx
    securityContext:
      privileged: true
```
When created in a `restricted` namespace with **Enforce mode**, Kubernetes rejects it: 
```plaintext
Error: pod is restricted: privileged mode is not allowed.
```

# **4. PSA Exemptions** 
Some components may require exemptions from PSA rules. 

### **Configuring PSA Exemptions via Admission Configuration File** 
In Kubernetes, **Pod Security Admission (PSA)** can be configured using an **admission configuration file**, which is then passed to the **API server** via the `--admission-control-config-file` flag. 

This allows administrators to define **PSA exemptions** for specific namespaces, users, or runtime classes. 

## **Example Admission Configuration File** 
Create an admission configuration file (e.g., `admission-config.yaml`) to define **PSA exemptions**. 
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
name: PodSecurity
    configuration:
      apiVersion: pod-security.admission.config.k8s.io/v1alpha1
     kind: PodSecurityConfiguration
      defaults:
        enforce: "restricted"
        enforce-version: "latest"
        audit: "baseline"
        audit-version: "latest"
        warn: "baseline"
        warn-version: "latest"
      exemptions:
        usernames:
"system:serviceaccount:kube-system:default"
"system:masters"
        runtimeClasses:
"kata-containers"
        namespaces:
"kube-system"
"custom-exempt-ns"
```
### *Explanation of Configuration:
**Defaults:** 
- `enforce: "restricted"` → Enforces **restricted** security policies by default. 
- `audit: "baseline"` → Logs pods violating **baseline** policies. 
- `warn: "baseline"` → Warns users when creating pods that don’t meet **baseline** standards. 
**Exemptions:** 
- **Exempting users** (e.g., `system:masters`, `system:serviceaccount:kube-system:default`). 
- **Exempting runtime classes** (e.g., `kata-containers`). 
- **Exempting namespaces** (e.g., `kube-system`, `custom-exempt-ns`). 

## **Passing the Configuration File to the API Server** 
Once the `admission-config.yaml` file is created, pass it to the **API server** using the `--admission-control-config-file` flag. 


# **5. Conclusion** 
**PSP was complex, required RBAC, and was removed in Kubernetes 1.25**. 
**PSA is a default admission controller, using Pod Security Standards (PSS) to enforce security at the namespace level**. 
**PSA modes (Enforce, Audit, Warn) provide flexible security enforcement**. 
**PSA allows exemptions for system-critical namespaces and users**. 
By understanding and implementing **PSA correctly**, Kubernetes clusters can be **secured effectively** while maintaining usability.**
