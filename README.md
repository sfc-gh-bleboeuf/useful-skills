# Useful Skills

A collection of reusable [Cortex Code](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code) skills for common Snowflake tasks.

## Skills

### cost-estimator

Estimates Snowflake credit and dollar costs for customer use cases. Covers data ingestion (Snowpipe, COPY INTO), warehouse compute, Cortex Analyst, Cortex Search, Cortex AISQL functions, and storage.

**Triggers:** "estimate cost", "sizing exercise", "credit estimate", "pricing calculator"

## Installation

Copy any skill folder into your global skills directory:

```bash
cp -r <skill-name> ~/.snowflake/cortex/skills/
```

Once installed, skills activate automatically in Cortex Code when matching trigger phrases are used.

## Usage

Each skill contains a `SKILL.md` file that guides Cortex Code through a structured workflow. Simply describe your use case in conversation and the relevant skill will activate.

## Contributing

Add new skills as directories containing a `SKILL.md` file following the [Cortex Code skill format](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code).
