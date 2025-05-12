# Docstrings: Variables

Prerequisite:
* [definitions](definitions.md)

### Docstring: `Comment`
Inline comments are **not** a sentence and do not required a period or capital
to start (but are preferred). Full line comments which are statements do.
``` yaml
# Full line statement comment with period.

line of code  # inline comment
gather_facts: true  # ansible_virtualization_role
package import for testing  # ansible.builtin.module (specific reason)
```

### Docstring: `Requires`
Explicit requirements for variable to be used or activated.
``` yaml
Requires {VAR}={VALUE}.
```
* Immediately following one-line variable description.
* Simple requirements only; if description is needed use complex Requires
  (below).
* lines end in `.`.

``` yaml
Requires:
  * {VAR}={VALUE} - {DESCRIPTION}.
  * {VAR}={VALUE} - {DESCRIPTION}.
  * {VAR}={VALUE} {OP} {VAR}={VALUE} ... - {DESCRIPTION}
```
* [`description`](definitions.md#description) for nuance only.
* Indent minimum of **2** spaces.
* Multi-lines end in `.`.

### Docstring: `Special Case`
Exceptions to `Values`, `Defaults`, or specific rules to follow for them.
``` yaml
Special Case:
  {VALUE}[:] {DESCRIPTION}.
  {VALUE}[:] {DESCRIPTION EXTENDED}.
             {DESCRIPTION EXTENDED}.
```
* Indent minimum of **2** spaces.
* Vertically align `:` separators.
* `:` dropped if special case is only a list that applies to everything.
* Multi-lines end in `.`.
* Internal (variable cases) only; external requirements should be placed in
  [`Requires`](variables.md#docstring-requires).

``` yaml
Special Case:
    ~: user home directory.
    *: Tokens as specified from TOKENS section (see reference).
    *: ENVIRONMENT VARIABLES as specified from tokens section (see reference).
  ':': clarify colon usage or cases where colon could be confused.

Special Case:
  * All paths are prepended with some_var.
  * Wildcards allowed.
```

### Docstring: `Values`
Explicit accepted default values.
``` yaml
Values:
  {VALUE}: {DESCRIPTON}.
  {VALUE}: {DESCRIPTON EXTENDED}.
           {DESCRIPTON EXTENDED}.
```
* Indent minimum of **2** spaces.
* Vertically align `:` separators.
* Nested `Values` are indented **4** spaces from description.
* Single-lines have **no** `.`.
* Multi-lines end in `.`.

``` yaml
Values:
           none: explicitly disable use of an authentication agent.
             '': no default system agent (default).
            ':': clarify colon usage or cases where colon could be confused.
  SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable.
             $*: strings beginning with $ will be treated as environment
                 variable containing the location of the socket.
                     Values:
                       {VALUE}: additional values for value element.
```

### Docstring: `Default`
Defines a default value.
``` yaml
Default: {VALUE} [({COMMENT})].
Default: N/A ({COMMENT}).
```
* [`value`](definitions.md#value)
* [`comment`](definitions.md#comment)
* [`N/A`](definitions.md#value)
* Required values (wiht no defaults) must use `N/A` with a comment; these
  should be exceptions (for instance with systemd).
* Always end in declarative `.`.

``` yaml
Default: [] (no options).

Default: ''.
```

### Docstring: `Variable`
Defines an inline extended variable for a variable defined in another file.
**Only** used for extremely large and complex configurations that have multiple
overlapping, duplicated options, resulting in 5k+ line default values files.

Extended options are defined in a separate file (with no actual YAML variables)
and cross-reference the actual configuration location.

See `r_pufky.srv.systemd` defaults for example.
``` yaml
# Variable: {VARIABLE} | {TYPE}
```
* `variable` defines the variable name.

``` yaml
# Variable: exec_search_path | str
```

### Docstring: `Reference`
Reference resources for additional information; may contain references to other
role-related files as well.
``` yaml
# Reference:
# * {REFERENCES}
```
* `references`: string reference
* no line-length limit.

``` yaml
Reference:
* https://example.com/reference/url/over/80/characters
```

### Docstring: `Security`
Reference specific security mitigations. Must include decision, reference, and
CVE (or other reference) if it exists. Convey the impact of the vulnerability
and why it is implemented.

Disabled options should remain (but commented out) for easy re-implementation
by user..
``` yaml
# Security: {CVE}
#   CVE: {LINK}
#   * {LINK}
#   Decision: {DECISION} - {DESCRIPTION}.
#   Mitigation: {DESCRIPTION}.
#   {REFERENCE}
```
* use same section for all locations that it is implemented (write it once and
  copy to each location).
* primary CVE listed in header if available.
* related CVE listed in CVE.
* [`description`](definitions.md#description)
* [`decision`](#docstring-decision)
* [`reference`](#docstring-reference)
* no line-length limit.
* Place search instructions in defaults/main.yml to locate:

``` yaml
# Find security mitigations by searching for CVE or security headers:
#
# grep -ri 'CVE'
# grep -ri 'Security:'
```

``` yaml
# Security: CVE-2023-48795
#   CVE:
#   * https://nvd.nist.gov/vuln/detail/cve-2023-48795
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46445
#   * https://nvd.nist.gov/vuln/detail/cve-2023-46446
#   * https://nvd.nist.gov/vuln/detail/cve-2024-41909
#   Decision: remove chacha20 cipher, *-etm MAC - disables attack for modern
#       systems, remove when OpenSSH 9.6 released with strict key exchange.
#   Mitigation:
#   * remove 'chacha20-poly1305@openssh.com' from client ciphers.
#   * remove '*-etm@openssh.com' from server macs.
#   Reference:
#   * https://terrapin-attack.com/#scanner
#   * https://www.openssh.com/txt/release-9.6
#
# Default:
#   # - 'chacha20-poly1305@openssh.com'
#   - 'aes128-ctr'
#   - 'aes192-ctr'
#   - 'aes256-ctr'
#   - 'aes128-gcm@openssh.com'
#   - 'aes256-gcm@openssh.com'
ssh_client_ciphers:
  # - 'chacha20-poly1305@openssh.com'
  - 'aes128-ctr'
  - 'aes192-ctr'
  - 'aes256-ctr'
  - 'aes128-gcm@openssh.com'
  - 'aes256-gcm@openssh.com'
```

### Docstring: `WARNING`
Only for critical cases which a hard to debug failure will occur. These should
be limited in use.

``` yaml
# WARNING
# > {DESCRIPTION}
```
* All warnings are copied to README.md.
* use `>` qoutes for description block.
* [`description`](definitions.md#description)

### Docstring: `NOTE`
Only for critical cases which a hard to debug failure will occur; these are not
exposed to the consumer, but the developer of the role.

``` yaml
# NOTE
# > {DESCRIPTION}
```
* use `>` qoutes for description block.
* [`description`](definitions.md#description)

### Docstring: `Decision`
Explain an opinionated or complex decision for implementing a specific way. Be
clear and concise as to the decision and the supporting reasoning.

``` yaml
# Decision: {DECISION} - {DESCRIPTION}.
# Decision: {DECISION} - {DESCRIPTION}.
#     {DESCRIPTION EXTENDED}.
```
* `decision`
* [`description`](definitions.md#description)
* [`description extended`](definitions.md#description-extended)
* Insert next to first step in implementation of decision.

### Docstring: `TODO`
Identifying future work or known issues.

``` yaml
# TODO({ID}): {DESCRIPTION}.
# TODO({ID}): {DESCRIPTION EXTENDED}.
#     {DESCRIPTION EXTENDED}.
```
* [`id`](#id)
* [`description`](definitions.md#description)
* [`description extended`](definitions.md#description-extended)

### Docstring: `File`
Show file location, user, permissions and contents. May be in diff format.

`0755 root root` /file/location.yml
``` yaml
---
some_yaml:
  listing
```

`0644 {USER} {USER}` /file/with/diff.conf
``` diff
-old line
+new line
```

### Variable: `single-line`
``` yaml
# {DESCRIPTION}. {DOCSTRING DEFAULT}.
```
* [`description`](definitions.md#description)
* [`docstring default`](#docstring-default)

``` yaml
# Enabled periodic upgrades and updates. Default: True.
role_variable: true

# Address. Default: 'any' (any address family).
role_address: 'any'

# List of str filters to use. Default: ['test', 'values', 'example'].
role_filters:
  - 'test'
  - 'values'
  - 'example'
```

### Variable: `multi-line`
Simple variable with multiple values.
``` yaml
# {DESCRIPTION}.
#
# {REQUIRES}.
#
# {DETAILED DESCRIPTION}.
#
# {DOCSTRING SPECIAL CASE}
#
# {DOCSTRING VALUES}
#
# {DOCSTRING VARIABLE}
#
# {DOCSTRING DEFAULT}.
#
# {DOCSTRING REFERENCE}
```
* [`description`](definitions.md#description)
* [`detailed description`](definitions.md#detailed-description)
* [`docstring special case`](#docstring-special-case)
* [`docstring values`](#docstring-values)
* [`docstring variable`](#docstring-variable):
  * **optional**
  * Used for extended variable file definitions. See reference.
* [`docstring default`](#docstring-default):
  * Always define even if actual value is multi-line.
  * Prefer single-line Default line for this docstring if possible.
* [`docstring reference`](#docstring-reference)

``` yaml
# Unix domain socket to communicate with the authentication agent.
#
# Overrides SSH_AUTH_SOCK environment variable.
#
# Special Case:
#   ~: user home directory.
#   *: Tokens as specified from TOKENS section (see reference).
#   *: ENVIRONMENT VARIABLES as specified from tokens section (see reference).
#
# Values:
#            none: explicitly disable use of an authentication agent.
#              '': no default system agent (default).
#   SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable.
#              $*: strings beginning with $ will be treated as environment
#                  variable containing the location of the socket.
#                      Values:
#                        {VALUE}: additional values for value element.
#
# Default: ''.
#
# Reference:
# * https://exampl.com/reference/url
role_multi_simple: ''
```

### Variable: `multi-line-complex`
Complex variable (holds multiple simple variables) with multiple values.

Use standard YAML formatting for nested variable definitions.
``` yaml
# {DESCRIPTION}.
#
# {REQUIRES}.
#
# {DETAILED DESCRIPTION}.
#
# {DOCSTRING SPECIAL CASE}
#
# {ROLE}_variable_name:
#     {TYPE} - {DESCRIPTION}.
#              {DESCRIPTION EXTENDED}.
#   {VALUE}: {TYPE} - {DESCRIPTION}.
#         {DESCRIPTION EXTENDED}.
#
#         {DOCSTRING REFERENCE}
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
#     {VALUE}: {TYPE} - {DESCRIPTION}.
#           {DESCRIPTION EXTENDED}.
#
#           {DOCSTRING REFERENCE}
#         {DOCSTRING SPECIAL CASE}
#         {DOCSTRING VALUES}
#         {DOCSTRING DEFAULT}.
#
# {VAR EXAMPLE}
#
# {DOCSTRING DEFAULT}.
#
# {DOCSTRING REFERENCE}
```
* [`description`](definitions.md#description)
* [`description extended`](definitions.md#description-extended)
* [`detailed description`](definitions.md#detailed-description)
* [`docstring special case`](#docstring-special-case)
* [`value`](definitions.md#value)
* [`type`](definitions.md#type)
* [`docstring values`](#docstring-values)
* [`docstring default`](#docstring-default): always define even if actual value
  is multi-line.
* `var example`: working examples of var usage (may include multiple).
* [`docstring reference`](#docstring-reference)
* May be unindented if variables are pulled to separate files due to large
  complex configurations requiring many duplicates. See systemd role.

``` yaml
# Manage files within the first directory listed in ssh_client_include.
#
# Requires role_other_option=true.
#
# role_multi_complex:
#     list of dict - definitions for local configuration.
#   - config: str - name of local config ({INCLUDE}/{CONFIG}).
#     state: str - whether config should be managed or removed.
#           Additional description details here.
#
#           Reference:
#           * https://example.com
#         Values:
#           present: manage configuration
#            absent: remove configuration
#         Default: 'present'.
#     options: dict - ssh_config options to manage (per ssh_config options).
#       file: str - file location.
#       mode: str - octal file permissions.
#           Default: '0644'.
#
# role_multi_complex:
#   - config: 'custom_ssh_config'
#     state: 'present'
#     options:
#       file: '/etc/example'
#       mode: '0644'
#
# role_multi_complex:
#   - config: 'alternative_config'
#     state: 'absent'
#   - config: 'custom_ssh_config'
#     state: 'present'
#     options:
#       file: '/etc/example'
#       mode: '0644'
#
# Default: [] (un-managed).
#
# Reference:
# * https://manpages.debian.org/bookworm/openssh-client/ssh_config.5.en.html
role_multi_complex: []
```
