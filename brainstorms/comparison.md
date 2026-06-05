
# Sketchpad for Access Request Design

## Unification

```yaml

# `@id`: An identifier for the message.
#
# OPTIONAL UNIQUE IRI<Message>
#
# Aliases:
# - Alt. key `uid` (in line with ODRL)
#
@id?: IRI<Message>   # canonical
uid?: IRI<Message>   # alias

# `@type`: The type of the message.
#
# RECOMMENDED IRI<MessageClass>
# - REQUIRED UNIQUE `dspace:ContractRequestMessage` for DataSpaces Negotiation
# - SHOULD probably NOT impact processing
#
# Aliases:
# - Alt. domain `string` (as IRI with generic AS domain if missing)
# - Alt. key `type` (optional JSON-LD '@')
#
@type?: IRI<MessageClass>   # canonical (DataSpaces)
type?: string               # alias

# `policy`: Details of the requested or offered usage or access policy terms.
#
# RECOMMENDED Policy
# - REQUIRED UNIQUE for DataSpaces (`offer`)
#
# Aliases:
# - Alt. key `offer` for DataSpaces
# - Alt. key `agreement` for DataSpaces
# - Alt. key `access_token` for GNAP
#
policy?: Policy         # canonical
offer?: Policy          # alias: DataSpaces
agreement?: Policy      # alias: DataSpaces
access_token?: Policy   # alias: GNAP

# `interact`: Methods to interact with the user (RP), supported by the user agent negotiation this request.
#
# RECOMMENDED InteractionDetails
# - REQUIRED for GNAP if interaction is supported at all
#
interact?: InteractionDetails   # canonical (GNAP)

[interact]

  # `start`: Method to start interaction with the user (RP).
  #
  # RECOMMENDED InteractionStart
  # - REQUIRED for GNAP (if interaction is supported at all)
  #
  # Aliases:
  # - Alt. domain IRI (shorthand)
  # - Alt. domain string for GNAP
  #
  start?: InteractionStart | IRI | string      # canonical (GNAP)

  # `finish`: Method to finish interaction with the user (RP).
  #
  # OPTIONAL InteractionFinish
  # - OPTIONAL UNIQUE for GNAP
  #
  # Aliases:
  # - Alt. domain string (in line with GNAP)
  #
  finish?: InteractionFinish | IRI | string    # canonical (GNAP)

  # `hints`: Suggested parameters (preferences) for user (RP) interaction.
  #
  # !! TODO: this seems like redundant (not semantically valuable) nesting.
  #
  # OPTIONAL UNIQUE Hints
  # - OPTIONAL UNIQUE for GNAP
  #
  # Examples:
  # - ui_locales?: array[string]
  #
  hints?: object  # canonical

[interact.start]

  # `method`: Method to start interaction with the user (RP).
  #
  # RECOMMENDED StartMethod
  # - REQUIRED UNIQUE for GNAP (redirect | app | user_code | user_code_uri)
  #
  # Aliases:
  # - Alt. domain IRI (shorthand)
  # - Alt. domain string for GNAP
  # - Alt. shorthand (interact) key `start` for GNAP
  #
  method?: StartMethod | IRI | string             # canonical (GNAP)
  [interact].start?: StartMethod | IRI | string   # alias (shorthand): GNAP

  # `[param]`: Any other parameter that details this interaction start method.
  #
  # OPTIONAL any
  #
  [...]?: any

[interact.finish]

  # `method`: Method to finish interaction with the user (RP).
  #
  # OPTIONAL FinishMethod
  # - OPTIONAL UNIQUE for GNAP (redirect | push)
  #
  # Aliases:
  # - Alt. domain IRI (shorthand)
  # - Alt. domain string (shorthand)
  # - Alt. shorthand (interact) key `finish` (in line with GNAP)
  #
  method?: FinishMethod | IRI | string             # canonical (GNAP)
  [interact].finish?: FinishMethod | IRI | string  # alias (shorthand)

  # `uri`: URI at the user agent at which to target the finish
  #
  # OPTIONAL URL
  # - REQUIRED UNIQUE for GNAP redirect or push methods
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. (root) key `redirect_uri` for OAuth 2.x
  # - Alt. (root) key `callbackAddress` for DataSpaces
  #
  uri?: string                      # canonical (GNAP)
  [root].redirect_uri?: string      # alias: OAuth 2.x (authz flow)
  [root].callbackAddress?: string   # alias: DataSpaces

  # `state`: An opaque message-specific handle.
  #
  # This parameter enables the recipient to bind the response (or callback) to the request, in order for the sender to assess the relevance of incoming messages (e.g., at its redirect or callback endpoint), and mitigate replay attacks, by matching the binding value to its known state. It MUST contain a different (pseudo-)random value for each message (request); values MUST NOT be reused, even in case of an error.
  #
  # RECOMMENDED UNIQUE string
  # - OPTIONAL for OAuth 2.x & UMA authorization requests (as `[root].state`)
  #
  # Aliases:
  # - Alt. (shorthand) key `[root].state` for OAuth 2.x & UMA
  #
  state?: string          # canonical
  [root].state?: string   # alias (shorthnd): OAuth 2.x & UMA

  # `nonce`: An opaque session-specific (pseudo-)random value.
  #
  # This parameter enables the server to bind the (final) response -- and/or its contents (e.g. tokens) -- to the interaction/flow, in order for the client to assess the relevance of the response and mitigate replay attacks, by matching the binding value to its known 'per-session' state. It MUST contain a different (pseudo-)random value for each session (negotiation flow); values MUST NOT be reused, even in case of an error.
  #
  # RECOMMENDED UNIQUE string
  # - REQUIRED for GNAP
  #
  # Aliases:
  # - Alt. (shorthand) key `[root].nonce`
  #
  nonce?: string          # canonical (GNAP)
  [root].nonce?: string   # alias (shorthand)

  # `challenge`: A hash derived from a secure (pseudo-)random seed -- a secret nonce.
  #
  # This parameter enables the sender to bind a (single) subsequent message to this one, in order for the receiver to verify their mutual origin (protecting the exchange), by matching its value to the hash of the secret disclosed in the subsequent message (cf. `challenge_method`). It MUST contain a different (pseudo-)random value for each session (negotiation flow); values MUST NOT be reused, even in case of an error.
  #
  # RECOMMENDED UNIQUE string
  # - REQUIRED for OAuth 2.x PKCE (as `[root].code_challenge`)
  #
  # Aliases:
  # - Alt. (shorthand) key `[root].challenge`
  # - Alt. key `[root].code_challenge` for Oauth 2.x PKCE
  #
  # challenge?: string              # canonical
  # [root].challenge?: string       # alias (shorthand)
  # [root].code_challenge?: string  # alias: Oauth 2.x PKCE

  # `challenge_method`: An identifier of the encoding (e.g., hashing algorithm) of the secret nonce in the `challenge` parameter.
  #
  # This parameter defaults to 'plain' for compatibility purposes, but this is INSECURE, and SHOULD NOT be used.
  #
  # RECOMMENDED UNIQUE IRI
  # - RECOMMENDED string for OAuth 2.x PKCE (as `[root].code_challenge_method`)
  #
  # Aliases:
  # - Alt. domain string for Oauth 2.x PKCE
  # - Alt. (shorthand) key `[root].challenge_method`
  # - Alt. key `[root].code_challenge_method` for Oauth 2.x PKCE
  #
  # challenge_method?: IRI | string         # canonical
  # [root].challenge_method?: IRI | string  # alias (shorthand)
  # [root].code_challenge_method?: string   # Oauth 2.x PKCE (plain | S256 | ...)

  # `[param]`: Any other parameter that details the interaction finish method.
  #
  # OPTIONAL any
  #
  [...]?: any

[policy]

  # `@id`: An identifier for the policy.
  #
  # RECOMMENDED UNIQUE IRI<Policy>
  # - REQUIRED for DataSpacesand ODRL (`uid`)
  # - REQUIRED for GNAP (`label`) if multiple policies
  #
  # Aliases:
  # - Alt. domain `string`
  # - Alt. key `uid` for ODRL
  #
  @id?: IRI<Policy>   # canonical (DataSpaces)
  uid?: IRI<Policy>   # alias: ODRL
  label?: string       # alias: GNAP

  # `@type`: The type of a policy. Defaults to `odlr:Policy`.
  #
  # RECOMMENDED IRI<PolicyClass>
  # - REQUIRED UNIQUE `dspace:Offer/Agreement` for DataSpaces
  # - SHOULD probably NOT impact processing
  #
  # Aliases:
  # - Alt. domain `string` (as IRI with generic AS domain if missing)
  # - Alt. key `type` (optional JSON-LD '@')
  #
  @type?: IRI<PolicyClass>  # canonical (DataSpaces)
  type?: string             # alias

  # `profile`: The (ODRL) profile of a policy.
  #
  # OPTIONAL IRI<Profile>
  #
  # Aliases:
  # - Global (root) key `profile`
  #
  profile?: IRI<Profile>          # canonical (ODRL)
  [root].profile?: IRI<Profile>   # alias (global)

  # `inheritFrom`: A reference to another policy, the rules of which are included in the current policy.
  #
  # While the status of the referenced policy SHOULD NOT be inherited, it can be taken into account in evaluating the current policy.
  #
  # OPTIONAL PolicyReference
  # - Cf. `inheritFrom.value` ...
  #
  # Aliases:
  # - Alt. key `grant`
  # - Alt. key `refresh`
  # - Alt. key `upgrade`
  # - Global (root) key `inheritFrom`
  # - Global (root) key `grant`
  # - Global (root) key `refresh`
  # - Global (root) key `upgrade`
  #
  inheritFrom?: PolicyReference                   # canonical (ODRL)
  grant?: PolicyReference.                        # alias
  refresh?: PolicyReference                       # alias
  upgrade?: PolicyReference                       # alias
  [root].inheritFrom?: PolicyReference | string   # alias (global)
  [root].grant?: PolicyReference | string         # alias (global)
  [root].refresh?: PolicyReference | string       # alias (global)
  [root].upgrade?: PolicyReference | string       # alias (global)

  # `rule`: A rule of a policy.
  #
  # RECOMMENDED Rule
  # - REQUIRED for ODRL
  #
  # Aliases:
  # - Alt. key `access` for GNAP
  # - Alt. key `permission` for ODRL & DS
  # - Alt. key `prohibition` for ODRL & DS
  # - Alt. key `obligation` for ODRL & DS
  # - Global (root) key `rule`
  # - Global (root) key `access`
  # - Global (root) key `permission`
  # - Global (root) key `prohibition`
  # - Global (root) key `obligation`
  #
  rule?: Rule                                # canonical
  permission?: Permission                    # alias: ODRL & DS
  obligation?: Duty                          # alias: ODRL & DS
  prohibition?: Prohibition                  # alias: ODRL & DS
  [root].rule?: Rule                         # alias (global)
  [root].authorization?: Permission          # alias (global)
  [root].permission?: Permission             # alias (global)
  [root].obligation?: Duty                   # alias (global)
  [root].prohibition?: Prohibition           # alias (global)

  # `conflict`: The (ODRL) conflict resolution strategy for the policy.
  #
  # OPTIONAL UNIQUE ConflictTerm
  #
  # Aliases:
  # - Global (root) key `conflict`
  #
  conflict?: ConflictTerm         # canonical (ODRL)
  [root].conflict?: ConflictTerm  # alias (global)

  # `record`: A description of the requested way to record the policy decision.
  #
  # This parameter defaults to `token`.
  #
  # OPTIONAL RecordFormat
  # - REQUIRED UNIQUE string for OAuth authz requ (as `[root].response_type`)
  #
  # Aliases:
  # - Alt. domain string for OAuth 2.x
  # - Global key `[root].record`
  # - Global key `[root].response_type` for OAuth 2.x
  #
  record?: RecordFormat | string          # canonical
  [root].record?: RecordFormat | string   # alias (global)
  [root].response_type: string            # alias (global): OAuth 2.x

  # `grant`: A proof of previous agreement.
  #
  # OPTIONAL Grant
  #
  # Aliases:
  # - Alt. domain string
  # - Global key `[root].grant`
  #
  grant?: Grant | string          # canonical
  [root].grant?: Grant | string   # alias (global)

  # `timestamp`: The timestamp of an agreed-upon policy ('agreement').
  #
  # OPTIONAL UNIQUE xsd:DateTime
  # - REQUIRED in DataSpaces agreement
  #
  # Aliases:
  # - Global (root) key `timestamp`
  #
  timestamp?: string          # canonical (DataSpaces)
  [root].timestamp?: string   # alias (global)

[policy.grant]

  # `format`: The format of the policy reference.
  #
  # Possible values are: client_credentials, refresh_token, code (ticket?)
  #
  # RECOMMENDED UNIQUE IRI
  # - REQUIRED string for OAuth 2.x & UMA (as `[root].grant_type`)
  #
  # Aliases:
  # - Alt. domain `string`
  # - Global (root) key `grant_type` for OAuth 2.x & UMA
  #
  # # @type: IRI
  # # type: IRI | string
  format: IRI | string        # canonincal
  [root].grant_type: string   # alias (global): OAuth 2.x & UMA

  # `value`: The value of the policy reference.
  #
  # OPTIONAL UNIQUE any
  # - REQUIRED string for UMA (as `[root].rpt`) (or `[root].ticket` ?)
  # - REQUIRED string for OAuth 2.x (as `[root].code/refresh_token`)
  #
  # Aliases:
  # - Alt. domain `string` for GNAP
  #
  # # token: string
  value: string | any
  [root].rpt?: string                             # alias (global): UMA
  [root].refresh_token?: string                   # alias (global): OAuth 2.x
  [root].code?: string                            # alias (global): OAuth 2.x

  # `...`: Additional (meta)parameters about the referenced policy.
  #
  # OPTIONAL any
  #
  ...: any

[policy.record]

  # `flags`: A description of the requested way to record the policy decision.
  #
  # OPTIONAL string
  # - OPTIONAL for GNAP (as `[policy].flags`; e.g., bearer)
  #
  # Aliases:
  # - Shorthand (policy) key `flags` for GNAP
  #
  flags?: string             # canonical
  [policy].flags?: string    # alias (shorthand): GNAP

[policy.rule]

  # `@id`: An identifier for the rule.
  #
  # OPTIONAL UNIQUE IRI<Rule>
  #
  # Aliases:
  # - Alt. key `id` (optional JSON-LD '@')
  # - Alt. key `uid` (in line with ODRL)
  #
  @id?: IRI<Rule>   # canonical
  id?: IRI<Rule>    # alias
  uid?: IRI<Rule>   # alias

  # `@type`: The type of a rule. Defaults to `odlr:Permission`.
  #
  # RECOMMENDED IRI<RuleClass>
  #
  # Aliases:
  # - Alt. domain `string` (as IRI with generic AS domain if missing)
  # - Alt. key `type` (optional JSON-LD '@')
  #
  @type?: IRI<PolicyClass>  # canonical
  type?: string             # alias

  # `assigner`: The party assigning the policy (to the assignee).
  #
  # OPTIONAL UNIQUE Party
  # - REQUIRED UNIQUE for ODRL offers (as `[root/policy].assigner`)
  # - REQUIRED UNIQUE string for DataSpaces (as `[root].providerPid`)
  # - REQUIRED UNIQUE string for DataSpaces agreement (as `[root].assigner`)
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. key `controller`
  # - Alt. key `provider`
  # - Alt. key `providerPid` (in line with DataSpaces)
  # - Shared (policy) key `assigner` for ODRL
  # - Shared (policy) key `controller`
  # - Shared (policy) key `provider`
  # - Shared (policy) key `providerPid` (in line with DataSpaces)
  # - Global (root) key `assigner` for ODRL & DataSpaces
  # - Global (root) key `controller`
  # - Global (root) key `provider`
  # - Global (root) key `providerPid` for DataSpaces
  #
  assigner?: string | Party             # canonical (ODRL)
  controller?: string | Party           # alias
  provider?: string | Party             # alias
  providerPid?: string                  # alias
  [policy].assigner?: string | Party    # alias (shared): ODRL
  [policy].controller?: string | Party  # alias (shared)
  [policy].provider?: string | Party    # alias (shared)
  [policy].providerPid?: string         # alias (shared)
  [root].assigner?: string | Party      # alias (global): ODRL & DataSpaces
  [root].controller?: string | Party    # alias (global)
  [root].provider?: string | Party      # alias (global)
  [root].providerPid?: string           # alias (global): DataSpaces

  # `assignee`: The party to which the policy is assigned/applicable.
  #
  # RECOMMENDED UNIQUE Party
  # - REQUIRED UNIQUE for ODRL offers/agreements (as `[root/policy].assignee`)
  # - REQUIRED UNIQUE string for DataSpaces (as `[root].consumerPid`)
  # - REQUIRED UNIQUE string for DataSpaces agreement (as `[root].assignee`)
  # - OPTIONAL UNIQUE for GNAP (as `[root].user`)
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. key `consumer`
  # - Alt. key `user` (in line with GNAP)
  # - Shared (policy) key `assignee` for ODRL
  # - Shared (policy) key `consumer`
  # - Shared (policy) key `user` (in line with GNAP)
  # - Global (root) key `assignee` for ODRL & DataSpaces
  # - Global (root) key `consumer`
  # - Global (root) key `user` for GNAP
  #
  assignee?: string | Party             # canonical (ODRL)
  consumer?: string | Party             # alias
  user?: string | Party                 # alias
  [policy].assignee?: string | Party    # alias (shared): ODRL
  [policy].consumer?: string | Party    # alias (shared)
  [policy].user?: string | Party        # alias (shared)
  [root].assignee?: string | Party      # alias (global): ODRL & DataSpaces
  [root].consumer?: string | Party      # alias (global)
  [root].user?: string | Party          # alias (global): GNAP

  # `operation`: The content/details of the operation to which this rule applies.
  #
  # RECOMMENDED UNIQUE OperationDetails (or `RuleContent` ?)
  # - REQUIRED for GNAP (as `[root].access_token.access`)
  # - REQUIRED for OAuth 2.x RAR (as `[root].authorization_details`)
  # - REQUIRED string for UMA (as `[root].ticket`)
  #
  # Aliases:
  # - Alt. domain `string` for UMA
  # - Alt. key `content`
  # - Alt. key `details`
  # - Shared (policy) key `access` for GNAP
  # - Global (root) key `authorization_details` for OAuth 2.x RAR
  # - Global (root) key `ticket` for UMA
  #
  operation?: OperationDetails                     # canonical
  content?: OperationDetails                       # alias
  details?: OperationDetails                       # alias
  [policy].access?: OperationDetails               # alias (shared): GNAP
  [root].authorization_details?: OperationDetails  # alias (global): OAuth RAR
  [root].ticket?: OperationDetails | string        # alias (global): UMA

[policy.rule.assigner]

  # ... only in aggreement?

[policy.rule.assignee]

  # `credential`: An identifier or (formatted) assertion about the assignee.
  #
  # RECOMMENDED Assertion
  # - OPTIONAL for GNAP (as `[root].user.assertions/sub_ids`)
  #
  # Assertions under `assertion/s` or `sub_id/s` need to adhere to rfc9635.
  # String values are interpreted as the `value` of an Assertion with the `uri` format, unless a more specific format can unambiguously be derived from the structure of the string or the key used to link them:
  # - The keys `pid` & `consumerPid` indicate the format `dspace:consumerPid`.
  #
  # Aliases:
  # - Alt. domain `string` for ODRL & DataSpaces
  # - Alt. keys `id` & `identifier`
  # - Alt. keys `assertions` & `assertions` for GNAP
  # - Alt. keys `sub_ids` & `sub_ids` for GNAP
  # - Alt. keys `pid` & `consumerPid` (in line with DataSpaces)
  # - Shared (policy) key `consumerPid` (in line with DataSpaces)
  # - Global (root) key `consumerPid` for DataSpaces
  #
  credential: Assertion             # canonical
  assertion/s: Assertion            # alias: GNAP
  sub_id/s: Assertion               # alias: GNAP
  id/entifier?: Assertion | string  # alias
  pid?: string                      # alias
  consumerPid?: string              # alias
  [policy].consumerPid?: string     # alias (shared)
  [root].consumerPid?: string       # alias (global): DataSpaces

  # claim_set?: graph
  # pct?: string # UMA claim set ref

[policy.rule.assignee.credential]

  # `format`: The format of the identifier or assertion.
  #
  # RECOMMENDED UNIQUE IRI
  # - REQUIRED for GNAP
  #
  # Aliases:
  # - Alt. domain `string` for GNAP
  #
  format: IRI | string

  # `value`: The value of the identifier or assertion.
  #
  # OPTIONAL UNIQUE any
  # - REQUIRED string for GNAP
  #
  value: string | any

  # `...`: Additional (meta)parameters about the credential.
  #
  # OPTIONAL any
  #
  ...: any

[policy.rule.operation]

  # `type`: The type of OperationDetails, which determines the format of this object, i.e., how its parameters are to be interpreted.
  #
  # RECOMMENDED IRI<OperationFormat>
  # - REQUIRED string for GNAP (`access_token.access.type`)
  # - REQUIRED string for OAuth 2.x RAR (`authorization_details.type`)
  #
  # Aliases:
  # - Alt. domain `string` (as IRI with generic AS domain if missing)
  # - Alt. key `type` (optional JSON-LD '@')
  #
  @type?: IRI<OperationFormat>  # canonical
  type?: string             # alias: GNAP & OAuth 2.x RAR
  format?: string           # alias

  # `action`: A definition of the action(s) that this operation performs.
  #
  # RECOMMENDED Action
  # - REQUIRED UNIQUE for OAuth 2.x RI (as `[root].resource`)
  # - REQUIRED UNIQUE for ODRL permissions/prohibitions (on [rule] or [policy])
  # - OPTIONAL UNIQUE for ODRL duties (on [rule] or [policy])
  # - OPTIONAL UNIQUE for DataSpaces (on [offer]; or derived from dataset)
  # - OPTIONAL UNIQUE for OAuth 2.x RAR (as `[authz_details].identifier`)
  # - OPTIONAL UNIQUE for GNAP (as `[access_token.access].actions`)
  # - IMPLICITLY REQUIRED in UMA ticket
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. key `actions` for GNAP & OAuth 2.x RAR
  # - Alt. key `privileges` for GNAP & OAuth 2.x RAR
  # - Alt. key `scope` (in line with OAuth 2.x & UMA)
  # - Shared (rule) key `action` for ODRL
  # - Shared (rule) key `actions` (in line with GNAP & OAuth 2.x RAR)
  # - Shared (rule) key `privileges` (in line with GNAP & OAuth 2.x RAR)
  # - Shared (rule) key `scope` (in line with ODRL)
  # - Shared (policy) key `action` for ODRL
  # - Shared (policy) key `actions` (in line with GNAP & OAuth 2.x RAR)
  # - Shared (policy) key `privileges` (in line with GNAP & OAuth 2.x RAR)
  # - Shared (policy) key `scope` (in line with ODRL)
  # - Global (root) key `action` (in line with OAuth 2.x & UMA)
  # - Global (root) key `actions` (in line with GNAP & OAuth 2.x RAR)
  # - Global (root) key `privileges` (in line with GNAP & OAuth 2.x RAR)
  # - Global (root) key `scope` for OAuth 2.x & UMA
  #
  action/s?: Action | string              # canonical (GNAP & OAuth 2.x RAR)
  scope?: string                          # alias
  [policy.rule].action?: Action | string  # alias (shared): ODRL
  [policy.rule].scope?: string            # alias (shared)
  [policy].action?: Action | string       # alias (shared): ODRL
  [policy].scope?: string                 # alias (shared)
  [root].action?: Action | string         # alias (global)
  [root].scope?: string                   # alias (global): OAuth 2.x & UMA

  # `target`: A definition of the assets targetted by the operation.
  #
  # RECOMMENDED Asset
  # - REQUIRED UNIQUE for ODRL permissions/prohibitions (on [rule] or [policy])
  # - OPTIONAL UNIQUE for ODRL duties (on [rule] or [policy])
  # - OPTIONAL UNIQUE for DataSpaces (on [offer]; or derived from dataset)
  #
  # Aliases:
  # - Alt. domain `AssetCollection` for ODRL & DataSpaces
  # - Alt. key `resource`
  # - Shared (policy) key `target` for ODRL & DataSpaces
  # - Shared (policy) key `resource`
  # - Global (root) key `target`
  # - Global (root) key `resource`
  #
  target?: Asset(Collection)                     # canonical
  resource?: Asset(Col.) | string                # alias
  [policy.rule].target?: Asset(Collection)       # alias (shared): ODRL & DS
  [policy.rule].resource?: Asset(Col.) | string  # alias (shared): ODRL & DS
  [policy].target?: Asset(Collection)            # alias (shared): ODRL & DS
  [policy].resource?: Asset(Col.) | string       # alias (shared): ODRL & DS
  [root].target?: Asset(Collection)              # alias (global)
  [root].resource?: Asset(Col.) | string         # alias (global)

  # `agent`: The software instances that will execute the action of this rule on behalf of the assignee, including the client application making the initial request.
  #
  # RECOMMENDED Agent
  # - REQUIRED UNIQUE for GNAP (as `[root].client`)
  #
  # Aliases:
  # - Alt. domain `string` for GNAP (& OAuth 2.0 & UMA)
  # - Alt. key `agent` (in line with GNAP)
  # - Shared (rule) key `client` (in line with GNAP)
  # - Shared (rule) key `agent`
  # - Shared (policy) key `client` (in line with GNAP)
  # - Shared (policy) key `agent`
  # - Global (root) key `client` for GNAP
  # - Global (root) key `agent`
  #
  agent?: Agent                           # canonical
  client?: Agent | string                 # alias
  [policy.rule].agent?: Agent             # alias (shared)
  [policy.rule].client?: Agent | string   # alias (shared)
  [policy].agent?: Agent                  # alias (shared)
  [policy].client?: Agent | string        # alias (shared)
  [root].agent?: Agent                    # alias (global)
  [root].client?: Agent | string          # alias (global): GNAP

  # `[param]`: Any other parameter that details this operation.
  #
  # OPTIONAL any
  #
  # Examples:
  # - privileges?: string   # GNAP & OAuth 2.x RAR
  # - locations?: string    # GNAP & OAuth 2.x RAR
  # - datatypes?: string    # GNAP & OAuth 2.x RAR
  [...]?: any

[policy.rule.operation.target]

  # `@id`: An identifier of the asset targetted by the operation.
  #
  # RECOMMENDED IRI
  # - REQUIRED UNIQUE for OAuth 2.x RI (as `[root].resource`)
  # - RECOMMENDED UNIQUE for ODRL
  # - OPTIONAL UNIQUE for DataSpaces (on [offer]; or derived from dataset)
  # - OPTIONAL UNIQUE for GNAP/RAR (as `[authorization_details].identifier`)
  # - IMPLICITLY REQUIRED in UMA ticket
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. key `uid` for ODRL & DataSpaces
  # - Alt. key `identifier` (in line with GNAP & OAuth 2.x RAR)
  # - Shorthand key `target` for ODRL & DataSpaces
  # - Shorthand key `resource` for OAuth 2.x RI
  # - Shorthand key `identifier` for GNAP & OAuth 2.x RAR
  # - Shared (policy) key `target` for ODRL & DataSpaces
  # - Shared (policy) key `resource` for OAuth 2.x RI
  # - Shared (policy) key `identifier` for GNAP & OAuth 2.x RAR
  # - Global (root) key `target` for ODRL
  # - Global (root) key `resource` for OAuth 2.x RI
  #
  @id?: string                      # canonical
  uid?: string                      # alais: ODRL & DataSpaces
  [policy.rule.operation].identifier?: string  # alias: GNAP & OAuth 2.x RAR
  [policy.rule].target?: string     # alias (shorthand): ODRL & DataSpaces
  [policy.rule].resource?: string   # alias (shorthand)
  [policy].target?: string          # alias (shared): ODRL & DataSpaces
  [policy].resource?: string        # alias (shared)
  [root].target?: string            # alias (global)
  [root].resource?: string          # alias (global): OAuth 2.x RI

  # `partOf`: The AssetCollection this asset is part of.
  #
  # OPTIONAL AssetCollection
  # - OPTIONAL for ODRL & DataSpaces
  #
  partOf?: AssetCollection          # canonical (ODRL & DataSpaces)

  # `source`: The source AssetCollection of which this collection is a refinement.
  #
  # OPTIONAL (Logical)Constraint
  # - REQUIRED for ODRL & DataSpaces AssetCollections with a `refinement`
  #
  source?: AssetCollection          # canonical (ODRL & DataSpaces)

  # # GNAP ID Token request
  # subject?: # GNAP (subject data, directly from "AS" issuer)
  # subject.assertion_formats: string[] # response encoding: idtoken/saml2
  # subject.sub_id_formats: string[] # requested rfc9635 formats
  # subject.sub_ids[]?: # known rfc9635 subject info (format-based)
  # subject.sub_ids[].format: string
  # subject.sub_ids[].\[param]: string
  # subject.updated_at?: string

  # `refinement`: A refinement of another AssetCollection (`source`), applied as a filter to identify which individual Asset(s) are members of this derived collection.
  #
  # OPTIONAL (Logical)Constraint
  # - OPTIONAL for ODRL & DataSpaces
  #
  refinement?: (Logical)Constraint  # canonical (ODRL & DataSpaces)

[policy.rule.operation.target.refinement]

  - operator?: Operator
  - leftOperand?: LeftOperand
  - rightOperand?: RightOperand
  - and|andSequence|or|xone?: Constraint

[policy.rule.operation.action]

  # `@id`: An identifier of the action performed by this operation.
  #
  # RECOMMENDED IRI<ActionDetails>
  # - REQUIRED UNIQUE for OAuth 2.x (as `[root].scope`)
  # - REQUIRED UNIQUE for ODRL (as `[policy(.rule)].action`)
  # - REQUIRED UNIQUE for DataSpaces (as `[offer].action`)
  # - OPTIONAL for OAuth 2.x RAR (as `[authorization_details].actions`)
  # - OPTIONAL for GNAP (as `[access_token.access].actions`)
  # - IMPLICITLY REQUIRED in UMA ticket
  #
  # Aliases:
  # - Alt. domain `string` for all
  # - Alt. key `uid` for ODRL & DataSpaces
  # - Shorthand (operation) key `actions` for GNAP & OAuth 2.x RAR
  # - Shorthand (rule) key `action` for ODRL & DataSpaces
  # - Shorthand (rule) key `scope`
  # - Shared (policy) key `action` for ODRL & DataSpaces
  # - Shared (policy) key `scope`
  # - Global (root) key `action`
  # - Global (root) key `scope` for OAuth 2.x
  #
  @id?: string                      # canonical
  uid?: string                      # alais
  [policy.rule.operation].actions?: string  # alias (shorthand): GNAP &  OAuth 2.x RAR
  [policy.rule].action?: string     # alias (shorthand): ODRL & DataSpaces
  [policy.rule].scope?: string      # alias (shorthand)
  [policy].action?: string          # alias (shared): ODRL & DataSpaces
  [policy].scope?: string           # alias (shared)
  [root].action?: string            # alias (global)
  [root].scope?: string             # alias (global): OAuth 2.x RI

  # `rdf:value`: The Action of which of which this action is a refinement.
  #
  # OPTIONAL (Logical)Constraint
  # - REQUIRED UNIQUE for ODRL & DataSpaces Actions with a `refinement`
  #
  rdf:value?: Action                # canonical (ODRL & DataSpaces)

  # `refinement`: A refinement of the extent of another Action (`rdf:value`), resulting in the semantics of this derived Action.
  #
  # OPTIONAL (Logical)Constraint
  # - OPTIONAL for ODRL & DataSpaces
  #
  refinement?: (Logical)Constraint  # canonical (ODRL & DataSpaces)

[policy.rule.operation.action.refinement]

  - operator?: Operator
  - leftOperand?: LeftOperand
  - rightOperand?: RightOperand
  - and|andSequence|or|xone?: Constraint

[policy.rule.operation.agent]

  # `@id`: An identifier of the agent that will perform this operation.
  #
  # RECOMMENDED IRI<Agent>
  # - REQUIRED UNIQUE for OAuth 2.x & UMA (as `[root].client_id`)
  # - REQUIRED UNIQUE for GNAP (as `[root.client].id`) ?
  #
  # Aliases:
  # - Alt. domain `string` OAuth 2.x & UMA & GNAP
  # - Alt. key `id` (without '@')
  # - Alt. key `uri`
  # - Alt. key `display.uri` for GNAP
  # - Global (root) key `client_id` for OAuth 2.x & UMA
  # - Global (root) key `client_uri` for OAuth 2.x & Client Metadata
  @id: IRI                    # canonical
  id: IRI | string            # alias
  uri?: string                # alias
  display?.uri?: string       # alias: GNAP
  identifier: string          # alias
  [root].client_id: string    # alias (global): OAuth 2.x & UMA
  [root].client_uri?: string  # alias (global): OAuth 2.x Client Metadata

  # `...`: ... of the agent that will perform this operation.
  #
  # OPTIONAL IRI<Agent>
  # - REQUIRED UNIQUE for OAuth/UMA client credentials (as `[root].client_id`)
  #
  # Aliases:
  # - Alt. domain `string` OAuth 2.x & UMA & GNAP
  # - Global (root) key `client_secret` for OAuth 2.x & UMA
  ...: ...                        # canonical
  secret?: string                 # alias
  [root].client_secret?: string   # alias: OAuth 2.x & UMA

  # `key`: A public key of the agent that will perform this operation.
  #
  # RECOMMENDED Key
  # - REQUIRED UNIQUE for OAuth 2.x & UMA (as `[root].jwks(_uri)`)
  # - REQUIRED UNIQUE for GNAP (as `[root.client].key`) ?
  #
  # Aliases:
  # - Alt. domain `string` OAuth 2.x & UMA & GNAP
  # - Global (root) key `client_id` for OAuth 2.x
  key?: Key | IRI         # canonical (GNAP)
  [root].jwks?: JWKSet    # alias: OAuth 2.x & UMA
  [root].jwks_uri?: IRI   # alias: OAuth 2.x & UMA

  # `display`: A non-canonical nested object grouping human-facing info.
  #
  # NOT RECOMMENDED object
  # - RECOMMENDED UNIQUE for GNAP
  #
  display?: object            # non-canonical (GNAP)

  # `[param]`: More info about the agent that will perform this operation.
  #
  # OPTIONAL any
  #
  # Examples:
  #
  # A human-readable name for the agent
  #  name?: string               # canonical
  #  client_name?: string        # alias: OAuth 2.x Client Metadata
  #  display?.name?: string      # alias: GNAP
  #
  # A logo to display for the agent
  #  logo?: string               # canonical
  #  logo_uri?: string           # alias: OAuth 2.x Client Metadata
  #  display?.logo?: string      # alias
  #  display?.logo_uri?: string  # alias: GNAP
  #
  # A policy document ...
  #  policy?: uri                # canonical
  #  policy_uri?: uri            # alias: OAuth 2.x Client Metadata
  #
  # A terms of service document
  #  tos?: uri                   # canonical
  #  tos_uri?: uri               # alias: OAuth 2.x Client Metadata
  #
  # contacts?: string[]         # canonical (OAuth 2.x Client Metadata)


#   # `credential`: An identifier or (formatted) assertion about the client.
#   #
#   # RECOMMENDED Assertion
#   #
#   # Aliases:
#   # - Alt. domain `string`
#   # - Alt. keys `id` & `identifier`
#   # - Alt. keys `assertion`
#   #
#   credential: Assertion             # canonical
#   assertion: Assertion              # alias

# [policy.rule.client.credential]

#   # `format`: The format of the identifier or assertion.
#   #
#   # RECOMMENDED UNIQUE IRI
#   #
#   format: IRI | string

#   # `value`: The value of the identifier or assertion.
#   #
#   # OPTIONAL UNIQUE any
#   #
#   value: string | any

#   # `...`: Additional (meta)parameters about the credential.
#   #
#   # OPTIONAL any
#   #
#   ...: any


  class?: AgentClass
  software?: AgentClass

[policy.rule.operation.agent.class]

  # format?: string
  # value?: string
  # [param]?: any

  #@id?: uri

  id?: string
  [policy.rule.agent].class_id?: string # client software(+version) indication
  [policy.rule.agent].software_id?: string # OAUTH client info

  version?: string
  [policy.rule.agent].software_version?: string # OAUTH client info

[policy.rule]

  # `condition`: A rule that must hold for this rule to hold.
  #
  # OPTIONAL IRI<Rule>
  #
  # Aliases:
  # - Alt. key `duty` for ODRL
  # - Alt. key `if`
  # - Shared (policy) key `condition`
  # - Shared (policy) key `duty`
  # - Shared (policy) key `if`
  # - Global (root) key `condition`
  # - Global (root) key `duty`
  # - Global (root) key `if`
  #
  condition?: Rule           # canonical
  duty?: Duty                # alias: ODRL
  if?: Rule                  # alias
  [policy].condition?: Rule  # alias (shared)
  [policy].duty?: Duty       # alias (shared)
  [policy].if?: Rule         # alias (shared)
  [root].condition?: Rule    # alias (global)
  [root].duty?: Duty         # alias (global)
  [root].if?: Rule           # alias (global)

  # `resolution`: A rule that holds when this rule is not met.
  #
  # OPTIONAL IRI<Rule>
  #
  # Aliases:
  # - Alt. key `failure` for ODRL
  # - Alt. key `consequence` for ODRL
  # - Alt. key `remedy` for ODRL
  # - Alt. key `else`
  # - Shared (policy) key `resolution`
  # - Shared (policy) key `failure`
  # - Shared (policy) key `consequence`
  # - Shared (policy) key `remedy`
  # - Shared (policy) key `else`
  # - Global (root) key `resolution`
  # - Global (root) key `failure`
  # - Global (root) key `consequence`
  # - Global (root) key `remedy`
  # - Global (root) key `else`
  #
  resolution?: Rule            # canonical
  failure?: Duty               # alias: ODRL
  remedy?: Duty                # alias: ODRL
  consequence?: Duty           # alias: ODRL
  else?: Rule                  # alias
  [policy].resolution?: Rule   # alias
  [policy].failure?: Duty      # alias
  [policy].remedy?: Duty       # alias
  [policy].consequence?: Duty  # alias
  [policy].else?: Rule         # alias
  [root].resolution?: Rule     # alias
  [root].failure?: Duty        # alias
  [root].remedy?: Duty         # alias
  [root].consequence?: Duty    # alias
  [root].else?: Rule           # alias

  # `constraint`: A constraint/condition on (the applicability of) this rule.
  #
  # OPTIONAL (Logical)Constraint
  # - OPTIONAL for ODRL & DataSpaces
  #
  constraint?: (Logical)Constraint  # canonical (ODRL & DataSpaces)

[policy.rule.constraint]

  - operator?: Operator
  - leftOperand?: LeftOperand
  - rightOperand?: RightOperand
  - and|andSequence|or|xone?: Constraint

```

