
# Sketchpad for unification of nonces

The nonce challenge at the same time serves as a CSRF protection. However, if cryptographic holder binding is not used, the nonce challenge cannot serve this function. [@openid:security]

```yaml

  # `state`: A message-specific value, based on a secure (pseudo-)random seed, binding ...
  enables the client to
  verify the validity of the request
  validate that the authz response is not altered and sent by the original server which the authz request was sent
  cross check the authorization request and response

protect the callback endpoint by binding original authorization requests to responses.
- State provides a separate layer of CSRF protection by validating the integrity of the redirect flow.

   by matching the binding value to the user-agent's authenticated state

  #
  # OPTIONAL URL
  # - OPTIONAL UNIQUE for OAuth 2.x & UMA
  #
  # Aliases:
  #
  [root].state?: string # OAUTH & UMA (opaque) client state (authz req)


  # `challenge`:
  protects the token exchange process
  binding the code to the client (validated by the authorization server)
  #
  # RECOMMENDED UNIQUE string
  # - REQUIRED for OAuth 2.x PKCE
  #
  # Aliases:
  #
  [root].code_challenge_method?: string # OAUTH (authz req.): plain | S256 | [extensions]
  [root].code_challenge?: string # OAUTH (authz req.)


  hash_method?: string


  # `nonce`: An interaction-specific value, based on a secure (pseudo-)random seed, binding ...
  - "per-session state" that binds (signed) tokens to (the initial/original request of) a client (session/flow), who can use it to verify whether a callback/token stems from a known authentication flow, thus mitigating replay attacks.
  It MUST be different for each negotiation; it MUST NOT be reused, even in case of negotiation failure.

  ...store a cryptographically random value as an HttpOnly session cookie or local storage, and use a cryptographic hash of the value as the nonce parameter. In that case, the nonce in the returned ID Token is compared to the hash of the session cookie to detect ID Token replay by third parties.

  (A less secure way to protect against replay attacks would be to check the aud (audience) claim of the token.)

  #
  # RECOMMENDED URL
  # - REQUIRED UNIQUE for GNAP
  #
  # Aliases:
  #
  nonce: string


  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  # - privileges?: string   # GNAP & OAuth 2.x RAR
  # - locations?: string    # GNAP & OAuth 2.x RAR
  # - datatypes?: string    # GNAP & OAuth 2.x RAR
  [...]?: any


  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  thread: context_handle

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  stitches:
  ditch:
  seam:

NONCE: known content not hashed and not signed -> what
CHALLENGE: secret content hashed and not signed -> what, how (alg), where
HMAC: known content hashed and signed with shared key -> what, how (alg), which (key)
PKI: known content hashed and signed with private key -> what, how (alg), who (key)

keys: JWKSet

S:
  - secret Ss1
  - (content Cs1)
  - method Ms1
  - hash/signature Hs1
S->R:
  - hash Hs1
  - method Ms1
R:
  - validate Hs1 (if Ms1 immediate)
  - secret Sr1
  - (content Cr1)
  - method Mr1
  - hash/signature Hr1
R->S:
  - hash Hs1
  - hash Hr1
  - method Ms1
S:
  - validate Hr1 (if Mr1 immediate)
  - secret Ss2
  - (content Cs2)
  - method Ms2
  - hash/signature Hs2
S->R:
  - secret Ss1 (if Ms1 indirect)
  - hash Hr1
  - hash Hs2
  - method Ms2
R:
  - validate Hs1 (if Ms1 indirect)
  - validate Hs2 (if Ms2 immediate)
etc...

[stitches]

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  zig: client nonce, hashed

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  zag: server nonce, hashed

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  back: correspondent nonce, returned

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  tack: (bast) manual temporary stitch to hold fabric in position
  mark: dots/pins on a pattern, indicating positions

  snip: (single or double; nips, or balance marks) seam allowance for accurate junctions; reduce fabric bulk
    notch: for CONCAVE seams
    clip: for CONVEX seams
    trim: equal

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  lock: last seed

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  cast (on):

  # `[param]`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  bind/cast (off):


  # `nonce`: Any other parameter that details this interaction finish method.
  #
  # OPTIONAL any
  #
  # Examples:
  # - privileges?: string   # GNAP & OAuth 2.x RAR
  # - locations?: string    # GNAP & OAuth 2.x RAR
  # - datatypes?: string    # GNAP & OAuth 2.x RAR
  [...]?: any
````
