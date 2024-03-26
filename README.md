# Baylon-node
Discord https://discord.gg/babylonglobal

Babylon - это новый проект Cosmos, целью которого является использование безопасности Bitcoin для повышения безопасности зон Cosmos и других цепочек PoS.

Ссылка на команду https://babylonchain.io/about

Сайт: https://babylonchain.io/foundation

Твиттер: https://www.twitter.com/babylon_chain

GitHub https://github.com/babylonchain/babylonchain.github.io

Zealy https://zealy.io/c/babylonchain/invite/1zn87lyrLTOaCWHZgpHR8

Требования к серверу

Recommended 4CPU 32RAM 1000GB

как вариант заказать здесь https://powervps.net/ru/?from=91820

Все действия делаем в приложени MobXterm ( туда ставиться сервер ( любой  гайд в ютуб по установке сервера на MobXterm, там буквально 2 действия после покупки сервера) и по очереди вводите туда команды и все должно получиться ) 

удалить старую версию
~~~
 $ sudo systemctl stop babylond && sudo systemctl disable babylond && sudo rm /etc/systemd/system/babylond.service && sudo systemctl daemon-reload && rm -rf $HOME/.babylond && rm -rf babylon && sudo rm -rf $(which babylond) 
~~~
Обновление пакетов сервера и подготовка к развертыванию ноды
~~~
sudo apt update && sudo apt upgrade -y
~~~
Настройка имени валидатора Сначала измените «Имя вашего валидатора» на выбранное вами имя валидатора и введите следующую команду:
~~~
MONIKER="Имя_вашего_валидатора"
~~~
Установите зависимости и установите инструменты сборки GO Install.
~~~
sudo apt -qy install curl git jq lz4 build-essential
~~~
Установить ГО
~~~
ver="1.22.0"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
~~~
Загрузите и создайте двоичные файлы
~~~
# Clone project repository
cd $HOME
rm -rf babylon
git clone https://github.com/babylonchain/babylon.git
cd babylon
git checkout v0.8.3
~~~
~~~
# Build binaries
make build
~~~
~~~
# Prepare binaries for Cosmovisor
mkdir -p ~/.babylond
mkdir -p ~/.babylond/cosmovisor
mkdir -p ~/.babylond/cosmovisor/genesis
mkdir -p ~/.babylond/cosmovisor/genesis/bin
mkdir -p ~/.babylond/cosmovisor/upgrades
~~~
~~~
mv build/babylond $HOME/.babylond/cosmovisor/genesis/bin/
rm -rf build
~~~
~~~
# Create application symlinks
sudo ln -s $HOME/.babylond/cosmovisor/genesis $HOME/.babylond/cosmovisor/current -f
sudo ln -s $HOME/.babylond/cosmovisor/current/bin/babylond /usr/local/bin/babylond -f
~~~
Настройте Cosmovisor и создайте соответствующий сервис
~~~
# Download and install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
~~~
~~~
# Create and start service
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=Babylon daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_NAME=babylond"
Environment="DAEMON_HOME=${HOME}/.babylond"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"

[Install]
WantedBy=multi-user.target
EOF
~~~
Меняем порты на нестандартные
~~~
sudo systemctl daemon-reload
sudo systemctl enable babylond.service
~~~
~~~
# Initialize the node
babylond init $MONIKER --chain-id bbn-test-3
~~~
~~~
# Add seeds
sed -i -e "s|^seeds *=.*|seeds = \"49b4685f16670e784a0fe78f37cd37d56c7aff0e@3.14.89.82:26656,9cb1974618ddd541c9a4f4562b842b96ffaf1446@3.16.63.237:26656\"|" $HOME/.babylond/config/config.toml

# Set minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.00001ubbn\"|" $HOME/.babylond/config/app.toml

