Scoping Policy and Reference County Assignments

**RFC-001 Addendum A**
Status: Draft
Applies to: RFC-001 (Michigan MeshCore Regions), Draft
Supersedes: the August 2026 draft "Unscoped traffic and default scope"
Companion file: `data/county-subregions.json`

**Goal:**
Make RFC-001's hierarchy do work. Part 1 recommends what repeaters and
companions do with traffic that carries no region, so that adding regions
actually reduces flooding. Part 2 gives every Michigan county a default
subregion so that new operators, maps, and tools have a shared reference
without turning RFC-001's flexible edges into borders.

## How to read this addendum

RFC-001 defines names and a hierarchy. It is explicit that it does not
assign ownership, grant authority, or fix boundaries, and it notes that
merely adding named regions does not disable unscoped flooding. This
addendum takes both statements as its starting point.

Everything below is a recommendation with its consequence stated beside it.
Nothing here can be enforced; Michigan's repeaters are run by independent
operators, and each makes their own call. The addendum's job is to make sure
that call is informed, and that operators who choose differently say so, so
that behavior across the mesh stays diagnosable.

---

# Part 1 — Unscoped traffic and default scope

## Why this part exists

Chicago traffic is what brought this up, but it is not the reason for it.
MeshCore is growing fast across the United States, and the Netherlands and
the rest of Europe have already shown what happens when a mesh fills in
with no scoping: every flood packet from every area is repeated everywhere
it can reach, and the local mesh stops being usable for the people standing
under it. Grand Rapids packets already reach Muskegon regularly, the Thumb
often, and Detroit is filling in. This is coming for Michigan whether or not
a duct ever forms over Lake Michigan again.

The goal is to get ahead of it while the network is still small enough to
change: keep the mesh usable for people in their own area by default, and
make reaching far-away areas something a person does deliberately — a
scoped channel, a chosen scope — rather than something every message does
every time.

The trigger, and the case study used below: West Michigan repeaters
regularly hear flood traffic from Chicago carried across Lake Michigan by
tropospheric ducting. The link is one-way, so it is pure noise here, and
every Grand Rapids repeater that hears a packet re-floods it. One audible
out-of-area node becomes airtime loss across the whole local mesh.

## Why regions alone do not fix it

A repeater forwards traffic scoped to regions it carries and ignores traffic
scoped to regions it does not. Unscoped traffic — anything sent without a
region, which is everything from a companion that has not set one — is
forwarded by every repeater regardless of what it carries. The Chicago
traffic is unscoped. Regions do nothing about it unless operators also
decide what to do with unscoped traffic.

Blocking unscoped traffic outright (`region denyf *`) is the wrong first
answer: every new user starts out unscoped, and blocking them silences them
until they find a setting they do not know exists.

## Recommendations

### 1. Tag every repeater

**Recommendation:** every Michigan repeater carries the full RFC-001
ancestry for its location and, where firmware supports it, sets its default
scope to `mi`.

**Consequence of skipping it:** a repeater without `mi` configured will not
forward `mi`-tagged traffic. Until every repeater has it, anyone who scopes
to `mi` gets less reach than the unscoped users, and early adopters will
conclude regions are broken. This step must be complete before step 2
begins.

The commands depend on repeater firmware. Check with `ver`. Examples are for
a Grand Rapids repeater; substitute the local region per RFC-001 and Part 2.

Repeater firmware 1.16 or later:

```
region def midwest mi mi-west grr
region default mi
region save
```

Repeater firmware 1.15 (no `region def`):

```
region put midwest
region put mi midwest
region put mi-west mi
region put grr mi-west
region default mi
region save
```

Repeater firmware 1.10 through 1.14 (`region default` does not exist; the
repeater's own adverts stay unscoped, but it forwards scoped traffic
correctly, which is what this step needs; discovery from the app needs
1.12):

```
region put midwest
region put mi midwest
region put mi-west mi
region put grr mi-west
region save
```

Below 1.10, regions do not exist. Upgrade first; if the site cannot be
reached to reflash, say so in the group so it is accounted for in step 2.

### 2. Cap unscoped floods on every repeater

**Recommendation:** `flood.max.unscoped 3` on every Michigan repeater. This
is the value the MeshCore documentation suggests as the alternative to
`region denyf *`.

**Rollout order:** tall and heavily-linked sites first, because they hear
the most out-of-area traffic and re-flood the most; everyone else as they
get to it.

**Why every repeater rather than only tall sites:** a duct can drop Chicago
into any lakeshore repeater. If only the high sites are capped, a packet that
lands on an uncapped repeater is re-flooded from there, and the high sites
end up hearing it from inside Michigan with fewer hops on it than the direct
path had. Capping everywhere means it dies within a couple of hops of
wherever it enters. One rule that applies to every repeater is also easier
to adopt and to check than a list of sites.

