---
title: "名词解释"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2024-07-30T19:28:12+08:00
---

## CI/CD

- CI 全名(Continuous Integration)持续集成，
- CD 全名是(Continuous Deployment)，是持续部署。CD 还有个小号，叫持续交付，英文全称是 Continuous delivery，缩写也是 CD。

现在很多公司都有做持续集成，Jenkins 就是一个持续集成的工具，开源的，基于 JAVA 语言的。

CI 是持续集成，意思是让多个开发者，能够共同在同个代码库中开发不同的新功能，然后能够持续整合到某个分支上面。之所要持续做这件事，是因为开发不同功能时，代码可能会有冲突，而持续地整合就能及早化解冲突。在 CI 的流程中，也会有自动化测试与建构，来避免等要上线前一次合并才发现有测试跑不过，或者建构有错误。

CD 则是持续交付/部署，在整合完后有自动化的部署流程。最理想的状况，是当开发者把代码合并后，就会开始整合、测试，最终部署，整个流程不需用有人工介入，一切自动化完成，新的功能就会到最端使用者手上。

{{< figure src="/cicd_1.png" width="80%" >}}

{{< notice tip "补充说明: CI/CD 与 DevOps 的关系？">}}

可能有读者会问 CI/CD 与 DevOps 的关系。在爬梳这段历史时，看到 DevOps 约是 2008 - 2009 年左右被提出；而 CI/CD 则是更早之前就有，只是后来才逐渐在业界普及。目前 CI/CD 是 DevOps 中的重要元素。可以在 Atlassian 这篇介绍看到，建置 CI/CD 流程是 DevOps 工程师的重要能力之一 ，但要成为 DevOps 工程师，还需要有其他技能。
{{< /notice >}}

## Footprint

程序波及的范围，这个范围可以是多个层面的，比如说内存访问的局部性强，也就是footprint小。

## Banner

Banner 表示旗帜或者标志物。
