# Tech-Docs

# Threading Models: Rust, Go, and Python

## Rust
- **Thread Type:** True OS-level threads
- **Multi-threading:** Supports true multi-threading
- **Multi-core Usage:** Can utilize multiple cores effectively
- **Notes:** 
  - Uses 1:1 threading model (one OS thread per Rust thread)
  - Provides both low-level threading primitives and higher-level abstractions

## Go (Golang)
- **Thread Type:** Goroutines (lightweight, green threads)
- **Multi-threading:** Supports concurrent execution via goroutines
- **Multi-core Usage:** Can utilize multiple cores effectively
- **Notes:** 
  - Uses M:N threading model (M goroutines mapped to N OS threads)
  - Go runtime schedules goroutines across available OS threads

## Python
- **Thread Type:** Depends on implementation
  - CPython (standard): OS-level threads, but with Global Interpreter Lock (GIL)
  - Other implementations (e.g., Jython, IronPython): May use true threads
- **Multi-threading:** 
  - CPython: Limited by GIL for CPU-bound tasks
  - Other implementations: May support true multi-threading
- **Multi-core Usage:**
  - CPython: Limited for CPU-bound tasks due to GIL
  - Can use multiprocessing module for multi-core parallelism
- **Notes:** 
  - CPython uses 1:1 threading model, but GIL limits concurrent execution of Python bytecode
  - Asyncio provides concurrent I/O-bound operations using coroutines (not true multi-threading)

## Summary
- **True OS-level Threads:** Rust, Python (with caveats)
- **Green Threads:** Go (goroutines)
- **True Multi-threading:** Rust, Go
- **Effective Multi-core Usage:** Rust, Go, Python (with multiprocessing)

# Asynchronous Programming: Rust, Go, and Python

## Rust

- **Async Model:** Future-based
- **Key Features:**
  - Zero-cost abstractions
  - Compile-time guarantees
  - No runtime overhead
- **Syntax:** Uses `async`/`await` keywords
- **Runtime:** 
  - No built-in runtime
  - Popular runtimes: tokio, async-std
- **Concurrency:** Combines well with Rust's ownership system
- **Example:**
  ```rust
  async fn fetch_data() -> Result<String, Error> {
      // Asynchronous operations here
  }

  async fn main() {
      let result = fetch_data().await;
  }
  ```

## Go (Golang)

- **Async Model:** CSP (Communicating Sequential Processes)
- **Key Features:**
  - Goroutines (lightweight threads)
  - Channels for communication
  - Built-in scheduler
- **Syntax:** Uses `go` keyword for concurrency
- **Runtime:** Built into the language
- **Concurrency:** First-class language feature
- **Example:**
  ```go
  func fetchData() chan string {
      resultChan := make(chan string)
      go func() {
          // Asynchronous operations here
          resultChan <- result
      }()
      return resultChan
  }

  func main() {
      result := <-fetchData()
  }
  ```

## Python

- **Async Model:** Coroutine-based
- **Key Features:**
  - Event loop
  - Coroutines using generators (Python 3.4+)
  - Native coroutines and `async`/`await` syntax (Python 3.5+)
- **Syntax:** Uses `async`/`await` keywords
- **Runtime:** 
  - asyncio built into standard library
  - Third-party options: trio, curio
- **Concurrency:** Good for I/O-bound tasks
- **Example:**
  ```python
  import asyncio

  async def fetch_data():
      # Asynchronous operations here
      return result

  async def main():
      result = await fetch_data()

  asyncio.run(main())
  ```

## Key Differences

1. **Language Integration:**
   - Rust: Async is a language feature, but requires external runtimes
   - Go: Async is deeply integrated into the language and runtime
   - Python: Async is a language feature with a standard library module (asyncio)

2. **Performance:**
   - Rust: Highest performance, zero-cost abstractions
   - Go: Very good performance, lightweight goroutines
   - Python: Good for I/O-bound tasks, less suitable for CPU-bound tasks

3. **Learning Curve:**
   - Rust: Steepest learning curve due to ownership system
   - Go: Relatively easy to learn and use
   - Python: Familiar syntax, but async concepts can be challenging

