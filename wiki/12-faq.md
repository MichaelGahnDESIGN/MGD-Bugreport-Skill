# 12. FAQ

## Do I need a separate update server?

Not always. A static JSON file on normal web hosting can be enough for phase 1.

## Should I build an auto-updater first?

Usually no. Start with a manual update notice.

## Can I use GitHub Releases?

Yes for public releases. Avoid private tokens in shipped apps.

## Are checksums enough?

Checksums help detect corruption. For serious products, use signing too.

## What about mobile apps?

Use app stores for binary updates and your own backend for minimum version checks and content updates.
