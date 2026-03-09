# How to Work with Agents Effectively

## Instructing the Agent

1. Be specific.
2. Don't be superfluous.
3. Choose words with care. Read [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119). Avoid indirect references ("it", "that") — name the subject.
4. If it's not written, it will be assumed.
5. If it's assumed it won't meet your expectations.
6. Never grant exceptions. Exceptions drift to become standard practice.
7. Minimize the number of instruction files. Agents skip reading files when they can.
8. Proof of verification is a gate to finalize all work.
9. Teach the agent what correct means. Compiling does not mean bug-free.
10. Commit work often. git bisect helps quickly identify where issues were introduced.

## Leveraging the Agent's Self-Awareness

Ask the agent to:

1. Recommend tools or workflows that would improve its output for your project.
2. Create user instructions (AGENTS.md, workflows, skills).
3. Review and simplify your existing user instructions.
4. Check if user instructions conflict with the agent's standard process (system prompt, internal assumptions, operation method).
5. Root cause why the agent did not do what you expected and recommend better ways to instruct it.
