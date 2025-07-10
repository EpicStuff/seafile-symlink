# seafile-symlink

> Utility to help sync and restore symlinks in Seafile.

As of July ~~2018~~ 2025 ~~(v.6.1.8)~~, Seafile still cannot sync symlinks natively. It
follows them, and syncs them as if they were normal files and directories or just ignores them entirely.
In most cases, this is undesirable.

~~These~~ This scripts ~~are~~ is meant to address this shortcoming. ~~They~~ It will find symlinks,
add them to `seafile-ignore.txt`, store their data in syncable placeholder
files or in a text database (configurable), and restore them therefrom.

~~There are two versions: PowerShell and bash, for Windows and Linux/macOS~~
~~respectively. The first is refactored and feature-complete, but the latter~~
~~may need more work. PRs are welcome.~~ This should work on Linux, has not been tested on Windows.

Lastly, Seafile has a long-standing (~~5~~ 11+ years) issue about symlinks:

https://github.com/haiwen/seafile/issues/288

Keep an eye on it for developments. It takes a lot more work to develop a
backwards-compatible feature for a massive userbase than to whip up some
~~shell~~nim script~~s~~ over the weekend. But if you're interested in seeing this
solved properly, please head on over there, upvote the issue, and
contribute to the conversation.

In the meantime, I hope ~~these~~ this script~~s~~ help your workflow.



# Usage

There are two ~~scripts~~programs included ~~with this repository~~ in the release ~~: PowerShell and bash.~~

~~[seafile-symlink.ps1](seafile-symlink.ps1) was developed on Windows 7 SP1~~
~~with PowerShell 5.0.~~

~~[seafile-symlink.sh](seafile-symlink.sh) was developed on macOS 10.12.6~~
~~with bash 3.2.57.~~

Please do not attempt to run the ~~`.ps1`~~ `exe` on Linux, nor the ~~`.sh`~~ other on Windows.
That sort of use is completely untested and runs the risk of unintended
overwrites. Cross-platform path resolution is tricky.

The tool is designed to be installed either in a (sub-)subdirectory of
a Seafile library, or somewhere else on your computer in case you'd like
to use a single installation to control multiple libraries.

You can ~~install~~ download the tool ~~via git or by downloading this repository:~~ by going to the releases.

~~```bash~~
~~git clone https://github.com/IllyaMoskvin/seafile-symlink~~
~~```~~



## Configuration

Make a copy of `presets/default.taml`. For the purposes of this guide,
we'll assume it's named `presets/custom.taml`.

Inside, you'll find the following settings:

```yaml
library_path: ..\             # Absolute, or relative to seafile-symlink (library folder to the binary)
storage_method: placeholder   # Either 'database' or 'placeholder'
placeholder_ext: seaflnk      # Only applies if using placeholders
db_file: seafile-symlink.txt  # If using 'database', what the file should be named
```

By default, `library_path` assumes that the `seafile-symlink` directory is
a first-level subdirectory of a library.

The `storage_method` indicates whether to use `placeholder` files or a
single `database` text file to track symlinks.

Placeholders offer potentially better preservation of relative symlinks
when you reorganize directories. The database offers centralized storage
for symlink data, similar to `seafile-ignore.txt`, whose conventions it
mimics, but it is less resistant to reorganization.

Personally, ~~I~~ the original author uses `database` for ~~my~~ their projects, but the default is `placeholder`.

You can switch between `placeholder` and `database` at any time. The next
time you run the script after modifying this setting, it'll migrate your
placeholder data into the database and remove them, or vice versa.

The `placeholder_ext` and `db_file` is provided as a matter of convenience.


## Example

```bash
./seafile-symlink custom                   # loads presets/custom.ini
./seafile-symlink custom.taml              # ditto
./seafile-symlink ~/foobar/custom.taml     # absolute path is still ok
./seafile-symlink custom\ 1 custom\ 2      # multiple at once
```


## Workflow

~~When creating symlinks, you'll need to pause syncing.~~

1. ~~Disable auto-sync for the Seafile library~~ Make sure to enable the Don't sync symbolic links option in seafile
2. ~~Create whatever symlinks you need~~
3. ~~Run `seafile-symlink`~~
4. ~~Re-enable auto-sync~~

Removing symlinks takes an extra step now. If you don't remove references to
it, it will be re-created the next time you run `seafile-symlink`. Checklist:

1. Remove the symlink
2. Remove the corresponding placeholder, if any
3. Remove its reference in `seafile-symlink.txt`, if any

When syncing symlinks across devices:

1. Wait for the database or placeholder files to sync
2. Run `seafile-symlink`

The scripts should update `seafile-ignore.txt` before creating new symlinks,
but you may want to disable syncing whenever running them just in case.

~~The PowerShell version is pretty good about avoiding unnecessary file writes,~~
~~but the bash version uses append liberally. This may trigger unnecessary but~~
~~harmless syncs in Seafile.~~

Lastly, if Seafile follows your symlinks and syncs them:

1. Enable auto-sync
2. Manually remove symlinks
3. Allow Seafile time to sync the changes
4. Disable auto-sync
5. Follow the steps for creating symlinks above

As mentioned, Seafile follows symlinks and sees them as actual files and
directories. This tool adds symlinks to `seafile-ignore.txt` to prevent
them from getting synced in the first place.

However, when an item is added to `seafile-ignore.txt`, all that tells
Seafile is that it should ignore any _subsequent_ changes made to it.
Deleting an ignored file will not propogate the changes to the server.

Thus, you must delete any synced symlinks before running this tool, and
allow time for Seafile to sync the deletion.



# Limitations

Only symbolic links are supported, not hard links or directory junctions.
This is intentional.

* [What is the difference between a symbolic link and a hard link?](https://stackoverflow.com/questions/185899/what-is-the-difference-between-a-symbolic-link-and-a-hard-link)
* [What is the difference between NTFS hard links and directory junctions?](https://superuser.com/questions/67870/what-is-the-difference-between-ntfs-hard-links-and-directory-junctions)

~~This tool has only been tested on NTFS and FAT file systems.~~ This tool has basically not been tested.

This tool is meant primarily for syncing symlinks with relative targets
within the same Seafile library. Support of symlinks with absolute paths
or with relative paths pointing outside the Seafile library is limited.

It's on my radar to add a config option to treat all symlinks below the
library root but on the same drive as relative. Currently, they are
converted to absolute.

Symlinks pointing to network locations are unsupported.

Symlinks pointing to other drives are untested.

Symlinks that contain the `~` home path in their target are untested.

Absolute paths might get sketchy when transferred cross-platform.



# Compiling

To compile this project yourself (first why?, second) go checkout/use https://github.com/EpicStuff/zxc


# License

Released under the [MIT](License.txt) license. Feel free to use it however
you'd like, but I cannot guarantee that it won't break on you or corrupt your
data. Playing with symlinks is dangerous business, if the wrong paths get
overwritten.
