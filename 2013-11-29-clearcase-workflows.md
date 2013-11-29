---
layout: article
title: ClearCase Workflows
category: ClearCase
date: 2013-11-29 13:03:19
excerpt:
  ClearCase标准的工作流程
---

# Process Model

There are two process models that have been implemented on top of ClearCase and ClearQuest: Base and UCM (Unified Change Management), and UCM is recommended in State Street Corporation.

Base ClearCase, as implement of the second generation theory of configuration management solution, can only deal with Files and Directories. It uses branch and view to support parallel development, and tag a label for each release.

UCM ClearCase, as implement of the third generation theory of SCM solution, bring in some new concepts, activity, change set, baseline, stream etc. It manages change set and activity rather than files.

| Base      | UCM        |
| --------- | ---------- |
| Base View | UCM View   |
| /         | Component  |
| Branch    | Stream     |
| /         | Activity   |
| Label     | Baseline   |
| Element   | Element    |
| /         | Change Set |

Brief concept for UCM:

1. VOB (Version Object Base): is a data repository that holds the files, directories, and any objects that collectively represent a product or a development effort.
  + Project VOB: keeps the metadata of VOB. Like projects, stream, baseline, activity, change set etc.
  + Components VOB: is a group of related directory and file elements, which are developed, integrated, and released together. 
1. Stream: is private work area of developers. It contains a set of baselines, uses to instead branch. Developers can create a development stream base on a baseline for whole project, while branch can only create a branch for single file.
1. Baseline: represents a version of one or more components. It contains a set of activities. To instead label in base.
1. Activity: records a set of files that a developer creates or modifies to complete a development task, such as fixing a bug.
1. Change Set: is a set of files associated with an activity.
1. Element: is a file or directory. An element contains one or more versions.

# UCM Workflows

In a UCM project, work typically progresses as follows:

1. ClearCase administrators create the VOBs. 

    ClearCase Administrators create the VOBs that will contain all the files and directories that represent the development effort, such as a new product release.

1. Project managers create the UCM project. 

    Project manager create a UCM project and identify an initial set of components as a starting point (the baseline based on).

1. Developers join the UCM project.

    Developers join the project by creating their private work areas and populating them with the contents of the project's baselines.

1. Development is ongoing. 

    Developers create, or are assigned, activities, and work on one activity at a time.

1. Developers deliver work to the shared work area. 

    When developers complete activities, they build and test their work in their private work areas (Development Stream). When developers create a successful software build in their private work area, they share their work with the project team by performing deliver operations. A deliver operation merges the work from the developer's private work area to the project's shared work area (Integration Stream).

1. Release engineering builds the product.

    Periodically, the delivered work from developers is integrated by building the project's executable files in the shared work area. This work is typically done by an individual in the development group who is assigned to build the product.

1. Project managers create new baselines. 

    If the project builds successfully, project manager create a new set of baselines. In a separate work area, a team of Quality Engineers performs more extensive testing of the new baselines.

1. Project managers promote specific baselines that reflect a particular project milestone.

    Periodically, as the quality and stability of baselines improve, project manager adjust the promotion level attribute of the baselines to reflect appropriate milestones, such as Built, Tested, or Released. When the new baselines pass a sufficient level of testing, project manager designate them as the recommended set of baselines

1. Developers adjust their private work areas to the latest available baselines.

    Developers perform rebase operations to update their private work areas to include the set of versions represented by the new recommended baselines.

    Developers continue the cycle of working on activities, delivering completed activities, and updating their private work areas with new baselines.

For production hot fix process:

1. Developers create a hot fix development stream

    Developers create a new development stream base on the production baseline

1. Development is ongoing

    Developer applies hot fix changes on new stream.

1. Project manager create a new baseline on HOT FIX stream

    When developers complete changes, they build and test their work in their private work areas. Then, project manager create a baseline on HOT FIX stream. The new baseline contains production release and hot fix changes only.

1. Developers deliver hot fix changes to current development stream

    To merge the hot fix to current develop task.
