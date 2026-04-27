# Security Specification - Vanta Store

## Data Invariants
1. **Product Integrity**: Products can only be modified by admins. Standard users have read-only access.
2. **Identity Isolation**: Users can ONLY read/write their own private data and cart.
3. **Price Protection**: Users cannot modify the price of items when adding them to their cart.
4. **PII Protection**: Private data (email, address) is stored in a `private/data` document that is ONLY accessible by the owner.

## The "Dirty Dozen" Payloads (Denial Tests)

1. **Self-Promotion**: Authenticated user tries to create an `admin` role document for themselves.
2. **Price Spoof**: User tries to add a product to their cart with a `price` field that they've changed (Price should be fetched on the backend or strictly validated for display). *Note: In this client-only setup, we'll focus on cart ownership.*
3. **Cart Hijacking**: User A tries to add an item to User B's cart by changing the `userId` in the path or payload.
4. **PII Snooping**: User A tries to `get()` User B's `/users/userId/private/data`.
5. **Product Sabotage**: User tries to `update()` or `delete()` a document in the `/products/` collection.
6. **Ghost Field Injection**: User tries to `create()` a profile with an `isVerified: true` field that isn't in the schema.
7. **Resource Exhaustion**: User tries to create a cart item with a `productId` that is 1MB of junk text.
8. **Invalid ID**: User tries to target a document with an ID containing malicious characters (e.g. `../`).
9. **Email Spoofing**: User tries to register with an email but `email_verified` is false, yet they attempt to access "verified-only" features.
10. **State Shortcutting**: User tries to set their cart item `addedAt` to a time in the future.
11. **Negative Quantity**: User tries to add `-5` items to their cart.
12. **Orphaned Cart**: User tries to add a cart item for a `productId` that does not exist in the `/products` collection.

## Test Runner (Logic Outline)
We will verify:
- `allow read: if true` for `/products`.
- `allow write: if false` for `/products` (except admins).
- `allow read, write: if request.auth.uid == userId` for `/users/{userId}/cart/`.
- `allow read, write: if request.auth.uid == userId` for `/users/{userId}/private/`.
- `allow read: if isSignedIn()` for `/users/{userId}/public/`.
