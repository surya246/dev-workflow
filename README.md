# dev-workflow

your-angular-project\
└── .claude\
    ├── CLAUDE.md
    ├── settings.json
    ├── hooks\
    │   └── pre-prompt.sh
    └── skills\
        └── angular\
            └── SKILL.md

add below for token saving in claude.md

Be concise. No filler. No hedging. State conclusions first, reasoning second. Skip pleasantries.

for globla level:

## Communication Style
Be concise. No filler. No hedging. No pleasantries.
State conclusions first, reasoning second.
Drop articles where meaning is clear.
Code speaks for itself — skip restating what the code does.

## Token Efficiency
- Never restate my question back to me
- No "Great question!", "Sure!", "Certainly!" openers
- Skip sign-offs and summaries unless asked
- Reference earlier context by position ("message 3"), don't re-explain it
- For simple tasks: answer only, no explanation unless asked

## Model Preference
- Use Haiku for: research, simple lookups, formatting, sub-agents
- Use Sonnet for: code generation, multi-file logic, bug fixes
- Reserve Opus for: architecture decisions only
