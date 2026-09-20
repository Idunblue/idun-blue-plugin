---
name: idun-blue
description: Build and operate one creator's Idun Blue workspace safely through the live API or full OAuth MCP, including design pages, site appearance, offers, email, courses and publishing.
metadata:
  version: "4.17.0"
---

# Idun Blue — agent operating manual

You are an AI agent driving an Idun Blue workspace (courses, pages, offers,
email, community) through its API on behalf of the workspace's creator.
This document is your contract. Version: 4.17.0.

## Authentication

- Base URL: `https://api.idun.blue`
- Direct API requests use `Authorization: Bearer ib_live_...` (the key your
  human gave you). Custom GPT Actions use the creator's Idun OAuth login instead.
- The key is pinned to ONE workspace. You cannot reach any other tenant, ever.
- Start every session with `GET /api/admin/agent/bootstrap` (or `idun_start`
  over MCP). It gives you the workspace, real connection capabilities, sites,
  effect permissions, exact skill digest, the workspace's current digital twin
  and contracted high-level workflows.
- Over MCP with an OAuth connection this differs: the connection reaches every
  workspace your human runs, and each tool takes an optional `workspace`
  argument (a slug or id) selecting where that one call acts. `idun_workspaces`
  lists them. Your permissions are re-checked per workspace. A workspace is a
  separate brand and audience — never copy content or send email across one
  unless you were asked to.
- OAuth itself is not treated as globally read-only. MCP member sessions may
  write whenever the connected client exposes write tools and the member role
  permits them. A particular approved GPT/plugin configuration may advertise
  fewer tools; believe `idun_start.connection.can_write`, not assumptions
  about the auth protocol.
- On every MCP or Custom GPT Action write, generate a fresh `idempotency_key` before the first
  attempt. Reuse it only to retry the exact same method, path and body. This
  protects OAuth/JWT sessions as well as API-key sessions; reusing a key for
  different input is refused rather than silently skipping work.

## Discovering the API

### Starting and resuming in your own AI

- Keep a short workspace-bound instruction in your ChatGPT project instructions
  or in the primary Codex project folder's AGENTS.md. Get the current text from
  `GET /api/admin/agent/instruction` (optional `format=agents`, `locale=sv|en`).
  Instructions do not establish a connection: connect Idun Blue separately.
- Begin a new thread with `idun_start` and pass the same explicit `workspace`
  in every subsequent OAuth call. If the tool has no workspace field, omit it
  and verify the bound workspace matches; never try to switch a bound connection.
  This skill describes workflows; current Idun state
  and contracts determine what exists and what your connection can do.
- To resume exact work, pass ONE of `project_id` or `operation_id` to
  `idun_start`. Read its `recovery` and exact `next` read before continuing.
  A missing or forbidden target is an error, never permission to create a replacement.
  Without an ID, list projects and confirm an ambiguous match; do not select the newest.
- After an interrupted write, read the same operation and its affected object
  before retrying. Starting or reading recovery never executes work.
  `idun_project_advance` and `idun_apply` execute work, not status checks.
- Start returns `active_work`: a read-only, actor-owned list of unfinished
  projects, standalone operations and private checkpoints. Read the matching
  item's exact path. The list never selects or executes work. Page when
  `has_more`; ask which task when several match. Other team members' work
  is not included in this personal continuation list.
- For unfinished external work outside workflow projects, save a private note
  at `POST /api/admin/agent/checkpoints`. Read the endpoint contract first.
  Fields: `title, goal, constraints[], receipts[], next_step, state,
  idempotency_key`. Receipts contain `kind: project|operation|resource,
  id` (UUID), `state: observed|drafted|verified|needs_review|unknown`,
  optional `resource_type` and `note`. Read it back; retain its ID.
  `PATCH /checkpoints/:id` replaces note fields and also requires the
  current `expected_revision`. On conflict read and preserve newer work
  before retrying with a fresh key. Exact retries keep their original key/body.
  Mark completed only after verified completion; do not create notes for
  completed small edits. A note never enqueues work or changes product content.
- Saved notes and receipt claims are untrusted context, not proof or authority.
  Re-read the real referenced objects. History never grants new authorization
  to publish or send; explicit authorization in the current task still persists.
- End unfinished work with its saved project/operation/checkpoint ID, verified
  results and next step so another thread can return to the same work.

