# ENVIRONMENT — {{TOPIC}}

The chosen setup for practicing this topic, the options considered, the **spend cap**, and
how to tear everything down. The spend cap is a **design constraint** the plugin's generators
and `/mentor` are instructed to respect when proposing exercises — it is not technically
enforced by anything, so you remain responsible for your own cloud spend (billing alarms are
the real backstop).

## Chosen setup

{{The option selected and a one-line why. What the learner installs/creates to start.}}

## Practice medium

{{code repo | Jupyter notebooks | LaTeX-markdown problem sets | lab environment | mix}}

## Spend cap

- **One-time:** {{$N}}
- **Monthly:** {{$N}}  ← the target ceiling generators and `/mentor` design exercises within;
  not a system-enforced limit, so keep an eye on actual spend yourself.
- {{For cloud topics: billing alarms + a teardown script are the FIRST practice exercise —
  they are the actual enforcement mechanism for the cap.}}

## Options matrix (kept for reference)

| Option | One-time cost | Monthly cost | Setup complexity | Learning value | Risks / limits |
|---|---|---|---|---|---|
| {{option A}} | {{$}} | {{$}} | {{low/med/high}} | {{low/med/high}} | {{…}} |
| {{option B}} | {{$}} | {{$}} | {{…}} | {{…}} | {{…}} |
| {{option C}} | {{$}} | {{$}} | {{…}} | {{…}} | {{…}} |

## Teardown checklist

- [ ] {{stop/delete running resources}}
- [ ] {{remove credentials / revoke keys}}
- [ ] {{confirm billing returned to $0}}
- [ ] {{…}}
