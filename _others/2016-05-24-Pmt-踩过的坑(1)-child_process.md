---
title: Pmt 踩过的坑(1) child_process
---

### 场景

当守护进程不存在的时候，需要把守护进程启动起来。
守护进程需要后台运行，当前进程需要正常退出。
选择用**child_process**来实现启动守护进程。

### 实现

#### bootDaemon

```javascript
var child = require('child_process').spawn('node', ['daemon.js'], {
    detached   : true,
    stdio      : ['ipc', 'ignore', 'ignore']
});
child.once('message', function (msg){
    if (msg === 'success'){
        child.unref();
        child.disconnect();
    }
})
```

#### daemon.js

```javascript
process.send('success');
```

### 关键点
1. **detached : true**。当 detached 设为 true 时子进程可以在父进程退出后可以继续运行。
2. **stdio: ['ipc', 'ignore', 'ignore']**。在父子进程之间创建一个IPC通道，child.send方法只有在开启IPC通道时才可以激活。默认情况下，子进程会一同附着在父进程所附着的终端上，将 stdout 与 stderr 设为 ignore 是为了把子进程的输出指向/dev/null，不然就算父进程退出了，子进程还是附着在终端上，依旧无法后台运行。
3. **child.unref()**。默认情况下，父进程会等待子进程的退出自己才会正常退出，child.unref()会将子进程从父进程的循环引用计数中移除，使得父进程不必等待子进程退出后自己才能退出。
4. **child.disconnect**。用来关闭之前开启的IPC通道，使得父进程可以自己安全退出。