## Grouped paths

```yaml
@type?: (ContractRequest)Message # DS ; not type in other specs ?

access_token*[]: # GNAP
access_token*[].flags: string[] # GNAP e.g. token type ('bearer')
access_token*[].label: string   # GNAP for multiple tokens

\[req:access_token.access|authorization_details|offer]*[]: # DS (max 1)

@id?: uri # sugar ?? (but clashes with request/message ID ?)
uid?: uri # sugar ?? (but clashes with request/message ID ?)
\[req]*[].@id?: uri # DS
\[req]*[].uid?: uri # ODRL alias

@type?:  uri # sugar ?? (but clashes with request/message @type)
type: string # sugar ??
profile?: *[uri] # sugar ??
grant_type: string # OAUTH (client_cred., refresh_token, code) & UMA (ticket)
\[req]*[].@type?:  uri # odlr:Policy / ds:Offer subclass (~ ds:Agreement)
\[req]*[].type: string # alias GNAP & OAUTH (RAR) (max 1 ~ more "profile")
\[req]*[].profile?: *[uri] # "alias" ODRL

\[req]*[].inheritFrom?: IRI[] # ODRL

\[req]*[].conflict?: ConflictTerm # ODRL

\[req]*[].timestamp?: string # (only in DS Agreement)

# sugar: top-level permission ??
\[req]*[]: Rule[] # sugar ?? (but how to distinguish top-level offer vs. rule ?)
\[rule:permission|prohibition|obligation]*[]?: Rule[] # sugar ?
\[req]*[].\[rule:permission|prohibition|obligation]*[]?: Rule[] # ODRL & DS

uid?: uri # sugar ?? (always permission)
\[req]*[].uid?: uri # sugar ?? (but clashes with req uid ?!)
\[rule]*[].uid?: uri # sugar ? (but clashes with req uid ?!)
\[req]*[].\[rule]*[].uid?: uri

\[else:consequence|remedy|duty]*[]?: Duty[] # sugar ?? (only Duty if top = perm)
\[rule]*[].\[else:consequence|remedy|duty]*[]?: Duty[] # for Duty/Proh/Perm
\[rule]*[].\[else:consequence|remedy|duty]*[]?: Duty[] # for Duty/Proh/Perm
\[req]*[].\[rule]*[].\[else:consequence|remedy|duty]*[]?: Duty[] # idem

resource?: string # OAUTH (RI): target or its location (server/provider)
target?: string | Asset # sugar ?
\[req]*[].target?: string | Asset # DS Dataset ID (+ ODRL sugar)
\[req]*[].\[rule]*[].target?: string | Asset(Col) # ODRL uid or (refined) object
\[req]*[].identifier?: string # GNAP & OAUTH (RAR): local id at RS (ex. account)
\[req]*[].locations?: string[] # GNAP & OAUTH (RAR): target or its location
\[req]*[].datatypes?: string[] # GNAP & OAUTH (RAR): shape (images, metadata)

scope?: string # OAUTH & UMA
action?: string | Action # sugar ??
actions?: string[] # sugar ??
\[req]*[].action?: string | Action # ODRL sugar
\[req]*[].actions?: string[] # GNAP & OAUTH (RAR): mode (read, write)
\[req]*[].\[rule]*[].action?: string | Action # ODRL uid or (refined) object
\[req]*[].privileges?: string[] # GNAP & OAUTH (RAR): scope/role (admin, user)

subject?: # GNAP (subject data, directly from "AS" issuer)
subject.assertion_formats: string[] # response encoding: idtoken/saml2
subject.sub_id_formats: string[] # requested rfc9635 formats
subject.sub_ids[]?: # known rfc9635 subject info (format-based)
subject.sub_ids[].format: string
subject.sub_ids[].\[param]: string

\[req]*[].\[rule]*[].constraint*[]?: # ODRL/DS (Logical)Constraint
\[req]*[].\[rule]*[].constraint*[].operator: Operator
\[req]*[].\[rule]*[].constraint*[].leftOperand: LeftOperand
\[req]*[].\[rule]*[].constraint*[].rightOperand: RightOperand
\[req]*[].\[rule]*[].constraint*[].[logic:and|andSequence|or|xone]?: Constraint[]

providerPid?: string # DS provider msgs/events/agreements/... (= RO ~ assigner)
assigner?: string | Party[] # sugar ? (RO/RC != provider)
\[req]*[].assigner?: string[] | Party(Col)[] # DS Agreement 1 PID (+ ODRL sugar)
\[req]*[].\[rule]*[].assigner?: string[] | Party(Col)[] # ODRL uid or refinement

consumerPid: string # DS consumer msgs/events/agreements/... (= RP)
user?: string | ... # GNAP 'end user' (held to be RO, but should be RP)
user.sub_ids[]?: \[] # user ids (rfc9635 format)
user.sub_ids[].format: string
user.sub_ids[].\[param]: any
user.assertions[]?: \[ # user claims
user.assertions[].format: strings # encoding ('id_token', 'saml2')
user.assertions[].value: string
user.updated_at?: string
assignee?: string | Party[] # sugar ? (RP != client/'agent')
\[req]*[].assignee?: string[] | Party(Col)[] # DS Agreement 1 PID (+ ODRL sugar)
\[req]*[].\[rule]*[].assignee?: string[] | Party(Col)[] # ODRL uid or refinement

# grant_type: string # OAUTH (client_cred., refresh_token, code) & UMA (ticket)

refresh_token: string # OAUTH & UMA
rpt?: string # UMA token to upgrade

ticket?: string # UMA: grant + resource/target + action/scope
code?: string # OAUTH (code flow)
pct?: string # UMA claim set ref

jwks?: JWKSet # OAUTH client instance's keys
jwks_uri?: uri # OAUTH alt ref to keys
client.key: string | object # client instance's key
client_secret?: string # OAUTH alt to keys

client_id: string # OAUTH & UMA alias ...
client: string | object # GNAP (APP != RP/assignee)

client.display?: # human-readable client representation

client_name?: string # OAUTH client info
client.display.name?: string

software_id?: string # OAUTH client info
software_version?: string # OAUTH client info
client.class_id?: string # client software(+version) indication

client_uri?: uri # OAUTH client info
client.display.uri?: uri

logo_uri?: uri # OAUTH client info
client.display.logo_uri?: uri

policy_uri?: uri # OAUTH client info
tos_uri?: uri # OAUTH client info
contacts?: string[] # OAUTH client info

interact: # GNAP interaction methods (supported by client)

interact.hints?: # optional info for interaction
interact.hints.ui_locales?: array[string]

interact.finish.nonce: string
interact.finish.hash_method?: string

interact.finish?: # how to return feedback from interaction
interact.finish.method: string # redirect | push (POST ~ DS callback)
interact.finish.uri: string # ~ redirect_uri or callbackAddress
callbackAddress?: uri # DS alias (~~~ redirect_uri, but for SERVER)
redirect_uri: uri # OAUTH alias (~~~ callbackAddress, but for CLIENT authz req)
response_type: string # OAUTH (authz req.): code | [extensions]

interact.start[]: string[] | object[] # how to start interaction
interact.start[].mode: string # redirect | app | user_code | user_code_uri
interact.start[].\[param]: any
code_challenge_method?: string # OAUTH (authz req.): plain | S256 | [extensions]
code_challenge?: string # OAUTH (authz req.)
state?: string # OAUTH & UMA (opaque) client state (authz req)
```