4. **Use Cases:**
   - Rust: Systems programming, high-performance networking
   - Go: Web services, distributed systems
   - Python: Web development, data processing, scripting

# Async Programming: Pros and Cons for Different Task Types

## 1. CPU-bound Tasks

### Pros:
- Can improve responsiveness of the application by allowing other tasks to run
- Useful for managing multiple CPU-intensive tasks in a single-threaded environment

### Cons:
- Generally doesn't improve overall performance for pure CPU tasks
- Can add overhead due to task switching and management
- In languages with a Global Interpreter Lock (like Python), async doesn't help utilize multiple cores

### Best Use:
- When you need to maintain responsiveness while performing CPU-intensive operations
- In scenarios where you're managing multiple CPU tasks but don't need true parallelism

## 2. Network-bound Tasks

### Pros:
- Significantly improves performance by allowing multiple I/O operations concurrently
- Reduces idle time waiting for network responses
- Enables handling of many concurrent connections with fewer resources
- Improves scalability for network-heavy applications

### Cons:
- Can make code more complex and harder to reason about
- Requires careful error handling and timeout management
- Potential for callback hell or complex control flow (mitigated by modern async/await syntax)

### Best Use:
- Web servers handling multiple concurrent requests
- Applications making multiple API calls
- Real-time communication systems
- Any scenario with high-latency, low-bandwidth operations

## 3. Disk I/O-bound Tasks

### Pros:
- Allows multiple I/O operations to be in flight simultaneously
- Can significantly improve performance for applications with frequent disk access
- Useful for scenarios with many small read/write operations

### Cons:
- Benefits may be less pronounced than with network I/O due to lower latency
- SSDs reduce the need for async I/O in some cases
- Can complicate error handling and resource management

### Best Use:
- Applications dealing with many small files
- Scenarios where you need to perform multiple unrelated I/O operations
- Systems that need to remain responsive while performing background I/O

## General Considerations

- **Language and Runtime:** The effectiveness of async varies by language and runtime. For example, Node.js excels at async I/O, while Python's async is more beneficial for network than disk I/O.
- **Complexity vs. Performance:** Always consider if the performance gain justifies the added complexity of async code.
- **Scalability:** Async often shines in high-concurrency scenarios, regardless of the task type.
- **Resource Management:** Async can help manage limited resources (like file descriptors or database connections) more efficiently.

# Best Performance Approaches by Task Type and Language

## Rust

### CPU-bound Tasks
- **Best Approach:** Use native threads with `std::thread`
- **Why:** Rust's zero-cost abstractions and lack of runtime overhead make native threads very efficient
- **Example:**
  ```rust
  use std::thread;

  fn main() {
      let handles: Vec<_> = (0..num_cpus::get()).map(|_| {
          thread::spawn(|| {
              // CPU intensive work here
          })
      }).collect();

      for handle in handles {
          handle.join().unwrap();
      }
  }
  ```

### Network-bound Tasks
- **Best Approach:** Async with Tokio or async-std
- **Why:** Efficient handling of many concurrent connections with minimal overhead
- **Example (with Tokio):**
  ```rust
  use tokio;

  #[tokio::main]
  async fn main() {
      let handles = (0..1000).map(|_| {
          tokio::spawn(async {
              // Network operation here
          })
      }).collect::<Vec<_>>();

      for handle in handles {
          handle.await.unwrap();
      }
  }
  ```

### Disk I/O-bound Tasks
- **Best Approach:** Async with Tokio or async-std for many small operations, native threads for fewer, larger operations
- **Why:** Async is efficient for many concurrent small I/O operations, while threads work well for larger, less frequent operations
- **Example (Async with Tokio):**
  ```rust
  use tokio::fs::File;
  use tokio::io::AsyncReadExt;

  #[tokio::main]
  async fn main() -> Result<(), Box<dyn std::error::Error>> {
      let mut file = File::open("foo.txt").await?;
      let mut contents = vec![];
      file.read_to_end(&mut contents).await?;
      Ok(())
  }
  ```

