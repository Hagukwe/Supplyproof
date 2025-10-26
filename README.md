# SupplyProof — STX Supply Chain Tracking Contract

> **A student-friendly Clarity smart contract for supply chain product tracking with ownership transfers and checkpoint history.**

---

## 📦 What This Project Is

**SupplyProof** is a Clarity smart contract that tracks physical products through a supply chain. Each product is registered once, and then successive checkpoints (transfers) are recorded with:
- Owner (principal)
- Status
- Location
- Timestamp
- Notes

**Purpose:** Learn how to design state and checkpoint history using Clarity's maps and variables, and test contracts with Clarinet/Vitest.

---

## 📁 Repository Layout

| File/Folder | Description |
|------------|-------------|
| `contracts/supply-proof.clar` | Main Clarity contract implementing product registration and transfer logic |
| `tests/supply-proof.test.ts` | Vitest tests using Clarinet SDK |
| `package.json` | Scripts and dependencies (Vitest, Clarinet SDK) |
| `Clarinet.toml` | Clarinet configuration file |
| `settings/` | Network configuration files (Devnet, Testnet, Mainnet) |

---

## 🧠 Contract Overview (For Learners)

### Data Structures

#### 1. **products** map
- **Key:** `{ product-id: (string-ascii 36) }`
- **Value:** Product record with:
  - `name` — product name
  - `manufacturer` — original registrant
  - `current-owner` — current owner principal
  - `status` — current status (e.g., "registered", "transferred")
  - `location` — current location
  - `timestamp` — last update block height
  - `is-active` — boolean flag

#### 2. **product-checkpoints** map
- **Key:** `{ product-id: (string-ascii 36), checkpoint-index: uint }`
- **Value:** Checkpoint history entry with:
  - `handler` — who performed the action
  - `previous-owner` — owner before transfer
  - `new-owner` — owner after transfer
  - `status` — checkpoint status
  - `location` — location at checkpoint
  - `timestamp` — block height
  - `notes` — custom notes (up to 500 chars)

#### 3. **checkpoint-counters** map
- **Key:** `{ product-id: (string-ascii 36) }`
- **Value:** `{ count: uint }` — tracks number of checkpoints

#### 4. **total-products** variable
- Tracks the total number of products registered

---

### Error Constants

| Error | Code | Description |
|-------|------|-------------|
| `ERR-NOT-AUTHORIZED` | u101 | Caller is not authorized to perform action |
| `ERR-PRODUCT-NOT-FOUND` | u102 | Product ID does not exist |
| `ERR-PRODUCT-EXISTS` | u103 | Product ID already registered |
| `ERR-INVALID-OWNER` | u104 | Invalid owner specified |
| `ERR-INVALID-STATUS` | u105 | Invalid status provided |

---

### Public Functions

#### `register-product`
```clarity
(define-public (register-product
  (product-id (string-ascii 36))
  (name (string-ascii 100))
  (initial-location (string-ascii 100))
))
```
- **Purpose:** Register a new product in the supply chain
- **Caller becomes:** manufacturer and initial owner
- **Creates:** 
  - Product record
  - First checkpoint (index 1)
  - Initializes counters
- **Returns:** `(ok true)` or `ERR-PRODUCT-EXISTS`

#### `transfer-ownership`
```clarity
(define-public (transfer-ownership
  (product-id (string-ascii 36))
  (new-owner principal)
  (new-location (string-ascii 100))
  (notes (string-ascii 500))
))
```
- **Purpose:** Transfer product ownership to a new principal
- **Authorization:** Only the current owner can call this
- **Updates:**
  - Product record (new owner, location, timestamp, status)
  - Adds new checkpoint entry
  - Increments checkpoint counter
- **Returns:** `(ok true)`, `ERR-NOT-AUTHORIZED`, or `ERR-PRODUCT-NOT-FOUND`

---

### Read-Only Functions

| Function | Parameters | Returns | Description |
|----------|-----------|---------|-------------|
| `get-product` | `product-id` | `(optional product-record)` | Fetch product details |
| `get-checkpoint` | `product-id`, `checkpoint-index` | `(optional checkpoint-record)` | Fetch specific checkpoint |
| `get-checkpoint-count` | `product-id` | `{ count: uint }` | Get number of checkpoints |
| `get-product-owner` | `product-id` | `(optional principal)` | Get current owner |
| `get-total-products` | — | `uint` | Get total registered products |

