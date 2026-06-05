
# Summary of GNAP

## Grant Negotiation \& Authorization Protocol (RFC 9635)

A mechanism for delegating authorization to a piece of software (the client instance) and conveying the results and artifacts of that delegation (access to a set of APIs as well as subject information) to the software.

GNAP allows the client instance to request, negotiate, and receive delegated authorization to
  - resource servers (which can be used to access resources and APIs on behalf a resource owner)
  - subject information (whihc can be used to make authenticaiton decidions)

This delegation is facilitated by an authorization server, usually on behalf of a resource owner.

The end user operating the software can interact with the authorization server to authenticate, provide consent, and authorize the request as a resource owner.

The process by which the delegation happens is known as a grant, and GNAP allows for the negotiation of the grant process over time by multiple parties acting in distinct roles.

### Roles

Authorization Server (AS):
  The server, uniquely defined by the (absolute) grant endpoint URI, that grants delegated privileges (access, subject information) to a particular instance of client software.

Client:
  Application that consumes resources, operated by the end user, or run autonomously on behalf of a resource owner. Differentiate between specific instance, identified by unique key, and software running the instance.

Resource Server (RS):
  Server that provides (protected) resources (API).

Resource Owner (RO):
  Subject entity that may grant or deny operations on resources it has authority upon.

End user:
  Natural person that operates a client instance. This may or may not be the same entity as the RO.

### Elements

In addition to the roles above, the protocol also involves several elements that are acted upon by the roles throughout the process.

Protected Resource:
  Protected API that is served by an RS and that can be accessed by a client, if and only if a valid and sufficient access token is provided.

  !! GNAP mentiones that '''To avoid complex sentences, the specification document may simply refer to "resource" instead of "protected resource".''' Does thi s entail the same public-resource problem as with UMA?

Access Token:
  A data artifact representing a set of rights and/or attributes.

Grant:
  The permission of an instance of client software to receive some attributes at a specific time and with a specific duration of validity and/or to exercise some set of delegated rights to access a protected resource; or the act of granting such permission to a client instance.

Privilege:
  Right or attribute associated with a subject, defined and maintained by the RO, who might temporarily delegate some of them to an end user ("privilege delegation").

Right:
  Ability given to a subject to perform a given operation on a resource under the control of an RS.

Subject:
  Person or organization. The subject decides whether and under which conditions its attributes can be disclosed to other parties.

Subject Information:
  Set of statements and attributes asserted by an AS about a subject.

### Trust relationships

GNAP defines its trust objective as follows: "the RO trusts the AS to ensure access validation and delegation of protected resources to end users, through third party clients."

This can be decomposed into trust relationships between software elements and roles:

end user/RO (when different):
  The end user needs some out-of-band mechanism of getting the RO consent. GNAP generally assumes that humans can be authenticated, thanks to identity protocols.

end user/client:
  The client acts as a user agent. Depending on the technology used, some interactions may or may not be possible, or may require client developers to implement requirements / recommendations / best practices, so that the end users may confidently use their software.

end user/AS:
  During authorization, the end user might interact with the AS through an AS-provided interface.

client/AS:
  An honest AS may face an attacker's client (as discussed just above), or the reverse. GNAP aims to make common attacks impractical, with opaque access tokens, well-defined request/response schemes, avoiding extra trust hypotheses, cryptographic attestations etc.

RS/RO:
  The RS promises to protect the RO's resources from unauthorized access and only accepts valid access tokens issued by a trusted AS.

AS/RO:
  The AS is expected to follow the decisions made by the RO. Privacy considerations aim to reduce the risk of an honest but too-curious AS or the consequences of an unexpected user data exposure.

AS/RS:
  The AS promises to issue valid access tokens to legitimate client requests (due diligence).

### Protocol Flow

GNAP is designed as a multi-stage, stateful process, in which the underlying requested grant moves through several states, as different actions take place in which parties provide information into the system to alter/augment its state and artifacts.

