### FileCoin运维之恢复错误的扇区
> 多的就不说了，直接贴出miner日志

![](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog8-1.png)

**出问题的原因详细来说：其中一台存储机器的阵列卡出问题，那么存储机器的文件系统肯定也坏了，共享的nfs更不用说了，上miner机器** ```df -h```**一看，果然卡死了，网关机器也是一样的，** ```lotus-miner proving deadlines```**找出有错误的partition，然后再找出错误的扇区** ```lotus-miner proving check --only-bad 3```

![](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog8-3.png)

看到这里有两种错误，一种是第一张图上的错误，还有一种是找不到cache或者sealed文件。

![](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog8-2.png)

* 提示说```cant acquire read lock ```，这个状态下无法修改扇区状态，我谷歌之后也没有找到一个确切的说法，只能说找到一个折中的解决方法：重启网关机器，让miner调度网关机器转移扇区，成功了就解决了，失败了扇区状态会变 *FinalizeFailed* ，这个时候修改扇区的状态为Proving，``` lotus-miner sectors update-state --really-do-it=true <sector_id> Proving```，这时候再查找错误扇区，就会变成第二种错误。
* 既然说找不到cache或者sealed，那就先看看哪些找到了哪些没找到，``` lotus-miner storage find <sector_id>```
    1. 如果是cache不见了，那么需要去worker1机器上找对应应用的data目录下的cache目录
    2. 如果是sealed不见了，那么也是worker1机器上找对应应用的data目录下的sealed目录
    > 如果有的话，拷贝至时空证明的存储目录，如果找不到了，那么这个扇区可能就废了。

拷贝完扇区还有最后一步，就是手动声明扇区存储路径。这里我直接引用了[RockYang大佬的运维教程，有需要的可以自行查看](https://www.r9it.com/20200618/filecoin-env-and-operation.html)

---

以下场景可能需要手动声明扇区位置：
1. 部分 FinalizeFailed 的扇区。
2. FinalizeSector 成功了，但是 ***lotus-miner proving check*** 又提示 ***can not cache/sealed path***，导致时空证明过不了，掉算力。
此时我们可以手动完成 FinalizeSector 过程：
1. 首先找到扇区文件在哪个机器上（假设扇区 ID 为 100，矿工 ID 为 f01000）：
```
lotus-miner storage find 100
```
如果找不到的话，可以通过运维系统批量推送脚本到执行：
```
ls -ld /gamma/lotus-p1-worker/data/cache/s-t01000-100
ls -ld /gamma/lotus-p1-worker/data/sealed/s-t01000-100
```
2. 手动拷贝扇区到你的扇区存盘路径，假设为 /data01，扇区所在 Worker 机器 IP 为 192.168.1.100
```
scp -r root@192.168.1.100:/gamma/lotus-p1-worker/data/cache/s-t01000-100 /data01/cache
scp -r root@192.168.1.100:/gamma/lotus-p1-worker/data/sealed/s-t01000-100 /data01/sealed
```
3. 手动申明扇区，假设 /data01/sectorstore.json 文件对应的 Storage ID 为： e0d9481a-3d85-4464-8f0d-2af9a5c755d1
```
# 声明 cache 文件
lotus-miner storage declare-sector --really-do-it=true --type=cache --sector=100 --storage=e0d9481a-3d85-4464-8f0d-2af9a5c755d1
# 声明 sealed 文件
lotus-miner storage declare-sector --really-do-it=true --type=sealed --sector=100 --storage=e0d9481a-3d85-4464-8f0d-2af9a5c755d1
```
4. 同时，你需要删除错误的 storage 声明（如果有的话）。
```
lotus-miner storage drop-sector --really-do-it=true --type=cache --sector=100 --storage=<Storage-ID>
```