## Go (Golang)

### CPU-bound Tasks
- **Best Approach:** Goroutines
- **Why:** Go's runtime efficiently schedules goroutines across available CPU cores
- **Example:**
  ```go
  import "runtime"

  func main() {
      numCPU := runtime.NumCPU()
      runtime.GOMAXPROCS(numCPU)

      for i := 0; i < numCPU; i++ {
          go func() {
              // CPU intensive work here
          }()
      }
      // Wait for goroutines to finish
  }
  ```

### Network-bound Tasks
- **Best Approach:** Goroutines with standard library's net package
- **Why:** Go's concurrency model is well-suited for handling many network connections efficiently
- **Example:**
  ```go
  import "net/http"

  func main() {
      http.HandleFunc("/", handler)
      http.ListenAndServe(":8080", nil)
  }

  func handler(w http.ResponseWriter, r *http.Request) {
      // Handle request
  }
  ```

### Disk I/O-bound Tasks
- **Best Approach:** Goroutines
- **Why:** Go's standard library uses non-blocking I/O under the hood, making goroutines efficient for both small and large I/O operations
- **Example:**
  ```go
  import "os"

  func main() {
      go func() {
          file, _ := os.Open("file.txt")
          defer file.Close()
          // Read file
      }()
      // Do other work
  }
  ```

## Python

### CPU-bound Tasks
- **Best Approach:** multiprocessing
- **Why:** Bypasses the Global Interpreter Lock (GIL), allowing true parallelism
- **Example:**
  ```python
  from multiprocessing import Pool

  def cpu_bound_task(x):
      # CPU intensive work here
      return result

  if __name__ == '__main__':
      with Pool() as p:
          results = p.map(cpu_bound_task, range(10))
  ```

### Network-bound Tasks
- **Best Approach:** asyncio
- **Why:** Efficiently handles many concurrent connections without the overhead of threads
- **Example:**
  ```python
  import asyncio
  import aiohttp

  async def fetch(url):
      async with aiohttp.ClientSession() as session:
          async with session.get(url) as response:
              return await response.text()

  async def main():
      urls = ['http://example.com' for _ in range(100)]
      tasks = [asyncio.create_task(fetch(url)) for url in urls]
      results = await asyncio.gather(*tasks)

  asyncio.run(main())
  ```

### Disk I/O-bound Tasks
- **Best Approach:** asyncio for many small operations, multiprocessing for fewer, larger operations
- **Why:** asyncio is efficient for concurrent small I/O, while multiprocessing helps with larger operations that might block
- **Example (asyncio):**
  ```python
  import asyncio
  import aiofiles

  async def read_file(filename):
      async with aiofiles.open(filename, mode='r') as file:
          return await file.read()

  async def main():
      filenames = ['file1.txt', 'file2.txt', 'file3.txt']
      tasks = [asyncio.create_task(read_file(filename)) for filename in filenames]
      results = await asyncio.gather(*tasks)

  asyncio.run(main())
  ```

#### TCP 3 Way Handshake

# TCP 3-Way Handshake

## Overview

This repository explains the TCP (Transmission Control Protocol) 3-way handshake process, a fundamental mechanism used to establish a connection between a client and a server in TCP/IP networks.

## What is the TCP 3-Way Handshake?

The TCP 3-way handshake is a procedure used to establish a reliable connection between two devices before data transmission begins. It ensures that both sides are ready to communicate and agree on the initial sequence numbers for the connection.

## Process

The handshake involves three steps:

1. **SYN (Synchronize)**
2. **SYN-ACK (Synchronize-Acknowledge)**
3. **ACK (Acknowledge)**

## Detailed Explanation

### Step 1: SYN

- The client initiates the connection by sending a SYN packet to the server.
- This packet contains an initial sequence number (x).
- The SYN flag in the TCP header is set to 1.

### Step 2: SYN-ACK

