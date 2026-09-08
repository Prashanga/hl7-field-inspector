# HL7 Field Inspector

Download and open `public\index.html` directly in a modern browser. It is a self-contained local app: no installation, build step, server, uploads or external dependencies are needed.

The app provides four tools: **Inspector**, **Compare**, **De-identify** and **PMS Preview**.

> **Intended use:** HL7 Field Inspector is a development, interoperability and troubleshooting utility for examining HL7 messages. It is not intended to diagnose, treat, monitor or prevent disease, provide clinical decision support, or be used as the basis for patient-care decisions. Results, warnings, PMS previews and de-identification output should be independently reviewed before use in a production healthcare environment.

## Quick start

1. Open `hl7-field-inspector.html` in your browser.
2. In **Inspector**, paste an HL7 message or choose **Open HL7 file**, then choose **Parse message**. Use **Load sample** to explore with synthetic data.
3. Select a field to inspect its value and components. Use search, segment filters and batch navigation to narrow the view.
4. Open **Compare** to compare two messages, **De-identify** to generate and review replacements, or **PMS Preview** to see an illustrative report. Each tool can take input from Inspector.
5. Use the **Theme** selector in the header to choose your preferred appearance.

## Changes in this version

- **Clearer interface and dark mode:** distinct panel headers, stronger borders, primary buttons, selected states and keyboard focus indicators, with Light, Dark and System themes.
- **Responsive layouts:** controls wrap on smaller screens, option and report grids adapt to the available width, and wide review tables scroll within their panels.
- **One generic PMS preview:** vendor profiles have been removed. Each order has its own report and checks, with a message-level verdict covering all orders.
- **Order-aware comparison:** observations are matched within aligned orders, reducing misleading differences when orders are rearranged.
- **Expanded de-identification and review:** additional date and composite-field handling, reusable replacement sessions and a field-level before/after audit.

This workspace copy contains the updates; the original files in Downloads have not been modified.

## Appearance

Use **Theme** in the header to choose **Light**, **Dark** or **System**. System follows your operating system’s appearance and updates when it changes. Your explicit selection is remembered in this browser; if browser storage is unavailable, the control still works for the current page.

Both themes use stronger panel headers and borders, a distinct primary action, visible focus rings, readable status colors and clearer table/field separation. Changing themes keeps your current inputs and results intact. Only the theme preference is stored; HL7 messages and de-identification sessions stay in memory.

## Generic PMS preview

The preview is labelled **“How this might look in a PMS.”** There is one generic view, with no vendor selector or vendor-specific rules. It reconstructs report content from OBR, OBX and NTE segments. **It is illustrative only: it does not emulate a real PMS and should not be used to determine how a production clinical system will process a message.** It does not predict import acceptance, database patient matching, routing or notifications.

1. Paste a message, open a local file, load the sample, or use the active Inspector message.
2. Choose **Build preview**.
3. For batches, select **Message in batch**.
4. Choose an **Order / report** to see that order's provider, service, observations, field values and checks.

The top message verdict combines checks across all order/content groups in the selected message. Any failed check produces **Blocking issues**; warnings or unknown checks produce **Review needed**; otherwise the verdict is **No issues detected by these checks**. These labels describe the local checks, not an actual PMS import decision. The detailed check list and report panel describe the selected order.

Order grouping follows PID/PV1 context, ORC boundaries and repeated OBR headers. An ORC can supply context to several OBR groups until the next ORC or patient/visit boundary. Observations before an OBR stay in a separate group with a missing-header finding. This is a practical result-message grouping model, not a full HL7 message-profile validator.

## Compare

Compare shows field, component and subcomponent differences between the first message on each side. Paste or open a message on each side, then choose **Compare messages**. Batch inputs are explicitly identified as using their first message.

Orders are aligned first by unique filler or placer identifiers, then by a unique service value. If unmatched order counts agree, a positional fallback is attempted within the same patient occurrence. The alignment notice labels service and positional matches as heuristic and asks you to verify ambiguous pairings. Unmatched orders are shown as added or removed.

