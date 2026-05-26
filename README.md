# Texas Coast — Orphan-Well Inspection Priority with Sea-Level-Rise Risk

An interactive, single-file HTML map that merges every public Texas General Land
Office (GLO) coastal GIS layer with NOAA's Environmental Sensitivity Index and
Sea-Level-Rise inundation polygons, then ranks offshore orphan wells/platforms
by inspection priority and predicts the lowest SLR scenario at which each
becomes inundated.

[**Live demo →**](https://HL-opt-2023.github.io/texas-coast-orphan-wells-slr/)
&nbsp;·&nbsp;
[Issues](https://github.com/HL-opt-2023/texas-coast-orphan-wells-slr/issues)

> ⚠️ Research / demo code. Predictions use NOAA's bathtub inundation model and a
> derived orphan-well proxy from GLO lease status — not an authoritative Texas
> Railroad Commission inspection schedule. See **Caveats** below.

---

## What it does

Three tools, each independently toggleable from a checkbox menu in the side
panel:

### 1. Conflict analysis
Flags any lease, easement, or offshore structure in the current viewport that
overlaps a GLO **Priority Protection Habitat** polygon (or optionally an
**ESI ≥ 8** sensitive shoreline segment). Per-layer hit counts are shown in
the status panel.

### 2. Orphan-well inspection priority
For each **offshore Well or Platform** point sitting inside an inactive GLO
lease (status `Expired`, `Terminated`, `Forfeited`, or `Cancelled`), computes
a composite priority score and lists the top-N for inspection. Scoring:

| Component | Range | Definition |
|---|---|---|
| `E` Environmental sensitivity | 0–4 | PPA priority (High 3 / Med 2 / Low 1) + 1 if lease touches ESI ≥ 8 |
| `A` Abandonment age | 0–3 | Years since `LEASE_STATUS_DATE`: ≥20→3, ≥10→2, ≥5→1 |
| `S` Surviving structures | 0–3 | Wells / platforms still inside the lease polygon |
| `Size` | 0–2 | Gross acres: ≥100→2, ≥10→1 |
| Asset-type bonus | 0–1 | +1 if asset is a Well, 0 if Platform |
| **Σ Total** | 0–~16 | `2·E + A + S + Size + bonus` (environmental risk weighted ×2) |

Top-N wells appear as numbered red (Well) / orange (Platform) markers; clicking
a row in the side-panel list smoothly pans the map to that well and opens its
attribute popup.

### 3. Sea-level-rise risk
Queries [NOAA Office for Coastal Management's SLR inundation
polygons](https://coast.noaa.gov/digitalcoast/tools/slr.html) at 1, 2, … 10 ft
above current Mean Higher High Water and predicts, for each ranked well, the
**lowest SLR scenario at which it first becomes inundated**.

The continuous slider (0.0 – 10.0 ft, 0.1 ft steps) fades the inundation
overlay opacity smoothly between integer NOAA thresholds and updates marker
states live. Scenario labels translate feet to **Sweet et al. (2022) NOAA
SLR scenarios**:

```
1.6 ft ≈ Intermediate-Low (~0.5 m by 2100)
3.3 ft ≈ Intermediate     (~1.0 m by 2100)
4.9 ft ≈ Intermediate-High(~1.5 m by 2100)
6.6 ft ≈ High             (~2.0 m by 2100)
8.2 ft ≈ Extreme          (~2.5 m by 2100)
```

The primary risk modality is **inundation**: once a wellhead submerges,
saltwater accelerates casing/cement corrosion, inspection access is lost, and
any leak releases directly into open water.

---

## Quick start

### Option A — try the live demo
Just open the [GitHub Pages
link](https://HL-opt-2023.github.io/texas-coast-orphan-wells-slr/) in a modern
browser. The map loads in seconds; data is fetched live from public GLO and
NOAA ArcGIS services on demand.

### Option B — run locally
```bash
git clone https://github.com/HL-opt-2023/texas-coast-orphan-wells-slr.git
cd texas-coast-orphan-wells-slr
open index.html        # macOS — or just double-click in Finder / File Explorer
```

No build step, no dependencies installed locally — the file pulls
Leaflet, Esri-Leaflet, Turf.js, and the grouped layer control from public CDNs
(all with Subresource Integrity hashes pinned). Needs internet access for the
libraries and the live GLO/NOAA REST queries.

### How to use the orphan-well + SLR workflow
1. Pan/zoom to a Texas coastal bay (Galveston, Matagorda, Corpus Christi, etc.)
   until the **Current zoom** indicator in the Tools menu shows **✓ ready**
   (zoom ≥ 8 / ~42%).
2. Click **Build orphan-well list** in the *Orphan-well priority* section.
   A progress bar walks through six phases (Indexing → Fetching habitat →
   Linking wells → Scoring → Done) in a few seconds.
3. Top wells appear as numbered markers; the side panel shows a sortable
   ranking with Σ / E / A / asset-type columns. Click a row → smooth fly-to
   plus pulse-ring highlight.
4. *(Optional)* Check **Sea-level rise risk** in the Tools menu, click
   **Compute SLR thresholds**, then drag the **Scenario** slider to watch
   wells flip into the inundated tier (cyan glow) as the simulated sea
   level rises.

---

## Data sources

### Texas General Land Office (GLO) — public ArcGIS Hub
28 layers from
[`services1.arcgis.com/YWG34dhJxrbxQWdF`](https://data-glo.hub.arcgis.com/),
grouped in the layer control as:

| Group | Layers |
|---|---|
| Sensitive habitat | ESI Shoreline · Priority Protection Habitat Areas |
| ESI Biology | Birds (poly + point colonies) · Fish · Invertebrates · Benthic · Habitat · Marine mammals · Terrestrial mammals · Reptiles/amphibians |
| Leases & activity | O&G Leases (Active / Inactive) · O&G Units (Active / Inactive) · O&G Nominated Tracts · Hard Mineral Leases · Coastal Lease Polygons & Points · Misc Easements · Upland Surface Leases · Offshore O&G Structures |
| Land & jurisdiction | Permanent School Fund Lands · State Agency Lands · Submerged Tracts with RMC · Navigation Districts · Coastal Zone Boundary · Three Nautical Mile Line · Three Marine League Line |
| Spill response | Dispersant Pre-Approval Zone · In-Situ Burn Exclusion · Beach & Bay Access |

### NOAA Office for Coastal Management — SLR inundation
Ten MapServer endpoints at `coast.noaa.gov/arcgis/rest/services/dc_slr/slr_Nft`
for N = 1 .. 10 ft above current MHHW.

All layers load **dynamically** for the current viewport — nothing is
downloaded to disk, and the map always reflects the live published data.

---

## Tech stack

| Library | Version | Role |
|---|---|---|
| [Leaflet](https://leafletjs.com) | 1.9.4 | Base map, tiles, layer control |
| [Esri-Leaflet](https://esri.github.io/esri-leaflet/) | 3.0.12 | Dynamic FeatureLayer loading from ArcGIS REST |
| [leaflet-groupedlayercontrol](https://github.com/ismyrnow/leaflet-groupedlayercontrol) | 0.6.1 | Grouped/categorized layer toggles |
| [Turf.js](https://turfjs.org) | 6.5.0 | Client-side geometry (bbox, intersect, point-in-polygon, buffer) |

All four libraries are loaded from CDN with **Subresource Integrity (SHA-384)**
hashes pinned to exact versions. If the CDN ever serves a file that doesn't
match, the browser refuses to load it.

Three basemaps: OpenStreetMap, Esri Imagery, Esri Oceans.

---

## Performance notes

The map handles bay-scale viewports (zoom 8–12). At wider views some layers
hit the ArcGIS server-side cap of 2000 records per query — the tools detect
this and warn you to zoom in for full coverage.

Heavy operations are protected from freezing the browser:
- Lease bboxes are cached once and reused via cheap bbox-overlap pre-filters
  before any expensive Turf intersection.
- All large loops yield to the browser every 5–50 features so the UI stays
  responsive and the progress bar updates live.
- Server-side `where` filters and `geometry` envelopes scope every query to
  the current viewport.

---

## Caveats

1. **Orphan-well proxy.** GLO does not publish a direct "orphan wells" feed.
   This map infers orphan status from inactive lease records (`Expired`,
   `Terminated`, `Forfeited`, `Cancelled`) combined with the continued
   presence of physical Wells/Platforms in GLO's Offshore Inventory. A
   production-grade list would also pull from the **Texas Railroad
   Commission**'s well-status data (different agency, different API).

2. **Bathtub inundation model.** NOAA's published polygons fill land below
   each MHHW + N ft level with no consideration of erosion, subsidence (Texas
   Gulf coast has measurable subsidence in some areas), or storm-driven
   temporary flooding. For absolute date-anchored projections use regional
   SSP scenarios from
   [USGCRP NCA5](https://nca2023.globalchange.gov/) or NASA-IPCC.

3. **Live external services.** The map queries `services1.arcgis.com` and
   `coast.noaa.gov` every time it loads. If either service is down or your
   network blocks them, layers won't render. Nothing is stored locally.

4. **Modeling, not prescription.** The priority score is a research-quality
   ranking, not an inspection schedule.

---

## License

This project is licensed under the [Creative Commons
Attribution 4.0 International License](LICENSE) (CC BY 4.0).

You are free to share and adapt the material for any purpose, including
commercially, provided you give appropriate credit and indicate if changes
were made. See [LICENSE](LICENSE) for the full text.

GLO and NOAA source data are public-domain US-government / Texas-government
datasets and remain so.

## Author / contact

**Hongcheng Liu** — University of Florida
📧 [liu.h@ufl.edu](mailto:liu.h@ufl.edu)

Questions, bug reports, or collaboration ideas: please open an
[issue](https://github.com/HL-opt-2023/texas-coast-orphan-wells-slr/issues)
or email directly.

## How to cite

If this map informs research or policy work, please cite:

```
Liu, H. (2026). Texas Coast Orphan-Well Inspection Priority with
Sea-Level-Rise Risk. Open-source interactive map combining Texas GLO
and NOAA OCM datasets.
https://github.com/HL-opt-2023/texas-coast-orphan-wells-slr
```

Underlying data: cite the original publishers — **Texas General Land Office**
([data hub](https://data-glo.hub.arcgis.com/)) and **NOAA Office for Coastal
Management** ([SLR Viewer](https://coast.noaa.gov/digitalcoast/tools/slr.html)).
Scenario anchoring follows **Sweet, W.V. et al. (2022). Global and Regional
Sea Level Rise Scenarios for the United States. NOAA Technical Report NOS 01**
([report](https://oceanservice.noaa.gov/hazards/sealevelrise/sealevelrise-tech-report.html)).

---

## Acknowledgements

- Texas General Land Office Geospatial Team for publishing the underlying
  layers via the [ArcGIS Hub](https://data-glo.hub.arcgis.com/).
- NOAA Office for Coastal Management for the SLR viewer and inundation
  services.
- The Leaflet / Esri-Leaflet / Turf.js / leaflet-groupedlayercontrol
  maintainers.
