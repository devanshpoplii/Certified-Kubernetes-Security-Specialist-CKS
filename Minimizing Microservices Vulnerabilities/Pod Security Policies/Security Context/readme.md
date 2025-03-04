# Security Context in Kubernetes
Security Context in Kubernetes allows you to define privilege and access controls for pods and containers. This ensures that workloads run with the least privilege necessary, improving security and reducing risks. 
## Setting Security Context at Pod and Container Level 
Security context can be defined at both **pod** and **container** levels. 

- **Pod-level security context:** Applies to all containers in the pod unless overridden. 
- **Container-level security context:** Overrides pod-level settings for a specific container. 
#### Example of Pod-Level Security Context 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:  # This applies to all containers
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
name: example-container
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
``` 
#### Example of Container-Level Security Context 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-container
spec:
  containers:
name: example-container
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:  # Overrides pod-level settings
        runAsUser: 2000
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
``` 

## Default User When Security Context Is Not Defined 
If `runAsUser` is not set in the security context: 

- By default, most container images run as **root** (UID 0). 
- This can be a security risk because a compromised container could gain full system privileges. 

**Best Practice:** 
Always specify `runAsUser` in the security context to avoid running as root. 

Example:
```yaml
securityContext:
  runAsUser: 1000  # Non-root user
``` 

## Defining Linux Capabilities at the Container Level 
Linux capabilities provide fine-grained privileges beyond root or non-root access. **Linux capabilities can only be defined at Container Level.**
- Capabilities allow specific privileged operations without full root access. 
- Examples: `NET_ADMIN` (modify network settings), `SYS_TIME` (change system time). 
#### Adding and Dropping Capabilities 
Example:
```yaml
securityContext:
  capabilities:
    add: ["NET_ADMIN", "SYS_TIME"]  # Adds specific privileges
    drop: ["ALL"]  # Drops all capabilities except the added ones
``` 
**Best Practice:** 
- Drop all capabilities (`drop: ["ALL"]`) by default and only add required ones. 
- Avoid granting powerful capabilities like `SYS_ADMIN`. 

## Configuring User and Group 
Setting `runAsUser` and `runAsGroup` ensures containers run with a specific user and group ID. 

Example:
```yaml
securityContext:
  runAsUser: 1000  # Run container as UID 1000
  runAsGroup: 3000  # Group ID 3000
  fsGroup: 2000  # Files created by the pod belong to group 2000
``` 

## What Is Privilege Escalation? 
Privilege escalation occurs when a process gains higher privileges than intended. 
**Preventing Privilege Escalation in Kubernetes:** 
```yaml
securityContext:
  allowPrivilegeEscalation: false
``` 
- When `allowPrivilegeEscalation: false`, the container cannot gain more privileges than its parent process. 
- This prevents a non-root process from elevating privileges inside the container. 

## Read-Only Root File System 
A read-only root file system prevents containers from modifying system files. 
```yaml
securityContext:
  readOnlyRootFilesystem: true
``` 
- This limits attack surfaces by preventing malware from modifying binaries. 
- Use `emptyDir` volumes for temporary writable space. 

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-rootfs
spec:
  containers:
    - name: secure-container
      image: busybox
      securityContext:
        readOnlyRootFilesystem: true
``` 

## Conclusion 
Using Kubernetes Security Context effectively improves container security by enforcing: 

:heavy_check_mark: Non-root users 

:heavy_check_mark: Limited capabilities 

:heavy_check_mark: Preventing privilege escalation 

:heavy_check_mark: Read-only file system 

**Best Practices:** 
- Always set `runAsUser` to avoid running as root. 
- Use minimal capabilities (`drop: ["ALL"]` and add only necessary ones). 
- Set `allowPrivilegeEscalation: false` to block privilege escalation. 
- Enable `readOnlyRootFilesystem` to prevent modifications. 

By applying these security measures, you reduce attack surfaces and ensure safer workloads in Kubernetes. 

Reference: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
