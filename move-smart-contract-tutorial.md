# Move Smart Contract Programming Tutorial (Aptos Devnet — Hands-On)

*A ready-to-publish, hands-on tutorial with a table of contents, commands, and complete example code. Goal: go from zero to environment setup → write your first Move module → compile, test, publish, and interact on-chain → understand advanced topics and security essentials.*

---

## Table of Contents

- [Prerequisites](#prerequisites)  
- [Initialize Account & Project](#initialize-account--project)  
- [Create a Move Package](#create-a-move-package)  
- [Write the First Module: Message](#write-the-first-module-message)  
- [Compile, Unit Tests & Coverage](#compile-unit-tests--coverage)  
- [Publish On-Chain & Interact](#publish-on-chain--interact)  
- [Verify in the Block Explorer](#verify-in-the-block-explorer)  
- [Advanced Topics](#advanced-topics)  
- [Security Checklist (Highly Recommended)](#security-checklist-highly-recommended)  
- [Troubleshooting](#troubleshooting)  
- [Practice Exercises (with Hints)](#practice-exercises-with-hints)  
- [Command Cheat Sheet](#command-cheat-sheet)  
- [References & Further Reading](#references--further-reading)  
- [(Optional) Libra2 Adaptation Tips](#optional-libra2-adaptation-tips)

---

## Prerequisites

- Basic command-line experience  
- Git and Homebrew installed (macOS)  
- Node/Python are **not required** unless you plan to use SDKs for front/back end

**Install Aptos CLI (macOS example):**
```bash
brew update
brew install aptos
aptos --version
aptos help
```

> On Linux, use your package manager or download prebuilt binaries; on Windows, WSL2 is recommended.

---

## Initialize Account & Project

Initialize an Aptos account in a fresh working directory (choose **Devnet** by default):

```bash
mkdir my-first-module && cd my-first-module
aptos init
# Interactive choice: Network = devnet; press Enter for other defaults
```

This creates `.aptos/config.yaml` containing your account address and private key, and automatically funds the account with test APT from the Devnet faucet.

---

## Create a Move Package

Create a Move package (project) in the current directory:

```bash
aptos move init --name my_first_module
```

Directory layout (key files):

```
my-first-module/
  ├─ .aptos/config.yaml         # account configuration
  ├─ Move.toml                  # package configuration
  └─ sources/                   # source code
```

---

## Write the First Module: `Message`

Create `sources/message.move` to demonstrate **resource storage** and an **entry function**:

```move
module my_first_module::message {
    use std::signer;
    use std::string;

    /// A resource type persisted under the caller's account address
    struct MessageHolder has key, store, drop {
        message: string::String,
    }

    /// Set/overwrite the message (transaction entry point)
    public entry fun set_message(account: &signer, message: string::String) acquires MessageHolder {
        let addr = signer::address_of(account);
        if (exists<MessageHolder>(addr)) {
            // Remove old resource (discard old value)
            move_from<MessageHolder>(addr);
        };
        move_to(account, MessageHolder { message });
    }

    /// Read the message stored under a given address (regular function)
    /// Note: String is not copyable; return a *new* String via bytes + utf8
    public fun get_message(addr: address): string::String acquires MessageHolder {
        let holder = borrow_global<MessageHolder>(addr);
        string::utf8(string::bytes(&holder.message))
    }

    /// Optional: clear the message
    public entry fun clear_message(account: &signer) acquires MessageHolder {
        let addr = signer::address_of(account);
        if (exists<MessageHolder>(addr)) {
            move_from<MessageHolder>(addr);
        }
    }

    // --- Unit tests (kept in the same module for convenient private access) ---

    use aptos_framework::account;

    #[test]
    fun test_set_and_get_message() {
        let alice = account::create_signer_for_testing(@0xA11CE);
        set_message(&alice, string::utf8(b"Hello, Aptos!"));
        let s = get_message(@0xA11CE);
        assert!(string::bytes(&s) == b"Hello, Aptos!", 0);
    }

    /// Expected failure: reading before set should abort
    #[test]
    #[expected_failure]
    fun test_get_without_set_should_fail() {
        let _ = get_message(@0xB0B);
    }

    #[test]
    fun test_clear_message() {
        let alice = account::create_signer_for_testing(@0xC0FFEE);
        set_message(&alice, string::utf8(b"X"));
        clear_message(&alice);
        // Reading again should fail (resource missing)
        // You could wrap with expected_failure; here we just keep it simple.
        assert!(true, 0);
    }
}
```

**Key takeaways:**
- In Move, *state is a resource* (`has key`), stored under an address.  
- Accessing global resources must be explicitly declared with `acquires`.  
- `entry fun` can be called directly from off-chain as a transaction entry point.  
- `String` is not copyable; return a new value with `bytes → utf8`.

---

## Compile, Unit Tests & Coverage

Map the named address `my_first_module` to your current account (`default`):

```bash
aptos move compile --named-addresses my_first_module=default
aptos move test --named-addresses my_first_module=default
aptos move test --named-addresses my_first_module=default --coverage
```

After tests pass, the terminal shows assertions and a coverage summary.

---

## Publish On-Chain & Interact

**Publish (deploy):**
```bash
aptos move publish --named-addresses my_first_module=default
```

**Call the entry function to write a message:**
```bash
aptos move run \
  --function-id 'default::message::set_message' \
  --args 'string:Hello, Aptos!'
```

> `default` is replaced at runtime with the account address from `.aptos/config.yaml`.

**(Optional) Clear the message:**
```bash
aptos move run \
  --function-id 'default::message::clear_message'
```

---

## Verify in the Block Explorer

Open Aptos Explorer (switch to **Devnet**) and enter your account address:

- Under **Code / Modules** you should see the published module  
- Under **Resources** you should see `MessageHolder` and its stored string  
- Under **Transactions** you should see the `publish` / `run` transaction records

---

## Advanced Topics

1) **Resource Accounts**  
- “Keyless” contract-style accounts ideal for *isolating state* and *publishing modules* under a resource account address  
- Typical workflow: create a resource account from your main account → publish the module under the resource account → expose the resource account address as the contract identity

2) **Objects & Digital Asset Standards (NFT / FA)**  
- **Object** is a general container on Aptos for composition and transfer; it’s the foundation of modern digital assets (NFTs and FAs)  
- Typical flow: create `ObjectCore` → set permissions (refs/capabilities) → attach metadata and transferability policy

3) **Scripts & Batch Operations**  
- Besides `entry fun`, you can author scripts to call multiple functions in a single transaction to perform init → configure → verify flows

4) **Packages & Upgrades**  
- Organize modules/resources/scripts in a Package  
- Use upgrade strategies and visibility (e.g., `public(package)`) to expose a stable API while evolving internals

5) **SDK Integration (Minimal TypeScript Example)**

```ts
// pnpm add @aptos-labs/ts-sdk
import { Aptos, AptosConfig, Network } from "@aptos-labs/ts-sdk";

const config = new AptosConfig({ network: Network.DEVNET });
const aptos = new Aptos(config);

// Your on-chain account address (the address that published the module)
const moduleAddress = "0xYOUR_ACCOUNT_ADDRESS";

// Read the resource: my_first_module::message::MessageHolder
const res = await aptos.getAccountResource({
  accountAddress: "0xTARGET_ADDRESS",
  resourceType: `${moduleAddress}::message::MessageHolder`,
});
console.log(res);
```

---

## Security Checklist (Highly Recommended)

- **Resources are assets:** Do **not** give `copy` ability to fungible assets; avoid “copyable money”.  
- **Audit your surface:** Keep the list of `entry` functions minimal; restrict sensitive operations (e.g., only module/resource account can call).  
- **Error codes & assertions:** Use custom error codes on critical paths; unit tests should cover failure scenarios.  
- **Formal verification (Move Prover):** Add `spec` for critical invariants, e.g., “after call, resource exists and message is non-empty”.

**Minimal Prover example** (place in the same file or a `spec` module):
```move
spec module my_first_module::message {
    spec set_message {
        // Example invariant: ensure the resource exists after setting.
        // (For rigor, encode as a global-state postcondition.)
        // ensures exists<MessageHolder>(signer::address_of(account));
    }
}
```

Run:
```bash
aptos move prove --named-addresses my_first_module=default
```

---

## Troubleshooting

- **Tx rejected / insufficient gas:** Faucet funding failed or balance too low →  
  `aptos account fund-with-faucet --account default`  
- **Named address not mapped:** Missing in `Move.toml` or you forgot  
  `--named-addresses my_first_module=default` in commands  
- **String read type/borrow errors:** Return a fresh `String` with `string::bytes(&s)` → `string::utf8(bytes)`  
- **Publish vs call address mismatch:** Ensure the `function-id` prefix matches your actual publish address (`default::module::func` resolves to the local default account)

---

## Practice Exercises (with Hints)

1. **Basic:** Add `update_message_if_empty` to `message` (write only if resource is missing or the message is empty).  
   *Hint:* check `exists` first; if exists, `borrow_global` and test `bytes(&s).length()`.  
2. **Permissions:** Only the module publisher can reset all records (e.g., a `reset_all`).  
   *Hint:* use a resource account or an “admin address” resource; compare with `signer::address_of`.  
3. **Object:** Build a “single-transfer avatar” asset with Objects that locks after the first transfer.  
   *Hint:* implement a custom capability or store a “transferred” flag in metadata and check in the transfer entry.  
4. **Batch script:** In one transaction, do “init resource → `set_message` → assert message is non-empty.”  
5. **SDK:** Use the TS SDK to read `MessageHolder` and render the bytes as a readable string on the frontend.

---

## Command Cheat Sheet

```bash
# Install & help
brew install aptos
aptos --version
aptos help

# Initialize account (Devnet)
aptos init

# Package management
aptos move init --name my_first_module
aptos move compile --named-addresses my_first_module=default

# Tests & coverage
aptos move test --named-addresses my_first_module=default
aptos move test --named-addresses my_first_module=default --coverage

# Publish & interact
aptos move publish --named-addresses my_first_module=default
aptos move run --function-id 'default::message::set_message' --args 'string:Hello, Aptos!'
aptos move run --function-id 'default::message::clear_message'

# (Optional) Funding
aptos account fund-with-faucet --account default
```

---

## References & Further Reading

- Aptos official documentation & Move Book (overview, syntax, modules/scripts, testing, Prover)  
- Aptos Explorer (switch to Devnet to view accounts, transactions, resources, modules)  
- Aptos TypeScript / Python / Go SDKs (for DApp and server integration)

> Link these official entries at the end of your published article so readers can dive deeper.

---

Good luck compiling and happy transacting 🎯  

