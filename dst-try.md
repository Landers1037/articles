---
title: 饥荒服务器搭建尝试
name: dst-try
date: 2020-01-17
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


和好兄弟一起开黑饥荒的时候，因为个人电脑的服务器性能太低老是掉线，于是在阿里云的服务器上自建了专用服务器，这是一点心得


<!--more-->


和好兄弟一起开黑饥荒的时候，因为个人电脑的服务器性能太低老是掉线，于是在阿里云的服务器上自建了专用服务器，这是一点心得

<!--more-->

# 写在最前

之前使用的US服务器，在运行饥荒时显示专用服务器已挂载到US Lobby，分析饥荒可能分国内和国外专用服务器区域，所以还是选择国内主机响应更快

## Linux主机配置

CentOS 7 64位

## 开始搭建

### 环境配置

因为Linux上的steam和饥荒程序都是32位程序，而我们的系统是64位系统在编译32位程序时可能会出现问题，所以提前把编译环境安装好

```bash
yum -y update 
yum install glibc.i686 libstdc++.i686
```

不同版本的CentOS自带的库不一样，可能已经自带这些库。在后面的编译中可能遇到libcurl库缺少的问题，如果出现该问题先查看是否存在该库，不存在则安装

```bash
cd /usr/lib
cat libcurl
```

查看是否存在此库文件，不存在则

```bash
yum install libcurl.i686
```

创建动态链接库的软链接，见**饥荒安装配置**

### steam安装

```bash
cd /home && mkdir steam
cd /home/steam
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
```

解压安装

```bash
tar -xvzf steamcmd_linux.tar.gz
./steamcmd.sh
```

安装完成后会进入steam的命令模式`Steam>`

### 饥荒安装

在Steam命令模式下，登录你的steam账户

```bash
Steam>login 你的用户名
要求你输入密码，可能还需要验证steam令牌
```

安装游戏

```bash
force_install_dir /home/dst_server
app_update 343050 validate
```

等待安装完成后退出steam,输入`quit`

此时在你的`/home/dst_server`目录下已经有安装好的饥荒服务端

```bash
cd /home/dst_server
cd bin
echo ./dontstarve_dedicated_server_nullrenderer -console -cluster MyDediServer -shard Master > master.sh
echo ./dontstarve_dedicated_server_nullrenderer -console -cluster MyDediServer -shard Caves > caves.sh

chmod +x master.sh caves.sh
```

到这里我们生成了饥荒的地上世界`master`和洞穴`caves`的启动脚本

#### 测试启动

```bash
./master.sh
```

> 这里可能出现错误，动态编译库缺失
>
> ```bash
> ln -s /usr/lib/libcurl.so.4 /home/dst_server/bin/lib32/libcurl-gnutls.so.4
> ```
>
> 

启动成功后，`ctrl+c`退出程序，配置世界

### 饥荒配置

#### 世界

在`/home/dstsave`目录下可以看到生成的新世界`World1`，如果刚才只是启动了master脚本则里面只有`Master`目录代表地上世界。

```bash
cd Master
nano server.ini
```

**配置服务器文件**

```bash
cd /home/dstsave/World1
nano cluster.ini
```

修改`cluster.ini`文件

```ini
[GAMEPLAY]
max_players = 6
pvp = false
game_mode = endless
pause_when_empty = true
vote_kick_enabled = true

[NETWORK]
cluster_description = 服务器描述
cluster_name = 服务器名称
cluster_password = 服务器密码
cluster_intention = cooperative

[MISC]
max_snapshots = 6
console_enabled = true

[SHARD]
shard_enabled = true
bind_ip = 你的服务器ip
master_ip = 你的服务器ip
master_port = 10086
cluster_key = defaultPass
```

> 简单的生成方法，在电脑饥荒里创建世界，设置好需要生成的世界，然后在`我的文档/klei/一串数字/Cluster_x`目录下找到你创建的新世界的ini文件，复制里面的内容
>
> 记得修改端口和服务器ip地址

**生成token文件**

参考keli官网的饥荒token生成

把生成的token文件放在`cluster_token.txt`文件内，该文件应该放在服务器的`/home/dstsave/`下，不确定位置就每个目录放一个

**配置地上世界文件**

```ini
[NETWORK]
server_port = 10999

[SHARD]
is_master = true

[STEAM]
authentication_port = 10011
master_server_port = 10012

[ACCOUNT]
encode_user_path = true
```

