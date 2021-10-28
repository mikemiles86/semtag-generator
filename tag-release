#!/bin/bash
ARGUMENTS="$@"
BRANCH="main"
DRYRUN="0"
GITPARAMS=()
RELEASEDATE=$(date '+%Y%m%d')
RELEASENOTES=""
REMOTE="origin"
PREVIOUS_COMMIT=""
RUNSILENT="0"
VERBOSE="0"
VERSIONTYPE="patch"


# Function to return help text when requested with -h option.
help() {
    # Display help.
    echo
    echo "Generates, increments and pushes an annotated semantic version tag for current git repository."
    echo
    echo "Usage:"
    echo "  sh tag-release [-b|--branch] [-d|--date] [-h|--help] [-m|--message] [-p|--previous] [-r|--remote] [-v|--version-type]"
    echo
    echo "Options:"
    echo "-b,--branch <branch>"
    echo "      Branch to generate release tag on."
    echo "      If ommited, defaults to '$BRANCH'"
    echo
    echo "-d,--date <date>"
    echo "      Date string to specify when release was created."
    echo "      If ommited, defaults to %Y%m%d of current date."
    echo
    echo "-h,--help, help"
    echo "      Prints this help."
    echo
    echo "-m,--message <message>"
    echo "      Message to use to annotate release."
    echo "      If ommited a list of non-merge commit messages will be compiled as release annotation."
    echo "      If ommited and -p <commit> is given, will compile a list of non-merge commit messages between <commit> and HEAD."
    echo "      If ommited and -p is not given, will compile a list of non-merge commit messages between last found release and HEAD."
    echo
    echo "-n,--dry-run"
    echo "      Do everything except actually send the updates."
    echo "      If -q is also given then only error messages will be output."
    echo
    echo "-p,--previous <commit>"
    echo "      Previous commit to use to generate release notes."
    echo "      If ommited, will attempt to get commit hash of last release tag."
    echo
    echo "-q,--quiet"
    echo "      Supress all output, unless an error occurs."
    echo
    echo "-r,--remote <remote>"
    echo "      Name of remote to use for pushing."
    echo "      If ommited, defaults to '$REMOTE'"
    echo
    echo "-t,--version-type [major|minor|patch]"
    echo "      Type of semantic version to create. Valid options are 'major', 'minor' or 'patch'"
    echo "          major: Will bump up to next major release (i.e 1.0.0 -> 2.0.0)"
    echo "          minor: Will bump up to next minor release (i.e 1.0.1 -> 1.1.0)"
    echo "          patch: Will bump up to next patch release (i.e 1.0.2 -> 1.0.3)"
    echo "      If ommited, will default to '$VERSIONTYPE'"
    echo
    echo "-v,--verbose"
    echo "      Run verbosley."
    echo
}

conditional_echo() {
    if [[ "$RUNQUIET" -eq 0 ]]; then
        echo "$1"
    fi
}

while [[ "$#" -gt 0 ]]
do
    case $1 in
      -b|--branch)
        BRANCH=$2
        ;;
      -d|--date)
        RELEASEDATE=$2
        ;;
      -h|--help|help)
        help
        exit
        ;;
      -m|--message)
        RELEASENOTES=$2
        ;;
      -n|--dry-run)
        DRYRUN="1"
        ;;
      -p|--previous)
        PREVIOUS_COMMIT=$2
        ;;
      -r|--remote)
        REMOTE=$2
        ;;
      -t|--type)
        VERSIONTYPE=$2
        ;;
      -q|--quiet)
        RUNQUIET="1"
        GITPARAMS+=(--dry-run)
        ;;
      -v|--verbose)
        # See if a second argument is passed, due to argument reassignment to -v.
        if [[ "$1" == "-v" ]] && [[ -n "$2" ]]; then
            echo "ERROR: Unsupported value \"$2\" passed to -v argument. If trying to set semantic version tag, use the -t or --type argument".
            exit
        fi
        VERBOSE="1"
        GITPARAMS+=(--verbose)
        ;;
    esac
    shift
done

# Get top-level of git repo.
REPO_DIR=$(echo $(git rev-parse --show-toplevel))
# CD into the top level
cd "${REPO_DIR}"

