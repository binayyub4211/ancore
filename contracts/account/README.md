# Ancore Account Contract

Core smart account contract implementing account abstraction for Stellar/Soroban.

## Overview

The Ancore Account contract is the foundation of the Ancore smart wallet system. It provides:

- **Programmable Validation**: Custom signature validation logic
- **Session Keys**: Time-limited signing permissions
- **Upgradeability**: Safe upgrade mechanisms
- **Batching**: Execute multiple operations atomically

## Security

⚠️ **CRITICAL**: This contract handles user funds and must be thoroughly audited.

### Security Properties

1. **Owner Control**: Only the owner can execute transactions
2. **Nonce Protection**: Prevents replay attacks
3. **Session Keys**: Time-limited, permission-scoped keys
4. **Upgrade Safety**: Owner-controlled upgrades

### Audit Status

- [ ] Internal review
- [ ] External audit (Trail of Bits / OpenZeppelin)
- [ ] Formal verification
- [ ] Bug bounty

## Building

```bash
cd contracts/account
cargo build --target wasm32-unknown-unknown --release
```

Or using soroban-cli:

```bash
soroban contract build
```

## Testing

```bash
cargo test
```

## Deployment

### Testnet

```bash
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/ancore_account.wasm \
  --network testnet
```

### Mainnet

See `docs/contracts/DEPLOYMENT.md` for mainnet deployment instructions.

## Contract Interface

### Initialize

```rust
fn initialize(env: Env, owner: Address)
```

Initialize the account with an owner address.

### Execute

```rust
fn execute(
    env: Env,
    to: Address,
    function: Symbol,
    args: Vec<Val>,
) -> bool
```

Execute a transaction on behalf of the account.

### Session Keys

```rust
fn add_session_key(
    env: Env,
    public_key: BytesN<32>,
    expires_at: u64,
    permissions: Vec<u32>,
)

fn revoke_session_key(env: Env, public_key: BytesN<32>)

fn get_session_key(env: Env, public_key: BytesN<32>) -> Option<SessionKey>
```

Manage session keys for the account.

## Contract Errors

The contract uses structured error codes to provide clear feedback for failure conditions. These error codes are essential for SDK and frontend error handling.

### Error Code Reference

| Error Code | Variant | Description |
|------------|---------|-------------|
| 1 | `AlreadyInitialized` | Account is already initialized |
| 2 | `NotInitialized` | Account is not initialized |
| 3 | `Unauthorized` | Caller is not authorized |
| 4 | `InvalidNonce` | Invalid nonce provided |
| 5 | `SessionKeyNotFound` | Session key not found |
| 6 | `SessionKeyExpired` | Session key has expired |
| 7 | `InsufficientPermission` | Insufficient permissions |
| 8 | `InvalidVersion` | Invalid version provided for migration |

### Error Handling Examples

#### SDK Integration
```rust
match contract_result {
    Ok(result) => handle_success(result),
    Err(ContractError::InvalidNonce) => {
        // Handle nonce mismatch - refresh nonce and retry
    },
    Err(ContractError::SessionKeyExpired) => {
        // Handle expired session key - request new key
    },
    Err(error) => {
        // Handle other errors
        eprintln!("Contract error: {:?}", error);
    }
}
```

#### Frontend Integration
```javascript
try {
    await contract.execute(params);
} catch (error) {
    switch (error.code) {
        case 4: // InvalidNonce
            showNonceError();
            break;
        case 6: // SessionKeyExpired
            showSessionKeyExpiredError();
            break;
        case 7: // InsufficientPermission
            showPermissionError();
            break;
        default:
            showGenericError(error.message);
    }
}
```

## Development

### Prerequisites

- Rust toolchain (1.74.0+)
- Soroban CLI: `cargo install --locked soroban-cli`
- wasm32-unknown-unknown target: `rustup target add wasm32-unknown-unknown`

### Local Development

```bash
# Build
cargo build

# Test
cargo test

# Build optimized WASM
cargo build --target wasm32-unknown-unknown --release
```

## License

Apache-2.0
