# dosbox-staging package as deb

![push to PPA](https://github.com/FeignInt/dosbox-staging-deb/workflows/push%20to%20PPA/badge.svg)
![Weekly PPA](https://github.com/FeignInt/dosbox-staging-deb/workflows/Weekly%20PPA/badge.svg)

This github repo will automagically push debianised source packages of
[dosbox-staging](https://dosbox-staging.github.io/about)
to Launchpad's PPA, which in turn compiles and packages the final deb package
 and makes them available to apt update.

Currently two PPA repositories

| Stable Release Series | Weekly snapshots |
|:-:                    |:-:               |
| [ppa:feignint/dosbox-staging](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging) | [ppa:feignint/dosbox-staging-weekly](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging-weekly) |

:information_source: *Weekly snapshots* will have priority over *Stable Release*

The snapshots represent recent Work in Progress code,
 including features and bug fixes not yet released.
[the dosbox-staging projects page](https://github.com/dosbox-staging/dosbox-staging/projects)
 should give you some idea of what to expect.

Plese note that weekly snapshots may contain undocumented features ( aka bugs )
check both the
[open](https://github.com/dosbox-staging/dosbox-staging/issues?q=is%3Aopen+is%3Aissue)
and
[closed](https://github.com/dosbox-staging/dosbox-staging/issues?q=is%3Aissue+is%3Aclosed)
issues at
[dosbox-staging issues](https://github.com/dosbox-staging/dosbox-staging/issues)
If your bug is not listed, see if you can reproduce it using the most recent
  *Stable Release*

It is possible that the bug you are experiencing is specific to the deb
 packaging, especially if you are unable to install. Feel free to report that
 [here](https://github.com/FeignInt/dosbox-staging-deb/issues)

---

## Series

:information_source: no apt repo for pure Debian yet, for the moment use the
 best match as per table below or [compile your own](#compiling-yourself).

| Ubuntu       | Debian base     | Stable Release     | Weekly             |Notes           |
|:-            | :-              | :-:                | :-:                |:-              |
| 20.10 Groovy | 11 Bullseye/sid | :heavy_check_mark: | :heavy_check_mark: | <sup>[2]</sup> |
| 20.04 Focal  | 11 Bullseye/sid | :heavy_check_mark: | :heavy_check_mark: | <sup>[2]</sup> |
| 19.10 Eoan   | 10 Buster/sid   | :heavy_check_mark: |                    |                |
| 18.04 Bionic | 10 Buster/sid   | :heavy_check_mark: |                    |                |
| ~~16.04 Xenial~~ | ~~9  Stretch/sid~~  | :heavy_check_mark: | | <sup>[1],[3]</sup> |
| ~~14.04 Trusty~~ | ~~8  Jessie/sid~~   | :heavy_check_mark: | | <sup>[1],[3]</sup> |

<sup>[1] - no link-time optimizer ( -flto )</sup>\
<sup>[2] - [future fluidsynth support](https://github.com/dosbox-staging/dosbox-staging/issues/262#)</sup>\
<sup>[3] - 2020-07-30 - script will no longer push to PPA</sup>

## Architectures

Launchpad's PPA build farms can produce packages for a number of architectures

| Architecture                     | Builds             | Runs               |
|-                                 |:-:                 |:-:                 |
|AMD x86-64 (amd64)                | :heavy_check_mark: | :heavy_check_mark: |
|Intel x86 (i386)                  | :heavy_check_mark: | :grey_question:    |
|ARM ARMv8 (arm64)                 | :heavy_check_mark: | :grey_question:    |
|ARM ARMv7 Hard Float (armhf)      | :heavy_check_mark: | :grey_question:    |
|PowerPC64 Little-Endian (ppc64el) | :heavy_check_mark: | :grey_question:    |
|IBM System z (s390x)              | :heavy_check_mark: | :grey_question:    |

---
## Automagical
Automation is achieved with [.github/workflows/push_to_ppa.yml](https://github.com/FeignInt/dosbox-staging-deb/blob/debianish/.github/workflows/push_to_ppa.yml)

The script currently packages for
 *Groovy, Focal, Eoan, Bionic, ~~Xenial and Trusty~~<sup>[3]</sup>.*
 This list is derived from Launchpad's API listing of supported builders,
 excluding Precise as it lacks *libopusfile*.

- It is triggered daily with a *schedule:* event trigger
  - When triggered on a schedule the upstream
 [dosbox-staging](https://github.com/dosbox-staging/dosbox-staging/) tags are
 queried, filtering out any tags that correspond to tags on this repo. Should
 a new tag exist the script proceeds with packaging the source and pushing to
 PPA where it is compiled, packaged and served.
  - Should there be more than one new tag, the oldest is processed, leaving the
 newer tags for next scheduled run.
- New tags, Two tag formats are monitored
  1. `v*` is largely redundant, upstream tags should not exist on this repo.
  1. `debianish/*`, Tags ending with `-1` are ignored, since they would have
 been created by a scheduled run. Tags ending with **-{2..99}** is indicative
 of changes made to packaging ( not upstream code ) It could be:
    - new compiler flags
    - some patch not yet available upstream
    - simply a rebuild with newer toolchain.

---
## Adding the dosbox-staging repo to Ubuntu and derivatives.

If you don't have `apt-add-repository`, for instance Debian,
 see [Manual adding of dosbox-staging repo](#Manual-adding-of-dosbox-staging-repo-to-debian)

:warning: Ubuntu's apt-add-repository blindly adds uri for your series
 regardless of it not existing. I had expected it to query and select best
 match, it does not.

```
sudo apt-add-repository -s ppa:feignint/dosbox-staging
```

:information_source: **-s** is optional, it allows downloading source packages.

## Manual adding of dosbox-staging repo to Debian

See the **Technical details about this PPA** on
 [launchpad PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging)

:warning: If you are on a Debian release and apt refuses to install due to
 missing dependencies... [Don't break Debian](https://wiki.debian.org/DontBreakDebian)
 by manually installing things. Instead, see how far you get with
  [compiling your own](#compiling-yourself).
 I plan to put up Debian specific repos at some point.

You can find your distro's codename with **lsb_release -a**

This would create a .list for the Eoan series, which would best match a current
 Buster release.

```
sudo cat > /etc/apt/sources.list.d/dosbox-staging.list <<EOF
deb http://ppa.launchpad.net/feignint/dosbox-staging/ubuntu eoan main
deb-src http://ppa.launchpad.net/feignint/dosbox-staging/ubuntu eoan main
EOF
```

Of course, use whichever editor you prefer.

:information_source: The deb-src line is optional.

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 \
                 --recv-keys 80FFDEB381C2ECCD9B6CC21837E03C78EA70A526
```

The `<key-id>` can be found on the
 [launchpad PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging)
 page.

Don't forget to `sudo apt update`

:warning: if update complains about signing key, re-do *apt-key* and update again.

---
:information_source:
 I have recently discovered that Debian does have *apt-add-repository*
 available in the package **software-properties-common**
 however it does not really help that much:

- :+1: automatically gets signing-key
- :-1: appears to default to latest bleeding edge series, groovy at time of writing

  - Since I have added "Active development" series,
 if you are on sid this is actually a :+1:

 Personally I don't think is worth messing with.

---

If the packages are not working on your distro an option would be to
[compile yourself](#compiling-yourself) which has a much greater chance of working.

---
## Compiling yourself.
The simplest way, assuming you have added the [launchpad PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging),

:warning: the deb-src line must be uncommented
If using Ubuntu's *add-apt-repository* include **-s**

```
sudo apt update &&
sudo mk-build-dep -i dosbox-staging &&
apt source --build dosbox-staging
```

This makes it much easier to clean up later. e.g.

```
sudo apt purge dosbox-staging-build-deps &&
sudo apt autopurge

```

---
### [cowbuilder](https://wiki.debian.org/cowbuilder)
**Note:** This is just a brief introduction, the above cowbuilder link is more informative.

```
sudo cowbuilder create # first run
sudo cowbuilder update # keep it fresh
```

```
apt source -d dosbox-staging &&
sudo cowbuilder --build dosbox-staging_0.75.0-1.dsc
```

You will find the packages in:

```
/var/cache/pbuilder/result/
/var/cache/pbuilder/<codename>-<arch>/result/
```

Consult the manual for details on how to set result dir.

---
### [git-buildpackage](https://honk.sigxcpu.org/piki/projects/git-buildpackage/)

A lot more involved, as the name suggests it revolves around git.

Providing **--git-pbuilder** option is passed it uses pbuilder with cowbuilder
as the default backend.

```
sudo apt update &&
sudo apt install git-buildpackage
```

### [dosbox-staging](https://github.com/dosbox-staging/dosbox-staging/) source

*In your chosen source dir.*

```
git clone https://github.com/dosbox-staging/dosbox-staging.git &&
cd dosbox-staging &&
git remote add -t debianish -f --tags \
  debianish https://github.com/FeignInt/dosbox-staging-deb.git &&
git checkout debianish
```

```
git-pbuilder create # first run
git-pbuilder update # keep it fresh
```

:information_source: You may endup with an error at the very end, complaining
 about not being able to find the signing key. This is not fatal but can be
 avoided by passing **-uc** ( unsigned changes )

```
gbp buildpackage -i'(?!debian/)' \
  --git-overlay --git-pbuilder   \
  --git-export-dir=../build-area \
  --git-export="$(git tag -l "debianish*" --sort="-taggerdate" | head -n1 )"
```

:information_source:

- **-i'(?!debian/)'** This makes sure that only the debian/ tree is included,
 without it the build will fail since `.github/**` and this README.md
 will interfere.

- Without **--git-overlay** gbp will only export **debian/** which it is not going
 to get very far with.

- **--git-export-dir** you could use ram for build-area, note it is relative to
  **PWD**

- Regards **--git-export** feel free to use whichever tag, commit, branch you
 like. Note that it must have a **debian/** tree on it.

:+1: Using gbp/pbuilder/cowbuilder other packages installed on your system
 won't effect the build.

:+1: You can build for i386 ( try compiling wine on amd64/i386 multilib )

:-1: It takes a little longer. This can be mitigated.

#### Mitigate increased build time

```
<pbuilder|cowbuilder|git-pbuilder> login --save-after-login
apt install <build-deps>
exit
```

From now on, it won't spend time installing the dependencies each run, of
 course the disadvantage with this is the build env. is less clean for other
 projects, thus negating the initial advantage.

*Hint:*

```
man gbp-buildpackage # look for --git-pbuilder-options
man cowbuilder       # look for --basepath
```

---
### Bleeding edge

If you want to build something other than a *predefined tag*, for instance you
 want to build the `HEAD` of `dosbox-staging/master` or any other branch.

```
git fetch origin # /!\ assumes dosbox-staging is on *origin* remote
gbp dch -N $( git describe origin/master | sed 's/-/\~/g;s/^v//' )-1 -- debian/
git diff
```

```
gbp buildpackage -i'(?!debian/)'        \
  --git-overlay --git-pbuilder          \
  --git-export-dir="../build-area"      \
  --git-export="WC"
```

:information_source:

- **WC** is Working Copy, it will include everything at your
 feet, including uncommitted changes.
- The debianised version string e.g. **`0.76.0~alpha~327~g93448542-1`** will
 result in *gbp* packaging commit
 [v0.76.0-alpha-327-g93448542](https://github.com/dosbox-staging/dosbox-staging/commit/934485421ff4558c269606e4007a4e712adfc6ca)
- Replace **origin/master** with any branch or commit you like.

---
## Bonus for reading this far.

If you have a
 [reprepro](https://wiki.debian.org/DebianRepository/SetupWithReprepro) setup
 add
  *--git-postbuild="/path/to/populate_repo.sh"*
 to the *gbp buildpackage* above

The postbuild hook will run after a successful build, it exports env. var.
 `GBP_CHANGES_FILE` containing the full path to the .changes file.

The following script will push the .changes file to `/src/reprepro`
 modify to point at your own reprepro.

:information_source: passing `--update` is optional

:+1: only the my_reprepro.list is updated, so quicker.

:-1: requires root, so would get in the way of scripts.

```bash
#!/bin/bash

base_dir="/srv/reprepro"
MyRepoList="my_reprepro.list"

update-repo ()
{
  for slist in "$@"
  do
    sudo apt-get update \
        -o Dir::Etc::sourcelist="sources.list.d/${slist}" \
        -o Dir::Etc::sourceparts="-"                      \
        -o APT::Get::List-Cleanup="0"
  done
}
_reprepro()
{
  reprepro -b "${base_dir%/}"    \
    --ignore=wrongdistribution   \
    include unstable             \
    "${GBP_CHANGES_FILE}"
}

[[ ! -e "/etc/apt/sources.list.d/${MyRepoList}" ]] &&
  printf "/etc/apt/sources.list.d/%s does not exist\n" \
         "${MyRepoList}" 1>&2  && exit 2

[[ ! ${GBP_CHANGES_FILE:=$1} =~ .+[.]changes$ ]] &&
  printf "no changes file\n" 1>&2                && exit 3

_reprepro || exit 3
# quickly added a cheap conditional check for update
if [[ $* =~ --update ]]
then
  update-repo "${MyRepoList}" || exit 4
fi
```
