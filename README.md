# HW4: Final Research Project -- Agentic Skill Composition Analyzer

**CS 292C -- Spring 2026**
**Due: June 6, 2026 (Friday of Finals Week), 11:59 PM**

## Overview

Modern AI agents compose *skills* -- small, natural-language tool descriptions --
to accomplish complex tasks. Each skill may look safe in isolation, but when
composed together they can introduce **cross-skill vulnerabilities**: sensitive
data leaking between skills, credentials shared without least-privilege, or
untrusted input flowing into privileged operations.

Your task is to build a **static analyzer** that takes a directory of skill
files (Markdown descriptions of agent tools) and reports potential security
findings as a **standardized JSON array**.

There is no written report. Your code *is* the deliverable.

## Standardized JSON Interface

Your analyzer **must** be invocable as:

```bash
python3 run.py <path-to-skill-directory>
```

and print to **stdout** a JSON array of finding objects. Each finding has this
schema:

```json
{
  "id": "FINDING-001",
  "severity": "high",
  "title": "Short human-readable title",
  "description": "Detailed explanation of the vulnerability.",
  "location": {
    "file": "skill_poster.md",
    "line": 4
  },
  "cross_skill": true,
  "related_skills": ["skill_reader.md", "skill_poster.md"],
  "cwe": "CWE-200"
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the finding (e.g., `FINDING-001`) |
| `severity` | string | One of: `critical`, `high`, `medium`, `low`, `info` |
| `title` | string | Short title (< 120 chars) |
| `description` | string | Detailed explanation |
| `location.file` | string | Skill filename where the issue manifests |
| `cross_skill` | bool | `true` if the finding spans multiple skills |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `location.line` | int | Line number (if applicable) |
| `related_skills` | list[str] | Other skill files involved |
| `cwe` | string | CWE identifier (e.g., `CWE-200`) |

## Suggested Research Directions

You may pursue **any** analysis approach. Here are eight ideas to get you
started -- pick one, combine several, or invent your own:

1. **Taint Tracking Across Skills** -- Model context variables as taint
   sources/sinks and track data flow across skill boundaries.

2. **Capability Lattice Analysis** -- Build a lattice of capabilities (file
   read, network write, credential access) and flag when a composed workflow
   violates least-privilege.

3. **LLM-Assisted Semantic Analysis** -- Use an LLM to interpret natural
   language skill descriptions and extract security-relevant operations, then
   apply traditional analysis on the extracted model.

4. **Pattern-Based Detection** -- Define a library of cross-skill vulnerability
   patterns (e.g., "read secret then post to network") and match against
   skill compositions.

5. **Information-Flow Type System** -- Assign security types (public, secret,
   PII) to data mentioned in skills and check that compositions respect the
   type constraints.

6. **Abstract Interpretation of Skill Chains** -- Define abstract domains for
   data sensitivity and compute a fixpoint across a skill pipeline.

7. **Graph-Based Composition Analysis** -- Build a data-flow graph across skills
   and use reachability or graph patterns to find vulnerabilities.

8. **Differential Analysis** -- Compare a skill's behavior in isolation versus
   in composition and flag emergent capabilities that only arise from
   composition.

## Benchmarks

The `benchmarks/` directory contains eight harnesses. Each is a directory of
2–3 skill files that form a multi-agent pipeline. Every harness has at least
one cross-skill vulnerability — a security issue that only emerges from the
composition, not from any individual skill in isolation.

| Harness | Skills | Domain |
|---------|--------|--------|
| `harness_read_and_post/` | 2 skills | File reading + notification |
| `harness_code_review/` | 3 skills | Code fetch + lint + reporting |
| `harness_data_pipeline/` | 2 skills | Data ingestion + cloud upload |
| `harness_deploy_pipeline/` | 2 skills | Build + deployment |
| `harness_research_assistant/` | 3 skills | Paper search + summarization + saving |
| `harness_incident_response/` | 2 skills | System diagnostics + external analysis |
| `harness_onboarding/` | 2 skills | Account provisioning + notification |
| `harness_monitoring/` | 2 skills | Metric collection + alerting |

**You are not told what the vulnerabilities are.** Discovering them is part of
the project. Your analyzer should find at least one cross-skill issue per
harness.

The autograder will also test on **additional hidden harnesses** not included
here.

## Project Structure

```
hw4-template/
  README.md          # This file (update with your Problem/Approach/Evaluation)
  run.py             # YOUR ENTRY POINT -- accepts a harness directory, outputs JSON
  validate.py        # Validates your JSON output format
  check.sh           # Submission validator script
  benchmarks/        # 8 multi-skill harnesses (2–3 skills each)
    harness_read_and_post/
    harness_code_review/
    harness_data_pipeline/
    harness_deploy_pipeline/
    harness_research_assistant/
    harness_incident_response/
    harness_onboarding/
    harness_monitoring/
```

## Grading Rubric

| Component | Points | Criteria |
|-----------|--------|----------|
| **Benchmark Detection** | 40 | At least one cross-skill finding per harness (3 harnesses). Quality and precision of findings. |
| **Analysis Approach** | 30 | Novelty and soundness of your technique. Does it generalize beyond the provided benchmarks? |
| **Code Quality** | 15 | Clean, readable, well-structured code. |
| **JSON Compliance** | 15 | Output passes `validate.py` on all harnesses. Correct schema, unique IDs, valid severities. |

**Total: 100 points**

## Submission

1. Ensure `python3 run.py <path>` works and outputs valid JSON to stdout.
2. Run `bash check.sh` and fix any errors.
3. Push your code to your GitHub Classroom repository before the deadline.

## Academic Integrity

You may use any libraries, tools, or LLM APIs in your implementation. You must
write your own analysis logic -- do not copy another student's analyzer. If you
use an LLM as part of your analysis pipeline (Direction 3), document which model
and how it is used in code comments.
