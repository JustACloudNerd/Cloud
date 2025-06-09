icon: simple/securityscorecard
# Managing Secrets with Sealed Secrets in Kubernetes

## **Introduction** 
<!-- ![image info](images\sealedsecret.png) -->

Sealed Secrets is a Kubernetes-native tool designed to securely manage secrets by encrypting them for safe storage in public repositories, such as GitHub. Unlike standard Kubernetes secrets, which are only base64-encoded and vulnerable if exposed, Sealed Secrets encrypts secrets using asymmetric cryptography, ensuring they remain secure in version control.

## **Why Sealed Secrets?**

Kubernetes secrets, by default, are base64-encoded, which is not encryption and provides no real security if the encoded data is exposed. Storing such secrets in a public or shared repository poses a significant risk. Sealed Secrets addresses this by:

- Encrypting secrets with a public key, allowing safe storage in public repositories.
- Enabling decryption only within the Kubernetes cluster using a private key managed by the Sealed Secrets Operator.
- Integrating seamlessly with GitOps workflows, ensuring secrets are version-controlled securely.

## **Sealed Secrets Components**

Sealed Secrets comprises three core components:

- **Operator/Controller**: A Kubernetes controller that decrypts Sealed Secrets and creates standard Kubernetes secrets. It manages the private key used for decryption.

- **Kubeseal CLI**: A command-line tool that encrypts Kubernetes secrets using the public key provided by the operator, producing a SealedSecret custom resource.

- **Custom Resource Definition (CRD)**: Defines the SealedSecret resource type in Kubernetes, which encapsulates encrypted secrets.

## **How Sealed Secrets Works**

The workflow for using Sealed Secrets in a Kubernetes cluster is as follows:

1. **Deploy the Sealed Secrets Operator**: Use a Helm chart to deploy the Sealed Secrets Operator, which generates a public/private key pair for encryption and decryption.
2. **Key Generation**: The operator creates a public key (shared via a certificate) and a private key (stored securely within the cluster).
3. **Encrypting Secrets**:
    - Create a standard Kubernetes secret in YAML format.
    - Use the kubeseal CLI to encrypt the secret with the public key, producing a SealedSecret CRD.
4. **Deploying the Sealed Secret**: Apply the SealedSecret YAML to the Kubernetes cluster using 
``` yaml
kubectl apply -f sealsecret.yaml
```
5. **Decryption and Secret Creation**: The operator detects the new SealedSecret, decrypts it using the private key, and creates a standard Kubernetes secret.
6. **Usage**: The resulting secret can be used like any other Kubernetes secret (e.g., as environment variables or mounted volumes).
7. **Secure Storage**: The SealedSecret (encrypted) can be safely committed to a Git repository, as only the cluster's private key can decrypt it

## **Deployment Steps**

1. Deploy the Sealed Secrets Operator
Use the Bitnami Helm chart to deploy the [Sealed Secrets Operator](https://artifacthub.io/packages/helm/bitnami/sealed-secrets?modal=install).

``` yaml
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install sealed-secrets bitnami/sealed-secrets --namespace kube-system
```
or
Install the SealedSecret CRD and server-side controller into the kube-system namespace:
``` yaml
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.29.0/controller.yaml
```
2. Install the Kubeseal CLI
Download and install the kubeseal CLI from the [official releases](https://github.com/bitnami-labs/sealed-secrets/releases).
``` yaml
# Example for Linux (adjust for your OS)
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.29.0/kubeseal-0.29.0-linux-amd64.tar.gz"
tar -xvzf kubeseal-0.29.0-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```
3. Establish Connection Between Kubeseal and Kubernetes
Fetch the public certificate from the Sealed Secrets Operator:
``` yaml
kubeseal --fetch-cert --controller-name <KubeSeal SVC Name> --controller-namespace <Namespace of the Sealed secretes Controller>
```
4. Create a Kubernetes Secret
Generate a standard Kubernetes secret in YAML format (dry-run mode):
``` yaml
kubectl create secret generic database -n default --from-literal=DB_PASSWORD=password123 --dry-run=client -o yaml > secret.yaml
```
5. Encrypt the Secret Using Kubeseal
Encrypt the secret to create a SealedSecret:
``` yaml
kubeseal --controller-name <KubeSeal SVC Name> --controller-namespace <Namespace of the Sealed secretes Controller> --format yaml < secret.yaml > sealed-secret.yaml
```
6. Apply the Sealed Secret
Deploy the SealedSecret to the Kubernetes cluster:
``` yaml
kubectl apply -f sealed-secret.yaml
```
Verify the SealedSecret:

``` yaml
kubectl get sealedsecret -n default
```
The operator will decrypt the SealedSecret and create a standard secret, viewable with:
``` yaml
kubectl get secret database -n default -o yaml
```
## **Benefits of Sealed Secrets**

- **Secure GitOps**: Encrypted secrets can be safely stored in public or private Git repositories.
- **Seamless Integration**: Works with existing Kubernetes workflows, allowing secrets to be used as environment variables or volumes.
- **Operator-Driven Decryption**: Only the cluster's private key can decrypt secrets, ensuring security even if the repository is compromised.

## **Conclusion**
Sealed Secrets provides a robust solution for managing sensitive data in Kubernetes, enabling secure storage of secrets in version control while maintaining compatibility with standard Kubernetes workflows. By leveraging the Sealed Secrets Operator and kubeseal CLI, teams can implement secure GitOps practices with confidence.


















