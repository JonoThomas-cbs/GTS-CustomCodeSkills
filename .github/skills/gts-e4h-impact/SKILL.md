---
name: gts-e4h-impact
description: Analyze SAP GTS 11 ABAP code for migration to SAP GTS edition for SAP HANA (GTS e4H). Use for deprecated object detection, old-to-new table and field mappings, migration impact assessment, and replacement suggestions based on the bundled workbook-derived reference.
---

# SAP GTS 11 to GTS e4H impact analysis

Analyze the supplied or accessible ABAP source; produce evidence-based findings and replacement suggestions. Default to analysis only. Edit source only when explicitly requested.

## Load the mapping reference

Read [references/gts-object-mappings.md](references/gts-object-mappings.md) before analyzing code. It contains every data row from both sheets of `GTS e4H impact.xlsx`, with original worksheet row numbers. Search this file for each relevant object if tools support search; fetch surrounding rows and all matches, not just the first match. If search is unavailable, read the complete reference in sections. Do not silently analyze with only a truncated portion of the reference. If the reference cannot be accessed, report that mapping-based analysis is blocked and ask for it.

Treat the workbook as the supplied migration catalog, not proof of release-specific SAP support or API availability. Preserve its source values. Ignore surrounding whitespace and case for lookup. Blank, `#N/A`, numeric `0`, and textual `0` mean **no replacement supplied**; never suggest those values as object names. Keep every candidate for duplicate old names. Combine evidence across both sheets without losing object types, row numbers, or conflicting mappings. A blank in one sheet does not cancel a nonblank mapping in the other. Do not assume a mapping is bidirectional or transitive.

## Establish scope

1. Record source GTS version and target e4H release if known. Continue with an explicit target-release assumption if missing; do not label recommendations release-verified.
2. Identify the code actually available: selected snippet, active editor, named object, accessible includes/classes, local files, or explicitly requested project scope. Enumerate inspected objects and unavailable dependencies. Do not claim to scan the SAP repository when only editor content or cached source is accessible. Request pasted or exported source when necessary.
3. Inspect relevant includes, called custom classes/functions, DDIC definitions and CDS sources when accessible. Separate inspected dependencies from unresolved ones. Do not scan the skill/reference files as application code.

## Detect usages

Match complete identifiers case-insensitively. Do not match an object merely because its name is a substring of another identifier. Preserve namespace slashes, underscores and digits. Analyze ABAP syntax and context, not just text occurrences.

Inspect SELECT/JOIN and SQL updates, declarations using TYPE/LIKE/TABLES, structure components, table types, method/function signatures, CALL FUNCTION, SUBMIT, lock calls, DDIC references, dynamic SQL, ASSIGN and string-built object names. Check CDS/DDLS and AMDP SQLScript if included in scope, using their own syntax rules.

For field mappings such as `TABLE~FIELD`, recognize direct qualified SQL usage, `TABLE-FIELD` DDIC typing, aliases (`FROM table AS a` followed by `a~field`), and variables or work areas with resolvable declared types (`wa-field`). Trace aliases and types before assigning a field-level finding. If unresolved, report a potential match with the missing type evidence. Do not flag every identically named field on unrelated tables. When the catalog has field-level mappings only, report a table-only use as requiring field inspection; do not invent a whole-table replacement.

Treat full-line `*` comments and inline `"` comments as comments in ABAP, accounting for quoted literals and string templates. Report comment-only mentions separately, excluded from executable-impact totals. Strings may be executable dependencies in dynamic SQL or CALL FUNCTION; inspect their use and flag unresolved constructed names as coverage gaps. For CDS/AMDP use the corresponding comment syntax.

Apply repository object types accurately: TABL can describe a table or a structure; verify DDIC category when available. FUGR denotes a function group, not a callable function module. Do not infer a function-module replacement by renaming its group. ENQU denotes a lock object; verify generated functions and lock semantics. TTYP, DTEL, DOMA, VIEW, PROG and other types require their own usage context. For package/interface/variant metadata, do not declare a runtime failure from a textual mention alone.

## Assess each finding

- **Deprecated with candidate:** an exact match to the Deprecated Object sheet plus one or more usable candidates from either sheet.
- **Deprecated without supplied replacement:** an exact deprecated match, with no usable candidate in either sheet. Recommend redesign/research rather than guessing.
- **Mapping candidate only:** an exact Replacement Table match with no corresponding deprecation evidence. Recommend assessment; do not label it confirmed deprecated.
- **Ambiguous/conflicting:** multiple distinct candidates or incompatible evidence. Show all candidates and the context needed to choose. Repeated identical candidates are corroboration, not ambiguity.
- **Potential/dynamic:** a match whose runtime resolution or alias/type evidence is incomplete.

Separate match confidence from replacement confidence. An exact catalog match does not prove a candidate preserves business semantics. Use High/Medium/Low impact with a short reason: critical business updates/status/locks and incompatible interfaces deserve priority; comments alone do not.

Before proposing a concrete rewrite, compare target fields, keys, cardinality, joins, status meaning, data types, lengths, conversions, client handling, authorization, exceptions and update/lock behavior as relevant. Pay particular attention to one-to-many field replacements: do not choose a status field merely from its name. Never suggest direct database writes as a substitute for a business API without validated support. Check candidate existence and suitability in the actual target system or authoritative SAP documentation when tools permit. State which checks were performed and which remain open.

Do not fabricate replacement names, function modules, classes, CDS views, RAP BOs, OData services or SAP Notes. Do not apply S/4HANA Public Cloud API restrictions to a GTS installation without confirming its environment. For UI5/RAP/OData-related changes, consult current official documentation for the applicable product/release; prefer OData V4 where supported, without inventing a service migration.

## Output

Start with scope, inspected files/objects and limitations, then summarize finding counts by category. Deduplicate repeated occurrences in summary totals but retain every actionable source location. Distinguish distinct impacted identifiers from occurrence count.

Provide a table:

| Source location | Old object/field | Usage | Catalog status | Candidate(s) | Impact / reason | Confidence (match / replacement) | Evidence | Action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

Cite evidence as `Deprecated Object!row N` or `Replacement Table!row N`, including every row supporting alternatives. Use actual file/object and line numbers where available. For pasted snippets state that line numbers are snippet-relative; never invent repository locations.

For each actionable finding explain the compatibility issue, list all catalog candidates, recommend next steps and give a minimal before/after ABAP example only when enough target definition evidence exists. Otherwise supply clearly labeled pseudocode or a verification checklist, not supposedly compilable code with invented fields.

Finish with prioritized remediation and validation: target-system syntax/activation and ATC where available, DDIC/interface checks, regression tests for affected business processes, selection results and joins, status transitions, authorizations and locking/transaction behavior. Report validation as pending unless actually executed.

If no matches exist, say `No catalog matches in the inspected source`; never conclude the program is migration-safe. List uninspected dependencies and dynamic usages separately. State that the supplied catalog does not cover every possible migration change.
