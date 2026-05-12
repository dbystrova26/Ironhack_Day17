# Cover Letter Project

## Structure

- `skill.md` - skill instructions for drafting tailored cover letters
- `agents/openai.yaml` - agent metadata for the skill
- `job-search/` - working folder with the CV used as input context
- `wolt_cover_letter.txt` - drafted cover letter source text
- `wolt_cover_letter.pdf` - formatted PDF version of the cover letter

## Use

Use this project to draft and format tailored cover letters from a job description and CV context.

## How to Use

1. Put the CV in `job-search/` as `Daria_Bystrova_CV_2026.docx`.
2. Add the job description in your prompt or working notes, including the role title and company name.
3. Check `skill.md` for the required cover-letter structure and tone.
4. Draft the cover letter as plain text first in `wolt_cover_letter.txt` or a new text file.
5. Export the final version to PDF when the wording is approved.
6. Use `agents/openai.yaml` if you are wiring this into an agent workflow.
