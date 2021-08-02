---
title: iOS逆向获取block的参数类型和返回值
tags: 流弊技能
keywords: iOS, block, 逆向工程, iOSRe, iOS逆向
description: iOS block怎么逆向，怎么逆向block参数？
---

#### block内存结构

```cpp
struct Block_literal_1 {
    void *isa; // initialized to &_NSConcreteStackBlock or &_NSConcreteGlobalBlock
    int flags;
    int reserved;
    void (*invoke)(void *, ...);
    struct Block_descriptor_1 {
        unsigned long int reserved; // NULL
        unsigned long int size;     // sizeof(struct Block_literal_1)
        // optional helper functions
        void (*copy_helper)(void *dst, void *src);     // IFF (1<<25)
        void (*dispose_helper)(void *src);             // IFF (1<<25)
        // required ABI.2010.3.16
        const char *signature;                         // IFF (1<<30)
    } *descriptor;
    // imported variables
};
```

可以看到第一个成员是 isa，说明了 Block 在 Objective-C 当中也是一个对象。我们重点要关注的就是 void (*invode)(void *, ...); 和 descriptor 中的 const char *signature，前者指向了 Block 具体实现的地址，后者是表示 Block 函数签名的字符串。

~~~rust
//得到block地址
(lldb) po $x5
<__NSMallocBlock__: 0x17425ff20>

//得到invoke函数指针地址（invoke 函数指针的地址就是在第 16 个字节之后）
(lldb) memory read --size 8 --format x 0x17425ff20
0x17425ff20: 0x000000019a64f188 0x00000000c3000002
0x17425ff30: 0x0000000183b5d908 0x00000001969e89f0
0x17425ff40: 0x000000014f841800 0x000000017466d840
0x17425ff50: 0xbadd740d3e15bead 0x0000000174440fc0

//对这个地址反汇编
(lldb) disassemble --start-address 0x0000000183b5d908
CFNetwork`__62-[__NSCFLocalDataTask0x00000001969e89f0 _onqueue_didReceiveResponse:completion:]_block_invoke:
    0x183b5d908 <+0>:  stp    x20, x19, [sp, #-0x20]!
    0x183b5d90c <+4>:  stp    x29, x30, [sp, #0x10]
    0x183b5d910 <+8>:  add    x29, sp, #0x10            ; =0x10 
    0x183b5d914 <+12>: sub    sp, sp, #0x40             ; =0x40 
    0x183b5d918 <+16>: mov    x19, x1
    0x183b5d91c <+20>: mov    x20, x0
    0x183b5d920 <+24>: ldr    x0, [x20, #0x20]
    0x183b5d924 <+28>: adrp   x8, 77511`


//或者下断电就可以进入到该方法
(lldb) br s -a "0x0000000183b5d908"
Breakpoint 2: where = CFNetwork`HTTPProtocol::pacExecutionComplete(__CFArray const*, __CFError*) + 500, address = 0x0000000183be5908

~~~

#### 找出 Block 的函数签名 (确认block有函数签名)

要找出 Block 的函数签名，需要通过 descriptor 结构体中的 signature 成员，然后通过它得到一个 NSMethodSignature 对象。

首先，需要找到 descriptor 结构体。这个结构体在 Block 中是通过指针持有的，它的位置正好在 invoke 成员后面，占用 8 个字节。可以从上面的内存打印中看到 descriptor 指针的地址是 0x00000001969e89f0。

接下来，就可以通过 descriptor 的地址找到 signature 了。但是，文档指出并不是每个 Block 都是有方法签名的，我们需要通过 flags 与 block 中定义的枚举掩码进行与判断。还是在刚刚的 llvm 文档中，我们可以看到掩码的定义如下：

``` Objective-C
enum {
    BLOCK_HAS_COPY_DISPOSE =  (1 << 25),
    BLOCK_HAS_CTOR =          (1 << 26), // helpers have C++ code
    BLOCK_IS_GLOBAL =         (1 << 28),
    BLOCK_HAS_STRET =         (1 << 29), // IFF BLOCK_HAS_SIGNATURE
    BLOCK_HAS_SIGNATURE =     (1 << 30),
};
*/
(lldb) memory read --size 4 --format x 0x17425ff20
0x17425ff20: 0x9a64f188 0x00000001 0xc3000002 0x00000000
0x17425ff30: 0x83b5d908 0x00000001 0x969e89f0 0x00000001
(lldb) expr -- ((0xc3000002 & (1 << 30)) != 0)
(bool) $28 = true
/*

由于 ((0xc3000002 & (1 << 30)) != 0)，因此我们可以确定这个 Block 是有签名的。
为了找出 signature 的地址，我们还需要确认这个 Block 是否拥有 copy_helper 和 disponse_helper 这两个可选的函数指针。
由于 ((0xc3000002 & (1 << 25)) != 0)，因此我们可以确认这个 Block 拥有刚刚提到的两个函数指针。
*/

```

~~~Rust
/*
现在可以总结下：
signature 的地址是在 descriptor 下偏移两个 unsiged long 和两个指针后的地址，即 32 个字节后。
*/
(lldb) memory read --size 8 --format x 0x00000001969e89f0
0x1969e89f0: 0x0000000000000000 0x0000000000000030
0x1969e8a00: 0x0000000183b5da48 0x0000000183b5da84
0x1969e8a10: 0x0000000183d0a5ee 0x0000000000000200
0x1969e8a20: 0x0000000000000000 0x0000000000000028

(lldb) p (char *)0x0000000183d0a5ee
(char *) $8 = 0x0000000183d0a5ee "v16@?0q8"

//通过 NSMethodSignature 找出它的参数类型
(lldb) po [NSMethodSignature signatureWithObjCTypes:"v16@?0q8"]
<NSMethodSignature: 0x170873600>
    number of arguments = 2
    frame size = 224
    is special struct return? NO
    return value: -------- -------- -------- --------
        type encoding (v) 'v'
        flags {}
        modifiers {}
        frame {offset = 0, offset adjust = 0, size = 0, size adjust = 0}
        memory {offset = 0, size = 0}
    argument 0: -------- -------- -------- --------
        type encoding (@) '@?'
        flags {isObject, isBlock}
        modifiers {}
        frame {offset = 0, offset adjust = 0, size = 8, size adjust = 0}
        memory {offset = 0, size = 8}
    argument 1: -------- -------- -------- --------
        type encoding (q) 'q'
        flags {isSigned}
        modifiers {}
        frame {offset = 8, offset adjust = 0, size = 8, size adjust = 0}
        memory {offset = 0, size = 8}

(lldb) 
//没有返回值，两个参数一个我们自己的 block 的引用，一个longlong类型的
~~~

### 福利部分
[阿里云服务器](https://www.aliyun.com/minisite/goods?source=5176.11533457&userCode=j6bryttg)

[阿里云企业级服务器限时特惠](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?source=5176.11533457&userCode=j6bryttg)

[阿里云建站 速成美站](https://ac.aliyun.com/application/webdesign/sumei?source=5176.11533457&userCode=j6bryttg)

[阿里云SaaS商标注册服务](https://tm.aliyun.com/?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

[CDN爆款产品](https://yqh.aliyun.com/live/cdncarnival?userCode=j6bryttg&source=5176.11533457&userCode=j6bryttg)

---------------------------------------------------------------------------------------------
[腾讯云服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1067&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云境外服务器](https://cloud.tencent.com/act/cps/redirect?redirect=1001&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)

[腾讯云数据库](https://cloud.tencent.com/act/cps/redirect?redirect=1003&cps_key=49b140a6ab2ad2bbd5cd3a951a0661fc&from=console)
