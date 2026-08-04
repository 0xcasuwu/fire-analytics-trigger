# fire-analytics-trigger

Public, unprivileged trigger for the FIRE analytics data refresh.

The analytics site is a static export of a **private** repo, so its refresh pipeline cannot be
started by site visitors directly — GitHub allows nobody to touch a private repo's Actions without
a credential. This repo is the bridge: **opening an issue here titled `refresh` asks the pipeline
to run.** Any GitHub account can do it; no credential ever reaches the browser.

任何 GitHub 账号都可以在此仓库开一个标题为 `refresh` 的 issue 来请求刷新 FIRE 数据看板。

## How it works

1. You open an issue titled `refresh` (the site's "Refresh now" button prefills it).
2. The [`dispatch-refresh`](.github/workflows/dispatch-refresh.yml) workflow (running here, on
   GitHub's free public-repo minutes) checks whether a refresh is already running or just ran,
   then dispatches the private repo's `refresh.yml` using a repo secret
   (`REFRESH_DISPATCH_TOKEN` — a fine-grained PAT scoped to only the private repo, Actions
   read & write). The secret lives server-side in Actions; it is never shipped to any browser.
3. The workflow labels the issue with its progress (`refresh:running` → `refresh:done` /
   `refresh:failed` / `refresh:skipped`), comments the outcome, and closes it. The site's header
   button polls this repo's public issues API to mirror that status inline.

A refresh takes ~15–22 minutes and only publishes data that passes the pipeline's invariant
suite — a `refresh:failed` outcome usually means an upstream data source is inconsistent and the
gate refused to publish, not that your request was wrong.

## Rate limiting

One refresh at a time; requests within 45 minutes of the last run are skipped with a comment
(the data is already at most that old). Issues not titled `refresh` are ignored by automation.
