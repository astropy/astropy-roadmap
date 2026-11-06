# The Astropy Roadmap

The roadmap is a set of objectives, each with measurable key results, maintained by the
Astropy Strategic Planning and Organizing Committee (SPOC).

## For readers

- **Read the roadmap:** https://astropy-roadmap.readthedocs.io
- **Objectives in priority order:** the [project board](https://github.com/orgs/astropy/projects/16)
- **Discuss an objective or key result:** comment on its issue, linked from the roadmap
  page, or browse the [objectives](https://github.com/astropy/astropy-roadmap/issues?q=label%3Aobjective)
  and [key results](https://github.com/astropy/astropy-roadmap/issues?q=label%3A%22key+result%22)

## For the SPOC

The roadmap lives in the issues of this repository. Each objective is an issue with the
`objective` label whose title starts with its identifier, such as `O3: ...`, and each of
its key results is a sub-issue with the `key result` label, titled `KR3.1: ...`,
`KR3.2: ...` and so on. Identifiers are chosen by hand, take the next unused number, and
are never renumbered or reused.

To add an objective:

1. Open an issue titled `O<n>: <objective>`, using the next unused number.
2. Add the `objective` label.
3. Add it to the project board and drag it to its rank. Every open objective must be on
   the board.

To add a key result:

1. Open an issue titled `KR<n>.<m>: <key result>`, where `<n>` is its objective's number
   and `<m>` the next unused number under that objective.
2. Add the `key result` label.
3. Make it a sub-issue of its objective.

Details that would make a key result's title too long can go in its issue description,
which is shown under it, in italics, on the page. Status updates and links to related work
belong in comments on the issue, which are not published.

Close an issue as completed when its entry is done, and as not planned when it no longer
applies; a not-planned entry stays in the roadmap, marked as such, as a record. An issue
opened by mistake is taken out of the roadmap by removing its label.

Every Monday, and whenever it is run by hand from the Actions tab, a workflow records the
issues and the board ranking in `roadmap.yaml` and opens a pull request with the changes.
Merging it updates the published page. Titles and key result descriptions are published as
written, including any HTML or Sphinx markup, and anyone who opened an issue can edit its
description later, so check the diff of `roadmap.yaml` before merging. Warnings for things
that are allowed but look like mistakes (an issue with an identifier but no label, a
skipped number, an objective with no planned key results) are listed in the pull request.
If the issues are inconsistent (a labelled issue's title has no identifier, a key result
is not a sub-issue of an objective or its identifier does not match that objective, an
identifier is used twice, or an open objective is not on the board), no pull request is
opened; the problems are reported in a "Roadmap snapshot failed" issue instead, which the
next successful run closes.

The wording of the page around the objectives, such as the introduction, is in
`text.yaml`, and is changed by pull request.

## For repository maintainers

`python roadmap.py` reads the issues and the board through the GitHub GraphQL API, using
`gh`, and writes `roadmap.yaml`. The repository and board it reads are set at the top of
`roadmap.py`; the section at the end of the page in `text.yaml` links to both, so update
it too if either changes. The documentation build calls `roadmap.render()`, which combines
`roadmap.yaml`, `text.yaml` and `templates/roadmap.md` into the page; nothing generated is
committed. Running `make html` in `docs/` builds the page locally, without GitHub access.

The workflow in `.github/workflows/roadmap.yml` needs two things set up in the repository:

- **Settings → Actions → General → Workflow permissions:** tick "Allow GitHub Actions to
  create and approve pull requests". If it is greyed out, an organization owner must first
  allow it in the astropy organization settings.
- **The `ROADMAP_TOKEN` secret**, described below.

The default workflow token cannot read project boards, so `roadmap.py` runs with the
`ROADMAP_TOKEN` secret, which it uses to read both the issues and the board. The board is
private, so the token can only read it if the account that creates the token can see the
board.

1. As an astropy member with access to the board, go to
   https://github.com/settings/personal-access-tokens/new to create a fine-grained token.
2. Set the **resource owner** to astropy. If astropy is not offered, the organization does
   not allow fine-grained tokens; see the alternative below.
3. Set the **expiration** to the longest the organization allows, and note the date: when
   the token expires, the weekly run fails and opens a "Roadmap snapshot failed" issue.
4. Under **repository access**, choose "Only select repositories" and pick
   `astropy/astropy-roadmap`, with **Issues** set to read-only.
5. Under **organization permissions**, set **Projects** to read-only.
6. Generate the token. If the organization requires approval for fine-grained tokens, an
   organization owner approves it under the organization's Settings → Personal access
   tokens → Pending requests.
7. Check that it can read the board; the reply should give the board's title and the
   number of items on it:

   ```
   GH_TOKEN=<token> gh api graphql -f query='{ organization(login: "astropy") { projectV2(number: 16) { title items(first: 1) { totalCount } } } }'
   ```

8. In this repository, go to Settings → Secrets and variables → Actions and add a
   repository secret named `ROADMAP_TOKEN` with the token as its value.

If the organization does not allow fine-grained tokens, a classic token with only the
`read:project` scope, created at https://github.com/settings/tokens/new, also works.

The token belongs to the account that created it, so it stops working if that account
loses access to the board. Replace it when it expires or changes hands by creating a new
one and updating the secret.
