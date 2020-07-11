# dosbox-staging package as deb

![push to PPA](https://github.com/FeignInt/dosbox-staging-deb/workflows/push%20to%20PPA/badge.svg?branch=debianish)

This github repo will automagically push a debianised source package of
[dosbox-staging](https://dosbox-staging.github.io/about)
to an
[unofficial PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging),
 where it is built and made available to apt.

---

## Series

:information_source: no apt repo for pure Debian yet, for the moment use the
 best match as per table below or [compile your own](#compiling-yourself).

| Codename | Release | Builds             | Debian base  | Debian Version |
|:-        |:-:      |:-:                 |-:            |:-:             |
| Focal    | 20.04   | :heavy_check_mark: | Bullseye/sid | 11             |
| Eoan     | 19.10   | :heavy_check_mark: | Buster/sid   | 10             |
| Bionic   | 18.04   | :heavy_check_mark: | Buster/sid   | 10             |
| Xenial   | 16.04   | :heavy_check_mark: | Stretch/sid  | 9              |
| Trusty   | 14.04   | :heavy_check_mark: | Jessie/sid   | 8              |

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

The script currently packages for *Focal, Eoan, Bionic, Xenial and Trusty.*
 This list is derived from Launchpad's API listing of supported builders,
 excluding Precise as it lacks `libopusfile`.

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

:information_source: ppa:feignint/**bionic**/dosbox-staging forces *bionic*

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

#### Create a tag

```
git fetch origin # assumes dosbox-staging is on this remote
git tag --force ss0.76.0-a origin/master
# it is just a throw away tag that gbp can grab
```

:information_source: *The fetch is optional, but you want the latest right?*

#### Create a snapshot changelog

```
gbp dch -S -N 0.76.0~a

```

:information_source:  just drop the `-N <version>` to increment snapshot,
 **don't forget to retag!**

```
gbp buildpackage -i'(?!debian/)'        \
  --git-overlay --git-pbuilder          \
  --git-export-dir="../build-area"      \
  --git-export="WC"                     \
  --git-upstream-tag="ss%(version%~%-)s"
```

:information_source:

- **WC** is Working Copy, it will include everything at your
 feet, including uncommitted changes.

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
