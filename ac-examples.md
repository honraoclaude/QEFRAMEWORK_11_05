# Reference: Acceptance Criteria — Before / After Examples

Use these patterns to rewrite weak acceptance criteria during story review.

## Pattern 1: Vague qualitative → specific quantitative

**Weak**
> The page should load quickly.

**Strong**
> Given a user on a 4G connection
> When they navigate to the dashboard
> Then the Largest Contentful Paint (LCP) is < 2.5s at p75
> And the page is interactive (TTI) within 3s at p75

## Pattern 2: "And" compound → multiple ACs

**Weak**
> User can search products and filter by category and sort by price and add to cart.

**Strong** — Split into four ACs, each independently testable:
> AC1: User can search products by keyword
> AC2: User can filter the result list by category
> AC3: User can sort the result list by price (ascending/descending)
> AC4: User can add a product from the result list to the cart

## Pattern 3: Solutionizing → behavior-focused

**Weak**
> Use Redis to cache the user profile so the API is faster.

**Strong**
> Given a user has logged in within the last hour
> When the profile endpoint is called repeatedly
> Then the response time at p95 is < 50ms
> (Implementation note: caching may be considered)

## Pattern 4: Happy path only → positive + negative

**Weak**
> User can upload a profile picture.

**Strong** — Add the negative paths:
> AC1: Given an image file ≤ 5MB in PNG/JPG/WebP format, user can upload it as profile picture
> AC2: Given a file > 5MB, upload is rejected with message "File too large. Max 5MB."
> AC3: Given an unsupported format (e.g., .bmp, .tiff, .gif), upload is rejected with message "Format not supported. Use PNG, JPG, or WebP."
> AC4: Given a file with image MIME type but malformed content, upload is rejected
> AC5: Given the upload service is unavailable, user sees retry option and existing picture is preserved

## Pattern 5: Untestable assertion → observable outcome

**Weak**
> The system should be secure.

**Strong**
> AC1: Authentication uses OAuth 2.0 PKCE flow; no credentials in URLs or logs
> AC2: Session tokens expire after 30 min of inactivity
> AC3: All API endpoints reject requests without valid bearer token (returns 401)
> AC4: Rate limiting: max 5 failed login attempts per IP per minute (returns 429)
> AC5: All PII fields are encrypted at rest (AES-256) and in transit (TLS 1.3)

## Pattern 6: Missing observability → instrumentable

**Weak**
> Track when users complete checkout.

**Strong**
> Given a user completes the checkout flow
> When the order confirmation page renders
> Then an analytics event `checkout_complete` is emitted with properties: order_id, user_id (hashed), total_amount, currency, item_count, payment_method, time_to_complete_ms

## Pattern 7: Hidden assumptions → explicit context

**Weak**
> User can reset their password.

**Strong** — Make the context explicit:
> AC1: Given a user with a registered, verified email address
> When they request password reset and follow the link in their email
> Then they can set a new password meeting complexity rules
>
> AC2: Given an unverified email address
> Then password reset request shows "Please verify your email first" with re-send option
>
> AC3: Given a non-existent email address
> Then the response is identical to the success case (no user enumeration)

## Pattern 8: One-shot → state-aware

**Weak**
> User receives a notification when their order ships.

**Strong**
> AC1: Given a user has email notifications enabled
> When the order transitions to "shipped"
> Then they receive exactly one email within 5 minutes
>
> AC2: Given the same order is updated again (e.g., tracking number changes)
> Then a second email is NOT sent (or sent only if explicitly required)
>
> AC3: Given the user has disabled email notifications
> Then no email is sent regardless of order events
>
> AC4: Given the email service is unavailable
> Then the notification is queued and retried with exponential backoff up to 24h

## The Gherkin discipline

When writing Given/When/Then:

- **Given** = preconditions (state, not actions). Past tense or stative verbs.
- **When** = the single action under test. Present tense, active voice.
- **Then** = observable outcome. State what changes, not how to check.
- **And** = adds to the previous Given/When/Then; never starts a new clause type.

**Avoid**:
- Putting actions in "Given" ("Given the user clicks submit...")
- Putting expectations in "When" ("When the page should display...")
- Multiple "When" steps in one scenario (split it)
- UI-coupling ("Then click the button" — describe outcome, not navigation)
