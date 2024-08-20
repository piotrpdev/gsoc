<!--
This GSoC work product may feature elements from the "Google Summer of Code Guides"
(<https://google.github.io/gsocguides/>) which are licensed under CC BY-SA 3.0.
Accordingly, the rest of the work product (made by Piotr Płaczek) is also under the
same license. To view a copy of this license, visit <http://creativecommons.org/licenses/by-sa/3.0/>.
-->

<!--
Useful Links:

https://www.electronjs.org/blog/2024-summer-of-code

https://electronhq.notion.site/Electron-Google-Summer-of-Code-2024-Ideas-List-a1cb01daab3c48a98c30e411e96b218d

https://electronhq.notion.site/Electron-Google-Summer-of-Code-2024-Contributor-Guidance-f1f4de7a0d9a4664a96c8d4dd70cb208
-->

# Electron 2024 Work Product: API History in Electron Documentation

## Preamble

I want to sincerely thank my mentors:

- David Sanders [(@dsanders11)][dsanders11]
- Keeley Hammond [(@VerteDinde)](https://github.com/VerteDinde)
- Erick Zhao [(@erickzhao)][erickzhao]

...and the rest of the Electron team for answering my questions
and taking the time to give me feedback on my pull requests.
It is very much appreciated.

Links to all of my work can be found in the ["Deliverables"](#deliverables) section.

## Contact Information

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

## Synopsis

Over the course of the [Google Summer of Code (GSoC)](https://summerofcode.withgoogle.com/)
program I implemented an API history feature for the Electron documentation and its
functions, classes, etc. in a similar fashion to the
[Node.js documentation](https://nodejs.org/en/docs): by allowing the
use of a simple but powerful YAML schema in the API documentation Markdown files
and displaying it nicely on the Electron documentation website.

### Details

#### API history documentation system / YAML schema

The API history for a function/class/etc. is now placed directly after the
Markdown header for that item in the form of a YAML code block encapsulated by
a HTML comment.

~~~yaml
#### `win.setTrafficLightPosition(position)` _macOS_

<!--
```YAML history
added:
  - pr-url: https://github.com/electron/electron/pull/22533
changes:
  - pr-url: https://github.com/electron/electron/pull/26789
    description: "Made `trafficLightPosition` option work for `customButtonOnHover` window."
deprecated:
  - pr-url: https://github.com/electron/electron/pull/37094
    breaking-changes-header: deprecated-browserwindowsettrafficlightpositionposition
```
-->

* `position` [Point](structures/point.md)

Set a custom position for the traffic light buttons. Can only be used with `titleBarStyle` set to `hidden`.
~~~

I believe using YAML like the Node.js docs do was the best approach because it
is easy to read. The API history isn't extremely complicated and should ideally
be as easy to write and read as possible.

The final design shown above is actually significantly different to the one I proposed:

~~~yaml
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

One large change is the removal of version numbers:

> "[...] There’s one somewhat significant change we’d like to call out about
> the proposal, which came up during discussion when we were reviewing proposals.
> [...]
>
> [we] decided that the approach with the least drawbacks would be to only
> use PR URLs (the root PRs to main) instead of hardcoded version strings as in
> the proposal.
>
> This can serve as a single source of truth which can then be used
> to derive exact version numbers, and no further documentation changes on main
> are necessary if the change is backported to other branches."
>
> — David Sanders [(@dsanders11)][dsanders11] via Slack

We also didn't include removals in the API History, since when an API is removed
from Electron, it is also removed from the documentation.

#### JavaScript details

I originally planned to create a new `@electron/docs-api-history-tools`
NPM package that would contain scripts for extracting, validating/linting and converting
the API history in the documentation files.

About a week before the coding period began, and after some discussion with my
mentors, I realized that was probably unnecessary:

> "Hi everyone, I was thinking about the project after our huddle: Considering
> that extraction logic will have to be handled differently in `e/website` and
> `e/lint-roller` because of their dependencies, maybe there is no need for a
> separate package for API history stuff?"
>
> | Proposed                  | Revised                 |
> |:-------------------------:|:-----------------------:|
> | ![proposed][js-proposed]  | ![revised][js-revised]  |
>
> — Piotr Płaczek (me) via Slack

We ultimately decided to not go ahead with my original idea:

> "@Piotr Płaczek that seems fine to me! I think we can always refactor out to a
> separate module in a later iteration if we find that we need to share a lot of
> code between the two implementations anyways :slightly_smiling_face:"
>
> — Erick Zhao ([@erickzhao][erickzhao]) via Slack

Instead, we divided those various tools across the Electron repos that were most
relevant to them:

- `yaml-api-history-schema.json`
  - -> `electron/electron` (`api-history.schema.json`)
- `lint-yaml-api-history.ts`
  - -> `electron/lint-roller` (`lint-markdown-api-history.ts`)
- `extract-yaml-api-history.ts`
  - -> `electron/website` (`preprocess-api-history.ts`)
- `yaml-api-history-to-markdown.ts`
  - -> `electron/website` (`transformers/api-history.ts`)
  - -> `electron/website` (`ApiHistoryTable.tsx`)

#### Placeholder Version Replacing GitHub Action / Bot

I originally planned on creating a simple GitHub Action / Bot that would replace
placeholder version values. However, this was no longer necessary after we decided
to derive those version numbers from Pull Request URLs instead.

#### UI and styling for Electron documentation website

I originally proposed a simple table to display the API History data:

| Design Prototype (Closed)           | Design Prototype (Open)           |
|:-----------------------------------:|:---------------------------------:|
| ![demo1][prototype-closed-proposed] | ![demo2][prototype-open-proposed] |

This is what the final implemented design looks like:

![demo3][open-implemented]

Pretty much the same as the prototype. The most significant addition is the use
of [SemVer](https://semver.org/) ranges, which were chosen to better communicate
which versions a feature is present in.

#### Usage/style Guide

I added a usage/style guide dedicated to writing API history documentation for
new features. I described proper usages of the YAML schema in detail, provided
typical/useful examples, etc.

#### Migration Guide

Since existing API's have to be migrated to the new documentation system, I created
a migration guide. It features the typical steps of what a developer has
to do when migrating old APIs: looking through breaking changes, browsing through
the past releases, maybe looking through old commits, etc.
Then instructing the developer to follow the usage/style guide to add API history
documentation for each previously existing class/function/etc.

Sadly, I couldn't think of a way to automate this effectively. This would probably
be a great task for an AI/ML engineer; however, I don't possess those skills and
was too afraid of accidentally introducing [hallucinations](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence))
into the API history. Even if automated, the information would still probably have
to be verified by a human in the end :/. This task will probably have to be done
manually, on a file-by-file basis, [just like it was done for the
Node.js documentation](https://github.com/nodejs/node/issues/6578).

## Benefits to Electron

I believe the addition of API history to the documentation will make the lives of
developers using Electron a lot easier, especially ones attempting to migrate their
existing app from a several year old Electron version. I think this feature will
also increase the credibility of Electron and entice more people to use it.

## Deliverables

> [!NOTE]
>
> - The deadlines for the deliverables are included in the ["Timeline"](#timeline)
>   section.

- `api-history.schema.json`
  - A comprehensive YAML schema for documenting API history which includes support
  for:
    - [x] Additions
    - [x] Depreciations
    - [ ] ~~Removals~~
    - [x] Changes
    - [x] Links to relevant pull requests
    - [x] Backports
    - [x] etc.
  - [x] Proposed in: [electron/rfc#6][rfc]
  - [x] Implemented/Used in: [electron/electron#42982][electron]
  - [x] Used in: [electron/website#594][website]

- `lint-markdown-api-history.ts`
  - Script for linting YAML API history written according to a custom YAML
    (technically JSON) schema.
    - [x] Useful error messages
    - [x] Comprehensive documentation / code comments
    - [x] Extensive ~~Jest~~ Vitest tests
    - [x] Good performance
  - [x] Implemented in: [electron/lint-roller#73][lint-roller]
  - [x] Used in: [electron/electron#42982][electron]

- `preprocess-api-history.ts`
  - Performs simple validation just in case incorrect API History manages to make
    it into the repo. Also strips the HTML comment tags that wrap API History blocks
    since [Docusaurus](https://docusaurus.io/) cannot parse them.
  - [x] Implemented/Used in: [electron/website#594][website]

- `transformers/api-history.ts`
  - Script for converting YAML API history blocks in the Markdown documentation files
    to ~~Markdown/HTML~~ [React](https://react.dev/) tables (`ApiHistoryTable.tsx`).
  - [x] Implemented/Used in: [electron/website#594][website]

- `ApiHistoryTable.tsx`
  - React table component used to display parsed API History data on the
    documentation website.
    - [x] Uses styling that follows the rest of the website's design.
    - [x] Responsive, accessible, and generally well written HTML, CSS, and JS.
    - [x] etc.
  - [x] Implemented/Used in: [electron/website#594][website]

- `styleguide.md`
  - Usage/style guide section for new API history documentation system.
    - [x] Easy to understand
    - [x] Well written
    - [x] Includes examples
    - [x] etc.
  - [x] Implemented/Used in: [electron/electron#42982][electron]

- `api-history-migration-guide.md`
  - Migration guide for new API history documentation system.
    - [x] Easy to understand
    - [x] Well written
    - [x] Includes examples
    - [x] etc.
  - [x] Implemented/Used in: [electron/electron#42982][electron]

## Timeline

> [!NOTE]
>
> - My university's exam period finished on the 17th of May and the next term
> started in September. With this in mind, I was able to fully dedicate my time
> to implementing the proposal without issues.
> - You can view all of the Pull Requests / Issues I worked on here:
>   - [Issues](https://github.com/issues?q=sort%3Aupdated-desc+is%3Aissue+author%3A%40me+org%3Aelectron)
>   - [Pull Requests](https://github.com/pulls?q=sort%3Aupdated-desc+is%3Apr+author%3A%40me+org%3Aelectron)

| **No.** | **Week**                      | **Task** |
| ------- | ----------------------------- | -------- |
| 1       | May 27th - June 2nd           | - Discussed project with mentors <br> - Familiarized myself with the codebase <br> - Researched existing approaches to problem e.g. in [Flask](https://flask.palletsprojects.com/en/3.0.x/api/) docs <br> - Researched potential libraries to use |
| 2       | June 3rd - June 9th           | - Implemented primitive version of proposal |
| 3       | June 10th - June 16th         | - Researched custom YAML schema <br> - Started implementing YAML schema <br> - Asked mentors and developers of Electron for feedback |
| 4       | June 17th - June 23rd         | - Added comprehensive schema validation to linting script <br> - Added comprehensive error messages to linting <br> - Added comprehensive code comments to linting |
| 5       | June 24th - June 30th         | - Implemented Vitest tests for linting script <br> - Benchmarked the linting script <br> - Researched/implemented performance enhancements |
|         | **June 30th**                 | **- Midterm evaluation / Reflected and adjusted plans** |
| 6       | July 1st - July 7th           | - Kept adding features to linting script |
| 7       | July 8th - July 14th          | - Kept working on UI and styling for API History table <br> - Finalized schema |
| 8       | July 15th - July 21st         | - Created usage/style guide section for new API history documentation system |
| 9       | July 22nd - July 28th         | - Start migrating API history to docs using new system |
| 10      | July 29th - August 4th        | - Created migration guide for new API history documentation system |
| 11      | August 5th - August 11th      | - Kept migrating API history and asked mentors for review |
| **12**  | **August 12th - August 18th** | **- Wrote Final Report** |
| 13      | August 19th - August 26nd     | - Worked on last minute changes |

[dsanders11]: https://github.com/dsanders11
[erickzhao]: https://github.com/erickzhao
[js-proposed]: https://github.com/user-attachments/assets/3ec6c87b-1569-443b-8b63-eb20ee4fd96b
[js-revised]: https://github.com/user-attachments/assets/321745a4-ad09-4717-94cc-7b3e08a85d3c
[prototype-closed-proposed]: https://github.com/user-attachments/assets/a1c329d0-9f6b-4079-94d3-f9ff5b80c9ea
[prototype-open-proposed]: https://github.com/user-attachments/assets/f1556d89-8fc8-485f-995c-1edaf8ee1413
[open-implemented]: https://github.com/user-attachments/assets/a8ceb931-ac10-46bc-b747-bf8fd97c839f
[electron]: https://github.com/electron/electron/pull/42982
[website]: https://github.com/electron/website/pull/594
[lint-roller]: https://github.com/electron/lint-roller/pull/73
[rfc]: https://github.com/electron/rfcs/pull/6
