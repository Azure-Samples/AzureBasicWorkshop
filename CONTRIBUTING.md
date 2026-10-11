# Contributing to Azure Basic Workshop

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

 - [Code of Conduct](#coc)
 - [Issues and Bugs](#issue)
 - [Feature Requests](#feature)
 - [Submission Guidelines](#submit)

## <a name="coc"></a> Code of Conduct
Help us keep this project open and inclusive. Please read and follow our [Code of Conduct](https://opensource.microsoft.com/codeofconduct/).

## <a name="issue"></a> Found an Issue?
If you find a bug in the source code or a mistake in the documentation, you can help us by
[submitting an issue](#submit-issue) to the GitHub Repository. Even better, you can
[submit a Pull Request](#submit-pr) with a fix.

## <a name="feature"></a> Want a Feature?
You can *request* a new feature by [submitting an issue](#submit-issue) to the GitHub
Repository. If you would like to *implement* a new feature, please submit an issue with
a proposal for your work first, to be sure that we can use it.

* **Small Features** can be crafted and directly [submitted as a Pull Request](#submit-pr).

## <a name="submit"></a> Submission Guidelines

### <a name="submit-issue"></a> Submitting an Issue
Before you submit an issue, search the archive, maybe your question was already answered.

If your issue appears to be a bug, and hasn't been reported, open a new issue.
Help us to maximize the effort we can spend fixing issues and adding new
features, by not reporting duplicate issues.  Providing the following information will increase the
chances of your issue being dealt with quickly:

* **Overview of the Issue** - if an error is being thrown a non-minified stack trace helps
* **Version** - what version is affected (e.g. 0.1.2)
* **Motivation for or Use Case** - explain what are you trying to do and why the current behavior is a bug for you
* **Browsers and Operating System** - is this a problem with all browsers?
* **Reproduce the Error** - provide a live example or a unambiguous set of steps
* **Related Issues** - has a similar issue been reported before?
* **Suggest a Fix** - if you can't fix the bug yourself, perhaps you can point to what might be
  causing the problem (line of code or commit)

You can file new issues in the [upstream Azure Basic Workshop issue tracker](https://github.com/Azure-Samples/AzureBasicWorkshop/issues/new). Issues are not enabled on the current fork. For security vulnerabilities, use [SECURITY.md](SECURITY.md) instead of a public issue. See [SUPPORT.md](SUPPORT.md) for the support status.

### <a name="submit-pr"></a> Submitting a Pull Request (PR)
Before you submit your Pull Request (PR) consider the following guidelines:

* Search the [upstream pull requests](https://github.com/Azure-Samples/AzureBasicWorkshop/pulls) for an open or closed PR
  that relates to your submission. You don't want to duplicate effort.

* Make your changes in a dedicated branch of your fork. Keep unrelated local changes intact.

* Commit your changes using a descriptive commit message
* Push your fork to GitHub:
* In GitHub, create a pull request against the intended repository's `main` branch. Use the repository's pull request template.
* If we suggest changes then:
  * Make the required updates.
  * Add follow-up commits to your pull request branch. Do not rewrite shared history without the maintainer's approval.

## Workshop validation

This workshop uses Azure Portal and Bastion. It does not require `npm install`, notebooks, or a devcontainer.

Run all nine commands in [AGENTS.md](AGENTS.md#검증-하네스) from the repository root using Git Bash or a Bash-based environment. The checks require gitleaks 8.19 or newer, Python 3.10 or newer, and PyYAML. Record each command, its exit status, and its actual result; a final `PASS` does not override an earlier error.

Keep validation logs and decision records under the Git-ignored `local-only` directory. Share only redacted summaries in a pull request. Preserve lab commands, images, navigation, and the existing MIT copyright notice. Document any changes to lab behavior separately.

Automated checks do not replace a complete portal run or a review of screenshots for private information. Leave `validated_on` blank until a person completes the workshop and provides the date. Do not invent durations, cost estimates, or successful execution results. List unresolved TODOs, including licensing and support policy decisions, in the pull request.

That's it! Thank you for your contribution!