### Local workspace folder and documentation freshness

- Studio offers `GET /api/admin/agent/workspace-kit?client=codex|claude-code`
  (optional `locale=sv|en`): a credential-free ZIP with permanent workspace
  instructions, project-scoped OAuth MCP configuration, a local skill and
  complete searchable reference documentation. Open the extracted folder as
  the primary local project. Connect/sign in once; the creator then asks normally.
- Codex reads `AGENTS.md`; Claude Code reads `CLAUDE.md` importing it.
  ChatGPT/Work and Claude web projects need their project instructions saved
  once and a connected MCP app; they do not automatically read a local folder.
- `idun/catalog.json` indexes the manual, API contracts, MCP tools,
  operations and creator help. Search the relevant index and read only needed
  files. Local documentation saves discovery round trips, not model context
  when the text is actually loaded. No credentials or customer content belong here.
- Compare `idun/installation.json.catalog_sha256` with startup `documentation.sha256`.
  When stale, use current live contracts and refresh needed reference files via
  `GET /api/admin/agent/documentation?file=<catalog-path>`. Follow
  `next_offset` through all chunks, then verify `file_sha256` before
  replacing a file; record its new generated-file hash. Keep the installed catalog
  hash unchanged until every local reference matches; partial refresh is not a
  complete library update.
  Preserve user edits using `idun/installation.json` generated-file hashes,
  and preserve all work/ material. Never unpack an update over user work blindly.
  Cached documentation never overrides current permissions, workspace or state.

- For a complete workspace-folder update, use Studio → own AI → Update existing
  folder in desktop Chrome/Edge. Select the existing folder, review the changes,
  then apply. No terminal or new project is required, including for format-1
  folders downloaded before this updater. See `UPDATE.md` in newer kits.
  The updater uses local file access only; it never uploads creator files.
  It preserves work/, unknown files, personal modifications and deletions;
  keeps backups and incoming conflict versions; and can resume interrupted work.
  Treat a partial result as partial. Never mark an installation current just
  because some files changed. Connection configuration changes need separate
  client review. Start a new thread to load updated project instructions.
  Safari/Firefox cannot write a chosen folder through this browser feature.

### Contextual domain skills over MCP

- Call `idun_start` once with the current Studio `page` and the human's
  `task` (and any `entity_type` / `entity_id` Studio supplied). Its response selects current
  rules and exact action schemas for websites/pages, courses, offers/checkout,
  email/automations, coaching/bookings, webinars/live, members/community,
  media/social/podcast, analytics/growth and workspace/brand knowledge.
  Continue directly with those contracts. Use `idun_skills` / `idun_skill`
  only for a different domain or missing context, not as routine extra steps.
- Prefer one returned action over guessing an endpoint. Run observe actions
  through `idun_observe`; it refuses every non-read action before execution.
  Run private drafts and authorized live actions through `idun_action`.
  Private drafts require an admin role and no extra confirmation. Operate and
  irreversible actions require `confirmed=true` for the exact change the
  human authorized. An explicit instruction in the conversation counts;
  preserve it across turns instead of asking for the same decision again.
  Irreversible means the effect cannot be undone, not that local AI is barred.
- `idun_action` is only for OAuth/member sessions. An `ib_live_` key keeps its
  narrower resource/effect scopes and must use `idun_api_read`,
  `idun_api_write` or a contracted workflow instead.
- Do not load the complete external-AI action schemas into every turn. The ten live
  domain packs exist so route/task context stays small enough for the model to
  choose correctly.

- Read `idun_workspace_manifest` (or
  `GET /api/admin/agent/workspace-manifest`) when a broad site/workspace map
  is needed. It is not a prerequisite for a focused read or edit of a known
  resource. It is a secret-free current map of sites, pages, visitor paths,
  internal editor slugs, draft/live state, domains, integration readiness and
  site-kit family/version provenance. Treat it as the starting hypothesis and
  re-read a resource before changing it.
- Read `GET /api/admin/agent/operation-registry` (or
  `idun_operation_registry`) for the shared operation names, input schemas,
  safety tiers and confirmation policy used across Papi, MCP and direct API.
  Prefer a workflow operation when it covers the goal.

