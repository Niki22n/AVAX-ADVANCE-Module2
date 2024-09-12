# Creating a Custom Subnet with Avalanche HyperSDK
## Project Overview
This guide walks you through the process of utilizing the Avalanche HyperSDK to create a customized virtual machine and subnet on the Avalanche platform. HyperSDK enables developers to design and tailor blockchains for specific purposes such as token creation, transfers, and asset management.

## Features
Custom Virtual Machine: Build and customize a blockchain using HyperSDK.
Token Management: Define and manage rules for token creation, minting, and transfers.
Order Book Management: Create and handle an order book for asset trading.
## Objective
Our startup aims to develop a custom virtual machine to facilitate token minting and transfer operations. HyperSDK offers the flexibility needed to construct a blockchain with precise functionality, including token management and asset trading.

## Important Note
HyperSDK is in its alpha phase. For stability, we are using the example token VM, which is among the few tested and supported configurations. Keep an eye on the repository for updates.

## Getting Started
### Prerequisites
Ensure that Go is installed on your system.

### Installation
Clone the Repository:

git clone https://github.com/Metacrafters/tokenvm.git
Normalize Dependencies:

go mod tidy
Configure Project Constants:
Edit consts/consts.go (same as below) to define the constants and initialize necessary values for your project.

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
	HRP = "nikita"
	// Name for your hyperchain
	Name = "nikitaChain"
	// Token symbol for your hyperchain
	Symbol = "NKTA"
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
Register Actions:
Edit registry/registry.go (same as below) to register actions and initialize registries for your custom blockchain project.

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

## Run the Virtual Machine Locally:
Ensure Go is on your path. If not, run:

export PATH=$PATH:$(go env GOPATH)/bin
Execute the following commands to run and build the VM:
MODE="run-single" ./scripts/run.sh
./scripts/build.sh
Load Demo Private Key:
Import the demo private key included in the project:

./build/token-cli key import demo.pk
./build/token-cli chain import-anr

## Interact with hyperchain
### Mint and Trade
#### Step 1: Create Your Asset
First up, let's create our own asset. You can do so by running the following command from this location:

./build/token-cli action create-asset
When you are done, the output should look something like this:

```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: Em2pZtHr7rDCzii43an2bBi1M2mTFyLN33QP1Xfjy7BcWtaH9
metadata (can be changed later): MarioCoin
continue (y/n): y
✅ txID: 27grFs9vE2YP9kwLM5hQJGLDvqEY9ii71zzdoRHNGC4Appavug
```
txID is the assetID of your new asset.

The "loaded address" here is the address of the default private key (demo.pk). We use this key to authenticate all interactions with the tokenvm.

#### Step 2: Mint Your Asset
After we've created our own asset, we can now mint some of it. You can do so by running the following command from this location:

./build/token-cli action mint-asset
When you are done, the output should look something like this (usually easiest just to mint to yourself).
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: Em2pZtHr7rDCzii43an2bBi1M2mTFyLN33QP1Xfjy7BcWtaH9
assetID: 27grFs9vE2YP9kwLM5hQJGLDvqEY9ii71zzdoRHNGC4Appavug
metadata: MarioCoin supply: 0
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 15000
continue (y/n): y
✅ txID: X1E5CVFgFFgniFyWcj5wweGg66TyzjK2bMWWTzFwJcwFYkF72
```
#### Step 3: View Your Balance
Now, let's check that the mint worked right by checking our balance. You can do so by running the following command from this location:

./build/token-cli key balance
When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: Em2pZtHr7rDCzii43an2bBi1M2mTFyLN33QP1Xfjy7BcWtaH9
assetID (use TKN for native token): 27grFs9vE2YP9kwLM5hQJGLDvqEY9ii71zzdoRHNGC4Appavug
metadata: MarioCoin supply: 10000 warp: false
balance: 15000 27grFs9vE2YP9kwLM5hQJGLDvqEY9ii71zzdoRHNGC4Appavug
```

Closing the Local Avalanche Network:
To shut down the local Avalanche network, run:

killall avalanche-network-runner


## CONCLUSION
We have successfully created a custom virtual machine to handle token minting and transfers. By using HyperSDK, you can further tailor the blockchain to meet your specific requirements.

Authors
Nikita

License
This project is licensed under the MIT License - see the LICENSE.md file for details.
