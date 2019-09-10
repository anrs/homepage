Title: Orphan or Not
Date: 2012-03-03 12:22
Category: blog
Summary: 

进程死后，他的子进程会变成孤儿，但某些场景下我们希望父死子殉，为此 hongqn 专门搞了 [CaoE](https://github.com/douban/CaoE) 来自动支持这件事。(CaoE 这个名字来自[这里](https://zh.wikipedia.org/wiki/曹娥))

但实际上，在进程死时，会给他的子进程们重新寻父，寻父无果的情况下才会让 init 成为这些子进程的父进程。寻父函数是 kernel/exit.c:find_new_reaper()，这个函数顶上有段注释是：

```c
/*
 * When we die, we re-parent all our children.
 * Try to give them to another thread in our thread
 * group, and if no such member exists, give it to
 * the child reaper process (ie “init”) in our pid
 * space.
 */
static struct task_struct *find_new_reaper(struct task_struct *father)
```

注释写得很清楚，并不是所有情况下该函数都会返回 init 进程的，下面是寻父过程的代码段:

```c
thread = father;
while_each_thread(father, thread) {
    if (thread->flags & PF_EXITING)
        continue;
    if (unlikely(pid_ns->child_reaper == father))
        pid_ns->child_reaper = thread;
    return thread;
}
```

find_new_reaper() 一开始就遍历待死进程的线程组，试图找到其他还存活着的进程，如果找到了，就直接返回。遍历过程是通过 while_each_thread 宏和 next_thread() 函数实现的：

```c
#define while_each_thread(g, t) while ((t = next_thread(t)) != g)

static inline struct task_struct *next_thread(const struct task_struct *p) {
    return list_entry_rcu(p->thread_group.next, struct task_struct, thread_group);
}
```

可以看到只有 find_new_reaper() 在待死进程的线程组中没有找到其他活着的进程，才会返回 init 进程。

```c
/*
 * We can not clear ->child_reaper or leave it alone.
 * There may by stealth EXIT_DEAD tasks on ->children,
 * forget_original_parent() must move them somewhere.
 */
pid_ns->child_reaper = init_pid_ns.child_reaper;
…
return pid_ns->child_reaper;
```

顺便说下我在查进程终结的代码时，发现了 LKD 3rd 中文版这本神作，今天还跟员外在 twitter 上讨论该书的奇葩翻译来着。我目前看了大概过一半，基本上都是边看英文电子版边翻中文版撑过来的。比如对 mutex_trylock() 返回值的描述，原文是”returns one if successful and the lock is acquired and zero otherwise”，译文变成了“ 如果成功则返回1；否则锁被获取，返回值是0”（标点我都没改）。

我觉得原文是有点歧义，如果能写成”returns one if successful and the lock is acquired, and zero otherwise”就清楚明白了，可是，就算有歧义那函数名中的 trylock 总明白无误了吧，有 try 失败了还能得到锁的函数吗？好吧，就算函数名不足以说明问题，原文又有点歧义，那我直接翻代码吧，找到了 mutex_trylock() 的函数声明（在 <linux/mutex.h> 文件中）：

```c
/*
 * NOTE: mutex_trylock() follows the spin_trylock() convention,
 * not the down_trylock() convention!
 *
 * Returns 1 if the mutex has been acquired successfully, and 0 on contention.
*/
extern int mutex_trylock(struct mutex *lock);
```

看见没有，代码作者 Ingo Molnar 的注释可写得清楚明白准确无误没歧义的，还有个闪瞎黄金狗眼的硕大逗号。所以可以有理由怀疑，中文版某位译者的英文造诣绝对高过计算机造诣的，就这水平去译散文译哲学专著那多好多造福人类啊。