![overview](./assets/logo.png)

# Flex Contracts Monorepo

## Repository Structure

The repository is organized into the following directories:

1. `stakingpool`: Includes implementation of the staking NFT pool contracts.
#### Overview
The `StakingPool` contract allows users to stake their NFTs (ERC721 tokens) from specified eligible collections and earn rewards over time. This mechanism is designed to incentivize NFT holders by rewarding them based on the duration their NFTs remain staked.

#### Key Features

- **NFT Staking:** Users can stake their NFTs from collections that are approved by the contract owner.
- **Reward Accumulation:** Rewards are earned based on the amount of time an NFT is staked. The longer an NFT is staked, the more rewards it accrues.
- **Flexibility:** The contract supports various NFT collections, each with customizable reward parameters set by the contract owner.

#### Staking Process

 **Stake an NFT:**
   - Users call the `stakeNFT` function, providing the collection address and the token ID of the NFT they wish to stake.
   - Once staked, the NFT is locked in the contract and cannot be transferred until it is unstaked.

 **Unstake an NFT:**
   - Users can call the `unstakeNFT` function to retrieve their staked NFT.
   - During the unstaking process, any accumulated rewards are claimed and credited to the user.

 **Claiming Rewards:**
   - Rewards are calculated based on the staking duration and the reward rate set for the NFT collection.
   - Users can view their accumulated rewards for each staked NFT using the `getUserPointByItem` function.

#### Contract Configuration

- **Eligible Collections:** Only NFTs from collections that have been approved by the contract owner can be staked.
- **Reward Calculation:** The contract owner sets the time unit (in seconds) and the reward per unit time for each collection. These parameters determine how rewards accumulate for staked NFTs.

#### Security Considerations

- The contract implements reentrancy protection to secure staking and unstaking operations.
- Only the contract owner has the authority to modify which NFT collections are eligible for staking and to set the reward parameters.
  

1. `openedition`: Includes implementation of open-editions NFT minting mechanism contracts.
   
 **The List of contracts in openedition**
  1. ERC721OpenEditionMultiMetadata Contract
  2. ERC721OpenEdition Contract
  3. FlexDrop Contract

### ERC721OpenEditionMultiMetadata

The `ERC721OpenEditionMultiMetadata` contract is designed to handle complex interactions between its various functions and components to enable flexible minting phases, secure operations, and dynamic configuration for ERC721 Open Edition Token. It supports multiple metadata. It uses components imported from Alexandria Storage, OpenZeppelin,openedition utils. ETC.

### contract features
  1. **ERC721OpenEditionMultiMetadata Management**
  This contract is built to leverage the `ERC721MultiMetadataComponent` from openzeppelin which allows it to handle multiple sets of metadata of the ERC721OpenEdition Token.

  2. **Integration of FlexDrop logic**
  The contract integrates the `IFlexDropDispatcher` and `INonFungibleFlexDropToken` interfaces from openedition used for creation and management of the drop phases. This mechanism is designed to facilitate flexible minting and phase management, allowing drops to be updated, extended, or configured.

  3. **Validation of Allowed FlexDrop Contract**
  This contract has a mapping and a list tracking the "allowed" FlexDrop contract `allowed_flex_drop`; This means that Only contract listed  in the mapping is allowed to interact with the minting functionalities. This access control logic makes the contract more secure, restricting minting of the ERC721OpenEdition token to the approved contract address.

  4. **Phase Management**
  The contract includes functions for creating, updating, and managing phases of drops. It uses PhaseDrop structures and allows different minting conditions based on phases, which adds more control and scheduling to how and when tokens can be minted. This is especially useful for open edition drops where tokens might be minted in large quantities within set timeframes or conditions.

  5. **Ownership Control**
  The contract utilizes the`OwnableComponent` by adding functions like `assert_owner_or_self`, which ensures that certain actions can only be performed by either the contract itself or the owner. This provides enhanced control over who can actually call these functions.

  6. **Reentrancy Protection**
  This contract has `ReentrancyGuardComponent` for protecting the contract against reentrancy attacks. This means that, an external contract can not call a function from the contract before the initial function execution is completed.

  7. **ERC721OpenEditionMultiMetadata Configuration**
  Through the multi_configure function, this contract allows the owner to set multiple parameters, such as `base_uri` and `contract_uri`, phase details, and allowed payer for gas fee. This congiguration makes the contract flexible and easier to manage.


### ERC721OpenEdition Contract
 The `ERC721OpenEdition` contract is a StarkNet smart contract that is designed to integrate the functionalities of an ERC-721 (NFT) standards and flex-drop for minting of the openedition token.
 The contract imports the `OwnableComponent` from openzeppelin to authourise only the owner to call state changing functions, `ReentrancyGuardComponent` for prevent the contract from vulnerabilities or malicious actors. ETC.
 Anytime state changing transactions are executed in the contract, the contract emit  events that log the changes.For example an event emitted in the contract is; UpdateAllowedFlexDrop event, it will send a notification when the allowed flex drop is updated.

