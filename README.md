# Bot team build spec

`BUILD.md` is a spec you give to your own bb agent. The agent checks your bb install, interviews you, proposes a team of three to five bots, and builds it on [tobi/bb-bots-sidebar](https://github.com/tobi/bb-bots-sidebar) after you approve.

The spec describes behavior and file formats. It contains no code. Your agent writes its own implementation against the plugin as installed on your machine.

## Use

1. Install bb and the Bots Sidebar plugin.
2. Open a new bb conversation in any project.
3. Paste `BUILD.md` as the first message, or tell the agent to read it.
4. Answer the interview. Approve the proposal when it looks right.

The build has three tiers. Tier 1 is the core team: roster, charters, rules, routing and sync. Tier 2 adds wakeups, pulses and a digest on bb automations. Tier 3 adds approval cards and a task-system connection. You can stop after any tier.

## Files

- `BUILD.md`: the spec.
- `test/persona.md`: the scripted owner used to test the spec on a clean install.

## Credits

Built on [tobi/bb-bots-sidebar](https://github.com/tobi/bb-bots-sidebar) by Tobias Lütke.

## License

None yet.
