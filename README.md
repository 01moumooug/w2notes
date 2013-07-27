w2notes
=======

마크다운으로 쓴 개인 노트를 정리하기 위해 만든 것입니다. 마크다운으로 작성한 문서를 간편하게 보고, 간단한 분류도 할 수 있도록 했습니다.

리눅스에서 Perl로 작성했고, Text::Markdown 모듈과 DBI모듈, DBD::SQLite 모듈이 있어야 합니다. 

    sudo cpan Text::Markdown
    sudo cpan DBI
    sudo cpan DBD::SQLite

데비안 계열 리눅스를 쓴다면 다음과 같이 모듈을 설치할 수도 있습니다.

    sudo apt-get install libtext-markdown-perl
    sudo apt-get install libdbd-sqlite3-perl

사용법
----

### 설정 파일 만들기 ###

샘플용 `NotesConfig.sample.pm`을 필요에 맞게 수정하고 파일 이름을 `NotesConfig.pm`으로 고칩니다.

### 헤더 ###

문서에 적힌 일종의 헤더를 바탕으로 문서를 분류합니다. 헤더는 문서 제일 처음에 나오고, `:`로 키와 값을 구분하며, 각 키-값 쌍('필드'라고 하겠습니다)이 한 줄에 하나씩 들어가며 빈 줄로 본문과 구분됩니다. 예를 들어 다음과 같이 쓰면

	Tags: perl, markdown
	Date: 2013-05-24 00:58:28
	Type: 메모
	Notes: 어쩌고 저쩌고

	이것은 예시고, 여기는 본문입니다. 위에는 헤더죠. 위에 적힌 것을 토대로 문서를 분류할 것입니다. 

`Tags:` 부터 처음 나오는 빈 줄까지 헤더가 됩니다. 키는 한 단어여야 하고, (아마도?) 라틴 문자와 숫자의 조합으로 써야합니다. 대소문자는 구분하지 않습니다. 키 다음에는 공백 없이 바로 `:`가 오며, 그 다음에는 값을 씁니다. 문서의 첫 줄이 이를 지키지 않으면 처음부터 본문이 시작되는 것으로 간주합니다.

각 필드는 마크다운에서 목록을 적는 것처럼 앞에 `-`나 `*`등을 붙일 수도 있습니다. 

	- Tags: perl, markdown
	- Date: ...

### 분류용 필드 ###

`NotesConfig.pm`에 보면 `idx_fields`라는 항목에, 어떤 필드를 분류에 사용할지 정하는 부분이 있을 것입니다. 기본으로는 `Tags`가 포함돼 있습니다. 

	'idx_fields' => [
		['tags'  ,'(none)'],
		['type'  ,'(none)'],
		['author',''],
		['status','']
	],

위 문서에는 헤더가 있고, 헤더에 `Tags` 필드가 있으며, 그 값은 `perl, markdown`입니다. 그러면 w2notes는 쉼표로 구분된 값을 각각 데이터베이스에 저장합니다. 각 값의 좌우 공백은 무시되며, `,`만 아니면 어느 문자든 상관 없고, 중복된 값은 무시될 것입니다. 

설정 파일을 인용한 곳에서, 분류용 필드 이름 옆에 적힌 것은 기본값입니다. 헤더에 해당 필드가 없는 경우 기본으로 붙일 값입니다. 물론 문서에 손을 대지는 않고, 데이터베이스에만 저장됩니다. 빈 문자열인 기본값은 저장되지 않습니다.

### 특수한 필드 ###

다음 필드는 특수한 목적이 있는 필드입니다. 이 필드를 분류용 필드로 지정할 경우 어떤 일이 벌어질지는 저도 잘 모릅니다.

#### `title` ####

문서의 제목으로 사용할 필드입니다. 이 필드가 없는 경우 파일 이름이 제목이 됩니다. 기본으로는 파일 이름에서 `_`와 확장자를 제거합니다. 파일 이름을 제목으로 바꾸는 과정은 설정 파일에서 변경할 수 있습니다.

#### `date` ####

날짜로 사용할 필드이며, 분류 정보를 색인할 때 이 필드가 없으면 현재 시간을 기준으로 문서에 추가됩니다. 헤더가 없으면 헤더를 만들어서 거기에 추가합니다. 