## Paths

```yaml
@type?: (ContractRequest)Message # DS ; not type in other specs ?
access_token*[]: # GNAP
access_token*[].flags: string[] # GNAP e.g. token type ('bearer')
access_token*[].label: string   # GNAP for multiple tokens
\[req:access_token.access|authorization_details|offer]*[]: # DS (max 1)
\[req]*[].@id?: uri # DS
\[req]*[].uid?: uri # ODRL alias
\[req]*[].@type?:  uri # odlr:Policy / ds:Offer subclass (~ ds:Agreement)
\[req]*[].type: string # alias GNAP & OAUTH (RAR) (max 1 ~ more "profile")
\[req]*[].profile?: *[uri] # "alias" ODRL
\[req]*[].inheritFrom?: IRI[] # ODRL
\[req]*[].conflict?: ConflictTerm # ODRL
\[req]*[].\[rule:permission|prohibition|obligation]*[]?: Rule[] # ODRL & DS
\[req]*[].\[rule]*[].uid?: uri
\[req]*[].\[rule]*[].target?: string | Asset(Col) # ODRL uid or (refined) object
\[req]*[].\[rule]*[].action?: string | Action # ODRL uid or (refined) object
\[req]*[].\[rule]*[].assigner?: string[] | Party(Col)[] # ODRL uid or refinement
\[req]*[].\[rule]*[].assignee?: string[] | Party(Col)[] # ODRL uid or refinement
\[req]*[].\[rule]*[].constraint*[]?: # ODRL/DS (Logical)Constraint
\[req]*[].\[rule]*[].constraint*[].operator: Operator
\[req]*[].\[rule]*[].constraint*[].leftOperand: LeftOperand
\[req]*[].\[rule]*[].constraint*[].rightOperand: RightOperand
\[req]*[].\[rule]*[].constraint*[].[logic:and|andSequence|or|xone]?: Constraint[]
\[req]*[].\[rule]*[].\[else:consequence|remedy|duty]*[]?: Duty[] # for Duty/Proh/Perm
\[req]*[].target?: string | Asset # DS Dataset ID (+ ODRL sugar)
\[req]*[].assigner?: string[] | Party(Col)[] # DS Agreement 1 PID (+ ODRL sugar)
\[req]*[].assignee?: string[] | Party(Col)[] # DS Agreement 1 PID (+ ODRL sugar)
\[req]*[].timestamp?: string # (only in DS Agreement)
\[req]*[].action?: string | Action # ODRL sugar
\[req]*[].actions?: string[] # GNAP & OAUTH (RAR): mode (read, write)
\[req]*[].identifier?: string # GNAP & OAUTH (RAR): local id at RS (ex. account)
\[req]*[].locations?: string[] # GNAP & OAUTH (RAR): target or its location
\[req]*[].datatypes?: string[] # GNAP & OAUTH (RAR): shape (images, metadata)
\[req]*[].privileges?: string[] # GNAP & OAUTH (RAR): scope/role (admin, user)
\[rule:permission|prohibition|obligation]*[]?: Rule[] # sugar ?
resource?: string # OAUTH (RI): target or its location (server/provider)
target?: string | Asset # sugar ?
action?: string | Action # sugar ??
actions?: string[] # sugar ??
scope?: string # OAUTH & UMA
#
subject?: # GNAP (subject data, directly from "AS" issuer)
subject.assertion_formats: string[] # response encoding: idtoken/saml2
subject.sub_id_formats: string[] # requested rfc9635 formats
subject.sub_ids[]?: # known rfc9635 subject info (format-based)
subject.sub_ids[].format: string
subject.sub_ids[].\[param]: string
#
assigner?: string | Party[] # sugar ? (RO/RC != provider)
assignee?: string | Party[] # sugar ? (RP != client/'agent')
providerPid?: string # DS provider msgs/events/agreements/... (= RO ~ assigner)
consumerPid: string # DS consumer msgs/events/agreements/... (= RP)
user?: string | ... # GNAP 'end user' (held to be RO, but should be RP)
user.sub_ids[]?: \[] # user ids (rfc9635 format)
user.sub_ids[].format: string
user.sub_ids[].\[param]: any
user.assertions[]?: \[ # user claims
user.assertions[].format: strings # encoding ('id_token', 'saml2')
user.assertions[].value: string
user.updated_at?: string
#
jwks?: JWKSet # OAUTH client instance's keys
jwks_uri?: uri # OAUTH alt ref to keys
client_secret?: string # OAUTH alt to keys
client_id: string # OAUTH & UMA alias ...
client: string | object # GNAP (APP != RP/assignee)
client.key: string | object # client instance's key
client.class_id?: string # client software(+version) indication
client.display?: # human-readable client representation
client.display.name?: string
client.display.uri?: uri
client.display.logo_uri?: uri
client_name?: string # OAUTH client info
client_uri?: uri # OAUTH client info
policy_uri?: uri # OAUTH client info
tos_uri?: uri # OAUTH client info
logo_uri?: uri # OAUTH client info
contacts?: string[] # OAUTH client info
software_id?: string # OAUTH client info
software_version?: string # OAUTH client info
#
interact: # GNAP interaction methods (supported by client)
interact.start[]: string[] | object[] # how to start interaction
interact.start[].mode: string # redirect | app | user_code | user_code_uri
interact.start[].[param]: any
interact.finish?: # how to return feedback from interaction
interact.finish.method: string # redirect | push (POST ~ DS callback)
interact.finish.uri: string # ~ redirect_uri or callbackAddress
interact.finish.nonce: string
interact.finish.hash_method?: string
interact.hints?: # optional info for interaction
interact.hints.ui_locales?: array[string]
callbackAddress?: uri # DS alias (~~~ redirect_uri, but for SERVER)
redirect_uri: uri # OAUTH alias (~~~ callbackAddress, but for CLIENT authz req)
response_type: string # OAUTH (authz req.): code | [extensions]
code_challenge?: string # OAUTH (authz req.)
code_challenge_method?: string # OAUTH (authz req.): plain | S256 | [extensions]
state?: string # OAUTH & UMA (opaque) client state (authz req)
#
grant_type: string # OAUTH (client_cred., refresh_token, code) & UMA (ticket)
refresh_token: string # OAUTH & UMA
ticket?: string # UMA: grant + resource/target + action/scope
code?: string # OAUTH (code flow)
pct?: string # UMA claim set ref
rpt?: string # UMA token to upgrade
```

