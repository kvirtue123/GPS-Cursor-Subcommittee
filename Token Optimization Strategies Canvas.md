Token Optimization Strategies
:dart: What is this?

This is the working canvas for the Token Optimization workstream — a subgroup of the GPS Cursor Subcommittee. The goal is to identify, test, and document strategies that reduce token consumption without sacrificing quality or speed.

How to contribute: Add your findings, tips, and questions directly to this canvas! Drop notes under the relevant section, add your name/handle next to contributions, and flag anything that needs group discussion with a :speech_balloon: emoji.


:busts_in_silhouette: Workstream Lead

TBD — interested? Reach out to @Keegan Virtue @Steve Shin 

:bulb: Focus Areas

1. Preventing Duplicate Work

Creating a searchable repository for completed assets (LWCs, budget grids, etc.) so SEs can find and reuse existing components before building from scratch.
Open Questions / Ideas — add yours below:

* Where should the searchable repo live? (GitHub? Google Drive? Slack canvas?)
  → Answered: GitHub repo (cursor-best-practices) is the install source. Slack canvases are the quick-lookup layer. Google Drive is for binaries/recordings. The repo README explains the split.
* What metadata makes assets findable?
  → Each catalog entry has: ID, what it is, when to use, when NOT to use, dependencies, and provenance. The R1 rule encodes a COPY/ADAPT/SKIP registry per project so the agent itself knows what's reusable before it starts building.
* Add your ideas here...


2. Workflow & Configuration Best Practices

Guidelines for managing tokens effectively within day-to-day SE workflows.
Focus areas identified so far:

* Using the correct model for the task (e.g., don't use a large model for simple completions)
  → P26 (Scoped UI-polish prompt pack) formalizes this: each item in a batch names the model to use. Reserve frontier models for architecture/complex debugging; use smaller models for isolated CSS tweaks, boilerplate, simple edits.
* Managing context window size appropriately
  → Three catalog assets attack this directly:
     R7 (Context-scoping ignore file) — removes dead/superseded code from the agent's index entirely so it never loads into context.
     R2 (Doc-sync rule, glob-scoped) — loads documentation rules only when relevant files are touched, not on every request.
     R3 (Goals & Doc-Map rule) — replaces "go read everything" with "@-reference the exact doc for this question."
* Properly utilizing sub-agents to scope work
  → P4 (Phased delivery structure) — breaks work into self-contained phases so each agent session has a narrow, well-defined scope.
     P23 (Ask→Plan→Agent lifecycle) — gather context in Ask mode first, approve a plan, then let Agent implement. Prevents the expensive "agent went off in the wrong direction" cycle.

Contribute your tips below:

* Add a workflow tip here (with your handle so we can follow up)
* Add a configuration best practice here...


3. Automated Token Management Tools

Exploring third-party skill repositories and tooling that handle token optimization automatically.
Tools under consideration:

Tool
	What it Does
	Status
	Notes

GSD
	Auto-manages model spinning/switching in workflows
	Exploring
	Add notes

Superpowers
	Token optimization via sub-agent orchestration
	Exploring
	Add notes

GStack
	Handles model context within workflows
	Exploring
	Add notes

Add a tool...
	
	
	



:pencil: Research 

Drop experiments, results, and observations here — include your handle and a date.
Feel free to add!

June 2026 · @Keegan Virtue
Across 8 GPS projects, the most impactful token savings came from three places in order: (1) glob-scoped rules that don't load unless relevant files are touched — R2 and R8 together prevented large rule payloads from loading on unrelated requests; (2) ignore files (R7) that removed dead/legacy code from the agent's index — one project eliminated an entire deprecated agent version from context, cutting retrieval noise significantly; (3) the P23 Ask→Plan→Agent workflow — gathering context before implementation prevented the expensive "agent built the wrong thing, now rewrite it" loops that burn the most tokens. Catalog entries for all of these are in the GitHub repo.

:link: Related Resources

Resource
	Link

GPS Cursor Subcommittee Hub
	Open canvas

GitHub Repo (catalog + token optimization rules/patterns)
	[cursor-best-practices](https://github.com/kvirtue123/GPS-Cursor-Subcommittee) → see R2, R3, R7, P4, P23, P26

Google Drive
	Open folder


Last updated: May 15th · Workstream: Token Optimization · Parent: GPS Cursor Subcommittee
