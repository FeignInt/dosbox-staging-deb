# dosbox-staging package as deb
![push to PPA](https://github.com/FeignInt/dosbox-staging-deb/workflows/push%20to%20PPA/badge.svg?branch=debianish)

This github repo will automagically push a debianised source package of [dosbox-staging](https://dosbox-staging.github.io/about) to an [unofficial PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging), where it is built and made available to apt.

Currently the target release is Ubuntu Bionic ( 18.04 )
The reason is simply the fact that the default github workflow runner ( ubuntu-latest ) is 18.04. The dependencies have been relaxed, by stripping the minimum version requirements with the following in _`debian/rules`_

```
override_dh_gencontrol:
        sed -i -E 's/ [(][^,]+//g' debian/*.substvars
        dh_gencontrol
```
  - It is an ugly hack, but seemed to be the easiest way to accommodate the widest range of distributions.

Thus the packages should work with many "Debian Like" distros.

```
sudo apt-add-repository ppa:feignint/dosbox-staging
```

:warning: if you are not on ubuntu bionic!
Ubuntu's apt-add-repository blindly adds uri for your codename regardless of it not existing. I had expected it to query and select best match, it does not.

Manually add,
See the *Technical details about this PPA* on [launchpad PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging)

This would create a .list for the bionic versioned
```
sudo cat > /etc/apt/sources.list.d/dosbox-staging.list <<EOF
deb http://ppa.launchpad.net/feignint/dosbox-staging/ubuntu bionic main
deb-src http://ppa.launchpad.net/feignint/dosbox-staging/ubuntu bionic main
EOF
```
Of course, use which ever editor you prefer

don't forget to _**sudo apt update**_

- If the packages are not working on your distro an option would be to [compile_yourself](#compiling-yourself) which has a much greater chance of working.
---
#### Automagical
Automation is achieved with _`.github/workflows/push_to_ppa.yml`_
1. It is triggered daily with a _`schedule:`_ event trigger
  - When triggered on a schedule the upstream [dosbox-staging](https://github.com/dosbox-staging/dosbox-staging/) tags are queried, filtering out any tags that correspond to tags on this repo. Should a new tag exist the script proceeds with packaging the source and pushing to PPA where it is compiled, packaged and served.
  - Should there be more than one new tag, the oldest is processed, leaving the newer tags for next scheduled run.
2. New tags, Two tag formats are monitored
   1. _`v*`_ is largely redundant, upstream tags should not exist on this repo.
   1. `debianish/*`
     1. tags ending with `-1` are ignored, since they would have been created by a scheduled run.
     1. Tags ending with **`-{2..99}`** is indicative of changes made to `debian/*`. It could be:
         - new compiler flags
         - some patch not yet available upstream
         - simply a rebuild with newer toolchain.
---
#### Architectures
Launchpad's PPA build farms can produce packages for a number of architectures

*Compiled/Packaged with Ubuntu 18.04*
| Architecture                     | Builds             | Runs               | comment       |
|-                                 |:-:                 |:-:                 |-              |
|AMD x86-64 (amd64)                | :heavy_check_mark: | :heavy_check_mark: | bullseye/sid, |
|Intel x86 (i386)                  | :heavy_check_mark: | :grey_question:    |               |
|ARM ARMv8 (arm64)                 | :heavy_check_mark: | :grey_question:    |               |
|ARM ARMv7 Soft Float (armel)      | :no_entry:         |                    | ppa ignored   |
|ARM ARMv7 Hard Float (armhf)      | :heavy_check_mark: | :grey_question:    |               |
|PowerPC (powerpc)                 | :no_entry:         |                    | ppa ignored   |
|PowerPC64 Little-Endian (ppc64el) | :heavy_check_mark: | :grey_question:    |               |

  - I should probably try the i386 myself
---
#### Compiling yourself.
The simplest way, assuming you have added the [launchpad PPA](https://launchpad.net/~feignint/+archive/ubuntu/dosbox-staging),

:warning: the deb-src line must be uncommented
If using Ubuntu's _add-apt-repository_ include **-s**, short for **--enable-source**

```
sudo apt update &&
sudo mk-build-dep -i dosbox-staging &&
apt source --build dosbox-staging

```
- :information_source:
  - Requres the package _`devscripts`_
  - _`mk-build-dep`_ generates a metapackage for the build depends.
  - _`-i`_ installs it.
  - _`sudo`_ is optional, but -i won't work and you will have to get root to install the package anyway.

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

    /var/cache/pbuilder/result/
    /var/cache/pbuilder/<codename><arch>/result/

Consult the manual for details on how to set result dir.

---
### [git-buildpackage](https://honk.sigxcpu.org/piki/projects/git-buildpackage/)

A lot more involved, as the name suggests it revolves around git.

Providing _`--git-pbuilder`_ option is passed it uses pbuilder with cowbuilder as the default backend.
```
sudo apt update &&
sudo apt install git-buildpackage
```

First get the [dosbox-staging](https://github.com/dosbox-staging/dosbox-staging/) source

*In your chosen source dir.*
```
git clone https://github.com/dosbox-staging/dosbox-staging.git &&
cd dosbox-staging &&
git remote add -t debianish \
  -f --tags \
  debianish https://github.com/FeignInt/dosbox-staging-deb.git &&
git checkout debianish
```
```
git-pbuilder create # first run
git-pbuilder update # keep it fresh
```
```
gbp buildpackage -i'(?!debian/)' \
  --git-overlay --git-pbuilder   \
  --git-export-dir=../build-area \
  --git-export="$(git tag -l "debianish*" --sort="-taggerdate" | head -n1 )"
```
- :information_source:
  - `-i'(?!debian/)'` This makes sure that only the debian/ tree is included, without it the build will fail since `.github/**` and this README will interfere.
  - Without `--git-overlay` gbp will only export `debian/*` which it is not going to get very far with.
`--git-export-dir` you could use ram for build-area, note it is relative to `PWD`
  - Regards `--git-export` feel free to use whichever tag, commit, branch you like. Note that it must have a debian/ tree on it.

:+1: Using gbp/pbuilder/cowbuilder other packages installed on your system won't effect the build.

:+1: You can build for i386 ( try compiling wine on amd64/i386 multilib )

:-1: It takes a little longer. This can be mitigated.

**Mitigate increased build time**
```
<pbuilder|cowbuilder|git-pbuilder> login --save-after-login
apt install <build-deps>
exit
```
From now on, it won't spend time installing the dependencies each run, of course the disadvantage with this is the build env. is less clean for other projects, thus negating the initial advantage.

*Hint:*
```
man gbp-buildpackage # look for --git-pbuilder-options
man cowbuilder       # look for --basepath
```

---
### Bleeding edge
If you want to build something other than a _predefined tag_, for instance you want to build the `HEAD` of `dosbox-staging/master` or any other branch.

**Create a tag**
```
git fetch origin # assumes dosbox-staging is on this remote
git tag --force ss0.76.0-a origin/master
# it is just a throw away tag that gbp can grab
```
_The fetch is optional, but you want the latest right?_

**Create a snapshot changelog**
```
gbp dch -S -N 0.76.0~a
```
_just drop the `-N <version>` to increment snapshot, **don't forget to retag!**_
```
gbp buildpackage -i'(?!debian/)'        \
  --git-overlay --git-pbuilder          \
  --git-export-dir=../build-area        \
  --git-export="WC"                     \
  --git-upstream-tag="ss%(version%~%-)s"
```
*WC is Working Copy, it will include everything at your feet, including uncommitted changes.*

---
#### Bonus for reading this far.

If you have a [reprepro](https://wiki.debian.org/DebianRepository/SetupWithReprepro) setup

The postbuild hook will run after a successful build, it exports env. var. `GBP_CHANGES_FILE` containing the full path to the .changes file.

The following script will push the .changes file to [reprepro](https://wiki.debian.org/DebianRepository/SetupWithReprepro)

add _`--git-postbuild="populate_repo.sh"`_ to the _`gbp buildpackage`_ above

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
    "${GBP_CHANGES_FILE}"       #&&
      #update-repo "${MyRepoList}"
      #disabled auto update-repo
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