## Merged

```yaml
@type?: (ContractRequest)Message # DS ; not type in other specs ?
access_token: *\[ # GNAP
  flags: string[] # GNAP e.g. token type ('bearer')
  label: string   # GNAP for multiple tokens
  access_token.access: # GNAP alias ... (always permissions!)
]
access_token.access: # ... (nested in GNAP) ...
authorization_details: # OAUTH (RAR) alias ...
offer: *\[ # DS (max 1) ; 'agreement' if finalized
  @id?: uri # DS
  uid?: uri # ODRL alias
  @type?:  uri # odlr:Policy / ds:Offer subclass (~ ds:Agreement)
  type: string # alias GNAP & OAUTH (RAR) (max 1 ~ more "profile")
  profile?: *[uri] # "alias" ODRL
  inheritFrom?: IRI[] # ODRL
  conflict?: ConflictTerm # ODRL
  permission|prohibition|obligation?: *\[ # ODRL & DS
    uid?: uri
    target?: string | Asset(Col) # ODRL uid or (refined) object
    action?: string | Action # ODRL uid or (refined) object
    assigner?: string[] | Party(Col)[] # ODRL uid or refinement
    assignee?: string[] | Party(Col)[] # ODRL uid or refinement
    constraint?: *\[ # ODRL/DS (Logical)Constraint
      operator: Operator
      leftOperand: LeftOperand
      rightOperand: RightOperand
      \[logic:and|andSequence|or|xone]?: Constraint[]
    ]
    \[else:consequence|remedy|duty]*[]?: Duty[] # for Duty/Proh/Perm
  ]
  target?: string | Asset # DS Dataset ID (+ ODRL sugar)
  assigner?: string[] | Party(Col)[] # DS 1 PID in Agreement (+ ODRL sugar)
  assignee?: string[] | Party(Col)[] # DS 1 PID in Agreement (+ ODRL sugar)
  timestamp?: string  # (only in DS Agreement)
  action?: string | Action # ODRL sugar
  actions?: string[] # GNAP & OAUTH (RAR): mode (operational: read/write/...)
  identifier?: string # GNAP & OAUTH (RAR): local id at RS (account number ...)
  locations?: string[] # GNAP & OAUTH (RAR): target or its location
  datatypes?: string[] # GNAP & OAUTH (RAR): shape (images/metadata/...)
  privileges?: string[] # GNAP & OAUTH (RAR): scope (level/role: admin/user/..)
]
permission|prohibition|obligation?: Rule[] # sugar ?
resource?: string # OAUTH (RI): target or its location (server/provider)
target?: string | Asset # sugar ?
action?: string | Action # sugar ??
actions?: string[] # sugar ??
scope?: string # OAUTH & UMA
#
subject?: # GNAP (request subject/RO data, directly from "AS" issuer)
  assertion_formats: string[] # requested response encoding: idtoken/saml2
  sub_id_formats: string[] # requested rfc9635 formats
  sub_ids[]?: \[ # known rfc9635 subject info (format-based)
    format: string
    \[param]: any
  ]
#
assigner?: string | Party[] # sugar ? (RO/RC != provider)
assignee?: string | Party[] # sugar ? (RP != client/'agent')
providerPid?: string # DS provider msgs/events/agreements/... (= RO ~ assigner)
consumerPid: string # DS consumer msgs/events/agreements/... (= RP)
user?: string | ... # GNAP 'end user' (held to be RO, but should be RP)
  sub_ids[]?: \[] # user ids (rfc9635 format)
    format: string
    \[param]: any
  ]
  assertions[]?: \[ # user claims
    format: strings # encoding ('id_token', 'saml2')
    value: string
  ]
  updated_at?: string
#
jwks?: JWKSet # OAUTH client instance's keys
jwks_uri?: uri # OAUTH alt ref to keys
client_secret?: string # OAUTH alt to keys
client_id: # OAUTH & UMA alias ...
client: string | ... # GNAP (APP != RP/assignee)
  key: string | object # client instance's key
  class_id?: string # client software(+version) indication
  display?: # human-readable client representation
    name?: string
    uri?: uri
    logo_uri?: uri
client_name?: string # OAUTH client info
client_uri?: uri # OAUTH client info
policy_uri?: uri # OAUTH client info
tos_uri?: uri # OAUTH client info
logo_uri?: uri # OAUTH client info
contacts?: string[] # OAUTH client info
software_id?: string # OAUTH client info
software_version?: string # OAUTH client info
#
interact: # GNAP interaction methods (supported by client)
  start: string[] | \[ # how to start interaction (string as shorthand)
    mode: string # redirect | app | user_code | user_code_uri
    \[param]: any
  ]
  finish?: # how to return feedback from interaction
    method: string # redirect | push (POST ~ DS callback)
    uri: string # ~ redirect_uri or callbackAddress
    nonce: string
    hash_method?: string
  hints?: # optional info for interaction
    ui_locales?: array[string]
callbackAddress?: uri # DS alias (~~~ redirect_uri, but for SERVER)
redirect_uri: uri # OAUTH alias (~~~ callbackAddress, but for CLIENT authz req)
response_type: string # OAUTH (authz req.): code | [extensions]
code_challenge?: string # OAUTH (authz req.)
code_challenge_method?: string # OAUTH (authz req.): plain | S256 | [extensions]
state?: string # OAUTH & UMA (opaque) client state (authz req)
#
grant_type: string # OAUTH (client_cred., refresh_token, code) & UMA (ticket)
refresh_token: string # OAUTH & UMA
ticket?: string # UMA: grant + resource/target + action/scope
code?: string # OAUTH (code flow)
pct?: string # UMA claim set ref
rpt?: string # UMA token to upgrade
```

