# Linux/Unix에서 한 번에 여러 번 Command 실행 하기

## Syntax
```bash
## run "date" command 10 times
for i in {1..10};
do
    date;
done
```

다음과 같이 c 스타일처럼 작성할 수도 있다.
```bash
for ((n=0; i<10; n++))
do
    date
done
```

## Using while loop
```bash
END=5
x=$END
while [ $x -gt 0 ];
do
    date
    x=$(($x-1))
done
```

## repeat for zsh
```bash
$ repeat 10 { date }
```

## seq command
> seq LAST  
seq FIRST LAST  
seq FIRST INCREMENT LASE  
seq LAST | xargs command  
seq FIRST LAST | xargs command  
seq FIRST INCREMENT LASE | xargs command

```bash
$ seq 1 5
1
2
3
4
5
```

xargs command와 함께 사용 가능하다.
```bash
$ seq 1 5 | xargs -I{} date
```

여러 argument와 함께 사용 가능하다.
```bash
$ seq 1 5 | xargs -I{} sh -c "date && sleep 1"
```
