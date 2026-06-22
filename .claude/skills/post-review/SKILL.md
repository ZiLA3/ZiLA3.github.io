---
name: post-review
description: 다른 레포의 커밋에서 코드 파일의 전/후를 비교해 _posts/Programing/ 에 코드 리뷰 블로그 포스트를 작성합니다.
---

# /post-review

다른 레포의 특정 커밋에서 코드 파일의 Before/After를 가져와 블로그 포스트로 작성합니다.

## Usage

```
/post-review <repo-path> <commit-SHA> <code-file>
```

| 인자 | 설명 | 예시 |
|------|------|------|
| `<repo-path>` | 대상 레포 절대 경로 | `D:\Github\Minimi` |
| `<commit-SHA>` | 리팩토링 커밋 SHA (short/full 모두 가능) | `9b0965a` |
| `<code-file>` | 레포 루트 기준 파일 경로 | `Assets/Scripts/Data/Map/GridMap.cs` |

## Steps

### 1. Before / After 코드 가져오기

```bash
# Before (커밋 직전)
git -C <repo-path> show <commit-SHA>^:<code-file>

# After (커밋 시점)
git -C <repo-path> show <commit-SHA>:<code-file>
```

### 2. 포스트 파일명 결정

- 오늘 날짜: `YYYY-MM-DD`
- `<code-file>` 에서 경로와 확장자를 제거한 이름 → `FileName`  
  (예: `Assets/Scripts/Data/Map/GridMap.cs` → `GridMap`)
- 출력 경로: `_posts/Programing/{YYYY-MM-DD}-코드리뷰-{FileName}.md`

### 3. 언어 매핑

| 확장자 | 코드블록 언어 |
|--------|--------------|
| `.cs` | `csharp` |
| `.ts`, `.tsx` | `typescript` |
| `.js`, `.jsx` | `javascript` |
| `.py` | `python` |
| `.go` | `go` |
| `.java` | `java` |
| `.cpp`, `.cc` | `cpp` |
| `.rs` | `rust` |
| 그 외 | 확장자 그대로 |

### 4. 포스트 형식

```markdown
---
title: "코드 리뷰-{FileName}"
categories:
  - Programing
tags:
  - Review
  - Refactoring
---

<details>
<summary>Before</summary>

​```{language}
{before-code}
​```

</details>

<details>
<summary>After</summary>

​```{language}
{after-code}
​```

</details>
```

## Notes

- `_posts/Programing/` 에 같은 날짜·같은 파일명 포스트가 이미 있으면 덮어쓰기 전에 사용자에게 확인한다.
- Before/After 코드 사이에 빈 줄(`\n`)이 있어야 `<details>` 내부 마크다운이 렌더링된다.
