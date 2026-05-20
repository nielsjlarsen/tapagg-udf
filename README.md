# Arista TapAgg UDF Builder

A single-file, self-hosted webpage for crafting Arista TapAgg User Defined Field
(UDF) ACL rules by clicking on packet fields — Wireshark-style.

![Screenshot of the UDF builder showing a UDP-over-IPv4 packet with the first byte of the IPv4 header selected, and the generated rule at the bottom](docs/screenshot.png)

*Above: a UDP-over-IPv4 packet with the **Version + IHL** byte selected. The hex
pane shows the bytes colour-coded by layer (Ethernet → IPv4 → UDP → Payload), the
matching field is lit up in the field tree, and the generated rule lands at the
bottom:*

```
[sequence] [permit | deny] [protocol] [source] [destination] payload header start offset 0 pattern 0x45000000 mask 0x00ffffff
```

*The dimmed `[sequence] [permit | deny] [protocol] [source] [destination]` part is just a placeholder
showing where to splice the rule into your own ACL line — **Copy** puts only the
`payload …` portion on the clipboard.*

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
   - TCP / UDP / ICMP, or
   - **GRE** (plain) — configurable Protocol Type and C/K/S flags, or
   - **ERSPAN** Type II / Type III (over GRE) — auto-forces IP Protocol=47,
     hides the L4 / Payload controls, and exposes an inner-frame builder
     for the captured frame (its own Ethernet/VLAN/IPv4/TCP/UDP/payload).
2. The hex pane shows the resulting bytes coloured by layer — the same accent
   colour is used in the legend swatch, the hex byte tint, and the field-tree
   group header.
3. Click a field in the tree, click bytes in the hex pane, or enable
   **Bit-level view** to click individual bits. Modifiers:
   - Plain click — replace the selection
   - **Shift-click** — extend a contiguous range from the previous click
   - **⌘ / Ctrl-click** — toggle (add or remove a *separate* block)
   - **Clear selection** — empty everything

   Each click generates the matching UDF rule, e.g.

   ```
   payload header end offset 0 pattern 0x45b80000 mask 0x0000ffff
   ```

   - The `offset` is in **4-byte words**, measured either from the **start**
     or the **end** of the IP header (Header keyword radios). On Arista,
     GRE is parsed as part of the IP header chain, so `header end offset 0`
     lands after the GRE header when GRE/ERSPAN is on the wire.
   - The `pattern` and `mask` are 32-bit, MSB-aligned. The `mask` is a reverse
     wildcard mask (1 = ignore, 0 = care).
   - If your selection spans more than 4 bytes, multiple rules are emitted
     (all must match in the ACL).
4. Hit **Copy** to put the rule on your clipboard.
