# Docstrings: Definitions
All styles are required unless noted for specific definitions.

[Variables](variables.md)

[Task Parameters](parameters.md)

## Style
* **80** character width.
* **Single** vertical spacing between sections.
* **4** spaces for a continued line.
* Variables **unquoted** in paragraphs:
  ``` yaml
  # this is test containing role_variables and
  # usage of another_variable.
  ```
* Follow `yamllint` and `ansible-lint` style guides:
  * `{ROLE}_{VAR}`: defined in `defaults`.
  * `{ROLE}_role_{VAR}`: defined in `vars`.
  * `_{ROLE}_{VAR}`: *private* role vars, `args`, `generates`, and `exports`.

## Definitions

###### type
Abbreviated datatype
 label              | empty value
--------------------|------------------
 `int`              | `#` (no empty)
 `float`            | `#.#` (no empty)
 `bool`             | `False`, `True` (no empty)
 `str`              | `''`
 `list`             | `[]`
 `dict`             | `{}`
 `{TYPE} or {TYPE}` | multiple types, preferred type first

###### description
Use present active voice. Include datatype if it is not implicitly apparent
what it is. Single line descriptions do not end in `.` unless noted.

###### detailed description
Detailed description explaining complex usage and edge cases; may be multiple
paragraphs.

###### description extended
Multi-line description for values (same as [description](#description)) ends in
a `.`.

###### decision
Concise and clear decision on a complex topic.

###### ID
Short identifier for future reference. These should focus on being explicit as
possible:

* `{ROLE}` for anything role related.
* `{VERSION}` for anything requiring a specific ansible version.
* `{RELEASE}` for any OS/service release related.

###### var example
Working examples of var usage (may include multiple examples with one vertical
whitespace).

###### value
Explicit value including [empty datatypes](#type).

 label              | use
--------------------|----------------------------------------------------------
 `{VAR WITH SPACE}` | reference other variables
 `{data_type}`      | variable reference (e.g. `{[1]}` (first element of list))
 `{VAL}`            | variable value substitution.
 `{REF}`            | use syntax format from references (TIME, SIZE)
 `{OMIT}`           | behavior if this variable is undefined
 `^$*{VAL}`         | regex expressions are allowed; use {VAL}

###### comment
Additional context/clarification.