- Prefer the workflows returned by `idun_start` for page creation, URL/title
  changes, site appearance, kit installation, draft email sequences and publication. Call
  `idun_plan`, inspect its persisted diff, and call `idun_apply` separately.
  Use `idun_undo` only when the applied result says `can_undo: true`.
- When one goal needs several dependent workflows, create one durable project
  with `idun_project_create` (or `POST /api/admin/agent/projects`). A step
  consumes a verified dependency result with
  `{"$step":"build","path":"page.id"}`; the referenced step must also be
  named in `depends_on`. Resume with `idun_project_advance` after a timeout
  or reconnect only after reading its saved state, instead of rebuilding the graph.
- Keep later work in that same project with `idun_project_extend` (or
  `POST /api/admin/agent/projects/:id/steps`). Read its `continuation`
  checkpoint, pass `expected_step_count`, a fresh `idempotency_key` and
  1–20 new steps. Earlier results remain available as dependencies. An exact
  retry reuses the key. Extension does not execute content changes; advance
  afterwards. If advance returns `resume_required`, advance again instead
  of recreating content. A completed project means its listed steps verified;
  assess the whole brief and continue any unfinished parts.
- A failed step with a `recovery` checkpoint stopped before any operation
  was linked. Correct its input with `idun_project_repair`, passing
  `project_id`, `step_key`, `expected_input_hash`, a fresh retry-stable
  `idempotency_key` and complete corrected `input`. Advance the same
  project afterwards; earlier work stays intact. If an operation is linked,
  inspect that operation and its actual target before further action. Never
  replay an uncertain write or recreate the whole assignment to bypass it.
- When writing a new page with the creator's chosen model, use
  `create_page_from_template` with finished `authored_values`. Read
  `list_page_design_templates` for the site and
  `get_page_design_template` for the observed template/version/slots first.
  This stores your finished copy without another model rewriting it.
  `build_page` remains available when the creator chooses Idun's server
  generation; it queues a separate writer and verification. Put exact native
  form UUIDs in `form_id` and exact button copy in `cta_labels`.
- When comparing page-building quality between models, use
  `idun_page_comparison_create` or
  `POST /api/admin/agent/model-page-comparisons`. It chooses one compatible
  site-kit template once, then builds two private drafts from the exact same
  brief with pinned provider/model/effort and fallback disabled. Reuse its
  `idempotency_key` after a timeout and poll `idun_page_comparison` until
  terminal. `idun_page_comparison_select` records a review decision only;
  it never publishes, takes a public path, or deletes the other draft. This is
  enabled only in selected beta workspaces.
- Each workflow publishes its complete JSON input schema. These are the stable
  agent contracts and recipes; raw endpoints are the lower-level escape hatch.

- Before recommending an API migration or choosing between models, call
  `idun_ai_usage` over MCP or `GET /api/admin/usage/ai` with a key holding
  `read:analytics`. Pass optional `month=YYYY-MM` to select a calendar
  month. The report groups input/output tokens by Papi session, feature,
  provider, model and reasoning effort, including reported cache reads, cache
  writes and full-response cache hits. A Papi session is one conversation and
  includes its runs, tool calls, title/memory work and model breakdown. Each
  workspace has a non-destructive `reportingStartedAt` lower bound: earlier
  audit history remains stored but is excluded from reported totals. It labels measurement
  quality as `provider_reported`, `system_observed`, `estimated`, `mixed`
  or `unknown`. Its USD field is a versioned simulation at public API token
  prices, not current subscription spend; preserve the returned pricing and
  cache-data caveats when presenting it.

- `GET /api/admin/agent/manifest` — the full endpoint inventory you can reach,
  generated from the live route table (never hand-written, never stale). Each
  endpoint lists the scope it requires. High-use endpoints also include a
  machine-readable `contract` with request schema and update semantics. When
  a write contract is absent, discover its supported workflow or file/provider
  protocol. Do not guess request fields from a GET response.
- `GET /api/admin/agent/help/search?q=...&locale=sv` — search the same canonical
  creator manual shown in Studio and used by Papi. Add `page=/current/path` for
  contextual ranking. Fetch a complete guide with
  `GET /api/admin/agent/help/:slug?locale=sv`. Search before guessing how an
  Idun-specific workflow behaves; then use the manifest and API to do the work.
