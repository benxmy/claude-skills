---
description: Send notifications and permission prompts to Slack (default 120 min, or specify duration)
---

Enable Slack mode so notifications and permission prompts go to Slack instead of the terminal. Automatically turns off after the specified duration.

Run the slack-mode script: $ARGUMENTS

- No arguments or a number = enable Slack for N minutes (default 120)
- "off" = back to terminal-only
- "status" = show current state

```bash
~/tools/hooks/slack-mode.sh $ARGUMENTS
```

After running, confirm to the user what happened (Slack enabled until when, or back to terminal).
