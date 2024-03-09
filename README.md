# GSoC Proposals

## General Information

- <https://google.github.io/gsocguides/student/writing-a-proposal>

## Inspiration

- [*"jQuery Mobile GSoC 2015 Proposal"* by Amanpreet Singh](https://docs.google.com/document/d/16xIbGHdqmJExXlksMso--rOecZ34j0-iVskw9gUK3P8/edit?usp=sharing)
- [*"Write JITLink support for a new format/architecture (ELF/AARCH64)"* by Sunho Kim](https://compiler-research.org/assets/docs/Sunho_Kim_Proposal_2022.pdf)
- [*"Mission Support System (MSS): Advanced CLI Plotting"* by Sreelakshmi P Jayarajan](https://blogs.python-gsoc.org/media/proposals/GSoC_proposal_kXzduWp.pdf)
- Other proposals found using <https://www.google.com/search?q=%22gsoc%22+%22proposal%22+filetype%3Apdf>

## Tips

### Date generation

You're often asked to generated a schedule/list of deliverables for a certain time range, this is how I do it faster:

- Copy and paste your range of weeks [from here](https://www.epochconverter.com/weeks/2024) into Microsoft Excel.
- Replace all ", (year)" with "".
- Inset this formula in `D1`: `=TEXTJOIN(" - ", TRUE, B1, C1)`.
- Select `D1` down to bottom of weeks and 'Fill > Down'.

## License

This repository and its contents may feature elements from the "Google Summer of Code Guides" (<https://google.github.io/gsocguides/>) which are licensed under CC BY-SA 3.0. Accordingly, the rest of the repository contents (made by Piotr Płaczek) are also under the same license. To view a copy of this license, visit <http://creativecommons.org/licenses/by-sa/3.0/>.
