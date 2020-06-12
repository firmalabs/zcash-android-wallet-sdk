# Module android-sdk

This lightweight SDK connects Android to Zcash. It welds together Rust and Kotlin in a minimal way, allowing third-party Android apps to send and receive shielded transactions easily, securely and privately.

### Overview

At a high level, this SDK simply helps native Android codebases connect to Zcash's Rust crypto libraries without needing to know Rust or be a Cryptographer. Think of it as welding. The SDK takes separate things and tightly bonds them together such that each can remain as idiomatic as possible. Its goal is to make it easy for an app to incorporate shielded transactions while remaining a good citizen on mobile devices.

Given all the moving parts, making things easy requires coordination. The [Synchronizer](cash.z.ecc.android.sdk/-synchronizer/index.html) provides that layer of abstraction so that the primary steps to make use of this SDK are simply:

1. Start the [Synchronizer](cash.z.ecc.android.sdk/-synchronizer/index.html)
2. Subscribe to wallet data

The [Synchronizer](cash.z.ecc.android.sdk/-synchronizer/index.html) takes care of

    - Connecting to the light wallet server
    - Downloading the latest compact blocks in a privacy-sensitive way
    - Scanning and trial decrypting those blocks for shielded transactions related to the wallet
    - Processing those related transactions into useful data for the UI
    - Sending payments to a full node through the light wallet server
    - Monitoring sent payments for status updates

To accomplish this, these responsibilities of the SDK are divided into separate components. Each component is coordinated by the [Synchronizer](docs/-synchronizer/README.md), which is the thread that ties it all together.



# Package cash.z.ecc.android.sdk
 The primary classes of the SDK such as the [Synchronizer](cash.z.ecc.android.sdk/-synchronizer/index.html) and the [Initializer](cash.z.ecc.android.sdk/-initializer/index.html) used to construct it.

# Package cash.z.ecc.android.sdk.annotation
Annotations that help with testing.

# Package cash.z.ecc.android.sdk.block
 Classes that store, process or download CompactBlocks.

# Package cash.z.ecc.android.sdk.db
The databases that support the SDK such as the store for CompactBlocks.

# Package cash.z.ecc.android.sdk.db.entity
 The classes that model the database tables.

# Package cash.z.ecc.android.sdk.exception
A single file containing all the typed exceptions thrown by the SDK.

# Package cash.z.ecc.android.sdk.ext
Extension functions that support development with the SDK such as currency formatters like [toZec](cash.z.ecc.android.sdk.ext/kotlin.-double/to-zec.html) and [toZecString](cash.z.ecc.android.sdk.ext/kotlin.-double/to-zec-string.html) which work with BigDecimals to produce properly rounded values.

# Package cash.z.ecc.android.sdk.ext.android
Extension functions that represent missing functionality in Android that is helpful for the SDK such as [FlowPagedListBuilder](cash.z.ecc.android.sdk.ext.android/-flow-paged-list-builder/index.html) which is scheduled to be available in the Android SDK, soon.

# Package cash.z.ecc.android.sdk.jni
Java Native Interface classes for integrating with Rust such as [RustBackendWelding](cash.z.ecc.android.sdk.jni/-rust-backend-welding/index.html) which defines the contract with Rust and [RustBackend](cash.z.ecc.android.sdk.jni/-rust-backend/index.html) which implements it.

# Package cash.z.ecc.android.sdk.service
Classes related to interacting with lightwalletd, over grpc.

# Package cash.z.ecc.android.sdk.transaction
Classes that are related to storing and manipulating transactions.

# Package cash.z.ecc.android.sdk.validate
 Helper types that facilitate validation such as [AddressType](cash.z.ecc.android.sdk.validate/-address-type/index.html), which can be used to indicate the type of address and its validity.
