# Release Guide

Remove cached ansible collections, commit (with signature),
``` bash
find . -type d -name '.ansible' -exec rm -rfv {} \;
git commit -S
git tag {VERSION} {COMMIT}
git push
git push --tags
```

Build release artifact. Sanity check size/contents for unwanted inclusions.
``` bash
ansible-galaxy collection build
```

Create new release on GitHub using tag:
* Generate release notes (right of tag)
* Use galaxy.yml for commit template. See previous releases.
* Manually attach build artifact to tagged release