`NETWORK`里的端口是饥荒运行的端口，`STEAM`里的端口是steam运行的端口，设置完端口记得在服务器的**安全组**里开放对应的端口（如何配置安全组，自行百度）

**配置世界管理员**

在你的世界目录下找到存档位置，`/Master/save`在里面添加`adminlist.txt`文件，每行放一个用户的标识ID

**配置地图文件**

为了简化步骤，少写代码，`Mater/ worldgenoverride.lua`文件直接在电脑的饥荒客户端生成。

在客户端创建世界后，在`我的文档/klei`在找到你的世界配置文件即可

*默认配置参数*

```lua
return {
    override_enabled = true,
    preset = "SURVIVAL_TOGETHER",       -- "SURVIVAL_TOGETHER", "MOD_MISSING", "SURVIVAL_TOGETHER_CLASSIC", "SURVIVAL_DEFAULT_PLUS", "COMPLETE_DARKNESS", "DST_CAVE", "DST_CAVE_PLUS"
    overrides = {
        -- default is "never", "rare", "default", "often", "always"
  
        -- MISC
        task_set = "default",           -- "classic", "default", "cave_default"
        start_location = "default",     -- "caves", "default", "plus", "darkness"
        world_size = "default",         -- "small", "medium", "default", "huge"
        branching = "default",          -- "never", "least", "default", "most"
        loop = "default",               -- "never", "default", "always"
        autumn = "default",             -- "noseason", "veryshortseason", "shortseason", "default", "longseason", "verylongseason", "random"
        winter = "default",             -- "noseason", "veryshortseason", "shortseason", "default", "longseason", "verylongseason", "random"
        spring = "default",             -- "noseason", "veryshortseason", "shortseason", "default", "longseason", "verylongseason", "random"
        summer = "default",             -- "noseason", "veryshortseason", "shortseason", "default", "longseason", "verylongseason", "random"
        season_start = "default",       -- "default", "winter", "spring", "summer", "autumnorspring", "winterorsummer", "random"
        day = "default",                -- "default", "longday", "longdusk", "longnight", "noday", "nodusk", "nonight", "onlyday", "onlydusk", "onlynight"
        weather = "default",
        earthquakes = "default",
        lightning = "default",
        frograin = "default",
        wildfires = "default",
        touchstone = "default",
        regrowth = "default",           -- "veryslow", "slow", "default", "fast", "veryfast"
        cavelight = "default",          -- "veryslow", "slow", "default", "fast", "veryfast"
        boons = "default",
        prefabswaps_start = "default",  -- "classic", "default", "highly random"
        prefabswaps = "default",        -- "none", "few", "default", "many", "max"
  
        -- RESOURCES
        flowers = "default",
        grass = "default",
        sapling = "default",
        marshbush = "default",
        tumbleweed = "default",
        reeds = "default",
        trees = "default",
        flint = "default",
        rock = "default",
        rock_ice = "default",
        meteorspawner = "default",
        meteorshowers = "default",
        mushtree = "default",
        fern = "default",
        flower_cave = "default",
        wormlights = "default",
  
        -- UNPREPARED
        berrybush = "default",
        carrot = "default",
        mushroom = "default",
        cactus = "default",
        banana = "default",
        lichen = "default",
  
        -- ANIMALS
        rabbits = "default",
        moles = "default",
        butterfly = "default",
        birds = "default",
        buzzard = "default",
        catcoon = "default",
        perd = "default",
        pigs = "default",
        lightninggoat = "default",
        beefalo = "default",
        beefaloheat = "default",
        hunt = "default",
        alternatehunt = "default",
        penguins = "default",
        cave_ponds = "default",
        ponds = "default",
        bees = "default",
        angrybees = "default",
        tallbirds = "default",
        slurper = "default",
        bunnymen = "default",
        slurtles = "default",
        rocky = "default",
        monkey = "default",
  
        -- MONSTERS
        spiders = "default",
        cave_spiders = "default",
        hounds = "default",
        houndmound = "default",
        merm = "default",
        tentacles = "default",
        chess = "default",
        lureplants = "default",
        walrus = "default",
        liefs = "default",
        deciduousmonster = "default",
        krampus = "default",
        bearger = "default",
        deerclops = "default",
        goosemoose = "default",
        dragonfly = "default",
        bats = "default",
        fissure = "default",
        worms = "default",
    },
}
```

**配置mod文件**

同样先在客户端上下载好服务端需要的mod文件，并且配置完成。可以在`我的文档/klei/一串数字/Cluster_x`下找到mod配置文件`modoverrides.lua`

*示例配置参数*