# Switch to signet
sed -i -e "s|^network *=.*|network = \"signet\"|" $HOME/.babylond/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.babylond/config/app.toml
~~~
~~~
wget https://github.com/babylonchain/networks/raw/main/bbn-test-3/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
mv genesis.json ~/.babylond/config/genesis.json
~~~
~~~
sed -i.bak -e "s%:26658%:35658%; s%:26657%:35657%; s%:6060%:6960%; s%:26656%:35656%; s%:26660%:35660%" $HOME/.babylond/config/config.toml && sed -i.bak -e "s%:9090%:9990%; s%:9091%:9991%; s%:1317%:2217%; s%:8545%:9445%; s%:8546%:9446%; s%:6065%:6965%" $HOME/.babylond/config/app.toml && sed -i.bak -e "s%:26657%:35657%" $HOME/.babylond/config/client.toml
~~~
~~~
sudo systemctl restart babylond.service && sudo journalctl -u babylond.service -f --no-hostname -o cat
~~~
если не коннектится, можно взять пиры сдесь
~~~
https://www.polkachu.com/testnets/babylon/peers
~~~
~~~
https://nodestake.org/babylon
~~~
скачать снап
~~~
https://nodestake.org/babylon
~~~
Стать валидатором
~~~
# Create a New Key
babylond keys add wallet
~~~
или восстанавливаем прошлый
~~~
babylond keys add wallet --recover
~~~
вводим сид фразу от кошелька

Получите средства от крана тестовой сети Babylon

идем в дискорд и запрашиваем токены
!faucet адрес кошелька

Проверить баланс своего кошелька можно с помощью этой команды:

проверяем их на кошельке
~~~
babylond q bank balances $(babylond keys show wallet -a)
~~~
Убедитесь, что вы успешно получили  100 000 ubbn. Имейте в виду, что вы можете получать только 0/1 BBN из крана каждые 24 часа.

Создайте пару ключей BLS
~~~
babylond create-bls-key $(babylond keys show wallet -a)
~~~
~~~
sed -i -e "s|^key-name *=.*|key-name = \"wallet\"|" $HOME/.babylond/config/app.toml
~~~
~~~
sed -i -e "s|^timeout_commit *=.*|timeout_commit = \"10s\"|" $HOME/.babylond/config/config.toml
~~~
перезапускаем и проверяем логи
~~~
sudo systemctl daemon-reload
~~~
~~~
sudo systemctl start babylond.service && sudo journalctl -u babylond.service -f --no-hostname -o cat
~~~
Создайте валидатор
~~~
babylond status | jq .SyncInfo
~~~
После синхронизации создаем файл validator.json. ( Синхронизация это когда ваши цифры в бегущей строке догонят последний блок на сайте Baylon)
~~~
{
        "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"SBLBy6fLwg4TZFqTdT3muxffkTj9YlsIyPB3L8oYg="},
        "amount": "1000000ubbn",
        "chain-id": "bbn-test-3",
        "moniker": "Ваш_моникер",
        "website": "@WingsNodeTeam",
        "security": "telegramm @WingsNodeTeam",
        "details": "-_-",
        "commission-rate": "0.1",
        "commission-max-rate": "0.2",
        "commission-max-change-rate": "0.01",
        "min-self-delegation": "1"
}
~~~
где, pubkey можно узнать команду
~~~
babylond tendermint show-validator
~~~
сохраняем и записываем путь к этому файлу, пример
/root/.babylond/config/validator.json


создаем валидатора
~~~
babylond tx checkpointing create-validator /root/.babylond/config/validator.json --from=адрес_кошелька --chain-id bbn-test-3 --fees=12000ubbn -y
~~~
ждем 1 час и проверяем валидатора в браузере
~~~
https://testnet.babylon.explorers.guru/validators
~~~
делегировать себя
~~~
babylond tx делегата обновления адрес_валидатора 1000000ubbn --from=ваш_кошелек --chain-id bbn-test-3 --fees=12000ubbn -y
~~~


Делигировать себя можно получая 0.1 токен с одного дискора на 1  кошелек и далее отправляя на основоной кошелек по 0.1 токену после получения командой

~~~
babylond tx bank send bbn1qez5keyrf9g38dw4apknk9n2qqlhltpqq09ga5 bbn1sn3hv74nxda95ts9wnpl6qzmsevfq5q96qjgkx 80000ubbn --from wallet --chain-id bbn-test-3 --gas-adjustment 1.4 --gas auto --fees 98ubbn -y
~~~



Доп команды 
посмотреть кошельки  в ноде
~~~
babylond keys list
~~~



