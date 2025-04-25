# Core Devs Community Call 36
### Meeting Date/Time: April 24th, 2025, 6:00-6:30 UTC
### Meeting Duration: 30 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/122)
### Agenda
* The GreatVoyage-v4.8.0(Kant) release and mainnet upgrade process
* Hard fork proposals plan

## Details

* Jake

  Hello everyone, welcome to Core Devs Community Call 36. Today’s agenda focuses on discussing the release of 4.8.0, the procedural work for the mainnet upgrade, and finalizing the launch timelines and order for three newly introduced proposals.

  Let’s dive in. Raymond, could you start by sharing the preparation status for the 4.8.0 release?

* Raymond

  The next version is 4.8.0. All related features have been fully developed and merged into the master branch. Key features include functionality related to the Ethereum Cancun upgrade, a new version of the event service, and several other enhancements. The testnet went live on March 17, and three associated proposals have already been activated. Mainnet gray testing began on March 26, and the TIPs related to 4.8.0 have all been merged into the master branch with their statuses marked as "final." That’s the current update.

* Murphy

  So, can we confirm a specific release date for 4.8.0 now?

* Raymond

  The tentative date is next Tuesday, April 29.

* Murphy

  Any issues with April 29 as the release date?

* Jake

  If we stick to this date, this will be the last collective meeting to discuss the release before launch.

  Assuming no objections, Murphy, please proceed.

* Murphy

  Alright, I’ll share my screen.

  Following Raymond’s update on the 4.8.0 development readiness, since we’ve tentatively set the release date as April 29, the community team will now outline the process for driving the mainnet upgrade.

  The estimated total duration for the mainnet upgrade is approximately 30 days, from April 29, 2025, to May 31, 2025. Here are the key steps:

    * Within 1–2 days after the version is released on GitHub (approximately May 1–2), the community team will notify community channels (Telegram, Discord, etc.), Super Representatives (SRs), and key project partners like exchanges and wallets to initiate upgrades.

    * Weekly follow-ups will be conducted with these key partners to track their upgrade progress, maintain records, and ensure cooperation, aiming for full completion by May 31.

    * At developer meetings near the deadline, the community team will share upgrade progress. If any critical partners haven’t upgraded for any reason, we’ll provide timely feedback.

  That’s the plan for the mainnet upgrade. Any questions? If not, let’s move on.

* Murphy

  Next, Raymond will introduce the three new proposals, and we’ll discuss their launch timelines, order, and how the community should coordinate.

* Raymond

  Version 4.8.0 introduces three proposals. Two are related to the Virtual Machine: One controls the activation of instructions for transient storage and memory copy; and the other controls two blob-related instructions. The third proposal pertains to consensus validation, which will be activated at network-layer.

* Jake

  Is there a specific order required for launching these proposals?

* Raymond

  They are entirely independent and have no interdependencies.

* Murphy

  So we can initiate discussions for all three proposals simultaneously, then determine the voting dates for each based on the discussion progress. Is that acceptable?

* Raymond

  Yes, that works.

* Murphy

  I have a question about TIP-697, which switches all arithmetic libraries to `strict.Math`. I recall that in the previous version, this was activated via a hard fork, providing backward compatibility. Since this switch is forward-compatible, we don’t need a new proposal for it, right?

* Boson

  This is bundled with #87.

* Murphy

  Alright, moving on if there are no issues here. Finally, let’s sync on the upcoming proposal launch process.

  First, we need to confirm whether these three proposals require adaptation work from external projects. For example, when the dynamic energy model was activated, wallets with too low feeLimit settings caused transaction failures, requiring advance notice for adaptation. Developers should assess whether these proposals will impact project partners. If so, we need at least 30 days to notify them for adaptation. Please reach out to us promptly if adaptation is needed.

  Second, regarding the voting timeline:

    * If the mainnet upgrade progresses smoothly and completes by May 31, we’ll schedule developers to publish the proposals on GitHub between June 1–5, 2025 (including draft reviews and validation).

    * From June 7–28, 2025 (2–3 weeks), we’ll invite community developers and project partners to participate in discussions. During this period, we’ll update progress at a developer meeting between June 14–21 and collectively determine the exact voting start dates based on discussion outcomes.

  That’s the overview. Any questions?

* Jake

  Murphy’s timeline is based on the April 29 release. If the release is delayed or rescheduled, all subsequent milestones will shift accordingly. Please raise any concerns or changes to the release status now.

* Murphy

  No issues? We’ll proceed then—feel free to contact us later with questions.

* Jake

  Great. I’ll finalize the meeting minutes shortly. For further discussions, let’s use GitHub.

  Thanks everyone for joining.

* Murphy

  Thanks for attending, goodbye!

### Attendance
* Boson
* Aaron
* Wayne
* Blade
* Allen
* Gordan
* Leem
* Mia
* Sunny
* Raymond
* Sunny Sun
* Elvis
* Daniel
* Lucas
* Federico
* Murphy
* Jake