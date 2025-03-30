
### Lab 4 : Interaction avec Smart Contracts via Web3.js & Ethers.js (Comparaison)

#### Description du Lab

Dans ce lab, vous apprendrez à interagir avec un smart contract via Web3.js et Ethers.js, deux des bibliothèques les plus populaires pour travailler avec la blockchain Ethereum. Vous comprendrez comment ces bibliothèques diffèrent dans leur approche de la connexion, de l'interaction et de l'envoi de transactions vers un contrat, tout en accomplissant les mêmes tâches avec chacune d'elles.

----------

### Prérequis

Avant de commencer ce lab, assurez-vous d'avoir configuré votre environnement comme suit :

-   Node.js >= 16.x  
      
    
-   Metamask ou un autre wallet Ethereum installé dans votre navigateur  
      
    
-   Compte Ethereum sur Sepolia Testnet  
      
    
-   GitHub Codespaces ou un IDE local avec Web3.js, Ethers.js et Hardhat installés  
      
    
-   Infura/Alchemy pour interagir avec Sepolia Testnet  
      
    

----------

### Objectifs du Lab

1.  Créer un smart contract de paiement (similaire à celui utilisé dans les labs précédents).  
      
    
2.  Interagir avec ce contrat via Ethers.js.  
      
    
3.  Interagir avec le même contrat via Web3.js.  
      
    
4.  Comparer les deux bibliothèques en termes de syntaxe et de facilité d'utilisation.  
      
    
5.  Déployer et interagir avec le contrat sur le testnet Sepolia.  
      
    

----------

### Étapes du Lab

#### 1. Préparer l'environnement

  

Installer les dépendances en utilisant npm :  
  
```bash  
cd lab-web3-ethers-comparison
npm install
```

1.  Vérifier la configuration de Hardhat :  
      
    

-   Assurez-vous que vous avez le fichier hardhat.config.js configuré pour Sepolia comme suit :  
      
    

```javascript  
require('@nomiclabs/hardhat-ethers');  

module.exports = {

solidity: "0.8.19", // Dernière version de solidity

networks: {

sepolia: {

url: `https://sepolia.infura.io/v3/YOUR_INFURA_KEY`,

accounts: [`0x${YOUR_PRIVATE_KEY}`],

},

},

};
```

2.  Vérification de la configuration Metamask :     
    
-   Assurez-vous que Metamask est connecté au testnet Sepolia avec un solde suffisant pour déployer un contrat.       
    

----------

#### 2. Créer un contrat solidity - InstantPaymentHub

1.  Créez un fichier contracts/InstantPaymentHub.sol dans le répertoire contracts/.  
      
    
2.  Implémentez un contrat de paiement instantané :  
      
    

```solidity

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

  

contract InstantPaymentHub {

address public owner;

mapping(address => uint256) public balances;

  

event PaymentMade(address indexed sender, address indexed receiver, uint256 amount);

  

modifier onlyOwner() {

require(msg.sender == owner, "Only the owner can execute this");

_;

}

  

constructor() {

owner = msg.sender;

}

  

// Déposer de l'ether dans le contrat

function deposit() public payable {

balances[msg.sender] += msg.value;

}

  

// Effectuer un paiement instantané entre utilisateurs

function instantPayment(address recipient, uint256 amount) public {

require(balances[msg.sender] >= amount, "Insufficient balance");

balances[msg.sender] -= amount;

balances[recipient] += amount;

emit PaymentMade(msg.sender, recipient, amount);

}

  

// Retirer des fonds du contrat

function withdraw(uint256 amount) public {

require(balances[msg.sender] >= amount, "Insufficient balance");

payable(msg.sender).transfer(amount);

balances[msg.sender] -= amount;

}

}
```

----------

#### 3. Déployer le contrat sur Sepolia

1.  Créer un script de déploiement dans scripts/deploy.js :  
      
    

```javascript
async function main() {

const [deployer] = await ethers.getSigners();

console.log("Déployé par : ", deployer.address);

  

const Contract = await ethers.getContractFactory("InstantPaymentHub");

const contract = await Contract.deploy();

console.log("Contrat déployé à l'adresse : ", contract.address);

}

  

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});
```

  

Exécuter le script de déploiement :  
  
```bash  
npx hardhat run scripts/deploy.js --network sepolia
```

2.  Cela déploiera le contrat sur le testnet Sepolia et vous donnera l'adresse du contrat déployé.  
      
    

----------

#### 4. Interagir avec le contrat via Ethers.js

1.  Créez un fichier scripts/interact-ethers.js pour interagir avec le contrat déployé via Ethers.js :  
      
    

```javascript
async function main() {

const [deployer] = await ethers.getSigners();

const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT"; // Remplacez par l'adresse du contrat déployé

const contract = await ethers.getContractAt("InstantPaymentHub", contractAddress);

  

// Déposer de l'ether dans le contrat

const depositTx = await contract.deposit({ value: ethers.utils.parseEther("1.0") }); // Déposer 1 Ether

await depositTx.wait();

console.log("1 Ether déposé dans le contrat");

  

// Effectuer un paiement instantané

const recipient = "ADRESSE_D_UN_UTILISATEUR"; // Remplacez par l'adresse du destinataire

const paymentTx = await contract.instantPayment(recipient, ethers.utils.parseEther("0.5"));

await paymentTx.wait();

console.log("0.5 Ether payé à", recipient);

  

// Retirer de l'ether du contrat

const withdrawTx = await contract.withdraw(ethers.utils.parseEther("0.2"));

await withdrawTx.wait();

console.log("0.2 Ether retiré du contrat");

}

  

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});
```

Exécuter le script Ethers.js :  
  
```bash  
npx hardhat run scripts/interact-ethers.js --network sepolia
```

----------

#### 5. Interagir avec le contrat via Web3.js

1.  Créez un fichier scripts/interact-web3.js pour interagir avec le même contrat via Web3.js :  
      
    

```javascript