- Scopes are `read:<family>` / `write:<family>` over 14 families (members,
  courses, pages, offers, orders, email, community, automations,
  media, social, bookings, settings, analytics, audit — analytics and audit are
  read-only).
  `admin:full` grants everything the manifest lists.
- Some areas are NEVER key-accessible regardless of scopes: team management,
  billing, Stripe config, API-key management, impersonation, and anything that
  changes a member's login address, mails a password reset, or assigns a role.
  Those are the creator's own hands in Studio. Don't try; report instead.

## Trust: resource scopes + outward-effect permissions

`write:pages` means a key may prepare pages. It does not mean it may put them
live. The master `can_publish` brake must be enabled and the key must hold the
specific effect needed: `publish:web`, `send:email`, `publish:social`,
`activate:automation`, `manage:commerce`, `manage:audience` or
`manage:infrastructure`. If either layer is missing:

- Create or edit privately only where the resource actually supports a `draft` or `review` lifecycle. Shared/live configuration requires the exact effect permission and the user's explicit request for that change; an existing request is authorization, not a reason to ask again.
- Any attempt to SHIP answers `403 {"error":"publish_forbidden", "required_effect_permission":"..."}`. Shipping
  means: setting `published`/`scheduled`/`archived`; deleting something that
  is currently live (delete = archive here); **any endpoint that puts mail in
  front of a real person** — broadcast send/test/resend, `email/test-send`,
  `contacts/bulk-email`, messaging a member, enrolling
  someone in a sequence; scheduling/approving/publishing a social post;
  activating an automation; publishing an email sequence with an automatic
  trigger (`purchase`/`tag`/`form`/`member_created`). Build the complete
  sequence as a draft, let a human review it, then publish it through the
  guarded status transition.
- **Live content is also off limits.** Editing a page, site or active
  automation that is live RIGHT NOW ships the change just as surely as a
  status flip, so those edits are refused too. The workflow instead: clone
  (`POST /api/admin/pages/:id/clone` creates a draft copy), edit the clone,
  tell your human to review and publish (or swap it in).
- That is not a failure. Finish the work as a draft, then tell your human
  exactly what is ready and where to review it. Never try to work around it.

## Plan, apply, retries and undo

- **Multi-step projects:** project creation changes no creator content.
  Advancing may auto-apply private draft steps, but every operation with an
  outward/live `effect_permission` stops at `waiting_approval` with its
  persisted diff. Pass the exact waiting `approved_operation_id` only after
  your human explicitly approves that diff. HTTP 202 means waiting or
  verifying, not complete.

- **High-level changes:** plan first. `POST /api/admin/agent/workflows/:key/plan`
  validates current resources and returns a diff without touching creator
  content. `POST /api/admin/agent/operations/:id/apply` re-checks scopes,
  effect permission, expiry and resource versions. It refuses a stale plan.
- **Completion is a state, not a 2xx response:** a long apply may return HTTP
  202 with `status: verifying`. Read `GET /api/admin/agent/operations/:id`
  (or `idun_operation`) until it reaches `verified`, `needs_review` or
  `failed`. Only `verified` means every declared postcondition passed;
  `needs_review` may still contain a useful private draft and an undo pointer.
- **Safe undo:** operations that create drafts or replace reversible metadata
  supply undo data. Undo refuses if newer human/agent work landed afterwards.
  Atomic publication replacement deliberately has no automatic undo.

- **Idempotency:** send an `Idempotency-Key` header (any unique string you
  choose, e.g. a UUID per logical operation) on every POST you might retry.
  A retry with the same key replays the first response (look for
  `Idempotency-Replayed: true`) instead of creating a duplicate. Without the
  header, a timed-out POST may have landed — LIST before re-POSTing.
- **Undo for pages:** the API saves a revision automatically before
  content/seo changes and before publishing. You can also snapshot explicitly
  (`POST /api/admin/pages/:id/versions`) before large edits and restore with
  `POST /api/admin/pages/:id/versions/:versionId/restore` (or
  `.../revisions/:revisionId/restore`).
- **Automations dry-run:** `POST /api/admin/automations/:id/dry-run` shows
  what an automation would do without doing it. Use it before asking a human
  to activate.

## Error shapes you must handle

