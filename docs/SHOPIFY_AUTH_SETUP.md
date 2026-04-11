# Frontend Shopify App Proxy Setup

This document describes how the theme frontend is connected to the backend safely.

The current implementation uses Shopify App Proxy for storefront interactions.

## 1) Theme behavior

The Journey block in:
- `blocks/ai_gen_block_8925a30.liquid`

defaults to this API base:
- `/apps/ro-adventure-community`

This means browser requests are made to storefront proxy URLs, not directly to Railway.

## 2) Required Shopify App Proxy config

In Shopify Dev Dashboard, app version settings:
- `prefix`: `apps`
- `subpath`: `ro-adventure-community`
- `url`: `https://ro-adventure-backend-production-5546.up.railway.app/api/public`

Release the version after changing proxy settings.

## 3) Backend routes used by frontend

The block uses:
- `GET /apps/ro-adventure-community/posts`
- `POST /apps/ro-adventure-community/posts`
- `POST /apps/ro-adventure-community/posts/:id/comments`
- `POST /apps/ro-adventure-community/posts/:id/like`

Shopify forwards these to:
- `/api/public/posts`
- `/api/public/posts/:id/comments`
- `/api/public/posts/:id/like`

and adds signed query parameters (`signature`, `shop`, `timestamp`, etc).

## 4) Security model

1. Frontend writes are accepted only on `/api/public/...` with valid App Proxy signature.
2. Legacy direct backend write routes (`/api/posts...`) are admin-key only.
3. Shopify admin sync routes are admin-key only.
4. Webhook route is HMAC-verified.

## 5) Frontend test checklist

1. Open:
- `https://<shop-domain>/apps/ro-adventure-community/posts`
- Expect: JSON with `ok: true`

2. On storefront page with Journey block:
- create post works
- comment works
- like works

3. Network tab should show requests to `/apps/ro-adventure-community/...`
not direct Railway `/api/public/...`.

4. Direct Railway request should fail:
- `https://ro-adventure-backend-production-5546.up.railway.app/api/public/posts`
- Expect: `401` signature error

## 6) Troubleshooting

1. `Cannot GET /api/public/posts`
- Railway is on old deploy. Redeploy latest backend.

2. `Invalid app proxy signature`
- `SHOPIFY_API_SECRET` mismatch in Railway.
- Proxy requests not going through `/apps/...` path.
- App proxy URL/prefix/subpath mismatch in Dev Dashboard.

3. Writes fail only on one domain
- Ensure you are testing on the store that installed this app version.
- Confirm app proxy is released and installed for that store.