- Upon receiving the SYN packet, the server responds with a SYN-ACK packet.
- This packet acknowledges the client's SYN and includes the server's own initial sequence number (y).
- The ACK number is set to the client's sequence number plus one (x+1).
- Both SYN and ACK flags are set to 1 in the TCP header.

### Step 3: ACK

- The client receives the SYN-ACK and sends back an ACK packet to acknowledge the server's response.
- The sequence number is set to x+1 (the ACK number received from the server).
- The ACK number is set to y+1 (the server's sequence number plus one).
- The ACK flag is set to 1, while the SYN flag is set to 0.

After these three steps, the connection is established, and both the client and server can begin sending data.

## Key Points

- Ensures both sides are ready to communicate
- Prevents old duplicate connections from being established
- Allows both sides to agree on the Maximum Segment Size (MSS) for the connection
- Designed to work reliably even if packets are lost, delayed, or duplicated

## Importance

The TCP 3-way handshake is crucial for:

- Establishing reliable connections
- Ensuring ordered delivery of data
- Setting the foundation for handling potential issues like packet loss, reordering, and duplication during data transfer

## Further Reading

- [RFC 793: Transmission Control Protocol](https://tools.ietf.org/html/rfc793)
- [TCP/IP Guide](http://www.tcpipguide.com/free/t_TCPConnectionEstablishmentProcessTheThreeWayHandsh.htm)

## Contributing

Feel free to contribute to this explanation by submitting pull requests or opening issues for any corrections or additional information.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

#### DNS Resolver

# DNS Resolving Process

## Overview

This repository explains the Domain Name System (DNS) resolving process, a fundamental mechanism used to translate human-readable domain names into IP addresses that computers use to identify each other on the internet.

## What is DNS Resolving?

DNS resolving is the process of converting a domain name (like www.example.com) into an IP address (like 192.0.2.1). This translation is crucial for navigating the internet, as it allows users to use memorable domain names while computers communicate using numerical IP addresses.

## Detailed Explanation

The DNS resolving process typically involves the following steps:

1. **Client Query**
   - User enters a domain name in their browser
   - Computer checks its local DNS cache
   - If not found, sends a query to the Local DNS Resolver

2. **Local DNS Resolver**
   - Checks its own cache for the IP address
   - If not found, initiates a recursive query process

3. **Root DNS Servers**
   - Local DNS Resolver queries a Root DNS Server
   - Root Server provides referral to relevant Top-Level Domain (TLD) DNS servers

4. **TLD DNS Servers**
   - Resolver queries the TLD DNS server
   - TLD server provides the address of the Authoritative DNS server for the domain

5. **Authoritative DNS Server**
   - Resolver queries the Authoritative DNS server
   - This server returns the actual IP address for the requested domain

6. **Resolution Complete**
   - Local DNS Resolver caches the IP address
   - Resolver returns the IP address to the client's computer

7. **Client Receives IP**
   - Computer receives the IP address
   - Can now establish a connection with the web server at that IP address

## Key Points

- Hierarchical system of servers distributes the load and maintains efficiency
- Caching occurs at multiple levels to speed up future requests
- Process typically takes only milliseconds to complete
- DNS primarily uses UDP for queries and responses, with TCP as a fallback
- Security measures like DNSSEC can ensure integrity and authenticity of responses

## Importance of DNS

DNS is critical to the functioning of the internet because it:
- Allows the use of human-readable domain names
- Provides a distributed, scalable system for name resolution
- Enables load balancing and fault tolerance for web services
- Supports email routing and other internet services

## Further Reading

- [RFC 1034: Domain Names - Concepts and Facilities](https://tools.ietf.org/html/rfc1034)
- [RFC 1035: Domain Names - Implementation and Specification](https://tools.ietf.org/html/rfc1035)
- [DNS Security Extensions (DNSSEC)](https://www.icann.org/resources/pages/dnssec-qaa-2014-01-29-en)

## Contributing

We welcome contributions to improve this explanation. Please submit pull requests or open issues for any corrections or additional information.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

# Software Licensing Models: A Comparison

## 1. Open Source Licenses

### GNU General Public License (GPL)
**Pros:**
- Ensures software remains free and open
- Promotes sharing and collaboration
- Protects user freedoms

**Cons:**
- Can be restrictive for commercial use
- May discourage some businesses from using the software

### MIT License
**Pros:**
- Very permissive
- Simple and easy to understand
- Allows commercial use and modification

**Cons:**
- Provides little protection for the original author
- Derivative works can be made proprietary

### Apache License 2.0
**Pros:**
- Permissive for commercial use
- Includes patent license
- Compatible with GPL v3

**Cons:**
- More complex than MIT or BSD licenses
- May require including a copy of the license with distribution

## 2. Proprietary Licenses

### Commercial Software License
**Pros:**
- Provides revenue for developers
- Can offer specialized support and features

**Cons:**
- Restricts user freedoms
- Often more expensive for users
- Source code typically not available

### Shareware
**Pros:**
- Allows users to try before buying
- Can lead to wider distribution

**Cons:**
- Limited functionality in free version
- May have time restrictions

## 3. Hybrid Licenses

### Dual Licensing
**Pros:**
- Offers flexibility for different users
- Can generate revenue while still offering open source option

**Cons:**
- Can be complex to manage
- May confuse users

### Freemium
**Pros:**
- Attracts users with free basic version
- Can convert free users to paying customers

**Cons:**
- Must balance free and paid features carefully
- May limit growth of paid user base

## 4. Creative Commons Licenses

### CC BY (Attribution)
**Pros:**
- Allows wide use while requiring attribution
- Simple to understand and implement

**Cons:**
- Doesn't prevent commercial use (if that's a concern)
- Doesn't protect against misuse or misrepresentation

### CC BY-SA (Attribution-ShareAlike)
**Pros:**
- Ensures derivatives are shared under the same terms
- Promotes open collaboration

**Cons:**
- Can be restrictive for some commercial uses
- May limit adoption in certain contexts

## 5. Public Domain (e.g., CC0)

**Pros:**
- Maximum freedom for users
- No restrictions on use or modification

**Cons:**
- No protection for the original creator
- Potential for misuse without attribution

## Key Considerations
- **Commercial vs. Non-Commercial Use:** Some licenses restrict commercial use, while others allow it.
- **Derivative Works:** Consider whether you want to allow modifications and under what terms.
- **Patent Rights:** Some licenses (like Apache 2.0) explicitly address patent rights.
- **Compatibility:** Ensure the license is compatible with other software or libraries you're using.
- **Community Building:** Some licenses are better for building open-source communities.
- **Legal Protection:** Consider the level of legal protection you need for your software.

# SSL/TLS Encryption Process

## Overview

This repository explains the Secure Sockets Layer (SSL) and Transport Layer Security (TLS) encryption process, which is fundamental to secure communication on the internet, particularly when accessing HTTPS websites.

## What is SSL/TLS Encryption?

SSL/TLS encryption is a security protocol that provides privacy, authentication, and integrity to internet communications. It's the technology that creates a secure, encrypted connection between a web browser and a web server, ensuring that all data passed between them remains private and integral.

## Detailed Explanation

The SSL/TLS encryption process involves the following steps when you access an HTTPS website:

1. **Initiating the Connection**
   - Browser initiates connection to web server
   - HTTPS in URL indicates SSL/TLS should be used

2. **SSL/TLS Handshake**
   a) **Client Hello**
      - Client sends supported SSL/TLS versions, cipher suites, and Client Random
   
   b) **Server Hello and Certificate**
      - Server responds with chosen version, cipher suite, Server Random, and SSL certificate
   
   c) **Certificate Verification**
      - Client verifies server's certificate with a trusted Certificate Authority (CA)
   
   d) **Key Exchange**
      - Client generates and sends encrypted Pre-Master Secret
      - Both sides generate Master Secret
   
   e) **Finished**
      - Both sides send encrypted "Finished" message