const Web3 = require('web3');

const { abi } = require('../artifacts/contracts/InstantPaymentHub.sol/InstantPaymentHub.json'); // L'ABI du contrat

  
async function main() {

const web3 = new Web3('https://sepolia.infura.io/v3/YOUR_INFURA_KEY'); // Connectez-vous à Sepolia via Infura

const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT"; // Remplacez par l'adresse du contrat déployé

  

const accounts = await web3.eth.getAccounts();

const contract = new web3.eth.Contract(abi, contractAddress);

  

// Déposer de l'ether dans le contrat

await contract.methods.deposit().send({ from: accounts[0], value: web3.utils.toWei('1', 'ether') });

console.log("1 Ether déposé dans le contrat");

  

// Effectuer un paiement instantané

const recipient = "ADRESSE_D_UN_UTILISATEUR"; // Remplacez par l'adresse du destinataire

await contract.methods.instantPayment(recipient, web3.utils.toWei('0.5', 'ether')).send({ from: accounts[0] });

console.log("0.5 Ether payé à", recipient);

  

// Retirer de l'ether du contrat

await contract.methods.withdraw(web3.utils.toWei('0.2', 'ether')).send({ from: accounts[0] });

console.log("0.2 Ether retiré du contrat");

}
  

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});

```

  

Exécuter le script Web3.js :  
  
```bash  
node scripts/interact-web3.js
```

----------

#### 6. Comparaison entre Ethers.js et Web3.js

-   Ethers.js est une bibliothèque plus moderne, plus légère et plus orientée sécurité, idéale pour la gestion des transactions et des smart contracts avec une syntaxe propre et simple.  
      
    
-   Web3.js est plus ancien, mais reste largement utilisé dans l'écosystème Ethereum. Il est plus lourd mais offre un large éventail de fonctionnalités.  
      
    

| **<br>                                Fonctionnalité<br>                            **           | **<br>                                Ethers.js<br>                            **                      | **<br>                                Web3.js<br>                            **                        |
|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| **<br>                                Installation<br>                            **             | <br>                                npm install ethers<br>                                             | <br>                                npm install web3<br>                                               |
| **<br>                                Interaction avec contrat<br>                            ** | <br>                                contract.methods.function().send()<br>                             | <br>                                contract.methods.function().send()<br>                             |
| **<br>                                Gestion des comptes<br>                            **      | <br>                                getSigners()<br>                                                   | <br>                                web3.eth.getAccounts()<br>                                         |
| **<br>                                Performance<br>                            **              | <br>                                Plus légère, plus rapide<br>                                       | <br>                                Plus lourde, mais fonctionnelle<br>                                |


----------

  

### Fichiers du Projet

Voici un aperçu des fichiers et dossiers dans le repo pour ce lab :

```lua  

lab-web3-ethers-comparison/

│

├── contracts/

│ └── InstantPaymentHub.sol

│

├── scripts/

│ ├── deploy.js

│ ├── interact-ethers.js

│ └── interact-web3.js

│

├── test/

│ └── instant-payment-hub.test.js (optionnel pour les tests unitaires)

│

├── hardhat.config.js

├── package.json

├── README.md

```

----------

### Objectifs de l'étudiant après ce lab

-   Comprendre la différence entre Ethers.js et Web3.js.
    
-   Savoir déployer et interagir avec un contrat via Ethers.js et Web3.js.  

-   Être capable de comparer et choisir la bibliothèque la mieux adaptée à un projet.
