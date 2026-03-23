## Treage version update runbook

- [ ] Update version string in `treage.js` header comment
- [ ] Update footer version string in `TREAGE_BODY` inside `treage.js`
- [ ] Update CDN refs in all consumer files from old tag to new tag:
  - `index.html`
  - `examples/starter.html`
  - `examples/boilerplate.html`
  - `examples/elasticsearch-red-recovery.html`
  - `playground/playground.html` (buildPreviewHTML function)
  - etc...
  ```
  grep -rn "treage@" index.html examples/ playground/playground.html
  grep -rln "treage@X.Y.Z" index.html examples/ playground/playground.html | xargs sed -i '' 's/treage@X\.Y\.Z/treage@A.B.C/g'
  ```

- [ ] Commit all changes atomically:
  ```
  git add -A
  git commit -m "feat: vX.X.X — short description"
  ```

- [ ] Push to main:
  ```
  git push origin main
  ```

- [ ] Tag the commit:
  ```
  git tag vX.X.X
  git push origin vX.X.X
  ```

- [ ] (optional) Purge jsDelivr cache: 
  ( only needed if re-pushing to an existing tag)
  ```
  https://purge.jsdelivr.net/gh/rseldner/treage@X.X.X/treage.js
  ```

- [ ] Verify CDN is live:
  ```
  https://cdn.jsdelivr.net/gh/rseldner/treage@X.X.X/treage.js
  ```

- [ ] Close the relevant GitHub issue with commit SHA reference
(if needed): get last commit
  ```
  git log --oneline -1
  ```
- [ ] Review Open Issues
   ```
   gh issue list --state open --json number,title,labels,updatedAt,body
   ```