- **Functions in the  ERC721OpenEdition Contract**:
 
  1. `constructor(ref self: ContractState, creator: ContractAddress, name: ByteArray, symbol: ByteArray, token_base_uri: ByteArray,
  allowed_flex_drop: Array::<ContractAddress>)`:
  The contract's `constructor` function intializes  parameters which will be exectuted once when the contract is deployed.
  That means this parameters can not be changed after the contract is deployed

  2. `update_allowed_flex_drop(ref self: ContractState, allowed_flex_drop: Array::<ContractAddress>)`: The function is designed to manage the FlexDrop by updating the State with the new allowed flex drop when this function is called. It loops the allpwed flex drop, checks the length and increase it by 1 to include the index of the new allowed flex drop.

  3. `mint_flex_drop(ref self: ContractState, minter: ContractAddress, phase_id: u64, quantity: u64)`: This function allows FlexDrop contracts to mint the RC721OpenEdition NFT. It utilizes reentrancy protection and it internally calls `safe_mint_flex_drop` to perform the minting of the token explicitly specifying the required parameters.

  4. `create_new_phase_drop(ref self: ContractState, flex_drop: ContractAddress, phase_detail: PhaseDrop, fee_recipient: ContractAddress)`: This function interacts with the FlexDrop dispatcher to initiate new minting phases. The function asserts for the restriction of only owner and also for the allowed flex drop. 

  5. `update_phase_drop(ref self: ContractState, flex_drop: ContractAddress, phase_id: u64, phase_detail: PhaseDrop)`:This function updates the details of existing phaseDrops in the contract. It has a check for access control through the `assert_owner_or_self` to ensures that only the contract owner or the contract itself  can execute the function and also validation  check through  `assert_allowed_flex_drop`. It also allows for modifying the minting conditions within existing phase.

  6. `update_creator_payout(ref self: ContractState, flex_drop: ContractAddress, payout_address: ContractAddress) `: This function is designed to update the payout address for FlexDrop by changing the address that receives payouts with correct`payout_address` as specified in the function parameters. It as checks who execute the function through `assert_owner_or_self` to ensures of the ownable logic.

  7. `update_payer(ref self: ContractState, flex_drop: ContractAddress, payer: ContractAddress, allowed: bool)`: The function updates the address for paying gas fee of minting the NFT in the contract. It checks if the payers address to update the contract state is allowed, if yes it returns true; else it returns false. It asserts for the authourised caller of the function and allowed flex drop.

  8. ` multi_configure(ref self: ContractState, config: MultiConfigureStruct)`: This function which is only executed by the owner of the contract enables dynamic changes like setting the `base_uri`and `contract_uri` if their length is > 0, updating `phase_drops`, and modifies `update_creator_payout` and `update_payer` with configured parameters. It also update the`create_new_phase_drop` and `update_phase_drop` function based on the provided configuration Parameters.

  9. `get_mint_state(self: @ContractState, minter: ContractAddress, phase_id: u64) -> (u64, u64, u64)`: This function provides information on the minting state, such as the `total_minted` token per a wallet address,The `current_total_supply` of the erc721openedition token that is minted.

  10. `get_current_token_id(self:@ContractState) -> u256`: This function reads the `ContractState` and return the id of the current erc721openedition token that has been minted.

  11. `get_allowed_flex_drops(self:@ContractState) -> Span::<ContractAddress>`: The function is executed to retrieve and return the list of currently allowed FlexDrop addresses from the contract's storage.

  12. `safe_mint_flex_drop(ref self: ContractState, to: ContractAddress, phase_id: u64, quantity: u64)`: This is an internal function that performs the minting operation. It mints tokens to the `ContractAddress`, taking into consideration the parameters specified and also ensuring they are no reentrancy attacks. this function updates the `index` and `current_token_id` in the contract.

  13. `assert_allowed_flex_drop(self: @ContractState, flex_drop: ContractAddress)`: This function is put in place to check whether a FlexDrop address is allowed to execute minting and phase update functions. It reverts a message which reads `Only allowed FlexDrop'` if the address calling the functions with the check are not allowed.
  
  14. `get_total_minted(self: @ContractState) -> u64 `: The function reads the contract state retrieves the `total_minted` value of OpenEdition Non-Fungible Token minted in the contract.
 
  15. `assert_owner_or_self(self: @ContractState)`: This function is put in place verifies that the contract utilizes the ownable logic. That is, only the contract owner or the contract itself can perform state changing updates, used across multiple functions that require access control.
  

### FlexDrop Contract
The `FlexDrop` contract is designed for managing ERC721OpenEdition NFT drops with features such as the minting phase of the NFT, payment handling, whitelist minting, phase update amongst others. 

#### Contract Features 

This contract integrates imported interfaces and internal logic to manage flexible and secure NFT drops. It uses PausableComponent, security features such as reentrancy guard and ownership check from the openzeppelin. This ensures that the flexdrop securely managed.

The flex drop contract has structures(struct) for the storage, flexdrop minted, phase drop updated,creatorpayout updated, fee recipient updated, and payer updated. Aside the storage struct, every of the struct emits an event when they are updated. other events in this contract include; OwnableEvent, PausableEvent, and ReentrancyGuardEvent. 


