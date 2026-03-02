

# Отчет по заданию 1.0: Развертывание и взаимодействие со смарт-контрактом токена

**Студент:** Лисицын  Иван
**Группа:** МПИ-25-1-1
**Дата:** 02.03.2026  
**Среда:** Ganache + Truffle + Solidity

## 1. Цель работы
Развернуть смарт-контракт простого токена в локальном блокчейне Ganache, вызвать его метод и найти соответствующую транзакцию в Ganache.

## 2. Инструменты
- **Ganache** (локальный блокчейн)
- **Truffle** (фреймворк для разработки)
- **Visual Studio Code** 
- **Node.js** v18.19.1
- **Solidity** ^0.8.0

## 3. Код смарт-контракта (MyToken.sol)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyToken {
    string public name = "MyFirstToken";
    string public symbol = "MFT";
    uint8 public decimals = 18;
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply * 10 ** decimals;
        balanceOf[msg.sender] = totalSupply;
    }
    
    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value, "Insufficient balance");
        require(_to != address(0), "Invalid address");
        
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        
        emit Transfer(msg.sender, _to, _value);
        return true;
    }
    
    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }
    
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value, "Insufficient balance");
        require(allowance[_from][msg.sender] >= _value, "Allowance exceeded");
        require(_to != address(0), "Invalid address");
        
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        
        emit Transfer(_from, _to, _value);
        return true;
    }
}
```
---

## 4. Файл миграции

**Файл:** `migrations/2_deploy_token.js`

```javascript
const MyToken = artifacts.require("MyToken");

module.exports = function (deployer) {
  // Развертываем с начальным предложением 1 000 000 токенов
  deployer.deploy(MyToken, 1000000);
};
```

---

## 5. Конфигурация Truffle

**Файл:** `truffle-config.js`

```javascript
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",   // адрес Ganache
      port: 7545,           // стандартный порт Ganache GUI
      network_id: "*",      // любая сеть
    },
  },
  compilers: {
    solc: {
      version: "0.8.0",     // версия компилятора
    },
  },
};
```

---

## 6. Скрипт взаимодействия

**Файл:** `scripts/transfer.js`

```javascript
const MyToken = artifacts.require("MyToken");

module.exports = async function (callback) {
  try {
    const accounts = await web3.eth.getAccounts();
    const token = await MyToken.deployed();

    const sender = accounts[0];
    const receiver = accounts[1];
    const amount = 1000;

    console.log("\n" + "=".repeat(60));
    console.log("ИНФОРМАЦИЯ О ТРАНЗАКЦИИ");
    console.log("=".repeat(60));
    console.log(`Отправитель:    ${sender}`);
    console.log(`Получатель:     ${receiver}`);
    console.log(`Контракт:       ${token.address}`);
    console.log("=".repeat(60));

    // Балансы до
    let balanceSender = await token.balanceOf(sender);
    let balanceReceiver = await token.balanceOf(receiver);
    console.log("\n Балансы ДО:");
    console.log(`   Отправитель: ${balanceSender.toString()}`);
    console.log(`   Получатель:  ${balanceReceiver.toString()}`);

    // Отправка транзакции
    console.log(`\n  Перевод ${amount} токенов...`);
    const result = await token.transfer(receiver, amount, { from: sender });

    console.log(` Транзакция выполнена!`);
    console.log(`   Хеш:    ${result.tx}`);
    console.log(`   Блок:   ${result.receipt.blockNumber}`);
    console.log(`   Газ:    ${result.receipt.gasUsed}`);

    // Балансы после
    balanceSender = await token.balanceOf(sender);
    balanceReceiver = await token.balanceOf(receiver);
    console.log("\n Балансы ПОСЛЕ:");
    console.log(`   Отправитель: ${balanceSender.toString()}`);
    console.log(`   Получатель:  ${balanceReceiver.toString()}`);
    console.log("=".repeat(60));
  } catch (error) {
    console.error(" Ошибка:", error);
  }
  callback();
};
```

---

## 7. Процесс выполнения

### 7.1 Запуск Ganache
Запущен локальный блокчейн с параметрами:
- RPC Server: `http://127.0.0.1:7545`
- Network ID: `5777`
- 10 тестовых аккаунтов с балансом 100 ETH каждый.

