---
title: Minecraft
draft: true
tags:
  - homelab
  - minecraft
  - pterodactyl
---

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



# Permission

## ChestShop
ChestShop's permission nodes

Permission nodes enabled by default:
**ChestShop.shop.*** - lets you create shops, buy from them and sell to them  
↓ (it grants people these nodes)  
**ChestShop.shop.create** - lets you create shops  
**ChestShop.shop.buy** - lets you buy from shops  
**ChestShop.shop.sell** - lets you sell to shops  
  

Other permission nodes:
**ChestShop.shop.buy.(itemid)** - lets you buy only specific items from shops  
**ChestShop.shop.sell.(itemid)** - lets you sell only specific items from shops  
**ChestShop.iteminfo** - allows access to the /iteminfo command  
**ChestShop.shopinfo** - allows access to the /shopinfo command  
**ChestShop.accesstoggle** - allows access to the /csaccess command  
**ChestShop.admin** - (granted to OPs) - lets you create Admin Shops and create/destroy other people's shops as well as use the /csgive, /csmetrics and /csversion commands.  
**ChestShop.mod** - lets you look into other shop chests  
**ChestShop.name.(playername)** - lets you create, destroy and access shops for another player. Hint: You can use ChestShop.name.* for all player. Same with the permissions below.  
**ChestShop.othername.create.(**playername**)** - lets you create shops for another player  
**ChestShop.othername.destroy.(**playername**)** - lets you destroy shops for another player  
**ChestShop.othername.access.(**playername**)** - lets you access shops for another player  
**ChestShop.nofee** - you don't have to pay shop creation fee  
**ChestShop.notax.buy** - you don't have to pay tax when buying at a shop  
**ChestShop.notax.sell** - you don't have to pay tax when selling at a shop  
  

More ChestShop.shop.create nodes:
**ChestShop.shop.create.buy** - lets you create buy-only shops  
**ChestShop.shop.create.sell** - lets you create sell-only shops  
**ChestShop.shop.create.(itemid)** - ...shops that sell specific items  
**ChestShop.shop.create.food** - ...shops that sell food  
**ChestShop.shop.create.diamondgrade** - lets you create shops that sell diamond-grade items  
**ChestShop.shop.create.irongrade, ....goldgrade, ...stonegrade, ...woodgrade  
ChestShop.shop.create.diamondarmor, ...goldarmor**, etc.  
**ChestShop.shop.create.diamondtools, ...goldtools**, etc.  
**ChestShop.shop.create.ore  
ChestShop.shop.create.ingots  
ChestShop.shop.create.monsterdrops**

## ClearLagg

**Commands**  
( Permissions are just lagg.command-name)

- **/lagg clear** (Clears configured entities)
- **/lagg check [world1, world2...]** (Displays world information + more)
- **/lagg reload** (Reloads the configuration)
- **/lagg killmobs** (Kills configured mobs)
- **/lagg area radius** (Removes entities in given radius)
- **/lagg tpchunk  x  z  [world]** (Teleports to given chunk)
- **/lagg admin** (Manage modules)
- **/lagg gc** (Force request Garbage collection [NOT RECOMMENDED])
- **/lagg tps** (View estimated TPS [Not as accurate as Spigot's /tps])
- **/lagg halt** (Temporary disable configured basic server functions)
- **/lagg sampleMemory** time (Sample memory usage per-tick, and garbage collection timings)
- **/lagg sampleTicks** ticks [raw] (Sample how long ticks took to complete)
- **/lagg unloadchunks** (Attempts to unload chunks - [Not recommended on later Spigot's])
- **/lagg profile time type** (Profile certain activities such as redstone to see which chunk is the most active)
- **/lagg memory** (View your memory heap in realtime)
- **/lagg performance** (View your main-thread usage in real-time)

## JobsReborn

[Permissions · Zrips/Jobs Wiki · GitHub](https://github.com/Zrips/Jobs/wiki/Permissions)
Have custom config

## Essentials
Have custom config

## veinminer
[Veinminer - Minecraft Data Pack](https://modrinth.com/datapack/veinminer)

## tabtps
[TabTPS - Minecraft Plugin](https://modrinth.com/plugin/tabtps)


# Requirements
## Stellarity
enable-command-block: true >  server.properties

## Cadmus & Argonauts

[Project Odyssey | Terrarium Docs](https://docs.terrarium.earth/)