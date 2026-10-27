# GitHub Actions on Apple Silicon

The [Apple Silicon CI workflow](../.github/workflows/apple-silicon-ci.yml) runs on the Xcode 27 Apple Silicon runner. It checks the
runner and SDKs, runs `Tests/run.sh`, runs `Tests/audit-local-runtime.py`, and builds both Xcode targets for device ARM64 with
code signing disabled. It also records whether the exact simulator runtime required by this repository is present and, when found,
runs the tracked cache-input validation.

## What the artifact is

The workflow uploads a ZIP containing an **unsigned** `LiveSimBooter.app` and `LiveProcess.appex`, plus logs and a build manifest.
This is a CI/build artifact, not an installable IPA and not a complete simulator environment. iOS will not install it as-is.

The project intentionally does not check Apple runtime binaries or developer signing material into Git. The supported guest input is
the original **iOS Simulator 26.0, build 23A343, arm64** runtime. Cache and resource preparation must be performed against that exact
runtime; a newer runtime is not a substitute. The CI runner may not have this older runtime installed. In that case, runtime validation
is explicitly skipped and reported rather than silently treating the project as device-ready.

## What is required for a phone install

A signed build needs the developer's own registered identifiers, a valid development certificate, and provisioning profiles for both the
host app and the `LiveProcess` extension. Both profiles must grant the required App Group and
`com.apple.developer.kernel.increased-memory-limit` capability. Merely putting the entitlement into the source file is not enough.
Do not put private keys, exported certificates, provisioning profiles, or account/device data in the repository.

The prepared simulator `RuntimeRoot` and `SimulatorData` are separate from the app build and are much larger than normal CI artifacts.
They are based on Apple-supplied binaries and are deliberately not included in the uploaded artifact or published as a GitHub release.
Prepare them under the setup runbook, using original inputs from your own Apple developer/Xcode installation.

Finally, GitHub-hosted runners have no USB connection to your physical iPhone. Pairing, Developer Mode, installation, and runtime testing
must be performed through a separate device-connected workflow or on a machine with access to the phone. A successful CI run proves the
source tests and unsigned build passed; it does not prove code-signature acceptance, runtime activation, or a simulator boot on hardware.
