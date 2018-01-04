---
layout: page
title: Install QMiner
---

### Prerequisites

**node.js v9.x, v8.x, v7.x, v6.x, v5.x, v4.x and npm 5.3 or higher**

To test that your node.js version is correct, run ```node --version``` and ```npm --version```. Not compatible with nodejs v0.12 or older.

**Windows**
- [Visual C++ Redistributable Packages for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145) download **vcredist_x64.exe** for node.js x64 or **vcredist_x86.exe** for node.js x86.

### Install

	npm install qminer

**Test**

	node -e "require('qminer'); console.log('OK')"


### Build from source

#### Linux

These instructions are for compiling, unit testing and debugging the C++ based node.js addon.
If you only want to use the qminer module, just follow the steps above.

**Prerequisites**

Basic development environment and git

	sudo apt-get install build-essential
	sudo apt-get install git

Python v2.7.3 is recommended. It was also tested on python 2.6. It does not work on python 3.

	sudo apt-get install python2.7

Node.js version v9.x, v8.x, v7.x, v6.x, v5.x or v4.x.


**Build**

	git clone https://github.com/qminer/qminer.git && cd qminer
	npm install --build-from-source

**Run unit tests (from qminer root folder)**

	npm test


#### Windows

These instructions are for compiling, unit testing and debugging the C++ based node.js addon. If you only want to use the qminer module, just follow the steps in README.md.

**Prerequisites**

- Python v2.7.3 is recommended. It was also tested on python 2.6.

- [Visual C++ Redistributable Packages for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145) download `vcredist_x64.exe` for Node.js x64 or `vcredist_x86.exe` for Node.js x86.

- Node.js version v9.x, v8.x, v7.x, v6.x, v5.x or v4.x. 

- Install all the required tools and configurations using Microsoft's windows-build-tools using `npm install --global --production windows-build-tools` from an elevated PowerShell or CMD.exe (run as Administrator).

**Build**

	git clone https://github.com/qminer/qminer.git && cd qminer
	npm install --build-from-source

After the first build you can open the Visual Studio solution file `qminer\build\binding.sln` and build from IDE.

**Run unit tests (from qminer root folder)**

	npm test

**Debug C++ code**

Instructions tested on Node 8.9.4. For other versions of node change the version accordingly in the instructions below.

Let's say our working directory is "C:\code". 
 - Download [https://nodejs.org/dist/v8.9.4/node-v8.9.4.tar.gz]
and extract the archive to `C:\code`. This will create the folder `C:\code\node-v8.9.4`
	
Build node:

	cd "C:\code\node-v8.9.4"
	vcbuild release x64 nosign
	vcbuild debug x64 nosign

Now when you open the solution (`binding.sln` in `qminer\build`), first select _Debug configuration_.
Then select `action_after_build` - _properties_ - _Debugging_ and set:
- Command: `c:\code\node-v8.9.4\build\Debug\node.exe`
- Command Arguments: `script.js`
- Working Directory: `path_to_script_folder`

**Building debug mode**

	npm install --build-from-source --debug --msvs_version=2015
	

These instructions are for compiling, unit testing and debugging the C++ based node.js addon. If you only want to use the qminer module, just follow the steps in README.md.

#### MacOS

**Prerequisites**

- Apple XCode

- Python v2.7.3 is recommended. The versions of MacOS after El Capitan come with Python 2.7 out of the box. It was also tested on Python 2.6. It does not work on Python 3.

	brew install python

- Node.js version v9.x, v8.x, v7.x, v6.x, v5.x or v4.x. 

**Build**

	git clone https://github.com/qminer/qminer.git && cd qminer
	npm install --build-from-source

**Run unit tests (from qminer root folder)**

	npm test

**XCode**

To generate XCode project files, say:

	npm install node-gyp -g
	node-gyp configure -- -f xcode

---