**How it works:** the repeater drops an unscoped flood packet once its path
already carries 3 hops. A new user's zero-hop advert and first messages
reach their local repeaters; a Chicago packet already repeated three times
before crossing the lake does not get a fourth.

**Consequence:** an unscoped user cannot be heard more than 3 hops out. A
companion with a default scope set is unaffected, so the cost falls only on
users who have not set one, and the remedy is the one setting everyone is
being asked to make anyway.

**Sparse-area carve-out:** where a new user would need more than 3 hops to
reach anyone — much of the northern Lower Peninsula and the Upper Peninsula
today — operators may reasonably leave the cap at default until coverage
fills in. Operators who do are asked to say so in the group.

```
set flood.max.unscoped 3
get flood.max.unscoped
```

Requires repeater firmware 1.16. Operators are asked to trial the cap for a
week and share numbers before treating it as permanent.

### 3. Strict forwarding (future, optional)

`region denyf *` is appropriate only when the operator is satisfied that
companions in its coverage have broadly set a default scope, and only in
coordination with neighboring repeaters. This addendum sets no date or
trigger. An operator who enables it earlier is asked to say so in the
group, so that "my new radio can't reach anyone" reports can be diagnosed.

### Companion configuration

MeshCore app 1.43 or later, companion firmware 1.15 or later:

1. Settings → Experimental → default scope region: `mi`
2. Channel scopes per RFC-001: `#michigan` → `mi`, `#wmi` → `mi-west`,
   `#grr` → `grr`, `#azo` → `azo`; Public scoped to the local region for
   everyday conversation, `mi-west` or `mi` when the conversation warrants.

Default scope covers everything a channel scope does not: adverts, direct
messages, logins, requests. A user who only scopes the Michigan channel is
still sending unscoped adverts and DMs, and those hit the cap. A companion
with default scope set and channels scoped is unaffected by any unscoped
policy a repeater adopts, including strict forwarding.

## Firmware requirements

| Feature | Minimum version |
| --- | --- |
| Region configuration and filtering on repeaters | Repeater firmware 1.10 |
| Region discovery from the app (Scan Local / Discover Regions) | Repeater firmware 1.12 |
| Default scope on companions and repeaters (`region default`) | Firmware 1.15, MeshCore app 1.43 |
| `flood.max.unscoped` | Repeater firmware 1.16 |
| `region def` single-line hierarchy command | Firmware 1.16 |

## Verification

- **Regions:** from the app, Discover Regions (Scan Local). Every repeater in
  direct range that has completed step 1 reports `mi` and its ancestry.
- **Airtime:** operators with an observer or CoreScope record, before and
  after a site enables the cap, the share of received packets that are
  unscoped with more than 3 hops, and the site's own transmit airtime.
  Expected: a large drop in re-transmitted unscoped packets at the capped
  site with no change in scoped Michigan traffic.
- **New users:** a freshly flashed companion with no default scope can still
  zero-hop advert, reach a repeater within 3 hops, and complete a first
  direct message. If it cannot, either the user is in a sparse area where 3
  hops reaches nobody, or a nearby repeater has enabled strict forwarding.

---

# Part 2 — Reference county assignments for subregions

## Scope and intent of the reference

RFC-001 defines the Michigan hierarchy and is deliberate about what it does
not do: it does not fix county or municipal boundaries, and it says regions
are RF propagation and community domains rather than political ones. This
addendum does not change that. It adds a *reference*: for every county, one
subregion that is the reasonable default absent other information.

The reference exists for three practical reasons:

1. A new repeater operator in, say, Isabella County should be able to learn
   in one lookup that `mi-central` is the subregion to start from.
2. A map on michmesh.com needs a rule for what to color.
3. A configuration helper needs data to generate `region put` commands.

None of those needs is served by "edges are flexible" alone, and all of them
are harmed by a hard border. The reference is the middle path: a default
that a community can depart from for good RF reasons, with the departure
documented rather than argued.

This part does not define local regions (`grr`, `azo`, and their peers).
Local regions remain the business of the communities that operate them, per
RFC-001.

## Precedence

Where this part and reality disagree, reality wins, in this order:

1. **Coverage.** A repeater carries the subregion of the area it actually
   serves. A hilltop repeater in Ionia County whose footprint is mostly Grand
   Rapids belongs in `mi-west` and `grr` regardless of where Ionia appears
   below.
2. **Community agreement.** If the operators in an area have agreed on a
   subregion for it, that agreement supersedes this table. The table should
   then be updated to match, with a note.
3. **This reference.** In the absence of the above, a county's subregion is
   the one listed here.

A repeater at an edge may reasonably carry two subregions — for example
`mi-west` and `mi-central` on a site between Grand Rapids and Lansing — since
MeshCore allows many carried regions. This part takes no position on
whether an edge site should; that is the operator's call, informed by what
traffic the site actually hears.

