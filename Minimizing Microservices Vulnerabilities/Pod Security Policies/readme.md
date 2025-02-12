PodSecurityPolicy was deprecated in Kubernetes v1.21, and removed from Kubernetes in v1.25.
Instead of using PodSecurityPolicy, you can enforce similar restrictions on Pods using either or both:

Pod Security Admission
a 3rd party admission plugin, that you deploy and configure yourself

https://kubernetes.io/docs/concepts/security/pod-security-policy/


Pod Security policy was deployed as an Admission controller.
To configure it on API server, add the --enable-admission-plugins-PodSecurityPolicy. You also need to add a security policy which it would refer to.
PSP can be used to validate + mutate as well.



PSA Pod Security Admission:
Kubernetes offers a built-in Pod Security admission controller to enforce the Pod Security Standards.
You can execute the below command and find PodSecurity already enabled in API server:
k exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

You can set a label to the namespace to define the Pod Security Standard level:
pod-security.kubernetes.io/<MODE>: <LEVEL>

<LEVEL> are set of policies/profile defined for security. These are PSS Pod Security Standards
three levels defined by the Pod Security Standards: privileged, baseline, and restricted
Profile	Description
Privileged	Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.
Baseline	Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.
Restricted	Heavily restricted policy, following current Pod hardening best practices.



Mode	Description
enforce	Policy violations will cause the pod to be rejected.
audit	Policy violations will trigger the addition of an audit annotation to the event recorded in the audit log, but are otherwise allowed.
warn	Policy violations will trigger a user-facing warning, but are otherwise allowed.
A namespace can configure any or all modes, or even set a different level for different modes.

