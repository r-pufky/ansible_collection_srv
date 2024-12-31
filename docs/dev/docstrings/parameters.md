# Docstrings: Parameters
Defined when variables are used within role tasks which are not included within
`defaults` option definitions.

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
* Do **not** include `default` and `vars` variables.
* Follow [style guidelines](definitions.md#style)
* [`value`](definitions.md#value)
* [`type`](definitions.md#type)
* [`description`](definitions.md#description)
* [`description-extended`](definitions.md#description-extended)
* [`docstring special case`](variables.md#docstring-special-case)
* [`docstring default`](variables.md#docstring-default)

``` yaml
# Args:
#   _overseerr_target: str - full version (vx.x.x) to install
#   _overseerr_target_dir: str - versioned target install location
#   _overseerr_target_url: str - download url for target release
#   _overseerr_archive: str - local versioned archive location
```

## Generates
Variables created in tasks intended to be consumed by other tasks within the
same role.

``` yaml
# Generates:
#   {VALUE}: {TYPE} - {DESCRIPTION}
#         {DESCRIPTION EXTENDED}.
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
```
* Appears at file header comment below `Args:`.
* Do **not** include `default` and `vars` variables.
* Follow [style guidelines](definitions.md#style)
* [`value`](definitions.md#value)
* [`type`](definitions.md#type)
* [`description`](definitions.md#description)
* [`description-extended`](definitions.md#description-extended)
* [`docstring special case`](variables.md#docstring-special-case)
* [`docstring default`](variables.md#docstring-default)

``` yaml
# Generates:
#   _overseerr_current: str - full version (vx.x.x) of current installed
#       version. '' if there is no detected installed version
#   _overseerr_target_url: str - download url for target release
#   _overseerr_existing_installs: stat - for existing versioned directories
```

### Exports
Variables created in the role that are intended for external consumption in the
same play by other roles. **Requires** updating role `README.md`.

Only list variables in `Exports` if they are both **generated** and
**exported**.

``` yaml
# Exports:
#   {VALUE}: {TYPE} - {DESCRIPTION}
#         {DESCRIPTION EXTENDED}.
#       {DOCSTRING SPECIAL CASE}
#       {DOCSTRING VALUES}
#       {DOCSTRING DEFAULT}.
```
* Appears at file header comment below `Args:` and `Generates`.
* Do **not** include `default` and `vars` variables.
* Follow [style guidelines](definitions.md#style)
* [`value`](definitions.md#value)
* [`type`](definitions.md#type)
* [`description`](definitions.md#description)
* [`description-extended`](definitions.md#description-extended)
* [`docstring special case`](variables.md#docstring-special-case)
* [`docstring default`](variables.md#docstring-default)

``` yaml
# Exports:
#   _sqlite_sql_results: dict - return values from command execution.
```

README.md
``` md
### Generated Variables
After successful execution the following variables are available for further
manipulation during the same play (standard role variable scope):

 Variable            | type | Description
---------------------|------|-----------------------------------------
 _sqlite_sql_results | dict | registered return results from command.
```
