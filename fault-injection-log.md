| Faulted Stage | How the Fault Was Introduced | Observed Pipeline Behaviour | Why This Behaviour Is Correct |
| ------------- | ---------------------------- | --------------------------- | ----------------------------- |

1. Lint Stage Failure

#### Fault introduced: 
Introduce a syntax error in src/index.js (e.g. missing bracket)

#### Observed behaviour:

- Pipeline stops immediately at Lint stage
- Build, Verify, Archive, Publish are skipped
- Job marked as FAILED

#### Why correct:
Early validation prevents invalid code from entering the build or test pipeline, saving compute and maintaining quality standards.

2. Build Stage Failure
#### Fault introduced:
Break scripts/build.js (e.g. reference undefined variable)

#### Observed behaviour:
- Lint passes
- Build fails
- Verify, Archive, Publish skipped

#### Why correct:
A failed build means no valid artifact exists, so downstream stages must not execute.

3. Test Stage Failure (Verify → Test branch)
#### Fault introduced:
test('fail test', () => {
    expect(true).toBe(false);
});

#### Observed behaviour:
- Build succeeds
- Verify stage starts
- Test branch fails
- Security audit may still run (parallel execution behavior)
- Pipeline ultimately marked FAILED

#### Why correct:
Parallel verification allows independent checks, but any failed branch invalidates the release candidate.

4. Security Audit Failure (Verify → Audit branch)
#### Fault introduced:
npm install vulnerable-package 

OR simulate:

npm audit --audit-level=low

#### Observed behaviour:
- Test branch passes
- Audit branch fails
- Verify stage fails overall
- Publish is skipped

#### Why correct:
Security checks are release gates; vulnerabilities block deployment even if functional tests pass.

5. Publish Stage Failure (Nexus)
##### Fault introduced:
Wrong NEXUS_URL or invalid credentials

#### Observed behaviour:
- Lint → Build → Verify → Archive all pass
- Publish fails at Nexus connection/auth step
- Artifact NOT uploaded
- Pipeline marked FAILED
- Previous stages remain valid artifacts in Jenkins workspace/logs

#### Why correct:
Artifact integrity is preserved — no partial or unverified package is published to registry.