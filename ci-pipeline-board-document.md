### How Code Becomes a Safe Release in KijaniKiosk

Every time a developer updates KijaniKiosk, their code does not go straight into production. Instead, it passes through an automated quality system designed to ensure that only reliable, secure, and tested changes are released. This system is called a continuous integration pipeline.

When a developer pushes code, the system automatically retrieves it into a controlled environment. The first thing that happens is a quick review step that checks whether the code follows basic rules such as correct syntax and formatting. If issues are found here, the process stops immediately and no further work is done. This prevents broken or low-quality code from progressing further.

If the code passes this initial check, the next step is to assemble the application. This step prepares the program into a structured output that can be tested and shared. The result of this step is stored as a build artifact, which is a packaged version of the application ready for validation.

After the application is built, two important checks run at the same time. The first runs automated tests to confirm that the system behaves as expected. The second checks the application for known security risks in its dependencies. Running these checks in parallel reduces total processing time while still ensuring both quality and safety are verified.

If both checks succeed, the system records the build output as a versioned artifact. This version includes both a standard release number and a unique identifier tied to the specific code change, making it possible to trace exactly what code was used to create it.

Once verified, the artifact is stored in a central repository where approved software versions are kept. This repository ensures that every version is preserved, accessible, and uniquely identifiable. It also prevents accidental overwriting of previous versions, which is important for traceability and rollback scenarios.

### Stage Overview
| Stage          | What It Confirms                          |
| -------------- | ----------------------------------------- |
| Code Check     | Code follows basic quality rules          |
| Build          | Application can be successfully assembled |
| Testing        | System behaves as expected                |
| Security Check | No known vulnerabilities in dependencies  |
| Publishing     | Verified version is stored for future use |

### What Happens When Something Goes Wrong

If an issue is detected at any stage, the process stops immediately. No further steps are executed, and no new version is stored. This ensures that incomplete or unreliable software never reaches the shared repository.

The team is notified of the failure, and logs are provided showing exactly where the process stopped. This makes it easy to identify and fix the problem before trying again. Importantly, earlier successful steps are still visible for reference, but they do not progress into a final release.

This approach ensures that quality issues are caught early rather than being discovered later in production, where they would be more costly and disruptive.

### What This System Does Not Yet Do

While this system ensures that code is checked, tested, and safely stored, it does not yet deploy software automatically to users. It also does not perform large-scale performance testing or simulate heavy user traffic. Additionally, there is no automatic rollback mechanism if issues are discovered after release.

These capabilities are typically introduced in later stages of system maturity once the integration pipeline has proven stable and reliable.

### Closing Perspective

This pipeline acts as a controlled pathway between development and release. It ensures that every change is reviewed, tested, and verified before it becomes part of a shared software version. The goal is not speed alone, but confidence — ensuring that every published version of KijaniKiosk can be trusted by both engineers and business stakeholders.