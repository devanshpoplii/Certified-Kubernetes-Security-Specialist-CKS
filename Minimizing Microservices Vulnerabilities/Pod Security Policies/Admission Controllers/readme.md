# **Admission Controllers in Kubernetes** 
Admission controllers in Kubernetes are mechanisms that intercept requests to the API server before objects are persisted in etcd. They help enforce policies, validate requests, and even modify them if necessary. 

There are two types of admission controllers: 
- **Mutating Admission Controllers** – Modify requests before they are stored. 
- **Validating Admission Controllers** – Validate requests and reject them if they don’t meet certain conditions. 

**Note:** Some controllers perform both the operations.

## **Order of Execution: Mutating Comes First** 
When a request reaches the API server, Kubernetes first processes it through **mutating admission controllers**, modifies it if necessary, and then sends it to **validating admission controllers**. This order ensures that any required modifications (such as adding default values) are applied before validation occurs. 
### **Example: NamespaceExists and NamespaceAutoProvision** 
Previously, Kubernetes had two controllers: 
- **NamespaceExists (Validating)**: Ensured that a namespace must exist before a resource is created in it. 
- **NamespaceAutoProvision (Mutating)**: Automatically created a namespace if it didn’t exist. 

If both were enabled, `NamespaceAutoProvision` would run first, creating the missing namespace, allowing `NamespaceExists` to validate the request successfully. However, since both are now deprecated, the `NamespaceLifecycle` admission controller replaces their functionality. 

### **Example** 
```sh
kubectl run test-pod --image=nginx --namespace=my-nonexistent-ns
```
If `NamespaceAutoProvision` were enabled, it would create `my-nonexistent-ns` automatically before `NamespaceExists` could reject the request. 

## **Viewing Enabled Admission Controllers** 
To see which admission controllers are enabled, run: 
```sh
kube-apiserver -h | grep enable-admission-plugins
```
Alternatively, check the running `kube-apiserver` pod in a kubeadm cluster: 
```sh
kubectl -n kube-system get pod -l component=kube-apiserver -o yaml | grep enable-admission-plugins
```

## **Common Admission Controllers** 
### **DefaultStorageClass** (Mutating) 
Automatically assigns a default storage class to PVCs (Persistent Volume Claims) if none is specified. 
**Example Command:** 
```sh
kubectl apply -f pvc.yaml  # PVC gets assigned a default storage class automatically
```
---
## **Adding or Removing Admission Controllers** 
### **Enabling an Admission Controller** 
Modify the API server configuration: 
```yaml
spec:
  containers:
name: kube-apiserver
    command:
kube-apiserver
--enable-admission-plugins=DefaultStorageClass,NamespaceLifecycle
```
Restart the `kube-apiserver` pod to apply changes. 
### **Disabling an Admission Controller** 
```yaml
spec:
  containers:
name: kube-apiserver
    command:
kube-apiserver
--disable-admission-plugins=NamespaceLifecycle
```
This prevents `NamespaceLifecycle` from running.

## **External Admission Controllers: Webhooks** 
In addition to built-in admission controllers, Kubernetes allows external controllers using **admission webhooks**. These webhooks enable custom validation and mutation logic by forwarding API requests to an external webhook server.

### **Mutating and Validating Admission Webhooks** 
- **Mutating Admission Webhooks** modify requests before they are saved. 
- **Validating Admission Webhooks** validate requests before they are processed. 

Since Kubernetes executes mutating webhooks first, external webhooks should follow this pattern if both are needed.

## **How Webhooks Work** 
1. The API server receives a request. 
2. If a webhook is configured for the request type, the API server sends an **AdmissionReview** request to the webhook server. 
3. The webhook server processes the request and returns an **AdmissionReview** response. 
4. If it’s a **mutating webhook**, changes are applied before validation. 
5. If it’s a **validating webhook**, the request is either allowed or denied. 

## **Deploying an Admission Webhook Server** 
A webhook server is typically deployed as a Kubernetes **Deployment** and exposed via a **ClusterIP** service.

## **Webhook Configuration Object** 
A webhook needs to be registered with the Kubernetes API server using a **MutatingWebhookConfiguration** or **ValidatingWebhookConfiguration** object.
### **Example: MutatingWebhookConfiguration** 
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: example-mutating-webhook
webhooks:
name: example.webhook.com
    clientConfig:
      service:
        name: webhook-service
        namespace: default
        path: "/mutate"
      caBundle: "<BASE64_ENCODED_CA>"
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
    admissionReviewVersions: ["v1"]
    sideEffects: None
```
### **Breaking Down the Configuration:** 
- **`clientConfig`**: Specifies how the API server communicates with the webhook server. 
- **`rules`**: Defines which resources and operations trigger the webhook. 
- **`admissionReviewVersions`**: Ensures compatibility with the API server version. 
---
## **Conclusion** 
Admission controllers are a critical part of Kubernetes security and governance. With built-in controllers like `NamespaceLifecycle` and `DefaultStorageClass`, Kubernetes ensures best practices. However, external webhooks provide flexibility for custom policies. 

By understanding how admission controllers work—including the execution order of mutating and validating webhooks—you can effectively manage your Kubernetes clusters while enforcing security and compliance.*
