
---

# 🧩 **1. Git Fundamentals**

Git is a **distributed version control system** that tracks changes to files and enables collaboration between developers.

### 🔹 Key Commands & Examples

#### 🧱 `git init`

Initialize a new local Git repository.

```bash
mkdir myapp
cd myapp
git init
```

> Creates a `.git` folder — where Git stores all commits, branches, and configurations.

#### 🌍 `git clone`

Clone a remote repository to your local system.

```bash
git clone https://github.com/rajeshdevopsengineer/devops-notes.git
```

> Used when starting to work on an existing project hosted on GitHub/GitLab/Azure Repos.

#### 💾 `git add` & `git commit`

Stage and save your changes.

```bash
git add app.py
git commit -m "Added login feature"
```

> `git add` moves files to the staging area; `git commit` saves them with a message.

#### ⬆️ `git push`

Push your local commits to the remote repository.

```bash
git push origin main
```

> The `origin` is the remote alias, and `main` is the branch name.

#### ⬇️ `git pull`

Fetch and merge remote changes into your local branch.

```bash
git pull origin main
```

> Ensures your local repo is up to date before new work.

#### 🔀 `git merge`

Combine another branch into your current one.

```bash
git merge feature/login
```

> Used when completing a feature and merging it into the main branch.

#### ♻️ `git rebase`

Replay your commits on top of another branch.

```bash
git checkout feature/login
git rebase main
```

> Keeps a **clean, linear history** (preferred in CI/CD pipelines).

---

# 🧩 **2. Branching Strategies**

Branching allows multiple developers to work independently.
Here are 3 popular strategies:

---

### 🔸 **Git Flow (Traditional Enterprise)**

Used for long-lived projects with release cycles.

**Branches:**

* `main` → production code
* `develop` → latest tested code
* `feature/*` → new features
* `release/*` → stabilization for a release
* `hotfix/*` → urgent production fixes

**Example:**

```bash
git checkout -b feature/login
# work on code
git commit -am "login feature"
git push origin feature/login
```

> Merge feature into `develop`, then `develop` → `main` when releasing.

---

### 🔸 **Trunk-Based Development (Modern DevOps)**

All developers commit frequently to a single branch (`main` or `trunk`).

**Key Ideas:**

* Small, frequent commits
* Use **feature flags** to control unfinished features
* Continuous Integration pipelines run on every push

> Great for teams practicing **CI/CD** and **feature toggling**.

---

### 🔸 **Feature Branching**

Each feature gets its own short-lived branch.

```bash
git checkout -b feature/add-search
```

* Developers push work here.
* When done → Create Pull Request → Merge into `main` after code review.

> Ideal for teams using **GitHub/GitLab workflows**.

---

# 🧩 **3. Git Tags & Versioning**

Tags are used to **mark releases or checkpoints** in your history.

### Types of Tags:

* **Lightweight Tag** – points directly to a commit
* **Annotated Tag** – includes metadata (author, date, message)

**Examples:**

```bash
# Annotated tag
git tag -a v1.0 -m "Initial release"
git push origin v1.0

# List tags
git tag
```

> Common for marking app versions (v1.0.1, v1.1.0).
> CI/CD tools (like Azure DevOps or Jenkins) can automatically deploy tagged releases.

---

# 🧩 **4. Resolving Merge Conflicts & Safe Rebasing**

When two developers modify the same line of code, Git can’t auto-merge — it creates a **conflict**.

### Example Conflict:

```bash
<<<<<<< HEAD
print("Hello from Rajesh")
=======
print("Hello from Team")
>>>>>>> feature/team-message
```

### To Fix:

1. Manually edit to keep the correct version.
2. Mark conflict resolved:

   ```bash
   git add app.py
   git commit
   ```
3. Continue rebase if applicable:

   ```bash
   git rebase --continue
   ```

> **Golden Rule:** Never rebase public branches that others are using. Use it for your local clean-up before merging.

---

# 🧩 **5. Working with Remotes & Submodules**

### 🌐 Remotes

Remote repositories live on platforms like GitHub, GitLab, or Azure DevOps.