The state of the grant request is defined and managed by the AS, though the client instance also needs to manage its view of the grant request over time. The means by which these roles manage their state are outside the scope of this specification. [...] While it is possible to deploy an AS in a stateless environment, GNAP is a stateful protocol, and such deployments will need a way to manage the current state of the grant request in a secure and deterministic fashion without relying on other components, such as the client software, to keep track of the current state.

[Processing]

Grant requests are in this state when
- they are created upon receiving an access request
- when an they are updated by the client instance (and the interaction is completed)

In this state, the AS processes the context of the grant request. On exiting this state, a response can be returned to the client instance.

- If approval is required, the request moves to the pending state, and the AS returns a continuation response, along with any appropriate interaction responses.
- If no such approval is required, such as when the client instance is acting on its own behalf or the AS can determine that access has been fulfilled, the request moves to the approved state.
- If the AS determines that no additional processing can occur (such as a timeout or an unrecoverable error), the grant request is moved to the finalized state and is terminated.

[Pending]

When a request needs to be approved by an RO, or interaction with the end user is required, the grant request enters a state of pending. In this state, no access tokens can be granted, and no subject information can be released to the client instance. While a grant request is in this state, the AS seeks to gather the required consent and authorization (Section 4) for the requested access. A grant request in this state is always associated with a continuation access token bound to the client instance's key (see Section 3.1 for details of the continuation access token). If no interaction finish method (Section 2.5.2) is associated with this request, the client instance can send a polling continuation request (Section 5.2) to the AS. This returns a continuation response (Section 3.1) while the grant request remains in this state, allowing the client instance to continue to check the state of the pending grant request. If an interaction finish method (Section 2.5.2) is specified in the grant request, the client instance can continue the request after interaction (Section 5.1) to the AS to move this request to the processing state to be re-evaluated by the AS. Note that this occurs whether the grant request has been approved or denied by the RO, since the AS needs to take into account the full context of the request before determining the next step for the grant request. When other information is made available in the context of the grant request, such as through the asynchronous actions of the RO, the AS moves this request to the processing state to be re-evaluated. If the AS determines that no additional interaction can occur, e.g., all the interaction methods have timed out or a revocation request (Section 5.4) is received from the client instance, the grant request can be moved to the finalized state.

[Approved]

When a request has been approved by an RO and no further interaction with the end user is required, the grant request enters a state of approved. In this state, responses to the client instance can include access tokens for API access (Section 3.2) and subject information (Section 3.4). If continuation and updates are allowed for this grant request, the AS can include the continuation response (Section 3.1). In this state, post-interaction continuation requests (Section 5.1) are not allowed and will result in an error, since all interaction is assumed to have been completed. If the client instance sends a polling continuation request (Section 5.2) while the request is in this state, new access tokens (Section 3.2) can be issued in the response. Note that this always creates a new access token, but any existing access tokens could be rotated and revoked using the token management API (Section 6). The client instance can send an update continuation request (Section 5.3) to modify the requested access, causing the AS to move the request back to the processing state for re-evaluation. If the AS determines that no additional tokens can be issued and that no additional updates are to be accepted (e.g., the continuation access tokens have expired), the grant is moved to the finalized state.

[Finalized]

After the access tokens are issued, if the AS does not allow any additional updates on the grant request, the grant request enters the finalized state. This state is also entered when an existing grant request is revoked by the client instance (Section 5.4) or otherwise revoked by the AS (such as through out-of-band action by the RO). This state can also be entered if the AS determines that no additional processing is possible, for example, if the RO has denied the requested access or if interaction is required but no compatible interaction methods are available. Once in this state, no new access tokens can be issued, no subject information can be returned, and no interactions can take place. Once in this state, the grant request is dead and cannot be revived. If future access is desired by the client instance, a new grant request can be created, unrelated to this grant request.

### Sequences

Overall Sequence of GNAP

(A) The end user interacts with the client instance to indicate a need for resources on behalf of the RO. This could identify the RS that the client instance needs to call, the resources needed, or the RO that is needed to approve the request. Note that the RO and end user are often the same entity in practice, but GNAP makes no general assumption that they are.

