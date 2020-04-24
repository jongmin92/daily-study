# cut
파일 내용을 각 필드로 구분하고 필드별로 내용을 추출하여 각 필드들을 구분자로 구분할 수 있는 명령어이다. awk의 print $N 명령어 셋과 유사하나 제한 사항을 갖고 있으며, 스크립트를 작성할 경우 awk보다 더 간편하게 사용이 가능하다. 중요한 옵션으로는 -d(구분자)와 -f
(필드 지시자)가 있다. 파일의 각 라인에서 특정 부분을 제거하거나 추출한다.

```bash
$ cut [옵션] [파일명]
```

## 옵션
**-c, --characters 문자위치**  
: 잘라낼 곳의 글자 위치를 지정한다. 콤마를 사용하거나 하이픈을 사용하여 범위를 지정할 수도 있으며, 혼합해서 사용 가능하다.

**-f, --fields 필드**  
: 잘라낸 필드를 지정한다. 지정하는 방법은 -c 옵션과 같다.

**-d, --delimiter 구분자**  
: 필드를 구분하는 문자를 지정한다. 디폴트는 tab 문자이다.

**-s, --only-delimited**  
: 필드 구분자를 포함하지 않는 줄은 출력하지 않는다.

## 예제
```bash
$ cat cut_test.txt

1234
123 456 789             #공백 구분자
123     456     789     #TAB 구분자
abc def ghi             #공백 구분자
abc     def     ghi     #TAB 구분자
```

-c 옵션을 사용해서 1~3까지의 문자를 출력할 수 있다.
```bash
$ cut -c 1-3 cut_test.txt

123
123
123
abc
abc
```

-f 옵션을 사용하면 아래와 같은 결과를 얻는다.
```bash
$ cut -f 3 cut_test.txt     #파일에서 3번째 필드 출력

1234
123 456 789                 #탭 단위로 라인 전체 출력
789                         #3번째 탭인 '789' 출력
abc def ghi
ghi                         #3번째 탭인 'ghi' 출력
```

-d 옵션을 사용하면 아래와 같은 결과를 얻는다.
```bash
$ cut -f 2 -d 4 cut_test.txt    #필드 구분 문자 : 4, 2번째 필드 출력
56 789                          #'4' 이 후 필드 '56 789'만 출력
56      789                     #'4' 이후의 필드인 '56    789'만 출력
abc def ghi                     #구분자 '4'가 없어 전체 라인 출력
abc     def     ghi
```