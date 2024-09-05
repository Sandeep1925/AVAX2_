
# Creating a Custom Subnet with Avalanche HyperSDK

## Project Overview

This guide walks you through the process of utilizing the Avalanche HyperSDK to create a customized virtual machine and subnet on the Avalanche platform. HyperSDK enables developers to design and tailor blockchains for specific purposes such as token creation, transfers, and asset management.

## Features

1. Custom Virtual Machine: Build and customize a blockchain using HyperSDK.
2. Token Management: Define and manage rules for token creation, minting, and transfers.
3. Order Book Management: Create and handle an order book for asset trading.

## Objective

Our startup aims to develop a custom virtual machine to facilitate token minting and transfer operations. HyperSDK offers the flexibility needed to construct a blockchain with precise functionality, including token management and asset trading.

## Important Note

HyperSDK is in its alpha phase. For stability, we are using the example token VM, which is among the few tested and supported configurations. Keep an eye on the repository for updates.

## Getting Started

### Prerequisites

Ensure that Go is installed on your system.

### Installation

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/Metacrafters/tokenvm.git

2. **Normalize Dependencies:**

   ```bash
   go mod tidy

### Configure Project Constants:


Edit `consts/consts.go` (same as below) to define the constants and initialize necessary values for your project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package consts

import (
	"github.com/ava-labs/avalanchego/ids"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"
	"github.com/ava-labs/hypersdk/consts"
)

const (
	// Human-readable part for your hyperchain
	HRP = "sandeepChain"
	// Name for your hyperchain
	Name = "sandeepkaur"
	// Token symbol for your hyperchain
	Symbol = "SDK"
)

var ID ids.ID

func init() {
	b := make([]byte, consts.IDLen)
	copy(b, []byte(Name))
	vmID, err := ids.ToID(b)
	if err != nil {
		panic(err)
	}
	ID = vmID
}

// Instantiate registry here so it can be imported by any package. We set these
// values in [controller/registry].
var (
	ActionRegistry *codec.TypeParser[chain.Action, *warp.Message, bool]
	AuthRegistry   *codec.TypeParser[chain.Auth, *warp.Message, bool]
)

```
### Register Actions:


Edit `registry/registry.go` (same as below) to register actions and initialize registries for your custom blockchain project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package registry

import (
	"github.com/ava-labs/avalanchego/utils/wrappers"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"

	"tokenvm/actions"
	"tokenvm/auth"
	"tokenvm/consts"
)

// Setup types
func init() {
	consts.ActionRegistry = codec.NewTypeParser[chain.Action, *warp.Message]()
	consts.AuthRegistry = codec.NewTypeParser[chain.Auth, *warp.Message]()

	errs := &wrappers.Errs{}
	errs.Add(
		// When registering new actions, ALWAYS make sure to append at the end.
		consts.ActionRegistry.Register(&actions.Transfer{}, actions.UnmarshalTransfer, false),

		// Register the CreateAsset action
		consts.ActionRegistry.Register(&actions.CreateAsset{}, actions.UnmarshalCreateAsset, false),
		// Register the MintAsset action
		consts.ActionRegistry.Register(&actions.MintAsset{}, actions.UnmarshalMintAsset, false),

		consts.ActionRegistry.Register(&actions.BurnAsset{}, actions.UnmarshalBurnAsset, false),
		consts.ActionRegistry.Register(&actions.ModifyAsset{}, actions.UnmarshalModifyAsset, false),

		consts.ActionRegistry.Register(&actions.CreateOrder{}, actions.UnmarshalCreateOrder, false),
		consts.ActionRegistry.Register(&actions.FillOrder{}, actions.UnmarshalFillOrder, false),
		consts.ActionRegistry.Register(&actions.CloseOrder{}, actions.UnmarshalCloseOrder, false),

		consts.ActionRegistry.Register(&actions.ImportAsset{}, actions.UnmarshalImportAsset, true),
		consts.ActionRegistry.Register(&actions.ExportAsset{}, actions.UnmarshalExportAsset, false),

		// When registering new auth, ALWAYS make sure to append at the end.
		consts.AuthRegistry.Register(&auth.ED25519{}, auth.UnmarshalED25519, false),
	)
	if errs.Errored() {
		panic(errs.Err)
	}
}

