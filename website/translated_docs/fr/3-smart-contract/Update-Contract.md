---
id: Update-Contract
title: Mise à Jour du Contrat
sidebar_label: Mise à Jour du Contrat
---

## Features

Une fois le contrat déployé sur la blockchain, le développeur peut souhaiter lui apporter des modifications, telles que des bugfix, des mises à jours, etc.

Nous proposons un mécanisme de mise à jour complet qui permet de facilement mettre à jour un smart contract via l'envoi d'une transaction.
Plus important, nous proposons un contrôle de permission flexible permettant de se conformer à n'importe quel besoin.

Afin de pouvoir mettre à jour le smart contract, il est nécessaire d'implémenter la fonction suivante :
```js
can_update(data) {
}
```

Lors de la réception d'une requête de mise à jour de contrat, le système va d'abord appeler la fonction can_update(data). data est un paramètre d'entrée facultatif de type string. Si la fonction renvoie true, la mise à jour est effectuée. Si non, une erreur `Update Refused` est renvoyée.

Avec l'écriture correcte de cette fonction il est possible d'implémenter une gestion des permissions telle que : seulement mettre à jour lorsque deux personnes l'autorisent en même temps, ou via un vote, etc.

Si la fonction n'est pas implémentée dans le contrat, celui-ci n'est pas autorisée à se mettre à jour par défaut.

## Hello BlockChain

Ci-dessous nous prenons un smart contract simple comme exemple pour illustrer ce processus.

### Création du Contrat

Tout d'abord créer un compte de mise à jour à l'aide de la commande `iwallet`, enregistrer le ID renvoyé sur l'écran
```console
./iwallet account -n update
return:
the iost account ID is:
IOSTURXazDVc1hJ9R9HdFxt2PivzKxUdUaN1A7rgRkoBDMJZ9qj2h
```

Créer un nouveau fichier contrat helloContract.js et son fichier ABI helloContract.json correspondant avec le contenu suivant
```js
class helloContract
{
    constructor() {
    }
    init() {
    }
    hello() {
		const ret = storage.put("message", "hello block chain");
        if (ret !== 0) {
            throw new Error("storage put failed. ret = " + ret);
        }
    }
	can_update(data) {
		const adminID = "IOSTURXazDVc1hJ9R9HdFxt2PivzKxUdUaN1A7rgRkoBDMJZ9qj2h";
		const ret = BlockChain.requireAuth(adminID);
		return ret;
	}
};
module.exports = helloContract;
```
```json
{
    "lang": "javascript",
    "version": "1.0.0",
    "abi": [
        {
            "name": "hello",
            "args": []
        },
		{
			"name": "can_update",
			"args": ["string"]
		}
    ]
}
```
Remarquez l'Implémentation de la fonction can_update() dans le fichier contrat, qui permet la mise à jour uniquement lorsque l'autorisation adminID est utilisée.

### Déployer le contrat

Se référer à [Deployment-and-invocation](../3-smart-contract/Deployment-and-invocation)

Remember to record contractID like ContractHDnNufJLz8YTfY3rQYUFDDxo6AN9F5bRKa2p2LUdqWVW

### Update Contract
First edit the contract file helloContract.js to generate a new contract code as follows:
```js
class helloContract
{
    constructor() {
    }
    init() {
    }
    hello() {
		const ret = storage.put("message", "update block chain");
        if (ret !== 0) {
            throw new Error("storage put failed. ret = " + ret);
        }
    }
	can_update(data) {
		const adminID = "IOSTURXazDVc1hJ9R9HdFxt2PivzKxUdUaN1A7rgRkoBDMJZ9qj2h";
		const ret = BlockChain.requireAuth(adminID);
		return ret;
	}
};
module.exports = helloContract;
```
We modified the implementation of the hello() function to change the contents of the "message" written to the database to "update block chain".

Use the following command to upgrade your smart contract:

```console
./iwallet compile -u -e 3600 -l 100000 -p 1 ./helloContract.js ./helloContract.json <合约ID> <can_update 参数> -k ~/.iwallet/update_ed25519
```
-u indicates to update contract, -k indicates the private key used for signing and publishing, here the account `update` is used to authorize the transaction

After the transaction is confirmed, you can call the hello() function via iwallet and check the contents of the database "message" to see the content changes.
