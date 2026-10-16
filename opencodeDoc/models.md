# Models
 
OpenCode uses a model registry to manage which models are available for different agents and tasks.
 
## Registry
 
The registry defines the models that can be used. You can configure it in your OpenCode configuration file:
 
```jsonc title="opencode.jsonc"
{
  "models": {
    "claude-3-5-sonnet": {
      "provider": "anthropic",
      "model": "claude-3-5-sonnet-20240620",
      "variant": "high"
    },
    "gpt-4o": {
      "provider": "openai",
      "model": "gpt-4o"
    }
  }
}
```
 
When a model is not explicitly defined in the `models` registry, OpenCode will attempt to use the default model provided by the model's `provider`.
 
## Agent Model Configuration
 
Each agent can specify which model it prefers to use. This can be done via the `model` field in the agent's configuration.
 
```jsonc title="opencode.jsonc"
{
  "agents": {
    "writer": {
      "model": "claude-3-5-sonnet"
    }
  }
}
```
 
If the `model` field in the agent configuration specifies a model ID that is not in the `models` registry, OpenCode will use the model ID directly (e.g., `anthropic/claude-3-5-sonnet`).
 
## Variants
 
Some providers support different variants of the same model (e.g., different levels of reasoning or speed). These can be specified using the `#variant` suffix in the `model` field or within the `models` registry.
 
```yaml
model: anthropic/claude-3-5-sonnet#high
```
 
## Context Window
 
OpenCode automatically manages context window limits. When a session exceeds the model's context limit, OpenCode will summarize the conversation history to keep the context manageable.
 
## Costs
 
OpenCode does not provide built-in cost tracking. For information on pricing, please refer to the respective provider's website.
