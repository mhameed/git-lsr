# git-lsr: git Levenshtein ratio #

Calculates the Levenshtein similarity ratio between commits.
A measurement which can be used to identify complex/large commits, which should be checked to ensure commits are concise and only address one issue at a time.

doing ```git diff --numstat``` is one such measure, but it can easely be thrown off by something simple as reindenting code.


# Demo #


    [/tmp]$: mkdir lsrTest
    [/tmp]$: cd $!
    [/tmp/lsrTest]$: git init
    Initialized empty Git repository in /tmp/lsrTest/.git/
    [/tmp/lsrTest(master)]$: echo "readme" >readme
    [/tmp/lsrTest(master)]$: git add readme 
    [/tmp/lsrTest(master)]$: git commit -m "Added initial readme."
    [master (root-commit) 817c4bf] Added initial readme.
     1 file changed, 1 insertion(+)
     create mode 100644 readme

    [/tmp/lsrTest(master)]$: # similarity of a commit against itself is clearly 100%
    [/tmp/lsrTest(master)]$: git lsr HEAD HEAD
    817c4bf 817c4bf = 100.00%
    [/tmp/lsrTest(master)]$: echo "a second line" >>readme 
    [/tmp/lsrTest(master)]$: git commit -am "Updated readme."
    [master e19d40a] Updated readme.
     1 file changed, 1 insertion(+)
    [/tmp/lsrTest(master)]$: git lsr HEAD^ HEAD
    817c4bf e19d40a = 72.14%
    [/tmp/lsrTest(master)]$: # the ratio is symmetric:
    [/tmp/lsrTest(master)]$: git lsr HEAD HEAD^
    e19d40a 817c4bf = 72.14%
    [/tmp/lsrTest(master)]$: echo "a third line." >>readme 
    [/tmp/lsrTest(master)]$: git commit -am "Yet another update to readme."
    [master 31a6435] Yet another update to readme.
     1 file changed, 1 insertion(+)
    [/tmp/lsrTest(master)]$: git lsr HEAD HEAD^
    31a6435 e19d40a = 80.68%
    [/tmp/lsrTest(master)]$: git lsr HEAD HEAD^^
    31a6435 817c4bf = 70.39%
    [/tmp/lsrTest(master)]$: echo "tempfile" >tempfile
    [/tmp/lsrTest(master)]$: git add tempfile 
    [/tmp/lsrTest(master)]$: git commit -am "Added a tempfile."
    [master b9adbb2] Added a tempfile.
     1 file changed, 1 insertion(+)
     create mode 100644 tempfile

    [/tmp/lsrTest(master)]$: git lsr HEAD^ HEAD # we didn't touch the same files, so expect no simularity.
    31a6435 b9adbb2 = 0%
    [/tmp/lsrTest(master)]$: echo "a second line to the temp file." >>tempfile 
    [/tmp/lsrTest(master)]$: echo "one last line to the readme." >>readme 
    [/tmp/lsrTest(master)]$: git commit -am "Updated both tempfile and readme."
    [master 927e488] Updated both tempfile and readme.
     2 files changed, 2 insertions(+)
    [/tmp/lsrTest(master)]$: git lsr HEAD^ HEAD
    b9adbb2 927e488 = 34.92%
    [/tmp/lsrTest(master)]$: git rm tempfile 
    rm 'tempfile'
    [/tmp/lsrTest(master)]$: git commit -am "Removed tempfile."
    [master ab00b79] Removed tempfile.
     1 file changed, 2 deletions(-)
     delete mode 100644 tempfile
    [/tmp/lsrTest(master)]$: git lsr HEAD^ HEAD
    927e488 ab00b79 = 75.5%
    [/tmp/lsrTest(master)]$: git lsr HEAD^^ HEAD
    b9adbb2 ab00b79 = 86.22%


Looking at history, which commits are likely to be less intuative and
we need to ensure that they are clean and have good/clear commit messages, or alternatively broken up to smaller commits:

    [/tmp/lsrTest(master)]$: git rev-list HEAD | xargs git lsr
    ab00b79 927e488 = 75.5%
    927e488 b9adbb2 = 34.92%
    b9adbb2 31a6435 = 0%
    31a6435 e19d40a = 80.68%
    e19d40a 817c4bf = 72.14%

We should have a look at what was introduced by b9adbb2 and 927e488.
The point is not to eliminate the low numbers, but to make sure we document/use good commit messages when the repository is changing significantly.

The toy repo above does not do this justice, so try it out on a larger repository.


# Dependencies #

- calc: ```sudo apt-get install apcalc```
- python-levenshtein: ```sudo apt-get install python-levenshtein```


# Install #

- git clone this repo.
- create symlinks in ```~/bin/`` for lsr and git-lsr.


