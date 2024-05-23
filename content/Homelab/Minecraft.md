[Chunky - Minecraft Plugin](https://modrinth.com/plugin/chunky)
[Veinminer - Minecraft Data Pack](https://modrinth.com/datapack/veinminer)
[CoreProtect - Minecraft Plugin](https://modrinth.com/plugin/coreprotect)



[spark](https://spark.lucko.me/)


Pterodactyl/Chunky `[warning][os,thread] Failed to start thread "Unknown thread" - pthread_create failed (EAGAIN) for attributes: stacksize: 1024k, guardsize: 0k, detached.` error fix

```yaml title="etc/pterodactyl/config.yaml"
docker:
	...
	container_pid_limit: 0
```

default = 512
unlimited = 0 = probably gevaarlijk om  te laten staan
chunky = 1024