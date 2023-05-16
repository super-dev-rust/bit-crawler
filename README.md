# Bit-crawler (bitnodes)

Scan the Bitcoin nodes and find the malicious ones.

### Dependencies and setup

```
# Install pyenv dependencies
sudo apt update && sudo apt install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev apt-transport-https libevent-dev libzstd-dev tcpdump

# Install pyenv
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
cd ~/.pyenv && src/configure && make -C src
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo 'eval "$(pyenv init -)"' >> ~/.profile

# Install Redis
cd && wget https://github.com/redis/redis/archive/7.0.10.tar.gz && tar xzf 7.0.10.tar.gz && cd redis-7.0.10 && make
sudo make install

sudo nano /etc/init.d/redis_0
_____________________________
#!/bin/sh
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis_0.pid
CONF="/etc/redis/0.conf"
REDISSOCKET="/run/redis.sock"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
            echo "$PIDFILE exists, process is already running or crashed"
        else
            echo "Starting Redis server..."
            $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -s $REDISSOCKET -a $REDISPASSWORD shutdown
            while [ -x /proc/${PID} ]
            do
                echo "Waiting for Redis to shutdown ..."
                sleep 1
            done
            echo "Redis stopped"
        fi
        ;;
    status)
        PID=$(cat $PIDFILE)
        if [ ! -x /proc/${PID} ]
        then
            echo 'Redis is not running'
        else
            echo "Redis is running ($PID)"
        fi
        ;;
    restart)
        $0 stop
        $0 start
        ;;
    *)
        echo "Please use start, stop, restart or status as first argument"
        ;;
esac
_____________________________

sudo chmod 755 /etc/init.d/redis_0
sudo update-rc.d redis_0 defaults
sudo mkdir /etc/redis
sudo nano /etc/redis/0.conf
_____________________________
pidfile /var/run/redis_0.pid
daemonize yes
port 0
unixsocket /run/redis.sock
unixsocketperm 777
save ""
maxclients 50000
maxmemory 34326183936
maxmemory-policy volatile-lru
notify-keyspace-events K$lz
activerehashing no
client-output-buffer-limit normal 512mb 256mb 300
client-output-buffer-limit replica 512mb 256mb 300
client-output-buffer-limit pubsub 512mb 256mb 300
_____________________________
sudo service redis_0 restart

# Setup project
source ~/.zshrc
pyenv install 3.11.2
cd && git clone https://github.com/ayeowch/bitnodes.git && cd bitnodes
~/.pyenv/versions/3.11.2/bin/python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pytest
```

### Run

Run the master and some workers with this command:

```
source venv/bin/activate
python crawl.py conf/crawl.conf.default master
```

### More info

See also [Provisioning Bitcoin Network Crawler](https://github.com/ayeowch/bitnodes/wiki/Provisioning-Bitcoin-Network-Crawler).
