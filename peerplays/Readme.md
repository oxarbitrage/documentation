
# GPOS

Gamified Proof of Stake is a protocol built for peerplays blockchain that changes how voting and dividends work.

Voting currently in the peerplays network(and other graphene blockchains) is done with the amount of core token(PPY) each user haves in balance. Gamification Proof of Stake(GPOS) protocol will introduce a new type of vesting balance named `gpos`. Participants who want to vote need to have their stake located in a `gpos` vesting balance. This new type of vesting balance works as a normal vesting balance except that it haves a linear 30 days default policy. It is just the type that identify the balance as valid to vote, one account can have multiple `gpos` balance and they will all sum when adding a vote.

At the moment of deciding (maintenance time) how many votes a worker, witness or advisor haves , votes will be computed by the `gpos` vested stake of all the users that are voting for a particular votable object.

Votes can decay as time pass if the staking account don't participate in further voting.

When GPOS period starts all the accounts will have a multiplier(`vesting_factor`) of 1, so voting power if a stakeholder haves 100 PPY vested will be:

`100 PPY * 1.0 = 100 voting power`

As time pass and stakeholder do not participate in further governance activities `vesting_factor` will be lower, lets suppose 0.5 at a given time so voting power will be:

`100 PPY * 0.5  = 50`

So basically , in GPOS: amount of core is not the same as voting power.

### What is considered voting activity?

We added a new field to the statistics object of each account: `last_date_voted`. This date will be updated each time the account:

- Vote for any votable object.
- Change his `vote_as` field(update proxy).

If user haves a proxy and proxy votes actively this will NOT be considered as voting activity for the stakeholder but yes will be computed to the proxy. ** check this in a test case

### GPOS and Dividends

In GPOS dividends will be paid only to the users that haves core token inside `gpos` vested balances. Users will not get paid dividends only by holding core token but they actually need to put this coins in this new special type of vested balance in order to get the benefits of the dividends.

Furthermore, our vesting factor(determined by our activity in the network governance) will also be used to multiply the amount of dividends users will get paid and not only for votes.

A user with a factor of 0.8 at dividend payment time assuming there are 10 PPY to share to his account, will get paid 8 PPY. 

The rest(2 PPY in this case) will be sent to the `committee-account`.

### GPOS global variables

GPOS introduces the following 3 global parameters to the blockchain that can be updated by the committee:


- `gpos_period` - Is the total duration of a GPOS period(default 6 months)
- `gpos_subperiod` - Is the duration of subperiods. Period must be divisible by subperiod(default 1 month).
- `gpos_period_start` - Is the date where the current gpos period start.(default: date of gpos hardfork)

Limitation: GPOS is only available(hardcoded) in the core token.

### Technical

Most of the voting and dividend logic is in `db_maint.cpp` file and by this it will be the most heavy modified code to implement gpos. 

Basically the GPOS protocol is implemented as:

- First, we introduce a new hardfork date (`HARDFORK_GPOS_TIME`). https://github.com/peerplays-network/peerplays/commit/88f18d16a7d971832c7c1435488d1b02d88c202c
- Next, we add the global variables needed. https://github.com/peerplays-network/peerplays/commit/62d2c1c63b4d6689e3a80884463d894b6e190dc5
- We create `vesting_type` field into the vesting balances object.  This is an `enum` that can take 2 values: `gpos` or `unespecified`. Before hf all vesting balances are of type unspecified. After hf the user can create gpos balances. As mentioned the gpos vesting balance haves a hardcoded linear policy of 30 days. https://github.com/peerplays-network/peerplays/commit/57a54f571488c6263b964e345dfa48ab98a9a6d6

Stakeholders can immediately vote after creating gpos vesting balance but they will have their funds locked for 1 gpos subperiod(generally equal to when dividends are paid).

No predefined period is required to vote as long as the vesting balance is there at the time of computing dividends and votes. We are encouraging stakeholders to take actions, if one of them decides to vest before payment and unvest after it we consider this activity and thus stakeholder must be paid.


- We only want the users to vest balances of gpos type after HF, our next step is to add this protection into the evaluator among with a forced vesting policy for gpos vesting types. Custom policy will be ignored. We also add this protection of not allowing gpos balances to be  created before HF into proposals evaluator. https://github.com/peerplays-network/peerplays/commit/46467a7bbe787581d02c74a51b936a41b4809d40

- As we added a new field to the vesting balance object, we need to bump the database. This is what we do next. https://github.com/peerplays-network/peerplays/commit/3de55137d5ccdb86481e655250a12a00353bdfbc

