![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/31237af3-0dd0-4302-b5e9-8ceb7fc7af22)

## Обзор и настройка инструмента для мониторинга работоспособности Cosmos нод(0G Labs, Side Protocol, PRYZM, Artela...)

**Tenderduty** - это комплексный инструмент мониторинга для сетей Tendermint. Его основная функция - оповещать пользователя о наличии проблем с нодами, например , если в них отсутствуют блоки.

Настроив данный инструмент я смог мониторить свою ноду Side Protocol через браузер, ниже я представил подробный скрин того, как это выглядит:

![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/26339d1c-2ba3-408f-88d6-317fc0352301)

## Настройка Tenderduty

**1. После того, как Вы успешно установили свою Cosmos ноду на сервер, Вам необходимо сделать еще один апдейт пакетов на сервере, а также установить необходимое ПО для должной работы Tenderduty:**

```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl build-essential git wget jq make gcc tmux pkg-config libssl-dev libleveldb-dev tar -y
```

**2. Большинство Cosmos нод работают на ЯП Go, но если Ваша нода исключение, то Вам необходимо дополнительно установить Go:**

```
ver="1.18.2"
```

```
cd $HOME
```

```
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
```

```
sudo rm -rf /usr/local/go
```

```
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
```

```
rm "go$ver.linux-amd64.tar.gz"
```

```
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
```

```
source ~/.bash_profile
```

```
go version
```

**3. Переходим установке самого инструмента Tenderduty:**

```
cd $HOME
```

```
rm -rf tenderduty
```

```
git clone https://github.com/blockpane/tenderduty
```

```
cd tenderduty
```

```
go install
```

```
cp example-config.yml config.yml
```

**4. Открываем конфиг Tenderduty и настраиваем его:**

```
sudo nano $HOME/tenderduty/config.yml
```

**5. Для мониторинга нам достаточно настроить следующие параметры:**

*- Osmosys to <Имя_Вашей_Ноды>;*

*- chain_id: osmosis-1 to chain_id: <ID_блокчейна;*

*- valoper_address: osmovaloper1xxxxxxx... to valoper_address: <Адрес_кошелька_Вашего_валидатора>;*

*- url: tcp://localhost:26657 TO url: tcp://localhost:<Порт_на_котором_работает_нода>*




Для того, чтобы узнать какой порт задействован под Вашу ноду, Вы можете выполнить эти команды на сервере:

```
sudo netstat -tuln | grep <номер_PID_процесса>
```

```
sudo ss -tuln | grep <номер_PID_процесса>
```

PID процесса можно узнать, если запросить статус работы сервиса ноды. Обычно это команда:

```
sudo systemctl status <Имя_сервиса>

Пример с нодой Side Protocol: sudo systemctl status sided
```

![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/b1fded8d-54f3-41dc-8882-c7441e29b679)

![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/b7ebd8b6-07f6-493b-82a9-69a7d942c173)

**Дополнительно:**

Если у Вас на сервере находится несколько нод Cosmos, Вы можете продублировать код ниже в файл конфига и адаптировать его под остальные ноды на Вашем сервере:

```
chains:

  # The user-friendly name that will be used for labels. Highly suggest wrapping in quotes.
  "Osmosis":
    # chain_id is validated for a match when connecting to an RPC endpoint, also used as a label in several places.
    chain_id: osmosis-1
    # Hooray, in v2 we derive the valcons from abci queries so you don't have to jump through hoops to figure out how
    # to convert ed25519 keys to the appropriate bech32 address
    valoper_address: osmovaloper1xxxxxxx...
    # Should the monitor revert to using public API endpoints if all supplied RCP nodes fail?
    # This isn't always reliable, not all public nodes have websocket proxying setup correctly.
    public_fallback: no

    # Controls various alert settings for each chain.
    alerts:
      # If the chain stops seeing new blocks, should an alert be sent?
      stalled_enabled: yes
      # How long a halted chain takes in minutes to generate an alarm
      stalled_minutes: 10

      # Most basic alarm, you just missed x blocks ... would you like to know?
      consecutive_enabled: yes
      # How many missed blocks should trigger a notification?
      consecutive_missed: 5
      # NOT USED: future hint for pagerduty's routing
      consecutive_priority: critical

      # For each chain there is a specific window of blocks and a percentage of missed blocks that will result in
      # a downtime jail infraction. Should an alert be sent if a certain percentage of this window is exceeded?
      percentage_enabled: no
      # What percentage should trigger the alert
      percentage_missed: 10
      # Not used yet, pagerduty routing hint
      percentage_priority: warning

      # Should an alert be sent if the validator is not in the active set ie, jailed,
      # tombstoned, unbonding?
      alert_if_inactive: yes
      # Should an alert be sent if no RPC servers are responding? (Note this alarm is instantaneous with no delay)
      alert_if_no_servers: yes

      # for this *specific* chain it's possible to override alert settings. If the api_key or webhook addresses are empty,
      # the global settings will be used. Note, enabled must be set both globally and for each chain.

      # Chain specific setting for pagerduty
      pagerduty:
        enabled: yes
        api_key: "" # uses default if blank

      # Discord settings
      discord:
        enabled: yes
        webhook: "" # uses default if blank

      # Telegram settings
      telegram:
        enabled: yes
        api_key: "" # uses default if blank
        channel: "" # uses default if blank

    # This section covers our RPC providers. No LCD (aka REST) endpoints are used, only TM's RPC endpoints
    # Multiple hosts are encouraged, and will be tried sequentially until a working endpoint is discovered.
    nodes:
      # URL for the endpoint. Must include protocol://hostname:port
      - url: tcp://localhost:26657
        # Should we send an alert if this host isn't responding?
        alert_if_down: yes
      # repeat hosts for monitoring redundancy
      - url: https://some-other-node:443
        alert_if_down: no
```

**6. Создаем сервис мониторинга(вводите одной командой):**

```
sudo tee /etc/systemd/system/tenderdutyd.service << EOF
[Unit]
Description=Tenderduty
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=180

User=$USER
WorkingDirectory=$HOME/tenderduty
ExecStart=$(which tenderduty)

# there may be a large number of network connections if a lot of chains
LimitNOFILE=infinity

# extra process isolation
NoNewPrivileges=true
ProtectSystem=strict
RestrictSUIDSGID=true
LockPersonality=true
PrivateUsers=true
PrivateDevices=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

**Далее выполняем команды:**

```
sudo systemctl daemon-reload
```

```
sudo systemctl enable tenderdutyd
```

```
sudo systemctl start tenderdutyd
```

**Можем увидеть логи сервиса(см. скрин ниже)**

![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/c9d2f70e-b57a-431e-91ce-b42a93638a2a)

**На данном этапе все, переходим в браузер, вставляем адрес своего сервера в формате: http://<IP_Вашего_сервера>:8888**

![image](https://github.com/Mozgiii9/CosmosNodesMonitoringSystem/assets/74683169/24dd39cd-9eef-41e2-b0d0-b2b81f14b350)

### Обязательно проведите собственный ресерч проектов перед тем как ставить ноду. Сообщество NodeRunner не несет ответственность за Ваши действия и средства. Помните, проводя свой ресёрч, Вы учитесь и развиваетесь.

### Связь со мной: [Telegram(@M0zgiii)](https://t.me/m0zgiii)

### Мои соц. сети: [Twitter](https://twitter.com/m0zgiii) 




