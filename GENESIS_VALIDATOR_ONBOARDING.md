# Genesis Validator Onboarding Guide for Pastel Network 2.0 (Lumera Network)

## Introduction

This guide provides step-by-step instructions for validators who wish to join Pastel Network 2.0 (Lumera Network) at genesis. Please follow each step carefully to ensure a smooth onboarding process.

## Step 1: Prerequisites

### System Requirements

Make sure your system meets the following prerequisites to onboard as a Genesis Validator. To run Validator in production follow refer to the [Validator Guide](VALIDATOR_GUIDE.md).

  - **CPU**: Minimum 8 cores with x86_64 architecture
  - **RAM**: At least 32 GB
  - **Storage**: Minimum of 100 GB available
  - **Operating System**: Ubuntu 22.04 LTS or higher

### Install the `pasteld` binary

```shell
wget https://github.com/innovatingcapital/pastel/releases/download/latest/pastel_latest_linux_amd64.tar.gz
tar xzvf pastel_latest_linux_amd64.tar.gz
chmod +x pasteld
sudo mv pasteld /usr/local/bin
```

### Prepare required validator information

- `MONIKER`: Your validator's moniker (e.g., `MyAwesomeValidator`).
- `CHAIN_ID`: The chain ID of the network (`pastel-mainnet-1` or `pastel-testnet-1`).
- `KEYNAME`: A name for your validator's key (e.g., `my_validator_key`).
- `AMOUNT`: The initial amount of tokens you will receive for genesis staking (Reach out to Pastel Network).
- `VAL_COMMISSION_RATE`: The initial commission rate for your validator (e.g., "0.05").
- `VAL_COMMISSION_MAX_RATE`: The maximum commission rate for your validator (e.g., "0.25").
- `VAL_COMMISSION_MAX_CHANGE_RATE`: The maximum change rate for your commission (e.g., "0.05").
- `MIN_SELF_DELEGATION`:
- `VAL_DETAILS`: Additional details about your validator (optional).
- `VAL_IDENTITY`: Your validator's identity.
- `VAL_SECURITY_CONTACT`: Your validator's security contact.
- `VAL_WEBSITE`: Your validator's website (optional).

> **IMPORTANT!!!**
> Contact the core team and request amount of tokens you would be allocated for genesis staking.

## Step 2: Clone the Repository and verify Genesis file

0. Set network you are setting validator for:
```shell
NETWORK="mainnet" OR "pastel-testnet-1"
```
OR
```shell
NETWORK="testnet"
```

2. Clone the repository to your local machine using the following command:
```shell
git clone <repository_url>
cd <repository_name>
```

```shell
pastel-networks/
├── config/
│   └── app.toml
└── docs/
│   └── VALIDATOR_GUIDE.md
├── mainnet/
│   ├── genesis.json
│   ├── genesis.asc
│   └── gentx/
├── mainnet/
│   ├── genesis.json
│   ├── genesis.asc
│   └── gentx/
├── create_genesis_hashes.sh
├── verify_genesis_hashes.sh
├── README.md
```

2. Verify the hashes of the genesis file
```shell
./create_genesis_hashes.sh $NETWORK
```

> If the hashes do not match, something is wrong with the genesis file. Please do not proceed until the genesis file hashes are valid. Contact the core team if you encounter this issue.

## Step 3: Prepare Your Validator Account and Initialize Chain

> **Important:** Do _not_ start the local network with `pasteld start` before completing this step. All the following commands must be executed before starting your validator node.

1. **Setup environment variables**
```shell
# Network Configuration
CHAIN_ID="pastel-$NETWORK-1"

# Validator Configuration
MONIKER="MyAwesomeValidator"
KEYNAME="my_validator_key"
AMOUNT="1000000upsl"
VAL_DETAILS="My amazing validator"
VAL_IDENTITY="A45BC..."
VAL_SECURITY_CONTACT="security@myvalidator.com"
VAL_WEBSITE="https://myvalidator.com"

# Commission Configuration
VAL_COMMISSION_RATE="0.10"
VAL_COMMISSION_MAX_RATE="0.25"
VAL_COMMISSION_MAX_CHANGE_RATE="0.05"
MIN_SELF_DELEGATION="1"
```

2. **Initialize the Chain:**
```shell
pasteld init $MONIKER --chain-id $CHAIN_ID
```

> **IMPORTANT!!!**
> This will create two important files:
> 	`$HOME/.pasteld/config/node_key.json`
> 	`$HOME/.pasteld/config/priv_validator_key.json`
> Keep them safe and secure. These files are required to run your validator node. Do not share these with anyone. Losing these files requires regeneration of your gentx.

3. **Copy Genesis File:**
```shell
cp $NETWORK/genesis.json $HOME/.pasteld/config
```

4. Before making changes, ensure the existing `genesis.json` file is valid:
```snell
pasteld genesis validate
```

