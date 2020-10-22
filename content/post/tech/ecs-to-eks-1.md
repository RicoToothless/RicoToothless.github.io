+++
author = "Rico"
title = "ECS migrate to EKS part 1"
date = "2020-01-13"
description = "why we start ECS migrate to EKS?"
tags = [
    "ecs",
    "eks",
    "docker",
    "kubernetes",
    "cdk",
    "aws",
]
categories = [
    "tech",
]
archives = "2020-01"
+++

# What is ECS?

這裡簡單的介紹一下 ECS ，他是一個簡單好上手的 container orchestration ，個人覺得比 Docker Swarm 還要容易上手。

ECS 我們都只使用 EC2 的底層，他的架構如下：

![ecs-architecture](/ecs-architecture.png)

我特別標示了 Auto Scaling Group 的部分可以看出，實際上 ECS host 也就是所謂的 EC2 Instance ，如果要一個 cluster 裡面要多台 EC2 就是靠 Auto Scaling Group 自動 create EC2 給你，之後再靠 `task definition file` 和 `service` 來定義 container 要怎麼啟動。

# What is EKS?

EKS 本質上就是受到 AWS 託管的 Kubernetes ，跟自架的 Kubernetes cluster 相比， EKS 幫使用者做好了 control plane master node HA 之外， network 的部分也整合到 AWS VPC 裡面，這樣就不用太過煩惱要用什麼 Kubernetes 那種網路方案，也不用煩惱網路效能的問題。還有整合 AWS IAM 到 EKS 裡，權限的控管就不用特別在 EKS 裡再做一次了。

EKS 的架構如下：

![eks-architecture](/eks-architecture.png)

其實 EC2 的增長跟 ECS 用的都是同一套 Auto Scaling Group ，而使用者的 application 怎麼跑就是使用 Kubernetes API 定義的範圍裡寫 YAML file 來定義。

# Why ECS migrate to EKS?

一開始我們的架構很單純， ECS 裡面架設自己的 app 加上 RDS ，但後來漸漸地需要各式各樣的工具，像是儲存 secret 、 message queue 和 cache 等等。而且這些工具都是 stateful 的服務，光是本身要做 HA 就不太容易，而在 ECS 上做 HA 更是讓 ECS 的優勢不再凸顯，在 ECS 上架設 HA 跟直接在一般的 EC2 上架設差不多。

一大主因是對於 ECS storage 雖然也有類似 Kubernetes PV （Persistent Volume）的概念，而且 ECS 用 EBS 也可以用 user-data 做到類似的效果，但基本上 ECS 對於 mount volume 只有 Docker 的實作，而 Kubernetes 的實作包含了兩個，一個是 controller + CRI（通常都是用 Docker） 的實作。所以最大問題就來了， ECS 最一開始 volume 要 assign 到哪一個 node 上要怎麼決定？在 ECS 對於 worker node 都是以 Auto Scaling Group 的做法， user-data 的方法很在多台 node 下的 cluster 明顯是有問題的。


|     | 如何 assign volume 到 worker node       | 如何 mount 到 container 裡                        |
|-----|-----------------------------------------|---------------------------------------------------|
| ECS | user-data or 自己手動                   | Docker 實作                                       |
| EKS | master node 上的 AttachDetachController | worker node 裡 kubelet 的 VolumeManagerReconciler |



（不過最近 ECS 正在推出 EFS storage ，[教學文傳送門在此](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_efs.html)，雖然沒有實際實驗過，看起來是為了 application 不一定非要綁死特定的 node 上而設計的，目前正在 beta 版中。）

或許會問，不論是 secret 、 message queue 或 cache 也好，這些其實都有 AWS 現成的服務，為何不選用呢？原因不外乎別的，就是成本考量。而 Kubernetes 有個蓬勃發展的生態圈，像是 Helm 也是讓很多管理人員省下不少事，而雖然 Helm 很方便，但需要 debug 的時候還是得了解其原理才行。幸好我在 ECS 架設 Vault 、 Redis 以及 Rabbitmq 時已經了解不少，畢竟要是不了解根本無法在 ECS 上＂完美的＂架起來，因為有點追求完美的個性，會希望即使關機也能夠快速恢復，但又因為 ECS 的特性，最後真的用得很不俐落。

還有一點推力是 ECS & EKS 本是同根生，同樣都是使用 container 技術， migrate 非常容易， code 不需要做任何變更，所以完全不會影響開發進度。

原本我們只有兩個環境，當增長到三個環境時候發現，手動建置整個環境非常勞累，綜合上述的總總，是時候該還技術債了。

而為什麼我們選擇 EKS 而非搬到 GCP 上？除了我們已經有很多像是 IAM 的權限、網路 VPN 以及 ECR 的服務之外，最殺手級的主因是 AWS CDK 。

# What & Why AWS CDK?

AWS CDK 是專門定義 AWS resources 的 IaC 工具，他支援多種語言像是 TypeScript 、 Python 、 JAVA 等等，也陸陸續續有新的語言支援，而 AWS CDK 最大的罩門就是目前只限於 AWS resources 。其實 AWS 原本就有 IaC 工具為 CloudFormation ，而 AWS CDK 再往上做一層抽象層，所以 AWS CDK 實際要執行的時候，最後 generate 出來的其實就是 CloudFormation 的格式。另外 AWS CDK 可以使用原本熟悉語言的特性，和這個工具本身就是 AWS 自己的產品，理應會得到 AWS resource 自身最好的整合，這也是我最後不選擇 Terraform 的原因。

我本身對於 IaC 就一直非常想要導入， IaC 好處非常得多，像是快速部屬多個環境、減少不一致性、增加 Infra 透明度、版本控管、減少人工手動建設的時間（節省人力成本）等等好處。另外還有一些個人的原因就是我想一些寫程式的實戰經驗，其他人寫程式不外乎就是寫爬蟲、網站或聊天機器人，可是這些我真的沒興趣，寫 IaC 就是比較可以給我成就感，或許只有我這麼怪異吧？

另外 AWS CDK 是 open source project，有覺得不符合預期行為的地方，都可以去翻 code 是不是 bug ，或提出自己新 feature 的想法。 AWS CDK 由 AWS 自己主導外，也歡迎所有有志之士一起貢獻，例如目前只會開 issue 的我。

# Manager support

當我把這個計劃提出後，主管算蠻支持的，除了把遇到的難題跟未來的瓶頸講清楚，以及這樣 refactor architecture 的好處是什麼，不過只要服務不中斷以及 billing 成本不要上升（其實很難，尤其 EKS control plane 又有另外收管理費的錢，只能從 spot instance & EKS Fargate 著手了），基本上主管都放手讓我去做。

# Conclusion

* ECS 不太適合使用 stateful services 所以遷移到 EKS 上
* Kubernetes 生態圈可以快速對整個系統做客製化
* IaC 好處很多，且工具有許多選擇
* AWS CDK 確切的符合我們的需求，再加上是 open source project
* 主管的目標導向要先了解，符合他們的需求後再做導入