## Information in a request

### Type of request

- ODRL: `Policy.profile`
- UMA: (NA)
- GNAP: (`access_token.access.type`)
- OAUTH: (`authorization_details.type`)
- DS: `offer.profile`

### Unique identifier

- ODRL: `Policy.uid`
- UMA: (NA)
- GNAP: (`access_token.label`)
- OAUTH: (NA)
- DS: `offer.@id`

### Human-readable description

- ODRL: `Policy[dc:description]`
- UMA: (NA)
- GNAP: (`hints.xyz`)
- OAUTH: (NA)
- DS: (NA)

### Target of request

- ODRL: `Rule.target` (as `permission[x].target`)
- UMA: (implicit in `ticket`)
- GNAP: (in `access_token.access`)
- OAUTH: `resource` (or in `authorization_details`)
- DS: `offer.target`

### Permission / action / scope

- ODRL: `action` (as `permission[x].action`)
- UMA: (implicit in `ticket`)
- GNAP: (`access_token.access[x].action`)
- OAUTH: `scope` or (`authorization_details[x].action`)
- DS: `offer.{obligation|permission|prohibition}[x].action`

#### Other authorization / offer details

- ODRL: `permission[x].{type|...}`
- UMA: NA
- GNAP: `access_token.access[x].{type|...}`, `access_token.{flags|label}`
- OAUTH: `authorization_details[x].{type|...}`
- DS: `offer.{obligation|permission|prohibition}[x].xyz`