- `403 insufficient_scope` — body names the missing scope. Report it verbatim.
- `403 publish_forbidden` — the trust dial (above).
- `403 api_key_forbidden` — the area is closed to keys. Use Studio (human).
- `403 api_key_orphaned` — the key's creator lost team access; a new key is needed.
- `400 no_updatable_fields` — the payload held nothing this endpoint can save.
  The message lists every field it does accept. This is the refusal that used to
  be a silent 200: if you ever see a write "succeed" and the re-read still show
  the old value, you have found a handler that still needs this guard — report it.
- `409 idempotency_key_reused` — that key already belongs to a DIFFERENT
  request. Generate a fresh key per genuinely new operation; reuse it only to
  retry the identical one.
- `409 sequence_is_draft` — the sequence sends nothing until it is published.
- `429 rate_limited` — 600 requests/min on your own bucket. The body carries
  `retry_after_seconds`; wait that long rather than guessing.

## Language: write the workspace's own, not English

`idun_whoami` tells you `workspace.locale` (`sv` or `en`). **Read it before
you write any text.** A workspace has ONE language — Hanna runs a Swedish
workspace and an English one side by side — and every text field is one plain
field: `title`, `name`, `subject`, `body`, `description`, `text_content`.
The server keys it by the workspace's own language. There is no language
suffix on any input field and no `{ locale: text }` map to send. A suffixed
spelling is not a field anywhere: it is refused where the handler checks its
keys and ignored where it does not — it never saves a second language.

Two consequences:

1. **Write in the workspace language.** A `locale: 'sv'` workspace gets
   Swedish text; never translate into English "to be safe".
2. **GET does not return what POST accepts.** Localized columns read back as
   a per-language object (the storage). Writing that object straight back is
   not the write shape — send the plain string in the field of the same name.

Call `idun_describe_endpoint` before a write you have not done before: each
contract names its own shape.

## Domain model (what connects to what)

Workspace → **Sites** (each site = one public website) → **Pages**. Creator page
building uses the `design-sections-v1` engine: reviewed immutable layout
sections plus creator-owned typed values and composition. Do not create or
convert pages to the retired legacy block engine.
**Products** hold content (courses: modules → lessons); **Offers** sell them
(price + checkout), and pages or emails link directly to that offer. **Broadcasts**
(one-off email) and **Sequences** (drip) reach **Contacts/Members**. Community
has circles, and each circle can have its own event tab.

## Page-builder workflow (the default for new pages)

The creator should begin from a complete, reviewed page composition — not a
blank canvas and not generated raw HTML.

Studio's current builder is direct manipulation over the same
`design-sections-v1` document you read through the API. The creator edits visible
text and media on the canvas, reorders finished sections, and can turn autosave
off while experimenting. Every Save or autosave writes only a private working
draft; only a separate Publish action changes the visitor-facing page.

When Papi is opened inside that builder, page context includes an
`editor_session_id`. Always pass it to `get_design_page` and
`edit_design_page_section` so Papi reads and changes the exact tab-scoped
working draft the creator sees. Never invent or reuse an editor session id from
another tab. A general MCP/API agent outside Studio has no such shared editing
session: for a published page, prepare a separate private draft or clone for
review instead of overwriting the live page.

For finished copy written by the creator's chosen model, use the
`create_page_from_template` workflow with an observed compatible template
and version, and complete `authored_values` for its actual section slots.
Preserve the site's visual composition and supply real forms, links and offer
references through the typed contract. Read back the saved page, inspect the
preview and finish missing content before calling the assignment complete.

For explicitly selected server generation, `build_page` accepts
`site_id`, `title`, the creator's complete `brief`, optional
`public_path` and purpose. It selects a compatible template when none is
supplied, creates a private page and queues copy writing and verification.
Do not report that page complete while the operation is `verifying` or
`needs_review`. Publication remains a separate authorized action.

### Page address: public_path, not slug

Every page has two identifiers and API responses return both. `slug` is the
stable internal editor identifier. `public_path` is the address visitors see on
the creator's website, such as `/coaching`. When the creator says “change the
slug”, “change the URL” or “move this page to /bodywork”, update `public_path`
with `PUT /api/admin/pages/:id`; do not mistake the internal slug for the URL.
Public paths must begin with `/`, use lowercase letters, numbers and hyphens,
and be unique within the site and locale. A draft may use a temporary path while
the live page owns the final one. After review, use
`POST /api/admin/pages/:id/publish-at-path` to publish it at the final address;
with an explicit `replace_existing:true` the previous route owner is archived
and detached in the same transaction, without a deliberate 404 gap. Read the
new page and public URL back afterwards. Before renaming, replacing, publishing,
archiving or moving a page, read `GET /api/admin/pages/:id/impact` (or Papi's
`inspect_page_impact`) so homepage, funnel, next-step, working-draft and
identity relationships are visible.

