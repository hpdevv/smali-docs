# Документация SMALI на русском

Небольшая помощь в Smali

 # 
Общая информация
 # 
Smali
Виды(Types)
Байт-код Dalvik имеет два основных класса типов, примитивные типы и ссылочные типы. Типы ссылок - это объекты и массивы, все остальное является примитивным.
Примитивы представлены одной буквой.

V - Void - может использоваться только для типов возврата

Z - Boolean (логический)

B - Byte (байт)

S - Short (короткий)

C - Char

I - Integer (Целое число)

J - Long (64 bits) (Длинный)

F - Float (плавающий)

D - Double (64 bits) (Двойной )

Объекты принимают форму Lpackage/name/ObjectName; - где ведущий "L" указывает, что это тип объекта, package/name/ - это пакет, в котором находится объект, ObjectName - это имя объекта и ";" обозначает конец имени объекта. Это будет эквивалентно имени package.name.ObjectName в java. Или для более конкретного примера, Ljava/lang/String; эквивалентно java.lang.String
Массивы принимают форму [I - это будет массив целых чисел с одним измерением. т.е. int[] в Java. Для массивов с несколькими измерениями вы просто добавляете больше "[" символов. [[I = int[][], [[[I = int[][][] и т.д. (Примечание: максимальное количество измерений, которое вы можете иметь, составляет 255).
Вы также можете иметь массивы объектов, [Ljava/lang/String; будет массив строк.

Методы(Methods)
Методы всегда указываются в очень подробной форме, которая включает тип, содержащий метод, имя метода, типы параметров и тип возвращаемого значения. Вся эта информация необходима, чтобы виртуальная машина могла находить правильный метод и иметь возможность выполнять статический анализ на байт-коде 
Они принимают форму Lpackage/name/ObjectName;->MethodName(III)Z
В этом примере вы должны распознать Lpackage/name/ObjectName; как тип. MethodName - это имя метода. (III)Z является сигнатурой метода. III - параметры (в данном случае 3 целых числа), а Z - тип возврата (bool).
Параметры метода перечислены один за другим, без разделителей между ними.
Вот более сложный пример:
Lpackage/name/ObjectName;->MethodName(I[[IILjava/lang/String;[Ljava/lang/Object;)Ljava/lang/String;
В Java, это было бы
String MethodName(int, int[][], int, String, Object[])

Поля(Fields)
Поля также всегда указывается в многословной форме, которая включает тип, содержащее поле, имя поля и тип поля. Опять же, это позволяет виртуальной машине находить правильное поле, а также выполнять статический анализ на байт-коде.
Они принимают форму Lpackage/name/ObjectName;->FieldName:Ljava/lang/String;
Это должно быть довольно очевидно - это пакет и имя объекта, имя поля и тип поля соответственно
 # 
Регистры (Register)
Вступление
В байт-коделе dalvik регистры всегда 32 бита и могут содержать любой тип значения. 2 регистров используются для хранения 64-битных типов (длинный - Long и двойной - Double).

Указание количества регистров в методе
Существует два способа указать, сколько регистров доступно в методе. директива .registers указует общее количество регистров в методе, в то время как альтернативное .locals директива определяет число регистров без параметров в методе. Общее количество регистров будет также включать в себя, однако многие регистры необходимы для хранения параметров метода.

Как параметры метода передаются в метод
Когда метод вызывается, параметры метода помещаются в последние n регистров. Если метод имеет 2 аргумента и 5 регистров (v0-v4), аргументы будут помещены в последние 2 регистра - v3 и v4.
Первым параметром для нестатических методов(non-static methods) всегда является объект, на который вызывается метод (this объект)
Например, допустим, вы пишете нестатический метод LMyObject;->callMe(II)V. Этот метод имеет 2 целочисленных(integer) параметра, но также имеет неявный LMyObject;параметр перед обоими целыми параметрами, поэтому для метода существует всего 3 аргумента.
Предположим, вы указали, что в методе (v0-v4) есть 5 регистров, либо с директивой .registers 5, либо с директивой .locals 2 (т.е. 2 local registers + 3 parameter registers). Когда метод вызывается, объект, к которому выполняется метод (т.е. this ссылка), будет в v2, первый целочисленный(integer) параметр будет в v3, а второй целочисленный(integer) параметр будет в v4.
Для статических методов (static methods) это одно и то же, за исключением того, что неявный этот аргумент.

Регистрация имен (Register names)
Существует две схемы именования для регистров - обычная схема именования v# и схема именования p# для регистров параметров. Первый регистр в схеме именования p#является первым регистром параметров в методе. Итак, вернемся к предыдущему примеру метода с 3-мя аргументами и 5-ю полными регистрами. В следующей таблице показано обычное имя v# для каждого регистра, за которым следует имя p# для регистров параметров(parameter registers)

v0 Первый локальный регистр

v1 Второй локальный регистр

v2 p0 Первый регистр параметров

v3 p1 Второй регистр параметров

v4 p2 Третий регистр параметров
Вы можете ссылаться на регистры параметров по имени - это не имеет никакого значения.

Введения регистров параметров(parameter registers)
p# схема именования была введена в качестве практического вопроса
Скажем, у вас есть существующий метод с несколькими параметрами, и вы добавляете некоторый код в этот метод, и вы обнаружите, что вам нужен дополнительный регистр. Вы думаете: «Ничего страшного, я просто увеличу количество регистров, указанных в директиве .registers!».
К сожалению, это не так просто. Имейте в виду, что параметры метода хранятся в последних регистрах в методе. Если вы увеличиваете количество регистров - вы меняете, какие регистры вводят аргументы метода. Поэтому вам придется изменить директиву .registers и перенумеровать каждый регистр параметров.
Но если схема именования p# использовалась для ссылки на регистры параметров во всем методе, вы можете легко изменить количество регистров в методе, не беспокоясь о перонумеровании любых существующих регистров.

Длинные/двойные значения(Long/Double values)
Как упоминалось ранее, длинные и двойные примитивы (J и D соответственно) имеют 64-битные значения и требуют 2 регистра. Это важно иметь в виду, когда вы ссылаетесь на аргументы метода. Например, предположим, что у вас есть (нестатический - non-static) метод LMyObject;->MyMethod(IJZ)V. Параметрами метода являются LMyObject;,int,long,bool. Таким образом, для всех его параметров потребуется 5 регистров.

p0 this

p1 I

p2, p3 J

p4 Z

Кроме того, когда вы вызываете метод позже, вам нужно указать оба регистра для любых аргументов с двойным расширением в списке регистров для инструкции типа invoke.
 # 
array (массивы)
array-length vA, vB
A: Регистр назначения (4 бита)
B: Массив reference-bearing регистр (4 бит)
Сохраняет длину (количество записей) указанного массива vB в vA

fill-array-data vA+, :target
A: Регистрация пары(pair), содержащей ссылку на массив
B: Целевая метка, определяющая таблицу данных массива
Заполняет заданный массив vA+ указанными данными в цели(target). Ссылка должна быть в массиве примитивов, и таблица данных должна соответствовать ей по типу и размеру. Ширина массива определяется в таблице.
Регистровые пары занимают vX и vX+1. например, v1, v2.
Пример таблицы данных:
:target
.array-data 0x2
0x01 0x02
0x03 0x04
.end array-data

new-array vA+, vB, Lclass;->type
A: Регистр назначения (8 бит)
B: Регистр размеров
C: Ссылка на тип
Создает новый массив указанного типа и размера. Тип должен быть типом массива.

filled-new-array { vA [ vB, v.., vX ]}, Lclass;->type
vA-vX: Регистры аргументов (по 4 бита)
B: Ссылка на тип
Создает новый массив указанного типа и размера. Тип должен быть типом массива. Ссылка на вновь сгенерированный массив может быть получена командой move-result-object, непосредственно после команды fill-new-array.

filled-new-array/range { vA .. vX }, Lclass;->type
vA .. vX: Диапазон регистров, содержащих параметры массива (по 4 бита)
B: Ссылка на тип (16 бит)
Создает новый массив указанного типа. Тип должен быть типом массива. Ссылка на вновь созданный массив может быть получена командой move-result-object, непосредственно после команды fill-new-array/range.
 # 
array accessors (приемники массивов)
Легенда:
A(aget): Регистр назначения
A(aput): Исходный регистр
B: Ссылка на массив
C: Индекс в массиве
aget vA, vB, vC

Получает целое(integer) значение с индексом vC из массива, на который ссылается vB, и сохраняет в vA

aput vA, vB, vC

Сохраняет целое(integer) значение от vA в массиве, на который ссылается vB на индекс vC
Также есть другие aget/aput, путем добавления окончания меняется тип значения. Например: aget-objec(Получает объект(object)).
-boolean

-byte

-char

-object

-short

-wide
 # 
comparison
Легенда:
A: Регистр назначения
B: Первый регистр источника
C: Второй регистр источника
B+: Первая пара регистров источника (pair)
C+: Вторая пара регистров источника (pair)
cmp-long vA, vB+, vC+
Сравнивает длинные(long) значения в исходных регистрах, сохраняя 0;
Если vB+ == vC+ то сохраняет 1;
Если vB+ < vC+ или vB+ > vC+ то сохраняет -1.

cmpg-double vA, vB+, vC+
Сравнивает двойные(double) значения в исходных регистрах, сохраняя 0;
Если vB+ == vC+ то сохраняет 1;
Если vB+ < vC+ или vB+ > vC+ то сохраняет -1.
Если vB+ или vC+ не являются числом, возвращается 1.

cmpg-float vA, vB, vC
Сравнивает значения с плавающей(float) запятой в исходных регистрах, сохраняя 0;
Если vB == vC то сохраняет 1;
Если vB < vC или vB > vC то сохраняет -1.
Если vB или vC не являются числом, возвращается 1.

cmpl-double vA, vB+, vC+
Сравнивает двойные(double) значения в исходных регистрах, сохраняя 0;
Если vB+ == vC+ то сохраняет 1;
Если vB+ < vC+ или vB+ > vC+ то сохраняет -1.
Если либо vB+, либо vC+ не являются числом, возвращается -1.

cmpl-float vA, vB, vC
Выполняет указанное сравнение с плавающей(float) запятой, сохраняя 0;
Если vB == vC то сохраняет 1;
Если vB < vC или vB > vC то сохраняет -1.
Если vB или vC не являются числом, возвращается -1.
 # 
const
const vAA, #+BBBBBBBB
A: Регистр назначения (8 бит)
B: 32-разрядное постоянное целое число со знаком
Переместит заданное постоянное целочисленное(integer) значение в указанный регистр vAA.

const/16 vAA, #+BBBB
A: Регистр назначения (8 бит)
B: Целое число(integer) (16 бит)
Заносит #+BBBB в регистр vAA 

const/4 vA, #+B
A: Регистр назначения (4 бит)
B: Целое число (4 бит)
Помещает заданную 4-битную целую константу в регистр назначения vA.

const/high16 vAA, #+BBBB
A: Регистр назначения (8 бит)
B: Целое число (16 бит)
Поместит 16-битную константу в самые верхние биты регистра vAA. Используется для инициализации значений float.

const-class vAA, Lclass
A: Регистр назначения (8 бит)
class: Ссылка на класс
Переместит ссылку на класс(class), указанный в регистре назначения vAA. В случае, когда указанный тип является примитивным, это будет хранить ссылку на особый класс примитивного типа.

const-string vAA, "BBBB"
A: Регистр назначения (8 бит)
B: Строковое значение (string)
Переместит ссылку на строку, указанную в регистре назначения vAA

const-string/jumbo vAA, "BBBBBBBB"
A: Регистр назначения (8 бит)
B: Строковое значение (string)
Переместит ссылку на строку, указанную в регистре назначения vAA
jumbo - обозначает, что значение будет "большим"

const-wide/16 vA+, #+BBBB
#Пока пусто

const-wide/high16 vA+, #+BBBB
#Пока пусто

const-wide vA+, #+BBBBBBBBBBBBBBBB
#Пока пусто
 # 
goto
goto - Безусловный переход к :target.
goto :target

goto/16 :target #16bit

goto/32 :target #32bit
Примечание: goto буквально использует +/- смещения от текущей команды. APKTool преобразует их в метки для удобства чтения. Если внутри кода, для смещения требуется 16-битное значение, следует использовать goto/16, или для 32-битного значение, следует использовать goto/32. Почти невозможно определить, требуется ли goto/16 или goto/32 при добавлении новой команды(если не знаешь наверняка). Если вы точно не знаете какой бит, goto/16 может заменить любой goto, а goto/32 может заменить любой goto/16 или goto.
Только замена не может производиться на оборот: goto не может заменить goto/16, а он в свою очередь не может заменить goto/32.
 # 
if
Легенда:
A: Первый регистр для проверки (integer)
B: Второй регистр для проверки (integer)
target: Целевая метка
Примечание: != неравно
if-eq vA, vB, :target
Выполнение переходит к :target если vA == vB

if-eqz vA, :target
:target если vA == 0

if-ge vA, vB, :target
:target если vA >= vB

if-gez vA, :target
:target если vA >= 0

if-gt vA, vB, :target
:target если vA > vB

if-gtz vA, :target
:target если vA > 0

if-le vA, vB, :target
:target если vA <= vB

if-lez vA, :target
:target если vA <= 0

if-lt vA, vB, :target
:target если vA < vB

if-ltz vA, :target
:target если vA < 0

if-ne vA, vB, :target
:target если vA != vB

if-nez vA, :target
:target если vA != 0
 # 
invoke
Легенда:
vA-vX: Аргументы, передаваемые методу
class: Имя класса, содержащего метод
method: Имя метода для вызова
R : Тип возврата.
invoke-direct { vA, v.., vX }, Lclass;->method()R
Вызывает нестатический(non-static) direct метод (то есть метод экземпляра, который по своей природе не переопределяется, а именно либо метод private instance, либо конструктор).

invoke-interface { vA, v.., vX }, Lclass;->method()R
Вызывает метод интерфейса(interface method) (то есть объект, чей конкретный класс неизвестен, используя метод, который ссылается на интерфейс).

invoke-static { vA, v.., vX }, Lclass;->method()R
Вызывает статический метод(static method) (который всегда считается прямым методом).

invoke-super { vA, v.., vX }, Lclass;->method()R
Вызывает виртуальный метод(virtual method) непосредственного родительского класса.

invoke-virtual { vA, v.., vX }, Lclass;->method()R
Вызывает виртуальный метод(virtual method) (метод, который не является статическим или окончательным, и не является конструктором).
Примечание:
Если метод возвращает значение (R не является «V» для Void), он должен быть зафиксирован в следующей строке одним из операторов move-result или он будет потерян.

Так-же можно не перечислять все аргументы vA-vX, а сделать диапазон аргументов(Range of arguments) путем добавления окончание /range. Например: invoke-direct/range { vA .. vX }, Lclass;->method()R И так можно делать с любым выше перечисленным invoke.
invoke-direct { v1, v2, v3 } тоже самое что и invoke-direct/range { v1 .. v3 }
invoke-direct { v0 } тоже самое что и invoke-direct/range { v0 .. v0 }

Часто приводит к ошибкам использование invoke-virtual{ vX } в место invoke-virtual/range{ vX .. vX } в методах с большим количеством локальных регистров( v1, v2, v22 )
 # 
misc/разное
check-cast vAA, Lclass
A: Референтный регистр (8 bits)
B: Ссылка на тип (16 bits)
Проверяет, может ли ссылка на объект в vAA быть передана экземпляру типа, на который ссылается class(класс).
Выдает ClassCastException, если это невозможен, продолжается выполнение иначе.

instance-of vA, vB, Lclass
A: Регистр назначения (4 bits)
B: Референтный регистр (4 bits)
C: Ссылка на класс (16 bits)
#Описания пока нету

new-instance vAA, Lclass
A: Регистр назначения (8 bits)
B: Ссылка на тип
Создает объект class(класса) типа и помещает ссылку на вновь созданный экземпляр в vAA.
Тип должен относиться к классу(class) non-array.

nop
Пустая команда/Нет операции

throw vAA
A: Exception-bearing register (8 bits)
Выбрасывает указанное исключение. Ссылка на объект(object) исключений находится в vAA.
 # 
move
Легенда:
A: Регистр назначения (4, 8, 16 bits)
B: Исходный регистр (4, 16 bits)
#A: x bits. B: x bits не является частью кода. Добавил только для обозначения бит в регистрах
move vA, vB #A: 4 bits. B: 4 bits
Перемещает содержимого одного регистра не-объекта(non-object) к другому.

move/16 vAAAA, vBBBB #A: 16 bits. B: 16 bits
Делает тоже самое, что и move. Только исходный регистр и регистр назначения 16 bits

move/from16 vAA, vBBBB #A: 8 bits. B: 16 bits
Делает тоже самое, что и move/16. Только регистр назначения 8 bits

move-exception vAA #A: 8 bits
Сохраняет только что пойманное исключение в vAA. Это должна быть первая инструкция любого обработчика исключений, чье исключение исключений не должно игнорироваться, и эта инструкция может когда-либо возникать только в качестве первой инструкции обработчика исключений. P.S: без тавтологии никуда)

move-object vA, vB #A: 4 bits. B: 4 bits
Перемещает содержимое одного объекта, несущего регистра в другой.

move-object/16 vAAAA, vBBBB #A: 16 bits. B: 16 bits
Делает тоже самое, что и move-object. Только исходный регистр и регистр назначения 16 bits

move-object/from16 vAA, vBBBB #A: 8 bits. B: 16 bits
Делает тоже самое, что и move-object/from16. Только регистр назначения 8 bits

move-result vAA #A: 8 bits.
Переносит результат одиночного слова не объект(non-object) из самого последнего типа invoke в vAA. Это должно быть сделано как инструкция сразу после вида invoke, результат которого (однословный, не объект) не должен игнорироваться.

move-result-object vAA #A: 8 bits.
Переносит результат объекта из последнего вида invoke в vAA. Это должно выполняться как инструкция сразу после типа invoke-типа или fill-new-array, чей (объект) результат не должен игнорироваться.

move-result-wide vA+ #A: 8 bits.
#Пока пусто

move-wide vA+, vB+ #A: 4 bits. B: 16 bits
#Пока пусто

move-wide/16 vA+, vB+ #A: 16 bits. B: 16 bits
#Пока пусто

move-wide/from16 vA+, vBBBB #A: 8 bits. B: 16 bits
#Пока пусто
 # 
operations (операции)
Оператор ADD - cкладывает значения по обе стороны от оператора
 # 
add-double vA+, vB+, vC+
A: Пара регистров назначения (8 бит)
B: Пара регистров источника 1 (8 бит)
C: Пара регистров источника 2 (8 бит)
Вычисляет vB+ + vC+ и сохраняет результат в vA+

add-double/2addr vA+, vB+
A: Пара регистров источника 1 / регистр назначения (8 бит)
B: Пара регистров источника 2 (8 бит)
Вычисляет vA + vB и сохраните результат в vA+

add-float vA, vB, vC
A: Регистр назначения (4 бит)
B: Регистр источника 1 (4 бит)
C: Регистр источника 2 (4 бит)
Вычисляет vB + vC и сохраняет результат в vA

add-float/2addr vA, vB
A: регистр источника 1 / регистр назначения (4 бит)
B: регистр источника 2 (4 бит)
Вычисляет vA + vB и сохраняет результат в vA

add-int vA, vB, vC
A: регистр назначения (4 бит)
B: регистр источника 1 (4 бит)
C: регистр источника 2 (4 бит)
Вычисляет vB + vC и сохраняет результат в vA

add-int/lit8 vA, vB, 0xC
A: регистр назначения (8 бит)
B: регистр источника (8 бит)
C: подписанное постоянное значение константы (8 бит)
Вычисляет vB + 0xC и сохраняет результат в vA

add-int/lit16 vA, vB, 0xC
A: регистр назначения (4 бита)
B: регистр источника (4 бит)
C: подписанное постоянное значение константы (16 бит)
Вычисляет vB + 0xC и сохраняет результат в vA

add-int/2addr vA, vB
A: регистр источника 1 / регистр назначения (4 бит)
B: регистр источника 2 (4 бит)
Вычисляет vA + vB и сохраняет результат в vA


Оператор AND - бинарный оператор копирует бит в результат, если он существует в обоих операндах.

 # 
#Пока пусто


Оператор DIV - делит левый операнд на правый операнд

 # 
#Пока пусто


Оператор MUL - умножает значения по обе стороны от оператора

 # 
#Пока пусто


Оператор OR - копирует бит, если он существует в любом из операндов.

 # 
#Пока пусто


Оператор REM - делит левый операнд на правый операнд и возвращает остаток

 # 
#Пока пусто


Оператор SHL - значение левых операндов перемещается влево на количество бит, заданных правым операндом.

 # 
#Пока пусто


Оператор SHR - значение правых операндов перемещается вправо на количество бит, заданных левых операндом.

 # 
#Пока пусто


Оператор SUB - вычитает из правого операнда левый операнд

 # 
#Пока пусто


Оператор USHR - #описания нету

 # 
#Пока пусто


Оператор XOR - копирует бит, если он установлен в одном операнде, но не в обоих.

 # 
#Пока пусто

 # 
return
Оператор return используется для выполнения явного возврата из метода. То есть он снова передает управление объекту, который вызвал данный метод. Оператор returnпредписывает интерпретатору остановить выполнение текущего метода. Если метод возвращает значение, оператор return сопровождается некоторым выражением. Значение этого выражения становится возвращаемым значением метода.
return vAA
A: Регистр возвращаемого значения (8 bits)
Возврат из метода возвращаемого значения non-object со значением vAA.

return-object vAA
A: Регистр возвращаемого значения (8 bits)
Возврат из метода object-returning с помощью object-reference в vAA.

return-void
Возврат из метода void без значения.

return-wide vA+
A: Пара регистров возвращаемого значения (8 bits)
Возврат double/long (64-bit) значение в vA+.
 # 
switch (переключатели)
Легенда:
A: Регистр который проверяется
target: Целевая метка таблицы packed-switch(переключателей)
packed-switch vAA, :target
Реализует оператор switch, где константы case являются последовательными. Инструкция(сценарий выполнения кода) использует таблицу индексов. vAA указатели в эту таблицу, чтобы найти смещение инструкции для конкретного случая. Если vAA выпадает из таблицы индексов, выполнение продолжается в следующей команде (случай по умолчанию). pack-switch используется, если возможные значения vAA являются последовательными независимо от самого минимального значения.
Пример таблицы с переключателями:
:target
.packed-switch 0x1 # 0x1 = самое низкое/минимальное значение vAA
:pswitch_0 # Перейдет к pswitch_0 если vAA == 0x1
:pswitch_1 # Перейдет к pswitch_1 если vAA == 0x2
.end packed-switch

sparse-switch vAA, :target
Реализует оператор switch, где константы case не являются последовательными. В инструкции используется таблица поиска с константами case и смещениями для каждой константы случая. Если в таблице нет соответствия, выполнение продолжается в следующей команде (случай по умолчанию).
:target
.sparse-switch
0x3 -> :sswitch_1 # Перейдет к sswitch_1 если vAA == 0x3
0x65 -> :sswitch_2 # Перейдет к sswitch_2 если vAA == 0x65
.end sparse-switch
