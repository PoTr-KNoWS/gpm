
# Unified Policy Management

This repository contains research materials on models and languages to describe policy management -- in particular the messages involved in negotiation processes for access and usage policies.

## About this repo

### Structure

The contents of this repo are structured as follows:

- [./brainstorms/](./brainstorms/) contains working documents (summaries, comparisons, sketches ...);
- [./paper/](./paper/) contains the initial draft of a paper on a unified model;
- [./mappings/](./mappings/) contains several technical mappings of different models and languages to the unified one.

### Tooling

All dependencies are managed my Mise-en-place, which can be installed by running `curl https://mise.run | sh` (or via any of the other [installation methods](https://mise.jdx.dev/installing-mise.html#installation-methods)).
Mise needs to be actived to make the required tooling available. This can be done manually per session by running `mise activate`, or automatically by including this command [in your shell setup](https://mise.jdx.dev/installing-mise.html#shells).
The necessary tooling, specified in [./.mise.toml](./.mise.toml), is then set up by running `mise install`, and updated with `mise upgrade`.

The following tools are used in this project:

- [Quarto](https://quarto.org/): scientific publication tool.
- [Treefmt](https://treefmt.com/): manages linters and formatters; configured in [./.treefmt.toml](./.treefmt.toml), run with `treefmt <paths...>`.
- [Vale](https://vale.sh/): a prose linter; configured in [./.vale.ini](./.vale.ini).
- [RDFizer](https://github.com/SDM-TIB/SDM-RDFizer/): an RML engine.