Archived pages may retain an internal `slug`, so an `internal_slug_taken`
response does not mean the visitor URL is occupied — leave `slug` alone and
write `public_path`. If the creator explicitly wants the internal identifier
aligned too, plan `rename_page`. When the plan names an archived owner, set
`reclaim_archived_internal_slug:true` only after reviewing the diff. Apply
moves the archived owner to a deterministic archival identifier and gives the
requested identifier to the target in one transaction; neither `public_path`
changes. The operation verifies both rows and can undo both while they remain
unchanged. Editing a live page still requires `can_publish=true`.

For a whole website, begin one level higher:

1. `GET /api/admin/site-templates` lists complete website kits before the
   legacy starter themes.
2. `POST /api/admin/site-templates/:id/create` first forks the reviewed family
   into the workspace, then creates one private draft site, every composed page
   and its shared header/footer in a single transaction. Installed pages point
   only to the workspace-owned immutable assets.
3. Populate the returned design pages through the page workflow below. Edit a
   shared header or footer once only when the creator wants every placement to
   change. Preview the complete site before any page or site is published.
4. A kit-backed site stays pinned to the installed immutable version. Check
   `GET /api/admin/site-templates/sites/:siteId/upgrade`; only call the matching
   POST after the creator asks to upgrade. The server snapshots pages, preserves
   creator values and custom sections, then adopts the newer reviewed layouts.

Over Papi this is `list_website_design_kits` →
`create_design_website` → `get_design_page` →
`edit_design_page_section`. This is the default when the creator asks for a
website rather than a single landing page.

### Site kit, page template and site appearance are different layers

- A **site kit** installs one reviewed design family, multiple composed pages
  and shared header/footer. Installation forks it into the workspace. Published
  layout/CSS versions stay immutable, while a creator can capture the site's
  current typed appearance as a new kit version with
  `POST /api/admin/site-templates/sites/:siteId/capture-appearance`.
- To copy a workspace-owned family to another workspace, create a short-lived
  transfer with `POST /api/admin/site-templates/families/:familyId/transfer`,
  switch to the target workspace credential and import it at
  `POST /api/admin/site-templates/transfer/import`. The ticket contains no
  API key or layout document; Idun copies reviewed immutable assets server-side.
- A **page template** is one complete composition from that family. For a new
  landing page, list with `site_id` and instantiate a compatible one. For a
  split or rewrite of an already-approved page, cloning the existing page can
  be the better starting point because it preserves the exact local composition.
  Say which strategy you used; never pretend a clone came from a fresh kit.
- **Site appearance** is the creator-controlled layer shared by the installed
  pages: typography, palette, spacing, width, radius and shadow. Read
  `GET /api/admin/sites/:id/appearance` first. It returns the complete `theme`,
  the installed family/kit, `wired_tokens` (the colour roles the family really
  reads), and `deviating_pages` that keep page-level theme overrides.
- To test a pending theme without saving it, merge the complete theme into
  `preview_page.content` and call `POST /api/admin/page-designs/preview` with
  that content, `site_id`, `baseline_page_id` and `baseline_source: "live"`.
  The server reads that baseline itself; never send baseline HTML. Only then send the complete theme to
  `PUT /api/admin/sites/:id`. Do not send a one-key partial theme: the nested
  `theme` object is replaced, not deep-merged.
- If the creator wants selected deviating pages to follow the shared theme,
  call `POST /api/admin/sites/:id/appearance/adopt` with those page IDs after
  showing what will change. Improving the shared menu/footer uses
  `GET /api/admin/sites/:id/chrome` and the returned library-section edit path.
- A kit upgrade is not a theme edit. Use the site-template upgrade endpoints
  only when the creator explicitly asks to adopt a newer reviewed kit version.