> If `pasteld genesis validate` fails, something is wrong with the genesis file. Please do not proceed until the genesis file is valid. Contact the core team if you encounter this issue.

5. **Create a Validator Account:**
```shell
pasteld keys add $KEYNAME --keyring-backend file
```

> **IMPORTANT!!!**
> Safely store the mnemonic phrase displayed after the command. You will need this to recover your account.
> Remember (write down) password

6.  **Get Your Validator Address:**
```shell
VALIDATOR_ADDRESS=$(pasteld keys show $KEYNAME -a --keyring-backend file)
echo "Validator Address: $VALIDATOR_ADDRESS"
```

> Note this address for the next step and for your records

7. **Add your account to Genesis:**

> **IMPORTANT!!!**
> You are already supposed to contact the core team about amount of tokens you would use for genesis staking. If not - do it now
> Once amount is confirmed, add your account to genesis

```shell
pasteld genesis add-genesis-account $VALIDATOR_ADDRESS $AMOUNT --keyring-backend file
```

## Step 4: Create Your Validator Gentx

1. Generate a `gentx` file for your validator node using the following command:
```shell
pasteld genesis gentx $KEYNAME $AMOUNT \
	--chain-id=$CHAIN_ID \
	--moniker=$MONIKER \
	--commission-rate=$VAL_COMMISSION_RATE \
	--commission-max-rate=$VAL_COMMISSION_MAX_RATE \
	--commission-max-change-rate=$VAL_COMMISSION_MAX_CHANGE_RATE \
	--min-self-delegation=$MIN_SELF_DELEGATION \
	--details="$VAL_DETAILS" \
	--identity="$VAL_IDENTITY" \
	--security-contact="$VAL_SECURITY_CONTACT" \
	--website="$VAL_WEBSITE" \
	--keyring-backend=file
```

> **IMPORTANT!!!**
> This will create file:
> 	`$HOME/.pastel/config/gentx/gentx-<HEX>.json`

2. Validate updated `genesis.json` file
```snell
pasteld genesis validate
```

## Step 5: Submit Your gentx and Updated `genesis.json` file

1. Create a new branch in the repository based on the `main` branch:
```shell
git checkout -b validator-gentx-$MONIKER
```

2. Copy your gentx file to the `gentx` directory inside repository
```shell
cp `$HOME/.pastel/config/gentx/gentx-*.json $NETWORK/gentx`
cp `$HOME/.pastel/config/genesis.json $NETWORK/
```

3. Calculate new hash of `genesis.json`
```shell
./verify_genesis_hashes.sh $NETWORK
```

3. Commit changes:
```shell
git add $NETWORK/gentx/gentx-*.json $NETWORK/genesis.json $NETWORK/genesis.asc
git commit -m "Add gentx and account for $MONIKER"
git push origin validator-gentx-$MONIKER
```

5. Create a Pull Request (PR) to merge your branch into the `main` branch, using following PR template.
```md
## Validator Information
Moniker: $MONIKER

## Checklist
- [ ] I have backed up my validator keys securely
- [ ] I have stored my mnemonic phrase safely
- [ ] I have verified new genesis file is valid (i ran `pasteld genesis validate`)
- [ ] I have verified my gentx file is valid
- [ ] I created new hashes (i ran `./verify_genesis_hashes.sh $NETWORK`)
- [ ] I will be available during the network launch
```

## Step 7: Await Approval

1. Wait for the core team to review your PR and merge it to `main`.
2. Once merged, your gentx will be included in the final genesis file.

## Step 8: Validator Node Setup

Refer to the "Validator Operations Manual" for detailed instructions on how to set up and run your validator node.

## Important Notes

1. **Key Security**
   - Back up `$HOME/.pasteld/config/priv_validator_key.json`
   - Back up `$HOME/.pasteld/config/node_key.json`
   - Safely store your mnemonic phrase and password used for key ring
   - Never share your private keys

2. **Timeline**
   - PR submission deadline: [DEADLINE]
   - Genesis file publication: [GENESIS_DATE]
   - Network launch: [LAUNCH_DATE]

3. **Requirements**
   - All commits must be signed with GPG
   - GenTx must be valid and use the correct chain-id
   - Must use the specified denom and amounts
   - Must be available at network launch time

4. **Communication**
   - Join validator chat: [CHAT_LINK]
   - Monitor announcements: [ANNOUNCEMENT_CHANNEL]
   - Emergency contact: [EMERGENCY_CONTACT]

## Post-Submission

1. After your PR is merged:
   - Wait for final genesis file
   - Download and verify the genesis file checksum
   - Configure your node with final genesis
   - Be ready at the specified launch time

2. Launch Coordination:
   - Be online in validator chat at launch time
   - Follow launch coordinator instructions
   - Report any issues immediately

---
(C) 2024 Pastel Network
