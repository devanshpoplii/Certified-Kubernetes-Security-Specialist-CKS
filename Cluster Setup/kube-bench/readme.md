# **kube-bench Introduction** 
[kube-bench](https://github.com/aquasecurity/kube-bench) is an open-source tool that checks Kubernetes clusters against the CIS (Center for Internet Security) Kubernetes Benchmark. It evaluates the security configurations of your cluster components and provides recommendations for improvements. 
## **Installation and Running kube-bench** 
### **Manual Installation** 
If you manually install `kube-bench`, you need to specify the correct configuration directory and file when running the tool. This is because `kube-bench` relies on benchmark configuration files to determine the checks to perform. 

To run `kube-bench` with a manually installed setup, use: 
```bash
kube-bench --config-dir <config-directory> --config <config-file>
```
Example: 
```bash
kube-bench --config-dir cfg --config cfg/config.yaml
```
**cfg** directory is installed while you're installing kube-bench utility.
### **Why Do We Need `--config-dir` and `--config`?** 
- When installed manually, `kube-bench` does not have a default location for its configuration files. 
- The `--config-dir` flag specifies the directory where the benchmark configurations are stored. 
- The `--config` flag points to the specific CIS benchmark version YAML file. 
- Without these flags, `kube-bench` will fail to locate the appropriate security checks and may not run correctly.
## **Running kube-bench When Already Configured** 
If `kube-bench` is installed using a package manager (e.g., a Kubernetes Job, Helm Chart, or system package), the necessary configuration files are usually preloaded in the correct locations. In this case, you do not need to manually specify `--config-dir` or `--config`. 

Simply running: 
```bash
kube-bench
```
will automatically detect the appropriate configuration files. 
### **Why Don't We Need to Pass Config?** 
- Pre-configured installations place benchmark files in the expected default paths. 
- `kube-bench` automatically determines the Kubernetes version and selects the correct benchmark configuration. 
- It follows predefined directory structures, eliminating the need for manual file specification.
## **Understanding Targets in kube-bench** 
`kube-bench` supports running security checks for different Kubernetes components. These components are referred to as **targets**. 
Common `kube-bench` targets include: 
- **`master`**: Checks the security configuration of API Server, Controller Manager, Scheduler, and etcd. 
- **`node`**: Checks security settings for Kubelet. 
- **`etcd`**: Runs security checks specifically for the etcd key-value store. 
- **`controlplane`**: Evaluates the API Server, Scheduler, and Controller Manager. 
- **`policies`**: Checks policies such as RBAC and Network Policies. 

### **Running kube-bench with a Specific Target** 
You can specify a target using the `--targets` flag: 
```bash
kube-bench --targets master
```
This will run security checks only for the Kubernetes master components. 
Example for checking worker nodes: 
```bash
kube-bench --targets node
```
## **Conclusion** 
- If `kube-bench` is installed manually, you must specify `--config-dir` and `--config` to locate the correct benchmark files. 
- If it is pre-configured, these flags are not needed as `kube-bench` automatically detects the configuration. 
- Targets allow you to run security checks for specific Kubernetes components. 
- This setup ensures that `kube-bench` runs the appropriate security benchmarks for your Kubernetes environment.
