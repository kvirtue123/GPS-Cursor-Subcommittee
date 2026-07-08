Skills Best Practices
:dart: What is this?

This workstream focuses on establishing best practices for Cursor AI skill usage across the GPS SE org — covering how we educate users, standardize skill files, and determine who owns and governs skills long-term.

How to contribute: Drop findings, prompts, or lessons learned under the relevant section below. Include your handle and date. Flag discussion items with :speech_balloon: and tag a co-lead if it needs follow-up.


:busts_in_silhouette: Workstream Lead

TBD — interested? Reach out to @Keegan Virtue or @Steve Shin

:bulb: Focus Areas (feel free to add!)


1. Standardizing Workflows with Skill Files

To bring order to rapidly evolving tool usage, the group proposed using common "skill files" as a standardized gateway and baseline for workflows. These files could also be configured to auto-search repositories for existing content when a user starts a new idea.
Open Questions / Ideas — add yours below:

* How do we configure skills to auto-search the GitHub repo for existing content?
  → The R1 (Reuse Registry & Architecture) rule encodes a COPY/ADAPT/SKIP registry directly in the project. When the agent starts a session it reads what exists and what to reuse before building anything new. Template in the repo: rules/templates/agent-script-architecture.mdc
* Should skill files be versioned? Who approves changes?
  → The GitHub repo is the source of truth. Skills live in skills/<name>/SKILL.md with a README. Changes go through co-lead review (PR or direct triage). The BEST-PRACTICES.md catalog entry is the "approved" record — if it has a card, it's been vetted.
* Add your ideas here...


2. Centralizing Skill Ownership

The current landscape is a "wild west" — multiple OUs are building and hoarding their own separate skills with no shared governance. The committee identified an open action item: determine who officially owns and manages skills going forward (COE or another centralized owner).
Open Questions / Ideas — add yours below:

* What does a skill governance model look like in practice? Review cadence? Approval gates?
  → Working model (from 8 projects): each asset needs (a) a real project it came from, (b) a "when to use / when NOT to use," (c) a co-lead-assigned ID. That's the bar. Quarterly review to check for drift/staleness. See BEST-PRACTICES.md for the full framework.
* How do we surface and consolidate skills that OUs have already built in silos?
  → This catalog IS that consolidation — 8 GPS projects, 45 assets extracted and centralized. The contribution path: post in the Feed tagged [Rule], [Skill], or [Pattern] → co-lead assigns ID and writes the catalog card → merged to repo.
* Add your ideas here...


:pencil: Research & Additional Findings Log

Drop experiments, results, and observations here — include your handle and a date.

Date
	Contributor
	Finding / Observation

June 2026
	@Keegan Virtue
	Cataloged 45 reusable assets (13 Rules, 6 Skills, 26 Patterns) extracted from 8 GPS delivery projects. Every asset has a "when to use / when NOT to use," provenance, and a canonical example. The catalog directly answers both focus areas above — standardization framework exists, consolidation is done for GPS projects to date. Full catalog: BEST-PRACTICES.md in the GitHub repo. Quick-lookup cards: Skills Best Practices Canvas (this canvas). Getting started in 3 steps: see the repo README.

	
	Add a finding...



:link: Related Resources

Resource
	Link

GPS Cursor Subcommittee Hub
	Open canvas

Subcommittee Channel
	#gps-ae-club-cursor-subcommittee

Google Drive
	Session recordings & decks

GitHub Repo (catalog + installable templates)
	[cursor-best-practices](https://github.com/kvirtue123/GPS-Cursor-Subcommittee) → start at README.md


Last updated: May 15th · Workstream: Skills Best Practices · Parent: GPS Cursor Subcommittee
