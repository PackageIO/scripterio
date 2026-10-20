# Release Strategy

Releases are published by `.github/workflows/release_package.yml` when a version tag matching `v*` is pushed. The tag must match the version in `package.json` exactly.

## One-Time npm Setup

The package name is `@therootsoft/scripterio`. The npm scope is separate from the GitHub organization, so confirm that the `therootsoft` npm organization exists and that a maintainer can publish to it.

For the first publication, a maintainer with npm publish access must publish the package once from the repository root:

```sh
npm login
npm ci
npm test
npm publish --access public
```

After the package exists on npm, configure **Trusted Publishing** in the package's npm settings:

- Provider: GitHub Actions
- Owner: `TheRootSoft`
- Repository: `scripterio`
- Workflow filename: `release_package.yml`
- Allow direct publishing with `npm publish`

The workflow uses GitHub's OIDC identity and does not need an `NPM_TOKEN` secret. Keep the package public and the repository public for npm provenance.

## Prepare a Release

1. Merge release changes into `master` and confirm CI passes.
2. Update the `Unreleased` section in `CHANGELOG.md` with the release notes and date.
3. Choose a semantic version increment and update both package files. For example, for a patch release:

   ```sh
   npm version patch --no-git-tag-version
   ```

   Use `minor` or `major` instead when appropriate. Review the changes to `package.json` and `package-lock.json` and commit them together with `CHANGELOG.md`.
4. Push the release commit to `master` before creating the tag.
5. Create and push a tag matching the package version:

   ```sh
   VERSION="$(node -p "require('./package.json').version")"
   git tag "v$VERSION"
   git push origin "v$VERSION"
   ```

The tag push starts the release workflow. It checks the tag/version match, installs dependencies, runs tests, verifies the package contents with `npm pack --dry-run`, publishes the package publicly, and creates a GitHub release with generated notes.

## Release Constraints

- This workflow currently publishes to npm's default `latest` dist-tag. Use stable versions only; prerelease publishing to a separate tag such as `beta` is not configured.
- Do not move or reuse a version tag after publication. If a released package needs a fix, prepare and publish a new version.
- If publishing fails because npm scope permissions or Trusted Publishing are not configured, correct the npm settings and rerun the failed workflow for the same tag.