# zardkat 🐱

A [hardhat-circom](https://github.com/projectsophon/hardhat-circom) template to generate zero-knowledge circuits, proofs, and solidity verifiers

## Quick Start

Compile the zkcircuit() circuit and validate it using a smart contract verifier.
```
pragma circom 2.0.0;

include "../../node_modules/circomlib/circuits/gates.circom";

/* Circuit that proves the inputs A (0) & B (1) yields a 0 output.*/

template zkcircuit () {
    // Inputs
    signal input a;
    signal input b;
    
    // Internal signals
    signal x;
    signal y;

    // Output
    signal output q;

    // Components
    component andGate = AND();
    component notGate = NOT();
    component orGate = OR();

    // Wiring the components
    andGate.a <== a;
    andGate.b <== b;
    x <== andGate.out;

    notGate.in <== b;
    y <== notGate.out;

    orGate.a <== x;
    orGate.b <== y;
    q <== orGate.out;

    // Logging the output
    log("Output q", q);
}

// Instantiating the circuit
component main = zkcircuit();
```
### Install
`npm i`

### Compile
`npx hardhat circom` 

This process will produce the **out** file containing circuit intermediaries and generate the **ZkcircuitVerifier.sol** contract.

```
explaination: 
a = 0
b = 1

x = 0 AND 1 => 0

y = !1 => 0

q = 0 OR 0 => 0 
```

### Prove and Deploy
`npx hardhat run scripts/deploy.ts`
The script performs four main tasks:

1. Deploys the MultiplierVerifier.sol contract.
2. Creates a proof using circuit intermediaries with the `generateProof()` function.
3. Generates calldata using the `generateCallData()` function.
4. Invokes `verifyProof()` on the verifier contract with the generated calldata.

## Configuration
### Directory Structure
**circuits**
```
├── zkcircuit
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── zkcircuit.r1cs
│       ├── zkcircuit.vkey
│       └── zkcircuit.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit resides in its own directory. At the top level of each circuit directory, you'll find the circom circuit and its input.
The **out** directory is automatically generated to store compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, eliminating the need for a new ceremony.

**contracts**
```
contracts
└── ZkcircuitVerifier.sol
```
Verifier contracts are automatically generated and prefixed with the circuit name, as demonstrated in this example with Zkcircuit.

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```

**adding circuits**   
To add a new circuit, you can execute the `newcircuit` hardhat task to automatically generate configuration and directories i.e.,
```
npx hardhat newcircuit --name newcircuit
```
