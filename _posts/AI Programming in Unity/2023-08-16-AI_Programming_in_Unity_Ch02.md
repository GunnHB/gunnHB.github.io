---
title:  "[유니티 게임 AI 프로그래밍] 002. 유한 상태 기계"
excerpt: "유한 상태 기계를 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-08-16
last_modified_at: 2023-08-16
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice--primary}

<div class="notice--info" markdown="1">
2장에서는 유니티를 이용하여 FSM 사용 방법을 다룰 예정입니다.

- 유니티의 상태 기계 기능 이해
- 자신만의 상태와 전이 만들기
- 예제를 사용해서 간단한 씬 만들기

우리의 게임에서 플레이어는 탱크를 조종할 수 있습니다. 적 탱크는 4개의 웨이포인트를 참조하면서
씬 위를 돌아다니다가 플레이어 탱크가 자신의 시야에 들어오면 추격을 시작합니다.
그리고 충분한 사거리 내로 진입하면 플레이어 탱크를 공격하게 만들 것입니다.

적 탱크의 인공지능을 구현하기 위해 FSM을 사용해 봅시다!
</div>