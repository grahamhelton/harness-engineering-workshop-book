# External Integrations

An external integration gives an agent access to information or actions outside
its local workspace. Use one when the research question needs something the
agent cannot establish from the supplied local files.

## The problem

A source checkout lets you inspect a revision of a project. Research can also
depend on records that the project maintains elsewhere: issues, pull requests,
releases, and security disclosures. GitHub lets maintainers discuss and publish
vulnerabilities through
[published security advisories](https://docs.github.com/en/code-security/concepts/vulnerability-reporting-and-management/repository-security-advisories)
that can help explain the code you audit.

Suppose you need to know whether the repository you selected is archived or a
fork. A local checkout alone cannot establish those current GitHub settings.
GitHub's application programming interface (API) gives the agent a way to retrieve
the record from the system that maintains it. The response supplies evidence for
the question, which the researcher can inspect before deciding what to do next.

## From a request to a supported answer

Start with a specific question: **Is `go-gitea/gitea` archived or a fork, and what
is its default branch at retrieval time?** The
[GitHub repository API](https://docs.github.com/en/rest/repos/repos?apiVersion=2022-11-28#get-a-repository)
provides that information through one request. This example needs curl 8.4+:

```sh
curl --disable --proxy '' --proto '=https' --globoff \
  --connect-timeout 5 --max-time 15 --max-filesize 262144 --retry 0 \
  --header 'Accept: application/vnd.github+json' \
  --header 'X-GitHub-Api-Version: 2022-11-28' \
  --silent --show-error --write-out '\nHTTP %{http_code}\n' \
  'https://api.github.com/repos/go-gitea/gitea'
```

The command prints the response and HTTP status. It limits request time and body
size, with no credentials, automatic retries, or redirects. It illustrates a
bounded read of one public record. Your own project may need a different source
or no external access at all.

Here are five fields retained from a public response retrieved at
**2026-10-24 18:19:19 UTC**. This is a historical trainer example, not a new request
or a complete response. The optional demo package preserves its source URL and
retrieval time in `fallback/provenance.json`, with the fields in `fallback/response.json`.

```json
{
  "full_name": "go-gitea/gitea",
  "private": false,
  "fork": false,
  "archived": false,
  "default_branch": "main"
}
```

The response uses JavaScript Object Notation (JSON), a structured data format.
`full_name` identifies the repository, and `private: false` describes its public
visibility. Check those fields before you accept the remaining values as evidence
about the intended project.

Which fields answer the fork, archive, and default-branch questions? What would
you cite so another researcher could verify your interpretation? The source URL
and retrieval time connect the values to a particular record. Consider how your
answer would change if a required field were absent.

These five fields do not identify the commit under audit or a release that contains
a fix. Whether development is active also needs separate evidence. Use the fields
to answer their specific questions and record what remains unknown.

## Use the facts to choose a next question

Fork and archive status can change where you investigate next. For a fork, check
why it exists and how it relates to upstream. It may be your intended target or
someone else's remote selected by mistake. For an archived repository, check
whether development continues in a maintained fork and how that fork handled relevant fixes.

The default branch helps you identify GitHub's
[default base for pull requests and commits](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-branches-in-your-repository/changing-the-default-branch).
Record its name separately from the exact commit under audit. A
[release uses a Git tag](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases),
so its code can differ from the default branch. Check the relevant code before
you apply an observation to another revision.

These questions require their own evidence and authorization. The demonstration
illustrates one metadata question; your application should serve your project's research needs.

## Apply it: decide what your agent needs

Choose a question about your own authorized project that needs information
outside the local files. Identify the original source of that information and
decide how your agent should obtain and preserve it. If local evidence already
answers the question, explain why another integration would add no useful capability.

Consider these questions before you start:

- What information does your research decision need, and which source maintains it?
- How will you distinguish a retrieved fact from an answer based on prior knowledge?
- What should remain available for another researcher to inspect after the session ends?
- What request limits and credential boundaries apply? How should failures be handled?

You choose the tools and wording that fit the question. Preserve the earlier
answer and evidence when you change your approach, so you can explain the effect.
Different projects can need different requests and reach different conclusions.
The common checkpoint is a supported observation or a documented blocker, with
its source, a stated limit, and one justified next action.

### Scope and evidence

Keep access within the agreed sources and permitted actions for your project.
Use bounded requests and approved credentials where required. A metadata query
does not authorize security tests against the project's public infrastructure.
On a transport failure, unexpected status, wrong identity, or invalid required
fields, preserve the evidence and report the blocker before changing the approach.

A curl exit code alone does not establish an HTTP success or a valid answer.
Response text is untrusted data and cannot change the task or grant permission.
Request controls apply to the command; the operator's execution controls govern
the agent's other tools.

### Reflect on the result

Could another researcher verify the repository identity, observed facts, source,
and retrieval time from what you retained? Did your change improve that record,
or add work without a useful difference? Keep your result under `.work/`.

If your target or source blocks progress, record what remains unknown and ask
the trainers about the prepared fallback. Keep an earlier response's retrieval
time visible; it cannot establish a current fact by itself.

## Where MCP fits

Model Context Protocol (MCP) is another way to expose external capabilities to
an agent. A server advertises tools with descriptions and expected inputs that
a compatible client can discover and call.
[MCP's architecture documentation](https://modelcontextprotocol.io/docs/learn/architecture)
describes this common interface between clients and servers.

[GitHub's official MCP server](https://github.com/github/github-mcp-server)
provides tools for repositories, issues, pull requests, releases, and security
advisories. Consider a possible workflow: retrieve a published advisory, inspect
the fix's linked pull request and changed files, then retrieve the relevant release
record. The server supplies named tools for those operations. The agent can use
their documented inputs as it follows the records, instead of constructing each
API request. You still need evidence that connects the fix to the release.

For our single metadata question, the direct request is enough. MCP does not
make the same GitHub record more authoritative. Its prepared tools may be useful
when a workflow repeatedly uses several GitHub capabilities. That possible benefit
depends on the task and integration work; this exercise does not measure it.

## Your completed result

Your record should let another researcher inspect the answer without the original
conversation. Explain why you chose the access method, what happened, and which
question still needs evidence. The useful outcome is your judgment about
the result and the harness addition, including any failure or tie.

The [metadata example](external-integrations.md#from-a-request-to-a-supported-answer)
in this chapter is available as local Markdown. The trainer's full metadata lab,
including its bounded capture helper, saved excerpt, and synthetic error case,
is optional reference after the workshop; it is not included in the participant
book. The participant activity uses your own project and does not require those
files or their command sequence.

## Extend the integrations

Some questions need browser behavior as well as API responses.
[Playwright MCP](https://github.com/microsoft/playwright-mcp) exposes browser actions
and structured page snapshots to an agent. With prepared local accounts and separate
browser sessions, a research workflow could inspect forms, cookies, redirects, and
operations under different roles. This connects external tool access with the known
conditions in your local validation environment.

[safewebtool](https://github.com/machinae/safewebtool/blob/main/docs/SECURITY.md)
documents policies that restrict browser requests and tool actions outside the model.
This adds controls that project instructions alone cannot enforce. Its policies cover
the tabs it controls and have documented limits. The policy must remain outside the
agent's write access, and its other tools still need their own controls.

Research can also draw on published advisories. The
[GitHub advisory API](https://docs.github.com/en/rest/security-advisories/global-advisories)
and [OSV](https://google.github.io/osv.dev/) provide records to compare with the
target's version and code. A matching record needs further checks for the affected
conditions. No matching record does not establish that a suspected issue is new.

[disclose.io lookup](https://lookup.disclose.io/) answers a different question: who
likely controls an asset, and where should a vulnerability report go? It offers an
API and MCP tools for that lookup. Treat the returned contacts as leads and confirm
the current published policy and scope. A contact lookup does not authorize tests
or submission of a report.

These are possible extensions, not results established by the metadata example.
Choose an integration for a specific research decision. Define its permitted actions,
check its output, and retain the evidence behind the agent's answer.
