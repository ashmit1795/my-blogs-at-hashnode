---
title: "Git Merge vs Git Rebase — What's the Difference and When to Use Which"
seoTitle: "Git Merge vs Git Rebase — Differences & When to Use Which"
seoDescription: "Learn the difference between git merge and rebase with diagrams and real commands. Know when to use each — and when rebase can go wrong."
datePublished: 2026-04-21T16:32:12.472Z
cuid: cmo8udgvp00al1qocd1j77se7
slug: git-merge-vs-git-rebase-what-s-the-difference-and-when-to-use-which
cover: https://cdn.hashnode.com/uploads/covers/69b5b16d1f4d4527ed9008ad/58e97624-4b34-4c74-bdb1-016cca697790.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69b5b16d1f4d4527ed9008ad/0d8b8785-ab6a-415d-b978-80a94eb96ace.png
tags: github, programming, version-control, git, devops, backend-development

---

> You've been working on a feature branch. Meanwhile, `main` has moved ahead. Now you need to bring those changes together. You have two options: **merge** or **rebase**. Most developers pick one without really understanding the other. This article will change that.

* * *

## Table of Contents

1.  [The Situation — Setting the Stage](#1-the-situation--setting-the-stage)
    
2.  [Git Merge — Keeping History as It Happened](#2-git-merge--keeping-history-as-it-happened)
    
3.  [Git Rebase — Rewriting History for a Clean Line](#3-git-rebase--rewriting-history-for-a-clean-line)
    
4.  [Merge vs Rebase — Side by Side](#4-merge-vs-rebase--side-by-side)
    
5.  [When to Use Merge](#5-when-to-use-merge)
    
6.  [When to Use Rebase](#6-when-to-use-rebase)
    
7.  [The Golden Rule of Rebase](#7-the-golden-rule-of-rebase)
    
8.  [Summary and Conclusion](#8-summary-and-conclusion)
    

* * *

## 1\. The Situation — Setting the Stage

Let's say your repository looks like this:

```plaintext
main:    A --- B --- C
                \
feature:         D --- E
```

You branched off `main` at commit `B` to work on a new feature. While you were working, your teammates pushed new commits to `main` — so `main` is now at `C`, but your `feature` branch only knows about `A` and `B`.

Now you want to **bring the feature branch into main**. You have two ways to do this:

*   `git merge`
    
*   `git rebase`
    

Both accomplish the goal — but they tell *very different stories* in your commit history.

* * *

## 2\. Git Merge — Keeping History as It Happened

### What it does

`git merge` takes the two branch histories and **joins them together**, creating a brand new commit called a **merge commit**. It keeps the full record of everything — when branches diverged, when they came back together, and who did what.

Think of it as: *"Keep everything as it happened. Just join the branches."*

### How it looks

Before merge:

```plaintext
main:    C1(ABC) --- C2(AB'C) --- C3(AB'C')
                \
feature:         F1(A'BC) --- F2(A'BCD)
```

After `git merge main` (from the feature branch):

```plaintext
main:    C1 --- C2 --- C3 -----
                \               \
feature:         F1 --- F2 ----- NC (Merge Commit)
                                 (A'B'C'D)
```

Git creates a **new merge commit (NC)** that has **two parents** — the tip of `main` (`C3`) and the tip of `feature` (`F2`). This merge commit contains all the changes from both branches combined.

### Commands

```bash
# Step 1: Make sure you're on the feature branch
git checkout feature

# Step 2: Merge main INTO feature
git merge main

# This creates the merge commit (NC) automatically
```

### What git log looks like after merge

```plaintext
NC   ← merge commit (A'B'C'D0)
C3
F2
F1
C2
C1
```

The log shows the full branching history — you can see exactly when and where things diverged.

### Key characteristics of git merge

*   ✅ **Keeps original history** — nothing is rewritten, everything is preserved exactly as it happened
    
*   ✅ **History is 100% honest** — shows when branches were created and merged
    
*   ✅ **Safe for shared/public branches** — you're never rewriting commits others may have based work on
    
*   ✅ **Shows when branches were merged** — useful context for teams
    
*   ⚠️ **Creates a merge commit** — adds an extra commit to the log
    
*   ⚠️ **Log can look non-linear** — with many branches, history can look like a web of lines
    

* * *

## 3\. Git Rebase — Rewriting History for a Clean Line

### What it does

`git rebase` takes your feature branch commits and **replays them on top of the latest main**. Instead of creating a merge commit, it moves your commits so it looks like you *started working from the latest version of main all along*.

Think of it as: *"Make it look like I started from the latest main."*

### How it looks

Before rebase:

```plaintext
main:    C1(ABC) --- C2(AB'C) --- C3(AB'C')
                \
feature:         F1(A'BC) --- F2(A'BCD)
```

After `git rebase main` (from the feature branch):

```plaintext
main:    C1 --- C2 --- C3
                            \
feature (rebased):           F1'(A'B'C') --- F2'(A'B'C'D)
```

Your feature commits (`F1`, `F2`) are **replayed on top of** `C3`, creating brand new commits (`F1'`, `F2'`). The original `F1` and `F2` are gone — replaced by their rewritten versions. The history now looks perfectly linear, as if you had branched off `C3` from the start.

### Commands

```bash
# Step 1: Make sure you're on the feature branch
git checkout feature

# Step 2: Rebase onto main
git rebase main

# Git will replay your commits one by one on top of main
```

### What git log looks like after rebase (on main)

```plaintext
F2'  ← new commit (A'B'C'D)
F1'  ← new commit (A'B'C')
C3
C2
C1
```

Clean, linear, no merge commit. It reads like a straight line from start to finish.

### Key characteristics of git rebase

*   ✅ **History is linear and clean** — easy to read and navigate
    
*   ✅ **No merge commit** — log stays uncluttered
    
*   ✅ **Looks like you started from the latest main** — great before opening a Pull Request
    
*   ⚠️ **Rewrites history** — commits get new SHA hashes (F1 → F1', F2 → F2')
    
*   ⚠️ **Risky if misused on shared branches** — rewriting commits others are using causes serious problems
    
*   ⚠️ **Loses the "when did branches diverge" information** — history is cleaned up, not preserved
    

* * *

## 4\. Merge vs Rebase — Side by Side

|  | Git Merge | Git Rebase |
| --- | --- | --- |
| **History** | Keeps original history | Rewrites history |
| **Merge Commit** | Creates a merge commit | No merge commit |
| **Log appearance** | Looks like branches (non-linear) | Looks linear (clean) |
| **Safety** | Safe for teams | Risky if misused |
| **Use case** | Integrating completed features | Cleaning up before PR |
| **Conflict handling** | Resolve once in the merge commit | Resolve per commit during replay |
| **Transparency** | Shows full branching story | Hides branching story |

* * *

## 5\. When to Use Merge

Use `git merge` when:

*   **Integrating a completed feature into** `main` — this is the standard way to merge a Pull Request
    
*   **Working in a team on shared branches** — merge is safe because it doesn't rewrite any existing commits
    
*   **You want to preserve the full history** — stakeholders or reviewers need to see when and where things happened
    
*   **You're merging long-running branches** — branches with a lot of history are safer to merge than rebase
    

```bash
# Standard team workflow — merge feature into main
git checkout main
git merge feature
```

* * *

## 6\. When to Use Rebase

Use `git rebase` when:

*   **Cleaning up your feature branch before opening a Pull Request** — makes your commits easier to review
    
*   **Your branch has fallen behind** `main` — rebase brings it up to date without a merge commit
    
*   **Working on a personal/local branch that hasn't been shared** — safe to rewrite since no one else has based work on it
    
*   **You want a clean, readable git log** — especially important for open source projects
    

```bash
# Clean up your feature branch before a PR
git checkout feature
git rebase main

# Then open your Pull Request
```

* * *

## 7\. The Golden Rule of Rebase

> ⚠️ **Never rebase a branch that other people are working on.**

This is the most important rule in all of Git rebase.

Here's why it's dangerous: When you rebase, Git creates **brand new commits** with new SHA hashes to replace the old ones. If a teammate has already pulled your old commits and based new work on them, their history now conflicts with yours. When they try to push or pull, Git gets confused because it sees two different versions of what should be the "same" commits.

The result is a messy, painful conflict that's hard to untangle and can genuinely corrupt shared history.

**Safe rule of thumb:**

| Branch type | Use |
| --- | --- |
| Your local feature branch (not pushed yet) | ✅ Rebase freely |
| Your feature branch (pushed, but only you use it) | ✅ Rebase with `git push --force-with-lease` |
| Shared branch (`main`, `develop`, `staging`) | ❌ Never rebase |

* * *

## 8\. Summary and Conclusion

Here's the mental model to carry with you:

**Use merge** when you're integrating finished work. It's honest, safe, and shows the full story of your history.

**Use rebase** when you're cleaning up *your own* work before sharing it. It produces a cleaner, linear history that's easier for reviewers to follow.

They're not enemies — they're different tools for different moments in your workflow. A common pattern in professional teams looks like this:

```bash
# 1. While developing, keep your feature branch up to date via rebase
git checkout feature
git rebase main          # Stay current, clean history

# 2. When the feature is done and ready — merge via Pull Request
git checkout main
git merge feature        # Integrate the finished work safely
```

This gives you the best of both worlds: a clean feature branch that's easy to review, and a main branch that safely preserves the full integration history.

* * *

### The one-line summary

> **Merge preserves history. Rebase rewrites it. Use merge to integrate. Use rebase to clean up. Never rebase shared branches.**

* * *

*Found this helpful? Share it with a developer who's been avoiding rebase out of fear — it's one of those things that clicks once and then you can't unsee it.* 🚀