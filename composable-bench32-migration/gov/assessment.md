# Assessment

The `gov` module `is being affected` by the migration process.

## SetDeposit
This function sets a Deposit to the gov store.
```go
// SetDeposit sets a Deposit to the gov store
func (keeper Keeper) SetDeposit(ctx sdk.Context, deposit v1.Deposit) {
	store := ctx.KVStore(keeper.storeKey)
	bz := keeper.cdc.MustMarshal(&deposit)
	depositor := sdk.MustAccAddressFromBech32(deposit.Depositor)

	store.Set(types.DepositKey(deposit.ProposalId, depositor), bz)
}
```
Deposit defines an amount deposited by an account address to an active proposal. In Deposit, it has Depositor which is a bech32 address.
```go
type Deposit struct {
	// proposal_id defines the unique id of the proposal.
	ProposalId uint64 `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty"`
	// depositor defines the deposit addresses from the proposals.
	Depositor string `protobuf:"bytes,2,opt,name=depositor,proto3" json:"depositor,omitempty"`
	// amount to be deposited by depositor.
	Amount []types.Coin `protobuf:"bytes,3,rep,name=amount,proto3" json:"amount"`
}
```
So the solution here is get all the data from types.DepositKey() and change the Depositor field into new prefix

## SetProposal
This function sets a proposal to store.
```go
// SetProposal sets a proposal to store.
func (keeper Keeper) SetProposal(ctx sdk.Context, proposal v1.Proposal) {
	bz, err := keeper.MarshalProposal(proposal)
	if err != nil {
		panic(err)
	}

	store := ctx.KVStore(keeper.storeKey)

	if proposal.Status == v1.StatusVotingPeriod {
		store.Set(types.VotingPeriodProposalKey(proposal.Id), []byte{1})
	} else {
		store.Delete(types.VotingPeriodProposalKey(proposal.Id))
	}

	store.Set(types.ProposalKey(proposal.Id), bz)
}
```
Proposal defines the core field members of a governance proposal. In Proposal, it has Proposer which is a bech32 address.
```go
// Proposal defines the core field members of a governance proposal.
type Proposal struct {
	// id defines the unique id of the proposal.
	Id uint64 `protobuf:"varint,1,opt,name=id,proto3" json:"id,omitempty"`
	// messages are the arbitrary messages to be executed if the proposal passes.
	Messages []*types1.Any `protobuf:"bytes,2,rep,name=messages,proto3" json:"messages,omitempty"`
	// status defines the proposal status.
	Status ProposalStatus `protobuf:"varint,3,opt,name=status,proto3,enum=cosmos.gov.v1.ProposalStatus" json:"status,omitempty"`
	// final_tally_result is the final tally result of the proposal. When
	// querying a proposal via gRPC, this field is not populated until the
	// proposal's voting period has ended.
	FinalTallyResult *TallyResult `protobuf:"bytes,4,opt,name=final_tally_result,json=finalTallyResult,proto3" json:"final_tally_result,omitempty"`
	// submit_time is the time of proposal submission.
	SubmitTime *time.Time `protobuf:"bytes,5,opt,name=submit_time,json=submitTime,proto3,stdtime" json:"submit_time,omitempty"`
	// deposit_end_time is the end time for deposition.
	DepositEndTime *time.Time `protobuf:"bytes,6,opt,name=deposit_end_time,json=depositEndTime,proto3,stdtime" json:"deposit_end_time,omitempty"`
	// total_deposit is the total deposit on the proposal.
	TotalDeposit []types.Coin `protobuf:"bytes,7,rep,name=total_deposit,json=totalDeposit,proto3" json:"total_deposit"`
	// voting_start_time is the starting time to vote on a proposal.
	VotingStartTime *time.Time `protobuf:"bytes,8,opt,name=voting_start_time,json=votingStartTime,proto3,stdtime" json:"voting_start_time,omitempty"`
	// voting_end_time is the end time of voting on a proposal.
	VotingEndTime *time.Time `protobuf:"bytes,9,opt,name=voting_end_time,json=votingEndTime,proto3,stdtime" json:"voting_end_time,omitempty"`
	// metadata is any arbitrary metadata attached to the proposal.
	Metadata string `protobuf:"bytes,10,opt,name=metadata,proto3" json:"metadata,omitempty"`
	// title is the title of the proposal
	//
	// Since: cosmos-sdk 0.47
	Title string `protobuf:"bytes,11,opt,name=title,proto3" json:"title,omitempty"`
	// summary is a short summary of the proposal
	//
	// Since: cosmos-sdk 0.47
	Summary string `protobuf:"bytes,12,opt,name=summary,proto3" json:"summary,omitempty"`
	// Proposer is the address of the proposal sumbitter
	//
	// Since: cosmos-sdk 0.47
	Proposer string `protobuf:"bytes,13,opt,name=proposer,proto3" json:"proposer,omitempty"`
}
```
So the solution here is get all the data from types.ProposalKey() and change the Proposer field into new prefix

## SetVote
This function sets a Vote to the gov store.
```go
// SetVote sets a Vote to the gov store
func (keeper Keeper) SetVote(ctx sdk.Context, vote v1.Vote) {
	store := ctx.KVStore(keeper.storeKey)
	bz := keeper.cdc.MustMarshal(&vote)
	addr := sdk.MustAccAddressFromBech32(vote.Voter)

	store.Set(types.VoteKey(vote.ProposalId, addr), bz)
}
```
Vote defines a vote on a governance proposal. In Vote, it has Voter which is a bech32 address.
```go
// Vote defines a vote on a governance proposal.
// A Vote consists of a proposal ID, the voter, and the vote option.
type Vote struct {
	// proposal_id defines the unique id of the proposal.
	ProposalId uint64 `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty"`
	// voter is the voter address of the proposal.
	Voter string `protobuf:"bytes,2,opt,name=voter,proto3" json:"voter,omitempty"`
	// options is the weighted vote options.
	Options []*WeightedVoteOption `protobuf:"bytes,4,rep,name=options,proto3" json:"options,omitempty"`
	// metadata is any  arbitrary metadata to attached to the vote.
	Metadata string `protobuf:"bytes,5,opt,name=metadata,proto3" json:"metadata,omitempty"`
}
```
So the solution here is get all the data from types.VoteKey() and change the Voter field into new prefix