<!--
This GSoC proposal may feature elements from the "Google Summer of Code Guides"
(<https://google.github.io/gsocguides/>) which are licensed under CC BY-SA 3.0.
Accordingly, the rest of the proposal (made by Piotr Płaczek) is also under the
same license. To view a copy of this license, visit <http://creativecommons.org/licenses/by-sa/3.0/>.
-->

<!--
Useful Links:

https://www.electronjs.org/blog/2024-summer-of-code

https://electronhq.notion.site/Electron-Google-Summer-of-Code-2024-Ideas-List-a1cb01daab3c48a98c30e411e96b218d

https://electronhq.notion.site/Electron-Google-Summer-of-Code-2024-Contributor-Guidance-f1f4de7a0d9a4664a96c8d4dd70cb208
-->

# Electron 2024 Proposal: API History in Electron Documentation

[*View this proposal nicely formatted as a GitHub Gist.*](https://gist.github.com/piotrpdev/9ed9bd7f0f3ab192a5ae58a35fbe03e7)

## Contact Information

<!-- Putting your full name on the proposal is not enough. Provide full contact
information, including your preferred name, email address, websites, etc. -->

### Personal Details

- **Name:** Piotr Płaczek (you can call me Peter)
- **Email:** [piotrpdev@gmail.com](mailto:piotrpdev@gmail.com)
- **Website:** [piotrp.dev](https://piotrp.dev/)
- **Resume:** [cv.piotrp.dev](https://cv.piotrp.dev/)
- **Location:** Waterford, Ireland

### Social Media

- **GitHub:** [piotrpdev](https://github.com/piotrpdev)
- **Twitter:** [@piotrpdev](https://twitter.com/piotrpdev)
- **LinkedIn:** [piotrpdev](https://www.linkedin.com/in/piotrpdev/)

### Working Hours

- **Time Zone:** UTC+1 (Ireland)
- **Working Hours:** 9:00 - 18:00 (flexible, just want to avoid burnout)

## Synopsis

<!-- If the format allows, start your proposal with a short summary, designed to
convince the reviewer to read the rest of the proposal. -->

The Electron documentation currently doesn't feature API history for its functions,
classes, etc. This proposal aims to implement the missing API history in a similar
fashion to the [Node.js documentation](https://nodejs.org/en/docs): by allowing the
use of a simple but powerful YAML schema in the API documentation Markdown files
and displaying it nicely on the Electron documentation website.

### Details

#### API history documentation system / YAML schema

The API history for a function/class/etc. will be placed directly after the
Markdown header for that item in the form of a YAML code block encapsulated by
a HTML comment.

~~~js
<!--
```YAML history
added: v10.0.0
deprecated: v25.0.0
removed: v28.0.0
changes:
  - version: v13.0.0
    pr-url: https://github.com/electron/electron/pull/26789
    description: Made `trafficLightPosition` option work for `customButtonOnHover` window.
```
-->
~~~

I believe using YAML like the Node.js docs do is the best approach because it
is easy to read. The API history isn't extremely complicated and should ideally
be as easy to write and read as possible. At first thought Markdown might seem
like a plausible alternative, however I don't think it provides the information
efficiency needed for the task. API history can be efficiently and clearly
conveyed through key value pairs, a table, etc. while I believe Markdown is more
suited for writing paragraphs and such.

*"Why not just place the YAML directly in the code block?"* - I believe my approach
provides the best readability and ease of use. A developer can create a fenced
code block with `YAML history` specified to get syntax highlighting (putting
`history` after YAML doesn't mess up highlighting) and just comment it out with a
shortcut once they're done. This also reduces the possibility of a formatting
error messing something up (since indentation is so important in YAML).

I believe this is the best approach since it indicates the API history is metadata
rather than a prop of the item or something similar. I think this also makes the
documentation easier to read when rendered on GitHub (since comments are hidden)
although the user can still switch to the code view and see the API history if
they desire.

#### JavaScript details

![diagram1](https://i.imgur.com/vC5NOLA.png)

I plan to create a new `@electron/docs-api-history-tools`
NPM package that will contain scripts for extracting, validating/linting and converting
the API history in the documentation files.

I then plan to introduce a `lint:docs-history` script into the [`electron/electron`](https://github.com/electron/electron)
`package.json` file and add it to the end of the `lint:docs` script. This will
run a linting script provided by my package (probably re-exported by [`@electron/lint-roller`](https://github.com/electron/lint-roller/))
and validate the YAML API history in the Markdown docs against a custom schema
using the [`ajv`](https://github.com/ajv-validator/ajv) package. This schema will
support additions, depreciations etc. and will help ensure consistency for API history
across the docs.

If validating the entire docs at once is too slow, I will look at other approaches
e.g. creating a GitHub Action that only validates the API history schema in pull
requests.

Furthermore, I will modify the [`scripts/pre-build.ts`](https://github.com/electron/website/blob/main/scripts/pre-build.ts)
script in [`electron/website`](https://github.com/electron/website) to use my package
in order to convert the YAML API history into a Markdown table (or maybe a HTML table
if the design calls for it) for the website. I will also modify the website CSS files
to style this API history table nicely (see UI section below for details).

Extraction and replacement of the YAML API history blocks will probably be done using
 simple regular expressions e.g. with [`String.prototype.replace()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace).

Instead of the library that
the Node.js doctool uses for parsing YAML ([`js-yaml`](https://github.com/nodeca/js-yaml)),
I plan to use the [`yaml`](https://github.com/eemeli/yaml) package since it is
regularly maintained and has no dependencies. [David Sanders (@dsanders11)](https://github.com/dsanders11)
suggested this and also mentioned that the Electron team is moving new code to
this package.

Of course, this package will have extensive tests written for it (probably using Jest).

#### Placeholder Version Replacing GitHub Action / Bot

I am planning on creating a simple GitHub Action/Bot that replaces placeholder
version values. This is needed so a developer can add API history documentation along
with their new feature, have it merged into a beta release, and when the official
release comes out the API documentation for the feature lists that official version
as the version the feature was added in.

Usage of my package for this will likely not be necessary; running a simple file
text find and replace will likely be sufficient ([that is what Node.js does](https://github.com/nodejs/node-core-utils/blob/b70e6368307156b4386129a30e07519aa5957eb3/lib/prepare_release.js#L427-L435)).

#### UI and styling for Electron documentation website

I will display the parsed API history data in a nice-looking format on the documentation
website. I plan on keeping it simple and readable like the Node.js documentation
does, but maybe make it look a bit nicer / more modern since Node.js's one looks
very basic. A responsive table will probably work best for most scenarios.

|          Design Prototype (Closed)        |           Design Prototype (Open)         |
|:-----------------------------------------:|:-----------------------------------------:|
| ![demo1](https://i.imgur.com/Mz1Wv1P.png) | ![demo2](https://i.imgur.com/svLg076.png) |

#### Usage/style Guide

I will add a section to the
[`electron/docs/styleguide.md`](https://github.com/electron/electron/blob/main/docs/styleguide.md)
usage/style guide dedicated to writing API history documentation for new features.
I will describe proper usages of the YAML schema in detail, provide typical/useful
examples, etc. I'm aiming for it to make the process of documenting API history as
easy as possible, especially considering that a lot of existing APIs will have to
be migrated to the new system.

#### Migration Guide

Since existing API's have to be migrated to the new documentation system, I will
create a migration guide (probably in `electron/docs/apihistoryguide.md`). It will
likely feature the typical steps of what a developer currently has to do when migrating
an old Electron app to a new release:
Read through the Electron [`breaking-changes.md`](https://github.com/electron/electron/blob/main/docs/breaking-changes.md)
file, browse the through the past [releases](https://github.com/electron/electron/releases),
maybe look through old commits, etc. They will then be instructed to follow the
usage/style guide to add API history documentation for each previously existing class/function/etc.

The (archived) [`electron-api-historian`](https://github.com/electron/electron-api-historian)
project should be able to speed up the process of finding when APIs were first added
(thanks [David Sanders (@dsanders11)](https://github.com/dsanders11) for the idea).
However, finding when individual functions, parameters, etc. were changed is a lot
more complicated.

Sadly, I can't think of a way to automate this effectively. This would probably be
a great task for an AI/ML engineer; however, I don't possess those skills and would
be too afraid of accidentally introducing [hallucinations](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence))
into the API history. Even if automated the information would still probably have
to be verified by a human in the end :/. This task will probably have to be done
manually, on a file-by-file basis, [just like it was done for the
Node.js documentation](https://github.com/nodejs/node/issues/6578).

I will however try using [`git log -L :<funcname>:<file>`](https://stackoverflow.com/questions/4781405/git-how-do-i-view-the-change-history-of-a-method-function)
along with some other Git tricks and see it if is useful.

## Benefits to Electron

<!-- Don’t forget to make your case for a benefit to the organization, not just to
yourself. Why would Google and your organization be proud to sponsor this work?
How would open source or society as a whole benefit? What cool things would be
demonstrated? -->

I believe the addition of API history to the documentation will make the lives of
developers using Electron a lot easier, especially ones attempting to migrate their
existing app from a several year old Electron version. I think this feature would
also increase the credibility of Electron and entice more people to use it.

## Deliverables

<!-- Include a brief, clear work breakdown structure with milestones and deadlines.
Make sure to label deliverables as optional or required. You may want your plan to
start by producing some kind of white paper, or planning the project in traditional
Software Engineering style. It’s OK to include thinking time (“investigation”) in
your work schedule. Deliverables should include investigation, coding and
documentation. -->

> [!NOTE]
>
> - All of the following are ***required*** deliverables. However, if some take
> longer than expected I will most likely reduce the time migrating API's to the
> new documentation system.
>
> - The deadlines for the deliverables are included in the timeline.

- `@electron/docs-api-history-tools` NPM package with:
  - Scripts for parsing and linting/validating YAML API history written according
to a custom YAML (technically JSON) schema.
  - Script for converting YAML API history blocks in the Markdown documentation files
to Markdown/HTML tables.
  - Useful error messages
  - Comprehensive documentation / code comments
  - Extensive Jest tests
  - Good performance
  - Code following best coding practices, Electron linting/style guide, etc.

- A comprehensive YAML schema for documenting API history which includes support
for:
  - Additions
  - Depreciations
  - Removals
  - Changes
  - Links to relevant pull requests
  - Backports
  - etc.

- A GitHub Action/Bot that will replace placeholder version values in the API history
documentation.
  - This is needed so a developer can add API history documentation along with their
    new feature, have it merged into a beta release, and when the official release
    comes out the API documentation for the feature lists that official version as
    the version the feature was added in.

- Nice UI and styling for API history on the Electron documentation website.
  - Styling that follows the rest of the website's design.
  - Responsive, accessible, and generally well written HTML, CSS, and JS.
  - etc.

- Usage/style guide section for new API history documentation system that is:
  - Easy to understand.
  - Well written.
  - Includes examples.
  - etc.

- Migration guide for new API history documentation system that is:
  - Easy to understand.
  - Well written.
  - Includes examples.
  - etc.

## Timeline

<!-- Include a brief, clear work breakdown structure with milestones and deadlines.
Make sure to label deliverables as optional or required. You may want your plan to
start by producing some kind of white paper, or planning the project in traditional
Software Engineering style. It’s OK to include thinking time (“investigation”) in
your work schedule. Deliverables should include investigation, coding and
documentation. -->

> [!NOTE]
>
> - My university's exam period finishes on the 17th of May and the next term
> starts in September. With this in mind, I should be able to fully dedicate
> my time to implementing the proposal without issues.

| **No.** | **Week** | **Task** |
| ------- | -------- | -------- |
| 1 | May 27th - June 2nd | - Discuss project with mentors <br> - Familiarize myself with the codebase <br> - Research existing approaches to problem e.g. in [Flask](https://flask.palletsprojects.com/en/3.0.x/api/) docs <br> - Research potential libraries to use |
| 2 | June 3rd - June 9th | - Implement primitive version of `@electron/docs-api-history-tools` <br> - Create GitHub Action to automatically publish package to NPM on new release |
| 3       | June 10th - June 16th         | - Research custom YAML schema <br> - Ask mentors and developers of Electron for feedback <br> - Finalize schema |
| 4       | June 17th - June 23rd         | - Add comprehensive input/schema validation <br> - Add comprehensive error messages <br> - Add comprehensive documentation / code comments |
| 5       | June 24th - June 30th         | - Implement Jest tests <br> - Benchmark the package <br> - Research/implement performance enhancements |
|         | **June 30th**                 | **- Midterm evaluation / Reflect and adjust plans** |
| 6       | July 1st - July 7th           | - Create GitHub Action/Bot to replace placeholder version values on new release |
| 7       | July 8th - July 14th          | - Add UI and styling for API history to the Electron docs website |
| 8       | July 15th - July 21st         | - Create usage/style guide section for new API history documentation system |
| 9       | July 22nd - July 28th         | - Create migration guide for new API history documentation system |
| 10      | July 29th - August 4th        | - Start migrating API history to docs using new system <br> - Get `lint:docs-history` and `electron/website` `pre-build.ts` changes merged |
| 11      | August 5th - August 11th      | - Keep migrating API history and ask mentors for review |
| **12**  | **August 12th - August 18th** | **- Write Final Report** |

## Related Work

<!-- You should understand and communicate other people’s work that may be related
to your own. Do your research, and make sure you understand how the project you are
proposing fits into the target organization. Be sure to explain how the proposed
work is different from similar related work. -->

As suggested in the original idea, I plan to implement an API history documentation
system similar to the one used in the Node.js documentation ([Node.js doctool](https://github.com/nodejs/node/tree/main/tools/doc),
[example](https://github.com/nodejs/node/blob/5117c6c9e7418eecd6e73145a1f8339e18db323f/doc/api/fs.md?plain=1#L655-L662)).
After researching it and looking at how it works, I really like it and think it would
be a good fit for the Electron documentation.

I like the way [Flask](https://github.com/pallets/flask/tree/main/docs) displays
API changes in their API documentation as well, however they use [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)
and include all of the historical API changes for a function in the source code
itself, which I don't really like.

I plan to take inspiration from various API history examples out there.

<!-- The following image contains screenshots from various open-source
projects including: Node.js, Flask, Django, Python, React, MDN Web Docs
and PSPDFKit. I do not own or hold copyright over any of the content
in the image, all rights belong to their respective owners. -->

![inspiration](https://i.imgur.com/waKV6LS.png)

## Biographical Information

<!-- Keep your personal info brief. Be sure to communicate personal experiences
and skills that might be relevant to the project. Summarize your education, work,
and open source experience. List your skills and give evidence of your qualifications.
Convince your organization that you can do the work. Any published work, successful
open source projects and the like should definitely be mentioned. -->

### General

- I am 20 years old
- I was born in Poland
- I am currently living in Waterford, Ireland
- I started programming back in 2018 with C++
- My first real project was a Python web scraper I made in 2020

### Work

- I did a 7 month work placement at Red Hat in 2023 as part of my university course.
I mainly worked on an AI/ML model delivery pipeline running on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
using [Tekton](https://tekton.dev/). I also wrote a Medium article detailing
my work, [here is a link](https://medium.com/piotr-p%C5%82aczeks-red-hat-internship-experience/bringing-ai-to-the-edge-839f96cc6756).

### Education

- **University:** South East Technological University
- **Degree:** BSc (Hons.) in Software Systems Development
- **Expected Graduation:** May, 2025

### Related Open-Source Work

Some of my open-source work that showcases I have the prior experience necessary
to implement the proposal:

- [`electron/electron (#41642)`](https://github.com/electron/electron/pull/41642):
A simple bugfix PR I made.
  - Set-up [`electron/build-tools`](https://github.com/electron/build-tools) on a
[Hetzner](https://www.hetzner.com/) VPS.
  - Ran the build, linting, and test scripts.
  - Created a PR, implemented suggested changes and rebased before merge.

- [`piotrpdev/cao-calculator`](https://github.com/piotrpdev/cao-calculator): Android
app for calculating university entrance points, built with React Native and TypeScript.

- [`piotrpdev/WIT-Timetable-Generator`](https://github.com/piotrpdev/WIT-Timetable-Generator):
Web scraper for my university's timetable page, built with JavaScript.

- [`piotrpdev/resume`](https://github.com/piotrpdev/resume): My resume, built with
React and JavaScript.

### Interests

- Music (performance, composition, listening, etc.)
- Electronics (Arduino, Raspberry Pi, ESP32, etc.)
- Gaming (Arma 3, Dark Souls, Pokémon, etc.)
