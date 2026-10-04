---
layout: post
title: "The Vibe Cycle"
description: "mark-comms is a free browser tool for commenting on markdown, rebuilt after Sidemarkr went offline. What happens when throwaway software loses support?"
date: 2026-10-05
---

Working with agents, one of the main things you spend time on is a lot of markdown.
Markdown drives formats like `AGENTS.md`, `CLAUDE.md` for basic customisation, files like `tasks.md` if you're using Spec driven development. 

One of our favourite tools for adding comments to a markdown file was `sidemarkr.com` which as of late september 2026 now gives a GoDaddy domain hold page. What was excellent about sidemarkr was the ability to upload a markdown file from any device, highlight the text you're unhappy with and pull it back down again. The flow is inherently familiar to anyone who's done a PR Review in hosted git where you can just select text and add commentary. A full git flow is excellent for tracking changes overtime, but for many just trying to review a file that will likely be deleted in a few hours, the setup would be overkill. 

In the absence of knowing when sidemarkr will return, with many `.md` files locally bearing the format, we've created `mark-comms` as shorthand for markdown-comments. Naming likely overlaps with `marcomms` for those in marketing, but if a marketer accidentally ends up finding this, they're welcome to send us a better name. 

The tool is live now at <https://www.waratahlabs.com/project/mark-comms/> following the pattern from sidemarkr. Your draft never leaves the browser and is persisted only in browser state. The only thing sent out is a page-view count, so we can see which pages get attention. 

Whether anyone else finds this useful or utilises it we're not sure at this stage, but we do feel like as coding agents improve, the ability to clone functionally based on limited information will lead to more and more "temporary" apps being spun up. Events already generate throwaway apps for "Some Conference 2025" where the schedule and floorplan are hardcoded and then "Some Conference 2026" is pushed to the app stores to replace it. The pattern therefore is already established but now continuing to branch out. The lingering question we have for the "vibe cycle" of apps is at what point does the software become abandonware? If it was never really supported to begin with, just a web app and a domain name thrown into the world, with no expectation of support.

For mark-comms, our site is publicly hosted at the time of writing on github pages, allowing anyone to pick it up and pivot if we ever decide to stop hosting. Potentially, that's the future for some of this abandonware, to create a github repo, archive it as read-only and then the next developer can pick it up if there's anything useful.

In our case, sidemarkr's format captured with 21 comments on a previous writeup was sufficient to recreate the app in about 20 minutes with backwards compatibility, but reverse engineering a markdown editor is not the most complex software.

Perhaps the vibe cycle will at one point just be the accepted norm for software to be single use, but creating a minimum expectation of source before dropping support feels like the right move.
