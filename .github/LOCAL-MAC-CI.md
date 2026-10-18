# Local Mac mini GitHub Actions CI

Runner: `mac-mini-m4`, labels `[mac-mini-m4, macos-arm64]`, native macOS ARM64.
Workflow: `.github/workflows/mac-mini-ci.yml`. Runs on pushes to main/master or the trusted setup branch and via workflow_dispatch; no pull_request/fork trigger on this user-account host.

Scope: **Static application JavaScript syntax and extension manifests; browser interaction NOT_RUN**.

Commands:
```bash
node - <<'CHECK'
const cp=require('node:child_process'),fs=require('node:fs');
const files=cp.execFileSync('git',['ls-files','-z']).toString().split('\0').filter(Boolean);
const js=files.filter(p=>p.endsWith('.js')&&!p.includes('node_modules/')&&!p.endsWith('.min.js'));
for(const p of js)cp.execFileSync(process.execPath,['--check',p],{stdio:'inherit'});
const manifests=files.filter(p=>/(^|\/)manifest\.json$/.test(p));
for(const p of manifests)JSON.parse(fs.readFileSync(p,'utf8'));
if(!js.length&&!manifests.length&&!files.some(p=>p.endsWith('.html')))throw Error('No static app sources');
console.log('JavaScript files checked:',js.length,'extension manifests:',manifests.length);
console.log('Browser interaction: NOT_RUN');
CHECK
```

No deploy, publishing, host installer/apply, Apple Notes automation or signing is performed. Failures are preserved; jobs do not use continue-on-error. A source/data check is not a native application or hardware test. Initial run results belong to the actual GitHub job links, not this plan.

Each repository registration can run one job; GitLab concurrent=3 does not cap aggregate GitHub work. Initial rollout is dispatched in batches of at most three repositories.

Test reports are copied to `~/.local/share/github-ci-reports/<owner>/<repo>/<run-id>/` on the Mac. GitHub storage is quota-limited; remote artifact upload is opt-in with repository variable `MAC_CI_UPLOAD_ARTIFACTS=true`. All test/build failures still fail the job; only the optional report transport is disabled by default.