- Next we introduce the gpos changes into `db_maint.cpp`. https://github.com/peerplays-network/peerplays/commit/6d0128614093025bd389418ad5fed51d16ec17c4, https://github.com/peerplays-network/peerplays/commit/602b8530daa7b9fc51a382fd69f39e34d8343f84, https://github.com/peerplays-network/peerplays/commit/6b533828db53029702bf1fc690474096b0f54e75, https://github.com/peerplays-network/peerplays/commit/d468b146e878d13d26fd778adf8d81ff22293d63, 

- Next we add hardfork protection in proposals to avoid the creation of gpos vesting balances. https://github.com/peerplays-network/peerplays/commit/83ea345042b8764ec0094a5a7d59d1a4768d7fba

- We add the test cases. https://github.com/peerplays-network/peerplays/commit/49333a2c12cf7a7e559aa7775c11546453ba1d9c

- We add a create gpos vesting balance function from the cli wallet at https://github.com/peerplays-network/peerplays/pull/8/commits/610266481b29e01ff5ca8cf7c2286af3bfb9262d.

- Changed the global parameters to extensions to avoid different chain id at: https://github.com/peerplays-network/peerplays/pull/8/commits/d602e7160108e9e4b2d59538f793e90c77527e9c

- Refactor of `calculate_vesting_factor` is done at https://github.com/peerplays-network/peerplays/pull/8/commits/30c95f6010927339d91cb08e0e168f69549a0129, https://github.com/peerplays-network/peerplays/pull/8/commits/ec64df788981fed5fc17d7cc83b63fcd21465d8a and https://github.com/peerplays-network/peerplays/pull/8/commits/0e3d87eaad6e80eac1062447d8ced2b7d9a7a446

### The calculate_vesting_factor function

function `calculate_vesting_factor(account)` returns vesting factor of an account that will be used to calculating voting power or used to calculate dividends share at dividend payment time.

#### explication of the function:

   - get last time account in question voted form stats.
   - get from the blockchain properties, current global GPOS variables.
   - create and initialize additional variables needed for calculation.
   - make sure we are inside a GPOS period, assert if we are not.
   - get in what GPOS subperiod we are(current_subperiod).
   - make sure the current subperiod where we are is valid.
   - if account last vote is before gpos period start then vesting factor will be 0 and we exit.
   - vesting factor is = numerator / number of subperiods.
   - starting numerator is equal to the number of subperiods.
   - for subperiod = 1 and if the user voted the coefficient will be always 1.
   - if we are in a subperiod greater than 1 we need to adjust coeffcient as follows, lets assume 6 subperiods in this gpos setup:
      - create a list that goes from subperiod 2(remember we exluded subperiod 1 from calculations) to current subperiod(lets assume we are in subperiod 4).
      - reverse the list so recent past periods will be checked first.
      - we loop throw the list, for example if we are in period 4 reversed list will be: {4,3,2}.
      - by default numerator gets decreased with each iteration, following the example numerator at the first iteration will be 6 - 1 = 5.
      - inmeditly we check if he voted in the last subperiod(subperiod 3)
      - if account voted in subperiod 3 then numerator is increased(5 + 1) and we exit the loop. voting power here is full as account voted in the last period. this will also be happening if we are for example in period 5 and the account voted in period 4, if we are in period 3 and account voted in period 2, etc.
      - if the user did not voted in the past period then the loop continues, numerator is decrased again from 5 to 4 and we check period 2 for a vote.
      - if found, numerator is increased to 5 so coeffcient will be 5/6 for this account at this moment.
      - if vote not found in period 2 then loop continues, numerator is decrased to 3 and we check votes for the account in period 1
      - we are in the last iteration , if vote found in period 1 then numerator is increased back to 4 and coeffcient will be 4/6.
      - if no vote is found in period 1 the loop is over and the final numerator will be 3 and coeffcient 3/6, however we will never reach this level because this means that last vote is before gpos period started so coeffcient will be 0 before entering the loop.

Function Code: https://github.com/peerplays-network/peerplays/blob/0e3d87eaad6e80eac1062447d8ced2b7d9a7a446/libraries/chain/db_maint.cpp#L728-L785  

### Conclusion

This document is not complete and open to modifications. Developers and others are encouraged to check the `gpos_tests.cpp` to understand the protocol further.

### References

- https://www.peerplays.com/how-does-the-profit-sharing-function-work/
- https://github.com/oxarbitrage/documentation/blob/master/peerplays/Graphene-Improvement-Proposal-DPOS2GPOS.pdf
- https://github.com/peerplays-network/pips/blob/master/pip-0002.md