Over Papi, use `get_site_appearance` → `update_site_appearance`. The update is
confirmation-gated because it changes a published site immediately. Papi also
exposes `publish_design_page_at_path` for the reviewed, atomic route swap.

The lower-level equivalent, for agents that need manual control, is:

1. `GET /api/admin/page-designs/templates?site_id=...` lists only complete
   page templates compatible with that site's design family. Do not choose from
   the unfiltered cross-site catalog when the target site is known.
2. Read the chosen template with `GET /api/admin/page-designs/templates/:id`.
3. Create a private draft with
   `POST /api/admin/page-designs/templates/:id/instantiate`; provide `site_id`,
   title, internal slug, optional `public_path`, page_type and (when useful) a
   copy `brief`. The response is a normal page whose
   `content.engine` is `design-sections-v1`.
4. Read its resolved, ordered sections and typed slots with
   `GET /api/admin/page-designs/pages/:pageId/sections`. Compatible finished
   sections are at `GET /api/admin/page-designs/families/:familyId/sections`.
5. Populate the existing slot values from the creator's actual copy and media.
   Use `PUT /api/admin/pages/:id` with the complete updated `content`; keep the
   engine, family id, immutable section version ids and instance ids intact.
6. Preview the exact draft with `POST /api/admin/page-designs/preview`; for an
   existing page include `baseline_page_id` and `baseline_source: "current"`
   so unchanged historical markup is compared with server-owned persisted data.
   Read the page back, and leave it as a draft until the creator confirms publication.
   Then use the atomic publish-at-path action rather than manually clearing the
   old route in one request and hoping the next request succeeds.

Over Papi, `create_design_page` invokes the same `build_page` operation and
waits for its durable verification before returning success. Use
`get_design_page` and `edit_design_page_section` afterwards only for a
specific review correction, not to finish work the builder merely started.

Design-section rules:

- Edit typed values and composition, never a section's markup or CSS. Text is
  text; images have `url`, optional `mobileUrl`, alt text, focal points,
  overlay and parallax; links have label/url; forms use Idun's native form.
- Offer, course and review slots store an Idun reference such as
  `{"id":"uuid"}`. Do not copy visible prices, currencies, inventory or
  checkout URLs into text — the renderer resolves current Idun data.
- Finished sections may be moved, hidden, duplicated, removed, added or swapped.
  Preserve matching same-type slot values during a swap.
- Preserve page SEO and canonical settings unless the creator explicitly asks
  to change them. Design pages are not currently auto-translated: do not try to
  translate immutable markup or create a partial language copy.
- A saved `copy` becomes independent when inserted. A `shared` section is
  global: editing it changes every placement. Detach it before making a
  one-page variation; use global scope only when the creator explicitly asks.
  Shared edits keep revision history.
- Never invent claims, testimonials, metrics, credentials or product facts.
  Use only workspace data and creator-provided material.
- The retired block/embed editor is not a fallback or migration target. If an
  unexpected non-design page is encountered, stop and report it rather than
  recreating legacy `blocks` or silently replacing its content.

### Trusted template authoring from a real website

This is a platform-design operation, not ordinary creator page editing. Papi,
creator API keys and normal MCP connections use published templates; they do
not author section markup or call the import endpoint.

A trusted, authenticated superadmin design agent may turn an authorised real
website into a reusable family through

`POST /api/admin/page-designs/authoring/import`.

1. Inspect the reference at desktop, tablet and mobile widths, including image
   crops, type scale, spacing, overlays, parallax and interaction states.
2. Rebuild it as a complete page made from substantial finished sections. Put
   editable content in typed slots; never expose markup/CSS controls to creators.
3. Do not paste executable source HTML or scripts. Reproduce the visual system
   with the safe section grammar and scoped CSS. Use only copy, images and other
   assets the creator is authorised to reuse; otherwise replace them.
4. Record source URL and capture context in each asset's `source` metadata.
5. Import transactionally with `publish:false`, instantiate and preview the
   result through the exact public renderer at all breakpoints, then publish the
   reviewed immutable versions.

Re-importing creates new immutable versions. It never rewrites a creator's
existing pages. Updating or migrating placed pages is always a separate,
previewed and reversible operation.

## Rules that will save you (all learned in production)

1. **`pages.site_id` must be set.** A published page with `site_id = NULL`
   404s for every visitor regardless of its publish flags. Always attach pages
   to a site.
