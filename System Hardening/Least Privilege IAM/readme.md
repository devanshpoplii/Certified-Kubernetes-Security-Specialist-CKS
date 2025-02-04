It's strongly recommended **not** to use the root AWS account for interacting with an EKS or any Kubernetes cluster. Here’s why:

### Security Best Practices: Least Privilege Principle
- The **root account has full administrative access** to everything in AWS, including billing, IAM, and infrastructure management.
- If compromised, an attacker can **delete everything, access sensitive data, and even lock you out** of your own AWS account.
- Instead, AWS best practice is to create **IAM users with the minimum necessary permissions** to perform specific tasks.
