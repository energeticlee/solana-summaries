# Hawk Token Staking Documentation
[Github](https://github.com/hawksightco/hawk-staking)

[Loom video](Hawk%20Token%20Staking%20Documentation%205bb1a4e4437a4a5b84469f0f4efbc8e1/screen-capture_(1).mp4)

[High level breakdown](https://medium.com/openhaus/solana-staking-program-breakdown-43f28270b94f)

# HS_Dapp Token Staking Doc

**Root Path:** HS-DAPP/web/src

## Program

**Link:** [https://github.com/hawksightco/hawk-staking/blob/main/programs/step-staking/src/lib.rs](https://github.com/hawksightco/hawk-staking/blob/main/programs/step-staking/src/lib.rs)

### IDL File

**Path:** configs/token_staking.json

### Instruction
[ `initialize`, `updateLockEndDate`, `toggleFreezeProgram`, `stake`, `unstake`, `emitPrice`, `emitReward` ]
1. `initialize`
    1. Initializes an empty token account account for the `token-staking` program
    2. Accounts
        1. `token_mint`
        2. `token_vault`
        3. `staking_account`
        4. `initalizer` : `payer`
        5. `system_program`
        6. `token_program`
        7. `rent_sysvar_account`
    3. Arguments (non-account)
        1. `_nonce_vault`
        2. `_nonce_staking`
        3. `lock_end_date` : `u64` - set lock end date of staking_account
    4. Account Validation
        1. `token_mint `: `Mint` address to be constants::TOKEN_MINT_PUBKEY
        2. `token_vault`: `TokenAccount`
              1. token::mint = `token_mint`
              2. token::authority = `token_vault` : the PDA address is both the vault account and the authority
              3. seeds = `constants::TOKEN_MINT_PUBKEY`
              4. bump = `_nonce_vault`
        3. `staking_account` : `StakingAccount`
              1. payer = `initializer`
              2. seeds = `constants::TOKEN_MINT_PUBKEY`
              2. bump = `_nonce_staking`
        4. `initalizer` : `payer`
    5. Handling Logic
        1. Creates / initializes the staking account
        2. `staking_account.initializer_key` assign to `initializer.key`
        3. Set `staking_account.lock_end_date` to `lock_end_date`
        4. Staking account is created with the necessary rent balance transferred from the payer to the staking account (PDA)
2. `updateLockEndDate`
    1. Update lock end date of `staking account`
    2. Accounts
        1. `initalizer` : `Signer`
        2. `staking_account` : `StakingAccount`
    3. Arguments (non-account)
        1. `_nonce_staking`
        3. `new_lock_end_date` : `u64`
    4. Account Validation
        1. `initalizer`
        2. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
              2. bump = `_nonce_staking`
              3. constraint = `staking_account.initializer_key` == `initializer.key`
    5. Handling Logic
        1. Set `staking_account.lock_end_date` to `new_lock_end_date`
3. `toggleFreezeProgram`
    1. Toogle freeze of `staking account`
    2. Accounts
        1. `initalizer` : `Signer`
        2. `staking_vault` : `StakingAccount`
    3. Arguments (non-account)
        1. `_nonce_staking`
    4. Account Validation
        1. `initalizer` : `Signer`
        2. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
              2. bump = `_nonce_staking`
              3. constraint = `staking_account.initializer_key` == `initializer.key`
    5. Handling Logic
        1. Set `staking_account.lock_end_date` to `!staking_account.lock_end_date`
4. `stake`
    1. User stake token in exchange for xToken
    2. Accounts
        1. `token_mint` : `Mint`
        2. `token_from` : `TokenAccount`
        3. `token_from_authority` : `Signer`
        4. `token_vault` : `TokenAccount`
        5. `staking_account` : `StakingAccount`
        6. `user_staking_account` : `UserStakingAccount`
        7. `system_program`
        8. `token_program`
        9. `rent_sysvar_account`
    3. Arguments (non-account)
        1. `_nonce_vault`
        2. `_nonce_staking`
        3. `_nonce_user_staking`
        4. `amount` : `u64`
    4. Account Validation
        1. `token_mint `: `Mint` address to be constants::TOKEN_MINT_PUBKEY
        2. `token_from_authority` : `Signer`
        3. `token_vault`:
              1. seeds = `token_mint.key()`
              2. bump = `_nonce_vault`
        3. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
              2. bump = `_nonce_staking`
              3. constraint = `!staking_account.freeze_program`
        4. `user_staking_account`:
              1. init_if_needed
              2. payer = `token_from_authority`
              3. seeds = `token_from_authority`
              4. bump = `_nonce_user_staking`
    5. Handling Logic
        1. if `staking_account` or `token_vault` is empty: token to xToken exchange rate is 1:1.
        2. else: calculate exchange rate then update `user_staking_accoung` & `staking_account` xToken amount
        3. Transfer token from `token_from` to `token_vault`
        4. Update `user_staking_accoung` token amount
5. `unstake`
    1. User unstake xToken in exchange for Token
    2. Accounts
        1. `token_mint` : `Mint`
        2. `x_token_from_authority` : `Signer`
        3. `token_vault` : `TokenAccount`
        4. `staking_account` : `StakingAccount`
        5. `user_staking_account` : `UserStakingAccount`
        6. `token_to` : `TokenAccount`
        7. `token_program`
    3. Arguments (non-account)
        1. `nonce_vault`
        2. `_nonce_staking`
        3. `_nonce_user_staking`
        4. `amount` : `u64`
    4. Account Validation
        1. `token_mint `: `Mint` address to be constants::TOKEN_MINT_PUBKEY
        2. `x_token_from_authority` : `Signer`
        3. `token_vault`:
              1. seeds = `token_mint.key()`
              2. bump = `_nonce_vault`
        3. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
              2. bump = `_nonce_staking`
              3. constraint = !staking_account.freeze_program
        4. `user_staking_account`:
              1. seeds = `x_token_from_authority`
              2. bump = `_nonce_user_staking`
              3. constraint = `user_staking_account.x_token_amount >= amount`
    5. Handling Logic
        1. Return error if lock_end_date have not pass.
        2. Reduce unstake xToken amount from `staking_account` & `user_staking_account`
        3. Calculate share of token from token_vault
        4. Compute vault signer seed and transfer token from `token_vault` to `user`
        5. Calculate user_staking_account
6. `emitPrice`
    1. Emit latest price
    2. Accounts
        1. `token_mint` : `Mint`
        2. `token_vault` : `TokenAccount`
        3. `staking_account` : `StakingAccount`
    3. Arguments (non-account)
        1. `Null`
    4. Account Validation
        1. `token_mint `: `Mint` address to be constants::TOKEN_MINT_PUBKEY
        2. `token_vault`:
              1. seeds = `token_mint.key()`
        3. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
    5. Handling Logic
        1. Emit Latest Price
7. `emitReward`
    1. Emit latest reward
    2. Accounts
        1. `token_mint` : `Mint`
        2. `token_vault` : `TokenAccount`
        3. `staking_account` : `StakingAccount`
        4. `token_from_authority` : `AccountInfo`
        5. `user_staking_account` : `UserStakingAccount`
    3. Arguments (non-account)
        1. `Null`
    4. Account Validation
        1. `token_mint `: `Mint` address to be constants::TOKEN_MINT_PUBKEY
        2. `token_vault`:
              1. seeds = `token_mint.key()`
        3. `staking_account`
              1. seeds = `constants::STAKING_PDA_SEED`
        4. `user_staking_account`
              1. seeds = `token_from_authority.key()`
    5. Handling Logic
        1. Emit Latest Reward

### Program Accounts

[stakingAccount, userStakingAccount]

## Views

**Path:** views/tokenStaking/index.tsx

### Description

The main token staking page, excluding the top and left nav bar, and bottom footer.

All UI wrappers are defined in this file.

### State

**userHawkVault:** To store the amount of hawk in the user staking vault

## Context

**Path:** contexts/tokenStaking.tsx

### Description

This file contains all logic and the token mint address which is hard-coded

### Functions

**getUserVaultAmount:** return the total amount of hawk staked by the user.

**getUserWalletAmount:** return the total amount of hawk in the user wallet.

**handleStaking:** send stake RPC request, update frontend user vault state, and close modal

**handleUnstake:** send unstake RPC request, update the frontend user vault state, and close modal

## Components

**Path:** components/modals/TokenStaking/TokenStaking.tsx

### Description

Handle modal UI, collect user info, and call RPC function.

### Global Modal State

**useGlobalModalContext:** modal UI

**Path:** contexts/modalmanager

### Base Modal State

**BaseModal:** Base modal component

**getAccentColorNew:** return color base on props

**Status:** enum of different value which is used to get a color from getAccentColorNew

**Path:** components/modals/BaseModal/index.ts

### State

**anxLoading:** ??

**activeStaketype:** handle which function to trigger, enum [ “CLAIM”, “STAKE”, “UNSTAKE” ]

**userHawkWallet:** the amount of hawk in user wallet

**amount:** user input amount
