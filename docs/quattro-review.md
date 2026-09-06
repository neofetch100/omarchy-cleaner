# Quattro compatibility review

Reviewed [PR #5](https://github.com/maxart/omarchy-cleaner/pull/5) at `d268284cb1fbd5f6fdee3bdff9576943dd9056c3`, including its three discussion comments and commit history. There were no submitted reviews or inline review comments. The author reports use on 4.0.1; the comments contain no automated failure-path coverage.

The comparison used actual upstream sources from released [4.0.2](https://github.com/omacom/omarchy/tree/346e69e1cec6c4e8924531874af6ba010a1bc99e) and [Quattro at 959e49d](https://github.com/omacom/omarchy/tree/959e49dc52a60d35b299d4e13372bc74dd0797ae), plus 4.0.0 and 3.8.4 compatibility fixtures. Source files and revisions are recorded in [fixture provenance](../tests/fixtures/README.md).

The proposal should not be accepted unchanged. Its catalogue updates and use of personal Lua overrides are useful, but the following cases require correction. All findings below have high confidence from source inspection and/or isolated reproductions.

| Finding | Evidence in proposed script | Result in this update |
| --- | --- | --- |
| Any script mentioning `mise` is considered removable, including personal wrappers and comments. | `is_npm_cli_installed`, line 294 | Match complete known wrappers and command/package pairs; reject symlinks; recheck before deleting. Hermes uses upstream's read-only ownership query. |
| Appended unbinds can disable personal replacement bindings. Commented unbind examples are also treated as active. | `append_lua_unbinds`, lines 597–622 | Prepend default unbinds before personal overrides; recognize only active standalone unbinds and avoid duplicates. |
| Backup/write errors do not stop edits or prevent a success report. Two edits in one second can overwrite the first backup. | `remove_bindings_from_file`, lines 530–565; `append_lua_unbinds`, lines 611–625 | One checked backup with a unique timestamped filename, one atomic replacement, preserved file mode, concurrent-edit checks, and visible cleanup failures. Part of this predates the proposal. |
| Shortcuts are removed before uninstall success is known. | `remove_items`, lines 1026–1073 | Clean only shortcuts associated with successful removals. This ordering issue predates the proposal. |
| Shortcut-only webapps can report success even when cleanup was declined. | `get_installed_webapps`, lines 311–315; `remove_webapps`, lines 846–853 | Label shortcut-only selections explicitly, skip them when cleanup is declined, and count cleanup failures. |
| URLs are matched by unanchored regex; removal commands interpolate paths through `bash -c`. | `find_bindings_in_file`, lines 423–429; `remove_npm_clis`, line 910 | Compare URL hostnames literally and pass removal targets as arguments. These issues predate the proposal. |

PR #5 covers 4.0.2's package and CLI lists. Current Quattro additionally installs `agy`, `hey`, `ori`, and `hermes`; those are now offered with ownership checks. The package catalogue retains older names, and the known Docker/Disk Usage desktop launchers can be selected separately. Packages are removed together so selected Docker dependencies do not block one another due to removal order.

The implementation retains Bash, four-space indentation, the gum palette, newline-preserving selection, itemized confirmation, and the existing privilege split. The proposed branch contains tool-attribution commit trailers, which conflict with this repository's commit convention; the replacement commit does not use them.

Validation uses Bash syntax checking, ShellCheck, and offline regression tests with stubbed pacman/sudo/removal helpers. Tests include real upstream wrapper templates and bindings, user overrides, long Lua comments/strings, failure paths, cancellation, and the piped entry point. No installed software was removed during validation. A live Omarchy VM/TUI smoke test has not been performed. Multiline/computed Lua bindings and ambiguous long-string layouts are deliberately left unchanged; wrapper removal keeps downloaded runtimes, caches, and user data.