OBX matching is scoped to each aligned order and uses the observation identifier, coding system and sub-ID. Reordering orders therefore no longer aligns repeated observation codes from unrelated orders just by global occurrence. Associated NTE segments follow the observation within its order. Multiple-patient messages use patient occurrence for alignment; review these carefully if patients were reordered.

## De-identify and review

Paste or open a message, choose the categories to replace, then select **De-identify message**. Review the output and audit before using **Copy output**, **Save .hl7** or **Open output in Inspector**.

The engine handles date-valued OBX observations using OBX-2, legacy TS precision components, supported date ranges and timing components, common person/contact datatypes in OBX-5, both components of EIP specimen identifiers and additional contact fields.

Supported full dates shift by the same nonzero day offset while preserving available time precision and timezone suffixes. Partial dates (such as `1985` or `198501`) and invalid or unsupported dates stay unchanged and are flagged for review. Time-only values are deliberately retained. This is not an exhaustive implementation of every temporal component in every HL7 composite.

### Reuse a session for working/failing examples

**Reuse replacements and date shift across messages** is checked by default. Generate the first message, save or copy its output, then replace the input with the second message and generate again. The same source identifiers in the same namespace receive the same replacement, and supported dates use the same offset. Provider identifiers stay consistent across name-format differences when their ID, authority and identifier type agree.

**New session** forgets the in-memory replacement maps, creates a fresh offset and clears the old generated output. Turning reuse off starts a fresh session for every run. Closing or reloading the page forgets the session. No message data or replacement maps are saved to browser storage; only the appearance preference is saved. The **Clear** button clears input/output and the review table while retaining the session for the next example; use **New session** to forget the mappings too.

### Review table

After generation, the table lists each populated input field with its message number, path, before value, after value and action/review reason. Filters show:

- **Needs review:** uncovered fields, custom Z fields, disabled-rule fields, unsupported dates and retained identifier composite parts.
- **Changed:** all changed fields, including any that also need review.
- **All populated input fields:** the complete field-level audit, including structural values.

Long values are abbreviated in the table; complete values remain in the input and output. Large audits are displayed in pages of 200 rows. Original values in the review table may contain identifying information. Copy/save exports the output message, not the before/after audit or the pseudonym maps.

Input and option changes invalidate the output, review table and copy/save actions. Source parse errors or ambiguous delimiters prevent generation. The output is parsed again to check message count and structural parser errors; parser warnings remain visible. A successful reparse is not full conformance validation.

De-identification remains best effort. Unmapped fields and retained composite parts can still contain identifying information. A changed field does not certify an anonymous message. Review the output before sharing it, and use Inspector for additional examination.

## Inspector

The field tree, practical version-aware dictionary, search, escape decoding, batch navigation, JSON export and local embedded PDF viewer remain available. **Open output in Inspector** transfers generated de-identification output; **Open message in Inspector** transfers the selected preview message.

## Regression checks

`regression.cjs` uses Node's built-in libraries and synthetic messages. From this folder run:

```sh
node regression.cjs
```

The tests cover verdict aggregation, order grouping and alignment, date handling, shared sessions, namespace-aware pseudonyms, composite specimen IDs, residual-field audits, invalid inputs, batches and removal of vendor-specific predictions. No dependencies are installed and no files are written by the tests.

### Validation performed for this update

- All 28 automated regression checks passed.
- All four tools were exercised in light and dark mode, including theme changes preserving current input and results.
- Theme preference persistence and System mode were checked, along with handling unavailable browser storage.
- Desktop (1280 × 720) and mobile (390 × 844) layouts were checked for readability and page overflow.
- Sampled rendered text passed contrast checks, keyboard focus indicators were checked, and no browser console warnings or errors were reported during the final check.

These checks are not a complete accessibility audit or a guarantee of compatibility with every browser or HL7 profile.

## Attribution

Based in part on HL7 Message Analyzer by Joe Bartlett. The original 2024 copyright and MIT licence notice are preserved in the HTML.
