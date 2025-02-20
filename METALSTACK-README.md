# kubriX on Metalstack

## how to set it up

### prereqs

#### aws route53 credentials

if you manage your DNS-Names in AWS Route53 with external-dns:
create IAM policy on AWS for AWS Route53: https://kubernetes-sigs.github.io/external-dns/v0.14.1/tutorials/aws/#iam-policy

then create User and static credentials based on https://kubernetes-sigs.github.io/external-dns/v0.14.1/tutorials/aws/#static-credentials
#TODO: i created the user in console, because I was too confused with logging into with aws cli. I need to understand and document this.

create a file `credentials` where "Access key" and "Secret access key" are stored like this:

```
[default]
aws_access_key_id = <access key>
aws_secret_access_key = <secret access key>
```

#### letsencrypt staging root certs

Currently we issue our certificates automatically with letsencrypt. However, due to some limits we don't use the production but the staging letsencrypt API. So you need to put those two root certs in your truststore: https://letsencrypt.org/docs/staging-environment/#root-certificates

#### create OAuth App in your Github Organization for Backstage login: https://backstage.io/docs/auth/github/provider/

- Homepage URL: https://portal-metalstack.platform-engineer.cloud/
- Authorization callback URL: https://portal-metalstack.platform-engineer.cloud/

use GITHUB_CLIENTSECRET and GITHUB_CLIENTID from your Github OAuth App for the following environment variables in step 3

### 1. create metalstack cluster

Create a K8s cluster on https://console.metalstack.cloud/, copy the KUBECONFIG to your local machine to `~/.kube/metalstack-config ` and set the KUBECONFIG to this file.

```
export KUBECONFIG=~/.kube/metalstack-config 
```

### 2. create AWS secret for external-dns 
create namespace and secret and delete local credentials file for security reasons:
```
kubectl create ns external-dns
kubectl create secret generic -n external-dns sx-external-dns --from-file credentials
rm credentials
```

### 3. define some variables so the platform can access github

```
# Github clientsecret and clientid from GitHub OAuth App for Backstage
export KUBRIX_BACKSTAGE_GITHUB_CLIENTSECRET=<value from steps above>
export KUBRIX_BACKSTAGE_GITHUB_CLIENTID=<value from steps above>
# Github token Backstage uses to get the catalog yaml form github
export KUBRIX_BACKSTAGE_GITHUB_TOKEN=<your personal access token>
# Github token ArgoCD uses for the SCM Provider
export KUBRIX_ARGOCD_APPSET_TOKEN=<github-pat-for-argocd-appsets-only-read-permissions-needed>
# Kargo Git Promotion credentials
export KUBRIX_KARGO_GIT_USERNAME=<username-for-kargo-git-promotion>
export KUBRIX_KARGO_GIT_PASSWORD=<username-for-kargo-git-promotion>
# set the current repository to the origin or to your fork
export KUBRIX_REPO=https://github.com/suxess-it/kubriX.git
# if you want to test another branch, specify something else than main
export KUBRIX_REPO_BRANCH=main
# username and password to access kubriX git repository within ArgoCD
export KUBRIX_REPO_USERNAME=<kubrix-repo-username>
export KUBRIX_REPO_PASSWORD=<kubrix-repo-password-or-personal-access-token>
export KUBRIX_TARGET_TYPE=DEMO-METALSTACK
# if a K3d cluster should get created:
export KUBRIX_CREATE_K3D_CLUSTER=false
```

### 4. install platform-stack

clone the upstream repo (or your personal fork) and optionally switch to specific branch

```
git clone ${KUBRIX_REPO}
# change to repo directory (if it is something else then kubriX, please change)
cd kubriX
checkout ${KUBRIX_REPO_BRANCH}
```

and install specific stack

```
./install-platform.sh
```

A "bootstrap argocd" get's installed via helm.
A "boostrap-app" gets installed which references all other apps in the plattform-stack (app-of-apps pattern)
ArgoCD itself is also then managed by an argocd app.

The platform stack will be installed automagically ;)

* backstage
* argocd (managed by argocd)
* argo-rollouts
* kargo
* cert-manager
* crossplane
* kyverno
* prometheus
* grafana
* promtail
* loki
* tempo
* kubecost

### 5. log in to the tools

| Tool    | URL | Username | Password |
| -------- | ------- | ------- | ------- |
| Backstage  | https://portal-metalstack.platform-engineer.cloud/  | via github | via github |
| ArgoCD | https://argocd-metalstack.platform-engineer.cloud/ | admin | `kubectl get secret -n argocd argocd-initial-admin-secret -o=jsonpath='{.data.password}' \| base64 -d` |
| Kargo | https://kargo-metalstack.platform-engineer.cloud/     | admin | - |
| Grafana    | https://grafana-metalstack.platform-engineer.cloud/   | admin | prom-operator |
| Kubevirt-Manager    | https://kubevirt-manager-metalstack.platform-engineer.cloud/   | - | - |

### 6. Onboard teams and applications

In our [App-Onboarding-Documentation](https://github.com/suxess-it/kubriX/blob/main/backstage-resources/docs/onboarding/onboarding-apps.md) and [Team-Onboarding-Documentation](https://github.com/suxess-it/kubriX/blob/main/backstage-resources/docs/onboarding/onboarding-teams.md ) we explain how new teams and apps get onboarded on the platform in a gitops way.

### 7. Promote apps with Kargo

tbd

# Troubleshooting

If argocd ingress is not working, use port-forwarding for accessing argocd console and investigate what is wrong
```
kubectl port-forward svc/sx-argocd-server -n argocd 8080:80
```

- Username: `admin`
- Password: `kubectl get secret -n argocd argocd-initial-admin-secret -o=jsonpath='{.data.password}' | base64 -d`

# AWS helpful things (DRAFT.)

install aws cli:
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install -i ~/bin/aws-cli -b ~/bin
rm -rf aws
rm -rf awscliv2.zip
```
configure aws cli: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html

create IAM policy:
```
aws iam create-policy --policy-name "AllowExternalDNSUpdates" --policy-document file://aws-resources/route53-iam-policy.json
```


# Tests

this ist just a test
something from suXess upstream

and something new

new change in upstream repo

another change in origin repo
