# ledger-app-one 
Ledger hardware wallet support for Harmony ONE.  
This repository contains software and firmware source code for Harmony hardware wallet using Ledger Nano S.  
An example of Ledger Nano S running Harmony ONE app (LED UI) and companion app manual can be found  [here](https://docs.harmony.one/sdk-wiki/wallet-developers-guide/ledger).


## Step 0: Update your Ledger Nano S Firmware to ver 1.6.0 and Load app onto Ledger Nano S

Before attempting to load the hex file, make sure your Ledger Nano S 
is connected and the firware is updated to version [1.6.0](https://support.ledgerwallet.com/hc/en-us/articles/360002731113-Update-the-firmware).

Enter your PIN and **make sure you're seeing the Dashboard app

## Step 1: Install Ledger Firmware Tootls 

#### Install virtualenv
```bash
[sudo] pip install -U setuptools
[sudo] pip install virtualenv
```

#### Create new virtualenv
#### linux dependencies for ledgerblue module
> libudev1 libudev-dev libusb-1.0-0-dev

```bash
virtualenv -p python3 venv
source venv/bin/activate
pip install ledgerblue
or pip install git+https://github.com/LedgerHQ/blue-loader-python.git 
```

## Step 2: Install the Pre-Build release version from github release/ver1_app.hex

After checkout the code from github and upgrade Ledger Nano S firmware to version 1.6.0, you can directly install the pre-build firmware using the following.

```bash
sudo venv/bin/python -m ledgerblue.loadApp --appFlags 0x40 --path "44'/1023'" --curve secp256k1 --tlv --targetId 0x31100004 --targetVersion="1.6.0" --delete --fileName release/ver1_app.hex --appName One --appVersion 1.0.0 --dataSize 0 --icon 01ffffff00ffffff00ffffffffffffc7e1bbcdbbddbbcdbbc50bd8a3ddbbddbbddb3edc7e3ffffffff
```

 
## Build firmware yourself

If you want to build the firmware from source code yourself, please following the instructions below.


### Code List
```
  oneledger.go   simple companion host side app for testing Ledger Nano S firmware
  src/           firmware source code for Ledger Nano S
  wallet/        harmony-one hardware wallet main application code
  icos/          gif files for Ledger Nano S app icon and Ledger Live icon
  glyphs/        icon files used in firmware GUI
```

### Docker toolchain image
In order to make compiling as early as possible you can make use of a docker image containing all the necessary compilers and the [nanos-secure-sdk](https://github.com/LedgerHQ/nanos-secure-sdk).

Make sure you have [Docker](https://www.docker.com/community-edition) installed.

#### Step 1 - Pull docker image from docker.io:

```bash
docker pull coolcottontail/harmony:ledger-env
```

#### Step 2 - Use Docker image
if you pull the image from docker.io, then use the following command:
```bash
docker run --rm -v "$(pwd)":/one-ledger -w /one-ledger coolcottontail/harmony:ledger-env make
```

After the build, the firmware created as bin/app.hex 

## Load app onto Ledger Nano S

Before attempting to load the hex file, make sure your Ledger Nano S 
is connected and the firware is updated to the [latest version](https://support.ledgerwallet.com/hc/en-us/articles/360002731113-Update-the-firmware).

Enter your PIN and **make sure you're seeing the Dashboard app**.


### Using Docker image

If you run into errors here, make sure you have the required dependencies installed. See [Ledger - Loader Python](https://github.com/LedgerHQ/blue-loader-python).

This step will create a directory venv and put all python related modules there.

#### Step 3 - Load HEX file
This step requires sudo permission
```bash
./load_fw.sh
```

this bash actually execute the following:
```bash
sudo ./venv/bin/python -m ledgerblue.loadApp --appFlags 0x40 --path "44'/1023'" --curve secp256k1 --tlv --targetId 0x31100004 --targetVersion="1.6.0" --delete --fileName bin/app.hex --appName One --appVersion 1.0.0 --dataSize $((0x`cat debug/app.map |grep _envram_data | tr -s ' ' | cut -f2 -d' '|cut -f2 -d'x'` - 0x`cat debug/app.map |grep _nvram_data | tr -s ' ' | cut -f2 -d' '|cut -f2 -d'x'`)) `ICONHEX=\`python3 /opt/bolos/nanos-secure-sdk/icon3.py --hexbitmaponly nanos_app_one.gif  2>/dev/null\` ; [ ! -z "$ICONHEX" ] && echo "--icon $ICONHEX"`
```

## Build host companion app
This is only for testing the ledger firmware. If you use javascript, then don't need to do this step.
```
go build oneledger.go
```
