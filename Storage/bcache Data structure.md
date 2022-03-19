## bcache 代码解读

[BCache源码浅析之一基本使用与代码模块](https://blog.csdn.net/wanthelping/article/details/50448947)

## 结构体

### bio

```c
struct bio {
        struct bio          *bi_next;   /* request queue link */
        struct block_device *bi_bdev;
        unsigned int        bi_flags;   /* status, command, etc */
        int                 bi_error;
        unsigned long       bi_rw;      /* 末尾 bit 表示 READ/WRITE,
                                         * 起始 bit 表示优先级
                                         */
        struct bvec_iter    bi_iter;

        /* 当完成物理地址合并之后剩余的段的数量 */
        unsigned int        bi_phys_segments;

        /*
         * To keep track of the max segment size, we account for the
         * sizes of the first and last mergeable segments in this bio.
         */
        unsigned int        bi_seg_front_size;
        unsigned int        bi_seg_back_size;

        /* 关联 bio 的数量 */
        atomic_t            __bi_remaining;
        bio_end_io_t        *bi_end_io;
        void                *bi_private;
        unsigned short      bi_vcnt;    /* how many bio_vec's */

        /*
         * Everything starting with bi_max_vecs will be preserved by bio_reset()
         */
        unsigned short      bi_max_vecs;    /* max bvl_vecs we can hold */
        /* 当前 bio 的引用计数，当该数据为 0 时才可以 free */
        atomic_t            __bi_cnt;       /* pin count */
        struct bio_vec      *bi_io_vec;     /* the actual vec list */
        struct bio_set      *bi_pool;

        /*
         * We can inline a number of vecs at the end of the bio, to avoid
         * double allocations for a small number of bio_vecs. This member
         * MUST obviously be kept at the very end of the bio.
         * 表示跟在 bio 后面的数据集合
         */
        struct bio_vec      bi_inline_vecs[0];
};
```



### io

```c
struct io {
	/* Used to track sequential IO so it can be skipped */
	struct hlist_node	hash;
	struct list_head	lru;

	unsigned long		jiffies;
	unsigned int		sequential;
	sector_t		last;
};
```

### cached_dev
```c

struct cached_dev {
	struct list_head	list;
	struct bcache_device	disk;
	struct block_device	*bdev;

	struct cache_sb		sb;
	struct bio		sb_bio;
	struct bio_vec		sb_bv[1];
	struct closure		sb_write;
	struct semaphore	sb_write_mutex;

	/* Refcount on the cache set. Always nonzero when we're caching. */
	refcount_t		count;
	struct work_struct	detach;

	/*
	 * Device might not be running if it's dirty and the cache set hasn't
	 * showed up yet.
	 */
	atomic_t		running;
    ······
};
```

### bcache_device

```c
struct bcache_device {
	struct closure		cl;

	struct kobject		kobj;

	struct cache_set	*c;
	unsigned int		id;
#define BCACHEDEVNAME_SIZE	12
	char			name[BCACHEDEVNAME_SIZE];

	struct gendisk		*disk;

	unsigned long		flags;
#define BCACHE_DEV_CLOSING		0
#define BCACHE_DEV_DETACHING		1
#define BCACHE_DEV_UNLINK_DONE		2
#define BCACHE_DEV_WB_RUNNING		3
#define BCACHE_DEV_RATE_DW_RUNNING	4
	unsigned int		nr_stripes;
	unsigned int		stripe_size;
	atomic_t		*stripe_sectors_dirty;
	unsigned long		*full_dirty_stripes;

	struct bio_set		bio_split;

	unsigned int		data_csum:1;

	int (*cache_miss)(struct btree *b, struct search *s,
			  struct bio *bio, unsigned int sectors);
	int (*ioctl)(struct bcache_device *d, fmode_t mode,
		     unsigned int cmd, unsigned long arg);
};
```



### search

```c
/* Cache lookup */

struct search {
	/* Stack frame for bio_complete */
	struct closure		cl;

	struct bbio		bio;
	struct bio		*orig_bio;
	struct bio		*cache_miss;
	struct bcache_device	*d;

	unsigned int		insert_bio_sectors;
	unsigned int		recoverable:1;
	unsigned int		write:1;
	unsigned int		read_dirty_data:1;
	unsigned int		cache_missed:1;

	unsigned long		start_time;

	struct btree_op		op;
	struct data_insert_op	iop;
};
```
### cached_sb
```c
struct cache_sb {
	__u64			csum;
	__u64			offset;	/* sector where this sb was written */
	__u64			version;

	__u8			magic[16];

	__u8			uuid[16];
	union {
		__u8		set_uuid[16];
		__u64		set_magic;
	};
	__u8			label[SB_LABEL_SIZE];

	__u64			flags;
	__u64			seq;
	__u64			pad[8];

	union {
	struct {
		/* Cache devices */
		__u64		nbuckets;	/* device size */

		__u16		block_size;	/* sectors */
		__u16		bucket_size;	/* sectors */

		__u16		nr_in_set;
		__u16		nr_this_dev;
	};
	struct {
		/* Backing devices */
		__u64		data_offset;

		/*
		 * block_size from the cache device section is still used by
		 * backing devices, so don't add anything here until we fix
		 * things to not need it for backing devices anymore
		 */
	};
	};

	__u32			last_mount;	/* time overflow in y2106 */

	__u16			first_bucket;
	union {
		__u16		njournal_buckets;
		__u16		keys;
	};
	__u64			d[SB_JOURNAL_BUCKETS];	/* journal buckets */
};
```

## 函数

+ ***Drivers/md/bcache/request.c***

> static void cached_dev_write(struct cached_dev ** dc*, struct search **s*)
>
> static void cached_dev_read(struct cached_dev ** dc*, struct search **s*)
>
> > static void cache_lookup(struct closure **cl*)
> >
> > > static int cache_lookup_fn(struct btree_op ** op*, struct btree * *b*, struct bkey **k*)
> > >
> > > 

[块设备内核参数max_segments和max_sectors_kb解析](https://blog.csdn.net/wangww631/article/details/78798637)

[扇区和文件对应关系](https://blog.csdn.net/u010278923/article/details/79901155)

+ ***Driver/md/bcache/super.c***

> static ssize_t register_bcache(struct kobject *k*, struct kobj_attribute **attr*,
>
> ​          const char **buffer*, size_t *size*)
>
> > static int register_bdev(struct cache_sb *sb*, struct page *sb_page*, struct block_device *bdev*, struct cached_dev **dc*)
> >
> > >  static int cached_dev_init(struct cached_dev **dc*, unsigned int *block_size*)
> > >
> > >  > static int bcache_device_init(struct bcache_device **d*, unsigned int *block_size*,
> > >  >
> > >  > ​         sector_t *sectors*)
> > >  >
> > >  > void bch_cached_dev_request_init(struct cached_dev **dc*)
> > >  >
> > >  > void bch_cached_dev_writeback_init(struct cached_dev **dc*)
> > >
> > >  int bch_cached_dev_run(struct cached_dev **dc*)
> >
> > static int register_cache(struct cache_sb *sb*, struct page *sb_page*, struct block_device *bdev*, struct cache **ca*)
> >
> > > static const char *register_cache_set(struct cache **ca*)
> > >
> > > static int run_cache_set(struct cache_set **c*)

+ ***Driver/md/bcache/request.c***

> static blk_qc_t cached_dev_make_request(struct request_queue ** q* ,struct bio **bio*)
>
> > static void cached_dev_read(struct cached_dev * *dc*, struct search ** s*)
> >
> > > static void cache_lookup(struct closure **cl*)
> > >
> > > > int bch_btree_map_keys(struct btree_op * *op*, struct cache_set ** c*,
> > > >
> > > > > struct bkey * *from*, btree_map_keys_fn **fn*, int *flags*)  ==/btree.c==
> > > > >
> > > > > > \#define btree_root(*fn*, *c*, *op*, ...)  ==/btree.c==
> > > > >
> > > > > static int cache_lookup_fn(struct btree_op ** op*, struct btree ** b*, struct bkey **k*)
> > > >
> > > > 
> > >
> > > static void cached_dev_read_done_bh(struct closure **cl*)
