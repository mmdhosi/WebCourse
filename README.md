# Internet Engineering Git Assignment Answers
**Mohammad Hossein Hassanpour**  
**Student ID:** 40131009

---

## Question 1

### Part 1
The `merge` command combines the changes of a branch with the current branch and does not rewrite previous commits. If two branches have independent commits after their common ancestor, a merge commit with two parents is usually created; therefore, the main development paths remain visible in the history. If the current branch has no independent commits, the merge can be performed as a fast-forward; in this case, the branch pointer moves forward, and a merge commit is not necessary.

To merge a completed feature into a shared branch after team coordination and reviewing the changes, sample commands are as follows:

```bash
git switch main
git merge feature-description
```

This method is suitable for shared branches because it preserves the hashes of existing commits and maintains the relationship between branches in the history.

### Part 2
The `rebase` command reapplies the exclusive commits of the current branch onto a new base; as a result, new hashes are usually generated for the reapplied commits. Normally, the history becomes more linear, and no separate merge commit is produced. The main difference is that `merge` connects the existing history together, whereas `rebase` rewrites the commit history of the branch.

To update a local branch that has not yet been shared with others, one can write:

```bash
git switch feature-description
git rebase main
```

This method is suitable because the feature changes are placed on top of the latest state of `main`, making the history more readable. Rebasing commits that others have used as the basis of their work without coordination is not appropriate; because their hashes change, separating teammates' history from the new history.

---

## Question 2

### Part 1
A change section or `hunk` is a set of closely located changes in a file that is displayed along with a few context lines in the `diff` output. Its header is usually marked with `@@` symbols and shows the line ranges of the old and new versions. Deleted lines are marked with a minus (`-`) and added lines with a plus (`+`). Context lines are there to understand the location of the change.

### Part 2
For a tracked file, I first review the changes and then run interactive staging:

```bash
git diff README.md
git add -p README.md
```

Git displays each hunk separately. The `y` option stages that hunk into the staging area, and `n` skips it. The `s` option splits the hunk into smaller parts if possible. The `e` option allows manual editing of the patch, and `q` quits the selection. The `?` option displays the help for the options. Unselected hunks remain in the working tree.

After selection, I check the staging status and the remaining changes:

```bash
git diff --cached README.md
git diff README.md
git commit -m "Update installation instructions"
```

A normal commit records the contents of the staging area; therefore, staged changes in other files must also be checked. This method allows changes related to a single goal—for example, fixing the installation guide—to be separated from unrelated changes, creating more cohesive commits for review and reverting.

---

## Question 3

### Part 1
The `bisect` command uses binary search in the history to find the first commit where the bug under investigation appeared. I specify one good version and one bad version, and then Git selects a middle commit for testing. By declaring it good or bad, the search range becomes smaller. This continues until the first bad commit is identified.

