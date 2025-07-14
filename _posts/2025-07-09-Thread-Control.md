---
title: 'Thread Control'
date: 2025-07-09 12:18:00 +0900
categories: ['OS', 'Concurrency']
tags: [os, concurrency, thread, lock, semaphore]
---

24-2 김진수 교수님의 운영체제 수업에서 배운 내용을 정리한 글입니다.

어느정도 복잡한 코드를 작성하다보면, 필수적으로 따라오는것이 `thread` 사용과 관련한 문제들이다. thread를 사용하는 이유는 여러가지가 있는데,  
1. I/O Concurrency를 위해: Network, Disk 등의 I/O wait time 동안 다른 코드를 돌리고
2. Multicore 사용: 활용할 수 있는 cpu 자원을 최대한 활용하고
3. 편의성: 백그라운드에서 주기적으로 실행하는 작업을 수행하는 등   


등등 여러가지 이유로 코드에서 thread를 사용하게 된다.  

장점이 매우 많고 성능 향상을 위해 반드시 thread를 활용해야하지만 그에 따라 반드시 따라오는 문제가 thread 간에 Resource를 공유하는 문제이다.  
C 언어 기준으로 thread 간에 공유하는 Resource는  
- Local Variable: thread Stack에 있기에 공유 x
- Global Variable: data segment에 있기에 공유됨
- Dynamic Objects: thread간에 공유됨  


이외에도 shared memory를 생성하는 등 thread간에 공유되는 resource가 필연적으로 존재하게 된다.  

```c
int withdraw(account, amount)
{
	balance = get_balance(account);
	balance = balance - amount;
	put_balance(account, balance);
	return balance;
}
```
이런 코드에서 두 사용자 A, B가 동시에 하나의 account에 대해서 withdraw를 할 때, 은행 서버에서 아래와 같은 순서로 코드가 스케쥴링된다면,  
```
A.  
	balance = get_balance(account);  
	balance = balance - amount;  
B.  
	balance = get_balance(account);
	balance = balance - amount;
	put_balance(account, balance);  
A.  
	put_balance(account, balance);
```
두 사용자가 인출한 금액이 account의 balance보다 크더라도 아무 문제가 생기지 않는다. 이러한 현상을 `race condition`이라고 한다. 이것이 문제가 되는 이유는 시스템의 의도대로 코드가 작동하지 않기 때문도 있지만, 프로그램의 결과가 `non-deterministic` 해지기 때문이다. 위 코드가 다른 방식(예를 들어 A코드 모두 완료 후 B코드 수행)으로 스케쥴링 되었다면 프로그램은 원래의 의도대로 정상적으로 작동했을 것이다.  

이런 문제를 해결하기 위해 공유하는 resource에 대해 접근을 관리하는 것을 `Synchronization`이라고 한다. Synchronization을 하는 것은 어렵지 않은데, 우선 공유 Resource를 접근하는 코드 부분을 찾고, 이 부분을 한번에 최대 하나의 thread만 수행할 수 있도록 제한하면 된다. 이러한 부분을 `critical section`이라고 부르고, 이에 대해 제한하는 것을 `mutual exclusion`이라고 한다. 만약 어떤 thread가 critical section을 수행하고 있다면, 다른 thread는 그것이 끝날때까지 기다리게 만들면 된다.  

### 1. Locks(Spin Lock)
그러면 mutual exclusion을 어떻게 보장을 할까?  
일반적으로 다음과 같은 operation을 갖는 `Lock`이라는 primitive를 통해 보장할 수 있다.  
- acquire(): lock이 free일때 까지 기다리고, lock을 잡는다.
- release(): lock을 해제한다.  
```c
int withdraw(account, amount)
{
	acquire(lock);
	balance = get_balance(account);
	balance = balance - amount;
	put_balance(account, balance);
	release(lock);
	return balance;
}
```
기존 코드를 위와 같이 바꾼다면, 공유하는 resouce(balance)에 대해 critical section을 형성하고, mutual exclusion이 보장된다.  

lock에는 크게 두 방식이 있는데, 
1. `Spin Lock`: lock이 free일때까지 기다리는 thread가 busy waiting(cpu 자원을 소모하면서 대기) 하는 방식 
2. `Sleep Lock`: lock이 free일때까지 기다리는 thread가 sleep/block 하(cpu 자원을 소모하지 않으면서 대기)는 방식  


이렇게 보면 Sleeplock이 무조건 좋아보이지만, Sleeplock을 구현하기 위해서는 부분적으로 Spinlock을 필수로 사용해야한다. 또한 Spinlock의 구현이 비교적 더 직관적이므로, 우선 Sleeplock에 대해 설명하고자 한다.  

