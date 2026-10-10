# Debian 13 CIS

## October 2026 - QA fixes

- 5.4.1.5 root checked only when deb13cis_inactivelock_include_root is true
- 6.1.1.1.x journald tests run whatever the syslog choice, 6.1.1.1.4 journald only
- goss files start with --- on line 1
- 7.1.11 to 7.1.13 read the mount point from $6
- 7.1.13 SUID/SGID review checks package ownership and integrity
- 1.3.1.3 rewritten for dash, complain and unconfined counts
- 5.3.3.4.1 and 5.3.3.4.2 list the common-* files explicitly
- 7.2.9 uid range read with awk
- 2.1.2 package avahi-daemon, 2.1.16 package telnetd
- 2.1.22 postfix check only when postfix is installed
- 6.2.3.19 asserts the chown syscalls
- 6.2.4.6 and 6.2.4.7 use find ! -user / ! -group root
- 5.4.1.2 lists only passwords with min days below 1
- 5.4.1.5 uses the benchmark inactive check
- 1.2.1.6 and 1.2.1.9 use find with owner and mode
- 5.4.2.1 to 5.4.2.3 use awk instead of character classes
- 7.2.5 to 7.2.8 sort before uniq -d
- standalone.yml globs aligned with goss.yml
- config file assertions anchored so comments cannot satisfy them
- 6.1.2.11 CA file variable read from .Vars, rsyslog audits no longer abort
- 6.1.2.9 invalid exec attribute removed from the package test
- 6.1.2.3 reads the effective journald ForwardToSyslog value
- 6.1.2.6 gated on deb13cis_remote_log_server, 6.1.2.7 split client and server
- 6.3.1 gated on deb13cis_config_aide
- level gates corrected for 3.3.2.1, 3.3.2.2, 5.4.1.2 and 6.3.3
- deb13cis_exclude_unowned_search_path spacing fixed
- Company name updated to MindPoint Group - A Quantum Sky Company
- run_audit.sh uses syver v0.13.0 by default
- README describes syver

## Based on CIS v1.0.0 - Branch sept26_updates

- 1.7.2 defined the goss key gdm_profile_banner twice in one command: map, so the
  banner-message-enable assertion was lost and goss aborts on a duplicate key
- 1.7.10 and 1.7.11 both keyed on /etc/gdm3/custom.conf; level_1 and level_2 both default
  true, so goss merged them and 1.7.10 was silently dropped. Both now use named keys
- 1.6.1, 1.6.2 and 1.6.3 asserted '!/(?i)linux' with no closing /, which goss reads as a
  literal string, so the OS-disclosure check never fired. Host-proven: all three banner
  files contained "Debian GNU/Linux" while the audit reported clean
- 1.1.1.6 checked the non-existent overlayfs module; the FATAL "module not found" message
  itself contained the string being grepped, so it always passed. Now overlay

- 6.2.3.10 was a placeholder that echoed "Manual" then asserted the output must not contain
  "Manual", so it could only ever fail and checked nothing. Replaced with conf and running
  checks on the -k privileged rules, in the POSIX echo-on-failure style used by 6.2.3.9

- updated from Audit to AUDIT

## Based on CIS v1.0.0 - Branch align_1.0.0

- 5.4.2.8 used bash process substitution. Goss runs commands under sh, which is dash on Debian,
  so the check failed to parse, produced no output and always reported compliant. Rewritten
  POSIX-safe and verified against a real host
- 5.4.1.6 used a bash [[ ]] test, so the comparison never fired, and asserted on "Failure" while
  the script echoes "failure". Both corrected, plus a guard for accounts with no last-change date
- 2.3.2.1 joined the configured NTP names with no separator, so the pattern only matched when
  exactly one pool or server was set
- Added .yamllint so the repo has a working YAML gate, and cleared the issues it surfaced in
  vars/CIS.yml (sequence indentation, comment spacing, missing trailing newline)
- Updated `run_audit.sh` to the current version shared by the other audit repos. OS discovery now
  derives the content path from `BENCHMARK_OS` instead of parsing `/etc/os-release`, and the goss
  version check reads only the first line of `goss -v` so multi-line version output no longer fails
  the pre-check. Minimum goss version raised to 0.4.8 in the script and the README
