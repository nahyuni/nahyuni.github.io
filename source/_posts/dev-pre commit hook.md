---
title: [Subversion pre-commit hook]
date: 2020-02-06 14:30:19
tags: [svn]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---

Subversion은 Repository Hook이라는 메커니즘을 가지고 있다.

pre-commit hook을 사용해, commit하는 내용이나 사용자에 따라, commit할지 말지 여부를 결정하거나, 내용을 수정할 수 있으며, post-commit hook을 사용하면, 모든 commit에 대해 메일을 보내거나 통계를 업데이트할 수 있다.

pre-commit hook을 등록 할 때는 다음 세 가지를 신경 쓰면 된다.
<!-- more -->
실행 가능: Subversion repository의 hooks/pre-commit 파일을 액세스하는 프로그램 (e.g. 웹 서버)이 실행 권한을 가질 수 있도록 해야 한다.
exit code: exit code에 따라 commit 여부가 결정된다. 0이라면 commit 성공, 1이라면 commit 실패.
stderr: 표준 에러로 출력한 메시지가 클라이언트의 에러 메시지로 출력된다.
예를 들어, 공백만 있는 commit log와, 특정 author에 의한 commit을 막으려면 다음과 같은 pre-commit hook 스크립트를 사용하면 된다

```bash
#!/bin/sh
REPOS="$1"
TXN="$2"
SVNLOOK=/usr/bin/svnlook
$SVNLOOK log -t "$TXN" "$REPOS" | grep "[^[:space:]]" > /dev/null
if [ $? -ne 0 ]; then
echo -n 'EMPTY commit log is NOT ALLOWED' 1>&2
exit 1
fi
$SVNLOOK author -t "$TXN" "$REPOS" | grep -v -e "^public$" > /dev/null
if [ $? -ne 0 ]; then
echo -n 'USER public is NOT ALLOWED' 1>&2
exit 1
fi
exit 0
```
이러한 hook들은 shell script로만 한정되는 것은 아니므로, python이나 외부 프로그램을 실행하는 스크립트 등의 방식을 활용할 수 있다.

Subversion repository에 강제하고 싶은 규칙이 있다면, Subversion의 pre-commit hook을 활용해 보자.

참조
http://blog.lastmind.io/archives/592**