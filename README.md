# Arista TapAgg UDF Builder

A single-file, self-hosted webpage for crafting Arista TapAgg User Defined Field
(UDF) ACL rules by clicking on packet fields — Wireshark-style.

## Run it

Just open `index.html` in any modern browser, or serve the folder statically:

```sh
cd tapagg-udf
python3 -m http.server 8080
# then visit http://localhost:8080
```

No build step, no dependencies, no network.

## How it works

1. Configure the packet on the left:
   - Ethernet (+ optional 802.1Q / Q-in-Q VLAN tags)
   - **MPLS** label stack (1–6 labels with Label/TC/TTL; S bit auto-set)
   - IPv4 (with options; TotalLen/checksum auto-computed)
   - TCP or UDP, or
   - **ERSPAN** Type II / Type III (over GRE) — auto-forces IP Protocol=47,
     hides the L4 / Payload controls, and exposes an inner-frame builder
     for the captured frame (its own Ethernet/VLAN/IPv4/TCP/UDP/payload).
2. The hex pane shows the resulting bytes coloured by layer.
3. Click a field in the tree, click bytes in the hex pane, or enable
   **Bit-level view** to click individual bits. Modifiers:
   - Plain click — replace the selection
   - **Shift-click** — extend a contiguous range from the previous click
   - **⌘ / Ctrl-click** — toggle (add or remove a *separate* block)
   - **Clear selection** — empty everything

   Each click generates the matching UDF rule, e.g.

   ```
   permit ip any any payload header start offset 0 pattern 0x45b80000 mask 0x0000ffff
   ```

   - The `offset` is in **4-byte words** from the start of the selected
     reference layer (auto by default; selectable via the dropdown). So byte 8
     of the IP header → offset 2.
   - The `pattern` and `mask` are 32-bit, MSB-aligned. The `mask` is a reverse
     wildcard mask (1 = ignore, 0 = care).
   - If your selection spans more than 4 bytes, multiple rules are emitted
     (all must match in the ACL).
4. Hit **Copy** to put the rule on your clipboard.
