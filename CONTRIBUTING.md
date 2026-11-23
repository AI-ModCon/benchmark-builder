# Contributing to benchmark-builder

Thank you for your interest in contributing to benchmark-builder, an agent skill for
building MLCommons-ontology-conformant scientific ML benchmarks. This document explains how
to report problems, propose changes and submit pull requests.

## Code of Conduct

This project and everyone participating in it is governed by our
[Code of Conduct](./CODE_OF_CONDUCT.md). By participating, you are expected to uphold this
code.

## How to Contribute

### Reporting bugs

Check the [issue list](https://github.com/AI-ModCon/benchmark-builder/issues) first. When you
open a bug report (the "Bug report" template), include as much detail as possible:

- a clear and descriptive title;
- what you asked the agent to do, and the exact steps that reproduce the problem;
- what happened, and what you expected instead;
- the files involved (a `SKILL.md` or reference section, a script and its command line, a
  template);
- your environment: agent client and version (e.g. Claude Code), OS, Python version.

**Security vulnerabilities don't go in public issues.** See [SECURITY.md](./SECURITY.md).

### Suggesting enhancements

Enhancement suggestions are tracked as GitHub issues (the "Feature request" template).
Describe the current behavior, the proposed behavior, and why it would help benchmark
authors. A concrete benchmark where the skill fell short is the most useful evidence.

### Pull requests

1. Fork the repository and create a branch: `git checkout -b feature/my-change`.
2. Make your change. Keep it focused, and match the surrounding style (see "Style
   guidelines").
3. Run the checks below and fix what they report.
4. Add an entry to `CHANGELOG.md` under "Unreleased".
5. Open a pull request using the template, and fill in how you tested the change.

Every pull request needs review by a human maintainer before it merges.

## Development setup

benchmark-builder is a skill, not a Python package: the content is `SKILL.md`,
`references/`, `scripts/` and `assets/`. To work on it:

```bash
git clone https://github.com/AI-ModCon/benchmark-builder.git
cd benchmark-builder
python3 -m venv .venv && . .venv/bin/activate
python -m pip install numpy pandas pyyaml matplotlib
```

To try your working copy in Claude Code, link it into a skills directory, e.g.
`ln -s "$PWD" ~/.claude/skills/benchmark-builder`.

## Checks to run before a pull request

```bash
# every script and template parses
python -m py_compile scripts/*.py assets/repo/*.py assets/codabench/*.py
for f in $(git ls-files '*.sh' '*.sbatch' '*.pbs'); do bash -n "$f"; done

# both worked Codabench bundles still validate (tiers 1-3)
python scripts/validate_codabench_bundle.py assets/codabench/example_bundle \
    --submission assets/codabench/example_bundle/starting_kit/sample_submission.zip --task-index 0
python assets/codabench/results_example/build_bundle.py
python scripts/validate_codabench_bundle.py assets/codabench/results_example/build/bundle \
    --submission assets/codabench/results_example/build/bundle/starting_kit/sample_submission.zip
python scripts/validate_codabench_bundle.py assets/codabench/results_example/build/bundle \
    --submission assets/codabench/results_example/build/extra/baseline_mean_submission.zip
rm -rf assets/codabench/results_example/build

# the skill still passes the Genesis Mission skill validator (run from the checkout,
# which must be named benchmark-builder: the skill's name has to match its directory)
git clone --filter=blob:none https://github.com/AI-ModCon/genesis-skills /tmp/genesis-skills
python /tmp/genesis-skills/tools/validate_skills.py --profile all --no-color --format text "$PWD"
```

The validator should report 0 errors. Expect a warning about `SKILL.md`'s length; don't
add new ones. If you change a script's behavior, run it on the inputs the docs describe
and say in the pull request what you ran.

## Style guidelines

### Skill content

- `SKILL.md` holds the workflow the agent follows. Put detail in `references/` and point to
  it; keep each fact in one place (see the "Documentation cleanup" entry in
  `CHANGELOG.md`).
- Templates in `assets/` stay generic. A fix found while building one benchmark goes into
  that benchmark's own code and docs, not into these templates (see `SKILL.md`, "Build the
  artifacts").
- Write facts you verified, with the evidence (a command, a file, a date). Mark anything
  unverified as such.
- Paths to files in a generated benchmark repository are written `<repo>/...`.

### Python

- Follow [PEP 8](https://peps.python.org/pep-0008/); use type hints where they help.
- Scripts take arguments on the command line, document their usage in the module
  docstring, and exit non-zero with a message naming the problem when something's wrong.
- Keep dependencies minimal: the standard library, numpy and pandas where needed.

### Commit messages

- Use the present tense and the imperative mood ("Add feature", not "Added feature").
- Limit the first line to 72 characters or less.
- Reference issues and pull requests after the first line.

## Guidelines for AI/LLM-Assisted Contributions

- **Remain accountable for all your outputs and decisions.**
   Individuals remain fully responsible and accountable for the accuracy, quality, appropriateness, and consequences of their work. Use of AI does not transfer this responsibility to the AI model, agent, or other tool.
- **Understand your work.**
   Regardless of how code or PR was produced, this project requires that authors illustrate a thorough understanding of any proposed changes. You must review such code line-by-line; it is your responsibility to ensure that it is correct, and that it does not breach copyright. Always critically engage with AI outputs, do not trust them implicitly. AI-assisted code, analysis, and artifacts must be tested and validated at a level appropriate to their impact. Authors are responsible for ensuring that generated code is correct, secure, maintainable, non-obfuscated, appropriately scoped, documented, and reproducible where relevant.
- **Disclose AI-generated or AI-assisted work.**
   If AI/LLM tools were primarily used to generate code or artifacts, this should be clearly indicated in the PR.
- **Use of AI to review PRs.**
   All PRs must be reviewed by a human reviewer. An LLM review may be used in addition to a human reviewer since this can help spot issues that a human may have missed, but this should not be the sole reviewer. The human reviewer should be fully accountable and responsible for the review feedback or comments (see 1).
- **Proprietary or personal information.**
   For this project, proprietary or personal information should never be sent to code generators or AI tools.
- **Be transparent, assume goodwill, and share what you learn.**
   Contributors should be open about relevant AI use, disclose details of AI use as appropriate to the project, engage constructively with colleagues, and share experiences and lessons learned with the project.

This policy is the ModCon Base policy, from
[AI-ModCon/BaseTemplate](https://github.com/AI-ModCon/BaseTemplate).

## Questions?

Open an issue with the label `question`.

Thank you for contributing!
