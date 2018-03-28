# Create a cryptocurrency 101

## Notes

* This is a work in progress
* The goal of this guide is to create:
  * A premined cryptocurrency
  * node
  * simplewallet
  * walletd
  * miner
  * GUI wallet
* You need at least two Ubuntu 16.04 servers before you start to act as seed nodes

## On one of the nodes

	1. sudo apt-get update
	2. sudo apt-get upgrade
	3. wget https://github.com/forknote/forknote/releases/download/2.1.2/forknote-linux.tar.gz
	4. tar -zxvf forknote-linux.tar.gz
	5. cd forknote-linux
	6. mkdir configs
	7. Generate a new config file at http://forknote.net/create/ for your coin.
		1. Do not forget to add your seed nodes to the configuration file.
		2. Leave the premined address blank.
		3. Leave the genesis block blank.
	8. After generating the config file, manually edit the following values:
		1. Set the value of "UPGRADE_HEIGHT_V3" to "2" *
		2. Add ZAWY_DIFFICULTY_BLOCK_INDEX to "10" *
	9. Add your configuration file to the "configs" folder.
	10. ./simplewallet --config-file configs/fakecoin.conf --generate-new-wallet genesis.wallet --password 12345
		1. Copy new wallet address (For example, FSgLDzpszX3S3RMK5p8PfCeY1eqchFSZsf5LcgPsA4EcJ6wpXrKGr7AViBqLATZ9K6CqQPgR8opQq6zY2HTCsVuWEANv6pq)
		2. Type the "exit" command to close the simplewallet.
	11. ./forknoted --config-file configs/fakecoin.conf --print-genesis-tx --genesis-block-reward-address FSgLDzpszX3S3RMK5p8PfCeY1eqchFSZsf5LcgPsA4EcJ6wpXrKGr7AViBqLATZ9K6CqQPgR8opQq6zY2HTCsVuWEANv6pq
	12. Copy genesis block and add it to configs/fakecoin.conf, it should be without spaces or new lines, if there is a space in the genesis block you did something wrong.