Examples: identifier, locations, datatypes, privileges, actions ...

GNAP also has: `subject.{sub_ids|sub_id_formats|assertion_formats}`

### Provider identification

- ODRL: ...
- UMA: (NA)
- GNAP: ...
- OAUTH: (NA)
- DS: `providerPid`

.......

### Grant type

- ODRL: (NA)
- UMA: `grant_type` (with `ticket`)
- GNAP: ``
- OAUTH: ``
- DS: ``

### Claims

- ODRL: (NA)
- UMA: `claim_token` (with `claim_token_format`), `pct` (persisted)
- GNAP: `user.sub_ids`, `user.asssertions.{format|value}`, `user.updated_at`
- OAUTH: ``
- DS: ``

### Client identification (grants)

- ODRL: ...
- UMA: `client_id` (with `ticket`)
- GNAP: `client`
- OAUTH: `client_id` (with `client_secret`, `code`, or `refresh_token`)
- DS: `consumerPid`

### Client metadata

#### Callback / redirect URI

- ODRL: (NA)
- UMA: (NA)
- GNAP: `interact.{start|finish}`
- OAUTH: `redirect_uri`
- DS: `callbackAddress`

#### Key info

- ODRL: (NA)
- UMA: (NA)
- GNAP: `client.key`
- OAUTH: `jwks` or `jwks_uri`
- DS: (NA)