년, 월, 일, 시, 분, 초를 숫자가 아닌 문자로만 구분해주면 됩니다. 

	2000/5/4 18:08
	2013년 12월 3일
	...

#### `css` ####

문서를 보여줄 때 사용할 css 파일을 지정합니다. `template/view.template` 파일을 참고.

#### `js` ####

문서를 보여줄 때 사용할 자바스크립트 파일을 지정합니다.

### 서버 시작하기 ###

일단 w2notes 실행 파일이 있는 디렉토리를 `$PATH`에 추가하거나, `$PATH`에 포함된 디렉토리에 심볼릭 링크를 만들면 편하게 쓸 수 있을 것입니다.

서버를 시작하려면 w2notes를 실행하면 됩니다.

	$ w2notes start &

다음과 같이 치면 상태를 알 수 있습니다. 

	$ w2notes status
	The server seems running at 8888.

실행 중인 서버는 다음과 같이 종료할 수 있습니다.

	$ w2notes stop

분류용 필드를 색인하려면 다음과 같이 실행합니다. 

	$ w2notes update

위 명령을 실행하면 설정 파일의 `root` 항목에서 설정한 디렉토리 아래에 있고, 파일 이름이 `suffix`로 끝나는 모든 파일을 읽고 정보를 기록합니다. 서버를 실행할 경우 `update`는 자동으로 실행됩니다. 데이터베이스용 파일이 생성되지 않은 경우에는 `w2notes start`를 먼저 실행해야 `update`가 제대로 작동합니다. 서버가 꺼져있을 때에도 위 명령은 실행할 수 있습니다.

리눅스를 사용하는 경우 inotify를 이용해, 루트 디렉토리를 감시하면서 자동으로 업데이트를 하도록 할 수도 있습니다. 데비안의 경우 inotify-tools 패키지를 설치하고 다음과 같이 스크립트 작성하고 실행해볼 수 있을 것입니다.

	#!/bin/bash

	while true;do 
		inotifywait -rq -e create -e move -e delete -e modify [루트 디렉토리] 1>/dev/null && \
		w2notes update
	done;

### 연결된 문서 보기 ###

문서에서 참조식 링크를 쓰면 링크가 걸린 문서에서 어디에서 링크를 걸었는지 알 수 있습니다.

	[참조식][예시] 링크는 이런 [링크][]를 말합니다.

	[예시]: another_document.md
	[링크]: yet_another_document.md

### 기본 CSS 파일 ###

기본 CSS 파일 없이 웹 브라우저에서 문서를 보면 벌거벗은 HTML 문서가 보일 것입니다. style.css 파일을 `루트/.css/`에 두면 그나마 보기 좋을 것입니다(혹은 아닐수도?).

### 문서 편집 ###

문서 편집 기능은 따로 없습니다. 다만 설정 파일에서 설정한 임의 프로토콜과 편집용 프로그램을 연동하면 브라우저에서 바로 편집기를 띄우고 문서를 편집할 수 있을 것입니다. 제가 주로 사용하는 Iceweasel에서는 `about:config`에 들어가 다음과 같은 두 `boolean` 항목을 추가하고, 편집 링크를 클릭해서 이 프로젝트에 포함된 `edit.sample.pl` 파일을 선택했습니다.

	network.protocol-handler.expose.edit      false
	network.protocol-handler.external.edit    true

그리고 `NotesConfig.pm`에서 `edit_url`의 값을 `edit://%u`로 매겼습니다. 

이렇게 하면 가령 `testfile.md` 파일을 볼 때 `edit://testfile.md`와 같은 링크가 생성되고, 이 링크를 클릭하면 Iceweasel이 edit라는 프로토콜을 `edit.sample.pl`을 실행해서 처리하며, `edit.sample.pl`는 제가 간단한 글을 쓸 때 사용하는 에디터인 leafpad를 실행해서 해당 파일을 엽니다.

### Windows에서 사용하기 ###

Windows에서 사용할 경우 `w2notes` 파일을 열어서 24번째 줄의 `Server.pm`을 `SimpleServer.pm`으로 바꿉니다. 파일 이름을 utf-8로 인코딩하지 않는 환경일 경우 설정 파일에서 코드 페이지를 지정해주셔야 합니다(한글 Window의 경우 `cp949`).