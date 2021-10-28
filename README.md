# semtag-generator
A bash script to help generate git tags following semantic versioning

Generates, increments and pushes an annotated semantic version tag for current git repository.

## Usage:
  `sh tag-release [-b|--branch] [-d|--date] [-h|--help|help] [-m|--message] [-n|--dry-run] [-p|--previous] [-q|--quiet] [-r|--remote] [-t|--type] [-v|--verbose]`

## Options:
`-b, --branch <branch>`

Branch to generate release tag on.
     If ommited, defaults to '*main*'

`-d, --date <date>`

Date string to specify when release was created.

If ommited, defaults to *%Y%m%d* of current date.


`-h, --help, help`

Prints this help.

`-m, --message <message>`

Message to use to annotate release.

If ommited a list of non-merge commit messages will be compiled as release annotation.

If ommited and `-p <commit>` is given, will compile a list of non-merge commit messages between `<commit>` and HEAD.

If ommited and `-p` is not given, will compile a list of non-merge commit messages between last found release and HEAD.

`-n, --dry-run`

Do everything except actually send the updates.

If -q is also given then only error messages will be output.

`-p, --previous <commit>`

Previous commit to use to generate release notes.

If ommited, will attempt to get commit hash of last release tag.

`-r, --remote <remote>`

Name of remote to use for pushing.

If ommited, defaults to '*origin*'

`-t, --type [major|minor|patch]`

Type of semantic version to create. Valid options are '*major*', '*minor*' or '*patch*'

- major: Will bump up to next major release (*i.e 1.0.0 -> 2.0.0*)
- minor: Will bump up to next minor release (*i.e 1.0.1 -> 1.1.0*)
- patch: Will bump up to next patch release (*i.e 1.0.2 -> 1.0.3*)

If ommited, will default to '*patch*'

`-v, --verbose`

Run verbosley.
