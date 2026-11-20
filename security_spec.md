# Security Specification — Prompt Archive (`prompts`)

## 1. Data Invariants
1. Guests (unauthenticated users) can only `get` and `list` documents in `/prompts/{promptId}`.
2. Write operations (`create`, `update`, `delete`) on `/prompts/{promptId}` are strictly restricted to the verified admin account (`request.auth.token.email_verified == true && request.auth.token.email == 'cjinh1105@gmail.com'`).
3. Every document in `/prompts/{promptId}` must contain strictly the keys `['title', 'category', 'content', 'createdAt']` with no shadow fields.
4. `promptId` must match `^[a-zA-Z0-9_\-]+$` and be at most 128 characters.
5. `title` must be a non-empty string with length between 1 and 120 characters.
6. `category` must be one of `'실사·합성'`, `'스타일 변환'`, `'캐릭터·포스터'`.
7. `content` must be a non-empty string with length between 1 and 20,000 characters.
8. `createdAt` must equal `request.time` on document creation and remain immutable on updates.
9. Unmapped collections and paths are denied by default.

## 2. Adversarial "Dirty Dozen" Payloads (Expected: PERMISSION_DENIED)
1. **Unauthenticated Create**: Guest attempting to create a prompt.
2. **Non-Admin Authenticated Create**: Signed-in non-admin user attempting to create a prompt.
3. **Unverified Admin Email Spoof**: Token with `email == 'cjinh1105@gmail.com'` and `email_verified == false`.
4. **Non-Admin Update / Delete**: Non-admin user attempting to update or delete an existing prompt.
5. **Shadow Field Injection on Create**: Admin payload with extra `isAdmin: true` field.
6. **Oversized Document ID**: `/prompts/${"a".repeat(200)}`
7. **Empty / Oversized Title**: `""` or `"a".repeat(121)`
8. **Unsupported Category**: `category: "임의분류"`
9. **Resource Exhaustion Content (>20,000 chars)**: `content: "a".repeat(20001)`
10. **Spoofed Client Timestamp on Create**: `createdAt: Timestamp.fromMillis(1000)`
11. **Mutating Immutable `createdAt` on Update**: `{ createdAt: serverTimestamp() }`
12. **Write to Unmapped Collection**: `/admin/config` write attempt
