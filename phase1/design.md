# zk-agreement

## Overview

The zk-agreement is a zero-knowledge protocol that enables two parties to reach a service agreement (for example, an SLA to host a Docker container in the cloud) without revealing the details within the agreement. Additionally, it offers a mechanism to prevent either party from violating the agreement. There will also be third-party oracles to monitor the validity of the behavior of both parties.
![alt text](img/phase1.png)

## Research Problems

1. What problems can Zero-Knowledge Proofs (ZKP) solve in private agreements?
2. To what extent can ZKPs provide privacy in private agreements?
3. How can it be verified that both parties act in good faith?
4. How can it be verified that one of the bad parties act in bad faith?
5. How to design the crypto protocol for a private agreement?

## Blockchain and Smart Contracts

Reasons for the protocol to have blockchain:

1. **public verification of Proofs of Agreement**: Blockchain allows the ZKP of proofs of agreement to be verified
2. **smart Contracts are unstoppable and deterministic**: Once deployed, smart contracts execute exactly as programmed without any possibility of downtime, censorship, fraud, or third-party interference.
3. **cryptographic guaranteed trust between two parties**: By anchoring the agreement in a blockchain, both parties can trust that the agreement cannot be altered retroactively without mutual consent. This trust is not based on the reputation or promises of either party but on the immutable and transparent nature of the blockchain, where all transactions and agreements are permanently recorded and verifiable.
4. **integration with oracles for real-world data**: Blockchain oracles can be used to securely feed real-world data into the blockchain to trigger agreement conditions encoded in smart contracts. This capability allows for dynamic agreements that can adapt to external conditions or events, enhancing the flexibility and applicability of the zk-agreement protocol to a wide range of use cases.

## Zero Knowledge Proofs

Reasons for the protocol to use Zero Knowledge Proofs:

1. **preserves privacy**: ZKP allows the two parties involved in an agreement (e.g., hosting a Docker container in the cloud) to verify the compliance with the terms without revealing the specific details of the agreement to each other or to third parties.

2. **reduces on chain data**: By compressing off-chain agreement data and code, ZKP minimizes the need for extensive on-chain storage.

3. **enhances trust**: Through the use of third-party oracles to monitor the validity of both parties' behavior, ZKP enables trust in the agreement's enforcement without requiring the oracles to have direct access to the contents of the agreement.
4. **prevents violations**: ZKP provides a mechanism to detect and prevent one party from violating the agreement.

5. **facilitates dispute resolution**: Through crypto protocol design, ZKP could allow for verification of claims related to the agreement without exposing its specifics to the adjudicating authority.

## Public Informations

### public data:

- (could be private) identity of party A, **cloud service provider**
- (could be private) identity of party B, **end users**
- public contents in terms of a service agreement
  - service description
  - service avaiablilities
  - general terms and conditions
  - service level objectives
    - uptime percentage
    - response times

## Operations

### Generating public keys

#### Key-pairs for `user` and `cloud service provider`

Each client is responsible for generating and maintaining three long-term keypairs:

- Encrypting key pair: `encPublicKey`, `encPrivateKey`

  - usage: offline service agreement negotiation
  - `encPublicKey`: an ECDH public key
  - `encPrivateKey`: an ECDH private key

- Off-chain Signing key pair: `offChainSigPublicKey`, `offChainsigPrivateKey`

  - usage: offline service agreement signing
  - `offChainSigPublicKey`: an P256 ecdsa public key
  - `offChainsigPrivateKey`: an P256 ecdsa key

- On-chain Signing key pair:
  `onChainSigPublicKey`, `onChainSigPrivateKey`

  - usage: on chain crypto wallet, used submitting transactions
  - `offChainSigPublicKey`: an P256 ecdsa public key
  - `offChainsigPrivateKey`: an P256 ecdsa key

#### Key-pairs for `smart contract deployer` and `maintainer`

- On-chain Signing key pair:
  `onChainSigPublicKey`, `onChainSigPrivateKey`

  - usage: on chain crypto wallet, used submitting transactions
  - `offChainSigPublicKey`: an P256 ecdsa public key
  - `offChainsigPrivateKey`: an P256 ecdsa key

#### Key-pairs for `third-party data oracle`

- On-chain Signing key pair:
  `onChainSigPublicKey`, `onChainSigPrivateKey`

  - usage: on chain crypto wallet, used submitting transactions
  - `offChainSigPublicKey`: an P256 ecdsa public key
  - `offChainsigPrivateKey`: an P256 ecdsa key

### Negotiating Service Level Agreement

To make it simple and easy to quantify for the v1 design, these are the inputs that are negotiable between the user and cloud service provider.

- `effective date`: The start day when a contract or rule begins to apply.
- `expiration date`: The end day when a contract or rule stops being in effect.
- `uptime percentage`: How often a service (like a website) is working properly. Higher numbers mean it hardly ever has problems.
- `availability percentage`: Similar to uptime, it shows how often you can use a service, but it considers times when the service is purposely not available, like during scheduled maintenance.
- `api rate limiting`: Rules that limit how many times you can ask for data from a service in a certain period, like 1000 times an hour, to prevent overload.

An example for the service level agreement:

```json=
{
  "SLA": {
    "serviceProvider": "XYZ Corporation",
    "client": "ABC Company",
    "effectiveDate": "2024-03-01", //negotiable
    "expirationDate": "2025-02-28", //negotiable
    "servicesProvided": {
        "serviceName": "Cloud Hosting Service",
        "description": "Providing secure and scalable cloud hosting solutions.",
        "availability": "99.9%" // negotiable
    },
    "performanceMetrics": {
      "uptime": "99.9%", //negotiable
      "responseTime": "Less than 1 hour during business hours",
      "resolutionTime": "Less than 72 hours for critical issues"
    },
    "apiRateLimit": {
      "limit": "1000 requests per hour", //negotiable
      "description": "Specifies the maximum number of API requests that a client can make in an hour. Exceeding this limit may result in temporary suspension of service access.",
      "resetInterval": "Hourly",
      "exceedingPenalties": "Temporary suspension of API access for 1 hour, notification sent to client's contact email",
      "measurementMethod": "Requests are counted per access token issued to the client."
    },
    "responsibilities": {
      "serviceProvider": [
        "Maintain service uptime as per the agreement",
        "Provide customer support as specified",
        "Notify client of scheduled maintenance"
      ],
      "client": [
        "Pay service fees as agreed",
        "Use services according to the terms of service",
        "Report issues in a timely manner"
      ]
    },
    "revisionPolicy": {
      "noticePeriod": "30 days",
      "methodOfNotification": "Email"
    }
  }
}
```

### Negotiating Penalty Clause

To make it simple and easy to quantify for the v1 design, these are the penalty conditions that are negotiable between the user and cloud service provider.

- uptime
  - guaranteed uptime is same as the SLA
  - penalty format: credit of x% of monthly fee for each y% below agreed uptime
- availability
  - guaranteed availability is same as the SLA
  - penalty format: credit of x% of monthly fee for each hour above agreed availability time
- api rate limiting
  - guaranteed availability is same as the SLA
  - penalty format: credit of x% of monthly fee per additional violation beyond the second in a single month

An example for the service level agreement:

```json=
{
  "penalties": [
      {
        "condition": "uptime less than 99.9%",
        "penalty": "credit of 5% of monthly fee for each 0.1% below agreed uptime"
      },
      {
        "condition": "availability time less than 97%",
        "penalty": "credit of 1% of monthly fee for each hour above agreed availability time"
      },
      {
        "condition": "api rate limit exceeded more than twice in a month",
        "penalty": "credit of 2% of monthly fee per additional violation beyond the second in a single month"
      }
    ],
}
```

## ZK Circuits

### inputs to ZK proof

#### private inputs:

- pricing information
- private contents in terms of service agreement
  - uptime percentage
  - availability percentage
  - api rate limiting
- private contents in terms of penalty clause
- service nullifier: differentiating both parties could set up multiple services, but these services are unique
- service hash
- container hash

#### public inputs:

- identity of user
- identity of cloud service provider
- hash(public contents within service agreement)
  - effective date
  - expiration date
- hash(public contents within service agreement)
- metadata of the service agreement order book

#### Contract check

- `effective date < operation date < expiration date`: The date when the service or operation is happening (Operation Date) must be after the date the agreement starts (Effective Date) and before the date the agreement ends (Expiration Date).
- `actual uptime percentage > negotiated uptime percentage`: The real amount of time the service is up and running properly (Actual Uptime Percentage) should be higher than what was promised or agreed upon in the contract (Negotiated Uptime Percentage).
- `actual availability percentage > negotiated availability percentage`:The actual measure of how often the service is available for use (Actual Availability Percentage) needs to be greater than the level of availability agreed upon in the contract (Negotiated Availability Percentage).
- `actual rate limit < negotiated rate limit`: The real limit on how many times the service can be used or accessed (Actual Rate Limit) should be less strict than what was set in the agreement (Negotiated Rate Limit).
- other conditions: ...

## Service Agreement Order Book

The order book is designed for the user to act as the taker, taking this order and paying the blockchain transaction fee.

```json=
{
    "order": {
        "cspSignature": "3045022100afb03175dbc5b1ac3638621bf4e652b3afa87f2c7e8b1d8f55eb58ec3af90bfb022068d8a585189785e9d9460d4a54098ebe7fdfe93a277660020ec0f277afe6e289",
        "usrSignature": "3044022053e49c6ab37c5f52daa59d211e060b3aae34834fb44aa45f1bdd375972c6f80d022063a3151a7256d3c9885a276e0b78172874f037cc50a2b792621d8f872aa73a6d",
        "maker": "0x56eb0ad2dc746540fab5c02478b31e2aa9ddc38c", // cloud service provider id
        "taker": "0x27DE3fd75B0540DB22E41038ac692116e0dfea0B", // user id, 0 hash means anyone can fullfil
        "makerServiceHash": "dab8bff7c90f990c8edc29dd455b85dca5615f3dc3c5e56c0dcd5c4478dd83c1",
        "takerContainersHash": "3356b94b5b51ff487099ce4d684ed5d9613e0a76ff5f2acf3cb4d1fd41e47670",
        "agreementCommitment": "8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92",
        "penaltyCommitment": "c0034605ea413370d5ad022b8d2f7fe33461bf6d7e5f4ac78f02c27b793673c9",
        "fullfilAmount": "10000000",
        "fullfilToken": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
        "salt": "40584472803756371677282334946406041345967204972423156532532776379801646390127",
        "verifyingContract": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
        "expiry": "1614959239",
    },
    "metaData": {
        "orderHash": "0xe4b47eb16a24c7e2de185e64ad9234b13052f1f60a0a7c108f3ec0feec3cf8ac",
        "createdAt": "2021-03-05T15:32:18.473Z"
    }
}
```

## Next Steps

1. provide a circuit template, smart contracts
2. provide a PoC code for service agreement
3. Keep refactoring the problem and possible solutions
