---
title: "Things to Do to Keep Project Grows Well"
date: 2016-02-28 15:00:00
layout: post
categories: thoughts, notes
---

It's easy to start a project, but hard to keep it grows well. Here are some
lessons I learnt from my mostly failed expriences:

## 1. Write Unit Tests

Why: you usually can make thing right at the very moment when you create it,
but you can't keep them right as time changes. Unit tests is the safe net of
your project.

How: you don't need to hold a 100% coverage record all the time, because
real world problem may be untestable. But 100% coverage for testable is rather
easy.


### 2. Isolate Your Code with Separation of Concerns Pattern

Why: we're making real product here, so we all encounter with business logic.
Mixing up business logic with non-business logic will screw your code. SoC can
prevent you from breaking the old behaviours when changing the code.

How: just as mention above, isolate business related code from the normal,
find ways to test these logic.


### 3. Use a Code Linter (or Formatter)

Why: as project evolves, your team grows too. So rather than mess up code with
each other, use a linter to automate the thing.

How: there [are][jshint] [many][gofmt] [tools][flake8] [for][editorconfig] linting.

[jshint]: http://jshint.com/
[gofmt]: https://golang.org/cmd/gofmt/
[flake8]: https://pypi.python.org/pypi/flake8
[editorconfig]: http://editorconfig.org


### 4. Make Your Code Profile-able

Why: your code may act well in low traffic, but you have to prepare for scale.
In order to figure what block your project, you should investigate with statistic.

How: [log][log] your program, add metric inside your program.

[log]: http://12factor.net/logs
