---
layout: page
title: Install QMiner
---

### Prerequisites

**node.js v8.x, v6.x, v4.x and npm 5.3 or higher**

To test that your node.js version is correct, run ```node --version``` and ```npm --version```. Not compatible with nodejs v0.10 or older.

**Windows**
- [Visual C++ Redistributable Packages for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145)   download **vcredist_x64.exe** for node.js x64 or **vcredist_x86.exe** for node.js x86.

### Install

	npm install qminer

**Test**

	node -e "require('qminer'); console.log('OK')"

---