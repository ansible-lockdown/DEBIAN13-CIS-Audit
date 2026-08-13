# Debian 13 CIS

## Based on CIS v1.1.0 - Branch benchmark_1.1.0

Upgrade from CIS Debian Linux 13 Benchmark v1.0.0 to v1.1.0. 350/350 controls covered.

- Retire 14 goss files for the controls v1.1.0 drops or merges: 1.5.10, 1.7.1, 1.7.3 to 1.7.9,
  6.1.1.1.4, 6.1.1.2.1 to 6.1.1.2.3 and 6.2.3.34
- Renumber 48 goss files, matched to the new numbering by control title rather than by position
- Remove the section_6/cis_6.1.1.2.x directory, v1.1.0 has no 6.1.1.2 subsection. Its surviving
  check moves to cis_6.1.1.1.x as 6.1.1.1.2, and goss.yml no longer includes the directory
- 6.1.2.3 becomes 6.1.1.1.3 but stays in the cis_6.1.2.x directory. goss.yml includes that
  directory only when deb13cis_syslog is rsyslog, which is the only case where "journald is
  configured to send logs to rsyslog" applies, so the audit gate mirrors the remediation gate
- Add 21 goss files for the new controls: 1.2.1.10 to 1.2.1.15, 1.6.4, 1.6.5, 1.6.9 to 1.6.12,
  1.7.2 to 1.7.5, 3.2.7, 3.3.1.19, 5.1.2, 5.3.1.4 and 7.2.11
- Add deb13cis_sshd_banner_file, deb13cis_disable_dynamic_motd, deb13cis_screensaver_idle_delay
  and deb13cis_screensaver_lock_delay to vars/CIS.yml, all passed through from the remediation role
- Move 1.2.1.3, 3.3.1.1, 5.1.9 and 5.1.10 to the Level 1 gate, following the v1.1.0 profile change
- The 1.6.11 and 1.6.12 checks assert that update-notifier-motd is neither enabled nor active
  rather than using a service resource, so they pass cleanly on Debian 13 where the unit is absent
  and still fail if it is installed and enabled
- 7.2.11 asserts on a counted result rather than on empty output, so a check that stops producing
  output fails instead of silently passing
- Fix 1.7.1, which defined the goss key `gdm_profile_banner` twice under the same `command:`
  block. Goss refused to parse the file and **aborted the entire audit** with
  "mapping key already defined", so any host with deb13cis_desktop_required set to true got no
  audit at all. Split into gdm_banner_message_enable and gdm_banner_message_text. Found by a real
  host run, not by yamllint or the gap report - PyYAML accepts duplicate keys and keeps the last
- Fix 1.7.6 and 1.7.7, which both registered a `file:` resource under the key
  /etc/gdm3/custom.conf. Goss merges every gossfile fragment into one namespace, so 1.7.7 silently
  replaced 1.7.6 and the XDMCP check never ran - it was absent from the results rather than
  failing. Keys are now gdm_xdmcp_disabled and gdm_xwayland_configured
- 3.2.7 is a Manual control. It now reports the loaded network protocol modules for review using
  the "Manual Check Required" pattern, rather than asserting no kernel/net module is loaded, which
  failed on any host with normal networking (14 modules loaded on a stock Debian 13)

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
