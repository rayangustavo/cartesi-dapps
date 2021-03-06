# SQLite-extended DApp

This example extends the SQLite Dapp and shows how to build and interact with a Cartesi Rollups application that acts transparently on personal data, seeking to comply with the requirements of the GDPR - General Data Protection Regulation. 

There is a already created database with a table named medical. You can perform predefined operations to create, update and delete records, acting as the data subject. On the other hand, you can also operate as a data controller/processor to perform queries and build new datasets for distribution or specific processing. Data subject and data controller/processor are actors within the definition of GDPR - General Data Protection Regulation

The example highlights some aspects of GDPR, such as the guarantee of the data subject to no longer have their data in a dataset or the guarantee of their right to know how their data is being used, as well as the end consumers of the data will also be guaranteed of legitimate origin on the data used. Right to be forgotten and other security aspects are not present in this conceptual Dapp.

## Building the environment

To run the SQLite example, clone the repository as follows:

```shell
$ git clone https://github.com/cartesi/rollups-examples.git
```

Then, build the back-end for the example:

```shell
$ cd rollups-examples/sqlite-extended
$ make machine
```

## Running the environment

In order to start the containers in production mode, simply run:

```shell
$ docker-compose up --build
```

_Note:_ If you decide to use [Docker Compose V2](https://docs.docker.com/compose/cli-command/), make sure you set the [compatibility flag](https://docs.docker.com/compose/cli-command-compatibility/) when executing the command (e.g., `docker compose --compatibility up`).

Allow some time for the infrastructure to be ready.
How much will depend on your system, but after some time showing the error `"concurrent call in session"`, eventually the container logs will repeatedly show the following:

```shell
server_manager_1      | Received GetVersion
server_manager_1      | Received GetStatus
server_manager_1      |   default_rollups_id
server_manager_1      | Received GetSessionStatus for session default_rollups_id
server_manager_1      |   0
server_manager_1      | Received GetEpochStatus for session default_rollups_id epoch 0
```

To stop the containers, first end the process with `Ctrl + C`.
Then, remove the containers and associated volumes by executing:

```shell
$ docker-compose down -v
```

## Interacting with the application

With the infrastructure in place, you can interact with the application using a set of Hardhat tasks. 

First, go to a separate terminal window, switch to the `sqlite-extended/contracts` directory, and run `yarn`:

```shell
$ cd contracts/
$ yarn
```

Then, send an input as follows to insert your anonymous personal data into our DApp's internal database:

```shell
$ npx hardhat --network localhost sqlite:addInput --input "INSERT INTO Medical VALUES ('35', 'male', '32.4', '0', 'no', 'southeast', '10000.0000' )"
```
column names are as follows 
```
# 0,age,TEXT,0,,0
# 1,sex,TEXT,0,,0
# 2,bmi,TEXT,0,,0
# 3,children,TEXT,0,,0
# 4,smoker,TEXT,0,,0
# 5,region,TEXT,0,,0
# 6,charges,TEXT,0,,0

*bmi = kg/m?? 
```

The input will have been accepted when you receive a response similar to the following one:

```shell
Added input 'INSERT INTO Medical VALUES ('35', 'male', '32.4', '0', 'no', 'southeast', '10000.0000' )' to epoch '0' (index: '0', timestamp: 1651462950, signer: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266, tx: 0xb5be62ec2ac76d5a7a50e681b9b1ac2e5de9e282d9aa69c84c1fae1141de9ae3)
```
Once data has been inserted into the database, it can be queried, updated or deleted using regular SQL statements such as the following:

```shell
$ npx hardhat --network localhost sqlite:addInput --input "SELECT * FROM Medical WHERE age='35' AND sex='male' AND bmi='32.4' AND children='0' AND smoker='no' AND region='southeast' AND charges='10000.0000'"
$ npx hardhat --network localhost sqlite:addInput --input "UPDATE Medical SET charges='11000.0000' WHERE age='35' AND sex='male' AND bmi='32.4' AND children='0' AND smoker='no' AND region='southeast' AND charges='10000.0000'"
$ npx hardhat --network localhost sqlite:addInput --input "DELETE FROM Medical WHERE age='35' AND sex='male' AND bmi='32.4' AND children='0' AND smoker='no' AND region='southeast' AND charges='11000.0000'"
```

Note that the query's results will not be retrieved immediately. Rather, whenever a submitted input corresponds to a valid SQL query, the DApp will make the corresponding results available in the form of a _notice_.

In order to verify the notices generated by your inputs, run the command:

```shell
$ curl http://localhost:4000/graphql -H 'Content-Type: application/json' -d '{ "query" : "query getNotice { GetNotice( query: { session_id: \"default_rollups_id\", epoch_index: \"0\", input_index: \"0\" } ) { session_id epoch_index input_index notice_index payload } }" }'
```
Or

```shell
$ npx hardhat --network localhost sqlite:getNotices --epoch 0 --payload string
```

Assuming the command "SELECT * FROM Medical" was performed, the response should be something like this:

```shell
{"session_id":"default_rollups_id","epoch_index":"0","input_index":"1","notice_index":"0","payload":"[[\"19\", \"female\", \"27.9\", \"0\", \"yes\", \"southwest\", \"16884.924\"], [\"18\", \"male\", \"33.77\", \"1\", \"no\", \"southeast\", \"1725.5523\"], [\"28\", \"male\", \"33\", \"3\", \"no\", \"southeast\", \"4449.462\"], [\"33\", \"male\", \"22.705\", \"0\", \"no\", \"northwest\", \"21984.47061\"], [\"32\", \"male\", \"28.88\", \"0\", \"no\", \"northwest\", \"3866.8552\"], [\"31\", \"female\", \"25.74\", \"0\", \"no\", \"southeast\", \"3756.6216\"], [\"46\", \"female\", \"33.44\", \"1\", \"no\", \"southeast\", \"8240.5896\"], [\"37\", \"female\", \"27.74\", \"3\", \"no\", \"northwest\", \"7281.5056\"], [\"37\", \"male\", \"29.83\", \"2\", \"no\", \"northeast\", \"6406.4107\"], [\"60\", \"female\", \"25.84\", \"0\", \"no\", \"northwest\", \"28923.13692\"], [\"25\", \"male\", \"26.22\", \"0\", \"no\", \"northeast\", \"2721.3208\"], [\"62\", \"female\", \"26.29\", \"0\", \"yes\", \"southeast\", \"27808.7251\"], [\"23\", \"male\", \"34.4\", \"0\", \"no\", \"southwest\", \"1826.843\"], [\"56\", \"female\", \"39.82\", \"0\", \"no\", \"southeast\", \"11090.7178\"], [\"27\", \"male\", \"42.13\", \"0\", \"yes\", \"southeast\", \"39611.7577\"], [\"19\", \"male\", \"24.6\", \"1\", \"no\", \"southwest\", \"1837.237\"], [\"52\", \"female\", \"30.78\", \"1\", \"no\", \"northeast\", \"10797.3362\"], [\"23\", \"male\", \"23.845\", \"0\", \"no\", \"northeast\", \"2395.17155\"], [\"56\", \"male\", \"40.3\", \"0\", \"no\", \"southwest\", \"10602.385\"], [\"30\", \"male\", \"35.3\", \"0\", \"yes\", \"southwest\", \"36837.467\"], [\"60\", \"female\", \"36.005\", \"0\", \"no\", \"northeast\", \"13228.84695\"], [\"30\", \"female\", \"32.4\", \"1\", \"no\", \"southwest\", \"4149.736\"], [\"18\", \"male\", \"34.1\", \"0\", \"no\", \"southeast\", \"1137.011\"], [\"34\", \"female\", \"31.92\", \"1\", \"yes\", \"northeast\", \"37701.8768\"], [\"37\", \"male\", \"28.025\", \"2\", \"no\", \"northwest\", \"6203.90175\"], [\"59\", \"female\", \"27.72\", \"3\", \"no\", \"southeast\", \"14001.1338\"], [\"63\", \"female\", \"23.085\", \"0\", \"no\", \"northeast\", \"14451.83515\"], [\"55\", \"female\", \"32.775\", \"2\", \"no\", \"northwest\", \"12268.63225\"], [\"23\", \"male\", \"17.385\", \"1\", \"no\", \"northwest\", \"2775.19215\"], [\"31\", \"male\
.
.
.
```

It can take many minutes for the query to be executed by the Notice command, so if the GetNotice command does not return anything, wait and try again.

To make some dataset based on horizontal filter, as BRAID architecture*, filtering only one kind of gender:

```shell
npx hardhat --network localhost sqlite:addInput --input "CREATE TABLE Horizontal_Filter (age text, sex text, bmi text, children text, smoker text, region text, charges text)"
npx hardhat --network localhost sqlite:addInput --input "INSERT INTO Horizontal_Filter SELECT * FROM Medical WHERE sex='male'"
```
to make datasets based on vertical filter, as BRAID architecture*, excluding some attributes:

```shell
npx hardhat --network localhost sqlite:addInput --input "CREATE TABLE Vertical_Filter (age text, bmi text, charges text)"
npx hardhat --network localhost sqlite:addInput --input "INSERT INTO Vertical_Filter (age, bmi, charges) SELECT age, bmi, charges FROM Medical"
```

*AMNESIA: A Technical Solution towards GDPR-compliant Machine Learning. January 2020. DOI:10.5220/0008916700210032. Conference: 6th International Conference on Information Systems Security and Privacy

These datasets can be consumed by solutions inside or outside the cartesi machine, using only simple select statements, maintaining clear transparency of what data is actually being used.

## Advancing time

To advance time, in order to simulate the passing of epochs, run:

```shell
$ npx hardhat --network localhost util:advanceTime --seconds 864010
```

## Running the environment in host mode

When developing an application, it is often important to easily test and debug it. For that matter, it is possible to run the Cartesi Rollups environment in [host mode](../README.md#host-mode), so that the DApp's back-end can be executed directly on the host machine, allowing it to be debugged using regular development tools such as an IDE.

The first step is to run the environment in host mode using the following command:

```shell
$ docker-compose -f docker-compose.yml -f docker-compose-host.yml up --build
```

The next step is to run the SQLite server in your machine. The application is written in Python, so you need to have `python3` installed.

In order to start the server, run the following commands in a dedicated terminal:

```shell
$ cd sqlite-extended/server/
$ python3 -m venv .env
$ . .env/bin/activate
$ pip install -r requirements.txt
$ HTTP_DISPATCHER_URL="http://127.0.0.1:5004" gunicorn --reload --workers 1 --bind 0.0.0.0:5003 sqlite:app
```

This will run the server on port `5003` and send the corresponding notices to port `5004`. The server will also automatically reload if there is a change in the source code, enabling fast development iterations.

The final command, which effectively starts the server, can also be configured in an IDE to allow interactive debugging using features like breakpoints. In that case, it may be interesting to add the parameter `--timeout 0` to gunicorn, to avoid having it time out when the debugger stops at a breakpoint.

After the server successfully starts, it should print an output like the following:

```
[2022-05-02 00:39:21 -0300] [398197] [INFO] Starting gunicorn 19.9.0
[2022-05-02 00:39:21 -0300] [398197] [INFO] Listening at: http://0.0.0.0:5003 (398197)
[2022-05-02 00:39:21 -0300] [398197] [INFO] Using worker: sync
/usr/lib/python3.8/os.py:1023: RuntimeWarning: line buffering (buffering=1) isn't supported in binary mode, the default buffer size will be used
  return io.open(fd, *args, **kwargs)
[2022-05-02 00:39:21 -0300] [398199] [INFO] Booting worker with pid: 398199
[2022-05-02 00:39:25,108] INFO in sqlite: HTTP dispatcher url is http://127.0.0.1:5004
```

After that, you can interact with the application normally [as explained above](#interacting-with-the-application).

When you add an input, you should see it being processed by the server as follows:

```shell
[2022-05-02 00:42:39,311] INFO in sqlite: Received advance request body {'metadata': {'msg_sender': '0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266', 'epoch_index': 0, 'input_index': 0, 'block_number': 11, 'time_stamp': 1651462950}, 'payload': '0x494e5345525420494e544f204d65646963616c2056414c5545532028273335272c20276d616c65272c202733322e34272c202730272c20276e6f272c2027736f75746865617374272c202731303030302e30303030272029'}
[2022-05-02 00:42:39,318] INFO in sqlite: Received statement: 'INSERT INTO Medical VALUES ('35', 'male', '32.4', '0', 'no', 'southeast', '10000.0000' )'
[2022-05-02 00:42:39,343] INFO in sqlite: Finishing
[2022-05-02 00:42:39,444] INFO in sqlite: Received finish status 202
```

Finally, to stop the containers, removing any associated volumes, execute:

```shell
$ docker-compose -f docker-compose.yml -f docker-compose-host.yml down -v
```
