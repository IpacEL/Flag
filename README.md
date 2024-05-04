
#### 参考文档: 
> https://wiki.openjdk.org/display/shenandoah/Main  
> 2019.12基准报告: https://ionutbalosin.com/2019/12/jvm-garbage-collectors-benchmarks-report-19-12/  
> 2020.1性能调优指南: https://ionutbalosin.com/2020/01/hotspot-jvm-performance-tuning-guidelines/  
> https://it-blog-cn.com/blogs/jvm/jit_aot.html  
> https://it-blog-cn.com/blogs/jvm/jit_optimize.html  
> 搜索引擎, 以及各种文章: https://cn.bing.com/  

#### 参考实例: 
> SGC: https://github.com/hilltty/hilltty-flags  
> aikar-G1GC: https://aikar.co/mcflags.html  
> etil-G1GC-new: https://github.com/etil2jz/etil-minecraft-flags  
> VeroFess-ZGC: https://blog.binklac.com/e6ad4dc21152/  
> GraalVM-G1GC: https://github.com/brucethemoose/Minecraft-Performance-Flags-Benchmarks  

---
### 成品/说明和草稿

最后更新时间: `2023年1月28日`  
```
-server -Xms16G -Xmx16G -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCMode=iu -XX:+AlwaysPreTouch -XX:+ParallelRefProcEnabled -XX:+DisableExplicitGC -XX:+UseNUMA -XX:ReservedCodeCacheSize=512M -XX:NonNMethodCodeHeapSize=16M -XX:NonProfiledCodeHeapSize=194M -XX:NmethodSweepActivity=1 -XX:+UseCriticalJavaThreadPriority -XX:MaxInlineSize=256 -XX:InlineSmallCode=4000 -XX:+SegmentedCodeCache -XX:-DontCompileHugeMethods -XX:+OmitStackTraceInFastThrow -XX:ThreadPriorityPolicy=1 -XX:+TrustFinalNonStaticFields -XX:+UseFastUnorderedTimeStamps -XX:+AllowParallelDefineClass -XX:UseAVX=3 -XX:+UseFMA -XX:+UseSSE42Intrinsics -XX:+UseXmmI2D -XX:+UseXmmI2F -XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb -Dlog4j2.formatMsgNoLookups=true --add-modules=jdk.incubator.vector -jar
```

尝试在客户端使用  
```
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCMode=iu -XX:+AlwaysPreTouch -XX:+ParallelRefProcEnabled -XX:+DisableExplicitGC -XX:+UseNUMA -XX:MaxInlineSize=256 -XX:+SegmentedCodeCache -XX:-DontCompileHugeMethods -XX:+OmitStackTraceInFastThrow -XX:ThreadPriorityPolicy=1 -XX:+TrustFinalNonStaticFields -XX:+UseFastUnorderedTimeStamps -XX:+AllowParallelDefineClass -XX:UseAVX=3 -XX:+UseFMA -XX:+UseSSE42Intrinsics -XX:+UseXmmI2D -XX:+UseXmmI2F -XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb --add-modules=jdk.incubator.vector
```

---

注释: 
```
-server	# 这在JDK中是默认的, 只是习惯性放在这里
-Xms16G -Xmx16G	# 预计分配的内存大小
-XX:+UnlockExperimentalVMOptions	# 启用实验选项
```

```
-XX:+UseShenandoahGC		# 启用SGC
-XX:ShenandoahGCMode=iu	# SGC模式 [实验选项]

-XX:+AlwaysPreTouch		# 分配连续的内存并在启动时保留

# 大页面内存需要权限, 在windows中名为 "锁定内存页"
[需要权限] -XX:+UseLargePages		# 使用大页面内存
[需要权限] -XX:LargePageSizeInBytes=1G		# 内存分页 1G
[需要权限] -XX:+UseSHM
[需要权限] -XX:+UseTransparentHugePages		# 透明大页面
[影响性能分析器] -XX:+PerfDisableSharedMem		# 将匿名内存用于性能计数器

```

```
-XX:+ParallelRefProcEnabled	# 尽可能启用并行引用处理 缓存
-XX:+DisableExplicitGC		# 忽略代码中的 System.gc() 调用 防止插件调用GC
[需要系统支持] -XX:+UseNUMA		# NUMA
# 代码缓存
-XX:ReservedCodeCacheSize=512M
-XX:NonNMethodCodeHeapSize=16M
-XX:NonProfiledCodeHeapSize=194M
-XX:NmethodSweepActivity=1	# 允许不常使用的代码缓存保存更长时间
-XX:+UseCriticalJavaThreadPriority	# 使应用线程的优先级更高(高于GC和编译器)
-XX:ThreadPriorityPolicy=1	# 允许提高线程优先级
-XX:MaxInlineSize=256		# 方法体大小小于此值进行内联
-XX:InlineSmallCode=4000	# 允许内联更多代码
-XX:+SegmentedCodeCache		# 使用分段的代码缓存区
-XX:-DontCompileHugeMethods	# 允许大方法被JIT编译
[!] -XX:+OmitStackTraceInFastThrow	# 在快速抛出中省略堆栈跟踪
[!] -XX:+TrustFinalNonStaticFields	# 信任常量折叠的最终非静态声明
-XX:+UseFastUnorderedTimeStamps	# 快速无序时间戳
-XX:+AllowParallelDefineClass	# 允许并行加载类
# 使用更新的指令集
-XX:UseAVX=3 -XX:+UseFMA -XX:+UseSSE42Intrinsics
-XX:+UseXmmI2D -XX:+UseXmmI2F
-XX:+UseVectorCmov -XX:+UseNewLongLShift -XX:+UseFastStosb

# 模块系统
--add-modules=jdk.incubator.vector	# pufferfish
```

临时: 
```
-XX:+UseNUMAInterleaving	# 在不支持完整NUMA的系统上提供该子集
# -XX:CompileThreshold=500	# 使用C2编译的阈值
-Dlog4j2.formatMsgNoLookups=true	# log4j2 漏洞解决方案
[影响性能分析器] -XX:+PerfDisableSharedMem		# 将匿名内存用于性能计数器

# SGC
-XX:ShenandoahGCHeuristics=static

# jconsole
-Djava.rmi.server.hostname=0.0.0.0 -Dcom.sun.management.jmxremote.port=2777 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dspring.profiles.active=ci

# 原版服务端
-nogui	# 关闭GUI

# spigot
--world-dir <worlds>	# 将地图放进单独的目录
--forceUpgrade	# 升级所有区块到当前版本
```
