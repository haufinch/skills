---
name: r-conventions
description: R code conventions for this user — function and argument naming schemes, argument defaults, function definition layout, package installation, and formatting with air. Load before writing or editing any R file (.R, .r, .Rmd, .qmd).
---

# R conventions

## Packages

Install missing packages rather than working around them or asking.

## Formatting

Format files with `air`.

## Function naming

Name functions `(group)?_(subgroup)_(action|assertion)?_(subject)?`.

`group`, `subgroup` and `subject` may be multi-word.

### group

- How a set of functions are related.
- A noun when grouping components: `input_file()`, `input_numeric()`.
- A verb when grouping processes: `build_budget()`, `build_report()`.
- Never include the group in an internal helper — it is wrong and it spends the
  character budget. `input_file()`'s helpers are `file_check_path()` and
  `file_check_size()`, not `input_file_check_size()`.

### subgroup

- How helper functions are related.
- Always required.
- A noun: `file`, `date_range`, `amort_group`.
- A helper inherits its parent's subgroup. That inheritance is what ties
  `file_check_path()`, `file_read_contents()` and `file_button()` back to
  `input_file()`.
- Read the inheritance backwards to split a multi-word name: the subgroup is
  the token that appears in the helper names. `amort_group_check_rate()` shows
  `budget_amort_group_is_valid()` splits as `budget` + `amort_group`.

### action

- What the function does.
- A verb: `report_compose()`, `report_write()`.
- Required whenever the function does something: `file_read_contents()`, not
  `file_contents()`.
- Omit for constructors, which are named for what they return, not what they
  do: `input_file()`, `file_button()`.

### assertion

- What the function checks.
- A third-person verb, for predicate functions: `budget_is_valid()`,
  `budget_has_total()`.
- High-level assertions throw errors: `validate_schedule()`.
- Low-level assertions return a boolean: `budget_has_deficit()`.

### subject

- What the action operates on, what the assertion is about, or what a
  constructor returns.
- Optional.
- Singular, unless the noun has no singular form: `contents`, `data`.
- A noun after an action or `has`: `file_check_path()`, `budget_has_total()`.
- An adjective after `is`: `budget_is_valid()`, `scenario_is_baseline()`.

## Function definition layout

Define standalone functions with the function name and `<-` on one line, then an 
indented `function(`, one argument per line:

```r
budget_table_fetch <-
  function(
    period,
    ...,
    scenario = NULL
  ) {
    ...
  }
```

Does not apply to R6 methods.

## Function arguments

Name arguments `(group)?_(name)`.

Never use `.` as a separator, use `_`. A leading `.` is allowed, and useful for
avoiding collisions with user arguments in functions taking `...`:
`map(.x, .f)`. See <https://design.tidyverse.org/dots-prefix.html>.

### group

- What relates a set of arguments.
- Optional.
- Include to associate related arguments:
  `report_write(content, page_size = "letter", page_margin = 1)`.
- Never repeats the function's group or subgroup.

### name

- What the argument is.
- Always required.
- A noun. Wanting a verb is a sign the work belongs in a helper function.

## Argument defaults

- Required arguments do not have default values.
- Prefer enumerated choices over booleans: `na = c("drop", "error")`, not
  `na.rm = TRUE`. See <https://design.tidyverse.org/boolean-strategies.html>.
- Match the choice in the body with `match.arg()` or `rlang::arg_match()`.
- Arguments with defaults come after `...`, when `...` is present.
