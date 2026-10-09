# Prompt: adapt a Python library to NVIDIA DGX Spark

Copy the prompt below into an agent session and replace the repository placeholder.
The baseline reflects our requested setup; it is not a claim that every library supports it.

---

Act as a Python packaging and GPU compatibility engineer. Inspect and modify
`<TARGET_REPOSITORY_URL_OR_PATH>` so that it can be installed and used on NVIDIA
DGX Spark with uv. Make the necessary repository changes; do not stop at advice.

## Target baseline

Unless I explicitly specify otherwise, target:

- NVIDIA DGX Spark with its GB10 GPU.
- Linux ARM64 / aarch64, not Linux x86_64.
- uv as the environment and dependency manager.
- CPython 3.13.x.
- PyTorch 2.10.0 with CUDA 13.0 wheels (`cu130`).
- Matching TorchAudio / TorchVision versions only if the library needs them.
- Standard Hugging Face cache reuse when the project uses Hub models.

These are requested defaults, not permission to silently upgrade or downgrade.
If this combination has a verified blocker, explain the exact dependency conflict,
show the evidence, and propose the smallest viable alternative before changing
the baseline. Do not assume that a CUDA wheel exists for ARM64 just because it
exists for x86_64.

You may not have a DGX Spark or access to its GPU. Complete useful configuration,
source inspection, and lightweight checks anyway. Clearly separate what you
verified from what I must test on my Spark.

## Inspect before editing

1. Read applicable AGENTS.md instructions and inspect the current branch,
   working tree, packaging files, lockfiles, CI, installation docs, and examples.
   Preserve unrelated work and follow the repository's contribution rules.
2. Find every dependency declaration and installation path, including
   pyproject.toml, requirements files, setup files, environment files, Dockerfiles,
   and scripts that install or override Torch. Determine which is authoritative.
3. Identify direct and transitive constraints on Python, Torch, CUDA, NumPy,
   model frameworks, and native extensions. Check optional accelerators such as
   FlashAttention, xFormers, Triton, and quantization backends where actually used.
4. Consult current primary sources: official package indexes, wheel metadata,
   project documentation, and upstream source. Verify the exact Python ABI,
   architecture, version, and CUDA variant for required binary dependencies.
   Record links and the date of verification for important compatibility decisions.
5. Distinguish the wheel's CUDA runtime, the installed NVIDIA driver, and the
   local CUDA toolkit used for source builds. Do not assume selecting cu130
   installs or upgrades the driver or supplies a complete build toolkit.

## Modify dependencies with uv

- Make the smallest coherent changes that support the baseline.
- Declare Torch directly when the library imports it. Align TorchAudio and
  TorchVision with Torch using the official compatibility matrix; their version
  numbers do not necessarily match each other.
- Configure an explicit named uv index for
  `https://download.pytorch.org/whl/cu130` and route only the relevant PyTorch
  packages to it through `[tool.uv.sources]`. Keep unrelated packages on their
  normal index. Do not use an indiscriminate extra-index or unsafe index strategy.
- Ensure the selected versions resolve to CUDA 13.0 builds on Linux ARM64.
  Use exact version/build pins when needed for the dedicated Spark setup.
- For a Spark-only fork, Python `>=3.13,<3.14` and uv environment restrictions
  for Linux aarch64 may be appropriate. For a general-purpose library, preserve
  existing supported platforms and Python versions; use an explicit Spark
  configuration, markers, or extras instead of imposing global restrictions.
- Set the default development interpreter to 3.13 in a way consistent with the
  repo, such as .python-version. Explain that requires-python declares supported
  versions, whereas an interpreter pin selects the development default.
- If uv platform restrictions are used, consider both `environments` and
  `required-environments` so resolution checks ARM64 wheel availability.
- Remember that `[tool.uv.sources]` is uv-specific and is not an index directive
  embedded in published package metadata. Document the supported install command.
- Update a tracked lockfile when dependency resolution is available. Do not
  fabricate or hand-edit lockfile hashes. If locking cannot be completed, state
  that clearly and give the exact command I should run.
- Avoid blanket dependency upgrades, unrelated refactors, and unnecessary packages.
  Do not remove a required feature simply to make resolution succeed.

## Native extensions and examples

- Do not make optional acceleration libraries mandatory unless their compatibility
  with this exact target is verified. Use a supported built-in backend, such as
  PyTorch SDPA, when the model actually supports it. Do not claim SDPA and
  FlashAttention have identical behavior or performance.
- If a native build is required, document the supported build tools, headers,
  toolkit, and build-isolation requirements based on upstream guidance. Avoid
  guessing GPU architecture flags or using unrelated third-party wheels.
- Check example scripts and CLI defaults for hardcoded paths, forced unsupported
  attention backends, stale dependency instructions, and invalid Hub model IDs.
- For Hugging Face models, use valid `namespace/model` IDs without trailing
  slashes. Reuse the standard cache and respect HF_HOME / HF_HUB_CACHE.
  Do not manually construct cache snapshot paths or redownload weights into the repo.
- Explain cache reuse versus fully offline execution. Reference audio, tokenizers,
  configs, and other assets may have separate download requirements. If exposing
  local_files_only or cache_dir, verify those settings reach all nested loaders.
- Do not download large models, install a multi-gigabyte GPU stack, or run expensive
  inference just to validate metadata unless that is part of my request.

## Validate and hand off

Run checks proportionate to the changes:

1. Parse TOML and check dependency markers and edited Python syntax.
2. Resolve dependencies / validate the lockfile if feasible, explicitly checking
   the target architecture rather than accepting an x86_64-only resolution.
3. Use focused existing tests or a stub loader to check example loading arguments
   where useful. A mocked loader does not prove cache access or GPU compatibility.
4. Review the final diff and verify no unrelated files or secrets were included.

Provide copy-and-paste commands for the Spark to:

- Install/select Python 3.13 and synchronize with uv.
- Print Python version, machine architecture, Torch version, Torch's CUDA version,
  CUDA availability, and the GPU name/capability when CUDA is available.
- Download required models into the HF cache if applicable.
- Run the smallest meaningful real example and identify its output.
- Capture the full traceback and relevant environment details if it fails.

Finish with the files changed, compatibility decisions and source links, checks
actually run, checks still pending, and any exact blockers. Never describe
configuration-only validation as successful DGX Spark inference. Follow my
requested branch and publishing instructions, and report the commit or PR when
changes are published.