(1) The client instance determines what access is needed and which AS to approach for access. Note that for most situations, the client instance is pre-configured with which AS to talk to and which kinds of access it needs, but some more dynamic processes are discussed in Section 9.1.

(2) The client instance requests access at the AS (Section 2).

(3) The AS processes the request and determines what is needed to fulfill the request (see Section 4). The AS sends its response to the client instance (Section 3).

(B) If interaction is required, the AS interacts with the RO (Section 4) to gather authorization. The interactive component of the AS can function using a variety of possible mechanisms, including web page redirects, applications, challenge/response protocols, or other methods. The RO approves the request for the client instance being operated by the end user. Note that the RO and end user are often the same entity in practice, and many of GNAP's interaction methods allow the client instance to facilitate the end user interacting with the AS in order to fulfill the role of the RO.

(4) The client instance continues the grant at the AS (Section 5). This action could occur in response to receiving a signal that interaction has finished (Section 4.2) or through a periodic polling mechanism, depending on the interaction capabilities of the client software and the options active in the grant request.

(5) If the AS determines that access can be granted, it returns a response to the client instance (Section 3), including an access token (Section 3.2) for calling the RS and any directly returned information (Section 3.4) about the RO.

(6) The client instance uses the access token (Section 7.2) to call the RS.

(7) The RS determines if the token is sufficient for the request by examining the token. The means of the RS determining this access are out of scope of this specification, but some options are discussed in [GNAP-RS].

(8) The client instance calls the RS (Section 7.2) using the access token until the RS or client instance determines that the token is no longer valid.

(9) When the token no longer works, the client instance rotates the access token (Section 6.1).

(10) The AS issues a new access token (Section 3.2) to the client instance with the same rights as the original access token returned in (5).

(11) The client instance uses the new access token (Section 7.2) to call the RS.

(12) The RS determines if the new token is sufficient for the request, as in (7).

(13) The client instance disposes of the token (Section 6.2) once the client instance has completed its access of the RS and no longer needs the token.


## GNAP Resource Server Connections (RFC 9767)

Access tokens
= A mechanism for an AS to provide a client instance limited access to an RS.
= Artifacts representing a particular set of access rights granted to the client instance to act on behalf of the RO.

### Concept (consistent across all systems)

Shared access token model incorporating a (non-exhaustive/-universal/-comprehensive) common set of aspects as guidance for implementers to facilitate structural interoperability. Where possible, mappings to the JWT standard format are provided.

#### Value

= the string that is passed on the wire between parties, which must be unique within a security domain, to be differentiable at runtime

  - When the token is structured according to a format known to AS and RS, other items in this model are contained within the token's value.
  - When the token is unstructured, other items is this model are usually retrieved via some kind of introspection service.

!! this value is conveyed in `value` field of a GNAP `access_token` response !!

#### Issuer

= the identity of the AS (for recognition by the RS, not as information to the client), typically `iss` in JWT

#### Audience

= an agreed upon identifier (e.g., URI) of the RSs the access token is intended for

This allows each RS to ensure that it is not receiving a token intended for someone else, typically `aud` in JWT

More complex cases can make use of the `location` field of objects in the GNAP access array to convey additional audience information (in which case the client might need to know this information to differentiate between RSs)

#### Key Binding

GNAP's Access tokens (except Bearer tokens) are bound to the client instance's registered or presented key, except in cases where the access token is a bearer token.

AS and RS need to be able to identify which (public or shared) key the token is bound to, and what the parameters are for the proofing mechanism (e.g., signing/digest algorithm), so that an attacker cannot substitute their own key.

The source of this key information can vary (decided by AS):
  - All tokens issued to a client instance can always be bound to that client instance's current key.
  - Each token could be bound to a specific key that is managed separately from client instance information.

Typically `cnf` in JWT or `key` in introspection response.
This value is conveyed to the client instance in the `key` field of the GNAP access_token response, but in the common case that it concerns the client instance's registered key, this field can be omitted.

