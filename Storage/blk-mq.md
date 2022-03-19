# BLK-MQ 设计模型

# ==Linux-5.4.43==

### 关键结构体

+ ***block/blk_mq_tags.h***

```c
/*
 * Tag address space map.
 */
struct blk_mq_tags {
	unsigned int nr_tags;
	unsigned int nr_reserved_tags;

	atomic_t active_queues;

	struct sbitmap_queue bitmap_tags;
	struct sbitmap_queue breserved_tags;

	struct request **rqs;
	struct request **static_rqs;
	struct list_head page_list;
};

```

+ ***block/blk_mq.h***

```c
struct blk_mq_tag_set {
/*
* map[] holds ctx -> hctx mappings, one map exists for each type
* that the driver wishes to support. There are no restrictions
* on maps being of the same size, and it's perfectly legal to
* share maps between types.
*/
struct blk_mq_queue_map	map[HCTX_MAX_TYPES];
unsigned int		nr_maps;	/* nr entries in map[] */
const struct blk_mq_ops	*ops;
unsigned int		nr_hw_queues;	/* nr hw queues across maps */
unsigned int		queue_depth;	/* max hw supported */
unsigned int		reserved_tags;
unsigned int		cmd_size;	/* per-request extra data */
int			numa_node;
unsigned int		timeout;
unsigned int		flags;		/* BLK_MQ_F_* */
void			*driver_data;

struct blk_mq_tags	**tags;

struct mutex		tag_list_lock;
struct list_head	tag_list;
};
```

### 注册pcie设备 获取设备能力信息。SQ、CQ队列  初始化


+ ***drivers/nvme/host/pci.c***

> static int nvme_probe(struct pci_dev \***pdev**, const struct pci_device_id \***id**) 
>
> >  static void nvme_reset_work(struct work_struct **work*) 
> >
> >  > static int nvme_pci_enable(struct nvme_dev **dev*)
> >  >
> >  > ```c
> >  > dev->ctrl.cap = lo_hi_readq(dev->bar + NVME_REG_CAP);
> >  > dev->q_depth = min_t(int, NVME_CAP_MQES(dev->ctrl.cap) + 1,
> >  >                      io_queue_depth);//dev->q_depth 为 设备能力（队列深度） 和 1024 中较小值
> >  > ```
> >  >
> >  > static int nvme_setup_io_queues(struct nvme_dev **dev*)
> >  >
> >  > > static int nvme_create_io_queues(struct nvme_dev **dev*)
> >  > >
> >  > > > int nvme_set_queue_count(struct nvme_ctrl *\*ctrl*, int **count*)
> >  > > >
> >  > > > ```c
> >  > > > *count = min(*count, nr_io_queues);//nvme io queue 队列的数量为设备能力和cpu数量的较小值
> >  > > > ```
> >  > > >
> >  > > > static int nvme_create_queue(struct nvme_queue **nvmeq*, int *qid*, bool *polled*)
> >  > > >
> >  > > > > static int adapter_alloc_cq(struct nvme_dev **dev*, u16 *qid*,
> >  > > > >
> >  > > > > ​    struct nvme_queue **nvmeq*, s16 *vector*)
> >  > > > >
> >  > > > > static int adapter_alloc_sq(struct nvme_dev **dev*, u16 *qid*,
> >  > > > >
> >  > > > > ​            struct nvme_queue **nvmeq*)
> >  > > > >
> >  > > > > ```c
> >  > > > > c.create_sq.qsize = cpu_to_le16(nvmeq->q_depth - 1);//sq队列深度为31？
> >  > > > > ```
> >  > > > >
> >  > > > > static void nvme_init_queue(struct nvme_queue **nvmeq*, u16 *qid*)
> >  >
> >  > static void nvme_dev_add(struct nvme_dev **dev*) 
> >  >
> >  > ```c
> >  > dev->tagset.ops = &nvme_mq_ops;
> >  > dev->tagset.nr_hw_queues = dev->online_queues - 1;//硬件队列数量为 io queue 数量减一
> >  > dev->tagset.nr_maps = 2; /* default + read */
> >  > if (dev->io_queues[HCTX_TYPE_POLL])
> >  >     dev->tagset.nr_maps++;
> >  > dev->tagset.timeout = NVME_IO_TIMEOUT;
> >  > dev->tagset.numa_node = dev_to_node(dev->dev);
> >  > dev->tagset.queue_depth =
> >  >     min_t(int, dev->q_depth, BLK_MQ_MAX_DEPTH) - 1;//硬件队列深度取 设备能力和10240 较小值减一
> >  > dev->tagset.cmd_size = sizeof(struct nvme_iod);
> >  > dev->tagset.flags = BLK_MQ_F_SHOULD_MERGE;
> >  > dev->tagset.driver_data = dev;
> >  > ```

### 硬件队列tag和request 以及映射map 初始化


+ ***block/blk-mq.c***

