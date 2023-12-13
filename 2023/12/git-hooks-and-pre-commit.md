%- # TOC start (generated with https://github.com/derlin/bitdowntoc) -%}

- [Hooks and Webhooks](#hooks-and-webhooks)
- [Git Hook Basics](#git-hook-basics)
- [Practical Git Hook Usage](#practical-git-hook-usage)
- [Git Hook Automation With pre-commit](#git-hook-automation-with-precommit)
- [Working With Contributed pre-commit Code](#working-with-contributed-precommit-code)
- [File Filtering](#file-filtering)
- [Conclusion](#conclusion)

{%- # TOC end -%}

Git is a powerful tool to enable developers to organize and share their code. One of the more interesting features is hooks. This allows for execution of scripts at certain points in the git workflow. In this article I'll be showcasing one of the simple git workflow events: pre-commit. I'll also be introducing a tool to help with automation of it, the aptly named [pre-commit](https://pre-commit.com/). Note that the code used for this is the final step of my python beginners series and can be [found in a code repository](https://github.com/cwgem/my-pdm-project) in GitHub. Simply click on the green "Code" button and select "Download ZIP". Then extract it into your preferred directory of choice.

## Hooks and Webhooks

To clear up any potential confusion, Git hooks and GitHub webhooks are different entities (though GitHub webhooks most likely is powered by git hooks). Git hooks are specific to the git software package. [GitHub webhooks](https://docs.github.com/en/webhooks) are a feature of the GitHub platform that pushes JSON payloads based on various GitHub related events. The same goes for similar source control service sites such as [GitLab](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html) 

## Git Hook Basics

A git hook is essentially a script that is executed at a certain point in the git workflow. The basic rules of hooks are:

- Set as executable
- Strip the extension (with the exception of certain windows extensions such as `.exe`)
- Located in a `.git/hooks` directory under the repository parent directory

To see how this looks download the code mentioned in the introduction paragraph and make sure there's no `.git` directory in it (remove it if there is). Then run `git init` to initialize the repository:

```
$ git init
$ git branch -m main
```

The second command ensures the default branch is main, which is fairly common for new repositories on major code hosting sites. After the git repository has been initialized it's time to look at the contents of `.git/hooks`:

```
$ cd .git/hooks
$ ls -1
applypatch-msg.sample
commit-msg.sample
fsmonitor-watchman.sample
post-update.sample
pre-applypatch.sample
pre-commit.sample
pre-merge-commit.sample
prepare-commit-msg.sample
pre-push.sample
pre-rebase.sample
pre-receive.sample
push-to-checkout.sample
update.sample
```

Each of these has examples. To make them work we'll need to make a copy with `.sample` removed and ensure it's executable. Let's take a look at the `pre-commit.sample` contents:

```bash
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

if git rev-parse --verify HEAD >/dev/null 2>&1
then
        against=HEAD
else
        # Initial commit: diff against an empty tree object
        against=$(git hash-object -t tree /dev/null)
fi

# If you want to allow non-ASCII filenames set this variable to true.
allownonascii=$(git config --type=bool hooks.allownonascii)

# Redirect output to stderr.
exec 1>&2

# Cross platform projects tend to avoid non-ASCII filenames; prevent
# them from being added to the repository. We exploit the fact that the
# printable range starts at the space character and ends with tilde.
if [ "$allownonascii" != "true" ] &&
        # Note that the use of brackets around a tr range is ok here, (it's
        # even required, for portability to Solaris 10's /usr/bin/tr), since
        # the square bracket bytes happen to fall in the designated range.
        test $(git diff --cached --name-only --diff-filter=A -z $against |
          LC_ALL=C tr -d '[ -~]\0' | wc -c) != 0
then
        cat <<\EOF
Error: Attempt to add a non-ASCII file name.

This can cause problems if you want to work with people on other platforms.

To be portable it is advisable to rename the file.

If you know what you are doing you can disable this check using:

  git config hooks.allownonascii true
EOF
        exit 1
fi

# If there are whitespace errors, print the offending file names and fail.
exec git diff-index --check --cached $against --
```

So this will do a few basic checks against file names and whitespace issues. As one of the comments mentions `To enable this hook, rename this file to "pre-commit"`. We can then just run `git commit` afterwards to run it:

```
$ cp pre-commit.sample pre-commit
$ cd ../../
$ git commit
```

At this point nothing happened as nothing was added. I'll go ahead and add a file with a Japanese name to see what happens:

```
$ touch テスト
$ git add テスト
$ git commit
$ git commit
Error: Attempt to add a non-ASCII file name.

This can cause problems if you want to work with people on other platforms.

To be portable it is advisable to rename the file.

If you know what you are doing you can disable this check using:

  git config hooks.allownonascii true
```

As you can see our hook is working and is preventing the non ASCII named file from being committed. I'll go ahead and cleanup the test:

```
$ git reset テスト
$ rm テスト
```

## Practical Git Hook Usage

So we've seen a very basic example of how hooks work. Now the more practical example would be to use it to run tests on code before committing. Note that I don't recommend this if your project is in the prototyping phase where you may be doing many commits and may not have tests setup due to the frequency of code architecture changes. In this case the project in question has [tox](https://tox.wiki/en/4.11.4/) setup which runs various python linting and tests. Before running the examples you'll need to ensure `pdm` is installed and running under python 3.11. Instructions for this can be found in my [pdm tutorial](https://dev.to/cwprogram/beginning-python-project-management-with-pdm-13m0), or you can install it on your own. Once installed we'll make sure the necessary packages are available for tox to work with:

```
$ pdm install
```

Then I'll edit the `.git/hooks/pre-commit` file to contain the following:

```bash
#!/bin/sh
pdm run tox

if [ $? -ne 0 ]; then
  echo "tox checks failed" >&2
  exit 1
fi
```

Now when `git commit` runs:

```
$ git commit
lint: install_deps> pdm sync --no-self --group testing --group lint
lint: commands[0]> flake8
```

`tox` is run to initiate all the checks. As is the code should be in working condition so you'll see a commit screen after the tox run. Go ahead and close it out without putting a message to abort the comment. When there's an issue with something (I went ahead and commented out one of the imports):

```
$ git commit
lint: install_deps> pdm sync --no-self --group testing --group lint
lint: commands[0]> flake8
<snip>
  lint: FAIL code 1 (4.34=setup[3.69]+cmd[0.65] seconds)
  test: FAIL code 1 (11.80=setup[10.09]+cmd[1.71] seconds)
  docs: OK (14.44=setup[11.79]+cmd[2.65] seconds)
  evaluation failed :( (30.79 seconds)
tox checks failed
```

A message at the end shows that `tox` has failed to run and no commit screen is displayed.

## Git Hook Automation With pre-commit

Given how useful checking code quality is before committing, there's actually a framework around git hooks called [pre-commit](https://pre-commit.com/). It's somewhat like a mini-GitHub Actions that can be used to run various commands during the `pre-commit` phase. This is where you normally want most CI/CD like checks. Before continuing you'll want to remove the existing hook or you'll end up having duplicated tool runs. We'll also go ahead and add all the files in the repo so there's something to check:

```
$ rm .git/hooks/pre-commit
$ git add .
```

Given how useful `pre-commit` is across projects I generally recommend installing via `pip install --user`, making it part of a tooling virtual environment, or using [pipx](https://github.com/pypa/pipx):

```
$ pip install --user pre-commit
$ pipx install pre-commit
```

Next we'll need to create a YAML configuration file in the root of our repository called `.pre-commit-config.yaml`. To make things simple you can bootstrap a basic one via:

```
$ pre-commit sample-config > .pre-commit-config.yaml
```

As is the file looks something like this:

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
```

This already has some great entries for basic git related checks. Now we'll add a new entry to cover tox checking:

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: local
    hooks:
    -   id: tox check
        name: tox-validation
        entry: pdm run tox
        language: system
        types: [python]
        pass_filenames: false
```

`repo: local` tells pre-commit that what I'm doing is not a managed action by code someone else wrote. It's completely custom code written by me. The code itself will run `pdm run tox` in the system environment when it finds python files have been modified. This is one of the more powerful features of `pre-commit` over the standard git hook. There's more flexibility on when certain commands get run. You don't need to check tests if only the README file was updated. Now to actually have this integrated with our workflow we'll need to run:

```
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
```

The generated file is simply an entry point to the actual `pre-commit` program which handles the processing of what we want to do:

```bash
#!/usr/bin/env bash
# File generated by pre-commit: https://pre-commit.com
# ID: 138fd403232d2ddd5efb44317e38bf03

# start templated
INSTALL_PYTHON=/home/johndoe/.pyenv/versions/py311/bin/python3.11
ARGS=(hook-impl --config=.pre-commit-config.yaml --hook-type=pre-commit)
# end templated

HERE="$(cd "$(dirname "$0")" && pwd)"
ARGS+=(--hook-dir "$HERE" -- "$@")

if [ -x "$INSTALL_PYTHON" ]; then
    exec "$INSTALL_PYTHON" -mpre_commit "${ARGS[@]}"
elif command -v pre-commit > /dev/null; then
    exec pre-commit "${ARGS[@]}"
else
    echo '`pre-commit` not found.  Did you forget to activate your virtualenv?' 1>&2
    exit 1
fi
```

Now after this I'll add the pre-commit YAML as there's a sanity check to ensure it's part of the repository:

```
$ git add .pre-commit-config.yaml
```

This is because the end goal is that anyone who downloads the code can install your `pre-commit` setup themselves and be able to run the appropriate tests. Now looking at the result:

```
$ git commit
[INFO] Installing environment for https://github.com/pre-commit/pre-commit-hooks.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Trim Trailing Whitespace.................................................Failed
- hook id: trailing-whitespace
- exit code: 1
- files were modified by this hook

Fixing README.md

Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check for added large files..............................................Passed
tox-validation...........................................................Failed
```

So as the file I purposely broke hasn't been fixed the `tox-validation` stage has failed. It also found an issue with trailing whitespace in my README.md and even fixed it for me. I'll go ahead and add the updated README and fix the commented out import so things are working again:

```
$ git add README.md
$ vim src/my_pdm_project_cwprogram_test/mymath.py
$ git add src/my_pdm_project_cwprogram_test/mymath.py
$ git commit
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check for added large files..............................................Passed
tox-validation...........................................................Passed
```

Now that everything has passed I'm given the ability to commit. The output is also much more user friendly and mimics what you'd expect from a CI/CD or test suite run.

## Working With Contributed pre-commit Code

In the configuration file you might have noticed:

```yaml
repo: https://github.com/pre-commit/pre-commit-hooks
```

The basic functionality of this works similar to a GitHub repo with GitHub Actions code. You can even see the code for [end_of_file_fixer](https://github.com/pre-commit/pre-commit-hooks/blob/main/pre_commit_hooks/end_of_file_fixer.py) as an example:

```python
def fix_file(file_obj: IO[bytes]) -> int:
    # Test for newline at end of file
    # Empty files will throw IOError here
    try:
        file_obj.seek(-1, os.SEEK_END)
    except OSError:
        return 0
    last_character = file_obj.read(1)
    # last_character will be '' for an empty file
    if last_character not in {b'\n', b'\r'} and last_character != b'':
        # Needs this seek for windows, otherwise IOError
        file_obj.seek(0, os.SEEK_END)
        file_obj.write(b'\n')
        return 1
```

In fact there's actually a fairly sizeable amount of these [listed on the pre-commit website](https://pre-commit.com/hooks.html). In fact another one is [provided by the PDM project](https://pdm-project.org/latest/usage/advanced/#hooks-for-pre-commit) to make sure `pdm.lock` is up to date. I'll go ahead and add this in:

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: local
    hooks:
    -   id: tox check
        name: tox-validation
        entry: pdm run tox
        language: system
        types: [python]
        pass_filenames: false
-   repo: https://github.com/pdm-project/pdm
    rev: 2.10.4 # a PDM release exposing the hook
    hooks:
    -   id: pdm-lock-check
```

Now I'll manually edit one of my dependencies in `pyproject.toml` and check the result:

```ini
dependencies = [
    "numpy>=1.25.1",
    "requests>=2.31.0",
]
```

```
$ git add pyproject.tmol .pre-commit-config.yaml
$ git commit
[INFO] Initializing environment for https://github.com/pdm-project/pdm.
[INFO] Installing environment for https://github.com/pdm-project/pdm.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check for added large files..............................................Passed
tox-validation...........................................................Passed
pdm-lock-check...........................................................Failed
- hook id: pdm-lock-check
- exit code: 1

Lock file hash doesn't match pyproject.toml, packages may be outdated
```

In this case it noticed that my `numpy` dependency has changed. I'll go ahead and revert this and see what happens:

```
$ vim pyproject.toml
$ git add pyproject.toml
$ git commit
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check for added large files..............................................Passed
tox-validation...........................................................Passed
pdm-lock-check...........................................................Passed
```

Now everything is looking good and I'm allowed to commit again. Trying to check if the pdm lock file was synced would have taken a substantial amount of time via a normal git hook, primarily due to not having context on the `pdm` code base. Instead I can use this hook provided by the developers in less than ten minutes to do it instead.

## File Filtering

To showcase this best we'll want to commit what we have now so there's better control over what files get added:

```
$ git commit -m "Initial Commit"
```

Now since `tox` can run specific stages we can use this to break out tasks based on what's actually been committed. Considering when we'd want things to run:

- All files should have the basic large files, trailing whitespace, and end of files check
- `README.md` should have a markdown linter run against it
- `pyproject.toml` should have a toml linter run against it
- Linting should be run if files in `src` or `tests` are modified
- Tests should be run if files in `src` or `tests` are modified
- Documentation building should be run if files in `src` (for automodule generation) or `docs` are modified
- Everything should run if `pyproject.toml` is updated as it controls settings and manages dependencies

So let's see what a solution like this would look like:

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-toml
    -   id: check-added-large-files
-   repo: local
    hooks:
    -   id: tox lint
        name: tox-validation
        entry: pdm run tox -e test,lint
        language: system
        files: ^src\/.+py$|pyproject.toml|^tests\/.+py$
        types_or: [python, toml]
        pass_filenames: false
    -   id: tox docs
        name: tox-docs
        language: system
        entry: pdm run tox -e docs
        types_or: [python, rst, toml]
        files: ^src\/.+py$|pyproject.toml|^docs\/
        pass_filenames: false
-   repo: https://github.com/pdm-project/pdm
    rev: 2.10.4 # a PDM release exposing the hook
    hooks:
    -   id: pdm-lock-check
-   repo: https://github.com/jumanjihouse/pre-commit-hooks
    rev: 3.0.0
    hooks:
    -   id: markdownlint
```

First off we have `check-toml` for linting our `pyproject.toml` file. `markdownlint` is taken from [jumanjihouse/pre-commit-hooks](https://github.com/jumanjihouse/pre-commit-hooks). Note that `check-*` and the markdown linter generally has code to check if appropriate files were added so there's no need to add filters. Another thing to keep in mind is that it's recommended to pin `rev` against a specific revision (usually a tag) versus something like `master` or `main`. This will reduce "unexpected" surprises due to code changes. Here we see the actual filtering at work:

```yaml
    -   id: tox lint
        name: tox-validation
        entry: pdm run tox -e test,lint
        language: system
        files: ^src\/.+py$|pyproject.toml|^tests\/.+py$
        types_or: [python, toml]
```

So `files: ^src\/.+py$|pyproject.toml|^tests\/.+py$` is a regular expression to show what files we're interested in. In this case it's files under `src/` and `tests/` as well as `pyproject.toml`. `types_or` (requires 2.9.0 or later pre-commit) also ensures we're only looking at `python` or `toml` files. If you're wondering what to put in for `types_or` the `identify-cli` tool will let you know appropriate values that can be used:

```
$ identify-cli docs/source/index.rst
["file", "non-executable", "rst", "text"]
```

`file` is the most generic type. You can also be more specific with `rst` for example. `types` can be used if you want something that works off `AND` comparison instead:

```yaml
        types: [python, executable]
```

This will run if a file is a python file **and** executable as well. Now that we've seen how comparisons work, it's time to put this into practice. I'll add our updated pre-commit YAML, modify `README.md`, and then run `git commit`:

```
$ vim README.md #changes here
$ git add .pre-commit-config.yaml README.md
$ git commit
[INFO] Initializing environment for https://github.com/jumanjihouse/pre-commit-hooks.
[INFO] Installing environment for https://github.com/jumanjihouse/pre-commit-hooks.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check Toml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
tox-validation.......................................(no files to check)Skipped
tox-docs.............................................(no files to check)Skipped
pdm-lock-check.......................................(no files to check)Skipped
Check markdown files.....................................................Passed
```

Given that no `toml` or appropriate python files were modified unnecessary checks are skipped according to our setup. The `README.md` file does have `markdownlint` run against it and has passed the checks. I'll go ahead and commit the file with the message "Updated README.md title". Now it's time to see what happens when we make changes to `docs/`:

```
$ vim docs/source/index.rst
$ git add docs/source/index.rst
$ git commit
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check Toml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
tox-validation.......................................(no files to check)Skipped
tox-docs.................................................................Passed
```

In this case `tox-docs` is run but since no python files are present neither linting nor tests were run. Now I'll revert the docs change and modify one of the tests. This should kick off the linting/tests phase but not do anything with the docs building:

```
$ git reset docs/source/index.rst
$ git checkout docs/source/index.rst
$ vim tests/test_mymath.py
$ git add tests/test_mymath.py
$ git commit
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check Toml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
tox-validation...........................................................Passed
tox-docs.............................................(no files to check)Skipped
pdm-lock-check.......................................(no files to check)Skipped
Check markdown files.................................(no files to check)Skipped
```

Finally we'll make an update to `pyproject.toml` which should run everything (I'll also reset the state of the test file so there are no false positives):

```
$ git reset tests/test_mymath.py
$ git checkout tests/test_mymath.py
$ vim pyproject.toml
$ git add .pre-commit-config.yaml pyproject.toml
$ git commit
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check Toml...............................................................Passed
Check for added large files..............................................Passed
tox-validation...........................................................Passed
tox-docs.................................................................Passed
pdm-lock-check...........................................................Passed
Check markdown files.................................(no files to check)Skipped
```

Everything has been run. I will say that technically it wasn't necessary to run this as I only made a description update and nothing was changed in the actual dependencies or tool configuration. That means nothing was changed that would require linting/tests/documentation updates. Even so, trivial `pyproject.toml` changes will generally be very rare after the project is flushed out and you should only really be updating for dependency or tool configuration changes from there.

## Conclusion

`pre-commit` is definitely one of the tools I wish I knew about sooner in my development career. It's a great way to avoid having people frown at your PRs for having 3 different lint fix only commits. Given that it is another roadblock to pushing commits you'll want to make sure your linting passes on the entire project via `pre-commit run -a`. Otherwise your coworkers will most definitely find a way to bypass it.

If you like what you see I'm am also available for hire. Those interested can find out more info in my dev.to profile.