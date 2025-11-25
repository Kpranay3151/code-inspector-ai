 Stop Writing "Fixed Bug" - Let AI Handle Your Git Workflow 🚀

How we built an AI Code Inspector that automates commits, writes PR descriptions, and auto-approves quality code.

![AI Code Inspector Banner](https://images.unsplash.com/photo-1555066931-4365d14bab8c?auto=format&fit=crop&w=1000&q=80)

 The "LGTM" Problem

We've all been there. It's 5 PM on a Friday. You've just finished a feature. You run `git commit -m "updates"`, push to main, and ask your colleague for a review. They glance at it for 10 seconds, type "LGTM" (Looks Good To Me), and merge it.

Monday morning, production is down.

The problem isn't just bad code; it's the friction of the process. Writing detailed commit messages is tedious. Summarizing changes for a PR description takes time. And reviewing code thoroughly requires mental energy we often lack.

What if we could offload the boring parts to AI?

 Introducing AI Code Inspector

We built AI Code Inspector, a CLI tool and GitHub integration that acts as your tireless pair programmer. It doesn't just write code; it manages the workflow around it.

 Core Features

1.  🤖 Automated Conventional Commits: No more `fix: stuff`. The AI analyzes your `git diff` and generates semantic commit messages like `feat(auth): implement JWT validation with expiration check`.
2.  📝 Instant PR Descriptions: It reads your commit history and writes a comprehensive PR description, categorizing changes by Backend, Frontend, and Tests.
3.  🛡️ The AI Quality Gate: This is the killer feature. When a PR is opened, the AI runs a suite of checks (Tests, Linting, Secrets, Coverage).
       Pass: The AI auto-approves the PR.
       Fail: The AI rejects the PR and comments on specific lines that need fixing.

 Under the Hood: How It Works

The project is built with Python, Google Gemini 2.0 Flash, and Google Cloud Run.

 1. The CLI Agent
The CLI (`codeinspector`) uses `GitPython` to read your local changes. It sends the diff to the Gemini API with a system prompt designed for Conventional Commits.

```python
 The prompt that powers commit generation
prompt = f"""
You are an expert developer. Generate a Conventional Commit message for the following git diff.
Rules:
- Use the format: <type>(<scope>): <description>
- Types: feat, fix, docs, style, refactor, test, chore
Diff:
{diff}
"""
```

 2. The Quality Analyzer Agent
Instead of just running a linter, we built a `QualityAnalyzer` agent. It takes the raw output of tools like `pytest`, `flake8`, and `coverage`, and makes a semantic decision.

It's not just "Error code 1". The agent understands why it failed and explains it in the PR comment.

```python
 The brain of the operation
if "DECISION: APPROVE" in decision_text:
    gh_client.approve_pr(pr, f"🤖 Auto-Approved: {decision_text}")
else:
    gh_client.reject_pr(pr, f"❌ Issues found: {decision_text}")
```

 3. Serverless Deployment on Google Cloud Run
To make this run automatically on every PR, we deployed a webhook server to Google Cloud Run. This ensures the tool is always on, scalable, and cost-effective.

Here is how we deployed it:

Step 1: Build the Container
We used Google Cloud Build to create the Docker image without needing a local Docker daemon:
```bash
gcloud builds submit --tag gcr.io/$PROJECT_ID/codeinspector
```

Step 2: Deploy to Cloud Run
We deployed the service with a single command, passing in our API keys as environment variables:

```bash
gcloud run deploy codeinspector-service \
  --image gcr.io/$PROJECT_ID/codeinspector \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars GOOGLE_API_KEY=$KEY,GITHUB_TOKEN=$TOKEN,GITHUB_WEBHOOK_SECRET=$SECRET
```

Step 3: Connect GitHub Webhook
Finally, we pointed our GitHub repository to the new Cloud Run URL:
   Payload URL: `https://codeinspector-service-xyz.a.run.app/webhook/github`
   Event: `pull_request`

Now, whenever a PR is opened, GitHub pings Cloud Run, which wakes up, runs the analysis, and goes back to sleep. Cost? Near zero.

 Impact

Since using this tool, our workflow has transformed:
   Commit Quality: 100% adherence to Conventional Commits.
   Review Time: Reduced by ~40%. The AI catches the trivial stuff (linting, secrets, missing tests) before a human ever looks at it.
   Focus: We spend more time coding and less time writing boilerplate text.

 Try It Yourself

The project is open source and easy to deploy. You can run it locally as a CLI or deploy it to your own Google Cloud project.

Check out the repository: [GitHub Link Placeholder]

Installation:
```bash
pip install codeinspector
codeinspector commit
```

Stop writing "wip" commits. Let the AI handle the rest.

---

## Build and Blog Marathon Submission Details

### Application Use Case
AI Code Inspector is a CLI tool and GitHub integration designed to automate the tedious parts of the software development lifecycle. It solves the "LGTM" problem by using AI to generate semantic commit messages, write comprehensive PR descriptions, and perform intelligent quality checks (linting, testing, security scanning) to auto-approve or reject Pull Requests. Its core purpose is to reduce friction, improve code quality, and free up developers to focus on logic rather than boilerplate.

### High-Level Design
The architecture consists of three main components:
1.  **CLI Tool (Python)**: Runs locally to analyze `git diff` and uses Google Gemini 2.0 Flash to generate Conventional Commits.
2.  **GitHub Webhook Server (Flask on Cloud Run)**: Listens for PR events. When triggered, it clones the repository and initiates the analysis.
3.  **AI Agents (Google Gemini)**:
    *   `CommitGenerator`: Creates semantic messages.
    *   `PRDescriber`: Summarizes changes.
    *   `QualityAnalyzer`: Evaluates test results/coverage and makes the final GO/NO-GO decision for auto-approval.

### Sample Dataset / Artifacts Description
The artifacts include the source code for the CLI and Webhook server. The "dataset" processed is the git history and diffs of the user's repository. Specifically, the system processes:
*   `git diff` output for commit generation.
*   `pytest` and `flake8` output logs for quality analysis.
*   No external training dataset is used; it relies on the pre-trained knowledge of Gemini 2.0 Flash applied to the live codebase context.