```bash
git remote -v
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

### 📦 Submodules

Used when one repository depends on another (like microservices or libraries).

```bash
git submodule add https://github.com/org/shared-lib.git libs/shared-lib
git submodule update --init --recursive
```

> Useful when managing **shared DevOps modules** (Terraform modules, scripts, etc.) across multiple repos.

---

# 🧩 **6. Git Hooks & Automation**

**Git hooks** are scripts that run automatically when certain events happen (commit, push, merge).

Common Hooks:

* `pre-commit`: Lint or format code before committing
* `pre-push`: Run tests or code quality checks
* `post-merge`: Auto install dependencies

Example:

```bash
# .git/hooks/pre-commit
#!/bin/bash
echo "Running code lint before commit..."
eslint .
```

Make executable:

```bash
chmod +x .git/hooks/pre-commit
```

> In DevOps: Hooks automate code quality, prevent bad commits, or trigger build pipelines.

---

# 🧩 **7. GitHub / GitLab Workflows & Collaboration**

### 🔹 Pull Requests (PRs) / Merge Requests (MRs)

* Developers propose changes via PRs/MRs
* Reviewers approve or request changes
* CI pipelines run automatically

**Example Flow:**

1. `git push origin feature/api`
2. Create PR → Reviewer approves
3. Pipeline runs → Deploys to dev/test environment

---

### 🔹 Issues

Used for tracking bugs, enhancements, and tasks.

> Integrates with Jira or Azure Boards for full Agile workflow.

### 🔹 Webhooks

Automate actions (trigger CI/CD, update Slack).

```yaml
# Example GitHub webhook event
{
  "event": "push",
  "repository": "devops-notes",
  "pusher": "rajesh"
}
```

### 🔹 Approvals & Environments

Require specific reviewers before merging.

```yaml
# Azure DevOps YAML snippet
environment:
  name: 'prod'
  approval: required
```

> Promotes **controlled releases** and compliance.

---

# 🧩 **8. Authentication (PATs, SSH, GPG Signing)**

### 🔹 **Personal Access Tokens (PATs)**

* Alternative to passwords for HTTPS.

```bash
git clone https://<username>@github.com/user/repo.git
# Git asks for token instead of password
```

### 🔹 **SSH Keys**

* Secure, passwordless authentication.

```bash
ssh-keygen -t ed25519 -C "rajesh@example.com"
cat ~/.ssh/id_ed25519.pub
# Add to GitHub → Settings → SSH Keys
```

### 🔹 **GPG Signing Commits**

Sign commits to verify identity:

```bash
gpg --list-secret-keys --keyid-format LONG
git config --global user.signingkey ABC12345
git commit -S -m "Signed commit"
```

> This adds “Verified” badges on commits — often required in secure enterprise projects.

---

# 🚀 **9. GitOps Concepts with ArgoCD & Flux**

GitOps = **Git + Ops**
→ Manage your **infrastructure and deployments declaratively** through Git.

---

## 🔸 **Core Principles**

1. **Declarative Infrastructure:** All infra/app manifests (Kubernetes YAML, Terraform, Helm) live in Git.
2. **Versioned Source of Truth:** Git is the single source of configuration truth.
3. **Automated Delivery:** Tools like ArgoCD or Flux automatically sync Git → cluster.
4. **Continuous Reconciliation:** System constantly ensures cluster state matches Git.

---

## 🧭 **ArgoCD Workflow**

1. Developer commits new version of Helm chart:

   ```bash
   git commit -am "Update image to v2.1"
   git push origin main
   ```
2. ArgoCD detects the change and applies it to Kubernetes.
3. If drift occurs (manual change), ArgoCD automatically reverts it.

**CLI Example:**

```bash
argocd app create myapp \
--repo https://github.com/rajeshdevopsengineer/gitops-configs.git \
--path helm/myapp \
--dest-server https://kubernetes.default.svc \
--dest-namespace prod
```

> Use cases:
> ✅ Continuous delivery for microservices
> ✅ Rollbacks via Git history
> ✅ Environment promotion (dev → test → prod)

---

## 🔸 **FluxCD Workflow**

Flux continuously monitors Git repositories and reconciles the state.

**Example Commands:**

```bash
flux install
flux create source git devops-configs \
  --url=https://github.com/rajeshdevopsengineer/gitops-configs.git --branch=main

flux create kustomization myapp \
  --source=GitRepository/devops-configs --path="./k8s/myapp"
```

Flux automatically updates and deploys your Kubernetes manifests when Git changes.

---

## 🔒 **GitOps Benefits**

| Feature          | Description                                   |
| ---------------- | --------------------------------------------- |
| **Auditability** | Every change is tracked in Git                |
| **Consistency**  | All environments deployed from same manifests |
| **Rollback**     | Use Git revert to roll back deployments       |
| **Security**     | RBAC & policy enforcement tied to Git reviews |
| **Automation**   | Continuous sync = no manual `kubectl apply`   |

---

# ✅ **Summary Table**

| Topic                         | Purpose                       | Example Tool/Command        |
| ----------------------------- | ----------------------------- | --------------------------- |
| `git init`, `clone`, `commit` | Start and manage repositories | `git commit -m "Add login"` |
| Branching strategies          | Team collaboration            | Git Flow / Trunk-Based      |
| Tags & versioning             | Release management            | `git tag -a v1.0`           |
| Merge conflicts               | Collaborative editing         | `git merge` / `git rebase`  |
| Remotes/Submodules            | Multi-repo integration        | `git submodule add`         |
| Hooks                         | Automation                    | `pre-commit`, `pre-push`    |
| GitHub/GitLab workflows       | Collaboration                 | PRs, Issues, Approvals      |
| Authentication                | Secure access                 | SSH, PAT, GPG               |
| GitOps                        | Declarative automation        | ArgoCD / FluxCD             |

---

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/484b8e5d-f603-463c-92d8-a1fb1d45f315" />

