# Nobuyuki Kobayashi's GitHub README Style

This document defines the principles for writing and maintaining GitHub README files in the style preferred by Nobuyuki Kobayashi.

When creating or revising a README, do not mechanically apply a conventional README template. First inspect the actual repository and consider what users need to do with it.

## Most important principle

**Do not decide the README structure first. Organize the README around what users actually need to do to accomplish their goal with the repository.**

A README should be optimized for the person who is trying to use the repository, not for conformity to a standard documentation structure.

Do not automatically use a structure such as:

```text
Overview
Features
Requirements
Installation
Configuration
Usage
Contributing
License
```

simply because it looks like a conventional README.

Instead, first ask:

**What does a first-time user of this repository actually want to do?**

Build the README around the answer.

## Basic philosophy

A README is primarily a practical document for using the repository.

In particular, it should make clear:

- what the repository does;
- how to actually use it;
- what commands should be executed;
- in what order they should be executed;
- which URLs, paths, ports, versions, and other concrete values are required;
- why a particular setup is used, when that information is important.

Neither a long README nor a short README is inherently better.

**Write what is necessary, concretely. Do not add what is unnecessary.**

## Explain the purpose first

Immediately after the repository title, briefly explain what the repository does.

For a simple utility, one line may be enough.

Do not add a long introduction merely because a README is expected to have one.

For a more complex system, provide enough explanation for the user to understand the instructions that follow.

## Follow the user's workflow

Prefer the order in which users actually perform their work over classification by documentation category.

For example, a simple command-line program may naturally follow:

```text
What the program does
↓
Build
↓
Run
↓
Input
↓
Output
↓
Details
```

A container repository may follow:

```text
What the container provides
↓
Pull / Download
↓
Run
↓
Use
↓
Details
```

A DAQ system may instead require:

```text
What the system connects
↓
Data flow
↓
Setup
↓
Start components
↓
Start DAQ
↓
Check operation
↓
Details
```

These are examples, not templates.

Choose the structure appropriate for the repository.

## Show the actual operations

Do not stop at conceptual explanations. Show what the user actually needs to do.

For example:

```bash
git clone https://github.com/...
cd ...
make
./program input.dat
```

When multiple steps are required, show them in the order in which the user should perform them.

A user should not finish reading the README and still have to ask: **"So, what command do I actually run?"**

## Make commands copy-pasteable

Whenever practical, provide complete commands that can be copied and executed directly.

For example, if a SIF image is distributed through GitHub Releases, prefer:

```bash
curl -L -O \
  https://github.com/.../releases/download/latest/example.sif
```

over telling the user only to download the SIF file from GitHub Releases.

Do not make users search for a URL that can be given directly.

The same principle applies to clone URLs, image names, filenames, command-line arguments, and other information required to perform the operation.

## Do not make users guess

When a concrete value is known and required for normal use, write it explicitly.

Examples include repository names, container image names, image tags, software versions, architectures, filenames, directories and paths, port numbers, host names, environment variables, input and output files, and hardware or FEM IDs.

Avoid placeholders such as `appropriate path`, `your port`, or `some directory` when the actual value is known.

Something that is obvious to the developer may not be obvious to a new user.

## Place Quick Start or Usage where it helps the user

For a simple utility or container, `How to use`, `Usage`, or `Quick start` should usually appear early so that a user can begin using the software quickly.

This is not a fixed rule.

For a complex DAQ system, interface, framework plugin, or other multi-component software, the user may need to understand data flow, architecture, repository layout, or component responsibilities first.

**The goal is not to place `Usage` at a particular position. The goal is to choose the order that makes the repository easiest to understand and use.**

## Explain reasons when they matter

When a technical choice would otherwise be difficult to understand or reproduce later, explain why it was made.

Examples include why a particular software version or architecture is required, why a compiler or network option is necessary, why components are separated, and compatibility constraints.

Do not add explanations merely to make the README longer or more complete-looking. Include them when they help someone use, reproduce, debug, or maintain the system.

## Show repository or container layout when useful

When the directory structure helps users or developers understand the repository, show it explicitly. Do not list every directory simply for completeness.

Show the structure that helps the reader understand how to use or develop the software.

## For Developers

Use a `For Developers` section whenever it is useful.

It may contain information such as local builds, helper scripts, GitHub Actions, image and tag policies, development environments, maintenance procedures, release procedures, and AI-assisted maintenance.

There is no rule that `For Developers` must be small.

If the README becomes difficult to navigate, reconsider the organization of the sections rather than automatically removing useful technical information.

## Keep the README consistent with the repository

Before writing or updating a README, inspect the actual repository whenever possible, including source code, scripts, Dockerfiles, workflow files, configuration files, and directory structure.

Do not invent commands based only on what seems conventional. Do not document scripts that do not exist. Do not leave obsolete ports, versions, paths, filenames, or commands in the README.

The README should describe the repository as it actually exists.

## Do not mechanically apply README conventions

General README conventions and best practices may be useful references. They are not requirements by themselves.

Do not include a section merely because a typical README contains it.

A small repository may need only a few lines. A complex repository may require a long and detailed README.

Judge the README by whether users can accomplish their goals, not by its length, number of sections, or conformity to a template.

## Instructions for AI

When an AI creates or revises a README, it must not optimize primarily for making the document look like a conventional, polished README.

In particular:

- Do not replace concrete commands with abstract explanations.
- Do not omit useful URLs.
- Do not generalize known versions, ports, paths, or other concrete values.
- Do not disturb the actual order of operations merely to fit a standard documentation structure.
- Do not remove necessary technical details merely because they appear too detailed for a README.
- Do not add unnecessary prose or sections merely to make the document look complete.
- Do not force the repository into a fixed README template.
- Do not assume that a conventional README structure is automatically easier for users.

Before editing the README, understand how the repository is actually used.

## Learn from Kobayashi's corrections

Do not assume that an AI-generated README in an existing repository automatically represents Kobayashi's preferred style.

Explicit corrections made by Kobayashi are stronger evidence of the intended style.

Examples include requests to move `How to use` earlier, make commands directly copy-pasteable, provide the actual download URL, explicitly state versions and architectures, reorganize the README around the user's workflow, and place development and maintenance information under `For Developers`.

If a new explicit correction conflicts with this document, prefer the new correction and update this style guide accordingly.