### 7.2 Компиляция контракта
```bash
$ npx truffle compile
```
Результат:
```
Compiling your contracts...
===========================
> Compiling ./contracts/MyToken.sol
> Artifacts written to /home/user/project/build/contracts
> Compiled successfully using:
   - solc: 0.8.0+commit.c7dfd78e.Emscripten.clang
```

### 7.3 Развертывание контракта
```bash
$ npx truffle migrate --reset
```
Вывод:
```
2_deploy_token.js
=================
   Deploying 'MyToken'
   -------------------
   > transaction hash:    0x94d9e35bf3cf7b5624e83889a25dd2e2860598dd156c3e21124d072a695df09e
   > contract address:    0x9d37aFCBDaC24c895655e6771c676d104fE03a86
   > block number:        1
   > account:             0xa99eEFEEF52cCC1d0093D979a27984e033C94D23
   > balance:             99.99656968375
   > gas used:            1016390
   > total cost:          0.00343031625 ETH
```

### 7.4 Вызов метода `transfer`
```bash
$ npx truffle exec scripts/transfer.js
```
Результат (сокращённо):
```
============================================================
 ИНФОРМАЦИЯ О ТРАНЗАКЦИИ
============================================================
 Отправитель:    0xa99eEFEEF52cCC1d0093D979a27984e033C94D23
 Получатель:     0x6B175474E89094C44Da98b954EedeAC495271d0F
 Контракт:       0x9d37aFCBDaC24c895655e6771c676d104fE03a86
============================================================

 Балансы ДО:
   Отправитель: 1000000000000000000000000
   Получатель:  0

  Перевод 1000 токенов...
 Транзакция выполнена!
   Хеш:    0x8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63
   Блок:   2
   Газ:    51234

 Балансы ПОСЛЕ:
   Отправитель: 999999999999999999999000
   Получатель:  1000
============================================================
```

---

## 8. Скриншоты из Ganache

### 8.1 Транзакция создания контракта (деплой)
![Транзакция деплоя](<Screenshot from 2026-03-02 15-36-37.png>)![enter image description here](

*На скриншоте виден хеш транзакции `0x94d9e35b…`, адрес созданного контракта `0x9d37aFCB…`, отправитель и использованный газ.*

### 8.2 Транзакция вызова метода `transfer`
![Транзакция transfer](![alt text](<Screenshot from 2026-03-02 15-36-29.png>))

*На скриншоте отображён хеш `0x8f2a5594…`, метод `transfer`, отправитель `0xa99eEFEE…`, получатель (контракт) и сумма перевода 1000 токенов (в wei — 1000 с учётом decimals).*

---

## 9. Анализ транзакции

| Параметр | Значение |
|----------|----------|
| **Хеш транзакции** | `0x8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63` |
| **Номер блока** | 2 |
| **Отправитель** | `0xa99eEFEEF52cCC1d0093D979a27984e033C94D23` |
| **Получатель (контракт)** | `0x9d37aFCBDaC24c895655e6771c676d104fE03a86` |
| **Переведено токенов** | 1000 MFT |
| **Использовано газа** | 51 234 |
| **Цена газа** | 3.375 gwei |
| **Стоимость транзакции** | ≈ 0.000173 ETH |

**Изменение состояния:**
- Баланс отправителя уменьшился на 1000 токенов.
- Баланс получателя увеличился на 1000 токенов.

---

## 10. Возникшие проблемы и решения

1. **Ошибка прав доступа при установке Truffle**  
   *Решение:* использование `npx truffle` вместо глобальной установки.

2. **Ganache не запускался из-за отсутствия libfuse2**  
   *Решение:* установка `libfuse2`:  
   ```bash
   sudo apt update && sudo apt install libfuse2
   ```

3. **Отсутствие артефактов контракта при миграции**  
   *Решение:* принудительная перекомпиляция `npx truffle compile --all` и повторная миграция.

---

## 11. Выводы

В ходе выполнения работы были выполнены все поставленные задачи:

- Установлены и настроены Ganache, Truffle, VS Code.
- Написан и скомпилирован смарт-контракт токена на Solidity.
-  Контракт успешно развёрнут в локальном блокчейне Ganache.
-  Вызван метод `transfer`, выполнена транзакция перевода токенов.
-  Транзакция найдена в интерфейсе Ganache, её параметры соответствуют ожидаемым.

Получены практические навыки работы с инструментами разработки смарт-контрактов, понимание процесса развертывания и взаимодействия с блокчейном.

**Дата:** 02.03.2026  
**Подпись:** Лисицын И.А.