---

## 🎓 Why This Structure?

- **Maps** model on-chain records and history with compound keys
- **Checkpoint counters** enable ordered, collision-free checkpoint appending
- **Explicit error constants** make testing clearer and debugging easier
- **Ownership checks** (`tx-sender` vs `current-owner`) enforce authorization in transfers
- **`stacks-block-height`** provides immutable timestamps for each action

---

## 🚀 Quick Start (Run Tests Locally)

### 1. Install Dependencies
```bash
npm install
```

### 2. Run Tests
```bash
npm test
```
- Runs Vitest tests configured in `package.json`
- Tests use the Clarinet simnet environment

### 3. Optional: Check Contract Syntax
```bash
clarinet check
```
- Validates Clarity syntax and catches compile errors
- Contract currently passes with expected warnings about unchecked input data

---

## 💡 Example Workflow

### 1. Register a Product
- Choose a `product-id` (recommend UUID format, e.g., `"prod-0001-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`)
- Call `register-product` with name and initial location
- Contract creates product record and checkpoint #1

### 2. Transfer Ownership
- Current owner calls `transfer-ownership`
- Provides new owner principal, new location, and notes
- Contract verifies caller is current owner
- Updates product and appends checkpoint entry

### 3. Query History
- Use `get-product` to see current state
- Use `get-checkpoint-count` to find total checkpoints
- Loop through checkpoints with `get-checkpoint(product-id, index)` for full history

---

## 🧪 Testing Tips (Student-Friendly)

1. **Start Simple:**
   - Test registering a product
   - Expect `(ok true)` response
   - Verify `get-product` returns expected fields

2. **Test Authorization:**
   - Call `transfer-ownership` from a non-owner
   - Expect `ERR-NOT-AUTHORIZED`

3. **Test Checkpoint Indexing:**
   - After N transfers, verify checkpoint count is correct
   - Fetch checkpoints at indexes 1 through N

4. **Use Deterministic IDs:**
   - In tests, use simple IDs like `"test-product-001"` for readability

5. **Test Edge Cases:**
   - Duplicate registration (expect `ERR-PRODUCT-EXISTS`)
   - Transfer non-existent product (expect `ERR-PRODUCT-NOT-FOUND`)

---

## 📏 Type Limits & Constraints

- **product-id:** `string-ascii 36` (typically UUID format)
- **name:** `string-ascii 100`
- **location:** `string-ascii 100`
- **status:** `string-ascii 50`
- **notes:** `string-ascii 500` (keep messages within limit)
- **principals:** Clarity `principal` type (addresses or contract identifiers)

---

## 🔮 Next Steps / Extension Ideas

### Access Control
- Add role-based permissions (manufacturer, distributor, retailer)
- Create an allowlist for verified handlers

### Proof of Authenticity
- Store IPFS content hashes in checkpoints
- Add cryptographic signatures for verification

### Events & Indexing
- Emit events for off-chain indexing
- Build a timeline API for frontend consumption

### Frontend Integration
- Create a web UI to query product history
- Display checkpoint timeline with map visualization

### Enhanced Status Management
- Add more granular status types (in-transit, delivered, inspected)
- Implement status transitions with validation rules

---

## 📚 Learning Resources

- [Clarity Language Documentation](https://docs.stacks.co/clarity)
- [Clarinet Testing Guide](https://docs.hiro.so/stacks/clarinet-js-sdk)
- [Stacks Blockchain](https://www.stacks.co/)

---

## 🤝 Contributing

This is a learning project! Feel free to:
- Add more comprehensive tests
- Improve error handling
- Extend functionality
- Improve documentation

---

## 📄 License

ISC

---

## 🙋 Questions or Issues?

- Check the contract comments in `contracts/supply-proof.clar`
- Review test examples in `tests/supply-proof.test.ts`
- Run `clarinet check` to validate your changes
- Run `npm test` to verify functionality

Happy learning! 🎉
