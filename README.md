cryptonote-nodejs-pool
======================

High performance Node.js (with native C addons) mining pool for CryptoNote based coins. Comes with lightweight example front-end script which uses the pool's AJAX API. Support for Cryptonight (Original, RandomX sispop) Cryptonight Fast (Electronero/Crystaleum), and Cryptonight Heavy (Sumokoin) algorithms.


#### Table of Contents
* [Features](#features)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Starting the Pool](#3-start-the-pool)
  * [Host the front-end](#4-host-the-front-end)
  * [Customizing your website](#5-customize-your-website)
  * [SSL](#ssl)
  * [Upgrading](#upgrading)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Referral Links](#referral-links)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)
* [Pay for build](#Pay for build)


Features
===

#### Optimized pool server
* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* Support for Cryptonight (Original, Monero v7, Stellite v7), Cryptonight Light (Original, Aeon v7, IPBC) and Cryptonight Heavy (Sumokoin) algorithms.
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Ability to configure multiple ports - each with their own difficulty
* Miner login (wallet address) validation
* Workers identification (specify worker name as the password)
* Variable difficulty / share limiter
* Set fixed difficulty on miner client by passing "address" param with "+[difficulty]" postfix
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* SSL support for both pool and API servers
* RBPPS (PROP) payment system

#### Live statistics API
* Currency network/block difficulty
* Current block height
* Network hashrate
* Pool hashrate
* Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, payout estimate, etc)
* Blocks found (pending, confirmed, and orphaned)
* Historic charts of pool's hashrate, miners count and coin difficulty
* Historic charts of users's hashrate and payments

#### Mined blocks explorer
* Mined blocks table with block status (pending, confirmed, and orphaned)
* Blocks luck (shares/difficulty) statistics
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)

#### Smart payment processing
* Splintered transactions to deal with max transaction size
* Minimum payment threshold before balance will be paid out
* Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option
* Option to enable dynamic transfer fee based on number of payees per transaction and option to have miner pay transfer fee instead of pool owner (applied to dynamic fee only)
* Control transactions priority with config.payments.priority (default: 0).
* Set payment ID on miner client when using "[address].[paymentID]" login
* Integrated payment ID addresses support for Exchanges

#### Admin panel
* Aggregated pool statistics
* Coin daemon & wallet RPC services stability monitoring
* Log files data access
* Users list with detailed statistics

#### Pool stability monitoring
* Detailed logging in process console & log files
* Coin daemon & wallet RPC services stability monitoring
* See logs data from admin panel

#### Extra features
* An easily extendable, responsive, light-weight front-end using API to display data
* 2realminers [keepalive function](https://github.com/perl5577/cpuminer-multi/commit/0c8aedb)
* Support for merged mining
* Support for slush mining system (disabled by default)
* E-Mail Notifications on worker connected, disconnected (timeout) or banned (support MailGun, SMTP and Sendmail)
* Telegram channel notifications when a block is unlocked
* Top 10 miners report
* Multilingual user interface

Usage
===

#### Requirements
* Coin daemon(s) (find the coin's repo and build latest version from source)
* [Node.js](http://nodejs.org/) v11.0+
  * For Ubuntu: 18.04 - 20.*
 ```
  curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash
  sudo apt-get install -y nodejs
 ```
  * Or use NVM(https://github.com/creationix/nvm) for debian/ubuntu.


* [Redis](http://redis.io/) key-value store v2.6+ 
  * For Ubuntu: 
```
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
 ```
 Dont forget to tune redis-server:
 ```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1024 > /proc/sys/net/core/somaxconn
 ```
 Add this lines to your /etc/rc.local and make it executable
 ```
 chmod +x /etc/rc.local
 ```
 
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`

* Boost is required for the cryptoforknote-util module
  * For Ubuntu: `sudo apt-get install libboost-all-dev`
  
* libsodium  
  * For Ubuntu: `sudo apt-get install libsodium-dev`


##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.

[**Redis warning**](http://redis.io/topics/security): It'sa good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).

**Do not run the pool as root** : create a new user without ssh access to avoid security issues :
```bash
```
sudo adduser your-user
```
sudo usermod -aG sudo your-user
```
To login with this user : 
```
sudo su - your-user
```

#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/2realminers/sispop-pool.git pool
cd pool

npm update
```

#### 2) Configuration

Copy  `this code and save in pool folder` file of your choice to `config.json` then overview each options and change any to match your preferred setup.

Explanation for each field:
```javascript
{
  "poolHost": "randomx.2realminers.com",

  "coin": "sispop",
  "symbol": "SISPOP",
  "coinUnits": 1000000000,
  "coinDecimalPlaces": 4,
  "coinDifficultyTarget": 120,
  "blockchainExplorer": "https://explorer.sispop.site/block/{id}",
  "transactionExplorer": "https://explorer.sispop.site/tx/{id}",
  "daemonType": "default",
  "cnAlgorithm": "randomx",
  "cnVariant": 0,
  "cnBlobType": 5,
  "includeHeight": true,
  "isRandomX": true,

  "logging": {
    "files": {
      "level": "info",
      "directory": "logs",
      "flushInterval": 5
    },
    "console": {
      "level": "info",
      "colors": true
    }
  },
  "childPools": null,
  "poolServer": {
    "enabled": true,
    "mergedMining": false,
    "clusterForks": "auto",
    "poolAddress": "your-wallet-address",
    "intAddressPrefix": 19,
    "blockRefreshInterval": 1000,
    "minerTimeout": 900,
    "sslCert": "./cert.pem",
    "sslKey": "./privkey.pem",
    "sslCA": "./chain.pem",
    "ports": [
      {
        "port": 6666,
        "difficulty": 5000,
        "desc": "Low end hardware"
      },
      {
        "port": 6667,
        "difficulty": 15000,
        "desc": "Mid range hardware"
      },
      {
        "port": 6668,
        "difficulty": 500000,
        "desc": "High end hardware"
      }
      
    ],
    "varDiff": {
      "minDiff": 100,
      "maxDiff": 100000000,
      "targetTime": 120,
      "retargetTime": 60,
      "variancePercent": 30,
      "maxJump": 100
    },
    "paymentId": {
      "addressSeparator": ".",
      "validation": false,
      "validations": ["1,16", "64"],
      "ban": true
    },
    "fixedDiff": {
      "enabled": true,
      "addressSeparator": "."
    },
    "shareTrust": {
      "enabled": true,
      "min": 10,
      "stepDown": 3,
      "threshold": 10,
      "penalty": 30
    },
    "banning": {
      "enabled": false,
      "time": 600,
      "invalidPercent": 25,
      "checkThreshold": 30
    },
    "slushMining": {
      "enabled": false,
      "weight": 300,
      "blockTime": 60,
      "lastBlockCheckRate": 1
    }
  },

  "payments": {
                "enabled": true,
                "interval": 1800,
                "maxAddresses": 15,
                "mixin": 15,
                "priority": 1,
                "transferFee": 50000000,
                "dynamicTransferFee": true,
                "minerPayFee": true,
                "minPayment": 50000000000,
                "maxPayment": 50000000000000,
                "maxTransactionAmount": 50000000000000,
                "denomination": 100000000
        },

        "blockUnlocker": {
                "enabled": true,
                "interval": 30,
                "depth": 60,
                "poolFee": 0.8,
                "devDonation": 0.2,
                "useFirstVout": true
        },

  "api": {
    "enabled": true,
    "hashrateWindow": 600,
    "updateInterval": 5,
    "bindIp": "0.0.0.0",
    "port": 8117,
    "blocks": 30,
    "payments": 30,
    "password": "yourpassword",
    "ssl": true,
    "sslPort": 8119,
    "sslCert": "./cert.pem",
    "sslKey": "./newkey.pem",
    "sslCA": "./chain.pem",
    "trustProxyIP": false
  },

  "daemon": {
    "host": "127.0.0.1",
    "port": 30000
  },

  "wallet": {
    "host": "127.0.0.1",
    "port": 18082
  },

  "redis": {
    "host": "127.0.0.1",
    "port": 6379,
    "auth": null,
    "db": 7,
    "cleanupInterval": 15
  },

  "notifications": {
    "emailTemplate": "email_templates/default.txt",
    "emailSubject": {
      "emailAdded": "Your email was registered",
      "workerConnected": "Worker %WORKER_NAME% connected",
      "workerTimeout": "Worker %WORKER_NAME% stopped hashing",
      "workerBanned": "Worker %WORKER_NAME% banned",
      "blockFound": "Block %HEIGHT% found !",
      "blockUnlocked": "Block %HEIGHT% unlocked !",
      "blockOrphaned": "Block %HEIGHT% orphaned !",
      "payment": "We sent you a payment !"
    },
    "emailMessage": {
      "emailAdded": "Your email has been registered to receive pool notifications.",
      "workerConnected": "Your worker %WORKER_NAME% for address %MINER% is now connected from ip %IP%.",
      "workerTimeout": "Your worker %WORKER_NAME% for address %MINER% has stopped submitting hashes on %LAST_HASH%.",
      "workerBanned": "Your worker %WORKER_NAME% for address %MINER% has been banned.",
      "blockFound": "Block found at height %HEIGHT% by miner %MINER% on %TIME%. Waiting maturity.",
      "blockUnlocked": "Block mined at height %HEIGHT% with %REWARD% and %EFFORT% effort on %TIME%.",
      "blockOrphaned": "Block orphaned at height %HEIGHT% :(",
      "payment": "A payment of %AMOUNT% has been sent to %ADDRESS% wallet."
    },
    "telegramMessage": {
      "workerConnected": "Your worker _%WORKER_NAME%_ for address _%MINER%_ is now connected from ip _%IP%_.",
      "workerTimeout": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has stopped submitting hashes on _%LAST_HASH%_.",
      "workerBanned": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has been banned.",
      "blockFound": "*Block found at height* _%HEIGHT%_ *by miner* _%MINER%_*! Waiting maturity.*",
      "blockUnlocked": "*Block mined at height* _%HEIGHT%_ *with* _%REWARD%_ *and* _%EFFORT%_ *effort on* _%TIME%_*.*",
      "blockOrphaned": "*Block orphaned at height* _%HEIGHT%_ *:(*",
      "payment": "A payment of _%AMOUNT%_ has been sent."
    }
  },

  "email": {
    "enabled": false,
    "fromAddress": "your@email.com",
    "transport": "sendmail",
    "sendmail": {
      "path": "/usr/sbin/sendmail"
    },
    "smtp": {
      "host": "smtp.example.com",
      "port": 587,
      "secure": false,
      "auth": {
        "user": "username",
        "pass": "password"
      },
      "tls": {
        "rejectUnauthorized": false
      }
    },
    "mailgun": {
      "key": "your-private-key",
      "domain": "mg.yourdomain"
    }
  },

  "telegram": {
    "enabled": false,
    "botName": "",
    "token": "",
    "channel": "",
    "channelStats": {
      "enabled": false,
      "interval": 30
    },
    "botCommands": {
      "stats": "/stats",
      "report": "/report",
      "notify": "/notify",
      "blocks": "/blocks"
    }
  },

  "monitoring": {
    "daemon": {
      "checkInterval": 60,
      "rpcMethod": "getblockcount"
    },
    "wallet": {
      "checkInterval": 60,
      "rpcMethod": "getbalance"
    }
  },

  "prices": {
    "source": "cryptonator",
    "currency": "USD"
  },

  "charts": {
    "pool": {
      "hashrate": {
        "enabled": true,
        "updateInterval": 60,
        "stepInterval": 1800,
        "maximumPeriod": 86400
      },
      "miners": {
        "enabled": true,
        "updateInterval": 60,
        "stepInterval": 1800,
        "maximumPeriod": 86400
      },
      "workers": {
        "enabled": true,
        "updateInterval": 60,
        "stepInterval": 1800,
        "maximumPeriod": 86400
      },
      "difficulty": {
        "enabled": true,
        "updateInterval": 1800,
        "stepInterval": 10800,
        "maximumPeriod": 604800
      },
      "price": {
        "enabled": true,
        "updateInterval": 1800,
        "stepInterval": 10800,
        "maximumPeriod": 604800
      },
      "profit": {
        "enabled": true,
        "updateInterval": 1800,
        "stepInterval": 10800,
        "maximumPeriod": 604800
      }
    },
    "user": {
      "hashrate": {
        "enabled": true,
        "updateInterval": 180,
        "stepInterval": 1800,
        "maximumPeriod": 86400
      },
      "worker_hashrate": {
        "enabled": true,
        "updateInterval": 60,
        "stepInterval": 60,
        "maximumPeriod": 86400
      },
      "payments": {
        "enabled": true
      }
    },
    "blocks": {
      "enabled": true,
      "days": 30
    }
  }
}

```

#### 3) Start the pool

```bash
node init.js --config=sispop.json
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```
#### 3.1) Start payment
Go to your command line wallet
```bash
./sispop-wallet-rpc --rpc-bind-port 18082 --wallet-file WALLETNAME --password PASSWORDWALLET --daemon-address http://127.0.0.1:30000/ --untrusted-daemon --disable-rpc-login --rpc-bind-ip 127.0.0.1 --confirm-external-bind
```
This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis
* `chartsDataCollector` - Processes miners and workers hashrate stats and charts
* `telegramBot`	- Processes telegram bot commands


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.

To keep your pool up, on operating system with systemd, you can create add your pool software as a service.  
Use this [example](https://github.com/dvandal/cryptonote-nodejs-pool/blob/master/deployment/cryptonote-nodejs-pool.service) to create the systemd service `/lib/systemd/system/cryptonote-nodejs-pool.service`
Then enable and start the service with the following commands : 

```
sudo systemctl enable cryptonote-nodejs-pool.service
sudo systemctl start cryptonote-nodejs-pool.service
```

#### 4) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to (override daemon setting if set) */
var poolHost = "poolhost.com";

/* Number of coin decimals places (override daemon setting if set) */
"coinDecimalPlaces": 4,

/* Contact email address. */
var email = "support@poolhost.com";

/* Pool Telegram URL. */
var telegram = "https://t.me/YourPool";

/* Pool Discord URL */
var discord = "https://discordapp.com/invite/YourPool";

/*Pool Facebook URL */
var facebook = "https://www.facebook.com/<YourPoolFacebook";

/* Market stat display params from https://www.cryptonator.com/widget */
var marketCurrencies = ["{symbol}-BTC", "{symbol}-USD", "{symbol}-EUR", "{symbol}-CAD"];

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/light.css";

/* Default language */
var defaultLang = 'en';

```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.

#### SSL

You can configure the API to be accessible via SSL using various methods. Find an example for nginx below:

* Using SSL api in `config.json`:

By using this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "https://poolhost:8119";`

* Inside your SSL Listener, add the following:

``` javascript
location ~ ^/api/(.*) {
    proxy_pass http://127.0.0.1:8117/$1$is_args$args;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js` to include the /api. For example:  
`var api = "http://poolhost/api";`

You no longer need to include the port in the variable because of the proxy connection.

* Using his own subdomain, for example `api.poolhost.com`:

```bash
server {
    server_name api.poolhost.com
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    ssl_certificate /your/ssl/certificate;
    ssl_certificate_key /your/ssl/certificate_key;

    location / {
        more_set_headers 'Access-Control-Allow-Origin: *';
        proxy_pass http://127.0.01:8117;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

By adding this you will need to update your `api` variable in the `website_example/config.js`. For example:  
`var api = "//api.poolhost.com";`

You no longer need to include the port in the variable because of the proxy connection.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here:
* Daemon https://wiki.bytecoin.org/wiki/JSON_RPC_API
* Wallet https://wiki.bytecoin.org/wiki/Wallet_JSON_RPC_API


Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [Netdata](https://github.com/firehol/netdata)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever) or [PM2](https://github.com/Unitech/pm2)


Community / Support
===

* [GitHub Issues](https://github.com/dvandal/cryptonote-nodejs-pool/issues)
* [Telegram Group](http://t.me/CryptonotePool)

#### Pools Using This Software

* https://sispop.2realminers.com/

Referral Links
--------------
* NiceHash Miner - Test your mining pool: [https://www.nicehash.com/?refby=938d7799-8f8e-4935-975e-897a1567b1ed](https://www.nicehash.com/?refby=938d7799-8f8e-4935-975e-897a1567b1ed)
* Binance Exchange - Buy and Sell cryptos: [https://www.binance.com/en/register?ref=92696209](https://www.binance.com/en/register?ref=92696209)
* Coinbase Wallet - Buy 100$ USD and get 10$ USD free: [https://www.coinbase.com/join/vandal_y](https://www.coinbase.com/join/vandal_y)
* Shakepay Wallet - Buy 100$ CAD and get 30$ CAD free: [https://shakepay.me/r/VDAIT0G](https://shakepay.me/r/VDAIT0G)

Donations
---------

Thanks for supporting my works on this project! If you want to make a donation to [Dvandal](https://github.com/dvandal/), the developper of this project, you can send any amount of your choice to one of theses addresses:

* SISPOP (SIS): `47aX25v57uAd1pJR54WWgD9jqaZcKawYaTojcK6ao9hQgFkkkrVEqhgDfY61wvM6TogfXDXddwzsh69TB7FVhTYQRRBmSRU`
Credits
---------

* [fancoder](//github.com/fancoder) - Developper on cryptonote-universal-pool project from which current project is forked.
* [dvandal](//github.com/dvandal) - Developer of cryptonote-nodejs-pool software

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html

Pay for build
-------

I will build pool for you 80$ in crypto.
Discord for Support or pay:
discord: 2realminers.com