#### Other metadata

- ODRL: ...
- UMA: (NA)
- GNAP:
  - `client.class_id`
  - `client.display.{name,uri,logo_uri}`
  - `hints.{ui_locales|...}`
- OAUTH:
  - `grant_type`
  - `response_type`
  - ...
  - `contacts`
  - `client_name`
  - `client_uri`
  - `logo_uri`
  - `tos_uri`
  - `policy_uri`
  - `software_id`
  - `software_version`
- DS: ...

### Client state

- ODRL: (NA)
- UMA: `state`
- GNAP: (NA)
- OAUTH: `state`
- DS: (NA)

## Appendix A: Overview of request specifications

```yaml

# A4DS-ODRL (proposal)

@context: http://www.w3.org/ns/odrl.jsonld

@type: Request
profile:
  @id: https://w3id.org/oac#
uid: `http://example.org/HCPX-request/11092193193131`
description: HCPX to read Alice's health data for bariatric care
permission: Permission[] # rules
prohibition: Prohibition[] # rules
obligation: Duty[] # rules
inheritFrom?: IRI[]
conflict?: ConflictTerm
grant_type: urn:ietf:params:oauth:grant-type:uma-ticket
ticket: 016f84e8-f9b9-11e0-bd6f-0021cc6004de
claims: array[
  claim_token: ...
  claim_token_format: ...
]

