# Solana NFT Minting Program

A Solana smart contract (program) for minting NFTs with tiered access control and pricing. Built with Anchor framework and Metaplex for NFT metadata management.

## Features

- **Multi-tier Minting System**
  - OG (Original) List - Exclusive early access
  - Whitelist (WL) - Early access for whitelisted users
  - Public Sale - Open to everyone
  
- **Flexible Pricing**
  - Configurable pricing for each tier (OG, WL, Public)
  - Dynamic price updates by admin
  
- **Supply Management**
  - Max supply cap
  - Per-tier quantity limits
  - Per-user minting limits for each tier
  
- **Access Control**
  - Admin-controlled minting stages
  - Whitelist and OG list management
  - Stage-based activation (disabled/OG-WL/public)

- **NFT Standards**
  - SPL Token standard
  - Metaplex metadata
  - Master Edition NFTs

## Prerequisites

- [Node.js](https://nodejs.org/) (v14 or higher)
- [Anchor](https://www.anchor-lang.com/) (v0.19.0)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)
- [Rust](https://www.rust-lang.org/tools/install)

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd solana-nft-mint
```

2. Install dependencies:
```bash
npm install
```

3. Build the program:
```bash
anchor build
```

4. Configure your Solana wallet:
```bash
solana config set --url devnet
solana-keygen new  # if you don't have a wallet
```

## Configuration

Update `Anchor.toml` with your program ID and wallet path:

```toml
[programs.devnet]
wallet_nft_mint = "YOUR_PROGRAM_ID"

[provider]
cluster = "devnet"
wallet = "~/.config/solana/id.json"
```

## Program Structure

### Main Instructions

#### `initialize`
Initializes the minting account with configuration parameters.

**Parameters:**
- `authorized_creator`: Pubkey of the authorized creator
- `max_supply`: Maximum total NFTs that can be minted
- `og_max`: Max NFTs per user in OG phase
- `wl_max`: Max NFTs per user in whitelist phase
- `public_max`: Max NFTs per user in public phase
- `og_price`: Price in lamports for OG minting
- `wl_price`: Price in lamports for whitelist minting
- `public_price`: Price in lamports for public minting

#### `add_og_list`
Add a user to the OG (Original) list.

**Parameters:**
- `user`: Pubkey of the user to add

#### `remove_og_list`
Remove a user from the OG list.

#### `add_wl_list`
Add a user to the whitelist.

**Parameters:**
- `user`: Pubkey of the user to add

#### `remove_wl_list`
Remove a user from the whitelist.

#### `update_price`
Update pricing for different tiers (admin only).

**Parameters:**
- `new_og_price`: New price for OG minting
- `new_wl_price`: New price for whitelist minting
- `new_public_price`: New price for public minting

#### `update_amount`
Update minting limits for different tiers (admin only).

**Parameters:**
- `new_og_amount`: New max for OG users
- `new_wl_amount`: New max for whitelist users
- `new_public_amount`: New max for public users

#### `set_stage`
Set the current minting stage (admin only).

**Parameters:**
- `new_stage`: 
  - `0` = Disabled
  - `1` = OG/Whitelist phase
  - `2` = Public phase

#### `set_uri`
Set the base URI for NFT metadata (admin only).

**Parameters:**
- `new_uri`: Base URI string (e.g., "https://yourdomain.com/metadata/")

#### `mint_nft`
Mint an NFT to a user.

**Parameters:**
- `creator_key`: Pubkey of the creator
- `title`: Title of the NFT

## Usage Examples

### Initialize the Program

```typescript
const [mintingPubkey] = await anchor.web3.PublicKey.findProgramAddress(
  [Buffer.from('wallet_nft_minting')],
  program.programId
);

await program.methods
  .initialize(
    authorizedCreator,
    new anchor.BN(10000),  // max supply
    new anchor.BN(5),      // OG max per user
    new anchor.BN(3),      // WL max per user
    new anchor.BN(1),      // Public max per user
    new anchor.BN(1e9),    // OG price (1 SOL)
    new anchor.BN(2e9),    // WL price (2 SOL)
    new anchor.BN(3e9)     // Public price (3 SOL)
  )
  .accounts({
    mintingAccount: mintingPubkey,
    initializer: wallet.publicKey,
    systemProgram: SystemProgram.programId,
    tokenProgram: TOKEN_PROGRAM_ID,
    rent: anchor.web3.SYSVAR_RENT_PUBKEY,
  })
  .rpc();
```

### Add User to Whitelist

```typescript
await program.methods
  .addWlList(userPublicKey)
  .accounts({
    admin: adminWallet.publicKey,
    mintingAccount: mintingPubkey,
    wlList: whitelistPDA,
    systemProgram: SystemProgram.programId,
    tokenProgram: TOKEN_PROGRAM_ID,
    rent: anchor.web3.SYSVAR_RENT_PUBKEY,
  })
  .rpc();
```

### Set Minting Stage

```typescript
const [mintingPubkey, bump] = await anchor.web3.PublicKey.findProgramAddress(
  [Buffer.from('wallet_nft_minting')],
  program.programId
);

await program.methods
  .setStage(bump, 1)  // Enable OG/WL phase
  .accounts({
    mintingAccount: mintingPubkey,
    admin: adminWallet.publicKey,
  })
  .rpc();
```

### Mint an NFT

```typescript
const mintKey = anchor.web3.Keypair.generate();
const NftTokenAccount = await getAssociatedTokenAddress(
  mintKey.publicKey,
  wallet.publicKey
);

const metadataAddress = await getMetadata(mintKey.publicKey);
const masterEdition = await getMasterEdition(mintKey.publicKey);

await program.methods
  .mintNft(creatorPublicKey, 'My NFT Title')
  .accounts({
    mintAuthority: wallet.publicKey,
    mint: mintKey.publicKey,
    tokenAccount: NftTokenAccount,
    tokenProgram: TOKEN_PROGRAM_ID,
    metadata: metadataAddress,
    tokenMetadataProgram: TOKEN_METADATA_PROGRAM_ID,
    payer: wallet.publicKey,
    owner: ownerPublicKey,
    mintingAccount: mintingPubkey,
    wlList: whitelistPDA,
    ogList: ogListPDA,
    userMintingCounterAccount: userCounterPDA,
    systemProgram: SystemProgram.programId,
    rent: anchor.web3.SYSVAR_RENT_PUBKEY,
    masterEdition: masterEdition,
  })
  .rpc();
```

## Testing

Run the test suite:

```bash
anchor test
```

Run tests on devnet:

```bash
anchor test --provider.cluster devnet
```

## Deployment

### Deploy to Devnet

```bash
anchor build
anchor deploy --provider.cluster devnet
```

### Deploy to Mainnet

1. Update `Anchor.toml` cluster to mainnet:
```toml
[provider]
cluster = "mainnet-beta"
```

2. Deploy:
```bash
anchor build
anchor deploy --provider.cluster mainnet-beta
```

## Program Architecture

### Account Structures

#### MintingAccount
Main program configuration account.

```rust
pub struct MintingAccount {
    pub admin_key: Pubkey,              // Admin public key
    pub aury_vault: Pubkey,             // Vault address
    pub authorized_creator: Pubkey,     // Authorized creator
    pub max_supply: u64,                // Maximum supply
    pub og_max: u64,                    // OG max per user
    pub wl_max: u64,                    // Whitelist max per user
    pub public_max: u64,                // Public max per user
    pub og_price: u64,                  // OG price in lamports
    pub wl_price: u64,                  // WL price in lamports
    pub public_price: u64,              // Public price in lamports
    pub cur_num: u64,                   // Current minted count
    pub freeze_program: bool,           // Emergency freeze
    pub cur_stage: i8,                  // Current minting stage
    pub base_uri: String,               // Base URI for metadata
}
```

#### WhiteList / OriginalList
User access control accounts.

```rust
pub struct WhiteList {
    user: Pubkey,
    minting_account: Pubkey,
    initializer: Pubkey,
    count: u64
}
```

#### UserMintingAccount
Tracks per-user minting count.

```rust
pub struct UserMintingAccount {
    pub cur_num: u64,
}
```

## Security Considerations

- ✅ Admin access control on sensitive operations
- ✅ Stage-based minting validation
- ✅ Supply cap enforcement
- ✅ Per-user minting limits
- ✅ Payment verification
- ✅ PDA-based account security

## Error Codes

| Error | Description |
|-------|-------------|
| `NotAllowed` | User doesn't have permission or limit reached |
| `NotActive` | Minting is currently disabled |
| `InvalidStage` | Invalid stage number provided |
| `InsufficientFunds` | User doesn't have enough SOL |

## Program ID

**Devnet:** `4qLzEGU2KaBggBkVgELQAU8wayPMTGa9EwxGWwzKRmKT`

## Dependencies

- `@project-serum/anchor` - Anchor framework
- `@metaplex/js` - Metaplex JavaScript SDK
- `@solana/spl-token` - SPL Token library
- `mpl-token-metadata` - Metaplex Token Metadata program
