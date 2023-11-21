---
title:  "[유니티 게임 AI 프로그래밍] 007. 퍼지 로직을 사용한 인공지능 개선"
excerpt: "퍼지 로직을 사용하는 방법에 대해 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-11-21
last_modified_at: 2023-11-21
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
퍼지 로직을 활용하면 좀 더 섬세한 방식으로 게임의 규칙을 표현할 수 있습니다.
기본적으로 퍼지 로직은 수학과 매우 관련 있습니다. 물론 해당 포스트에서 다루는
내용은 한계가 있습니다. 이 글을 보고 계시는 분 중 수학에 관심과 조예가 깊은 
분이시라면 다양한 형태로의 응용을 찾아보시는 것을 추천드립니다.

- 퍼지 로직의 정의
- 퍼지 로직은 주로 어디에 사용하는가
- 퍼지 로직 컨트롤러 구현 방법
- 퍼지 로직 개념을 사용하는  다양한 형태
</div>

## 퍼지 로직 정의