> int blk_mq_alloc_tag_set(struct blk_mq_tag_set **set*) ////Alloc a tag set to be associated with one or more request queues.
>
> > static int blk_mq_update_queue_map(struct blk_mq_tag_set **set*) //建立CPU和硬件队列的映射
> >
> > static int blk_mq_alloc_rq_maps(struct blk_mq_tag_set **set*) 
> >
> > > static int __blk_mq_alloc_rq_maps(struct blk_mq_tag_set **set*)
> > >
> > > >  static bool __blk_mq_alloc_rq_map(struct blk_mq_tag_set **set*, int *hctx_idx*)
> > > >
> > > > > struct blk_mq_tags *blk_mq_alloc_rq_map(struct blk_mq_tag_set **set*, unsigned int *hctx_idx*, unsigned int *nr_tags*, unsigned int *reserved_tags*)//分配返回硬件队列专属的 blk_mq_tags
> > > > >
> > > > > int blk_mq_alloc_rqs(struct blk_mq_tag_set \*set, struct blk_mq_tags \*tags, unsigned int *hctx_idx*, unsigned int *depth*)//为hctx_idx编号的硬件队列，分配请求

+ ***drivers/nvme/host/core.c***

> static int nvme_alloc_ns(struct nvme_ctrl **ctrl*, unsigned *nsid*)//[块设备创建、基本信息填充和块设备注册到内核等工作](https://blog.csdn.net/yiyeguzhou100/article/details/102596393)
>
> > void device_add_disk(struct device \*parent*, struct gendisk \*disk*, const struct attribute_group ***groups*)

### 软件队列初始化并与硬件队列建立映射关系

+ ***block/blk-mq.c***

> blk_mq_init_queue函数整体来说，是创建request_queue运行队列并初始化其成员，分配每个CPU专属的软件队列，分配硬件队列，对二者做初始化，并建立软件队列和硬件队列联系。
>
> struct request_queue *blk_mq_init_queue(struct blk_mq_tag_set **set*)
>
> > struct request_queue *blk_alloc_queue_node(gfp_t *gfp_mask*, int *node_id*) //***block/block-core.c***
> >
> > struct request_queue *blk_mq_init_allocated_queue(struct blk_mq_tag_set **set*, struct request_queue **q*, bool *elevator_init*)
> >
> > > static void blk_mq_realloc_hw_ctxs(struct blk_mq_tag_set \*set*,struct request_queue *\*q*)//分配每一个硬件队列具体的数据结构blk_mq_hw_ctx
> >
> > > void blk_queue_make_request(struct request_queue \*q, make_request_fn **mfn*)//block/blk-settings.c==*//设置rq的make_request_fn 为blk_mq_make_request*==
> > >
> > > ```c
> > > q->nr_requests = BLKDEV_MAX_RQ;//默认软件队列深度为128
> > > ```
> > >
> > > static void blk_mq_add_queue_tag_set(struct blk_mq_tag_set \*set*,  struct request_queue \*q*)*//共享tag设置*
> > >
> > > static void blk_mq_map_swqueue(struct request_queue *q)// Map software to hardware queues.//完成硬件队列与软件队列的映射

### 请求产生并提交到nvme设备驱动

+ ***block/blk-mq.c***

