# .github

Public organization configuration and shared defaults for repositories in `srcery-colors`.

## Renovate

Repositories can inherit the shared Renovate preset with:

```json
{
  "extends": ["github>srcery-colors/.github:renovate-config"]
}
```

The preset defines the organization-wide dependency update policy. Repository-specific managers, grouping, and custom-manager behavior should stay in the consuming repository.

### Automerge policy

The shared preset enables automerge for routine updates that should not normally require manual handling:

- dependency pinning updates;
- patch and minor updates for stable dependencies whose current version is `>=1.0`;
- GitHub Actions digest, patch, and minor updates;
- scheduled lock-file maintenance.

Pre-1.0 dependencies are intentionally excluded from the generic patch/minor automerge rule because `0.x` releases may contain breaking changes without a major version bump.

Major updates are kept visible but require explicit approval from the Renovate Dependency Dashboard before Renovate creates their pull requests.

Automerge eligibility does not bypass repository merge requirements. Renovate uses GitHub platform automerge, so repositories must provide an appropriate CI gate before relying on the shared policy.

### Required branch ruleset

Repositories using the shared automerge policy should protect their default branch with an active GitHub branch ruleset.

At minimum, configure the ruleset to:

- target the default branch;
- require changes to be merged through a pull request;
- require the repository's relevant CI/status check before merge;
- allow zero required approving reviews for fully automated dependency updates, unless human approval is intentionally required;
- keep GitHub repository `Allow auto-merge` enabled.

The required status check is the safety boundary for platform automerge. Renovate can mark an eligible pull request for auto-merge while CI is still pending; GitHub will then merge it as soon as the required check succeeds instead of waiting for a later Renovate run.

A typical flow is:

```text
Renovate opens an eligible dependency PR
            |
            v
GitHub auto-merge is enabled for the PR
            |
            v
required repository CI runs
            |
      +-----+-----+
      |           |
      v           v
    passes       fails
      |           |
      v           v
GitHub merges   PR remains open
immediately     for investigation
```

Do not rely on automerge in repositories without meaningful required CI checks. If a repository cannot provide a suitable validation gate, override or disable the inherited automerge behavior locally.
