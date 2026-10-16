# Skills
 
Skills are modular capabilities that OpenCode can use to perform specialized tasks. They provide extra tools, specific workflows, or extra information to the agent.
 
## Definition
 
A skill is a Markdown file that describes how to perform a task. It can include:
- **Instructions**: Guidelines for the agent on how to use the skill.
- **Tools**: Specific CLI tools or API calls that the skill can invoke.
- **Examples**: Samples of how to use the skill effectively.
- **Context**: Background information or references relevant to the skill's domain.
 
## Configuration
 
Skills can be enabled or disabled in the OpenCode configuration file:
 
```jsonc title="opencode.jsonc"
{
  "skills": {
    "archify": {
      "enabled": true,
      "description": "Create visual architecture diagrams"
    },
    "doc-merge-zh": {
      "enabled": false
    }
  }
}
```
 
## Activation
 
When an agent needs to perform a task that matches a skill's description, it can use the `skill` tool to load and execute the skill. OpenCode will then provide the skill's instructions and tools to the agent's context.
 
## Development
 
You can create your own skills by defining a Markdown file and adding it to your project's `.opencode/skills/` directory or your global configuration. 
 
Example of a custom skill:
 
```md title=".opencode/skills/my-custom-skill.md"
---
description: Perform a custom data cleanup task.
---
 
Step 1: Identify the data files.
Step 2: Run the cleanup script provided in the `scripts/cleanup.py` file.
Step 3: Verify the results by checking the logs.
```
 
## Management
 
- **Discovery**: OpenCode automatically discovers skills in the `.opencode/skills/` directory.
- **Prioritization**: If multiple skills match a request, the agent will select the most relevant one based on the `description` field.
- **Updating**: You can update skills by modifying their respective Markdown files. OpenCode will pick up the changes immediately.
