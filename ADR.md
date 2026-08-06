# Architecture Decision Record

## NAV D/F use Right Option

**Status:** Accepted

### Context

Karabiner-Elements uses Left Option as a held modifier to reverse horizontal
scroll-ring input. The NAV-layer D and F bindings previously emitted shortcuts
that included Left Option, causing Karabiner to activate the scroll modifier
while those shortcuts were used.

### Decision

The NAV-layer D and F bindings use Right Option (`RA`) rather than Left Option
(`LA`). Their other modifiers remain unchanged: Left Shift, Left Control, and
Left GUI.

### Consequences

Left Option remains exclusively available for the Karabiner scroll-ring rule.
The NAV D and F shortcuts now send Right Option, so any application-specific
shortcut mappings must recognize that modifier combination.