> static blk_qc_t blk_mq_make_request(struct request_queue *\*q*, struct bio **bio*)
>
> > blk_attempt_plug_merge(*q*, *bio*, nr_segs, &same_queue_rq))
> >
> > static struct request *blk_mq_get_request(struct request_queue **q*,
> >
> > ​           struct bio **bio*,
> >
> > ​           struct blk_mq_alloc_data **data*)
> >
> > > *data*->ctx = blk_mq_get_ctx(*q*);
> > >
> > > *data*->hctx = blk_mq_map_queue(*q*, *data*->cmd_flags, *data*->ctx);
> > >
> > > unsigned int blk_mq_get_tag(struct blk_mq_alloc_data *\*data*)//***block/blk-mq-tag.c***
> > >
> > > static struct request *blk_mq_rq_ctx_init(struct blk_mq_alloc_data *\*data*,
> > >
> > > ​    unsigned int *tag*, unsigned int *op*, u64 *alloc_time_ns*) //*对新分配的req进行初始化，赋值软件队列、req起始时间等*
> >
> > static void blk_mq_bio_to_request(struct request *\*rq*, struct bio **bio*,
> >
> > ​    unsigned int *nr_segs*)
> >
> > void blk_mq_sched_insert_request(struct request **rq*, bool *at_head*,
> >
> > ​         bool *run_queue*, bool *async*)//***block/blk-mq-sched.c***
> >
> > > ==blk_mq_try_issue_directly()类的req direct 派发是针对单个req的，blk_mq_run_hw_queue()是派发类软件队列ctx->rq_list、硬件hctx->dispatch链表、IO调度算法队列上的req的，这是二者最大的区别==
> > >
> > > bool blk_mq_run_hw_queue(struct blk_mq_hw_ctx **hctx*, bool *async*)
> > >
> > > > bool blk_mq_run_hw_queue(struct blk_mq_hw_ctx **hctx*, bool *async*)
> > > >
> > > > > static void __blk_mq_delay_run_hw_queue(struct blk_mq_hw_ctx **hctx*, bool *async*,
> > > > >
> > > > > ​          unsigned long *msecs*)
> > > > >
> > > > > ==blk_mq_try_issue_list_directly、__blk_mq_try_issue_directly这种direct 模式，是直接把req派发给磁盘设备驱动。剩余的就是各种队列的req派发：软件队列ctx->rq_list链表、硬件队列hctx->dispatch链表、IO调度算法队列的相关链表、req plug模式的plug->mq_list。==
> > > > >
> > > > > > static void __blk_mq_run_hw_queue(struct blk_mq_hw_ctx *\*hctx*)  ***block/blk-mq-sched.c***
> > > > > >
> > > > > > > void blk_mq_sched_dispatch_requests(struct blk_mq_hw_ctx **hctx*)//
> > > > > > >
> > > > > > > 1 执行blk_mq_dispatch_rq_list()派发硬件队列hctx->dispatch链表上的req
> > > > > > > 2 执行blk_mq_do_dispatch_sched()派发调度器队列上的req。
> > > > > > > 3 执行blk_mq_do_dispatch_ctx ()函数派发软件队列ctx->rq_list链表上的req。
> > > > > > >
> > > > > > > > static void blk_mq_do_dispatch_sched(struct blk_mq_hw_ctx **hctx*)
> > > > > > > >
> > > > > > > > static void blk_mq_do_dispatch_ctx(struct blk_mq_hw_ctx **hctx*)
> > > > > > > >
> > > > > > > > >  bool blk_mq_dispatch_rq_list(struct request_queue *\*q*, struct list_head **list*,
> > > > > > > >
> > > > > > > > ​         bool *got_budget*)
> > > > > > >
> > > > > > > void blk_flush_plug_list(struct blk_plug *\*plug*, bool *from_schedule*) ***block/blk-mq-sched.c***
> >
> > static void blk_mq_try_issue_directly(struct blk_mq_hw_ctx *\*hctx*,struct request *\*rq*, blk_qc_t **cookie*)
> >
> > > static blk_status_t __blk_mq_try_issue_directly(struct blk_mq_hw_ctx **hctx*,
> > >
> > > ​            struct request **rq*,
> > >
> > > ​            blk_qc_t **cookie*,
> > >
> > > ​            bool *bypass_insert*, bool *last*)
> > >
> > > > bool blk_mq_get_driver_tag(struct request **rq*)
> > > >
> > > > static blk_status_t __blk_mq_issue_directly(struct blk_mq_hw_ctx **hctx*,
> > > >
> > > > ​            struct request **rq*,
> > > >
> > > > ​            blk_qc_t **cookie*, bool *last*)
> > > >
> > > > > ret = q->mq_ops->queue_rq(*hctx*, &bd);

### nvme 驱动处理

+ ***driver/nvme/host/pci.c***

```c
static const struct blk_mq_ops nvme_mq_admin_ops = {
    .queue_rq	= nvme_queue_rq,
    .complete	= nvme_pci_complete_rq,
    .init_hctx	= nvme_admin_init_hctx,
    .init_request	= nvme_init_request,
    .timeout	= nvme_timeout,
};
static const struct blk_mq_ops nvme_mq_ops = {
    .queue_rq	= nvme_queue_rq,
    .complete	= nvme_pci_complete_rq,
    .commit_rqs	= nvme_commit_rqs,
    .init_hctx	= nvme_init_hctx,
    .init_request	= nvme_init_request,
    .map_queues	= nvme_pci_map_queues,
    .timeout	= nvme_timeout,
    .poll		= nvme_poll,
};
```

+ ***driver/nvme/host/core.c***

> static blk_status_t nvme_queue_rq(struct blk_mq_hw_ctx **hctx*,
>
> ​       const struct blk_mq_queue_data **bd*)
>
> > void blk_mq_start_request(struct request *\*rq*) //***block/blk-mq.c***

### 请求处理完成

> static void nvme_pci_complete_rq(struct request **req*)
>
> > void nvme_complete_rq(struct request *\*req*) 
> >
> > > void blk_mq_end_request(struct request *\*rq*, blk_status_t *error*) //***block/blk-mq.c***
> > >
> > > > inline void __blk_mq_end_request(struct request **rq*, blk_status_t *error*)
> > > >
> > > > > static inline void blk_mq_sched_completed_request(struct request **rq*, u64 *now*)
> > > > >
> > > > > > static void __blk_mq_complete_request(struct request **rq*)

![img](https://img-blog.csdnimg.cn/20210511101820498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phc29uYWN0aW9ucw==,size_16,color_FFFFFF,t_70)







