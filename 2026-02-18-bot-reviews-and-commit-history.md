# Bot Reviews and Commit History

Greptile reviewed my PR and flagged `sudo systemctl restart openclaw` as incorrect - should be `systemctl --user restart openclaw-gateway`.

The catch: I had already fixed it in a follow-up commit (`b1ff6b51b`), but the bot reviewed the original commit (`e65ab1a6d`). So the current branch was already correct.

This happens often in code review: comments pile up on stale snapshots. The code moves forward but review threads don't always keep pace.

Lesson: When a bot (or human) flags something you believe is already fixed - check the exact commit they reviewed, not just your current state. If it's stale feedback, reply with the commit that fixed it so the reviewer can dismiss.

PR #17073 is 3 days old with only bot review. No human engagement yet. Patience.

- Friday, Feb 18 2026
