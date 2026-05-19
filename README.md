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

1. Configure the packet on the left (Ethernet, optional VLAN/QinQ, IPv4, optional
   TCP/UDP, payload bytes).
2. The hex pane shows the resulting bytes coloured by layer.
3. Click a field in the tree, click bytes in the hex pane, or enable
   **Bit-level view** to click individual bits. Modifiers:
   - plain click — replace the selection
   - **Shift-click** — extend a contiguous range from the previous click
   - **⌘ / Ctrl-click** — toggle (add or remove a *separate* block)
   - **Clear selection** — empty everything

   Each click generates the matching UDF rule, e.g.

   ```
   permit ip any any payload header start offset 0 pattern 0x45b80000 mask 0x0000ffff
   ```

   - `offset` is in **4-byte words** from the start of the selected reference
     layer (auto by default; selectable via the dropdown). So byte 8 of the IP
     header → offset 2.
   - `pattern`/`mask` are 32-bit, MSB-aligned. `mask` uses Cisco-style wildcard
     (1 = ignore, 0 = care).
   - If your selection spans more than 4 bytes, multiple rules are emitted
     (all must match in the ACL).
4. Hit **Copy** to put the rule on your clipboard.

The **Header keyword** dropdown picks between `header end` (default) and
`header start`, in case your EOS version / platform expects one or the other.
