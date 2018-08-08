
### 基于EOS源码部署本地节点


```
EOSIO tag: v1.0.10
系统：ubuntu 16.10
```


1. 下载源代码

```
$ git clone https://github.com/eosio/eos --recursive
$ git checkout v1.0.10
```

2. build eos

```
$ sh eosio_build.sh Linux
```

当最后出现：EOSIO has been successfully built， 代表build成功了。
执行如下命令测试(命令有差异，根据实际提示命令执行)：

```
$ export PATH=${HOME}/opt/mongodb/bin:$PATH
$ /home/sunyoubo/opt/mongodb/bin/mongod -f /home/sunyoubo/opt/mongodb/mongod.conf &
$ cd /home/sunyoubo/eos/eos/build; make test
```

直到至少显示 37/37测试通过（版本不同有差异）。

3. 生成执行文件

```
$ cd build
$ sudo make install
```

这时候自动生成nodeos等执行文件到/usr/local/bin/目录下。

4. 启动单节点测试网


```
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --contracts-console --plugin eosio::http_plugin --access-control-allow-origin='*'
```

启动成功后会在用户当前目录下创建目录：/home/sunyoubo/.local/share/eosio/nodeos/config.ini
删除历史区块启动：

```
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --contracts-console --plugin eosio::http_plugin –delete-all-blocks --access-control-allow-origin='*'
```


5. 测试部署eosio.token合约

创建钱包：

```
$ cd  /home/sunyoubo/eos/eos/build/programs/cleos
$ cleos --wallet-url http://127.0.0.1:8888 wallet create
```

导入eosio账户密钥：

在文件/home/sunyoubo/.local/share/eosio/nodeos/config/config.ini中找到signature-provider即为eosio账户密钥对("EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3")

```
$ cleos --wallet-url http://127.0.0.1:8888 wallet unlock
$ cleos --wallet-url http://127.0.0.1:8888 wallet import "5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"
```

为eosio.token账号创建2对密钥对，OwnerKey和ActiveKey，合约和账户需要绑定起来：

```
$ cleos --wallet-url http://127.0.0.1:8888 create key
Private key: 5JWEaPV3dVg5bbFY8ZoofrJMJBGAkeDANrJiXUwzY7rJH93pLgc
Public key: EOS7oGS3v7Pg3AgNScgSUzE25fu9JkmoZ34jDSaUxzqz3mBHUTjBW
```

```
$ cleos --wallet-url http://127.0.0.1:8888 create key
Private key: 5JDGYbMSe1VP1SgtfWGEE73udPFjeSzu6zQ4U4GJ4pewoWKiGhx
Public key: EOS6APWJbmARHgGvVfG1E4FWzac5k7cb4W5BWzY6Thay8ccXWrTXF
```


导入2对密钥的私钥到default钱包：

```
$ cleos --wallet-url http://127.0.0.1:8888 wallet import "5JWEaPV3dVg5bbFY8ZoofrJMJBGAkeDANrJiXUwzY7rJH93pLgc"
$ cleos --wallet-url http://127.0.0.1:8888 wallet import "5JDGYbMSe1VP1SgtfWGEE73udPFjeSzu6zQ4U4GJ4pewoWKiGhx"
```


创建eosio.token账号：

```
格式如：cleos create account eosio token {public-OwnerKey} {public-ActiveKey}
```


```
$ cleos --wallet-url http://127.0.0.1:8888 create account eosio token  EOS7oGS3v7Pg3AgNScgSUzE25fu9JkmoZ34jDSaUxzqz3mBHUTjBW  EOS6APWJbmARHgGvVfG1E4FWzac5k7cb4W5BWzY6Thay8ccXWrTXF
```

验证账号信息：

```
$ cleos --wallet-url http://127.0.0.1:8888 get account token --json
```


部署eosio.token合约：

```
$ cd /home/sunyoubo/eos/eos/build
$ cleos --wallet-url http://127.0.0.1:8888 set contract token ./contracts/eosio.token/ ./contracts/eosio.token/eosio.token.wast ./contracts/eosio.token/eosio.token.abi
```

验证上传code:

```
$ cleos  --wallet-url http://127.0.0.1:8888 get code token
```


创建并发行token:

```
$ cleos --wallet-url http://127.0.0.1:8888 push action token create '{"issuer":"token","maximum_supply":"1000000.0000 YBT","can_freeze":"0","can_recall":"0","can_whitelist":"0"}' -p token
$ cleos --wallet-url http://127.0.0.1:8888  push action token issue '{"to":"token","quantity":"10000.0000 YBT","memo":""}' -p token
```

查询账户token余额：

```
$ cleos --wallet-url http://127.0.0.1:8888 get table token token accounts
```


转账token:

```
$ cleos --wallet-url http://127.0.0.1:8888 push action token transfer '{"from":"token","to":"eosio","quantity":"20.0000 YBT","memo":"my first transfer"}' -p token

```
查询账户余额：

```
$ cleos --wallet-url http://127.0.0.1:8888 get table token token accounts
$ cleos --wallet-url http://127.0.0.1:8888 get table token eosio accounts
```

