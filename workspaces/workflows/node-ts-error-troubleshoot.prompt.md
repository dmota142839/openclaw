# Node.js/TypeScript Error Log Troubleshooting Prompt

## Purpose
Review error logs related to Node.js/TypeScript projects (especially module resolution and plugin activation errors) and generate a structured troubleshooting plan.

## Instructions
Given an error log or session transcript, perform the following:

1. **Summarize the Error(s):**
   - Identify the main error messages (e.g., ERR_MODULE_NOT_FOUND, plugin activation failures).
   - Note affected files, modules, or plugins.

2. **Diagnose Likely Causes:**
   - Suggest possible reasons for the error (e.g., missing files, incorrect import paths, build issues).
   - Reference any relevant context (e.g., directory checks, file existence).

3. **Recommend Next Steps:**
   - List concrete troubleshooting actions (e.g., verify file existence, check import paths, reinstall dependencies, rebuild project).
   - If relevant, suggest commands to run (e.g., `ls`, `npm install`, `tsc`).

4. **Format Output:**
   - Use clear sections: **Summary**, **Diagnosis**, **Recommended Actions**.
   - Use bullet points or numbered lists for clarity.

## Example Invocation
- "Review this error log and provide a troubleshooting plan."
- "Analyze this plugin activation failure and suggest next steps."

---

**Tip:** You can adapt this prompt for other languages or error types by changing the context in the instructions.
