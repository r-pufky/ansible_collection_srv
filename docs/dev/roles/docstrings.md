# Role Docstrings
Assumes ansible and collection [environment setup](../collection/setup.md).

# Variable Naming

## Definitions
All docstrings conform to **80** characters unless noted; variables are
unquoted in paragraphs my_var, my_var_{CUSTOM VALUE}, etc.

###### type
Abbreviated datatype
 label   | empty value
---------|------------------
 `int`   | `#` (no empty)
 `float` | `#.#` (no empty)
 `bool`  | `False`, `True` (no empty)
 `str`   | `''`
 `list`  | `[]`
 `dict`  | `{}`

###### description
Use present active voice. Include datatype if it is not implicitly apparent
what it is. Single line descriptions do not end in `.` unless noted.

###### detailed description
Detailed description explaining complex usage and edge cases; may be multiple
paragraphs.

###### description extended
Multi-line description for values (same as [description](#description)) ends in
a `.`.

###### var example
Working examples of var usage (may include multiple examples with one vertical
whitespace).

###### value
Explicit value including [empty datatypes](#empty)

 label              | use
--------------------|-------------------------------------------------------------------------------------
 `{VAR WITH SPACE}` | reference other variables
 `{data_type}`      | variable reference (e.g. `{[1]}` (first element of list)

###### comment
Additional context/clarification.

## Docstrings
All docstrings conform to **80** characters unless noted.

### Docstring: `Special Case`
Exceptions to `Values`, `Defaults`, or specific rules to follow for them.
``` yaml
Special Case:
  {VALUE}: {DESCRIPTON}
  {VALUE}: {DESCRIPTON EXTENDED}
           {DESCRIPTON EXTENDED}.
```
* Indent minimum of **2** spaces.
* Vertically align `:` separators.
* Single-lines have **no** `.`.
* Multi-lines end in `.`.

``` yaml
Special Case:
  ~: user home directory
  *: Tokens as specified from TOKENS section (see reference)
  *: ENVIRONMENT VARIABLES as specified from tokens section (see reference)
```

### Docstring: `Values`
Explicit accepted default values.
``` yaml
Values:
  {VALUE}: {DESCRIPTON}
  {VALUE}: {DESCRIPTON EXTENDED}
           {DESCRIPTON EXTENDED}.
```
* Indent minimum of **2** spaces.
* Vertically align `:` separators.
* Nested `Values` are indented **4** spaces from description.
* Single-lines have **no** `.`.
* Multi-lines end in `.`.

``` yaml
Values:
           none: explicitly disable use of an authentication agent
             '': no default system agent (default)
  SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable
             $*: strings beginning with $ will be treated as environment
                 variable containing the location of the socket.
                     Values:
                       {VALUE}: additional values for value element
```

### Docstring: `Default`
Defines a default value.
``` yaml
Default: {VALUE} [({COMMENT})].
```
* [`value`](#value)
* [`comment`](#comment)
* Always end in declarative `.`.

``` yaml
Default: [] (no options).

Default: ''.
```

### Docstring: `Reference`
Reference resources for additional information; may contain references to other
role-related files as well.
``` yaml
Reference:
* {REFERENCES}
```
* `references`: string reference, **no** `.`; no line-length limit.

``` yaml
Reference:
* https://example.com/reference/url/over/80/characters
```

### Variable: `single-line`
``` yaml
# {DESCRIPTION}. {DOCSTRING DEFAULT}.
```
* [`description`](#description)
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
Simple variable with mutiple values.
``` yaml
# {DESCRIPTION}.
#
# {DETAILED DESCRIPTION}.
#
# {DOCSTRING SPECIAL CASE}
#
# {DOCSTRING VALUES}
#
# {DOCSTRING DEFAULT}.
#
# {DOCSTRING REFERENCE}
```
* [`description`](#description)
* [`detailed description`](#detailed-description)
* [`docstring special case`](#docstring-special-case)
* [`docstring values`](#docstring-values)
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
#   ~: user home directory
#   *: Tokens as specified from TOKENS section (see reference)
#   *: ENVIRONMENT VARIABLES as specified from tokens section (see reference)
#
# Values:
#            none: explicitly disable use of an authentication agent
#              '': no default system agent (default)
#   SSH_AUTH_SOCK: get socket location from SSH_AUTH_SOCK environment variable
#              $*: strings beginning with $ will be treated as environment
#                  variable containing the location of the socket.
#                      Values:
#                        {VALUE}: additional values for value element
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
# {DETAILED DESCRIPTION}.
#
# {DOCSTRING SPECIAL CASE}
#
# {ROLE}_variable_name:
#     {TYPE} - {DESCRIPTION}[.]
#              {DESCRIPTON EXTENDED}.
#   {VALUE}: {TYPE} - {DESCRIPTION}
#         {DESCRIPTION EXTENDED}.
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
#     {VALUE}: {TYPE} - {DESCRIPTION}
#           {DESCRIPTION EXTENDED}.
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
* [`description`](#description)
* [`detailed description`](#detailed-description)
* [`docstring special case`](#docstring-special-case)
* [`value`](#value)
* [`type`](#type)
* [`docstring values`](#docstring-values)
* [`docstring default`](#docstring-default): always define even if actual value
  is multi-line.
* `var example`: working examples of var usage (may include multiple).
* [`docstring reference`](#docstring-reference)

``` yaml
# Manage files within the first directory listed in ssh_client_include.
#
# role_multi_complex:
#     list of dict - definitions for local configuration.
#   - config: str - name of local config ({INCLUDE}/{CONFIG})
#     state: str - whether config should be managed or removed
#           Additional description details here.
#         Values:
#           present: manage configuration
#            absent: remove configuration
#         Default: 'present'.
#     options: dict - ssh_config options to manage (per ssh_config options)
#       file: str - file location
#       mode: str - octal file permissions
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

# Role Parameters
Defined when variables are used within role tasks which are not included within
`defaults` definitions.

## Args
Task consumes these parameters.

``` yaml
# Args:
#   {VALUE}: {TYPE} - {DESCRIPTION}
#         {DESCRIPTION EXTENDED}.
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
```
* Appears at file header comment.
* Follow variable formatting rules.

``` yaml
# Args:
#   _overseerr_target: str - full version (vx.x.x) to install
#   _overseerr_target_dir: str - versioned target install location
#   _overseerr_target_url: str - download url for target release
#   _overseerr_archive: str - local versioned archive location
```

## Generates
Variables created in tasks intended to be consumed by other tasks.

``` yaml
# Generates:
#   {VALUE}: {TYPE} - {DESCRIPTION}
#         {DESCRIPTION EXTENDED}.
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
```
* Appears at file header comment.
* Follow variable formatting rules.

``` yaml
# Generates:
#   _overseerr_current: str - full version (vx.x.x) of current installed
#       version. '' if there is no detected installed version
#   _overseerr_target_url: str - download url for target release
#   _overseerr_existing_installs: stat - for existing versioned directories
```