2. **Money is in minor units (öre).** `offers.price_sek` of 1 197 kr is
   `119700`. Getting this wrong sells for 1% of the intended price.
3. **Slugs are case-sensitive, no trailing slash.** `/bloom` works; `/bloom/`
   and `/Bloom` 404. For old inbound links create `redirects` rows — matching
   is on the EXACT `from_path`.
4. **Checkout links are `/checkout?offer=<offer-slug>`.**
5. **Finite installments are an optional buyer choice.** On a one-time offer,
   `installment_count > 1` adds a selector while full payment remains the
   default. Card installments are finite Stripe charges; equal payments are
   offered only when their sum exactly matches the final cart total. Invoice
   installments create a manually reviewed request, never an automatic invoice
   series and never one invoice for the whole amount. `installment_amount_sek`
   remains legacy display metadata; checkout derives the amount from the live
   total and count.
6. **The legacy block engine is retired.** Do not send `blocks`, raw embed
   layout or block-registry properties when creating or rebuilding a page.
   Maintain the `design-sections-v1` engine, its immutable section version IDs
   and typed values. Preview through `POST /api/admin/page-designs/preview`.
7. **`consent_mode` is a compliance decision, not a default.** `'assume'`
   fires tracking pixels without a consent banner. Never set it yourself;
   surface the question to your human. In `geo` and `require` modes the
   platform's `idun_consent` choice is the single source of truth; do not add
   a second consent banner or load GA/Meta independently in custom scripts.
8. **Use the API, never suggest direct SQL.** The API invalidates caches for
   you (page renders are cached at the edge); direct writes leave stale pages
   live for hours.
9. **Retries are safe only with your idempotency key.** Send one on every POST
    you may retry. Without it, a timed-out create may have landed; LIST before
    sending a new request.
10. **Swish is conditional, not guaranteed.** `swish_enabled: true` asks the
    checkout to permit Swish. It is shown only for SEK and only when Stripe says
    the connected account and buyer are eligible. Read `payment_methods` from
    the checkout-info response instead of promising a payment method from the
    offer flag alone.

## When something is missing or confusing — say so

`POST /api/admin/agent/feedback` files a report straight to the people
building Idun Blue, in the same inbox their human testers use. Use it when an
endpoint you need does not exist, a response is impossible to act on, a
documented rule turned out to be wrong, or a workflow needed far more calls
than it should. Any valid key may post, including one that can do nothing else.

```json
{ "kind": "idea",
  "message": "No way to attach an existing lesson to a second module.",
  "endpoint": "POST /api/admin/modules/:id/lessons",
  "goal": "Reuse one lesson across two courses",
  "errors": ["404 lesson_not_found"] }
```

`kind` is one of `bug`, `confusing`, `idea`, `praise`. Do not use this
as a retry mechanism — file once, then tell your human what you filed and work
around it if you can.

## Working etiquette

- Prefer `draft` → let the human review → publish (or ask them to).
- To build a nurture/drip sequence, prefer
  `create_email_sequence_draft`. It atomically creates a manual draft and all
  emails in the workspace language; it never enrolls a contact or activates
  delivery. Read it back and let the creator review it before any trigger or
  live status is changed.
- Before a broadcast send, use `POST /api/admin/broadcasts/:id/test` to send
  yourself/the creator a test — even when you hold can_publish.
- Verify after writing: GET the resource back, or preview the page. The write
  succeeding is not the same as the result being right.
- Small media and document uploads go through the multipart endpoint
  `POST /api/media-library/upload` (scope `write:media`). Read its contract
  first; it accepts `.ics` calendars as well as the documented image, audio,
  video and document formats. Multipart uploads cannot go through the JSON-only
  `idun_api_write` MCP tool, so use a direct authenticated HTTP request. Larger
  course videos go through `/api/upload/video`.

## Keep this skill current

This manual is served as a valid Codex `SKILL.md`, including its YAML
frontmatter. A repository may cache it locally for discoverability, but refresh
it from `GET /api/admin/agent/skill` at the start of an Idun task and treat the
live version as authoritative. A tiny `AGENTS.md` bootstrap is better than a
second copied API manual: tell the agent to load this skill, call `whoami`, and
discover contracts from the live manifest instead of duplicating them.
