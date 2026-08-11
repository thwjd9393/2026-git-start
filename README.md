# 2026-git-start
## 📌 오늘의 실습 주제

오늘은 하나의 GitHub 저장소를 두 명의 작업자가 함께 사용하는 상황을 만들어
Git의 협업 과정을 실습했다.

실제 작업자는 한 명이지만 같은 GitHub 저장소를 두 개의 폴더에 Clone하여
**작업자 A와 작업자 B가 협업하는 상황**을 재현했다.

### 실습에서 배운 주요 내용

- 하나의 GitHub 저장소를 여러 작업자가 사용하는 방법
- `git fetch`로 다른 작업자의 변경 사항 확인하기
- `git merge`로 다른 작업자의 변경 사항 합치기
- Push가 거절되는 이유 이해하기
- Merge Conflict가 발생하는 이유 이해하기
- 충돌을 직접 해결하고 다시 Push하기
- 다른 작업자가 최종 결과를 동기화하는 방법


---

## 🏗️ 실습 구조

```text
          ☁️ GitHub
         공용 저장소
          ↕      ↕
     fetch/push  fetch/push
          ↕      ↕
    💻 작업자 A   💻 작업자 B
    로컬 저장소   로컬 저장소

```

```mermaid
sequenceDiagram
    autonumber

    actor A as 👩 작업자 A
    participant A_PC as 💻 A의 컴퓨터
    participant GitHub as ☁️ GitHub<br/>공용 저장소
    participant B_PC as 💻 B의 컴퓨터
    actor B as 👨 작업자 B

    Note over A_PC,B_PC: 📌 같은 프로젝트를 각자의 컴퓨터에 따로 복사해서 작업한다

    %% 1. 정상적인 협업
    rect rgb(235, 248, 255)
        Note over A,GitHub: ① 서로 다른 작업을 할 때

        A->>A_PC: A 전용 문서 작성
        A_PC->>GitHub: 작업 내용 올리기 (Push)
        GitHub-->>A_PC: ✅ 저장 완료

        Note over GitHub,B_PC: B의 컴퓨터에는<br/>A의 변경이 자동으로 생기지 않음

        B->>B_PC: GitHub의 새 변경 확인 (Fetch)
        GitHub-->>B_PC: A의 새 작업 정보 전달

        B->>B_PC: 내 작업에 반영 (Merge)
        B_PC-->>B: ✅ A의 문서가 내 컴퓨터에도 반영됨

        B->>B_PC: B 전용 문서 작성
        B_PC->>GitHub: 작업 내용 올리기 (Push)
        GitHub-->>B_PC: ✅ 저장 완료

        A->>A_PC: B의 변경 가져오기
        GitHub-->>A_PC: B의 새 작업 전달
        A_PC-->>A: ✅ A와 B의 작업 모두 확인
    end

    Note over A_PC,B_PC: 서로 다른 파일을 수정하면<br/>대부분 자동으로 합칠 수 있다

    %% 2. 충돌 준비
    rect rgb(255, 250, 230)
        Note over A,B: ② 이번에는 둘이 같은 문장을 수정한다

        Note over A_PC,B_PC: 시작 전 두 컴퓨터의 README.md는 동일한 상태

        A->>A_PC: 같은 문장을<br/>"작업자 A의 Git 협업 실습"으로 수정
        A_PC->>GitHub: 먼저 올리기 (Push)
        GitHub-->>A_PC: ✅ A의 변경 저장

        B->>B_PC: A의 변경을 받지 않은 상태에서<br/>같은 문장을 다른 내용으로 수정
        B_PC->>GitHub: 내 변경 올리기 (Push)
        GitHub--xB_PC: ❌ 업로드 거절
    end

    Note over GitHub,B_PC: 💡 GitHub에는 이미 A의 새 작업이 있는데<br/>B의 컴퓨터에는 그 내용이 없기 때문

    %% 3. Fetch / Merge
    rect rgb(255, 240, 240)
        Note over B_PC,GitHub: ③ B가 A의 변경을 가져온다

        B->>B_PC: 최신 변경 확인 (Fetch)
        GitHub-->>B_PC: A의 변경 정보 전달

        B->>B_PC: A의 변경과 내 변경 합치기 (Merge)

        B_PC--xB: ⚠️ 충돌 발생!
        Note over B_PC: 둘이 README.md의<br/>같은 문장을 서로 다르게 수정함
    end

    %% 4. Conflict resolution
    rect rgb(245, 235, 255)
        Note over B,B_PC: ④ 사람이 직접 어떤 내용을 남길지 결정한다

        B->>B_PC: A의 내용과 B의 내용 비교

        Note over B_PC: A의 내용<br/>"작업자 A의 Git 협업 실습"<br/><br/>B의 내용<br/>"작업자 B의 Merge 충돌 실습"

        B->>B_PC: 두 사람의 의도를 합쳐 최종 문장 작성

        Note over B_PC: ✨ 최종 문장<br/>"작업자 A·B의 Git 협업 및<br/>Merge 충돌 해결"

        B->>B_PC: 충돌 해결 완료 표시 (Add)
        B->>B_PC: 합쳐진 결과 저장 (Commit)
        B_PC-->>B: ✅ Merge 완료
    end

    %% 5. Final sync
    rect rgb(235, 255, 235)
        Note over A,GitHub: ⑤ 해결된 결과를 모두가 공유한다

        B_PC->>GitHub: 해결 결과 올리기 (Push)
        GitHub-->>B_PC: ✅ 최종 결과 저장

        A->>A_PC: GitHub의 최신 결과 가져오기
        GitHub-->>A_PC: B가 해결한 최종 결과 전달

        A_PC-->>A: ✅ 최신 상태
        B_PC-->>B: ✅ 최신 상태

        Note over A_PC,B_PC: 🎉 A의 컴퓨터 = GitHub = B의 컴퓨터<br/>모두 같은 최신 상태
    end
```
