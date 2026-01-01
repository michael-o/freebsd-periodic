# Scripts for the FreeBSD `periodic(8)` system

This repository contains scripts for the FreeBSD `periodic(8)` system that
I have used privately on my FreeBSD systems for years.

## Installation

Install either via the `sysutils/michaelo-periodic` port/package or by manually
copying the scripts into `%%LOCALBASE%%/etc/periodic/`.

## Scripts

The following scripts are included:

### `weekly/500.gitup`

Uses `gitup(1)` to update Git repository clones by reading sections from
`gitup.conf`, e.g., FreeBSD `src` or `ports`.

The following variables can be configured in `periodic.conf`:
* `weekly_gitup_enable`: (bool) Configures whether this script is enabled.
  The default is `"NO"`.
* `weekly_gitup_sections`: (list) Provides a space-separated list of gitup sections.
* `weekly_gitup_path`: (str) Path to the `gitup` executable; it must be installed
  before use. The default is `"%%LOCALBASE%%/bin/gitup"`.

### `weekly/550.make-src`

Builds FreeBSD from source in `/usr/src` with supplied targets and number of jobs.
Targets will be built separately and output will be redirected to
`/var/log/weekly-make-src-{target}.log`.

The following variables can be configured in `periodic.conf`:
* `weekly_make_src_enable`: (bool) Configures whether this script is enabled.
  The default is `"NO"`.
* `weekly_make_src_targets`: (list) Provides a space-separated list of `make` targets.
* `weekly_make_src_jobs`: (num) Number of jobs `make` will use to build the targets.
  The default is `$(( ($(sysctl -n kern.smp.cpus) + 1) / 2 ))`.

## Sample Usages

The following sample usages can be applied:

### `weekly/500.gitup` + `weekly/550.make-src`

Track your stable or current branch of the FreeBSD source tree in `gitup.conf`
and set the following in your `periodic.conf`:
```
weekly_gitup_enable="YES"
weekly_gitup_sections="src"
weekly_make_src_enable="YES"
weekly_make_src_targets="buildworld buildkernel"
```
It will build world and kernel in the background weekly. When you are ready
to update your system perform according to the [handbook](https://docs.freebsd.org/en/books/handbook/cutting-edge/#makeworld):
```
# cd /usr/src
# make -j4 installkernel
# shutdown -r now
# etcupdate -p
# cd /usr/src
# make -j4 installworld
# etcupdate -B
# shutdown -r now
```
This should take only minutes instead of hours, because the weekly build has
already completed the expensive compilation steps.

