---
layout: post 
title: "Tracing mechanism of QEMU"
categories: Virtualization 
toc: true
comments: true
---

> tracing tools is very useful for debugging, profiling and studying source code. I tried to use QEMU's tracing tool to study the source code. under source code directory docs/tracing.txt provided lots of usefull information.

## How to use simple trace
Make sure you have libsdl-dev installed, e.g. libsdl1.2-dev, other wise, when you run your QEMU, you will have no "display", then you need use vnc connect to guest OS.
or you can use GTK. To speed up the building processing, I only build target for i386,x86_64,ppc and arm.

```bash
$ ./configure --enable-trace-backend=simple --target-list=i386-softmmu,\
	x86_64-softmmu,ppc-softmmu,arm-softmmu --enable-debug --extra-cflags=-g3

SDL support       yes (1.2.15)
GTK support       no
GTK GL support    no

$ make
```

After a while, you should can run QEMU:

```bash
$ sudo ./x86_64-softmmu/qemu-system-x86_64 -display sdl  # you will get a window here

$ sudo ./x86_64-softmmu/qemu-system-x86_64 -display gtk  # for my case gtk support is no.
qemu-system-x86_64: -display gtk: GTK support is disabled
```

Download a linux image, I recommand CirrOS. CirrOS is a minimal Linux distribution that was designed for use as a test image on clouds such as OpenStack Compute. You can download a CirrOS image in various formats from the CirrOS download page. [http://docs.openstack.org/image-guide/obtain-images.html](http://docs.openstack.org/image-guide/obtain-images.html){:target="_blank"}


```bash
$ echo load_file > /tmp/events
$ sudo ./x86_64-softmmu/qemu-system-x86_64 ./cirros-0.3.4-x86_64-disk.img -trace events=/tmp/events -monitor stdio
```
You will get a trace-Pid, Pid is the PID of current running QEMU, you can use simpletrace script to decode that file.

```bash
./scripts/simpletrace.py trace-events trace-11013

load_file 0.000 pid=11013 name=bios-256k.bin path=/xx/qemu-2.8.0-rc0/pc-bios/bios-256k.bin
load_file 281.451 pid=11013 name=bios-256k.bin path=/xx/qemu-2.8.0-rc0/pc-bios/bios-256k.bin
load_file 2112.729 pid=11013 name=kvmvapic.bin path=/xx/qemu-2.8.0-rc0/pc-bios/kvmvapic.bin
load_file 24813.877 pid=11013 name=vgabios-stdvga.bin path=/xx/qemu-2.8.0-rc0/pc-bios/vgabios-stdvga.bin
load_file 12734.305 pid=11013 name=efi-e1000.rom path=/xx/qemu-2.8.0-rc0/pc-bios/efi-e1000.rom
```

"-monitor stdio" provide a CLI interface, so you can interact with QEMU. e.g. 

```bash
# list status of all trace-events
(qemu) info trace-events 

runstate_set : state 0
load_file : state 1
vm_state_notify : state 0
balloon_event : state 0
cpu_out : state 0

# Disable load_file

(qemu) trace-event load_file off

# Check status of load_file again
(qemu) info trace-events load_f*
load_file : state 0
```

## Adding customized trace event
Adding a new trace event is very easy, open trace-events file under source code directory.
I'm tring to learn how TCG works, So I added a trace event called tcg_gen_code, it receive two argument,

```diff
diff --git a/trace-events b/trace-events
@@ -178,3 +178,5 @@ disable vcpu guest_user_syscall(uint64_t num, uint64_t arg1, uint64_t arg2, uint
 # Mode: user
 # Targets: TCG(all)
 disable vcpu guest_user_syscall_ret(uint64_t num, uint64_t ret) "num=0x%016"PRIx64" ret=0x%016"PRIx64
+
+tcg_gen_code(uint64_t cs, uint64_t pc) "cs=0x%016 pc=0x%016"
```

You should trigger the event by calling trace_XXX, XXX is the event name you added.
```diff
diff --git a/target-i386/translate.c b/target-i386/translate.c
@@ -8325,6 +8325,8 @@ void gen_intermediate_code(CPUX86State *env, TranslationBlock *tb)
     cs_base = tb->cs_base;
     flags = tb->flags;
 
+       trace_tcg_gen_code(cs_base, pc_start);
+
     dc->pe = (flags >> HF_PE_SHIFT) & 1;
     dc->code32 = (flags >> HF_CS32_SHIFT) & 1;
     dc->ss32 = (flags >> HF_SS32_SHIFT) & 1;
```

Now rebuild QEMU.

```bash
$ echo tcg_gen_code >> /tmp/events
$ sudo ./x86_64-softmmu/qemu-system-x86_64 ./cirros-0.3.4-x86_64-disk.img -trace events=/tmp/events -monitor stdio
```

Use simpletrace script to decode trace log, you will get the following output.
This is interesting that even both of Host and Guest System are X86_64, QEMU still using TCG to translate binary code.

```bash
$ ./scripts/simpletrace.py trace-events trace-11123

tcg_gen_code 22223.773 pid=11013 cs=0xffff0000 pc=0xfffffff0
tcg_gen_code 86.982 pid=11013 cs=0xf0000 pc=0xfe05b
```

You may got empty output, that because QEMU has not write output buffer to trace-pid(trace-11123) yet. 
if you get empty output, please try to flush trace-file.

```
(qemu) trace-file flush
```

You can also use CLI to disable/enable the trace event you just added:

```bash
# Disable load_file
(qemu) trace-event tcg_gen_code off/on
```

## Simple trace internal 
As you can see, we can easily add a tracing event with only 2 line code, it allows us to interact with CLI. 
QEMU makefile will generate code automatically. when we define a tracing event in trace-events, it will automatically generate the following code:

```C
// File: trace/generated-tracers.c
TraceEvent _TRACE_TCG_GEN_CODE_EVENT = {
    .id = 0,
    .vcpu_id = TRACE_VCPU_EVENT_NONE,
    .name = "tcg_gen_code",
    .sstate = TRACE_TCG_GEN_CODE_ENABLED,
    .dstate = &_TRACE_TCG_GEN_CODE_DSTATE  
};
TraceEvent *common_trace_events[] = {
...
&_TRACE_TCG_GEN_CODE_EVENT,
...
};

void _simple_trace_tcg_gen_code(uint64_t cs, uint64_t pc)
{   
    TraceBufferRecord rec;
    
    if (!trace_event_get_state(TRACE_TCG_GEN_CODE)) {
        return;
    }
    
    if (trace_record_start(&rec, _TRACE_TCG_GEN_CODE_EVENT.id, 8 + 8)) {
        return; /* Trace Buffer Full, Event Dropped ! */
    }
    trace_record_write_u64(&rec, (uint64_t)cs);
    trace_record_write_u64(&rec, (uint64_t)pc);
    trace_record_finish(&rec);
}   
```

In ./trace/generated-tracers.h file:

```C
static inline void trace_tcg_gen_code(uint64_t cs, uint64_t pc)
{
    if (true) {
        _simple_trace_tcg_gen_code(cs, pc);
    }   
}   
```

Remember we called trace_tcg_gen_code before, you call trace_tcg_gen_code from other place where you want to observe too.

```diff
diff --git a/target-i386/translate.c b/target-i386/translate.c
@@ -8325,6 +8325,8 @@ void gen_intermediate_code(CPUX86State *env, TranslationBlock *tb)
     cs_base = tb->cs_base;
     flags = tb->flags;
 
+       trace_tcg_gen_code(cs_base, pc_start);
+
     dc->pe = (flags >> HF_PE_SHIFT) & 1;
     dc->code32 = (flags >> HF_CS32_SHIFT) & 1;
     dc->ss32 = (flags >> HF_SS32_SHIFT) & 1;
```