```
### Run the Virtual Machine Locally:

Ensure Go is on your path. If not, run:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```
### Execute the following commands to run and build the VM:

```bash
MODE="run-single" ./scripts/run.sh
./scripts/build.sh
```
### Load Demo Private Key:

Import the demo private key included in the project:

```bash
./build/token-cli key import demo.pk
./build/token-cli chain import-anr
```
### Interact with Your HyperChain:

### Mint and Trade
#### Step 1: Create Your Asset
First up, let's create our own asset. You can do so by running the following
command from this location:
```bash
./build/token-cli action create-asset
```

When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2qBqBsx7VyNCSeSHw8vPgHK4iFtLBzrCppsVQEE4YRmtVBpBXF
metadata (can be changed later): TKN
continue (y/n): y
✅ txID: SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
```
_`txID` is the `assetID` of your new asset._

The "loaded address" here is the address of the default private key (`demo.pk`). We
use this key to authenticate all interactions with the `tokenvm`.

#### Step 2: Mint Your Asset

After we've created our own asset, we can now mint some of it. You can do so by
running the following command from this location:
```bash
./build/token-cli action mint-asset
```

When you are done, the output should look something like this (usually easiest
just to mint to yourself).
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2qBqBsx7VyNCSeSHw8vPgHK4iFtLBzrCppsVQEE4YRmtVBpBXF
assetID: SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
metadata: MyCoin supply: 0
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 10000
continue (y/n): y
✅ txID: X1E5CVFgFFgniFyWcj5wweGg66TyzjK2bMWWTzFwJcwFYkF81
```

### Transfer Assets to Another Subnet

Unlike the mint and trade demo, the AWM demo only requires running a single
command. You can kick off a transfer between the 2 Subnets you created by
running the following command from this location:

```bash
./build/token-cli action export
```

When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2qBqBsx7VyNCSeSHw8vPgHK4iFtLBzrCppsVQEE4YRmtVBpBXF
✔ assetID (use TKN for native token): SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
balance: 1000 SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 10
reward: 0
available chains: 1 excluded: [Em2pZtHr7rDCzii43an2bBi1M2mTFyLN33QP1Xfjy7BcWtaH9]
0) chainID: cKVefMmNPSKmLoshR15Fzxmx52Y5yUSPqWiJsNFUg1WgNQVMX
destination: 0
swap on import (y/n): n
continue (y/n): y
✅ txID: 24Y2zR2qEQZSmyaG1BCqpZZaWMDVDtimGDYFsEkpCcWYH4dUfJ
perform import on destination (y/n): y
22u9zvTa8cRX7nork3koubETsKDn43ydaVEZZWMGcTDerucq4b to: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp source assetID: TKN output assetID: 2rST7KDPjRvDxypr6Q4SwfAwdApLwKXuukrSc42jA3dQDgo7jx value: 10000000000 reward: 10000000000 return: false
✔ switch default chain to destination (y/n): y
```

_The `export` command will automatically run the `import` command on the
destination. If you wish to import the AWM message using a separate account,
you can run the `import` command after changing your key._


### Closing the Local Avalanche Network:
To shut down the local Avalanche network, run:

```bash
killall avalanche-network-runner

```
### CONCLUSION

We have successfully created a custom virtual machine to handle token minting and transfers. By using HyperSDK, you can further tailor the blockchain to meet your specific requirements.

## Authors

Sandeep Kaur @Sandeep1925

## License

This project is licensed under the MIT License - see the LICENSE.md file for details.
