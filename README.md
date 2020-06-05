## Description

`git_xy` helps to synchronize sub directories between git repositories
semi-automatically, and may generate pull requests on `github` if
changes are detected on the destination repository.

`git_xy` reads a list of source/destination specifications from a
configuration file, and for each of them, `git_xy` fetches changes
from the source repository and synchronizes them to destination path
(thanks to `rsync`). It finally generates commit and creates new PR
(pull request) if necessary.

## TOC

* [Description](#description)
* [Usage](#usage)
  * [Installation](#installation)
  * [Configuration](#configuration)
  * [Invocation](#invocation)
  * [Sample Prs on Github](#saple-prs-on-github)
  * [How it works](#how-it-works)
* [TODO](#todo)
* [Why](#why)
* [Author. License](#author-license)

## Usage

**WARNING:** The project is still in `\alpha` stage.

### Installation

`git_xy` is a Bash4 script. It requires some additional tools on system:

* GNU tools: `awk`, `rsync`, `bash`, `git`, `grep`, `sed`
* Github command tool for PR creation: https://github.com/cli/cli/releases

The main program `git_xy` can be installed anywhere on your search path

```
$ sudo wget -O /usr/local/bin/git_xy \
    https://github.com/icy/git_xy/raw/ng/git_xy.sh

$ sudo chmod 755 /usr/local/bin/git_xy
```

### Configuration

Configuration consists of source/destination specification in the following
format:

```
source_reposity branch path/   dst_repository branch path/ [pr_base_repo]
```

The option `pr_base_repo` is optional and is used to specify where
you want the PR arrives. By default, it's the upstream repository.

See examples in [git_xy.config-sample.txt](git_xy.config-sample.txt).

```
git@github.com:icy/pacapt ng lib/ git@github.com:icy/pacapt master lib/
```

### Invocation

Now execute the script

```
GIT_XY_CONFIG="git_xy.config-sample.txt" ./git_xy.sh
```

the script will fetch changes in `lib` directory from branch `ng`
in the `pacapt` repository,
and update the same repository on another branch `master`.
If changes are detected, a new branch will be created and/or
some pull request will be generated.

### Sample Prs on Github

* https://github.com/icyfork/pacapt/pull/1
* https://github.com/icy/pacapt/pull/140
* https://github.com/icy/pacapt/pull/139

### How it works

Nothing magic, it's a wrapper of `git clone, rsync and git commit`:)
Let's say we have configuration file

```
src_repo src_branch src_path dst_repo dst_branch dst_path
```

the script will do as below

* Create a clone of the `src_repo` in `~/.local/share/git_xy/src_repo`
  (The actual folder name is a bit different to avoid some special characters
  in the user input.)
* Check out the existing branch `src_branch`
* Create a clone of the `dest_repo` in `~/.local/share/git_xy/dst_repo`
* Check out the existing branch `dst_branch`
* Create new branch from `dst_branch` (if neccessary).
  This branch is specially used for PR creation.
  The name of the new branch is derived from `git_xy__${src_branch}/${src_path}__${dst_branch}/${dst_path}`
* Use `rsync` to synchronize the contents of the `src_path` and `dst_path`.
  On the local machine where the script runs, it's a variant of the command
  `rsync -ra --delete SRC/ DST/` here
  `SRC` is `~/.local/share/git_xy/src_repo/src_path/` and
  `DST` is `~/.local/share/git_xy/dst_repo/dst_path/`
* Generate new commit and/or use the external tool `gh` to generate PR

Well, it's so easy right? It's an automation support of your handy commands.

## TODO

- [ ] Create a hook script to create pull requests
- [ ] Add tests and automation support for the project
- [ ] Provide a link to the original source
- [ ] More `POSIX` ;)
- [ ] Better error reporting

Done

- [x] Re-use existing `git_xy` branch
- [x] Better hook to handle where PRs will be created
- [x] Add some information from the last commit of the source repository
- [x] Make sure the top/root directory is not used (we allow that)
- [x] Allow a repository to update itself

## Why

There are many tools trying to solve the code-sharing problem:

* `git submodule`
* `git subtree`
* https://github.com/ingydotnet/git-subrepo
* https://github.com/twosigma/git-meta
* https://github.com/mateodelnorte/meta
* https://github.com/splitsh/lite
* https://github.com/unravelin/tomono
* https://sourceforge.net/projects/gitslave/
* https://github.com/teambit/bit (bit only)
* https://github.com/lerna/lerna (javascript only)
* https://gerrit.googlesource.com/git-repo/ (Android only?)
* https://github.com/microsoft/VFSForGit (sic, Windows only)

Well, there are too many tools...
What I really need is a simple way to pull changes from some repository
to another repository, generates some pull request for reviewing,
and the downstream maintainer will decide what they would do next.

Morever, this process should be done automatically when the upstream
repository is updated. Human intervention is not the right way when
there are just 100 or 500 repositories because of the raise of the
micro-repository `design` (if any) :D

## Author. License

The script is writtedn by Ky-Anh Huynh.
The work is released under a MIT license.
