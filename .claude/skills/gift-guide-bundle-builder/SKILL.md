---
name: gift-guide-bundle-builder
description: Build Osmossi gift guides (price tiers and recipients) and value-offer bundles for gifting moments such as the holidays, Mother's Day, and Amor y Amistad. Use before each gifting moment, at least 3 weeks ahead.
---

# gift-guide-bundle-builder

Owner agent: `growth-planner`.

## Rules
- **No blanket discounts.** Offers are value offers: gift note, gift wrapping, a small gift
  with purchase, faster shipping. Standard US shipping is already free from $70, so free shipping
  is **not** a new offer for bags.
- A bundle discount or any offer that costs margin needs Juan Pablo's approval.
  TODO(input): minimum margin per order.
- In-stock products only; hero products lead each section.

## Steps
1. Pull active, in-stock products from Shopify.
2. Group the guide by **price tier** (how people shop for gifts):
   - **Under $50**: placemats.
   - **$150–$230**: Pausayu bags, palm hats, wall decor.
   - **The statement gift, $467–$499**: Opalo bum bags and mini bags (lead with bum bags).
3. Add **recipient angles** in the copy, not as separate collections: "for your mamá", "for
   your sister", "for the hostess", "for yourself".
4. Propose 1–2 bundles as value offers (for example, a bag + a placemat set for the hostess),
   with the retail total and what the customer gets extra.
5. Spec the Shopify collection: title, handle, description (brand voice), product order, and
   the date it must be live. **Creating or publishing the collection is a partner action**:
   draft it here; a partner approves.

## Output format
A guide spec in `docs/plans/`: tiers with product links, intro copy, bundle proposals, the
Shopify collection spec, and the dates it must be live.
