---
title: Database Indexes Simplified
layout: post
---

Database indexes seem to trip up a lot of developers. They seem to either index everything, nothing, or just columns that refer to other tables (ie: user_id in a posts table). I've thought about it for a while and finally decided to write up some thoughts that will hopefully be helpful.

Comparing something someone understands with something similar that they do not understand is a great way to teach. Everyone understands what filing cabinets are and how they work. Filing cabinets are old school databases, so let's use filing cabinets, folders and paper as a way to learn a bit more about database indexes.

## The Fastest Way to File

Just for kicks, lets refer to filing a paper as a **write** (insert in db terms) and finding a previously filed paper a **read** (select in db terms). The fastest way to file a paper (write) is to just throw it in the cabinet. This makes it super easy to file papers. Open drawer, throw in paper, and close the drawer.

Lets say you do this for several years for every piece of paper you need to keep -- receipts, tax documents, car titles, mortgage papers, etc. Out of nowhere, your walkman stops working. You just bought it, like a month ago, and you put the receipt in your filing cabinet, as a responsible music listener should. All you need to get a replacement walkman is the receipt, so you head to your filing cabinet.

Finding your receipt, even though it was a recent purchase, is really hard. Your only option is to start at the top and work your way through the stack of papers you have filed over the past few years until you find the receipt for your walkman.

This incident with your walkman makes you realize throwing all your papers into one drawer in a file cabinet is great for filing (writes), but terrible for finding (reads).