@type: Rule # Permission | Prohibition | Duty
uid?: IRI
action: Action
target?: Asset # and/or other 'relation' sub-properties to Assets
assigner?: Party[]  # and/or other 'function' sub-properties to Parties
assignee?: Party[]  # and/or other 'function' sub-properties to Parties
[failure?: Rule[]] # as sub-property; cf. sub-classes
constraint?: (Logical)Constraint[]

@type: Permission # inherits from Rule
duty?: Duty[]

@type: Prohibition # inherits from Rule
remedy?: Duty[]

@type: Duty # inherits from Rule
consequence?: Duty[] # not with remedy ?



# UMA(+)

client_id: string
grant_type: string            # not in DS
permission?: string          # .offer.permission in DS
ticket?: string               # not in DS
scope?: string                # .offer.permission.action
claim_token?: string          # not in DS ?
claim_token_format?: string   # not in DS ?
pct?: string
rpt?: string
state?: string

# GNAP

client: string | ...
  key: string | object
  class_id?: string
  display?:
    name?: string
    uri?: string
    logo_uri?: string
access_token?: (array)[
  access: array[string | ...        # ~ RAR authorization_details
    type: string
    identifier?: string
    locations?: array[string]
    datatypes?: array[string]
    privileges?: array[string]
    actions?: array[string]
  ]
  flags?: array[strings] # 'bearer'
  label?: string
]
subject?:
  sub_id_formats: array[strings]    # https://www.rfc-editor.org/rfc/rfc9493
  assertion_formats: array[strings] # 'id_token', 'saml2'
  sub_ids?: array[objects]          # https://www.rfc-editor.org/rfc/rfc9493
user?: string | ...
  sub_ids?: array[objects]          # https://www.rfc-editor.org/rfc/rfc9493
  assertions?: array[
    format: strings                 # 'id_token', 'saml2'
    value: string
  ]
  updated_at?: string
interact?:
  start: array[strings] # 'redirect,' 'app', 'user_code', 'user_code_uri'
  finish?:
    method: string # 'redirect', 'push'
    uri: string
    nonce: string
    hash_method?: string
  hints?:
    ui_locales?: array[string]


# OAuth 2.0 RFCs

# common
client_id: string
scope?: string
resource?: string                 # Resource Identifier
authorization_details?:           # Rich Authorization Request
  type: string
  identifier?: string
  locations?: array[string]
  datatypes?: array[string]
  privileges?: array[string]
  actions?: array[string]
state?: string

# auth endpoint
redirect_uri: string
response_type: string
code_challenge: string
code_challenge_method?: string

# token endpoint
grant_type: string
client_secret?: string
refresh_token?: string
code_verifier?: string
code?: string

# additional client metadata
contacts?: array[string]
client_name?: string
client_uri?: string
logo_uri?: string
tos_uri?: string
policy_uri?: string
jwks?: JWKSet
jwks_uri?: string
software_id?: string
software_version?: string


# DataSpaces

@context: https://w3id.org/dspace/2025/1/context.jsonld
# ODRL profile: https://eclipse-dataspace-protocol-base.github.io/DataspaceProtocol/HEAD/message/schema/odrl.jsonld

@type: ContractRequestMessage
consumerPid: string
offer:
  @type: Offer | MessageOffer
  @id: string
  obligation?: array[Duty]
  permission?: array[Permission]
  profile?: string | array[string]
  prohibition?: array[Prohibition]
  target?: string
callbackAddress?: string
providerPid?: string

@type: Permission | Prohibition | Duty
action: Action # not defined; seems to be used with string values ~ scopes
constraint?: array[
  @type: Constraint
  leftOperand: LeftOperand
  operator: Operator
  rightOperand: RightOperand
  and?: array[Constraint]
  andSequence?: array[Constraint]
  or?: array[Constraint]
  xone?: array[Constraint]
]

```

---

# Now with graphs

```yaml


  content?: Graph
  details?: Graph


[policy.rule.content]

  subject: Party
  [policy.rule].assignee: Party
  #
  predicate: Action
  [policy.rule].action: Action
  #
  object: Asset
  [policy.rule].target: Asset

  subject: Party|Action|Target
  #
  predicate: Selector
  [policy.rule.assignee|action|target].refinement?.leftOperand: LeftOperand
  [policy.rule.assignee|action|target].refinement?.[...]: and | or | ...
  #
  object: BNode
  #
  subject: BNode
  #
  predicate: Comparator
  [policy.rule.assignee|action|target].refinement?.operator: Operator
  #
  object: any
  [policy.rule.assignee|action|target].refinement?.rightOperaond: RightOperand

  subject: Party|Action|Target
  #
  predicate: Selector
  [policy.rule.assignee|action|target].refinement?.[...]: and | or | ...
  #
  object: List<Triple>
  #
  subject: BNode
  #
  predicate: Comparator
  [policy.rule.assignee|action|target].refinement?.operator: Operator
  #
  object: any
  [policy.rule.assignee|action|target].refinement?.rightOperaond: RightOperand



  subject: Action
  [policy.rule].action?: Action

  predicate: LogicalConstraint
  [policy.rule.assignee].refinement?.[and|or|...]: .
  [policy.rule.action].refinement?.[and|or|...]: .
  [policy.rule.target].refinement?.[and|or|...]: .
  [policy.rule].constraint?.[and|or|...]: .

  object: List<Triple>
  [policy.rule.assignee].refinement?.[and|or|...]: Constraint
  [policy.rule.action].refinement?.[and|or|...]: Constraint
  [policy.rule.target].refinement?.[and|or|...]: Constraint
  [policy.rule].constraint?.[and|or|...]: Constraint

[policy.rule.content.object]

  subject: any
  [policy.rule.constraint]."subject": any

  predicate: Predicate
  [policy.rule.constraint].leftOperand: Selector

  object: BNode
  # _:v

  subject: BNode
  # _:v

  predicate: Predicate
  [policy.rule.constraint].operator: Operator

  object: any
  [policy.rule.constraint].rightOperand: any


  # `...`: A definition of resource(s) targetted by (the actions of) this rule.
  #
  # RECOMMENDED AssetDescription
  # - REQUIRED UNIQUE for ODRL permissions/prohibitions
  # - OPTIONAL UNIQUE for ODRL duties
  # - OPTIONAL UNIQUE for DataSpaces (derived from dataset context)
  # - IMPLICITLY REQUIRED in UMA ticket
  #
  # Aliases:
  # - Alt. domain `string`
  # - Alt. key `failure` for ODRL
  constraint*[]?: # ODRL/DS (Logical)Constraint
  constraint*[].operator: Operator
  constraint*[].leftOperand: LeftOperand
  constraint*[].rightOperand: RightOperand
  constraint*[].[logic:and|andSequence|or|xone]?: Constraint[]

  LeftOperand Operator RightOperand
  Selector Comparator Value

  constraint:
  constraint*[].operator: Operator
  constraint*[].leftOperand: LeftOperand
  constraint*[].rightOperand: RightOperand

  constraint*[].[logic:and|andSequence|or|xone]?: Constraint[]


  subject: Action
  [policy.rule].action: Action
  #
  predicate: withAgent
  #
  object: Agent
  [policy.rule].target: Agent ...
  ```
