# 실무에서 유용한 Git 꿀팁

## fit-flow

[git-flow](https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html)

- 브랜치 전략이자 이 전략을 사용하기 위한 별도의 기능을 제공한다.
- 팀마다 적절한 브랜치 전략을 세운다.

## PR로 merge 하기

[PR의 3가지 merge 방법](https://meetup.toast.com/posts/122)

## 마지막 commit 수정하기

- amend
- 서버에 push 한 상태에서 마지막 커밋을 수정하려면?
    - 우선 마지막 커밋을 수정한다.
    - 절대 브랜치를 바꾸지 말고 그 상태에서 git push -f를 입력한다.

## 원하는 commit만 가져와 현재 branch에 붙이기

- cherry-pick