3. **Secure Communication Established**
   - Encrypted connection is set up
   - Session keys derived from Master Secret

4. **Data Encryption and Transmission**
   - All data encrypted using session keys (typically with symmetric encryption)

5. **Decryption at Endpoints**
   - Client and server decrypt data using session keys

## Key Points

- Handshake uses asymmetric encryption for key exchange
- Data transmission uses symmetric encryption for efficiency
- Process ensures authentication, confidentiality, and integrity
- Happens automatically when accessing HTTPS sites

## Importance of SSL/TLS Encryption

SSL/TLS encryption is crucial for:
- Protecting sensitive information (passwords, credit card details, etc.)
- Verifying the authenticity of websites
- Maintaining data integrity during transmission
- Building trust with users and customers

## Further Reading

- [RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3](https://tools.ietf.org/html/rfc8446)
- [OWASP Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)

## Contributing

We welcome contributions to improve this explanation. Please submit pull requests or open issues for any corrections or additional information.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## GRPC Server And Client

# gRPC Server and Client in Go

## Overview

This repository demonstrates how to implement a gRPC server and client in Go, showcasing all four types of gRPC communication methods. gRPC is a high-performance, open-source framework developed by Google for remote procedure calls (RPC).

## What is gRPC?

gRPC (gRPC Remote Procedure Call) is a modern, open-source framework that enables efficient communication between distributed systems. It uses Protocol Buffers as its interface definition language and underlying message interchange format.

## gRPC Communication Types

gRPC supports four types of communication:

1. **Unary RPC**: Client sends a single request and receives a single response.
2. **Server Streaming RPC**: Client sends a single request and receives a stream of responses.
3. **Client Streaming RPC**: Client sends a stream of requests and receives a single response.
4. **Bidirectional Streaming RPC**: Both client and server send streams of messages to each other.

## Project Structure

```
.
├── main.go
├── echo.proto
└── README.md
```

## Prerequisites

- Go 1.15+
- Protocol Buffers compiler (`protoc`)
- Go plugins for Protocol Buffers and gRPC

## Setup

1. Install the required Go packages:
   ```
   go get google.golang.org/grpc
   go get google.golang.org/protobuf/cmd/protoc-gen-go
   go get google.golang.org/grpc/cmd/protoc-gen-go-grpc
   ```

2. Generate Go code from the `.proto` file:
   ```
   protoc --go_out=. --go-grpc_out=. echo.proto
   ```

## Running the Example

1. Start the server and client:
   ```
   go run main.go
   ```

This will start the gRPC server and then run the client, demonstrating all four types of gRPC communication.

## Code Explanation

### Server Implementation

The server implements four methods corresponding to the four types of gRPC communication:

- `UnaryEcho`: Demonstrates Unary RPC
- `ServerStreamingEcho`: Demonstrates Server Streaming RPC
- `ClientStreamingEcho`: Demonstrates Client Streaming RPC
- `BidirectionalStreamingEcho`: Demonstrates Bidirectional Streaming RPC

### Client Implementation

The client demonstrates how to use each type of gRPC communication:

- Unary RPC: Single request and response
- Server Streaming RPC: Receives multiple responses in a loop
- Client Streaming RPC: Sends multiple requests in a loop
- Bidirectional Streaming RPC: Sends and receives streams concurrently using goroutines

## Key Points

- gRPC uses Protocol Buffers for efficient serialization.
- It runs over HTTP/2, allowing for features like multiplexing and header compression.
- The server implements the service interface generated from the `.proto` file.
- The client uses a generated stub to make RPC calls.
- Error handling is done using the `status` package, which provides gRPC-specific error codes.

## Further Reading

- [gRPC Documentation](https://grpc.io/docs/)
- [Protocol Buffers Documentation](https://developers.google.com/protocol-buffers)
- [gRPC-Go GitHub Repository](https://github.com/grpc/grpc-go)

## Contributing

Contributions to improve the example or documentation are welcome. Please submit a pull request or open an issue to discuss proposed changes.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

```go
// File: main.go

package main

import (
	"context"
	"fmt"
	"io"
	"log"
	"net"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"google.golang.org/protobuf/types/known/emptypb"
)

// Assuming you have already generated the gRPC code from your .proto file
// For this example, we'll define the service and messages inline

type EchoService struct {
	UnimplementedEchoServiceServer
}

func (s *EchoService) UnaryEcho(ctx context.Context, req *EchoRequest) (*EchoResponse, error) {
	return &EchoResponse{Message: req.Message}, nil
}

func (s *EchoService) ServerStreamingEcho(req *EchoRequest, stream EchoService_ServerStreamingEchoServer) error {
	for i := 0; i < 5; i++ {
		if err := stream.Send(&EchoResponse{Message: fmt.Sprintf("%s %d", req.Message, i)}); err != nil {
			return err
		}
		time.Sleep(time.Second)
	}
	return nil
}

func (s *EchoService) ClientStreamingEcho(stream EchoService_ClientStreamingEchoServer) error {
	var messages []string
	for {
		req, err := stream.Recv()
		if err == io.EOF {
			return stream.SendAndClose(&EchoResponse{Message: fmt.Sprintf("Received: %v", messages)})
		}
		if err != nil {
			return err
		}
		messages = append(messages, req.Message)
	}
}

func (s *EchoService) BidirectionalStreamingEcho(stream EchoService_BidirectionalStreamingEchoServer) error {
	for {
		req, err := stream.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			return err
		}
		if err := stream.Send(&EchoResponse{Message: req.Message}); err != nil {
			return err
		}
	}
}

func main() {
	// Start the server
	go startServer()

	// Wait for the server to start
	time.Sleep(time.Second)

	// Run the client
	runClient()
}

func startServer() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	RegisterEchoServiceServer(s, &EchoService{})
	log.Println("Starting gRPC server on port 50051")
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

func runClient() {
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	client := NewEchoServiceClient(conn)

	// Unary RPC
	resp, err := client.UnaryEcho(context.Background(), &EchoRequest{Message: "Hello, Unary!"})
	if err != nil {
		log.Fatalf("UnaryEcho error: %v", err)
	}
	log.Printf("UnaryEcho response: %s", resp.Message)

	// Server Streaming RPC
	stream, err := client.ServerStreamingEcho(context.Background(), &EchoRequest{Message: "Hello, Server Streaming!"})
	if err != nil {
		log.Fatalf("ServerStreamingEcho error: %v", err)
	}
	for {
		resp, err := stream.Recv()
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Fatalf("ServerStreamingEcho stream error: %v", err)
		}
		log.Printf("ServerStreamingEcho response: %s", resp.Message)
	}

	// Client Streaming RPC
	clientStream, err := client.ClientStreamingEcho(context.Background())
	if err != nil {
		log.Fatalf("ClientStreamingEcho error: %v", err)
	}
	for i := 0; i < 5; i++ {
		if err := clientStream.Send(&EchoRequest{Message: fmt.Sprintf("Hello, Client Streaming %d!", i)}); err != nil {
			log.Fatalf("ClientStreamingEcho send error: %v", err)
		}
	}
	resp, err = clientStream.CloseAndRecv()
	if err != nil {
		log.Fatalf("ClientStreamingEcho close error: %v", err)
	}
	log.Printf("ClientStreamingEcho response: %s", resp.Message)

	// Bidirectional Streaming RPC
	bidiStream, err := client.BidirectionalStreamingEcho(context.Background())
	if err != nil {
		log.Fatalf("BidirectionalStreamingEcho error: %v", err)
	}
	waitc := make(chan struct{})
	go func() {
		for {
			resp, err := bidiStream.Recv()
			if err == io.EOF {
				close(waitc)
				return
			}
			if err != nil {
				log.Fatalf("BidirectionalStreamingEcho receive error: %v", err)
			}
			log.Printf("BidirectionalStreamingEcho response: %s", resp.Message)
		}
	}()
	for i := 0; i < 5; i++ {
		if err := bidiStream.Send(&EchoRequest{Message: fmt.Sprintf("Hello, Bidi Streaming %d!", i)}); err != nil {
			log.Fatalf("BidirectionalStreamingEcho send error: %v", err)
		}
	}
	bidiStream.CloseSend()
	<-waitc
}
```