# Lava Test Server

A Rust-based test server that runs a test suite for the Borrower CLI when receiving a specific request.

## Features

- HTTP server built with Actix Web
- Runs a test suite that:
  1. Generates a new mnemonic and new receiving addresses for BTC and LavaUSD
  2. Calls testnet faucet endpoints to receive BTC and LavaUSD
  3. Downloads and installs the Loans Borrower CLI
  4. Creates a new loan with specific parameters
  5. Repays the loan
  6. Verifies the loan is closed with a corresponding repayment transaction
- Docker and docker-compose support for easy deployment

## Prerequisites

- Docker and docker-compose installed

## Building and Running

### Using Docker Compose (Recommended)

```bash
# Clone the repository
git clone <repository-url>
cd lava-test-server

# Build and run the server
docker-compose up --build -d

# Check logs
docker-compose logs -f
```

### Using Cargo (Development)

```bash
# Clone the repository
git clone <repository-url>
cd lava-test-server

# Build the server
cargo build --release

# Run the server
RUST_LOG=info ./target/release/lava-test-server
```

## API Endpoints

### 1. Run Test

Triggers the test suite execution.

- **URL**: `/run-test`
- **Method**: `POST`
- **Request Body**:
  ```json
  {
    "run_id": "optional-custom-id"  // Optional
  }
  ```
- **Response**:
  ```json
  {
    "run_id": "generated-or-provided-id",
    "status": "started",
    "message": "Test started successfully"
  }
  ```

### 2. Check Test Status

Get the status of a previously run test. (Requires DB storage feature to be enabled)

- **URL**: `/test-status/{run_id}`
- **Method**: `GET`
- **Response** (if found):
  ```json
  {
    "id": "test-id",
    "timestamp": "2023-10-20T12:34:56Z",
    "success": true,
    "mnemonic": "word1 word2 ...",
    "btc_address": "tb1q...",
    "lava_usd_pubkey": "...",
    "contract_id": "...",
    "collateral_repayment_txid": "...",
    "details": { ... }
  }
  ```
- **Response** (if not found):
  ```json
  {
    "error": "Test not found or database storage not enabled"
  }
  ```

### 3. Health Check

Check if the server is running.

- **URL**: `/health`
- **Method**: `GET`
- **Response**:
  ```json
  {
    "status": "healthy",
    "timestamp": "2023-10-20T12:34:56Z"
  }
  ```

## Configuration

The server can be configured using environment variables:

- `BIND_ADDRESS`: The address and port to bind the server to (default: `0.0.0.0:8080`)
- `RUST_LOG`: Logging level (default: `info`)

## Optional Features

### Database Storage

To enable storing test results in a database, build with the `db_storage` feature:

```bash
cargo build --release --features db_storage
```

In `docker-compose.yml`, you can add a SQLite or PostgreSQL database depending on your needs.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 