## Basis: broadcast markets

The reference uses Nielsen's Designated Market Areas — the county groupings
that describe where each Michigan television market's transmitters reach —
as its starting point. They were chosen over administrative schemes (state
prosperity regions, congressional districts, MDOT regions) because they are
the only widely published Michigan county grouping that is derived from RF
propagation rather than from politics or economics. A DMA is, roughly, the
set of counties that can see the same tower. That is close to what a MeshCore
subregion is trying to describe.

Michigan is covered by seven in-state markets (Detroit; Grand Rapids–
Kalamazoo–Battle Creek; Flint–Saginaw–Bay City; Lansing; Traverse City–
Cadillac; Marquette; Alpena) and four spillovers from neighboring markets
(South Bend–Elkhart, Toledo, Green Bay–Appleton, Duluth–Superior). Together
they assign every county exactly once.

The markets were then mapped onto RFC-001's five subregions with the
adjustments listed under each.

## Reference assignments

| Subregion | Counties | List |
| --- | --- | --- |
| `mi-west` | 16 | Allegan, Barry, Berrien, Branch, Calhoun, Cass, Ionia, Kalamazoo, Kent, Montcalm, Muskegon, Newaygo, Oceana, Ottawa, St. Joseph, Van Buren |
| `mi-central` | 19 | Arenac, Bay, Clinton, Eaton, Genesee, Gladwin, Gratiot, Hillsdale, Huron, Ingham, Iosco, Isabella, Jackson, Midland, Ogemaw, Saginaw, Sanilac, Shiawassee, Tuscola |
| `mi-east` | 9 | Lapeer, Lenawee, Livingston, Macomb, Monroe, Oakland, St. Clair, Washtenaw, Wayne |
| `mi-north` | 24 | Alcona, Alpena, Antrim, Benzie, Charlevoix, Cheboygan, Clare, Crawford, Emmet, Grand Traverse, Kalkaska, Lake, Leelanau, Manistee, Mason, Mecosta, Missaukee, Montmorency, Osceola, Oscoda, Otsego, Presque Isle, Roscommon, Wexford |
| `mi-upper` | 15 | Alger, Baraga, Chippewa, Delta, Dickinson, Gogebic, Houghton, Iron, Keweenaw, Luce, Mackinac, Marquette, Menominee, Ontonagon, Schoolcraft |

Total: 83 counties, each listed once.

### `mi-west`

Grand Rapids–Kalamazoo–Battle Creek DMA, plus the two counties Nielsen
assigns to South Bend–Elkhart (Berrien, Cass).

*Adjustment:* Berrien and Cass are placed here rather than in any Indiana
region. A repeater there that mostly serves Indiana traffic is exactly the
cross-border case RFC-001 leaves to the operator; the reference gives it a
Michigan home by default.

Local regions currently adopted beneath `mi-west`: `grr`, `azo`. RFC-001's
example `mkg` (Muskegon) would also nest here.

### `mi-central`

Flint–Saginaw–Bay City DMA plus the Lansing DMA, plus Sanilac.

*Adjustments:* Lansing is merged rather than kept as its own subregion
because "Mid-Michigan" is how the area describes itself, a five-county
subregion would be the size of a local region, and RFC-001 already places
`midstate` beneath `mi-central`. Sanilac is moved from the Detroit DMA so
the Thumb (Huron, Sanilac, Tuscola) is whole, matching RFC-001's `thumb`
local region.

Local regions RFC-001 names beneath `mi-central`: `thumb`, `midstate`. The
existing MeshMapper zones `fnt` (Flint) and `mbs` (Midland–Bay City–Saginaw)
also fall within it.

### `mi-east`

Detroit DMA less Sanilac, plus Lenawee.

*Adjustment:* Lenawee is Nielsen's Toledo spillover; it is placed with its
Michigan neighbors. Southeast Michigan operators may prefer a name other than
`mi-east`; this part keeps RFC-001's name and does not propose a change.

Local regions: RFC-001's example `det`. The MeshMapper zone `det` exists.

### `mi-north`

Traverse City–Cadillac DMA, Lower Peninsula counties only, plus the Alpena
DMA.

*Adjustments:* The three Upper Peninsula counties Nielsen assigns to Traverse
City (Chippewa, Luce, Mackinac) are moved to `mi-upper`; the Straits are a
real RF boundary and the market assignment reflects where transmitters stood
in 2000. Alpena's two-county market is folded in rather than standing alone.

Local regions: RFC-001's example `tvc`. Note that the existing MeshMapper
zone `gdw` (Gladwin) straddles `mi-central` and `mi-north` as currently
drawn; that is a local-region question, not a subregion one, and is left to
those operators.

