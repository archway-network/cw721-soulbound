# CW721 Spec (soulbound): Non Fungible Tokens

CW721 is a specification for non-fungible tokens based on CosmWasm.
The name and design is based on Ethereum's ERC721 standard,
with some enhancements. The types in here can be imported by 
contracts that wish to implement this spec, or by contracts that call 
to any standard cw721 contract.

The specification is split into multiple sections, a contract may only
implement some of this functionality, but must implement the base.

Soulbound tokens are implemented by removing transfer and send capabilities.

## Base

This handles ownership, and allowances. These must be supported
as is by all CW721 contracts. Note that all tokens must have an owner, 
as well as an ID. The ID is an arbitrary string, unique within the contract.

### Messages

`Approve{spender, token_id, expires}` - Grants permission to `spender` to
operate as spender of the given token. This can only be performed when
`env.sender` is the owner of the given `token_id` or an `operator`. 
There can be multiple spender accounts per token, since the token is soulbound
this allows the approved spender to `Burn`

`Revoke{spender, token_id}` - This revokes a previously granted permission
to `Burn` the given `token_id`. This can only be granted when
`env.sender` is the owner of the given `token_id` or an `operator`.

`ApproveAll{operator, expires}` - Grant `operator` permission to burn
all tokens owned by `env.sender`. This approval is tied to the owner, not the
tokens and applies to any future token that the owner receives as well.

`RevokeAll{operator}` - Revoke a previous `ApproveAll` permission granted
to the given `operator`.

### Queries

`OwnerOf{token_id, include_expired}` - Returns the owner of the given token,
as well as anyone with approval on this particular token. If the token is
unknown, returns an error. Return type is `OwnerOfResponse`. If
`include_expired` is set, show expired owners in the results, otherwise, ignore
them.

`Approval{token_id, spender, include_expired}` - Return an approval of `spender`
about the given `token_id`. Return type is `ApprovalResponse`. If
`include_expired` is set, show expired owners in the results, otherwise, ignore
them.

`Approvals{token_id, include_expired}` - Return all approvals that owner given
access to. Return type is `ApprovalsResponse`. If `include_expired` is set, show
expired owners in the results, otherwise, ignore them.

`AllOperators{owner, include_expired, start_after, limit}` - List all
operators that can access all of the owner's tokens. Return type is
`OperatorsResponse`. If `include_expired` is set, show expired owners in the
results, otherwise, ignore them. If `start_after` is set, then it returns the
first `limit` operators *after* the given one.

`NumTokens{}` - Total number of tokens issued

### Queries

`ContractInfo{}` - This returns top-level metadata about the contract.
Namely, `name` and `symbol`.

`NftInfo{token_id}` - This returns metadata about one particular token.
The return value is based on *ERC721 Metadata JSON Schema*, but directly
from the contract, not as a Uri. Only the image link is a Uri.

`AllNftInfo{token_id}` - This returns the result of both `NftInfo`
and `OwnerOf` as one query as an optimization for clients, which may
want both info to display one NFT.

## Enumerable

### Queries

Pagination is achieved via `start_after` and `limit`. Limit is a request
set by the client, if unset, the contract will automatically set it to
`DefaultLimit` (suggested 10). If set, it will be used up to a `MaxLimit`
value (suggested 30). Contracts can define other `DefaultLimit` and `MaxLimit`
values without violating the CW721 spec, and clients should not rely on
any particular values.

If `start_after` is unset, the query returns the first results, ordered 
lexicographically by `token_id`. If `start_after` is set, then it returns the
first `limit` tokens *after* the given one. This allows straightforward 
pagination by taking the last result returned (a `token_id`) and using it
as the `start_after` value in a future query. 

`Tokens{owner, start_after, limit}` - List all token_ids that belong to a given owner.
Return type is `TokensResponse{tokens: Vec<token_id>}`.

`AllTokens{start_after, limit}` - Requires pagination. Lists all token_ids controlled by 
the contract.
