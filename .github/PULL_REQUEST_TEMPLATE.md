## Purpose
<!-- Describe the intention of the changes being proposed. What problem does it solve or functionality does it add? -->
* ...

## Does this introduce a breaking change?
<!-- Mark one with an "x". -->
```
[ ] Yes
[ ] No
```

## Pull Request Type
What kind of change does this Pull Request introduce?

<!-- Please check the one that applies to this PR using "x". -->
```
[ ] Bugfix
[ ] Feature
[ ] Code style update (formatting, local variables)
[ ] Refactoring (no functional changes, no api changes)
[ ] Documentation content changes
[ ] Other... Please describe:
```

## How to Test
*  Get the code

```
git clone https://github.com/kyungtaak/AzureBasicWorkshop.git
cd AzureBasicWorkshop
git checkout [branch-name]
```

* Test the code
<!-- Add steps to run the tests suite and/or manually test -->
Run all nine workshop checks from [AGENTS.md](../AGENTS.md#검증-하네스) in Git Bash. Include each exact command, its exit status, and its result. If a check was not run, explain why. Do not attach unredacted logs or private resource identifiers.

Record portal E2E validation separately: who completed it, the completion date, and any unresolved steps. Leave `validated_on` blank if a complete run has not been performed.

## What to Check
Verify that the following are valid
- [ ] Lab procedures and commands are preserved, or intentional behavior changes are explained.
- [ ] Relative links, images, and previous/next navigation work.
- [ ] All nine checks have results; failures and unexecuted checks are listed.
- [ ] Logs and screenshots do not disclose credentials or private information.
- [ ] Durations and E2E dates are supported by actual measurements.
- [ ] Remaining TODOs, licensing decisions, and support policy holds are listed.

## Other Information
<!-- Add any other helpful information that may be needed here. -->