FROM rust:1.81 as builder

# Install dependencies
RUN apt-get update && apt-get install -y pkg-config libssl-dev libpq-dev

# Set up build directory
WORKDIR /app

# Copy and build dependencies first (for better caching)
COPY Cargo.toml ./

# Create a dummy main.rs to build dependencies
RUN mkdir -p src && echo "fn main() {}" > src/main.rs
RUN cargo build --release

# Remove the dummy file
RUN rm -f src/main.rs

# Copy the actual source code
COPY src ./src

# Build the application with the db_storage feature always enabled
RUN cargo build --release

# Create a smaller runtime image
FROM ubuntu:22.04

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libssl-dev \
    libpq-dev \
    ca-certificates \
    curl \
    procps \
    file \
    gdb \
    strace

# Create temp directory for CLI downloads
RUN mkdir -p /tmp/lava-cli

# Create directory for database
RUN mkdir -p /app/data && chmod 777 /app/data

# Set up application directory
WORKDIR /app

# Copy the built binary from the builder stage
COPY --from=builder /app/target/release/lava-test-server /app/lava-test-server

# Create a startup script
RUN echo '#!/bin/bash\necho "Starting server..."\n/app/lava-test-server "$@"' > /app/start.sh && \
    chmod +x /app/start.sh

# Set environment variable for database URL
ENV DATABASE_URL=/app/data/tests.db

# Start the application
ENTRYPOINT ["/app/start.sh"] 