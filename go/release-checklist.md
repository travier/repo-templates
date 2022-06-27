---
# Template generated by https://github.com/coreos/repo-templates; do not edit downstream
---

Release checklist:

Tagging:
 - [ ] Write release notes in `docs/release-notes.md`. Get them reviewed and merged
     - [ ] If doing a branched release, also include a PR to merge the `docs/release-notes.md` changes into main
 - [ ] Ensure your local copy is up to date with the upstream main branch (`git@github.com:coreos/{{ git_repo }}.git`)
 - [ ] Ensure your working directory is clean (`git clean -fdx`)
 - [ ] Ensure you can sign commits and any yubikeys/smartcards are plugged in
 - [ ] Run `./tag_release.sh <vX.Y.z> <git commit hash>`
 - [ ] Push that tag to GitHub

Fedora packaging:
 - [ ] Update the spec file in [Fedora](https://src.fedoraproject.org/rpms/{{ fedora_package }}):
   - Bump the `Version`
   - Switch the `Release` back to `1%{?dist}`
   - Remove any patches obsoleted by the new release
   - Run `go-mods-to-bundled-provides.py | sort` while inside of the `{{ git_repo }}` directory you ran `./tag_release` from & copy output into spec file in `# Main package provides` section
   - Update changelog
 - [ ] Run `spectool -g -S {{ fedora_package }}.spec`
 - [ ] Run `kinit your_fas_account@FEDORAPROJECT.ORG`
 - [ ] Run `fedpkg new-sources tarball-name`
 - [ ] PR the changes in [Fedora](https://src.fedoraproject.org/rpms/{{ fedora_package }})
 - [ ] Once the PR merges to rawhide, merge rawhide into the other relevant branches (e.g. f{{ current_fedora }}) then push those, for example:
   ```bash
   git checkout rawhide
   git pull --ff-only
   git checkout f{{ current_fedora }}
   git merge --ff-only rawhide
   git push origin f{{ current_fedora }}
   ```
 - [ ] On each of those branches run `fedpkg build` including rawhide.
 - [ ] Once the builds have finished, submit them to [bodhi](https://bodhi.fedoraproject.org/updates/new), filling in:
   - `{{ fedora_package }}` for `Packages`
   - Selecting the build(s) that just completed, except for the rawhide one (which gets submitted automatically)
   - Writing brief release notes like "New upstream release; see release notes at `link to docs/release-notes.md on GH tag`"
   - Leave `Update name` blank
   - `Type`, `Severity` and `Suggestion` can be left as `unspecified` unless it is a security release. In that case select `security` with the appropriate severity.
   - `Stable karma` and `Unstable` karma can be set to `2` and `-1`, respectively.
{%- if do_fast_track %}
 - [ ] [Submit a fast-track](https://github.com/coreos/fedora-coreos-config/actions/workflows/add-override.yml) for FCOS testing-devel
 - [ ] [Submit a fast-track](https://github.com/coreos/fedora-coreos-config/actions/workflows/add-override.yml) for FCOS next-devel if it is [open](https://github.com/coreos/fedora-coreos-pipeline/blob/main/next-devel/README.md)
{%- endif %}

GitHub release:
 - [ ] Wait until the Bodhi update shows "Signed :heavy_check_mark:" in the Metadata box.
 - [ ] [File a releng ticket](https://pagure.io/releng/new_issue) based on [prior signing tickets]({{ sample_signing_ticket }}).
   - [ ] Update the script and test it locally by running it like `FAKESIGN=1 ./script`
     - [ ] If a new Fedora release has gone stable, update the signing key in the script to use the new Fedora signing key [found here](https://getfedora.org/security).
 - [ ] Wait for the ticket to be closed
 - [ ] Download the artifacts and signatures
 - [ ] Verify the signatures
 - [ ] Find the new tag in the [GitHub tag list](https://github.com/coreos/{{ git_repo }}/tags) and click the triple dots menu, and create a draft release for it.
 - [ ] Upload all the release artifacts and their signatures. Copy and paste the release notes from `docs/release-notes.md` here as well.
   - [ ] If the signing key has changed, note the change in the GitHub release notes as done [here](https://github.com/coreos/{{ git_repo }}/releases/tag/{{ sample_signing_key_update_tag }}).
 - [ ] Publish the release

Quay release:
 - [ ] Visit the [Quay tags page](https://quay.io/repository/{{ quay_repo }}?tab=tags) and wait for a versioned tag to appear
 - [ ] Click the gear next to the tag, select "Add New Tag", enter `release`, and confirm
{%- for repo in quay_legacy_repos|default(value=[]) %}
 - [ ] Visit the [Quay tags page](https://quay.io/repository/{{ repo }}?tab=tags) for the legacy `{{ repo }}` repo and wait for a versioned tag to appear
 - [ ] Click the gear next to the tag, select "Add New Tag", enter `release`, and confirm
{%- endfor %}

RHCOS packaging for the current RHCOS development release:
 - [ ] Update the spec file
   - Bump the `Version`
   - Switch the `Release` back to `1%{?dist}`
   - Remove any patches obsoleted by the new release
   - Run `go-mods-to-bundled-provides.py | sort` while inside of the `{{ git_repo }}` directory you ran `./tag_release` from & copy output into spec file in `# Main package provides` section
   - Update changelog
 - [ ] Run `spectool -g -S {{ rhel8_package }}.spec`
 - [ ] Run `kinit your_account@REDHAT.COM`
 - [ ] Run `rhpkg new-sources tarball-name`
 - [ ] PR the changes
 - [ ] Get the PR reviewed and merge it
 - [ ] Update your local repo and run `rhpkg build`
{%- if do_ocp_mirror %}
 - [ ] File ticket similar to [this one](https://issues.redhat.com/browse/ART-3711) to sync the new version to mirror.openshift.com
{%- endif %}

CentOS Stream 9 packaging:
  - [ ] To be written

{% if macports_package -%}
Housekeeping:
 - [ ] Ask bgilbert to update the [MacPorts package](https://github.com/macports/macports-ports/tree/master/{{ macports_package }})
{%- endif %}