# Security Policy

## Reporting a vulnerability

**Please don't report security vulnerabilities through public GitHub issues.**

Report them privately through GitHub's private vulnerability reporting: on the
repository's **Security** tab, choose **Report a vulnerability**
(<https://github.com/AI-ModCon/benchmark-builder/security/advisories/new>). Only the
maintainers can see the report.

Please include:
- a description of the vulnerability;
- steps to reproduce it;
- its potential impact;
- a suggested fix, if you have one.

We will acknowledge receipt within 48 hours and keep you updated on progress toward a fix.

## Scope

benchmark-builder is an agent skill: instructions, reference documents, templates, and
scripts that an agent runs on the user's machine. Reports we especially want:
- a script that executes untrusted input in a way its documentation doesn't warn about;
- instructions that would lead an agent to expose credentials or publish something without
  the user's confirmation.

**Trust boundary of the Codabench validator.** `scripts/validate_codabench_bundle.py`
*runs* a bundle's ingestion and scoring programs and a submission's code: on the host's
Python in tier 3, and in Docker in tier 4. Run it only on bundles and submissions you
trust, as you would any code. It never pushes images.

## Supported versions

There are no tagged releases yet. Security fixes go to the `main` branch.

## Security updates

We will publish fixes as soon as possible after a vulnerability is confirmed, and notify
affected users through GitHub security advisories.
