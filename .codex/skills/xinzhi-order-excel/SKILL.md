---
name: xinzhi-order-excel
description: Generate order Excel workbooks and freight settlement images using the user's “行致” and “承晟” pricing rules. Apply when an order is provided and the user requests either pricing mode or gives a freight amount.
metadata:
  short-description: 行致/承晟订单调价与运费结算
---

# 行致 / 承晟订单处理

Use the user's order document (spreadsheet, photo, scan, or handwritten sheet) to prepare a priced order workbook. When the user also provides “运费XX”, perform the freight allocation and create a settlement image.

## Pricing modes

Choose the multiplier from the user's trigger phrase:

- “行致”: source unit price × 1.03.
- “承晟”: source unit price × 1.115.

For either mode, round the adjusted unit price based on its value before rounding:

- Below 10: 2 decimal places.
- 10 to below 20: 1 decimal place.
- 20 or above: whole number.

Use ordinary Excel/half-up rounding for positive prices. Calculate each line amount as quantity × rounded unit price, then sum the line amounts.

## Read order documents

- Read item names, units, quantities, and source prices from the provided document. Preserve visible order identifiers and customer/date information when useful.
- For handwriting or unclear image text, make a best-effort reading from the marks and context. The user permits reasonable judgment calls; do not stop to ask about every unclear character or digit. Do not invent entries without a reasonable visual or contextual basis. Mention material guesses in the final response.
- If no order document or usable order data is available, ask the user to provide it.

## Excel output

When producing an Excel workbook, choose the worksheet layout based on the number of source order documents:

- For one source order document, create exactly two worksheets, in this order:
  1. `采购`: original order data, preserved unchanged.
  2. `发货`: linked item name, unit, and quantity; adjusted unit prices using the selected mode and rounding tiers; line amounts; and a final `合计` row.
- For two or more source order documents, create exactly one procurement worksheet per source, followed by one combined shipment worksheet. For N sources, create exactly N+1 worksheets in this order:
  1. `采购1` through `采购N`: each sheet contains the original data from its corresponding source document, in the order provided.
  2. `发货`: combine the order lines from all N sources into one normalized shipment table, then apply the same selected pricing mode, unit-price rounding tiers, line amount calculation, and final `合计` as in the one-source workflow.

For a multi-source order, do not omit any source or create separate shipment totals: the `发货` sheet contains all lines and one overall total. Preserve source information in the respective numbered procurement sheets. If a source is an image or scan, transcribe its order data into its procurement sheet and preserve the source distinction. Use formulas to reference procurement sheets and calculate prices/totals where practical. The multi-source layout changes only the procurement-sheet count and the consolidation of the `发货` sheet; all pricing, rounding, freight, and output-location rules remain unchanged. Verify sheet names and order, inclusion of lines from every source, representative price tiers, line amounts, and final total. Save the workbook directly to the user's Desktop by default; follow a different location if the user requests one.

## Freight settlement image

When the user provides an order and says “运费XX” with a freight amount `F`:

1. Let `T0` be the pre-freight grand total on `发货`, after applying the selected pricing mode and its unit-price rounding.
2. Calculate the freight-inclusive target `T1 = T0 + F × 1.1`.
3. Calculate `K = T1 ÷ T0` at full precision. Never round `K` for calculations.
4. For every line, multiply its already adjusted and tier-rounded unit price by `K`. Apply the same tiered rounding again to this new unrounded unit price. Recalculate line amounts from quantity × final rounded unit price, then sum them. Because unit prices are rounded, this final sum may differ from `T1`; use the final sum as the result and do not force a balancing adjustment.
5. Generate a new PNG table with item, unit, quantity, adjusted unit price, final settlement unit price, line amount, and total. Label the last row `合计`. Keep only the upper order table and total; omit any calculation-basis block below the table.
6. Name the PNG `<final sum>结算依据.png` (for example, `100结算依据.png`) and save it directly to the Desktop by default. Follow a different location or filename if requested.

Do not assume a freight amount. If the user requests only the freight settlement image, create the image without generating another Excel workbook unless they ask for one. Briefly report the pre-rounding freight-inclusive target and the final sum when they differ.

## Boundaries

- The user's confirmed rules are the two multipliers, the three price-rounding tiers, the ×1.1 freight addition, and proportional unit-price allocation described above.
- Do not add inferred taxes, discounts, currency conversions, or other charges. Freight is applied only when the user supplies its amount.
- Follow an explicit per-task multiplier, location, filename, or other rule when the user gives one.


