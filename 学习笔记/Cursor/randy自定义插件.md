提供两个工具，两个工程都要求本地openspec >= 1.6.0。有问题可以给我反馈，尤其是aihelp-workspace，[握手]

1. aihelp-dev-plugin升级到0.1.7，自行升级就行了
    
2. 主要改动是增加了update-change的skill和多仓开发的基础skill，其他变化看 [https://git.aihelp.net/randy/aihelp-cursor-plugin/-/blob/main/CHANGELOG.md](https://git.aihelp.net/randy/aihelp-cursor-plugin/-/blob/main/CHANGELOG.md) 。
    
3. 当工程中未归档的change需要做修改可以使用explore/review + update-change来变更change，然后继续使用Apply继承上下文实现变更。
    

4. aihelp-workspace多仓库开发的基础工程：[https://git.aihelp.net/devops1/aihelp-workspace](https://git.aihelp.net/devops1/aihelp-workspace) 使用方法和流程看git中的readme。必须要有aihelp-dev-plugin的skill支持和openspec >= 1.6.0。主仓库的spec可以随MR提交的时候一起提交到workspace中，归档后的change可以提交可以不提，重点是这里的spec。

![](https://static.dingtalk.com/media/lQLPJwivkCk2QOPNC8jNB3qw-vcg9rNu2uoKQ0jmXkQnAA_1914_3016.png)