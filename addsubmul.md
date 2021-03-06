#덧셈,뺄셈,곱셈
사칙연산은 기본이겠지요. add, sub, mul, imul, div, idiv, inc, dec, neg 명령들이 사칙연산과 관련있습니다.

```
org 100h
 
mov ax, 3
mov bx, 5
 
add ax, 1
sub bx, 1
add ax, bx
sub bx, ax
 
ret
```

add와 sub는 설명이 필요없겠지요?
다음은 mul, imul을 실험해보겠습니다.
``` 
org 100h
 
mov al, 90h
mov bl, 90h
mul bl
 
mov ax, 1000h
mov bx, 1000h
mul bx
 
mov ax, 20
mov bx, -20
imul bx
 
mov ax, 0ffffh
mov bx, 0ffffh
imul bx
 
mov ax, 0ffffh
mov bx, 0ffffh
mul bx
 
 
ret
```
먼저 알아야할 것이 있습니다. 하나의 레지스터의 크기는 16비트입니다. 그럼 레지스터끼리 곱했을 때 결과값인 몇 비트가 될까요?

100과 100을 곱하면 0이 2개 + 2개 해서 4개가 됩니다. 그럼 100h와 100h를 곱하면 어떻게 될까요? 마찬가지 입니다. 10000h가 됩니다. 즉 자리수가 두배가 되는 것이지요. 16비트끼리 곱하면 32비트가 됩니다. 무슨 말이냐면 16비트끼리 곱했을 때 32비트의 자리가 있어야 결과값을 저장할 수 있다는 것입니다. 안그러면 16비트끼리 곱하도록 허용할 필요가 없지요.

그래서 곱하기 연산에는 명령에는 안보이는 추가 레지스터를 사용합니다.

먼저 mul bl 이라는 명령은 실행해보면 bl이 8비트이므로 프로세서는 8비트끼리의 연산이라고 생각합니다. 그래서 16비트의 결과값을 저장하려고합니다. 곱하려는 수는 bl이고 곱해지는 수는 al입니다. al은 명령어에 없습니다. 그냥 프로세스가 무조건 곱하기에 al이 사용된다고 미리 정해버린겁니다. 그리고 결과값은 ax입니다. 16비트가 필요하므로 16비트 레지스터 ax 전체를 사용합니다. 90h와 90h를 곱하면 16진수로 5100입니다. 51이 ah에 00이 al에 쓰여있나요?

이전 글에서 ax 레지스터의 이름이 accumulate register 이고 계산에 사용되는 레지스터라고 소개했습니다. 이렇게 mul이나 div에서 ax 레지스터를 무조건 사용하기 때문에 계산에 특화된 레지스터라고 말씀드린 것입니다.

그 다음으로 mul bx 명령은 ax와 bx를 곱하는 명령입니다. ax와 bx를 곱하면 32비트의 결과값이 나오는데 ax는 16비트입니다. 나머지 16비트는 어디에 저장될까요? 흥미를 위해 답을 알려드리지 않겠습니다. 에물레이터로 실험해보면 어떤 레지스터에 100h값이 나타납니다. 1000h * 1000h는 1000000h입니다. 이 값을 16비트씩 끊어보면 상위 16비트는 100h이고 하위 16비트는 0000h입니다. 상위 16비트가 저장되는 레지스터에는 100h가 저장되고 하위 16비트가 저장되는 ax에 0000h가 저장됩니다.

설명이 좀 길고 주절주절했습니다만 mul명령에 대해 두가지만 기억하시면 됩니다. 8비트 연산을 때는 ax레지스터가 자동으로 사용된다는 것과 16비트 일때는 dx도 사용된다는 것입니다.

이제 imul을 알아보겠습니다. imul은 부호를 생각하는 곱셈입니다. mul은 부호를 고려하지않고 무조건 곱셈을 합니다. 하지만 imul은 부호를 고려합니다. 다시한번 말씀드리자면 가장 왼쪽 비트가 1이면 음수이고 0이면 양수입니다.

20과 -20을 곱해볼까요? 코드에는 -20이라고 써놨지만 컴퓨터는 -라는 부호를 모릅니다. 부호비트가 1인지만 생각합니다. 에물레이터에서 -20이 어떻게 나타나는지 보세요. 0ffech입니다. 20은 14h이지요. 0ffech + 14h는 0이니까 0ffech는 -20이 맞겠지요? imul bx의 결과값은 음수가 되어야합니다. 그리고 dx에도 결과값이 저장되야하지요. 그래서 dx 값이 뭔가요? 0ffffh이지요. 즉 결과값이 0fffffe70h이라는 것입니다. 부호비트가 1이지요. 따라서 음수라는 것입니다. 400이 16진수로 190h입니다. 0fffffe70h에 190h를 더해보세요. 100000000h이지요. 그런데 32비트라는 제약때문에 제일 왼쪽 1은 날아갑니다. 즉 0이되는 것이지요.그래서 0fffffe70h가 -400이 되는 것입니다.

0ffffh는 -1입니다. -1에 -1을 곱하면 1입니다. 1이 되나 해보세요. 만약에 0fffe0001라면 부호없는 곱하기가 되는 것이고 1이라면 부호있는 곱하기 입니다. dx 값이 0인지 0fffeh인지 확인해보시기 바랍니다.

마지막이 부호없는 0ffffh와 0ffffh의 곱셈입니다. -1과 -1의 곱셈과 다른지 확인해보세요.