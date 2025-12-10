# gRPC Bitcoin Payment Service

A simple gRPC-based Bitcoin payment service implementation in Rust, demonstrating client-server communication using Protocol Buffers and Tonic.

## Overview

This project implements a basic Bitcoin payment service using gRPC. It consists of:
- A **gRPC server** that handles payment requests
- A **gRPC client** that sends payment requests to the server
- **Protocol Buffers** definitions for the service and message types

## Project Structure

```
grpc-impl/
├── proto/
│   └── payments.proto      # Protocol Buffers definition
├── src/
│   ├── server.rs           # gRPC server implementation
│   └── client.rs           # gRPC client implementation
├── build.rs                # Build script for compiling .proto files
├── Cargo.toml              # Project dependencies
└── README.md               # This file
```

## Prerequisites

Before running this project, ensure you have the following installed:

1. **Rust** (latest stable version)
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. **Protocol Buffers Compiler** (`protoc`)
   ```bash
   # macOS
   brew install protobuf
   
   # Ubuntu/Debian
   sudo apt install protobuf-compiler
   
   # Arch Linux
   sudo pacman -S protobuf
   ```

## Dependencies

The project uses the following key dependencies:

- **tonic** (0.14.2) - gRPC framework for Rust
- **tonic-prost** (0.14.2) - Prost integration for Tonic
- **prost** (0.14.1) - Protocol Buffers implementation
- **tokio** (1.48.0) - Async runtime
- **tonic-build** (0.14.2) - Build-time code generation
- **tonic-prost-build** (0.14.2) - Build-time Prost integration

## How It Works

### 1. Protocol Buffers Definition

The [`proto/payments.proto`](file:///Users/subh/Desktop/code-playground/grpc-impl/proto/payments.proto) file defines:

- **Service**: `Bitcoin` with a `SendPayment` RPC method
- **Request Message**: `BTCPaymentRequest` containing sender address, receiver address, and amount
- **Response Message**: `BTCPaymentResponse` containing success status and message

```protobuf
service Bitcoin {
    rpc SendPayment (BTCPaymentRequest) returns (BTCPaymentResponse);
}

message BTCPaymentRequest {
    string from_addr = 1;
    string to_addr = 2;
    uint32 amount = 3;
}

message BTCPaymentResponse {
    bool successful = 1;
    string message = 2;
}
```

### 2. Build Script

The [`build.rs`](file:///Users/subh/Desktop/code-playground/grpc-impl/build.rs) file compiles the `.proto` file at build time, generating Rust code for the service and message types:

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    tonic_prost_build::compile_protos("proto/payments.proto")?;
    Ok(())
}
```

### 3. Server Implementation

The [`src/server.rs`](file:///Users/subh/Desktop/code-playground/grpc-impl/src/server.rs) implements the gRPC server:

- Includes generated code using `tonic::include_proto!("payments")`
- Implements the `Bitcoin` trait for `BitcoinService`
- Handles `SendPayment` requests and returns formatted responses
- Listens on `[::1]:50051` (localhost IPv6)

### 4. Client Implementation

The [`src/client.rs`](file:///Users/subh/Desktop/code-playground/grpc-impl/src/client.rs) implements the gRPC client:

- Connects to the server at `http://[::1]:50051`
- Creates a payment request with sample data
- Sends the request and prints the response

## Building the Project

To build the project:

```bash
cargo build
```

This will:
1. Run the build script to compile `.proto` files
2. Generate Rust code from Protocol Buffers
3. Compile the server and client binaries

## Running the Application

### Step 1: Start the Server

In one terminal, run:

```bash
cargo run --bin payments-server
```

The server will start listening on `[::1]:50051`.

### Step 2: Run the Client

In another terminal, run:

```bash
cargo run --bin payments-client
```

You should see output like:

```
Response: Response { 
    metadata: MetadataMap { headers: {"content-type": "application/grpc", ...} }, 
    message: BtcPaymentResponse { 
        successful: true, 
        message: "Payment from 123456 to 654321 for 100 satoshis" 
    }, 
    extensions: {} 
}
```

## Understanding the Code Flow

1. **Build Time**:
   - `build.rs` runs and compiles `payments.proto`
   - Generated Rust code is placed in the build output directory
   - Code is included via `tonic::include_proto!("payments")`

2. **Server Startup**:
   - Server creates a `BitcoinService` instance
   - Registers the service with Tonic's server builder
   - Starts listening for incoming gRPC requests

3. **Client Request**:
   - Client connects to the server
   - Creates a `BtcPaymentRequest` with payment details
   - Sends the request via the `send_payment` RPC method
   - Receives and prints the `BtcPaymentResponse`

4. **Server Processing**:
   - Server receives the request
   - Extracts payment details
   - Creates a response with a success message
   - Returns the response to the client

## Common Issues and Solutions

### Issue: `protoc` not found

**Solution**: Install Protocol Buffers compiler:
```bash
brew install protobuf  # macOS
```

### Issue: `tonic_prost` module not found

**Solution**: Ensure `tonic-prost` is in your dependencies:
```bash
cargo add tonic-prost
```

### Issue: Connection refused

**Solution**: Make sure the server is running before starting the client.

## Customization

To modify the service:

1. **Change the proto definition**: Edit `proto/payments.proto`
2. **Update server logic**: Modify `src/server.rs` to implement new behavior
3. **Update client requests**: Modify `src/client.rs` to send different data
4. **Rebuild**: Run `cargo build` to regenerate code from updated `.proto` files

## Learning Resources

- [Tonic Documentation](https://docs.rs/tonic/)
- [Protocol Buffers Guide](https://protobuf.dev/)
- [gRPC Concepts](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [Tokio Async Runtime](https://tokio.rs/)

## License

This is a learning project and is provided as-is for educational purposes.