모던 아키텍쳐에서는 `Atomic Instruction`을 지원한다. 이는 Read-Modify-Write operation의 코드 묶음을 하드웨어적으로 `atomic`하게 수행됨을 보장해주는 기능이다. 대표적으로 `Test-And-Set`이 있는데,  
```c
int TestAndSet(int *v, int new) {
	int old = *v;
	*v = new;
	return old;
}
```
이 함수의 작동은 하드웨어에 의해 atomicity가 보장된다.  
```c
struct lock { int held = 0; }

void acquire(struct lock *l) {
	while (TestAndSet(&l->held, 1));
}

void release(struct lock *l) {
	l->held = 0;
}
```
이를 활용하면 위와 같이 간단한 spinlock을 구현할 수 있다. 이것이 spinlock인 이유는 `TestAndSet(&l->held, 1))`의 결과가 0일때 까지 thread가 스케쥴링 될때마다 할당된 모든 시간 동안 while문을 spin하기 때문이다. 따라서 spinlock을 통해 mutual exclusion을 보장할 수 있지만 기다리는 thread 또한 cpu 자원을 사용하기 있기에 그렇게 효율적이지는 못하다. 또한 이런 하드웨어의 Atomic Instruction은 다른 코드에 비해 상대적으로 부담이 크다. 따라서 아래와 같이 약간의 최적화를 진행할 수는 있다.  
```c
void acquire(struct lock *l) {
	while (l->held || TestAndSet(&l->held, 1));
}
```
조금 더 많이 최적화를 하고 싶다면, MCS Lock이라는 것이 존재한다.
[MCS 설명](https://courses.cs.washington.edu/courses/cse451/15sp/lectures/mcsRCULockExample.pdf)

그리고 이처럼 하드웨어의 지원을 이용하지 않고 오로지 소프트웨어로만 spinlock을 만드는 방법도 있다. (Peterson’s Algorithm)
```c
int turn;
int interested[2];
void acquire(int process) {
	int other = 1 - process;
	interested[process] = TRUE;
	turn = other;
	while (interested[other] && turn == other);
}
void release(int process) {
	interested[process] = FALSE;
}
```


### 2. Semaphore
지금까지 살펴본 Spin Lock은 mutual exclusion을 쉽게 보장하지만, lock을 잡기 위해 기다리는 thread가 cpu 자원을 동일하게 소모하기 때문에 Spin Lock을 직접적으로 사용해 mutual exclusion을 보장하는 것은 좋은 선택지가 아니다.  
이를 해결하기 위해서 가장 흔하게 사용하는 방법 중 하나는 Semaphore이다.  
Semaphore는 Spin Lock과 정수 value를 가진 자료구조로, 2가지 operation을 지원한다.
- Wait(): decrement the value, and wait until the value is >= 0
- Signal(): Increment the value, then wake up a single waiter  


xv6 코드를 기준으로, 구현하는 방식을 살펴보려고 한다.  

```c
void sleep(void *chan, struct spinlock *lk) {
	struct proc *p = myproc();
	if (lk != &p->lock) {
		acquire(&p->lock);
		release(lk);
	}
	p->chan = chan;
	p->state = SLEEPING;
	sched();
	p->chan = 0;
	if (lk != &p->lock) {
		release(&p->lock);
		acquire(lk);
	}
}

void wakeup(void *chan) {
	struct proc *p;
	for (p = proc; p < &proc[NPROC]; p++) {
		acquire(&p->lock);
		if (p->state == SLEEPING &&
			p->chan == chan) {
			p->state = RUNNABLE;
		}
		release(&p->lock);
	}
}
```
xv6 코드에는 sleep와 wakeup 함수가 있다. 이 두 함수를 사용하면 xv6에서 semaphore를 구현할 수 있다.  

```c
struct semaphore {
  int count;
  struct spinlock lock;
};

void semaphore_init(struct semaphore *s, int init) {
  initlock(&s->lock, "semaphore");
  s->count = init;
}

void semaphore_wait(struct semaphore *s) {
  acquire(&s->lock);
  while (s->count == 0) {
    sleep((void*)s, &s->lock);
  }
  s->count--;
  release(&s->lock);
}

void semaphore_signal(struct semaphore *s) {
  acquire(&s->lock);
  s->count++;
  wakeup((void*)s);
  release(&s->lock);
}

```
이렇게 구현하면, count가 0 이하일 때 wait를 호출했을 경우, 해당 thread는 대기하는 대부분의 시간을 `SLEEPING`상태로 보낸다. 이 상태에서는 thread가 스케쥴링 되지 않아서 cpu 자원을 소모하지 않는다. 그리고 코드에서 가장 주의 깊게 보아야 하는 부분은 아래 부분이다. 
```c
  while (s->count == 0) {
    sleep((void*)s, &s->lock);
  }
```
우선 sleep를 while문으로 감싸는 이유는, 해당 thread가 일어났을 때 여전히 count가 0 이하일 가능성이 있기 때문이다. thread 여러개가 잠들고 있는 경우, wakeup은 해당 semaphore를 기다리고 있는 모든 thread를 깨운다. 그러면 가장 먼저 일어난 thread가 count를 1 줄이고 다른 thread는 조건이 충족되지 않기에 다시 sleep 상태가 되어야만 한다.  
그리고 sleep함수는 spinlock을 파라미터로 갖는다. sleep하기 전에 lock을 hold하고 있다가 `SLEEPING`상태가 되기 이전에 lock을 free해주어야 한다. 이렇게 하지 않고 lock을 hold한채 잠에 들면 deadlock이 일어나기에 필수적인 과정이다.  

번외로 동일하게 spin lock, sleep, wake를 사용하면 xv6에서 sleep lock을 만들 수 있다. 
```c
struct sleeplock {
	uint locked;
	struct spinlock lk;
	char *name;
	int pid;
};

void initsleeplock(struct sleeplock *lk, char *name) {
	initlock(&lk->lk, "sleep lock");
	lk->name = name;
	lk->locked = 0;
	lk->pid = 0;
}

void acquiresleep(struct sleeplock *lk) {
	acquire(&lk->lk);
	while (lk->locked) {
		sleep(lk, &lk->lk);
	}
	lk->locked = 1;
	lk->pid = myproc()->pid;
	release(&lk->lk);
}

void releasesleep(struct sleeplock *lk) {
	acquire(&lk->lk);
	lk->locked = 0;
	lk->pid = 0;
	wakeup(lk);
	release(&lk->lk);
}
```

Semaphore를 활용하는 가장 대표적인 방법 중 하나는 Bounded Buffer Problem이다. 하나의 circular Buffer가 있고, 이를 활용하는 여러개의 Producer와 여러개의 Consumer가 있을 때, race condition을 해결하는 문제이다.  
```c
Semaphore
	mutex = 1;
	empty = N;
	full = 0;

struct item buffer[N];
int in, out;

void produce(data)
{
	wait(&empty);
	wait(&mutex);
	buffer[in] = data;
	in = (in+1) % N;
	signal(&mutex);
	signal(&full);
}
void consume(&data)
{
	int in, out;
	wait(&full);
	wait(&mutex);
	*data = buffer[out];
	out = (out+1) % N;
	signal(&mutex);
	signal(&empty);
}
```

### 3. Condition Variable
마지막은 Condition Variable이다. Semaphore와 유사한 부분이 있지만, 기능과 쓰임새에서 다른 부분들이 있다.  
우선 Condition Variable(CV)는 어떤 event를 기다리게 하는 기능을 구현하는 데 주로 사용된다. CV는 하나의 queue라고 생각하면 좋은데, 특정 event가 일어나기 전까지 기다리고 있는 thread들을 queue에 모아두었다가, event가 일어나면 queue의 모든 thread를 깨우는 것이다.  
CV는 mutex(Sleeping Lock)와 함께 사용되고, 아래의 operation을 지원한다.
- wait(cond_t \*cv, mutex_t \*mutex): caller는 sleep하고 mutex를 놓는다. wake하면 다시 mutex를 잡는다.
- signal(cond_t \*cv): sleep하고 있는 하나의 thread를 깨운다.
- boradcast(cond_t \*cv): sleep하고 있는 모든 thread를 깨운다.  


Semaphore와의 차이점은 value값이 없다는 점이다. **따라서 signal 또는 broadcast를 했을때 잠자고있는 thread가 없는 경우 아무런 일이 일어나지 않는다.** 반면의 Semaphore에서는 잠자고 있는 thread가 없더라도 value가 1 증가하기 때문에 추후의 동작에 영향을 준다.  

xv6에서는 Semaphore와 마찬가지로 sleep, wakeup, mutex(Sleep Lock)를 사용해서 CV를 구현할 수 있다.  
```c
mutex_t m = MUTEX_INITIALIZER;
cond_t c = COND_INITIALIZER;
int done = 0

void thread_exit() {
	mutex_lock(&m);
	done = 1;
	cond_signal(&c);
	mutex_unlock(&m);
}

void thread_join() {
	mutex_lock(&m);
	while (done == 0)
	cond_wait(&c, &m);
	mutex_unlock(&m);
}

void *child(void *arg) {
	thread_exit();
	return NULL;
}

int main(int argc, char *argv[]) {
	thread_t p;
	thread_create(&p, NULL, child, NULL);
	thread_join();
	return 0;
}
```
위의 예시는 thread를 활용하는 대표적인 예시인 thread create-join에 대한 것이다.  