```lua
return {
  ["workshop-1079538195"]={
    configuration_options={
      beebox=true,
      birdcage=true,
      cartographydesk=true,
      cookpot=true,
      dragonflychest=true,
      dragonflyfurnace=true,
      endtable=true,
      firesuppressor=true,
      icebox=true,
      lightning_rod=true,
      meatrack=true,
      modsupport=false,
      moondial=true,
      mushroom_farm=true,
      mushroom_light=true,
      nightlight=true,
      perdshrine=true,
      pottedfern=true,
      rainometer=true,
      researchlab=true,
      researchlab2=true,
      researchlab3=true,
      researchlab4=true,
      resurrectionstatue=true,
      saltlick=true,
      scarecrow=true,
      sculptingtable=true,
      succulent_potted=true,
      townportal=true,
      treasurechest=true,
      wardrobe=true,
      winterometer=true 
    },
    enabled=true 
  },
  ["workshop-1903300952"]={
    configuration_options={ largebackpack=12, largericebox=20, largertreasurechest=12, recipe_complexity=0 },
    enabled=true 
  },
  ["workshop-374550642"]={ configuration_options={ MAXSTACKSIZE=250 }, enabled=true },
  ["workshop-375859599"]={
    configuration_options={
      divider=5,
      random_health_value=0,
      random_range=0,
      send_unknwon_prefabs=false,
      show_type=0,
      unknwon_prefabs=1,
      use_blacklist=true 
    },
    enabled=true 
  },
  ["workshop-378160973"]={
    configuration_options={
      ENABLEPINGS=true,
      FIREOPTIONS=2,
      OVERRIDEMODE=false,
      SHAREMINIMAPPROGRESS=true,
      SHOWFIREICONS=true,
      SHOWPLAYERICONS=true,
      SHOWPLAYERSOPTIONS=2 
    },
    enabled=true 
  },
  ["workshop-462372013"]={ configuration_options={  }, enabled=true },
  ["workshop-462434129"]={
    configuration_options={
      MOD_RESTART_ALLOW_KILL=true,
      MOD_RESTART_ALLOW_RESTART=true,
      MOD_RESTART_ALLOW_RESURRECT=true,
      MOD_RESTART_CD_BONUS=0,
      MOD_RESTART_CD_KILL=3,
      MOD_RESTART_CD_MAX=0,
      MOD_RESTART_CD_RESTART=5,
      MOD_RESTART_CD_RESURRECT=7,
      MOD_RESTART_FORCE_DROP_MODE=0,
      MOD_RESTART_IGNORING_ADMIN=true,
      MOD_RESTART_MAP_SAVE=2,
      MOD_RESTART_RESURRECT_HEALTH=0,
      MOD_RESTART_TRIGGER_MODE=1,
      MOD_RESTART_WELCOME_TIPS=true,
      MOD_RESTART_WELCOME_TIPS_TIME=6 
    },
    enabled=true 
  },
  ["workshop-791838548"]={
    configuration_options={
      Building_SD=33,
      Dispenser_Difficulty="default",
      EHat_Difficulty="default",
      Scrap_Difficulty="default",
      Sentry_Damage=9.5,
      Sentry_Difficulty="default",
      Sentry_Health=200,
      Sentry_ROF=1,
      Sentry_Range=13,
      Teleporter_Difficulty="default",
      buildinglimit="n",
      dispenseramount=4,
      dispenserhealingrate=3,
      dispenserlimit="n",
      disprange=2,
      ehardhatdura=295,
      engiedmgdebuff=0.8,
      engiesciencebonus=1,
      hardhatabsorb=0.7,
      rocketsplashdmg="y",
      sentry_ff="yesff",
      sentryamount=4,
      sentryh_regen="n",
      sentrylimit="n",
      teleporteramount=4,
      teleporterlimit="n",
      tf2Wrench_Difficulty="default",
      tf2wrenchdmg=17,
      tf2wrenchuses=150 
    },
    enabled=true 
  } 
}
```

**上传mod**

mod文件需要上传至服务器上，把电脑的mods目录压缩后上传至服务器的mods目录

安装目录设置在`/home/dst_server/mods` 

### 启动

为了保持后台运行饥荒，先安装`screen`

```bash
yum install -y screen
```

然后启动饥荒

```bash
screen -S dst
cd /home/dst_server/bin
./master.sh
```

运行成功后，`ctrl+a+d`退出screen终端，保持饥荒后台运行

在下次登录时，恢复之前的screen终端

```bash
screen -r dst
```



### 感谢阅读