# Get current active branch
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
# Switch to production branch
if [ $CURRENT_BRANCH != "$BRANCH" ]; then
    conditional_echo "- Switching from $CURRENT_BRANCH to $BRANCH branch. (stashing any local change)"
    # stash any current work
    git stash "${GITPARAMS[@]}"
    # go to the production branch
    git checkout $BRANCH "${GITPARAMS[@]}"
fi

conditional_echo "- Updating local $BRANCH branch."
# pull latest version of production branch
git pull $REMOTE $BRANCH "${GITPARAMS[@]}"
# fetch remote, to get latest tags
git fetch $REMOTE "${GITPARAMS[@]}"

# Get previous release tags
conditional_echo "- Getting previous tag."
PREVIOUS_TAG=$(echo $(git ls-remote --tags --ref --sort="v:refname" $REMOTE | tail -n1))

# If specific commit not set, get the from the previous release.
if [ -z "$PREVIOUS_COMMIT" ]; then
    # Split on the first space
    PREVIOUS_COMMIT=$(echo $PREVIOUS_TAG | cut -d' ' -f 1)
fi

conditional_echo "-- PREVIOUS TAG: $PREVIOUS_TAG"

# Get previous release number
PREVIOUS_RELEASE=$(echo $PREVIOUS_TAG | cut -d'/' -f 3 | cut -d'v' -f2 )

conditional_echo "- Creating release tag"
# Get last commit
LASTCOMMIT=$(echo $(git rev-parse $REMOTE/$BRANCH))
# Check if commit already has a tag
NEEDSTAG=$(echo $(git describe --contains $LASTCOMMIT 2>/dev/null))

if [ -z "$NEEDSTAG" ]; then
    conditional_echo "-- Generating release number ($VERSIONTYPE)"
    # Replace . with spaces so that can split into an array.
    VERSION_BITS=(${PREVIOUS_RELEASE//./ })
    # Get number parts, only the digits.
    VNUM1=${VERSION_BITS[0]//[^0-9]/}
    VNUM2=${VERSION_BITS[1]//[^[0-9]/}
    VNUM3=${VERSION_BITS[2]//[^0-9]/}
    # Update tagging number based on option that was passed.
    if [ "$VERSIONTYPE" == "major" ]; then
        VNUM1=$((VNUM1+1))
        VNUM2=0
        VNUM3=0
    elif [ "$VERSIONTYPE" == "minor" ]; then
        VNUM2=$((VNUM2+1))
        VNUM3=0
    else
        # Assume TAGTYPE = "patch"
        VNUM3=$((VNUM3+1))
    fi

    # Create new tag number
    NEWTAG="v$VNUM1.$VNUM2.$VNUM3"
    conditional_echo "-- Release number: $NEWTAG"
    # Check to see if new tag already exists
    TAGEXISTS=$(echo $(git ls-remote --tags --ref $REMOTE | grep "$NEWTAG"))

    if [ -z "$TAGEXISTS" ]; then
        # Check if release notes were not provided.
        if [ -z "$RELEASENOTES" ]; then
            conditional_echo "- Generating basic release notes of commits since last release."
            # Generate a list of commit messages since the last release.
            RELEASENOTES=$(git log --pretty=format:"- %s" $PREVIOUS_COMMIT...$LASTCOMMIT  --no-merges)
        fi
        # Tag the commit.
        if [[ "$DRYRUN" -eq 0 ]]; then
            conditional_echo "-- Tagging commit. ($LASTCOMMIT)"
            git tag -a $NEWTAG -m"$RELEASEDATE: Release $VNUM1.$VNUM2.$VNUM3" -m"$RELEASENOTES" $LASTCOMMIT
            conditional_echo "- Pushing release to $REMOTE"
            # Push up the tag
            git push $REMOTE $NEWTAG "${GITPARAMS[@]}"
        else
            conditional_echo "Release Notes:"
            conditional_echo "$RELEASENOTES"
        fi
    else
        conditional_echo "-- ERROR: TAG $NEWTAG already exists."
        exit 1
    fi
else
    conditional_echo "-- ERROR: Commit already tagged as a release. ($LASTCOMMIT)"
    exit 1
fi

# Switch to back to original branch
if [ $CURRENT_BRANCH != "$BRANCH" ]; then
    conditional_echo "- Switching back to $CURRENT_BRANCH branch. (restoring local changes)"
    git checkout "$CURRENT_BRANCH" "${GITPARAMS[@]}"
    # remove the stash
    git stash pop "${GITPARAMS[@]}"
fi

exit 0
