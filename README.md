# Learnings


Parcel allows us to get a no-frills, React-Typescript App deployed with minimal effort (if you know what you're doing).
This repository contains an implementation taken from [here](https://gist.github.com/croaky/e3394e78d419475efc79c1e418c243ed)
but without any of the directory hierarchies.

More importantly, the app is Web3-React enabled.

## 0. Biggest Stumbling Blocks

A. Failing to realize the subtle difference between **.ts** and **.tsx** files.
B. Consequently, not realizing that entrypoint.tsx is essentially index.ts helped me to lose time.
C. Parcel supports folks that just need TypeScript (alone) with the **.ts** extension.
D. Parcel supports folks needing TypeScript + React + JSX with the **.tsx** extension.

So take heed!

```javascript
import * as React from 'react'
import { render } from 'react-dom'
import { App } from './App'

render(
  <App />,
  document.getElementById('root')
)
```

**Note:** It should have been obvious to anyone that wasn't panicking and rushing, but you can read more [here](https://parceljs.org/typeScript.html).

## 1. Usage Guide

A. Fortunately, parcel.js performs no type checking by default. You have to use ```tsc --noEmit``` to have TypeScript check your files.

## 2. Installation Instructions

### Step 0: Prepare the EC2 Instance [Optional]

### Step 1: Install Parcel

Install parcel-bundler globally:

```bash
sudo npm install -g parcel-bundler
```

### Step 2: Create *package.json* File

Generate basic project info for your app:

```bash
npm init -y
```

### Step 3: Create *tsconfig.json* File

Let Parcel know to compile JSX for React:

```javascript
{
  "compilerOptions": {
    "jsx": "react"
  }
}
```

### Step 4: Create *index.html* File

This file should reference the entry point, which I'd guess is typically

A. **index.js** for plain HTML, CSS, and Javascript
B. **index.ts** for TypeScript
C. **index.tsx** for TypeScript + React + JSX

However, in this repository, the entry point was set to **entrypoint.tsx**.

```html
<html>
<body>
  <script src="./entrypoint.tsx"></script>
</body>
</html>
```

### Step 5: Create *entrypoint.tsx* File

To make sure you've done everything correctly up to this point, create an entrypoint.tsx with just the following:

```javascript
console.log('hello world')
```

Then run:

```bash
parcel index.html
```

Check the console output on a browser pointed at http://localhost:1234/ (make sure that you forwarded the port).

If it works, replace the contents with:

```javascript
import * as React from 'react'
import { render } from 'react-dom'
import { App } from './App'

render(
  <App />,
  document.getElementById('root')
)
```

You could just stop there and place all your TypeScript/Javascript/JSX stuff in this file. Unfortunately, folks like to have a separate App file. So...

### Step 6: Create *App.tsx* File

This file should hold your webapp. This webapp was stolen from [Lorenzo Sicilia](https://consensys.net/blog/developers/how-to-fetch-and-update-data-from-ethereum-with-react-and-swr/):

```javascript
import React from 'react'
import { Web3ReactProvider } from '@web3-react/core'
import { Web3Provider } from '@ethersproject/providers'
import { useWeb3React } from '@web3-react/core'
import { InjectedConnector } from '@web3-react/injected-connector'

export const injectedConnector = new InjectedConnector({
  supportedChainIds: [
    1, // Mainet
    3, // Ropsten
    4, // Rinkeby
    5, // Goerli
    42, // Kovan
  ],
})

function getLibrary(provider: any): Web3Provider {
  const library = new Web3Provider(provider)
  library.pollingInterval = 12000
  return library
}

export const Wallet = () => {
  const { chainId, account, activate, active } = useWeb3React<Web3Provider>()

  const onClick = () => {
    activate(injectedConnector)
  }

  return (
    <div>
      <div>ChainId: {chainId}</div>
      <div>Account: {account}</div>
      {active ? (
        <div>✅ </div>
      ) : (
        <button type="button" onClick={onClick}>
          Connect
        </button>
      )}
    </div>
  )
}

export const App = () => {
  return (
    <Web3ReactProvider getLibrary={getLibrary}>
      <Wallet />
    </Web3ReactProvider>
  )
}
```
