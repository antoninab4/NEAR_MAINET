# NEAR_MAINET
# Установка ноды NEAR в Main сети!

## Гайд был написан с использованиям официальных источников блокчейна NEAR

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install python3 git curl
```
```
sudo apt install clang build-essential make
```
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
```
source $HOME/.cargo/env
```
```
git clone https://github.com/near/nearcore.git
```

## Установите тег последнего выпуска.
Последний тег выпуска, можно посмотреть здесь: https://github.com/near/nearcore/releases.
Примечание: RC тег только для тестовой сети, поэтому использовать нужно latest релиз, а не pre-release.
```
export NEAR_RELEASE_VERSION=1.30.0
```
```
cd nearcore
git checkout $NEAR_RELEASE_VERSION
make release
```

## Ставим Nodejs and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install build-essential nodejs
PATH="$PATH"
```

## Ставим Near CLI
```
sudo npm install -g near-cli
```

## Запустите эту команду, чтобы установить Near Mainnet Environment:
```
export NEAR_ENV=mainnet
echo 'export NEAR_ENV=mainnet' >> ~/.bashrc
```

## ЛОГИНЕМСЯ
```
near login
```
## Инициализация и запуск узла
```
./target/release/neard init --chain-id=mainnet \
 --account-id=<POOL_ID>.poolv1.near
```

Download the latest snapshot from the snapshot page. https://near-nodes.io/intro/node-data-snapshots
(загрузить снепшот сети командами на странице по ссылке), затем продолжать команды ниже.

Создаем сервисный файл 
```
sudo nano /etc/systemd/system/neard.service
```
Прописываем в нем свои данные и пути
и вставляем в файл, сохраняем.

```
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
Запускаемся

```
sudo systemctl enable neard
```
```
sudo systemctl start neard
```
```
sudo apt install ccze
```
```
journalctl -n 100 -f -u neard | ccze -A
```
```
near call <pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```

## УСТАНОВКА АВТОПИНГА
```
nano nearcore/scripts/ping.sh
```
```
#!/bin/sh
# Ping call to renew Proposal added to crontab

export NEAR_ENV=mainnet
export LOGS=logs
export POOLID=<pool name>

echo "---" >> $LOGS/all.log
date >> $LOGS/all.log
near call <pool name>.poolv1.near ping '{}' --accountId <your NEAR account> --gas=300000000000000 >> $LOGS/all.log
near proposals | grep $POOLID >> $LOGS/all.log
near validators current | grep $POOLID >> $LOGS/all.log
near validators next | grep $POOLID >> $LOGS/all.log
```
## сохраняем

## Задаем интервал в кроне на запуск пинга
```
mkdir $HOME/logs
```
```
chmod +x nearcore/scripts/ping.sh
```
```
crontab -e
```
```
0 */12 * * * sh nearcore/scripts/ping.sh
```
## ДОП КОМАНДЫ:

Для создания своего пула (в данном примере комиссия у пула будет 10%):
```
near call poolv1.near create_staking_pool '{"staking_pool_id": "<pool name>", "owner_id": "<your near account>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 10, "denominator": 100}}' --accountId="<your near account>" --amount=30 --gas=300000000000000
```
## Все готово. (обращаю внимание что скобки при указании своих данных убираются <> ).

