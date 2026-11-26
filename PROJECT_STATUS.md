# BF OSD Text Editor — shared project status
By Squadding Quads and FPSteVe

## Collaboration
Read this file and inspect current branch/commits before working. Update this file after each investigation or change; fetch its latest blob SHA before editing, reconcile concurrent edits, and prefer feature branches/PRs for substantial changes. Never force-push. Record evidence, test results, unresolved questions and commit IDs. Do not publish firmware as safe without hardware validation.

## Current baseline
- Public repository: https://github.com/alfonsogordon/bf-osd-text-editor
- Pages: https://alfonsogordon.github.io/bf-osd-text-editor/
- v0.2 (commit 3e91dfd1b2228cf606bb5c67f124604e6ab4b86b): browser-local Intel HEX parsing/checksums, SHA-256, candidate scanning, duplicate addresses, editable plain-text preview, JSON report. Format strings locked; firmware export disabled. Deployment/browser testing not independently verified here.
- Original research build: Betaflight 4.5.5 STM32F722 FLYWOOF722PROV2 b5a42002. Original firmware not published in this repo. Second target and hardware test pending.
- Current editor limits are original string lengths, not established OSD limits.

## Source-confirmed findings (4.5.5)
- [osd.h](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/osd/osd.h): OSD_ELEMENT_BUFFER_LENGTH = 32.
- [osd_warnings.h](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/osd/osd_warnings.h): OSD_WARNINGS_MAX_SIZE = 12; OSD_FORMAT_MESSAGE_BUFFER_SIZE = 13.
- [osd_elements.c](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/osd/osd_elements.c#L940): DISARMED uses tfp_sprintf(element->buff, "DISARMED"); static elementBuff at line 246; warning renderer receives element->buff at line 1718.
- [osd_warnings.c](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/osd/osd_warnings.c#L144): CRASH FLIP SWITCH (17 characters) writes directly to warningText. At lines 272–287 the ESC warning uses a separate 13-byte temporary buffer and 12-character formatting logic. Therefore 12 is not a universal warning cap. The 32-byte element buffer implies at most 31 single-byte characters plus NUL for that buffer, not a guaranteed safe per-message or display limit.
- Literal format strings must not gain percent directives. Preserve all existing formatting and warning semantics.

## Binary observations reported in earlier research (reproduce before use)
DISARMED candidates 0x0805B760 and 0x08062C86; LOW BATTERY 0x08063007; LAND NOW 0x08062FC2; LINK QUALITY 0x08061E90 and 0x08062FB5. Earlier analysis reported raw pointer matches, not verified instruction references. Extra zero bytes are not proof of available storage. Candidate scanner can also match suffixes of longer strings.

## Next investigations
1. Reproduce original HEX analysis and distinguish OSD references from unrelated duplicates using Thumb disassembly, including literal pools and computed addresses.
2. Trace element buffer assignment and display backends; establish per-message limits and clipping.
3. Obtain exact target linker layout and flash reservations; absent HEX bytes are not proof of free flash.
4. Prefer verified in-place DISARMED test first; then investigate relocation only with proven references and reserved storage. No universal relocation or 24/31-character claim yet.
5. Verify byte-level changes, record checksums, address map, format strings, integrity checks and original recovery path. Test on spare FC with props removed.
6. Add regression tests and share results here. Keep export disabled until verification gates are implemented.

## Hardware test log
None confirmed yet.

## Change log
- Initial v0.1 inspector and joint branding.
- v0.2 unlocked plain-text preview fields, including duplicates.
- Shared status document: source-level buffer and ESC formatting findings; no firmware or app changes in this investigation.

## Further character-limit research — 4.5.5
- [osd_elements.c](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/osd/osd_elements.c#L2104): activeElement.buff = elementBuff confirms the 32-byte destination for ordinary foreground elements.
- [displayport_msp.c](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/io/displayport_msp.c#L109): MSP_OSD_MAX_STRING_LENGTH = 30; writeString clamps strlen to 30 before transmission.
- [max7456.c](https://github.com/betaflight/betaflight/blob/4.5.5/src/main/drivers/max7456.c#L502): analog writes stop at CHARS_PER_LINE, so visible length depends on starting column. Digital receiver layout also needs verification.
- 30 ASCII characters is a theoretical MSP transmission ceiling for ordinary direct-rendered messages, not verified binary patch capacity. Keep current export disabled and original-length preview limits until further work; no hardware tests performed.
