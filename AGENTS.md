# DSH Desktop repository rules

This repository owns the desktop product around an unmodified DeepSeek Harness checkout.

## 本 fork 的长期规则（No-ni/dsh-desktop）

- 只推送到自己的 fork（`origin = No-ni/dsh-desktop`）；上游是 `anywhere-labs/dsh-desktop`，同步用 merge，默认不提 PR、不开 issue；提交文案用中文，提交前先给用户看。
- 与上游保持一致优先：fork 只保留必要的差异，不做无关重构、批量格式化或上游文档重写；与上游一致的格式问题不单独修。
- 默认在 GitHub Actions 构建（`.github/workflows/ci.yml`）；只有用户明确要求才在本机打包，且不借构建绕过「不推送」的要求。
- 上游只以固定版本进入产品：`deepseek-harness/` 子模块与 `upstream.json` 在同一个独立提交里更新，不修改子模块内的文件，也不从子模块链接源码。
- 文档按仓库的双语约定维护：受管 Markdown 由同目录 `*.i18n.yaml` 记录 Git blob hash，改文档必须同步更新记录，否则 `check:bilingual-docs` 失败。
- 除本节外，本 fork 目前与上游 `master` 一致，尚未引入身份、数据目录、更新源或平台范围的渠道差异；引入任何此类差异时，先在本节登记边界，再改实现。
- 本节只记长期规则；提交号、构建状态和产物编号现查，不写入本文件。

## Prerequisites and setup

- Use Node.js `^22.19.0` or `>=24.0.0` and the root Yarn `4.18.0` release through Corepack.
- Initialize the pinned upstream checkout with `git submodule update --init --recursive`.
- Install root dependencies with `corepack yarn install --immutable`.

## Build, run, and verify

- Start the desktop development workflow with `corepack yarn dev`.
- Build the desktop package with `corepack yarn build`.
- Root build, dev/start, and packaging commands refresh `dshmarket` from npm `latest` for Stable, Beta, and Next via `corepack yarn market:prepare`. Commit its exact dependency versions, Desktop compatibility patch, and lockfile together. `corepack yarn market:check` verifies freshness without writes; offline lookups or incompatible patches fail rather than silently retaining an old market. Installed apps do not hot-update executable plugins on launch.
- Before each release, run `corepack yarn aa:prepare-release` to build the latest official Agents Anywhere `main` for both Desktop channels. Commit the resulting artifact, provenance, manifests, and lockfile before packaging. Signed macOS releases and root Windows distribution commands verify freshness and installed versions; `DSH_AA_SOURCE_REF=pinned` is no longer supported.
- Run unit tests with `corepack yarn test`.
- Run type checking with `corepack yarn typecheck`.
- Run the complete headless gate with `corepack yarn check`.
- `corepack yarn dev:next` explicitly launches the experimental Next app; `corepack yarn check:next` validates it without a graphical application. Next uses the official published Web frontend and the recorded upstream Desktop presentation, with product capabilities composed as a separate bundle.
- Develop and validate Desktop feature changes in `dsh-plugin-desktop-beta/` first, then synchronize shared changes into `dsh-plugin-desktop/` while preserving declared variant differences. Before committing or pushing shared Desktop changes, run `corepack yarn check:desktop-variants` and validate both affected packages; neither package automatically inherits the other's source edits.
- Next and Beta may follow different upstream releases: the `next` and `beta` channels in `upstream.json`. While those versions differ, `scripts/verify-desktop-variants.mjs` treats the runtime version as a declared pair, lets each variant keep its own upstream-family pins (`@deepseek-ai/dsh*`, Cordis, Schemastery, Cosmokit) in `package.json` and its own `src/upstream-family.ts`, `src/upstream-family-labels.ts` and `tests/upstream-family.ts`, and keeps lines between `// next-lead:start` and `// next-lead:end` in Next only. Keep every other upstream API difference behind those files or regions; once both follow the same release they must mirror again.
- Run upstream operations through the root scripts, such as `corepack yarn upstream:build`.

- `deepseek-harness/` is a pinned upstream Git submodule. Never edit files inside it from a desktop feature branch.
- `dsh-plugin-desktop/` owns the Cordis Host and Client faces, Electron bootstrap, packaging, and release tests.
- `dsh-desktop-next/` owns the separate experimental Desktop shell, Profiles and recovery, and adapters for the existing AA bridge and Community Market. Next-only changes do not belong in the Stable/Beta variant mirror. Keep its upstream reference and published runtime versions aligned; do not fork the official main frontend.
- `dsh-community-fabric/` owns the community interoperability RFC. Until schemas and a reviewed reference adapter exist, it remains a private documentation scaffold and must not declare loadable DSH or package entry points.
- `dsh-community-market/` owns the community-market shell. Until its runtime is implemented, it remains a private documentation scaffold and must not declare loadable DSH or package entry points.
- The outer repository and all owned packages use the root Yarn release with `nodeLinker: node-modules`.
- The upstream submodule keeps its own pnpm workspace. Run upstream commands through the root `upstream:*` scripts, whose Yarn portable-shell commands enter the submodule before invoking Corepack.
- Compatibility mode must run the upstream default client without overrides. Advanced presentation belongs to desktop-owned client plugins and may replace documented slots or services through profile composition.
- Keep graphical application launch explicit. Builds, typechecks, unit tests, and Loader smokes must remain headless-safe.
- Commit before major changes of direction and keep the submodule pin update separate from desktop behavior changes.
- Keep the repository topology and package-manager split consistent with the [owning Agent Note](.agents/notes/implemented/process/2026-08-15-pinned-upstream-and-isolated-yarn-workspace.md).
