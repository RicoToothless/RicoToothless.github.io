+++
author = "Rico"
title = "AWS CDK cross-reference"
date = "2021-05-23"
description = "Several ways to cross-reference in AWS CDK"
tags = [
    "aws",
    "cdk",
    "typescript",
]
categories = [
    "tech",
]
archives = "2021-05"
+++

## Why we need cross-reference in AWS CDK?

AWS CDK 是採用 CloudFormation 為底層的工具，可以因應不同的 project、environment、organization 或 resources 做 [apps](https://docs.aws.amazon.com/cdk/latest/guide/apps.html) 或 [stacks](https://docs.aws.amazon.com/cdk/latest/guide/stacks.html) 切分。

最理想的情況下，各個 `apps` 或 `stacks` 之間沒有 dependency 當然是最好的，但最常見的情況是很多不同的服務要跑在同一個 VPC 上面，這個時候就不得不做 cross apps/stacks reference。

處理 cross-reference 的難易度取決於：
1. 究竟是 `apps`, `stacks` or `constructs` reference
2. cross `stacks` `constructs` reference 兩個 `constructs` 是否一個 higher-level 一個 lower-level（這也是我噩夢的開始，促成本篇文章的起源）
3. 是否同個 region or AWS account

目前依據自己的使用心得大致歸納出這四種 reference 的方法，也是一路走來慢慢演進的做法：
1. CDK dynamic reference
2. CloudFormation export value reference
3. SSM parameter reference
4. Nested stacks reference（沒有研究，但聽說問題蠻多的，不贅述）

## What happens in my cross-reference journey?

當一開始接觸 CDK 時，沒有意外都會先按照範例去描述我們的 infrastructure 要長怎樣，大部分 CDK `constructs` 都是別人封裝好的，也就是所謂的 higher-level `constructs`，使用時不會遇到什麼困難。

基本上只要在同一個 `stack` 做 constructs reference 都不會有太大的問題，再加上同個 `stack` 一定是同個 region & AWS account，是最理想的狀況。

如果要 cross `stacks` reference 就取決於哪種類型的 `constructs`，如果兩邊都是 higher-level `constructs` 且同個 region & AWS account 就不用煩惱太多，很多[範例](https://docs.aws.amazon.com/cdk/latest/guide/resources.html#resource_stack)可以參考。

**當開始使用 lower-level `constructs` 時，問題就來了。**

之所以 lower-level `constructs` 會遇到挑戰來自於它只能 consume 固定的 resources `srting` id，不像 higher-level `constructs` 都用 `interface`，必須要用傳統 CloudFormation `Output` & `Fn::ImportValue` 的方式做 cross `stacks` reference。

CloudFormation 優缺點很明顯，雖然它讓整個 Infrastructure 有強一致性，但 `Output` & `Fn::ImportValue` 的方式讓兩個 `stacks` 有 deadlock 的問題，可以參考[這篇](https://www.endoflineblog.com/cdk-tips-03-how-to-unblock-cross-stack-references)。

但值得開心的是，同一個 `stack` 即使 `constructs` 是 lower-level 也都很好做 reference。

**當需求一多，問題也來了**

不同 `apps` reference 最常見的情況是在不同 CDK 專案之間要 reference 既有的 resources，的確可以很直覺的 [import exist resources](https://docs.aws.amazon.com/cdk/latest/guide/resources.html#resources_importing) 的方式來解決，但是當 CDK 專案一多的時候，很難管理專案之間的關係。

不同 region 很難做 reference 主要來自 CloudFormation 先天上的瓶頸，CloudFormation 本身就是 region 級別的服務，但實務上就是有這些 cross region 的需求（像是 [Route53 CMKs for DNSSEC](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec-cmk-requirements.html)）。

不同 AWS account 除了 CloudFormation 先天上的瓶頸之外，也要做 multi-account 的管理，無疑讓管理的成本墊高許多。（CDK multi-account management 基本上又可以再寫一篇了。）

## CDK dynamic reference

這個方式最為大宗，很多教學跟範例可參考，同個 `app`、同個 region、同個 AWS account 和沒有使用到 lower-level `constructs` 都不會有問題。

值得一提的是 [import exist resources](https://docs.aws.amazon.com/cdk/latest/guide/resources.html#resources_importing)，除了可以直接以 ARN 作為依據，例如 `s3.Bucket.fromArn`；或者以名字作為依據，例如像 `s3.Bucket.fromBucketName` reference physical names。

也可以做到邏輯抽象的 reference 如下。

```typescript
ec2.Vpc.fromLookup(this, 'DefaultVpc', {
    isDefault: true
});

ec2.Vpc.fromLookup(this, 'PublicVpc', 
    {tags: {'aws-cdk:subnet-type': "Public"}});
```

這邊寫說我要 reference VPC 是否為已經存在的 default VPC 或者已經存在的 VPC 上是否有特定的 tag 為 `'aws-cdk:subnet-type': "Public"`。

## CloudFormation export value reference

如前面所述，lower-level `constructs` 會遇到挑戰來自於它只能 consume 固定的 resources `srting` id，producing 與 consuming `stacks` 之間是靠 CloudFormation `Output` & `Fn::ImportValue` 實現，更多討論可以參考[這篇](https://github.com/aws/aws-cdk/issues/603)。

Producing `stack` `Output` values 之後可以在 AWS CloudFormation console 上看到，可參考[這篇](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html)，確認 CloudFormation 有預期的 values 之後就可以在 consuming `stack` 做 `Fn::ImportValue` 的動作了。

跟 CDK dynamic reference 相比之下，其實只有解決 lower-level `constructs` 的問題，不同 `app`、region 和 AWS account 的問題並沒有解決。

## SSM parameter reference

此方法可以一舉解決 lower-level `constructs`、不同 `app`、不同 region 和不同 AWS account 的問題。

追根究底是因為 CDK 社群對於 [cross apps](https://github.com/aws/aws-cdk/issues/1095) 跟 [cross region/account](https://github.com/aws/aws-cdk/issues/49) 並沒有最佳解的作法而催生出 SSM parameter reference。

SSM parameter reference 作法很單純：
1. producing `stack` 部署之後才會產生的 resource id
2. 把 resource id 寫到 SSM parameter 上
3. consuming `stack` 再去 SSM parameter 上抓取 resource id

在 producing 與 consuming `stacks` 之間以 SSM parameter 做設定的確可以達到很好的解藕，但維護這些 SSM parameters 的確蠻手動的，而且沒有意外的話，維護這些 SSM parameters 的專案是獨立出來的。

**所以很有可能只是單純新增一對相應的 cross-reference resources，卻必須要在多個專案之間穿梭**。

因為 SSM parameter 也是 region 級別的服務，所以建議在設定 SSM parameters 時把 AWS account & region 的資訊帶入 parameter name，例如 `/123456777/us-east-1/SomeResourceId`，這樣在跨 AWS account 或 region 的時候比較不會混肴。

## Conclusion

目前即使同個 region、同個 AWS account、同個 `app` 在**不同** `stacks` 之下，我也會選擇使用 SSM parameter reference 為了讓保持所有 CDK 專案的一致性。

雖然 SSM parameter reference 不是官方最佳作法（因為根本沒有共識），但在 CDK 社群有不少人使用，也是我在茫茫的 documents、issues 以及 community 大海中，找出還算漂亮的解法。

希望看完文章的你可以少走一點彎路。

## Reference

* [Best practices for cross-stack CDK to CFN and vice versa #603](https://github.com/aws/aws-cdk/issues/603)
* [Exporting/importing across CDK apps #1095](https://github.com/aws/aws-cdk/issues/1095)
* [Cross-region/account references #49](https://github.com/aws/aws-cdk/issues/49)
* [Exporting/importing across CDK apps #1095](https://github.com/aws/aws-cdk/issues/1095)
* [Docs lack examples or explanation of how to export and import values between stacks #10480](https://github.com/aws/aws-cdk/issues/10480)
* [AWS CDK document - Resources](https://docs.aws.amazon.com/cdk/latest/guide/resources.html)
* [Exporting stack output values](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html)
* [Walkthrough: Refer to resource outputs in another AWS CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-crossstackref.html)
* [Enabling DNSSEC on AWS Route 53 using CloudFormation.](https://jonathanj.nl/2021/02/03/aws-route53-dnssec-cloudformation.html)
* [Working with customer managed CMKs for DNSSEC](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec-cmk-requirements.html)
* [CDK tips, part 3 – how to unblock cross-stack references](https://www.endoflineblog.com/cdk-tips-03-how-to-unblock-cross-stack-references)