- **Functions in the FlexDrop contract**

  1. `constructor(ref self: ContractState, owner: ContractAddress, currency_manager: ContractAddress, fee_currency: ContractAddress, fee_mint: u256, fee_mint_when_zero_price: u256, new_phase_fee: u256, domain_hash: felt252, validator: ContractAddress, signature_checker: ContractAddress,fee_recipients: Span::<ContractAddress>)`:This function initializes the contract with parameters setting it up to the intial state which will be exectuted once when the contract is deployed.

  2. `mint_public(ref self: ContractState, nft_address: ContractAddress, phase_id: u64, fee_recipient: ContractAddress, minter_if_not_payer: ContractAddress, quantity: u64, is_warpcast: bool)`: The function allow for public minting of the openedition NFT. It checks if the contract is not paused so as to prevent any action if it is. The contract checks reentrancy at the beginning and the end of the minting process. It validates the mint request, including phase status, allowed payer, and fee recipient. It also calculates the total mint price and processes the mint and pay.

  3. `whitelist_mint(ref self: ContractState, whitelist_data: WhiteListParam, fee_recipient: ContractAddress, proof: Array<felt252>)`: This function permits for minting of the openedition NFT for a whitelisted address based on the provided proof. It checks and validate the whitelist proof using the signature checker and checks if the proof is already used or if the address can mint the NFT in that particular phase. It protects for reentrancy at the beginning and end of minting. 

  4. `start_new_phase_drop(ref self: ContractState, phase_drop_id: u64, phase_drop: PhaseDrop, fee_recipient: ContractAddress)`: This function is designed to facilitate starting a new phase drop. It Checks if the caller is allowed. Most importanctly, the function asserts for only nonfungible flexdrop token, if the phase is not already started, that is; if its paused or unpaused. It checks the phase details and ensures that the required fee for starting a new phase drop is paid by which is an ERC20 token from the payer.

  5. `update_phase_drop(ref self: ContractState, phase_drop_id: u64, phase_drop: PhaseDrop)`: The function is designed to update the process of phase drops for the openedition NFT. It is done within the specified `start_time` and `timeout` for updating flexDrop. It checks to safegaurd against incorrect or unauthorized token type, for reentrancy at the beginning and end of the update process. It also emit an event for `PhaseDropUpdated`. 

  6. `update_creator_payout_address(ref self: ContractState, new_payout_address: ContractAddress)`: This function updates the payout address for the creator of  openedition NFT. Ensures the provided payout address is not address zero. It emits an event for `CreatorPayoutUpdated` to log the new payout address.

  7. `update_payer(ref self: ContractState, payer: ContractAddress, allowed: bool)`: This function adds the allowed payer for minting if allowed returns true and removes the payer if allowed returns false respectively as the case may be. It then emit an event for `PayerUpdated`.  Generally, It manages the list of allowed payers for the openedition NFT. 

  8. `pause(ref self: ContractState)` and `unpause(ref self: ContractState)`: These two functions are designed to enable or disable certain functions based on the contract state. The contract ensures that only the contract owner can call both the pause and the unpause functions respectfully.

  9. `change_currency_manager(ref self: ContractState, new_currency_manager: ContractAddress)`:This function is responsible for updating the currency manager contract address with the `new_currency_manager` strictly to be done by the contract owner thereby ensuring secure updates.

  10. `change_protocol_fee_mint(ref self: ContractState, new_fee_currency: ContractAddress, new_fee_mint: u256)`: This function is designed to update the minting fee and its currency in the contract. It checks if the old mint fee and currency differ then make changes for both to the  `new_fee_mint` and `new_fee_currency` respectively.

  11. `change_protocol_fee_mint_when_zero_price(ref self: ContractState, new_fee_currency: ContractAddress, new_fee_mint_when_zero_price: u256)`: The function checks if the `old_fee_mint_when_zero_price` is not equal to the `new_fee_mint_when_zero_price` then change the previous fee_mint to the new fee_mint and also update `fee_currency` when the mint price is zero.

  12. `update_protocol_fee_recipients(ref self: ContractState, fee_recipient: ContractAddress, allowed: bool)`: This function manages the  allowed address for the fee_recipient, ensuring that the allowed address is not an address zero or a duplicate fee recipient. If reverse is the case and all assertions for the two conditions do not pass, it through errors for `Only nonzero fee recipient` and `Duplicate fee recipient` respectively.

  13. `get_fee_currency(self: @ContractState) -> ContractAddress`: This function fetch and returns the `fee_currency` that is paid while minting the openedition NFT from the contract state.

  14. `get_fee_mint(self: @ContractState) -> u256` and `get_fee_mint_when_zero_price(self: @ContractState) -> u256`: The `get_fee_mint` function retrieves the fee that is paid for minting the NFT. While the `fee_mint_when_zero_price` function fetches the fee that is paid even when the mint price of the NFT is zero. 

  15. `get_new_phase_fee(self: @ContractState) -> u256`: This function retrieves and return the fee for new minting phase of the openedition NFT.

  16. `update_new_phase_fee(ref self: ContractState, new_fee: u256)`: This function leverages the ownable logic allowing only the owner to update the contract's state with the new fee to paid for each minting phase.

  17. `update_validator(ref self: ContractState, new_validator: ContractAddress)`, `get_validator(self: @ContractState) -> ContractAddress`: The `update_validator` function is primarily to update the new validator's address who is resposible for verifying signatures and proofs provided in the contract for specific functions. Only the contract owner is allowed to make this update.  For the `get_validator` function, it fetches and return the `new_validator` address that has been updated to ensure it is the right validator.

  18. `update_domain_hash(ref self: ContractState, new_domain_hash: felt252)`: This function is designed to update the contract's state with the `new_domain_hash` which is used by the contract to define the domain where certain transactions like proof verifications will be carried out. This function is only executed by the owner of the contract to ensure security and to restrict other addresses from having access to the function.

  19. `get_domain_hash(self: @ContractState) -> felt252`: This function retrieves and return the current domain hash that has been updated and is stored in the contract's state.

  20. `update_signature_checker(ref self: ContractState, new_signature_checker: ContractAddress)`: The function is designed to update the the contracts signature checker with the `new_signature_checker` in the contract's state. This function call is  restricted to only the contract owner.

  21. `get_signature_checker(self: @ContractState) -> ContractAddress`: This function fetches and return the contract's current signature checker from the contract state where it was stored.

  22. `get_phase_drop(self: @ContractState, nft_address: ContractAddress, phase_id: u64) -> PhaseDrop`: The `get_phase_drop` function retrieves the information of the phase drop associated with the specified `nft_address` and `phase_id`.

  23. `get_currency_manager(self: @ContractState) -> ContractAddress`: This function returns the `contract_address` of the current currency manager from the `currency_manager` storage. 

  24. `get_protocol_fee_recipients(self: @ContractState, fee_recipient: ContractAddress) -> bool`:This function checks if a specified `fee_recipient`address is actually the recipient of the protocol fee. If the address is the right address it returns true, else; it returns false.

  25. `get_creator_payout_address(self: @ContractState, nft_address: ContractAddress) -> ContractAddress`: This function retrieves the new `creator_payout_address` for making payment to the nft address from the contract's state which should not be an address zero.

  26. `get_enumerated_allowed_payer(self: @ContractState, nft_address: ContractAddress) -> Span::<ContractAddress>`: The function returns an an address from a list of addresses which is allowed to make payment for the `nft_address` provided. 

  27. `assert_only_non_fungible_flex_drop_token(self: @ContractState)`:This function writes a check to ensure that only supported NFT flex drops

  28.  `validate_new_phase_drop(self: @ContractState, phase_drop: @PhaseDrop)`:This function validates  the new phase drop configurations to ensure they meet the  supported standards. It asserts for start and end time for the phase drop to be as specified in the contract, the phase type which should == 1, the max mint Per wallet which should be > 0, and the whitelisted_currency.

  29. `assert_active_phase_drop(self: @ContractState, phase_drop: @PhaseDrop)`: To check that a phase drop is active, this function asserts that the phase drop's `start_time` is less than or equal the `block_time` and `end_time` is greater than the block time stamp else it reverts with the error message "public drop not active".

  30. `assert_whitelisted_currency(self: @ContractState, currency: @ContractAddress)`: This function asserts that a currency is verified by the currency manager as being whitelisted. If it's not whitelisted, the assertion will fail and revert with a "Currency not whitelisted" error message

  31. `assert_allowed_payer(self: @ContractState, nft_address: ContractAddress, payer: ContractAddress)`: This function checks that the specified payer's address is an allowed_payer. If the address is not allowed, it reverts with the "Only allowed payer" error message.

  32. `assert_valid_mint_quantity(self: @ContractState, nft_address: @ContractAddress, minter: @ContractAddress, phase_id: u64, quantity: u64, max_total_mint_per_wallet: u64,)`: The function is resposible for checking that; The minted quantity is greater than zero, the total minted quantity is less than or equal max total mint per wallet, The current supply and quantity is less than or equal total supply.

  33. `assert_allowed_fee_recipient(self: @ContractState, fee_recipient: @ContractAddress,)`: This function checks that the fee recipient is the allowed fee recipient in the flexDrop contract, if not it will revert that "only allowed fee recipient".

  34. `mint_and_pay(ref self: ContractState, nft_address: ContractAddress, payer: ContractAddress, minter: ContractAddress, is_warpcast: bool, phase_id: u64, quantity: u64, currency_address: ContractAddress, total_mint_price: u256, fee_recipient: ContractAddress, is_whitelist_mint: bool)`: The mint and pay function is reponsible for managing the minting of the NFT and all the payments made during the minting process. It calls the `split_payout` function to make the payment to all recipients. After the successful minting and  payments, an event is emitted for `flexDropMinted`.
 
  35. `split_payout(ref self: ContractState, from: ContractAddress, is_warpcast: bool, nft_address: ContractAddress, fee_recipient: ContractAddress, currency_address: ContractAddress, total_mint_price: u256, is_whitelist_mint: bool,)`: This function is designed to distribute fees from the openediton NFT minting process to address specified in the contract . It is called in the `mint_and_pay` function to split and send fees to each recipient who took part in the minting proccess like the creator.

  36. `remove_enumerated_allowed_payer(ref self: ContractState, nft_address: ContractAddress, to_remove: ContractAddress)`: The function loops through the list of allowed payers and remove a specified address of the `enumerated_allowed_payer` from the list.
 


3. `marketplace`: Includes implementation of the marketplace contracts.


#### Development Setup

You will need to have Scarb and Starknet Foundry installed on your system. Refer to the documentations below:

