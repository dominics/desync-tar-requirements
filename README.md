## desync tar requirement example

This repository has a working example of what is required for `desync tar` to
be able to process a tar file

- An example target we want to archive, in the directories `a/` and `empty/`
- Two new-line separated file lists, suitable for passing to GNU tar's `--files-from` option
  - `broken.list` has two entries, but has missing intermediate directories implied by one of those paths; for those
    directories (`a/`, `a/b/`, `a/b/c/`), there's _no entry_ placed in the resulting tar.
  - `working.list` has two entries as well, but because it targets `a/`, the resulting tar *does* have the intermediate
    directories present
- 

### Conclusion

It's not enough for the tar to have a common parent directory, nor enough to
use `--add-parent`

. It's also necessary
that any intermediate directories appear in the tar before the files contained in
them.

When passing a file list to GNU tar, this means:

- You can pass a list of directories, with `--recursive` (the default)
- If this list of directories has no missing intermediate directories, it'll work 
- But if it has entries with missing intermediate directories, then the resulting tar is _not suitable_ as a target for desync at the moment

Your options are:

- Include the intermediate directories *alone* (without other sibling entries that you didn't want included.)
  - e.g. consider if there was a file at `./a/b/foo` that you _didn't_ want in the resulting tar
    - You could `--exclude` the other contents of intermediate directories, but this gets complicated/unwieldy if the file tree in question is large
    - Otherwise, you could do the recursion yourself, and pass the `--no-recursion` flag; this allows you to control the contents of the tar much more exactly, because you can include _just_ the intermediate directories
- Don't use CLI tar (build the tar archive yourself)? pax? cpio?

## Example output

### working

```
$ ./working
./empty/
./empty/.gitignore
./a/
./a/b/
./a/b/c/
./a/b/c/d/
./a/b/c/d/1
./a/b/c/d/3
Result of desync run was: 0
.caidx produced was: -rw-r--r--  1 user  group   144B 14 Jan 10:31 working.caidx
```

### broken

```
$ ./broken
./empty/
./empty/.gitignore
./a/b/c/d/
./a/b/c/d/1
./a/b/c/d/3
Result of desync run was: 141
.caidx produced was: -rw-r--r--  1 user  group   144B 14 Jan 10:31 broken.caidx
```
