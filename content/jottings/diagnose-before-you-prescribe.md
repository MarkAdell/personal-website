---
title: Diagnose Before You Prescribe
date: 2024-04-24
tags: ["programming", "software engineering"]
ShowToc: false
---

Imagine you go to the doctor feeling unwell, and without examining you, they immediately start writing a prescription. This is exactly what some software engineers occasionally do.

When we are faced with an issue in a particular component of our system, especially if it's a performance issue that is challenging to debug, it's not uncommon to find ourselves trying to think of ways to fix it. We may even feel urged to redesign the entire component in a way that we believe is more efficient than the current one, and spare ourselves the burden of debugging.

This will always be the wrong approach. Rushing to solutions without fully understanding the problem can lead to wasted time because there is a high chance that the solution doesn't address the problem. Even if the solution works, you will never be sure whether it really solves the problem or if it's just a coincidence.

Therefore, it's important to resist the urge to jump to solutions and instead, invest time in thoroughly understanding the problem. When faced with a performance issue, the first thing you should do is try to understand the root cause. This involves debugging, examining the logs and relevant performance metrics, etc. Only then should you start looking for a solution, which might be a very simple update that requires minimal effort.
