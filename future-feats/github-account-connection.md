# GitHub Account Connection

## Goal

Let a player connect GitHub identities to Gitmatica securely from inside the
client. This feature owns authorization, identity, credential storage,
selection, disconnect, and recovery.

It does **not** link a project to a repository or implement clone, fetch, pull,
push, merge, or other repository data transfer. Those operations are a separate
remote-support feature that consumes the authenticated-account boundary defined
here.

GitHub.com is the initial provider. Keep the model host-aware so another provider
does not require changing account identity or secret-storage rules.

## Primary Authorization Flow

Use a GitHub App user-authorization flow in the system browser:

1. Start an authorization-code request with PKCE, a high-entropy `state`, and a
   short lifetime.
2. Bind a one-shot callback listener to an ephemeral port on `127.0.0.1`.
3. Open only an allowlisted GitHub HTTPS authorization URL in the system browser.
4. Validate the callback path and `state`, exchange the code, and close the
   listener immediately.
5. Query GitHub for the authenticated user and key the account by GitHub host
   plus stable numeric user ID, not by mutable login name.
6. Store the resulting short-lived credentials only after identity validation.

Do not embed a GitHub login page or collect a GitHub password. Keep browser and
network work off Minecraft's render thread and return UI changes to the client
thread.

Use GitHub's device authorization flow only when the browser callback cannot be
completed. Show the exact GitHub domain, user code, expiration, and explicit
**Open Browser**, **Copy Code**, and **Cancel** actions. Poll at GitHub's supplied
interval, honor rate-limit slowdown, and discard credentials if cancellation
races with successful authorization.

Authorizing an identity is not the same as granting access to a repository. A
later project-linking flow may guide the user through GitHub App installation or
repository access selection.

## Credential Storage

Store access and refresh credentials in the operating system's credential
vault:

- Windows Credential Manager on Windows.
- Login Keychain on macOS.
- Freedesktop Secret Service's default collection on Linux.

Store only non-secret presentation state in Gitmatica's client configuration:
host, numeric account ID, login, display metadata, and which identity is
selected. Never store account state in a Gitmatica project repository.

If the vault is missing, locked, denied, or unavailable, explain that persistent
sign-in is unavailable and offer session-only authentication. Session-only
credentials remain in memory and require authorization again after restart.
Never silently downgrade to plaintext or home-grown file encryption.

The vault protects secrets at rest, not against another malicious mod in the
same Minecraft JVM or software running as the same OS user. Retrieve credentials
only for the operation that needs them, minimize caching, and do not promise
perfect in-memory isolation.

Support multiple stored identities, identified by host and numeric account ID,
with one selected identity for later repository workflows. Switching identities
must not rewrite project repositories or remote URLs.

### Implementation constraints

- Linux Secret Service depends on the desktop D-Bus session and may have no
  provider or default collection. Locate records by attributes rather than a
  session-specific object path, and handle collections re-locking.
- Windows Credential Manager and macOS Keychain may show native prompts or deny
  access. On macOS the requesting application may appear as Java or the launcher
  because Gitmatica shares their process.
- Cross-platform Java keyring wrappers found during research are either small or
  archived and bring native/JNA packaging risks. Prefer narrow, auditable
  platform adapters or an intentionally maintained vendored wrapper. Pin and
  test native dependencies against other Fabric mods instead of allowing
  duplicate JNA loaders.
- Do not add an AES file fallback. A key next to the encrypted file adds no
  meaningful same-user protection; a master-password vault adds substantially
  more UX and recovery complexity than signing in again.

## Lifecycle And Recovery

- Validate the authenticated identity after every successful authorization.
- Recover cleanly from browser-open failure, callback timeout, offline or proxy
  errors, denial, expired codes, revoked tokens, expired tokens, locked vaults,
  cancelled native prompts, and account switching.
- Treat a vault entry disappearing or becoming unreadable as a recoverable
  signed-out state.
- **Disconnect locally** removes Gitmatica's local credential and account
  reference without claiming to revoke it at GitHub.
- **Revoke on GitHub** is a distinct, clearly labelled action. If remote
  revocation fails, do not report the local credential as remotely revoked.
- Show the connected login and granted access clearly, but never display the
  token itself.

## Security Invariants

Tokens, authorization codes, refresh tokens, and token-bearing headers must
never enter:

- A Gitmatica repository or committed manifest.
- `.git/config`, a remote URL, or URL user-info.
- Minecraft chat or commands.
- Normal configuration files or a key stored beside encrypted credentials.
- Logs, progress text, exception details, crash reports, or the clipboard.

Allowlist the GitHub host and authorization paths before opening a browser or
injecting credentials into an HTTP/JGit request. Do not provide an option to
disable TLS certificate verification.

Later remote operations receive an authenticated account identity and a
short-lived credential provider. They do not read vault implementation details
or persist the returned secret. Repository identity, URL, ref, and pull/push
policy belong to the separate remote-support feature.

## Alternatives Considered