### `mi-upper`

Marquette DMA, plus Chippewa, Luce, and Mackinac (from Traverse City), plus
the Green Bay–Appleton (Menominee) and Duluth–Superior (Gogebic) spillovers.

*Adjustment:* All spillovers placed with the peninsula. The UP has no adopted
local region yet; RFC-001's example is `mqt`.

## Machine-readable reference

`data/county-subregions.json` lists every county by FIPS code with its name,
subregion, and full ancestry (`midwest`, `mi`, subregion). It is generated
from the table above and should be regenerated, not hand-edited, when the
table changes. Tools consuming it should treat a county's entry as a default
and expose the precedence rule to users.

---

# Part 3 — Repeaters at a boundary

Part 2's table has edges, and Part 1 depends on repeaters carrying the right
regions, so the question of what a site near a subregion line should carry
belongs here rather than in either part alone.

A repeater's carried regions are forwarding permissions, and a repeater can
carry many. A site on the `mi-west` / `mi-central` line may carry both,
along with their shared ancestors. It then forwards traffic scoped to
either. Its own adverts go out under `region default`, which is a single
choice.

Carrying both makes the site a bridge: `mi-west`-scoped traffic is forwarded
by it into `mi-central` territory, to the next repeater, which drops it if
it carries only `mi-central`. One bridge leaks one hop. That is often
useful. It stops being useful when every edge site bridges, because scoped
traffic then walks across a neighboring subregion one hop at a time and the
network has rebuilt unscoped flooding with extra steps.

**Recommendations:**

1. **Home region is where most of the footprint is.** Set `region default`
   to it. If unsure, observe for a week which subregion's repeaters the site
   hears most.
2. **Carry a neighboring subregion only with real coverage there.** Hearing
   a neighbor's repeater on a good day is not coverage; serving companions
   in the neighbor's area is. Tropospheric propagation does not count.
3. **Bridges are a network decision, not a site decision.** A few deliberate
   bridge sites per boundary, announced in the group, are preferable to
   every edge repeater quietly carrying both. Two bridges within a few
   miles of each other on the same boundary is one too many.
4. **Local regions follow the same logic**, independently of subregion. A
   site serving two metro areas carries both local regions.
5. **Announce bridges.** A bridge nobody knows about is how cross-region
   traffic becomes a mystery.

Example for a Grand Rapids-side site on the Ionia line that serves both.
Commands depend on repeater firmware as in step 1; check with `ver`.

Repeater firmware 1.16 or later:

```
region def midwest mi mi-west grr
region def midwest mi mi-central
region default mi-west
region save
```

Repeater firmware 1.15 (no `region def`):

```
region put midwest
region put mi midwest
region put mi-west mi
region put grr mi-west
region put mi-central mi
region default mi-west
region save
```

Repeater firmware 1.10 through 1.14 (no `region default`; the site forwards
both regions' scoped traffic correctly, but its own adverts stay unscoped):

```
region put midwest
region put mi midwest
region put mi-west mi
region put grr mi-west
region put mi-central mi
region save
```

Do not set `region default` to `mi` on a border site to "cover both"; that
makes its adverts statewide. And do not leave the unscoped cap at default on
a bridge; the bridge is where a leaked unscoped flood gets a fresh set of
hops, so `flood.max.unscoped 3` matters most there.

---

## Open items

- Whether Lansing-area operators want `mi-central` or their own subregion.
- Whether the Thumb should be a local region under `mi-central` (per
  RFC-001) or a subregion; this addendum follows RFC-001.
- The name `mi-east` for Southeast Michigan.
- Any local region's boundary, including how `gdw` relates to `mi-north`.
- Which default scope Kalamazoo companions use, and whether Kalamazoo wants
  cross-region discovery with Grand Rapids.
- Coordination with Chicago-area and other neighboring communities on
  `midwest`, so it is a channel choice there as well and not a default.

## References

- MeshCore CLI reference: https://docs.meshcore.io/cli_commands/
- MeshCore blog, region filtering: https://blog.meshcore.io/2026/01/20/region-filtering
- MeshCore blog, default scope: https://blog.meshcore.io/2026/04/17/default-scope
- MeshCore 1.16.0 release notes: https://blog.meshcore.io/2026/06/06/release-1-16-0
- `flood.max.unscoped` implementation: https://github.com/meshcore-dev/MeshCore/pull/2661
- Pacific Northwest region rollout: https://gessaman.com/meshcore/regions/rollout/
- Nielsen DMA county assignments (via michiguide.com/tvmarkets and ustvdb.com)

## Change history

- 2026-09 — Initial draft. Combines the August unscoped-traffic proposal
  (updated: cap of 3 on every repeater, tall sites first, sparse-area
  carve-out) with the county reference assignments and boundary guidance.
