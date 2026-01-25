# SSL/TLS Fundamentals (Prerequisite for Kubernetes Certificates)

## Intro: why this video exists

# 

- Speaker introduces **video #20** in the CK 2024 series.
- This video is a **prerequisite** for the next video on **certificates in Kubernetes**.
- Goal: a “deep dive” into **how SSL/TLS works end-to-end**.
- Engagement target: **120 likes + 120 comments**.

## Client–server communication over HTTP (baseline example)

# 
- Setup: a **user** talking to a **server** over the internet.
- Speaker notes: the “user could be a client,” so it’s a **client–server interaction**.
- Example request: user sends an **HTTP GET** request.
- Speaker mentions HTTP methods like **GET, PUT, POST**, etc.
- Protocol framing:
- HTTP = *“hypertext transfer protocol.”*
- Speaker says web traffic goes through **HTTP or HTTPS** (HTTPS discussed later).

### Authentication step (and the risk)

# 
- Server asks the user to identify themselves.
- User sends credentials (e.g., username/password).
- Speaker’s definition: *“authentication is nothing but you are defining who you are—who you actually are.”*
- Threat: a hacker “sniffing the network” can intercept an unsecured connection.
- If the hacker captures credentials, they can send requests to the server as the user.
- Speaker emphasizes impact for sensitive sites (banking, stock trading/brokerage, insurance, etc.).
- Takeaway: HTTP is “not considered secure” in this context.

## Attempt 1: Symmetric encryption (and why it fails on the internet)

# 
- Idea: encrypt credentials/data before sending.

### Symmetric key flow

# 
- User generates a **key** to encrypt data.
- Data is encrypted and sent to the server.
- Server needs the **same key** to decrypt.
- Speaker definition: *“This type of encryption is called symmetric encryption—in symmetric encryption the same key is used to encrypt and decrypt the data.”*

### Loophole described

# 
- Hacker can sniff:
1. the encrypted data
2. the key when it’s sent to the server
- Result: hacker has both the data and the key, so the communication is compromised.
- Conclusion: symmetric encryption alone is not a best practice for transferring keys over the internet (as described).

## Attempt 2: Asymmetric encryption (SSH analogy)

### ### SSH key pair (public/private)

# 

- Scenario: user wants to create an **SSH connection** to a server without using a password (passwords are “too risky”).
- User generates a **public/private key pair**.

### ### ssh-keygen

# 
- Utility mentioned: **ssh-keygen** generates the **public key** and **private key**.

### Asymmetric definition and placement

# 
- Speaker’s reasoning: use different keys for encryption vs decryption.
- Speaker definition: *“This type of encryption is called asymmetric encryption.”*
- Where keys go (as described):
- Keep the **private key** with the user.
- Send the **public key** to the server.
- Public key is added to `authorized_keys` (speaker references “SSH authorized keys”).
- User initiates SSH and provides the private key; server verifies/decrypts using the public key; connection is established (speaker’s high-level flow).

## Applying asymmetric ideas to HTTP (TLS-style handshake explanation)

# 
- Now the focus shifts back to **HTTP** traffic.

### Server key pair generation

# 
- Server has:
- **server public key**
- **server private key**

### OpenSSL

# - - Utility mentioned: **OpenSSL** is used to generate “certificates and keys” on the server.
- Private key stays on the server (used for decryption); public key is sent out (used for encryption).

### Hybrid approach described: server asymmetric + user symmetric

# 
- User receives the **server public key**.
- User also creates/has a **symmetric key**.
- User encrypts their symmetric key using the **server public key**.
- The encrypted bundle is sent to the server.
- Server uses its **private key** to decrypt and recover the user’s symmetric key.
- After that, speaker says: future communication can be handled using that **symmetric key**.

### Why a sniffer can’t read it (in this model)

# 
- Hacker may capture:
- encrypted symmetric key
- server public key
- But hacker lacks the **server private key**, so cannot decrypt the symmetric key.
- Speaker’s conclusion: the intercepted encrypted data is “of no use” without the server’s private key.

## Remaining risk: man-in-the-middle / phishing proxy scenario

# 
- Speaker describes a scenario where the user talks to a **fake site** (phishing) that looks like the real bank site.
- The hacker acts as a proxy:
- Server sends its public key to the hacker (thinking hacker is the user).
- Hacker can then establish a secure session with the server using the hacker’s own symmetric key.
- Core issue: user hasn’t verified they’re receiving keys from the right server.

## Certificates and HTTPS: validating the server identity

# 
- Solution introduced: validate the server identity using **certificates** instead of just a raw public key.
- Speaker states: “instead of public and private key we are using certificates.”

### Browser validation idea

# 
- User’s browser can validate whether:
- the certificate was issued to the domain
- the certificate is valid
- Speaker references the browser UI: “connection is secure” and “certificate is valid,” plus viewing details (common name/domain, organization, public key, fingerprints, issued/expiry, etc.).
- Once validated, the certificate is used to encrypt the symmetric key and establish **HTTPS**:
- Speaker definition-style phrasing: *HTTPS is “secured hypertext transfer protocol.”*
- “That is why this connection is called SSL/TLS and it is over HTTPS” (speaker’s wording).

## How certificates get issued (CA flow)

# 
- Server generates a **CSR (Certificate Signing Request)**.
- Server sends CSR to a **Certificate Authority (CA)**.
- CA validates:
- the request
- the domain
- authentication / domain ownership (speaker’s description)
- CA signs it and sends the certificate back to the server.
- Speaker notes public/private concept again:
- server retains the private side
- browsers have trusted public CA certificates and use them for validation (as he explains).

### Public CA vs internal/private CA

# 
- For public internet domains: use public CAs (speaker gives examples like “Digi…” and others).
- For internal company sites (intranet): use a **custom/internal CA** hosted within the organization to issue and sign certificates.

## Wrap-up: what to focus on + what’s next

# 
- Speaker acknowledges the topic can be confusing and suggests:
- rewatching
- taking notes
- drawing diagrams yourself
- “Main part” to understand (speaker emphasis):
- securely sending the **user’s symmetric key** to the server by encrypting it with the server’s public key, decryptable only by the server’s private key.
- then “replace” public/private keys with certificates, and add CA signing/validation.
- Next video: certificates **specifically in Kubernetes**, including creating a **CSR** there.
- No GitHub exercise for this video (no hands-on), but speaker suggests making diagrams/blogs and sharing progress.

# 

# 

#