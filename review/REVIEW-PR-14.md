# Review: PR #14 — Replace deprecated `DOMNodeInserted` listener in `eb-animation-load.js`

- PR: https://github.com/EssentialBlocks/eb-openverse-block/pull/14
- Branch reviewed: `fix/animation-mutation-event` (copy: `review/openverse-pr-14`), base `dev`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Approve**, with a suggestion to rebase so the PR diff shows only the real change

## What the PR does
`lib/resources/js/eb-animation-load.js` attached a `DOMNodeInserted` listener for the admin panel. The PR replaces it with one capture-phase `change` listener on `document` that checks `event.target.id === 'eb-animation-style'`.

## "18 files" is a display artifact
The branch was cut from `fix/audit-followups-1.2.2`, squash-merged into `dev` as #13, so GitHub still lists that PR's commits. A 3-way merge into `dev` (`git merge-tree origin/dev origin/fix/animation-mutation-event`) is **conflict-free** and changes **1 file**: `lib/resources/js/eb-animation-load.js`, +12 / -16. The line count is higher than in the other standalone plugins (+7 / -11) only because this file is Prettier-formatted (multi-line arguments); the logic is identical.

Suggestion: `git rebase --onto origin/dev 8b6e977` so the PR shows just that one commit.

## Does it work? (tested in a real browser)
Headless Chrome 153, WordPress 7.1.3, Local site, this plugin active alone (PHP 8.2.29). `node --check` and `php -l` pass.

1. `DOMNodeInserted` listeners registered while the editor loads: **1 before, 0 after**. No console errors in either case.
2. Functional test of the admin handler: add `<div class="eb___animated eb___fadeIn">` in view, add `<select id="eb-animation-style">` after page load (like the inspector), fire `change` on the select, wait 900 ms.

| JS | Div classes after `change` |
|---|---|
| old (`dev`) | `eb___animated eb___fadeIn` (nothing happened) |
| PR #14 | `eb__animated eb__fadeIn` (handler ran) |

So this is not only a console-warning cleanup: Chrome removed mutation events in Chrome 127, so on current Chrome the old admin handler **never ran**. The PR fixes a real functional bug. Caveat: the select was injected by script, not driven through the real inspector UI.

## Findings
No blocking bugs.
1. (suggestion) Rebase onto `dev` as above.
2. (info) The script is registered under the shared handle `essential-blocks-eb-animation`; the first standalone EB plugin to register it wins, so every standalone plugin needs this change before the problem is gone. Here the file lives at `lib/resources/js/` (other plugins: `assets/js/`).
3. (info, pre-existing) `replaceAnimationClasses` queries only the main document, not the block-editor iframe canvas. Unchanged behaviour.
4. (info, pre-existing) `waitForElement` has an unguarded `observer.observe(document.body, ...)`. It did not fire here: in this plugin the script loads only in the main editor page, not in the canvas iframe (see Typing Text's REVIEW-PR-34.md for the case where it does).

## Observation outside this PR: the block icon on the highlighted row
The other standalone plugins (Testimonial, Toggle Content, Typing Text) each have a PR that keeps the block icon visible on the blue highlighted `/` inserter row and in List View. **Openverse has no such PR**, and the same problem is visible here: the icon is a blue/purple gradient (`fill="#275EFF"` plus `paint0_linear` gradient) that nearly disappears on the `rgb(56,88,233)` highlighted row (`review/screenshots/openverse-icon-on-highlighted-row.png`). The other plugins' CSS targets `#6C3BFF` fills, so it would **not** match this icon as is. If wanted, a separate icon fix needs rules for this icon's own colours. Not part of #14 and not verified beyond the screenshot.
