#### Build and run Thingsboard

Follow these guides:

- https://thingsboard.io/docs/user-guide/contribution/how-to-contribute/
- https://thingsboard.io/docs/user-guide/install/building-from-source/

In short:

- Clone (Thingsboard)[https://github.com/thingsboard/thingsboard]
- Install Java 11
- Install PostgreSQL [https://www.postgresql.org/download/linux/ubuntu/]
  - Allow all host (including remote) connection to DB [https://www.bigbinary.com/blog/configure-postgresql-to-allow-remote-connection]
  - Find the Location PostgreSQL config files `psql --version` `ls /etc/postgresql/<version>/main/`
  - Edit the **postgresql.conf** to add/enable this config `listen_addresses = '*'` use the command
    `sudo vi /etc/postgresql/<version>/main/postgresql.conf`
  - Add the lines below at the end of **pg_hba.conf** use the command `sudo vi /etc/postgresql/<version>/main/pg_hba.conf`
    ```

    host    all             all              0.0.0.0/0                       md5
    host    all             all              ::/0                            md5
    
    ```
  - Restart PostgreSQL db `sudo systemctl restart postgresql`
  - Check status of db `service postgresql status`
  - Check the host:port config status `netstat -nlt`
  - Login to postgre LocalDB: `sudo -u postgres psql`
  - Create user for TB app `ALTER USER postgres with encrypted password 'postgres';`
  - Login to RemoteDB (if-required): `psql -U postgres -d postgres -h <REMOTE_DB_IP> -W`
  - Update the file `thingsboard-test/application/src/main/resources/thingsboard.yml` with replacing `localhost` by `REMOTE_DB_IP`
  - Find config file location (if-required): `sudo -u postgres psql -c 'SHOW config_file'`
- Initialize dev DB
  - Note: When on Windows, use `application/target/windows/install_dev_db.bat`.
- Build jar: `mvn clean install -DskipTests`
  -  Build skip licensing (if-required): `mvn clean install -DskipTests -Dlicense.skip`
- Start TB: `java -jar application/target/thingsboard-${VERSION}-boot.jar`
- Sign in [if-required: replace localhost with your hostname]: http://localhost:8080 / Username: tenant@thingsboard.org / PW: tenant

#### Configure TB for device attribute inefficiency

Inefficiency being tested:
`./rule-engine/rule-engine-components/src/main/java/org/thingsboard/rule/engine/util/EntitiesRelatedDeviceIdAsyncLoader.java:42`

1. Add relation to device

- Open: http://localhost:8080/devices [if-required: replace localhost with your hostname]
- Select device "Thermostat T1" from left "Devices" menu
- Open relations tab and click "plus"
- Select entity type = "Device" and entity = "Test Device A1"
- Click "Add"
  - Final attributes tab should look like the following:
    ![Thermostat T1 Relations](docs/Thermostat_T1_Relations.png?raw=true "Thermostat T1 Relations")

2. Add attributes node to thermostat rule chain

- Go to Rule Chains: http://localhost:8080/ruleChains [if-required: replace localhost with your hostname]
- Select Thermostat rule chain
- Select "Open rule chain"
- Drag a "related device attributes" node into the graph
- Fill in any name (e.g. Inefficiency Test)
  - No other fields need to be changed
- Click add
- Attach node to output of "Save Timeseries" node (on the Post telemetry branch)
- Select "Success" for link label
  - Final chain should look like the following:
    ![Thermostat rule chain](docs/Thermostat_Rule_Chain.png?raw=true "Thermostat rule chain")
- Click orange check in bottom right to save chain

#### Configure script
- Install nodejs in Ubuntu 18.04
  - Add repo : `curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`
  - Install: `sudo apt install nodejs`
  - Check version `node --version` and `npm --version`
- Change TB host and port, iteration count, and delay if needed in `.env`
  - Iterations/delay also be passed as command-line arguments
- Install dependencies via `npm i`

#### [If-Required] Limit node bandwidth
- Install wondershaper bandwidth limiter: `sudo apt install wondershaper`
- Specify Upload Download limit
  - Old version: `wondershaper [ interface ] [ downlink ] [ uplink ]`
  - Newer version: `wondershaper -a [interface] -d [downlink] -u [uplink]`
- Find Network Interface
  - check `ifconfig`
  - [If required] `sudo apt install net-tools`
- Finally set bandwidth limit example: `sudo wondershaper eno1 256 256` setting both upload/download speed as 256 Kpbs
- Clear all limits: `sudo wondershaper clear eno1`
For more details check [https://averagelinuxuser.com/limit-bandwidth-linux/]

#### Install nodejs in Ubuntu 18.04
- Add suitable node.js version for your OS: `curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`
- Install nodejs: `sudo apt install nodejs`
- Check installed nodejs version: `node --version`
- Check node package manager version: `npm --version`

#### Run script

- `node ./index.js`: Will repeatedly post telemetry to the Thermostat T1 device, causing the rule chain to look up information about the related device (Test Device A1).
