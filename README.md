# DevOps-demo
This repository servers as a demo for setting up a DevOps Pipeline. Essentially, we're using an empty React.js project and trunk based development model.

* https://reactjs.org/docs/create-a-new-react-app.html

* https://cloud.google.com/solutions/devops/devops-tech-trunk-based-development

# Tooling
Listing of tools we have found useful. Lots of alternatives for each and every one of them exists out there!


## Developer
Tooling used on Developers local environment

### Pre-commit Hooks
https://github.com/typicode/husky

In our project, pre-commit hooks are enabled and managed by Husky, which almost lived up to it's slogan "Git hooks made easy"

#### Talisman
https://github.com/thoughtworks/talisman

Talisman is a tool that installs a hook to your repository to ensure that potential secrets or sensitive information do not leave the developer's workstation. It validates the outgoing changeset for things that look suspicious - such as potential SSH keys, authorization tokens, private keys etc.

We have Talisman installed on our local workstations as a pre-commit git hook template.

#### Lint-staged
https://github.com/okonet/lint-staged

Lint-staged is a tool that enables us to run various lint processes as a pre-commit hook, and enabled us to use the following linters as a pre-commit hook

#### Eslint
https://eslint.org/

In addition to having eslint installed in our IDE's, we also set up the linter to run as a pre-commit hook. Eslint analyzes the code and looks for problems, and fixes them automatically if possible and adds them to your commit, which was enabled by lint-staged.

#### Prettier
https://prettier.io/

Prettier is an opinionated code formatter which ensures that the newly created code is always clean and tidy format-wise, and is ran as a pre-commit hook through lint-staged.

### Linters
Also remember to set up a linter into your IDE, depends on the language you are developing on. Git hooks for example could also be used to enforce project coding standards.


### IDE Plugins
There are lots of plugins for every IDE out there, that boosts greatly boosts productivity and minimizes errors.  For example some I've installed on my VSCode: Docker, ESLint, GitHub Actions, GitLens, Remote - WSL 


## Infrastructure as Code

Developed in Docker environment -> changes pushed to git -> CI/CD tests pass -> docker image is built -> deployment into docker image repository (dockerhub) -> server will pull the latest image from hub -> rolling update to replace existing version with new

### Docker
https://www.docker.com/

Docker for containerization.
We use docker to create similar environment for all coders to develop the program. Also the final application is deployed as docker image.

### Kubernetes
https://kubernetes.io/

Kubernetes on server side handles docker container orhcestration. This allows us to perform rolling updates with 0 downtime, get information of currently running pods and nodes and scale the application as needed. The configuration files for this setup can be found in `./kubeconfigs/` and they are mainly running on a cloud server with k3s deployed. Should be noted that we found kubernetes overkill and unnecessary for a small project like this, but we considered it an important technology to learn and as such we included it in our pipeline.

The entire stack is essentially built out of:
* k3s (kubernetes distro, with loadbalancer and traefik disabled since k3s ships with traefik v1 and we want v2. For using load balancer, consider MetalLB instead for baremetal configuration such as ours in the cloud) `curl -sfL https://get.k3s.io | sh -s - --no-deploy=traefik --no-deploy=servicelb`)
* helm (kubernetes package manager) to install traefik
* `devops.yaml` defining the `deployment`, `service` and `ingress`
* `devops-tls.yaml` defining the TLS secured `IngressRoute`
* `traefik-values.yaml` to override some of traefik default configuration values

Quick tutorial to setting it all up for MVP configuration:
Setting up the master & worker node on same machine `curl -sfL https://get.k3s.io | sh -s - --no-deploy=traefik --no-deploy=servicelb`  
Download files from `kubeconfigs`, apply with  
`kubectl apply -f devops.yaml`  
`kubectl apply -f devops-tls.yaml`  
`helm repo add traefik https://helm.traefik.io/traefik`  
`helm repo update`  
`helm install traefik traefik/traefik`  
`helm --kubeconfig /etc/rancher/k3s/k3s.yaml --namespace traefik upgrade --install traefik traefik/traefik -f traefik-values.yml` (namespace arg may be unnecessary)

(Full tutorial on downloading helm, setting up traefik ingress & TLS secured routing: https://community.traefik.io/t/traefik-v2-helm-a-tour-of-the-traefik-2-helm-chart/6126 )


## Security

### Dependabot (GitHub native)
https://dependabot.com/

Dependabot is the GitHub native solution for Software Composition Analysis (SCA). It checks for updates for possible outdated and vulnerable packages, and opens a pull-request automatically for every out-of-date dependency, which we then only need to review and merge to the project.

### GitHub Code scanning (w/ CodeQL, GitHub native)
https://github.blog/2020-09-30-code-scanning-is-now-available/

We are also using GitHub code scanning, which is new GitHub-native approach to easily find security vulnerabilities. 

### Njsscan
https://github.com/ajinabraham/njsscan#github-action

Njsscan is an open source Statis Application Security Testing (SAST) tool that aims to find insecure code patterns in node.js applications, and is currently integrated to our CI/CD pipeline workflow.

It utilizes libsast pattern matcher and the syntax-aware semantic code pattern search tool semgrep, and is set to run on every push to our development branch and scan the entire project.

### OWASP ZAP Full Scan
https://github.com/marketplace/actions/owasp-zap-full-scan

OWASP ZAP (Zed Attack Proxy) Full Scan is an open source Dynamic Application Security Testing (DAST) tool that is designed to perform automated security scans against applications and their environment.

The ZAP Full Scan runs the ZAP spider and optionally also an ajax spider scan against the specified target site. The tool then reports it's findigs and automatically creates GitHub issues to the corresponding project in case it finds something alarming.


## Monitoring

On server `traefik` dashboard allows to get data of current kube configuration. Further tools such as prometheus could be considered for this.

# Resources
## DevOps
* https://roadmap.sh/devops
## DevSecOps
* https://notsosecure.com/achieving-devsecops-with-open-source-tools/
* https://medium.com/fraktal/practical-framework-for-devsecops-dd7fd9e63866
