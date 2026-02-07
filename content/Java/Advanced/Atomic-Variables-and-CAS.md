---
title: Atomic Variables & CAS
---

# 1. The Problem: Compound Actions

We learned that `volatile` ensures visibility (reads are correct), but it does not ensure atomicity for compound actions.