### Part 2
For example, I assume version `v1.0` runs the user login feature properly, and the current version has broken this feature. With a clean working tree, I start the search:

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0
```

In each selected commit, I run the exact same test for user login. If there is no error, I run the first command, and if there is an error, I run the second command (these two commands are alternative choices and should not be run back-to-back for a single test result):

```bash
git bisect good
git bisect bad
```

After the first bad commit is announced, I inspect it and exit the search mode:

```bash
git show <bad-commit>
git bisect reset
```

The `<bad-commit>` expression must be replaced with the found commit hash. Here, the `reset` command reselects the initial branch or commit from before the search and does not delete commits from the history. If a version cannot be tested, `git bisect skip` can be used; however, skipping versions may make it difficult to uniquely identify the source of the bug.

### Part 3
This method is useful for a bug introduced among a large number of commits because, instead of testing all versions, the number of tests is normally logarithmic. For about 64 versions, around 6 intermediate tests are sufficient. The test must be repeatable, and the good version must truly not have the bug in question; otherwise, the search result will not be reliable.

---

## Question 4

### Part 1
To view the overall status and the two categories of changes, I use the following commands:

```bash
git status
git diff README.md
git diff --cached README.md
```

The `status` command shows staged changes, unstaged changes, and untracked files. The `diff` command without the `--cached` option compares the working tree with the staging area, thus showing unstaged edits. The `diff` command with the `--cached` option compares the staging area with the last commit and shows changes that are ready for the next commit (the `--staged` option is synonymous with `--cached`).

Because the file was edited again after staging, it can appear in both categories simultaneously. Untracked files in the `tmp` folder are not displayed in a normal `diff`, but they are identifiable in `status`.

### Part 2
By making a normal commit without the `-a` option, only the staged version is recorded:

```bash
git commit -m "Update README"
```

Subsequent edits to `README.md` that have not been staged again do not enter this commit and remain in the working tree. Untracked files in `tmp` are also not committed. The reason for this behavior is that a normal commit is built from the staging area, not directly from the latest content of all working files. If other files were previously staged, their changes will also be included in this same commit.

### Part 3
In the `.gitignore` file at the root of the repository, I add the following line so that the `tmp` folder at the root and its untracked contents are ignored:

```gitignore
/tmp/
```

Then I stage the configuration file itself for committing:

```bash
git add .gitignore
git commit -m "Ignore temporary files"
```

This setting is not sufficient for a file that was previously tracked, because `.gitignore` does not stop tracking files already present in the staging area/index. For example, if `tmp/cache.txt` was previously tracked, I remove it from the staging area while keeping the local copy:

```bash
git rm --cached tmp/cache.txt
git commit -m "Stop tracking temporary cache"
```

The second commit records the removal of the file from the tracked version of the repository, while the file remains on the device. In the question's scenario, since the `tmp` files are all untracked, this extra step is not necessary.

---

## Question 5

### Part 1
Both branches have modified the exact same line in two different ways relative to their common ancestor; therefore, Git cannot automatically determine the final content of that line. Modifying a file by two people does not necessarily create a conflict; independent changes in separate sections or identical changes are often merged automatically. The problem here is incompatible changes in a shared section.

### Part 2
Assuming I am on `main` and the merge is paused, I check the unresolved files:

```bash
git status
git diff --name-only --diff-filter=U
git diff README.md
```

In the standard conflict marker format, the section after `<<<<<<< HEAD` is the text from the current branch, which is `main`. The `=======` symbol separates the two versions, and the section after it up to `>>>>>>> feature-description` is the text from the incoming branch. These markers indicate the boundary of the conflict and are not part of the file's final text.

To view the exact two versions being merged, one can also write:

```bash
git show :2:README.md
git show :3:README.md
```

In this standard merge, stage `2` is the version of the current branch, and stage `3` is the version of the incoming branch.

### Part 3
1. I open the `README.md` file and, in coordination with my teammate, write a final text that preserves the information from both changes. Simply deleting one version is not enough; the combined text must be semantically and grammatically correct.

2. I remove all start, separator, and end conflict markers from the file, and then review the final content and differences. To check for any remaining markers and whitespace errors, I run the following command:

```bash
git diff --check
```

3. I stage the resolved file (this tells Git that its conflict has been resolved):

```bash
git add README.md
```

4. I verify the staged content and ensure there are no unresolved files left:

```bash
git diff --cached --check
git diff --cached README.md
git diff --name-only --diff-filter=U
git status
```

The output of the command listing unresolved files must be empty. The absence of markers alone does not guarantee semantic correctness, so the final text must also be reviewed. If the project has related tests, I run them as well.

5. By making a commit, I complete the merge:

```bash
git commit -m "Merge feature-description and combine README changes"
git status
```

The merge commit is created, and if there are no other changes, the working tree will be clean. If the result needs to be shared, I push it to the shared branch after these steps.

### Part 4
As long as the merge is in progress and the merge commit has not yet been recorded, I cancel it with the following command:

```bash
git merge --abort
```

Given the assumption that the working tree was clean before the merge, Git restores the state prior to the start of the merge. The previous commits of both branches are preserved, and the incomplete merge does not continue.

---

## Question 6

### Part 1
To discard the unstaged edits of this specific file, the following command is appropriate:

```bash
git restore --worktree notes.txt
```

The default source for restoring the working tree is the staging area. Since, according to the question's assumption, the staged version is identical to the last commit, the working file also reverts to that same version. Only the unstaged edits of `notes.txt` are discarded; the staging area, commit history, and other files do not change.

### Part 2
To unstage the changes from the staging area while preserving the edits in the working file, I write:

```bash
git restore --staged notes.txt
```

With the `--staged` option, the default source is the last commit; therefore, the version of `notes.txt` in the staging area becomes identical to `HEAD`. The content of the working file remains untouched, and its changes appear as unstaged. No commit is created or deleted, and the history does not change.

### Part 3
To undo the effect of a published commit, I use `revert`. Assuming the hash of the bad commit is `abc1234` (this hash must be replaced with the actual hash):

```bash
git switch main
git revert abc1234
git push origin main
```

Assuming there are no conflicts, the `revert` command applies the inverse change of that commit and creates a new commit. The working files and staging area are updated according to the result of this commit, and upon successful completion, the working tree is usually clean. The mistaken commit remains in the history, and the new commit counteracts its effect. This action does not necessarily remove all subsequent changes.

This method is suitable for a shared branch because it preserves the published history. Deleting a commit by moving the branch pointer and force-pushing alters the history that teammates have already pulled and may also make subsequent commits unreachable from the branch; therefore, in this scenario, reverting with a new commit is a more appropriate choice.

---

> **AI Usage Note:** The formatting of this file and the translation to english were prepared with the help of ChatGPT.
