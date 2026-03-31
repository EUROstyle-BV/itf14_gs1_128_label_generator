\# ITF-14 / GS1-128 Label Generator



Always read `mission.md` first before any coding task.



\## Key facts

\- Output: single `index.html`, no build tools, no npm

\- Barcodes: JsBarcode CDN (ITF14 + CODE128 formats)

\- GS1-128 top, ITF-14 bottom (bearer frame)

\- PI=0 for GS1-128 AI(02), PI=1 for ITF-14

\- Default test data: EAN `8711731033602`, batch `06022629`, THT `2028-03-31`

\- Verify: `gs1CheckDigit("1871173103360") === 9`