#### Flags

= GNAP data flags associated with the token, indicating special processing or considerations, typically `flags` in introspection response

Client can request a set of flags using the `flags` field of the GNAP access_token grant request parameter.
These flags are conveyed from the AS to the client in the `flags` field of the GNAP access_token section of the grant response

#### Access Rights

= the set of access rights associated with an access token, which specify what/how/where the token can be used, typically `access` in introspection response

These are calculated from
- the rights available to the client instance making the request
- the rights available to be approved by the RO
- the rights actually approved by the RO
- the rights corresponding to the RS in question

The rights for a specific access token are a subset of the overall rights in a grant request.

These rights are requested by the client instance in the `access` field of the GNAP access_token request.
The rights associated with an issued access token are conveyed to the client instance in the `access` field of the GNAP access_token response.

#### Time Validity Window

The issuance time of the token, typically `iat` in JWT or introspection response
The expiration time of a token, after which it is to be rejected, typically `exp` in JWT or a token introspection response.
The starting time of a token's validity window, before which it is to be rejected, typically `nbf` in JWT or a token introspection response.

Since access tokens could be revoked at any time for any reason outside of a client instance's control, the client instance often does not know or concern itself with the validity time window of an access token. However, this information can be made available to it by using the `expires_in` field of a GNAP access token response

#### (internal) Token Identifier

= a unique internal identifier to allow the AS to differentiate between multiple separate tokens. This internal value can often also be used as the external identifier. Typically `jti` in JWT or introspection response

#### ((Authorizing)) Resource Owner

= the identity of the RO, which can be used by the RS to determine exactly which resource to access or which kinds of access to allow, typically `sub` in JWT or introspection response

!! For example, an access token used to access identity information can hold a user identifier to allow the RS to determine which profile information to return. The nature of this information is subject to agreement by the AS and RS. !! ISSUE: this breaks separation of concerns

!! In many cases, returning this information to the client instance would be a privacy violation on the part of the AS.

!! GNAP **does** allow for the return of subject information separately from the access token, in the form of identifiers and assertions. These values are returned directly to the client separately from any access tokens that are requested, though it's common that they represent the same party.

#### End User

= The end user is the party operating the client software instance, which facilitates their interaction with the AS

#### Client Instance

= the identity of the specific client instance to which the AS has issued the token, typically `client_id` in JWT or `instance_id` in introspection response

This allows the RS to process the token in a client specific manner (e.g., to verify client-bound tokens)

#### (Token) Label

= an identifier, unique per grant request, for the access token to differentiate between multiple tokens in a request/response, typically `label` in introspection response

A client instance can request a specific label using the `label` field of a GNAP access_token request
The AS can inform the client instance of a token's label using the `label` field of a GNAP access_token response

#### Parent Grant Request

= the specific grant request from a client instance that forms the context the issuance of this token

It is a unique tuple of:

  - The AS processing the grant request
  - The client instance making the grant request
  - The RO (or set of ROs) approving the grant request (or needing to approve it)
  - The access rights granted by the RO
  - The current state of the grant request

...

  The AS can use this information to tie common information to a specific token. For instance, instead of specifying a client instance for every issued access token for a grant, the AS can store the client information in the grant itself and look it up by reference from the access token.

  The AS can also use this information when a grant request is updated. For example, if the client instance asks for a new access token from an existing grant, the AS can use this link to revoke older non-durable access tokens that had been previously issued under the grant.

  A client instance will have its own model of an ongoing grant request, especially if that grant request can be continued using the API defined in Section 5 of [GNAP] where several pieces of statefulness need to be kept in hand. The client instance might need to keep an association with the grant request that issued the token in case the access token expires or does not have sufficient access rights, so that the client instance can get a new access token without having to restart the grant request process from scratch.

  Since the grant itself does not need to be identified in any of the protocol messages, GNAP does not define a specific grant identifier to be conveyed between any parties in the protocol. Only the AS needs to keep an explicit connection between an issued access token and the parent grant that issued it.


