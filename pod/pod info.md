# pod状态、成因、处理方法

|      pod状态      | 是否正常 |                             成因                             |                             影响                             |                           处理方法                           |
| :---------------: | :------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|      Pending      | Unknown  | 未调度状态，长时间处于Pending状态是不正常的，可能的原因有：<br />1.调度器未正常工作<br />2.无匹配资源 | 1.占用调度资源，一直在队列中重新调度<br />2.console用户看到容器一直转圈，无法正常启动<br /> | 1.更新调度器版本，解决占用调度资源问题<br />2.调度器不正常工作问题需人工介入 |
|   OutOfstorage    | Unknown  |      调度到某节点，kubelet检查Storage资源不匹配，未准入      | 1.直接用pod创建的资源创建失败<br />2.rc/rs/deploy等创建的资源无影响 |                 通过中控节点删掉、或者不处理                 |
|     OutOfcpu      |    No    |       调度到某节点，kubelet检查到CPU资源不匹配，未准入       | 1.直接用pod创建的资源创建失败<br />2.rc/rs/deploy等创建的资源无影响 |                 通过中控节点删掉、或者不处理                 |
|    OutOfmemory    |    No    |      调度到某节点，kubelet检查memory资源不匹配，未准入       | 1.直接用pod创建的资源创建失败<br />2.rc/rs/deploy等创建的资源无影响 |                 通过中控节点删掉、或者不处理                 |
| ImagePullBackOff  |    No    | 镜像拉取失败，可能的原因有：<br />1.镜像不存在<br />2.网络问题导致无法成功拉取镜像 |                  容器启动失败，无法正常工作                  |                   人工介入，检查镜像和网络                   |
| ContainerCreating | Unknown  | 容器创建中，长时间处于Creating状态是不正常的，可能的原因有：<br />1.磁盘挂载超时<br />2.其它 |                长时间Creating导致容器启动失败                | 1.挂载磁盘超时问题通过定期扫描宿主机目录解决<br />2.其他问题待具体分析 |
| NetworkPingFailed |    No    |                         容器网络不通                         |                  容器网络不通，无法正常工作                  |                     待网络组提供解决方案                     |
|      Running      |   Yes    |                             N/A                              |                             N/A                              |                             N/A                              |
|    Rebuilding     | Unknown  | 容器重建中，长时间处于Rebuilding状态是不正常的，可能的原因有：<br />1.待补充 |                             N/A                              |                            待观察                            |
| CrashLoopBackOff  |    No    | 容器start成功，但是容器进程异常退出，再次启动前，会处于CrashLoopBackOff状态，有一定的等待时间 |                       容器无法正常工作                       |                      需用户更新启动程序                      |
|       Error       |    No    | 容器启动过程中某环节报错，可能是第一次启动，也可能是因某些原因stop之后再次启动，原因有可能是：<br />1.因网络导致启动失败，日志：NetworkPlugin cni failed on the status hook for pod<br />2.kubelet挂掉，或者kubelet无法正常工作，比如status为Running，但是日志停留在很久之前<br />3.待补充 |                  容器启动出错，无法正常工作                  |     1.待网络组提供解决方案<br />2.归属于kubelet修复问题      |
|      Evicted      |    No    |                          容器被驱逐                          | 1.直接用pod创建的资源创建失败<br />2.rc/rs/deploy等创建的资源无影响 |                 通过中控节点删掉、或者不处理                 |
|    Terminating    | Unknown  | 容器删除中，长时间处于Terminating状态是不正常的，可能的原因：<br />1.容器磁盘清理环境报错<br />2.容器清理环节报错<br />3.kubelet挂了 |                资源长时间被占用，影响新的pod                 | 1.脚本处理，通过观察两种报错的日志信息，做处理操作<br />2.kubelet挂了归属于kubelet修复问题 |
|      RPCErr1      |    No    | 容器启动报错，可能原因有：<br />1.盘没了、或坏了，lvs、vgs、pvs无任何输出，或提示输入输出错误<br />2.待补充 |                       容器无法正常工作                       |                         1.需人工介入                         |
|      RPCErr2      |    No    | 容器启动报错，原因是docker-daemon出问题，手动执行docker start也报同样错误 |                       容器无法正常工作                       |  需要解决docker-daemon问题，或者重启修复，若重启需人工介入   |
|      RPCErr3      |    No    | 容器启动报错，原因是pause容器为exited状态，pause容器为exited状态的原因可能有：<br />1.NetworkPlugin cni failed on the status hook for pod<br />2.待补充 |                       容器无法正常工作                       |                     待网络组提供解决方案                     |
|      RPCErr4      |    No    | 容器启动报错，可能的原因有：<br />1.盘没了、或坏了，lvs、vgs、pvs无任何输出，或提示输入输出错误<br />2.待补充 |                       容器无法正常工作                       |                         1.需人工介入                         |



### 注释：

RPCErr1：rpc error: code = 2 desc = failed to start container "": Error response from daemon: chown /export/docker/devicemapper/mnt: read-only file system

RPCErr2：rpc error: code = 2 desc = failed to start container "": Error response from daemon: Container command not found or does not exist.

RPCErr3：rpc error: code = 2 desc = failed to start container "": Error response from daemon: cannot join network of a non running container: 

RPCErr4：rpc error: code = 2 desc = Error response from daemon: devmapper: Error saving transaction metadata: devmapper: Error creating metadata file: open /export/docker/devicemapper/metadata/.tmp796870143: read-only file system