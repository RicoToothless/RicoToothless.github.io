+++
author = "Rico"
title = "ECS migrate to EKS part 3"
date = "2020-10-20"
description = "AWS CDK EKS package experience"
tags = [
    "eks",
    "cdk",
    "kubernetes",
    "aws",
]
categories = [
    "tech",
]
archives = "2020-10"
+++

# How to start your AWS CDK journey?

在台灣 [AWS CDK meetup#1](https://cdkmeetup.kktix.cc/events/fristmeetup) 裡有分享過[新手 operator 寫 CDK 之旅](https://speakerdeck.com/ricotoothless/taiwan-cdk-meetup-rookie-operators-cdk-journey)（建議把整個 PDF 下載下來，裡面有很多連結），初學者怎麼入門後快速熟悉整個 CDK 生態圈，最近 AWS 在全球非常積極推動 CDK，各個關於 CDK 的分享或資源真的全世界各個國家都有，裡面的資源可以陪伴新手從初級到進階都沒問題。

# AWS CDK EKS Overview

AWS CDK 把每一個 AWS 服務包成一個 package，每一個 package 成熟度不一，像是 AWS CDK EKS 就還在 experiment 階段，也曾經因為 CDK EKS 有非常大的 breaking change，不得不重新創一個新的 EKS cluster，並且把所有的 application  migrate 過去。最近 EKS 的功能越來越齊全了，像是可以創建 IRSA（IAM Roles for Service Accounts）的 Service Account 之外，還會把 OIDC ID provider 自動創建起來，非常地方便。實際建置時 EKS 除了 application CICD 是用 Jenkins 做之外，像是 backing services、monitoring 的建置，還有 backing services 本身、monitoring 與 application 本身的 IRSA 都是靠 CDK 建置的。例如：[Hashicorp Vault KMS Auto-Unseal](https://learn.hashicorp.com/tutorials/vault/autounseal-aws-kms) 會需要 AWS KMS 的權限來調用 KMS 做為 Vault Master Key 來加解密敏感資料。

## AWS CDK EKS Helm Chart management

Kubernetes 使用者對於 Helm 的使用率還是非常地高，建置服務的時候可以大幅縮短自己造輪子的時間，想當初建置一個 rabbitmq cluster、redis sentinel 或者 vault 等等都要在不彈性的 ECS 上建置，就會覺得有類似於 package manager 功能的工具是非常需要的。也因此，CDK EKS 有支援 Helm 的確不太意外，但在實際上使用的時候會發現有客製 chart repository server 的需求，就要小心不要遇到雞生蛋蛋生雞的問題，例如把 chart repository 放在 [Nexus](https://www.sonatype.com/nexus/repository-oss) 裡，然後 Nexus 是透過 ingress controller 代理的，又剛好這個 ingress controller 是客製化的 chart，就有可能會變成在 CDK deploy 的時候因為 ingress 沒有啟動而抓不到 Nexus server 裡面的 chart。所以這裡的建議是放在 ECR or S3 會比較好一點，做法就是一樣用 CDK 建置 chart repository 專用的 repository or Bucket 之後，chart 再從 source control 出發開始跑 CICD 最後到 chart repository 裡，最後 CDK EKS helm chart 再從 chart repository 裡 pull 我們客製化過的 chart。

## AWS CDK Kubernetes resources management

AWS CDK EKS 在 Kubernet resource management 歷史比較久而且變化一直都蠻大的，最一開始只有用 `kubectl apply` 來實踐 Kubernetes resource 的創建或修改，對我而言其實算夠用了，畢竟大部分的 services 都是靠 Helm 管理，只有少部分的 resources 像是 PV、PVC 或 storageClass 需要額外管理。在最近的 CDK 版本中，多了一個新的 [CDK8s](https://github.com/awslabs/cdk8s) 的選擇之外，CDK EKS 本身也多了 query 的概念，像是在[官方文件](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_eks.README.html#id11)中的案例為抓取已經建立好的 LoadBalancer DNS name，我自己本身也是使用 LoadBalancer 想說希望有 query 的功能，但山不轉路轉，我後來是選擇使用 [external-dns](https://github.com/kubernetes-sigs/external-dns) 做為 service discovery 的工具。

## EKS ingress solution: Traefik (v1.7) + ALB

對於 ingress 的實踐相對複雜一點，原本是使用 Traefik (v1.7) + Network Load Balancer，但面對複雜的 routing 需求再加上想要與 AWS resources 有更好地整合，像是可以直接套用 [WAF](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/annotation/#wafv2)、[ACM](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/annotation/#ssl) 或 [Security Group](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/annotation/#access-control) 等等。後來有想導入 AWS ALB controller 來替 ingress 加上 ALB，但現實上遇到的問題就是走 microservices 要替每一個 ingress 建立一個 ALB 的話不太符合經濟效益。所以最後的做法是 Traefik (v1.7) + ALB，關於怎麼運作的流程圖：

![traefik-alb.png](/traefik-alb.png)

> 在公開的 [Helm charts stable](https://github.com/helm/charts/tree/master/stable/traefik) 裡，最多只有到 service，並沒有 ingress object，所以要能夠使用這個架構必須自己客製 chart。

流程大致如下：
1. 使用 [Helm charts incubator aws-alb-ingress-controller](https://github.com/helm/charts/tree/master/incubator/aws-alb-ingress-controller) 建置 ALB controller，並且額外建立 service account for IRSA
2. 使用客製的 Traefik chart，透過 ingress object 讓 ALB controller 知道要建立 Traefik 專屬的 ALB
3. 建立想要讓外面流量可以訪問的 application，透過 ingress object 讓 Traefik 知道要建立 routing rule
4. 當一般的 user 做訪問的時候會先從 ALB 進來到 Traefik service，再來是 Traefik pods，之後 Traefik 會把 application 的行為傳回 user

# Conclusion

* CDK package 每個成熟度不一，依照公司可以承擔的風險做選擇
* CDK EKS package 在 experiment 中，但依舊有很多好用的功能像是 IRSA、Helm 或進階的 Kubernetes resources management
* 自建的 chart repository 要避免落入雞生蛋蛋生雞的情境
* CDK EKS 對 Kubernetes resources 除了 create or modify 之外，新增了 query 的功能
* 設計系統的過程中要適時的山不轉路轉
* Traefik + ALB 可以讓 routing 做更多的變化，與 AWS resources 有更好地整合

# Reference

* [Introducing fine-grained IAM roles for service accounts](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
* [Backing services](https://12factor.net/backing-services)
* [Traefik Kubernetes User Guide](https://doc.traefik.io/traefik/v1.7/user-guide/kubernetes/) 
* [AWS ALB Ingress Controller¶](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/)
