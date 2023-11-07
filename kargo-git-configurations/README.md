- https://kargo.akuity.io/how-to-guides/managing-credentials

- Kargo borrows its general credential-management approach from Argo CD, meaning that credentials are stored as Kubernetes Secret resources containing specially-formatted data. These secrets generally take the following form:

- Example create gitlab token secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <name>
  namespace: <namespace>
  labels:
    kargo.akuity.io/secret-type: <secret type>
stringData:
  type: <type>
  url: <repo url>
  username: <username>
  password: <password>
```
```details
name: a unique name for your Kubernetes secret.
namespace: the namespace in Kubernetes where you want to store your secret.
secret-type: for GitLab, you would use repo-creds if it's for repository credentials.
type: the type of version control system, which in this case is GitLab.
url: the URL to your GitLab repository.
username: your GitLab username or 'gitlab' if username is not required for token use.
password: your GitLab personal access token
```