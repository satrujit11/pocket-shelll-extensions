# Testing an extension

`pocket-shell-cli ext test <folder>` validates the manifest and runs every `tests/*.yaml` of the folder. Each fixture runs the **real engine** (the same code the agent uses) against **fake programs** you declare, so it needs none of the tools installed and is the same everywhere.

## A fixture

```yaml
name: containers, detail and stopping one
fakes:                           # programs the extension may call, as shell scripts
  docker: |
    case "$1" in
      ps) cat <<'EOF'
    {"ID":"aaa111","Names":"web","Image":"nginx:1","State":"running","Status":"Up 2 hours"}
    EOF
      ;;
      inspect) echo '[{"Name":"/web","State":{"Running":true}}]' ;;
    esac
checks:
  - screen: main
    contains: ['"1 containers"', 'nginx:1']
  - screen: detail
    params: { id: aaa111 }
    offers: [stop]
    not_offers: [start]
  - name: stopping needs confirming
    action: stop
    screen: detail
    params: { id: aaa111 }
    refused: confirmed
    ran: ["docker stop"]         # if it was refused, this command must NOT have run
  - name: stop runs docker stop
    action: stop
    screen: detail
    params: { id: aaa111 }
    confirmed: true
    ran: ["docker stop aaa111"]
```

Every fake also logs its own command line (`docker stop aaa111`), which is what `ran` / `not_ran` look at. A `sudo` fake that just runs its command is added automatically; `sudo -n -- cmd` shows in the log.

## Keys of a check

A check is either a **screen** read or an **action** press.

| key | for | meaning |
|---|---|---|
| `name` | both | label in failure messages |
| `screen` | both | the screen to read; for an action, the screen the button is on |
| `params` | both | the screen's parameters |
| `contains` / `not_contains` | screen | substrings of the screen's JSON (what the phone would get) |
| `offers` / `not_offers` | screen | ids of buttons that must / must not be shown |
| `errors` | screen | a data source is expected to fail with a message containing these. Without it, any source error fails the check |
| `action` | action | the button to press |
| `form` | action | answers to its form |
| `confirmed`, `confirm_text` | action | what the person did in the confirmation |
| `refused` | action | the agent must refuse with a message containing this, and run nothing |
| `ran` / `not_ran` | action | substrings of commands that must / must not have been executed |
| `output` | action | substrings of the action's output |
| `fails` | action | the command must run and exit non-zero (its output is still checked) |

## What every fixture should prove

- The main screen shows the numbers and names the real tool would produce (copy real output into the fake).
- Each button is offered when it should be and not otherwise (`offers` / `not_offers`).
- A dangerous or confirmed button **refuses** without confirmation and without a typed name, and then runs exactly the command you expect (`refused`, `ran`).
- A form rejects bad input (`refused: not in the right format`).

## When a check fails

The failure prints the screen JSON that was produced, so you can see what the fake produced and fix the expression, parser or fixture.

## Real tools

`ext try` (see [building.md](building.md)) runs the real commands. Do both: fixtures keep it correct over time, `ext try` proves it works on the real tool's output. If you can, note in the PR which version of the tool you tried.