- **Fine-grained PAT:** Possible advanced fallback if separately approved, but
  not the primary connection flow. It would require minimal repository grants,
  explicit expiry guidance, identity validation, and the same vault rules.
- **GitHub CLI:** Optional future interoperability for users who already use
  `gh`; it is not a standalone solution and may itself fall back to plaintext
  credential storage when no vault exists.
- **Git Credential Manager:** Strong for native Git, but JGit does not directly
  implement normal Git credential-helper integration. Executing arbitrary
  configured helpers also expands the trust boundary.
- **SSH or native Git:** Useful expert transport options, but they concern
  repository transport rather than connecting a GitHub account and would add an
  external-tool requirement.
- **Encrypted credential file:** Rejected. A key stored beside its ciphertext is
  obfuscation, while a user-master-password design adds recovery and prompting
  complexity that is worse than session-only authorization.

## Minecraft Git Prior Art

No credible Fabric project found in the research sweep used a GitHub App,
browser/device OAuth flow, or OS credential vault. Existing projects still
provide useful UI and operation patterns, but their credential handling is not
a security baseline for Gitmatica.

| Project | Useful precedent | Do not copy |
| --- | --- | --- |
| [MineGIT](https://modrinth.com/mod/minegit) ([source](https://github.com/ModMonster/MineGIT)) | JGit 7.6, graphical onboarding, provider abstraction, repository creation, credential test, background progress | Broad manual PAT guidance, home-grown AES with colocated key material, TLS bypass, uncancellable raw threads, hard-reset/force-push divergence choices |
| [MCGit](https://modrinth.com/mod/mcgit) ([source](https://github.com/itzmetanjim/mcgit)) | JGit-backed build history and concise clone/fetch/pull/push/branch commands | Plaintext credential file, secrets entered through commands, synchronous networking, casual force push |
| [FastBack](https://modrinth.com/mod/fastback) ([source](https://github.com/pcal43/fastback)) | Mature operation locking, progress, preflight identity checks, recovery, and separate local/remote result reporting | Requiring native Git and Git LFS by default; maintaining both native-Git and JGit behavior proved costly |
| [World Syncer](https://modrinth.com/mod/world-syncer) ([source](https://github.com/Tomeshu/world-syncer)) | Separates GitHub API repository discovery/creation from JGit transport | Broad classic PAT in ordinary config and destructive hard-reset/clean synchronization |
| [GitPuller](https://modrinth.com/mod/gitpuller) ([source](https://github.com/Neylz/gitpuller)) | Explicit environment, persistent, and session-only credential precedence | Plaintext config, tokens in commands, synchronous operations, and process-level failure handling |
| [GitMC](https://modrinth.com/mod/gitmc) | Embedded/relocated JGit and distinct feedback for quick versus slow operations | Public source was unavailable, so its authentication and security behavior is not an auditable implementation reference |
| [Backup Scripts](https://modrinth.com/mod/git-backup-mod) ([source](https://github.com/ElectroluxV2/git-backup-fabric-mod)) | Runs expensive backup/network work away from the game thread and delegates system Git authentication | Shell and native-Git requirements, arbitrary command execution, and launcher-dependent configuration |

The recommended combination is MineGIT's approachable UI, World Syncer's
separation between provider API and Git transport, and FastBack's operational
discipline, paired with GitHub App authorization and native credential storage.
Gitmatica should keep JGit as its portable primary engine; FastBack's reasons
for moving away from JGit primarily concern very large binary world backups and
do not automatically apply to Gitmatica's smaller semantic repositories.

## Validation

- Cover browser authorization success, cancellation, timeout, denial, state
  mismatch, invalid callback, token-exchange failure, and identity-validation
  failure.
- Cover device authorization success, cancellation races, expiry, denial,
  polling slowdown, and offline recovery.
- Cover persistent vault round trips, locked and missing vaults, cancelled OS
  prompts, session-only fallback, restart, account switching, local disconnect,
  and remote revocation failure.
- Verify secrets never appear in configuration, repositories, URLs, commands,
  clipboard contents, logs, progress messages, exceptions, or crash reports.
- Confirm connecting or disconnecting an account does not link, clone, fetch,
  pull, push, merge, or otherwise modify a project repository.

## Primary References

- [GitHub App user access tokens and device flow](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app)
- [GitHub OAuth authorization, PKCE, and loopback redirects](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps)
- [GitHub OAuth application security guidance](https://docs.github.com/en/enterprise-cloud@latest/apps/oauth-apps/building-oauth-apps/best-practices-for-creating-an-oauth-app)
- [Freedesktop Secret Service specification](https://specifications.freedesktop.org/secret-service/latest-single/)
- [Microsoft credential-handling guidance](https://learn.microsoft.com/en-us/windows/win32/secbp/handling-passwords)
- [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain-services)
- [GitHub CLI authentication behavior](https://cli.github.com/manual/gh_auth_login)
- [Git Credential Manager credential stores](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/credstores.md)
- [JGit capabilities and missing credential-helper support](https://github.com/eclipse-jgit/jgit)
