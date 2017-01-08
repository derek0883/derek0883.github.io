---
layout: post 
title: "Ubuntu 16.04 systemtap fixing"
categories: Linux
comments: true
---

> Updated my system to ubuntu 16.04, systemtap script can't not run, here is the step how manully solve this issue.

## fixing steps

Updated my system to ubuntu 16.04 for few months, Trying to run a simple systemtap script, always got error like this:

```
/usr/share/systemtap/runtime/transport/symbols.c: In function ?_stp_module_update_self?:
/usr/share/systemtap/runtime/transport/symbols.c:243:44: error: ?struct module? has no member named ?symtab?
if (attr->address == (unsigned long) mod->symtab)
```

Found the following post through google search:

[https://bugs.launchpad.net/ubuntu/+source/systemtap/+bug/1537125](https://bugs.launchpad.net/ubuntu/+source/systemtap/+bug/1537125){:target="_blank"}

Based on [this post](https://sourceware.org/bugzilla/show_bug.cgi?id=19644){:target="_blank"},
The root cause is: 
> Linux kernel commit 8244062ef1 removes the 'symtab' member of 'struct module'. 

It also says:

> David Smith 2016-02-16 17:07:37 UTC

> Fixed in commit 64ffc49.

At beginning I thought the fix commit 64ffc49 is for the kernel, checked out kernel git log of include/linux/module.h, didn't found that commit.
Then checked out systemtap source code git://sourceware.org/git/systemtap.git

```
commit 64ffc49b5deb57e19e94f67f9878f5c03e617775
Author: David Smith <dsmith@redhat.com>
Date:   Tue Feb 16 11:06:58 2016 -0600

    Fix PR19644 by updating the runtime to handle linux 4.5 commit 8244062ef1.

    * runtime/transport/symbols.c (_stp_module_update_self): Handle kernel
      change moving module symbol table information into a 'struct
      mod_kallsyms'.
    * runtime/linux/autoconf-mod_kallsyms.c: New autoconf test.
    * buildrun.cxx (compile_pass): Add autoconf test for 'struct
      mod_kallsyms'.

commit 4075f4b3ad52cc2f41808e9c069aaa6edee4d7e0
```

git diff 4075f4b  64ffc49 get following changes:

```diff
diff --git a/buildrun.cxx b/buildrun.cxx
index a923a6a..3a427a8 100644
--- a/buildrun.cxx
+++ b/buildrun.cxx
@@ -444,6 +444,8 @@ compile_pass (systemtap_session& s)

   output_autoconf(s, o, "autoconf-module_layout.c",
          "STAPCONF_MODULE_LAYOUT", NULL);
+  output_autoconf(s, o, "autoconf-mod_kallsyms.c",
+         "STAPCONF_MOD_KALLSYMS", NULL);

   o << module_cflags << " += -include $(STAPCONF_HEADER)" << endl;

diff --git a/runtime/linux/autoconf-mod_kallsyms.c b/runtime/linux/autoconf-mod_kallsyms.c
new file mode 100644
index 0000000..e286440
--- /dev/null
+++ b/runtime/linux/autoconf-mod_kallsyms.c
@@ -0,0 +1,3 @@
+#include <linux/module.h>
+
+struct mod_kallsyms mk;
diff --git a/runtime/transport/symbols.c b/runtime/transport/symbols.c
index 41782f2..cb7964f 100644
--- a/runtime/transport/symbols.c
+++ b/runtime/transport/symbols.c
@@ -248,10 +248,17 @@ static int _stp_module_update_self (void)
            found_eh_frame = true;
        }
        else if (!strcmp(".symtab", attr->name)) {
-           _stp_module_self.sections[0].static_addr = attr->address;
+#ifdef STAPCONF_MOD_KALLSYMS
+           struct mod_kallsyms *kallsyms = rcu_dereference_sched(mod->kallsyms);
+           if (attr->address == (unsigned long) kallsyms->symtab)
+               _stp_module_self.sections[0].size =
+                   kallsyms->num_symtab * sizeof(kallsyms->symtab[0]);
+#else
            if (attr->address == (unsigned long) mod->symtab)
                _stp_module_self.sections[0].size =
                    mod->num_symtab * sizeof(mod->symtab[0]);
+#endif
+           _stp_module_self.sections[0].static_addr = attr->address;
        }
        else if (!strcmp(".text", attr->name)) {
            _stp_module_self.sections[1].static_addr = attr->address;
```

It is very simple change, only 3 files. cheers! buildrun.cxx is not related if you don't compile systemtap from scratch.
I don't want to compile systemtap from scratch, so just saved this diff to /tmp/stap.diff, and removed changes for buildrun.cxx
then goto /usr/share/systemtap, 

```bash
/usr/share/systemtap $ sudo patch -p1 < /tmp/stap.diff.
```

After that, I manually added "#define STAPCONF_MOD_KALLSYMS 1" at front of runtime/linux/autoconf-mod_kallsyms.c.

Then all of my systemtap scripts works again. Cheers!

## SystemTap error on new kernel
Recently I updated kernel to 4.8.28 on my thinkpad, SystemTap script can't run again, got error:

```
/usr/share/systemtap/runtime/linux/access_process_vm.h:35:29: error: 
passing argument 1 of ‘get_user_pages’ makes integer from pointer without a 
cast [-Werror=int-conversion]
ret = get_user_pages (tsk, mm, addr, 1, write, 1, &page, &vma);
```

That becaus Systemtap version is not match newer kernel version, I Use following command to install old kernel back.

```bash
$ sudo apt install linux-image-4.4.0-59-generic-dbgsym \
	linux-image-4.4.0-59-generic \
	linux-headers-4.4.0-59-generic \
	linux-image-extra-4.4.0-59-generic

```

Use strace to find the error:

```
$ sudo strace -e open stap --vp 09 -e 'probe kernel.function("sys_open") {log("hello world") exit()}' |& grep vmlinu
```