-   [Starknet Foundry](https://foundry-rs.github.io/starknet-foundry/index.html)
-   [Scarb](https://docs.swmansion.com/scarb/download.html)

To use this repository, first clone it:

```
git clone git@github.com:Flex-NFT-Marketplace/Flex-Marketplace-Contract.git
```

### Building contracts

To build the contracts, run the command:

```
scarb build
```

Note: Use scarb version `2.6.3`.

### Running Tests

To run the tests contained within the `tests` folder, run the command:

```
scarb test
```
3. `marketplace`: Includes implementation of the marketplace contracts.


### Marketplace Contracts

#### Marketplace Contracts List

* CurrencyManager
* ExecutionManager
* Marketplace 
* Proxy
* RoyaltyFeeManager
* RoyaltyFeeRegistry
* SignatureChecker2
* StrategyHighestBidderAuctionSale
* StrategyStandardSaleForFixedPrice
* TransferManagerNFT
* ERC1155TransferManager
* TransferSelectorNFT

#### CurrencyManager 
  The `CurrencyManager` contract is designed to manage a whitelist of currencies. It operates under the control of an owner who has the authority to add or remove currencies from this list. The contract stores whitelisted currencies and their indices using a `LegacyMap` and maintains a separate `LegacyMap` to track each currency's index.

#### Contract Features
1. **Ownership Control** :

    The contract is built on the concept of ownership. It uses OpenZeppelin's `OwnableComponent`, meaning only the owner of the contract has the power to make changes, such as adding or removing currencies from the whitelist. This ensures that the whitelist is securely managed.

2. **Whitelisting Currencies** : 

    The core function of this contract is to manage a whitelist of currencies. Think of the whitelist as a VIP list of currencies that are allowed to participate in the system. The contract lets the owner add new currencies to this list and remove them when they’re no longer needed.

3. **Event-Driven** :

    The contract is designed to be very communicative. Every time a currency is added or removed from the whitelist, the contract emits an event, like sending out a notification. These events are crucial for keeping other parts of the system, or even external systems, informed about the changes.

#### Contract Functions

1. `initializer(ref self: ContractState, owner: ContractAddress, proxy_admin: ContractAddress)` :
   
    This function sets up the initial state of the contract. It assigns the owner and ensures that the contract can only be initialized once. If the contract has already been initialized, it will throw an error. It also calls the initializer function of the OwnableComponent to manage ownership logic.

2. `add_currency(ref self: ContractState, currency: ContractAddress)` :

    This function allows the contract owner to add a new currency to the whitelist. It first checks if the currency is already whitelisted by looking up its index. If not, it increments the count of whitelisted currencies, adds the new currency to the list, and updates the mapping of currency indices. It then emits a `CurrencyWhitelisted` event with the current timestamp.

3. `remove_currency(ref self: ContractState, currency: ContractAddress)` :

    This function allows the owner to remove a currency from the whitelist. It verifies whether the currency is currently whitelisted by checking its index. If whitelisted, the function replaces the currency with the last one in the list, updates the indices, and reduces the count of whitelisted currencies. It also emits a `CurrencyRemoved` event with the current timestamp.

4. `is_currency_whitelisted(self: @ContractState, currency: ContractAddress) -> bool` :

    This function checks if a given currency is on the whitelist by looking up its index. If the index is non-zero, the currency is considered whitelisted, and the function returns true; otherwise, it returns `false`.

5. `whitelisted_currency_count(self: @ContractState) -> usize` :

    This function returns the total number of currencies currently on the whitelist. It simply reads the `whitelisted_currency_count` from storage, which tracks how many currencies have been added to the whitelist.

6.  `whitelisted_currency(self: @ContractState, index: usize) -> ContractAddress` :

    This function retrieves the currency at a specific position in the whitelist based on the provided index. It looks up the currency’s address in the list and returns it. This is useful for iterating through or displaying the list of whitelisted currencies.

#### ExecutionManager 
  The `ExecutionManager` contract is a core component designed to manage and control a list of strategies within a decentralized system. It maintains a whitelist of strategies, each identified by a `ContractAddress`, determining which strategies are authorized for execution. The contract leverages OpenZeppelin's `OwnableComponent`, ensuring that only the contract owner can modify the whitelist.

#### Contract Features
1. **Ownership and Access Control** :

    The contract uses the `OwnableComponent` from OpenZeppelin to manage ownership. This component restricts critical functions so that only the contract's owner can execute them. This ensures that sensitive operations, like adding or removing strategies, are secure and only performed by an authorized entity.

1. **Strategy Whitelisting** :

    The core functionality of this contract is to manage a whitelist of execution strategies. Strategies are represented by their `ContractAddress`, and only those on the whitelist are valid for execution within the system.

1. **Event Emission** :

    The contract emits events whenever a strategy is added to or removed from the whitelist. This allows for transparency and traceability, enabling external observers to track changes to the list of approved strategies.

#### **Contract Functions**
1. `initializer(ref self: ContractState, owner: ContractAddress)` :
   
    This function initializes the contract, setting up the initial state and assigning the owner. It ensures the contract is only initialized once by checking if it has already been initialized `(assert!(!self.initialized.read(), "ExecutionManager: already initialized");)`. Once initialized, the `OwnableComponent` is set up with the provided owner address.

2. `add_strategy(ref self: ContractState, strategy: ContractAddress)` :

    This function allows the owner to add a new strategy to the whitelist. It checks if the strategy is already whitelisted using the `whitelisted_strategies_index` map. If not, the strategy is added, the strategy count is incremented, and a `StrategyWhitelisted` event is emitted.

3. `remove_strategy(ref self: ContractState, strategy: ContractAddress)` :

    This function allows the owner to remove a strategy from the whitelist. It verifies that the strategy is currently whitelisted, then removes it and adjusts the indices in the list. A `StrategyRemoved` event is emitted to inform external observers of the change.

4. `is_strategy_whitelisted(self: @ContractState, strategy: ContractAddress) -> bool` :

    This function checks if a specific strategy is whitelisted. It returns `true` if the strategy is on the whitelist and `false` if it is not. This function is crucial for determining the validity of a strategy before execution.

1. `get_whitelisted_strategies_count(self: @ContractState) -> usize` :

    This function returns the total number of whitelisted strategies. It provides a simple way to retrieve the size of the whitelist, which can be useful for iterating over all strategies or for display purposes.

1. `get_whitelisted_strategy(self: @ContractState, index: usize) -> ContractAddress` :

    This function retrieves a strategy from the whitelist based on its index. It allows external contracts or users to access specific strategies by their position in the list, enabling iteration or specific strategy retrieval for execution or analysis.

#### MarketPlace 

The `MarketPlace` contract is a key component managing decentralized trading of assets on a Starknet. It facilitates various operations, such as initializing the marketplace, managing orders, and executing trades, all while ensuring the integrity and security of the transactions through ownership, upgradability, and reentrancy protection. The contract's modular design allows for flexibility and future enhancements, making it a powerful tool for decentralized trading platforms.

#### Contract Features

1. **Initialization and Ownership** :

  - The contract supports initialization through the `initializer` function, setting up essential parameters like `hash_domain`, `protocol_fee_recipient`, and various manager contracts (currency, execution, royalty, etc.).

  - Ownership is controlled through the `OwnableComponent`, allowing only the owner to execute certain privileged operations, such as contract upgrades.

2. **Upgradability** :

    The contract is designed to be upgradeable, meaning its logic can be updated without disrupting the entire system. This is achieved using the `UpgradeableComponent`, ensuring the contract can evolve over time.

3. **Order Management** :

    Users can cancel all their active orders or specific orders using `cancel_all_orders_for_sender` and `cancel_maker_order`, respectively. This provides flexibility and control over the trading process.

4. **Order Matching** :

    The core functionality revolves around matching orders between buyers and sellers. The contract handles both `match_ask_with_taker_bid` and `match_bid_with_taker_ask` scenarios, ensuring orders are matched correctly, fees are transferred, and assets are moved securely.

5. **Auction Handling** :

    The contract also supports auction-style sales, which are executed through the `execute_auction_sale` function. This allows for more dynamic pricing mechanisms based on bidding.

6. **Fee and Royalty Management** :

    The contract manages protocol fees and royalties, ensuring that all participants are compensated fairly. Various managers (currency, execution, royalty fee) are updated through dedicated functions.

7. **Signature Verification** :

    To ensure the integrity of transactions, the contract includes robust signature verification through the `signature_checker` , which validates the authenticity of orders before execution.

8. **Security and Reentrancy Protection** :

    The contract includes a `ReentrancyGuardComponent` to prevent reentrancy attacks, ensuring that each function is executed atomically and securely.

#### Contract Functions

1. `initializer()` :

    This function initializes the contract by setting up critical state variables like the hash domain, protocol fee recipient, and various managers (currency, execution, royalty, etc.). It also assigns ownership and administrative roles.

2. `upgrade()` :

    Allows the contract owner to upgrade the contract's logic by providing a new class hash. This is essential for maintaining the contract's functionality over time.

3. `cancel_all_orders_for_sender()` :

    Cancels all orders for the sender with a nonce greater than a specified minimum. This function emits an event to notify the marketplace of the cancellation.

4. `cancel_maker_order()` :

    Cancels a specific order based on its nonce. It ensures that the order is not executed or canceled multiple times and emits an event upon successful cancellation.

5. `match_ask_with_taker_bid()` :

    Matches a taker bid with a maker ask, facilitating a transaction between the buyer and seller. The function validates the orders, transfers fees, funds, and the NFT, and then emits an event to record the transaction.

6. `match_bid_with_taker_ask()` :

    Matches a taker ask with a maker bid, enabling the sale of an asset to the highest bidder. This function follows similar steps as match_ask_with_taker_bid() but in reverse order, ensuring that all conditions are met before executing the sale.

7. `execute_auction_sale()` :

    Executes a sale through an auction mechanism, matching the highest bid with the reserve price. It handles the transfer of assets and funds and ensures the auction rules are followed.

8. `update_hash_domain()` :

    Updates the hash domain used for computing order hashes. This function can be used to modify the underlying cryptographic parameters of the marketplace.

9.  `update_protocol_fee_recepient()` :

    Allows the contract owner to update the recipient of protocol fees, enabling dynamic management of fee collection.

10. `update_currency_manager()`, `update_execution_manager()`, `update_royalty_fee_manager()`, `update_transfer_selector_NFT()`, `update_signature_checker()` :

    These functions allow the contract owner to update various managers responsible for different aspects of the marketplace, such as currency management, execution strategies, and signature verification.

11. `get_hash_domain()`, `get_protocol_fee_recipient()`, `get_currency_manager()`, `get_execution_manager()`, `get_royalty_fee_manager()`, `get_transfer_selector_NFT()`, `get_signature_checker()` :

    These getter functions return the current state of various contract variables, allowing users and external systems to query important information about the marketplace's configuration.

12. `get_user_min_order_nonce()`:  

     Retrieves the minimum order nonce for a given user, which is used to track the user's active orders and prevent replay attacks.

14. `get_is_user_order_nonce_executed_or_cancelled()`:  

    Checks whether a specific order nonce has been executed or canceled, providing a way to verify the status of an order.

#### Proxy  

  The Proxy contract is a fundamental component in decentralized systems that allows the upgrade of smart contracts.This capability is crucial in a rapidly evolving environment like StarkNet, where contract logic might need to be updated without disrupting the existing system or user interactions.

#### Contract Features
1. **Contract Upgradeability** :
   
    The primary feature of the Proxy contract is its ability to upgrade the underlying implementation of the contract. This is done through the `upgrade` function, which allows the owner (or admin) to replace the current contract logic with a new one, represented by a new `ClassHash`.
 
1. **Admin Control** :

    The contract has an admin management feature, allowing the current admin to designate a new admin using the `set_admin` function. 
    The admin have the authority to initiate upgrades and control the overall behavior of the Proxy contract.

2. **Default Fallback Mechanism** :

    The contract provides a fallback mechanism `__default__` and `__l1_default__` functions that captures any calls to functions not explicitly defined in the Proxy contract. It is a standard feature in proxy patterns, that ensuring any undefined function calls are forwarded to the current implementation, enabling the Proxy to act as a transparent intermediary.

#### Contract Functions

1. `upgrade(ref self: ContractState, new_implementation: ClassHash)` :

   This function is responsible for upgrading the contract's logic by updating the `ClassHash` to point to a new implementation. This allows the contract to evolve without changing its address or disrupting ongoing operations.

2. `set_admin(ref self: ContractState, new_admin: ContractAddress)` :
    
    This function sets a new admin for the contract, transferring control and responsibility to the new address. The admin is the entity with the authority to upgrade the contract and manage its critical settings.

3.  `get_implementation(self: @ContractState) -> ClassHash` :
  
    This function returns the current implementation's `ClassHash`, allowing external parties to verify which contract logic is currently active.

4.  `get_admin(self: @ContractState) -> ContractAddress` :
  
    This function retrieves the current admin's address, providing transparency regarding who controls the Proxy's upgrade functionality.

5.  `__default__(self: @ContractState, selector: felt252, calldata: Span<felt252>) -> Span<felt252>` :
   
    This fallback function is invoked when a call is made to a function that does not exist in the Proxy contract. It ensures that such calls are forwarded to the current implementation, maintaining the Proxy's role as a conduit for interacting with the underlying logic.

6.  `_l1_default__(self: @ContractState, selector: felt252, calldata: Span<felt252>)` :
  
    Similar to `__default__`, this function handles calls from Layer 1 (L1), ensuring that these calls are also forwarded to the appropriate implementation logic.

#### RoyaltyFeeManager

  The `RoyaltyFeeManager` contract is designed to manage and calculate royalty fees associated with the sale of digital assets, such as NFTs, on the StarkNet platform. It ensures that creators or rights holders receive appropriate fees whenever their assets are sold. This contract leverages established standards like ERC-2981 for royalty calculations and integrates with a royalty fee registry to manage the logic and data involved in this process.

#### Contract Features

1. **Royalty Calculation and Distribution** :
   
    At the core of the RoyaltyFeeManager contract is the ability to calculate the correct royalty fee and determine the recipient of this fee. This is facilitated through the `calculate_royalty_fee_and_get_recipient` function, which checks both the royalty fee registry and the ERC-2981 standard to determine the appropriate recipient and fee amount for any given transaction.

2. **Integration with Royalty Fee Registry** :
   
    The contract interfaces with a `RoyaltyFeeRegistry`, which stores information on royalty fees for different collections. This registry acts as the primary source for royalty fee calculations, and the contract fetches the relevant data from it using the `get_royalty_fee_registry` function.

1. **Support for ERC-2981 Standard** :
   
    The contract supports the ERC-2981 standard, a widely recognized standard for royalty fees in the NFT space. The `INTERFACE_ID_ERC2981` function returns the standard identifier for ERC-2981, allowing the contract to check if a given collection supports this standard and, if so, to retrieve royalty information directly from the collection.

1. **Upgradeable Architecture** :
   
    The RoyaltyFeeManager contract is designed to be upgradeable, allowing the logic to be modified without needing to deploy a new contract. This is achieved through the `upgrade` function, which allows the contract owner to change the underlying implementation by specifying a new ClassHash.

1. **Ownership and Access Control** :
   
    The contract uses the OpenZeppelin `OwnableComponent` to manage ownership. The owner has exclusive rights to perform critical operations, such as upgrading the contract or initializing it with the appropriate fee registry and owner addresses. The `initializer` function sets up the contract's state, including assigning ownership and linking it to the royalty fee registry.

### Contract Functions

1. `initializer(ref self: ContractState, fee_registry: ContractAddress, owner: ContractAddress)` :
   
    This function initializes the contract, setting the ERC-2981 interface ID, linking the contract to the specified royalty fee registry, and assigning the contract's owner. It is called once during contract deployment to configure the initial state.

2. `upgrade(ref self: ContractState, impl_hash: ClassHash)` :
   
   This function allows the contract owner to upgrade the contract by providing a new implementation `ClassHash`. This is crucial for maintaining the contract's flexibility and ensuring it can adapt to new requirements or improvements without redeploying.

3. `INTERFACE_ID_ERC2981(self: @ContractState) -> felt252` :
   
    This function returns the unique identifier for the ERC-2981 interface, allowing the contract to check whether a given collection adheres to this standard. It is used in conjunction with other functions to determine how to calculate royalties.

4. `calculate_royalty_fee_and_get_recipient(self: @ContractState, collection: ContractAddress, token_id: u256, amount: u128) -> (ContractAddress, u128)` :
   
    This function calculates the royalty fee for a given sale and returns the recipient address and fee amount. It first checks the royalty fee registry, and if no data is found, it checks if the collection supports ERC-2981 to retrieve the necessary information. This function is central to ensuring that creators receive their due royalties during asset sales.

5. `get_royalty_fee_registry(self: @ContractState) -> IRoyaltyFeeRegistryDispatcher` :
    This function returns the contract address of the linked royalty fee registry. The registry is where the contract looks first to find information about royalty fees for specific collections, making it an essential component of the overall royalty calculation process.

### RoyaltyFeeRegistry

The RoyaltyFeeRegistry contract is designed to manage and enforce royalty fees for digital asset transactions on the StarkNet platform. It plays a critical role in ensuring that creators and rights holders receive appropriate royalties when their assets are sold. The contracts is allowing the owner to set royalty limits, update royalty information for specific collections, and retrieve royalty details when its needed.

### Contract Features

1. **Royalty Fee Management** :

    The contract allows for the registration and management of royalty fees for different digital asset collections. It ensures that these fees adhere to a maximum limit, protecting users from excessive charges. The `update_royalty_info_collection function` is central to this feature, enabling the owner to set or update the royalty information for specific collections.

2. **Ownership and Access Control** :

    The contract uses the `OwnableComponent` from OpenZeppelin to ensure that only the contract owner can make critical changes, such as updating the royalty fee limit or modifying royalty information. The `initializer` and `update_royalty_fee_limit functions` rely on this ownership control to restrict access to sensitive operations.

3. **Event Emission for Transparency** :

    To maintain transparency, the contract emits events whenever there are updates to the royalty fee limit or changes to the royalty information of a collection. The `NewRoyaltyFeeLimit` and `RoyaltyFeeUpdate` events are triggered by the respective functions to log these changes on the blockchain.

4. **Royalty Fee Limit Enforcement** :

    The contract enforces a maximum royalty fee limit (set during initialization) to ensure that fees do not exceed a predefined threshold. This is important for maintaining fairness and protecting users from potential exploitation. The `update_royalty_fee_limit` function allows the owner to adjust this limit within acceptable bounds.

5. **Royalty Fee Calculation** :

    The contract provides functions to calculate and retrieve royalty fees for a given transaction. The `get_royalty_fee_info` function calculates the royalty amount based on the transaction value and the registered fee for the collection. This ensures that the correct amount is distributed to the designated recipient.


#### Contract Functions

1. `initializer(ref self: ContractState, fee_limit: u128, owner: ContractAddress)` :

    This function initializes the contract, setting the maximum royalty fee limit and assigning ownership. It ensures that the contract is only initialized once, preventing reconfiguration after deployment. It also checks that the initial fee limit is within the allowed maximum.

2. `update_royalty_fee_limit(ref self: ContractState, fee_limit: u128)` :

    This function allows the contract owner to update the maximum allowable royalty fee limit. It ensures that the new limit is within the predefined maximum and emits an event to log the change. This function is crucial for maintaining control over the fee structure as market conditions evolve.

3. `update_royalty_info_collection(ref self: ContractState, collection: ContractAddress, setter: ContractAddress, receiver: ContractAddress, fee: u128)` :

    This function updates the royalty information for a specific digital asset collection. It records who set the royalty, who will receive it, and the percentage fee. It ensures that the fee does not exceed the limit set by the owner and emits an event to log the update. This function is key to maintaining accurate and fair royalty distribution.

4. `get_royalty_fee_limit(self: @ContractState) -> u128` :

   function retrieves the current maximum royalty fee limit. It is used internally to validate that any updates to collection royalties do not exceed this limit. It also provides transparency by allowing anyone to check the enforced fee limit.

5. `get_royalty_fee_info(self: @ContractState, collection: ContractAddress, amount: u128) -> (ContractAddress, u128)` :

    This function calculates the royalty amount for a given transaction and returns the recipient's address and the royalty amount. It ensures that the correct royalty is applied based on the transaction value and the fee percentage registered for the collection.

6. `get_royalty_fee_info_collection(self: @ContractState, collection: ContractAddress) -> (ContractAddress, ContractAddress, u128)` :
   **
    This function retrieves detailed royalty information for a specific collection, including the addresses of the setter and receiver, and the fee percentage. It provides transparency and allows stakeholders to verify the registered royalty details.

### Signature_Checkers2

The `SignatureChecker2` contract is designed for a marketplace application on Starknet, focusing on the verification of digital signatures, particularly for Maker Orders and whitelist minting processes. It ensures that orders and whitelist claims are authentic and authorized by the correct entities.

### Contract Features

1. **WhiteListParam Structure** :

    Represents a whitelist entry, containing a `phase_id`, `nft_address`, and minter. This structure is crucial for validating whether a specific user (minter) is authorized to mint NFTs during a particular phase.

2. **Maker Order** :
    A core element of marketplace transactions, the MakerOrder contains details about an order, including whether it's an ask or bid, the price, the NFT's collection, and the time frame within which the order is valid.

3. **Hash Constants** :
    Several hash constants (`STARKNET_MESSAGE`, `HASH_MESSAGE_SELECTOR`, etc.) are defined, representing specific data structures and types. These constants are used in hashing processes to generate unique identifiers for different structures and orders.


#### Contract Functions

1. **Signature Verification** :
  The contract offers methods (`verify_maker_order_signature`, `verify_maker_order_signature_v2`) to verify the authenticity of a MakerOrder using its digital signature. This ensures that only orders signed by authorized entities can be executed.

2. **Hash Computation** :
  The contract provides functions (`compute_maker_order_hash`, `compute_message_hash`, etc.) to compute unique hashes for orders and whitelist entries. These hashes are essential for verifying the integrity and authenticity of the data.

3. **Whitelist Minting** :
  The contract includes functionality to handle whitelist minting, where a specific message hash is computed based on the whitelist data (`compute_whitelist_mint_message_hash`). This ensures that only those on the whitelist can mint NFTs during specific phases.

4. **Struct Hashing** :
  The contract defines several implementations of a `hash_struct` trait for different data types (`WhiteListParam`, `MakerOrder`, `u256`). This trait allows these structures to be hashed consistently, which is vital for their use in signature verification.

### StrategyHighestBidderAuctionSale
  This contract, `StrategyHighestBidderAuctionSale`, is designed to facilitate and manage auction-based sales within a decentralized marketplace on Starknet. It offers a range of features tailored for handling auctions, including fee management, execution checks for both taker and maker orders, and upgradeability, all secured by ownership controls.

### Comtract Features :
1. **Ownership and Control** : The contract integrates ownership functionality using the OwnableComponent, allowing only the contract owner to execute certain actions like updating the protocol fee or upgrading the contract. This ensures that critical operations remain under the control of a trusted party.

2. **Protocol Fee Management** : The contract maintains a protocol_fee, which can be set during initialization and updated later by the owner. This fee is applied to transactions within the auction, ensuring the platform can generate revenue or cover operational costs.

3. **Order Execution Checks** : The contract provides mechanisms to validate whether a taker’s bid or ask can be executed against a maker’s order. It checks for matching token IDs, valid time frames, and whether the bid meets the highest bid criteria, ensuring that only valid transactions go through.

4. **Upgradeable Architecture** : Leveraging the UpgradeableComponent, this contract can be upgraded with a new implementation without disrupting the existing state. This feature allows the contract to evolve and adapt to new requirements or improvements over time.

### Contract Functions:
1. `initializer` : Sets the initial protocol fee and assigns ownership of the contract. This function is crucial for setting up the contract’s parameters and ensuring that the correct entity has control.

2. `update_protocol_fee` : Allows the owner to modify the protocol fee. This function is restricted to the owner, ensuring that fee adjustments cannot be made arbitrarily.

3. `protocol_fee` : Returns the current protocol fee. This function provides transparency about the fee structure to users and other contracts interacting with this contract.

4. `can_execute_taker_ask` : Evaluates whether a taker’s ask can be executed against a maker’s bid. It checks conditions like token ID matching, valid auction times, and whether the bid is high enough. If all conditions are met, the function returns true along with the relevant token ID and price.

5. `can_execute_taker_bid` : Similar to the above, but for evaluating whether a taker’s bid can be executed against a maker’s ask. This function ensures that bids meet the criteria to proceed with the transaction.

6. `upgrade` : Allows the contract to be upgraded with a new implementation by the owner. This function is critical for maintaining the contract’s relevance and security as the underlying technology evolves.

### StrategyStandardSaleForFixedPrice Contracts
  The `StrategyStandardSaleForFixedPrice`, is designed to implement a strategy for fixed-price sales within a decentralized marketplace. It allows for the execution of orders where buyers and sellers can interact according to predefined rules, and it includes upgradability and ownership features for future modifications.

#### Contract Features and Functions :

1. **Ownership and Upgradability** :
  The contract uses components from OpenZeppelin's OwnableComponent and UpgradeableComponent. These components ensure that only the owner of the contract can update critical parameters (like fees) or upgrade the contract to a new implementation. This is managed through the OwnableImpl and OwnableInternalImpl implementations, which provide ownership-related functionalities, such as asserting ownership before performing certain actions.

2. **Protocol Fee Management** :
  The contract has a `protocol_fee` that can be initialized during the contract's deployment and later updated by the owner. This fee likely represents a percentage or fixed amount taken from each sale as a service fee for using the marketplace.

* `initializer` : This function sets the initial protocol fee and assigns the contract's owner.

* `update_protocol_fee` : Allows the owner to update the protocol fee. This function ensures that only the owner can make this change.

3. **Order Execution Logic** :
  The contract contains logic to determine whether an order can be executed based on predefined conditions, ensuring that the buyer (taker) and seller (maker) are in agreement on key parameters like price and token ID.

* `can_execute_taker_ask` : Validates whether a taker’s ask (selling request) can be matched with a maker’s bid (buying offer). It checks that the price and token ID match, and that the maker's bid is within a valid time range.

* `can_execute_taker_bid` : Similar to `can_execute_taker_ask` , but for matching a taker’s bid (buying request) with a maker’s ask (selling offer).

4. **Security and Validation** :
  The contract performs several checks to ensure the validity of orders. For example, it verifies that the order’s timing is correct (e.g., the current block timestamp is within the start and end time specified in the maker’s order) and that the price and token ID match between the buyer and seller. These checks ensure that transactions are fair and adhere to the agreed-upon terms, preventing issues like undercutting or executing orders that are no longer valid.

5. **Upgrade Mechanism** :
  The `upgrade` function allows the contract owner to upgrade the contract's implementation by providing a new class hash `impl_hash`. This feature is crucial for maintaining the contract's relevance and security over time as it allows for bug fixes, optimizations, and new features to be added without deploying a new contract.

### TransferManagementNFT
The `TransferManagerNFT` contract is designed to manage the transfer of non-fungible tokens (NFTs) within a decentralized marketplace on StarkNet. It ensures that NFT transfers are executed according to marketplace rules and ownership permissions.

#### Contracts and Features

1. **Ownership Management**:
* **Ownable Component** : The contract inherits ownership functionalities from OpenZeppelin's `OwnableComponent`. This allows only the designated owner to perform specific actions, such as initializing the contract or updating the marketplace address. The ownership logic is handled via the `OwnableImpl` and `OwnableInternalImpl` implementations.

* `Initializer` : The `initializer` function sets the contract’s marketplace address and assigns ownership. It ensures that the contract is configured correctly before any operations can take place.

2. **NFT Transfer Functionality** :
* **transfer_non_fungible_token** : This function enables the transfer of NFTs (ERC-721 tokens) from one address to another. It ensures that only the authorized marketplace contract can initiate these transfers, adding a layer of security. The function utilizes the `IERC721CamelOnlyDispatcher` to perform the token transfer, enforcing that the caller is the marketplace.

* **Secure Transfer Verification** : Before executing a transfer, the contract verifies that the caller is indeed the marketplace contract. This prevents unauthorized transfers and ensures that the transfer logic aligns with marketplace transactions.

3. **Marketplace Address Management** :
* **update_marketplace** : This function allows the contract owner to update the address of the marketplace contract. This is useful if the marketplace contract needs to be replaced or upgraded, ensuring the `TransferManagerNFT` contract remains compatible with the correct marketplace.

* `get_marketplace`: This function retrieves the current marketplace address stored in the contract. It ensures that the correct marketplace address is being used for validating transactions.


### ERC1155TransferManager
  The `ERC1155TransferManager` contract is designed to manage the secure transfer of ERC-1155 tokens by restricting transfer functionality to a specific marketplace contract. It incorporates ownership controls and is upgradeable, ensuring both security and flexibility in its deployment.

### Contracts Features
1. **Ownership and Access Control** :
    The contract implements ownership control using the OwnableComponent from OpenZeppelin. This allows only the designated owner to perform certain actions, such as updating the marketplace address. The contract includes functionality for initializing the owner and checking ownership status before executing sensitive functions.

2. **Upgradeable Contract** :
    The contract is upgradeable, utilizing the UpgradeableComponent from OpenZeppelin. This feature ensures that the contract can be upgraded or modified in the future without disrupting the existing state or functionality.

3. **NFT Transfer Management** :
    The primary function of the contract is to manage the transfer of ERC-1155 tokens. It ensures that only the marketplace contract can initiate token transfers, adding a layer of security by restricting who can call the transfer function.

4. **Event Emission** :
    The contract emits events related to ownership and upgrades, allowing off-chain systems to track important state changes, such as the transfer of ownership or contract upgrades.

### Contract Functions
1. `initializer(ref self: ContractState, marketplace: ContractAddress, owner: ContractAddress)`

    This function initializes the contract by setting the marketplace address and the owner of the contract. It is typically called once when the contract is deployed to set up the initial state.

2. `transfer_non_fungible_token(ref self: ContractState, collection: ContractAddress, from: ContractAddress, to: ContractAddress, token_id: u256, amount: u128, data: Span<felt252>)`

    This function facilitates the transfer of ERC-1155 tokens from one address to another. The function checks that the caller is the marketplace contract before proceeding with the transfer. It then interacts with the ERC-1155 token contract to execute the transfer.

3. `update_marketplace(ref self: ContractState, new_address: ContractAddress)`

    This function allows the owner to update the marketplace contract address. It ensures that only the contract owner can make this change by using the ownership assertion provided by the OwnableComponent.

4. `get_marketplace(self: @ContractState) -> ContractAddress`

    This is a simple getter function that returns the current marketplace address stored in the contract. It is used to verify the marketplace address when needed, such as during token transfers.

### TransferSelectorNFT
The `TransferSelectorNFT` contract is designed to handle the management and selection of transfer managers for ERC-721 and ERC-1155 tokens. It provides flexibility by allowing specific managers to be assigned to collections and ensures that only the correct managers are used for transfers. The contract also includes robust ownership controls and emits events for transparency in managing transfer-related operations.

### Contract Features

1. **Multi-Token Transfer Management** :
    The contract manages the transfer of different types of tokens, specifically ERC-721 and ERC-1155, by selecting the appropriate transfer manager for each token type. It ensures that the correct transfer manager is used for each token collection, making it versatile and adaptable to various NFT standards.

2. **Ownership and Access Control** :
    The contract uses an OwnableComponent to implement ownership controls. Only the owner of the contract can perform certain actions, such as updating transfer managers or modifying collection-specific transfer managers. This feature is crucial for maintaining control over sensitive operations within the contract.

3. **Interface Identification** :
    The contract stores and manages interface IDs for ERC-721 and ERC-1155 standards, which are used to determine the type of token a particular collection adheres to. This identification is essential for selecting the correct transfer manager when handling tokens.

4. **Collection-Specific Transfer Management** :
    The contract allows the owner to assign specific transfer managers to individual token collections. This feature provides flexibility by enabling different collections to use different transfer managers, which is useful for handling custom token types or specific use cases.

5. **Event Emission** :
  Events are emitted when a collection transfer manager is added or removed, providing transparency and traceability for these actions. This helps external systems and users to monitor changes in transfer management settings for collections.

#### Contract Function

1. `initializer(ref self: ContractState, transfer_manager_ERC721: ContractAddress, transfer_manager_ERC1155: ContractAddress, owner: ContractAddress)` :
   
    Initializes the contract by setting the transfer managers for ERC-721 and ERC-1155 tokens and assigns the owner. This function ensures that the contract is properly configured before any other operations can take place.

2. `add_collection_transfer_manager(ref self: ContractState, collection: ContractAddress, transfer_manager: ContractAddress)` :
   
   This function allows the contract owner to associate a specific transfer manager with a particular token collection. It ensures that only valid addresses are used and emits an event to log the addition of the transfer manager.

3. `remove_collection_transfer_manager(ref self: ContractState, collection: ContractAddress)` :
   
   Allows the owner to remove the transfer manager associated with a specific collection. This function is useful when a collection no longer requires a custom transfer manager or when it needs to be reassigned. It also logs the removal via an event.

4. `update_TRANSFER_MANAGER_ERC721(ref self: ContractState, manager: ContractAddress)` :

    This function lets the owner update the global transfer manager for ERC-721 tokens. It is essential for maintaining or upgrading the transfer mechanism for ERC-721 tokens across the platform.

5. `update_TRANSFER_MANAGER_ERC1155(ref self: ContractState, manager: ContractAddress)` :

    Similar to the ERC-721 update function, this allows the owner to update the transfer manager for ERC-1155 tokens, ensuring the platform can handle changes or improvements in ERC-1155 token transfer logic.

6. `get_INTERFACE_ID_ERC721(self: @ContractState) -> felt252`
   `get_INTERFACE_ID_ERC1155(self: @ContractState) -> felt252` :

    These getter functions return the interface IDs for ERC-721 and ERC-1155, respectively. They are used internally to determine the token standard of a collection when selecting the appropriate transfer manager.


7. `get_TRANSFER_MANAGER_ERC721(self: @ContractState) -> ContractAddress`
   `get_TRANSFER_MANAGER_ERC1155(self: @ContractState) -> ContractAddress` :

    These functions return the current transfer manager addresses for ERC-721 and ERC-1155 tokens, respectively. They are vital for ensuring the correct manager is used when transferring tokens of these standards.

8. `get_transfer_manager_selector_for_collection(self: @ContractState, collection: ContractAddress) -> ContractAddress`
   `check_transfer_manager_for_token(self: @ContractState, collection: ContractAddress) -> ContractAddress` :

    The first function retrieves the transfer manager assigned to a specific collection, while the second checks and returns the appropriate transfer manager for a collection, considering both the assigned manager and the token standard. This logic ensures that the correct manager is used for each transfer operation.

#### Overview
![overview](./assets/marketplace-overview.png)

#### Listing
![listing](./assets/marketplace-listing.png)

#### Buy
![buy](./assets/marketplace-buy.png)

#### Make Offer
![make-offer](./assets/marketplace-make-offer.png)

#### Accept Offer
![accept-offer](./assets/marketplace-accept-offer.png)