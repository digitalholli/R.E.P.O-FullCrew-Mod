# Full Crew testing and bug reports

Help test the Full Crew mod for R.E.P.O. You do not need programming or professional testing experience.

## Start here

1. Read the [player guide](PLAYER-GUIDE.md) for installation and settings. Obtain the test build from the maintainer; this repository currently contains documentation, not a downloadable build.
2. Follow the quick pass in the [testing guide](TESTING.md), which includes detailed log-file instructions.
3. Use the [mod compatibility checklist](MOD-COMPATIBILITY.md) when testing with other mods.
4. Open **Issues → New issue** and choose **UAT test results** or **Bug report**. Check existing issues first to avoid duplicates.

You can also copy [TEST-REPORT.md](TEST-REPORT.md) or [ISSUE-REPORT.md](ISSUE-REPORT.md) and fill it in outside GitHub. A GitHub account is needed to submit an issue directly.

## Reporting results

Record the tested Full Crew version, game version, mod list, host settings, actual player count and test IDs. Say what you expected and what happened. Not sure and Could not test are valid answers. A completed test plan is not evidence that the tests passed.

Save the host log before relaunching; include a guest log for guest-only problems. This repository and its issues are public. Review attachments for personal information and remove secrets before posting. Do not upload game assemblies, account credentials or private source code.

## Tracking work

Use one bug issue per distinct problem and one UAT results issue per testing session. Link related bugs from your session report. Maintainers can acknowledge, investigate and request a retest in the issue thread. Closing a bug should reference the fix version and the retest evidence, or explain another resolution.

The source code is maintained separately in a private repository. Testers do not need access to it.
