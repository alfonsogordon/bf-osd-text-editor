# BF OSD Text Editor — shared project status
By Squadding Quads and FPSteVe

## Collaboration
Read this file and inspect current branch/commits before working. Update this file after each investigation or change; fetch its latest blob SHA before editing, reconcile concurrent edits, and prefer feature branches/PRs for substantial changes. Never force-push. Record evidence, test results, unresolved questions and commit IDs. Do not publish firmware as safe without hardware validation.

## Current baseline
- Public repository: https://github.com/alfonsogordon/bf-osd-text-editor
- Pages: https://alfonsogordon.github.io/bf-osd-text-editor/
- v0.2 (commit 3e91dfd1b2228cf606bb5c67f124604e6ab4b86b): browser-local Intel HEX parsing/checksums, SHA-256, candidate scanning, duplicate addresses, editable plain-text preview, JSON report. Format strings locked; firmware export disabled. Deployment/browser testing not independently verified here.
- Original research build: Betaflight 4.5.5 STM32F722 FLYWOOF722PROV2 b5a42002. Original firmware not published in this repo. Second target and hardware test pending.
- Current v0.3 preview permits 30 ASCII characters; original lengths remain conservative in-place budgets, not verified patch limits.

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
- 30 ASCII characters is a theoretical MSP transmission ceiling for ordinary direct-rendered messages, not verified binary patch capacity. Keep export disabled; v0.3 now permits extended preview only; no hardware tests performed.

## v0.3 editor update
- Commit 73653b223342a93eb3faf0dce1e20411af0b5ea3: 30-character plain-text preview, original byte budget, live extended-storage indicator, JSON metadata, percent directives excluded from new plain text. Dynamic formats remain locked. Export remains disabled. No browser or hardware test confirmed.

## v0.4 experimental export
- Commit 6230ffc1e824fc1e18159b0a5b4bbce97df73f20 enables exactly one same-length (8 ASCII bytes) DISARMED candidate edit, with explicit bench-test acknowledgement. Preserves original record types and addresses, recomputes changed checksums, reparses output and compares every byte/address. Candidate identity and firmware runtime safety remain unverified. No browser or hardware tests confirmed.
- JSON includes aligned raw pointer candidates for each string; these are not proven code references. Extended relocation/export remains blocked pending target flash layout and disassembly. Earlier export-disabled statements describe previous versions.

- v0.4.1 aligned columns and clarified export eligibility. v0.4.2 hides format placeholders in labels and displays illustrative dynamic values; raw strings preserved in analysis and firmware. Browser testing pending.

## First exported HEX comparison
Both uploaded original and experimental files independently parsed: 397979 bytes, 24886 records each, identical addresses, all checksums valid, one changed record and seven changed bytes. Export at 0x0805B760 changed NOT_DISARMED to NOT_FPSteVe! (suffix false positive); do not flash this export. Standalone DISARMED at 0x08062C86 has preceding and trailing NUL. v0.4.3 requires preceding NUL to conservatively reject suffixes; may miss valid strings. Actual OSD reference and hardware test remain unverified.

## Second export independent binary comparison
Original b5a42002(1).hex versus EXPERIMENTAL (1).hex: both 397979 data bytes, 24886 records, all record checksums valid, identical address sets, exactly one changed record and seven changed bytes. At 0x08062C86 standalone DISARMED becomes FPSteve! (8 bytes), surrounding bytes and NUL preserved. NOT_DISARMED at 0x0805B760 unchanged. Binary comparison passed; actual OSD behavior and hardware safety still unverified. Bench test only on spare FC with props removed and recovery backup.

## Agreed next milestone: v0.5 TEST (planned, not implemented)
FPSteVe approved preparing the UI and validation while awaiting the first spare-FC DISARMED hardware test. Keep current experimental export restrictions intact: one verified candidate, eight printable ASCII bytes, no percent directives, explicit bench-test acknowledgement. Do not unlock other messages or relocation yet.

### Limits and implementation plan
- Betaflight 4.5.5 standard OSD element buffer is 32 bytes including NUL; MSP DisplayPort caps strings at 30 characters; MAX7456 row width is 30. ESC warning formatting has a separate 12-character limit. These are not universal per-message permissions. See upstream osd.h, osd_warnings.h, osd_warnings.c, osd_elements.c, io/displayport_msp.c and drivers/max7456.c at tag 4.5.5.
- Provide up to 30-character previews for ordinary messages, with lower verified per-path limits where applicable; clearly distinguish preview/display limit from original allocated storage and export eligibility. Preserve dynamic format directives and account for worst-case rendered length; keep dynamic exports locked until verified. Do not allow blanking critical warnings.
- Add mandatory visible pre-download verification report: original/output byte counts, intentional and unexpected byte changes, address differences, checksum failures, PASS/FAIL and hardware-unverified notice. Fail closed. Existing export reparses output and compares every mapped byte/address against the intended patch.
- Longer replacements require verified flash/linker layout, reserved-region protection, actual ARM code reference analysis, and safe storage allocation; raw pointers, zero padding and HEX gaps are not proof of safe relocation. Verify each message's OSD-only references and rendering path before unlocking. Test shorter replacements and multiple edits separately, then expand across versions/targets.
- Hardware gate: spare FC, props removed, recovery backup, normal boot/configurator connection, intended DISARMED text, other OSD/config intact, arm/disarm behavior. Binary comparison is not runtime certification. No hardware result reported yet.
- Coordinate with SQ: read this file and latest source before changes, reconcile concurrent commits, update this file after implementation. Prefer grouping changes to reduce Pages deployments. This entry is a plan only; no app changes in this commit.
