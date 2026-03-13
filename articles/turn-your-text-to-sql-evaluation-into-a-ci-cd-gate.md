
## 2) `articles/turn-your-text-to-sql-evaluation-into-a-ci-cd-gate.md`

```markdown
# Turn Your Text-to-SQL Evaluation into a CI/CD Gate

**Subtitle:** How to wire your rubric and golden dataset into an automated, no-regressions pipeline  
**Author:** Srividya Vaidyanathan  
**Original published version:** https://srividya76.substack.com/p/turn-your-text-to-sql-evaluation  
**Published on Substack:** January 26, 2026

---

## Why this article exists

Once you have a rubric and a golden dataset, the next question is operational:

How do you make sure every new version of your text-to-SQL agent is checked automatically before it reaches users?

That is where evaluation becomes part of the delivery pipeline.

The goal is not fancy infrastructure. The goal is simple: every meaningful change to the agent should trigger a repeatable evaluation run, and regressions should block promotion.

## What automation means here

When people say “wire evaluation into CI/CD,” the core requirement is straightforward.

You should have one command that:

1. Runs the agent on the golden dataset  
2. Scores each test case  
3. Aggregates the results  
4. Compares them to thresholds  
5. Exits with success or failure

Everything else — GitHub Actions, GitLab CI, Jenkins, or any other CI system — is just a wrapper around that command.

The most important principle is this:

Evaluation should be a single, scriptable step that developers can run locally and CI can run automatically.

## Step 1 — Standardize the evaluation runner

Create a single entry point in your repo.

It might be:

- a Python CLI
- a Node script
- a small internal tool

The technology choice matters less than consistency.

At a high level, the runner should:

1. Load the golden dataset  
2. Run the agent on each test case  
3. Capture the generated SQL, execution result, final answer, and optional trace  
4. Compute scores  
5. Aggregate results by test case and by pillar  
6. Return a machine-readable pass/fail outcome

A minimal command might look like:

```bash
python -m eval_runner --suite text2sql
