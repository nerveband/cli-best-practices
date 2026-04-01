# Ten rules for agent-friendly CLIs

**Source:** Eric Zakariasson (Cursor), posted on [X in March 2026](https://x.com/ericzakariasson/status/2036762680401223946). 483K views, 1,816 likes.

The list is short but nearly everything on it has bitten me at least once across [craft-cli](https://github.com/nerveband/craft-cli), [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), and the other CLIs I've built.

## 1. Make it non-interactive

If your CLI drops into a prompt mid-execution, an agent is stuck. It can't press arrow keys or type "y" at the right moment. Every input should be passable as a flag. Interactive mode is a fallback for when flags are missing, never the default path.

## 2. Progressive help, not a wall of text

An agent runs `mycli`, sees subcommands, picks one, runs `mycli deploy --help`, gets what it needs. It never has to read the full manual. Let it discover as it goes.

## 3. Useful --help

Every subcommand gets `--help`. Every `--help` includes examples. Agents pattern-match off `mycli deploy --env staging --tag v1.2.3` faster than they parse a paragraph description.

## 4. Flags and stdin for everything

Agents think in pipelines. They chain commands and pipe output. Don't require positional args in weird orders or fall back to interactive prompts for missing values.

## 5. Fail fast with actionable errors

Missing a required flag? Don't hang. Print the correct invocation and exit. Agents self-correct when you give them something concrete.

## 6. Idempotent commands

Agents retry constantly. Network timeouts, context loss mid-task. Running the same deploy twice should return "already deployed, no-op," not create a duplicate.

## 7. --dry-run for destructive actions

Let agents preview what a delete or deploy would do before committing. Validate the plan, then run it for real.

## 8. --yes to skip confirmations

Humans get "are you sure?" and agents pass `--yes`. The safe path is still the default, but bypassing it is one flag away.

## 9. Predictable command structure

If an agent learns `mycli service list`, it should be able to guess `mycli deploy list` and `mycli config list`. Pick a pattern (resource + verb) and stick with it.

## 10. Return data on success

Show the deploy ID and the URL. Structured data the agent can parse and act on. Not just "Done!" with a confetti emoji.
