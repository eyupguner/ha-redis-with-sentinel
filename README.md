# HA Redis Cluster with Sentinel and Sentinel Proxy
    This chart will install ha redis cluster with sentinel on kubernetes. You can reach the source files on  https://github.com/eyupguner/ha-redis-with-sentinel. The Redis will work one master pod and two slaves pods. Slaves pods will replicate from master pod. If master pod crash, sentinels will select new master.  

# This chart will create following recources. 

    I did not use StatefulSet because of redis/master configuration.  
    - Three Redis Pod
    - Three Sentinel Pods 
    - Three Redis Pod's PVC called pvc-redis
    - Three Sentinel Pod's PVC called pvc-sentinel
    - Redis Pod's ConfigMap called redis-config
    - Sentinel Proxy Pod and PVC
    - 8 ClusterIP Service and 1 NodePort Service

# Service Explain. 
    - svc-redis-sentinel-proxy service forwards internal request to only master pod.
    - svc-redis-sentinel-proxy-nodeport forwards external request to only master pod.
    - Other services forward request to own pods.

# Values.yaml Explain. 
    
    #Change storage class and capacity.
    storage:
    storageClassName: ""
    redisStorage: "10Gi"
    sentinelStorage: "1Gi"

    #Redis and sentinel security pass.
    securityContext:
    redisPass: "admin"
    sentinelPass: "admin"

    #Redis pods recource and limits. 
    resources:
    limits:
        cpu: 1000m
        memory: 2048Mi
    requests:
        cpu: 500m
        memory: 1024Mi 

# Redis Config 
    - Here is the Redis Configuration. You can change configuration after deployment. Pods will update after configmap change. 

    daemonize no
    pidfile "/var/run/redis/6379.pid"
    dir "/data"
    port 6379
    bind 0.0.0.0
    timeout 0
    tcp-keepalive 0
    tcp-backlog 64534
    logfile "/data/redis.log"
    loglevel notice
    syslog-enabled yes
    syslog-ident "redis_6379"
    syslog-facility user
    databases 128

    # Snapshotting
    save 900 1
    save 300 10
    save 60 10000
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename "dump.rdb"

    # Replication
    replica-serve-stale-data yes
    replica-read-only yes
    repl-disable-tcp-nodelay no
    replica-priority 100
    min-replicas-max-lag 10
    masterauth {{ .Values.securityContext.redisPass }} // From values.yaml

    # Security
    requirepass {{ .Values.securityContext.redisPass }} // From values.yaml

    # Limits
    maxclients 65504
    maxmemory-policy volatile-lru

    # Append Only Mode
    appendonly no
    appendfilename "appendonly.aof"
    appendfsync everysec
    no-appendfsync-on-rewrite no
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb

    # Lua
    lua-time-limit 5000

    # Slow Log
    slowlog-log-slower-than 10000
    slowlog-max-len 128

    # Event Notification
    notify-keyspace-events "xE"

    # Advanced
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    list-max-ziplist-entries 512
    list-max-ziplist-value 64
    set-max-intset-entries 512
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    activerehashing yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit replica 1gb 1gb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    hz 10
    aof-rewrite-incremental-fsync yes