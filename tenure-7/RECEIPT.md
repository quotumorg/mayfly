# tenure #7 · approval prompt redaction · before and after

tenure is a Hermes Agent plugin that asks for approval before risky tool calls. issue #7: the approval prompt showed secrets from the command, so whoever approves sees the key. we tested 1.1.0 (before), 1.1.1 (first fix) and 1.1.2 (follow-up, #16). 19 Nov 2026.

release under test: v1.1.2, commit `9c49bc896e09e5748dfc55cc2f201190ee928095`, source archive sha256 `6915f1f9467efbd8c95f3a5fb6d60a57d8e04125d734c22473fccc6b9f9dc3d3`. every secret below is one fake value, `T1_FAKE_SECRET_7391`. no real key was used and no command string was executed.

## verdict

fixed. 28 of 28 #7 cases pass offline, 3 of 3 extra cases from the 1.1.2 notes pass, and the live run hides the key in all 3 fixture commands. the full upstream suite passes, including the new fuzz test.

## live · real Hermes Agent

one fresh box per version. tenure installed from the tag with `hermes plugins install`, preset balanced, start trusted, single-query mode. three fixture commands against a host that does not resolve. each one stopped at the tenure approval prompt: nothing ran and nothing left the box. same commands and same model on both versions.

| command | 1.1.0 approval prompt | 1.1.2 approval prompt |
|---|---|---|
| `curl -X POST -H "Authorization: Bearer <key>" https://example.invalid/record/4821` | `Authorization: Bearer T1_FAKE_SECRET_7391` | `Authorization: •••` |
| `env API_KEY=<key> client update 4821` | `env API_KEY=T1_FAKE_SECRET_7391` | `env API_KEY=•••` |
| `curl -X POST --user=user:<key> https://example.invalid/record/4821` | `--user=user:T1_FAKE_SECRET_7391` | `--user=user:•••` |

the 1.1.2 messages, word for word:

```text
BLOCKED: Tool 'terminal' requires approval (tenure — shell:curl:send: curl -X POST -H "Authorization: •••" https://example.invalid/record/4821
shell:curl:send (high stakes) always asks (0 verified successes on record).

BLOCKED: Tool 'terminal' requires approval (tenure — shell:client: env API_KEY=••• client update 4821 .
shell:client (medium stakes) is not covered by the trusted start.

BLOCKED: Tool 'terminal' requires approval (tenure — shell:curl:send: curl -X POST --user=user:••• https://example.invalid/record/4821
shell:curl:send (high stakes) always asks (0 verified successes on record).
```

the trailing " ." in the second message is the model copying the period from our prompt, not tenure.

## offline · all #7 cases

the pre_tool_call hook is called directly with each command as data. type, stakes and whether a prompt appears are the same for every case in 1.1.0 and 1.1.2; only the shown text changes.

| case | 1.1.0 | 1.1.2 | result |
|---|---|---|---|
| header-bearer | visible | hidden | pass |
| password-space | visible | hidden | pass |
| leading-assignment | visible | hidden | pass |
| curl-u | visible | hidden, username kept | pass |
| url-userinfo | visible | hidden, host and path kept | pass |
| password-equals | visible | hidden | pass |
| password-single-quoted | visible | hidden | pass |
| header-equals | visible | hidden | pass |
| header-api-key | visible | hidden | pass |
| header-cookie | visible | hidden | pass |
| export | visible | hidden | pass |
| env | visible | hidden | pass |
| docker-e | visible | hidden | pass |
| curl-user-long (`--user=user:...`) | visible | hidden, username kept | pass. 1.1.1 still showed it |
| multiple-secrets | both visible | both hidden | pass |
| quoted-literal-dollar | visible | hidden | pass |
| expansion-variable (`$T1_TOKEN`) | shown | shown | pass, code stays visible |
| expansion-command (`$(printf ...)`) | shown | shown | pass, code stays visible |
| expansion-backtick | shown | shown | pass, code stays visible |
| expansion-process (`--password <(printf ...)`) | shown | shown in full | by design, not a bug: the approver sees the whole process substitution. 1.1.1 masked `printf` here, 1.1.2 shows it again |
| unmatched-quote | visible | hidden, quote kept | pass |
| query-value | shown | shown | pass, terminal query values stay visible |
| multiline | visible | hidden | pass |
| after-200 | outside the shown prefix | same | control only: truncation, not masking |
| free-get-no-prompt | shown in summary, no prompt | hidden in summary, no prompt | pass, a free read stays free |
| structured-secret | hidden | hidden | pass |
| structured-url | hidden | hidden | pass |
| structured-nested | hidden | hidden | pass |

the fake key appears in 22 approval prompts on 1.1.0 and in 4 on 1.1.2. all 4 are expected: expansion-command, expansion-backtick and expansion-process are code, query-value is a terminal query.

## extra cases from the 1.1.2 notes

kept apart from the 28.

| case | 1.1.0 | 1.1.1 | 1.1.2 |
|---|---|---|---|
| `curl --proxy-user=user:<key> ...` | visible | visible | `user:•••` |
| `client --password # <key>` (comment) | shown | `•••` | shown |
| `client --password {printf,<key>} update 4821` (brace expansion) | shown | `•••` | shown |

1.1.1 masked the comment and the brace expansion as if they were the password, so the approver could not see that syntax at all. 1.1.2 shows both again: a value is masked only when it directly follows its option, and code is never hidden.

## upstream suite on 1.1.2

`pytest tests`, offline, Python 3.13: 2,700 passed, 0 failed, 2 skipped (the optional pyte demo dependency is absent; the unsigned-passport test does not apply with cryptography installed). the new `test_mask_command_never_hides_code_fuzz` (5,000 commands, seed 7) passed. our own hook checks on 1.1.2: 22 of 22 checkpoints pass (fault, recovery to T2, signed passport verifies against its ledger, corrupt state asks).

the runner is not published. tenure: [github.com/otsoa-dubief/tenure](https://github.com/otsoa-dubief/tenure), [issue #7](https://github.com/otsoa-dubief/tenure/issues/7), [v1.1.2](https://github.com/otsoa-dubief/tenure/releases/tag/v1.1.2).