## Now, on the same node

	1. git clone https://github.com/forknote/cryptonote-generator
	2. cd cryptonote-generator
	3. bash install_dependencies.sh
	4. sudo apt-get -y install libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev (@TODO: Merge pull request on forknote repository) *
	5. Create fakecoin.json, add the GENESIS_COINBASE_TX_HEX that you created in previous steps and make sure that your .conf and .json configuration file has the exact same values.
	6. In the extensions section of the .json, add any additional extension you need on your coin (You can see the extensions at https://github.com/forknote/cryptonote-generator/tree/master/extensions) *
	7. Add fakecoin.json to generator/configs
	8. bash generator.sh -f configs/fakecoin.json -c '-j2'
	9. After the generation process is finished, use Cyberduck or any other SFTP client to transfer generated_files/builds/yourfakecoin.tar.gz to the second node

## On both nodes

	1. tar -zxvf yourfakecoin.tar.gz
	2. nohup ./fakecoind & (@TODO: Demonize the seed node)

## Finally, on your local computer

	1. If you have Ubuntu/OSX locally, use Cyberduck or any other SFTP client to download generated_files/builds/yourfakecoin.tar.gz
	2. Run the generated daemon and wait until it syncs
		1. ./fakecoind
	3. To start mining
		1. ./miner --address FSgLDzpszX3S3RMK5p8PfCeY1eqchFSZsf5LcgPsA4EcJ6wpXrKGr7AViBqLATZ9K6CqQPgR8opQq6zY2HTCsVuWEANv6pq
	4. Use simplewallet to use your wallet
		1. /simplewallet

## Genesis block reward address 
	1. You need to use the --SYNC_FROM_ZERO command in your simplewallet to sync from block 0 (genesis block) to see the reward. Remember, you need to be past block 10 and mine at least one block in your reward address.

## To compile your coin on Ubuntu 16.04

	1. sudo apt-get -y install build-essential python-dev gcc-4.9 g++-4.9 git cmake libboost1.58-all-dev librocksdb-dev libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
	2. rm -rf build; mkdir -p build/release; cd build/release
	3. cmake -D STATIC=ON -D ARCH="default" -D CMAKE_BUILD_TYPE=Release ../..
	4. PORTABLE=1 make
  
## To compile your coin on Windows 10

	1. You need to have the following dependencies installed:
		1. Windows 10
		2. Boost 1.59
		3. Visual Studio 2017
	2. mkdir build
	3. cd build
	4. cmake.exe -DBOOST_ROOT=C:\boost_1_59_0 -DBOOST_LIBRARYDIR=C:\boost_1_59_0\libs -G "Visual Studio 15 Win64" ..
	5. Open Bytecoin.snl in Visual Studio 2017 and change target to "Release" "x64"
	6. From the Solution explorer, build "external" and then build "ALL_BUILD"
	7. You will find your compiled files at build/src/Release (yourcoind.exe, miner.exe, simplewallet.exe and walletd.exe)

## QandA

### Q: How do I get the genesis block reward?
* A: You need to start mining, by default you will get the reward after mining past block 10

### Q: How do I speed up the reward of the geneis block?
* A: Edit the value of CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW

### Q: Do you need to send more initial coins?
* A: Edit the value of MAX_BLOCK_SIZE_INITIAL

### Q: Need a reference repo?
* A: Check out https://github.com/inbestcoin/inbestcoin and https://github.com/inbestcoin/inbestcoin-gui

### Q: What extenions are recommended for my coin?
* A: Depends on your coin, but these should do the trick:
  * "core/bytecoin.json"
  * "versionized-parameters.json"
  * "print-genesis-tx.json"
  * "genesis-block-reward.json"
  * "bug-fixes.json"
  * "zawy-difficulty-algorithm.json"
  * "enable-cors.json"
  * "blockchain-explorer.json"
  
### Q: What is mixin?
* See this for a comprenhensive explanation: https://monero.stackexchange.com/questions/3308/what-is-a-mixin-and-how-does-it-work

### Q: How to fix "Wallet Sync Issue [Proof of work too weak for block]"?
* Follow @zmrhab fix for this: https://github.com/forknote/forknote/issues/41#issuecomment-361446961

### Q: Why there is a sudded spike of difficulty mining the first blocks?
* There is a bug that makes the difficulty of 5th or 6th block very high if the miner mines the first blocks very fast. A workaround now is to stop the miner right after you start it, before it mines 4 blocks. Then wait a few seconds and then relaunch it.

## To create a GUI for your coin (work in progress)

### To compile it under Windows 10

	1. Fix repo that references a submodule that doesn't exist
	2. mkdir build
	3. cd build
	4. cmake.exe -DBOOST_ROOT=C:\boost_1_59_0 -DBOOST_LIBRARYDIR=C:\boost_1_59_0\libs\ -DCMAKE_PREFIX_PATH=D:\Qt\5.10.0\msvc2015_64 -G "Visual Studio 15 Win64" ..
	5. Open fakecoin.sln in Visual Studio 2017
	6. Change to release and compile solution
	
### To compile it under OS X

----------
MAC Wallet
#mac
#intensecoin
-----------
xcode-select --install
brew install cmake
brew install boost
brew install qt


Download YourcoinCode
Download Intensecoin Code: (We only need Wallet, PaymentGateService, External folders from here)
REPLACE:
YourcoinCode/Wallet, 
YourcoinCode/PaymentGateService, 
YourcoinCode/External
WITH:


Intensecoin/Wallet, 
Intensecoin/PaymentGateService, Intensecoin/External


Download IntencoinWallet GUI
cd IntencoinWallet
rm -Rf cryptonote
ln -s ../YourcoinCode cryptonote

mkdir build && cd build && cmake -DSTATIC=1 -DCMAKE_PREFIX_PATH=/usr/local/Cellar/qt/5.10.0_1 .. && make

cd Build
/usr/local/Cellar/qt/5.10.0_1/bin/./macdeployqt Intensecoin.app -dmg

## Files you need to modify in your coin to match intensecoingui and be able to compile under Windows 10

* external/rocksdb
* external/CMakeLists.txt
* src/PaymentGateService/PaymentGateService.cpp
* src/Wallet/WalletGreen.cpp
* src/Wallet/WalletGreen.h
* tests/CMakeLists.txt
* CMakeLists.txt

## Files you need to edit to change the ticker on intensecoingui

* src/CryptoNoteWrapper/CryptoNoteAdapter.cpp
* CryptoNoteWallet.cmake

## If you do a search and replace on intensecoingui (intensecoin to fakecoin and/or intense coin to fakecoin), you need to rename the following files

* src/images/intensecoin.icns to fakecoin.icns
* src/images/intensecoin.ico to fakecoin.ico
* src/images/intensecoin.png to fakecoin.png
* intensecoinwallet.desktop to fakecoinwallet.desktop
* intensecoinwallet.qss to fakecoinnwallet.qss

## Files you need to modify to change the branding of intensecoingui

* src/icons/logo.png
* src/icons/logo_bl.png
* src/images/cryptonote.ico
* src/images/cryptonote.icns
* src/images/cryptonote.png
* src/images/intensecoin.ico
* src/images/intensecoin.icns
* src/images/intensecoin.png
* src/images/splash.png