#### AS-Specific Access Tokens

Certain access tokens can be used at the GNAP APIs themselves:
  - a continuation access token (`continue` field of grant response) for the grant continuation API
  - a token management access token (`manage` field of the grant response) for the token management API
  - a resource server management access token for the RS-facing API

The AS MUST separate these access tokens from other access tokens used at an RS, e.g. by
- use of a flag on the access token data structure
- use of a special internal access right
- any other means

Unlike other access tokens, which are only opaque to the client, the contents of these AS-specific access tokens are also opaque to the RS. When introspection is used, the AS MUST NOT return an active value of true for AS-specific access tokens to the RS.

The client MUST also differentiate these special-purpose access tokens from access tokens used at an RSs, e.g. by storing AS-specific access tokens separately. SInce these tokense should never be presented to the RS, any attempts to process one MUST fail.

### Format (varies in different systems)

!! GNAP makes no assumptions on the format or contents !!

- They are agreed and communicated dynamically between AS and RS
  - AS can declare support in RS-facing discovery
  - RS can require a specific token format
  - AS can return the token's format in an introspection response
- They are opaque to the client

- They can be self-contained structured tokens, which allow conveyance of information without need for the RS to call the AS at runtime, and in which case the AS does not need to store it (rather, a signed token identifier can be transfered and validated)
- They can be used with introspection
- They can be both of the above.

- They can allow the RS to derive sub-tokens without having to call the AS

### RS-facing API

The AS can offer an RS-facing API consisting of any of the following: discovery, introspection, token chaining, resource reference registration.

#### Discovery

...

#### Introspection

...

#### Token chaining

...

#### Resource reference registration

...

### Error from the RS-facing API

The AS can respond to the RS with HTTP status code 400 (Bad Request) and a JSON object consisting of a single `error` field, which is either an object or a string.

The error value is
  - the error code in case of a string
  - the error object in case of an object

An error object contains the following fields:
  REQUIRED code: error code defining the error (ASCII)
  OPTIONAL description: human-readable description intended for the developer of the client

This specification defines the following error code values:
"invalid_request": A required parameter is invalid, missing, or otherwise malformed.
"invalid_resource_server": The RS was not recognized/allowed by the AS, or the RS's signature validation failed.
"invalid_access": RS is not permitted to register/introspect for the requested "access" value.

### Deriving a Downstream Token

Some architectures require an RS to act as a client instance and use a derived access token for a secondary RS. Since the RS is not the same entity that made the initial grant request, the RS is not capable of referencing or modifying the existing grant. As such, the RS needs to request or generate a new access token for its use at the secondary RS. This internal secondary token is issued in the context of the incoming access token.

While it is possible to use a token format (Section 2) that allows for the RS to generate its own secondary token, the AS can allow the RS to request this secondary access token using the same process used by the original client instance to request the primary access token. Since the RS is acting as its own client instance from the perspective of GNAP, this process uses the same grant endpoint, request structure, and response structure as a client instance's request.

  1. The client instance calls RS1 with an access token.
  2. RS1 presents that token to the AS to get a derived token for use at RS2. RS1 indicates that it has no ability to interact with the RO. Note that RS1 signs its request with its own key, not the token's key or the client instance's key.
  3. The AS returns a derived token to RS1 for use at RS2.
  4. RS1 calls RS2 with the token from (3).
  5. RS2 fulfills the call from RS1.
  6. RS1 fulfills the call from the original client instance.

If the RS needs to derive a token from one presented to it, it can request one from the AS by making a token request as described in [GNAP] and presenting the existing access token's value in the "existing_access_token" field.

Since the RS is acting as a client instance, the RS MUST identify itself with its own key in the client field and sign the request just as any client instance would, as described in Section 3.2. The AS MUST determine that the token being presented is appropriate for use at the RS making the token chaining request.

The AS responds with a token for the downstream RS2 as described in [GNAP]. The downstream RS2 could repeat this process as necessary for calling further RSs.