- Corrected 12 files gated on the wrong `deb13cis_rule_*` toggle, which silently disabled the neighbouring control's check: 1.1.2.6.4, 1.1.2.7.4, 2.1.7, 2.1.15, 2.1.21, 2.1.22, 2.1.23, 2.4.1.9, 3.2.3, 3.2.5, 3.2.6, 5.3.3.3.3
- Corrected the 1.7.10 / 1.7.11 GDM config path `/etc/gdm/custom.conf` -> `/etc/gdm3/custom.conf`, the path remediation actually writes on Debian
- Corrected the 6.3.3 AIDE config path `/etc/aide.conf` -> `/etc/aide/aide.conf` and the five audit tool paths `/sbin/*` -> `/usr/sbin/*`, matching remediation
- Removed 6 unreferenced ssh algorithm lists from vars/CIS.yml - no goss test consumed them and their names had drifted from the remediation `sshd_`-prefixed equivalents
- Aligned `deb13cis_legacy_boot` and `deb13cis_set_boot_pass` to the remediation defaults (false); as `true` they produced false failures in standalone audit runs
- Added CONTRIBUTING.rst (was missing) and replaced the stub .gitignore with the standard Lockdown set
- Fixed inverted 2.1.22 gate - the check only ran when the host *was* a mail server, the opposite of the remediation condition, so it never ran on a normal host
- Fixed 2.1.22 file resource path `/etc/postfix/main.conf` -> `/etc/postfix/main.cf`, the file the remediation actually writes
- Removed orphan `section_6/cis_6.1.1.1.x/cis_6.1.1.4.yml` - filename ID does not exist in v1.0.0, it reused the 6.1.1.1.4 toggle for an unrelated syslog-service check with no remediation counterpart
- Corrected the 2.1.22 title - wrong leading ID (2.1.21), missing separator, and singular "agent" where the benchmark says "agents are"
- Rewrote both 1.2.1.3 checks - they used bash process substitution `< <(find ...)` and `$'\0'`, which dash rejects outright, so the tests failed on syntax before any logic ran; the first also had a malformed line continuation that folded `done` into the `stat` arguments. Now a single POSIX `find` with `-exec`
- Rewrote 1.2.1.1 with `find` - it relied on brace expansion `*.{list,sources}`, which dash does not perform, so any offending file in `/etc/apt/sources.list.d/` would have been missed
- Corrected 1.3.1.4 expected value 0 -> 1. The benchmark contradicts itself: title, description and remediation all say the setting must be enabled (1), while its audit text says verify 0. The test had copied the audit text, so it disagreed with remediation; 0 would disable the protection
- Converted 1.5.12 / 1.5.13 from `command` + grep to `file:` resources on the exact paths remediation writes (`/etc/systemd/coredump.conf.d/60-coredump.conf` and `/etc/systemd/coredump.conf`). The old stdout patterns `'/^*.conf:...'` were degenerate (`^` followed by `*`) and could never match, and 1.5.13 searched only the drop-in directory while remediation writes `Storage=none` to `coredump.conf` itself
- Rewrote 5.1.4 - `exec` produced `AllowUsers <value>` while `stdout` expected `allowusers: <value>`, a format grep never emits, and all four allow/deny directives were required even though every one of the backing variables defaults to empty. Each directive is now asserted only when its variable is set
- Closed three unterminated regexes in 6.1.1.2.2 - `'/ServerKeyFile=.*.pem'` has no closing slash, so goss searched for that text literally. The file was correctly configured all along
- Rewrote 6.2.3.8 / 6.2.3.9 to assert a rule for every network config path that exists, matching the remediation, which gates each auditd rule on path existence. The tests previously demanded rules for `/etc/netplan` and `/etc/NetworkManager` unconditionally and so failed on hosts where those paths are absent
- Fixed a false pass in the 6.2.3.9 conf check - a plain `grep system-locale` matched the commented-out NetworkManager line, reporting compliance based on a comment. Both checks now match only lines beginning `-a`
- Fixed the 7.2.9 permissions check - `exec` ran bare `stat` (multi-line output) while `stdout` expected `stat -c '%a'` format, so it could never match. Now lists non-compliant home directories and asserts none

## March 2026 — audit alignment

- Removed CIS v8-only markers
- Added YAML document headers (`---`) to `goss.yml`, `standalone.yml`, and `vars/CIS.yml`
- Aligned Goss titles and `CIS_ID` values
- corrected several mis-keyed section 2 / 3 / 5 audit files
- Spellings
- variable naming aligned with remediate
  - deb13cis_gui variable renamed to deb13cis_desktop_required
  - deb13cis_time_pool_name to deb13cis_time_pool
  - deb13cis_syslog_server to deb13cis_system_is_log_server

# 1.0.0 - Initial
